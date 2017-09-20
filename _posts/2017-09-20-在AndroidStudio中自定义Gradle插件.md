---

layout: post
title:  "在AndroidStudio中自定义Gradle插件"
date:   2017-09-20 1:06:00
catalog:  true
tags:

   - AndroidStudio
   - Gradle插件
   - gradle

   
   
       
   
---


## 1 创建Gradle Module

AndroidStudio中是没有新建类似Gradle Plugin这样的选项的，那我们如何在AndroidStudio中编写Gradle插件，并打包出来呢？

- (1) 首先，你得新建一个Android Project
- (2) 然后再新建一个Module，这个Module用于开发Gradle插件，同样，Module里面没有gradle plugin给你选，但是我们只是需要一个“容器”来容纳我们写的插件，因此，你可以随便选择一个Module类型（如Phone&Tablet Module或Android Librarty）,因为接下来一步我们是将里面的大部分内容删除，所以选择哪个类型的Module不重要。
- (3) 将Module里面的内容删除，只保留build.gradle文件和src/main目录。
由于gradle是基于groovy，因此，我们开发的gradle插件相当于一个groovy项目。所以需要在main目录下新建groovy目录
- (4) groovy又是基于Java，因此，接下来创建groovy的过程跟创建java很类似。在groovy新建包名，如：com.hc.plugin，然后在该包下新建groovy文件，通过new->file->MyPlugin.groovy来新建名为MyPlugin的groovy文件。
- (5) 为了让我们的groovy类申明为gradle的插件，新建的groovy需要实现org.gradle.api.Plugin接口。如下所示:

```
package  com.hc.plugin

import org.gradle.api.Plugin
import org.gradle.api.Project

public class MyPlugin implements Plugin<Project> {

    void apply(Project project) {
        System.out.println("========================");
        System.out.println("hello gradle plugin!");
        System.out.println("========================");
    }
}
```

- (6) 现在，我们已经定义好了自己的gradle插件类，接下来就是告诉gradle，哪一个是我们自定义的插件类，因此，需要在main目录下新建resources目录，然后在resources目录里面再新建META-INF目录，再在META-INF里面新建gradle-plugins目录。最后在gradle-plugins目录里面新建properties文件，注意这个文件的命名，你可以随意取名，但是后面使用这个插件的时候，会用到这个名字。比如，你取名为com.hc.gradle.properties，而在其他build.gradle文件中使用自定义的插件时候则需写成：
` apply plugin: 'com.hc.gradle'`

然后在com.hc.gradle.properties文件里面指明你自定义的类:
`implementation-class=com.hc.plugin.MyPlugin`

现在，你的目录应该如下：

![](http://img.blog.csdn.net/20160702123302689)

- (7) 因为我们要用到groovy以及后面打包要用到maven,所以在我们自定义的Module下的build.gradle需要添加如下代码：

```
apply plugin: 'groovy'
apply plugin: 'maven'

dependencies {
    //gradle sdk
    compile gradleApi()
    //groovy sdk
    compile localGroovy()
}

repositories {
    mavenCentral()
}
```

## 2 打包到本地Maven

前面我们已经自定义好了插件，接下来就是要打包到Maven库里面去了，你可以选择打包到本地，或者是远程服务器中。在我们自定义Module目录下的build.gradle添加如下代码：

```
apply plugin: 'groovy'
apply plugin: 'maven'


dependencies {
    //gradle sdk
    compile gradleApi()
    //groovy sdk
    compile localGroovy()
}

repositories {
    mavenCentral()
}

//group和version在后面使用自定义插件的时候会用到
group='com.hc.plugin'
version='1.0.0'

uploadArchives {
    repositories {
        mavenDeployer {
            //提交到远程服务器：
            // repository(url: "http://www.xxx.com/repos") {
            //    authentication(userName: "admin", password: "admin")
            // }
            //本地的Maven地址设置为D:/repos
            repository(url: uri('../repo'))
        }
    }
}
```

其中，group和version后面会用到，我们后面再讲。虽然我们已经定义好了打包地址以及打包相关配置，但是还需要我们让这个打包task执行。点击AndroidStudio右侧的gradle工具，如下图所示：
![](http://img.blog.csdn.net/20160702130539639)

可以看到有uploadArchives这个Task,双击uploadArchives就会执行打包上传啦！执行完成后，去我们的Maven本地仓库查看一下：

![](http://img.blog.csdn.net/20160702130836877)


## 3 使用自定义的插件

接下来就是使用自定义的插件了，一般就是在app这个模块中使用自定义插件，因此在app这个Module的build.gradle文件中，需要指定本地Maven地址、自定义插件的名称以及依赖包名。简而言之，就是在app这个Module的build.gradle文件中后面附加如下代码：

```
apply plugin: 'com.hc.gradle'
//apply plugin: 'com.auto.aspectj'

buildscript {
    repositories {
        maven {//本地Maven仓库地址
            url uri('../repo')
        }
        
    }
    dependencies {
        //格式为-->group:module:version
        classpath 'com.hc.plugin:helloplugin:1.0.0'
    }
}
```
好啦，接下来就是看看效果啦！切换到app目录下，终端执行：`../gradlew asembleDebug`,可看到如下输出：

```
Starting a Gradle Daemon, 1 busy and 1 incompatible Daemons could not be reused, use --status for details
Incremental java compilation is an incubating feature.
========================
hello gradle plugin!
========================
:app2:preBuild UP-TO-DATE
:app2:preDebugBuild UP-TO-DATE
:app2:checkDebugManifest
:app2:preReleaseBuild UP-TO-DATE

```

但是在library module中同样配置却一直无法生效，暂时没有找到原因，在Stack Overflow上也有人遇到同样问题，有个国外程序员自己开发插件解决了这个问题，语音是kotlin，github地址如下：
[https://github.com/Archinamon/GradleAspectJ-Android](https://github.com/Archinamon/GradleAspectJ-Android)，配置也很简单:

```
apply plugin: 'com.archinamon.aspectj'

buildscript {
    repositories {
        mavenCentral()
        maven { url "https://jitpack.io" }
    }
    dependencies {
        classpath 'com.github.Archinamon:GradleAspectJ-Android:3.0.3'
    }
}
```

## 4 开发只针对当前项目的Gradle插件

前面我们讲了如何自定义gradle插件并且打包出去，可能步骤比较多。有时候，你可能并不需要打包出去，只是在这一个项目中使用而已，那么你无需打包这个过程。

只是针对当前项目开发的Gradle插件相对较简单。步骤之前所提到的很类似，只是有几点需要注意：

- 新建的Module名称必须为BuildSrc
- 无需resources目录

目录结构如下所示：

![](http://img.blog.csdn.net/20160702135323958)

其中，build.gradle内容为：

```
apply plugin: 'groovy'

dependencies {
    compile gradleApi()//gradle sdk
    compile localGroovy()//groovy sdk
}

repositories {
    jcenter()
}
```

SecondPlugin.groovy内容为：

```
package  com.hc.second

import org.gradle.api.Plugin
import org.gradle.api.Project

public class SecondPlugin implements Plugin<Project> {

    void apply(Project project) {
        System.out.println("========================");
        System.out.println("这是第二个插件!");
        System.out.println("========================");
    }
}
```

在app这个Module中如何使用呢？直接在app的build.gradle下加入:

`apply plugin: com.hc.second.SecondPlugin`

验证方法同上。

## 5 Android Studio 调试 Gradle Plugin

### 5.1、创建remote调试任务：

选择 Eidt Configurations:

![](http://img.blog.csdn.net/20170216194659360?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2VhYmll/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

点左上角的 + 号，选择 remote。Name可以随意命名，其他配置可以不用动，端口就5005，点ok关闭

![](http://img.blog.csdn.net/20170216194740626?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2VhYmll/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

### 5.2 命令行运行

在plugin的apply方法处添加断点，打开Terminal窗口（一般在底下的工具栏上），在当前的工程目录下，输入 ：
`./gradlew assembleDebug -Dorg.gradle.daemon=false -Dorg.gradle.debug=true`,
assembleDebug 可以为其他的构建命令，但参数-Dorg.gradle.daemon=false -Dorg.gradle.debug=true要有。运行后出现`To honour the JVM settings for this build a new JVM will be forked.`,说明JVM在等待调试。

### 5.3 开始调试

选择前边创建的远程调试任务，点击debug开始调试：

![](http://img.blog.csdn.net/20170216194659360?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvY2VhYmll/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

注意，这时候窗口的焦点实在debug的输出窗口上，Terminal还是被挂起的，要点下Terminal窗口，gradle任务才会继续执行，并进入调试状态。





