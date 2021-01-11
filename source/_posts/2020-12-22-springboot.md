---
layout: post
title: 工作中的Spring
description: Spring常见问题
category: blog
date: 2020-01-07 13:50:39
---

## Springboot项目上传大文件所需配置
application.properties配置：
spring.servlet.multipart.max-file-size=128MB
spring.servlet.multipart.max-request-size=128MB
spring.servlet.multipart.enabled=true

## 报错:The last packet successfully received from the server was 2,272 milliseconds ago. The last packet sent successfully to the server was 2,258 milliseconds ago.
解决方法:将useSSL=true改为useSSL=false    



















