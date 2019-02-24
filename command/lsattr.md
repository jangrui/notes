# lsattr

lsattr 命令用于查看文件的扩展属性。

## 语法

```bash
lsattr [选项] [文件或目录]
```

## 参数

- `-R` 递归查看目录的扩展属性
- `-a` 显示所有文件包括隐藏文件的扩展属性
- `-d` 显示目录的扩展属性

## 实例

1. 查看文件的扩展属性

```bash
[root@localhost data]# lsattr -R log/
-------------e-- log/access_www_2019-01-08.log
-------------e-- log/access_www_2019-01-13.log
-------------e-- log/access_www_2019-01-05.log
-------------e-- log/access_www_2019-01-02.log
-------------e-- log/access_www_2019-01-14.log
-------------e-- log/access_www_2019-01-10.log
-------------e-- log/access_www_2019-01-12.log
-------------e-- log/access_www_2019-01-09.log
-------------e-- log/access_www_2019-01-07.log
-------------e-- log/access_www_2019-01-03.log
-------------e-- log/access_www_2019-01-04.log
-----a-------e-- log/test_log.sh
-------------e-- log/access_www_2019-01-15.log
-------------e-- log/access_www_2019-01-06.log
-------------e-- log/access_www_2019-01-11.log
-------------e-- log/access_www_2019-01-01.log
```

2. 查看目录的扩展属性

```bash
[root@localhost data]# lsattr -d log/
-------------e-- log/
[root@localhost data]# chattr +i log/
[root@localhost data]# lsattr -d log/
----i--------e-- log/
```