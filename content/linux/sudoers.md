---
title: "RHEL 7 sudoer"
date: 2018-03-15 09:47:59
---
修改用户sudo权限一般可以用`visudo`，此命令相当于`vi /etc/sudoers`。
或者可以在`/etc/sudoers.d`目录下新建个文本，将sudo权限写入，在`/etc/sudoers`会有默认`#includedir /etc/sudoers.d`一行，这个并不是备注，同时`#`号后面也不能有空格。

规范来讲，建议在/etc/sudoers.d下面新建个文件，而不是直接写入到/etc/sudoers去。
下列例子说明sudoer的用法：

### 通过不同的别名对命令进行分类
```
Cmnd_Alias SYSTEM = /usr/bin/su, /usr/sbin/shutdown, /usr/bin/yum
Cmnd_Alias MAINT = /usr/bin/yum
Cmnd_Alias NETWORKING = /sbin/route, /sbin/ifconfig, /bin/ping, /sbin/dhclient, /usr/bin/net, /sbin/iptables, /usr/bin/rfcomm, /usr/bin/wvdial, /sbin/iwconfig, /sbin/mii-tool
Cmnd_Alias SOFTWARE = /bin/rpm, /usr/bin/up2date, /usr/bin/yum
Cmnd_Alias      POWER = /sbin/shutdown, /sbin/halt, /sbin/reboot, /sbin/restart

User_Alias      GROUPONE = abby, brent, carl
User_Alias      GROUPTWO = brent, doris, eric,
User_Alias      GROUPTHREE = doris, felicia, grant


%netadmin ALL = NETWORKING  # Example 1
%wheel ALL=(ALL) ALL          # Example 2
GROUPTWO    ALL = SOFTWARE    # Example 3
GROUPTHREE  ALL = POWER       # Example 4
```
例1，将别名NETWORKING的命令组分配给netadmin的用户组，ALL表示对登陆主机不限制。
例2， 授权用户组wheel执行所有命令。括号里面的ALL表示能切换到任何用户。
例3， 授权用户组别名GROUPTWO的用户有执行SOFTWARE命令的权限。
例4， 授权用户组别名GROUPTHREE的用户有执行POWER命令的权限。

```
Runas_Alias     WEB = www-data, apache
GROUPONE    ALL = (WEB) NOPASSWD: ALL
```
上例中，将授权GROUPONE的用户有执行www-data, apache的权限, 且sudo的时候不需要输入密码, NOPASSWD只针对后面一个命令有效。

```
GROUPTWO    ALL = NOPASSWD: /usr/bin/updatedb, PASSWD: /bin/kill
```
为了安全起见，可以分别对某些命令设置sudo密码，而其他的则不需要输入密码





