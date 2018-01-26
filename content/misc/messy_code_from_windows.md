---
title: "Messy Code from Windows"
date: 2018-01-15 18:03:33
---
[TOC]

从windowscopy过来的文件，有很多在Linux平台下显示为乱码，
#查看Linux平台locale：
[kingsley@oc7421025535 note]$ file 新建文本文档.txt 
新建文本文档.txt: ISO-8859 text, with CRLF line terminators
[kingsley@oc7421025535 note]$ locale
LANG=en_US.UTF-8
LC_CTYPE="en_US.UTF-8"
LC_NUMERIC="en_US.UTF-8"
LC_TIME="en_US.UTF-8"
LC_COLLATE="en_US.UTF-8"
LC_MONETARY="en_US.UTF-8"
LC_MESSAGES="en_US.UTF-8"
LC_PAPER="en_US.UTF-8"
LC_NAME="en_US.UTF-8"
LC_ADDRESS="en_US.UTF-8"
LC_TELEPHONE="en_US.UTF-8"
LC_MEASUREMENT="en_US.UTF-8"
LC_IDENTIFICATION="en_US.UTF-8"
LC_ALL=
[kingsley@oc7421025535 note]$ cat /etc/sysconfig/i18n
LANG="en_US.UTF-8"
SYSFONT="latarcyrheb-sun16"
字符编码为UTF-8，尝试用dos2unix转换，也失败，但在大部分Linux平台，有iconv可以对字符编码进行转换：
#查看该文件编码方式
[kingsley@oc7421025535 note]$ file 新建文本文档.txt 
新建文本文档.txt: ISO-8859 text, with CRLF line terminators
#查询iconv是否支持中文字符
iconv -l|grep -i iso-8859
尝试用iconv转换成ISO-8859的，表示失败，最后尝试转换成GBK，成功：
iconv -f GBK -t UTF-8 新建文本文档.txt -o 1.txt
[kingsley@oc7421025535 note]$ file 1.txt 
1.txt: UTF-8 Unicode text, with CRLF line terminators


## the ksh script to remove the ^M on AIX
```sh
#!/bin/ksh
DIR=/usr/local/
for file in $(find $DIR -type f); do
tr -d '\r' <$file>temp.$$ && mv temp.$$ $file
done
```
