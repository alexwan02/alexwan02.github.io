title: Android：Handler、Message、MessageQueue、Looper
date: 2016-06-28 09:46:10
tags: android-aosp
categories: android-aosp
thumbnailImage: http://res.cloudinary.com/dmfz9aun7/image/upload/v1467081862/android/android_aosp_code.png
---
Android中引入消息事件驱动的机制。每个Thread通过ThreadLocal来绑定一个Looper对象，每个Looper对应一个消息队列MessageQueue，在创建一个Handler时，绑定当前线程的MessageQueue，来用来接收消息和分发处理消息。

![img](http://res.cloudinary.com/dmfz9aun7/image/upload/v1467083788/android/android_message_mechanism.png)

Handler、Message、MessageQueue、Message关系图

## 一、Handler

### 1.创建一个Handler
```java
Handler handler = new Handler();
```
Handler在创建时，都做了什么呢？
{% codeblock lang:java Handler.java%}

public interface Callback {
    public boolean handlerMessage(Message msg);
}
...
/*---Handler构造器---*/
/**
* Handler默认构造器，关联当前线程的Looper对象，
* 如果当前线程Looper没有初始化， Handler则抛出异常
*/
public Handler(){
    this(null , false);
}

/**
* 关联当前线程Looper对象的Handler构造器，接受一个callback接口来处理Messages
* 如果当前线程Looper没有初始化， Handler无法接收消息并抛出异常
*/
public Handler(Callback callback){
    this(callback ,false);
}

/**
* 使用提供的Looper 替换默认Looper的
*/
public Handler(Looper looper) {
    this(looper, null, false);
}

/**
* 使用提供的Looper ，接受一个callback接口来处理消息
*/
public Handler(Looper looper, Callback callback) {
    this(looper, callback, false);
}
...
/**
* 默认Handler构造 最终会调用这个方法来初始化Handler对象
*/
public Handler(Callback callback  , boolean async){
    if(FIND_POTENTIAL_LEAKS){
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
            (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }

        // 获取当前线程绑定的Looper
        mLooper = Looper.myLooper();
        if(mLooper == null){
            throw new RuntimeException(
                "Can't create handler inside thread that has not called Looper.prepare()");
        }
        // 绑定Looper的MessageQueue
        mQueue = mLooper.mQueue;
        mCallback = callback;
        mAsynchronous = async;
	}

}

/**
* 指定Looper，Callback 初始化Handler
* Handler默认为同步调用
*/
public Handler(Looper looper, Callback callback, boolean async) {
    mLooper = looper;
    mQueue = looper.mQueue;
    mCallback = callback;
    mAsynchronous = async;
}
...
{% endcodeblock %}

每个Handler都关联一个消息队列，消息队列被封装在Looper中，每个Looper又会关联一个Thread，也就是每个消息队列都会关联一个Thread。
### 2.发送消息
Handler就是一个消息处理器，将消息投递给消息队列，然后再由对应的线程从消息队列中取出消息。
Handler发送消息分为postXXX和sendXXX方法，postXXX方法，接受Runnable对象作为参数
{% codeblock lang:java Handler.java%}

// post
...
public final boolean post(Runnable r){
    return sendMessageDelay(getPostMessage(r) , 0);
}
...
// 通过getPostMessage方法把Runnable作为参数分装为Message
private static Message getPostMessage(Runnable r){
    Message m = Message.obtain();
    m.callback = r;
    return m;
}

{% endcodeblock %}

sendXXX方法用Message对象作为参数。
发送Message的方法最后都调用sendMessageAtTime方法，

{% codeblock lang:java Handler.java%}
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    // 获得绑定的MessageQueue
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
...
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    // 将Message target 与当前Handler对象绑定
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    // 调用MessageQueue的enqueueMessage方法，将消息加入消息队列中
    return queue.enqueueMessage(msg, uptimeMillis);
}
{% endcodeblock %}

Handler发送消息，通过在初始化时绑定的MessageQueue对象，将要发送的消息加入到当线程的唯一的MessageQueue队列中。
那么Handler在哪里来处理从消息队列中发送出的消息呢？答案在Looper源码中

## 二、Looper
Looper 用来运行循环线程中消息队列，但是默认情况下Thread并没有关联的消息循环。Looper提供`prepare()`方法将一个Looper对象与Thread来进行关联。并调用`loop()`方法来开启消息队列的循环任务。比如下面是Looper与Thread关联的经典范例。
{% codeblock lang:java Looper.java%}
class LooperThread extends Thread{
    public Handler mHandler;
    public void run(){
        // init looper here
        Looper.prepare();
        mHandler = new Handler(){
            public void handleMessage(Message msg) {
                // 处理接收到的消息
            }
        };
         // start loop message queue here actually
         Looper.loop();
    }
}
... 
{% endcodeblock %}

Looper调用`prepare()`将当前线程与Looper对象进行关联

```java
/*
* 构造方法为private ， 不能直接通过构造方法直接初始化
* 初始化MessageQueue，与当前线程进行关联
*/
private Looper(boolean quitAllowed) {
    mQueue = new MessageQueue(quitAllowed);
    mThread = Thread.currentThread();
}

/**
* 初始化当前线程为Looper
*/
public static void prepare() {
    prepare(true);
}

// 实际调用
private static void prepare(boolean quitAllowed){
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}


```
Looper在初始化时，先通过ThreadLocal获取当前线程关联的Looper对象，判断是否Thread已经关联过Looper，如果已经关联，抛出异常。如果未关联，在将实例化一个新的Looper实例，并保存到ThreadLocal中
在实例化Looper中，创建MessageQueue对象，关联当前线程。
### 1.ThreadLocal
什么是ThreadLocal，ThreadLocal是一个关于创建线程局部变量的类。使用ThreadLocal创建的变量只能被当前线程访问，其他线程则无法访问和修改。
- 实现单个线程单例以及当个线程上下文信息存储
- 实现线程安全，非线程安全的对象使用ThreadLocal之后变为线程安全。因为每个线程都对应一个实例。
- 承载一些线程相关的数据，避免在方法中来回传递参数
- ThreadLocal不会产生内存泄露，ThreadLocal选做Key时，是ThreadLocal实例的弱引用

```java 

// a simple threadlocal sample

Thread t = new Thread{
    ThreadLocal<String> mStringThreadLocal = new ThreadLocal<String>(){
        public void run(){
            super.run();
            mStringThreadLocal.set("alexwan");
            mStringThreadLocal.get();
        }
    }
}
...
t.start();

```
ThreadLocal的`set`方法把当前线程作为key，把需要存储的变量作为值存储在ThreadLocalMap中
{% codeblock lang:java ThreadLocal %}
public class ThreadLocal {
    public ThreadLocal(){}
    ...
    public void set(T value){
        // get current thread
        Thread t = Thread.currentThread();
        // get thread local map values;
        ThreadLocalMap map = getMap(t);
        if( map != null ){
            map.set(this , value);
        } else {
            // create local values map
            createMap(t , value);
         }
    }
    ...
    ThreadLocalMap getMap(Thread t){
        return t.threadLocals;
    }
    ...
}
{% endcodeblock %}

Looper 本地维护一个ThreadLocal对象，保证当前线程中Looper对象的唯一性，

{% codeblock lang:java Looper%}
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
{% endcodeblock %}

### 2.循环消息队列
执行Looper.prepare()之后，可以在外部通过调用Looper.myLooper()获取当前线程关联的Looper对象。
```java
public static Looper myLooper(){
    return sThreadLocal.get();
}
```
Looper初始化后，调用方法`loop()`开始执行循环消息的任务

{% codeblock lang:java Looper %}

/**
* 执行循环消息任务，调用quit()方法结束循环
*/
public static void loop() {
    // 获取当前线程关联的Looper对象  
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    // Looper实例化时创建的MessageQueue对象
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) {
        // 无限循环消息队列，调用MessageQueue next()方法取出消息
        Message msg = queue.next(); // 可能会阻塞
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }
        // 这里的target对象即是Handler，这里调用的Handler dispatchMessage()方法来处理
        // 从消息队列中取出的Message
        msg.target.dispatchMessage(msg);

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked();
    }
}

{% endcodeblock %}

Looper获取当前线程关联的Looper对象，通过Looper对象拿到Looper创建的MessageQueue消息队列，对消息队列开启无限循环。从MessageQueue中取消息，如果队列无消息，则等待新的消息。取到消息时，则由Masssge对象绑定的target来分发处理消息。在Handler分析的中Message的target关联就是发送消息的Handler对象。调用Handler的`dispatchMessage `方法让Handler处理Message

{% codeblock lang:java Handler %}
/**
* Handle system messages here.
*/
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
{% endcodeblock %}

dispatchMessage 只是一个分发的方法，如果callback不为空，表示是Handler是调用`postxxx`方法发送的消息，callback实际为Runnable对象，此时调用`handleCallback`方法，则执行callback的run方法。否则判断mCallback是否为空，mCallback是在创建Handler时传递的Callbacl接口对象。如果mCallback接口对象也为空，则具体实现由handler的方法`handleMessage`实现

```java
private static void handleCallback(Message message) {
    message.callback.run();
}
...
/**
* 子类具体实现
*/
public void handleMessage(Message msg) {
}

```

## 三、主线程MainThread

Android的程序入口为ActivityThread的main方法，而主线程的Looper也是在这里初始化

{% codeblock lang:java ActivityThread %}

public static void main(String[] args){
    ...
    // Make sure TrustedCertificateStore looks in the right place for CA certificates
    final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
    TrustedCertificateStore.setDefaultUserDirectory(configDir);

    Process.setArgV0("<pre-initialized>");
    // 初始化主线程的Looper
    Looper.prepareMainLooper();

    ActivityThread thread = new ActivityThread();
    ...

    // End of event ActivityThreadMain.
    Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
    // 开启消息循环
    Looper.loop();
    ...
}
...
{% endcodeblock %}

主线程的消息队列在AcitivityThread的main中，调用Looper的静态方法prepareMainLooper()来创建，最后调用looper来启动消息循环。
在很多情况下，在线程中执行耗时任务，通过调用主线程Handler更新UI。

```java
Handler handler = new Handler(Looper.getMainLooper()){
    @Override
    public void handleMessage(Message msg) {
        // 更新界面UI TODO
    }
};

Thread thread = new Thread(new Runnable() {
    @Override
    public void run() {       
        // 处理耗时任务
        Message msg = new Message();
        // message 设置处理的结果
        // 发送带有处理结果的消息
        handler.sendMessage(msg);
    }
});

```
## 四、总结

Android的消息事件机制基本的流程：线程关联有且唯一的Looper对象，Looper对象中维护一个消息队列MessageQueue对象。通过Handler来发送或处理消息Message或Runnable的属性的Message，加入到队列中。Looper对象开启无限的消息循环，来取队列中Message，调用发送消息的Handler的消息分发方法，由开发者来具体处理业务或UI界面的操作。