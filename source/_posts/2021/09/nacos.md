---
title: Nacos使用
date: 2021-09-03 21:28:39
description: 
tags: [编程, 过去]
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

### 添加Jar包依赖，Redission提供了在Java中操作Redis的功能，此处我们使用Redission
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



### 注解@ConditionalOnClass(X.class),X不存在时的探究

@ConditionalOnClass通常与@Configuration 结合使用，意思是当classpath中存在某类时满足条件

看到这个注解的时候是有点疑问的，注解本来就是判断X.class是否存在，问题在于X.class不存在的话，无法import引入X类，都无法通过编译期；如果能引入X类，则classpath中肯定有这个类，被注解的类肯定会自动装配。

让我们带着问题寻找答案。

Maven的依赖中有一个`optional`标签，先来说一下这个标签的作用。

假如你的Project A的某个依赖D添加了`<optional>true</optional>`，当别人通过pom依赖Project A的时候，D不会被传递依赖进来。

用我们上面的项目举例子：redis-spring-boot-starter项目中打开pom.xml文件，添加redisson依赖标签`<optional>true</optional>`，如下：

```xml
<dependency>
	<groupId>org.redisson</groupId>
	<artifactId>redisson</artifactId>
	<version>3.11.1</version>
	<optional>true</optional>
</dependency>
```

刷新maven依赖后我们回到测试项目，发现`RedisDemoController`类已找不到redisson的class，把该类注释，启动项目会发现自动装配已失效，因为classpath中没有redisson，所以不会自动装配redisson的bean；此时我们在测试项目pom.xml文件中添加redisson依赖如下，重启项目，发现classpath中有redisson，redisson的bean已成功加载。

```xml
<dependency>
	<groupId>org.redisson</groupId>
	<artifactId>redisson</artifactId>
	<version>3.11.1</version>
</dependency>
```



### 为什么要写spring.factories文件？

让我们带着问题寻找答案：明明自动配置的类已经打上了@Configuration的注解，为什么还要写spring.factories文件？

先从@SpringBootApplication注解说起，这个注解有两个重要的注解：@EnableAutoConfiguration和@ComponentScan

#### @ComponentScan

@ComponentScan注解的作用是扫描@SpringBootApplication所在的Application类（即spring-boot项目的入口类）所在的包（basepackage）下所有的@component注解（或拓展了@component的注解）标记的bean，并注册到spring容器中。

那么，在spring-boot项目中pom文件里面添加的依赖中的bean（spring-boot项目外的bean）是如何注册到spring-boot项目的spring容器中的呢？这就需要讨论下一个注解了。

#### @EnableAutoConfiguration

以后再填坑吧，反正就是去加载spring.factories文件



## classpath

在Java中，我们经常听到`classpath`这个名词。比如说刚才提到的`@ConditionalOnClass(X.class)`注解，那么到底什么是`classpath`?

`classpath`是JVM用到的一个环境变量，它用来指示JVM如何搜索`class`。

因为Java是编译型语言，源码文件是`.java`，而编译后的`.class`文件才是真正可以被JVM执行的字节码。因此，JVM需要知道，如果要加载一个`abc.xyz.Hello`的类，应该去哪搜索对应的`Hello.class`文件。

所以，`classpath`就是一组目录的集合，它设置的搜索路径与操作系统相关。例如，在Linux系统上，用`:`分隔，比如这样：

```
-classpath /Users/raines/IdeaProjects/Lovol/learn/comm/target/classes:/Users/raines/rainesComm/mavenRepository/org/springframework/boot/spring-boot-starter/2.1.3.RELEASE/spring-boot-starter-2.1.3.RELEASE.jar
```

我们假设`classpath`是`.;C:\work\project1\bin;C:\shared`，当JVM在加载`abc.xyz.Hello`这个类时，会依次查找：

- <当前目录>\abc\xyz\Hello.class
- C:\work\project1\bin\abc\xyz\Hello.class
- C:\shared\abc\xyz\Hello.class

注意到`.`代表当前目录。如果JVM在某个路径下找到了对应的`class`文件，就不再往后继续搜索。如果所有路径下都没有找到，就报错。

`classpath`的设定方法有两种：

在系统环境变量中设置`classpath`环境变量，不推荐；

在启动JVM时设置`classpath`变量，推荐。

我们强烈*不推荐*在系统环境变量中设置`classpath`，那样会污染整个系统环境。在启动JVM时设置`classpath`才是推荐的做法。实际上就是给`java`命令传入`-classpath`或`-cp`参数：

```
java -classpath .;C:\work\project1\bin;C:\shared abc.xyz.Hello
```

或者使用`-cp`的简写：

```
java -cp .;C:\work\project1\bin;C:\shared abc.xyz.Hello
```

没有设置系统环境变量，也没有传入`-cp`参数，那么JVM默认的`classpath`为`.`，即当前目录：

```
java abc.xyz.Hello
```

上述命令告诉JVM只在当前目录搜索`Hello.class`。

在IDE中运行Java程序，IDE自动传入的`-cp`参数是当前工程的`bin`目录和引入的jar包。

通常，我们在自己编写的`class`中，会引用Java核心库的`class`，例如，`String`、`ArrayList`等。这些`class`应该上哪去找？

有很多“如何设置classpath”的文章会告诉你把JVM自带的`rt.jar`放入`classpath`，但事实上，根本不需要告诉JVM如何去Java核心库查找`class`，JVM怎么可能笨到连自己的核心库在哪都不知道？

 不要把任何Java核心库添加到classpath中！JVM根本不依赖classpath加载核心库！

更好的做法是，不要设置`classpath`！默认的当前目录`.`对于绝大多数情况都够用了。



## 参考链接

[廖雪峰的官方网站](https://www.liaoxuefeng.com/wiki/1252599548343744/1260466914339296)











