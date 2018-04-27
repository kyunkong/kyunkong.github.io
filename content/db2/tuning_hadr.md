---
title: "Tuning HADR"
date: 2018-01-31 12:09:46
draft: true
---
[TOC]
## synchronization mode
### SYNC
主备节点是完全同步状态，主节点一个事务的完成必须等待日志传输到备节点，且备节点完成apply。

### NEARSYNC(default)
当主节点接收到备节点已经收到log buffer的信息就可以提交一个事务

### ASYNC
主节点提交事务，同时发送log buffer到备节点，主节点并不需要等待备节点接收到log buffer

### SUPERSYNC
事务日志的写入和传输完全是独立的，在此模式下，永远都不会存在peer模式，一直都是remote catchup状态

## HADR related EDUs
### On primary
* db2hadrp
    HADR主线程，职责为log shipping及接收standby的确认信息
* db2loggw
    Log writer
* db2lfr
    在supersync模式下读取日志文件的线程

### On standby
* db2hadrs
    HADR在standby上的主线程，负责接收日志和发送确认信息，同时将redo record写入standby日志
* db2shred
    从db2hadrs接收到log buffer，db2shred会将这些log buffer分成单独的redo record以应用到表空间
* db2redom
    Redo Master， 从db2shred接收redo record并管理这些log record
* db2redow
    Redo Worker, 这进程由db2redom指派，重演log record到表空间
* db2lfr
    log file reader, 有时候需要从日志文件中读取旧的日志记录，并分发这些记录到db2shred

## Tools for debug HADR

## Tools for monitoring HADR

## Performance tuning for HADR

