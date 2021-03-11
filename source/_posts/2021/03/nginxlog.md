---
title: Nginx 访问日志文件太大处理方案
date: 2021-03-08 19:01:39
description: 
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 112 Linux

---

## 背景

由于项目中频繁使用地理信息，便不可避免的频繁使用经纬度逆解析、不同坐标系经纬度转换等接口，而当业务系统频繁调用这些公共接口时，如果不对访问日志做特殊处理，采用nginx的默认方案的话，则会产生非常庞大的日志文件，于是，便有了此文。

## Nginx access.log文件太大处理方案

Nginx在涉及大流量时，会产生非常庞大的日志文件，包含access.log和error.log，日志会随着连接不断增加，到无限大。如果日志文件太大，会导致Nginx运行缓慢，卡顿，也是存储资源的浪费。

该文件为nginx的访问日志文件可以删除,删除后nginx启动还会产生
如果要关闭日志功能,在nginx配置文件中找到access_log一行,改为`access_log off;` 可用于`http`, `server`, `location`, `if in location`, `limit_except`块。

### 手动释放清理Nginx日志文件access.log
查看并查找相关信息及路径

```
# 查看空间占用
$ df -h

# 定位Nginx
$ which nginx
/usr/local/nginx/logs

# 列出日志文件
$ cd /usr/local/nginx/logs
ls

# 查看日志文件大小
$ du -sh ./*

# 暂停Nginx并删除日志文件
# nginx -s stop
rm -rf *.log
```

另外，你也可以使用覆盖日志的方法清理Nginx日志文件
`echo "" > /usr/local/nginx/access.log`
如果不需要日志文件就直接关闭（不建议），修改nginx.conf
`access_log off; `

### 对Nginx access.log进行分割
通过shell脚本+linux的定时任务进行的一个平滑切分。不需要重启nginx进程。代码cut_logs.sh

```sh
#!/bin/bash
log_path=/usr/local/nginx/logs/access.log
save_path=/usr/local/nginx/logs/bak/access_$(date +%Y%m%d -d 'yesterday').log
cp $log_path $save_path && echo > $log_path
```

设置定时任务 `crontab -e` 输入
`0 0  * * * /usr/bin/sh cut_logs.sh #每天的00:00执行日志切分`
然后运行命令`crontab -l`查看定时任务是否添加成功