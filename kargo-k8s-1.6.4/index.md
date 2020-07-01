# kargo kubernetes 1.6.4



# 1、初始化环境

> kargo update k8s 1.6.4


## 1.1、环境：

| 节点 |    IP   |  角色  |
|------|---------|--------|
|node-1|10.6.0.52| Master |
|node-2|10.6.0.53| Master |
|node-3|10.6.0.55| Node   |
|node-4|10.6.0.56| Node   |

## 1.2、配置SSH Key 登陆

```
# 确保本机也可以 ssh 连接，否则下面部署失败

ssh-keygen -t rsa -N ""

ssh-copy-id -i /root/.ssh/id_rsa.pub 10.6.0.52

ssh-copy-id -i /root/.ssh/id_rsa.pub 10.6.0.53

ssh-copy-id -i /root/.ssh/id_rsa.pub 10.6.0.55

ssh-copy-id -i /root/.ssh/id_rsa.pub 10.6.0.56

```



# 2、获取 Kargo

> Kargo 官方github
> https://github.com/kubernetes-incubator/kargo


## 2.1、安装基础软件

> Kargo 是基于 ansible 统一部署，所以必须安装 ansible

```
# 安装 centos 额外的yum源
yum install -y epel-release

# 安装 软件
yum install -y python-pip python34 python-netaddr python34-pip ansible

# 如果 报 no test named 'equalto' ，需要升级 Jinja2
pip install --upgrade Jinja2
```

## 2.2、获取源码

```
git clone https://github.com/kubernetes-incubator/kargo
```

## 2.3、编辑配置文件

```
cd kargo

vim inventory/group_vars/k8s-cluster.yml


这里主要修改一些 网段，密码 等信息


```

## 2.4、生成集群配置文件

```
cd kargo

CONFIG_FILE=inventory/inventory.cfg python3 contrib/inventory_builder/inventory.py 10.6.0.52 10.6.0.53 10.6.0.55 10.6.0.56

# 输入如下：

DEBUG: Adding group all
DEBUG: Adding group kube-master
DEBUG: Adding group kube-node
DEBUG: Adding group etcd
DEBUG: Adding group k8s-cluster:children
DEBUG: Adding group calico-rr
DEBUG: adding host node1 to group all
DEBUG: adding host node2 to group all
DEBUG: adding host node3 to group all
DEBUG: adding host node4 to group all
DEBUG: adding host kube-node to group k8s-cluster:children
DEBUG: adding host kube-master to group k8s-cluster:children
DEBUG: adding host node1 to group etcd
DEBUG: adding host node2 to group etcd
DEBUG: adding host node3 to group etcd
DEBUG: adding host node1 to group kube-master
DEBUG: adding host node2 to group kube-master
DEBUG: adding host node1 to group kube-node
DEBUG: adding host node2 to group kube-node
DEBUG: adding host node3 to group kube-node
DEBUG: adding host node4 to group kube-node


# 生成的配置文件在当前目录，既 kargo/inventory 目录下 inventory.cfg

# 配置文件如下(默认配置双master，可自行修改)：
# SSH 非 22 端口 添加 ansible_port=xxx

[all]
node1    ansible_host=10.6.0.52 ansible_port=33 ip=10.6.0.52
node2    ansible_host=10.6.0.53 ansible_port=33 ip=10.6.0.53
node3    ansible_host=10.6.0.55 ansible_port=33 ip=10.6.0.55
node4    ansible_host=10.6.0.56 ansible_port=33 ip=10.6.0.56

[kube-master]
node1    
node2    

[kube-node]
node1    
node2    
node3    
node4    

[etcd]
node1    
node2    
node3    

[k8s-cluster:children]
kube-node        
kube-master      

[calico-rr]

```

```
# 1.6.4 镜像下载

http://pan.baidu.com/s/1nvUc5mx


```

## 2.5、部署集群

```
# 执行如下命令，请确保SSH KEY 登陆, 端口一致

ansible-playbook -i inventory/inventory.cfg cluster.yml -b -v --private-key=~/.ssh/id_rsa



# 升级的命令，没有确认

ansible-playbook upgrade-cluster.yml -b -i inventory/inventory.cfg -e kube_version=v1.6.4 -v --private-key=~/.ssh/id_rsa 


```

## 2.6、测试

```
# 两个 master 中使用 kubectl get nodes

[root@k8s-node-1 ~]# kubectl get nodes
NAME         STATUS    AGE       VERSION
k8s-node-1   Ready     16m       v1.6.4+coreos.0
k8s-node-2   Ready     20m       v1.6.4+coreos.0
k8s-node-3   Ready     16m       v1.6.4+coreos.0
k8s-node-4   Ready     16m       v1.6.4+coreos.0



[root@k8s-node-2 ~]# kubectl get nodes
NAME         STATUS    AGE       VERSION
k8s-node-1   Ready     11m       v1.6.4+coreos.0
k8s-node-2   Ready     16m       v1.6.4+coreos.0
k8s-node-3   Ready     11m       v1.6.4+coreos.0
k8s-node-4   Ready     11m       v1.6.4+coreos.0




[root@k8s-node-1 ~]# kubectl get pods --namespace=kube-system
NAME                                  READY     STATUS    RESTARTS   AGE
dnsmasq-411420702-z0gkx               1/1       Running   0          16m
dnsmasq-autoscaler-1155841093-1hxdl   1/1       Running   0          16m
elasticsearch-logging-v1-kgt1t        1/1       Running   0          15m
elasticsearch-logging-v1-vm4bd        1/1       Running   0          15m
fluentd-es-v1.22-6gql6                1/1       Running   0          15m
fluentd-es-v1.22-8zkjh                1/1       Running   0          15m
fluentd-es-v1.22-cjskv                1/1       Running   0          15m
fluentd-es-v1.22-j4857                1/1       Running   0          15m
kibana-logging-2924323056-x3vjk       1/1       Running   0          15m
kube-apiserver-k8s-node-1             1/1       Running   0          15m
kube-apiserver-k8s-node-2             1/1       Running   0          20m
kube-controller-manager-k8s-node-1    1/1       Running   0          16m
kube-controller-manager-k8s-node-2    1/1       Running   0          21m
kube-proxy-k8s-node-1                 1/1       Running   0          16m
kube-proxy-k8s-node-2                 1/1       Running   0          21m
kube-proxy-k8s-node-3                 1/1       Running   0          16m
kube-proxy-k8s-node-4                 1/1       Running   0          16m
kube-scheduler-k8s-node-1             1/1       Running   0          16m
kube-scheduler-k8s-node-2             1/1       Running   0          21m
kubedns-3830354952-pfl7n              3/3       Running   4          16m
kubedns-autoscaler-54374881-64x6d     1/1       Running   0          16m
nginx-proxy-k8s-node-3                1/1       Running   0          16m
nginx-proxy-k8s-node-4                1/1       Running   0          16m



[root@k8s-node-1 ~]# kubectl get pods
NAME                             READY     STATUS    RESTARTS   AGE
netchecker-agent-3x3sj           1/1       Running   0          16m
netchecker-agent-ggxs2           1/1       Running   0          16m
netchecker-agent-hostnet-45k84   1/1       Running   0          16m
netchecker-agent-hostnet-kwvc8   1/1       Running   0          16m
netchecker-agent-hostnet-pwm77   1/1       Running   0          16m
netchecker-agent-hostnet-z4gmq   1/1       Running   0          16m
netchecker-agent-q3291           1/1       Running   0          16m
netchecker-agent-qtml6           1/1       Running   0          16m
netchecker-server                1/1       Running   0          16m


```


```
# 配置一个 nginx deplyment 与 nginx  service

apiVersion: extensions/v1beta1 
kind: Deployment 
metadata: 
  name: nginx-dm
spec: 
  replicas: 2
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

```

```
apiVersion: v1 
kind: Service
metadata: 
  name: nginx-dm 
spec: 
  ports: 
    - port: 80
      targetPort: 80
      protocol: TCP 
  selector: 
    name: nginx
```


```
# 导入 yaml 文件


[root@k8s-node-1 ~]# kubectl apply -f nginx.yaml 
deployment "nginx-dm" created
service "nginx-dm" created



[root@k8s-node-1 ~]# kubectl get pods -o wide
NAME                             READY     STATUS    RESTARTS   AGE       IP              NODE
nginx-dm-4194680597-0h071        1/1       Running   0          9m        10.233.75.8     k8s-node-4
nginx-dm-4194680597-dzcf3        1/1       Running   0          9m        10.233.76.124   k8s-node-3


[root@k8s-node-1 ~]# kubectl get svc -o wide    
NAME                 CLUSTER-IP      EXTERNAL-IP   PORT(S)          AGE       SELECTOR
kubernetes           10.233.0.1      <none>        443/TCP          39m       <none>
netchecker-service   10.233.0.126    <nodes>       8081:31081/TCP   33m       app=netchecker-server
nginx-dm             10.233.56.138   <none>        80/TCP           10m       name=nginx

```


```
# 部署一个  curl 的 pods 用来测试 内部通信


apiVersion: v1
kind: Pod
metadata:
  name: curl
spec:
  containers:
  - name: curl
    image: radial/busyboxplus:curl
    command:
    - sh
    - -c
    - while true; do sleep 1; done

```    

```    
# 导入 yaml 文件

[root@k8s-node-1 ~]# kubectl apply -f curl.yaml 
pod "curl" created

    
[root@k8s-node-1 ~]# kubectl get pods -o wide
NAME                             READY     STATUS    RESTARTS   AGE       IP              NODE
curl                             1/1       Running   0          2m        10.233.75.22    k8s-node-4


```

```
# 测试 curl --> nginx-svc


[root@k8s-node-1 ~]# kubectl exec -it curl curl nginx-dm


<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
<style>
    body {
        width: 35em;
        margin: 0 auto;
        font-family: Tahoma, Verdana, Arial, sans-serif;
    }
</style>
</head>
<body>
<h1>Welcome to nginx!</h1>
<p>If you see this page, the nginx web server is successfully installed and
working. Further configuration is required.</p>

<p>For online documentation and support please refer to
<a href="http://nginx.org/">nginx.org</a>.<br/>
Commercial support is available at
<a href="http://nginx.com/">nginx.com</a>.</p>

<p><em>Thank you for using nginx.</em></p>
</body>
</html>

```







```
# 创建一个 zk  集群 zk-deplyment  和 service


apiVersion: extensions/v1beta1
kind: Deployment 
metadata: 
  name: zookeeper-1
spec: 
  replicas: 1
  template: 
    metadata: 
      labels: 
        name: zookeeper-1 
    spec: 
      containers: 
        - name: zookeeper-1
          image: jicki/zk:alpine 
          imagePullPolicy: IfNotPresent
          env:
          - name: NODE_ID
            value: "1"
          - name: NODES
            value: "0.0.0.0,zookeeper-2,zookeeper-3"
          ports:
          - containerPort: 2181

```

```
apiVersion: extensions/v1beta1 
kind: Deployment
metadata:
  name: zookeeper-2
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: zookeeper-2
    spec:
      containers:
        - name: zookeeper-2
          image: jicki/zk:alpine
          imagePullPolicy: IfNotPresent
          env:
          - name: NODE_ID
            value: "2"
          - name: NODES
            value: "zookeeper-1,0.0.0.0,zookeeper-3"
          ports:
          - containerPort: 2181

```


```
apiVersion: extensions/v1beta1
kind: Deployment
metadata:
  name: zookeeper-3
spec:
  replicas: 1
  template:
    metadata:
      labels:
        name: zookeeper-3
    spec:
      containers:
        - name: zookeeper-3
          image: jicki/zk:alpine
          imagePullPolicy: IfNotPresent
          env:
          - name: NODE_ID
            value: "3"
          - name: NODES
            value: "zookeeper-1,zookeeper-2,0.0.0.0"
          ports:
          - containerPort: 2181
```

```
apiVersion: v1 
kind: Service 
metadata: 
  name: zookeeper-1 
  labels:
    name: zookeeper-1
spec: 
  ports: 
    - name: client
      port: 2181
      protocol: TCP
    - name: followers
      port: 2888
      protocol: TCP
    - name: election
      port: 3888
      protocol: TCP
  selector: 
    name: zookeeper-1

```

```

apiVersion: v1 
kind: Service 
metadata: 
  name: zookeeper-2
  labels:
    name: zookeeper-2
spec: 
  ports: 
    - name: client
      port: 2181
      protocol: TCP
    - name: followers
      port: 2888
      protocol: TCP
    - name: election
      port: 3888
      protocol: TCP
  selector: 
    name: zookeeper-2

```

```

apiVersion: v1 
kind: Service 
metadata: 
  name: zookeeper-3
  labels:
    name: zookeeper-3
spec: 
  ports: 
    - name: client
      port: 2181
      protocol: TCP
    - name: followers
      port: 2888
      protocol: TCP
    - name: election
      port: 3888
      protocol: TCP
  selector: 
    name: zookeeper-3

```

```
# 查看状态  pods 与 service

[root@k8s-node-1 ~]# kubectl get pods -o wide
NAME                             READY     STATUS    RESTARTS   AGE       IP              NODE
zookeeper-1-3762028479-gd5rm     1/1       Running   0          1m        10.233.76.125   k8s-node-3
zookeeper-2-4266983361-cz80w     1/1       Running   0          1m        10.233.75.23    k8s-node-4
zookeeper-3-479264707-hlv3x      1/1       Running   0          1m        10.233.75.24    k8s-node-4



[root@k8s-node-1 ~]# kubectl get svc -o wide    
NAME                 CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE       SELECTOR
zookeeper-1          10.233.25.46    <none>        2181/TCP,2888/TCP,3888/TCP   1m        name=zookeeper-1
zookeeper-2          10.233.49.4     <none>        2181/TCP,2888/TCP,3888/TCP   1m        name=zookeeper-2
zookeeper-3          10.233.50.206   <none>        2181/TCP,2888/TCP,3888/TCP   1m        name=zookeeper-3

```


## 2.7、部署一个 Nginx Ingress

> kubernetes 暴露服务的方式目前只有三种：LoadBlancer Service、NodePort Service、Ingress；
> 什么是 Ingress ?  Ingress 就是利用 nginx haproxy 等负载均衡工具来暴露 kubernetes 服务。


```

# 首先 部署一个 http-backend, 用于统一转发 没有的域名 到指定页面。
# 官方 nginx  ingress 库 https://github.com/kubernetes/ingress/tree/master/examples/deployment/nginx

```


```
# 下载官方的 nginx  backend 文件

curl -O https://raw.githubusercontent.com/kubernetes/ingress/master/examples/deployment/nginx/default-backend.yaml


# 直接导入既可
[root@k8s-node-1 ~]# kubectl apply -f default-backend.yaml 
deployment "default-http-backend" created
service "default-http-backend" created

# 查看 deployment 与 service

[root@k8s-node-1 ~]# kubectl get deployment --namespace=kube-system
NAME                   DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
default-http-backend   1         1         1            1           33m


[root@k8s-node-1 ~]# kubectl get svc --namespace=kube-system       
NAME                    CLUSTER-IP      EXTERNAL-IP   PORT(S)         AGE
default-http-backend    10.233.20.232   <none>        80/TCP          33m

```


```
# 部署 Ingress Controller 组件

```

```
# 下载 官方 nginx-ingress-controller 的yaml文件

curl -O https://raw.githubusercontent.com/kubernetes/ingress/master/examples/deployment/nginx/nginx-ingress-controller.yaml

# 编辑 yaml 文件，打开 hostNetwork: true , 将端口绑定到宿主机中
# 这里面deployment 默认只启动了一个pods, 这里可以修改 kind: Deployment 为 kind: DaemonSet  并注释掉 replicas
# 或者 修改 replicas: 1  为 N 


vi nginx-ingress-controller.yaml

将 hostNetwork: true  前面的注释去掉



# 导入 yaml 文件

[root@k8s-node-1 ~]# kubectl apply -f nginx-ingress-controller.yaml 
deployment "nginx-ingress-controller" created


# 查看 deployment  或者  daemonsets
[root@k8s-node-1 yaml]# kubectl get deployment --namespace=kube-system
NAME                       DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-ingress-controller   1         1         1            1           31s


[root@k8s-node-1 yaml]# kubectl get daemonsets --namespace=kube-system
NAME                       DESIRED   CURRENT   READY     NODE-SELECTOR   AGE
nginx-ingress-controller   4         4         4         <none>          1m


```


```
# 最后开始 部署 Ingress
# 这里请先看看官方 ingress 的 yaml 写法
# https://kubernetes.io/docs/user-guide/ingress/


# 我们使用 之前创建的 nginx-dm  service，我们来写一个 ingress
# 首先查看一下 svc 

[root@k8s-node-1 yaml]# kubectl get svc
NAME                 CLUSTER-IP      EXTERNAL-IP   PORT(S)                      AGE
kubernetes           10.233.0.1      <none>        443/TCP                      1d
netchecker-service   10.233.0.126    <nodes>       8081:31081/TCP               1d
nginx-dm             10.233.56.138   <none>        80/TCP                       1d
zookeeper-1          10.233.25.46    <none>        2181/TCP,2888/TCP,3888/TCP   1d
zookeeper-2          10.233.49.4     <none>        2181/TCP,2888/TCP,3888/TCP   1d
zookeeper-3          10.233.50.206   <none>        2181/TCP,2888/TCP,3888/TCP   1d



# 创建 yaml 文件， 这里特别注意，如果 svc 在 kube-system 下
# 必须在 metadata: 下面添加 namespace: kube-system 指定命名空间

vim nginx-ingress.yaml

apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  rules:
  - host: nginx.jicki.cn
    http:
      paths:
      - backend:
          serviceName: nginx-dm
          servicePort: 80


# 导入 yaml 文件

[root@k8s-node-1 ~]# kubectl apply -f nginx-ingress.yaml 
ingress "nginx-ingress" created


# 查看一下 创建的 ingress

[root@k8s-node-1 ~]# kubectl get ingresses
NAME            HOSTS            ADDRESS   PORTS     AGE
nginx-ingress   nginx.jicki.cn             80        17s

# 这里显示 ADDRESS 为 空 实际上 所有 master 与 nodes 都绑定了
# 将域名解析到 任何一个 IP 上都可以。






# 下面访问 http://nginx.jicki.cn/

# 这里注意，Ingresses 只做简单的端口转发。


```

![nginx][1]



# 维护 FAQ

```
# 卸载

cd kargo

ansible-playbook -i inventory/inventory.cfg reset.yml -b -v --private-key=~/.ssh/id_rsa

```


```
# 增加节点

# 首先编辑 inventory/inventory.cfg  增加一个节点 例：node5

[kube-node]
node1
node2
node3
node4
node5


# 执行命令 使用 --limit 参数

ansible-playbook -i inventory/inventory.cfg cluster.yml -b -v --private-key=~/.ssh/id_rsa --limit node5


```

```
# 报错1
# hostname 的问题
# 部署 kargo 必须配置 hostname 否则 多 master 会出现 无法创建 api 等 pods
# 如果 执行了 ansible-playbook 之前没改 hostname 必须删除 /tmp 下的 node[N]
# 否则更改 /etc/hosts 失败
```

```
# 报错 2

TASK [vault : check_vault | Set fact about the Vault cluster's initialization state] ***
Monday 10 April 2017  17:47:42 +0800 (0:00:00.088)       0:01:22.030 ********** 
fatal: [node1]: FAILED! => {"failed": true, "msg": "'dict object' has no attribute 'vault'"}
fatal: [node3]: FAILED! => {"failed": true, "msg": "'dict object' has no attribute 'vault'"}

# 解决方案 升级 ansible => 2.2.1.0

```



  [1]: https://jicki.cn/img/posts/kagro/1.png




