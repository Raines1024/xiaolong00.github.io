---
title: 阿里云oss安装ossftp
date: 2021-01-30 08:43:39
tags: [编程, 过去]
category:
- 200 工作类
- 230 工作技能
- 236 使用软件
---

## 业务需求

公司合作需要将若干大文件与山农大教授共享，通过阿里云oss进行存储，使用ossftp共享。

## ossftp概述

ossftp是一个特殊的FTP server，可以将对文件、文件夹的操作映射为对OSS的操作，使您可以基于FTP协议来管理存储在OSS上的文件。

下载地址：Linux和macOS：[ossftp-1.1.0-linux-mac.zip](https://gosspublic.alicdn.com/ossftp/ossftp-1.1.0-linux-mac.zip)

## 安装步骤

1. 解压已下载的安装包。

   安装包解压后的路径不能包含中文。

2. 运行ossftp。

   ossftp运行后会默认打开本机的TCP 2048和TCP 8192端口。其中2048端口作为FTP服务端口，用于接收FTP请求；8192端口作为Web服务端口，用于打开ossftp的图形化管理界面。若您需要将服务提供给他人使用，请在防火墙配置中开放这两个端口。linux的运行方式如下：

   1. 解压下载的文件，命令：`unzip ossftp-1.0.3-linux-mac.zip`

   2. 进入解压后的文件夹，运行start.sh。

      ```shell
      cd ossftp-1.0.3-linux-mac
      bash start.sh
      ```

   3. 通过浏览器访问ossftp的图形化管理界面，访问域名为`http://127.0.0.1:8192`。

      若本机无图形化管理界面，可使用其他电脑访问ossftp图形化管理界面，访问域名为`http://Linux服务器IP:8192`。

## 总结

参考链接：https://help.aliyun.com/document_detail/197500.html?spm=a2c4g.11186623.6.909.74f37a74wVG9mv

这种东西没什么意思，比较浅显，寻找不到本质。



























