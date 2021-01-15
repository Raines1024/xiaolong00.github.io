---
layout: post
title: HDFS常用Shell操作
description: 
category: blog
date: 2020-01-07 13:50:39
---

## HDFS常用Shell操作

### 列出文件目录：hadoop fs -ls 目录路径
- 查看HDFS根目录下的目录：hadoop fs -ls /
- 递归查看HDFS根目录下的目录：hadoop fs -lsr /

###  在HDFS中创建文件夹：hadoop fs -mkdir 文件夹名称

### 上传文件到HDFS中：hadoop fs -put 本地源路径 目标存放路径
将本地系统中的一个log文件上传到di文件夹中：hadoop fs -put test.log /di

### 从HDFS中下载文件：hadoop fs -get HDFS文件路径 本地存放路径
将刚刚上传的test.log下载到本地的Desktop文件夹中：hadoop fs -get /di/test.log /home/hadoop/Desktop

### 直接在HDFS中查看某个文件：hadoop fs -text(-cat) 文件存放路径
在HDFS查看刚刚上传的test.log文件：hadoop fs -text /di/test.log

### 删除在HDFS中的某个文件(夹)：hadoop fs -rm(r) 文件存放路径

### 善用help命令求帮助：hadoop fs -help 命令
查看ls命令的帮助：hadoop fs -help ls



