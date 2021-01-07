---
layout: post
title: Redis实现分布式锁（Java语言实现）
description: Redis使用（一）
category: blog
---

## 基础准备

### Redis简介
Redis是一个Key-Value存储系统。   

### Redis命令参考  
http://redisdoc.com  

## 使用set函数的NX参数实现分布式锁
对于java中想操作redis，好的方式是使用jedis，首先pom中引入依赖（使用Jedis 2.9.3版本）：   

```
    <dependency>
        <groupId>redis.clients</groupId>
        <artifactId>jedis</artifactId>
    </dependency>
```

### 初始化Jedis

```
JedisPool jedisPool;
JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
String ip = "127.0.0.1";
int port = 6379;
int timeout = 2000;

public RedisLockUtils() {
    // 初始化jedis
    // 设置配置
    jedisPoolConfig.setMaxTotal(1024);
    jedisPoolConfig.setMaxIdle(100);
    jedisPoolConfig.setMaxWaitMillis(100);
    jedisPoolConfig.setTestOnBorrow(false);//jedis 第一次启动时，会报错
    jedisPoolConfig.setTestOnReturn(true);
    // 初始化JedisPool
    jedisPool = new JedisPool(jedisPoolConfig, ip, port, timeout);
}
```

### 如何使用nx生成锁  
#### 创建锁的策略：  
redis的普通key一般都允许覆盖，A用户set某个key后，B在set相同的key时同样能成功，如果是锁场景，那就无法知道到底是哪个用户set成功的；这里jedis的setnx方式为我们解决了这个问题，简单原理是：当A用户先set成功了，那B用户set的时候就返回失败，满足了某个时间点只允许一个用户拿到锁。  
#### 锁过期时间：  
某个抢购场景时候，如果没有过期的概念，当A用户生成了锁，但是后面的流程被阻塞了一直无法释放锁，那其他用户此时获取锁就会一直失败，无法完成抢购的活动；当然正常情况一般都不会阻塞，A用户流程会正常释放锁；过期时间只是为了更有保障。   

#### 注意点在于jedis的set方法，其参数的说明如下：  
NX：只在键不存在时，才对键进行设置操作。执行 SET key value NX 的效果等同于执行 SETNX key value 。  
PX milliseconds：将键的过期时间设置为 milliseconds 毫秒。执行 SET key value PX milliseconds 的效果等同于执行 PSETEX key milliseconds value。  
==下面是其他参数介绍==  
XX：只在键已经存在时，才对键进行设置操作。  
EX seconds：将键的过期时间设置为 seconds 秒。执行 SET key value EX seconds 的效果等同于执行 SETEX key seconds value 。  

```
public boolean setnx(String key, String val) {
    Jedis jedis = null;
    try {
        jedis = jedisPool.getResource();
        if (jedis == null) {
            return false;
        }
        return jedis.set(key, val, "NX", "PX", 10000 * 60).
                equalsIgnoreCase("ok");
    } catch (NullPointerException ex) {
    } catch (Exception ex) {
        ex.printStackTrace();
    } finally {
        if (jedis != null) {
            jedis.close();
        }
    }
    return false;
}
```

### 如何删除锁
上面是创建锁，同样的具有有效时间，但是我们不能完全依赖这个有效时间，场景如：有效时间设置1分钟，本身用户A获取锁后，没遇到什么特殊情况正常生成了抢购订单后，此时其他用户应该能正常下单了才对，但是由于有个1分钟后锁才能自动释放，那其他用户在这1分钟无法正常下单（因为锁还是A用户的），因此我们需要A用户操作完后，主动去解锁：   
这里也使用了jedis方式，直接执行lua脚本：根据val判断其是否存在，如果存在就del；  
其实个人认为通过jedis的get方式获取val后，然后再比较value是否是当前持有锁的用户，如果是那最后再删除，效果其实相当；只不过直接通过eval执行脚本，这样避免多一次操作了redis而已，缩短了原子操作的间隔。同样这里创建个get方式的api来测试：  
注意的是delnx时，需要传递创建锁时的value，因为通过et的value与delnx的value来判断是否是持有锁的操作请求，只有value一样才允许del.  
通过key和value删除指定键  

```
    public int delnx(String key, String val) {
        Jedis jedis = null;
        try {
            jedis = jedisPool.getResource();
            if (jedis == null) {
                return 0;
            }

            //if redis.call('get','orderkey')=='1111' then return redis.call('del','orderkey') else return 0 end
            StringBuilder sbScript = new StringBuilder();
            sbScript.append("if redis.call('get','").append(key).append("')").append("=='").append(val).append("'").
                    append(" then ").
                    append(" return redis.call('del','").append(key).append("')").
                    append(" else ").
                    append(" return 0").
                    append(" end");

            return Integer.valueOf(jedis.eval(sbScript.toString()).toString());
        } catch (Exception ex) {
        } finally {
            if (jedis != null) {
                jedis.close();
            }
        }
        return 0;
    }
```

### 模拟抢单动作(10w个人开抢)
有了上面对分布式锁的粗略基础，我们模拟下10w人抢单的场景，其实就是一个并发操作请求而已，由于环境有限，只能如此测试；如下初始化10w个用户，并初始化库存，商品等信息.  
有了上面10w个不同用户，我们设定商品只有10个库存，然后通过并行流的方式来模拟抢购  
这里实现的逻辑是：  
parallelStream()：并行流模拟多用户抢购  
(startTime + timeout) >= System.currentTimeMillis()：判断未抢成功的用户，timeout秒内继续获取锁  
获取锁前和后都判断库存是否还足够  
jedisCom.setnx(shangpingKey, b)：用户获取抢购锁  
获取锁后并下单成功，最后释放锁：jedisCom.delnx(shangpingKey, b)  

```
//总库存
private long nKuCuen;
//商品key名字
private String shangpingKey = "computer_key";
//获取锁的超时时间 秒
private int timeout = 30 * 1000;
static RedisLockUtils jedisCom = new RedisLockUtils();

@GetMapping("/qiangdan")
public List<String> qiangdan() {
    //抢到商品的用户
    List<String> shopUsers = new ArrayList<>();

    //构造很多用户
    List<String> users = new ArrayList<>();
    IntStream.range(0, 100000)/*.parallel()*/.forEach(b -> {
//        IntStream.range(0, 100000).forEach(b -> {
        users.add("神牛-" + b);
    });
    //初始化库存
    nKuCuen = 10;

    //模拟开抢(parallelStream()：并行流模拟多用户抢购)
    users.parallelStream().forEach(b -> {
        String shopUser = qiang(b);
        if (!StringUtils.isEmpty(shopUser)) {
            shopUsers.add(shopUser);
        }
    });

    return shopUsers;
}

/**
 * 模拟抢单动作
 *
 * @param b
 * @return
 */
private String qiang(String b) {
    //用户开抢时间
    long startTime = System.currentTimeMillis();

    //未抢到的情况下，30秒内继续获取锁
    while ((startTime + timeout) >= System.currentTimeMillis()) {
        //获取锁前和后都判断库存是否还足够
        //商品是否剩余
        if (nKuCuen <= 0) {
            break;
        }
        if (jedisCom.setnx(shangpingKey, b)) {
            //用户b拿到锁
            logger.info("用户{}拿到锁...", b);
            try {
                //商品是否剩余
                if (nKuCuen <= 0) {
                    break;
                }

                //模拟生成订单耗时操作，方便查看：神牛-50 多次获取锁记录
                try {
                    TimeUnit.SECONDS.sleep(1);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }

                //抢购成功，商品递减，记录用户
                nKuCuen -= 1;

                //抢单成功跳出
                logger.info("用户{}抢单成功跳出...所剩库存：{}", b, nKuCuen);

                return b + "抢单成功，所剩库存：" + nKuCuen;
            } finally {
                logger.info("用户{}释放锁...", b);
                //释放锁
                jedisCom.delnx(shangpingKey, b);
            }
        } else {
            //用户b没拿到锁，在超时范围内继续请求锁，不需要处理
            // logger.info("用户{}等待获取锁...", b);
        }
    }
    return "";
}
```






































