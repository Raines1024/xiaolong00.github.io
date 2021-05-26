---
title: 快速排序算法介绍
date: 2021-05-21 20:27:39
description: Java实现常用算法（四）
tags: [编程, 过去, 算法]
category:
    - 100 学习类
    - 110 编程
    - 111 Java


---



## 快速排序算法

### 简介

快速排序是对冒泡排序的一种改进，通过一趟排序将要排序的数据序列分成独立的俩部分，其中一部分的所有数据比另一部分的所有数据都要小，然后按此方法对两部分数据分别进行快速排序，整个排序过程递归进行，最终使整个数据序列变成有序的数据序列。

### 原理

选择一个关键值作为基准值（一般选择第1个元素为基准元素），将比基准值大的都放在右边的序列中，将比基准值小的都放在左边的序列中。

### 循环过程

1. 从后向前比较，用基准值和做后一个值进行比较。如果比基准值小，则交换位置；如果比基准值大，则继续比较下一个值，直到找到第1个比基准值小的值才交换位置。

2. 在从后向前找到第1个比基准值小的值并交换位置后，从前向后开始比较。如果有比基准值大的，则交换位置；如果没有，则继续比较下一个，直到找到第1个比基准值大的值才交换位置。

3. 重复执行以上过程，直到从前向后比较的索引大于等于从后向前比较的索引，则结束一次循环。这时对于基准值来说，左右两边都是有序的数据序列。

4. 重复循环以上过程，分别比较左右两边的序列，直到整个数据序列有序。

## show code

### 快速排序算法Java实现

```java
public static int[] quickSort(int[] arr, int low, int high) {
    //从前向后比较的索引
    int start = low;
    //从后向前比较的索引
    int end = high;
    //基准值
    int key = arr[low];
    while (end > start) {
        //从后向前比较
        while (end > start && arr[end] >= key) {
            end--;
        }
        //如果没有比基准值小的，则比较下一个，直到有比基准值小的，则交换位置，然后又从前向后比较
        if (arr[end] <= key) {
            int temp = arr[end];
            arr[end] = arr[start];
            arr[start] = temp;
        }
        //从前向后比较
        while (end > start && arr[start] <= key) {
            start++;
        }

        //如果没有比基准值大的，则比较下一个，直到有比基准值大的，则交换位置
        if (arr[start] >= key) {
            int temp = arr[start];
            arr[start] = arr[end];
            arr[end] = temp;
        }
        //此时第1此循环比较结束，基准值的位置已经确定，左边的值都比关键值小，右边的值逗比关键值大，但是两边的顺序还有可能不一样，接着进行下面的递归调用
    }
    if (start > low) quickSort(arr, low, start - 1);
    if (end < high) quickSort(arr, end + 1, high);
    return arr;
}
```

### 测试

```java
public static void main(String[] args) {
    int[] array = new int[5];
    array[0] = 6;
    array[1] = 9;
    array[2] = 5;
    array[3] = 7;
    array[4] = 8;
    System.out.println("排序前" + JSON.toJSONString(array));
    System.out.println("排序后"+JSON.toJSONString(quickSort(array,0,array.length-1)));
}
```

