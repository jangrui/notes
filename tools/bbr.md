<!--
 * @Author: jangrui
 * @Date: 2019-08-01 05:17:01
 * @LastEditors: jangrui
 * @LastEditTime: 2019-10-15 18:12:02
 * @version: 
 * @Descripttion: 
 -->
# 一键安装最新内核并开启 BBR 脚本

BY : [秋水逸冰](https://teddysun.com/489.html)

## 使用方法

```bash
sudo wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
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

## CentOS 7 安装最新版内核

```bash
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
rpm -Uvh http://www.elrepo.org/elrepo-release-7.0-4.el7.elrepo.noarch.rpm

yum remove -y kernel-headers
yum --enablerepo=elrepo-kernel -y install kernel-ml-headers

grub2-set-default 0

# 只保留两个内核版本
sed -i 's,^installonly_limit=*,installonly_limit=2,' /etc/yum.conf
reboot
```

成功安装后，再把那些之前对内核 headers 依赖的安装包，比如 gcc、gcc-c++ 之类的再安装一次即可。

```bash
yum group remove -y "development tools"
yum group install -y "development tools"
```

这是因为 shadowsocks-libev 版有个 tcp fast open 功能，如果不安装最新版内核 headers 的话，这个功能是无法开启的。
