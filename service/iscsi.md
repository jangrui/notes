<!--
 * @Author: jangrui
 * @Date: 2019-07-31 07:48:24
 * @LastEditors: jangrui
 * @LastEditTime: 2019-08-26 04:45:48
 * @version: 
 * @Descripttion: 
 -->

# ISCSI

iSCSI 是一种将 SCSI 接口与以太网技术相结合的新型存储技术，可以用来在网络中传输 SCSI 接口的命令和数据。不仅克服了传统 SCSI 接口设备的物理局限性，实现了跨区域的存储资源共享，还可以在不停机的状态下扩展存储容量。

> IDE是一种成熟稳定、价格便宜的并行传输接口。
>
> SATA是一种传输速度更快、数据校验更完整的串行传输接口。
>
> SCSI是一种用于计算机和硬盘、光驱等设备之间系统级接口的通用标准，具有系统资源占用率低、转速高、传输速度快等优点。

iSCSI 技术在工作形式上分为服务端（target）与客户端（initiator）。

- `iSCSI 服务端`: 即用于存放硬盘存储资源的服务器，能够为用户提供可用的存储资源。

- `iSCSI 客户端`: 则是用户使用的软件，用于访问远程服务端的存储资源。

> 虚拟机模拟配置 iSCSI 服务端 IP (192.168.10.10)、客户端 IP (192.168.10.20)。

## iSCSI 服务端配置

1. 磁盘准备

添加四块硬盘结合 RAID 阵列作为 iSCSI 资源池。

```bash
ll /dev/sd*
brw-rw----. 1 root disk 8,  0 8月  25 22:23 /dev/sda
brw-rw----. 1 root disk 8,  1 8月  25 22:23 /dev/sda1
brw-rw----. 1 root disk 8,  2 8月  25 22:23 /dev/sda2
brw-rw----. 1 root disk 8, 16 8月  25 22:23 /dev/sdb
brw-rw----. 1 root disk 8, 32 8月  25 22:23 /dev/sdc
brw-rw----. 1 root disk 8, 48 8月  25 22:23 /dev/sdd
brw-rw----. 1 root disk 8, 64 8月  25 22:23 /dev/sde

yum install -y mdadm
mdadm -Cv /dev/md0 -n 3 -l 5 -x 1 /dev/sd[b-e]
mdadm -D /dev/md0
```

2. 安装 iSCSI 服务端及配置工具

```bash
yum install -y targetd targetcli
systemctl enable targetd
systemctl start targetd
```

3. 配置共享资源池

加入 `/dev/md0` 到 iSCSI 资源池。

```bash
targetcli ls
targetcli /backstores/block create disk0 /dev/md5
targetcli ls
```

4. 创建共享资源

```bash
targetcli /iscsi ls
iscsiname=`targetcli /iscsi create|grep "iqn"|awk '{print $3}'`
iscsiname=${iscsiname%.*}
echo $iscsiname > ~/iscsi.name
targetcli /iscsi/${}/tpg1/luns create /backstores/block/disk0
```

> ${iscsiname%.*}: 删除最后一个`.`及其右边的字符串。

5. 设置访问控制列表（ACL)

iSCSI协议是通过客户端名称进行验证的，也就是说，在访问存储共享资源时不需要输入密码，只要iSCSI客户端的名称与服务端中设置的访问控制列表中某一名称条目一致即可，因此需要在iSCSI服务端的配置文件中写入一串能够验证用户信息的名称。

`acls` 目录用于存放能够访问iSCSI服务端共享存储资源的客户端名称。

```bash
targetcli /iscsi/${iscsiname}/tpg1/acls create ${iscsiname}:client
targetcli /iscsi ls
```

6. 设置监听 ip 地址和端口号

```bash
targetcli /iscsi/${iscsiname}/tpg1/portals create 192.168.10.10

systemctl restart targetd
firewall-cmd --permanent --add-service=iscsi-target
firewall-cmd --reload
```

## iSCSI 客户端配置

```bash
yum install -y iscsi-initiator-utils

iscsiname=`ssh root@192.168.10.10 echo iscsi.name`
sed -i "s,InitiatorName=.*,InitiatorName=$iscsiname:client,g" /etc/iscsi/initiatorname.iscsi

systemctl start iscsid
systemctl enable iscsid

iscsiadm -m discovery -t st -p 192.168.10.10

fdisk -l

iscsiadm -m node -T $iscsiname -p 192.168.10.10 --login

fdisk -l
ll /dev/sd*
```

> 参数 `--login`: 连接共享资源，`-u`: 退出连接共享资源

### 挂载

```bash
mkfs.xfs /dev/sdb
mkdir /iscsi

uuid=`blkid|grep /dev/sdb|awk '{print $2}'|sed 's,\",,g'`
echo "$uuid /iscsi xfs defaults,_netdev 0 0" >> /etc/fstab
mount -a

df -Th
```
