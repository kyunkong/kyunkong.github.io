---
title: "Oracle Wait Interface"
date: 2018-01-15 11:38:50
---

[TOC]

# OWI tools
1. OWI dynamic performance views
2. Other critical dynamic performance views
3. Extended SQL trace(10046 event)
4. oradebug & dump
5. AWR

## 1. OWI dynamic performance views overview
```bash
V$EVENT_NAME    : 提供所有OWI的信息，包括名字，P1，P2，P3等
V$SYSTEM_EVENT  : 实例启动以来所有seesion发生的OWI的累计信息
V$SESSION_EVENT : 当前已连接session的等待事件累计
v$session_wait  : 提供各session当前正在等待事件的信息
V$SESSION_WAIT_HISTORY
V$SYSTEM_WAIT_CLASS
V$SESSION_WAIT_CLASS
V$EVENT_HISTOGRAM

V$SESSION                                                 : session information
V$ACTIVE_SESSION_HISTORY                                  : session history information
V$PROCESS                                                 : process information
V$TRANSACTION                                             : transaction infor
V$LATCH, V$LATCH_PARENT, V$LATCH_CHILDREN, V$LATCH_HOLDER : latch infor
V$LOCK, V$LOCKED_OBJECT, V$ENQUEUE_LOCK                   : lock infor
V$SQL, V$SQLTEXT                                          : SQL infor
V$LIBRARYCACHE, X$KGLLK, X$KGLPN                          : library cache infor
V$ROWCACHE, V$ROWCACHE_PARENT                             : row cache infor
V$SGASTAT                                                 : SGA infor
V$SEGMENT_STATISTICS                                      : segment level statistics infor
V$SESSION_TIME_MODEL, V$SYS_TIME_MODEL                    : time model infor
X$BH, V$BH                                                : data buffer cache infor
```

## 2. Extended SQL trace
* 10046 event
```sql
alter session set events '10046 trace name context forever, level 12'
alter session set events '10046 trace name context forever off'
```

* oradebug
```sql
oradebug setospid 1235
oradebug unlimit
oradebug event 10046 trace name context forever, level 12
oradebug event 10046 trace name context forever off
oradebug tracefile_name
```

### 2.1 oradebug dump
With oradebug, SGA, PGA, library cache, row cache, buffers, enqueue, latch, heap, hanganylyze can be dumped.
```sql
oradebug setmypid
--for library cache dump
oradebug dump library_cache 10
--row cache dump
oradebug dump row_cache 10
--buffers
oradebug dump buffers 10
--enqueue dump
oradebug dump enqueues 10
--latch dump
oradebug dump latches 10
--headdump
oradebug dump headdump 2
--system state
oradebug dump systemstate 10
--process dump
oradebug setospid <PID>
oradebug dump processstate 10
```
### 2.2 oradebug control process
```sql
oradebug setospid <PID>
--suspend process
oradebug suspend
--resume process
oradebug resume
```

# 3. Latch
Latch是为了串行访问机制
## 3.1 有关latch的重要动态性能视图
```
v$latch
v$latch_parent
v$latch_children
重要栏位:
GETS             :   Willing-to-wait 模式下睡眠之前的latch请求成功次数
MISSES           :   Willing-to-wait 模式下睡眠之前获得latch的获得失败次数
SPIN_GETS        :   Willing-to-wait 模式下睡眠之前的spin阶段的获得成功次数
SLEEPS           :   Willing-to-wait 模式下睡眠次数
IMMEDIATE_GETS   :   No-wait 模式下latch获得成功次数
IMMEDIATE_MISSES :   No-wait 模式下latch获得失败次数
sleep1 ~ sleep4  :   1~3次的睡眠次数和4次以上的睡眠次数，10G后使用v$event_histogram替代
WAIT_TIME        :   为获得latch而等待的时间(ms)
```
如果出现latch的争用，高CPU不一定是由实际工作导致的，还有可能是在等待latch的时由于spin(_SPIN_COUND隐藏参数)产生的CPU消耗。但在CPU的系统上不执行spin。

## 3.2 Latch相关的等待事件
- latch: cache buffers chains
    欲在buffer cache搜索特定块的进程，需要获得cache buffers chains，出现争用，则出现此事件
- latch: cache buffers lru chain
    Buffer cache上欲搜索free buffer和dirty buffer的进程，需要获得cache buffers lru chain, 若发生争用，则出现此事件
- latch: shared pool
    Shared pool中欲分配到新的chunk进程， 需要获得shared pool latch， 若发生争用，则出现此事件
- latch: library cache
    欲搜索buffer cache的进程，需要获得library cache latch，若发生争用，则出现此事件
- latch: redo copy
    欲将DML引起的修改事项记录到redo buffer的进程，需要在工作之前获得redo copy latch，若发生争用，则出现次事件

## 3.4 Lock相关
| 模式 | 说明                                                   |
| ---- | ----                                                   |
| 0    | None                                                   |
| 1    | Null(N)                                                |
| 2    | Sub-Shared(SS)  or Row-Shared(RS)                      |
| 3    | Sub-Exclusive(SX) or Row-Exclusive(RX)                 |
| 4    | Shared(S)                                              |
| 5    | Shared-Sub-Exclusive(SSX) or Shared-Row-Exclusive(SRX) |
| 6    | Exclusive(X)                                           |

### 3.4.1 Enqueue相同关视图
```
--v$lock
SID     : 正拥有或者请求锁会话的SID，若LMODE> 0时， 表示正拥有锁，反正则正在请求锁
Type    : Enqueue的类型：TM， TX， UL，US， CI， TC等
ID1     : 资源ID1， 与v$event_name的P2对应
ID2     : 资源ID2， 与V$EVENT_NAME的P3对应
LMODE   : 正在拥有锁的模式
Request : 正请求锁的模式
CTIME   : 拥有或者请求锁的时间(s)
Block   : 锁的阻塞状态，1为阻塞，0为未阻塞
```

v$locked_object提供正在获取中的TX锁相关信息, 可以通过object_id观察相关的object，dba_waiters也能轻松了解block关系

### 3.4.2 普通锁相关的等待事件
TX锁引起的等待事件有：
- enq: TX - row lock contention
- enq: TX - allocate ITL entry
- enq: TX - index contention
- enq: TX - contention

若不是Enqueue锁，
```
row cache lock                             : row cache lock引起的竞争
buffer busy waits, read by another session : buffer lock引起的竞争
library cache lock                         : library cache lock引起的竞争
library cache pin                          : library cache pin引起的竞争
```


# 4. Oracle内部结构与OWI
x$bh视图
```
HLADDR  : 掌管相应缓冲区的latch地址，与 V$LATCH_CHILDREN.addr结合，获得所需的latch信息
TS#     : block的表空间信息，v$tablespace.ts#
DBARFIL : DBA中的文件编号
DBABLK  : DBA中的块编号
CLASS   : 块的类型
TCH     : Touch count
STATE   : 缓冲区的状态
   0 : Free, no valid block image
   1 : XCUR, a current mode block, exclusive to this instance
   2 : SCUR, a current mode block, shared  with other instance(RAC)
   3 : CR, a consistent read block image
   4 : READ, buffer is reserved for a block being read from disk
   5 : MREC, a block media recovery mode
   6 : IREC, a block crash recovery mode
   8 : PI, past image(only in RAC)
```

## 4.1 Shared Pool相关
与shared pool相关的等待事件：
- latch: library cache
- latch cache lock
- latch cache pin

### 4.1.1 SQL的执行的顺序跟shared pool的关系
1. 用户提交SQL， 首先检测语法语义权限等，下一步获得hash bucket的library cache latch， 然后确认share pool 上是否存在相同的SQL(soft parse), 若获得 library cache latch中发生争用，则出现 latch: library cache 事件， 若存在相同的SQL，眺至第6步直接执行
2. 若不存在相同的SQL， 在获得shared pool latch后，从空闲列表中查找大小合适的chunk， 若在获取shared pool latch发生争用，则出现latch: shared pool事件
3. 通过一系列算法分配或者查找合适的shared pool空间
4. 如果在分配shared pool时失败，返回ORA-04031
5. 创建执行计划，生成执行树的(2 ~ 5: Hard Parse)
6. Oracle对SQL Cursor以shared模式获得library cache lock和library cache pin, 并执行SQL。此过程如果发生争用，会出现library cache lock和library cache pin等待事件
7. Fetch阶段，SQL Cursor将library cache lock变为NULL模式，同时释放library cache pin.

# 5. 事务相关的等待事件
Oracle内部对事务的处理顺序：
1. 分配online的回滚段， 如果不能获取online的回滚端，会出现enq: US-contention事件，直到获取到为止
2. 分配到回滚段后，在回滚段头创建transaction table slot
3. 创建事务表后会生成TXID(transaction id), 再将此ID分配给事务, TXID通过v$transaction.xidsun, xidslot, xidsqn表现
4. 加载事务对象的数据块至buffer cache, 在块头的ITL(Interested Transaction List)上登记transaction entry, 在获取transaction entry时候，有可能出现enq: TX-allocate ITL entry事件
5. 将需要修改的块的修改信息存储到PGA，存储名为change vector. 修改一行时，分别创建change vector#1(撤销头块)，change vector#2(撤销块), change vector#3(数据块)相应的change vector，进程将PGA的改变向量复制到redo log buffer，在复制到redo log buffer过程中，需获得redo copy latch, redo allocation latch, redo writing latch, 因此可能会发生以下等待事件：latch:redo copy, latch:redo allocation, latch:redo writing
6. 将before image记录到undo block，继而修改数据块， 此时数据快变为脏数据快. 而且，buffer cache上创建关于已修改的数据快的consistent read块，如果修改的行正在被其他事务所改变，则发生等待事件enq:TX-row lock contention
7. commit后给transaction分配scn，commit information存储在redo buffer
8. 回滚段中的事务表存储已经成功提交的信息，释放资源
9. redo buffer被记录在redo log上，脏块被DBWE记录到data file中

# 6. 个别等待事件
## 6.1 cache buffers chains
- 低效的SQL
- Hot Block
在V$LATCH_CHILDREN中，比较cache buffers chains对应的child#和gets， sleeps值，以此判断特定子latch上的使用次数和争用是否集中，利用以下语句，获取sleeps次数搞的的子latch:
```
select * from
   (select child#, gets, sleeps from v$latch_children
   where name = 'cache buffers chains'
   order by 3 desc
   ) where rownum <=20;
```

若以上结果中特定的子latch的gets，sleeps值比其他的高很多，则表示有hot block. 另一种方法判断是否存在热快，从v$session_wait视图获得latch的地址，如在cache buffers chains，v$session_event.p1raw就是其地址，如果这个地址过多重复出现，就意味着相应的latch发生次数偏多，此时可解释为热块
适当的parallel query可以减少cache buffers chains，因为PQ不经过SGA，直接direct path read, 但这个方法不推荐为一般解决方案

## 6.2 cache buffers lru chain
此事件是在查看LRU/LRUW列表时发生，等待事件为latch:cache buffers lru chain:
1. 欲读取/修改的块在磁盘中，需要加载进内存，此时Oracle进程会先查询LRU表分配到合适的缓冲区，在此过程需要cache buffers lru chain.
2. DBWR为了将脏数据写入磁盘，查询LRUW列，将相应缓冲区移动到LRU列的过程中也要用到cache buffers lru chain latch.

## 6.3 buffer busy waits/read by other session/gc buffer busy
Oracle虽然使用行锁，但I/O是以块为最小单位，因此存在块单位锁。假设有两行数据存在同一个物理块中，R1，R2，各自有进程对他们进行更新，逻辑上这种并发没问题，由于物理限制，两个块的操作不能同时完成，在获得TX锁前，需要保障当前只有自己在修改块，此时需要获得的锁就叫buffer lock.
buffer busy waits的参数如下：
```
P1 :  绝对file#
P2 :  Block#
P3 :  v$waitclass
```
可以适当增加SGA大小以减少此类事件产生。对SQL进行调优.


# Scenario
Customer complained about performance are getting worse and worse after migration.

# Analysis
## 1. AWR
* DB time= 300 mins, Elapsed time = 60 mins, DB server loading is very heavy.

![load profile](/attach/oracle/load_profile.jpg)

* log file sync(normal case, the avg wait time is less then 10ms)in the top 5 event(along with "log file parallel write"), and most of the wait class in the top 5 wait event are "User I/O"
* top 5 event also shows "log buffer space", which means log buffer is too small

![top 10 events](/attach/oracle/top_10events.jpg)

* Log switch(derived), over 150 per/hour

![log switch](/attach/oracle/log_switch.jpg)



If "log file sync" and "log file parallel write" are approximate equal, we can consider the Disk I/O is poor
If "log file sync" is much higher than wait time for "log file parallel", means most of the time waiting is not due to slow I/O, and is most commonly contention caused by user over committing. We also can check the "Instance Activity Statistics" section of AWR, the expect value is 25 user calls/per commit
![user calls](/attach/oracle/user_calls.jpg)



## 2. Alert log and trace file
* alert log shows log file switch happens 5-10/per minute during the peek hours, and lots of "Checkpoint not complete" warning, if this happens, we can consider to increase the log file size
* lots of "Warning: log write elapsed time xxms, size xxKB" shows in the lgwr trace file, see MOS note 601316.1

## 3. Parameters
* log buffer is 2MB
* log file size is 50MB

## 4. Conclusion
What cause "log file sync":

1. Slow Disk I/O
2. High commit activity
3. High CPUs(Not in our case), if the 'r' column of vmstat output is higher than CPU numbers, we can consider excessive consuming
4. Bugs(Not in our case)

# Recommendation
1. Check with storage/hardware to see if there have any performance issue
2. Put the redo log file in the faster disk group, such as RAID1+0, don't put redo log in the RAID5 or SSD, see MOS note 857576.1
3. Increase the log buffer to 10MB
4. Increase the log file size to 512MB
5. Bulk commit in the SQL statements

# Explanation of log file sync
While user commit/rollback, commit will trigger log file sync, that is lgwr writing log buffer(memory) to log file(disk), before the commit/rollback completing, user will see the "log file sync" wait event.

For diagnosing log file sync issue, we can:
1. Check the AWR report
2. Check the alert log, alert log shows how frequently redo log file are switching, recommended time for redo log switching is 15 mins to 30 mins
3. As of 10.2.0.4, Oracle will write the warning message("log write elapsed time xxms, xxKB") to LGWR trace file, if the size is very small, we can consider the I/O is poor

