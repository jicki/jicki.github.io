# Prometheus 理论与实践



# Prometheus

{{< figure src="/img/posts/prometheus/prometheus-jg.png" >}}



## 监控设计

> 监控分类

* `业务监控` - 包含 用户访问QPS、DAU日活、访问状态、业务接口、产品转化率、业务异常数据等.

* `系统监控` - 包含 系统相关的 CPU、内存、磁盘容量、磁盘IO、TCP连接、网络接口流量、连接数等.

* `网络监控` - 包含 网络状态的一些监控 丢包率、延迟、连通率等.

* `日志监控` - 包含 日志相关的 Error 级别日志、异常的访问记录、自定义日志级别 等.

* `程序监控` - 包含 程序相关的 程序接口、程序中链路的状态等.



## Prometheus 简介


**Prometheus 是一个开源系统监控和报警的工具集合. 由 SoundCloud 公司于2012年开发. 2016年 加入 CNCF 组织. 是继 kubernetes 后 第二个加入 CNCF 组织的成员**



### Prometheus 特性


* 基于 `time-series` 时间序列模型.

  * 时间序列 - 是一些列有序的数据, 通常是相同时间间隔采集的数据.

---

* 基于 `Key/Value` 的数据模型.

---

* 基于 Block (块) 存储.

  * `time-series` 数据, 每2小时会打包成一个 Block (块).

  * 每一个 `Block` - 分为多个 `chunk` 文件. `chunk` 是存储的基本单位, `index` 和 `metadata` 是作为子集.

    * `chunk`文件 - 是用于存放采集的 `metadata` 数据文件 和 `index` 索引文件.

      * `index` 文件 - 是对 `metrics` 和 `labels` 进行索引后存储 `chunk` 文件中.

---

* 数据采集会先存储于内存中从而加快搜索和访问.

  * 当出现错误或者宕机时, `Prometheus` 有一种保护机制叫 `WAL` - 会将数据定期写入磁盘中, 以 `chunk` 表示, 系统恢复时会将丢失的数据恢复到内存中.

---

* 数据查询 根据 数据运算( + - * / ) 的表达式进行查询. 并且提供了专有的查询 `console`. 

  * 数据运算表达式: ` ( 数据A + 数据B ) / 总数据C > 固定百分比` 

---

* 基于 HTTP  `pull / push` 两种数据采集传输方式.

  * 所有数据采集都基于HTTP.

---

* 精细的数据采集 - Prometheus 的采集间隔基本都是 秒级 的采集.


---

* 开源、成品的插件多.

---



## Prometheus 组件






