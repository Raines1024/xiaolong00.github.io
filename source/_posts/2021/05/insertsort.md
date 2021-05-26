---
title: 插入排序算法介绍
date: 2021-05-21 20:17:39
description: Java实现常用算法（三）
tags: [编程, 过去, 算法]
category:
    - 100 学习类
    - 110 编程
    - 111 Java





---



## 插入排序算法

插入排序的基本思路是将一个数据插入已经排好序的序列中，从而得到一个新的有序数据，该算法适用于少量数据的排序，是稳定的排序算法。

## show code

### 插入排序算法Java实现

```java
/**
 * 升序插入排序
 * @param arr 待排序数组
 * @return 升序排序数组
 */
public static int[] insertSort(int[] arr){
    for (int i = 1; i < arr.length; i++) {
        //待插入的数
        int insertVal = arr[i];
        //待插入的位置（准备和前一个数进行比较）
        int index = i-1;
        //从数组中找到比待插入数据大的数据的索引位置index，然后将该index位置后的元素向后移动，接着将待插入的数据插入index+1的位置，如此重复，直到整个数组排序完成
        //如果已存在的数比待插入的数小
        while (index>=0 && insertVal<arr[index]){
            //则将已存在的数向后移动
            arr[index+1] = arr[index];
            //将index向前移动
            index--;
        }
        //将插入的数放入合适的位置
        arr[index+1] = insertVal;
    }
    return arr;
}
```

### 测试

```java
public static void main(String[] args) {
    int[] array = new int[5];
    array[0] = 6;
    array[1] = 2;
    array[2] = 5;
    array[3] = 8;
    array[4] = 7;
    System.out.println("排序前"+ JSON.toJSONString(array));
    System.out.println("排序后"+JSON.toJSONString(insertSort(array)));
}
```

### 结果

```
排序前[6,2,5,8,7]
排序后[2,5,6,7,8]
```

