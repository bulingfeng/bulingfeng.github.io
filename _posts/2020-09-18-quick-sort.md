---
title: 快速排序
tags: 算法 排序
categories: 算法
---

* TOC
{:toc}


## 简介

```
常见的排序有冒泡排序，插入排序，快速排序等。其中冒泡排序和插入排序的时间复杂度为O(n2)。而快速排序的时间复杂度为O(nlogn)。所以一般来说快速排序的效率会比冒泡和插入效率高（如何判断效率高低，只能计算n2=nlogn的交点）。
```

参考文章

```
该文章描写的挖坑然后填坑的方式，个人感觉还是比较形象并且比较容易理解的。
https://www.runoob.com/w3cnote/quick-sort.html
```

## 代码实现

```
package com.blf.book.sort;

import java.util.Arrays;

/**
 * @Author:bulingfeng
 * @Date: 2020-09-18
 * 快速排序
 * @link{参考文章:https://www.runoob.com/w3cnote/quick-sort.html}
 */
public class QuickSort {
    public static void main(String[] args) {
        // 使用参考文章中的方法，就是填坑的方式来做快速排序。并且每次都是执行一样的逻辑，所以采用递归的方式来实现。
        int[] arrays={1,3,5,9,8,7};
        actionSort(arrays,0,arrays.length-1);
        Arrays.stream(arrays).forEach(System.out::println);
    }

    /**
     *  1、左边放比基准值小的数，从左边找比基准值大的数字放到右侧。
     *  2、右边放比基准值大的数，从右边找到比基准值小的数字放到左侧。
     * @param array
     * @param start
     * @param end 就是最后基准数据所在数组的位置
     *
     */
    public static int sort(int[] array,int start,int end){
        int baseNum=array[start];
        int i=start;
        int j=end;
        while (i<j){
            // 开始从右边寻找到比基准值小的数字 然后放到基准值左侧
            while (i<j && array[j]>=baseNum)
                j--;
            // 如果不符合上面的while条件的话就会执行一下操作
            if (i<j){
                array[i]=array[j];
                // i++的目的就是为了下面的循环来判断是否该下标写的值大于基准值
                i++;
            }


            // 从左边开始寻找到比基准值大的数字 然后放到右侧
            while (i<j && array[i]<baseNum)
                i++;
            if (i<j){
                array[j]=array[i];
                // j--的目的是为了上个循环使用，判断是否j下标的值是否小于基准值
                j--;
            }
        }

        // 当x=j的时候开始退出
        array[i]=baseNum;
        return i;
    }


   public static void actionSort(int s[], int l, int r)
    {
        if (l < r)
        {
            // 给每个数组段的数据已基准值为标准，大于基准值的放到右边，小于基准值的放到左边。
            // 并且拿到基准值的位置
            int i = sort(s, l, r);
            actionSort(s, l, i - 1);
            actionSort(s, i + 1, r);
        }
    }
}
```

## 总结

```
快速排序相比冒泡还是比较复杂的，其实其主要的思想有两点，抓住这两点的话再次编写是非常容易的。
1、左边比基准值小，右边为基准点大。每次基准的小标为数组的中间值即可。
2、考虑到每次执行的逻辑是一样的，所以可以使用递归的方式来解决。
```

