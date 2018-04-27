---
title: "Typeset"
date: 2018-01-15 21:22:25
---
[TOC]

# Concept
`typeset` command setting the attributes of a variable, such as its case, width, and left or right justification. It's a permanent change.

## upper case(-u)  or lower case(-l)
```
$ typeset -u name="Kingsley"
$ echo $name
KINGSLEY

$ typeset -l name="Kingsley"
$ echo $name
kingsley
```

## Korn shell -- Left(right)-justified fixed-with 4 character field
```
$ name="kingsley"
$ typeset -R4 name
$ echo $name
king

$ name="kingsley"
$ typeset -R4 name
$ echo $name
sley
```

## Left-justified, zero-padded integer(Korn shell)
```
$ typeset -Z15 n
$ n=25
$ echo $n
000000000000025
```

## Left justify one lower/upper letter(Korn shell)
```
$ typeset -lL1 answer=YES
$ echo $answer
y

$ typeset -uL1 answer=no
$ echo $answer
N
```

## other uses of the `typeset` command

| Command                    | What is does                                                      |
| ----------                 | -----------------------------                                     |
| typeset                    | Display all the variables                                         |
| typeset -i num             | Will only accept integer values for num                           |
| typeset -x                 | Display exported variables                                        |
| typeset a b c              | If defined in the function, creates a, b, c to be local variables |
| typeset -r x=foo           | Set x=foo and makes it read only                                  |
| typeset -f [function_name] | List functions and their definations                              |
| typeset +f                 | List only function names                                          |
| unset -f name              | Unsets a function                                                 |

```bash
[kingsley@oc7421025535:/worktmp/note/scripts/practice]$ cat myfunc
# my function library
addem(){
    echo $(( $1 + $2 ))
    }

multem(){
    echo $(( $1 * $2 ))
    }

divem(){
    if [ $2 -ne 0 ]; then
        echo $(( $1 / $2 ))
    else
        echo -1
    fi
    }
[kingsley@oc7421025535:/worktmp/note/scripts/practice]$ . ./myfunc
[kingsley@oc7421025535:/worktmp/note/scripts/practice]$ typeset -f
addem(){
    echo $(( $1 + $2 ))
    }

divem(){
    if [ $2 -ne 0 ]; then
        echo $(( $1 / $2 ))
    else
        echo -1
    fi
    }
multem(){
    echo $(( $1 * $2 ))
    }

[kingsley@oc7421025535:/worktmp/note/scripts/practice]$ typeset +f
addem()
divem()
multem()
```






