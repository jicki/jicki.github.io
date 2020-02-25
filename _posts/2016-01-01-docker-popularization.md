---
layout: post
title: docker 基础知识
categories: docker
description: docker 基础知识
keywords: docker
catalog:    true
header-img: "img/pexels/triangular.jpeg"
---


# Docker

* 本文用于科普 docker 的一些基础知识以及概念

* LXC = Linux Container

## 一、什么是docker

* Docker 是一个开放源代码软件项目, 让应用程序部署在软件货柜下的工作可以自动化进行, 借此在 Linux 操作系统上, 提供一个额外的软件抽象层, 以及操作系统层虚拟化的自动管理机制。Docker 利用 Linux 核心中的资源分离机制，例如 `cgroups`,以及 Linux 核心名字空间`(LXC Namespace)`，来创建独立的容器。


## 二、资源限制与隔离

* 资源隔离 (Namespace):

  * Linux 实现了 6 项资源隔离 

|Namespace|系统调用参数|隔离内容|支持内核版本|
|-|-|-|-|
|UTS|`CLONE_NEWUTS`|主机名和域名|2.6.19|
|IPC|`CLONE_NEWIPC`|信号量、消息队列和共享内存|2.6.19|
|PID|`CLONE_NEWPID`|进程编号|2.6.24|
|NetWork|`CLONE_NEWNET`|网络设备、网络栈、端口等|2.6.29|
|Mount|`CLONE_NEWNS`|挂载点(文件系统)|2.4.19|
|User|`CLONE_NEWUSER`|用户和用户组|3.8|


* 资源限制 (cgroups):

  * Linux 通过 cgroups 实现 cpu、内存、IO的限制以及网络的分配, (Linux 系统目录 /sys/fs/cgroup/ )

  * 限制内存以及虚拟内存并关闭oom-kill  `docker run --memory 500M --memory-swap 200M --oom-kill-disable `

  * 绑定CPU核心 `docker run --cpu-cpus 1` , `docker run --cpuset-cpus="0-2"` 
 
  * 限制CPU `docker run -c 2048 --cpu-period=50000 --cpu-quota=50000` 

    * `--cpu-quota` cpu调度范围    
  
    * `--cpu-period` cpu调度周期内使用时间
 
    * `-c` cpu 核心ms值

  * 限制磁盘IO

    * `--device-read-bps` 限制此设备上的读速度(bytes per second), 单位可以是kb、mb或者gb

    * `--device-read-iops` 通过每秒读IO次数来限制指定设备的读速度

    * `--device-write-bps` 限制此设备上的写速度（bytes per second），单位可以是kb、mb或者gb。

    * `--device-write-iops` 通过每秒写IO次数来限制指定设备的写速度。

    * `--blkio-weight` 容器默认磁盘IO的加权值，有效值范围为10-100。

    * `--blkio-weight-device` 针对特定设备的IO加权控制。其格式为`DEVICE_NAME:WEIGHT`


 
## 三、容器 VS 虚拟机 

  * 相比传统的虚拟化技术(vm), 使用Docker在CPU, Memory, Disk IO, Network IO  上的性能损耗都有同样水平甚至更优的表现。Container的快速创建、启动、销毁受到很多赞誉。

  * 容器是直接使用宿主机的内核的, 而虚拟机(VM)是虚拟化一个完整操作系统。所以单数量上一台物理机能跑的容器是远大于虚拟机的数量。 


