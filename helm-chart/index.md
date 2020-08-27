# Helm Chart & template 函数



# Helm Chart


Chart 是 helm 管理应用的包, 一个Chart对应一个或一套应用. Chart 内部由YAML描述文件组成.

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

* Helm 的模板语法使用Golang 语言的 template 模块. 

  * 参考: https://jicki.cn/golang-study-note-8/#template-%E8%AF%AD%E6%B3%95 

---

注: 所有函数中 函数名称前面带: `must` 如: mustxxx 的函数,指定出错不会 panic , 而是会输出 错误.

---

### 比较函数

> <center>比较函数</center>

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


> <center>模板函数 之 流程控制</center>


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

> <center>if / else 条件控制</center>

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

> <center>with 范围控制</center>


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

> <center>range 循环控制</center>


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

> <center>Go 语言函数</center>


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

> <center>and 函数</center>

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

> <center>or 函数</center>

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

> <center>not 函数</center>

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

> <center>len 函数</center>

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

> <center>index 函数</center>

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

> <center>html 函数</center>

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

> <center>js 函数</center>

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

> <center>slice 函数</center>


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

slice: [5 7]
```


---

> <center>print / printf / println 函数</center>


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

> <center>urlquery 函数</center>

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

> <center>hello 函数</center>

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

> <center>now 函数</center>

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

> <center>ago 函数</center>

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

> <center>date 函数</center>

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

> <center>dateInZone 函数</center>

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

> <center>dateModify 函数</center>

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

> <center>durationRound 函数</center>

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

> <center>htmlDate 函数</center>

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

> <center>htmlDateInZone 函数</center>

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

> <center>mustDateModify 函数</center>

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

> <center>toDate 函数</center>

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

> <center>unixEpoch 函数</center>

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

> <center>abbrev 函数</center>

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

> <center>trunc 函数</center>

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

> <center>trim 函数</center>

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

> <center>trimAll 函数</center>

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

> <center>trimSuffix 函数</center>

* trimSuffix 函数

  * 清除 右边指定的 字符


---

> <center>trimPrefix 函数</center>

* trimPrefix 函数

  * 清除左边指定的 字符.


---

> <center>nospace 函数</center>

* nospace 函数

  * 清除字符串中的所有空格.



---

> <center>initials 函数</center>

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

> <center>upper 函数</center>

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

> <center>lower 函数</center>

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

> <center>title 函数</center>

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

> <center>swapcase 函数</center>

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

> <center>substr 函数</center>

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

> <center>repeat 函数</center>

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

> <center>randAlpha 函数</center>

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

> <center>randNumeric 函数</center>

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


> <center>randAlphaNum 函数</center>

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

> <center>randAscii 函数</center>

* randAscii 函数

  * 输出指定位数随机 Ascii 码


---

> <center>until 函数</center>

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

> <center>untilStep 函数</center>

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


> <center>shuffle 函数</center>

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

> <center>snakecase 函数</center>

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

  * 生成 sha1sum 字符串.


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

  * 生成 sha256sum 字符串.


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

> b64enc 函数

* b64enc 函数

  * 使用 b64enc 加密算法 对指定字符串加密.

```yaml
b64enc: |
        {{ b64enc "jicki" }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/b64enc.yaml
---
# Source: myapp/templates/b64enc.yaml
b64enc: |
        amlja2k=

```


---

> b64dec 函数

* b64dec 函数

  * 对使用 b64enc 加密过的字符串进行解密.


```yaml
b64enc: {{ b64enc "jicki" }}
b64dec: {{ b64enc "jicki" | b64dec }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/b64dec.yaml
---
# Source: myapp/templates/b64dec.yaml
b64enc: amlja2k=
b64dec: jicki

```

---

> encryptAES 函数

* encryptAES 函数

  * 使用 AES 加密算法加密 字符串. (需指定一个 偏引量如: "98765")


```yaml

encryptAES: {{ encryptAES "98765" "jicki" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/encryptAES.yaml
---
# Source: myapp/templates/encryptAES.yaml
encryptAES: Lj9RyzT4kG6Q7K+hRil1/4l8EL4rXPqGD3/CF1KSS70=
```


---

> decryptAES 函数

* decryptAES 函数

  * 解密 AES 加密过的字符串.


```yaml
decryptAES: |
        encryptAES: {{ encryptAES "98765" "jicki" }}
        decryptAES: {{ encryptAES "98765" "jicki" | decryptAES "98765" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/decryptAES.yaml
---
# Source: myapp/templates/decryptAES.yaml
decryptAES: |
        encryptAES: H/UMBNfOx69KFHskREB4PeUEbdzv+ZLkCFUlN4vzRgw=
        decryptAES: jicki

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


> deepCopy 函数

* deepCopy 函数

  * 深度拷贝. 生成一份新的.


```yaml
deepCopy: |
        {{- $li := list 1 2 4 5 }}
        {{ deepCopy $li }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/deepCopy.yaml
---
# Source: myapp/templates/deepCopy.yaml
deepCopy: |
        [1 2 4 5]
```


---

> deepEqual 函数

* deepEqual 函数

  * 深度比较两个 元素 是否相等.

```yaml
deepEqual: |
        {{- $li1 := list 12 23 34 }}
        {{- $li2 := list 12 23 34 }}
        {{ deepEqual $li1 $li2 }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/deepEqual.yaml
---
# Source: myapp/templates/deepEqual.yaml
deepEqual: |
        true
```

---

> typeOf 函数

* typeOf 函数

  * 获取 元素 的类型 


```yaml
typeOf: |
        {{- $li := list 12 23 34 }}
        {{ typeOf $li }}
        {{ typeOf "String" }}
        {{ typeOf 123 }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/typeOf.yaml
---
# Source: myapp/templates/typeOf.yaml
typeOf: |
        []interface {}
        string
        int

```


---

> typeIs 函数

*  typeIs 函数

  * 类型断言, 判断元素的类型是否为指定类型


```yaml
typeIs: |
        {{- if typeIs "int" 123 }}
        typeIs: true
        {{- else }}
        typeIs: false
        {{- end }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/typeIs.yaml
---
# Source: myapp/templates/typeIs.yaml
typeIs: |
        typeIs: true

```



---

> kindOf 函数

* kindOf 函数

  * 获取一个元素的类型, 深度获取. 可查看如下与 `typeOf` 的输出区别.


```yaml
kindOf: |
        {{- $li := list 12 23 34 }}
        {{ kindOf $li }}
        {{ kindOf "String" }}
        {{ kindOf 123 }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/kindOf.yaml
---
# Source: myapp/templates/kindOf.yaml
kindOf: |
        slice
        string
        int
```


---

> kindIs 函数

* kindIs 函数

  * 类型断言, 判断元素的类型是否为指定类型, 可获取深度的类型. 如: interface .


```yaml

kindIs: |
        {{- $li := list 12 34 56 }}
        {{- if kindIs "slice" $li }}
        kindIs: slice
        {{- else }}
        kindIs: interface
        {{- end }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/kindIs.yaml
---
# Source: myapp/templates/kindIs.yaml
kindIs: |
        kindIs: slice
```


---



> getHostByName 函数

* getHostByName 函数

  * 获取指定 名称 的 IPv4 地址.


```yaml
jicki.cn: {{ getHostByName "www.jicki.cn" }}
localhost: {{ getHostByName "localhost" }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/getHostByName.yaml
---
# Source: myapp/templates/getHostByName.yaml
jicki.cn: 163.181.36.174
localhost: 127.0.0.1

```


---


> base 函数

* base 函数

  * 获取 URL 的文件名


```yaml
base: |
        {{ base "/usr/local/bin/helm" }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/base.yaml
---
# Source: myapp/templates/base.yaml
base: |
        helm
```



---

> dir 函数

* dir 函数

  * 获取文件 URL 的路径


```yaml
dir: |
        {{ dir "/usr/local/bin/helm" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/dir.yaml
---
# Source: myapp/templates/dir.yaml
dir: |
        /usr/local/bin
```



---

> clean 函数

* clean 函数

  * 删除 URL 路径末尾的 `/` 号.

```yaml
clean: |
        {{ clean "/usr/local/bin/" }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/clean.yaml
---
# Source: myapp/templates/clean.yaml
clean: |
        /usr/local/bin
```



---

> ext 函数

* ext 函数

  * 获取 文件名的 后缀.


```yaml
root@kubernetes:/opt/helm/myapp# cat templates/ext.yaml
ext: |
        {{ ext "/opt/helm/Chart.yaml" }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/ext.yaml
---
# Source: myapp/templates/ext.yaml
ext: |
        .yaml
```


---


> isAbs 函数

* isAbs 函数

  * 判断 路径 是否为 绝对路径

```yaml
isAbs: |
        {{- if isAbs "/opt/helm" }}
        isAbs: true
        {{- else }}
        isAbs: false
        {{- end }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/isAbs.yaml
---
# Source: myapp/templates/isAbs.yaml
isAbs: |
        isAbs: true
```



---

> tuple / list 函数

* tuple / list 函数

  * 创建一个 列表/元祖.


```yaml
tuple: |
        {{- $tp:= tuple "1" "2" "3" }}
        tuple: {{ $tp }}
        typeOf: {{ typeOf $tp }}
        kindOf: {{ kindOf $tp }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/tuple.yaml
---
# Source: myapp/templates/tuple.yaml
tuple: |
        tuple: [1 2 3 4]
        typeOf: []interface {}
        kindOf: slice
```

---


> dict 函数

* dict 函数

  * 创建一个 map.


```yaml
dict: |
        {{- $dt:= dict "name" "jicki"  "age" 22 }}
        tuple: {{ $dt }}
        typeOf: {{ typeOf $dt }}
        kindOf: {{ kindOf $dt }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/dict.yaml
---
# Source: myapp/templates/dict.yaml
dict: |
        tuple: map[age:22 name:jicki]
        typeOf: map[string]interface {}
        kindOf: map

```

---

> append 函数

* append 函数

  * 追加 元素 到 list  中.


```yaml
append: |
        {{- $li := list }}
        {{ $li = append $li 123 }}
        {{ $li = append $li 567 }}
        list: {{ $li }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/append.yaml
---
# Source: myapp/templates/append.yaml
append: |


        list: [123 567]
```

---

> prepend 函数

* prepend 函数

  * 在 list 中的最前面 追加 元素.

```yaml
prepend: |
        {{- $li := list }}
        {{ $li = prepend $li 123 }}
        {{ $li = prepend $li 567 }}
        list: {{ $li }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/prepend.yaml
---
# Source: myapp/templates/prepend.yaml
prepend: |


        list: [567 123]

```


---

> first 函数

* first 函数

  * 获取 list 中的第一个元素.


```yaml
first: |
        {{- $li := list 1 2 3 4 5 }}
        {{ first $li }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/first.yaml
---
# Source: myapp/templates/first.yaml
first: |
        1
```

---


> rest 函数

* rest 函数

  * 获取 list 中 除第一个元素外的 其他所有 元素.


```yaml
rest: |
        {{- $li := list 1 2 3 4 5 }}
        {{ rest $li }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/rest.yaml
---
# Source: myapp/templates/rest.yaml
rest: |
        [2 3 4 5]

```

---


> last 函数

* last 函数

  * 获取 list 中 最后一个元素.

```yaml
last: |
        {{- $li := list 1 2 3 4 5 }}
        {{ last $li }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/last.yaml
---
# Source: myapp/templates/last.yaml
last: |
        5

```



---

> initial 函数

* initial 函数

  * 获取 list 中 除最后一个元素 外的所有元素.


```yaml
initial: |
        {{- $li := list 1 2 3 4 5 }}
        {{ initial $li }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/initial.yaml
---
# Source: myapp/templates/initial.yaml
initial: |
        [1 2 3 4]

```

---

> reverse 函数

* reverse 函数

  * 对 list 进行 倒序 排序.


```yaml
reverse: |
        {{- $li := list 1 2 3 4 5 }}
        {{ reverse $li }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/reverse.yaml
---
# Source: myapp/templates/reverse.yaml
reverse: |
        [5 4 3 2 1]
```

---

> uniq 函数

* uniq 函数

  * 去除重复的元素


```yaml
uniq: |
        {{- $li := list 1 1 2 3 3 4 5 5 }}
        {{ uniq $li }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/uniq.yaml
---
# Source: myapp/templates/uniq.yaml
uniq: |
        [1 2 3 4 5]

```

---

> without 函数

* without 函数

  * 删除 list 中的一个元素


```yaml
without: |
        {{- $li := list 1 2 3 4 5 }}
        {{ without $li 3 }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/without.yaml
---
# Source: myapp/templates/without.yaml
without: |
        [1 2 4 5]
```

---

> has 函数

* has 函数

  * 判断指定 元素 是否在 list 中.


```yaml
has: |
        {{- $li := list 1 2 3 4 5 }}
        {{- if has 2 $li }}
        has: true
        {{- end }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/has.yaml
---
# Source: myapp/templates/has.yaml
has: |
        has: true

```


---

> concat 函数

* concat 函数

  * 合并两个 list 


```yaml
concat: |
        {{- $li1 := list 1 2 3 4 }}
        {{- $li2 := list 5 6 7 8 }}
        {{ concat $li1 $li2 }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/concat.yaml
---
# Source: myapp/templates/concat.yaml
concat: |
        [1 2 3 4 5 6 7 8]

```


---


> get 函数

* get 函数

  * 获取 map 中指定 key 的 value 值.


```yaml
get: |
        {{- $mp := dict "name" "jicki" "age" 12 }}
        {{ get $mp "name" }}

``` 

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/get.yaml
---
# Source: myapp/templates/get.yaml
get: |
        jicki
```


---

> set 函数

* set 函数

  * 给指定的 map 设置 key/value 键值对.


```yaml

set: |
        {{- $mp := dict "name" "jicki" "age" 12 }}
        {{ set $mp "email" "jicki@qq.com" }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/set.yaml
---
# Source: myapp/templates/set.yaml
set: |
        map[age:12 email:jicki@qq.com name:jicki]

```



---

> unset 函数

* unset 函数

  * 删除 map 中指定的 key/value 值.


```yaml
unset: |
        {{- $mp := dict "name" "jicki" "age" 12 }}
        {{ unset $mp "age"}}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/unset.yaml
---
# Source: myapp/templates/unset.yaml
unset: |
        map[name:jicki]

```



---

> hasKey 函数

* hasKey 函数

  * 判断 map 中是否有指定的 key .


```yaml
hasKey: |
        {{- $mp := dict "name" "jicki" "age" 12 }}
        {{ hasKey $mp "age"}}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/haskey.yaml
---
# Source: myapp/templates/haskey.yaml
hasKey: |
        true
```


---

> pluck 函数

* pluck 函数

  * 获取两个 map 中指定的 相同的 key 值, 组成一个新的 map.

```yaml
pluck: |
        {{- $mp1 := dict "name" "jicki" "age" 12 }}
        {{- $mp2 := dict "name" "tom" "age" 20 }}
        {{ pluck "name" $mp1 $mp2 }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/pluck.yaml
---
# Source: myapp/templates/pluck.yaml
pluck: |
        [jicki tom]
```


---

> keys 函数

* keys 函数

  * 获取一个或多个 map 中的所有 key.


```yaml
keys: |
        {{- $mp1 := dict "name" "jicki" "age" 12 }}
        {{- $mp2 := dict "name" "tom" "age" 20 }}
        {{ keys $mp1 $mp2 }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/keys.yaml
---
# Source: myapp/templates/keys.yaml
keys: |
        [name age name age]

```

---

> values 函数

* values 函数

  * 获取 map 中的 value 值.

```yaml
values: |
        {{- $mp := dict "name" "jicki" "age" 12 "email" "jicki@qq.com" }}
        {{ values $mp }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/values.yaml
---
# Source: myapp/templates/values.yaml
values: |
        [jicki 12 jicki@qq.com]
```


---

> pick 函数

* pick 函数

  * 提取 map 中的 key/value 值 组成一个新的 map.


```yaml
pick: |
        {{- $mp1 := dict "name" "jicki" "age" 12 "email" "jicki@qq.com" }}
        {{ pick $mp1 "name" "email" }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/pick.yaml
---
# Source: myapp/templates/pick.yaml
pick: |
        map[email:jicki@qq.com name:jicki]

```


---

> omit 函数

* omit 函数

  * 删除 map 中指定的 key 值.

```yaml
omit: |
        {{- $mp1 := dict "name" "jicki" "age" 12 "email" "jicki@qq.com" }}
        {{ omit $mp1 "age" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/omit.yaml
---
# Source: myapp/templates/omit.yaml
omit: |
        map[email:jicki@qq.com name:jicki]

```


---

> merge 函数

* merge 函数

  * 合并指定的两个 map 到指定的新 map 中 相同的 key 会取`前面`的覆盖`后面`的.

```yaml
merge: |
        {{- $mp := dict }}
        {{- $mp1 := dict "mp1" "mmm" "mp22" 12 "mp3" "jicki@qq.com" }}
        {{- $mp2 := dict "mp1" "mmmm" "mp222" 20 "mp4" "tom@qq.com" }}
        {{ merge $mp $mp1 $mp2 }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/merge.yaml
---
# Source: myapp/templates/merge.yaml
merge: |
        map[mp1:mmm mp22:12 mp222:20 mp3:jicki@qq.com mp4:tom@qq.com]

```


---

> mergeOverwrite 函数

* mergeOverwrite 函数

  * 合并指定的两个 map 到指定的新 map 中 相同的 key 会取`后面`的覆盖`前面`的.


---


> genPrivateKey 函数

* genPrivateKey 函数

  * 获取 Private Key 私钥密文.


```yaml
{{ $key := genPrivateKey "rsa" }}
key: {{ replace "\n" "" $key }}
```

* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/genPrivateKey.yaml
---
# Source: myapp/templates/genPrivateKey.yaml
key: -----BEGIN RSA PRIVATE KEY-----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-----END RSA PRIVATE KEY-----

```

---

> genCA 函数

* genCA 函数

  * 生成一个 CA 证书


```yaml
{{ $ca := genCA "jicki CA" 365 }}
ca.crt: {{ $ca.Cert | b64enc |quote }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/genCA.yaml
---
# Source: myapp/templates/genCA.yaml
ca.crt: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSUM4ekNDQWR1Z0F3SUJBZ0lSQVB2bkxzRng0V0Jwb3ppSWtXMXhpbzR3RFFZSktvWklodmNOQVFFTEJRQXcKRXpFUk1BOEdBMVVFQXhNSWFtbGphMmtnUTBFd0hoY05NakF3T0RJMk1UQXhPVEF5V2hjTk1qRXdPREkyTVRBeApPVEF5V2pBVE1SRXdEd1lEVlFRREV3aHFhV05yYVNCRFFUQ0NBU0l3RFFZSktvWklodmNOQVFFQkJRQURnZ0VQCkFEQ0NBUW9DZ2dFQkFMbUM0Yk45VFRPTi9TdGZqL0hyRGZsamw4N2M2QUxSRGJtV0VqalNaaVliZ0hkaCt5UXoKRnM4dG5PQXpQM255WTBHdmgveGZFeGhMY25Hb0dxd2pQVXhEb2VxdmRmNmE5UFB0THdkZmxrQ3A5UzRNSTd0QgpGbXBMTVNxTW1BcVRSUEY2ck1QNVI5VStQRGI4WHVkcHVLVjcrSHV4aE1DaGRpSjhibS9Fd3ovV2tNMUF5VFlmCnV4TVkxQU44aGlKMXZybXkrb0J6U0VFV1hHZmlUdTR4UGs3NmtncmpjU1BDNlJXVjh3NXdlZStNYURtbnRLMWIKTGs1dWdvb2wvQjlvRHJIdG43OEpRUXExM0xZRkE1TlIzVHVDREFzcjgvSk9kUm8zSE12ZmxlQUI5aDFrdzZ5eQo1ak9xajFHcTNSbUIwOG1SeWJVZ0V1c2VRczVqbmhYck9JVUNBd0VBQWFOQ01FQXdEZ1lEVlIwUEFRSC9CQVFECkFnS2tNQjBHQTFVZEpRUVdNQlFHQ0NzR0FRVUZCd01CQmdnckJnRUZCUWNEQWpBUEJnTlZIUk1CQWY4RUJUQUQKQVFIL01BMEdDU3FHU0liM0RRRUJDd1VBQTRJQkFRQTRmbW91aCtRYlBMbXQxdGhIY1lTZ2VuZS9GeDhBUm5Vegp5YTdHS00yVTErbVppMEtGcjVpcEFJNXdTWnBwN001UC8xZ0doeGk1TTVUQVlPZ1Q2OGtNNXVnTjUzeWI1WnhxCjFwMndocktMOTg1bTNzT25Ka0kxRWF4THZteGxsU0hyaFlVZldiTXJRNkYvaXpJSGNaYmtwSVNyZy81TzZLSXoKOU5Gc2ZBNmZQZVBPTGdpMUdtOE1Ua2xyWVoxUTBZb3YvMzI2SFlpSUd4bmZtOURZZTMydEZzNnRQSHFnOXk5RgpuaTlDWmh4SFpCalc1QmVoTWQ5ZHEwTWk1d2U4MWhFYk1TTC9NRy84NktuNzRwWDBEazV4Mnh0VHRtdlVpNzJwCkM4ZUJZWVN1bmx6TTlnRC9JQ2hBcUxvMmJKZE4rVTd3dHJTMndZYnBwRENRVHpUNDU5VUoKLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="

```


---

> genSignedCert 函数

* genSignedCert 函数

  * 生成一个带签名的 证书.



```yaml
{{ $ca := genCA "jicki CA" 365 }}
{{ $cert := genSignedCert "jicki.cn" nil (list "jicki.cn" "www.jicki.cn") 365 $ca }}
tls.crt: {{ $cert.Cert | b64enc | quote }}

tls.key: {{ $cert.Key | b64enc | quote }}

```


* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/genSignedCert.yaml
---
# Source: myapp/templates/genSignedCert.yaml
tls.crt: "LS0tLS1CRUdJTiBDRVJUSUZJQ0FURS0tLS0tCk1JSURFakNDQWZxZ0F3SUJBZ0lRTVplNjRFbUlSK0NBVzYwbElQK2M4akFOQmdrcWhraUc5dzBCQVFzRkFEQVQKTVJFd0R3WURWUVFERXdocWFXTnJhU0JEUVRBZUZ3MHlNREE0TWpZeE1ESTFOVEZhRncweU1UQTRNall4TURJMQpOVEZhTUJNeEVUQVBCZ05WQkFNVENHcHBZMnRwTG1OdU1JSUJJakFOQmdrcWhraUc5dzBCQVFFRkFBT0NBUThBCk1JSUJDZ0tDQVFFQXRtL0YxUHVXYVdyczR3dEdYaGorTUVlcDZjQi91aDhDczE1SmJIVFJQc0Jac2cxM2NXdmgKV1JiSExCSHhza1FQR3NFZ1FxUTlIMVRiT0FobDJlYmdPc2hjN3czbjBhY202a3pWWDhnSDFXVkg3V0VOK1g5eQpWRG9sditPVTBJdGxDVzdTV0FKdDBIbUd4UGhxTWhZMVdVeGs2R2VUZmNkVGdKR054L0VmVjk2MjZYQTJ3QmxICnRMeGltQ1RGSVUyUDZZbEVscWxVZk5WU0g0YzZmUmhhSEhxeFhJQ04zZm8wTUw5aDg2VHRFNWVtMnJlalRMNmgKTGhsK2JkaG9sVG93Q1Y0QXhrWTB1cDkzUlN4aUNXYk1MTnM4c1RNRHArbzNNQmNFOXUyZU40VjRvTHY5VWtQRwpCdnM4V3QybHBLUjc3a0VqZ1luZEZ1YkdvSE41YWVhVVF3SURBUUFCbzJJd1lEQU9CZ05WSFE4QkFmOEVCQU1DCkJhQXdIUVlEVlIwbEJCWXdGQVlJS3dZQkJRVUhBd0VHQ0NzR0FRVUZCd01DTUF3R0ExVWRFd0VCL3dRQ01BQXcKSVFZRFZSMFJCQm93R0lJSWFtbGphMmt1WTI2Q0RIZDNkeTVxYVdOcmFTNWpiakFOQmdrcWhraUc5dzBCQVFzRgpBQU9DQVFFQU1OR29uNGJBSGpicjFmRFVmTVRNRGZ6MDdvb29mSFNCWjhONWFZMjY1RnNKZnpWajFWWlU0cHdiClVoU2QzN21oclJabUpjNXZtbEJNZm1aamx0Qmc1ZzhGeGNVZXZvV3hscXFFWUtZcWQzaFZwUzB0OFBkU3Zsa2oKMFk4L3Q0ekxKRlBUV2dpL1Fwbi8vUGN4OGJIWEVHM1IvQWlWR0VaQ0dFb1pNUmpMQzJocGJsQXFGV3loYStCNgpVK1puMk16Q1lnZ3NyZTZsTk5pMW9yRWRJV2xBTE53KzZoaGxBUkFKc09iZklTYzBuaUN2ODB3TzAvQ2Y1dVl3CksxdXZqcHdPZG1VeFJDZmxlMEZ6eXI5RUNJSE1rcks4dE9GM29va0N4bW0zUm91OHVaYS9qVXIvakFHVWtuZ2IKNmE0NGE5eGw1dGNvbU1PdG1aZEZMV0hWemFjZk13PT0KLS0tLS1FTkQgQ0VSVElGSUNBVEUtLS0tLQo="

tls.key: "LS0tLS1CRUdJTiBSU0EgUFJJVkFURSBLRVktLS0tLQpNSUlFb2dJQkFBS0NBUUVBdG0vRjFQdVdhV3JzNHd0R1hoaitNRWVwNmNCL3VoOENzMTVKYkhUUlBzQlpzZzEzCmNXdmhXUmJITEJIeHNrUVBHc0VnUXFROUgxVGJPQWhsMmViZ09zaGM3dzNuMGFjbTZrelZYOGdIMVdWSDdXRU4KK1g5eVZEb2x2K09VMEl0bENXN1NXQUp0MEhtR3hQaHFNaFkxV1V4azZHZVRmY2RUZ0pHTngvRWZWOTYyNlhBMgp3QmxIdEx4aW1DVEZJVTJQNllsRWxxbFVmTlZTSDRjNmZSaGFISHF4WElDTjNmbzBNTDloODZUdEU1ZW0ycmVqClRMNmhMaGwrYmRob2xUb3dDVjRBeGtZMHVwOTNSU3hpQ1diTUxOczhzVE1EcCtvM01CY0U5dTJlTjRWNG9MdjkKVWtQR0J2czhXdDJscEtSNzdrRWpnWW5kRnViR29ITjVhZWFVUXdJREFRQUJBb0lCQUZBQ2VSck5NOHdMenRSTQpMNUk1RjlHSXZHWDl2SWFkN3d0SFFLQkdJemFJR1U1VFJaMENtUlAvUDE1K2lDZU1YYXQ0STNQV244L0w0VkNUCnJrZUFUN3E0QUxuK3VUcGpPbGZyVm5EcFF6WTljdXdTY3BTSFpsYTJJYlFrVlRHWTBMandWMk90dlFkL0pMSGgKMklFYTZFNi9pRW04a3h6SWZFQ1lsVHVvN2Z3VXZISlZWUVpWb0hjV2hRcU00SktIZWlMRDQrV25jVkp1YlFqMwpGTXdySi9aN3dZN1NTL1dZc1dmRitLc2hjaVp1ZUc5YVNwZkJjdFZ5Y3pqdSs2OG5qMDJCaGFvZlQwME9NR0VTCkFNKzZHRDhOVXloeEtpTDZEWDBKcHNraXlYQ3kzR3BkTkpBa0ZpUWZuanlNL0pzQkd0N215Umg4YURGSm1PQkwKOEw5ZVp1RUNnWUVBeEwxemNlcXdza3A5TVYyY1BpajJka09hOHozYVZaOEtXTlN1UkJrY1I4QThiMG5qR25DMwpDaDQ3YkJDR3JEMGdxK00rcFBYdE9KSDRPdWJnWVcxdlloM2RVSkNIVnAwVmc1eVJlWG11ZkZ5VHZrZDcvYUZPCkE1aEkzU0czY2RIa0NndFNUN3NpQ1FVSUNKWlJxSElEMGtRYW9yaHM3MUk4eW00OWpoQ0xNVWtDZ1lFQTdXTmoKeW52SVFPK3lyVk9yU2hzcVhUc2pTaWQ5QzBpYkUrQXlOZ0tDUGpQemd2c0RaRGNEbEJoVUNhK25UTlBTcEVUcgpnQW80QzYrV3ZTV0NMMlV2VGcvVnp6RG96bDFueGIzYWQvYTEzeDA4OGoxSTZZWFExalJBb0xFRW9PY2lGdzNqCjBXdGdMRExWTTJqMHVSaWoyajRQYXFaVHpuMTdocExUR1FMMzVTc0NnWUFMUytGOEVnQ3hUQXVpTVFET3BPVjUKNXVuWHU1NTB1aHdLKzdOQjM3czY5M1BBNUJveEkzV3ZGQXRQYWllQmJrVVkrWVJZVG5LZmcrb2YzNi9VaUVjVAorQ2tEL2poM0phL2RqYmpnbzdiOEZ3aTRyVHdXVlJPNHF4N0w2NnF2MDJCbm56ekxyVEFJR296YWlWOEk3L3IrCk1NRGl4UG9rUjdHTDRnYVF5S3hsV1FLQmdDSjYwRERGMytWR3E0NHZXKzdNbVUrbldrM1lCSHFTRml4QjRTa2wKSGlQSXlmTFpZTG02bitOdjBTMEMvV3JVVFlFY25aUWdaOW1TckhOV3NsME45bHdCUXMzd1RiQkRzdUh1M0grVwpMdjUwTWJrQm04aUhiamplcUJCdkJid1ZOa2RnOWhraDNuc3MrdmlYb3d3TGZ5a2c0SDVlSUVnYXc4bGRKQm82CjZ5UzNBb0dBZGIvUFNyNzRHK3YxU0luNzlaMUJiWjBIbkY5RVNUMWVVL1UraXF4cWhZcU9IYzV3aVgySEM3MVUKbmhESk1OQmY0NVJOWUMwMG1FTTFMalRsUFhuQXFDSDJYdW9zTG9sbHNncDh1SkNOblZHQS92UWIxMmtLdWgvbgpBaGM0SithMnhtZWY0NTJ0TkFsWDVCUjhvZHlpTDNSMDZZa1ZadWFjSmJCY3JZZnpuaW89Ci0tLS0tRU5EIFJTQSBQUklWQVRFIEtFWS0tLS0tCg=="

```


---

> derivePassword 函数

* derivePassword 函数

  * 生成一个指定类型的密码


```yaml
derivePassword: {{ derivePassword 20 "medium" "123456" "jicki" "jicki.cn" }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/derivePassword.yaml
---
# Source: myapp/templates/derivePassword.yaml
derivePassword: Foy3$Xox

```


---


> uuidv4 函数

* uuidv4 函数

  * 生成一个 uuid .


```yaml
uuidv4: {{ uuidv4 }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/uuidv4.yaml
---
# Source: myapp/templates/uuidv4.yaml
uuidv4: 894a3cce-1f72-4535-a4cf-67a563a9e130

```

---

> semver 函数

* semver 函数

  * 构造一个基于 semver 的版本号. 

  * semver 是 语义化版本 规范 的一个实现 . 

    * 版本号格式:  主版本号[MAJOR].次版本号[MINOR].修订号[PATCH]. 

      * 主版本号:  当做了不兼容的 API 修改.
      * 次版本号:  当做了向下兼容的功能性新增.
      * 修订号:    当做了向下兼容的问题修正.

```yaml
semver: {{ semver "1.2.0" }}

```

* 运行 template

```shell

root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/semver.yaml
---
# Source: myapp/templates/semver.yaml
semver: 1.2.0

```

---

> semverCompare 函数

* semverCompare 函数

  * semver 版本的对比 比较. 


```yaml
{{- $semversion := "1.2.0" }}
{{- if semverCompare ">=1.9.0" $semversion }}
api: "v2"
{{- else }}
api: "v1"
{{- end }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/semverCompare.yaml
---
# Source: myapp/templates/semverCompare.yaml
api: "v1"

```


---

> fail 函数

* fail 函数

  * 抛出异常, 输出自定义的错误. 类似于 go 语言的 panic .


```yaml
{{ " my error to failed " | fail }}
```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/fail.yaml
Error: template: myapp/templates/fail.yaml:1:28: executing "myapp/templates/fail.yaml" at <fail>: error calling fail:  my error to failed

```


---


> regexMatch 函数

* regexMatch 函数

  * 正则表达式


```yaml
regexMatch: |
        {{- $IP := "127.0.0.1" }}
        {{- if (regexMatch `^[0-9]{1,3}(\.[0-9]{1,3}){3}$` $IP) }}
        IP: {{ $IP }}
        {{- end }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/regexMatch.yaml
---
# Source: myapp/templates/regexMatch.yaml
regexMatch: |
        IP: 127.0.0.1
```


---


> regexFind 函数

* regexFind 函数

  * 使用 正则表达式 匹配字符串中的内容.



---


> regexFindAll 函数

* regexFindAll 函数

  * 使用 正则表达式 匹配所有字符串中的内容 输出 N 条数 (-1) 为匹配所有 .



```yaml
regexFindAll: {{ regexFindAll "\\d+" "123abc432hhhs9982sahc3hc245" -1 }}

```


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/regexFindAll.yaml
---
# Source: myapp/templates/regexFindAll.yaml
regexFindAll: [123 432 9982 3 245]

```



---

> regexReplaceAll 函数

* regexReplaceAll 函数

  * 用 正则表达式 匹配所有字符串中的内容 并替换成 指定的 字符串.


```yaml
regexReplaceAll: |
        {{ regexReplaceAll "\\d+" "123abc432hhhs9982sahc3hc245" "==" }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/regexReplaceAll.yaml
---
# Source: myapp/templates/regexReplaceAll.yaml
regexReplaceAll: |
        ==abc==hhhs==sahc==hc==

```



---

> regexSplit 函数

* regexSplit 函数

  * 用 正则表达式 匹配所有字符串中的内容 并进行字符串分割 返回一个 list.



```yaml
regexSplit: {{ regexSplit "\\d+" "123abc432hhhs9982sahc3hc245" -1 }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/regexSplit.yaml
---
# Source: myapp/templates/regexSplit.yaml
regexSplit: [ abc hhhs sahc hc ]

```



---

> urlParse 函数

* urlParse 函数

  * url 解析, 返回 url 的信息, 写入到 map 中.


```yaml
urlParse: |
        {{ urlParse "https://jicki.cn" }}

``` 


* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/urlParse.yaml
---
# Source: myapp/templates/urlParse.yaml
urlParse: |
        map[fragment: host:jicki.cn hostname:jicki.cn opaque: path: query: scheme:https userinfo:]

```



---

> urlJoin 函数

* urlJoin 函数

  * 将 map 中的 信息 组合成一个 url .


```yaml
urlJoin: |
        {{- $UrlMap := dict "scheme" "https" "host" "www.jicki.cn" }}
        Url: {{ urlJoin $UrlMap }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/urlJoin.yaml
---
# Source: myapp/templates/urlJoin.yaml
urlJoin: |
        Url: https://www.jicki.cn
```



---

> define 函数

* define 函数

  * 定义一个模板 template .


```yaml

{{- define "mytemplate.home" }}
  home:
    path: "/"
    title: "My Home"
    date: {{ now | htmlDate }}
{{- end }}
```


```yaml
define: |
        {{- template "mytemplate.home" }}
```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/define.yaml
---
# Source: myapp/templates/define.yaml
define: |
  home:
    path: "/"
    title: "My Home"
    date: 2020-08-27

```

---

> Files 函数

* Files 函数

  * 操作文件.


```yaml

Files: |
        {{- $file := .Files }}
        {{- range list "README" }}
        {{ . }}: |-
                {{ $file.Get . }}
        {{- end }}

```

* 运行 template

```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/files.yaml
---
# Source: myapp/templates/files.yaml
Files: |
        README: |-
                myapp by jicki

```



---

> fromJson 函数

* fromJson 函数

  * 将 Json 格式的内容转换成一个 对象 进行操作.



---

> fromYaml 函数

* fromYaml 函数

  * 将 YAML 格式的内容转换成一个 对象 进行操作.



---

> toToml 函数

* toToml 函数

  * 将 其他格式内容 转换成一个 Toml 格式.


---

> toYaml 函数

* toYaml 函数

  * 将 其他格式内容 转换成一个 Yaml 格式.

---


> include 函数

* include 函数

  * 导入模板文件. 引入模板内容.


```yaml
{{- define "mytemplate.home" }}
  home:
    path: "/"
    title: "My Home"
    date: {{ now | htmlDate }}
{{- end }}

```


```yaml
include: |
        {{- include "mytemplate.home" . }}

```


* 运行 template


```shell

root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/include.yaml
---
# Source: myapp/templates/include.yaml
include: |
  home:
    path: "/"
    title: "My Home"
    date: 2020-08-27

```



---


> tpl 函数

* tpl 函数

  * 将 模板中引用 模板变量的 内容输出.


* `values.yaml`

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
http: "{{ .Values.containerPort.http }}"
https: "{{ .Values.containerPort.https }}"
```

```yaml
notTpl: |
        http: {{ .Values.http }}
        https: {{ .Values.https }}
tpl: |
        http: {{ tpl .Values.http . }}
        https: {{ tpl .Values.https . }}
```

* 运行 template


```shell
root@kubernetes:/opt/helm/myapp# helm template . --show-only templates/tpl.yaml
---
# Source: myapp/templates/tpl.yaml
notTpl: |
        http: {{ .Values.containerPort.http }}
        https: {{ .Values.containerPort.https }}
tpl: |
        http: 80
        https: 443

```




