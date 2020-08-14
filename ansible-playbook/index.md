# Ansible 从入门到放弃


{{< figure src="/img/posts/ansible/ansible.png" >}}


# 前言

**运维自动化的发展历程以及技术应用**

1. 本地部署 `On-Premises` 如下都需要自己 `部署/配置` 以及维护。

  * `Application`
  * `Data`
  * `Runtime`
  * `Middleware`
  * `OS`
  * `Virtualization`
  * `Servers`
  * `Storage`
  * `NetWorking`

2. 基础设施既服务 ( 如 阿里云 ) `IaaS` - `Infrastructure as a Service` 如下需要自己 `部署/配置` 以及维护。

  * `Application`
  * `Data`
  * `Runtime`
  * `Middleware`
  * `OS`

3. 平台既服务 ( 如 阿里云- ACK 容器服务 ) `PaaS` - `Platform as a Service` 如下需要自己 `部署/配置` 以及维护。

  * `Application`
  * `Data`

4. 软件既服务 ( 如 各类软件 微信、钉钉、邮箱 ) `SaaS` - `Software as a Service` 所有的服务软件都不需要自己维护, 直接使用既可。


# ansible

## ansible 简介

> ansible 是基于 `python2-paramiko` 模块开发的自动化运维工具。 实现了批量系统配置, 批量程序部署, 批量运行命令等功能。`ansible` 是基于模块工作的, 本身没有批量部署的能力。真正具有批量部署的是 `ansible` 所运行的模块, `ansible` 只是提供了一种框架。

**ansible 集合了众多运维工具（ pupet、cfengine、chef、func、fabric、saltstack ）的优点**

* ansible 发展史

  * ansible 作者是 `Michael DeHaan` 同时他也是 `Cobbler 与 Func` 作者。

  * 2012-03-09 发布 0.0.1 版本。

  * 2015-10-17 被 `Red Hat` 收购。



* ansible 特性

  * 基于 Python 开发

  * 模块化: 调用特定的模块(如: Paramiko、PyYAML、jinja2 等), 完成特定的任务。

  * 支持自定义模块

  * 部署简单, 基于`Linux`内置的 `Python` 、`Open-SSH` 和另一个 `agentless` 组件.

  * 支持 `PlayBook` 编排任务

  * 幂等性: 任务重复执行等于只执行一次, 不会重复执行多次相同命令。

  * 无需代理不依赖PKI.

  * 支持多语言模块编写.

  * `YAML`格式编排任务,支持丰富的数据结构.
 

## ansible 架构

{{< figure src="/img/posts/ansible/ansible-jg.png" >}}



{{< figure src="/img/posts/ansible/ansible-yl.png" >}}


* `ansible` 主要组成部分:

  * `ansible playbooks`: 任务剧本(任务集), 通过编排定义`ansible`任务集合的配置文件, 由`ansible` 顺序依次执行, 文件通常是 `JSON`格式的`YML`文件。

  * `Roles`: 角色, 多个 `ansilbe playbooks` 的集合.

  * `Inventory`: `ansible` 管理主机的清单 默认为 `/etc/ansible/hosts` 文件。

  * `Modules`: `ansible` 执行命令的功能模块, 一般为`ansible`内置核心模块, 也可以自定义第三方模块.

  * `plugins`: `ansible` 功能插件, 是功能模块的补充. 如: 连接类型插件、循环插件、变量插件、过滤插件等等.

  * `Api`: 提供第三方程序调用的开放接口.

  * `ansible`: `ansible` 的客户端命令工具. 执行 `ansible` 命令的主要程序.

    * `Ad-Hoc` 用户执行单条或多条的 `ansible` 命令.

    * `ansible playbook` 通过编写编排文件 `ansible` 执行命令集合.

      * `ansible playbook` 执行过程 -> 将编排好的任务写入 `ansible-playbook` 编排文件中 -> 通过 `ansible-playbook` 命令拆分任务集合, 然后按照预定的顺序逐条执行`ansible` 命令.



## ansible 安装

* `ansible` 安装很简单, 也有很多种方式.

  * 安装 `epel` 源以后 执行 `yum -y install ansible`

  * 使用 源码包 进行 安装

  * 通过 `git clone https://github.com/ansible/ansible` 以后进行安装

  * 使用 pip 命令安装




```shell

[root@jicki ~]# ansible --version
ansible 2.9.10
  config file = /etc/ansible/ansible.cfg
  configured module search path = [u'/root/.ansible/plugins/modules', u'/usr/share/ansible/plugins/modules']
  ansible python module location = /usr/lib/python2.7/site-packages/ansible
  executable location = /usr/bin/ansible
  python version = 2.7.5 (default, Apr  2 2020, 13:16:51) [GCC 4.8.5 20150623 (Red Hat 4.8.5-39)]

```


* `ansible` 相关说明

  * `/etc/ansible/ansible.cfg` 主配置文件, 配置`ansible`的工作特性.

  * `/etc/ansible/hosts` 主机清单.

  * `/etc/ansible/roles/` 存放(roles)角色的目录.

  * `/usr/bin/ansible` 二进制执行文件, `ansible` 主程序.

  * `/usr/bin/ansilbe-doc` 配置文档, 模块功能查看工具.

  * `/usr/bin/ansible-galaxy` 用于上传/下载 `roles` 模块到官方平台的工具.

  * `/usr/bin/ansible-playbook` 自动化任务、编排剧本工具`/usr/bin/ansible-pull` 远程执行命令的工具.

  * `/usr/bin/ansible-vault` 文件(如: playbook 文件) 加密工具.

  * `/usr/bin/ansible-console` 基于 界面的用户交互执行工具.     




## /etc/ansible/hosts

> 创建秘钥 `ssh-keygen -t rsa -f ~/.ssh/id_rsa  -C "jicki"`
>
> 拷贝秘钥到其他被控端 `ssh-copy-id ip`


**主机清单文件**


```shell
# #号为注释

# 单主机 直接写入 ip 或 nameserver
192.168.168.10

# 非标准22端口可直接用 : 填入
192.168.168.10:999
db1.example.com:999


# []包含的为主机组 如下 
[webservers]
192.168.168.10
192.168.168.11
192.168.168.12
192.168.168.13


# 主机也支持 nameserver 如下
[dbservers]
db1.example.com
db2.example.com
db3.example.com
db4.example.com

# 主机支持 [1:10] 类型的多主机 如下 
[docker]
# 等同于 192.168.168.10 ~ 192.168.168.120
192.168.168.1[0:20]


# 主机也支持 [a:z] 类型的 nameserver 如下
[kubernetes]
# 等同于 master-a ~ master-c
master-[a:c]
# 等同于 node-c ~ node-g
node-[c:g]

```


* 操作主机

  * `-m` 为指定模块 `ping` 为 模块名称

  * `-k` 为密码方式, 默认为 ssh-key 免密码方式登录


```shell
# ansible 通过 单主机进行操作 ( -k 为用户密码方式, 默认为 ssh-key )
ansible 192.168.168.10 -m ping -k


# ansible 通过 ':' 组合进行操作
ansible "192.168.168.10:192.168.168.20" -m ping -k


# ansible 通过 通配符加主机 进行操作
ansible 192.168.168.* -m ping -k


# ansible 通过 hosts 组名称 进行操作
ansible webservers -m ping -k


# ansible 通过 ':' 组合组进行操作
ansible 'webservers:dbservers' -m ping -k


# ansible 通过 通配符 进行操作
ansible '*servers' -m ping -k


# ansible 通过 ':&' 逻辑与 (两个组中都包含的主机)
ansible 'webservers:&dbservers' -m ping -k


# ansible 通过 ':!' 逻辑非 (在webservers 但不在 dbservers的主机)
ansible 'webservers:!dbservers' -m ping -k


# ansible 也支持多逻辑的组合
ansible 'webservers:dbserver:&appserver:!ftpservers' -m ping -k


# ansible 也支持正则表达式
ansible '~(web|db)serever' -m ping -k


# ansible 通过 all 对 hosts 清单下所有主机进行操作
ansible all -m ping -k


# ansible 通过 通配符 对 hosts 清单下所有主机进行操作
ansible '*' -m ping -k

```

* 输出结果

```shell
[root@jicki opt]# ansible all -m ping
10.0.3.13 | SUCCESS => {
    "ansible_facts": {
        "discovered_interpreter_python": "/usr/bin/python"
    },
    "changed": false,
    "ping": "pong"
}
```


## /etc/ansible/ansible.cfg

**ansible 主配置文件**


```shell
# defaults 为默认配置
[defaults]

# 主机清单的路径, 默认为如下
#inventory      = /etc/ansible/hosts

# 模块存放的路径 
#library        = /usr/share/my_modules/

# utils 模块存放路径
#module_utils   = /usr/share/my_module_utils/

# 远程主机脚本临时存放目录
#remote_tmp     = ~/.ansible/tmp

# 管理节点脚本临时存放目录 
#local_tmp      = ~/.ansible/tmp

# 插件的配置文件路径
#plugin_filters_cfg = /etc/ansible/plugin_filters.yml

# 执行并发数
#forks          = 5

# 异步任务查询间隔 单位秒
#poll_interval  = 15

# sudo 指定用户
#sudo_user      = root

# 运行 ansible 是否提示输入sudo密码
#ask_sudo_pass = True

# 运行 ansible 是否提示输入密码 同 -k
#ask_pass      = True

# 远程传输模式
#transport      = smart

# SSH 默认端口
#remote_port    = 22

# 模块运行默认语言环境
#module_lang    = C


# roles 存放路径
#roles_path    = /etc/ansible/roles

# 不检查 /root/.ssh/known_hosts 文件 建议取消
#host_key_checking = False


# ansible 操作日志路径 建议打开
#log_path = /var/log/ansible.log


```



## ansible 相关操作





### ansible 执行过程


1. 加载配置文件 `/etc/ansible/ansible.cfg`.

2. 加载对应的模块文件.

3. 通过 ansible 将模块或命令生成对应的临时 `py` 文件, 并将该临时文件 传输至远程服务器的对应 执行用户 临时目录下 `$HOME/.ansible/tmp/ansible-tmp-2123/xxx.py` >文件.

4. 对临时 `py` 文件授权 ( chmod u+x xx.py ).

5. 执行 `py` 文件,并返回执行结果.

6. 删除临时的 `py` 文件, sleep 0 退出.


---

* 执行状态

  * 绿色: 执行操作成功并且不需要做更改的操作.

  * 黄色: 执行操作成功, 并且对目标主机进行了变更操作. 如 更改文件内容, 重启服务,  删除文件等.

  * 红色: 执行操作失败.


---

### ansible-doc

* 显示模块帮助

  * `-l, --list` 列出可用模块

  * `-s, --snippet` 显示指定模块的 `playbook` 片段


```shell
# 例子

ansible-doc -l

ansible-doc ping

ansible-doc -s ping

```


---

### ansible

* `ansible <host-pattern>  [-m module_name] [-a args]`

  * `host-pattern`: 主机ip、主机名、主机组。

  * `module_name`: 模块的名称。如果不写 默认为 `-m command` 。

  * `args`: 模块的参数, 需要加上 `-a` 进行指定模块的参数。如: `ansible all -a 'hostname'

  * `-v、-vv、-vvv`: 显示详细的命令输出日志, v 越多越详细。如: `ansible all -m ping -vvv`

  * `--list`: 显示主机的列表。 如: `ansible all --list`

  * `-k / --ask-pass`: 提示输入ssh连接密码, 默认为 ssh-key 认证。如: `ansible all -m ping -k`

  * `-K / --ask-become-pass`: 提示输入 sudo 的密码。

  * `-C / --check`: 检查命令操作, 并不会执行。如: `ansible all -m ping -C` 

  * `-T / --timeout`: 执行命令的超时时间, 默认为 10s。如: `ansible all -m ping -T=2` 

  * `-u / --user`: 执行远程操作的用户. 如: `ansible all -m ping -u=root`

  * `-b / --become`: 代替旧版的 `sudo` 切换。 


### ansible 常用模块

> 截止 2020-08-10 ansible 模块为 3387 个.

---


#### Command 模块

* `command` 模块: 在远程主机上执行命令, 支持条件判断. ansible 默认模块, 可忽略 `-m` 参数直接操作.  注: `command` 模块 不支持 `$VARNAME` `<` `>` `|` `;` `&` 等符号. 

  * `ansible all -m command -a 'systemctl stop docker'

  * `ansible all -a 'docker ps -a'

  * `ansible all -a 'removes=/opt/ansible df -h'` 

    * 如果 /opt/ansible 不存在 就不执行 `df -h` 操作, 如果 /opt/ansible 存在, 就执行 `df -h` 操作.
 
  * `ansible all -a 'creates=/opt/ansible df -h'`
    
    * 如果 /opt/ansible 不存在 就执行 `df -h` 操作, 如果存在 /opt/ansible 就不执行 `df -h` 操作.

  * `ansible all -a 'chdir=/opt ls -lt'` 

    * 切换目录, 等于 `cd /opt && ls -lt` 操作


---


#### shell 模块


* `shell` 模块: shell 模块支持 command 所有的操作, 而且支持 `$VARNAME` `<` `>` `|` `;` `&` 等符号操作.  

  * `ansible all -m shell  -a 'ps -ef|grep docker'`



---


#### Script 模块

* `script` 模块: 执行脚本的命令. 只需要调用 ansible 本机上的脚本就可以在所有的选择主机上面执行脚本中的内容.

  * `ansible all -m script -a '/root/1.sh'`  

    * `/root/1.sh` 脚本只需要在 ansible 管理主机上既可.


---



#### Copy 模块


* `copy` 模块: 拷贝文件到远程主机.  

  * `ansible all -m copy -a 'src=/root/xxx.txt dest=/root/1.txt backup=yes'

    * `src`: 原文件路径

    * `dest`: 目标主机路径

    * `backup`: 如果目标主机文件存在, 会先进行备份, 再进行覆盖.


  * `ansible all -m copy -a 'src=/root/1.txt dest=/root/2.txt mode=0644 owner=jicki group=jicki'` 

    * `mode`: 修改权限

    * `owner`: 修改用户

    * `group`: 修改用户组


  * `ansible all -m copy -a 'content="hello\nworld\n" dest=/root/2.txt'` 

    * `content`: 将内容写入到目标文件中.

---



#### Fetch 模块


* `fetch` 模块: 将远程主机的文件, 下载到本机中, 下载成功会存放在以 主机 IP/名称 的文件夹中, 会包含原文件的整体路径.  (只能下载单个文件, 不支持目录, 可先打包成压缩包, 再进行下载)

  * `ansible all -m fetch -a 'src=/var/log/messages dest=/root/'`

    * `src`: 远程主机的文件路径

    * `dest`: 本机的文件夹路径


---



#### File 模块


* `file` 模块: 操作远程主机的文件. 如: 创建文件 `touch`、 删除文件 `absent` 等 

  * `ansible all -m file -a 'name=/root/1.txt owner=jicki group=jicki mode=0755 recurse=yes'`

    * `mode`: 修改权限

    * `owner`: 修改用户

    * `group`: 修改用户组

    * `recurse=yes`: 递归, 指对目录递归授权.

  * `ansible all -m file -a 'name=/root/1.txt state=touch'` 

    * `dest`、`name`、`path`: 指定远程主机的文件路径.

    * `state`: 文件操作类型. ( 默认为 `absent` )

      * `touch`: 创建空文件.
      * `directory`: 创建文件夹.
      * `absent`: 递归删除文件夹/文件.
      * `link`: 创建软连接.

  * `ansible all -m file -a 'src=/root/1.txt dest=/root/1.link state=link'`

    * `src`: 源文件, 这里用于指定 软连接的源文件.

---



#### Cron 模块


* `cron` 模块: 为远程主机添加定时任务. 

  * `ansible all -m cron -a 'weekday=1-5  job="echo `date`  >> /root/1.txt" name=echocron'`

    * `day`: 表示 天.  支持 ( 1-31, *, */2 ) 写法

    * `hour`: 表示 小时.  支持 ( 0-23, *, */2 ) 写法

    * `minute`: 表示 分钟. 支持 ( 0-59, *, */2 ) 写法

    * `month`: 表示 月. 支持 ( 1-12, *, */2 ) 写法

    * `weekday`: 表示 星期. 支持 ( 0-6, Sunday-Saturday, * )写法

    * `job`: 表示 计划任务的内容. 

    * `name`: 表示 计划任务名称. 相同的计划任务名称会覆盖. 

  * `ansible all -m cron -a 'disabled=true  job="echo `date`  >> /root/1.txt" name=echocron'`

    * `disabled`: 只是注释掉计划任务 并非删除. 

      * `true`、`yes` : 关闭计划任务. 关闭计划任务 必须指定 `job` 和 `name`.
      * `false`、`no`: 重新打开计划任务. 必须指定 `job` 和 `name`.


  * `ansible all -m cron -a 'name=echocron state=absent'`

    * `state`
      * `absent` 删除计划任务. 删除计划任务 只需要指定 `name` 既可.



---


#### Yum 模块



* `yum` 模块: 利用 yum 操作软件包, 如 安装、查询、卸载等.

  * `ansible all -m yum -a 'name=sysstat state=present'`

  * `ansible all -m yum -a 'name=/tmp/sysstat-12.3.3-1.2.x86_64.rpm'`

  * `ansible all -m yum -a 'name=sysstat update_cache=yes disable_gpg_check=yes'`

    * `name`: 软件包的名称, 或者rpm包, 远程服务器必须存在 rpm 包. 安装多个软件使用 `,` 号隔开. 如 `name=sysstat,mysql,redis`

    * `state`

      * `present`、`installed`:  安装软件. 默认为 `present`, 可不填 `state=present` .
      * `absent`、`removed`: 卸载/删除软件. 

    * `update_cache=yes`: 更新 yum 缓存后 在安装软件.

    * `disable_gpg_check=yes`: 禁用 gpg 检查.

  * `ansible all -m yum -a 'list=installed'`

    * `list`: 列出安装包.

      * `installed`: 已安装的软件
      * `updates`: 可以升级的软件
      * `available`: 可以安装的软件
      * `repos`: yum 源 


---


#### Service 模块


* `service`: 软件服务管理模块. 启动、关闭、重启 等操作.

  * `ansible all -m service -a 'name=vsftpd state=started enabled=yes'`

    * `name`: 服务名称.

    * `state`
      * `started`: 启动服务.
      * `stopped`: 停止服务.
      * `restarted`: 重启服务.
      * `reloaded`: 重新加载配置.

    * `enabled`:  设置是否开机启动.
      * `yes、true`
      * `no、false`



---



#### User 模块

* `user`: 管理系统用户的模块

  * `ansible all -m user -a 'name=nginx shell=/sbin/nologin system=yes home=/var/nginx groups=root uid=80 comment="nginx service user"'`

    * `name`: 用户名

    * `shell`: 指定用户的 `shell` 类型
   
    * `system`: 指定是否为 系统用户

    * `home`: 指定用户额外的home目录,默认为 /home/用户名 .

    * `groups`: 用户额外的 `groups` 组. 

    * `uid`: 指定用户的 uid 号.

    * `comment`: 用户额外说明. 


  * `ansible all -m user -a 'name=nginx state=absent remove=yes'`

    * `state`
      * `present`: 创建用户 (默认为present)
      * `absent`: 删除用户

    * `remove`: 是否删除 用户 home 目录.


---

#### Group 模块


* `group`: 管理系统用户组的模块

  * `ansible all -m group -a 'name=nginx system=yes gid=80'`

    * `name`: 用户组名

    * `system`: 是否为系统用户组

    * `gid`: 指定 gid 号

    * `state`
      * `present`: 创建用户组 (默认为present)
      * `absent`: 删除用户组

  * `ansible all -m group -a 'name=nginx state=absent'`


---

### ansible-galaxy

> 通过 https://galaxy.ansible.com/ 页面下载

**ansible-galaxy 工具用于下载对应的`roles`**


* `ansible-galaxy list geerlingguy.nginx`

  * `list`: 查看本地的 roles 角色.


* `ansible-galaxy install geerlingguy.nginx`

  * `install`: 下载 `roles` 角色. 会下载到 `$HOME/.ansible/roles/` 目录下


* `ansible-galaxy remove geerlingguy.nginx`

  * `remove`: 删除已下载的 `roles` 角色. 在目录中删除也可以.


---


### ansible playbook


> playbook 流程图

{{< figure src="/img/posts/ansible/playbooks.jpg" >}}


---


#### playbook 与 YAML 描述


* `playbook` 由一个或多个 `play` 组成.

* `playbook` 中 每个`play` 必须包含 `hosts` 和 `tasks`.

* `playbook` 以 `YAML` 语法编写.

  * `YAML` 约定以 `---` 开头 和 开始不同的 `play` .

  * `YAML` 以 `#` 作为注释.

  * `YAML` 必须统一缩进, 空格 与 `tab` 不能混用, 缩进的级别也必须相同, 同级缩进代表同样的级别.

  * `YAML` 文件内容 是大小写敏感的, 跟 Linux 一样区分大小写.

  * `YAML`  key/value 形式可写在同一行也可以换行写. 同行使用 `:` 隔开.

  * `YAML`  一个完整的代码块功能最少包含2个元素. 如 name: task

  * `YAML`  一个 name 下只能包含一个 task

  * `YAML`  `-` 开头的为列表, `key/value` 形式的为字典.

  * `YAML` 特性
    * 可读性好
    * 和脚本语言交互性好
    * 使用实现语言的数据类型
    * 一致性的信息模型
    * 易于实现
    * 基于流处理
    * 表达能力好, 扩展性强
  


---

#### playbook 核心元素


* `hosts` : 远程主机列表 ( ip / 主机名 / 组名 )

* `tasks` : 任务集, 任务列表, 有两种写法

  * `action: module args` : action: 模块名 参数

  * `module: args` : 模块名: 参数 (一般使用这种)

  * `ignore_errors: True` 当前 task 出错时仍然会向下执行

* `varniables` : 内置变量或自定义变量在 `playbook` 文件中调用

* `templates` : 模板, 可替换模板文件中的变量并实现一些简单逻辑的文件

* `handles` : 与 `notity` 结合使用, 由特定条件触发的操作, 满足条件才执行, 否则不执行

* `tags` : 标签 指定任务执行, 用于执行一个 `playbook` 中的部分代码. 主要用于测试.

---

#### ansible-playbook 命令

* `ansible-playbook`

  * `-C / --check` : Check 检查脚本运行情况, 不会在远程服务器里运行.

  * `--list-hosts` : 列出运行此 任务 的主机.

  * `--list-tasks` : 列出任务组的具体任务列表.

  * `--limit` : 只对主机列表中的某台主机执行.

  * `-v -vv -vvv` : 显示详细的执行过程, `v` 越多就越详细.

---


#### ansible-playbook 变量

* 变量名要求: 只允许使用 `字母` 、`数字` 、 `_` 组成, 而且只能以 `字母`开头.


---


* 内置的公共变量: 


  * 使用 `ansible all -m setup` 可以获取到主机的系统变量名称.

    * `ansible all -m setup -a 'filter=*addresses*'` , 可使用 filter 参数进行过滤.

---

* 通过文件自定义变量:

  * `/etc/ansible/hosts` 文件中定义

    * 对主机组中的主机单独定义变量, 优先级高于公共变量.

    * 对主机组中的所有主机定义统一变量, 优先级低于对单独主机定义的变量.


```ini

[appserver]
# 对单独主机 定义变量 node_id
10.0.3.13 node_id=13

# 对主机组 定义统一变量 domain_name
[appserver:vars]
domain_name=jicki.cn


```

* 使用变量 就可以灵活配置不同主机的 `hostname`

```yml
---
- hosts: all
  remote_user: root

  tasks:
    - name: set hostname
      hostname: name={{ node_id }}.{{ domain_name }}

```



---

* 通过命令行定义变量: 通过命令行定义的变量优先级是最高的
  
  * `ansible-playbook -e varname=valur`

---
  
* 在 `playbook` 文件里 定义变量.

  * 通过 `{{ 变量名 }}` 使用变量.

  * 通过 `vars:` 列表 定义多个 变量. 


```yml

---
- hosts: all
  remote_user: root

  # 定义变量
  vars:
    - pkg_name: httpd 

  tasks:
    - name: install {{ pkg_name }}
      # 使用变量
      yum: name={{ pkg_name }}

```


---

* 通过定义单独的变量文件 用于统一存放变量, 可避免变量的重复定义.

  * 定义单独的 变量文件, 只需要将所有变量以 `key: value` 形式写入到 `yml` 文件中既可.

  * 在 `playbook` 文件中, 只需要使用 `vars_files:` 指定 `yml` 文件路径既可.


---

* vars.yml 变量文件

```yml
---
pkg_name: httpd
file_name: jicki.cn

```


* install.yml  playbook 文件

```yml
---
- hosts: all
  remote_user: root

  # 配置模板文件
  vars_files:
    # 指定文件的路径
    - vars.yml

  tasks:
    - name: install {{ pkg_name }}
      yum: name={{ pkg_name }}
    - name: create {{ file_name }} file
      file: name=/root/{{ file_name }}.txt state=touch

```


* 执行 playbook 操作


```shell

[root@jicki ansible]# ansible-playbook install.yml

PLAY [all] *******************************************************************************************************

TASK [Gathering Facts] *******************************************************************************************
ok: [10.0.3.13]

TASK [install httpd] *********************************************************************************************
changed: [10.0.3.13]

TASK [create jicki.cn file] **************************************************************************************
changed: [10.0.3.13]

PLAY RECAP *******************************************************************************************************
10.0.3.13                  : ok=3    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

---


#### ansible-playbook template

* template 是一个模块,并且只能用于 playbook 下.


---
* `templates` 文件, 可嵌套脚本 ( 利用模板编程语言 Jinja2 编写 )

  * `Jinja2` 语言, 使用字面量.
  
    * 字符串: 使用单引号或双引号.

    * 数字: 整数, 浮点数.

    * 列表: [A1, A2, ...]

    * 元组: (B1, B2, ...)

    * 字典: {key1:value1, key2:value2, ...}

    * 布尔值: true/false

    * 算术运算: `+`, `-`, `*`, `/`, `//`, `%`, `**` 

    * 比较操作: `==`, `!=`, `>`, `>=`, `<`, `<=` 

    * 逻辑运算: and, or, not

    * 流表达式: for,  if,  when



---

* `templates` 根据模板块文件动态生成对应的配置文件

  * `templates` 文件一般约定存放于 templates 目录下, 并且以 `.j2` 为后缀

  * templates 目录需要与 `playbook` 的 `yml` 文件在同级目录中.


```shell
[root@jicki nginx]# tree .
.
|-- nginx.yml
`-- templates
    `-- nginx.conf.j2
```

---


* 算术运算 例子 



* nginx.conf.j2 文件

```j2
user nginx;
# 这里使用 环境变量 vcpus * 2 
worker_processes {{ ansible_processor_vcpus * 2 }};
error_log /var/log/nginx/error.log;
pid /run/nginx.pid;

include /usr/share/nginx/modules/*.conf;

events {
    worker_connections 1024;
}

http {
    log_format  main  '$remote_addr - $remote_user [$time_local] "$request" '
                      '$status $body_bytes_sent "$http_referer" '
                      '"$http_user_agent" "$http_x_forwarded_for"';

    access_log  /var/log/nginx/access.log  main;

    sendfile            on;
    tcp_nopush          on;
    tcp_nodelay         on;
    keepalive_timeout   65;
    types_hash_max_size 2048;

    include             /etc/nginx/mime.types;
    default_type        application/octet-stream;

    include /etc/nginx/conf.d/*.conf;

    server {
        listen       80 default_server;
        listen       [::]:80 default_server;
        server_name  _;
        root         /usr/share/nginx/html;

        # Load configuration files for the default server block.
        include /etc/nginx/default.d/*.conf;

        location / {
        }

        error_page 404 /404.html;
            location = /40x.html {
        }

        error_page 500 502 503 504 /50x.html;
            location = /50x.html {
        }
    }
}

```


---

* nginx.yml `playbook` 文件

```yml
---
- hosts: all
  remote_user: root

  tasks:
    - name: install nginx
      yum: name=nginx
    - name: template conf
      # 如果 yml 与 templates 目录同级, src 直接写.j2 文件
      template: src=nginx.conf.j2 dest=/etc/nginx/nginx.conf
      notify:
        - restart nginx
    - name: start nginx
      service: name=nginx state=started enabled=yes

  handlers:
    - name: restart nginx
      service: name=nginx state=restarted

```


---

* `when` 条件语句 例子

```shell
[root@jicki nginx]# tree .
.
|-- nginx.yml
`-- templates
    |-- nginx.conf.centos7.j2
    `-- nginx.conf.centos8.j2
```

---

*  `playbook`  文件

```yml
---
- hosts: all
  remote_user: root

  tasks:
    - name: install nginx
      yum: name=nginx
    - name: template centos 7 conf
      # 如果 yml 与 templates 目录同级, src 直接写.j2 文件
      template: src=nginx.conf.centos7.j2 dest=/etc/nginx/nginx.conf
      # 使用 when 语句进行判断 如果变量为 "7" 执行这个
      when: ansible_distribution_major_version == "7"
      notify:
        - restart nginx
    - name: template centos 8 conf
      # 如果 yml 与 templates 目录同级, src 直接写.j2 文件
      template: src=nginx.conf.centos8.j2 dest=/etc/nginx/nginx.conf
      # 使用 when 语句进行判断 如果变量为 "8" 执行这个
      when: ansible_distribution_major_version == "8"
      notify:
        - restart nginx
    - name: start nginx
      service: name=nginx state=started enabled=yes

  handlers:
    - name: restart nginx
      service: name=nginx state=restarted

```


* 执行 playbook 文件

  * `skipping` 状态表示跳过执行这个 TASK .

```shell

[root@jicki nginx]# ansible-playbook nginx.yml

PLAY [all] *******************************************************************************************

TASK [Gathering Facts] *******************************************************************************
ok: [10.0.3.13]

TASK [install nginx] *********************************************************************************
ok: [10.0.3.13]

TASK [template centos 7 conf] ************************************************************************
changed: [10.0.3.13]

TASK [template centos 8 conf] ************************************************************************
skipping: [10.0.3.13]

TASK [start nginx] ***********************************************************************************
ok: [10.0.3.13]

RUNNING HANDLER [restart nginx] **********************************************************************
changed: [10.0.3.13]

PLAY RECAP *******************************************************************************************
10.0.3.13      : ok=5    changed=2    unreachable=0    failed=0    skipped=1    rescued=0    ignored=0

```



---

* 迭代 `with_items` 执行重复任务.

  * 对于迭代选项, 固定变量名为 `item` .
  * 在 `task` 中使用 `with_items` 指定需要迭代的元素列表.
    * 元素列表 支持 `字符串` 和 `字典` .


* `playbook` 文件

```yml
---
- hosts: all
  remote_user: root

  tasks:
    - name: create some files
      # {{ item }} 为特殊变量, 代表 with_items 列表中的内容
      file: name=/tmp/{{ item }} state=touch
      with_items:
        - file1
        - file2
        - file3
        - file4
    - name: install some software
      yum: name={{ item }}
      with_items:
        - htop
        - sl
        - hping3
```


---


* 迭代嵌套子变量.

  * 对迭代中的变量进行嵌套关联的操作.
 

* `playbook` 文件


```yml

---
- hosts: all
  remote_user: root

  tasks:
    - name: create some files
      # {{ item }} 为特殊变量, 代表 with_itmes 列表中的内容
      file: name=/tmp/{{ item }} state=touch
      with_items:
        - file1
        - file2
        - file3
        - file4

    - name: create some group
      group: name={{ item }}
      with_items:
        - j1
        - j2
        - j3
        - j4

    - name: create some user
      # 使用 item.key值 进行引用
      user: name={{ item.name }} group={{ item.group }}
      # 使用 字典 定义 嵌套的子 变量
      with_items:
        - { name: 'file1', group: 'j1' }
        - { name: 'file2', group: 'j2' }
        - { name: 'file3', group: 'j3' }
        - { name: 'file4', group: 'j4' }

    - name: permission some files
      file: name=/tmp/{{ item.name }} owner={{ item.name }} group={{ item.group }}
      with_items:
        - { file: 'file1', name: 'file1', group: 'j1' }
        - { file: 'file2', name: 'file2', group: 'j2' }
        - { file: 'file3', name: 'file3', group: 'j3' }
        - { file: 'file4', name: 'file4', group: 'j4' }


```

* 执行 `playbook` 文件

```shell
[root@jicki ansible]# ansible-playbook file.yml

PLAY [all] *****************************************************************************************

TASK [Gathering Facts] *****************************************************************************
ok: [10.0.3.13]

TASK [create some files] ***************************************************************************
changed: [10.0.3.13] => (item=file1)
changed: [10.0.3.13] => (item=file2)
changed: [10.0.3.13] => (item=file3)
changed: [10.0.3.13] => (item=file4)

TASK [create some group] ***************************************************************************
changed: [10.0.3.13] => (item=j1)
changed: [10.0.3.13] => (item=j2)
changed: [10.0.3.13] => (item=j3)
changed: [10.0.3.13] => (item=j4)

TASK [create some user] ****************************************************************************
changed: [10.0.3.13] => (item={u'group': u'j1', u'name': u'file1'})
changed: [10.0.3.13] => (item={u'group': u'j2', u'name': u'file2'})
changed: [10.0.3.13] => (item={u'group': u'j3', u'name': u'file3'})
changed: [10.0.3.13] => (item={u'group': u'j4', u'name': u'file4'})

TASK [permission some files] ***********************************************************************
changed: [10.0.3.13] => (item={u'group': u'j1', u'name': u'file1', u'file': u'file1'})
changed: [10.0.3.13] => (item={u'group': u'j2', u'name': u'file2', u'file': u'file2'})
changed: [10.0.3.13] => (item={u'group': u'j3', u'name': u'file3', u'file': u'file3'})
changed: [10.0.3.13] => (item={u'group': u'j4', u'name': u'file4', u'file': u'file4'})

PLAY RECAP *****************************************************************************************
10.0.3.13    : ok=5    changed=4    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```



















---



#### Example - tasks

* 基础的例子 

```yml
---
# 指定主机组
- hosts: all
  # 指定执行 用户
  remote_user: root

  # 任务
  tasks:
    # 任务的名称
    - name: ping server
      ping:
    - name: echo hostname
      # shell 为模块名, 后面等同于 -a '' 参数
      shell: hostname
    - name: touch file
      file: name=/tmp/file.txt state=touch
    - name: echo file
      shell: ls -l /tmp/file.txt
```

---

* `ansible-playbook hello.yml` 运行 `playbook`

  * 输出如下:

    * `ok` : ok 表示没有任何更改, 绿色

    * `changed` : changed 表示有修改, 黄色


```shell
PLAY [all] **********************************************************

TASK [Gathering Facts] **********************************************
ok: [10.0.3.13]

TASK [ping server] **************************************************
ok: [10.0.3.13]

TASK [echo hostname] ************************************************
changed: [10.0.3.13]

TASK [touch file] ***************************************************
changed: [10.0.3.13]

TASK [echo file] ****************************************************
changed: [10.0.3.13]


PLAY RECAP **********************************************************
10.0.3.13   : ok=5    changed=3    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```


---

#### Example - handles

* `handles` 与 `notity` 结合的例子 

  * 同一个`name` 下可以定义多个 `notify` 配置关联到不同的 `handlers` 中.

---

```yml
- hosts: all
  remote_user: root

  tasks:
    - name: copy httpd.conf
      copy: src=/root/ansible/httpd.conf dest=/etc/httpd/conf/httpd.conf backup=yes
      # 关联多个触发器的写法
      notify:
        - restart httpd
        - check status httpd
        - check network port

```
---


* 例子:

```yml
---
- hosts: all
  remote_user: root

  tasks:
    - name: install httpd
      yum: name=httpd
    - name: copy httpd.conf
      copy: src=/root/ansible/httpd.conf dest=/etc/httpd/conf/httpd.conf backup=yes
      # 此任务 如果有变动会触发如下定义名称的触发器
      notify: restart httpd
    - name: start httpd
      service: name=httpd state=started enabled=yes

  # 触发器, 需要配置 notify 触发
  handlers:
    - name: restart httpd
      service: name=httpd state=restarted

```

---

* `ansible-playbook httpd.yml` 

  * 第一次执行输出如下:

```shell

PLAY [all] ********************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [10.0.3.13]

TASK [install httpd] **********************************************************************************************
changed: [10.0.3.13]

TASK [copy httpd.conf] ********************************************************************************************
ok: [10.0.3.13]

TASK [start httpd] ************************************************************************************************
changed: [10.0.3.13]

PLAY RECAP ********************************************************************************************************
10.0.3.13                  : ok=4    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```

---

* 修改 httpd.conf 文件以后

  * 第二次执行输出如下:

```shell

PLAY [all] ********************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [10.0.3.13]

TASK [install httpd] **********************************************************************************************
ok: [10.0.3.13]

TASK [copy httpd.conf] ********************************************************************************************
changed: [10.0.3.13]

TASK [start httpd] ************************************************************************************************
ok: [10.0.3.13]

RUNNING HANDLER [restart httpd] ***********************************************************************************
changed: [10.0.3.13]

PLAY RECAP ********************************************************************************************************
10.0.3.13                  : ok=5    changed=2    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0


```


---

#### Example - tags

* 基于 `tags` 的例子

  * 定义了 `tags` 后可通过定义的 `tags` 单独运行该 `tags`. 运行多个可用 `,` 号隔开.

  * 多个不同的任务 可以定义相同名称的 `tags`. 

```yml
---
- hosts: all
  remote_user: root

  tasks:
    - name: install httpd
      yum: name=httpd
    - name: copy httpd.conf
      copy: src=/root/ansible/httpd.conf dest=/etc/httpd/conf/httpd.conf backup=yes
      # 此任务 如果有变动会触发如下定义名称的触发器
      notify:
        - restart httpd
      # 定义标签
      tags: cpconf
    - name: start httpd
      service: name=httpd state=started enabled=yes
      # 定义标签
      tags: sthttpd

  # 触发器, 需要配置 notify 触发
  handlers:
    - name: restart httpd
      service: name=httpd state=restarted
```

---

* 执行命令 `-t` 指定标签

```shell
[root@jicki ansible]# ansible-playbook -t sthttpd httpd.yml

PLAY [all] ********************************************************************************************************

TASK [Gathering Facts] ********************************************************************************************
ok: [10.0.3.13]

TASK [start httpd] ************************************************************************************************
changed: [10.0.3.13]

PLAY RECAP ********************************************************************************************************
10.0.3.13                  : ok=2    changed=1    unreachable=0    failed=0    skipped=0    rescued=0    ignored=0

```






---

### ansible-vault

* `playbook` 文件加密工具


* `ansible-vault encrypt hello.yml`

  * `encrypt`: AES256 加密 ( 会提示输入密码 )

  * `view`: 加密的情况下 查看 原来的内容.

  * `edit`: 编辑加密的 playbook 文件.

  * `decrypt`: 解密.

  * `rekey`: 修改加密密码.


---

### ansible-console

* `ansible-console`: 可交互执行命令, 支持 `Tab` 键.


```shell
[root@jicki ~]# ansible-console
Welcome to the ansible console.
Type help or ? to list commands.

root@all (1)[f:5]$

```

* `root@all (1) [f:5]$`

  * `root`: 当前执行用户.
  * `all`: 表示当前主机清单.
  * `(1)`: 表示当前主机清单下包含 `1` 台主机. 
  * `[f:5]`: 表示并发执行任务数为 `5` 个. 



