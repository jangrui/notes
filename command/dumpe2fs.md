# dumpe2fs

## 命令名称

dumpe2fs - 显示ext2/ext3/ext4文件系统信息。

## dumpe2fs命令语法
　　
dumpe2fs [ -bfhixV ] [ -o superblock=superblock ] [ -o blocksize=blocksize ] device

语法看起来比较复杂，看不懂的直接看下面的常用命令选项和实例。

## dumpe2fs命令描述
　　
显示device中文件系统的超级块和块组信息。

## dumpe2fs常用命令选项　　

|选项	|描述
|-|-|
-b	|打印文件系统中的坏块
-o	|不常用，检查严重损坏文件系统时指定
-f	|强制显示所有信息，即便dumpe2fs对有些文件系统功能标识不能识别。
-i	|显示image文件系统信息。device指定image文件的路径
-h	|只显示超级块信息
-x	|将已分组的块的数量用十六进制显示
-v	|显示dumpe2fs的版本号并推出

## dumpe2fs命令实例

dumpe2fs查看/dev/sda1的文件系统信息。

```bash
[root@localhost ~]# dumpe2fs /dev/sda1
dumpe2fs 1.41.12 (17-May-2010)
Filesystem volume name:   <none>
Last mounted on:          /boot
Filesystem UUID:          d1488e05-0442-4514-82ff-5f877a51df23
Filesystem magic number:  0xEF53
Filesystem revision #:    1 (dynamic)
Filesystem features:      has_journal ext_attr resize_inode dir_index filetype needs_recovery extent flex_bg sparse_super huge_file uninit_bg dir_nlink extra_isize
Filesystem flags:         signed_directory_hash
Default mount options:    user_xattr acl
Filesystem state:         clean
Errors behavior:          Continue
Filesystem OS type:       Linux
Inode count:              128016
Block count:              512000
Reserved block count:     25600
Free blocks:              465996
Free inodes:              127978
First block:              1
Block size:               1024
Fragment size:            1024
Reserved GDT blocks:      256
Blocks per group:         8192
Fragments per group:      8192
Inodes per group:         2032
Inode blocks per group:   254
Flex block group size:    16
Filesystem created:       Tue Apr 15 04:22:51 2014
Last mount time:          Tue Sep 16 01:22:26 2014
Last write time:          Tue Sep 16 01:22:26 2014
Mount count:              5
Maximum mount count:      -1
Last checked:             Tue Apr 15 04:22:51 2014
Check interval:           0 (<none>)
Lifetime writes:          43 MB
Reserved blocks uid:      0 (user root)
Reserved blocks gid:      0 (group root)
First inode:              11
Inode size:               128
Journal inode:            8
Default directory hash:   half_md4
Directory Hash Seed:      f562772f-0825-4753-a0b1-5bc295821472
Journal backup:           inode blocks
Journal features:         (none)
Journal size:             8M
Journal length:           8192
Journal sequence:         0x00000018
Journal start:            0
Group 0: (Blocks 1-8192) [ITABLE_ZEROED]
  Checksum 0xcdec, unused inodes 2015
...................略
```