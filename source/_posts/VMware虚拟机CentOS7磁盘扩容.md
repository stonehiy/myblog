---
title: VMware虚拟机CentOS7磁盘扩容
date: 2021-01-11 15:30:40
tags:
---


### 查看分区

1.查看```df -h```

```
[root@localhost ~]# cd /
[root@localhost /]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 899M     0  899M    0% /dev
tmpfs                    910M     0  910M    0% /dev/shm
tmpfs                    910M  9.5M  901M    2% /run
tmpfs                    910M     0  910M    0% /sys/fs/cgroup
/dev/mapper/centos-root   17G   17G  1.6M  100% /
/dev/sda1               1014M  194M  821M   20% /boot
tmpfs                    182M     0  182M    0% /run/user/0

```

### 虚拟机硬盘扩展

 先进入虚拟机设置里增大磁盘空间

1.VMware中关闭CentOS

2.打开VMware CentOS设置

3.到硬盘(SCSI)选项中-磁盘实用工具-扩展

4.扩展完成，重新开启CentOS


### CentOS7磁盘扩容

1.查看```fdisk -l```

```
[root@localhost lib]# sudo fdisk -l

磁盘 /dev/sda：48.3 GB, 48318382080 字节，94371840 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000dffc7

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    41943039    19921920   8e  Linux LVM

磁盘 /dev/mapper/centos-root：18.2 GB, 18249416704 字节，35643392 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/mapper/centos-swap：2147 MB, 2147483648 字节，4194304 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节

```

2.接下来增加一个分区```fdisk /dev/sda```

```
[root@localhost lib]# fdisk /dev/sda

欢迎使用 fdisk (util-linux 2.23.2)。

更改将停留在内存中，直到您决定将更改写入磁盘。
使用写入命令前请三思。


命令(输入 m 获取帮助)：m
命令操作
   a   toggle a bootable flag
   b   edit bsd disklabel
   c   toggle the dos compatibility flag
   d   delete a partition
   g   create a new empty GPT partition table
   G   create an IRIX (SGI) partition table
   l   list known partition types
   m   print this menu
   n   add a new partition
   o   create a new empty DOS partition table
   p   print the partition table
   q   quit without saving changes
   s   create a new empty Sun disklabel
   t   change a partition's system id
   u   change display/entry units
   v   verify the partition table
   w   write table to disk and exit
   x   extra functionality (experts only)

命令(输入 m 获取帮助)：n
Partition type:
   p   primary (2 primary, 0 extended, 2 free)
   e   extended
Select (default p): p
分区号 (3,4，默认 3)：3
起始 扇区 (41943040-94371839，默认为 41943040)：
将使用默认值 41943040
Last 扇区, +扇区 or +size{K,M,G} (41943040-94371839，默认为 94371839)：
将使用默认值 94371839
分区 3 已设置为 Linux 类型，大小设为 25 GiB

命令(输入 m 获取帮助)：w
The partition table has been altered!

Calling ioctl() to re-read partition table.

WARNING: Re-reading the partition table failed with error 16: 设备或资源忙.
The kernel still uses the old table. The new table will be used at
the next reboot or after you run partprobe(8) or kpartx(8)
正在同步磁盘。

```

注意：“起始扇区”那里直接回车，随便乱写容易造成空间浪费

现在系统就有3个分：sda1,sda2,sda3

```
[root@localhost /]# fdisk -l

磁盘 /dev/sda：48.3 GB, 48318382080 字节，94371840 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节
磁盘标签类型：dos
磁盘标识符：0x000dffc7

   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200    41943039    19921920   8e  Linux LVM
/dev/sda3        41943040    94371839    26214400   83  Linux

磁盘 /dev/mapper/centos-root：18.2 GB, 18249416704 字节，35643392 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节


磁盘 /dev/mapper/centos-swap：2147 MB, 2147483648 字节，4194304 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 512 字节
I/O 大小(最小/最佳)：512 字节 / 512 字节

```

### 创建物理卷

1.命令：```pvcreate /dev/sda3```

```
[root@localhost lib]# pvcreate /dev/sda3
Device /dev/sda3 not found.
```

2.重启后再：```pvcreate /dev/sda3```提示成功

```
[root@localhost /]# pvcreate /dev/sda3
Physical volume "/dev/sda3" successfully created.
```

3.使用```vgscan```查询物理卷,可以查到本机物理卷名称为```centos```

```
[root@localhost /]# vgscan
  Reading volume groups from cache.
  Found volume group "centos" using metadata type lvm2

```

4.使用新增物理卷扩展```centos```命令：```vgextend centos  /dev/sda3```

```
[root@localhost /]# vgextend centos  /dev/sda3
  Volume group "centos" successfully extended

```

5.命令```lvextend -L +24G /dev/mapper/centos-root```:使用```lvextend```命令为逻辑卷```/dev/mapper/centos-root```增加24G空间

```
[root@localhost /]# lvextend -L +24G /dev/mapper/centos-root
  Size of logical volume centos/root changed from <17.00 GiB (4351 extents) to <41.00 GiB (10495 extents).
  Logical volume centos/root successfully resized.

```

6.命令```df -h```发现实际容量并没有变化

```
[root@localhost /]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 899M     0  899M    0% /dev
tmpfs                    910M     0  910M    0% /dev/shm
tmpfs                    910M  9.5M  901M    2% /run
tmpfs                    910M     0  910M    0% /sys/fs/cgroup
/dev/mapper/centos-root   17G   17G  1.6M  100% /
/dev/sda1               1014M  194M  821M   20% /boot
tmpfs                    182M     0  182M    0% /run/user/0

```

因为我们的系统还不认识刚刚添加进来的磁盘的文件系统，所以还需要对文件系统进行扩容

xfs_growfs  加上要扩展的分区名

或者

resize2fs – f 加 上要扩展的分区名

```
[root@localhost /]# xfs_growfs  /dev/mapper/centos-root
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=1113856 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=4455424, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 4455424 to 10746880

```
7..命令```df -h```,由17G扩展为41G

```
[root@localhost /]# df -h
文件系统                 容量  已用  可用 已用% 挂载点
devtmpfs                 899M     0  899M    0% /dev
tmpfs                    910M     0  910M    0% /dev/shm
tmpfs                    910M  9.5M  901M    2% /run
tmpfs                    910M     0  910M    0% /sys/fs/cgroup
/dev/mapper/centos-root   41G   17G   25G   42% /
/dev/sda1               1014M  194M  821M   20% /boot
tmpfs                    182M     0  182M    0% /run/user/0

```

