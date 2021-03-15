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

---

## InfluxDB 概念


---

**假设定义一个数据库 TestDB 定义一个数据表 weather 字段如下**

* 时间 time

* 温度 temperature

* 湿度 humidity

* 地区 area

* 海拔 altitude

---

>  InfluxDB 与传统数据库概念的比较


|传统数据库中的概念 | InfluxDB 名词 | 
|      -          |      -        |
|   数据库         |  database     |
|  数据库的表      |  measurement  |
| 表里面的一行数据　|  points       |

---

> InfluxDB 中独有的一些念概

* Points 由时间戳（time）、数据（field）、标签（tags）组成。


| 传统数据库中的属性 | Points 属性 |
|        -         |      -      |
| 每个数据记录时间, 是数据库中的主索引(会自动生成) | time |
| 各种记录值（没有索引的属性）也就是记录的值: 温度, 湿度 | fields |
| 各种有索引的属性: 地区, 海拔 | tags |




## InfluxDB 高可用方案

> 目前 InfluxDB 包含两种常用的 高可用方案. 

* 官方的高可用方案 `influxdb-relay`

* 360开源的高可用方案 `InfluxDB Proxy`


---


> influxdb-relay 官方开源了一段时间后, 不知道因为什么原因后来就不维护了, 目前项目处于停滞更新状态.



* influxdb-relay 架构图如下:


```bash
        ┌─────────────────┐                 
        │writes & queries │                 
        └─────────────────┘                 
                 │                          
                 ▼                          
         ┌───────────────┐                  
         │               │                  
┌────────│ Load Balancer │─────────┐        
│        │               │         │        
│        └──────┬─┬──────┘         │        
│               │ │                │        
│               │ │                │        
│        ┌──────┘ └────────┐       │        
│        │ ┌─────────────┐ │       │┌──────┐
│        │ │/write or UDP│ │       ││/query│
│        ▼ └─────────────┘ ▼       │└──────┘
│  ┌──────────┐      ┌──────────┐  │        
│  │ InfluxDB │      │ InfluxDB │  │        
│  │ Relay    │      │ Relay    │  │        
│  └──┬────┬──┘      └────┬──┬──┘  │        
│     │    |              |  │     │        
│     |  ┌─┼──────────────┘  |     │        
│     │  │ └──────────────┐  │     │        
│     ▼ ▼               ▼  ▼       │        
│  ┌──────────┐      ┌──────────┐  │        
│  │          │      │          │  │        
└─▶│ InfluxDB │      │ InfluxDB │◀─┘        
   │          │      │          │           
   └──────────┘      └──────────┘           

```



