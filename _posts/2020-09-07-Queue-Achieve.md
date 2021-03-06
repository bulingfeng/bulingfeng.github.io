---
title: 数组来实现队列
tags: java 算法 队列
categories: 算法
---

* TOC
{:toc}

## 简述

```
队列的逻辑是FIFO，队列又可以分为数组队列和列表队列（其实就是是使用数组来实现队列还是用链表来实现队列，
此时的话是不是感觉很熟悉了，因为线程池中的存放任务的队列分别为ArrayBlockingQueue和LinkedBlockingQueue）。
区别是数组是有限长度的，如果队列被塞满的话，要不舍弃新进来的元素，要不进行扩容。
列表实现的队列可以增加节点让上个节点的引用指向最新的节点即可，缺点就是指针消耗资源。
```

## 数组来实现队列

### 队列的实现逻辑

```
package com.blf.book.quequ;

/**
 * @Author:bulingfeng
 * @Date: 2020-09-07
 */
// 使用数组来实现队列
public class ArrayQueue {
    private String[] items;
    private int capital;

    // 头和尾部的指针(此处的指针不等于编程语言中的指针，此指针仅仅用来定位元素)
    private int head;
    private int tail;

    public ArrayQueue(int capital) {
        this.items = new String[capital];
        this.capital = capital;
    }


    // 插入队列 此方法没有进行数据的移动
    public boolean enqueue(String item){
        // 判断是否应该移动数组里面的数据
        int moveCount=0;
        if ((tail == capital) && head!=0){
            System.out.println("触发了移动");
            for (int i=head;i<tail;i++){
                items[i-head]=items[head];
                System.out.printf("第%s移动 \n",++moveCount);
            }
            tail=tail-head;
            head=0;

        }

        items[tail]=item;
        tail++;
        return true;
    }

    // 从队列中取出数据
    public String outqueue(){
        if (tail==0) return null; // 当队列中为空的时候 返回null
        if (head==capital) throw new RuntimeException("数据已经被取完，没有数据可取");
        String item=items[head];
        head++;
        return item;
    }
}
```

### 测试类

```
package com.blf.book.quequ;

/**
 * @Author:bulingfeng
 * @Date: 2020-09-07
 */
public class QueueTest {
    public static void main(String[] args) {
        ArrayQueue queue=new ArrayQueue(10);
        // 入列
        for (int i = 0; i < 10; i++) {
            queue.enqueue(i+"");
        }

        // 出列
        for (int i = 0; i < 5; i++) {
            System.out.println(queue.outqueue());
        }

        // 测试数据的移动
        queue.enqueue("test");
    }
}
```

