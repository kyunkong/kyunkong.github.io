---
title: "Shell command line cheetsheet"
date: 2018-01-16 14:35:27
---
[TOC]
#shell bash/zsh command line key shortcuts --vi like
```
g~iw:   Change a word into upper case
vaw~:   The same effects
```
# Normal mode
```
s:      change a char
S:      change a line
w:      Forward a word
b:      Backwards a word
0:      Beginning of a line
$:      eol
v:      editor mode
C-w:    delete the word before the cursor(not include), also suitable for insert mode
C-u:    delete the line before the cursor(not include), also suitable for insert mode
C-k:    delete the line after the cursor(include cursor)
C-r:    search the command line history
C-t:    switch the position of the current char and previous char
C-p:    find the previous command history
C-n:    find the next command history
```



#Insert mode
***can refer to EMACS like***
```
C-h:    backspace
```


================================================================


# Shell(bash) command line key shortcuts --Emacs like
```
/* C means CTRL, A means ALT */
C-a: To the beginning of a line
C-e: To the end of a line
C-b: Back to one character
C-f: Forward to one character
C-w: Change a word before the cursor
C-u: Change a line before the cursor
C-y: Recover the characters that last C-u deleted
C-d: Delete the current character      # C-d equals to enter in my env
C-h: Backspace, delete a character forward
C-k: Change a line after(including) the current cursor
C-l: Clear the screen, equal the clear command
C-o: Equal enter,<CR>
C-j: Equal C-o
C-p: Equal up arrow key, find the previous command history
C-n: Equal down arrow key, find the next command history
C-?: Undo the last input
C-x: Jump between the current cursor and the beginning of the line*     # Not work in my env
C-t: Exchange the position between current cursor and the previous character
C-r: Search history commands
C-i: TAB, auto-completion
C-z: Pause the current command, "fg %job_number" and "bg %job_number" can bring the job back to foreground and background respectively
A-u: The word at the cursor to uppercase (from the cursor to the end of a word)
A-l: The word at the cursor to lowercase (from the cursor to the end of a word)
A-c: The word at the cursor to the first letter capitalized (from the cursor to the end of a word)
A-d: Delete the word after(including) the cursor
A-b: Forward a word
^  : Substitution last command. This substitution must be command only. ^old_cmd ^new_cmd

Ctrl+Alt+T: Open a terminal
Ctrl+Shift+N:  Open a new terminal
Ctrl+Shift+T:  Open a new tab
Ctrl+Shift+W:  Close a tab
Ctrl+Shift+Q:  Close terminal
Ctrl+Shift+C:  Copy
Ctr+Shift+V:   Paste
```


