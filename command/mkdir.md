# mkdir

mkdir 命令用于建立目录。

## 语法

```bash
mkdir [-p] 目录名称
```

## 参数说明

- `-p` 确保目录名称存在，不存在就建一个

## 实例

在工作目录下，建立一个名为AAA的子目录

```bash
mkdir AAA
```

在工作目录下的BBB目录中，建立一个名为Test的子目录。若BBB目录原本不存在，则建立一个。

> 注： 本利若不加 `-p`,原本BBB目录不存在，则产生错误。

```bash
mkdir -p BBB/Test
```