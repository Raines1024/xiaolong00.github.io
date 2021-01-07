---
layout: post
title: 递归介绍(二）：换零钱方式的统计--树形递归
description: 主要参考书籍为《计算机程序的构造和解释》，改用Java语言重写
category: blog
---

## 实例：换零钱方式的统计  

### 问题
给了50美分、25美分、10美分、5美分、1美分的硬币，将1美元换成零钱，一共有多少种不同方式？给定了任意数量的现金，写一个程序，计算出所有换零钱方式的种数。  

### 思考过程
假定我们所考虑的可用硬币类型种类排了某种顺序，于是有了下面的关系：  
将总数为a的现金换成n种硬币的不同方法的数目等于：  

> 将现金数a换成除第一种硬币之外的所有其他硬币的不同方式数目，加上  将现金数a-d换成所有种类的硬币的不同方式数目，其中的d是第一种硬币的币值。  

显然，换成零钱的全部方式的数目，就等于完全不用第一种硬币的方式的数目，加上用了第一种硬币的换零钱方式的数目。而后一个数目也就等于去掉一个第一种硬币值后，剩下的现金数的换零钱方式数目。  
这样就把将某个给定现金数的换零钱的方式的问题，递归的规约为对更少现金数或者更少种类硬币的同一个问题。利用上面的规则写出一个算法来：    

> 如果a就是0，应该算作是有1种换零钱的方式。  
如果a小于0，应该算作是有0种换零钱的方式。  
如果n是0，应该算作是有0种换零钱的方式。  


### 根据书中scheme代码重写为Java

```
/**
 * 换零钱方式的统计
 * @author raines
 */
public class SwapMoneyCount {

    private int countChange(int amount) {
        return cc(amount, 5);
    }

    /**
     * 计算有多少种换零钱的方式
     *
     * @param amount       钱数
     * @param kindsOfCoins 可用的硬币种数
     * @return 换零钱方式的总数
     */
    private int cc(int amount, int kindsOfCoins) {
        if (amount == 0) {
            return 1;
        } else if (amount < 0 || kindsOfCoins == 0) {
            return 0;
        } else {
            return cc(amount, kindsOfCoins - 1) + cc(amount - firstDenomination(kindsOfCoins), kindsOfCoins);
        }
    }

    /**
     * 以可用的硬币种数作为输入，返回第一种硬币的币值，默认硬币已经从最大到最小排序好了
     * 1美元 = 100美分
     * 硬币种数：50、25、10、5、1美分
     *
     * @param kindsOfCoins 可用的硬币种数
     * @return 每种硬币的面值
     */
    private int firstDenomination(int kindsOfCoins) {
        if (kindsOfCoins == 1) {
            return 1;
        } else if (kindsOfCoins == 2) {
            return 5;
        } else if (kindsOfCoins == 3) {
            return 10;
        } else if (kindsOfCoins == 4) {
            return 25;
        } else if (kindsOfCoins == 5) {
            return 50;
        }
        return 0;
    }

}
```

























































