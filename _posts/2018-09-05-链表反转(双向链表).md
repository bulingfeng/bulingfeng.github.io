---
title: 链表反转(双向链表)
tags: 算法 链表
categories: 算法
---

* TOC
{:toc}

# 链表反转(双向链表)

## 反转逻辑

```
由于链表是使用引用来作为连接桥梁的，所以此处的坑是在变量的时候有同学没有保存当前节点next节点的引用，从而造成了循环遍历的时候造成死循环等已系列问题。
下面代码就是只操作当前节点之前的引用，然后又一个临时变量来保存下个节点的数据。
```

## 代码逻辑

```java
// 链表的反转
    public void reverse() {
        Node<E> temp1 = first;
        Node<E> newFist = null;

        while (temp1 != null) {
            Node<E> pre = temp1.pre;
            Node<E> next = temp1.next;
            // 核心要点就是操作当前节点之前的索引，不要操作当前遍历节点的next引用，所以要使用一个临时的变量来存储next节点数据
            if (pre == null) {
                temp1.next = null;
                temp1=next;
            } else if (next == null) {
                temp1.pre = null;
                temp1.next = pre;
                pre.pre = temp1;
                newFist = temp1;
                temp1=null;
            } else {
                temp1.next=pre;
                pre.pre =temp1;
                temp1=next;
            }

        }


        for (Node<E> temp = newFist; temp != null; temp = temp.next) {
            System.out.println(temp.item);
        }
    }
```

