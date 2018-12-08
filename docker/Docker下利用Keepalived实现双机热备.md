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
        auth_pass  123456
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


> 通过docker start node1 或者任何一个节点是启动不了的，原因是集群之前的同步机制造成的，启动任何一个节点，该节点都会去其它节点同步数据，其它节点仍处于宕机状态，所以该节点启动失败，这也是pxc集群的强一致性的表现，解决方式是，删除所有节点

```bash
docker rm node1 node2 node3 node4 node 5
```

和数据卷中的grastate.dat文件

```bash
rm -rf /var/lib/docker/volumes/v1/_data/grastate.dat
rm -rf /var/lib/docker/volumes/v2/_data/grastate.dat
rm -rf /var/lib/docker/volumes/v3/_data/grastate.dat
rm -rf /var/lib/docker/volumes/v4/_data/grastate.dat
rm -rf /var/lib/docker/volumes/v5/_data/grastate.dat
```

重新执行集群创建的命令即可，因为数据都在数据卷中，所有放心，集群重新启动都数据仍然都在.