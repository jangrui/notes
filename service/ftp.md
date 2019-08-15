# FTP

FTP是一种在互联网中进行文件传输的协议，基于客户端/服务器模式，默认使用20、21号端口，其中端口20（数据端口）用于进行数据传输，端口21（命令端口）用于接受客户端发出的相关FTP命令与参数。FTP服务器普遍部署于内网中，具有容易搭建、方便管理的特点。而且有些FTP客户端工具还可以支持文件的多点下载以及断点续传技术，因此FTP服务得到了广大用户的青睐。

![FTP协议的传输拓扑](../_media/FTP.png)

FTP协议有两种工作模式

- 主动模式：FTP 服务器主动向客户端发起连接请求。
- 被动模式：FTP 服务器等待客户端发起连接请求（FTP 默认工作模式）。

## 安装 FTP 服务

- `vsftpd`（very secure ftp daemon，非常安全的FTP守护进程）是一款运行在Linux操作系统上的FTP服务程序，不仅完全开源而且免费，此外，还具有很高的安全性、传输速度，以及支持虚拟用户验证等其他FTP服务程序不具备的特点。

```bash
[root@localhost ~]# hostname
localhost.localdomain
[root@localhost ~]# yum install -y vsftpd
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * extras: mirrors.163.com
 * updates: mirrors.163.com
正在解决依赖关系
--> 正在检查事务
---> 软件包 vsftpd.x86_64.0.3.0.2-25.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

===============================================================================================================
 Package                  架构                     版本                           源                      大小
===============================================================================================================
正在安装:
 vsftpd                   x86_64                   3.0.2-25.el7                   base                   171 k

事务概要
===============================================================================================================
安装  1 软件包

总下载量：171 k
安装大小：353 k
Downloading packages:
vsftpd-3.0.2-25.el7.x86_64.rpm                                                          | 171 kB  00:00:01
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : vsftpd-3.0.2-25.el7.x86_64                                                                 1/1
  验证中      : vsftpd-3.0.2-25.el7.x86_64                                                                 1/1

已安装:
  vsftpd.x86_64 0:3.0.2-25.el7

完毕！
[root@localhost ~]# systemctl start vsftpd
[root@localhost ~]# netstat -lntp|grep vsftpd
tcp6       0      0 :::21                   :::*                    LISTEN      49228/vsftpd
```

防火墙放行

```bash
[root@localhost ~]# firewall-cmd --add-service=ftp
success
[root@localhost ~]# firewall-cmd --add-service=ftp --permanent
success
[root@localhost ~]# firewall-cmd --list-all
public (active)
  target: default
  icmp-block-inversion: no
  interfaces: ens32
  sources:
  services: ssh dhcpv6-client ftp
  ports:
  protocols:
  masquerade: no
  forward-ports:
  source-ports:
  icmp-blocks:
  rich rules:

[root@localhost ~]#
```

- `ftp` 是 Linux 系统中以命令行界面的方式来管理 FTP 传输服务的客户端工具。

安装 ftp 客户端并测试 vsftp 服务

```bash
[root@client ~]# hostname
client
[root@client ~]# yum install -y ftp
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: mirrors.aliyun.com
 * elrepo: hkg.mirror.rackspace.com
 * elrepo-kernel: hkg.mirror.rackspace.com
 * epel: hkg.mirror.rackspace.com
 * extras: mirrors.aliyun.com
 * updates: mirror.jdcloud.com
正在解决依赖关系
--> 正在检查事务
---> 软件包 ftp.x86_64.0.0.17-67.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

===============================================================================================================
 Package                架构                      版本                           源                       大小
===============================================================================================================
正在安装:
 ftp                    x86_64                    0.17-67.el7                    base                     61 k

事务概要
===============================================================================================================
安装  1 软件包

总下载量：61 k
安装大小：96 k
Downloading packages:
ftp-0.17-67.el7.x86_64.rpm                                                              |  61 kB  00:00:00
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : ftp-0.17-67.el7.x86_64                                                                     1/1
  验证中      : ftp-0.17-67.el7.x86_64                                                                     1/1

已安装:
  ftp.x86_64 0:0.17-67.el7

完毕！
[root@client ~]# ftp 192.168.11.130
Connected to 192.168.11.130 (192.168.11.130).
220 (vsFTPd 3.0.2)
Name (192.168.11.130:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (192,168,11,130,80,13).
150 Here comes the directory listing.
drwxr-xr-x    2 0        0               6 Oct 30  2018 pub
226 Directory send OK.
ftp> exit
221 Goodbye.
```

## vsftpd 配置说明

vsftpd 默认配置文件

```bash
[root@localhost ~]# cat /etc/vsftpd/vsftpd.conf |grep -v '^$'|grep -v '#'
anonymous_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
```

常用参数及作用：

|参数|作用|
|-|-|
|listen=`[YES、NO]`|是否以独立运行的方式监听服务|
|listen_address=`IP`|设置要监听的IP地址|
|listen_port=`21`|设置FTP服务的监听端口|
|download_enable＝`[YES、NO]`|是否允许下载文件|
|userlist_enable=`[YES、NO]`|是否设置用户列表为“允许”操作|
|userlist_deny=`[YES、NO]`|是否设置用户列表为“禁止”操作|
|max_clients=`0`|最大客户端连接数，0为不限制|
|max_per_ip=`0`|同一IP地址的最大连接数，0为不限制|
|anonymous_enable=`[YES、NO]`|是否允许匿名用户访问|
|anon_upload_enable=`[YES、NO]`|是否允许匿名用户上传文件|
|anon_umask=`022`|匿名用户上传文件的umask值|
|anon_root=`/var/ftp`|匿名用户的FTP根目录|
|anon_mkdir_write_enable=`[YES、NO]`|是否允许匿名用户创建目录|
|anon_other_write_enable=`[YES、NO]`|是否开放匿名用户的其他写入权限（包括重命名、删除等操作权限）|
|anon_max_rate=`0`|匿名用户的最大传输速率（字节/秒），0为不限制|
|local_enable=`[YES、NO]`|是否允许本地用户登录FTP|
|local_umask=`022`|本地用户上传文件的umask值|
|local_root=`/var/ftp`|本地用户的FTP根目录|
|chroot_local_user=`[YES、NO]`|是否将用户权限禁锢在FTP目录，以确保安全|
|local_max_rate=`0`|本地用户最大传输速率（字节/秒），0为不限制|

## 认证模式

vsftpd作为更加安全的文件传输的服务程序，允许用户以三种认证模式登录到FTP服务器上。

- `匿名开放模式`：是一种最不安全的认证模式，任何人都可以无需密码验证而直接登录到FTP服务器。

- `本地用户模式`：是通过Linux系统本地的账户密码信息进行认证的模式，相较于匿名开放模式更安全，而且配置起来也很简单。但是如果被黑客破解了账户的信息，就可以畅通无阻地登录FTP服务器，从而完全控制整台服务器。

- `虚拟用户模式`：是这三种模式中最安全的一种认证模式，它需要为FTP服务单独建立用户数据库文件，虚拟出用来进行口令验证的账户信息，而这些账户信息在服务器系统中实际上是不存在的，仅供FTP服务程序进行认证使用。这样，即使黑客破解了账户信息也无法登录服务器，从而有效降低了破坏范围和影响。

### 匿名访问模式

一般用来访问不重要的公开文件。

vsftpd服务程序默认开启了匿名开放模式，我们需要做的就是开放匿名用户的上传、下载文件的权限，以及让匿名用户创建、删除、更名文件的权限。

可以向匿名用户开放的权限参数以及作用：

|参数|作用|
|-|-|
|anonymous_enable=`YES`|允许匿名访问模式|
|anon_umask=`022`|匿名用户上传文件的umask值|
|anon_upload_enable=`YES`|允许匿名用户上传文件|
|anon_mkdir_write_enable=`YES`|允许匿名用户创建目录|
|anon_other_write_enable=`YES`|允许匿名用户修改目录名称或删除目录|

```bash
[root@localhost ~]# cat <<EOF>> /etc/vsftpd/vsftpd.conf
> anon_umask=022
> anon_upload_enable=YES
> anon_mkdir_write_enable=YES
> anon_other_write_enable=YES
> EOF
[root@localhost ~]# cat /etc/vsftpd/vsftpd.conf
anonymous_enable=YES
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES

anon_umask=022
anon_upload_enable=YES
anon_mkdir_write_enable=YES
anon_other_write_enable=YES
[root@localhost ~]# systemctl restart vsftpd
```

客户端测试：

匿名开放认证模式下，其账户统一为anonymous，密码为空。而且在连接到FTP服务器后，默认访问的是/var/ftp目录。

```bash
[root@client ~]# ftp 192.168.11.130
Connected to 192.168.11.130 (192.168.11.130).
220 (vsFTPd 3.0.2)
Name (192.168.11.130:root): anonymous
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (192,168,11,130,245,214).
150 Here comes the directory listing.
drwxr-xr-x    2 0        0               6 Oct 30  2018 pub
226 Directory send OK.
ftp> cd pub
250 Directory successfully changed.
ftp> mkdir files
550 Create directory operation failed.
ftp>
```

系统提示创建目录操作失败。

查看 `/var/ftp/pub` 目录属性，以及关闭 SELinux 服务

```bash
[root@localhost ~]# ll -ld /var/ftp/
drwxr-xr-x. 3 root root 17 7月   1 04:16 /var/ftp/
[root@localhost ~]# ll -ld /var/ftp/pub/
drwxr-xr-x. 3 root root 19 7月   1 05:06 /var/ftp/pub/
[root@localhost ~]# chown -R ftp /var/ftp/pub/
[root@localhost ~]# ll -ld /var/ftp/pub/
drwxr-xr-x. 3 ftp root 19 7月   1 05:06 /var/ftp/pub/
[root@localhost ~]# getenforce
Enforcing
[root@localhost ~]# setenforce 0
```

再次测试创建文件

```bash
ftp> mkdir files
257 "/pub/files" created
ftp> ls
227 Entering Passive Mode (192,168,11,130,135,155).
150 Here comes the directory listing.
drwxr-xr-x    2 14       50              6 Jun 30 21:06 files
226 Directory send OK.
ftp>
```

### 本地用户模式

相较于匿名开放模式，本地用户模式要更安全，而且配置起来也很简单。

本地用户模式使用的权限参数以及作用：

|参数|作用|
|-|-|
|anonymous_enable=`NO`|禁止匿名访问模式|
|local_enable=`YES`|允许本地用户模式|
|write_enable=`YES`|设置可写权限|
|local_umask=`022`|本地用户模式创建文件的umask值
|userlist_deny=`YES`|启用“禁止用户名单”，名单文件为ftpusers和user_list|
|userlist_enable=`YES`|开启用户作用名单文件功能|

> 虚拟机恢复默认安装状态。

```bash
[root@localhost ~]# cat <<EOF> /etc/vsftpd/vsftpd.conf
> anonymous_enable=NO
> local_enable=YES
> write_enable=YES
> local_umask=022
> dirmessage_enable=YES
> xferlog_enable=YES
> connect_from_port_20=YES
> xferlog_std_format=YES
> listen=NO
> listen_ipv6=YES
> pam_service_name=vsftpd
> userlist_enable=YES
> tcp_wrappers=YES
> EOF
[root@localhost ~]# cat /etc/vsftpd/vsftpd.conf
anonymous_enable=NO
local_enable=YES
write_enable=YES
local_umask=022
dirmessage_enable=YES
xferlog_enable=YES
connect_from_port_20=YES
xferlog_std_format=YES
listen=NO
listen_ipv6=YES
pam_service_name=vsftpd
userlist_enable=YES
tcp_wrappers=YES
[root@localhost ~]# systemctl restart vsftpd
[root@localhost ~]# systemctl enable vsftpd
Created symlink from /etc/systemd/system/multi-user.target.wants/vsftpd.service to /usr/lib/systemd/system/vsftpd.service.
[root@localhost ~]# firewall-cmd --add-service=ftp --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# getenforce
Enforcing
[root@localhost ~]#
```

客户端测试：

```bash
[root@client ~]# ftp 192.168.11.130
Connected to 192.168.11.130 (192.168.11.130).
220 (vsFTPd 3.0.2)
Name (192.168.11.130:root): root
530 Permission denied.
Login failed.
ftp>
```

系统提示拒绝访问。

这是因为 vsftpd 服务程序所在的目录中默认存放着两个名为“用户名单”的文件 `ftpusers` 和 `user_list`:

```bash
[root@localhost ~]# cat /etc/vsftpd/ftpusers
# Users that are not allowed to login via ftp
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
[root@localhost ~]# cat /etc/vsftpd/user_list
# vsftpd userlist
# If userlist_deny=NO, only allow users in this file
# If userlist_deny=YES (default), never allow users in this file, and
# do not even prompt for a password.
# Note that the default vsftpd pam config also checks /etc/vsftpd/ftpusers
# for users that are denied.
root
bin
daemon
adm
lp
sync
shutdown
halt
mail
news
uucp
operator
games
nobody
[root@localhost ~]#
```

vsftpd服务程序为了保证服务器的安全性而默认禁止了root管理员和大多数系统用户的登录行为。

我们可以选择ftpusers和user_list文件中没有的一个普通用户尝试登录FTP服务器：

```bash
[root@client ~]# ftp 192.168.11.130
Connected to 192.168.11.130 (192.168.11.130).
220 (vsFTPd 3.0.2)
Name (192.168.11.130:root): jangrui
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (192,168,11,130,84,105).
150 Here comes the directory listing.
226 Directory send OK.
ftp> mkdir 123
257 "/home/jangrui/123" created
ftp> ls
227 Entering Passive Mode (192,168,11,130,135,156).
150 Here comes the directory listing.
drwxr-xr-x    2 1000     1000            6 Jun 30 20:34 123
226 Directory send OK.
ftp>
```

> 采用本地用户模式登录FTP服务器后，默认访问的是该用户的家目录

### 虚拟用户模式

虚拟用户是FTP服务器的专有用户，使用虚拟用户登录FTP，只能访问FTP服务器提供的资源，大大增强了系统的安全。

有两种方式实现虚拟用户，本地数据文件和数据库服务器。

> 虚拟机恢复默认安装状态。

#### 本地数据文件方式

1、 添加虚拟用户名和密码，一行用户名，一行密码，以此类推。奇数行为用户名，偶数行为密码。

```bash
[root@localhost ~]# cat <<end> /etc/vsftpd/vsftpduser.txt
> userone
> passwdone
> usertwo
> passwdtwo
> end
[root@localhost ~]#
```

2、 生成虚拟用户口令认证文件

明文信息既不安全，也不符合让vsftpd服务程序直接加载的格式，因此需要使用db_load命令用哈希（hash）算法将原始的明文信息文件转换成数据库文件，并且降低数据库文件的权限（避免其他人看到数据库文件的内容），然后再把原始的明文信息文件删除。

```bash
[root@localhost ~]# db_load -T -t hash -f /etc/vsftpd/vsftpduser.txt /etc/vsftpd/vsftpduser.db
[root@localhost ~]#
```

3、 创建 vsftpd 的 PAM 认证文件

```bash
[root@localhost ~]# cat <<end> /etc/pam.d/vsftpd.vuser
> auth required /lib/security/pam_userdb.so db=/etc/vsftpd/vsftpduser
> account required /lib/security/pam_userdb.so db=/etc/vsftpd/vsftpduser
> end
[root@localhost ~]# cat /etc/pam.d/vsftpd.vuser
auth required /lib/security/pam_userdb.so db=/etc/vsftpd/vsftpduser
account required /lib/security/pam_userdb.so db=/etc/vsftpd/vsftpduser
[root@localhost ~]#
```

4、 建立本地映射用户并设置宿主目录权限

创建一个可以映射到虚拟用户的系统本地用户。

简单来说，就是让虚拟用户默认登录到与之有映射关系的这个系统本地用户的家目录中，虚拟用户创建的文件的属性也都归属于这个系统本地用户，从而避免Linux系统无法处理虚拟用户所创建文件的属性权限。

```bash
[root@localhost ~]# id vsftpuser
id: vsftpuser: no such user
[root@localhost ~]# useradd -s /sbin/nologin vsftpuser
[root@localhost ~]# ll -ld /home/vsftpuser/
drwx------. 2 vsftpuser vsftpuser 62 7月   1 05:01 /home/vsftpuser/
[root@localhost ~]# chmod -R 755 /home/vsftpuser/
[root@localhost ~]#
```

5、 设置虚拟用户认证

在 vsftpd 服务程序的主配置文件中通过 pam_service_name 参数将 PAM 认证文件的名称修改为 vsftpd.vuser。

利用 PAM 文件进行认证时使用的参数以及作用：

|参数|作用|
|-|-|
|anonymous_enable=`NO`|禁止匿名开放模式|
|local_enable=`YES`|允许本地用户模式|
|guest_enable=`YES`|开启虚拟用户模式|
|guest_username=`virtual`|指定虚拟用户账户|
|pam_service_name=`vsftpd.vu`|指定PAM文件|
|allow_writeable_chroot=`YES`|允许对禁锢的FTP根目录执行写入操作，而且不拒绝用户的登录请求|

```bash
[root@localhost ~]# cat <<end> /etc/vsftpd/vsftpd.conf
> anonymous_enable=NO
> local_enable=YES
> guest_enable=YES
> guest_username=vsftpuser
> allow_writeable_chroot=YES
> write_enable=YES
> local_umask=022
> dirmessage_enable=YES
> xferlog_enable=YES
> connect_from_port_20=YES
> xferlog_std_format=YES
> listen=NO
> listen_ipv6=YES
> pam_service_name=vsftpd.vuser
> userlist_enable=YES
> tcp_wrappers=YES
> end
[root@localhost ~]# systemctl restart vsftpd
[root@localhost ~]# firewall-cmd --add-service=ftp --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# getenforce
Enforcing
[root@localhost ~]#
```

客户端测试：

```bash
[root@client ~]# ftp 192.168.11.130
Connected to 192.168.11.130 (192.168.11.130).
220 (vsFTPd 3.0.2)
Name (192.168.11.130:root): userone
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp>
```

6、 为虚拟用户设置不同的权限。

```bash
[root@localhost ~]# mkdir /etc/vsftpd/vsftpuser
[root@localhost ~]# cat <<end> /etc/vsftpd/vsftpuser/userone
> anon_upload_enable=YES
> anon_mkdir_write_enable=YES
> anon_other_write_enable=YES
> end
[root@localhost ~]# touch /etc/vsftpd/vsftpuser/usertwo
[root@localhost ~]# cat <<end>> /etc/vsftpd/vsftpd.conf
> user_config_dir=/etc/vsftpd/vsftpuser
> end
[root@localhost ~]# systemctl restart vsftpd
[root@localhost ~]#
```

客户端测试：

```bash
[root@client ~]# ftp 192.168.11.130
Connected to 192.168.11.130 (192.168.11.130).
220 (vsFTPd 3.0.2)
Name (192.168.11.130:root): userone
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (192,168,11,130,233,3).
150 Here comes the directory listing.
226 Directory send OK.
ftp> mkdir one
257 "/one" created
ftp> exit
221 Goodbye.
[root@client ~]# ftp 192.168.11.130
Connected to 192.168.11.130 (192.168.11.130).
220 (vsFTPd 3.0.2)
Name (192.168.11.130:root): usertwo
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (192,168,11,130,102,90).
150 Here comes the directory listing.
drwx------    2 1001     1001            6 Jun 30 20:40 one
226 Directory send OK.
ftp> mkdir two
550 Permission denied.
ftp>
```

#### 数据库服务认证方式

以 mariadb 为例：

> 虚拟机恢复默认安装状态。

1、 安装 mariadb 以及 vsftpd

```bash
[root@localhost ~]# yum install -y mariadb-server.x86_64 vsftpd
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * extras: mirrors.163.com
 * updates: mirrors.163.com
正在解决依赖关系
--> 正在检查事务
---> 软件包 mariadb-server.x86_64.1.5.5.60-1.el7_5 将被 安装
--> 正在处理依赖关系 mariadb(x86-64) = 1:5.5.60-1.el7_5，它被软件包 1:mariadb-server-5.5.60-1.el7_5.x86_64 需要
--> 正在处理依赖关系 perl-DBI，它被软件包 1:mariadb-server-5.5.60-1.el7_5.x86_64 需要
--> 正在处理依赖关系 perl-DBD-MySQL，它被软件包 1:mariadb-server-5.5.60-1.el7_5.x86_64 需要
--> 正在处理依赖关系 perl(DBI)，它被软件包 1:mariadb-server-5.5.60-1.el7_5.x86_64 需要
---> 软件包 vsftpd.x86_64.0.3.0.2-25.el7 将被 安装
--> 正在检查事务
---> 软件包 mariadb.x86_64.1.5.5.60-1.el7_5 将被 安装
---> 软件包 perl-DBD-MySQL.x86_64.0.4.023-6.el7 将被 安装
---> 软件包 perl-DBI.x86_64.0.1.627-4.el7 将被 安装
--> 正在处理依赖关系 perl(RPC::PlServer) >= 0.2001，它被软件包 perl-DBI-1.627-4.el7.x86_64 需要
--> 正在处理依赖关系 perl(RPC::PlClient) >= 0.2000，它被软件包 perl-DBI-1.627-4.el7.x86_64 需要
--> 正在检查事务
---> 软件包 perl-PlRPC.noarch.0.0.2020-14.el7 将被 安装
--> 正在处理依赖关系 perl(Net::Daemon) >= 0.13，它被软件包 perl-PlRPC-0.2020-14.el7.noarch 需要
--> 正在处理依赖关系 perl(Net::Daemon::Test)，它被软件包 perl-PlRPC-0.2020-14.el7.noarch 需要
--> 正在处理依赖关系 perl(Net::Daemon::Log)，它被软件包 perl-PlRPC-0.2020-14.el7.noarch 需要
--> 正在处理依赖关系 perl(Compress::Zlib)，它被软件包 perl-PlRPC-0.2020-14.el7.noarch 需要
--> 正在检查事务
---> 软件包 perl-IO-Compress.noarch.0.2.061-2.el7 将被 安装
--> 正在处理依赖关系 perl(Compress::Raw::Zlib) >= 2.061，它被软件包 perl-IO-Compress-2.061-2.el7.noarch 需要
--> 正在处理依赖关系 perl(Compress::Raw::Bzip2) >= 2.061，它被软件包 perl-IO-Compress-2.061-2.el7.noarch 需要
---> 软件包 perl-Net-Daemon.noarch.0.0.48-5.el7 将被 安装
--> 正在检查事务
---> 软件包 perl-Compress-Raw-Bzip2.x86_64.0.2.061-3.el7 将被 安装
---> 软件包 perl-Compress-Raw-Zlib.x86_64.1.2.061-4.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

===============================================================================================================
 Package                             架构               版本                            源                大小
===============================================================================================================
正在安装:
 mariadb-server                      x86_64             1:5.5.60-1.el7_5                base              11 M
 vsftpd                              x86_64             3.0.2-25.el7                    base             171 k
为依赖而安装:
 mariadb                             x86_64             1:5.5.60-1.el7_5                base             8.9 M
 perl-Compress-Raw-Bzip2             x86_64             2.061-3.el7                     base              32 k
 perl-Compress-Raw-Zlib              x86_64             1:2.061-4.el7                   base              57 k
 perl-DBD-MySQL                      x86_64             4.023-6.el7                     base             140 k
 perl-DBI                            x86_64             1.627-4.el7                     base             802 k
 perl-IO-Compress                    noarch             2.061-2.el7                     base             260 k
 perl-Net-Daemon                     noarch             0.48-5.el7                      base              51 k
 perl-PlRPC                          noarch             0.2020-14.el7                   base              36 k

事务概要
===============================================================================================================
安装  2 软件包 (+8 依赖软件包)

总下载量：21 M
安装大小：111 M
Downloading packages:
(1/10): perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64.rpm                                  |  32 kB  00:00:00
(2/10): perl-Compress-Raw-Zlib-2.061-4.el7.x86_64.rpm                                   |  57 kB  00:00:00
(3/10): perl-DBD-MySQL-4.023-6.el7.x86_64.rpm                                           | 140 kB  00:00:00
(4/10): perl-Net-Daemon-0.48-5.el7.noarch.rpm                                           |  51 kB  00:00:00
(5/10): perl-PlRPC-0.2020-14.el7.noarch.rpm                                             |  36 kB  00:00:00
(6/10): perl-IO-Compress-2.061-2.el7.noarch.rpm                                         | 260 kB  00:00:00
(7/10): perl-DBI-1.627-4.el7.x86_64.rpm                                                 | 802 kB  00:00:00
(8/10): vsftpd-3.0.2-25.el7.x86_64.rpm                                                  | 171 kB  00:00:01
(9/10): mariadb-server-5.5.60-1.el7_5.x86_64.rpm                                        |  11 MB  00:00:02
(10/10): mariadb-5.5.60-1.el7_5.x86_64.rpm                                              | 8.9 MB  00:00:03
---------------------------------------------------------------------------------------------------------------
总计                                                                           6.7 MB/s |  21 MB  00:00:03
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64                                                1/10
  正在安装    : 1:mariadb-5.5.60-1.el7_5.x86_64                                                           2/10
  正在安装    : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64                                               3/10
  正在安装    : perl-IO-Compress-2.061-2.el7.noarch                                                       4/10
  正在安装    : perl-Net-Daemon-0.48-5.el7.noarch                                                         5/10
  正在安装    : perl-PlRPC-0.2020-14.el7.noarch                                                           6/10
  正在安装    : perl-DBI-1.627-4.el7.x86_64                                                               7/10
  正在安装    : perl-DBD-MySQL-4.023-6.el7.x86_64                                                         8/10
  正在安装    : 1:mariadb-server-5.5.60-1.el7_5.x86_64                                                    9/10
  正在安装    : vsftpd-3.0.2-25.el7.x86_64                                                               10/10
  验证中      : 1:mariadb-server-5.5.60-1.el7_5.x86_64                                                    1/10
  验证中      : vsftpd-3.0.2-25.el7.x86_64                                                                2/10
  验证中      : perl-Net-Daemon-0.48-5.el7.noarch                                                         3/10
  验证中      : perl-DBD-MySQL-4.023-6.el7.x86_64                                                         4/10
  验证中      : perl-IO-Compress-2.061-2.el7.noarch                                                       5/10
  验证中      : 1:perl-Compress-Raw-Zlib-2.061-4.el7.x86_64                                               6/10
  验证中      : 1:mariadb-5.5.60-1.el7_5.x86_64                                                           7/10
  验证中      : perl-DBI-1.627-4.el7.x86_64                                                               8/10
  验证中      : perl-Compress-Raw-Bzip2-2.061-3.el7.x86_64                                                9/10
  验证中      : perl-PlRPC-0.2020-14.el7.noarch                                                          10/10

已安装:
  mariadb-server.x86_64 1:5.5.60-1.el7_5                      vsftpd.x86_64 0:3.0.2-25.el7

作为依赖被安装:
  mariadb.x86_64 1:5.5.60-1.el7_5                       perl-Compress-Raw-Bzip2.x86_64 0:2.061-3.el7
  perl-Compress-Raw-Zlib.x86_64 1:2.061-4.el7           perl-DBD-MySQL.x86_64 0:4.023-6.el7
  perl-DBI.x86_64 0:1.627-4.el7                         perl-IO-Compress.noarch 0:2.061-2.el7
  perl-Net-Daemon.noarch 0:0.48-5.el7                   perl-PlRPC.noarch 0:0.2020-14.el7

完毕！
[root@localhost ~]# systemctl start mariadb.service vsftpd
[root@localhost ~]#
```

2、 建立用户口令数据库

```bash
[root@localhost ~]# mysql -uroot -p
Enter password:
Welcome to the MariaDB monitor.  Commands end with ; or \g.
Your MariaDB connection id is 3
Server version: 5.5.60-MariaDB MariaDB Server

Copyright (c) 2000, 2018, Oracle, MariaDB Corporation Ab and others.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

MariaDB [(none)]> create database vsftpuser;
Query OK, 1 row affected (0.00 sec)

MariaDB [(none)]> use vsftpuser;
Database changed
MariaDB [vsftpuser]> create table users(name char(16) binary, passwd char(16) binary);
Query OK, 0 rows affected (0.00 sec)

MariaDB [vsftpuser]> insert into users(name, passwd) values('userone',password('passwdone'));
Query OK, 1 row affected, 1 warning (0.00 sec)

MariaDB [vsftpuser]> insert into users(name,passwd) values('usertwo',password('passwdtwo'));
Query OK, 1 row affected, 1 warning (0.00 sec)

MariaDB [vsftpuser]> grant select on vsftpuser.users to vsftpuser@'192.168.11.%' identified by 'vsftppasswd';
Query OK, 0 rows affected (0.00 sec)

MariaDB [vsftpuser]> flush privileges;
Query OK, 0 rows affected (0.01 sec)

MariaDB [vsftpuser]> show tables;
+---------------------+
| Tables_in_vsftpuser |
+---------------------+
| users               |
+---------------------+
1 row in set (0.00 sec)

MariaDB [vsftpuser]> select * from users;
+---------+------------------+
| name    | passwd           |
+---------+------------------+
| userone | *6336F1A3C79741C |
| usertwo | *804FA602F20405D |
+---------+------------------+
2 rows in set (0.00 sec)

MariaDB [vsftpuser]> exit
Bye
[root@localhost ~]#
```

> 由于mariadb的安装方式不同，pam_mysql.so基于unix sock连接mariadb服务器时可能会出问题，可授权一个可远程连接的mariadb并访问vsftpd数据库的用户。

3、 安装 pam_mysql 认证模块

查看 `/lib64/security` 目录下有没有 mysql 对应的 pam 模块

```bash
[root@localhost ~]# ll /lib64/security/ |grep mysql
[root@localhost ~]#
```

安装 pam_myslq：

```bash
[root@localhost ~]# wget "ftp://ftp.pbone.net/mirror/archive.fedoraproject.org/fedora-secondary/releases/23/Everything/source/SRPMS/p/pam_mysql-0.7-0.20.rc1.fc23.src.rpm"
--2019-07-01 05:41:52--  ftp://ftp.pbone.net/mirror/archive.fedoraproject.org/fedora-secondary/releases/23/Everything/source/SRPMS/p/pam_mysql-0.7-0.20.rc1.fc23.src.rpm
           => “pam_mysql-0.7-0.20.rc1.fc23.src.rpm.1”
正在解析主机 ftp.pbone.net (ftp.pbone.net)... 93.179.225.212
正在连接 ftp.pbone.net (ftp.pbone.net)|93.179.225.212|:21... 已连接。
正在以 anonymous 登录 ... 登录成功！
==> SYST ... 完成。   ==> PWD ... 完成。
==> TYPE I ... 完成。 ==> CWD (1) /mirror/archive.fedoraproject.org/fedora-secondary/releases/23/Everything/source/SRPMS/p ... 完成。
==> SIZE pam_mysql-0.7-0.20.rc1.fc23.src.rpm ... 348672
==> PASV ... 完成。   ==> RETR pam_mysql-0.7-0.20.rc1.fc23.src.rpm ... 完成。
长度：348672 (340K) (非正式数据)

100%[=====================================================================>] 348,672     5.98KB/s 用时 43s

2019-07-01 05:42:40 (7.85 KB/s) - “pam_mysql-0.7-0.20.rc1.fc23.src.rpm.1” 已保存 [348672]

[root@localhost ~]# yum install -y make gcc-c++ autoconf automake libtool rpm-build pam-devel mysql-devel openssl-devel cyrus-sasl-devel
已加载插件：fastestmirror
Loading mirror speeds from cached hostfile
 * base: centos.ustc.edu.cn
 * extras: mirrors.163.com
 * updates: mirrors.163.com
软件包 1:make-3.82-23.el7.x86_64 已安装并且是最新版本
软件包 gcc-c++-4.8.5-36.el7_6.2.x86_64 已安装并且是最新版本
软件包 autoconf-2.69-11.el7.noarch 已安装并且是最新版本
软件包 automake-1.13.4-3.el7.noarch 已安装并且是最新版本
软件包 libtool-2.4.2-22.el7_3.x86_64 已安装并且是最新版本
软件包 rpm-build-4.11.3-35.el7.x86_64 已安装并且是最新版本
正在解决依赖关系
--> 正在检查事务
---> 软件包 cyrus-sasl-devel.x86_64.0.2.1.26-23.el7 将被 安装
--> 正在处理依赖关系 cyrus-sasl(x86-64) = 2.1.26-23.el7，它被软件包 cyrus-sasl-devel-2.1.26-23.el7.x86_64 需要
---> 软件包 mariadb-devel.x86_64.1.5.5.60-1.el7_5 将被 安装
---> 软件包 openssl-devel.x86_64.1.1.0.2k-16.el7_6.1 将被 安装
--> 正在处理依赖关系 zlib-devel(x86-64)，它被软件包 1:openssl-devel-1.0.2k-16.el7_6.1.x86_64 需要
--> 正在处理依赖关系 krb5-devel(x86-64)，它被软件包 1:openssl-devel-1.0.2k-16.el7_6.1.x86_64 需要
---> 软件包 pam-devel.x86_64.0.1.1.8-22.el7 将被 安装
--> 正在检查事务
---> 软件包 cyrus-sasl.x86_64.0.2.1.26-23.el7 将被 安装
---> 软件包 krb5-devel.x86_64.0.1.15.1-37.el7_6 将被 安装
--> 正在处理依赖关系 libkadm5(x86-64) = 1.15.1-37.el7_6，它被软件包 krb5-devel-1.15.1-37.el7_6.x86_64 需要
--> 正在处理依赖关系 libverto-devel，它被软件包 krb5-devel-1.15.1-37.el7_6.x86_64 需要
--> 正在处理依赖关系 libselinux-devel，它被软件包 krb5-devel-1.15.1-37.el7_6.x86_64 需要
--> 正在处理依赖关系 libcom_err-devel，它被软件包 krb5-devel-1.15.1-37.el7_6.x86_64 需要
--> 正在处理依赖关系 keyutils-libs-devel，它被软件包 krb5-devel-1.15.1-37.el7_6.x86_64 需要
---> 软件包 zlib-devel.x86_64.0.1.2.7-18.el7 将被 安装
--> 正在检查事务
---> 软件包 keyutils-libs-devel.x86_64.0.1.5.8-3.el7 将被 安装
---> 软件包 libcom_err-devel.x86_64.0.1.42.9-13.el7 将被 安装
---> 软件包 libkadm5.x86_64.0.1.15.1-37.el7_6 将被 安装
---> 软件包 libselinux-devel.x86_64.0.2.5-14.1.el7 将被 安装
--> 正在处理依赖关系 libsepol-devel(x86-64) >= 2.5-10，它被软件包 libselinux-devel-2.5-14.1.el7.x86_64 需要
--> 正在处理依赖关系 pkgconfig(libsepol)，它被软件包 libselinux-devel-2.5-14.1.el7.x86_64 需要
--> 正在处理依赖关系 pkgconfig(libpcre)，它被软件包 libselinux-devel-2.5-14.1.el7.x86_64 需要
---> 软件包 libverto-devel.x86_64.0.0.2.5-4.el7 将被 安装
--> 正在检查事务
---> 软件包 libsepol-devel.x86_64.0.2.5-10.el7 将被 安装
---> 软件包 pcre-devel.x86_64.0.8.32-17.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================================================================================================================
 Package                                         架构                               版本                                              源                                   大小
================================================================================================================================================================================
正在安装:
 cyrus-sasl-devel                                x86_64                             2.1.26-23.el7                                     base                                310 k
 mariadb-devel                                   x86_64                             1:5.5.60-1.el7_5                                  base                                754 k
 openssl-devel                                   x86_64                             1:1.0.2k-16.el7_6.1                               updates                             1.5 M
 pam-devel                                       x86_64                             1.1.8-22.el7                                      base                                184 k
为依赖而安装:
 cyrus-sasl                                      x86_64                             2.1.26-23.el7                                     base                                 88 k
 keyutils-libs-devel                             x86_64                             1.5.8-3.el7                                       base                                 37 k
 krb5-devel                                      x86_64                             1.15.1-37.el7_6                                   updates                             271 k
 libcom_err-devel                                x86_64                             1.42.9-13.el7                                     base                                 31 k
 libkadm5                                        x86_64                             1.15.1-37.el7_6                                   updates                             178 k
 libselinux-devel                                x86_64                             2.5-14.1.el7                                      base                                187 k
 libsepol-devel                                  x86_64                             2.5-10.el7                                        base                                 77 k
 libverto-devel                                  x86_64                             0.2.5-4.el7                                       base                                 12 k
 pcre-devel                                      x86_64                             8.32-17.el7                                       base                                480 k
 zlib-devel                                      x86_64                             1.2.7-18.el7                                      base                                 50 k

事务概要
================================================================================================================================================================================
安装  4 软件包 (+10 依赖软件包)

总下载量：4.1 M
安装大小：11 M
Downloading packages:
(1/14): keyutils-libs-devel-1.5.8-3.el7.x86_64.rpm                                                                                                       |  37 kB  00:00:00
(2/14): libcom_err-devel-1.42.9-13.el7.x86_64.rpm                                                                                                        |  31 kB  00:00:00
(3/14): cyrus-sasl-2.1.26-23.el7.x86_64.rpm                                                                                                              |  88 kB  00:00:00
(4/14): libselinux-devel-2.5-14.1.el7.x86_64.rpm                                                                                                         | 187 kB  00:00:00
(5/14): libverto-devel-0.2.5-4.el7.x86_64.rpm                                                                                                            |  12 kB  00:00:00
(6/14): cyrus-sasl-devel-2.1.26-23.el7.x86_64.rpm                                                                                                        | 310 kB  00:00:00
(7/14): mariadb-devel-5.5.60-1.el7_5.x86_64.rpm                                                                                                          | 754 kB  00:00:00
(8/14): libsepol-devel-2.5-10.el7.x86_64.rpm                                                                                                             |  77 kB  00:00:00
(9/14): libkadm5-1.15.1-37.el7_6.x86_64.rpm                                                                                                              | 178 kB  00:00:00
(10/14): zlib-devel-1.2.7-18.el7.x86_64.rpm                                                                                                              |  50 kB  00:00:00
(11/14): krb5-devel-1.15.1-37.el7_6.x86_64.rpm                                                                                                           | 271 kB  00:00:01
(12/14): pam-devel-1.1.8-22.el7.x86_64.rpm                                                                                                               | 184 kB  00:00:00
(13/14): pcre-devel-8.32-17.el7.x86_64.rpm                                                                                                               | 480 kB  00:00:00
(14/14): openssl-devel-1.0.2k-16.el7_6.1.x86_64.rpm                                                                                                      | 1.5 MB  00:00:01
--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------
总计                                                                                                                                            1.9 MB/s | 4.1 MB  00:00:02
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : libcom_err-devel-1.42.9-13.el7.x86_64                                                                                                                      1/14
  正在安装    : libsepol-devel-2.5-10.el7.x86_64                                                                                                                           2/14
  正在安装    : cyrus-sasl-2.1.26-23.el7.x86_64                                                                                                                            3/14
  正在安装    : pcre-devel-8.32-17.el7.x86_64                                                                                                                              4/14
  正在安装    : libselinux-devel-2.5-14.1.el7.x86_64                                                                                                                       5/14
  正在安装    : libkadm5-1.15.1-37.el7_6.x86_64                                                                                                                            6/14
  正在安装    : zlib-devel-1.2.7-18.el7.x86_64                                                                                                                             7/14
  正在安装    : libverto-devel-0.2.5-4.el7.x86_64                                                                                                                          8/14
  正在安装    : keyutils-libs-devel-1.5.8-3.el7.x86_64                                                                                                                     9/14
  正在安装    : krb5-devel-1.15.1-37.el7_6.x86_64                                                                                                                         10/14
  正在安装    : 1:openssl-devel-1.0.2k-16.el7_6.1.x86_64                                                                                                                  11/14
  正在安装    : 1:mariadb-devel-5.5.60-1.el7_5.x86_64                                                                                                                     12/14
  正在安装    : cyrus-sasl-devel-2.1.26-23.el7.x86_64                                                                                                                     13/14
  正在安装    : pam-devel-1.1.8-22.el7.x86_64                                                                                                                             14/14
  验证中      : pam-devel-1.1.8-22.el7.x86_64                                                                                                                              1/14
  验证中      : keyutils-libs-devel-1.5.8-3.el7.x86_64                                                                                                                     2/14
  验证中      : libverto-devel-0.2.5-4.el7.x86_64                                                                                                                          3/14
  验证中      : zlib-devel-1.2.7-18.el7.x86_64                                                                                                                             4/14
  验证中      : libkadm5-1.15.1-37.el7_6.x86_64                                                                                                                            5/14
  验证中      : krb5-devel-1.15.1-37.el7_6.x86_64                                                                                                                          6/14
  验证中      : pcre-devel-8.32-17.el7.x86_64                                                                                                                              7/14
  验证中      : libselinux-devel-2.5-14.1.el7.x86_64                                                                                                                       8/14
  验证中      : cyrus-sasl-2.1.26-23.el7.x86_64                                                                                                                            9/14
  验证中      : cyrus-sasl-devel-2.1.26-23.el7.x86_64                                                                                                                     10/14
  验证中      : libsepol-devel-2.5-10.el7.x86_64                                                                                                                          11/14
  验证中      : libcom_err-devel-1.42.9-13.el7.x86_64                                                                                                                     12/14
  验证中      : 1:openssl-devel-1.0.2k-16.el7_6.1.x86_64                                                                                                                  13/14
  验证中      : 1:mariadb-devel-5.5.60-1.el7_5.x86_64                                                                                                                     14/14

已安装:
  cyrus-sasl-devel.x86_64 0:2.1.26-23.el7       mariadb-devel.x86_64 1:5.5.60-1.el7_5       openssl-devel.x86_64 1:1.0.2k-16.el7_6.1       pam-devel.x86_64 0:1.1.8-22.el7

作为依赖被安装:
  cyrus-sasl.x86_64 0:2.1.26-23.el7       keyutils-libs-devel.x86_64 0:1.5.8-3.el7       krb5-devel.x86_64 0:1.15.1-37.el7_6       libcom_err-devel.x86_64 0:1.42.9-13.el7
  libkadm5.x86_64 0:1.15.1-37.el7_6       libselinux-devel.x86_64 0:2.5-14.1.el7         libsepol-devel.x86_64 0:2.5-10.el7        libverto-devel.x86_64 0:0.2.5-4.el7
  pcre-devel.x86_64 0:8.32-17.el7         zlib-devel.x86_64 0:1.2.7-18.el7

完毕！
[root@localhost ~]# rpmbuild --rebuild pam_mysql-0.7-0.20.rc1.fc23.src.rpm
正在安装 pam_mysql-0.7-0.20.rc1.fc23.src.rpm
警告：pam_mysql-0.7-0.20.rc1.fc23.src.rpm: 头V3 RSA/SHA1 Signature, 密钥 ID 873529b8: NOKEY
警告：用户mockbuild 不存在 - 使用root
警告：群组mockbuild 不存在 - 使用root
警告：用户mockbuild 不存在 - 使用root
警告：群组mockbuild 不存在 - 使用root
警告：用户mockbuild 不存在 - 使用root
警告：群组mockbuild 不存在 - 使用root
警告：用户mockbuild 不存在 - 使用root
警告：群组mockbuild 不存在 - 使用root
警告：用户mockbuild 不存在 - 使用root
警告：群组mockbuild 不存在 - 使用root
警告：%changelog （更新日志）中存在虚假的日期：Thu Jan 9 2008 lonely wolf <wolfy at fedoraproject dot org> - 0.7-0.3.rc1.1
警告：%changelog （更新日志）中存在虚假的日期：Fri Nov 13 2005 Ignacio Vazquez-Abrams <ivazquez@ivazquez.net> 0.6.2-2
警告：%changelog （更新日志）中存在虚假的日期：Fri Nov 13 2005 Ignacio Vazquez-Abrams <ivazquez@ivazquez.net> 0.6.2-1
警告：%changelog （更新日志）中存在虚假的日期：Wed Mar 15 2005 Ignacio Vazquez-Abrams <ivazquez@ivazquez.net> 0.50-3
执行(%prep): /bin/sh -e /var/tmp/rpm-tmp.a2kWEm
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd /root/rpmbuild/BUILD
+ rm -rf pam_mysql-0.7RC1
+ /usr/bin/gzip -dc /root/rpmbuild/SOURCES/pam_mysql-0.7RC1.tar.gz
+ /usr/bin/tar -xf -
+ STATUS=0
+ '[' 0 -ne 0 ']'
+ cd pam_mysql-0.7RC1
+ /usr/bin/chmod -Rf a+rX,u+w,g-w,o-w .
+ echo 'Patch #0 (pam_mysql-0.7RC1-resps-segfault.patch):'
Patch #0 (pam_mysql-0.7RC1-resps-segfault.patch):
+ /usr/bin/cat /root/rpmbuild/SOURCES/pam_mysql-0.7RC1-resps-segfault.patch
+ /usr/bin/patch -p1 --fuzz=0
patching file pam_mysql.c
+ echo 'Patch #1 (pam_mysql-0.7RC1-first-pass.patch):'
Patch #1 (pam_mysql-0.7RC1-first-pass.patch):
+ /usr/bin/patch -p1 --fuzz=0
+ /usr/bin/cat /root/rpmbuild/SOURCES/pam_mysql-0.7RC1-first-pass.patch
patching file pam_mysql.c
+ echo 'Patch #2 (pam_mysql-0.7RC1-scrambled.patch):'
Patch #2 (pam_mysql-0.7RC1-scrambled.patch):
+ /usr/bin/cat /root/rpmbuild/SOURCES/pam_mysql-0.7RC1-scrambled.patch
+ /usr/bin/patch -p1 --fuzz=0
patching file pam_mysql.c
+ mv CREDITS AUTHORS
+ autoreconf -fiv
autoreconf: Entering directory `.'’
autoreconf: configure.in: not using Gettext
autoreconf: running: aclocal --force
aclocal: warning: autoconf input should be named 'configure.ac', not 'configure.in'
autoreconf: configure.in: tracing
autoreconf: running: libtoolize --copy --force
libtoolize: putting auxiliary files in `.'.
libtoolize: copying file `./ltmain.sh'
libtoolize: Consider adding `AC_CONFIG_MACRO_DIR([m4])' to configure.in and
libtoolize: rerunning libtoolize, to keep the correct libtool macros in-tree.
libtoolize: Consider adding `-I m4' to ACLOCAL_AMFLAGS in Makefile.am.
aclocal: warning: autoconf input should be named 'configure.ac', not 'configure.in'
autoreconf: running: /usr/bin/autoconf --force
autoreconf: running: /usr/bin/autoheader --force
autoreconf: running: automake --add-missing --copy --force-missing
automake: warning: autoconf input should be named 'configure.ac', not 'configure.in'
configure.in:6: warning: AM_INIT_AUTOMAKE: two- and three-arguments forms are deprecated.  For more info, see:
configure.in:6: http://www.gnu.org/software/automake/manual/automake.html#Modernize-AM_005fINIT_005fAUTOMAKE-invocation
Makefile.am:6: warning: 'INCLUDES' is the old name for 'AM_CPPFLAGS' (or '*_CPPFLAGS')
automake: warning: autoconf input should be named 'configure.ac', not 'configure.in'
Makefile.am: installing './depcomp'
autoreconf: Leaving directory `.'
+ exit 0
执行(%build): /bin/sh -e /var/tmp/rpm-tmp.0XdncS
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd pam_mysql-0.7RC1
+ CFLAGS='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches   -m64 -mtune=generic'
+ export CFLAGS
+ CXXFLAGS='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches   -m64 -mtune=generic'
+ export CXXFLAGS
+ FFLAGS='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches   -m64 -mtune=generic -I/usr/lib64/gfortran/modules'
+ export FFLAGS
+ FCFLAGS='-O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches   -m64 -mtune=generic -I/usr/lib64/gfortran/modules'
+ export FCFLAGS
+ LDFLAGS='-Wl,-z,relro '
+ export LDFLAGS
+ '[' 1 == 1 ']'
+ '[' x86_64 == ppc64le ']'
++ find . -name config.guess -o -name config.sub
+ for i in '$(find . -name config.guess -o -name config.sub)'
++ basename ./config.guess
+ '[' -f /usr/lib/rpm/redhat/config.guess ']'
+ /usr/bin/rm -f ./config.guess
++ basename ./config.guess
+ /usr/bin/cp -fv /usr/lib/rpm/redhat/config.guess ./config.guess
‘/usr/lib/rpm/redhat/config.guess’ -> ‘./config.guess’
+ for i in '$(find . -name config.guess -o -name config.sub)'
++ basename ./config.sub
+ '[' -f /usr/lib/rpm/redhat/config.sub ']'
+ /usr/bin/rm -f ./config.sub
++ basename ./config.sub
+ /usr/bin/cp -fv /usr/lib/rpm/redhat/config.sub ./config.sub
‘/usr/lib/rpm/redhat/config.sub’ -> ‘./config.sub’
+ ./configure --build=x86_64-redhat-linux-gnu --host=x86_64-redhat-linux-gnu --program-prefix= --disable-dependency-tracking --prefix=/usr --exec-prefix=/usr --bindir=/usr/bin --sbindir=/usr/sbin --sysconfdir=/etc --datadir=/usr/share --includedir=/usr/include --libdir=/usr/lib64 --libexecdir=/usr/libexec --localstatedir=/var --sharedstatedir=/var/lib --mandir=/usr/share/man --infodir=/usr/share/info --with-openssl --with-pam-mods-dir=/lib64/security --enable-static=no --with-cyrus-sasl2
checking for a BSD-compatible install... /usr/bin/install -c
checking whether build environment is sane... yes
checking for a thread-safe mkdir -p... /usr/bin/mkdir -p
checking for gawk... gawk
checking whether make sets $(MAKE)... yes
checking whether make supports nested variables... yes
checking whether to enable maintainer-specific portions of Makefiles... no
checking for bison... bison -y
checking for x86_64-redhat-linux-gnu-g++... no
checking for x86_64-redhat-linux-gnu-c++... no
checking for x86_64-redhat-linux-gnu-gpp... no
checking for x86_64-redhat-linux-gnu-aCC... no
checking for x86_64-redhat-linux-gnu-CC... no
checking for x86_64-redhat-linux-gnu-cxx... no
checking for x86_64-redhat-linux-gnu-cc++... no
checking for x86_64-redhat-linux-gnu-cl.exe... no
checking for x86_64-redhat-linux-gnu-FCC... no
checking for x86_64-redhat-linux-gnu-KCC... no
checking for x86_64-redhat-linux-gnu-RCC... no
checking for x86_64-redhat-linux-gnu-xlC_r... no
checking for x86_64-redhat-linux-gnu-xlC... no
checking for g++... g++
checking whether the C++ compiler works... yes
checking for C++ compiler default output file name... a.out
checking for suffix of executables...
checking whether we are cross compiling... no
checking for suffix of object files... o
checking whether we are using the GNU C++ compiler... yes
checking whether g++ accepts -g... yes
checking for style of include used by make... GNU
checking dependency style of g++... none
checking for x86_64-redhat-linux-gnu-gcc... no
checking for gcc... gcc
checking whether we are using the GNU C compiler... yes
checking whether gcc accepts -g... yes
checking for gcc option to accept ISO C89... none needed
checking dependency style of gcc... none
checking how to run the C preprocessor... gcc -E
checking whether ln -s works... yes
checking whether make sets $(MAKE)... (cached) yes
checking build system type... x86_64-redhat-linux-gnu
checking host system type... x86_64-redhat-linux-gnu
checking how to print strings... printf
checking for a sed that does not truncate output... /usr/bin/sed
checking for grep that handles long lines and -e... /usr/bin/grep
checking for egrep... /usr/bin/grep -E
checking for fgrep... /usr/bin/grep -F
checking for ld used by gcc... /usr/bin/ld
checking if the linker (/usr/bin/ld) is GNU ld... yes
checking for BSD- or MS-compatible name lister (nm)... /usr/bin/nm -B
checking the name lister (/usr/bin/nm -B) interface... BSD nm
checking the maximum length of command line arguments... 1572864
checking whether the shell understands some XSI constructs... yes
checking whether the shell understands "+="... yes
checking how to convert x86_64-redhat-linux-gnu file names to x86_64-redhat-linux-gnu format... func_convert_file_noop
checking how to convert x86_64-redhat-linux-gnu file names to toolchain format... func_convert_file_noop
checking for /usr/bin/ld option to reload object files... -r
checking for x86_64-redhat-linux-gnu-objdump... no
checking for objdump... objdump
checking how to recognize dependent libraries... pass_all
checking for x86_64-redhat-linux-gnu-dlltool... no
checking for dlltool... no
checking how to associate runtime and link libraries... printf %s\n
checking for x86_64-redhat-linux-gnu-ar... no
checking for ar... ar
checking for archiver @FILE support... @
checking for x86_64-redhat-linux-gnu-strip... no
checking for strip... strip
checking for x86_64-redhat-linux-gnu-ranlib... no
checking for ranlib... ranlib
checking command to parse /usr/bin/nm -B output from gcc object... ok
checking for sysroot... no
checking for x86_64-redhat-linux-gnu-mt... no
checking for mt... no
checking if : is a manifest tool... no
checking for ANSI C header files... yes
checking for sys/types.h... yes
checking for sys/stat.h... yes
checking for stdlib.h... yes
checking for string.h... yes
checking for memory.h... yes
checking for strings.h... yes
checking for inttypes.h... yes
checking for stdint.h... yes
checking for unistd.h... yes
checking for dlfcn.h... yes
checking for objdir... .libs
checking if gcc supports -fno-rtti -fno-exceptions... no
checking for gcc option to produce PIC... -fPIC -DPIC
checking if gcc PIC flag -fPIC -DPIC works... yes
checking if gcc static flag -static works... no
checking if gcc supports -c -o file.o... yes
checking if gcc supports -c -o file.o... (cached) yes
checking whether the gcc linker (/usr/bin/ld -m elf_x86_64) supports shared libraries... yes
checking whether -lc should be explicitly linked in... no
checking dynamic linker characteristics... GNU/Linux ld.so
checking how to hardcode library paths into programs... immediate
checking whether stripping libraries is possible... yes
checking if libtool supports shared libraries... yes
checking whether to build shared libraries... yes
checking whether to build static libraries... no
checking how to run the C++ preprocessor... g++ -E
checking for ld used by g++... /usr/bin/ld -m elf_x86_64
checking if the linker (/usr/bin/ld -m elf_x86_64) is GNU ld... yes
checking whether the g++ linker (/usr/bin/ld -m elf_x86_64) supports shared libraries... yes
checking for g++ option to produce PIC... -fPIC -DPIC
checking if g++ PIC flag -fPIC -DPIC works... yes
checking if g++ static flag -static works... no
checking if g++ supports -c -o file.o... yes
checking if g++ supports -c -o file.o... (cached) yes
checking whether the g++ linker (/usr/bin/ld -m elf_x86_64) supports shared libraries... yes
checking dynamic linker characteristics... (cached) GNU/Linux ld.so
checking how to hardcode library paths into programs... immediate
checking for ANSI C header files... (cached) yes
checking arpa/inet.h usability... yes
checking arpa/inet.h presence... yes
checking for arpa/inet.h... yes
checking netinet/in.h usability... yes
checking netinet/in.h presence... yes
checking for netinet/in.h... yes
checking netdb.h usability... yes
checking netdb.h presence... yes
checking for netdb.h... yes
checking for string.h... (cached) yes
checking for strings.h... (cached) yes
checking sys/socket.h usability... yes
checking sys/socket.h presence... yes
checking for sys/socket.h... yes
checking for sys/types.h... (cached) yes
checking for sys/stat.h... (cached) yes
checking sys/param.h usability... yes
checking sys/param.h presence... yes
checking for sys/param.h... yes
checking fcntl.h usability... yes
checking fcntl.h presence... yes
checking for fcntl.h... yes
checking syslog.h usability... yes
checking syslog.h presence... yes
checking for syslog.h... yes
checking for unistd.h... (cached) yes
checking stdarg.h usability... yes
checking stdarg.h presence... yes
checking for stdarg.h... yes
checking errno.h usability... yes
checking errno.h presence... yes
checking for errno.h... yes
checking crypt.h usability... yes
checking crypt.h presence... yes
checking for crypt.h... yes
checking for stdbool.h that conforms to C99... yes
checking for _Bool... yes
checking for an ANSI C-conforming const... yes
checking for inline... inline
checking for size_t... yes
checking for working volatile... yes
checking for preprocessor stringizing operator... yes
checking for pid_t... yes
checking for working alloca.h... yes
checking for alloca... yes
checking for stdlib.h... (cached) yes
checking for GNU libc compatible malloc... yes
checking for working memcmp... yes
checking ELOOP availability... yes
checking EOVERFLOW availability... yes
checking for socket in -lsocket... no
checking for gethostbyname... yes
checking for gethostbyname in -lnsl... no
checking for inet_ntop... yes
checking for getaddrinfo... yes
checking for freeaddrinfo... yes
checking for strcasecmp... yes
checking for strdup... yes
checking PF_INET6 availability... yes
checking for struct sockaddr_in6... yes
checking for struct in6_addr... yes
checking for gethostbyname_r... yes
checking if gethostbyname_r() is part of glibc... yes
checking size of char... 1
checking size of short... 2
checking size of int... 4
checking size of long... 8
checking if /usr /usr/local /usr/mysql /opt/mysql is a mysql_config script... no
checking mysql_config availability in /usr/bin... yes
checking for mysql_real_query... yes
checking for mysql_real_escape_string... yes
checking for make_scrambled_password_323... yes
checking for x86_64-redhat-linux-gnu-pkg-config... no
checking for pkg-config... /usr/bin/pkg-config
checking pkg-config is at least version 0.9.0... yes
checking for openssl_CFLAGS...
checking for openssl_LIBS... -lssl -lcrypto
checking pam_appl.h usability... yes
checking pam_appl.h presence... yes
checking for pam_appl.h... yes
checking pam_modules.h usability... yes
checking PAM_CONV_AGAIN availability... yes
checking PAM_INCOMPLETE availability... yes
checking PAM_NEW_AUTHTOK_REQD availability... yes
checking PAM_AUTHTOK_RECOVERY_ERR availability... yes
checking if the second argument of pam_get_user() takes const pointer... no
checking if the third argument of pam_get_data() takes const pointer... no
checking if the third argument of pam_get_item() takes const pointer... no
checking if the second argument of pam_conv.conv() takes const pointer... no
checking if md5.h is derived from Cyrus SASL Version 1... no
checking md5.h usability... no
checking md5.h presence... no
checking for md5.h... no
checking if md5.h is Solaris's... no
checking for md5.h... (cached) no
checking for MD5Data... no
checking for crypt in -lcrypt... yes
checking for crypt... yes
checking that generated files are newer than configure... done
configure: creating ./config.status
config.status: creating Makefile
config.status: creating pam_mysql.spec
config.status: creating config.h
config.status: executing depfiles commands
config.status: executing libtool commands
+ make -j4
make  all-am
make[1]: Entering directory `/root/rpmbuild/BUILD/pam_mysql-0.7RC1'
/bin/sh ./libtool  --tag=CC   --mode=compile gcc -DHAVE_CONFIG_H -I. -I/usr/include/security -I/usr/include  -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches   -m64 -mtune=generic  -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches   -m64 -mtune=generic -I/usr/include/mysql     -c -o pam_mysql.lo pam_mysql.c
libtool: compile:  gcc -DHAVE_CONFIG_H -I. -I/usr/include/security -I/usr/include -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches -m64 -mtune=generic -I/usr/include/mysql -c pam_mysql.c  -fPIC -DPIC -o .libs/pam_mysql.o
pam_mysql.c: In function 'pam_mysql_retrieve_ctx':
pam_mysql.c:2074:5: warning: passing argument 3 of 'pam_get_data' from incompatible pointer type [enabled by default]
     (PAM_GET_DATA_CONST void**)pretval)) {
     ^
In file included from pam_mysql.c:156:0:
/usr/include/security/pam_modules.h:27:1: note: expected 'const void **' but argument is of type 'void **'
 pam_get_data(const pam_handle_t *pamh, const char *module_data_name,
 ^
pam_mysql.c: In function 'pam_mysql_converse':
pam_mysql.c:3153:4: warning: passing argument 3 of 'pam_get_item' from incompatible pointer type [enabled by default]
    (PAM_GET_ITEM_CONST void **)&conv))) {
    ^
In file included from /usr/include/security/pam_appl.h:18:0,
                 from pam_mysql.c:155:
/usr/include/security/_pam_types.h:175:1: note: expected 'const void **' but argument is of type 'void **'
 pam_get_item(const pam_handle_t *pamh, int item_type, const void **item);
 ^
pam_mysql.c:3197:4: warning: passing argument 2 of 'conv->conv' from incompatible pointer type [enabled by default]
    conv->appdata_ptr))) {
    ^
pam_mysql.c:3197:4: note: expected 'const struct pam_message **' but argument is of type 'struct pam_message **'
pam_mysql.c: In function 'pam_sm_authenticate':
pam_mysql.c:3332:4: warning: passing argument 2 of 'pam_get_user' from incompatible pointer type [enabled by default]
    NULL))) {
    ^
In file included from pam_mysql.c:156:0:
/usr/include/security/pam_modules.h:31:1: note: expected 'const char **' but argument is of type 'char **'
 pam_get_user(pam_handle_t *pamh, const char **user, const char *prompt);
 ^
pam_mysql.c:3343:4: warning: passing argument 3 of 'pam_get_item' from incompatible pointer type [enabled by default]
    (PAM_GET_ITEM_CONST void **)&rhost)) {
    ^
In file included from /usr/include/security/pam_appl.h:18:0,
                 from pam_mysql.c:155:
/usr/include/security/_pam_types.h:175:1: note: expected 'const void **' but argument is of type 'void **'
 pam_get_item(const pam_handle_t *pamh, int item_type, const void **item);
 ^
pam_mysql.c:3353:5: warning: passing argument 3 of 'pam_get_item' from incompatible pointer type [enabled by default]
     (PAM_GET_ITEM_CONST void **)&passwd);
     ^
In file included from /usr/include/security/pam_appl.h:18:0,
                 from pam_mysql.c:155:
/usr/include/security/_pam_types.h:175:1: note: expected 'const void **' but argument is of type 'void **'
 pam_get_item(const pam_handle_t *pamh, int item_type, const void **item);
 ^
pam_mysql.c: In function 'pam_sm_acct_mgmt':
pam_mysql.c:3578:4: warning: passing argument 2 of 'pam_get_user' from incompatible pointer type [enabled by default]
    NULL))) {
    ^
In file included from pam_mysql.c:156:0:
/usr/include/security/pam_modules.h:31:1: note: expected 'const char **' but argument is of type 'char **'
 pam_get_user(pam_handle_t *pamh, const char **user, const char *prompt);
 ^
pam_mysql.c:3589:4: warning: passing argument 3 of 'pam_get_item' from incompatible pointer type [enabled by default]
    (PAM_GET_ITEM_CONST void **)&rhost)) {
    ^
In file included from /usr/include/security/pam_appl.h:18:0,
                 from pam_mysql.c:155:
/usr/include/security/_pam_types.h:175:1: note: expected 'const void **' but argument is of type 'void **'
 pam_get_item(const pam_handle_t *pamh, int item_type, const void **item);
 ^
pam_mysql.c: In function 'pam_sm_chauthtok':
pam_mysql.c:3739:4: warning: passing argument 2 of 'pam_get_user' from incompatible pointer type [enabled by default]
    NULL))) {
    ^
In file included from pam_mysql.c:156:0:
/usr/include/security/pam_modules.h:31:1: note: expected 'const char **' but argument is of type 'char **'
 pam_get_user(pam_handle_t *pamh, const char **user, const char *prompt);
 ^
pam_mysql.c:3750:4: warning: passing argument 3 of 'pam_get_item' from incompatible pointer type [enabled by default]
    (PAM_GET_ITEM_CONST void **)&rhost)) {
    ^
In file included from /usr/include/security/pam_appl.h:18:0,
                 from pam_mysql.c:155:
/usr/include/security/_pam_types.h:175:1: note: expected 'const void **' but argument is of type 'void **'
 pam_get_item(const pam_handle_t *pamh, int item_type, const void **item);
 ^
pam_mysql.c:3850:6: warning: passing argument 3 of 'pam_get_item' from incompatible pointer type [enabled by default]
      (PAM_GET_ITEM_CONST void **)&old_passwd);
      ^
In file included from /usr/include/security/pam_appl.h:18:0,
                 from pam_mysql.c:155:
/usr/include/security/_pam_types.h:175:1: note: expected 'const void **' but argument is of type 'void **'
 pam_get_item(const pam_handle_t *pamh, int item_type, const void **item);
 ^
pam_mysql.c:3950:4: warning: passing argument 3 of 'pam_get_item' from incompatible pointer type [enabled by default]
    (PAM_GET_ITEM_CONST void **)&new_passwd);
    ^
In file included from /usr/include/security/pam_appl.h:18:0,
                 from pam_mysql.c:155:
/usr/include/security/_pam_types.h:175:1: note: expected 'const void **' but argument is of type 'void **'
 pam_get_item(const pam_handle_t *pamh, int item_type, const void **item);
 ^
pam_mysql.c: In function 'pam_sm_open_session':
pam_mysql.c:4112:4: warning: passing argument 2 of 'pam_get_user' from incompatible pointer type [enabled by default]
    NULL))) {
    ^
In file included from pam_mysql.c:156:0:
/usr/include/security/pam_modules.h:31:1: note: expected 'const char **' but argument is of type 'char **'
 pam_get_user(pam_handle_t *pamh, const char **user, const char *prompt);
 ^
pam_mysql.c:4123:4: warning: passing argument 3 of 'pam_get_item' from incompatible pointer type [enabled by default]
    (PAM_GET_ITEM_CONST void **)&rhost)) {
    ^
In file included from /usr/include/security/pam_appl.h:18:0,
                 from pam_mysql.c:155:
/usr/include/security/_pam_types.h:175:1: note: expected 'const void **' but argument is of type 'void **'
 pam_get_item(const pam_handle_t *pamh, int item_type, const void **item);
 ^
pam_mysql.c: In function 'pam_sm_close_session':
pam_mysql.c:4215:4: warning: passing argument 2 of 'pam_get_user' from incompatible pointer type [enabled by default]
    NULL))) {
    ^
In file included from pam_mysql.c:156:0:
/usr/include/security/pam_modules.h:31:1: note: expected 'const char **' but argument is of type 'char **'
 pam_get_user(pam_handle_t *pamh, const char **user, const char *prompt);
 ^
pam_mysql.c:4226:4: warning: passing argument 3 of 'pam_get_item' from incompatible pointer type [enabled by default]
    (PAM_GET_ITEM_CONST void **)&rhost)) {
    ^
In file included from /usr/include/security/pam_appl.h:18:0,
                 from pam_mysql.c:155:
/usr/include/security/_pam_types.h:175:1: note: expected 'const void **' but argument is of type 'void **'
 pam_get_item(const pam_handle_t *pamh, int item_type, const void **item);
 ^
/bin/sh ./libtool  --tag=CC   --mode=link gcc  -O2 -g -pipe -Wall -Wp,-D_FORTIFY_SOURCE=2 -fexceptions -fstack-protector-strong --param=ssp-buffer-size=4 -grecord-gcc-switches   -m64 -mtune=generic -I/usr/include/mysql     -module -avoid-version -Wl,-z,relro  -o pam_mysql.la -rpath /lib64/security pam_mysql.lo  -L/usr/lib64/mysql -lmysqlclient -lpthread -lz -lm -ldl -lssl -lcrypto -lssl -lcrypto     -lcrypt
libtool: link: gcc -shared  -fPIC -DPIC  .libs/pam_mysql.o   -L/usr/lib64/mysql -lmysqlclient -lpthread -lz -lm -ldl -lssl -lcrypto -lcrypt  -O2 -m64 -mtune=generic -Wl,-z -Wl,relro   -Wl,-soname -Wl,pam_mysql.so -o .libs/pam_mysql.so
libtool: link: ( cd ".libs" && rm -f "pam_mysql.la" && ln -s "../pam_mysql.la" "pam_mysql.la" )
make[1]: Leaving directory `/root/rpmbuild/BUILD/pam_mysql-0.7RC1'
+ mv AUTHORS AUTHORS.lame
+ iconv -f latin1 -t utf-8 -o AUTHORS AUTHORS.lame
+ touch -r README AUTHORS
+ exit 0
执行(%install): /bin/sh -e /var/tmp/rpm-tmp.NXGtLO
+ umask 022
+ cd /root/rpmbuild/BUILD
+ '[' /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64 '!=' / ']'
+ rm -rf /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64
++ dirname /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64
+ mkdir -p /root/rpmbuild/BUILDROOT
+ mkdir /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64
+ cd pam_mysql-0.7RC1
+ rm -rf /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64
+ make install DESTDIR=/root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64
make[1]: Entering directory `/root/rpmbuild/BUILD/pam_mysql-0.7RC1'
 /usr/bin/mkdir -p '/root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/lib64/security'
 /bin/sh ./libtool   --mode=install /usr/bin/install -c   pam_mysql.la '/root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/lib64/security'
libtool: install: /usr/bin/install -c .libs/pam_mysql.so /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/lib64/security/pam_mysql.so
libtool: install: /usr/bin/install -c .libs/pam_mysql.lai /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/lib64/security/pam_mysql.la
libtool: install: warning: remember to run `libtool --finish /lib64/security'
make[1]: Nothing to be done for `install-data-am'.
make[1]: Leaving directory `/root/rpmbuild/BUILD/pam_mysql-0.7RC1'
+ find /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64 -name '*.la' -exec rm '{}' ';'
+ /usr/lib/rpm/find-debuginfo.sh --strict-build-id -m --run-dwz --dwz-low-mem-die-limit 10000000 --dwz-max-die-limit 110000000 /root/rpmbuild/BUILD/pam_mysql-0.7RC1
extracting debug info from /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/lib64/security/pam_mysql.so
dwz: Too few files for multifile optimization
/usr/lib/rpm/sepdebugcrcfix: Updated 1 CRC32s, 0 CRC32s did match.
192 blocks
+ /usr/lib/rpm/check-buildroot
+ /usr/lib/rpm/redhat/brp-compress
+ /usr/lib/rpm/redhat/brp-strip-static-archive /usr/bin/strip
+ /usr/lib/rpm/brp-python-bytecompile /usr/bin/python 1
+ /usr/lib/rpm/redhat/brp-python-hardlink
+ /usr/lib/rpm/redhat/brp-java-repack-jars
处理文件：pam_mysql-0.7-0.20.rc1.el7.x86_64
执行(%doc): /bin/sh -e /var/tmp/rpm-tmp.dKygeM
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd pam_mysql-0.7RC1
+ DOCDIR=/root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/usr/share/doc/pam_mysql-0.7
+ export DOCDIR
+ /usr/bin/mkdir -p /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/usr/share/doc/pam_mysql-0.7
+ cp -pr ChangeLog /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/usr/share/doc/pam_mysql-0.7
+ cp -pr COPYING /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/usr/share/doc/pam_mysql-0.7
+ cp -pr AUTHORS /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/usr/share/doc/pam_mysql-0.7
+ cp -pr NEWS /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/usr/share/doc/pam_mysql-0.7
+ cp -pr README /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64/usr/share/doc/pam_mysql-0.7
+ exit 0
Provides: pam_mysql = 1:0.7-0.20.rc1.el7 pam_mysql(x86-64) = 1:0.7-0.20.rc1.el7
Requires(rpmlib): rpmlib(CompressedFileNames) <= 3.0.4-1 rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1
Requires: libc.so.6()(64bit) libc.so.6(GLIBC_2.14)(64bit) libc.so.6(GLIBC_2.2.5)(64bit) libc.so.6(GLIBC_2.3.4)(64bit) libc.so.6(GLIBC_2.4)(64bit) libcrypt.so.1()(64bit) libcrypt.so.1(GLIBC_2.2.5)(64bit) libcrypto.so.10()(64bit) libcrypto.so.10(libcrypto.so.10)(64bit) libdl.so.2()(64bit) libm.so.6()(64bit) libmysqlclient.so.18()(64bit) libmysqlclient.so.18(libmysqlclient_18)(64bit) libpthread.so.0()(64bit) libpthread.so.0(GLIBC_2.2.5)(64bit) libssl.so.10()(64bit) libz.so.1()(64bit) rtld(GNU_HASH)
处理文件：pam_mysql-debuginfo-0.7-0.20.rc1.el7.x86_64
Provides: pam_mysql-debuginfo = 1:0.7-0.20.rc1.el7 pam_mysql-debuginfo(x86-64) = 1:0.7-0.20.rc1.el7
Requires(rpmlib): rpmlib(FileDigests) <= 4.6.0-1 rpmlib(PayloadFilesHavePrefix) <= 4.0-1 rpmlib(CompressedFileNames) <= 3.0.4-1
检查未打包文件：/usr/lib/rpm/check-files /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64
写道:/root/rpmbuild/RPMS/x86_64/pam_mysql-0.7-0.20.rc1.el7.x86_64.rpm
写道:/root/rpmbuild/RPMS/x86_64/pam_mysql-debuginfo-0.7-0.20.rc1.el7.x86_64.rpm
执行(%clean): /bin/sh -e /var/tmp/rpm-tmp.niS4CF
+ umask 022
+ cd /root/rpmbuild/BUILD
+ cd pam_mysql-0.7RC1
+ rm -rf /root/rpmbuild/BUILDROOT/pam_mysql-0.7-0.20.rc1.fc23.x86_64
+ exit 0
执行(--clean): /bin/sh -e /var/tmp/rpm-tmp.nNs3vD
+ umask 022
+ cd /root/rpmbuild/BUILD
+ rm -rf pam_mysql-0.7RC1
+ exit 0
[root@localhost ~]# yum localinstall -y rpmbuild/RPMS/x86_64/pam_mysql-0.7-0.20.rc1.el7.x86_64.rpm
已加载插件：fastestmirror
正在检查 rpmbuild/RPMS/x86_64/pam_mysql-0.7-0.20.rc1.el7.x86_64.rpm: 1:pam_mysql-0.7-0.20.rc1.el7.x86_64
rpmbuild/RPMS/x86_64/pam_mysql-0.7-0.20.rc1.el7.x86_64.rpm 将被安装
正在解决依赖关系
--> 正在检查事务
---> 软件包 pam_mysql.x86_64.1.0.7-0.20.rc1.el7 将被 安装
--> 解决依赖关系完成

依赖关系解决

================================================================================================================================================================================
 Package                           架构                           版本                                         源                                                          大小
================================================================================================================================================================================
正在安装:
 pam_mysql                         x86_64                         1:0.7-0.20.rc1.el7                           /pam_mysql-0.7-0.20.rc1.el7.x86_64                          92 k

事务概要
================================================================================================================================================================================
安装  1 软件包

总计：92 k
安装大小：92 k
Downloading packages:
Running transaction check
Running transaction test
Transaction test succeeded
Running transaction
  正在安装    : 1:pam_mysql-0.7-0.20.rc1.el7.x86_64                                                                                                                         1/1
  验证中      : 1:pam_mysql-0.7-0.20.rc1.el7.x86_64                                                                                                                         1/1

已安装:
  pam_mysql.x86_64 1:0.7-0.20.rc1.el7

完毕！
[root@localhost ~]#
```

4、 建立本地映射用户并设置宿主目录权限

```bash
[root@localhost ~]# useradd -s /sbin/nologin vsftpuser
[root@localhost ~]# chmod -R 755 /home/vsftpuser
[root@localhost ~]#
```

5、 配置 pam 认真文件

```bash
[root@localhost ~]# cat <<end> /etc/pam.d/vsftpd.db
> auth required pam_mysql.so user=vsftpuser passwd=vsftppasswd host=192.168.11.130 db=vsftpuser table=users usercolumn=name passwdcolumn=passwd crypt=2
> account required pam_mysql.so user=vsftpuser passwd=vsftppasswd host=192.168.11.130 db=vsftpuser table=users usercolumn=name passwdcolumn=passwd crypt=2
> end
[root@localhost ~]#
```

> crypt=0：表示口令使用明文方式保存在数据库中
>
> crypt=1：表示口令使用UNIX的DES加密方式加密后保存在数据库中
>
> crypt=2：表示口令使用MySQL的password()函数加密后保存在数据库中
>
> crypt=3：表示口令使用MD5散列值的方式保存在数据库中

6、 配置虚拟用户认证

```bash
[root@localhost ~]# cat <<end> /etc/vsftpd/vsftpd.conf
> anonymous_enable=NO
> local_enable=YES
> guest_enable=YES
> guest_username=vsftpuser
> allow_writeable_chroot=YES
> write_enable=YES
> local_umask=022
> dirmessage_enable=YES
> xferlog_enable=YES
> connect_from_port_20=YES
> xferlog_std_format=YES
> listen=NO
> listen_ipv6=YES
> pam_service_name=vsftpd.db
> userlist_enable=YES
> tcp_wrappers=YES
> end
[root@localhost ~]# systemctl restart vsftpd
[root@localhost ~]# firewall-cmd --add-service=ftp --permanent
success
[root@localhost ~]# firewall-cmd --reload
success
[root@localhost ~]# getenforce
Enforcing
[root@localhost ~]# setenforce 0
[root@localhost ~]#
```

7、 为虚拟用户设置不同的访问权限

```bash
[root@localhost ~]# mkdir -p /etc/vsftpd/vusers_config
[root@localhost ~]# cat <<end> /etc/vsftpd/vusers_config/userone
> anon_upload_enable=YES
> anon_mkdir_write_enable=YES
> anon_other_write_enable=YES
> end
[root@localhost ~]# echo "user_config_dir=/etc/vsftpd/vusers_config" >> /etc/vsftpd/vsftpd.conf
[root@localhost ~]# systemctl restart vsftpd
[root@localhost ~]#
```

8、 客户端测试

```bash
[root@client ~]# ftp 192.168.11.130
Connected to 192.168.11.130 (192.168.11.130).
220 (vsFTPd 3.0.2)
Name (192.168.11.130:root): userone
331 Please specify the password.
Password:
230 Login successful.
Remote system type is UNIX.
Using binary mode to transfer files.
ftp> ls
227 Entering Passive Mode (192,168,11,130,93,245).
150 Here comes the directory listing.
226 Directory send OK.
ftp> mkdir 123
257 "/123" created
ftp> ls
227 Entering Passive Mode (192,168,11,130,155,20).
150 Here comes the directory listing.
drwx------    2 1001     1001            6 Jun 30 20:45 123
226 Directory send OK.
ftp> exit
221 Goodbye.
[root@client ~]#
```
