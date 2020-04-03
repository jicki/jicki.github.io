---
layout: post
title: Golang 正则表达式
categories: [golang,Go]
description: Golang 正则表达式
keywords: golang,Go
header-img: "img/pexels/triangular.jpeg"
catalog:    true
tags:
    - golang
---

# 正则表达式

* 在 Golang 中 正则表达式 使用 `regexp` 包



## 常用正则语法

|语法|说明|表达式实例|
|-|-|-|-|
|`\d`|表示数字[0-9]|`a\dc`|
|`\D`|表示非数字的其他字符|`a\Dc`|
|`\w`|单词字符: 大小写字母、数字、下划线|`a\wc`|
|`\W`|非单词字符|`a\Wc`|
|`\s`|空白字符: `\t`、`\n`、`\r`、`\f` 其中之一|`a\sc`|
|`\S`|非空白字符|`a\Sc`|
|`.`|除了换行符之外的任意字符|`a.c`|
|`\.`| 表示符号 . |`abc\.com`|
|`+`|匹配前一个字符1次或无限次|`abc+`|
|`*`|匹配前一个字符0或无限次|`abc*`|
|`?`|匹配前一个字符0次或1次|`abc?`|
|`{m}`|匹配前一个字符m次|`ab{2}c`|
|`{m,n}`|匹配前一个字符m至n次。|`ab{1,2}c`|
|`{,n}`|匹配前面一个字符0至n次。|`ab{,2}c`|
|`{m,}`|匹配前面一个字符m至无限次|`ab{3,}`|
|`[...]`|匹配`[]`内的字符其中之一|`a[bcd]c`|
|`[\s\S]`|匹配任意字符|`12[\s\S]d`|
|`[a-z]`|匹配 a到z中的任意一个小写字符|`1[a-z]2`|
|`[^abc]`|匹配 除了abc之外的任意字符|`aa[^abc]bb`|
|`|`|代表左右表达式任意匹配一个|`abc|def`|
|`^`|匹配字符串开头|`^abc`|
|`$`|匹配字符串末尾|`abc$`|


## regexp 模块使用


```go
package main

import (
	"fmt"
	"regexp"
)

func main() {

	buf := "abc azc a7c aac 888 a9c  tac"

	//1) 解释规则, 解析正则表达式，如果成功返回解释器
	reg1 := regexp.MustCompile(`a[^a]c`)
	if reg1 == nil {
		fmt.Println("regexp err")
		return
	}

	//2) 根据规则提取关键信息
	result1 := reg1.FindAllStringSubmatch(buf, -1)
	fmt.Println("result1 = ", result1)
}
```

* 输出结果


```
result1 =  [[abc] [azc] [a7c] [a9c]]
```

