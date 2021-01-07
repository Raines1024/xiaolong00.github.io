---
layout: post
title: Shell使用
description: 
category: blog
---

## zsh与bash切换
- zsh切换为bash  
chsh -s /bin/bash  
- bash切换为zsh  
chsh -s /bin/zsh  

### 查看当前运行的 Shell
echo $SHELL  

### 查看当前的 Linux 系统安装的所有 Shell
cat /etc/shells

## Bash 变量
env命令或printenv命令，可以显示所有环境变量。  

```
常见的环境变量。
BASHPID：Bash 进程的进程 ID。  
BASHOPTS：当前 Shell 的参数，可以用shopt命令修改。
DISPLAY：图形环境的显示器名字，通常是:0，表示 X Server 的第一个显示器。
EDITOR：默认的文本编辑器。
HOME：用户的主目录。
HOST：当前主机的名称。
IFS：词与词之间的分隔符，默认为空格。
LANG：字符集以及语言编码，比如zh_CN.UTF-8。
PATH：由冒号分开的目录列表，当输入可执行程序名后，会搜索这个目录列表。
PS1：Shell 提示符。
PS2： 输入多行命令时，次要的 Shell 提示符。
PWD：当前工作目录。
RANDOM：返回一个0到32767之间的随机数。
SHELL：Shell 的名字。
SHELLOPTS：启动当前 Shell 的set命令的参数，参见《set 命令》一章。
TERM：终端类型名，即终端仿真器所用的协议。
UID：当前用户的 ID 编号。
USER：当前用户的用户名。
```
set命令可以显示所有变量（包括环境变量和自定义变量），以及所有的 Bash 函数。  
查看单个环境变量的值，可以使用printenv命令或echo命令。  
注意，printenv命令后面的变量名，不用加前缀$。  






























