# RHCE

## RHCE 模拟试题

```pdf
/rhce/RHCE-7-模拟试题.pdf
```

## 模拟试题环境

[百度云链接](https://pan.baidu.com/s/1M3Jm5JGdCmWFeMBz1dkSYg) | 密码 : i790

> - 添加设置 vmnet11 网段 , 地址为 172.25.0.0 , 子网掩码为 255.255.255.0 .
>
> - 解压后找到 classroom-rh124.vmx server-rh124.vmx desktop-rh124.vmx (共3个虚拟机), 双机可添加到虚拟机中 .
>
> - 若提示已移动或已复制 点击已复制 .
>
> - 设置每个虚拟机的网络为 vmnet11 .
> 
> - classroom-rh124 虚拟机带有 rhcsa 和 rhce 考场环境快照 , 可根据需要切换 .
> 
> - classroom-rh124 启动后无需任何操作 , 也不需要登录进入系统 .
>
> - server-rh124 恢复快照到初始化化状态 , 开启虚拟机后 , 用 lab examrhcsa setup 布置 rhcsa 考试环境 , lab examrhce setup 布置 rhce 考试环境 .
> 
> - desktop-rh124 用 lab examrhce setup 布置 rhce 考试环境 .  

## 解题

## 1. 修改 selinux

```bash
[root@server0 Desktop]# grep ^SELINUX= /etc/selinux/config
SELINUX=permissive
[root@server0 Desktop]# sed -i "/SELINUX/s/permissive/enforcing/g" /etc/selinux/config`
[root@server0 Desktop]# grep ^SELINUX= /etc/selinux/config
SELINUX=enforcing
[root@server0 Desktop]# setenforce 1
[root@server0 Desktop]# 
```

## 2. 配置防护墙对 ssh 的限制

```bash
firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source address=172.17.10.0/24 service name=ssh reject'
firewall-cmd --permanent --add-service=ssh
firewall-cmd --reload
firewall-cmd --list-all
```

## 3. 配置 IPV6 地址

```bash
nmcli connection modify eth0 ipv6.method manual ipv6.addresses fddb:fe2a:ab1e::c0a8:1/64 
nmcli connection up eth0
systemctl restart network
```

## 4. 配置链路聚合

```bash
nmcli connection add con-name team0 ifname team0 type team config '{"runner":{"name":"activebackup"}}'
nmcli connection add con-name team0-port1 ifname eth1 type team-slave master team0
nmcli connection add con-name team0-port2 ifname eth2 type team-slave master team0
nmcli connection modify team0 ipv4.addresses 192.168.0.101/24 ipv4.method manual
nmcli connection up team0
```

详解:

```bash
[root@server0 ~]# nmcli device status 
DEVICE  TYPE      STATE      ConnectionECTION 
eth0    ethernet  connected  eth0
eth1    ethernet  connected  --
eth2    ethernet  connected  --  
lo      loopback  unmanaged  --         
[root@server0 ~]# nmcli connection add con-name team0 ifname team0 type team config '{"runner":{"name":"activebackup"}}' 
Connection 'team0' (f5d2096c-c209-4b0c-b8e1-99e52c0a5a46) successfully added.
[root@server0 ~]# nmcli connection add con-name team0-port1 ifname eth1 type team-slave master team0
Connection 'team0-port1' (fe527b97-87cb-4f97-8488-fe1040033fd2) successfully added.
[root@server0 ~]# nmcli connection add con-name team0-port2 ifname eth2 type team-slave master team0
Connection 'team0-port2' (e2480622-ea17-4b67-a663-5f79fb3ee679) successfully added.
[root@server0 ~]# nmcli connection show
NAME         UUID                                  TYPE            DEVICE 
team0        f5d2096c-c209-4b0c-b8e1-99e52c0a5a46  team            team0
eth0         5fb06bd0-0bb0-7ffb-45f1-d6edd65f3e03  802-3-ethernet  eth0
team0-port1  fe527b97-87cb-4f97-8488-fe1040033fd2  802-3-ethernet  eth1
team0-port2  e2480622-ea17-4b67-a663-5f79fb3ee679  802-3-ethernet  eth2
[root@server0 ~]# nmcli connection modify team0 ipv4.addresses 192.168.0.101/24 ipv4.method manual
[root@server0 ~]# nmcli connection up team0
Connection successfully activated (D-Bus active path: /org/freedesktop/NetworkManager/ActiveConnection/3)
[root@server0 ~]# ip addr |grep team0 |grep inet
    inet 192.168.0.101/24 brd 192.168.0.255 scope global team0
[root@server0 ~]# 
```

## 5. 自定义用户环境

```bash
echo "alias qstat='/bin/ps -Ao pid,tt,user,fname,rsz'" >> /etc/bashrc
source /etc/bashrc
qstat
```

## 6. postfix

```bash
postconf -e "inet_interfaces = loopback-only"
postconf -e "local_transport = error:local"
postconf -e "mydestinaton = "
postconf -e "myorigin = example.com"
postconf -e "relayhost = classroom.example.com"
postconf -e "mynetworks = 127.0.0.0/8 [::1]/128"
systemctl restart postfix.service 
systemctl enable postfix.service
firewall-cmd --permanent --add-service=smtp
firewall-cmd --reload
```

## 7. 端口转发

```bash
firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source address=172.25.0.0/24 forward-port port=5423 protocol=tcp to-port=80'
firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source address=172.25.0.0/24 forward-port port=5423 protocol=udp to-port=80'
firewall-cmd --reload
```

## 8. samba

```bash
firewall-cmd --permanent --add-service=samba
firewall-cmd --permanent --add-service=samba-client
firewall-cmd --reload
yum install -y samba samba-*
systemctl restart smb nmb
systemctl enable smb nmb
mkdir /common
semanage fcontext -a -t 'samba_share_t' '/common(/.*)?'
restorecon -RvF /common/
chmod -Rf o+w /common
vim /etc/samba/smb.conf

workgroup = STAFF

[common]
	path = /common
	hosts allow = 172.25.0.
	browseable = yes
	writeable = no
	write list = brian

systemctl restart smb nmb 
useradd -s /sbin/nologin rob
useradd -s /sbin/nologin brian
echo redhat |smbpasswd -a rob
echo redhat |smbpasswd -a brian
```

## 9. samba client

```bash
firewall-cmd --add-service=samba --permanent
firewall-cmd --add-service=samba-client --permanent
firewall-cmd --reload
yum install -y cifs-utils samba samba-*

systemctl restart smb nmb
systemctl enable smb nmb

sambaclient -L server0.example.com -U brian

mkdir -p /mnt/multiuser

vim /root/multiuser.text
username=brian
password=redhat

vim /etc/fstab

//server0.example.com/common /mnt/multiuser cifs credentials=/root/multiuser.text,multiuser,sec=ntlmssp 0 0

mount -a 

df -Th

su - brian

cifscreds add server0.example.com
```

## 10. nfs

```bash
mkdir /public
mkdir /protected
wget -O /etc/krb5.keytab http://classroom.example.com/pub/keytabs/server0.keytab
mkdir /protected/project
chown -Rf ldapusre0 /protected/project

yum install -y nfs-*
# 指定版本
vim /etc/sysconfig/nfs
RPCNFSDARGS="-V 4.2"

vim /etc/exports
/protected	*.example.com(rw,sec=krb5p)
/public		*.example.com(ro)

systemctl restart nfs-server nfs-secure nfs-secure-server
systemctl enable nfs-server nfs-secure nfs-secure-server

firewall-cmd --permanent --add-service=nfs
firewall-cmd --permanent --add-service=mountd
firewall-cmd --permanent --add-service=rpc-bind
firewall-cmd --reload 
```

## 11. desktop nfs

```bash
mkdir /mnt/nfsmount
mkdir /mnt/nfssecure
wget -O /etc/krb5.keytab http://classroom.example.com/pub/keytabs/desktop0.keytab
yum install -y nfs-*
# 指定版本
vim /etc/sysconfig/nfs
RPCNFSDARGS="-V 4.2"

vim /etc/fstab
server0.example.com:/protected /mnt/nfssecure nfs defaults,sec=krb5p 0 0
server0.example.com:/publice /mnt/nfsmount nfs defaults,v4.2 0 0

systemctlc start nfs-secure nfs-secure-server nfs-server
systemctlc enable nfs-secure nfs-secure-server nfs-server
mount -a
df -Th

# 测试
ssh ldapuser0@localhost
passwd=kerberos
touch /protected/project/123
exit
```

## 12. web

```bash
firewall-cmd --permanent --add-service=http
firewall-cmd --add-rich-rule 'rule family=ipv4 source address=172.17.10.0/24 service name=http reject' --permanent 
firewall-cmd --reload
yum install -y httpd
systemctl start httpd
systemctl enable httpd

wget -O /var/www/html/index.html http://classroom.example.com/materials/station.html

vim /etc/httpd/conf.d/vhost-server0.conf
<VirtualHost *:80>
	DocumentRoot "/var/www/html"
	ServerName "server0.example.com"
	ErrorLog "/var/log/httpd/server0.errorlog"
	CustomLog "/var/log/httpd/server0.accesslog" combined
	<Directory "/var/www/html">
		AllowOverride none
		Options Indexes FollowSymLinks
		<RequireAll>
			require all granted
			require not host .my133t.org
		</RequireAll>
	<Directory/>
</VirtualHost>

httpd -t

systemctl restart httpd
```

## 13. web https

```bash
yum install -y mod_ssl
firewall-cmd --permanent --add-service=https
firewall-cmd --add-rich-rule 'rule family=ipv4 source address=172.17.10.0/24 service name=https reject' --permanent 
firewall-cmd --reload

wget -O /etc/pki/tls/certs/server0.crt http://classroom.example.com/pub/tls/certs/server0.crt
wget -O /etc/pki/tls/private/server0.key http://classroom.example.com/pub/tls/private/server0.key
wget -O /etc/pki/tls/certs/example-ca.crt http://classroom.example.com/pub/example-ca.crt

vim /etc/httpd/conf.d/ssl.conf
Listen 443 https
<VirtualHost *:443>
ErrorLog logs/ssl_error_log
TransferLog logs/ssl_access_log
LogLevel warn
SSLEngine on
SSLProtocol all -SSLv2
SSLCertificateFile /etc/pki/tls/certs/server0.crt
SSLCertificateKeyFile /etc/pki/tls/private/server0.key
DocumentRoot "/var/www/html"
ServerName "server0.example.com"
​	<Directory "/var/www/html">
​		Options Indexes FollowSymLinks
​		AllowOverride none
​		<RequireAll>
​			Require all granted
​			Require not host .my133t.org
​		</RequireAll>
​	</Directory>
</VirtualHost>
```

## 14. web

```bash
mkdir /var/www/virtual
wget -O /var/www/virtual/index.html http://classroom.example.com/materails/www.html
ll -d /var/www/virtual
id floyd
useradd -G root floyd
chmod g=rwx,g+s /var/www/virtual
vim /etc/httpd/conf.d/vhost-www0.conf
<VirtualHost *:80>
​	DocumentRoot "/var/www/virtual"
​	ServerName "www0.example.com"
​	<Directory "/var/www/virtual">
​		AllowOverride none
​		Options Indexes FollowSymlinks
​		Require all granted
​	</Directory>
</VirtualHost>

httpd -t
systemctl restart httpd
```

## 15. web

```bash
mkdir /var/www/virtual/private
wget -O /var/www/virtual/private/index.html http://classroom.example.com/materials/www.html

vim /etc/httpd/conf.d/vhost-www0.conf
<VirtualHost *:80>
​	DocumentRoot "/var/www/virtual"
​	ServerName "www0.example.com"
​	<Directory "/var/www/virtual">
​		AllowOverride none
​		Options Indexes FollowSymlinks
​		Require all granted
​	</Directory>
​	<Directory "/var/www/virtual/private">
​		AllowOverride none
​		Options Indexes FollowSymlinks
​		Require all denied
​		Require local
​	</Directory>
</VirtualHost>
```

## 16. web wsgi

```bash
yum install -y mod_wsgi
semanage port -a -t http_port_t -p tcp 8908
firewall-cmd --permanent --add-rich-rule rule family=ipv4 source address=172.25.0.0/24 port port=8908 protocol=tcp accept
firewall-cmd --reload
mkdir /var/www/webapp

wget -O /var/www/webapp/webinfo.wsgi http://classroom.example.com/materials/webinfo.wsgi

vim /etc/httpd/conf.d/vhost-webapp.conf
listen 8908
<VirtualHost *:8908>
wsgiscriptalias / /var/www/webapp/webinfo.wsgi
servername webapp0.example.com
</VirtualHost>

systemctl restart httpd
systemctl enable httpd
```

## 17. foo.sh

```bash
vim /root/foo.sh
#/bin/bash
case $1 in
redhat)
	echo fedora
fedora)
	echo redhat
*)
	echo "/root/foo.sh redhat|fedora"
	exit 1
esac
```

## 18. userfile

```bash
vim /root/userfile
#/bin/bash
if [ $# -eq 1 ];then
	if [ -f $1 ];then
	while read userlist
	do
	useradd -s /bin/false $userlist
	done < $1
	else
		echo "Input file not found"
		exit 2
	fi
else 
	echo "Usage: /root/batusers userfile"
	exit 1
fi
```

## 19. iscsi server0

```bash
yum install -y targetcli*
systemctl restart target
systemctl enable target
firewall-cmd --add-rich-rule 'rule family=ipv4 source address=172.25.0.10/24 port port=3260 protocol=tcp accept' --permanent
firewall-cmd --reload

fdisk -l

fdisk /dev/sdb

partprobe

pvcreate /dev/sdb1

vgcreate vg1 /dev/sdb1

lvcreate -n iscsi_store -L 3G vg1

targetcli

/backstore/block create disk1 /dev/vg1/iscsi_store
/iscsi create iqn.2014-11.com.example:server0
/iscsi create iqn.2014-11.com.example:server0/tpg1/acls create iqn.2014-11.com.example:desktop0
/iscsi create iqn.2014-11.com.example:server0/tpg1/luns create /backstores/block/disk1
/iscsi/iqn.2014-11.com.example:server0/tpg1/luns/portals create 172.25.0.11 ip_port=3260
saveconfig
exit
systemctl restart target
```

## 20. iscsi client

```bash
yum install -y iscsi-*

vim /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2014-11.com.example:desktop0

systemctl restart iscsi
systemctl enable iscsi

iscsiadm -m discovery -t st -p 172.25.0.11

iscsiadm -m node -T iqn.2014.com.example.com:server0 -l

fdisk -l

fdisk /devsdc

partprobe

mkfs.ext4 /dev/sdc1

mkdir /mnt/data

blkid /dev/sdc1 >> /etc/fstab

vim /etc/fstab
UUID=***************** /mnt/data ext4 defaults,_netdev 0 0

mount -a

df -Th
```

## 21. mariadb

```bash
yum install -y mariadb mariadb-*

systemctl restart mariadb
systemctl enable mariadb

firewall-cmd --add-service=mysql

mysql_secure-installation

wget http://content.example.com/courses/rhce/rhel7.0/materials/mariadb/mariadb.dump

mysql -uroot -proot_password

show database;

create datanase legacy;

use legacy;

show tables;

source /root/mariadb.dump;

show tables;

grant select on legacy.* to 'mary'@'localhost' identified by 'mary_password'
grant select,insert,update,delete on legacy.* to 'legacy'@'localhost' identified by 'legacy_password'
grant select on legacy.* to 'report'@'localhost' identified by 'report_password'

flush privileges

exit
```

## 22. mariadb selecs

```bash
mysql -uroot -p

use legacy;

show tables;

desc product;

select id from product where name='RT-AC68U' 

desc catagroy;

select id from category where name='servers'
```
