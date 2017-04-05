---

layout: post
title:  " Xposed模块开发指南"
date:   2017-04-01 1:05:00
catalog:  true
tags:

   - xposed
   - development
   - 模块
  
   
---

# 开发指南

> 本篇翻译自xposed作者，原文链接：[https://github.com/rovo89/XposedBridge/wiki/Development-tutorial](https://github.com/rovo89/XposedBridge/wiki/Development-tutorial)

你想要学习如何创建一个Xposed的module吗?通过阅读本文就可以，不仅包括技术上“创建这个文件,并插入…”等等,同时让你知道背后的实现原理，知其然，也知其所以然。当然你也可以只看最后的源代码和阅读[“Making the project an Xposed module”](#make)这个章节。但你会得到一个更好的理解如果你通读整篇文章。

## 开发什么Module

本文带领大家重新创建Red Clock例子可以在[Github](https://github.com/rovo89/XposedExamples/tree/master/RedClock)上找到。它包括状态栏时钟的颜色更改为红色和添加一个笑脸表情。我选择这个例子,因为它是一个很小,但容易看到效果。此外,它使用了框架提供的一些基本的方法。


## How Xposed works？
动手coding之前你应该阅读下本章节，关于Xposed的工作原理，如果觉得很无聊也可以直接跳过。

Android系统里有一个叫“Zygote”的进程，它是Android运行时的核心，每个Android应用都通过它的副本的形式被fork出来。Zygote进程在系统启动时被/init.rc脚本启动，同时启动了用来加载必要的classes和执行初始化方法的/system/bin/app_process进程。

这就到了Xposed起作用的地方，当Xposed framework安装成功后，一个扩展的app_process被拷贝到/system/bin.这个扩展的app_process添加了一个额外的jar到classpath中，以在一定的时机调用jar里的方法。比如，在VM创建完成后，甚至Zygote的main方法被调用前。这样我们就可以在这个上下文执行我们想要的操作了。

这个jar位于/data/data/de.robv.android.xposed.installer/bin/XposedBridge.jar，它的源码在[这里](https://github.com/rovo89/XposedBridge)。打开[XposedBridge](https://github.com/rovo89/XposedBridge/blob/master/src/de/robv/android/xposed/XposedBridge.java)这个类，可以看到main方法，它在app_process启动时被调用。一些初始化和modules的加载也在这个时机被完成（稍后会将module的加载）。

### Method hooking/replacing

Xposed真正强大的地方在于对Method调用的hook，如果通过反编译来验证逻辑猜想的话，只能多次的反编译代码，修改代码，重新签名打包来完成。而使用Xposed的hook功能，不用修改apk方法中的代码，通过再目标method前后插入要执行的代码即可，method是java中最小的执行单元。

XposedBridge有一个private native方法：hookMethodNative，这个方法的实现在扩展后的app_process中，这意味着，每次被hook的方法被调用时，hookMethodNative将被调用，在这个方法里，XposedBridge中的handleHookedMethod被调用，通过this参数传递了被调用method的信息。然后这个方法回调注册过该method的hook方法，对参数进行修改，修改运行结果等，非常灵活。

理论到此为止，接下来开始创建一个xposed module

## 创建project

一个xposed module本质上是一个普通的Android app，只是多了几个meta data和文件而已。因此，首先创建一个普通的Android project，sdk版本选择4.0.3（API 15）以上，因为module没有界面，因此不必要创建activity，这时，应该已经创建好了一个空的Android project。


## <span id="make">Making the project an Xposed module</span>

接下来把上一步刚创建好的project改造成可以被Xposed加载的module，分为以下几个步骤：

### 添加Xposed Framework API到project

每个Xposed module需要引用这个API库，下边看下正确做法：

#### Android Studio（基于Gradle）

The Xposed Framework API is published on Bintray/jCenter: [https://bintray.com/rovo89/de.robv.android.xposed/api](https://bintray.com/rovo89/de.robv.android.xposed/api)，这样在app/build.gradle中引入非常方便：

    repositories {
        jcenter();
    }

    dependencies {
        provided 'de.robv.android.xposed:api:82'
    }
    
需要注意的是，这里是有provide而不是compile。如果使用compile，api库的classes将会被打包到apk中，会导致bug，特别是在Android 4.x设备上。使用provided只是让module可以通过编译，在apk中只有对api的引用，真正的实现在由运行设备的Xposed Framework提供。

  大部分情况下，repositories声明已经存在。这时只需要添加provided这行到reposi
  tories中就可以了。
  
  可以通过如下方式引入文档和源码：
  
      provided 'de.robv.android.xposed:api:82'
      provided 'de.robv.android.xposed:api:82:sources'
      
  另外，需要禁用AS的instant run，否则你自己的classes不是直接打包进apk，而是通过一个子应用，xposed不能处理这种case。
  
#### Eclipse

下载jar：[https://jcenter.bintray.com/de/robv/android/xposed/api/](https://jcenter.bintray.com/de/robv/android/xposed/api/)

建议jar放到一个新建目录lib，不要直接放到libs，因为放到libs中，jar包的classes文件将会被打包进apk。右键jar文件，Build Path -> Add to Build Path。

#### API versions

一般来说，API version等于构建它的Xposed的version，但是只有部分framework修改会导致API接口改变，只有在API接口改变时，我才会更新API version，因此API version小于等于对应的Xposed version，当你使用API version 82编译了一个module，它在Xposed version90也有可能正常工作。

我一直建议用户使用最新的Xposed可用version，以得到更好的兼容性。在AndroidManifest.xml里设置xposedminversion的值为你使用的API version。如果你依赖一个未修改API version的修改（bugfix），只要修改xposedminversion的值为最新的Xposed version就可以了。

### AndroidManifest.xml

Xposed Installer能列出系统已经安装的module，是因为它通过application中一个特殊的meta flag来识别。到AndroidManifest.xml => Application => Application Nodes (at the bottom) => Add => Meta Data.在Application节点底部添加几个meta data.

1. name:xposedmodule,value:true;
2. name:xposedminversion,value:上一节获得的API version；
3. name:xposeddescription,value:module简单的描述

添加完成后，xml如下：

    <?xml version="1.0" encoding="utf-8"?>
    <manifest xmlns:android="http://schemas.android.com/apk/res/android"
    package="de.robv.android.xposed.mods.tutorial"
    android:versionCode="1"
    android:versionName="1.0" >

    <uses-sdk android:minSdkVersion="15" />

    <application
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name" >
        <meta-data
            android:name="xposedmodule"
            android:value="true" />
        <meta-data
            android:name="xposeddescription"
            android:value="Easy example which makes the status bar clock red and adds a smiley" />
        <meta-data
            android:name="xposedminversion"
            android:value="53" />
    </application>
    </manifest>
    
### Module实现

现在可以为module创建一个class了，我创建的class名字是Tutorial，包名是：de.robv.android.xposed.mods.tutorial：

    package de.robv.android.xposed.mods.tutorial;

    public class Tutorial {

    }
    
作为第一步，我们只添加一些log，用来证明我们的module被加载了。module有几个hook入口，具体选择那个入口取决于你想修改的内容。你可以让Xposed调用你module中的函数，在系统启动时，或一个新的app被加载时，或一个app的资源被初始化时等待。

  所有的hook入口类必须是IXposedMod的实现类，在这里（app被加载），你只需要实现IXposedHookLoadPackage，它只有一个方法带有一个参数，这个参数带有上下文信息。这个例子里，我们记录下被加载app的名字:
  
      package de.robv.android.xposed.mods.tutorial;

      import de.robv.android.xposed.IXposedHookLoadPackage;
      import de.robv.android.xposed.XposedBridge;
      import de.robv.android.xposed.callbacks.XC_LoadPackage.LoadPackageParam;

      public class Tutorial implements IXposedHookLoadPackage {
          public void handleLoadPackage(final LoadPackageParam lpparam) throws Throwable {
              XposedBridge.log("Loaded app: " + lpparam.packageName);
          }
      }
      
  这个log方法会输出到logcat（tag是Xposed），同时输出到文件：/data/data/de.robv.android.xposed.installer/log/debug.log（可以通过Xposed Installer查看）。
  
### assets/xposed_init

最后要做的一件事是告诉XposedBridge那个class包含hook入口，通过一个叫xposed_init的文件。在asset目录下新建这个文件，该文件每行包含一个class的全限定名，在这里是：de.robv.android.xposed.mods.tutorial.Tutorial。

## 编译运行
保存上述修改，运行这个application，因为是首次安装这个module，需要首先打开Xposed Installer激活。在Modules tab找到刚安装的module，激活，重启设备.通过logcat观察输出，应该看到如下输出：

    Loading Xposed (for Zygote)...
    Loading modules from /data/app/de.robv.android.xposed.mods.tutorial-1.apk
    Loading class de.robv.android.xposed.mods.tutorial.Tutorial
    Loaded app: com.android.systemui
    Loaded app: com.android.settings
    ... (many more apps follow)
    
## 找到目标并修改它

接下来讲解不同需求差异化非常大的部分，一般来讲，首先需要知道目标的实现细节，本指南以status bar为例，了解下他的细节：

方法一：反编译，能获取到精确信息，但不易读，因为反编译后是smali格式。
方法二：从AOSP获取信息（不同rom实现不同，但本例应该改动不大），如果不够，再观察反编译代码。

可以用关键字“clock”查找，或者使用被用到的layout和其他资源，如果下载了AOSP代码，可以在frameworks/base/packages/SystemUI查找。你会发现只有少数几个地方使用“clock”，这是一种正确的方法。记住我们只能hook 方法，所以必须找到一个地方能插入一些代码，在目标方法执行前或后，或替换整个method，应该尽量寻找比较特殊的method，不要寻找那些被高频或多个地方调用的method，避免性能问题和非预料中的实现。

这个例子中，你会在res/layout/status_bar.xml中发现有一个自定义view 的引用，com.android.systemui.statusbar.policy.Clock。文本颜色通过textAppearance属性定义，所以最容易的修改方法是修改textAppearance的定义。但是，对Xposed来说修改styles是无法办到的。替换statusbar的layout倒是有可能，但修改有些大。观察这个class，有一个method叫updateClock，看起来像是每分钟被调用一次。

    final void updateClock() {
    mCalendar.setTimeInMillis(System.currentTimeMillis());
    setText(getSmallTime());
    }

看起来很完美，这个method功能很单一，只用来更新时间，如果在它后边修改字体颜色，应该可以实现。

## 使用反射寻找要hook的method

我们已经知道com.android.systemui.statusbar.policy.Clock有一个我们想要拦截的方法：updateClock。同时发现这个class在SystemUI的源码中，因此这个类只存在于SystemUI进程中。因此我们在handleLoadPackage中需要做一些条件判断：

    public void handleLoadPackage(LoadPackageParam lpparam) throws Throwable {
        if (!lpparam.packageName.equals("com.android.systemui"))
            return;

        XposedBridge.log("we are in SystemUI!");
    }

使用lpparam参数，可以检查是否在我们要拦截的进程中，检测通过，则使用该参数获得对应的classloader，然后hook目标方法：

    package de.robv.android.xposed.mods.tutorial;

    import static de.robv.android.xposed.XposedHelpers.findAndHookMethod;
    import de.robv.android.xposed.IXposedHookLoadPackage;
    import de.robv.android.xposed.XC_MethodHook;
    import de.robv.android.xposed.callbacks.XC_LoadPackage.LoadPackageParam;

    public class Tutorial implements IXposedHookLoadPackage {
    public void handleLoadPackage(final LoadPackageParam lpparam) throws Throwable {
        if (!lpparam.packageName.equals("com.android.systemui"))
            return;

        findAndHookMethod("com.android.systemui.statusbar.policy.Clock", lpparam.classLoader, "updateClock", new XC_MethodHook() {
            @Override
            protected void beforeHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called before the clock was updated by the original method
            }
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                // this will be called after the clock was updated by the original method
            }
    });
    }
    }
    
findAndHookMethod是一个工具方法，注意静态导入部分，它使用SystemUI的classloader寻找Clock类，然后寻找他的updateClock方法，如果这个方法有多个参数，还需要列出参数的class类型。最后一个参数，需要提供一个XC_MethodHook的实现类。如果是很小的修改，可以直接使用匿名内部类，如果改动很大，建议单独创建一个实现类。

XC_MethodHook有两个method，beforeHookedMethod和afterHookedMethod。根据名字很容易知道这两个method在被hook的method前后执行。你可以使用before方法重新计算结果，甚至阻断对hook方法的调用，直接返回你的结果。after方法用来根据被hook方法的返回值做一些操作，也可以修改返回值，当然，可以添加自己的代码。

> 如果想完全替换一个method，使用XC_MethodReplacement，只需要重写方法：replaceHookedMethod

XposedBridge维护了一个要hook方法的callback的list，优先级（在hookMethod定义）最高的先被回调，被hook方法优先级最低。所以，如果你对一个method注册了两个回调A（高优先级）和B（默认优先级），每次目标方法被执行，回调流程如下：A.before -> B.before -> original method -> B.after -> A.after。A可以修改B看到的参数，A也可以决定最终的结果。

## 最后一步：执行hook代码

我们已经hook了updateClock方法，现在开始修改写东西。

第一件事，校验，我们是否有一个Clock引用，通过param.thisObject获取，因此，如果目标方法通过myClock.updateClock()被调用，param.thisObject就是myClock.

第二步，我们能做什么？Clock这个类是不可用的，不能强转（想都不要想），但是他继承于TextView，因此可以使用它TextView的方法，setText,setTextColor等等。这个修改在目标方法被执行后调用，因此beforeHookedMethod留空。

    package de.robv.android.xposed.mods.tutorial;

    import static de.robv.android.xposed.XposedHelpers.findAndHookMethod;
    import android.graphics.Color;
    import android.widget.TextView;
    import de.robv.android.xposed.IXposedHookLoadPackage;
    import de.robv.android.xposed.XC_MethodHook;
    import de.robv.android.xposed.callbacks.XC_LoadPackage.LoadPackageParam;

    public class Tutorial implements IXposedHookLoadPackage {
    public void handleLoadPackage(final LoadPackageParam lpparam) throws Throwable {
        if (!lpparam.packageName.equals("com.android.systemui"))
            return;

        findAndHookMethod("com.android.systemui.statusbar.policy.Clock", lpparam.classLoader, "updateClock", new XC_MethodHook() {
            @Override
            protected void afterHookedMethod(MethodHookParam param) throws Throwable {
                TextView tv = (TextView) param.thisObject;
                String text = tv.getText().toString();
                tv.setText(text + " :)");
                tv.setTextColor(Color.RED);
            }
        });
    }
    }
    
## 欢呼吧

再次安装module，可以看到状态栏的时间变成骚红色了。