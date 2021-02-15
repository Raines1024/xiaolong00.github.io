---
title: maven打包的jar指定启动类
date: 2021-02-09 08:20:39
description:
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java
---

## 背景

 某些操作需要在服务器上执行某个类的main方法，比如调服务间内网的消息中间件等等。而此等操作大多都用到第三方以来，且大部分时间当作一个即兴的脚本开发，并不需要引入springboot，此时可以通过在pom中指定启动类来解决这个问题。

## 解决方案

第一种：
     如果你的pom是继承spring-boot-starter-parent的话，只需要在pom的root如下指定就行

```xml
    <properties>
        <!-- 指定启动类 -->
        <start-class>com.besttop.BaseServerApplication</start-class>
    </properties>
```

第二种：

​    如果你的pom中没有继承Spring-boot-start-parent，那么需要通过如下配置实现。

```xml
<build>  
    	<plugins>  
            <plugin>
                <groupId>org.springframework.boot</groupId>
                <artifactId>spring-boot-maven-plugin</artifactId>
                <version>1.3.5.RELEASE</version>
                <configuration>
                    <!-- 指定启动类 -->
                    <mainClass>com.lovol.Main</mainClass>
                </configuration>
                <executions>
                    <execution>
                      <goals>
                        <goal>repackage</goal>
                      </goals>
                    </execution>
                </executions>
            </plugin>
    	</plugins>  
</build>  
```



































