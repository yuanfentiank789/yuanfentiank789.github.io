---
layout: post
title:  "Java面试题库"
date:   2016-07-07 1:05:00
catalog:  true
tags:
    - 面试
  
       

---

# Java 面试题库
## 一.Java基础
### String

1 String的值传递，写出如下代码运行结果

    public static void main(String[] args) {

        String str = new String("adcdefg");
        char[] chars = {'s', 'd', 'f'};
        process(str, chars);
        System.out.println(str);
        System.out.println(chars);

    }

    private static void process(String str, char[] chars) {
        str = "fsdf";
        chars[0] = 'a';
    }

输结果应为：

    adcdefg
    adf
    
    
String虽然不是基本类型，但也用值传递，如果需要改变string的值，可改用StringBuilder代替。

[参考http://freej.blog.51cto.com/235241/168676](http://freej.blog.51cto.com/235241/168676)


### Class

1  Class.forName和ClassLoader.loadClass的比较

Class的装载分了三个阶段，loading，linking和initializing，分别定义在The Java Language Specification的12.2，12.3和12.4。
Class.forName(className)实际上是调用Class.forName(className, true, this.getClass().getClassLoader())。注意第二个参数，是指Class被loading后是不是必须被初始化。
ClassLoader.loadClass(className)实际上调用的是ClassLoader.loadClass(name, false)，第二个参数指出Class是否被link。
区别就出来了。Class.forName(className)装载的class已经被初始化，而ClassLoader.loadClass(className)装载的class还没有被link。
一般情况下，这两个方法效果一样，都能装载Class。但如果程序依赖于Class是否被初始化，就必须用Class.forName(name)了。
例如，在JDBC编程中，常看到这样的用法，Class.forName("com.mysql.jdbc.Driver")，如果换成了getClass().getClassLoader().loadClass("com.mysql.jdbc.Driver")，就不行。
为什么呢？打开com.mysql.jdbc.Driver的源代码看看，

    // Register ourselves with the DriverManager
    //
    static {
        try {
            java.sql.DriverManager.registerDriver(new Driver());
        } catch (SQLException E) {
        throw new RuntimeException("Can't register driver!");
        }
    }
    
原来，Driver在static块中会注册自己到java.sql.DriverManager。而static块就是在Class的初始化中被执行。所以这个地方就只能用Class.forName(className)。

