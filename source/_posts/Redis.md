---
title: Redis
date: 2020-02-26 17:58:27
categories: [Redis]
tags: 
visitors:

---

### Redis相关问题

#### 缓存与数据一致性问题

对于既有数据库操作又有缓存操作的接口，一般分为两种执行顺序。

1. 先操作数据库，再操作缓存。这种情况下如果数据库操作成功，缓存操作失败就会导致缓存和数据库不一致。
2. 第二种情况就是先操作缓存再操作数据库，这种情况下如果缓存操作成功，数据库操作失败也会导致数据库和缓存不一致。

大部分情况下，我们的缓存理论上都是需要可以从数据库恢复出来的，所以基本上采取第一种顺序都是不会有问题的。针对那些必须保证数据库和缓存一致的情况，通常是不建议使用缓存的，如果必须使用的话，可以参考[这篇文章](https://www.jianshu.com/p/a532962cb9e9)来解决这个缓解缓存与数据库的一致性问题。

#### 缓存击穿问题

缓存击穿表示恶意用户频繁的模拟请求缓存中不存在的数据，以致这些请求短时间内直接落在了数据库上，导致数据库性能急剧下降，最终影响服务整体的性能。这个在实际项目很容易遇到，如抢购活动、秒杀活动的接口 API 被大量的恶意用户刷，导致短时间内数据库宕机。

解决方案：

1. 使用互斥锁排队。当从缓存中获取数据失败时，给当前接口加上锁，从数据库中加载完数据并写入后再释放锁。若其它线程获取锁失败，则等待一段时间后重试。
2. 使用布隆过滤器。将所有可能存在的数据缓存放到布隆过滤器中，当黑客访问不存在的缓存时迅速返回避免缓存及 DB 挂掉。

#### 缓存雪崩问题

问题：短时间内有大量缓存失效，如果这期间有大量的请求发生同样也有可能导致数据库发生宕机。在 Redis 机群的数据分布算法上如果使用的是传统的 hash 取模算法，在增加或者移除 Redis 节点的时候就会出现大量的缓存临时失效的情形。

解决方案：

1. 像解决缓存穿透一样加锁排队。
2. 建立备份缓存，缓存 A 和缓存 B，A 设置超时时间，B 不设值超时时间，先从 A 读缓存，A 没有读 B，并且更新 A 缓存和 B 缓存。
3. 计算数据缓存节点的时候采用一致性 hash 算法，这样在节点数量发生改变时不会存在大量的缓存数据需要迁移的情况发生。

#### 缓存并发问题

问题：多个 Redis 客户端同时 set 值引起的并发问题。

解决方案：把 set 操作放在队列中使其串行化，必须得一个一个执行。



### SpringBoot 整合Redis

1.创建一个SpringBoot项目

2.pom添加Redis依赖

```java
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>
```

3.application.properties配置Redis参数

```java
# Redis数据库索引（默认为0）
spring.redis.database=0 
# Redis服务器地址
spring.redis.host=localhost
# Redis服务器连接端口
spring.redis.port=6379 
# Redis服务器连接密码（默认为空）
spring.redis.password=
#连接池最大连接数（使用负值表示没有限制）
spring.redis.pool.max-active=8 
# 连接池最大阻塞等待时间（使用负值表示没有限制）
spring.redis.pool.max-wait=-1 
# 连接池中的最大空闲连接
spring.redis.pool.max-idle=8 
# 连接池中的最小空闲连接
spring.redis.pool.min-idle=0 
# 连接超时时间（毫秒）
spring.redis.timeout=5000
```

4.config文件夹下创建RedisConfig

```java
package com.example.demo.config;

import com.fasterxml.jackson.annotation.JsonAutoDetect;
import com.fasterxml.jackson.annotation.PropertyAccessor;
import com.fasterxml.jackson.databind.ObjectMapper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.condition.ConditionalOnMissingBean;
import org.springframework.boot.autoconfigure.jdbc.DataSourceAutoConfiguration;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.data.redis.connection.RedisConnectionFactory;
import org.springframework.data.redis.core.RedisTemplate;
import org.springframework.data.redis.core.StringRedisTemplate;
import org.springframework.data.redis.serializer.Jackson2JsonRedisSerializer;

/**
 * Created by ZhouFangyuan on 2020-02-24.
 * Time: 20:02
 * Information:
 */
@Configuration
@EnableAutoConfiguration(exclude = {DataSourceAutoConfiguration.class})
public class RedisConfig {
    final static Logger logger = LoggerFactory.getLogger(RedisConfig.class);

    @Bean
    @ConditionalOnMissingBean(name = "redisTemplate")
    public RedisTemplate<String, Object> redisTemplate(RedisConnectionFactory redisConnectionFactory) {
        Jackson2JsonRedisSerializer<Object> jackson2JsonRedisSerializer = new Jackson2JsonRedisSerializer<Object>(Object.class);
        ObjectMapper om = new ObjectMapper();
        om.setVisibility(PropertyAccessor.ALL, JsonAutoDetect.Visibility.ANY);
        om.enableDefaultTyping(ObjectMapper.DefaultTyping.NON_FINAL);
        jackson2JsonRedisSerializer.setObjectMapper(om);

        RedisTemplate<String, Object> template = new RedisTemplate<String, Object>();
        template.setConnectionFactory(redisConnectionFactory);
        //key采用jackson的序列化方式
        template.setKeySerializer(jackson2JsonRedisSerializer);
        template.setValueSerializer(jackson2JsonRedisSerializer);
        template.setHashKeySerializer(jackson2JsonRedisSerializer);
        template.setHashValueSerializer(jackson2JsonRedisSerializer);
        template.afterPropertiesSet();
        return template;
    }

    @Bean
    @ConditionalOnMissingBean(StringRedisTemplate.class)
    public StringRedisTemplate stringRedisTemplate(RedisConnectionFactory redisConnectionFactory) {
        StringRedisTemplate template = new StringRedisTemplate();
        template.setConnectionFactory(redisConnectionFactory);
        return template;
    }
}

```

SpringBoot 的 spring-boot-starter-data-redis 为 Redis 的相关操作提供了一个高度封装的 `RedisTemplate` 类，而且对每种类型的数据结构都进行了归类，将同一类型操作封装为 operation 接口。RedisTemplate 对五种数据结构分别定义了操作，如下所示：

- 操作字符串：`redisTemplate.opsForValue()`
- 操作 `Hash：redisTemplate.opsForHash()`
- 操作 `List：redisTemplate.opsForList()`
- 操作 `Set：redisTemplate.opsForSet()`
- 操作 `ZSet：redisTemplate.opsForZSet()`

但是对于 string 类型的数据，SpringBoot 还专门提供了 `StringRedisTemplate` 类，而且官方也建议使用该类来操作 String 类型的数据。那么它和 `RedisTemplate` 又有啥区别呢？

1. `RedisTemplate` 是一个泛型类，而 `StringRedisTemplate` 不是，后者只能对键和值都为 `String` 类型的数据进行操作，而前者则可以操作任何类型。
2. 两者的数据是不共通的，`StringRedisTemplate` 只能管理 `StringRedisTemplate` 里面的数据，`RedisTemplate` 只能管理 `RedisTemplate` 中 的数据。

参考资料：<https://www.ibm.com/developerworks/cn/java/know-redis-and-use-it-in-springboot-projects/index.html>

### Redis 分布式锁实现

加锁

```java
public class RedisTool {
  private static final String LOCK_SUCCESS = "OK";
  private static final String SET_IF_NOT_EXIST = "NX";
  private static final String SET_WITH_EXPIRE_TIME = "PX";

  /**
   * 尝试获取分布式锁
   * @param jedis Redis客户端
   * @param lockKey 锁
   * @param requestId 请求标识
   * @param expireTime 超期时间
   * @return 是否获取成功
   */
  public static boolean tryGetDistributedLock(Jedis jedis, String lockKey, String requestId, int expireTime) {

      String result = jedis.set(lockKey, requestId, SET_IF_NOT_EXIST, SET_WITH_EXPIRE_TIME, expireTime);

      if (LOCK_SUCCESS.equals(result)) {
          return true;
      }
      return false;

  }
}
```

解锁

```java
public class RedisTool {
  private static final Long RELEASE_SUCCESS = 1L;

  /**
   * 释放分布式锁
   * @param jedis Redis客户端
   * @param lockKey 锁
   * @param requestId 请求标识
   * @return 是否释放成功
   */
  public static boolean releaseDistributedLock(Jedis jedis, String lockKey, String requestId) {

      String script = "if redis.call('get', KEYS[1]) == ARGV[1] then return redis.call('del', KEYS[1]) else return 0 end";
      Object result = jedis.eval(script, Collections.singletonList(lockKey), Collections.singletonList(requestId));

      if (RELEASE_SUCCESS.equals(result)) {
          return true;
      }
      return false;

  }
}
```

参考资料：

<https://xiaozhuanlan.com/topic/4672859130>

<https://redis.io/topics/distlock>