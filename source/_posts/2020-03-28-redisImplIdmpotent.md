---
layout: post
title: Redis实现幂等性接口（Java语言实现）
description: Redis使用（二）
category: blog
---

## 基础准备

### 幂等性接口概念
任意多次执行所产生的影响均与一次执行的影响相同。按照这个含义，最终的含义就是：对数据库的影响只能是一次性的，不能重复处理。  

### 如何保证其幂等性，通常有以下手段：  
1. 数据库建立唯一性索引，可以保证最终插入数据库的只有一条数据    
2. token机制，每次接口请求前先获取一个token，然后再下次请求的时候在请求的header体中加上这个token，后台进行验证，如果验证通过删除token，下次请求再次判断token  
3. 悲观锁或者乐观锁，悲观锁可以保证每次for update的时候其他sql无法update数据(在数据库引擎是innodb的时候,select的条件必须是唯一索引,防止锁全表)  
4. 先查询后判断，首先通过查询数据库是否存在数据，如果存在证明已经请求过了，直接拒绝该请求，如果没有存在，就证明是第一次进来，直接放行。  

## Redis实现幂等性接口

### 搭建redis的服务Api
1. 首先是搭建redis服务器。  
2. 引入springboot中到的redis的stater，或者Spring封装的jedis也可以，后面主要用到的api就是它的set方法和exists方法,由于上篇Redis实现分布式锁时我们用的jedis，这里我们再尝试使用springboot的封装好的redisTemplate  

```
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```
3. redisTemplate所需工具类及具体使用参照之前 SpringBoot中使用Redis 博文

### 自定义注解AutoIdempotent(自定义注解使用方式可以参照之前写过的 Java自定义注解 博文)
自定义一个注解，定义此注解的主要目的是把它添加在需要实现幂等的方法上，凡是某个方法注解了它，都会实现自动幂等。后台利用反射如果扫描到这个注解，就会处理这个方法实现自动幂等，使用元注解ElementType.METHOD表示它只能放在方法上，etentionPolicy.RUNTIME表示它在运行时。   

```
@Target({ElementType.METHOD})
@Retention(RetentionPolicy.RUNTIME)
public @interface AutoIdempotent {
}
```

### token创建和检验
1. token服务接口   
我们新建一个接口，创建token服务，里面主要是两个方法，一个用来创建token，一个用来验证token。创建token主要产生的是一个字符串，检验token的话主要是传达request对象，为什么要传request对象呢？主要作用就是获取header里面的token,然后检验，通过抛出的Exception来获取具体的报错信息返回给前端。   

```
/**
 * 幂等接口所需功能
 */
public interface TokenService {

    /**
     * 创建token
     *
     * @return
     */
    String createToken();

    /**
     * 检验token
     *
     * @param request
     * @return
     */
    boolean checkToken(HttpServletRequest request) throws Exception;
}
```

2. token的服务实现类  
token引用了redis服务，创建token采用随机算法工具类生成随机uuid字符串,然后放入到redis中(为了防止数据的冗余保留,这里设置过期时间为10000秒,具体可视业务而定)，如果放入成功，最后返回这个token值。checkToken方法就是从header中获取token到值(如果header中拿不到，就从paramter中获取)，如若不存在,直接抛出异常。这个异常信息可以被拦截器捕捉到，然后返回给前端。  

```
/**
 * 幂等接口功能实现类
 */
@Service
public class TokenServiceImpl implements TokenService {


    @Autowired
    private DataCache dataCache;

    /**
     * 检验token
     *
     * @param request
     * @return
     */
    @Override
    public boolean checkToken(HttpServletRequest request) throws Exception {
        String token = request.getHeader(Constant.TOKEN_NAME);
        if (StrUtil.isBlank(token)) {
            // header中不存在tokenw
            token = request.getParameter(Constant.TOKEN_NAME);
            if (StrUtil.isBlank(token)) {
            // parameter中也不存在token
                throw new Exception("parameter中也不存在token");
            }
        }
        if (!dataCache.exists(token)) {
            throw new Exception("找不到token");
        }
        boolean remove = dataCache.remove(token);
        if (!remove) {
            throw new Exception("redis错误");
        }
        return true;
    }

    /**
     * 创建token
     *
     * @return
     */
    @Override
    public String createToken() {
        String str = UUID.randomUUID().toString();
        StringBuilder token = new StringBuilder();
        try {
            token.append(str);
            dataCache.set(token.toString(), token.toString(), 10000L);
            boolean notEmpty = StrUtil.isNotBlank(token.toString());
            if (notEmpty) {
                return token.toString();
            }

        } catch (Exception ex) {
            ex.printStackTrace();
        }
        return null;
    }

}
```

### 拦截器的配置
1. web配置类，实现WebMvcConfigurer，主要作用就是添加拦截器AutoIdempotentInterceptor到配置类中，这样我们到拦截器才能生效，注意使用@Configuration注解，这样在容器启动是时候就可以添加进入context中。   

```
@Configuration
public class WebConfiguration implements WebMvcConfigurer {

    @Resource
    private AutoIdempotentInterceptor autoIdempotentInterceptor;

    /**
     * 添加拦截器
     *
     * @param registry
     */
    @Override
    public void addInterceptors(InterceptorRegistry registry) {
        registry.addInterceptor(autoIdempotentInterceptor);
    }
}
```
2. 拦截处理器：主要的功能是拦截扫描到AutoIdempotent注解的方法,然后调用tokenService的checkToken()方法校验token是否正确，如果捕捉到异常就将异常信息渲染成json返回给前端   

```
/**
 * 幂等接口拦截器
 */
@Component
public class AutoIdempotentInterceptor implements HandlerInterceptor {

    @Autowired
    private TokenService tokenService;

    /**
     * 预处理
     *
     * @param request
     * @param response
     * @param handler
     * @return
     * @throws Exception
     */
    @Override
    public boolean preHandle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception {
        System.out.println("走了AutoIdempotentInterceptor.preHandle");
        if (!(handler instanceof HandlerMethod)) {
            return true;
        }
        HandlerMethod handlerMethod = (HandlerMethod) handler;
        Method method = handlerMethod.getMethod();
        //被ApiIdempotment标记的扫描
        AutoIdempotent methodAnnotation = method.getAnnotation(AutoIdempotent.class);
        if (methodAnnotation != null) {
            try {
                return tokenService.checkToken(request);
                // 幂等性校验, 校验通过则放行, 校验失败则抛出异常, 并通过统一异常处理返回友好提示
            } catch (Exception ex) {
                writeReturnJson(response, "返回的json错误信息");
                throw ex;
            }
        }
        //必须返回true,否则会被拦截一切请求
        return true;
    }

    /**
     * 返回的json值
     *
     * @param response
     * @param json
     * @throws Exception
     */
    private void writeReturnJson(HttpServletResponse response, String json) throws Exception {
        PrintWriter writer = null;
        response.setCharacterEncoding("UTF-8");
        response.setContentType("text/html; charset=utf-8");
        try {
            writer = response.getWriter();
            writer.print(json);
        } catch (IOException e) {
        } finally {
            if (writer != null)
                writer.close();
        }
    }

    @Override
    public void postHandle(HttpServletRequest request, HttpServletResponse response, Object handler, ModelAndView modelAndView) throws Exception {
    }


    @Override
    public void afterCompletion(HttpServletRequest request, HttpServletResponse response, Object handler, Exception ex) throws Exception {
    }


}
```

### 测试用例
1、模拟业务请求类  
首先我们需要通过/get/token路径通过getToken()方法去获取具体的token，然后我们调用test/Idempotence方法，这个方法上面注解了@AutoIdempotent，拦截器会拦截所有的请求，当判断到处理的方法上面有该注解的时候，就会调用TokenService中的checkToken()方法，如果捕获到异常会将异常抛出调用者。  

```
/**
 * 幂等接口测试Controller
 */
@RestController
public class BusinessController {


    @Resource
    private TokenService tokenService;


    /**
     * 幂等接口，必须携带token
     * @return
     */
    @AutoIdempotent
    @PostMapping("/test/Idempotence")
    public String testIdempotence() {
        String businessResult = "幂等接口";
        if (StrUtil.isNotBlank(businessResult)) {
            return businessResult;
        }
        return StrUtil.EMPTY;

    }

    /**
     * 获取幂等接口所需token
     * @return
     */
    @PostMapping("/get/token")
    public String getToken() {
        String token = tokenService.createToken();
        if (StrUtil.isNotBlank(token)) {
            return token;
        }
        return StrUtil.EMPTY;
    }

}

```

## 总结
介绍了使用springboot和拦截器、redis来优雅的实现接口幂等，对于幂等在实际的开发过程中是十分重要的，因为一个接口可能会被无数的客户端调用，如何保证其不影响后台的业务处理，如何保证其只影响数据一次是非常重要的，它可以防止产生脏数据或者乱数据，也可以减少并发量，实乃十分有益的一件事。而传统的做法是每次判断数据，这种做法不够智能化和自动化，比较麻烦。而今天的这种自动化处理也可以提升程序的伸缩性。  




































