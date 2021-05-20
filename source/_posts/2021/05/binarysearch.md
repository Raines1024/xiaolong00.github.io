---
title: 二分查找算法介绍
date: 2021-05-20 20:17:39
description: Java实现常用算法（一）
tags: [编程, 过去, 算法]
category:
    - 100 学习类
    - 110 编程
    - 111 Java



---



## 二分查找算法

二分查找又叫作折半查找，二分查找算法要求要查找的序列是有序的，每次查找都取中间位置的值与待查关键字进行比较，如果中间位置的值比待查关键字大，则在序列的左半部分继续执行该查找过程，如果中间位置的值比待查关键字小，则在序列的右半部分继续执行该查找过程，直到查找到关键字为止，否则在序列中没有待查关键字。

## show code

### 二分查找算法Java实现

```java
/**
 * 二分查找
 * @param array 源数组
 * @param a 目标值
 * @return 目标值在数组中索引位置，未找到返回-1
 */
public static int binarySearch(int[] array, int a) {
    // 最小数据索引
    int low = 0;
    //最大数据索引
    int high = array.length - 1;
    //中间数据索引
    int mid;
    //通过while循环在数组中查找传入的数据，在该数据大于中间位置的数据时向右查找，即最大索引位置不变，将最小索引设置为上次循环的中间索引+1；在该数据小于中间位置的数据时向左查找，即最小索引位置不变，然后将最大索引设置为上次循环的中间索引并-1。重复以上过程，直到中间索引位置的数据等于要查找的数据，则将该数据对应的索引返回。
    while (low <= high) {
        //中间位置
        mid = (low + high) / 2;
        if (array[mid] == a) {
            return mid;
        } else if (a > array[mid]) {
            //从右查找
            low = mid + 1;
        } else {
            //向左查找
            high = mid - 1;
        }
    }
    return -1;
}
```

### 测试

```java
public static void main(String[] args) {
    int[] array = new int[100];
    for (int i = 111; i < 211; i++) {
        array[i-111] = i;
    }
    System.out.println(binarySearch(array,188));
}
```

## 注意

二分查找算法要求要查找的集合是有序的，如果不是有序集合，要先通过排序算法排序后再进行查找。