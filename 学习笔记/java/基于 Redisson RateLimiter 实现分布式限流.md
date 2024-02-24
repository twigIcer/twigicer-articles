# 基于 Redisson RateLimiter 实现分布式限流

上篇文章写了限流算法，和限流的实现，提到了使用 Redisson RateLimiter 实现分布式限流，但是并不细致，这篇文章补充记录下基于 Redisson RateLimiter 实现分布式限流的具体步骤和代码。

> Redisson RateLimiter 和 分布式限流的概念不再介绍，可以参考上一篇文章：[twigicer's blog - 爱上小树枝 (xiaoshuzhi.love)](https://www.xiaoshuzhi.love/2024/02/17/study-12/)

## 实现步骤:

官方仓库地址及教程：[redisson/redisson: Redisson - Easy Redis Java client with features of In-Memory Data Grid. Sync/Async/RxJava/Reactive API. Over 50 Redis based Java objects and services: Set, Multimap, SortedSet, Map, List, Queue, Deque, Semaphore, Lock, AtomicLong, Map Reduce, Bloom filter, Spring Cache, Tomcat, Scheduler, JCache API, Hibernate, RPC, local cache ... (github.com)](https://github.com/redisson/redisson)

### 1. 下载安装redis并启动：

这步不介绍，有很多相关教程，安装方式很简单，傻瓜式安装。

redis官方下载地址：[Download | Redis](https://redis.io/download/)

### 2. 引依赖：

导入 Redisson  和 Redis 的依赖：

```xml
<!--Redis依赖-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-data-redis</artifactId>
</dependency>

<!--Redission依赖-->
<dependency>
    <groupId>org.redisson</groupId>
    <artifactId>redisson</artifactId>
    <version>3.21.3</version>
</dependency>
```

### 3. 在yml文件中填写redis配置：

在 application.yml 文件中填写自己的Redis配置，如果有密码需要填写密码：

```yml
spring:  
  redis:
    database: 1
    host: localhost
    port: 6379
    timeout: 5000
```

注意：更规范的写法是将 Redission 配置独立出来，单独配置：

```yml
spring: 
  redis:
    database: 1
    host: localhost
    port: 6379
    timeout: 5000
  redission:
    database: 2
    host: localhost
    port: 6379
    timeout: 5000
```

我为了简单演示使用第一种。

### 4. 创建客户端：

编写一个创建Redission客户端的配置类：

```java
import lombok.Data;

import org.redisson.Redisson;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;
import org.springframework.boot.context.properties.ConfigurationProperties;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;

@Configuration
@ConfigurationProperties(prefix = "spring.redis")
@Data
public class RedissonConfig {

    private Integer database;

    private String host;

    private Integer port;

    private String password;

    @Bean
    public RedissonClient redissonClient() {
        Config config = new Config();
        config.useSingleServer()
                .setDatabase(database)
                .setAddress("redis://" + host + ":" + port);
        RedissonClient redisson = Redisson.create(config);
        return redisson;
    }
}
```

注意：

1. `@ConfigurationProperties(prefix = "spring.redis")`  可以去读取 application.yml 里以  "spring.redis" 开头的配置，并将配置的值映射到该类中名称相同的属性中。

   比如：`private Integer database;` 就能将 application.yml文件中的 `spring.redis.datebase` 的值 1 填充给类里的database 

2. 如果第3步中独立配置了 Redission ，就应该将注解的前缀改为 `spring.redission` 即：`@ConfigurationProperties(prefix = "spring.redission")` 

3. 如果配置了密码，应该将密码设置进去：

   ```java
   config.useSingleServer().setPassword(password);
   ```

### 5.  配置限流规则：

编写一个通用类来配置限流规则：

```java
import com.twigIcer.twigBI.common.ErrorCode;
import com.twigIcer.twigBI.exception.BusinessException;
import org.redisson.api.RRateLimiter;
import org.redisson.api.RateIntervalUnit;
import org.redisson.api.RateType;
import org.redisson.api.RedissonClient;
import org.springframework.stereotype.Service;

import javax.annotation.Resource;

/**
 * 专门提供 RedisLimiter 限流基础服务的（提供了通用的能力）
 */
@Service
public class RedisLimiterManager {

    @Resource
    private RedissonClient redissonClient;

    /**
     * 限流操作
     *
     * @param key 区分不同的限流器，比如不同的用户 id 应该分别统计
     */
    public void doRateLimit(String key) {
        // 创建一个名称为user_limiter的限流器，每秒最多访问 2 次
        RRateLimiter rateLimiter = redissonClient.getRateLimiter(key);
        rateLimiter.trySetRate(RateType.OVERALL, 2, 1, RateIntervalUnit.SECONDS);
        // 每当一个操作来了后，请求一个令牌
        boolean canOp = rateLimiter.tryAcquire(1);
        if (!canOp) {
            throw new BusinessException(ErrorCode.TOO_MANY_REQUEST);
        }
    }
}
```

代码解释：

1. `public void doRateLimit(String key) {`: 这是一个公有方法，命名为`doRateLimit`，接受一个参数`key`，用于标识要限流的资源。
2. `RRateLimiter rateLimiter = redissonClient.getRateLimiter(key);`: 这一行通过Redisson客户端`redissonClient`获取一个名为`key`的限流器对象`rateLimiter`。Redisson是一个基于Redis的Java库，它提供了许多分布式对象和服务，包括分布式限流器。
3. `rateLimiter.trySetRate(RateType.OVERALL, 2, 1, RateIntervalUnit.SECONDS);`: 这一行设置了限流器的速率。它指定了以秒为单位的速率为每秒2次（2次请求）。参数含义分别是：速率类型（OVERALL表示总体速率），速率（每秒2次），权重（这里为1，通常用于更复杂的限流策略，这里不起作用），速率单位（秒）。
4. `boolean canOp = rateLimiter.tryAcquire(1);`: 这一行尝试从限流器获取一个令牌（即许可证）。`tryAcquire`方法尝试获取指定数量的令牌，这里是1个。如果有足够的令牌可用，则返回`true`，否则返回`false`。
5. `if (!canOp) { throw new BusinessException(ErrorCode.TOO_MANY_REQUEST); }`: 这一行检查前一步是否成功获取了令牌。如果没有成功（即返回`false`），则抛出一个业务异常，其错误代码为`TOO_MANY_REQUEST`，表示请求过多。

## 测试：

编写一个测试类来测试限流功能：

```java
import org.junit.jupiter.api.Test;
import org.springframework.boot.test.context.SpringBootTest;

import javax.annotation.Resource;

@SpringBootTest
class RedisLimiterManagerTest {

    @Resource
    private RedisLimiterManager redisLimiterManager;

    @Test
    void doRateLimit() throws InterruptedException {
        String userId = "1";
        for (int i = 0; i < 5; i++) {
            redisLimiterManager.doRateLimit(userId);
            System.out.println("成功");
        }
    }
}
```

向userId为1的限流器连续发送5次请求，可以看到只能成功两次：

![image-20240217190442513](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240217190442513.png)

修改测试代码：

```java
@SpringBootTest
class RedisLimiterManagerTest {

    @Resource
    private RedisLimiterManager redisLimiterManager;

    @Test
    void doRateLimit() throws InterruptedException {
        String userId = "1";
        for (int i = 0; i < 2; i++) {
            redisLimiterManager.doRateLimit(userId);
            System.out.println("成功" + i);
        }
        Thread.sleep(1000);
        for (int i = 0; i < 5; i++) {
            redisLimiterManager.doRateLimit(userId);
            System.out.println("成功" + i);
        }
    }
}
```

先向userid为1的限流器中连续发送2次请求，休眠1s，再连续向该限流器中发送5次请求，可以看到只有第一次两次请求和第二次的前两次请求成功：

![image-20240217191037982](https://cdn.jsdelivr.net/gh/twigIcer/markdown-img@main/imgs/image-20240217191037982.png)

## 小结：

还是那句话，其实使用各个组件的步骤都是一样的：引依赖 - 写配置 - 创建客户端 - 自定义组件使用规则 - 使用。

在正式项目中使用时，可以在Key中加一个方法名前缀，表示对该方法进行限流。

最好的教程就是官方文档，虽然有的官方文档写的确实有点拉，但是至少更新比较及时，不会出现版本问题。