# Linux 命令

好记性不如烂笔头，学习Linux命令笔记。

- [常用命令](/command/常用命令)

- 文件目录操作

|命令								|描述|
|-									|-|
|[pwd](command/pwd)					|显示当前所在的位置|
|[cd](command/cd)					|切换目录|
|[tree](command/tree)				|以树目录形结构显示目录下的内容|
|[mkdir](command/mkdir)				|创建目录|
|[touch](command/touch)				|创建文件或改变文件的时间戳|
|[ls](command/ls)					|列出目录下的内容|
|[cp](command/cp)					|复制文件或目录|
|[mv](command/mv)					|移动或重命名文件|
|[rm](command/rm)					|删除文件或目录|
|[rmdir](command/rmdir)				|删除空目录|
|[ln](command/ln)					|硬链接与软连接|
|[readlink](command/readlink)		|查看符号链接文件的内容|
|[find](command/find)				|查找目录下的文件|
|[xargs](command/xargs)				|将标准输入转换成命令行参数|
|[rename](/command/rename)			|重命名文件|
|[basename](/command/basename)		|显示文件名或目录名|
|[dirname](command/dirname)			|显示文件或目录的路径|
|[chattr](command/chattr)			|改变文件扩展属性|
|[lsattr](command/lsattr)			|查看文件扩展属性|
|[file](command/file)				|显示文件的类型|
|[md5sum](command/md5sum)			|计算和校验文件的MD5值|
|[chown](command/chown)				|改变文件或目录的用户和用户组|
|[chmod](command/chmod)				|改变文件或目录的权限|
|[chgrp](command/chgrp)				|改变文件或目录的用户组|
|[umask](command/umask)				|显示或设置权限掩码|


- 防火墙

|命令									|描述|
|-										|-|
|[iptables](/command/iptables)			|iptables用于过滤数据包，属于网络层防火墙|
|[firewall-cmd](command/firewall-cmd)	|可用于服务、端口等，属于更高一层的防火墙|
|[TCP Wrappers](command/TCP-Wrappers)	|TCP_Wrappers是一个工作在第四层（传输层）的的安全工具|

- 文件系统管理

|命令								|描述|
|-									|-|
|[dumpe2fs](command/dumpe2fs) 		|显示 ext2/ext3/ext4 文件系统信息|
|[e2fsck](command/e2fsck)			|检查 ext2/3/4 文件系统|
|[resize2fs](command/resize2fs)		|加载 ext2/3/4 文件系统|
|[fsck](command/fsck)				|文件系统检查修复|
|[df](command/df)					|显示目前在 Linux 系统上的文件系统的磁盘使用情况统计|
|[du](command/du)					|显示目录或文件的大小|

- 文字处理

|命令								|描述|
|-									|-|
|[bc](command/bc)					| 计算器 |
|[xargs](command/xargs)				| 作用是将列表转换成小块分段传递给其他命令|
|[cat](command/cat)					|查看文件内容或合并文件|
|[tac](command/tac)					|反向查看文件内容|
|[more](command/more)				|分页显示文件内容|
|[less](command/less)				|分页显示文件内容(more的升级版)|
|[head](command/head)				|显示文件内容头部|
|[tail](command/tail)				|显示文件内容尾部|
|[tailf](command/tailf)				|跟踪日志文件|
|[cut](command/cut)					|从文本中提取一段文字并输出|
|[split](command/split)				|分割文件|
|[paste](command/paste)				|合并文件|
|[sort](command/sort)				|文本排序|
|[join](command/join)				|按两个文件的相同字段合并|
|[uniq](command/uniq)				|去除重复行|
|[wc](command/wc)					|统计文件的行数|
|[iconv](command/iconv)				|转换文件的编码格式|
|[dos2unix](command/dos2unix)		|将DOS格式文件转换成UNIX格式|
|[diff](command/diff)				|比较两个文件的不同|
|[vimdiff](command/vimdiff)			|可视化比较工具|
|[rev](command/rev)					|反向输出文件内容|
|[tr](command/tr)					|替换或删除字符|
|[od](command/od)					|按不同进制显示文件|
|[tee](command/tee)					|多重定向|
|[vi/vim](command/vim)				|纯文本编辑器|


- 文本处理三剑客

|命令								|描述|
|-									|-|
|[grep](command/grep)				|文本过滤工具|
|[sed](command/sed)					|字符流编辑器|
|[awk](command/awk)					|不仅是文本工具,更是一个编程语言|

- 信息显示与搜索文件

|命令								|描述|
|-									|-|
|[uname](command/uname)				|显示系统信息|
|[hostname](command/hostname)		|显示或设置系统主机名|
|[dmesg](command/dmesg)				|系统启动异常诊断|
|[stat](command/stat)				|显示文件或文件系统状态|
|[du](command/du)					|统计磁盘空间使用情况|
|[data](command/data)				|显示与设置系统时间|
|[echo](command/echo)				|显示一行文本|
|[watch](command/watch)				|监视命令|
|[which](command/which)				|显示命令的全路径|
|[whereis](command/whereis)			|显示命令及其相关文件全路径|
|[locate](command/locate)			|快速定位文件路径|
|[updatedb](command/updatedb)		|更新 mlocate 数据库|

- 文件备份与压缩

|命令								|描述|
|-									|-|
|[tar](command/tar)					|打包备份文件|
|[gzip](command/gzip)				|压缩或解压文件|
|[zip](command/zip)					|打包压缩文件|
|[unzip](command/unzip)				|解压 zip 文件|
|[scp](command/scp)					|远程文件复制|
|[rsync](command/rsync)				|文件同步工具|

- 用户管理及用户信息查询

|命令								|描述|
|-									|-|
|[useradd](command/useradd)			|创建用户|
|[usermod](command/usermod)			|修改用户信息|
|[userdel](command/userdel)			|删除用户|
|[groupadd](command/groupadd)		|创建新的用户组|
|[groupdel](command/groupdel)		|删除用户组|
|[passwd](command/passwd)			|修改用户密码|
|[chage](command/chage)				|修改用户密码有效期|
|[chpasswd](command/chpasswd)		|批量更新用户密码|
|[su](command/su)					|切换用户|
|[visudo](command/visudo)			|编辑 sudoers 文件|
|[sudo](command/sudo)				|以另一个管理员身份执行命令|
|[id](command/id)					|显示用户与用户组信息|
|[w](command/w)						|显示已登录用户详细信息|
|[who](command/who)					|显示已登录用户信息|
|[users](command/users)				|显示已登录用户|
|[whoami](command/whoami)			|显示当前登录的用户名|
|[last](command/last)				|显示用户登录列表|
|[lastb](command/lastb)				|显示用户登录失败的记录|
|[lastlog](command/lastlog)			|显示所有用户的最近登录记录|

- 磁盘与文件系统管理

|命令								|描述|
|-									|-|
|[fdisk](command/fdisk)				|磁盘分区工具|
|[partprob](command/partprob)		|更新内核的硬盘分区表信息|
|[tune2fs](command/tune2fs)			|调整 ext2/ext3/ext4 文件系统参数|
|[parted](command/parted)			|磁盘分区工具|
|[mkfs](command/mkfs)				|创建 linux 文件系统|
|[dumpe2fs](command/dumpe2fs)		|导出 ext2/ext3/ext4 文件系统信息|
|[resize2fs](command/resize2fs)		|调整 ext2/ext3/ext4 文件系统大小|
|[fsck](command/fsck)				|检查并修复 linux 文件系统|
|[dd](command/dd)					|转换或复制文件|
|[mount](command/mount)				|挂载文件系统|
|[umount](command/umount)			|卸载文件系统|
|[df](command/df)					|报告文件系统磁盘空间的使用情况|
|[mkswap](command/mkswap)			|创建交换分区|
|[swapon](command/swapon)			|激活交换分区|
|[swapoff](command/swapoff)			|关闭交换分区|
|[sync](command/sync)				|刷新文件系统缓冲区|

- 进程管理

|命令								|描述|
|-									|-|
|[ps](command/ps)					|查看进程|
|[pstree](command/pstree)			|显示进程状态树|
|[pgrep](command/pgrep)				|查找匹配条件的进程|
|[kill](command/kill)				|终止进程|
|[killall](command/killall)			|通过进程名终止进程|
|[pkill](command/pkill)				|通过进程名终止进程|
|[top](command/top)					|实时显示系统中各个进程的资源占用状况|
|[nice](command/nice)				|调整程序运行时的优先级|
|[renice](command/renice)			|调整运行中的进程的优先级|
|[nohup](command/nohup)				|用户退出系统进程继续工作|
|[strace](command/strace)			|跟踪进程的系统调用|
|[ltrace](command/ltrace)			|跟踪进程调用库函数|
|[runlevel](command/runlevel)		|输出当前运行级别|
|[init](command/init)				|初始化 linux 进程|
|[service](command/service)			|管理系统服务|

- 网络管理

|命令								|描述|
|-									|-|
|[ifconfig](command/ifconfig)		|配置或显示网络接口信息|
|[ifup](command/ifup)				|激活网络接口|
|[ifdown](command/ifdown)			|禁用网络接口|
|[route](command/route)				|显示或管理路由表|
|[arp](command/arp)					|管理系统的 arp 缓存|
|[ip](command/ip)					|网络配置工具|
|[netstat](command/netstat)			|查看网络状态|
|[ss](command/ss)					|查看网络状态|
|[ping](command/ping)				|测试主机之间网络的连通性|
|[traceroute](command/traceroute)	|追踪数据传输路由状况|
|[arping](command/arping)			|发送 arp 请求|
|[telnet](command/telnet)			|远程登录主机|
|[nc](command/nc)					|多功能网络工具|
|[ssh](command/ssh)					|安全地远程登录主机|
|[wget](command/wget)				|命令行下载工具|
|[mailq](command/mailq)				|显示邮件传输队列|
|[mail](command/mail)				|发送和接受邮件|
|[nslookup](command/nslookup)		|域名查询工具|
|[dig](command/dig)					|域名查询工具|
|[host](command/host)				|域名查询工具|
|[nmap](command/nmap)				|网络探测工具和安全/端口扫描器|
|[tcpdump](command/tcpdump)			|监听网络流量|

- 系统管理

|命令								|描述|
|-									|-|
|[lsof](command/lsof)				|查看进程打开的文件|
|[uptime](command/uptime)			|显示系统的运行时间及负载|
|[free](command/free)				|查看系统内存信息|
|[iftop](command/iftop)				|动态显示网络接口流量信息|
|[vmstat](command/vmstat)			|虚拟内存统计|
|[mpstat](command/mpstat)			|CPU 信息统计|
|[iostat](command/iostat)			|I/O信息统计|
|[iotop](command/iotop)				|动态显示磁盘I/O统计信息|
|[sar](command/sar)					|收集系统信息|
|[chkconfig](command/chkconfig)		|管理开机服务|
|[ntsysv](command/ntsysv)			|管理开机服务|
|[setup](command/setup)				|系统管理工具|
|[ethtool](command/ethtool)			|查询网络参数|
|[mii-tool](command/miitool)		|管理网络接口的状态|
|[dmidecode](command/dmidecode)		|查询系统硬件信息|
|[lspci](command/lspci)				|显示所有 PCI 设备|
|[ipcs](command/ipcs)				|显示进程间通信设施的状态|
|[ipcrm](command/ipcrm)				|清楚 ipc 相关信息|
|[rpm](command/rpm)					|RPM 包管理器|
|[yum](command/yum)

- 系统常用内置命令

|命令								|描述|
|-									|-|
|`:`								|执行完这个命令不会对系统造成任何影响|
|`.`								|在当前的 shell 环境中执行 shell 脚本，和 `source` 功能一样|
|`[`								|构造条件测试表达式，常用语 shell 脚本，功能类似于命令 `test`|
|`alias`							|显示和创建已有命令的别名|
|`bg`								|把任务放到后台|
|`bind`								|显示和设置命令行的键盘序列绑定功能|
|`break`							|跳出循环，常用于 shell 脚本的循环语句|
|`builtin`							|运行一个内置 shell 命令|
|`caller`							|返回所有活动子函数调用的上下文|
|`cd`								|切换目录，具体使用方法见 [cd](command/cd) 命令|
|`command`							|即使有同名函数，也仍然执行该命令|
|`compgen`							|筛选补全结果|
|`complete`							|指定可以补全的参数|
|`compopt`							|修改补全设置|
|`continue`							|忽略本次循环的剩余代码，进入下一次循环，常用于 shell 脚本的循环语句|
|`declare`							|声明一个变量或变量类型|
|`dirs`								|显示当前储存目录的列表|
|`disown`							|从任务表中删除一个活动任务|
|`echo`								|显示一样文本，具体使用方法见 [echo](command/echo) 命令|
|`enable`							|启用或禁用内置命令|
|`eval`								|读入参数，并将它们组合成一个新的命令，然后执行|
|`exec`								|用于指定命令替换 shell 进程|
|`exit`								|退出 shell|
|`export`							|设置或显示环境变量|
|`false`							|错误，假|
|`fc`								|查看历史命令|
|`fg`								|把后台任务放到前台|
|`getopts`							|分析指定的位置参数|
|`hash`								|查找并记住指定命令的全路径名|
|`help`								|显示内置命令的帮助信息|
|`history`							|显示带行号的命令历史列表|
|`jobs`								|显示放到后台的任务|
|`kill`								|杀死指定进程，具体使用方法见 [kill](command/kill) 命令|
|`let`								|用来计算算数表达式的值，并把算数运算的结果赋值给变量|
|`local`							|用在函数中，把变量的作用域限制在函数内部|
|`logout`							|退出登录 shell|
|`mapfile`							|从标准输入读取数据并写入数组|
|`popd`								|从目录栈中删除项|
|`printf`							|使用格式化字符串显示文本|
|`pushd`							|向目录栈中增加项|
|`pwd`								|显示当前的工作目录，具体使用方法见 [pwd](command/pwd) 命令|
|`read`								|从标准输入读取一行。保存到变量中|
|`readonly`							|将变量设为只读，不允许重置该变量|
|`return`							|从函数中退出|
|[set](command/sed)					|设置并显示环境变量的值|
|`shift`							|将位置变量左移 n 位|
|`shopt`							|打开 / 关闭控制 shell 可选行为的变量值|
|`source`							|在当前的 shell 环境中执行 shell 脚本，与 `.` 的功能一样|
|`suspend`							|终止当前 shell 的运行（对登录 shell 无效）|
|`test`								|构造条件测试表达式，功能类似命令 `[` |
|`times`							|显示累计的用户和系统时间|
|`trap`								|抓取 shell 收到的信号|
|`true`								|正确，真|
|`type`								|显示命令的类型|
|`typeset`							|同 declare ，设置变量并赋予其属性|
|`ulimit`							|显示或设置进程可用资源的最大限额|
|`umask`							|为新建的文件和目录设置默认权限，具体使用方法见 [umask](/command/umask) 命令|
|`unalias`							|取消指定的命令别名设置|
|`unset`							|取消置顶变量的值或函数的定义|
|`wait`								|等待指定的进程完成，并返回退出状态码|