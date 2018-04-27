---
title: "Extending file system on AIX"
date: 2018-01-16 16:32:49
---
[TOC]

# 一、扩展swap
```
# lsps -a
Page Space      Physical Volume   Volume Group Size %Used Active  Auto  Type Chksum
hd6             hdisk0            rootvg         512MB     2   yes   yes    lv     0
# lsvg rootvg
VOLUME GROUP:       rootvg                   VG IDENTIFIER:  00c224c600004c000000014ce545ae9f
VG STATE:           active                   PP SIZE:        256 megabyte(s)
VG PERMISSION:      read/write               TOTAL PPs:      1092 (279552 megabytes)
MAX LVs:            256                      FREE PPs:       1044 (267264 megabytes)
LVs:                11                       USED PPs:       48 (12288 megabytes)
OPEN LVs:           10                       QUORUM:         1 (Disabled)
TOTAL PVs:          2                        VG DESCRIPTORS: 3
STALE PVs:          0                        STALE PPs:      0
ACTIVE PVs:         2                        AUTO ON:        yes
MAX PPs per VG:     32512                                     
MAX PPs per PV:     1016                     MAX PVs:        32
LTG size (Dynamic): 1024 kilobyte(s)         AUTO SYNC:      no
HOT SPARE:          no                       BB POLICY:      relocatable 
PV RESTRICTION:     none                     INFINITE RETRY: no
```
## 1、扩展成4GB大小，每个PP 256M，即需要16个PP，原来有2个PP，加14个即可
```
#chps -s 14 hd6
# lsps -a
Page Space      Physical Volume   Volume Group Size %Used Active  Auto  Type Chksum
hd6             hdisk0            rootvg        4096MB     1   yes   yes    lv     0
```

#二、从原有vg中创建文件系统
##1、创建lv，-c表示copy，仅针对有做镜像的VG生效
```
[root@:/mnt/installp]# mklv -c 2 -t jfs2 -y oralv rootvg 120 hdisk0 hdisk1
oralv
[root@:/mnt/installp]# lsvg -l rootvg
rootvg:
LV NAME             TYPE       LPs     PPs     PVs  LV STATE      MOUNT POINT
hd5                 boot       1       2       2    closed/syncd  N/A
hd6                 paging     16      32      2    open/syncd    N/A
hd8                 jfs2log    1       2       2    open/syncd    N/A
hd4                 jfs2       2       4       2    open/syncd    /
hd2                 jfs2       10      20      2    open/syncd    /usr
hd9var              jfs2       2       4       2    open/syncd    /var
hd3                 jfs2       1       2       2    open/syncd    /tmp
hd1                 jfs2       1       2       2    open/syncd    /home
hd10opt             jfs2       2       4       2    open/syncd    /opt
hd11admin           jfs2       1       2       2    open/syncd    /admin
livedump            jfs2       1       2       2    open/syncd    /var/adm/ras/livedump
oralv               jfs2       120     240     2    closed/syncd  N/A
```

##2、创建文件系统
```
[root@:/mnt/installp]# mkdir /u01
[root@:/mnt/installp]# crfs -v jfs2 -d oralv -A yes -m /u01
File system created successfully.
31456116 kilobytes total disk space.
New File System size is 62914560
[root@:/mnt/installp]# mount /u01
[root@:/mnt/installp]# lsvg -l rootvg
rootvg:
LV NAME             TYPE       LPs     PPs     PVs  LV STATE      MOUNT POINT
hd5                 boot       1       2       2    closed/syncd  N/A
hd6                 paging     16      32      2    open/syncd    N/A
hd8                 jfs2log    1       2       2    open/syncd    N/A
hd4                 jfs2       2       4       2    open/syncd    /
hd2                 jfs2       10      20      2    open/syncd    /usr
hd9var              jfs2       2       4       2    open/syncd    /var
hd3                 jfs2       1       2       2    open/syncd    /tmp
hd1                 jfs2       1       2       2    open/syncd    /home
hd10opt             jfs2       2       4       2    open/syncd    /opt
hd11admin           jfs2       1       2       2    open/syncd    /admin
livedump            jfs2       1       2       2    open/syncd    /var/adm/ras/livedump
oralv               jfs2       120     240     2    open/syncd    /u01


## Extend the file system
chfs -a size=+1G /opt
```


## How to locate HACMP script
```
>smitty hacmp>>Cluster Applications and Resources>>Resources>>Configure user Applications (Scripts and Monitors)
>>Applications control Scripts>>Change/Show Application controller scripts
```

