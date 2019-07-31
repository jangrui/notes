# md5sum

md5sum 命令用于计算和校验文件的 MD5 值。

MD5 (Message-Digest Algorithm 5)（信息-摘要算法 5），他是一种不可逆的加密算法。

## 语法

```bash
md5sum [选项] [文件]
```

## 参数

- `-b`        二进制模式读取文件
- `-c`        从指定文件中读取 MD5 校验值，并进行校验*
- `-t`        从文本模式读取文件，这是默认模式
- `--quiet`   校验文件使用的参数，验证不通过不输出 OK
- `--status`  校验文件使用的参数，不输出任何信息，可以通过命令的返回值来判断

## 实例

- 生成一个文件 MD5 值

```bash
[root@localhost data]# md5sum 1txt
43450f66850aece964616f4add84fb15  1txt
```

- 校验文件是否发生改变

```bash
[root@localhost data]# md5sum 1txt > md5.log
[root@localhost data]# cat md5.log
43450f66850aece964616f4add84fb15  1txt
[root@localhost data]# md5sum -c md5.log
1txt: 确定
[root@localhost data]# echo hello > 1txt
[root@localhost data]# md5sum -c md5.log
1txt: 失败
md5sum: 警告：1 个校验和不匹配
[root@localhost data]# md5sum --status -c md5.log
[root@localhost data]# echo $?
1
```
