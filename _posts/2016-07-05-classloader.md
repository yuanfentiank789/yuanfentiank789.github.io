---
layout: post
title:  "Java ClassLoader原理分析"
date:   2016-07-05 1:05:00
catalog:  true
tags:
    - ClassLoader
  
       

---

# Java ClassLoader原理分析

## 一.认识ClassLoader

Java中的所有类，必须被装载到jvm中才能运行，这个装载工作是由jvm中的类装载器完成的，类装载器所做的工作实质是把类文件从硬盘读取到内存中，JVM在加载类的时候，都是通过ClassLoader的loadClass()方法来加载class的。需要注意的是，程序在启动的时候，并不会一次性加载所有的class文件，而是根据需要，通过ClassLoader来动态加载相应的class文件到内存中。

## 二、Java默认提供的三个ClassLoader

先看下面一段代码：

    public class ClassLoaderSample{
    public static void main(String[]args){
        ClassLoader loader=ClassLoaderSample.class.getClassLoader();
        while(loader!=null){
            System.out.println(loader);
            loader=loader.getParent();
        }
        System.out.println(loader);
    }
    }
    
代码执行结果如下：

    sun.misc.Launcher$AppClassLoader@63e68a2b
    sun.misc.Launcher$ExtClassLoader@3479404a
    null


通过输出可看到通过ClassLoaderSample类直接获得的类加载器是App ClassLoader，而App ClasLoader的父类是Extension ClassLoader，但是Extension ClassLoader的父类却是null,实际上最顶层的是Bootstrap ClassLoader，只不过它是由C++写的，负责加载JDK中的核心类库。而这样设计的原因是涉及到一个类似操作系统中"鸡生蛋，蛋生鸡"的问题，因为所有的类都是通过ClassLoader装载，可是ClassLoader本身也是一个类，那第一个ClassLoader由谁来负责装载？实际上这部分工作就是由Bootstrap ClassLoader来完成的，类似于操作系统启动时的boot loader.

#### Bootstrap ClassLoader

Bootstrap ClassLoader称为启动加载器，是Java类中加载层次最顶层的类加载器，负责加载JDK中的核心类库，如rt.jar,resources.jar,charsets.jar等；另外，在命令行中用-Xbootclasspath选项指定的jar包也是通过Bootstrap ClassLoader加载。

看下面一段代码:


    public static void main(String[]args){
    URL[]urls=sun.misc.Launcher.getBootstrapClassPath().getURLs();
    for(int i=0;i<urls.length;++i){
        System.out.println(urls[i].toExternalForm());
     }
    }
    
输出结果如下：

    file:/Library/Java/JavaVirtualMachines/jdk1.7.0_65.jdk/Contents/Home/jre/lib/resources.jar
    file:/Library/Java/JavaVirtualMachines/jdk1.7.0_65.jdk/Contents/Home/jre/lib/rt.jar
    file:/Library/Java/JavaVirtualMachines/jdk1.7.0_65.jdk/Contents/Home/jre/lib/sunrsasign.jar
    file:/Library/Java/JavaVirtualMachines/jdk1.7.0_65.jdk/Contents/Home/jre/lib/jsse.jar
    file:/Library/Java/JavaVirtualMachines/jdk1.7.0_65.jdk/Contents/Home/jre/lib/jce.jar
    file:/Library/Java/JavaVirtualMachines/jdk1.7.0_65.jdk/Contents/Home/jre/lib/charsets.jar
    file:/Library/Java/JavaVirtualMachines/jdk1.7.0_65.jdk/Contents/Home/jre/lib/jfr.jar
    file:/Library/Java/JavaVirtualMachines/jdk1.7.0_65.jdk/Contents/Home/jre/classes

其实上述结果也是通过查找sun.boot.class.path这个系统属性所得知的,可以得到相同结果

    System.out.println(System.getProperty("sun.boot.class.path"));
    
#### Extension ClassLoader

扩展类加载器，负责加载Java的扩展类库，默认加载JAVA_HOME/jre/lib/ext/目录下的所有jar;另外，用-Djava.ext.dirs指定目录下的jar包也是通过Extension ClassLoader加载。

#### App ClassLoader

系统类加载器，负责加载应用程序classpath目录下的所有jar和class文件；另外，通过-Djava.class.path所指定的目录下的类和jar包也是通过它加载；

**注意**： 除了Java默认提供的三个ClassLoader之外，用户还可以根据需要定义自已的ClassLoader，而这些自定义的ClassLoader都 必须继承自java.lang.ClassLoader类，也包括Java提供的另外二个ClassLoader（Extension ClassLoader和App ClassLoader）在内，但是Bootstrap ClassLoader不继承自ClassLoader，因为它不是一个普通的Java类，底层由C++编写，已嵌入到了JVM内核当中，当JVM启动 后，Bootstrap ClassLoader也随着启动，负责加载完核心类库后，并构造Extension ClassLoader和App ClassLoader类加载器。

## 三 ClassLoader加载类的原理

### 加载原理

ClassLoader使用的是双亲委托模型来 搜索类的，每个ClassLoader实例都有一个父类加载器的引用（不是继承的关系，是一个包含的关系），虚拟机内置的类加载器（Bootstrap ClassLoader）本身没有父类加载器，但可以用作其它ClassLoader实例的的父类加载器。当一个ClassLoader实例需要加载某个 类时，它会试图亲自搜索某个类之前，先把这个任务委托给它的父类加载器，这个过程是由上至下依次检查的，首先由最顶层的类加载器Bootstrap ClassLoader试图加载，如果没加载到，则把任务转交给Extension ClassLoader试图加载，如果也没加载到，则转交给App ClassLoader 进行加载，如果它也没有加载得到的话，则返回给委托的发起者，由它到指定的文件系统或网络等URL中加载该类。如果它们都没有加载到这个类时，则抛出 ClassNotFoundException异常。否则将这个找到的类生成一个类的定义，并将它加载到内存当中，最后返回这个类在内存中的Class实 例对象。

这个加载过程的示意图如下：

![](https://img-blog.csdn.net/20160913070004322)

### 为什么要使用双亲委托这种模型呢？

因为这样可以避免重复加载，当父亲已经加载了该类的时候，就没有必要子ClassLoader再加载一次。考虑到安全因素，我们试想一下，如果不使 用这种委托模式，那我们就可以随时使用自定义的String来动态替代java核心api中定义的类型，这样会存在非常大的安全隐患，而双亲委托的方式， 就可以避免这种情况，因为String已经在启动时就被引导类加载器（Bootstrcp ClassLoader）加载，所以用户自定义的ClassLoader永远也无法加载一个自己写的String，除非你改变JDK中 ClassLoader搜索类的默认算法。

### 自定义ClassLoader进行动态远程加载

既然JVM已经提供了默认的类加载器，为什么还要定义自已的类加载器呢？
因为Java中提供的默认ClassLoader，只加载指定目录下的jar和class，如果我们想加载其它位置的类或jar时，比如：我要加载 网络上的一个class文件，通过动态加载到内存之后，要调用这个类中的方法实现我的业务逻辑。在这样的情况下，默认的ClassLoader就不能满足 我们的需求了，所以需要定义自己的ClassLoader。

定义自已的类加载器分为两步：

1、继承java.lang.ClassLoader

2、重写父类的findClass方法

读者可能在这里有疑问，父类有那么多方法，为什么偏偏只重写findClass方法？
因为JDK已经在loadClass方法中帮我们实现了ClassLoader搜索类的算法，当在loadClass方法中搜索不到类 时，loadClass方法就会调用findClass方法来搜索类，所以我们只需重写该方法即可。如没有特殊的要求，一般不建议重写loadClass 搜索类的算法。


从上面的分析可知，利用ClassLoader可以动态加载类，不仅是从本地，还可以从远程服务器上加载。

首先是服务端的代码：


    package wang.imallen.blog;
    public class Candy {

    public void display()
    {
        System.out.println("I'm marshmallow instead of Lollipop");
    }
    }
    
然后将编译生成的Candy.class部署到wang/imallen/blog的目录下。

下面是客户端的代码，为了从服务端加载我们需要的类，我们自定义了一个ClassLoader，代码如下：

    import java.io.ByteArrayOutputStream;
    import java.io.IOException;
    import java.io.InputStream;
    import java.net.URL;

    public class NetworkClassLoader extends ClassLoader
    {

    private static final String TAG=NetworkClassLoader.class.getSimpleName();

    private String url;

    public NetworkClassLoader(String url)
    {
        this.url=url;
    }

    @Override
    protected Class<?>findClass(String name)throws ClassNotFoundException
    {

        Class clazz=null;
        byte[]classData=downloadClassFile(name);
        if(classData==null)
        {
            throw new ClassNotFoundException();
        }
        clazz=defineClass(name,classData,0,classData.length);

        return clazz;   
    }

    private byte[]downloadClassFile(String name)
    {
        InputStream is=null;
        try
        {
            String path=className2Path(name);
            URL url=new URL(path);
            byte[]buff=new byte[1024];
            int len=-1;
            is=url.openStream();
            ByteArrayOutputStream baos=new ByteArrayOutputStream();
            while((len=is.read(buff))!=-1)
            {
                baos.write(buff,0,len);
            }   
            System.out.println("now we have downloaded the class file");
            return baos.toByteArray();
        }
        catch(Exception ex)
        {
            ex.printStackTrace();
        }
        finally
        {
            if(is!=null)
            {
                try
                {
                    is.close();
                }
                catch(IOException ex)
                {
                    ex.printStackTrace();
                }
            }
        }

        return null;
    }

    private String className2Path(String name)
    {
        return url+"/"+name.replace(".", "/")+".class";
    }
    }
    
下面是测试代码，为简便起见，也写在NetworkClassLoader中：

    public static void main(String[]args)
    {
    String rootUrl="http://192.168.2.102:8080/Test";
    String className="wang.imallen.blog.Candy";
    NetworkClassLoader loader=new NetworkClassLoader(rootUrl);

    try
    {
        Class<?>clazz=loader.loadClass(className);
        Object obj=clazz.newInstance();
        clazz.getMethod("display").invoke(obj); 
    }
    catch(Exception ex)
    {
        ex.printStackTrace();
    }

    }
    
下面是测试结果：

![image](http://7xn1yt.com1.z0.glb.clouddn.com/DynamicLoader.png)

上面这种方法其实有非常重要的应用，比如对已经发布的应用进行热补丁修复，或者是动态地添加新功能，而不需要重新发布，这也是我们需要自定义ClassLoader的一个重要原因。

###  JVM如何判断两个class是否相同

JVM在判定两个class是否相同时，不仅要判断两个类名是否相同，而且要判断是否由同一个类加载器实例加载的。只有两者同时满足的情况下，JVM才认为这两个class是相同的。否则，即使是同一个class文件，如果被两个不同的ClassLoader实例所加载，JVM也会认为它们是两个不同的类。要验证也很简单，我们只需要将在Candy中增加如下方法之后重新部署：


    public void test(Object obj)
      {
        if(obj instanceof UCandy)
        {
            System.out.println("Same class");
        }
        else
        {
            System.out.println("Different classes");
        }
      }
      
而测试代码如下所示：

    public static void main(String[]args)
    {   
    String rootUrl="http://192.168.2.102:8080/Test";
    String className="wang.imallen.blog.Candy";
    NetworkClassLoader loader1=new NetworkClassLoader(rootUrl);
    NetworkClassLoader loader2=new NetworkClassLoader(rootUrl);

    try
    {
        Class<?>clazz1=loader1.loadClass(className);
        Class<?>clazz2=loader2.loadClass(className);
        Object obj1=clazz1.newInstance();
        Object obj2=clazz2.newInstance();
        clazz1.getMethod("test",Object.class).invoke(obj1,obj2);    
    }catch(Exception ex)
    {
        ex.printStackTrace();
    }

    }   
    
测试结果如下：

    ![image](http://7xn1yt.com1.z0.glb.clouddn.com/DifferentClasses.png)
    
显然由于使用的ClassLoader不同，导致加载出来的类也不同。


