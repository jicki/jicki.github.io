# InfluxDB 高可用解决方案


# {{< figure src="/img/posts/influxdb/influxdb-logo.png" >}}


# InfluxDB

> InfluxDB 是一个开源的分布式时序、时间和指标数据库, 使用go语言编写, 无需外部依赖。

* InfluxDB 特性

  * `时序性（Time Series）` - 与时间相关的函数的灵活使用（诸如最大、最小、求和等）。

  * `度量（Metrics）` - 对实时大量数据进行计算。

  * `事件（Event）` - 支持任意的事件数据, 换句话说, 任意事件的数据都可以做操作。

---

* InfluxDB 特点

  * Schemaless(无结构) 可以是任意数量的列。

  * 支持 min, max, sum, count, mean, median 一系列函数, 方便统计。

  * Native HTTP API, 内置http支持, 使用http读写。

  * Powerful Query Language 类Sql 语法。

  * Built-in Explorer 自带 WEB-UI 管理工具。

