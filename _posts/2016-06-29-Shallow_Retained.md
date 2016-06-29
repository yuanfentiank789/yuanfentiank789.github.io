---
layout: post
title:  "Shallow Size和Retained Size"
date:   2016-06-29 1:05:00
catalog:  true
tags:
    - Shallow
    - Retained
       

---

## Shallow size
Shallow size就是对象本身占用内存的大小，不包含其引用的对象。常规对象（非数组）的Shallow size有其成员变量的数量和类型决定。数组的shallow size有数组元素的类型（对象类型、基本类型）和数组长度决定。Shallow size of a set of objects represents the sum of shallow sizes of all objects in the set.在32位系统上，对象头占用8字节，int占用4字节，不管成员变量（对象或数组）是否引用了其他对象（实例）或者赋值为null它始终占用4字节。故此，对于String对象实例来说，它有三个int成员（3*4=12字节）、一个char[]成员（1*4=4字节）以及一个对象头（8字节），总共3*4 +1*4+8=24字节。根据这一原则，对String a=”rosen jiang”来说，实例a的shallow size也是24字节。（注意：上述String是jdk1.5的，代码如下：）

    public final class String  
    implements java.io.Serializable, Comparable<String>, CharSequence  
    {  
    /** The value is used for character storage. */  
    private final char value[];  
  
    /** The offset is the first index of the storage that is used. */  
    private final int offset;  
  
    /** The count is the number of characters in the String. */  
    private final int count;  
  
    /** Cache the hash code for the string */  
    private int hash; // Default to 0  

>jdk1.7的String实现已经变了。）

## Retained size

Retained size是该对象自己的shallow size，加上从该对象能直接或间接访问到对象的shallow size之和。换句话说，retained size是该对象被GC之后所能回收到内存的总和。为了更好的理解retained size，不妨看个例子。

把内存中的对象看成下图中的节点，并且对象和对象之间互相引用。这里有一个特殊的节点GC Roots，这就是reference chain的起点。

![image](http://www.yourkit.com/docs/90/help/retained_objects.gif)![image](http://www.yourkit.com/docs/90/help/retained_objects_2.gif)

从obj1入手，上图中蓝色节点代表仅仅只有通过obj1才能直接或间接访问的对象。因为可以通过GC Roots访问，所以左图的obj3不是蓝色节点；而在右图却是蓝色，因为它已经被包含在retained集合内。

所以对于左图，obj1的retained size是obj1、obj2、obj4的shallow size总和；右图的retained size是obj1、obj2、obj3、obj4的shallow size总和。
对于obj2，它的retained size是：在左图中，是obj2和obj4的shallow size的和；在右图中，是obj2、obj3和obj4的shallow size的和。

总之，retained size是一个整体度量，有助于理解内存结构和对象图中的依赖关系并找到根节点。

## 参考

[http://www.yourkit.com/docs/90/help/sizes.jsp](http://www.yourkit.com/docs/90/help/sizes.jsp)
