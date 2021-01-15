---
title: Centos下Mysql安装
date: 2021-01-08 09:03:39
tags: [编程, 过去]
category:
- 100 学习类
- 110 编程
- 113 数据库
---

1. 下载：  
下载地址：https://dev.mysql.com/downloads/mysql/5.7.html
   
2. 解压、移动  
tar -zxvf mysql-5.7.26-linux-glibc2.12-x86_64.tar.gz    
mv mysql-5.7.26-linux-glibc2.12-x86_64  /usr/local/mysql    
3. mysql用户组、权限配置   
创建mysql用户组和用户并修改权限  
```
groupadd mysql
useradd -r -g mysql mysql
```
创建数据目录（/data/mysql ），并赋予权限  
```
mkdir -p  /data/mysql              #创建目录
chown mysql:mysql -R /data/mysql   #赋予权限
```
4. 配置 my.cnf ：
vim /etc/my.cnf   
内容如下：  
```
[mysqld]
bind-address=0.0.0.0
port=3306
user=mysql
basedir=/usr/local/mysql
socket=/tmp/mysql.sock

datadir=/data/mysql
log-error=/data/mysql/mysql.err
pid-file=/data/mysql/mysql.pid

symbolic-links=0
explicit_defaults_for_timestamp=true

sql_mode=STRICT_TRANS_TABLES,NO_ZERO_IN_DATE,NO_ZERO_DATE,ERROR_FOR_DIVISION_BY_ZERO,NO_AUTO_CREATE_USER,NO_ENGINE_SUBSTITUTION

# 默认引擎
default-storage-engine=InnoDB


## 默认字符集 utf8
init_connect='SET collation_connection = utf8_unicode_ci'
init_connect='SET NAMES utf8'
character-set-server=utf8
collation-server=utf8_unicode_ci
skip-character-set-client-handshake

[mysql]
default-character-set=utf8

[client]
default-character-set=utf8
```
注意：  
在 [mysqld] 中使用 default-character-set 设置字符集， mysql 启动会报错导致无法启动。  
5. 初始化数据库  
进入mysql 的 bin 目录  
cd /usr/local/mysql/bin/  
初始化：   
```
./mysqld  --defaults-file=/etc/my.cnf --basedir=/usr/local/mysql/ --datadir=/data/mysql/ --user=mysql --initialize
```
注意：可能遇到的问题
```
./mysqld: error while loading shared libraries: libaio.so.1: cannot open shared object file: No such file or directory
```
解决方法：  
出现该问题首先检查该 链接库文件 有没有安装， 使用命令进行核查。  
rpm -qa | grep libaio  
运行该命令后发现系统中无该链接库文件。   
使用命令 ：  
yum  install  libaio-devel.x86_64    
安装成功后，继续运行数据库的初始化命令，提示成功。  
6. 查看mysql 初始化密码（随机生成的）  
cat /data/mysql/mysql.err   
7. 启动mysql  
先将 mysql.server 放置到 /etc/init.d/mysql 中：  
启动Mysql ：   
service mysql start   
上面的启动没有报错，则说明启动成功。下面是查看是否有mysql进程   
ps -ef|grep mysql    
如果有mysql进程，说明mysql已经安装成功。  
8. 修改密码   
前面的步骤我们能查出mysql 随机生成的密码，接下来，登录mysql，修改密码。   
./mysql -u root -p密码      #bin目录下  
再执行下面三步操作，然后重新登录。  
```
set password = password('1234@Mfg');
alter user 'root'@'localhost' password expire never;
flush privileges;                                     
```
此时，mysql的root 用户的密码是 1234@Mfg 。  
9. 远程连接
```
use mysql;                                          # 访问 mysql 库
update user set host = '%'  where  user = 'root';   # 使 root 能再任何 host 访问
flush privileges;                                   # 刷新
quit;                                               # 退出 mysql
```
10. 将mysql bin添加的系统bin中  
如果不希望每次都到bin目录下使用 mysql 命令则执行以下命令  
```
ln -s  /usr/local/mysql/bin/mysql /usr/bin
```










































