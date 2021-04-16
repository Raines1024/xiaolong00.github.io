---
title: Jenkins配置Gitlab钩子自动发布
date: 2021-04-16 19:01:39
description: 测试环境自动发布的良好实践
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 116 常用工具
---

## 背景

为什么说Jenkins配置Gitlab钩子自动发布是良好实践呢？

它可能有这些问题：

1. gitlab提交记录频繁，而不是每次都需要发布，而每次发布期间都导致系统无法正常测试
2. 未经检查的代码提交后无法运行导致测试环境宕机

## Jenkins配置Gitlab钩子

### 安装Jenkins插件

安装Jenkins插件：*Manage Jenkins* -> *Manage Plugins* -> *Available*搜索安装以下插件

Gitlab Hook plugin、Git plugin、Gitlab Authentication plugin

如图：

![安装插件](https://cdn.jsdelivr.net/gh/Xiaolong00/raines-photo@master/blog/xxx.4oe8x2wv20a0.png)

###在Jenkins的Job中配置获取钩子信息

  在Jenkins的每个任务中，在钩子的*Build Triggers*配置处获取到Jenkins生成的钩子回调地址，并且可以配置*token*，由于图片太小无法显示，此次配置的token为123456。后面需要将这两个信息copy到GitLab上的配置中，然后Save即可。如图：

![配置钩子](https://cdn.jsdelivr.net/gh/Xiaolong00/raines-photo@master/blog/xxx.4uld7v7b9wy0.png)

### 在GitLab端配置钩子

在GitLab端可以配置全局的钩子，也可以在项目上配置钩子（推荐直接在项目上配置，可以细粒度的配置）。进入项目后选择*Settings* -> *Integrations*,就进入了钩子配置页面。每一个项目可以配置多个钩子，根据不同的触发条件进行配置。比如我们之前项目会安装测试环境和开发环境，根据不同的项目分支（Branch）进行配置和条件触发。

1. 将jenkins上的钩子地址和Secret token拷贝到上面，

2. 并且配置触发条件，我们一般选择`Push events`往仓库中进行push操作，并且可以配置分支（如：dev）

![GitLab配置钩子](https://cdn.jsdelivr.net/gh/Xiaolong00/raines-photo@master/blog/xxx.rmp6avywru8.png)

点击*Add webhook*，这样就已经配置好了钩子。

- 注：此处可能点击添加钩子后报错：Url is blocked: Requests to the local network are not allowed

  解决方案：登陆管理员账号，进入 *Admin area* => *Settings* => *Network* ，然后点击*Outbound requests*右边的“expand”按钮，如图：

  ![解决钩子报错](https://cdn.jsdelivr.net/gh/Xiaolong00/raines-photo@master/blog/xxx.3s0c2rvbhac0.png)

  在上图红框打勾，并点击*Save changes*按钮即可！

### 页面测试钩子

添加好钩子后，在下面的列表中查看是否配置好，并且可以点击Test 中的事件进行触发钩子测试。如图：

![测试钩子](https://cdn.jsdelivr.net/gh/Xiaolong00/raines-photo@master/blog/image.xz1cdtezvsg.png)

点击后，如果正确会提示success200

再看看Jenkins上是否已经触发了：

![成功](https://cdn.jsdelivr.net/gh/Xiaolong00/raines-photo@master/blog/xxx.35tecigk3qg0.png)







