---
title: "Reactive Stream基本概念"
subtitle: "Reactive Stream"
layout: post
author: "bulingfeng"
header-style: text
tags:
- develop
- Reactive Stream
---

# Reactive Stream

## 编程模型

- 语言模型：响应式编程+函数式编程
- 并发模型：多线程编程
- 对立模型：命令式编程

## Reactive 数据结构

- 流式（Streams）
- 序列（Sequences）
- 事件（Events）

## Reactive 设计模式

- 拓展模式：观察者模式（Observer）（推的模式）
- 对立模式：迭代器（Iterator）（拉的模式）
- 混合模式：反应堆（Reactor）-同步，Proactor 异步

## Reactive Streams的使用场景

- 通常这种方式并不能使应用运行的更快。
- 利用较少的资源来提升伸缩性。

## Rective Streams的API组件

- Publisher：数据发布者（上游）
- Subscriber：数据订阅者（下游）
  - onSubscribe:下游订阅
  - onNext:下游接受到数据的时候
  - onComplete:当数据(Data Stream)执行完成的时候
  - onError:发生错误的时候
- Subscription：订阅信号
  - request：请求上游元素的数量
  - cancel：请求停止发送数据并且清除资源
- Processor：Publisher和Subscriber之间的一个存在；

## Reactor 框架的运用

### 核心API

- Mono:0-1的非阻塞结果
- Flux:0-N的非阻塞序列
- Scheduler:Reactor的调度线程池

## 代码实现

引入pom依赖

```xml
        <dependency>
            <groupId>io.projectreactor</groupId>
            <artifactId>reactor-core</artifactId>
            <version>3.5.4</version>
        </dependency>
```

### 使用非异步的方式

```java
public class ReactiveDemo {
    public static void main(String[] args) {
        print("main");
        Flux.just("a","b","c")
                .subscribe(ReactiveDemo::print, // 输出
                        ReactiveDemo::print,
                        ()->{print("执行完成....");}
                ); // 打印异常
        System.out.println("main method finishes");
    }


    public static void print(Object object){
        String name = Thread.currentThread().getName();
        System.out.println("current thread's name is:"+name+","+object);
    }
}
```

```
输出结果：
current thread's name is:main,main
current thread's name is:main,a
current thread's name is:main,b
current thread's name is:main,c
current thread's name is:main,执行完成....
main method finishes
```

以上结果可以看到所有的执行线程都是`main`线程。

### 使用异步的方式来执行任务

```java
public class ReactiveDemo {
    public static void main(String[] args) {
        print("main");
        Flux.just("a","b","c")
                .publishOn(Schedulers.boundedElastic())
                .subscribe(ReactiveDemo::print, // 输出
                        ReactiveDemo::print, // 打印异常
                        ()->{print("执行完成....");}
                ); 
        System.out.println("main method finishes");
    }


    public static void print(Object object){
        String name = Thread.currentThread().getName();
        System.out.println("current thread's name is:"+name+","+object);
    }
}
```

输出结果

```
current thread's name is:main,main
main method finishes
current thread's name is:boundedElastic-1,a
current thread's name is:boundedElastic-1,b
current thread's name is:boundedElastic-1,c
```

发现执行a,b,c的方法是线程boundedElastic-1，不再是main线程了。但是有个问题是，`执行完成`这句话并没有输出出来。这是异步编程常遇到的问题。

解决方案有2种，第一种就是在main方法阻塞一段时间，让a,b,c输入完成以后再打印`执行完成`；

第二种方案就是通过`Subscription`的request方法来实现背压，这样就能让数据流处理完成的时候，输出最后的`执行完成`。

```java
public class ReactiveDemo {
    public static void main(String[] args) {
        print("main");
        Flux.just("a","b","c")
                .publishOn(Schedulers.boundedElastic())
                .subscribe(ReactiveDemo::print, // 输出
                        ReactiveDemo::print,  // 打印异常
                        ()->{print("执行完成....");},
                        subscription -> {
                            // 只要request的数量大于等于流中元素的数量就可以把最终结果`执行完成`打印出来
                            subscription.request(3);
//                            subscription.request(Integer.MAX_VALUE);
                        }
                );
        System.out.println("main method finishes");
    }


    public static void print(Object object){
        String name = Thread.currentThread().getName();
        System.out.println("current thread's name is:"+name+","+object);
    }
}
```

输出结果

```
current thread's name is:main,main
main method finishes
current thread's name is:boundedElastic-1,a
current thread's name is:boundedElastic-1,b
current thread's name is:boundedElastic-1,c
current thread's name is:boundedElastic-1,执行完成....
```

匿名内部类的实现方式

```java
public static void main(String[] args) {
        print("main");
        Flux.just("a","b","c")
                .publishOn(Schedulers.boundedElastic())
                .subscribe(new Subscriber<String>() {
                               @Override
                               public void onSubscribe(Subscription subscription) {

                               }

                               @Override
                               public void onNext(String s) {

                               }

                               @Override
                               public void onError(Throwable throwable) {

                               }

                               @Override
                               public void onComplete() {

                               }
                           }
                );
        System.out.println("main method finishes");
    }


    public static void print(Object object){
        String name = Thread.currentThread().getName();
        System.out.println("current thread's name is:"+name+","+object);
    }
```

