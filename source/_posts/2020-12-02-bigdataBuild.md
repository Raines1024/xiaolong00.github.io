---
layout: post
title: 搭建Hadoop-2.9.2集群
description: 
category: blog
---

## 准备

### 在安装 Hadoop 之前，请确认集群的每台机器上均安装 JDK，以及搭配环境变量。
- 添加Java环境变量  
vim ~/.bash_profile  

```
# Java Environment
JAVA_HOME=/usr/java/latest
JRE_HOME=${JAVA_HOME}/jre
CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib
PATH=${JAVA_HOME}/bin:$PATH
PATH=$PATH:$HOME/bin

export PATH
```   

- 生效  
source ~/.bash_profile    

### 在三台机器/etc/hosts写入对应机器,修改对应 hostname

```
172.29.32.21 openshift1
172.29.32.22 openshift2
172.29.32.23 openshift3
```

### 配置ssh无密码登陆（见之前文章）

### CentOS7关闭防火墙
- 查看防火墙状态
firewall-cmd --state
- 停止firewall
systemctl stop firewalld.service
- 禁止firewall开机启动
systemctl disable firewalld.service 

## 下载二进制文件

```
cd /opt
wget https://mirrors.tuna.tsinghua.edu.cn/apache/hadoop/common/hadoop-2.9.2/hadoop-2.9.2.tar.gz
tar -zxvf hadoop-2.9.2.tar.gz -C .
```

## 修改配置文件

```
vim /etc/profile
# Flink Environment
export HADOOP_HOME=/opt/hadoop-2.9.2
export PATH=$PATH:${HADOOP_HOME}/bin:${HADOOP_HOME}/sbin
```

### core-site.xml
指定 NameNode 的 IP 地址和端口号

```
<configuration>
  <property>
    <name>fs.default.name</name>
    <value>hdfs://openshift1:9000</value>
  </property>
</configuration>
```

### hdfs-site.xml
dfs.replication 指定备份数目为 3，dfs.name.dir 指定 NameNode 的文件存储路径，dfs.data.dir 指定 DataNode 的文件存储路径。

```
<configuration>
  <property>
    <name>dfs.replication</name>
    <value>3</value>
  </property>
  <property>
    <name>dfs.namenode.name.dir</name>
    <value>file:/opt/hadoop-2.9.2/data/namenode</value>
  </property>
  <property>
    <name>dfs.datanode.data.dir</name>
    <value>file:/opt/hadoop-2.9.2/data/datanode</value>
  </property>
</configuration>
```

### mapred-site.xml
cp mapred-site.xml.template mapred-site.xml
然后修改mapred-site.xml的内容

```
<configuration>
  <property>
    <name>mapreduce.framework.name</name>
    <value>yarn</value>
  </property>
</configuration>
```

### yarn-site.xml

```
<configuration>

<!-- Site specific YARN configuration properties -->
  <property>
    <name>yarn.nodemanager.aux-services</name>
    <value>mapreduce_shuffle</value>
  </property>
  <property>
    <name>yarn.nodemanager.aux-services.mapreduce.shuffle.class</name>
    <value>org.apache.hadoop.mapred.ShuffleHandler</value>
  </property>
  <property>
    <name>yarn.resourcemanager.resource-tracker.address</name>
    <value>openshift1:8025</value>
  </property>
  <property>
    <name>yarn.resourcemanager.scheduler.address</name>
    <value>openshift1:8030</value>
  </property>
  <property>
    <name>yarn.resourcemanager.address</name>
    <value>openshift1:8050</value>
  </property>
</configuration>
```

### slaves
添加 slave 节点的 hostname 到该文件中

```
openshift2
openshift3
```

### 设置环境变量
修改hadoop-env.sh

```
export JAVA_HOME=/usr/java/latest
```
## 分发配置文件

```
scp -r /opt/hadoop-2.9.2 openshift2:/opt/hadoop-2.9.2
scp -r /opt/hadoop-2.9.2 openshift3:/opt/hadoop-2.9.2
```

## 启动集群

### 格式化 HDFS
hdfs namenode -format
### 启动集群
start-dfs.sh  
start-yarn.sh
### 使用 jps 命令查看服务运行情况
- master节点中运行的服务  
25928 SecondaryNameNode  
25742 NameNode  
26387 Jps  
26078 ResourceManager  
-  slave节点中运行的服务  
24002 NodeManager  
23899 DataNode  
24179 Jps  
### 提交示例任务

```
cd /opt/hadoop-2.9.2
hdfs dfs -mkdir /wordcount
hdfs dfs -mkdir /wordcount/input
# 把当前路径下的 LICENSE.txt 文件复制到 HDFS 中
hadoop fs -put ./LICENSE.txt /wordcount/input
# 提交任务，最后两个参数分别指定任务的输入和输出
hadoop jar $HADOOP_HOME/share/hadoop/mapreduce/hadoop-mapreduce-examples-2.9.2.jar wordcount /wordcount/input /wordcount/output
# 查看输出路径
hadoop fs -ls /wordcount/output
# 如果一切正常，该路径下包含两个文件
# 第一个文件是空文件，表示任务运行成功
/wordcount/output/_SUCCESS
# 第二个文件是输出文件，统计了 LICENSE.txt 中每个单词出现的次数
/wordcount/output/part-r-00000
```

## 坑

### 多次NameNode执行format后DataNode启动不了解决方案
1.问题

执行start-dfs.sh后在进程中查看jps，发现NameNode启动，但DataNode没有

2.原因

在失败的.log文件中看到datanode的clusterID 和 namenode的clusterID 不一致

原因可能是多次Hadoop namenode -format导致clusterID不一致

3.解决方法

1）先去hadoop路径下的配置文件hdfs-site.xml可知dfs.namenode.name.dir的地址和dfs.datanode.data.dir的地址

默认：file://${hadoop.tmp.dir}/dfs/name、file://${hadoop.tmp.dir}/dfs/data

（/opt/module/hadoop-2.7.2/data/tmp/dfs/name、/opt/module/hadoop-2.7.2/data/tmp/dfs/data）

2）在.../name/current/VERSION 中获得clusterID  

```
more VERSION 
#Mon Sep 02 18:06:26 CST 2019
namespaceID=1033971221
clusterID=CID-98e754ef-ad92-49f8-88b2-6830888f2d48
cTime=0
storageType=NAME_NODE
blockpoolID=BP-1400239548-192.168.1.201-1567418785939
layoutVersion=-63
```
3）将clusterID修改到.../dfs/data/current/VERSION（所有启动不了DataNode的节点都要改）

```
vi VERSION 
#Sun Sep 01 19:46:01 CST 2019
storageID=DS-330d79ed-7c1b-4d40-b151-81ffcadcf9f0
#clusterID=CID-ae479da3-0b1e-44b0-a383-029a213b3481
clusterID=CID-98e754ef-ad92-49f8-88b2-6830888f2d48
cTime=0
datanodeUuid=67fcc2ae-1b74-46cd-90df-336a0b1950e6
storageType=DATA_NODE
layoutVersion=-56
```
4）再次启动DataNode，成功启动
sbin/hadoop-daemon.sh start datanode











