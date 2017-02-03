---
layout: post
title:  "Java多线程之wait（）， notify() and notifyAll() "
date:   2016-06-20 1:05:00
catalog:  true
tags:
    - wait
    - notify
    - notifyAll

---

Java的多线程是一个很复杂的主题,需要耗费大量的注意力特别的处理多个线程访问同时一个或多个共享资源。Java 5引进了一些类如BlockingQueue和Executors,通过提供易于使用的api带减少这些复杂性。程序员使用这些类会更加轻松比直接使用wait()和notify()处理同步逻辑。我也会推荐在处理同步逻辑使用这些更新的api。但是很多时候我们必须使用,因为各种原因如维护遗留代码。在本教程中,我围绕wait(),notify()和notifyAll()讨论这些概念。

## 什么是wait(), notify() and notifyAll() 方法?


开始之前，先了解几个关于这几个方法的基本概念，java的Object类有3个final methods允许线程通信一个资源锁的状态，如下：

1 **wait()** : 告诉调用wait的线程释放锁继续休眠，直到其他持有同一对象锁的线程调用notify唤醒。wait方法释放锁先于重新获得锁，先于wait方法返回。wait方法使用中和synchronized机制结合，依赖于底层实现，也就是说，我们不能用纯java代码实现一个wait方法，因为他是用native代码实现的。wait的一般用法如下：

    ```
    synchronized( lockObject )
    { 
       while( ! condition )
        { 
            lockObject.wait();
        }
     
        //take the action here;
    }
    
    ```
2 **notify()** : 唤醒一个在同一对象锁上等待（该线程不参与本次竞争）的线程，需要注意的是：执行这个方法并不会真正释放该线程持有的对象锁，只是告诉在同一对象锁上等待的线程可以被唤醒了，直到synchronized代码块执行完才会释放锁。notify()的一般用法如下：

    ```
    synchronized(lockObject) 
    {
        //establish_the_condition;
 
        lockObject.notify();
     
       //any additional code if needed
    }
    ```

3 **notifyAll()**: 唤醒所有在同一对象锁上等待（该线程不参与本次竞争）的线程，具有最高优先级的线程将第一个运行在大多数情况下，但不能完全保证，其他和notify（）方法一样,一般用法如下：

    ````
    synchronized(lockObject) 
    {
        establish_the_condition;
 
        lockObject.notifyAll();
    } 
    ```
    
>In general, a thread that uses the wait() method confirms that a condition does not exist (typically by checking a variable) and then calls the wait() method. When another thread establishes the condition (typically by setting the same variable), it calls the notify() method. The wait-and-notify mechanism does not specify what the specific condition/ variable value is. It is on developer’s hand to specify the condition to be checked before calling wait() or notify().


到现在为止，讲了几个你可能已经知道的基本概念，下面通过一个小程序来更好的理解如何使用这些方法得到想要的结果。

##如何使用wait(), notify() and notifyAll() 方法

这个练习中，我们使用wait和notify方法解决经典的生产者，消费者问题。简单起见，我们的场景只包含一个producer和一个consumer。
（1） Producer线程每秒产生一个资源并放入taskQueue；

（2） Consumer线程每秒处理一个taskQueue中的资源，并移除；

（3） taskQueue同时最多可以存储5个资源；

（4） 两个线程无限运行。

### 生产者Thread

    ```
    static class Producer implements Runnable {
        private final List<Integer> taskQueue;
        private final int MAX_CAPACITY;

        public Producer(List<Integer> sharedQueue, int size) {
            this.taskQueue = sharedQueue;
            this.MAX_CAPACITY = size;
        }

        @Override
        public void run() {
            int counter = 0;
            while (true) {
                try {
                    produce(counter++);
                } catch (InterruptedException ex) {
                    ex.printStackTrace();
                }
            }
        }

        private void produce(int i) throws InterruptedException {
            synchronized (taskQueue) {
                while (taskQueue.size() == MAX_CAPACITY) {
                    System.out.println("Queue is full " + Thread.currentThread().getName() + " is waiting , size: " + taskQueue.size());
                    taskQueue.wait();
                }

                Thread.sleep(1000);
                taskQueue.add(i);
                System.out.println("Produced: " + i);
                taskQueue.notifyAll();
            }
        }
    }
    ```

### 消费者Thread

    ```
    static class Consumer implements Runnable {
        private final List<Integer> taskQueue;

        public Consumer(List<Integer> sharedQueue) {
            this.taskQueue = sharedQueue;
        }

        @Override
        public void run() {
            while (true) {
                try {
                    consume();
                } catch (InterruptedException ex) {
                    ex.printStackTrace();
                }
            }
        }

        private void consume() throws InterruptedException {
            synchronized (taskQueue) {
                while (taskQueue.isEmpty()) {
                    System.out.println("Queue is empty " + Thread.currentThread().getName() + " is waiting , size: " + taskQueue.size());
                    taskQueue.wait(100);
                }
                Thread.sleep(500);
                int i = (Integer) taskQueue.remove(0);
                System.out.println("Consumed: " + i);
                taskQueue.notifyAll();
            }
        }
    }
    ```
### 测试代码
    ```
    public static void main(String[] args) {
        List<Integer> taskQueue = new ArrayList<Integer>();
        int MAX_CAPACITY = 5;
        Thread tProducer = new Thread(new Producer(taskQueue, MAX_CAPACITY), "Producer");
        Thread tConsumer = new Thread(new Consumer(taskQueue), "Consumer");
        tProducer.start();
        tConsumer.start();
    }
    ```
    
建议更改生产者和消费者的休眠时间得到不同的输出。

## 需要注意的地方


   #调用obj的wait(), notify()方法前，必须获得obj锁，也就是必须写在synchronized(obj) {…} 代码段内。

　　# 调用obj.wait()后，线程A就释放了obj的锁，否则线程B无法获得obj锁，也就无法在synchronized(obj) {…} 代码段内唤醒A。

　　# 当obj.wait()方法返回后，线程A需要再次获得obj锁，才能继续执行。

　　# 如果A1,A2,A3都在obj.wait()，则B调用obj.notify()只能唤醒A1,A2,A3中的一个（具体哪一个由JVM决定）。

　　# obj.notifyAll()则能全部唤醒A1,A2,A3，但是要继续执行obj.wait()的下一条语句，必须获得obj锁，因此，A1,A2,A3只有一个有机会获得锁继续执行，

　　　　例如A1，其余的需要等待A1释放obj锁之后才能继续执行。

　　# 当B调用obj.notify/notifyAll的时候，B正持有obj锁，因此，A1,A2,A3虽被唤醒，但是仍无法获得obj锁。直到B退出synchronized块，释放obj锁后，

　　A1,A2,A3中的一个才有机会获得锁继续执行