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


### influxdb-relay


> influxdb-relay 官方开源了一段时间后, 不知道因为什么原因后来就不维护了, 目前项目处于停滞更新状态, 大概官方要推自己的 企业集群版本. 


### influxdb-relay 架构

* influxdb-relay 架构图如下:

  * 该架构非常简单, 由一个负载均衡器, 两个或多个 `InfluxDB Relay` 进程以及两个或多个 `InfluxDB` 进程组成。负载平衡器应将UDP流量和HTTP POST请求/write指向两个中继的路径, 同时将GET请求指向/query两个`InfluxDB`服务器的路径。 

  * 中继将侦听HTTP或UDP写入, 并通过它们的HTTP写入端点将数据写入这两个服务器。如果写入是通过HTTP发送的, 则只要两个 `InfluxDB` 服务器之一返回成功, 中继就会返回成功响应。如果任一 `InfluxDB` 服务器返回400响应, 则该响应将立即返回给客户端。如果两个服务器都返回500, 则500将返回给客户端。

  * 通过这种设置, 可以维持一个中继或一个 `InfluxDB` 的故障, 同时仍然可以进行写入和服务查询。但是, 恢复过程将需要操作员干预。


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
│     ▼  ▼                ▼  ▼     │        
│  ┌──────────┐      ┌──────────┐  │        
│  │          │      │          │  │        
└─▶│ InfluxDB │      │ InfluxDB │◀─┘        
   │          │      │          │           
   └──────────┘      └──────────┘           

```



---

> 部署 influxdb-relay 也非常简单, 只需要通过 配置 `relay.toml` 文件中 `output` 属性指定多个 InfluxDB 既可。


```toml

[[http]]

name = "example-http"


bind-addr = "127.0.0.1:9096"

# Array of InfluxDB instances to use as backends for Relay.
output = [
    # name: name of the backend, used for display purposes only.
    # location: full URL of the /write endpoint of the backend
    # timeout: Go-parseable time duration. Fail writes if incomplete in this time.
    { name="local1", location="http://127.0.0.1:8086/write", timeout="10s" },
    { name="local2", location="http://127.0.0.1:7086/write", timeout="10s" },
]

[[udp]]
# Name of the UDP server, used for display purposes only.
name = "example-udp"

# UDP address to bind to.
bind-addr = "127.0.0.1:9096"

# Socket buffer size for incoming connections.
read-buffer = 0 # default

# Precision to use for timestamps
precision = "n" # Can be n, u, ms, s, m, h

# Array of InfluxDB instances to use as backends for Relay.
output = [
    # name: name of the backend, used for display purposes only.
    # location: host and port of backend.
    # mtu: maximum output payload size
    { name="local1", location="127.0.0.1:8089", mtu=512 },
    { name="local2", location="127.0.0.1:7089", mtu=1024 },
]

```


---

> 启动 influxdb-relay 服务 既可.

```bash

./influxdb-relay -config relay.toml &

```



---

### influxdb-relay 性能分析


> `influxdb-relay` 性能分析


**在当前最新版本 influxdb 单机版中 `vegeta 5000/s` 的情况下, 可以保持99%以上的成功率。然而 influxdb-relay 却在 `vegeta 3000/s` 的速度的时候就已经撑不住了**


**具体原因搜索了一下已经有人分析过了, 如下为别人的分析**



```go
var responses = make(chan *responseData, len(h.backends))

for _, b := range h.backends {
  b := b
  go func() {
    defer wg.Done()
    resp, err := b.post(outBytes, query, authHeader)
    if err != nil {
      log.Printf("Problem posting to relay %q backend %q: %v", h.Name(), b.name, err)
    } else {
      if resp.StatusCode/100 == 5 {
        log.Printf("5xx response for relay %q backend %q: %v", h.Name(), b.name, resp.StatusCode)
      }
      responses <- resp
    }
  }()
}

go func() {
  wg.Wait()
  close(responses)
  putBuf(outBuf)
}()

var errResponse *responseData

for resp := range responses {

...

```


* 首先这里, 创建了一个 `channel` -  `var responses = make(chan *responseData, len(h.backends))` ,  只有当 所有的 `backends` 都回复了之后, `responses channel` 才会关闭, 客户端才能拿到结果. 然而一旦某一个 `backends` 卡住了, 就要等待 go 的 http client timeout了, 这个timeout默认时间是10s, 相当于说客户端至少要等待 10s, 然而实际并不止这样。


* `retry.go` 中的部分代码:


```go
interval := r.initialInterval
for {
  resp, err := r.p.post(buf.Bytes(), batch.query, batch.auth)
  if err == nil && resp.StatusCode/100 != 5 {
    batch.resp = resp
    atomic.StoreInt32(&r.buffering, 0)
    batch.wg.Done()
    break
  }

  if interval != r.maxInterval {
    interval *= r.multiplier
    if interval > r.maxInterval {
      interval = r.maxInterval
    }
  }

  time.Sleep(interval)
}
```

* 当超时等 `statusCode >= 500 `的错误发生时, retry 会将这个请求加入bufer中, 然后由run方法获取batch并向后端influxdb请求。这时的逻辑是, 一旦请求失败, 就 sleep 一定时间, 而这个一定时间就是初始时间乘以一个放大因子, 放大因子默认是2, 于是客户端 就会在不断等待中, 最后超时。



---

### influxdb-relay 使用问题


1. grafana 需要配置很多个数据源。

2. 用户不能根据 `measurement` 来订阅数据。

3. 数据库挂掉, 就需要修改 grafana 的数据源。

4. 维护困难, 比如需要新增数据库, 用户需要配置多个数据源, 不能统一接入点。

5. 用户查询直连数据库, 用户 `select *` 容易导致数据库直接 OOM, 数据库重启。

6. relay提供的重写功能, 数据是保留在内存中, 一旦 influxdb 挂掉, 就会导致relay机器内存疯涨。



---



### InfluxDB Proxy


> 360开源的高可用方案 `InfluxDB Proxy` , 360官方 github 中此项目也已经2年未更新, 但是相对与 `influxdb-relay` 还是有更好的可用性.


---


