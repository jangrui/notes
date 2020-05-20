# MySQL Galera Cluster

## Galera Cluster 特点

- `多主架构`: 真正的多点读写的集群，在任何时候读写数据都是最新的。
- `同步复制`: 集群不同节点之间数据同步，没有延迟，在数据库挂掉之后，数据不会丢失。
- `并发复制`: 从节点APPLY数据时，支持并行执行有更好的性能。
- `故障切换`: 数据库故障时，因为支持多点写入，切换容易。
- `热插拔`: 在服务期间，如果数据库挂了，只要监控程序发现的够快，不可服务的时间就会非常少。在节点故障期间，节点本身对集群的影响非常小。
- `自动节点克隆`: 在新增节点，或者停机维护时，增量数据或者基础数据不需要人工手动备份提供，Galera Cluster会自动拉取在线节点数据，最终集群会变为一致。
- `对应用透明`: 集群的维护，对应用程序是透明的。

## 实验环境

|ip|hostname|
|-|-|
|192.168.10.11|node1.example.com|
|192.168.10.12|node2.example.com|
|192.168.10.13|node3.example.com|

## 初始化

[CentOS 7 初始化](https://notes.jangrui.com/#/?id=centos-7-%e5%88%9d%e5%a7%8b%e5%8c%96)

```bash
cat >> /etc/hosts <<EOF
192.168.10.11 node1.example.com
192.168.10.12 node2.example.com
192.168.10.13 node3.example.com
EOF
```

> 虚拟机环境手动实现三台机器域名访问。

## 构建镜像

```bash
cat > ./Dockerfile <<EOF
FROM percona/percona-xtradb-cluster

USER root
RUN ln -sf /usr/share/zoneinfo/Asia/Shanghai /etc/localtime

USER mysql
EOF

docker build -t jangrui/pxc .
```

## 部署

- node1

```bash
docker run -dit --rm --net host \
-e MYSQL_ROOT_PASSWORD=password \
-e XTRABACKUP_PASSWORD=password \
-e CLUSTER_NAME=mypxc \
-v dbdata:/var/lib/mysql \
-v dblogs:/var/log/mysql \
--name pxc \
jangrui/pxc \
--character-set-server=utf8mb4
```

- node2

```bash
docker run -dit --rm --net host \
-e MYSQL_ROOT_PASSWORD=password \
-e XTRABACKUP_PASSWORD=password \
-e CLUSTER_NAME=mypxc \
-e CLUSTER_JOIN=node1.example.com \
-v dbdata:/var/lib/mysql \
-v dblogs:/var/log/mysql \
--name pxc \
jangrui/pxc \
--character-set-server=utf8mb4
```

- node3

```bash
docker run -dit --rm --net host \
-e MYSQL_ROOT_PASSWORD=password \
-e XTRABACKUP_PASSWORD=password \
-e CLUSTER_NAME=mypxc \
-e CLUSTER_JOIN=node1.example.com \
-v dbdata:/var/lib/mysql \
-v dblogs:/var/log/mysql \
--name pxc \
jangrui/pxc \
--character-set-server=utf8mb4
```

## 状态信息

- 状态信息

```bash
docker exec -it pxc mysql -uroot -ppassword -e "show status like 'wsrep%';"
```

- 队列

```bash
docker exec -it pxc mysql -uroot -ppassword -e "show status like '%queue%'";
```

| 参数 | 描述 |
|:--------------------------------:|:-----:|
| wsrep_local_send_queue           | 发送队列的长度 |
| wsrep_local_send_queue_max       | 发送队列的最大长度 |
| wsrep_local_send_queue_min       | 发送队列的最小长度 |
| wsrep_local_send_queue_avg       | 发送队列的平均长度 |
| wsrep_local_recv_queue           | 接收队列的长度 |
| wsrep_local_recv_queue_max       | 接收队列的最大长度 |
| wsrep_local_recv_queue_min       | 接收队列的最小长度 |
| wsrep_local_recv_queue_avg       | 接收队列的平均长度 |


- 复制

| 参数 | 描述 |
|:--------------------------------:|:-----:|
| wsrep_last_applied               | 同步应用次数|
| wsrep_last_committed             | 事务提交次数|
| wsrep_replicated                 | 被其它节点复制的总数|
| wsrep_replicated_bytes           | 被其它节点复制的数据总数|
| wsrep_received                   | 从其它节点收到的写入请求总数|
| wsrep_received_bytes             | 从其它节点收到的同步数据总数|

- 流控

```bash
docker exec -it node1 mysql -uroot -ppassword -e "show status like '%flow%'";
```

| 参数 | 描述 |
|:--------------------------------:|:-----:|
| wsrep_flow_control_paused_ns     | 流控暂停状态下花费的总时间（纳秒）|
| wsrep_flow_control_paused        | 流控暂停时间的占比（0~1）|
| wsrep_flow_control_sent          | 发送的流控暂停事件的数量 |
| wsrep_flow_control_recv          | 接收的流控暂停事件的数量 |
| wsrep_flow_control_interval      | 流控的下限和上限。上限是队列中允许的最大请求数。如果队列达到上限，则拒绝新的请求。当处理现有请求时，队列会减少，一旦达到下限，将再次允许新的请求 |
| wsrep_flow_control_interval_low  | 停止流量控制的下限 |
| wsrep_flow_control_interval_high | 触发流量控制的上限 |
| wsrep_flow_control_status        | 流控状态 |

- 事务

| 参数 | 描述 |
|:--------------------------------:|:-----:|
| wsrep_cert_deps_distance         | 事务执行并发数 |
| wsrep_apply_oooe                 | 接收队列中事务的占比 |
| wsrep_apply_oool                 | 接收队列中事务乱序执行的频率 |
| wsrep_apply_window               | 接收队列中事务的平均数量 |
| wsrep_commit_oooe                | 发送事务中事务的占比 |
| wsrep_commit_oool                | 无任何意义（不存在本地乱序提交）|
| wsrep_commit_window              | 发送队列中事务的平均数量 |

- 状态

| 参数 | 描述 |
|:--------------------------------:|:-----:|
| wsrep_local_state_comment        | 节点状态 |
| wsrep_incoming_addresses         | 集群节点 ip 地址 |
| wsrep_desync_count               | 延时节点数量 |
| wsrep_cluster_size               | 节点数量 |
| wsrep_cluster_status             | 集群状态（PRIMARY、NON_PRIMARY、Disconnected）|
| wsrep_connected                  | 节点是否连接到集群 |
| wsrep_ready                      | 集群是否正常工作 |

## 故障重启

- 普通故障

```bash
# node1.example.com
docker stop pxc

docker run -dit --rm -net host \
-e MYSQL_ROOT_PASSWORD=password \
-e XTRABACKUP_PASSWORD=password \
-e CLUSTER_NAME=mypxc \
-e CLUSTER_JOIN=node2.example.com \
-v dbdata:/var/lib/mysql \
-v dblogs:/var/log/mysql \
--name pxc \
jangrui/pxc \
--character-set-server=utf8mb4
```

> 以最后一个退出节点作为为主节点启动

- 所有节点同时意外退出


```bash
# node1.example.com
sed -i "/safe_to_bootstrap/ s,0,1," /var/lib/docker/volumes/dbdata/_data/grastate.dat

docker run -dit --rm -net host \
-e MYSQL_ROOT_PASSWORD=password \
-e XTRABACKUP_PASSWORD=password \
-e CLUSTER_NAME=mypxc \
-v dbdata:/var/lib/mysql \
-v dblogs:/var/log/mysql \
--name pxc \
jangrui/pxc \
--character-set-server=utf8mb4
```


```bash
# node2.example.com && node3.example.com
docker run -dit --rm --net host \
-e MYSQL_ROOT_PASSWORD=password \
-e XTRABACKUP_PASSWORD=password \
-e CLUSTER_NAME=mypxc \
-e CLUSTER_JOIN=node1.example.com \
-v dbdata:/var/lib/mysql \
-v dblogs:/var/log/mysql \
--name pxc \
jangrui/pxc \
--character-set-server=utf8mb4
```

> 修改任意节点配置文件，作为主节点启动，其它节点正常上线。
