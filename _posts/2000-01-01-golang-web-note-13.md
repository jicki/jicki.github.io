---
layout: post
title: Gin templates
categories: [golang,Go,gin]
description: Gin templates
keywords: golang,Go,gin
header-img: "img/pexels/triangular.jpeg"
catalog:    true
tags:
    - golang
    - gin
---

# Go Web 编程


## gin 框架 templates


* templates 的基础语法以及概念在前面的文章中有记录

* [HTML 基础概念](https://jicki.me/golang/go/2000/01/01/golang-study-note-8 "HTML 基础")


### gin 下的例子

* go 代码

```go
package main

import (
	"fmt"
	"html/template"
	"net/http"

	"github.com/gin-gonic/gin"
)

type User struct {
	Name  string
	Age   int
	Hobby []string
}

// template 语法
func main() {
	s1 := User{
		Name: "张大仙",
		Age:  20,
		Hobby: []string{
			"王者",
			"英雄联盟",
			"说骚话",
		},
	}
	s2 := User{
		Name: "魔教教主",
		Age:  30,
		Hobby: []string{
			"英雄联盟",
			"刺激战场",
			"说骚话",
		},
	}
	r := gin.Default()

	// 自定义函数
	custom := func(name string) (string, error) {
		return fmt.Sprintf("hello %s", name), nil
	}
	// 导入 自定义函数到 模板中
	// 导入 自定义函数必须在 加载模板 之前,否则会报错
	r.SetFuncMap(template.FuncMap{
		"custom": custom,
	})

	// 加载模板
	r.LoadHTMLGlob("templates/*")
	// html 基础
	r.GET("/basic", func(c *gin.Context) {
		c.HTML(http.StatusOK, "basic.tmpl", gin.H{
			"s1": s1,
			"s2": s2,
		})
	})
	// html 自定义函数
	r.GET("/custom", func(c *gin.Context) {
		c.HTML(http.StatusOK, "custom.tmpl", gin.H{
			"s1": s1,
		})
	})
	// html 嵌套
	r.GET("/nest", func(c *gin.Context) {
		c.HTML(http.StatusOK, "nest.tmpl", nil)
	})
	_ = r.Run(":8888")
}
```

{% raw %}

* html 代码


* basic.tmpl

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <!-- 标题 -->
    <title>HTML基础</title>
</head>
<body>
    <!-- .变量 的使用-->
    <p>欢迎光临 {{ .s1.Name }}</p>
    <p>姓名: {{ .s1.Name }}</p>
    <p>年龄: {{ .s1.Age }}</p>
    <!-- 变量定义 与 判断语句 -->
    {{ $age := .s1.Age}}
    {{ if lt $age 22}}
        <p> {{ .s1.Name }} 好好学习 </p>
    {{else}}
        <p> {{ .s1.Name }} 好好工作 </p>
    {{end}}
    <!-- range 循环 -->
    <p>爱好:
        <br>
        {{ range $index, $hobby := .s1.Hobby }}
        {{ $index }} - {{ $hobby }} </p>
    {{end}}
    <hr>
    <!-- with 的使用 -->
    {{ with .s2 }}
    <p>欢迎光临 {{ .Name }}</p>
    <p>姓名: {{ .Name }}</p>
    <p>年龄: {{ .Age }}</p>
    <!-- 变量定义 与 判断语句 -->
    {{ $age := .Age }}
    {{ if lt $age 22 }}
        <p> {{ .Name }} 好好学习 </p>
    {{else}}
        <p> {{ .Name }} 好好工作 </p>
    {{end}}
    <!-- range 循环 -->
    <p>爱好:
        <br>
        {{ range $index, $hobby := .Hobby }}
        {{ $index }} - {{ $hobby }} </p>
    {{end}}
    {{end}}
    <hr>
    <!-- index 索引取值 -->
    <p>index 取 s2 Hobby 索引为[2]的值 = {{index .s2.Hobby 2}} </p>
</body>
</html>
```

* custom.tmpl

```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <!-- 标题 -->
    <title>HTML 进阶</title>
</head>
<body>
    {{ with .s1 }}
        <!-- 引用 自定义函数 -->
    <p> {{ custom .Name }}</p>
    {{end}}
</body>
</html>
```


* nest.tmpl


```html
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <!-- 标题 -->
    <title>HTML 嵌套</title>
</head>
<body>
    <!-- 嵌套另一个单独的模板文件 -->
    {{template "nesting" }}
<hr>
    {{template "temp"}}
</body>
</html>
<!-- define 自定义模板 -->
{{ define "temp"}}
    <ol>
        <li>青铜</li>
        <li>白银</li>
        <li>黄金</li>
    </ol>
{{end}}

```


* nesting.tmpl


```html

{{ define "nesting" }}
<!DOCTYPE html>
<html lang="zh-CN">
<head>
    <meta http-equiv="Content-Type" content="text/html; charset=utf-8">
    <!-- 标题 -->
    <title>HTML 嵌套的模板</title>
</head>
<body>
    <p>HTML 嵌套模板</p>
    <ul>
        <li>钻石</li>
        <li>星耀</li>
        <li>王者</li>
    </ul>
</body>
</html>
{{end}}

```
{% endraw %}
