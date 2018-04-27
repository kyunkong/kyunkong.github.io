---
title: "Shell script tips"
date: 2018-01-16 14:56:52
---
[TOC]

Shell中特殊符号的含义：

* ';'  分号（Command separator）

表示连续命令，如 `cd /backup ; mkdir -p test`

* 'strings'  单引号（single quote）
引号内为单一字符串，引号内的字符串禁止任何变量替换，如：
```
[kingsley@oc7421025535 Desktop]$ export IDS=ykings
[kingsley@oc7421025535 Desktop]$ echo '$IDS'
$IDS
[kingsley@oc7421025535 Desktop]$ echo "$IDS"
ykings
```

* "strings"   双引号（double quote）
类似单引号，但双引号允许变量扩展，只是禁止了通配符

* :  冒号（colon）
内建指令，表示什么都不做，如 `:>file` 等价与`cat /dev/null > file`

* ${} 变量的正则表达式

```
[kingsley@oc7421025535 practice]$ TIME=`date +%Y%m%d`
[kingsley@oc7421025535 practice]$ echo ${TIME:0:4}
2015
[kingsley@oc7421025535 practice]$ echo $TIME
20151113
[kingsley@oc7421025535 practice]$ echo ${TIME/2015/2016}
20161113
```

* $n 执行脚本时候后面接的变量，如$0表示脚本本身，$1表示第一个变量，如果变量超过两位数，则需要用${10}

* $@ 跟$*类似，两者的区别在于，符号 $* 将所有的引用变量视为一个整体。但符号 $@ 则仍旧保留每个引用变量的区段观念。

* $# 传递给脚本的参数个数

* $* 传递给脚本的所有参数

* $? 脚本返回结果，0为成功，1为失败

```
[kingsley@oc7421025535 practice]$ cat t.ksh
#!/bin/ksh
if [[ $# -ne 2 ]]; then
echo "please use t.ksh \$1 \$2"
exit 1
else
expr $1 + $2
    echo "File Name: $0"
    echo "First Parameter : $1"
    echo "First Parameter : $2"
    echo "Quoted Values: $@"
    echo "Quoted Values: $*"
    echo "Total Number of Parameters : $#"
fi
exit 0
```

# Shell中if判断条件

if有两种判断，一种是接command，一种是接expression， 如果接command， 且返回值为0， 表示命令执行成功，则会执行then后面的语句，否则执行else或者跳出if语句

=========字符串比较=========================
1. -z string   如果字符串长度为0，则为真

```
[kingsley@oc7421025535 practice]$ cat if.ksh
#!/bin/ksh
echo "Please enter a parameter:"
read PARA
if [ -z $PARA ]; then
   echo "Parameter is null"
else
   echo "First Parameter is: $PARA"
fi
exit 0
[kingsley@oc7421025535 practice]$ ./if.ksh
Please enter a parameter:
abc
First Parameter is: abc
[kingsley@oc7421025535 practice]$ ./if.ksh
Please enter a parameter:

Parameter is null
```

2. -n strings  如果字符串长度为非0，则为真

=================算数比较===================
```
1. -eq 等于
2. -ne 不等于
3. -lt 小于
4. -le 小于等于
5. -ge 大于等于
6. -gt 大于
```

==================文件及目录判断=================
```
1. -e filename 文件存在为真
2. -d dirname  如果是目录，则为真
3. -f filename 如果为常规文件，则为真
4. -L filename 如果为symbolic link，则为真
5. -r filename 如果文件可读，则为真
6. -w filename 如果文件可写，则为真
7. -x filename 如果文件可执行，则为真
8. f1 -nt f2   如果f1比f2新，则为真
9. f1 -ot f2   如果f1比f2旧，则为真

[kingsley@oc7421025535 Jason_db2level_scan]$ echo -e "1\n2\n3"
1
2
3
[kingsley@oc7421025535 Jason_db2level_scan]$ echo  "1\n2\n3"
1\n2\n3

```



==================Print color output in screen===================
Colors are represented by color codes, some examples being, reset = 0, black = 30, red = 31,green = 32, yellow = 33, blue = 34, magenta = 35, cyan = 36, and white = 37.
```
echo -e "\e[1;31m This is red text \e[0m"
```

Here, \e[1;31m is the escape string that sets the color to red and \e[0m resets the color back. Replace 31 with the required color code.

For a colored background, reset = 0, black = 40, red = 41, green = 42, yellow = 43, blue = 44, magenta = 45, cyan = 46, and white=47, are the color codes that are commonly used.
To print a colored background, enter the following command:
```
echo -e "\e[1;42m Green Background \e[0m"
```


===================================
===      Finding length of strings==
===================================
```
[root@oc7421025535:/worktmp/note/scripts/practice]# export IDS=yking
[root@oc7421025535:/worktmp/note/scripts/practice]# length=${#IDS}
[root@oc7421025535:/worktmp/note/scripts/practice]# echo $length
5
```


===========================================
===   Checking for root user        =======
===========================================
```
#!/bin/sh
if [ $UID -ne 0 ]; then
    echo "Non root user, please run as root."
else
    echo "ROOT user."
fi
exit 0
```


===========================================
===   Read the parameter from a file    ======
===========================================
```
[root@oc7421025535 practice]# cat out.txt
file
[root@oc7421025535 practice]# find . -type f -print | xargs $(cat out.txt )
[root@oc7421025535 practice]# find . -type f -print | xargs `awk {'print $0'} out.txt`
var= $( cat file )
```


===========================================
===   Regular Expression            =======
===========================================
#1. Regex

```
^  行首定位符  /^love/  匹配所有以love开头的行
$  行尾定位符  /love$/  匹配所有以love结尾的行
.  匹配单个字符   /l..e/   匹配包含一个l，后面跟两个字符，再跟一个e的行
*  匹配0或者多个重复的位于*前的字符    / *love/    匹配包含跟在0个或者多个字符后的love的行
[]    匹配一组字符中的任意一个   /[Ll]ove/   匹配Love或者love
[x-y]    匹配指定范围内的一个字符   /[A-Z]ove/  匹配后面跟着ove的一个A至Z的任意字符
[^]   匹配不在指定组内的字符  /[^A-Z]/    匹配不在范围A至Z之间的任意一个字符
\  转义字符    /love\./    匹配包含love，后面跟一个句号
```


#2. grep command parameters
```
-E 扩展grep，等同于egrep         如: grep -E "[a-z]+" filename等价于egrep "[a-z]+" filename
-o 不输出整行，只输出匹配字符串     echo "Hello world"|grep -o world,只输出world
-c 匹配行出现的次数，而不是字符串出现的次数  echo -e "this is a word word" |grep -c word结果为1，而不是2
-n 打印匹配行的行号
-Rn   递归查找匹配行          grep -Rn ./ "text"
-e 匹配多个匹配项          如果想匹配多个条件，可使用-e参数:grep -e "p1" -e "p2"
-A/-B 打印匹配行的前后记录       grep "text" -A 5 -B 10表示显示前十行和后五行
```

#3 cut command
```
-fn      n为数字，打印第n列。 cut -f1打印第一列；cut -f 2,3打印2，3列
-fn -d";"   指定分割符，这里为";"
-c    打印匹配字符串，cut -c1-5 打印前五个字符
```

#4 sed command

```
命令                             功能
sed -n ‘10,20p file                 打印10~20行
sed ‘1,10s/AM/PM/g’ file               将1~10行所有AM修改为PM
sed ‘/March/!d’ file                   删除所有不含March的行
sed ‘/reports/s/5/8’ file              将所有包含reports的行出现的第一个5改为8
sed ‘s/….//’ file                      删除每行的前四个字符
sed ‘s/…$//’ file                      删除每行的后三个字符
sed ‘/east/,/west/s/A/B/’ file            把从east到west这个范围内所有行中出现的A改为B
sed -n ‘/Time/w newfile’ file             将file中所有包含Time的行写入到newfile中
sed ‘s/\([Oo]ccur\)ence/\1rence/’ file       将所有的Occurence替换成Occurrence，将所有的occurence替换成occurrence
sed '/^$/d' file                    删除空行
sed -i 'patten/s/abc/xyz/g'   file        直接将所有匹配行的abc修改为xyz，直接保存到原文件
sed 's/this/THIS/3g'                从第三个匹配项开始替换
echo an example | sed 's/\w\+/[&]/g'      结果为[an] [example], /w/表示每一个词，把它取代为[&],&代表之前的每一个匹配
```

#5 awk command
```
$0 表示整条记录
NR 表示每条记录的记录号
NF 表示多少个字段，即多少列
$NF   表示最后一列
```

### awk 'NR < 5' # first four lines #

```
awk 'NR==1,NR==4' #First four lines
awk '/linux/' # Lines containing the pattern linux (we can specify regex)
awk '!/linux/' # Lines not containing the pattern linux
```

#6 summary
```
--Parsing email address:
egrep -o '[A-Za-z0-9._]+@[A-Za-z0-9.]+\.[a-zA-Z]{2,4}' file

--Finding largest size files
du -ak ./ | sort -nrk 1 | head
find . -type f -exec du -k {} \; | sort -nrk 1 | head
```

#Delete duplicate lines
```
1. sort -n $file | awk '{if($0!=line)print; line=$0}'
2. sort -n test.txt | uniq
```

#Replace tab to other
```
sed 's/\t/./g' file  #replace the tab to '.'
```

* 批量替换当前目录所有文件，将YES修改为NO
```
sed -i "s/YES/NO/g" `grep "YES" -rl *`
sed -i "s/nil/null/g" `grep "nil" -rl *`
```

## Finding dotfile
```
du -sh * .[^.]*
```

## bash cut usages
### select column of characters
Below example will show the first characters of /etc/passwd file, you can replace the number one to other numbers, it means which characters
```
cut -c1 /etc/passwd
```
### select column of characters using Range
Below example will print the first 9 characters of /etc/passwd.
```
cut -c1-9 /etc/passwd
```
Below example will print from the third charater to end of file /etc/passwd
```
cut -c3- /etc/passwd
```
Below example will print from first charater to 10th character of file /etc/passwd
```
cut -c-10 /etc/passwd
```
### select a specific field from a file
This usage somewhat like awk, we can compare with these two commands
Print filed one of file /etc/passwd, the delimiter is ":"
```
cut -d":" -f1 /etc/passwd
awk -F: '{print $1'} /etc/passwd
```
### select multiple fields from a file
Print field 1,3 and 5; print from filed 1 to 3 plus 5.
```
cut -d":" -f1,3,5 /etc/passwd #awk -F: '{print $1, $3, $5}'
cut -d":" -f1-3,5 /etc/passwd
```
### Select Fields Only When a Line Contains the Delimiter
In our /etc/passwd example, if you pass a different delimiter other than : (colon), cut will just display the whole line.
In the following example, we’ve specified the delimiter as | (pipe), and cut command simply displays the whole line, even when it doesn’t find any line that has | (pipe) as delimiter.
```
cut -d"|" -f1 /etc/passwd
```
But, it is possible to filter and display only the lines that contains the specified delimiter using -s option.
The following example doesn’t display any output, as the cut command didn’t find any lines that has | (pipe) as delimiter in the /etc/passwd file.
```
cut -d"|" /etc/passwd -s -f1
```
### select all fields except the specified field
Print all the fields except field one
```
cut -d":" --complement -s -f1 /etc/passwd
```
### change output delimiter for display
```
grep "/bin/bash" /etc/passwd|cut -d":" -s -f1-3 --output-delimiter='*'
```


##Remove the ^M character from Linux/UNIX
```
https://www.cyberciti.biz/faq/sed-remove-m-and-line-feeds-under-unix-linux-bsd-appleosx/
cat -v file_name will show the ^M character in the unix/linux
```
### Linux Platform
```
dos2unix file_name

```
### Unix
```
To remove the ^M characters at the end of all lines in vi, use:
:%s/^V^M//g
The ^v is a CONTROL-V character and ^m is a CONTROL-M. When you type this, it will look like this:
:%s/^M//g

OR easy to use sed syntax to remove carriage return in Unix or Linux:
sed 's/\r$//' input > output
sed 's/\r$//g' input > output
# GNU/sed syntax
sed -i 's/\r$//g' input
```
### the ksh script to remove the ^M on AIX
```sh
#!/bin/ksh
for file in $(find /home/db2prod/sprint201604/ -type f); do
tr -d '\r' <$file>temp.$$ && mv temp.$$ $file
done
```

## Meaning of PS1, PS2, PS3 and PS4
### PS1 -- The default command line prompt
```
export PS1="$USERNAME@$HOSTNAME: $PWD >"
kingsley@oc7421025535.ibm.com: /worktmp/note/scripts/practice >
```

### PS2 -- Continuation interactive prompt
```
export PS2="==> "
echo "Hello, \
==> Wolrd"
```

### PS3 --  prompt used by "select" inside the shell script
```
PS3="Select an option and press Enter:"
select i in OS Host Filesystems Date Users Quit
...
```
Will show: `Select an option and press Enter:` on the select menu.

### PS4 --   Used by "set -x " to prefix tracing output
Default PS4 is "++"

### PROMPT_COMMAND
[kingsley@: /home/kingsley]$ export PROMPT_COMMAND="echo -n [$(date +%k:%m:%S)]"
[20:08:59][kingsley@: /home/kingsley]$ echo "hello"
hello
[20:08:59][kingsley@: /home/kingsley]$ 

## special dolar($) in linux
- $n, n is a positive integer number. stands for the position of the parameters
- $0, the script name
- $@, all positional parameters
- $*, all positional parameters
- $#, the number of the positional parameters
- $-, current options set for the shell
- $$, pid of the current shell
- $_, most recent parameter
- $?, the return status of a command
- $!, the PID of the most recent background command
    $ sleep 25&
    [1] 6237
    $ echo $!
    6237

### The difference between $@ and $*
```bash
$ set 'apple pie' pears peaches
$ for i in $*; do echo $i; done
apple
pie
pears
peaches

$ set 'apple pie' pears peaches
$ for i in $@; do echo $i; done
apple
pie
pears
peaches

$ for i in "$*"; do echo $i; done
apple pie pears peaches

$ for i in "$@"; do echo $i; done
apple pie
pears
peaches
```

### set command in shell script
The `set` command will reset or unset all of the positional parameters, for example, I have a script named `args.sh` contains following content
```
echo "The name of the script is: $0"
echo "all of the parameters are: $*"
set Mike Jordan Jake
echo "All of the parameter are: $*"
```
when the `args.sh` executed like `./args.sh a b c d`, the first "$*" will show "a b c d", but the second "$*" will change to "Mike Jordan Jake"

## How to check for null value
```
if [ "$name" == "" ]; # the same as [ ! "$name" ] or [ -z "$name" ]
   echo -e "\$name is null"
fi
orausr=`grep -w "oracle" /etc/passwd|awk -F: '{ print $1 }'`
if [ "X$orausr" != "X" ]; then
   echo "oracle user exist..."
else
   echo "oracle user does not exist..."
fi
```

## testing standard error messages STDERR
```
echo "This is an error message..." >&2
echo "This is normal output..."
# usage:
# ./file_descriptor will display all the lines
# ./file_descriptor 2>erro.out will display normal output to screen,
# the error message will be saved to erro.out file.

# below example will redirect the normal output to file1
# and the error output to file2
# ls -latr file_descriptor.ksh tttttttt 2>file2 1>file1

# silent execution of script
# ./file_descriptor.ksh >/Dev/null 2>&1

1>&2  #redirect output to where error is going
2>&1  #redirect error to where output is going
```
## Remote display setting on Linux Client and Server(VNC)
* 1. On server side, such as my vm 192.168.56.56
Modify `X11Forwarding yes` to `X11Forwarding no` on ***/etc/ssh/sshd_config***, it's ***/etc/ssh/sshd_config***, not ***ssh_config***

```
sed -i '/^X11Forwarding/s/yes/no/g' /etc/ssh/sshd_config
```
* 2. On ssh client, such as my laptop 192.168.56.1
Change the option of `ForwardX11` to `ForwardX11 yes` on ***/etc/ssh/ssh_config***, not ***sshd_config***

After that, when ssh to server end, the server end will auto setting DISPLAY to localhost:10.0

* 3. On Server end, modify below on `/etc/X11/xinit/xserverrc`

```
$ cat /etc/X11/xinit/xserverrc
#!/bin/sh
exec /usr/bin/X11/X -dpi 100
```


## Investigating high CPU on Linux
`ps -eo pid,user,pcpu,command --sort=-pcpu`



