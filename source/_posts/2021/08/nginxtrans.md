---
title: Nginx转发域外地图
date: 2021-08-28 22:17:39
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 112 Linux

---



## 背景

由于国内已经无法访问谷歌中国地图，鉴于公司需要，遂购买香港服务器转发。

## 问题

不知为何，直接配置mapproxy（见配置文件）会提示404，无法正常使用。然而如此转发后恢复正常，查看官方文档未找到原因，奇哉怪也。

## 瓦片实例

http://47.75.118.xx/vt/lyrs=s&hl=zh-CN&gl=CN&x=871364&y=409384&z=20&s=Gali

## nginx配置文件

```
user root;
worker_processes  2;

events {
    worker_connections  2048;
}

http {
    include       mime.types;
    default_type  application/octet-stream;
    client_max_body_size 200m;

    sendfile        on;
    keepalive_timeout  75;
    proxy_http_version 1.1;
    proxy_set_header Connection "";

    #proxy_temp_path ../proxy_cache/tianditu_temp;
    proxy_cache_path /nginx_cache levels=1:2 keys_zone=mycache:1000m max_size=20g inactive=7d use_temp_path=off;

    upstream mapproxy {
        server mt0.google.cn;
        server mt1.google.cn;
        server mt2.google.cn;
        server mt3.google.cn;
    }

    upstream mapproxy1 {
        server 172.21.238.xxx:8080;
        server 172.21.238.xxx:8081;
        server 172.21.238.xxx:8082;
        server 172.21.238.xxx:8083;
    }

    server {
        listen        80;
        server_name   172.16.70.12;
        #client_max_body_size 200M;
        #proxy_read_timeout 600;
        location / {
            proxy_cache mycache;
            add_header X-Cache-Status $upstream_cache_status;
            proxy_cache_revalidate on;
            proxy_cache_min_uses 2;
            proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
            proxy_cache_lock on;
            #proxy_pass http://mt2.google.cn;
            #proxy_pass https://www.google.com.hk;
            proxy_pass http://mapproxy1;
        }
   }

    server {
        listen        8082;
        server_name	172.16.70.xx;
        #client_max_body_size 200M;
        #proxy_read_timeout 600;
        location / {
            #proxy_cache mycache;
            #add_header X-Cache-Status $upstream_cache_status;
	    #proxy_cache_revalidate on;
	    #proxy_cache_min_uses 3;
	    #proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
	    #proxy_cache_lock on;

            proxy_pass http://mt2.google.cn;
            #proxy_pass https://www.google.com.hk;
        }
   }

    server {
        listen        8080;
        server_name     172.16.70.xx;
        #client_max_body_size 200M;
        #proxy_read_timeout 600;
        location / {
            #proxy_cache mycache;
            #add_header X-Cache-Status $upstream_cache_status;
	    #proxy_cache_revalidate on;
	    #proxy_cache_min_uses 3;
	    #proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
	    #proxy_cache_lock on;
            proxy_pass http://mt0.google.cn;
            #proxy_pass https://www.google.com.hk;
        }   
   }

    server {
        listen        8081;
        server_name     172.16.70.xx;
        #client_max_body_size 200M;
        #proxy_read_timeout 600;
        location / {
            #proxy_cache mycache;
            #add_header X-Cache-Status $upstream_cache_status;
	    #proxy_cache_revalidate on;
	    #proxy_cache_min_uses 3;
	    #proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
	    #proxy_cache_lock on;
            proxy_pass http://mt1.google.cn;
            #proxy_pass https://www.google.com.hk;
        }   
   }

    server {
        listen        8083;
        server_name     172.16.70.xx;
        #client_max_body_size 200M;
        #proxy_read_timeout 600;
        location / {
            #proxy_cache mycache;
            #add_header X-Cache-Status $upstream_cache_status;
	    #proxy_cache_revalidate on;
	    #proxy_cache_min_uses 3;
	    #proxy_cache_use_stale error timeout updating http_500 http_502 http_503 http_504;
	    #proxy_cache_lock on;
            proxy_pass http://mt3.google.cn;
            #proxy_pass https://www.google.com.hk;
        }   
   }
}


```



## 参考链接
[Nginx upstream模块文档](http://nginx.org/en/docs/http/ngx_http_upstream_module.html)





















