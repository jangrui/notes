# Linux 命令

好记性不如烂笔头，学习Linux命令笔记。

- [常用命令](/command/常用命令)

- [文件目录操作]

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
|[dumpe2fs](command/dumpe2fs) 		| 显示 ext2/ext3/ext4 文件系统信息|
|[e2fsck](command/e2fsck)			| 检查 ext2/3/4 文件系统|
|[resize2fs](command/resize2fs)		| 加载 ext2/3/4 文件系统|
|[fsck](command/fsck)				| 文件系统检查修复|
|[df](command/df)					| 显示目前在 Linux 系统上的文件系统的磁盘使用情况统计|
|[du](command/du)					| 显示目录或文件的大小|

- [文字处理]

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


- [文本处理三剑客]
|命令								|描述|
|-									|-|
|[grep](command/grep)				|文本过滤工具|
|[sed](command/sed)					|字符流编辑器|
|[awk](command/awk)					|不仅是文本工具,更是一个编程语言|

- [信息显示与搜索文件]

|命令								|描述|
|-									|-|
|[uname](command/uname)				|显示系统信息|
|[hostname](command/hostname)		|显示或设置系统主机名|
|[dmesg](command/dmesg)				|系统启动异常诊断|
|[stat](command/stat)				|
|[du](command/du)					|
|[data](command/data)				|
|[echo](command/echo)				|
|[watch](command/watch)				|
|[which](command/which)				|
|[whereis](command/whereis)			|
|[locate](command/locate)			|
|[updatedb](command/updatedb)		|	

- [文件备份与压缩]

|命令								|描述|
|-									|-|
|[tar](command/tar)					|
|[gzip](command/gzip)				|
|[zip](command/zip)					|
|[unzip](command/unzip)				|
|[scp](command/scp)					|
|[rsync](command/rsync)				|

- [用户管理及用户信息查询]

|命令								|描述|
|-									|-|
|[useradd](command/useradd)			|
|[usermod](command/usermod)			|
|[userdel](command/userdel)			|
|[groupadd](command/groupadd)		|
|[groupdel](command/groupdel)		|
|[passwd](command/passwd)			|
|[chage](command/chage)				|
|[chpasswd](command/chpasswd)		|
|[su](command/su)					|
|[visudo](command/visudo)			|
|[sudo](command/sudo)				|
|[id](command/id)					|
|[w](command/w)						|
|[who](command/who)					|
|[users](command/users)				|
|[whoami](command/whoami)			|
|[last](command/last)				|
|[lastb](command/lastb)				|
|[lastlog](command/lastlog)			|

- [磁盘与文件系统管理]

|命令								|描述|
|-									|-|
|[fdisk](command/fdisk)				|
|[partprob](command/partprob)		|
|[tune2fs](command/tune2fs)			|
|[parted](command/parted)			|
|[mkfs](command/mkfs)				|
|[dumpe2fs](command/dumpe2fs)		|
|[resize2fs](command/resize2fs)		|
|[fsck](command/fsck)				|
|[dd](command/dd)					|
|[mount](command/mount)				|
|[umount](command/umount)			|
|[df](command/df)					|
|[mkswap](command/mkswap)			|
|[swapon](command/swapon)			|
|[swapoff](command/swapoff)			|
|[sync](command/sync)				|

- [进程管理]

|命令								|描述|
|-									|-|
|[ps](command/ps)					|
|[pstree](command/pstree)			|
|[pgrep](command/pgrep)				|
|[kill](command/kill)				|
|[killall](command/killall)			|
|[pkill](command/pkill)				|
|[top](command/top)					|
|[nice](command/nice)				|
|[renice](command/renice)			|
|[nohup](command/nohup)				|
|[strace](command/strace)			|
|[ltrace](command/ltrace)			|
|[runlevel](command/runlevel)		|
|[init](command/init)				|
|[service](command/service)			|

- [网络管理]

|命令								|描述|
|-									|-|
|[ifconfig](command/ifconfig)		|
|[ifup](command/ifup)				|
|[ifdown](command/ifdown)			|
|[route](command/route)				|
|[arp](command/arp)					|
|[ip](command/ip)					|
|[netstat](command/netstat)			|
|[ss](command/ss)					|
|[ping](command/ping)				|
|[traceroute](command/traceroute)	|
|[arping](command/arping)			|
|[telnet](command/telnet)			|
|[nc](command/nc)					|
|[ssh](command/ssh)					|
|[wget](command/wget)				|
|[mailq](command/mailq)				|
|[mail](command/mail)				|
|[nslookup](command/nslookup)		|
|[dig](command/dig)					|
|[host](command/host)				|
|[nmap](command/nmap)				|
|[tcpdump](command/tcpdump)			|

- [系统管理]

|命令								|描述|
|-									|-|
|[lsof](command/lsof)				|
|[uptime](command/uptime)			|
|[free](command/free)				|
|[iftop](command/iftop)				|
|[vmstat](command/vmstat)			|
|[mpstat](command/mpstat)			|
|[iostat](command/iostat)			|
|[iotop](command/iotop)				|
|[sar](command/sar)					|
|[chkconfig](command/chkconfig)		|
|[ntsysv](command/ntsysv)			|
|[setup](command/setup)				|
|[ethtool](command/ethtool)			|
|[mii-tool](command/miitool)		|
|[dmidecode](command/dmidecode)		|
|[lspci](command/lspci)				|
|[ipcs](command/ipcs)				|
|[ipcrm](command/ipcrm)				|
|[rpm](command/rpm)					|
|[yum](command/yum)

- [系统常用内置命令]

|[alias](command/alias)
|[bg](command/bg)
|[bind](command/bind)
|[break](command/break)
|[builtin](command/builtin)
|[caller]()
|[command](command/command)
|[compgen](command/compgen)
|[complete](command/complete)
|[compopt](command/compopt)
|[continue](command/continue)
|[declare](command/declare)
|[dirs](command/dirs)
|[disown](command/disown)
|[echo](command/echo)
|[enable](command/enable)
|[eval](command/eval)
|[exec](command/exec)
|[exit](command/exit)
|[export](command/export)
|[false](command/false)
|[fc](command/fc)
|[fg](command/fg)
|[getopts](command/getopts)
|[hash](command/hash)
|[help](command/help)
|[history](command/history)
|[jobs](command/jobs)
|[let](command/let)
|[local](command/local)
|[logout](command/logout)
|[mapfile]()
|[popd]()
|[printf]()
|[pushd]()
|[pwd]()
|[read]()
|[readonly]()
|[return]()
|[set]()
|[shift]()
|[shopt]()
|[source]()
|[suspend]()
|[test]()
|[times]()
|[trap]()
|[true]()
|[type]()
|[typeset]()
|[ulimit]()
|[umask]()
|[unalias]()
|[unset]()
|[wait]()