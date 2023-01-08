# Flink

## 核心特性

- 统一数据处理组件栈,处理不同类型的数据需求(Batch,Stream,Machine Learning,Graph)
- 支持事件时间(EventTime),接入时间(Ingestion Time),处理时间(Processing Time)等时间概念
- 基于轻量级分布式快照(Snapshot)实现的容错
- 支持有状态计算
    - Support for very large state
    - queryable state支持
    - 灵活的state-backend(HDFS,内存,RocksDB)
- 支持高度灵活的窗口(Window)操作
- 带反压的连续流模型
- 基于JVM 实现独立的内存管理:
    - Flink 在 JVM 中实现了自己的内存管理。
- 应用可以超出主内存的大小限制,并且承受更少的垃圾收集的开销
- 对象序列化二进制存储,类似于C对内存的管理