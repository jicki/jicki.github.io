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


* `ansible` 主要组成部分:

  * `ansible playbooks`: 任务剧本(任务集), 通过编排定义`ansible`任务集合的配置文件, 由`ansible` 顺序依次执行, 文件通常是 `JSON`格式的`YML`文件。

  * `Inventory`: `ansible` 管理主机的清单 默认为 `/etc/ansible/hosts` 文件。

  * `Modules`: `ansible` 执行命令的功能模块, 一般为`ansible`内置核心模块, 也可以自定义第三方模块.

  * `plugins`: `ansible` 功能插件, 是功能模块的补充. 如: 连接类型插件、循环插件、变量插件、过滤插件等等.

  * `Api`: 提供第三方程序调用的开放接口.

  * `ansible`: `ansible` 的客户端命令工具. 执行 `ansible` 命令的主要程序.

    * `Ad-Hoc` 用户执行单条或多条的 `ansible` 命令.

    * `ansible playbook` 通过编写编排文件 `ansible` 执行命令集合.

      * `ansible playbook` 执行过程 -> 将编排好的任务写入 `ansible-playbook` 编排文件中 -> 通过 `ansible-playbook` 命令拆分任务集合, 然后按照预定的顺序逐条执行`ansible` 命令.



## ansible 安装

* `ansible` 安装很简单, 也有很多种方式.

  * 安装 `epel` 源以后 执行 `yum -y install ansible`

  * 使用 源码包 进行 安装

  * 通过 `git clone https://github.com/ansible/ansible` 以后进行安装

  * 使用 pip 命令安装




```shell

[root@jicki ~]# ansible --version
ansible 2.9.10
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr  2 2020, 13:16:51) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]

```



