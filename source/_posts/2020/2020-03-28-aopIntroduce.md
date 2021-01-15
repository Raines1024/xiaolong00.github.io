---
layout: post
title: AOP介绍
description: Java中面向切面介绍
category: blog
date: 2020-01-07 13:50:39
---

## 基础  

### AOP中 @Before @After @AfterThrowing @AfterReturning 的执行顺序
    
```
public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
   Object result;
   try {
       //@Before
       result = method.invoke(target, args);
       //@After
       return result;
   } catch (InvocationTargetException e) {
       Throwable targetException = e.getTargetException();
       //@AfterThrowing
       throw targetException;
   } finally {
       //@AfterReturning
   }
}
```

























































