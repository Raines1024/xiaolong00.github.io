---
title: 记一次利用shell减少重复劳动的尝试
date: 2021-01-18 08:17:39
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 113 Linux
---



## 背景

一直在用 git 命令行在添加文件、提交、更新、推送，耗时且毫无意义，遂想偷懒之法。故了解shell脚本以破之。

## 所需基础知识

1. 创建文件   `touch demo.sh`
2. 使用 vim 编辑文件   `vim demo.sh`
3. 赋予脚本执行权限   `chmod +x ./test.sh`
4. 执行脚本  `./test.sh`   或   `sh test.sh`        

## 实践出真知--推送本地所有变更文件到git远端分支

- 精简版shell

  ```sh
  #!/bin/bash
  cd /Users/raines/Desktop/my/xiaolong00.github.io
  git status
  git add .
  git commit -m test
  git push
  ```

  "#!" 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种 Shell。

  其他命令查阅git基础知识，小龙很久前也写过git的基础命令。

  由于该shell只会把本地所有变更的文件全都推送，甚至都不给你反悔的余地，then--我决定改良一下它，让它告诉我有哪些文件改动了，会不会有误改的文件，给自己留一个反悔的机会。

- 留有余地的shell

  ```sh
  #!/bin/bash
  cd /Users/raines/Desktop/my/xiaolong00.github.io
  git status
  read -r -p "Are You Sure? [Y/n] " input
  
  case $input in
      [yY][eE][sS]|[yY])
  		git add .
  		git commit -m a
  		git push
  		;;
  
      [nN][oO]|[nN])
  		echo "No"
         	;;
  
      *)
  		echo "Invalid input..."
  		exit 1
  		;;
  esac
  ```

  这个shell就人性化多了，它会拿着需要更改的文件询问你，Are You Sure? 如果你y（是），则推送到远程分支；或者你发现自己错了，选n（否），那就用"echo" 命令用于向窗口输出文本“No”，然后退出；或者你睡着了，误触了其他按键，则提醒你无效，退出脚本。但是--有人感觉自己睡不着，但是单身这么多年手速实在太快，误触了就很伤脑筋，我还给你准备了个预选方案：

- #### 提示进行确认（输入正常退出，输入错误则需重新输入）

  ```sh
  #!/bin/bash
  
  while true
  do
  	read -r -p "Are You Sure? [Y/n] " input
  
  	case $input in
  	    [yY][eE][sS]|[yY])
  			echo "Yes"
  			exit 1
  			;;
  
  	    [nN][oO]|[nN])
  			echo "No"
  			exit 1	       	
  			;;
  
  	    *)
  			echo "Invalid input..."
  			;;
  	esac
  done
  
  ```

  

## 彩蛋

一般新建一个文件扩展名为 sh（sh代表shell），扩展名并不影响脚本执行，见名知意就好。比如你用 php 写 shell 脚本，扩展名就用 php 好了。

## 突发奇想--把大象放到冰箱

1.把大象放进冰箱要几个步骤?

答:3个.先打开冰箱门,再把大象放进去,再关上冰箱门.

2.把长颈鹿放进冰箱要几个步骤?

答:4个.先打开冰箱门,再把大象拿出来,再把长颈鹿放进去,然后关上冰箱门.

3.森林里要开森林大会,规定所有的动物都要去参加,可是有一样动物没有到,你知道是什么动物吗?

答:长颈鹿.因为它被关在冰箱里.

4.有一条河,里面有很多凶狠的鳄鱼,河上又没有桥,有一个人却成功过河了,你知道他是怎么过去的吗?

答:游过去的.因为鳄鱼去开森林大会了.















































