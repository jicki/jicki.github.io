# ansible playbook 从入门到放弃


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



## ansible 命令


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

* `shell` 模块: shell 模块支持 command 所有的操作, 而且支持 `$VARNAME` `<` `>` `|` `;` `&` 等符号操作.  

  * `ansible all -m shell  -a 'ps -ef|grep docker'`



---

* `script` 模块: 执行脚本的命令. 只需要调用 ansible 本机上的脚本就可以在所有的选择主机上面执行脚本中的内容.

  * `ansible all -m script -a '/root/1.sh'`  

    * `/root/1.sh` 脚本只需要在 ansible 管理主机上既可.


---

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

* `fetch` 模块: 将远程主机的文件, 下载到本机中, 下载成功会存放在以 主机 IP/名称 的文件夹中, 会包含原文件的整体路径.  (只能下载单个文件, 不支持目录, 可先打包成压缩包, 再进行下载)

  * `ansible all -m fetch -a 'src=/var/log/messages dest=/root/'`

    * `src`: 远程主机的文件路径

    * `dest`: 本机的文件夹路径


---

* `file` 模块: 操作远程主机的文件. 如: 创建文件 `touch`、 删除文件 `absent` 等 

  * `ansible all -m file -a 'name=/root/1.txt owner=jicki group=jicki mode=0755'`

    * `mode`: 修改权限

    * `owner`: 修改用户

    * `group`: 修改用户组  

  * `ansible all -m file -a 'name=/root/1.txt state=touch'` 

    * `dest`、`name`、`path`: 指定远程主机的文件路径.

    * `state`: 文件操作类型.

      * `touch`: 创建空文件.
      * `directory`: 创建文件夹.
      * `absent`: 递归删除文件夹/文件.
      * `link`: 创建软连接.

  * `ansible all -m file -a 'src=/root/1.txt dest=/root/1.link state=link'

    * `src`: 源文件, 这里用于指定 软连接的源文件.

---

* `cron` 模块: 为远程主机添加定时任务. 

  * `ansible all -m cron -a 'weekday=1-5  job="echo `date`  >> /root/1.txt" name=echocron'`

    * `day`: 表示 天.  支持 `1-31`, `*`, `*/2` 写法

    * `hour`: 表示 小时.  支持 `0-23`, `*`, `*/2` 写法

    * `minute`: 表示 分钟. 支持 `0-59`, `*`, `*/2` 写法

    * `month`: 表示 月. 支持 `1-12`, `*`, `*/2` 写法

    * `weekday`: 表示 星期. 支持 `0-6`, `Sunday-Saturday`, `*` 写法




## ansible 执行过程


1. 加载配置文件 `/etc/ansible/ansible.cfg`.

2. 加载对应的模块文件.

3. 通过 ansible 将模块或命令生成对应的临时 `py` 文件, 并将该临时文件 传输至远程服务器的对应 执行用户 临时目录下 `$HOME/.ansible/tmp/ansible-tmp-2123/xxx.py` 文件.

4. 对临时 `py` 文件授权 ( chmod u+x xx.py ).

5. 执行 `py` 文件,并返回执行结果.

6. 删除临时的 `py` 文件, sleep 0 退出.


---

* 执行状态 

  * 绿色: 执行操作成功并且不需要做更改的操作.

  * 黄色: 执行操作成功, 并且对目标主机进行了变更操作. 如 更改文件内容, 重启服务,  删除文件等.

  * 红色: 执行操作失败.



