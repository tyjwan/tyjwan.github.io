---
layout: post
title: Synchronized原理
categories: Java
description: 对synchronize的一些理解
keywords: synchronized, 多线程，锁，同步
---

众所周知 `synchronized` 关键字是解决并发问题常用解决方案，有以下三种使用方式:

- **同步普通方法，锁的是当前对象。**
- **同步静态方法，锁的是当前 `Class` 对象。**
- **同步块，锁的是 `()` 中的对象。**

实现原理： `JVM` 是通过进入、退出对象监视器( `Monitor` )来实现对方法、同步块的同步的。

具体实现是在编译之后在同步方法调用前加入一个 `monitor.enter` 指令，在退出方法和异常处插入 `monitor.exit` 的指令。

其本质就是对一个对象监视器( `Monitor` )进行获取，而这个获取过程具有排他性从而达到了同一时刻只能一个线程访问的目的。

而对于没有获取到锁的线程将会阻塞到方法入口处，直到获取锁的线程 `monitor.exit` 之后才能尝试继续获取锁。



### 一. 方法级的同步

​		方法级的同步是隐式的。同步方法的常量池中会有一个**ACC_SYNCHRONIZED**标志。当某个线程要访问某个方法的时候，会检查是否有**ACC_SYNCHRONIZED**，如果有设置，则需要先获得**监视器锁**，然后开始执行方法，方法执行之后再释放监视器锁。这时如果其他线程来请求执行方法，会因为无法获得监视器锁而被阻断住。值得注意的是，如果在方法执行过程中，**发生了异常，并且方法内部并没有处理该异常，那么在异常被抛到方法外面之前监视器锁会被自动释放**。

1. **代码**

```java
    //同步方法
    public synchronized void testFunction(){
        System.out.println("test func synchronized");
    }
```

2. **字节码**

```java
public synchronized void testFunction();
    descriptor: ()V
    flags: ACC_PUBLIC, ACC_SYNCHRONIZED
    Code:
      stack=2, locals=1, args_size=1
         0: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         3: ldc           #3                  // String Hello World
         5: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
         8: return
```



### 二. 代码块同步

​		同步代码块使用`monitorenter`和`monitorexit`两个指令实现。可以把执行`monitorenter`指令理解为加锁，执行`monitorexit`理解为释放锁。 每个对象维护着一个记录着被锁次数的计数器。未被锁定的对象的该计数器为0，当一个线程获得锁（执行`monitorenter`）后，该计数器自增变为 1 ，当同一个线程再次获得该对象的锁的时候，计数器再次自增。当同一个线程释放锁（执行`monitorexit`指令）的时候，计数器再自减。当计数器为0的时候。锁将被释放，其他线程便可以获得锁。

1. **代码**

```java
    //同步代码块
    public void test(){
        synchronized (SynchronizedDemo.class){
            System.out.println("test");
        }
    }
```

2. **字节码**

```java
public void test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=3, args_size=1
         0: ldc           #5                  // class com/hollis/SynchronizedTest
         2: dup
         3: astore_1
         4: monitorenter
         5: getstatic     #2                  // Field java/lang/System.out:Ljava/io/PrintStream;
         8: ldc           #3                  // String Hello World
        10: invokevirtual #4                  // Method java/io/PrintStream.println:(Ljava/lang/String;)V
        13: aload_1
        14: monitorexit
        15: goto          23
        18: astore_2
        19: aload_1
        20: monitorexit
        21: aload_2
        22: athrow
        23: return

```



#### 总结：

​	对于同步方法，JVM采用`ACC_SYNCHRONIZED`标记符来实现同步。 对于同步代码块。JVM采用`monitorenter`、`monitorexit`两个指令来实现同步。无论是`ACC_SYNCHRONIZED`还是`monitorenter`、`monitorexit`都是基于Monitor实现的，在Java虚拟机(HotSpot)中，Monitor是基于C++实现的，由ObjectMonitor实现。

​	ObjectMonitor类中提供了几个方法，如`enter`、`exit`、`wait`、`notify`、`notifyAll`等。`sychronized`加锁的时候，会调用objectMonitor的enter方法，解锁的时候会调用exit方法。Monitor的具体实现原理可看：https://www.hollischuang.com/archives/2030



### 参考资料

https://github.com/crossoverJie/JCSprout/blob/master/MD/Synchronize.md

https://juejin.cn/post/6844903653824790542

https://www.hollischuang.com/archives/1883

https://www.hollischuang.com/archives/2030