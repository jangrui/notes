# chgrp

chgrp 命令用于改变文件的用户组，功能已被 chown 取代了。

## 语法

```bash
chgrp [选项] [用户组] [文件或目录]
```

## 参数

- `-R` 递归更改目录的用户组

## 实例

```bash
[root@localhost data]# chgrp -R root log
[root@localhost data]# ll -a log
总用量 3
d--------- 2 jangrui root 4096 1月  15 00:00 .
drwxrwxr-x 7 root    root 4096 2月  24 22:21 ..
---------- 1 jangrui root    0 1月   1 00:00 access_www_2019-01-01.log
---------- 1 jangrui root    0 1月   2 00:00 access_www_2019-01-02.log
---------- 1 jangrui root    0 1月   3 00:00 access_www_2019-01-03.log
```