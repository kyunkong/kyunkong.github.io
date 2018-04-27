---
title: "Tuning instance recovery"
date: 2018-01-30 16:44:25
---
[TOC]
## In DB2
在尝试activate前，可尝试调小`DB2_OVERRIDE_BPF`：
```
db2set DB2_OVERRIDE_BPF = 1,2000;2,1000 
```
以上命令调整bufferpoll ID1 为2000pages，ID2 bufferpool为1000 pages
## In Oracle
* `LOG_CHECKPOIN_INTERVAL`
    调大会加快crash recovery速度，但会影响事务性能.
    指定redo log blocks在上一次incremental checkpoin和最后一次写到redo log的块数。这个块指的是OS的块，而不是数据库的块。如果这个值设置为0， switch log的时候一定会发生checkpoint。
    Limits the number of redo blocks generated between the most recent redo record and the checkpoint. 
* `LOG_CHECKPOINT_TIMEOUT`
    Default 1800 sec, 这是在上一次进行incremental checkpoint后强制进行checkpoint的时间间隔
    Limits the number of seconds between the most recent redo record and the checkpoint.
* `FAST_START_MTTR_TARGET`
    此参数会被`LOG_CHECKPOINT_INTERVAL`覆盖, 此参数控制cache recovery的时间
    Lets you specify in seconds the expected "mean time to recover" (MTTR), which is the expected amount of time Oracle takes to perform recovery and startup the instance. 

* `V$INSTANCE_RECOVERY`
    此视图含有当前的参数配置，其中包括`TARGET_MTTR`和`ESTIMATED_MTTR`

