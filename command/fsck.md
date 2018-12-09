# fsck用于文件系统检查修复

Linux fsck命令用于 检查与修复 Linux 档案系统，可以同时检查一个或多个 Linux 档案系统。

## 语法

```bash
fsck [-sACVRP] [-t fstype] [--] [fsck-options] filesys [...]
```

### 参数 ：

> filesys ： device 名称(eg./dev/sda1)，mount 点 (eg. / 或 /usr)
> -t : 给定档案系统的型式，若在 /etc/fstab 中已有定义或 kernel 本身已支援的则不需加上此参数
> -s : 依序一个一个地执行 fsck 的指令来检查
> -A : 对/etc/fstab 中所有列出来的 partition 做检查
> -C : 显示完整的检查进度
> -d : 列印 e2fsck 的 debug 结果
> -p : 同时有 -A 条件时，同时有多个 fsck 的检查一起执行
> -R : 同时有 -A 条件时，省略 / 不检查
> -V : 详细显示模式
> -a : 如果检查有错则自动修复
> -r : 如果检查有错则由使用者回答是否修复

## 实例

检查 msdos 档案系统的 /dev/hda5 是否正常，如果有异常便自动修复 :

```bash
fsck -t msdos -a /dev/hda5
```

> 注意 此指令可与 /etc/fstab 相互参考操作来加以了解。