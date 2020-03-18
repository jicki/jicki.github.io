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

* `kube-apiserver` 集群控制入口, 提供了 `HTTP Rest` 接口的关键服务进程，是 `Kubernetes` 里所有资源的增删改查等操作的唯一入口, 也是集群控制的入口。

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





## Pod 的生命周期

* Pod对象自从其创建开始至其终止退出的时间范围称为其生命周期。

  1. 创建主容器（main container）为 `必需`的操作。

  2. 初始化容器（init container）。

  3. 容器启动后钩子（post start hook）。

  4. 容器的存活性探测（liveness probe）。

  5. 就绪性探测（readiness probe）。

  6. 容器终止前钩子（pre stop hook）

  7. 其他Pod的定义操作。

![图2][2]



### 初始化容器(init container)

* 初始化容器（init container）即应用程序的主容器启动之前要运行的容器，常用于为主容器执行一些预置操作，它们具有两
种典型特征。

  1. 初始化容器必须运行完成直至结束，若某初始化容器运行失败，那么kubernetes需要重启它直到成功完成。（注意：如果pod的`spec.restartPolicy`字段值为 `Never`，那么运行失败的初始化容器不会被重启。）

  2. 每个初始化容器都必须按定义的顺序串行运行。


### pod phase

* Pod 的 status 属性是一个 `PodStatus` 对象，拥有一个 phase 字段。它简单描述了 Pod 在其生命周期的阶段。

|pod phase|描述|
|-|-|
|挂起(Pending)|kubernetes 通过`apiserver`创建了pod 资源对象并存入etcd中, 但它尚未被调度完成, 或者仍处于从仓库下载镜像的过程中|
|运行中(Running)| Pod已经被调度至某节点，并且所有容器都已经被kubelet创建完成，至少一个容器正在运行。|
|成功(Succeeded)| Pod中的所有容器都已经成功并且不会被重启。|
|失败(Failed)| Pod中的所有容器都已终止了，并且至少有一个容器是因为失败终止。即容器以非0状态退出或者被系统禁止。|
|未知(Unknown)| `ApiServer` 无法正常获取到Pod对象的状态信息，通常是由于无法与所在工作节点的kubelet通信所致。|



### Pod conditions

* Pod 的 status 属性是一个 `PodStatus` 对象, 里面包含 PodConditions 数组，代表 Condition 是否通过。

* PodCondition 属性描述

|字段|描述|
|-|-|
|lastProbeTime|最后一次探测 Pod Condition 的时间戳。|
|lastTransitionTime|上次 Condition 从一种状态转换到另一种状态的时间。|
|message|上次 Condition 状态转换的详细描述。|
|reason|Condition 最后一次转换的原因。|
|status|Condition 状态类型，可以为 True False Unknown|
|type|Condition类型(PodScheduled, Ready, Initialized, Unschedulable, ContainersReady)|

* Condition Type 说明

  * `PodScheduled` Pod 已被调度到一个节点 

  * `Ready` Pod 能够提供请求，应该被添加到负载均衡池中以提供服务

  * `Initialized` 所有 init containers 成功启动

  * `Unschedulable` 调度器不能正常调度容器，例如缺乏资源或其他限制

  * `ContainersReady` Pod 中所有容器全部就绪


### Container probes

* Probe 是在容器上 kubelet 的定期执行的诊断，kubelet 通过调用容器实现的 Handler 来诊断。

  * `Success` 容器诊断通过。

  * `Failure` 容器诊断失败。

  * `Unknown` 诊断失败，因此不应采取任何措施。

* Handlers 包含如下三种

  * `ExecAction` 在容器内部执行指定的命令，如果命令以状态代码 0 退出，则认为诊断成功。

  * `TCPSocketAction` 对指定 IP 和端口的容器执行 TCP 检查，如果端口打开，则认为诊断成功。

  * `HTTPGetAction` 对指定 IP + port + path 路径上的容器的执行 HTTP Get 请求。如果响应的状态代码大于或等于 200 且小于 400，则认为诊断成功。


* kubelet 可以选择性地对运行中的容器进行两种探测器执行和响应。

  * `livenessProbe`  存活性探测, 探测容器是否正在运行，如果活动探测失败，则 kubelet 会杀死容器，并且容器将受其 重启策略 的约束。如果不指定活动探测, 默认状态是 Success。


  * `readinessProbe`  就绪性探测, 探测容器是否已准备好为请求提供服务，如果准备情况探测失败，则控制器会从与 Pod 匹配的所有服务的端点中删除 Pod 的 IP 地址。初始化延迟之前的默认准备状态是 `Failure`, 如果容器未提供准备情况探测，则默认状态为 Success。


* 以下为 Nginx 应用的两种探针配置示例

```shell
apiVersion: apps/v1
kind: Deployment 
metadata: 
  name: nginx-dm
spec: 
  replicas: 2
  selector:
    matchLabels:
      name: nginx
  template: 
    metadata: 
      labels: 
        name: nginx 
    spec: 
      containers: 
        - name: nginx 
          image: nginx:alpine 
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
              name: http
          # readinessProbe - 检测pod 的 Ready 是否为 true
          readinessProbe:
            tcpSocket:
              port: 80
            # 启动后5s 开始检测
            initialDelaySeconds: 5  
            # 检测 间隔为 10s
            periodSeconds: 10
          # livenessProbe - 检测 pod 的 State 是否为 Running
          livenessProbe:
            httpGet:
              path: /
              port: 80
            # 启动后 15s 开始检测
            # 检测时间必须在 readinessProbe 之后
            initialDelaySeconds: 15
            # 检测 间隔为 20s
            periodSeconds: 20
```



### Pod RestartPolicy (重启策略)

* PodSpec 中有一个`restartPolicy`字段，可能的值为Always、OnFailure和Never。默认为Always。restartPolicy适用于Pod中的所有容器。而且它仅用于控制在同一节点上重新启动Pod对象的相关容器。

* 首次需要重启的容器，将在其需要时立即进行重启，随后再次需要重启的操作将由kubelet延迟一段时间后进行，且反复的重启操作的延迟时长依次为10秒、20秒、40秒... 300秒是最大延迟时长。

* Pod，一旦绑定到一个节点，Pod对象将永远不会被重新绑定到另一个节点，它要么被重启，要么终止，直到节点发生故障或被删除。


### Pod 创建过程

* 创建一个 pod 的过程

  1. 用户通过kubectl或其他API客户端提交了Pod Spec给API Server。

  2. API Server尝试着将Pod对象的相关信息存入etcd中，待写入操作执行完成，API Server即会返回确认信息至客户端。

  3. API Server开始检测etcd中的状态变化。

  4. 所有的kubernetes组件均使用`watch`机制来跟踪检查API Server上的相关的变动。

  5. kube-scheduler（调度器）通过其`watcher`觉察到API Server创建了新的Pod对象但尚未绑定至任何工作节点。

  6. kube-scheduler（调度器）为Pod对象挑选一个工作节点并将结果信息更新至API Server。

  7. 调度结果信息由API Server更新至etcd存储系统，而且API Server也开始反映此Pod对象的调度结果。

  8. Pod被调度到的目标工作节点上的kubelet尝试在当前节点上调用Docker启动容器，并将容器的结果状态返回送至API Server。 

  9. API Server将Pod状态信息存入etcd系统中。 

  10. 在etcd确认写入操作成功完成后，API Server将确认信息发送至相关的kubelet，事件将通过它被接受。



























  [1]: http://jicki.me/img/posts/kubernetes/kubernetes.png

  [2]: http://jicki.me/img/posts/kubernetes/pod.png



