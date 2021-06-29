---
title: nginx报错：nginx读取响应时过早关闭连接
date: 2021-06-29 21:27:39
description: nginx报错导致无法访问接口
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java



---

## nginx报错无法访问接口

### 报错信息

```
2021/06/23 15:25:38 [error] 10902#0: *1779657 upstream prematurely closed connection while reading response header from upstream, client: 172.16.70.181, server: 172.16.70.12, request: "POST /farm/api/farmmenu/menuTreeListByRole HTTP/1.0", upstream: "http://172.16.70.12:8091/farm/api/farmmenu/menuTreeListByRole", host: "172.16.70.12", referrer: "http://ifarming.xxx.com:8081/farm/"
```

### 解决方案


nginx.conf文件中server代码块添加以下内容
        proxy_read_timeout 600s;
        proxy_connect_timeout 90s;

#### 注意

添加时需要把源nginx服务配置文件及目的nginx服务配置文件同步修改

## 参考链接

[stackoverflow：nginx读取响应时过早关闭连接](https://stackoverflow.com/questions/36488688/nginx-upstream-prematurely-closed-connection-while-reading-response-header-from)