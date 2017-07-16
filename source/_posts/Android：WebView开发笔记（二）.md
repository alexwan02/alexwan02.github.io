---
title: Android：WebView开发笔记（二）
date: 2016-01-21 15:07:47
tags:
---
[WebView小结](http://www.jianshu.com/p/897d9e3bc783?utm_campaign=haruki&utm_content=note&utm_medium=reader_share&utm_source=qq)

[WebView相关API](http://developer.android.com/intl/ja/reference/android/webkit/WebView.html)
### 一、WebView介绍
1.权限
```java
<uses-permission android:name="android.permission.INTERNET" />
```
2.基本使用

创建有两种方式1.在layout布局文件中使用`<WebView/>`标签或在代码中动态创建 WebView对象
```java
WebView webView = new WebView(this);
setContentView(webView);

```
在浏览器中加载
```java
Uri uri = Uri.parse("http://www.example.com"); 
Intent intent = new Intent(Intent.ACTION_VIEW, uri); 
startActivity(intent);
```
webview加载页面
```java
// 加载URL
webView.loadUrl("http://slashdot.org/");

//加载String形式的HTML,
String summary = "<html><body>You scored <b>192</b> points.</body></html>";
// 有时候会显示为乱码,可以把 string 的内容也变为 utf-8的编码，统一编码格式 
// summary = new String(summary.getBytes() , "UTF-8");
webView.loadData(summary);

// 加载本地文件
webView.loadUrl("file:///android_asset/XX.html");
```
设置WebSetting
```Java
WebSettings webSetting = webView.getSettings()
// 允许JS
webSetting.setJavaScriptEnabled(true);
```
设置WebViewClient
```java
webView.setWebViewClient(new WebViewClient() {
   public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
     Toast.makeText(activity, "Oh no! " + description, Toast.LENGTH_SHORT).show();
   }
 });

```
设置WebChromeClient
```java
webView.setWebChromeClient(new WebChromeClient() {
   public void onProgressChanged(WebView view, int progress) {
     // Activities and WebViews measure progress with different scales.
     // The progress meter will automatically disappear when we reach 100%
     activity.setProgress(progress * 1000);
   }
 });
```
Js调用Native层代码

- html代码
```html
<html>
<script language="javascript">
  function wave() {
    document.getElementById("droid").src="android_waving.png";
  }
</script>
<body>
  <!-- js调用java方法 window.js_callback.clickOnAndroid().index(this)); -->
  <!-- 形式：window.指定string.函数名 -->
  <a onClick="window.android.clickOnAndroid()">
  <img id="droid" src="android_normal.png" mce_src="android_normal.png"/><br> Click me! </a>
</body>
</html>
```
- java对应的代码
```java
// 这里假设已经允许JS webSetting.setJavaScriptEnabled(true);
webView.addJavascriptInterface(new JavaScriptInterface() , "android")
// JS调用的Native函数对象
public class JavaScriptInterface{
    /*4.2以后才有@JavascriptInterface，因为安全隐患问题。
    * 因为 JavaScript 可以通过反射访问注入 webview 的 java 对象的 public fields，
    * 使用宿主程序的权限执行 java 代码。所以，4.2以后，任何为 JS 暴露接口的，都要加上@JavascriptInterface。
    * 这样，这个 Java 对象的 fields 将不允许被 js 访问。*/
    @JavascriptInterface
    public void clickOnAndroid(){
        //TODO
    };
}
```
Native调用Js

```java
webView.loadUrl("javascript:alert()");
```
### 二、WebSettings

WebView配置管理类。当WebView第一次创建时包含默认配置。通过`getter`方法来获取配置信息。调用WebView.getSetting()方法将WebSettings与WebView的生命周期绑定。如果WebView被销毁，调用任何WebView的方法都会抛出`IllegalStateException`异常信息

- setSupportZoom(boolean support) 

	是否支持缩放，配合方法`setBuiltInZoomControls`使用，默认true
- setMediaPlaybackRequiresUserGesture(boolean require) 

	是否需要用户手势来播放Media，默认true
- setBuiltInZoomControls(boolean enabled) 

	是否使用WebView内置的缩放组件，由浮动在窗口上的缩放控制和手势缩放控制组成，默认false
- setDisplayZoomControls(boolean enabled) 

	是否显示窗口悬浮的缩放控制，默认true
- setAllowFileAccess(boolean allow)

	是否允许访问WebView内部文件，默认true
- setAllowContentAccess(boolean allow)
	
	是否允许获取WebView的内容URL ，可以让WebView访问ContentPrivider存储的内容。 默认true

- setLoadWithOverviewMode(boolean overview)
	
	是否启动概述模式浏览界面，当页面宽度超过WebView显示宽度时，缩小页面适应WebView。默认false
- setSaveFormData(boolean save)
	
	是否保存表单数据，默认false
- setTextZoom(int textZoom)
	
	设置页面文字缩放百分比，默认100%
- setUseWideViewPort(boolean use)

	是否支持ViewPort的meta tag属性，如果页面有ViewPort meta tag 指定的宽度，则使用meta tag指定的值，否则默认使用宽屏的视图窗口

- setSupportMultipleWindows(boolean support)

	是否支持多窗口，如果设置为true ，`WebChromeClient#onCreateWindow`方法必须被主程序实现，默认false

- setLayoutAlgorithm(LayoutAlgorithm l)

	指定WebView的页面布局显示形式，调用该方法会引起页面重绘。默认LayoutAlgorithm#NARROW_COLUMNS
- setStandardFontFamily(String font)
	
	设置标准的字体族，默认"sans-serif"。font-family 规定元素的字体系列。font-family 可以把多个字体名称作为一个“回退”系统来保存。如果浏览器不支持第一个字体，则会尝试下一个。也就是说，font-family 属性的值是用于某个元素的字体族名称或/及类族名称的一个优先表。浏览器会使用它可识别的第一个值。

- setFixedFontFamily(String font)
	
	设置混合字体族。默认"monospace"

- setSansSerifFontFamily(String font)
	
	设置SansSerif字体族。默认"sans-serif"
- setSerifFontFamily(String font)

	设置SerifFont字体族，默认"sans-serif"
- setCursiveFontFamily(String font)

	设置CursiveFont字体族，默认"cursive"
- setFantasyFontFamily(String font)
	
	设置FantasyFont字体族，默认"fantasy"
- setMinimumFontSize(int size)

	设置最小字体，默认8. 取值区间[1-72]，超过范围，使用其上限值。
- setMinimumLogicalFontSize(int size)
	
	设置最小逻辑字体，默认8. 取值区间[1-72]，超过范围，使用其上限值。
	
- setDefaultFontSize(int size)
	
	设置默认字体大小，默认16，取值区间[1-72]，超过范围，使用其上限值。
- setDefaultFixedFontSize(int size)
	
	设置默认填充字体大小，默认16，取值区间[1-72]，超过范围，使用其上限值。
- setLoadsImagesAutomatically(boolean flag)

	设置是否加载图片资源，`注意：`方法控制所有的资源图片显示，包括嵌入的本地图片资源。使用方法`setBlockNetworkImage`则只限制网络资源图片的显示。值设置为true后，webview会自动加载网络图片。默认true
- setBlockNetworkImage(boolean flag)

	是否加载网络图片资源。`注意`如果`getLoadsImagesAutomatically`返回false，则该方法没有效果。如果使用`setBlockNetworkLoads`设置为false，该方法设置为false，也不会显示网络图片。当值从true改为false时。WebView会自动加载网络图片。
- setBlockNetworkLoads(boolean flag)
	
	设置是否加载网络资源。`注意`如果值从true切换为false后，WebView不会自动加载，除非调用WebView#reload().如果没有`android.Manifest.permission#INTERNET`权限，值设为false，则会抛出`java.lang.SecurityException`异常。默认值：有`android.Manifest.permission#INTERNET`权限时为false，其他为true。
- setJavaScriptEnabled(boolean flag)

	设置是否允许执行JS。
- setAllowUniversalAccessFromFileURLs(boolean flag)

	是否允许Js访问任何来源的内容。包括访问file scheme的URLs。考虑到安全性，限制Js访问范围默认禁用。`注意：`该方法只影响file scheme类型的资源，其他类型资源如图片类型的，不会受到影响。`ICE_CREAM_SANDWICH_MR1`版本以及以下默认为true，`JELLY_BEAN`版本以上默认为false

- setAllowFileAccessFromFileURLs(boolean flag)

	是否允许Js访问其他file scheme的URLs。包括访问file scheme的资源。考虑到安全性，限制Js访问范围默认禁用。`注意：`该方法只影响file scheme类型的资源，其他类型资源如图片类型的，不会受到影响。如果`getAllowUniversalAccessFromFileURLs`为true，则该方法被忽略。`ICE_CREAM_SANDWICH_MR1`版本以及以下默认为true，`JELLY_BEAN`版本以上默认为false

- setGeolocationDatabasePath(String databasePath)
	
	设置存储定位数据库的位置，考虑到位置权限和持久化Cache缓存，Application需要拥有指定路径的write权限
- setAppCacheEnabled(boolean flag)
	
	是否允许Cache，默认false。考虑需要存储缓存，应该为缓存指定存储路径`setAppCachePath`
- setAppCachePath(String appCachePath)
	
	设置Cache API缓存路径。为了保证可以访问Cache，Application需要拥有指定路径的write权限。该方法应该只调用一次，多次调用自动忽略。
- setDatabaseEnabled(boolean flag)
	
	是否允许数据库存储。默认false。查看`setDatabasePath` API 如何正确设置数据库存储。该设置拥有全局特性，同一进程所有WebView实例共用同一配置。`注意：`保证在同一进程的任一WebView加载页面之前修改该属性，因为在这之后设置WebView可能会忽略该配置
- setDomStorageEnabled(boolean flag)
	
	是否存储页面DOM结构，默认false。
- setGeolocationEnabled(boolean flag)

	是否允许定位，默认true。`注意：`为了保证定位可以使用，要保证以下几点：
	- Application 需要有`android.Manifest.permission#ACCESS_COARSE_LOCATION`的权限
	- Application 需要实现`WebChromeClient#onGeolocationPermissionsShowPrompt`的回调，接收Js定位请求访问地理位置的通知
- setJavaScriptCanOpenWindowsAutomatically(boolean flag)
	
	是否允许JS自动打开窗口。默认false
- setDefaultTextEncodingName(String encoding)
	
	设置页面的编码格式，默认UTF-8
- setUserAgentString(String ua)
	
	设置WebView代理，默认使用默认值
- setNeedInitialFocus(boolean flag)
	
	通知WebView是否需要设置一个节点获取焦点当`WebView#requestFocus(int, android.graphics.Rect)`被调用的时候，默认true
- setCacheMode(int mode)
	
	基于WebView导航的类型使用缓存：正常页面加载会加载缓存并按需判断内容是否需要重新验证。如果是页面返回，页面内容不会重新加载，直接从缓存中恢复。`setCacheMode`允许客户端根据指定的模式来使用缓存。
	- LOAD_DEFAULT 默认加载方式
	- LOAD_CACHE_ELSE_NETWORK 按网络情况使用缓存
	- LOAD_NO_CACHE 不使用缓存
	- LOAD_CACHE_ONLY 只使用缓存
- setMixedContentMode(int mode)
	
	设置加载不安全资源的WebView加载行为。`KITKAT`版本以及以下默认为`MIXED_CONTENT_ALWAYS_ALLOW`方式，`LOLLIPOP`默认`MIXED_CONTENT_NEVER_ALLOW`。强烈建议：使用`MIXED_CONTENT_NEVER_ALLOW`

### 三、WebViewClient
- shouldOverrideUrlLoading(WebView view, String url)

	当加载新的URL时是否由主程序控制显示。如果没有提供`WebViewClient`，则选择合适的url处理方式。如果指定`WebViewClient`，返回值为true意味主程序处理url，返回值为false意味当前`WebView`处理url。`注意：`如果使用`POST`方法该方法不会被调用。
- onPageStarted(WebView view, String url, Bitmap favicon)。

	通知主程序页面开始加载。该方法只有在加载main frame时加载一次，如果一个页面有多个frame，`onPageStarted`只在加载main frame时调用一次。也意味着若内置frame发生变化，`onPageStarted`不会被调用，如：在iframe中打开url链接。
- onPageFinished(WebView view, String url)
	
	通知主程序页面加载结束。方法只被main frame调用一次。
- onLoadResource(WebView view, String url)

	通知主程序开始加载指定url的资源文件。
- shouldInterceptRequest(WebView view,WebResourceRequest request)
	
	通知主程序资源请求，是否允许程序返回指定数据资源。如果返回值为`null`，表示继续正常加载资源文件，否则使用返回响应和值。`注意：`该方法在线程被调用而不是UI线程，所有客户端在调用私有数据或在与View交互的时候应该注意。
- onReceivedError(WebView view, int errorCode, String description, String failingUrl)

	返回访问错误通知。
- onFormResubmission(WebView view, Message dontResend, Message resend)

	是否重发POST请求数据，默认不重发。
- doUpdateVisitedHistory(WebView view, String url, boolean isReload)

	更新访问历史
- onReceivedSslError(WebView view, SslErrorHandler handler, SslError error)

	通知主程序加载资源的SSL错误。必须调用`handler.cancel()`或`handler.proceed()`。该设置可能会为之后资源访问的SSL错误保留使用当前的设定。默认`cancel`
- onReceivedClientCertRequest(WebView view, ClientCertRequest request)

	通知主程序处理SSL客户端认证请求。如果需要提供密钥，主程序负责显示UI界面。有三个响应方法：`proceed()`, `cancel()` 和 `ignore()`。如果调用`proceed()`和`cancel()`，webview将会记住response，对相同的host和port地址不再调用onReceivedClientCertRequest方法。如果调用`ignore()`方法，webview则不会记住response。该方法在UI线程中执行，在回调期间，连接被挂起。默认`cancel()`，即无客户端认证
- onReceivedHttpAuthRequest(WebView view, HttpAuthHandler handler, String host, String realm)

	通知主程序：WebView接收HTTP认证请求，主程序可以使用`HttpAuthHandler`为请求设置WebView响应。默认取消请求。
- shouldOverrideKeyEvent(WebView view, KeyEvent event)

	是否让主程序同步处理Key Event事件，如过滤菜单快捷键的Key Event事件。如果返回true，WebView不会处理Key Event，如果返回false，Key Event总是由WebView处理。默认：false
- onUnhandledInputEvent(WebView view, InputEvent event)
	
	通知主程序输入事件不是由WebView调用。是否让主程序处理WebView未处理的Input Event。除了系统按键，WebView总是消耗掉输入事件或`shouldOverrideKeyEvent`返回true。该方法由event 分发异步调用。`注意：`如果事件为`MotionEvent`，则事件的生命周期只存在方法调用过程中，如果WebViewClient想要使用这个Event，则需要复制Event对象。
- onScaleChanged(WebView view, float oldScale, float newScale)

	通知主程序WebView的大小发生变化。
- onReceivedLoginRequest(WebView view, String realm, String account, String args)
	
	通知主程序执行了自动登录请求。

### 四、WebChromeClient
- onProgressChanged(WebView view, int newProgress)
	
	通知程序当前页面加载进度
- onReceivedTitle(WebView view, String title) 
	
	通知页面标题变化
- onReceivedIcon(WebView view, Bitmap icon)
	
	通知当前页面网站新图标
-  onReceivedTouchIconUrl(WebView view, String url, boolean precomposed)

	通知主程序图标按钮URL
- CustomViewCallback
	```java
	public interface CustomViewCallback {
        // 通知当前页面自定义的View被关闭
        public void onCustomViewHidden();
    }
	```
- onShowCustomView(View view, CustomViewCallback callback)
	
	通知主程序当前页面将要显示指定方向的View，该方法用来全屏播放视频。
- onHideCustomView()
	
	与`onShowCustomView`对应，通知主程序当前页面将要关闭Custom View
- onCreateWindow(WebView view, boolean isDialog, boolean isUserGesture, Message resultMsg)

	请求主程序创建一个新的Window，如果主程序接收请求，返回true并创建一个新的WebView来装载Window，然后添加到View中，发送带有创建的WebView作为参数的resultMsg的给Target。如果主程序拒绝接收请求，则方法返回false。默认不做任何处理，返回false
- onRequestFocus(WebView view)

	显示当前WebView，为当前WebView获取焦点。
- onCloseWindow(WebView window)
	
	通知主程序关闭WebView，并从View中移除，`WebCore`停止任何的进行中的加载和JS功能。
- onJsAlert(WebView view, String url, String message, JsResult result)

	告诉客户端显示Js 提示框。如果客户端返回true，由客户端处理提示框。否则继续显示Js 提示框。
- onJsConfirm(WebView view, String url, String message, JsResult result)

	告诉客户端显示确认提示框。如果返回true，由客户端处理确认提示框，调用合适的JsResult方法。如果返回false，则返回默认值false给javascript。默认：false
- onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result)
	
	告诉客户端显示提示框。如果返回true，由客户端处理确认提示框，调用合适的JsPromptResult方法。如果返回false，则返回默认值false给javascript。默认：false
- onJsBeforeUnload(WebView view, String url, String message, JsResult result)

	告诉客户端显示离开当前页面的导航提示框。如果返回true，由客户端处理确认提示框，调用合适的JsResult方法。如果返回false，则返回默认值true给javascript接受离开当前页面的导航。默认：false。JsResult设置false，当前页面取消导航提示，否则离开当前页面。
- onGeolocationPermissionsShowPrompt(String origin, GeolocationPermissions.Callback callback)

	通知主程序web内容尝试使用定位API，但是没有相关的权限。主程序需要调用调用指定的定位权限申请的回调。更多说明查看`GeolocationPermissions`相关API。
- onGeolocationPermissionsHidePrompt()

	通知程序有定位权限请求。如果`onGeolocationPermissionsShowPrompt`权限申请操作被取消，则隐藏相关的UI界面。
- onPermissionRequest(PermissionRequest request)

	通知主程序web内容尝试申请指定资源的权限（权限没有授权或已拒绝），主程序必须调用`PermissionRequest#grant(String[])`或`PermissionRequest#deny()`。如果没有覆写该方法，默认拒绝。
- onPermissionRequestCanceled(PermissionRequest request)
	
	通知主程序相关权限被取消。任何相关UI都应该隐藏掉。
- onJsTimeout() 
	
	通知主程序 执行的Js操作超时。客户端决定是否中断`JavaScript`继续执行。如果客户端返回true，JavaScript中断执行。如果客户端返回false，则执行继续。`注意：`如果继续执行，重置JavaScript超时计时器。如果Js下一次检查点仍没有结束，则再次提示。
- onConsoleMessage(ConsoleMessage consoleMessage)
	
	通知主程序的Js控制台消息，`ChromeClient`应该覆写该方法在合适的时候来打印日志。
- getDefaultVideoPoster()
	
	当停止播放，Video显示为一张图片。默认图片可以通过HTML的Video的poster属性标签来指定。如果poster属性不存在，则使用默认的poster。该方法允许`ChromeClient`提供默认图片。
- getVideoLoadingProgressView()

	当用户重放视频，在渲染第一帧前需要花费时间去缓冲足够的数据。在缓冲期间，`ChromeClient`可以提供一个显示的View。如：可以显示一个加载动画。
- getVisitedHistory(ValueCallback<String[]> callback)

	获取访问历史Item，用于链接颜色。
- onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback,
            FileChooserParams fileChooserParams)

	通知客户端显示文件选择器。用来处理file类型的HTML标签，响应用户点击选择文件的按钮操作。调用`filePathCallback.onReceiveValue(null)`并返回true取消请求操作。
- FileChooserParams 
	
	`onShowFileChooser`方法使用的参数
	- MODE_OPEN 打开
	- MODE_OPEN_MULTIPLE 选中多个文件打开
	- MODE_OPEN_FOLDER 打开文件夹（暂不支持）
	- MODE_SAVE 保存
	- parseResult(int resultCode, Intent data)
		
		解析文件选择Activity返回的结果。需要和`createIntent`一起使用。
	- createIntent()
	
		创建Intent对象来启动文件选择器。Intent支持可访问的简单类型文件资源。不支持高级文件资源如`live media capture`媒体快照。如果需要访问这些资源或其他高级文件类型资源可以自己创建Intent对象。
	- getMode()
		
		返回文件选择模式
	- getAcceptTypes() 
	
		返回可访问`MIME`类型数组，如`audio/*`，如果没有指定可访问类型，数组返回为null
	- isCaptureEnabled()
		
		返回优先的媒体快照类型值如`Camera`、`Microphone`。true：允许快照。false，禁止快照。使用`getAcceptTypes`方法确定合适的`capture`设备。
	- getTitle()
		
		返回文件选择器的标题。如果为null，使用默认名称。
	- getFilenameHint()
		
		指定默认选中的文件名或为null
- setupAutoFill(Message msg) `暂不支持`
	
	告诉客户端当前页面以自动填充表单形式查看。
	
		

	