---
title: "再谈OOM"
subtitle: "再谈OOM"
layout: post
author: "bulingfeng"
header-style: text
tags:
- java基础
---

## 简介

有些知识真的是需要不断的复习和回顾，否则以前可能非常熟悉的东西，时间一长的时候自己就可能会模糊甚至会忘记。特别是当面试的时候，当面试官询问自己相关的问题的时候，隐隐约约感觉自己都会，但是又莫名奇妙的说不出来，这个时候就比较尴尬了。

下面的事关于Java常见的oom的分析，java的版本是1.8。

## OOM发生的场景

在聊java发生OOM之前呢，先来看下JVM的组成结构，这样可以就可以分析不同区域内发现OOM的情况。

![](https://bulingfeng.com/img/java基础/JVM/3-JVM的结构.png)

这里就每个部分就不做过多的介绍了，重点来描述下`方法区`、`堆`这两块区域，因为这两块区域是发生oom最常见的两块区域。

**方法区**

方法区其实一种规定，不同的虚拟机和不同版本的虚拟机有不同的实现，比如在jdk7的时候方法区的实现是使用**永久代**来实现的。

在jdk8呢，这个永久代被移除了，然后用**metaspace**来实现方法区。

> 方法区主要存储的信息有：类的信息，静态常量，方法信息，运行时常量池，JIT编译代码缓存。

这个帖子有各种模仿OOM的场景

```
https://zzuli-tech.github.io/interview/jvm/memory/OutOfMemoryError.html
```

其实OOM可以大致分为这几类：

1. 堆内发生OOM
2. 堆外发生OOM（metaspace）
3. 虚拟机栈发生OOM

### 堆内发生OOM

1、流未关闭

```
https://www.sharkchili.com/pages/b69661/#%E5%8E%9F%E5%9B%A0%E5%88%86%E6%9E%90
```

2、创建大量对象

比如解析或者下载excel的时候，一次性查询出来大量的对象，然后这些对象还都是在使用过程中，所以造成OOM。

3、请求量增大，造成制造大量对象

4、代码问题，造成一些对象没有被回收。比如错误使用ThreadLocal

5、创建数组这样的大对象，从而造成分配的时候发生OOM

### metaspace发生OOM

1、装载的类太多

2、自定义转载的类太多

### 虚拟机栈OOM

递归的层数太多
