---
layout: post
title:  "android打印调用栈的方法"
date:   2016-11-17 1:05:00
catalog:  true
tags:

   - Android
   - stacktrace
   - C
   
     


---

## android打印调用栈的方法

打印调用栈是android平台问题定位的基本方法，如果需要知道谁在调用某个函数，可以在此函数中添加打印调用栈函数，弄清楚函数之间的调用关系。

## Java层打印调用栈方法

    RuntimeException here = new RuntimeException("here");
    here.fillInStackTrace();
    Log.w(TAG, "Called: " + this, here);


## C++层打印调用栈方法

    CallStack stack;
    stack.update();
    stack.dump();

备注：下面操作是可选操作，但加上去之后会有一些额外的功能

 #define HAVE_DLADDR 1 ：可以从lib中自己转成c++代码行，不需要手动反编译
 #define HAVE_CXXABI 1：将c++ 已被name mangling的函数名转化为源文件中定义的函数名。
并在文件frameworks/base/libs/utils/Android.mk中大约105行（LOCAL_SHARED_LIBRARIES）后添加
ifeq ($(TARGET_OS),linux)
LOCAL_SHARED_LIBRARIES += libdl
endif重新编译push生成的libutils.so到/system/lib/目录下，重启设备。
此外，由于CallStack.dump中使用的LOGD进行的打印，因此需要将后台的Log Level设置为D一下才能出来。

## C函数打印调用栈

可以参考CallStack.cpp的实现，通过调用_Unwind_Backtrace完成。

## Kernel层打印调用栈方法

dump_stack();函数

