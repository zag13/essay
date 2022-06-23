# Flink

## 组件

### JobManager

JobManager 控制着单个应用程序的执行

1. 从 ResourceManager 申请执行任务的必要资源（TaskManager Solt）
2. 收到了足够的数量的 TaskManager Solt 后
3. 将 ExecutionGraph（一个 jar 文件） 中的任务分发给 TaskManager 来执行

### ResourceManager

ResourceManager 负责管理 Flink 的处理资源单元————TaskManager Solt

1. 针对不同的环境和资源提供者，Flink 提供了不同的资源管理器
2. JobManager 申请资源时，ResourceManager 会指示一个拥有空闲处理槽的 TaskManager 将其处理槽分配给 JobManager
3. 处理槽数量不足时，ResourceManager 会和资源提供者进行通信，启动更多的 TaskManager 进程。同时，ResourceManager 还会负责终止空闲的 TaskManager 以释放计算资源

### TaskManager

TaskManager 是 Flink 的工作进程

1. Flink 一般会启动多个 TaskManager，每个 TaskManager 提供一定数量的处理槽，处理槽的数目限制了一个 TaskManager 可执行的任务数
2. TaskManager 启动后，会向 ResourceManager 注册它的处理槽
3. 接收到 ResourceManager 的指示时，TaskManager 会向 JobManager 提供一个或多个处理槽

### Dispatcher

Dispatcher 会跨多个作业运行，他会提供一个 REST 接口来让我们提交需要执行的应用

## 程序流程

1. 设置执行环境
    - 集群、本地、流处理、批处理
2. 读取一到多个数据源
3. 根据业务逻辑对数据流进行转换
4. 将结果输出到 Sink
5. 调用作业执行函数

## 数据类型

- 基础类型
    - Java 的所有基础数据类型，诸如int、Double、Long(包括Java原生类型int和装箱后的类型Integer)、String，以及Date、BigDecimal和BigInteger
- 数组
    - 基础类型或其他对象类型组成的数组，如String[]
- Java POJO
    - 该类必须用 public 定义
    - 该类必须有一个 public 的无参数的构造函数
    - 该类的所有非静态(non-static)、非瞬态(non-transient)字段必须是public的，如果字段不是public的则必须有标准的 getter 和 setter 方法，比如对于字段 A 有 getA() 和
      setA(a)
    - 所有子字段必须是Flink支持的数据类型
- Tuple
    - Tuple即元组
- 辅助类型
    - Java的ArrayList、HashMap和Enum

## 常见 Transformation

单数据流基本转换

> 单数据流基本转换主要对单个数据流上的各元素进行处理

- **map**
    - map()对一个DataStream中的每个元素使用用户自定义的Mapper函数进行处理，每个输入元素对应一个输出元素，最终整个数据流被转换成一个新的DataStream
- **filter**
    - filter()对每个元素进行过滤，过滤的过程使用一个Filter函数进行逻辑判断
- **flatMap**
    - flatMap()和map()有些相似，输入都是数据流中的每个元素，与之不同的是，flatMap()的输出可以是零个、一个或多个元素

基于Key的分组转换

> 对数据分组主要是为了进行后续的聚合操作，即对同组数据进行聚合分析

- **keyBy**
    - 将一个 DataStream 转换为一个 KeyedStream
    - 根据事件的某种属性或数据的某个字段进行分组，然后对一个分组内的数据进行处理
- **Aggregation**
    - 将一个 KeyedStream 转换为一个 DataStream
    - 常见的聚合(Aggregation)函数有sum()、max()、min()等，这些聚合函数统称为聚合
- **reduce**
    - 两两合一地进行汇总操作，生成一个同类型的新元素

多数据流转换

- **union**
    - 在 DataStream 上使用union()可以合并多个同类型的数据流
    - 数据将按照先进先出(First In First Out)的模式合并，且不去重
- **connect**
    - connect()只能连接两个数据流，union()可以连接多个数据流
    - connect()所连接的两个数据流的数据类型可以不一致，union() 所连接的两个或多个数据流的数据类型必须一致
    - 两个DataStream经过connect()之后被转化为ConnectedStreams，ConnectedStreams会对两个流的数据应用不同的处理方法，且两个流之间可以共享状态

数据重分布

- **shuffle**
    - shuffle()基于正态分布，将数据随机分布到下游各算子子任务上
- **rebalance 与 rescale**
    - rebalance() 使用Round-Ribon方法将数据均匀分布到各子任务上
    - rescale() 就近发送给下游子任务
- **broadcast**
    - rescale() 发送给下游的所有子任务上
- **global**
    - global()会将所有数据发送给下游算子的第一个子任务上，使用global()时要小心，以免造成严重的性能问题
- **partitionCustom**

自定义函数

- **接口**
    - 对于map()、flatMap()、reduce()等函数，我们可以实现MapFunction、FlatMapFunction、ReduceFunction等接口
- **Lambda表达式**
- **Rich函数类**

####                         

## 时间和窗口

### 时间语义

> 在 1.12 以后版本默认是使用 EventTime，如果要显示使用 ProcessingTime，可以关闭 watermark（自动生成 watermark 的间隔设置为 0）
> `env.getConfig().setAutoWatermarkInterval(0);`

Event Time

- 一般是事件发生的时间，由于事件从发生到进入Flink时间算子之间有很多环节，一个较早发生的事件因为延迟可能较晚到达，因此使用Event Time意味着事件到达有可能是乱序的
- 在实际应用中，当涉及对事件按照时间窗口进行统计时，Flink会将窗口内的事件缓存下来，直到接收到一个Watermark，Watermark假设不会有更晚到达的事件
- Watermark意味着在一个时间窗口下，Flink会等待一个有限的时间，这在一定程度上降低了计算结果的绝对准确性， 而且增加了系统的延迟

Processing Time

- 对于某个算子来说，Processing Time指使用算子的当前节点的操作系统时间
- 对于一个程序，在同一个环境中，每个算子都有一定的耗时，同一个事件的Processing Time，第n个算子和第n+1个算子的不同
- Processing Time在时间窗口下的计算会有不确定性

### 窗口算子

在流处理场景下,数据以源源不断的流的形式存在,数据一直在产生,没有始末。我们要对数据进行处理时,往往需要明确一个时间窗口,在一个时间窗口维度上对数据进行聚合,划分窗口是流处理需要解决的问题。

- 滚动窗口（Tumbling Window）
    - 滚动窗口模式下,窗口之间不重叠,且窗口长度(Window Size)是固定的
- 滑动窗口（Sliding Window）
    - 滑动窗口以一个步长(Slide)不断向前滑动,窗口的Size固定
- 会话窗口（Session Window）
    - 会话窗口模式下,两个窗口之间有一个间隙,称为Session Gap。当一个窗口在大于Session Gap的时间内没有接收到新数据时,窗口将关闭

窗口处理函数

- 增量计算
  > 窗口保存一份中间数据,每流入一个新元素,新元素与中间数据两两合一,生成新的中间数据,再保存到窗口中
    - ReduceFunction
    - AggregateFunction
- 全量计算
  > 指的是窗口先缓存所有元素,等触发条件后才对窗口内的全量元素执行计算
    - ProcessWindowFunction
- ProcessWindowFunction与增量计算相结合
  > 当我们想访问窗口里的元数据,但又不想缓存窗口里的所有数据时,可以将ProcessWindowFunction与增量计算函数reduce()和aggregate()相结合
- 拓展和自定义窗口
    - Trigger
        - 触发器(Trigger)决定了何时启动窗口处理函数来处理窗口中的数据,以及何时将窗口内的数据清除
    - Evictor
        - 清除器(Evictor)是在WindowAssigner和Trigger的基础上的一个可选选项,用来清除一些数据
        - 可以在窗口处理函数执行前或执行后调用Evictor

### 双流连接

- 窗口连接（Window Join）
- 时间间隔连接（Interval Join）

### 处理迟到数据

- 直接将迟到数据丢弃
- 将迟到数据发送到另一个数据流
- 重新执行一次计算,将迟到数据考虑进来,更新计算结果

## 状态和检查点

### 状态

### 检查点

### 保存点

## 参考资料

Flink 原理和实践