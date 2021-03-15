---
title: Linux系统free命令的buff/cache占用过高
date: 2021-03-15 19:01:39
description: 因buff/cache占用过高系统内存不足最终java应用程序崩溃解决方案
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 112 Linux

---

## 背景

Java程序运行时间长了，使用`free -h`命令时经常会发现buff/cache占用相当高，甚至会因为占用太高内存而导致Java应用程序崩溃。

## 解决方案

### free命令简介

先来说一下free命令

```sh
[root@centos demo]# free -h
              total        used        free      shared  buff/cache   available
Mem:            31G         22G        9.1G        8.8M        202M        8.9G
Swap:           14G        1.8M         14G
```

其中：
total 内存总数
used 已经使用的内存数
free 空闲的内存数
shared 多个进程共享的内存总额
buffers Buffer Cache和cached Page Cache 磁盘缓存的大小
-buffers/cache 的内存数:used - buffers - cached
+buffers/cache 的内存数:free + buffers + cached

然后我们来搞点有意思的：如果我执行复制文件，内存会发生什么变化。

```sh
[root@centos demo]# cp -r /var /data/demo/
[root@centos demo]# free -h
              total        used        free      shared  buff/cache   available
Mem:            31G         22G        1.0G        8.8M        8.1G        8.7G
Swap:           14G        1.8M         14G
```

在我命令执行结束后，used为22G，free为1.0G，buff/cache为8.1G，和我们刚才执行free命令的时候对比，发现竟有近8个G的内存都被buff/cache吃掉了。别紧张，这是为了提高文件读取效率的做法。

为了提高磁盘存取效率，Linux做了一些精心的设计，除了对dentry进行缓存（用于VFS，加速文件路径名到inode的转换），还采取了两种主要Cache方式：BufferCache和PageCache。前者针对磁盘块的读写，后者针对文件inode的读写。这些Cache有效缩短了I/O系统调用（比如read，write，getdents）的时间。

接着来，就让我们手动释放掉这些内存吧。

### sync命令简介

**sync命令**用于强制被改变的内容立刻写入磁盘，更新超块信息。

在Linux/Unix系统中，在文件或数据处理过程中一般先放到内存缓冲区中，等到适当的时候再写入磁盘，以提高系统的运行效率。sync命令则可用来强制将内存缓冲区中的数据立即写入磁盘中。用户通常不需执行sync命令，系统会自动执行update或bdflush操作，将缓冲区的数据写 入磁盘。只有在update或bdflush无法执行或用户需要非正常关机时，才需手动执行sync命令。

#### buffer与cache

- buffer：为了解决写磁盘的效率
- cache：为了解决读磁盘的效率

linux系统为了提高读写磁盘的效率，会先将数据放在一块buffer中。在写磁盘时并不是立即将数据写到磁盘中，而是先写入这块buffer中了。此时如果重启系统，就可能造成数据丢失。

sync命令用来flush文件系统buffer，这样数据才会真正的写到磁盘中，并且buffer才能够释放出来，flush就是用来清空buffer。sync命令会强制将数据写入磁盘中，并释放该数据对应的buffer，所以常常会在写磁盘后输入sync命令来将数据真正的写入磁盘。

如果不去手动的输入sync命令来真正的去写磁盘，linux系统也会周期性的去sync数据。

### 手动释放缓存

/proc是一个虚拟文件系统，我们可以通过对它的读写操作做为与kernel实体间进行通信的一种手段。也就是说可以通过修改/proc中的文件，来对当前kernel的行为做出调整。那么我们可以通过调整/proc/sys/vm/drop_caches来释放内存。操作如下：

1. 查看/proc/sys/vm/drop_caches的值，默认为0。

   ```sh
   [root@centos demo]# cat /proc/sys/vm/drop_caches
   0
   ```

2. 执行sync命令

   描述：sync 命令运行 sync子例程。如果必须停止系统，则运行sync 命令以确保文件系统的完整性。sync命令将所有未写的系统缓冲区写到磁盘中，包含已修改的 i-node、已延迟的块 I/O和读写映射文件。

```sh
[root@centos demo]# sync
```

3. 将/proc/sys/vm/drop_caches值设为3

   ```sh
   [root@centos demo]# echo 3 > /proc/sys/vm/drop_caches
   [root@centos demo]# cat /proc/sys/vm/drop_caches
   ```

   

4. 再运行free命令,会发现现在的used为22G，free为9.2G，buff/cache为93M。那么有效的释放了buff/cache。

   ```sh
   [root@centos demo]# free -h
                 total        used        free      shared  buff/cache   available
   Mem:            31G         22G        9.2G        8.8M         93M        9.0G
   Swap:           14G        1.8M         14G
   ```

   

## 总结

执行以下命令：

```sh
sync && echo 1 > /proc/sys/vm/drop_caches
sync && echo 2 > /proc/sys/vm/drop_caches
sync && echo 3 > /proc/sys/vm/drop_caches
```

drop_caches的值可以是0-3之间的数字，代表不同的含义：
0：不释放（系统默认值）
1：释放页缓存
2：释放dentries和inodes
3：释放所有缓存

其他：

- 可以设置定时任务执行：
  这里可以考虑用定时器，几天或者几小时执行一次。
  执行：crontab -e
  然后在配置文件中加入 0 4 * * * sync && echo 1 > /proc/sys/vm/drop_caches
  表示每天凌晨4点清理1次buff/cache，当然这个执行周期可以随意设置。
- 注意：在执行这三条命令之前一定要先执行sync命令（描述：sync 命令运行 sync 子例程。如果必须停止系统，则运行sync 命令以确保文件系统的完整性。
  sync 命令将所有未写的系统缓冲区写到磁盘中，包含已修改的 i-Node、已延迟的块 I/O 和读写映射文件）



不过，一般情况下，应用在系统上稳定运行了，free值也会保持在一个稳定值的，虽然看上去可能比较小。
当发生内存不足、应用获取不到可用内存、OOM错误等问题时，还是更应该去分析应用方面的原因，如用户量太大导致内存不足、发生应用内存溢出等情况，否则，清空buffer，强制腾出free的大小，可能只是把问题给暂时屏蔽了。











