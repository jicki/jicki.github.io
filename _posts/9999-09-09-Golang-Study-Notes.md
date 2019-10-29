---
layout: post
title: Go 语言学习笔记
categories: [golang,Go]
description: Go 语言学习笔记
keywords: golang,Go
header-img: "img/pexels/triangular.jpeg"
catalog:    true
tags:
    - golang
---


![logo][1]

# Go 简介

* Go（又称Golang）是Google开发的一种静态强类型、编译型、并发型，并具有垃圾回收功能的编程语言。

* 罗伯特·格瑞史莫，罗勃·派克（Rob Pike）及肯·汤普逊于2007年9月开始设计Go，稍后 Ian Lance Taylor、Russ Cox加入项目。

* Go是基于Inferno操作系统所开发的。

* Go于2009年11月正式宣布推出，成为开放源代码项目，支持Linux、Mac OS X、Windows等操作系统。

* Go在2016年，Go被软件评价公司TIOBE 选为“TIOBE 2016 年最佳语言”。


# Go 代理

* 为解决 go get 下载慢或者下载不到的问题

```shell
# Go 1.13 中使用 goproxy.cn 作为代理
# 执行如下命令

go env -w GOPROXY=https://goproxy.cn,direct

```


# Go 学习目录

[ 【Go学习笔记 - Go 语言结构】](https://jicki.me/golang/go/2000/01/01/golang-study-note-0 "Go 学习第零天")

[ 【Go学习笔记 - Go 常量变量 && 数据类型】](https://jicki.me/golang/go/2000/01/01/golang-study-note-1 "Go 学习第一天")

[ 【Go学习笔记 - Go 运算符 && 流程控制】](https://jicki.me/golang/go/2000/01/01/golang-study-note-2 "Go 学习第二天")

[ 【Go学习笔记 - Go 数组 切片 Map】](https://jicki.me/golang/go/2000/01/01/golang-study-note-3 "Go 学习第三天")

[ 【Go学习笔记 - Go 函数 闭包 panic 指针】](https://jicki.me/golang/go/2000/01/01/golang-study-note-4 "Go 学习第四天")

[ 【Go学习笔记 - 反射】](https://jicki.me/golang/go/2000/01/01/golang-study-note-5 "Go 学习第五天")

[ 【Go学习笔记 - 并发 && 锁】](https://jicki.me/golang/go/2000/01/01/golang-study-note-6 "Go 学习第六天")

[ 【Go学习笔记 - 网络编程-HTTP-TCP-UDP】](https://jicki.me/golang/go/2000/01/01/golang-study-note-7 "Go 学习第七天")

[ 【Go学习笔记 - HTML 基础】](https://jicki.me/golang/go/2000/01/01/golang-study-note-8 "Go 学习第八天")

  [1]: http://jicki.me/img/posts/golang/logo.jpg
