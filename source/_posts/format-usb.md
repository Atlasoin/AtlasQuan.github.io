---
title: Linux下U盘无法挂载
date: 2020-08-22 22:22:28
tags:
- Linux
- USB
---

U盘无法挂载，这种事情总是会被遇上的嘛。比如我之前想把U盘格式化为FAT32格式，但是我也不知道为什么会出现格式化失败的情况，然后U盘就处于损坏状态，再也无法被系统识别（既无法被Windows识别，也无法被Ubuntu识别），也就是它再也无法出现在文件管理器里。

数据倒是之前有备份过，但是想着这个U盘从此就这么废了还挺可惜。加上今天正好打算做一个多系统启动盘，就想着能不能想办法救活这个U盘。

查了一下其实操作也简单，虽然U盘内部损坏了，不能被系统挂载，但是U盘本身的磁盘（disk）是可以被计算机检测到的，所以我们可以通过以下步骤来格式化特定的磁盘（disk）：

1. 查找U盘所在的分区
使用``sudo fdisk -l``指令列出当前的所有分区，通常U盘所在分区会出现在最后。

```
$ sudo fdisk -l
...
Disk /dev/sda: 28.9 GiB, 31029460992 bytes, 60604416 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x456f9c1d

Device     Boot Start      End  Sectors  Size Id Type
/dev/sda1  *    36096 60604415 60568320 28.9G  c W95 FAT32 (LBA)
```

比如我的U盘所在的分区就是/dev/sda1。

2. 格式化此分区

使用``mkdosfs -F 32 -I /dev/sda1``指令后即可格式化U盘了。


操作完成以上步骤后我的U盘终于重新出现在了任务管理器里！

