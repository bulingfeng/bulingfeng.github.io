---
title: "ReentrantReadWriteLock总结"
subtitle: "ReentrantReadWriteLock总结"
layout: post
author: "bulingfeng"
header-style: text
tags:
- java基础
---

## 简介

ReentrantLock虽然已经能够实现锁的灵活控制，但是有个缺陷就是假如有读和写的两个方法，想通过保证读的值是准确的，写的值也是准确的。ReentrantLock虽然也能够实现，但是读和写总是互斥的，从而降低了程序的效率。

而ReentrantReadWriteLock有两把锁：读锁和写锁。这个锁的好处就是多个线程的读是可以并行的，但是多个线程的读和写是不行的，当然多个线程的写和写当然也是不行的。

ReentrantReadWriteLock的实现接口为`ReadWriteLock`，代码如下：

```java
public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}
```

```java
public class ReentrantReadWriteLockDemo {

    private ReentrantReadWriteLock rwLock=new ReentrantReadWriteLock();

    private Lock rLock = rwLock.readLock(); //读锁
    private Lock wLock = rwLock.writeLock(); //写锁


    private static int testNum=0;


    public static void main(String[] args) throws InterruptedException {
        ReentrantReadWriteLockDemo demo=new ReentrantReadWriteLockDemo();
        Thread t1=new Thread(()->{
            for (int i = 1; i < 10; i++) {
                demo.writeTest(i);
            }
        });

        Thread t2=new Thread(()->{
            while (testNum<10){
                demo.readTest();
            }
        });

        Thread t3=new Thread(()->{
            while (testNum<10){
                demo.readTest();
            }
        });


        t1.start();
        t2.start();
        t3.start();

        t1.join();
        t2.join();
        t3.join();
    }


    public void writeTest(int num){
        wLock.lock();
        try {
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            testNum=num;
            System.out.println("writeTest-testNum:"+testNum);
        }finally {
            wLock.unlock();
        }
    }

    public int readTest(){
        rLock.lock();
        try {

            System.out.println("testNum-readTest:"+testNum+",threadName:"+Thread.currentThread().getName());
            try {
                Thread.sleep(500);
            } catch (InterruptedException e) {
                throw new RuntimeException(e);
            }
            return testNum;
        } finally {
            rLock.unlock();
        }
    }
}
```

> 上面的代码会发现，当写数据的时候是不能读的。但是一旦写数据的线程完成以后，t2和t3的线程就开始同步进行读取，并且没有阻塞（因为两者是几乎同时出现的，而不是间隔500ms才出现的）

## 锁升级和锁降级

> 所谓的ReentrantReadWriteLock锁升级就是**同一个线程**在获取读锁的同时，然后再重入写锁。
>
> 锁降级就是**同一个线程**在获取到读锁的同时，然后重入写锁。
>
> > 其实也根本不用记这些所谓的锁升级和锁降级，其实就是描述了ReentrantReadWriteLock的基本特性：
> >
> > 同一个线程获取到写锁之后，不能够再获取到读锁或者写锁。
> >
> > 同意各方线程获取到读锁之后，既能够获取读锁也能够获取写锁。

锁升级代码

```java
// 当前线程获取到读锁以后，当获取写锁的时候会获取不到，这个时候读锁又不释放，所以就卡这里了
public void lockUpgrade(){
    rLock.lock();

    wLock.lock();

    rLock.unlock();

    wLock.unlock();
}
```

锁降级

```java
public void lockDegrade(){
    wLock.lock();
    // 写入数据
    testNum=1;

    rLock.lock();

    // 读数据
    System.out.println("testNum:"+testNum);

    wLock.unlock();

    rLock.unlock();
}
```

写锁加锁的流程图

![](https://bulingfeng.com/img/java基础/多线程/6-ReentrantReadWriteLock-写锁加锁流程图.png)

读锁的加锁流程图

![](https://bulingfeng.com/img/java基础/多线程/7-ReentrantReadWriteLock-读锁加锁的过程.png)
