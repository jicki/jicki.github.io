---
layout: post
title: hyperledger-fabric v2.0
categories: fabric
description: hyperledger-fabric v2.0
keywords: fabric
feature-img: "assets/img/pexels/desk-top.jpeg"
catalog:    true
tags:
    - fabric
    - docker
---

> fabric v2.0 , 单机 多节点 RAFT 手动部署, 所有服务均 开启 SSL 认证。


# hyperledger fabric



## Hyperleger fabric 是什么?

* Linux基金会于2015年创建了Hyperledger项目，以推进跨行业的区块链技术。Hyperledger Fabric是Hyperledger中的区块链项目之一。像其他区块链技术一样，它具有分类帐，使用智能合约，并且是参与者用来管理其交易的系统。Hyperledger Fabric与其他区块链系统最大的不同体现在私有和许可。

![图1][1]


* Hyperledger Fabric 是一个（平台），提供分布式账本解决方案的平台，Hyperledger Fabric 由模块化架构支撑，并具备极佳的`保密性`、`可伸缩性`、`灵活性`、`可扩展性`，Hyperledger Fabric 被设计成（模块直接插拔，适用多种场景），支持不同的模块组件直接拔插启用，并能适应在经济生态系统中错综复杂的各种场景。




## Hyperledger Fabric 交易流程

![图2][2]






## 部署 Hyperledger Fabric v2.0

> 官网 github ---  https://github.com/hyperledger/fabric



### 准备阶段

* docker-ce

* docker-compose

* Hyperledger Fabric images



### 安装 docker


* 升级内核

```shell

# 导入 Key

rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org



# 安装 Yum 源

rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm



# 更新 kernel

yum --enablerepo=elrepo-kernel install -y kernel-lt kernel-lt-devel


# 配置 内核优先

grub2-set-default 0

```


* 开启内核 namespace 支持

```
# 执行如下
grubby --args="user_namespace.enable=1" --update-kernel="$(grubby --default-kernel)"



# 必须重启系统

reboot

```




* 修改内核参数

```
cat<<EOF > /etc/sysctl.d/docker.conf
# 要求iptables不对bridge的数据进行处理
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-arptables = 1
EOF



# 生效配置
sysctl --system
```


* 检查环境状态

```shell
curl -s https://raw.githubusercontent.com/docker/docker/master/contrib/check-config.sh | bash

```



* 安装 docker

```
# 指定安装,并指定安装源

export VERSION=19.03
curl -fsSL "https://get.docker.com/" | bash -s -- --mirror Aliyun

```


* docker 额外配置

```shell
mkdir -p /etc/docker/
cat>/etc/docker/daemon.json<<EOF
{
  "bip": "172.17.0.1/16",
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://dockerhub.azk8s.cn","https://gcr.azk8s.cn","https://quay.azk8s.cn"],
  "data-root": "/opt/docker",
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m",
    "max-file": "5"
  }
}
EOF
```


* 启动 docker


```shell
# 配置开机启动
systemctl enable docker


# 启动 docker
systemctl start docker

```


### 安装 docker-compose


```
# 国内下载点
curl -L https://get.daocloud.io/docker/compose/releases/download/1.25.4/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose



# github 下载
curl -L "https://github.com/docker/compose/releases/download/1.25.4/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose

chmod +x /usr/local/bin/docker-compose
```



### Hyperledger Fabric images

* 由于 image 镜像较大,下载过程会比较久, 所以这里提前将 images 下载到本地。


```shell
#!/bin/bash

FABRIC_TAG=2.0

function pullImages() {
  docker pull hyperledger/fabric-ca:1.4.6
  for image in peer orderer ccenv tools; do
    echo "Pull image hyperledger/fabric-$image:$FABRIC_TAG"
    docker pull hyperledger/fabric-"$image":"$FABRIC_TAG"
    sleep 1
  done
}

echo "Pull images for hyperledger fabric network."

pullImages

```

```shell
[root@localhost jicki]# docker images
REPOSITORY                   TAG                 IMAGE ID            CREATED             SIZE
hyperledger/fabric-tools     2.0                 5c9a03790913        3 weeks ago         512MB
hyperledger/fabric-peer      2.0                 5c7e5946f3dc        3 weeks ago         57.2MB
hyperledger/fabric-orderer   2.0                 92bd220edcdd        3 weeks ago         39.7MB
hyperledger/fabric-ccenv     2.0                 800087268d9b        3 weeks ago         529MB
hyperledger/fabric-ca        1.4.6               3b96a893c1e4        3 weeks ago         150MB

```




### 创建 crypto-config.yaml 文件


* Fabric 网络成员加密配置

* 如下配置 3个 orderer 节点, 2个Org组织, 每个组织下有 2个Peer节点。

```
OrdererOrgs:
  - Name: Orderer
    Domain: jicki.me
    CA:
        Country: CN
        Province: GuangDong
        Locality: ShenZhen
    Specs:
      - Hostname: orderer0
      - Hostname: orderer1
      - Hostname: orderer2

PeerOrgs:
  - Name: Org1
    Domain: org1.jicki.me
    EnableNodeOUs: true
    CA:
        Country: CN
        Province: GuangDong
        Locality: ShenZhen
    Template:
      Count: 2
    Users:
      Count: 1

  - Name: Org2
    Domain: org2.jicki.me
    EnableNodeOUs: true
    CA:
        Country: CN
        Province: GuangDong
        Locality: ShenZhen
    Template:
      Count: 2
    Users:
      Count: 1

```


### 创建 configtxgen.yaml 文件

* 网络初始化以及通道配置



```shell

---
Organizations:
    - &OrdererOrg
        Name: OrdererOrg
        ID: OrdererMSP
        MSPDir: ./channel-artifacts/crypto-config/ordererOrganizations/jicki.me/msp
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Writers:
                Type: Signature
                Rule: "OR('OrdererMSP.member')"
            Admins:
                Type: Signature
                Rule: "OR('OrdererMSP.admin')"

        OrdererEndpoints:
            - orderer0.jicki.me:7050
    - &Org1
        Name: Org1MSP
        ID: Org1MSP
        MSPDir: ./channel-artifacts/crypto-config/peerOrganizations/org1.jicki.me/msp
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.peer', 'Org1MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org1MSP.admin', 'Org1MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org1MSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('Org1MSP.peer')"

        AnchorPeers:
            - Host: peer0.org1.jicki.me
              Port: 7051
    - &Org2
        Name: Org2MSP
        ID: Org2MSP
        MSPDir: ./channel-artifacts/crypto-config/peerOrganizations/org2.jicki.me/msp
        Policies:
            Readers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.peer', 'Org2MSP.client')"
            Writers:
                Type: Signature
                Rule: "OR('Org2MSP.admin', 'Org2MSP.client')"
            Admins:
                Type: Signature
                Rule: "OR('Org2MSP.admin')"
            Endorsement:
                Type: Signature
                Rule: "OR('Org2MSP.peer')"

        AnchorPeers:
            - Host: peer0.org2.jicki.me
              Port: 7051
Capabilities:
    Channel: &ChannelCapabilities
        V2_0: true

    Orderer: &OrdererCapabilities
        V2_0: true

    Application: &ApplicationCapabilities
        V2_0: true

Application: &ApplicationDefaults
    Organizations:
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        LifecycleEndorsement:
            Type: ImplicitMeta
            Rule: "MAJORITY Endorsement"
        Endorsement:
            Type: ImplicitMeta
            Rule: "MAJORITY Endorsement"

    Capabilities:
        <<: *ApplicationCapabilities

Orderer: &OrdererDefaults
    OrdererType: etcdraft
    BatchTimeout: 2s
    BatchSize:
        MaxMessageCount: 10
        AbsoluteMaxBytes: 99 MB
        PreferredMaxBytes: 512 KB
    Organizations:
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
        BlockValidation:
            Type: ImplicitMeta
            Rule: "ANY Writers"

Channel: &ChannelDefaults
    Policies:
        Readers:
            Type: ImplicitMeta
            Rule: "ANY Readers"
        Writers:
            Type: ImplicitMeta
            Rule: "ANY Writers"
        Admins:
            Type: ImplicitMeta
            Rule: "MAJORITY Admins"
    Capabilities:
        <<: *ChannelCapabilities

Profiles:
    TwoOrgsChannel:
        Consortium: SampleConsortium
        <<: *ChannelDefaults
        Application:
            <<: *ApplicationDefaults
            Organizations:
                - *Org1
                - *Org2
            Capabilities:
                <<: *ApplicationCapabilities

    SampleMultiNodeEtcdRaft:
        <<: *ChannelDefaults
        Capabilities:
            <<: *ChannelCapabilities
        Orderer:
            <<: *OrdererDefaults
            OrdererType: etcdraft
            EtcdRaft:
                Consenters:
                - Host: orderer0.jicki.me
                  Port: 7050
                  ClientTLSCert: ./channel-artifacts/crypto-config/ordererOrganizations/jicki.me/orderers/orderer0.jicki.me/tls/server.crt
                  ServerTLSCert: ./channel-artifacts/crypto-config/ordererOrganizations/jicki.me/orderers/orderer0.jicki.me/tls/server.crt
                - Host: orderer1.jicki.me
                  Port: 7050
                  ClientTLSCert: ./channel-artifacts/crypto-config/ordererOrganizations/jicki.me/orderers/orderer1.jicki.me/tls/server.crt
                  ServerTLSCert: ./channel-artifacts/crypto-config/ordererOrganizations/jicki.me/orderers/orderer1.jicki.me/tls/server.crt
                - Host: orderer2.jicki.me
                  Port: 7050
                  ClientTLSCert: ./channel-artifacts/crypto-config/ordererOrganizations/jicki.me/orderers/orderer2.jicki.me/tls/server.crt
                  ServerTLSCert: ./channel-artifacts/crypto-config/ordererOrganizations/jicki.me/orderers/orderer2.jicki.me/tls/server.crt
            Addresses:
                - orderer0.jicki.me:7050
                - orderer1.jicki.me:7050
                - orderer2.jicki.me:7050

            Organizations:
            - *OrdererOrg
            Capabilities:
                <<: *OrdererCapabilities
        Application:
            <<: *ApplicationDefaults
            Organizations:
            - <<: *OrdererOrg
        Consortiums:
            SampleConsortium:
                Organizations:
                - *Org1
                - *Org2

```



### 生成证书和创世区块


```shell
# 下载证书生成工具

cd /opt/jicki

wget https://github.com/hyperledger/fabric/releases/download/v2.0.0/hyperledger-fabric-linux-amd64-2.0.0.tar.gz

```

```shell
[root@localhost jicki]# tar zxvf hyperledger-fabric-linux-amd64-2.0.0.tar.gz 
bin/
bin/configtxgen
bin/orderer
bin/peer
bin/discover
bin/idemixgen
bin/configtxlator
bin/cryptogen
config/
config/configtx.yaml
config/core.yaml
config/orderer.yaml
```


```shell
# 为方便使用 我们配置一个 环境变量

vi /etc/profile


# fabric env
export PATH=$PATH:/opt/jicki/bin


# 使文件生效
source /etc/profile
```


```shell
# 首先需要创建一个文件夹
mkdir -p /opt/jicki/channel-artifacts

```


* 生成证书

```shell
# 然后这里使用 cryptogen 软件来生成相应的证书了

[root@localhost jicki]# cryptogen generate --config=./crypto-config.yaml
org1.jicki.me
org2.jicki.me

```

* 生成区块

> `-profile` 在 configtx.yaml 文件中查找 Profiles: 下面字段

```shell

# 生成启动 orderer 所需要的创世区块文件

configtxgen -profile SampleMultiNodeEtcdRaft \
 -outputBlock ./channel-artifacts/genesis.block \
 -channelID mychannel

```

```shell
# 输出如下:
2020-03-23 19:01:00.548 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-03-23 19:01:00.576 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 002 orderer type: etcdraft
2020-03-23 19:01:00.576 CST [common.tools.configtxgen.localconfig] completeInitialization -> INFO 003 Orderer.EtcdRaft.Options unset, setting to tick_interval:"500ms" election_tick:10 heartbeat_tick:1 max_inflight_blocks:5 snapshot_interval_size:16777216 
2020-03-23 19:01:00.576 CST [common.tools.configtxgen.localconfig] Load -> INFO 004 Loaded configuration: /opt/jicki/configtx.yaml
2020-03-23 19:01:00.578 CST [common.tools.configtxgen] doOutputBlock -> INFO 005 Generating genesis block
2020-03-23 19:01:00.578 CST [common.tools.configtxgen] doOutputBlock -> INFO 006 Writing genesis block
```



```shell
# 生成用于配置自定义 channel 的交易文件

configtxgen \
    -profile TwoOrgsChannel \
    -outputCreateChannelTx ./channel-artifacts/channel.tx \
    -channelID mychannel

```

```shell
# 输出如下:

2020-03-23 19:02:33.902 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-03-23 19:02:33.929 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /opt/jicki/configtx.yaml
2020-03-23 19:02:33.929 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 003 Generating new channel configtx
2020-03-23 19:02:33.932 CST [common.tools.configtxgen] doOutputChannelCreateTx -> INFO 004 Writing new channel tx

```




```shell
# 生成用于配置 anchor peer 的交易文件


# Org1

configtxgen \
    -profile TwoOrgsChannel \
    -outputAnchorPeersUpdate ./channel-artifacts/Org1MSPanchors.tx \
    -channelID mychannel \
    -asOrg Org1MSP



# Org2

configtxgen \
  -profile TwoOrgsChannel \
  -outputAnchorPeersUpdate ./channel-artifacts/Org2MSPanchors.tx \
  -channelID mychannel \
  -asOrg Org2MSP


```




```shell
# 输出如下:

2020-03-23 19:04:41.859 CST [common.tools.configtxgen] main -> INFO 001 Loading configuration
2020-03-23 19:04:41.888 CST [common.tools.configtxgen.localconfig] Load -> INFO 002 Loaded configuration: /opt/jicki/configtx.yaml
2020-03-23 19:04:41.889 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 003 Generating anchor peer update
2020-03-23 19:04:41.890 CST [common.tools.configtxgen] doOutputAnchorPeersUpdate -> INFO 004 Writing anchor peer update

```
























  [1]: http://jicki.me/img/posts/fabric/fabric.png
  [2]: http://jicki.me/img/posts/fabric/fabric-2.png
