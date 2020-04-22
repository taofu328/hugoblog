---
title: "centos7下lvm磁盘扩容"
date: 2020-02-16T19:41:26+08:00
draft: false
tags: [
    "centos",
    "linux",
	"服务器",
]
---



机器有两块硬盘，安装操作系统的时候用了一块，准备把另一块扩充进来。

- 查看现有的硬盘分区
```shell
[root@indexserver1b ~]# df -h
文件系统             容量  已用  可用 已用% 挂载点
/dev/mapper/cl-root   50G  3.3G   47G    7% /
devtmpfs             7.7G     0  7.7G    0% /dev
tmpfs                7.8G   84K  7.8G    1% /dev/shm
tmpfs                7.8G  9.4M  7.7G    1% /run
tmpfs                7.8G     0  7.8G    0% /sys/fs/cgroup
/dev/sda1           1014M  173M  842M   17% /boot
/dev/mapper/cl-home  873G   33M  873G    1% /home
tmpfs                1.6G   16K  1.6G    1% /run/user/42
tmpfs                1.6G     0  1.6G    0% /run/user/0
```

- 查看新增加的硬盘路径
```shell
[root@indexserver1b ~]# fdisk -l
磁盘 /dev/sda：1000.2 GB, 1000204886016 字节，1953525168 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节
磁盘标签类型：dos
磁盘标识符：0x000be271
   设备 Boot      Start         End      Blocks   Id  System
/dev/sda1   *        2048     2099199     1048576   83  Linux
/dev/sda2         2099200  1953523711   975712256   8e  Linux LVM
磁盘 /dev/sdb：1000.2 GB, 1000204886016 字节，1953525168 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节
磁盘 /dev/mapper/cl-root：53.7 GB, 53687091200 字节，104857600 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节
磁盘 /dev/mapper/cl-swap：8388 MB, 8388608000 字节，16384000 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节
磁盘 /dev/mapper/cl-home：937.0 GB, 937045262336 字节，1830166528 个扇区
Units = 扇区 of 1 * 512 = 512 bytes
扇区大小(逻辑/物理)：512 字节 / 4096 字节
I/O 大小(最小/最佳)：4096 字节 / 4096 字节
```

- 初始化分区sdb为物理卷pv
```shell
[root@indexserver1b ~]# pvcreate /dev/sdb
  Physical volume "/dev/sdb" successfully created.
[root@indexserver1b ~]# pvdisplay
  --- Physical volume ---
  PV Name               /dev/sda2
  VG Name               cl
  PV Size               930.51 GiB / not usable 4.00 MiB
  Allocatable           yes 
  PE Size               4.00 MiB
  Total PE              238210
  Free PE               1
  Allocated PE          238209
  PV UUID               tFdpom-cQYU-IAe0-HmWi-DVBi-hBKR-g3csJ3
   
  "/dev/sdb" is a new physical volume of "931.51 GiB"
  --- NEW Physical volume ---
  PV Name               /dev/sdb
  VG Name               
  PV Size               931.51 GiB
  Allocatable           NO
  PE Size               0   
  Total PE              0
  Free PE               0
  Allocated PE          0
  PV UUID               GW4MP-O2zZ-akX-Zq80-VoaS-y7uj-wBUvq3
```

- 刚创建的PV加入相应的VG
```shell
[root@indexserver1b ~]# vgextend cl /dev/sdb
  Volume group "cl" successfully extended
[root@indexserver1b ~]# vgdisplay
  --- Volume group ---
  VG Name               cl
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                3
  Open LV               3
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               1.82 TiB
  PE Size               4.00 MiB
  Total PE              476677
  Alloc PE / Size       238209 / 930.50 GiB
  Free  PE / Size       238468 / 931.52 GiB
  VG UUID               3Ba7mm-epEg-CIGm-JgEr-0GsY-RjJc-ytdDZv
```

- 把VG加入到LV
```shell
[root@indexserver1c ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/cl/swap
  LV Name                swap
  VG Name                cl
  LV UUID                S5syZb-ieb0-SdXX-M2Bz-CJQy-bohH-GqlZYy
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-12-04 14:21:24 +0000
  LV Status              available
  # open                 2
  LV Size                7.81 GiB
  Current LE             2000
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/cl/home
  LV Name                home
  VG Name                cl
  LV UUID                uGjleY-QvjI-t5ys-lJW8-lcUH-rRaK-pxnuTh
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-12-04 14:21:24 +0000
  LV Status              available
  # open                 1
  LV Size                872.69 GiB
  Current LE             223409
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/cl/root
  LV Name                root
  VG Name                cl
  LV UUID                yrxmYZ-1E7b-bfQB-eF2k-E3eQ-r40b-SCrnCm
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-12-04 14:21:29 +0000
  LV Status              available
  # open                 1
  LV Size                50.00 GiB
  Current LE             12800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
[root@indexserver1b ~]#  lvextend -l +238468 /dev/cl/home
  Size of logical volume cl/home changed from 872.69 GiB (223409 extents) to 1.76 TiB (461877 extents).
  Logical volume cl/home successfully resized.
  
[root@indexserver1c ~]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/cl/swap
  LV Name                swap
  VG Name                cl
  LV UUID                S5syZb-ieb0-SdXX-M2Bz-CJQy-bohH-GqlZYy
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-12-04 14:21:24 +0000
  LV Status              available
  # open                 2
  LV Size                7.81 GiB
  Current LE             2000
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:1
   
  --- Logical volume ---
  LV Path                /dev/cl/home
  LV Name                home
  VG Name                cl
  LV UUID                uGjleY-QvjI-t5ys-lJW8-lcUH-rRaK-pxnuTh
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-12-04 14:21:24 +0000
  LV Status              available
  # open                 1
  LV Size                872.69 GiB
  Current LE             223409
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:2
   
  --- Logical volume ---
  LV Path                /dev/cl/root
  LV Name                root
  VG Name                cl
  LV UUID                yrxmYZ-1E7b-bfQB-eF2k-E3eQ-r40b-SCrnCm
  LV Write Access        read/write
  LV Creation host, time localhost.localdomain, 2017-12-04 14:21:29 +0000
  LV Status              available
  # open                 1
  LV Size                50.00 GiB
  Current LE             12800
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     256
  Block device           253:0
```
这里的扩容容量+238468是前面vgdisplay打印结果中的Free PE。

- 调整文件系统大小
```shell
[root@indexserver1b ~]# xfs_growfs /dev/mapper/cl-home
meta-data=/dev/mapper/cl-home    isize=512    agcount=4, agsize=57192704 blks
         =                       sectsz=4096  attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=228770816, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=111704, version=2
         =                       sectsz=4096  sunit=1 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 228770816 to 472962048
[root@indexserver1b ~]# df -h
文件系统             容量  已用  可用 已用% 挂载点
/dev/mapper/cl-root   50G  3.3G   47G    7% /
devtmpfs             7.7G     0  7.7G    0% /dev
tmpfs                7.8G   84K  7.8G    1% /dev/shm
tmpfs                7.8G  9.4M  7.7G    1% /run
tmpfs                7.8G     0  7.8G    0% /sys/fs/cgroup
/dev/sda1           1014M  173M  842M   17% /boot
/dev/mapper/cl-home  1.8T   33M  1.8T    1% /home
tmpfs                1.6G   16K  1.6G    1% /run/user/42
tmpfs                1.6G     0  1.6G    0% /run/user/0
```
可以看到可用/dev/mapper/cl-home 可用空间已经变成1.8T
简化步骤
```shell
pvcreate /dev/sdb
vgextend cl /dev/sdb
lvextend -l +238468 /dev/cl/home
xfs_growfs /dev/mapper/cl-home
```
