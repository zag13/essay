# Flink

## 组件

### JobManager

    - JobManager 控制着单个应用程序的执行
        1. 从 ResourceManager 申请执行任务的必要资源（TaskManager Solt）
        2. 收到了足够的数量的 TaskManager Solt 后
        3. 将 ExecutionGraph（一个 jar 文件） 中的任务分发给 TaskManager 来执行

### ResourceManager

    - ResourceManager 负责管理 Flink 的处理资源单元————TaskManager Solt
        1. 针对不同的环境和资源提供者，Flink 提供了不同的资源管理器
        2. JobManager 申请资源时，ResourceManager 会指示一个拥有空闲处理槽的 TaskManager 将其处理槽分配给 JobManager
        3. 处理槽数量不足时，ResourceManager 会和资源提供者进行通信，启动更多的 TaskManager 进程。同时，ResourceManager 还会负责终止空闲的 TaskManager 以释放计算资源

### TaskManager

    - TaskManager 是 Flink 的工作进程
        1. Flink 一般会启动多个 TaskManager，每个 TaskManager 提供一定数量的处理槽，处理槽的数目限制了一个 TaskManager 可执行的任务数
        2. TaskManager 启动后，会向

### Dispatcher

