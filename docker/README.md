# DOCKER 笔记

记录学习docker的笔记。

## 安装（内核3.10以上）

### ubuntu

```shell
wget -qO- https://get.docker.com/ | sh
```

- 以非root用户直接运行docker

```shell
sudo usermod -aG docker runoob
```

- 启动docker后台服务

```
sudo service docker start
```

### CentOS

- 软件源安装

```shell
yum -y install docker-io
```

- 启动dokcer后台服务

```shell
systemctl start docker
```

- 脚本安装

更新yum包到最新版：

```shell
sudo yum update
```

docker按照脚本：

```shell
curl -fsSL https://get.docker.com/ | sh
```

- 启动docker进程

```shell
service docker start
```

## 镜像加速

新版的 Docker 使用 /etc/docker/daemon.json（Linux） 或者 %programdata%\docker\config\daemon.json（Windows） 来配置 Daemon。

请在该配置文件中加入（没有该文件的话，请先建一个）：

```
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```
