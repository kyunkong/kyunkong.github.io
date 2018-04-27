---
title: "lsof command"
date: 2018-01-16 14:30:01
---

lsof:
   list opening file descriptor

## Default lsof output
| Column  | Description                                                                                 |
| ------- | ------------------------------------------------------------------------------------------- |
| COMMAND | The first nine characters of the name of the command in the process                         |
| PID     | The process id of the process                                                               |
| USER    | The login name of the user who owns the process                                             |
| FD      | The file descriptor number and the access type, r for read, w for write, u for read/write   |
| TYPE    | The type of file, CHR for character, BLK for block, DIR for directory, REG for regular file |
| DEVICE  | The device number of the device, major and the minor                                        |
| SIZE    | If available, the file size                                                                 |
| NODE    | The node number of the local file                                                           |
| NAME    | The name of file                                                                            |


## lsof essential options
```
-a:   list 
-c<command>:  list the opening files by the specified command
-g:   list the details of gid process
-d<file_number>:     list the process who owns the specified file
+d:   list the opening file under the specified directory
+D:   recursive option for +d
-p:   list the opening file by specified process
-u:   list the detail process of uid
```



## examples
### list all the opening files under the /worktmp
```
$ lsof +D /worktmp/
COMMAND   PID     USER   FD   TYPE DEVICE SIZE/OFF    NODE NAME
bash     5029 kingsley  cwd    DIR  253,1     4096 3409241 /worktmp/note/scripts/practice
bash    16358 kingsley  cwd    DIR  253,1     4096 2105360 /worktmp/note/shellscript
vimx    16487 kingsley  cwd    DIR  253,1     4096 2105360 /worktmp/note/shellscript
```

### list the opening file of the given process
```
$ lsof -p 29396
COMMAND   PID     USER   FD   TYPE             DEVICE SIZE/OFF    NODE NAME
tmux    29396 kingsley  cwd    DIR              253,1     4096 3409241 /worktmp/note/scripts/practice
```

### list the openingfile of the given user
```
 lsof -u kingsley|tail -5
 bash      29399 kingsley  mem       REG              253,1     26060     135506 /usr/lib64/gconv/gconv-modules.cache
 bash      29399 kingsley  255u      CHR              136,1       0t0          4 /dev/pts/1
```

### list only process of the given user
```
$ lsof -t -u kingsley|tail -5
24266
29307
29396
29398
29399
```


***if you want to kill someone's opening files***
```
kill -9 `lsof -t -u usernmae`
```

### list all of the tcp, udp connections:
```
lsof -i tcp
lsof -i udp
lsof -i:22
```

### find the process of the specified NIC
```
$ lsof -i@9.83.19.119
COMMAND   PID     USER   FD   TYPE DEVICE SIZE/OFF NODE NAME
sametime 5461 kingsley  326u  IPv4 310317      0t0  TCP 9.83.19.119:36909->d03im403.boulder.ibm.com:virtual-places (ESTABLISHED)
lnotes   5709 kingsley   48u  IPv4 284523      0t0  TCP 9.83.19.119:43825->aa.04.2da9.ip4.static.sl-reverse.com:lotusnote (CLOSE_WAIT)
firefox  8187 kingsley   45u  IPv4 884148      0t0  TCP 9.83.19.119:37990->sgce02.sg.ibm.com:http (ESTABLISHED)
firefox  8187 kingsley   54u  IPv4 892001      0t0  TCP 9.83.19.119:38017->sgce02.sg.ibm.com:http (ESTABLISHED)
```

### find the opening file by specified application
```
$ lsof -c firefox
COMMAND  PID     USER   FD   TYPE             DEVICE SIZE/OFF    NODE NAME
firefox 8187 kingsley  cwd    DIR              253,1     4096 1048688 /home/kingsley
firefox 8187 kingsley  rtd    DIR              253,1     4096       2 /
firefox 8187 kingsley  txt    REG              253,1   129032  524462 /usr/lib64/firefox/firefox
firefox 8187 kingsley  mem    REG              253,1   177520  541673 /lib64/libk5crypto.so.3.1
```

### disk space warning
```
lsof|grep -i delete
```


