---
title: "DB2 data movement"
date: 2018-01-16 16:54:11
---
[TOC]

## moving data between tables
- Insert with subselect
    `db2 "insert into t1 select * from t2"`
- Export
    `db2 "export to output.ixf of ixf select * from table_name"`
- Import
    `db2 "import from input.ixf of ixf insert into table_name"`
- Load from files, pipes and cursors

```
    --from files
    db2 "load from input.ixf of ixf insert into table_name"
    --from cursors
    db2 "create table tt like t in fung"
    db2 "declare cursor1 cursor for select * from t"
    db2 load from cursor1 of cursor messages load.msg insert into tt
    --load status
    db2 "load query table table_name"
```

## capturing DDL and duplicating the table definitions
- Import of IXF file with create option

```
    db2 "export to data.ixf of ixf select * from table_name where 1=0"
    db2 "import from data.ixf of ixf create into table_name"
```
- Create table X like Y
- db2look
- ADMIN_COPY_SCHEMA procedure can copy schema with or without data

## moving/copying databases and tablespaces
- Backup/Restore

```
    old location: /home/db2inst1/db2inst1/NODE0000/SQL00001/
    new location: /db/db2inst1/sample/
    db2 "backup db sample to /backup"
    db2 drop db sample
    db2 "restore db sample from /backup to /db/db2inst1/sample"
    --if the database not dropped, the to clause will be ignored
```
- db2relocatedb
- db2move
- Split mirror database copy

### Using redirect restore to move tablespace containers
```
Database: testdb tablespace: DMSTS1, id=8 type=DMS
old containers: /db/db2inst1/testdb/db2inst1/NODE0000/SQL00001/DMSCONT1
                /db/db2inst1/testdb/db2inst1/NODE0000/SQL00001/DMSCONT2
new containers: /db/db2inst1/db2backup/db2inst1/NODE0000/TESTDB/T0000008/DMSCONT1
                /db/db2inst1/db2backup/db2inst1/NODE0000/TESTDB/T0000008/DMSCONT2

db2 "backup database testdb tablespace dmsts1 to /db/db2inst1/db2backup/"
db2 "restore database testdb from /db/db2inst1/db2backup redirect"
db2 " set tablespace containers for 8 using
(path '/db/db2inst1/db2backup/db2inst1/NODE0000/TESTDB/T0000008/DMSCONT1', path '/db/db2inst1/db2backup/db2inst1/NODE0000/TESTDB/T0000008/DMSCONT2') "
db2 " restore database testdb continue "
```

## Change automatic storage path
```
db2 "restore db testdb from <path> on /dbdir1, /dbdir2, /dbdir3"
```

## Restore rebuild
```
db2 "restore db testdb rebuild with all tablespaces in database except tablespace(SMS01, SMS02) taken at timestamp"
```


## db2relocatedb
db2relocatedb is an offline, standalone tool that can be used to change:
- database name
- database path
- instance name
- location of one or more tablespace containers
- automatic storage path

```
Usages:
db2relocatedb -f config_file

config_file:
db_name=oldname, newname
db_path=oldpath, newpath
instance=oldinst, newinst
nodenum=nodenumber
lod_dir=oldlogdir, newlogdir
cont_path=oldpath1, newpath1
cont_path=oldpath2, newpath2
storage_path=oldpath1, newpath1
storage_path=oldpath2, newpath2
```

Examples:
```
db2 create database sales on /db
```

Change the path to /db2

1. DB must deactive
2. Move control file and containers to new location
3. Create the configuration file of db2relocatedb.
db_name=sales
db_path=/db, /db2
instance=db2inst1
nodenum=0

### move containers
```
db2 -tvf getcontainer.sql 
select substr(TBSP_NAME,1,16) as TBS_NAME, total_pages, substr(CONTAINER_NAME,1,60) as Container_name from sysibmadm.snapcontainer order by 1,3

TBS_NAME         TOTAL_PAGES          CONTAINER_NAME                                              
---------------- -------------------- ------------------------------------------------------------
FUNG                            12288 /db2/data/db2v97i/NODE0000/MYSAMPLE/T0000006/C0000000.LRG   
```

**move to /db/data/db2v97i/NODE0000/MYSAMPLE/T0000006/C0000000.LRG**

- step 1
```
    mkdir -p /db/data/db2v97i/NODE0000/MYSAMPLE/T0000006
    cp /db2/data/db2v97i/NODE0000/MYSAMPLE/T0000006/* /db/data/db2v97i/NODE0000/MYSAMPLE/T0000006/
```
- step 2
```
    db_name=mysample
    db_path=/db2/data
    instance=db2v97i
    cont_path=/db2/data/db2v97i/NODE0000/MYSAMPLE/T0000006/C0000000.LRG, /db/data/db2v97i/NODE0000/MYSAMPLE/T0000006/C0000000.LRG
```
- step 3
```
   db2relocatedb -f cont.cfg
```


