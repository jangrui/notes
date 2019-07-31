# file

file 命令用于显示文件的类型

## 语法

```bash
file [选项] [文件或目录]
```

## 参数

- `-b` 输出信息使用精简格式，不输出文件名

## 实例

```bash
[root@localhost data]# file -b 1txt
ASCII text
[root@localhost data]# file *
123:   directory
1txt:  ASCII text
234:   directory
2txt:  empty
345:   directory
3txt:  empty
456:   directory
4txt:  empty
a.jpg: empty
b.jpg: empty
c.jpg: empty
d.jpg: empty
file1: empty
file2: empty
file3: empty
file4: empty
log:   directory
```
