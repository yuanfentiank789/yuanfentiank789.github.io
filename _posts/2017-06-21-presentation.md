---

layout: post
title:  "跳出手机的Dialog---Presentation"
date:   2017-06-21 1:05:00
catalog:  true
tags:

   - Presentation
   - 分屏
   
   
       
   
---

[转载自http://www.th7.cn/Program/Android/201607/901164.shtml](http://www.th7.cn/Program/Android/201607/901164.shtml)

## 写在前面：

Presentation 是 what？ 
也许你刚看到标题的时候，会默默把这个单词扔到翻译工具里面，就像老大最开始跟我提起这个单词的时候一样。


## 容我想想

Presentation是说明书？ 
Presentation是一个颁奖典礼？ 
Presentation还是某卖药公司UE总监让所有IT人尴尬癌尽犯的PPT Presentation？

公司做的是智能硬件方向，国内有关Presentation资料几乎是空白的，所以我的研究更多参考了Presentation官方文档和一些英文资料。如果各位看官有什么建议，一定要记得补充。

## presentation的定义
好吧，既然上面这些都不是，Presentation是一个类，我们翻过太平洋的墙，来看看Presentation的定义。

> A presentation is a special kind of dialog whose purpose is to present content on a secondary display.

我们仅仅先来看这一句定义，因为当你对一个东西完全不了解时候，知道的越多，越会影响你的判断。

翻译下：presentation 是一种特殊的 dialog ，目的是为了在辅助屏幕上展示不同的内容。

在这句话上，我收集到了两个关键的信息：

- presentation 是一个 dialog 
- 根据生物遗传学的角度，presentation 无论被描述成什么天花乱坠的模样，它也是一个dialog。

presentation 目的是显示在辅助屏幕上

进一步思考下，也就是说：

我可以拿着我自己的手机，点击一个按钮，然后在你的电脑上或者手机上，弹出一个自定义的Dialog（脑补一下恶作剧场景O(∩_∩)O）？

这与我们之前，通过一些软件，将手机屏幕同步到电脑上，区别又在哪里呢？

相信很多人都能立刻想明白，区别在于：展示不同内容

通过软件同步到电脑，展示的东西始终与我的手机屏幕相同。

而利用 presentation 我可以自由的展示我想展示的内容，因为它是一个Dialog，是局部可控的。

寻找并投影到辅助屏幕
产品经理找到我，向我提出了以下几个疑问：

现在手里有一部Android手机

能否连接以下几种设备

另一部Android手机

笔记本电脑

智能电视

小米盒子等

并且连接之后，利用presentation展示不同内容。

我乍一看这几个设备，感觉都没问题呀。可是当我拿着手机挨个尝试，几次失败，并且耐心分析之后，发现了问题。

首先Presentation是Android 4.2引出的，与之同时Android 4.2 还支持 Miracast 影像传输协议。所以它俩一定是有联系的。


## Miracast简介

Miracast是一种基于WIFI的传输协议，Android 4.2以上的手机、Win8电脑、智能电视、盒子几乎都是支持它的。

不过Miracast它将设备分为发送端和接收端 
发送端有手机、电脑。 
接收端有智能电视、电视盒子。

所以，手机连手机或电脑展示Presentation，是行不通的。手机作为发射端，去寻找智能电视和盒子才是正解。

## Presentation
终于弄明白了要寻找的设备是怎样的，建立连接之前，参考官方文档的样例，我们先把Presentation给搭建好。

![image](http://www.th7.cn/d/file/p/2016/07/07/2118fae0b4371f9f3adc342b8a91bdfc.jpg)

可以看到，和Activity一样，可以通过setContentView来给Presentation设置一个布局。自然布局里可以有各种各样的组件，还可以有像GLSurfaceView、SurfaceView 这种重量级的组件，来显示炫酷的动画。这里我们就仅仅写一个TextView，展示一行“show a Presentation”文字。

值得一提的是，在Presentation中的getContext得到的context与它依附的Activity的context是不同的，Presentation的context是目标屏幕属性的context，包含着辅助屏幕的属性信息。

## 获取辅助屏幕
获取辅助屏幕有两种方式

- MediaRouter

- DisplayManager

### MediaRouter

利用MediaRouter的API寻找周围设备是一种最简单的方式了，它会直接绑定周围最合适的设备。就相当于你用谷歌搜索直接点击“手气不错”

代码如下：

![点击按钮，展示Presentation](http://www.th7.cn/d/file/p/2016/07/07/6050ae2d82d254dfe10f4701309742e0.jpg)

可以看到在Presentation的构造中，传入了一个display，这就是搜索到的那个设备

先来测试一下，Android 4.2的手机在开发中选项中，都有模拟辅助屏幕的功能，我们选择一个分辨率，打开它，模拟一个外部的屏幕。

![打开模拟辅助屏幕](http://www.th7.cn/d/file/p/2016/07/07/0783a1eb2498e6054cc0f5ce6b7f56cf.jpg)

默认辅助屏幕是同步手机屏幕的，打开之后，进入测试app，点击按钮：

![这里写图片描述](http://www.th7.cn/d/file/p/2016/07/07/3c5e951d043700a14298651146422b36.jpg)

注意这可不是一个Dialog，而是我们把内容展示在了一个模拟的辅助屏幕上，回头看看标题，是不是就实现了呢？

### DisplayManager

第二种搜索设备的方法是DisplayManager，他可以搜索周围所有可用的display，产生一个display数组，然后你就可以选择合适的设备进行展示了。

代码如下：

![image](http://www.th7.cn/d/file/p/2016/07/07/6a83a469799be140ec91a08ff372afce.jpg)

代码还是挺简单的，搜索到周围所有可用设备之后，展示到ListView上，点击条目，在APP上和Presentataion上分别跑一个秒表，看看延时性如何，截图如下。

![DisplayManager](http://www.th7.cn/d/file/p/2016/07/07/4ec7b73d2e895fe44e28539739e6bdb1.jpg)

可以看到，搜索到的设备名称是 叠加视图#1 ，点击条目之后两个秒表也分别跑了起来。

## 总结：

上面对Presentation进行了一个简略的介绍，因为相信大家如果做的不是智能硬件方向，基本上不会遇到这个需求。关于Activity对Presentation的管理方式，官方文档的有两个Demo可以参考，需要时可以去查看。

## 写在后面：

周末在连接智能电视测试时，跑秒表发现延时还是很大的，所以目前考虑是否可以用采用HDMI有线连接的方式来减小延时。 
关于Presentation资料比较少，欢迎大家一同交流