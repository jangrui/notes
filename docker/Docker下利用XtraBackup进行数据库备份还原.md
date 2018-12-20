# Docker下利用XtraBackup进行数据库备份还原

由于之前启动pxc集群时没有指定备份目录，这里需要删除某个节点然后指定一个用于备份的目录。

```bash
docker stop noe1 && docker rm node1 
docker volume create backup
docker run -d \
	-e MYSQL_ROOT_PASSWORD=abc123456 \
	-e CLUSTER_NAME=PXC \
	-e XTRABACKUP_PASSWORD=abc123456 \
	-e CLUSTER_JOIN=node2 \
	-p 3360:3306 \
	-v backup:/data \
	-v v1:/var/lib/mysql \
	--privileged \
	--net=net1
	--name=node1
	pxc
```

> 由于最新版的pxc镜像不支持安装软件，需要用老版本（5.7.21）创建pxc集群。
>
> docker pull docker.io/percona/percona-xtradb-cluster:5.7.21

## 热备份数据

> 数据库可以热备份，但不能热还原。
>
> 为了避免恢复过程中的数据同步，采用空白的Mysql还原数据，然后再建PXC集群。 

```bash
#进入node1容器
docker exec -it node1 bash
#更新软件包
apt-get update
#安装热备工具
apt-get install percona-xtrabackup-24
#全量热备
innobackupex --user=root --password=abc123456 /data/backup/full
```

冷还原数据 停止其余4个节点，并删除节点

```bash
docker stop node2
docker stop node3
docker stop node4
docker stop node5
docker rm node2
docker rm node3
docker rm node4
docker rm node5
```

node1容器中删除MySQL的数据

```bash 
# 删除数据
rm -rf /var/lib/mysql/*
# 清空事务
innobackupex --user=root --password=abc123456 --apply-back /data/backup/full/2018-04-15_05-09-07/
# 还原数据
innobackupex --user=root --password=abc123456 --copy-back  /data/backup/full/2018-04-15_05-09-07/
```

> --apply-back # 事务回滚
> 
> --copy-back # 还原

重新创建其余4个节点，组建PXC集群。