# Kafka

Kafka是基于分布式的发布订阅的消息队列，主要应用于大数据的实时处理领域

# Kafka和MQ的区别

作为消息队列来说，Rabbit，Rocket等**mq性能一般但可靠性较强**

* 1)在架构模型方面，

RabbitMQ遵循AMQP协议，RabbitMQ的broker由Exchange,Binding,queue组成，其中exchange和binding组成了消息的路由键；客户端Producer通过连接channel和server进行通信，Consumer从queue获取消息进行消费（长连接，queue有消息会推送到consumer端，consumer循环从输入流读取数据）。rabbitMQ以broker为中心；有消息的确认机制。

kafka遵从一般的MQ结构，producer，broker，consumer，以consumer为中心，消息的消费信息保存的客户端consumer上，consumer根据消费的点，从broker上批量pull数据；无消息确认机制。

* 2)在吞吐量，

rabbitMQ在吞吐量方面稍逊于kafka，他们的出发点不一样，rabbitMQ支持对消息的可靠的传递，支持事务，不支持批量的操作；基于存储的可靠性的要求存储可以采用内存或者硬盘。

**kafka具有高的吞吐量**，内部采用消息的批量处理，zero-copy机制，数据的存储和获取是本地磁盘顺序批量操作，具有O(1)的复杂度，消息处理的效率很高。

* 3)在可用性方面，

rabbitMQ支持miror的queue，主queue失效，miror queue接管。

kafka的broker支持主备模式。

* 4)在集群负载均衡方面，

rabbitMQ的负载均衡需要单独的loadbalancer进行支持。

kafka采用zookeeper对集群中的broker、consumer进行管理，可以注册topic到zookeeper上；通过zookeeper的协调机制，producer保存对应topic的broker信息，可以随机或者轮询发送到broker上；并且producer可以基于语义指定分片，消息发送到broker的某分片上。

# 消息队列的好处

1、解耦，不需要客户端和服务端同时在线

2、做一个缓冲，削峰

3、可恢复性，当一个处理消息的进程挂掉，恢复之后任然可以继续处理消息

# 消息队列的两种模式

点对点模式（一对一，消费者主动拉取数据，消息消费后消息清除）

发布/订阅模式（一对多，消费者消费数据之后不会清除消息）

> 缺点是消费者需要去topic队列轮询有没有自己订阅的消息

# 架构

Cluster集群

zookeeper注册消息，管理集群

broker节点

 -Topic主题（把消息分类）

 -partition分区（提高负载能力，提高并发度，里面分为leader和followerr（副本），follower提供一个副本冗余（放在另一个节点）。

producer生产者

Consumer消费者（一个分区只能被同一个Consumer group里面的消费者消费）

Kafka不能保证消息的全局有序，只能保证消息在partition内有序，因为消费者消费消息是在不同的partition中随机的

# offset

每个partition对应于一个log文件，该log文件中存储的就是生产者生成的数据，生产者生成的数据会不断的追加到该log的文件末端，且每条数据都有自己的offset，消费者都会实时记录自己消费到了那个offset，以便出错的时候从上次的位置继续消费，这个offset就保存在index文件中

kafka的offset是分区内有序的，但是在不同分区中是无顺序的，kafka不保证数据的全局有序

> 引入分片和索引来防止log过大
>
> 二分查找索引 索引找到消息内容的位置offset

# 分区的策略

方便集群扩展（提高负载能力）、提高并发



我们需要将producer发送的数据封装成一个Producer Record对象。

分区原则：

（1）指明partition的情况下，直接将指明的值作为partition值

（2）没有指明partition值但有key的情况下，将key的hash值与topic的partition树进行取余得到partition；

（3）既没有parttion值又没有key值的情况下，第一次调用时随机生成一个整数（



ZooKeeper 主要为 Kafka 提供元数据的管理的功能。

从图中我们可以看出，Zookeeper 主要为 Kafka 做了下面这些事情：

1. **Broker 注册** ：在 Zookeeper 上会有一个专门**用来进行 Broker 服务器列表记录**的节点。每个 Broker 在启动时，都会到 Zookeeper 上进行注册，即到/brokers/ids 下创建属于自己的节点。每个 Broker 就会将自己的 IP 地址和端口等信息记录到该节点中去
2. **Topic 注册** ： 在 Kafka 中，同一个**Topic 的消息会被分成多个分区**并将其分布在多个 Broker 上，**这些分区信息及与 Broker 的对应关系**也都是由 Zookeeper 在维护。比如我创建了一个名字为 my-topic 的主题并且它有两个分区，对应到 zookeeper 中会创建这些文件夹：`/brokers/topics/my-topic/Partitions/0`、`/brokers/topics/my-topic/Partitions/1`
3. **负载均衡** ：上面也说过了 Kafka 通过给特定 Topic 指定多个 Partition, 而各个 Partition 可以分布在不同的 Broker 上, 这样便能提供比较好的并发能力。 对于同一个 Topic 的不同 Partition，Kafka 会尽力将这些 Partition 分布到不同的 Broker 服务器上。当生产者产生消息后也会尽量投递到不同 Broker 的 Partition 里面。当 Consumer 消费的时候，Zookeeper 可以根据当前的 Partition 数量以及 Consumer 数量来实现动态负载均衡。

#kafka自带的消费机制

　　kafka有个offset的概念，当每个消息被写进去后，都有一个offset，代表他的序号，然后consumer消费该数据之后，隔一段时间，会把自己消费过的消息的offset提交一下，代表我已经消费过了。下次我要是重启，就会继续从上次消费到的offset来继续消费。

　　但是当我们直接kill进程了，再重启。这会导致consumer有些消息处理了，但是没来得及提交offset。等重启之后，少数消息就会再次消费一次。

　　其他MQ也会有这种重复消费的问题，那么针对这种问题，我们需要从业务角度，考虑它的幂等性。

#通过保证消息队列消费的幂等性来保证

**就是用户对于同一操作发起的一次请求或者多次请求的结果是一致的，不会因为多次点击而产生了副作用。**　　

举个例子，当消费一条消息时就往数据库插入一条数据。如何保证重复消费也插入一条数据呢？

　　那么我们就需要从幂等性角度考虑了。幂等性，我通俗点说，就一个数据，或者一个请求，无论来多次，对应的数据都不会改变的，不能出错。

 

**怎么保证消息队列消费的幂等性？**

我们需要结合业务来思考，比如下面的例子：

　　1.比如某个数据要写库，你先根据主键查一下，如果数据有了，就别插入了，update一下好吧

　　2.比如你是写redis，那没问题了，反正每次都是set，天然幂等性

　　3.对于消息，我们可以建个表（专门存储消息消费记录）

　　　　生产者，发送消息前判断库中是否有记录（有记录说明已发送），没有记录，先入库，状态为待消费，然后发送消息并把主键id带上。

　　　　消费者，接收消息，通过主键ID查询记录表，判断消息状态是否已消费。若没消费过，则处理消息，处理完后，更新消息记录的状态为已消费。



