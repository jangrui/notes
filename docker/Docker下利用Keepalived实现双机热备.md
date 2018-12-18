# Docker 下利用 Keepalived 实现双机热备

## 安装 keepalived

进入 haproxy 容器,更新源后安装 keepalived

```bash
docker exec -it haproxy1 bash
apt-get update
apt-get install -y keepalived
```

## 配置 keepalived

```bash
vim /etc/keepalived/keepalived.conf

vrrp_instance  VI_1 {
    state  MASTER
    interface  eth0
    virtual_router_id  51
    priority  100
    advert_int  1
    authentication {
        auth_type  PASS
        auth_pass  abc123456
    }
    virtual_ipaddress {
        172.18.0.201
    }
}
```

启动 keepalived

```bash
service keepalived start
```

在宿主机里测试keepalived是否启动

```bash
ping 172.18.0.201
```

### 宿主机安装配置 keepalived

```bash
yum install -y keepalived

ip addr

vim /etc/keepalived/keepalived.conf

```

keepalived 配置参数:

```bash
vrrp_instance VI_1 {
    state MASTER
    interface ens33
    # 绑定网卡
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass 1111
    }
    virtual_ipaddress {
       	192.168.41.170
       	# 指定一个虚拟 ip
    }
}


virtual_server 192.168.41.170 8888 {
    delay_loop 3
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 172.18.0.201 8888 {
        weight 1
    }
}


virtual_server 192.168.41.170 3306 {
    delay_loop 3
    lb_algo rr 
    lb_kind NAT
    persistence_timeout 50
    protocol TCP

    real_server 172.18.0.201 3306 {
        weight 1
    }
}

```

重启 keepalived

```bash
systemctl restart keepalived
systemctl enable keepalived	
```

## 检测

停止某个haproxy节点

```bash
docker pause haproxy1
```

重启haproxy1节点

```bash
docker unpause haproxy1
```

> stop 指令停止容器，重新启动容器需要进入容器加载haproxy配置文件和启动keepalived服务。

通过测试证实集群可正常运行，实现集群负载均衡高可用方案。