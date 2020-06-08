# Order By优化

> mysql测试版本：`mysql  Ver 14.14 Distrib 5.7.22, for Win64 (x86_64)`

order by排序

- 只需使用索引取得数据，不需额外排序，索引是有序的，根据索引进行排序不需额外的操作。（以下这种情况称为索引排序）
- 使用filesort排序



测试表：

```sql
CREATE TABLE `user` (
  `id` int(11) NOT NULL AUTO_INCREMENT,
  `name` varchar(255) DEFAULT NULL,
  `age` int(11) DEFAULT '0',
  `desc` varchar(255) DEFAULT 'no desc',
  `height` double DEFAULT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB AUTO_INCREMENT=21 DEFAULT CHARSET=utf8
```

## 使用索引排序情况

一些情况下，`mysql`可能会使用索引排序，避免执行`filesort` 

- 如果order by语句没有完全匹配到索引，索引仍然会使用，只要满足索引未使用的部分和所有额外的order by列在where语句中是常量即可
- 如果索引不包含select的所有字段，仅当索引访问比其他访问方法耗性能更小时才会使用



可能会使用索引排序的情况：

> 假设user表中有聚合索引user(name,age)

1. order by语句中能匹配到索引，

   ```sql
   create index idx_name_age on user(name,age);
   ```

   语句一：

   ```sql
   explain select * from user order by name,age;
   ```

   执行计划：

   ![1570608273712](order by语句优化.assets\1570608273712.png)

   语句二：

   ```sql
   explain select name,age from user order by name,age;
   ```

   执行计划：

   ![1570608357154](order by语句优化.assets\1570608357154.png)

   语句三：

   ```sql
   explain select id,name,age from user order by name,age;
   ```

   执行计划：

   ![1570608390761](order by语句优化.assets\1570608390761.png)

   语句四：

   ```sql
   explain select id,name,age from user order by name desc,age desc;
   ```

   执行计划：

   ![1570610355915](order by语句优化.assets\1570610355915.png)

   语句五：

   ```sql
   explain select id,name,age from user order by name asc,age desc;
   ```

   执行计划：

   ![1570610452213](order by语句优化.assets\1570610452213.png)

   > asc desc混合使用，引起filesort

   结论：

   - 当查询的字段比order by语句的字段多时，优化器只有当使用索引排序比filesort耗性能小时才会使用索引排序（官方解释）。上述语句一中未使用索引排序，因为优化器判定使用filesort性能好
   - 当查询字段（可以加上主键）刚好与order by字段匹配时，优化器会使用索引排序
   - 当order by中的字段降序、升序混合使用，可能会引起filesort

2. where语句（条件需和常量比较）结合order by语句匹配索引情况

   类型一：

   ```sql
   explain select * from user where name='tom' order by age;
   ```

   执行计划：

   ![1570609958318](order by语句优化.assets\1570609958318.png)

   注意这里extra是using index condition而不是using index

   - using index：覆盖索引
   - using index condition：使用覆盖索引排除数据
     - 一般抓取数据情况：根据索引抓取到一行数据，然后根据where条件筛选这行是否合适
     - 使用覆盖索引抓取数据情况：根据覆盖索引直接筛选这行数据是否满足条件而不需抓取整行数据

   ```sql
   explain select name,age from user where name='tom' order by age;
   ```

   执行计划：

   ![1570610120766](order by语句优化.assets\1570610120766.png)

   > where中的name（需与常量比较）与order by中的age匹配了索引，优化器判定使用索引排序性能高

   ```sql
   explain select * from user where name='tom' and age>20 order by age;
   ```

   ![1570616254403](order by语句优化.assets\1570616254403.png)

   > 由于index(name,age)中name的条件是常量，即是确定的，所以还是可能会使用索引排序

   类型二：匹配索引，左端是范围

   ```sql
   explain select name,age from user where name >'tom' order by name asc;
   ```

   ![1570615865625](order by语句优化.assets\1570615865625.png)

   ```sql
   explain select name,age from user where name <'tom' order by name desc;
   ```

   ![1570615918397](order by语句优化.assets\1570615918397.png)

   > 若使用索引排序性能高则使用

## 不使用索引排序情况

1. order by语句中匹配了多个索引

   ```sql
   /**索引为user(name)、user(age)*/
   explain select name,age from user u order by name,age;
   ```

   ![1570617238041](order by语句优化.assets\1570617238041.png)

2. order by语句中未有序匹配索引

   ```sql
   /**索引为user(name,age)*/
   explain select name,age from user order by age,name;
   ```

   ![1570612498315](order by语句优化.assets\1570612498315.png)

   > 这里using index表示使用了覆盖索引

3. 混合使用asc和desc

   ```sql
   /**索引为user(name,age)*/
   explain select name,age from user order by name asc,age desc;
   ```

   ![1570612569727](order by语句优化.assets\1570612569727.png)

   

4. 抓取数据依据的索引（where）与order by匹配的索引不一样

   ```sql
   /**索引为user(name)、user(age)*/
   explain select * from user u WHERE name='tom' order by age;
   ```

   ![1570617186993](order by语句优化.assets\1570617186993.png)

5. order by语句中的字段并非表原有字段，比如做了函数操作

   ```sql
   /**索引为user(name,age)*/
   explain select name,age from user order by abs(age);
   ```

   ![1570612835978](order by语句优化.assets\1570612835978.png)

   ```sql
   /**索引为user(name,age)*/
   explain select name,age from user order by -age;
   ```

   ![1570612868112](order by语句优化.assets\1570612868112.png)

6. The query joins many tables, and the columns in the `ORDER BY` are not all from the first nonconstant table that is used to retrieve rows. (This is the first table in the [`EXPLAIN`](https://dev.mysql.com/doc/refman/5.7/en/explain.html) output that does not have a [`const`](https://dev.mysql.com/doc/refman/5.7/en/explain-output.html#jointype_const) join type.)

7. order by字段和group by字段不一致

   ```sql
   /**索引为user(name,age)*/
   explain select name,age from user group by name,age order by age;
   ```

   ![1570614172035](order by语句优化.assets\1570614172035.png)

   ```sql
   /**索引为user(name,age)*/
   explain select name,age from user group by name,age order by name;
   explain select name,age from user group by name,age order by name,age;
   ```

   ![1570614247227](order by语句优化.assets\1570614247227.png)

8. 索引未有序存储行数据，比如， this is true for a `HASH` index in a `MEMORY` table（哈希索引）

9. 索引了部分情况

   ```sql
   /**索引为user(name,age)*/
   explain select name,age from user where name like 'to%' order by age;
   ```

   ![1570613655383](order by语句优化.assets\1570613655383.png)

   > 索引不能区分也就是不能确定"to%"中“%”的内容，所以不会使用索引排序

   ```sql
   /**索引为user(name,age)*/
   explain select name,age from user where name ='tom' order by age;
   ```

   ![1570613815515](order by语句优化.assets\1570613815515.png)

   ```sql
   /**索引为user(name,age)*/
   explain select name,age from user group by name,age order by name;
   explain select name,age from user group by name,age order by name,age;
   ```

   ![1570614247227](order by语句优化.assets\1570614247227.png)

10. 字段别名使用的影响

    ```sql
    /**name是单个索引字段*/
    explain select name from user order by name;
    ```

    ![1570617499897](order by语句优化.assets\1570617499897.png)

    > 使用filesort

    ```sql
    /**age是单个索引字段*/
    explain select ABS(age) as age from user order by age;
    ```

    ![1570617701704](order by语句优化.assets\1570617701704.png)

    > 因为别名影响，order by中age字段对应到了别名age，使用了函数操作的ABS(age)，使用filesort

    ```sql
    /**age是单个索引字段*/
    explain select ABS(age) as b from user order by age;
    ```

    ![1570617896792](order by语句优化.assets\1570617896792.png)

    > age并未对应上别名b，正常使用索引



## filesort

filesort是查询执行的额外的排序操作。有两种实现方式：

- 内存中操作：优化器会预先给每个会话分配`sort_buffer_size`字节的内存，可修改。适合完全内存操作：

  - SELECT ... FROM *single_table* ... ORDER BY *non_index_column* [DESC] LIMIT [*M*,]*N*;

    如：

    ```sql
    SELECT col1, ... FROM t1 ... ORDER BY name LIMIT 10;
    SELECT col1, ... FROM t1 ... ORDER BY RAND() LIMIT 15;
    ```

- 磁盘中操作：当结果集太大，内存不能满足时，会使用临时磁盘文件来完成排序操作



# group by优化

https://dev.mysql.com/doc/refman/5.7/en/group-by-optimization.html

group by语句默认是隐式排序，但不会影响速度。不建议通过group by的显式排序或者隐式排序来实现排序效果，建议通过order by语句进行排序。

避免隐式排序：

```sql
select name,age from user group by name,age order by null;
```

优化器会尝试使用索引而避免通过创建临时表的方式实现group by

使用索引先决条件：

- group by来自同一索引中的字段
- 索引按顺序存储数据（如BTREE索引，不能是hash索引）

group by语句有两种通过索引执行查询的方法：

- 将分组操作和所有范围扫描（如果存在）一起运用
- 先执行范围扫描，然后对所得的元组数据进行分组



## 松散索引扫描

处理group by最有效的方式就是直接通过索引（使用覆盖索引）获得group by的字段，MYSQL是通过索引中的数据是有序的这一属性实现的（BTREE），这种方式使得查找匹配where条件的分组时而不需考虑索引中的全部数据（即不需索引中所有数据），而只需考虑部分的，所以称为松散索引扫描

假设index `idx(c1,c2,c3)` on table `t1(c1,c2,c3,c4)`，会使用松散索引扫描情况：

```sql
SELECT c1, c2 FROM t1 GROUP BY c1, c2;
SELECT DISTINCT c1, c2 FROM t1;
SELECT c1, MIN(c2) FROM t1 GROUP BY c1;
SELECT c1, c2 FROM t1 WHERE c1 < const GROUP BY c1, c2;
SELECT MAX(c3), MIN(c3), c1, c2 FROM t1 WHERE c2 > const GROUP BY c1, c2;
SELECT c2 FROM t1 WHERE c1 < const GROUP BY c1, c2;
SELECT c1, c2 FROM t1 WHERE c3 = const GROUP BY c1, c2;
```

> 索引从group by中开始匹配，中间没隔断

## 紧密索引扫描

紧密索引扫描可能是全索引扫描或者是范围索引扫描，取决于查询条件

```sql
/**索引为user(name,age)*/
explain select name,age from user where name='tom' group by age;
```

![1570628798928](order by语句优化.assets\1570628798928.png)

> 最左匹配不是从group by匹配起，extra未显示`Using index for group-by`和`using temporary`，表示是使用了紧密索引扫描

错误示范：

```sql
explain select height from user group by height;
```

![1570628421505](order by语句优化.assets\1570628421505.png)

> using temporary就是使用了临时文件去实现分组，避免出现这种情况



