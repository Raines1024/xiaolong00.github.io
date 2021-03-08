---
title: 从零到一搭建Hadoop+Flink集群
date: 2021-02-27 08:40:39
description: 从创建虚拟机开始···
tags: [编程, 现在]
category:
    - 100 学习类
    - 110 编程
    - 112 Linux
---

开始之前，我还不知道我要干什么，emmmmmm，茫然亦前行。

## 创建虚拟机

使用vmware esxi创建新虚拟机，选择8核心16G，使用iso镜像文件安装。

### 初始化Centos

进入系统界面，选择中国、简体中文，选择默认分区，点击确定后，选择设置登录密码，设置登录密码

### 配置ip

CentOS 7 网卡命令规则变化，命名规则根据系统固件和硬件来命名为 ifcfg-en* 类型，网络设置及静态IP配置文件
/etc/sysconfig/network-scripts/ifcfg-ens192

```yaml
TYPE=Ethernet
PROXY_METHOD=none
BROWSER_ONLY=no
BOOTPROTO=none
DEFROUTE=yes
IPV4_FAILURE_FATAL=no
IPV6INIT=yes
IPV6_AUTOCONF=yes
IPV6_DEFROUTE=yes
IPV6_FAILURE_FATAL=no
IPV6_ADDR_GEN_MODE=stable-privacy
NAME=ens192
UUID=e90f25a3-3f05-4555-b107-ec7a17d6d1b2
DEVICE=ens192
ONBOOT=yes
IPADDR=172.29.32.11
PREFIX=24
GATEWAY=172.29.32.1
IPV6_PRIVACY=no
DNS1=172.16.61.253
```

重启网络
service network restart

### 系统配置

3台虚拟机
flink1: 16g内存 8核 300g硬盘
flink2: 16g内存 8核 300g硬盘
flink3: 16g内存 8核 300g硬盘

-----------

## CentOS7关闭防火墙
- 查看防火墙状态
`firewall-cmd --state`
- 停止firewall
`systemctl stop firewalld.service`
- 禁止firewall开机启动
`systemctl disable firewalld.service`

----

## 常用软件包

### ifconfig所需软件包
yum install net-tools.x86_64
### 查看某端口占用
yum install lsof
使用示例`lsof -i:80`
### vim编辑器
yum install vim
### centos 安装rzsz
yum install lrzsz
### wget
yum install wget

------

## 修改主机名

1. 修改主机名
   hostnamectl set-hostname flink1
   more /etc/hostname
   退出重新登陆即可显示新设置的主机名flink1

2.  修改hosts文件

   vim /etc/hosts

   ```yaml
   172.29.32.11    flink1
   172.29.32.12    flink2
   172.29.32.13    flink3
   ```

3. 其他俩台主机亦是如此

-----

## CentOS服务器间配置ssh免密登陆

1. 11上`ssh-keygen -t rsa`生成密码,在/home/.ssh下生成了密钥对：id_rsa为私钥，id_rsa.pub为公钥
2. 将刚生成的公钥传到12上，且12上以root用户登陆。 `ssh-copy-id root@172.29.32.12`
3. 在11上用root用户登陆12: `ssh 172.29.32.12` 验证不需要密码可以登陆。

其他机器亦是如此

-----

## 安装Java

### 上传
使用rz命令把.tar.gz文件上传到服务器

### 解压并删除
tar -zxvf *.gz
rm -f *.gz
### 传到其他服务器
scp -r /home/java root@172.29.32.12:/home/java
scp -r /home/java root@172.29.32.13:/home/java

### 修改环境变量
vim /etc/profile
插入：

```yaml
JAVA_HOME=/home/java/jdk1.8.0_201
JRE_HOME=$JAVA_HOME/jre
PATH=$PATH:$JAVA_HOME/bin:$JRE_HOME/bin
CLASSPATH=.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib
export JAVA_HOME JRE_HOME PATH CLASSPATH
```



生效：`source /etc/profile` 

|   变量    |                             含义                             |
| :-------: | :----------------------------------------------------------: |
| JAVA_HOME | 指明JDK安装路径，就是刚才安装时所选择的路径，此路径下包括lib，bin，jre等文件夹（tomcat，Eclipse的运行都需要依靠此变量）。 |
| CLASSPATH | 为java加载类(class or lib)路径，只有类在classpath中，java命令才能识别，设：`.:$JAVA_HOME/lib/dt.jar:$JAVA_HOME/lib/tools.jar:$JRE_HOME/lib`。CLASSPATH 变量值中的.表示当前目录 |
|   PATH    | 使得系统可以在任何路径下识别java命令，设为：`$JAVA_HOME/bin:$JRE_HOME/bin`。 |
| 特别注意  |       环境变量值的结尾没有任何符号，不同值之间用:隔开        |

### 验证

命令`java -version`

------

## zookeeper集群搭建

注：zookeeper集群节点数必须为奇数，满足leader选举算法(当选leader的节点所需支持节点数过半的原则)，且>=3

1、下载zookeeper安装包

```
curl -O https://archive.apache.org/dist/zookeeper/zookeeper-3.4.10/zookeeper-3.4.10.tar.gz
```

2、解压安装包

```
tar -zxvf zookeeper-3.4.10.tar.gz
```

3、拷贝"xxx安装目录/zookeeper-3.4.10/conf"目录下的zoo_sample.cfg文件，重命名为zoo.cfg，并进行编辑，添加如下设置

```yaml
cp zoo_sample.cfg zoo.cfg
vim zoo.cfg
插入内容：
dataDir=/home/data/zookeeper
server.1=flink1:2888:3888
server.2=flink2:2888:3888
server.3=flink3:2888:3888
```

4、远程传输到flink2、flink3节点上

```
scp -r /home/zookeeper/zookeeper-3.4.10 root@flink2:/home/zookeeper/zookeeper-3.4.10
scp -r /home/zookeeper/zookeeper-3.4.10 root@flink3:/home/zookeeper/zookeeper-3.4.10
```

5、在配置的文件目录/home/data/zookeeper目录下，新建myid文件并编辑(用于标识在zookeeper集群的编号)

如上配置，flink1服务器为1，flink2服务器内容为2，flink3服务器内容为3

6、启动/停止zookeeper

```yaml
bin/zkServer.sh start          //启动zookeeper
bin/zkServer.sh stop           //停止zookeeper
bin/zkServer.sh status         //查看zookeeper状态
```

三个节点的zookeeper服务都需要开启，仔细查看是否每台节点的zk服务是否都开启。并且开启后稍等片刻，看着否状态显示正常。

----

## Flink 1.10 on yarn集群搭建(Hadoop-2.9.2)

### 安装

1. 下载Flink 1.10安装包
2. 解压安装包`tar zxvf *.tgz`
3. 远程传输到flink2、flink3节点

### JobManager高可用配置

1. Hadoop配置文件yarn-site.xml

   Hadoop主目录下etc/hadoop/yarn-site.xml(hadoop安装目录下的yarn集群配置文件）

   添加如下配置

   ```xml
   <property>
     <name>yarn.resourcemanager.am.max-attempts</name>
     <value>4</value>
     <description>
       The maximum number of application master execution attempts.
     </description>
   </property>
   ```

2. Flink配置文件flink-conf.yaml

   Flink主目录下conf/flink-conf.yaml

   屏蔽jobmanager.rpc.address配置

   高可用设置

   ```yaml
   high-availability: zookeeper
   high-availability.zookeeper.quorum: hadoop1:2181,hadoop2:2181,hadoop3:2181
   high-availability.storageDir: hdfs:///flink/recovery
   high-availability.zookeeper.path.root: /flink
   yarn.application-attempts: 10
   ```

3. 添加flink环境变量/etc/profile

   vim /etc/profile

   ```
   #flink环境变量配置
   export FLINK_HOME=/usr/local/softwareinstall/flink-1.10.0
   export PATH=$PATH:$FLINK_HOME/bin
   #flink on yarn配置
   HADOOP_CONF_DIR=$HADOOP_HOME
   export HADOOP_CLASSPATH=`hadoop classpath`
   ```

   `source /etc/profile`   使环境变量生效

4. Flink具体使用后续再写

-----

## 挂载硬盘

由于安装虚拟机时使用默认分区，存储挂载在/home目录，有时强迫症犯了还是喜欢修改为/data

### 直接挂载。但是是用逻辑卷的名称挂载。硬盘上的数据还在。

1、用 `fdisk -l` 命令，查看服务器物理分区，逻辑卷的信息(前面是物理分区，后面是逻辑卷信息)

2、用`lvdisplay`命令查看逻辑卷的具体信息

3、用`mount /dev/mapper/centos-home /data`命令将逻辑卷（如：逻辑卷的名称为：/dev/mapper/centos-home ，即在第1步中看到的逻辑卷信息）挂载在目录（如：/data）下。

### 其它命令：

df -h（查看分区情况及数据盘名称）

umount /home（卸载硬盘已挂载的home目录）

自动挂载配置文件:/etc/fstab （编辑fstab文件修改或添加，使重启后可以自动挂载）

### 其它链接：

[CentOS 7 下挂载新硬盘](https://segmentfault.com/a/1190000008007157)





















