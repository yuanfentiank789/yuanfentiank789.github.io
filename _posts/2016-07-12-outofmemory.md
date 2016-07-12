---
layout: post
title:  "深入理解Java虚拟机：OutOfMemory实战"
date:   2016-07-12 1:05:00
catalog:  true
tags:
    - OutOfMemory
   
---

>摘要
本文摘自《深入理解Java虚拟机》，模拟JVM中各内存区域的OOM溢出及配置参数。有删改。

# 概述

在Java虚拟机规范的描述中，除了程序计数器外，虚拟机内存的其他几个运行时区域都有发生OutOfMemoryError（下文称OOM）异常的可能，本节将通过若干实例来验证异常发生的场景。并且会初步介绍几个与内存相关的最基本的虚拟机参数。

本节内容的目的有两个：第一，通过代码验证Java虚拟机规范中描述的各个运行时区域存储的内容；第二，希望读者在工作中遇到实际的内存溢出异常时，能根据异常的信息快速判断是哪个区的内存溢出，知道什么样的代码可能导致这些区域内存溢出，以及出现这些异常后该如何处理。

下文代码的开头都注释了执行时所需要设置的虚拟机启动参数（注释中“VM args”后面跟着的参数），这些参数对实验的结果有直接影响，在调试代码的时候千万不要忽略。如果使用控制台命令来执行程序，那直接跟在Java命令后书写就可以。如果使用的是Android Studio，则可以参考下图页签中的设置：

![image](/images/oom/Snip20160712_9.png)

下文的代码都是基于Sun公司的**HotSpot**虚拟机运行的，对于不同公司的不同版本的虚拟机，参数和程序运行的结果可能会有所差别。版本如下：

    appledeMacBook-Pro:DisplayingBitmaps apple$ java -version
    java version "1.7.0_65"
    Java(TM) SE Runtime Environment (build 1.7.0_65-b17)
    Java HotSpot(TM) 64-Bit Server VM (build 24.65-b04, mixed mode)


## Java堆溢出 

Java堆用于存储对象实例，只要不断的创建对象，并且保证GC Roots到对象之间有可达路径来避免垃圾回收机制来清除这些对象，那么对象数量到达最大堆容量限制后就会产生内存溢出异常。

下面代码中，Java堆的大小限制为20M，不可扩展（将堆的最小值-Xms参数与最大值-Xmx最大值参数设置为一样，避免自动扩展）通过参数-XX：+HeapDumpOnOutOfMemoryError，可以让虚拟机在出现内存溢出时Dump出当前的内存转储快照以便事后进行分析。 代码中可通过Runtime.getRuntime().maxMemory()获取当前堆内存大小，来验证参数设置是否成功。

    /**
     * VM Args:-Xms20m -Xmx20m -XX:+HeapDumpOnOutOfMemoryError
     */
    public class HeapOOM {
        static class OOMObject {}

        public static void main(String[] args) {
            List<OOMObject> list = new ArrayList<OOMObject>();
            while (true) {
                list.add(new OOMObject());
            }
        }
    }
    

运行结果：

        20MB
    java.lang.OutOfMemoryError: Java heap space
    Dumping heap to java_pid90595.hprof ...
    Heap dump file created [27688820 bytes in 0.283 secs]
    Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at java.util.Arrays.copyOf(Arrays.java:2245)
	at java.util.Arrays.copyOf(Arrays.java:2219)
	at java.util.ArrayList.grow(ArrayList.java:242)
	at java.util.ArrayList.ensureExplicitCapacity(ArrayList.java:216)
	at java.util.ArrayList.ensureCapacityInternal(ArrayList.java:208)
	at java.util.ArrayList.add(ArrayList.java:440)
	at com.example.OutOfMemoryTest.main(OutOfMemoryTest.java:19)
	at sun.reflect.NativeMethodAccessorImpl.invoke0(Native Method)
	at sun.reflect.NativeMethodAccessorImpl.invoke(NativeMethodAccessorImpl.java:57)
	at sun.reflect.DelegatingMethodAccessorImpl.invoke(DelegatingMethodAccessorImpl.java:43)
	at java.lang.reflect.Method.invoke(Method.java:606)
	at com.intellij.rt.execution.application.AppMain.main(AppMain.java:144)

Java堆内存的OOM异常时实际应用中常见的内存溢出异常情况。当出现Java对内存溢出时，异常堆栈信息“java.lang.OutOfMemoryError”会跟着进一步提示“Java heap space”。

要解决这个区域的异常，一般的手段是先通过内存映像工具如（Eclipse MemoryAnalyzer）对Dump出来的堆转储快照进行分析，重点是确认内存中的对象是否是必要的，也就是要先分析到底是出现了内存泄露（Memory Leak）还是内存溢出（Memory Overflow）。该文件位于project根目录下。

如果是内存泄露，可进一步通过工具查看泄露对象到GC Roots的引用链，于是就能找到泄露对象是通过怎样的路径与GC Roots相关联并导致垃圾回收器无法自动回收它们的。掌握了泄露对象的类型信息及GC Roots引用链的信息，就可以比较容易确定发生泄露的代码位置。 

如果不存在内存泄露，换句话说，就是内存中的对象确实都还必须存活着，那就应当检查虚拟机堆参数（-Xmx与-Xms），与机器物理内存对比看是否还可以调大，从代码上检查是否存在某些对象生命周期过长、持有状态时间过长的情况，尝试减少程序运行期的内存消耗。 


## 虚拟机栈和本地方法栈溢出

由于在Hotspot虚拟机中并不区分虚拟机栈和本地方法栈，因此，对于Hotspot来说，虽然-Xoss参数（设置本地方法栈大小）存在，但实际上是无效的，栈容量只由-Xss参数设定，关于虚拟机栈和本地方法栈可以出现以下两种异常： 

<li>如果线程请求的栈深度大于虚拟机所允许的最大深度，将抛出StackOverflowError异常 </li>

<li>如果虚拟机在扩展时无法申请到足够的内存空间，则抛出OutOfMemoryError异常</li>

下面这个例子，将实验范围限制于单线程中的操作，尝试了下面两种方法均无法让虚拟机产生OutOfMemoryError异常，尝试的结果都是获得StackOverflowError异常：

  <li>使用-Xss 参数减少栈内存容量，结果：抛出StackOverflowError异常，异常出现时输出的栈的深度相应缩小</li> 

  <li>定义了大量的本地变量，增大此方法帧中本地变量表的长度。结果：抛出StackOverflowError异常时输出的堆栈深度相应减小。 </li>


测试代码如下：

    /**
     * VM Args: -Xss128K
     */
    public class JavaVMStackSOF {

        private int stackLength = 1;


        public void stackLeak() {
            stackLength++;
            stackLeak();
        }

        public static void main(String[] args) throws Throwable {
            JavaVMStackSOF oom = new JavaVMStackSOF();
            try {
                oom.stackLeak();
            } catch (Throwable e) {
                System.out.println("stack length:" + oom.stackLength);
                throw e;
            }
        }
    }
    
    
运行结果：

    stack length:751
    Exception in thread "main" java.lang.StackOverflowError
	at com.example.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:12)
	at com.example.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:13)
	at com.example.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:13)
	at com.example.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:13)
	at com.example.JavaVMStackSOF.stackLeak(JavaVMStackSOF.java:13)


经测试，在jdk7上最小的栈内存容量为160k，不能指定更低的容量了,可以看到方法调用的最大深度为751.

实验结果表明：在单个线程下，无论是由于栈帧太大还是虚拟机容量太小，当内存无法分配的时候虚拟机都抛出的是StackOverflowError。 

如果测试不限于单线程，通过不断的建立线程的方式倒是可以产生内除溢出异常，但是这样产生的内存溢出与栈空间是否够大不存在任何联系，或者说，为每个线程的栈分配的内存越大，反而越容易产生内存溢出异常。

原因是，操作系统分配给每个线程的内存是有限的，譬如32位Windows限制为2GB。虚拟机提供了参数来控制Java堆和方法区的这两部分内存的最大值。剩余的内存为2G（操作系统内存）减去Xmx（堆最大容量），再减去MaxPermSize（最大方法区容量），程序计数器消耗的内存很小，可以忽略。如果虚拟机进程本身耗费的内存不计算在内，剩下的内存就由虚拟机栈和本地方法栈“瓜分”了。每个线程分配到栈容量越大，可以建立的线程数自然越少，建立线程时越容易把剩余的内存耗尽。

这一点需要在开发多线程的应用时特别注意，出现StackOverflowError异常时有错误堆栈可以阅读，相对来说，比较容易找到问题的所在。而且，如果使用虚拟机默认参数，栈深度在大多数情况下（因为每个方法压入栈的帧大小并不是一样的，所以只能说在大多数情况下）达到1000~2000完全没有问题，对于正常的方法调用（包括递归），这个深度应该完全够用了。但是，如果是建立过多线程导致的内存溢出，在不能减少线程数或者更换64位虚拟机的情况下，就只能通过减少最大堆和减少栈容量来换取更多的线程了。如果没有这方面的处理经验，这种通过“减少内存”的手段来解决内存溢出的方式会比较难以想到。

以下代码通过创建多线程导致内存溢出异常：

    /**
     * VM Args: -Xss2M(这时候不妨设置大些)
     */
        public class JavaVMStackOOM {

        private void dontStop() {
            while (true) {
            }
        }

       public void stackLeakByThread() {
         while (true) {
            Thread thread = new Thread(new Runnable() {
                public void run() {
                    dontStop();
                }
            });

            thread.start();
        }
      }

        public static void main(String[] args) {
          JavaVMStackOOM oom = new JavaVMStackOOM();
          oom.stackLeakByThread();
      }
    }
    
注意：特别提示一下，如果要尝试运行上面这段代码，记得要先保存当前的工作。由于在Windows平台的虚拟机中，Java的线程时映射到操作系统的内核线程上的，因此上述代码执行时有较大风险，可能会导致操作系统假死。

运行结果：
    