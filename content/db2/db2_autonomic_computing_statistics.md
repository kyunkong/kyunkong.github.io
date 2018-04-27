---
title: "DB2 automatically computing statistics"
date: 2018-01-16 16:44:30
---

[TOC]

- Self-tuning memory manager(STMM)
- Automatic storage
- Data compression
- Automatic maintenance
   - Automatic database backup
   - Automatic reorganization
   - Automatic statistics collection
- Configuration advisor
- Utility throttling

##STMM
STMM based on workload change and do not require DBA intervention.
###Enabling the STMM
- Enable self_tuning_mem on
```sql
db2 connect to sample
db2 update db cfg for sample using self_tuning_mem on
```
- Enable memory areas
```sql
db2 connect to sample
db2 "update db cfg for sample using
PCKCACHESZ AUTOMATIC
LOCKLIST AUTOMATIC
MAXLOCKS AUTOMATIC
SORTHEAP AUTOMATIC
SHEAPTHRES_SHR AUTOMATIC
DATABASE_MEMORY AUTOMATIC
"
```
   - Enable new creation buffer pools
       ```sql
       db2 connect to sample
       db2 "create bufferpool bpool8k size AUTOMATIC pagesize 8 k"
       ```
   - Enable existing buffer pools
       ```sql
       db2 connect to sample
       db2 "alter bufferpool bpool4k size AUTOMATIC"
       ```
###Disabling STMM
```sql
db2 connect to sample
db2 update db cfg for sample using self_tuning_mem off
```
- Disabling memory areas in STMM
```sql
db2 connect to sample
db2 "update db cfg for sample using
PCKCACHESZ MANUAL
LOCKLIST MANUAL
MAXLOCKS MANUAL
SORTHEAP MANUAL
SHEAPTHRES_SHR MANUAL
DATABASE_MEMORY MANUAL
"
```
- Disabling STMM of buffer pool
    - Disabling new creation of buffer pool
        ```sql
        db2 connect to sample
        db2 "create bufferpool bpool8k size 1000 pagesize 8 k"
        ```
    - Disabling existing buffer pool
        ```sql
        db2 connect to sample
        db2 "alter bufferpool bpool4k size 1000"
        ```

##Automatic storage
By default, all databases are created with automatic storage.

- How to enable automatic storage manually
    ```sql
    db2 "create database sample automatic storage yes on '/db2data/','/db2data2/', '/db2backup'  dbpath on '/db2data'"
    ```
    There are two ON clause from above create database sql, one is automatic storage, another is dbpath:

- Automatic storage ON
   This ON clause depends on the AUTOMATIC STORAGE option, if the AUTOMATIC STORAGE NO is specified, then only one path can be included, if the path is not specified, the dftdbpath(db cfg parameter) will be used as default database creation path.
   If the AUTOMATIC STORAGE YES is specified, then multiple paths can be specified follow with ON clause, if no path specified, then it will use default storage path, which dftdbpath is specified.

- DBPATH
   DBPATH specified on which path to create the database, if the DBPATH is not specified, then the database will be create on the first path listed in the ON option, if no paths are specified, the database is create on the dftdbpath default path. The database path is a location to store database structures:
- Bufferpool information
- Table space information
- Storage path information
- Database configuration information
- History file information regarding backups, restores, loading of tables, reorganization of tables, altering of table spaces, and other database changes
- Log control files with information about active logs
        It is suggested that the DBPATH ON option be used when automatic storage is enabled to keep the database information separate from the database data.
###For AUTOMATIC STORAGE databases
- The system catalog table space (SYSCATSPACE) will use the directory /AS_PATH/$instance/NODExxxx/$DBNAME/T0000000
- The system temporary table space (TEMPSPACE1) will use the directory T0000001
- The default user table space (USERSPACE1) will use the directory T0000002
###ADMIN_GET_STORAGE_PATHS administrative view(DB2 V10.1)
```sql
db2 "SELECT VARCHAR(STORAGE_GROUP_NAME, 30) AS STOGROUP, VARCHAR(DB_STORAGE_PATH, 40) AS STORAGE_PATH FROM TABLE(ADMIN_GET_STORAGE_PATHS('',-1)) AS T"
```
##Data compression
- Table compression
```sql
db2 "create table sales compress yes"
```
- Index compression
By default, index compression is enabled for compressed tables, and disabled for uncompressed tables.
```sql
db2 "ALTER INDEX DB2ADMIN.IDX_ABSTRCOMPLISTSPEC_KINDID COMPRESS YES"
```
- Backup compression
```sql
db2 "backup database sample to '/db2backup' compress"
```

##Automatic maintenance
```sql
db2 get db cfg for mysample |grep -i AUTO_
Automatic maintenance                      (AUTO_MAINT) = ON
Automatic database backup            (AUTO_DB_BACKUP)   = OFF
Automatic table maintenance          (AUTO_TBL_MAINT)   = ON
Automatic runstats                  (AUTO_RUNSTATS)     = ON
Real-time statistics            (AUTO_STMT_STATS)       = ON
Statistical views              (AUTO_STATS_VIEWS)       = OFF
Automatic sampling                (AUTO_SAMPLING)       = OFF
Automatic statistics profiling    (AUTO_STATS_PROF)     = OFF
Statistics profile updates        (AUTO_PROF_UPD)       = OFF
Automatic reorganization               (AUTO_REORG)     = ON
```

##Configuration advisor
Using <code>autoconfigure</code> command to generate advise values, the APPLY NONE means view the configuration recommendation but not apply them.
```sql
db2 "AUTOCONFIGURE USING
MEM_PERCENT 60
WORKLOAD_TYPE MIXED
NUM_STMTS 500
ADMIN_PRIORITY BOTH
IS_POPULATED YES
NUM_LOCAL_APPS 0
NUM_REMOTE_APPS 20
ISOLATION RR
BP_RESIZEABLE YES
APPLY NONE
"
```


