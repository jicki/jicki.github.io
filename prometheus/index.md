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



### Prometheus 概念理论 



> <center>Prometheus metrics</center>


* Prometheus 对于采集的数据统称为 `metrics` 数据.

  * `metrics` 是一种对采集数据的总称, 它不代表某一种具体的数据格式, 是一种对于度量计算单位的抽象.


---

> metrics 数据类型

---

> Gauges 


* Gauges 最简单的度量指标, 只有一个最简单的返回值, 或者瞬时状态. 

  * 比如: 磁盘的容量、内存的使用量, 就使用 Gauges 的 metrics 格式来度量.  因为 磁盘容量、内存使用量 都是随时会发生变化, 而且这种变化是无规律的变化. 这种类型的数据就是Gauges类型的代表.


---


> Counters


* Counter 就是计数器, 从数据 0 开始累加计算, 正常状况下永远是累加或者不变 不会减少. 

  * 比如: 用户访问量, 就是一直不断的累加.

---


> Histograms

* Histogram 统计数据的分布情况. 最大值、最小值、平均值、中位数、的百分比数值 . 

  * 比如: 用户响应时间, 每个用户到达服务端的网络请求时间都有差异. 这个时候就可以使用 Histogram 类型分别统计出 影响时间中 快慢的占比. 如 小于 0.5s 占 80% , 大于 1s 占 10%  大于 5s 占 7% 大于10s 的占 3%.  




---



> <center>Prometheus K/V 数据形式</center>

* Prometheus 中采集的 metrics 数据都是以 key/value 格式形式存储的.

  * 比如: 使用 curl http://localhost:9100/metrics  可以查看到数据采集都是 key/value 格式. 

    * `process_max_fds 65535` 前面是 key 空格后是 value .



---



> <center>Prometheus exporter</center> 


* 为 Prometheus 提供监控数据源的应用都被统称为 exporter. 如:

  * `blackbox_exporter` -- 黑盒数据采集插件, 采集一些最基本的数据, 如: 机器连通性、服务是否running、端口是否打开 等.

  * `node_exporter` -- Linux 系统相关的数据采集插件. 


* Prometheus Server 提供PromQL查询语言能力、负责数据的采集和存储等主要功能.

* 数据的采集主要通过周期性的从 exporter 所暴露出来的HTTP服务地址 ( /metrics ) 来获取监控数据。

* exporter 在实际运行的时候根据其支持的方式也会分为:

  * 独立运行的 exporter 应用, 通过HTTP服务地址提供相应的监控数据. 如: `node_exporter`

  * 内置在监控目标中, 通过HTTP服务地址提供相应的监控数据. 如: `kubernetes`


---

> <center>Prometheus pushgateway</center>


* pushgateway 本身是一个 HTTP 服务器. 用户通过将采集的数据推送到 pushgateway 中, 然后再由 Prometheus Pull 下来.


---






