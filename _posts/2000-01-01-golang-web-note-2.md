---
layout: post
title: Gin 框架
categories: [golang,Go]
description: Gin 框架
keywords: golang,Go
header-img: "img/pexels/triangular.jpeg"
catalog:    true
tags:
    - golang
---

# Go Web 编程

## Gin 框架

>官方 中文文档 https://gin-gonic.com/zh-cn/docs/


## Gin 简介

* Gin 是一个用 Go (Golang) 编写的 HTTP web 框架。 使用 `httprouter`, 因此是一个拥有很好性能的API框架。


## Gin 特性

* 快速
  * 基于 Radix 树的路由，内存占用小。没有使用反射。API 性能可以直观的测试出来。

* 支持中间件
  * 传入的 HTTP 请求可以由一系列中间件和最终操作来处理。 例如：Logger，Authorization，GZIP，最终操作 DB。

* Crash 处理
  * Gin 可以 捕获 发生在 HTTP 请求中的 panic 并 recover 它。这样，你的服务器将始终可用。例如，你可以向 Sentry 报告这个 panic .

* JSON 验证
  * Gin 可以解析并验证请求的 JSON，例如检查所需值的存在。

* 路由组
  * 更好地组织路由。是否需要授权，不同的 API 版本,  此外，这些组可以无限制地嵌套而不会降低性能。

* 错误管理
  * Gin 提供了一种方便的方法来收集 HTTP 请求期间发生的所有错误。使用中间件可以将错误写入日志文件，数据库里。

* 内置渲染
  * Gin 为 JSON，XML 和 HTML 渲染提供了易于使用的 API。

* 可扩展性
  * 可以很简单创建中间件。

## 安装使用

* 通过 go get 并使用 import 导入包, 既可。

```shell
go get -u github.com/gin-gonic/gin
```

```go

import "github.com/gin-gonic/gin"

```

* 一个例子

```go
package main

import (
	"github.com/gin-gonic/gin"
	"log"
)

func main() {
	// 创建一个 gin实例,返回一个 *engine 路由引擎
	r := gin.Default()
	// 创建一个GET 方法 的 /hello 的路由
	// func 使用 匿名函数方式
	r.GET("/hello",func(c *gin.Context){
		// 使用 JSON格式,方式, 状态码为 200
		// gin.H 是返回一个map
		c.JSON(200,gin.H{
			"message":"hello world",
		})
	})
	// 启动 gin 服务
	if err := r.Run(":8888");err!=nil{
		log.Fatal(err.Error())
	}
}
```

```shell
# 访问 http://127.0.0.1:8888/hello

# 显示一个 json 格式的数据

{"message":"hello world"}

```
