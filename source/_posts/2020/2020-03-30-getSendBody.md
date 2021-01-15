---
layout: post
title: 谁说 HTTP GET 就不能通过 Body 来发送数据呢？
description: 去大胆尝试
category: blog
date: 2020-01-07 13:50:39
---

## 转载地址  
https://yanbin.blog/why-http-get-cannot-sent-data-with-reuqest-body/  
当我们被问及 HTTP 的 GET 与 POST 两种请求方式的区别的时候，很多答案是说 GET 的数据须通过 URL 以 Query Parameter 来传送，而 POST 可以通过请求体来发送数据，所以因 URL 的受限，往往 GET 无法发送太多的字符。这个回答好比在启用了 HTTPS 时，GET 请求 URL 中的参数仍然是明文传输的一样。  
GET 果真不能通过 Request Body 来传送数据吗？非也。如此想法多半是因循着网页中 form 的 method 属性只有 get 与 post 两种而来。因为把 form 的 method 设置为 post, 表单数据会放在 body 中，而 method 为 get(默认值) 时, 提交时浏览器会把表单中的字符拼接到 action 的 URL 后作为 query parameter 传送。于是乎就有了这么一种假像：HTTP GET 必须通过 URL 的查询参数来发送数据。  
其实 HTTP 规范并未规定说 GET 就不能发送 body 数据，在 RFC GET 中只是说  

> The GET method means retrieve whatever information (in the form of an entity) is identified by the Request-URI.

只是说 GET 意味着通过 URI 来识别资源。

我也是本着传统上对 GET 与 POST 区别的误解很多年，今天突然意识到 GET 应该可以使用 body, 况且 HTTP 本身是一个纯文本的协议。没有测试就没有 100% 的发言权，所以做了如下的测试。

在 rfc7231 中对 GET 的说明明确了  
> A payload within a GET request message has no defined semantics;
sending a payload body on a GET request might cause some existing
implementations to reject the request.  

## 四种常见的 POST 提交数据方式
转载自： https://imququ.com/post/four-ways-to-post-data-in-http.html  

HTTP/1.1 协议规定的 HTTP 请求方法有 OPTIONS、GET、HEAD、POST、PUT、DELETE、TRACE、CONNECT 这几种。其中 POST 一般用来向服务端提交数据，本文主要讨论 POST 提交数据的几种方式。   
我们知道，HTTP 协议是以 ASCII 码传输，建立在 TCP/IP 协议之上的应用层规范。规范把 HTTP 请求分为三个部分：请求行、请求头、消息主体。类似于下面这样：  

```
<method> <request-URL> <version>
<headers>

<entity-body>
```
协议规定 POST 提交的数据必须放在消息主体（entity-body）中，但协议并没有规定数据必须使用什么编码方式。实际上，开发者完全可以自己决定消息主体的格式，只要最后发送的 HTTP 请求满足上面的格式就可以。  
但是，数据发送出去，还要服务端解析成功才有意义。一般服务端语言如 php、python 等，以及它们的 framework，都内置了自动解析常见数据格式的功能。服务端通常是根据请求头（headers）中的 Content-Type 字段来获知请求中的消息主体是用何种方式编码，再对主体进行解析。所以说到 POST 提交数据方案，包含了 Content-Type 和消息主体编码方式两部分。下面就正式开始介绍它们。  
- application/x-www-form-urlencoded  
这应该是最常见的 POST 提交数据的方式了。浏览器的原生 <form> 表单，如果不设置 enctype 属性，那么最终就会以 application/x-www-form-urlencoded 方式提交数据。请求类似于下面这样（无关的请求头在本文中都省略掉了）：  

```
POST http://www.example.com HTTP/1.1
Content-Type: application/x-www-form-urlencoded;charset=utf-8

title=test&sub%5B%5D=1&sub%5B%5D=2&sub%5B%5D=3
```
首先，Content-Type 被指定为 application/x-www-form-urlencoded；其次，提交的数据按照 key1=val1&key2=val2 的方式进行编码，key 和 val 都进行了 URL 转码。大部分服务端语言都对这种方式有很好的支持。   
- multipart/form-data  
这又是一个常见的 POST 数据提交的方式。我们使用表单上传文件时，必须让 <form> 表单的 enctype 等于 multipart/form-data。   
- application/json  
application/json 这个 Content-Type 作为响应头大家肯定不陌生。实际上，现在越来越多的人把它作为请求头，用来告诉服务端消息主体是序列化后的 JSON 字符串。由于 JSON 规范的流行，除了低版本 IE 之外的各大浏览器都原生支持 JSON.stringify，服务端语言也都有处理 JSON 的函数，使用 JSON 不会遇上什么麻烦。  
JSON 格式支持比键值对复杂得多的结构化数据，这一点也很有用。  
这种方案，可以方便的提交复杂的结构化数据，特别适合 RESTful 的接口。各大抓包工具如 Chrome 自带的开发者工具、Firebug、Fiddler，都会以树形结构展示 JSON 数据，非常友好。  
- text/xml  
它是一种使用 HTTP 作为传输协议，XML 作为编码方式的远程调用规范。典型的 XML-RPC 请求是这样的：

```
POST http://www.example.com HTTP/1.1 
Content-Type: text/xml

<?xml version="1.0"?>
<methodCall>
    <methodName>examples.getStateName</methodName>
    <params>
        <param>
            <value><i4>41</i4></value>
        </param>
    </params>
</methodCall>
```

## 实战

### 在一个 Spring Boot Web 项目中创建的 GET 请求 API

```
@RestController
public class DemoController {
    @RequestMapping(value = "/getSendBody", method = RequestMethod.GET)
    public String getRequest(@RequestParam("id") String id,@RequestBody Person body) {
        return id+"|"+body.toString();
    }
}
```
Person实体类

```
@Data
@ToString
public class Person implements Serializable {
    private Integer age;
    private String name;
}
```

### Postman测试
路径：http://localhost:8080/getSendBody?id=123   
选择Postman中body，选择raw，选择JSON，输入如下：

> {"name":"Raines"}

返回

> 123|Person(age=null, name=Raines)

### curl测试

命令行输入命令：  
> curl -v -X GET -H 'Content-Type: application/json' http://127.0.0.1:8080/getSendBody\?id\=123 -d '{"name":"sd"}'

返回   
```
*   Trying 127.0.0.1...
* TCP_NODELAY set
* Connected to 127.0.0.1 (127.0.0.1) port 8080 (#0)
> GET /getSendBody?id=123 HTTP/1.1
> Host: 127.0.0.1:8080
> User-Agent: curl/7.64.1
> Accept: */*
> Content-Type: application/json
> Content-Length: 13
>
* upload completely sent off: 13 out of 13 bytes
< HTTP/1.1 200
< Content-Type: text/plain;charset=UTF-8
< Content-Length: 59
< Date: Mon, 30 Mar 2020 02:48:51 GMT
<
* Connection #0 to host 127.0.0.1 left intact
123|Person(age=null, name=sd)* Closing connection 0
```

#### curl参数解释
- -v    可以看到详细的请求响应数据，请求头的 Content-Length 都是 13, 即 {"name":"sd"} 的长度，它们确实是在 Request Body 中，服务端接送 GET 来的 body 数据也没有半点问题。  
- -X GET    指定 request 请求方式  
- -H    'key:value'指定请求头 header 键值对
- -d/--data <data>  HTTP POST方式传送数据

#### 注意
curl 命令, 需要用 -X 指定为 GET 请求，否则 curl 在使用 -d 发送 body 数据时自动切换为 POST 请求

## HTTP 规范链接
https://tools.ietf.org/html/rfc7235

## 总结
虽然 GET 请求可以通过 body 发送数据，但并不推荐通过body传参。









































