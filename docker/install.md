# docker 安装及加速

Docker 分为 CE 和 EE 两大版本。CE 即社区版（免费，支持周期 7 个月），EE 即企业版，强调安全，付费使用，支持周期 24 个月。

Docker CE 分为 stable, test, 和 nightly 三个更新频道。每六个月发布一个 stable 版本 (18.09, 19.03, 19.09...)。

<!-- tabs:start -->

## ** 在线安装 **

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

- yum 源安装

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
sed -i '/net.ipv4.ip_forward/d' /etc/sysctl.conf
sed -i '/net.ipv4.tcp_tw_reuse/d' /etc/sysctl.conf
sed -i '/net.bridge.bridge-nf-call-iptables/d' /etc/sysctl.conf
sed -i '/net.bridge.bridge-nf-call-ip6tables/d' /etc/sysctl.conf
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.ipv4.tcp_tw_reuse = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
sysctl -p
```

- 启动 docker 并配置国内镜像

```bash
sudo systemctl start docker
sudo systemctl enable docker

sudo sh -c 'cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
      "https://u4kqosl2.mirror.aliyuncs.com",
      "https://dockerhub.azk8s.cn",
      "http://hub-mirror.c.163.com"
    ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  }
}
EOF'

sudo systemctl restart docker
```

> registry-mirrors: 指定镜像源；
>
> 国内镜像源
>
> Azure: <https://dockerhub.azk8s.cn>
>
> 163: <http://hub-mirror.c.163.com>
>
> exec-opts: 指定 Cgroup Driver；

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

## docker-compose 命令补全

- Linux

```bash
sudo curl -L https://raw.githubusercontent.com/docker/compose/$(docker-compose -v|awk -F ',| ' '{print $3}')/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
```

- MacOS

```bash
brew install bash-completion

sudo curl -L https://raw.githubusercontent.com/docker/compose/$(docker-compose -v|awk -F ',| ' '{print $3}')/contrib/completion/bash/docker-compose -o /usr/local/etc/bash_completion.d/docker-compose

tee >> ~/.bash_profile <<EOF
if [ -f \$(brew --prefix)/etc/bash_completion ]; then
. \$(brew --prefix)/etc/bash_completion
fi
EOF
```

- Oh My Zsh

```bash
sed '/^plugins=*/s,), docker-compose&,g' ~/.zshrc

curl -L https://raw.githubusercontent.com/docker/compose/$(docker-compose -v|awk -F ',| ' '{print $3}')/contrib/completion/zsh/_docker-compose > ${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-docker-compose/_docker-compose

echo 'fpath=(${ZSH_CUSTOM:=~/.oh-my-zsh/custom}/plugins/zsh-docker-compose $fpath)' >> ~/.zshrc
echo 'autoload -Uz compinit && compinit -i' >> ~/.zshrc
exec $SHELL -l
```

## ** 离线安装 **

```bash
version=19.03.13
curl -O https://download.docker.com/linux/static/stable/x86_64/docker-$version.tgz

mkdir -p /usr/local/docker-$version
tar xvf docker-$version.tgz --strip-components 1 -C /usr/local/docker-$version/
ln -sf /usr/local/docker-$version/* /usr/bin/

cat > /usr/lib/systemd/system/docker.service <<EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service 
Wants=network-online.target

[Service]
Type=notify
# the default is not to use systemd for cgroups because the delegate issues still
# exists and systemd currently does not support the cgroup feature set required
# for containers run by docker
ExecStart=/usr/bin/dockerd -H unix://var/run/docker.sock
ExecReload=/bin/kill -s HUP \$MAINPID
TimeoutSec=0
RestartSec=2
Restart=always

# Note that StartLimit* options were moved from "Service" to "Unit" in systemd 229.
# Both the old, and new location are accepted by systemd 229 and up, so using the old location
# to make them work for either version of systemd.
StartLimitBurst=3

# Note that StartLimitInterval was renamed to StartLimitIntervalSec in systemd 230.
# Both the old, and new name are accepted by systemd 230 and up, so using the old name to make
# this option work for either version of systemd.
StartLimitInterval=60s

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not support it.
# Only systemd 226 and above support this option.
TasksMax=infinity

# set delegate yes so that systemd does not reset the cgroups of docker containers
Delegate=yes

# kill only the docker process, not all processes in the cgroup
KillMode=process

[Install]
WantedBy=multi-user.target
EOF
ln -sf /usr/lib/systemd/system/docker.service /etc/systemd/system/multi-user.target.wants/docker.service

cat > /etc/docker/daemon.json <<EOF
{
  "registry-mirrors": [
      "https://u4kqosl2.mirror.aliyuncs.com",
      "https://dockerhub.azk8s.cn"
    ],
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  }
}
EOF

systemctl daemon-reload && systemctl enable docker && systemctl restart docker

sed -i '/net.ipv4.ip_forward/d' /etc/sysctl.conf
sed -i '/net.ipv4.tcp_tw_reuse/d' /etc/sysctl.conf
sed -i '/net.bridge.bridge-nf-call-iptables/d' /etc/sysctl.conf
sed -i '/net.bridge.bridge-nf-call-ip6tables/d' /etc/sysctl.conf
echo "net.ipv4.ip_forward = 1" >> /etc/sysctl.conf
echo "net.ipv4.tcp_tw_reuse = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-iptables = 1" >> /etc/sysctl.conf
echo "net.bridge.bridge-nf-call-ip6tables = 1" >> /etc/sysctl.conf
sysctl -p

# docker-compose
# curl -L https://github.com/docker/compose/releases/download/1.17.1/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
curl -L https://dl.bintray.com/docker-compose/master/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose
chmod +x /usr/local/bin/docker-compose

# docker-compose completion
curl -L https://raw.githubusercontent.com/docker/compose/$(docker-compose -v|awk -F ',| ' '{print $3}')/contrib/completion/bash/docker-compose -o /etc/bash_completion.d/docker-compose
chmod a+x /etc/bash_completion.d/docker-compose
source /etc/bash_completion.d/docker-compose
```

<!-- tabs:end -->