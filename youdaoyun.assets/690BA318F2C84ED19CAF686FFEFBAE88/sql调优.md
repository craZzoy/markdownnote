#### 16个语句的调优方案
1. 在保证功能的基础上尽量减少对数据库的访问次数，减少对表的访问行数，从而减轻网络负担；在数据窗口使用SQL时，尽量把使用的索引放在选择的首列；避免使用游标;不要过多使用通配符select * from table, 要用几列就写几列。
应考虑在 where 及 order by 涉及的列上建立索引。

2. 避免不兼容的数据 例如float 和int，char和varchar，binary和varbinary 是不兼容的。数据类型的不兼容可能使优化器无法执行一些本来可以进行的优化操作。

3. 尽量避免在where子句中对字段进行函数或者表达式操作，这将导致引擎放弃使用索引而进行全表扫描。例如：
```
select col1 from table1 where col2/2 =50
```
应该为 
```
select col1 from table1 where col2 =50*2 select * from table1 where datediff(yy,birthday,getdate())>21 select * from table1 where birthday< datediff(yy,-21,getdate())
```
4. 避免使用！= <>,is null 或者是 is not null， in， not in 等这样的操作符，因为这样会使系统无法使用索引。

5. 尽量使用数字型字段，引擎在处理和连接会逐个比较字符串中每一个字符，而对与数字型而言只需要比较一次就够了


6. 合理使用exist 和 not exist 可以使用exist 代替count（*）来验证表里是否存在某条记录 例如：
```
select count（*）from table1 where (select count(*) from table2 where table1.id=table2.id )>0 
select count（*）from table1 where exist (select count(*) from table2 where table1.id=table2.id )
```
 后者不会产生大量的表扫描或者是索引扫描。

7. 尽量避免在索过的字符数据中使用非打头字母搜索，这使得引擎无法利用索引。
```
 select * from table1 where substring（name，2,1）=l
```

8. 拆分多条件连接，俩表之间不只一个的连接条件可在where子句中将连接条件完整的写上，可能大大提高查询速度。
```
select * from table1,table2 where a.key=b.key and a,count=b.account
```

9. 消除対大表数据的顺序存取
```
select * from table1 where (name=xiaoniu and number>200 ) or number= 1000
```
   应该为 
```
select * from table1 where (name=xiaoniu and number>200 )  
union 
select * from table1 where number= 1000 
```

10. 避免困难的正则表达式
```
select * from table1 where zipcode like'12___'

```
将采用顺序扫描，可改为 

```
select * from table1 where zipcode >'12000'
```

11. 能用between 的就不用in （in 不会走索引）

12. 能用distinct 的就不用group by

13. 部分利用索引
```
select * from table where dept='prod' or city='shanghai' or division='food'
```
 可改为
```
  select  * from table  where dept='prod'  union all select * from table where  city='shanghai'  union all  select * from table where division='food'
```
 如果dept 有建立索引，则第二种方案会使用部分索引

14. 能用union all 就不用 union， union 使用distinct 使用资源

15. 尽量不要用 select into 语句 select into 会导致表锁定，阻止其他用户访问该表

16. 必要时强制执行查询优化器使用某个索引
```
select * from table （index=ix_name） where name in ('a','b','c')
```


#### 导致引擎放弃使用索引而进行全表扫描的一些情况及一些建议
1. 避免在 where 子句中对字段进行 null 值判断
```
select id from t where num is null
```
可以改为

```
select id from t where num=0 
```

2. 避免在 where 子句中使用!=或<>操作符

3. 避免在 where 子句中使用 or 来连接条件，可使用union。

```
select id from t where num=10 or num=20
```
可以改为

```
select id from t where num=10 

union all 

select id from t where num=20 
```

4. 慎用in 和 not in（用between）

5. 慎用模糊查询 

6. 如果在 where 子句中使用参数，也会导致全表扫描。因为SQL只有在运行时才会解析局部变量，但优化程序不能将访问计划的选择推迟到运行时；它必须在编译时进行选择。然而，如果在编译时建立访问计划，变量的值还是未知的，因而无法作为索引选择的输入项。如下面语句将进行全表扫描： 
```
select id from t where num=@num 
```
可以改为强制查询使用索引：
```
select id from t with(index(索引名)) where num=@num
```

7. 避免在 where 子句中对字段进行表达式操作。

8. 避免在where子句中对字段进行函数（内置函数）操作。

9. 在使用索引字段作为条件时，如果该索引是复合索引，那么必须使用到该索引中的第一个字段作为条件时才能保证系统使用该索引，否则该索引将不会被使用，并且应尽可能的让字段顺序与索引顺序相一致。

10. 很多时候用 exists 代替 in 是一个好的选择。

11. 
```
select num from a where num in(select num from b) 
```
用下面的语句替换：

```
select num from a where exists(select 1 from b where num=a.num) 
```

12. 并不是所有索引对查询都有效，SQL是根据表中数据来进行查询优化的，当索引列有大量数据重复时，SQL查询可能不会去利用索引（除非是位图索引），如一表中有字段sex，male、female几乎各一半，那么即使在sex上建了索引也对查询效率起不了作用。

13. 索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。

14. 索引并不是越多越好，索引固然可以提高相应的 select 的效率，但同时也降低了 insert 及 update 的效率，因为 insert 或 update 时有可能会重建索引，所以怎样建索引需要慎重考虑，视具体情况而定。一个表的索引数最好不要超过6个，若太多则应考虑一些不常使用到的列上建的索引是否有必要。

15. 应尽可能的避免更新 clustered（复合） 索引数据列，因为 clustered 索引数据列的顺序就是表记录的物理存储顺序，一旦该列值改变将导致整个表记录的顺序的调整，会耗费相当大的资源。若应用系统需要频繁更新 clustered 索引数据列，那么需要考虑是否应将该索引建为 clustered 索引。

16. 尽可能的使用 varchar/nvarchar 代替 char/nchar，最好用varchar2（自变长度） ，因为首先变长字段存储空间小，可以节省存储空间，其次对于查询来说，在一个相对较小的字段内搜索效率显然要高些。 

17. 任何地方都不要使用 select * from t ，用具体的字段列表代替“*”，不要返回用不到的任何字段。

18. 尽量使用表变量来代替临时表（一般用在存储过程中）。如果表变量包含大量数据，请注意索引非常有限（只有主键索引）。

19. 避免频繁创建和删除临时表，以减少系统表资源的消耗。


20. 临时表并不是不可使用，适当地使用它们可以使某些例程更有效，例如，当需要重复引用大型表或常用表中的某个数据集时。但是，对于一次性事件，最好使用导出表。

21. 在新建临时表时，如果一次性插入数据量很大，那么可以使用 select into A select。。。 代替 create table，避免造成大量 log ，以提高速度；如果数据量不大，为了缓和系统表的资源，应先create table，然后insert。

22. 如果使用到了临时表，在存储过程的最后务必将所有的临时表显式删除，先 truncate table ，然后 drop table ，这样可以避免系统表的较长时间锁定。 

23. 尽量避免使用游标，因为游标的效率较差，如果游标操作的数据超过1万行，那么就应该考虑改写。

24. 使用基于游标的方法或临时表方法之前，应先寻找基于集的解决方案来解决问题，基于集的方法通常更有效。

25. 与临时表一样，游标并不是不可使用。对小型数据集使用 FAST_FORWARD 游标通常要优于其他逐行处理方法，尤其是在必须引用几个表才能获得所需的数据时。在结果集中包括“合计”的例程通常要比使用游标执行的速度快。如果开发时间允许，基于游标的方法和基于集的方法都可以尝试一下，看哪一种方法的效果更好。

26. 在所有的存储过程和触发器的开始处设置 SET NOCOUNT ON ，在结束时设置 SET NOCOUNT OFF 。无需在执行存储过程和触发器的每个语句后向客户端发送 DONE_IN_PROC 消息。

27. 尽量避免大事务操作，提高系统并发能力。 
28. 尽量避免向客户端返回大数据量，若数据量过大，应该考虑相应需求是否合理。






 

