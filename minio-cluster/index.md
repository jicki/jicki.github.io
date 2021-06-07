# Minio Cluster 详解



> Minio 


# Minio

* 官方详细文档  `https://docs.min.io/docs`


## Minio 介绍


**MinIO 是根据GNU Affero 通用公共许可证 v3.0 发布的高性能对象存储。它与 Amazon S3云存储服务兼容。**




## Minio Erasure Code

**Minio 使用 Erasure Code 提供数据的冗余.**

* `Erasure Code` 是一种恢复丢失和损坏数据的数学算法,  Minio 采用 `Reed-Solomon Code` 将对象拆分成 N/2 数据 和 N/2 奇偶校验块。 

* 这就意味着如果是12块盘, 一个对象会被分成6个数据块、6个奇偶校验块, 你可以丢失任意6块盘（不管其是存放的数据块还是奇偶校验块）, 你仍可以从剩下的盘中的数据进行恢复.

* 默认情况下, MinIO 在 N / 2 个数据 和 N / 2 个奇偶校验驱动器上分片对象。我们建议使用 N / 2 数据和 奇偶校验块，因为它可以确保对驱动器故障提供最佳保护。

* MinIO对每个对象分别进行编码，因此可以逐步修复对象。部署后的存储服务器在服务器的整个生命周期内都不需要更换驱动器或进行修复。MinIO的擦除编码后端旨在提高运营效率，并在可用时充分利用硬件加速的优势。


> Erasure Code 数据图


![Erasure-code][1]






* `DataDrives ：` 纠删码中的数据盘，用来存储 Object 原始数据。

* `ParityDrives：` 纠删码中的冗余盘，用来存储 Object 计算出来的冗余数据。


* Minio 纠删码的默认配置为 `1 : 1`, 即数据盘 ( DataDrives )和 冗余盘 ( ParityDrives ) 个数相同， 所以我们真正可用的存储空间，只有我们总空间的一半大小。



---


![architecture][2]





* 因为一个 `ErasureCode 组/ Sets` 中有多个 Drive（DataDrives + ParityDrives）, 所以 Minio 会先将一块 Block 按照 Drives 数量, 划分为多个小块, 这些小块在 Minio 中叫做 Shards。比如一个 Block 是 10MB， 而 Set 里有 16 个 Drive（8 DataDrives + 8 ParityDrives）, 此时 Minio 会将 Block 按照 10 MB / 8 DataDrives 的方式，将数据划分到 8 个 Data Shards，并额外再创建 8 个 空 Shards，用来存储编码后的冗余数据。

* 接着 Minio 就会对 Data Shards 进行纠删码编码, 并将编码后的冗余数据存储到前面创建的 8 个空 Shards 中, 也就是 parity shards 中。




## 数据保护


**分布式 MinIO 使用 `Erasure Code` 提供针对 多个节点/驱动器 故障 和 `Bit Rot` 保护。**


* 由于分布式 MinIO 所需的 最小磁盘为 4（与擦除编码所需的最小磁盘相同）,  因此在您启动分布式 MinIO 时，擦除代码会自动启动。

* 因此 只要 `m / 2` 个服务器 或 `m * n / 2` 个或更多磁盘在线, 具有 m个服务器 和 n个磁盘 的分布式 MinIO 设置将使您的数据安全。  ( M 表示 服务器总数,   N 表示  数据库盘 总数 )


* `Erasure Code` 是用来保证 Object 的每个 Block 的数据正确和可恢复的, 而 `Bitrot` 技术是用来检验磁盘数据的正确性的。

* `Erasure Code` 技术比较复杂, 但是 `Bitrot` 技术就比较简单了。本质就是在写数据之前, 先计算好数据的 hash, 然后将 hash 先写入磁盘, 再写入需要存储的数据。这样读取数据时，就可以通过重新计算 hash，和原始写入的 hash进行一致性比较，来判断数据是否有损坏。

* 上传文件时，Minio 不是直接上传到 object 在磁盘的最终存储目录的，而是先写到一个临时目录，等所有数据都写到临时目录后，Minio 才会进行 rename 操作，将 object 数据存储到最终存储目录。



* Minimum System Configuration for Production

|服务器节点|节点磁盘数|Sets 纠删组磁盘数量|奇偶校验块数量|读取数据保障|写入数据保障|
|-|-|-|-|-|-|
|4|2|8|4|2|1|
|5|2|10|4|2|2|
|6|2|12|4|2|2|
|7|2|14|4|2|2|
|8|2||16|4|2|2|
|9|2|9|4|4|4|
|10|2|10|4|4|4|



## 一致性保证


**对于分布式和独立模式下的所有 I/O 操作，MinIO 遵循严格的先读后写和写后列表一致性模型。仅当您使用磁盘文件系统（例如 xfs、ext4 或 zfs 等）进行分布式设置时，才能保证此一致性模型。**





## Minio 分布式集群配置


> 分布式集群节点计算器: https://min.io/product/erasure-code-calculator


* 所有运行分布式 MinIO 的节点都应该共享一个公共根凭证, 以便节点相互连接和信任。为此，建议在执行MinIO服务器命令之前，在所有节点上配置环境变量 `MINIO_ROOT_USER` 和 `MINIO_ROOT_PASSWORD`。如果未配置, 默认的帐号密码为: `minioadmin/minioadmin` 。

* MinIO 创建每组 `4 ~ 16` 个驱动器的纠删码组。您提供的驱动器总数必须是这些数字之一的倍数。

  * 纠删码组:  Minio 会将 磁盘 ( 4 、6、8、10、12、14、16 ) 总个数 组成 纠删组 （EC 集 / Sets ) . 既 比如 36 个总磁盘 最大可组成 3 个 16 磁盘 的 EC 集  .    ( Minio 默认会划分最大的总和纠删组 )  —  具体的  纠删组 可在 磁盘目录  .minio.sys/format.json   文件中查看. 

* MinIO 自动选择选择最大的 EC 集大小, 它分为给定的驱动器总数或节点总数 - 确保保持均匀分布, 即每个节点参与每组相同数量的驱动器。

* 每个对象都写入单个 EC 集, 因此分布在不超过 16 个驱动器上。

* 建议所有运行分布式 MinIO 设置的节点是相同的, 即相同的操作系统、相同的磁盘数量和相同的网络互连。

* MinIO 分布式模式需要新目录。如果需要，驱动器可以与其他应用程序共享。您可以通过使用 MinIO 专有的子目录来完成此操作。例如，如果您在 下安装了您的卷/export，/export/data 则将其作为参数传递给 MinIO 服务器。

* 运行分布式 MinIO 实例的服务器之间的间隔应小于 15 分钟。您可以启用NTP服务作为最佳实践，以确保跨服务器的时间相同。



## 分布式集群节点数量限制


* Minio 官方建议一个集群最少 4 个节点，最多 32 个节点, 实质上是没有最多节点限制，但官方建议 32 个节点集群的原因有以下: 

  * 32 个节点 是一个很好的平衡故障域和规模的代理，使用超过 32 个节点违背了运行容器化工作负载的基本前提：应将应用程序（及其相关联的存储）分解成尽可能小的部分
一个 32 个节点 的集群的最大存储容量为 200 PB，你可以在 32 节点集群存储大量的数据.

  * 在单个集群上管理超过 200 PB 所带来的复杂性非常大，当你有一个非常大的故障域时，如果少量服务器宕机，则整个基础架构都将崩溃，在管理大量节点的情况下，故障发生只是时间问题，而非是否发生的问题.

  * 比较少的节点配置会增加可管理性，以及分析故障发生的根本原因更加容易.

  * 对于大量节点集群来说，备份和迁移数据的时间需要很多.

  * 云原生架构的基本前提，无论是计算还是存储，都是以小的、离散的块消耗资源；扩容时，应该添加更多实例，并将其尽可能密集的打包在基础架构中，而不是无限的扩展一个实例

  * 即使每个集群只有32个节点，也可以将这些群集联合在一起，使它们可以作为一个巨型集群运行（并扩展到无穷大），同时仍保持体系结构的可管理性.



## Minio 存储类


**Minio分为两种存储类**




### 标准存储类


**标准存储类, 通过环境变量  MINIO_STORAGE_CLASS_STANDARD="EC:5" 配置比率**

* `"EC:5"` 表示允许 标准存储类 5个磁盘 故障损坏 不丢失数据.


---


* 标准存储类是部署 Minio 的默认存储类存储类。 一旦设置，默认情况下所有 PutObject 请求将遵循标准存储类下设置的数据/奇偶校验配置。

  * 默认情况下，标准存储类数据和奇偶校验驱动器设置为 N/2（并且不能设置高于此值）。 N 为 磁盘总数量。

  * 您可以选择将对象元数据集设置为为相应对象 `X-Amz-Storage-Class:STANDARD` 启用 STANDARD 存储类。

  * Minio 服务器不会在元数据字段中返回存储类，如果类设置为STANDARD. 这符合 AWS S3 PutObject 行为。




### 低冗余存储类


**低冗余存储类, 通过环境变量  MINIO_STORAGE_CLASS_STANDARD="EC:2"  配置比率**


* `"EC:2"` 表示允许 冗余存储类 2个磁盘 故障损坏 不丢失数据.


---

* 低冗余存储类 可以应用于不太重要的对象，需要较少的复制。要应用此类，需要在 `X-Amz-Storage-Class:REDUCED_REDUNDANCY` 在 PutObject（或多部分）请求中设置对象元数据。这表明 MinIO 服务器存储相应的对象，其数据和奇偶校验由低冗余类定义。

  * 默认情况下，低冗余存储类奇偶校验驱动器设置为 2（并且不能设置为低于此值）。

  * 您需要设置 X-Amz-Storage-Class:REDUCED_REDUNDANCY 对象元数据，以便服务器端低冗余来启动。

  * 如果类设置为低冗余，MinIO 服务器将在元数据字段中返回存储类。






## 分布式 Minio 扩容


> 分布式 Minio 需要扩容需要满足 `http://{...}/data{...}`  的语法参数。



```
# 创建集群
export MINIO_ACCESS_KEY=<ACCESS_KEY>
export MINIO_SECRET_KEY=<SECRET_KEY>
minio server http://host{1...4}/export{1...16}


#扩容集群
export MINIO_ACCESS_KEY=<ACCESS_KEY>
export MINIO_SECRET_KEY=<SECRET_KEY>
minio server http://host{1...4}/export{1...16} http://host{5...12}/export{1...16}
```


---

* 在扩容集群时用到了 2组 `http://{...}`  的语法参数，所以旧集群以及扩容都必须使用这种语法形式创建， 命令行中的每组 server 称为一个 zone, 此示例中有 2 server pools，对于外部的写数据请求，Minio Server 会首先查找可用空间多的 Zone，然后再在 Zone 内选择 set 和 disk drive 。


  * 注意:  添加的每个 Zone 必须具有与 原始 Zone 相同的 `Set集大小` , 以便维持相同的数据冗余 SLA。 例如, 如果您的第一个 Zone 是8个磁盘, 则可以添加每个 Server pools 包含 16、32或1024 个磁盘的 Zone。您只需确保部署SLA是原始数据冗余SLA的倍数。






## Minio 安装部署


> Minio 分为 物理机部署, docker 部署, Kubernetes Operator 部署.


**以下为 物理机部署方式**




### 初始化 Minio 节点



> 内核优化


```bash
#!/bin/bash

cat > sysctl.conf <<EOF

# maximum number of open files/file descriptors
fs.file-max = 4194303

# use as little swap space as possible
vm.swappiness = 1

# prioritize application RAM against disk/swap cache
vm.vfs_cache_pressure = 50

# minimum free memory
vm.min_free_kbytes = 1000000

# follow mellanox best practices https://community.mellanox.com/s/article/linux-sysctl-tuning
# the following changes are recommended for improving IPv4 traffic performance by Mellanox

# disable the TCP timestamps option for better CPU utilization
net.ipv4.tcp_timestamps = 0

# enable the TCP selective acks option for better throughput
net.ipv4.tcp_sack = 1

# increase the maximum length of processor input queues
net.core.netdev_max_backlog = 250000

# increase the TCP maximum and default buffer sizes using setsockopt()
net.core.rmem_max = 4194304
net.core.wmem_max = 4194304
net.core.rmem_default = 4194304
net.core.wmem_default = 4194304
net.core.optmem_max = 4194304

# increase memory thresholds to prevent packet dropping:
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 65536 4194304

# enable low latency mode for TCP:
net.ipv4.tcp_low_latency = 1

# the following variable is used to tell the kernel how much of the socket buffer
# space should be used for TCP window size, and how much to save for an application
# buffer. A value of 1 means the socket buffer will be divided evenly between.
# TCP windows size and application.
net.ipv4.tcp_adv_win_scale = 1

# maximum number of incoming connections
net.core.somaxconn = 65535

# maximum number of packets queued
net.core.netdev_max_backlog = 10000

# queue length of completely established sockets waiting for accept
net.ipv4.tcp_max_syn_backlog = 4096

# time to wait (seconds) for FIN packet
net.ipv4.tcp_fin_timeout = 15

# disable icmp send redirects
net.ipv4.conf.all.send_redirects = 0

# disable icmp accept redirect
net.ipv4.conf.all.accept_redirects = 0

# drop packets with LSR or SSR
net.ipv4.conf.all.accept_source_route = 0

# MTU discovery, only enable when ICMP blackhole detected
net.ipv4.tcp_mtu_probing = 1

EOF

sysctl -p

# `Transparent Hugepage Support`*: This is a Linux kernel feature intended to improve
# performance by making more efficient use of processor’s memory-mapping hardware.
# But this may cause https://blogs.oracle.com/linux/performance-issues-with-transparent-huge-pages-thp
# for non-optimized applications. As most Linux distributions set it to `enabled=always` by default,
# we recommend changing this to `enabled=madvise`. This will allow applications optimized
# for transparent hugepages to obtain the performance benefits, while preventing the
# associated problems otherwise. Also, set `transparent_hugepage=madvise` on your kernel
# command line (e.g. in /etc/default/grub) to persistently set this value.

echo "Enabling THP madvise"
echo madvise | sudo tee /sys/kernel/mm/transparent_hugepage/enabled

```


---


> host 配置


* 配置 host 连接 已便用于 `http://host`  的方式配置 server  所有  minio 节点均配置


```bash
vi /etc/hosts

## Minio Server Host
10.3.2.104 minio-1
10.3.2.105 minio-2
10.3.2.106 minio-3
10.3.2.107 minio-4
## Minio Server Host
```



> 创建 Minio 用户

```bash
useradd minio -m
```




> 创建 磁盘目录


```bash
mkdir /export{1..9}

chown -R minio:minio /export{1..9}
```



> 配置挂载

```bash
# 查看磁盘 uuid, 使用 uuid 挂载, 可避免 磁盘名称变化而出问题
blkid 


# 配置自动挂载

vi /etc/fstab            


# minio volume
UUID=0db41c13-d45c-4162-95b2-37e28b6d2f33 /export1 xfs defaults 0 0 
UUID=fdfcfe36-4a4a-46ff-af98-229692a9e585 /export2 xfs defaults 0 0

```




### 安装配置 Minio


> 下载 Minio 二进制文件

```bash
# 创建 minio 文件目录

mkdir -p /opt/minio/{bin,env}


# 授权 minio home 目录

chown -R minio:minio /opt/minio


# 下载 minio 二进制文件

cd /opt/minio/bin/

curl -O http://dl.minio.org.cn/server/minio/release/linux-amd64/minio

chmod +x  minio
```



> 创建 minio env 配置文件


```bash
cd /opt/minio/env


# 创建 变量 配置文件

vi minio.env


# Volume to be used for MinIO server.
MINIO_VOLUMES="http://minio{1...4}/export{1...2}"
# Use if you want to run MinIO on a custom port.
MINIO_OPTS="--address :9000"
# Access Key of the server.
MINIO_ACCESS_KEY="minio"
# Secret key of the server.
MINIO_SECRET_KEY="miniodev"
# Prometheus auth
MINIO_PROMETHEUS_AUTH_TYPE="public"
# 标准存储类 DataDrives 与 ParityDrives. 
MINIO_STORAGE_CLASS_STANDARD="EC:4"
# 低冗余存储类 DataDrives 与 ParityDrives. 
MINIO_STORAGE_CLASS_RRS="EC:2"
```



> 创建 systemctl 文件


```bash
#  创建 service 文件

vi /etc/systemd/system/minio.service




[Unit]
Description=MinIO
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/opt/minio/bin/minio

[Service]
WorkingDirectory=/opt/minio/

User=minio
Group=minio

EnvironmentFile=/opt/minio/env/minio.env
ExecStartPre=/bin/bash -c "if [ -z \"${MINIO_VOLUMES}\" ]; then echo \"Variable MINIO_VOLUMES not set in /opt/minio/env/minio.env\"; exit 1; fi"

ExecStart=/opt/minio/bin/minio server $MINIO_OPTS $MINIO_VOLUMES

# Let systemd restart this service always
Restart=always

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65536

# Specifies the maximum number of threads this process can create
TasksMax=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target

# Built for ${project.name}-${project.version} (${project.name})
```



```bash
# 授权 service 文件
chmod +x /etc/systemd/system/minio.service
```



> 启动 Minio 服务

* Server 节点间启动的时间间隔不要超过 15 分钟


```bash

#  启动服务

systemctl enable minio.service
systemctl start minio



# 查看服务

systemctl status minio


● minio.service - MinIO
   Loaded: loaded (/etc/systemd/system/minio.service; enabled; vendor preset: enabled)
   Active: active (running) since Wed 2021-06-02 08:14:06 UTC; 8min ago
     Docs: https://docs.min.io
  Process: 10344 ExecStartPre=/bin/bash -c if [ -z "${MINIO_VOLUMES}" ]; then echo "Variable MINIO_VOLUMES not set in /opt/minio/env/minio.env"; exit 1; fi (code=exited, status=0/SUCCESS)
 Main PID: 10345 (minio)
    Tasks: 46
   CGroup: /system.slice/minio.service
           └─10345 /opt/minio/bin/minio server --address :9000 http://minio-{1...4}/export{1...2}

Jun 02 08:15:00 minio-1 minio[10345]: Browser Access:
Jun 02 08:15:00 minio-1 minio[10345]:    http://10.3.2.104:9000  http://127.0.0.1:9000
Jun 02 08:15:00 minio-1 minio[10345]: Object API (Amazon S3 compatible):
Jun 02 08:15:00 minio-1 minio[10345]:    Go:         https://docs.min.io/docs/golang-client-quickstart-guide
Jun 02 08:15:00 minio-1 minio[10345]:    Java:       https://docs.min.io/docs/java-client-quickstart-guide
Jun 02 08:15:00 minio-1 minio[10345]:    Python:     https://docs.min.io/docs/python-client-quickstart-guide
Jun 02 08:15:00 minio-1 minio[10345]:    JavaScript: https://docs.min.io/docs/javascript-client-quickstart-guide
Jun 02 08:15:00 minio-1 minio[10345]:    .NET:       https://docs.min.io/docs/dotnet-client-quickstart-guide
Jun 02 08:15:00 minio-1 minio[10345]: Waiting for all MinIO IAM sub-system to be initialized.. lock acquired
Jun 02 08:15:00 minio-1 minio[10345]: IAM initialization complete


```



---


### 配置 minio mc 客户端


> 下载 mc 客户端


```bash
cd /opt/minio/bin/

curl -O http://dl.minio.org.cn/client/mc/release/linux-amd64/mc

chmod +x mc
```



> 创建 mc 客户端配置文件  (  mc 会读取  ~/.mc/config.json  文件 )


```bash
vi ~/.mc/config.json


{
	"version": "10",
	"aliases": {
		"minio": {
			"url": "http://s3.jicki.cn",
			"accessKey": "minio",
			"secretKey": "miniodev",
			"api": "S3v4",
			"path": "auto"
		}
	}
}
```



### 安装配置 Minio Console 管理 WebUI



> 下载 Console 二进制文件

```bash
cd /opt/minio/bin/


curl -o minio-console https://github.com/minio/console/releases/latest/download/console-linux-amd64


chmod +x minio-console

chown minio:minio minio-console
```




> 创建 minio-console  env 文件


```bash

vi /opt/minio/env/minio-console.env


# Salt to encrypt JWT payload
CONSOLE_PBKDF_PASSPHRASE="SECRET"
# Required to encrypt JWT payload
CONSOLE_PBKDF_SALT="SECRET"
# MinIO Endpoint
CONSOLE_MINIO_SERVER="http://s3.jicki.cn"
```




> 创建 systemctl 文件


```bash
#  创建 service 文件

vi /etc/systemd/system/minio-console.service



[Unit]
Description=MinIO-ConSole
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/opt/minio/bin/minio-console

[Service]
WorkingDirectory=/opt/minio/

User=minio
Group=minio

EnvironmentFile=/opt/minio/env/minio-console.env
ExecStartPre=/bin/bash -c "if [ -z \"${CONSOLE_MINIO_SERVER}\" ]; then echo \"Variable CONSOLE_MINIO_SERVER not set in /opt/minio/env/minio-console.env\"; exit 1; fi"

ExecStart=/opt/minio/bin/minio-console server

# Let systemd restart this service always
Restart=always

# Specifies the maximum file descriptor number that can be opened by this process
LimitNOFILE=65536

# Specifies the maximum number of threads this process can create
TasksMax=infinity

# Disable timeout logic and wait until process is stopped
TimeoutStopSec=infinity
SendSIGKILL=no

[Install]
WantedBy=multi-user.target

```



```bash
# 授权 service 文件
chmod +x /etc/systemd/system/minio-console.service
```



> 启动 minio console 服务


```bash
#  启动服务

systemctl enable minio-console.service
systemctl start minio-console
systemctl status minio-console
```




>  创建 minio console 登录用户


```bash
# 创建一个 admin 权限的用户  # minio 为 mc 配置文件中的 aliases 下配置的名称

mc admin user add minio/


Enter Access Key: console
Enter Secret Key: 
Added user `console` successfully.
```



```bash
#  配置 console 用户的权限 配置文件

cat > admin.json << EOF
{
	"Version": "2012-10-17",
	"Statement": [{
			"Action": [
				"admin:*"
			],
			"Effect": "Allow",
			"Sid": ""
		},
		{
			"Action": [
                "s3:*"
			],
			"Effect": "Allow",
			"Resource": [
				"arn:aws:s3:::*"
			],
			"Sid": ""
		}
	]
}
EOF
```



```bash
# 创建 权限 policy 信息 consoleAdmin

mc admin policy add minio consoleAdmin admin.json


Added policy `consoleAdmin` successfully.
```




```bash
# 用户配置 权限

mc admin policy set minio consoleAdmin user=console


Policy `consoleAdmin` is set on user `console`
```



### 配置 Prometheus


**监控路径 path: /minio/v2/metrics/cluster**

* 访问授权需要配置 Token 或者 配置 `MINIO_PROMETHEUS_AUTH_TYPE="public"`


> prometheus targets 添加


```bash
scrape_configs:
- job_name: minio
  metrics_path: /minio/v2/metrics/cluster
  scheme: http
  static_configs:
  - targets: 
    - '10.3.2.104:9000'
    - '10.3.2.105:9000'
    - '10.3.2.106:9000'
    - '10.3.2.107:9000'
```





## Minio 压力测试


* 压力测试使用官方提供工具  `https://github.com/minio/warp`



---


>  Minio Server = 4  Data = 4  Parity =  4  EC 比率为 4:4


```bash
mc admin config get minio storage_class

storage_class standard=EC:4 rrs=EC:2 dma=read+write

```



* Uploading 2500 objects of Random data; 10485760 bytes total 

  * 集群测试


```bash
warp mixed --host=10.3.2.10{4...7}:9000 --access-key=minio --secret-key=miniodev --autoterm --analyze.v
```


* 输出结果


```
Mixed operations.
Operation: DELETE - total: 1418, 10.0%, Concurrency: 20, Duration: 4m2s, starting 2021-06-04 06:26:20.534 +0000 UTC

Throughput by host:
 * http://10.3.2.105:9000: Avg: 1.61 obj/s.
 * http://10.3.2.104:9000: Avg: 1.25 obj/s.
 * http://10.3.2.107:9000: Avg: 1.49 obj/s.
 * http://10.3.2.106:9000: Avg: 1.56 obj/s.

Requests considered: 1419:
 * Avg: 40ms, 50%: 11ms, 90%: 23ms, 99%: 643ms, Fastest: 9ms, Slowest: 1.101s

Requests by host:
 * http://10.3.2.104:9000 - 303 requests: 
	- Avg: 44ms Fastest: 9ms Slowest: 1.101s 50%: 11ms 90%: 24ms
 * http://10.3.2.105:9000 - 390 requests: 
	- Avg: 40ms Fastest: 9ms Slowest: 886ms 50%: 11ms 90%: 23ms
 * http://10.3.2.106:9000 - 369 requests: 
	- Avg: 40ms Fastest: 9ms Slowest: 1.013s 50%: 11ms 90%: 20ms
 * http://10.3.2.107:9000 - 359 requests: 
	- Avg: 35ms Fastest: 9ms Slowest: 730ms 50%: 11ms 90%: 22ms

Operation: GET - total: 6367, 45.0%, Concurrency: 20, Duration: 4m2s, starting 2021-06-04 06:26:20.198 +0000 UTC

Throughput by host:
 * http://10.3.2.106:9000: Avg: 64.36 MiB/s, 6.44 obj/s.
 * http://10.3.2.105:9000: Avg: 68.60 MiB/s, 6.86 obj/s.
 * http://10.3.2.107:9000: Avg: 65.98 MiB/s, 6.60 obj/s.
 * http://10.3.2.104:9000: Avg: 64.65 MiB/s, 6.47 obj/s.

Requests considered: 6368:
 * Avg: 347ms, 50%: 30ms, 90%: 1.024s, 99%: 1.46s, Fastest: 15ms, Slowest: 2.5s
 * First Byte: Avg: 100ms, Median: 7ms, Best: 4ms, Worst: 1.613s
 * First Access: Avg: 799ms, 50%: 812ms, 90%: 1.198s, 99%: 1.694s, Fastest: 17ms, Slowest: 2.5s
 * First Access TTFB: Avg: 220ms, Median: 205ms, Best: 4ms, Worst: 1.613s

Requests by host:
 * http://10.3.2.104:9000 - 1563 requests: 
	- Avg: 366ms Fastest: 15ms Slowest: 2.5s 50%: 33ms 90%: 1.035s
	- First Byte: Avg: 105ms, Median: 8ms, Best: 4ms, Worst: 1.211s
 * http://10.3.2.105:9000 - 1660 requests: 
	- Avg: 328ms Fastest: 16ms Slowest: 2.226s 50%: 29ms 90%: 1s
	- First Byte: Avg: 94ms, Median: 6ms, Best: 4ms, Worst: 1.216s
 * http://10.3.2.106:9000 - 1558 requests: 
	- Avg: 336ms Fastest: 17ms Slowest: 2.143s 50%: 25ms 90%: 1.045s
	- First Byte: Avg: 96ms, Median: 6ms, Best: 4ms, Worst: 1.613s
 * http://10.3.2.107:9000 - 1591 requests: 
	- Avg: 359ms Fastest: 16ms Slowest: 1.9s 50%: 37ms 90%: 1.011s
	- First Byte: Avg: 105ms, Median: 8ms, Best: 4ms, Worst: 1.29s

Operation: PUT - total: 2109, 14.9%, Concurrency: 20, Duration: 4m2s, starting 2021-06-04 06:26:20.148 +0000 UTC

Throughput by host:
 * http://10.3.2.107:9000: Avg: 21.48 MiB/s, 2.15 obj/s.
 * http://10.3.2.106:9000: Avg: 23.13 MiB/s, 2.31 obj/s.
 * http://10.3.2.105:9000: Avg: 22.12 MiB/s, 2.21 obj/s.
 * http://10.3.2.104:9000: Avg: 21.19 MiB/s, 2.12 obj/s.

Requests considered: 2110:
 * Avg: 1.179s, 50%: 1.12s, 90%: 1.574s, 99%: 2.339s, Fastest: 363ms, Slowest: 3.076s

Requests by host:
 * http://10.3.2.104:9000 - 510 requests: 
	- Avg: 1.182s Fastest: 451ms Slowest: 3.076s 50%: 1.117s 90%: 1.633s
 * http://10.3.2.105:9000 - 532 requests: 
	- Avg: 1.189s Fastest: 576ms Slowest: 3.045s 50%: 1.13s 90%: 1.601s
 * http://10.3.2.106:9000 - 559 requests: 
	- Avg: 1.175s Fastest: 541ms Slowest: 2.732s 50%: 1.117s 90%: 1.511s
 * http://10.3.2.107:9000 - 517 requests: 
	- Avg: 1.175s Fastest: 363ms Slowest: 2.585s 50%: 1.124s 90%: 1.576s

Operation: STAT - total: 4237, 29.9%, Concurrency: 20, Duration: 4m2s, starting 2021-06-04 06:26:20.556 +0000 UTC

Throughput by host:
 * http://10.3.2.107:9000: Avg: 4.31 obj/s.
 * http://10.3.2.104:9000: Avg: 4.39 obj/s.
 * http://10.3.2.105:9000: Avg: 4.66 obj/s.
 * http://10.3.2.106:9000: Avg: 4.20 obj/s.

Requests considered: 4238:
 * Avg: 14ms, 50%: 4ms, 90%: 7ms, 99%: 338ms, Fastest: 3ms, Slowest: 1.453s
 * First Access: Avg: 16ms, 50%: 4ms, 90%: 7ms, 99%: 437ms, Fastest: 3ms, Slowest: 1.453s


Requests by host:
 * http://10.3.2.104:9000 - 1055 requests: 
	- Avg: 17ms Fastest: 3ms Slowest: 1.453s 50%: 4ms 90%: 7ms
 * http://10.3.2.105:9000 - 1126 requests: 
	- Avg: 12ms Fastest: 3ms Slowest: 974ms 50%: 4ms 90%: 6ms
 * http://10.3.2.106:9000 - 1018 requests: 
	- Avg: 14ms Fastest: 3ms Slowest: 1.014s 50%: 4ms 90%: 7ms
 * http://10.3.2.107:9000 - 1041 requests: 
	- Avg: 14ms Fastest: 3ms Slowest: 931ms 50%: 4ms 90%: 6ms

Cluster Total: 349.67 MiB/s, 58.28 obj/s over 4m3s.
 * http://10.3.2.107:9000: 87.36 MiB/s, 14.54 obj/s
 * http://10.3.2.104:9000: 85.44 MiB/s, 14.14 obj/s
 * http://10.3.2.105:9000: 90.40 MiB/s, 15.29 obj/s
 * http://10.3.2.106:9000: 87.20 MiB/s, 14.44 obj/s

```

---


* Minio 节点 负载采集  `top -d 5  -u minio -b`


* minio-1 - 取最后五次


```bash
top - 06:30:01 up 3 days, 20:15,  1 user,  load average: 15.85, 12.92, 9.40
Tasks: 805 total,   1 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.2 us,  0.6 sy,  0.0 ni, 87.8 id, 10.4 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392556+total, 24926662+free,  2644988 used, 12013964 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25770344+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18005 minio     20   0 1393320 431032  35964 S 112.1  0.2 100:54.48 minio



top - 06:30:06 up 3 days, 20:15,  1 user,  load average: 15.95, 12.99, 9.44
Tasks: 805 total,   1 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.3 us,  0.6 sy,  0.0 ni, 85.2 id, 12.8 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392556+total, 24911628+free,  2639184 used, 12170092 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25770809+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18005 minio     20   0 1393320 428080  35964 S 126.2  0.2 101:00.84 minio



top - 06:30:12 up 3 days, 20:15,  1 user,  load average: 15.39, 12.93, 9.44
Tasks: 805 total,   1 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.9 us,  0.5 sy,  0.0 ni, 89.4 id,  9.1 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392556+total, 24897552+free,  2646352 used, 12303696 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25769963+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18005 minio     20   0 1393320 438984  35964 S  93.7  0.2 101:05.56 minio



top - 06:30:17 up 3 days, 20:15,  1 user,  load average: 15.44, 12.98, 9.47
Tasks: 805 total,   1 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.1 us,  0.5 sy,  0.0 ni, 87.2 id, 11.1 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392556+total, 24881540+free,  2652652 used, 12457512 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25769230+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18005 minio     20   0 1393320 438984  35964 S 104.6  0.2 101:10.82 minio



top - 06:30:22 up 3 days, 20:15,  1 user,  load average: 14.52, 12.83, 9.44
Tasks: 805 total,   1 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.1 us,  0.5 sy,  0.0 ni, 89.0 id,  9.2 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392556+total, 24868499+free,  2651148 used, 12589424 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25769321+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18005 minio     20   0 1393320 441428  35964 S 107.7  0.2 101:16.25 minio

```


---

* minio-2 - 取最后五次


```bash
top - 06:30:02 up 3 days, 20:24,  1 user,  load average: 14.55, 12.20, 8.60
Tasks: 807 total,   1 running, 341 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.4 us,  0.6 sy,  0.0 ni, 85.6 id, 12.2 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem : 26392688+total, 21531648+free,  2711936 used, 45898468 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25721062+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18969 minio     20   0 1392168 438676  35260 S 134.2  0.2  70:49.42 minio
17293 minio     20   0  904264  63908  35560 S   0.0  0.0   0:46.15 minio-console



top - 06:30:07 up 3 days, 20:24,  1 user,  load average: 15.47, 12.43, 8.70
Tasks: 807 total,   1 running, 341 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.4 us,  0.6 sy,  0.0 ni, 84.9 id, 12.9 wa,  0.0 hi,  0.2 si,  0.0 st
KiB Mem : 26392688+total, 21516976+free,  2703348 used, 46053772 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25721790+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18969 minio     20   0 1392168 434516  35260 S 132.9  0.2  70:56.12 minio
17293 minio     20   0  904264  63908  35560 S   0.0  0.0   0:46.15 minio-console



top - 06:30:12 up 3 days, 20:24,  1 user,  load average: 15.03, 12.39, 8.70
Tasks: 807 total,   1 running, 341 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.1 us,  0.4 sy,  0.0 ni, 90.2 id,  8.2 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392688+total, 21503340+free,  2716244 used, 46177220 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25720403+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18969 minio     20   0 1392168 447076  35260 S  94.2  0.2  71:00.86 minio
17293 minio     20   0  904264  63908  35560 S   0.0  0.0   0:46.15 minio-console



top - 06:30:17 up 3 days, 20:24,  1 user,  load average: 15.27, 12.49, 8.75
Tasks: 805 total,   1 running, 341 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.3 us,  0.6 sy,  0.0 ni, 87.3 id, 10.7 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392688+total, 21486862+free,  2713744 used, 46344508 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25720536+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18969 minio     20   0 1392168 447076  35260 S 117.3  0.2  71:06.77 minio
17293 minio     20   0  904264  63908  35560 S   1.0  0.0   0:46.20 minio-console



top - 06:30:22 up 3 days, 20:24,  1 user,  load average: 15.25, 12.53, 8.79
Tasks: 806 total,   1 running, 341 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.1 us,  0.5 sy,  0.0 ni, 91.0 id,  7.3 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392688+total, 21472577+free,  2730024 used, 46471080 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25718838+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18969 minio     20   0 1392168 458988  35260 S 101.8  0.2  71:11.90 minio
17293 minio     20   0  904264  63908  35560 S   0.0  0.0   0:46.20 minio-console

```


---

* minio-3 - 取最后五次


```bash
top - 06:30:03 up 3 days, 20:23,  1 user,  load average: 15.41, 13.39, 9.42
Tasks: 797 total,   1 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.3 us,  0.6 sy,  0.0 ni, 87.2 id, 10.7 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392558+total, 24932112+free,  2597284 used, 12007168 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25775200+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
19902 minio     20   0 1391208 426812  35260 S 129.0  0.2  70:06.34 minio



top - 06:30:08 up 3 days, 20:23,  1 user,  load average: 16.26, 13.60, 9.51
Tasks: 797 total,   1 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.2 us,  0.7 sy,  0.0 ni, 85.2 id, 12.7 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392558+total, 24918041+free,  2585608 used, 12159552 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25776248+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
19902 minio     20   0 1391208 419832  35260 S 126.8  0.2  70:12.72 minio



top - 06:30:13 up 3 days, 20:24,  1 user,  load average: 14.96, 13.38, 9.46
Tasks: 798 total,   1 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.0 us,  0.4 sy,  0.0 ni, 87.8 id, 10.7 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392558+total, 24904404+free,  2594100 used, 12287432 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25775200+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
19902 minio     20   0 1391208 418444  35260 S  92.7  0.2  70:17.39 minio



top - 06:30:18 up 3 days, 20:24,  1 user,  load average: 14.24, 13.26, 9.44
Tasks: 798 total,   1 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.3 us,  0.5 sy,  0.0 ni, 86.1 id, 12.0 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392558+total, 24888230+free,  2592204 used, 12451064 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25775408+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
19902 minio     20   0 1391208 417608  35324 S 124.4  0.2  70:23.66 minio



top - 06:30:23 up 3 days, 20:24,  1 user,  load average: 13.98, 13.22, 9.45
Tasks: 798 total,   1 running, 340 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.1 us,  0.5 sy,  0.0 ni, 88.2 id, 10.0 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392558+total, 24874072+free,  2611424 used, 12573432 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25773409+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
19902 minio     20   0 1391208 426540  35324 S 105.0  0.2  70:28.94 minio

```

---

minio-4 - 取最后五次


```bash
top - 06:30:03 up 3 days, 20:33,  2 users,  load average: 13.69, 12.33, 8.86
Tasks: 811 total,   1 running, 346 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.0 us,  0.5 sy,  0.0 ni, 88.0 id, 10.4 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392697+total, 24979328+free,  2678672 used, 11455032 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25771513+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18464 minio     20   0 1459244 503752  35004 S  96.6  0.2  70:23.37 minio



top - 06:30:08 up 3 days, 20:33,  2 users,  load average: 13.24, 12.25, 8.86
Tasks: 809 total,   1 running, 346 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.1 us,  0.5 sy,  0.0 ni, 87.1 id, 11.2 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392697+total, 24964654+free,  2677376 used, 11603064 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25771518+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18464 minio     20   0 1459244 503752  35004 S 102.0  0.2  70:28.51 minio



top - 06:30:13 up 3 days, 20:33,  2 users,  load average: 13.86, 12.40, 8.92
Tasks: 810 total,   1 running, 346 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.1 us,  0.5 sy,  0.0 ni, 90.0 id,  8.3 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392697+total, 24951321+free,  2674656 used, 11739100 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25771692+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18464 minio     20   0 1459244 497056  35004 S 102.6  0.2  70:33.68 minio



top - 06:30:18 up 3 days, 20:33,  2 users,  load average: 14.11, 12.47, 8.97
Tasks: 810 total,   1 running, 346 sleeping,   0 stopped,   0 zombie
%Cpu(s):  1.1 us,  0.6 sy,  0.0 ni, 89.3 id,  8.9 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392697+total, 24934313+free,  2675584 used, 11908264 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25771502+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18464 minio     20   0 1459244 494048  35004 S 109.1  0.2  70:39.17 minio



top - 06:30:23 up 3 days, 20:33,  2 users,  load average: 14.90, 12.67, 9.05
Tasks: 810 total,   1 running, 346 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.9 us,  0.5 sy,  0.0 ni, 90.0 id,  8.5 wa,  0.0 hi,  0.1 si,  0.0 st
KiB Mem : 26392697+total, 24920787+free,  2696904 used, 12022196 buff/cache
KiB Swap:  8388604 total,  8388604 free,        0 used. 25769259+avail Mem 

  PID USER      PR  NI    VIRT    RES    SHR S  %CPU %MEM     TIME+ COMMAND
18464 minio     20   0 1459244 516168  35004 S  91.7  0.2  70:43.79 minio

```



### 文件恢复测试


* 当前 minio 配置为 4 : 4 的冗余  所以删除  4  个磁盘中 的文件  (  直接删除  data 磁盘中 的文件 )  

  * 修复 30GiB 的文件 大概 2分多钟的时间


```bash
time mc admin heal -r minio-1/test


 ◐  test/file
    0/1 objects; 30 GiB in 2m4s
    ┌────────┬────┬─────────────────────┐
    │ Green  │ 12 │ 100.0% ████████████ │
    │ Yellow │  0 │   0.0%              │
    │ Red    │  0 │   0.0%              │
    │ Grey   │  0 │   0.0%              │
    └────────┴────┴─────────────────────┘

real	2m5.075s
user	0m0.170s
sys	0m0.153s
```



## mc 命令


* 官方 mc 操作命令  

  * 客户端常用命令:  `http://docs.minio.org.cn/docs/master/minio-client-complete-guide`

  * 管理员操作命令: `http://docs.minio.org.cn/docs/master/minio-admin-complete-guide`




> 配置 Minio 存储类

  * 注意:  使用 环境变量  `MINIO_STORAGE_CLASS_RRS` 、 `MINIO_STORAGE_CLASS_STANDARD`  设置的配比 在 `mc config get storage_class`  显示为 默认值. 



```bash
## 通过命令查看 storage class 配比 
mc admin config get minio storage_class

## 通过命令设置配置
mc admin config set minio storage_class "standard=EC:4 rrs=EC:2"

## 修改后需要重启才能生效
mc admin service restart minio

```

































  [1]: https://jicki.cn/img/posts/minio/erasure-code.jpg
  [2]: https://jicki.cn/img/posts/minio/minio-architecture.png
