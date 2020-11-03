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





### PrometheusRules 告警规则


**Prometheus Operator模式中，告警规则(Rules) 是一个通过 Kubernetes API 声明式创建的一个资源清单**


* `PrometheusRules` 支持 两种 类型的规则

  * 记录规则 `record` - 记录规则主要是为了简写报警规则和提高规则复用以及提高性能. 记录规则是顺序执行, 后面的规则可以引用前面定义好的规则.

    * 记录规则 `record` - 允许您预计算经常需要或计算开销大的表达式，并将其结果保存为一组新的时间序列。查询预计算的结果通常比每次需要时执行原始表达式快得多。这对于仪表板 `grafana` 特别有用，因为仪表板每次刷新时都需要重复查询同一表达式。

  * 报警规则 `alert`  - 报警规则是真正去判定是否需要报警的规则, 报警规则中可以使用记录规则.



---

> <center>PrometheusRules 资源清单</center>


* `Prometheus-Record`: 

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: example
    role: record-rules
  name: prometheus-example-record
spec:
  groups:
  - name: example.record
    rules:
    - expr: sum(min by(cluster, node) (kube_pod_info{node!=""}))
      record: :kube_pod_info_node_count: 
      labels: 
        message: '统计 Node 总数'
```

---

* `Prometheus-Alert`

```
apiVersion: monitoring.coreos.com/v1
kind: PrometheusRule
metadata:
  labels:
    prometheus: example
    role: alert-rules
  name: prometheus-example-alert
spec:
  groups:
  - name: example.rules
    rules:
    - alert: ExampleAlert
      expr: vector(1)
      for: 10m
      labels:
        severity: critical
      annotations:
        description: 这是一个告警例子
```
---


* `groups` -  告警规则的组, 每个资源清单都包含一个 `groups` 一个 `rules` 以及 多个 `alert` or `record` 规则.

  * `name` -  告警规则的名称.

  * `rules` - 具体的告警规则.

    * `expr` -  待评估项 `PromQL` 表达式,  每个评估周期都会在当前时间进行评估，并将结果记录为一组新的时间序列，其中度量标准名称由 记录 给出。

    * `alert` - 告警规则的名称

    * `record` - 记录规则中使用, 要输出的时间序列的名称. 必须是有效的度量标准名称.

    * `for` - 使 Prometheus 在第一次遇到新的表达式输出向量元素和将此警告作为此元素的触发计数之间等待一段时间.

    * `labels` - 指定要附加到警报的一组附加标签. 任何现有的冲突标签都将被覆盖. 支持模板语法.

    * `annotations` - 额外的 注释/信息 标签，可用于存储更长的附加信息，例如警报描述或链接. 支持模板语法.

---


* FAQ - `PrometheusRules` 资源清单中的变量

  * `PrometheusRules` 资源清单中的变量与 `Helm` 中的变量都使用 temlpate 模板中的定义方式 `{{ }}` Helm 使用时会出现 变量未定义的情况.

    * 解决方式:  将 {{ }} 定义为 {{`{{`}} $value {{`}}`}}


---


### PromQL 语法


**Prometheus 提供一个函数式的表达式语言 PromQL (Prometheus Query Language), 可以使用户实时地查找和聚合时间序列数据. 表达式计算结果可以在图表中展示，也可以在 Prometheus 表达式浏览器中以表格形式展示, 或者作为数据源, 以HTTP API的方式提供给外部系统使用.**

---

* Prometheus 的 PromQL (表达式语言) 中, 任何表达式或者子表达式都可以归为四种类型: 

  * `string` (字符串) --- 一个当前没有被使用的简单字符串.

  * `scalar` (标量) --- 一个简单的浮点值.

  * `instant vector` (瞬时向量) --- 它是指在同一时刻， 抓取的所有度量指标数据. 这些度量指标数据的key都是相同的, 也即相同的时间戳.

  * `range vector` (范围向量) --- 它是指在任何一个时间范围内, 抓取的所有度量指标数据.

---


* `PromQL` 字面量

---

> 字符串 

* 字符串可以用单引号，双引号或反引号指定为文字.

* PromQL 遵循与Go相同的转义规则. 在单引号，双引号中，反斜杠成为了转义字符. 但是反引号 内不处理转义字符. PromQL 不会丢弃反引号中的换行符.

---

> 浮点数标量

* 标量浮点值可以直接写成形式 `[-](digits)[.(digits)]` 既 `-2.43`.


---

> 瞬时向量

* 瞬时向量选择器允许在给定时间戳（即时）为每个选择一组时间序列和单个样本值：在最简单的形式中，仅指定度量名称. 这会生成包含具有此度量标准名称的所有时间序列的元素的即时向量

  * 通过在度量指标后面增加{}一组标签可以进一步地过滤这些时间序列数据.

    * 采用不匹配的标签值也是可以的，或者用正则表达式不匹配标签. 标签匹配如下:

      * `=` - 精确地匹配标签给定的值

      * `!=` - 不等于给定的标签值

      * `=~` - 正则表达匹配给定的标签值

      * `!~` - 给定的标签值不符合正则表达式

      * 正则表达式使用 RE2 语法:  `https://github.com/google/re2/wiki/Syntax` 



* 例:

```
# 1. 选择所有时间序列度量名称为 node_cpu_seconds_total 的样本数据
node_cpu_seconds_total

```

```
# 2. 通过 {} 进一步过滤时间序列的数据
node_cpu_seconds_total{job="prometheus-node-exporter-metrics", cpu="0"}

```

```
# 3. 通过 正则表达式 匹配 mode 多个值
node_cpu_seconds_total{job="prometheus-node-exporter-metrics", mode=~"idle|system|user"}

```

---

> 范围向量

* 范围向量文字像即时向量文字一样工作，除了它们从当前时刻选择一系列样本. 在语法上，范围持续时间附加在向量选择器末尾的方括号（[]）中，以指定应为每个结果范围向量元素提取多长时间值。

  * 持续时间指定为数字, 后面是以下单位之一
  
    * `s` - seconds
    * `m` - minutes
    * `h` - hours
    * `d` - days
    * `w` - weeks
    * `y` - years


* 例:


```
# 1. 通过 [] 选择时间内的指定度量单位的时间序列记录, 如下为过去5分钟内的记录
node_cpu_seconds_total{job="prometheus-node-exporter-metrics"}[5m]

```


---

> 偏移修饰符 offset 

* 这个 `offset` 偏移修饰符允许在查询中改变单个瞬时向量和范围向量中的时间偏移. 简单来说就是 - 假设当前时间为 `10:00:00` 使用偏移修饰符 offset 1w , 既查询 1个周前 `10:00:00` 的时间序列记录.

  * `offset` 必须直接包含在选择器后面.


* 例: 

```
# 1. 通过 offset 使记录返回过去相对于当前查询评估时间的过去5分钟的时间序列记录

node_cpu_seconds_total{job="prometheus-node-exporter-metrics"} offset 5m

```


```
# 2. 通过时间偏移量与范围向量组合查询时间序列记录

node_cpu_seconds_total{job="prometheus-node-exporter-metrics"}[5m] offset 1h


```

---

> 聚合操作符

* 聚合操作符 用于聚合单个即时向量的元素，从而生成具有聚合值的较少元素的新向量.

  * `sum` - 在维度上求和
  * `max` - 在维度上求最大值
  * `min` - 在维度上求最小值
  * `avg` - 在维度上求平均值
  * `stddev` - 求标准差
  * `stdvar` - 求方差
  * `count` - 统计向量元素的个数
  * `count_values` - 统计相同数据值的元素数量
  * `bottomk` - 样本值第k个最小值
  * `topk` - 样本值第k个最大值
  * `quantile` - 统计分位数

---

> sum

* `sum` 在维度上求和

  * `by` 的含义表示将结果根据 instance 来进行区分. 跟 mysql 语句中 group by 差不多.

* 例:


```
# 1. 查看 1分钟内 非 idle 的 cpu 使用率,并进行求和, 使用 instance 来进行区分
sum(rate(node_cpu_seconds_total{mode!="idle"}[1m])) by (instance)

```

---



## Prometheus 告警


* 告警能力在 `Prometheus` 的架构中被划分成两个独立的部分 ( PrometheusRules 和 Alertmanager )。

  * 通过在 `Prometheus` 中定义 `AlertRule`（告警规则）, Prometheus 会周期性的对告警规则进行计算, 如果满足告警触发条件就会向 `Alertmanager` 发送告警信息。


* 在 `Prometheus` 中一条告警规则主要由以下部分组成

  * 告警名称: - 用户需要为告警规则命名, 当然对于命名而言, 需要能够直接表达出该告警的主要内容。

  * 告警规则: - 告警规则实际上主要由 `PromQL` 进行定义, 其实际意义是当表达式（PromQL）查询结果持续多长时间（During）后出发告警。


* 在 `Prometheus` 中还可以通过 `Group（告警组）` 对一组相关的告警进行统一定义。当然这些定义都是通过YAML文件来统一管理的。


* `Alertmanager` - 作为一个独立的组件, 负责接收并处理来自 `Prometheus` 的告警信息。 

* `Alertmanager` - 可以对这些告警信息进行进一步的处理, 比如当接收到大量重复告警时能够消除重复的告警信息, 同时对告警信息进行分组并且路由到正确的通知方,`Prometheus` 内置了对邮件, `Slack` 等多种通知方式的支持, 同时还支持与 `Webhook` 的集成, 以支持更多定制化的场景。目前 `Alertmanager` 还不支持钉钉, 那用户完全可以通过 `Webhook` 与钉钉机器人进行集成, 从而通过钉钉接收告警信息。同时 `AlertManager` 还提供了 静默 和 告警抑制机制 来对告警通知行为进行优化。


---

### Alertmanager 特性

* AlertManager 告警分组

  * 告警分组机制 - 可以将详细的告警信息合并成一个通知。在某些情况下, 比如由于系统宕机导致大量的告警被同时触发, 在这种情况下分组机制可以将这些被触发的告警合并为一个告警通知, 避免一次性接受大量的告警通知, 而无法对问题进行快速定位。

  * 当集群中有数百个正在运行的服务实例, 并且为每一个实例设置了告警规则。假如此时发生了网络故障, 可能导致大量的服务实例无法连接到数据库, 结果就会有数百个告警被发送到 `Alertmanager`。而作为用户, 可能只希望能够在一个通知中中就能查看哪些服务实例收到影响。这时可以按照服务所在集群或者告警名称对告警进行分组, 而将这些告警内聚在一起成为一个通知。

  * 告警分组, 告警时间, 以及告警的接受方式可以通过 `Alertmanager` 的配置文件进行配置。


* AlertManager 抑制
  
  * 抑制 - 是指当某一告警发出后, 可以停止重复发送由此告警引发的其它告警的机制。

  * 当集群不可访问时触发了一次告警, 通过配置 `Alertmanager` 可以忽略与该集群有关的其它所有告警。这样可以避免接收到大量与实际问题无关的告警通知。

  * 抑制 通过 `Alertmanager` 的配置文件进行设置。


* AlertManager 静默

  * 静默 - AlertManager 提供了一个简单的机制可以快速根据标签对告警进行静默处理。如果接收到的告警符合静默的配置, `Alertmanager` 则不会发送告警通知。

  * 静默 设置需要在 `Alertmanager` 的 `Web` 页面上进行设置。
