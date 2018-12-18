# Docker下pxc集群数据热备份

## 热备份数据

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

重新创建其余4个节点，组件PXC集群