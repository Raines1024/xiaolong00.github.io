---
layout: post
title: docker入门（一）
description: Centos环境下安装及使用
category: blog
date: 2020-01-07 13:50:39
---

## 安装docker
1. 官网步骤安装  
2. yum安装  
yum install docker  
- 启动  
systemctl start docker  
- 查看安装的docker版本  
docker version  
3. https://www.cnblogs.com/hellxz/p/11044012.html

## springboot初使用

### 目录结构
    .
    ├── pom.xml
    └── src
        └── main
            ├── docker
            │   └── Dockerfile
            ├── java
            │   └── com
            │       └── docker
            │           └── DockerApplication.java
            └── resources
                └── application.properties

### pom.xml
- properties节点中设置docker镜像的前缀“springboot”：
  
- 加入maven插件“docker-maven-plugin”： 


    <build>
        <plugins>
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
            </plugin>
            <plugin>
                <groupId>com.spotify</groupId>
                <artifactId>docker-maven-plugin</artifactId>
                <version>0.4.3</version>
                <configuration>
                    <imageName>${docker.image.prefix}/${project.artifactId}</imageName>
                    <dockerDirectory>src/main/docker</dockerDirectory>
                    <resources>
                        <resource>
                            <targetPath>/</targetPath>
                            <directory>${project.build.directory}</directory>
                            <include>${project.build.finalName}.jar</include>
                        </resource>
                    </resources>
                </configuration>
            </plugin>
        </plugins>
    </build>
    

### 创建“src/main/docker/Dockerfile”文件：

    FROM frolvlad/alpine-oraclejdk8:slim
    VOLUME /tmp
    ADD spring12.jar app.jar
    RUN sh -c 'touch /app.jar'
    ENV JAVA_OPTS=""
    ENTRYPOINT [ "sh", "-c", "java $JAVA_OPTS -Djava.security.egd=file:/dev/./urandom -jar /app.jar" ]

FROM为使用哪个镜像

VOLUME为挂载路径

ADD为复制文件到镜像中

RUN为初始化时运行的命令

ENV为设置环境变量

ENTRYPOINT为启动时运行的命令

## 使用Docker部署服务

### 所需环境
jdk，maven  
安装成功标志：  
java -version  
mvn -version  

### 生成镜像并运行
1. 将src文件和pom放在任意文件夹下，执行命令  


    mvn package docker:build

2. 查看镜像  


    docker images
    
docker中存在的镜像、标签、镜像ID、已经创建的时间和大小，看下springboot/raines-learn 这个是在pom中<imageName>${docker.image.prefix}/${project.artifactId}</imageName>配置的，比较重要，因为它和接下来要讲的将镜像提交到DockerHub有着密切的联系。
    
3. 运行镜像  
启动服务  


    docker run -d -p 8765:8080 springboot/raines-learn 

* 解释下这个命令  
-d 代表后台运行  
-p 标识宿主机与docker服务的端口映射，注意谁前谁后：【宿主端口：docker内服务端口】  
springboot/raines-learn 就是启动镜像的名称，当然了使用IMAGE ID 也是可以的

4. 查看docker是否将服务启动成功


    docker ps

* 解释
CONTAINER ID 容器ID  
PORTS宿主与docker内部的服务映射  
NAMES 容器名称，跟容器ID对应，如果你不指定名称的话，docker会随机给你分配一个name  

## 推送到DockerHub  

### 地址
https://hub.docker.com/

### 登录
docker login -u[用户名] -p[密码]

### 推送到DockerHub
docker push springboot/raines-learn:latest  
latest是tag，相当于版本号  
[denied: requested access to the resource is denied]解决方案：  
docker tag : 标记本地镜像，将其归入某一仓库。  
docker tag [OPTIONS] IMAGE[:TAG] [REGISTRYHOST/][USERNAME/]NAME[:TAG]  
eg: docker tag springboot/raines-learn:latest 1150079039/raines-learn:v1
此处的1150079039需要改为你的用户名  

### 拉取镜像
docker pull springboot/raines-learn

## docker常用命令

### images

- 搜索image

    
    docker search image_name
- 下载image


    docker pull image_name
- 列出镜像列表


    docker images
- 删除images


    docker rmi <image id>
- 删除images id 为none的


    docker rmi $(docker images | grep "^<none>" | awk "{print $3}")
- 删除全部image


    docker rmi $(docker images -q)
- 显示一个镜像的历史


    docker history image_name


### container

- 列出当前所有正在运行的container


    docker ps
- 列出所有的container


    docker ps -a
- 列出最近一次启动的container


    docker ps -l
- 停止所有的container，这样才能够删除其中的images：


    docker stop $(docker ps -a -q)
- 删除所有container：


    docker rm $(docker ps -a -q)

## Docker 快速删除所有容器
- 查看运行容器  
docker ps  
- 查看所有容器  
docker ps -a  
- 进入容器,其中字符串为容器ID:   
docker exec -it d27bd3008ad9 /bin/bash  
1. 停用全部运行中的容器:  
docker stop $(docker ps -q)  
2. 删除全部容器：  
docker rm $(docker ps -aq)  
3. 一条命令实现停用并删除容器：  
docker stop $(docker ps -q) & docker rm $(docker ps -aq)  

## 删除所有none镜像
docker rmi $(docker images | grep "none" | awk '{print $3}') 






















