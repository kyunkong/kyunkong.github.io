---
title: "DB2 performance tuning SQLs"
date: 2018-01-16 16:58:20
---
[TOC]

## Monitoring performance with SQL -- buffer pools
- showing buffer pools hitratios
```
db2 "
select substr(bp_name,1,30) as bp_name ,
data_hit_ratio_percent,
index_hit_ratio_percent ,
total_hit_ratio_percent
from sysibmadm.bp_hitratio
where bp_name not like 'IBMSYSTEM%'
"
```

## Monitoring performance with SQL -- Sorts
```
select sheapthres_shr as "Shared_sort_heap" , \
sort_shrheap_allocated as "Shared_sort_allocated" , \
dec((100 * sort_shrheap_allocated)/sheapthres_shr,5,2) \
as " % Sheap_alloc" , \
dec((100* sort_shrheap_top)/sheapthres_shr,5,2) \
as " % Max Sheap_alloc" , \
sort_overflows as "Sort_Overflows", \
active_sorts as "Active_Sorts" \
from (select int(value) as sheapthres_shr from sysibmadm.dbcfg where name='sheapthres_shr') as a, sysibmadm.snapdb
```

```
db2 "
with dbcfg1 as
(select int(value) as sheapthres_shr from sysibmadm.dbcfg where name='sheapthres_shr')
select sheapthres_shr as "Shared_sort_heap" ,
sort_shrheap_allocated as "Shared_sort_allocated" ,
dec((100 * sort_shrheap_allocated)/sheapthres_shr,5,2) as "Sheap_alloc_Percentage" ,
dec((100* sort_shrheap_top)/sheapthres_shr,5,2) as "Max_Sheap_alloc_Percentage" ,
sort_overflows as "Sort_Overflows",
active_sorts as "Active_Sorts"
from dbcfg1, sysibmadm.snapdb
"
```

## Monitoring performance with SQL -- package cache
```
db2 "
with dbcfg1 as
(select int(value) as pckcachesz from sysibmadm.dbcfg where name='pckcachesz')
select pckcachesz as "Package_cache_size",
pkg_cache_lookups as "Lookups",
pkg_cache_inserts as "Inserts",
pkg_cache_num_overflows as "Overflows",
100 * pkg_cache_size_top/(pckcachesz*4096) as "PKG_cache_alloc_percent"
from dbcfg1, sysibmadm.snapdb
"
```

## Top dynamic SQLs
- order by number of executions
```
db2 "
select NUM_EXECUTIONS as "Num_Execs",
AVERAGE_EXECUTION_TIME_S as "Avg_Time_sec",
STMT_SORTS,
SORTS_PER_EXECUTION,
substr(stmt_text,1,150) as "SQL_STMT"
from sysibmadm.top_dynamic_sql
where NUM_EXECUTIONS>0
order by 1 desc
fetch first 10 rows only
"
```

```
select num_executions as "Num Execs", average_execution_time_s as "Avg Time (sec)",
stmt_sorts as "Num Sorts", sorts_per_execution as "Sorts Per Stmt",
substr(stmt_text,1,35) as "SQL Stmt" from sysibmadm.top_dynamic_sql where
num_executions > 0
order by 1 desc
fetch first 5 rows only;
```


- order by AVERAGE_EXECUTION_TIME_S
```
db2 "
select NUM_EXECUTIONS as "Num_Execs",
AVERAGE_EXECUTION_TIME_S as "Avg_Time_sec",
STMT_SORTS,
SORTS_PER_EXECUTION,
substr(stmt_text,1,150) as "SQL_STMT"
from sysibmadm.top_dynamic_sql
where NUM_EXECUTIONS>0
order by 2 desc
fetch first 10 rows only
"
```

```
select num_executions as "Num Execs", average_execution_time_s as "Avg Time (sec)",
stmt_sorts as "Num Sorts", sorts_per_execution as "Sorts Per Stmt",
substr(stmt_text,1,35) as "SQL Stmt" from sysibmadm.top_dynamic_sql where
num_executions > 0
order by 2 desc
fetch first 5 rows only;
```


## Long running SQL
```
db2 "
select substr(appl_name, 1, 20) as Appl_Name,
ELAPSED_TIME_MIN, APPL_STATUS,
substr(AUTHID, 1, 10) as auth_id,
substr(INBOUND_COMM_ADDRESS, 1, 15) as IP_ADDR,
substr(stmt_text,1, 50) as SQL_stmt
from sysibmadm.long_running_sql
order by 2 desc
"
```

## Lock wait time
```
db2 "
select substr(ai.appl_name, 1, 20) as appl_name,
substr(ai.primary_auth_id, 1, 10) as auth_id,
ap.lock_waits as lockwaits,
ap.lock_wait_time/1000 as Total_wait,
(ap.lock_wait_time/ap.lock_waits) as Avg_wait
from sysibmadm.snapappl_info ai, sysibmadm.snapappl ap
where ai.agent_id=ap.agent_id and ap.lock_waits>0
"
```

## Lock chains
```
db2 "
select substr(ai_h.appl_name, 1, 10) as "Hold_appl",
substr(ai_h.primary_auth_id,1,10) as "Holder",
substr(lw.authid, 1, 10) as Waiter,
substr(lw.appl_name, 1, 10) as Wait_AppL,
lw.lock_mode,
lw.lock_object_type,
substr(lw.tabname, 1, 10) as tabname,
substr(lw.tabschema, 1, 10) as tabschema,
timestampdiff(2, char(lw.snapshot_timestamp-lw.lock_wait_start_time)) as waitings
from sysibmadm.lockwaits lw,sysibmadm.snapappl_info ai_h
where lw.agent_id_holding_lk=ai_h.agent_id
"
```

```
select substr(ai_h.appl_name,1,10) as "Hold App",
substr(ai_h.primary_auth_id,1,10) as "Holder",
substr(lw.appl_name,1,10) as "Wait App",
substr(lw.authid,1,10) as "Waiter",
lw.lock_mode ,
lw.lock_object_type ,
substr(lw.tabname,1,10) as "TabName",
substr(lw.tabschema,1,10) as "Schema",
timestampdiff(2,char(lw.snapshot_timestamp - lw.lock_wait_start_time))
as "waiting (s)"
from
sysibmadm.lockwaits lw ,
sysibmadm.snapappl_info ai_h
where
lw.agent_id_holding_lk = ai_h.agent_id ;
```


## Lock memory usage
```
db2 "
with dbcfg1 as
( select float(int(value) * 4096) as locklist
from sysibmadm.dbcfg where name = 'locklist' ) ,
dbcfg2 as ( select float(int(value)) as maxlocks from sysibmadm.dbcfg where name = 'maxlocks' )
select dec((lock_list_in_use/locklist)*100,4,1) as "Lock_List_Percent",
dec((lock_list_in_use/(locklist*(maxlocks/100))*100),4,1) as "Maxlockpercent",
appls_cur_cons as "Number_of_Cons",
lock_list_in_use/appls_cur_cons as "Avg_Lock_Mem_Per_Con_bytes"
from dbcfg1, dbcfg2 , sysibmadm.snapdb
"
```

```
with dbcfg1 as
( select float(int(value) * 4096) as locklist from sysibmadm.dbcfg where name =
'locklist' ) ,
dbcfg2 as
( select float(int(value)) as maxlocks from sysibmadm.dbcfg where name = 'maxlocks'
)
select dec((lock_list_in_use/locklist)*100,4,1) as "% Lock List",
dec((lock_list_in_use/(locklist*(maxlocks/100))*100),4,1)
as "% to Maxlock",
appls_cur_cons as "Number of Cons",
lock_list_in_use/appls_cur_cons as "Avg Lock Mem Per Con (bytes)"
from dbcfg1, dbcfg2 , sysibmadm.snapdb ;
```


## Lock escalation, deadlocks and timeouts
```
db2 "
select substr(ai.appl_name,1,10) as Application,
substr(ai.primary_auth_id,1,10) as AuthID,
int(ap.locks_held) as "Locks_held",
int(ap.lock_escals) as "Escalations",
int(ap.lock_timeouts) as "Lock_Timeouts",
int(ap.deadlocks) as "Deadlocks",
int(ap.int_deadlock_rollbacks) as "Dlock_Victim",
substr(inbound_comm_address,1,15) as "IP_Addr"
from
sysibmadm.snapappl ap,
sysibmadm.snapappl_info ai
where ap.agent_id = ai.agent_id
"
```

```
select substr(ai.appl_name,1,10) as Application,
substr(ai.primary_auth_id,1,10) as AuthID,
int(ap.locks_held) as "# Locks",
int(ap.lock_escals) as "Escalations",
int(ap.lock_timeouts) as "Lock Timeouts",
int(ap.deadlocks) as "Deadlocks",
int(ap.int_deadlock_rollbacks) as "Dlock Victim",
substr(inbound_comm_address,1,15) as "IP Address"
from
sysibmadm.snapappl ap, sysibmadm.snapappl_info ai
where ap.agent_id = ai.agent_id ;
```


## Costly table scan
***High rows read to rows selected ratio indicates table scans***
```
db2 "
select substr(authid,1,10) as authid,
substr(appl_name,1,20) as appl_name ,
AGENT_ID,
percent_rows_selected
from sysibmadm.appl_performance
"
```

## Prefetchers too aggressive or not aggressive enough
***If too many pages are read asynchronously from disk into the buffer pool and no application ever accesses those pages, then the prefetching is a waste.***
```
db2 "
with bp_snap as (
select substr(bp_name,1,30) as bp_name ,
unread_prefetch_pages ,
pool_async_data_reads + pool_async_index_reads as Async_Reads ,
pool_data_p_reads + pool_index_p_reads + pool_temp_data_p_reads + pool_temp_index_p_reads as Total_Reads
from sysibmadm.snapbp
where bp_name not like 'IBMSYSTEM%'
)
select bp_name, unread_prefetch_pages,
dec ( 100 * ( Total_reads - Async_reads ) / Total_reads ,5,2) as "Synch_Reads_Percent",
dec ( 100 * unread_prefetch_pages / Total_reads ,5,2) as "Unread_Pages_Percent"
from bp_snap
"
```

## Database memory usage
```
db2 "
select pool_id, pool_secondary_id, pool_cur_size, pool_watermark
from sysibmadm.snapdb_memory_pool
"
```

## Log space usage
```
db2 "
select int(total_log_used/1024/1024) as "Log_Used_Meg",
int(total_log_available/1024/1024) as "Log_Space_Free_Meg",
int((float(total_log_used) / float(total_log_used+total_log_available))*100) as "Pct_Used",
int(tot_log_used_top/1024/1024) as "Max_Log_Used_Meg",
int(sec_log_used_top/1024/1024) as "Max_Sec_Used_Meg",
int(sec_logs_allocated) as "Secondaries"
from sysibmadm.snapdb
"
```

```
select
int(total_log_used/1024/1024) as "Log Used (Meg)",
int(total_log_available/1024/1024)
as "Log Space Free (Meg)",
int((float(total_log_used) /
float(total_log_used+total_log_available))*100)
as "Pct Used",
int(tot_log_used_top/1024/1024) as "Max Log Used (Meg)",
int(sec_log_used_top/1024/1024) as "Max Sec. Used (Meg)",
int(sec_logs_allocated) as "Secondaries"
from sysibmadm.snapdb ;
```


```
SELECT
TOTAL_LOG_USED AS LOG_USED,
TOTAL_LOG_AVAILABLE as LOG_REMAINING ,
SEC_LOGS_ALLOCATED AS SEC_LOGFILES
FROM SYSIBMADM.SNAPDB ;
```


## Oldest transaction holding log space
```
db2 "
select substr(ai.appl_status,1,20) as "Status",
substr(ai.primary_auth_id,1,10) as "Authid",
substr(ai.appl_name,1,15) as "Appl_Name",
int(ap.UOW_LOG_SPACE_USED/1024/1024) as "Log_Used_Mge",
int(ap.appl_idle_time/60) as "Idle_for_min",
ap.appl_con_time as "Connected_Since"
from sysibmadm.snapdb db,
sysibmadm.snapappl ap,
sysibmadm.snapappl_info ai
where ai.agent_id = db.APPL_ID_OLDEST_XACT
and ap.agent_id = ai.agent_id
"
```

## Tablespace size
```
db2 "
select substr(tbsp_name,1,30) as "tbsname",
tbsp_type as "Type",
substr(tbsp_state,1,20) as "Status",
(tbsp_total_size_kb / 1024 ) as "Size_in_Meg",
smallint((float(tbsp_free_size_kb) / float(tbsp_total_size_kb))*100) as "Free_Space_in_percent",
int((tbsp_free_size_kb) / 1024 ) as "Free_Space_in_Meg",
from sysibmadm.tbsp_utilization
"
```

```
select substr(tbsp_name,1,30) as "Tablespace Name", tbsp_type as
"Type",
substr(tbsp_state,1,20) as "Status",
(tbsp_total_size_kb / 1024 ) as "Size (Meg)",
smallint((float(tbsp_free_size_kb) /
float(tbsp_total_size_kb))*100)
as "% Free Space",
int((tbsp_free_size_kb) / 1024 )
as "Meg Free Space"
from sysibmadm.tbsp_utilization
;
```
## DB storage path
```
db2 "
select substr(type,1,20) as type,
substr(path,1,50) as path
from sysibmadm.dbpaths
order by type
"
```

## get system information
```
db2 "select * from table(sysproc.env_get_sys_info()) as systeminfo"
db2 "select * from table(sysproc.env_get_inst_info()) as instanceinfo"
db2 "select * from table(sysproc.env_get_prod_info()) as prodinfo"
```


## get container
```
select substr(TBSP_NAME,1,16) as TBS_NAME, total_pages, substr(CONTAINER_NAME,1,60) as
Container_name from sysibmadm.snapcontainer order by 1,3;
```


