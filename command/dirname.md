# dirname

dirname 命令用于显示文件或目录的路径。

## 语法

```bash
dirname [文件或目录]
```

## 实例

```bash
[root@localhost log]# pwd
/root/data/log
[root@localhost log]# dirname log/access_www_2019-01-01.log
log
[root@localhost log]# dirname access_www_2019-01-01.log
.
[root@localhost log]# dirname /etc/cron.d/atop
/etc/cron.d
```
