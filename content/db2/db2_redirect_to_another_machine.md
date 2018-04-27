---
title: "Redirect DB2 to another machine"
date: 2018-01-16 17:33:53
---
[TOC]

Target:
host-> hadr01 
inst-> db2inst1
db-> testdb

Source:
host-> db2srv
inst-> db2inst1
db-> testdb

##1. online backup source database 
```
[db2inst1@db2srv ~]$ db2 "select * from t"

ID          NAME                
----------- --------------------
      51369 Fung                

[db2inst1@db2srv ~]$ db2 backup db testdb online to /db2/backup/db2inst1/testdb/
Backup successful. The timestamp for this backup image is : 20151209190441
[db2inst1@db2srv ~]$ ll /db2/backup/db2inst1/testdb/
total 131576
-rw------- 1 db2inst1 db2adm 134733824 Dec  9 19:04 TESTDB.0.db2inst1.NODE0000.CATN0000.20151209190441.001
[db2inst1@db2srv ~]$ db2 "insert into t values('51368','Kong')"
DB20000I  The SQL command completed successfully.
[db2inst1@db2srv ~]$ db2 "select * from t"

ID          NAME                
----------- --------------------
      51369 Fung                
      51368 Kong 
```

## 2. generate redirect script from source db
```
[db2inst1@db2srv testdb]$ db2 restore db testdb from /db2/backup/db2inst1/testdb/ taken at 20151209190441 redirect generate script ~/redirect.clp
DB20000I  The RESTORE DATABASE command completed successfully.
-- rebuild tablespace
db2 "restore db mysample rebuild with tablespace (SYSCATSPACE,FUNG) from /db2/backup/db2v97i/mysample/ taken at 20160225201758 redirect generate script redirect.clp"

db2 restore db mydb rebuild with all tablespaces in image except tablespace (T0999, T1000) taken at BK1 without prompting
```


##3. scp files from source db to target db
before scp those files, you need create necessary directories on target server, such as active log directory and archive log directory
1) backup files
2) redirect scripts
3) active logs and archive logs :
```
[db2inst1@db2srv NODE0000]$ db2 get db cfg |grep -i LOGARCHMETH1
 First log archive method                 (LOGARCHMETH1) = DISK:/db2/arch/db2inst1/testdb/
[db2inst1@db2srv NODE0000]$ db2 get db cfg |grep -i log |grep -i path
 Path to log files                                       = /db2/log/db2inst1/testdb/NODE0000/
```

##4. executes the redirect script on target db
```
db2 -tvf redirect.clp
```

##5. rollforward to end of logs and stop
```
db2 "rollforward database testdb to end of logs and stop"
```

##6. verify restore result

```
[db2inst1@hadr01 testdb]$ db2 "rollforward database testdb to end of logs and stop"

                                 Rollforward Status

 Input database alias                   = testdb
 Number of nodes have returned status   = 1

 Node number                            = 0
 Rollforward status                     = not pending
 Next log file to be read               =
 Log files processed                    = S0000001.LOG - S0000003.LOG
 Last committed transaction             = 2015-12-09-11.44.06.000000 UTC

DB20000I  The ROLLFORWARD command completed successfully.
[db2inst1@hadr01 testdb]$ db2 connect to testdb

   Database Connection Information

 Database server        = DB2/LINUXX8664 9.7.10
 SQL authorization ID   = DB2INST1
 Local database alias   = TESTDB

[db2inst1@hadr01 testdb]$ db2 "select * from t"

ID          NAME                
----------- --------------------
      51369 Fung                
      51368 Kong                

  2 record(s) selected.

```

