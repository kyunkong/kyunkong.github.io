---
title: "Oracle Useful Scripts"
date: 2018-01-15 12:11:25
---
[TOC]

# showlocks.sql show all the lock information
```
-- name: showlock.sql
column o_name format a10
column lock_type format a20
column object_name format a15
select rpad(oracle_username,10) o_name,session_id sid,
decode(locked_mode,0,'None',1,'Null',2,'Row share',
3,'Row Exclusive',4,'Share',5,'Share Row Exclusive',6,'Exclusive') lock_type,
object_name ,xidusn,xidslot,xidsqn
from v$locked_object,all_objects
where v$locked_object.object_id=all_objects.object_id;
```

# showalllocks.sql show all locks information
```
select sid,type,id1,id2,
decode(lmode,0,'None',1,'Null',2,'Row share',
3,'Row Exclusive',4,'Share',5,'Share Row Exclusive',6,'Exclusive')
lock_type,request,ctime,block
from v$lock
where TYPE IN('TX','TM');
```



# my_sess_event.sql-show current session status
```sql
select event, total_waits, time_waited
from v$session_event
where sid=(select sid from v$mystat where rownum=1)
order by 3 desc;
```

# show_param.sql-show parameter including un-documented parameters
```sql
select ksppinm, ksppstvl
from x$ksppi x, x$ksppcv y
where (x.indx=y.indx)
and (translate(ksppinm,'_','#') like '%&1%);
```

# system_event.sql, show the entire system waiting status
```sql
select * from
(select event, total_waits, time_waited
from v$system_event
where wait_class<>'Idel'
order by 3 desc)
where rownum<=100;
```

# sesstat.sql-show current session v$sesstat
```sql
select n.name, sum(s.value)
from v$sesstat s, v$statname n
where n.name like '%&stat_name%'
and s.statistic#=n.statistic#
and s.sid=(select sid from v$mystat where rownum=1)
group by n.name;
```

# undosize.sql, get the current undo information
```sql
select used_ublk, used_urec
from v$transaction t, v$session s
where s.sid=(select sid from v$mystat where rownum=1)
and s.taddr=t.addr;
```

# show_cursor_num.sql- finding the max cursor number
```
-- statistics for database cursor number
-- Purpose:   确定一个业务逻辑需要的Cursor数量
-- name: cursor_num.sql
-- From:
col cumu_open_cur for 9999
col max_open_cur for a10

select max(a.value) as cumu_open_cur, p.value as max_open_cur
from v$sesstat a, v$statname b, v$parameter p
where a.statistic# = b.statistic#
and b.name = 'opened cursors cumulative'
and p.name = 'open_cursors'
and a.sid = (select max(sid) from v$mystat)
group by p.value

```

# log_frequency.sql- finding the log switch frequency
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

# lfsdiag.sql- Diagnose log file sync
```
-- purpose: Debug log file sync wait event
-- name: lfsdiag.sql
-- from: Script to Collect Log File Sync Diagnostic Information (lfsdiag.sql) [Document 1064487.1]
set echo off
set feedback off
column timecol new_value timestamp
column spool_extension new_value suffix
select to_char(sysdate,'Mondd_hh24mi') timecol,
'.out' spool_extension from sys.dual;
column output new_value dbname
select value || '_' output
from v$parameter where name = 'db_name';
spool lfsdiag_&&dbname&&timestamp&&suffix
set trim on
set trims on
set lines 130
set pages 100
set verify off
alter session set optimizer_features_enable = '10.2.0.4';

PROMPT LFSDIAG DATA FOR &&dbname&&timestamp
PROMPT Note: All timings are in milliseconds (1000 milliseconds = 1 second)

PROMPT
PROMPT IMPORTANT PARAMETERS RELATING TO LOG FILE SYNC WAITS:
column name format a40 wra
column value format a40 wra
select inst_id, name, value from gv$parameter
where ((value is not null and name like '%log_archive%') or
name like '%commit%' or name like '%event=%' or name like '%lgwr%')
and name not in (select name from gv$parameter where (name like '%log_archive_dest_state%'
and value = 'enable') or name = 'log_archive_format')
order by 1,2,3;

PROMPT
PROMPT HISTOGRAM DATA FOR LFS AND OTHER RELATED WAITS:
PROMPT
PROMPT APPROACH: Look at the wait distribution for log file sync waits
PROMPT by looking at "wait_time_milli". Look at the high wait times then
PROMPT see if you can correlate those with other related wait events.
column event format a40 wra
select inst_id, event, wait_time_milli, wait_count
from gv$event_histogram
where event in ('log file sync','gcs log flush sync',
'log file parallel write','wait for scn ack',
'log file switch completion','gc cr grant 2-way',
'gc buffer busy','gc current block 2-way') or
event like '%LGWR%' or event like '%LNS%'
order by 2 desc,1,3;

PROMPT
PROMPT ORDERED BY WAIT_TIME_MILLI
select inst_id, event, wait_time_milli, wait_count
from gv$event_histogram
where event in ('log file sync','gcs log flush sync',
'log file parallel write','wait for scn ack',
'log file switch completion','gc cr grant 2-way',
'gc buffer busy','gc current block 2-way')
or event like '%LGWR%' or event like '%LNS%'
order by 3,1,2 desc;

PROMPT
PROMPT REDO WRITE STATS
PROMPT
PROMPT "redo write time" in centiseconds (100 per second)
PROMPT 11.1: "redo write broadcast ack time" in centiseconds (100 per second)
PROMPT 11.2: "redo write broadcast ack time" in microseconds (1000 per millisecond)
column value format 99999999999999999999
column milliseconds format 99999999999999.999
select v.version, ss.inst_id, ss.name, ss.value,
decode(substr(version,1,4),
'11.1',decode (name,'redo write time',value*10,
'redo write broadcast ack time',value*10),
'11.2',decode (name,'redo write time',value*10,
'redo write broadcast ack time',value/1000),
decode (name,'redo write time',value*10)) milliseconds
from gv$sysstat ss, v$instance v
where name like 'redo write%' and value > 0
order by 1,2,3;

PROMPT
PROMPT ASH THRESHOLD...
PROMPT
PROMPT This will be the threshold in milliseconds for average log file sync
PROMPT times. This will be used for the next queries to look for the worst
PROMPT 'log file sync' minutes. Any minutes that have an average log file
PROMPT sync time greater than the threshold will be analyzed further.
column threshold_in_ms new_value threshold format 999999999.999
select min(threshold_in_ms) threshold_in_ms
from (select inst_id, to_char(sample_time,'Mondd_hh24mi') minute,
avg(time_waited)/1000 threshold_in_ms
from gv$active_session_history
where event = 'log file sync'
group by inst_id,to_char(sample_time,'Mondd_hh24mi')
order by 3 desc)
where rownum <= 5;

PROMPT
PROMPT ASH WORST MINUTES FOR LOG FILE SYNC WAITS:
PROMPT
PROMPT APPROACH: These are the minutes where the avg log file sync time
PROMPT was the highest (in milliseconds).
column event format a30 tru
column program format a35 tru
column total_wait_time format 999999999999.999
column avg_time_waited format 999999999999.999
select to_char(sample_time,'Mondd_hh24mi') minute, inst_id, event,
sum(time_waited)/1000 TOTAL_WAIT_TIME , count(*) WAITS,
avg(time_waited)/1000 AVG_TIME_WAITED
from gv$active_session_history
where event = 'log file sync'
group by to_char(sample_time,'Mondd_hh24mi'), inst_id, event
having avg(time_waited)/1000 > &&threshold
order by 1,2;

PROMPT
PROMPT ASH LFS BACKGROUND PROCESS WAITS DURING WORST MINUTES:
PROMPT
PROMPT APPROACH: What is LGWR doing when 'log file sync' waits
PROMPT are happening? LMS info may be relevent for broadcast
PROMPT on commit and LNS data may be relevant for dataguard.
PROMPT If more details are needed see the ASH DETAILS FOR WORST
PROMPT MINUTES section at the bottom of the report.
column inst format 999
column event format a30 tru
column program format a35 wra
select to_char(sample_time,'Mondd_hh24mi') minute, inst_id inst, program, event,
sum(time_waited)/1000 TOTAL_WAIT_TIME , count(*) WAITS,
avg(time_waited)/1000 AVG_TIME_WAITED
from gv$active_session_history
where to_char(sample_time,'Mondd_hh24mi') in (select to_char(sample_time,'Mondd_hh24mi')
from gv$active_session_history
where event = 'log file sync'
group by to_char(sample_time,'Mondd_hh24mi'), inst_id
having avg(time_waited)/1000 > &&threshold)
and (program like '%LGWR%' or program like '%LMS%' or
program like '%LNS%' or event = 'log file sync')
group by to_char(sample_time,'Mondd_hh24mi'), inst_id, program, event
order by 1,2,3,5 desc, 4;

PROMPT
PROMPT AWR WORST AVG LOG FILE SYNC SNAPS:
PROMPT
PROMPT APPROACH: These are the AWR snaps where the average 'log file sync'
PROMPT times were the highest.
column begin format a12 tru
column end format a12 tru
column name format a13 tru
select dhs.snap_id, dhs.instance_number inst, to_char(dhs.begin_interval_time,'Mondd_hh24mi') BEGIN,
to_char(dhs.end_interval_time,'Mondd_hh24mi') END,
en.name, se.time_waited_micro/1000 total_wait_time, se.total_waits,
se.time_waited_micro/1000 / se.total_waits avg_time_waited
from dba_hist_snapshot dhs, wrh$_system_event se, v$event_name en
where (dhs.snap_id = se.snap_id and dhs.instance_number = se.instance_number)
and se.event_id = en.event_id and en.name = 'log file sync' and
dhs.snap_id in (select snap_id from (
select se.snap_id, se.time_waited_micro/1000 / se.total_waits avg_time_waited
from wrh$_system_event se, v$event_name en
where se.event_id = en.event_id and en.name = 'log file sync'
order by avg_time_waited desc)
where rownum < 4)
order by 1,2;

PROMPT
PROMPT AWR REDO WRITE STATS
PROMPT
PROMPT "redo write time" in centiseconds (100 per second)
PROMPT 11.1: "redo write broadcast ack time" in centiseconds (100 per second)
PROMPT 11.2: "redo write broadcast ack time" in microseconds (1000 per millisecond)
column stat_name format a30 tru
select v.version, ss.snap_id, ss.instance_number inst, sn.stat_name, ss.value,
decode(substr(version,1,4),
'11.1',decode (stat_name,'redo write time',value*10,
'redo write broadcast ack time',value*10),
'11.2',decode (stat_name,'redo write time',value*10,
'redo write broadcast ack time',value/1000),
decode (stat_name,'redo write time',value*10)) milliseconds
from wrh$_sysstat ss, wrh$_stat_name sn, v$instance v
where ss.stat_id = sn.stat_id
and sn.stat_name like 'redo write%' and ss.value > 0
and ss.snap_id in (select snap_id from (
select se.snap_id, se.time_waited_micro/1000 / se.total_waits avg_time_waited
from wrh$_system_event se, v$event_name en
where se.event_id = en.event_id and en.name = 'log file sync'
order by avg_time_waited desc)
where rownum < 4)
order by 1,2,3;

PROMPT
PROMPT AWR LFS AND OTHER RELATED WAITS FOR WORST LFS AWRs:
PROMPT
PROMPT APPROACH: These are the AWR snaps where the average 'log file sync'
PROMPT times were the highest. Look at related waits at those times.
column name format a40 tru
select se.snap_id, se.instance_number inst, en.name,
se.total_waits, se.time_waited_micro/1000 total_wait_time,
se.time_waited_micro/1000 / se.total_waits avg_time_waited
from wrh$_system_event se, v$event_name en
where se.event_id = en.event_id and (en.name in
('log file sync','gcs log flush sync',
'log file parallel write','wait for scn ack',
'log file switch completion','gc cr grant 2-way',
'gc buffer busy','gc current block 2-way')
or en.name like '%LGWR%' or en.name like '%LNS%')
and se.snap_id in (select snap_id from (
select se.snap_id, se.time_waited_micro/1000 / se.total_waits avg_time_waited
from wrh$_system_event se, v$event_name en
where se.event_id = en.event_id and en.name = 'log file sync'
order by avg_time_waited desc)
where rownum < 4)
order by 1, 6 desc;

PROMPT
PROMPT AWR HISTOGRAM DATA FOR LFS AND OTHER RELATED WAITS FOR WORST LFS AWRs:
PROMPT Note: This query won't work on 10.2 - ORA-942
PROMPT
PROMPT APPROACH: Look at the wait distribution for log file sync waits
PROMPT by looking at "wait_time_milli". Look at the high wait times then
PROMPT see if you can correlate those with other related wait events.
select eh.snap_id, eh.instance_number inst, en.name, eh.wait_time_milli, eh.wait_count
from wrh$_event_histogram eh, v$event_name en
where eh.event_id = en.event_id and
(en.name in ('log file sync','gcs log flush sync',
'log file parallel write','wait for scn ack',
'log file switch completion','gc cr grant 2-way',
'gc buffer busy','gc current block 2-way')
or en.name like '%LGWR%' or en.name like '%LNS%')
and snap_id in (select snap_id from (
select se.snap_id, se.time_waited_micro/1000 / se.total_waits avg_time_waited
from wrh$_system_event se, v$event_name en
where se.event_id = en.event_id and en.name = 'log file sync'
order by avg_time_waited desc)
where rownum < 4)
order by 1,3 desc,2,4;

PROMPT
PROMPT ORDERED BY WAIT_TIME_MILLI
PROMPT Note: This query won't work on 10.2 - ORA-942
select eh.snap_id, eh.instance_number inst, en.name, eh.wait_time_milli, eh.wait_count
from wrh$_event_histogram eh, v$event_name en
where eh.event_id = en.event_id and
(en.name in ('log file sync','gcs log flush sync',
'log file parallel write','wait for scn ack',
'log file switch completion','gc cr grant 2-way',
'gc buffer busy','gc current block 2-way')
or en.name like '%LGWR%' or en.name like '%LNS%')
and snap_id in (select snap_id from (
select se.snap_id, se.time_waited_micro/1000 / se.total_waits avg_time_waited
from wrh$_system_event se, v$event_name en
where se.event_id = en.event_id and en.name = 'log file sync'
order by avg_time_waited desc)
where rownum < 4)
order by 1,4,2,3 desc;

PROMPT
PROMPT ASH DETAILS FOR WORST MINUTES:
PROMPT
PROMPT APPROACH: If you cannot determine the problem from the data
PROMPT above, you may need to look at the details of what each session
PROMPT is doing during each 'bad' snap. Most likely you will want to
PROMPT note the times of the high log file sync waits, look at what
PROMPT LGWR is doing at those times, and go from there...
column program format a45 wra
column sample_time format a25 tru
column event format a30 tru
column time_waited format 999999.999
column p1 format a40 tru
column p2 format a40 tru
column p3 format a40 tru
select sample_time, inst_id inst, session_id, program, event, time_waited/1000 TIME_WAITED,
p1text||': '||p1 p1,p2text||': '||p2 p2,p3text||': '||p3 p3
from gv$active_session_history
where to_char(sample_time,'Mondd_hh24mi') in (select
to_char(sample_time,'Mondd_hh24mi')
from gv$active_session_history
where event = 'log file sync'
group by to_char(sample_time,'Mondd_hh24mi'), inst_id
having avg(time_waited)/1000 > &&threshold)
order by 1,2,3,4,5;

select to_char(sysdate,'Mondd hh24:mi:ss') TIME from dual;

spool off

PROMPT
PROMPT OUTPUT FILE IS: lfsdiag_&&dbname&&timestamp&&suffix
PROMPT
```

# glogin.sql
```
--
-- Copyright (c) 1988, 2004, Oracle Corporation.  All Rights Reserved.
--
-- NAME
--   glogin.sql
--
-- DESCRIPTION
--   SQL*Plus global login "site profile" file
--
--   Add any SQL*Plus commands here that are to be executed when a
--   user starts SQL*Plus, or uses the SQL*Plus CONNECT command
--
-- USAGE
--   This script is automatically run
--

-- Used by Trusted Oracle
COLUMN ROWLABEL FORMAT A15

-- Used for the SHOW ERRORS command
COLUMN LINE/COL FORMAT A8
COLUMN ERROR    FORMAT A65  WORD_WRAPPED

-- Used for the SHOW SGA command
COLUMN name_col_plus_show_sga FORMAT a24
COLUMN units_col_plus_show_sga FORMAT a15
-- Defaults for SHOW PARAMETERS
COLUMN name_col_plus_show_param FORMAT a36 HEADING NAME
COLUMN value_col_plus_show_param FORMAT a30 HEADING VALUE

-- Defaults for SHOW RECYCLEBIN
COLUMN origname_plus_show_recyc   FORMAT a16 HEADING 'ORIGINAL NAME'
COLUMN objectname_plus_show_recyc FORMAT a30 HEADING 'RECYCLEBIN NAME'
COLUMN objtype_plus_show_recyc    FORMAT a12 HEADING 'OBJECT TYPE'
COLUMN droptime_plus_show_recyc   FORMAT a19 HEADING 'DROP TIME'

-- Defaults for SET AUTOTRACE EXPLAIN report
-- These column definitions are only used when SQL*Plus
-- is connected to Oracle 9.2 or earlier.
COLUMN id_plus_exp FORMAT 990 HEADING i
COLUMN parent_id_plus_exp FORMAT 990 HEADING p
COLUMN plan_plus_exp FORMAT a60
COLUMN object_node_plus_exp FORMAT a8
COLUMN other_tag_plus_exp FORMAT a29
COLUMN other_plus_exp FORMAT a44

-- Default for XQUERY
COLUMN result_plus_xquery HEADING 'Result Sequence'

set termout off

col dbname new_value prompt_dbname
select substr(global_name,1,instr(global_name,'.')-1) dbname
from global_name;
set sqlprompt "_USER'@'&&prompt_dbname> "

-- set title of CMD
col mysid new_value mysid
alter session set NLS_DATE_FORMAT = 'DD.MM.YYYY HH24:MI:SS';

set serveroutput on
set verify off
set linesize 400
set pagesize 1115
set tab off

#DEFINE _EDITOR = vim
col STANDBY_DEST format a12
col ARCHIVED    format a8
col FAL         format a3


REM ********************
REM   v$logfile
REM ********************
REM GROUP#                           NUMBER
REM STATUS                           VARCHAR2(7)
REM TYPE                             VARCHAR2(7)
REM MEMBER                           VARCHAR2(513)
col IS_RECOVERY_DEST_FILE format a21


REM **********************
REM   V$ARCHIVE_DEST
REM **********************
col DEST_NAME    format a19
col STATUS   format a9
col TARGET   format a7
col DESTINATION format a20
col TRANSMIT_MODE  format a13
col AFFIRM         format a6
col DB_UNIQUE_NAME  format a14


REM **********************
REM   V$DATABASE
REM **********************
col DBID                         format     NUMBER
col NAME                         format a9
col RESETLOGS_CHANGE#            format     NUMBER
col RESETLOGS_TIME               format     DATE
col PROTECTION_MODE              format a20
col PROTECTION_LEVEL             format a20
col REMOTE_ARCHIVE               format a14
col DATABASE_ROLE                format a16
col ARCHIVELOG_CHANGE#           format     NUMBER
col GUARD_STATUS                 format a12
col FORCE_LOGGING                format a13

REM **********************
REM DBA_COMMON_AUDIT_TRAIL
REM **********************
col AUDIT_TYPE format a18
col DB_USER    format a7
col USERHOST   format a32
col OBJECT_SCHEMA format a13
col OBJECT_NAME   format a11
col SQL_TEXT      format a30

REM **************
REM all_sequences
REM **************
col SEQUENCE_OWNER format a14
col SEQUENCE_NAME  format a25
col CYCLE_FLAG     format a10
col ORDER_FLAG     format a10

REM *****************
REM v$parameter
REM *****************
col name             format a36
col value            format a30
col isdefault        format a9
col isses_modifiable format a16
col issys_modifiable format a16

REM *****************
REM v$sql
REM *****************
col sql_text format a100

REM *****************
REM v$session
REM *****************
col machine format a22

REM *****************
REM user_tables
REM *****************
col table_name format a30

REM *****************
REM Dictionary
REM *****************
col comments  format a120

REM **************************
REM     Tabelle employees
REM **************************
col first_name   format a11
col last_name    format a11
col email        format a8
col phone_number format a18
col hire_date    format a15

COL owner            format a8
COL search_condition format a30
COL column_name      format a15
COL constraint_name  format a20
COL data_default     format a13
COL text             format a55
COL view_name        format a20
COL index_name       format a22
COL object_name      format a22
COL object_type      format a22
COL trigger_name     format a10
COL table_owner      format a11
COL triggering_event format a10
COL trigger_type     format a16
COL grantee          format a10
COL grantor          format a10
COL role             format a25
COL username         format a20
col kommentar        format a20
-- col name             format a66
col large_pool_size  format a15
col file_name        format a66
col tablespace_name  format a15
col segment_name     format a30
col member           format a60
col description      format a35
col trigger_body     format a35
col parameter        format a35
col data_default     format a20
col resource_name    format a30
col limit            format a20
col profile          format a20
col attribute        format a15
col char_value       format a15
col member           format a50
col host_name        format a20
col "%"              format 990.99 justify right
col AVERAGE_WAIT     format 9999990.00
col account_status       format a10
col default_tablespace   format a20
col temporary_tablespace format a20
col password         format a17
col modus            format a5
col operation        format a30
col options          format a15
col index_free_percentage format 90.99

col os_username      format a20
col terminal         format a20
col obj_name         format a20
col action_time      format a20
col action_name      format a20

col event            format a30
col p1text           format a20
col sid              format 999
col what             format a30

col department_name  format a20
col property_value   format a30
col property_name    format a30
col statistics_name  format a30
col activation_level format a20
col optimizer        format a10
col program          format a30
col instance_name    format a20
col status           format a20
col type             format a20
col namespace        format a20
col osuser           format a20
col logon_time       format a20
col deferrable       format a15
col deferred         format a15
col r_owner          format a15
col tablespace       format a20
col datafile         format a50
set termout on
```

# show_space.sql by Tom Kyte
```
-- -----------------------------------------------------------------------------------
-- Author       : Tom Kyte
-- Description  : Displays free and unused space for the specified object.
-- Call Syntax  : EXEC Show_Space('Tablename');
-- Requirements : SET SERVEROUTPUT ON
-- Last Modified: June 22, 2010
-- This enhance version has all the fixes for ASSM, LMT, partitions etc (Oracle version 10gr2 +)
-- -----------------------------------------------------------------------------------
set define off

create or replace procedure show_space
( p_segname in varchar2,
  p_owner   in varchar2 default user,
  p_type    in varchar2 default 'TABLE',
  p_partition in varchar2 default NULL )
-- this procedure uses authid current user so it can query DBA_*
-- views using privileges from a ROLE and so it can be installed
-- once per database, instead of once per user that wanted to use it authid current_user
AUTHID CURRENT_USER
as
    l_free_blks                 number;
    l_total_blocks              number;
    l_total_bytes               number;
    l_unused_blocks             number;
    l_unused_bytes              number;
    l_LastUsedExtFileId         number;
    l_LastUsedExtBlockId        number;
    l_LAST_USED_BLOCK           number;
    l_segment_space_mgmt        varchar2(255);
    l_unformatted_blocks number;
    l_unformatted_bytes number;
    l_fs1_blocks number; l_fs1_bytes number;
    l_fs2_blocks number; l_fs2_bytes number;
    l_fs3_blocks number; l_fs3_bytes number;
    l_fs4_blocks number; l_fs4_bytes number;
    l_full_blocks number; l_full_bytes number;

    -- inline procedure to print out numbers nicely formatted
    -- with a simple label
    procedure p( p_label in varchar2, p_num in number )
    is
    begin
        dbms_output.put_line( rpad(p_label,40,'.') ||
                              to_char(p_num,'999,999,999,999') );
    end;
begin
   -- this query is executed dynamically in order to allow this procedure
   -- to be created by a user who has access to DBA_SEGMENTS/TABLESPACES
   -- via a role as is customary.
   -- NOTE: at runtime, the invoker MUST have access to these two
   -- views!
   -- this query determines if the object is a ASSM object or not
   begin
      execute immediate
          'select ts.segment_space_management
             from dba_segments seg, dba_tablespaces ts
            where seg.segment_name      = :p_segname
              and (:p_partition is null or
                  seg.partition_name = :p_partition)
              and seg.owner = :p_owner
              and seg.segment_type = :p_type
              and seg.tablespace_name = ts.tablespace_name'
             into l_segment_space_mgmt
            using p_segname, p_partition, p_partition, p_owner, p_type;
   exception
       when too_many_rows then
          dbms_output.put_line
          ( 'This must be a partitioned table, use p_partition => ');
          return;
   end;


   -- if the object is in an ASSM tablespace, we must use this API
   -- call to get space information, else we use the FREE_BLOCKS
   -- API for the user managed segments
   if l_segment_space_mgmt = 'AUTO'
   then
     dbms_space.space_usage
     ( p_owner, p_segname, p_type, l_unformatted_blocks,
       l_unformatted_bytes, l_fs1_blocks, l_fs1_bytes,
       l_fs2_blocks, l_fs2_bytes, l_fs3_blocks, l_fs3_bytes,
       l_fs4_blocks, l_fs4_bytes, l_full_blocks, l_full_bytes, p_partition);

     p( 'Unformatted Blocks ', l_unformatted_blocks );
     p( 'FS1 Blocks (0-25)  ', l_fs1_blocks );
     p( 'FS2 Blocks (25-50) ', l_fs2_blocks );
     p( 'FS3 Blocks (50-75) ', l_fs3_blocks );
     p( 'FS4 Blocks (75-100)', l_fs4_blocks );
     p( 'Full Blocks        ', l_full_blocks );
  else
     dbms_space.free_blocks(
       segment_owner     => p_owner,
       segment_name      => p_segname,
       segment_type      => p_type,
       freelist_group_id => 0,
       free_blks         => l_free_blks);

     p( 'Free Blocks', l_free_blks );
  end if;

  -- and then the unused space API call to get the rest of the
  -- information
  dbms_space.unused_space
  ( segment_owner     => p_owner,
    segment_name      => p_segname,
    segment_type      => p_type,
    partition_name    => p_partition,
    total_blocks      => l_total_blocks,
    total_bytes       => l_total_bytes,
    unused_blocks     => l_unused_blocks,
    unused_bytes      => l_unused_bytes,
    LAST_USED_EXTENT_FILE_ID => l_LastUsedExtFileId,
    LAST_USED_EXTENT_BLOCK_ID => l_LastUsedExtBlockId,
    LAST_USED_BLOCK => l_LAST_USED_BLOCK );

    p( 'Total Blocks', l_total_blocks );
    p( 'Total Bytes', l_total_bytes );
    p( 'Total MBytes', trunc(l_total_bytes/1024/1024) );
    p( 'Unused Blocks', l_unused_blocks );
    p( 'Unused Bytes', l_unused_bytes );
    p( 'Last Used Ext FileId', l_LastUsedExtFileId );
    p( 'Last Used Ext BlockId', l_LastUsedExtBlockId );
    p( 'Last Used Block', l_LAST_USED_BLOCK );
end;
/
set define on
```

# print_table.sql by Tom Kyte
```sql
create or replace procedure print_table( p_query in varchar2 )
AUTHID CURRENT_USER
is
    l_theCursor     integer default dbms_sql.open_cursor;
    l_columnValue   varchar2(4000);
    l_status        integer;
    l_descTbl       dbms_sql.desc_tab;
    l_colCnt        number;
begin
    execute immediate
    'alter session set
        nls_date_format=''dd-mon-yyyy hh24:mi:ss'' ';

    dbms_sql.parse(  l_theCursor,  p_query, dbms_sql.native );
    dbms_sql.describe_columns
    ( l_theCursor, l_colCnt, l_descTbl );

    for i in 1 .. l_colCnt loop
        dbms_sql.define_column
        (l_theCursor, i, l_columnValue, 4000);
    end loop;

    l_status := dbms_sql.execute(l_theCursor);

    while ( dbms_sql.fetch_rows(l_theCursor) > 0 ) loop
        for i in 1 .. l_colCnt loop
            dbms_sql.column_value
            ( l_theCursor, i, l_columnValue );
            dbms_output.put_line
            ( rpad( l_descTbl(i).col_name, 30 )
              || ': ' ||
              l_columnValue );
        end loop;
        dbms_output.put_line( '-----------------' );
    end loop;
    execute immediate
        'alter session set nls_date_format=''dd-MON-rr'' ';
exception
    when others then
      execute immediate
          'alter session set nls_date_format=''dd-MON-rr'' ';
      raise;
end;
/
```


# show_space.sql by Tom Kyte
```sql
-- -----------------------------------------------------------------------------------
-- Author       : Tom Kyte
-- Description  : Displays free and unused space for the specified object.
-- Call Syntax  : EXEC Show_Space('Tablename');
-- Requirements : SET SERVEROUTPUT ON
-- Last Modified: June 22, 2010
-- This enhance version has all the fixes for ASSM, LMT, partitions etc (Oracle version 10gr2 +)
-- -----------------------------------------------------------------------------------

set define off

create or replace procedure show_space
( p_segname in varchar2,
  p_owner   in varchar2 default user,
  p_type    in varchar2 default 'TABLE',
  p_partition in varchar2 default NULL )
-- this procedure uses authid current user so it can query DBA_*
-- views using privileges from a ROLE and so it can be installed
-- once per database, instead of once per user that wanted to use it authid current_user
as
    l_free_blks                 number;
    l_total_blocks              number;
    l_total_bytes               number;
    l_unused_blocks             number;
    l_unused_bytes              number;
    l_LastUsedExtFileId         number;
    l_LastUsedExtBlockId        number;
    l_LAST_USED_BLOCK           number;
    l_segment_space_mgmt        varchar2(255);
    l_unformatted_blocks number;
    l_unformatted_bytes number;
    l_fs1_blocks number; l_fs1_bytes number;
    l_fs2_blocks number; l_fs2_bytes number;
    l_fs3_blocks number; l_fs3_bytes number;
    l_fs4_blocks number; l_fs4_bytes number;
    l_full_blocks number; l_full_bytes number;

    -- inline procedure to print out numbers nicely formatted
    -- with a simple label
    procedure p( p_label in varchar2, p_num in number )
    is
    begin
        dbms_output.put_line( rpad(p_label,40,'.') ||
                              to_char(p_num,'999,999,999,999') );
    end;
begin
   -- this query is executed dynamically in order to allow this procedure
   -- to be created by a user who has access to DBA_SEGMENTS/TABLESPACES
   -- via a role as is customary.
   -- NOTE: at runtime, the invoker MUST have access to these two
   -- views!
   -- this query determines if the object is a ASSM object or not
   begin
      execute immediate
          'select ts.segment_space_management
             from dba_segments seg, dba_tablespaces ts
            where seg.segment_name      = :p_segname
              and (:p_partition is null or
                  seg.partition_name = :p_partition)
              and seg.owner = :p_owner
              and seg.segment_type = :p_type
              and seg.tablespace_name = ts.tablespace_name'
             into l_segment_space_mgmt
            using p_segname, p_partition, p_partition, p_owner, p_type;
   exception
       when too_many_rows then
          dbms_output.put_line
          ( 'This must be a partitioned table, use p_partition => ');
          return;
   end;


   -- if the object is in an ASSM tablespace, we must use this API
   -- call to get space information, else we use the FREE_BLOCKS
   -- API for the user managed segments
   if l_segment_space_mgmt = 'AUTO'
   then
     dbms_space.space_usage
     ( p_owner, p_segname, p_type, l_unformatted_blocks,
       l_unformatted_bytes, l_fs1_blocks, l_fs1_bytes,
       l_fs2_blocks, l_fs2_bytes, l_fs3_blocks, l_fs3_bytes,
       l_fs4_blocks, l_fs4_bytes, l_full_blocks, l_full_bytes, p_partition);

     p( 'Unformatted Blocks ', l_unformatted_blocks );
     p( 'FS1 Blocks (0-25)  ', l_fs1_blocks );
     p( 'FS2 Blocks (25-50) ', l_fs2_blocks );
     p( 'FS3 Blocks (50-75) ', l_fs3_blocks );
     p( 'FS4 Blocks (75-100)', l_fs4_blocks );
     p( 'Full Blocks        ', l_full_blocks );
  else
     dbms_space.free_blocks(
       segment_owner     => p_owner,
       segment_name      => p_segname,
       segment_type      => p_type,
       freelist_group_id => 0,
       free_blks         => l_free_blks);

     p( 'Free Blocks', l_free_blks );
  end if;

  -- and then the unused space API call to get the rest of the
  -- information
  dbms_space.unused_space
  ( segment_owner     => p_owner,
    segment_name      => p_segname,
    segment_type      => p_type,
    partition_name    => p_partition,
    total_blocks      => l_total_blocks,
    total_bytes       => l_total_bytes,
    unused_blocks     => l_unused_blocks,
    unused_bytes      => l_unused_bytes,
    LAST_USED_EXTENT_FILE_ID => l_LastUsedExtFileId,
    LAST_USED_EXTENT_BLOCK_ID => l_LastUsedExtBlockId,
    LAST_USED_BLOCK => l_LAST_USED_BLOCK );

    p( 'Total Blocks', l_total_blocks );
    p( 'Total Bytes', l_total_bytes );
    p( 'Total MBytes', trunc(l_total_bytes/1024/1024) );
    p( 'Unused Blocks', l_unused_blocks );
    p( 'Unused Bytes', l_unused_bytes );
    p( 'Last Used Ext FileId', l_LastUsedExtFileId );
    p( 'Last Used Ext BlockId', l_LastUsedExtBlockId );
    p( 'Last Used Block', l_LAST_USED_BLOCK );
end;
/
set define on
```

# waitprof.sql-按等待事件频率分组
```
-- https://blog.tanelpoder.com/files/scripts/
--
-- File name:   waitprof.sql ( Session Wait Profiler )
-- Purpose:     Sample V$SESSION_WAIT at high frequency and show resulting 
--              session wait event and parameter profile by session
--
-- Author:      Tanel Poder
-- Copyright:   (c) http://www.tanelpoder.com
--
-- Usage:       @waitprof <print|noprint> <sid> <e[123s]> <#samples>
--
--                  <print|noprint>
--                              - whether to print P2,P3,SEQ# values or not
--
--                  <sid>       - session ID of session to sample
--
--                  <[e123s]>   - sample grouping
--                      e - group by event name
--                      1 - group by P1 of v$session_wait event
--                      2 - group by P2
--                      3 - group by P3
--                      s - group by SEQ#
--
--                  <#samples>  - how many samples to take (a modern CPU can take
--                                tens of thousands to low hundreds of k samples
--                                per second)
--
--  Examples:
--              @waitprof noprint 350 e 1000000   -- take million samples, group by event only
--              @waitprof print 350 e123 500000 -- take 500k samples, group by event,p1,p2,p3
--              @waitprof print 350 e3 1000000  -- take million samples, group by event,p3
--              @waitprof print 350 es 1000000  -- take million samples, group by event,seq#
--
--  Other:
--              The sampling relies on NESTED LOOP join method and having
--              V$SESSION_WAIT as the inner (probed) table. Note that on 9i
--              you may need to run this script as SYS as it looks like otherwise
--              the global USE_NL hint is not propagated down to X$ base tables
--
--              If sampling always reports a single distinct event even though 
--              many different events (or parameter values) are expected then 
--              the execution plan used is not right.
--
--------------------------------------------------------------------------------

DEF _swp_print=&1
DEF _swp_sid=&2
DEF _swp_p123=&3
DEF _swp_samples=&4

col sw_event    head EVENT for a35 truncate
col sw_p1transl head P1TRANSL for a42
col sw_sid      head SID for 999999
col swp_p1 head P1 for a26 word_wrap
col swp_p2 head P2 for a16 word_wrap &_swp_print
col swp_p3 head P3 for a16 word_wrap &_swp_print
col swp_seq# head SEQ# &_swp_print
col pct_total_samples head "% Total|Time" format 999.99
col waitprof_total_ms head "Total Event|Time ms" format 9999999.999
col dist_events head Distinct|Events
col average_samples head Average|Samples
col waitprof_avg_ms head "Avg time|ms/Event" format 99999.999

prompt
prompt -- WaitProf 1.04 by Tanel Poder ( http://www.tanelpoder.com )

WITH 
    t1 AS (SELECT hsecs FROM v$timer),
    samples AS (
    SELECT /*+ ORDERED NO_MERGE USE_NL(sw.gv$session_wait.s) */
        s.sid sw_sid,
        CASE WHEN sw.state = 'WAITING' THEN 'WAITING' ELSE 'WORKING' END AS state,
        CASE WHEN sw.state = 'WAITING' THEN event ELSE 'On CPU / runqueue' END AS sw_event,
        CASE WHEN sw.state = 'WAITING' AND '&_swp_p123' LIKE '%1%' THEN sw.p1text || '= ' || CASE WHEN (LOWER(sw.p1text) LIKE '%addr%' OR sw.p1 >= 536870912) 
THEN RAWTOHEX(sw.p1raw) ELSE TO_CHAR(sw.p1) END ELSE NULL END swp_p1,
        CASE WHEN sw.state = 'WAITING' AND '&_swp_p123' LIKE '%2%' THEN sw.p2text || '= ' || CASE WHEN (LOWER(sw.p2text) LIKE '%addr%' OR sw.p2 >= 536870912) 
THEN RAWTOHEX(sw.p2raw) ELSE TO_CHAR(sw.p2) END ELSE NULL END swp_p2,
        CASE WHEN sw.state = 'WAITING' AND '&_swp_p123' LIKE '%3%' THEN sw.p3text || '= ' || CASE WHEN (LOWER(sw.p3text) LIKE '%addr%' OR sw.p3 >= 536870912) 
THEN RAWTOHEX(sw.p3raw) ELSE TO_CHAR(sw.p3) END ELSE NULL END swp_p3,
        CASE WHEN LOWER('&_swp_p123') LIKE '%s%' THEN sw.seq# ELSE NULL END seq#,
        COUNT(*) total_samples,
        COUNT(DISTINCT seq#) dist_events,
        TRUNC(COUNT(*)/COUNT(DISTINCT seq#)) average_samples
    FROM
        (   SELECT /*+ NO_MERGE */ TO_NUMBER(&_swp_sid) sid FROM 
            (SELECT rownum r FROM dual CONNECT BY ROWNUM <= 1000) a,
            (SELECT rownum r FROM dual CONNECT BY ROWNUM <= 1000) b,
            (SELECT rownum r FROM dual CONNECT BY ROWNUM <= 1000) c
           WHERE ROWNUM <= &_swp_samples
        ) s,
        v$session_wait sw
    WHERE
        s.sid = sw.sid
    GROUP BY
        s.sid,
        CASE WHEN sw.state = 'WAITING' THEN 'WAITING' ELSE 'WORKING' END,
        CASE WHEN sw.state = 'WAITING' THEN event ELSE 'On CPU / runqueue' END,
        CASE WHEN sw.state = 'WAITING' AND '&_swp_p123' LIKE '%1%' THEN sw.p1text || '= ' || CASE WHEN (LOWER(sw.p1text) LIKE '%addr%' OR sw.p1 >= 536870912) 
THEN RAWTOHEX(sw.p1raw) ELSE TO_CHAR(sw.p1) END ELSE NULL END,
        CASE WHEN sw.state = 'WAITING' AND '&_swp_p123' LIKE '%2%' THEN sw.p2text || '= ' || CASE WHEN (LOWER(sw.p2text) LIKE '%addr%' OR sw.p2 >= 536870912) 
THEN RAWTOHEX(sw.p2raw) ELSE TO_CHAR(sw.p2) END ELSE NULL END,
        CASE WHEN sw.state = 'WAITING' AND '&_swp_p123' LIKE '%3%' THEN sw.p3text || '= ' || CASE WHEN (LOWER(sw.p3text) LIKE '%addr%' OR sw.p3 >= 536870912) 
THEN RAWTOHEX(sw.p3raw) ELSE TO_CHAR(sw.p3) END ELSE NULL END,
        CASE WHEN LOWER('&_swp_p123') LIKE '%s%' THEN sw.seq# ELSE NULL END
    ORDER BY
        CASE WHEN LOWER('&_swp_p123') LIKE '%s%' THEN -seq# ELSE total_samples END DESC
),
    t2 AS (SELECT hsecs FROM v$timer)
SELECT /*+ ORDERED */
    s.sw_sid,
    s.state,
    s.sw_event,
    s.swp_p1,
    s.swp_p2,
    s.swp_p3,
    s.seq# swp_seq#,
    s.total_samples / &_swp_samples * 100 pct_total_samples,
    (t2.hsecs - t1.hsecs) * 10 * s.total_samples / &_swp_samples waitprof_total_ms,
    s.dist_events,
--  s.average_samples,
    (t2.hsecs - t1.hsecs) * 10 * s.total_samples / dist_events / &_swp_samples waitprof_avg_ms
FROM
    t1,
    samples s,
    t2
/

--UNDEF _swp_sid=&1
--UNDEF _swp_p123=&2
--UNDEF _swp_samples=&3
```

## dba_function script
```
#-----------------------------------------------------------#
# show environment variables in sorted list
  function envs {
    if test -z "$1"
      then /bin/env | /bin/sort
      else /bin/env | /bin/sort | /bin/grep -i $1
    fi
  } # envs
#-----------------------------------------------------------#
# login to sqlplus
  function sp {
    time sqlplus "/ as sysdba"
  } # sp
#-----------------------------------------------------------#
# find largest files below this point
function flf {
  find . -ls | sort -nrk7 | head -10
}
#-----------------------------------------------------------#
# find largest directories consuming space below this point
function fld {
  du -S . | sort -nr | head -10
}
#-----------------------------------------------------------#
# cd to bdump
  function bdump {
   echo $ORACLE_HOME | grep 11 >/dev/null
   if [ $? -eq 0 ]; then
     lower_sid=$(echo $ORACLE_SID | tr '[:upper:]' '[:lower:]')
     cd $ORACLE_BASE/diag/rdbms/$lower_sid/$ORACLE_SID/trace
   else
     cd $ORACLE_BASE/admin/$ORACLE_SID/bdump
   fi
  } # bdump
#-----------------------------------------------------------#
# view alert log
  function valert {
   echo $ORACLE_HOME | grep 11 >/dev/null
   if [ $? -eq 0 ]; then
     lower_sid=$(echo $ORACLE_SID | tr '[:upper:]' '[:lower:]')
     view $ORACLE_BASE/diag/rdbms/$lower_sid/$ORACLE_SID/trace/alert_$ORACLE_SID.log
   else
     view $ORACLE_BASE/admin/$ORACLE_SID/bdump/alert_$ORACLE_SID.log
   fi
  } # valert
#-----------------------------------------------------------#

export EDITOR=vi
export VISUAL=$EDITOR
export SQLPATH=$HOME/scripts
set -o vi
#
# list directories only
alias lsd="ls -p | grep /"
# show top cpu consuming processes
alias topc="ps -e -o pcpu,pid,user,tty,args | sort -n -k 1 -r | head"
# show top memory consuming processes
alias topm="ps -e -o pmem,pid,user,tty,args | sort -n -k 1 -r | head"
#
alias sqlp='sqlplus "/ as sysdba"'
alias shutdb='echo "shutdown immediate;" | sqlp'
alias startdb='echo "startup;" | sqlp'
```
