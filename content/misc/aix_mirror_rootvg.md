---
title: "Mirror rootvg in AIX"
date: 2018-01-16 17:30:44
---
[TOC]
##1、查看当前rootvg是否存在镜像
```
# lsvg -l rootvg
rootvg:
LV NAME             TYPE       LPs     PPs     PVs  LV STATE      MOUNT POINT
hd5                 boot       1       1       1    closed/syncd  N/A
hd6                 paging     2       2       1    open/syncd    N/A
hd8                 jfs2log    1       1       1    open/syncd    N/A
hd4                 jfs2       2       2       1    open/syncd    /
hd2                 jfs2       10      10      1    open/syncd    /usr
hd9var              jfs2       2       2       1    open/syncd    /var
hd3                 jfs2       1       1       1    open/syncd    /tmp
hd1                 jfs2       1       1       1    open/syncd    /home
hd10opt             jfs2       2       2       1    open/syncd    /opt
hd11admin           jfs2       1       1       1    open/syncd    /admin
livedump            jfs2       1       1       1    open/syncd    /var/adm/ras/livedump
# lsvg -p rootvg
rootvg:
PV_NAME           PV STATE          TOTAL PPs   FREE PPs    FREE DISTRIBUTION
hdisk0            active            546         522         109..106..89..109..109
```

如果PPs是LPs的两倍，则表示存在镜像。
##2、检查镜像设备，并设置为PV
```
# lsdev -Ccdisk
hdisk0 Available 00-08-00 SAS Disk Drive
hdisk1 Available 00-08-00 SAS Disk Drive
# chdev -l hdisk1 -a pv=yes
hdisk1 changed
```
##3、确认两块盘大小是否一致
```
# lsattr -El hdisk0
# lsattr -El hdisk1
```
## 4、将hdisk1加入rootvg
```
# extendvg -f rootvg hdisk1
```
##5、开始镜像
```
# mirrorvg rootvg
```
##6、设置boot信息
```
bosboot -a -d /dev/hdisk1
```
##7、更改启动顺序
```
bootlist -m normal hdisk0 hdisk1 cd0
```
##8、查看引导顺序
```
bootlist -m normal -o
```

##删除镜像步骤：
```
1、unmirrorvg rootvg hdisk1
2、chpv -c hdisk1
3、reducevg rootvg hdisk1
4、bootlist -m normal hdisk0 
```


