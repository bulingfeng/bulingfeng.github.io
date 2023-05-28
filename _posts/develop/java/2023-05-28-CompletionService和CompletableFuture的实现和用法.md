---
title: "CompletionService和CompletableFuture的实现和用法"
subtitle: "CompletionService和CompletableFuture的实现和用法"
layout: post
author: "bulingfeng"
header-style: text
tags:
- develop
- 多线程
---

# CompletionService和CompletableFuture的实现和用法

## CompletionService

该接口提供了一些能力，可以让实现类来异步的执行一些列任务。配合future来实现，即当某个任务完成的时候，即可以获取到结果。

```java
import java.util.concurrent.*;

/**
 * 实现了一组线程异步完成，take操作可以根据已经完成的线程返回的数据来获取结果
 */
public class CompletionServiceTest {
    public static void main(String[] args) throws InterruptedException, ExecutionException {
        ExecutorService executorService = Executors.newFixedThreadPool(5);
        CompletionService<Integer> completionService=new ExecutorCompletionService<>(executorService);

        for (int i = 0; i < 10; i++) {
             completionService.submit(new Task(i + 1));
        }


        for (int i = 0; i < 10; i++) {
            Integer result = completionService.poll().get();
            System.out.println("任务i=="+result+"完成!");
        }


        // 关闭线程池
        executorService.shutdown();
    }


    static class Task implements Callable<Integer> {
        private Integer i;

        public Task(Integer i) {
            this.i = i;
        }

        public Integer getI() {
            return i;
        }

        public void setI(Integer i) {
            this.i = i;
        }

        @Override
        public Integer call() throws Exception {
            if (i==5)
                Thread.sleep(5000);
            else
                Thread.sleep(5000);

            System.out.println("i:"+this.i);

            return this.i;
        }
    }
}
```

## CompletableFuture

该类能够提供一种能力，让一组任务异步或者同步的按照顺序执行。

```java
public class CompletableFutureTest extends DataLoader{
    public static void main(String[] args) {
        CompletableFutureTest test=new CompletableFutureTest();
        test.load();

        System.out.println(Thread.currentThread().getName()+"main");
    }

    protected void load(){
        CompletableFuture.runAsync(super::loadConfiguration)
                .thenRun(super::loadOrders)
                .thenRun(super::loadUsers)
                .whenComplete((result,throwable)->{
                    System.out.println(Thread.currentThread().getName()+"load finish....");
                }).join();

    }

}
```

辅助类

```java
public class DataLoader {

    public void loadConfiguration(){
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            throw new RuntimeException(e);
        }
        System.out.println(Thread.currentThread().getName()+"loadConfiguration....");
    }


    public void loadOrders(){
        System.out.println(Thread.currentThread().getName()+"loadOrders....");
    }

    public void loadUsers(){
        System.out.println(Thread.currentThread().getName()+"loadUsers....");
    }
}
```

