# ln

ln 命令用于为某一个文件或目录在另外一个位置建立一个同步的链接。

当我们需要在不同的目录，用到相同的文件时，不需要再每个需要的目录都放一个相同的文件，只要在某个固定的目录，放入该文件，然后在其他的目录下用ln指令链接（link）它就可以，不必重复的占用磁盘空间。

## 语法

```bash
ln [参数] [源文件或目录] [目标文件或目录]
ln [-bdfinsvF] [-S backup-suffix] [-V {numbered,existing,simple}][--help][--version][--]
```

## 命令功能

链接可分为两种：

硬链接（hard link）：

- 以文件的副本形式存在
- 可以有多个名称
- 不允许给目录创建硬链接
- 只有在一个文件系统中才能创建

软连接（symbolic link）：

- 以路径的形式存在，类似于Windows中的快捷方式
- 可以跨文件系统
- 可以对一个不存在的文件进行连接
- 可以对目录进行连接

不论是硬链接或软连接都不会将原本的档案复制一份，只会占用非常少量的磁盘空间。

## 命令参数

### 必要参数

- `-b` 删除，覆盖以前建立的链接
- `-d` 允许超级用户制作目录的硬链接
- `-f` 强制执行
- `-i` 交互模式，文件存在则提示用户是否覆盖
- `-n` 把符号链接视为一般目录
- `-s` 软链接（符号链接）
- `-v` 显示详细的处理过程

### 选择参数

- `-S` `-S<字尾备份字符串>` 或 `--suffix=<字尾备份字符串>`
- `-V` `-V<备份方式>` 或 `--version-control=<备份方式>` 
- `--help` 显示帮助信息
- `--version` 显示版本信息

## 实例

给文件创建软链接，为log2019.log文件创建软连接link2019，如果log2019.log丢失，link2019将失效：

```bash
ln -s log2019.log link2019
```

输出：

```bash
[root@localhost test]# ll
总用量 0
-rw-r--r-- 1 root root 0 2月  22 23:15 log2019.log
[root@localhost test]# ln -s log2019.log link2019
[root@localhost test]# ll
总用量 0
lrwxrwxrwx 1 root root 11 2月  22 23:15 link2019 -> log2019.log
-rw-r--r-- 1 root root  0 2月  22 23:15 log2019.log
```

给文件创建硬链接，为log2019.log创建ln2019，log2019.log与ln2019的各项属性相同

```bash
ln log2019.log ln2019
```

输出：

```bash
[root@localhost test]# ll
总用量 0
-rw-r--r-- 1 root root 0 2月  22 23:15 log2019.log
[root@localhost test]# ln log2019.log ln2019
[root@localhost test]# ll
总用量 0
-rw-r--r-- 2 root root 0 2月  22 23:15 ln2019
-rw-r--r-- 2 root root 0 2月  22 23:15 log2019.log

```