---
layout: post
title: JVM相关
description: 乱七八糟
category: blog
date: 2020-01-07 13:50:39
---

## JVM相关

### 打印GC相关的信息并分析
JVM提供了大量命令行参数,打印信息,供调试使用.主要有以下一些:   
java -verbose:gc hash  或  java -XX:+PrintGC hash  
输出形式:  
[GC 118250K->113543K(130112K), 0.0094143 secs]   
[Full GC 121376K->10414K(130112K), 0.0650971 secs]  

-XX:+PrintGCDetails  
输出形式:  
[GC [DefNew: 8614K->781K(9088K), 0.0123035 secs] 118250K->113543K(130112K), 0.0124633 secs]   
[GC [DefNew: 8614K->8614K(9088K), 0.0000665 secs][Tenured: 112761K->10414K(121024K), 0.0433488 secs] 121376K->10414K(130112K), 0.0436268 secs]  

以其中一行为例来解读下日志信息：  
[GC (Allocation Failure) [ParNew: 367523K->1293K(410432K), 0.0023988 secs] 522739K->156516K(1322496K), 0.0025301 secs] [Times: user=0.04 sys=0.00, real=0.01 secs]  

- GC：  
表明进行了一次垃圾回收，前面没有Full修饰，表明这是一次Minor GC ,注意它不表示只GC新生代，并且现有的不管是新生代还是老年代都会STW。  
- Allocation Failure：  
表明本次引起GC的原因是因为在年轻代中没有足够的空间能够存储新的数据了。   
- ParNew：  
表明本次GC发生在年轻代并且使用的是ParNew垃圾收集器。ParNew是一个Serial收集器的多线程版本，会使用多个CPU和线程完成垃圾收集工作（默认使用的线程数和CPU数相同，可以使用-XX：ParallelGCThreads参数限制）。该收集器采用复制算法回收内存，期间会停止其他工作线程，即Stop The World。  
- 367523K->1293K(410432K)：  
单位是KB  

三个参数分别为：GC前该内存区域(这里是年轻代)使用容量，GC后该内存区域使用容量，该内存区域总容量。  
- 0.0023988 secs：  
该内存区域GC耗时，单位是秒  
- 522739K->156516K(1322496K)：  
三个参数分别为：堆区垃圾回收前的大小，堆区垃圾回收后的大小，堆区总大小。  
- 0.0025301 secs：  
该内存区域GC耗时，单位是秒  
- [Times: user=0.04 sys=0.00, real=0.01 secs]：  
分别表示用户态耗时，内核态耗时和总耗时

#### 分析下可以得出结论：
该次GC新生代减少了367523-1293=366239K  
Heap区总共减少了522739-156516=366223K   
366239 – 366223 =16K，说明该次共有16K内存从年轻代移到了老年代，可以看出来数量并不多，说明都是生命周期短的对象，只是这种对象有很多。  
我们需要的是尽量避免Full GC的发生，让对象尽可能的在年轻代就回收掉，所以这里可以稍微增加一点年轻代的大小，让那16K的数据也保存在年轻代中。  

#### GC时，用什么方法判断哪些对象是需要回收：
1. 引用计数法(已经不用了)   
简而言之就是给对象添加一个引用计数器，有其他地方引用时这个计数器+1，引用失效时-1，为0时就可以删除掉了。但是它不能解决循环引用的问题，所以一般使用的都是后一种算法。  
2. 可达性分析法  
可达性分析法的基本思路就是通过一系列名为GC Roots的对象作为起始点，从这些节点开始向下搜索，搜索所走过的路径称为引用链(Reference Chain)，当一个对象到GC Roots没有任何引用链相连时，则证明此对象是不可用的，那就可以回收掉了。  
GC Roots一般都是些堆外指向堆内的引用，例如：  
JVM栈中引用的对象   
方法区中静态属性引用的对象   
方法区中常量引用的对象  
本地方法栈中引用的对象   


参考链接：https://www.cnblogs.com/jpfss/p/8618297.html  

### java-xx参数介绍及调优总结
https://blog.csdn.net/weixin_39407066/article/details/84582380

### 实战:通过gc比较String.replace性能差异
1. 代码  
```
public class TestStringReplace {

    private static String replace(String src, String pattern, String replacement) {
        int i = src.indexOf(pattern);
        if (i >= 0) {
            int ptnLen = pattern.length();
            StringBuilder sb = new StringBuilder(src.length());
            sb.append(src, 0, i).append(replacement);
            i += ptnLen;
            for (;;) {
                int j = src.indexOf(pattern, i);
                if (j >= 0) {
                    sb.append(src, i, j).append(replacement);
                    i = j + ptnLen;
                } else {
                    return sb.append(src, i, src.length()).toString();
                }
            }
        } else {
            return src;
        }
    }

    private static final int LOOPS = 100 * 10000;
    private static final String SRC = "asdfasdfassdd2edsdfasdfasdfasdfasdfasdfasdsdfasdfasdfasdfasdfasdfasdsdfasdfasdfasdfsd";
    private static final String PATTERN = "sdf";
    private static final String REP = "ijkk2eeew;";

    public static void main(String[] args) {
        System.out.println(replace(SRC, PATTERN, REP));
        System.out.println(replace(SRC, PATTERN, REP).equals(SRC.replace(PATTERN, REP)));

        System.out.println("test replace1---------------------------------------");
        timeit(() -> {
            for (int i = 0; i < LOOPS; i++) {
                replace(SRC, PATTERN, REP);
            }
        }, 6);

        System.out.println("test string.replace-------------------------------------");
        timeit(() -> {
            for (int i = 0; i < LOOPS; i++) {
                SRC.replace(PATTERN, REP);
            }
        }, 6);
    }

    private static void timeit(Runnable r, int times) {
        long total = 0;
        for (int i = 0; i < times; i++) {
            long start = System.currentTimeMillis();
            r.run();
            long end = System.currentTimeMillis();
            total += end - start;
            System.out.println(end - start);
        }
        System.out.println("AVG " + total / times);
    }

}
```  
2. 执行命令运行TestStringReplace  
time java -XX:+UseG1GC -Xmx4G -Xms4G -XX:+PrintGC TestStringReplace  
```
true
test replace1---------------------------------------
[GC pause (G1 Evacuation Pause) (young) 204M->608K(4096M), 0.0016481 secs]
[GC pause (G1 Evacuation Pause) (young) 236M->709K(4096M), 0.0017248 secs]
1333
[GC pause (G1 Evacuation Pause) (young) 2454M->750K(4096M), 0.0036411 secs]
1239
688
[GC pause (G1 Evacuation Pause) (young) 2454M->684K(4096M), 0.0019600 secs]
593
[GC pause (G1 Evacuation Pause) (young) 2454M->666K(4096M), 0.0023456 secs]
579
572
AVG 834
test string.replace-------------------------------------
[GC pause (G1 Evacuation Pause) (young) 2454M->662K(4096M), 0.0020937 secs]
[GC pause (G1 Evacuation Pause) (young) 2454M->689K(4096M), 0.0025844 secs]
1857
[GC pause (G1 Evacuation Pause) (young) 2454M->729K(4096M), 0.0022971 secs]
1789
[GC pause (G1 Evacuation Pause) (young) 2454M->708K(4096M), 0.0021608 secs]
1792
[GC pause (G1 Evacuation Pause) (young) 2454M->710K(4096M), 0.0023275 secs]
1781
[GC pause (G1 Evacuation Pause) (young) 2454M->661K(4096M), 0.0022797 secs]
1809
[GC pause (G1 Evacuation Pause) (young) 2454M->662K(4096M), 0.0020625 secs]
[GC pause (G1 Evacuation Pause) (young) 2454M->778K(4096M), 0.0023225 secs]
1787
AVG 1802
java -XX:+UseG1GC -Xmx4G -Xms4G -XX:+PrintGC TestStringReplace  15.63s user 1.02s system 102% cpu 16.233 total
```
总结：主要是节省了官方实现通过Regex这一步，虽然性能差距不大，但是这并非是对性能的吹毛求疵，而是一眼看过去的直觉就感觉不是很舒服。  









































































