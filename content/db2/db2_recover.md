---
title: "DB2 recovery"
date: 2018-01-16 16:44:30
---
[TOC]
## rollforward usage
**query status**           : query current database status, to see if in rollforward pending.
**stop/complate**          : stop the rolling forward of log records, and completes the rollforward recovery process by rolling back incomplete transactions and turning off the rollforward pending state of the database.
**cancel**                 : cancels the rollforward recovery operation. This puts the database in restore pending state.
**ONLINE**                 : Tablespace level recovery to be done online. Makes the database can be accessible while rollforward in progress.

## db2 backup
```
db2 "
backup database DBNAME [tablespace TBSNAME] [online]
[with num-buff buffers] [buffer buffer-size]
to /dir
compress [include|exclude logs]
```

## delete the backup
```
db2 prune history *timestamp* with force option and delete
```

## db2 restore
```
db2 restore db DBNAME from BKDIR taken at TIMESTAMP with 4 buffers buffer 2048 parallelism 2
db2 "restore db DBNAME tablespace(TBSNAME) from ..."
```

### restore with redirect option

## db2 rollforward
```
db2 "rollforward database DBNAME to end of logs tablespace(TBSNAME) online overflow log path(/overflowpath)"
```
**The overflow path is db2 looking for logs where rollforward needed**

## db2ckbkp and db2flsn
```
db2ckbkp -l backup_images
```
above command and help to find out the Log Sequence Number, with db2flsn, we can find the db2 log name:

```
[db2inst1@db2srv db2backup]$ db2ckbkp -l TESTDB.0.db2inst1.NODE0000.CATN0000.20160830175741.001 |grep -i lsn
lowtranlsn = 0000000006590010
minbufflsn = 0000000006590010
headlsn = 0000000006590010
[db2inst1@db2srv db2backup]$ db2flsn -db testdb 0000000006590010
Given LSN is contained in log page 1 in log file S0000013.LOG.
```

### Recovery a dropped table in db2 with dropped table recovery
```
[db2inst1@db2srv db2backup]$ db2 backup db testdb online 
[db2inst1@db2srv db2backup]$ db2 "select substr(TBSPACE,1,20) as TBS,TBSPACETYPE,DROP_RECOVERY from SYSCAT.TABLESPACES"
[db2inst1@db2srv db2backup]$ db2 drop table fung.t
[db2inst1@db2srv db2backup]$  db2 list history dropped table all for testdb
[db2inst1@db2srv db2backup]$ db2 "restore database testdb tablespace(DMSTS1) taken at 20160830194056"
DB20000I  The RESTORE DATABASE command completed successfully.
[db2inst1@db2srv db2backup]$ db2 "rollforward database testdb to end of logs tablespace online recover \
dropped table 000000000000661700080004 to /db/db2inst1/db2backup"
db2 => CREATE TABLE "FUNG    "."T" ( "ID" VARCHAR(10) , "NAME" VARCHAR(20) )  IN "DMSTS1"
[db2inst1@db2srv db2backup]$ db2 load from ./NODE0000/data of del insert into fung.t
[db2inst1@db2srv db2backup]$ db2 backup db testdb online to /db/db2inst1/db2backup/
```

### Recovery a dropped/deleted table in db2 with rebuild tablespace
```
[db2inst1@db2srv db2backup]$ db2 backup db testdb online compress include logs
[db2inst1@db2srv db2backup]$ db2 "delete from fung.t"
[db2inst1@db2srv db2backup]$ db2 "select TBSPACE from syscat.tables where tabname='T'"
[db2inst1@db2srv db2backup]$ db2 "restore db testdb rebuild with tablespace(SYSCATSPACE, DMSTS1) \
from /db/db2inst1/db2backup taken at 20160830200511 redirect generate script redirect.sql"
```
***Edit the redirect.sql, don't overwrite the current database path***
```
[db2inst1@db2srv db2backup]$ grep -v ^- redirect.sql 
UPDATE COMMAND OPTIONS USING S ON Z ON TESTDB_NODE0000.out V ON;
SET CLIENT ATTACH_DBPARTITIONNUM  0;
SET CLIENT CONNECT_DBPARTITIONNUM 0;
RESTORE DATABASE TESTDB
REBUILD WITH TABLESPACE (
  SYSCATSPACE
, TEMPSPACE1
, DMSTS1
, SYSTOOLSTMPSPACE
)
FROM '/db/db2inst1/db2backup'
TAKEN AT 20160830200511
--modified to different for auto storage path
ON '/db/db2inst1/backup/restore'
INTO newdb
--reset to a new name, and modified the log path
NEWLOGPATH '/db/db2inst1/db2backup/restore/'
REDIRECT
;
SET TABLESPACE CONTAINERS FOR 8
USING (
  FILE   'DMSCONT1'                                                         1000
, FILE   'DMSCONT2'                                                         1000
);
RESTORE DATABASE TESTDB CONTINUE;

[db2inst1@db2srv db2backup]$ db2 rollforward db newdb query status
**copy the necessary log file to the NEWDB log dir**
[db2inst1@db2srv db2backup]$ db2 get db cfg for testdb |grep -i sqlogdir
 Path to log files                                       = /db/db2inst1/testdb/db2inst1/NODE0000/SQL00001/SQLOGDIR/
[db2inst1@db2srv db2backup]$ db2 get db cfg for newdb|grep -i "Path to log files"
 Changed path to log files                  (NEWLOGPATH) = 
  Path to log files                                       = /db/db2inst1/db2backup/restore/
[db2inst1@db2srv db2backup]$ cp /db/db2inst1/arch/db2inst1/TESTDB/NODE0000/C0000001/S0000026.LOG /db/db2inst1/db2backup/restore/
[db2inst1@db2srv db2backup]$ db2 "rollforward db newdb to end of logs and complete"
[db2inst1@db2srv db2backup]$ db2 connect to newdb
[db2inst1@db2srv db2backup]$ db2 "export to fung.t.ixf of ixf select * from fung.t"
[db2inst1@db2srv db2backup]$ db2 connect to testdb
[db2inst1@db2srv db2backup]$ db2 load from fung.t.ixf of ixf insert into fung.t
[db2inst1@db2srv db2backup]$ db2 drop db newdb
```


