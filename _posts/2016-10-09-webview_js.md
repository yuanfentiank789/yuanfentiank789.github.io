---
layout: post
title:  "WebView的JavaScript与本地代码三种交互方式"
date:   2016-10-09 1:05:00
catalog:  true
tags:

   - webview
   - JavaScript
     


---

# WebView的JavaScript与本地代码三种交互方式

## WebView的漏洞分析

漏洞产生的原因

最近在开发过程中遇到一个问题，就是 WebView 使用的时候，还是需要解决之前系统(4.2之前)导致的一个漏洞，虽然现在这个系统版本用户很少了，但是也不能忽视，关于这个漏洞，这里就不多做解释了，可能有的同学早就了解了，本来想写一篇文章详细介绍一下，但是网上的知识太多了，而且都很详细，就没弄了，这里大致简单明了的说几句： 
漏洞产生的原因 
这个漏洞导致的原因主要是因为Android中 WebView 中的 JS访问本地方法 的方式存在缺陷，我们做过交互的都知道，Android中的交互方式是通过 WebView 的一个方法：

    addJavascriptInterface(new JSObject(), "myObj");
    
第一个参数：Android本地对象；第二个参数：JS代码中需要使用的对象。所以这里看其实就相当于一个映射关系，把Android中的本地对象和JS中的对象关联即可。

那么这里就存在这样的一个问题了，在4.2系统之前，JS中通过对象调用的方法无需任何注解约束，那么就意味着JS中拿到这个对象，就可以调用这个对象中所有的方法。

而我们知道Android中的对象有一个公共的方法 getClass() 方法，而这个方法可以获取到当前类 类型Class，而这个类型有一个很关键的方法就是 Class.forName，这个方法可以加载一个类，这里可以加载 Java.lang.Runtime 这个类，而这个类就可以执行本地命令了，那么就会发生危险了，比如这里可以执行命令获取本地设备的SD卡中的文件等信息，非常危险的。 
上面可能说的还是有些同学不太了解，下面就用一段简单的JS代码来了解一下： 

![image](http://img.blog.csdn.net/20160930093136013)

看到这段JS之后的同学应该好理解了，因为我们Android本地通过 WebView 进行了对象映射，那么 WebView 加载页面中如果包含这段JS代码，那么就会存在这个问题，这里先遍历window中所有的对象，然后查找这个对象是否有 getClass方法，有的话就在利用反射调用 Runtime类 执行具体命令即可。其实这个漏洞也得力于JS中的语法特性，这里可以看到JS语法非常的灵活。

修复漏洞

当然对于 Android4.2 之后系统修复了这个漏洞，修复方法也很简单，加上注解约束：@JavascriptInterface 就是只有加上这个注解的方法才会被JS调用，那么我们知道getClass 是 Object类 中的，肯定没有这个注解，那么上面的JS代码肯定执行不到了。就这样解决了这个漏洞。

还有一个问题，就是Android系统默认的会给 WebView 添加一个JS映射对象：searchBoxJavaBridge_ 这个对象是 Android3.0 之后默认加上去的，也就是说通过这个对象也是可以操作的上面的代码的。

## JS代码与本地代码交互

说完了，这个漏洞，下面开始说今天的正题了，为什么要先介绍这个漏洞呢？原因就是如果要在 4.2以下版本 中解决这个漏洞的话就需要借助今天介绍的内容了，首先来看看今天的内容主要是介绍 Android 中 WebView的JS 和 本地交互 的三种方式：

### 第一种方式：通过addJavascriptInterface方法进行添加对象映射

这种方式不多解释了，也是Android中最常用的方式，但是这种方式会存在风险就是上面说到的漏洞问题。 

![image](http://img.blog.csdn.net/20160930095322495)

这里定义一个简单的本地对象，在需要被调用的方法加上约束注解。JS代码也很简单：

![image](http://img.blog.csdn.net/20160930095354739)

这种方式的好处在于使用简单明了，本地和JS的约定也很简单，就是对象名称和方法名称约定好即可，缺点就是存在漏洞问题。

![image](http://img.blog.csdn.net/20160930095422263)

### 第二种方式：利用WebViewClient接口回调方法拦截url

这种方式其实实现也很简单，就是我们可以添加 WebViewClient 回调接口，在 shouldOverrideUrlLoading 回调方法中拦截url，然后解析这个url的协议，如果发现是我们约定好的协议就开始解析参数执行具体逻辑： 

![image](http://img.blog.csdn.net/20160930095650990)

代码也很简单，这个方法可以拦截 WebView 中加载url的过程，得到对应的url，我们就可以通过这个方法，和网页JS约定好一个协议，看一下JS代码： 

![image](http://img.blog.csdn.net/20160930095720517)

这个JS代码执行之后，就会触发本地的 shouldOverrideUrlLoading 方法，然后进行参数解析，调用指定方法。

这个方法是不会存在第一种方法的漏洞问题，但是细心的同学可以发现，这里本地和JS之间的约定有点繁琐，比如要约定好协议，参数名称等信息，没有第一种方式方便。而且最主要的问题是，这个只能主动的调用本地化方法，如果想得到方法的返回值，只能通过 WebView 的 loadUrl 方法去执行JS方法，把返回值传递回去：mWebView.loadUrl(“JavaScript:clicktworesult(“+res+”)”); 

![image](http://img.blog.csdn.net/20160930095803580)

看到这种方式是非常繁琐的。在Android中也是不提倡使用的。

注意：在 iOS 中没有像 Android 中的第一种方式，他也是为了安全考虑，所以 iOS 中的交互就是采用的第二种方式，通过 拦截url 来进行操作的。 
第三种方式：利用WebChromeClient回调接口的三个方法拦截消息 
这个方法的原理其实和第二种方式差不多，只是拦截的接口方法不一样： 

![image](http://img.blog.csdn.net/20160930095956098)

Android中的 WebView 添加 WebChromeClient 接口，可以拦截JS中的几个提示方法，也就是几种样式的对话框，在JS中有三个常用的对话框方法：

alert： 是弹出警告框，在文本里面加入\n就可以换行。

confirm： 弹出确认框，会返回布尔值，通过这个值可以判断点击时确认还是取消。true表示点击了确认，false表示点击了取消。

prompt： 弹出输入框，点击确认返回输入框中的值，点击取消返回null。

那么这三种对话框都是可以在本地拦截到的，那么第三种方法的原理就是拦截这些方法，得到他们的消息内容，然后解析即可，比如这里我们拦截了 prompt 方法内容： 

![image](http://img.blog.csdn.net/20160930100107646)

本地拦截的方法参数说明： 

![image](http://img.blog.csdn.net/20160930100140961)

为什么要拦截 prompt 方法，因为这个方法可以返回想要的值，而对于 alert 是无法得到返回值的，confirm 只能得到两个返回值。只有 prompt 方法可以返回各种类型的值，操作最方便。

然后在这个方法中和第二种方法一样的原理解析消息内容即可。

执行结果 
下面直接看看上面的三种方式的执行效果： 

![image](http://img.blog.csdn.net/20160930100437464)
![image](http://img.blog.csdn.net/20160930100447714)
![image](http://img.blog.csdn.net/20160930100457149)

其中html代码如下：

![image](http://img.blog.csdn.net/20160930100529856)

## 三种方式总结

好了，到这里我们就介绍完了Android中WebView的JS和本地交互的三种方式：

第一种方式：是最普遍的用法，方便简单，但是在4.2系统以下存在漏洞问题。

第二种方式：是通过拦截url，解析约定之后的协议调用本地方法，缺点是约束协议比较繁琐，而且传回执行之后的返回值比较麻烦。但是不会存在漏洞问题，而这种方式也是iOS中采用的方式。

第三种方式：其实和第二种方式差不多，只是拦截的方法变了，这里拦截的是JS中的三种对话框方法，而这三种对话框方法的区别就在于返回值问题，alert 对话框方法没有返回值，confirm 对话框方法只有两种状态的返回值，prompt 对话框方法可以返回任意类型的返回值。缺点和第二种方法一样，协议约定比较繁琐，但是不会存在漏洞问题。

    @Override
        public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
    //            result.confirm();//无法返回给js
    //            result.cancel();//无法返回给js
            return super.onJsAlert(view, url, message, result);
        }

        @Override
        public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
            result.confirm();//var ret=confirm("are you sure?"); ret为true
            result.cancel();//var ret=confirm("are you sure?"); ret为false
            return super.onJsConfirm(view, url, message, result);
        }

    /**
         * @param view
         * @param url
         * @param message
         * @param defaultValue
         * @param result
         * @return 是否处理该js弹窗, 不处理则js负责弹窗, 处理则Android负责弹窗, 另外可以调用result的confirm方法,
         * 给js返回用户输入的结果, 一般应该是Android处理弹窗才需要调用
         */
        @Override
        public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
    //            result.cancel();//var result = prompt("typein please","10"); result 为null
    //            result.confirm();//var result = prompt("typein please","10"); 没有返回值
            result.confirm("haha");//var result = prompt("typein please","10"); result 为haha
            return super.onJsPrompt(view, url, message, defaultValue, result);
        }

## 4.2以下漏洞修复策略

最后在来介绍一下文章中开始介绍的漏洞问题，虽然google在4.2之后修复了这个漏洞，但是对于4.2之前的用户该如何处理这个漏洞呢？这里主要就是需要借助第三种方式了，拦截prompt 方法，修复步骤很简单：

1 我们自己显示一个 WebView 的包装器，重写他的 addJavascriptInterface 方法，然后在内部自己维护一个对象映射关系Map。

2 在 WebView 加载页面的方法中构造一段本地JS代码： 

![image](http://img.blog.csdn.net/20160930100657795)

关于这段JS代码的含义：

上面代码中的 jsInterface就是要注册的对象名，它注册了两个方法，onButtonClick(arg0)和onImageClick(arg0, arg1, arg2)，如果有返回值，就添加上return。

prompt 中是我们约定的字符串，它包含特定的标识符 MyApp:，后面包含了一串JSON字符串，它包含了方法名，参数，对象名等。

当JS调用 onButtonClick 或 onImageClick 时，就会回调到Java层中的 onJsPrompt 方法，我们再解析出方法名，参数，对象名，再反射调用方法。

window.jsInterface 这表示在window上声明了一个Js对象，声明方法的形式是：方法名: function(参数1，参数2)。

而加载这段JS代码的时机是什么时候呢？

刚开始时在当WebView正常加载URL后去加载JS，但发现会存在问题，如果当 WebView 跳转到下一个页面时，之前加载的JS就可能无效了，所以需要再次加载。这个问题经过尝试，需要在以下几个方法中加载Js：

它们是 WebChromeClient 和 WebViewClient 的方法：onLoadResource，doUpdateVisitedHistory，onPageStarted，onPageFinished，onReceivedTitle，onProgressChanged。

让JS调用一个Javascript方法，这个方法中是调用 prompt 方法，通过 prompt 把JS中的信息传递过来，这些信息应该是我们组合成的一段有意义的文本，可能包含：特定标识，方法名称，参数等。在 onJsPrompt 方法中，我们去解析传递过来的文本，得到方法名，参数等，再通过反射机制，调用指定的方法，从而调用到Java对象的方法。

关于返回值，可以通过 prompt 返回回去，这样就可以把Java中方法的处理结果返回到JS中。通过这几步，就可以简单的修复漏洞问题，但是还是需要注意几个问题：

需要过滤掉 Object类 的方法，由于通过反射的形式来得到指定对象的方法，他会把基类的方法也会得到，最顶层的基类就是Object，所以我们为了不把 getClass 方法注入到Js中，所以我们需要把Object的公有方法过滤掉。这里严格说来，应该有一个需要过滤方法的列表。目前我的实现中，需要过滤的方法有： “getClass”,”hashCode”,”notify”,”notifyAll”,”equals”,”toString”,”wait”。

在Android 3.0以下，系统自己添加了一个叫 searchBoxJavaBridge_ 的Js接口，要解决这个安全问题，我们也需要把这个接口删除，调用 removeJavascriptInterface 方法。这个 searchBoxJavaBridge_ 好像是跟google的搜索框相关的。

在实现过程中，我们需要判断系统版本是否在4.2以下，因为在4.2以上，Android修复了这个安全问题。我们只是需要针对4.2以下的系统作修复。 
项目案例下载地址： 
http://download.csdn.net/detail/jiangwei0910410003/9641825



## Webview调用JavaScript后获取返回值

前边部分提到了javascript获取java代码执行的返回值，主要包括两种方式：

（1）java代码执行完毕后，主动调用JavaScript函数，把值传递给JavaScript；

（2）拦截prompt方法，通过JsPromptResult的confirm方法传回返回值。

下边讨论java获取JavaScript执行后的返回值：

一般java执行js的函数为：

    /**
     * Loads the given URL.
     *
     * @param url the URL of the resource to load
     */
    public void loadUrl(String url) {
        checkThread();
        mProvider.loadUrl(url);
    }
    
 可以看到这个函数是没有返回值的，为了解决这个问题，Android在4.4以后引入了一个新的API：
 
     /**
     * Asynchronously evaluates JavaScript in the context of the currently displayed page.
     * If non-null, |resultCallback| will be invoked with any result returned from that
     * execution. This method must be called on the UI thread and the callback will
     * be made on the UI thread.
     *
     * @param script the JavaScript to execute.
     * @param resultCallback A callback to be invoked when the script execution
     *                       completes with the result of the execution (if any).
     *                       May be null if no notificaion of the result is required.
     */
    public void evaluateJavascript(String script, ValueCallback<String> resultCallback) {
        checkThread();
        mProvider.evaluateJavaScript(script, resultCallback);
    }
    
调用该函数，通过传入一个callback来接收返回值。那么在4.4以下怎么办呢？

既然我们可以在java中调用js代码，那就可以为所欲为了，分为两步：

step 1 ：向js中注入一个java对象:javaObj;

    /**
     * Passed in addJavascriptInterface of WebView to allow web views's JS execute
     * Java code
     */
    public class JavaScriptInterface {
	private final CallJavaResultInterface mCallJavaResultInterface;

	public JavaScriptInterface(CallJavaResultInterface callJavaResult) {
		mCallJavaResultInterface = callJavaResult;
	}

	@JavascriptInterface
	public void returnResultToJava(String value, int callIndex) {
		mCallJavaResultInterface.jsCallFinished(value, callIndex);
	}
    }



    final JavaScriptInterface jsInterface = new JavaScriptInterface(callJavaResult);
		mWebView.addJavascriptInterface(jsInterface, "javaObj");
		
		

step 2: 使用javaObj回调：

     mWebView.loadUrl("javaObj.returnResultToJava(eval('Math.sqrt(2 * 8);'), 2);");
     
 就可以在JavaScriptInterface中接收返回值了。
 
 该方法实现可以参考开源库：
 
 [https://github.com/evgenyneu/js-evaluator-for-android](https://github.com/evgenyneu/js-evaluator-for-android)
 
 和这个库的实现原理是一样的。
 
 但是4.2以下又有安全问题，于是终极方案出来了：
 
 通过js的prompt方法回传参数，java中拦截prompt方法。
 
 

## 总结

在Android中WebView的作用还是举足轻重的，加上现在很多应用都开始采用网页版功能，那么在这个过程中无法避免就是需要JS和本地交互，本文就详细的介绍了现阶段的三种交互方式，每种方式都有缺点和优点，当然最好的方式还是采用系统提供的也就是本文介绍的第一种方式，但是需要修复Android4.2以下存在的漏洞问题即可。

