# k8s 1.23.5 to kubespray



# Kubernetes 1.23.5

> kubespray Deploy a Production Ready Kubernetes Cluster
>
> kubespray 利用 Ansible 自动部署 更新 kubernetes 集群


## 环境说明


> Ubuntu 18.x

|节点|IP|
|----|----|
|kubernetes-1|10.9.9.91|
|kubernetes-2|10.9.9.92|
|kubernetes-3|10.9.9.93|
|kubernetes-4|10.9.9.96|

  
## 配置ssh key 认证

```
# 确保本机也可以 ssh 连接，否则下面部署失败

ssh-keygen -t rsa -N ""

ssh-copy-id -i /root/.ssh/id_rsa.pub 10.9.9.91

ssh-copy-id -i /root/.ssh/id_rsa.pub 10.9.9.92

ssh-copy-id -i /root/.ssh/id_rsa.pub 10.9.9.93

ssh-copy-id -i /root/.ssh/id_rsa.pub 10.9.9.96
```



## 增加 Kernel Modules 与 Ipvs Modules 配置

> 由于 Kubernetes 新版本 Service 实现切换到 IPVS，所以需要确保内核加载了 IPVS modules；以下命令将设置系统启动自动加载 IPVS 相关模块， 执行完成后需要重启。



```
# sysctl

cat > /etc/sysctl.d/50-kubernetes.conf <<EOF
net.ipv4.ip_forward=1
net.bridge.bridge-nf-call-iptables=1
net.bridge.bridge-nf-call-ip6tables=1
fs.inotify.max_user_watches=525000
EOF

```

```
# Kernel modules

cat > /etc/modules-load.d/50-kubernetes.conf <<EOF
# Load some kernel modules needed by kubernetes at boot
nf_conntrack
br_netfilter
ip_vs
ip_vs_lc
ip_vs_wlc
ip_vs_rr
ip_vs_wrr
ip_vs_lblc
ip_vs_lblcr
ip_vs_dh
ip_vs_sh
ip_vs_fo
ip_vs_nq
ip_vs_sed
EOF
```

```
# 重启系统后 验证 ipvs 模块

lsmod | grep ip_vs


----
ip_vs_sed              16384  0
ip_vs_nq               16384  0
ip_vs_fo               16384  0
ip_vs_sh               16384  0
ip_vs_dh               16384  0
ip_vs_lblcr            16384  0
ip_vs_lblc             16384  0
ip_vs_wrr              16384  0
ip_vs_rr               16384  0
ip_vs_wlc              16384  0
ip_vs_lc               16384  0
ip_vs                 151552  22 ip_vs_wlc,ip_vs_rr,ip_vs_dh,ip_vs_lblcr,ip_vs_sh,ip_vs_fo,ip_vs_nq,ip_vs_lblc,ip_vs_wrr,ip_vs_lc,ip_vs_sed
nf_defrag_ipv6         20480  1 ip_vs
nf_conntrack          135168  1 ip_vs
libcrc32c              16384  3 nf_conntrack,raid456,ip_vs
---

```



# kubespray


## 下载源码


```
# git clone 源码

cd /opt/

git clone https://github.com/kubernetes-sigs/kubespray

```



## 安装 kubespray 依赖

```
cd /opt/kubespray

# 安装依赖 ( ansible 4.8.0 )  requirements.txt 默认安装 ansible 5.5.0


python3 -m pip -r requirements-2.11.txt


Successfully installed MarkupSafe-1.1.1 ansible-4.8.0 ansible-core-2.11.6 cffi-1.15.0 cryptography-2.8 jinja2-2.11.3 jmespath-0.9.5 netaddr-0.7.19 packaging-21.3 pbr-5.4.4 pycparser-2.21 pyparsing-3.0.7 resolvelib-0.5.5 ruamel.yaml-0.16.10 ruamel.yaml.clib-0.2.6

```



## 配置 kubespray


### inventory 配置

```
# 复制一份 自己的配置

cd /opt/kubespray

cp -rfp inventory/sample inventory/jicki


# 修改配置 inventory.ini

cd /opt/kubespray/inventory/jicki

rm -rf inventory.ini


vi inventory.ini

[all]
kubernetes-1 ansible_host=10.9.9.91 ansible_port=22 # ip=10.9.9.91 etcd_member_name=etcd1
kubernetes-2 ansible_host=10.9.9.92 ansible_port=22 # ip=10.9.9.92 etcd_member_name=etcd2
kubernetes-3 ansible_host=10.9.9.93 ansible_port=22 # ip=10.9.9.93 etcd_member_name=etcd3
kubernetes-4 ansible_host=10.9.9.96 ansible_port=22 # ip=10.9.9.96
# node5 ansible_host=95.54.0.16  # ip=10.3.0.5 etcd_member_name=etcd5
# node6 ansible_host=95.54.0.17  # ip=10.3.0.6 etcd_member_name=etcd6

# ## configure a bastion host if your nodes are not directly reachable
# [bastion]
# bastion ansible_host=x.x.x.x ansible_user=some_user

[kube_control_plane]
kubernetes-1
kubernetes-2
kubernetes-3

[etcd]
kubernetes-1
kubernetes-2
kubernetes-3

[kube_node]
kubernetes-4

[calico_rr]

[k8s_cluster:children]
kube_control_plane
kube_node
calico_rr

```


### etcd 配置

```
cd /opt/kubespray/inventory/jicki/group_vars


cat etcd.yml


---
## Etcd auto compaction retention for mvcc key value store in hour
etcd_compaction_retention: 1

## Set level of detail for etcd exported metrics, specify 'extensive' to include histogram metrics.
# etcd_metrics: basic

## Etcd is restricted by default to 512M on systems under 4GB RAM, 512MB is not enough for much more than testing.
## Set this if your etcd nodes have less than 4GB but you want more RAM for etcd. Set to 0 for unrestricted RAM.
etcd_memory_limit: "2G"

## Etcd has a default of 2G for its space quota. If you put a value in etcd_memory_limit which is less than
## etcd_quota_backend_bytes, you may encounter out of memory terminations of the etcd cluster. Please check
## etcd documentation for more information.
## ===========8G==================== ##
etcd_quota_backend_bytes: "8589934592"

### ETCD: disable peer client cert authentication.
# This affects ETCD_PEER_CLIENT_CERT_AUTH variable
etcd_peer_client_auth: true

```


### 全局配置


> all.yml 配置

```
cd /opt/kubespray/inventory/jicki/group_vars/all

cat all.yml  # 注意的配置有如下:


## Internal loadbalancers for apiservers
loadbalancer_apiserver_localhost: true
# valid options are "nginx" or "haproxy" 
loadbalancer_apiserver_type: nginx 

```


> containerd.yml 配置

```
cat containerd.yml   # 注意的配置有如下:

containerd_storage_dir: "/opt/containerd"
containerd_state_dir: "/run/containerd"
# OOM 评分 
containerd_oom_score: -999

containerd_snapshotter: "overlayfs"

# 开启 metrics
containerd_metrics_address: "127.0.0.1:1234"

# logs 最大行数
containerd_max_container_log_line_size: 16384

```


### k8s 功能配置

```
cd /opt/kubespray/inventory/jicki/group_vars/k8s-cluster

vi k8s-cluster.yml


# 按照自己的需求修改需要注意的是如下部分


# Kubernetes internal network for services, unused block of space.
kube_service_addresses: 10.233.0.0/18

# internal network. When used, it will assign IP
# addresses from this range to individual pods.
# This network must be unused in your network infrastructure!
kube_pods_subnet: 10.233.64.0/18

# internal network node size allocation (optional). This is the size allocated
# to each node for pod IP address allocation. Note that the number of pods per node is
# also limited by the kubelet_max_pods variable which defaults to 110.
#
# Example:
# Up to 64 nodes and up to 254 or kubelet_max_pods (the lowest of the two) pods per node:
#  - kube_pods_subnet: 10.233.64.0/18
#  - kube_network_node_prefix: 24
#  - kubelet_max_pods: 110
#
# Example:
# Up to 128 nodes and up to 126 or kubelet_max_pods (the lowest of the two) pods per node:
#  - kube_pods_subnet: 10.233.64.0/18
#  - kube_network_node_prefix: 25
#  - kubelet_max_pods: 110
kube_network_node_prefix: 24


## Container runtime
## docker for docker, crio for cri-o and containerd for containerd.
## Default: containerd
container_manager: containerd

```



### Download 程序与镜像


>  国外镜像在国内无法下载的问题  修改 `roles/download/defaults/main.yml`


```
# gcr and kubernetes image repo define
#gcr_image_repo: "gcr.io"
gcr_image_repo: "registry.aliyuncs.com"
#kube_image_repo: "k8s.gcr.io"
kube_image_repo: "registry.aliyuncs.com/google_containers"

```

> coredns_image_repo 的问题, `registry.aliyuncs.com/google_containers` 的下载地址需要修改为下:

```
# coredns_image_repo: "{{ kube_image_repo }}{{'/coredns/coredns' if (coredns_image_is_namespaced | bool) else '/coredns' }}"

coredns_image_repo: "{{ kube_image_repo }}{{'/coredns' if (coredns_image_is_namespaced | bool) else '/coredns' }}"

```



> github.com 下载文件的问题  添加 `/etc/hosts` 解决可查看 `https://github.com/ineo6/hosts`
> 如果实在下载不到可以提前下载然后分发至每个节点 `/tmp/releases` 目录下

---

> 这里有一些镜像存在一些问题需要手动操作一下, 在 `registry.aliyuncs.com` 镜像并不全
> `kube_version: v1.23.5`

* 修改如下配置

```
nodelocaldns_version: "1.21.1"
#nodelocaldns_image_repo: "{{ kube_image_repo }}/dns/k8s-dns-node-cache"
nodelocaldns_image_repo: "jicki/k8s-dns-node-cache"

dnsautoscaler_version: 1.8.5
#dnsautoscaler_image_repo: "{{ kube_image_repo }}/cpa/cluster-proportional-autoscaler-{{ image_arch }}"
dnsautoscaler_image_repo: "jicki/cluster-proportional-autoscaler-amd64"
```




## 安装k8s集群

```
cd /opt/kubespray

ansible-playbook -i inventory/jicki/inventory.ini --become --become-user=root cluster.yml




PLAY RECAP ***************************************************************************************************************************************************************************************************
kubernetes-1               : ok=717  changed=80   unreachable=0    failed=0    skipped=1280 rescued=0    ignored=3   
kubernetes-2               : ok=646  changed=68   unreachable=0    failed=0    skipped=1119 rescued=0    ignored=1   
kubernetes-3               : ok=648  changed=69   unreachable=0    failed=0    skipped=1117 rescued=0    ignored=1   
kubernetes-4               : ok=504  changed=50   unreachable=0    failed=0    skipped=761  rescued=0    ignored=0   
localhost                  : ok=4    changed=0    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0   

Friday 08 April 2022  03:11:59 +0000 (0:00:00.319)       0:42:13.720 ********** 
=============================================================================== 
download : download_file | Validate mirrors --------------------------------------------------------------------------------------------------------------------------------------------------------- 328.91s
download : download_file | Validate mirrors --------------------------------------------------------------------------------------------------------------------------------------------------------- 297.09s
container-engine/runc : download_file | Validate mirrors -------------------------------------------------------------------------------------------------------------------------------------------- 217.00s
container-engine/nerdctl : download_file | Validate mirrors ----------------------------------------------------------------------------------------------------------------------------------------- 116.73s
container-engine/crictl : download_file | Validate mirrors ------------------------------------------------------------------------------------------------------------------------------------------ 111.30s
download : download_file | Validate mirrors --------------------------------------------------------------------------------------------------------------------------------------------------------- 109.08s
download : download_file | Validate mirrors --------------------------------------------------------------------------------------------------------------------------------------------------------- 108.66s
download : download_file | Validate mirrors --------------------------------------------------------------------------------------------------------------------------------------------------------- 108.17s
container-engine/containerd : download_file | Validate mirrors --------------------------------------------------------------------------------------------------------------------------------------- 74.74s
download : download_file | Validate mirrors ---------------------------------------------------------------------------------------------------------------------------------------------------------- 74.15s
download : download_file | Validate mirrors ---------------------------------------------------------------------------------------------------------------------------------------------------------- 73.47s
download : download_file | Validate mirrors ---------------------------------------------------------------------------------------------------------------------------------------------------------- 73.44s
container-engine/nerdctl : download_file | Validate mirrors ------------------------------------------------------------------------------------------------------------------------------------------ 39.11s
kubernetes/control-plane : Joining control plane node to the cluster. -------------------------------------------------------------------------------------------------------------------------------- 34.71s
kubernetes/control-plane : kubeadm | Initialize first master ----------------------------------------------------------------------------------------------------------------------------------------- 32.44s
kubernetes/kubeadm : Join to cluster ----------------------------------------------------------------------------------------------------------------------------------------------------------------- 31.15s
etcd : reload etcd ----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- 21.08s
network_plugin/calico : Wait for calico kubeconfig to be created ------------------------------------------------------------------------------------------------------------------------------------- 10.58s
kubernetes/preinstall : Update package management cache (APT) ---------------------------------------------------------------------------------------------------------------------------------------- 10.41s
etcd : Configure | Wait for etcd cluster to be healthy ------------------------------------------------------------------------------------------------------------------------------------------------ 9.89s


```



## 验证k8s集群


### 查看集群状态

```
[root@kubernetes-1 ~]# kubectl  get node -o wide
NAME           STATUS   ROLES                  AGE   VERSION   INTERNAL-IP   EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION       CONTAINER-RUNTIME
kubernetes-1   Ready    control-plane,master   87m   v1.23.5   10.9.9.91     <none>        Ubuntu 18.04.4 LTS   4.15.0-175-generic   containerd://1.6.1
kubernetes-2   Ready    control-plane,master   86m   v1.23.5   10.9.9.92     <none>        Ubuntu 18.04.4 LTS   4.15.0-175-generic   containerd://1.6.1
kubernetes-3   Ready    control-plane,master   86m   v1.23.5   10.9.9.93     <none>        Ubuntu 18.04.4 LTS   4.15.0-175-generic   containerd://1.6.1
kubernetes-4   Ready    <none>                 85m   v1.23.5   10.9.9.96     <none>        Ubuntu 18.04.4 LTS   4.15.0-175-generic   containerd://1.6.1

```


### 查看集群组件


```
[root@kubernetes-1 ~]# kubectl get all --all-namespaces
NAMESPACE     NAME                                           READY   STATUS    RESTARTS   AGE
kube-system   pod/calico-kube-controllers-75fcdd655b-lbhhj   1/1     Running   0          84m
kube-system   pod/calico-node-lhkbx                          1/1     Running   0          85m
kube-system   pod/calico-node-lsmxg                          1/1     Running   0          85m
kube-system   pod/calico-node-n688h                          1/1     Running   0          85m
kube-system   pod/calico-node-qndjh                          1/1     Running   0          85m
kube-system   pod/coredns-884c84f48-87z87                    1/1     Running   0          84m
kube-system   pod/coredns-884c84f48-t4mmt                    1/1     Running   0          83m
kube-system   pod/dns-autoscaler-5b7fddbc74-zqjpf            1/1     Running   0          84m
kube-system   pod/kube-apiserver-kubernetes-1                1/1     Running   1          87m
kube-system   pod/kube-apiserver-kubernetes-2                1/1     Running   1          87m
kube-system   pod/kube-apiserver-kubernetes-3                1/1     Running   1          86m
kube-system   pod/kube-controller-manager-kubernetes-1       1/1     Running   1          87m
kube-system   pod/kube-controller-manager-kubernetes-2       1/1     Running   1          87m
kube-system   pod/kube-controller-manager-kubernetes-3       1/1     Running   1          86m
kube-system   pod/kube-proxy-5gccw                           1/1     Running   0          85m
kube-system   pod/kube-proxy-drh5z                           1/1     Running   0          85m
kube-system   pod/kube-proxy-r7g22                           1/1     Running   0          85m
kube-system   pod/kube-proxy-xm4vj                           1/1     Running   0          85m
kube-system   pod/kube-scheduler-kubernetes-1                1/1     Running   1          87m
kube-system   pod/kube-scheduler-kubernetes-2                1/1     Running   1          87m
kube-system   pod/kube-scheduler-kubernetes-3                1/1     Running   1          86m
kube-system   pod/nginx-proxy-kubernetes-4                   1/1     Running   0          85m
kube-system   pod/nodelocaldns-b6nk2                         1/1     Running   0          84m
kube-system   pod/nodelocaldns-f9ddv                         1/1     Running   0          84m
kube-system   pod/nodelocaldns-p65m8                         1/1     Running   0          84m
kube-system   pod/nodelocaldns-zdbhg                         1/1     Running   0          84m

NAMESPACE     NAME                 TYPE        CLUSTER-IP   EXTERNAL-IP   PORT(S)                  AGE
default       service/kubernetes   ClusterIP   10.233.0.1   <none>        443/TCP                  87m
kube-system   service/coredns      ClusterIP   10.233.0.3   <none>        53/UDP,53/TCP,9153/TCP   84m

NAMESPACE     NAME                          DESIRED   CURRENT   READY   UP-TO-DATE   AVAILABLE   NODE SELECTOR            AGE
kube-system   daemonset.apps/calico-node    4         4         4       4            4           kubernetes.io/os=linux   85m
kube-system   daemonset.apps/kube-proxy     4         4         4       4            4           kubernetes.io/os=linux   87m
kube-system   daemonset.apps/nodelocaldns   4         4         4       4            4           kubernetes.io/os=linux   84m

NAMESPACE     NAME                                      READY   UP-TO-DATE   AVAILABLE   AGE
kube-system   deployment.apps/calico-kube-controllers   1/1     1            1           84m
kube-system   deployment.apps/coredns                   2/2     2            2           84m
kube-system   deployment.apps/dns-autoscaler            1/1     1            1           84m

NAMESPACE     NAME                                                 DESIRED   CURRENT   READY   AGE
kube-system   replicaset.apps/calico-kube-controllers-75fcdd655b   1         1         1       84m
kube-system   replicaset.apps/coredns-884c84f48                    2         2         2       84m
kube-system   replicaset.apps/dns-autoscaler-5b7fddbc74            1         1         1       84m
```


### 查看 ipvs


```
[root@kubernetes-1 ~]# ipvsadm -L -n
IP Virtual Server version 1.2.1 (size=4096)
Prot LocalAddress:Port Scheduler Flags
  -> RemoteAddress:Port           Forward Weight ActiveConn InActConn
TCP  10.233.0.1:443 rr
  -> 10.9.9.91:6443               Masq    1      1          0         
  -> 10.9.9.92:6443               Masq    1      2          0         
  -> 10.9.9.93:6443               Masq    1      1          0         
TCP  10.233.0.3:53 rr
  -> 10.233.102.1:53              Masq    1      0          0         
  -> 10.233.115.1:53              Masq    1      0          0         
TCP  10.233.0.3:9153 rr
  -> 10.233.102.1:9153            Masq    1      0          0         
  -> 10.233.115.1:9153            Masq    1      0          0         
UDP  10.233.0.3:53 rr
  -> 10.233.102.1:53              Masq    1      0          0         
  -> 10.233.115.1:53              Masq    1      0          0         

```



### Containerd 服务

* 可使用 `nerdctl`、`crictl` 命令操作


```bash

[root@kubernetes-1 ~]#  nerdctl ps

CONTAINER ID    IMAGE                                                                      COMMAND                   CREATED              STATUS    PORTS    NAMES
0437f7a052c6    registry.aliyuncs.com/google_containers/pause:3.3                          "/pause"                  2 hours ago          Up                 k8s://kube-system/kube-scheduler-kubernetes-1                                     
11f3bf27b86a    docker.io/jicki/cluster-proportional-autoscaler-amd64:1.8.5                "/cluster-proportion…"    2 hours ago          Up                 k8s://kube-system/dns-autoscaler-5b7fddbc74-zqjpf/autoscaler                      
25ef1214ce41    registry.aliyuncs.com/google_containers/kube-controller-manager:v1.23.5    "kube-controller-man…"    2 hours ago          Up                 k8s://kube-system/kube-controller-manager-kubernetes-1/kube-controller-manager    
2a775d1787dc    registry.aliyuncs.com/google_containers/kube-proxy:v1.23.5                 "/usr/local/bin/kube…"    2 hours ago          Up                 k8s://kube-system/kube-proxy-drh5z/kube-proxy                                     
722ad9ac470e    docker.io/jicki/k8s-dns-node-cache:1.21.1                                  "/node-cache -locali…"    2 hours ago          Up                 k8s://kube-system/nodelocaldns-b6nk2/node-cache                                   
793809690178    registry.aliyuncs.com/google_containers/pause:3.3                          "/pause"                  About an hour ago    Up                 k8s://kube-system/kube-apiserver-kubernetes-1                                     
7f7b8b940f6c    registry.aliyuncs.com/google_containers/pause:3.3                          "/pause"                  2 hours ago          Up                 k8s://kube-system/kube-controller-manager-kubernetes-1                            
8cdcef52ecec    registry.aliyuncs.com/google_containers/kube-scheduler:v1.23.5             "kube-scheduler --au…"    2 hours ago          Up                 k8s://kube-system/kube-scheduler-kubernetes-1/kube-scheduler                      
c2227239d213    registry.aliyuncs.com/google_containers/pause:3.3                          "/pause"                  2 hours ago          Up                 k8s://kube-system/dns-autoscaler-5b7fddbc74-zqjpf                                 
c274f8608188    registry.aliyuncs.com/google_containers/kube-apiserver:v1.23.5             "kube-apiserver --ad…"    About an hour ago    Up                 k8s://kube-system/kube-apiserver-kubernetes-1/kube-apiserver                      
c3e197f196f5    registry.aliyuncs.com/google_containers/pause:3.3                          "/pause"                  2 hours ago          Up                 k8s://kube-system/calico-node-qndjh                                               
c8e32dde7953    registry.aliyuncs.com/google_containers/pause:3.3                          "/pause"                  2 hours ago          Up                 k8s://kube-system/kube-proxy-drh5z                                                
d1fd556ff7c1    registry.aliyuncs.com/google_containers/pause:3.3                          "/pause"                  2 hours ago          Up                 k8s://kube-system/nodelocaldns-b6nk2                                              
e47e4d5d1905    quay.io/calico/node:v3.21.4                                                "start_runit"             2 hours ago          Up                 k8s://kube-system/calico-node-qndjh/calico-node                                   


```

---


```bash
[root@kubernetes-1 ~]#  nerdctl images


REPOSITORY                                                         TAG                                                                 IMAGE ID        CREATED           PLATFORM       SIZE         BLOB SIZE
jicki/k8s-dns-node-cache                                           1.21.1                                                              04c4f6b1f2f2    16 seconds ago    linux/amd64    168.8 MiB    40.5 MiB
nginx                                                              1.21.4                                                              366e9f1ddebd    3 hours ago       linux/amd64    823.2 MiB    54.1 MiB
quay.io/calico/cni                                                 v3.21.4                                                             36acb85e6080    25 hours ago      linux/amd64    353.1 MiB    76.8 MiB
quay.io/calico/kube-controllers                                    v3.21.4                                                             f71a293e43f6    25 hours ago      linux/amd64    378.3 MiB    52.1 MiB
quay.io/calico/node                                                v3.21.4                                                             acb402642ba8    25 hours ago      linux/amd64    428.9 MiB    70.6 MiB
quay.io/calico/pod2daemon-flexvol                                  v3.21.4                                                             baeaa86e5919    25 hours ago      linux/amd64    75.9 MiB     8.8 MiB
registry.aliyuncs.com/google_containers/kube-apiserver             v1.23.5                                                             ddf5bf7196eb    3 hours ago       linux/amd64    145.8 MiB    31.1 MiB
registry.aliyuncs.com/google_containers/kube-controller-manager    v1.23.5                                                             cca0fb3532ab    3 hours ago       linux/amd64    136.1 MiB    28.8 MiB
registry.aliyuncs.com/google_containers/kube-proxy                 v1.23.5                                                             c1f625d115fb    3 hours ago       linux/amd64    254.9 MiB    37.5 MiB
registry.aliyuncs.com/google_containers/kube-scheduler             v1.23.5                                                             489efb65da9e    3 hours ago       linux/amd64    67.9 MiB     14.4 MiB
registry.aliyuncs.com/google_containers/pause                      3.3                                                                 a319ac2280eb    25 hours ago      linux/amd64    672.0 KiB    292.5 KiB
sha256                                                             0184c1613d92931126feb4c548e5da11015513b9e4c104e7305ee8b53b50a9da    a319ac2280eb    25 hours ago      linux/amd64    672.0 KiB    292.5 KiB
sha256                                                             3c53fa8541f95165d3def81704febb85e2e13f90872667f9939dd856dc88e874    c1f625d115fb    3 hours ago       linux/amd64    254.9 MiB    37.5 MiB
sha256                                                             3fc1d62d65872296462b198ab7842d0faf8c336b236c4a0dacfce67bec95257f    ddf5bf7196eb    3 hours ago       linux/amd64    145.8 MiB    31.1 MiB
sha256                                                             5bae806f8f123c54ca6a754c567e8408393740792ba8b89ee3bb6c5f95e6fbe1    04c4f6b1f2f2    17 seconds ago    linux/amd64    168.8 MiB    40.5 MiB
sha256                                                             884d49d6d8c9f40672d20c78e300ffee238d01c1ccb2c132937125d97a596fd7    489efb65da9e    3 hours ago       linux/amd64    67.9 MiB     14.4 MiB
sha256                                                             ab768d7a914ffead3d0fe5da418af51ad8c26037d2f3f72f07021ea1ea95f93a    baeaa86e5919    25 hours ago      linux/amd64    75.9 MiB     8.8 MiB
sha256                                                             b0c9e5e4dbb14459edc593b39add54f5497e42d4eecc8d03bee5daf9537b0dae    cca0fb3532ab    3 hours ago       linux/amd64    136.1 MiB    28.8 MiB
sha256                                                             c59896fc7ca446a841242e4d09b93600dc828a849f615369bd9c69fa65b439bb    acb402642ba8    25 hours ago      linux/amd64    428.9 MiB    70.6 MiB
sha256                                                             c95ddb97ba59c46acef5fbd8c4aa5d7e0a52c63f963e7a43227c4280de6988ed    f71a293e43f6    25 hours ago      linux/amd64    378.3 MiB    52.1 MiB
sha256                                                             f1de15d70851b1b506f4d0800f847bf6767a3a100baf9be8685e78bc3640db28    36acb85e6080    25 hours ago      linux/amd64    353.1 MiB    76.8 MiB
sha256  


```



> 部署 一个 Nginx `deployment` 服务

```

apiVersion: apps/v1
kind: Deployment 
metadata: 
  name: nginx
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
---

apiVersion: v1 
kind: Service
metadata: 
  name: nginx-svc 
spec: 
  ports: 
    - port: 80
      name: http
      targetPort: 80
      protocol: TCP 
  selector: 
    name: nginx

```

---

```
[root@kubernetes-1 ~]# kubectl  apply -f nginx-deployment.yaml 
deployment.apps/nginx created
service/nginx-svc created

```

---

```
[root@kubernetes-1 ~]# kubectl get pods -o wide
NAME                   READY   STATUS    RESTARTS   AGE   IP             NODE           NOMINATED NODE   READINESS GATES
nginx-5fcd4fb4-l57rz   1/1     Running   0          66s   10.233.127.1   kubernetes-4   <none>           <none>
nginx-5fcd4fb4-zkhtc   1/1     Running   0          66s   10.233.127.2   kubernetes-4   <none>           <none>
```

---

```
[root@kubernetes-1 ~]# kubectl get svc -o wide    
NAME         TYPE        CLUSTER-IP     EXTERNAL-IP   PORT(S)   AGE    SELECTOR
kubernetes   ClusterIP   10.233.0.1     <none>        443/TCP   104m   <none>
nginx-svc    ClusterIP   10.233.58.77   <none>        80/TCP    103s   name=nginx
```

---

```
[root@kubernetes-1 ~]# curl -I http://10.233.58.77
HTTP/1.1 200 OK
Server: nginx/1.21.6
Date: Fri, 08 Apr 2022 04:52:57 GMT
Content-Type: text/html
Content-Length: 615
Last-Modified: Tue, 25 Jan 2022 15:26:06 GMT
Connection: keep-alive
ETag: "61f0168e-267"
Accept-Ranges: bytes

```

---

```
[root@kubernetes-4 ~]# nerdctl ps |grep nginx
0f9639d0f868    registry.aliyuncs.com/google_containers/pause:3.3             "/pause"                  5 minutes ago    Up                 k8s://default/nginx-5fcd4fb4-l57rz                                                    
201f8d0a332d    registry.aliyuncs.com/google_containers/pause:3.3             "/pause"                  5 minutes ago    Up                 k8s://default/nginx-5fcd4fb4-zkhtc                                                    
59092315d085    docker.io/library/nginx:alpine                                "/docker-entrypoint.…"    5 minutes ago    Up                 k8s://default/nginx-5fcd4fb4-l57rz/nginx                                              
ab92c05fcc53    docker.io/library/nginx:alpine                                "/docker-entrypoint.…"    5 minutes ago    Up                 k8s://default/nginx-5fcd4fb4-zkhtc/nginx 

```




## Nginx-ingress

> Ingress  https://kubernetes.github.io/ingress-nginx

* 新版变化

  * apiVersion 更新:   `apiVersion: networking.k8s.io/v1` 
  * 指定:  `--ingress-class=nginx`
  * `backend:` 写法有变化, 具体看后面例子

---

* Download 

```bash

wget https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.2/deploy/static/provider/cloud/deploy.yaml
```


* Update Imges

```bash
image: k8s.gcr.io/ingress-nginx/controller:v1.1.2@sha256:28b11ce69e57843de44e3db6413e98d09de0f6688e33d4bd384002a44f78405c

# 修改为

image: jicki/controller:v1.1.2


```
---


```bash
image: k8s.gcr.io/ingress-nginx/kube-webhook-certgen:v1.1.1@sha256:64d8c73dca984af206adf9d6d7e46aa550362b1d7a01f3a0a91b20cc67868660

# 修改为

image: jicki/kube-webhook-certgen:v1.1.1
```

---


* Edit Config


```bash
# 增加副本数
spec:
  replicas: 3
  minReadySeconds: 0
  revisionHistoryLimit: 10


```

---


```bash

# 查询如下:
    spec:
      dnsPolicy: ClusterFirst


# 修改为如下:
    spec:
      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet


```


---

```bash
# 在如上添加的配置下添加 affinity 与 tolerations


      hostNetwork: true
      dnsPolicy: ClusterFirstWithHostNet
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - kubernetes-1
                - kubernetes-2
                - kubernetes-3
        podAntiAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            - labelSelector:
                matchExpressions:
                  - key: app.kubernetes.io/name
                    operator: In
                    values:
                    - ingress-nginx
              topologyKey: "kubernetes.io/hostname"
      tolerations:
      - key: node-role.kubernetes.io/master
        effect: NoSchedule

```

---

> 创建一个 Nginx 应用, 主要查看 `Ingress` 配置变化


```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 1
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:alpine
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
      name: web
      targetPort: 80
      protocol: TCP
  selector:
    app: nginx
  type: ClusterIP
---
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
spec:
  ingressClassName: nginx
  rules:
  - host: nginx.jicki.cn
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: nginx-svc
            port:
              number: 80

```



# upgrades 版本


## upgrades

> 优雅更新 版本


```
git fetch origin
git checkout origin/master

ansible-playbook -i inventory/jicki/inventory.ini --become --become-user=root upgrade-cluster.yml -e kube_version=v1.23.6


```

