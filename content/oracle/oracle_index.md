---
title: "Oracle Index"
date: 2018-01-15 12:22:20
---
[TOC]

# 综述
SQL查询一般流程：
SQL > 解析器 > 查询优化器 > 执行计划 > 执行 > 返回
SQL进来会由解析器进行语法/语义分析，如果出错，返回错误结果；否则进入查询优化器中，优化器会从缓存中(shared pool中的library cache)查找是否有相同的执行计划hash值，如果有，直接将此执行计划给下一步执行(软解析)；否则，优化器会通过一定的规则去生成执行树，并生成最佳的执行计划(选择表的连接顺序，连接方法，访问路径)，然后执行，最后返回结果。

当一个表及其对应的索引选择数据时，一般对应三种情况：
index range scan;
index fast full scan;
1. 查询所需的所有表的数据都在索引结构中，只需要访问索引块
2. 查询所需信息没有都包含在索引块中，查询优化器选择既访问索引块也访问数据块
3. 优化器选择不使用索引，只访问数据块

## oracle索引的三大特性
* 索引本身有序
* 索引存储着索引列的key值和数据块的rowid
* 索引树高度比较低

## Estimate index space cost
```
SQL> exec dbms_stats.gather_table_stats(user,'T_MSSM');
SQL> variable used_bytes number
SQL> variable allocated_bytes number
SQL> exec dbms_space.create_index_cost('create index obj_id on t_mssm(OBJECT_ID)', :used_bytes, :allocated_bytes);
SQL> print :used_bytes;
SQL> print allocated_bytes;
```

## Query index information from dba_indexes or dba_ind_columns
```
col index_name for a20
col index_type for a50
col table_name for a20
col tablespace_name for a20
select index_name, index_type, table_name, tablespace_name, status
from dba_indexes
where table_name in ('CUST', 'T_MSSM');
--get ddl
select dbms_metadata.get_ddl('INDEX', 'OBJ_ID') from dual;
```

## Monitoring index usage
```
alter index fung.obj_idx monitoring usage;
select index_name, monitoring, used from v$object_usage;
--监控索引使用方式，如fast full scan等
select d.object_name, d.operation, d.options, count(1)
from dba_hist_sql_plan d, dba_hist_sqlstat h
where d.object_owner <> 'SYS' and d.operation like '%INDEX%'
and d.sql_id = h.sql_id group by d.object_name, d.operation, d.options
order by 1,2,3;
```

# Explain中关于statistics
1. DB block gets(当前请求的块数目):
当前模式块意思就是在操作中正好提取的块数目，而不是在一致性读的情况下而产生的块数。正常的情况下，一个查询提取的块是在查询开始的那个时间点上存在的数据块，当前块是在这个时刻存在的数据块，而不是在这个时间点之前或者之后的数据块数目。
number of data blocks read in CURRENT mode ie) not in a read consistent fashion, but the current version of the data blocks.
2. Consistent gets:
这里的概念是在处理你这个操作的时候需要在一致性读状态上处理多少个块，这些块产生的主要原因是因为由于在你查询的过程中，由于其他会话对数据块进行操作，而对所要查询的块有了修改，但是由于我们的查询是在这些修改之前调用的，所以需要对回滚段中的数据块的前映像进行查询，以保证数据的一致性。这样就产生了一致性读。 不过精确翻译来说，前两个不是块数而是IO请求次数，不过对于这两者应该是一次读一块的。
3. Physical read: 磁盘读

logical reads(内存读) = db block gets + consistent gets

# 索引失效的场景
## 索引有效，但CBO不会选择
* 使用索引代价反而更高
* 索引列的类型转换(隐式转换)
    - 比如索引列ID为varchar类型，但是where字句中使用了整型"where id = 1"
* 对索引列进行运算
    - 比如索引列ID为number整型， 但是where子句使用了"where id/2 > 30"或者where upper(name)=''
* 使用" < > ", 'not in', 'not exists', '!='导致索引失效, B-tree index 'is not null'不走索引, 但is null会走
* 使用like且%在前
* 使用复合索引，但并未使用索引第一列
* 将空变量值直接与运算符比较

## 索引失效，变成unusable, 需要rebuild index
* LONG类型列调整
* alter table move
* 分区表
    - truncate/drop/exchange分区表导致全局索引失效，解决方法：1.全部使用local index；2.truncate/drop/exchange分区时加上update global indexes
    - split分区会导致全局索引和局部索引失效，增加update global indexes可以保持全局索引有效, 但local index需要重新rebuild

## select count(*) 的优化
* 考虑到索引列为空是不会走索引，如果有PK值，由于PK不为空，count(*)可以走索引，但如果没有PK，且索引栏位允许为空，此时需要不允许索引列为空才会走索引
* 同理，sum/avg都可以用这种方法优化
* 如果select count(*)比较多，表更新很少,可以考虑在某个重复量大的栏位创建bitmap index，如status，性别之类的(慎用)

## 索引回表优化(Table access by index rowid)
* 取消多余字段，少用select *
* 考虑组合索引
* 不可避免table access by index rowid的情况，考虑 clustering factor

## index fast full scan和index full scan
* 两者都为索引全扫描，从头到尾遍历索引
* fast full scan一次读取多次索引块， index full scan一次只读取一个块，索引fast要快
* 由于fast一次读取多次块，不容易保证有序，因此在一些需要排序的查询中，full scan可以消除排序的消耗

## union 合并的优化
* 尝试用union all代替union，消除排序


# tips
* 外建约束一定要建索引
* 复合索引，当一列是范围查询，一列是等值查询，索引中，通常将等值那一列放在前面，但如果范围查找的那一列经常用到，且重复值少，则需要相反
* 范围查询改写为in效率更高

# 表的连接
***嵌套循环连接和哈希连接表有驱动顺序，驱动表的顺序不同将影响表连接的性能，而排序合并没有驱动的概念***
***除了嵌套循环，其他连接都需要排序***
***哈希连接仅支持等值连接***

# 嵌套循环与索引
* 两表关联返回记录不多
* 驱动表的限制条件所在列有索引
* 被驱动表的连接条件所在列有索引

# 反向索引
* 索引值反过来存储在索引块
* 对于RAC索引块竞争，且该索引更新比较频繁，使用等值查询很有效
* 不能进行范围查找，如果范围查找频繁，建议使用哈希分区表
* 当等待事件出现`buffer busy wait`, `read by other session`, 数据块可能存在竞争


# 多表连接方式
## Nested loop
访问次数：驱动表返回几条，被驱动表访问多次
驱动顺序：有
排序差别：无须排序
限制场景：无
索引限制：驱动表限制条件有索引，被驱动表连接条件有索引
## Hash join
访问次数：最多被访问一次
驱动顺序：有
排序差别：无，但是消耗内存的hash_area_size
限制场景：只能等值查询
索引限制：索引列在表连接中无特殊要求，跟单表情况一样
## Merge Sorted Join
访问次数：最多被访问一次
驱动顺序：无
排序差别：需要排序sort_area_size
限制场景：连接条件为< > like导致无法使用
索引限制：索引消除排序

## Oracle STA
### Verifying Automatic job Running
```
SELECT client_name, status, consumer_group
FROM dba_autotask_client
ORDER BY client_name;
```

### view the maintenance window details
```
SELECT window_name,TO_CHAR(window_next_time,'DD-MON-YY HH24:MI:SS')
,sql_tune_advisor, optimizer_stats, segment_advisor
FROM dba_autotask_window_clients;
```

## Index scans
### Index unique scan
等值查询且仅返回一行
如where子句只有包含PK值, 下例中object_id为PK
```
FUNG@linora > explain plan for select * from test where object_id = :B;
FUNG@linora > select * from table(dbms_xplan.display_cursor('','','all, allstats'));
--------------------------------------------------------------------------------------------
| Id  | Operation                   | Name        | E-Rows |E-Bytes| Cost (%CPU)| E-Time   |
--------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT            |             |      1 |    89 |     2   (0)| 00:00:01 |
|   1 |  TABLE ACCESS BY INDEX ROWID| TEST        |      1 |    89 |     2   (0)| 00:00:01 |
|*  2 |   INDEX UNIQUE SCAN         | SYS_C003999 |      1 |       |     1   (0)| 00:00:01 |
```

### Index range scan
使用于高选择性的数据，返回的数据是升序的。在col1为组合索引中的leading column情况下，会执行index range scan
```
col1 = :b1
col1 > :b1
col1 < :b1
```

### Index skip scan
当查询一个复合索引且where子句中跳过了leading column时候，会产生index skip scan。下面的例子中，PK为(cust_id, first_name, last_name), 因为where子句只有first_name, 不包含leading column cust_id, 所以产生index skip scan
```
FUNG@linora > explain plan for select * from cust where first_name =:B;
FUNG@linora > select * from table(dbms_xplan.display('','','all, allstats'));
-----------------------------------------------------------------------------
| Id  | Operation        | Name    | E-Rows |E-Bytes| Cost (%CPU)| E-Time   |
-----------------------------------------------------------------------------
|   0 | SELECT STATEMENT |         |      1 |    15 |     1   (0)| 00:00:01 |
|*  1 |  INDEX SKIP SCAN | CUST_PK |      1 |    15 |     1   (0)| 00:00:01 |
-----------------------------------------------------------------------------
```
### Index full scan
如果需要全表扫描，再进行排序的时候，index full scan是一种很好的选择。存在以下情况，会出现index full scan
* 查询需要sort merge join，查询所引用的所有列都在索引中，且索引列顺序相同
* 查询中包含order by，子句中所有列都必须在索引列中
* 查询包含group by， 索引和group by必须包含相同列，顺序可以不同

```
FUNG@linora > explain plan for select * from test order by object_id;
FUNG@linora > select * from table(dbms_xplan.display('','','all, allstats'));
--------------------------------------------------------------------------------------------
| Id  | Operation                   | Name        | E-Rows |E-Bytes| Cost (%CPU)| E-Time   |
--------------------------------------------------------------------------------------------
|   0 | SELECT STATEMENT            |             |  14036 |  1219K|   241   (0)| 00:00:03 |
|   1 |  TABLE ACCESS BY INDEX ROWID| TEST        |  14036 |  1219K|   241   (0)| 00:00:03 |
|   2 |   INDEX FULL SCAN           | SYS_C003999 |  14036 |       |    29   (0)| 00:00:01 |
--------------------------------------------------------------------------------------------
```

### Index fast full scan
索引本身包含查询中的所有列时，使用index fast full scan
```
FUNG@linora > explain plan for select count(*) from test;
FUNG@linora > select * from table(dbms_xplan.display('','','all, allstats'));
------------------------------------------------------------------------------
| Id  | Operation             | Name        | E-Rows | Cost (%CPU)| E-Time   |
------------------------------------------------------------------------------
|   0 | SELECT STATEMENT      |             |      1 |     9   (0)| 00:00:01 |
|   1 |  SORT AGGREGATE       |             |      1 |            |          |
|   2 |   INDEX FAST FULL SCAN| SYS_C003999 |  14036 |     9   (0)| 00:00:01 |
------------------------------------------------------------------------------
```


