# e2fsck 用于检查 ext2/3/4 文件系统

e2fsck命令用于检查使用 Linux ext2 档案系统的 partition 是否正常工作。

## 语法

```bash
e2fsck [-pacnydfvFV] [-b superblock] [-B blocksize] [-l|-L bad_blocks_file] [-C fd] device
```

### 参数说明

> device ： 预备检查的硬盘 partition，例如：/dev/sda1
> -a : 对 partition 做检查，若有问题便自动修复，等同 -p 的功能
> -b : 设定存放 superblock 的位置
> -B : 设定单位 block 的大小
> -c : 检查该partition 是否有坏轨
> -C file : 将检查的结果存到 file 中以便查看
> -d : 列印 e2fsck 的 debug 结果
> -f : 强制检查
> -F : 在开始检查前，将device 的 buffer cache 清空，避免有错误发生
> -l bad_blocks_file : 将有坏轨的block资料加到 bad_blocks_file 里面
> -L bad_blocks_file : 设定坏轨的block资料存到 bad_blocks_file 里面，若无该档则自动产生
> -n : 将档案系统以[唯读]方式开启
> -p : 对 partition 做检查，若有问题便自动修复
> -v : 详细显示模式
> -V : 显示出目前 e2fsck 的版本
> -y : 预先设定所有检查时的问题均回答[是]

## 实例

检查 /dev/hda5 是否正常，如果有异常便自动修复，并且设定若有问答，均回答[是] :

```bash
e2fsck -a -y /dev/hda5
```

> 注意 :
>大部份使用 e2fsck 来检查硬盘 partition 的情况时，通常都是情形特殊，因此最好先将该 partition umount，然后再执行 e2fsck 来做检查，若是要非要检查 / 时，则请进入 singal user mode 再执行。
