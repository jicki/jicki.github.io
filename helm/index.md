# Kubernetes 包管理 Helm



# Helm

*  Helm 是 Kubernetes 的包管理器, 可以帮我们简化 kubernetes 应用部署操作, 在多项目的环境下, 应用的部署与管理就会变的频繁与复杂. Helm 通过包的方式, 可简化应用的版本发布、管理、控制.  


* Helm 本质是在 Kubernetes 的应用管理上模板化, 能动态的生成如`Deployment`、`Service` 等的 yaml 资源文件.


* Helm 有三个重要概念

  * `chart` :是创建一个应用的信息集合, 既一个应用所包含的所有需要部署的服务如`deployment`、`service`、`pv`、`pvc` 等的各种 kubernetes 对象的配置模板、参数定义、依赖关系、文档、帮助信息等. `chart` 是应用部署的自包含逻辑单元, 类似于 `Yum` 中的软件rpm安装包. 

  * `reporitory` :是`chart` 的仓库, Helm v2 中需要一个 HTTP 服务器存放 `Charts` 包. Helm v3 中 可以将 Docker 私有仓库当做 `chart` 仓库.

  * `release` :是`chart` 的运行实例, 代表了一个正在运行的应用. 当 `chart` 被安装到 kubernetes 集群中时就生成一个 `release`. 每执行一次 `chart` 安装就会生成一个 `release`.  


---

## Helm 目录 

1. [ 【Helm v2 版本】]({{< ref "helm-v2" >}})

2. [ 【Helm v3 版本】]({{< ref "helm-v3" >}})

3. [ 【Helm Chart 与 模板函数】]({{< ref "helm-chart" >}})

