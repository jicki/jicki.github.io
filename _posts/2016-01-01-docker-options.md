---
layout: post
title: docker 基础设置
categories: docker
description: docker 基础设置
keywords: docker
catalog:    true
header-img: "img/pexels/triangular.jpeg"
---


# docker 基础


## docker install


```
# 官网安装
wget -qO- https://get.docker.com/ | sh

```


## docker 设置


```
# 第一种方式 修改 /lib/systemd/system/docker.service


ExecStart=/usr/bin/dockerd 

# 修改为

ExecStart=/usr/bin/dockerd --graph=/opt/docker --registry-mirror=http://b438f72b.m.daocloud.io --iptables=false


```


```
# 第二种方式，增加 opts

mkdir -p /etc/systemd/system/docker.service.d/


# 增加 docker.service 文件

cat >> /etc/systemd/system/docker.service << EOF

[Unit]
Description=Docker Application Container Engine
Documentation=http://docs.docker.com
After=network.target docker-storage-setup.service
Wants=docker-storage-setup.service

[Service]
Type=notify
Environment=GOTRACEBACK=crash
ExecReload=/bin/kill -s HUP $MAINPID
Delegate=yes
KillMode=process
ExecStart=/usr/bin/docker daemon \
          $DOCKER_OPTS \
          $DOCKER_STORAGE_OPTIONS \
          $DOCKER_NETWORK_OPTIONS \
          $DOCKER_DNS_OPTIONS \
          $INSECURE_REGISTRY
TasksMax=infinity
LimitNOFILE=1048576
LimitNPROC=1048576
LimitCORE=infinity
TimeoutStartSec=1min
Restart=on-abnormal

[Install]
WantedBy=multi-user.target
EOF



# 增加配置文件

cat >> /etc/systemd/system/docker.service.d/docker-options.conf << EOF
[Service]
Environment="DOCKER_OPTS=--graph=/opt/docker --registry-mirror=http://b438f72b.m.daocloud.io --iptables=false"
EOF

```
