title: Linux下Redis安装与使用简介
date: 2021-04-02 19:01:39
description: CentOS7安装Redis6.0
tags: [编程, 过去]
category:

    - 100 学习类
    - 110 编程
    - 112 Linux

## 软件及环境
1. CentOS7
2. [redis-6.0.8.tar.gz](https://redis.io/download)

## redis安装
1. 预先安装gcc和make
避免待会儿make时由于没安装gcc失败，提前使用`yum install -y gcc make`安装gcc
注：可通过`whereis gcc make`检查软件是否已安装

2. 在上传(或下载)redis的目录下进行解压
- 创建redis目录
  mkdir /usr/local/redis 