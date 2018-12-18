# 一键安装最新内核并开启 BBR 脚本

BY : [秋水逸冰](https://teddysun.com/489.html)

## 使用方法

使用root用户登录，运行以下命令：

```bash
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
```

安装完成后，脚本会提示需要重启 VPS，输入 y 并回车后重启。

重启完成后，进入 VPS，验证一下是否成功安装最新内核并开启 TCP BBR，输入以下命令：

```bash
uname -r
```

查看内核版本，显示为最新版就表示 OK 了

```bash
sysctl net.ipv4.tcp_available_congestion_control
```

返回值一般为：

> net.ipv4.tcp_available_congestion_control = bbr cubic reno

或者为：

> net.ipv4.tcp_available_congestion_control = reno cubic bbr

```bash
sysctl net.ipv4.tcp_congestion_control
```

返回值一般为：

> net.ipv4.tcp_congestion_control = bbr

```bash
sysctl net.core.default_qdisc
```

返回值一般为：

> net.core.default_qdisc = fq

```bash
lsmod | grep bbr
```

返回值有 tcp_bbr 模块即说明 bbr 已启动。注意：并不是所有的 VPS 都会有此返回值，若没有也属正常。

### 优化TCP并发

步骤1，增加打开文件描述符的最大数量
要处理数千个并发TCP连接，我们应该增加打开的文件描述符的限制。

编辑 limits.conf

```bash
vi /etc/security/limits.conf
```

添加这两行

* soft nofile 51200
* hard nofile 51200

然后，在启动shadowsocks服务器之前，先设置ulimit

ulimit -n 51200

步骤2，调整内核参数

调整shadowocks参数的原则是

尽快重用端口和连接。

尽可能扩大队列和缓冲区。

选择TCP拥塞算法以获得大延迟和高吞吐量。

以下是/etc/sysctl.conf我们的生产服务器示例：

```bash
fs.file-max = 65535
net.ipv4.conf.all.arp_notify = 1
net.ipv4.tcp_max_tw_buckets = 60000
net.ipv4.tcp_sack = 1
net.ipv4.tcp_window_scaling = 1
net.ipv4.tcp_rmem = 4096 87380 4194304
net.ipv4.tcp_wmem = 4096 16384 4194304
net.ipv4.tcp_max_syn_backlog = 65536
net.core.netdev_max_backlog = 32768
net.core.somaxconn = 32768
net.core.wmem_default = 8388608
net.core.rmem_default = 8388608
net.core.rmem_max = 16777216
net.core.wmem_max = 16777216
net.ipv4.tcp_timestamps = 1
net.ipv4.tcp_fin_timeout = 20
net.ipv4.tcp_synack_retries = 2
net.ipv4.tcp_syn_retries = 2
net.ipv4.tcp_syncookies = 1
net.ipv4.tcp_tw_reuse = 1
net.ipv4.tcp_mem = 94500000 915000000 927000000
net.ipv4.tcp_max_orphans = 3276800
net.ipv4.ip_forward = 1
net.ipv4.ip_local_port_range = 1024 65000
net.nf_conntrack_max = 6553500
net.netfilter.nf_conntrack_max = 6553500
net.netfilter.nf_conntrack_tcp_timeout_close_wait = 60
net.netfilter.nf_conntrack_tcp_timeout_fin_wait = 120
net.netfilter.nf_conntrack_tcp_timeout_time_wait = 120
net.netfilter.nf_conntrack_tcp_timeout_established = 3600
net.core.default_qdisc = fq
net.ipv4.tcp_congestion_control = bbr
```

当然，记得执行sysctl -p以在运行时重新加载配置。

## CentOS 下最新版内核 headers 安装方法

本来打算在脚本里直接安装 kernel-ml-headers，但会出现和原版内核 headers 冲突的问题。因此在这里添加一个脚本执行完后，手动安装最新版内核 headers 之教程。

执行以下命令

```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-3.el7.elrepo.noarch.rpm
yum --enablerepo=elrepo-kernel -y install kernel-ml-headers
```

> 如果是 CentOS 6 :
> 
> rpm -Uvh https://www.elrepo.org/elrepo-release-6-8.el6.elrepo.noarch.rpm

根据 CentOS 版本的不同，此时一般会出现类似于以下的错误提示：

```bash
Error: kernel-ml-headers conflicts with kernel-headers-2.6.32-696.20.1.el6.x86_64
Error: kernel-ml-headers conflicts with kernel-headers-3.10.0-693.17.1.el7.x86_64
```

因此需要先卸载原版内核 headers ，然后再安装最新版内核 headers。执行命令：

```bash
yum remove kernel-headers
```

确认无误后，输入 y，回车开始卸载。注意，有时候这么操作还会卸载一些对内核 headers 依赖的安装包，比如 gcc、gcc-c++ 之类的。不过不要紧，我们可以在安装完最新版内核 headers 后再重新安装回来即可。

卸载完成后，再次执行上面给出的安装命令。

```bash
yum --enablerepo=elrepo-kernel -y install kernel-ml-headers
```

成功安装后，再把那些之前对内核 headers 依赖的安装包，比如 gcc、gcc-c++ 之类的再安装一次即可。

为什么要安装最新版内核 headers 呢？

这是因为 shadowsocks-libev 版有个 tcp fast open 功能，如果不安装的话，这个功能是无法开启的。

## 内核升级方法

如果是 CentOS 系统，执行如下命令即可升级内核：

```bash
yum -y install kernel-ml kernel-ml-devel
```

如果你还手动安装了新版内核 headers ，那么还需要以下命令来升级 headers ：

```bash
yum -y install kernel-ml-headers
```

- CentOS 6 的话，执行命令：

```bash
sed -i 's/^default=.*/default=0/g' /boot/grub/grub.conf
```

- CentOS 7 的话，执行命令：

```bash
grub2-set-default 0
```

如果是 Debian/Ubuntu 系统，则需要手动下载最新版内核来安装升级。

去这里下载最新版的内核 deb 安装包。

如果系统是 64 位，则下载 amd64 的 linux-image 中含有 generic 这个 deb 包；

如果系统是 32 位，则下载 i386 的 linux-image 中含有 generic 这个 deb 包；

安装的命令如下（以最新版的 64 位 4.12.4 举例而已，请替换为下载好的 deb 包）：

```bash
dpkg -i linux-image-4.12.4-041204-generic_4.12.4-041204.201707271932_amd64.deb
```

安装完成后，再执行命令：

```bash
/usr/sbin/update-grub
```

最后，重启 VPS 即可。

## 特别说明

如果你使用的是 Google Cloud Platform （GCP）更换内核，有时会遇到重启后，整个磁盘变为只读的情况。只需执行以下命令即可恢复：

```bash
mount -o remount rw /
```