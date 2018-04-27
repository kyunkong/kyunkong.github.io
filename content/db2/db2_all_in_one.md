---
title: "DB2 All in One"
date: 2018-01-17 11:15:28
---
[TOC]


## DB2 Foudamentals Concepts
### background process:
1.db2sysc
DB2 main system controller or engine, starting from v9.5, there is only one mutil-threaded main engine process for the entire partition. All EDUs(Engine Dispatchable Units) are threads inside this process. Without this process, database server cannot funtion. The process responsible for start-up and shut-down and the management of the runing instance.
2.db2wdog
DB2 watch dog, is the parent of db2sysc, it clean up resources if the db2sysc process abnormally terminates. The daemon s funtion like PMON in oracle.
3.db2acd
The automatic computing daemon, it performs client side automatic tasks, such as health monitor, automatic maintenance utilities.
4.db2vend
Third party vendors process interact with DB2, such as TSM--dsmc in TSM client backup schedule.
5.db2fmp
Fenced processes that run user code on the server outside the firewall for both stored procedures and user defined functions.
6.db2tcpcm
TCP/IP communication protocal listener.
7.db2agent
Coordinator agent that perform database operations on behalf of applications.
8.db2agentp ---------**************-----------
Perform database operations for the application. db2agent will coordinate the work between the different db2agentp subagents. The dbm parameter "INTRA_PARALLEL" set to YES.
9.db2pfchr
DB2 asynchronous I/O data prefetcher(NUM_IOSERVERS)--like Oracle server process
10.db2pclnr
DB2 asynchronous I/O data writer(NUM_IOCLEANERS)--like Oracle DBWn


## db2cfexp & db2cfimp
```
[db2ace@oc7421025535 ~]$ db2cfexp
 db2cfexp exports configuration information.

   db2cfexp fname [ template | backup | maintain ]
                    -          -        -

     fname - path/filename of the export profile.

Options:
   template - create a generic export profile to
              transfer this configuration to another
              workstation. (incl. configuration info,
              catalog info, ODBC info ).

   maintain - create a Database export profile to
              transfer the Database catalog information
              to another workstation.

   backup   - create a backup configuration profile for
              backup purposes at this workstation only.

[db2ace@oc7421025535 ~]$ db2cfimp
 db2cfimp imports configuration information.

   db2cfimp fname

     fname - path/filename of the profile to be imported
```

### DB2 tablespace status
```
0x0                 Normal
0x1                 Quiesced: SHARE
0x2                 Quiesced: UPDATE
0x4                 Quiesced: EXCLUSIVE
0x8                 Load pending
0x10               Delete pending
0x20               Backup pending
0x40               Roll forward in progress
0x80               Roll forward pending
0x100             Restore pending
0x100             Recovery pending (not used)
0x200             Disable pending
0x400             Reorg in progress
0x800             Backup in progress
0x1000           Storage must be defined
0x2000           Restore in progress
0x4000           Offline and not accessible
0x8000           Drop pending
0x2000000     Storage may be defined
0x4000000     StorDef is in 'final' state
0x8000000     StorDef was changed prior to rollforward
0x10000000   DMS rebalancer is active
0x20000000   TBS deletion in progress
0x40000000   TBS creation in progress
0x8                 For service use only
```

###how to find table space status in DB2
```
[db2v10i@hadr02 ~]$ db2tbst 0x8000
State = Drop pending
```



db2iupgrade VS db2iupdt
When upgrading an instance from one fixpack to a higher fixpack on the same release use db2iupdt.

If you are upgrading an instance from one release to another (Eg. from v9.7 to v10.1) use db2iupgrade.

If you try to use db2iupdt to upgrade an instance from one release to another, it will fail with a DBI1978E error.

Example: Trying to upgrade instance - db2inst1 from DB2 v9.7 to DB2 v10.1:


## ./db2iupdt -u db2inst1 db2inst1
DBI1446I  The db2iupdt command is running.

DB2 installation is being initialized.

DBI1978E  Updating instance db2inst1 failed because the release
      of the DB2 copy on which the instance is running is not the same
      as the release of the target DB2 copy. Release of DB2 copy on
      which the instance is running: 9.7. Release of
      target DB2 copy: 10.1.
##诊断Client连接问题
1.查看client是否有连接
db2 list applications
2.查看Server端环境变量设置
db2set -all
3.查看DB manager配置
db2 get dbm cfg |grep -i svcename
4.查看server端/etc/services
cat /etc/services |grep -i db2
5.查看端口是否有被监听
netstat -apn |grep -i db2sysc

##Rename column
```
db2 "alter table table_name rename column COL_NAME to NEW_COL_NAME"
```


##tablespace 相关操作
```
db2 list tablespaces [show detail]
db2 list tablespace container for container_id [show detail]
db2 "select * from syscat.tablespaces"
db2 get snapshot for tablespaces on db_name
db2pd -db db_name -tablespaces
--AR #AutoReize
--AS #AutoStorage

db2pd -d db_name -storagepaths
```


##查看bufferpool
```
db2 "select char(bpname,20) as bpname,BUFFERPOOLID,char(DBPGNAME,20) as dbpgname,NPAGES,PAGESIZE,ESTORE,NUMBLOCKPAGES,BLOCKSIZE,char(NGNAME,20) as ngname from syscat.bufferpools"
db2pd -d db_name -buff
SELECT TOTAL_HIT_RATIO_PERCENT, DATA_HIT_RATIO_PERCENT, INDEX_HIT_RATIO_PERCENT, substr(BP_NAME,1,20) as bufferpool FROM SYSIBMADM.BP_HITRATIO
```

##查看安装了哪些DB2组件
```
[db2inst1@db2srv ~]$ db2ls -a -q -b /opt/ibm/db2/V9.7/
db2inst3@pokziscsrv40:/opt/ibm/db2/V9.7_FP8> db2ls -q -p -b /opt/ibm/db2/V9.7_FP11_SB_35317

Install Path : /opt/ibm/db2/V9.7_FP11_SB_35317

Product Response File ID                  Level   Fix Pack   Product Description
---------------------------------------------------------------------------------------------------------------------
ENTERPRISE_SERVER_EDITION               9.7.0.11         11   DB2 Enterprise Server Edition
```


##find table in which tablespaces
```
db2 "select substr(tabname,1,20),substr(tbspace,1,30) from syscat.tables where TBSPACE='SYSTOOLSPACE'"
db2 "select substr(tabname,1,20),substr(tbspace,1,30) from syscat.tables where tabname='POLICY'"
```

##查看syscat所有的catalog view
```
list tables for schema syscat
select tabname from syscat.tables where tabschema='SYSCAT'


[db2inst1@db2srv worktmp]$ db2 list command options

     Command Line Processor Option Settings

 Backend process wait time (seconds)        (DB2BQTIME) = 1
 No. of retries to connect to backend        (DB2BQTRY) = 60
 Request queue wait time (seconds)          (DB2RQTIME) = 5
 Input queue wait time (seconds)            (DB2IQTIME) = 5
 Command options                           (DB2OPTIONS) =

 Option  Description                               Current Setting
 ------  ----------------------------------------  ---------------
   -a    Display SQLCA                             OFF
   -c    Auto-Commit                               ON
   -d    Retrieve and display XML declarations     OFF
   -e    Display SQLCODE/SQLSTATE                  OFF
   -f    Read from input file                      OFF
   -i    Display XML data with indentation         OFF
   -l    Log commands in history file              OFF
   -m    Display the number of rows affected       OFF
   -n    Remove new line character                 OFF
   -o    Display output                            ON
   -p    Display interactive input prompt          ON
   -q    Preserve whitespaces & linefeeds          OFF
   -r    Save output to report file                OFF
   -s    Stop execution on command error           OFF
   -t    Set statement termination character       OFF
   -v    Echo current command                      OFF
   -w    Display FETCH/SELECT warning messages     ON
   -x    Suppress printing of column headings      OFF
   -z    Save all output to output file            OFF
```


##Memory allocation
DSS: evenly allocate between buffer pool and sort area, because OLAP need scan large of data and perform a number of sorts
OLTP: dedicate approximately 75 percent of your available memory to the buffer pool and divide the rest among the sort heaps.


##查看用户表空间
```
[db2inst1@db2srv scripts]$ db2 "select tabname, tbspace from syscat.tables where tabschema='DB2INST1'"
```

##Extract the whole schema
```
db2look -d testdb -z FUNG -e -o ~/schema_db2look.sql

db2look -d dn_name -l --extract DDL about tablespaces
db2look -d db_name -t table_name -e --extract DDL about tables
db2 "select 'db2look -d testdb -t '|| rtrim(TABSCHEMA) ||'.'||'\"'||TABNAME||'\"'|| ' -e' from syscat.tables where TBSPACEID='2'" > db2look.sql
./db2look.sql >o.txt
sed -i '/^COMMIT\ WORK/d' o.txt
sed -i '/^CONNECT\ RESET/d' o.txt
sed -i '/^TERMINATE/d' o.txt
```

##truncate a table in DB2
```
[db2inst1@db2srv sqllib]$ db2 "select * from t"

ID          NAME                 LOCATION
----------- -------------------- ----------
      51369 Fung                 -
      51368 Kong                 Shenzhen
      51370 Kyun                 CQ
[db2inst1@db2srv ~]$ touch null.dat
[db2inst1@db2srv ~]$ db2 "import from null.dat of del replace into T"
SQL3109N  The utility is beginning to load data from file "null.dat".

SQL3110N  The utility has completed processing.  "0" rows were read from the
input file.

SQL3221W  ...Begin COMMIT WORK. Input Record Count = "0".

SQL3222W  ...COMMIT of any database changes was successful.

SQL3149N  "0" rows were processed from the input file.  "0" rows were
successfully inserted into the table.  "0" rows were rejected.


Number of rows read         = 0
Number of rows skipped      = 0
Number of rows inserted     = 0
Number of rows updated      = 0
Number of rows rejected     = 0
Number of rows committed    = 0

[db2inst1@db2srv ~]$ db2 "select * from t"

ID          NAME                 LOCATION
----------- -------------------- ----------

  0 record(s) selected.
```

## DB2 authorizations
```
[db2inst1@db2srv ~]$ db2 get AUTHORIZATIONS

 Administrative Authorizations for Current User

 Direct SYSADM authority                    = NO
 Direct SYSCTRL authority                   = NO
 Direct SYSMAINT authority                  = NO
 Direct DBADM authority                     = YES
 Direct CREATETAB authority                 = NO
 Direct BINDADD authority                   = NO
 Direct CONNECT authority                   = NO
 Direct CREATE_NOT_FENC authority           = NO
 Direct IMPLICIT_SCHEMA authority           = NO
 Direct LOAD authority                      = NO
 Direct QUIESCE_CONNECT authority           = NO
 Direct CREATE_EXTERNAL_ROUTINE authority   = NO
 Direct SYSMON authority                    = NO

 Indirect SYSADM authority                  = YES
 Indirect SYSCTRL authority                 = NO
 Indirect SYSMAINT authority                = NO
 Indirect DBADM authority                   = NO
 Indirect CREATETAB authority               = YES
 Indirect BINDADD authority                 = YES
 Indirect CONNECT authority                 = YES
 Indirect CREATE_NOT_FENC authority         = NO
 Indirect IMPLICIT_SCHEMA authority         = YES
 Indirect LOAD authority                    = NO
 Indirect QUIESCE_CONNECT authority         = NO
 Indirect CREATE_EXTERNAL_ROUTINE authority = NO
 Indirect SYSMON authority                  = NO
```


##Client Connections Relative
1.Server端配置TCP/IP信息
```
db2set db2comm=tcpip
db2 update dbm cfg using svcename SERVICE_NAME/PORT_NUMBER(如果使用service_name，那么service name要和/etc/services里面的一致)
```
设置完之后需重启实例使设置生效
2.客户端配置
##catalog node
```
db2 catalog tcpip node node_alias remote 192.168.122.144 server 60000
```
##catalog db
```
db2 catalog database db_name as db_alias at node node_alias [authentication SERVER]
```
3.客户端验证
```
db2 list node directory
db2 list db directory
```
##login using client
```
db2 connect to db_alias user db2_inst_users_in_server
```


##DB2随机启动
```
[db2inst1@db2srv ~]$ db2iauto -on db2inst1

#Automatic maintenance configuration parameter
1. AUTO_DB_BACKUP: Default set to OFF,
2. AUTO_TBL_MAINT: Default set to ON,
3. AUTO_RUNSTATS: Default set to ON,
4. AUTO_STMT_STATS: Default set to ON,
5. AUTO_STATS_VIEWS: Default set to OFF,
6. AUTO_SAMPLING: Default set to OFF,
7. AUTO_STATS_PROF: Default set to OFF,
8. AUTO_PROF_UPD: Default set to OFF,
9. AUTO_REORG: Default set to ON,
10. AUTO_REVAL: Default set to DEFERRED,
```


##DB2日志状态
DB2日志有三种状态
1. active
Active状态的日志，即此日志包含了已提交但尚未写入磁盘的数据、未提交但已写入磁盘的数据；尚未提交或者rollback的数据。总而言之，就是在crash recovery需要用到日志均为active日志。
2.online archive log
Online archive log在crash recovery时已经不需要使用到了，说它是online，仅仅是因为这些日志文件还在LOG_DIR中
3.offline archive log
已经归档到其他地方，如TSM中

--Infinite Active Logging
条件：
1.LOGARCHMETH1参数必须设置为USEREXIT, DISK, TSM, or VENDOR
2.LOGSECOND参数设置为-1
作用：
在归档模式下，日志写满即归档，如果不是infinite archive的配置，则需要等待被归档完的日志状态变成inactive才会rename他们，如果事务比较大，在secondary log分配完后仍旧没有日志inactive，或者磁盘空间慢，数据库此时会hang住，无法写入日志，同时无法写日志的事务会进行回滚。但如果是处于infinite archive logging模式下，则日志写满即归档后，不会等待日志状态inactive才rename日志，而需要用到此日志时候会立即rename，在这种情况下，大事务也不会因为上述的情况而导致无法写日志。但infinite archive log也存在不好的地方，当crash recovery的时候，有可能需要从归档路径还原日志，因此，处于此模式下的db恢复的时候时间会长很多。

##Monitor log space usage:
```
select int(total_log_used/1024/1024) as "Log Used (Meg)",
       int(total_log_available/1024/1024) as "Log Space Free (Meg)",
       int((float(total_log_used)/ float(total_log_used + total_log_available))*100) as "Pct Used",
       int(tot_log_used_top/1024/1024) as "Max Log Used (Meg)",
       int(sec_log_used_top/1024/1024) as "Max Sec. Used (Meg)",
       int(sec_logs_allocated) as "Secondaries"
from sysibmadm.snapdb;
```


##Find fence ID owner
```
[db2v97i@db2srv ~]$ ls -l $HOME/sqllib/adm/.fenced
r--r--r-- 1 db2fenc1 db2fadm1 0 Oct 14 17:44 /home/db2v97i/sqllib/adm/.fenced
```

##Change log mode to archive log
```
db2 update db cfg for testdb using LOGARCHMETH1 disk:/data2/arch
db2 force application all
db2 backup db testdb to /data2/backup/
```
--修改完归档模式后，需要对数据库做一次offline全备
##查找archive log history
```
db2 list history archive log all for db_name
```
##手动归档日志
```
db2 connect reset
db2 archive log for database db_name
```
##修改在线日志数目
```
db2 update db cfg for testdb using logprimary 10
```
##查看数据库是否处于归档模式
```
db2 get db cfg for testdb|grep -i log
结果中，如果以下两个值为YES和不为空，一般可以判断当前数据库为归档模式
 User exit for logging status                            = YES
 First log archive method                 (LOGARCHMETH1) = DISK:/data2/arch/
```


##offline backup
```
db2 backup db db_name to backup_path
```
##online backup plus logs
```
db2 backup db db_name online to backup_path include logs			-- normal backup
db2 backup db db_name online incremental to backup_path include logs		-- incremental backup
db2 backup db db_name online incremental delta to bk_path include logs		-- incremental delta backup
--增量备份需要开启TRACKMOD，且在做第一次增量备份前需有一次开启tracemod后的全备为基础，具体可以查看IBM Knowledge center中trackmod的解析
db2 update db cfg for db_name using trackmod yes immediate
```



##Looking for DB backup progress
```
db2 list utilities show detail
```

##备份介质管理
--list backup history
```
db2 list history backup all for db_name
db2 list history backup since <YYYYMMDD> for <db_name> |grep -v "  00"
```
--delete backup set
```
db2 prune history
```

##Naming convention
```
TESTDB.0.db2inst1.NODE0000.CATN0000.20150806143318.001
对于DB2备份文件，字段间以“.”分割，分别的含义如下：
TESTDB:			数据库名称
0:			备份类型，0为全备，3为表空间备份，4为Load COPY备份
NODE0000:		分区节点名称，单分区环境中，固定为node0000
CATN0000:		catalog节点名称，单分区环境中，固定为catn0000
20150806143318：	备份时间戳
001:			备份sequence number
```
##db2ckbkp命令--检查介质完整性，显示备份文件的metadata信息
```
[db2inst1@db2srv backup]$ db2ckbkp -h TESTDB.0.db2inst1.NODE0000.CATN0000.20150806143318.001
=====================
MEDIA HEADER REACHED:
=====================
	Server Database Name           -- TESTDB
	Server Database Alias          -- TESTDB
	Client Database Alias          -- TESTDB
	Timestamp                      -- 20150806143318
	Database Partition Number      -- 0
	Instance                       -- db2inst1
	Sequence Number                -- 1
	Release ID                     -- D00			--版本号，900-v7, A00-v8, B00-v9.1 C00-v9.5 D00-v9.7, F00-v10.1
	Database Seed                  -- CCD849F4
	DB Comment's Codepage (Volume) -- 0
	DB Comment (Volume)            --
	DB Comment's Codepage (System) -- 0
	DB Comment (System)            --
	Authentication Value           -- -1
	Backup Mode                    -- 1 			--0-offline backup	1-online backup
	Includes Logs                  -- 1			  --0-no			1-yes
	Compression                    -- 0
	Backup Type                    -- 0			--0-full backup		1-tablesapce
	Backup Gran.                   -- 0			--0-normal backup	16-incremental		48-delta
	Merged Backup Image            -- 0
	Status Flags                   -- 21
	System Cats inc                -- 1
	Catalog Partition Number       -- 0
	DB Codeset                     -- UTF-8
	DB Territory                   --
	LogID                          -- 1438758002
	LogPath                        -- /data1/db2inst1/NODE0000/SQL00001/SQLOGDIR/
	Backup Buffer Size             -- 3280896
	Number of Sessions             -- 1
	Platform                       -- 1E
 The proper image file name would be:
TESTDB.0.db2inst1.NODE0000.CATN0000.20150806143318.001
[1] Buffers processed:	#################################
Image Verification Complete - successful.
```
##检测前滚所需日志--增量恢复检查
```
[db2inst1@db2srv backup]$ db2ckbkp -a TESTDB.0.db2inst1.NODE0000.CATN0000.20150806143318.001 |grep -i "File Number"
                      File Number   [000] = 2
                      File Number   [001] = 3
                      File Number   [002] = 4
                      File Number   [003] = 5
                      File Number   [004] = 6
                      File Number   [005] = 7
                      File Number   [006] = 8
                      File Number   [007] = 9
                      File Number   [008] = 10
                      File Number   [009] = 11
```

##incremental restore
--finding whick backup image needed when restore from an incremental backup
```
[db2v97i@db2srv ~]$ db2 backup db mysample online incremental to /db2/backup/db2v97i/mysample/

Backup successful. The timestamp for this backup image is : 20160302165459

[db2v97i@db2srv ~]$ db2ckrst -d mysample -t 20160302165459

Suggested restore order of images using timestamp 20160302165459 for
database mysample.
====================================================================
 restore db mysample incremental taken at 20160302165459
 restore db mysample incremental taken at 20160225201758
 restore db mysample incremental taken at 20160302165459
--restore incremental backup image
db2 restore db mysample incremental taken at 20160302165459
db2  restore db mysample incremental taken at 20160225201758
db2 restore db mysample incremental taken at 20160302165459
--automatic restore incremental backup
db2 restore db mysample incremental automatic taken at 20160302165459
```

##Recovery
DB2支持三种模式恢复：
1.Crash Recover:	类似oracle的Crash recover，有个db cfg的AUTORESTART参数可以让crash recover在重新启动的时候自动运行，一般此参数设置为YES
2.Version Recover： 	类似oracle的restore，如果从online backup中restore，则有可能需要roll forward
3.Rollforward：		类似oracle的recover，需要注意的是，roll forward有两个选项，end of backup和end of log，如果是backup，则恢复到backup时间段，如果是log，则恢复到log的时间段
同时，DB2提供db2ckrst工具，该工具能提供恢复增量恢复的顺序，如下（建议直接使用incremental automatic进行恢复）
```
[db2inst1@db2srv backup]$ db2ckrst -d testdb -t 20150807105110 -r database
Suggested restore order of images using timestamp 20150807105110 for
database testdb.
====================================================================
 restore db testdb incremental taken at 20150807105110
 restore db testdb incremental taken at 20150807101213
 restore db testdb incremental taken at 20150807104804
 restore db testdb incremental taken at 20150807105110
====================================================================
```

##Restore & Recovery Examples
1.end of backup 和 end of log的区别（完全恢复）
```
[db2inst1@db2srv backup]$ db2 backup db testdb online to /data2/backup/ include logs
Backup successful. The timestamp for this backup image is : 20150807101213
[db2inst1@db2srv backup]$ db2 connect to testdb
   Database Connection Information
 Database server        = DB2/LINUXX8664 9.7.10
 SQL authorization ID   = DB2INST1
 Local database alias   = TESTDB
[db2inst1@db2srv backup]$ db2 list tables
Table/View                      Schema          Type  Creation time
------------------------------- --------------- ----- --------------------------
  0 record(s) selected.
```
在备份前是没有用户表的，创建一张表，然后进行restore
```
[db2inst1@db2srv backup]$ db2 "create table t(id int,name varchar(20)) in fung"
DB20000I  The SQL command completed successfully.
[db2inst1@db2srv backup]$ db2 list tables
Table/View                      Schema          Type  Creation time
------------------------------- --------------- ----- --------------------------
T                               DB2INST1        T     2015-08-07-10.19.22.891815
```
###以end of log进行roll forward
```
[db2inst1@db2srv backup]$ db2 restore db testdb from /data2/backup taken at 20150807101213
[db2inst1@db2srv backup]$ db2 rollforward db testdb to end of logs and stop
[db2inst1@db2srv backup]$ db2 connect to testdb
[db2inst1@db2srv backup]$ db2 list tables
Table/View                      Schema          Type  Creation time
------------------------------- --------------- ----- --------------------------
T                               DB2INST1        T     2015-08-07-10.19.22.891815
```
###以end of backup进行roll forward
```
[db2inst1@db2srv backup]$ db2 restore db testdb from /data2/backup taken at 20150807101213
[db2inst1@db2srv backup]$ db2 rollforward db testdb to end of backup and stop
[db2inst1@db2srv backup]$ db2 list tables
Table/View                      Schema          Type  Creation time
------------------------------- --------------- ----- --------------------------
  0 record(s) selected.
```

2.增量备份及完全恢复
```
[db2inst1@db2srv backup]$ db2 backup db testdb online incremental to /data2/backup/ include logs
Backup successful. The timestamp for this backup image is : 20150807104804
[db2inst1@db2srv backup]$ db2 "create table t(id int,name varchar(20))"
DB20000I  The SQL command completed successfully.
[db2inst1@db2srv backup]$ db2 "insert into t values('51369','Fung')"
DB20000I  The SQL command completed successfully.
[db2inst1@db2srv backup]$ db2 commit
DB20000I  The SQL command completed successfully.
[db2inst1@db2srv backup]$ db2 "select * from t"
ID          NAME
----------- --------------------
      51369 Fung
  1 record(s) selected.
[db2inst1@db2srv backup]$ db2 backup db testdb online incremental delta to /data2/backup/ include logs
Backup successful. The timestamp for this backup image is : 20150807105110
[db2inst1@db2srv backup]$ db2 restore db testdb incremental automatic from /data2/backup/ taken at 20150807105110
[db2inst1@db2srv backup]$ db2 rollforward db testdb to end of logs and stop
[db2inst1@db2srv backup]$ db2 "select * from t"
ID          NAME
----------- --------------------
      51369 Fung
```
restore incremental automatic表示自动恢复，即从db2ckrst中列出的items中自动恢复，如果没有automatic选项，则需要手动从list中一个个以incremental恢复。

3.不完全恢复
```
[db2inst1@db2srv backup]$ date
Fri Aug  7 10:59:14 CST 2015
[db2inst1@db2srv backup]$
[db2inst1@db2srv backup]$ db2 "delete from (select * from t where id='51369')"
DB20000I  The SQL command completed successfully.
[db2inst1@db2srv backup]$ db2 commit
DB20000I  The SQL command completed successfully.
[db2inst1@db2srv backup]$ db2 restore db testdb incremental automatic from /data2/backup/ taken at 20150807105110
[db2inst1@db2srv backup]$ db2 rollforward db testdb to 2015-08-07.10.57.41.00000 using local time
[db2inst1@db2srv backup]$ db2 rollforward db testdb stop
[db2inst1@db2srv backup]$ db2 "select * from t"
ID          NAME
----------- --------------------
      51369 Fung
```

4.表空间的恢复
```
[db2inst1@db2srv backup]$ db2 "restore db testdb tablespace ( FUNG) online from /data2/backup taken at 20150807101213"
```
--指定online表示其他用户可以连接，并且访问其他表空间
```
[db2inst1@db2srv backup]$ db2 "rollforward db testdb to end of logs tablespace (FUNG)"
```

5.日志的恢复
对于include log的备份，可以指定在某个位置仅恢复日志文件，然后利用这些日志进行前滚
```
[db2inst1@db2srv backup]$ db2 backup db testdb online to /data2/backup include logs
Backup successful. The timestamp for this backup image is : 20150807145504
```
仅恢复日志：
```
[db2inst1@db2srv backup]$ db2 restore db testdb logs from /data2/backup/ taken at 20150807145504 logtarget /data2/logs/
DB20000I  The RESTORE DATABASE command completed successfully.
[db2inst1@db2srv backup]$ ll /data2/logs/
total 12
-rw-------. 1 db2inst1 db2iadm1 12288 Aug  7 15:02 S0000022.LOG
```
如果需要连数据库一起恢复，再通过日志进行rollforward，则不需要指定logs参数：
```
db2 restore db testdb from /data2/backup/ taken at 20150807145504 logtarget /data2/logs/
db2 "rollforward db testdb to end of logs and stop overflow log path (/data2/logs) "
```

6.异机恢复
异机或者需要恢复到另一个目录，restore提供了三个选项：
I.TO target_directory
II.DBPATH ON target_directory
III.ON path_list
如果目标库不存在，restore的TO和DBPATH ON选项指定目标库的数据库目录，ON指定automatic storage路径，如果只有ON，则数据库目录放在ON指定的第一个目录。

7.redirect恢复
Redirect指在restore过程中，重新定义表空间container，类似Oracle RMAN中的set new name。但对于使用了automatic storage管理的TBS不能用此选项。
```
db2 restore db db_name into target_dbname redirect
```
接着修改container属性
```
db2 "set tablespace containers for 3 using (file '/tbs/ts1/cont000' 3000)"
db2 restore db db_name continue
```
对于多个表空间，DB2 V9提供了一个可以产生脚本的工具：redirect generate scripts，这个脚本可以帮助我们改善"set tablespace containers"的可操作性。
```
db2 restore db db_name redirect generate script redirect.dll
```
然后通过db2 -tvf redirect.dll进行重定向恢复

--example
###backup in the source db
```
[db2inst1@db2srv ~]$ db2 backup db testdb online to /restore/
Backup successful. The timestamp for this backup image is : 20160112175052
[db2inst1@db2srv ~]$ chmod 775 /restore/TESTDB.0.db2inst1.NODE0000.CATN0000.20160112175052.001
```

###restore tablespace on the target db
```
[db2inst3@db2srv restore]$ db2set DB2_RESTORE_GRANT_ADMIN_AUTHORITIES=ON
[db2inst3@db2srv restore]$ db2 "restore db testdb  REBUILD WITH tablespace (FUNG,SYSCATSPACE) taken at  20160112175240 on /restore/datafile dbpath on /restore/datafile  into  tmpdb logtarget  /restore/actlog newlogpath /restore/actlog redirect generate script redirect.clp"
DB20000I  The RESTORE DATABASE command completed successfully.
[db2inst3@db2srv restore]$ db2 -tvf redirect.clp
UPDATE COMMAND OPTIONS USING S ON Z ON TESTDB_NODE0000.out V ON
DB20000I  The UPDATE COMMAND OPTIONS command completed successfully.

SET CLIENT ATTACH_DBPARTITIONNUM  0
DB20000I  The SET CLIENT command completed successfully.

SET CLIENT CONNECT_DBPARTITIONNUM 0
DB20000I  The SET CLIENT command completed successfully.

RESTORE DATABASE TESTDB REBUILD WITH TABLESPACE ( SYSCATSPACE , TEMPSPACE1 , FUNG , SYSTOOLSTMPSPACE ) FROM '/restore' TAKEN AT 20160112175240 ON '/restore/datafile' DBPATH ON '/restore/datafile' INTO TMPDB LOGTARGET '/restore/actlog/' NEWLOGPATH '/restore/actlog/' REDIRECT
SQL1277W  A redirected restore operation is being performed.  Table space
configuration can now be viewed and table spaces that do not use automatic
storage can have their containers reconfigured.
DB20000I  The RESTORE DATABASE command completed successfully.

RESTORE DATABASE TESTDB CONTINUE
DB20000I  The RESTORE DATABASE command completed successfully.

[db2inst3@db2srv ~]$ db2 list utilities [show detail]

ID                               = 1
Type                             = RESTORE
Database Name                    = TMPDB
Partition Number                 = 0
Description                      = automatic db rebuild SYSCATSPACE...
Start Time                       = 01/13/2016 18:02:13.623404
State                            = Executing
Invocation Type                  = User

[db2inst3@db2srv restore]$  db2 rollforward db tmpdb query status

                                 Rollforward Status

 Input database alias                   = tmpdb
 Number of nodes have returned status   = 1

 Node number                            = 0
 Rollforward status                     = DB  pending
 Next log file to be read               = S0000008.LOG
 Log files processed                    =  -
 Last committed transaction             = 2016-01-12-09.52.40.000000 UTC


[db2inst3@db2srv restore]$ db2 rollforward db tmpdb to end of logs and stop
SQL1271W  Database "TMPDB" is recovered but one or more table spaces are off-line on node(s) "0".
```




8.前滚恢复
--前滚到某个时间点
```
db2 "select current timestamp from sysibm.sysdummy1"
db2 rollforward db testdb to 2015-08-10-09.24.55.858964 using local time
```
--前滚到日志结尾
```
db2 rollforward db testdb to end of logs and stop
```
--前滚到备份结尾
```
db2 rollforward db testdb to end of backup and stop
```


9. Recovering a dropped table
1) finding the tablespace status whether enable "DROPPED TABLE RECOVERY"
```
  db2 "select substr(TBSPACE,1,20) as TBS,TBSPACETYPE,DROP_RECOVERY from SYSCAT.TABLESPACES"
```
2) finding the dropped tables
```
  db2 list history dropped table all for mysample
```
3) restore a database-level or tablespace-level backup image taken before the table was dropped
```
  db2 "restore db mysample tablespace (FUNG) online from /db2/backup/db2v97i/mysample/ taken at 20160218174445"
```
4) create a export directory to which files contaning the table data are to be written
```
  mkdir -p /restore
```
5) rollforward to a PIT
```
  db2 "ROLLFORWARD DB mysample TO END OF LOGS TABLESPACE ONLINE RECOVER DROPPED TABLE 000000000000f35c00060004 TO /restore"
--db2 "ROLLFORWARD DB db_name TO END OF LOGS TABLESPACE ONLINE RECOVER DROPPED TABLE dropped_table_id TO export_dir"
```
--dropped table ID can get from list history command
6) re-create the dropped table (DDL can obtain from list history command )
7) import the data from that was export during the roll forward operation into the table
```
  db2 "IMPORT FROM /restore/NODE0000/data OF DEL INSERT INTO T"
  --db2 "IMPORT FROM export_dir OF DEL INSERT INTO table"
```




##运维工具
1.runstats
runstats为DB2收集统计信息的工具。它可以使用distribution参数收集数据分布，数据分布有两种，一种为Frequency(频率采样)，另一种为Quantile（百分比采样）
--收集表及索引信息，包括数据分布
```
db2 runstats on table schema_name.table_name on all columns with distribution and detailed indexes all
```
--收集索引统计信息
```
db2 runstats on table schema_name.table_name for indexes all
```
--收集表和索引统计信息
```
db2 runstats on table schema_name.table_name and detailed indexes all
```
--收集统计信息的同时，让该表可读写
```
db2 runstats on table db2user.employee allow write access
```
--收集表的统计信息，同时收集empid, empname的直方图信息，同时指定此表在runstats的时候只读
```
db2 runstats on table db2user.employee with distribution on columns ( empid, empname ) allow read access
```



--查看一张表是否有收集统计信息
```
[db2inst1@db2srv ~]$ db2 "select char(tabname,10) as tabname, stats_time from syscat.tables where tabname='T'"
TABNAME    STATS_TIME
---------- --------------------------
T          2015-08-10-09.46.54.131244

  1 record(s) selected.
```
--runstats脚本
```
#!/bin/bash
if [ "$#" < 3 ] ; then
echo "USAGE:$0 DB_NAME DB_USER_NAME DB_PASSWORD"
exit
fi
DB=$1
DB_USER=$2
DB_PWD=$3
db2 connect to $DB user $DB_USER using $DB_PWD
db2 "select trim('RUNSTATS ON TABLE ' || trim(tabschema) || '.' || tabname || ' ON ALL COLUMNS WITH DISTRIBUTION ON ALL COLUMNS AND SAMPLED DETAILED INDEXES ALL ALLOW WRITE ACCESS;') from syscat.tables where type='T'" > createRunstats.txt
grep RUNSTATS createRunstats.txt > runstats_detailed.sql
#db2 -tvf runstats_detailed.sql
```



###export table to ixf
```
db2 "export to account_revocation.txt2 of ixf select * from wiusrapp.account_revocation"


SELECT 'DB2 EXPORT TO ' || tabname ||'.DEL OF DEL MODIFIED BY COLDEL@ CODEPAGE=1208 MESSAGES ' || tabname || '.expout SELECT * FROM ' || tabname AS Exports, 'IMPORT FROM ' || tabname || '.del OF DEL MODIFIED BY CODEPAGE=1208 COLDEL@ MESSAGES ' || tabname || '.impout REPLACE INTO ' || tabname AS Imports FROM  syscat.tables  WHERE tabschema = 'DB2INST1'
```

###import table
```
#replace_create means delete old data and insert new data, if no table, will create table first
db2 "import from t.txt2 of ixf messages msg.out replace_create into t"

#append data
db2 "import from t.txt2 of ixf messages msg.out insert into t"

#append and update data
db2 "import from t.txt2 of ixf messages msg.out replace into t"
```

INSERT :
Adds the imported data to the table without changing the existing table data. The target table must already exist.
INSERT_UPDATE :
Adds the imported data to the target table or updates existing rows with matching primary keys. The target table must already exist defined with primary keys.
CREATE :
Creates the table, index definitions, and row contents. The input file must use the IXF format because this is the only format that stores table and index definitions.
REPLACE :
Deletes all existing data from the table and inserts the imported data. The table definition and index definitions are not changed.
REPLACE_CREATE:
If the table exists, this option behaves like the replace option. If the table does not exist,this option behaves like the create option, which creates the table and index definitions and then inserts the row contents. This option requires the input file to be in IXF format.


--db2 load
```
db2 "load from db2inst.t.ixf of ixf \
savecount 1000 \
warningcount 10 \
messages db2inst.t.ixf.out \
insert into t"

Four phase of LOAD:
Load:
Build:
Delete:
Index Copy:
```









2.reorg
reorg整理碎片，减少overflow(Oracle中的行迁移)，减少索引层次，但鉴于对资源消耗较大，建议停机或者业务空闲时间做。
reorgchk工具用来判断一张表或索引是否需要reorg，同时sysibmadm.snaptab也能判断一张表或索引是否需要reorg。
reorgchk利用8个公式，3个表公式，5个索引公式判断表或者索引是否需要重组，F1~F3标记为"*"，表明表需要重组，F4~F8标记为"*"，表明index需要reorg。
```
db2 reorgchk on schema db2inst1
```
--离线重组
离线重组默认是allow read access
```
db2 reorg table db2inst1.t
```
检查reorg是否完成
--db2 get snapshot
```
[db2inst1@db2srv ~]$ db2 get snapshot for tables on testdb
 Table Schema        = DB2INST1
 Table Name          = T
 Table Type          = User
 Data Object Pages   = 1
 Index Object Pages  = 4
 Rows Read           = Not Collected
 Rows Written        = 0
 Overflows           = 0
 Page Reorgs         = 0
 Table Reorg Information:
   Reorg Type        =
        Reclaiming
        Table Reorg
        Allow Read Access
        Reorg Data Only
   Reorg Index       = 0				--reorg index id
   Reorg Tablespace  = 4				--reorg tablespace
   Start Time        = 08/10/2015 10:52:38.625729	--reorg start time
   Reorg Phase       = 3 - Index Recreate
   Max Phase         = 3
   Phase Start Time  = 08/10/2015 10:52:39.444076
   Status            = Completed			--reorg status
   Current Counter   = 0
   Max Counter       = 0
   Completion        = 0
   End Time          = 08/10/2015 10:52:39.572889	--reorg end time
```

--db2pd
```
[db2inst1@db2srv ~]$ db2pd -d testdb -reorg
```

--db2 list history reorg all for testdb
```
[db2inst1@db2srv ~]$ db2 list history reorg all for testdb
 Op Obj Timestamp+Sequence Type Dev Earliest Log Current Log  Backup ID
 -- --- ------------------ ---- --- ------------ ------------ --------------
  G  T  20150810105238      F       S0000025.LOG S0000029.LOG
 ----------------------------------------------------------------------------
  Table:  "DB2INST1"."T"

 ----------------------------------------------------------------------------
    Comment: REORG
 Start Time: 20150810105238
   End Time: 20150810105239
     Status: A
 ----------------------------------------------------------------------------
  EID: 127
--其中Op=G表示reorg，Obj=T表示table，Obj=I表示index，Type=F表示offline，type=N表示online
```

--在线重组，会记录大量日志，可以随时启动和终止
```
[db2inst1@db2srv ~]$ db2 reorg table db2inst1.t inplace allow write access
```

--reorg index
```
[db2inst1@db2srv ~]$ db2 reorg indexes all for table t
```
对于索引reorg的监控，只能通过db2 list history all查看，或者查看diaglog。

## Reorg history
```
db2 "
SELECT SUBSTR(TABNAME, 1, 15) AS TAB_NAME,
SUBSTR(TABSCHEMA, 1, 15) AS TAB_SCHEMA,
REORG_PHASE, SUBSTR(REORG_TYPE, 1, 20) AS REORG_TYPE,
REORG_STATUS, REORG_COMPLETION, DBPARTITIONNUM
FROM SYSIBMADM.SNAPTAB_REORG ORDER BY DBPARTITIONNUM
"
```


##数据库占用空间的大小

```
[db2inst1@db2srv db2dump]$ db2 "call get_dbsize_info(?,?,?,0)"

  Value of output parameters
  --------------------------
  Parameter Name  : SNAPSHOTTIMESTAMP			--执行快照的时间戳
  Parameter Value : 2015-08-10-11.09.06.992189

  Parameter Name  : DATABASESIZE			--返回数据库的大小（bytes）
  Parameter Value : 101294080

  Parameter Name  : DATABASECAPACITY
  Parameter Value : 35067944960				--数据库容量大小（bytes）,即OS或者存储的总空间

  Return Status = 0
```

0表示0分钟，即立刻刷新

##表空间占用大小

```
list tablespaces show detail只能查看DMS的表空间大小，无法查看SMS的。对于automatic，只要该目录有空间，DB2会自动扩展。
tablespace_usage.xx:
db2 "select substr(tbsp_name,1,30) as TBS_NAME,\
substr(tbsp_type,1,10) as TBSP_TYPE,\
substr(tbsp_content_type,1,10) as TBS_TYPE,\
sum(tbsp_total_size_kb)/1024 as TOTAL_MB,\
sum(tbsp_used_size_kb)/1024 as USED_MB,\
sum(tbsp_free_size_kb)/1024 as FREE_MB,\
tbsp_page_size as PAGE_SIZE \
from sysibmadm.tbsp_utilization \
where tbsp_type!='SMS' \
group by tbsp_name,tbsp_type,tbsp_content_type,tbsp_page_size order by 1"
db2 -tvf tablspace_uage.xx
```

##表/索引占用空间大小
```
--table占用空间有db2pd -d db_name -tcbstats, admin_get_tab_inf函数和sysibmadm.admintabinfo:
```

###Perform the select below to know the size one table:
```
db2 "SELECT TABSCHEMA, TABNAME, SUM(DATA_OBJECT_P_SIZE)+
SUM(INDEX_OBJECT_P_SIZE)+ SUM(LONG_OBJECT_P_SIZE)+
SUM(LOB_OBJECT_P_SIZE)+ SUM(XML_OBJECT_P_SIZE) FROM
SYSIBMADM.ADMINTABINFO where tabschema='<tabschema_name>' and tabname='<table_name>' group by tabschema,tabname"
```

###Perform the select below to know the size all tables in a specific schema:
```
db2 "SELECT tabname,TABSCHEMA, SUM(DATA_OBJECT_P_SIZE)+
SUM(INDEX_OBJECT_P_SIZE)+ SUM(LONG_OBJECT_P_SIZE)+
SUM(LOB_OBJECT_P_SIZE)+ SUM(XML_OBJECT_P_SIZE) FROM
SYSIBMADM.ADMINTABINFO where TABSCHEMA=<tabschema_name> group by tabname,tabschema"
```

###Perform the select below to know the size of all schema:
```
 db2 "SELECT TABSCHEMA, SUM(DATA_OBJECT_P_SIZE)+
 SUM(INDEX_OBJECT_P_SIZE)+ SUM(LONG_OBJECT_P_SIZE)+
 SUM(LOB_OBJECT_P_SIZE)+ SUM(XML_OBJECT_P_SIZE) FROM
 SYSIBMADM.ADMINTABINFO GROUP BY TABSCHEMA"
```


##Lock
DB2事务隔离级别
1.Uncommitted Read
2.Cursor Stability		--Default Isolation Level
3.Read Stability
4.Repeatable Read
在DB2 V9.7之前，在default isolation leve（CS）下，读取数据的事务遇到正在被其他事务更新的数据必须等待更新事务的结束，9.7后，提供连Current Committed机制，类似Oracle中的Read consistency，用来解决高并发中的此问题

###查询当前session isolation level
```
[db2inst1@db2srv ~]$ db2 "SELECT CURRENT ISOLATION FROM sysibm.sysdummy1"

1
--
  1 record(s) selected.
```
空白表示为默认级别

Oracle事务隔离级别(isolation level)
1.read committed
2.read only
3.serializable

锁：
共享锁,Share lock
排它锁,eXclusive lock

DB2中常见的锁主要包括表锁及行锁，锁有三个基本属性：锁的模式、锁的对象及锁的持久性。
--表锁模式
表锁分为强类型锁和弱类型锁，强类型锁适用于表里的所有行，主要包括S，U，X，Z四种类型。弱锁类型也叫意向锁(intent),主要包括IN，IS，IX和SIX，意向锁的目的是配合行锁，在获得行锁前必须先获得表锁(Intent)
默认情况下，DB2不会实施强锁类型，只有通过lock table锁表或者发生锁升级的时候才会使用。
--弱锁类型（intent，意向锁，配合行锁使用，即在申请行锁前，必须先获得表级意向锁）
IN（Intent None）：			无意向锁，锁的拥有者可以读取表中的任何数据，包括其他事务未提交的数据，但不能修改任何一行，其他并发应用可以读写表中任意行
IS（Intent Share）：			意向共享锁，lock owner可以读，但不能修改，其他并发应用可以读写。当需要读表中的行时，需要先获取表上的IS锁
IX（Intent eXclusive）：			意向排它锁，lock owner和其他application都可以读写数据，一般当需要修改表中的行时（DML），需要先获取表上的IX锁
SIX（Shard with Intent eXclusive）：	lock owner可以读取表中所有行，如果owner有行级X锁，则可以修改该行。其他并发应用可以读取该表数据, S+IX或者IX+S
IS、IX、SIX方式用于表一级并需要行锁配合，他们可以阻止其他应用程序对该表加上排它锁。
--强锁类型
S（Share）：		lock owner和其他应用都可以读取表中任何数据，但不能更改
U（Update）：		lock owner可以读取表中任何数据，如果升级到X锁后，可以更新表中任何数据。该锁是处于等待对数据进行处理的中间状态（类似select for update）
X（eXclusive）：		lock owner能读写表中的任何数据，其他应用程序只有在Uncommitted Read事务隔离级别才可以读取未提交的数据
Z（Super eXclusive）：	当create、alter、drop时，需要Z锁（DDL语句），如果某表加上Z锁，那么其他应用程序，包括UR级别都无法读取该表数据

--行锁模式
行锁主要分为读锁和写锁，在获得行锁前，必须先获得表级锁
S（Share）：			lock owner和其他应用都可以读取该行，但不能更改						要求表锁最低模式为IS
U（Update）：			lock owner可以读取该行，并且有可能修改该行，其他并发应用只能读取该行				要求表锁最低模式为IX
X（eXclusive）：			lock owner可以读写该行，其他应用不能读写该行，但在UR可以读取					要求表锁最低模式为IX
W（Weak exclusive）：		当一行数据被插入表中时，该行会获得W锁，lock owner能修改该行，基本和X锁相同，但W锁同时兼容NW锁		要求表锁最低模式为IX
NS（next key share）：		在CS或者RS隔离级别替代S锁									要求表锁最低模式为IS
NW（next ket weak share）：	与X和W类似，当插入索引时候，该行下一键获得NW锁							要求表锁最低模式为IX
当有索引时才会用到W、NW，NS和S是在不同事务隔离级别下读锁的模式，CS和RS读锁为NS，RR读锁为S
db2pd -d db_name -locks可以查看到当前DB的所有锁

--锁等待
DB2中db cfg的locktimeout参数，锁超时时间，默认为-1，即为无限。可以根据不同应用类型修改此参数。
```
[db2inst1@db2srv ~]$ db2 get db cfg for sample |grep -i locktimeout
 Lock timeout (sec)                        (LOCKTIMEOUT) = -1
[db2inst1@db2srv ~]$ db2 update db cfg for testdb using locktimeout 10
```
重启实例后生效
```
[db2inst1@db2srv ~]$ db2 get db cfg for testdb |grep -i locktimeout
 Lock timeout (sec)                        (LOCKTIMEOUT) = 10
[db2inst1@db2srv ~]$ db2 +c "delete from t where name='Kyun'"
DB20000I  The SQL command completed successfully.
```
--在另一session
```
[db2inst1@db2srv ~]$ db2 +c "delete from t where location='CQ'"
DB21034E  The command was processed as an SQL statement because it was not a
valid Command Line Processor command.  During SQL processing it returned:
SQL0911N  The current transaction has been rolled back because of a deadlock
or timeout.  Reason code "68".  SQLSTATE=40001
```
--查找引起锁等待的语句
```
db2pd -d testdb -locks showlocks wait -tra -app -dyn > db2pd.out
db2pd -d mysample -locks -transactioins -applications -dynamic
```


--死锁
DB2中，死锁的发生最少要4条SQL语句，DB2中有个参数默认每隔十秒检查是否存在死锁，如果有，将会随机终止一个，然后让另一方继续进程事务。
```
[db2inst1@db2srv ~]$ db2 get db cfg for sample |grep -i dlchktime
 Interval for checking deadlock (ms)         (DLCHKTIME) = 10000
```

--Lock escalation ， 锁升级
DB2通过locklist和maxlocks触发锁升级。locklist指定每个数据库可以使用的最大锁内存大小，maxlocks指定每个应用可以占用内存的百分比。在V9版本之后，这两个参数可以调整为automatic，即自动调整
```
[db2inst1@db2srv ~]$ db2 get db cfg for sample |grep LOCK
 Max storage for lock list (4KB)              (LOCKLIST) = 4096
 Percent. of lock lists per application       (MAXLOCKS) = 10
```
锁升级的条件：锁内存使用超出了locklist大小，某个应用使用的锁空间内存达到连locklist*maxlocks%，当发生锁升级的时候，diag.log会记录锁升级的详细信息

##DB2内存使用查看
```
db2mtrk -i -d -p -v
db2pd -dbptnmem  --实例范围，不需要指定dbname
db2pd -memset -db db_name --实例或者数据库范围，指定dbname仅查询指定db内存
db2pd -mempool		--用法类似memset
```

##DB2监控工具
1.snapshot命令行
2.snapshot管理视图
3.db2pd
4.db2top
5.事件监控器 --尽量少用，对生产环境会有很大影响

--查看本机安装了几个DB版本
```
db2greg -dump
```




##db2gcfg查看实例状态
```
[db2inst1@db2srv ~]$ db2gcf -s -i db2inst1

Instance  : db2inst1
DB2 State : Available
```

--export multiple tables in DB2
```
#!/usr/bin/sh

db2 connect to sourcedb
list="
db2v97i.ACCESS_ID
db2v97i.ACCESS_ID_HIST
db2v97i.APPL_PRIV_CODE_RULE_BLUE_GROUP
db2v97i.CUST_RFC_BATCH_ERR
db2v97i.CUST_DTRMNTN_LOOKUP_HIST
db2v97i.NOTIFIED_USER
db2v97i.NOTIFIED_USER_HIST
db2v97i.SAMETIME_USERS
db2v97i.SAP_IDOC_USER_ID_XREF
db2v97i.SAP_IDOC_USER_ID_XREF_HIST
db2v97i.USER_PRIV
db2v97i.USER_PRIV_HIST
db2v97i.USERS
db2v97i.USERS_HIST
db2v97i.WEB_AUTH_REG
"

for one in $list
do

 db2 -v "export to $one.ixf of ixf select * from $one with ur" >>export.log


if [ $? -ne 0 ]; then
    echo "error table: $one">>error.log
fi
done
```

##finding rebalance status
```
db2 "
select
varchar(tbsp_name, 30) as tbsp_name,
dbpartitionnum,
member,
rebalancer_mode,
rebalancer_status,
rebalancer_extents_remaining,
rebalancer_extents_processed,
rebalancer_start_time
from table(mon_get_rebalance_status(NULL,-2)) as t
"
```


--DB2 Oracle数据字典的兼容
```
[db2inst1@db2srv ~]$ db2set DB2_COMPATIBILITY_VECTOR=ORA
[db2inst1@db2srv ~]$ db2stop
08/11/2015 09:33:42     0   0   SQL1025N  The database manager was not stopped because databases are still active.
SQL1025N  The database manager was not stopped because databases are still active.
[db2inst1@db2srv ~]$ db2start
08/11/2015 09:33:44     0   0   SQL1026N  The database manager is already active.
SQL1026N  The database manager is already active.

CLPPlus: Version 1.6
Copyright (c) 2009, 2011, IBM CORPORATION.  All rights reserved.
SQL> connect db2v10i@192.168.122.144:50001/testdb

Database Connection Information :
---------------------------------
Hostname = 192.168.122.144
Database server = DB2/LINUXX8664  SQL10055
SQL authorization ID = db2v10i
Local database alias = TESTDB
Port = 50001

SQL> select sysdate from dual;

1
---------------------
2015-08-31 17:31:22



db2 get dbm cfg > dbmcfg.bk
db2 get db cfg for database_name > dbcfg.bk
db2set -all > db2set.bk
db2 list db directory > systemdbdir.bk
db2 list node directory > nodedir.bk
db2 list dcs directory > dcsdir.bk
```

#db2rbind & db2 bind
db2rbind:
Usage notes

    This command uses the rebind API (sqlarbnd) to attempt the re-validation of all packages in a database.
    Use of db2rbind is not mandatory.
    For packages that are invalid, you can choose to allow package revalidation to occur implicitly when the package is first used. You can choose to selectively revalidate packages with either the REBIND or the BIND command.
    If the rebind of any of the packages encounters a deadlock or a lock timeout the rebind of all the packages will be rolled back.
    If you issue the db2rbind command and the instance is active, you will see the SQL1026N error message.
    Every compiled SQL object has a dependent package. The package can be rebound at any time by using the REBIND_ROUTINE_PACKAGE procedure. Explicitly rebinding the dependent package does not revalidate an invalid object. Revalidate an invalid object with automatic revalidation or explicitly by using the ADMIN_REVALIDATE_DB_OBJECTS procedure. Object revalidation automatically rebinds the dependent package.

BIND command

Invokes the bind utility, which prepares SQL statements stored in the bind file generated by the precompiler, and creates a package that is stored in the database.
Scope

This command can be issued from any database partition in db2nodes.cfg. It updates the database catalogs on the catalog database partition. Its effects are visible to all database partitions.

Usage notes

Binding a package using the REOPT option with the ONCE or ALWAYS value specified might change the static and dynamic statement compilation and performance.

Binding can be done as part of the precompile process for an application program source file, or as a separate step at a later time. Use BIND when binding is performed as a separate process.

The name used to create the package is stored in the bind file, and is based on the source file name from which it was generated (existing paths or extensions are discarded). For example, a precompiled source file called myapp.sql generates a default bind file called myapp.bnd and a default package name of MYAPP. However, the bind file name and the package name can be overridden at precompile time by using the BINDFILE and the PACKAGE options.

Binding a package with a schema name that does not already exist results in the implicit creation of that schema. The schema owner is SYSIBM. The CREATEIN privilege on the schema is granted to PUBLIC.

BIND executes under the transaction that was started. After performing the bind, BIND issues a COMMIT or a ROLLBACK to terminate the current transaction and start another one.

Binding stops if a fatal error or more than 100 errors occur. If a fatal error occurs, the utility stops binding, attempts to close all files, and discards the package.
When a package exhibits bind behavior, the following will be true:

    The implicit or explicit value of the BIND option OWNER will be used for authorization checking of dynamic SQL statements.
    The implicit or explicit value of the BIND option QUALIFIER will be used as the implicit qualifier for qualification of unqualified objects within dynamic SQL statements.
    The value of the special register CURRENT SCHEMA has no effect on qualification.

In the event that multiple packages are referenced during a single connection, all dynamic SQL statements prepared by those packages will exhibit the behavior as specified by the DYNAMICRULES option for that specific package and the environment they are used in.

Parameters displayed in the SQL0020W message are correctly noted as errors, and will be ignored as indicated by the message.

If an SQL statement is found to be in error and the BIND option SQLERROR CONTINUE was specified, the statement will be marked as invalid. In order to change the state of the SQL statement, another BIND must be issued . Implicit and explicit rebind will not change the state of an invalid statement. In a package bound with VALIDATE RUN, a statement can change from static to incremental bind or incremental bind to static across implicit and explicit rebinds depending on whether or not object existence or authority problems exist during the rebind.

The privileges from the roles granted to the authorization identifier used to bind the package (the value of the OWNER bind option) or to PUBLIC, are taken into account when binding a package. Roles acquired through groups, in which the authorization identifier used to bind the package is a member, will not be used.

For an embedded SQL program, if the bind option is not explicitly specified the static statements in the package are bound using the FEDERATED_ASYNC configuration parameter. If the FEDERATED_ASYNCHRONY bind option is specified explicitly, that value is used for binding the packages and is also the initial value of the special register. Otherwise, the value of the database manager configuration parameter is used as the initial value of the special register. The FEDERATED_ASYNCHRONY bind option influences dynamic SQL only when it is explicitly set.

The value of the FEDERATED_ASYNCHRONY bind option is recorded in the FEDERATED_ASYNCHRONY column in the SYSCAT.PACKAGES catalog table. When the bind option is not explicitly specified, the value of FEDERATED_ASYNC configuration parameter is used and the catalog shows a value of -2 for the FEDERATED_ASYNCHRONY column.

If the FEDERATED_ASYNCHRONY bind option is not explicitly specified when a package is bound, and if this package is implicitly or explicitly rebound, the package is rebound using the current value of the FEDERATED_ASYNC configuration parameter.


##DB2重命名或者relocation
DB2提供了可供重命名及迁移数据文件的命令：db2relocatedb
```
db2relocatedb -f param.file
param.file内容如下：
DB_NAME=oldName,newName
DB_PATH=oldName,newName
INSTANCE=oldInst,newInst
NODENUM=nodeNumber
LOG_DIR=oldDirPath,newDirPath
CONT_PATH=oldContPath,newContPath
```



Trouble shooting
##如果在restore中遇到
```
SQL2583N  The intended restore command cannot be processed because a previous incremental restore is still in progress.
```
可考虑用incremental abort命令终止此restore动作，然后重新执行restore：
```
[db2inst1@db2srv backup]$ db2 restore db testdb incremental automatic from /data2/backup/ taken at 20150807105110
SQL2583N  The intended restore command cannot be processed because a previous
incremental restore is still in progress.
[db2inst1@db2srv backup]$ db2 restore db testdb incremental abort from /data2/backup/ taken at 20150807105110
SQL2001N  The utility was interrupted.  The output data may be incomplete.
[db2inst1@db2srv backup]$ db2 restore db testdb incremental automatic from /data2/backup/ taken at 20150807105110
```

##create primary constrain counter SQL0542N
```
[db2inst1@db2srv ~]$ db2 "alter table db2inst1.t add constraint t_pk primary key(id)"
DB21034E  The command was processed as an SQL statement because it was not a valid Command Line Processor command.  During SQL processing it returned:
SQL0542N  The column named "ID" cannot be a column of a primary key or unique key constraint because it can contain null values.  SQLSTATE=42831
[db2inst1@db2srv ~]$ db2 "alter table db2inst1.t alter column id set not null"
DB20000I  The SQL command completed successfully.
[db2inst1@db2srv ~]$ db2 "alter table db2inst1.t add constraint t_pk primary key (id)"
DB21034E  The command was processed as an SQL statement because it was not a valid Command Line Processor command.  During SQL processing it returned:
SQL0668N  Operation not allowed for reason code "7" on table "DB2INST1.T".
SQLSTATE=57016
[db2inst1@db2srv ~]$ db2 reorg table db2inst1.t
DB20000I  The REORG command completed successfully.
[db2inst1@db2srv ~]$ db2 "alter table db2inst1.t add constraint t_pk primary key (id)"
DB20000I  The SQL command completed successfully.
```





##Performance tunning tips
1. Lock escalation
  1) Find "lock escalation" warnings in the db2diag log.
  2) Find the db parameter locklist parameter
  3) Find the db parameter maxlocks parameter
  4) Find the dynamic SQL
```
    db2pd -d testdb -locks showlocks wait -tra -app -dyn > db2pd.out
    db2 get snapshot for dynamic sql on testdb
```


2. Taking snapshots
  1) Find out what monitors are turning on (session level)
```
    db2 -v get monitor switches
```
  2) Turn on monitors(session level)
```
    db2 -v update monitor switches using bufferpool on
    db2 -v update monitor switches using lock on
    db2 -v update monitor switches using sort on
    db2 -v update monitor switches using statement on
    db2 -v update monitor switches using table on
    db2 -v update monitor switches using UOW on
    --turn on all monitors (session level)
    db2 update monitor switches using bufferpool on lock on sort on statement on table on timestamp on uow on
    --turn on monitors (instance level)
    db2 update dbm cfg using DFT_MON_BUFPOOL ON
    db2 update dbm cfg using DFT_MON_LOCK ON
    db2 update dbm cfg using DFT_MON_SORT ON
    db2 update dbm cfg using DFT_MON_STMT ON
    db2 update dbm cfg using DFT_MON_TABLE ON
    db2 update dbm cfg using DFT_MON_UOW ON
```


  3) Run the applications
  4) close monitor (instance level)
```
    db2 -v reset monitor all
    --database level
    db2 reset monitor for database database_name
```

  5) get snapshots
```
    Lock snapshot:      db2 get snapshot for locks on DB_NAME
    DBM snapshot:       db2 get snapshot for dbm
    DB snapshot:        db2 get snapshot for database on DB_NAME
    TBS snapshot:       db2 get snapshot for tablespaces on DB_NAME
    Bufferpool:         db2 get snapshot for bufferpools on DB_NAME
    Applications:       db2 get snapshot for applications on DB_NAME
    Dynamic SQL:        db2 get snapshot for dynamic sql on DB_NAME
    Table:              db2 get snapshot for tables on DB_NAME
```

--Run the following command to take a snapshot and report the read times:
```
db2 get snapshot for bufferpools on ODRPTDB |awk '{ if ( $1=="Bufferpool" && $2=="name" ){print $0} if ( index($0, "pool read time")){print "\t"$0}}'
```

--Run the following command to take a snapshot and report the number of logical and physical reads:
```
db2 get snapshot for bufferpools on ODRPTDB | awk '{
if($1=="Bufferpool" && $2=="name"){print$0}
if (index($0,"cal reads")){print "\t"$0}
}' | grep -v temp
```

  6) event monitor
```
    --create Event Monitor mymon1
    db2 "create event monitor mymon1 for database, tables, statements, DEADLOCKS, tablespaces, bufferpools, connections,transactions write to file '/restore'"
    --turn on Event Monitor
    db2 "set event monitor mymon1 STATE=1"
    db2 "select * from t"
    --turn off Event Monitor
    db2 "set event monitor mymon1 STATE=0"
    --the Event Monitor is dropped
    db2 "drop event monitor mymon1"
    --analyze with command db2evmon
    db2evmon -path /restore
```

    Check Evnet Monitor State :
--the Function event_mon_state() can be used to check the monitor status, to see if it is active or inactive.
    Example:
    The following example selects all of the defined event monitors, and indicates whether each is active or inactive:
```
    db2 "SELECT EVMONNAME,CASE WHEN EVENT_MON_STATE(EVMONNAME) = 0 THEN 'In-active' WHEN EVENT_MON_STATE(EVMONNAME) = 1 THEN 'Active' END FROM SYSCAT.EVENTMONITORS"
```


3. How to diagnose high CPU
  1) using db2pd to find out top 5 high CPU
```
  db2pd -edus interval=10 top=5
```
  2) using the EDU ID to find out the application handle
```
  db2pd -agents EDU_ID
```
  3) db2 get snapshot for application agentid Appl.Handle
    Some of the typical reasons why an application consumes a lot of CPU time are: frequent query compilation; too many sorts or large sorts; too many table scans.
-- finding most sort SQL in dynamic SQL
```
db2 "select STMT_SORTS,SORTS_PER_EXECUTION,SUBSTR(STMT_TEXT,1,60) AS STMT_TEXT from sysibmadm.TOP_DYNAMIC_SQL order by STMT_SORTS desc fetch first 5 rows only"
```

--Query focused on CPU cycles using MON_GET_PKG_CACHE_STMT
```
db2 "
WITH SUM_TAB (SUM_CPU) AS (
        SELECT FLOAT(SUM(TOTAL_CPU_TIME))
        FROM TABLE(MON_GET_PKG_CACHE_STMT ( 'D', NULL, NULL, -2)) AS T)
SELECT
        SUBSTR(STMT_TEXT,1,80) AS STATEMENT,
        TOTAL_CPU_TIME,
        DECIMAL(100*(FLOAT(TOTAL_CPU_TIME)/SUM_TAB.SUM_CPU),5,2) AS PCT_TOT_CPU,
        DECIMAL(FLOAT(TOTAL_CPU_TIME)/FLOAT(NUM_EXECUTIONS),10,2) AS AVG_CPU_TIME_PER_EXE,
        NUM_EXECUTIONS
    FROM TABLE(MON_GET_PKG_CACHE_STMT ( 'D', NULL, NULL, -2)) AS T, SUM_TAB
    ORDER BY TOTAL_CPU_TIME DESC FETCH FIRST 5 ROWS ONLY WITH UR
"
```

4. DIAGNOSING LOCK PROBLEMS
  1) db2 list applications show detail
    Focus on "status","status change time","Appl.Handle"
    Status: A value of Lock-wait means the application is blocked by a lock held by a different application. Don’t be confused by a value of UOW Waiting, which  means that the application (unit of work) is in progress and not blocked by a lock. It is simply not doing any work at the moment.
    Status change time: This is of particular interest for an application with Lock-wait status. The value shows when the lock wait began. Note that the UOW monitor switch must be on for the status change time to be reported.
  2) if needed, force application handle
```
    db2 force application (Appl.Handle,Appl.Handle)
```

5. Explain tool and Explain tables
  1) create Explain table manually
```
    [db2v97i@db2srv NODE0000]$ find /opt/ibm/db2/V10.1_FP5/ -name "EXPLAIN*"
    /opt/ibm/db2/V10.1_FP5/misc/EXPLAIN.DDL
    db2 -tvf EXPLAIN.DDL
```
  2) Explain for query
```
    db2 explain plan with snapshot for "select * from t"
    or:
    db2 set current explain mode yes
    db2 set current explain snapshot yes
    query
    --format the explain table
    db2exfmt
```

6. Design Advisor
  1) Using below SQL to find out bad SQLs:
```
  db2 "SELECT
(case
when num_executions >0 then (rows_read / num_executions)
else 0
end) as avg_rows_read,
(case
when num_executions >0 then (rows_written / num_executions)
else 0
end) as avg_rows_written,
(case
when num_executions >0 then (stmt_sorts / num_executions)
else 0
end) as avg_sorts,
(case
when num_executions >0 then (total_exec_time / num_executions)
else 0
end) as avg_exec_time,
substr(stmt_text,1,200) as SQL_Stmt
FROM table (snapshot_dyn_sql ('DB_NAME', -1) ) as snapshot_dyn_sql "
```
  2) extract the statement text from the output above, and put it into the file bad.sql
  3) db2advis -d DB_NAME -i bad.sql

7. snapshot_ tables
  Snapshot_ table functions: return information from a snapshot, these snapshot can be :
  1) SNAPSHOT_AGENT: Deprecated and replaced by SNAP_GET_AGENT.Returns information about agents from an application snapshot.
  2) SNAPSHOT_APPL: Deprecated and replaced by SNAP_GET_APPL.Returns general information from an application snapshot.
  3) SNAPSHOT_APPL_INFO: Deprecated and replaced by SNAP_GET_APPL_INFO.Returns general information from an application snapshot.
  4) SNAPSHOT_BP: Deprecated and replaced by SNAP_GET_BP.Returns information from a buffer pool snapshot.
  5) SNAPSHOT_CONTAINER: Deprecated and replaced by SNAP_GET_CONTAINER.Returns container configuration information from a table space snapshot.
  6) SNAPSHOT_DATABASE: Deprecated and replaced by SNAP_GET_DB.Returns information from a database snapshot.
  7) SNAPSHOT_DBM: Deprecated and replaced by  SNAP_GET_DBM.Returns information from a snapshot of the DB2® database manager.
  8) SNAPSHOT_DYN_SQL: Deprecated and replaced by SNAP_GET_DYN_SQL.
  9) SNAPSHOT_LOCK: Deprecated and replaced by MON_GET_APPL_LOCKWAIT table function, MON_GET_LOCKS table function, and MON_FORMAT_LOCK_NAME table function.
  10) SNAPSHOT_LOCKWAIT: Deprecated and replaced by MON_GET_APPL_LOCKWAIT table function, MON_GET_LOCKS table function, and MON_FORMAT_LOCK_NAME table function.
  11) SNAPSHOT_STATEMENT: Deprecated and replaced by SNAP_GET_STMT.
  12) SNAPSHOT_TABLE: Deprecated and replaced by SNAP_GET_TAB.
  13) SNAPSHOT_TBREORG: Deprecated and replaced by SNAP_GET_TAB_REORG.Retrieve table reorganization snapshot information.
  14) SNAPSHOT_TBS: Deprecated and replaced by SNAP_GET_TBSP.
  15) SNAPSHOT_TBS_CFG: Deprecated and replaced by SNAP_GET_TBSP_PART.
  Usages:
```
    db2 "select * from table(snapshot_tbs('mysample',-1)) as snapshot_*"
    --mysample means DB_NAME, -1 means current database member, -2 means all active database members.
```


8. snapshot views
Function map elements from sections of get snapshot reports:

| snapshot report | table function             | views                 |
| --------------- | --------------             | ---------             |
| Application     | snap_get_agent             | snapagent             |
|                 | SNAP_GET_AGENT_MEMORY_POOL | snapagent_memory_pool |
|                 | snap_get_appl              | snapappl              |
|                 | SNAP_GET_APPL_INFO         | snapappl_info         |
|                 | snap_get_stmt              | snapstmt              |
|                 | snap_get_subsection        | snapsubsection        |
|                 | snap_get_lockwait          | snaplockwait          |
| Buffer pool     | snap_get_bp                | snapbp                |
|                 | snap_get_bp_part           | snapbp_part           |
| Locks           | snap_get_lock              | snaplock              |
| Tables          | snap_get_tab_v91           | snaptab               |
|                 | snap_get_tab_reorg         | snaptab_reorg         |
| Dynamic SQL     | snap_get_dyn_sql_v91       | snapdyn_sql           |
| Database        | snap_get_db_v91            | snapdb                |
|                 | snap_get_detaillog_v91     | snapdetaillog         |
|                 | snap_get_db_memory_pool    | snapdb_memory_pool    |
|                 | snap_get_hadr              | snaphadr              |
|                 | snap_get_storage_paths     | snapstorage_paths     |
| Tablespace      | snap_get_tbsp_v91          | snaptbsp              |
|                 | snap_get_tbsp_part_v91     | snaptbsp_part         |
|                 | snap_get_container_v91     | snapcontainer         |
|                 | snap_get_tbsp_quiescer     | snaptbsp_quiescer     |
|                 | snap_get_tbsp_range        | snaptbsp_range        |
| DBM             | snap_get_dbm               | snapdbm               |
|                 | snap_get_dbm_memory_pool   | snapdbm_memory_pool   |
|                 | snap_get_fcm               | snapfcm               |
|                 | snap_get_fcm_part          | snapfcm_part          |
|                 | snap_get_switches          | snapswitches          |


##DB2 utilities
1. load query--finding table status
load query used to obtain the table state.
```
[db2inst1@db2srv ~]$ db2 load query table t
Tablestate:
  Normal
```

2. list utilities
This command display the list of active utilities on the instance:
```
db2 list utilities show detail
```

3. db2relocatedb
db2 set write suspend for database
This utility renames a database or relocates a database in a parameter file, you can alter the following properties of a database:
1) the database name
2) the instance belongs to
3) the database directory
4) the database partition number
5) the log directory
6) the location of table space containers

db2relocatedb -f param.file
param.file内容如下：
```
DB_NAME=oldName,newName
DB_PATH=oldName,newName
INSTANCE=oldInst,newInst
NODENUM=nodeNumber
LOG_DIR=oldDirPath,newDirPath
CONT_PATH=oldContPath,newContPath
```

example:
A. Change db name(database SAMPLE1 belongs to db2inst1, and the db path is under the /home/db2inst1, just change the db name from SAMPLE1 to SAMPLE)
```
[db2inst1@db2srv ~]$ db2 list db directory|grep -i sample -A 1
 Database alias                       = SAMPLE1
 Database name                        = SAMPLE1
 Local database directory             = /home/db2inst1
[db2inst1@db2srv ~]$ cat relocate.txt
DB_NAME=sample1,sample
DB_PATH=/home/db2inst1
INSTANCE=db2inst1
[db2inst1@db2srv ~]$ db2relocatedb -f relocate.txt
Files and control structures were changed successfully.
Database was catalogued successfully.
DBT1000I  The tool completed successfully.
[db2inst1@db2srv ~]$  db2 list db directory|grep -i sample -A 1
 Database alias                       = SAMPLE
 Database name                        = SAMPLE
 Local database directory             = /home/db2inst1
```

B. To move the testdb database from the instance db2inst1 on path /db2/data to db2inst2 on the same path, do the following:
a) Move the all the files in /db2/data/db2inst1 to /db2/data/db2inst2
b) Edit the relocate parameter file
```
DB_NAME=SAMPLE
DB_PATH=/db2/data
INSTANCE=db2inst1, dn2inst2
```

4. db2look
db2look extracts DDL of database objects, beside that, the tool can also generate the following:
1) UPDATE statistics statements
2) DCL, such as GRANT, REVOKE
3) UPDATE command for DBM configuration

5. runstats
The RUNSTATS utility updates statistics about the physical characteristics of a table and the associated indexes.Characteristics include the number of records (cardinality), the number of pages, the average record length, and so on.

6. reorg and reorgchk
  runstats=>reorgchk=>reorg=>runstats=>rebind
  1) offline reorg (default)
  2) online (inplace) reorg

only for needed no index:
```
db2 -x "call sysproc.REORGCHK_TB_STATS('T','ALL')"|grep "\*"|awk '{print "REORG TABLE "$1"."$2";";print "RUNSTATS on table "$1"."$2" and indexes all"";"}' > reorg_runstat.ddl

db2 -v "reorgchk current statistics on table all" |grep -v "SYSIBM" |grep -v "SYSCAT" |grep -v "SYSSTAT" |grep -v "SYSTOOLS">>db2reorgchk.20160417
```

7. db2pd
The tool collects information without acquiring any latches or using any engine resources.It is therefore possible (and expected) to retrieve information that is changing while db2pd is collecting information; hence the data might not be completely accurate.
  1) Find the wait locks
```
    db2pd -d DB_NAME -wlocks
    db2pd -d DB_NAME -apinfo APPHandle
    db2pd -d DB_NAME -locks -transactions -applications -dynamic

    SELECT SUBSTR(TABSCHEMA,1,8) AS TABSCHEMA,
    SUBSTR(TABNAME,1,15) AS TABNAME,
    LOCK_OBJECT_TYPE, LOCK_MODE, LOCK_MODE_REQUESTED,
    AGENT_ID_HOLDING_LK
    FROM SYSIBMADM.LOCKWAITS
```

  2) Monitoring memory Usage
```
    db2pd -memblock
    db2pd -memb pid=
```
  3) tablesapce
```
    db2pd -d DB_NAME -tcbstats
    db2pd -d DB_NAME -tablespaces
```
  4) dynamic SQL
```
    db2pd -d DB_NAME -dyn
```
  5) agent
```
    db2pd -agent
```
  6) monitoring recovery
```
    db2pd -d DB_NAME -recovery
```
  7) Determining the amount of resources a transaction is using
```
    db2pd -d DB_NAME transactions
```
  8) monitoring log Usage
```
    db2pd -d DB_NAME -logs
```
  9) Monitoring the progress of index reorganization
```
    db2pd -d mysample -reorgs index
```
  10)  Displaying the top EDUs by processor time consumption and displaying EDU stack information
```
    db2pd -edus interval=5 top=5
```
  11) monitoring runstats
```
    db2pd -d DB_NAME runstats
```
  12) db2pd -d mysample -dbcfg

  13) restart the tsm vendor process:
```
  db2pd -d db_name -fvp LOGARCHMETH1 term
```

##generate duplicate database DDL by using db2look
```
  db2look -d mysample -a -e -m -l -x -f -o mysample.sql
  --edit output file, modify database names, container path, and adjust the tbs size
```


##Data movement
--duplicate table data
```
db2 "create table t1 like t"
db2 "insert into t1 select * from t"
```
IXF formatted file contains tables and indexes defination, but not contains constraint violation check defination
1. export import --essentially, export/import just a set of SQL, INSERT,UPDATE OR DELETE, so it may generate lots of logs
  --export table to ixf format
```
  db2 "export to t1.ixf of ixf messages export.out select * from t1 "
```
  --import from an ixf format file
```
  db2 "import from t1.ixf of ixf messages import.out replace_create into t2"
```
  there are 5 methods to import into a table: INSERT,INSERT_UPDATE,REPLACE, above 3 methods only can be used when target table exist. The last 2 are: REPLACE_CREATE and CREATE, can be used when table target table not exist.
  1) INSERT:   only append data to target table, do not change any data in the target table.
  2) INSERT_UPDATE:  insert data into target table, but when the data confilct with target table, will update the data. Only use when the target table have PK constraint.
  3) REPLACE: delete exsist data and insert new one.
  4) REPLACE_CREATE: like REPLACE, delete exsist data, if target table or index do not exsist, then create it.
  5) CREATE: create target table or index, can specify new tablespaces

  some parameters within import utilities need to pay attention to:
  1) Access method
    when loading data from a file by using import, you can specify whether the target table can be accessed or not when the import is in progress
    ALLOW NO ACCESS: will lock target table with X lock, so other application cannot access this table while import happened.
    ALLOW WRITE ACCESS:
  2) COMMITCOUNT  n
    every import n records will commit.
  3) RESTARTCOUNT n
    when import failed, you can specify from which record to continue. For example, if you were importing 500 records table while import terminated by accident and still imported 400 records, you can using RESTARTCOUNT 401, and import can be continue starts from the 401  record.

2. load--load skip trigger and log mechanism, it load formatted data into database directly, so it's more efficiency then export/import
  The load process can be divided into 4 parts: LOAD, BUILD, DELETE, INDEX COPY:
  1) LOAD
    during LOAD phase, data is loaded into table, index keys and table statistics are collected
  2) BUILD
    during BUILD phase, indexes are produced based on the index keys collected during LOAD phase. The index keys are sorted during the LOAD phase, and index statistics are collected. The statistics are similar to those collected through the RUNSTATS command
  3) DELETE
    druing the DELETE phase, the rows data that caused a unique or primary key violation are removed from the table, the deleted rows are stored in the load exception table, if one was specified.
  4) INDEX COPY
    during the INDEX COPY phase, the index data is copied from a system temporary tablespace to original tablespace, this will only occur if a system temp tablespace was specified for index creation during load operation with the read access option specified.
  When loading data, the following information must be required:
  1) the path and the input file name
  2) the name of target table
  3) the format of input source, like IXF, ASC, DEL
  4) whether the input data is to be appended to the table or replace the data
  Load methods:
  1) INSERT: just append data into target table without any changing to the exsiting data
  2) REPLACE: deletes exsiting data from the table and populcates it with input data
  3) RESTART: when an interruptted occur, in most case, the load utility can resume from the phase it failed in
  4) TERMINATE: a failed load operation is rolled back
  Monitoring load process
  1) db2 list utilities
  2) db2 load query table schema_name.table_name

3. db2move--have 3 modes, EXPORT,IMPORT,LOAD and COPY, COPY mode is for duplication of schema. Only extract data, not DDL.
The tool queries the system catalog tables for a particular database and compiles a list of all user tables. It then exports these tables in PC/IXF format. The PC/IXF files can be imported or loaded to another local DB2 database on the same system, or can be transferred to another workstation platform and imported or loaded to a DB2 database on that platform. Tables with structured type columns are not moved when this tool is used.
  1) Command syntax: action must be EXPORT, IMPORT, LOAD OR COPY.
```
                                .----------------------------.
                            V                            |
>>-db2move--dbname--action----+------------------------+-+-----><
                              +- -tc--table-definers---+
                              +- -tn--table-names------+
                              +- -sn--schema-names-----+
                              +- -ts--tablespace-names-+
                              +- -tf--filename---------+
                              +- -io--import-option----+
                              +- -lo--load-option------+
                              +- -co--copy-option------+
                              +- -l--lobpaths----------+
                              +- -u--userid------------+
                              +- -p--password----------+
                              '- -aw-------------------' --allow waring
```
  2) Examples
    --export all tables in the MYSAMPLE database, be careful, when default options is used, this command can generate lots of files under the current folder.
```
    db2move mysample export
    --load/import all tables in the mysample database, lobpath and /tmp are used for search LOB objects
    db2move mysample load/import -l /home/db2v97i/lobpath,/tmp
    --duplicate schema db2v97i to fungkong
    db2move mysample copy -sn db2v97i -co target_db sample USER myuser1 using mypass1 SCHEMA_MAP \(\(db2v97i,fungkong\)\) TABLESPACE_MAP ((ts1,ts2), SYS_ANY))
```

4. db2look--extract DDL
  db2look extracts DDL of database objects, beside that, the tool can also generate the following:
  1) UPDATE statistics statements
  2) DCL, such as GRANT, REVOKE
  3) UPDATE command for DBM configuration
```
  db2look -d mysample -e -a -o db2look_mysample.sql
```

5. heterogeneous migration of db2 by using db2move and db2look
  1) export all tables in the source db
```
    db2move source_db_name export
```
  2) extracts all DDL from source db
```
    db2look -d source_db_name -e -a -o db2look_mysample.sql
```
  3) transfer all the files that generated by db2move, involving all *.ixf files, db2move.lst and db2look generated files
  4) create target db in the target server
```
    db2 create db target_db_name
```
  5) create objects defination by using db2look generated file
```
    db2 connect to target_db_name
    db2 -tvf db2look_mysample.sql
```
  6) load the data into target db
```
    db2move target_db_name load
```
  7) constraints violation check
```
    db2 connect to target_db_name
    db2 set integrity for schema.table immediate checked
```


###find the special build version of db2 source code
```
cd $CODEDIR/.metadata/BASE_CLIENT_R/
cat spec
$ ccat spec
arch=x86_64
baseptf=IP23657
bldlevel=20141015
ext_buildlevel=s141015
fixpack=10
interim=
minimum_version=9.7.0.0
os=Linux
ptf=IP23657
sb=0
specialbuild=
vrmf=9.7.0.10
```

###db2updv97: sqlcode -912
```
error:
/opt/ibm/db2/V9.7_FP11_SB_35317/bin/db2updv97 -d mlmdb |tee -a db2updv97.mlmdb.out
MESSAGE: Error creating system module : SYSIBMADM.MONREPORT_LOCKWAIT
MESSAGE: Rolling back work due to sqlcode: -912 on line 998.
db2updv97 processing failed for database 'mlmdb'.
```

Solution:
Cause:
Any one of the following may be the reason:
1) The maximum number of lock requests has been reached for the database.
2) The whole database and it's active and archive logs are using one rather small file system.

Diagnosing the problem
1) Check the value assigned to LOCKLIST using below command:
```
$ db2 get db cfg for SAMPLE | find /i "LOCKLIST"
```
2) File system on which database, its active and archive logs is same and smaller one.

Resolving the problem
One of the following is recommended:
1) The maximum number of locks for the database was reached because insufficient memory was allocated to the lock list.
Consider increasing the database configuration parameter (locklist) to allow more lock list space.
You might want to double the locklist value and rerun the db2updv97 program.
For Example :
Update LOCKLIST using the command:
$ db2 "update db cfg for SAMPLE using LOCKLIST <new_value>".
2) Either increase the file system on which the whole database and it's active and archive logs resides, or delete the old archived logs which are not required.


###Finding the table stataus
```
db2 "select substr(tabschema,1,10) as owner,substr(tabname,1,10) as tabname,status from syscat.tables where owner='SYSCAT'"
```

###Finding all database path
```
db2 "select DBPARTITIONNUM, substr(TYPE,1,15) as TYPE, substr(PATH,1,70) as PATH from sysibmadm.DBPATHS"
```



###Finding db2level via sys views
```
db2 "select substr(inst_name,1,20) as inst_name, substr(service_level,1,20) as db_level,substr(bld_level,1,15) as bld_level,fixpack_num from SYSIBMADM.ENV_INST_INFO"
```

### Discovering and hiding server instances and databases
To allow clients to discover server instances on a system, set the discover_inst database manager configuration parameter in each server instance on the system to ENABLE (this is the default value). Set this parameter to DISABLE to hide this instance and its databases from Discovery.

To allow a database to be discovered from a client, set the discover_db database configuration parameter to ENABLE (this is the default value). Set this parameter to DISABLE to hide the database from Discovery.

```
$ db2 get dbm cfg |grep -i discover_inst
 Discover server instance                (DISCOVER_INST) = ENABLE
```

### Finding all tables status
```
Select substr(tabschema,1,8) as "Qualified Name",
substr(tabname,1,50) as "Table name",
CASE type
WHEN 'A' THEN 'Alias'
WHEN 'H' THEN 'Hierarchy Table'
WHEN 'N' THEN 'Nickname'
WHEN 'S' THEN 'Summary Table'
WHEN 'T' THEN 'Table'
WHEN 'U' THEN 'Typed Table'
WHEN 'V' THEN 'View'
WHEN 'W' THEN 'Typed View'
END as "Table Type",
CASE status
WHEN 'N' THEN 'Normal'
WHEN 'C' THEN 'Check Pending'
WHEN 'X' THEN 'Inoperative'
END as "Table Status"
from syscat.tables;
```

### health monitor -- tablespace
```
db2 get health monitor snapshot for database on db_name
```


### Log utilization
```
db2 "select * from sysibmadm.log_utilization"
```

# Windows server operations
### Multiple instances switch
When there are multiple DB2 copies on the same computer, the PATH environment variable can only point to one of them: the DEFAULT COPY. For example, if DB2COPY1 is under c:\sqllib\bin and is the default copy; and DB2COPY2 is under d:\sqllib\bin. If you want to use DB2COPY2 in a regular command window, you would run d:\sqllib\bin\db2envar.bat in that command window. This adjusts the PATH (and some other environment variables) for this command window so that it will pick up binaries from d:\sqllib\bin.
DB2INSTANCE is only valid for instances under the DB2 copy that you are using. However, if you switch copies by running the db2envar.bat command, DB2INSTANCE will be updated to the value of DB2INSTDEF for the DB2 copy that you switched to initially.
DB2INSTANCE is the current DB2 instance that will be used by applications that are executing in that DB2 copy. When you switch between copies, by default, DB2INSTANCE is changed to the value of DB2INSTDEF for that copy. DB2INSTDEF is less meaningful on a one copy system because all the instances are in the current copy; however, it is still applicable as being the default instance, if another instance is not set.

### Issues
Attempts to open a DB2 Command Window or DB2 Command Line Processor fail with:
DB2CMD fails with 'C:\Program' is not recognized as an internal or external command, operable program or batch file.

Resolving the problem

Windows support for 8.3 short names is normally enabled by default, and is controlled by this registry key:

```
    HKLM\SYSTEM\CurrentControlSet\Control\FileSystem\NtfsDisable8dot3NameCreation
```


8.3 short names are enabled if it is set to 0 or 2.

Setting the above key to 0 will not create 8.3 short names for existing directories that were created when 8.3 short names was disabled. Unfortunately, if DB2 was installed when 8.3 short names was disabled, DB2 must be re-installed after 8.3 short names have been enabled in Windows.

Windows 7 and later versions include the 'fsutil' command that can be used to create the 8dot3 name once the registry key is set. For example,


```
    fsutil file setshortname "C:\Program Files\IBM\DB2" PROGRA~1
```


The use of the 'fsutil' command will avoid the requirement for re-installing DB2.

This problem can also be avoided by alternately installing DB2 in a directory that does not contain spaces.



##Converting SMS tablespace to DMS tablespace(Migrating the data from one tablespace to another tablespace)
```
[db2inst1@db2srv ~]$ db2 "select substr(TABLE_SCHEMA,1,10) as schema,substr(TABLE_NAME,1,15) as tabname,substr(TABLESPACE_NAME,1,15) as tbsname from sysibmadm.dba_all_tables where TABLESPACE_NAME='USERSPACE1'"
SCHEMA     TABNAME         TBSNAME
---------- --------------- ---------------
DB2INST1   CL_SCHED        USERSPACE1
DB2INST1   DEPARTMENT      USERSPACE1
DB2INST1   EMPLOYEE        USERSPACE1
DB2INST1   EMP_PHOTO       USERSPACE1
DB2INST1   EMP_RESUME      USERSPACE1
DB2INST1   PROJECT         USERSPACE1
DB2INST1   PROJACT         USERSPACE1
DB2INST1   EMPPROJACT      USERSPACE1
DB2INST1   ACT             USERSPACE1
DB2INST1   IN_TRAY         USERSPACE1
DB2INST1   ORG             USERSPACE1
DB2INST1   STAFF           USERSPACE1
DB2INST1   SALES           USERSPACE1
DB2INST1   STAFFG          USERSPACE1
DB2INST1   ADEFUSR         USERSPACE1
FUNG       CL_SCHED        USERSPACE1
FUNG       DEPARTMENT      USERSPACE1
FUNG       EMPLOYEE        USERSPACE1
FUNG       EMP_PHOTO       USERSPACE1
FUNG       EMP_RESUME      USERSPACE1
FUNG       PROJECT         USERSPACE1
FUNG       PROJACT         USERSPACE1
FUNG       EMPPROJACT      USERSPACE1
FUNG       ACT             USERSPACE1
FUNG       IN_TRAY         USERSPACE1
FUNG       ORG             USERSPACE1
FUNG       STAFF           USERSPACE1
FUNG       SALES           USERSPACE1
FUNG       STAFFG          USERSPACE1
FUNG       ADEFUSR         USERSPACE1
```
## Option 1. Extract the whole DDLs by the specific schema
```
db2look -d testdb -z FUNG -e -o fung_schema.sql
db2look -d testdb -z DB2INST1 -e -o db2inst1_schema.sql
```
## Option 2. Extract the whole DDLs by the specific tablespace
```
db2 "select 'db2look -d testdb -t '|| rtrim(TABSCHEMA) ||'.'||'\"'||TABNAME||'\"'|| ' -e' from syscat.tables where TBSPACEID='2'" > db2look.sql
./db2look.sql >o.txt
sed -i '/^COMMIT\ WORK/d' o.txt
sed -i '/^CONNECT\ RESET/d' o.txt
sed -i '/^TERMINATE/d' o.txt
```

## 2. Export all the data belong to the TBS
```
mkdir -p db2move; cd db2move
db2move testdb export -ts USERSPACE1
```

## 3. Create new tbs
```
db2 "create tablespace convert1"
```

## 4. Drop the tbs
```
db2 "drop tablespace userspace1"
```

## 5. Replace the tablespace to convert1 in the fung_schema.sql and db2inst1_schema.sql
```
sed -i '/USERSPACE1/s/USERSPACE1/CONVERT1/g' fung_schema.sql
sed -i '/USERSPACE1/s/USERSPACE1/CONVERT1/g' db2inst1_schema.sql
```

## 6. Create the table
```
db2 connect to testdb
db2 -tvf fung_schema.sql
db2 -tvf db2inst1_schema.sql
```

## 7. Load the data back to the new tablespace
```
[db2inst1@db2srv db2move]$ db2move testdb load -l ./
```

## 8. Set integrity
### Finding the table status
```
Select substr(tabschema,1,8) as "Qualified Name",
substr(tabname,1,50) as "Table name",
CASE type
WHEN 'A' THEN 'Alias'
WHEN 'H' THEN 'Hierarchy Table'
WHEN 'N' THEN 'Nickname'
WHEN 'S' THEN 'Summary Table'
WHEN 'T' THEN 'Table'
WHEN 'U' THEN 'Typed Table'
WHEN 'V' THEN 'View'
WHEN 'W' THEN 'Typed View'
END as "Table Type",
CASE status
WHEN 'N' THEN 'Normal'
WHEN 'C' THEN 'Check Pending'
WHEN 'X' THEN 'Inoperative'
END as "Table Status"
from syscat.tables;

[db2inst1@db2srv ~]$ db2 -tvf finding_all_table_status.sql |grep -v Normal
[db2inst1@db2srv ~]$ db2 "select 'set integrity for '|| rtrim(tabschema)||'.'||tabname|| ' IMMEDIATE CHECKED;' from syscat.tables where status !='N'">setint.sql
[db2inst1@db2srv ~]$ db2 connect to testdb
[db2inst1@db2srv ~]$ db2 -tvf setint.sql
[db2inst1@db2srv ~]$ db2 -tvf finding_all_table_status.sql |grep -v Normal
```
###If some tables are co-dependents tables, we need to execute the tables as a whole set integrity command

```
SET INTEGRITY FOR <table1>, <table2> IMMEDIATE CHECKED
```


##Privilege
```
db2 -x "select 'grant select on table ' || rtrim(tabschema) || '.' || rtrim(tabname) || ' to user JOHN_DOE' from syscat.tables where tabschema like 'FOO%' and (type = 'T' or type = 'V')" | db2 +p -tv

db2 "SELECT substr(AUTHID,1,10) as AUTHID, PRIVILEGE, substr(OBJECTNAME,1,20) as OBJECTNAME, substr(OBJECTSCHEMA,1,10) as OBJECTSCHEMA, OBJECTTYPE
       FROM SYSIBMADM.PRIVILEGES where authid='LISHUAI'" |more

db2 grant dbadm on database to user
db2 revoke dbadm on database from user
db2 "select * from SYSCAT.DBAUTH  where GRANTEE = 'USER' "

db2 "select char(grantee,8) as grantee, char(granteetype,1) as type, \
char(dbadmauth,1) as dbadm,  char(createtabauth,1) as createtab,  \
char(bindaddauth,1) as bindadd,  char(connectauth,1) as connect,  \
char(nofenceauth,1) as nofence,  char(implschemaauth,1) as implschema,  \
char(loadauth,1) as load,  char(externalroutineauth,1) as extroutine,  \
char(quiesceconnectauth,1) as quiesceconn,  \
char(libraryadmauth,1) as libadm, char(securityadmauth,1) \
as securityadm from  syscat.dbauth order by grantee"


db2 "select substr(authority,1,25) as authority, d_user, d_group,
d_public, role_user, role_group, role_public_role from table(
sysproc.auth_list_authorities_for_authid ('public','g'))as t
order by authority"      "
```

##Binding Packages
```
bind "C:\PROGRA~1\IBM\SQLLIB\bnd\db2schema.bnd" blocking all grant public
bind "C:\PROGRA~1\IBM\SQLLIB\bnd\@db2ubind.lst" blocking all grant public
bind "C:\PROGRA~1\IBM\SQLLIB\bnd\@db2cli.lst" blocking all grant public
db2 bind db2clpcs.bnd blocking all grant public
```


###TSM restart db2vend
```
db2pd -d db_name -fvp lam1 term
```
https://www.ibm.com/support/knowledgecenter/en/SSEPGG_10.1.0/com.ibm.db2.luw.admin.cmd.doc/doc/r0011729.html?cp=SSEPGG_10.1.0


##SQL0551N user do not have the required authorization
After restoring from anther user and didn't set the db2set DB2_RESTORE_GRANT_ADMIN_AUTHORITIES=ON, the current user id would not have the access the previous user ID's table, to resolve this issue, grant the secadm with the previous user id(db2inst1) to current id(db2v97i), grant the dbamin to current id, assumed the two ids are located on the same machine:
```
[db2inst1@db2srv ~]$ . ~db2v97i/sqllib/db2profile
[db2inst1@db2srv ~]$ db2 connect to testdbd
[db2inst1@db2srv ~]$ db2 grant secadm to db2v97i
[db2inst1@db2srv ~]$ db2 GRANT DBADM ON DATABASE TO USER db2v97i
```

### getting the quiesce status of instance/database
```
instgltp@g01acirdb009:/db/instgltp/db2diag$ db2 get snapshot for database on MAXDBPRD |grep -i status
Database status                            = Active
instgltp@g01acirdb009:/db/instgltp/db2diag$ db2 get snapshot for dbm |grep -i status
Database manager status                        = Active
instgltp@g01acirdb009:/db/instgltp/db2diag$
```

### TSM kud issue
1. Use Root user to stop the process(AIX team)
```
/opt/IBM/ITM/bin/itmcmd agent -o instwem1 stop ud
```

2. sudo to instance user and start the kud process.
```
nohup /opt/IBM/ITM/bin/itmcmd agent -o instwem1 start ud &
```


DB2 trace
the only thing I can suggest you is to collect DB2 trace when you will try to reproduce the issue again
something like:
```
db2trc on -f db2trc.dmp
db2 connect to ESWFE1 user a1inoppt using password
db2trc off
db2trc fmt db2trc.dmp db2trc.fmt
db2trc flw db2trc.dmp db2trc.flw
```
when you will be not able to connect due to the same reason


## stop tivoli monitoring before upgrade
```
/opt/IBM/ITM/bin/itmcmd agent -o <instance_name> stop ud
/opt/IBM/ITM/bin/itmcmd agent -o <instance_name> start ud
```

## db2 error code:
```
db2 ? sqlstate -10060
db2 ? SQLnnnn (nnnn = 4 or 5 digit SQLCODE)
db2 ? nnnnn (nnnnn = 5 digit SQLSTATE)

db2diag -rc 2029059891
```
## To display all logged error messages containing the DB2 ZRC return code
     0x87040055=-2029780907=SQLD_PRGERR, enter:
 db2diag -g msg:=0x87040055 -l Error



## DB2 privilege catalog view
```
SYSCAT.DBAUTH              Database privileges
SYSCAT.COLAUTH             Table and View Column privileges
SYSCAT.INDEXAUTH           Index privileges
SYSCAT.PACKAGEAUTH         Access Package privileges
SYSCAT.SCHEMAAUTH          Schema privileges
SYSCAT.TABAUTH             Table and view privileges
SYSCAT.TBSPACEAUTH         Table space privileges
```


## snap catalog view
```
db2 "select substr(VIEWSCHEMA,1,10) as viewschema, substr(VIEWNAME,1,20) as viewname from syscat.views where owner='SYSIBM' and viewname like  '%SNAP%'"
VIEWSCHEMA VIEWNAME
---------- --------------------
SYSIBMADM  SNAPLOCK
SYSIBMADM  SNAPAGENT
SYSIBMADM  SNAPAGENT_MEMORY_POO
SYSIBMADM  SNAPAPPL
SYSIBMADM  SNAPAPPL_INFO
SYSIBMADM  SNAPBP
SYSIBMADM  SNAPBP_PART
SYSIBMADM  SNAPCONTAINER
SYSIBMADM  SNAPDB
SYSIBMADM  SNAPDB_MEMORY_POOL
SYSIBMADM  SNAPDBM
SYSIBMADM  SNAPDBM_MEMORY_POOL
SYSIBMADM  SNAPDETAILLOG
SYSIBMADM  SNAPDYN_SQL
SYSIBMADM  SNAPFCM
SYSIBMADM  SNAPFCM_PART
SYSIBMADM  SNAPHADR
SYSIBMADM  SNAPLOCKWAIT
SYSIBMADM  SNAPSTMT
SYSIBMADM  SNAPSTORAGE_PATHS
SYSIBMADM  SNAPSUBSECTION
SYSIBMADM  SNAPSWITCHES
SYSIBMADM  SNAPTAB_REORG
SYSIBMADM  SNAPTAB
SYSIBMADM  SNAPTBSP
SYSIBMADM  SNAPTBSP_PART
SYSIBMADM  SNAPTBSP_QUIESCER
SYSIBMADM  SNAPTBSP_RANGE
SYSIBMADM  SNAPUTIL
SYSIBMADM  SNAPUTIL_PROGRESS
```


## DB2 table size
```
select
  char(date(t.stats_time))||' '||char(time(t.stats_time)) as statstime
  ,substr(t.tabschema,1,8)||'.'||substr(t.tabname,1,24) as tabname
  ,card as rows_per_table
  ,decimal(float(t.npages)/ ( 1024 / (b.pagesize/1024)),9,2) as used_mb
  ,decimal(float(t.fpages)/ ( 1024 / (b.pagesize/1024)),9,2) as allocated_mb
from
  syscat.tables t , syscat.tablespaces b
where t.tbspace=b.tbspace
order by 5 desc with ur
```

## DB2 index size
```
select
  rtrim(substr(i.tabschema,1,8))||'.'||rtrim(substr( i.tabname, 1,24)) as tabname
 ,decimal(sum(i.nleaf)/( 1024 / (b.pagesize/1024)),12,2) as indx_used_per_table_mb
from
   syscat.indexes i, syscat.tables t , syscat.tablespaces b
where
   i.tabschema is not null and i.tabname=t.tabname
   and i.tabschema=t.tabschema and t.tbspace=b.tbspace
group by
   i.tabname,i.tabschema, b.pagesize order by 2 desc with ur
```

# db2dart demo
## 1. extract the DDL with db2look
```
db2look -d testdb -e -z SCHEMA_NAME -t TABNAME -o OUTPUT.ddl
```

## 2. finding the specific table resided in which tablespace
```
db2 "select substr(tabname,1,20) as tabname, TBSPACEID from syscat.tables where tabname='TABNAME'"
```

## 3. extract the data with db2dart
```
db2dart testdb /DDEL
>EMPLOYEE,2,0,100
```

## 4. load the data into the table
```
db2 -tvf OUTPUT.ddl
db2 "load from TBALE.DEL of DEL insert into TABNAME"
db2 set integrity for TABNAME immediate checked
```
