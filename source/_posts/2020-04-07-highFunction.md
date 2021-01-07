---
layout: post
title: 用高阶函数做抽象
description: 主要参考书籍为《计算机程序的构造和解释》，改用Java语言重写
category: blog
---

## 以计算数值为例

### 定义过程计算数值
定义过程，做某些行为的封装

```
public class SumCount {

    /**
     * 计算从a到b各整数之和
     */
    public int sumIntegers(int a, int b) {
        if (a > b) return 0;
        return a + sumIntegers(a + 1, b);
    }

    /**
     * 计算给定范围内(a-b)整数的立方之和
     */
    public int sumCubes(int a, int b) {
        if (a > b) return 0;
        return cube(a) + sumCubes(a + 1, b);
    }

    public double piSum(int a, int b) {
        if (a > b) return 0;
        return 1.0 / (a * (a + 2)) + piSum(a + 4, b);
    }

    /**
     * 计算x的立方
     */
    private int cube(int x) {
        return x * x * x;
    }

}
```

### 用高阶函数做抽象计算数值（过程作为参数或过程作为返回值）
将过程作为参数传递，能够显著增强我们的程序语言的表达能力。  
sum方法：为公共的模式命名，建立抽象，而后直接在抽象的层次上进行工作。    
以过程作为参数，或者以过程作为返回值，这类能操作过程的过程称为高阶过程。  

```
public class NewSumCount {
    
    private static int sum(Method term, Integer a, Method next, Integer b) throws InvocationTargetException, IllegalAccessException {
        if (a > b) return 0;
        return (Integer) term.invoke(Integer.class, a) + sum(term, (Integer) next.invoke(Integer.class, a), next, b);
    }

    /**
     * 计算从a到b各整数之和
     */
    public static int sumIntegers(int a, int b) throws Exception {
        return sum(NewSumCount.class.getDeclaredMethod("identity", Integer.class), a, NewSumCount.class.getDeclaredMethod("inc", Integer.class), b);
    }

    /**
     * 计算从a到b各整数的立方和
     */
    public static int sumCubes(int a, int b) throws Exception {
        return sum(NewSumCount.class.getDeclaredMethod("cube", Integer.class), a, NewSumCount.class.getDeclaredMethod("inc", Integer.class), b);
    }

    public static void main(String[] args) throws Exception {
        System.out.println(sumIntegers(1, 10));
        System.out.println(sumCubes(1, 10));
    }

    /**
     * 计算x的立方
     */
    private static int cube(Integer x) {
        return x * x * x;
    }

    private static int identity(Integer x) {
        return x;
    }

    private static int inc(Integer a) {
        return a + 1;
    }
    
}
```




















































