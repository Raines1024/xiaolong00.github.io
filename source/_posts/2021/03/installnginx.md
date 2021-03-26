---
title: Centos安装nginx
date: 2021-03-26 20:01:39
description: 源码安装nginx
tags: [编程,过去]
category:
    - 100 学习类
    - 110 编程
    - 112 Linux

---

## 源码安装nginx

1. 部署nginx

   ```sh
   [root@flink1 ~]# yum -y install gcc gcc-c++ make zlib-devel pcre pcre-devel openssl-devel
   ```

2. 下载并解压nginx程序压缩包

   [nginx官网下载地址](http://nginx.org/en/download.html)

   ```sh
   [root@flink1 ~]# rz			#上传nginx压缩包（或使用wget命令下载）
   [root@flink1 ~]# tar xvf nginx-1.18.0.tar.gz
   [root@flink1 ~]# cd nginx-1.18.0
   ```

3. 开始安装

   执行预编译（检测环境）并指定安装目录，使用`--prefix=`指定

   安装nginx预编译时指定安装目录为/usr/local/nginx

   configure命令最终在当前目录下生成Makefile文件

   ```sh
   [root@flink1 nginx-1.18.0]# ./configure --prefix=/usr/local/nginx
   [root@flink1 nginx-1.18.0]# make -j 4		#按Makefile文件编译，-j 4表示指定4核心CPU同时编译，提升速度
   [root@flink1 nginx-1.18.0]# make install		#按Makefile顶一的文件路径安装
   ```

4. 运行nginx测试

   ```sh
   [root@flink1 ~]# /usr/local/nginx/sbin/nginx
   ```

   查看nginx端口占用

   ```sh
   [root@flink1 sbin]# netstat -lntup |grep nginx
   tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      1285/nginx: master 
   ```

   









