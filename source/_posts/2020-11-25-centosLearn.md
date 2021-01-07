---
layout: post
title: CentOS必知必会
description: CentOS学习
category: blog
---

## centos安装rzsz
yum install lrzsz

## 查看centos系统的版本
cat /etc/redhat-release

## 常用命令
free    # 查看服务器内存使用
netstat -lntup    # 查看端口情况  l->listen,n->num,t->tcp,u->udp,p->process
cat /etc/locale.conf   查看使用编码     
lsblk  查看盘符  



## zip压缩当前文件夹下所有文件
zip -r myfile.zip ./*

## 两台centos之间传送文件
scp -r /data/appmanage root@172.16.70.11:/data/appmanage

## 对当前文件夹打包
zip -r myfile.zip ./*   

## CentOS下查看文件和文件夹大小
查看data文件夹下大小
du -h --max-depth=1 data/
https://blog.csdn.net/qq_33326449/article/details/107446140

## 设置环境变量
vim /etc/profile   
- eg:添加maven环境变量   
  
  ```
  M2_HOME=/usr/maven/apache-maven-3.6.3
  export PATH=${M2_HOME}/bin:${PATH}
  ```
source /etc/profile   






## linux crontab 定时任务
## cron介绍
我们经常使用的是crontab命令是cron table的简写，它是cron的配置文件，也可以叫它作业列表，我们可以在以下文件夹内找到相关配置文件。  
/var/spool/cron/ 目录下存放的是每个用户包括root的crontab任务，每个任务以创建者的名字命名  
/etc/crontab 这个文件负责调度各种管理和维护任务。  
/etc/cron.d/ 这个目录用来存放任何要执行的crontab文件或脚本。  
我们还可以把脚本放在/etc/cron.hourly、/etc/cron.daily、/etc/cron.weekly、/etc/cron.monthly目录中，让它每小时/天/星期、月执行一次。   

```
crontab [-u username]　　　　//省略用户表表示操作当前用户的crontab
    -e      (编辑工作表)
    -l      (列出工作表里的命令)
    -r      (删除工作表)
```
我们用crontab -e进入当前用户的工作表编辑，是常见的vim界面。每行是一条命令。  
crontab的命令构成为 时间+动作，其时间有分、时、日、月、周五种，操作符有  

```
* 取值范围内的所有数字
/ 每过多少个数字
- 从X到Z
，散列数字
```

### 使用实例

```
实例1：每1分钟执行一次myCommand
* * * * * myCommand

实例2：每小时的第3和第15分钟执行
3,15 * * * * myCommand

实例3：在上午8点到11点的第3和第15分钟执行
3,15 8-11 * * * myCommand
```


