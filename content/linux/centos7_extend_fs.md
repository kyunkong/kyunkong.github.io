---
title: "Extend file system on CentOS 7"
date: 2018-03-15 11:57:48
---

大部分的命令及方法都没变，唯一的区别是CentOS 7默认的file system变成了xfs，因此最后自动扩展文件系统的时候命令变成了`xfs_growfs`.

### Extend PV
```
[root@centos worktmp]# pvcreate /dev/sdb
```
### Extend VG
```
[root@centos worktmp]# vgdisplay 
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        1
  Metadata Sequence No  3
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                1
  Act PV                1
  VG Size               14.70 GiB
  PE Size               4.00 MiB
  Total PE              3764
  Alloc PE / Size       3764 / 14.70 GiB
  Free  PE / Size       0 / 0   
  VG UUID               Nas46G-QvjO-t8O5-FG2n-OTGE-08Pm-xBPD3n

[root@centos worktmp]# vgextend centos /dev/sdb
```


### Extend LV
```
[root@centos worktmp]# vgdisplay 
  --- Volume group ---
  VG Name               centos
  System ID             
  Format                lvm2
  Metadata Areas        2
  Metadata Sequence No  5
  VG Access             read/write
  VG Status             resizable
  MAX LV                0
  Cur LV                2
  Open LV               2
  Max PV                0
  Cur PV                2
  Act PV                2
  VG Size               <17.70 GiB
  PE Size               4.00 MiB
  Total PE              4531
  Alloc PE / Size       4272 / <16.69 GiB
  Free  PE / Size       259 / 1.01 GiB
  VG UUID               Nas46G-QvjO-t8O5-FG2n-OTGE-08Pm-xBPD3n


[root@centos worktmp]# lvdisplay
  --- Logical volume ---
  LV Path                /dev/centos/root
  LV Name                root
  VG Name                centos
  LV UUID                ciOwqE-K2ME-dJvf-Tl82-6y7B-UsGM-f4C07Q
  LV Write Access        read/write
  LV Creation host, time localhost, 2018-03-14 16:22:22 +0800
  LV Status              available
  # open                 1
  LV Size                12.70 GiB
  Current LE             3252
  Segments               1
  Allocation             inherit
  Read ahead sectors     auto
  - currently set to     8192
  Block device           253:0

[root@centos worktmp]# lvextend -l +250 /dev/centos/root
```

### Enable file system size

```
[root@centos worktmp]# xfs_growfs /dev/mapper/centos-root 
meta-data=/dev/mapper/centos-root isize=512    agcount=4, agsize=832512 blks
         =                       sectsz=512   attr=2, projid32bit=1
         =                       crc=1        finobt=0 spinodes=0
data     =                       bsize=4096   blocks=3330048, imaxpct=25
         =                       sunit=0      swidth=0 blks
naming   =version 2              bsize=4096   ascii-ci=0 ftype=1
log      =internal               bsize=4096   blocks=2560, version=2
         =                       sectsz=512   sunit=0 blks, lazy-count=1
realtime =none                   extsz=4096   blocks=0, rtextents=0
data blocks changed from 3330048 to 4106240
```
