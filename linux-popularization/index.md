# Linux 基础知识



# Linux

* Linux 相关基础知识..


## Linux Network Namespace


* Network Namespace 是 `Linux` 内核的具有的功能. 它能创建多个隔离的网络空间, 它们有独自的网络栈信息。

* 使用 `ip` 命令创建、管理 网络空间.

```bash
# 查看 帮助
ip netns help

Usage:	ip netns list
	ip netns add NAME
	ip netns attach NAME PID
	ip netns set NAME NETNSID
	ip [-all] netns delete [NAME]
	ip netns identify [PID]
	ip netns pids NAME
	ip [-all] netns exec [NAME] cmd ...
	ip netns monitor
	ip netns list-id [target-nsid POSITIVE-INT] [nsid POSITIVE-INT]
NETNSID := auto | POSITIVE-INT

```

> 创建网络命名空间

* 每个 network namespace , 它会有自己独立的网卡、路由表、ARP 表、iptables 等和网络相关的资源。`ip` 命令提供了 `ip netns exec` 子命令可以在对应的 network namespace 中执行命令, 比如查看 network namespace 中有哪些网卡。更棒的是, 要执行的可以是任何命令，不只是和网络相关的（当然, 和网络无关命令执行的结果和在外部执行没有区别）


```bash
ip netns add jicki 
mount --make-shared /run/netns failed: Operation not permitted

```

* 出现此错误是由于用户权限的问题. 需要使用 `sudo` 权限


```bash
sudo ip netns add jicki

```


* `ip` 命令创建的 network namespace 会出现在 `/var/run/netns/` 目录下, 如果需要管理其他不是 `ip netns` 创建的 network namespace, 只要在这个目录下创建一个指向对应 network namespace 文件的链接就行。


```bash
ls -lt /var/run/netns/

总用量 0
-r--r--r-- 1 root root 0 10月 20 15:30 jicki

```


> 查看


```bash
ip netns list

jicki
```


> 管理

* `--rcfile` 标志以后可以修改 hostname 标识, 这样用于区分进入的 网络空间



```bash
sudo  ip netns exec jicki bash --rcfile <(echo "PS1=\"jicki> \"")

jicki> ip link
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

```


* 新创建的 网络空间 默认 lo 环回口 网络状态为 DOWN ( state DOWN mode )


```bash
# 启动 lo 网络
jicki> ip link set lo up


jicki> ip link
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00

```



```bash

jicki> ping -c 3 127.0.0.1

PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 比特，来自 127.0.0.1: icmp_seq=1 ttl=64 时间=0.061 毫秒
64 比特，来自 127.0.0.1: icmp_seq=2 ttl=64 时间=0.044 毫秒
64 比特，来自 127.0.0.1: icmp_seq=3 ttl=64 时间=0.044 毫秒

--- 127.0.0.1 ping 统计 ---
已发送 3 个包， 已接收 3 个包, 0% 包丢失, 耗时 2041 毫秒
rtt min/avg/max/mdev = 0.044/0.049/0.061/0.008 ms
```


> 删除

```bash
sudo ip netns delete jicki

```



> 双 网络空间 之间的通讯


* 默认情况下 网络空间 之间是不能互相通讯的

```bash
sudo ip netns add a1      
sudo ip netns add a2
ip netns ls

a2
a1
```



* 创建 `veth pair`


```bash
sudo ip link add a1 type veth peer name a2

```


```bash
ip link

...
112: a2@a1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether b2:18:3e:65:29:5b brd ff:ff:ff:ff:ff:ff
113: a1@a2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether d6:5d:d0:14:24:a6 brd ff:ff:ff:ff:ff:ff
```


* 关联各自的 `veth pair`

  * 这里需要注意, 这边创建 `veth` 时起的名字与 namespace 空间的名字是相同的.

```bash
sudo ip link set a1 netns a1

sudo ip link set a2 netns a2
```


```bash
sudo ip netns exec a1 ip link


1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
113: a1@if112: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether d6:5d:d0:14:24:a6 brd ff:ff:ff:ff:ff:ff link-netns a2
```


* 配置各自的 `ip` 地址


```bash
sudo ip netns exec a1 ip addr add dev a1 192.168.1.100/24

sudo ip netns exec a2 ip addr add dev a2 192.168.1.101/24

```


* 启动 `veth` 网卡


```bash
sudo ip netns exec a1 ip link set a1 up

sudo ip netns exec a2 ip link set a2 up

```

```bash
sudo ip netns exec a1 ip addr



1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
113: a1@if112: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether d6:5d:d0:14:24:a6 brd ff:ff:ff:ff:ff:ff link-netns a2
    inet 192.168.1.100/24 scope global a1
       valid_lft forever preferred_lft forever
    inet6 fe80::d45d:d0ff:fe14:24a6/64 scope link 
       valid_lft forever preferred_lft forever

```

```bash
sudo ip netns exec a2 ip addr        



1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
112: a2@if113: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    link/ether b2:18:3e:65:29:5b brd ff:ff:ff:ff:ff:ff link-netns a1
    inet 192.168.1.101/24 scope global a2
       valid_lft forever preferred_lft forever
    inet6 fe80::b018:3eff:fe65:295b/64 scope link 
       valid_lft forever preferred_lft forever
```

互相关联的两个 `veth` 需要都 启动 状态才会都变成 `state UP` , 否则状态为 `state LOWERLAYERDOWN` .



* 测试通讯


```bash
sudo ip netns exec a1 ping -c 3 192.168.1.101



PING 192.168.1.101 (192.168.1.101) 56(84) bytes of data.
64 比特，来自 192.168.1.101: icmp_seq=1 ttl=64 时间=0.063 毫秒
64 比特，来自 192.168.1.101: icmp_seq=2 ttl=64 时间=0.057 毫秒
64 比特，来自 192.168.1.101: icmp_seq=3 ttl=64 时间=0.056 毫秒

--- 192.168.1.101 ping 统计 ---
已发送 3 个包， 已接收 3 个包, 0% 包丢失, 耗时 2037 毫秒
rtt min/avg/max/mdev = 0.056/0.058/0.063/0.003 ms
```


```bash
sudo ip netns exec a2 ping -c 3 192.168.1.100



PING 192.168.1.100 (192.168.1.100) 56(84) bytes of data.
64 比特，来自 192.168.1.100: icmp_seq=1 ttl=64 时间=0.035 毫秒
64 比特，来自 192.168.1.100: icmp_seq=2 ttl=64 时间=0.052 毫秒
64 比特，来自 192.168.1.100: icmp_seq=3 ttl=64 时间=0.052 毫秒

--- 192.168.1.100 ping 统计 ---
已发送 3 个包， 已接收 3 个包, 0% 包丢失, 耗时 2052 毫秒
rtt min/avg/max/mdev = 0.035/0.046/0.052/0.008 ms
```



```bash
sudo ip netns exec a1 route -n   


内核 IP 路由表
目标            网关            子网掩码        标志  跃点   引用  使用 接口
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 a1
```


```bash
sudo ip netns exec a2 route -n


内核 IP 路由表
目标            网关            子网掩码        标志  跃点   引用  使用 接口
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 a2

```



> 多 网络空间 之间的通讯


* 创建一个 叫 `wq` 的网桥

```bash
sudo ip link add wq type bridge

```

```bash
sudo ip link


120: wq: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 06:1e:3e:b0:17:3a brd ff:ff:ff:ff:ff:ff
```


* 创建 a1 到 `wq` 网桥之间通许的 虚拟网卡


```bash
sudo ip link add wq2a1 type veth peer name a12wq

```

```bash
sudo ip link


121: a12wq@wq2a1: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether b6:b4:85:d8:b0:f9 brd ff:ff:ff:ff:ff:ff
122: wq2a1@a12wq: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 1a:2e:f1:e5:5c:44 brd ff:ff:ff:ff:ff:ff
```


* 创建 a2 到 `wq` 网桥之间通许的 虚拟网卡

```bash
sudo ip link add wq2a2 type veth peer name a22wq

```


```bash
sudo ip link


123: a22wq@wq2a2: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 16:83:79:6d:79:45 brd ff:ff:ff:ff:ff:ff
124: wq2a2@a22wq: <BROADCAST,MULTICAST,M-DOWN> mtu 1500 qdisc noop state DOWN mode DEFAULT group default qlen 1000
    link/ether 46:4a:1b:cc:67:7c brd ff:ff:ff:ff:ff:ff
```


* 分别 关联 创建的虚拟网卡


```bash
sudo ip link set a12wq netns a1

```


```bash
sudo ip netns exec a1 ip addr


1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
121: a12wq@if122: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether b6:b4:85:d8:b0:f9 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```



```bash
sudo ip link set a22wq netns a2

```


```bash
sudo ip netns exec a2 ip addr



1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
123: a22wq@if124: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 16:83:79:6d:79:45 brd ff:ff:ff:ff:ff:ff link-netnsid 0
```


* 关联 网桥


```bash
sudo ip link set wq2a1 master wq

sudo ip link set wq2a2 master wq

```


```bash
sudo bridge link


122: wq2a1@if121: <BROADCAST,MULTICAST> mtu 1500 master wq state disabled priority 32 cost 2 
124: wq2a2@if123: <BROADCAST,MULTICAST> mtu 1500 master wq state disabled priority 32 cost 2
```




* 配置 虚拟网卡 的IP


```bash
sudo ip netns exec a1 ip addr add dev a12wq 192.168.1.100/24

sudo ip netns exec a2 ip addr add dev a22wq 192.168.1.101/24
```


```bash
sudo ip netns exec a1 ip addr


1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
121: a12wq@if122: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether b6:b4:85:d8:b0:f9 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.1.100/24 scope global a12wq
       valid_lft forever preferred_lft forever
```


```bash
sudo ip netns exec a2 ip addr


1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
123: a22wq@if124: <BROADCAST,MULTICAST> mtu 1500 qdisc noop state DOWN group default qlen 1000
    link/ether 16:83:79:6d:79:45 brd ff:ff:ff:ff:ff:ff link-netnsid 0
    inet 192.168.1.101/24 scope global a22wq
       valid_lft forever preferred_lft forever
```


* 启动 网桥 与 虚拟网卡


```bash
sudo ip link set wq up

```


```bash
sudo ip link set wq2a1 up

sudo ip link set wq2a2 up

```



```bash
sudo ip netns exec a1 ip link set a12wq up

sudo ip netns exec a2 ip link set a22wq up
```


```bash
sudo bridge link

122: wq2a1@if121: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master wq state forwarding priority 32 cost 2 
124: wq2a2@if123: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 master wq state forwarding priority 32 cost 2
```



```bash
sudo ip link


120: wq: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP mode DEFAULT group default qlen 1000
    link/ether 1a:2e:f1:e5:5c:44 brd ff:ff:ff:ff:ff:ff
122: wq2a1@if121: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master wq state UP mode DEFAULT group default qlen 1000
    link/ether 1a:2e:f1:e5:5c:44 brd ff:ff:ff:ff:ff:ff link-netns a1
124: wq2a2@if123: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master wq state UP mode DEFAULT group default qlen 1000
    link/ether 46:4a:1b:cc:67:7c brd ff:ff:ff:ff:ff:ff link-netns a2
```


* 测试 通讯

```bash
sudo ip netns exec a1 ping -c 3 192.168.1.101


PING 192.168.1.101 (192.168.1.101) 56(84) bytes of data.
64 比特，来自 192.168.1.101: icmp_seq=1 ttl=64 时间=0.084 毫秒
64 比特，来自 192.168.1.101: icmp_seq=2 ttl=64 时间=0.092 毫秒
64 比特，来自 192.168.1.101: icmp_seq=3 ttl=64 时间=0.097 毫秒

--- 192.168.1.101 ping 统计 ---
已发送 3 个包， 已接收 3 个包, 0% 包丢失, 耗时 2049 毫秒
rtt min/avg/max/mdev = 0.084/0.091/0.097/0.005 ms
```

```bash
sudo ip netns exec a2 ping -c 3 192.168.1.100



PING 192.168.1.100 (192.168.1.100) 56(84) bytes of data.
64 比特，来自 192.168.1.100: icmp_seq=1 ttl=64 时间=0.078 毫秒
64 比特，来自 192.168.1.100: icmp_seq=2 ttl=64 时间=0.091 毫秒
64 比特，来自 192.168.1.100: icmp_seq=3 ttl=64 时间=0.036 毫秒

--- 192.168.1.100 ping 统计 ---
已发送 3 个包， 已接收 3 个包, 0% 包丢失, 耗时 2057 毫秒
rtt min/avg/max/mdev = 0.036/0.068/0.091/0.023 ms

```

```bash
sudo ip netns exec a1 route -n


内核 IP 路由表
目标            网关            子网掩码        标志  跃点   引用  使用 接口
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 a12wq
```

```bash
sudo ip netns exec a2 route -n


内核 IP 路由表
目标            网关            子网掩码        标志  跃点   引用  使用 接口
192.168.1.0     0.0.0.0         255.255.255.0   U     0      0        0 a22wq
```

