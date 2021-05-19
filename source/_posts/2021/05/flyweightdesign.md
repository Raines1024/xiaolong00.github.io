---
title: 享元模式介绍
date: 2021-05-19 20:17:39
description: 设计模式系列
tags: [编程, 过去]
category:
    - 100 学习类
    - 110 编程
    - 111 Java


---



## 享元模式概念

享元模式主要通过对象的复用来减少对象创建的次数和数量，以减少系统内存的使用和降低系统的负载。享元模式属于结构性模式，在系统需要一个对象时享元模式首先在系统中查找并尝试重用现有的对象，如果未找到匹配的对象，则创建新对象并将其缓存在系统中以便下次使用。
享元模式主要用于避免在有大量对象时频繁创建和销毁对象造成系统资源的浪费，把其中共同的部分抽象出来，如果有相同的业务请求，则直接返回内存中已有的对象，避免重新创建。
以内存的申请和使用为例介绍享元模式的使用方法，创建一个MemoryFactory作为内存管理的工厂，用户通过工厂获取内存，在系统内存池有可用内存时直接获取该内存，如果没有则创建一个内存对象放入内存池，等下次有相同的内存请求过来时直接将该内存分配给用户即可。



## 代码实现

### 定义实体类Memory

```java
import lombok.Data;

@Data
public class Memory {

    //内存大小，单位为MB
    private int size;
    //内存是否在被使用
    private  boolean isused;
    //内存id
    private String id;

    public Memory(int size, boolean isused, String id) {
        this.size = size;
        this.isused = isused;
        this.id = id;
    }

}
```

### 定义MemoryFactory工厂

```java
import com.alibaba.fastjson.JSON;
import lombok.extern.slf4j.Slf4j;
import org.apache.tomcat.util.json.JSONParser;

import java.util.ArrayList;
import java.util.List;
import java.util.UUID;

@Slf4j
public class MemoryFactory {

    //内存对象列表
    private static List<Memory> memoryList = new ArrayList<>();

    public static Memory getMemory(int size){
        Memory memory = null;
        for (int i = 0;i<memoryList.size();i++){
            memory = memoryList.get(i);
            //如果存在和需求size一样大小并且未使用的内存块，则直接返回，并将该内存的使用状态设置为已使用。
            if (memory.getSize() == size && !memory.isIsused()) {
                memory.setIsused(true);
                memoryList.set(i,memory);
                log.info("从集合中获取内存对象"+ JSON.toJSONString(memory));
                break;
            }
        }
        //如果内存不存在，则从系统中申请新的内存返回，并将该内存加入到内存对象列表中
        if (memory == null){
            memory = new Memory(32,false, UUID.randomUUID().toString());
            log.info("添加一个内存对象到集合");
            memoryList.add(memory);
        }
        return memory;
    }

    //释放内存，将内存的使用状态设置为false
    public static void releaseMemory(String id){
        for (int i = 0; i < memoryList.size(); i++) {
            Memory memory = memoryList.get(i);
            //如果存在和需求size一样大小并且未使用的内存块，则直接返回
            if (memory.getId().equals(id)){
                memory.setIsused(false);
                memoryList.set(i,memory);
                log.info("释放内存对象"+id);
                break;
            }
        }
    }

}
```

### 使用享元模式

```java
public class Main {

    public static void main(String[] args) {
        //首次获取内存，将创建一个内存
        Memory memory = MemoryFactory.getMemory(32);
        //在使用后释放内存
        MemoryFactory.releaseMemory(memory.getId());
        //重新获取内存
        MemoryFactory.getMemory(32);
    }

}
```













