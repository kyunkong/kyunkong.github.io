---
title: "VIM Tutorial"
date: 2018-01-15 17:58:06
---
[TOC]

### install vim from source code
- remove old vim version

```vim
sudo apt-get remove vim vim-runtime gvim
cd ~
git clone https://github.com/vim/vim.git
cd vim
./configure --with-features=huge --enable-multibyte --enable-rubyinterp --enable-pythoninterp --with-python-config-dir=/usr/lib/python2.7/config-x86_64-linux-gnu --enable-perlinterp --enable-luainterp --enable-gui=gtk2 --enable-cscope --enable-python3interp --prefix=/usr 
make VIMRUNTIMEDIR=/usr/share/vim/vim74
sudo make install
```

```vim
yum install -y python-devel python3-devel ruby-devel lua-devel libX11-devel gtk-devel gtk2-devel gtk3-devel ncurses-devel
git clone git@github.com:vim/vim.git
cd vim/
./configure --with-features=huge --enable-pythoninterp --enable-rubyinterp --enable-luainterp --enable-perlinterp --with-python-config-dir=/usr/lib/python2.7/config/ --enable-gui=gtk2 --enable-cscope --prefix=/usr
make
make install
sudo apt-get install libncurses5-dev libgnome2-dev libgnomeui-dev \
libgtk2.0-dev libatk1.0-dev libbonoboui2-dev \
libcairo2-dev libx11-dev libxpm-dev libxt-dev python-dev \
ruby-dev git

sudo yum install -y ruby ruby-devel lua lua-devel luajit \
    luajit-devel ctags git python python-devel \
    python3 python3-devel tcl-devel \
    perl perl-devel perl-ExtUtils-ParseXS \
    perl-ExtUtils-XSpp perl-ExtUtils-CBuilder \
    perl-ExtUtils-Embed



--enable-pythoninterp、--enable-rubyinterp、--enable-perlinterp、--enable-luainterp 
supprot python,ruby,perl,lua and so on. 
--enable-gui=gtk2
using gnome2 style gvim
--with-python-config-dir=/usr/lib/python2.7/config/ 
specify python location

```


#Import files need to be synced 
1. ~/.inputrc #define the vi-mode in command line
2. ~/.vimrc   #no need to explain 
3. ~/.bashrc  #user environments

#Common operations
x  delete character under the cursor (short for "dl")
X  delete character before the cursor (short for "dh")
D  delete from cursor to end of line (short for "d$")
dw delete from cursor to next start of word
db delete from cursor to previous start of word
diw   delete word under the cursor (excluding white space)
daw   delete word under the cursor (including white space)
dG delete until the end of the file
dgg   delete until the start of the file
L   move the cursor to the bottom of the screen
H   move the cursor to the top of the screen  
M   move the cursor to the middle of the screen
C-g move the screen down but remain the cursor
C-d down the cursor half of the screen a time
C-u up the cursor half of the screen a time

#find the vim history command 
q: find and edit it 

#Spell check in vim
: set spell
[s: jump to the mis-spell words
z=: fix mis-spell by auto-recommended words
1z=:    automatically fix mis-spell
noremap <leader>sp :normal! mm[s1z=`m<cr>

#exit buffers
:bd exit current buffer without saving
:bw exit current buffer with saving
:bn jump to next buffer 

#How to delete the blank of space at the end of line
g_   last character on line

we can use marcro to record the duplication of delete space end of line: 
this is a test line
this is another test line

qag_lDq 
@a

or just key mapping in normal mode
nmap <leader><space> g_lD


#For blogging tabular data
|C1       |  C2        | C3           |
|---------|------------|--------------|
|t1t1t1t1 |  tetetetet | tetetetetet  |
|ababab   |  dbdbdb    | cd cdcdcdcdcd|
    * Operations: Assume cursor located at first line 0 character
   > 1. yyp             Copy the first line and paste to below line
   > 2. V r-               Select the line which just pasted, and replace it to "-"
   > 3. k C-v 3j I|<Esc>       Back to prev line, change to the block visual mode, insert "|" 
   > 4. C-v 3j r|
   > 5. C-v 3j r|
   > 6. C-v 3j $A|            Move the first line end, append | to end

#Change column of test
li.one         a{ background-image: url('/images/sprite.png'); }
li.two         a{ background-image: url('/images/sprite.png'); }
li.three       a{ background-image: url('/images/sprite.png'); }
   * Operations: change all three images to contens
   > 1. Located in the begining of images word
   > 2. C-v 2je c<Esc>  and change to contents





# Change column in tag
<a href="#">one</a>
<a href="#">two</a>
<a href="#">three</a>
   * Operations: change the word to uppercase
   > gUit j. j.


# Folding manually
zfap    Manually folding the current cursor
zo Open the folding lines
zc Close the folding



# High frequency and efficiency
```
   > yy Copy a line 
   > yl Copy a charater
   > yw Copy a word, this action is copy the word from cursor to end of word, not an entir word
   > yaw Copy a whole word 
   > p Paste the copy content
    * cc Change a whole line, that is delete the line, and insert it.  
    * dd Delete a line 
    * c means change, d means delete, y measn yank. The usages of these three charater is the same. For example, there are something in the brackets, such as (some thing need to change). 
   > yi( Will copy the "some thing need to change". ci( will change the strings inside the brackets, and di( will delete the strings. 
    * Trigger matrics
    |Trigger|Effect|
    |----|----|
    |c   | Change|
    |d   | Delete|
    |y   | Yank into register|
    |g~  | Swap case|
    |gu  | Make lowercase|
    |gU  | Make uppercase|
    |>   | Shift right|
    |<   | Shift left|
    |=   | Autoindent|
    * . (dot) Repeat last change
    * I/i UPPERCASE insert before a line, but ignore space. For example, if I push I in this line, it will start insertion at the begining of star character. While i insert in front of the cursor. 
    * A/a Means append, UPPERCASE append append the end of line. While a insert the end of cursor. 
    * O/o Means OPEN, UPPERCASE open a new line above the current cursor position, while o open below of the line.  
    * S/s UPPERCASE S will delete the whole line and insert something in it, just equals cc triggers. equals cl. 
    * ; (semicolon) this trigger works when f find some charaters, press ; will find the next character. and press , will find the prev ones. Below is the example to explain ";" and ","
   > foo=arg1+arg2+arg3, and I want to add space before and behind the plus, such as arg1 + arg2 + arg3. 
   > Solution: f+ then sSPACE+SPACE then reture to normal mode, press ; find the next one, and then press . 
```

## Combination keyboard
* C-d Change the line from cursor to end of line

#Command mode
####Split window
:split split horizontal window, using Ctrl+w to jump between window. 
:close close the split window
:only close all other windows

:vsplit split vertical window 

###My vim window setting
**:sp filename**: open a file horizontal window
**:vsp filename**: open a file vertical window
**<c-h/j/k/l>**: Move the cursor to the other window
**<c-w>n+/-**: Adjust current window vertically, n stands for number
**<c-w>n>/<**: Adjust current window horizontal, n stands for number

###移动窗口
**<c-w>r**: 向右或者向下交换窗口, **<c-w>R**则相反

####Tab
**gt**: Switch the tab
**tabnew filename**: open a file in a new tab
**tabc**: Close current tab

####Modify default highlight filetype
set filetype=ruby

####Seeking for help
:help
If using help, you can jump to hyperlink by using Ctrl+], which means jump to tag, and back to the preceding place by Ctrl+O, O is capital alphabet. Type :help ctrl+O for more information

####Case sensitive in search mode
:set ignorecase
:set noignorecase

#Normal mode
####Join two lines together
J

####Change case
gUw changing the word into upcase behand the cursor
guw changing the word into lowercase behand the cursor


####redo 
ctrl+r

####undo and undo a line
u for undo, u for undo a line.

####moving to a character
fa search forward in the line for the single character a. fa backward in the line of a character.

####jump half a screen
ctrl+d scrolling down half a screen
ctrl+u scrolling up half a screen. these two commands with cursor company.

ctrl+e scrolling up once a line and keep the cursor stay in where you current are. 
ctrl+y vice verse 

ctrl+f pagedown 
ctrl+b pageup

zz put the current cursor in the center of the screen 
zt put the current cursor in the top 
zb put the current cursor at the bottom


####change/delete words/lines
dd delete the whold line
dw delete the whold word
cc change the whole line
d2w delete 2 end of words 
d2e delete 2 words
ci( change the words in the  round brackets
di( delete the words in the round breackets
D stands for d$
C stands for c$
s stands for cl, change one character
Ci stands for cc

#Insert mode


> finding what keys are mapping
:verbose map


###number item list
My first line
My second line
My third line
My fourth line
My fifth line
My sixth line
My seventh line
My eighth line
My ninth line

change above lines to 

1) My first line
2) My second line
3) My third line
4) My fourth line
5) My fifth line
6) My sixth line
7) My seventh line
8) My eighth line
9) My ninth line

using :
```
:let i=1
qa
I<C-r>=i<cr>) <Esc>
:let i += 1
q
jVG
:normal @a
```

###Edit the content of the Macro
1.put a Macro into the document from register 
:put a

2. yank the Macro from the Document to register
"add

