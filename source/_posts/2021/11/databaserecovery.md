---
title: 数据库宕机后的数据恢复
date: 2021-11-02 21:28:39
description: 
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java

---



## 背景

时间真是过得太快了。年前的壮志

## 介绍

- cluster
集群
所有 Kafka broker 组成的集群


- controller
控制者
Kafka broker 中比较特殊的一个，相当于其他主从结构系统中的 master


- replication
副本
同一分区的数据会在不同的 broker 上存储多份，每一份都称为一个副本


- leader
领导者

- follower
跟随者
持有分区副本的 broker，不是领导者的就是跟随者，唯一的任务就是从领导者同步最新的消息，相当于备用领导者

### 集群的基本组织方式
#### broker 的启动
当每个 broker 启动时，会在 ZooKeeper 中的 /brokers/ids 路径下创建一个节点来注册自己，节点 ID 为配置文件中的 broker.id 参数，不同节点不可以指定相同的 ID，后注册的 broker 会报 NodeExists 的错。

> 如果不指定 broker.id 或者指定成 -1，节点 ID 会从 reserved.broker.max.id 这个参数加 1 的值开始，这个参数默认值是 1000，所以经常可以看见 1001、1002 的 broker ID。

连接zookeeper进入命令行，使用`ls /brokers/ids`查看broker的节点，用`get /brokers/ids/0`命令来查看broker注册的信息，里面包含了broker的IP和端口、注册时间、版本以及其它属性，这种树形结构和文件系统结构非常类似。

#### broker 的监听
每个 broker 除了注册自身之外，还会监听 /brokers/ids 这个节点，当这个节点下增加或删除子几点时，ZooKeeper 会通知监听了的 broker。每个broker创建的节点都是临时节点，如果 broker 下线，/brokers/ids 下对应的节点就会被删除。

### controller 的作用
controller 除了是一个普通的 broker 之外，还是集群的总扛把子，它负责副本leader的选举、topic的创建和删除、副本的迁移、副本数的增加、broker上下线的管理等等，这一小节主要关注副本 leader 的选举这一个功能。
#### controller 的选举
选举过程如下：每个 broker 启动的时候其实都想成为 controller，成为 controller 的方法就是在 ZooKeeper 中创建 /controller 临时节点节点并注册自己的信息，先注册上的 broker 就是 controller，后来的发现已经有这个节点就知道已经有人抢先了，就放弃了，但是它们会监听这个节点的状态，伺机而动。
用 get /controller 命令看下这个节点的内容：
```
[zk: localhost:2181(CONNECTED) 0] get /controller
{"version":1,"brokerid":1,"timestamp":"1633968391492"}
```
可以看到，ID 为 1 的 broker 目前是 controller。
#### controller的重新选举
当 controller 下线时，因为无法和 ZooKeeper 保持心跳，/controller 临时节点会被删除，每个 broker 发现这一点之后，就开始新一轮的选举（其实就是抢）。选举出新 controller 的同时，/controller_epoch 中的值也会加 1，这个节点记录 controller 的代数，broker 也会监听这个节点，并且知道当前 controller 的代数，旧的 controller 的指令即使继续发送指定给它们，指令也会被忽略。
重新选举的过程过程也可以手动触发，我们在 ZooKeeper 里用 rmr /controller 命令删掉这个节点，然后再查看这个节点：
```
[zk: localhost:2181(CONNECTED) 1] rmr /controller
[zk: localhost:2181(CONNECTED) 2] get /controller
{"version":1,"brokerid":0,"timestamp":"1635129671205"}
```
果然，controller 换人了。
#### controller 管理 broker 的离开和加入
当有 broker 下线时，由于这个 broker 可能是多个分区的leader，所以controller需要给这些分区重新选举一个新的leader，说是选举，其实就是直接指定每个分区副本列表里的下一个broker，举个例子：
```
[zk: localhost:2181(CONNECTED) 24] get /brokers/topics/lovol_farmcar
{"version":1,"partitions":{"4":[1,0],"5":[2,1],"1":[1,2],"0":[0,1],"2":[2,0],"3":[0,2]}}
```
lovol_farmcar这个 topic 创建时有五个分区，每个分区有二个副本，分区 0 的 leader 最好是 0，为什么说是最好呢，因为 leader 需要处理所有客户端的请求，任务比较重，Kafka 有一套负载均衡机制来确定每个分区的 leader 应该在哪台 broker 上，这个 broker 叫做 preferred_leader，且跟在 0 后面的 broker ID 是 1，在没有 broker 下线的时候，看下分区 0 的 leader：
```
[zk: localhost:2181(CONNECTED) 28] get /brokers/topics/lovol_farmcar/partitions/0/state
{"controller_epoch":200,"leader":0,"version":1,"leader_epoch":0,"isr":[0,1]}
```
这样的状态是正常的，注意 leader_epoch 值为 0，表示当前 leader 的代数，现在我把 0 这台 broker 关掉，会发生什么变化：
进入0的broke服务器，cd到kafka目录，执行`bin/kafka-server-stop.sh`命令关闭后回zookeeper命令行查看
```
[zk: localhost:2181(CONNECTED) 2] get /brokers/topics/lovol_farmcar
{"version":1,"partitions":{"4":[1,0],"5":[2,1],"1":[1,2],"0":[0,1],"2":[2,0],"3":[0,2]}}
[zk: localhost:2181(CONNECTED) 3] get /brokers/topics/lovol_farmcar/partitions/0/state
{"controller_epoch":202,"leader":1,"version":1,"leader_epoch":1,"isr":[1]}
```
leader 变成 1 了，并且 leader_epoch  变成了 1，这个过程中，controller 会向这个分区的所有副本，也就是 1 和 2 发送 leader 变更的通知，然后 2 就成为 leader 并开始处理客户端的生产和消费请求。
假如 0 这台 broker 又重新上线呢？
再次进入0的broke服务器，cd到kafka目录，执行`bin/kafka-server-stop.sh nohup bin/kafka-server-start.sh config/server.properties 1>null 2>null &`启动kafka后回zookeeper命令行查看
```
[zk: localhost:2181(CONNECTED) 5] get /brokers/topics/lovol_farmcar                   
{"version":1,"partitions":{"4":[1,0],"5":[2,1],"1":[1,2],"0":[0,1],"2":[2,0],"3":[0,2]}}
[zk: localhost:2181(CONNECTED) 4] get /brokers/topics/lovol_farmcar/partitions/0/state
{"controller_epoch":202,"leader":0,"version":1,"leader_epoch":2,"isr":[1,0]}
```
可以看到，0 会重新变成这个分区的 leader，并且 leader_epoch  又加了 1，因为它是 preferred leader，但这个过程并不是立马发生的，broker 有一个参数叫做 leader.imbalance.check.interval.seconds，默认值为 300，也就是每 5 分钟会检测一次每个分区的 leader 是不是 preferred leader， 如果不是的话，会主动更换 leader。


#### leader 和 follower 的关系
我们已经知道，在每个分区的多个副本中，有一个是 leader，其它是 follower，follower 就干一件事，从 leader 同步数据，然后等着 leader 挂掉，自己有机会成为 leader。
leader干两件事，一是服务用户发来的生产和消费请求，二是监控 follower 同步数据的进度，只有一直和 leader 保持同步的 follower 才有机会成为下一个 leader，那么保持同步的标准是什么呢？标准就是，如果一个 follower 超过 10 秒没有向 leader 请求数据或者请求数据了，但是没有在 10 秒内跟上 leader 的最新消息，那么这个 follower 就是不同步的，当然这里的 10 秒是个可配置的参数，叫做 `replica.lag.time.max.ms`。
对于能够和 leader 保持同步的 follower，有个专门的名词就做 ISR，是 in-sync replicas 的缩写，其实在上面的截图里也可以看到每个 partition 也会维护哪些 follower 是在 ISR 里的。

