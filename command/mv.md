# mv

mv 命令用于文件或目录改名，或将文件与目录移动到其他位置。

## 语法

```bash
mv [options] source dest
mv [options] source... directory
```

## 参数说明

- `-i` 若指定目录已有同名文件，则先询问是否覆盖
- `-f` 覆盖已有的目标文件时不给任何提示，强制执行

## 实例

将文件 aaa 更名为 bbb

```bash
mv aaa bbb
```

将info目录放入logs目录中。

> 注： 若果logs目录不存在，则该命令将info改名为logs。

```bash
mv info logs
```
将/usr/student下的所有文件和目录移动到当前目录下

```bash
mv /usr/student/* .
```