---
layout: post
title: Kubernetes RollingUpdate
categories: kubernetes
description: Kubernetes RollingUpdate 
keywords: kubeernetes
feature-img: "assets/img/pexels/desk-top.jpeg"
catalog:    true
tags:
    - kubernetes
---


# RollingUpdate

  * kubernetes 应用升级操作

    * `Deployment`

    * `StatefulSet`

    * `DaemonSet`


## 应用部署/升级

> kubernetes 在`deployment` `statefulset` `daemonset` 应用yaml中, 配置 `RollingUpdate`  kubernetes 会将 `pod` 分批次逐步替换掉，可用来实现服务热升级。


* 一般来讲我们`部署`/`升级`应用都会遇到如下问题:

  * pod启动以后, 应用需要花费一定的时间初始化(加载配置), 在这段时间内,程序是无法对外提供服务。

    * 对于这种情况我们可以使用 `ReadinessProbe` (就绪探针) 。

    * `ReadinessProbe` 能让 `kubernetes` 更准确地判断容器是否就绪，从而构建出更健壮的应用。`kubernetes` 保证只有 pod 中的所有容器全部通过了就绪探测，才允许 service 将流量发往该 pod。一旦就绪探测失败，`kubernetes` 会停止将流量发往该 pod 。

    * `ReadinessProbe` 探针的配置非常灵活，用户可以指定探针的探测频率、探测成功阈值、探测失败阈值等。

  * pod启动完成, 应用也启动成功, 但是并不代表应用此时就可以正常提供服务。

    * 对于这种情况我们可以使用 `LivenessProbe` (活性探针)。

    * `LivenessProbe` 能让 `kubernetes` 更准确地判断容器是否正常运行。如果容器没有通过活性探测，kubelet 会将其停止，并根据重启策略决定下一步的动作。 

    * `LivenessProbe` 探针的配置非常灵活，用户可以指定探针的探测频率、探测成功阈值、探测失败阈值等。

  * 应用如果无法对外提供服务,此时应用不一定就能立刻退出。

  * 应用升级过程中需要保证即将下线的应用实例不会接收到新的请求且有足够时间处理完当前请求。 



