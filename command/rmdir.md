# rmdir

rmdir 命令用于删除空目录。

## 语法

```bash
rmdir [-p] dirName
```

## 参数

- `-p` 当子目录被删除后使它成为空目录的话，则顺便一并删除

## 实例

将工作目录下，名为AAA的子目录删除

```bash
rmdir AAA
```

在工作目录下的BBB目录中，删除名为AAA的子目录，若AAA删除后，BBB目录成为空目录，则BBB也删除。

```bash
rmdir -P BBB/AAA
```
