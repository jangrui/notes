<!--
 * @Author: jangrui
 * @Date: 2019-07-31 07:48:02
 * @LastEditors: jangrui
 * @LastEditTime: 2019-08-24 22:07:10
 * @version: 
 * @Descripttion: Proxy Server
 -->

# Proxy Server

代理服务器（Proxy Server）主要用于转发客户系统的网络访问请求。

但是，代理服务器不只是简单地向真正的因特网服务器转发请求，它还可以控制用户的行为，对接收到的客户请求进行决策，并根据过滤规则对用户请求进行过滤。

从代理服务器工作的层次角度来说可以分为【应用层代理】、【传输层代理】和【SOCKS代理】。

- `应用层代理`: 工作在TCP/IP模型的应用层之上，它只能用于支持代理的应用层协议（如HTTP，FTP）。它提供的控制最多，但是不灵活，必须要有相应的协议支持。
- `传输层代理`: 直接与TCP层交互，更加灵活。要求代理服务器具有部分真正服务器的功能：监听特定TCP或UDP端口，接收客户端的请求同时向客户端发出相应的响应。
- `Socks`: 是一个 c/s 架构的代理协议。它包括两个主要的组件，Socks 服务端和 Socks 客户端。Socks 服务端实现在应用层，Socks 客户端实现在应用层与传输层之间。当客户想立到应用服务器的连接时，先连接到代理服务器，代理服务器再连接到应用服务器。代理服务器在客户端与应用服务器之间中转数据。


## Squid

Squid 是 Linux 系统中最为流行的一款高性能代理服务软件，工作在应用层，通常用作 Web 网站的前置缓存服务，能够代替用户向网站服务器请求页面数据并进行缓存。

简单来说，Squid 服务程序会按照收到的用户请求向网站源服务器请求页面、图片等所需的数据，并将服务器返回的数据存储在运行 Squid 服务程序的服务器上。当有用户再请求相同的数据时，则可以直接将存储服务器本地的数据交付给用户，这样不仅减少了用户的等待时间，还缓解了网站服务器的负载压力。

Squid服务程序具有配置简单、效率高、功能丰富等特点，它能支持HTTP、FTP、SSL等多种协议的数据缓存，可以基于访问控制列表（ACL）和访问权限列表（ARL）执行内容过滤与权限管理功能，还可以基于多种条件禁止用户访问存在威胁或不适宜的网站资源，因此可以保护企业内网的安全，提升用户的网络体验，帮助节省网络带宽。

### 代理模式

- `正向代理模式`: 是指让用户通过Squid服务程序获取网站页面等资源，以及基于访问控制列表（ACL）功能对用户访问网站行为进行限制，在具体的服务方式上又分为标准代理模式与透明代理模式。

  - `标准正向代理模式`: 是把网站数据缓存到服务器本地，提高数据资源被再次访问时的效率，用户在上网时必须通过代理服务器的 IP 地址与端口号访问，否则默认不使用代理服务。
  - `透明正向代理模式`: 作用与标准正向代理模式基本相同，区别是用户不需要手动指定代理服务器的IP地址与端口号，对于用户来讲是相对透明的。

- `反向代理模式`: 是指让多台节点主机反向缓存网站数据，可以就近为用户分配节点并传输资源，从而加快用户访问速度。


### 部署

```bash
yum install -y squid
systemctl start squid
systemctl enable squid

firewall-cmd --permanent --add-service=squid
firewall-cmd --reload
```

主配置文件`/etc/squid/squid.conf`参数说明：

```bash
# 信任用户与目标控制，定义可能使用 proxy 的外部用户(内网)
acl localnet src 10.0.0.0/8	# RFC1918 possible internal network
acl localnet src 172.16.0.0/12	# RFC1918 possible internal network
acl localnet src 192.168.0.0/16	# RFC1918 possible internal network
acl localnet src fc00::/7       # RFC 4193 local private network range
acl localnet src fe80::/10      # RFC 4291 link-local (directly plugged) machines

# 定义可获取到数据的端口
acl SSL_ports port 443
acl Safe_ports port 80		# http
acl Safe_ports port 21		# ftp
acl Safe_ports port 443		# https
acl Safe_ports port 70		# gopher
acl Safe_ports port 210		# wais
acl Safe_ports port 1025-65535	# unregistered ports
acl Safe_ports port 280		# http-mgmt
acl Safe_ports port 488		# gss-http
acl Safe_ports port 591		# filemaker
acl Safe_ports port 777		# multiling http

acl CONNECT method CONNECT

# 定义可放行的标准依据(有顺序)
# 拒绝非正规的端口访问要求
http_access deny !Safe_ports
# 拒绝非正规的加密端口访问要求
http_access deny CONNECT !SSL_ports
# 放行本机管理
http_access allow localhost manager
# 拒绝其他管理来源
http_access deny manager
# 放行内部网络的用户来源
http_access allow localnet
# 放行本机的使用
http_access allow localhost
# 全部都予以拒绝
http_access deny all

# 监听的端口号
http_port 3128
# 透明代理
#http_port 3128 transparent

# 默认的 squid 快取放置的目录
coredump_dir /var/spool/squid
# 磁盘中快取的存在时间
refresh_pattern ^ftp:		1440	20%	10080
refresh_pattern ^gopher:	1440	0%	1440
refresh_pattern -i (/cgi-bin/|\?) 0	0%	0
refresh_pattern .		0	20%	4320
```

### 正向代理

默认配置就可以提供标准正向代理。

> 虚拟机模拟正向代理访问:

```bash
curl www.linuxprobe.com

systemctl stop squid

curl www.linuxprobe.com -x localhost:3128

systemctl start squid

curl www.linuxprobe.com -x localhost:3128
```

### ACL 访问控制

1. 只允许 ip 地址为 192.168.10.20 的客户端访问代理服务，禁止其余所有主机代理请求。

```bash
sed -i '/acl CONNECT method CONNECT/a\acl client src 192.168.10.20' /etc/squid/squid.conf
sed -i '/!Safe_ports/i\http_access allow client\nhttp_access deny all' /etc/squid/squid.conf

systemctl restart squid
```

> 虚拟机模拟测试，本地 ip：192.168.10.10

```bash
nmcli show
nmcli con mod ens32 ipv4.addr 192.168.10.20/24
nmcli con up ens32
curl www.linuxprobe.com -x 192.168.10.20:3128

nmcli con mod ens32 ipv4.addr 192.168.10.10/24
nmcli con up ens32
curl www.linuxprobe.com -x 192.168.10.10:3128
curl www.linuxprobe.com
```

2. 禁止所有客户端访问网址中包含 linux 关键词的网站。

```bash
cat /etc/squid/squid.conf.default > /etc/squid/squid.conf
sed -i '/acl CONNECT method CONNECT/a\acl deny_keyword url_regex -i linux' /etc/squid/squid.conf
sed -i '/!Safe_ports/i\http_access deny deny_keyword' /etc/squid/squid.conf

systemctl restart squid

curl www.linuxprobe.com
curl www.linuxprobe.com -x localhost:3128
```

3. 禁止所有用户访问某个特定的网站。

```bash
cat /etc/squid/squid.conf.default > /etc/squid/squid.conf
sed -i '/acl CONNECT method CONNECT/a\acl deny_url url_regex www.linuxprobe.com' /etc/squid/squid.conf
sed -i '/!Safe_ports/i\http_access deny deny_url' /etc/squid/squid.conf

systemctl restart squid

curl www.linuxprobe.com
curl www.linuxprobe.com -x localhost:3128
```

4. 禁止下载带有某些后缀的文件。

```bash
cat /etc/squid/squid.conf.default > /etc/squid/squid.conf
sed -i '/acl CONNECT method CONNECT/a\acl badfile urlpath_regex -i \.png$' /etc/squid/squid.conf
sed -i '/!Safe_ports/i\http_access deny badfile' /etc/squid/squid.conf

systemctl restart squid

curl www.linuxprobe.com/linux.png
curl www.linuxprobe.com/linux.png -x localhost:3128
```

### 反向代理

```bash
yum install -y httpd
systemctl start httpd

nmcli con show ens32 |grep ipv4.addr
nmcli con mod ens32 +ipv4.addr 192.168.10.50/24
nmcli con up ens32

ping -I 192.168.10.50 www.linuxprobe.com

echo "192.168.10.50 www.example.com" >> /etc/hosts
curl www.example.com

cat /etc/squid/squid.conf.default > /etc/squid/squid.conf
sed -i 's,http_port 3128,& 192.168.10.10:8080 vhost,' /etc/squid/squid.conf
sed -i '/http_port/a\cache_peer 192.168.10.50 parent 80 0 originserver' /etc/squid/squid.conf

systemctl restart squid

curl www.example.com -x 192.168.10.10:8080
```

### 匿名代理

- 透明代理(Transparent Proxy)
  - `REMOTE_ADDR` = Proxy IP
  - `HTTP_VIA` = Proxy IP
  - `HTTP_X_FORWARDED_FOR` = Your IP
  - 透明代理虽然可以直接“隐藏”你的IP地址，但是还是可以从HTTP_X_FORWARDED_FOR来查到你是谁。

- 匿名代理(Anonymous Proxy)
  - `REMOTE_ADDR` = Proxy IP
  - `HTTP_VIA` = Proxy IP
  - `HTTP_X_FORWARDED_FOR` = proxy IP
  - 匿名代理比透明代理进步了一点：别人只能知道你用了代理，无法知道你是谁。

- 混淆代理(Distorting Proxies)
  - `REMOTE_ADDR` = Proxy IP
  - `HTTP_VIA` = Proxy IP
  - `HTTP_X_FORWARDED_FOR` = Random IP address
  - 与匿名代理相同，如果使用了混淆代理，别人还是能知道你在用代理，但是会得到一个假的IP地址，伪装的更逼真

- 高匿代理(Elite proxy或High Anonymity Proxy)
  - `REMOTE_ADDR` = Proxy IP
  - `HTTP_VIA` = not determined
  - `HTTP_X_FORWARDED_FOR` = not determined
  - 高匿代理让别人根本无法发现你是在用代理，所以是最好的选择。

> 虚拟机模拟测试：

```bash
nmcli con show ens32|grep ipv4.addr
nmcli con mod ens32 +ipv4.addr 192.168.10.20/24
nmcli con up ens32

yum install -y nginx
systemctl start nginx
echo "192.168.10.10 www.example.com" >> /etc/hosts

cat /etc/squid/squid.conf.default > /etc/squid/squid.conf
sed -i 's,http_port 3128,http_port 192.168.10.20:3128,' /etc/squid/squid.conf
sed -i '/http_port/i\forwarded_for delete' /etc/squid/squid.conf
sed -i '/http_port/i\follow_x_forwarded_for deny all' /etc/squid/squid.conf
sed -i '/http_port/i\request_header_access Via deny all' /etc/squid/squid.conf
sed -i '/http_port/i\request_header_access X-Forwarded-For deny all' /etc/squid/squid.conf
sed -i '/http_port/i\request_header_access From deny all' /etc/squid/squid.conf
# 禁用缓存
sed -i '/#cache_dir/i\acl NCACHE method GET' /etc/squid/squid.conf
sed -i '/#cache_dir/i\no_cache deny NCACHE' /etc/squid/squid.conf

systemctl restart squid

curl -I www.example.com
curl -I www.example.com -x 192.168.10.20:3128
curl -I www.example.com -x 192.168.10.20:3128
curl -I www.example.com -x 192.168.10.20:3128

cat /var/log/nginx/access.log
```