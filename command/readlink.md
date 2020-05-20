# readlink

readlink 命令用于查看符号链接文件的真实内容。

使用cat命令查看软连接文件时，会发现只能看到源文件的内容，看不到软连接文件的真实内容，因此需要使用readlink命令。

## 语法

```bash
readlink [选项] [文件]
```

## 参数说明

- `-f` 一直跟随符号链接，知道非符号链接的文件位置，但要保证最后必须存在一个非符号链接的文件

## 实例

查看符号链接文件的内容

```bash
[root@dvm test]# ll -h
总用量 8.0K
lrwxrwxrwx 1 root root  8 2月  22 23:35 123 -> link2019
lrwxrwxrwx 1 root root 11 2月  22 23:30 link2019 -> log2019.log
-rw-r--r-- 2 root root 12 2月  22 23:30 ln2019
-rw-r--r-- 2 root root 12 2月  22 23:30 log2019.log
[root@dvm test]# readlink 123
link2019
[root@dvm test]# cat 123
12312312323
[root@dvm test]# cat link2019
12312312323
```

查看链接文件的最后一个非符号链接文件

```bash
[root@dvm test]# readlink -f 123
/root/test/log2019.log
[root@dvm test]# cat log2019.log
12312312323
[root@dvm test]#
```
