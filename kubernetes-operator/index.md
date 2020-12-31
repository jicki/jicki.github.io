# Kubernetes Operator 模式


{{< figure src="/img/posts/kubernetes/operator.jpeg" >}}


# Kubernetes Operator

**Operator 是 Kubernetes 的扩展软件，它利用 自定义资源(Custom Resource)管理应用及其组件。 Operator 遵循 Kubernetes 的理念，特别是在控制回路方面。其目的应该是帮助我们更轻松地解决应用程序 Pod 中的某些问题。**

---

> 为什么需要 Operator

在 Kubernetes 中已经提供了很多控制器(Controller), 如: Deployments、Statefulsets、ReplicaSet、DaemonSet 等, 它们负责 管理/维护 Pods 对象的状态。

But!

  如果我们需要对一些专业的服务如: Mysql、Prometheus 等. 进行更好的控制与管理, 我们可能需要更精细, 更具体的领域知识进行操作, 以正确管理我们的应用程序和组件的状态, 但是实际上我们希望服务的 开发人员/用户/管理者 可以更容易, 更透明的进行一些重复的操作与控制。 这时我们就需要使用 Kubernetes Operator.   Operator 可以让 开发人员/用户/管理者 在使用这些专业的服务时, 像使用 Kubernetes 一样 只需要编写约定的内置资源就可以简单的 创建/管理 这些服务.
  

---

> Custom Resource

* `定制资源（Custom Resource）` -  是对 Kubernetes API 的扩展。

  * `定制资源` - 所代表的是对特定 Kubernetes 安装的一种定制。 不过，很多 Kubernetes 核心功能现在都用定制资源来实现，这使得 Kubernetes 更加模块化。

  * `定制资源` - 可以通过动态注册的方式在运行中的集群内或出现或消失，集群管理员可以独立于集群 更新定制资源。一旦某定制资源被安装，用户可以使用 kubectl 来创建和访问其中的对象，就像他们为 pods 这种内置资源所做的一样。

  * `资源（Resource）` - 是 Kubernetes API 中的一个端点， 其中存储的是某个类别的 API 对象 的一个集合。 例如内置的 Pods 资源包含一组 Pod 对象。

---

> Custom Controller

* `定制资源 (Custom Resource)` - 本身而言，它只能用来存取结构化的数据。 

* `控制器` - 负责将结构化的数据解释为用户所期望状态的记录，并持续地维护该状态。

*  所以当 `定制资源` 与 `定制控制器` 相结合时，定制资源就能够提供真正的声明式 API（Declarative API）。使用声明式 API, 你可以 声明 或者设定你的资源的期望状态，并尝试让 Kubernetes 对象的当前状态 同步到其期望状态。



## 如何开发 Operator


![图2][2]



`定制控制器` 支持多种语言开发, 如: Golang、Python 等, 我们只需要实现 Controller 与 Kubernetes API 通信既可。


* 在开发 Kubernetes Operator 时需要遵循 Kubernetes 规则, 其实社区中已经有大量的开源可用的框架. 如:

  * `Kubebuilder` - https://github.com/kubernetes-sigs/kubebuilder

    * 该框架来自 kubernetes-SIG（特殊兴趣小组），成立于2018年7月。它有望提高速度并减少开发人员管理的复杂性。


  * `Operator SDK` - https://sdk.operatorframework.io/

    * 这是 `Red Hat` 的产品, 不仅具有使用 Go语言 构建 `Operator` 的能力, 而且还具有 `Ansible`和 `Helm Charts`的 构建能力。

    * 它与 `CNCF Landscape Operator Framework` 相关联。

      * `CNCF Landscape Operator Framework` - https://landscape.cncf.io/selected=operator-framework


  * `KUDO` - https://kudo.dev/

    * KUDO 号称是使用 声明式方式 构建生产级别的 Kubernetes Operator 服务.


### Docker / Kubernetes

* 在开发 Operator 之前, 我们需要一个 Kubernetes, 如果您没有 Kubernetes，我建议您使用 `Kind`。

  * `Kind` - https://kind.sigs.k8s.io/ -- 它是一种使用 Docker 容器做为 `Node` 运行 Kubernetes 本地集群的工具。


---

> Kubernetes Operator 工作流程

*  如下我们开发一个名为 `PodBuggerTool` 的 Kubernetes Operator . 如下为  Operator PodBuggerTool 工作流程


![图3][3]


---

1. 不断 Watching  Pods 的 事件(events).

2. 当在事件 events 普抓到 创建/更新 的事件时,  对该事件进行处理.

3. 将 `Pod` 的  Labels 与我们使用 `PodBuggerTool` 插入的 Labels 进行比较（基于其CRD）.

4. 如果我们意识到 `Pod` 包含我们感兴趣的任何 Labels，既可进行一些操作.

5. 使用同样由我们的 `PodBuggerTool` 指定的 image，在此 `Pod` 中添加一个新容器（作为临时容器）（在那里，我们可以在其定义中找到 Label 与 image 的相关性）.























  [1]: https://jicki.cn/img/posts/kubernetes/kubernetes-logs.png
  [2]: https://jicki.cn/img/posts/kubernetes/kubernetes-go.png
  [3]: https://jicki.cn/img/posts/kubernetes/operator-workflow.gif
