---
layout: post
title: 全局唯一ID生成器 Sonyflake
categories: [golang,Go]
description: 全局唯一ID生成器 Sonyflake
keywords: golang,Go
header-img: "img/pexels/triangular.jpeg"
catalog:    true
tags:
    - golang
---

# Go Web 编程


## Sonyflake

> 官方地址 github.com/sony/sonyflake

* `Sonyflake`是Sony公司基于 Twitter `snowflake` 实现的的一个Golang的全局性唯一ID生成器的开源项目, 基本思路和`snowflake` 差不多, 但也稍有不同。

* 如下为 Sonyflake 的ID 组成

|||||
|-|-|-|-|
|1 Bit Unused|39 Bit Timestamp|8 Bit Sequence ID|16 Bit Machine ID|



* 如下为 snowflake 的ID 组成

|||||
|-|-|-|-|
|1 Bit Unused|41 Bit Timestamp|10 Bit Node ID|12 Bit Sequence ID|


* 名词解析

  * Unused: 未使用, 二进制中最高位为1的都是负数, 我们生成的id一般都是整数, 所以这个最高位固定是0。

  * Timestamp: 时间戳. 

  * Sequence ID: 由程序在运行期生成。

  * Node ID: 机器编号的位长。

  * Machine ID:  其实就是 Node ID。


### 安装

```shell
go get -u github.com/sony/sonyflake

```

### import

```shell

import "github.com/sony/sonyflake"

```

