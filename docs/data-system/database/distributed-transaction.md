# 分布式事务

## A~~C~~ID

- **原子性(Atomicity)**

  一系列的操作整体不可拆分，要么同时成功，要么同时失败

- **一致性(Consistency)**

  对数据的一组特定约束必须始终成立（一致性（在 ACID
  意义上）是应用程序的属性）

- **隔离性(Isolation)**

  事务的执行是相互独立的，它们不会相互干扰，一个事务不会看到另一个正在运行过程中的事务的数据

- **持久性(Durabilily)**

  一个事务完成之后，事务的执行结果必须是落盘在数据库持久

## CA~~P~~

- **一致性(Consistency)**

  线性一致性，即所有节点在同一时间看到的数据是一样的

- **可用性(Availability)**

  在集群中一部分节点故障后，非故障的节点在合理的时间内返回合理的响应（保证返回正确的结果）

- **分区容错性(Partition tolerance)**

  系统能够在节点之间发生网络分区（Partition）的情况下仍然能够正常运行。在分布式系统中，网络无法100%可靠，分区其实是一个必然现象

## 分布式事务方案

分布式事务是为了解决分布式系统中的共识（数据一致性问题），保证分布式系统中的数据最终一致性。

在分布式系统中，事务提交必须是不可撤销的 ——
事务提交之后，你不能改变主意，并追溯性地中止事务。这个规则的原因是，一旦数据被提交，其结果就对其他事务可见，因此其他客户端可能会开始依赖这些数据。

### 2PC

``` mermaid
sequenceDiagram
    participant coordinator as 协调者
    activate coordinator
    coordinator-&gt;&gt;coordinator: 生成 t_id=1
    participant Alice
    participant Bob
    Note left of coordinator: I. 准备阶段
    par 
        coordinator-&gt;&gt;Alice: Prepare(t_id=1)
        activate Alice
        Alice-&gt;&gt;Alice: 锁定资源
        Alice--&gt;&gt;coordinator: Success
        deactivate Alice
    and 
        coordinator-&gt;&gt;Bob: Prepare(t_id=1)
        activate Bob
        Bob-&gt;&gt;Bob: 锁定资源
        Bob--&gt;&gt;coordinator: Success
        deactivate Bob
    end
    Note left of coordinator: II. 提交阶段
    par 
        coordinator-&gt;&gt;Alice: Commit(t_id=1)
        activate Alice
        Alice-&gt;&gt;Alice: 应用更改，释放资源
        Alice--&gt;&gt;coordinator: Success
        deactivate Alice
    and 
        coordinator-&gt;&gt;Bob: Commit(t_id=1)
        activate Bob
        Bob-&gt;&gt;Bob: 应用更改，释放资源
        Bob--&gt;&gt;coordinator: Success
        deactivate Bob
    end
    deactivate coordinator
```

两阶段提交（two-phase commit）有两个关键点：当参与者投票 “是”
时，它承诺它稍后肯定能够提交（尽管协调者可能仍然选择放弃）；以及一旦协调者做出决定，这一决定是不可撤销的。这些承诺保证了
2PC
的原子性（单节点原子提交将这两个事件合为了一体：将提交记录写入事务日志）。

### 3PC

``` mermaid
sequenceDiagram
    participant coordinator as 协调者
    activate coordinator
    coordinator-&gt;&gt;coordinator: 生成 t_id=1
    participant Alice
    participant Bob
    Note left of coordinator: I. 准备阶段
    par 
        coordinator-&gt;&gt;Alice: Prepare(t_id=1)
        activate Alice
        Alice-&gt;&gt;Alice: 检查资源
        Alice--&gt;&gt;coordinator: Success
        deactivate Alice
    and 
        coordinator-&gt;&gt;Bob: Prepare(t_id=1)
        activate Bob
        Bob-&gt;&gt;Bob: 检查资源
        Bob--&gt;&gt;coordinator: Success
        deactivate Bob
    end
    Note left of coordinator: II. 预提交阶段
    par 
        coordinator-&gt;&gt;Alice: Precommit(t_id=1)
        activate Alice
        Alice-&gt;&gt;Alice: 锁定资源
        Alice--&gt;&gt;coordinator: Success
        deactivate Alice
    and 
        coordinator-&gt;&gt;Bob: Precommit(t_id=1)
        activate Bob
        Bob-&gt;&gt;Bob: 锁定资源
        Bob--&gt;&gt;coordinator: Success
        deactivate Bob
    end
    Note left of coordinator: III. 提交阶段
    par 
        coordinator-&gt;&gt;Alice: Commit(t_id=1)
        activate Alice
        Alice-&gt;&gt;Alice: 应用更改，释放资源
        Alice--&gt;&gt;coordinator: Success
        deactivate Alice
    and 
        coordinator-&gt;&gt;Bob: Commit(t_id=1)
        activate Bob
        Bob-&gt;&gt;Bob: 应用更改，释放资源
        Bob--&gt;&gt;coordinator: Success
        deactivate Bob
    end
    deactivate coordinator
```

三阶段提交（three-phase
commit）相对于二阶段提交（2PC）有两个改进：引入了超时机制，以防止协调者在第二阶段失败时永远阻塞（超时自动提交）；将2PC的准备阶段拆分为两个阶段，这个阶段是重负载的操作（降低失败成本）。

### XA

XA（eXtended
Architecture）是一种分布式事务处理的标准协议，用于确保多个资源管理器（Resource
Manager）之间的事务一致性。它提供了在分布式环境中同时提交或回滚多个资源的机制。
