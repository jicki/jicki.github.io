---
layout: post
title: kubernetes 基础知识
categories: kubernetes
description: kubernetes 基础知识
keywords: docker,kuebrnetes
catalog:    true
header-img: "img/pexels/triangular.jpeg"
---

# Kubernetes

> Kubernetes 是 Google 基于 Borg 开源的容器编排调度，用于管理容器集群自动化部署、扩容以及运维的开源平台。作为云原生计算基金会 CNCF（Cloud Native Computing Foundation）最重要的组件之一（CNCF 另一个毕业项目 Prometheus ），它的目标不仅仅是一个编排系统，而是提供一个规范，可以让你来描述集群的架构，定义服务的最终状态，Kubernetes 可以帮你将系统自动地达到和维持在这个状态，Kubernetes 也可以对容器(Docker)进行集群管理和服务编排（Docker Swarm 类似的功能）,对于大多开发者来说，以容器的方式运行一个程序是一个最基本的需求，跟多的是 Kubernetes 能够提供路由网关、水平扩展、监控、备份、灾难恢复等一系列运维能力。


* 本文用于科普 `Kubernetes` 的一些基础知识以及概念。


## Kubernetes 核心组件

![图1][1]


* `Kubernetes` 集群由两种角色组成:

  * `Master` 集群调度节点, 管理集群。(kube-apiserver, kube-controller-manager, kube-scheduler, etcd)

  * `Node` 引用程序实际运行的工作节点。(kubelet, kube-proxy)



### Master 端

* `kube-apiserver` 集群控制入口, 提供了 `HTTP Rest` 接口的关键服务进程，是 `Kubernetes` 里所有资源的增删改查等操作的唯一入口, 也是集群控制的入口进行。

  * 提供集群管理的 REST API 接口，包括认证授权、数据校验以及集群状态变更等。

  * 提供其他模块之间的数据交互和通信的枢纽（其他模块通过 API Server 查询或修改数据，只有 API Server 才直接操作 etcd）



* `kube-controller-manager` 服务运行控制器, 处理常规任务的后台线程 比如故障检测、自动扩展、滚动更新等。kube-controller-manager 由一系列的控制器组成包括如下控制器

  * `Replication Controller`

  * `Node Controller`

  * `CronJob Controller`

  * `Daemon Controller`

  * `Deployment Controller`

  * `Endpoint Controller`

  * `Garbage Collector`

  * `Namespace Controller`

  * `Job Controller`

  * `Pod AutoScaler`

  * `RelicaSet`

  * `Service Controller`

  * `ServiceAccount Controller`

  * `StatefulSet Controller`

  * `Volume Controller`

  * `Resource quota Controller`  


* `kube-scheduler` 负责 Pod 资源调度。监视新创建没有分配到Node的Pod, 为Pod选择一个Node。调度方式:

  * 公平调度

  * 资源高效利用

  * QoS

  * affinity 和 anti-affinity (约束)

  * 数据本地化（data locality）

  * 内部负载干扰（inter-workload interference）

  * deadlines



* `etcd` 用于共享配置和服务发现，存储 Kubernetes 集群所有的网络配置和对象的状态信息。


### Node 端


* `kubelet` 负责 Pod 对应的容器的创建，启动等任务，同时与Master节点密切协作, kubelet 提供了一系列`PodSpecs`集合规范，并确保这些`PodSpecs`中描述的容器运行正常。 kubelet是主要的节点代理，它会监视已分配给节点的pod, 具体功能

  * 安装 Pod 所需的volume。

  * 下载 Pod 的Secrets。

  * 监控 Pod 中运行的 docker（或experimentally，rkt）容器。 

  * 定期执行容器健康检查。

  * 回传 pod 的状态到其他 kubernetes 服务中。

  * 回传 节点 的状态到其他 kubernetes 服务中。


* `kube-proxy` 实现 Kubernetes SVC 的通信与负载均衡机制的重要组件。通过在主机上维护网络规则并执行连接转发来实现Kubernetes服务抽象。



## Kubernetes 基本对象与术语


### Pod

* Pod 是一组紧密关联的容器集合，它们共享 `PID`、`IPC`、`Network` 和 `UTS namespace`，是 Kubernetes 调度的基本单位。Pod 的设计理念是支持多个容器在一个 Pod 中共享网络和文件系统，可以通过进程间通信和文件共享这种简单高效的方式组合完成服务。



### Label

* Label 是识别 Kubernetes 对象的标签，以 `key/value` 的方式附加到对象上（key最长不能超过63字节，value 可以为空，也可以是不超过253字节的字符串）。 Label 不提供唯一性，并且实际上经常是很多对象（如 Pods）都使用相同的 label 来标志具体的应用。 Label 定义好后其他对象可以使用 Label Selector 来选择一组相同 label 的对象（比如 Service 用 label 来选择一组 Pod）。Label Selector 支持以下几种方式:

  * 等于和不等于, 如app=nginx和env!=production

  * 集合，如env in (production, test)

  * 多个label（它们之间是AND关系），如app=nginx, env=test


### Namespace

* Namespace 是对一组资源和对象的抽象集合，比如可以用来将系统内部的对象划分为不同的项目组或用户组。常见的 pods, services, deployments 等都是属于某一个 namespace 的（默认是default），而 Node, PersistentVolumes 等则不属于任何 namespace。



### ReplicaSet

* ReplicaSet 它的主要作用是确保 `Pod` 以你指定的副本数运行,  如果有容器异常退出, 会自动创建新的 `Pod` 来替代, 而异常多出来的容器也会自动回收。 支持集合式的 `selector`。ReplicaSet 可以独立使用, 但建议使用 `Deployment` 来自动管理 ReplicaSet, 这样就无需担心跟其他机制的不兼容问题（比如 ReplicaSet 不支持 rolling-update 但 Deployment 支持）, 并且 `Deployment` 还支持版本记录、回滚、暂停升级等高级特性。



### Deployment

* Deployment 确保任意时间都有指定数量的 `Pod` 在运行。如果为某个 `Pod` 创建了 Deployment 并且指定3个副本，它会创建3个 Pod，并且持续监控它们。如果某个 Pod 不响应，那么 Deployment 会替换它，保持总数为3。如果之前不响应的 Pod 恢复了，现在就有4个 Pod 了，那么 Deployment 会将其中一个终止保持总数为3。如果在运行中将副本总数改为5，Deployment 会立刻启动2个新 Pod，保证总数为5。Deployment 还支持回滚和滚动升级。

* 创建 Deployment 需要指定:
  
  * Pod 模板 用于配置 pod 生成的属性和副本。

  * Label 标签 Deployment 需要监控的 Pod 的标签。 


### StatefulSet

* StatefulSet 是为了解决有状态服务的问题 (对应 Deployments 和 ReplicaSets是为无状态服务而设计), 其应用场景包括

  * 稳定的持久化存储，即 Pod 重新调度后还是能访问到相同的持久化数据，基于 PVC 来实现。

  * 稳定的网络标志，即 Pod 重新调度后其 `PodName` 和 `HostName` 不变，基于 Headless Service（即没有Cluster IP的Service）来实现  
 
  * 有序部署，有序扩展，即 Pod 是有顺序的，在部署或者扩展的时候要依据定义的顺序依次依次进行（即从0到N-1，在下一个Pod运行之前所有之前的 Pod 必须都是 Running 和 Ready 状态），基于 init containers来实现。

  * 有序收缩，有序删除（即从N-1到0）


### DaemonSet

* DaemonSet 保证在每个Node上都运行一个容器副本，常用来部署一些集群的日志、监控或者其他系统管理应用。


### Service

* Service 是应用服务的抽象，通过 labels 为应用提供负载均衡和服务发现。匹配 labels 的Pod IP 和端口列表组成 endpoints，由 kube-proxy 负责将服务 IP 负载均衡到这些endpoints 上。每个 Service 都会自动分配一个 cluster IP（仅在集群内部可访问的虚拟地址）和 DNS 名，其他容器可以通过该地址或 DNS 来访问服务，而不需要了解后端容器的运行。



### Job

* Job 负责批量处理短暂的一次性任务 (short lived one-off tasks)，即仅执行一次的任务，它保证批处理任务的一个或多个Pod成功结束。

* Kubernetes 支持以下几种 Job

  * 非并行 Job：通常创建一个 Pod 直至其成功结束

  * 固定结束次数的 Job：设置 `.spec.completions`, 创建多个 Pod，直到 `.spec.completions` 个 Pod 成功结束

  * 带有工作队列的并行 Job：设置 `.spec.Parallelism` 但不设置 `.spec.completions`，当所有Pod结束并且至少一个成功时，Job 就认为是成功



### CronJob

* CronJob 即定时任务，就类似于 Linux 系统的 crontab，在指定的时间周期运行指定的任务。


### Horizontal Pod Autoscaler（HPA）

* Horizontal Pod Autoscaling 可以根据 CPU 使用率或应用自定义 metrics 自动扩展 Pod 数量（支持replication controller、deployment和replica set）, 从而合理的扩展性能与使用资源。







  [1]: http://jicki.me/img/posts/kubernetes/kubernetes.png




