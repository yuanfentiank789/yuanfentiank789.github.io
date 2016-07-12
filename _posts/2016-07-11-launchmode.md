---
layout: post
title:  "Android中 的Activity launchMode"
date:   2016-07-11 1:05:00
catalog:  true
tags:
    - launchMode
   
  
       

---

# 前言
Android系统中的Activity可以说一件很赞的设计，它在内存管理上良好的设计，使得多任务管理在Android系统中运行游刃有余。但是Activity绝非启动展示在屏幕而已，其启动方式也大有学问，本文讲具体介绍Activity的启动模式的诸多细节，纠正一些开发中可能错误的观点，帮助大家深入理解Activity。

## 为何有启动模式 

应用中的每一个Activity都是进行不同的事物处理。以邮件客户端为例，InboxActivity目的就是为了展示收件箱，这个Activity不建议创建成多个实例。而ComposeMailActivity则是用来撰写邮件，可以实例化多个此Activity对象。合理地设计Activity对象是否使用已有的实例还是多次创建，会使得交互设计更加良好，也能避免很多问题。至于想要达到前面的目标，就需要使用今天的Activity启动模式。

## 如何使用

使用很简单，只需要在manifest中对应的Activity元素加入android:launchMode属性即可。如下述代码

    <activity
      android:name=".SingleTaskActivity"
      android:label="singleTask launchMode"
      android:launchMode="singleTask">
    </activity>

接下来就是介绍launchMode的四个值的时刻了。

## standard

这是launchMode的默认值，Activity不包含android:launchMode或者显示设置为standard的Activity就会使用这种模式。

一旦设置成这个值，每当有一次Intent请求，就会创建一个新的Activity实例。举个例子，如果有10个撰写邮件的Intent，那么就会创建10个ComposeMailActivity的实例来处理这些Intent。结果很明显，这种模式会创建某个Activity的多个实例。

## singletop

## singletask

## singleinstance

[http://droidyue.com/blog/2015/08/16/dive-into-android-activity-launchmode/](http://droidyue.com/blog/2015/08/16/dive-into-android-activity-launchmode/)