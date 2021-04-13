---
title: Spring异常统一处理方案
date: 2021-04-13 19:01:39
description: 自定义异常处理
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java
---

## 背景

前后端分离项目中抛出异常通常由后端统一异常处理捕获异常，然后再统一返回前端具体的错误信息，具体异常可以输出到日志文件或者写入es等备用，而不是把异常抛出到调用方。

实际项目开发过程中，因为业务流程的复杂性，我们经常需要手动抛出异常从而使Spring进行事物回滚操作，而此时，自定义异常是必不可少的，就restful风格的接口我们做一下SpringBoot项目的全局异常捕获及处理。

## 注解解释

### @ExceptionHandler注解

使用该注解有一个不好的地方就是：进行异常处理的方法必须与出错的方法在同一个Controller里面。这种方式最大的缺陷就是不能全局控制异常。每个类都要写一遍。

### 使用 @ControllerAdvice+@ExceptionHandler 注解

@ExceptionHandler 需要进行异常处理的方法必须与出错的方法在同一个Controller里面。那么当代码加入了@ControllerAdvice，则不需要必须在同一个 controller 中了。这也是 Spring 3.2 带来的新特性。从名字上可以看出大体意思是控制器增强。 也就是说，@Controlleradvice + @ ExceptionHandler 也可以实现全局的异常捕捉。

**请确保此WebExceptionHandle 类能被扫描到并装载进 Spring 容器中。**

### @RestControllerAdvice与@ControllerAdvice的区别

@RestControllerAdvice与@ControllerAdvice的区别就和@RestController与@Controller的区别类似，@RestControllerAdvice注解包含了@ControllerAdvice注解和@ResponseBody注解。

当自定义类加@ControllerAdvice注解时，方法需要返回json数据时，每个方法还需要添加@ResponseBody注解：
当自定义类加@RestControllerAdvice注解时，方法自动返回json数据，每个方法无需再添加@ResponseBody注解：

## 实战：捕获自定义业务异常并以固定格式返回前端

1. 自定义异常

   ```java
   public class LogicException extends RuntimeException{
   
       protected Integer code;
   
       public LogicException() {
           super();
       }
   
       public LogicException(String message, Integer code) {
           super(message);
           this.code = code;
       }
   
       public LogicException(String message) {
           super(message);
           this.code = 500;
       }
   
   }
   ```

2. 统一异常处理

   ```java
   @RestControllerAdvice
   @Slf4j
   public class ExceptionController {
   
       /**
        * 业务逻辑处理异常
        */
       @ExceptionHandler(LogicException.class)
       public Map<String, Object> handleLogicException(HttpServletRequest request, LogicException exception) {
           log.error("业务逻辑处理异常",exception);
           String message = exception.getMessage();
           List<String> errors = new ArrayList<>();
           errors.add(message);
           Map<String, Object> data = new HashMap<>();
           data.put("errors", errors);
           return reason(exception.code,message,data);
       }
   
       // 捕捉其他所有异常
       @ExceptionHandler(Exception.class)
       public Object globalException(HttpServletRequest request, Exception e) {
           log.error("未知异常",e);
           return reason(500,"未知异常",e.getMessage());
       }
     
       public static Map<String, Object> reason(Integer code, String msg,Object data) {
           Map<String, Object> resultMap = new HashMap<>();
           resultMap.put("code", code);
           resultMap.put("msg", msg);
           resultMap.put("data", data);
           resultMap.put("timestamp", new Date().getTime());
           return resultMap;
       }
   
   }
   ```

3. http接口实现

   ```
   @RestController
   @RequestMapping("/demo")
   public class DemoController {
   
       @GetMapping("/login")
       public boolean login() {
           throw new LogicException("error!!!");
       }
   
   }
   ```

4. 接口测试

   访问`http://localhost:port/demo/login`测试可发现响应数据如下

   ```
   {
     "msg": "error!!!",
     "code": 500,
     "data": {
       "errors": [
         "error!!!"
       ]
     },
     "timestamp": 1618292724522
   }
   ```

   

## 拓展

实现 HandlerExceptionResolver 接口也可以进行全局的异常控制。例如：

```java
@Component  
public class ExceptionTest implements HandlerExceptionResolver{  
 
    /**  
     * TODO 简单描述该方法的实现功能（可选）.  
     * @see org.springframework.web.servlet.HandlerExceptionResolver#resolveException(javax.servlet.http.HttpServletRequest, javax.servlet.http.HttpServletResponse, java.lang.Object, java.lang.Exception)  
     */   
    public ModelAndView resolveException(HttpServletRequest request, HttpServletResponse response, Object handler,  
            Exception ex) {  
        System.out.println("This is exception handler method!");  
        return null;  
    }  
}
```

















