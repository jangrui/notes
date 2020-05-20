# firewall-cmd

## 全部参数

|参数    |作用|
|-|-|
|--add-forward-port=                        |指定端口接收到的请求转发到另一指定端口
|--add-icmp-block=                          |按icmp类型 设置阻塞
|--add-interface=<网卡名称>                  |将源自该网卡的所有流量都导向某个指定区域
|--add-lockdown-whitelist-command=          |添加一个具体的command
|--add-lockdown-whitelist-context=          |添加一个具体的selinux contenxt
|--add-lockdown-whitelist-uid=              |添加一个具体指定的用户开放配置权限
|--add-lockdown-whitelist-user=             |添加一个具体指定的用户开放配置权限
|--add-masquerade                           |设置 ip地址伪装
|--add-port=<端口号/协议>                     |设置默认区域允许该端口的流量
|--add-rich-rule                            |设置富规则(自定义规则)
|--add-service=<服务名>                      |设置默认区域允许该服务的流量
|--add-source=                              |将源自此IP或子网的流量导向指定的区域
|--change-interface=<网卡名称>               |将某个网卡与区域进行关联
|--change-source=                           |改变source地址所绑定的zone，如果原来没有绑定则进行绑定，这样就跟--add-source的作用一样了
|--change-zone=                             |改变临时区域
|--complete-reload                          |完全刷新永久配置（终端修改后不符合的操作将会中断）
|--direct                                   |firewall直接接口
|--get-active-zones                         |显示当前正在使用的区域与网卡名称
|--get-default-zone                         |查询默认的区域名称
|--get-icmptypes                            |查询 icmp 类型
|--get-services                             |显示预先定义的服务
|--get-zone-of-interface=网卡名              |查询网卡所在区域|
|--get-zones                                |显示可用的区域
|--help                                     |帮助
|--list-all                                 |显示当前区域的网卡配置参数、资源、端口以及服务等信息
|--list-all-zones                           |显示所有区域的网卡配置参数、资源、端口以及服务等信息
|--list-forward-ports                       |列出当前区域的端口转发信息
|--list-icmp-blocks                         |列出当前区域 icmp 阻塞信息
|--list-interfaces                          |列出网卡接口
|--list-lockdown-whitelist-commands         |列出所有白名单中配置了的command
|--list-lockdown-whitelist-contexts         |列出所有白名单中配置了的selinux contenxt
|--list-lockdown-whitelist-uids             |列出一个具体指定的用户开放配置权限
|--list-lockdown-whitelist-users            |列出一个具体指定的用户开放配置权限
|--list-ports                               |列出默认区域端口配置信息
|--list-rich-rules                          |显示当前区域的富规则配置情况
|--list-services                            |显示当前区域可用的服务
|--list-sources                             |列出指定区域的所有绑定的source地址
|--lockdown-on                              |开启白名单
|--lockdown-off                             |关闭白名单
|--panic-off                                |关闭应急状况模式
|--panic-on                                 |开启应急状况模式
|--permanent                                |全局设置(重启后生效)
|--query-forward-port=                      |查询指定端口接收到的请求转发到另一指定端口
|--query-icmp-block=                        |查询指定 icmp 报文阻塞配置规则
|--query-interface=                         |查询网卡规则
|--query-lockdown                           |查询白名单开启状态
|--query-lockdown-whitelist-command=        |查询一个具体的command
|--query-lockdown-whitelist-context=        |查询一个具体的selinux contenxt
|--query-lockdown-whitelist-uid=            |查询一个具体指定的用户开放配置权限
|--query-lockdown-whitelist-user=           |查询一个具体指定的用户开放配置权限
|--query-masquerade                         |查询 ip 伪装状态
|--query-panic                              |查询应急模式是状态
|--query-port=<端口号/协议>                   |查看默认区域该端口配置规则情况
|--query-rich-rule                          |查看默认区域指定富规则配置情况
|--query-service=                           |查询默认区域该服务的配置规则情况
|--query-source=                            |查询指定zone是否跟指定source地址进行了绑定
|--reload                                   |让“永久生效”的配置规则立即生效，并覆盖当前的配置规则
|--remove-forward-port=                     |设置默认区域不再转发该端口的流量
|--remove-icmp-block=                       |取消 icmp 类型阻塞
|--remove-interface=                        |删除默认网卡配置规则
|--remove-lockdown-whitelist-command=       |删除一个具体的command
|--remove-lockdown-whitelist-context=       |删除一个具体的selinux contenxt
|--remove-lockdown-whitelist-uid=           |删除一个具体的指定的用户开放配置权限
|--remove-lockdown-whitelist-user=          |删除一个具体的指定的用户开放配置权限
|--remove-masquerade                        |删除 ip 伪装
|--remove-port=<端口号/协议>                  |设置默认区域不再允许该端口的流量
|--remove-rich-rule=                        |删除默认区域富规则配置规则
|--remove-service=<服务名>                   |设置默认区域不再允许该服务的流量
|--remove-source=                           |不再将源自此IP或子网的流量导向某个指定区域
|--set-default-zone=<区域名称>               |设置默认的区域，使其永久生效
|--state                                    |查看 firewall 状态
|--version                                  |查看 firewall 版本
|--zone=                                    |指定临时区域

## 策略规则

firewalld中常见的区域名称（默认为public）以及相应的策略规则:

|区域|默认规则策略|
|-|-|
|trusted  | 允许所有的数据包
|home     | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、mdns、ipp-client、amba-client与dhcpv6-client服务相关，则允许流量
|internal | 等同于home区域
|work     | 拒绝流入的流量，除非与流出的流量数相关；而如果流量与ssh、ipp-client与dhcpv6-client服务相关，则允许流量
|public   | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh、dhcpv6-client服务相关，则允许流量
|external | 拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量
|dmz      |拒绝流入的流量，除非与流出的流量相关；而如果流量与ssh服务相关，则允许流量
|block    | 拒绝流入的流量，除非与流出的流量相关
|drop     | 拒绝流入的流量，除非与流出的流量相关

## 常用模块

|模块|功能|查询|添加|删除|
|-|-|-|-|-|
|-forward-port= |端口转发      |--list-forward-port=     |--add-forward-port=   |--remove-forward-port=     |
|-icmp-block=   |icmp报文阻塞  |--list-icmp-block=       |--add-icmp-block=     |--remove-icmp-block=       |
|-interface=    |网卡接口      |--list-interface=        |--add-interface=      |--remove-interface=        |
|-masquerade    |ip地址伪装    |--list-masquerade        |--add-masquerade      |--remove-masquerade        |
|-port=         |端口         |--list-port=             |--add-port=           |--remove-port=             |
|-rich-rule     |自定义规则    |--list-rich-rule         |--add-rich-rule       |--remove-rich-rule         |
|-service=      |服务         |--list-service=          |--add-service=        |--remove-service=          |
|-source=       |源地址       |--list-source=           |--add-source=         |--remove-source=           |

## 模块详解

### forward-port

端口转发，比如要将在80端口接收到tcp请求转发到8080端口

```bash
firewall-cmd --add-forward-port=port=80:proto=tcp:toport=8080
```

forward-port还支持范围转发，比如将80到85端口的所有请求都转发到8080端口，这时只需要将上面命令中的port修改为80-85即可。

在zone配置文件中节点如下:

```xml
<zone>
    <forward-port port="portid[-portid]" protocol="tcp|udp" [to-port="portid[-portid]"] [to-addr="ipv4address"]/>
</zone>
```

- 相关操作命令

```bash
firewall-cmd [--permanent] [--zone=zone] --list-forward-ports
firewall-cmd [--permanent] [--zone=zone] --add-forward-port=port=portid[-portid]:proto=protocol[:toport=portid[-portid]][:toaddr=address[/mask]][--timeout=seconds]
firewall-cmd [--permanent] [--zone=zone] --remove-forward-port=port=portid[-portid]:proto=protocol[:toport=portid[-portid]][:toaddr=address[/mask]]
firewall-cmd [--permanent] [--zone=zone] --query-forward-port=port=portid[-portid]:proto=protocol[:toport=portid[-portid]][:toaddr=address[/mask]]
```

### icmp-block

icmp-block是按照icmp的类型进行设置阻塞，比如不想接受ping报文就可以使用下面的命令来设置

```bash
firewall-cmd --add-icmp-block=echo-request
```

当然，如果需要长久保存就需要加--permanent选项，不过那样就需要reload才能生效。

icmp-block在zone配置文件中的节点为:

```xml
<zone>
    <icmp-block name="string"/>
</zone>
```

- 相应操作命令为

```bash
firewall-cmd [--permanent] [--zone=zone] --list-icmp-blocks
firewall-cmd [--permanent] [--zone=zone] --add-icmp-block=icmptype [--timeout=seconds]
firewall-cmd [--permanent] [--zone=zone] --remove-icmp-block=icmptype
firewall-cmd [--permanent] [--zone=zone] --query-icmp-block=icmptype
```

#### icmptype 类型

`destination-unreachable`：当收到这种类型数据包之后相应地址的连接将会被断开，如果是攻击者伪造的数据包，那么会将很多正常连接断开。当将其设置到zone中后,本机发送的请求还是可以收到destination-unreachable类型回复的，只是直接发给destination-unreachable数据包进不来了，所以建议大家可以阻止。

`echo-request`：主要用于接收ping请求，阻塞之后,主机将不可被ping，不过打开后又有可能被攻击，有种惯用的做法是设置开通的频率，比如1秒钟只可以被ping一次，不过这种功能直接使用上面给的方法是无法设置的，可以通过firewalld中的direct配置。

`echo-reply`：这是回应ping信息的包，一般来说应该将其阻止，因为他跟destination-unreachable一样，如果是本机发出的即使设置了阻止也还是可以接收到的。

`parameter-problem`：当接收到的报文参数错误，无法解析时会返回这种类型的报文。

`redirect`：路由器发现主机针对某个目的ip有一个更好的路由策略，其使用ICMP redirect方式通知主机。icmp redirect 能避免带宽的不必要浪费。

`router-advertisement`和`router-solicitation`：这是一对报文，他们的作用是用来发现路由设备的地址，主机发出router-solicitation类型数据包来查找路由设备，路由设备可以发出router-advertisement类型ICMP数据包来告诉主机自己是路由设备

`source-quench`：当源地址设备（比如路由设备）资源紧张时就会发出这种数据包

`time-exceeded`：数据包超时。

### interface

interface有两个可以配置的位置：

- zone所对应的xml配置文件

- 网卡配置文件（也就是 ifcfg-* 文件）。

第一种配置跟source大同小异，这里就不再细述了，interface在zone配置文件中的节点为

```xml
<zone>
    <interface name="string"/>
</zone>
```

- 相关操作命令:

```bash
firewall-cmd [--permanent] [--zone=zone] --list-interfaces
firewall-cmd [--permanent] [--zone=zone] --add-interface=interface
firewall-cmd [--zone=zone] --change-interface=interface
firewall-cmd [--permanent] [--zone=zone] --query-interface=interface
firewall-cmd [--permanent] [--zone=zone] --remove-interface=interface
```

另外，还可以在网卡配置文件中进行配置，比如可以在ifcfg-eno16777728文件中添加下面的配置

```bash
ZONE=public
```

这行配置就相当于下面的命令

```bash
firewall-cmd --zone=public --change-interface=em1
```

这样配置之后来自em1的连接就会使用public这个zone进行管理（如果source匹配了其他的zone除外）。

### masquerad

打开规则里的 IP 伪装。用源地址而不是目的地址来把伪装限制在这个区域内。在此，指定一个动作是不被允许的。

masquerade 其作用就是ip地址伪装，也就是NAT转发中的一种，具体处理方式是将接收到的请求的源地址设置为转发请 求网卡的地址，这在路由器等相关设备中非常重要，比如很多都使用的是路由器连接的局域网，而想上互联网就得将ip地址给修改一下，要不然都是 192.168.1.XXX的内网地址，那请求怎么能正确返回呢？所以在路由器中将请求实际发送到互联网的时候就会将请求的源地址设置为路由器的外网地 址，这样请求就能正确地返回给路由器了，然后路由器再根据记录返回给发送请求的主机了，这就是masquerade。

其设置非常简单，在zone中是一个没有参数（属性）的节点:

```xml
<zone>
    <masquerade/>
</zone>
```

- 相应操作命令为

```bash
firewall-cmd [--permanent] [--zone=zone] --add-masquerade [--timeout=seconds]
firewall-cmd [--permanent] [--zone=zone] --remove-masquerade
firewall-cmd [--permanent] [--zone=zone] --query-masquerade
```

### port

port 是直接对端口的操作，他和 service 非常相似，所以这里也不详细介绍了，port 在 zone 中的配置节点为

```xml
<zone>
    <port port="portid[-portid]" protocol="tcp|udp"/>
</zone>
```

- 相应操作命令为

```bash
firewall-cmd [--permanent] [--zone=zone] --list-ports
firewall-cmd [--permanent] [--zone=zone] --add-port=portid[-portid]/protocol [--timeout=seconds]
firewall-cmd [--permanent] [--zone=zone] --remove-port=portid[-portid]/protocol
firewall-cmd [--permanent] [--zone=zone] --query-port=portid[-portid]/protocol
```

### rich-rule

rich-rule可以用来定义一条复杂的规则，其在zone配置文件中的节点定义如下:

```xml
<zone>
    <rule [family="ipv4|ipv6"]>
               [ <source address="address[/mask]" [invert="bool"]/> ]
               [ <destination address="address[/mask]" [invert="bool"]/> ]
               [
                 <service name="string"/> |
                 <port port="portid[-portid]" protocol="tcp|udp"/> |
                 <protocol value="protocol"/> |
                 <icmp-block name="icmptype"/> |
                 <masquerade/> |
                 <forward-port port="portid[-portid]" protocol="tcp|udp" [to-port="portid[-portid]"] [to-addr="address"]/>
               ]
               [ <log [prefix="prefixtext"] [level="emerg|alert|crit|err|warn|notice|info|debug"]/> [<limit value="rate/duration"/>] </log> ]
               [ <audit> [<limit value="rate/duration"/>] </audit> ]
               [ <accept/> | <reject [type="rejecttype"]/> | <drop/> ]
     </rule>
</zone>
```

可以看到这里一条rule的配置的配置项非常多，比zone本身还多出了destination、log、audit等配置项。

其实这里的rule就相当于使用iptables时的一条规则。rule的操作命令如下

```bash
firewall-cmd [--permanent] [--zone=zone] --list-rich-rules
firewall-cmd [--permanent] [--zone=zone] --add-rich-rule='rule' [--timeout=seconds]
firewall-cmd [--permanent] [--zone=zone] --remove-rich-rule='rule'
firewall-cmd [--permanent] [--zone=zone] --query-rich-rule='rule'
```

这里的参数`'rule'`代表一条规则语句，语句结构就是直接按照上面的节点结构去掉尖括号来书写就可以了，比如要设置地址为 192.168.10.10的source就可以写成source address="192.168.10.10"，也就是直接写标签名，然后跟着写属性就可以了，例如:

```bash
firewall-cmd --add-rich-rule='rule family="ipv4" source address="192.168.10.10" drop'
firewall-cmd --permanent --add-rich-rule 'rule family=ipv4 source address=172.25.0.0/24 forward-port port=5423 protocol=tcp to-port=80'
```

### service

service 是 firewalld 提供的其中一种服务。

service 的配置和source基本相同，只不过同一个service可以配置到多个不同的zone中，当然也就不需要`--change`命令了，他在zone配置文件中的节点为:

```xml
<zone>
    <service name="string"/>
</zone>
```

- 相应操作命令为

```bash
firewall-cmd [--permanent] [--zone=zone] --get-services
firewall-cmd [--permanent] [--zone=zone] --list-services
firewall-cmd [--permanent] [--zone=zone] --add-service=service [--timeout=seconds]
firewall-cmd [--permanent] [--zone=zone] --remove-service=service
firewall-cmd [--permanent] [--zone=zone] --query-service=service
```

`--add-service`中的`--timeout`的含义是这样的：添加一个服务，但是不是一直生效而是生效一段时间，过期之后自动删除。

这个选项非常有用，比如想暂时开放一个端口进行一些特殊的操作（比如远程调试），等处理完成后再关闭，不过有时候,处理完之后就忘记关闭了， 而现在的`--timeout`选项就可以很好地解决这个问题，在打开的时候就可以直接设置一个时间，到时间之后他自动就可以关闭了。另外，这个参 数还有更有用的用法,当然`--timeout`和`--permanent`是不可以一起使用的。

### source

source是在zone的xml文件中配置的，其格式为

```xml
<zone>
    <source address="address[/mask]"/>
</zone>
```

只要将source节点放入相应的zone配置文件中就可以了，节点的address属性就是源地址，不过要注意相同的source节点只 可以在一个zone中进行配置，也就是说同一个源地址只能对于一个zone，另外，直接编辑xml文件之后还需要reload才可以起作用.

使用firewall-cmd命令进行配置，这里主要有五个相关命令（参数）

```bash
firewall-cmd [--permanent] [--zone=zone] --list-sources
firewall-cmd [--permanent] [--zone=zone] --query-source=source[/mask]
firewall-cmd [--permanent] [--zone=zone] --add-source=source[/mask]
firewall-cmd [--zone=zone] --change-source=source[/mask]
firewall-cmd [--permanent] [--zone=zone] --remove-source=source[/mask]
```

- `--list-sources`：用于列出指定zone的所有绑定的source地址

- `--query-source`：用于查询指定zone是否跟指定source地址进行了绑定

- `--add-source`：用于将一个source地址绑定到指定的zone（只可绑定一次，第二次绑定到不同的zone会报错）

- `--change-source`：用于改变source地址所绑定的zone，如果原来没有绑定则进行绑定，这样就跟`--add-source`的作用一样了

- `--remove-source`：用于删除source地址跟zone的绑定

命令中有两个可选参数：`--permanent`和`--zone`:

`--permanent` 表示存储到配置文件中（如果存储到配置文件中不会立即生效）;

`--zone` 用于指定所要设置的zone，如果不指定则使用默认zone。

例如:

```bash
firewall-cmd --zone=drop --change-source=192.168.10.10
```

这样就可以将192.168.10.10绑定到drop这个zone中了，如果没有修改过drop规则的话所有来自192.168.10.10这个ip的连接将会被drop。

至于什么时候使用add什么时候使用change，如果只是想将某源地址绑定到指定的zone那么最好使用change，而如果想在源地址没绑定的时候进行绑定，如果已经绑定过则不绑定那么就使用add。

## direct

使用他可以直接使用iptables、ip6tables中的规则进行配置。

### direct结构

先从配置文件入手，direct的配置文件为/etc/firewalld/direct.xml文件，结构如下

```xml
<?xml version="1.0" encoding="utf-8"?>
<direct>
   [ <chain ipv="ipv4|ipv6" table="table" chain="chain"/> ]
   [ <rule ipv="ipv4|ipv6" table="table" chain="chain" priority="priority"> args </rule> ]
   [ <passthrough ipv="ipv4|ipv6"> args </passthrough> ]
</direct>
```

可以看到这里的direct一共有三种节点：chain、rule和passthrough，他们都是可选的，而且都可以出现多次。

### 属性

- `ipv`：这个属性非常简单，表示ip的版本

- `table`：chain和rule节点中的table属性就是iptables/ip6tables中的table

- `chain`：chain中的chain属性用于指定一个自定义链的名字，注意，不可与已有链重名；rule中的chain属性既可以是内建的（也就是iptables/ip6tables中的五条链），也可以是在direct中自定义的chain

- `priority`：优先级，用于设置不同rule的优先级，就像iptables中规则的前后顺序，数字越小优先级越高

- `args`：rule和passthrough中的args就是iptables/ip6tables中的一条具体的规则，不过他们可以使用自定义的chain。

### 使用

因为direct的使用跟iptables/ip6tables非常相似，换句话说，想用好direct需要有iptables/ip6tables的基础

direct中自定义chain跟iptables/ip6tables中使用-N新建chain类似，创建完之后就可以在规则中使用-j或者-g来使用了。

Firewalld中跟direct相关的命令如下:

```bash
firewall-cmd [--permanent] --direct --get-all-chains
firewall-cmd [--permanent] --direct --get-chains { ipv4 | ipv6 | eb } table
firewall-cmd [--permanent] --direct --add-chain { ipv4 | ipv6 | eb } table chain
firewall-cmd [--permanent] --direct --remove-chain { ipv4 | ipv6 | eb } table chain
firewall-cmd [--permanent] --direct --query-chain { ipv4 | ipv6 | eb } table chain

firewall-cmd [--permanent] --direct --get-all-rules
firewall-cmd [--permanent] --direct --get-rules { ipv4 | ipv6 | eb } table chain
firewall-cmd [--permanent] --direct --add-rule { ipv4 | ipv6 | eb } table chain priority args
firewall-cmd [--permanent] --direct --remove-rule { ipv4 | ipv6 | eb } table chain priority args
firewall-cmd [--permanent] --direct --remove-rules { ipv4 | ipv6 | eb } table chain
firewall-cmd [--permanent] --direct --query-rule { ipv4 | ipv6 | eb } table chain priority args

firewall-cmd --direct --passthrough { ipv4 | ipv6 | eb } args
firewall-cmd --permanent --direct --get-all-passthroughs
firewall-cmd --permanent --direct --get-passthroughs { ipv4 | ipv6 | eb }
firewall-cmd --permanent --direct --add-passthrough { ipv4 | ipv6 | eb } args
firewall-cmd --permanent --direct --remove-passthrough { ipv4 | ipv6 | eb } args
firewall-cmd --permanent --direct --query-passthrough { ipv4 | ipv6 | eb } args
```

下面是文档（firewalld.direct(5)）中提供的例子

```xml
<?xml version="1.0" encoding="utf-8"?>
<direct>
    <chain ipv="ipv4" table="raw" chain="blacklist"/>
    <rule ipv="ipv4" table="raw" chain="PREROUTING" priority="0">-s 192.168.1.0/24 -j blacklist</rule>
    <rule ipv="ipv4" table="raw" chain="PREROUTING" priority="1">-s 192.168.5.0/24 -j blacklist</rule>
    <rule ipv="ipv4" table="raw" chain="blacklist" priority="0">-m limit --limit 1/min -j LOG --log-prefix "blacklisted: "</rule>
    <rule ipv="ipv4" table="raw" chain="blacklist" priority="1">-j DROP</rule>
</direct>
```

在这个例子中首先自定义了一个叫blacklist的链，然后将所有来自192.168.1.0/24和192.168.5.0/24的数据包都指向了这个链，最后定义了这个链的规则：首先进行记录，然后drop，记录的方法是使用“blacklisted: ”前缀并且限制1分钟记录一次。

## whitelist

白名单跟防火墙结合在一起很容易将其理解为规则白名单，不过在Firewalld中whitelist却并不是规则白名单的含义。

当服务器中的某个服务（比如http）出现漏洞时，攻击者如果可以执行命令那么是不是就可以使用firewall-cmd工具来修防火墙的规则呢？如果真是这样那么后果可想而知，攻击者不但可以开放原来没有开放的端口，甚至还可以搞恶作剧——将正常服务的端口给关闭！

Firewalld中whitelist就是来解决这个问题的，他可以限制谁能对防火墙规则进行修改，也就是说这里的whitelist其实使用用来配置可以修改防火墙规则的主体的白名单。

### 使用条件

在默认配置下whitelist是不启用的，需要将Lockdown设置为yes才可以启用.

- 相应操作命令为

```bash
firewall-cmd --lockdown-on
firewall-cmd --lockdown-off
firewall-cmd --query-lockdown
```

第一个是开启Lockdown，也就是让whitelist起作用，第二个是关闭Lockdown，第三个是查询当前Lockdown的状态。

对Lockdown进行修改时配置文件和运行时环境会同时进行修改，当使用firewall-cmd命令对Lockdown的状态进行修改后首先可以立即生效,在重启后也不会失效。

另外，在使用--lockdown-on的时候要特别小心，要先看自己在不在whitelist范围内，如果不在，启用之后自己也不可以对防火墙进行操作了！

### 配置文件

whitelist的配置文件是位于/etc/firewalld目录下的lockdown-whitelist.xml文件，其结构如下

```xml
<whitelist>
    [<selinux context="selinuxcontext"/>]
    [<command name="commandline[*]"/>]
    [<user {name="username"|id="userid"}/>]
</whitelist>
```

这里面有三个可选的配置节点：selinux、command和user，每个配置节点都可以配置多个，配置进来的就表示可以修改防火墙规则。

#### selinux

一说到selinux可能有的人就会产生畏惧心，不过这里用到的非常简单，只需要将某进程的content给设置进去就行了，具体某个进程的content可以使用`ps -e --context`命令来查找，找出来之后设置到context属性中就可以了。

可以直接编辑xml配置文件，另外也可以使用firewall-cmd命令来操作，相关命令如下

```bash
firewall-cmd [--permanent] --add-lockdown-whitelist-context=context
firewall-cmd [--permanent] --remove-lockdown-whitelist-context=context
firewall-cmd [--permanent] --query-lockdown-whitelist-context=context
firewall-cmd [--permanent] --list-lockdown-whitelist-contexts
```

这四个命令也非常容易理解，他们分别表示添加、删除、查询一个具体的selinuxcontenxt以及罗列出所有白名单中配置了的selinuxcontenxt，使用--permanent可以持久化保存，不使用可以立即生效。

#### command

通过command节点可以针对具体的command命令进行配置，配置之后此命令就可以被一般用户执行了。比如想将panic模式的开启和关闭命令开发，这样当遇到紧急情况时一般用户也可以启动panic模式，这种需求可以使用下面的配置即可

```xml
<whitelist>
    <command name="/usr/bin/python /bin/firewall-cmd --panic-on"/>
    <command name="/usr/bin/python /bin/firewall-cmd --panic-off"/>
</whitelist>
```

另外，command还可以使用通配符 `*`，所以上面的配置还可以简化为

```xml
<whitelist>
    <command name="/usr/bin/python /bin/firewall-cmd --panic-*"/>
</whitelist>
```

当然，command也可以使用firewall-cmd命令来操作，相关命令如下

```bash
firewall-cmd [--permanent] --add-lockdown-whitelist-command=command
firewall-cmd [--permanent] --remove-lockdown-whitelist-command=command
firewall-cmd [--permanent] --query-lockdown-whitelist-command=command
firewall-cmd [--permanent] --list-lockdown-whitelist-commands
```

#### user

这里的user指的就是linux中的用户，通过这项可以对指定的用户开放配置权限，指定用户的方法有两种：通过userId和通过userName都可以，在默认的lockdown-whitelist.xml配置文件中就设置了id为0的user，也就是root用户

```xml
<whitelist>
  ...
  <user id="0"/>
</whitelist>
```

当然，通过name属性设置用户名也是可以的。user也可以使用firewall-cmd命令来操作，而且uid和name是分开操作的.

user相关的命令一共有八个:

```bash
firewall-cmd [--permanent] --add-lockdown-whitelist-uid=uid
firewall-cmd [--permanent] --remove-lockdown-whitelist-uid=uid
firewall-cmd [--permanent] --query-lockdown-whitelist-uid=uid
firewall-cmd [--permanent] --list-lockdown-whitelist-uids

firewall-cmd [--permanent] --add-lockdown-whitelist-user=user
firewall-cmd [--permanent] --remove-lockdown-whitelist-user=user
firewall-cmd [--permanent] --query-lockdown-whitelist-user=user
firewall-cmd [--permanent] --list-lockdown-whitelist-users
```

前四个是对uid进行操作，后四个是对username进行操作，具体含义应该很容易理解。

- 特别注意

> 在使用whitelist的时候要特别注意一点，whitelist只针对规则的修改（包括添加和删除）起作用，但不会限制查询。如果使用root配置好防火墙后很少修改，也没有使用脚本动态修改等特殊需求的话可以将/bin/firewall-cmd的权限设置为750或者更低。

## panic模式

Firewalld有一种Panic模式，Panic的单词含义为“恐慌”、“惊慌”，在firewalld中他表示当发生紧急情况（比如遭到攻击）时启用的一种“禁行模式”，启用这种模式后所有的进包和出包都会被丢弃，和panic模式相关的有三个命令

```bash
firewall-cmd --panic-on
firewall-cmd --panic-off
firewall-cmd --query-panic
```

这三个命令很容易理解，第一个是启用panic模式，也就是“禁行模式”，第二个是禁用panic模式，第三个是查询是否已启用panic模式。

当启用了panic模式后所有的进包和出包都会被丢弃，不过如果对于原来已经建立的连接并不会马上断开，只是双方不能进行通信了而已，当达到设置的最长不活动（inactivity）周期后才会断开，而如果在断开前将panic模式关闭的话连接就不会受影响。

因为启用panic模式后会丢弃所有进包和出包，所以使用时要格外谨慎，另外，如果是使用的ssh连接的话，启动panic模式后ssh的连接也会被断开（准确来说是不可通信了），这时更加需要注意。

- firewall-cmd的本质

firewall-cmd其实是一个位于/usr/bin目录下的Python脚本，如果想了解firewall-cmd命令的具体的细节而且又熟悉Python语言的话就可以直接打开这个文件进行代码阅读。

另外，这个命令有一个对于安全来说非常重要但是又很不容易引起注意的问题，首先来看一下这个脚本文件的属性

```bash
ll /usr/bin/firewall-cmd
-rwxr-xr-w. 1 root root 62012 Nov 20 20:35 /usr/bin/firewall-cmd
```

可以看到这里的权限是755，也就是说所有用户都可以执行该命令，当然，这么设计主要是为了使用其他程序通过D-BUS接口来操作firewalld有关，可以通过whitelist来设置，不过只有将Lockdown配置为yes后whitelist才会生效，而且默认配置为no，也就是说默认情况下所有程序（用户）都可以执行firewall-cmd命令，这当然是不安全的，如果不需要使用其他程序对其进行操作的话可以直接将其权限改为750，这样更加安全

```bash
chmod 750 /usr/bin/firewall-cmd
```

## 参考

- [Excelib](http://www.excelib.com/article/293/show/)
- [RedHat](https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/html/security_guide/sec-using_firewalls#sec-Choosing_a_Network_Zone)

```pdf
https://access.redhat.com/documentation/zh-cn/red_hat_enterprise_linux/7/pdf/security_guide/Red_Hat_Enterprise_Linux-7-Security_Guide-zh-CN.pdf
```
