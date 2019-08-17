# AutoFs 自动挂载服务

Autofs 与 Mount/Umount 的不同之处在于，它是一种看守程序。

如果检测到用户正试图访问一个尚未挂接的文件系统，它就会自动检测该文件系统，如果存在，那么Autofs会自动将其挂载。

如果它检测到某个已挂接的文件系统在一段时间内没有被使用，那么Autofs会自动将其卸载。

因此一旦运行了Autofs后，用户就不再需要手动完成文件系统的挂载和卸载。

## 安装

```bash
yum install -y autofs
systemctl start autofs
systemctl enable autofs
```

## 配置

默认配置文件如下：

```bash
cat /etc/auto.master |grep -Ev '^$|#'
/misc    /etc/auto.misc
/net    -hosts
+dir:/etc/auto.master.d
+auto.master
```

在autofs服务程序的主配置文件中需要按照 `挂载目录 子配置文件` 的格式进行填写。挂载目录是设备挂载位置的上一级目录。

子配置文件格式：`挂载点 -fstype=文件类型,权限 挂载目标`。

- 挂载 iso

例如，光盘设备一般挂载到 /media/cdrom 目录中，那么挂载目录写成 /media 即可。对应的子配置文件则是对这个挂载目录内的挂载设备信息作进一步的说明。子配置文件需要用户自行定义，文件名字没有严格要求，但后缀建议以.misc结束。

```bash
cat <<EOF>> /etc/auto.master
/media /etc/auto.master.d/iso.misc
EOF

cat <<EOF>> /etc/auto.master.d/iso.misc
iso -fstype=iso9660,ro,nosuid,nodev :/dev/cdrom
EOF

systemctl restart autofs

df -h
cd /media/iso
df -h
```

- 挂载 nfs

```bash
cat <<EOF>> /etc/auto.master
/home /etc/nfs.mics
EOF

cat <<EOF>> /etc/nfs.misc
nfs -fstype=nfs 192.168.11.130:/home/nfsfiles
EOF

systemctl restart autofs
```

- 挂载 cifs

```bash
cat <<EOF>> /etc/auto.master
/home /etc/cifs.mics
EOF

cat <<EOF>> /etc/samba.misc
samba -fstype=cifs,credentials=/home/auth.smb ://192.168.11.130/share
EOF

systemctl restart autofs
```
