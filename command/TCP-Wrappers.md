# TCP Wrappers

TCP Wrappers是RHEL 7系统中默认启用的一款流量监控程序，它能够根据来访主机的地址与本机的目标服务程序作出允许或拒绝的操作。换句话说，Linux系统中其实有两个层面的防火墙，第一种是前面讲到的基于TCP/IP协议的流量过滤工具，而TCP Wrappers服务则是能允许或禁止Linux系统提供服务的防火墙，从而在更高层面保护了Linux系统的安全运行。

TCP Wrappers服务的防火墙策略由两个控制列表文件所控制，用户可以编辑允许控制列表文件来放行对服务的请求流量，也可以编辑拒绝控制列表文件来阻止对服务的请求流量。控制列表文件修改后会立即生效，系统将会先检查允许控制列表文件（/etc/hosts.allow），如果匹配到相应的允许策略则放行流量；如果没有匹配，则去进一步匹配拒绝控制列表文件（/etc/hosts.deny），若找到匹配项则拒绝该流量。如果这两个文件全都没有匹配到，则默认放行流量。

TCP Wrappers服务的控制列表文件配置起来并不复杂，常用的参数如下表所示。

|客户端类型      |示例                         |满足示例的客户端列表|
|-|-|-|
|单一主机        |192.168.10.128              |IP地址为192.168.10.10的主机
|指定网段        |192.168.10.                 |IP段为192.168.10.0/24的主机
|指定网段        |192.168.10.0/255.255.255.0  |IP段为192.168.10.0/24的主机
|指定DNS后缀     |.jangrui.com                |所有DNS后缀为.jangrui.com的主机
|指定主机名称     |www.jangrui.com             |主机名称为www.jagnrui.com的主机
|指定所有客户端   |ALL                         |所有主机全部包括在内

在配置TCP Wrappers服务时需要遵循两个原则：

- 编写拒绝策略规则时，填写的是服务名称，而非协议名称；

- 建议先编写拒绝策略规则，再编写允许策略规则，以便直观地看到相应的效果。

例如禁止本机 sshd 服务的所有流量:

```bash
[root@localhost ~]# echo 'sshd:*' >> /etc/hosts.deny
[root@localhost ~]# ssh 192.168.10.128
ssh_exchange_identification: read: Connection reset by peer
```

如果不知道服务名称可用相关命令模糊搜索:

```bash
[root@localhost ~]# systemctl list-unit-files |grep "ssh"
anaconda-sshd.service                       static  
sshd-keygen.service                         static  
sshd.service                                enabled
sshd@.service                               static  
sshd.socket                                 disabled
```

其中 sshd.service 就是 ssh 的服务程序,sshd 就是 ssh 的服务名称.

接下来,再允许策略规则文件中加入一条规则,使其放行源自192.168.10.0/24网段，访问本机sshd服务的所有流量。可以看到，服务器立刻就放行了访问sshd服务的流量，效果非常直观：

```bash
[root@localhost ~]# echo 'sshd:192.168.10.' >> /etc/hosts.allow
[root@localhost ~]# ssh 192.168.10.128
The authenticity of host '192.168.10.128 (192.168.10.128)' can't be established.
ECDSA key fingerprint is 47:b8:a9:92:67:d3:87:64:12:9f:ea:9c:e7:55:07:82.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added '192.168.10.128' (ECDSA) to the list of known hosts.
root@192.168.10.128's password:
Last login: Wed Nov  7 09:35:27 2018 from 192.168.10.1
```
