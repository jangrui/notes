# Docker 下部署 RedisCluster 集群

> RedisCluster 无中心节点，客户端与redis节点直连，不需要中间代理层；
>
> 数据可以被分片存储，可配置冗余节点；
>
> 管理方便，可自行增添或摘除节点；
>
> Redis 集群中应包含奇数个Mater节点，至少应该有3个Master即节点；
>
> Redis 集群中每个Master节点都应该有Slave节点；

## 安装Redis镜像

```bash
docker pull racccosta/redis:3.2.8
```

## 创建redis-net网段

```bash
docker network create --subnet=172.19.0.0/16 redis-net
```

## 创建6节点Redis容器

```bash
docker run -d -p 6379:6379 --name redis-c1 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf
docker run -d -p 6380:6379 --name redis-c2 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf
docker run -d -p 6381:6379 --name redis-c3 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf
docker run -d -p 6382:6379 --name redis-c4 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf
docker run -d -p 6383:6379 --name redis-c5 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf
docker run -d -p 6384:6379 --name redis-c6 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf

# 建立集群关系
docker exec -it redis-c1 /usr/redis/redis-trib.rb create --replicas 1 172.19.0.2:6379 172.19.0.3:6379 172.19.0.4:6379 172.19.0.5:6379 172.19.0.6:6379 172.19.0.7:6379
# 查看集群节点
docker exec -it redis-c1 redis-cli -c cluster nodes
```

## Redis集群的几种模式

### 以独立模式运行

```bash
docker run -d -p 6379:6379 --name redis racccosta/redis:3.2.8
```

### 组建具有2个从属的主服务器（主从）

```bash
docker network create redis-net
docker run -d -p 6379:6379 --name redis    --net redis-net racccosta/redis:3.2.8
docker run -d -p 6380:6379 --name redis-s1 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis.conf --slaveof redis 6379
docker run -d -p 6381:6379 --name redis-s2 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis.conf --slaveof redis 6379
```

### 组建包含3个主节点和3个从节点群集（分片）

```bash
docker network create redis-net
docker run -d -p 6379:6379 --name redis-c1 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf
docker run -d -p 6380:6379 --name redis-c2 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf
docker run -d -p 6381:6379 --name redis-c3 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf
docker run -d -p 6382:6379 --name redis-c4 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf
docker run -d -p 6383:6379 --name redis-c5 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf
docker run -d -p 6384:6379 --name redis-c6 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis-cluster.conf

docker exec -it redis-c1 /usr/redis/redis-trib.rb create --replicas 1 172.18.0.2:6379 172.18.0.3:6379 172.18.0.4:6379 172.18.0.5:6379 172.18.0.6:6379 172.18.0.7:6379
```

### 组建包含2个从属（主从）节点和3个Sentinels（高可用）节点集群

```bash
docker network create redis-net
docker run -d -p 6379:6379 --name redis    --net redis-net racccosta/redis:3.2.8
docker run -d -p 6380:6379 --name redis-s1 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis.conf --slaveof redis 6379
docker run -d -p 6381:6379 --name redis-s2 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/redis.conf --slaveof redis 6379

docker run -d -p 26379:26379 --name sentinel1 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/sentinel.conf --sentinel
docker run -d -p 26380:26379 --name sentinel2 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/sentinel.conf --sentinel
docker run -d -p 26381:26379 --name sentinel3 --net redis-net racccosta/redis:3.2.8 redis-server /usr/redis/sentinel.conf --sentinel

docker exec sentinel1 redis-cli SENTINEL MONITOR redis 172.18.0.2 6379 2
docker exec sentinel2 redis-cli SENTINEL MONITOR redis 172.18.0.2 6379 2
docker exec sentinel3 redis-cli SENTINEL MONITOR redis 172.18.0.2 6379 2

docker exec sentinel1 redis-cli SENTINEL SET redis down-after-milliseconds 3000
docker exec sentinel2 redis-cli SENTINEL SET redis down-after-milliseconds 3000
docker exec sentinel3 redis-cli SENTINEL SET redis down-after-milliseconds 3000

docker exec sentinel1 redis-cli SENTINEL SET redis failover-timeout 10000
docker exec sentinel2 redis-cli SENTINEL SET redis failover-timeout 10000
docker exec sentinel3 redis-cli SENTINEL SET redis failover-timeout 10000

docker exec sentinel1 redis-cli SENTINEL SET redis parallel-syncs 5
docker exec sentinel2 redis-cli SENTINEL SET redis parallel-syncs 5
docker exec sentinel3 redis-cli SENTINEL SET redis parallel-syncs 5
```

## 测试

### 测试独立模式

```bash
docker exec redis redis-cli -c set 1 "Redis is ok"
docker exec redis redis-cli -c get 1
```

### 测试主从模式

```bash
docker exec redis redis-cli -c set 1 "Redis is OK on Master"
docker exec redis    redis-cli -c get 1
docker exec redis-s1 redis-cli -c get 1
docker exec redis-s2 redis-cli -c get 1
```

### 测试高可用节点

```bash
docker exec redis-c1 redis-cli -c set 1 "Redis is OK on Cluster"
```

### redis.conf

```bash
bind 0.0.0.0
protected-mode yes
port 6379
tcp-backlog 511
timeout 0
tcp-keepalive 300
daemonize yes # 以后台进程运行
supervised no
pidfile /var/run/redis_6379.pid
loglevel notice
logfile ""
databases 16
always-show-logo yes
save 900 1
save 300 10
save 60 10000
stop-writes-on-bgsave-error yes
rdbcompression yes
rdbchecksum yes
dbfilename dump.rdb
dir ./
replica-serve-stale-data yes
replica-read-only yes
repl-diskless-sync no
repl-diskless-sync-delay 5
repl-disable-tcp-nodelay no
replica-priority 100
lazyfree-lazy-eviction no
lazyfree-lazy-expire no
lazyfree-lazy-server-del no
replica-lazy-flush no
appendonly yes # 开启AOF持久化模式
appendfilename "appendonly.aof"
appendfsync everysec
no-appendfsync-on-rewrite no
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
aof-load-truncated yes
aof-use-rdb-preamble yes
lua-time-limit 5000
slowlog-log-slower-than 10000
slowlog-max-len 128
latency-monitor-threshold 0
notify-keyspace-events ""
hash-max-ziplist-entries 512
hash-max-ziplist-value 64
list-max-ziplist-size -2
list-compress-depth 0
set-max-intset-entries 512
zset-max-ziplist-entries 128
zset-max-ziplist-value 64
hll-sparse-max-bytes 3000
stream-node-max-bytes 4096
stream-node-max-entries 100
activerehashing yes
client-output-buffer-limit normal 0 0 0
client-output-buffer-limit replica 256mb 64mb 60
client-output-buffer-limit pubsub 32mb 8mb 60
hz 10
dynamic-hz yes
aof-rewrite-incremental-fsync yes
rdb-save-incremental-fsync yes
cluster-enabled yes # 开启集群
cluster-config-file nodes-6379.conf # 集群配置文件
cluster-node-timeout 15000 #超时时间
```

> 注意：redis配置文件里必须要设置bind 0.0.0.0，这是允许其他IP可以访问当前redis。如果不设置这个参数，就不能组建Redis集群。
