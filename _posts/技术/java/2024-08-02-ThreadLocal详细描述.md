---
title: "ThreadLocal的详细描述"
subtitle: "ThreadLocal的详细描述"
layout: post
author: "bulingfeng"
header-style: text
tags:
- java基础
---

## 简介

多线程出现线程安全的重要原因就是共享变量，如果没有共享变量，那么也就不存在线程安全的问题。而ThreadLocal可以从感觉上来让某些变量变成共享，但是又是线程安全的。

这个因为是感觉上是共享，只不过是都是同一个线程执行的时候才能拿到同样的共享数据，虽然没有显式的调用。

这是因为Thread类中有个成员变量`ThreadLocal.ThreadLocalMap threadLocals = null;` 每个线程只不过是把数据放到这里罢了。

## 源码实现

```java
public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null) {
            map.set(this, value);
        } else {
            createMap(t, value);
        }
}
```

```java
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

```java
 public void remove() {
     ThreadLocalMap m = getMap(Thread.currentThread());
     if (m != null) {
         m.remove(this);
     }
 }
```

需要注意的是，当使用完ThreadLocal以后，应该显式的调用remove()函数，让其从线程的ThreadLocalMap中删除，否则会出现内存泄露。

> 比如Tomcat中的线程池如果是配置为共享的线程池，那么线程池的生命周期其实和Tomcat的生命周期是一样的，即只有Tomcat停止之后线程池才会关闭，而进行热加载的之类的不会让这些线程销毁，由于大量的线程持有ThreadLocal.ThreadLocalMap的引用，所以会造成Tomcat的内存泄露。

