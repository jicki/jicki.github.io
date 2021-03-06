# Helm - v2



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


