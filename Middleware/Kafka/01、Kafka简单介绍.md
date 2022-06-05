# Kafka

## 基本概念

- 主题（Topic）
    - kafka 将一组消息抽象为一个主题
- 消息（Record）
    - kafka 通信的基本单位，由一个固定长度的消息头和一个可变长度的消息体构成。
- 分区和副本（Partition、Replica）
    - 每个主题被分成一个或多个分区，每个分区又有一至多个副本
    - 每个分区由一系列有序、不可变的消息组成，是一个有序队列
    - 分区的副本分布在集群的不同代理上
- Leader 副本和 Follower 副本
- 偏移量
    - 任何发布到分区的消息都会被追加进日志文件的尾部，消息在该日志文件中的位置就会对应一个按序增加的偏移量
- 日志段（LogSegment）
    - 一个日志又被划分为多个日志段，日志段是 kafka 日志对象分片的最小单位
    - 一个日志段对应磁盘上的一个具体日志文件（.log）和两个索引文件（.index、.timeindex）
- 代理（Broker）
    - kafka 集群是由一个或多个 kafka 实例构成，其中的 kafka 实例被称为代理
- 生产者
    - 向 kafka 代理发送消息的客户端
- 消费者和消费组
    - 消费的客户端
    - 每一个消费者都属于一个特定消费组
    - 同一主题下的一个消息只能被同一个消费组下某一个消费者消费，但不同消费组的消费者可以同时消费该消息
- ISR（In-sync Replica）
    - 保存同步的副本列表
- ZooKeeper
    - kafka 使用 zookeeper 保存相应元数据
    - kafka3.0 开始逐步测试使用 KRaft 代替 zookeeper

## 重要配置

`server.properties`  服务端的配置文件

```
# broker的全局唯一编号，不能重复
broker.id=0
 
# 用来监听链接的端口，producer、consumer将在此端口建立连接
port=9092
 
# 处理网络请求的线程数量，也就是接收消息的线程数
num.network.threads=3
 
# 消息从内存中写入磁盘是时候使用的线程数量
num.io.threads=8
 
# Socket 发送消息缓冲区大小
socket.send.buffer.bytes=102400
 
# Socket 接收消息缓冲区大小
socket.receive.buffer.bytes=102400
 
# 请求套接字的缓冲区大小
socket.request.max.bytes=104857600
 
# kafka运行日志存放的路径
log.dirs=/tmp/kafka-logs
 
# topic 在当前 broker 上的分片个数
num.partitions=2
 
# 用来恢复和清理data下数据的线程数量
num.recovery.threads.per.data.dir=1
 
# segment 文件保留的最长时间，默认保留7天（168小时），
log.retention.hours=168
 
# 滚动生成新的 segment 文件的最大时间
log.roll.hours=168
 
# 日志文件中每个 segment 的大小，默认为1G
log.segment.bytes=1073741824
 
# 多久检查一次文件大小
log.retention.check.interval.ms=300000
 
# 日志清理是否打开
log.cleaner.enable=true
 
# broker 需要使用 zookeeper 保存元数据
zookeeper.connect=zk-01:2181,zk-02:2181,zk-03:2181
 
# zookeeper 连接超时时间
zookeeper.connection.timeout.ms=6000
 
# partion buffer中，消息的条数达到阈值，将触发将消息从内存刷到磁盘
log.flush.interval.messages=9223372036854775807
 
# 消息 buffer 的时间，达到阈值，将触发将消息从内存刷到磁盘
log.flush.interval.ms=
 
# 允许删除 topic
delete.topic.enable=true
```

`producer.properties`  生产端的配置文件

```
# kafka 节点列表，用于获取 metadata，不必全部指定
metadata.broker.list=kafka01:9092,kafka02:9092,kafka03:9092
 
# 0:不在乎是否写入成功； 1:写入leader成功； 2:写入leader和所有副本都成功；
request.required.acks=0
 
# sync:同步； async:异步；
producer.type=sync
 
# 当 producer 接收到 error ACK,或者没有接收到 ACK 时,允许在丢弃该消息前重试的次数
message.send.max.retries=3
 
# producer 每次重试前会更新 topic 的 MetaData 信息，以检查新的 Leader 是否被选举出来
# 此选项指定更新 topic 的 MetaData 之前 producer 要等待的时间
retry.backoff.ms=100

# async 模式下，当 message 被缓存的时间超过此值后，会被批量发送给 broker
queue.buffering.max.ms = 5000

# async 模式下，producer 允许缓存的最大消息量
queue.buffering.max.messages=10000

# async 模式下，指定每次批量发送的数据量
batch.num.messages=200

# 在需要 ack 时，producer 等待 broker 应答的超时时间，否则发送错误到客户端
request.timeout.ms=1500

# Socket 发送消息缓冲区大小
send.buffer.bytes=102400

# producer 刷新 topic metadata 的时间间隔
topic.metadata.refresh.interval.ms=60000

# producer 指定的一个标识字段，用来追踪调用
client.id=console-producer

# -1:不限制阻塞超时时间 0:立即清空队列,消息被抛弃 uint:"阻塞"的时间
queue.enqueue.timeout.ms=-1
```

`consumer.properties`  消费端的配置文件

```
# 消费者集群通过连接 ZK 来找到 broker
zookeeper.connect=zk-01:2181,zk-02:2181,zk-03:2181
 
# zookeeper 的 session 过期时间，用于检测消费者是否挂掉
zookeeper.session.timeout.ms=18000
 
# 当消费者挂掉，其他消费者要等该指定时间才能检查到并且触发重新负载均衡
zookeeper.connection.timeout.ms=10000
 
# ZK follower 可以落后于 ZK leader 多长时间
zookeeper.sync.time.ms=2000
 
# 消费组ID
group.id=/
 
# 消费者客户端编号，用于区分不同客户端，默认客户端程序自动产生
client.id=/

# 消息的 Key 反序列化类
key.deserializer=/

# 消息的 Value 反序列化类
value.deserializer=/

# 是否开启自动提交消费偏移量
enable.auto.commit=true

# 自动提交消费偏移量的时间间隔
auto.commit.interval.ms=5000

# 一次拉取消息的最大数量
max.poll.records=500

# 使用消费组管理消费者时，消费者超过这个时间还没有 poll 动作，
# 消费组会认为消费者已离开消费组，触发平衡操作
max.poll.interval.ms=3000000
 
# Socket 发送消息缓冲区大小
send.buffer.bytes=131072

# Socket 接收消息缓冲区大小
receive.buffer.bytes=65536
 
# 一次拉取操作等待消息的最小字节数
fetch.min.bytes=1

# 一次拉取操作获取消息的最大字节数
fetch.max.bytes=57671680

# 若是不满足 fetch.min.bytes 时，客户端等待请求的最长等待时间
fetch.wait.max.ms=500
```

## 推荐阅读

[KAFKA DOCS](https://kafka.apache.org/documentation)

[Kafka 入门与实践](https://search.jd.com/Search?keyword=kafka%E5%85%A5%E9%97%A8%E4%B8%8E%E5%AE%9E%E8%B7%B5)

[Kafka 权威指南](https://search.jd.com/Search?keyword=kafka%E6%9D%83%E5%A8%81%E6%8C%87%E5%8D%97)
