---

layout: post
title:  "漫谈AOP开发初探之AspectJ的用法"
date:   2017-08-30 1:05:00
catalog:  true
tags:

   - AOP
   - AspectJ
   
   
       
   
---

## 一、为什么需要AOP技术

AOP 是一个很成熟的技术。
假如项目中有方法A、方法B、方法C……等多个方法，
如果项目需要为方法A、方法B、方法C……这批方法增加具有通用性质的横切处理。
 
下图可以形象的说明具有通用性质的横切处理的思想：

![](http://img1.ph.126.net/KYJQyy-_cxnnvvpbgZoXgw==/2865978212885316121.png?_=5802662)

**在以前传统的做法是**
 

1. 先定义一个Advice方法，该方法实现这个通用性质的横切处理。
2. 打开方法A、方法B、方法C……的源代码修改，使得方法A、方法B、方法C……去调用Advice方法。

客户电话： 为每个方法都增加日志。

客户电话： 为每个方法前都增加权限控制。

客户电话： 为每个方法都加……

….

如果使用AOP，可以做到程序员无需修改方法A、方法B、方法C……，但又可以为方法A、方法B、方法C增加调用Advice方法。

**面向切面编程（AOP）是作为面向对象编程（OOP）的补充**:

AOP框架具有如下两个特征：

1. 各步骤之间的良好隔离性。
2. 源代码无关性。

## 二、AOP的功能

保证程序员不修改方法A、方法B、方法C……的前提下，可以为方法A、方法B、方法C……增加通用处理。

AOP的本质：依然要去【修改】方法A、方法B、方法C……

—— 只是这个修改由AOP框架完成，程序员不需要改。

AOP要求去修改，到底怎么去修改方法A、方法B、方法……

**AOP的实现方式有两种**：

1. AOP框架在编译阶段，就对目标类进行修改，得到的class文件已经是被修改过的。生成静态的AOP代理类（生成*.class文件已经被改掉了，需要使用特定的编译器）。以AspectJ为代表 —— 静态AOP框架。
2. AOP框架在运行阶段，动态生成AOP代理（在内存中动态地生成AOP代理类），以实现对目标对象的增强。它不需要特殊的编译器。以Spring AOP为代表。—— 动态AOP框架。
 
上面两种，哪种性能更好？很明显静态的AOP框架更好。
下面我们进入AspectJ的学习.


## 三. AspectJ简介
AOP虽然是方法论，但就好像OOP中的Java一样，一些先行者也开发了一套语言来支持AOP。目前用得比较火的就是AspectJ了，它是一种几乎和Java完全一样的语言，而且完全兼容Java（AspectJ应该就是一种扩展Java，但它不是像Groovy[1]那样的拓展。）。当然，除了使用AspectJ特殊的语言外，AspectJ还支持原生的Java，只要加上对应的AspectJ注解就好。所以，使用AspectJ有两种方法：

1. 完全使用AspectJ的语言。这语言一点也不难，和Java几乎一样，也能在AspectJ中调用Java的任何类库。AspectJ只是多了一些关键词罢了。
2. 使用纯Java语言开发，然后使用AspectJ注解，简称@AspectJ。

### 3.1 Aspect (切面)

实现了cross-cutting功能，是针对切面的模块。最常见的是logging模块、方法执行耗时模块，这样，程序按功能被分为好几层，如果按传统的继承的话，商业模型继承日志模块的话需要插入修改的地方太多，而通过创建一个切面就可以使用AOP来实现相同的功能了，我们可以针对不同的需求做出不同的切面。

### 3.2 jointpoint（连接点)
Join Points（以后简称JPoints）是AspectJ中最关键的一个概念。什么是JPoints呢？JPoints就是程序运行时的一些执行点。那么，一个程序中，哪些执行点是JPoints呢？比如：

- 一个函数的调用可以是一个JPoint。比如Log.e()这个函数。e的执行可以是一个JPoint，而调用e的函数也可以认为是一个JPoint。
- 设置一个变量，或者读取一个变量，也可以是一个JPoint。比如Demo类中有一个debug的boolean变量。设置它的地方或者读取它的地方都可以看做是JPoints。
- for循环可以看做是JPoint。

理论上说，一个程序中很多地方都可以被看做是JPoint，但是AspectJ中，只有如表1所示的几种执行点被认为是JPoints：
![](http://upload-images.jianshu.io/upload_images/1000896-7e934c5b467582d3?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3.3 Pointcuts

怎么从一堆一堆的JPoints中选择自己想要的JPoints呢？恩，这就是Pointcuts的功能。一句话，Pointcuts的目标是提供一种方法使得开发者能够选择自己感兴趣的JoinPoints

### 3.4 advice（处理逻辑）

advice是我们切面功能的实现，它是切点的真正执行的地方。比如像写日志到一个文件中，advice（包括：before、after、around等）在jointpoint处插入代码到应用程序中。我们来看一看原AspectJ程序和反编译过后的程序。看完下面的图我们就大概明白了AspectJ是如何达到监控源程序的信息了。

<table border="1" cellspacing="0" cellpadding="0"> <tbody><tr>  <td style="background:#8064A2;"><p><strong>关键词</strong></p></td>  <td style="background:#8064A2;"><p align="center"><strong>说明</strong></p>  <p><strong>&nbsp;</strong></p></td>  <td style="background:#8064A2;"><p align="center"><strong>示例</strong></p></td> </tr> <tr>  <td style="background:#8064A2;"><p><strong>before()</strong></p></td>  <td style="background:#BFB1D0;"><p>before advice</p></td>  <td style="background:#BFB1D0;"><p>表示在JPoint执行之前，需要干的事情</p></td> </tr> <tr>  <td style="background:#8064A2;"><p><strong>after()</strong></p></td>  <td style="background:#DFD8E8;"><p>after advice</p></td>  <td style="background:#DFD8E8;"><p>表示JPoint自己执行完了后，需要干的事情。</p></td> </tr> <tr>  <td style="background:#8064A2;"><p><strong>after():returning</strong>(返回值类型)<strong></strong></p>  <p><strong>after():throwing</strong>(异常类型)<strong></strong></p></td>  <td style="background:#BFB1D0;"><p>returning和throwing后面都可以指定具体的类型，如果不指定的话则匹配的时候不限定类型</p></td>  <td style="background:#BFB1D0;"><p>假设JPoint是一个函数调用的话，那么函数调用执行完有两种方式退出，一个是正常的return，另外一个是抛异常。</p>  <p>注意，after()默认包括returning和throwing两种情况</p></td> </tr> <tr>  <td style="background:#8064A2;"><p><strong>返回值类型 around()</strong></p></td>  <td style="background:#DFD8E8;"><p>before和around是指JPoint执行前或执行后备触发，而around就替代了原JPoint</p></td>  <td style="background:#DFD8E8;"><p>around是替代了原JPoint，如果要执行原JPoint的话，需要调用proceed</p></td> </tr></tbody></table>

### 3.5 实例说明

```
@Before("execution(* android.app.Activity.on**(..))")
    public void onActivityMethodBefore(JoinPoint joinPoint) throws Throwable {
}
```
- @Before 这是一个advice
- execution 这是一个Join Point
- (* android.app.Activity.on**(..)" 这是一个正则表达式
   - 第一个*表示返回值(任意类型) - 方法的路径(通过正则匹配) - ()表示方法的参数，可以指定类型
- onActivityMethodBefore 表示切入点的方法

本例中的pointcuts是匿名定义，无法复用，也可以按如下格式定义：

```
@Pointcut("execution(* android.app.Activity.on**(..))"
public void myPointCut(){};

@Before("myPointCut()")
    public void onActivityMethodBefore(JoinPoint joinPoint) throws Throwable {
}
```



## 四、实战AspectJ

### 4.1 java中使用Aspectj

AspectJ是一个基于Java语言的AOP框架，提供了强大的AOP功能，其他很多AOP框架都借鉴或采纳其中的一些思想。

1. 下载和安装AspectJ
    
    下载地址：[http://www.eclipse.org/aspectj/downloads.php](http://www.eclipse.org/aspectj/downloads.php)
    - 运行、下载得到的安装JAR包。
      ```
      java -jar /Users/guangjie.peng/development/aspectj-1.8.10.jar
      ```
      接下来，会自动检测JAVA_HOME,安装目录需要手动选择，一路Next；
 - 添加~/development/aspectj/bin到环境变量，配置完毕后输入terminal输入ajc，验证是否配置成功，该命令位于bin目录下。
 - 添加lib下的两个jar到classpath：
   ![](media/15040666572272.jpg)
mac上添加到classpath的方法是：把jar包复制到`/Library/Java/Extensions`目录下。
    
    
2. 使用AspectJ
   
- step 1 编写如下java源文件：
   
```
   package com.example.aspectj;

/**
 * Created by guangjie.peng on 17/8/30.
 */

public class AspectTest {
    public static void main (String[] args){
       System.out.println("execute main methid!!");
    }
}
   
```

  -  step 2 编写aspect如下：

```   
public aspect AuthAspectj {
        // execution(* com.example.aspectj.*.*(..)执行 任意返回值 该包下的任意类的任意方法形参不限
        before():execution(* com.example.aspectj.*.*(..)){
// 对原来方法进行修改、增强。
        System.out.println("----------模拟执行权限检查----------");
        }
}
   
```

- step 3 编译：

```
cd com/example/aspectj/
 ajc -d *.java
 java com.example.aspectj.AspectTest
```

得到输出如下： 
 
```
 ----------模拟执行权限检查----------
execute main method!!

``` 

### 4.2 android studio 配置AspectJ 环境

#### 4.2.1 新建Android project，并创建一个Android module：aspectj，在aspectj的build.gradle中配置如下代码:

```
import org.aspectj.bridge.IMessage
 import org.aspectj.bridge.MessageHandler
 import org.aspectj.tools.ajc.Main
 apply plugin: 'com.android.library'

 buildscript {

     repositories {
         jcenter()
     }
     dependencies {
         classpath 'com.android.tools.build:gradle:2.3.3'
         classpath 'org.aspectj:aspectjtools:1.8.9'
         classpath 'org.aspectj:aspectjweaver:1.8.9'
     }
 }



 android {
     compileSdkVersion 26
     buildToolsVersion "25.0.3"
     lintOptions {
         abortOnError false
     }
 }

 dependencies {
     compile 'org.aspectj:aspectjrt:1.8.9'
 }

 android.libraryVariants.all { variant ->
 //    LibraryPlugin plugin = project.plugins.getPlugin(LibraryPlugin)
     JavaCompile javaCompile = variant.javaCompile
     javaCompile.doLast {
         String[] args = ["-showWeaveInfo",
                          "-1.5",
                          "-inpath", javaCompile.destinationDir.toString(),
                          "-aspectpath", javaCompile.classpath.asPath,
                          "-d", javaCompile.destinationDir.toString(),
                          "-classpath", javaCompile.classpath.asPath,
                          "-bootclasspath", project.android.bootClasspath.join(
                 File.pathSeparator)]
         MessageHandler handler = new MessageHandler(true);
         new Main().run(args, handler)

         def log = project.logger
         for (IMessage message : handler.getMessages(null, true)) {
             switch (message.getKind()) {
                 case IMessage.ABORT:
                 case IMessage.ERROR:
                 case IMessage.FAIL:
                     log.error message.message, message.thrown
                     break;
                 case IMessage.WARNING:
                 case IMessage.INFO:
                     log.info message.message, message.thrown
                     break;
                 case IMessage.DEBUG:
                     log.debug message.message, message.thrown
                     break;
             }
         }
     }
 }


```
> 网上配置方法会报一个错误 No such property: project for class: com.android.build.gradle.LibraryPlugin.
> 
> 修复办法：不使用 LibraryPlugin 直接使用 project

#### 4.2.2 修改住module的build.gradle

- 依赖aspectj module:`dependencies{ compile project("aspectl")}`;
- 并添加如下代码：

```
 import org.aspectj.bridge.IMessage
import org.aspectj.bridge.MessageHandler
import org.aspectj.tools.ajc.Main

buildscript {
    repositories {
        mavenCentral()
    }
    dependencies {
        classpath 'org.aspectj:aspectjtools:1.8.9'
    }
}

final def log = project.logger
final def variants = project.android.applicationVariants

variants.all { variant ->
    if (!variant.buildType.isDebuggable()) {
        log.debug("Skipping non-debuggable build type '${variant.buildType.name}'.")
        return;
    }

    JavaCompile javaCompile = variant.javaCompile
    javaCompile.doLast {
        String[] args = ["-showWeaveInfo",
                         "-1.5",
                         "-inpath", javaCompile.destinationDir.toString(),
                         "-aspectpath", javaCompile.classpath.asPath,
                         "-d", javaCompile.destinationDir.toString(),
                         "-classpath", javaCompile.classpath.asPath,
                         "-bootclasspath", project.android.bootClasspath.join(File.pathSeparator)]
        log.debug "ajc args: " + Arrays.toString(args)

        MessageHandler handler = new MessageHandler(true);
        new Main().run(args, handler);
        for (IMessage message : handler.getMessages(null, true)) {
            switch (message.getKind()) {
                case IMessage.ABORT:
                case IMessage.ERROR:
                case IMessage.FAIL:
                    log.error message.message, message.thrown
                    break;
                case IMessage.WARNING:
                    log.warn message.message, message.thrown
                    break;
                case IMessage.INFO:
                    log.info message.message, message.thrown
                    break;
                case IMessage.DEBUG:
                    log.debug message.message, message.thrown
                    break;
            }
        }
    }
}

```

#### 4.2.3 在model(aspectj)(main/java/xxx)中新建class -> ActivityLifeAspect

代码如下：


```
@Aspect
public class ActivityLifeAspect {

    private static final String TAG = ActivityLifeAspect.class.getSimpleName();

    @Before("execution (* com.github.zdongcoding.aspectjdemo.MainActivity.onCreate(..))")
    public void adviceOnCreate(JoinPoint joinPoint) {
        MethodSignature signature = (MethodSignature) joinPoint.getSignature();
        Toast.makeText((Context) joinPoint.getTarget(), "aspectj"+signature.toShortString(), Toast.LENGTH_SHORT).show();
        Log.e(TAG, ""+joinPoint.getTarget());
    }

}


```
运行，即可看到代码生效了。

#### 4.2.4 查看编译后源码

源码路径在 ： app/build/intermediates/classes/xxx/MainActivity.class
你会看到多了 如下代码：

```
JoinPoint var4 = Factory.makeJP(ajc$tjp_0, this, this, savedInstanceState);
     ActivityLifeAspect.aspectOf().adviceOnCreate(var4);
```
 
   
以上是一个aspectj的入门，更多语法可以参考官网教程。

本文参考：[http://www.cnblogs.com/lihuidu/p/5802662.html](http://www.cnblogs.com/lihuidu/p/5802662.html)

[http://www.jianshu.com/p/139c87e2a2b5](http://www.jianshu.com/p/139c87e2a2b5)


