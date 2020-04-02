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
   
    * 对于这种情况我们可以使用 `terminationGracePeriodSeconds`。

    * `terminationGracePeriodSeconds` 当 `kubernetes` 准备删除一个 pod 时，会向该 pod 中的容器发送 TERM 信号并同时将 pod 从 service 的 endpoint 列表中移除。如果容器无法在规定时间（默认 30 秒）内终止，k8s 会向容器发送 SIGKILL 信号强制终止进程。 

  * 应用升级过程中需要保证即将下线的应用实例不会接收到新的请求且有足够时间处理完当前请求。 

    * 对于这种情况我们可以使用 `maxSurge` 和 `maxUnavailable`。

    * `maxSurge`  指定在滚动更新过程中最多可创建多少个额外的 pod, 可以是数字或百分比。

    * `maxUnavailable` 指定在滚动更新过程中最多允许多少个 pod 不可用, 可以是数字或百分比。 



## deployment 例子

  * 基于 java 的应用能更好的体现大多数的问题

```
apiVersion: apps/v1
kind: Deployment 
metadata: 
  name: spring-boot-probes
  labels:
    app: spring-boot-probes
spec:
  replicas: 4
  strategy:
    # 配置 滚动更新 策略
    type: RollingUpdate
    rollingUpdate:
      # 更新时增加新的 pods 数量(4 + 2 = 6)
      maxSurge: 2
      # 更新时允许最大 pods 数量不对外服务
      maxUnavailable: 1
  # 新的 pods 就绪状态观察时间。
  # 必须保证处于就绪状态的 pod 能经历一个完整的活性探测周期。
  minReadySeconds: 120
  selector:
    matchLabels:
      name: spring-boot-probes
  template:
    metadata:
      labels:
        app: spring-boot-probes
        version: v1.0.0
    spec:
      containers:
      - name: spring-boot-probes
        image: registry.cn-hangzhou.aliyuncs.com/log-service/spring-boot-probes:1.0.0
        ports:
        - containerPort: 8080
	# resources 资源限制
        resources:
          limits:
            cpu: 500m
            memory: 400Mi
          requests:
            cpu: 200m
            memory: 200Mi
        # 就绪探针 如果探针判断失败,则不会有流量发往到这个pod。 
        readinessProbe:
          # http Get请求
          httpGet:
            path: /actuator/health
            port: 8080
          # 初始化检测时间, 指启动后30s开始探针检测
          initialDelaySeconds: 30
          # 探针探测的时间间隔为10s
          periodSeconds: 10
          # 探针探测失败后, 最少连续探测成功多少次才被认定为成功 
          successThreshold: 1
          # 探测成功后, 最少连续探测失败多少次才被认定为失败
          failureThreshold: 1
          # periodSeconds = 10s failureThreshold = 1 大约 10 秒后就不会有流量发往它
        # 活性探测 如果探针判断失败, 则会重启这个 pod。
        livenessProbe:
          # http Get请求
          httpGet:
            path: /actuator/health
            port: 8080
          # 初始化检测时间, 指启动后40s开始探针检测
          # 必须在 readinessProbe 探针检测之后
          initialDelaySeconds: 40
          # 探针探测的时间间隔为20s
          periodSeconds: 20
          # 探针探测失败后, 最少连续探测成功多少次才被认定为成功
          successThreshold: 1
          # 探测成功后, 最少连续探测失败多少次才被认定为失败
          failureThreshold: 3
          # periodSeconds = 20s failureThreshold = 3 当容器异常时, 大约 60 秒后就不会被重启。

---
apiVersion: v1 
kind: Service
metadata: 
  labels:
    app: spring-boot-probes
  name: spring-boot-probes-svc 
spec: 
  ports: 
    - port: 8080
      name: http
      targetPort: 8080
      protocol: TCP 
  selector: 
    name: spring-boot-probes
```



  
