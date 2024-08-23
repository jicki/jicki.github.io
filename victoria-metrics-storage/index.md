# victoriametrics


{{< figure src="/img/posts/victoriametrics/victoriametrics.png" >}}


# VictoriaMetrics 集群分析


## 简介

> VictoriaMetrics的集群主要由 `vmstorage`、`vminsert`、`vmselect` 三部分组成, 其中，


* `vmstorage` 负责提供数据存储服务; 


* `vminsert`   是数据存储 vmstorage 的代理，使用一致性 hash算法 进行写入分片；

* `vmselect`   负责数据查询，根据输入的查询条件从 `vmstorage` 中查询数据;


<br>

---
## Vmstorage


> `vmstorage` nodes don't know about each other, don't communicate with each other and don't share any data. This is [shared nothing architecture](https://link.segmentfault.com/?enc=XF1dso%2FIAgPziF%2FSLb8kYg%3D%3D.gGS%2FCkry0zYtCB6bBUbiItc0mNB3hm0W53Vx1nBHypO91e5WYJlGZ0U%2FZUHg7lST8vq%2Bcu9QMoHfudI26%2BJ%2BgQ%3D%3D). It increases cluster availability, simplifies cluster maintenance and cluster scaling.

<br>

* `vmstorage` 的多节点采用  **无共享架构**（**SN**）各节点间不共享数据，也不知道彼此的存在。

* 为保证时序数据的可用性，采用复制的方法，通过对 `vminsert` 声明 `replicationFactor` N 数量,  即每份数据同时写入N个不同的 `vmstorage` 节点，在查询时，同时查询多个节点，去重后返回给 Client。

<br>

### `vmstorage` 存储目录

<br>

```shell
drwxr-xr-x    2 root     root          4096 Nov  2 00:30 snapshots
-rw-r--r--    1 root     root             0 Nov  1 06:55 flock.lock
drwxr-xr-x    5 root     root          4096 Nov  1 06:54 cache
drwxr-xr-x    4 root     root          4096 Oct 13 10:01 data
drwxr-xr-x    6 root     root          4096 Oct 13 10:01 indexdb
drwxr-xr-x    2 root     root          4096 Oct 13 10:01 metadata
```

<br>

#### Snapshots 目录

* `snapshots`： 该目录用于创建 snapshots 备份时的存储目录.

<br>

#### Data 目录


* `data`: 该目录用于存储具体的数据.  包含 `big` 与 `small` 两个目录.

<br>

```shell
data/
├── big
│   ├── 2023_10
│   │   ├── 17936D6D965C707A
│   │   │   ├── index.bin
│   │   │   ├── metadata.json
│   │   │   ├── metaindex.bin
│   │   │   ├── timestamps.bin
│   │   │   └── values.bin
│   │   └── 17936D6D965C707B
│   │       ├── index.bin
│   │       ├── metadata.json
│   │       ├── metaindex.bin
│   │       ├── timestamps.bin
│   │       └── values.bin
│   └── snapshots
└── small
    ├── 2023_10
    │   ├── 17936D6D965C7079
    │   │   ├── index.bin
    │   │   ├── metadata.json
    │   │   ├── metaindex.bin
    │   │   ├── timestamps.bin
    │   │   └── values.bin
    │   └── parts.json
    └── snapshots

```

<br>

---

* `small` 目录，以月为单位不断生成 `partition` 目录，如上中 的 2023\_10 目录, `partition` 目录存储数据 `part` 目录( 目录名称如`17936D6D965C7079` 是生成目录时的系统纳秒时间戳的16进制表示 )

<br>

* `small` 目录和 `big` 目录都会周期性检查,  是否需要做 `part ` 的合并。VictoriaMetrics 默认每个 `10ms` 检查一次 `partition` 目录下的 `part` 是否需要做 `merge` 。如果检查出有满足 `merge` 条件的 `parts` ，则这些`parts` 合并为一个 `part` 。如果没有满足条件的`part` 进行 `merge` ，则以10ms为基进行指数休眠，最大休眠时间为10s。

<br>

* VictoriaMetrics 在写数据时，先写入 `small` 目录下的相应 `partition` 目录下面的，`small` 目录下的每个`partition` 最多 256个 `part` 。VictoriaMetrics 在 Compaction 时，默认一次最多合并 15个  `part` ，且 `small` 目录下的 `part` 最多包含 `1000W` 行数据，即 `1000W` 个数据点。因此，当一次待合并的 `parts` 中包含的行数超过 `1000W` 行时，其合并的输出路径为 `big` 目录下的同名 `partition` 目录下。

<br>

因此，`big` 目录下的数据由 `small` 目录下的数据在后台 `compaction` 时合并生成的。 那么为什么要分成big目录和small目录呢？

<br>

这个主要是从磁盘空间占用上来考虑的。时序数据经常读取最近写入的数据，较少读历史数据。而且，时序数据的数据量比较大，通常会使用压缩算法进行压缩。

<br>

数据进行压缩后，读取时需要解压，采用不同级别的压缩压缩算法其解压速度不一样，通常压缩级别越高，其解压速度越慢。VictoriaMetrics 在时序压缩的基础上，又采用了`ZSTD` 这个通用压缩算法进一步压缩了数据，以提高压缩率。 `small` 目录中的 `part` 数据，采用的是低级别的ZSTD，而 `big` 目录下的数据，采用的是高级别的ZSTD。

<br>

因此，VictoriaMetrics 分成 `small` 目录和 `big` 目录,  主要是兼顾近期数据的读取和历史数据的压缩率。

<br>

---

#### indexdb 索引目录

<br>


```shell
indexdb/
├── 178D9F81E688A5B6   <<<--- table Name
│   └── parts.json
├── 178D9F81E688A5B7   <<<--- table Name
│   ├── 178D9F81F2BD163D  <<<--- part Name
│   │   ├── index.bin
│   │   ├── items.bin
│   │   ├── lens.bin
│   │   ├── metadata.json
│   │   └── metaindex.bin
│   ├── 1790B179E9F13004  <<<--- part Name 
│   │   ├── index.bin
│   │   ├── items.bin
│   │   ├── lens.bin
│   │   ├── metadata.json
│   │   └── metaindex.bin
│   └── parts.json
└── snapshots
```

<br>

* `indexdb` 目录下由多个 `table` 目录，每个 `table` 目录代表一个完整的自治的索引，每个 `table` 目录下，又有多个不同的 `part` 目录.  每个 `part` 下包含一个 `metadata.json` 记录了 `ItemsCount` 、`BlocksCount`、`FirstItem`、`LastItem` 信息.

<br>

* VictoriaMetrics 会根据用户设置的数据保留周期 `retention`  来定期滚动索引目录，当前一个索引目录的保留时间到了，就会切换一个新的目录，重新生成索引。

<br>

---

#### 文件格式

<br>

> {"metric":{"\_\_name\_\_":"bid","ticker":"MSFT"},"values":\[1.67],"timestamps":\[1583865146520]}

<br>

* `metadata.json` :  源数据属性文件, 记录了 `RowsCount`、`BlocksCount`、`MinTimestamp` 、 `MaxTimestamp` 、`MinDedupInterval`  属性的数据.


* `timestamps.bin` :   时间戳列数据文件

* `values.bin`:   value 列数据文件

<br>


VictoriaMetrics 在接受到写入请求时，会对请求中包含的时序数据做转换处理,  首先先根据包含 metric和 labels 的MetricName 生成一个唯一标识 TSID，然后 metric + labels + TSID 作为索引 index， TSID + timestamp + value 作为数据data，最后索引 index 和数据 data 分别进行存储和检索。

---

## vminsert

> vminsert 是 VictoriaMetrics 的写入组件，用于将外部数据写入到 VictoriaMetrics 中。

<br>

### 数据分片

<br>

* 集群设置中的每个 `vmstorage` 都表示为一个 **`shard`**。分片仅保存一小部分数据。在写入期间，`vminsert` 组件一致地将数据**分片 **写入到可用的 `vmstorage` 节点上，确保每个唯一的时间序列最终位于 同一分片/vmstorage 上

<br>

例如，2 分片集群中的时间系列 `foo` 和 `bar` 的写入最终将位于单独的 `vmstorage` 上，无论有多少个  `vminsert` 进行写入。这种方法提供了更好的数据局部性的好处，提高了搜索速度、压缩比、缓存的内存使用、页面缓存等。

<br>

---

## vmselect

> vmselect 是 VictoriaMetrics 的查询组件，用于从 VictoriaMetrics 中检索数据。

<br>

* `vmselect`  向每个 `vmstorage` 节点发送请求, 查询时序数据,  数据返回后,  经过合并去重, 返回给 `client`. 

