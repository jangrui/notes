# rm

rm 命令用于删除文件呢或者目录

## 语法

```bash
rm [options] name ...
```

## 参数说明

- `-i` 删除前逐一询问确认
- `-f` 不有任何提示
- `-r` 将目录及以下内容逐一删除

## 实例

> 删除文件可以直接使用 `rm` 命令，若删除目录则必须配合选项 `-r`

```bash
# rm test.txt
rm: 是否删除 一般文件"test.txt"? y
# rm homework/
rm: 无法删除目录 "homework": 是一个目录
# rm -r homework/
rm: 是否删除 目录 "homework"? y
```

强制删除当前目录下的所有文件及目录

```bash
rm -rf *
```