---
title: "Oracle Concept"
date: 2018-01-15 11:29:46
---

[TOC]

#1. 数据库是如何实现一致性的
* 事务一致性，即原子性，一个事务的所有操作要么都成功，要么都不成功。数据库通过先写日志的方式来保证此功能。如在事务处理过程中，遇到意外导致事务中断，通过在日志
中记录的顺序去回滚这些尚未提交的事务，如果已经提交但没有被写入磁盘的，此时也会进行前滚，把数据写入磁(持久性).

* Oracle 利用多版本一致性模型（multiversion consistency model），各种类型的锁及事务来管理多用户系统中的数据一致性（data consistency）。

##1.1 Oracle多版本并发控制
Oracle 能够自动地实现一个查询的读一致性，即一个查询所获得的数据来自同一时间点（single point in time）（这也被称为语句级读一致性（statement-level read consiste
ncy））。Oracle 还能令一个事务内的所有查询都具备读一致性（即事务级读一致性（transaction-level read consistency））。

Oracle 利用回滚段中的信息生成一个能保证一致性的数据视图。回滚段内保存了未提交或最近提交的事务中所修改数据的原值.

##1.2 Oracle UNDO机制
UNDO作用：
1、回滚，rollback时
2、构造CR块，提供读一致性
3、回滚，实例恢复的时候

#2. 事务隔离级别
##2.1 事务隔离级别及并发可能导致的问题
###2.1.1 并发事务需要解决的问题
* 1.1 更新丢失 lost update
    - 第一类更新丢失： 回滚覆盖，撤销一个事务，该事务内的写操作要回滚，把其他已提交事务写入的数据覆盖了。
    - 第二类更新丢失： 提交覆盖，一个事务提交，写操作依赖于事务内读到的数据，读发生在其他事务提交前，写发生在其他事务提交后，把其他已经提交的事务写入的数据覆
盖了。这也是不可重复读的特例。
* 1.2 脏读 dirty read
    一个事务读到了其他事务没提交的数据。
* 1.3 不可重复读 non-repeatable-read
    一个事务中，两次读到同一行数据不一致。
* 1.4 幻读 phantom read
    一个事务中两次查询，结果差了或者多了几行/几列

###2.1.2 事务隔离级别
* 2.1 Read Uncommitted, 读未提交：不允许第一类更新丢失，允许脏读，不隔离事务。
* 2.2 Read Committed, 读已提交： 不允许脏读，允许不可重复读。
* 2.3 Repeatable Read, 可重复读： 不允许不可重复读，但可能出现幻读。
* 2.4 Serializable, 串行化， 所有DML串行化

事务隔离级别是为了解决并发过程中遇到的脏读，不可重复读和幻读。


#3. Oracle事务锁机制
在数据库中有两种基本的锁类型：排它锁（Exclusive Locks，即X锁）和共享锁（Share Locks，即S锁）。当数据对象被加上排它锁时，其他的事务不能对它读取和修改。加了共享
锁的数据对象可以被其他事务读取，但不能修改。数据库利用这两种基本的锁类型来对数据库的事务进行并发控制。
```
LMODE取值0,1,2,3,4,5,6, 数字越大锁级别越高, 影响的操作越多。
1级锁：
Select，有时会在v$locked_object出现。
2级锁即RS锁
相应的sql有：Select for update ,Lock xxx in  Row Share mode，select for update当对
话使用for update子串打开一个游标时，所有返回集中的数据行都将处于行级(Row-X)独
占式锁定，其他对象只能查询这些数据行，不能进行update、delete或select for update 操作。
3级锁即RX锁
相应的sql有：Insert, Update, Delete, Lock xxx in Row Exclusive mode，没有commit
之前插入同样的一条记录会没有反应, 因为后一个3的锁会一直等待上一个3的锁, 我们
必须释放掉上一个才能继续工作。
4级锁即S锁
相应的sql有：Create Index, Lock xxx in Share mode
5级锁即SRX锁
相应的sql有：Lock xxx in Share Row Exclusive mode，当有主外键约束时update/delete ... ; 可能会产生4,5的锁。
6级锁即X锁
相应的sql有：Alter table, Drop table, Drop Index, Truncate table, Lock xxx in Exclusive mode
```
##3.1 DML lock:保护数据的完整性
* TX: Transaction eXclusive
    在Oracle的每行数据上，都有一个标志位来表示该行数据是否被锁定。Oracle不像DB2那样，建立一个链表来维护每一行被加锁的数据，这样就大大减小了行级锁的维护开销，
也在很大程度上避免了类似DB2使用行级锁时经常发生的锁数量不够而进行锁升级的情况。数据行上的锁标志一旦被置位，就表明该行数据被加X锁，Oracle在数据行上没有S锁。
* TM: DML Enqueue, 保证表结构不被改变
Oracle数据库的一个显著特点是，在缺省情况下，单纯地读数据（SELECT）并不加锁，Oracle通过回滚段（Rollback segment）来保证用户不读"脏"数据。这些都提高了系统的并发
程度。

##3.2 DDL lock:保护数据库对象的结构

#4. 多表连接方式
##4.1 Nested loop
访问次数：驱动表返回几条，被驱动表访问多次
驱动顺序：有
排序差别：无须排序
限制场景：无
索引限制：驱动表限制条件有索引，被驱动表连接条件有索引
##4.2 Hash join
访问次数：最多被访问一次
驱动顺序：有
排序差别：无，但是消耗内存的hash_area_size
限制场景：只能等值查询
索引限制：索引列在表连接中无特殊要求，跟单表情况一样
##4.3 Merge Sorted Join
访问次数：最多被访问一次
驱动顺序：无
排序差别：需要排序sort_area_size
限制场景：连接条件为< > like导致无法使用
索引限制：索引消除排序

#5. DataGuard
##5.1 Max Protection
当事务提交时，日志必须同时写到primary和至少一个standby，以保证数据实时同步。但：1.会影响primary性能；2.当日志无法同步到standby，primary会被强制shutdown
##5.2 Max Availability
正常情况下，跟Max Protection一样，但当日志无法同步到standby时，会以最大性能模式工作，当所有日志同步到standby时候，又切换回最大保护模式。
##5.3 Max Performance
当primary日志无法传递到standby的时候，primary不受影响，且会不断尝试向standby传输日志，直至成功。保证了primary的性能，但存在数据丢失的风险。可尝试部署监控脚本
定时检查GAP，以减少数据丢失的风险


#6. RMAN完全恢复和不完全恢复
完全恢复一般用于物理介质故障，如介质失败，磁盘或者数据文件故障；不完全恢复一般用于逻辑业务故障，如用户误删除关键业务数据，或者用于日志不全的情况下的恢复，比如
丢失数据文件，要做完全恢复，发现部分归档已经无法使用。此时只能恢复到可用归档日志的那一刻时间点。通过RMAN完全恢复，可以将数据库恢复到失败点状态；通过RMAN不完全
恢复，可以将数据库恢复到备份点与失败点之间某个时刻的状态。

完全恢复意味着你能恢复所有在失败前已提交的事务。因此，完全恢复是不需要将所有数据文件都还原并且恢复的。我们需要做的只是恢复那些损坏了的数据文件。Oracle通过对比
控制文件中的SCN值和dafatile header中的SCN值来确定哪些数据文件需要被恢复。

不完全恢复意味着不是所有已提交事务都能进行恢复，所以肯定丢失部分数据。不完全恢复一般用在指定时间点的恢复，由于误操作而导致的数据丢失，可以采用不完全恢复至数据
删除前一刻。不完全恢复是通过指定recover database until命令进行的。所以不完全恢复也被称之为基于时间点的恢复(database point-in-time recovery)。

#7. RAC缓存融合
Cache Fusion 就是通过互联网络（高速的 Private interconnect）在集群内各节点的 SGA 之间进行块传递，这是RAC最核心的工作机制，他把所有实例的SGA虚拟成一个大的SGA区
，每当不同的实例请求相同的数据块时，这个数据块就通过 Private interconnect 在实例间进行传递。


#8. v$sql v$sqlarea
* v$sql是从parent cursor级别统计sql的执行信息
* v$sqlarea是从child cursor级别统计sql的执行信息



