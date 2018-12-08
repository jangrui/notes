# RHCE

## 1. 修改 selinux

```bash
[root@server0 Desktop]# grep SELINUX= /etc/selinux/config 
# SELINUX= can take one of these three values:
SELINUX=permissive
[root@server0 Desktop]# sed -i "s/SELINUX=permissive/SELINUX=enforcing/g" `grep "SELINUX=permissive" -rl /etc/selinux/config`
[root@server0 Desktop]# grep SELINUX= /etc/selinux/config # SELINUX= can take one of these three values:
SELINUX=enforcing
[root@server0 Desktop]# 
```

## 2. 配置防护墙对 ssh 的限制

```bash
firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source address=172.17.10.0/24 service name=ssh reject'
firewall-cmd --reload
firewall-cmd --list-all
```

## 3. 配置 IPV6 地址

```bash
nmcli connection modify eth0 ipv6.addresses fddb:fe2a:ab1e::c0a8:1/64 ipv6.method manual
nmcli connection up eth0
systemctl restart network
```

## 4. 配置链路聚合

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
echo 'alias qstat="/bin/ps -Ao pid,tt,user,fname,rsz"' >> /etc/bashrc
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
```

## 7. 端口转发

```bash
firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source address=172.25.0.0/24 forward-port port=5423 protocol=tcp to-port=80'
firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source address=172.25.0.0/24 forward-port port=5423 protocol=udp to-port=80'
firewall-cmd --reload
firewall-cmd --list-all
```

## 8. samba

```bash
firewall-cmd --permanent --add-service=samba
firewall-cmd --permanent --add-service=samba-client

yum install -y samba samba-client

systemctl restart smb nmb
systemctl enable smb nmb

mkdir /common

semanage fcontext -a -t 'samba_share_t' '/common(/.*)?'

chcon -R -t 'samba_share_t' /common

restorecon -RvF /common/

chmod -Rf o+w /common

vim /etc/samba/smb.conf
workgroup = STAFF

[common]
	comment = this is common !
	path = /common
	browseable = yes
	writeable = no
	write list = brian
	hosts allow = 172.25.0.

systemctl restart smb nmb 

useradd rob brian
echo redhat |smbpasswd -a rob
echo redhat |smbpasswd -a brian
```

## 9. samba client

```bash
yum install -y cifs-utils samba samba-client

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

vim /etc/exports
/protected	*.example.com(rw,sec=krb5p)
/public		*.example.com(ro)

wget -O /etc/krb5.keytab http://classroom.example.com/pub/keytabs/server0.keytab

yum install -y nfs-utils nfs-server nfs-secure nfs-secure-server

systemctl restart nfs-server nfs-secure nfs-secure-server
systemctl enable nfs-server nfs-secure nfs-secure-server

firewall-cmd --permanent --add-service=nfs
firewall-cmd --reload 

mkdir /protected/project
chown -Rf ldapusre0 /protected/project
ls -ad /protected/project
```

## 11. desktop nfs

```bash
yum install -y nfs-utils

systemctlc start nfs-secure nfs-secure-server nfs-server
systemctlc enable nfs-secure nfs-secure-server nfs-server

mkdir /mnt/nfsmount
mkdir /mnt/nfssecure

wget -O /etc/krb5.keytab http://classroom.example.com/pub/keytabs/desktop0.keytab

vim /etc/fstab
server0.example.com:/protected /mnt/nfssecure nfs defaults,sec=krb5p 0 0
server0.example.com:/publice /mnt/nfsmount nfs defaults 0 0

systemctlc start nfs-secure nfs-secure-server nfs-server

mount -a

df -Th

ssh ldapuser0@server0.example.com
passwd=kerberos
touch /protected/project/123
exit
ls /mnt/nfssecure/project
```

## 12. web

```bash
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

firewall-cmd --permanent --add-service=http
firewall-cmd --reload
```

## 13. web https

```bash
yum install -y mod_ssl

wget -O /etc/pki/tls/certs/server0.crt http://classroom.example.com/pub/tls/private/server0.crt
wget -O /etc/pki/tls/private/server0.key http://classroom.example.com/pub/tls/private/server0.key
wget -O /etc/pki/tls/example-ca.crt http://classroom.example.com/pub/example-ca.crt

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

firewall-cmd --permanent --add-service=https
firewall-cmd --reload
```

## 14. web

```bash
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

mkdir /var/www/virtual

wget -O /var/www/virtual/index.html http://classroom.example.com/materails/www.html

systemctl restart httpd

id floyd

useradd floyd

chown -Rf floyd /var/www/virtual
setfacl -m u:floyd:rwx /var/www/virtual
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

## 16. web

```bash
yum install -y mod_wsgi

mkdir /var/www/webapp

wget -O /var/www/webapp/webinfo.wsgi http://classroom.example.com/materials/webinfo.wsgi

vim /etc/httpd/conf.d/vhost-webapp.conf
listen 8908
<VirtualHost *:8908>
wsgiscriptalias / /var/www/webapp/webinfo.wsgi
servername webapp0.example.com
</VirtualHost>

semanage port -a -t http_port_t -p tcp 8908

firewall-cmd --permanent --add-rich-rule rule family=ipv4 source address=172.25.0.0/24 port port=8908 protocol=tcp accept

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
yum install -y targetcli

systemctl restart target
systemctl enable target

fdisk -l

fdisk /dev/sdb

partprobe

pvcreate /dev/sdb1

vgcreate vg1 /dev/sdb1

lvcreate -n iscsi_store -L 3G vg1

targetcli
ls
cd /backstore/block

create disk1 /dev/vg1/iscsi_store

/iscsi create iqn.2014-11.com.example:server0
/iscsi create iqn.2014-11.com.example:server0/tpg1/
/iscsi create iqn.2014-11.com.example:server0/tpg1/acls create iqn.2014-11.com.example:desktop0
/iscsi create iqn.2014-11.com.example:server0/tpg1/luns create /backstores/block/disk1
/iscsi/iqn.2014-11.com.example:server0/tpg1/luns/portals create 172.25.0.11 ip_port=3260
saveconfig
exit
```

## 20. iscsi client

```bash
yum install -y iscsi-initiator-utils.i686

systemctl restart iscsi
systemctl enable iscsi

vim /etc/iscsi/initiatorname.iscsi
InitiatorName=iqn.2014-11.com.example:desktop0

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
server0
yum install -y mariadb mariadb-server
desktop0:
yum install -y mariadb

systemctl restart mariadb
systemctl enable mariadb

mysql_secure-installation

password Y
anonymous users Y
root login remotely Y
test database and access to it Y
privilege tables now Y

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

select * from product where id_category=2;
```