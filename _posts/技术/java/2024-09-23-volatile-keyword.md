---
title: "The Volatile Keyword in Java"
subtitle: "The Volatile Keyword in Java"
layout: post
author: "bulingfeng"
header-style: text
tags:
- Java-English
---

## Overview

Without necessary synchronizations, the compiler, runtime, or processors may apply all sorts of optimizations. Even though these optimizations are usually benefical, sometimes they can cause **subtle** issues.

Caching and recording are optimizations that may surprise us in concurrent contexts. Java and the JVM provide many ways to control memory order, and the volatile keyword is one of them.

## Shared Multiprocessor Architecture

Processors are responsible for executing program **instructions**. Therefore, they must **retrieve** the program instructions and required data from RAM.

As CUPs can carry many instuctions per seconds, fetching from RAM isn't ideal for them. To improve this situation, processors use tricks like Out of Order Excution, Branch Prediction, Speculative Execution, and Caching.

This is where the following memory hierarchy comes into play:

![](https://bulingfeng.com/img/java基础/english/cups-architecture.png)

As different cores execute more instructions and **manipulate** more data, they fill their caches with more relevant data instructions. This will improve the overall performance at the expense of introducing cache **coherence** challenges.

We should think twice about what happens when one thread updates a cached value.

## Cache Coherence Challenges

```java
public class TaskRunner {

    private static int number;
    private static boolean ready;

    private static class Reader extends Thread {

        @Override
        public void run() {
            while (!ready) {
                Thread.yield();
            }

            System.out.println(number);
        }
    }

    public static void main(String[] args) {
        new Reader().start();
        number = 42;
        ready = true;
    }
}
```

The TaskRunner class maintains two simple variables. Its main method creates another thread that **spins** on the ready variable as long as it's false. When the variable becomes true, the thread prints the number variable.

Many may expect this program to print 42 after a short delay; however, the delay may be be much longer. It may even hang forever or pint zero.

The cause of these **anomalies** is the lack of proper memory visibility and reordering.

### Memory Visibility

This simple example has two application threads: the main and the reader threads. Let's imagine a **scenario** in which the os schedules those threads on two different CUP cores, where:

- The main thread has its copy of ready and number variables in its core cache.
- The reader thread ends up with its copies, too.
- The main thread updates the cached values.

Most modern processors write requests that won't be applied right after they're issued. Processors tend to queue those writes in a special write buffer. After a while, they'll apply those writes to the main memory all at once.

With all that being said, when the main thread updates the number and ready vaiables, there's no guarantee about what the reader thread may see. In other words, the reader thread may immediately see the updated value, with some delaym or never at all.

**This memory visibility may cause liveness issues in programs relying on visibility.**

### Reordering

To make matters even worse, the reader thread may see those writes in an order other than the actual program order.

```java
public static void main(String[] args) { 
    new Reader().start();
    number = 42; 
    ready = true; 
}
```

We may expect the reader thread to print 42. But it's actually possible to see zero as the printed value.

Reordering is an optimization technique for performance improvements. Interestingly, different componets may apply this optimazation:

- The processor may flush its write buffer in an order other than the program order.
- The processor may apply an out-of-order execution technique.
- The JIT compiler may optimize via reordering.

## Volatile Memory Order

We can use volatile to tackle the issues with Cach Coherence.

To ensure that updates to variables **propagate** predicatably to other threads, we should apply the volatile modifier to those variables.

This way, we can communicate with runtime and processor to not reorder any instruction involving the volatile variable. Also, processors understand that they should immediately flush any updates to these variables.

## Volatile and Thread Synchronization

For multi-thread applications, we need to ensure a couple of rules for consistent behaviour:

- **Mutual Exclusion**-only one thread executes a critical section at a time.
- Visibility- changes made by one thread to the shared data are visible to other threads to maintain data consistency.

synchronized methods and blocks provide both of the above properties at the cost of application performance.

volatile is quite a useful keyword because it can help ensure the visibility aspect of the data change without providing **mutual exclusion**. Thusm it's useful where we're ok with mutiple threads executing a block of code in parallel, but we need to ensure the visibility property.

## Happens-Before Ordering

The memory visibility effects of volatile variables extend beyond the volatile variables themselves.

To make matters more **concrete**, suppose thread A writes to a volatile varablem and then thread B reads the same volatile variable. In such cases, the values that were visible to A before writing the volatile variable will be visible to B after reading the volatile variable:

![](https://bulingfeng.com/img/java基础/english/happend-before.png)

Technically, any write to a volatile field happens-before every **subsequent** read of the same field.

### Piggybacking

Because of the strength of the happens-before memory ordering, sometimes we can piggyback on the visibility properties of another volatile variable.

```java
public class TaskRunner {

    private static int number; // not volatile
    private volatile static boolean ready;

    // same as before
}
```

Anything prior to writing true to the ready variable is visible to anything after reading the ready variable. Therefore, the number variable piggybacks on the memory visibility enforced by the ready variable. Simple put, even though it's not a volatile variable, it's **exhibiting** a volatile behavior.

Using these **semantics**, we can define only a few variables in our class as volatile and optimize the visibility guarantee.

## Conclusion

In this article, we explored the volatile keyword, its capabilities, and the improvements made to it, starting with Java 5.

