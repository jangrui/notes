# ls

ls 用于显示指定工作目录下的内容。

## 语法

```bash
ls [-alrtAFR] [name...]
```

## 参数

-a 显示所有文件及目录了（包括隐藏文件）
-l 出文件名称外，将文件类型、权限、拥有者、文件大小、时间等资讯详细列出
-r 将文件呢以相反次序显示（原定依英文字母次序）
-t 将文件依建立时间先后次序列出
-A 同 `-a` ,但不列出 `.`（当前目录）及 `..`（父目录）
-F 在列出的文件名称后加对应文件类型符号，例如可执行文件则加 `*`，目录则加 `/`
-R 若目录下有文件，则以下文之文件皆依次列出

## 实例

列出根目录下的所有文件

```bash
# ls /
bin               dev   lib         media  net   root     srv  upload  www
boot              etc   lib64       misc   opt   sbin     sys  usr
home  lost+found  mnt    proc  selinux  tmp  var
```

列出当前工作目录下所有名称以s开头的文件，越新的排越后面

```bash
ls -ltr s*
```

将/bin目录一下所有目录及文件详细资料列出

```bash
ls -lR /bin
```

列出当前工作目录下所有文件及目录；目录于名称后加 `/`，可执行文件于名称后加 `*`:

```bash
ls -AF
```