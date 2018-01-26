---
title: "SQL Tuning"
date: 2018-01-22 16:55:27
---
[TOC]

***Excerpt from Troubleshooting Oracle Performance***
## Parent Cursor & Child Cursor
父游标主要存储跟游标有关的SQL文本, 子游标主要存储执行环境，如优化器模式等，及执行计划。这些元素指定oracle是如何去执行一个SQL语句的。共享游标存储在shared poll中的library cache中。

如：
```
SQL1: select * from emp where emp_no = '1000';
SQL2: SELECT * FROM EMP WHERE EMP_NO = '1000';
```
分属不同父游标。

```
SQL3: select /*+ first_rows_1 */ * from emp where emp_no = '1000';
SQL4: select /*+ first_rows_10 */ * from emp where emp_no = '1000';
```
共享同一个父游标，但分属不同子游标

## Life Cycle of a Cursor
* 打开游标：私有SQL区域在UGA中被分配，用以打开游标，此时还没有sql语句
* 解析游标：SQL语句与游标关联，并将执行计划加载到library cache中, 更新私有SQL区域，在私有SQL区域中存储共享SQL区域的指针
* 定义输出变量：仅在有返回结果时
* 绑定输入变量：如果sql语句存在绑定变量
* 执行游标：sql语句被执行，在这个阶段，数据库引擎并不会做太多事情，对于很多查询，真正处理数据是在获取游标阶段
* 获取游标：如果sql语句返回数据，这个步骤就是获取数据，尤其是查询语句，此阶段处理了大部分的事情
* 关闭游标：释放UGA中私有sql区域的内存分配，共享sql区域并不会从SGA中移除，以备下次使用(软解析)

## How Parsing work
* 包含VPD(Virtual Private Database,也叫行级安全控制)：如果使用了行级安全控制，安全策略生成的约束条件填加到where字句中
* 语法、语义及权限的检查：检查SQL语句的正确性，对象的存在性及用户的访问权限
* 在共享SQL区域存储父游标信息：如果父游标不存在，则需在library cache分配内存空间用于存储父游标信息
* 在共享SQL区域存储子游标信息：分配内存空间，存储子游标，并与父游标关联

一旦在library cache存储了父、子游标信息，就可以通过`v$sqlare`(父游标), `v$sql`(子游标)获取相关信息
如果父游标和子游标都在SGA中，以上步骤只需要执行前两步骤，称之为软解析

## Binding peeking
使用绑定变量，可以减少硬解析的。但是同时也有缺点。因为在使用绑定变量的时候，查询优化器会把一些关键信息隐藏，如直方图。在使用绑定变量的时候，优化器会忽略绑定变量的值，有可能做出错误的选择。
为了防止这种情况，从9i开始引进绑定变量窥视。在产生执行计划之前，优化器会窥视一个绑定变量的值，并将它具体化。但是这样也有个缺点这个具体值是基于第一次执行的情况的，因此还是会存在执行计划不准确的情况，于是ACS应运而生。

## Adaptive Cursor Sharing
从oracle 11g开始，ACS被引进用来解决绑定变量窥视所带来的弊端。ACS在遇到绑定变量的时候，会自动多次对绑定变量进行窥视，并对不同变量的值生成不同的子游标，以期获得准确的执行计划。在`v$sql`中，有三个栏位跟acs有关：

* `is_bind_sensitive`: 不仅指明是否启用绑定变量同时也表明是否考虑ACS
* `is_bind_aware`:   表面是否启用ACS
* `is_shareable`: 表明游标是否可被共享

## Obtain execution plan
### explain plan for
通常需要先创建一个`plan table`, 一般使用`@?/rdbms/admin/utlxplan.sql`去创建这个表, 示例：

```
SQL> @?/rdbms/admin/utlxplan.sql
SQL> explain plan for select last_name from cust where cust_id = '100';
SQL> select * from table(dbms_xplan.display());
```
在使用绑定变量的时候，不能显示的将一个具体值代入绑定变量中，需要将绑定变量一起explan：

```
explain plan for select * from emp where emp_no >= :B;
```
explain有几个弊端：

* explain中的绑定变量默认为varchar2类型，如果谓语中的type是其他类型，会导致隐式转换, 如以下带****的地方:
```
FUNG@linora > explain plan for select first_name, last_name cust_id from cust where cust_id >= :B;

Explained.

FUNG@linora > select * from table(dbms_xplan.display('','','all'));

PLAN_TABLE_OUTPUT
-----------------------------------------------
Plan hash value: 4243541345

----------------------------------------------------------------------------
| Id  | Operation        | Name    | Rows  | Bytes | Cost (%CPU)| Time     |
----------------------------------------------------------------------------
|   0 | SELECT STATEMENT |         |     2 |    30 |     1   (0)| 00:00:01 |
|*  1 |  INDEX RANGE SCAN| CUST_PK |     2 |    30 |     1   (0)| 00:00:01 |
----------------------------------------------------------------------------

Query Block Name / Object Alias (identified by operation id):
-------------------------------------------------------------

   1 - SEL$1 / CUST@SEL$1

Predicate Information (identified by operation id):
---------------------------------------------------

****   1 - access("CUST_ID">=TO_NUMBER(:B))

Column Projection Information (identified by operation id):
-----------------------------------------------------------

   1 - "LAST_NAME"[VARCHAR2,30], "FIRST_NAME"[VARCHAR2,30]
```

* explain plan不会进行绑定变量窥视

***explain plan只是对执行计划做一个预估，因为它并不会去真正执行sql，所以出来的结果有可能是不准确的(同set autot on stats)***

### Query dynamic performance view
动态性能视图存储的是已经在library cache的游标信息。有如下几个关键视图：

* `v$sql_plan`
提供了基本和plan table一样的信息.

* `v$sql_plan_statistics`
提供了执行的统计信息，例如在v$sql_plan中的每个操作的持续时间、处理的行数。

* `v$sql_workarea`
提供了需要执行一个游标的内存工作区域相关信息

* `v$sql_plan_statistics_all`
此视图集中了以上三个视图的功能

在实际应用中，都不会去直接查询这几个视图，而是通过`dbms_xplan.display_***`去查询:
```
SELECT * FROM table(dbms_xplan.display_cursor('1hqjydsjbvmwq', 0));
```

***从动态性能视图查询出来的执行计划是准确的，因为它是已经被执行过的sql。***

### AWR/Statspack
通过`dbms_xplan.display_awr`可以查看AWR里面的执行计划, 同时AWR还分别提供了可执行计划及资源消耗的对比脚本`awrsqrpt.sql` and `sprepsql.sql`。

### Trace facility
* 10053
* 10046

## `dbms_xplan`
### `display_cursor`
format:
   * iostats:控制I/O统计的显示
   * memstats:控制PGA相关统计的显示
   * allstats:这是一个对于iostats,memstats的一个快捷方式
   * last:默认是所有执行的累积统计被显示如果这个值被指定只有最后的执行统计被显示
   * advanced:完全格式+outline
   * outline:以hint方式显示
   * peeked_binds:是否显示绑定变量窥视信息
   * buffstats:是否显示内存读次数，包括一致性读和当前读

### `display_awr`

## Execution plan output
## How to read execution plan
阅读执行计划，可以通过ID将他们的关系以树形结构画出来, 如：
```
----------------------------------------------
| Id  | Operation                     | Name |
----------------------------------------------
|   0 | UPDATE STATEMENT              |      |
|   1 |  UPDATE                       | T    |
|   2 |   NESTED LOOPS                |      |
|   3 |    TABLE ACCESS FULL          | T    |
|   4 |    INDEX UNIQUE SCAN          | T_PK |
|   5 |   SORT AGGREGATE              |      |
|   6 |    TABLE ACCESS BY INDEX ROWID| T    |
|   7 |     INDEX FULL SCAN           | I    |
|   8 |   TABLE ACCESS BY INDEX ROWID | T    |
|   9 |    INDEX UNIQUE SCAN          | T_PK |
----------------------------------------------
```
![execution tree](/attach/oracle/execution_tree.jpg)


