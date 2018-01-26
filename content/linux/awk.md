---
title: "AWK"
date: 2018-01-15 21:20:04
---

[TOC]

### awk with regular expression
```
awk '/^Mary/{printf "[%10s]\n", $1 }' database
```

### records and fields
```
$0       the whole record of tile
```

```
awk '{print $0}' database.txt
```

$n       n stands for integer decimal number, the field, if no comma specified, there will be no delimiter between the fields.
```
awk '{print $1, $2, $5 }' database
```
The comma is an internal variable of awk, known as output field separator(OFS), the default of OFS is space. When using print to print something, the comma will transform to space automatically.

```
$NR      $NR stands for record numbers
$NF      stands for the fields number
```


### the match operator(~)
```
awk '$1 ~ /[bB]illy/' file    #seeking for Billy/billy of field one
awk '$1 !~ /ly$/' file        #seeking for not end with ly of filed one
```


## awk built-in function
### sub & gsub functions
***Format***
```
sub(regrex, substitution string );
sub(regrex, substitution string, target string )
```
***Examples***
```
awk '{ sub(/Tom/, "Tommy"); print $1 }' datafile         #Everywhere the regrex Tom is found in the record($0), it will be replaced to Tommy
awk '{ sub(/Tom/, "Tommy", $2; print $1) }' datafile     #Everywhere the regrex Tom is found in the $2, it will be replaced to Tommy
```

### index function
The index function returns the first position where a string is found in a string.
***Format***
```
index(strings, substring)
```
***Examples***
```
echo "h" |awk '{print index("helloworld", "world")}'
6
```

### length function
The length function returns the length of the strings
***Format***
```
length(strings)
echo "hello" |awk '{ print length("hello") }'
5
```

### substr function
The substr returns the substring of a string starting at a position where the first position is one.
***Format***
```
substr(string, starting position)
substr(string, starting position, length of the string)
```
***Examples***
```
echo "hello" |awk '{ print substr("helloworld", 6, 5) }'
world
```



