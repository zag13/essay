# Data System

## 起

数据库主要用于**事务处理**，存取款这种算是最典型的，特别强调每秒能干多少事儿：QPS（每秒查询数）、TPS（每秒事务数）、IOPS（每秒读写数）等等。

数据仓库主要用于**业务分析**相当于一个集成化数据管理的平台，从多个数据源抽取有价值的数据，在仓库内转换和流动，并提供给BI等分析工具来输出干货。

数据湖的本质，是由"数据存储架构"+"数据处理工具"组成的解决方案，而不是某个单一独立产品。

1. 数据存储架构，要有足够的扩展性和可靠性，要满足企业能把所有原始数据都“囤”起来，存得下、存得久。
   一般来讲，各大云厂商都喜欢用对象存储来做数据湖的存储底座，比如 Amazon Web Services（亚马逊云科技），修建“湖底”用的“砖头”，就是S3云对象存储
2. 数据处理工具，则分为两大类
    1. 第一类工具，解决的问题是如何把数据“搬到”湖里，包括定义数据源、制定数据访问策略和安全策略，并移动数据、编制数据目录等等
    2. 第二类工具，就是要从湖里的海量数据中“淘金”

## 承

### 流计算框架对比

| Apache | Flink                                                               | SparkStreaming                                            | Storm                                                                             | Samza                                                |
|--------|---------------------------------------------------------------------|-----------------------------------------------------------|-----------------------------------------------------------------------------------|------------------------------------------------------|
| 架构     | 架构介于spark和storm之间，主从结构与spark streaming相似，DataFlow Grpah与Storm相似     | 架构依赖spark，主从模式，每个Batch处理都依赖主（driver），可以理解为时间维度上的spark DAG | 主从模式，且依赖ZK，处理过程中对主的依赖不大                                                           | 过度依赖kafka，接入数据源相对单一，Samza处理数据流时，会分别按次处理每条收到的消息       |
| 处理模式   | Native                                                              | Micro-batch                                               | Native                                                                            | Native                                               |
| 容错     | 基于Chandy-Lamport distributed snapshots checkpoint机制   <br>     High | WAL及RDD 血统机制            <br>     High                     | Records ACK       <br>     Medium                                                 | Log-Based                      <br>     Medium       |
| 处理模型   | 单条事件处理                                                              | 一个事件窗口内的所有事件                                              | 单条事件处理                                                                            | 单条事件处理                                               |
| 延迟     | 毫秒级低延迟                                                              | 秒级低延迟                                                     | 毫秒级低延迟                                                                            | 毫秒级低延迟                                               |
| 吞吐量    | High                                                                | High                                                      | Low                                                                               | High                                                 |
| 数据处理保证 | exactly once                                                        | exactly once（实现采用ChandyLamport 算法，即marker-checkpoint）     | at least once(实现采用record-level acknowledgments)，Trident可以支持storm 提供exactly once语义 | at least once                                        |
| 高级API  | Flink栈中提供了提供了很多具有高级API和满足不同场景的类库：机器学习、图分析、关系式数据处理                   | 能够很容易的对接Spark生态栈里面的组件，同时能够对接主流的消息传输组件及存储系统                | 应用需要按照特定的storm定义的规则编写                                                             | Samza只支持JVM语言                                        |
| 易用性    | 支持SQL Steaming，Batch和STREAMING采用统一编程框架                              | 支持SQL Steaming Batch和STREAMING采用统一编程框架                    | 不支持SQL Steaming                                                                   | Samza提供的高级抽象使其在很多方面比Storm等系统提供的基元（Primitiv e）更易于配合使用 |
| 成熟性    | 已经发展一段时间                                                            | 相对较早的流系统，比较稳定                                             | 相对较早的流系统，比较稳定                                                                     | 相对较早的流系统，比较稳定                                        |
| 社区活跃度  | Contributors 1,095                                                  | Contributors 1,876                                        | Contributors 351                                                                  | Contributors 138                                     |
| 部署性    | 部署相对简单，只依赖JRE环境                                                     | 部署相对简单，只依赖JRE环境                                           | 依赖JRE环境和Zookeeper                                                                 | 依赖JRE环境                                              |

### HDFS

### OBS

### Hudi

### MapReduce

### Hive

### Spark

### SparkSQL

### SparkStreaming

### HetuEngine

### HetuEngine跨域

### Flink

### Flink

### DWS

### RDS

### HBase

### ElasticSearch

### ClickHouse

### IoTDB

### GES

### Redis

### Kafka

## 转

### MRS

### DLI

### CSS

### GaussDB(DWS)

## 合
