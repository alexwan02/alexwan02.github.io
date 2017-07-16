---
title: 深入理解OkHttp3的Connections（3）
date: 2017-04-24 17:34:07
tags: okhttp3
categories: okhttp3
---
![img](http://res.cloudinary.com/dmfz9aun7/image/upload/v1492768247/wechat/OkHttpCore.jpg)
OkHttp3的网络连接创建和数据传输由责任链网络层的`ConnectInterceptor`和`CallServerInterceptor`完成。`ConnectInterceptor`为请求创建与服务器网络连接，`CallServerInterceptor`负责网络数据读写的具体操作。

![img](http://res.cloudinary.com/dmfz9aun7/image/upload/v1493002739/wechat/OkHttp_connection.jpg)
## 1. StreamAllocation
StreamAllocation类似中介者模式，协调Connections、Stream和Call三者之间的关系。每个Call在Application层`RetryAndFollowUpInterceptor`实例化一个`StreamAllocation`。

相同Address（相同的Host与端口）可以共用相同的连接RealConnection。
1. StreamAllocation通过Address，从连接池ConnectionPools中取出有效的RealConnection，与远程服务器建立Socket连接。
2. 在处理响应结束后或出现网络异常时，释放Socket连接。
3. 每个RealConnection都持有对StreamAllocation的弱引用，用于连接闲置状态的判断。

查找可用连接
```java
  private RealConnection findConnection(int connectTimeout, int readTimeout, int writeTimeout,
      boolean connectionRetryEnabled) throws IOException {
    Route selectedRoute;
    synchronized (connectionPool) {
      // prediction check
      ...
      // 尝试使用已分配的连接
      ...
      // 尝试从连接池获取
      Internal.instance.get(connectionPool, address, this);
      ...
      selectedRoute = route;
    }

    // 未路由寻址时，查找可用路由
    ...

    // 立即创建RealConnection
    RealConnection result;
    // 并发
    synchronized (connectionPool) {
      route = selectedRoute;
      refusedStreamCount = 0;
      result = new RealConnection(connectionPool, selectedRoute);
      ...
      if (canceled) throw new IOException("Canceled");
    }

    // 进行TCP和TLS握手，建立与服务器连接通道
    result.connect(connectTimeout, readTimeout, writeTimeout, connectionRetryEnabled);
    // 路由可用，从失败路由表中移除
    routeDatabase().connected(result.route());

    Socket socket = null;
    synchronized (connectionPool) {
      // 添加到连接池，如果其他并发线程已创建同个多路复用的连接，则丢弃当前的连接
      // 并释放Socket资源
      ...
    }
    ...
    return result;
  }
```

## 2. ConnectionPool

Okhttp3封装了网络连接的逻辑，通过`RealConnection`建立Socket连接。因为建立Socket网络连接会增加网络延迟，尤其是对于应用程序客户端的频繁的网络请求，需要建立HTTP和HTTP/2的重用策略，降低网络连接代理性能损耗。

`ConnectionPool`针对于相同Address的请求复用同一个connection，，维护`RealConnection`类似循环队列形式的双端队列。
ConnectionPool默认的最大空闲连接数为5，最大的空闲时间为5分钟。

```java
  public ConnectionPool() {
    this(5, 5, TimeUnit.MINUTES);
  }
```

ConnectionPool使用ArrayDeque来维护池内的连接。`ArrayDeque`大多数操作时间复杂度为O(1)。对于remove、removeFirstOccurrence、removeLastOccurrence、contains和iterator.remove()方法，时间复杂度为O(n)，线性复杂度。双端队列中的元素可以从两端弹出，插入和删除操作限定在队列的两边进行。
### 2.1. 循环数组
```java
private final Deque<RealConnection> connections = new ArrayDeque<>();
```
ArrayDeque是双端队列的数组的实现，`ArrayDeque`是一个可变大小的循环数组，没有长度大小限制，非线程安全，因此不支持多线程并发，需要对数组的操作需要加上同步锁。
它的默认长度是16。
```java
    public ArrayDeque() {
        elements = new Object[16];
    }
```
设定指定长度数组时，则初始长度为比选择的数大的最小2的指数的整数。
```java
    private void allocateElements(int numElements) {
        int initialCapacity = MIN_INITIAL_CAPACITY;
        // 移位操作将最高位以下的为置为1，正好比
        if (numElements >= initialCapacity) {
            initialCapacity = numElements;
            initialCapacity |= (initialCapacity >>>  1);
            initialCapacity |= (initialCapacity >>>  2);
            initialCapacity |= (initialCapacity >>>  4);
            initialCapacity |= (initialCapacity >>>  8);
            initialCapacity |= (initialCapacity >>> 16);
            initialCapacity++; 
            ...
        }
        elements = new Object[initialCapacity];
    }
```

1. 添加连接
```java
  void put(RealConnection connection) {
    ...
    // 是否需要清理连接池
    ...
    connections.add(connection);
  }
```
2. 查找连接
```java
// 使用迭代来查找连接池连接
connections.iterator()
```
3. 清理连接
```java
  // 如果返回true , 表示连接已经从连接池移除，需要立刻关闭
  boolean connectionBecameIdle(RealConnection connection) {
    assert (Thread.holdsLock(this));
    if (connection.noNewStreams || maxIdleConnections == 0) {
      connections.remove(connection);
      return true;
    } else {
      // 唤醒清理线程，因为可能超过了最大空闲连接数
      notifyAll();
      return false;
    }
  }

```

### 2.2. 连接清理
内置的清理线程会定时对连接池检查，清理空闲连接。根据空闲连接的最大空闲时间和连接数，判断执行清理操作。

```java
  private final Runnable cleanupRunnable = new Runnable() {
    @Override public void run() {
      while (true) {
        long waitNanos = cleanup(System.nanoTime());
        // waitNanos等于-1表示无空闲或在用的连接
        ...
        // 如果有空闲或在用连接则定时启动清理线程
        ...
      }
    }
  };
  
  /**
  * 维护连接池，清理超过最大空闲存活时间或数量的空闲连接。
  * 返回下次调用这个方法的纳秒大小的睡眠时间。返回-1表示无须再次执行。
  */
  long cleanup(long now) {
    int inUseConnectionCount = 0;  // 在用连接数
    int idleConnectionCount = 0;   // 空闲连接数
    RealConnection longestIdleConnection = null; // 临时最大空闲连接
    long longestIdleDurationNs = Long.MIN_VALUE; // 最大空闲持续时间

    ...
    synchronized (this) {
      for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
        RealConnection connection = i.next();
        // 迭代查找最大空闲时间的连接，记录空闲连接和在用连接数
        ...
      }

      if (longestIdleDurationNs >= this.keepAliveDurationNs
          || idleConnectionCount > this.maxIdleConnections) {
        // 超过最大空闲连接时间或空闲数据超过最大空闲连接数，从连接池移除连接
        connections.remove(longestIdleConnection);
      } else if (idleConnectionCount > 0) {
        // 有空闲连接但是未超标，返回下次执行清理的时间间隔
        return keepAliveDurationNs - longestIdleDurationNs;
      } else if (inUseConnectionCount > 0) {
        // 所有连接都在使用，返回最大连接时间
        return keepAliveDurationNs;
      } else {
        // 无空闲和在用时间
        cleanupRunning = false;
        return -1;
      }
    }
    ...
    // 立即清理
    return 0;
  }
```
除了自动清理，还支持手动清理空闲的连接
```java
  public void evictAll() {
    List<RealConnection> evictedConnections = new ArrayList<>();
    synchronized (this) {
      for (Iterator<RealConnection> i = connections.iterator(); i.hasNext(); ) {
        RealConnection connection = i.next();
        if (connection.allocations.isEmpty()) {
          // 如果处于空闲状态，直接移除
          connection.noNewStreams = true;
          evictedConnections.add(connection);
          i.remove();
        }
      }
    }

    for (RealConnection connection : evictedConnections) {
      // 循环关闭连接
    }
  }
```
### 2.3. Address
连接池通过Address来判断是否有可复用的连接。依据服务器地址、端口号判断是否相同的Address
```java
@Address.java 
  @Override public boolean equals(Object other) {
    if (other instanceof Address) {
      Address that = (Address) other;
      return this.url.equals(that.url) && ...;
    }
    return false;
  }
```
`ConnectionPools`中不存在对应的Address的RealConnection则须进行路由寻址，初始化RealConnection，执行网络建立和传输操作。
## 3. RealConnection
初始化一个连接需要`ConnectionPool`和`Route`两个参数
```java
  public RealConnection(ConnectionPool connectionPool, Route route) {
    this.connectionPool = connectionPool;
    this.route = route;
  }
```
`Route` 在创建`RealConnection`实例前，执行路由寻址操作，找到可用的路由。出现异常，则返回到应用层进行重试处理。
## 3.1. 路由寻址
路由寻址为指定Address初始化Route实例。
```java
  public Route next() throws IOException {
    // Compute the next route to attempt.
    if (!hasNextInetSocketAddress()) {
      // 未进行寻址
      if (!hasNextProxy()) {
        // 无可用代理，尝试之前连接失败的路由
        ...
        return nextPostponed();
      }
      // 代理
      lastProxy = nextProxy();
    }
    lastInetSocketAddress = nextInetSocketAddress();

    Route route = new Route(address, lastProxy, lastInetSocketAddress);
    if (routeDatabase.shouldPostpone(route)) {
      // 如果路由曾失败过，添加到待用的路由表中
      postponedRoutes.add(route);
      // 继续查找合适的路由
      return next();
    }

    return route;
  }
```
代理设置，配置连接的host和端口号。
```java
  private void resetNextInetSocketAddress(Proxy proxy) throws IOException {
    // Clear the addresses. Necessary if getAllByName() below throws!
    inetSocketAddresses = new ArrayList<>();

    String socketHost;
    int socketPort;
    if (proxy.type() == Proxy.Type.DIRECT || proxy.type() == Proxy.Type.SOCKS) {
      // 无代理
      socketHost = address.url().host();
      socketPort = address.url().port();
    } else {
      // 代理
      ....
    }

    // 端口号检查
    ...
    if (proxy.type() == Proxy.Type.SOCKS) {
      ...
    } else {
      // Dns寻址
      List<InetAddress> addresses = address.dns().lookup(socketHost);
      for (int i = 0, size = addresses.size(); i < size; i++) {
        InetAddress inetAddress = addresses.get(i);
        inetSocketAddresses.add(new InetSocketAddress(inetAddress, socketPort));
      }
    }
    ...
  }
```
路由寻址成功后，初始化RealConnection并添加到连接池中，建立Socket连接
### 3.2. Socket连接
OkHttp3支持HTTP与HTTP/2。HTTP/2需要进行TLS的握手流程。
```java
@StreamAllocation.java
public void connect(
      int connectTimeout, int readTimeout, int writeTimeout, boolean connectionRetryEnabled) {
    if (protocol != null) throw new IllegalStateException("already connected");

    RouteException routeException = null;
    List<ConnectionSpec> connectionSpecs = route.address().connectionSpecs();
    ConnectionSpecSelector connectionSpecSelector = new ConnectionSpecSelector(connectionSpecs);

    if (route.address().sslSocketFactory() == null) {
      if (!connectionSpecs.contains(ConnectionSpec.CLEARTEXT)) {
        throw new RouteException(new UnknownServiceException(
            "CLEARTEXT communication not enabled for client"));
      }
      String host = route.address().url().host();
      if (!Platform.get().isCleartextTrafficPermitted(host)) {
        throw new RouteException(new UnknownServiceException(
            "CLEARTEXT communication to " + host + " not permitted by network security policy"));
      }
    }

    while (true) {
      try {
        if (route.requiresTunnel()) {
          // 使用隧道在不兼容的网络上传输数据，或在不安全网络上提供安全路径
          connectTunnel(connectTimeout, readTimeout, writeTimeout);
        } else {
          // 直接建立socket连接
          connectSocket(connectTimeout, readTimeout);
        }
        // 协议连接方式，HTTP/1或者HTTP/2
        establishProtocol(connectionSpecSelector);
        break;
      } catch (IOException e) {
        ...
      }
    }

    if (http2Connection != null) {
      synchronized (connectionPool) {
        allocationLimit = http2Connection.maxConcurrentStreams();
      }
    }
  }
```
TLS 握手流程

![img](http://res.cloudinary.com/dmfz9aun7/image/upload/v1493025333/wechat/tls_handshake.jpg)

建立Socket连接成功之后，客户端就可以与服务端进行通讯了。
