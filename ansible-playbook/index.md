# ansible playbook 从入门到放弃


{{< figure src="/img/posts/ansible/ansible.png" >}}


# 前言

**运维自动化的发展历程以及技术应用**

1. 本地部署 `On-Premises` 如下都需要自己 `部署/配置` 以及维护。

  * `Application`
  * `Data`
  * `Runtime`
  * `Middleware`
  * `OS`
  * `Virtualization`
  * `Servers`
  * `Storage`
  * `NetWorking`

2. 基础设施既服务 ( 如 阿里云 ) `IaaS` - `Infrastructure as a Service` 如下需要自己 `部署/配置` 以及维护。

  * `Application`
  * `Data`
  * `Runtime`
  * `Middleware`
  * `OS`

3. 平台既服务 ( 如 阿里云- ACK 容器服务 ) `PaaS` - `Platform as a Service` 如下需要自己 `部署/配置` 以及维护。

  * `Application`
  * `Data`

4. 软件既服务 ( 如 各类软件 微信、钉钉、邮箱 ) `SaaS` - `Software as a Service` 所有的服务软件都不需要自己维护, 直接使用既可。


# ansible

## ansible 简介

> ansible 是基于 `python2-paramiko` 模块开发的自动化运维工具。 实现了批量系统配置, 批量程序部署, 批量运行命令等功能。`ansible` 是基于模块工作的, 本身没有批量部署的能力。真正具有批量部署的是 `ansible` 所运行的模块, `ansible` 只是提供了一种框架。

**ansible 集合了众多运维工具（ pupet、cfengine、chef、func、fabric、saltstack ）的优点**

* ansible 发展史

  * ansible 作者是 `Michael DeHaan` 同时他也是 `Cobbler 与 Func` 作者。

  * 2012-03-09 发布 0.0.1 版本。

  * 2015-10-17 被 `Red Hat` 收购。



* ansible 特性

  * 基于 Python 开发

  * 模块化: 调用特定的模块(如: Paramiko、PyYAML、jinja2 等), 完成特定的任务。

  * 支持自定义模块

  * 部署简单, 基于`Linux`内置的 `Python` 、`Open-SSH` 和另一个 `agentless` 组件.

  * 支持 `PlayBook` 编排任务

  * 幂等性: 任务重复执行等于只执行一次, 不会重复执行多次相同命令。

  * 无需代理不依赖PKI.

  * 支持多语言模块编写.

  * `YAML`格式编排任务,支持丰富的数据结构.
 

## ansible 架构

{{< figure src="/img/posts/ansible/ansible-jg.png" >}}



{{< figure src="/img/posts/ansible/ansible-yl.png" >}}


