# docker 安装及加速

Docker 分为 CE 和 EE 两大版本。CE 即社区版（免费，支持周期 7 个月），EE 即企业版，强调安全，付费使用，支持周期 24 个月。

Docker CE 分为 stable, test, 和 nightly 三个更新频道。每六个月发布一个 stable 版本 (18.09, 19.03, 19.09...)。

## 安装

Linux 内核需3.10以上

### MacOS

```bash
brew cask install docker
```

### ubuntu

```bash
sudo apt-get update
sudo apt-get install docker-ce
```

### CentOS

- 卸载旧版:

```bash
sudo rpm -qa|grep ^docker
sudo yum remove -y docker docker-*
```

- yum源安装

鉴于国内网络问题，强烈建议使用国内源。

```bash
# aliyun
sudo curl -o /etc/yum.repo.d/docker-ce.repo https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum makecache fast
sudo yum install -y device-mapper-persistent-data lvm2 docker-ce
```

> 中国科技大学: <https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo>
>
> 官方: <https://download.docker.com/linux/centos/docker-ce.repo>

## 配置

- 开启 ipv4 和 iptables 内核转发功能

```bash
tee /etc/sysctl.conf <<EOF
net.ipv4.ip_forward = 1
net.bridge.bridge-nf-call-iptables = 1
net.bridge.bridge-nf-call-ip6tables = 1

EOF

sysctl -p
```

- 启动 docker 并配置国内镜像

```bash
systemctl start docker
systemctl enable docker

tee /etc/docker/daemon.json <EOF
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"],
  "exec-opts": ["native.cgroupdriver=systemd"]
}
EOF

systemctl restart docker
```

> registry-mirrors: 指定镜像源；
>
> exec-opts: 指定 Cgroup Driver；

### 加速地址

> 163: <http://hub-mirror.c.163.com>
>
> aliyun: <https://u4kqosl2.mirror.aliyuncs.com>
>
> docker中国: <https://registry.docker-cn.com>
>
> daocloud.io: <http://f1361db2.m.daocloud.io>

## 普通用户添加 docker 权限

```bash
sudo gpasswd -a $USER docker
newgrp docker
```

## 安装 docker-compose

```bash
sudo yum install -y python3-pip
# sudo apt-get install -y python3-pip
sudo pip3 install -U pip -i https://pypi.douban.com/simple
sudo pip install -U docker-compose -i https://pypi.douban.com/simple
docker-compose --version
```
