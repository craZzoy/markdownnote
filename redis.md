# redis中的数据结构

分布式锁使用场景：https://www.yisu.com/zixun/89793.html

缓存大致可分为两类：

- 应用内缓存
  - Map
  - EH cache
- 组件缓存
  - Memached
  - Redis：remote dictionary server（远程字典服务器），redis是一种基于key-value的高性能的存储系统，通过提供多种数据类型来适应不同场景下的缓存和存储需求。



## 安装（5.0）

1. 下载redis的安装包

2. tar -zxvf 解压

3. cd 到解压后的目录

4. 执行make 完成编译 

5. make test

6. make install

   

## redis启动和停止

启动redis服务器：

- 配置守护线程：在配置文件中加入：daemonize yes 

- src/redis-server
- src/redis-server redis.conf（启动端口在配置文件中修改）
- src/redis-server --port 7021

停止服务：确不可通过终止进程命名强行关闭redis，因为它可能还有未完成的任务，强行终止可能会导致数据丢失。当redis收到shutdown命令时，会先断开所有客户端连接，然后根据配置执行持久化，之中完成工作。

- redis-cli shutdown

启动客户端

- src/redis-cli
- src/redis-cli -p 6379



redis常用命令查询：

- 官网：redis.io
- redisdoc.com



## 数据类型

### 字符串类型

​	字符串类型是redis中最基本的数据类型，可存储任意形式的字符串，包括二进制数据（采用了安全的二进制存储方式：非c字符型：‘h’'e''l''\0''h' ，以'\0'结尾，读不到'h'，在sds结构中，通过len判断字符串的结尾）。一个字符串允许最大存储容量为521m。

#### 内部存储结构

String类型

- int：用于整型数据

- SDS（simple dynamic string）：存放字节、字符串、和浮点数据。redis中引入了五种sdshdr类型：

  ```c
  typedef char *sds;
  
  /* Note: sdshdr5 is never used, we just access the flags byte directly.
   * However is here to document the layout of type 5 SDS strings. */
  struct __attribute__ ((__packed__)) sdshdr5 {
      unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
      char buf[];
  };
  struct __attribute__ ((__packed__)) sdshdr8 {
      uint8_t len; /* used */
      uint8_t alloc; /* excluding the header and null terminator */
      unsigned char flags; /* 3 lsb of type, 5 unused bits */
      char buf[];
  };
  struct __attribute__ ((__packed__)) sdshdr16 {
      uint16_t len; /* used */
      uint16_t alloc; /* excluding the header and null terminator */
      unsigned char flags; /* 3 lsb of type, 5 unused bits */
      char buf[];
  };
  struct __attribute__ ((__packed__)) sdshdr32 {
      uint32_t len; /* used */
      uint32_t alloc; /* excluding the header and null terminator */
      unsigned char flags; /* 3 lsb of type, 5 unused bits */
      char buf[];
  };
  struct __attribute__ ((__packed__)) sdshdr64 {
      uint64_t len; /* used */
      uint64_t alloc; /* excluding the header and null terminator */
      unsigned char flags; /* 3 lsb of type, 5 unused bits */
      char buf[];
  };
  ```

  在创建一个sds时会根据sds的实际长度判断应该选用什么类型，默认是sdshdr8：

  ```c
  sds sdsnewlen(const void *init, size_t initlen) {
      void *sh;
      sds s;
      char type = sdsReqType(initlen);
      /* Empty strings are usually created in order to append. Use type 8
       * since type 5 is not good at this. */
      if (type == SDS_TYPE_5 && initlen == 0) type = SDS_TYPE_8;
      int hdrlen = sdsHdrSize(type);
      unsigned char *fp; /* flags pointer. */
      ...
  }
  ```

  可查看sdshdr8具体结构：

  ```c
  struct __attribute__ ((__packed__)) sdshdr8 {
      //表示当前sds长度（字节）
      uint8_t len; /* used */
      //已经为sds分配的内存大小（字节）
      uint8_t alloc; /* excluding the header and null terminator */
      //用一个字节标识数据类型
      /** 00000000 sdshdr5
      * 	00000001 sdshdr8
      *	00000010 sdshdr16
      *	00000011 sdshdr32
      *	00000100 sdshdr64
      */
      unsigned char flags; /* 3 lsb of type, 5 unused bits */
      //sds实际存放的位置
      char buf[];
  };
  ```

#### 常用命令

- GET
- SET
- SETNE
- SETEX
- INCR：原子递增操作，不要执行  GET A, SET A A+1

#### 使用场景

- 存储token

### 列表类型

​	列表类型可以存储一个有序的字符串列表，常用的操作是向列表两端添加元素或者获得列表中的某一个字段。列表类型内部使用链表实现，这种结构意味着往列表中前后端查找元素时间复杂度只为O（1），不过数据量大时查询效率较低，需遍历列表。

​	元素最大个数：2^32-1

#### 内部数据结构

​	redis3.2之前，列表中的valued对象内部以linkedlist或者ziplist实现，当list元素个数和单个元素个数的长度较小时将采用ziplist（压缩列表）实现，以此较少内存开销，否则将采用linkedlist实现。

​	redis3.2之后，则采用一种新的类型quicklist来存储list数据。

两种数据类型对比：

- 不压缩的双向链表，对于两端push、pop，插入操作复杂度较低，但内存消耗较大
- 压缩的ziplist，存储在一块连续的内存中，所以存储效率高，但插入、删除等操作都会频繁的申请和释放内存。

​	quicklist仍然是一个双向链表，只是列表的每一个节点都是ziplist或者quicklistLZF（使用LZF压缩算法压缩的quicklist）：

```c
/* Node, quicklist, and Iterator are the only data structures used currently. */
 
/* quicklistNode is a 32 byte struct describing a ziplist for a quicklist.
 * We use bit fields keep the quicklistNode at 32 bytes.
 * count: 16 bits, max 65536 (max zl bytes is 65k, so max count actually < 32k).
 * encoding: 2 bits, RAW=1, LZF=2.
 * container: 2 bits, NONE=1, ZIPLIST=2.
 * recompress: 1 bit, bool, true if node is temporarry decompressed for usage.
 * attempted_compress: 1 bit, boolean, used for verifying during testing.
 * extra: 12 bits, free for future use; pads out the remainder of 32 bits */
typedef struct quicklistNode {
  //quicklistNode的节点定义
    struct quicklistNode *prev;//前驱节点     8字节 32bit
    struct quicklistNode *next;//后继节点     8字节 32bit
    unsigned char *zl;//使用ziplist或者lzf编码的数据 8字节 32bit
    unsigned int sz;             /* ziplist size in bytes */
    //zl所占的字节数  32bit 4字节
    unsigned int count : 16;     /* count of items in ziplist */
    //zl所包含的元素个数 16bit
    unsigned int encoding : 2;   /* RAW==1 or LZF==2 */
    //zl的编码模式1为ziplist 2为lzf  2bit
    unsigned int container : 2;  /* NONE==1 or ZIPLIST==2 */
    //zl的数据是否被压缩，压缩则为2，否则为1.  2bit
    unsigned int recompress : 1; /* was this node previous compressed? */
    //quicklist是否已经压缩过  1bit
    unsigned int attempted_compress : 1; /* node can't compress; too small */
    //测试使用
    unsigned int extra : 10; /* more bits to steal for future usage */
    //预留位 10bit
    //该数据结构总共占32个字节 
} quicklistNode;
```

![1568179257424](redis.assets\1568179257424.png)

#### 常用命令

- (B)LPUSH、(B)RPUSH
- (B)LPOP、(B)RPOP

#### 使用场景

- 实现栈：LPUSH+LPOP

- 队列：LPUSH+RPOP

- 实现消息队列：LPUSH+BRPOP（阻塞获取）

  

### Hash类型

![1568185680715](redis.assets\1568185680715.png)

#### 数据结构

​	map提供两种数据结构存储

- ziplist：数据量小时使用
- hashTable：分为了三层

![1568188811050](redis.assets\1568188811050.png)

- dictEntry：管理一个key-value，同时保留同一个桶中相邻元素的指针，用来维护哈希桶的内部链。

  ```c
  typedef struct dictEntry {
      void *key;
      union {
          void *val;
          uint64_t u64;
          int64_t s64;
          double d;
      } v;
      //下一个节点的地址，用来处理碰撞。
      struct dictEntry *next;
  } dictEntry;
  ```

- dictht：实现一个hash表会使用一个buckets存放dictEntry的地址，一般情况下通过hash(key)%len得到的值就是buckets的索引，这个值决定了dictEntry节点数据将存到buckets的哪个索引中。这个buckets实际上就是我们所说的hashtable。dict.h的dictht结构中table存放的就是buckets的地址 。

  ```c
  typedef struct dictht {
      dictEntry **table;  //buckets的地址
      unsigned long size; //buckets的地址，总保持为2^n
      unsigned long sizemask;//掩码，用来计算hash值对应的buckets索引
      unsigned long used;//当前dictht有多少个dictEntry节点
  } dictht;
  ```

- dict：这是hash数据结构的核心，定义了两个dictht，用于rehash、遍历hash、拓容、缩容等操作

  ```c
  typedef struct dict {
      dictType *type;//dictType里存放的是一堆工具函数的函数指针，
      void *privdata;//保存type中的某些函数需要作为参数的数据
      dictht ht[2];//两个dictht，ht[0]平时用，ht[1] rehash时用
      long rehashidx; //当前rehash到buckets的哪个索引，-1时表示非rehash状态
      int iterators; //安全迭代器的计数。
  } dict;
  ```

#### 常用命令

- HGET
- HSET
- HSETNX
- HEXISTS
- HDEL

#### 使用场景



### 集合（Set）类型

​	集合类型中每个元素都是不同的，并且是无序的。一个集合类型键可以存储至多2^32-1个。

与列表最大的区别：

- 唯一性
- 无序性

​	在集合中加入、删除元素时间复杂度为O(1)，因为其数据结构是使用value为空的散列表（hash table）

#### 内部数据结构

- intset：只包含整数型元素的存储方法
- hashtable：元素不仅仅包含整数型类型的存储方式，用key存值，value为空。

#### 常用命令

- SADD
- SMENBERS
- SPOP：移除并返回一个元素

#### 使用场景

- 查找并集，如共同好友：SUNION



### 有序(Sorted Set)集合

​	和前面所述的集合相比，其是有序的。在集合的基础上，有序集合的每一个元素都关联了一个score，用于排序。每个元素不同，但是score可以相同。

#### 数据结构

zset类型

- ziplist
- skiplist+hashtable

![1568191126111](redis.assets\1568191126111.png)

#### 常用命令

- ZADD
- ZSCORE
- ZINCRBY 
  - ZSCORE salary tom
  - ZINCRBY salary 2000 tom
- ZCOUNT

#### 使用场景

- 打标签：用户等级



# redis的内部原理

## 过期时间设置

- EXPIRE：EXPIRE key second，给某一key设置超时时间，过期后会自动删除。
  - expire age 5
    - 返回
      - 1：设置成功
      - 0：设置失败或者键不存在
- PEXPIRE：跟EXPIRE类似，单位为ms
- TTL：TTL key，查看某一key的过期时间
  - 返回
    - 过期时间（s）
    - -2：key不存在
    - -1：指定的key未设置过期时间
- PTTL：跟TTL类似，单位是ms
- PERSIST：PERSIST key，持久化key，即设定可用于取消某个key的过期时间
  - 返回
    - 1：成功
    - 0：失败或未有指定key
- setex  key second value：添加一个包含过期时间的key
  - setex age 3000 79

### 过期删除的原理

- 消极(passive)方式：当主键访问时发现其已经失效，将其删除

- 积极(active)方式：周期性地从设置了失效时间的主键中选择一部分失效的主键删除。每秒进行10此操作，具体流程是

  1. 随机测试20个带有timeout信息的key
  2. 删除其中过期的key
  3. 如果超过25%的key被删除，则重复执行步骤1

  这是一个简单的概率算法，基于假设我们随机抽取的key代表了全部的key空间。



## 发布订阅模式

​	redis提供的发布订阅模式可用于消息传输，同样可以实现进程间的消息传递。

- 发布者：向指定频道发布消息，所有订阅此频道的订阅者都会收到消息

  - publish channel message：向某个频道发布消息，返回值就是此时收到消息的订阅者数量。

  ```txt
  [root@izwz9c61wsgboaq9aoxis9z ~]# cd /usr/sofeware/redis-5.0.5
  [root@izwz9c61wsgboaq9aoxis9z redis-5.0.5]# src/redis-cli
  127.0.0.1:6379> publish channel1 hello,world
  (integer) 0
  127.0.0.1:6379> publish channel1 hello,world
  (integer) 1
  127.0.0.1:6379> publish channel1 hhhh
  (integer) 1
  127.0.0.1:6379> 
  ```

- 订阅者：订阅一个或多个频道

  - subscribe channel ...：订阅某个频道的消息，执行subscribe命令后会进入订阅状态（阻塞）。

  ```txt
  127.0.0.1:6379> subscribe channel1
  Reading messages... (press Ctrl-C to quit)
  1) "subscribe"
  2) "channel1"
  3) (integer) 1
  1) "message"
  2) "channel1"
  3) "hello,world"
  1) "message"
  2) "channel1"
  3) "hhhh"
  
  ```

### channel类型

- 普通channel

  - subscribe abc

- pattern channel

  - psubscribe ab*

  如下图所示，若

  - client1订阅了channel abc

  - client2订阅了pattern channel *bc

    当producer1执行publish abc hello命令时，两个client都能收到

  ![1568298240484](redis.assets\1568298240484.png)



## Redis中的数据持久化

### RDB

​	根据指定的规则“定时”将内存中的数据存储在硬盘上。当符合条件时，redis会单独创建（fork）一个子进程来进行持久化，会先将数据备份到一个临时文件，备份完成后替换原有的rdb文件，重启时读取这个文件的数据将其加入内存中。

​	fork的作用是创建一个与当前进程一样的进程，其中所有的数据都将与原线程一样。

优点：

- 由子进程负责持久化数据，主进程不负责IO操作，确保了极高的性能
- 如果进行大规模的数据恢复，数据恢复完整性并不是那么敏感，RDB比AOP更高效

缺点

- RDB最后一次持久化的数据可能会丢失

#### 进行数据快照的方式

1. 根据配置规则进行自动快照

   - 在配置文件中可配置规则，save second changekeycount

     - 如默认配置了三条规则（或的关系）

       ```properties
       #900s内有一个以上的key被更改则进行快照
       save 900 1
       #300s内有10个以上的key被更改则进行快照
       save 300 10
       #60s内有10000个以上的key被更改则进行快照
       save 60 10000
       ```

2. 用户执行save或者bgsave命令

   - save：触发执行快照行为，会阻塞来自客户端的请求，直至完成，数据量大时不建议使用。
   - bgsave：后台异步地执行快照动作，同时可以处理客户端的请求
   - lastsave：查看最后一次执行快照时间

   注：自动快照采用的是异步快照动作

3. 执行flushall命令

   - 命令执行后会清除内存中的数据
     - 开启了AOF时，当执行过flushall命令，可将AOF文件中的flushall命令删除，以恢复数据

4. 执行复制时

   - 在主从模式中，redis会初始化时进行自动快照。即此种模式下，即使没有定义生成快照的规则。也没有手动生成快照，仍然会生成快照文件。

#### 关闭RDB

注释conf文件内容：

```properties
#900s内有一个以上的key被更改则进行快照
#save 900 1
#300s内有10个以上的key被更改则进行快照
#save 300 10
#60s内有10000个以上的key被更改则进行快照
#save 60 10000
#加入save  “”
save “”
```



### AOF

​	每次执行命令后将命令保存下来。AOP可以将redis执行的每一条命令追加到硬盘文件中，这一过程可能会降低redis性能，但大部分情况下时可以接受的，另外使用较快的硬盘可以提升AOF的性能。在启动时，redis会逐个执行AOF文件中的命令来将硬盘中的数据载入到内存中，速度相对RDB文件较慢。

#### 开启AOF

​	配置文件中修改参数：appendonly yes(默认false)。开启AOF后，redis中执行的每一条命令都会保存到AOF文件中，文件目录跟RDB文件目录一样。默认文件是appendonly.aof，可通过redis.conf修改

- appendfilenane appendonly.aof

  ```properties
  appendonly yes
  
  # The name of the append only file (default: "appendonly.aof")
  
  appendfilename "appendonly.aof"
  
  ```

#### AOF实现

​	AOF文件以纯文本的方式记录redis执行的每一条命令，如执行下面命令：

```properties
set foo 1
set foo 2
set foo 3
get foo
set name tom
```

生成的AOF文件，由此可见只会保存一些设值命令

```properties
*2
$6
SELECT
$1
0
*3
$3
set
$3
foo
$1
1
*3
$3
set
$3
foo
$1
2
*3
$3
set
$3
foo
$1
3
*3
$3
set
$4
name
$3
tom

```

​	由上面生成的文件中可以看出，对于恢复数据，前面两条命令是多余的，可以不保存。去掉这些冗余的命令，可以配置一个条件，每当达到一定条件时重写AOF文件，以去掉冗余命令：

- auto-aof-rewrite-percentage：表示当目前的AOF文件大小超过上一次重写时的AOF文件大小的百分之几时会再次重写，若没有重写过，则以启动时AOF文件大小为依据
- auto-aof-rewrite-min-size：表示限制了允许重写的最小AOF文件的大小

​	当然，也可以手动触发AOF文件重写

- bgrewriteaof，重写后文件

  ```properties
  REDIS0009ú      redis-ver^E5.0.5ú
  redis-bitsÀ@ú^EctimeÂÝs{]ú^Hused-memÂ^P^F^M^@ú^Laof-preambleÀ^Aþ^@û^B^@^@^Dname^Ctom^@^CfooÀ^Cÿ§=uO^O<8c>ZÅ
  ```

#### AOF的重写原理

​	重写的流程，主进程会fork一个子进程进行AOF重写，但这个重写并不是基于原有的aof文件进行的，而是根据当前根据当前内存中的数据重写aof文件。

​	在重写过程中，服务端仍可以处理客户端的请求，重写过程中当有新数据存入或更新时，此时数据会缓存到aof_rewrite_buf中，当子进程完成后会把缓存中的数据追加到aof文件。

​	重写过程中会生成一个新的aof文件，重写完成后再替换原来的aof文件。如果rewrite过程中出现故障，不会影响原来aof文件的正常运行，只有rewrite时才会切换文件。因此rewrite过程是比较可靠的。

​	aof和rdb两者都存在，默认从rdb加载（?）。



### redis内存回收策略

​	当内存不足时，redis会淘汰内存中的对象以保证程序的正常运行。策略有：

- 默认策略为noeviction策略，即当内存使用达到阈值时，所有申请内存的命令会报错。redis.conf中有注明：

  ```properties
  # maxmemory <bytes>
  
  # MAXMEMORY POLICY: how Redis will select what to remove when maxmemory
  # is reached. You can select among five behaviors:
  #
  # volatile-lru -> Evict using approximated LRU among the keys with an expire set.
  # allkeys-lru -> Evict any key using approximated LRU.
  # volatile-lfu -> Evict using approximated LFU among the keys with an expire set.
  # allkeys-lfu -> Evict any key using approximated LFU.
  # volatile-random -> Remove a random key among the ones with an expire set.
  # allkeys-random -> Remove a random key, any key.
  # volatile-ttl -> Remove the key with the nearest expire time (minor TTL)
  # noeviction -> Don't evict anything, just return an error on write operations.
  #
  # LRU means Least Recently Used
  # LFU means Least Frequently Used
  #
  # Both LRU, LFU and volatile-ttl are implemented using approximated
  # randomized algorithms.
  #
  # Note: with any of the above policies, Redis will return an error on write
  #       operations, when there are no suitable keys for eviction.
  #
  #       At the date of writing these commands are: set setnx setex append
  #       incr decr rpush lpush rpushx lpushx linsert lset rpoplpush sadd
  #       sinter sinterstore sunion sunionstore sdiff sdiffstore zadd zincrby
  #       zunionstore zinterstore hset hsetnx hmset hincrby incrby decrby
  #       getset mset msetnx exec sort
  #
  # The default is:
  #
  # maxmemory-policy noeviction
  ```

- allkeys

  - allkeys-lru：从数据集（server.db[i].dict ）中挑选最少使用的数据淘汰
    - 适合场景：reids应用中缓存的都是相对热点数据
  - allkeys-random：随机移除某个key
    - 适合场景：运用中对于缓存key的访问概率大致相等

- volatile

  - volatile-random：从已设置过期时间中的数据集（server.db[i].expires ）中任意选择数据淘汰
  - valatile-lru：从已设置过期时间中的数据集（server.db[i].expires ）中挑选最近使用最少的数据淘汰
  - valatile-ttl：从已设置过期时间中的数据集（server.db[i].expires ）中挑选将要过期的数据淘汰
  - 适合场景：这种策略是的我们可以向redis提示哪些key更适合淘汰，可以自己控制

总结：redis实现的lru并不是可靠的lru，因为设计时需权衡：

- 如果在所有数据中搜索数据，redis是单线程，所以耗时的操作会更谨慎



## 单进程单线程Redis

​	redis是单进程单线程的，即是单线程处理所有客户端的请求。官方解释CPU并不是redis瓶颈所在，而是机器的内存和网络的带宽。redis是通过IO多路复用来处理高并发请求的。



## Redis与Lua脚本

### redis使用中面临的问题

- 原子性：redis虽是单线程的，但也会有多线程问题，这问题来自多个客户端的请求处理。若多个客户端对同一数据进行操作，可能会引发多线程问题，即不能满足原子性。
- 效率问题：当我们使用redis时，可能处理一个业务会多次请求redis获得数据，这是对redis的网路吞吐会造成一定压力。redis提供了pipeline管道用于接受处理多条请求，但其有一定局限，就是执行的多条命令之间是没有依赖关系的。

### LUA

​	redis中内嵌了对lua环境的支持，通过在客户端使用lua脚本，可直接在服务端原子的执行多条redis命令

​	使用脚本的好处：

- 多个命令放入一个脚本中，减少网络开销
- 原子性
- 复用性：多个客户端可共享

客户端调用lua脚本：

- eval script num args

  - eval "redis.call('set','msg','hello')" 0
  - eval "return redis.call('get','msg')" 0
  - eval "return redis.call('set',KEYS[1],ARGV[1])" 1 lua1 hello（KEYS、ARGV必须大写）

- evalsha：当lua脚本较长时，可先根据lua脚本内容生成SHA1摘要，保存在脚本缓存中，然后通过evalsha调用

  - script load "script"：生成摘要
  - evalsha sha：根据摘要执行lua脚本

  ```
  127.0.0.1:6379> script load "return redis.call('get','lua1')"
  "a5a402e90df3eaeca2ff03d56d99982e05cf6574"
  127.0.0.1:6379> evalsha "a5a402e90df3eaeca2ff03d56d99982e05cf6574" 0
  "hello"
  127.0.0.1:6379> 
  
  ```



