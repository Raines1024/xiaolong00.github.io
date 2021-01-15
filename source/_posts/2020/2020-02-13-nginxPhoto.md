---
layout: post
title: 使用nginx搭建图片服务器
description: 
category: blog
date: 2020-01-07 13:50:39
---

## nginx常用命令（路径替换为服务器具体路径）
- 查看nginx的配件文件是否正确  
/usr/local/nginx/sbin/nginx -t
- 重启nginx  
方式一：/etc/init.d/nginx restart   
方式二：进入nginx可执行目录sbin下，输入命令./nginx -s reload

## nginx.conf配置文件

### 准备：vim中的翻页命令
- 整页翻页 ctrl-f ctrl-b  
f就是forword b就是backward  

- 翻半页  
ctrl-d ctlr-u  
d=down u=up  

- 滚一行  
ctrl-e ctrl-y

- zz 让光标所在的行居屏幕中央  
- zt 让光标所在的行居屏幕最上一行 t=top  
- zb 让光标所在的行居屏幕最下一行 b=bottom  

### 解读
- Nginx默认是不允许列出整个目录的。  
如需此功能，打开nginx.conf文件，在location server 或 http段中加入    
autoindex on;  
另外两个参数最好也加上去:  
autoindex_exact_size off;  
默认为on，显示出文件的确切大小，单位是bytes。  
改为off后，显示出文件的大概大小，单位是kB或者MB或者GB  
autoindex_localtime on;  
默认为off，显示的文件时间为GMT时间。  
改为on后，显示的文件时间为文件的服务器时间  
- 访问图片403问题  
    - nginx.conf的nobody修改为进程启动的用户名称    
    - 权限问题，如果nginx没有web目录的操作权限，也会出现403错误：修改web目录的读写权限，或者是把nginx的启动用户改成目录的所属用户，重启Nginx即可解决  chmod -R 777 /data  
- 访问路径  
在server.location中监听访问路径  
例如，在以下配置文件下：访问  域名/photo/demo.jpg   
会去寻找服务器路径 /data/jar/photo下的demo.jpg文件；  
而访问  域名/photo/  
会显示photo文件夹的目录

### 配置文件详情

    #user  nobody;
    worker_processes  1;
    
    #error_log  logs/error.log;
    #error_log  logs/error.log  notice;
    #error_log  logs/error.log  info;
    
    #pid        logs/nginx.pid;
    
    
    events {
        worker_connections  1024;
    }
    
    
    http {
        include       mime.types;
        default_type  application/octet-stream;
    
        #log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
        #                  '$status $body_bytes_sent "$http_referer" '
        #                  '"$http_user_agent" "$http_x_forwarded_for"';
    
        #access_log  logs/access.log  main;
    
        sendfile        on;
        #tcp_nopush     on;
    
        #keepalive_timeout  0;
        keepalive_timeout  65;
    
        #gzip  on;
    
        server {
            listen       80;
            server_name  localhost;
    
            #charset koi8-r;
    
            #access_log  logs/host.access.log  main;
    
            location / {
                root   html;
                index  index.html index.htm;
            }
            location /photo/ {
                root   /data/jar/;
                autoindex on;
            autoindex_localtime on;
                    autoindex_exact_size on;
            charset utf-8;
            }
    
            #error_page  404              /404.html;
    
            # redirect server error pages to the static page /50x.html
            #
            error_page   500 502 503 504  /50x.html;
            location = /50x.html {
                root   html;
            }
    
            # proxy the PHP scripts to Apache listening on 127.0.0.1:80
            #
            #location ~ \.php$ {
            #    proxy_pass   http://127.0.0.1;
            #}
    
            # pass the PHP scripts to FastCGI server listening on 127.0.0.1:9000
            #
            #location ~ \.php$ {
            #    root           html;
            #    fastcgi_pass   127.0.0.1:9000;
            #    fastcgi_index  index.php;
            #    fastcgi_param  SCRIPT_FILENAME  /scripts$fastcgi_script_name;
            #    include        fastcgi_params;
            #}
    
            # deny access to .htaccess files, if Apache's document root
            # concurs with nginx's one
            #
            #location ~ /\.ht {
            #    deny  all;
            #}
        }
    
    
        # another virtual host using mix of IP-, name-, and port-based configuration
        #
        #server {
        #    listen       8000;
        #    listen       somename:8080;
        #    server_name  somename  alias  another.alias;
    
        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}
    
    
        # HTTPS server
        #
        #server {
        #    listen       443 ssl;
        #    server_name  localhost;
    
        #    ssl_certificate      cert.pem;
        #    ssl_certificate_key  cert.key;
    
        #    ssl_session_cache    shared:SSL:1m;
        #    ssl_session_timeout  5m;
    
        #    ssl_ciphers  HIGH:!aNULL:!MD5;
        #    ssl_prefer_server_ciphers  on;
    
        #    location / {
        #        root   html;
        #        index  index.html index.htm;
        #    }
        #}
    
    }














