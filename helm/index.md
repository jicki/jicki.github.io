# Kubernetes 包管理 Helm



# Helm

*  Helm 是 Kubernetes 的包管理器, 可以帮我们简化 kubernetes 应用部署操作, 在多项目的环境下, 应用的部署与管理就会变的频繁与复杂. Helm 通过包的方式, 可简化应用的版本发布、管理、控制.  


* Helm 本质是在 Kubernetes 的应用管理上模板化, 能动态的生成如`Deployment`、`Service` 等的 yaml 资源文件.


* Helm 有两个重要概念

  * `chart` :是创建一个应用的信息集合, 既一个应用所包含的所有需要部署的服务如`deployment`、`service`、`pv`、`pvc` 等的各种 kubernetes 对象的配置模板、参数定义、依赖关系、文档、帮助信息等. `chart` 是应用部署的自包含逻辑单元, 类似于 `Yum` 中的软件rpm安装包. 

  * `release` :是`chart` 的运行实例, 代表了一个正在运行的应用. 当 `chart` 被安装到 kubernetes 集群中时就生成一个 `release`. 每执行一次 `chart` 安装就会生成一个 `release`.  


---

## Helm v2


* 注: 当前 2020年8月 Helm v2 系列发布了 v2.16.10 版本， 这是 Helm v2 的最后一个 bugfix 版本，此后不会再为 Helm v2 提供错误修复。并且在三个月后，将停止为 Helm v2 提供安全补丁。届时， Helm v2 也就完全废弃，不会再去维护了。





### Helm 流程图


{{< figure src="/img/posts/helm/helm-flow.png" >}}



* Helm - 客户端 Helm Client 负责 `chart` 和 `release` 的创建、管理, 通过 gRPC 与 Tiller 服务端进行交互.

* Tiller - 服务端 运行在 Kubernetes 中, 处理 Helm Client 的请求, 通过 REST、JSON 与 kubernetes Api Server 进行交互.






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

### Helm 自定义模板


* 自定义 Helm 模板 需要包含 `Chart.yaml` 文件、`templates` 模板目录.

  * `Chart.yaml` 文件是描述文件, 用于描述该应用的信息. 必须包含 `name` 与 `version` .

  * `templates` 模板目录, 用于存放 kubernetes 资源清单. 目录必须为`templates`此名称.



---


> 基础的例子


* `Chart.yaml`


```yaml

name: my-nginx
version: 1.0.0

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











### helm 命令

> Helm 一些常用命令


```
# 查询 charts 包

[root@kubernetes-1 ~]# helm search mysql
NAME                            CHART VERSION   APP VERSION     DESCRIPTION                                                 
stable/mysql                    0.3.5                           Fast, reliable, scalable, and easy to use open-source rel...
stable/percona                  0.3.0                           free, fully compatible, enhanced, open source drop-in rep...
stable/percona-xtradb-cluster   0.0.2           5.7.19          free, fully compatible, enhanced, open source drop-in rep...
stable/gcloud-sqlproxy          0.2.3                           Google Cloud SQL Proxy                                      
stable/mariadb                  2.1.6           10.1.31         Fast, reliable, scalable, and easy to use open-source rel...
```


```
# 查询 包 的具体信息


[root@kubernetes-1 ~]# helm inspect stable/mysql
description: Fast, reliable, scalable, and easy to use open-source relational database
  system.
engine: gotpl
home: https://www.mysql.com/
icon: https://www.mysql.com/common/logos/logo-mysql-170x115.png
keywords:
......

```


```
# 部署 程序

helm install stable/mysql
```



```
# 查询支持的选项,用于set 参数


[root@kubernetes-1 ~]# helm inspect values stable/mysql
## mysql image version
## ref: https://hub.docker.com/r/library/mysql/tags/
##
image: "mysql"
imageTag: "5.7.14"

## Specify password for root user
##
## Default: random 10 character string
# mysqlRootPassword: testing
.......

```

```
# 根据上面 查询 values 的mysqlRootPassword ，修改为 jicki


helm install --name mysql --set mysqlRootPassword=jicki  stable/mysql
````


```
# 升级 设置，或者版本


helm upgrade db-mysql --set mysqlRootPassword=passwd --set image="jicki/mysql" --set imageTag="5.7.20" stable/mysql

```

```
# 回滚 到upgrade 之前的版本


helm rollback db-mysql 1

```


```
# 添加 额外 repo


helm repo add my-repo https://kubernetes-charts.jicki.cn/

```


```
# 查询 repo 列表


helm repo list
```


---

### Helm FAQ


* `Error: no available release name found`

  * 出现这个错误是因为 rbac 的问题。

