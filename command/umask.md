# umask

umask 命令通过八进制的数值来定义用户创建文件或目录的默认权限。

## 语法

```bash
umask [选项] [模式]
```

## 参数

- `-p` 输出的权限掩码可直接作为命令来执行
- `-S` 以字符方式输出权限掩码

## 通过 umask 计算文件目录权限

1. 文件权限计算

默认创建文件最大的权限是 666（-rw-rw-rw-），默认创建的文件没有可执行权限 `x` 位。

- 假设 umask 值为：022 （所有位为偶数）

```bash
666		#<<< 文件的起始权限
022-	#<<< umask 设置的值
644		#<<< 最后得到权限值
```

- 假设 umask 值为：045 （其他用户组位为奇数）

```bash
666		#<<< 文件的起始权限
045-	#<<< 假设 umask 设置的值
621		#<<< 计算出来的权限，由于 umask 的最后一位数字是奇数5，所以在其他用户组位在加1.
001+	#<<< umask 对应得奇数位加1
622		#<<< 最后所得权限
```

2. 目录权限计算（没有奇偶之分）

创建目录默认最大权限777（-rwx-rwx-rwx）,默认创建的目录属主是有 `x` 权限的，允许用户进入。

对于目录来说，umask 的设置实在嘉定文件拥有八位制777权限上进行，目录八进制权限777减去 umask 的掩码数值。

```bash
777		#<<< 目录的起始权限值
022-	#<<< umask 设置的值
755		#<<< 最后得到的权限值
```

## 实例

1. 系统默认的 umask 值

```bash
[jangrui@localhost ~]$ umask 	#<<< 普通用户的 umask 默认值
0002
[root@localhost ~]# umask 		#<<< root 的 umask 默认值
0022
```

2. `-p` 与 `-S` 参数

```bash
[root@localhost ~]# umask -p
umask 0022
[root@localhost ~]# umask -S
u=rwx,g=rx,o=rx
```

3. 修改 umask 值对文件权限的影响

```bash
[root@localhost data]# umask 
0022
[root@localhost data]# touch hello
[root@localhost data]# ll hello 
-rw-r--r-- 1 root root 0 2月  24 23:22 hello
[root@localhost data]# umask 044
[root@localhost data]# touch word
[root@localhost data]# ll word 
-rw--w--w- 1 root root 0 2月  24 23:23 word
```

4. 修改 umask 值对目录权限的影响

```bash
[root@localhost data]# umask
0044
[root@localhost data]# mkdir new1
[root@localhost data]# ll -d new1/
drwx-wx-wx 2 root root 4096 2月  24 23:25 new1/
[root@localhost data]# umask 022
[root@localhost data]# mkdir new2
[root@localhost data]# ll -d new2
drwxr-xr-x 2 root root 4096 2月  24 23:25 new2
```

5. 让 umask 永久生效

修改 `/etc/profile` 或 `/etc/bashrc` 实现对 umask 的永久修改。

但是几乎没有什么需求必须要修改 umask ，默认的 umask 是系统安全的临界点，是最合适的。

```bash
echo "umask 033" >> /etc/profile
```