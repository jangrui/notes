# Kubernetes 高可用集群部署

## 环境准备

### 服务器说明

五台 CentOS 虚拟机:

|IP				|节点	|CPU |Memory |Hostname	|
|-				|-		|-	 |-		 |-			|
|192.168.11.141 |master |>=2 |>=2G	 |m1		|
|192.168.11.142 |master |>=2 |>=2G	 |m2		|
|192.168.11.143 |master |>=2 |>=2G	 |m3		|
|192.168.11.151 |worker |>=2 |>=2G	 |w1		|
|192.168.11.152 |worker |>=2 |>=2G	 |w2		|

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
	"registry-mirrors": ["https://registry.aliyun.com"],
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

> 任选两个 master 节点部署 keepalived

```bash
yum install -y keepalived
systemctl daemon-reload && systemctl enable keepalived && systemctl start keepalived
``` 

修改 keepalived 配置文件：

```bash
ssh root@m1 cat > /etc/keepalived/keepalived.conf <<end
! Configuration File for keepalived
global_defs {
 router_id keepalive-master # 负载均衡名
}

vrrp_script check_apiserver { # 检测脚本。
 script "/etc/keepalived/check-apiserver.sh"
 interval 3
 weight -2
}

vrrp_instance VI-kube-master {
   state MASTER 		# 主服务
   interface ens32 		# 网卡
   virtual_router_id 68
   priority 100
   dont_track_primary
   advert_int 3
   virtual_ipaddress {
     192.168.11.188 	# 虚拟 ip (同网段)
   }
   track_script {
       check_apiserver
   }
}
end

ssh root@m2 cat > /etc/keepalived/keepalived.conf <<end
! Configuration File for keepalived
global_defs {
 router_id keepalive-backup # 负载均衡名
}

vrrp_script check_apiserver { # 检测脚本
 script "/etc/keepalived/check-apiserver.sh"
 interval 3
 weight -2	# 权重-2
}

vrrp_instance VI-kube-master {
   state BACKUP 		# 从服务
   interface ens32  	# 网卡
   virtual_router_id 68
   priority 99
   dont_track_primary
   advert_int 3
   virtual_ipaddress {
     192.168.11.188 	# 虚拟 ip (同网段)
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
kubeadm join ...
kubectl get nodes
```

### 加入worker节点

```bash
kubeadm join ...
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

# 打印dashboard-admin-secret的token
cat >> ~/.bashrc <<end
export KDT=$(kubectl describe secret -n kube-system `kubectl get secrets -n kube-system | grep dashboard-admin | awk '{print $1}'` | grep -E '^token' | awk '{print $2}')
alias kdt='echo $KDT'
end

kdt
```