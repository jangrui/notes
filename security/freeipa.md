# FreeIPA 

FreeIPA 是一个集成了 Linux (Fedora)、389 目录服务器、MIT Kerberos、NTP、DNS 和 Dogtag(证书系统)的安全信息管理解决方案。

## 模拟环境

||server|client|
|-|-|-|
|ip|192.168.10.10|192.168.10.11|
|gw|192.168.10.2|192.168.10.2|
|dns|192.168.10.2|192.168.10.2|
|linux|centos7|centos7|

- 防火墙、SELINUX 处于关闭状态

```bash
systemctl disable firewalld
systemctl stop firewalld
sed -i '/^SELINUX/s,enforcing,disabled,' /etc/selinux/config
setenforce 0
```

- 主机通讯

```bash
cat <<EOF>> /etc/hosts
192.168.10.10 server.example.com server
192.168.10.11 client.example.com client
EOF
```

- NTP 时间同步

```bash
yum install -y chrony
sed -i 's,^server,#&,' /etc/chrony.conf
sed -i '1iserver ntp.aliyun.com iburst' /etc/chrony.conf
systemctl restart chronyd
chronyc sources -v
```

- 指定 dns 

## 配置 Server

- 安装

```bash
yum install -y ipa-server-dns
```

- 配置 dns

```bash
nmcli con show
nmcli con mod ens32 ipv4.dns 192.168.10.10
nmcli con up ens32
```

- 配置 server

```bash
ipa-server-install --hostname server.example.com \
--setup-dns --forwarder=192.168.10.2 \
-r EXAMPLE.COM \
-n example.com \
-p dm_passwd \
-w admin_passwd
```

> IPA server 卸载：
> 
> `ipa-server-install --uninstall`

- 为首次登陆的用户创建主目录

```bash
authconfig --enablemkhomedir --update
```

- sssd 定时更新缓存

```bash
sed -i '14i ldap_sudo_smart_refresh_interval = 30' /etc/sssd/sssd.conf
sed -i '14i ldap_sudo_full_refresh_interval = 3600' /etc/sssd/sssd.conf
sed -i '14i debug_level = 0x3ff0' /etc/sssd/sssd.conf

service sssd restart
```

## 配置 Replica

- 安装

```bash
yum install -y ipa-server-dns
```

- 生成 gpg

```bash
ssh root@replica ipa-replica-prepare replica.example.com --ip-address 192.168.10.20
ssh root@replica ls /var/lib/ipa/
scp root@replica:/var/lib/ipa/replica-info-replica.example.com.gpg /var/lib/ipa
```

- 配置

```bash
ipa-replica-install /var/lib/ipa/replica-info-replica.example.com.gpg \
--setup-dns --forwarder 192.168.10.2 \
-r EXAMPLE.COM \
-n example.com \
-p dm_passwd \
-w admin_passwd \
--mkhomedir
```

## 配置 Client

- 安装

```bash
yum install -y ipa-client
```

- 配置

```bash
ipa-client-install --enable-dns-updates --mkhomedir --no-ntp -p admin
```

- 指定 ladp 用户默认 shell

```bash
sed -i '14i override_shell = /bin/bash' /etc/sssd/sssd.conf
sed -i '14i default_shell = /bin/bash' /etc/sssd/sssd.conf

service sssd restart
```

## IPA 管理

- 服务管理

```bash
# IPA 服务启动、停止、重启、状态
ipactl start|stop|restart|status
```

- 用户管理

```bash
# 添加用户
ipa user-add jsmith
# 修改用户
ipa user-mod jsmith --title="Editor III"
# 删除用户
ipa user-del jsmith
# 查找用户
ipa user-find smith
```
