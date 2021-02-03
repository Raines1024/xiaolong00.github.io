---
title: MySQL查看连接数，设置最大连接数
date: 2021-02-02 10:00:39
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 113 数据库
---

## 背景

测试mysql服务突然报连接数太多。

## 解决

- 查看mysql连接

  命令： show processlist;显示当前正在执行的mysql连接
  **如果是root帐号，你能看到所有用户的当前连接。如果是其它普通帐号，只能看到自己占用的连接。** 
  **show processlist;**只列出前100条，如果想全列出请使用**show full processlist;** 

- show variables like '%max_connections%'; 查看最大连接数

- 设置最大连接数的方法：

  set global max_connections=1000 重新设置最大连接数

  在/etc/my.cnf设置，[mysqld]下添加global max_connections=1000，重启mysql服务

## 参考链接

[另一种思路恢复单个innodb表](http://www.ttlsa.com/mysql/mysql-backup-recovery-innodb-table/)





































