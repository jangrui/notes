# rename

rename 命令用于通过字符串替换的方式批量修改文件名。

## 语法

```bash
rename old new file
```

## 参数

- `old` 代表需要替换或者需要处理的字符（一般是文件名的一部分，也包括扩展名）。
- `new` 把前面的 `from` 代表的内容替换为 `to` 代表的内容。
- `file` 待处理的文件，可以用 `*` 通配所有的文件。

## 实例

- 批量修改文件名

```bash
[root@localhost data]# find . -maxdepth 1 -type f |xargs
./file4 ./file1 ./2word ./file3 ./1word ./file2 ./arg.word ./sk.sh ./4word ./3word
[root@localhost data]# find . -maxdepth 1 -type f |xargs rename "word" "txt"
[root@localhost data]# find . -maxdepth 1 -type f |xargs
./3txt ./file4 ./file1 ./2txt ./file3 ./arg.txt ./file2 ./4txt ./1txt ./sk.sh
```

- 批量修改扩展名

```bash
[root@localhost data]# ls
123   234   345   456   a.png  c.png  file1  file3  log
1txt  2txt  3txt  4txt  b.png  d.png  file2  file4
[root@localhost data]# rename .png .jpg *.png
[root@localhost data]# ls
123   234   345   456   a.jpg  c.jpg  file1  file3  log
1txt  2txt  3txt  4txt  b.jpg  d.jpg  file2  file4
```
