---
layout: post
title: Cookie 与 Session
categories: [golang,Go]
description: Cookie 与 Session
keywords: golang,Go
header-img: "img/pexels/triangular.jpeg"
catalog:    true
tags:
    - golang
---

# Go Web 编程

* HTTP 协议是无状态的，对于一个浏览器发出的多次请求，WEB 服务器无法区分, 是不是来源于同一个浏览器, 所以诞生了Cookie 与 Session  使某个域名下的所有网页能够共享某些数据.

## Cookie

* `Cookie` 实际上是一小段的文本信息`（key-value)` 形式, `Cookie` 是纯文本格式，不包含任何可执行的代码.

* 客户端向服务器发起请求，如果服务器需要记录该用户状态，就使用`Response`向客户端浏览器颁发一个`Cookie`。客户端浏览器会把`Cookie`保存起来。当浏览器再请求该网站时，浏览器把请求的网址连同该`Cookie`一同提交给服务器。服务器检查该`Cookie`，以此来辨认用户状态。


### Cookie 机制

1. 客户端`User-Agent`发送一个请求到服务器。

2. 服务器发送一个`HttpResponse`响应到客户端，其中包含`Set-Cookie`的头部。

3. 客户端保存`Cookie`, 之后向服务器发送请求时, `HttpRequest`请求中会包含一个`Cookie`的头部。

4. 服务器返回响应数据。 


### Cookie 特点

1. 客户端发送请求的时候, 会携带服务端`HttpResponse` 之前`Set-Cookie`的`Cookie`信息。

2. 服务端可以设置`Cookie`数据`key/value`信息。 

3. `Cookie`是针对单个域名的，不同域名之间的`Cookie`是独立的。 

4. `Cookie`数据可以配置过期时间，过期的`Cookie`数据会被系统清除。



### Golang 使用 Cookie

* Go语言中 `net/http` 标准库定义了 `Cookie` 。

```go
type Cookie struct {
    Name       string
    Value      string
    Path       string
    Domain     string
    Expires    time.Time
    RawExpires string
    // MaxAge=0 表示未设置Max-Age属性
    // MaxAge<0 表示立刻删除该Cookie，等价于"Max-Age: 0"
    // MaxAge>0 表示存在Max-Age属性，单位是秒
    MaxAge   int
    Secure   bool
    HttpOnly bool
    Raw      string
    Unparsed []string // 未解析的"key/value"对的原始文本
}
```



* 使用 `net/http` 标准库中的 `SetCookie` 函数, 设置 `Cookie`. 

```go
func SetCookie(w ResponseWiter, cookie *Cookie)
```


* 获取 `Cookie`, `Request` 对象拥有两个获取`Cookie`的方法和一个添加`Cookie`的方法

  * 获取 `Cookie` 方法

```go
// 1. 解析并返回该请求的Cookie头设置的所有Cookie
func (r *Request) Cookies() []*Cookie

// 2. 返回请求中名为name的Cookie，如果未找到该Cookie会返回nil, ErrNoCookie。
func (r *Request) Cookie(name string) (*Cookie, error)
```

  * 添加Cookie的方法

```go
// AddCookie向请求中添加一个Cookie。
func (r *Request) AddCookie(c *Cookie)
```



### Gin 框架 Cookie




