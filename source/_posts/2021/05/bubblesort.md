---
title: 冒泡排序算法介绍
date: 2021-05-20 20:17:39
description: Java实现常用算法（二）
tags: [编程, 过去, 算法]
category:
    - 100 学习类
    - 110 编程
    - 111 Java




---



## 冒泡排序算法

它在重复访问要排序的元素列时，会依此比较相邻的两个元素，如果左边的元素大于右边的元素，就将二者交换位置，如此重复，直到没有相邻的元素需要交换位置，这时该列表的元素排序完成。

## show code

### 冒泡排序算法Java实现

```java
/**
 * 升序冒泡排序算法
 * @param arr 源数组
 * @return 升序排序数组
 */
public static int[] bubbleSort(int[] arr){
    //外层循环控制排序趟数
    for (int i = 0; i < arr.length - 1; i++) {
        //内层循环控制每一趟排序多少次
        for (int j = 0; j < arr.length - 1 - i; j++) {
            //比较当前数据和下一个数据的大小，如果当前数据大于下一个数据的大小，就交换二者的位置，这样重复进行判断，直至排序完成返回排序后的数组。
            if (arr[j] > arr[j+1]){
                int temp = arr[j];
                arr[j] = arr[j+1];
                arr[j+1] = temp;
            }
        }
    }
    return arr;
}
```

### 测试

```java
public static void main(String[] args) {
    int[] array = new int[10];
    for (int i = 0; i < 10; i++) {
        array[i] = -i;
    }
    System.out.println("排序前"+JSON.toJSONString(array));
    System.out.println("排序后"+JSON.toJSONString(bubbleSort(array)));
}
```

## 冷知识

冒泡排序算法名称的由来是越大的元素会经过交换慢慢“浮”到数列的顶端（升序或降序排序），就如同水的气泡最终会上浮到顶端。