---
title: feign使用示例
date: 2021-06-15 20:27:39
description: 单独使用feign作为http调用工具
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java



---



## 实战

### 添加maven依赖

添加feign-core依赖和jackson依赖

```xml
<!-- https://mvnrepository.com/artifact/io.github.openfeign/feign-core -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-core</artifactId>
    <version>11.2</version>
</dependency>
<!-- https://mvnrepository.com/artifact/io.github.openfeign/feign-jackson -->
<dependency>
    <groupId>io.github.openfeign</groupId>
    <artifactId>feign-jackson</artifactId>
    <version>11.2</version>
</dependency>
```

### 调用GET请求、路径传参的URL

```java
interface Demo {
  	//headerMap为header头信息
    @RequestLine("GET /test/{time}")
    Contributor contributors(@HeaderMap Map<String, Object> headerMap,@Param("time") String time);
}
class Contributor {
    public String msg;
    public String timestamp;
    public int code;
    public Map<String,Object> data;
}
public class MyApp {
    public static void main(String... args) throws JsonProcessingException {
        doCard();
    }
    public static void doCard(){
        long time = new Date().getTime()/1000;
        System.out.println(time);
        GitHub github = Feign.builder()
                .decoder(new JacksonDecoder())
                .target(Demo.class, "http://www.baid.com/");
        Map<String, Object> headerMap = new HashMap<>();
        headerMap.put("token","1");
        Contributor contributor = github.contributors(headerMap,time+"");
        System.out.println(contributor.data+"==="+contributor.code+"==="+contributor.msg + " (" + contributor.timestamp + ")");
    }
}
```

### 调用POST请求，表单传参的URL

```java
interface Demo {
		//可以通过对对象类型的参数加上QueryMap注解, 将对象的内容构造成查询参数
    @RequestLine("POST /iim-pocs/token")
    Issue createIssue(@HeaderMap Map<String, Object> headerMap,@QueryMap User user);
}
class Issue {
    public String errorMessage;
    public String result;
    public boolean success;
}

class User{
    public String userName;
    public String password;
    public User(String userName, String password) {
        this.userName = userName;
        this.password = password;
    }
}
    //表单格式post
    private static void formPost(){
        GitHub github = Feign.builder()
                .decoder(new JacksonDecoder())
                .target(GitHub.class, "http://111.111.190.10");

        // Fetch and print a list of the contributors to this library.
        Map<String, Object> headerMap = new HashMap<>();
        headerMap.put("Authorization","5f650c29-506e-43cd-b3c5-cb922260ef41");
        Issue contributor = github.createIssue(headerMap,new User("dyxdsfq","123456"));
        System.out.println(contributor.errorMessage+"==="+contributor.result+"==="+ " (" + contributor.success + ")");
    }
```

### 调用POST请求，json传参

```java
interface Demo {
    @RequestLine("POST /getRealTimeInfo")
    @Headers("Content-Type: application/json")
    @Body("{body}")
    RealInfo realTime(@Param("body") RealParam body);
}
class RealInfo{
    public Integer success;
    public Map<String,Object> data;
}
class RealParam{
    public String terminalId;
    public String params;
    public Integer flag;
    
    public RealParam(String terminalId, String params, Integer flag) {
        this.terminalId = terminalId;
        this.params = params;
        this.flag = flag;
    }

    @SneakyThrows
    @Override
    public String toString() {
        ObjectMapper objectMapper = new ObjectMapper();
        return objectMapper.writeValueAsString(this);
    }
}
    //json格式post
    private static void jsonPost(){
        GitHub github = Feign.builder()
                .decoder(new JacksonDecoder())
                .target(GitHub.class, "http://111.200.35.250:8090");
        RealInfo contributor = github.realTime(new RealParam("251712601170","A3,A0,C1,C2",1));
        System.out.println(contributor.data+"==="+contributor.success);
    }
```

构建一个远程代理接口的本地实例。使用Feign.builder() 构造器模式方法，带上一票配置方法的链式调用。主要的链式调用的配置方法介绍如下：

（1）target 配置方法

为构造器配置本地的代理接口，和远程的根目录。代理接口类的每一个接口方法前@RequestLine 声明的值，最终都会加上这个根目录。这个是最为重要的一个配置方法。代理接口类很重要，最终Feign.builder() 构造器返回的本地代理实例类型，就这个接口。

（2）options配置方法

options方法指定连接超时时长及响应超时时长。

（3）retryer配置方法

retryer方法主要是指定重试策略。

（4）decoder配置方法

decoder方法指定对象解码方式，这里用的是Jackson的解码方式，需要在pom.xml中添加Jackson的依赖。

主要的配置方法，就介绍这些，具体使用和其他的方法，请参见官网。

## 参考链接

[OpenFeign的GitHub仓库](https://github.com/OpenFeign/feign)

[feign 使用示例：@Body注解，http请求体](https://blog.csdn.net/qq_31772441/article/details/100176834)

[使用jackson进行字符串,集合和json之间的转换](https://blog.csdn.net/strophe/article/details/78781951)

[翻译: Spring Cloud Feign使用文档](https://segmentfault.com/a/1190000018313243)