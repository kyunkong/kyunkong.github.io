---
title: "Printf function in shell script"
date: 2018-01-15 18:11:53
---
[TOC]

## printf conversion characters
```
c    Character
s    string
d    decimal number
ld   long decimal number
u    unsigned decimal number
lu   long unsigned decimal number
x    hexadecimal number
lx   long hexadecimal number
o    octal number
lo   long octal number
f    floating point number
e    floating point number in scientific notation
g    either %f or %e
```

## printf format specifiers
```
%c    print a single ASCII character
%d    print a decimal number
%e    print a e-notation of a number
%f    print a float number
%o    print the octal number
%x    print the hexadecimal number
%s    print the strings
```


### examples
```
$ echo "UNIX" | awk '{ printf "|%15s|\n", $1}'
|           UNIX|

the number of 15 means right justification, -15 means left justification.
```

