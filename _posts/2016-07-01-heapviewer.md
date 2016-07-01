---
layout: post
title:  "HPROF Viewer and Analyzer"
date:   2016-06-29 1:05:00
catalog:  true
tags:
    - HPROF
    - Analyzer
    - Reference Tree
       

---

[翻译自https://developer.android.com/studio/profile/am-hprof.html](https://developer.android.com/studio/profile/am-hprof.html#why)

# HPROF Viewer and Analyzer


使用Android Monitor提供的Memory Monitor观察内存使用情况的同时，也可以把Java Heap导出为一个Android规格的HPROF格式文件的快照。HPROF Viewer可以显示java heap中每个class的实例，以及他们的reference tree，来帮助你跟踪内存使用情况和查找内存泄露，HPROF是一个J2SE中Heap的二进制格式。

## 为什么要观察Java Heap？

通过Java Heap可以知道：

<li>可以按类型显示对象申请的内存大小；</li>
<li>可以观察自动或手动触发的GC事件；</li>
<li>可以定位可能存在内存泄露的对象；</li>

但是，你需要一直盯着memory monitor中heap变化的图表。


HPROF Analyzer 可以发现以下潜在的问题：

所有已经被destroy但是仍然可以被GC root访问到的activity实例；

重复的string对象；

一个reference tree的顶级引用者，如果能把它移除，则他直接或间接引用的对象都可以被GC回收掉，这是一个释放内存的方法。

## 看懂HPROF Viewer

HPROF Viewer打开Hprof后如下图：

![image](https://developer.android.com/images/tools/am-hprofviewer.png)

可以显示如下信息：

<table>
  <tbody><tr>
    <th scope="col">Column</th>
    <th scope="col">Description</th>
  </tr>

  <tr>
    <td><strong>Class Name</strong></td>
    <td>The Java class responsible for the memory.</td>
  </tr>

  <tr>
    <td><strong>Total Count</strong></td>
    <td>Total number of instances outstanding.</td>
  </tr>
  <tr>
    <td><strong>Heap Count</strong></td>
    <td>Number of instances in the selected heap.</td>
  </tr>
  <tr>
    <td><strong>Sizeof</strong></td>
    <td>Size of the instances (currently, 0 if the size is variable).</td>
  </tr>
  <tr>
    <td><strong>Shallow Size</strong></td>
    <td>Total size of all instances in this heap.</td>
  </tr>
  <tr>
    <td><strong>Retained Size</strong></td>
    <td>Size of memory that all instances of this class is dominating.</td>
  </tr>
  <tr>
    <td><strong>Instance</strong></td>
    <td>A specific instance of the class.</td>
  </tr>
  <tr>
    <td><strong>Reference Tree</strong></td>
    <td>References that point to the selected instance, as well as references pointing to the
      references.</td>
  </tr>
  <tr>
    <td><strong>Depth</strong></td>
    <td>The shortest number of hops from any GC root to the selected instance.</td>
  </tr>
  <tr>
    <td><strong>Shallow Size</strong></td>
    <td>Size of this instance.</td>
  </tr>
  <tr>
    <td><strong>Dominating Size</strong></td>
    <td>Size of memory that this instance is dominating.</td>
  </tr>
</tbody></table>

打开右侧的 Analyzer Tasks窗口，有两个可以执行的task，查找内存泄露和查找重复string。

其中reference tree视图中，可以看到LeakActivity被多个对象引用，格式为：变量名 in 类名；

比如：this$0 in com.example.dunno.myapplication.LeakActivity$1，表示LeakActivity中有个$1类引用了LeakActivity，LeakActivity在该类中的名字是this$0；同时，LeakActivity中的$1类又被Message中的target和LeakActivity的myLeakHandler同时引用。

同级item之间是并列关系，表示引用了同一父级的类名部分的对象，也就是父级部分的类名部分对应子级的变量名，是同一个对象，一个表类型，一个表名称。把外层叫做父级，内层叫做子级，则子级引用了父级对象，每个父级可以有多个子级。

打开右侧的 Analyzer Tasks窗口，有两个可以执行的task，查找内存泄露和查找重复string。执行Detect Leaked Activities后，在Analysis Results窗口检测到泄露的Activity，有2个同类型的Activity对象，表示该对象泄露了，无法回收。


