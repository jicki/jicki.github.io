---
layout: post
title: Container Network Interface - CNI
categories: flannel
description: Container Network Interface - CNI
keywords: kubernetes
feature-img: "assets/img/pexels/desk-top.jpeg"
catalog:    true
tags:
    - kubernetes
    - network
---


# CNI

> CNI 既 Container Network Interface 的简写.

* **CNI 是 Kubernetes 网络的一个标准**

  * `CNI` 提供 `POD` 的网络连接.

  * `CNI` 提供分配IP地址的能力, 为`POD` 提供 ipv4 与 ipv6 的 DHCP 功能.

  * `CNI` 还提供了 `NetWrok Policy` 的能力, 定义 `POD` 与 `POD` 之间的网络规则.


- - -

* **CNI运行原理**

  * 根据`CNI`标准, 网络组件需要在 `CNI` 下创建二进制执行文件, 用于配置网络环境.

  * 当 `kubernetes` 创建/删除 `pod` 时, 调用二进制执行文件进行相应的网络配置. 

  * `CNI` 下的二进制执行文件会包含如下参数

    * `Container ID`&emsp; 容器ID号, 需要用于指定此容器配置网络, 也有一些`CNI`组件需要利用容器ID进行数据索引.

    * `Network namespace path`&emsp; 当前 `container` 真正的 `network namespace` 路径 . 每个 `container` 都会拥有至少一个 `network namespace`. `namespace` 是通过 `Linux kernel` 操作的, 因此大部分 `CNI` 组件都会根据这个路径, 通过 `network namespace` 进行相关的网络配置.

    * `Network configuration`&emsp; 用于定义`CNI`组件的相关配置. 包含两部分组成 `CNI`标准 和 网络组件额外定义.

    * `Name of the interface inside the container`&emsp; 在当前`container` 中创建网卡名称, 如: `eth0` 等.

 
- - -

**network namespace**

* 如上面所说每个 `container` 都至少拥有一个 `network namespace`, 但两个 `container` 可以共用一个相同的 `network namespace` .  

```
# 创建一个容器 n1
[root@k8s-node-1 ~]# docker run -d --name n1 alpine sleep 3600s


# 查看容器相关网络
[root@k8s-node-1 ~]# docker exec -it n1 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

```


```
# 创建另一个容器 n2 并配置网络 --net=container:{容器id}
# 容器id 是 n1 的容器id
[root@k8s-node-1 ~]# docker run -d --net=container:b27fa0d5db21 --name n2 alpine sleep 3600s


# 查看容器相关网络
[root@k8s-node-1 ~]# docker exec -it n2 ifconfig
eth0      Link encap:Ethernet  HWaddr 02:42:AC:11:00:02
          inet addr:172.17.0.2  Bcast:172.17.255.255  Mask:255.255.0.0
          UP BROADCAST RUNNING MULTICAST  MTU:1500  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:0
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)

lo        Link encap:Local Loopback
          inet addr:127.0.0.1  Mask:255.0.0.0
          UP LOOPBACK RUNNING  MTU:65536  Metric:1
          RX packets:0 errors:0 dropped:0 overruns:0 frame:0
          TX packets:0 errors:0 dropped:0 overruns:0 carrier:0
          collisions:0 txqueuelen:1
          RX bytes:0 (0.0 B)  TX bytes:0 (0.0 B)
```


* 可以看到如上两个容器的网络都是同一个, 这里就可以想到 `kubernetes` 的 `pod` 的网络.

* 实际 `pod` 中有更灵活的设置, 详见如下架构图

![cni][1]



- - -


**Network Configuration**

* `Network Configuration` 就是一个 `json` 格式的配置文件. 如上我们讲过 这个配置文件包含两部分

  * `CNI` 标准的配置 - 根据官方 `CNI Spec` 包含如下配置项

    * `cniVersion`&emsp; 定义`CNI`的版本, `CNI` 组件必须相同, 否则报错.

    * `name`&emsp; 唯一的名称标识.

    * `type`&emsp; `CNI`组件定义的名称, 上层应用会根据这个名称去找对应的`CNI`组件. 然后调用`CNI`组件的二进制文件进行网络配置. 这个配置非常重要.

    * `args`&emsp; 额外的配置, 主要用于上层应用调用`CNI`时的一些额外参数. 

    * `ipMasq`&emsp; 标注当前`CNI`组件是否支持 `SNAT`. 只做标识, 实际`SNAT`是在`CNI`组件中实现.

    * `dns`&emsp; `CNI`组件的 DNS 配置, 如: `nameserver`、`search`、`domain`、`options` 等. 注: 在`kubernetes`中, 会忽略这里的 DNS 配置, 因为 `kubernetes` 会在自身设置 DNS 配置.

    * `ipam`&emsp; 既 `IP Address Management Plugin`, 负责IP地址的分配. 一般为`host-local` 和 `dhcp` .


* 一个配置文件

```
{
  "cniVersion": "0.4.0",
  "name": "dbnet",
  "type": "bridge",
  // type (plugin) specific
  "bridge": "cni0",
  "ipam": {
    "type": "host-local",
    // ipam specific
    "subnet": "10.1.0.0/16",
    "gateway": "10.1.0.1"
  },
  "dns": {
    "nameservers": [ "10.1.0.1" ]
  }
}
```




  [1]: http://jicki.me/img/posts/cni/cni.png

