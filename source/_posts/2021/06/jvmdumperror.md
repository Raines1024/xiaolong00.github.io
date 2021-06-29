---
title: JVM报错：Failed to write core dump. Core dumps have been disabled.
date: 2021-06-28 20:27:39
description: 
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java



---



## 问题

### 报错信息

宕机后报错信息如下

```log
#
# A fatal error has been detected by the Java Runtime Environment:
#
#  SIGSEGV (0xb) at pc=0x00002b1fd6f01840, pid=27918, tid=0x00002b20b1716700
#
# JRE version: OpenJDK Runtime Environment (8.0_275-b01) (build 1.8.0_275-b01)
# Java VM: OpenJDK 64-Bit Server VM (25.275-b01 mixed mode linux-amd64 compressed oops)
# Problematic frame:
# C  [libcpplib.so+0x1a7840]  calculateDistance(Point const&, Point const&)+0x1c
#
# Failed to write core dump. Core dumps have been disabled. To enable core dumping, try "ulimit -c unlimited" before starting Java again
#
# An error report file with more information is saved as:
# /var/lib/jenkins/workspace/farm-develop/logs/hs_err_pid27918.log
#
# If you would like to submit a bug report, please visit:
#   https://bugzilla.redhat.com/enter_bug.cgi?product=Red%20Hat%20Enterprise%20Linux%207&component=java-1.8.0-openjdk
# The crash happened outside the Java Virtual Machine in native code.
# See problematic frame for where to report the bug.
```

报错信息提示尝试使用`ulimit -c unlimited`解决该问题，但是这个命令并不能解决问题

## 解决方案

问题的根本原因在于服务器的运行应用程序的打开文件的最大数及最大进程数设置的相对较小默认为4096

需要修改如下配置：

/etc/security/limits.conf

```
* soft nofile 327680
* hard nofile 327680
hdfs soft nproc 131072
hdfs hard nproc 131072
mapred soft nproc 131072
mapred hard nproc 131072
hbase soft nproc 131072
hbase hard nproc 131072
zookeeper soft nproc 131072
zookeeper hard nproc 131072
hive soft nproc 131072
hive hard nproc 131072
root soft nproc 131072
root hard nproc 131072
```



## 参考链接

[在高并发的大数据 场景下Linux服务器报错fork: retry:资源暂时不可用的解决办法](https://www.cnblogs.com/songyuejie/p/11221381.html)