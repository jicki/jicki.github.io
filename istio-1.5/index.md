# Istio 持续更新...


# Istio 

> 官方文档 https://istio.io/zh/docs

## Istio 是什么？(衣撕朵)

* 云平台令使用它们的公司受益匪浅。但不可否认的是，上云会给 DevOps 团队带来压力。为了可移植性，开发人员必须使用微服务来构建应用，同时运维人员也正在管理着极端庞大的混合云和多云的部署环境。 Istio 允许您连接、保护、控制和观察服务。

* 从较高的层面来说，Istio 有助于降低这些部署的复杂性，并减轻开发团队的压力。它是一个完全开源的服务网格，作为透明的一层接入到现有的分布式应用程序里。它也是一个平台，拥有可以集成任何日志、遥测和策略系统的 API 接口。Istio 多样化的特性使您能够成功且高效地运行分布式微服务架构，并提供保护、连接和监控微服务的统一方法。


## 服务网格 又是什么？

* `服务网格` - 用来描述组成这些应用程序的微服务网络以及它们之间的交互。随着服务网格的规模和复杂性不断的增长，它将会变得越来越难以理解和管理。它的需求包括服务发现、负载均衡、故障恢复、度量和监控等。服务网格通常还有更复杂的运维需求，比如 A/B 测试、金丝雀发布、速率限制、访问控制和端到端认证。



## Istio 架构

![istio架构][7]




* Istio 服务网格从逻辑上分为数据平面和控制平面。

  * `数据平面`&emsp; 由一组智能代理（Envoy）组成，被部署为 sidecar。这些代理通过一个通用的策略和遥测中心（Mixer）传递和控制微服务之间的所有网络通信。

  * `控制平面`&emsp; 管理并配置代理来进行流量路由。此外，控制平面配置 Mixer 来执行策略和收集遥测数据。




## Istio 组件


### Envoy

* `Istio` 使用 `Envoy` 代理的扩展版本。

* `Envoy` &emsp; 是用 `C++` 开发的高性能代理，用于协调服务网格中所有服务的入站和出站流量。

* `Envoy` 代理是唯一与数据平面流量交互的 `Istio` 组件。

* `Envoy` 代理被部署为服务的 `sidecar`，在逻辑上为服务增加了 `Envoy` 的许多内置特性:

  1. 动态服务发现
  2. 负载均衡
  3. TLS 终端
  4. HTTP/2 与 gRPC 代理
  5. 熔断器
  6. 健康检查
  7. 基于百分比流量分割的分阶段发布
  8. 故障注入
  9. 丰富的指标


* `sidecar` 代理模型

  * `sidecar` &emsp; 允许 `Istio` 提取大量关于流量行为的信号作为属性。反之，`Istio` 可以在 `Mixer` 中使用这些属性来执行决策，并将它们发送到监控系统，以提供整个网格的行为信息。 

  * `sidecar` &emsp; 还允许您向现有的部署添加 `Istio` 功能，而不需要重新设计架构或重写代码。


* `Envoy` 代理在 `istio` 中可以实现
  1. 流量控制功能：&emsp; 通过丰富的 HTTP、gRPC、WebSocket 和 TCP 流量路由规则来执行细粒度的流量控制。
  2. 网络弹性特性：&emsp; 重试设置、故障转移、熔断器和故障注入。
  3. 安全性和身份验证特性：&emsp; 执行安全性策略以及通过配置 API 定义的访问控制和速率限制。



### Mixer

* `Mixer` &emsp; 是一个平台无关的组件。`Mixer` 在整个服务网格中执行访问控制和策略使用，并从 `Envoy` 代理和其他服务收集遥测数据。代理提取请求级别属性，并将其发送到 `Mixer` 进行评估。您可以在我们的 `Mixer` 配置文档中找到更多关于属性提取和策略评估的信息。

* `Mixer` &emsp; 包括一个灵活的插件模型。该模型使 `Istio` 能够与各种主机环境和后端基础设施进行交互。因此，`Istio` 从这些细节中抽象出 `Envoy` 代理和 `Istio` 管理的服务。


### Pilot

* `Pilot` &emsp; 主要是为 `Envoy sidecar` 提供服务发现、用于智能路由的流量管理功能（例如，A/B 测试、金丝雀发布等）以及弹性功能（超时、重试、熔断器等）。


* `Pilot` &emsp; 将控制流量行为的高级路由规则转换为特定于环境的配置，并在运行时将它们传播到 `sidecar`。

* `Pilot` &emsp; 将特定于平台的服务发现机制抽象出来，并将它们合成为任何符合 `Envoy API` 的 `sidecar` 都可以使用的标准格式。



* `平台适配器` 与 `Envoy` 交互图 (`平台` &emsp; 支持 kubernetes、Consul、gcp、Nomad等) 
  1. 平台启动一个服务的新实例，该实例通知其平台适配器。
  2. 平台适配器使用 `Pilot` 抽象模型注册实例。
  3. `Pilot` 将流量规则和配置派发给 `Envoy` 代理，来传达此次更改。 
    1. 可以使用 `Istio` 的`流量管理 API`  通过 `Pilot` 优化 `Envoy` 配置，以便对服务网格中的流量进行更细粒度地控制。


![discovery][8]




### Citadel

* `Citadel` &emsp; 通过内置的身份和证书管理，可以支持强大的服务到服务以及最终用户的身份验证。

* `Citadel` &emsp; 可以用来升级服务网格中的未加密流量。

* `Citadel`、`operator` &emsp; 结合使用可以执行基于服务身份的策略，而不是相对不稳定的 3 层或 4 层网络标识。

* `Citadel` &emsp; 使用 `Istio` 的授权特性来控制谁可以访问您的服务。



### Galley

* `Galley` &emsp; 是 `Istio` 的配置验证、提取、处理和分发组件。它负责将其余的 `Istio` 组件与从底层平台（例如 Kubernetes）获取用户配置的细节隔离开来。



## Istio Install

> 我这里 Kubernetes 版本为1.18 因为我目前只有这么一个集群


* 安装 `Istio` 环境准备

  1. 搭建 `Kubernetes` 集群, ( 请按照官方提供的兼容测试版本安装 目前支持 1.14, 1.15, 1.16 )

  2. 下载 `Istio` 项目包. 项目包内包括(安装文件、示例和 istioctl 命令行工具。)

  3. 安装 `Istio`. 通过 `istioctl` 客户端工具直接安装`istio` 到 `Kubernetes 中`.


### 搭建 Kubernetes

> 这一部分这里就省略了, 如需这一部分 文档, 请参考其他的博文。


### 下载 Istio 项目包

* 在 `macOS` 或 `Linux` 系统中, 也可以通过以下命令下载最新版本的 Istio

```
# 新建目录
mkdir -p /opt/istio && cd /opt/istio


# 设置安装版本
export ISTIO_VERSION=1.5.1


# 下载文件
curl -L https://istio.io/downloadIstio | sh -

```

* 输出如下:

```
Istio 1.5.1 Download Complete!

Istio has been successfully downloaded into the istio-1.5.1 folder on your system.

Next Steps:
See https://istio.io/docs/setup/kubernetes/install/ to add Istio to your Kubernetes cluster.

To configure the istioctl client tool for your workstation,
add the /opt/istio/istio-1.5.1/bin directory to your environment path variable with:
         export PATH="$PATH:/opt/istio/istio-1.5.1/bin"

Begin the Istio pre-installation verification check by running:
         istioctl verify-install 

Need more information? Visit https://istio.io/docs/setup/kubernetes/install/ 
```

* 配置环境变量

```
vi /etc/profile

# 添加

# istio
export PATH="$PATH:/opt/istio/istio-1.5.1/bin"


# 生效配置
source /etc/profile

```

* 验证安装

```
istioctl verify-install

Checking the cluster to make sure it is ready for Istio installation...

#1. Kubernetes-api
-----------------------
Can initialize the Kubernetes client.
Can query the Kubernetes API Server.

#2. Kubernetes-version
-----------------------
Istio is compatible with Kubernetes: v1.18.0.

#3. Istio-existence
-----------------------
Istio will be installed in the istio-system namespace.

#4. Kubernetes-setup
-----------------------
Can create necessary Kubernetes configurations: Namespace,ClusterRole,ClusterRoleBinding,CustomResourceDefinition,Role,ServiceAccount,Service,Deployments,ConfigMap. 

#5. SideCar-Injector
-----------------------
This Kubernetes cluster supports automatic sidecar injection. To enable automatic sidecar injection see https://istio.io/docs/setup/kubernetes/additional-setup/sidecar-injection/#deploying-an-app

-----------------------
Install Pre-Check passed! The cluster is ready for Istio installation.

```

* 配置 istioctl 命令自动补全

```
#  创建目录
mkdir -p /usr/share/istio

# 拷贝补全脚本
cp tools/istioctl.bash /usr/share/istio/


# 导入自动补全
source /usr/share/istio/istioctl.bash



# 添加到 bashrc 中
vi ~/.bashrc

# 添加如下:

# istio
source /usr/share/istio/istioctl.bash




# 测试tab补全

[root@k8s-node-1 istio-1.5.1]# istioctl 
analyze          authz            dashboard        experimental     manifest         profile          proxy-status     upgrade          verify-install   
authn            convert-ingress  deregister       kube-inject      operator         proxy-config     register         validate         version  

```


* 目录结构说明

  * `bin`&emsp; 目录包含 istioctl 的客户端文件。istioctl 工具用于手动注入 Envoy sidecar 代理。

  * `manifest.yaml`&emsp; 文件的 sha码。

  * `samples`&emsp; 目录包含 istio 的实例应用程序。   

  * `tools`&emsp; 目录包含 一些脚本
    * `convert_RbacConfig_to_ClusterRbacConfig.sh`
    * `dump_kubernetes.sh` 
    * `_istioctl`
    * `istioctl.bash`&emsp; istio 命令tab自动补全的脚本

  * `install`&emsp; 目录包含如下目录: (istio除了支持 kubernetes 之外还支持 consul 和 gcp)
    * `consul`&emsp; 目录
    * `gcp`&emsp; 目录  
    * `kubernetes`&emsp; 目录包含 `kuebrnetes` 服务相关的 YAML 文件。
    * `tools` &emsp; 目录



### 安装 istio

* `istio` 包含两种安装方式

  1. 通过 istioctl 客户端命令安装 (推荐)

  2. 通过 helm 进行安装


```
# 通过如下命令进行安装 (manifest 是资源清单, profile 指定类型的清单)
istioctl manifest apply --set profile=demo 

```

* 输出如下:

```
- Applying manifest for component Base...
✔ Finished applying manifest for component Base.
- Applying manifest for component Pilot...
✔ Finished applying manifest for component Pilot.
- Applying manifest for component EgressGateways...
- Applying manifest for component IngressGateways...
- Applying manifest for component AddonComponents...
✔ Finished applying manifest for component EgressGateways.
✔ Finished applying manifest for component IngressGateways.
✔ Finished applying manifest for component AddonComponents.

✔ Installation complete
```

* 查看部署情况

  * 如果集群运行在一个不支持外部负载均衡器的环境中（例如：minikube），istio-ingressgateway 的 EXTERNAL-IP 将显示为 `<pending>` 状态。请使用服务的 NodePort 或 端口转发来访问网关。


```
kubectl get pods -n istio-system
NAME                                   READY   STATUS    RESTARTS   AGE
grafana-5f6f8cbf75-lqnjg               1/1     Running   0          17m
istio-egressgateway-74896c8487-mjlg8   1/1     Running   0          17m
istio-ingressgateway-54d494869-7npql   1/1     Running   0          17m
istio-tracing-9dd6c4f7c-x5kcp          1/1     Running   0          17m
istiod-756bd84654-n2k6m                1/1     Running   0          18m
kiali-869c6894c5-64vmw                 1/1     Running   0          17m
prometheus-c89875c74-rgzdx             2/2     Running   0          17m
```


```
kubectl get svc -n istio-system 
NAME                        TYPE           CLUSTER-IP      EXTERNAL-IP   PORT(S)                                                                                                                                      AGE
grafana                     ClusterIP      10.254.35.48    <none>        3000/TCP                                                                                                                                     158m
istio-egressgateway         ClusterIP      10.254.10.123   <none>        80/TCP,443/TCP,15443/TCP                                                                                                                     158m
istio-ingressgateway        LoadBalancer   10.254.26.46    <pending>     15020:32142/TCP,80:30000/TCP,443:32701/TCP,15029:30413/TCP,15030:30781/TCP,15031:31714/TCP,15032:32419/TCP,31400:30673/TCP,15443:31123/TCP   158m
istio-pilot                 ClusterIP      10.254.54.205   <none>        15010/TCP,15011/TCP,15012/TCP,8080/TCP,15014/TCP,443/TCP                                                                                     158m
istiod                      ClusterIP      10.254.3.7      <none>        15012/TCP,443/TCP                                                                                                                            158m
jaeger-agent                ClusterIP      None            <none>        5775/UDP,6831/UDP,6832/UDP                                                                                                                   158m
jaeger-collector            ClusterIP      10.254.2.80     <none>        14267/TCP,14268/TCP,14250/TCP                                                                                                                158m
jaeger-collector-headless   ClusterIP      None            <none>        14250/TCP                                                                                                                                    158m
jaeger-query                ClusterIP      10.254.61.2     <none>        16686/TCP                                                                                                                                    158m
kiali                       ClusterIP      10.254.7.135    <none>        20001/TCP                                                                                                                                    158m
prometheus                  ClusterIP      10.254.8.200    <none>        9090/TCP                                                                                                                                     158m
tracing                     ClusterIP      10.254.39.104   <none>        80/TCP                                                                                                                                       158m
zipkin                      ClusterIP      10.254.32.230   <none>        9411/TCP                                                                                                                                     158m
```



* 组件说明

  * `tracing`&emsp; 全链路监控。
  * `istio-pilot`&emsp; 服务发现与服务配置。
  * `kiali`&emsp; 可视化服务网格展示。 
    * 服务拓扑图
    * 分布式跟踪
    * 指标度量收集和图标
    * 配置校验
    * 健康检查和显示
    * 服务发现
  * `prometheus`&emsp; 大家都懂的监控。
  * `grafana`&emsp; `prometheus`监控的展示webui。
  * `istio-ingressgateway`&emsp; 出口网关。
  * `istio-egressgateway`&emsp; 入口网关。
  * `jaeger`&emsp;
    * `jaeger-agent`&emsp; jaeger client的一个代理程序，client将收集到的调用链数据发给agent，然后由agent发给collector；
    * `jaeger-collector`&emsp; 负责接收jaeger client或者jaeger agent上报上来的调用链数据，然后做一些校验，比如时间范围是否合法等，最终会经过内部的处理存储到后端存储；
    * `jaeger-query`&emsp; 专门负责调用链查询的一个服务。 




* 修改 `istio-ingressgateway` 网络类型 为 NodePort

```
kubectl patch service istio-ingressgateway -n istio-system -p '{"spec":{"type":"NodePort"}}'

```

```
kubectl get svc -n istio-system istio-ingressgateway 
NAME                   TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)                                                                                                                                      AGE
istio-ingressgateway   NodePort   10.254.26.46   <none>        15020:32142/TCP,80:30000/TCP,443:32701/TCP,15029:30413/TCP,15030:30781/TCP,15031:31714/TCP,15032:32419/TCP,31400:30673/TCP,15443:31123/TCP   176m
```


### 验证 istio


* 查看版本

```
[root@k8s-node-1 ~]# istioctl version
client version: 1.5.1
control plane version: 1.5.1
data plane version: 1.5.1 (3 proxies)
```


### Kiali 组件

> Kiali 以 web ui 的方式可视化服务网格。


*  查看 kiali  svc

```
[root@k8s-node-1 kubeadm]# kubectl get svc -n istio-system  kiali
NAME    TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)     AGE
kiali   ClusterIP   10.254.7.135   <none>        20001/TCP   4h24m
```


* 配置访问(我这里的node节点为云主机,所以我配置了一个 ingress)

```
[root@k8s-node-1 ~]# cat kiali-ingress.yaml 
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kiali-ingress
  namespace: istio-system
spec:
  rules:
  - host: kiali.jicki.cn
    http:
      paths:
      - backend:
          serviceName: kiali
          servicePort: 20001

```

* 创建ingress服务

```
[root@k8s-node-1 kubeadm]# kubectl apply -f kiali-ingress.yaml 
ingress.extensions/kiali-ingress created


# 查看服务
[root@k8s-node-1 ~]# kubectl get ingress -n istio-system 
NAME            CLASS    HOSTS             ADDRESS       PORTS   AGE
kiali-ingress   <none>   kiali.jicki.cn    10.254.8.81   80      2m31s

```


![kiali1][1]

![kiali1][2]

![kiali1][3]

![kiali1][4]

![kiali1][5]





### Istio Profile

> Profile 相关的介绍以及具体的区别


* `istioctl profile list` 命令可查看当前版本的 profile

```
[root@k8s-node-1 istio]# istioctl profile list
Istio configuration profiles:
    minimal
    remote
    separate
    default
    demo
    empty

```

* `profile`&emsp; 包含如下:
  1. `remote` 远程`kubernetes`部署, 以及多`kubernetes`集群
  2. `separatei` 独立部署,不建议使用,后续可能删除
  3. `default` 默认安装, 根据IstioControlPlaneAPI的默认设置启用组件, 建议用于生产部署
  4. `demo` 演示实例,展示`istio` 所有功能且资源需求适中的配置
  5. `empty` 不部署任何内容。用于导出空的配置文件。
  6. `minimal` 最小化安装。


|istio组件|default|demo|minimal|remote|empty|
|-|-|-|-|-|-|
|`istio-egressgateway`||✔||||
|`istio-ingressgateway`|✔|✔||✔||
|`istio-pilot`|✔|✔|✔|✔||
|`grafana`||✔||||
|`istio-tracing`||✔||||
|`Kiali`||✔||||
|`prometheus`|✔|✔||✔||
|`jaeger`||✔||||
|`zipkin`||✔||||



* `istioctl profile dump profileName` 可以打印或者导出`profile`配置

  * 这里导出的文件就是`kubernetes` 的 YAML 编排文件。api 为 istio 的 api。

  * 这里可以导出 `default` 然后根据自己的环境自定义适合自己的 profile。 

```
[root@k8s-node-1 istio]# istioctl profile dump default > default.yaml




```


### istio injection

* `injection` 注入后的变化

  * 原生 pod &emsp; -->  `pods` 包含 `程序` 项目。

  * 注入以后 pod &emsp; --> `pods` 包含 `程序` 项目、`istio-init`、`istio-proxy`。

    * `istio-init`&emsp; 用于初始化网络配置, `iptables` 路由配置。

    * `istio-proxy`&emsp; 用于当前`pod` 与集群内部其他资源进行交互。 

* 可以被 `injection` 的服务

  * `Deployment` - 注入后会添加 `istio-init`、`istio-proxy` 。

  * `ReplicaSet` - 注入后会添加 `istio-init`、`istio-proxy` 。

  * `DeamonSet` - 注入后会添加 `istio-init`、`istio-proxy` 。

  * `Pod` - 注入后会添加 `istio-init`、`istio-proxy` 。

  * `Job` - 注入后会添加 `istio-init`、`istio-proxy` 。

  * `Service` - 注入后不会添加任何组件。

  * `Secrets` - 注入后不会添加任何组件。

  * `ConfigMap` - 注入后不会添加任何组件。



* deployment yaml 编排文件注入

  * `istioctl kube-inject -f nginx-test.yaml` 对 `Deployment`编排文件进行注入(会修改yaml文件内容)

  * `kubectl apply -f <(istioctl kube-inject -f nginx-test.yaml)` 注入并 创建 服务到`kubernetes`中, 这样操作不会改变原有的编排文件。

```
# 创建一个 namespaces
[root@k8s-node-1 istio]# kubectl create ns jicki
namespace/jicki created
```


```
# 创建一个 deployment

[root@k8s-node-1 istio]# cat nginx-test.yaml 
apiVersion: apps/v1
kind: Deployment 
metadata: 
  name: nginx-deployment
  namespace: jicki
  labels:
    app: nginx
spec: 
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template: 
    metadata: 
      labels: 
        app: nginx 
        version: v1.0.0
    spec: 
      containers: 
        - name: nginx 
          image: nginx:alpine 
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
              name: http
```

* 导出注入后的编排文件

```
[root@k8s-node-1 istio]# istioctl kube-inject -f nginx-test.yaml > nginx-test-inject.yaml

```

```
# 注入后 yaml 发生变化, 会增加很多istio的配置
# 包含新增的两个容器分别为 istio-proxy 与 istio-init 

        image: docker.io/istio/proxyv2:1.5.1
        imagePullPolicy: IfNotPresent
        name: istio-proxy

        image: docker.io/istio/proxyv2:1.5.1
        imagePullPolicy: IfNotPresent
        name: istio-init

```

* 创建服务

```
[root@k8s-node-1 istio]# kubectl apply -f <(istioctl kube-inject -f nginx-test.yaml)
deployment.apps/nginx-deployment created
```

```
[root@k8s-node-1 istio]# kubectl describe pods/nginx-deployment-59897745f9-vlvn9 -n jicki


Events:
  Type    Reason     Age        From                 Message
  ----    ------     ----       ----                 -------
  Normal  Scheduled  <unknown>  default-scheduler    Successfully assigned jicki/nginx-deployment-59897745f9-vlvn9 to k8s-node-1
  Normal  Pulled     2m19s      kubelet, k8s-node-1  Container image "docker.io/istio/proxyv2:1.5.1" already present on machine
  Normal  Created    2m19s      kubelet, k8s-node-1  Created container istio-init
  Normal  Started    2m18s      kubelet, k8s-node-1  Started container istio-init
  Normal  Pulled     2m18s      kubelet, k8s-node-1  Container image "nginx:alpine" already present on machine
  Normal  Created    2m18s      kubelet, k8s-node-1  Created container nginx
  Normal  Started    2m17s      kubelet, k8s-node-1  Started container nginx
  Normal  Pulled     2m17s      kubelet, k8s-node-1  Container image "docker.io/istio/proxyv2:1.5.1" already present on machine
  Normal  Created    2m17s      kubelet, k8s-node-1  Created container istio-proxy
  Normal  Started    2m17s      kubelet, k8s-node-1  Started container istio-proxy
```

```
# 查看 istio 对外服务的端口以及相关进程
# 可以发现除了80端口还有其他5个额外的端口

[root@k8s-node-1 istio]# kubectl exec -it nginx-deployment-59897745f9-vlvn9 -n jicki -c istio-proxy -- netstat -ntlp
Active Internet connections (only servers)
Proto Recv-Q Send-Q Local Address           Foreign Address         State       PID/Program name    
tcp        0      0 0.0.0.0:15001           0.0.0.0:*               LISTEN      17/envoy            
tcp        0      0 0.0.0.0:15006           0.0.0.0:*               LISTEN      17/envoy            
tcp        0      0 0.0.0.0:80              0.0.0.0:*               LISTEN      -                   
tcp        0      0 0.0.0.0:15090           0.0.0.0:*               LISTEN      17/envoy            
tcp        0      0 127.0.0.1:15000         0.0.0.0:*               LISTEN      17/envoy            
tcp6       0      0 :::15020                :::*                    LISTEN      1/pilot-agent 


```

|Address|端口|类型|进程|说明|
|-|-|-|-|-|
|127.0.0.1|15000|TCP|envoy|envoy 命令行管理端口,可以获取程序运行信息。|
|0.0.0.0|15001|TCP|envoy|
|0.0.0.0|15006|TCP|envoy|
|0.0.0.0|15020|HTTP|pilot-agent|
|0.0.0.0|15090|HTTP|envoy|



![istio-proxy][6]

* `pilot-agent`

  * 生成 `envoy` 配置文件
  * 启动 `envoy` 进程
  * 监控和管理`envoy`进程的运行状态, 故障恢复, `envoy`配置变更以及重新加载。

* `envoy` &emsp; 轻量级的代理服务器。




* service yaml 编排文件注入

  * 上面已经提到过 `service` 注入是不会添加任何其他的服务到 `service` 中的。


```
# 创建一个基于 nginx 的 svc
[root@k8s-node-1 istio]# cat nginx-test-svc.yaml 
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: jicki
  labels:
    app: nginx
spec:
  ports:
    - port: 80
      name: http
      targetPort: 80
      protocol: TCP
  selector:
    app: nginx
```

```
# 创建服务
[root@k8s-node-1 istio]# kubectl apply -f <(istioctl kube-inject -f nginx-test-svc.yaml)
service/nginx-svc created

```

```
# 输出一个注入后的文件
[root@k8s-node-1 istio]# istioctl kube-inject -f nginx-test-svc.yaml -o nginx-test-svc-inject.yaml 

```

```
# 查看文件 (可以看出来在注入后文件是没有变化的)
[root@k8s-node-1 kubeadm]# cat nginx-test-svc-inject.yaml 
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc
  namespace: jicki
  labels:
    app: nginx
spec:
  ports:
    - port: 80
      name: http
      targetPort: 80
      protocol: TCP
  selector:
    app: nginx
---
```


* `injection` 注入检测 (istioctl analyze -n namespace 命令)

  * 运行此命令会检测 `namespace` 是否有 `istio` 的资源配置情况。

  * 如下提示 `default` 命令中间下,并没有 注入 `istio`，以及使用 `kubectl label` 标签进行注入。
 
```
[root@k8s-node-1 ~]# istioctl analyze -n jicki

Warn [IST0102] (Namespace jicki) The namespace is not enabled for Istio injection. Run 'kubectl label namespace jicki istio-injection=enabled' to enable it, or 'kubectl label namespace jicki istio-injection=disabled' to explicitly mark it as not needing injection
Error: Analyzers found issues when analyzing namespace: jicki.
See https://istio.io/docs/reference/config/analysis for more information about causes and resolutions.


```

* `injection` 自动注入

  * 对 `namespces` 设置 `labels` 标签, 可以使在这个 `namespaces` 生成的可注入的程序进行自动注入(`injection`)

```
# 创建一个 namespaces
[root@k8s-node-1 ~]# kubectl create namespace jicki
namespace/jicki created

# 注入
[root@k8s-node-1 ~]# kubectl label namespace jicki istio-injection=enabled
namespace/jicki labeled

```


```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: jicki
  labels:
    istio-injection: enabled
```




```
# 检测注入情况
[root@k8s-node-1 ~]# istioctl analyze -n jicki
✔ No validation issues found when analyzing namespace: jicki.


# 查看 namespace 的 labels
[root@k8s-node-1 ~]# kubectl get ns --show-labels
NAME              STATUS   AGE     LABELS
default           Active   9d      <none>
jicki             Active   4m46s   istio-injection=enabled

```



## Istio 流量管理

> Istio 的流量管理 就是配置 HTTP/TCP 路由功能。 

> Update Istio Version 1.9.9

* `Istio` 流量

  1. `data plane 数据面流量` &emsp; 是指 网格内服务之间业务互相调用所产生的流量。

  2. `control plane 控制面流量` &emsp; 是指 `Istio` 各组件之间配置和控制网格行为所产生的流量。

* `Istio` 流量管理分为六个部分

  1. `Destination Rule`

  2. `Envoy Filter`

  3. `Gateway`

  4. `Service Entry`

  5. `Sidecar`

  6. `Virtual Service`


### Istio 流量模型

* `Istio` 依赖与服务注入(inject)时所部署的 `Envoy` 代理。

* 网格内发送与接收的所有的流量, 都通过 `Envoy` 进行代理。

* `Envoy`代理 可以轻松的引导和控制网格的流量, 而不需要对服务进行任何修改。


### Virtual Service

* 在`Istio`所提供的基本连接和发现基础上, 通过虚拟服务(vitrual service), 能够将请求路由到`Istio`网格中的特定服务。每个虚拟服务(vitrual service)由一组路由规则组成, 这些路由规则使`Istio`能够将虚拟服务(vitrual service)的每个给定请求匹配到网格内特定的目标地址。

* `虚拟服务(Virtual Service)`&emsp; 定义了一组寻址主机时要使用的流量路由规则，每个路由规则为特定协议的流量定义匹配了条件。如果流量匹配，则将其发送到注册表中定义的命名目标服务（或其子集/版本）。同时流量来源也可以在路由规则中匹配，从而能够允许针对特定的客户端上下文自定义路由。
  * `服务（Service）`: &emsp; 应用绑定到服务注册表中的唯一名称。服务包含多个网络端点，这些端点由在Pod、容器和VM等上运行的工作负载实例进行实现。

  * `服务版本（Service versions）`或`子集`:&emsp; 在持续部署的场景中，对于给定的服务，可能存在不同的应用程序实例。它们可能是对同一服务的迭代更改，这些版本部署在不同的环境（产品，阶段和开发等）中。场景包括`A / B测试` 和 `金丝雀发布` 等。可以根据各种标准（标头，网址等）和/或通过分配给每个版本的权重来确定特定版本的选择。每个服务都有一个包含其所有实例的默认版本。 

  * `来源（Source）`:&emsp; 调用服务的下游客户端。

  * `主机（Host）`:&emsp; 下游客户端连接服务时所使用的地址。

  * `访问模式（Access model）`:&emsp; 应用程序仅需要处理目标服务（主机），而无需了解各个服务版本（子集）。版本的实际选择由代理/sidecar决定，从而使应用程序代码能够与所依赖服务的脱钩。




* 一个例子

  * `注`:&emsp; 所有服务(deployment)、(service)、包括 client 都必须被 `istio` 注入(injection)。

```
# 2 个 nginx deployment 3 个 service

[root@k8s-node-1 istio]# cat nginx-test.yaml 
apiVersion: apps/v1
kind: Deployment 
metadata: 
  name: nginx-1
  labels:
    web: nginx-1
spec: 
  replicas: 1
  selector:
    matchLabels:
      web: nginx-1
  template: 
    metadata: 
      labels: 
        app: nginx 
        web: nginx-1
        version: v1.0.0
    spec: 
      containers: 
        - name: nginx 
          image: nginx:alpine 
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
              name: http
          command: ["/bin/sh", "-c", "echo 'hello nginx-1' > /usr/share/nginx/html/index.html; nginx  -g  'daemon off;'"]
---
apiVersion: apps/v1
kind: Deployment 
metadata: 
  name: nginx-2
  labels:
    web: nginx-2
spec: 
  replicas: 1
  selector:
    matchLabels:
      web: nginx-2
  template: 
    metadata: 
      labels: 
        app: nginx 
        web: nginx-2
        version: v1.0.1
    spec: 
      containers: 
        - name: nginx 
          image: nginx:alpine 
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
              name: http
          command: ["/bin/sh", "-c", "echo 'hello nginx-2' > /usr/share/nginx/html/index.html; nginx  -g  'daemon off;'"]
---
apiVersion: apps/v1
kind: Deployment 
metadata: 
  name: nginx-3
  labels:
    web: nginx-3
spec: 
  replicas: 1
  selector:
    matchLabels:
      web: nginx-3
  template: 
    metadata: 
      labels: 
        app: nginx
        web: nginx-3
        version: v1.0.2
    spec: 
      containers: 
        - name: nginx 
          image: nginx:alpine 
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
              name: http
          command: ["/bin/sh", "-c", "echo 'hello jicki' > /usr/share/nginx/html/index.html; nginx  -g  'daemon off;'"]
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-1
  labels:
    web: nginx-1
spec:
  ports:
    - port: 80
      name: http
      targetPort: 80
      protocol: TCP
  selector:
    web: nginx-1
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-2
  labels:
    web: nginx-2
spec:
  ports:
    - port: 80
      name: http
      targetPort: 80
      protocol: TCP
  selector:
    web: nginx-2
---
apiVersion: v1
kind: Service
metadata:
  name: nginx-svc-3
  labels:
    web: nginx-3
spec:
  ports:
    - port: 80
      name: http
      targetPort: 80
      protocol: TCP
  selector:
    web: nginx-3
---
apiVersion: v1 
kind: Service
metadata: 
  name: nginx-svc 
  labels:
    app: nginx
spec: 
  ports: 
    - port: 80
      name: http
      targetPort: 80
      protocol: TCP 
  selector: 
    app: nginx
```


* 查看配置的三个 `service` 的 `endpoints`

```
[root@k8s-node-1 istio]# kubectl get ep
NAME          ENDPOINTS                                            AGE
kubernetes    172.18.186.159:6443                                  16d
nginx-svc     10.254.64.163:80,10.254.64.164:80,10.254.64.165:80   6m23s
nginx-svc-1   10.254.64.163:80                                     26s
nginx-svc-2   10.254.64.164:80                                     26s
nginx-svc-3   10.254.64.165:80                                     26s
```



* 配置一个 `Virtual Service`

```yaml
[root@k8s-node-1 istio]# cat nginx-test-vs.yaml 
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-svc-vs
spec:
  # 客户端访问服务的地址, ( svc名称 + namespace 用于跨 namespace 调用 )
  hosts: 
  - nginx-svc.default
  # http协议
  http:
  # 如下为匹配条件
  - match:
    # http 头 包含如下信息
    - headers:
        # key = vs-user
        vs-user:
          # exact 完全匹配 value = jicki
          exact: jicki
    route:
    - destination:
        host: nginx-svc-3.default
  # 如下为匹配路由规则
  - route:
    - destination:
        # 匹配的服务版本或子集为 nginx-1
        host: nginx-svc-1.default
    - destination:
        # 匹配的服务版本或子集为 nginx-2
        host: nginx-svc-2.default

```

* `VirtualService` 服务并非是 `kubernetes` 中实际的 `service` 服务。

  * `kubectl get virtualservices` 使用这个命令查看 `Virtual Service` 

```
[root@k8s-node-1 istio]# kubectl get virtualservices
NAME           GATEWAYS   HOSTS                                   AGE
nginx-svc-vs              [nginx-svc.default.svc.cluster.local]   3m1s

```



* 测试以及 kiali 中查看

```yaml
# 创建一个 busybox 的 deployment 作为客户端
[root@k8s-node-1 istio]# cat busybox.yaml 
apiVersion: apps/v1
kind: Deployment
metadata:
  name: busybox
  labels:
    app: busybox
spec:
  replicas: 1
  selector:
    matchLabels:
      app: busybox
  template:
    metadata:
      labels:
        app: busybox
    spec:
      containers:
        - name: busybox
          image: jicki/busybox-curl
          imagePullPolicy: IfNotPresent
          command: ["/bin/sh", "-c", "sleep 3600s"]
```

* 客户端 `istio` 注入


```
kubectl apply -f <(istioctl kube-inject -f busybox.yaml)

```



```
# 访问一下 nginx-svc
[root@k8s-node-1 istio]# kubectl exec -it busybox-7f6578cb8b-qfptk -c busybox -- sh
```



```
# 带 headers 的访问

curl --header "vs-user: jicki" http://nginx-svc


hello jicki

```


```
# 模拟流量

while true;
do
wget -q -O - http://nginx-svc;
done

```



* 查看 kiali


```bash
# 通过 istioctl 打开 kiali 

istioctl dashboard kiali
```




![virutalservice][9]




### Istio Gateway


* 创建 一个基于 Nginx Web 服务的 `Istio Gateway`


```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: nginx-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  - port:
      number: 80
      name: http
      protocol: HTTP
    hosts:
    - "*"
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-gw-vs
spec:
  hosts:
  # 不绑定 域名 可使用 *
  - "*"
  gateways:
  - nginx-gateway
  http:
  - route:
    - destination:
        host: nginx-svc
        port:
          number: 80
```

---


* 配置 `DestinationRule`

  * 目标规则: `https://istio.io/latest/zh/docs/reference/config/networking/destination-rule/`

  * 通过 `svc labels` 选择访问

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: nginx-destinationrule
spec:
  host: nginx-svc
  subsets:
  - name: nginx-1
    labels:
      web: nginx-1
  - name: nginx-2
    labels:
      web: nginx-2
  - name: nginx-3
    labels:
      web: nginx-3
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-gw-vs
spec:
  hosts:
  - "*"
  gateways:
  - nginx-gateway
  http:
  - route:
    - destination:
        host: nginx-svc
        port:
          number: 80
        subset: nginx-3
```


--- 

* 基于 TLS 的 Gateway

  * 创建一个 自签 的tls证书

```bash
# 使用 openssl 创建 根证书 与 私钥  (jicki.cn)
openssl req -x509 -sha256 -nodes -days 365 -newkey rsa:2048 -subj '/O=jicki Inc./CN=jicki.cn' -keyout jicki.cn.key -out jicki.cn.crt

Generating a RSA private key
............+++++
.....+++++
writing new private key to 'jicki.cn.key'
-----

# 输出证书

total 8
-rw-r--r-- 1 root root 1168 Oct 28 06:12 jicki.cn.crt
-rw------- 1 root root 1704 Oct 28 06:12 jicki.cn.key
```


* 利用 根证书与密钥 创建 `nginx.jicki.cn` 证书与私钥

```bash
openssl req -out nginx.jicki.cn.csr -newkey rsa:2048 -nodes -keyout nginx.jicki.cn.key -subj "/CN=nginx.jicki.cn/O=nginx organization"


Generating a RSA private key
.......................................................................................+++++
........+++++
writing new private key to 'nginx.jicki.cn.key'
-----


# 创建 crt 证书

openssl x509 -req -days 365 -CA jicki.cn.crt -CAkey jicki.cn.key -set_serial 0 -in nginx.jicki.cn.csr -out nginx.jicki.cn.crt



Signature ok
subject=CN = nginx.jicki.cn, O = nginx organization
Getting CA Private Key


# 输出证书

total 20
-rw-r--r-- 1 root root 1038 Oct 28 06:15 nginx.jicki.cn.crt
-rw-r--r-- 1 root root  936 Oct 28 06:14 nginx.jicki.cn.csr
-rw------- 1 root root 1704 Oct 28 06:14 nginx.jicki.cn.key
-rw-r--r-- 1 root root 1168 Oct 28 06:12 jicki.cn.crt
-rw------- 1 root root 1704 Oct 28 06:12 jicki.cn.key
```


---

* 创建 Gateway 中使用到的 `Secret`

   * 需要将 Secret 创建于 `istio-system` 这个 namespaces 下.

```bash
kubectl create -n istio-system secret tls nginx-tls --key=nginx.jicki.cn.key --cert=nginx.jicki.cn.crt

```

```bash
kubectl -n istio-system get secrets


NAME                                               TYPE                                  DATA   AGE
default-token-pvqnh                                kubernetes.io/service-account-token   3      14d
istio-ca-secret                                    istio.io/ca-root                      5      14d
istio-ingressgateway-service-account-token-jbnvz   kubernetes.io/service-account-token   3      14d
istio-reader-service-account-token-2kwcl           kubernetes.io/service-account-token   3      14d
istiod-service-account-token-n59qq                 kubernetes.io/service-account-token   3      14d
kiali-token-fhwkl                                  kubernetes.io/service-account-token   3      14d
nginx-tls                                          kubernetes.io/tls                     2      8s
```


---

```yaml
apiVersion: networking.istio.io/v1beta1
kind: Gateway
metadata:
  name: nginx-gateway
spec:
  selector:
    istio: ingressgateway # use istio default controller
  servers:
  # 每定义一个 hosts 需要多写一个 -port 段落
  - port:
      number: 443
      name: https
      protocol: HTTPS
    tls:
      # 单向 TLS 的模式名称为固定值 SIMPLE  双向 TLS 的模式名称为固定值 MUTUAL
      mode: SIMPLE
      credentialName: nginx-tls  # 上面创建的 证书 secret 名称
    hosts:
    - "nginx.jicki.cn"
---
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-gw-vs
spec:
  hosts:
  # 不绑定 域名 可使用 *
  - "nginx.jicki.cn"
  gateways:
  - nginx-gateway
  http:
  - route:
    - destination:
        host: nginx-svc
        port:
          number: 80
```


* 测试访问

```bash
curl -v https://nginx.jicki.cn/ --cacert jicki.cn.crt


*   Trying 10.9.9.216:443...
* TCP_NODELAY set
* Connected to nginx.jicki.cn (10.9.9.216) port 443 (#0)
* ALPN, offering h2
* ALPN, offering http/1.1
* successfully set certificate verify locations:
*   CAfile: jicki.cn.crt
  CApath: /etc/ssl/certs
* TLSv1.3 (OUT), TLS handshake, Client hello (1):
* TLSv1.3 (IN), TLS handshake, Server hello (2):
* TLSv1.3 (IN), TLS handshake, Encrypted Extensions (8):
* TLSv1.3 (IN), TLS handshake, Certificate (11):
* TLSv1.3 (IN), TLS handshake, CERT verify (15):
* TLSv1.3 (IN), TLS handshake, Finished (20):
* TLSv1.3 (OUT), TLS change cipher, Change cipher spec (1):
* TLSv1.3 (OUT), TLS handshake, Finished (20):
* SSL connection using TLSv1.3 / TLS_CHACHA20_POLY1305_SHA256
* ALPN, server accepted to use h2
* Server certificate:
*  subject: CN=nginx.jicki.cn; O=nginx organization
*  start date: Oct 28 06:15:47 2021 GMT
*  expire date: Oct 28 06:15:47 2022 GMT
*  common name: nginx.jicki.cn (matched)
*  issuer: O=jicki Inc.; CN=jicki.cn
*  SSL certificate verify ok.
* Using HTTP2, server supports multi-use
* Connection state changed (HTTP/2 confirmed)
* Copying HTTP/2 data in stream buffer to connection buffer after upgrade: len=0
* Using Stream ID: 1 (easy handle 0x55d575b42e10)
> GET / HTTP/2
> Host: nginx.jicki.cn
> user-agent: curl/7.68.0
> accept: */*
> 
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* TLSv1.3 (IN), TLS handshake, Newsession Ticket (4):
* old SSL session ID is stale, removing
* Connection state changed (MAX_CONCURRENT_STREAMS == 2147483647)!
< HTTP/2 200 
< server: istio-envoy
< date: Thu, 28 Oct 2021 06:29:55 GMT
< content-type: text/html
< content-length: 12
< last-modified: Tue, 19 Oct 2021 02:56:01 GMT
< etag: "616e33c1-c"
< accept-ranges: bytes
< x-envoy-upstream-service-time: 15
< 
hello jicki
* Connection #0 to host nginx.jicki.cn left intact
```



### Fault Injection

> 故障注入

* Istio 的故障注入规则可以帮助您识别微服务中有硬编码超时, 导致 服务失败等。此类异常, 而不会影响最终用户。


* 故障注入 - 延迟调用, 在使用条件匹配访问 vs-user 用户等于 jicki 的时候 延迟7s 后调用, 延迟比例为 100% 


```yaml
# 在 内部网格中实现 故障注入

[root@k8s-node-1 istio]# cat nginx-vs.yaml

apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-svc-vs
spec:
  # 客户端访问服务的地址
  hosts: 
  - nginx-svc
  # http协议
  http:
  # 如下为 故障注入
  - fault:
      delay:
        # 延迟 7秒
        fixedDelay: 7s
        # 延迟比例 100%
        percentage:
          value: 100
  # 如下为匹配条件
    match:
    # http 头 包含如下信息
    - headers:
        # key = vs-user
        vs-user:
          # exact 完全匹配 value = jicki
          exact: jicki
    route:
    - destination:
        host: nginx-svc-3
  # 如下为匹配路由规则
  - route:
    - destination:
        # 匹配的服务版本或子集为 nginx-1
        host: nginx-svc-1
    - destination:
        # 匹配的服务版本或子集为 nginx-2
        host: nginx-svc-2
```


```bash
time curl --header "vs-user: jicki" http://nginx-svc


hello jicki
real	0m 7.02s
user	0m 0.00s
sys	0m 0.00s
```


* 在 外部调用 中实现 故障注入


```yaml

apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-gw-vs
spec:
  hosts:
  - "*"
  gateways:
  - nginx-gateway
  http:
  # 如下为 故障注入
  - fault:
      delay:
        # 延迟 7秒
        fixedDelay: 7s
        # 延迟比例 100%
        percentage:
          value: 100
    route:
    - destination:
        host: nginx-svc
        port:
          number: 80
        subset: nginx-3
```




### Traffic Shifting

> 流量转移

* 在项目服务中, 通常会有一些新旧版本的逐步迁移, 如流量从微服务的一个版本的逐渐迁移到另一个版本。流量转移适用于web服务以及TCP服务。

  * 通过配置 `destination.weight` 来定义流量的百分比 (只用于网格内的服务)

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-svc-vs
spec:
  # 客户端访问服务的地址, ( svc名称 + namespace 用于跨 namespace 调用 )
  hosts: 
  - nginx-svc.default
  # http协议
  http:
  # 如下为匹配条件
  - match:
    # http 头 包含如下信息
    - headers:
        # key = vs-user
        vs-user:
          # exact 完全匹配 value = jicki
          exact: jicki
    route:
    - destination:
        host: nginx-svc-3.default
  # 如下为匹配路由规则
  - route:
    - destination:
        # 匹配的服务版本或子集为 nginx-1
        host: nginx-svc-1.default
      # 流量比例
      weight: 80
    - destination:
        # 匹配的服务版本或子集为 nginx-2
        host: nginx-svc-2.default
      # 流量比例
      weight: 20
```




### Request Timeouts

> 请求超时 

* 通过设置 请求超时 来测试服务的一些特性.


```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-gw-vs
spec:
  hosts:
  - "*"
  gateways:
  - nginx-gateway
  http:
  - route:
    - destination:
        host: nginx-svc
        port:
          number: 80
        subset: nginx-3
    timeout: 0.01s
```


* Nginx 服务会直接返回 `upstream request timeout` , 这里不需要修改 Nginx 的配置文件设置超时, 通过 vs 配既可实现。




### Circuit Breaking

> 熔断

* 熔断, 是创建弹性微服务应用程序的重要模式。熔断能够使您的应用程序具备应对来自故障、潜在峰值和其他未知网络因素影响的能力。

* 通过配置 `DestinationRule` 中的 `trafficPolicy` 配置熔断策略

  * 可以配置`tcp`与`http`协议的条件, 如下为最大连接数为`1`, 超过`1`的时候就会触发 熔断机制

```yaml
apiVersion: networking.istio.io/v1beta1
kind: DestinationRule
metadata:
  name: nginx-destinationrule
spec:
  host: nginx-svc
  trafficPolicy:
    connectionPool:
      tcp:
        maxConnections: 1
      http:
        http1MaxPendingRequests: 1
        maxRequestsPerConnection: 1
    outlierDetection:
      consecutiveErrors: 1
      interval: 1s
      baseEjectionTime: 3m
      maxEjectionPercent: 100
```


* 部署一个 `fortio` 客户端 https://github.com/fortio/fortio

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: fortio
  labels:
    app: fortio
spec:
  containers:
    - name: fortio
      image: fortio/fortio
      imagePullPolicy: IfNotPresent
      args: ["server", "&"]
```


* 触发一下熔断


```sh
kubectl -n jicki exec fortio -c fortio -- /usr/bin/fortio load -c 2 -qps 0 -n 20 -loglevel Warning http://nginx-svc/



09:18:17 I logger.go:127> Log level is now 3 Warning (was 2 Info)
Fortio 1.17.0 running at 0 queries per second, 24->24 procs, for 20 calls: http://nginx-svc/
Starting at max qps with 2 thread(s) [gomax 24] for exactly 20 calls (10 per thread + 0)
09:18:17 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:18:17 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:18:17 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:18:17 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
Ended after 92.685514ms : 20 calls. qps=215.78
Aggregated Function Time : count 20 avg 0.0089015828 +/- 0.004691 min 0.001575415 max 0.02210334 sum 0.178031656
# range, mid point, percentile, count
>= 0.00157541 <= 0.002 , 0.00178771 , 5.00, 1
> 0.002 <= 0.003 , 0.0025 , 15.00, 2
> 0.006 <= 0.007 , 0.0065 , 25.00, 2
> 0.007 <= 0.008 , 0.0075 , 45.00, 4
> 0.008 <= 0.009 , 0.0085 , 55.00, 2
> 0.009 <= 0.01 , 0.0095 , 80.00, 5
> 0.011 <= 0.012 , 0.0115 , 85.00, 1
> 0.012 <= 0.014 , 0.013 , 90.00, 1
> 0.018 <= 0.02 , 0.019 , 95.00, 1
> 0.02 <= 0.0221033 , 0.0210517 , 100.00, 1
# target 50% 0.0085
# target 75% 0.0098
# target 90% 0.014
# target 99% 0.0216827
# target 99.9% 0.0220613
Sockets used: 6 (for perfect keepalive, would be 2)
Jitter: false
Code 200 : 16 (80.0 %)
Code 503 : 4 (20.0 %)
Response Header Sizes : count 20 avg 190.55 +/- 95.28 min 0 max 239 sum 3811
Response Body/Total Sizes : count 20 avg 255.2 +/- 10.03 min 250 max 276 sum 5104
All done 20 calls (plus 0 warmup) 8.902 ms avg, 215.8 qps
```


* 当使用 2个 连接的时候 只会触发 `Code 503 : 4 (20.0 %)` 


```sh
kubectl -n jicki exec fortio -c fortio -- /usr/bin/fortio load -c 5 -qps 0 -n 20 -loglevel Warning http://nginx-svc/



09:21:22 I logger.go:127> Log level is now 3 Warning (was 2 Info)
Fortio 1.17.0 running at 0 queries per second, 24->24 procs, for 20 calls: http://nginx-svc/
Starting at max qps with 5 thread(s) [gomax 24] for exactly 20 calls (4 per thread + 0)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
09:21:23 W http_client.go:793> Parsed non ok code 503 (HTTP/1.1 503)
Ended after 64.311585ms : 20 calls. qps=310.99
Aggregated Function Time : count 20 avg 0.01221322 +/- 0.007708 min 0.003108022 max 0.026839277 sum 0.244264407
# range, mid point, percentile, count
>= 0.00310802 <= 0.004 , 0.00355401 , 10.00, 2
> 0.004 <= 0.005 , 0.0045 , 25.00, 3
> 0.005 <= 0.006 , 0.0055 , 35.00, 2
> 0.009 <= 0.01 , 0.0095 , 40.00, 1
> 0.01 <= 0.011 , 0.0105 , 55.00, 3
> 0.011 <= 0.012 , 0.0115 , 60.00, 1
> 0.012 <= 0.014 , 0.013 , 70.00, 2
> 0.016 <= 0.018 , 0.017 , 80.00, 2
> 0.02 <= 0.025 , 0.0225 , 85.00, 1
> 0.025 <= 0.0268393 , 0.0259196 , 100.00, 3
# target 50% 0.0106667
# target 75% 0.017
# target 90% 0.0256131
# target 99% 0.0267167
# target 99.9% 0.026827
Sockets used: 14 (for perfect keepalive, would be 5)
Jitter: false
Code 200 : 9 (45.0 %)
Code 503 : 11 (55.0 %)
Response Header Sizes : count 20 avg 107.35 +/- 118.7 min 0 max 239 sum 2147
Response Body/Total Sizes : count 20 avg 264.15 +/- 12.31 min 250 max 276 sum 5283
All done 20 calls (plus 0 warmup) 12.213 ms avg, 311.0 qps

```


* 配置 5个 连接的时候 `Code 503 : 11 (55.0 %)`




### Mirroring

> 镜像流量

*  创建一个 vs 将默认流量都到 `nginx-svc-1` 中去.
  * 通过 `mirror` 标签镜像 `nginx-svc-1` 的流量 到 `nginx-svc-2` 中. (mirrorPercent 为镜像流量的百分比)
 

```yaml
apiVersion: networking.istio.io/v1beta1
kind: VirtualService
metadata:
  name: nginx-svc-vs
spec:
  # 客户端访问服务的地址
  hosts: 
  - nginx-svc
  # http协议
  http:
  # 如下为匹配条件
  - match:
    # http 头 包含如下信息
    - headers:
        # key = vs-user
        vs-user:
          # exact 完全匹配 value = jicki
          exact: jicki
    route:
    - destination:
        host: nginx-svc-3
  # 如下为匹配路由规则
  - route:
    - destination:
        # 匹配的服务版本或子集为 nginx-1
        host: nginx-svc-1
      weight: 100
    mirror:
      host: nginx-svc-2
    mirrorPercent: 100
```


* 查看 `nginx-1` 与 `nginx-2` 的日志 - 可以看出各自的不同. 

  * 重点注意这些被镜像的流量是 `即发即弃` 的, 就是说镜像请求的响应会被丢弃。


```
# nginx-1

127.0.0.1 - - [26/Oct/2021:08:24:20 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:23 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:24 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:25 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:25 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:25 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:26 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:26 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:27 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:27 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:27 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"
127.0.0.1 - - [26/Oct/2021:08:24:28 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "-"

```


```
# nginx-2

127.0.0.1 - - [26/Oct/2021:08:24:20 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:23 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:24 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:25 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:25 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:25 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:26 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:26 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:27 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:27 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:27 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"
127.0.0.1 - - [26/Oct/2021:08:24:28 +0000] "GET / HTTP/1.1" 200 14 "-" "curl/7.30.0" "172.18.16.252"

```










## FAQ 


### Istio Proxy Limit


* 默认 Istio-proxy 资源限制比较低 cpu = 500m  memroy = 512Mi 

  * 修改 k8s 资源文件 添加: `spec.template.metadata.annotations` 

```yaml

  annotations:
    sidecar.istio.io/proxyCPU: 1000m
    sidecar.istio.io/proxyLimitCPU: 2000m
    sidecar.istio.io/proxyMemory: 1024Mi
    sidecar.istio.io/proxyLimitMemory: 2048Mi

```













## Istio Upgrade


### 注意事项

* 升级 Istio 之前, 请确认是否支持升级 `istioctl manifest versions` 查看支持版本。


* 升级过程中可能发生流量中断。为了缩短流量中断时间, 请确保每个组件（Citadel 除外）至少运行有两个副本。同时, 确保 `PodDistruptionBudgets` 配置最小可用性为 1。

* 确保 istio `profile` 与 需要升级的版本所配置的 `profile` 一致。



### 升级步骤

* 1.&emsp;下载需要升级的 istio 版本 `curl -L https://istio.io/downloadIstio | ISTIO_VERSION=1.5.1 sh -`

* 2.&emsp;替换 `istioctl` 二进制文件, 或者 更改 `istio` 的环境变量PATH路径到新版本的目录中。

* 3.&emsp;如果配置了 `istioctl` 自动补全,还需要替换为 新的 自动补全脚本。

* 4.&emsp;使用新的 `istioctl` 导出新版本的 profile 文件 `istioctl profile dump demo > demo.yaml`

* 5.&emsp;修改 `demo.yaml` 文件, 将其中 `jwtPolicy` 身份验证机构修改为 `first-party-jwt`。Istio 将默认使用第三方令牌。

  * 5.1&emsp; 验证是否支持 第三方令牌。`kubectl get --raw /api/v1 | jq '.resources[] | select(.name | index("serviceaccounts/token"))'`

    * 5.1.1&emsp;jwtPolicy = `third-party-jwt` 使用第三方令牌 更安全 Istio 默认使用这个选项。

    * 5.1.2&emsp; jwtPolicy = `first-party-jwt` 使用第一方令牌 属性比较不安全。

* 6.&emsp;`istioctl upgrade -f demo.yaml` 命令进行升级。
  
* 7.&emsp;观察 `kubernets` 中 `istio-system` 的服务更新完成。

* 8.&emsp;重新注入环境中部署的服务, 用以更新注入数据。

  * 8.1&emsp;对于自动注入的情况可使用如下命令:

    * 8.1.1&emsp;`kubectl rollout restart deployment` 

    * 8.1.2&emsp;`kubectl rollout restart statefulset`

    * 8.1.3&emsp;`kubectl rollout restart daemonset`

  * 8.2&emsp;对于手动注入的情况( 需要重新 apply 一下服务) :

    * 8.2.1&emsp;`kubectl apply -f <(istioctl kube-inject -f nginx-test.yaml)`

* 9.&emsp;检查升级, 执行 `istioctl version` 检查

  * 9.1&emsp;`client version` 版本是否为新版本。

  * 9.2&emsp;`control plane version` 版本是否为新版本。

  * 9.3&emsp;`data plane version` 中是否全部 `proxies` 都为新版本。


## Istio UnInstall

* 卸载 Istio 的话其实就是讲 `kubernetes` 所安装到的组件 `delete` 掉就可以了

```
# 可以直接运行istioctl 命令加上 kubectl delete 命令组合删除

istioctl manifest generate --set profile=demo |kubectl delete -f -


```





  [1]: https://jicki.cn/img/posts/istio/kiali.png
  [2]: https://jicki.cn/img/posts/istio/kiali-1.png
  [3]: https://jicki.cn/img/posts/istio/kiali-2.png
  [4]: https://jicki.cn/img/posts/istio/kiali-3.png
  [5]: https://jicki.cn/img/posts/istio/kiali-4.png
  [6]: https://jicki.cn/img/posts/istio/istio-proxy.png
  [7]: https://jicki.cn/img/posts/istio/istio.svg
  [8]: https://jicki.cn/img/posts/istio/discovery.svg
  [9]: https://jicki.cn/img/posts/istio/virtualservice.png

