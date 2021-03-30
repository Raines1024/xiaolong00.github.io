---
title: Linux磁盘介绍及管理
date: 2021-03-08 19:01:39
description: 使用fdisk命令管理磁盘分区
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 112 Linux

---

## 磁盘分区认识

### 在Linux中检查使用的是UEFI还是BIOS
最简单地找出使用的是UEFI还是BIOS的方法是查找`/sys/firmware/efi`文件夹。如果使用的 BIOS 那么该文件夹不存在。
### 在Linux中对磁盘分区的两个方案
1. MBR分区方案

   MBR分区方案特点

   1. 最多支持四个主分区，

   2. 在Linux上使用扩展分区和逻辑分区最多可以创建15个分区
   3. 由于分区中的数据以32位存储，使用MBR分区是最大支持2T空间
   4. 用fdisk管理工具来创建MBR分区

2. GPT分区方案

## MBR分区方案

### 分区名

命名方式：`/dev/sd[a-z]n`

- 查看分区 `ls /dev/sda*`

- /dev/sda1命名分解

  dev：设备文件的目录

  sd：SCSI硬盘

  a：第一块硬盘

  1：分区号

### 使用fdisk命令管理磁盘分区

命令格式： `fdisk [选项] 设备名`
常用选项：-l，查看磁盘分区表。例：`fdisk -l /dev/sda`

### 实战

#### 在虚拟机中添加一块磁盘，给磁盘划分分区

1. 在虚拟机上添加一块16G磁盘

2. 给磁盘划分分区

   ```sh
   [root@localhost ~]# fdisk /dev/sdb
   欢迎使用 fdisk (util-linux 2.23.2)。
   
   更改将停留在内存中，直到您决定将更改写入磁盘。
   使用写入命令前请三思。
   
   
   命令(输入 m 获取帮助)：m
   命令操作
      a   toggle a bootable flag
      b   edit bsd disklabel
      c   toggle the dos compatibility flag
      d   delete a partition
      g   create a new empty GPT partition table
      G   create an IRIX (SGI) partition table
      l   list known partition types
      m   print this menu
      n   add a new partition
      o   create a new empty DOS partition table
      p   print the partition table
      q   quit without saving changes
      s   create a new empty Sun disklabel
      t   change a partition's system id
      u   change display/entry units
      v   verify the partition table
      w   write table to disk and exit
      x   extra functionality (experts only)
   
   命令(输入 m 获取帮助)：p															#显示已有的分区表
   
   磁盘 /dev/sdb：17.2 GB, 17179869184 字节，33554432 个扇区
   Units = 扇区 of 1 * 512 = 512 bytes
   扇区大小(逻辑/物理)：512 字节 / 512 字节
   I/O 大小(最小/最佳)：512 字节 / 512 字节
   磁盘标签类型：dos
   磁盘标识符：0xce0f8376
   
      设备 Boot      Start         End      Blocks   Id  System
   
   命令(输入 m 获取帮助)：n															#新建一个分区
   Partition type:
      p   primary (0 primary, 0 extended, 4 free)		#表示创建主分区
      e   extended																		#表示创建扩展分区
   Select (default p): p
   分区号 (1-4，默认 1)：
   起始 扇区 (2048-33554431，默认为 2048)：
   将使用默认值 2048
   Last 扇区, +扇区 or +size{K,M,G} (2048-33554431，默认为 33554431)：+1G		#指定分区大小
   分区 1 已设置为 Linux 类型，大小设为 1 GiB
   
   命令(输入 m 获取帮助)：p
   
   磁盘 /dev/sdb：17.2 GB, 17179869184 字节，33554432 个扇区
   Units = 扇区 of 1 * 512 = 512 bytes
   扇区大小(逻辑/物理)：512 字节 / 512 字节
   I/O 大小(最小/最佳)：512 字节 / 512 字节
   磁盘标签类型：dos
   磁盘标识符：0xce0f8376
   
      设备 Boot      Start         End      Blocks   Id  System
   /dev/sdb1            2048     2099199     1048576   83  Linux
   命令(输入 m 获取帮助)：w							#保存并退出
   The partition table has been altered!
   
   Calling ioctl() to re-read partition table.
   正在同步磁盘。
   ```

   查看分区后的分区设备为`/dev/sdb1`

   ```
   [root@localhost ~]# ls /dev/sdb*
   /dev/sdb  /dev/sdb1
   ```

#### 使用sdb1新分区

新分的分区在使用之前，需要对分区进行格式化之后才能够使用。

命令：`mkfs`用于对分区进行格式化

使用：`mkfs.文件系统格式 [选项] 分区名`

选项：-f选项对已经存在文件系统的分区强制格式化

```sh
[root@localhost ~]# mkfs.xfs /dev/sdb1			#格式化为XFS格式
meta-data=/dev/sdb1              isize=512    agcount=4, agsize=65536 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0, sparse=0
data     =                       bsize=4096   blocks=262144, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal log           bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@localhost ~]# mkdir /sdb1							#创建挂载点
[root@localhost ~]# mount /dev/sdb1 /sdb1/	#挂载sdb1分区到/sdb1目录下
[root@localhost ~]# df -h										#查看文件系统
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   50G  1.7G   49G    4% /
devtmpfs                 7.8G     0  7.8G    0% /dev
tmpfs                    7.8G     0  7.8G    0% /dev/shm
tmpfs                    7.8G  9.2M  7.8G    1% /run
tmpfs                    7.8G     0  7.8G    0% /sys/fs/cgroup
/dev/loop2                50M   50M     0  100% /var/lib/snapd/snap/certbot/1042
/dev/loop1                62M   62M     0  100% /var/lib/snapd/snap/core20/904
/dev/loop0               100M  100M     0  100% /var/lib/snapd/snap/core/10908
/dev/loop3               100M  100M     0  100% /var/lib/snapd/snap/core/10859
/dev/sda1               1014M  145M  870M   15% /boot
/dev/mapper/centos-home  241G   36M  241G    1% /home
tmpfs                    1.6G     0  1.6G    0% /run/user/0
/dev/sdb1               1014M   33M  982M    4% /sdb1
```

#### 卸载

- 命令：umount

- 使用格式： 

  umount 挂载点，卸载挂载点

  或

  umount 设备路径，卸载设备，需要指定设备绝对路径

- 如果提示“目标忙”无法卸载，一般是因为当前的工作目录处在挂载点目录

  ```sh
  [root@localhost sdb1]# umount /sdb1
  umount: /sdb1：目标忙。
  ```

  

```sh
[root@localhost /]# umount /sdb1
[root@localhost /]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
/dev/mapper/centos-root   50G  1.7G   49G    4% /
devtmpfs                 7.8G     0  7.8G    0% /dev
tmpfs                    7.8G     0  7.8G    0% /dev/shm
tmpfs                    7.8G  9.2M  7.8G    1% /run
tmpfs                    7.8G     0  7.8G    0% /sys/fs/cgroup
/dev/loop2                50M   50M     0  100% /var/lib/snapd/snap/certbot/1042
/dev/loop1                62M   62M     0  100% /var/lib/snapd/snap/core20/904
/dev/loop0               100M  100M     0  100% /var/lib/snapd/snap/core/10908
/dev/loop3               100M  100M     0  100% /var/lib/snapd/snap/core/10859
/dev/sda1               1014M  145M  870M   15% /boot
/dev/mapper/centos-home  241G   36M  241G    1% /home
tmpfs                    1.6G     0  1.6G    0% /run/user/0
```

#### 写入配置文件，实现开机自动挂载

```
[root@localhost /]# vim /etc/fstab	#在配置文件最后写入以下挂载信息
/dev/sdb1       /sdb1   xfs     defaults        0       0
```

- 开机自动挂载内容含义


## 参考链接
[如何检查你的计算机使用的是 UEFI 还是 BIOS | Linux 中国](https://blog.csdn.net/F8qG7f9YD02Pe/article/details/79551663)

[Linux系统MBR分区和GPT分区的区别](https://cloud.tencent.com/developer/article/1691006)

[硬盘基本知识（磁头、磁道、扇区、柱面）](http://events.jianshu.io/p/ffc2056766b6)