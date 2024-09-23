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

The cause of these anomalies