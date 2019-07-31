# basename

basename 命令用于显示去除路径和文件后缀部分的文件名或目录。

## 语法

```bash
basename [文件或目录] [后缀]
```

> `[后缀]` 是可选参数，指定要去除的文件后缀字符串。

## 实例

显示文件或目录

```bash
[root@localhost data]# ls log/
access_www_2019-01-01.log  access_www_2019-01-07.log  access_www_2019-01-13.log
access_www_2019-01-02.log  access_www_2019-01-08.log  access_www_2019-01-14.log
access_www_2019-01-03.log  access_www_2019-01-09.log  access_www_2019-01-15.log
access_www_2019-01-04.log  access_www_2019-01-10.log  test_log.sh
access_www_2019-01-05.log  access_www_2019-01-11.log
access_www_2019-01-06.log  access_www_2019-01-12.log
[root@localhost data]# basename log/access_www_2019-01-01.log
access_www_2019-01-01.log
[root@localhost data]# basename log/access_www_2019-01-01.log .log
access_www_2019-01-01
[root@localhost data]# basename log
log
```
