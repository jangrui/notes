<!--
 * @Author: jangrui
 * @Date: 2019-07-31 07:45:23
 * @LastEditors: jangrui
 * @LastEditTime: 2019-08-22 20:00:19
 * @version: 
 * @Descripttion: NFS
 -->

# NFS

NFS（Network File System）即网络文件系统，允许网络中的计算机之间共享资源。可以透明地读写远端 NFS 服务器上的文件，就像访问本地文件一样。

## 安装

> 考虑范围： 防火墙、共享点、权限

```bash
yum install -y nfs-utils

firewall-cmd --permanent --add-service=nfs --add-service=nfs3 --add-service=rpc-bind
firewall-cmd --reload

mkdir -p /home/nfsfiles
chmod 777 /home/nfsfiles
echo "this is nfs share home" >> /home/nfsfiles/hello.txt
```

## 配置

NFS服务程序的配置文件为/etc/exports，默认情况下里面没有任何内容。

配置NFS服务程序配置文件的参数：

|参数|作用|
|-|-|
|`ro`|只读|
|`rw`|读写|
|`root_squash`|当NFS客户端以root管理员访问时，映射为NFS服务器的匿名用户|
|`no_root_squash`|当NFS客户端以root管理员访问时，映射为NFS服务器的root管理员|
|`all_squash`|无论NFS客户端使用什么账户访问，均映射为NFS服务器的匿名用户|
|`sync`|同时将数据写入到内存与硬盘中，保证不丢失数据|
|`async`|优先将数据保存到内存，然后再写入硬盘；这样效率更高，但可能会丢失数据|

格式： `共享目录` + `允许 ip 范围` + `(参数)`

```bash
cat <<EOF> /etc/exports
/home/nfsfiles 192.168.11.*(rw,sync,root_squash)
EOF

systemctl restart nfs-server
systemctl enable nfs-server
```

> 考虑范围： 开机启动

## 挂载

在 NFS 客户端创建挂载目录并挂载。

```bash
mkdir /home/mnt

yum install -y nfs-utils

cat <<EOF>> /etc/fstab
192.168.11.130:/home/nfsfiles /home/mnt nfs defaults 0 0
EOF

mount -a
cat /home/mnt/hello.txt
```
