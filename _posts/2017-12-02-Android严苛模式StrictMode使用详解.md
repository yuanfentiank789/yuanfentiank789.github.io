---
layout: post
title:  Linux C从入门到进阶
date:   2018-09-16 1:05:00
catalog:  true
tags:
    - clion
    - GCC     
       

---

StrictMode意思为严格模式，是用来检测程序中违例情况的开发者工具。使用一般是场景是检测主线程中本地磁盘和网络读写等耗时的操作。注意这个StrictMode是在Anroid2.3以后引入的。严格模式主要检测两大问题，一个是线程策略，即TreadPolicy，另一个是VM策略，即VmPolicy。

线程策略（ThreadPolicy）检测的内容有

- 1、自定义的耗时调用 使用detectCustomSlowCalls()开启
- 2、磁盘读取操作 使用detectDiskReads()开启
- 3、磁盘写入操作 使用detectDiskWrites()开启
- 4、网络操作 使用detectNetwork()开启

虚拟机策略（VmPolicy）检测的内容有

- 1、Activity泄露 使用detectActivityLeaks()开启
- 2、未关闭的Closable对象泄露 使用detectLeakedClosableObjects()开启
- 3、泄露的Sqlite对象 使用detectLeakedSqlLiteObjects()开启
- 4、检测实例数量 使用setClassInstanceLimit()开启

可以看到线程策略主要与异步处理相关，虚拟机策略主要与内存管理相关。setThreadPolicy()将对当前线程应用该策略。如果不指定检测函数，也可以用detectAll()来替代。penaltyLog()表示将警告输出到LogCat，你也可以使用其他或增加新的惩罚（penalty）函数，例如使用penaltyDeath()的话，一旦StrictMode消息被写到LogCat后应用就会崩溃。另外虚拟机策略（VmPolicy）不能通过一个对话框提供警告。

在线程策略（ThreadPolicy）检测的时候，有几个penalty系列方法。

- 1、penaltyDeath()，当触发违规条件时，直接Crash掉当前应用程序。
- 2、penaltyDeathOnNetwork()，当触发网络违规时，Crash掉当前应用程序。
- 3、penaltyDialog()，触发违规时，显示对违规信息对话框。
- 4、penaltyFlashScreen()，会造成屏幕闪烁，不过一般的设备可能没有这个功能。

### 如何使用严格模式

严格模式的开启可以放在Application或者Activity以及其他组件的onCreate方法。为了更好地分析应用中的问题，建议放在Application的onCreate方法中。设置一次就够了。

```
public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        StrictMode.setThreadPolicy(new StrictMode.ThreadPolicy.Builder()
                .detectAll()//开启所有的detectXX系列方法
                .penaltyDialog()//弹出违规提示框
                .penaltyLog()//在Logcat中打印违规日志
                .build());
        requestDataFromNet();
    }

    /**
     * 请求数据
     */
    private void requestDataFromNet() {
        URL url;
        try {
            url = new URL("http://www.baidu.com");
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.connect();
            BufferedReader reader = new BufferedReader(new InputStreamReader(conn.getInputStream()));
            String lines ;
            StringBuilder sb = new StringBuilder();
            while ((lines = reader.readLine()) != null) {
                sb.append(lines);
            }

            Log.d("response",sb.toString());
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
}
```
比如上面代码运行之后，就会有下面的弹窗出来警告。 

![](http://upload-images.jianshu.io/upload_images/1836169-d7ffd154f4648877.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

看一下logcat的日志，是在主线程中做了访问网络的操作。

```

12-19 17:28:54.226 2729-2729/? D/StrictMode: StrictMode policy violation; ~duration=44 ms: android.os.StrictMode$StrictModeNetworkViolation: policy=63 violation=4
            at android.os.StrictMode$AndroidBlockGuardPolicy.onNetwork(StrictMode.java:1123)
            at java.net.InetAddress.lookupHostByName(InetAddress.java:385)
            at java.net.InetAddress.getAllByNameImpl(InetAddress.java:236)
            at java.net.InetAddress.getAllByName(InetAddress.java:214)
            at libcore.net.http.HttpConnection.<init>(HttpConnection.java:70)
            at libcore.net.http.HttpConnection.<init>(HttpConnection.java:50)
            at libcore.net.http.HttpConnection$Address.connect(HttpConnection.java:340)
            at libcore.net.http.HttpConnectionPool.get(HttpConnectionPool.java:87)
            at libcore.net.http.HttpConnection.connect(HttpConnection.java:128)
            at libcore.net.http.HttpEngine.openSocketConnection(HttpEngine.java:316)
            at libcore.net.http.HttpEngine.connect(HttpEngine.java:311)
            at libcore.net.http.HttpEngine.sendSocketRequest(HttpEngine.java:290)
            at libcore.net.http.HttpEngine.sendRequest(HttpEngine.java:240)
            at libcore.net.http.HttpURLConnectionImpl.connect(HttpURLConnectionImpl.java:81)
            at zhangwan.wj.com.myshare.MainActivity.requestDataFromNet(MainActivity.java:35)
            at zhangwan.wj.com.myshare.MainActivity.onCreate(MainActivity.java:27)
            at android.app.Activity.performCreate(Activity.java:5104)
            at android.app.Instrumentation.callActivityOnCreate(Instrumentation.java:1092)
            at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:2148)
            at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:2254)
            at android.app.ActivityThread.access$600(ActivityThread.java:141)
            at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1234)
            at android.os.Handler.dispatchMessage(Handler.java:99)
            at android.os.Looper.loop(Looper.java:137)
            at android.app.ActivityThread.main(ActivityThread.java:5069)
            at java.lang.reflect.Method.invokeNative(Native Method)
            at java.lang.reflect.Method.invoke(Method.java:511)
            at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:793)
            at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:560)
            at dalvik.system.NativeStart.main(Native Method)
```

上面分析了是异步处理相关的例子，现在分析一个内存管相关的例子，比如检测内存泄露检测，用StrickMode怎么搞？

```

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        StrictMode.setVmPolicy(new StrictMode.VmPolicy.Builder()
                .detectActivityLeaks()//检测Activity泄露
                .penaltyLog()//在Logcat中打印违规日志
                .build());

        UserManger instance = UserManger.getInstance(this);
    }

}
```

比如上面的代码UserManger持有了Activity的引用，当反复进入Activity的时候，Activity不能被回收，导致了内存泄露，此时看Logcat。

```

12-20 09:57:07.626 4787-4787/? E/StrictMode: class zhangwan.wj.com.myshare.MainActivity; instances=3; limit=1
          android.os.StrictMode$InstanceCountViolation: class zhangwan.wj.com.myshare.MainActivity; instances=3; limit=1
          at android.os.StrictMode.setClassInstanceLimit(StrictMode.java:1)
```
明确告诉我们instances=3，说明泄露了3个MainActivity对象

严格模式除了可以检测Activity的内存泄露之外，还能自定义检测类的对象泄露。这个从从API 11 开始。

```
public
StrictMode.VmPolicy.Builder setClassInstanceLimit (Class klass, int instanceLimit)

```
比如，检测UserManger有没有泄露，可以这么写。

```
StrictMode.setVmPolicy(new VmPolicy.Builder().setClassInstanceLimit(UserManger.class, 1).penaltyLog().build());

```

在比如detectLeakedClosableObjects() 和 detectLeakedSqlLiteObjects()，资源没有正确关闭时回触发，detectLeakedRegistrationObjects() 用来检查 BroadcastReceiver 或者 ServiceConnection 注册类对象是否被正确释放等。

StrictMode有个更直接的办法，在部分手机上，可以在开发者选项中开启严格模式，开启之后，如果主线程中有执行时间长的操作，屏幕则会闪烁。OKStrictMode比较简单，到此结束！


