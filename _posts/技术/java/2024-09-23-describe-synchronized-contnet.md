---
title: "Describe the Synchronized Keyword in Java"
subtitle: "Describe the Synchronized Keyword in Java"
layout: post
author: "bulingfeng"
header-style: text
tags:
- Java-English
---

## Overview

In this article, we'll learn using synchronized block in Java.

**Simple put,** in a multi-thread environment, a race condition occurs when two or more threads attempt to update **mutable** share data at the same time. Jave Offers a **mechanism** to void race conditions by synchronizing thread access to shared data.

A piece of logic marked with synchronized becomes a synchronized block, allowing only one thread to execute at any given time.

## The Synchronized Keywork Usage

We can use the synchronized keyword on differnt leveles:

- Instance methods
- Static methods
- Code blocks

When we use a synchronized bock, Java internally uses a monitor, also known as monitor lock or **intrinsic** lock, to provide synchronization. These monitors are bound to a object;

Therefore, all synchronized blocks of the same object can have only one thread executing them at the same time.

### Synchronized Instance Method

We can add the synchronized keyword in the method declaration to make the method synchronized:

```java
public synchronized void sum(){
        count++;
}
```

Instance methods are synchroinzed over the instance of the class owning the method, **which means only one thread per instance of the class can execute this method.**

```java
public static void main(String[] args) throws InterruptedException {
        SynchronizedDemo synchronizedDemo1=new SynchronizedDemo();
        SynchronizedDemo synchronizedDemo2=new SynchronizedDemo();

        Thread t1=new Thread(()->{
            for (int i = 0; i < 100000; i++) {
                synchronizedDemo1.sum();
            }
        });


        Thread t2=new Thread(()->{
            for (int i = 0; i < 100000; i++) {
                synchronizedDemo2.sum();
            }
        });
        
        t1.start();
        t2.start();

        t1.join();
        t2.join();

        System.out.println("count:"+count);
    }
```

If there is two or more instances of class executing the synchronized method, synchronized keyword doesn't guarantee the method synchronizing, just like above code.

### Synchronized Static Method

Static methods are synchronized just like below:

```java
public static synchronized void sum(){
        count++;
}
```

These methods are synchronized on the Class object associated with the class. Since only one Class object exists  per JVM per class, **only on thread can execute inside a static synchronized method person class, irrespective of the number of instances it has.**

## Synchronized Blocks Within Methods

Sometimes we don't want to synchronize the entire method, only some instuctions within it.

```java
public  synchronized void sum(){
        synchronized (this){
            count++;
        }
}
```

## Reentrancy

Thre lock behind the synchronized methods and blocks is a reentrant. This means the current thread can acquire the same synchronized lock over and over again while holding it.

