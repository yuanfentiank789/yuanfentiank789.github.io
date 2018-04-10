---

layout: post
title:  "Gradle和Groovy"
date:   2017-09-21 1:06:00
catalog:  true
tags:

   - AndroidStudio
   - Gradle
   - Groovy

   
   
       
   
---

## 1  Groovy介绍

Groovy是一种动态语言。这种语言比较有特点，它和Java一样，也运行于Java虚拟机中。简单粗暴点儿看，你可以认为Groovy扩展了Java语言。比如，Groovy对自己的定义就是：Groovy是在 java平台上的、 具有像Python， Ruby 和 Smalltalk 语言特性的灵活动态语言， Groovy保证了这些特性像 Java语法一样被 Java开发者使用。除了语言和Java相通外，Groovy有时候又像一种脚本语言。前文也提到过，当我执行Groovy脚本时，Groovy会先将其编译成Java类字节码，然后通过Jvm来执行这个Java类。图1展示了Java、Groovy和Jvm之间的关系。

![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image001.png?_=5532006)

### 1.1 groovy开发环境安装

```
curl -s get.sdkman.io | bash
source "$HOME/.sdkman/bin/sdkman-init.sh"
sdk install groovy
groovy -version
```

### 1.2 groovy基础


- Groovy注释标记和Java一样，支持//或者/**/
- Groovy语句可以不用分号结尾。Groovy为了尽量减少代码的输入，确实煞费苦心
- Groovy中支持动态类型，即定义变量的时候可以不指定其类型。Groovy中，变量定义可以使用关键字def。注意，虽然def不是必须的，但是为了代码清晰，建议还是使用def关键字
  
  ```
   def variable1 = 1   //可以不使用分号结尾
   def varable2 = "I am a person"
   def  int x = 1   //变量定义时，也可以直接指定类型
 ```
- 函数定义时，参数的类型也可以不指定。比如
 
  ```
  String testFunction(arg1,arg2){//无需指定参数类型
  ...
   }
```
- 除了变量定义可以不指定类型外，Groovy中函数的返回值也可以是无类型的。比如：

```
//无类型的函数定义，必须使用def关键字
def  nonReturnTypeFunc(){
     last_line   //最后一行代码的执行结果就是本函数的返回值
}
//如果指定了函数返回类型，则可不必加def关键字来定义函数
String  getString(){
   return "I am a string"
}
```
其实，所谓的无返回类型的函数，我估计内部都是按返回Object类型来处理的。毕竟，Groovy是基于Java的，而且最终会转成Java Code运行在JVM上

- 函数返回值：Groovy的函数里，可以不使用return xxx来设置xxx为函数返回值。如果不使用return语句的话，则函数里最后一句代码的执行结果被设置成返回值。比如

```
//下面这个函数的返回值是字符串"getSomething return value"
def getSomething(){
      "getSomething return value" //如果这是最后一行代码，则返回类型为String
      1000 //如果这是最后一行代码，则返回类型为Integer
}
```
注意，如果函数定义时候指明了返回值类型的话，函数中则必须返回正确的数据类型，否则运行时报错。如果使用了动态类型的话，你就可以返回任何类型了。

- Groovy对字符串支持相当强大，充分吸收了一些脚本语言的优点：

  -  单引号''中的内容严格对应Java中的String，不对$符号进行转义
  
   ```
     def singleQuote='I am $ dolloar'  //输出就是I am $ dolloar
  ```
  -  双引号""的内容则和脚本语言的处理有点像，如果字符中有$号的话，则它会$表达式先求值。
   
   ```
   def doubleQuoteWithoutDollar = "I am one dollar" //输出 I am one dollar
   def x = 1
   def doubleQuoteWithDollar = "I am $x dolloar" //输出I am 1 dolloar
   ```
  -  三个引号'''xxx'''中的字符串支持随意换行 比如

  ```
  def multieLines = ''' begin
     line  1 
     line  2
     end '''
 ```
- 最后，除了每行代码不用加分号外，Groovy中函数调用的时候还可以不加括号。比如：

```
println("test") ---> println "test"
```
注意，虽然写代码的时候，对于函数调用可以不带括号，但是Groovy经常把属性和函数调用混淆。比如

```
def getSomething(){
   "hello"
}
```
getSomething()   //如果不加括号的话，Groovy会误认为getSomething是一个变量。

### 1.3 groovy中的数据类型

Groovy中的数据类型我们就介绍两种和Java不太一样的：

- 一个是Java中的基本数据类型。
- 另外一个是Groovy中的容器类。
- 最后一个非常重要的是闭包。

根据Groovy的原则，如果一个类中有名为xxyyzz这样的属性（其实就是成员变量），Groovy会自动为它添加getXxyyzz和setXxyyzz两个函数，用于获取和设置xxyyzz属性值。实际上在coding的时候，是离不开SDK的。Groovy的API文档位于 http://www.groovy-lang.org/api.html

#### 1.3.1 基本数据类型
作为动态语言，Groovy世界中的所有事物都是对象。所以，int，boolean这些Java中的基本数据类型，在Groovy代码中其实对应的是它们的包装数据类型。比如int对应为Integer，boolean对应为Boolean。比如下图中的代码执行结果：
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image005.png?_=5532006)

#### 1.3.2 容器类

Groovy中的容器类很简单，就三种：

- List：链表，其底层对应Java中的List接口，一般用ArrayList作为真正的实现类。
- Map：键-值表，其底层对应Java中的LinkedHashMap。
- Range：范围，它其实是List的一种拓展。

1. List类

变量定义：List变量由[]定义，比如

```
def aList = [5,'string',true] //List由[]定义，其元素可以是任何对象
```
变量存取：可以直接通过索引存取，而且不用担心索引越界。如果索引超过当前链表长度，List会自动
往该索引添加元素

```
assert aList[1] == 'string'
assert aList[5] == null //第6个元素为空
aList[100] = 100  //设置第101个元素的值为10
assert aList[100] == 100
//那么，aList到现在为止有多少个元素呢？
println aList.size  ===>结果是101
```

2. Map类
   
   变量定义：Map变量由[:]定义，比如
   
   ```
   def aMap = ['key1':'value1','key2':true] 
   ```
   Map由[:]定义，注意其中的冒号。冒号左边是key，右边是Value。key必须是字符串，value可以是任何对象。另外，key可以用''或""包起来，也可以不用引号包起来。比如:
   
```
def aNewMap = [key1:"value",key2:true] //其中的key1和key2默认被处理成字符串"key1"和"key2"
//不过Key要是不使用引号包起来的话，也会带来一定混淆，比如
def key1="wowo"
def aConfusedMap=[key1:"who am i?"]
//aConfuseMap中的key1到底是"key1"还是变量key1的值“wowo”？显然，答案是字符串"key1"。如果要是"wowo"的话，则aConfusedMap的定义必须设置成：
def aConfusedMap=[(key1):"who am i?"]
//Map中元素的存取更加方便，它支持多种方法：
println aMap.keyName    //<==这种表达方法好像key就是aMap的一个成员变量一样
println aMap['keyName'] //<==这种表达方法更传统一点
aMap.anotherkey = "i am map" // <==为map添加新元素
```
3. Range类

Range是Groovy对List的一种拓展，变量定义和大体的使用方法如下：

```
def aRange = 1..5  //<==Range类型的变量 由begin值+两个点+end值表示
//左边这个aRange包含1,2,3,4,5这5个值
//如果不想包含最后一个元素，则
def aRangeWithoutEnd = 1..<5  <==包含1,2,3,4这4个元素
println aRange.from
println aRange.to
```
#### 1.3.3 闭包

##### 1.3.3.1 闭包基础

闭包，英文叫Closure，是一种数据类型，它代表了一段可执行的代码。其外形如下：

```
def aClosure = {//闭包是一段代码，所以需要用花括号括起来..  
    String param1, int param2 ->  //这个箭头很关键。箭头前面是参数定义，箭头后面是代码  
    println"this is code" //这是代码，最后一句是返回值，  
   //也可以使用return，和Groovy中普通函数一样  
}  
```

简而言之，Closure的定义格式是：

```
def xxx = {paramters -> code}  //或者  
def xxx = {无参数，纯code}  这种case不需要->符号
```
说实话，从C/C++语言的角度看，闭包和函数指针很像。闭包定义好后，要调用它的方法就是：

闭包对象.call(参数)  或者更像函数指针调用的方法：

闭包对象(参数)  

比如：

```
aClosure.call("this is string",100)  或者  
aClosure("this is string", 100)  
```
上面就是一个闭包的定义和使用。在闭包中，还需要注意一点：

如果闭包没定义参数的话，则隐含有一个参数，这个参数名字叫it，和this的作用类似。it代表闭包的参数。

比如：

```
def greeting = { "Hello, $it!" }
assert greeting('Patrick') == 'Hello, Patrick!'

```
等同于：

```
def greeting = { it -> "Hello, $it!" }
assert greeting('Patrick') == 'Hello, Patrick!'
```
但是，如果在闭包定义时，采用下面这种写法，则表示闭包没有参数！

```
def noParamClosure = { -> true }
```
这个时候，我们就不能给noParamClosure传参数了！

```
noParamClosure ("test")  <==报错喔！
```
##### 1.3.3.2 Closure使用中的注意点


  闭包在Groovy中大量使用，比如很多类都定义了一些函数，这些函数最后一个参数都是一个闭包。比如：

```
public static <T> List<T> each(List<T> self, Closure closure)
```
上面这个函数表示针对List的每一个元素都会调用closure做一些处理。这里的closure，就有点回调函数的感觉。但是，在使用这个each函数的时候，我们传递一个怎样的Closure进去呢？比如：

```
def iamList = [1,2,3,4,5]  //定义一个List
iamList.each{  //调用它的each，这段代码的格式看不懂了吧？each是个函数，圆括号去哪了？
      println it
}
```
上面代码有两个知识点：

- 省略圆括号
 each函数调用的圆括号不见了！原来，Groovy中，当函数的最后一个参数是闭包的话，可以省略圆括号。比如
 
 ```
 def  testClosure(int a1,String b1, Closure closure){
      //do something
      closure() //调用闭包
}
//那么调用的时候，就可以免括号！
testClosure (4, "test", {
   println "i am in closure"
} )  //红色的括号可以不写..
```
注意，这个特点非常关键，因为以后在Gradle中经常会出现图7这样的代码：
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image008.png?_=5532006)
经常碰见图7这样的没有圆括号的代码。省略圆括号虽然使得代码简洁，看起来更像脚本语言，但是它这经常会让我confuse（不知道其他人是否有同感），以doLast为例，完整的代码应该按下面这种写法：

```
 doLast({
   println 'Hello world!'
})
```
有了圆括号，你会知道 doLast只是把一个Closure对象传了进去。很明显，它不代表这段脚本解析到doLast的时候就会调用println 'Hello world!' 。

但是把圆括号去掉后，就感觉好像println 'Hello world!'立即就会被调用一样！

- 如何确定Closure的参数

  另外一个比较让人头疼的地方是，Closure的参数该怎么搞？还是刚才的each函数：
  
  ```
  public static <T> List<T> each(List<T> self, Closure closure)
  ```
  如何使用它呢？比如：
  
  ```
  def iamList = [1,2,3,4,5]  //定义一个List变量
iamList.each{  //调用它的each函数，只要传入一个Closure就可以了。
  println it
}
  ```
看起来很轻松，其实：

对于each所需要的Closure，它的参数是什么？有多少个参数？返回值是什么？
我们能写成下面这样吗？

```
iamList.each{String name,int x ->
  return x
}  //运行的时候肯定报错！因为函数对这个闭包肯定是有限定条件的
```
所以，Closure虽然很方便，但是它一定会和使用它的上下文有极强的关联。要不，作为类似回调这样的东西，我如何知道调用者传递什么参数给Closure呢？

此问题如何破解？只能通过查询API文档才能了解上下文语义。比如下图：
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image009.png?_=5532006)
 图中：

each函数说明中，将给指定的closure传递Set中的每一个item。所以，closure的参数只有一个。

findAll中，绝对抓瞎了。一个是没说明往Closure里传什么。另外没说明Closure的返回值是什么.....。

对Map的findAll而言，Closure可以有两个参数。findAll会将Key和Value分别传进去。并且，Closure返回true，表示该元素是自己想要的。返回false表示该元素不是自己要找的。示意代码如图9所示：
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image011.png?_=5532006)
 Closure的使用有点坑，很大程度上依赖于你对API的熟悉程度，所以最初阶段，SDK查询是少不了的。
 
### 1.4 脚本类、文件I/O和XML操作
 
#### 1.4.1 脚本类

Groovy中可以像Java那样写package，然后写类。比如在文件夹com/cmbc/groovy/目录中放一个文件，叫Test.groovy，如图所示：

![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image012.png?_=5532006)

如果不声明public/private等访问权限的话，Groovy中类及其变量默认都是public的。

现在，我们在测试的根目录下建立一个test.groovy文件。其代码如下所示：![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image013.png?_=5532006)

这两个groovy文件的目录结构如图所示：![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image014.png?_=5532006)
 在groovy中，系统自带会加载当前目录/子目录下的xxx.groovy文件。所以，当执行groovy test.groovy的时候，test.groovy import的Test类能被自动搜索并加载到。
 
 **脚本到底是什么?**

Groovy可以像写脚本一样，把要做的事情都写在xxx.groovy中，而且可以通过groovy xxx.groovy直接执行这个脚本。这到底是怎么搞的？

既然是基于Java的，Groovy会先把xxx.groovy中的内容转换成一个Java类。比如：

test.groovy的代码是：

println 'Groovy world!'  
Groovy把它转换成这样的Java类：

执行 groovyc -d classes test.groovy

groovyc是groovy的编译命令，-d classes用于将编译得到的class文件拷贝到classes文件夹下

图13是test.groovy脚本转换得到的java class。用jd-gui反编译它的代码：![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image015.png?_=5532006)
从图中看出以下几点:

- test.groovy被转换成了一个test类，它从script派生。
- 每一个脚本都会生成一个static main函数。这样，当我们groovy test.groovy的时候，其实就是用java去执行这个main函数
- 脚本中的所有代码都会放到run函数中。比如，println 'Groovy world'，这句代码实际上是包含在run函数里的。
- 如果脚本中定义了函数，则函数会被定义在test类中。

groovyc是一个比较好的命令，读者要掌握它的用法。然后利用jd-gui来查看对应class的Java源码。

#### 1.4.2  脚本中的变量和作用域

前面说了，xxx.groovy只要不是和Java那样的class，那么它就是一个脚本。而且脚本的代码其实都会被放到run函数中去执行。那么，在Groovy的脚本中，很重要的一点就是脚本中定义的变量和它的作用域。举例：

def x = 1 <==注意，这个x有def（或者指明类型，比如 int x = 1）

```  
def printx(){  
   println x  
}  
```
printx()  <==报错，说x找不到

为什么？继续来看反编译后的class文件。
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image016.png?_=5532006)
printx被定义成test类的成员函数

def x = 1，这句话是在run中创建的。所以，x=1从代码上看好像是在整个脚本中定义的，但实际上printx访问不了它。printx是test成员函数，除非x也被定义成test的成员函数，否则printx不能访问它。

那么，如何使得printx能访问x呢？很简单，定义的时候不要加类型和def。即：

x = 1  <==注意，去掉def或者类型

```
def printx(){
   println x
}
printx()  <==OK
```
这次Java源码又变成什么样了呢？
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image017.png?_=5532006)
图15中，x也没有被定义成test的成员函数，而是在run的执行过程中，将x作为一个属性添加到test实例对象中了。然后在printx中，先获取这个属性。

注意，Groovy的文档说 x = 1这种定义将使得x变成test的成员变量，但从反编译情况看，这是不对的.....

虽然printx可以访问x变量了，但是假如有其他脚本却无法访问x变量。因为它不是test的成员变量。
比如，我在测试目录下创建一个新的名为test1.groovy。这个test1将访问test.groovy中定义的printx函数：
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image018.png?_=5532006)
这种方法使得我们可以将代码分成模块来编写，比如将公共的功能放到test.groovy中，然后使用公共功能的代码放到test1.groovy中。

执行groovy test1.groovy，报错。说x找不到。这是因为x是在test的run函数动态加进去的。怎么办？

import groovy.transform.Field;   //必须要先import
@Field x = 1  <==在x前面加上@Field标注，这样，x就彻彻底底是test的成员变量了。查看编译后的test.class文件，得到：
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image019.png?_=5532006)
这个时候，test.groovy中的x就成了test类的成员函数了。如此，我们可以在script中定义那些需要输出给外部脚本或类使用的变量了！

#### 1.4.3  文件I/O操作

#### 1.4.4 XML操作

## 2 Gradle介绍

Gradle是一个工具，同时它也是一个编程框架。前面也提到过，使用这个工具可以完成app的编译打包等工作，然你也可以用它干其他的事情。
- 当你把Gradle当工具看的时候，我们只想着如何用好它。会写、写好配置脚本就OK
- 当你把它当做编程框架看的时候，你可能需要学习很多更深入的内容。

### 2.1 Gradle开发环境部署
Gradle的官网：http://gradle.org/ ，文档位置：https://docs.gradle.org/current/release-notes 。页面顶部的User Guide和DSL Reference很关键。User Guide就是介绍Gradle的一本书，而DSL Reference是Gradle API的说明。以Ubuntu为例，下载Gradle：http://gradle.org/gradle-download/  选择Complete distribution和Binary only distribution都行。然后解压到指定目录。

最后，设置~/.bashrc，把Gradle加到PATH里，如图所示：![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image022.png?_=5532006)
执行source ~/.bashrc，初始化环境。

执行gradle --version，如果成功运行就OK了。注意，为什么说Gradle是一个编程框架？来看它提供的API文档：

https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html 原来，我们编写所谓的编译脚本，其实就是玩Gradle的API....所以它从更底层意义上看，是一个编程框架！

### 2.2 基本组件
Gradle中，每一个待编译的工程都叫一个Project。每一个Project在构建的时候都包含一系列的Task。比如一个Android APK的编译可能包含：Java源码编译Task、资源编译Task、JNI编译Task、lint检查Task、打包生成APK的Task、签名Task等。

一个Project到底包含多少个Task，其实是由编译脚本指定的插件决定。插件是什么呢？插件就是用来定义Task，并具体执行这些Task的东西。

刚才说了，Gradle是一个框架，作为框架，它负责定义流程和规则。而具体的编译工作则是通过插件的方式来完成的。比如编译Java有Java插件，编译Groovy有Groovy插件，编译Android APP有Android APP插件，编译Android Library有Android Library插件

好了。到现在为止，你知道Gradle中每一个待编译的工程都是一个Project，一个具体的编译过程是由一个一个的Task来定义和执行的。

#### 2.2.1  一个重要的例子

下面我们来看一个实际的例子。这个例子非常有代表意义。下图是一个名为posdevice的目录。这个目录里包含3个Android Library工程，2个Android APP工程。![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image024.png?_=5532006)
在例子中：

- CPosDeviceSdk、CPosSystemSdk、CPosSystemSdkxxxImpl是Android Library。其中，CPosSystemSdkxxxImpl依赖CPosSystemSdk
- CPosDeviceServerApk和CPosSdkDemo是Android APP。这些App和SDK有依赖关系。CPosDeviceServerApk依赖CPosDeviceSdk，而CPosSdkDemo依赖所有的Sdk Library。

那么问题来了，在上面这个例子中，有多少个Project？

答案是：每一个Library和每一个App都是单独的Project。根据Gradle的要求，每一个Project在其根目录下都需要有一个build.gradle。build.gradle文件就是该Project的编译脚本，类似于Makefile。

看起来好像很简单，但是请注意：posdevice虽然包含5个独立的Project，但是要独立编译他们的话，得：

1. cd  某个Project的目录。比如 cd CPosDeviceSdk
2. 然后执行 gradle  xxxx（xxx是任务的名字。对Android来说，assemble这个Task会生成最终的产物，所以gradle assemble）
这很麻烦啊，有10个独立Project，就得重复执行10次这样的命令。更有甚者，所谓的独立Project其实有依赖关系的。比如我们这个例子。

那么，我想在posdevice目录下，直接执行gradle assemble，是否能把这5个Project的东西都编译出来呢？

答案自然是可以。在Gradle中，这叫Multi-Projects Build。把posdevice改造成支持Gradle的Multi-Projects Build很容易，需要：

- 在posdevice下也添加一个build.gradle。这个build.gradle一般干得活是：配置其他子Project的。比如为子Project添加一些属性。这个build.gradle有没有都无所属。
- 在posdevice下添加一个名为settings.gradle。这个文件很重要，名字必须是settings.gradle。它里边用来告诉Gradle，这个multiprojects包含多少个子Project。
来看settings.gradle的内容，最关键的内容就是告诉Gradle这个multiprojects包含哪些子projects:

```
//通过include函数，将子Project的名字（其文件夹名）包含进来  
include  'CPosSystemSdk' ,'CPosDeviceSdk' ,  
       'CPosSdkDemo','CPosDeviceServerApk','CPosSystemSdkWizarPosImpl'  
```
另外，settings.gradle除了可以include外，还可以设置一些函数。这些函数会在gradle构建整个工程任务的时候执行，所以，可以在settings做一些初始化的工作。比如：我的settings.gradle的内容：

```
//定义一个名为initMinshengGradleEnvironment的函数。该函数内部完成一些初始化操作
//比如创建特定的目录，设置特定的参数等
def initMinshengGradleEnvironment(){  
    println"initialize Minsheng Gradle Environment ....."  
    ......//干一些special的私活....  
    println"initialize Minsheng Gradle Environment completes..."  
}  
//settings.gradle加载的时候，会执行initMinshengGradleEnvironment  
initMinshengGradleEnvironment()  
//include也是一个函数：  
include 'CPosSystemSdk' , 'CPosDeviceSdk' ,  
      'CPosSdkDemo','CPosDeviceServerApk','CPosSystemSdkWizarPosImpl'
```

#### 2.2.2 Gradle命令介绍

##### 2.2.2.1 `gradle projects`查看工程信息

到目前为止，我们了解了Gradle什么呢？

- 每一个Project都必须设置一个build.gradle文件。至于其内容，我们留到后面再说。
- 对于multi-projects build，需要在根目录下也放一个build.gradle，和一个settings.gradle。
- 一个Project是由若干tasks来组成的，当gradle xxx的时候，实际上是要求gradle执行xxx任务。这个任务就能完成具体的工作。
- 当然，具体的工作和不同的插件有关系。编译Java要使用Java插件，编译Android APP需要使用Android APP插件。这些我们都留待后续讨论.
gradle提供一些方便命令来查看和Project，Task相关的信息。比如在posdevice中，我想看这个multi projects到底包含多少个子Project：

执行 gradle projects，得到输出如下：
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image025.png?_=5532006)multi projects的情况下，posdevice这个目录对应的build.gradle叫Root Project，它包含5个子Project。

，想看某个Project包含哪些Task信息，只要执行：

gradle project-path:tasks  就行。注意，project-path是目录名，后面必须跟冒号。

对于Multi-project，在根目录中，需要指定你想看哪个poject的任务。不过你要是已经cd到某个Project的目录了，则不需指定Project-path。



##### 2.2.2.2 `gradle tasks`查看任务信息
##### 2.2.2.3 `gradle task-name`执行任务
  
  上图列出了好多任务，这时候就可以通过 gradle 任务名来执行某个任务。这和make xxx很像。比如：

gradle clean是执行清理任务，和make clean类似。
gradle properites用来查看所有属性信息。

### 2.3 Gradle工作流程

Gradle的工作流程其实蛮简单，用一个图来表达：
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image028.png?_=5532006)
该图告诉我们，Gradle工作包含三个阶段：

- 首先是初始化阶段。对我们前面的multi-project build而言，就是执行settings.gradle
- Initiliazation phase的下一个阶段是Configration阶段。
- Configration阶段的目标是解析每个project中的build.gradle。比如multi-project build例子中，解析每个子目录中的build.gradle。在这两个阶段之间，我们可以加一些定制化的Hook。这当然是通过API来添加的。
- Configuration阶段完了后，整个build的project以及内部的Task关系就确定了。恩？前面说过，一个Project包含很多Task，每个Task之间有依赖关系。Configuration会建立一个有向图来描述Task之间的依赖关系。所以，我们可以添加一个HOOK，即当Task关系图建立好后，执行一些操作。
- 最后一个阶段就是执行任务了。当然，任务执行完后，我们还可以加Hook。
最后，关于Gradle的工作流程，你只要记住：
- Gradle有一个初始化流程，这个时候settings.gradle会执行。
- 在配置阶段，每个Project都会被解析，其内部的任务也会被添加到一个有向图里，用于解决执行过程中的依赖关系。
- 然后才是执行阶段。你在gradle xxx中指定什么任务，gradle就会将这个xxx任务链上的所有任务全部按依赖顺序执行一遍！

下面来告诉你怎么写代码！

### 2.4 Gradle编程模型及API实例详解
https://docs.gradle.org/current/dsl/  <==这个文档很重要

Gradle基于Groovy，Groovy又基于Java。所以，Gradle执行的时候和Groovy一样，会把脚本转换成Java对象。Gradle主要有三种对象，这三种对象和三种不同的脚本文件对应，在gradle执行的时候，会将脚本转换成对应的对端：

- Gradle对象：当我们执行gradle xxx或者什么的时候，gradle会从默认的配置脚本中构造出一个Gradle对象。在整个执行过程中，只有这么一个对象。Gradle对象的数据类型就是Gradle。我们一般很少去定制这个默认的配置脚本。
- Project对象：每一个build.gradle会转换成一个Project对象。
- Settings对象：显然，每一个settings.gradle都会转换成一个Settings对象。

注意，对于其他gradle文件，除非定义了class，否则会转换成一个实现了Script接口的对象。这一点和Groovy的脚本类相似

当我们执行gradle的时候，gradle首先是按顺序解析各个gradle文件。这里边就有所所谓的生命周期的问题，即先解析谁，后解析谁。下图是Gradle文档中对生命周期的介绍：结合上一节的内容，相信大家都能看明白了。现在只需要看红框里的内容：![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image030.png?_=5532006)

#### 2.4.1 Gradle对象
我们先来看Gradle对象，它有哪些属性呢？如图所示：
![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image031.png?_=5532006)

在posdevice build.gradle中和settings.gradle中分别加了如下输出：

```
//在settings.gradle中，则输出"In settings,gradle id is"  
println "In posdevice, gradle id is " +gradle.hashCode()  
println "Home Dir:" + gradle.gradleHomeDir  
println "User Home Dir:" + gradle.gradleUserHomeDir  
println "Parent: " + gradle.parent  
```
得到结果如图所示：

![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image032.png?_=5532006)

- 在settings.gradle和posdevice build.gradle中，我们得到的gradle实例对象的hashCode是一样的（都是791279786）。
- HomeDir是我在哪个目录存储的gradle可执行程序。
- User Home Dir：是gradle自己设置的目录，里边存储了一些配置文件，以及编译过程中的缓存文件，生成的类文件，编译中依赖的插件等等。

#### 2.4.2 Project对象

每一个build.gradle文件都会转换成一个Project对象。在Gradle术语中，Project对象对应的是Build Script。

Project包含若干Tasks。另外，由于Project对应具体的工程，所以需要为Project加载所需要的插件，比如为Java工程加载Java插件。其实，一个Project包含多少Task往往是插件决定的。

所以，在Project中，我们要：

- 加载插件。
- 不同插件有不同的行话，即不同的配置。我们要在Project中配置好，这样插件就知道从哪里读取源文件等
- 设置属性。

##### 2.4.2.1 加载插件
Project的API位于https://docs.gradle.org/current/javadoc/org/gradle/api/Project.html 。加载插件是调用它的apply函数.apply其实是Project实现的PluginAware接口定义的：![](http://cdn.infoqstatic.com/statics_s2_20160517-0032u4/resource/articles/android-in-depth-gradle/zh/resources/image033.png?_=5532006)

参考： http://www.cnblogs.com/wxishang1991/p/5532006.html




