---
title: "CountDownLatch和CycliBarrier"
subtitle: "CountDownLatch和CycliBarrier"
layout: post
author: "bulingfeng"
header-style: text
tags:
- java基础
---

## 简介

CountDownLatch是线程挨个执行，直到全部完成一组线程的任务。CycliBarrier是线程之间会互相等待，到屏障的时候然后再一起执行任务。

```java
import java.util.concurrent.BrokenBarrierException;
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierExample {
    public static void main(String[] args) {
        // 创建一个 CyclicBarrier，当 3 个线程都到达屏障时执行指定的动作
        CyclicBarrier cyclicBarrier = new CyclicBarrier(3, () -> {
            System.out.println("所有线程都已到达屏障，开始下一阶段工作！");
        });

        // 创建并启动 3 个线程
        new Thread(new Task(cyclicBarrier, 1)).start();
        new Thread(new Task(cyclicBarrier, 2)).start();
        new Thread(new Task(cyclicBarrier, 3)).start();
    }

    static class Task implements Runnable {
        private CyclicBarrier cyclicBarrier;
        private int threadId;

        public Task(CyclicBarrier cyclicBarrier, int threadId) {
            this.cyclicBarrier = cyclicBarrier;
            this.threadId = threadId;
        }

        @Override
        public void run() {
            try {
                System.out.println("线程 " + threadId + " 正在执行任务...");
                Thread.sleep(threadId * 1000);
                System.out.println("线程 " + threadId + " 完成任务，等待其他线程到达屏障...");
                cyclicBarrier.await();
                System.out.println("线程 " + threadId + " 继续执行下一阶段任务...");
            } catch (InterruptedException | BrokenBarrierException e) {
                e.printStackTrace();
            }
        }
    }
}
```

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchExample {
    public static void main(String[] args) throws InterruptedException {
        // 创建一个初始计数值为 3 的 CountDownLatch
        CountDownLatch countDownLatch = new CountDownLatch(3);

        // 创建并启动 3 个工作线程
        new Thread(new Worker(countDownLatch, "Worker 1")).start();
        new Thread(new Worker(countDownLatch, "Worker 2")).start();
        new Thread(new Worker(countDownLatch, "Worker 3")).start();

        // 主线程等待 CountDownLatch 的计数值变为 0
        countDownLatch.await();
        System.out.println("所有工作线程都已完成任务，主线程继续执行。");
    }

    static class Worker implements Runnable {
        private CountDownLatch countDownLatch;
        private String name;

        public Worker(CountDownLatch countDownLatch, String name) {
            this.countDownLatch = countDownLatch;
            this.name = name;
        }

        @Override
        public void run() {
            try {
                System.out.println(name + " 正在执行任务...");
                Thread.sleep(1000);
                System.out.println(name + " 完成任务，进行倒计时。");
                countDownLatch.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    }
}
```

CountDownLatch也是可以线程CyclicBarrier.
