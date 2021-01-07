---
layout: post
title: Java中对象占用内存大小计算
description: Java中对象占用内存大小计算
category: blog
---

## 回顾

### byte与bit

#### 1个字节占8位

#### bit：位，比特。信息的最小单位，二进制数中的一个位数(二进制位)，其值为“0”或“1”；

#### byte: 字节。计算机文件大小的基本计算单位
- 原码：正数本身，负数，正数对应的二进制最高位为1(负)。  
- 反码：正数本身，负数，符号位不变，其余各位取反。  
- 补码：正数本身，负数：反码+1。  
- 0的反码、补码都为零。  

### 注意:Java中的char占两个字节，而c语言中char占一个字节

## Java中对象占用内存大小

### 对象的组成

#### Java对象实例、数组实例
- 对象头(Header)    
用于存储对象自身的运行时数据，如哈希码（HashCode）、GC分代年龄、锁状态标志、线程持有的锁、偏向线程ID、偏向时间戳等，这部分数据的长度在32位和64位的虚拟机（未开启压缩指针）中分别为32bit和64bit，官方称它为"Mark Word”。  
- 实例数据(Instance Data)  
对象真正存储的有效信息，也是在程序代码中所定义的各种类型的字段内容。无论是从父类继承下来的，还是在子类中定义的，都需要记录起来。  
- 对齐填充(Padding)  
第三部分对齐填充并不是必然存在的，也没有特别的含义，它仅仅起着占位符的作用。由于HotSpot VM的自动内存管理系统要求对象起始地址必须是8字节的整数倍，换句话说，就是对象的大小必须是8字节的整数倍。而对象头部分正好是8字节的倍数（1倍或者2倍），因此，当对象实例数据部分没有对齐时，就需要通过对齐填充来补全。  
HotSpot的对齐方式为8字节对齐：  


    (Header + Instance Data + Paddding) % 8 = 0 并且 0 <= padding < 8

#### 具体大小

    名称(单位byte)	32位	64位	开启指针压缩后(指针对64位有效且默认开启)
    对象头(Header)	8	16	12
    数组对象头	12	24	16
    引用(reference)	4	8	4

- 开启指针压缩指令-XX:+UseCompressedOops,关闭指令-XX:-UseCompressedOops,只在64位才有效且默认开启；  
- 数组对象头比普通对象多了个数组长度；  

## 使用Jprofile实战

### 环境
JDK：

    java version "1.8.0_201"
    Java(TM) SE Runtime Environment (build 1.8.0_201-b09)
    Java HotSpot(TM) 64-Bit Server VM (build 25.201-b09, mixed mode)
    
Jprofile11

### 名词解释
- Shallow Size  
对象自身占用的内存大小，不包括它引用的对象。  
    - 针对非数组类型的对象，它的大小就是对象与它所有的成员变量大小的总和。当然这里面还会包括一些java语言特性的数据存储单元。  
    - 针对数组类型的对象，它的大小是数组元素对象的大小总和。  

- Retained Size  
Retained Size=当前对象大小+当前对象可直接或间接引用到的对象的大小总和。(间接引用的含义：A->B->C, C就是间接引用)

### 查看对象内存占用
Heap Walker -> references

### 例子

#### 一

    public class A {
    	private int i;
    
    	public static void main(String[] args) throws InterruptedException {
    		A a = new A();
    		Thread.sleep(1000 * 1000);
    		System.out.println(a);
    	}
    }

分析: 64位下默认开启指针压缩，对象头位12byte, i4byte,此时12 + 4 = 16 可以整除8，所以padding=0,最终  
12(header) + 4(instance data)+0(padding)=16byte  
如果我们关掉指针压缩   
16(header) + 4(instance data)+4(padding)=24byte   
16+4=20，不能整除8，需要再加上4  

#### 二

    public class B {
       private int i = 5;
       private Integer ii = 128;
    
       public static void main(String[] args) throws InterruptedException {
          B b = new B();
          Thread.sleep(1000 * 1000);
          System.out.println(b);
       }
    }

开启指针压缩，计算内存大小  
Shallow Size： 12(B Header) + 4 (i instance) + 4 (ii reference) + 4(padding) = 24bytes  
Retained Size: 12(B Header) + 4 (i instance) + 4 (ii reference) + (12(ii header) + 4(instance)+ 0(padding)) + 4(padding) = 40bytes  

#### 三

    public class C {
       private int i;
       private char[] cc;
    
       public C() {
          i = 5;
          cc = new char[]{'a', 'b', 'c'};
       }
    
       public static void main(String[] args) throws InterruptedException {
          C c = new C();
          Thread.sleep(1000 * 1000);
          System.out.println(c);
       }
    }

多了数组，注意数组自己本身的padding  
Shallow Size： 12(C Header) + 4 (i instance) + 4 (cc reference) + 4(padding) = 24bytes  
Retained Size: 12(C Header) + 4 (i instance) + 4 (cc reference) + (16(cc header) + 2(instance) * 3+ 2(padding)) + 4(padding) = 48bytes  





























