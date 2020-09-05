---
title: 双向链表的实现
tags: 算法 链表
categories: 算法
---

* TOC
{:toc}

# 链表算法主要考点

1. **双向链表**
2. 链表翻转
3. 链表中环检测
4. 两个有序列表合并
5. 删除链表倒数第N个节点
6. 求链表的中间结点

## 双向链表实现

### 代码逻辑实现类

```
package com.blf.book.link;

/**
 * @Author:bulingfeng
 * @Date: 2020-09-04
 */
public class ArrayListDemo<E> {

    private Node<E> first;
    private Node<E> last;
    private int size;


    public void add(E e) {
        // 添加第一个节点
        if (first == null || last == null) {
            Node<E> newNode = new Node<>(e, null, null);
            first = newNode;
            last = newNode;
        } else {
            Node<E> newNode = new Node<>(e, last, null);
            // 把最后的节点的引用指向新节点
            last.next = newNode;
            newNode.pre = last;
            // 重置最后一个节点
            last = newNode;
        }
        size++;
    }

    public E get(int index) {
        if (index >= size) throw new RuntimeException("传入的index大于链表长度");
        int i = 0;
        Node<E> temp = first;
        while (true) {
            E item = temp.item;
            if (i == index) {
                return item;
            }
            temp = temp.next;
            i++;
        }
    }

    // 移除某个item 也可以移除item==null的值
    public boolean remove(E e) {
        if (e == null) {
            for (Node<E> temp = first.next; temp != null; temp = temp.next) {
                if (temp.item == null) {
                    unlink(temp);
                    return true;
                }
            }
        } else {
            for (Node<E> temp = first; temp != null; temp = temp.next) {
                if (temp.item.equals(e)) {
                    unlink(temp);
                    return true;
                }
            }
        }
        return false;
    }

    // 可以根据传入的节点来把前后的节点引用给处理下
    private E unlink(Node<E> node) {
        E e = node.item;
        Node<E> pre = node.pre;
        Node<E> next = node.next;
        if (pre == null) {
            // 这里只需要初始化下初始头节点即可
            first = next;
        } else {
            pre.next = next;
            // 改变当前节点的引用指向
            node.next = null;
        }

        if (next == null) {
            last = pre;
        } else {
            next.pre = pre;
            node.pre = null;
        }
        // 释放节点存值的引用
        node.item = null;
        size--;
        return e;
    }

    public static class Node<E> {
        private E item;
        private Node<E> pre;
        private Node<E> next;

        public Node(E item, Node<E> pre, Node<E> next) {
            this.item = item;
            this.pre = pre;
            this.next = next;
        }
    }
}
```

### 测试类

```
package com.blf.book.link;

/**
 * @Author:bulingfeng
 * @Date: 2020-09-04
 */
public class TestLinkDemo {
    public static void main(String[] args) {
        ArrayListDemo<String> list=new ArrayListDemo<>();
        list.add("1");
        list.add("2");
        list.add("3");

        // 测试remove某个数据
        list.remove("1");

        // 测试随机获取到某些index的数据
        System.out.println(list.get(0));
        System.out.println(list.get(1));


    }
}

```

## 总结

```
手动编写的双向链表的实现，是后续链表的翻转，有序链表的合并的基础。
```

