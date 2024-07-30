---
title: "AQS底层原理和ReentrantLock实现"
subtitle: "AQS底层原理和ReentrantLock实现"
layout: post
author: "bulingfeng"
header-style: text
tags:
- java基础
---

## 简介

虽然synchronized和ReentrantLock在性能上已经差别很小，但是ReentrantLock有很多特性是synchronized没有的特性。

共同点

- 都支持重入锁（实际上java的锁都支持重入锁，否则会造成很多死锁的情况）

ReentrantLock特有特性

- 设置锁是否是`公平锁`
- 可中断锁；这意味着在获取锁的时候，可以进行中断
- 判断是否获取到锁来进行制定逻辑，而synchronized只能是阻塞性的锁
- 获取锁的时候可以指定在一定的时间内来获取锁

注意事项

> tryLock，lock，lockInterruptibly的调用都放到try外面；因为这些方法可能会报异常，如果在try里面的话造成会执行finally，从而造成了另外的异常。

## AQS详解

> AQS的全称`AbstractQueueSynchroizer`，抽象队列同步器。
>
> JUC中的很多锁（Lock、ReadWriteLock）和同步工具（Condition、Semaphore、CountDownLatch）都是基于AQS来实现的。

源码和数据结构

```java
public abstract class AbstractQueuedSynchronizer
                      extends AbstractOwnableSynchronizer {
    private transient volatile Node head;
    private transient volatile Node tail;
    private volatile int state;
}

public abstract class AbstractOwnableSynchronizer {
    private transient Thread exclusiveOwnerThread;
}
```

> 包括继承的`AbstractOwnableSynchronizer`,一共4个成员变量，那么就从这4个成员变量来进行分析。

### state

存0代码没有被线程占用，存1表示所已经被占用。大于1表示锁重入的次数

### exclusiveOwnerThread

exclusiveOwnerThread成员变量存储持有锁的线程，它配合state成员变量，可以实现锁的重入机制。

#### head和tail

在ObjectMonitor中，_cxq、_EntryList用来存储等待锁的线程，_WaitSet用来存储调用了wait()函数等待条件变量的线程。相比而言，AQS只有一个等待队列，它既可以用来存储等待锁的线程，又可以用来存储等待条件变量的线程。在ObjectMonitor中，_cxq使用单链表来实现，_EntryList和_WaitSet使用双向链表来实现。在AQS中，等待队列使用双向链表来实现。双向链表的节点定义如下所示。AQS中的head和tail两个成员变量分别为双向链表的头指针和尾指针。

### AQS定义的模版方法

AQS定义了8个模版方法，其中可以分成两组：独占模版和共享模版；

比如Lock是排他性锁，所以只会使用独占的模版；ReadWriteLock为读共享，写独占，所以读的时候会使用共享模版，写的时候使用独占模版。

Semaphore和CountdownLatch这会用到AQS的共享模式。

```java
public final void acquire(int arg) { ... }
public final void acquireInterruptibly(int arg)
                  throws InterruptedException { ... }
public final boolean tryAcquireNanos(int arg, long nanosTimeout)
                  throws InterruptedException { ... }
public final boolean release(int arg) { ... }

public final void acquireShared(int arg) { ... }
public final void acquireSharedInterruptibly(int arg)
                  throws InterruptedException { ... }
public final boolean tryAcquireSharedNanos(int arg, long nanosTimeout)
                  throws InterruptedException { ... }
public final boolean releaseShared(int arg) { ... }
```

### ReentrantLock基于AQS的实现

因为ReentrantLock即支持公平锁又支持非公平锁，所以有两个实现子类NonfairSync和FairSync。而NonfairSync和FairSync的释放锁的逻辑是一样的，所以再抽象出来一个Sync类。如图所示

![](https://bulingfeng.com/img/java基础/多线程/5-ReentrantLock的实现图.png)

ReentrantLock的lock是基于AQS的acquire()方法来实现的，unlock方法是基于AQS的release()方法来实现的。所以下面着重来分析下这两个方法

#### acquire()

acquire又分为两种情况：`公平锁`和`非公平锁`，那么我们就挨个来看下这两个实现的方式把

公平锁的实现

```java
/**
 * Acquires only if thread is first waiter or empty
 */
protected final boolean tryAcquire(int acquires) {
  /**
  * 1：getState()来判断是否当前锁被占用
  * 2：hasQueuedPredecessors 判断是否当前线程是队列里面的第一个
  * 3：compareAndSetState通过cas来获取到锁
  * 4：setExclusiveOwnerThread设置当前线程为独占锁
  */
    if (getState() == 0 && !hasQueuedPredecessors() &&
        compareAndSetState(0, acquires)) {
        setExclusiveOwnerThread(Thread.currentThread());
        return true;
    }
    return false;
}
```

非公平锁

```java
// 非公平锁就更简单了，只需要判断该线程能否抢到锁可以了
protected final boolean tryAcquire(int acquires) {
            if (getState() == 0 && compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(Thread.currentThread());
                return true;
            }
            return false;
 }
```

#### release()

```java
@ReservedStackAccess
protected final boolean tryRelease(int releases) {
    int c = getState() - releases;
    if (getExclusiveOwnerThread() != Thread.currentThread())
        throw new IllegalMonitorStateException();
    boolean free = (c == 0);
    if (free)
        setExclusiveOwnerThread(null);
    setState(c);
    return free;
}
```

#### tryLock

```java
@ReservedStackAccess
final boolean tryLock() {
    Thread current = Thread.currentThread();
    int c = getState();
    if (c == 0) {
        if (compareAndSetState(0, 1)) {
            setExclusiveOwnerThread(current);
            return true;
        }
    } else if (getExclusiveOwnerThread() == current) {
        if (++c < 0) // overflow
            throw new Error("Maximum lock count exceeded");
        setState(c);
        return true;
    }
    return false;
}
```

