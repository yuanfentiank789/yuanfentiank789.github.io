---
layout: post
title:  "Android性能：远程触发GC"
date:   2016-09-09 1:05:00
catalog:  true
tags:

   - java
   - wait
   - sleep
     


---

# Java上调用Object.wait()函数后，当前线程是否还拿着Object对象锁？

## 一、 在synchronized代码块中，调用Object.wait()后， 对象锁是否会释放？

答案是： yes, 对象锁被释放了。

如下为测试用例：

    // 用来做同步的对象
		final Object object = new Object();
		// 执行Object.wait()的 runnable对象
		Runnable runnable = new Runnable() {
			@Override
			public void run() {
				System.out.println(Thread.currentThread()+" run1");
				synchronized (object) {
					try {
						System.out.println(Thread.currentThread()+" run2");
						object.wait();
						System.out.println(Thread.currentThread()+" run3");
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}
		};

		new Thread(runnable, "wait_thread_1").start();
		new Thread(runnable, "wait_thread_2").start();
		
		// sleep
		Thread.sleep(3000);
		
		new Thread( new Runnable() {
			
			@Override
			public void run() {
				synchronized (object) {
					System.out.println(Thread.currentThread()+" notifyAll");
					object.notifyAll();
				}
			}
		}, "notifyAll_thread").start();
	
执行结果：

    Thread[wait_thread_1,5,main] run1
    // 第一个线程进入同步块
    Thread[wait_thread_1,5,main] run2

    Thread[wait_thread_2,5,main] run1
    // 第二个线程进入同步块
    Thread[wait_thread_2,5,main] run2

    // 唤醒第一个和第二个线程
    Thread[notifyAll_thread,5,main] notifyAll

    // 第一个和第二个线程被唤醒
    Thread[wait_thread_2,5,main] run3
    Thread[wait_thread_1,5,main] run3
    
## 二、在synchronized代码块中，调用Thread.sleep()后，对象锁是否被释放？

答案是： no, 对象锁没有被释放。

如下为测试用例：

    // 用来做同步的对象
		final Object object = new Object();
		// 执行Sleep的 runnable对象
		Runnable runnable = new Runnable() {
			@Override
			public void run() {
				System.out.println(Thread.currentThread()+" run1");
				synchronized (object) {
					try {
						System.out.println(Thread.currentThread()+" run2");
						Thread.sleep(10*1000);
						System.out.println(Thread.currentThread()+" run3");
					} catch (InterruptedException e) {
						// TODO Auto-generated catch block
						e.printStackTrace();
					}
				}
			}
		};

		new Thread(runnable, "wait_thread_1").start();
		new Thread(runnable, "wait_thread_2").start();
		
		// sleep
		Thread.sleep(3000);
		
		new Thread( new Runnable() {
			
			@Override
			public void run() {
				System.out.println(Thread.currentThread()+" notifyAll 1");
				synchronized (object) {
					System.out.println(Thread.currentThread()+" notifyAll 2");
					object.notifyAll();
					System.out.println(Thread.currentThread()+" notifyAll 3");
				}
			}
		}, "notifyAll_thread").start();
	
执行结果：

    Thread[wait_thread_2,5,main] run1
    Thread[wait_thread_1,5,main] run1
    // 第一个线程进入同步块，并sleep
    Thread[wait_thread_2,5,main] run2

    Thread[notifyAll_thread,5,main] notifyAll 1

    // 第一个线程sleep结束
    Thread[wait_thread_2,5,main] run3

    // 第三个线程抢到同步锁，进入同步块
    Thread[notifyAll_thread,5,main] notifyAll 2
    Thread[notifyAll_thread,5,main] notifyAll 3

    // 第二个线程抢到同步锁，进入同步块
    Thread[wait_thread_1,5,main] run2
    Thread[wait_thread_1,5,main] run3
    