---

layout: post
title:  "Android Studio之maven Central，JCenter"
date:   2017-08-30 1:06:00
catalog:  true
tags:

   - maven
   - jcenter

   
   
       
   
---

## Android studio 是从哪里得到库的？

Android Studio是从build.gradle里面定义的Maven 仓库服务器上下载library的。Apache Maven是Apache开发的一个工具，提供了用于贡献library的文件服务器。总的来说，只有两个标准的Android library文件服务器：jcenter 和 Maven Central。

- jcenter 
jcenter是一个由 bintray.com维护的Maven仓库 。你可以在[这里](http://jcenter.bintray.com/)看到整个仓库的内容。

我们在项目的build.gradle 文件中如下定义仓库，就能使用jcenter了：

```
allprojects {
    repositories {
        jcenter()
    }
}
```

- Maven Central 
Maven Central 则是由sonatype.org维护的Maven仓库。你可以在这里看到整个仓库。

注：不管是jcenter还是Maven Central ，两者都是Maven仓库

我们在项目的build.gradle 文件中如下定义仓库，就能使用Maven Central了：

```
allprojects {
    repositories {
        mavenCentral()
    }
}
```

注意，虽然jcenter和Maven Central 都是标准的 android library仓库，但是它们维护在完全不同的服务器上，由不同的人提供内容，两者之间毫无关系。在jcenter上有的可能 Maven Central 上没有，反之亦然。

除了两个标准的服务器之外，如果我们使用的library的作者是把该library放在自己的服务器上，我们还可以自己定义特有的Maven仓库服务器。Twitter的Fabric.io 就是这种情况，它们在https://maven.fabric.io/public 上维护了一个自己的Maven仓库。如果你想使用Fabric.io的library，你必须自己如下定义仓库的url。

```
repositories {
    maven { url 'https://maven.fabric.io/public' }
}
```
然后在里面使用相同的方法获取一个library。

```
dependencies {
    compile 'com.crashlytics.sdk.android:crashlytics:2.2.4@aar'
}
```
但是将library上传到标准的服务器与自建服务器，哪种方法更好呢？当然是前者。如果将我们的library公开，其他开发者除了一行定义依赖名的代码之外不需要定义任何东西。因此这篇文章中，我们将只关注对开发者更友好的jcenter 和 Maven Central 。

实际上可以在Android Studio上使用的除了Maven 仓库之外还有另外一种仓库：Ivy 仓库

## 理解jcenter和Maven Central
为何有两个标准的仓库？

事实上两个仓库都具有相同的使命：提供Java或者Android library服务。上传到哪个（或者都上传）取决于开发者。

起初，Android Studio 选择Maven Central作为默认仓库。如果你使用老版本的Android Studio创建一个新项目，mavenCentral()会自动的定义在build.gradle中。

但是Maven Central的最大问题是对开发者不够友好。上传library异常困难。上传上去的开发者都是某种程度的极客。同时还因为诸如安全方面的其他原因，Android Studio团队决定把默认的仓库替换成jcenter。正如你看到的，一旦使用最新版本的Android Studio创建一个项目，jcenter()自动被定义，而不是mavenCentral()。

有许多将Maven Central替换成jcenter的理由，下面是几个主要的原因。

- jcenter通过CDN发送library，开发者可以享受到更快的下载体验。
- jcenter是全世界最大的Java仓库，因此在Maven Central 上有的，在jcenter上也极有可能有。换句话说jcenter是Maven Central的超集。
- 上传library到仓库很简单，不需要像在 Maven Central上做很多复杂的事情
- 友好的用户界面

## gradle是如何从仓库上获取一个library的？

我们在 build.gradle输入如下代码的时候，这些库是如果奇迹般下载到我们的项目中的。
`compile 'com.inthecheesefactory.thecheeselibrary:fb-like:0.9.3'
`
一般来说，我们需要知道library的字符串形式，包含3部分:
`GROUP_ID:ARTIFACT_ID:VERSION`

上面的例子中，GROUP_ID是com.inthecheesefactory.thecheeselibrary ，ARTIFACT_ID是fb-like，VERSION是0.9.3。

GROUP_ID定义了library的group。有可能在同样的上下文中存在多个不同功能的library。如果library具有相同的group，那么它们将共享一个GROUP_ID。通常我们以开发者包名紧跟着library的group名称来命名，比如com.squareup.picasso。然后ARTIFACT_ID中是library的真实名称。至于VERSION，就是版本号而已，虽然可以是任意文字，但是我建议设置为x.y.z的形式，如果喜欢还可以加上beta这样的后缀。

下面是Square library的一个例子。你可以看到每个都可以很容易的分辨出library和开发者的名称。

```
dependencies {
  compile 'com.squareup:otto:1.3.7'
  compile 'com.squareup.picasso:picasso:2.5.2'
  compile 'com.squareup.okhttp:okhttp:2.4.0'
  compile 'com.squareup.retrofit:retrofit:1.9.0'
}
```
那么在添加了上面的依赖之后会发生什么呢？简单。Gradle会询问Maven仓库服务器这个library是否存在，如果是，gradle会获得请求library的路径，一般这个路径都是这样的形式：GROUP_ID/ARTIFACT_ID/VERSION_ID。比如可以在http://jcenter.bintray.com/com/squareup/otto/1.3.7 和 https://oss.sonatype.org/content/repositories/releases/com/squareup/otto/1.3.7/

下获得com.squareup:otto:1.3.7的library文件。

然后Android Studio 将下载这些文件到我们的电脑上，与我们的项目一起编译。整个过程就是这么简单，一点都不复杂。

我相信你应该清楚的知道从仓库上下载的library只是存储在仓库服务器上的jar 或者aar文件而已。有点类似于自己去下载这些文件，拷贝然后和项目一起编译。但是使用gradle依赖管理的最大好处是你除了添加几行文字之外啥也不做。library一下子就可以在项目中使用了。

## 了解aar文件
我刚才说了仓库中存储的有两种类型的library：jar 和 aar。jar文件大家都知道，但是什么是aar文件呢？

aar文件时在jar文件之上开发的。之所以有它是因为有些Android Library需要植入一些安卓特有的文件，比如AndroidManifest.xml，资源文件，Assets或者JNI。这些都不是jar文件的标准。

因此aar文件就时发明出来包含所有这些东西的。总的来说它和jar一样只是普通的zip文件，不过具有不同的文件结构。jar文件以classes.jar的名字被嵌入到aar文件中。其余的文件罗列如下：

```
/AndroidManifest.xml (mandatory)
/classes.jar (mandatory)
/res/ (mandatory)
/R.txt (mandatory)
/assets/ (optional)
/libs/*.jar (optional)
/jni//*.so (optional)
/proguard.txt (optional)
/lint.jar (optional)
```
可以看到.aar文件是专门为安卓设计的。因此这篇文章将教你如何创建与上传一个aar形式的library。

## build.gradle与gradle-warpper 的区别和联系
![](http://img.blog.csdn.net/20160518093304476)

如图中圈出位置所示，这俩个文件在项目中的位置。

build.gradle 文件制定编译时的一些条件和依赖关系。 
gradle-warpper.properties主要用来制定当前使用的gradle版本从哪里获取。以及一些其他的参数。

下面来分析一下这俩文件中的内容：

build.gradle 
这个文件的内容并不是固定的，根据项目的需要会有不同的设置。这里给出一般情况下的内容：
这是Module的gradle文件

```
//这里指明这是一个android工程，也可以填com.android.library
//指明这是一个类库
apply plugin: 'com.android.application'
android {
//使用的编译版本SDK21
    compileSdkVersion 21
    //buildtool版本 指定为21.1.1
    buildToolsVersion 21.1.1
    defaultConfig {
    //最小SDK17
        minSdkVersion 17
        //目标版本19
        targetSdkVersion 19
    }
    // 打包签名
    signingConfigs {
    //指定debug模式下使用的签名文件
        debug { storeFile file("debug.keystore") }

        release {
        //发布正式版本模式下的使用的签名文件
            storeFile file('release.keystore')
            storePassword 'thisiskeystorepassword'
            keyAlias 'nim_demo'
            keyPassword 'thisiskeypassword'
        }
    }
//编译时脚本运行环境
    buildTypes {
        debug {
            signingConfig signingConfigs.debug
            manifestPlaceholders = [AMAP_KEY: "09fd4efd3e28e9bf1f449ecec7d34bfe"]
        }
//正式版本要混淆
        release {
            minifyEnabled true
            zipAlignEnabled true
            proguardFile('proguard.cfg')
            signingConfig signingConfigs.release
            manifestPlaceholders = [AMAP_KEY: "ee20324fba1c7f4ad7a4a207e7f08e8d"]
        }
    }
    sourceSets {

        main {
        //指定文件映射关系
            manifest.srcFile 'AndroidManifest.xml'
            java.srcDirs = ['src']
            resources.srcDirs = ['src']
            aidl.srcDirs = ['src']
            renderscript.srcDirs = ['src']
            res.srcDirs = ['res', 'res-avchat', 'res-chatroom']
            assets.srcDirs = ['assets']
            jniLibs.srcDirs = ['libs', 'libs-sdk']

        }

    }
    //防止lint时报错
lintOptions {
        abortOnError false
    }
    packagingOptions {
        exclude 'META-INF/LICENSE.txt'
        exclude 'META-INF/NOTICE.txt'
    }

}
//添加依赖jar,库工程
dependencies {
//依赖文件夹下的所有文件
    compile fileTree(dir: 'libs', exclude: ['android-support-*.jar'], include: '*.jar')

    compile project(path: ':uikit')
    compile 'com.android.support:appcompat-v7:21.0.3'
}
```

一个根目录下的gradle文件，这个文件的设置对project下的所有module都是有效的

```
buildscript {
//指定要使用的gradle版本
    dependencies {
        classpath 'com.android.tools.build:gradle:2.0.0'

        // NOTE: Do not place your application dependencies here; they belong
        // in the individual module build.gradle files
    }
   // apply from: 'script.gradle', to: buildscript
}
//指定远程仓库，建议使用jcenter
allprojects {
    repositories {
        jcenter()
    }
}
```

一个gradle-wrapper。properties文件。只要设置gradle的缓存地址和下载地址。

```
#Wed May 18 07:57:25 CST 2016
distributionBase=GRADLE_USER_HOME
distributionPath=wrapper/dists
zipStoreBase=GRADLE_USER_HOME
zipStorePath=wrapper/dists
#指定gradle的下载地址
distributionUrl=https\://services.gradle.org/distributions/gradle-2.10-all.zip
```

