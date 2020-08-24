# Kubernetes 包管理 Helm



# Helm

*  Helm 是 Kubernetes 的包管理器, 可以帮我们简化 kubernetes 应用部署操作, 在多项目的环境下, 应用的部署与管理就会变的频繁与复杂. Helm 通过包的方式, 可简化应用的版本发布、管理、控制.  


* Helm 本质是在 Kubernetes 的应用管理上模板化, 能动态的生成如`Deployment`、`Service` 等的 yaml 资源文件.


* Helm 有三个重要概念

  * `chart` :是创建一个应用的信息集合, 既一个应用所包含的所有需要部署的服务如`deployment`、`service`、`pv`、`pvc` 等的各种 kubernetes 对象的配置模板、参数定义、依赖关系、文档、帮助信息等. `chart` 是应用部署的自包含逻辑单元, 类似于 `Yum` 中的软件rpm安装包. 

  * `reporitory` :是`chart` 的仓库, Helm v2 中需要一个 HTTP 服务器存放 `Charts` 包. Helm v3 中 可以将 Docker 私有仓库当做 `chart` 仓库.

  * `release` :是`chart` 的运行实例, 代表了一个正在运行的应用. 当 `chart` 被安装到 kubernetes 集群中时就生成一个 `release`. 每执行一次 `chart` 安装就会生成一个 `release`.  


---

## Helm v2


* 注: 当前 2020年8月 Helm v2 系列发布了 v2.16.10 版本， 这是 Helm v2 的最后一个 bugfix 版本，此后不会再为 Helm v2 提供错误修复。并且在三个月后，将停止为 Helm v2 提供安全补丁。届时， Helm v2 也就完全废弃，不会再去维护了。





### Helm v2 流程图


{{< figure src="/img/posts/helm/helm-flow.png" >}}


---


* Helm - 客户端 Helm Client 负责 `chart` 和 `release` 的创建、管理, 通过 gRPC 与 Tiller 服务端进行交互.

* Tiller - 服务端 运行在 Kubernetes 中, 处理 Helm Client 的请求, 通过 REST、JSON 与 kubernetes API Server 进行交互.






### 客户端部署


>  安装依赖


```shell
# 首先要在所有 node 里安装 socat 软件

yum -y install socat

```

---


> 下载 helm 二进制文件

```shell

wget https://get.helm.sh/helm-v2.16.10-linux-amd64.tar.gz

tar zxvf helm-v2.16.10-linux-amd64.tar.gz 

cd linux-amd64/

mv helm /usr/local/bin/


# 验证服务

[root@kubernetes ~]# helm version

Client: &version.Version{SemVer:"v2.16.10", GitCommit:"bceca24a91639f045f22ab0f41e47589a932cf5e", GitTreeState:"clean"}
Error: could not find tiller

```


### 部署 Helm Tiller


* 在 Kubernetes 部署 Tiller 需要先做配置一个 认证 rbac .


```shell
# 创建 rbac 并修改为 admin 权限

kubectl create serviceaccount --namespace=kube-system tiller

kubectl create clusterrolebinding tiller-cluster-rule --clusterrole=cluster-admin --serviceaccount=kube-system:tiller

kubectl patch deploy --namespace=kube-system tiller-deploy -p '{"spec":{"template":{"spec":{"serviceAccount":"tiller"}}}}'

```

---


* Init Helm


```shell

root@kubernetes:/opt/helm# helm init --skip-refresh

Creating /root/.helm
Creating /root/.helm/repository
Creating /root/.helm/repository/cache
Creating /root/.helm/repository/local
Creating /root/.helm/plugins
Creating /root/.helm/starters
Creating /root/.helm/cache/archive
Creating /root/.helm/repository/repositories.yaml
Adding stable repo with URL: https://kubernetes-charts.storage.googleapis.com
Adding local repo with URL: http://127.0.0.1:8879/charts
$HELM_HOME has been configured at /root/.helm.

Tiller (the Helm server-side component) has been installed into your Kubernetes Cluster.

Please note: by default, Tiller is deployed with an insecure 'allow unauthenticated users' policy.
To prevent this, run `helm init` with the --tiller-tls-verify flag.
For more information on securing your installation see: https://v2.helm.sh/docs/securing_installation/
Happy Helming!
```


* 查看服务

```shell
root@kubernetes:/opt/helm# kubectl get pods --all-namespaces

NAMESPACE     NAME                                 READY   STATUS    RESTARTS   AGE
kube-system   coredns-66bff467f8-n7bc5             1/1     Running   0          16m
kube-system   coredns-66bff467f8-nln8j             1/1     Running   0          16m
kube-system   etcd-kubernetes                      1/1     Running   0          16m
kube-system   kube-apiserver-kubernetes            1/1     Running   0          16m
kube-system   kube-controller-manager-kubernetes   1/1     Running   0          16m
kube-system   kube-flannel-ds-amd64-tbxm6          1/1     Running   0          14m
kube-system   kube-proxy-g2js8                     1/1     Running   1          16m
kube-system   kube-scheduler-kubernetes            1/1     Running   0          16m
kube-system   tiller-deploy-55f5dfddc9-lqv9d       1/1     Running   0          80s
```

* 查看 version

```shell

root@kubernetes:/opt# helm version

Client: &version.Version{SemVer:"v2.16.10", GitCommit:"bceca24a91639f045f22ab0f41e47589a932cf5e", GitTreeState:"clean"}
Server: &version.Version{SemVer:"v2.16.10", GitCommit:"bceca24a91639f045f22ab0f41e47589a932cf5e", GitTreeState:"clean"}

```

---


### 添加/删除 repo

```
# helm 的 repo 默认是使用 谷歌的 repo ，国内访问不到，修改为 阿里云 的 repo

[root@kubernetes ~]# helm repo list

NAME    URL                                             
stable  https://kubernetes-charts.storage.googleapis.com
local   http://127.0.0.1:8879/charts 



[root@kubernetes ~]# helm repo remove stable

"stable" has been removed from your repositories



[root@kubernetes ~]# helm repo add stable https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts

"stable" has been added to your repositories


[root@kubernetes ~]# helm repo update

Hang tight while we grab the latest from your chart repositories...
...Skip local chart repository
...Successfully got an update from the "stable" chart repository
Update Complete. ⎈ Happy Helming!⎈ 



[root@kubernetes ~]# helm repo list

NAME    URL                                                   
local   http://127.0.0.1:8879/charts                          
stable  https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts


```


---

### Helm v2 自定义模板


* 自定义 Helm 模板 需要包含 `Chart.yaml` 文件、`templates` 模板目录.

  * `Chart.yaml` 文件是描述文件, 用于描述该应用的信息. 必须包含 `name` 与 `version` .

  * `templates` 模板目录, 用于存放 kubernetes 资源清单. 目录必须为`templates`此名称.



---


> 基础的例子


* `Chart.yaml`


```yaml
# api 版本 helm 2 只支持 v1 版本 (必填)
apiVersion: v1

# chart 名称 (必填)
name: my-nginx

# chart 版本 (必填)
version: 1.0.0

# 约束此 Chart 支持的 kubernetes 版本, 如果不支持会失败 (可选)
kubeVersion: >= 1.18.0

# chart 的描述信息 (可选)
description: "my-nginx"

# chart 关键词, 用于 chart 搜索 (可选)
keywords: 
  - mynginx
  - nginx

# 主页连接 (可选)
home: "https://jicki.cn"

# 源码路径,一般为 github 地址 (可选)
sources: "https://github.com/jicki"

# 此项目维护者信息(可选)
maintainers:
  - name: 小炒肉
    email: jicki@qq.com
    url: "https://jicki.cn"

# 默认都使用 gotpl (可选)
engine: gotpl 

# icon 地址 (可选)
icon: "https://jicki.cn/images/favicon.ico"

# App 的版本 (可选)
appVersion: 1.19.2

# 标注是否为过期 (可选)
deprecated:

# Tiller 版本. (可选)
tillerVersion: > 2.0.0
```


* `templates/deployment.yaml`


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-dm
  labels:
    app: nginx
spec:
  replicas: 2
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



* `templates/service.yaml`



```yaml
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



---

> 创建自定义模板的应用



* 安装自定义的模板

  * 在 `Chart.yaml` 所在目录下执行 `helm install -n my-nginx .` 命令
  
    * `-n` 参数是指定安装的应用的 `release` 名称. 


```shell

root@kubernetes:/opt/helm/nginx# helm install -n my-nginx .

NAME:   my-nginx
LAST DEPLOYED: Tue Aug 18 09:55:12 2020
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME      READY  UP-TO-DATE  AVAILABLE  AGE
nginx-dm  0/2    2           0          1s

==> v1/Pod(related)
NAME                       READY  STATUS             RESTARTS  AGE
nginx-dm-5f7cb96cd8-qjttm  0/1    ContainerCreating  0         1s
nginx-dm-5f7cb96cd8-rdrsh  0/1    ContainerCreating  0         1s

==> v1/Service
NAME       TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)  AGE
nginx-svc  ClusterIP  10.254.143.201  <none>       80/TCP   1s

```


* 查看服务


```shell

root@kubernetes:/opt/helm/nginx# helm ls

NAME    	REVISION	UPDATED                 	STATUS  	CHART         	APP VERSION	NAMESPACE
my-nginx	1       	Tue Aug 18 09:55:12 2020	DEPLOYED	my-nginx-1.0.0	           	default


```


```shell
root@kubernetes:/opt/helm/nginx# kubectl get pods,svc

NAME                            READY   STATUS    RESTARTS   AGE
pod/nginx-dm-5f7cb96cd8-qjttm   1/1     Running   0          2m2s
pod/nginx-dm-5f7cb96cd8-rdrsh   1/1     Running   0          2m2s

NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)   AGE
service/kubernetes   ClusterIP   10.254.0.1       <none>        443/TCP   92m
service/nginx-svc    ClusterIP   10.254.143.201   <none>        80/TCP    2m2s

```


* 测试访问


```shell
root@kubernetes:/opt/helm/nginx# curl -I 10.254.143.201

HTTP/1.1 200 OK
Server: nginx/1.19.2
Date: Tue, 18 Aug 2020 09:57:48 GMT
Content-Type: text/html
Content-Length: 612
Last-Modified: Tue, 11 Aug 2020 15:16:45 GMT
Connection: keep-alive
ETag: "5f32b65d-264"
Accept-Ranges: bytes

```

---

> 更新应用


* 更新应用 使用 `upgrade` 命令

  * `helm upgrade my-nginx .`  

  * 执行更新后 `REVISION` 属性会 +1  

```shell
root@kubernetes:/opt/helm/nginx# helm upgrade my-nginx .

Release "my-nginx" has been upgraded.
LAST DEPLOYED: Tue Aug 18 10:05:13 2020
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME      READY  UP-TO-DATE  AVAILABLE  AGE
nginx-dm  2/2    1           2          10m

==> v1/Pod(related)
NAME                       READY  STATUS             RESTARTS  AGE
nginx-dm-55c68cdfc9-7fmh2  0/1    ContainerCreating  0         0s
nginx-dm-5f7cb96cd8-qjttm  1/1    Running            0         10m
nginx-dm-5f7cb96cd8-rdrsh  1/1    Running            0         10m

==> v1/Service
NAME       TYPE       CLUSTER-IP      EXTERNAL-IP  PORT(S)  AGE
nginx-svc  ClusterIP  10.254.143.201  <none>       80/TCP   10m

```


```shell
root@kubernetes:/opt/helm/nginx# helm ls

NAME    	REVISION	UPDATED                 	STATUS  	CHART         	APP VERSION	NAMESPACE
my-nginx	2       	Tue Aug 18 10:05:13 2020	DEPLOYED	my-nginx-1.0.0	           	default
```


---

> 查看操作历史


* 查看应用操作历史 `history`

  * `helm history my-nginx`


```shell
root@kubernetes:/opt/helm/nginx# helm history my-nginx

REVISION	UPDATED                 	STATUS    	CHART         	APP VERSION	DESCRIPTION
1       	Tue Aug 18 09:55:12 2020	SUPERSEDED	my-nginx-1.0.0	           	Install complete
2       	Tue Aug 18 10:05:13 2020	DEPLOYED  	my-nginx-1.0.0	           	Upgrade complete
```


---

> 回滚应用


* 将应用回滚到之前的版本 `rollback`

  * `helm rollback my-nginx 1` 

    * `1` 为 REVISION 号, 使用 `helm history xx` 可查看应用的更新信息
  

```shell
root@kubernetes:/opt/helm/nginx# helm rollback my-nginx 1

Rollback was a success.

```

* 查看信息


```shell
root@kubernetes:/opt/helm/nginx# helm history my-nginx

REVISION	UPDATED                 	STATUS    	CHART         	APP VERSION	DESCRIPTION
1       	Tue Aug 18 09:55:12 2020	SUPERSEDED	my-nginx-1.0.0	           	Install complete
2       	Tue Aug 18 10:05:13 2020	SUPERSEDED	my-nginx-1.0.0	           	Upgrade complete
3       	Tue Aug 18 10:18:37 2020	DEPLOYED  	my-nginx-1.0.0	           	Rollback to 1

```



---


#### 自定义模板之 变量

*  创建一个 `values.yaml` 文件,  `values.yaml` 文件与 `Chart.yaml` 在同级目录下.

  * 在 `values.yaml` 文件中定义对应的变量 ( `key/value` 值). 在 templates 下的 kubernetes 资源清单文件中引用定义的 变量.

  * 在 `helm install --values xxx.yaml` 也可以指定 变量文件, 变量优先级 高于 默认 `values.yaml` 中定义的变量.

  * 在 `helm install --set image.tag='2.0'` 也可以定义 变量, 变量优先级 最高。 高于 `--values`  和 `values.yaml` 


---


* `Chart.yaml`

```shell
name: myapp
version: v1

```




* `values.yaml` 


```shell
image:
  repository: jicki/myapp
  tag: 'v1'

```




* `templates/deployment.yaml` 


```shell
apiVersion: apps/v1
kind: Deployment
metadata:
  name: myapp
  labels:
    app: myapp
spec:
  replicas: 2
  selector:
    matchLabels:
      app: myapp
  template:
    metadata:
      labels:
        app: myapp
        version: v1
    spec:
      containers:
        - name: myapp
          # {{ }} 表示引用变量, Values 表示 values.yaml 文件本身
          image: {{ .Values.image.repository }}:{{ .Values.image.tag }}
          imagePullPolicy: IfNotPresent
          ports:
            - containerPort: 80
              name: http
```


* `templates/service.yaml`


```shell
apiVersion: v1
kind: Service
metadata:
  name: myapp-svc
  labels:
    app: myapp
spec:
  ports:
    - port: 80
      name: http
      targetPort: 80
      protocol: TCP
  selector:
    app: myapp

```



---

* 创建 应用 `release` myapp


```shell

root@kubernetes:/opt/helm/myapp# helm install -n myapp .

NAME:   myapp
LAST DEPLOYED: Wed Aug 19 03:20:04 2020
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME   READY  UP-TO-DATE  AVAILABLE  AGE
myapp  0/2    2           0          0s

==> v1/Pod(related)
NAME                   READY  STATUS             RESTARTS  AGE
myapp-846dbc5c6-2c58f  0/1    ContainerCreating  0         0s
myapp-846dbc5c6-qb7jv  0/1    ContainerCreating  0         0s

==> v1/Service
NAME       TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)  AGE
myapp-svc  ClusterIP  10.254.90.247  <none>       80/TCP   0s

```

---

* 测试访问

```shell
root@kubernetes:/opt/helm/myapp# curl 10.254.90.247

Hello MyApp | Version: v1 | <a href="hostname.html">Pod Name</a>

```


---

> 通过变量更新 release 应用 


1. 可以通过修改 `values.yaml` 文件的版本号进行更新.

2. 可以通过 `--set key:value` 的形式进行更新.



* 修改 `values.yaml` 

```shell
image:
  repository: jicki/myapp
  tag: 'v2'
```


* 执行更新操作

```shell

root@kubernetes:/opt/helm/myapp# helm upgrade myapp .


Release "myapp" has been upgraded.
LAST DEPLOYED: Wed Aug 19 03:30:24 2020
NAMESPACE: default
STATUS: DEPLOYED

RESOURCES:
==> v1/Deployment
NAME   READY  UP-TO-DATE  AVAILABLE  AGE
myapp  2/2    1           2          10m

==> v1/Pod(related)
NAME                    READY  STATUS             RESTARTS  AGE
myapp-554ff5979c-5k8w5  0/1    ContainerCreating  0         0s
myapp-846dbc5c6-2c58f   1/1    Running            0         10m
myapp-846dbc5c6-qb7jv   1/1    Running            0         10m

==> v1/Service
NAME       TYPE       CLUSTER-IP     EXTERNAL-IP  PORT(S)  AGE
myapp-svc  ClusterIP  10.254.90.247  <none>       80/TCP   10m

```



---

* 测试访问


```shell
root@kubernetes:/opt/helm/myapp# helm history myapp

REVISION	UPDATED                 	STATUS    	CHART   	APP VERSION	DESCRIPTION
1       	Wed Aug 19 03:20:04 2020	SUPERSEDED	myapp-v1	           	Install complete
2       	Wed Aug 19 03:30:24 2020	DEPLOYED  	myapp-v1	           	Upgrade complete

```


```shell
root@kubernetes:/opt/helm/myapp# curl 10.254.90.247

Hello MyApp | Version: v2 | <a href="hostname.html">Pod Name</a>
```



### helm v2 命令


> Helm 一些常用命令


* 由于本文主要讲的是 v3 版本, 所以 v2 版本的命令就只记录了一些最最基础的.

---


* `install` 部署应用 `release`.

  * `helm install .` 创建一个随机 release 名称的自定义应用,  `.` 表示当前应用目录.

  * `helm install -n release_name .`  , `-n / --name` 为指定 release 名称.

  * `helm install -n mysql stable/mysql` 创建 repo 仓库中的应用,  `stable` 表示 repo 仓库名称.

  * `helm install --dry-run -n release_name .`  测试应用, 不会部署应用, 显示对应的 kubernetes 资源清单以及测试部署信息.

---

* `list / ls` 列出已经部署的应用 `release` .
  
  * `helm list` / `helm ls` 不包含 `DELETE` 状态的应用.

  * `helm list -a` 列出包含 所有状态 的所有应用

---

* `status` 查看指定 release 的状态

  * `helm status release_name`



---


* `delete / del` 删除指定 release 应用. 只删除 历史 并非完全删除 release , 可用 rollback 回滚.

  * `helm delete release_name`

  * `helm delete release_name --purge`  完全删除 release 应用.



---

* `history` 列出指定 release 应用的 所有 `REVISION` 版本.

  * `helm history release_name`


---

* `rollback` 回滚指定 release 应用的指定 `REVISION` 版本.

  * `helm rollback release_name 1`


---


---

### Helm v2 FAQ


* `Error: no available release name found`

  * 出现这个错误是因为 rbac 的问题。


---



### Helm v2 升级 Helm v3



{{< figure src="/img/posts/helm/helm-2to3.png" >}}




* 使用 helm v3 的 `plugin`  `helm-2to3` 进行升级

---

* 安装 `plugin`

  * 安装完毕以后 helm 命令下会有一个 `2to3` 参数选项.

```shell
helm3 plugin install https://github.com/helm/helm-2to3

```


---

* 将 helm v2 的配置文件, 转换配置文件为 helm v3 , 转换包含:

  * `Chart`

  * `repo`

  * `Plugins`


```shell
helm3 2to3 move config


# 输出如下信息为成功

Helm v2 configuration was moved successfully to Helm v3 configuration.

```

---


* 将 helm v2 中创建的 `release` 迁移到 helm v3 中

  * 每个 `release` 都需要进行迁移.

```shell
helm3 2to3 convert release_name 

```


---


* 清理 helm v2 `release` .

  * 执行完后, `Tiller Pod` 会被删除, 并且 `kube-system` 命名空间中 `configmaps` 历史版本信息也会被清理.

```shell
helm3 2to3 cleanup

```


---


## Helm v3


{{< figure src="/img/posts/helm/helm3.png" >}}


---

### Helm v3 新特性


1. Helm v3 移除了 `Tiller`. -- Helm v2 是 C/S 架构, 主要分为客户端 `helm` 和服务端 `Tiller`.  `Tiller` 主要用于在 Kubernetes 集群中管理各种应用发布的版本, 在 Helm v3 中移除了 `Tiller`, 版本相关的数据直接存储在了 Kubernetes 中.

---

2. Helm v2 中通过 `Tiller` 进行管理 Kubernetes 集群中应用, 而 `Tiller` 需要管理员的 `ClusterRole` 才能创建使用, 这就是一直被诟病的安全性问题. 而 Helm v3 中通过Helm 管理 Kubernetes 集群中的应用, Helm 使用 `KUBECONFIG` 配置权限 与 `kubectl` 上下文相同的访问权限.

---

3. Helm v2 在 `install` 时如果不指定 `release` 名称, 会随机生成一个, Helm v3 中 `install` 必须强制指定 `release` 名称, 或者使用 `--generate-name` 参数.

---

4. Helm v3 中 `release` 位于命名空间中, 既 不同的 namespace 可以使用相同的 `release` 名称.

---

5. `requirements.yaml` 文件合并到 `Chart.yaml` 文件中.

---

6. Helm v3 中 使用 `JSON Schema` 验证 charts 的 Values .

---

7. Helm v3 中 支持将 `chart`  Push 到 Docker 镜像仓库中.

---

8. 移动 `helm serve` , Helm v2 中可以通过 helm serve 来启动一个简单的 HTTP 服务, 用于托管 local repo 中的 `chart`. Helm v3 移除了此命令, 因为 Helm v3 中可以将`chart` 推送到 Docker 镜像仓库中.


---

### Helm v3 流程图



{{< figure src="/img/posts/helm/helm-v3-flow.jpeg" >}}



---



### 部署 Helm v3


> 下载 helm v3 二进制文件


```shell
# 下载
root@kubernetes:/opt/helm# wget https://get.helm.sh/helm-v3.3.0-linux-amd64.tar.gz



# 解压
root@kubernetes:/opt/helm# tar zxvf helm-v3.3.0-linux-amd64.tar.gz

linux-amd64/
linux-amd64/README.md
linux-amd64/helm
linux-amd64/LICENSE

# 切换目录
root@kubernetes:/opt/helm# cd linux-amd64/


# 剪切到 bin 目录下
root@kubernetes:/opt/helm/linux-amd64# mv helm /usr/local/bin/


# 授权文件
root@kubernetes:/opt/helm/linux-amd64# chmod a+x /usr/local/bin/helm



# 验证安装
root@kubernetes:/opt/helm# helm version

version.BuildInfo{Version:"v3.3.0", GitCommit:"8a4aeec08d67a7b84472007529e8097ec3742105", GitTreeState:"dirty", GoVersion:"go1.14.7"}

```


>  配置 Helm



---
* Helm 默认情况下不需要配置就可以直接使用. 

* Helm 会使用到的一直参数: 

  * `--kubeconfig` : 指定 kubernetes 的 config 文件, 不配置默认会读取 `$HOME/.kube/config`

  * `--kube-context` : 配置 kubernetes 上下文.  ( 在 kubeconfig 文件中 context 选项中的 use 值 ).


---


### Helm v3 命令



#### env 命令

> helm env 命令

* `helm env` 命令主要用于输出 helm 所在本地环境的环境变量.


```shell
root@kubernetes:/opt/helm# helm env

# helm 二进制文件名称
HELM_BIN="helm"
# helm 的缓存 目录
HELM_CACHE_HOME="/root/.cache/helm"
# helm 配置文件 目录
HELM_CONFIG_HOME="/root/.config/helm"
# helm 数据目录
HELM_DATA_HOME="/root/.local/share/helm"
# 是否开启 debug 模式
HELM_DEBUG="false"
# kubernetes 的 kube-apiserver
HELM_KUBEAPISERVER=""
# kubernetes 的 kube-context
HELM_KUBECONTEXT=""
# kubernetes 的 用户 Token 
HELM_KUBETOKEN=""
# helm 默认命名空间
HELM_NAMESPACE="default"
# helm 插件目录
HELM_PLUGINS="/root/.local/share/helm/plugins"
# helm 注册中心的配置文件
HELM_REGISTRY_CONFIG="/root/.config/helm/registry.json"
# helm 仓库的缓存目录
HELM_REPOSITORY_CACHE="/root/.cache/helm/repository"
# helm 仓库的配置文件
HELM_REPOSITORY_CONFIG="/root/.config/helm/repositories.yaml"

```

---

* 可通过修改如下 变量 修改 `helm env` .


| Name                                 | Description                                                                       |
|--------------------------------------|-----------------------------------------------------------------------------------|
| `$HELM_CACHE_HOME`                   | set an alternative location for storing cached files.                             |
| `$HELM_CONFIG_HOME`                  | set an alternative location for storing Helm configuration.                       |
| `$HELM_DATA_HOME`                    | set an alternative location for storing Helm data.                                |
| `$HELM_DRIVER`                       | set the backend storage driver. Values are: configmap, secret, memory, postgres   |
| `$HELM_DRIVER_SQL_CONNECTION_STRING` | set the connection string the SQL storage driver should use.                      |
| `$HELM_NO_PLUGINS`                   | disable plugins. Set `HELM_NO_PLUGINS=1` to disable plugins.                      |
| `$KUBECONFIG`                        | set an alternative Kubernetes configuration file (default "~/.kube/config")       |



---

---

#### repo 命令

> helm repo 命令


* `helm repo list` : 显示本地配置的 repo .

  * `-o` : 格式化输出, 支持 `table|json|yaml`. 例: `helm repo list -o json`

* `helm repo index chart` : 将本地的 chart 包生成一个 charts 索引文件.

* `helm repo add` : 添加 repo 到本地列表中. 例: `helm repo add aliyun https://kubernetes.oss-cn-hangzhou.aliyuncs.com/charts`
  
  * `--ca-file` : 指定 HTTPS 的 repo  tls 的 ca 证书文件.

  * `--cert-file` : 指定 HTTPS 的 repo tls 的 cert 证书文件.

  * `--key-file` : 指定 HTTPS 的 repo tls 的 key 证书文件.

  * `--insecure-skip-tls-verify` : 跳过 repo tls 检测.

  * `--no-update` : 只添加 repo 但是不更新 index 索引文件. 默认添加以后会自动更新 index 索引文件.

  * `--username` :  指定 repo 所需要的 用户名.

  * `--password` :  指定 repo 所需要的 密码. 

* `helm repo remove` : 删除本地列表中指定的 repo. 例: `helm repo remove aliyun`

* `helm repo update` : 更新本地列表中的 repo 的 index 索引文件. 




---

#### search 命令

> helm search 命令


* `helm search hub` : 从 docker 仓库中查找可用的 `chart`. 默认会从 `https://hub.helm.sh/charts` 官方 hub 中搜索.

  * `--endpoint` : 默认地址是 `https://hub.helm.sh/charts` 官方 hub . 通过此命令可指定自己的 hub .

  * `-o` : 格式化输出, 支持 `table、json、yaml` .


* `helm search repo` : 从本地配置的 repo 中查找可用的 `chart` . `helm repo list` 可查看本地配置的 repo .

  * `-o` : 格式化输出, 支持 `table、json、yaml` .

  * `--regexp` : 正则过滤 search . 例: `helm search repo --regexp ".*sql$"`

  * `--versions` : 列出 repo 仓库中所有的版本. 默认只会列出最新版本.

  * `--version` : 指定 `CHART` 版本搜索.

  * `--devel` : 会显示 开发版本的 `chart`.


---

#### install 命令

> helm install 命令


* helm 支持四种安装方式. 

  1. 通过指定 repo 安装. -- `helm install mysql -n mysql stable/mysql`

  2. 通过指定 chart 的 tgz 包 安装. -- `helm install  mysql -n mysql mysql-2.3.tgz`

  3. 通过自定义 chart 目录安装. -- `helm install mysql -n mysql .`

  4. 通过指定 url 进行安装. -- `helm install mysql -n mysql http://127.0.0.1:8879/charts/mysql`

---

* `helm install ` 的一些常用参数:

  * `--dry-run` : 测试安装, 不会实际部署.

  * `--no-hooks` : 不触发 hooks 的操作.

    * `Hooks` 包含如下:
      * `pre-install`: 预安装, 在模板渲染后, kubernetes 创建任何资源之前执行.
      * `post-install`: 安装后, 在所有 kubernetes 资源安装到集群后执行.
      * `pre-delete`: 预删除在从 kubernetes 删除任何资源之前执行删除请求.
      * `post-delete`: 删除后, 删除所有 release 的资源后执行.
      * `pre-upgrade`: 升级前, 在模板渲染后, 但在任何资源升级之前执行.
      * `post-upgrade`: 升级后, 在所有资源升级后执行.
      * `pre-rollback`: 预回滚, 在模板渲染后, 在任何资源回滚之前执行.
      * `post-rollback`: 回滚后, 在修改所有资源后执行回滚请求. 
      * `crd-install`: 预检查, 在运行其他检查之前添加 CRD 资源, 只能用于 chart 中其他的资源清单定义的 CRD 资源.
   
  * `--replace` : 重用被 `delete` 删除的 同名 `release`. 重用现有 `release` 并替换其资源, 而非重新创建. 

  * `--wait` : 等待应用 Successful 后才返回, 既 kubernetes 资源都 `Running`. 如果出错会在 `--timeout` 时间后返回. 

  * `--timeout ` : 设置超时时间 (default 5m0s).

  * `--generate-name` : 随机命名 `release` ,  默认必须指定 `release` 名称.

  * `--name-template` : 指定包含 `release` 命名的 模板. 

  * `--description` : 添加 `release` 备注, 说明.

  * `--devel` :  安装 `Beta` 版的 Chart.

  * `--dependency-update` : 更新依赖, 将 Helm v2 版本中的 `requirement.yaml` 更新到 `Chart.yaml` 中.

  * `--atomic` : 原子性, 既创建 `release` 失败就直接删除.

  * `--skip-crds` : 跳过 CRD 资源的添加.  

  * `--render-subchart-notes` : 显示 subchart notes .

  * `-o --output [table|json|yaml|` : 以什么格式输出安装结果.

  * `--values -f values.yaml` : 指定 `values` 变量的文件.

  * `--set value` : 设置 变量 `key=value` 格式. 

  * `--set-string` : 设置字符串类型的变量.

  * `--set-file` : 设置变量等于文件, `key=script.sh` .

  * `--version ` : 指定 `chart` 的安装版本.

  * `--verify` : 校验需要安装的 `chart` 完整性.

    * `--keyring` : 指定校验 `chart` 需要使用的 gpg 文件.

  * `--repo` : 指定安装的 repo 库.
 
    * `--username` : 配置 repo 库 的用户名.
    * `--password` : 配置 repo 库 的密码. 



---

#### uninstall 命令

> helm uninstall 命令


* `helm uninstall / un / del / delete  release_name ` : 删除已部署的 `release` .  

  * `-n` : 指定命名空间 ( namespace ) .

  * `--dry-run` : 测试删除, 不会实际删除.

  * `--no-hooks` : 不触发 hooks 的操作.

  * `--keep-history` : 删除后会在 `kubernetes 的 secret` 中保留历史记录 ( kubectl get secret ).  helm v2 的数据保存于 helm 数据中, 而 helm v3 的数据会保存在 kubernetes 中.
 
  * `--timeout 300s` : 设置删除超时时间. (default 5m0s)


---

#### status 命令

> helm status 命令

* `helm status release_name`  查看具体 `release` 的状态信息.

  * ` -o, --output` : 指定输出格式, `table|json|yaml` 默认为 table .

  * `--revision` : 指定查看 revision 版本.

---


#### list 命令


> helm list 命令

* `helm list` : 显示已经部署的 `release` .

  * `-n` :  指定 namespace 命名空间的 `release`. 不指定为 `defaule`命名空间下的 `release`. 

  * `--all-namespaces` : 显示所有 命名空间下的 `release`.

  * `helm list --all` : 列出所有 状态的 `release` . 默认不会列出 `uninstalled` 的 `release`.

    * `helm list --deployed` : 列出 deployed 状态的 `release`.

    * `helm list --failed` : 列出 failed 状态的 `release`.

    * `helm list --superseded` : 列出 superseded  状态的 `release`.

    * `helm list --uninstalled` : 列出 uninstalled 状态的 `release`.

    * `helm list --uninstalling` : 列出 uninstalling 状态的 `release`.

  * `helm list --data` :  按照 `UPDATED` 时间排序.   

  * `helm list --max int`: 最多列出多少个 `release`.

  * `helm list --offset int`: 从第几个 `release` 开始列出, 从 0 开始.

  * `helm list --filter ` : 过滤规则, 支持正则表达式.  例: `helm list --filter ".*app"` 

  * `helm list -o ` : 格式化输出, 支持 `table、json、yaml`. 例: `helm list -o json`


---

#### create 命令

> helm create 命令

  * `helm create chart_name` 创建一个 chart 所需要用到的模板目录.

    * `--starter` : 指定 chart 模板 进行创建. 例: `helm create myredis --starter /tmp/redis`

---

```shell
# 会初始化如下文件以及目录结构

root@kubernetes:/opt/helm/app# tree .

.
├── Chart.yaml
├── charts
├── templates
│   ├── NOTES.txt
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── hpa.yaml
│   ├── ingress.yaml
│   ├── service.yaml
│   ├── serviceaccount.yaml
│   └── tests
│       └── test-connection.yaml
└── values.yaml

```

---

* `Chart.yaml` 文件

```yaml
apiVersion: v2
name: app
description: A Helm chart for Kubernetes
type: application
version: 0.1.0
appVersion: 1.16.0
```


---

* `values.yaml` 文件


```yaml
replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  tag: ""

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  create: true
  annotations: {}
  name: ""

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  annotations: {}
    # kubernetes.io/ingress.class: nginx
    # kubernetes.io/tls-acme: "true"
  hosts:
    - host: chart-example.local
      paths: []
  tls: []
  #  - secretName: chart-example-tls
  #    hosts:
  #      - chart-example.local

resources: {}
  # limits:
  #   cpu: 100m
  #   memory: 128Mi
  # requests:
  #   cpu: 100m
  #   memory: 128Mi

autoscaling:
  enabled: false
  minReplicas: 1
  maxReplicas: 100
  targetCPUUtilizationPercentage: 80
  # targetMemoryUtilizationPercentage: 80

nodeSelector: {}

tolerations: []

affinity: {}

```

---

* `templates/deployment.yaml` 文件


```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "app.fullname" . }}
  labels:
    {{- include "app.labels" . | nindent 4 }}
spec:
{{- if not .Values.autoscaling.enabled }}
  replicas: {{ .Values.replicaCount }}
{{- end }}
  selector:
    matchLabels:
      {{- include "app.selectorLabels" . | nindent 6 }}
  template:
    metadata:
    {{- with .Values.podAnnotations }}
      annotations:
        {{- toYaml . | nindent 8 }}
    {{- end }}
      labels:
        {{- include "app.selectorLabels" . | nindent 8 }}
    spec:
      {{- with .Values.imagePullSecrets }}
      imagePullSecrets:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      serviceAccountName: {{ include "app.serviceAccountName" . }}
      securityContext:
        {{- toYaml .Values.podSecurityContext | nindent 8 }}
      containers:
        - name: {{ .Chart.Name }}
          securityContext:
            {{- toYaml .Values.securityContext | nindent 12 }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag | default .Chart.AppVersion }}"
          imagePullPolicy: {{ .Values.image.pullPolicy }}
          ports:
            - name: http
              containerPort: 80
              protocol: TCP
          livenessProbe:
            httpGet:
              path: /
              port: http
          readinessProbe:
            httpGet:
              path: /
              port: http
          resources:
            {{- toYaml .Values.resources | nindent 12 }}
      {{- with .Values.nodeSelector }}
      nodeSelector:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.affinity }}
      affinity:
        {{- toYaml . | nindent 8 }}
      {{- end }}
      {{- with .Values.tolerations }}
      tolerations:
        {{- toYaml . | nindent 8 }}
      {{- end }}
```

---

* `templates/service.yaml` 文件

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "app.fullname" . }}
  labels:
    {{- include "app.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "app.selectorLabels" . | nindent 4 }}
```

---

* `templates/serviceaccount.yaml` 文件


```yaml
{{- if .Values.serviceAccount.create -}}
apiVersion: v1
kind: ServiceAccount
metadata:
  name: {{ include "app.serviceAccountName" . }}
  labels:
    {{- include "app.labels" . | nindent 4 }}
  {{- with .Values.serviceAccount.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
{{- end }}

```

---

* `templates/ingress.yaml` 文件

```yaml

{{- if .Values.ingress.enabled -}}
{{- $fullName := include "app.fullname" . -}}
{{- $svcPort := .Values.service.port -}}
{{- if semverCompare ">=1.14-0" .Capabilities.KubeVersion.GitVersion -}}
apiVersion: networking.k8s.io/v1beta1
{{- else -}}
apiVersion: extensions/v1beta1
{{- end }}
kind: Ingress
metadata:
  name: {{ $fullName }}
  labels:
    {{- include "app.labels" . | nindent 4 }}
  {{- with .Values.ingress.annotations }}
  annotations:
    {{- toYaml . | nindent 4 }}
  {{- end }}
spec:
  {{- if .Values.ingress.tls }}
  tls:
    {{- range .Values.ingress.tls }}
    - hosts:
        {{- range .hosts }}
        - {{ . | quote }}
        {{- end }}
      secretName: {{ .secretName }}
    {{- end }}
  {{- end }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host | quote }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ . }}
            backend:
              serviceName: {{ $fullName }}
              servicePort: {{ $svcPort }}
          {{- end }}
    {{- end }}
  {{- end }}
```


---

* `templates/hpa.yaml` 文件

```yaml

{{- if .Values.autoscaling.enabled }}
apiVersion: autoscaling/v2beta1
kind: HorizontalPodAutoscaler
metadata:
  name: {{ include "app.fullname" . }}
  labels:
    {{- include "app.labels" . | nindent 4 }}
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: {{ include "app.fullname" . }}
  minReplicas: {{ .Values.autoscaling.minReplicas }}
  maxReplicas: {{ .Values.autoscaling.maxReplicas }}
  metrics:
  {{- if .Values.autoscaling.targetCPUUtilizationPercentage }}
    - type: Resource
      resource:
        name: cpu
        targetAverageUtilization: {{ .Values.autoscaling.targetCPUUtilizationPercentage }}
  {{- end }}
  {{- if .Values.autoscaling.targetMemoryUtilizationPercentage }}
    - type: Resource
      resource:
        name: memory
        targetAverageUtilization: {{ .Values.autoscaling.targetMemoryUtilizationPercentage }}
  {{- end }}
{{- end }}

```

---

* `templates/tests/test-connection.yaml` 文件

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: "{{ include "app.fullname" . }}-test-connection"
  labels:
    {{- include "app.labels" . | nindent 4 }}
  annotations:
    "helm.sh/hook": test-success
spec:
  containers:
    - name: wget
      image: busybox
      command: ['wget']
      args: ['{{ include "app.fullname" . }}:{{ .Values.service.port }}']
  restartPolicy: Never

```

---

---

#### template 命令

> helm template 命令

* `helm template chart_name` : 显示 chart 的 templates 模板内容. 既完整的 kubernetes  YAML 资源清单. 

  * `--show-only` : 只显示 templates 目录下的指定 模板资源内容. 例: `helm template myapp --show-only template/deployment.yaml`

  * `--output-dir` : 将模板资源内容输出到指定目录中. 例: `helm template myapp --output-dir /tmp/app/`

  * `--validate` :  模板的内容是否进行 kubernetes 资源校验.

  * `--no-hooks` : 不触发 hook 钩子操作.

  * `--dependency-update` : 是否更新依赖.

  * `--devel` : 显示开发版本的 `chart`. 


---

#### package 命令

> helm package 命令

* `helm package ` : 用于将 自定义的 `chart` 打包成 `chart` 压缩包, 以及生成 gpg 文件.

  * `--sign` : 此选项会生成 gpg 校验文件. 用于校验 `chart` 完整性以及来源. 如: `helm package --sign --key "markkey" --keyring path/key/keryring.gpg myapp` .

    * `--key` : 在 `--sign` 中需要指定 此 key 密码.

    * `--keyring` : 指定 keyring 文件.

      * `GNU Privacy Guard` :  简称 `GnuPG` 或 `GPG` 是一种加密软件, 依照由IETF订定的`OpenPGP`技术标准设计. `GnuPG` 用于加密、数位签章及产生非对称钥匙对的软件.
      * `gpg --gen-key` : 在Linux 下生成 gpg 密钥文件.

  * `--version` : 打包时指定 `chart` 版本号. `helm package myapp --version 2.0.1`

  * `--app-version` : 打包时指定 应用的 版本号. `helm package myapp --app-version v2`

  * `--destination` : 输出到指定目录. `--destination /tmp`. 默认会生成到当前目录下.

  * `--dependency-update` : 是否更新依赖.


---

#### lint 命令


> helm lint 命令

* `helm lint myapp` : 用于验证 `chart` 是否包含语法上的错误.

  * `--strict` : 严谨模式, 出现 warnings 警告类的错误, 也会验证失败.

  * `--with-subcharts` : 验证 `chart` 的同时会验证相关 子 `chart`. 



---

#### upgrade 命令

> helm upgrade 命令

* `helm upgrade myapp .` : 用于更新升级 `chart` .

  * `--install` :  如果 `release` 不存在, 会创建一个 `release`, 如果存在就更新.

  * `--devel` : 升级开发版本的 `release`.

  * `--dry-run` : 测试升级, 不会实际更新.

  * `--force` : 强制更新 `release` .

  * `--no-hooks` : 更新 `release` 不运行 hooks 钩子.

  * `--timeout` : 超时的等待时间. (default 5m0s)

  * `--reset-values` : 如果之前有使用 `--set` 设置 values , 可通过此命令, 不使用 `--set` 设置的 values.

  * `--reuse-values` : 使用原来 `--set` 设置的 values.

  * `--wait` : 更新后等待应用 Successful 后才返回, 既 kubernetes 资源都 Running. 如果出错会在 --timeout 时间后返回.  

  * `--history-max ` : 保留 `release` 版本记录. 默认是 10 条. 使用 `helm history` 的记录信息.

  * `--atomic` : 如果更新失败就回滚. 

  * `--cleanup-on-fail` : 如果更新失败会 清除此次更新. 

  * `--description` : 添加此次更新的说明/描述信息. 使用 `helm history myapp` 可查看到 DESCRIPTION .

  * `--version` : 更新到某个 version 中.

  * `--verify` : 验证更新的 `chart` 包的完整性.

    * `--keyring` : 指定验证 `chart` 包所需要的 gpg 文件.

  * `--repo` : 指定更新的 `repo` 地址.

    * `--username` : 指定 repo 仓库认证的 用户名.

    * `--password` : 指定 repo 仓库认证的 密码.

  * `--values -f` : 指定 values 变量文件.

  * `--set` : 设置 values 变量.

  * `--set-string` : 设置 string 类型 values 变量.

  * `--set-file` : 设置 values 为 脚本的 变量.

  * `-o、--output` : 格式化输出格式 支持 `table|json|yaml`.



---

#### rollback 命令

> helm rollback 命令


* `helm rollback myapp 2` : 回滚 `relesae` 到 REVISION 2 的版本. 如果不指定默认回滚到上一个版本 ( 上一个版本如果是 rollback 操作会回滚到之上的版本,如此类推 ).

  * `--dry-run` : 测试回滚, 不会实际操作回滚.

  * `--recreate-pods` :  强制重新创建 kubernetes 资源 pods .

  * `--no-hooks` : 不执行 hooks 钩子.

  * `--wait` : 回滚后等待应用 Successful 后才返回, 既 kubernetes 资源都 Running. 如果出错会在 --timeout 时间后返回.

  * `--timeout` : 超时的等待时间. (default 5m0s)

  * `--cleanup-on-fail` : 如果回滚 失败会清除此次回滚.
  


---

#### history 命令

> helm history 命令


* `helm history myapp` : 查看 `release` 历史版本的记录信息.

  * `--max` : 显示指定历史记录信息的条数.

  * `-o、--output` : 格式化输出 支持 `table|json|yaml` .



---

#### show 命令

> helm show 命令

* `helm show all myapp` : 显示 `chart` 所有的信息. 

* `helm show chart myapp` : 显示 `chart` 的 `Chart.yaml` 中的信息.

* `helm show readme myapp` : 显示 `chart` 的 README 中信息.

* `helm show values myapp` : 显示 `chart` 的 values 变量.

  * `--version` : 指定 version 版本号查看当前 `chart` 信息.

  * `--verify` : 显示 `chart` 信息并进行 校验.

  * `--repo` : 指定 repo 地址.

    * `--username` : 指定 repo 的验证 用户名.

    * `--password` : 指定 repo 的验证 密码.


---

#### get 命令

> helm get 命令

* `helm get all release_name` : 显示 `release` 所有的信息.

* `helm get hooks release_name` : 显示 `release` 钩子的信息. 

* `helm get manifest release_name` : 显示 `release`  kubernetes 资源清单YAML文件.

* `helm get notes release_name` : 显示 `release` 的 NOTES 提示信息. 

* `helm get values release_name` : 显示 `release` 部署时通过 `--set` 指定的 values 变量. `--all` 可以显示所有的 values 变量信息.

  * `--revision` : 指定 `release` 历史版本, 显示.

  * `-o、--output` : 格式化输出信息 支持 `table|json|yaml` .



---

#### dependency 命令


> helm dependency 命令 

* `dependency|dep|dependencies` 支持三个别名.


* `helm dependency update chart_name` : 根据 `Chart.yaml` 文件信息更新 `chart` 依赖并下载到 `chart` 的 `charts` 目录下.  

* `helm dependency list chart_name` :  显示 `chart` 的依赖信息. 如: app 依赖 mysql , redis 才能运行.

* `helm dependency build chart_path` : 根据 `Chart.lock` 文件信息构造 `chart` 的依赖包, 并生成到 `charts` 目录. 如果 `Chart.lock` 文件不存在, 会根据 `Chart.yaml` 文件构建.

---


#### plugin 命令

> helm plugin 命令


* `helm plugin list | ls` : 查看本地已安装的 插件列表.

* `helm plugin install | add` : 安装指定插件到本地.

  * `helm plugin install https://github.com/chartmuseum/helm-push` : URL 安装方式.

  * `helm plugin install /root/helm-push` : 插件目录方式安装.

* `helm plugin unstall | remove | rm ` : 删除本地已安装的插件.

* `helm plugin update | up` : 更新本地已安装的插件. 通过仓库以及URL安装方式才可以更新.

---

#### test 命令

> helm test 命令


* `helm test release_name` : 测试  `release` 是否正常.



---

#### pull 命令

> helm pull 命令


* `helm pull repo/chart_name` : 将 repo 仓库中指定的 `chart` 压缩包下载到本地.

  * `--devel` : 开发版本的 `chart` .

  * `--prov` : 下载 prov 文件.

  * `--untar` : 将下载的 `chart` 压缩包进行解压.

  * `--untardir` : 将下载的 `chart` 压缩包解压缩到指定目录. `helm pull stable/redis --untardir /root/redis`

  * `--destination` : 将下载的 `chart` 压缩包存到指定目录. `helm pull stable/redis --destination /root ` 

  * `--version` : 指定下载 `chart` 的版本.

  * `--verify` : 验证下载的 `chart` 压缩包完整性.

  * `--repo` : 从指定的 repo 中下载 `chart` .

    * `--username` :  设置 repo 所需的验证 用户名.
    * `--password` :  设置 repo 所需的验证 密码.

---

#### verify 命令

> helm verify 命令

* `helm verify myapp-1.0.0.tgz --keyring secring.gpg` : 检验 `chart` 包的完整性. 使用 gpg 进行校验.




---

#### registry 命令

> helm registry 命令

* 注: 使用 registry 命令, 需要将环境变量 `export HELM_EXPERIMENTAL_OCI=1` 才可以开启.


* `helm registry login docker_registry_url --username admin  --password admin` : 登录 docker registry 私有仓库.

* `helm registry logout docker_registry_url` : 登出 docker registry 私有仓库.


 

---


#### chart 命令

> helm chart 命令

* 注: 使用 chart 命令, 需要将环境变量 `export HELM_EXPERIMENTAL_OCI=1` 才可以开启.


* `helm chart list` : 显示本地已下载的 `chart` 包信息. 

* `helm chart pull` : 从指定 registry 下载 `chart` 包到本地. 例: `helm chart pull jicki.cn/myapp:1.0`

* `helm chart remove` : 删除已下载的本地 `chart` 包.

* `helm chart save` :  将本地的 `chart` 包, 保存到 `helm chart list` 列表中. 

* `helm chart push` : 将 `helm chart list` 列表中的 chart 提交到远程 registry 仓库中.



---



### Helm Chart


**Chart 是 helm 管理应用的包, 一个Chart对应一个或一套应用. Chart 内部由YAML描述文件组成.**

---



#### Chart 目录结构


* `Chart.yaml` :  chart 的描述文件, 包含版本信息, 名称 等.

* `Chart.lock` : chart 依赖的版本信息. ( apiVersion: v2 )

* `values.yaml` :  用于配置 `templates/` 目录下的模板文件使用的变量.

* `values.schema.json` : 用于校检 `values.yaml` 的完整性.

* `charts` : 依赖包的存储目录.

* `README` : 说明文件.

* `LICENSE` : 版权信息文件.

* `crd` : 存放 CRD 资源的文件的目录.

* `templates` : 模板文件存放目录.

  * `NOTES.txt` : 模板须知/说明文件. `helm install ` 成功后会显示此文件内容到屏幕.

  * `deployment.yaml` : kubernetes 资源文件. ( 所有类型的 kubernetes 资源文件都存放于 templates 目录下 )

  * `_helpers.tpl` :  以 `_` 开头的文件, 可以被其他模板引用. 



---


#### Chart.yaml 文件


> chart 的描述文件, 包含版本信息, Chart 名称,说明等.

* 这里仅说明 `apiVersion v2 版本`

```yaml
# Helm api 版本 (必填)
apiVersion: v2

# Chart 的名称 (必填)
name: myapp

# 此 Chart 的版本 (必填)
version: v1.0

# 约束此 Chart 支持的 kubernetes 版本, 如果不支持会失败 (可选)
kubeVersion: >= 1.18.0

# 此 Chart 说明信息 (可选)
description: "My App"

# Chart 的应用类型, 分别为 application (默认)和 library. (可选)
type: application

# Chart 关键词, 用于搜索时使用 (可选)
keywords
  - app
  - myapp

# Chart 的 home 的地址 (可选)
home: https://jicki.cn

# Chart 的源码地址 (可选)
sources:
  - https://github.com/jicki
  - https://jicki.cn


# Chart 的依赖信息, helm v2 是在 requirements.yaml 中. (可选)
dependencies:
  # 依赖的 chart 名称
  - name: nginx
  # 依赖的版本
    version: 1.2.3
  # 依赖的 repo 地址
    repository: https://kubernetes-charts.storage.googleapis.com
  # 依赖的 条件 如 nginx 启动
    condition: nginx.enabled
  # 依赖的标签 tags
    tags:
      - myapp-web
      - nginx-slb
    enabled: true
  # 传递值到 Chart 中. 
    import-values:
      - child:
      - parent:
  # 依赖的 别名
    alias: nginx-slb 


# Chart 维护人员信息 (可选) 
maintainers:
  - name: 小炒肉
    email: jicki@qq.com
    url: https://jicki.cn

# icon 地址 (可选)
icon: "https://jicki.cn/images/favicon.ico"

# App 的版本 (可选)
appVersion: 1.19.2

# 标注是否为过期 (可选)
deprecated: false

# 注释 (可选)
annotations:
  # 注释的例子
  example: 
```



---

#### Helm 内置变量

* Helm 内置对象包含

  * `Release` 相关的内置属性.

  * `Chart` 属性 - `Chart.yaml` 文件中定义的内容.

  * `Values` 属性 - `Values.yaml` 文件中定义的内容.


---

* `Release` 内置变量

| 变量名称           | 变量说明                                              |
|--------------------|-------------------------------------------------------|
| Release.Name       | release 名称                                          |
| Release.Namespace  | release 所在命名空间                                  |
| Release.Service    | release 发布的服务                                    |
| Release.IsUpgrade  | 如果 release 操作状态为 更新 / 回滚 , 此变量值为 true | 
| Release.IsInstall  | 如果 release 操作状态为 安装 , 此变量值为 true        | 
| Release.Revision   | release 版本序号, 初始为 1 , 执行 更新/ 回滚 版本 +1  |



---




