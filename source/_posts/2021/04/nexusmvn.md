---
title: Nexus添加的包无法下载解决方案
date: 2021-04-03 19:01:39
description: mvn -U的用法
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java
---

## 背景

最近在本地Nexus中添加了一个外部依赖包，通过Nexus后台可以查看，通过URL也可以访问，可是本地开发环境就是说找不到。

注：上传的时候一定要勾选`Generate a POM file with these coordinates`选项

## mvn -U的用法

### mvn -U说明

-U,--update-snapshots Forces a check for missing releases
and updated snapshots on remote repositories

意思是：强制刷新本地仓库不存在release版和所有的snapshots版本。

对于release版本，本地已经存在，则不会重复下载
对于snapshots版本，不管本地是否存在，都会强制刷新，但是刷新并不意味着把jar重新下载一遍。
只下载几个比较小的文件，通过这几个小文件确定本地和远程仓库的版本是否一致，再决定是否下载

## 使用

通过`mvn -U compile`就可以把项目需要的包下载下来了。

扩展：`mvn clean install -e -U ` -e详细异常,-U强制更新





















