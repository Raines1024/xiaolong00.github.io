---
layout: post
title: 乱七八槽的常用命令
date: 2021-01-07 13:50:39
tags: 工作
---

## redis、nginx


## redis 通配符 批量删除key

### redis 模糊搜索前缀为FILE的key
./redis-cli keys "FILE*"  
### Redis 中 DEL指令支持多个key作为参数进行删除但不支持通配符，无法通过通配符批量删除key，不过我们可以借助Linux的管道和 xargs 指令来完成这个动作。
比如要删除所有以FILE开头的key 可以这样实现：    
[root@dev_opayredis src]# ./redis-cli keys "FILE*"     
1) "user1"    
2) "user2"      
[root@dev_opayredis src]# ./redis-cli keys "FILE*" | xargs ./redis-cli del      
   (integer) 2        
   删除成功   

## nginx nginx.conf 更改配置client_max_body_size修改默认限制上传附件大小
client_max_body_size 200m;
### Nginx 上传大文件超时解决办法
- 情况如下：用nginx作代理服务器，上传大文件时（本人测试上传50m的文件），提示上传超时或文件过大。  
  原因是nginx对上传文件大小有限制，而且默认是1M。另外，若上传文件很大，还要适当调整上传超时时间。  
  解决方法是在nginx的配置文件下，加上以下配置：

```
client_max_body_size     50m; //文件大小限制，默认1m

client_header_timeout    1m;

client_body_timeout      1m;

proxy_connect_timeout     60s;

proxy_read_timeout      1m;

proxy_send_timeout      1m;
```

每个参数的意思：

client_max_body_size

限制请求体的大小，若超过所设定的大小，返回413错误。

client_header_timeout

读取请求头的超时时间，若超过所设定的大小，返回408错误。

client_body_timeout

读取请求实体的超时时间，若超过所设定的大小，返回413错误。

proxy_connect_timeout

http请求无法立即被容器(tomcat, netty等)处理，被放在nginx的待处理池中等待被处理。此参数为等待的最长时间，默认为60秒，官方推荐最长不要超过75秒。

proxy_read_timeout

http请求被容器(tomcat, netty等)处理后，nginx会等待处理结果，也就是容器返回的response。此参数即为服务器响应时间，默认60秒。

proxy_send_timeout

http请求被服务器处理完后，把数据传返回给Nginx的用时，默认60秒。

------------------------------------------------------------------------------------------------------------------------------------------------------------------------

nginx.conf

在nginx使用过程中，上传文件的过程中，通常需要设置nginx报文大小限制。避免出现413 Request Entity Too Large。

于是奇葩的问题被我们遇到了，详细配置请参考下面。我们的问题是，无论client_max_body_size设置在哪里，nginx －s reload后，依然一直报413.多次尝试reload，始终无效。最终决定kill 进程，restart，终于好了。

由此可见，nginx reload并不一定好使。有时候，为了保险起见。restart比较靠谱。

可以选择在http{ }中设置：client_max_body_size   20m;

也可以选择在server{ }中设置：client_max_body_size   20m;

还可以选择在location{ }中设置：client_max_body_size   20m;

三者到区别是：http{} 中控制着所有nginx收到的请求。而报文大小限制设置在server｛｝中，则控制该server收到的请求报文大小，同理，如果配置在location中，则报文大小限制，只对匹配了location 路由规则的请求生效。

## nginx 连接超时的配置
proxy_read_timeout  后端服务器处理请求的时间  
默认60s  
可设置在http, server, location块中。  











