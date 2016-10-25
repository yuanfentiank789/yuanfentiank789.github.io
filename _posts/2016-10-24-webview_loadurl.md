---
layout: post
title:  "Webview资源请求的拦截"
date:   2016-10-24 1:05:00
catalog:  true
tags:

   - Webview
   - Android
   - loadurl
   - shouldOverrideUrlLoading
     


---
Webview资源请求的拦截一般有以下几种实现方法：

## 一 shouldOverrideUrlLoading和loadUrl

shouldOverrideUrlLoading方法的官方说明如下：

     /**
     * Give the host application a chance to take over the control when a new
     * url is about to be loaded in the current WebView. If WebViewClient is not
     * provided, by default WebView will ask Activity Manager to choose the
     * proper handler for the url. If WebViewClient is provided, return true
     * means the host application handles the url, while return false means the
     * current WebView handles the url.
     * This method is not called for requests using the POST "method".
     *
     * @param view The WebView that is initiating the callback.
     * @param url The url to be loaded.
     * @return True if the host application wants to leave the current WebView
     *         and handle the url itself, otherwise return false.
     */
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        return false;
    }
    
### 返回值说明

该方法位于WebViewClient中，使用分为以下三种情况：

* 1 webview未设置WebViewClient，则点击一个链接后，交由系统处理，系统会弹出浏览器列表，让用户选择。

* 2 webview设置了WebViewClient，而且shouldOverrideUrlLoading返回false（默认），则交由当前webview处理该点击事件。

* 3 webview设置了WebViewClient，而且shouldOverrideUrlLoading返回true，则交由webview的宿主应用处理。

### 调用时机

当点击h5页面上一个链接打开新的页面时，webview的loadUrl方法在其后被调用。因此可以知道，要想拦截一个新的页面可以在loadUrl方法中拦截，替换，修改url。

## shouldInterceptRequest方法

从 Android API 11 (3.0) 开始，WebView 开始在 WebViewClient 内提供了这样一条 API ，如下：

    public WebResourceResponse shouldInterceptRequest(WebView view, String url)  
    
就是说只要实现 WebViewClient 的 shouldInterceptRequest 方法，然后调用 WebView 的setWebViewClient 就可以了。

但是，在 API21 以上又弃用了上述 API，使用了一条新的 API，如下：

    public WebResourceResponse shouldInterceptRequest(WebView view, final WebResourceRequest request) 
    

使用方法如下：

    @TargetApi(Build.VERSION_CODES.LOLLIPOP)
        @Override
        public WebResourceResponse shouldInterceptRequest(WebView view, final WebResourceRequest request) {
            Uri uri = request.getUrl();
            if (uri != null) {
                WebResourceResponse response = getWebResourceResponse(uri.toString());
                if (response != null) return response;
            }

            return super.shouldInterceptRequest(view, request);
        }

        @Nullable
        private WebResourceResponse getWebResourceResponse(String url) {
            try {
                if (url != null && url.contains("kaide.com")) {
                    URL newUrl = new URL(appendDeviceInfo(url));
                    URLConnection connection = newUrl.openConnection();
                    return new WebResourceResponse(connection.getContentType(), connection.getHeaderField("encoding"), connection.getInputStream());
                }
            } catch (MalformedURLException e) {
                e.printStackTrace();
            } catch (IOException e) {
                e.printStackTrace();
            }
            return null;
        }

        @TargetApi(Build.VERSION_CODES.HONEYCOMB)
        @Override
        public WebResourceResponse shouldInterceptRequest(WebView view, String url) {
            WebResourceResponse response = getWebResourceResponse(url);
            if (response != null) return response;
            return super.shouldInterceptRequest(view, url);
        }


该方法可以拦截各种资源的请求，但需要自己处理网络访问，以及获取charset，同时不能进行耗时操作，需要开启异步线程。


