# chmod

chmod 命令用于改变文件或目录的权限。

但是只有文件的属主和超级用户 root 才能够执行。

## 语法

```bash
chmod [选项] [模式] [文件或目录]
```

## 参数

- `-R` 递归操作指定目录以及其子目录下的所有文件

## 权限说明

|权限位	|全称	|含义		|对应数字		|
|-		|-		|-			|-			|
|`r`	|read	|可读权限		|4			|
|`w`	|write	|可写		|2			|
|`x`	|excute	|可执行		|1			|
|`-`	|		|没有任何权限	|0			|
|备注	|特殊权限	|t、s、x		|T、S、X		|
|用户类型|用户：u |用户组：g	|其他：o		|
|操作符号|加入：+	|减去：-		|设置：=		|

## 实例

1. 字母和操作费表达式

```bash
[root@localhost data]# chmod a=rwx md5.log 
[root@localhost data]# ll md5.log 
-rwxrwxrwx 1 root root 39 2月  24 22:21 md5.log
[root@localhost data]# chmod a=rw md5.log 
[root@localhost data]# ll md5.log 
-rw-rw-rw- 1 root root 39 2月  24 22:21 md5.log
[root@localhost data]# chmod u+x md5.log 
[root@localhost data]# ll md5.log 
-rwxrw-rw- 1 root root 39 2月  24 22:21 md5.log
[root@localhost data]# chmod g+x md5.log 
[root@localhost data]# ll md5.log 
-rwxrwxrw- 1 root root 39 2月  24 22:21 md5.log
[root@localhost data]# chmod a-x md5.log 
[root@localhost data]# ll md5.log 
-rw-rw-rw- 1 root root 39 2月  24 22:21 md5.log
[root@localhost data]# chmod u+x,g-w md5.log 
[root@localhost data]# ll md5.log 
-rwxr--rw- 1 root root 39 2月  24 22:21 md5.log
```

2. 数字权限

```bash
[root@localhost data]# chmod 000 md5.log 
[root@localhost data]# ll md5.log 
---------- 1 root root 39 2月  24 22:21 md5.log
[root@localhost data]# chmod 755 md5.log 
[root@localhost data]# ll md5.log 
-rwxr-xr-x 1 root root 39 2月  24 22:21 md5.log
[root@localhost data]# chmod -111 md5.log 
[root@localhost data]# ll md5.log 
-rw-r--r-- 1 root root 39 2月  24 22:21 md5.log
[root@localhost data]# chmod +111 md5.log 
[root@localhost data]# ll md5.log 
-rwxr-xr-x 1 root root 39 2月  24 22:21 md5.log
```

3. 递归处理

```bash
[root@localhost data]# chmod -R 000 log/
[root@localhost data]# ll -a log/
总用量 3
d--------- 2 jangrui jangrui 4096 1月  15 00:00 .
drwxrwxr-x 7 root    root    4096 2月  24 22:21 ..
---------- 1 jangrui jangrui    0 1月   1 00:00 access_www_2019-01-01.log
---------- 1 jangrui jangrui    0 1月   2 00:00 access_www_2019-01-02.log
---------- 1 jangrui jangrui    0 1月   3 00:00 access_www_2019-01-03.log
```