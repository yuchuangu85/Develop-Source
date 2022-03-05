<h1 align="center">WebView</h1>

[toc]

## 基本使用

### WebView

```java
// 获取当前页面的URL
public String getUrl();
// 获取当前页面的原始URL(重定向后可能当前url不同)
// 就是http headers的Referer参数，loadUrl时为null
public String getOriginalUrl();
// 获取当前页面的标题
public String getTitle();
// 获取当前页面的favicon
public Bitmap getFavicon();
// 获取当前页面的加载进度
public int getProgress();

// 通知WebView内核网络状态
// 用于设置JS属性`window.navigator.isOnline`和产生HTML5事件`online/offline`
public void setNetworkAvailable(boolean networkUp)

// 设置初始缩放比例
public void setInitialScale(int scaleInPercent)；

```

### WebSettings

```java
WebSettings settings = web.getSettings();

// 存储(storage)
// 启用HTML5 DOM storage API，默认值 false
settings.setDomStorageEnabled(true); 
// 启用Web SQL Database API，这个设置会影响同一进程内的所有WebView，默认值 false
// 此API已不推荐使用，参考：https://www.w3.org/TR/webdatabase/
settings.setDatabaseEnabled(true);  
// 启用Application Caches API，必需设置有效的缓存路径才能生效，默认值 false
// 此API已废弃，参考：https://developer.mozilla.org/zh-CN/docs/Web/HTML/Using_the_application_cache
settings.setAppCacheEnabled(true); 
settings.setAppCachePath(context.getCacheDir().getAbsolutePath());

// 定位(location)
settings.setGeolocationEnabled(true);

// 是否保存表单数据
settings.setSaveFormData(true);
// 是否当webview调用requestFocus时为页面的某个元素设置焦点，默认值 true
settings.setNeedInitialFocus(true);  

// 是否支持viewport属性，默认值 false
// 页面通过`<meta name="viewport" ... />`自适应手机屏幕
settings.setUseWideViewPort(true);
// 是否使用overview mode加载页面，默认值 false
// 当页面宽度大于WebView宽度时，缩小使页面宽度等于WebView宽度
settings.setLoadWithOverviewMode(true);
// 布局算法
settings.setLayoutAlgorithm(WebSettings.LayoutAlgorithm.NORMAL);

// 是否支持Javascript，默认值false
settings.setJavaScriptEnabled(true); 
// 是否支持多窗口，默认值false
settings.setSupportMultipleWindows(false);
// 是否可用Javascript(window.open)打开窗口，默认值 false
settings.setJavaScriptCanOpenWindowsAutomatically(false);

// 资源访问
settings.setAllowContentAccess(true); // 是否可访问Content Provider的资源，默认值 true
settings.setAllowFileAccess(true);    // 是否可访问本地文件，默认值 true
// 是否允许通过file url加载的Javascript读取本地文件，默认值 false
settings.setAllowFileAccessFromFileURLs(false);  
// 是否允许通过file url加载的Javascript读取全部资源(包括文件,http,https)，默认值 false
settings.setAllowUniversalAccessFromFileURLs(false);

// 资源加载
settings.setLoadsImagesAutomatically(true); // 是否自动加载图片
settings.setBlockNetworkImage(false);       // 禁止加载网络图片
settings.setBlockNetworkLoads(false);       // 禁止加载所有网络资源

// 缩放(zoom)
settings.setSupportZoom(true);          // 是否支持缩放
settings.setBuiltInZoomControls(false); // 是否使用内置缩放机制
settings.setDisplayZoomControls(true);  // 是否显示内置缩放控件

// 默认文本编码，默认值 "UTF-8"
settings.setDefaultTextEncodingName("UTF-8");
settings.setDefaultFontSize(16);        // 默认文字尺寸，默认值16，取值范围1-72
settings.setDefaultFixedFontSize(16);   // 默认等宽字体尺寸，默认值16
settings.setMinimumFontSize(8);         // 最小文字尺寸，默认值 8
settings.setMinimumLogicalFontSize(8);  // 最小文字逻辑尺寸，默认值 8
settings.setTextZoom(100);              // 文字缩放百分比，默认值 100

// 字体
settings.setStandardFontFamily("sans-serif");   // 标准字体，默认值 "sans-serif"
settings.setSerifFontFamily("serif");           // 衬线字体，默认值 "serif"
settings.setSansSerifFontFamily("sans-serif");  // 无衬线字体，默认值 "sans-serif"
settings.setFixedFontFamily("monospace");       // 等宽字体，默认值 "monospace"
settings.setCursiveFontFamily("cursive");       // 手写体(草书)，默认值 "cursive"
settings.setFantasyFontFamily("fantasy");       // 幻想体，默认值 "fantasy"


if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.KITKAT) {
    // 用户是否需要通过手势播放媒体(不会自动播放)，默认值 true
    settings.setMediaPlaybackRequiresUserGesture(true);
}
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.LOLLIPOP) {
    // 5.0以上允许加载http和https混合的页面(5.0以下默认允许，5.0+默认禁止)
    settings.setMixedContentMode(WebSettings.MIXED_CONTENT_ALWAYS_ALLOW);
}
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
    // 是否在离开屏幕时光栅化(会增加内存消耗)，默认值 false
    settings.setOffscreenPreRaster(false);
}

if (isNetworkConnected(context)) {
    // 根据cache-control决定是否从网络上取数据
    settings.setCacheMode(WebSettings.LOAD_DEFAULT);
} else {
    // 没网，离线加载，优先加载缓存(即使已经过期)
    settings.setCacheMode(WebSettings.LOAD_CACHE_ELSE_NETWORK);
}

// deprecated
settings.setRenderPriority(WebSettings.RenderPriority.HIGH);
settings.setDatabasePath(context.getDir("database", Context.MODE_PRIVATE).getPath());
settings.setGeolocationDatabasePath(context.getFilesDir().getPath());

```

### WebViewClient

```java
// 拦截页面加载，返回true表示宿主app拦截并处理了该url，否则返回false由当前WebView处理
// 此方法在API24被废弃，不处理POST请求
public boolean shouldOverrideUrlLoading(WebView view, String url) {
    return false;
}

// 拦截页面加载，返回true表示宿主app拦截并处理了该url，否则返回false由当前WebView处理
// 此方法添加于API24，不处理POST请求，可拦截处理子frame的非http请求
@TargetApi(Build.VERSION_CODES.N)
public boolean shouldOverrideUrlLoading(WebView view, WebResourceRequest request) {
    return shouldOverrideUrlLoading(view, request.getUrl().toString());
}

// 此方法废弃于API21，调用于非UI线程
// 拦截资源请求并返回响应数据，返回null时WebView将继续加载资源
public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
    return null;
}

// 此方法添加于API21，调用于非UI线程
// 拦截资源请求并返回数据，返回null时WebView将继续加载资源
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public WebResourceResponse shouldInterceptRequest(WebView view, WebResourceRequest request) {
    return shouldInterceptRequest(view, request.getUrl().toString());
}

// 页面(url)开始加载
public void onPageStarted(WebView view, String url, Bitmap favicon) {
}

// 页面(url)完成加载
public void onPageFinished(WebView view, String url) {
}

// 将要加载资源(url)
public void onLoadResource(WebView view, String url) {
}

// 这个回调添加于API23，仅用于主框架的导航
// 通知应用导航到之前页面时，其遗留的WebView内容将不再被绘制。
// 这个回调可以用来决定哪些WebView可见内容能被安全地回收，以确保不显示陈旧的内容
// 它最早被调用，以此保证WebView.onDraw不会绘制任何之前页面的内容，随后绘制背景色或需要加载的新内容。
// 当HTTP响应body已经开始加载并体现在DOM上将在随后的绘制中可见时，这个方法会被调用。
// 这个回调发生在文档加载的早期，因此它的资源(css,和图像)可能不可用。
// 如果需要更细粒度的视图更新，查看 postVisualStateCallback(long, WebView.VisualStateCallback).
// 请注意这上边的所有条件也支持 postVisualStateCallback(long ,WebView.VisualStateCallback)
public void onPageCommitVisible(WebView view, String url) {
}

// 此方法废弃于API23
// 主框架加载资源时出错
public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
}

// 此方法添加于API23
// 加载资源时出错，通常意味着连接不到服务器
// 由于所有资源加载错误都会调用此方法，所以此方法应尽量逻辑简单
@TargetApi(Build.VERSION_CODES.M)
public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
    if (request.isForMainFrame()) {
        onReceivedError(view, error.getErrorCode(), error.getDescription().toString(), request.getUrl().toString());
    }
}

// 此方法添加于API23
// 在加载资源(iframe,image,js,css,ajax...)时收到了 HTTP 错误(状态码>=400)
public void onReceivedHttpError(WebView view, WebResourceRequest request, WebResourceResponse errorResponse) {
}


// 是否重新提交表单，默认不重发
public void onFormResubmission(WebView view, Message dontResend, Message resend) {
    dontResend.sendToTarget();
}

// 通知应用可以将当前的url存储在数据库中，意味着当前的访问url已经生效并被记录在内核当中。
// 此方法在网页加载过程中只会被调用一次，网页前进后退并不会回调这个函数。
public void doUpdateVisitedHistory(WebView view, String url, boolean isReload) {
}

// 加载资源时发生了一个SSL错误，应用必需响应(继续请求或取消请求)
// 处理决策可能被缓存用于后续的请求，默认行为是取消请求
public void onReceivedSslError(WebView view, SslErrorHandler handler, SslError error) {
    handler.cancel();
}

// 此方法添加于API21，在UI线程被调用
// 处理SSL客户端证书请求，必要的话可显示一个UI来提供KEY。
// 有三种响应方式：proceed()/cancel()/ignore()，默认行为是取消请求
// 如果调用proceed()或cancel()，Webview 将在内存中保存响应结果且对相同的"host:port"不会再次调用 onReceivedClientCertRequest
// 多数情况下，可通过KeyChain.choosePrivateKeyAlias启动一个Activity供用户选择合适的私钥
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public void onReceivedClientCertRequest(WebView view, ClientCertRequest request) {
    request.cancel();
}

// 处理HTTP认证请求，默认行为是取消请求
public void onReceivedHttpAuthRequest(WebView view, HttpAuthHandler handler, String host, String realm) {
    handler.cancel();
}

// 通知应用有个已授权账号自动登陆了
public void onReceivedLoginRequest(WebView view, String realm, String account, String args) {
}
// 给应用一个机会处理按键事件
// 如果返回true，WebView不处理该事件，否则WebView会一直处理，默认返回false
public boolean shouldOverrideKeyEvent(WebView view, KeyEvent event) {
    return false;
}

// 处理未被WebView消费的按键事件
// WebView总是消费按键事件，除非是系统按键或shouldOverrideKeyEvent返回true
// 此方法在按键事件分派时被异步调用
public void onUnhandledKeyEvent(WebView view, KeyEvent event) {
    super.onUnhandledKeyEvent(view, event);
}

// 通知应用页面缩放系数变化
public void onScaleChanged(WebView view, float oldScale, float newScale) {
} 

```

### WebChromeClient

```java
// 获得所有访问历史项目的列表，用于链接着色。
public void getVisitedHistory(ValueCallback<String[]> callback) {
}

// <video /> 控件在未播放时，会展示为一张海报图，HTML中可通过它的'poster'属性来指定。
// 如果未指定'poster'属性，则通过此方法提供一个默认的海报图。
public Bitmap getDefaultVideoPoster() {
    return null;
}

// 当全屏的视频正在缓冲时，此方法返回一个占位视图(比如旋转的菊花)。
public View getVideoLoadingProgressView() {
    return null;
}

// 接收当前页面的加载进度
public void onProgressChanged(WebView view, int newProgress) {
}

// 接收文档标题
public void onReceivedTitle(WebView view, String title) {
}

// 接收图标(favicon)
public void onReceivedIcon(WebView view, Bitmap icon) {
}

// Android中处理Touch Icon的方案
// http://droidyue.com/blog/2015/01/18/deal-with-touch-icon-in-android/index.html
public void onReceivedTouchIconUrl(WebView view, String url, boolean precomposed) {
}

// 通知应用当前页进入了全屏模式，此时应用必须显示一个包含网页内容的自定义View
public void onShowCustomView(View view, CustomViewCallback callback) {
}

// 通知应用当前页退出了全屏模式，此时应用必须隐藏之前显示的自定义View
public void onHideCustomView() {
}


// 显示一个alert对话框
public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
    return false;
}

// 显示一个confirm对话框
public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
    return false;
}

// 显示一个prompt对话框
public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
    return false;
}

// 显示一个对话框让用户选择是否离开当前页面
public boolean onJsBeforeUnload(WebView view, String url, String message, JsResult result) {
    return false;
}


// 指定源的网页内容在没有设置权限状态下尝试使用地理位置API。
// 从API24开始，此方法只为安全的源(https)调用，非安全的源会被自动拒绝
public void onGeolocationPermissionsShowPrompt(String origin, GeolocationPermissions.Callback callback) {
}

// 当前一个调用 onGeolocationPermissionsShowPrompt() 取消时，隐藏相关的UI。
public void onGeolocationPermissionsHidePrompt() {
}

// 通知应用打开新窗口
public boolean onCreateWindow(WebView view, boolean isDialog, boolean isUserGesture, Message resultMsg) {
    return false;
}

// 通知应用关闭窗口
public void onCloseWindow(WebView window) {
}

// 请求获取取焦点
public void onRequestFocus(WebView view) {
}

// 通知应用网页内容申请访问指定资源的权限(该权限未被授权或拒绝)
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public void onPermissionRequest(PermissionRequest request) {
    request.deny();
}

// 通知应用权限的申请被取消，隐藏相关的UI。
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public void onPermissionRequestCanceled(PermissionRequest request) {
}

// 为'<input type="file" />'显示文件选择器，返回false使用默认处理
@TargetApi(Build.VERSION_CODES.LOLLIPOP)
public boolean onShowFileChooser(WebView webView, ValueCallback<Uri[]> filePathCallback, FileChooserParams fileChooserParams) {
    return false;
}

// 接收JavaScript控制台消息
public boolean onConsoleMessage(ConsoleMessage consoleMessage) {
    return false;
} 

```

## Webview 加载优化

- 使用本地资源替代

可以 将一些资源文件放在本地的 asset s目录, 然后重 写WebViewClient 的 ``shouldInterceptRequest`` 方法，对访问地址进行拦截，当 url 地址命中本地配置的url时，使用本地资源替代，否则就使用网络上的资源。

```java
mWebview.setWebViewClient(new WebViewClient() {   
     // 设置不用系统浏览器打开,
    @Override    
    public boolean shouldOverrideUrlLoading(WebView view, String url) {      
        view.loadUrl(url);     
        return true;    
    }   
         
    @Override    
    public WebResourceResponse shouldInterceptRequest(WebView view, String url) {      // 如果命中本地资源, 使用本地资源替代      
        if (mDataHelper.hasLocalResource(url)){         
             WebResourceResponse response = mDataHelper.getReplacedWebResourceResponse(getApplicationContext(), url);          
            if (response != null) {              
                return response;
            }      
        }      
        return super.shouldInterceptRequest(view, url);    
    }   
    
    @TargetApi(VERSION_CODES.LOLLIPOP)@Override    
    public WebResourceResponse shouldInterceptRequest(WebView view,WebResourceRequest request) {      
        String url = request.getUrl().toString();      
        if (mDataHelper.hasLocalResource(url)) {         
            WebResourceResponse response =  mDataHelper.getReplacedWebResourceResponse(getApplicationContext(), url);          
            if (response != null) {              
                return response;          
            }      
        }      
        return super.shouldInterceptRequest(view, request);    
    }
}); 
```

- WebView初始化慢，可以在初始化同时先请求数据，让后端和网络不要闲着。

- 后端处理慢，可以让服务器分trunk输出，在后端计算的同时前端也加载网络静态资源。

- 脚本执行慢，就让脚本在最后运行，不阻塞页面解析。

- 同时，合理的预加载、预缓存可以让加载速度的瓶颈更小。

- WebView初始化慢，就随时初始化好一个WebView待用。

- DNS和链接慢，想办法复用客户端使用的域名和链接。

- 脚本执行慢，可以把框架代码拆分出来，在请求页面之前就执行好。

![](media/WebView.png)

## 内存泄漏

直接 new WebView 并传入 application context 代替在 XML 里面声明以防止 activity 引用被滥用，能解决90+%的 WebView 内存泄漏。

```java
vWeb =  new WebView(getContext().getApplicationContext());
container.addView(vWeb);
```

销毁 WebView

```java
if (vWeb != null) {
    vWeb.setWebViewClient(null);
    vWeb.setWebChromeClient(null);
    vWeb.loadDataWithBaseURL(null, "", "text/html", "utf-8", null);
    vWeb.clearHistory();

    ((ViewGroup) vWeb.getParent()).removeView(vWeb);
    vWeb.destroy();
    vWeb = null;
} 
```

Dd

## JS和Java通信原理

在 Android 中，Android 与js 的交互分为两个方面：Android 调用 js 里的方法、js 调用 Android 中的方法。

- **Android调js。**Android 调 js 有两种方法：
   - **WebView.loadUrl("javascript:js中的方法名")。** 这种方法的优点是很简洁，缺点是没有返回值，如果需要拿到js方法的返回值则需要js调用Android中的方法来拿到这个返回值。
   - **WebView.evaluateJavaScript("javascript:js中的方法名",ValueCallback)。** 这种方法比 loadUrl 好的是可以通过 ValueCallback 这个回调拿到 js方法的返回值。缺点是这个方法 Android4.4 才有，兼容性较差。不过放在 2018 年来说，市面上绝大多数 App 都要求最低版本是 4.4 了，所以我认为这个兼容性问题不大。
- **js 调 Android。**js 调 Android有三种方法：
   - **`WebView.addJavascriptInterface()。`** 这是官方解决 js 调用 Android 方法的方案，需要注意的是要在供 js 调用的 Android 方法上加上 **@JavascriptInterface** 注解，以避免安全漏洞。这种方案的缺点是 Android4.2 以前会有安全漏洞，不过在 4.2 以后已经修复了。同样，在 2018 年来说，兼容性问题不大。
   - **重写****`WebViewClient的shouldOverrideUrlLoading()`方法来拦截url，** 拿到 url 后进行解析，如果符合双方的规定，即可调用 Android 方法。优点是避免了 Android4.2 以前的安全漏洞，缺点也很明显，无法直接拿到调用 Android 方法的返回值，只能通过 Android 调用 js 方法来获取返回值。
   - 重写 WebChromClient 的 `onJsPrompt()` 方法，同前一个方式一样，拿到 url 之后先进行解析，如果符合双方规定，即可调用Android方法。最后如果需要返回值，通过 `result.confirm("Android方法返回值")` 即可将 Android 的返回值返回给 js。方法的优点是没有漏洞，也没有兼容性限制，同时还可以方便的获取 Android 方法的返回值。其实这里需要注意的是在 WebChromeClient 中除 了 onJsPrompt 之外还有 onJsAlert 和 onJsConfirm 方法。那么为什么不选择另两个方法呢？原因在于 onJsAlert 是没有返回值的，而 onJsConfirm 只有 true 和 false 两个返回值，同时在前端开发中 prompt 方法基本不会被调用，所以才会采用 onJsPrompt。

## 为什么WebView加载会慢呢？

这是因为在客户端中，加载H5页面之前，需要先初始化WebView，在WebView完全初始化完成之前，后续的界面加载过程都是被阻塞的。

优化手段围绕着以下两个点进行：

- 预加载WebView。
- 加载WebView的同时，请求H5页面数据。

因此常见的方法是：

- 全局WebView。
- 客户端代理页面请求。WebView初始化完成后向客户端请求数据。
- asset存放离线包。

除此之外还有一些其他的优化手段：

- 脚本执行慢，可以让脚本最后运行，不阻塞页面解析。
- DNS链接慢，可以让客户端复用使用的域名与链接。
- React框架代码执行慢，可以将这部分代码拆分出来，提前进行解析。



## 参考

