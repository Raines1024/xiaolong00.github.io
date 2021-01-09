---
layout: post
title: 递归介绍(一）：线性递归、线性迭代、树形递归
description: 主要参考书籍为《计算机程序的构造和解释》，改用Java语言重写
category: blog
date: 2020-01-07 13:50:39
---

## 以计算阶乘(n!)为例解释线性递归和线性迭代

### 线性递归

#### 思路：对于一个正整数n，n!就等于 n*(n-1)!
代码：  

```
    public static int fuctorial(int n){
        if (n == 1){
            return 1;
        }else {
            return n * fuctorial(n-1);
        }
    }
```

3!的线性递归过程,利用代换模型观看这一过程在计算3!时表现出的行为（代换模型揭示出一种先逐步展开而后收缩的形状）。
在展开阶段里，这一计算过程构造起一个推迟进行的操作所形成的链条（在这里是一个乘法的链条），收缩阶段表现为这些运算的实际执行。
这种类型的计算过程由一个推迟执行的运算链条刻画，称为一个递归计算过程。  
要执行这种计算过程，解释器就需要维护好那些以后将要执行的操作的轨迹。
在计算阶乘n!时，推迟执行的乘法链条的长度也就是为保存其轨迹需要保存的信息量，这个长度随着n值而线性增长（正比于n），
就像计算中的步骤数目一样。这样的计算过程称为一个线性递归过程。  

```
3!的执行过程：
    fuctorial(3)
    3 * fuctorial(2)
    3 * 2 * fuctorial(1)
    3 * 2 * 1
    3 * 2
    6
```

### 线性迭代

#### 思路：先乘以1和2，而后一直乘到n

计算过程里并没有任何增长或者收缩，对于任何一个n，在计算过程中的每一步，在我们需要保存轨迹里，所有的东西就是变量product、counter和maxCount的当前值。这种过程为一个迭代计算过程。  
一般来说，迭代计算过程就是那种状态可以用固定数目的状态变量描述的计算过程；而与此同时，又存在着一套固定的规则，描述了计算过程在从一个状态到下一状态转换时，这些变量的更新方式；还有一个（可能有的）结果检测，它描述这一计算过程应该终止的条件。在计算n!时，所需的计算步骤随着n线性增长，这种过程称为线性迭代过程。
计算3!的线性迭代过程  

```
factiorial(3)
factIter(1,1,3)
factIter(1,2,3)
factIter(2,3,3)
factIter(6,4,3)
```
代码：  

```
public class Iteration {

    public static int factiorial(int n){
        return factIter(1,1,n);
    }

    private static int factIter(int product, int counter, int maxCount){
        if (counter > maxCount){
            return product;
        }else {
            return factIter(counter*product,++counter,maxCount);
        }
    }

    public static void main(String[] args) {
        System.out.println(factiorial(3));
    }
}

```

### 递归计算过程和递归过程的区别
当我们说一个过程是递归的时候，论述的是一个语法形式上的事实，说明这个过程的定义中（直接或间接的）引用了该过程本身。再说某一计算过程具有某种格式时（例如线性递归），我们说的是这一计算过程的进展方式，而不是相应过程书写上的语法形式。  
例如上个例子中说到的线性迭代，我们说这个递归过程将产生出一个迭代的计算过程，因为它的状态能由其中的三个状态变量完全刻画，解释器在执行这一计算过程时，只需要保持这三个变量的轨迹就足够了。  

## 以斐波那契数列为例解释计算模式--树形递归

### 代码  

```
public class Fibonacci {
    public static int fib(int n){
        if (n == 0){
            return 0;
        }else if (n == 1){
            return 1;
        }else {
            return fib(n-1)+fib(n-2);
        }
    }
}
```

### 计算fib(5)中的产生的树型递归计算过程

```
                    fib(5)
     fib(4)          +            flib(3)
fib(3)     fib(2)    +        fib(2)fib(1)
fib(2)+1  +  1+0     +          1+0  +   1
1+0+   1 +   1+0     +          1+0  +   1
5
```

### 注意
树形递归计算模式的每层分裂为两个分支（除了最下面），反映出对fib过程的每个调用中两次递归调用自身的事实。  
fib(n)的增长相对于n是指数级的，该过程所用的计算步骤数将随着输入增长而指数性地增长。  

### 线性迭代实现
规划出计算斐波那契数列的迭代计算过程，其基本想法就是用一对整数a和b，将它们分别初始化为flib(1)=1

```
public class IterFibonacci {

    public static int fib(int n) {
        return fibIter(1, 0, n);
    }

    public static int fibIter(int a, int b, int count) {
        if (count == 0) {
            return b;
        } else {
            return fibIter(a + b, a, --count);
        }
    }

}

```

### 树形递归和线性迭代实现比较
这两种方法在计算中所需的步骤差异巨大--线性迭代方法相对于n为线性的，树型递归实现增长像fib(n)一样快，即使不大的输入也可能造成很大的差异。   
当我们考虑的是在层次结构性的数据上操作，而不是对数操作时，将会发现树形递归计算过程是一种自然的、威力强大的工具。  
以以斐波那契数列为例，虽然树形递归计算过程远比线性迭代低效，但它更直截了当。而要规划出迭代过程，则需要注意到，这一计算过程可以重新塑造为一个采用三个状态变量的迭代。  





































