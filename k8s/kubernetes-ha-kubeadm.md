# Kubernetes 高可用集群部署

## 环境准备

### 服务器说明

五台 CentOS 虚拟机:

|IP				      |节点	  |CPU    |Memory   |Hostname	|
|-				      |-		  |-	    |-		    |-			  |
|192.168.11.141 |master |>=2    |>=2G	    |m1		    |
|192.168.11.142 |master |>=2    |>=2G	    |m2		    |
|192.168.11.143 |master |>=2    |>=2G	    |m3		    |
|192.168.11.151 |worker |>=2    |>=2G	    |w1		    |
|192.168.11.152 |worker |>=2    |>=2G	    |w2		    |

- keepalived 提供 kube-apiserver 对外服务的 VIP;
- haproxy 监听 VIP，后端连接所有 kube-apiserver 实例，提供健康检查和负载均衡功能；
- 由于 keepalived 是一主多备运行模式，故至少两个节点安装 keepalived 和 haporxy。
- 注意：如果是云服务器（需要申请虚拟IP并绑定到服务器上，公有云不支持 keepalived 虚拟VIP）


### 主机名

每个节点主机名必须不一样,并且保证所有节点之间可以通过 hostname 互相访问。

```bash
hostnamectl set-hostname <节点名> --static
```

所有节点添加 hosts 记录

```bash
cat >> /etc/hosts <<end
192.168.11.141 m1
192.168.11.142 m2
192.168.11.143 m3
192.168.11.151 w1
192.168.11.152 w2
end
```

### 关闭并清理防火墙

```bash
systemctl stop firewalld && systemctl disable firewalld
iptables -F && iptables -X && iptables -F -t nat && iptables -X -t nat && iptables -P FORWARD ACCEPT
```

### 关闭交换分区

```bash
swapoff -a && sed -i '/swap/s/^\(.*\)$/#\1/' /etc/fstab
```

### 关闭SELINUX

```bash
setenforce 0
sed -i 's/^SELINUX=.*/SELINUX=disabled/g' /etc/selinux/config
```

### 添加系统路由

```bash
tee /etc/sysctl.d/Kubernetes.conf <<end
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
vm.swappiness=0
vm.overcommit_memory=1
vm.panic_on_oom=0
fs.inotify.max_user_watches=89100
end
modprobe br_netfilter
sysctl -p /etc/sysctl.d/Kubernetes.conf
```

### 安装Docker

```bash
yum update -y
yum install -y conntrack ipvsadm ipset jq sysstat curl iptables libseccomp device-mapper-persistent-data lvm2
curl -so /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
yum install -y docker-ce
systemctl daemon-reload
systemctl enable docker
systemctl restart docker
```

配置Docker的启动参数

```bash
cat > /etc/docker/daemon.json <<end
{
	"registry-mirrors": ["https://registry.aliyuncs.com"],
	"exec-opts": ["native.cgroupdriver=cgroupfs"]
}
end
systemctl daemon-reload
systemctl restart docker
```

- registry-mirrors: 设置 docker 镜像地址，可配置国内镜像加速
- exec-opts: 设置 cgroup driver (默认是 cgroupfs，不推荐设置 systemd)
- graph: 设置 docker 数据目录地址

### 安装Kubernetes脚手架

- kubeadm: 部署集群用的命令
- kubelet: 在集群中每台机器上都要运行的组件，负责管理 pod、容器的生命周期
- kubectl: 集群管理工具（可选，只要在控制集群的节点上安装即可）

```bash
cat > /etc/yum.repos.d/kubernetes.repo <<end
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg
       http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
end
yum install -y kubectl kubeadm kubelet
systemctl daemon-reload
systemctl enabled kubelet
systemctl restart kubelet
```

> 官方地址：http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64

### 配置kubelet

kubelet 的 cgroupdriver 默认为 systemd，如果上面没有设置 docker 的 exec-opts 为 systemd，这里就需要将 kubelet 的设置为 cgroupfs。

由于各自的系统配置不同，配置位置和内容都不相同

1. /etc/systemd/system/kubelet.service.d/10-kubeadm.conf(如果此配置存在的情况执行下面命令：)

```bash
sed -i "s/cgroup-driver=systemd/cgroup-driver=cgroupfs/g" /etc/systemd/system/kubelet.service.d/10-kubeadm.conf
systemctl enable kubelet && systemctl start kubelet
```
2. /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf(如果 1 中的配置不存在，则此配置应该存在，不需要做任何操)

## 部署高可用集群

### 安装Keepalived

> 至少两台 master 节点安装 keepalived，这里选择所有节点都部署 keepalived。

```bash
yum install -y keepalived
systemctl daemon-reload && systemctl enable keepalived && systemctl start keepalived
``` 

修改 keepalived 配置文件：

```bash
ssh root@m1 cat > /etc/keepalived/keepalived.conf <<end
! Configuration File for keepalived
global_defs {
  router_id keepalive-1  # 主机名,每个节点不同
}

vrrp_script check_apiserver { # 检测脚本。
  script "/etc/keepalived/check-apiserver.sh"
  interval 3
  weight -2
}

vrrp_instance VI-kube {
  state MASTER                # 主服务器
  interface ens32             # VIP 漂移到的网卡
  virtual_router_id 68        # 多个几点必须相同
  priority 100                # 优先级，备用服务器比主服务器低
  dont_track_primary
  advert_int 3
  virtual_ipaddress {
    192.168.11.188         # vip 虚拟 ip (同网段)
  }
  track_script {
      check_apiserver
  }
}
end

ssh root@m2 cat > /etc/keepalived/keepalived.conf <<end
! Configuration File for keepalived
global_defs {
  router_id keepalive-2  # 主机名,每个节点不同
}

vrrp_script check_apiserver { # 检测脚本。
  script "/etc/keepalived/check-apiserver.sh"
  interval 3
  weight -2
}

vrrp_instance VI-kube {
  state BACKUP                # 主服务器
  interface ens32             # VIP 漂移到的网卡
  virtual_router_id 68        # 多个几点必须相同
  priority 90                # 优先级，备用服务器比主服务器低
  dont_track_primary
  advert_int 3
  virtual_ipaddress {
    192.168.11.188         # vip 虚拟 ip (同网段)
  }
  track_script {
      check_apiserver
  }
}
end

ssh root@m3 cat > /etc/keepalived/keepalived.conf <<end
! Configuration File for keepalived
global_defs {
  router_id keepalive-3  # 主机名,每个节点不同
}

vrrp_script check_apiserver { # 检测脚本。
  script "/etc/keepalived/check-apiserver.sh"
  interval 3
  weight -2
}

vrrp_instance VI-kube {
  state BACKUP                # 主服务器
  interface ens32             # VIP 漂移到的网卡
  virtual_router_id 68        # 多个几点必须相同
  priority 80                # 优先级，备用服务器比主服务器低
  dont_track_primary
  advert_int 3
  virtual_ipaddress {
    192.168.11.188         # vip 虚拟 ip (同网段)
  }
  track_script {
      check_apiserver
  }
}
end

ssh root@m1 cat > /etc/keepalived/check_apiserver.sh <<end
#!/bin/sh

netstat -ntlp|grep 6443 || exit 1

end

ssh root@m2 cat > /etc/keepalived/check_apiserver.sh <<end
#!/bin/sh

netstat -ntlp|grep 6443 || exit 1

end

ssh root@m3 cat > /etc/keepalived/check_apiserver.sh <<end
#!/bin/sh

netstat -ntlp|grep 6443 || exit 1

end
ssh root@m1 systemctl restart keepalived
ssh root@m2 systemctl restart keepalived
ssh root@m3 systemctl restart keepalived
```

### 安装HAproxy

所有 master 节点安装 haproxy

```bash
ssh root@m1 "yum install -y haproxy && systemctl daemon-reload && systemctl enable haproxy && systemctl start haproxy"

ssh root@m2 "yum install -y haproxy && systemctl daemon-reload && systemctl enable haproxy && systemctl start haproxy"

ssh root@m3 "yum install -y haproxy && systemctl daemon-reload && systemctl enable haproxy && systemctl start haproxy"
```

修改 haproxy 配置文件：

```bash
cat > /etc/haproxy/haproxy.cfg <<end
#---------------------------------------------------------------------
# Example configuration for a possible web application.  See the
# full configuration options online.
#
#   http://haproxy.1wt.eu/download/1.4/doc/configuration.txt
#
#---------------------------------------------------------------------

#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    #
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    #
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2

    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon

    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats

#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000

#---------------------------------------------------------------------
# main frontend which proxys to the backends
#---------------------------------------------------------------------
frontend  kubernetes-apiserver
    mode tcp
    bind *:16443
    option tcplog
    default_backend             kubernetes-apiserver

#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend kubernetes-apiserver
    mode tcp
    balance     roundrobin
    server  m1 192.168.11.141:6443 check
    server  m2 192.168.11.142:6443 check
    server  m3 192.168.11.143:6443 check

# 监控界面
listen  admin
    bind        *:8888                #监控界面的访问的IP和端口
    mode        http
    stats uri   /admin                #URI相对地址
    stats realm Global\ statistics    #统计报告格式
    stats auth  admin:abc123456       #登陆帐户信息
end
scp /etc/haproxy/haproxy.cfg root@m2:/etc/haproxy/haproxy.cfg
scp /etc/haproxy/haproxy.cfg root@m3:/etc/haproxy/haproxy.cfg
ssh root@m1 systemctl restart haproxy
ssh root@m2 systemctl restart haproxy
ssh root@m3 systemctl restart haproxy
```

查看端口是否正常：

```bash
ssh root@m1 "ss -lnt | grep -E '16443|8888'"
ssh root@m2 "ss -lnt | grep -E '16443|8888'"
ssh root@m3 "ss -lnt | grep -E '16443|8888'"
```

### 部署第一个Master

```bash
cat > ~/kubeadm-config.yml <<end
apiVersion: kubeadm.k8s.io/v1beta1
kind: ClusterConfiguration
kubernetesVersion: v1.15.0
controlPlaneEndpoint: "192.168.11.188:6443"
networking:
    # This CIDR is a Calico default. Substitute or remove for your CNI provider.
    podSubnet: "172.22.0.0/16"
imageRepository: registry.aliyuncs.com/google_containers
end

kubeadm init --config=kubeadm-config.yml --experimental-upload-certs
mkdir $HOME/.kube
cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
chown $(id -u):$(id -g) $HOME/.kube/config
kubectl get pods --all-namespaces
```

### 部署网络插件Calico

```bash
mkdir -p /etc/kubernetes/addons
curl -o /etc/kubernetes/addons/calico-policy-only.yaml https://docs.projectcalico.org/v3.8/manifests/calico-policy-only.yaml
curl -o /etc/kubernetes/addons/calico-rbac-kdd.yaml https://docs.projectcalico.org/v3.8/manifests/rbac/rbac-kdd-calico.yaml
sed -ie "s?192.168.0.0?172.22.0.0?g" /etc/kubernetes/addons/calico-policy-only.yaml
kubectl apply -f /etc/kubernetes/addons/calico-policy-only.yaml
kubectl apply -f /etc/kubernetes/addons/calico-rbac-kdd.yaml
```

### 加入其它Master节点

```bash
ssh root@m2 "kubeadm join ... && mkdir -p ~/.kube && cp -i /etc/kubernetes/admin.conf ~/.kube/config && chown $(id -u):$(id -g) ~/.kube/config && kubectl get nodes"
ssh root@m2 "kubeadm join ... && mkdir -p ~/.kube && cp -i /etc/kubernetes/admin.conf ~/.kube/config && chown $(id -u):$(id -g) ~/.kube/config && kubectl get nodes"
```

1. 利用未过期的 token 添加节点。

查看 token 命令：

```bash
kubeadm token list
```

例如：

```bash
[root@m1 ~]# kubeadm token list|awk 'NR!=1{print $1}'
f9ubb1.0l3jfp2dxz4v20sy
iasnf5.zlav24b7q28ekoxy
w8ewfm.kl9c7slc1qqjqhby
```

加入 master 节点：

```bash
kubeadm join --token iasnf5.zlav24b7q28ekoxy \
  --discovery-token-unsafe-skip-ca-verification \
  --experimental-control-plane
```

- --discovery-token-unsafe-skip-ca-verification: 忽略 ca 校验
- --experimental-control-plane: 添加 master 节点

2. 重新生成 token 添加节点。

```bash
kubeadm token generate
kubeadm token create <generated-token> --print-join-command --ttl=24h
```

生成的 token：

```bash
kubeadm join 192.168.11.188:6443 --token iasnf5.zlav24b7q28ekoxy     --discovery-token-ca-cert-hash sha256:cb503a6e95702a8ba9ebc35861301126cd03da3d8666d21bebf640cc661545eb 
```

- --ttl=24: 表示这个 token 的有效期为 24 小时，初始化默认生成的 token 有效期也是 24 小时

加入 master 节点：

```bash
ssh root@m2 kubeadm join 192.168.11.188:6443 \
  --token iasnf5.zlav24b7q28ekoxy \
  --discovery-token-ca-cert-hash \
  sha256:cb503a6e95702a8ba9ebc35861301126cd03da3d8666d21bebf640cc661545eb \
  --experimental-control-plane

ssh root@m3 kubeadm join 192.168.11.188:6443 \
  --token iasnf5.zlav24b7q28ekoxy \
  --discovery-token-ca-cert-hash \
  sha256:cb503a6e95702a8ba9ebc35861301126cd03da3d8666d21bebf640cc661545eb \
  --experimental-control-plane

kubectl get nodes
```

### 加入worker节点

> worker 节点的 join 命令没有 `--experimental-control-plane` 参数。

```bash
ssh root@w1 kubeadm join --token iasnf5.zlav24b7q28ekoxy --discovery-token-unsafe-skip-ca-verification

ssh root@w2 kubeadm join --token iasnf5.zlav24b7q28ekoxy --discovery-token-unsafe-skip-ca-verification

kubectl get nodes
```

### 移除node节点

查看所有节点：

```bash
kubectl get nodes
```

移除指定节点：

```bash
kubectl drain w2 --delete-local-data --force --ignore-daemonsets
kubectl delete node w2
```

在 w2 节点执行：

```bash
kubeadm reset
```

## 部署Dashboard

```bash
curl -o /etc/kubernetes/addons/kubernetes-dashboard.yaml https://raw.githubusercontent.com/kubernetes/dashboard/master/aio/deploy/recommended/kubernetes-dashboard.yaml
sed 's/targetPort.*/&\n      nodePort: 30005/g' /etc/kubernetes/addons/kubernetes-dashboard.yaml
cat >> /etc/kubernetes/addons/kubernetes-dashboard.yaml <<end
  type: NodePort
end
kubectl apply -f /etc/kubernetes/addons/kubernetes-dashboard.yaml
```

查看服务运行情况

```bash
$ kubectl get deployment kubernetes-dashboard -n kube-system
$ kubectl --namespace kube-system get pods -o wide
$ kubectl get services kubernetes-dashboard -n kube-system
$ netstat -ntlp|grep 30005
```

从 1.7 开始，dashboard 只允许通过 https 访问，我们使用nodeport的方式暴露服务，可以使用 https://NodeIP:NodePort 地址访问 关于自定义证书 默认dashboard的证书是自动生成的，肯定是非安全的证书，如果大家有域名和对应的安全证书可以自己替换掉。使用安全的域名方式访问dashboard。 在dashboard-all.yaml中增加dashboard启动参数，可以指定证书文件，其中证书文件是通过secret注进来的。

- –tls-cert-file
- dashboard.cer
- –tls-key-file
- dashboard.key

### 登录Dashboard

Dashboard 默认只支持 token 认证，所以如果使用 KubeConfig 文件，需要在该文件中指定 token，我们这里使用token的方式登录

```bash
# 创建service account
kubectl create sa dashboard-admin -n kube-system

# 创建角色绑定关系
kubectl create clusterrolebinding dashboard-admin --clusterrole=cluster-admin --serviceaccount=kube-system:dashboard-admin
```

### 打印Dashboard的登录token

```bash
cat >> ~/.bashrc <<end
export KDT=$(kubectl describe secret -n kube-system `kubectl get secrets -n kube-system | grep dashboard-admin | awk '{print $1}'` | grep -E '^token' | awk '{print $2}')
alias kdt='echo $KDT'
end

kdt
```