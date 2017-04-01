---

layout: post
title:  " Xposed模块开发指南"
date:   2017-04-01 1:05:00
catalog:  true
tags:

   - xposed
   - development
   - 模块
  
   
---

# 开发指南

> 本篇翻译自xposed作者，原文链接：[https://github.com/rovo89/XposedBridge/wiki/Development-tutorial](https://github.com/rovo89/XposedBridge/wiki/Development-tutorial)

你想要学习如何创建一个Xposed的module吗?通过阅读本文就可以，不仅包括技术上“创建这个文件,并插入…”等等,同时让你知道背后的实现原理，知其然，也知其所以然。当然你也可以只看最后的源代码和阅读[“Making the project an Xposed module”](#make)这个章节。但你会得到一个更好的理解如果你通读整篇文章。

## 开发什么Module

本文带领大家重新创建Red Clock例子可以在[Github](https://github.com/rovo89/XposedExamples/tree/master/RedClock)上找到。它包括状态栏时钟的颜色更改为红色和添加一个笑脸表情。我选择这个例子,因为它是一个很小,但容易看到效果。此外,它使用了框架提供的一些基本的方法。


## How Xposed works？
动手coding之前你应该阅读下本章节，关于Xposed的工作原理，如果觉得很无聊也可以直接跳过。

Android系统里有一个叫“Zygote”的进程，它是Android运行时的核心，每个Android应用都通过它的副本的形式被fork出来。Zygote进程在系统启动时被/init.rc脚本启动，同时启动了用来加载必要的classes和执行初始化方法的/system/bin/app_process进程。

这就到了Xposed起作用的地方，当Xposed framework安装成功后，一个扩展的app_process被拷贝到/system/bin.这个扩展的app_process添加了一个额外的jar到classpath中，以在一定的时机调用jar里的方法。比如，在VM创建完成后，甚至Zygote的main方法被调用前。这样我们就可以在这个上下文执行我们想要的操作了。

这个jar位于/data/data/de.robv.android.xposed.installer/bin/XposedBridge.jar，它的源码在[这里](https://github.com/rovo89/XposedBridge)。打开[XposedBridge](https://github.com/rovo89/XposedBridge/blob/master/src/de/robv/android/xposed/XposedBridge.java)这个类，可以看到main方法，它在app_process启动时被调用。一些初始化和modules的加载也在这个时机被完成（稍后会将module的加载）。

### Method hooking/replacing

Xposed真正强大的地方在于对Method调用的hook，如果通过反编译来验证逻辑猜想的话，只能多次的反编译代码，修改代码，重新签名打包来完成。而使用Xposed的hook功能，不用修改apk方法中的代码，通过再目标method前后插入要执行的代码即可，method是java中最小的执行单元。

XposedBridge有一个private native方法：hookMethodNative，这个方法的实现在扩展后的app_process中，这意味着，每次被hook的方法被调用时，hookMethodNative将被调用，在这个方法里，XposedBridge中的handleHookedMethod被调用，通过this参数传递了被调用method的信息。然后这个方法回调注册过该method的hook方法，对参数进行修改，修改运行结果等，非常灵活。

## 创建project


## <span id="make">Making the project an Xposed module</span>