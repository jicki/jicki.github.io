# MySQL MGR 高可用方案


# {{< figure src="/img/posts/mysql/mysql-logo.png" >}}


#  MySQL MGR 介绍

**MGR(MySQL Group Replication) 是一个MySQL Server插件, 可用于创建弹性, 高可用MySQL集群方案。通过内置的组成员服务, 在任何给定的时间点, 保持组的视图一致并可供所有服务器使用。服务器成员可以随时离开或加入组, 视图也会相应更新。 当成员离开组, 故障检测机制会检测到此情况并通知组视图已更改。**

---


* `MySQL Group Replication` 是建立在基于 `Paxos` 的 `XCom` 之上的, 正因为有了`XCom`基础设施, 保证数据库状态机在节点间的事务一致性, 才能在理论和实践中保证数据库系统在不同节点间的事务一致性。


---


# MySQL MGR 技术演进

---

## MySQL 主从复制


---

**传统的 MySQL 主从复制架构是 MySQL 保持数据一致性的最基本架构** 


---

{{< figure src="/img/posts/mysql/master-slave.png" >}}


---

* 主从复制架构:  从库给主库发起读数据请求后, 主库会通过 `dump` 线程 把 `binlog` 日志文件 推送 给从库, 从库的 `I/O` 线程把接收到数据更新到 `relay log`, 之后从库的 `SQL` 线程把 `relay log` 应用为 `binlog` 日志, 直到主库与从库的 `binlog` 日志文件完全数据一致, 达到主从同步。**


---

## MySQL 半同步复制

---

**半同步复制是建立在基本的主从复制基础上，利用插件完成半同步复制，传统的主从复制，不管从库是否正确获取到二进制日志，主库不断更新，半同步复制则当确认了从库把二进制日志写入中继日志才会允许提交，如果从库迟迟不返回ack,主库会自动将半同步复制状态取消，进入最基本的主从复制模式。**


---

{{< figure src="/img/posts/mysql/master-slave-2.png" >}}


---

* 半同步复制架构:  应用发来的事务请求，在主库执行后写入 `binlog`, 主库 `master` 把 `binlog` 日志推 送给 从库 `salve1` 和 `slave2` , 半同步主库需要等待其中任意一个从库更新数据到 `relay log` 成功并且告知主库, 主库才提交事务, 这样保证至少有一个从库同步上数据了, 也缩短了延迟时间, 保证了数据安全。


---

## MySQL 组复制

---

**MySQL MGR 集群是多个 MySQL Server 节点共同组成的分布式集群, 每个 Server 都有完整的副本, 它是基于 ROW 格式的二进制日志文件和 GTID 特性。**

---

{{< figure src="/img/posts/mysql/mysql-mgr.png" >}}


---

* 组复制架构:  应用发来的事务从 `MySQL Server` 经过 `MGR` 的 `APIs` 接口层分发到组件层, 组件层去 `capture` 事务相关信息, 然后经过复制协议层进行事务传输, 最后经过 `GCS API` + `Paxos` 引擎层保证事务在各个节点数据最终一致性。这是事务进入 `MGR` 层内部处理过程。


---

# MGR 事务生命周期

## MGR 各模块原理

---

* 从全局角度看事务整个生命周期, 如上图，Master1 、Master2 、Master3 构成的 `MGR` 集群, 集群中每个 Master 节点, 都有 `MGR` 层, `MGR` 层功能也可简单理解为由 `Paxos` 模块 和 冲突检测 `Certify` 模块 实现。

    * `MGR`层:  既 `Consensus` + `Certify` 组成。

---

* `Paxos` 模块是基于 `Paxos` 算法确保所有节点收到相同广播消息, `transaction message` 就是广播消息的内容结构; 冲突检测 `Certify` 模块进行冲突检测确保数据最终一致性, 其中 `certification info` 是冲突检测中内存结构。

    * `transaction message` 广播信息: 既 保存着事务要更新行的的相关信息, 有 `transaction_context_log_event` (事务上下文信息) 和 `gtid_log_event` 及 `log_event_group` 三部分组成。 其中 `transaction_context_log_event` 又由 `write set` 以及 `gtid_executed` 组成。

        * `write set` 叫写入集合, 是事务更新行相关信息的 `Hash` 值。 `write set=Hash` (库名+表名+主键(唯一键)字段信息)

        * `gtid_executed` 为已经执行过的事务 `gtid` 集合, 也即事务快照版本。

        * `gtid_log_event` 为已经执行过的事务 `gtid` 集合。

        * `log_event_group` 为事务日志信息, 后续要更新到 `relay log` 中。 

    * `certification info`:  保存了通过冲突检测的事务的 `write set` 和 `gtid_executed`。 certification info 相当于一个 map, key 是 string 结构, 保存 write set 中提取的主键值; value 是 set 集合, 保存 gtid_executed 事务快照版本。

        *  例如事务 T1 更新数据库 d1 中的表 t1 中两行数据 id=1 和 id=2, 它对应快照版本 UUID_MGR 是 :1-100,  刚开始 certification info 为空, 所以直接提交, 之后 certification info 中快照版本直接更新为 1-101。

---


* `Certify` 冲突检测模块. 通过上面的例子可以了解 冲突检测标准 : transaction - UUID_MGR ">=" certification info  - UUID_MGR, 则冲突检测通过。


    * 单一事务 冲突检测 原理分析

        * 例如事务 T2, 更新 id=2 的行, 事务 T2 的 UUID_MGR 为 1-102, 节点中冲突检测模块中的 certification info 中的 UUID_MGR 为 1-101, 这里 T2:UUID_MGR:1-102 > UUID_MGR:1-100, 则 T2 冲突检测通过。

        * 例如事务 T3, 更新 id=1 的行, 事务 T3 的 UUID_MGR 为 1-100, 节点中冲突检测模块中的 certification info 中的 UUID_MGR 为 1-101, 很明显 T3:UUID_MGR:1-100 < UUID_MGR:1-101, 则 T3 冲突检测不通过。

    * 多事务 冲突检测 原理分析 ( 多个事务 并行写入某个 MySQL 节点, 通过了 Paxos 协议模块达成一致性共识, 进行冲突检测时需遵循三个原则  `1.` 多个事务修改同一个 id 对应的数值, 需要按照先后顺序进行冲突检测。 `2.` 多个事务同时对不同的 id 进行修改, 各自进行修改即可。 `3.` 不同的事务对同一个 id 修改, 需要按照先后顺序进行冲突检测即。)

        * 例如事务 T4 和事务 T5 同时更新 id=1 的行, 



  













* 当 Master1 上有事务要执行时, 对 Master1 是来说本地事务, 对于 Master2 、Master3 来说是远端事务; 

* Master1 上在事务在被执行后, 会把执行事务信息广播给集群各个节点(Master1、Master2 、Master3), 包括 Master1 本身, 通过 `Paxos` 模块广播给 `MGR` 集群各个节点, `半数`以上的节点同意并且达成共识, 之后共识信息进入各个节点的冲突检测 `certify` 模块, 各个节点各自进行冲突检测验证, 最终保证事务在集群中最终一致性。

* 在冲突检测通过之后, 本地事务在 Master1 直接提交即可, 否则直接回滚。远端事务在 Master2 和 Master3 分别先更新到 `relay log`, 然后应用到 `binlog`, 完成数据的同步, 否则直接放弃该事务。


