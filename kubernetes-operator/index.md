# Kubernetes Operator 模式


{{< figure src="/img/posts/kubernetes/operator.jpeg" >}}


# Operator

**Operator 是 Kubernetes 的扩展软件，它利用 自定义资源(Custom Resource)管理应用及其组件。 Operator 遵循 Kubernetes 的理念，特别是在控制回路方面。**

---

## Custom Resource

* `定制资源（Custom Resource）` -  是对 Kubernetes API 的扩展。

  * `定制资源` - 所代表的是对特定 Kubernetes 安装的一种定制。 不过，很多 Kubernetes 核心功能现在都用定制资源来实现，这使得 Kubernetes 更加模块化。

  * `定制资源` - 可以通过动态注册的方式在运行中的集群内或出现或消失，集群管理员可以独立于集群 更新定制资源。一旦某定制资源被安装，用户可以使用 kubectl 来创建和访问其中的对象，就像他们为 pods 这种内置资源所做的一样。

  * `资源（Resource）` - 是 Kubernetes API 中的一个端点， 其中存储的是某个类别的 API 对象 的一个集合。 例如内置的 Pods 资源包含一组 Pod 对象。


## Custom Controller

* `定制资源 (Custom Resource)` - 本身而言，它只能用来存取结构化的数据。 

* `控制器` - 负责将结构化的数据解释为用户所期望状态的记录，并持续地维护该状态。

*  所以当 `定制资源` 与 `定制控制器` 相结合时，定制资源就能够提供真正的声明式 API（Declarative API）。使用声明式 API, 你可以 声明 或者设定你的资源的期望状态，并尝试让 Kubernetes 对象的当前状态 同步到其期望状态。


