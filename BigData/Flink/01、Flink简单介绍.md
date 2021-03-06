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

## 基础

### 程序流程

1. 设置执行环境
    - 集群、本地、流处理、批处理
2. 读取一到多个数据源
3. 根据业务逻辑对数据流进行转换
4. 将结果输出到 Sink
5. 调用作业执行函数

### 数据类型

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

## 开发

**看官网，官网很清晰**

## 参考资料

[Apache Flink 文档](https://nightlies.apache.org/flink/flink-docs-release-1.15/zh/)