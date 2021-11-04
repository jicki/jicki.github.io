# linux FAQ 以及 命令



# Linux 篇：


## CentOs 7 修改主机名

```
hostnamectl --static set-hostname <host-name>
```
 

## 删除0字节文件

```
find -type f -size 0c | xargs rm -f
```


## 授权某用户，用户组可执行


```
# jenkins 为更新权限, nginx 为 web 运行权限

usermod -a -G nginx jenkins



# 授权 用户组执行权限 (jenkins 可以在nginx 权限中执行 操作权限)
chmod -R ug+w directories

```
 

```
# 创建公私钥

ssh-keygen -t rsa -b 4096 -C "jicki@qq.com" -N "" -f /home/jicki/id_rsa

```



## 查看系统启动时间

```
date -d "$(awk -F. '{print $1}' /proc/uptime) second ago" +"%Y-%m-%d %H:%M:%S" 

```

## 查看系统运行时间

```
cat /proc/uptime| awk -F. '{run_days=$1 / 86400;run_hour=($1 % 86400)/3600;run_minute=($1 % 3600)/60;run_second=$1 % 60; printf("系统已运行：%d天%d时%d分%d秒",run_days,run_hour,run_minute,run_second)}' 


```


## 截取 N - M 的日志

```
sed '/13:30:00/,/13:50:00/!d' catalina.out >> 22222.txt
```



### Sed 替换变量带特殊字符

```
#/bin/bash

IMAGE_NAME="jicki/hd-cloud-admin:0fjdskfjdsklfds0"

echo $IMAGE_NAME

# 原替换语句为
# sed -i "s/jicki\/hd-cloud-admin.*$/${IMAGE_NAME}/g" 1.txt
# 因为变量包含特殊字符 / 号，所以以上语句会报错
# 修改为以下语句:

sed -i 's%jicki/hd-cloud-admin.*$%'"${IMAGE_NAME}"'%g' 1.txt

```






```
# 1.txt 内容如下:

jicki
hd-cloud-admin
image: jicki/hd-cloud-admin:12342423

```



 

## 添加主机路由


方法1：

```
cat /etc/sysconfig/network-scripts/route-em1               

# route-em1有严格的要求，em1必须与实际网卡名称对应，否则会失败
ADDRESS0=10.6.0.0                                                 
# 可以添加多条路由，必须从编号0开始
NETMASK0=255.255.0.0
GATEWAY0=172.16.1.1
```


方法2：

```
cat /etc/sysconfig/network-scripts/route-em1

10.6.0.0/16 via 172.16.1.1 dev em1
```
 

## centos 7 内核顺序变更

查看内核顺序:

```
awk -F\' '$1=="menuentry " {print $2}' /etc/grub2.cfg

# 选择内核0为默认

grub2-set-default 0
```
 

 

## 查询缺少的.so 文件

如:  ldd nginx

```
libcrypto.so.6 => not found

yum provides libcrypto.so.6

openssl098e-0.9.8e-29.el7.centos.i686 : A compatibility version of a general cryptography and TLS library

Repo        : base

Matched from:

Provides    : libcrypto.so.6
```

```
yum -y install openssl098e
```
 

 
## crontab FAQ


关于 Crontab 不能使用的问题..没安装等..

```
yum install vixie-cron

yum install crontabs

service crond start
```


## 修改时区

```
vi /etc/sysconfig/clock

ZONE="Aisa/Shanghai"

UTC=true

ARC=false

# 更新时间不生效，还是原来的时区...

cp /usr/share/zoneinfo/Asia/Shanghai /etc/localtime
```
 

 

## 创建大于2T的分区

Fdisk所使用的分区表为MBR，Master Boot Record，即主引导记录。

硬盘的0面、0道、1扇区就是主引导扇区，Fdisk将会写512个字节的记录在此，即MBR记录。

MBR分区表：（MBR含义：Master Boot Record，主引导记录）

所支持的最大卷：2T （T; terabytes,1TB=1024GB）

对分区的设限：最多4个主分区或3个主分区加一个扩展分区（扩展分区中支持无限制的逻辑驱动器）

GPT分区表：（GPT含义：GUID分区表）

支持最大卷：18EB，（E：exabytes,1EB=2(10) PB=1024PB，1PB=2(10) TB=1024TB）

每个磁盘最多支持128个分区

```

# parted /dev/sdb

(parted) mkpart primary 0% 10%

(parted) mkpart primary 10% 100%

(parted) p

Model: DELL MD3000 (scsi)

Disk /dev/sdb: 13.0TB

Sector size (logical/physical): 512B/512B

Partition Table: gpt

Number Start End Size File system Name Flags

1 17.4kB 1300GB 1300GB primary

2 1300GB 13.0TB 11.7TB primary

(parted)quit

```
 

## shell 下 sudu FAQ


Shell命令下使用sudo 提示 sudo: sorry, you must have a tty to run sudo 的错误

编辑 /etc/sudoers 文件

查找 “

```
Defaults  requiretty
```

修改为

```
Defaults:username  !requiretty
```


Shell 命令下使用 sudo echo > 这样的命令 依然提示 权限不够

这是因为重定向符号 “>” 也是 bash 的命令。sudo 只是让 echo 命令具有了 root 权限，

但是没有让 “>” 命令也具有root 权限，所以 bash 会认为这个命令没有写入信息的权限。

 
可以利用 “sh -c” 命令，它可以让 bash 将一个字串作为完整的命令来执行，这样就可以将 sudo 的影响范围扩展到整条命令。

 
```
sudo sh -c "echo 654321 > 1.txt"
```
 

 

 

## SVN FAQ

钩子日志输出

编辑 post-commit 文件

写入

```
svnlook changed /svn/yx > /svn/yx/changed.log && /shell/commit.sh
```

svnlook changed 命令 将 svn/yx 库操作记录到 changed.log 文件内... 然后用shell读取 changed.log 内的操作~执行脚本...

 

svn url 变更

```
svn switch --relocate svn://123.123.123.123/rl/api svn://192.168.0.74/rl/api


# svn switch --relocate 原url地址  新URL地址 
```
 


## Git FAQ

git pull / push 不输入密码

```
1.  首先执行一次 git pull 或者 git push 
然后会在 ~/  目录下生成一个 .git-credentials 文件


2. 在终端下执行  git config --global credential.helper store

3. 可以看到~/.gitconfig文件，会多了一项：
    [credential]
        helper = store

4. 最后测试一下 不用输入密码了

```




## Mysql FAQ


> 删除mysql 的binlog

一：查看备份的日志。

```
mysql> show binary logs;

+------------------+------------+

| Log_name     | File_size |

+------------------+------------+

| mysql-bin.000001 | 392914665 |

| mysql-bin.000002 |    2765 |

| mysql-bin.000003 | 1073742259 |

| mysql-bin.000004 | 1073741949 |

+------------------+------------+

11 rows in set (0.11 sec)
```
 

删除指定binglog , 如下语句，指删除3 之前的所有binlog,而非 一个binlog

```
mysql> purge binary logs to 'mysql-bin.00003';

mysql> show binary logs;

+------------------+------------+

| Log_name     | File_size |

+------------------+------------+

| mysql-bin.000004 | 1073741949 |

+------------------+------------+

11 rows in set (0.11 sec)
```
 


---

> 数据库 丢失 `ibdata1` 文件, 数据库 frm 与 ibd 文件还存在


* 借助 `Mysql-utilities` 恢复数据

```
# 安装 mysql-utilities1.6.5 版本即可, 它是基于python2实现的，所以python也不需要升级

wget https://cdn.mysql.com/archives/mysql-utilities/mysql-utilities-1.6.5.tar.gz

tar xvf mysql-utilities-1.6.5.tar.gz

cd mysql-utilities-1.6.5

python setup.py build

python setup.py install

mysqldiff --version

```

*  恢复表结构

```
mysqlfrm --diagnostic /opt/local/data/autumn2 > createtb.sql
```

通过 frm 文件生成了建表语句 createtb.sql

然后通过建表语句, 先创建好库, 然后通过 source /backup/createtb.sql，完成数据结构的恢复

使用的数据库版本需要和之前的数据库保持一致, 所使用字符集也需要保持一致, 相关的配置参数也都保持一致.

否则可能会出现相关问题


*  恢复数据


清理 `autumn2` 数据库中表的数据文件

```
mysql -uroot -p123456 -e "show tables from autumn2"| grep -v Tables_in_autumn2 | while read a; do mysql -uroot -p123456 -e "alter table autumn2.$a discard tablespace"; done

```

执行成功后，可以到对应的物理文件存储处，会发现没有 idb 文件了

停数据库服务,将有数据旧的idb文件复制到对应的文件存储的地方，再启数据库服务

数据文件导入数据库中

```
mysql -uroot -p123456 -e "show tables from autumn2"| grep -v Tables_in_autumn2 | while read a; do mysql -uroot -p123456 -e "alter table autumn2.$a import tablespace"; done

```

* 导入完成后，我们可以对表进行检查

```
mysqlcheck -c autumn2 -uroot -p123456
autumn2.alcohol_cargo_base_bt OK
autumn2.userbase OK
autumn2.worklog OK
```







 

## Mongodb FAQ

查看当前性能

```
mongodb/bin/mongostat -h xx.xx.xx.xx:27017
```
 

查看读写

```
mongodb/bin/mongotop -h xx.xx.xx.xx:27017
```
 

查看当前执行语句

```
db.currentOp()
```
 

杀掉进程(先执行 db.currentOp()获取进程号,类似ps -ef)

```
db.killOP(2920488)
```
 

查看最近错误

```
db.getLastError()db.getLastError()
```


## Python FAQ


```
# 模块版本不兼容

packages/requests/__init__.py:80: RequestsDependencyWarning: urllib3 (1.21.1) or chardet (2.2.1) doesn‘t match a supported version!

# 依次执行如下命令 (一条一条执行)

pip uninstall urllib3
pip uninstall requests
pip uninstall  chardet


pip install requests
```

## Docker FAQ

```
# 红帽系统安装 docker-ce

# 安装pigz
yum -y install http://mirror.centos.org/centos/7/extras/x86_64/Packages/pigz-2.3.3-1.el7.centos.x86_64.rpm

# 安装 container-selinux
yum -y install http://mirror.centos.org/centos/7/extras/x86_64/Packages/container-selinux-2.55-1.el7.noarch.rpm
```

## AliYun FAQ

```
# 删除阿里云盾

curl -sSL http://update.aegis.aliyun.com/download/quartz_uninstall.sh | sudo bash
 
rm -rf /usr/local/aegis
 
rm -rf /usr/sbin/aliyun-service

```

