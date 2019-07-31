# xargs

xargs 是给命令传递参数的一个过滤器，也是组合多个命令的一个工具。

xargs 可以将管道或标准输入（stdin）数据转换成命令行参数，也能够从文件的输出中读取数据。

xargs 也可以将单行或多行文本输入转换为其他格式，例如多行变单行，单行变多行。

xargs 默认的命令是 echo，这意味着通过管道传递给 xargs 的输入将会包含换行和空白，不过通过 xargs 的处理，换行和空白将被空格取代。

```bash
find /sbin -user root |xargs
```

xargs 一般是和管道一起使用。

## 命令格式

```bash
somecommand |xargs -item  command
```

## 参数

- `-a`        file 从文件中读入作为 sdtin（标准输入）
- `-d`        自定义分隔符。
- `-i`        以 `{}` 代替前面的结果。
- `-I`        制定一个符号代替前面的结果，而不用 `-i` 参数默认的 `{}`。
- `-lnum`     同 -L。(-l与数字间无空格）
- `-L num`    从标准输入一次读取 num 行。
- `-n`        指定每行的最大参数量 n ，可以将标准输入的文本划分为多行，每行 n 个参数，默认空格分割。
- `-p`        提示让用户确认是否执行后面操作，y 执行，n 不执行。
- `-t`        表示先打印命令，然后再执行。
- `-0`        用 null 代替空格作为分割符，配合 find 命令的 `-print0` 选项的输出使用。

## 实例

xargs 用作替换工具，读取输入数据重新格式化后输出。

定义一个测试文件，内有多行文本数据：

```bash
# cat test.txt
a b c d e f g
h i j k l m n
o p q
r s t
u v w x y z
```

- 多行输入单行输出：

```bash
# cat test.txt | xargs
a b c d e f g h i j k l m n o p q r s t u v w x y z
```

- -n 选项多行输出：

```bash
# cat test.txt | xargs -n3
a b c
d e f
g h i
j k l
m n o
p q r
s t u
v w x
y z
```

- -d 选项可以自定义一个定界符：

```bash
# echo "nameXnameXnameXname" | xargs -dX
name name name name
```

- 结合 -n 选项使用：

```bash
# echo "nameXnameXnameXname" | xargs -dX -n2
name name
name name
```

- 参数 -I 可以指定一个替换的字符

读取 stdin，将格式化后的参数传递给命令

xargs 的一个选项 -I，使用 -I 指定一个替换字符串 []，这个字符串在 xargs 扩展时会被替换掉，当 -I 与 xargs 结合使用，每一个参数命令都会被执行一次：

```bash
[root@localhost data]# ls
123   234   345   456   arg.txt  file2  file4  sk.sh
1txt  2txt  3txt  4txt  file1    file3  log
[root@localhost data]# ls |xargs -I [] echo hello-[]-word
hello-123-word
hello-1txt-word
hello-234-word
hello-2txt-word
hello-345-word
hello-3txt-word
hello-456-word
hello-4txt-word
hello-arg.txt-word
hello-file1-word
hello-file2-word
hello-file3-word
hello-file4-word
hello-log-word
hello-sk.sh-word
[root@localhost data]#
```

- 复制所有图片文件到 /data/images 目录下：

```bash
ls *.jpg | xargs -n1 -I cp {} /data/images
```

xargs 结合 find 使用

用 rm 删除太多的文件时候，可能得到一个错误信息：/bin/rm Argument list too long. 用 xargs 去避免这个问题：

```bash
find . -type f -name "*.log" -print0 | xargs -0 rm -f
```

xargs -0 将 \0 作为定界符。

统计一个源代码目录中所有 php 文件的行数：

```bash
find . -type f -name "*.php" -print0 | xargs -0 wc -l
```

查找所有的 jpg 文件，并且压缩它们：

```bash
find . -type f -name "*.jpg" -print | xargs tar -czvf images.tar.gz
```

xargs 其他应用

假如你有一个文件包含了很多你希望下载的 URL，你能够使用 xargs下载所有链接：

```bash
# cat url-list.txt | xargs wget -c
```
