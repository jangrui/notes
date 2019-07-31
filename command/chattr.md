# chattr

chattr （change attribute）命令用于改变文件的扩展属性。

与 chmod 命令相比，chmod 只是改变文件的读、写、执行权限，更底层的属性控制是由 chattr 来改变的。

## 语法

```bash
chattr [选项] [模式] [文件或目录]
```

## 参数

- `-R` 递归更改目录属性
- `-V` 显示命令执行过程

mode

- `+` 增加参数
- `-` 移除参数
- `=` 更新为指定参数
- `A` 高速系统不要修改这个文件的最后访问时间
- `a` *只能向文件中添加数据，而不能删除，多用于服务器日志文件安全*
- `i` *设定文件不能被删除、改名、写入或新增内容*

## 实例

- 设置只能往文件里追加内容，但不能删除文件

```bash
[root@localhost log]# lsattr test_log.sh
-------------e-- test_log.sh
[root@localhost log]# chattr +a test_log.sh
[root@localhost log]# lsattr test_log.sh
-----a-------e-- test_log.sh
[root@localhost log]# rm -rf test_log.sh
rm: 无法删除"test_log.sh": 不允许的操作
[root@localhost log]# echo hello >> test_log.sh
[root@localhost log]# echo word > test_log.sh
-bash: test_log.sh: 不允许的操作
```

- 给文件加锁，使其只读

```bash
[root@localhost data]# chattr +i 1txt
[root@localhost data]# lsattr 1txt
----i--------e-- 1txt
[root@localhost data]# rm 1txt
rm：是否删除普通文件 "1txt"？y
rm: 无法删除"1txt": 不允许的操作
[root@localhost data]# echo hello > 1txt
-bash: 1txt: 权限不够
[root@localhost data]# echo hello >> 1txt
-bash: 1txt: 权限不够
[root@localhost data]# cat 1txt
wordwordxwordttttt
```
