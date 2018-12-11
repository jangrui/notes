# RHCSA模拟试题（Circle制作）

考试时间为2.5小时

请首先按找以下要求配置考试系统:

* Hostname: server0.example.com
* IP address: 172.25.0.11
* Netmask: 255.255.255.0
* Gateway: 172.25.254.254
* Name server: 172.25.254.254
* 所有配置要求系统重启后依然生效

## 第一题：修改root密码

修改root密码

请修改系统的root帐号密码为glssndwyz,确保能够使用root帐号能够登陆系统。

##第二题：设定SeLinux

设定SeLinux

请按下列要求设定系统：

- [ ] SeLinux的工作模式为enforcing
- [ ] 要求系统重启后依然生效

##第三题：设定YUM软件仓库

设定YUM软件仓库

配置你的本地默认YUM软件仓库，仓库地址为http://classroom.example.com/content/rhel7.0/x86_64/dvd

## 第四题：调整逻辑卷容量

调整逻辑卷容量

请按照以下要求调整本地逻辑卷lvm1的容量：

- [ ] 调整后的逻辑卷及文件系统大小为770MiB
- [ ] 调整后确保文件系统中已存在的内容不能被破坏
- [ ] 调整后的容量可能出现误差，只要在730MiB - 805MiB之间都是允许的
- [ ] 调整后，保证其挂载目录不改变，文件系统完成

## 第五题：创建用户和用户组

创建用户和用户组

请按照以下要求创建用户、用户组：

- [ ] 新建一个名为adminuser的组，组id为40000
- [ ] 新建一个名为natasha的用户，并将adminuser作为其附属组
- [ ] 新建一个名为harry的用户，并将adminuser作为其附属组
- [ ] 新建一个名为sarah的用户，其不属于adminuser组，并将其shell设置为不可登陆shell
- [ ] natasha、harry和sarah三个用户的密码均设置为glegunge

## 第六题：文件权限设定

文件权限设定

复制文件/etc/fstab到/var/tmp目录下，并按照以下要求配置/var/tmp/fstab文件的权限:

- [ ] 该文件的所属人为root
- [ ] 该文件的所属组为root
- [ ] 该文件对任何人均没有执行权限
- [ ] 用户natasha对该文件有读和写的权限
- [ ] 用户harry对该文件既不能读也不能写
- [ ] 所有其他用户（包括当前已有用户及未来创建的用户）对该文件都有读的权限

## 第七题：建立计划任务

建立计划任务

对natasha用户建立计划任务，要求在本地时间的每天14：23执行以下命令：/bin/echo "rhcsa"

## 第八题：文件特殊权限设定

文件特殊权限设定

在/home目录下创建名为admins的子目录，并按以下要求设置权限：

- [ ] /home/admins的所属组为adminuser
- [ ] 该目录对adminuser组的成员可读可执行可写，但对其他用户没有任何权限，但root不受限制
- [ ] 在/home/admins目录下所创建的文件的所属组自动被设置为adminuser

## 第九题：升级内核

升级系统内核

请按下列要求更新系统的内核：

新内核的RPM包位于http://content.example.com/rhel7.0/x86_64/errata/Packages/kernel-3.10.0-123.1.2.el7.x86_64.rpm
系统重启后，默认以新内核启动系统，原始的内核将继续可用

## 第十题：配置LDAP客户端

配置LDAP客户端

在classroom.example.com上已经部署了一台LDAP认证服务器，按以下要求将你的系统加入到该LDAP服务中，并使用Kerberos认证用户密码：

- [ ] 该LDAP认证服务的Base DN为：dc=example,dc=com
- [ ] 该LDAP认证服务的LDAP Server为：classroom.example.com
- [ ] 密码认证服务的Kerberos Realm为：EXAMPLE.COM
- [ ] 密码认证服务的Kerberos KDC为：classroom.example.com
- [ ] 密码认证服务的Kerberos Admin Server为：classroom.example.com
- [ ] 认证的绘画连接需要使用TLS加密，加密所用证书请在此下载http://classroom.example.com/pub/example-ca.crt

## 第十一题：LDAP用户家目录自动挂载

配置LDAP用户家目录自动挂载

请使用LDAP服务器上的用户ldapuser0登陆系统，并满足以下要求：

- [ ] ldapuser0用户的家目录路径为/home/guests/ldapuser0
- [ ] ldapuser0用户登陆后，家目录会自动挂载到classroom.example.com服务通过nfs服务到处的/home/guests/ldapuser0

## 第十二题：同步时间

同步时间

配置您的系统时间与服务器classroom.example.com同步，要求系统重启后依然生效

## 第十三题：打包文件

打包文件

请对/etc/sysconfig目录进行打包并用bzip2压缩，生成的文件保存为/root/sysconfig.tar.bz2

## 第十四题： 添加用户

创建用户

请创建一个名为alex的用户，并满足以下要求：

- [ ] 用户id为3456
- [ ] 密码为glegunge

## 第十五题：创建SWAP分区

创建swap分区

为系统新增加一个swap分区：

- [ ] 新建的swap分区容量为512MiB
- [ ] 重启系统后，新建的swap分区会自动激活
- [ ] 不能删除或者修改原有的swap分区

## 第十六题：查找文件

查找文件

请把系统上拥有者为ira用户的所有文件，并将其拷贝到/root/findfiles目录中

## 第十七题：过滤文件

过滤文件

把/usr/share/dict/words文件中所有包含seismic字符串的行找到，并将这些行按照原始文件中的顺序存放到/root/wordlist中，/root/wordlist文件不能包含空行

## 第十八题：创建逻辑卷

新建逻辑卷

请按下列要求创建一个新的逻辑卷:

- [ ] 创建一个名为exam的卷组，卷组的PE尺寸为16MiB
- [ ] 逻辑卷的名字为lvm2,所属卷组为exam,该逻辑卷由8个PE组成
- [ ] 将新建的逻辑卷格式化为xfs文件系统，要求系统启动时，该逻辑卷能被自动挂载到/exam/lvm2目录