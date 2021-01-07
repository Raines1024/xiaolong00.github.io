---
layout: post
title: IDEA报错解决方案
description: 
category: blog
---

## Error:java: Compilation failed: internal java compiler error  
解决办法很简单：File-->Setting...-->Build,Execution,Deployment-->Compiler-->Java Compiler 设置相应Module的target bytecode version的合适版本（跟你jkd版本一致），这里我改成1.8版本的。   
这个错误很无趣，默认导入新项目时编译检查为JDK5特性  






























