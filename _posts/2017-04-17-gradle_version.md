---

layout: post
title:  "Android Studio中gradle版本对应关系"
date:   2017-04-17 1:05:00
catalog:  true
tags:

   - gradle
   - gradle plugin版本
   - SDK Build Tools版本
    
   
---
# Android Studio中gradle版本对应关系
Android studio 编译需要保证：SDK Build Tools 版本，Gradle 版本，Gradle Plugin 版本 兼容。

这里我们要重点关注Gradle版本的版本 ，因为版本决定了 SDK Build Tools 版本与 Gradle Plugin 版本。

## 查看Android studio 使用的Gradle 版本
打开 目录：File ->Setting -> Build、Execution,Deployment -> Gradle ，如图：
![image](http://static.open-open.com/lib/uploadImg/20170323/20170323150009_330.png)

如果之前没有修改过Gradle 版本，可以通过下边方法查看，如图：

![image](http://static.open-open.com/lib/uploadImg/20170323/20170323150010_898.png)

我采用的 版本为：Gradle-3.4.1

## 查看 Gradle Plugin 版本

直接上图：

![image](http://static.open-open.com/lib/uploadImg/20170323/20170323150010_890.png)

当然还有一个更便捷的方式查看当前工程所使用 Gradle 版本与 Gradle Plugin 版本

打开目录：File -> Project Stucture -> Project , 如图：

![image](http://static.open-open.com/lib/uploadImg/20170323/20170323150010_653.png)

重点来了， Gradle 版本与 Gradle Plugin 版本之间的兼容性：

<table>
<tbody><tr><th>Plugin version</th><th>Required Gradle version</th></tr>
<tr><td>1.0.0 - 1.1.3</td><td>2.2.1 - 2.3</td></tr>
<tr><td>1.2.0 - 1.3.1</td><td>2.2.1 - 2.9</td></tr>
<tr><td>1.5.0</td><td>2.2.1 - 2.13</td></tr>
<tr><td>2.0.0 - 2.1.2</td><td>2.10 - 2.13</td></tr>
<tr><td>2.1.3 - 2.2.3</td><td>2.14.1+</td></tr>
<tr><td>2.3.0+</td><td>3.3+</td></tr>
</tbody></table>

参考：[https://developer.android.com/studio/releases/gradle-plugin.html#updating-gradle](https://developer.android.com/studio/releases/gradle-plugin.html#updating-gradle)

## gradle plugin对SDK Build Tools要求

gradle plugin都是基于某个版本的build tools开发的，一般会在发布时，声明依赖的版本；

[https://developer.android.com/studio/releases/gradle-plugin.html#revisions](https://developer.android.com/studio/releases/gradle-plugin.html#revisions)