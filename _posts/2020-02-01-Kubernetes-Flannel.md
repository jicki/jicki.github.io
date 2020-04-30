---
layout: post
title: Flannel 网络模型
categories: flannel
description: Flannel 网络模型
keywords: kubernetes
feature-img: "assets/img/pexels/desk-top.jpeg"
catalog:    true
tags:
    - kubernetes
    - network
---

# Flannel

![flannel][1]


**Flannel 是 `CoreOS` 公司针对 `Kubernetes` 设计的一个基于`CNI`标准的网络工具, 其目在于帮助 `Kuberentes` 实现一个简单的3层网络。**

- - -

**常用网络术语**

* **2层网络**

  * `OSI` 网络模型中的数据链路层.
  * 处理网络上两个相邻节点之间的帧传递.
  * 以太网就工作在第2层, `MAC`地址表示为子层.


* **3层网络**

  * `OSI` 网络模型中的网络层.
  * 处理主机之间路由数据包.
  * IPv4、IPv6、ICMP 工作在第3层.


* **VXLAN**

  * VXLAN代表虚拟可扩展的`LAN`.
  * VXLAN是一种封装和覆盖协议, 可在现有网络上运行. 
  * VXLAN虚拟化与VLAN类似, 但提供更大的灵活性和功能, VLAN只有4096个网络ID.
  * VXLAN用于通过在UDP数据报中封装第2层以太网帧来实现大型网络部署.

* **Overlay**

  * Overlay网络是建立在现有网络之上的虚拟逻辑网络.
  * Overlay网络通常用于在现有网络之上提供有用的抽象, 并分离和保护不同的逻辑网络.

* **数据封装**

  * 数据封装是指在附加层中封装网络数据包以提供其他上下文和信息的过程.
  * 在Overlay网络中, 数据封装被用于从虚拟网络转换到底层地址空间, 从而能路由到不同的位置. 


* **网状网络(Mesh NetWork)**

  * 网状网络是指每个节点连接到许多其他节点以协作路由、并实现更大连接的网络.
  * 网状网络允许通过多个路径进行路由, 从而提供更可靠的网络.
  * 网状网络的缺点是每个附加节点都会增加大量开销.


* **BGP**

  * BGP代表"边界网关协议", 用于管理边缘路由器之间数据包的路由方式.
  * BGP通过考虑可用路径, 路由规则和特定网络策略, 帮助弄清楚如何将数据包从一个网络发送到另一个网络. 
  * BGP有时被用作CNI插件中的路由机制, 而不是封装的覆盖网络.














  [1]: http://jicki.me/img/posts/flannel/flannel-logo.png
