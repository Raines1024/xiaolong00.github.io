---
title: Caffeine缓存使用
date: 2021-05-08 14:17:39
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java

---



## 本地缓存介绍

缓存在日常开发中启动至关重要的作用，由于是存储在内存中，数据的读取速度是非常快的，能大量减少对数据库的访问，减少数据库的压力。
Redis这种NoSql作为缓存组件，它能够很好的作为分布式缓存组件提供多个服务间的缓存，但是Redis这种还是需要网络开销，增加时耗。本地缓存是直接从本地内存中读取，没有网络开销，例如秒杀系统或者数据量小的缓存等，比远程缓存更合适。
按 Caffeine Github 文档描述，Caffeine 是基于 JAVA 8 的高性能缓存库。并且在 spring5 (springboot 2.x) 后，spring 官方放弃了 Guava，而使用了性能更优秀的 Caffeine 作为默认缓存组件。

## Caffeine缓存使用场景
Caffeine一般当作本地一级缓存使用，使用中先从本地缓存中查找数据；若没有，再从集中式缓存中查询数据；若还没有，从DB中读取数据。


## 缓存的选择
- 一级缓存：Caffeine是一个一个高性能的 Java 缓存库；使用 Window TinyLfu 回收策略，提供了一个近乎最佳的命中率（Caffeine 缓存详解）。优点数据就在应用内存所以速度快。缺点受应用内存的限制，所以容量有限；没有持久化，重启服务后缓存数据会丢失；在分布式环境下缓存数据数据无法同步；
- 二级缓存：redis是一高性能、高可用的key-value数据库，支持多种数据类型，支持集群，和应用服务器分开部署易于横向扩展。优点支持多种数据类型，扩容方便；有持久化，重启应用服务器缓存数据不会丢失；他是一个集中式缓存，不存在在应用服务器之间同步数据的问题。缺点每次都需要访问redis存在IO浪费的情况。
我们可以发现Caffeine和Redis的优缺点正好相反，所以他们可以有效的互补。

## 策略介绍
大体介绍，具体请看参考链接 Caffeine缓存
填充策略（Population）
Caffeine 为我们提供了三种填充策略：手动加载、同步加载和异步加载

驱逐策略（eviction）
Caffeine提供三类驱逐策略：基于大小（size-based），基于时间（time-based）和基于引用（reference-based）。

## 使用
1. Maven 引入相关依赖

   ```xml
           <!--缓存-->
           <dependency>
               <groupId>com.github.ben-manes.caffeine</groupId>
               <artifactId>caffeine</artifactId>
           </dependency>
   ```

2. 配置缓存配置类

   ```java
   import com.github.benmanes.caffeine.cache.Cache;
   import com.github.benmanes.caffeine.cache.Caffeine;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.Configuration;
   import java.util.concurrent.TimeUnit;
   
   @Configuration
   public class CacheConfig {
   
       @Bean
       public Cache<String, Object> caffeineCache() {
           return Caffeine.newBuilder()
                   // 设置最后一次写入或访问后经过固定时间过期
                   .expireAfterWrite(60, TimeUnit.SECONDS)
                   // 初始的缓存空间大小
                   .initialCapacity(100)
                   // 缓存的最大条数
                   .maximumSize(1000)
                   .build();
       }
   
   }
   ```

   

3. 定义测试的实体对象

   ```java
   import lombok.Data;
   import lombok.ToString;
   
   @Data
   @ToString
   public class UserInfo {
       private Integer id;
       private String name;
       private String sex;
       private Integer age;
   }
   ```

4. 定义服务接口类和实现类

   ```java
   import com.raines.javaadvanced.firstCache.vo.UserInfo;
   
   public interface UserInfoService {
   
       /**
        * 增加用户信息
        *
        * @param userInfo 用户信息
        */
       void addUserInfo(UserInfo userInfo);
   
       /**
        * 获取用户信息
        *
        * @param id 用户ID
        * @return 用户信息
        */
       UserInfo getByName(Integer id);
   
       /**
        * 修改用户信息
        *
        * @param userInfo 用户信息
        * @return 用户信息
        */
       UserInfo updateUserInfo(UserInfo userInfo);
   
       /**
        * 删除用户信息
        *
        * @param id 用户ID
        */
       void deleteById(Integer id);
   
   }
   ```

   实现类

   ```java
   import com.github.benmanes.caffeine.cache.Cache;
   import com.raines.javaadvanced.firstCache.service.UserInfoService;
   import com.raines.javaadvanced.firstCache.vo.UserInfo;
   import lombok.extern.slf4j.Slf4j;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.stereotype.Service;
   import org.springframework.util.StringUtils;
   import java.util.HashMap;
   import java.util.Map;
   
   @Slf4j
   @Service
   public class UserInfoServiceImpl implements UserInfoService {
   
       /**
        * 模拟数据库存储数据
        */
       private Map<Integer, UserInfo> userInfoMap = new HashMap<>();
   
       @Autowired
       Cache<String, Object> caffeineCache;
   
       @Override
       public void addUserInfo(UserInfo userInfo) {
           log.info("create");
           userInfoMap.put(userInfo.getId(), userInfo);
           // 加入缓存
           caffeineCache.put(String.valueOf(userInfo.getId()),userInfo);
       }
   
       @Override
       public UserInfo getByName(Integer id) {
           // 先从缓存读取
           UserInfo userInfo = null;
           //根据key查询一个缓存，如果没有返回NULL
           Object obj = caffeineCache.getIfPresent(String.valueOf(id));
           if (obj != null){
               return (UserInfo)obj;
           }
           //可以使用Cache.asMap() 方法获取ConcurrentMap进而对缓存进行一些更改
   //        UserInfo userInfo = (UserInfo) caffeineCache.asMap().get(String.valueOf(id));
   //        if (userInfo != null){
   //            return userInfo;
   //        }
           // 如果缓存中不存在，则从库中查找
           log.info("去模拟库中查询");
           userInfo = userInfoMap.get(id);
           // 如果用户信息不为空，则加入缓存
           if (userInfo != null){
               caffeineCache.put(String.valueOf(userInfo.getId()),userInfo);
           }
           return userInfo;
       }
   
       @Override
       public UserInfo updateUserInfo(UserInfo userInfo) {
           log.info("update");
           if (!userInfoMap.containsKey(userInfo.getId())) {
               return null;
           }
           // 取旧的值
           UserInfo oldUserInfo = userInfoMap.get(userInfo.getId());
           // 替换内容
           if (!StringUtils.isEmpty(oldUserInfo.getAge())) {
               oldUserInfo.setAge(userInfo.getAge());
           }
           if (!StringUtils.isEmpty(oldUserInfo.getName())) {
               oldUserInfo.setName(userInfo.getName());
           }
           if (!StringUtils.isEmpty(oldUserInfo.getSex())) {
               oldUserInfo.setSex(userInfo.getSex());
           }
           // 将新的对象存储，更新旧对象信息
           userInfoMap.put(oldUserInfo.getId(), oldUserInfo);
           // 替换缓存中的值
           caffeineCache.put(String.valueOf(oldUserInfo.getId()),oldUserInfo);
           return oldUserInfo;
       }
   
       @Override
       public void deleteById(Integer id) {
           log.info("delete");
           userInfoMap.remove(id);
           // 从缓存中删除
           caffeineCache.asMap().remove(String.valueOf(id));
   //        caffeineCache.invalidate(String.valueOf(id));
       }
   
   }
   ```

5. 测试的 Controller 类

   ```java
   import com.raines.javaadvanced.firstCache.service.UserInfoService;
   import com.raines.javaadvanced.firstCache.vo.UserInfo;
   import org.springframework.beans.factory.annotation.Autowired;
   import org.springframework.web.bind.annotation.*;
   
   @RestController
   @RequestMapping
   public class UserInfoController {
   
       @Autowired
       private UserInfoService userInfoService;
   
       @GetMapping("/userInfo/{id}")
       public Object getUserInfo(@PathVariable Integer id) {
           UserInfo userInfo = userInfoService.getByName(id);
           if (userInfo == null) {
               return "没有该用户";
           }
           return userInfo;
       }
   
       @PostMapping("/userInfo")
       public Object createUserInfo(@RequestBody UserInfo userInfo) {
           userInfoService.addUserInfo(userInfo);
           return "SUCCESS";
       }
   
       @PutMapping("/userInfo")
       public Object updateUserInfo(@RequestBody UserInfo userInfo) {
           UserInfo newUserInfo = userInfoService.updateUserInfo(userInfo);
           if (newUserInfo == null){
               return "不存在该用户";
           }
           return newUserInfo;
       }
   
       @DeleteMapping("/userInfo/{id}")
       public Object deleteUserInfo(@PathVariable Integer id) {
           userInfoService.deleteById(id);
           return "SUCCESS";
       }
   
   }
   ```

   

## 参考链接
[Caffeine缓存](https://www.jianshu.com/p/9a80c662dac4)

[SpringBoot 使用 Caffeine 本地缓存](https://zhuanlan.zhihu.com/p/109226599)






















