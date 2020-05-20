# RHCSA

## RHCSA 模拟试题

```pdf
/rhce/RHCSA-7-模拟试题.pdf
```

## 答案

## 0. 破解 root 密码

进入单用户模式修改密码

开机出现引导菜单时按 e 键,在 linux16行后添加 rd.break 参数,Ctrl+X 引导启动

```bash
mount -o remount,rw /sysroot/
chroot /sysroot

echo redhat | passwd --stdin root
touch /.autorelabel
sync
exit

reboot
```

重启后虚拟机可能默认命令行界面,如需要切换到图形界面, root 登录,执行命令:

```bash
# 暂时图形化界面,重启后还是命令行界面
systemctl isolate graphical.target
```

如果每次切换麻烦,可改变默认界面:

```bash
systemctl get-default

# 设置默认图形化界面启动
systemctl set-default graphical.target

# 设置默认命令行界面启动
systemctl set-default multi-user.target

# 安装图形化界面
yum -y groupinstall "Server with GUI"
```

修改网络设置

```bash
hostnamectl set-hostname server0.example.com
nmcli connection add autoconnect yes con-name eth0 ifname eth0 type ethernet ip4 172.25.0.11/24 gw4 172.25.254.254 ipv4.dns 172.25.254.254
nmcli connection up eth0
systemctl restart network
```

## 1. 修改 root 密码

```bash
echo root_password |passwd --stdin root
```

## 2. 设置 selinux

```bash
[root@server0 Desktop]# grep ^SELINUX= /etc/selinux/config 
SELINUX=permissive
[root@server0 Desktop]# sed -i "/^SELINUX/s/permissive/enforcing/g" /etc/selinux/config`
[root@server0 Desktop]# grep ^SELINUX= /etc/selinux/config
SELINUX=enforcing
[root@server0 Desktop]# setenforce 1
[root@server0 Desktop]# 
```

## 3. 配置 YUM 仓库

```bash
yum-config-manager --add-repo=http:/classroom.example.com/conntent/rhel.7.0/x86_64/dvd
echo 'gpgcheck=0' >> /etc/yum.repos.d/classroom.example.com_content_rhel7.0_x86_64_dvd.repo 
yum list
```

## 4. 修改逻辑卷大小

创建扩展分区:

```bash
[root@server0 Desktop]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type:
   p   primary (1 primary, 0 extended, 3 free)
   e   extended
Select (default p): e
Partition number (2-4, default 2): 
First sector (1050624-20971519, default 1050624): 
Using default value 1050624
Last sector, +sectors or +size{K,M,G} (1050624-20971519, default 20971519): 
Using default value 20971519
Partition 2 of type Extended and of size 9.5 GiB is set

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
```

创建逻辑分区:

```bash
[root@server0 Desktop]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type:
   p   primary (1 primary, 1 extended, 2 free)
   l   logical (numbered from 5)
Select (default p): l
Adding logical partition 5
First sector (1052672-20971519, default 1052672): 
Using default value 1052672
Last sector, +sectors or +size{K,M,G} (1052672-20971519, default 20971519): +770M
Partition 5 of type Linux and of size 770 MiB is set

Command (m for help): t
Partition number (1,2,5, default 5): 
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
[root@server0 Desktop]# partprobe 
```

调整逻辑分区:

```bash
[root@server0 Desktop]# pvcreate /dev/sdb5
  Physical volume "/dev/sdb5" successfully created
[root@server0 Desktop]# vgextend vg1 /dev/sdb5
  Volume group "vg1" successfully extended
[root@server0 Desktop]# lvextend -L 770M /dev/vg1/lvm1 
  Rounding size to boundary between physical extents: 772.00 MiB
  Extending logical volume lvm1 to 772.00 MiB
  Logical volume lvm1 successfully resized
[root@server0 Desktop]# xfs_growfs /dev/vg1/lvm1
meta-data=/dev/mapper/vg1-lvm1   isize=256    agcount=4, agsize=16384 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0
data     =                       bsize=4096   blocks=65536, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal               bsize=4096   blocks=853, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 65536 to 197632
[root@server0 Desktop]# df -Th
Filesystem           Type      Size  Used Avail Use% Mounted on
/dev/sda1            xfs        10G  3.1G  7.0G  31% /
devtmpfs             devtmpfs  899M     0  899M   0% /dev
tmpfs                tmpfs     914M  140K  914M   1% /dev/shm
tmpfs                tmpfs     914M   17M  897M   2% /run
tmpfs                tmpfs     914M     0  914M   0% /sys/fs/cgroup
/dev/mapper/vg1-lvm1 xfs       769M   14M  756M   2% /vg1/lvm1
[root@server0 Desktop]# 
```	

> xfs 格式同步文件系统: xfs_growfs /dev/vg1/lvm1
> ext3/4 格式同步文件系统: resize2fs /dev/vg1/lvm1

## 5. 创建用户和用户组

```bash
groupadd -g 40000 adminuser
useradd -G adminuser natasha
useradd -G adminuser harry
useradd -s /sbin/nologin sarah
echo glegunge | passwd --stdin natasha
echo glegunge | passwd --stdin harry
echo glegunge | passwd --stdin sarah
```

## 6. 文件权限设置

```bash
cp -a /etc/fstab /var/tmp/
chown root:root /var/tmp/fstab
chmod -x /var/tmp/fstab
setfacl -m u:natasha:rw /var/tmp/fstab
serfacl -m u:harry:- /var/tmp/fstab
ll /var/tmp/fstab
```

## 7. 创建计划任务

```bash
crontab -e -u natasha

23 14 * * * /bin/echo "rhcsa"

crontab -l -u natasha
```

## 8. 文件特殊权限设置

```bash
mkdir /home/admins
chgrp adminuser /home/admins
chmod g=rwx,o=-,g+s /home/admins
ll -d /home/admins

drwxrws---. 2 root adminuser 6 Dec  9 02:12 /home/admins
```

## 9. 升级内核

rpm 安装:

```bash
rpm -ivh http://content.example.com/rhel7.0/x86_64/errata/Packages/kernel-3.10.0-123.1.2.el7.x86_64.rpm
```

> 如果题中是 yum 源方式:

```bash
yum-config-manager --add-repo=http://content.example.com/rhel7.0/x86_64/errata/Packages/
echo "gpgcheck=0" >> /etc/yum.repos.d/content.example.com_rhel7.0_x86_64_errata_Packages.repo
yum update kernel -y
```

## 10. 配置 LDAP 客户端

```bash
yum install -y authconfig-gtk sssd openldap openldap-client
authconfig-gtk
```

复制粘贴相关配置.

## 11. autofs 自动挂载

```bash
yum install -y autofs
systemctl restart autofs
systemctl enabel autofs
mkdir /home/guests
echo "* -rw,fstype classroom.example.com:/home/guests/&" > /etc/auto.ldap
echo "/home/guests /etc/auto.ldap" >> /etc/auto.master
systemctl restart autofs
su - ldapuser0
exit
```

## 12. 同步时间

```bash
vim /etc/chrony.conf

server classroom.example.com iburst

chronyc sources -v
systemctl restart chronyd
systemctl enabel chronyd
```

## 13. 打包文件

```bash
tar cjvf /root/sysconfig.tar.bz2 /etc/sysconfig/
tar tvf /root/sysconfig.tar.bz2
```

## 14. 添加用户

```bash
useradd -u 3456 alex
echo glegunge | passwd --stdin alex
```

## 15. 创建 swap 分区

```bash
[root@server0 ~]# fdisk /dev/sdb 
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type:
   p   primary (1 primary, 1 extended, 2 free)
   l   logical (numbered from 5)
Select (default p): l
Adding logical partition 6
First sector (2631680-20971519, default 2631680): 
Using default value 2631680
Last sector, +sectors or +size{K,M,G} (2631680-20971519, default 20971519): +512M
Partition 6 of type Linux and of size 512 MiB is set

Command (m for help): t
Partition number (1,2,5,6, default 6): 
Hex code (type L to list all codes): 82
Changed type of partition 'Linux' to 'Linux swap / Solaris'

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
[root@server0 ~]# partprobe  
[root@server0 ~]# mkswap /dev/sdb6
Setting up swapspace version 1, size = 524284 KiB
no label, UUID=6227a60f-f31e-47ac-a9bd-842f39d50780
[root@server0 ~]# echo "UUID=6227a60f-f31e-47ac-a9bd-842f39d50780 swap swap defaults 0 0" >> /etc/fstab
[root@server0 ~]# swapon -a
[root@server0 ~]# free -h
             total       used       free     shared    buffers     cached
Mem:          1.8G       1.4G       383M        18M       1.6M       594M
-/+ buffers/cache:       847M       979M
Swap:         511M         0B       511M
[root@server0 ~]# 
```

## 16. 查找文件

```bash
mkdir /root/findfiles
find / -user ira -exec cp -arp {} /root/findfiles/ \;
ll /root/findfiles
```

## 17. 过滤文件

```bash
cat /usr/share/dict/words |grep seismic |grep -v '^$' > /root/wordlist
cat /root/wordlist
```


## 18. 新建逻辑分区

```bash
[root@server0 ~]# fdisk /dev/sdb
Welcome to fdisk (util-linux 2.23.2).

Changes will remain in memory only, until you decide to write them.
Be careful before using the write command.


Command (m for help): n
Partition type:
   p   primary (1 primary, 1 extended, 2 free)
   l   logical (numbered from 5)
Select (default p): l
Adding logical partition 7
First sector (3682304-20971519, default 3682304): 
Using default value 3682304
Last sector, +sectors or +size{K,M,G} (3682304-20971519, default 20971519): 
Using default value 20971519
Partition 7 of type Linux and of size 8.3 GiB is set

Command (m for help): t
Partition number (1,2,5-7, default 7): 
Hex code (type L to list all codes): 8e
Changed type of partition 'Linux' to 'Linux LVM'

Command (m for help): w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: Device or resource busy.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
Syncing disks.
[root@server0 ~]# partprobe 
[root@server0 ~]# pvcreate /dev/sdb7
  Physical volume "/dev/sdb7" successfully created
[root@server0 ~]# vgcreate -s 16M exam /dev/sdb7
  Volume group "exam" successfully created
[root@server0 ~]# lvcreate -n lvm2 -l 8 exam
  Logical volume "lvm2" created
[root@server0 ~]# mkfs.xfs /dev/exam/lvm2 
meta-data=/dev/exam/lvm2         isize=256    agcount=4, agsize=8192 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=0
data     =                       bsize=4096   blocks=32768, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=0
log      =internal log           bsize=4096   blocks=853, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
[root@server0 ~]# mkdir /exam/lvm2 -p 
[root@server0 ~]# echo "/dev/exam/lvm2 /exam/lvm2 xfs defaults 0 0" >> /etc/fstab 
[root@server0 ~]# mount -a
[root@server0 ~]# df -Th
Filesystem                                   Type      Size  Used Avail Use% Mounted on
/dev/sda1                                    xfs        10G  3.3G  6.8G  33% /
devtmpfs                                     devtmpfs  899M     0  899M   0% /dev
tmpfs                                        tmpfs     914M  140K  914M   1% /dev/shm
tmpfs                                        tmpfs     914M   17M  897M   2% /run
tmpfs                                        tmpfs     914M     0  914M   0% /sys/fs/cgroup
/dev/mapper/vg1-lvm1                         xfs       769M   14M  756M   2% /vg1/lvm1
classroom.example.com:/home/guests/ldapuser0 nfs4       10G  7.0G  3.1G  70% /home/guests/ldapuser0
/dev/mapper/exam-lvm2                        xfs       125M  6.6M  119M   6% /exam/lvm2
[root@server0 ~]# 
```
