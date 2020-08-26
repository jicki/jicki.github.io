# Helm Chart & template 函数



# Helm Chart


**Chart 是 helm 管理应用的包, 一个Chart对应一个或一套应用. Chart 内部由YAML描述文件组成.**

---


## Chart 目录结构


* `Chart.yaml` :  chart 的描述文件, 包含版本信息, 名称 等.

* `Chart.lock` : chart 依赖的版本信息. ( apiVersion: v2 )

* `values.yaml` :  用于配置 `templates/` 目录下的模板文件使用的变量.

* `values.schema.json` : 用于校检 `values.yaml` 的完整性.

* `charts` : 依赖包的存储目录.

* `README` : 说明文件.

* `LICENSE` : 版权信息文件.

* `crd` : 存放 CRD 资源的文件的目录.

* `templates` : 模板文件存放目录.

  * `NOTES.txt` : 模板须知/说明文件. `helm install ` 成功后会显示此文件内容到屏幕.

  * `deployment.yaml` : kubernetes 资源文件. ( 所有类型的 kubernetes 资源文件都存放于 templates 目录下 )

  * `_helpers.tpl` :  以 `_` 开头的文件, 可以被其他模板引用. 



---


### Chart.yaml 文件


> Chart 的描述文件, 包含版本信息, Chart 名称,说明等.

* 这里仅说明 `apiVersion v2 版本`

```yaml
# Helm api 版本 (必填)
apiVersion: v2

# Chart 的名称 (必填)
name: myapp

# 此 Chart 的版本 (必填)
version: v1.0

# 约束此 Chart 支持的 kubernetes 版本, 如果不支持会失败 (可选)
kubeVersion: >= 1.18.0

# 此 Chart 说明信息 (可选)
description: "My App"

# Chart 的应用类型, 分别为 application (默认)和 library. (可选)
type: application

# Chart 关键词, 用于搜索时使用 (可选)
keywords
  - app
  - myapp

# Chart 的 home 的地址 (可选)
home: https://jicki.cn

# Chart 的源码地址 (可选)
sources:
  - https://github.com/jicki
  - https://jicki.cn


# Chart 的依赖信息, helm v2 是在 requirements.yaml 中. (可选)
dependencies:
  # 依赖的 chart 名称
  - name: nginx
  # 依赖的版本
    version: 1.2.3
  # 依赖的 repo 地址
    repository: https://kubernetes-charts.storage.googleapis.com
  # 依赖的 条件 如 nginx 启动
    condition: nginx.enabled
  # 依赖的标签 tags
    tags:
      - myapp-web
      - nginx-slb
    enabled: true
  # 传递值到 Chart 中. 
    import-values:
      - child:
      - parent:
  # 依赖的 别名
    alias: nginx-slb 


# Chart 维护人员信息 (可选) 
maintainers:
  - name: 小炒肉
    email: jicki@qq.com
    url: https://jicki.cn

# icon 地址 (可选)
icon: "https://jicki.cn/images/favicon.ico"

# App 的版本 (可选)
appVersion: 1.19.2

# 标注是否为过期 (可选)
deprecated: false

# 注释 (可选)
annotations:
  # 注释的例子
  example: 
```



---

## Helm 内置变量

* Helm 内置对象包含

  * `Release` 相关的内置属性.

  * `Chart` 属性 - `Chart.yaml` 文件中定义的内容.

  * `Values` 属性 - `Values.yaml` 文件中定义的内容.


---


### Release 内置变量


* `Release` 内置变量

| 变量名称           | 变量说明                                              |
|--------------------|-------------------------------------------------------|
| Release.Name       | release 名称                                          |
| Release.Namespace  | release 所在命名空间                                  |
| Release.Service    | release 发布的服务                                    |
| Release.IsUpgrade  | 如果 release 操作状态为 更新 / 回滚 , 此变量值为 true | 
| Release.IsInstall  | 如果 release 操作状态为 安装 , 此变量值为 true        | 
| Release.Revision   | release 版本序号, 初始为 1 , 执行 更新/ 回滚 版本 +1  |



---


## Helm 模板函数

* Helm 的模板语法使用Golang 语言的 template 模块 参考: https://jicki.cn/golang-study-note-8/#template-%E8%AF%AD%E6%B3%95 



### 内置的函数


---


> 比较函数

| 函数 |         含义              |
| ---- | ------------------------- |
| eq   | 如果 arg1 == arg2 返回 真 |
| ne   | 如果 arg1 != arg2 返回 真 |
| lt   | 如果 arg1 < arg2 返回 真  |
| le   | 如果 arg1 <= arg2 返回 真 |
| gt   | 如果 arg1 > arg2 返回 真  |
| ge   | 如果 arg1 >= arg2 返回 真 |

---


### 流程控制


> 模板函数 之 流程控制


* `values.yaml` 文件内容如下

```yaml
favorite:
  game: LOL
  drink: coffee
like:
  - football
  - basketball
  - volleyball
  - doubleball
global:
  service: web
image:
  repository: jicki/myapp
  tag: 'v2'
hostsPort:
  http: 80
  https: 443
containerPort:
  http: 80
  https: 443
```

---

> if / else 条件控制

* `if` / `else`  条件控制

  * `templates/if.yaml` 文件, 模板语法使用 go语言 template 语法

```yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "hello world"
  {{- if eq .Values.favorite.game "LOL" }}
  msg: "Good Game"
  {{- else }}
  msg: "Null"
  {{- end }}

```

* 查看 template

```shell

root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/if.yaml

---
# Source: myapp/templates/if.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-configmap
data:
  myvalue: "hello world"
  msg: "Good Game"

```


---

> with 范围控制


* `with` 范围控制, 加载范围主体为当前'.' , 后续通过 `.game | .drink` 直接调用

  * `templates/with.yaml` 文件内容


```yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "hello world"
  {{- with .Values.favorite }}
  # default 设置默认值. quote 是加上引号
  game: {{ .game | default "King" | quote }}
  # upper 将输出转换为 大写. quote 是加上引号 
  drink: {{ .drink | upper | quote }}
  {{- end }}

```


* 查看 template 

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/with.yaml
---
# Source: myapp/templates/with.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-configmap
data:
  myvalue: "hello world"
  game: "LOL"
  drink: "COFFEE"

```



---

> range 循环控制


* `range` 循环控制

  * `templates/range.yaml` 文件内容


```yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "hello world"
  like: |-
    # 遍历 数组
    {{- range .Values.like }}
    - {{ . }}
    {{- end }}
    # 遍历 map
    {{- range $key, $val := .Values.favorite }}
    {{ $key }} = {{ $val | quote }}
    {{- end }}

```


* 查看 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/range.yaml
---
# Source: myapp/templates/range.yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-configmap
data:
  myvalue: "hello world"
  like: |-
    # 遍历 数组
    - football
    - basketball
    - volleyball
    - doubleball
    # 遍历 map
    drink = "coffee"
    game = "LOL"
```


---

### 内置函数

> Go 语言函数


| 函数    |          含义                                         |
| ------- | ----------------------------------------------------- |
| and     | 函数返回它的第一个 empty 参数或者最后一个参数         |
| or      | 函数返回它的第一个非 empty 参数或者最后一个参数       |
| not     | 函数返回它的单个参数的布尔值是否定                    |
| len     | 函数返回它的参数的整数类型长度                        |
| index   | 函数执行结果为第一个参数以剩下的参数为索引/键指向的值 |
| html    | 函数返回其参数文本表示的 html 逸码等价表示            |
| js      | 函数返回其参数文本表示的 JavaScrpit 逸码等价表示      |
| slice   | 函数返回值为 slice 中的选定项                         |
| print   | 既 Go 语言中 fmt.Sprint                               |
| printf  | 既 Go 语言中 fmt.Sprintf                              |
| println | 既 Go 语言中 fmt.Sprintln                             |
| urlquery| 函数返回其参数文本表示可嵌入URL查询的逸码等价表示     |


---

> and 函数

* `and` 函数 

  * `templates/and.yaml` 文件内容

```yaml
# 如果 5 大于 1  and 3 小于等于 10
{{- if and (gt 5 1 ) (le 3 10) }}
msg: true
{{- else }}
msg: false
{{- end }}
```


* 运行 template 

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/and.yaml
---
# Source: myapp/templates/and.yaml
# 如果 5 大于 1  and 3 小于等于 10
msg: true

```


---

> or 函数

* `or` 函数

  * `templates/or.yaml` 文件内容

```yaml
# 判断 1 大于 2  或  1 小于 2 返回为 true
{{- if or (gt 1 2 ) (le 1 2) }}
msg: true
{{- else }}
msg: false
{{- end }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/or.yaml
---
# Source: myapp/templates/or.yaml
# 判断 1 大于 2  或  1 小于 2 返回为 true
msg: true
```



---

> not 函数

* `not` 函数

  * `templates/not.yaml` 文件内容


```yaml
# 判断 如果 1 大于 2 返回 false 取非 为 true
{{- if not (gt 1 2 ) }}
msg: true
{{- end}}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/not.yaml
---
# Source: myapp/templates/not.yaml
# 判断 如果 1 大于 2 返回 false 取非 为 true
msg: true

```



---

> len 函数

* `len` 函数

  * `templates/len.yaml` 文件内容

```yaml
service: {{ .Values.global.service }}

len: {{ len .Values.global.service }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/len.yaml
---
# Source: myapp/templates/len.yaml
service: web

len: 3

```



---

> index 函数

* `index` 函数

  * `templates/index.yaml` 文件内容

```yaml
# 定义一个变量 hostsPort
{{- $hostsPort := .Values.hostsPort -}}
apiVersion: v1
kind: ConfigMap
metadata:
  name: {{ .Release.Name }}-configmap
data:
  myvalue: "hello world"
  # 遍历 containerPort 的 k,v 值
  {{- range $name, $val := .Values.containerPort }}
  # $name 变量取值为 $hostsPort 中 $name 变量的值
  {{ $name }}: {{ index $hostsPort $name | default $val }}
  {{- end }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/index.yaml
---
# Source: myapp/templates/index.yaml
# 定义一个变量 hostsPort
apiVersion: v1
kind: ConfigMap
metadata:
  name: RELEASE-NAME-configmap
data:
  myvalue: "hello world"
  http: 80
  https: 443

```


---

> html 函数

* `html` 函数

  * `templates/html.yaml` 文件内容

```yaml

html: |
  {{ html "<html><head><title>Title</title></head></html>" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/html.yaml
---
# Source: myapp/templates/html.yaml
html: |
  &lt;html&gt;&lt;head&gt;&lt;title&gt;Title&lt;/title&gt;&lt;/head&gt;&lt;/html&gt;

```

---

> js 函数

* `js` 函数

  * `templates/js.yaml` 文件内容


```yaml
js: {{ js "<script>var a = 1; var b = a + 1; </scrpit>" }}

```


* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/js.yaml
---
# Source: myapp/templates/js.yaml
js: \x3Cscript\x3Evar a \x3D 1; var b \x3D a + 1; \x3C/scrpit\x3E

```



---

> slice 函数


* `slice` 函数

  * `templates/slice.yaml` 文件内容


```yaml
# 定义一个 slice slice{1, 3, 5, 7, 9}
{{ $li := list 1 3 5 7 9 }}

# 取 $li 中的 第2 第4 个值 从0开始
slice: {{ slice $li 2 4 }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/slice.yaml
---
# Source: myapp/templates/slice.yaml
# 定义一个 slice slice{1, 3, 5, 7, 9}

# 取 $li 中的 第2 第4 个值 从0开始
# 根据 go语言 slice 的取值 左包含右不包含 所以是5, 7
slice: [5 7]
```


---

> print / printf / println 函数


* `print`、`printf`、`println`  函数

  * `templates/print.yaml` 文件内容 


```yaml
apiVersion: v1
kind: Configmap
matadata:
  name: {{ .Release.Name }}-configmap
data:
  {{- $name := .Release.Name }}

  print: {{- print $name }}

  printf: {{- printf "game - %s drink - %s" .Values.favorite.game .Values.favorite.drink }}

  println: {{- println "The Game - " .Values.favorite.game }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/print.yaml
---
# Source: myapp/templates/print.yaml
apiVersion: v1
kind: Configmap
matadata:
  name: RELEASE-NAME-configmap
data:

  print:RELEASE-NAME

  printf:game - LOL drink - coffee

  println:The Game -  LOL

```


---

> urlquery 函数

* `urlquery`  函数

  * `templates/urlquery.yaml` 文件内容


```yaml

{{- $password := "https://jicki.cn/abc/" }}

password: {{ $password | urlquery }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/urlquery.yaml
---
# Source: myapp/templates/urlquery.yaml
password: https%3A%2F%2Fjicki.cn%2Fabc%2F

```


---


### template 模板函数


---

> hello 函数

* hello 函数

  * 输出 Hello! .

```yaml
hello: {{ hello }}
```

* 运行 template

```yaml
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/hello.yaml
---
# Source: myapp/templates/hello.yaml
hello: Hello!
```


---

> now 函数

* now 函数

  * 显示当前时间, 以 string 类型打印出来


```yaml
now: {{ now }}

```


* 运行 template

```yaml
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/now.yaml
---
# Source: myapp/templates/now.yaml
now: 2020-08-25 01:50:59.895593177 +0000 UTC m=+0.041493103
```


---

> ago 函数

* ago 函数

  * 当前时间减去过去时间

```yaml
ago {{ ago now }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/ago.yaml
---
# Source: myapp/templates/ago.yaml
ago: 0s
```


---

> date 函数

* date 函数

  * 格式化时间输出.  Go 语言的时间格式化

```yaml
date: {{ date "2006-01-02 15:04:05" (now) }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/date.yaml
---
# Source: myapp/templates/date.yaml
date: 2020-08-24 09:40:34
```



---

> dateInZone 函数

* dateInZone 函数

  * 带时区的时间格式化


```yaml
dateInZone: {{ dateInZone "2006-01-02 15:04:05" (now) "Asia/Shanghai" }}
```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/dateInZone.yaml
---
# Source: myapp/templates/dateInZone.yaml
dateInZone: 2020-08-24 17:48:42

```


---

> dateModify 函数

* `date_modify` 函数

  * 修改时间


```yaml
dateNow: {{ now }}
dateModify: {{ now | date_modify "-2h" }}
```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/datemodify.yaml
---
# Source: myapp/templates/datemodify.yaml
dateNow: 2020-08-24 09:52:41.016693113 +0000 UTC m=+0.045451911
dateModify: 2020-08-24 07:52:41.016733093 +0000 UTC m=-7199.954508131

```



---

> durationRound 函数

* durationRound 函数

  * 将指定秒 粗略转换成分钟 (m)

```yaml
durationRound: {{ durationRound "500s" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/durationRound.yaml
---
# Source: myapp/templates/durationRound.yaml
durationRound: 8m
```


---

> htmlDate 函数

* htmlDate 函数

  * 格式化时间为 年-月-日

```yaml
htmlDate: {{ now | htmlDate }}
```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/htmlDate.yaml
---
# Source: myapp/templates/htmlDate.yaml
htmlDate: 2020-08-24

```


---

> htmlDateInZone 函数

* htmlDateInZone 函数

  * 带时区的格式化时间 年-月-日

```yaml
htmlDateInZone: {{ htmlDateInZone now "Asia/Shanghai" }}
```


* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/htmlDateInZone.yaml
---
# Source: myapp/templates/htmlDateInZone.yaml
htmlDateInZone: 2020-08-24

```


---

> mustDateModify 函数

* mustDateModify 函数

  * mustDateModify 与 DateModify 函数一样, mustDateModify 会返回一个错误.

```yaml
DateNow: {{ now }}
mustDateModify: {{ mustDateModify "500s" now}}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/mustDateModify.yaml
---
# Source: myapp/templates/mustDateModify.yaml
DateNow: 2020-08-24 10:06:00.4835164 +0000 UTC m=+0.048029319
mustDateModify: 2020-08-24 10:14:20.483551062 +0000 UTC m=+500.048063982

```


---

> toDate 函数

* toDate 函数

  * 将 string 类型的时间转换成 时间类型并按照自定义格式化输出


```yaml
ToDate: {{ toDate "2006-01-02 15:04:05" "2020-08-25 09:58:00" }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/toDate.yaml
---
# Source: myapp/templates/toDate.yaml
ToDate: 2020-08-25 09:58:00 +0000 UTC

```


---

> unixEpoch 函数

* unixEpoch 函数

  * 将指定时间转换成 时间戳格式 输出

```yaml
unixEpoch: {{ unixEpoch now }}
``` 

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/unixEpoch.yaml
---
# Source: myapp/templates/unixEpoch.yaml
unixEpoch: 1598321573
```


---

> abbrev 函数

* abbrev 函数

  * 隐藏指定字符长度, 以 `...` 缩写. 


```yaml
abbrev: {{ "abcdefg" | abbrev 5 }}

``` 

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/abbrev.yaml
---
# Source: myapp/templates/abbrev.yaml
abbrev: ab...

```


---

> trunc 函数

* trunc 函数

  * 截取 指定长度的字符.


```yaml
trunc: {{ "abcdefg" | trunc 2 }}
```

* 运行 template

```shell

root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/trunc.yaml
---
# Source: myapp/templates/trunc.yaml
trunc: ab

```


---

> trim 函数

* trim 函数

  * 清除前后的 空格

```yaml
trim: {{ "     a  b  c  " |trim }}
```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/trim.yaml
---
# Source: myapp/templates/trim.yaml
trim: a  b  c
```

---

> trimAll 函数

* trimAll 函数

  * 清除前后指定的 字符.

```yaml
trimAll: {{ "++jicki++" | trimAll "+" }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/trimAll.yaml
---
# Source: myapp/templates/trimAll.yaml
trimAll: jicki

```


---

> trimSuffix 函数

* trimSuffix 函数

  * 清除 右边指定的 字符


---

> trimPrefix 函数

* trimPrefix 函数

  * 清除左边指定的 字符.


---

> nospace 函数

* nospace 函数

  * 清除字符串中的所有空格.



---

> initials 函数

* initials 函数

  * 截取字符串中每个单词的 第一个 字符.


```yaml
initials: {{ "Jicki hello world" |initials }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/initials.yaml
---
# Source: myapp/templates/initials.yaml
initials: Jhw

```



---

> upper 函数

* upper 函数

  * 将字符串全部转换成大写

```yaml
upper: {{ "abcdefg" | upper }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/upper.yaml
---
# Source: myapp/templates/upper.yaml
upper: ABCDEFG
``` 


---

> lower 函数

* lower 函数

  * 将字符串全部转换成小写

```yaml
lower: {{ "ABCDEFG" | upper }}
```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/lower.yaml
---
# Source: myapp/templates/lower.yaml
lower: abcdefg
```


---

> title 函数

* title 函数

  * 将单词的首写字母转换成大写


```yaml
title: {{ "jicki hello world" | title }}
title: {{ "jicki-hello-world" | title }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/title.yaml
---
# Source: myapp/templates/title.yaml
title: Jicki Hello World
title: Jicki-Hello-World
```



---

> swapcase 函数

* swapcase 函数

  * 将字符串中 大小写 互转. 


```yaml

swapcase: {{ "abcDEFghiJKL" |swapcase }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/swapcase.yaml
---
# Source: myapp/templates/swapcase.yaml
swapcase: ABCdefGHIjkl

```


---

> substr 函数

* substr 函数

  * 截取字符串 N-M 长度 的内容 ( N 从0开始, M = M-1).

```yaml
substr: {{ "abcdefghi" |substr 1 5 }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/substr.yaml
---
# Source: myapp/templates/substr.yaml
substr: bcde
```


---

> repeat 函数

* repeat 函数

  * 重复输出N次指定字符串.


```yaml
repeat: {{ "jicki" | repeat 5 }}
```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/repeat.yaml
---
# Source: myapp/templates/repeat.yaml
repeat: jickijickijickijickijicki

```

---

> randAlpha 函数

* randAlpha 函数

  * 输出指定位数随机 字母.


```yaml
randAlpha: {{ randAlpha 20 }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/randAlpha.yaml
---
# Source: myapp/templates/randAlpha.yaml
randAlpha: dmrHUNScimXhpUiGMapG

```


---

> randNumeric 函数

* randNumeric 函数

  * 输出指定位数的随机 数字


```yaml
randNumeric: {{ randNumeric 20 }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/randNumeric.yaml
---
# Source: myapp/templates/randNumeric.yaml
randNumeric: 37465873746598120398

```


---


> randAlphaNum 函数

* randAlphaNum 函数

  * 输出指定位数的随机 (数字 + 字母).


```yaml

randAlphaNum: {{ randAlphaNum 20 }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/randAlphaNum.yaml
---
# Source: myapp/templates/randAlphaNum.yaml
randAlphaNum: H2Bv00ToRKntFpy9PFYX

```


---

> randAscii 函数

* randAscii 函数

  * 输出指定位数随机 Ascii 码


---

> until 函数

* until 函数

  * 输出指定数的序列, 存到列表中.


```yaml
until: |
  {{ until 10 }}

```

* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/until.yaml
---
# Source: myapp/templates/until.yaml
until: |
        [0 1 2 3 4 5 6 7 8 9]

```


---

> untilStep 函数

* untilStep 函数

  * 输出指定 N-M 的序列, 并且可以设置 步长 , 存到列表中.


```yaml
untilStep: |
	{{ untilStep 1 10 2 }}

```

* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/untilStep.yaml
---
# Source: myapp/templates/untilStep.yaml
untilStep: |
        [1 3 5 7 9]
```


---


> shuffle 函数

* shuffle 函数

  * 将字符串的次序打乱.


```yaml
shuffle: {{ "jicki-hello-world" |shuffle }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/shuffle.yaml
---
# Source: myapp/templates/shuffle.yaml
shuffle: -lk-rdolijohwecli

```



---

> snakecase 函数

* snakecase 函数

  * 将字符串中的单词以 `_` 相连. 


```yaml
snakecase: {{ "jicki hello world"|snakecase }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/snakecase.yaml
---
# Source: myapp/templates/snakecase.yaml
snakecase: jicki_hello_world

```

---


> camelcase 函数

* camelcase 函数

  * 将字符串中的`_`相连的单词首字母大写并且去掉`_`.


```yaml
camelcase: {{ "jicki_hello_world_!" | camelcase }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/camelcase.yaml
---
# Source: myapp/templates/camelcase.yaml
camelcase: JickiHelloWorld!
```


---



> kebabcase 函数

* kebabcase 函数

  * 将字符串中的单词以 `-` 相连.


```yaml
kebabcase: {{ "jicki hello_world" | kebabcase }}

``` 

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/kebabcase.yaml
---
# Source: myapp/templates/kebabcase.yaml
kebabcase: jicki-hello-world
```



---

> wrapWith 函数

* wrapWith 函数

  * 在字符串中, 指定间隔插入指定字符串.


```yaml
wrapWith: |
        {{ "jickiHelloWorld" | wrapWith 5 ":-->" }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/wrapWith.yaml
---
# Source: myapp/templates/wrapWith.yaml
wrapWith: |
        jicki:-->Hello:-->World
```



---


> contains 函数

* contains 函数

  * 字符串中是否包含某些字符


```yaml
contains: |
  {{- $name := "jicki" }}
  {{- if contains $name "jicki hello world" }}
  name: "jicki"
  {{- else }}
  name: "Null"
  {{- end }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/contains.yaml
---
# Source: myapp/templates/contains.yaml
contains: |
  name: "jicki"
```


---


> hasPrefix 函数

* hasPrefix 函数

  * 字符串中是否包含指定的前缀.

```yaml
hasPrefix: |
        {{- if hasPrefix "https" "http://www.baidu.com" }}
        url: "https"
        {{- else }}
        url: "http"
        {{- end }}
```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/hasPrefix.yaml
---
# Source: myapp/templates/hasPrefix.yaml
hasPrefix: |
        url: "http"

```


---


> hasSuffix 函数

* hasSuffix 函数

  * 字符串中是否包含指定的后缀.

```yaml
hasSuffix: |
        {{- if hasSuffix "com" "https://baidu.com" }}
        url: "url com"
        {{- else }}
        url: "not com"
        {{- end }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/hasSuffix.yaml
---
# Source: myapp/templates/hasSuffix.yaml
hasSuffix: |
        url: "url com"
```


---


> quote 函数

* quote 函数

  * 将指定字符串加上 `" "` 号输出.


```yaml
quote: {{ quote "jicki" }}
Notquote: {{ "jicki" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/quote.yaml
---
# Source: myapp/templates/quote.yaml
quote: "jicki"
Notquote: jicki

```



---


> squote 函数

* squote 函数

  *  将指定字符串加上 `' '` 号输出.


```yaml
squote: {{ squote "jicki" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/squote.yaml
---
# Source: myapp/templates/squote.yaml
squote: 'jicki'

```


---

> cat 函数

* cat 函数

  * 字符串拼接, 多个字符串拼接成一个字符串.


```yaml
cat: |
        {{ cat "jicki" "hello" "world" "!" }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/cat.yaml
---
# Source: myapp/templates/cat.yaml
cat: |
        jicki hello world !

```


---

> indent 函数

* indent 函数

  * 字符串前面缩进指定的行数.


```yaml

indent: {{ "jicki" | indent 3 |quote }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/indent.yaml
---
# Source: myapp/templates/indent.yaml
indent: "   jicki"
```


---

> nindent 函数

* nindent 函数

  * 字符串换行后缩进指定的行数.

```yaml
nindent: |
  {{ "jicki" | nindent 3 |quote }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/nindent.yaml
---
# Source: myapp/templates/nindent.yaml
nindent: |
        "\n   jicki"
```


---

> replace 函数

* replace 函数

  * 替换字符串中的 n 字符为 m 字符.


```yaml
replace: |
        {{ "http://www.baidu.com" |replace "baidu" "qq" }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/replace.yaml
---
# Source: myapp/templates/replace.yaml
replace: |
        http://www.qq.com

```



---


> sha1sum 函数

* sha1sum 函数

  * 使用 sha1sum 算法加密 字符串.


```yaml
sha1sum: {{ sha1sum "jicki" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/sha1sum.yaml
---
# Source: myapp/templates/sha1sum.yaml
sha1sum: 7afef0c7f74b1aec91b159ac36946e0f865eb0e3
```


---


> sha256sum 函数

* sha256sum 函数

  * 使用 sha256sum 算法加密 字符串.


```yaml
sha256sum: {{ sha256sum "jicki" }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/sha256sum.yaml
---
# Source: myapp/templates/sha256sum.yaml
sha256sum: 57a4bffd5916cd9fded9ad94a76de152e56eb0bf9e34c94f5ea48f8cbd4f866a
```


---


> toString 函数

* toString 函数

  * 将 int 类型的函数转换成 string 类型.


```yaml
{{- $age := 22 }}
toString: {{ $age |toString }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/toString.yaml
---
# Source: myapp/templates/toString.yaml
toString: 22
```


---

> atoi 函数

* atoi 函数

  * 将 string 类型转换成 int 类型, 如果 string 为字符非数字, 转换后输出 0 .


```yaml
{{- $name := "2222" }}
atoi: {{ $name | atoi }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/atoi.yaml
---
# Source: myapp/templates/atoi.yaml
atoi: 2222
```


---

> int64 函数

* int64 函数

  * 将 int 类型转换成 int64 位的整型.

---


> float64 函数

* float64 函数

  * 将 float32 类型转换为 float64 位的浮点型.


---

> toDecimal 函数

* toDecimal 函数

  * 将 八进制 转换为 十进制. 


---

> split 函数

* split 函数

  * 以指定符号 分割 字符串, 并存到 map 中.


```yaml

split: |
        {{ split "," "jicki,hello,world!" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/split.yaml
---
# Source: myapp/templates/split.yaml
split: |
        map[_0:jicki _1:hello _2:world!]
```

---

> toStrings 函数

* toStrings 函数

  * 将 整型(int)的列表 转换成 字符串(string) 类型的列表


```yaml
toStrings: |
        {{- $li := list 1 2 3 }}
        {{ toStrings $li }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/toStrings.yaml
---
# Source: myapp/templates/toStrings.yaml
toStrings: |
        [1 2 3]
```

---

> toJson 函数

* toJson 函数

  * 将指定元素转换成 Json 


```yaml
toJson: {{ .Values.favorite | toJson }}
```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/toJson.yaml
---
# Source: myapp/templates/toJson.yaml
toJson: {"drink":"coffee","game":"LOL"}
```


---

> toPrettyJson 函数

* toPrettyJson 函数

  * 将指定元素转换成Json 并格式化输出.


```yaml
toPrettyJson: {{ .Values.favorite | toPrettyJson }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/toPrettyJson.yaml
---
# Source: myapp/templates/toPrettyJson.yaml
toPrettyJson: {
  "drink": "coffee",
  "game": "LOL"
}
```


---

> splitList 函数

* splitList 函数

  * 以指定符号 分割 字符串, 并存到 列表中.

```yaml

splitList: |
        {{ splitList "," "jicki,hello,world!" }}

```


* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/splitList.yaml
---
# Source: myapp/templates/splitList.yaml
splitList: |
        [jicki hello world!]

```


---

> splitn 函数

* splitn 函数

  * 以指定符号 分割 N 个字符串, 并存到 map 中.


```yaml

splitn: |
        {{ splitn ","  2  "jicki,hello,world!" }}

```

* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/splitn.yaml
---
# Source: myapp/templates/splitn.yaml
splitn: |
        map[_0:jicki _1:hello,world!]
```


---

> 算数运算 函数


| 运算符 |              说明                 |
| ------ | --------------------------------- |
| add    |  加法运算 {{ add 10 20 }} = 30    |
| sub    |  减法运算 {{ sub 20 10 }} = 10    |
| div    |  除法运算 {{ div 10 2 }}  = 5     |
| mul    |  乘法运算 {{ mul 3 3 }} = 9       |
| mod    |  取余 {{ mod 10 3 }} = 1          |  
| max    |  取最大值 {{ max 1 3 5 99 }} = 99 | 
| min    |  取最小值 {{ min 1 20 7 10 }} = 1 |
| ceil   |  向上取整 {{ ceil 3.823 }} = 4    |
| floor  |  向下取整 {{ floor 3.823 }} = 3   |
| round  |  四舍五入 {{ round 3.823 }} = 4   |


---


> join 函数

* join 函数

  * 通过指定 字符将 列表 中的元素合并为一个字符串.

```yaml

join: |
        {{- $li := list "a" "b" "c" "d" }}
        {{ join "_" $li }}

```



* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/join.yaml
---
# Source: myapp/templates/join.yaml
join: |
        a_b_c_d

```


---

> sortAlpha 函数

* sortAlpha 函数

  * 将 列表 中的字符串按照 字符的顺序进行排序.


```yaml
sortAlpha: |
        {{- $li := list "bb" "cc" "def" "func" "abc" }}
        {{ println "old-->" $li }}
        {{ $li = sortAlpha $li }}
        {{ println "new-->" $li }}
```

* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/sortAlpha.yaml
---
# Source: myapp/templates/sortAlpha.yaml
sortAlpha: |
        old--> [bb cc def func abc]


        new--> [abc bb cc def func]

```


---

> default 函数

* defalut 函数

  * 设置默认值.


```yaml
default: {{ defalut 99 "" }}

```


---


> empty 函数

* empty 函数

  * 判断字符串是否为空.


```yaml
empty: |
        {{- $name := "" }}
        {{- if empty $name }}
         name: "Null"
        {{- else }}
         name : {{ $name }}
        {{- end }}
```


* 运行 template 

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/empty.yaml
---
# Source: myapp/templates/empty.yaml
empty: |
         name: "Null"

```



---


> compact 函数

* compact 函数

  * 去除 列表 中为空 的元素.


```yaml
compact: |
        {{- $li := list "a" "" "c" "d" "f" }}
        {{ compact $li }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/compact.yaml
---
# Source: myapp/templates/compact.yaml
compact: |
        [a c d f]
```


---





