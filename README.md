# CentOS 7 初始化

![Linux](./_media/linux.png "linux.png")

<!-- tabs:start -->

## ** 手动初始化 **

- 只允许 wheel 组用户切换 root

```bash
sudo sh -c 'echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su'
sudo sh -c 'echo "SU_WHEEL_ONLY yes" >> /etc/login.defs'
sudo usermod -aG wheel $USER
```

- 普通用户无密码验证

```bash
sudo sh -c 'echo "%wheel ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers'
```

- sudo 提示找不到命令

```bash
sudo sed -i 's,env_reset,!&,' /etc/sudoers
echo "alias sudo='sudo env PATH=$PATH'" >> ~/.bashrc
source ~/.bashrc
```

- 关闭防火墙

```bash
sudo systemctl stop firewalld
sudo systemctl disable firewalld
```

- 关闭 SELINUX

```bash
sudo sed -i "/SELINUX/ s,enforcing,disabled,g" /etc/selinux/config
sudo setenforce 0
```

- 更换国内镜像

```bash
sudo curl -So /etc/yum.repos.d/Centos-Base.repo https://mirrors.aliyun.com/repo/Centos-7.repo
sudo yum makecache fast
```

- 安装 epel 源

```bash
sudo curl -So /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
sudo yum makecache fast
```

- 更新系统

```bash
sudo yum update -y
```

- 安装命令补全

```bash
sudo yum install -y bash-completion
source /etc/profile.d/bash_completion.sh
```

- 安装新版内核

```bash
sudo yum install -y screen
screen -S kernel
sudo sh -c '
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm
yum makecache fast
if [ `rpm -qa|grep ^kernel-headers|wc -l` -ge 1 ];then
    yum remove -y kernel-headers
fi
yum --enablerepo elrepo-kernel install -y kernel-ml kernel-ml-devel kernel-ml-headers
yum group remove -y "Development Tools"
yum group install -y "Development Tools"
grub2-set-default 0
'
```

> 删除 kernel-headers 会自动删除 gcc gcc-c++ 等依赖

- 调整 swap 分区

> [对于 Red Hat 平台，推荐设置的 SWAP 交换空间大小为多少？](https://access.redhat.com/zh_CN/solutions/881023)

|RAM 大小|SWAP 大小|如果允许休眠 SWAP 大小|
|:-:|:-:|:-:|
|2GB 或更少|2倍的 RAM 大小|3倍的 RAM 大小|
|2GB - 8GB	|与 RAM 大小相同|2倍的 RAM 大小|
|8GB - 64GB |至少 4GB|1.5倍的 RAM 大小|
|64GB 或更多 |至少 4GB|不推荐休眠|

```bash
mem=$(free -m|sed '1d'|awk '/Mem/{print $2}')
swap=`expr $mem / 2`
# 创建 swap 文件
if [ $mem -le 2048 ];then
    dd if=/dev/zero of=/tmp/swap bs=${swap}M count=4
elif [ $mem -gt 2048 && $mem -le 8192 ];then
    dd if=/dev/zero of=/tmp/swap bs=${swap}M count=2
else
    dd if=/dev/zero of=/tmp/swap bs=4G count=4
fi
sudo chown 0:0 /tmp/swap
sudo chmod 0600 /tmp/swap
# 格式化 swap 文件
sudo mkswap /tmp/swap
# 开机自动挂载
swapoff -a
sed -i "/swap/ s|^\(.*\)$|#\1|g" /etc/fstab
sudo sh -c 'echo "/tmp/swap swap swap defaults 0 0" >> /etc/fstab'
sudo swapon -a
```

- 调整内核

```bash
mem=$(free -m|sed '1d'|awk '/Mem/{print $2}')
shmmax=$(awk -v m=$mem 'BEGIN{printf("%.f\n",m*1024*1024*0.9)}')
shmall=$(awk -v m=$mem 'BEGIN{printf("%.f\n",m*1024*0.9/4)}')
grep -q "^kernel.shmall" /etc/sysctl.conf && sed -i "s,^kernel.shmmax.*,kernel.shmmax = $shmmax," /etc/sysctl.conf || echo "kernel.shmmax = $shmmax" >> /etc/sysctl.conf
grep -q "^kernel.shmall" /etc/sysctl.conf && sed -i "s,^kernel.shmall.*,kernel.shmall = $shmall," /etc/sysctl.conf || echo "kernel.shmall = $shmall" >> /etc/sysctl.conf
grep -q "^kernel.msgmax" /etc/sysctl.conf && sed -i "s,^kernel.msgmax.*,kernel.msgmax = 65535," /etc/sysctl.conf || echo "kernel.msgmax = 65535" >> /etc/sysctl.conf
grep -q "^kernel.msgmnb" /etc/sysctl.conf && sed -i "s,^kernel.msgmnb.*,kernel.msgmnb = 65535," /etc/sysctl.conf || echo "kernel.msgmnb = 65535" >> /etc/sysctl.conf
grep -q "^vm.swappiness" /etc/sysctl.conf && sed -i "s,^vm.swappiness.*,vm.swappiness = 30," /etc/sysctl.conf || echo "vm.swappiness = 30" >> /etc/sysctl.conf
grep -q "^fs.file-max" /etc/sysctl.conf && sed -i "s,^fs.file-max.*,fs.file-max = 6553560," /etc/sysctl.conf || echo "fs.file-max = 6553560" >> /etc/sysctl.conf
sysctl -p

sudo sysctl -p
```

> kernel.shmmax: 单个共享内存段的最大值；例如 4G RAM: `4*1024*1024*1024*0.9=3865470566`
>
> kernel.shmall: 共享内存总量；例如 4G RAM: `4*1024*1024*1024*0.9/4/1024=943718`
> 
> [参考：调整虚拟内存](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/6/html/performance_tuning_guide/s-memory-tunables)
> 
> [参考：配置系统内存容量](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/performance_tuning_guide/sect-red_hat_enterprise_linux-performance_tuning_guide-memory-configuration_tools#sect-Red_Hat_Enterprise_Linux-Performance_Tuning_Guide-Configuration_tools-Configuring_system_memory_capacity)

- 安装 docker

```bash
sudo yum remove -y docker*

sudo curl -So /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum makecache fast

sudo yum install -y lvm2 device-mapper-persistent-data docker-ce

sudo systemctl start docker
sudo systemctl enable docker

sudo sh -c 'cat <<EOF> /etc/docker/daemon.json
{
    "registry-mirrors": ["https://dockerhub.azk8s.cn"],
    "exec-opts": ["native.cgroupdriver=systemd"],
    "log-driver" "json-file",
    "log-opts": {
        "max-size": "100m"
    }
}
EOF'

sudo sh -c '
grep -q "^net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
grep -q "^net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
grep -q "^net.bridge.bridge-nf-call-iptables" /etc/sysctl.conf || echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
grep -q "^net.bridge.bridge-nf-call-ip6tables" /etc/sysctl.conf || echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
'

sudo systemctl restart docker

sudo usermod -aG docker $username

docker info
```

- 安装 docker-compose

```bash
sudo yum install -y python3-pip

sudo pip3 install -U pip -i https://pypi.douban.com/simple
sudo pip install docker-compose

docker-compose -v
```

- docker-compose 命令补全

```
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.25.5/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
source /etc/bash_completion.d/docker-compose
```

## ** 快速初始化 **

```bash
sudo sh -c 'curl -L https://www.jangrui.com/centos-init.sh|bash'
```

脚本内容：

```bash
#!/usr/bin/env bash
# CentOS 6/7/8 初始化
# jangrui <admin@jangrui.com>

# set -euxo pipefail

if [ `id -u` -ne 0 ];then
    echo "Please use root login."
    exit 1
fi

# 添加用户
read -p "Please input your username": username
if [ -z "$username" ];then
	echo "不添加用户"
elif [ `grep "$username" /etc/passwd|wc -l` -eq 0 ];then
    useradd "$username"
    read -p "Please input your passwd": passwd
    echo "$passwd" | passwd "$username" --stdin
else
    echo "$username is Already"
fi

# 只允许 wheel 组用户切换 root
if [ `grep -E "^auth.*pam_wheel.so" /etc/pam.d/su|wc -l` -eq 0 ];then
    echo "auth required pam_wheel.so use_uid" >> /etc/pam.d/su
fi

if [ `grep "SU_WHEEL_ONLY yes" /etc/login.defs|wc -l` -eq 0 ];then
    echo "SU_WHEEL_ONLY yes" >> /etc/login.defs
fi
usermod -aG wheel "$username"

# 普通用户无密码验证
if [ `grep -E "^%wheel.*NOPASSWD" /etc/sudoers|wc -l` -eq 0 ];then
    echo "%wheel ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers
fi

# 普通用户提示找不到命令
if [ `grep "!env_reset" /etc/sudoers|wc -l` -eq 0 ];then
    sed -i 's,env_reset,!&,' /etc/sudoers
    echo "alias sudo='env PATH=$PATH'" >> /home/$username/.bashrc
fi

# 关闭防火墙
systemctl stop firewalld
systemctl disable firewalld

# 关闭 SELINUX
sed -i "/SELINUX/ s,enforcing,permissive,g" /etc/selinux/config
setenforce 0

# 更换国内镜像
cp -a /etc/yum.repos.d/CentOS-Base.repo /etc/yum.repos.d/CentOS-Base.repo.bak
centos_version=$(cat /etc/redhat-release|sed -r 's/.* ([0-9]+)\..*/\1/')
curl -o /etc/yum.repos.d/CentOS-Base.repo https://mirrors.aliyun.com/repo/Centos-$centos_version.repo

# 安装 epel 源
curl -o /etc/yum.repos.d/epel.repo https://mirrors.aliyun.com/repo/epel-7.repo
sed -i "s|7|$centos_version|g" /etc/yum.repo.d/epel.repo
yum makecache

# 更新系统
yum update -y

# 安装常用包
yum install -y bash-completion vim wget iproute telnet htop conntrack ntp ipvsadm ipset jp iptables iptables-services curl sysstat libseccomp net-tools git

# 开启 iptables 防火墙
systemctl enable iptables && systemctl start iptables && iptables -F && iptables -Z && iptables -X && service iptables save

# 开启 lvs 服务
systemctl enable ipvsadm

# 时间同步
systemctl enable ntpd
systemctl restart ntpd
timedatectl set-timezone Asia/Shanghai
timedatectl set-ntp true

# 安装新版内核
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
yum install -y http://www.elrepo.org/elrepo-release-$centos_version.el$centos_version.elrepo.noarch.rpm

yum makecache fast
if [ `rpm -qa|grep ^kernel-headers|wc -l` -ge 1 ];then
    yum remove -y kernel-headers
fi
yum --enablerepo elrepo-kernel install -y kernel-lt kernel-lt-devel kernel-lt-headers
yum group remove -y "Development Tools"
yum group install -y "Development Tools"
grub2-set-default 0

# 开启 journal 日志持久化
mkdir /var/log/journal # 持久化保存日志的目录
mkdir /etc/systemd/journald.conf.d
cat > /etc/systemd/journald.conf.d/99-prophet.conf <<EOF
[Journal]
# 持久化保存到磁盘
Storage=persistent

# 压缩历史日志
Compress=yes

SyncIntervalSec=5m
RateLimitInterval=30s
RateLimitBurst=1000

# 最大占用空间 10G
SystemMaxUse=10G

# 单日志文件最大 200M
SystemMaxFileSize=200M

# 日志保存时间 2 周
MaxRetentionSec=2week

# 不将日志转发到 syslog
ForwardToSyslog=no
EOF

systemctl restart systemd-journald

# 创建 swap 文件
swapoff -a
sed -i "/swap/ s|^\(.*\)$|#\1|g" /etc/fstab
mem=$(free -m|sed '1d'|awk '/Mem/{print $2}')
swap=`expr $mem / 2`
if [ $mem -le 2048 ];then
    dd if=/dev/zero of=/tmp/swap bs=${swap}M count=4
elif [ $mem -gt 2048 && $mem -le 8192 ];then
    dd if=/dev/zero of=/tmp/swap bs=${swap}M count=2
else
    dd if=/dev/zero of=/tmp/swap bs=4G count=4
fi
chown 0:0 /tmp/swap
chmod 0600 /tmp/swap
# 格式化 swap 文件
mkswap /tmp/swap
# 开机自动挂载
echo "/tmp/swap swap swap defaults 0 0" >> /etc/fstab
swapon -a

# 调整 sysctl.conf
shmmax=$(awk -v m=$mem 'BEGIN{printf("%.f\n",m*1024*1024*0.9)}')
shmall=$(awk -v m=$mem 'BEGIN{printf("%.f\n",m*1024*0.9/4)}')
grep -q "^kernel.shmall" /etc/sysctl.conf && sed -i "s,^kernel.shmmax.*,kernel.shmmax = $shmmax," /etc/sysctl.conf || echo "kernel.shmmax = $shmmax" >> /etc/sysctl.conf
grep -q "^kernel.shmall" /etc/sysctl.conf && sed -i "s,^kernel.shmall.*,kernel.shmall = $shmall," /etc/sysctl.conf || echo "kernel.shmall = $shmall" >> /etc/sysctl.conf
grep -q "^kernel.msgmax" /etc/sysctl.conf && sed -i "s,^kernel.msgmax.*,kernel.msgmax = 65535," /etc/sysctl.conf || echo "kernel.msgmax = 65535" >> /etc/sysctl.conf
grep -q "^kernel.msgmnb" /etc/sysctl.conf && sed -i "s,^kernel.msgmnb.*,kernel.msgmnb = 65535," /etc/sysctl.conf || echo "kernel.msgmnb = 65535" >> /etc/sysctl.conf
grep -q "^vm.swappiness" /etc/sysctl.conf && sed -i "s,^vm.swappiness.*,vm.swappiness = 30," /etc/sysctl.conf || echo "vm.swappiness = 30" >> /etc/sysctl.conf
grep -q "^fs.file-max" /etc/sysctl.conf && sed -i "s,^fs.file-max.*,fs.file-max = 6553560," /etc/sysctl.conf || echo "fs.file-max = 6553560" >> /etc/sysctl.conf
sysctl -p

sh -c 'cat <<EOF> /etc/security/limits.conf
*               soft    nofile          1000000
*               hard    nofile          1000000
EOF'

# 开启 ipvs 转发
cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack

# 安装 docker
if [ `rpm -qa|grep ^docker|wc -l` -ge 1 ];then
    yum remove -y docker*
fi
curl -o /etc/yum.repos.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
yum makecache fast
containerd_rpm=http://mirrors.aliyun.com/docker-ce/linux/centos/7/x86_64/stable/Packages/containerd.io-1.2.6-3.3.el7.x86_64.rpm
yum install -y lvm2 device-mapper-persistent-data $containerd_rpm docker-ce
systemctl start docker
systemctl enable docker

sh -c 'cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": ["https://dockerhub.azk8s.cn"],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  }
}
EOF'

grep -q "^net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
grep -q "^net.ipv4.tcp_tw_reuse" /etc/sysctl.conf || echo "net.ipv4.tcp_tw_reuse = 1" >> /etc/sysctl.conf
grep -q "^net.ipv4.tcp_tw_recycle" /etc/sysctl.conf || echo "net.ipv4.tcp_tw_recycle = 0" >> /etc/sysctl.conf
grep -q "^net.netfilter.nf_conntrack_max" /etc/sysctl.conf || echo "net.netfilter.nf_conntrack_max = 2310720" >> /etc/sysctl.conf
grep -q "^net.bridge.bridge-nf-call-iptables" /etc/sysctl.conf || echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
grep -q "^net.bridge.bridge-nf-call-ip6tables" /etc/sysctl.conf || echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
sed -i '/net.core.default_qdisc/d' /etc/sysctl.conf
sed -i '/net.ipv4.tcp_congestion_control/d' /etc/sysctl.conf
echo "net.core.default_qdisc = fq" >> /etc/sysctl.conf
echo "net.ipv4.tcp_congestion_control = bbr" >> /etc/sysctl.conf

systemctl restart docker
usermod -aG docker $username
docker info

# 安装 docker-compose
yum install -y python3-pip
pip3 install -U pip -i https://pypi.douban.com/simple
pip install docker-compose -i https://pypi.douban.com/simple
docker-compose -v

# docker-compose 命令补全
curl -L https://raw.githubusercontent.com/docker/compose/1.25.5/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose

# 重启
read -p "是否立即重启服务器？（yes|no）": isyes
if [ "$isyes" = "yes" -o "$isyes" = "y" ];then
    reboot
else
    echo "稍后请手动重启服务器！"
fi
```

<!-- tabs:end -->
