---
title: Linux系统基本命令
date: 2021-03-20 19:01:39
description: CentOS 7.6操作系统
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 112 Linux
---

## 命令别名的使用

命令别名是把一个命令名称定义成另一个名称，在使用时，可以使用命令本身，也可以使用命令的别名。

命令别名是Shell的特性，只在当前终端生效。命令别名具体概念不再赘述，需要的转移搜索引擎，该博文面向实用开发，只说笔者自认为的常用用法。

### 设置命令别名永久生效

- 设置当前用户命令别名永久生效

  ```sh
  [root@localhost ~]# vim /root/.bashrc 			#插入以下内容
  alias cls='clear'
  [root@localhost ~]# source /root/.bashrc 		#重新加载该文件
  [root@localhost ~]# cls
  ```

  

- 设置系统所有用户命令别名永久生效

  ```sh
  [root@localhost ~]# vim /etc/bashrc 				#在文件最后插入
  alias cls='clear'
  [root@localhost ~]# su star									#切换到普通用户
  [root@localhost ~]# cls
  ```

- 个人定义

  修改`~/.bashrc`,在文件中加入`alias cls='clear'`即可。

































