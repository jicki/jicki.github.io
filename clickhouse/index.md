# CliCkHouse 列式数据库-OLAP分析引擎


{{< figure src="/img/posts/clickhouse/clickhouse-logo.png" >}}


# ClickHouse

## ClickHouse 介绍

> ClickHouse 是俄罗斯的 `Yandex` 于 2016 年开源的列式存储数据库`（DBMS）`，使用 `C++` 语言编写，主要用于在线分析处理查询`（OLAP）`，能够使用 `SQL` 查询实时生成分析数据报告。



## ClickHouse 特性

### 数据存储

> 列式存储

* 列式存储的格式如下

  * 分别为 Id, 名字, 年龄

|  |  |  |  |  |  |  |  |  |
| :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: | :----: |
| 1 | 2 | 3 | 张三 | 李四 | 王五 | 18 | 20 | 30 |


> 列式存储的优点

* 对于列的聚合，计数，求和等统计操作原因优于行式存储。

* 由于某一列的数据类型都是相同的，针对于数据存储更容易进行数据压缩，每一列选择更优的数据压缩算法，大大提高了数据的压缩比重。

* 由于数据压缩比更好，一方面节省了磁盘空间，另一方面对于 `Cache` 也有了更大的发挥空间。


### DBMS 操作

* 类 `SQL` 语法, `DDL` 创表语句等,  `DML` 查询语句等. 

* 其他配套函数, 如 `count`、`sum` 等.  

* 用户、权限管理.

* 数据备份、数据恢复操作.


### 存储引擎


* `ClickHouse` 和 `MySQL` 类似，把表级的存储引擎插件化，根据表的不同需求可以设定不同的存储引擎。目前包括如下四大类 ( 20 多种引擎 )

  * 合并树
  
  * 日志

  * 接口

  * 其他


### LSM Tree


* ClickHouse 采用类 `LSM Tree` 的结构，数据写入后定期在后台 `Compaction`。

* 通过类 `LSM tree` 的结构，ClickHouse 在数据导入时全部是顺序 append 写，写入后数据段不可更改，在后台 `compaction` 时也是多个段 `merge sort` 后顺序写回磁盘。

* 顺序写的特性，充分利用了磁盘的吞吐能力，即便在 HDD 上也有着优异的写入性能。


官方公开 benchmark 测试显示能够达到 `50MB-200MB/s` 的写入吞吐能力，按照每行 `100Byte` 估算，大约相当于 `50W-200W 条/s` 的写入速度。



### 数据分区与线程级并行处理


* ClickHouse 将数据划分为多个 `partition`，每个 `partition` 再进一步划分为多个 `index granularity`(索引粒度)，通过多个 CPU 核心分别处理其中的一部分来实现并行数据处理。

* 单条 Query 就能利用整机所有 CPU。极致的并行处理能力，极大的降低了查询延时。

* ClickHouse 即使对于大量数据的查询也能够化整为零平行处理。


但是有一个弊端就是对于单条查询使用多 cpu (单条查询就占用了所有的 CPU 资源)，就不利于同时并发多条查询。所以对于高 `qps`(每秒查询次数) 的查询业务，ClickHouse 并不是强项。



## ClickHouse 数据类型


### 整型

> 有符号

| 类型 | 取值范围 |
| :----: | :----: | 
| Int8 | -128 ~ 127 |
| Int16 | -32768 ~ 32767 |
| Int32 | -2147483648 ~ 2147483647 |
| Int64 | -9223372036854775808 ~ 9223372036854775807 |


> 无符号

| 类型 | 取值范围 |
| :----: | :----: | 
| Uint8 | 0 ~ 255 |
| Uint16 | 0 ~ 65535 |
| Uint32 | 0 ~ 4294967295 |
| Uint64 | 0 ~ 18446744073709551615 |


---


### 浮点型


| 类型 | 取值范围 |
| :----: | :----: | 
| Float32 | float |
| Float64 | double |


---


### 布尔类型


* clickHouse 并没有单独的类型用于存储 布尔值类型(true or false), 可使用 `uint8` , 取值 `0` 和 `1` 替代.



### Decimal 型


* 有符号浮点数, 可在加、减和乘法运算过程中保持精度。对于除法，最低有效数字会被丢弃（不舍入）。


| 类型 | 有效位数 |
| :----: | :----: | 
| Decimal32(s) | 1~9 |
| Decimal64(s) | 1~18 |
| Decimal128(s) | 1~38 |


`s`: 表示小数后保留位数.


### 字符串

> string

* 字符串可以任意长度的。它可以包含任意的字节集，包含空字节。


> FixedString(N)

* 固定长度 N 的字符串，N 必须是严格的正自然数。当服务端读取长度小于 N 的字符串时候，通过在字符串末尾添加空字节来达到 N 字节长度。 当服务端读取长度大于 N 的
字符串时候，将返回错误消息。



### 枚举类型

| 类型 | 对应关系 |
| :----: | :----: | 
| Enum8 | 'String'= Int8 |
| Enum16 | 'String'= Int16 |


* 事例
 
  * 这个 `x` 列只能存储类型定义中列出的值：'hello'或'world'
  * 如果保存任何其他值，`ClickHouse` 抛出异常

```bash
CREATE TABLE t_enum
(
  x Enum8('hello' = 1, 'world' = 2)
)
ENGINE = TinyLog;
```


### 时间类型


| 类型 | 接受字符串 |
| :----: | :----: | 
| Date | 2022-02-18 |
| Datetime | 2022-02-18 15:10:23 |
| Datetime64 | 2022-02-18 15:10:23.60 |




### 数组类型


* Array(T): 由 T 类型元素组成的数组。

  * 如下两种创建数组的方式. 直接使用 `[]` 或 `array()` 函数.

```bash
SELECT array(1, 2) AS x, toTypeName(x);

or

SELECT [1, 2] AS x, toTypeName(x);

```



## 表引擎


### 表引擎特点

> 表引擎决定了如何存储表的数据。

1. 数据的存储方式和位置，写到哪里以及从哪里读取数据。

2. 支持哪些查询以及如何支持。

3. 并发数据访问。

4. 索引的使用（如果存在）。

5. 是否可以执行多线程请求。

6. 数据复制参数。

---

  `表引擎的使用方式就是必须显式在创建表时定义该表使用的引擎，以及引擎使用的相关参数。`

  `特别注意: 引擎的名称大小写敏感, 引擎名称 均为大驼峰命名`



### TinyLog
 
> 日志类

* 以列文件的形式保存在磁盘上，不支持索引，没有并发控制。一般保存少量数据的小表。一般用于平时测试用。



### Memory

> 其他类

* 内存引擎，数据以未压缩的原始形式直接保存在内存当中，简单查询性能非常高，服务器重启数据就会消失。读写操作不会相互阻塞，不支持索引。 一般也用做测试。



### MergeTree

> 合并数类


