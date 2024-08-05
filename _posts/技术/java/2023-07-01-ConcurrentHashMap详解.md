---
title: "ConcurrentHashMap详解"
subtitle: "ConcurrentHashMap详解"
layout: post
author: "bulingfeng"
header-style: text
tags:
- java基础
---

## 简介

ConcurrentHashMap也是属于`读时复制`,也就是说当写进去一个元素的时候，同时你去取可能取不出来。相反，如果你删除一个元素的时候，同时写和进行读，可能你能读出来这个元素。这个到底算不算线程安全问题呢？

> 我的理解是：线程安全问题是造成了数据错误的问题，才能被描述为线程安全问题，上面所说的可以称之为：“数据不一致性”问题，因为你再次读就会得到正确的数据，从数据的角度来看，数据并没有错，只是读的实际不对，造成了数据的错误而已。

ConcurrentHashMap相比于其他的线程安全的Map，速度快，就是按照数组长度(n)，分为了n把锁，然后n个锁直接相互独立，互相不干扰。这就可以让锁的颗粒度更小，从而提高了访问效率。

## 详细介绍

从读，写这两个操作开始分析，而这三个操作设计到读，写，树化，扩容这几个动作。

### 读(get)

```java
public V get(Object key) {
  Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
  int h = spread(key.hashCode());
  if ((tab = table) != null && (n = tab.length) > 0 &&
      (e = tabAt(tab, (n - 1) & h)) != null) {
      if ((eh = e.hash) == h) {
          if ((ek = e.key) == key || (ek != null && key.equals(ek)))
              return e.val;
      }
      else if (eh < 0)
          return (p = e.find(h, key)) != null ? p.val : null;
      while ((e = e.next) != null) {
          if (e.hash == h &&
              ((ek = e.key) == key || (ek != null && key.equals(ek))))
              return e.val;
      }
  }
  return null;
}
```

get方法没有加锁，所以get可以在任何时候都进行获取，这个也造就了ConcurrentHashMap的效率高效。

### 写方法

写方法包括普通写，树化和扩容这个操作。

普通写

```java
public void put(K key, V value) {
  //1) 写操作逻辑
  int index = hash(key) & (table.length-1);
  // 通过hash值定位table[index],如果有值cas写入成功
  if (table[index] == null &&
      cas(table[index], null, new Node(key, value, null))) {
    return; //写入成功
  }
  // 如果table[index]存在对应的值，那么给table[index]加锁
  synchronzied(table[index]) {
    //写入逻辑：遍历链表查看是否存在key跟写入数据相同的节点，
    //如果存在，则更新此节点的value值；
    //如果不存在，则将写入数据对应的节点插入到链表的尾部。
  }

  //2) 树化逻辑
  //3) 扩容逻辑
}
```

树化

> 树化也要进行加锁，因为如果不加锁会造成数据的丢失。比如如果一个写操作认为是数组，另一个是正在进行树化，那么就会把树的引用给替换成链表的，因为调用树化的方法并没有在写的锁里面。

```java
private final void treeifyBin(Node<K,V>[] tab, int index) {
    Node<K,V> b; int n, sc;
    if (tab != null) {
        if ((n = tab.length) < MIN_TREEIFY_CAPACITY)
            tryPresize(n << 1);
        else if ((b = tabAt(tab, index)) != null && b.hash >= 0) {
            synchronized (b) {
                if (tabAt(tab, index) == b) {
                    TreeNode<K,V> hd = null, tl = null;
                    for (Node<K,V> e = b; e != null; e = e.next) {
                        TreeNode<K,V> p =
                            new TreeNode<K,V>(e.hash, e.key, e.val,
                                              null, null);
                        if ((p.prev = tl) == null)
                            hd = p;
                        else
                            tl.next = p;
                        tl = p;
                    }
                    setTabAt(tab, index, new TreeBin<K,V>(hd));
                }
            }
        }
    }
}
```

扩容

