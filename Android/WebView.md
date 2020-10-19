
[基本使用](#1)  
[WebView状态](#2)  
[前进后退](#3)  
[清除缓存](#4)  
[WebSettings](#5)  
[WebClient](#6)  
[WebChromeClient](#7)  
 
<h3 id="1"></h3>

## 基本使用
显示网页：  
 
 `webView.loadUrl("http://www.baidu.com");`  

默认使用手机浏览器打开网页,为了能够直接通过WebView显示网页，则必须设置:

```kotlin 
webView.webViewClient = object : WebViewClient() {
            override fun shouldOverrideUrlLoading(
                view: WebView?,
                request: WebResourceRequest?
            ): Boolean {

                view?.loadUrl(url)
                return true
            }
        }
```

## 使用详解
<h3 id="2"></h3>
### WebView状态
- 激活WebView为活跃状态，能正常执行网页的响应

`webView.onResume()`


- 当页面被失去焦点被切换到后台不可见状态，需要执行onPause(),通过onPause()动作通知内核暂停所有的动作，比如DOM的解析、JavaScript执行等.

`webView.onPause()`


- 当应用程序(存在webview)被切换到后台时，这个方法不仅仅针对当前的webview而是全局的全应用程序的webview,它会暂停所有webview的布局显示、解析、延时，从而降低CPU功耗.

`webView.pauseTimers()`

- 恢复pauseTimers状态

`webView.resumeTimers()`

- 销毁Webview //在关闭了Activity时，如果Webview的音乐或视频，还在播放，就必须销毁Webview。但是注意：webview调用destory时,webview仍绑定在Activity上,这是由于自定义webview构建时传入了该Activity的context对象,因此需要先从父容器中移除webview,然后再销毁webview.

`rootLayout.removeView(webView)`

`webView.destroy()`

<h3 id="3"></h3>
### 前进、后退网页

- 是否可以后退  
`Webview.canGoBack()`
- 后退网页  
`Webview.goBack()`
- 是否可以前进  
`Webview.canGoForward()`
- 前进网页  
`Webview.goForward()`
- 以当前的index为起始点前进或者后退到历史记录中指定的steps，负数后退，正数前进。  
`Webview.goBackOrForward(intsteps)`

问题：当不做任何处理时，浏览网页时，点击系统的“Back”键，则使用WebView的整个Browser浏览器是会直接调用finish()而结束自身而关闭当前Activity并返回到Home屏幕的（即直接出浏览器）；

<h3 id="4"></h3>
### 清除缓存

- 清除网页访问留下的缓存,由于内核缓存是全局的因此这个方法不仅仅针对webview而是针对整个应用程序.

`Webview.clearCache(true)`


- 只会清除webview访问历史记录里的所有记录,除了当前访问记录.

`Webview.clearHistory()`

- 这个api仅仅清除自动完成填充的表单数据，并不会清除WebView存储到本地的数据

`Webview.clearFormData()`

<h3 id="5"></h3>
### WebSetting
作用：对WebView进行配置和管理。

```kotlin
 val webSettings = webView.settings
 webSettings.javaScriptEnabled = true //如果访问的页面中要与Javascript交互，则webview必须设置支持Javascript

 //设置自适应屏幕，两者合用
 webSettings.useWideViewPort = true; //将图片调整到适合webview的大小
 webSettings.loadWithOverviewMode = true; // 缩放至屏幕的大小

 //缩放操作
 webSettings.setSupportZoom(true); //支持缩放，默认为true。是下面那个的前提。
 webSettings.builtInZoomControls = true; //设置内置的缩放控件。若为false，则该WebView不可缩放
 webSettings.displayZoomControls = false; //隐藏原生的缩放控件

 webSettings.cacheMode = WebSettings.LOAD_NO_CACHE //关闭webview中缓存
 //缓存模式如下：
 //LOAD_CACHE_ONLY: 不使用网络，只读取本地缓存数据
 //LOAD_DEFAULT: （默认）根据cache-control决定是否从网络上取数据。
 //LOAD_NO_CACHE: 不使用缓存，只从网络获取数据.
 //LOAD_CACHE_ELSE_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据

 webSettings.allowFileAccess = true; //设置可以访问文件
 webSettings.javaScriptCanOpenWindowsAutomatically = true; //支持通过JS打开新窗口
 webSettings.loadsImagesAutomatically = true; //支持自动加载图片
 webSettings.defaultTextEncodingName = "utf-8";//设置编码格式

```

<h3 id="6"></h3>
### WebClient
处理各种通知 & 请求事件

```kotlin
 webView.webViewClient = object : WebViewClient() {

            //打开网页时，不调用系统浏览器进行打开，而是在本WebView中直接显示。
            override fun shouldOverrideUrlLoading(
                view: WebView?,
                request: WebResourceRequest?
            ): Boolean {

                view?.loadUrl(request?.url.toString())
                return true
            }

            //开始载入页面时调用此方法，在这里我们可以设定一个loading的页面，告诉用户程序正在等待网络响应。
            override fun onPageStarted(view: WebView?, url: String?, favicon: Bitmap?) {
                super.onPageStarted(view, url, favicon)
            }

            //在页面加载结束时调用。我们可以关闭loading 条，切换程序动作。
            override fun onPageFinished(view: WebView?, url: String?) {
                super.onPageFinished(view, url)
            }

            //在加载页面资源时会调用，每一个资源（比如图片）的加载都会调用一次。
            override fun onLoadResource(view: WebView?, url: String?) {
                super.onLoadResource(view, url)
            }

            //加载页面的服务器出现错误时（如404）调用。
            override fun onReceivedError(
                view: WebView?,
                request: WebResourceRequest?,
                error: WebResourceError?
            ) {
                super.onReceivedError(view, request, error)
            }

            //处理https请求,webView默认是不处理https请求的，页面显示空白
            override fun onReceivedSslError(
                view: WebView?,
                handler: SslErrorHandler?,
                error: SslError?
            ) {
                super.onReceivedSslError(view, handler, error)
                handler?.proceed() //表示等待证书响应
                handler?.cancel()  //表示挂起连接，为默认方式
                handler?.handleMessage(null) //可做其他处理
            }
        }
    }

```

<h3 id="7"></h3>
### WebChromeClient类
作用：辅助 WebView 处理 Javascript 的对话框,网站图标,网站标题等等。

```kotlin

        webView.webChromeClient = object: WebChromeClient() {

            //作用：获得网页的加载进度并显示
            override fun onProgressChanged(view: WebView?, newProgress: Int) {
                super.onProgressChanged(view, newProgress)
            }

            //作用：获取Web页中的标题
            override fun onReceivedTitle(view: WebView?, title: String?) {
                super.onReceivedTitle(view, title)
            }

            //通知主机应用程序当前网页要以特定方向显示自定义视图。
            override fun onShowCustomView(view: View?, callback: CustomViewCallback?) {
                super.onShowCustomView(view, callback)
            }

            override fun onHideCustomView() {
                super.onHideCustomView()
            }
        }


```
