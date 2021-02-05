---
title: MySQL备份恢复单个innodb表
date: 2021-02-02 10:10:39
description: 
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 113 数据库
---

## 背景

之前用过mysql通过备份数据目录来实现数据库全量迁移，非常方便快速。当时尝试恢复单个表而没有成功，此时再次尝试终于如愿以偿。

本文将要说说怎么移动或复制部分或全部的表到另一台服务器上，而所要用到的技术点就是transportable tablespace特性，这就意味着MySQL5.6.6以及以上版本才支持。

表空间传输特性允许表空间从一个实例移动到另一个实例上。这在以前版本上，这对InnoDB表空间是不可能的，因为所有的表数据都是系统表空间的一部分。

## 实现

1. 将拷贝的数据文件  "qqq.idb"放在自己的数据目录下的数据库中
2. "qqq.idb" 改个名字-->"qqq--.idb", 主要是避免冲突
3. 执行 create table qqq(...) 语句，此时除了会生成一个  qqq.frm, 文件，还会新生成一个qqq.idb文件
4. 执行 ALTER TABLE qqq DISCARD TABLESPACE; 会自动删除 新生成的qqq.idb 文件
5. 改回 "qqq--.idb"文件名为 "qqq.idb"
6. 赋给qqq.idb权限：chown mysql:mysql -R qqq.idb
7. ALTER TABLE qqq IMPORT TABLESPACE; SHOW WARNINGS;

恢复完成！

如果import tablespace 的时候，报错 ibd文件与表的 ROW_TYPE_COMPACT 不兼容，则需要在建表语句最后 加上 ROW_FORMAT=COMPACT保持一致！



## 参考链接

[另一种思路恢复单个innodb表](http://www.ttlsa.com/mysql/mysql-backup-recovery-innodb-table/)





































