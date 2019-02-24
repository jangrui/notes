# touch

touch 命令用于修改文件或者目录的时间属性，包括存取时间和更改时间。若文件不存在，系统会建立一个新的文件。

ls -l 可以显示档案的时间记录。

## 语法

```bash
touch [-acfm][-d<日期时间>][-r<参考文件或目录>] [-t<日期时间>][--help][--version][文件或目录...]
```

## 参数时间

- `-a` 改变档案的读取时间记录
- `-m` 改变档案的修改时间记录
- `-c` 加入目的档案不存在，不会建立新的档案。 与 `--no-create` 的效果一样
- `-f` 不适用，是为了与其他unix系统的相容性而保留
- `-r` 使用参考档的时间记录，与 `--file` 的效果一样
- `-d` 设定时间与日期，可以使用各种不同的格式
- `-t` 设定档案的时间记录，格式与 `data` 指令相同
- `--no-create` 不会建立新档案
- `--help` 列出指令格式
- `--version` 列出版本讯息

## 实例

修改文件 testfile 的时间属性为当前系统时间

首先，使用 ls 命令查看 testfile 文件的属性

```bash
ls -l testfile
-rw-r--r-- 1 hdd hdd 55 2019-02-22 16:09 testfile 
```

修改文件属性后，再次查看文件的时间属性

```bash
touch testfile
ls -l testfile
-rw-r--r-- 1 hdd hdd 55 2011-08-22 16:12 testfile 
```

如果指定的文件不存在，则创建一个新的空白文件。 例如，在当前目录下创建哪一个空白文件file

```bash
touch file
```