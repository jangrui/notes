# chown

chown 命令用于改变文件或目录的用户和用户组。

## 语法

```bash
chown [选项] [用户:用户组] [文件或目录]
```

常用格式：

```bash
chown 用户 文件或目录		#<<< 仅授权用户
chown :组 文件或目录		#<<< 仅授权组
chown 用户:组 文件或目录	#<<< 授权用户和组
```

> 其中 `:` 可用 `.` 来代替

## 参数

- `-R` 递归更改目录的用户和用户组

## 实例

1. 更改文件所属的用户属性

```bash
[root@localhost data]# ll 1txt 
-rw-r--r-- 1 root root 6 2月  24 22:22 1txt
[root@localhost data]# chown xxx 1txt 
chown: 无效的用户: "xxx"
[root@localhost data]# chown jangrui 1txt 
[root@localhost data]# ll 1txt
-rw-r--r-- 1 jangrui root 6 2月  24 22:22 1txt
```

2. 更改文件所属组

```bash
[root@localhost data]# chown .jangrui 1txt 
[root@localhost data]# ll 1txt 
-rw-r--r-- 1 jangrui jangrui 6 2月  24 22:22 1txt
```

3. 同事更改文件所属用户和组属性

```bash
[root@localhost data]# ll 2txt 
-rw-r--r-- 1 root root 0 2月  24 17:02 2txt
[root@localhost data]# chown jangrui.jangrui 2txt 
[root@localhost data]# ll 2txt 
-rw-r--r-- 1 jangrui jangrui 0 2月  24 17:02 2txt
```

4. 递归更改目录及目录下的所有子目录及文件的用户和组属性

```bash
[root@localhost data]# ll -a log/
总用量 3
drwxr-xr-x 2 root root 4096 1月  15 00:00 .
drwxrwxr-x 7 root root 4096 2月  24 22:21 ..
-rw-r--r-- 1 root root    0 1月   1 00:00 access_www_2019-01-01.log
-rw-r--r-- 1 root root    0 1月   2 00:00 access_www_2019-01-02.log
-rw-r--r-- 1 root root    0 1月   3 00:00 access_www_2019-01-03.log
[root@localhost data]# chown -R jangrui.jangrui log
[root@localhost data]# ll -a log/
总用量 3
drwxr-xr-x 2 jangrui jangrui 4096 1月  15 00:00 .
drwxrwxr-x 7 root    root    4096 2月  24 22:21 ..
-rw-r--r-- 1 jangrui jangrui    0 1月   1 00:00 access_www_2019-01-01.log
-rw-r--r-- 1 jangrui jangrui    0 1月   2 00:00 access_www_2019-01-02.log
-rw-r--r-- 1 jangrui jangrui    0 1月   3 00:00 access_www_2019-01-03.log
```