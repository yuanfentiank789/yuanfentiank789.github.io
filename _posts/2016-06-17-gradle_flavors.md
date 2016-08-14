---
layout: post
title:  "flavors in gradle"
date:   2016-06-15 1:05:00
catalog:  true
tags:
    - gradle
    - flavor

---


# 如何编译出不同版本的apk

经常看到有人问：如何使同一app的不同版本有不同的host配置，icon，甚至包名等等，可见这种需求是大量存在的，比如漫画的单机包，代码逻辑一模一样，不同的只是资源文件，有一个简单的途径实现该功能，就是Product Flavors。今天我们学习下如何使用gradle的flavor编译出生产环境和开发环境不同flavor的apk：


## <font color='red'>gradle是什么?</font>

gradle是android studio用来编译打包apk的最新构建工具，可以在gradle官方网站查找到更多关于gradle的知识，但是最基本地，gradle是一个可以管理dependencies，自定义tasks，编译出不同版本的apk，自动发布等功能于一体的构建工具。

## flavors是什么

>A product flavor defines a customized version of the application build by the project. A single project can have different flavors which change the generated application.

gradle一个很有趣的功能是可以定义多个variants，或者叫做product flavors，这是一个很有用的功能，比如在以下场景:

（1）正在开发的app同一套代码有免费和收费两个版本；

（2） 多个app代码逻辑一样，但包名和资源文件不同。

（3） 其他需要打包多个版本的场景。（测试环境和生产环境使用不同的server）

 其中每个版本都叫一个flavor。
 
 那么，不同flavor都能自定义那些部分呢？目前结论是all，包括res文件和source code，以下都以开发环境（测试版）和生产环境（正式版）为例来讲解如何构建不同版本的flavor。
 
最终apk的种类数：buildtype*falvor。
 
## 如何定义flavor

 在build.gradle文件中，如下定义你想要的flavor：

    
    productFlavors {  
        ...
        devel {
            ...
        }

        prod {
            ...
        }
    }
    
    
   创建完后可以在build variant窗口切换不同的flavor，每个flavor都能自定义那些东西？接下来一一说明：
   
## flavor-实现多包名
如何在同一手机上同时安装开发版和正式版的app？你应该知道，同时只能安装一个同一包名的apk，如果安装时，手机上已经有该包名的apk，则新的apk会覆盖安装旧的apk。现在用flavor实现该功能，你只需要做一件事：为每个flavor定义一个applicationId,也就是该flavor的包名。


    
    android {  
    productFlavors {
        devel {
            applicationId "zuul.com.android.devel"
        }

        prod {
            applicationId "zuul.com.android"
        }
    }
    }
    
    
## flavor-实现不同变量配置

有时候不同flavor可能有不同的参数配置，在代码里动态获取，一样可以通过flavor实现，如下，为prod这个flavor配置了3个变量，格式为：buildConfigField 类型, 变量名, 变量值。

    
    prod {  
    applicationId "zuul.com.android"
    buildConfigField 'String', 'HOST', '"http://api.zuul.com"'
    buildConfigField 'String', 'FLAVOR', '"prod"'
    buildConfigField "boolean", "REPORT_CRASHES", "true"
        }
    

那么该如何获取这些属性呢？如下，通过BuildConfig类即可获取到配置的这些变量值。

    
    BuildConfig.HOST  
    BuildConfig.FLAVOR  
    BuildConfig.REPORT_CRASHES    
    


## flavor——实现不同res

先看下需求，现在需要打包测试和开发两个环境的apk，这两个apk唯一不同的地方是，访问的后台服务器不同，传统的实现方式是：在代码里定义两个同名变量，根据需要注释掉一个，如下：

    
    //线上环境
     public static final String root_url = "http://client.aaa.bbb.com";

    // 测试地址
    //    public static final String root_url = "http://221.179.193.164/xxx/api";
    
    
这样能满足需求，但有几个问题：

（1）需要经常手动修改源码，有时甚至会忘记修改；

（2）在同一device上同时只能安装一个flavor的app；

（3）最烦的一点，经常有人会问：小王，我手机上安装的是开发版还是正式版，鬼知道。

既然我们知道痛点在哪里了，我们该怎么办呢？简单起见，我们要实现两个目的；

（1）快速在开发环境和生产环境切换，而不用修改source code；

（2）轻松从视觉上能够区分开发版还是正式版。

我们可以轻松实现，按照如下步骤（经测试发现该步骤比较容易理解）：

（1）在project/app/build.gradle中的android部分增加dev和product两个flavor：

        
    android {
    compileSdkVersion 23
    buildToolsVersion "23.0.3"

    ...
    productFlavors {
        dev {
            applicationId "com.flavor.dev"

        }
        product {
            applicationId "com.flavor.product"
        }
    }
    ...
    }
    
   猜下会发生什么？给每个flavor定义了一个applicationId，也就是包名，意味着我们可以在同一device上安装两个不同flavor的apk，同时applicationId也只能在flavor和defaultConfig部分定义，flavor中的applicationId将覆盖defaultConfig中的id
（2）在project/app/src目录上右键---->new---->Android Resource File，
    ![image](/images/gradle/Snip20160617_9.png)
    
   看到如下界面：
   ![image](/images/gradle/Snip20160617_12.png)
   
   说明如下：
   
   1 处为资源文件名字，此处填写strings.xml,2处为资源类型，选择values，3处为source set，点击下拉列表可以看到，都是build type和flavor的组合，默认有debug和release两个build type，此处选择dev，同理创建product的strings.xml.结果如图：
   ![image](/images/gradle/Snip20160617_13.png)
   
   **注意** 图中可以看到有3个flavor，main，dev和product，main和product的res文件夹图标和dev的不一样，因为studio不会同时编译多个flavor，当前在build variant窗口选择了product，所以资源文件夹图标会改变。
   
 (3) 在每个flavor的strings.xml中定义如下string值：
 
 dev 
 
     
     <resources>
        <string name="root_url">"http://221.179.193.164/xxx/api</string>
    </resources>    
     
product

    
    <resources>
        <string name="root_url">http://client.aaa.bbb.com</string>
    </resources>
    




但是我们如何能快速知道当前安装版本是开发版还是正式版呢？一个简单的办法是：通过icon，也就是提供不同的launcher图标。然后在build variant窗口切换flavor即可打包不同flavor的apk，如图：

![image](/images/gradle/Snip20160617_14.png)

## flavor-不同source code

一个多flavor，每个flavor的source code不同的结构图如下：

     
     project/
        |
        |---src/
        |---flavorA2/
        |      |
        |      |---java/
        |      |     |---com.abc
        |      |                 |-----classA.java
        |      |                 |-----classB.java
        |      |---res/
        |      |---AndroidManifest.xml
        |
        |---main
        |      |---java/
        |      |     |---com.abc.flavorA
        |      |                 |-----classC.java
        |      |                 |-----classD.java
        |      |---res/
        |      |    |---drawable/
        |      |    |---layout/
        |      |    |---values/
        |      |         
        |      |---AndroidManifest.xml
        |
        |---falvorA
        |      |---java/
        |      |     |---com.abc
        |      |                 |-----classA.java
        |      |                 |-----classB.java
     
     

有如下几点需要说明：

（1）classA和classB不能定义在main flavor中，如果定义在main flavor中，该类就无法在其他flavor中有不同实现了；

（2）如果需要在main flavor中访问其他flavor中的类（main 中没有），则该类必须在其他所有flavor中定义，否则在切换到不含该类的flavor时，会找不到该类。

（3）引用关系：切换flavor时，同一时间，只有一个flavor可见，flavor（不含main）彼此不可见，flavor（不含main）中可以引用main flavor中的所有class，main中可以引用其他每个flavor都含有的class，但要保证切换flavor时，该类同样存在，否则报找不到类错误。

## 结论
由上而知，自定义flavor是很简单的，虽然只是一个简单的例子，但该方法可被用于更复杂的场景，不同flavor不仅可以有不同res文件，甚至不同source code。


参考：

  [http://inaka.net/blog/2014/12/22/create-separate-production-and-staging-builds-in-android/](http://inaka.net/blog/2014/12/22/create-separate-production-and-staging-builds-in-android/)
 
 [http://blog.brainattica.com/how-to-work-with-flavours-on-android/](http://blog.brainattica.com/how-to-work-with-flavours-on-android/)
 
 [http://stackoverflow.com/questions/23698863/build-flavors-for-different-version-of-same-class](http://stackoverflow.com/questions/23698863/build-flavors-for-different-version-of-same-class)
 
 [http://stackoverflow.com/questions/16737006/using-build-flavors-structuring-source-folders-and-build-gradle-correctly](http://stackoverflow.com/questions/16737006/using-build-flavors-structuring-source-folders-and-build-gradle-correctly)
 