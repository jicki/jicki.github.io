---
layout: post
title: ETCD 原理剖析
categories: etcd
description: ETCD 原理剖析
keywords: etcd
feature-img: "assets/img/pexels/desk-top.jpeg"
catalog:    true
tags:
    - etcd
---

# ETCD

![etcd][1]


* etcd 官方地址 `https://etcd.io/`

* etcd 是 CoreOS 公司, 最初用于解决集群管理系统中 OS 升级时的分布式并发控制、配置文件的存储于分发等问题。

  * etcd 被设计为提供高可用、强一致性的小型 `key/value` 数据存储服务。

* etcd 目前已经隶属于`CNCF (Cloud Native Computing Foundation)` 基金会, 被包含 AWS 、Google、Microsoft、Alibaba 等大型互联网公司广泛使用。

* etcd 最早在 2013年6月份 于 `github` 中开源。

* etcd 于 2014年6月份 正式被 `kubernetes` 使用, 用于存储 `kubernetes` 集群中的元数据。

* etcd 于 2015年2月份 发布 2.0 版本, 正式支持分布式协议 Raft, 并支持 1000/s 的并发 `writes` 。 

* etcd 于 2017年1月份 发布 3.1 版本, 全面优化了 `etcd` , 新的`API`、重写了强一致性(read)读的方法,并提供了 gRPC Proxy 接口, 以及大量 `gc` 的优化。

* etcd 于 2018年11月 加入 CNCF 基金会。

* etcd 于 2019年 发布 etcd v3.4 该版本又 `Google、AWS、Alibaba` 等大公司联合打造的一个版本。将进一步优化稳定性以及性能。


## Etcd 架构与内部机制

* etcd 是一个 `分布式`、`强一致性可靠`、`key/value` 存储系统。 它主要用于存储分布式系统中的 关键数据。

* etcd 集群通常由 奇数(最低3)个 etcd 组成。
  * 集群中多个 etcd 通过 `Raft consensus algorithm` 算法进行协同, 多个 etcd 会通过 `Raft` 算法选举出一个 `Leader`, 由 `Leader` 节点进行数据同步, 以及数据分发。

  * 当 `Leader` 出现故障时, 集群中的 etcd 会投票选举出另一个 `Leader` , 并重新进行数据同步以及数据分发。

  * 客户端从任何一个 etcd 都可以进行 `读/写` 操作。

  * 在 etcd 集群中 `故障容忍数 = ( n + 1 ) / 2`, 意思是 集群中可以容忍故障的数量, 如&emsp; `(3 + 1) / 2 = 1` 可以容忍的故障数为1台。 

  * 在 etcd 集群中 任意两个 `quorum` 一定会存在差异, 只要有任意一个`quorum`存活, 这其中肯定包含某个 etcd 节点 它包含 etcd 集群中最新的数据, 基于这种假设, `Raft` 强一致性算法会将最新数据同步到其他 etcd 节点中。 
 
    * `quorum`:&emsp; 在`Raft` 中超过一半以上的人数就是法定人数。

![etcd-1][2]



* etcd 提供了如下 `API`

  * `Put(key, value)` &emsp;增加
  * `Delete(key)` &emsp;删除
  * `Get(key)` / `Get(keyFrom, keyEnd)` &emsp;查询
  * `Watch(key)` / `Watch(key前缀)` &emsp;事件监听, 监听key的变化
  * `Transactions(if / then / else ops).Commit()` 事务操作, 指定某些条件, 为`true`时执行其他操作。 
  * `Leasesi Grant / Revoke / KeepAlive`


* etcd 的数据版本号机制

  * `term`:&emsp;全局单调递增 (64bits), `term` 表示整个集群中 `leader` 的任期, 当集群中发生 `leader` 切换, 如: `leader`节点故障、`leader`网络故障、整个集群重启都会发生 `leader` 切换, 这个时候 `term = term + 1`。 

  * `revision`:&emsp;全局单调递增 (64bits), `revision` 表示在整个集群中数据变更版本号, 当集群中数据发生变更, 包括 `创建`、`修改`、`删除` 的时候, `revision = revision + 1`。

  * `Key/Vaule`:
    * `Create_revision`:&emsp; 表示在当前 `Key/Value` 中在整个集群数据中创建时的版本号。每个 `Key/Value` 都有一个 `Create_revision`。

    * `mod_revision`:&emsp; 表示当前 `Key/Value` 最后修改 `Value` 的版本号。   

    * `version`: &emsp; 表示当前 `Key/Value`  被修改了多少次。


![etcd-3][3]



### etcd mvcc && watch

* `mvcc`: &emsp; 全称 `Multi-Version Concurrency Control` 即多版本并发控制。
  * `mvcc`: &emsp; 是一种并发控制的方法, 一般在数据库管理系统中, 实现对数据库的并发访问。

* 在 `etcd` 中, 支持对同一个 `Key` 发起多次数据修改。因为已经知道每次数据修改都对应一个版本号`(mod_revision)`, 多次修改就意味着一个 `key` 中存在多个版本, 在查询数据的时候可以通过不指定版本号查询`Get key`, 这时 `etcd` 会返回该数据的最新版本。当我们指定一个版本号查询数据后`Get --rev=1 Key`, 可以获取到一个 `Key` 的历史版本。


* 在 `watch` 的时候指定数据的版本, 创建一个 `watcher`, 并通过这个 `watcher` 提供的一个数据管道, 能够获取到指定的 `revision` 之后所有的数据变更。如果指定的 `revision` 是一个旧版本, 可以立即拿到从旧版本到当前版本所有的数据更新。并且, `watch` 的机制会保证 `etcd` 中, 该 `Key` 的数据发生后续的修改后, 依然可以从这个数据管道中拿到数据增量的更新。



* 在 `etcd` 中 所有的数据都存储在一个 `b + tree` 中。

  * `b + tree`&emsp;是保存在磁盘中, 并通过 `mmap` 的方式映射到内存用来查询操作。
    * `mmap`&emsp; 是一种内存映射文件的方法, 即将一个文件或者其对象映射到进程的内存地址空间, 实现文件磁盘地址和进程虚拟地址空间中一段虚拟地址的一一对映关系。

  * `b + tree`&emsp; 维护着 `revision` 到 `value` 的映射关系。也就是说当指定 `revision` 查询数据的时候, 就可以通过该 `b + tree` 直接返回数据。当我们通过 `watch` 来订阅数据的时候, 也可以通过这个 `b + tree` 维护的 `revision` 到 `value` 映射关系, 从而通过指定的 `revision` 开始遍历这个`b + tree` ,拿到所有的数据更新。


* 在 `etcd` 内部还维护着另外一个 `b + tree` 。它管理着 `key` 到 `revision` 的映射关系。当需要查询 `Key` 对应数据的时候, 会通过 `etcd` 内部的 `b + tree`, 将 `key` 翻译成 `revision`。再通过磁盘中的 `b + tree` 的 `revision` 获取到对应的 `value`。



* 在 `etcd` 中 一个数据是存在多个版本的。

* 在 `etcd` 持续运行过程中会不断的发生修改, 意味着 `etcd` 中内存及磁盘的数据都会持续增长。这对资源有限的场景来说是无法接受的。因此在 `etcd` 中会周期性的运行一个 `Compaction` 的机制来清理历史数据。 对于一个 `Key` 的历史版本数据, 可以选择清理掉。 



### etcd mini-transactions

* etcd 的 `transaction` 机制比较简单, 基本可以理解为一段 `if else` 程序, 在 `if` 中可以提供多个操作。

```
Txn.if(
  Compare(Value(key1), ">", "bar"),
  Compare(Version(key1), "=", 2),
  ...
).Then(
  Put(key2, valueX),
  Delete(key3)
).Else(
  Put(key2, valueY)
).Commit()

```

* 在 `etcd` 内部会保证整个事务操作的原子性。也就是说 `If` 操作所有的比较条件, 其看到的视图, 一定是一致的。同时它能够确保在争执条件中, 多个操作的原子性不会出现 `etc` 仅执行了一半的情况。

* 通过 `etcd` 提供的事务操作, 我们可以在多个竞争中去保证数据读写的一致性, 比如 `Kubernetes` , 它正是利用了 `etcd` 的事务机制, 来实现多个 `Kubernetes API server` 对同样一个数据修改的一致性。

![etcd-3][4]




### etcd lease

* `lease` 是分布式系统中一个常见的概念, 用于代表一个租约。通常情况下, 在分布式系统中需要去检测一个节点是否存活的时候, 就需要租约机制。


* `etcd` 通过 `CreateLease(时间)` 来创建一个租约。如: `lease = CreateLease(10s)` 创建一个 10s 过期的一个租约。

  * 通过 Put(key1, value1, lease) 可以将之前创建的 租约绑定到 `key1` 中(同一个租约可以绑定多个key)。当租约过期时, `etcd`会自动清理 `key1` 对应的 `value1` 值。

  * `KeepAlive` 方法:&emsp; 可以续约租期。
    * 如说需要检测分布式系统中一个进程是否存活, 那么就会在这个分布式进程中去访问 `etcd` 并且创建一个租约, 同时在该进程中去调用 `KeepAlive` 的方法, 与 `etcd` 保持一个租约不断的续约。当进程挂掉了, 租约在进程挂掉的一段时间就会被 `etcd` 自动清理掉。所以可以通过这个机制来判定节点是否存活。



















  [1]: http://jicki.me/img/posts/etcd/etcd.png
  [2]: http://jicki.me/img/posts/etcd/etcd-1.png
  [3]: http://jicki.me/img/posts/etcd/etcd-2.png
  [4]: http://jicki.me/img/posts/etcd/etcd-3.png
