# CentOS 7 自用初始化

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
# 创建 swap 文件
if [ $mem -le 2048 ];then
    dd if=/dev/zero of=/tmp/swap bs=${mem}M count=2
elif [ $mem -gt 2048 && $mem -le 8192 ];then
    dd if=/dev/zero of=/tmp/swap bs=${mem}M count=1
else
    dd if=/dev/zero of=/tmp/swap bs=4G count=4
fi
sudo chown 0:0 /tmp/swap
sudo chmod 0600 /tmp/swap
# 格式化 swap 文件
sudo mkswap /tmp/swap
# 开机自动挂载
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
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF'

sudo sh -c '
grep -q "^net.ipv4.ip_forward" /etc/sysctl.conf || echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
grep -q "^net.bridge.bridge-nf-call-iptables" /etc/sysctl.conf || echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
grep -q "^net.bridge.bridge-nf-call-ip6tables" /etc/sysctl.conf || echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
'

sudo systemctl restart docker

sudo gpasswd -a $USER docker
newgrp docker

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
sudo curl -L https://raw.githubusercontent.com/docker/compose/1.25.3/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
source /etc/bash_completion.d/docker-compose
```

## ** 快速初始化 **

```bash
sudo sh -c 'curl -L https://www.jangrui.com/centos_init.sh|bash'
```

<!-- tabs:end -->
