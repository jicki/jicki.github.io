---
layout: post
title: jekyll mermaid plugin
categories: [jekyll]
description: jekyll mermaid plugin
keywords: jekyll
header-img: "img/pexels/triangular.jpeg"
catalog:    true
tags:
    - jekyll
---

# jekyll mermaid plugin

## jekyll 插件 jekyll-mermaid

安装参见: [jekyll-mermaid](https://github.com/jasonbellamy/jekyll-mermaid/blob/master/readme.md#jekyll-mermaid). 其中 `mermaid.js` 的下载来源参考 [mermaid cdn](https://github.com/knsv/mermaid#cdn), 下载之后本地目录布局:

```
$ tree js/mermaid/
js/mermaid/
├── 7.1.2
│   ├── mermaid.core.js
│   └── mermaid.min.js
└── current -> 7.1.2/
```

之后 `_config.yml` 中的相关配置:

```
# Gems
# from PR#40, to support local preview for Jekyll 3.0
plugins: [jekyll-paginate, jekyll-mermaid]

mermaid:
  src: '/js/mermaid/current/mermaid.min.js'
```

之后在 Post 中使用姿势如下, 注意这里是使用 `{% raw %}{% mermaid  %}, {% endmermaid %}{% endraw %}` 来声明 mermaid block 的. mermaid block 内的语法可以参考 [mermaidjs](https://mermaidjs.github.io/).

{% mermaid %}
sequenceDiagram
    participant Alice
    participant Bob
    Alice->John: Hello John, how are you?
    loop Healthcheck
        John->John: Fight against hypochondria
    end
    Note right of John: Rational thoughts <br/>prevail...
    John-->Alice: Great!
    John->Bob: How about you?
    Bob-->John: Jolly good!
{% endmermaid %}

或者这样:

{% mermaid %}
gantt
        dateFormat  YYYY-MM-DD
        title Adding GANTT diagram functionality to mermaid
        section A section
        Completed task            :done,    des1, 2014-01-06,2014-01-08
        Active task               :active,  des2, 2014-01-09, 3d
        Future task               :         des3, after des2, 5d
        Future task2               :         des4, after des3, 5d
        section Critical tasks
        Completed task in the critical line :crit, done, 2014-01-06,24h
        Implement parser and jison          :crit, done, after des1, 2d
        Create tests for parser             :crit, active, 3d
        Future task in critical line        :crit, 5d
        Create tests for renderer           :2d
        Add to mermaid                      :1d
{% endmermaid %}

也可以这样:

{% mermaid %}
graph TD;
    A[我]-->B[热];
    A-->C[爱];
    B-->D[学习];
    C-->D;
{% endmermaid %}
