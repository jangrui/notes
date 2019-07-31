# find

find 命令用于查找目录下的文件，同时也可以调用其它命令执行相应的操作。

## 语法

```bash
find [-H] [-L] [-P] [-D debugopts] [-O level] [pathname] [expression]
find [选项]                                    [路径]     [操作语句]
```

![find命令语法](./find.png)

## 参数说明

- `pathname` 目标路径，例如用 `.` 来表示当前目录，用 `/` 来表示系统根目录。

### Options 模块

`-depth` 从指定目录下深层的子目录开始查找
`-maxdepth levels` 查找的最大目录级数，levels为自然数
`-regextype type` 改变正则表达式的模式。默认为`emacs`，还有`posix-awk`、`posix-basic`、`posix-egrep`、`posix-extended`

### Tests 模块

- `-mtime [-n|n|+n]` 按照文件的修改时间来查找文件：`-n` 表示文件更改时间距现在n天以内。`+n` 表示文件更改时间距现在n天以前。`n` 表示距离现在第n天。
- `-atime [-n|n|+n]` 按照文件的访问时间来查找文件，单位为天
- `-ctime [-n|n|+n]` 按照文件的状态时间来查找文件，单位为天
- `-amin` 按照文件的访问时间来查找文件，单位为分钟
- `-cmin` 按照文件的状态改变时间来查找文件，单位为分钟
- `-mmin` 按照文件的修改时间来查找文件，单位为分钟
- `-group` 按照文件所属的组来查找文件
- `-name` 按照文件名查找文件，只支持 `*`、`？`、`[]`等特殊通配符
- `-newer` 查找更改时间指定文件新的文件
- `-nogroup` 查找没有有效用户组的文件，即该文件所属的组在`/etc/groups`中不存在
- `-nouser` 查找没有有效属主的文件，即该文件的属主在`/etc/passwd`中不存在
- `-path pattern` 指定路径样式，配合 `-prune` 参数排除指定目录
- `-perm` 按照文件权限来查找文件
- `-regex` 接正则表达式
- `-iregex` 接正则表达式，不区分大小写
- `-size n[cwbkMG]` 查找文件长度为n块的文件，带有 `cwbkMG` 时表示文件长度以字节计
- `-user` 按照文件属主来查找文件
- `-type` 查找某一类型的文件：`b`（块设备文件）;`c`（字符设备文件）;`d`（目录）;`p`（管道文件）;`l`（符号链接文件）;`f`（普通文件）;`s`（socket文件）;`D`（door）。

### Actions 模块

- `-delete` 将查找出的文件删除
- `-exec` 对配的文件执行该参数所给出的shell命令
- `-ok` 和 `-exec` 作用相同，但在执行每个明星之前，都会让用户先确定是否执行
- `-prune` 使用这一选项使find命令不在当前指定的目录中查找
- `-print` 将匹配的文件输出到标准输出（默认功能，使用中可以省略）
- `OPERATORS` find 支持逻辑运算符
- `!` 取反
- `-a` 取交集（`--and`）
- `-o` 取并集（`--or`）

## 实例

### 基础范例

- 查找指定时间内修改过的文件

查找两天内受到访问的文件，使用atime，-2代表两天内。

```bash
[root@localhost data]# find . -atime -2
.
./file1.txt
./file2.txt
./dir2
./dir3
```

查找修改时间在5天以内的文件，使用mtime

```bash
[root@localhost data]# find /data/ -mtime -5
/data/
/data/file1.txt
/data/file2.txt
/data/dir2
/data/dir3
```

- 用 `-name` 指定关键字查找

在 /var/log 目录下查找5天前以 `.log` 结尾的文件

```bash
[root@localhost data]# find /var/log -mtime +5 -name '*.log'
/var/log/openvpn.log
```

- 用 `!` 反向查找

按类型查找，查找当前目录下的所有非目录

```bash
[root@localhost data]# find . ! -type d
./file1.txt
./file2.txt
```

- 用权限来查找文件

查找 `/data` 目录下权限是755的所有文件

```bash
[root@localhost data]# find /data -perm 755
/data/
/data/dir2
/data/dir3
```

- 按大小查找文件

查找当前目录下文件大小大于1000字节的文件

```bash
[root@localhost data]# find . -size +1000c
.
./dir2
./dir3
```

- 查找文件时希望忽略某个目录

指定路径样式，配合 `-prune` 参数用于排除指定目录

```bash
[root@localhost data]# find . -path "./dir3" -prune -o -print
/data
/data/file1.txt
/data/file2.txt
/data/dir2
```

> `-path "./dir3" -prune -o -print` 是 `-path "./dir3" -a -prune -o -print` 的简写。其中-a 和 -o 类似 shell 中的 `&&` 和 `||`，当 `-paht "./dir3"` 为真时，执行 `-prune`；为假时，执行 `-print`。

- 忽略多个目录

```bash
[root@localhost data]# find . \( -path ./dir2 -o -path ./dir3 \ ) -prune -o -print
.
./file1.txt
./file2.txt
```

> 使用圆括号可以将多个表达式结合在一起，但圆括号在命令行中有特殊的含义，所有此处使用 `\` 进行转义。
>
> `\( -path ./dir2 -o -path ./dir3 \ )` 中括号与表达式之间有空格，这是语法要求。

- 使用 `-user` 和 `-nouser` 选项

```bash
[root@localhost data]# chown -R nobody:nobody test
[root@localhost data]# ll
总用量 8
drwxr-xr-x  2 nobody nobody    62 2月  22 23:35 test
drwxr-xr-x  3 root   root      16 2月  23 02:29 wnnn
[root@localhost data]# find . -user nobody
./test
./test/log2019.log
./test/ln2019
./test/link2019
./test/123
[root@localhost data]# chown -R 999 wnnn
[root@localhost data]# ll
总用量 8
drwxr-xr-x  2 nobody  nobody    62 2月  22 23:35 test
drwxr-xr-x  3 999 root      16 2月  23 02:29 wnnn
[root@localhost data]# find . -nouser
总用量 0
./wnnn/222
```

这个例子的用途是为了查找那些属主账户已经被删除的文件，使用 `-nouser` 选项，不必给出用户名。

- 使用 `-group` 和 `-nogroup` 选项

```bash
[root@localhost data]# find . -group nobody
./test
./test/log2019.log
./test/ln2019
./test/link2019
./test/123
[root@localhost data]# find . -nogroup
[root@localhost data]# chown -R 999:999 wnnn/
[root@localhost data]# ll
总用量 8
drwxr-xr-x  2 nobody  nobody      62 2月  22 23:35 test
drwxr-xr-x  3 999 999    16 2月  23 02:29 wnnn
[root@localhost data]# find . -nogroup
./wnnn/222
```

- 查找比某个文件新或旧的文件

如果希望查找更改时间比某个文件(file1)新,但比另一个文件(file2)旧的所有文件,可以使用 `-newer` 选项。它的一般形式为：`-newer file1 ! -newer file2` ，其中 `!` 是逻辑非符号，取反的意思。

```bash
[root@localhost data]# ll -h
total 0
-rw-r--r--  1 root  staff     0B  2 23 19:03 file1
-rw-r--r--  1 root  staff     0B  2 23 19:07 file2
-rw-r--r--  1 root  staff     0B  2 23 19:05 file3
[root@localhost data]# find . -newer file1
.
./file3
./file2
[root@localhost data]# find . -newer file1 ! -newer file2
.
./file3
./file2
[root@localhost data]# find . -newer file1 ! -newer file3
.
./file3
```

- 逻辑操作符的使用

```bash
[root@localhost data]# ll
总用量 16
drwxrwxr-x 2 root root 4096 2月  23 19:32 123
drwxrwxr-x 2 root root 4096 2月  23 19:32 234
drwxrwxr-x 2 root root 4096 2月  23 19:32 345
drwxrwxr-x 2 root root 4096 2月  23 19:32 456
-rw-rw-r-- 1 root root    0 2月  23 19:32 file1
-rw-rw-r-- 1 root root    0 2月  23 19:32 file2
-rw-rw-r-- 1 root root    0 2月  23 19:32 file3
-rw-rw-r-- 1 root root    0 2月  23 19:32 file4
[root@localhost data]# find . -maxdepth 1 -type d
.
./123
./345
./234
./456
[root@localhost data]# find . -maxdepth 1 -type d ! -name "."
./123
./345
./234
./456
[root@localhost data]# find . -maxdepth 1 -type d ! -name "." -o -type f
./123
./file4
./file1
./345
./234
./file3
./file2
./456
[root@localhost data]# find . -maxdepth 1 -type d ! -name "." -a -name "123"
./123
```

> `-maxdepth 1 -type d` 查找一级目录
> `!` 取反
> `-o` 逻辑或的意思
> `-a` 逻辑并的意思

- 正规则表达式 (实际工作比较少用)

由于 `-name` 参数只支持 `*` 、`?` 、`[]` 这三种通配符，因此在碰到复杂的匹配需求时，就需要用到正则表达式。

find 正则表达式语法：

```bash
find pathname -regextype "type" -regex "pattern"
```

实例：

```bash
[root@localhost data]# find / -regex "find"        #<<< 给出的正则表达式必须匹配完整的文件路径
[root@localhost data]# find / -regex ".*find"
/usr/src/kernels/4.17.13-1.el7.elrepo.x86_64/include/config/generic/find
/usr/src/kernels/4.20.11-1.el7.elrepo.x86_64/include/config/generic/find
/usr/src/kernels/3.10.0-957.5.1.el7.x86_64/include/config/generic/find
/usr/src/kernels/3.10.0-862.9.1.el7.x86_64/include/config/generic/find
/usr/src/kernels/3.10.0-957.5.1.el7.x86_64.debug/include/config/generic/find
/usr/share/bash-completion/completions/find
/usr/bin/find
/usr/bin/oldfind
```

正则表达式的类型默认为emacs, 还有posix-awk、posix-basic、pos1x­egrep和posix-extended等。

下面是posix-extended的示例代码：

```bash
[root@localhost data]# find . -regextype "posix-egrep" -name '*[0-9]*'
./3txt
./123
./file4
./file1
./345
./234
./2txt
./file3
./file2
./4txt
./1txt
./456
```

- `-exec` 选项

```bash
[root@localhost data]# find . -type f -exec  ls -l {} \;
-rw-rw-r-- 1 root root 0 2月  23 19:56 ./3txt
-rw-rw-r-- 1 root root 0 2月  23 19:32 ./file4
-rw-rw-r-- 1 root root 0 2月  23 19:32 ./file1
-rw-rw-r-- 1 root root 0 2月  23 19:56 ./2txt
-rw-rw-r-- 1 root root 0 2月  23 19:32 ./file3
-rw-rw-r-- 1 root root 0 2月  23 19:32 ./file2
-rw-rw-r-- 1 root root 0 2月  23 19:56 ./4txt
-rw-rw-r-- 1 root root 0 2月  23 19:56 ./1txt
```

> find 命令匹配到了当前目录下的所有普通文件，并在 `-exec` 选项中使用 `ls -l` 命令将它们列出
>
> `-exec` 后面跟的是 *命令* ，最后以分后 `;` 作为结束标志，考虑到各个系统中 `;` 会有不同意义，所以前面加 `\` 转义。
>
> `{}` 的作用： 指代替前面的 *命令* 查找到的内容。

- 在目录中查找更改时间在 n 天以前的文件，并删除

```bash
[root@localhost data]# find . -type f -mtime +14 -exec rm {} \;
```

- 使用 `-exec` 选项的安全模式 `-ok`

```bash
[root@localhost data]# find . -type f -mmin +5 -name "*txt" -ok rm {} \;
< rm ... ./3txt > ? n
< rm ... ./2txt > ? n
< rm ... ./4txt > ? n
< rm ... ./1txt > ? n
```

- find 结合 xargs 使用

```bash
[root@localhost data]# find . -type f
./3txt
./file4
./file1
./2txt
./file3
./file2
./4txt
./1txt
[root@localhost data]# find . -type f | xargs
./3txt ./file4 ./file1 ./2txt ./file3 ./file2 ./4txt ./1txt
[root@localhost data]# find . -type f | xargs ls -l
-rw-rw-r-- 1 root root 0 2月  23 19:56 ./1txt
-rw-rw-r-- 1 root root 0 2月  23 19:56 ./2txt
-rw-rw-r-- 1 root root 0 2月  23 19:56 ./3txt
-rw-rw-r-- 1 root root 0 2月  23 19:56 ./4txt
-rw-rw-r-- 1 root root 0 2月  23 19:32 ./file1
-rw-rw-r-- 1 root root 0 2月  23 19:32 ./file2
-rw-rw-r-- 1 root root 0 2月  23 19:32 ./file3
-rw-rw-r-- 1 root root 0 2月  23 19:32 ./file4
[root@localhost data]$ find . -type f |xargs -i mv {} 123/
[root@localhost data]$ ls
123  234  345  456
[root@localhost data]$ ls 123
1txt  2txt  3txt  4txt  file1  file2  file3  file4
[root@localhost data]$ find . -type f |xargs -p rm -f         #<<< xargs 的 `-p` 选项会提示让你确认是否执行后面的操作
rm -f ./123/3txt ./123/file4 ./123/file1 ./123/2txt ./123/file3 ./123/file2 ./123/4txt ./123/1txt ?...n
```

- 进入 /root/data 目录删除以txt结尾的文件

```bash
[root@localhost data]# ls
123  1txt  234  2txt  345  3txt  456  4txt  file1  file2  file3  file4
[root@localhost data]# find /root/data/ -type f |xargs rm -f
[root@localhost data]# ls
123  234  345  456
```

- 结合 sed 命令

把/data 目录及其子目录下的所有以扩展名为 `txt` 结尾的文件中，把包含 `2txt` 的字符串全部替换为 `root_word`。

`-exec` 方法

```bash
[root@localhost data]# cat 1txt
txttxtxtxtttttt
[root@localhost data]# find /data -name "*txt" -exec sed -i 's/2txt/root_word/g' {} \;
[root@localhost data]# cat 1txt
wordwordxwordttttt
```

`find` + `xargs` 方法

```bash
[root@localhost data]# cat 1txt
txttxtxtxtttttt
[root@localhost data]# find ./ -name "*txt" |xargs sed -i 's/txt/word/g'
[root@localhost data]# cat 1txt
wordwordxwordttttt
```

高效处理方法 （一个命令语句中若有反引号，则优先执行反引号中的命令）

```bash
sed -i 's/2txt/root_word/g' `find /data -name "*txt"`
```

> sed 的 `-i` 选项用于直接操作文件

- 将 `/etc` 下所有普通文件打包成压缩文件

- 反引号法

```bash
tar zcvf file.tar.gz `find /etc -type f`
```

- xargs 方法

```bash
find /etc -type f |xargs tar czvf file.tar.gz
```

- 删除一个目录下的所有文件，但保留一个指定文件

删除所有文件，保留文件 file1

- find + xargs 命令方法

```bash
[root@localhost data]# ls
123 1txt 1.txt 234 2txt 2.txt 345 3txt 3.txt 456 4txt file1 file2 file3 file4 file.tar.gz
[root@localhost data]# find . -type f ! -name "file1" |xargs rm
[root@localhost data]# ls
123 234 345 456 file1
```

- 使用 exec 选项命令方法 (`-ok` 是 `-exec` 选项的安全模式)

```bash
[root@localhost data]# find . -type f ! -name "file1" -exec rm -rf {} \;
```

- 已知 apache 服务的访问日志按天记录在服务器本地目录 /data/logs 下，由于磁盘空间紧张，现要求只保留最近7天的访问日志（仅从日志着手解决）

测试日志文件生成脚本：

```bash
[root@localhost log]# cat test_log.sh
for n in `seq 14`
do
    date -s "2019/01/$n"
    touch access_www_`(date+%F)`.log
done
date -s "2019/01/15"
```

生成测试日志文件：

```bash
sh test_log.sh
[root@localhost log]# sh test_log.sh
2019年 01月 01日 星期二 00:00:00 CST
2019年 01月 02日 星期三 00:00:00 CST
2019年 01月 03日 星期四 00:00:00 CST
2019年 01月 04日 星期五 00:00:00 CST
2019年 01月 05日 星期六 00:00:00 CST
2019年 01月 06日 星期日 00:00:00 CST
2019年 01月 07日 星期一 00:00:00 CST
2019年 01月 08日 星期二 00:00:00 CST
2019年 01月 09日 星期三 00:00:00 CST
2019年 01月 10日 星期四 00:00:00 CST
2019年 01月 11日 星期五 00:00:00 CST
2019年 01月 12日 星期六 00:00:00 CST
2019年 01月 13日 星期日 00:00:00 CST
2019年 01月 14日 星期一 00:00:00 CST
2019年 01月 15日 星期二 00:00:00 CST
[root@localhost log]# ls
access_www_2019-01-01.log  access_www_2019-01-07.log  access_www_2019-01-13.log
access_www_2019-01-02.log  access_www_2019-01-08.log  access_www_2019-01-14.log
access_www_2019-01-03.log  access_www_2019-01-09.log  access_www_2019-01-15.log
access_www_2019-01-04.log  access_www_2019-01-10.log  test_log.sh
access_www_2019-01-05.log  access_www_2019-01-11.log
access_www_2019-01-06.log  access_www_2019-01-12.log
```

解决上述问题的方法有如下三种：

```bash
find . -type f -name "access*.log" -mtime +7|xargs rm -rf
find . -type f -name "access*.log" -mtime +7 -exec rm -rf {} \;
find . -type f -name "access*.log" -mtime +7 -delete
```

第四种方法用 Apache 服务的 cronolog 软件轮询日志

```bash
CustomLog “/usr/loacl/sbin/cronolog /data/logs /access_www_%w.log" combined
```

总共生成7天日志1-7，下周又覆盖1-7的日志。

- 将找到的文件移动到指定位置的集中方法

```bash
find . -name "*.txt" |xargs -i mv {} dir2/
find . -name "*.txt" |xargs mv -t dir2/
mv `find . -name "*.txt"` dir2/
```

> xargs 的 `-i` 参数可以使 `{}` 代替 find 找到的内容
>
> mv 的 `-t` 参数可以颠倒 *源* 和 *目标* 
