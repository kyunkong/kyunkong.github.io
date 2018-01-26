---
title: "Redo log files management"
date: 2018-01-18 10:55:52
---
[TOC]

## Displaying online redo log information
```
select
a.group#,a.member,b.status,b.archived,bytes/1024/1024
mbytes
from v$logfile a, v$log b
where a.group# = b.group#
order by 1,2;
```

-- output
```
    GROUP# MEMBER                                                                           STATUS    ARCHIVED     MBYTES
---------- -------------------------------------------------------------------------------- --------- -------- ----------
         1 /u01/app/oracle/oradata/LINORA/onlinelog/o1_mf_1_chkysp87_.log                   CURRENT   NO               50
         2 /u01/app/oracle/oradata/LINORA/onlinelog/o1_mf_2_chkyspbq_.log                   INACTIVE  YES              50
         3 /u01/app/oracle/oradata/LINORA/onlinelog/o1_mf_3_chkyspd6_.log                   INACTIVE  YES              50
```

## Status for Online Redo-Log Groups in the V$LOG View

| status           | Meaning                                                                     |
| ----             | ----                                                                        |
| Current          | 当前活动日志                                                                |
| Active           | Crash recovery所需日志，可能已归档或者尚未归档                              |
| Clearing         | 'alter database clear logfile'命令正在clear log                             |
| Clearing_current | The current log group is being cleared of a closed CLEARING_CURRENT thread. |
| INACTIVE         | Crash recovery不需要用到此日志，可能已归档或尚未归档                        |
| UNUSED           | 最近新建尚未使用的日志                                                      |

## Determining the Optimal Size of Online Redo-Log Groups
```
-- Purpose: Showing the log switch frequency
-- name: log_frequency.sql
-- from:
COL DAY FORMAT a15;
COL HOUR FORMAT a4;
COL TOTAL FORMAT 999;
SELECT TO_CHAR(FIRST_TIME,'YYYY-MM-DD') DAY,
TO_CHAR(FIRST_TIME,'HH24') HOUR,
COUNT(*) TOTAL
FROM V$LOG_HISTORY
GROUP BY TO_CHAR(FIRST_TIME,'YYYY-MM-DD'),TO_CHAR(FIRST_TIME,'HH24')
ORDER BY TO_CHAR(FIRST_TIME,'YYYY-MM-DD'),TO_CHAR(FIRST_TIME,'HH24')
ASC;
```

### `archive_lag_target` parameter
`archive_lag_target` 指定时间定时去进行log switch，在dataguard中可以调整此参数，使得归档日志可以及时传送到standby中。同时，结合`v$instance_recovery`中`optimal_logfile_size`可以参考redo log大小设置是否合适, Oracle官方建议redo log大小至少要跟`OPTIMAL_LOGFILE_SIZE`相当。

### Checkpoint not complete warning
告警日志发现这个问题，可以考虑:

* 增加日志组, 最容易实现，受限于控制文件中的`MAXLOGFILES`，增加的日志文件要和原来大小一致
* 降低FAST_START_MTTR_TARGET
* 增加DB_WRITER_PROCESSES

## Adding Online Redo-Log Groups
* OMF

```
alter database add logfile group 4 size 100M;
```
* Non-OMF

```
alter database add logfile group 4
('/u01/app/oracle/oradata/LINORA/onlinelog/redo04a.rdo',
'/u01/app/oracle/oradata/LINORA/onlinelog/redo04b.rdo'
)  size 100M;
```

***TIPS: redo log不能像data file一样可以直接在线扩充，它只能通过增加group，然后切换，最后删除原有日志组。***
大致步骤：

* add online redo log group
* alter system switch logfile
* alter system checkpoint
* drop old online redo group once confirm it's archived and inactive
    - alter database drop logfile group 1;
* alter database backup controlfile to trace resetlogs





