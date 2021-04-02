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
     `mkdir /usr/local/redis `

   - 解压到/usr/local/redis目录

     `tar -zxvf redis-6.0.8.tar.gz -C /usr/local/redis #解压到/usr/local/redis目录`

   - 切换到redis主目录

     `cd /usr/local/redis/redis-6.0.8`

3. 编译安装

   - 查看gcc版本(centos7默认安装的版本是4.8.5，但是redis6.0要求对应版本要在5.3以上)

     `gcc -v`

     如果小于5.3，则升级到5.3以上版本，依次执行命令

     ```sh
     yum -y install centos-release-scl
     yum -y install devtoolset-9-gcc devtoolset-9-gcc-c++ devtoolset-9-binutils
     scl enable devtoolset-9 bash
     echo “source /opt/rh/devtoolset-9/enable” >>/etc/profile　　#使永久生效
     ```

   - 安装tcl，执行命令

     `yum install tcl -y`

   - 编译并安装

     ```sh
     make #编译，之前的RPM安装包就是跳过了此步骤
     make install #安装，将redis的命令安装到/usr/local/bin/目录
     ```

     查看编译好的命令文件

     ```sh
     [root@localhost redis-6.0.8]# ls /usr/local/bin/redis-*
     /usr/local/bin/redis-benchmark	#性能测试工具
     usr/local/bin/redis-check-aof		#更新日志检查
     /usr/local/bin/redis-check-rdb	#本地数据文件检查
     /usr/local/bin/redis-cli				#命令行操作工具
     /usr/local/bin/redis-sentinel -> redis-server
     /usr/local/bin/redis-server			#服务器程序
     ```

4. 后端模式启动

   - 服务器防火墙配置(或者关闭防火墙)

     ```sh
     firewall-cmd --zone=public --add-port=6379/tcp --permanent ----添加6379端口
     firewall-cmd --reload ----重启防火墙
     firewall-cmd --list-port -----查看所有开放端口号
     firewall-cmd --query-port=6379/tcp -----查看指定端口是否开放
     ```

   - 修改redis.conf配置文件

     `vim /usr/local/redis/redis-6.0.8/redis.conf`

     - 修改前

       ```
       bind 127.0.0.1 #绑定ip：如果需要远程访问，可将此行注释，或绑定一个真实ip
       port 6379 #端口号
       protected-mode yes #是否开启保护模式
       daemonize no #是否设为后台运行
       #requirepass foobared #密码设置
       pidfile /var/run/redis_6379.pid #进程文件保存位置，redis运行后会在此位置自动生成
       logfile “” #日志文件保存位置
       dir ./ #redis位置
       ```

     - 修改后

       > :/prot (:/找询的单词,在Esc模式下输入)----作用快速找到需要更改内容

     - ```
       #bind 127.0.0.1 #允许所有IP访问
       port 6379 #端口号为6379
       protected-mode no #关闭保护模式，不然远程还是连接不了
       daemonize yes #设为后台运行
       #requirepass 123456 #简化开发，没有设置密码
       pidfile /var/run/redis_6379.pid #修改为你的安装目录 redis_端口号 端口改为该redis服务端口
       logfile /usr/local/redis/redis-6.0.8/redis_log.log #修改redis日志存放位置
       dir /usr/local/redis/redis-6.0.8 #修改redis位置
       ```

5. 启动和关闭redis

   ```
   redis-server /usr/local/redis/redis-6.0.8/redis.conf #使用指定配置启动[后台启动模式]
   ```

   - 启动成功测试

     ```sh
     [root@localhost redis-6.0.8]# ps -aux|grep redis
     root     27036  0.0  0.0 162420  7804 ?        Ssl  09:42   0:00 redis-server *:6379
     root     27047  0.0  0.0 112828   984 pts/0    S+   09:42   0:00 grep --color=auto redis
     ```

   - 关闭redis

     ```sh
     redis-cli shutdown #没有设置密码，运行此行代码
     redis-cli -a 123456 shutdown #设置密码，运行此行
     ```

   - 登录redis

     ```sh
     redis-cli -h 127.0.0.1 -p 6379 -a 123456
     ```





