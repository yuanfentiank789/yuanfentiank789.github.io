---

layout: post
title:  "记一个launcher的bug"
date:   2016-12-14 1:05:00
catalog:  true
tags:

   - launcher
   - bug
   - home键
   
---

>  关于Android app首次安装完成后在安装界面直接“打开”应用再按home键返回桌面，重新进入app重复实例化launcher activity的问题的解决

# 复现步骤

1. 在package installers 安装界面安装完一个应用后，直接打开app，然后进入了 Activity_1, 此时再通过此activity用startActivity(intent)的方法打开 Activity_2.

2. 然后按home键返回桌面，在桌面点击app图标进入，你觉得应该进入的是 Activity_2 ，实际上却是launcher Activity_1 .
3. 然而还没完，这时候你按 back 返回键，会发现返回到了之前打开的 Activity_2，再按返回，又出现 launcherActivity_1. 也就是说系统重复实例化了Activity_1.

4. 退出app后再次点击桌面图标进入，反复试验，没有再出现这个问题。也就是说，这个问题（bug ？）只出现在操作步骤（1）后才会产生.

另外发现，首次安装后，点击完成，然后通过命令启动app也会有这个问题：

    adb shell am start -n packagename/launcheractivityname
    
 但是如果指定action和category就不会有这个问题了：
 
     adb shell am start -n pkgname/actname -a android.intent.action.MAIN -c android.intent.category.LAUNCHER
    
以上问题我在一些知名厂商的app 上发现也存在这个BUG ,比如美团的App，首次安装后，点击打开美团，点击主界面的扫一扫，然后按Home键，再点击icon启动，发现扫一扫界面不见了，美团的处理方式是给了用户一个toast提示：扫描二维码已取消。

# 分析问题

多次跟踪发现，点击icon启动App后，系统竟然再次创建了launcher Activity的实例，会执行他的oncreate方法，上面的问题，我觉得是Android系统的bug。

# 解决方案

在super.onCreate(...)方法之后插入代码：

    if (!isTaskRoot()
            && getIntent().hasCategory(Intent.CATEGORY_LAUNCHER)
            && getIntent().getAction() != null
            && getIntent().getAction().equals(Intent.ACTION_MAIN)) {

        finish();
        return;
    }
    
当然可以根据实际情况变通下。


# 参考

[https://code.google.com/p/android/issues/detail?id=14262](https://code.google.com/p/android/issues/detail?id=14262)

[https://code.google.com/p/android/issues/detail?id=2373#c40](https://code.google.com/p/android/issues/detail?id=2373#c40)
