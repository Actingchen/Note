#启动

windows  cmd 

```
redis-server redis.windows.conf
```

报错解决方案

解决方案如下，按顺序输入如下命令就可以连接成功

1. redis-cli.exe
2. shutdown
3. exit
4. redis-server.exe redis.windows.conf

# 基础入门

- 数据结构
  - 字符串——strings
  - 哈希——hashes
  - 列表——lists (lpush rpop lindex lrange )
  - 集合——sets
- 特点
  - Redis将所有的数据都存放在内存中，所以它的读写性能十分惊人
  - 同时，Redis还可以将内存中的数据以快照或日志的形式保存到硬盘上，以保证数据的安全性
- 场景
  - 缓存、排行榜、计数器、社交网络、消息队列(但不是专门的）等
- 缺点？
  - 快照因为是把数据存进硬盘，它比较耗时，所以做存储的时候会阻塞，快照不适合实时立即去做，定时任务去做。
  - 日志是追加去做，不适合恢复数据

# Spring整合redis

- 引入依赖

  ```
  <dependency>
              <groupId>org.springframework.boot</groupId>
              <artifactId>spring-boot-starter-data-redis</artifactId>
              <version>2.1.5.RELEASE</version>
  </dependency>
  ```

- 配置Redis

  - .properties文件中配置数据库参数

  ```yml
  #Redis
  spring.redis.database=11
  spring.redis.host=localhost
  spring.redis.port=6379
  spring.redis.password=xxxxx
  ```

  - 编写配置类，构造RedisTemplate

- 访问Redis

  - redisTemplate.opsForValue()
  - redisTemplate.opsForHash()
  - redisTemplate.opsForList()
  - redisTemplate.opsForSet()
  - redisTemplate.opsForZSet()

基本命令

info replication



# 传统的ACID

事务的四个特性：

* 原子性：事务里 的 所有操作要么全部做完，要么都不做
* 一致性：事务的运行不会改变数据库原本的一致性
* 独立性：并发的事务之间不会互相影响
* 持久性：事务一旦提交，它的修改会永久保存在数据库上，即使宕机也不会丢失

# CAP

C：强一致性

A：可用性

P：分区容错性

CA-单点集群，满足一致性，可用性的系统，通常在可扩展性上不太强-——传统数据库Oracle

CP-满足一致性，分区容忍性的系统，通常性能不是特别高-——Redis、HBase

AP-满足可用性，分区容忍性的系统，通常可能对一致性要求低一些-——大多数网站架构下选择6

> 一个分布式存储系统最多只能同时满足两个需求
>
> redis 满足C P即强一致性和分区容错性

# BASE

BASE就是为了解决关系数据库强一致性引起的可用性降低而提出的解决方案。

基本概念

* 基本可用
* 软状态
* 最终一致性

它的思想是通过让系统放松某一个时刻数据一致性的要求来换取系统整体伸缩性和性能上的改观。







# 发布订阅

发布者发送消息，订阅者接收消息

订阅者：

> SUBSCRIBE runoobChat(订阅频道名)

发布者

> PUBLISH runoobChat(订阅频道名) "Redis PUBLISH test"(新消息)



# 事务

Redis 事务可以一次执行多个命令， 并且带有以下三个重要的保证：

- 批量操作在发送 EXEC 命令前被放入队列缓存。
- 收到 EXEC 命令后进入事务执行，**事务中任意命令执行失败，其余的命令依然被执行。**
- 在事务执行过程，其他客户端提交的命令请求不会插入到事务执行命令序列中。

一个事务从开始到执行会经历以下三个阶段：

- 开始事务。
- 命令入队。
- 执行事务。

eg:

```
redis 127.0.0.1:6379> MULTI
OK

redis 127.0.0.1:6379> SET book-name "Mastering C++ in 21 days"
QUEUED

redis 127.0.0.1:6379> GET book-name
QUEUED

redis 127.0.0.1:6379> SADD tag "C++" "Programming" "Mastering Series"
QUEUED

redis 127.0.0.1:6379> SMEMBERS tag
QUEUED

redis 127.0.0.1:6379> EXEC
1) OK
2) "Mastering C++ in 21 days"
3) (integer) 3
4) 1) "Mastering Series"
   2) "C++"
   3) "Programming"
```

# 主从复制

配置从库

`SLAVEOF ip 端口号`

> 主机挂了，会怎么样？——主机挂了，如果不选主的话，从机 会原地待命
>
> 从机挂了，会怎么样？——需要重新执行SLAVEOF命令建立主从关系，除非已经写进配置里面



# 项目中Mybatis、Redis、MySQL

此时，如果是跨平台的数据是需要序列化的，（因为进行跨平台存储和网络传输的方式是io，io支持的数据格式就是字节）

所以进（数据涉及跨平台且redis）

* 涉及到的类要实现系列化Serializable的接口

* 如果适用jdk默认的序列化，有一个缺点是可读性很差

优化序列化

* 先把之前类中的序列化删除
* 创建类RedisConfiguration
* flushdb 清空redis的旧数据，因为改了序列化，老数据已经不能兼容了，必须清空