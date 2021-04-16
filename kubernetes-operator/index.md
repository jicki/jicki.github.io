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



---

> 一级资源和二级资源

* 正如前面所说, `Operator` 是 `CRD` 相关的控制器, 它观察并根据配置变化或集群状态变化来采取行动。但实际上控制器需要监测两种资源, 一级资源和二级资源。

  * 对于 `ReplicaSet`, 一级资源是 `ReplicaSet` 本身, 指定运行的Docker镜像以及Pod的副本数。二级资源则是 `Pod`。 当 `ReplicaSet` 的定义发生改变（例如镜像版本或Pod数量）或者其中的Pod发生改变时（例如某个Pod被删除）, 控制器则会通过推出新版本来协调集群状态, 或是对Pod数量扩缩容。

  * 对于 `DaemonSet`, 一级资源是 `DaemonSet` 本身, 而二级资源是 `Pod` 和 `Node`。不同之处在于, 控制器也会监测集群 `Node` 变化, 以便在群集增大或缩小时添加或删除 `Pod`。




## 如何开发 Operator


{{< figure src="/img/posts/kubernetes/kubernetes-go.png" >}}


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

*  如下是一个名为 `PodBuggerTool` 的 Kubernetes Operator . 如下为  Operator PodBuggerTool 工作流程


{{< figure src="/img/posts/kubernetes/operator-workflow.gif" >}}


---

{{< figure src="/img/posts/kubernetes/kubernetes-logs.png" >}}

1. 不断 Watching  Pods 的 事件(events).

2. 当在事件 events 普抓到 创建/更新 的事件时,  对该事件进行处理.

3. 将 `Pod` 的  Labels 与我们使用 `PodBuggerTool` 插入的 Labels 进行比较（基于其CRD）.

4. 如果我们意识到 `Pod` 包含我们感兴趣的任何 Labels，既可进行一些操作.

5. 使用同样由我们的 `PodBuggerTool` 指定的 image，在此 `Pod` 中添加一个新容器（作为临时容器）（在那里，我们可以在其定义中找到 Label 与 image 的相关性）.



## 开发一个 Operator

* 利用 `Operator SDK` 创建一个 Pods 相关的 Operator, 它的作用是 启动一个容器 busybox 执行 Sleep 3600s , 并管理 Pods 数量的扩容和缩容。



> Operator SDK


* 安装 Operator SDK  Version 1.5.0

```bash
$ mkdir -p $GOPATH/src/github.com/operator-framework
$ cd $GOPATH/src/github.com/operator-framework
operator-framework$  git clone https://github.com/operator-framework/operator-sdk
operator-framework$  cd operator-sdk
operator-sdk [master]$ make install
```



---

* 初始化 Operator 项目


```bash
$ mkdir -p jicki/golang/pods-operator
$ cd jicki/golang/pods-operator
pods-operator$ operator-sdk init --domain jicki.cn --skip-go-version-check


Writing scaffold for you to edit...
Writing scaffold for you to edit...
Get controller runtime:
$ go get sigs.k8s.io/controller-runtime@v0.7.2
go: downloading sigs.k8s.io/controller-runtime v0.7.2
go: downloading k8s.io/utils v0.0.0-20200912215256-4140de9c8800
go: downloading k8s.io/component-base v0.19.2
go: downloading k8s.io/apiextensions-apiserver v0.19.2
go: downloading github.com/golang/groupcache v0.0.0-20191227052852-215e87163ea7
go: downloading k8s.io/kube-openapi v0.0.0-20200805222855-6aeccd4b50c6
go: downloading github.com/evanphx/json-patch v0.5.2
go: downloading k8s.io/client-go v1.5.2
Update dependencies:
$ go mod tidy
go: downloading github.com/onsi/ginkgo v1.14.1
go: downloading github.com/onsi/gomega v1.10.2
go: downloading github.com/Azure/go-autorest/autorest v0.9.6
go: downloading golang.org/x/lint v0.0.0-20191125180803-fdd1cda4f05f
go: downloading github.com/Azure/go-autorest/autorest/adal v0.8.2
go: downloading cloud.google.com/go v0.51.0
go: downloading golang.org/x/tools v0.0.0-20200616133436-c1934b75d054
go: downloading github.com/Azure/go-autorest/autorest/mocks v0.3.0
go: downloading github.com/Azure/go-autorest/autorest/date v0.2.0
go: downloading github.com/nxadm/tail v1.4.4
Next: define a resource with:
$ operator-sdk create api
```


```bash
pods-operator$  tree

.
├── config
│   ├── default
│   │   ├── kustomization.yaml
│   │   ├── manager_auth_proxy_patch.yaml
│   │   └── manager_config_patch.yaml
│   ├── manager
│   │   ├── controller_manager_config.yaml
│   │   ├── kustomization.yaml
│   │   └── manager.yaml
│   ├── manifests
│   │   └── kustomization.yaml
│   ├── prometheus
│   │   ├── kustomization.yaml
│   │   └── monitor.yaml
│   ├── rbac
│   │   ├── auth_proxy_client_clusterrole.yaml
│   │   ├── auth_proxy_role_binding.yaml
│   │   ├── auth_proxy_role.yaml
│   │   ├── auth_proxy_service.yaml
│   │   ├── kustomization.yaml
│   │   ├── leader_election_role_binding.yaml
│   │   ├── leader_election_role.yaml
│   │   ├── role_binding.yaml
│   │   └── service_account.yaml
│   └── scorecard
│       ├── bases
│       │   └── config.yaml
│       ├── kustomization.yaml
│       └── patches
│           ├── basic.config.yaml
│           └── olm.config.yaml
├── Dockerfile
├── go.mod
├── go.sum
├── hack
│   └── boilerplate.go.txt
├── main.go
├── Makefile
└── PROJECT

10 directories, 29 files
```



---

* 添加 `CRD` 和 `controller_manager` (控制器)


```bash

pods-operator$ operator-sdk create api --group apps --version v1alpha1 --kind PodSet --resource --controller


Writing scaffold for you to edit...
api/v1alpha1/podset_types.go
controllers/podset_controller.go
Update dependencies:
$ go mod tidy
Running make:
$ make generate
go: creating new go.mod: module tmp
Downloading sigs.k8s.io/controller-tools/cmd/controller-gen@v0.4.1
go: downloading sigs.k8s.io/controller-tools v0.4.1
go: downloading github.com/spf13/cobra v1.0.0
go: downloading k8s.io/api v0.18.2
go: downloading k8s.io/apimachinery v0.18.2
go: downloading gopkg.in/yaml.v3 v3.0.0-20190905181640-827449938966
go: downloading k8s.io/apiextensions-apiserver v0.18.2
go: downloading golang.org/x/tools v0.0.0-20200616195046-dc31b401abb5
go: downloading github.com/gobuffalo/flect v0.2.0
go: downloading github.com/mattn/go-isatty v0.0.8
go: downloading github.com/mattn/go-colorable v0.1.2
go: downloading k8s.io/utils v0.0.0-20200324210504-a9aa75ae1b89
go: downloading sigs.k8s.io/structured-merge-diff/v3 v3.0.0
go: downloading golang.org/x/net v0.0.0-20200226121028-0de0cce0169b
go: downloading golang.org/x/sys v0.0.0-20191022100944-742c48ecaeb7
go get: added sigs.k8s.io/controller-tools v0.4.1
/jicki/golang/pods-operator/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."


```


```bash
pods-operator$  tree 
.
├── api
│   └── v1alpha1
│       ├── groupversion_info.go
│       ├── podset_types.go
│       └── zz_generated.deepcopy.go
├── bin
│   └── controller-gen
├── config
│   ├── crd
│   │   ├── kustomization.yaml
│   │   ├── kustomizeconfig.yaml
│   │   └── patches
│   │       ├── cainjection_in_podsets.yaml
│   │       └── webhook_in_podsets.yaml
│   ├── default
│   │   ├── kustomization.yaml
│   │   ├── manager_auth_proxy_patch.yaml
│   │   └── manager_config_patch.yaml
│   ├── manager
│   │   ├── controller_manager_config.yaml
│   │   ├── kustomization.yaml
│   │   └── manager.yaml
│   ├── manifests
│   │   └── kustomization.yaml
│   ├── prometheus
│   │   ├── kustomization.yaml
│   │   └── monitor.yaml
│   ├── rbac
│   │   ├── auth_proxy_client_clusterrole.yaml
│   │   ├── auth_proxy_role_binding.yaml
│   │   ├── auth_proxy_role.yaml
│   │   ├── auth_proxy_service.yaml
│   │   ├── kustomization.yaml
│   │   ├── leader_election_role_binding.yaml
│   │   ├── leader_election_role.yaml
│   │   ├── podset_editor_role.yaml
│   │   ├── podset_viewer_role.yaml
│   │   ├── role_binding.yaml
│   │   └── service_account.yaml
│   ├── samples
│   │   ├── apps_v1alpha1_podset.yaml
│   │   └── kustomization.yaml
│   └── scorecard
│       ├── bases
│       │   └── config.yaml
│       ├── kustomization.yaml
│       └── patches
│           ├── basic.config.yaml
│           └── olm.config.yaml
├── controllers
│   ├── podset_controller.go
│   └── suite_test.go
├── Dockerfile
├── go.mod
├── go.sum
├── hack
│   └── boilerplate.go.txt
├── main.go
├── Makefile
└── PROJECT

17 directories, 43 files

```


---

> Operator 代码结构


* `api/v1alpha1/podset_type.go` 定义了 PodSet 预期状态的结构体. 


```go
// PodSetSpec defines the desired state of PodSet
type PodSetSpec struct {
	// INSERT ADDITIONAL SPEC FIELDS - desired state of cluster
	// Important: Run "make" to regenerate code after modifying this file

	// Size is the size of the PodSet deployment
	Size int32 `json:"size"`
}

// PodSetStatus defines the observed state of PodSet
type PodSetStatus struct {
	// INSERT ADDITIONAL STATUS FIELD - define observed state of cluster
  // Important: Run "make" to regenerate code after modifying this file
  
  // Nodes are the names of the PodSet pods
	Nodes []string `json:"nodes"`
}

```

---


* 修改了 `type` 资源, 执行 `make generate` 更新资源修改后的代码生成.

  * 上面的 `make generate` 目的调用 `controller-gen` 程序来更新 `api/v1alpha1/zz_generated.deepcopy.go` 文件, 以确保我们API的Go类型定义实现了 `runtime.Object` 所有 Kind 类型必须实现的接口。


```bash
pods-operator$  make generate


/jicki/golang/pods-operator/bin/controller-gen object:headerFile="hack/boilerplate.go.txt" paths="./..."
```

---


* 生成 `CRD` 资源清单 , 执行 `make manifests`

  * 生成的 `CRD` 存于 `config/crd/bases/apps.jicki.cn_podsets.yaml` 

```bash
pods-operator$ make manifests


/jicki/golang/pods-operator/bin/controller-gen "crd:trivialVersions=true,preserveUnknownFields=false" rbac:roleName=manager-role webhook paths="./..." output:crd:artifacts:config=config/crd/bases

```


```yaml
---
apiVersion: apiextensions.k8s.io/v1
kind: CustomResourceDefinition
metadata:
  annotations:
    controller-gen.kubebuilder.io/version: v0.4.1
  creationTimestamp: null
  name: podsets.apps.jicki.cn
spec:
  group: apps.jicki.cn
  names:
    kind: PodSet
    listKind: PodSetList
    plural: podsets
    singular: podset
  scope: Namespaced
  versions:
  - name: v1alpha1
    schema:
      openAPIV3Schema:
        description: PodSet is the Schema for the podsets API
        properties:
          apiVersion:
            description: 'APIVersion defines the versioned schema of this representation
              of an object. Servers should convert recognized schemas to the latest
              internal value, and may reject unrecognized values. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#resources'
            type: string
          kind:
            description: 'Kind is a string value representing the REST resource this
              object represents. Servers may infer this from the endpoint the client
              submits requests to. Cannot be updated. In CamelCase. More info: https://git.k8s.io/community/contributors/devel/sig-architecture/api-conventions.md#types-kinds'
            type: string
          metadata:
            type: object
          spec:
            description: PodSetSpec defines the desired state of PodSet
            properties:
              size:
                description: Size is the size of the PodSet deployment
                format: int32
                type: integer
            required:
            - size
            type: object
          status:
            description: PodSetStatus defines the observed state of PodSet
            properties:
              nodes:
                description: Nodes are the names of the PodSet pods
                items:
                  type: string
                type: array
            required:
            - nodes
            type: object
        type: object
    served: true
    storage: true
    subresources:
      status: {}
status:
  acceptedNames:
    kind: ""
    plural: ""
  conditions: []
  storedVersions: []
```



---

* `controllers` 控制器

  * `controllers/podset_controller.go` 文件中

  * `NewControllerManagedBy` 用于各种控制器的配置

  * `For(&appsv1alpha1.PodSet{})` 用于将 `PodSet{}` 类型指定为要监视的主要资源, 在对 `PodSet{}` 类型的资源进行 `Add/Update/Delete` 操作时, 循环发送 Request 

```go
// SetupWithManager sets up the controller with the Manager.
func (r *PodSetReconciler) SetupWithManager(mgr ctrl.Manager) error {
	return ctrl.NewControllerManagedBy(mgr).
		For(&appsv1alpha1.PodSet{}).
		Complete(r)
}
```




