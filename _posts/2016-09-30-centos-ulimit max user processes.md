---
layout: post
title: Ulimit max user processes
categories: centos
description: Ulimit max user processes
keywords: centos
catalog:    true
header-img: "img/pexels/triangular.jpeg"
---


# centos Ulimit max user processes

## 问题

最近在对tomcat 的一个 项目进行 压测， 普通用户 启动 tomcat 的时候 压力上去以后就会报 java.lang.OutOfMemoryError 的错误， 这种错误 按道理来说都是 系统 max user processes 的问题。

 

当时我登陆了服务器查看 系统 的 ulimit 

```
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 514585
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 655360
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 514585
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```


发现 并没有什么问题， 那就奇怪了， 问题一直困扰了很久，都没有找到问题。


后来我们使用了 root 用户去启动 tomcat 的时候，再进行压测，发现问题得到了解决，没有再出现 

java.lang.OutOfMemoryError 的错误。 


难道 root 用户 跟 普通 用户 ulimit 的值 不一样？


这次我们切换到 普通用户 下， 查看系统的 ulimit 发现

```
su tomcat
ulimit
```

```
core file size          (blocks, -c) 0
data seg size           (kbytes, -d) unlimited
scheduling priority             (-e) 0
file size               (blocks, -f) unlimited
pending signals                 (-i) 514585
max locked memory       (kbytes, -l) 64
max memory size         (kbytes, -m) unlimited
open files                      (-n) 655360
pipe size            (512 bytes, -p) 8
POSIX message queues     (bytes, -q) 819200
real-time priority              (-r) 0
stack size              (kbytes, -s) 8192
cpu time               (seconds, -t) unlimited
max user processes              (-u) 4096
virtual memory          (kbytes, -v) unlimited
file locks                      (-x) unlimited
```


为什么 普通用户下 max user processes 的值只有 4096 呢。那么这个值是从那里控制的呢？

 
按道理来说 ulimit 的数值都是 通过 /etc/security/limits.conf 来修改的，可是我们已经针对 /etc/security/limits.conf 做了 修改，但是为何 max user processes 的数值会不同呢？


后来我们发现 ulimit 下面 nproc 的数值 原来是通过 /etc/security/limits.d/20-nproc.conf 这里面的文件控制的。 我们查看 /etc/security/limits.d/20-nproc.conf  文件

```
cat /etc/security/limits.d/20-nproc.conf

# Default limit for number of user's processes to prevent
# accidental fork bombs.
# See rhbz #432903 for reasoning.
  
*          soft    nproc     4096
root       soft    nproc     unlimited
```