---
layout: post
title: Gin 使用 logrus 日志系统
categories: [golang,Go,web]
description: Gin 使用 logrus 日志系统
keywords: golang,Go,web
header-img: "img/pexels/triangular.jpeg"
catalog:    true
tags:
    - golang
    - web
---

# Go Web 编程

## logrus

* Logrus is a structured logger for Go (golang), completely API compatible with the standard library logger.

* Gin 框架的日志默认只会在控制台输出, 利用 `Logrus` 封装成一个中间件, 集成到 Gin 框架中。


### 安装

```shell
go get -u github.com/sirupsen/logrus

```


### import

```shell
import "github.com/sirupsen/logrus"

```




### Gin log 中间件

* Logrus 通过 `Hook` 支持多种方式的输出, Logrus 提倡将日志输出到 日志系统中, 而不是 文件, 如 我们常见的 `Elasticsearch` `Logstash` `Kibana` 等。

  1. 输出到文件中

  2. 输出到 Mongodb 中

  3. 输出到 Elasticsearch

  4. 输出到 Logstash 

  5. 输出到 Redis

  6. 输出到 InfluxDB

  7. 其他 Hook 


* logrus 日志中间件中, 可以定义几种方法, 用来区分日志发送到 `Hook` 如:

  1. `LoggerFile` 记录到 文件 的方法。

  2. `LoggerMongodb` 记录到 mongodb 的方法。

  3. `LoggerElasticsearch` 记录到 Elasticsearch 的方法。

  4. `LoggerLogstash` 记录到 Logstash 的方法。

  5.  ....


* logger.go

```go



```
