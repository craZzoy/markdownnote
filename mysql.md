# **mysql体系架构**



## 整体架构图

![img](youdaoyun.assets\cb4bc7c4db154ad8a16818120e9cd87d\ip_image002.jpeg)

1. connectors：接入方，支持的协议很多。
2. Management Services&Utilities：系统和管理工具，例如：备份恢复，mysql复制集群等。
3. connection pool：连接池，管理缓冲用户连接、用户名、密码、权限检验、线程处理等需要缓存的需求。
4. sql interface：sql接口。接收用户的sql的用户命令，并且返回用户查询的结果。比如select * from就是调用sql interface。
5. parser：解析器。sql命令传到解析器时会被解析器验证和解析。解析器是由Lex和YACC实现的。
6. optimizer：查询优化器，sql语句在执行之前会使用优化器优化。
7. cache和buffer（高度缓存区）：查询缓存。如果查询缓存有命中的结果，数据将会在缓存中读取返回。
8. pluggable storage enginies：插件式存储引擎。存储引擎是mysql中的具体的与文件打交道的子系统，即不同的存储引擎数据文件的存储方式可能不一样。也是mysql最具特色的一个地方，如innodb，myisam等。mysql的存储引擎是插件式的、热插拔的。
9. file system：文件系统。数据、日志（redo，undo）、索引、错误日志、查询记录、慢查询等。



## mysql运行时机理图

![img](youdaoyun.assets\25666d9d98d147b5a6216580bb637a40\665.jpg)

# **索引**

## **什么是索引？**

首先我们要明确，正确的创建合适的索引是提升数据库查询性能的基础。mysql的索引一般存在硬盘或者磁盘中，存在内存中的话会极大耗费资源并且没有持久化数据容易丢失。

索引的定义：是为了加速对表中的数据行的检索而创建的一种分散存储的数据结构。

如图，一个简单的例子：

![img](youdaoyun.assets\b9e9c361b54d4fd1961cfe43f6a0919c\clipboard.png)

## **为什么用索引**

1. 索引能极大的减少存储引擎需要扫描的数据量
2. 索引能够把随机IO变成顺序IO
3. 索引可以帮助我们在进行分组、排序等操作时，避免使用临时表。

。。。

## **Mysql为什么使用B+Tree作为索引存储的数据结构**

### **二叉查找树（binary search tree）**

二叉树是由n（n>=1）个有限节点组成的具有层次关系的集合。

![img](youdaoyun.assets\bbdec614230d44ef85c85956f50ae89f\clipboard.png)

### **平衡二叉查找树（balanced binary search tree）**

平衡二叉树又称AVL树。它是一颗空树或者它的左右两个子树的高度差的绝对值不超过1，并且左右两个子树都是一颗平衡二叉树。如下图：

![img](youdaoyun.assets\8982f635c7fb4d8e9f23700ade78a6ff\clipboard.png)

如果使用这种结构作为索引的数据结构，每个节点包括关键字（一般是索引字段值）、数据区（一般存储索引字段值对应的那行数据的地址指针）和子节点引用。假如索引的值是10，则在根节点中就命中了，之后从对应的地址中获取到数据返回。

这种结构有明显的缺点：太深、太小。

- 太深：数据处的（高）深度决定着它的IO操作次数（遍历一个节点就需要一次IO操作），而IO操作是很耗费资源的，所以IO耗时大。
- 太小：每一个磁盘块（节点/页）节点只存储一个关键字，保存的数据量太少了。并且没有很好的利用操作磁盘IO的数据交换特性，linux操作系统中，一次IO操作可加载4Kb（mysql中每一个节点的数据大小小于等于16K，相当于四次IO操作）的数据，只存储一个关键字太浪费了。同时它也没有利用好磁盘IO的预读能力（就是你加载某一块的数据，系统会帮你预读附近内存的数据，也就是说假如你加载4k数据，实际系统会帮你预读更多的数据，返回的可能是8k，16k），即空间局部性原理，从而带来了频繁的IO操作。

空间局部性：如果一个存储器的位置被引用，那么将来他附近的位置也会被引用。

### **多路平衡查找树**

多路平衡查招树是一颗绝对平衡的树（对于任意一个节点，左右子树高度相同）。这种结构相对于平衡二叉查找树它每一个节点的存储的关键字较多。如图，当关键字为17时，查找二叉树的根节点就能命中，当小于17时往左边遍历，大于17小于35时往P2指定的节点遍历，大于35往右边节点遍历，以此类推。

![img](youdaoyun.assets\55778f99f55946098a3ae2d35f592638\clipboard.png)

虽然这种结构一个节点存储的内容比较多，但mysql中使用的是加强版的多路平衡查招数B+Tree

### **多路平衡查找B+Tree**

如图，树的除叶子节点外都存储关键子等信息，只有叶子节点才存储数据（地址），这样就能最大化的将减少IO读取次数。

![img](youdaoyun.assets\d80acfd10896470289784b0bcb4e0f8d\clipboard.png)

### **B+Tree和B-Tree的区别**

- B+节点搜索采用左闭合区间
- B+非叶节点不保存数据相关信息，只保存关键字和子节点的引用
- B+关键字对应的数据保存在叶子节点信息
- B+叶子节点是顺序排列的，并且相邻节点具有顺序关系的引用关系，即数据的存储是连续的。

### **选用B+Tree的原因**

- B+Tree是B-Tree的变种多路平衡查找树，它具有B-Tree的优点。
- B+Tree扫库、表能力更强
- B+Tree磁盘读写能力更强（一次IO操作能加载更多的关键字）
- B+Tree的排序能力更强（顺序IO）
- B+Tree的查询效率更加稳定（由于B+Tree要找到某些数据需要遍历到叶子节点，故多次查找数据所花的时间是相差不少的，而B-Tree多次查询具有不稳定性，即它可能在根节点就命中了索引，也可能到最深的节点才能命中索引）

## **Mysql B+Tree索引体现形式**

###  **Myisam中的体现形式**

![img](youdaoyun.assets\8d9a2c75f44d4a12b2e1a7b6fb424825\clipboard.png)

查看mysql表文件存储路径：show global variables like "%datadir%"；

在mysql中，frm格式文件是表定义文件，每个存储引擎中都有。在mysiam引擎中，MYI格式文件是存储索引的文件，MYD格式文件是存储数据的文件。

如图，在Myisam存储引擎中，teacher.MYI格式文件存储的是索引，叶子节点中存储的是关键字对应那行数据的地址指针。

对于字符串类型的索引列，字符串的排序是基于最左匹配原则进行排序的，同理，叶子节点中存储的地址指向对应数据的存储位置。

![img](youdaoyun.assets\726bffeb340040cc9caea4949a2c4be3\clipboard.png)

### **Innodb中的体现形式**

**聚集索引**

![img](youdaoyun.assets\4b2562ada2014dbdbe3dd721ad8c9b30\clipboard.png)

首先理解下聚集索引的概念：数据库表行中数据的物理顺序与键值的逻辑（索引）顺序相同。当建表时，如果指定了索引，则数据的存储顺序则有这个主键决定，即与它的逻辑顺序相同。

在Innodb存储引擎中，数据和索引都存储在IBD文件中，不同于myisam模型的是，它叶子节点中存储的是对应的行数据。

**非聚集索引**

![img](youdaoyun.assets\f847f8fac8414c4181b6b6c8ce4fe35b\clipboard.png)

如图，如果存在非聚集索引（辅助索引），辅助索引维护为一个单独的索引，它叶子节点指向主键索引树根节点。

以辅助索引列作为查询条件时则会先遍历辅助索引树，然后主键索引列对应的主键关键字去遍历主键索引树遍历查找数据。这样存储的好处是当数据发生迁移时，辅助索引不需再去维护，只需维护主键索引。

而在myisam存储引擎中，辅助索引树的存储形式和主键索引树是类似的，它的叶子节点存数的数据同样指向对应表中的行数据。

![img](youdaoyun.assets\96dbf9bb90c5451389f99638eb77f3ba\clipboard.png)

## **索引知识补充**

### **列的离散型**

当数据的相似性约低时，则可以认为离散型高。 离散型越高，选择性越好。

![img](youdaoyun.assets\a1a65c9b9c1241e985916aa7baa0e160\clipboard.png)

### **最左匹配原则**

对索引关键字计算（对比），一定是从左到右依次进行，且不可跳过。

![img](youdaoyun.assets\e9f8dc0d8eee4e6eac162eb5cbaf18f3\clipboard.png)

### **联合索引**

即索引列有多个，匹配时根据最左匹配原则进行匹配，所以要特别注意索引列的顺序，较常用到的放在左边，其实单列索引也是联合索引，不过它是特殊的联合索引。

### **联合索引列选择原则**

- 经常用的列优先（最左匹配原则）
- 选择性（离散型）高的列优先（离散度高原则）
- 宽度小的列优先（最小空间原则）

覆盖索引：即查询的数据列就在索引列当中

总结

![img](youdaoyun.assets\245821d900d742f1b61e5f87617d469a\clipboard.png)

索引为null的情况

B+Tree

数据结构模拟网站

https://www.cs.usfca.edu/~galles/visualization/Algorithms.html





# **mysql插拔式的存储引擎**

## **存储引擎的特点**

1. 是插拔式的插件
2. 存储引擎是制定在表之上的，即一个库中的每一个表都可以指定一个存储引擎
3. 不管采用怎样的存储引擎，都会在数据区产生一个对应的frm表结构定义描述文件

### **CVS存储引擎**

cvs存储引擎是以cvs格式来存储文件的。

**特点：**

- 不能定义没有索引、列定义必须为not null、不能设置自增列，所以不适用大表或者数据的在线处理。
- 数据的存储用，隔开，可直接编辑cvs文件进行数据的编排，所以安全性很低

（编辑后，执行flush table XXX才能生效）

**应用场景：**

- 数据的快速导出导入
- 表格直接转化为cvs

### **archive存储引擎**

压缩协议进行数据的存储，数据存储为arz文件格式

**特点**

- 只支持insert和select两种操作
- 只允许自增id列建立索引
- 行级锁
- 不支持事务
- 数据占用磁盘少

**应用场景**

- 日志系统
- 大型的数据采集设备

### **memory存储引擎**

数据都是存储在内存中，IO效率比其他引擎高很多。但是重启后数据会丢失，内存数据表默认只有16M。

**特点**

- 支持hash索引，b+tree索引，默认时hash索引，查找时间复杂度为O（1）。
- 字段长度都是固定的varchar(32)=char(32)。
- 不支持大数据存储类型字段，如blog，text。
- 支持表级锁。

**应用场景**

- 等值查找热度较高数据（hash索引通过计算就能找到对应数据的位置）。
- 查询结果内存中的计算，大多数是采用这种存储引擎作为临时表存储需计算的数据。

### **myisam**

mysql5.5版本之前默认的存储引擎，较多的文件系统表也是采用这种存储引擎。系统临时表也会	用到这种存储引擎存储。

**特点**	

- select count(*) from table无需进行数据的扫描。
- 数据（MYD格式）和索引（MYI）分开存储。
- 表级锁
- 不支持事务

Innodb

mysql5.5之后建表时默认的存储引擎。

**特点**

- 支持事务ACID的控制
- 行级锁
- 以聚集索引（主键索引）方式进行数据的存储
- 支持外键关系保持数据之间的完整性

**各存储引擎的对比参考官网：**

https://dev.mysql.com/doc/refman/5.7/en/storage-engines.html



# mysql查询优化详解

## **mysql查询的执行路径**

![img](youdaoyun.assets/73051eaedcbd4319a4c67a557721df84/clipboard.png)

1. **mysql的客户端和服务端进行通信**。

mysql客户端和服务端的通信方式是半双工的。	

半双工：双向通信，同时只能是发送或者是接收。

全双工：可以同时发送或者接收。

特点和限制：客户端一旦开始接收消息，另一端要接收完整个消息才能响应。即客户端一旦开始接收数据就没法停下来发送指令。

可以通过语句查看当前客户端和服务端的通信状态：

show processlist / show full processlist  

例：

![img](youdaoyun.assets/8dc556d57bc8454f87ec3c4542e9a08f/clipboard.png)

常见状态：

- sleep：线程正在等待客户端发送数据。
- query：连接线程正在发起查询。
- locked：线程正在等待表锁的释放。
- sorting result：线程正在对结果经行排序。
- sending data：向请求端返回数据。

可以通过kill id	的方式将对应连接杀掉。

1. **查询缓存，如果有则直接返回**。

工作原理：缓存select操作的结果集和sql语句。如果是新的select语句，先去查找缓存，判断是否有存在可用的记录集。

判断标准：与缓存的sql语句是否完全一样，区分大小写（简单理解为一个key-value结构，key为sql语句，value为sql查询结果集）。

查看当前缓存配置参数：show variables like '%query_chahe'

![img](youdaoyun.assets/ee5fea49e3474608b477631989fe05a3/clipboard.png)

- query_cache_type：查询缓存类型，是否打开缓存。

a.（OFF）：未打开缓存，默认值。

b.（ON）：启用查询缓存，但在select中加入SQL_NO_CHAHE后，将不会使用缓存。

c.（DEMAND）：启用查询缓存，但只有在select语句中加入SQL_CHCHE后，才会使用缓存。

- query_cache_size：缓存使用的总内存空间大小，单位是字节。允许设置最小为40k，默认为1m（必须是1024的整数倍），推荐设置为64m/128m。
- query_cache_limit：允许缓存的单条查询结果集的最大容量，默认为1m，超过此参数的结果集将不会被缓存。
- query_cache_min_res_unit：分配内存时的最小单位大小，设置查询缓存每次分配的最小内存空间大小，即缓存结果集最小占用的内存空间大小。
- query_cache_wlock_invalidate：如果某个数据表被锁住，是否依然从缓存中返回数据，默认是off，表示仍然可以返回。

通过语句可以查看缓存情况：show status like 'Qcache%'

![img](youdaoyun.assets/1321d73aa32d4096a611675d0e56dd4f/clipboard.png)

- qcache_free_blocks：缓存池中空闲块的个数
- qcache_free_memory：缓存中空闲内存量
- qcache_hits：缓存命中次数
- qcache_inserts：缓存写入次数
- qcache_lowmen_prunes：因内存不足删除缓存次数
- qcache_not_caches：查询未被缓存次数，例如查询结果集查过缓存块大小，查询中包含可变函数等
- qcache_queries_in_cache：当前缓存中缓存的sql数量
- qcache_total_blocks：缓存总block数

例子：

打开缓存：修改安装目录中的my.ini文件

![img](youdaoyun.assets/08ff3e924fbe4057ab48ac35be4552ec/clipboard.png)

![img](youdaoyun.assets/5ab615940cf64826bccea2ee026f5bf5/clipboard.png)

重启mysql服务：net stop mysql 

net start mysql

可以看到配置参数已经改变：

![img](youdaoyun.assets/89279e53d7f54b5fb6218f166d0d224a/clipboard.png)

未执行查询前：

![img](youdaoyun.assets/8630289255be493fae1f7d06060bd275/clipboard.png)

先执行一条语句，插入了缓存

![img](youdaoyun.assets/aca089051dfa43d3b7da997f626087d3/clipboard.png)

再执行一遍，命中了缓存

![img](youdaoyun.assets/233a909437d247cd8fddb0c675124d43/clipboard.png)

**注意：有些情况不会缓存**

- 当查询语句中有一些不确定的数据时，则不会缓存。如包含函数now(),current_date()等类似的函数时，或者用户自定义的函数，存储过程，用户变量等都不会缓存。
- 当查询的结果集大于query_cache_limit设置的值时，不会查询缓存。
- 在innodb中引擎中，当在某个事务中修改了某个表，在这个事务提交之前，所有与这个表相关的查询都无法被缓存。所以长时间的执行事务，会大大降低缓存的命中率。
- 当查询的表是系统表时。
- 查询语句不涉及到表。

**在mysql中，查询缓存为什么默认时关闭的？**

- 在查询之前需要检查是否命中缓存，浪费系统资源。
- 当一个查询可以被查询时，执行后mysql发现其未被缓存的话则会加入缓存，这样会带来额外的系统消耗。
- 针对表进行写入或者更新的操作时，对应的表相关的缓存会失效。
- 如果查询缓存很多或者碎片很多时，这个操作可能会带来很多的系统资源消耗。

**使用场景**

以读为主的业务，数据生成之后不常改变的业务。比如门户类、新闻类、报表类、论坛类等。

1. **查询优化处理**。

**查询优化处理的三个阶段**：

- **解析sql**：通过lex词法分析，yacc语法分析将sql语句解析成解析树。

https://www.ibm.com/developerworks/cn/linux/sdk/lex/

- **预处理阶段**：根据mysql的语法规则进一步检查解析树的合法性。如检查数据表和列是否存在，解析名字和别名的设置，还会进行权限的验证等。
- **查询优化器**：优化器的作用就是找到最优的执行计划。

**查询优化处理时如何找到最优执行计划**

- **使用等价变换规则**

如：5=5 and a>5 改写为a>5

a<b and a=5改写为b>5 and a=5

基于联合索引，调整位置等。

- **优化count、min、max函数等**

min函数只需找到索引最左边

max函数只需找到索引最右边

myisam引擎中count(*)结果可以直接获取

- **覆盖索引扫描**
- **子查询优化**
- **提前终止查询**

即使用limit关键字或者使用不存在的条件

- **in的优化**

先进行排序，然后采用二分查找进行查询

......

mysql的查询优化器是基于成本计算的规则。他会通过数据采样的方式（随机的读取一个4k的数据块进行分析）尝试各种执行计划。

查看执行计划的语句：explain 【查询语句】 \G

![img](youdaoyun.assets/273e25555b0e48368b3f5d19afe1bb10/clipboard.png)

**执行计划-id**：select查询的序列号，标识运行的顺序。

- 如果id相同，计划从上往下执行。
- id越大，优先级越高，越早执行。

**执行计划-select_type**：查询的类型，主要区分普通查询、联合查询、子查询等。（测试mysql版本：5.7.22）

- simple：简单的select查询，查询中不包含子查询或者union。

例子如上图

- primary：查询中包含子部分，最外层查询则被标记为primary。（这个例子是网上抄的，在5.7.22版本中貌似被优化了？）

![img](youdaoyun.assets/a53f031a2caa48c8803f3f85db2b5910/clipboard.png)

- subquery/materialized：subquery表示在select或where列表中包含了子查询。materialized表示where后面in条件的子查询。

![img](youdaoyun.assets/7570d3fdec4442d89e43171558e64f92/clipboard.png)

- union：若第二个select出现在union之后，则标记为union。

![img](youdaoyun.assets/28c3158d332848a0b9c8d80ed4507d94/clipboard.png)

另一个语句是simple，为什么？

![img](youdaoyun.assets/c3ee0f411af14d2f988f5612c2f38901/clipboard.png)

- union result：从union表获取结果的select。即union的结果，一般没有id
- dependent union：第一个select的结果查询条件依赖union的结果。

![img](youdaoyun.assets/9ad46d76b0294bf4af3ca08dc695715b/clipboard.png)

- derived（导出）：从子查询中导出的结果（from 子句的子查询），如

![img](youdaoyun.assets/d1f3309f0ba94b9cbc8a3ae9cc10dcd8/clipboard.png)

再看一个例子，这个语句应该也会有derived，但貌似mysql把他优化为simple类型了（mysql版本：5.7.22）。

![img](youdaoyun.assets/d013e446a9b8451eb4b3a91f961b36f9/clipboard.png)

- dependent subquery：子查询中的第一个select，取决于外面的查询。例子同dependent union结果。

**执行计划-table**

直接显示表名或者表的别名：

- <union M,N>由id为M、N查询union产生的结果
- <subqueryN>由id为N查询产生的结果

**执行计划-type**

访问类型，sql查询优化中一个很重要的指标，结果值从好到坏依次是：

system>const>eq_ref>ref>range>index>ALL

- system：表只有一行记录（等于系统表），const类型的特例，基本不会出现，可以忽略不计。
- const：表示通过索引一次就找到了，const用于比较primary key或者unique索引。

![img](youdaoyun.assets/939ecd5ae1ae4c70bc7b2708f9596592/clipboard.png)

- eq_ref：唯一索引扫描，对于每个索引键，表中只有一条记录与之匹配。常见于主键或者唯一索引扫描。
- ref：非唯一索引扫描，返回匹配某个单独值的所有行，本质上也是一种索引访问。
- range：只检索给定范围的行，使用一个索引来选择行。
- index：full index scan，索引全表扫描，把索引从头到尾扫一遍。
- All：全表扫描。

**在实际应用中，type值最好是range以上级别**

**执行计划其他参数**

- prossible_key：查询过程中有可能用到的索引。
- key：实际使用的索引，如果是null，则没有使用索引。
- rows：根据表统计信息或者索引使用情况，大概估算出找到所需记录所需要读取的行数。
- filtered：表示返回的结果行占读取行的百分比。值越大越好。

**执行计划-extra**

![img](youdaoyun.assets/e301bc1bea0145049f390462f23d80c6/clipboard.png)

1. 查询执行引擎：调用插件式的存储引擎的原子api的功能进行执行计划的执行
2. 返回结果给客户端

a.有需要缓存的，先执行缓存操作

b.增量的返回结果：开始生成第一条结果时，mysql就开始给请求返回数据。

好处：mysql无法保存过多的数据，浪费内存。同时用户能立刻拿到数据，体验好。

**执行计划-partitions**：划分信息，非划分表值为null

更多https://dev.mysql.com/doc/refman/5.7/en/partitioning-info.html

**执行计划-ref**：表示什么字段或者常量去跟索引字段比较，如where name='tom'（name为索引）时，显示‘const’（constants）

**如何定位sql慢**

1. 业务驱动
2. 测试驱动
3. 慢查询日志

**慢查询日志相关配置**

**相关常用命令：**

show variables like 'slow_query_log%'

set global slow_query_log=on

set global slow_query_log_file = '/var/lib/mysql/gupaoedu-slow.log'

set global log_queries_not_using_indexes = on

set global long_query_time = 0.1 (秒)

**日志分析**

![img](youdaoyun.assets/f8eecaf6e2b04b44868db2e9cd1235ae/clipboard.png)

**慢查询日志查询工具推荐**

![img](youdaoyun.assets/04d2c693610d4c8998eb156f9a0e1fd9/clipboard.png)

**count(\*)、count(1)和count(col)的区别**

count(*)和count（1）执行效果是一样的，都是统计表中所有符合条件的记录数，扫描的是主键索引。而count(col)是计算所有符合条件的col的记录数。

例：

user表，id是唯一索引列

数据：

![img](youdaoyun.assets/a3f7f4e2f89945598868a183e074f882/clipboard.png)

执行count（1）

![img](youdaoyun.assets/0d1c1407e7af424a8d4b7b072a1ac8ea/clipboard.png)

对应执行计划：使用了索引扫描，命中了索引

![img](youdaoyun.assets/3355c56da3bf40c78f29988fae37e410/clipboard.png)

执行count(col)

![img](youdaoyun.assets/24f1cabe43774684aadd7415c6d94c1e/clipboard.png)

对应执行计划：是全表扫描，所以对应的列如果没有索引的话，效率可能会很低

![img](youdaoyun.assets/c374763d8f08425ab6d0ce873ac3cc61/clipboard.png)

再看看count(id)的执行计划：是与count(1)一样的效果，可以认为count(1)或者count(*)默认扫描的是主键索引。

![img](youdaoyun.assets/41d40f92ac094bb486183d2fdf101925/clipboard.png)

sql标准

一条sql只能用一个索引







锁始终是锁在索引上的

记录锁时的非唯一索引时，锁住的行（有点奇怪）

脏页

自增锁

s锁

利用锁解决不可重复读（S共享锁是怎么解决的：S锁只兼容S锁，而X锁既不兼容S锁，也不兼容X锁）

查看事务隔离级别：

select @@global.tx_isolation;

select @@tx_isolation;



**快照读和当前读**

- 快照读：又叫一致非锁定读，读取的数据是历史版本的数据，普通的select就是快照读。快照的数据是通过undo获得的。

有两个事务，A事务种查询读取的都是历史数据，隔离级别为read-commited时，读取的是最新的快照，而隔离级别为repeatable-read时读取的是事务开始时的快照，故这种隔离级别的读取的数据一致，而read-commited种不一致，即违反了ACID中隔离性

![img](youdaoyun.assets/923bcec02479490cb45f2650c54103e6/clipboard.png)

- 当前读：读取的数据是最新版本的数据。是通过锁机制保证数据不能被修改，确保数据的最新。update、delete、insert、select ... from table for update 等都是当前读。

**对读取的数据加锁：**

- select ... for update：加了X锁。其他事务想往被锁定的行中加任何锁都会被阻塞。
- select ... lock in share mode：加了S锁。其他事务申请S锁才不会被阻塞。

这两条语句需加begin、start transation或者set autocmmit=0时才有效（即要在一个事务中）。

**自增长和锁**

自增锁的目的是保证自增列的数据的不重复。innodb_autoinc_lock_mode参数指定对应的策略。

- 0：是5.1.22版本之前自增长的实现方式，即通过表锁的auto-inc locking方式，即表锁，多个事务操作时会阻塞。
- 1：默认值。对于”simple inserts“（插入前能确认插入行数的语句），该值会用互斥量去对内存中的数据进行累加的操作。而对于”bulk inserts“（插入前不能确认插入的行数），则使用anto-inc locking的方式产生值。
- 2：对于”insert-like“（全部插入）语句都采用累加互斥量的方式。这种方式产生的数据可能不是连续的，所以statement-base replication 可能会出现问题（不连续），而row-base replication则不会出现问题。

![img](youdaoyun.assets/de29d20ee6574f23a80f2dbad965cff9/clipboard.png)

**外键和锁**

在innodb存储引擎中，对于一个外键列，如果没有显示的建立索引，innodb引擎会自动给这列加上索引，以防止表锁出现，而oracle中不会自动添加，需要手动添加，所以oracle中死锁问题会很多（锁表）。

如下例：

![img](youdaoyun.assets/27a2abc305eb4cb6a4d063da19eff783/clipboard.png)

session B中的插入操作需要select父表操作，这种查询不是一致性非锁定读的方式（都快照），而是select ... lock in share mode的方式，故会尝试获取S锁。session A中已经往id=3的记录加上了X锁，故session B会被阻塞。

**锁的算法**

- record lock：单个记录的上的锁。总会锁住索引记录，如果innodb中的表没有任何索引，这是innodb会使用隐式的主键进行锁定。
- cap lock：间隙锁，锁定一个范围，但不包含记录本身，即左开右开。
- next-key：record lock+cap lock共同实现的，锁定一个范围，包含记录本身，左开右闭。这种算法是repeatable read模式下的默认的行记录锁定算法。

例子：

![img](youdaoyun.assets/969a8c1b130641f5b514fa3f2847dace/clipboard.png)

这种情况下，锁定的记录是（-∞，6]，故5、6数据是插不进去的。

![img](youdaoyun.assets/a3d4521adb214c6d8d6cd5ed1b651243/clipboard.png)

这种情况下，不需gap lock，只需record lock，故锁定的记录只有7一条

待续。。。

**锁问题**

- 丢失更新：如转账问题
- 脏读
- 不可重复读





# `mysql binlog`日志三种模式

- Row：记录每一行记录变更的语句，如批量查询会记录每一行的改动，对存储过程、函数等特殊情况也能很好记录，但耗性能，非常“沉重”
- Statement：根据执行语句记录，如批量查询只记录一条语句，对存储过程、函数等特殊情况也能很好不能很好记录
- Mixed：混合上述两种





# 实际问题

## longtext、text数据问题

https://blog.csdn.net/u013099854/article/details/115007969