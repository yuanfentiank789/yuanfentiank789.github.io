---

layout: post
title:  "java.lang.Thread.UncaughtExceptionHandler"
date:   2017-05-11 1:05:00
catalog:  true
tags:

   - UncaughtExceptionHandler
   
       
   
---

## 一、有什么用

当一个线程抛出异常，如果没有显式处理（即try catch），JVM会将该异常事件报告给该线程对象的Java.lang.Thread.UncaughtExceptionHandler，如果没有设置UncaughtExceptionHandler，那么默认将会把异常栈信息输出到System.err。

   所以，无论是线程池也好，自己写的线程也好，如果你希望他在挂了的时候能给到你通知并处理一下，除了显示处理，还可以提供未捕获异常处理器。
## 二、怎么用

源码解析之后知道原理，再来谈怎么用就多余了，干脆在这一小段直接做铺垫，剩下的都放到源码解析中来谈（本身原理源码也不复杂）。

  java.lang.Thread类中和UncaughtExceptionHandler有关系的粗略算下来就8处，未捕获异常的一个实例对象及其set\get方法、一个静态对象及其静态的set\get方法，还有未捕获异常的内部接口定义，最后一个就是Thread内部属性ThreadGroup。
  
  ![image](http://img.blog.csdn.net/20160416133535685?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
  是的，你没有看错，第八个就是ThreadGroup，这货是未捕获异常接口的实现类。
  
  ![image](http://img.blog.csdn.net/20160416133701121?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
  
## 三、源码解析

实际上在这里上个例子会比较好，但是因为太简单了，就省略了···你要是自己写，就在Thread对象的set方法中传入未捕获异常的匿名对象就好了···

![image](http://img.blog.csdn.net/20160416134508072?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

从JDK文档中直接可以看到该接口的描述，当一个线程因未捕获的异常而即将终止时，JAVA虚拟机将使用Thread.getUncaughtExceptionHandler()查询该线程以获得其UncaughtExceptionHandler，并调用该handler的uncaughtException()方法，将线程和异常作为参数传递。如果某一线程没有明确设置其UncaughtExceptionHandler，则将他的ThreadGroup对象作为其handler。如果ThreadGroup对象对异常没有什么特殊的要求，那么ThreadGroup可以将调用转发给默认的未捕获异常处理器（即Thread类中定义的静态的未捕获异常处理器对象）。
    所以，关于该接口的原理描述基本就那么多了，直接上源码吧。
    
```
// 接口定义  
public interface UncaughtExceptionHandler {  
    void uncaughtException(Thread t, Throwable e);  
}  
// 未捕获异常实例属性：未捕获异常  
private volatile UncaughtExceptionHandler uncaughtExceptionHandler;  
// 未捕获异常静态属性：默认未捕获异常  
private static volatile UncaughtExceptionHandler defaultUncaughtExceptionHandler;  
// 设置默认的未捕获异常  
public static void setDefaultUncaughtExceptionHandler(UncaughtExceptionHandler eh) {  
    SecurityManager sm = System.getSecurityManager();  
    if (sm != null) {  
        sm.checkPermission(  
            new RuntimePermission("setDefaultUncaughtExceptionHandler")  
                );  
    }  
    defaultUncaughtExceptionHandler = eh;  
 }  
// 获取默认未捕获异常  
public static UncaughtExceptionHandler getDefaultUncaughtExceptionHandler(){  
    return defaultUncaughtExceptionHandler;  
}  
// 获取未捕获异常  
public UncaughtExceptionHandler getUncaughtExceptionHandler() {  
    // 这里才是高潮，如果没有设置未捕获异常，那么就将group属性当作未捕获异常  
    return uncaughtExceptionHandler != null ?  
        uncaughtExceptionHandler : group;  
}  
// 设置未捕获异常  
public void setUncaughtExceptionHandler(UncaughtExceptionHandler eh) {  
    checkAccess();  
    uncaughtExceptionHandler = eh;  
}  
```

根据JDK文档的描述，当出现未捕获异常的时候，JVM调用的是Thread.getUncaughtExceptionHandler()，这个方法中，如果你没有调用Thread.setUncaughtExceptionHandler()设置未捕获异常处理器，那么将会返回Thread.group，将ThreadGroup当作未捕获异常处理器，而ThreadGroup实现了UncaughtExceptionHandler，所以转到ThreadGroup的uncaughtException(Thread, Throwable)方法。

```
public void uncaughtException(Thread t, Throwable e) {  
    if (parent != null) {  
        parent.uncaughtException(t, e);  
    } else {  
        Thread.UncaughtExceptionHandler ueh =  
            Thread.getDefaultUncaughtExceptionHandler();  
        if (ueh != null) {  
            ueh.uncaughtException(t, e);  
        } else if (!(e instanceof ThreadDeath)) {  
            System.err.print("Exception in thread \""  
                             + t.getName() + "\" ");  
            e.printStackTrace(System.err);  
        }  
    }  
}  
```
先忽略其他不重要的信息（什么parent，这个后面再讲，涉及到该类的构造函数和Thread类的init方法，也不复杂），这里直接调用Thread类中的静态方法获取静态属性默认未捕获异常处理器。

   所以源码追下来看，和JDK文档中保持一致。剩下的几个问题，一是Thread类中ThreadGroup的初始化，二是ThreadGroup中parent的追踪（这里就省略了，两个构造函数，瞅一眼就好了···）。
   
   ![image](http://img.blog.csdn.net/20160416140515507?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
   
   关于Thread中ThreadGroup的设置，其实在Thread的所有构造函数中都会转调init方法，其逻辑就是如果在实例化线程对象的时候没有默认传入ThreadGroup，那么就会通过Thread.currentThread.getThreadGroup来得到线程组对象，main方法中有一个默认的main线程组，所以，即便你不传入，还是会有一个默认的。可以写个小demo打印看看，这里就省略了。
   
```

    private void init(ThreadGroup g, Runnable target, String name,  
                  long stackSize, AccessControlContext acc) {  
      if (name == null) {  
        throw new NullPointerException("name cannot be null");  
      }  
  
    this.name = name.toCharArray();  
  
    Thread parent = currentThread();  
    SecurityManager security = System.getSecurityManager();  
    if (g == null) {  
        if (security != null) {  
            g = security.getThreadGroup();  
        }  
        if (g == null) {  
            g = parent.getThreadGroup();  
        }  
    }  
    g.checkAccess();  
  
    if (security != null) {  
        if (isCCLOverridden(getClass())) {  
            security.checkPermission(SUBCLASS_IMPLEMENTATION_PERMISSION);  
        }  
    }  
  
    g.addUnstarted();  
  
    this.group = g;  
    this.daemon = parent.isDaemon();  
    this.priority = parent.getPriority();  
    if (security == null || isCCLOverridden(parent.getClass()))  
        this.contextClassLoader = parent.getContextClassLoader();  
    else  
        this.contextClassLoader = parent.contextClassLoader;  
    this.inheritedAccessControlContext =  
            acc != null ? acc : AccessController.getContext();  
    this.target = target;  
    setPriority(priority);  
    if (parent.inheritableThreadLocals != null)  
        this.inheritableThreadLocals =  
            ThreadLocal.createInheritedMap(parent.inheritableThreadLocals);  
    /* Stash the specified stack size in case the VM cares */  
    this.stackSize = stackSize;  
  
    /* Set thread ID */  
    tid = nextThreadID();  
    }



```   

 漏了一个分发的方法，即JVM调用该方法从而得到未捕获异常处理器，这里补上···
 
 ![image](http://img.blog.csdn.net/20160426224437805?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQv/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)
 
 
## 四、总结

1. 如果设置了实例属性uncaughtExceptionHandler（每个线程对象独有），则调用该处理器对未捕获异常进行处理；
 2. 如果没有设置未捕获异常处理器（即1中的uncaughtExceptionHandler），则将线程对象的ThreadGroup当作未捕获异常处理器，在ThreadGroup中获取所以线程对象共享的静态属性defaultUncaughtExceptionHandler来处理未捕获异常（前提是静态的set方法你调用了并且传入处理器实例）；
 3. 如果两个set方法都没有调用，那么异常栈信息将被推送到System.err进行处理；