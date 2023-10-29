---
title: "Spring中Transactional注解的基本描述"
subtitle: "spring事务"
layout: post
author: "bulingfeng"
header-style: text
tags:
- spring
---
# Spring中Transactional注解的基本描述

官方文档：

> https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/annotation/Transactional.html

在写这个文章的时候多少还是有点感慨的，因为原来自己的学习模式都是通过搜索中文来进行然后再找博客来进行学习。但是这个模式缺陷是很大的的，缺陷如下：

> 1. 中文世界的博客文章大部分都是水文，都是互相抄袭的，检索的效率其实并不高；
> 2. 就算是没有互相抄袭，那么他的就一定对吗？这个还的需要进行验证，如果你没有进行验证，很可能就会被他给骗了。
>
> 而且有一个心痛的事实，那就是当自己会英文以后，发现中文的大部分博客和教程其实都可以消失了。因为很多东西官方网站都已经讲解的很明白了。下面我们就来结合官网来看下这个事务注解把。

## 关于Transactional注解的基本描述

这是代码的源码

```java
@Target({TYPE,METHOD})
@Retention(RUNTIME)
@Inherited
@Documented
@Reflective
public @interface Transactional
```

下面是对Transactional注解的描述，我把比较重要的描述都给摘抄出来，没错就是从官网摘抄出来！当自己写完这个博文的时候，可能会发现自己只是做对原文的摘抄或者说翻译。而中文世界的大多数可能连这个都没有达到，他们是在不断的复制和粘贴一些垃圾人写的垃圾文章而已。

```
Describes a transaction attribute on an individual method or on a class.
When this annotation is declared at the class level, it applies as a default to all methods of the declaring class and its subclasses. Note that it does not apply to ancestor classes up the class hierarchy; inherited methods need to be locally redeclared in order to participate in a subclass-level annotation. For details on method visibility constraints, consult the Transaction Management section of the reference manual.
```

然后来查看下比较重要的**rollbackFor**属性。

> Defines zero (0) or more exception [types](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Class.html), which must be subclasses of [`Throwable`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Throwable.html), indicating which exception types must cause a transaction rollback.
>
> By default, a transaction will be rolled back on [`RuntimeException`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/RuntimeException.html) and [`Error`](https://docs.oracle.com/en/java/javase/17/docs/api/java.base/java/lang/Error.html) but not on checked exceptions (business exceptions). See [`DefaultTransactionAttribute.rollbackOn(Throwable)`](https://docs.spring.io/spring-framework/docs/current/javadoc-api/org/springframework/transaction/interceptor/DefaultTransactionAttribute.html#rollbackOn(java.lang.Throwable)) for a detailed explanation.

翻译成中文就是:

> rollbackFor可以有一个或者多个异常的类型，但是这些异常类型必须都是Throwable的子类。这些异常或者异常的子类都会导致事务的回滚。
>
> 默认情况下，一个事务回滚是在RuntimeException和Error异常，并且这个异常并不包含检查时异常。
>
> > 检查性异常就是指：在编译节点会报错的异常。比如FileNotFoundException.

## 异常的几种常见情况

### 同一个类中的事务互相调用

```sql
    public void method1() throws FileNotFoundException {
        TableA a=new TableA();
        a.setName("method1");
        tableAMapper.insert(a);
        method2();
    }

    @Transactional(rollbackFor = Exception.class)
    public void method2() throws FileNotFoundException {
        TableB b=new TableB();
        b.setName("method2");
        tableBMapper.insert(b);
        throw new RuntimeException("method2发生异常");
    }
```

结果:

> tablea表中会多一条：method1；
>
> tableb表中会多一条：method2；
>
> 因为事务根本没有起到效果。
>
> 具体原因是什么呢？后续来分析源码来进行解释。

如果想让两者的事务都起到效果，那么两个方法都要加上事务，代码如下

```java
    @Transactional(rollbackFor = Exception.class)
    public void method1() throws FileNotFoundException {
        TableA a=new TableA();
        a.setName("method1");
        tableAMapper.insert(a);
        method2();
    }

    @Transactional(rollbackFor = Exception.class)
    public void method2() throws FileNotFoundException {
        TableB b=new TableB();
        b.setName("method2");
        tableBMapper.insert(b);
        throw new RuntimeException("method2发生异常");
    }
```

### 检查性异常不会生效

这个自己可以使用下，把RuntimeExcepton改成FileNotFoundException，那么异常就不会执行了。

### 如果是在两个类之间进行事务调用

这个情况是每个在方法上的事务都会进行回滚。但是前提要是必须在调用第一层的方法上进行加事务。