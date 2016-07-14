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

![image](http://static.oschina.net/uploads/space/2016/0428/093236_urpG_1434710.png)


## 方法区和运行时常量池溢出

由于运行时常量池是方法去的一部分，因此这两个区域的溢出测试可以放在一起进行。

String.intern()方法是一个native方法，它的作用是：如果字符串常量池中已经包含一个等于此String对象的字符串，则返回代表池中这个字符串的string对象；否则，将此String对象包含的字符串添加到常量池中，并返回此String对象的引用。 在JDK1.6及之前的版本中，由于常量池分配在永久代中，我们可以通过-XX:PermSize和-XX:MaxPermSize限制方法区大小，从而间接限制其中常量池的容量，代码如下：

    /**
     * VM Args: -XX:PermSize=10M -XX:MaxPermSize=10M
     */
    public class RuntimeConstantPoolOOM{
      public static void main(String[] args){
        // 使用List保持着常量池引用，避免Full GC 回收常量池行为
        List<String> list = new ArrayList<String>();
        //10MB的PermSize在int范围内足够产生OOM了
        int i = 0;
        while(true) {
            //调用intern方法，将字符串全部放在常量池中
            list.add(String.valueOf(i++).intern());
        }
      }
    }
    
运行结果（JDK1.6 HotSpot JVM）：

 ![image](http://static.oschina.net/uploads/space/2016/0427/193026_UmRn_1434710.png)
 
 从运行结果中可以看到，运行时常量池溢出，在OutOfMemoryError后面跟随的提示信息是“PermGen space”，说明运行时常量池属于方法区（HotSpot虚拟机中的永久代）的一部分。
 
 “而使用JDK 1.7运行这段程序就不会得到相同的结果，while循环将一直进行下去。关于这个字符串常量池的实现问题，还可以引申出一个更有意思的影响，如代码清单2-7所示。”

    public class RuntimeConstantPoolOOM {
    public static void main(String[] args) {
        String str1 = new StringBuilder("计算机").append("软件").toString();
        System.out.println(str1 == str1.intern());
        String str2 = new StringBuilder("ja").append("va").toString();
        System.out.println(str2 == str2.intern());
        String str3 = new StringBuilder("ja").append("va1").toString();
        System.out.println(str3 == str3.intern());

      }
    }

这段代码在JDK 1.6中运行，会得到两个false，而在JDK 1.7中运行，会得到一个true和一个false。产生差异的原因是：在JDK 1.6中，intern（）方法会把首次遇到的字符串实例复制到永久代中，返回的也是永久代中这个字符串实例的引用，而由StringBuilder创建的字符串实例在Java堆上，所以必然不是同一个引用，将返回false。而JDK 1.7（以及部分其他虚拟机，例如JRockit）的intern（）实现不会再复制实例，只是在常量池中记录首次出现的实例引用，因此intern（）返回的引用和由StringBuilder创建的那个字符串实例是同一个。对str2比较返回false是因为“java”这个字符串在执行StringBuilder.toString（）之前已经出现过（java这个字符串比较特殊？），字符串常量池中已经有它的引用了，不符合“首次出现”的原则，而“计算机软件”这个字符串则是首次出现的，因此返回true。

jdk1.7输出结果：

    true
    false
    true



方法区用于存放Class相关信息，如类名、访问修饰符、常量池、字段描述、方法描述等。对于这些区域的测试，基本的思路是运行时产生大量的类去填满方法区，直到溢出。这里借助CGLib直接操作字节码运行时生成了大量的动态类。

值得注意的是，我们在这个例子中模拟的场景并非纯粹是一个实验，这样的应用经常会出现在实际应用中：当前很多主流框架如Spring、Hibernate，在对类进行增强时，都会使用到CGLib这类字节码技术，增强的类越多，就需要越大的方法区来保证动态生成的Class可以载入内存。另外，JVM上的动态语言（例如Groovy等）通常都会持续创建类来实现语言的动态性，随着这类语言越来越流行，也越来越容易遇到以下代码类似的溢出场景：

    import java.lang.reflect.Method;
    import com.jvm.oom.HeapOOM.OOMObject;
    import net.sf.cglib.proxy.Enhancer;
    import net.sf.cglib.proxy.MethodInterceptor;
    import net.sf.cglib.proxy.MethodProxy;

    /**
     * VM Args: -XX:PermSize=10M -XX:MaxPermSize=10M
     */
    public class JavaMethodAreaOOM{

    public static void main(String[] args) {
        while(true){
            Enhancer e = new Enhancer();
            e.setSuperclass(OOMObject.class);
            e.setUseCache(false);
            e.setCallback(new MethodInterceptor(){
                public Object intercept(Object obj, Method method, 
                        Object[] args, MethodProxy proxy) throws Throwable{
                    return proxy.invokeSuper(obj,args);
                }
            });
            e.create();
        }
        }
    }
    
运行结果：

    Caused by: java.lang.OutOfMemoryError: PermGen space
    
方法区的溢出是一种常见的内存溢出异常，一个类要被垃圾收集器回收掉，判断条件是比较苛刻的。在经常动态生成大量Class应用中，需要特别注意类的回收情况。这类除了上面提到的程序使用了CGLib字节码增强和动态语言之外，常见的还有：还有大量jsp或动态产生jsp文件的应用（JSP第一次运行时需要编译为Java类）、基于OSGi的应用（即使是同一个类文件，被不同的加载器加载也会视为不同的类）等。

## 本机直接内存溢出 

DirectMemory容量可通过-XX：MaxDirectMemorySize指定，如果不指定，则默认与Java堆最大值（-Xmx指定）一样，下面的代码越过了DirectByteBuffer类，直接通过反射获取Unsafe实例进行内存分配（Unsafe类的getUnsafe()方法限制了只有引导类加载器才会返回示例，也就是设计者希望只有rt.jar中的类才能使用Unsafe的功能）。因为虽然使用DirectByteBuffer分配内存也会抛出内存溢出异常，但它抛出异常时并没有真正向操作系统申请分配内存，而是通过计算得知无法分配，于是手动抛出异常，真正申请分配内存的方法是unsafe.allocateMemory()。

    import java.lang.reflect.Field;
    import sun.misc.Unsafe;

    /**
     * VM Args: -Xmx20M -XX:MaxDirectMemorySize=10M
     */
    public class DirectMemoryOOM{
      private static final int _1MB = 1024*1024;
    
      public static void main(String[] args) throws Exception {
        Field unsafeField = Unsafe.class.getDeclaredFields()[0];
        unsafeField.setAccessible(true);
        Unsafe unsafe = (Unsafe)unsafeField.get(null);
        while(true){
            unsafe.allocateMemory(_1MB);
        }
      }
    }
    
运行结果：

    ![image](http://static.oschina.net/uploads/space/2016/0428/092714_85sz_1434710.png)
    
由DirectMemory导致的内存溢出，一个明显的特征是在Heap Dump文件中不会看见明显的异常，如果发现OOM之后文件很小，而程序中有直接或简介使用了NIO，那就可以考虑一下是不是这方面的原因。