---
title: "DB2 tablespace"
date: 2018-01-16 17:09:15
---
## creating the DMS tablespace
### non-auto resize
```
db2 "create tablespace tb2 managed by database using (file '/db/db2inst1/testdb/db2inst1/NODE0000/TESTDB/T0000005/myfile' 2048, file '/db/db2inst1/testdb/db2inst1/NODE0000/TESTDB/T0000005/myfile2' 2048) extentsize 4"
```

### auto resize
```
db2 "create tablespace tb1 pagesize 8192 managed by database using (file '/db/db2inst1/testdb/db2inst1/NODE0000/TESTDB/T0000006/tb1.dbf' 20M) autoresize yes maxsize none"
```

## alter tbs to autoresize
```
db2 "alter tablespace tb2 autoresize yes"
```

## alter tablespace add container
```
db2 "alter tablespace tb2 add (file '/db/db2inst1/testdb/db2inst1/NODE0000/TESTDB/T0000006/tb1' 10M) autoresize yes maxsize none"
```

## alter tablespace extend
```
db2 "alter tablespace TB2 extend (file '/db/db2inst1/testdb/db2inst1/NODE0000/TESTDB/T0000006/tb1' 768) autoresize yes maxsize none"
```

## alter tablespace reduce
```
db2 "alter tablespace TB2 reduce (file '/db/db2inst1/testdb/db2inst1/NODE0000/TESTDB/T0000006/tb1' 2048) autoresize yes maxsize none"
```


## Auto storage management
Create AUTOMATIC STORAGE table spaces
– No explicit container definitions are provided
– Containers automatically created across the storage paths
– Growth of existing containers and addition of new ones managed by DB2

### examples
```
db2 "create database testdb1 automatic storage no"
```
***Automatic storage enabled: NO***
***Database path: dftdbpath***

```
db2 "create database testdb1 on /db2/dir1, /db2/dir2, /db2/dir3
```
***AS enabled: YES***
***Database path: /db2/dir1***
***Storage path: /db2/dir1, /db2/dir2, /db2/dir3***

```
db2 "create database testdb1"
```
***AS enabled: YES***
***Database path: dftdbpath***
***Storage path: dftdbpath***

```
db2 "create database testdb1 on /stor1, /stor2 dbpath on /dbpath
```
***AS enabled: YES***
***Database path: /dbpath***
***Storage path: /stor1, stor2***

```
db2 "create database testdb1 on /dbdir"
```
***AS enabled: YES***
***Database path: /dbdir***
***Storage path: /dbdir***

## converting regular tablespace to large
convert=> reorg=> set integrity
```
db2 "alter tablespace <name> convert to large"
db2 "reorg all indexes for table <name> allow read access"
```


