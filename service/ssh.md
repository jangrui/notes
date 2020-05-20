# SSH

SSH (Secure Shell) 是基于 SSH 协议开发的一款远程管理服务程序 能够以安全的方式提供远程登录的协议，也是目前远程管理Linux系统的首选方式。

sshd 提供两种安全验证的方法：

- 基于口令的验证:
  - 用账户和密码来验证登录；
- 基于密钥的验证:
  - 本地私钥与服务器中的公钥进行比较；该方式相较来说更安全。

## 配置

几乎所有的 linux 发行版都默认安装了 sshd 服务。

sshd 服务默认配置文件：

```bash
cat /etc/ssh/sshd_config |grep -Ev '^$|#'
HostKey /etc/ssh/ssh_host_rsa_key
HostKey /etc/ssh/ssh_host_ecdsa_key
HostKey /etc/ssh/ssh_host_ed25519_key
SyslogFacility AUTHPRIV
AuthorizedKeysFile    .ssh/authorized_keys
PasswordAuthentication yes
ChallengeResponseAuthentication no
GSSAPIAuthentication yes
GSSAPICleanupCredentials no
UsePAM yes
X11Forwarding yes
AcceptEnv LANG LC_CTYPE LC_NUMERIC LC_TIME LC_COLLATE LC_MONETARY LC_MESSAGES
AcceptEnv LC_PAPER LC_NAME LC_ADDRESS LC_TELEPHONE LC_MEASUREMENT
AcceptEnv LC_IDENTIFICATION LC_ALL LANGUAGE
AcceptEnv XMODIFIERS
Subsystem    sftp    /usr/libexec/openssh/sftp-server
```

sshd 服务配置文件常用参数以及作用:

|参数|作用|
|-|-|
|`Port 22`                              |默认的sshd服务端口|
|`ListenAddress 0.0.0.0`                |设定sshd服务器监听的IP地址|
|`Protocol 2`                           | SSH协议的版本号|
|`HostKey /tc/ssh/ssh_host_key`         |SSH协议版本为1时，DES私钥存放的位置|
|`HostKey /etc/ssh/ssh_host_rsa_key`    |SSH协议版本为2时，RSA私钥存放的位置|
|`HostKey /etc/ssh/ssh_host_dsa_key`    |SSH协议版本为2时，DSA私钥存放的位置|
|`PermitRootLogin yes`                  |设定是否允许root管理员直接登录|
|`StrictModes yes`                      |当远程用户的私钥改变时直接拒绝连接|
|`MaxAuthTries 6`                       |最大密码尝试次数|
|`MaxSessions 10`                       |最大终端数|
|`PasswordAuthentication yes`           |是否允许密码验证|
|`PermitEmptyPasswords no`              |是否允许空密码登录（很不安全）|
|`ClientAliveInterval 420`              |服务器端向客户端请求消息间隔时间,默认是0,不发送|
|`ClientAliveCountMax 3`                |服务器发出请求后客户端没有响应的次数达到一定值,自动断开|
|`UseDNS`                               |进行DNS正向A记录查询,防止客户端欺骗,关闭可加快连接速度|

本地生成秘钥并上传至服务器。

```bash
ssh-keygen # 可一路回车
ssh-copy-id 192.168.11.130
```

## 必要的安全措施

> 考虑范围: 更改服务端口 ==> 拒绝口令认证 ==> 拒绝以 root 直接登录

```bash
sed -i 's,#Port 22,Port 2222,g' /etc/ssh/sshd_config
sed -i 's,#PermitRootLogin yes,PermitRootLogin no,g' /etc/ssh/sshd_config
sed -i 's,#UseDNS yes,UseDNS no,g' /etc/ssh/sshd_config
sed -i 's,PasswordAuthentication yes,PasswordAuthentication no,g' /etc/ssh/sshd_config

systemctl restart sshd
```

> 建议： 修改 sshd 配置时多开一个 ssh 连接窗口，以防自己被拒之门外。
