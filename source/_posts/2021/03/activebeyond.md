---
title: MacOS有效解决Beyond Compare “这个授权密钥已被吊销”的办法
date: 2021-03-27 20:01:39
description: Beyond Compare是一款专业的文件对比工具
tags: [应用,过去]
category:
    - 200 工作类
    - 230 工作技能
    - 236 使用软件

---

## Beyond Compare

使用Beyond Compare来进行文件对比及操作是提高生产力的最佳实践之一，请支持正版。

1. mac下进入应用文件夹

   > ~/Library/Application Support/Beyond Compare

2. 修改BCState.xml文件

   替换TCheckForUpdatesState标签

   ```xml
   <TCheckForUpdatesState>
       <Build Value="24545"/>
   </TCheckForUpdatesState>
   ```

3. 修改BCSessions.xml文件

   替换BCSessions标签

   ```xml
   <BCSessions Version="1" MinVersion="1">
   </BCSessions>
   ```

## 转载

简书：[转载地址](https://www.jianshu.com/p/c69639f15231)

本文仅为学习用，请支持正版，如有侵权请联系站长删除。