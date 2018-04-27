---
title: "python"
date: 2018-01-17 12:19:56
---
[TOC]
## Upgrade python2.6 to python2.7 without disrupting existing one
download latest python2.7 version source
```
yum install openssl-devel
yum install zlib-devel
tar xvf Python-2.7.13.tar.xz
cd Python-2.7.13
./configure --enable-shared
make
make altinstall
```
**Make a link to python library path**
```bash
#On 64 bit OS
# ldd `python2.7` to check the dependency
ln -s /usr/local/lib/libpython2.7.so.1.0 /usr/lib64/libpython2.7.so.1.0
#for python3.6
ln -s /usr/local/lib/libpython3.6m.so.1.0 /usr/lib64/libpython3.6m.so.1.0
```

**Install pip from get-pip.py**
```
curl  https://bootstrap.pypa.io/get-pip.py
python2.7 get-pip.py
```

https://ruiaylin.github.io/2014/12/12/python%20update/
https://github.com/h2oai/h2o-2/wiki/installing-python-2.7-on-centos-6.3.-follow-this-sequence-exactly-for-centos-machine-only
http://www.javavirtues.com/2016/12/installing-python-on-linux-without.html




##How to upgrade python in Centos/RHEL 6

- Download the source  code 
- Configure it 
``` bash
cd source_dir
.configure
```
- Make && make install 

``` bash
make && make install
```

Because yum in RHEL6 is depend on python2.6, so still need to change the head file in yum configuration, so do not remove old python version 

``` bash
mv /usr/bin/python /usr/bin/python2.6.6
ln -s /usr/local/bin/python2.7 /usr/bin/python
```

``` bash
vim /usr/bin/yum
#subtitute python to python2.6.6 in sha-bang
!/usr/bin/python2.6.6
```

##install pip in ubuntu
- install python2 pip
```
sudo apt install python-pip
```
- install python3 pip
```
wget https://bootstrap.pypa.io/get-pip.py
sudo python3 get-pip.py
```

## sys.exit() code on python:
```
/usr/include/asm-generic/errno.h
```


##ipython reload/auto-load customize module
On the ipython interpreter, if you want to import your owner module every time after ipython started up, and want to reload the module after you modified the module without re-open the ipython,  just do some configurations on the ipython profile.
* 1. Create the ipython profile
```
    ipython profile create
```
* 2. Edit the profile
```
    vi /home/fung/.ipython/profile_default/ipython_config.py
    c.InteractiveShellApp.extensions = ['autoreload']                      #for auto reload module
    c.InteractiveShellApp.exec_lines = ['%autoreload 2']                   #for auto reload module
    c.InteractiveShellApp.exec_lines = ['from tutorial.myModule import *'] #for auto import the module
```
* 3. Ipython command line to reload a module
```
    %load_ext autoreload
    %autoreload 2
```

##Function source code
```
In [34]: p99??
Signature: p99()
Source:   
def p99():
    for i in range(1,10):
        for j in range(1, i+1):
            print('{}*{}={}\t'.format(i,j,i*j), end='')
        print()
File:      ~/workspace/tutorial/myModule.py
Type:      function
```

##Python tutorial
https://pythonspot.com/en/


##comprehensions syntax
```python
def test(n):
   return [ i * i for i in range(1, N+1)]

"""
result:
In [28]: power3(10)
Out[28]: [1, 4, 9, 16, 25, 36, 49, 64, 81, 100]
"""

[ k*k for k in range(1, n+1) ]    #list comprehensions
( k*k for k in range(1, n+1) )    #generator comprehensions
{ k*k for k in range(1, n+1) }    #set comprehensions
{ k: k*k for k in range(1, n+1) }   #dict comprehensions
```


##Help from python
```
#1. from command line:
pydoc Module_name

#2. from intepreter:
Module_name.__doc__
or
help(Module_name)
or
print(sys.__doc__)
```

## File operations
```
f = open('myfile', 'w')       #open file 'myfile' for write
f.write('This is the first line\n')       #write a line to the file
f.close()                     #Close the file

f = open('myfile', 'r')        #open file for read
f.read()                      #read the whole file
f.readlines()                 # read entire file at once
f.readline()                  # read a line
f.close

f = open('myfile', 'a')       #append a file
f.write("this is second line\n")
f.close()
```

##Catching exception
```
def division(a, b):
   try:
      return (a // b)
   except ZeroDivisionError:
      print("Non-zero number needed")
```


## try-finally
```
def test(file):
   try:
      fin = open('file', 'W')
   finally:
      fin.close()
   except IOError:
      print("No such file or directory")
```


##Raising exception
```
def fun1(a, b):
   if b > a:
      raise Exception("Invalid number:", b)
```

## list method return values
tuples are similar with list, except that once tuple is created, you can not 
modify it by adding, deleting, replacing or reordering items
```
list = [1, 3, 5, 7, 9]
list.append(11)   #append 11 to the list
list.count(3)     #List how many times the 3 number in the list
list.extend(list2) #merge list2 to the list
list.index(1)        #return 0; return the index of the first occurrence of element x in the list
list.insert(0, 12)   #insert 12 to the first position of the list
list.remove(x)       #remove the first occurrence of element x and return None
list.reverse()       #reverse the list
list.pop(x)          #remove the element at the given position and return it
list.sort()          #sort the list and return none
```


