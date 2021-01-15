---
layout: post
title: Ubuntu 桌面版
description: Ubuntu Desktop
category: blog
date: 2020-01-07 13:50:39
---


## 为什么选择Ubuntu 
之前一直使用MBP作为开发的主力机使用，因公司原因，使用thinkpad作为日常开发机器，回归windows后使用一段时间感触颇深，实在是大大降低了工作效率，
主要原因有如下：命令行下不支持linux命令，没有tab补足git的命令的人性化设计；对docker的支持不完善；时常有弹窗广告，实在不胜其烦，广告事小，打断思路、惊扰神经事大。
实在忍无可忍，无奈之下决定体验一下Ubuntu的桌面版。  
Ubuntu还是大体让人满意的：最满意的当数命令行、完美支持docker、毫无弹窗广告。不过其不便也很明显：软件支持少，微信qq等等就放弃吧，不要曲线救国耗费心力了。   
是以，选择使用Ubuntu作为windows的虚拟机，用以当作开发的主力机。  
dd
## 软件
- Ubuntu上一个炫酷的终端程序--guake https://zhuanlan.zhihu.com/p/40684802  
    安装命令sudo apt install guake  

- ubuntu16.04下安装如何安装.deb安装包   

```
在Ubuntu下安装deb包需要使用dpkg命令.  
Dpkg 的普通用法：   
1、sudo dpkg -i <package.deb>  
安装一个 Debian 软件包，如你手动下载的文件。  
2、sudo dpkg -c <package.deb>  
列出 <package.deb> 的内容。  
3、sudo dpkg -I <package.deb>  
从 <package.deb> 中提取包裹信息。  
4、sudo dpkg -r <package>  
移除一个已安装的包裹。  
5、sudo dpkg -P <package>  
完全清除一个已安装的包裹。和 remove 不同的是，remove 只是删掉数据和可执行文件，purge 另外还删除所有的配制文件。  
6、sudo dpkg -L <package>  
列出 <package> 安装的所有文件清单。同时请看 dpkg -c 来检查一个 .deb 文件的内容。  
7、sudo dpkg -s <package>  
显示已安装包裹的信息。同时请看 apt-cache 显示 Debian 存档中的包裹信息，以及 dpkg -I 来显示从一个 .deb 文件中提取的包裹信息。  
8、sudo dpkg-reconfigure <package>  
重新配制一个已经安装的包裹，如果它使用的是 debconf (debconf 为包裹安装提供了一个统一的配制界面)。  
如果安装过程中出现问题,可以先使用命令:  
sudo apt-get update  
更新后再执行相应操作。  
```

## 快捷键
作为之前mbp的无鼠标开发人士，不得不吐槽thinkpad的触控板实在是渣渣。   
https://zhuanlan.zhihu.com/p/45535756   

## 日常使用

### 远程连接
- 安装ssh  
sudo apt install ssh  
- 开启  
service ssh start


### git 使用
- 修改git默认编辑器为Vim  
git config --global core.editor vim    
- git保存仓库的账号密码    
git config --global credential.helper store   


























