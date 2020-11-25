#Redis

##前序

传统数据库的ACID

原子性（要么全部成功，要么全部失败）、一致性（事务的运行不会改变数据库）、独立性（事务之间不会互相影响）、持久性（一旦事务提交，他的修改会永久保存在数据库上，即使宕机也不会丢失）

**cap原理**（最多只能同时满足两个）

C：Consistency（强一致性）

A：可用性

P：分区容错性

CA-单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强大

CP-满足一致性，分区容忍性的系统，通常性能不高

AP：-满足可用性、分区容忍性的系统，通常可能堆一致性要求低一些

![image-20201125105632480](\images\image-20201125105632480.png)

TPS数据库每秒执行的事务数

QPS数据库每秒执行次数

1、数据库的每秒执行事务数有限制，很容易出现慢查询

2、比如 **CPU Cache 缓存的是内存数据用于解决 CPU 处理速度和内存不匹配的问题，内存缓存的是硬盘数据用于解决硬盘访问速度过慢的问题。** **再比如操作系统在 页表方案 基础之上引入了 快表 来加速虚拟地址到物理地址的转换。我们可以把快表理解为一种特殊的高速缓冲存储器（Cache）。**

回归到业务系统来说：**我们为了避免用户在请求数据的时候获取速度过于缓慢，所以我们在数据库之上增加了缓存这一层来弥补。**

![image-20201121201529996](C:\Users\11468\AppData\Roaming\Typora\typora-user-images\image-20201121201529996.png)

缓存与数据库数据库不同步怎么办？加一个中间件。或者采取一个定时同步。

想加节点怎么办？一致性哈希要怎么做？

redis挂了怎么办？数据库穿透怎么办？

## 概述

Redis是一款基于键值对的NoSQL数据库，它的值支持多种数据结构。

**Redis 的数据是存在内存中的** ，也就是它是内存数据库，所以读写速度非常快，因此 Redis 被广泛应用于缓存方向。

另外，**Redis 除了做缓存之外，Redis 也经常用来做分布式锁，甚至是消息队列。**

**Redis 提供了多种数据类型来支持不同的业务场景。Redis 还支持事务 、持久化、Lua 脚本、多种集群方案。**

##基础

* 数据结构
  * 字符串——strings
  * 哈希——hashes
  * 列表——lists (lpush rpop lindex lrange )
  * 集合——sets
* 特点
  * Redis将所有的数据都存放在内存中，所以它的读写性能十分惊人
  * 同时，Redis还可以将内存中的数据以快照或日志的形式保存到硬盘上，以保证数据的安全性
* 场景
  * 缓存、排行榜、计数器、社交网络、消息队列(但不是专门的）等
* 缺点？
  * 快照因为是把数据存进硬盘，它比较耗时，所以做存储的时候会阻塞，快照不适合实时立即去做，定时任务去做。
  * 日志是追加去做，不适合恢复数据

> Spring整合redis

* 引入依赖

  ```
  <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-data-redis</artifactId>
              <version>2.1.5.RELEASE</version>
  </dependency>
  ```

* 配置Redis

  * .properties文件中配置数据库参数

  ```yml
  #Redis
  spring.redis.database=11
  spring.redis.host=localhost
  spring.redis.port=6379
  spring.redis.password=xxxxx
  ```

  * 编写配置类，构造RedisTemplate

* 访问Redis

  * redisTemplate.opsForValue()
  * redisTemplate.opsForHash()
  * redisTemplate.opsForList()
  * redisTemplate.opsForSet()
  * redisTemplate.opsForZSet()



