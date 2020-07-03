#### 创建语句：

```
create materialized view mv_materialized_test refresh force on demand start with sysdate next
to_date(concat(to_char( sysdate+1,'dd-mm-yyyy'),'10:25:00'),'dd-mm-yyyy hh24:mi:ss') as
select * from user_info; --这个物化视图在每天10：25进行刷新
```
物化视图也是种视图。物化视图可以查询表，视图和其它的物化视图。

#### 特点：
1. 物化视图在某种意义上说就是一个物理表(而且不仅仅是一个物理表)，这通过其可以被user_tables查询出来，而得到确认；
2. 物化视图也是一种段(segment)，所以其有自己的物理存储属性；
3. 物化视图会占用数据库磁盘空间，这点从user_segment的查询结果，可以得到佐证；
创建语句：create materialized view mv_name as select * from table_name
因为物化视图由于是物理真实存在的，故可以创建索引。
#### 创建时生成数据
分为两种：build immediate 和 build deferred，
1. build immediate是在创建物化视图的时候就生成数据。
2. build deferred则在创建时不生成数据，以后根据需要在生成数据。
如果不指定，则默认为build immediate。

#### 刷新模式
物化视图有二种刷新模式
1. on demand 顾名思义，仅在该物化视图“需要”被刷新了，才进行刷新(REFRESH)，即更新物化视图，以保证和基表数据的一致性;
2. on commit  提交触发，一旦基表有了commit，即事务提交，则立刻刷新，立刻更新物化视图，使得数据和基表一致。一般用这种方法在操作基表时速度会比较慢。创建物化视图时未作指定，则Oracle按 on demand 模式来创建.

上面说的是刷新的模式，针对于如何刷新，则有三种刷新方法：

==完全刷新（COMPLETE）==： 会删除表中所有的记录（如果是单表刷新，可能会采用TRUNCATE的方式），然后根据物化视图中查询语句的定义重新生成物化视图。
==快速刷新（FAST）==： 采用增量刷新的机制，只将自上次刷新以后对基表进行的所有操作刷新到物化视图中去。FAST必须创建基于主表的视图日志。对于增量刷新选项，如果在子查询中存在分析函数，则物化视图不起作用。
==FORCE方式==：这是默认的数据刷新方式。Oracle会自动判断是否满足快速刷新的条件，如果满足则进行快速刷新，否则进行完全刷新。
 
==关于快速刷新==：Oracle物化视图的快速刷新机制是通过物化视图日志完成的。Oracle通过一个物化视图日志还可以支持多个物化视图的快速刷新。物化视图日志根据不同物化视图的快速刷新的需要，可以建立为ROWID或PRIMARY KEY类型的。还可以选择是否包括SEQUENCE、INCLUDING NEW VALUES以及指定列的列表。

#### 查询重写（QueryRewrite）：
包括 ==enable query rewrite== 和 ==disable query rewrite== 两种。
分别指出创建的物化视图是否支持查询重写。查询重写是指当对物化视图的基表进行查询时，oracle会自动判断能否通过查询物化视图来得到结果，如果可以，则避免了聚集或连接操作，而直接从已经计算好的物化视图中读取数据。
默认为disable query rewrite。

#### 语法：

```
create materialized view view_name
refresh [fast|complete|force]
[
on [commit|demand] |
start with (start_time) next (next_time)
]
AS subquery;
```

#### 具体操作
##### 创建物化视图需要的权限：

```
grant create materialized view to user_name; 
```
##### 在源表建立物化视图日志：

```
create materialized view log on test_table  
tablespace test_space -- 日志空间  
with primary key;     -- 指定为主键类型
```
##### 在目标数据库上创建MATERIALIZED VIEW：

```
create materialized view mv_materialized_test refresh force on demand start with sysdate next
to_date(concat(to_char(sysdate+1,'dd-mm-yyyy'),'10:25:00'),'dd-mm-yyyy hh24:mi:ss') as
select * from user_info; --这个物化视图在每天10：25进行刷新 
```
##### 修改刷新时间：

```
alter materialized view mv_materialized_test refresh force on demand start with sysdate 
next to_date(concat(to_char(sysdate+1,'dd-mm-yyyy'),' 23:00:00'),'dd-mm-yyyy hh24:mi:ss');
或
alter materialized view mv_materialized_test refresh force on demand start with sysdate 
next trunc(sysdate,'dd')+1+1/24; -- 每天1点刷新 
```
##### 建立索引：

```
create index IDX_MMT_IU_TEST
on mv_materialized_test(ID,UNAME)  
tablespace test_space;
```
##### 删除物化视图及日志:

```
drop materialized view log on test_table;    --删除物化视图日志： 
drop materialized view mv_materialized_test; --删除物化视图
```
