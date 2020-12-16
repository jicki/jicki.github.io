# kubernetes 的挂载传播 mountPropagation


# kubernetes 的挂载传播 mountPropagation



* 说明

  * kubernetes 的 `mountPropagation` 翻译成中文就是挂载传播。挂载传播提供了共享卷挂载的能力, 它允许在同一个 Pod, 甚至同一个节点内, 在多个容器之间共享卷的挂载。 说白了就是 在 容器或 `host` 内的挂载目录中 再  `mount` 了一个别的挂载。

  * kubernetes 中 卷的挂载传播由 `Container.volumeMounts` 的 `mountPropagation` 字段控制。它的值包含如下三种

    * `None:` - 这种卷挂载将不会收到任何后续由 `host` 创建的在这个卷上或其子目录上的挂载。同样的, 由容器创建的挂载在 `host` 上也是不可见的。这是默认的模式。这个其实很好理解, 就是容器内和 `host` 的后续挂载完全隔离。


    * `HostToContainer:` - 这种卷挂载将会收到之后所有的由 `host` 创建在该卷上或其子目录上的挂载。换句话说, 如果 `host` 在卷挂载内挂载的任何内容, 在容器中都是可见的。同样, 如果任何具有 `Bidirectional` 的 Pod 挂载传播到该卷挂载上, 具有 `HostToContainer` 的挂载传播都可以看见。


    * `Bidirectional:` - 这种挂载机制和 `HostToContainer` 类似。此外, 任何在容器中创建的挂载都会传播到 `host`, 然后传播到使用相同卷的所有 Pod 的所有容器。注意: `Bidirectional` 挂载传播是很危险的。可能会危害到 `host` 的操作系统。因此只有特权 `securityContext: privileged: true` 容器在允许使用它。


---

* 挂载传播流程图


![图1][1]




## 测试


1. mountPropagation: None


```
apiVersion: v1
kind: Pod
metadata:
    name: mount-none
    label:
      app: mount
spec:
    containers:
    - name: main
      image: nginx:latest
      volumeMounts:
      - name: testmount
        mountPath: /home
        mountPropagation: None
    volumes:
    - name: testmount
      hostPath:
        path: /mnt/
```

在 `host` 上创建挂载, 既在 /mnt 的目录下创建另一个挂载

```
mkdir /mnt/none

# 将本地的 /var 目录挂载过来
sudo mount --bind /var /mnt/none

# 查看
ls /mnt/none

backups  cache	lib  local  lock  log  mail  opt  run  spool  tmp

```


在 `容器` 上查看, 可以看到只有创建的 `none` 目录, 但是挂载内容为空.

```
# 容器内的挂载点为 /home

ls /home/none

```



2. mountPropagation: HostToContainer


```
apiVersion: v1
kind: Pod
metadata:
    name: mount-host
    label:
      app: mount
spec:
    containers:
    - name: main
      image: nginx:latest
      volumeMounts:
      - name: testmount
        mountPath: /home
        mountPropagation: HostToContainer
    volumes:
    - name: testmount
      hostPath:
        path: /mnt/
```


在 `host` 上创建挂载, 既在 /mnt 的目录下创建另一个挂载

```
mkdir /mnt/hostto

# 将本地的 /var 目录挂载过来
sudo mount --bind /var /mnt/hostto

# 查看
ls /mnt/hostto

backups  cache	lib  local  lock  log  mail  opt  run  spool  tmp

```


在 `容器` 上查看, 可以看到只有创建的 `hostto` 目录, 里面是有内容的, 这里是 `host` 可以将挂载传递到 容器里面, 不过反过来容器是传递不到 `host` 里的。

```
# 容器内的挂载点为 /home

ls /home/hostto

backups  cache	lib  local  lock  log  mail  opt  run  spool  tmp

```


3. mountPropagation: Bidirectional


```
apiVersion: v1
kind: Pod
metadata:
    name: mount-bidir-a
    labels:
      app: mount
spec:
    containers:
    - name: main
      image: nginx:latest
      # Bidirectional 需要开启特权模式才可以使用
      securityContext:
        privileged: true
      volumeMounts:
      - name: testmount
        mountPath: /home
        mountPropagation: Bidirectional
    volumes:
    - name: testmount
      hostPath:
        path: /mnt/

---
apiVersion: v1
kind: Pod
metadata:
    name: mount-bidir-b
    labels:
      app: mount
spec:
    containers:
    - name: main
      image: nginx:latest
      # Bidirectional 需要开启特权模式才可以使用
      securityContext:
        privileged: true
      volumeMounts:
      - name: testmount
        mountPath: /home
        mountPropagation: Bidirectional
    volumes:
    - name: testmount
      hostPath:
        path: /mnt/
```


在容器 `mount-bidir-a` 中创建一个挂载

```
# exec 进入容器
kubectl exec -it mount-bidir-a sh

# 创建一个目录
mkdir /home/bidir


# 在容器中创建挂载
mount --bind /var /home/bidir

# 查看
ls /home/bidir

backups  cache	lib  local  lock  log  mail  opt  run  spool  tmp
```


在 `host` 上查看目录, 这里可以看到已经将 挂载传递到了 `host` 上

```
ls /mnt/bidir

backups  cache	lib  local  lock  log  mail  opt  run  spool  tmp

```


在容器 `mount-bidir-b` 上查看, 也已经将挂载传递到了 另一个容器 b 上


```
# exec 进入容器
kubectl exec -it mount-bidir-b sh


# 查看
ls /home/bidir

backups  cache	lib  local  lock  log  mail  opt  run  spool  tmp

```





# linux mount 的几种类型


* 上面介绍了 kubernetes 的挂载传播机制, 在 linux mount 中, 也有类似的概念。

* mount 分为下面几种:

  * `shared mount:` - 相当于上面所说的 Bidirectional 的挂载传播。

  * `slave mount:` - 每个 slave mount 都有一个 shared master mount, 挂载传播只能从 `master -> slave`, 等同于上面的 HostToContainer,  host 是 master，container 是 slave。

  * `private mount:` - 很明显, private 就是相当于 None，挂载不会向任何一方传播。

  * `unbindable mount:` - 其实就是 unbindable private mount, 也就是不允许使用 --bind 的挂载。






    [1]: https://jicki.cn/img/posts/kubernetes/mount-propagation.png
