# docker 国内加速

## 安装
内核需3.10以上

## ubuntu

```bash
wget -qO- https://get.docker.com/ | sh
```

- 以非root用户直接运行docker

```bash
sudo usermod -aG docker runoob
```

- 启动docker后台服务

```bash
sudo service docker start
```

## CentOS

- 软件源安装

```bash
yum -y install docker-io
```

- 启动dokcer后台服务

```bash
systemctl start docker
```

- 脚本安装

更新yum包到最新版：

```bash
sudo yum update
```

docker按照脚本：

```bash
curl -fsSL https://get.docker.com/ | sh
```

- 启动docker进程

```bash
service docker start
```

## 国内镜像加速

新版的 Docker 使用 /etc/docker/daemon.json（Linux） 或者 %programdata%\docker\config\daemon.json（Windows） 来配置 Daemon。

请在该配置文件中加入（没有该文件的话，请先建一个）：

```bash
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
```

### 加速地址:

> 163:
> http://hub-mirror.c.163.com
>
> aliyun:
> https://u4kqosl2.mirror.aliyuncs.com
>
> docker中国:
> https://registry.docker-cn.com
>
> daocloud.io:
> http://f1361db2.m.daocloud.io
