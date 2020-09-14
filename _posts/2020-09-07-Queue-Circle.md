---
title: 环形队列
tags: java 算法 队列
categories: 算法
---


## 简介

```
上次使用的数组来实现队列，但是存在一个问题当队列被塞满并且有些元素已经出列，会造成元素的不连续， 
所以需要使用移动队列中的元素，时间的复杂度为O(n)。 环形队列则会巧妙的解决该问题，因为环形队 
列仅仅维护head和tail的指针即可。
```

## 逻辑编写

### 图示

![环形队列示意图](http://bulingfeng.com/static/img/algorithm/circle-queue.jpg)

```
通过上图，我们会发现一个规律(head+1)%8才是head的真正下标，其实也很好理解，由于
环的容量是8，那么我们对8取模就会对于我们实际的下标位置。同理tail=(tail+1)%8。
```

### 代码实现

1、环形队列逻辑代码

```
package com.blf.book.quequ;

/**
 * @Author:bulingfeng
 * @Date: 2020-09-07
 * 环形的队列
 */
public class CircleQueue {

    private String[] items;
    private int capital;
    private int tail;
    private int head;

    public CircleQueue(int capital) {
        if (capital==0){
            capital=8;// 默认初始化为8个
        }
        this.items = new String[capital];
        this.capital = capital;
    }

    public boolean enqueue(String item){
        // 判断队列中有没有位置来存储数据
        if ((tail+1)%capital!=head){
            items[tail]=item;
        }else {
            // 其实这里还剩下一个坑位 并没有满员。这里实现的循环队列全部塞满实际上存了capital-1个元素
            throw new RuntimeException("现在队列已经满员");
        }
        tail=(tail+1)%capital;// 调整下次插入的指针
        return true;
    }

    public String outqueue(){
        // 如果队列内容为空 则返回空
        if (tail==head) return null;
        head=(head+1)%capital;
        return items[head];
    }
}

```

2、测试环形队列代码

```
package com.blf.book.quequ;

/**
 * @Author:bulingfeng
 * @Date: 2020-09-07
 */
public class CircleQueueTest {
    public static void main(String[] args) {
        CircleQueue circleQueue=new CircleQueue(10);
        for (int i = 0; i < 9; i++) {
            circleQueue.enqueue(i+"");
        }

        try {
            circleQueue.enqueue(10+"");
        } catch (Exception e) {
            System.out.println(e.getMessage());
        }

        circleQueue.outqueue();
        circleQueue.enqueue(10+"");

    }
}

```

