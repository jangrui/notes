<!--
 * @Author: jangrui
 * @Date: 2019-08-19 14:52:07
 * @LastEditors: jangrui
 * @LastEditTime: 2019-08-21 08:03:45
 * @Description: 
 -->
 
# DHCP

动态分配 IP (Dynamic Host Configuration Protocol) 

## 主要用途

- 用于内部网络和网络服务供应商自动分配IP地址给用户
- 用于内部网络管理员作为对所有电脑作集中管理的手段
- 使用场景
- 自动化安装系统
- 解决IPV4资源不足问题

## 名词

- `dhcp客户端`：需要获取ip等信息
- `dhcp服务端`：提供ip等信息
- `作用域`：一个完整的IP地址段，DHCP协议根据作用域来管理网络的分布、分配IP地址及其他配置参数。
- `超级作用域`：用于管理处于同一个物理网络中的多个逻辑子网段。超级作用域中包含了可以统一管理的作用域列表。
- `排除范围`：把作用域中的某些IP地址排除，确保这些IP地址不会分配给DHCP客户端。
- `保留地址`：（地址绑定）将ip和mac地址绑定
- `共享作用域`：（dhcp超级作用域）一堆作用域的集合
- `地址池`：在定义了DHCP的作用域并应用了排除范围后，剩余的用来动态分配给DHCP客户端的IP地址范围。
- `租约`：DHCP客户端能够使用动态分配的IP地址的时间。租约时间到期后自动回收主机的IP地址，以免造成IP地址的浪费。
- `预约`：保证网络中的特定设备总是获取到相同的IP地址。
- `中继代理`：用于在不同网段中，转发DHCP请求信息的设备

## 租约更新

- `过去50%时`：客户端主动联系服务端，如果收到回应，会使用服务端重新分配的ip（可能是之前的，可能为新），如果没有收到回应，继续使用
- `过去75%时`：客户端主动联系服务端，如果收到回应，会使用服务端重新分配的ip（可能是之前的，可能为新），如果没有收到回应，继续使用
- `过去87.5%时`：客户端主动联系服务端，如果收到回应，会使用服务端重新分配的ip（可能是之前的，可能为新），如果没有收到回应，客户端会重新开始租约过程，使用网内其它dhcp服务端提供的ip。如果网内没有dhcp服务端回应，客户端仍可使用之前的地址，直到租约到期
  
    > `注意`：客户端重启时会自动释放掉获取的 IP，重启后会主动联系 dhcp 服务端，如果租约没有到期，且得到服务端回应，客户机会继续使用之前的 IP，否则会每五分钟尝试一次 IP 租用。
    
> `租用失败`：windows 会自动设置成169.254.*.* 

## 安装 dhcp 服务

```bash
yum install -y dhcp
systemctl start dhcpd
```

## 默认配置示例文件

```bash
cat /usr/share/doc/dhcp-4.2.5/dhcpd.conf.example
#
# DHCP Server Configuration file.
#   see /usr/share/doc/dhcp*/dhcpd.conf.example
#   see dhcpd.conf(5) man page
#
option domain-name "example.org";
# 定义默认的搜索域
option domain-name-servers ns1.example.org, ns2.example.org;
# 定义客户端的 DNS 地址，这里我们设为DHCP服务器的IP地址
#option subnet-mask 255.255.255.0;
# 定义客户端默认的子网掩码
default-lease-time 600;
# 默认的租约时间，秒为单位
max-lease-time 7200;
# 最大的租约时间，秒为单位
#ddns-update-style none;
# 定义DNS服务动态更新的类型，类型包括：none（不支持动态更新）、interim（互动更新模式）与 ad-hoc（特殊更新模式）
#authoritative;
# 指定当一个客户端试图获得一个不是该DHCP服务器分配的IP信息，DHCP将发送一个拒绝消息，而不会等待请求超时。当请求被拒绝，客户端会重新向当前DHCP发送IP请求获得新地址。
#ignore client-updates;
# 忽略客户端更新DNS记录
log-facility local7;
# 定义日志服务，可以在日志配置文件中查看具体日志位置，默认是：/var/log/boog.log，但是在/var/log/messages里面也会记录dhcp日志

# 子网网段声明
subnet 10.152.187.0 netmask 255.255.255.0 {
}
subnet 10.254.239.0 netmask 255.255.255.224 {
# 分配的网段及子网掩码，代表只在 10.254.239.0 这个 C 类网段里生效，子网掩码设为255.255.255.0
  range 10.254.239.10 10.254.239.20;
  # 定义用于分配的 IP 地址池，起始到结束,尽量不要包含DHCP服务器的IP地址
  option routers rtr-239-0-1.example.org, rtr-239-0-2.example.org;
  # 定义客户端的网关地址
}
subnet 10.254.239.32 netmask 255.255.255.224 {
  range dynamic-bootp 10.254.239.40 10.254.239.60;
  option broadcast-address 10.254.239.31;
  # 定义客户端的广播地址
  option routers rtr-239-32-1.example.org;
}
subnet 10.5.5.0 netmask 255.255.255.224 {
  range 10.5.5.26 10.5.5.30;
  option domain-name-servers ns1.internal.example.org;
  option domain-name "internal.example.org";
  option routers 10.5.5.1;
  option broadcast-address 10.5.5.31;
  default-lease-time 600;
  max-lease-time 7200;
}
host passacaglia {
  hardware ethernet 0:0:c0:5d:bd:95;
  # 指定网卡接口的类型与 MAC 地址
  filename "vmunix.passacaglia";
  # 启动文件名称，用于无盘工作站
  server-name "toccata.fugue.com";
  # 向 DHCP 客户端通知 DHCP 服务器的主机名
}
host fantasia {
  hardware ethernet 08:00:07:26:c0:a5;
  fixed-address fantasia.fugue.com;
  # 将某个固定的 IP 地址分配给指定主机
}
class "foo" {
# 定义多个子网，class 后面写组名
  match if substring (option vendor-class-identifier, 0, 4) = "SUNW";
}
shared-network 224-29 {
# 定义多个子网，要从大往小写
  subnet 10.17.224.0 netmask 255.255.255.0 {
    option routers rtr-224.example.org;
  }
  subnet 10.0.29.0 netmask 255.255.255.0 {
    option routers rtr-29.example.org;
  }
  pool {
    allow members of "foo";
    range 10.17.224.10 10.17.224.250;
  }
  pool {
    deny members of "foo";
    range 10.0.29.10 10.0.29.230;
  }
}
```

## 虚拟机模拟 dhcp

关闭虚拟机自带 dhcp 功能，添两块网卡，两块网卡模式均设置为仅主机模式，一块网卡作为 dhcp 服务器网卡，另一张作为客户端测试网卡。

### 自动 ip 分配

```bash
nmcli con show
nmcli con mod ens32 ipv4.addr 192.168.10.10/24 gw4 192.168.10.2 ipv4.dns 192.168.10.2
nmcli con up ens32

cat <<EOF> /etc/dhcp/dhcpd.conf
ddns-update-style none;
ignore client-updates;
subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.50 192.168.10.150;
    option routers 192.168.10.1;
    option domain-name "linuxprobe.com";
    option domain-name-servers 192.168.10.1;
    default-lease-time 21600;
    max-lease-time 43200;
}
EOF

systemctl start dhcpd
systemctl enable dhcpd

nmcli con mod ens34 ipv4.method auto
nmcli con up ens34
ip addr
```

### 固定 ip 分配

```bash
ip link show ens34|grep link|awk '{print $2}'
cat <<EOF> /etc/dhcp/dhcpd.conf 
ddns-update-style none;
ignore client-updates;
subnet 192.168.10.0 netmask 255.255.255.0 {
    range 192.168.10.50 192.168.10.150;
    option routers 192.168.10.1;
    option domain-name "linuxprobe.com";
    option domain-name-servers 192.168.10.1;
    default-lease-time 21600;
    max-lease-time 43200;
    host linuxprobe {
        hardware ethernet 00:0c:29:27:c6:12;
        fixed-address 192.168.10.88;
    }
}
EOF

systemctl start dhcpd

nmcli con del ens34
nmcli con add con-name ens34 type ethernet ifname ens34 ipv4.method auto
nmcli con up ens34
ip addr
```
