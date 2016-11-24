---
layout: post
title:  "Android Studio导入Android源码"
date:   2016-11-24 1:05:00
catalog:  true
tags:

   - Android Studio
   - Android源码
   
---

## 背景


因为Android官方并没有把所有java层的API暴露给我们，只把希望我们看到的部分封装成Android Sdk供我们开发用，如果我们想深入了解一些东西只能通过从AOSP下载源码了。


## 下载Android源码

一般步骤如下：

### Installing Repo

    $ mkdir ~/bin
    $ PATH=~/bin:$PATH
    $ curl https://storage.googleapis.com/git-repo-downloads/repo > ~/bin/repo
    $ chmod a+x ~/bin/repo


### Initializing a Repo client

创建源码存放目录：

    $ mkdir WORKING_DIRECTORY
    $ cd WORKING_DIRECTORY

配置git：

    $ git config --global user.name "Your Name"
    $ git config --global user.email "you@example.com"
    
运行repo init拉取repo最新代码，以及指定一个manifest的url，该manifest内部包含了Android源码中所有git仓库的地址。

    $ repo init -u https://android.googlesource.com/platform/manifest
    
或者指定分支：

    $ repo init -u https://android.googlesource.com/platform/manifest -b android-4.0.1_r1
    
### Downloading the Android Source Tree

运行如下命令：

    $ repo sync
    
接下来就是漫长的等待，多则一晚上，少则半天。

### 编译

运行envsetup.sh初始化编译环境：

    $ source build/envsetup.sh
或者

    $ . build/envsetup.sh
    
接下来：

    $ lunch aosp_arm-eng
    
    $ make -j4 //开启4个线程编译，甚至可以分布式编译


又一次漫长的等待，如果一切顺利，大概需要三四个小时吧。

## 导入源码

如果一切顺利，恭喜你可以坚持下来。

这时，你应该会在/out/host/linux-x86/framework/目录下发现有一个idegen.jar文件，这是一个很关键的文件，决定我们能不能继续。

### 编译idegen模块

执行：

    mmm development/tools/idegen/
    
或者直接运行该模块下的sh脚本：

    development/tools/idegen/idegen.sh
    
看到如下输出：

    Read excludes: 6ms
    Traversed tree: 4717ms
    
谢天谢地，终极目标达到了。

会在源码根目录发现生成一个android.ipr文件，这是我们导入源码到as必备的。

### 导入framework源码到AS

打开 Android studio，点击 File > Open，选择刚刚生成的 android.ipr 打开，等待加载好了就可以了，大功告成。

### 捷径

做了这么多，顺利的话也要一两天，不顺利的话可能就中途放弃了。

但回头想想，我们真正需要的就是https://android.googlesource.com/platform/frameworks/base这个git仓库的代码，为什么要我下载几十G的代码？无非是为了全部编译一次获得，idegen.jar这个文件.突发奇想，这个文件网上能不能下载到呢？

一搜还真有，接下来的事情。。。

来到Android在Google code的托管网站：

[https://android.googlesource.com/](https://android.googlesource.com/)

可以看到整个代码结构是由多个git仓库组成的，接下来只下载我们需要的那个git仓库：

    git clone https://android.googlesource.com/platform/frameworks/base
    
就可以只下载单个仓库了，一分钟搞定。

>  PS:下载后可以像普通git仓库一样操作他，如查看分支，pull代码等等。

然后把前边下载好的idegen.jar放到/out/host/linux-x86/framework（可以自己手动创建，out和frameworks同级）目录下，激动人心的时刻到了，打开terminal，输入：

    development/tools/idegen/idegen.sh
    
奇迹发生了，同样也生成了android.ipr.

如果编译失败，可尝试下clone下development仓库：

    git clone https://android.googlesource.com/platform/development
    
最后，附上idegen.jar文件的下载地址：

[点我下载](https://github.com/yuanfentiank789/yuanfentiank789.github.io/blob/master/asset/idegen.jar)





