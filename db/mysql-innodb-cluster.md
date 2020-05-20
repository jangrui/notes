# InnoDB Cluster

`MySQL InnoDB Cluster`: 基于 MySQL Group Replication，提供了易于管理的 API、应用故障转移和路由、易于配置，提供比群组复制更高级别的可用性。

MGR 有两种模式，一种是单主模式（Single-Primary），一种是多主模式（Multi-Primary）。

Multi-Primary 模式中，所有的节点都是主节点，都可以同时被读写，看上去这似乎更好，但是因为多主的复杂性，在功能上如果设置了多主模式，则会有一些使用的限制，比如不支持级联操作（Foreign Keys with Cascading Constraints）。

## InnoDB Cluster 组件

![InnoDB集群概述](https://dev.mysql.com/doc/refman/8.0/en/images/innodb_cluster_overview.png ':size=300')

- `MySQL Group Replication`: 提供 DB 的扩展、自动故障转移。
- `MySQL Router`: 轻量级中间件，提供负载均衡和应用连接的故障转移。
- `MySQL Shell`: MySQL Server 的高级客户端和代码编辑器，提供了 JavaScript 和 Python 实现的 X DevAPI 和 AdminAPI。
    - `X DevAPI`: 通过 XProtocol 与服务器通信，对关系型数据模式和文档型数据模式进行操作。
    - `Admin API`: 可以用于对 Innodb Cluster 进行配置管理。

## 环境配置

- IP 映射到主机名

```bash
cat >> /etc/hosts <<EOF
192.168.10.11 node1.example.com
192.168.10.12 node2.example.com
192.168.10.13 node3.example.com
EOF
```

- 配置 ssh 免密登录

```bash
ssh-keygen -t rsa -N '' -q -f ~/.ssh/id_rsa
for i in `seq 2 3`;do
    ssh-copy-id -f root@node$i.example.com
done
```

> 请确认远程主机 SSH 服务开启了 `PermitRootLogin yes`。

- 修改主机名

```bash
for i in `seq 1 3`;do
    ssh root@node$i.example.com hostnamectl set-hostname node$i.example.com
    ssh root@node$i.example.com hostname
done
```

- 关闭防火墙

```bash
for i in `seq 1 3`;do
    ssh root@node$i.example.com systemctl stop firewalld
    ssh root@node$i.example.com systemctl disable firewalld
done
```

- 关闭 SELINUX

```bash
for i in `seq 1 3`;do
    ssh root@node$i.example.com sed -i "/SELINUX/ s,enforcing,disabled,g" /etc/selinux/config
    ssh root@node$i.example.com setenforce 0
done
```

## 安装软件

InnoDB Cluster 需要安装 MySQL Community Server、MySQL Router 和 MySQL Shell 是哪个软件包。

其中 `Server` 又依赖于 `Libs`、`Common`、`Client`。

CentOS 默认带有 MariaDB 相关包，与 MariaDB 存在冲突，所以还需要安装 `libs-compat` 解决冲突。

- `mysql-community-libs`: 客户端共享库
- `mysql-community-libs-compat`: MySQL 之前版本的共享兼容库
- `mysql-community-common`: 服务端和客户端的公共文件
- `mysql-community-client`: 客户端及相关工具

```bash
for i in `seq 1 3`;do
    ssh root@node$i.example.com yum install -y \
        http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-libs-8.0.19-1.el7.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-libs-compat-8.0.19-1.el7.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-common-8.0.19-1.el7.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-client-8.0.19-1.el6.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-server-8.0.19-1.el7.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-Router/mysql-router-community-8.0.19-1.el7.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-Shell/mysql-shell-8.0.19-1.el7.x86_64.rpm

    ssh root@node$i.example.com systemctl start mysqld
    ssh root@node$i.example.com systemctl enable mysqld
done
```

> [mysql 官方 repo](http://repo.mysql.com/mysql-community-release-el7.rpm)

## mysqlsh 示例

```bash
mysqlsh -? # 获取帮助
mysqlsh -- shell help # 获取 shell 相关帮助
mysqlsh -- util help # 获取 dba 相关帮助
mysqlsh -- dba help # 获取 dba 相关帮助
mysqlsh -p$passwd -- cluster help # 获取 cluster 相关帮助(cluster 比较特殊)
mysqlsh -i -e '\? cluster' # 获取 cluster 相关帮助
mysqlsh -i -e '\? cluster.addInstance' # 获取某个选项帮助
mysqlsh root@$HOSTNAME -p --sql # 连接到本地 mysql
mysqlsh root@$HOSTNAME -p --sql -e "show databases;" # 执行 sql 语句
```

> - `-i`: 提示
> - `-e`: 非交互式模式

> [MySQL Shell 8.0](https://dev.mysql.com/doc/mysql-shell/8.0/en/)

## 添加远程用户

三台主机修改 MySQL 默认密码，并创建远程管理员用户。

```bash
cat > ~/init_mysql_root_passwd.sh <<EOF
set -x
passwd=MyNewPass4!
dpasswd=\$(grep password /var/log/mysqld.log | awk '{print \$NF}')
mysqlsh -p\$dpasswd --sql -e "set password='\$passwd';create user root@'%' identified by '\$passwd';grant all on *.* to root@'%' with grant option;"
EOF

for i in `seq 1 3`;do
    scp ~/init_mysql_root_passwd.sh root@node$i.example.com:~/init_mysql_root_passwd.sh
    ssh root@node$i.example.com sh ~/init_mysql_root_passwd.sh
done
```

## 初始化实例

`dba.configureLocalInstance()` 会配置 Group Replication 并持久化配置。

```bash
passwd=MyNewPass4!
for i in `seq 1 3`;do
echo "Group Replication Init..."
    mysqlsh --nw -- dba configureLocalInstance root@node$i.example.com --password=$passwd --restart=true --clusterAdmin="'admin'@'%'" --clusterAdminPassword=$passwd
done

sleep 10s

for i in `seq 1 3`;do
    echo "Check Instance Configuration..."
    mysqlsh --nw -- dba checkInstanceConfiguration root@node$i.example.com --password=$passwd
done
```

- 创建 InnoDB Cluster

```bash
# 创建单主模式
mysqlsh --nw root@$HOSTNAME -p$passwd -- dba createCluster My_Innodb_Cluster

# 创建多主模式
# mysqlsh --nw root@$HOSTNAME -p"$passwd" -- dba createCluster My_Innodb_Cluster --multiPrimary --force

for i in `seq 2 3`;do
    echo "Cluster Add Instance..."
    mysqlsh root@$HOSTNAME -p$passwd -- cluster addInstance --recoveryMethod=clone root@node$i.example.com --password=$passwd
done

mysqlsh root@$HOSTNAME -p$passwd -- cluster status
```

`dba.createCluster('My_Innodb_Cluster')` 包含以下操作:

1. 在连接的实例上创建 mysql.mysql_innodb_cluster_metadata 存储元数据信息
2. 验证配置信息
3. 将此节点注册成 seed 节点
4. 创建必要的管理账号
5. 启动 Group Replication

> - `dba.configureInstance()`: 配置 MySQL InnoDB 群集的本地实例
> - `cluster.checkInstanceState()`: 查看是否可以添加为集群节点
> - `dba.createCluster()`: 创建集群
> - `dba.configureReplicaSetInstance()`: 验证并配置要在InnoDB中使用的实例复制集
> - `dba.createReplicaSet()`: 创建副本集
> - `dba.getReplicaSet()`: 获取副本集信息
> - `cluster.addInstance()`: 添加节点
> - `var cluster=dba.getCluster()`: 获取集群状态
> - `cluster.status()`: 查看集群状态
> - `cluster.removeInstance()`: 移除节点
> - `cluster.rejoinInstance()`: 重新加入节点
> - `cluster.describe()`: 获取集群结构信息
> - `Cluster.setOption()`: 
> - `Cluster.setInstanceOption()`: 
> - `Cluster.rescan()`: 扫描集群
> - `dba.dropMetadataSchema()`: 清空集群
> - `var cluster = dba.rebootClusterFromCompleteOutage()`: 关闭集群重启


- 配置 MySQL Router

官方推荐在每个应用程序所在机器上部署 Router，本机器连接本机器的 Router。

也可以搭配 Mycat、Keepalived、Pacemaker 等服务实现高可用。

```bash
echo $passwd|mysqlrouter -B root@$HOSTNAME --user=mysqlrouter --conf-use-gr-notifications
systemctl restart mysqlrouter
systemctl enable mysqlrouter
```

> - `--user=mysqlrouter`: 是使用 `mysqlrouter` 的系统用户。
> - `--conf-use-gr-notifications`: 是否启用 MGR 状态变化通知。

- 查看 Router 是否正常

```bash
for ((i=0;i<=5;i++));do
        echo "---------- RW node ----------"
        mysql -hnode1.example.com -P6446 -uroot -p$passwd  -e "select @@hostname;"
        echo "---------- RO Node ----------"
        mysql -hnode1.example.com -P6447 -uroot -p$passwd  -e "select @@hostname;"
done
```

## 模式切换

- 单主切换多主模式

```bash
mysqlsh root@$HOSTNAME -p$passwd -- cluster switchToMultiPrimaryMode
```

- 多主切换单主模式

```bash
mysqlsh root@$HOSTNAME -p$passwd -- cluster switchToSinglePrimaryMode $HOSTNAME
```

!> 最后的节点名选配，不指定将自动调配。

## 模拟故障恢复

- Secondary 故障恢复

```bash
# 停止 node2.example.com 节点 MySQL 服务
[root@node1 ~]# ssh node2.example.com systemctl stop mysqld
[root@node1 ~]# ssh node2.example.com systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since 日 2020-02-23 06:31:48 CST; 6min ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 4279 ExecStart=/usr/sbin/mysqld $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 4249 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 4279 (code=exited, status=0/SUCCESS)
   Status: "Server shutdown complete"

2月 23 06:31:06 node2.example.com systemd[1]: Starting MySQL Server...
2月 23 06:31:08 node2.example.com systemd[1]: Started MySQL Server.
2月 23 06:31:41 node2.example.com systemd[1]: Stopping MySQL Server...
2月 23 06:31:48 node2.example.com systemd[1]: Stopped MySQL Server.

# 此时 node2.example.com 节点已经处于 MISSING 状态
[root@node1 ~]# mysqlsh root@$HOSTNAME:6446 -p"$passwd" -- cluster status
WARNING: Using a password on the command line interface can be insecure.
{
    "clusterName": "My_Innodb_Cluster",
    "defaultReplicaSet": {
        "name": "default",
        "primary": "node1.example.com:3306",
        "ssl": "REQUIRED",
        "status": "OK_NO_TOLERANCE",
        "statusText": "Cluster is NOT tolerant to any failures. 1 member is not active",
        "topology": {
            "node1.example.com:3306": {
                "address": "node1.example.com:3306",
                "mode": "R/W",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            },
            "node2.example.com:3306": {
                "address": "node2.example.com:3306",
                "mode": "n/a",
                "readReplicas": {},
                "role": "HA",
                "shellConnectError": "MySQL Error 2003 (HY000): Can't connect to MySQL server on 'node2.example.com' (111)",
                "status": "(MISSING)"
            },
            "node3.example.com:3306": {
                "address": "node3.example.com:3306",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            }
        },
        "topologyMode": "Single-Primary"
    },
    "groupInformationSourceMember": "node1.example.com:3306"
}

# 查看 MGR 状态，自动剔除 node2.example.com 节点
[root@node1 ~]# mysqlsh root@$HOSTNAME:6446 -p"$passwd" --sql -e 'select * from performance_schema.replication_group_members;'
WARNING: Using a password on the command line interface can be insecure.
CHANNEL_NAME	MEMBER_ID	MEMBER_HOST	MEMBER_PORT	MEMBER_STATE	MEMBER_ROLE	MEMBER_VERSION
group_replication_applier	0049ff88-5575-11ea-b3f6-000c298deef7	node3.example.com	3306	ONLINE	SECONDARY	8.0.19
group_replication_applier	67f032ff-5573-11ea-b7b2-000c297ff54d	node1.example.com	3306	ONLINE	PRIMARY	8.0.19

# 恢复 node2.example.com 节点 MySQL 服务
[root@node1 ~]# ssh node2.example.com systemctl start mysqld
[root@node1 ~]# ssh node2.example.com systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: active (running) since 日 2020-02-23 06:42:36 CST; 23s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 4376 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 4404 (mysqld)
   Status: "Server is operational"
    Tasks: 47
   Memory: 514.7M
   CGroup: /system.slice/mysqld.service
           └─4404 /usr/sbin/mysqld

2月 23 06:42:35 node2.example.com systemd[1]: Starting MySQL Server...
2月 23 06:42:36 node2.example.com systemd[1]: Started MySQL Server.

# node1.example.com 节点已被自动加入集群
[root@node1 ~]# mysqlsh root@$HOSTNAME:6446 -p"$passwd" -- cluster status
WARNING: Using a password on the command line interface can be insecure.
{
    "clusterName": "My_Innodb_Cluster",
    "defaultReplicaSet": {
        "name": "default",
        "primary": "node1.example.com:3306",
        "ssl": "REQUIRED",
        "status": "OK",
        "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.",
        "topology": {
            "node1.example.com:3306": {
                "address": "node1.example.com:3306",
                "mode": "R/W",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            },
            "node2.example.com:3306": {
                "address": "node2.example.com:3306",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            },
            "node3.example.com:3306": {
                "address": "node3.example.com:3306",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            }
        },
        "topologyMode": "Single-Primary"
    },
    "groupInformationSourceMember": "node1.example.com:3306"
}
```

- Primary 故障恢复

```bash
# node1.example.com 是 Primary 节点
[root@node1 ~]# mysqlsh root@$HOSTNAME -p"$passwd" -- cluster status
WARNING: Using a password on the command line interface can be insecure.
{
    "clusterName": "My_Innodb_Cluster",
    "defaultReplicaSet": {
        "name": "default",
        "primary": "node1.example.com:3306",
        "ssl": "REQUIRED",
        "status": "OK",
        "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.",
        "topology": {
            "node1.example.com:3306": {
                "address": "node1.example.com:3306",
                "mode": "R/W",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            },
            "node2.example.com:3306": {
                "address": "node2.example.com:3306",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            },
            "node3.example.com:3306": {
                "address": "node3.example.com:3306",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            }
        },
        "topologyMode": "Single-Primary"
    },
    "groupInformationSourceMember": "node1.example.com:3306"
}

# 停止 node1.example.com 节点 MySQL 服务
[root@node1 ~]# ssh root@node1.example.com systemctl stop mysqld
[root@node1 ~]# ssh root@node1.example.com systemctl status mysqld
● mysqld.service - MySQL Server
   Loaded: loaded (/usr/lib/systemd/system/mysqld.service; enabled; vendor preset: disabled)
   Active: inactive (dead) since 日 2020-02-23 06:35:58 CST; 9s ago
     Docs: man:mysqld(8)
           http://dev.mysql.com/doc/refman/en/using-systemd.html
  Process: 2730 ExecStart=/usr/sbin/mysqld $MYSQLD_OPTS (code=exited, status=0/SUCCESS)
  Process: 2702 ExecStartPre=/usr/bin/mysqld_pre_systemd (code=exited, status=0/SUCCESS)
 Main PID: 2730 (code=exited, status=0/SUCCESS)
   Status: "Server shutdown complete"

2月 22 21:02:32 node1.example.com systemd[1]: Starting MySQL Server...
2月 22 21:02:35 node1.example.com systemd[1]: Started MySQL Server.
2月 23 06:35:52 node1.example.com systemd[1]: Stopping MySQL Server...
2月 23 06:35:58 node1.example.com systemd[1]: Stopped MySQL Server.

# 此时 node3.example.com 升级为 Primary 节点
[root@node1 ~]# mysqlsh root@$HOSTNAME:6446 -p"$passwd" -- cluster status
WARNING: Using a password on the command line interface can be insecure.
{
    "clusterName": "My_Innodb_Cluster",
    "defaultReplicaSet": {
        "name": "default",
        "primary": "node3.example.com:3306",
        "ssl": "REQUIRED",
        "status": "OK_NO_TOLERANCE",
        "statusText": "Cluster is NOT tolerant to any failures. 1 member is not active",
        "topology": {
            "node1.example.com:3306": {
                "address": "node1.example.com:3306",
                "mode": "n/a",
                "readReplicas": {},
                "role": "HA",
                "shellConnectError": "MySQL Error 2003 (HY000): Can't connect to MySQL server on 'node1.example.com' (111)",
                "status": "(MISSING)"
            },
            "node2.example.com:3306": {
                "address": "node2.example.com:3306",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            },
            "node3.example.com:3306": {
                "address": "node3.example.com:3306",
                "mode": "R/W",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            }
        },
        "topologyMode": "Single-Primary"
    },
    "groupInformationSourceMember": "node3.example.com:3306"
}
[root@node1 ~]# ssh root@node1.example.com systemctl start mysqld

# node1.example.com 节点已被自动加入集群，并调度为 Seconde 节点
[root@node1 ~]# mysqlsh root@$HOSTNAME:6446 -p"$passwd" -- cluster status
WARNING: Using a password on the command line interface can be insecure.
{
    "clusterName": "My_Innodb_Cluster",
    "defaultReplicaSet": {
        "name": "default",
        "primary": "node3.example.com:3306",
        "ssl": "REQUIRED",
        "status": "OK",
        "statusText": "Cluster is ONLINE and can tolerate up to ONE failure.",
        "topology": {
            "node1.example.com:3306": {
                "address": "node1.example.com:3306",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            },
            "node2.example.com:3306": {
                "address": "node2.example.com:3306",
                "mode": "R/O",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            },
            "node3.example.com:3306": {
                "address": "node3.example.com:3306",
                "mode": "R/W",
                "readReplicas": {},
                "replicationLag": null,
                "role": "HA",
                "status": "ONLINE",
                "version": "8.0.19"
            }
        },
        "topologyMode": "Single-Primary"
    },
    "groupInformationSourceMember": "node3.example.com:3306"
}
```

## 自动安装 

<!-- tabs:start -->

### ** 三节点单主模式 Shell 脚本 **

```bash
#!/usr/bin/env bash
set -euxo pipefail

red='\033[0;31m'
green='\033[0;32m'
yellow='\033[0;33m'
plain='\033[0m'

if [ `id -u` -ne 0 ];then
    echo -e "${green}Please use root login...${plain}"
    exit 1
fi

# 添加 IP 映射
cp -a /etc/hosts /etc/hosts-`date +%F`.bak
cat > /etc/hosts <<EOF
127.0.0.1   localhost localhost.localdomain localhost4 localhost4.localdomain4
::1         localhost localhost.localdomain localhost6 localhost6.localdomain6
192.168.10.11 node1.example.com
192.168.10.12 node2.example.com
192.168.10.13 node3.example.com
EOF

# 配置免密登录
if [ ! -f ~/.ssh/id_rsa ];then
    ssh-keygen -t rsa -N '' -q -f ~/.ssh/id_rsa
    for i in `seq 1 3`;do
        ssh-copy-id -f root@node$i.example.com
    done
else
    for i in `seq 1 3`;do
        ssh-copy-id -f root@node$i.example.com
    done
fi

# 修改主机名
for i in `seq 1 3`;do
    ssh root@node$i.example.com hostnamectl set-hostname node$i.example.com
    ssh root@node$i.example.com hostname
done

# 关闭防火墙
for i in `seq 1 3`;do
    ssh root@node$i.example.com systemctl stop firewalld
    ssh root@node$i.example.com systemctl disable firewalld
done

# 关闭 SELINUX
for i in `seq 1 3`;do
    getselinux=$(ssh root@node$i.example.com getenforce)
    if [[ $getselinux = "Enforcing" ]];then
        ssh root@node$i.example.com setenforce 0
        ssh root@node$i.example.com sed -i "/SELINUX/ s,enforcing,permissive,g" /etc/selinux/config
    fi
done

# 安装软件
for i in `seq 1 3`;do
    check_mysql=$(ssh root@node$i.example.com 'yum list installed|grep ^mysql|wc -l')
    if [ $check_mysql -ge "1" ];then
        ssh root@node$i.example.com yum remove -y mysql-* --skip-broken
        find / -name "mysql*" | xargs rm -rf {}
    fi
    echo -e "${green}node$i.example.com install mysql server...${plain}"
    ssh root@node$i.example.com yum install -y \
        http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-libs-8.0.19-1.el7.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-libs-compat-8.0.19-1.el7.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-common-8.0.19-1.el7.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-client-8.0.19-1.el6.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-8.0/mysql-community-server-8.0.19-1.el7.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-Router/mysql-router-community-8.0.19-1.el7.x86_64.rpm \
        http://mirrors.163.com/mysql/Downloads/MySQL-Shell/mysql-shell-8.0.19-1.el7.x86_64.rpm

    ssh root@node$i.example.com systemctl restart mysqld
    ssh root@node$i.example.com systemctl enable mysqld
done

# MySQL 添加远程用户
cat > ~/init_mysql_root_passwd.sh <<EOF
passwd=MyNewPass4!
dpasswd=`grep "temporary password" /var/log/mysqld.log | awk '{print \$NF}'`
mysqlsh -p"\$dpasswd" --sql -e "set password='\$passwd';create user root@'%' identified by '\$passwd';grant all on *.* to root@'%' with grant option;flush privileges;"
EOF

for i in `seq 1 3`;do
    scp ~/init_mysql_root_passwd.sh root@node$i.example.com:~/init_mysql_root_passwd.sh
    ssh root@node$i.example.com sh ~/init_mysql_root_passwd.sh
    if [ $? = 0 ];then
        echo -e "${green}MySQL default password is changed successfully !${plain}"
    else
        read -p "MySQL there is a password, please enter the password...": passwd
        ssh root@node$i.example.com 'sed -i "s|passwd=.*|passwd=$passwd|" ~/init_mysql_root_passwd.sh'
        ssh root@node$i.example.com sh ~/init_mysql_root_passwd.sh
    fi
done

# 初始化实例
for i in `seq 1 3`;do
echo -e "${green}Group Replication Init...${plain}"
    mysqlsh --nw -- dba configureLocalInstance root@node$i.example.com --password=$passwd --restart=true
done

sleep 10s

for i in `seq 1 3`;do
    echo -e "${green}Check Instance Configuration...${plain}"
    mysqlsh --nw -- dba checkInstanceConfiguration root@node$i.example.com --password=$passwd
done

# 创建 InnoDB Cluster
mysqlsh --nw root@$HOSTNAME -p$passwd -- dba createCluster My_Innodb_Cluster

# 判断 instance status ok
for i in `seq 2 3`;do
    echo -e "${green}Cluster Add Instance...${plain}"
    mysqlsh root@$HOSTNAME -p$passwd -- cluster addInstance --recoveryMethod=clone root@node$i.example.com --password=$passwd
done

mysqlsh root@$HOSTNAME -p$passwd -- cluster status

# 配置 MySQL Router
echo $passwd|mysqlrouter -B root@$HOSTNAME --user=mysqlrouter --conf-use-gr-notifications
systemctl restart mysqlrouter
systemctl enable mysqlrouter

# 查看 Router 是否正常
for ((i=0;i<=5;i++));do
        echo "${green}---------- RW node ----------${plain}"
        mysql -hnode1.example.com -P6446 -uroot -p$passwd  -e "select @@hostname;" 2> /dev/null
        echo "${green}---------- RO Node ----------${plain}"
        mysql -hnode1.example.com -P6447 -uroot -p$passwd  -e "select @@hostname;" 2> /dev/null
done

mysqlsh root@$HOSTNAME -p$passwd -- cluster listRouters

echo -e "${yellow}=========================================${plain}"
echo -e "${green} MySQL Root Password:${red} $(echo $passwd) ${plain}"
echo -e "${green} MySQL InnoDB Cluster Name:${red} $(mysqlsh root@$HOSTNAME -p$passwd -- cluster getName 2> /dev/null) ${plain}"
echo -e "${green} MySQL InnoDB Cluster RwPort:${red} $(mysqlsh root@$HOSTNAME -p$passwd -- cluster listRouters | awk -F ':|,' '/rwPort/ {print $2}') ${plain}"
echo -e "${green} MySQL InnoDB Cluster RoPort:${red} $(mysqlsh root@$HOSTNAME -p$passwd -- cluster listRouters | awk -F ':|,' '/roPort/ {print $2}') ${plain}"
echo -e "${yellow}=========================================${plain}"
```

### ** docker 部署 **

```bash

```

<!-- tabs:end -->
