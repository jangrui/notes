# docker 安装及加速

Docker 分为 CE 和 EE 两大版本。CE 即社区版（免费，支持周期 7 个月），EE 即企业版，强调安全，付费使用，支持周期 24 个月。

Docker CE 分为 stable, test, 和 nightly 三个更新频道。每六个月发布一个 stable 版本 (18.09, 19.03, 19.09...)。

## 安装
内核需3.10以上

## ubuntu

```bash
sudo apt-get update
sudo apt-get install docker-ce
```

## CentOS

- 卸载旧版:

```bash
sudo yum remove -y docker docker-*
```

- yum源安装

```bash
sudo yum install -y yum-utils \
                    device-mapper-persistent-data \
                    lvm2
```

执行下面的命令添加 yum 软件源:

鉴于国内网络问题，强烈建议使用国内源。

```bash
# aliyun
sudo yum-config-manager \ 
	--add-repo \
	https://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 中国科技大学
sudo yum-config-manager \
    --add-repo \
    https://mirrors.ustc.edu.cn/docker-ce/linux/centos/docker-ce.repo
# 官方
sudo yum-config-manager \
	--add-repo \
	https://download.docker.com/linux/centos/docker-ce.repo
sudo yum makecache fast
```

安装 docker-ce ：

```bash
sudo yum install -y docker-ce
```

## 国内镜像加速

```bash
tee /etc/docker/daemon.json <<-'end'
{
  "registry-mirrors": ["http://hub-mirror.c.163.com"]
}
end
```

重启 docker：

```bash
sudo systemctl daemon-reload
sudo systemctl restart docker
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
