---
title: 手写实现基于Redis的Starter
date: 2021-08-31 21:28:39
description: Spring系列之手写Starter
tags: [编程, 过去,Spring]
category:
    - 100 学习类
    - 110 编程
    - 111 Java

---



## 背景
前俩天看Spring的自动装配，它是Starter的基础，也是SpringBoot的核心，就是自动将Bean装配到IoC容器中，顺便学习了一下实现一个Starter，聊以本文以记之。

## Starter组件的功能
1. 涉及相关组件的Jar包依赖
2. 自动实现Bean的装配
3. 自动声明并加载application.yml文件中的属性配置

## Starter的命名规范
1. 官方命名格式：spring-boot-starter-模块名，比如spring-boot-starter-test
2. 自定义命名：模块名-spring-boot-starter，比如redis-spring-boot-starter

## 实现基于Redis的Starter

### 创建一个工程，命名为redis-spring-boot-starter

### 添加Jar包以来，Redission提供了在Java中操作Redis的功能，此处我们使用Redission
工程中添加redission依赖

```xml
<dependency>
  <groupId>org.redisson</groupId>
  <artifactId>redisson</artifactId>
  <version>3.11.1</version>
</dependency>
```

### 定义属性类，实现在application.yml中配置Redis的连接参数。

@ConfigurationProperties这个注解的作用是把当前类中的属性和配置文件（yml/properties）中的配置进行绑定，并且前缀是gp.redission

```java
package com.raines.redisstarter;

import org.springframework.boot.context.properties.ConfigurationProperties;

@ConfigurationProperties(prefix = "gp.redission")
public class RedissionProperties {
    private String host = "localhost";
    private String password;
    private int port = 6379;
    private int timeout;
    private boolean ssl;

    public String getHost() {
        return host;
    }

    public void setHost(String host) {
        this.host = host;
    }

    public String getPassword() {
        return password;
    }

    public void setPassword(String password) {
        this.password = password;
    }

    public int getPort() {
        return port;
    }

    public void setPort(int port) {
        this.port = port;
    }

    public int getTimeout() {
        return timeout;
    }

    public void setTimeout(int timeout) {
        this.timeout = timeout;
    }

    public boolean isSsl() {
        return ssl;
    }

    public void setSsl(boolean ssl) {
        this.ssl = ssl;
    }
}
```

### 定义需要自动装配的配置类，主要是把RdissionClient装配到IoC容器

@ConditionalOnClass注解表示一个条件，在classpath下存在Redission这个类的时候，RedissonAutoConfiguration才会实现自动装配。

```java
package com.raines.redisstarter;

import lombok.extern.slf4j.Slf4j;
import org.redisson.Redisson;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.redisson.config.SingleServerConfig;
import org.springframework.boot.autoconfigure.condition.ConditionalOnClass;
import org.springframework.boot.context.properties.EnableConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.util.StringUtils;

@Configuration
@ConditionalOnClass(Redisson.class)
@EnableConfigurationProperties(RedissionProperties.class)
@Slf4j
public class RedissonAutoConfiguration {

    @Bean
    RedissonClient redissonClient(RedissionProperties redissionProperties){
        log.info("redisson starter bean initialization");
        Config config = new Config();
        String prefix = "redis://";
        if (redissionProperties.isSsl()){
            prefix="rediss://";
        }
        SingleServerConfig singleServerConfig = config.useSingleServer()
                .setAddress(prefix+redissionProperties.getHost()+":"+redissionProperties.getPort())
                .setConnectTimeout(redissionProperties.getTimeout());
        if (!StringUtils.isEmpty(redissionProperties.getPassword())){
            singleServerConfig.setPassword(redissionProperties.getPassword());
        }
        return Redisson.create(config);
    }

}
```

### 在resources下创建META-INF/spring.factories文件，使得Spring Boot程序可以扫描该文件完成自动装配

spring.factories文件添加如下key、value

关于为什么要写spring.factories文件请看附录讲解

```yaml
org.springframework.boot.autoconfigure.EnableAutoConfiguration=\
  com.raines.redisstarter.RedissonAutoConfiguration
```

### 使用

### 新建Spring Boot web项目，添加redis-spring-boot-starter依赖

pom.xml文件添加依赖，groupId使用您自己的

```xml
<dependency>
	<groupId>com.raines</groupId>
	<artifactId>redis-spring-boot-starter</artifactId>
	<version>1.0</version>
</dependency>
```

### 在application.yml中配置host和port，属性会自动绑定到RedissionProperities中定义的属性上

yml文件添加配置

```yaml
gp:
  redission:
    host: 172.16.100.21
    port: 6379
```

## 测试

### 新建controller，使用RedissonClient进行测试

```java
package com.raines.javaadvanced.redisstarter;

import org.redisson.api.RBucket;
import org.redisson.api.RedissonClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
public class RedisDemoController {

    @Resource
    private RedissonClient client;

    @GetMapping("/redissonDemo/{key}")
    public Object getUserInfo(@PathVariable String key) {
        RBucket<String> bucket = client.getBucket("key");
        bucket.set(key);
        return bucket.get();
    }

}

```

### 调用localhost/redissonDemo/raines进行测试

至此，一个简单的starter完成。

## 附录

### 为什么要写spring.factories文件？

让我们带着问题寻找答案：明明自动配置的类已经打上了@Configuration的注解，为什么还要写spring.factories文件？

先从@SpringBootApplication注解说起，这个注解有两个重要的注解：@EnableAutoConfiguration和@ComponentScan

#### @ComponentScan

@ComponentScan注解的作用是扫描@SpringBootApplication所在的Application类（即spring-boot项目的入口类）所在的包（basepackage）下所有的@component注解（或拓展了@component的注解）标记的bean，并注册到spring容器中。

那么，在spring-boot项目中pom文件里面添加的依赖中的bean（spring-boot项目外的bean）是如何注册到spring-boot项目的spring容器中的呢？这就需要讨论下一个注解了。

#### @EnableAutoConfiguration























