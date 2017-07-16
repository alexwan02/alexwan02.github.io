---
title: 'Android:WebView开发笔记（一）'
date: 2016-01-19 16:58:34
tags:
---
[讲解WebView](http://kymjs.com/code/2015/05/03/01/)

[技术小黑屋：WebView重写onJsAlert那些事](http://droidyue.com/blog/2014/07/09/override-javascript-alert-in-android/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)

[技术小黑屋：Android中WebView拦截替换网络请求数据](http://droidyue.com/blog/2014/11/23/block-web-resource-in-webview/?hmsr=toutiao.io&utm_medium=toutiao.io&utm_source=toutiao.io)
### 一、Android WebView
1.添加网络访问权限
	
```java
<uses-permission android:name="android.permission.INTERNET" />
```
2.添加JavaScript支持
```java
	WebView.getSettings().setJavaScriptEnabled(true);
```
3.只在当前WebView进行跳转，则必须覆盖`WebViewClient`对象
```java
	mWebView.setWebViewClient(new WebViewClient(){
		public boolean shouldOverrideUrlLoading(WebView view, String url){ 
			view.loadUrl(url);
			return true;
		}
	});
```
4.支持点击返回键回到上一页，而不是关闭当前Actvitiy或Fragment，则复写`onBackPress`方法。
```java
	public boolean onKeyDown(int keyCode, KeyEvent event) {
		if ((keyCode == KEYCODE_BACK) && mWebView.canGoBack()) { 
			mWebView.goBack();
			return true;
		}
		return super.onKeyDown(keyCode, event);
	}
```
5.调用页面Js方法
```java
public class WebViewDemo extends Activity{
	mWebView.addJavascriptInterface(new Object() {
		public void clickOnAndroid() {
			mHandler.post(new Runnable() {
				// 在子线程中执行
				public void run() { 
					// java 调用Js 方法
					mWebView.loadUrl("javascript:wave()");
				}
			});
		}
	}, "demo"); 
	
	mWebView.loadUrl("file:///android_asset/demo.html"); 	
	}
}
```
主要方法
```java
	addJavascriptInterface(Object obj,String interfaceName)
```
`addJavascriptInterface`将Java对象绑定到JavaScript对象中,
```javascript
	<html>
	<script language="javascript">
	  function wave() {
	    document.getElementById("droid").src="android_waving.png";
	  }
	</script>
	<body>
		// js调用java方法
	  <a onClick="window.demo.clickOnAndroid()">
	  <img id="droid" src="android_normal.png" mce_src="android_normal.png"/><br> Click me! </a>
	</body>
	</html>
```

### 二、Js调用Android代码
1.	- WebView主要用来解析和渲染界面。
	- WebViewClient 处理通知、请求事件等。
	- WebChromeClient辅助WebView处理Js的对话框、网站图标、标题等。

```java
webView.addJavascriptInterface(new JavaScriperface(this) , "android");
// 加载 htmlText 是以字符串的方式读取assets下的html内容
webView.loadData(htmlText , "text/html" , "utf-8");

// 返回上一页
webview.goBack();
```

### 三、WebView缓存
1.加载网页时，webview会在`/data/data/包名`目录下生成database与cache两个文件夹。请求url的记录保存在`WebViewCache.db`，内容保存在WebViewCache文件夹下。
	
```java
	// 启用缓存
	// 优先使用缓存
	webView.getSetting.setCacheMode(WebSettings. LOAD_CACHE_ELSE_NETWORK);
	// 不使用缓存
	webView.getSetting.setCacheMode(WebSettings.LOAD_NO_CACHE);
```
2.删除缓存
```java
private int clearCacheFolder(File dir , long numDays){
	int deletedFiles = 0;
	if(dir != null && dir.isDirectory){
		try{
			for(File child:dir.listFiles()){
				if(child.isDirectory()){
					deletedFiles += clearCacheFolder(child , numDays);
				}
				if(child.lastModified() < numDays){
					if(child.delete){
						deleteFiles++;
					}
				}
			}
		}catch (Exception e){
			e.printStackTrace();	
		}
	}
	return deletedFiles;
}

```

### 四、处理404
	
```java
webView.setWebViewClient(new WebViewClient() {
	@Override
	public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
		// stop load and handle errorcode
		if(errorCode == 404){
			// load 404 
		}
	}
}
```

### 五、判断是否滑动到底部
- getScrollY() 当前可见区域的距离整个页面顶端的距离，即当前内容滚动的距离
- getHeight() 当前WebView的高度
- getBottom() 当前WebView
- getContentHeight() 返回整个html页面的高度，不是当前页面的高度，因为webview的缩放

```java
// 是否可以缩放
webView.getSettings().setSupportZoom(true);
// 是否显示缩放控制
webView.getSettings().setBuildInZoomControls(true);
// 默认缩放大小
webView.getSettings(). setDefaultZoom(ZoomDensity.CLOSE);
```
当前整个页面的高度
```java
int currentHeight = webView.getContentHeight * getScale();
boolean isBottom = currentHeight - webView.getHeight() - webView.getScroll() <= 0
```

### 六、服务器Session

```Java
CookieManager manager = CookieManager.getInstance();
// 删除cookie
manager.removeAllCookie();
// 获取cookie
manager.getCookie(url);
// 设置cookie，覆盖之前的path，host，name的cookie
manager.setCookie(url , cookie);
```
1.设置和清除SessionCookies

```java
/**
*  同步本地cookie到Webview中
*/
public void synCookies(){
	if(!CacheUtils.isLogin(this)) return;
	CookiesSyncManager.createInstance(this);
	CookieManager manager = CookieManager.getInstance();
	// 是否发送和接收cookies，默认为true
	manager.setAcceptCookie(true);
	// 移除session cookie
	manager.removeSessionCookies(new ValueCallback<Boolean>(){
			public void onReceiveValue(Boolean value){
				//TODO
			}
	});
	String cookies = PreferenceHelper.readString(this , key , key);
	manager.setCookie(url , cookies);
	CookieSyncManager.getInstance().sync();
}
```

2.WebView本身也记录html缓存
	
```java
// 清除缓存
webview.clearCache(true);
webview.clearHistory();
```
### 七、Android中WebView替换网络资源
`shouldInterceptRequest`
通知主程序WebView处理资源文件，允许主程序返回处理后的数据。如果返回数据为null，WebView会自行加载网络资源，否则使用主程序提供的数据。回调是在子线程中处理，不能直接进行UI操作。
```java
    @Deprecated
    public WebResourceResponse shouldInterceptRequest(WebView view,
            String url) {
        return null;
    }
	
    public WebResourceResponse shouldInterceptRequest(WebView view,
            WebResourceRequest request) {
        return shouldInterceptRequest(view, request.getUrl().toString());
    }
```
Example
```java
WebView webView = new WebView(this);
webView.setWebViewClient(new WebViewClient(){
	@Override
	public WebResourceResponse shouldInterceptRequest(WebView view,
            String url) {
		WebResourceResponse response = null;
		if(url.contains("logo")){
			InputStream localCopy = null;
			try{
				localCopy = getAssets().open("xx.png");
				// MIME类型，数据编码，数据(InputStream流形式)
				response = new WebResourceResponse("image/png" , "UTF-8" , localCopy);
			} catch(Exception e){
				e.printStackTrace();
			} finally{
				try{
					if(localCopy != null){
						localCopy.close();
					}
				}catch(Exception e){
					e.printStackTrace();
				}
			}
		}
        return response;
    }
})
```