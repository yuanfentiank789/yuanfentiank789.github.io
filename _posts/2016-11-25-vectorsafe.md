---
layout: post
title:  "Vector 是线程安全的吗？"
date:   2016-11-25 1:05:00
catalog:  true
tags:

   - vector
   - 线程安全
   
---

## 背景


大家应该经常被问到：Vector 与 ArrayList 的区别？
好多人一拍脑门就出：Vector 是线程安全的 (在任何情况下都是？)。。。

这样回答的原因应该是因为 Vector 的所有方法加上了 synchronized 关键字，从而保证访问 vector 的任何方法都必须获得对象的 intrinsic lock (或叫 monitor lock)，也即，在vector**内部内部内部**，其所有方法不会被多线程所访问。

## 问题
但是以下代码呢？

    if (!vector.contains(element)) 
    vector.add(element); 
    ...
    }
    
你还敢那么肯定的说出答案吗？

## 分析问题

  这是经典的 put-if-absent 情况，尽管 contains, add 方法都正确地同步了，但作为 vector 之外的使用环境，仍然存在  race condition（锁竞争）: 因为虽然条件判断contains与add都是原子性的操作 (atomic)，但在 if 条件判断为真后，那个用来访问vector.contains 方法的锁已经释放，在即将的 vector.add 方法调用 之间有间隙，在多线程环境中，完全有可能被其他线程获得 vector的 lock 并改变其状态, 此时当前线程的vector.add(element);  正在等待（只不过我们不知道而已）。只有当其他线程释放了 vector 的 lock 后，vector.add(element); 继续，但此时它已经基于一个错误的假设了。

  单个的方法 synchronized 了并不代表组合（compound）的方法调用具有原子性，使 compound actions  成为线程安全的可能解决办法之一还是离不开intrinsic lock (这个锁应该是 vector 的，但由 client 维护)：
  
  单个的方法 synchronized 了并不代表组合（compound）的方法调用具有原子性，使 compound actions  成为线程安全的可能解决办法之一还是离不开intrinsic lock (这个锁应该是 vector 的，但由 client 维护)
  
      / Vector v = ...
    public  boolean putIfAbsent(E x) {
    synchronized(v) { //注意此处的锁对象是vector实例和vector内部的锁对象一致
            boolean absent = !contains(x); 
            if (absent) { 
                add(x);
      } 
    }
        return absent; 
    }
    
## 结论

所以，正确地回答那个“愚蠢”的问题是：
Vector 和 ArrayList 实现了同一接口 List, 但所有的 Vector 的方法都具有 synchronized 关键修饰。但对于复合操作，Vector 仍然需要进行同步处理。 

这样做的后果，Vector 应该尽早地被废除，因为这样做本身没有解决多线程问题，反而，在引入了概念的混乱的同时，导致性能问题，因为 synchronized 的开销是巨大的：阻止编译器乱序，hint for 处理器寄存一/二级缓存。。。

