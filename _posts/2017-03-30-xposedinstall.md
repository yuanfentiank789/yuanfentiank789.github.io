---

layout: post
title:  " Xposed框架介绍与安装"
date:   2017-03-30 1:05:00
catalog:  true
tags:

   - xposed
   - install
  
   
---

# Xposed

Xposed框架是一款可以在不修改APK的情况下影响程序运行（修改系统）的框架服务，通过替换/system/bin/app_process程序控制zygote进程，使得app_process在启动过程中会加载XposedBridge.jar这个jar包，从而完成对Zygote进程及其创建的Dalvik虚拟机的劫持。

基于Xposed框架可以制作出许多功能强大的模块，且在功能不冲突的情况下同时运作。此外，Xposed框架中的每一个库还可以单独下载使用，如Per APP Setting（为每个应用设置单独的dpi或修改权限）、Cydia、XPrivacy（防止隐私泄露）、BootManager（开启自启动程序管理应用）对原生Launcher替换图标等应用或功能均基于此框架。

> 官网地址： [http://repo.xposed.info/ ](http://repo.xposed.info/ )
> 
  源码地址： [https://github.com/rovo89](https://github.com/rovo89)
  
## 运行环境

Xposed框架的基本运行环境如下：

### 1 硬件部分

1. Android 4.0以上手机；
2. 手机有root权限，并刷入第三方recovery，如twrp等；

### 2 软件部分介绍

1. XposedBridge.jar：XposedBridge.jar是Xposed提供的jar文件，负责在Native层与FrameWork层进行交互。/system/bin/app_process进程启动过程中会加载该jar包，其它的Modules的开发与运行都是基于该jar包的。
2. Xposed：Xposed的C++部分，主要是用来替换/system/bin/app_process，并为XposedBridge提供JNI方法。
3. XposedInstaller：Xposed的安装包，负责配置Xposed工作的环境并且提供对基于Xposed框架的Modules的管理。
4. XposedMods：使用Xposed开发的一些Modules，其中AppSettings是一个可以进行权限动态管理的应用

   
## 安装本地服务XposedInstaller
 Xposed框架是基于一个Android的本地服务应用XposedInstaller，与一个提供API 的jar文件来完成的。所以，包含两部分apk和一个zip压缩包，安装使用Xposed框架我们需要完成以下几个步骤：

1. 安装XposedInstaller_3.1.apk，下载地址：[http://repo.xposed.info/module/de.robv.android.xposed.installer](http://repo.xposed.info/module/de.robv.android.xposed.installer)，选择一个稳定版本，下载安装。用于管理已经安装的modules，否则framework部分将无法正常工作。
2. xposed*.zip包和xposed-uninstaller*.zip，用recover刷入zip包，下载地址：
5.0以上：[http://forum.xda-developers.com/showthread.php?t=3034811](http://dl-xda.xposed.info/framework/)，根据Android版本号和平台类型选择对应的zip包。

![image](http://img.blog.csdn.net/20150814114556942)

安装好后进入XposedInstaller应用程序，会出现需要激活框架的界面，如下图所示。这里我们点击“安装/更新”就能完成框架的激活了。部分设备如果不支持直接写入的话，可以选择“安装方式”，修改为在Recovery模式下自动安装即可。


