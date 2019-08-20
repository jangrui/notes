# BIND

## DNS 域名解析服务

DNS (Domain Name System，域名系统) 域名解析服务，用于管理和解析域名与IP地址对应关系的技术。

简单来说，就是能够接受用户输入的域名或IP地址，然后自动查找与之匹配（或者说具有映射关系）的IP地址或域名，即将域名解析为IP地址（正向解析），或将IP地址解析为域名（反向解析）。

DNS 共分为下面三种类型的服务器：

- `主服务器`: 在特定区域内具有唯一性，负责维护该区域内的域名与IP地址之间的对应关系。
- `从服务器`: 从主服务器中获得域名与IP地址的对应关系并进行维护，以防主服务器宕机等情况。
- `缓存服务器`: 通过向其他域名解析服务器查询获得域名与IP地址的对应关系，并将经常查询的域名信息保存到服务器本地，以此来提高重复查询时的效
- `转发服务器`

DNS域名解析服务采用分布式的数据结构来存放海量的“区域数据”信息，在执行用户发起的域名查询请求时，具有递归查询和迭代查询两种方式

- `递归查询`: DNS服务器在收到用户发起的请求时，必须向用户返回一个准确的查询结果。如果DNS服务器本地没有存储与之对应的信息，则该服务器需要询问其他服务器，并将返回的查询结果提交给用户。
- `迭代查询`: DNS服务器在收到用户发起的请求时，并不直接回复查询结果，而是告诉另一台DNS服务器的地址，用户再向这台DNS服务器提交请求，这样依次反复，直到返回查询结果。

DNS 域名解析过程，例如在访问网站：www.linuxprobe.com 时，其大致查询流程如下图：

![DNS 解析流程](../_media/bind-dns.png)

## 安装 Bind 服务

BIND（Berkeley Internet Name Domain，伯克利因特网名称域），使用最广泛的 DNS 服务器软件，最早由伯克利大学的一名学生编写。

```bash
yum install -y bind-chroot bind-utils
firewall-cmd --add-service=dhcp --add-service=dhcpv6
firewall-cmd --reload
```

> 在生产环境中安装部署bind服务程序时加上chroot（俗称牢笼机制）扩展包，以便有效地限制bind服务程序仅能对自身的配置文件进行操作，以确保整个服务器的安全。

## Bind 服务相关文件

- 主配置文件: `/etc/named.conf`
- 区域配置文件: `/etc/named.rfc1912.zones` 定义了域名与 IP 地址解析规则保存的文件位置以及服务类型。
- 数据配置文件目录: `/var/named` 用来保存域名和 ip 地址真是对应关系的数据配置文件。
- 日志: `/var/log/named.log`
- rdnc配置文件: `/etc/rndc.conf`
- rdnc秘钥文件: `/etc/rndc.key` 用来远程控制DNS服务的密钥文件

### 主配置文件

```bash
cat /etc/named.conf
options {
    listen-on port 53 { 127.0.0.1; };
    # IPv4 地址监听端口及监听的主机列表
    listen-on-v6 port 53 { ::1; };
    # IPv6 地址监听端口及监听的主机列表
    directory     "/var/named";
    # 服务器的工作目录
    dump-file     "/var/named/data/cache_dump.db";
    # 当执行 rndc dumpdb 时服务器 dump文件的路径
    statistics-file "/var/named/data/named_stats.txt";
    # 执行 rndc stats将服务器的统计信息写入文件,默认为named.stats
    memstatistics-file "/var/named/data/named_mem_stats.txt";
    # 默认为 named.memestats,当退出的服务的时候将服务器的统计信息写到文件中
    masterfile-format text;
    # 同步的文件格式，防止乱码
    recursing-file  "/var/named/data/named.recursing";
    secroots-file   "/var/named/data/named.secroots";
    allow-query     { localhost; };
    # 允许向我查询的主机列表，表示可以对主机列表里的主机提供服务
    allow-transfer { any；};
    # 指定哪些主机可以从服务器上接收区域传输,未指定将允许传输到所有的主机
    # 默认 any，极其不安全，通常修改为指定主机或 none

    recursion yes;
    # 是否开启递归查询请求，设置为 no 的话，不去寻根
    forworders { 114.114.114.114; 8.8.8.8; };
    #开启转发，启用 forwarders 查询会减少本地流量的浪费，直接从转发的服务器上查询的结果返回；

    dnssec-enable yes;
    # dns 安全策略，建议关闭
    dnssec-validation yes;
    # dns 安全策略，建议关闭

    bindkeys-file "/etc/named.iscdlv.key";
    managed-keys-directory "/var/named/dynamic";
    pid-file "/run/named/named.pid";
    session-keyfile "/run/named/session.key";
};
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
zone "." IN {
    type hint;
    file "named.ca";
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
```

### 区域配置文件

定义了域名与IP地址解析规则保存的文件位置以及服务类型等内容。

服务类型有三种：

- `hint`: 根区域
- `master`: 主区域
- `slave`: 辅助区域

```bash
cat /etc/named.rfc1912.zones
zone "localhost.localdomain" IN {
    # 正向解析
    type master;
    # 服务类型
    file "named.localhost";
    # 域名与 ip 地址解析规则保存的文件位置
    allow-update { none; };
    # 允许哪些客户动态更新解析信息
};

zone "localhost" IN {
    type master;
    file "named.localhost";
    allow-update { none; };
};

zone "1.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.0.ip6.arpa" IN {
    type master;
    file "named.loopback";
    allow-update { none; };
};

zone "1.0.0.127.in-addr.arpa" IN {
    # 反向解析
    type master;
    file "named.loopback";
    allow-update { none; };
};

zone "0.in-addr.arpa" IN {
    type master;
    file "named.empty";
    allow-update { none; };
};
```

### 数据配置文件

- 正向解析

```bash
$TTL 1D #生存周期为1天		
@       IN SOA      linuxprobe.com.         root.linuxprobe.com.	(	
    	#授权信息开始 #DNS区域的地址            #域名管理员的邮箱(不要用@符号)	
                                                                        0;serial    #更新序列号
                                                                        1D;refresh  #更新时间
                                                                        1H;retry    #重试延时
                                                                        1W;expire   #失效时间
                                                                        3H;)minimum #无效解析记录的缓存时间
        NS          ns.linuxprobe.com.      #域名服务器记录
ns      IN A        192.168.10.10           #地址记录(ns.linuxprobe.com.)
        IN MX 10	mail.linuxprobe.com.    #邮箱交换记录
mail    IN A	    192.168.10.10           #地址记录(mail.linuxprobe.com.)
www     IN A	    192.168.10.10           #地址记录(www.linuxprobe.com.)
bbs     IN A	    192.168.10.20           #地址记录(bbs.linuxprobe.com.)
```

- 反向解析

```bash
$TTL 1D
@	IN SOA	linuxprobe.com.     root.linuxprobe.com. (
                                                            0       ; serial
                                                            1D      ; refresh
                                                            1H      ; retry
                                                            1W      ; expire
                                                            3H )    ; minimum
	NS	    ns.linuxprobe.com.
ns  A       192.168.10.10
10  PTR     ns.linuxprobe.com.      # PTR为指针记录，仅用于反向解析中。
10  PTR     mail.linuxprobe.com.
10  PTR     www.linuxprobe.com.     # ip 主机位对应域名
20  PTR     bbs.linuxprobe.com.
```

### ACL

把一个或多个地址归并为一个集合，并通过一个统一的名称调用

四个内置 ACL:

- `none`: 没有一个主机
- `any`: 任意主机
- `localhost`: 本机
- `localnet`: 本机的IP同掩码运算后得到的网络地址，即本机 ip 所在的网段

> 注意：只能先定义，后使用；因此一般定义在配置文件中，处于options的前面

访问控制的指令：

- `allow-query {}`: 允许查询的主机；白名单
- `allow-transfer {}`: 允许区域传送的主机；白名单
- `allow-recursion {}`: 允许递归的主机,建议全局使用
- `allow-update {}`: 允许更新区域数据库中的内容

## 正向解析

1. 修改主配置文件

允许所有人对本服务器发送DNS查询请求。

```bash
sed -i -e 's,localhost,any,g' -e 's,127.0.0.1,any,g' /etc/named.conf
```

2. 编辑区域文件

添加 linuxprobe.com 域名解析信息。

```bash
cat <<EOF> /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
    type master;
    file "linuxprobe.com.zone";
    allow-update { none; };
};
EOF
```

3. 编辑数据配置文件

```bash
cat <<EOF> /var/named/linuxprobe.com.zone
\$TTL 1D
@       IN SOA     linuxprobe.com.      root.linuxprobe.com.    (
					                                                0	; serial
					                                                1D	; refresh
					                                                1H	; retry
					                                                1W	; expire
					                                                3H )	; minimum
        NS         ns.linuxprobe.com.
ns      IN A       192.168.10.10
        IN MX 10   mail.linuxprobe.com.
mail    IN A       192.168.10.10
www     IN A       192.168.10.10
bbs     IN A       192.168.10.20
EOF
cat /var/named/linuxprobe.com.zone

nmcli con show
nmcli con mod ens32 ipv4.dns "192.168.10.10"
nmcli con up ens32

systemctl restart named
systemctl enable named

nslookup www.linuxprobe.com
```

> 考虑范围：应由本机提供 DNS 查询服务
>
> `nslookup`: 用于检测能否从 DNS 服务器中查询到域名与IP地址的解析记录。

## 反向解析

在 DNS 域名解析服务中，反向解析的作用是将用户提交的 IP 地址解析为对应的域名信息，它一般用于对某个 IP 地址上绑定的所有域名进行整体屏蔽，屏蔽由某些域名发送的垃圾邮件。也可以针对某个 IP 地址进行反向解析，大致判断出有多少个网站运行在上面。

1. 区域文件

```bash
cat <<EOF>> /etc/named.rfc1912.zones
zone "10.168.192.in-addr.arpa" IN {
    type master;
    file "192.168.10.arpa";
};
EOF
```

2. 数据配置文件

```bash
cat <<EOF> /var/named/192.168.10.arpa
\$TTL 1D
@   IN SOA  linuxprobe.com.     root@linuxprobe.com. (
                                                    0	; serial
                                                    1D	; refresh
                                                    1H	; retry
                                                    1W	; expire
                                                    3H)	; minimum
    NS      ns.linuxprobe.com.
ns  A       192.168.10.10
10  PTR     ns.linuxprobe.com.
10  PTR     mail.linuxprobe.com.
10  PTR     www.linuxprobe.com.
20  PTR     bbs.linuxprobe.com.
EOF

systemctl restart named

nslookup 192.168.10.10 127.0.0.1
```

## 从服务器

|主机名称|IP地址|
|-|-|
|主服务器|192.168.10.10|
|从服务器|192.168.10.20|

1. 主服务器的区域配置

> 应允许从服务器的更新请求。

```bash
cat <<EOF> /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
    type master;
    file "linuxprobe.com.zone";
    masterfile-format text;
    # 同步的文件格式，不然会乱码
    allow-update { 192.168.10.20; };
};
zone "10.168.192.in-addr.arpa" IN {
    type master;
    file "192.168.10.arpa";
    masterfile-format text;
    allow-update { 192.168.10.20; };
};
EOF

systemctl restart named
```

2. 从服务器主配置

```bash
sed -i 's,127.0.0.1,localhost,' /etc/named.conf
```

> 127.0.0.1 是 ipv4 地址；localhost 是域名，同时还指向 ipv6 的 `::1`；

3. 从服务器的区域配置

> 服务类型应是 `slave`；配置主服务器的 `IP地址`、要抓取的`区域信息`以及`存放位置`。

```bash
cat <<EOF> /etc/named.rfc1912.zones
zone "linuxprobe.com" IN {
    type slave;
    masters { 192.168.10.10; };
    masterfile-format text;
    file "slaves/linuxprobe.com.zone";
};
zone "10.168.192.in-addr.arpa" IN {
    type slave;
    masters { 192.168.10.10; };
    masterfile-format text;
    file "slaves/192.168.10.arpa";
};
EOF

nmcli con show
nmcli con mod ens32 ipv4.dns "192.168.10.20"
nmcli con up ens32

systemctl restart named

nslookup www.linuxprobe.com
nslookup 192.168.10.10
```

## 加密传输

加密方式：

- `DES` (对称加密): 文件加密和解密使用相同的密钥，简单快捷。
  - 假定有发送方 A 和接收方 B，A 和 B 有相同的密钥，A 发送明文给 B 之前，通过密钥和加密算法，将明文加密成密文，发送给 B，B 再通过密钥和解密算法，将密文解密成明文。

- `IDEA` (非对称加密): 密钥包括公钥和私钥，安全性较 DES 方式高。
  - 假定有发送方 A 和接收方 B，B 有自己的私钥和公钥，A 需要获取 B 的公钥，获取之后，A 首先自己生成一个会话密钥，然后这个会话密钥通过 B 的公钥进行加密，加密后发送给 B，B 再通过自己的私钥对它进行解密，从而得到 A 生成的会话密钥。之后，A 通过自己的会话密钥将要发送的明文进行加密，发送给 B，B 通过事先得到的会话密钥对发送过来的密文进行解密从而得到明文。


DNS事务签名:

事务签名可以通过两种加密方式来实现，分别是：

- `TSIG`: 对称方式
- `SIGO`: 非对称方式

现在比较常用的是 TSIG 这种方法。

### 主从服务之间加密传输

1. 主服务器生成秘钥。

```bash
dnssec-keygen -a HMAC-MD5 -b 128 -n HOST master-slave
ls -l
-rw-------. 1 root root   56 7月   1 13:17 Kmaster-slave.+157+64465.key
-rw-------. 1 root root  165 7月   1 13:17 Kmaster-slave.+157+64465.private
cat Kmaster-slave.+157+64465.key
master-slave. IN KEY 512 3 157 O4KHJyU/PDS/TsEqMSogdA==
```

> `dnssec-keygen`: 用于生成安全的DNS服务密钥；
>
> `-a` : 加密算法;
>
> `-b` : 加密位数;
>
> `-n` : 可以选择ZONE或者HOST;
>
> `master-slave`: 密钥名称;

2. 在主服务器中创建密钥验证文件。

```bash
cat <<EOF> /var/named/chroot/etc/transfer.key
key "master-slave" {                    # 主从服务定义应相同
    algorithm hmac-md5;                 # 加密算法
    secret "O4KHJyU/PDS/TsEqMSogdA==";  # 密钥文件中的 key 值
};
EOF

chown root.named /var/named/chroot/etc/transfer.key
chmod 640 /var/named/chroot/etc/transfer.key
```

> 密钥权限应很小。

3. 开启 bind 服务密钥验证。

```bash
sed -i '/options/i\include "/var/named/chroot/etc/transfer.key";' /etc/named.conf
sed -i '/allow-query/a\\tallow-transfer { key master-slave; };' /etc/named.conf

rm -rf /var/named/slaves/*
named-checkconf
systemctl restart named
ls /var/named/slaves/
```

4. 从服务器开启密钥验证。

```bash
scp root@192.168.10.10:/var/named/chroot/etc/transfer.key /var/named/chroot/etc/

chown root.named /var/named/chroot/etc/transfer.key
chmod 640 /var/named/chroot/etc/transfer.key

sed -i '/options/i\include "/var/named/chroot/etc/transfer.key";' /etc/named.conf
sed -i '/dnssec-validation/a\\tdnssec-lookaside auto;' /etc/named.conf
sed -i '/logging/i\server 192.168.10.10{ keys { master-slave; }; };' /etc/named.conf

rm -rf /var/named/slaves/*
named-checkconf
systemctl restart named
ls /var/named/slaves/

dig www.linuxprobe.com  @127.0.0.1 -k /var/named/chroot/etc/transfer.key
```

## 缓存服务器

DNS缓存服务器（Caching DNS Server）是一种不负责域名数据维护的DNS服务器。

简单来说，缓存服务器就是把用户经常使用到的域名与IP地址的解析记录保存在主机本地，从而提升下次解析的效率。DNS缓存服务器一般用于经常访问某些固定站点而且对这些网站的访问速度有较高要求的企业内网中，但实际的应用并不广泛。而且，缓存服务器是否可以成功解析还与指定的上级DNS服务器的允许策略有关。

```bash
ping -c4 114.114.114.114
sed -i '/recursion yes/a\\tforwarders { 114.114.114.114; 8.8.8.8; };' /etc/named.conf

nmcli con show
nmcli con show ens32 |grep ipv4.dns:

systemctl restart named

nslookup 8.8.8.8
```

> 考虑范围：应由 dns 缓存服务器提供 DNS 查询服务

## 分离解析

可让位于不同地理位置的用户通过访问相同的域名，从不同的服务器获取到相同的数据。

虚拟机模拟不同位置的服务器和不同位置的用户：

|网卡名|主机名称|ip 地址|
|-|-|-|
|ens32|DNS 服务器   |192.168.10.10|
|     |大陆 DNS 记录 |122.71.115.15|
|     |美国 DNS 记录 |106.185.25.15|
|ens33|大陆用户      |192.168.11.130|
|ens34|美国用户      |192.168.22.140|

1. 主配置文件

```bash
cat <<EOF> /etc/named.conf
options {
	listen-on port 53 { localhost; };
	listen-on-v6 port 53 { ::1; };
	directory 	"/var/named";
	dump-file 	"/var/named/data/cache_dump.db";
	statistics-file "/var/named/data/named_stats.txt";
	memstatistics-file "/var/named/data/named_mem_stats.txt";
	recursing-file  "/var/named/data/named.recursing";
	secroots-file   "/var/named/data/named.secroots";
	allow-query     { localhost; };
	recursion yes;
	dnssec-enable yes;
	dnssec-validation yes;
	bindkeys-file "/etc/named.iscdlv.key";
	managed-keys-directory "/var/named/dynamic";
	pid-file "/run/named/named.pid";
	session-keyfile "/run/named/session.key";
};
logging {
        channel default_debug {
                file "data/named.run";
                severity dynamic;
        };
};
include "/etc/named.rfc1912.zones";
include "/etc/named.root.key";
EOF
```

2. 区域配置文件

```bash
cat <<EOF> /etc/named.rfc1912.zones
acl "china" { 192.168.11.0/24; };
acl "american" { 192.168.22.0/24; };
view "china" {
    match-clients { "china"; };
    zone "linuxprobe.com" {
        type master;
        file "linuxprobe.com.china";
    };
};
view "american" {
    match-clients { "american"; };
    zone "linuxprobe.com" {
        type master;
        file "linuxprobe.com.american";
    };
};
EOF
```

3. 数据配置文件

```bash
cat <<EOF> /var/named/linuxprobe.com.china
\$TTL 1D
@       IN SOA     linuxprobe.com.      root.linuxprobe.com.    (
                                                                    0    ; serial
                                                                    1D   ; refresh
                                                                    1H   ; retry
                                                                    1W   ; expire
                                                                    3H)  ; minimum
        NS         ns.linuxprobe.com.
ns      IN A       122.71.115.10
www     IN A       122.71.115.15
EOF

cat <<EOF> /var/named/linuxprobe.com.american
\$TTL 1D
@       IN SOA     linuxprobe.com.      root.linuxprobe.com.    (
                                                                    0    ; serial
                                                                    1D   ; refresh
                                                                    1H   ; retry
                                                                    1W   ; expire
                                                                    3H)  ; minimum
        NS         ns.linuxprobe.com.
ns      IN A       106.185.25.10
www     IN A       106.185.25.15
EOF

systemctl restart named
```

4. 模拟用户访问

虚拟机再添加两块网卡，自定义分配网络段分别为 `192.168.11.0/24` 和 `192.168.22.0/24`。

```bash
nmcli con show
nmcli con mod ens34 ipv4.method manual ipv4.addr 192.168.11.130/24 gw4 192.168.11.2 ipv4.dns 192.168.11.2
nmcli con mod ens35 ipv4.method manual ipv4.addr 192.168.22.140/24 gw4 192.168.22.2 ipv4.dns 192.168.22.2
nmcli con up ens34
nmcli con up ens35

nslookup www.linuxprobe.com 192.168.11.130
nslookup www.linuxprobe.com 192.168.22.140
```
