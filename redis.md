# redis中的数据结构

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

  ```redis
  SETEX mykey 10 "Hello"
  ```

  - 等同于：

    ```redis
    SET mykey value
    EXPIRE mykey seconds
    ```

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

- 积极(active)方式：周期性地从设置了失效时间的主键中选择一部分失效的主键删除。每秒进行10次操作，具体流程是

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



# 分布式redis

## 集群

为解决单点故障，可使用集群。

## 主从复制

​	主从复制就是我们常说的master-slave模式，主数据库可进行读写操作，并把数据同步到slave节点。在一般情况下，从数据库是只读的，并接受master节点同步过来的数据。一个master可以有多个slave

![1568464491877](D:/BaiduNetdiskDownload/markdown笔记/redis(二).assets/1568464491877.png)

### 配置

```txt
准备两台服务器，分别安装redis ， server1 server2,master节点可不用配置
\1. 在server2的redis.conf文件中增加 slaveof server1-ip 6379 、 同时将bindip注释掉，允许所
有ip访问
\2. 启动server2
\3. 访问server2的redis客户端，输入 INFO replication
\4. 通过在master机器上输入命令，比如set foo bar 、 在slave服务器就能看到该值已经同步过来了
```



master：

```
port 6380
#bind 127.0.0.1
```

slave1：

```properties
port 6381
#bind 127.0.0.1
slaveof 127.0.0.1 6380
```

slave2:

```properties
port 6382
#bind 127.0.0.1
slaveof 127.0.0.1 6380
```



### 原理

#### 全量复制

​	redis全量复制一般发生在slave初始化阶段，此时slave会同步master的全部数据，具体步骤如下：

![1568465474929](D:/BaiduNetdiskDownload/markdown笔记/redis(二).assets/1568465474929.png)

1. slave连接master，并发送sync
2. master执行bgsave生成快照，并记录此期间的写命令，发送快照给slave
3. slave根据快照载入数据
4. master发送缓存的写命令

​	完成上述步骤后就完成了slave数据的初始化，此时slave可以接受来自客户端的请求。

​	master/slave的复制策略采用的是乐观的，也就是不是强一致性，允许最终一致性。master的数据是异步同步到slave，这样能保证master能够正常处理客户端的请求。

​	为确保master的同步到slave，redis配置文件中提供了以下参数保证：

- min-slave-to-write 3：表示只有当3个或3个以上的slave连接到master，master才是可写的
- min-slave-max-lag 10：表示允许slave最长失去连接的时间，即10s内还没收到slave的响应，则master认为此slave已经失效

#### 增量复制

​	从redis2.8开始，就支持主从复制的断点续传，如果同步过程中网络中断了，那么slave将会接着上次的结果复制，而不是全量复制，其中原理是：

​	master node会在内存中创建一个backlog，master和slave都会保存一个replica offset还有一个master id，offset就是保存在backlog中的。如果master和slave网络连接断掉了，slave会让master从上次的replica offset开始继续复制。但是如果没有找到对应的offset，那么就会执行一次全量同步 

#### 无硬盘复制

​	依前面所诉，redis复制的原理是基于RDB方式的持久化实现的，也就是master发送硬盘中RDB快照到slave，slave根据这个快照同步数据，但这种方式存在问题：

1. 若master禁用RDB快照，执行主从复制时仍然会生成RDB文件，若下次重启redis根据RDB恢复数据，因为主从复制的时间点的数据，也就不知道具体保存的是哪个时间点的数据，这样数据可能会有问题。
2. 当硬盘性能低时，初始化过程会对性能造成影响

​	为解决上述问题，redis2.8.18版本引入了无磁盘复制功能，即RDB文件在master节点内存中生成并发送给slave，通过在conf文件中设置以下参数开启：

```properties
repl-diskless-sync yes
```



## 哨兵机制

​	当master节点故障时，需要在slave中选举一个节点作为master，而redis没有提供自动选举功能，而是需要借助一个哨兵来进行监控

### 什么是哨兵

​	哨兵的作用就是监控redis系统的运行情况，它的功能包括：

1. 监控master和slave是否正常运行
2. master出现故障时自动将slave数据库升级为master

​	哨兵是一个独立的进程，它的架构为：

![1568470362898](D:/BaiduNetdiskDownload/markdown笔记/redis(二).assets/1568470362898.png)

​	为了解决master选举问题引入了哨兵，此时哨兵也可能出现单点问题，这就需要对哨兵做集群操作。此时的哨兵不仅会监控master、slave节点，哨兵之间还会互相监控。这种方式叫哨兵集群，哨兵集群需要解决故障发现和master决策的协商问题

![1568470615527](D:/BaiduNetdiskDownload/markdown笔记/redis(二).assets/1568470615527.png)



### sentinel（哨兵）之间的互相感知

​	sentinel节点之间因为共同监视同一个master而产生关联，一个新加入的sentinel节点需要和其他监视相同master节点的sentinel相互感知，其机制如下图所示

![1568471049467](D:/BaiduNetdiskDownload/markdown笔记/redis(二).assets/1568471049467.png)

1. 需要互相感知的sentinel都向它们共同监视的master节点订阅channel:sentinel:hello
2. 新加入的sentinel节点向这个channel发布一条信息，包含自己本身的信息，这样订阅了这个channel的sentinel就可以发现这个新的sentinel
3. 新加入的sentinel与其他sentinel节点建立长连接



### master节点的故障发现

​	sentinel节点会定期向master节点发送心跳包来判断存活状态，一旦发现master节点不可用，sentinel会把master设为“主观不可用状态”，然后它会把“主观不可用状态”发送给其他sentinel节点确认，当确认的sentinel节点数大于quorum时，则会认为master节点是“客观不可用”，接着就进入新的master选举流程。

​	有一个问题，sentinel做了集群，如果多个节点同时发现master节点不可用，那这时谁去决策哪个redis节点充当master呢？这时需要从sentinel集群中选取一个节点充当leader来决策，而这里使用了一致性raft算法实现，类似paxos算法，都是分布式一致性算法，也是基于投票机制，只要保证过半数节点通过提议即可。

​	raft算法动画演示：http://thesecretlivesofdata.com/raft/



### 配置实现

#### 配置sentinel.conf

```properties
#哨兵启动端口
port 6040
#1为最低通过票数
sentinel monitor mymaster 127.0.0.1 6380 1
#表示5秒内mymaster没响应，就认为sdown
sentinel down-after-milliseconds mymaster 5000
#表示15s内master还没有活过来，则启动failover，从剩下的slave中选一个升级为master
sentinel failover-timeout mymaster 15000
```

#### 启动哨兵

- redis-sentinel sentinel.conf
- redis-server /path/to/sentinel.conf --sentinel

启动sentinel，关掉master，查看日志输出：

```properties
21306:X 14 Sep 2019 23:00:33.926 # WARNING: The TCP backlog setting of 511 cannot be enforced because /proc/sys/net/core/somaxconn is set to the lower value of 128.
21306:X 14 Sep 2019 23:00:33.926 # Sentinel ID is 7daabd30960e9c29a9758fe1a75e228d42588af3
21306:X 14 Sep 2019 23:00:33.926 # +monitor master mymaster 127.0.0.1 6380 quorum 1
#sdown（subjective） 主观认为master已关闭
21306:X 14 Sep 2019 23:01:43.455 # +sdown master mymaster 127.0.0.1 6380
#odown(objective) 客观认为master已关闭
21306:X 14 Sep 2019 23:01:43.455 # +odown master mymaster 127.0.0.1 6380 #quorum 1/1
21306:X 14 Sep 2019 23:01:43.455 # +new-epoch 1
#哨兵开始故障检查
21306:X 14 Sep 2019 23:01:43.455 # +try-failover master mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:43.457 # +vote-for-leader 7daabd30960e9c29a9758fe1a75e228d42588af3 1
21306:X 14 Sep 2019 23:01:43.457 # +elected-leader master mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:43.457 # +failover-state-select-slave master mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:43.534 # +selected-slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:43.534 * +failover-state-send-slaveof-noone slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:43.586 * +failover-state-wait-promotion slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:44.486 # +promoted-slave slave 127.0.0.1:6381 127.0.0.1 6381 @ mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:44.486 # +failover-state-reconf-slaves master mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:44.545 * +slave-reconf-sent slave 127.0.0.1:6382 127.0.0.1 6382 @ mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:45.517 * +slave-reconf-inprog slave 127.0.0.1:6382 127.0.0.1 6382 @ mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:45.517 * +slave-reconf-done slave 127.0.0.1:6382 127.0.0.1 6382 @ mymaster 127.0.0.1 6380
#哨兵完成故障恢复
21306:X 14 Sep 2019 23:01:45.593 # +failover-end master mymaster 127.0.0.1 6380
21306:X 14 Sep 2019 23:01:45.593 # +switch-master mymaster 127.0.0.1 6380 127.0.0.1 6381
#slave 列出了新的master和slave，可看到原来的master还在，因为其可能在某个时间点恢复，恢复后会以slave角色加入
21306:X 14 Sep 2019 23:01:45.593 * +slave slave 127.0.0.1:6382 127.0.0.1 6382 @ mymaster 127.0.0.1 6381
21306:X 14 Sep 2019 23:01:45.593 * +slave slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6381
21306:X 14 Sep 2019 23:01:50.619 # +sdown slave 127.0.0.1:6380 127.0.0.1 6380 @ mymaster 127.0.0.1 6381
```

此时6381节点已成为master：

```properties
127.0.0.1:6381> info replication
# Replication
role:master
connected_slaves:1
slave0:ip=127.0.0.1,port=6382,state=online,offset=18058,lag=1
master_replid:9edaa0fb5b6f03c25ee8f8c32a9190d5eba5b9f3
master_replid2:439910b5a5972acff7fda967ce22ab232a1d982e
master_repl_offset:18058
second_repl_offset:13318
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:1
repl_backlog_histlen:18058

```



## redis-cluster

​	即使使用哨兵，此时的redis集群的每个数据库依然存有集群中的所有数据，从而导致集群的总数据存储量受限于系统内存最小的节点，形成了木桶效应。

​	redis3.0之前通过在客户端做的分片，通过hash环的方式对key进行分片存储。这种方式虽能缓解各个节点的存储压力，但是维护成本高、增加移除节点叫繁琐。redis3.0后支持集群功能，集群的特点在于拥有和单机实例一样的性能，同时在网络分区以后能够提供一定的可访问性以及对主数据库故障恢复的支持。

​	哨兵和集群是两种不同的方式，当不需要对数据进行分片时使用哨兵就行，如果需要水平拓容，集群是一个计较好的方式。

### 拓扑结构和数据分区

#### 拓扑结构

![1568513279241](D:/BaiduNetdiskDownload/markdown笔记/redis(二).assets/1568513279241.png)

​	如图，在redis cluster结构中有以下特点

- 一个redis cluster由多个redis节点组成，每个节点（master-slave）对应一个分片，数据没有交集。
- 节点组内部分为主备两类节点，对应master和slave节点，两者数据准实时一致，通过异步化的机制进行复制
- 一个master可对应多个slave，master可读可写，slave只可读

​	redis-cluster是基于gossip协议实现的**无中心化节点**的集群，因为去中心化的架构不存在统一的配置中心，各个节点对整个集群状态的认知来自于节点之间的信息交互。在Redis Cluster，这个信息交互是通过Redis Cluster Bus来完成的 。

#### 数据分区	

​	分布式数据库分区时需解决将数据映射到不同节点的问题，即把数据划分到多个节点上，不同的节点负责不同的数据。而redis-cluster采用哈希分区规则，采用虚拟槽分区。

​	reids中，虚拟槽的范围是0~16383（2^14-1），槽是集群内数据管理和迁移的基本单位，每个节点负责一定数量的槽。计算公式：slot=CRC16(key)%16383。每一个节点负责维护一部分槽以及槽所映射的键值数据。

​	 

### HashTags

​	在单个redis服务器的情况下，数据的原子操作是可以保证的，如mset命令在这种情况下是原子性的，不可能出现一部分key更新而另一部分key未更新的情况（操作多个key）。但在分片的情况下，数据分布在哪个节点是不确定的，即mset操作的key可能分布在不同服务器中，可能会出现某些指定的keys更新了，而一部分未更新。

​	所以，我们需要考虑将key尽量分布在不同机器，又要考虑相关联的key尽量分布在同一个服务器，怎么解决？

​	在redis中引入了HashTags的概念，可以使得数据分布算法可以根据key的某一部分进行计算，然后让相关的key落到同一个数据分片，比如

- 若存在
  - user:user1:id、user:user1:name
- 通过hashTags表示，即只对{}中的内容做hash计算
  - user:{user1}:id、user:{user1}:name

​	这样一来就能控制相关的key落到同一个redis节点上。



### 重定向客户端

​	redis-cluster并不会代理查询，那么当客户端访问一个服务端不存在的key时会怎样呢？它可能会返回这样的结果：

```properties
-MOVED 254 127.0.0.1:6381
```

​	这表示根据key计算出来的槽式254，但这个槽分配到了`127.0.0.1:6381`这个redis服务端



### 分片迁移

​	在一个稳定的redis-cluster下，每一个slot对应的节点是确定的，但在某些情况下，节点和分片对应的关系会发生变更：

- 新加入master节点
- 某个节点宕机

此时需要将16384个槽做个再分配，这一过程现版本中需要人工介入

- 新增一个节点
  - 若新增一个节点D，redis-cluster的做法是从各自节点的前面拿一部分slot到D上，大致情况是：
    - 节点A覆盖1365-5460 
    - 节点B覆盖6827-10922
    - 节点C覆盖12288-16383
    - 节点D覆盖0-1364,5461-6826,10923-12287 
- 删除一个节点
  - 需要将删除节点上的数据迁移到其他节点下，才能执行删除



### 槽迁移的过程

​	槽迁移的过程有一个不稳定的状态，这样情况下服务端会有一些规则限制客户端的行为，以保证redis在不宕机的情况下也能执行迁移。下面表示迁移1、2、3槽的过程

![1568550489500](D:/BaiduNetdiskDownload/markdown笔记/redis(二).assets/1568550489500.png)

​	其中简单的工作流程

1. A向B发送状态变更命令，把B对应的slot状态设置为`IMPORTING`
   - 处于`IMPORTING`状态下的服务端B有以下限制
     - 对于B节点上的slot的所有非ASK跳转过来的操作，该服务端都不会进行处理，而是通过MOVED命令让客户端跳转到A节点执行，以保证数据的一致性（不这样处理，B中处理的命令后结果可能会和在A中处理的不一致）。
2. B向A发送状态变更命令，把A对应的slot状态设置为`MIGRATING`
   - 处于`MIGRATING`状态下的服务端A有以下限制
     - 如果客户端访问的key还没有迁移出去，则正常处理这个key
     - 如果客户端访问的key不存在或者已经迁移出去了，则返回ASK信息让它跳转到MasterB去执行



### 配置实现

#### redis.conf中相关参数介绍

- **cluster-enabled**：是否启用redis cluster support支持
- **cluster-config-file**：记录了集群的相关信息，用户不需配置
- **cluster-node-timeout**：redis集群节点最大不可用时长，不可用则进行故障处理，如主备切换
- **cluster-slave-validity-factor** **<factor>**：有效因子参数
  - 为0：不管主设备和从设备之间的链路断开多久，从设备都将尝试故障恢复从设备
  - 大于0：如有效因子为10，节点超时为5s，那么与主设备断开连接时长小于50s的从设备将尝试恢复故障，否则不会
- **cluster-migration-barrier <count>**：主设备将保持连接的最小从设备数
- **cluster-require-full-coverage <yes / no>**
  - yes：如果key的空间的某个百分比未被任何节点覆盖，则集群停止接受写入
  - no：集群接受写入



#### 步骤

1. 构建如下目录，redis-trib .rb可不需要

   ![1568595770744](D:/BaiduNetdiskDownload/markdown笔记/redis(二).assets/1568595770744.png)

2. 复制redis.conf、redis-server到每个目录，修改配置，启动各服务器

   ```properties
   cluster-enabled yes
   port ****
   ```

3. 执行构建集群命令，可看到构建成功，16384个槽位成功分配，7000、7001、7002为master节点，7003、7004、7005为slave节点

   ```properties
   [root@izwz9c61wsgboaq9aoxis9z redis-cluster]# redis-cli --cluster create 127.0.0.1:7000 127.0.0.1:7001 127.0.0.1:7002 127.0.0.1:7003 127.0.0.1:7004 127.0.0.1:7005 --cluster-replicas 1
   >>> Performing hash slots allocation on 6 nodes...
   Master[0] -> Slots 0 - 5460
   Master[1] -> Slots 5461 - 10922
   Master[2] -> Slots 10923 - 16383
   Adding replica 127.0.0.1:7004 to 127.0.0.1:7000
   Adding replica 127.0.0.1:7005 to 127.0.0.1:7001
   Adding replica 127.0.0.1:7003 to 127.0.0.1:7002
   >>> Trying to optimize slaves allocation for anti-affinity
   [WARNING] Some slaves are in the same host as their master
   M: fd50fc3fdb98ccad82c89a1438bfa562618ebcbf 127.0.0.1:7000
      slots:[0-5460] (5461 slots) master
   M: 9eb90ec303bf49b826036cde79781b5dd022a1e4 127.0.0.1:7001
      slots:[5461-10922] (5462 slots) master
   M: f37d4f1575b099c8b7cf9afa7d1ab0f0c4be4f8a 127.0.0.1:7002
      slots:[10923-16383] (5461 slots) master
   S: 4bbb7d5e5de2df43474472f612e2ff128e8880ec 127.0.0.1:7003
      replicates f37d4f1575b099c8b7cf9afa7d1ab0f0c4be4f8a
   S: 09f8474ebd21a699945533e2d22ff78b1fa62a06 127.0.0.1:7004
      replicates fd50fc3fdb98ccad82c89a1438bfa562618ebcbf
   S: 020093bc7b864a980a067947e1fa427731dc237e 127.0.0.1:7005
      replicates 9eb90ec303bf49b826036cde79781b5dd022a1e4
   Can I set the above configuration? (type 'yes' to accept): yes
   >>> Nodes configuration updated
   >>> Assign a different config epoch to each node
   >>> Sending CLUSTER MEET messages to join the cluster
   Waiting for the cluster to join
   ...
   >>> Performing Cluster Check (using node 127.0.0.1:7000)
   M: fd50fc3fdb98ccad82c89a1438bfa562618ebcbf 127.0.0.1:7000
      slots:[0-5460] (5461 slots) master
      1 additional replica(s)
   S: 4bbb7d5e5de2df43474472f612e2ff128e8880ec 127.0.0.1:7003
      slots: (0 slots) slave
      replicates f37d4f1575b099c8b7cf9afa7d1ab0f0c4be4f8a
   M: 9eb90ec303bf49b826036cde79781b5dd022a1e4 127.0.0.1:7001
      slots:[5461-10922] (5462 slots) master
      1 additional replica(s)
   S: 09f8474ebd21a699945533e2d22ff78b1fa62a06 127.0.0.1:7004
      slots: (0 slots) slave
      replicates fd50fc3fdb98ccad82c89a1438bfa562618ebcbf
   S: 020093bc7b864a980a067947e1fa427731dc237e 127.0.0.1:7005
      slots: (0 slots) slave
      replicates 9eb90ec303bf49b826036cde79781b5dd022a1e4
   M: f37d4f1575b099c8b7cf9afa7d1ab0f0c4be4f8a 127.0.0.1:7002
      slots:[10923-16383] (5461 slots) master
      1 additional replica(s)
   [OK] All nodes agree about slots configuration.
   >>> Check for open slots...
   >>> Check slots coverage...
   [OK] All 16384 slots covered.
   ```

4. 查看信息

   - info replication：查看主从信息

     ```properties
     127.0.0.1:7001> info replication
     # Replication
     role:master
     connected_slaves:1
     slave0:ip=127.0.0.1,port=7005,state=online,offset=2408,lag=1
     master_replid:1789c2a9c750b6eada5663f4df71b181f5f89eee
     master_replid2:0000000000000000000000000000000000000000
     master_repl_offset:2408
     second_repl_offset:-1
     repl_backlog_active:1
     repl_backlog_size:1048576
     repl_backlog_first_byte_offset:1
     repl_backlog_histlen:2408
     ```

   - cluster nodes，可以看出每个节点都有一个唯一的ID，此ID不可变，IP+端口组合可能会变，所有选择创建新的ID标识节点的唯一性

     ```properties
     9eb90ec303bf49b826036cde79781b5dd022a1e4 127.0.0.1:7001@17001 master - 0 1568597050098 2 connected 5461-10922
     fd50fc3fdb98ccad82c89a1438bfa562618ebcbf 127.0.0.1:7000@17000 master - 0 1568597051000 1 connected 0-5460
     4bbb7d5e5de2df43474472f612e2ff128e8880ec 127.0.0.1:7003@17003 slave f37d4f1575b099c8b7cf9afa7d1ab0f0c4be4f8a 0 1568597050000 4 connected
     09f8474ebd21a699945533e2d22ff78b1fa62a06 127.0.0.1:7004@17004 slave fd50fc3fdb98ccad82c89a1438bfa562618ebcbf 0 1568597052102 5 connected
     020093bc7b864a980a067947e1fa427731dc237e 127.0.0.1:7005@17005 myself,slave 9eb90ec303bf49b826036cde79781b5dd022a1e4 0 1568597050000 6 connected
     f37d4f1575b099c8b7cf9afa7d1ab0f0c4be4f8a 127.0.0.1:7002@17002 master - 0 1568597051100 3 connected 10923-16383
     
     ```

   - cluster info

     ```properties
     127.0.0.1:7005> cluster info 
     cluster_state:ok
     cluster_slots_assigned:16384
     cluster_slots_ok:16384
     cluster_slots_pfail:0
     cluster_slots_fail:0
     cluster_known_nodes:6
     cluster_size:3
     cluster_current_epoch:6
     cluster_my_epoch:2
     cluster_stats_messages_ping_sent:1955
     cluster_stats_messages_pong_sent:1961
     cluster_stats_messages_meet_sent:5
     cluster_stats_messages_sent:3921
     cluster_stats_messages_ping_received:1961
     cluster_stats_messages_pong_received:1960
     cluster_stats_messages_received:3921
     ```



### 使用创建集群脚本创建Redis集群

​	只需在Redis发行版中检查 utils/create-cluster 目录即可。 里面有一个名为create-cluster的脚本（与其包含的目录名称相同），它是一个简单的bash脚本。 要启动具有3个主站和3个从站的6个节点群集，只需输入以下命令：

1. create-cluster start
2. create-cluster create

​	当redis-trib实用程序希望您接受集群布局时，在步骤2中回复yes。您现在可以与群集交互，默认情况下，第一个节点将从端口30001开始。 完成后，停止群集：

- create-cluster stop

  更多查看README



### 测试故障转移

​	使某个master节点故障

```properties
[root@izwz9c61wsgboaq9aoxis9z 6380]# redis-cli -p 7002 debug segfault
Error: Server closed the connection
```

​	现在我们可以看一致性测试的输出以查看它报告的内容

```properties
18849 R (0 err) | 18849 W (0 err) |
23151 R (0 err) | 23151 W (0 err) |
27302 R (0 err) | 27302 W (0 err) |
... many error warnings here ...

29659 R (578 err) | 29660 W (577 err) |
33749 R (578 err) | 33750 W (577 err) |
37918 R (578 err) | 37919 W (577 err) |
42077 R (578 err) | 42078 W (577 err) |
```

​	正如您在故障转移期间所看到的，系统无法接受578次读取和577次写入，但是在数据库中未创建任何不一致。 这听起来可能会出乎意料，因为在本教程的第一部分中，我们声明Redis群集在故障转移期间可能会丢失写入，因为它使用异步复制。 我们没有说的是，这种情况不太可能发生，因为Redis会将答复发送给客户端，并将命令复制到从服务器，同时，因此会有一个非常小的窗口来丢失数据。 但是很难触发这一事实并不意味着这是不可能的，所以这不会改变Redis集群提供的一致性保证。

​	查看节点信息，可看出原来7002的从服务器已经成为主服务器

```properties
[root@izwz9c61wsgboaq9aoxis9z 6380]# redis-cli -p 7000 cluster nodes
fd50fc3fdb98ccad82c89a1438bfa562618ebcbf 127.0.0.1:7000@17000 myself,master - 0 1568598317000 1 connected 0-5460
4bbb7d5e5de2df43474472f612e2ff128e8880ec 127.0.0.1:7003@17003 master - 0 1568598316000 7 connected 10923-16383
9eb90ec303bf49b826036cde79781b5dd022a1e4 127.0.0.1:7001@17001 master - 0 1568598318000 2 connected 5461-10922
09f8474ebd21a699945533e2d22ff78b1fa62a06 127.0.0.1:7004@17004 slave fd50fc3fdb98ccad82c89a1438bfa562618ebcbf 0 1568598317000 5 connected
020093bc7b864a980a067947e1fa427731dc237e 127.0.0.1:7005@17005 slave 9eb90ec303bf49b826036cde79781b5dd022a1e4 0 1568598318117 6 connected
f37d4f1575b099c8b7cf9afa7d1ab0f0c4be4f8a 127.0.0.1:7002@17002 master,fail - 1568598203667 1568598200000 3 disconnected

```

​	重启故障服务器7002，查看信息，原本是master的7002已经成为7003的从节点

```properties
[root@izwz9c61wsgboaq9aoxis9z 7002]# redis-cli -p 7002 cluster nodes
09f8474ebd21a699945533e2d22ff78b1fa62a06 127.0.0.1:7004@17004 slave fd50fc3fdb98ccad82c89a1438bfa562618ebcbf 0 1568598776127 5 connected
4bbb7d5e5de2df43474472f612e2ff128e8880ec 127.0.0.1:7003@17003 master - 0 1568598773000 7 connected 10923-16383
020093bc7b864a980a067947e1fa427731dc237e 127.0.0.1:7005@17005 slave 9eb90ec303bf49b826036cde79781b5dd022a1e4 0 1568598774124 6 connected
f37d4f1575b099c8b7cf9afa7d1ab0f0c4be4f8a 127.0.0.1:7002@17002 myself,slave 4bbb7d5e5de2df43474472f612e2ff128e8880ec 0 1568598774000 3 connected
9eb90ec303bf49b826036cde79781b5dd022a1e4 127.0.0.1:7001@17001 master - 0 1568598775000 2 connected 5461-10922
fd50fc3fdb98ccad82c89a1438bfa562618ebcbf 127.0.0.1:7000@17000 master - 0 1568598775125 1 connected 0-5460
[root@izwz9c61wsgboaq9aoxis9z 7002]# redis-cli -p 7002 info replication
# Replication
role:slave
master_host:127.0.0.1
master_port:7003
master_link_status:up
master_last_io_seconds_ago:7
master_sync_in_progress:0
slave_repl_offset:4466
slave_priority:100
slave_read_only:1
connected_slaves:0
master_replid:ff9e2b97e192dd188d57a92bd0004df305212d63
master_replid2:0000000000000000000000000000000000000000
master_repl_offset:4466
second_repl_offset:-1
repl_backlog_active:1
repl_backlog_size:1048576
repl_backlog_first_byte_offset:4271
repl_backlog_histlen:196
```



### 手动故障转移

​	在一个从服务器上执行cluster failover执行故障转移，手动转移比实际主控导致的故障转移更安全，因为只有新服务器主节点处理完源主服务器传过来的数据流时才进行主设备切换 

​	如下，将7005切换为主设备，原来的master节点7001已经成为它的slave

```properties
[root@izwz9c61wsgboaq9aoxis9z 7002]# redis-cli -p 7005 cluster failover
OK
[root@izwz9c61wsgboaq9aoxis9z 7002]# redis-cli -p 7002 cluster nodes
09f8474ebd21a699945533e2d22ff78b1fa62a06 127.0.0.1:7004@17004 slave fd50fc3fdb98ccad82c89a1438bfa562618ebcbf 0 1568599647134 5 connected
4bbb7d5e5de2df43474472f612e2ff128e8880ec 127.0.0.1:7003@17003 master - 0 1568599649138 7 connected 10923-16383
020093bc7b864a980a067947e1fa427731dc237e 127.0.0.1:7005@17005 master - 0 1568599646000 8 connected 5461-10922
f37d4f1575b099c8b7cf9afa7d1ab0f0c4be4f8a 127.0.0.1:7002@17002 myself,slave 4bbb7d5e5de2df43474472f612e2ff128e8880ec 0 1568599648000 3 connected
9eb90ec303bf49b826036cde79781b5dd022a1e4 127.0.0.1:7001@17001 slave 020093bc7b864a980a067947e1fa427731dc237e 0 1568599646132 8 connected
fd50fc3fdb98ccad82c89a1438bfa562618ebcbf 127.0.0.1:7000@17000 master - 0 1568599648136 1 connected 0-5460
```



- codis
- twenproxy
- redis-cluster

# 



## 问题

1. 哨兵模式下，客户端应该连接到哪个redis-server

2. 集群模式中，为什么会有moved的error

   客户端情况下应该不会出现这种情况，因为初始化时已经处理了slot对应哪个节点。但若客户端在服务端重新分配槽之前已经初始化，可能会出现这种情况



# redis应用实战

## reids已有的客户端支持

- Jedis
  - redis的java实现客户端，其API提供了比较全面的Redis命令的支持
- Redisson
  - Redisson实现了分布式和可拓展的java数据结构，和jedis相比，功能较为简单，不支持字符串操作，不支持排序、事务、管道、分区等reids特性。redisson主要是促进使用者对redis的关注分离，从而让使用者能够将精力集中地放在处理业务逻辑上。
- Lettuce
  - lettuce是基于netty构建的一种可伸缩的线程安全的redis客户端，支持同步、异步、响应式模式。多个线程可以共享一个实例，而不必担心多线程并发问题



## API调用实例

### JedisAPI调用

#### 单redis实例调用

```java
package com.gupao;

import redis.clients.jedis.Client;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisSentinelPool;

import java.util.HashSet;
import java.util.Set;

public class JedisClientDemo {

    public static void main(String[] args) {

        //通过jedis连接
        Jedis jedis = new Jedis("119.23.201.241",6379);
        System.out.println(jedis.get("msg"));;
        //System.out.println();

        //通过jedisPool连接单个实例
        JedisPool jedisPool = new JedisPool("119.23.201.241",6379);
        Set<String> keys = jedisPool.getResource().keys("*");
        keys.stream().forEach(System.out::println);
        System.out.println(jedisPool.getResource().get("msg"));

    }
}

```



#### sentinel模式调用

```java
package com.gupao;

import redis.clients.jedis.Client;
import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisSentinelPool;

import java.util.HashSet;
import java.util.Set;

public class JedisClientDemo {

    public static void main(String[] args) {
        //sentinel模式调用
        String masterName = "mymaster";
        Set<String> stringSet = new HashSet<>();
        stringSet.add("119.23.201.241:6400");
        JedisSentinelPool jedisSentinelPool = new JedisSentinelPool(masterName,stringSet);
        System.out.println(jedisSentinelPool.getResource().get("name"));
    }
}
```

原理：

​	客户端通过连接到哨兵集群，通过发送Protocol.SENTINEL_GET_MASTER_ADDR_BY_NAME 获取到master节点的host、port信息，然后再在客户端获得master节点的连接。连接之后需要建立监听机制（监听sentinel节点），当master重新选举后，客户端需要重新连接到新的master节点。

​	获得master节点信息源码：

```java
    private HostAndPort initSentinels(Set<String> sentinels, String masterName) {
        HostAndPort master = null;
        boolean sentinelAvailable = false;
        this.log.info("Trying to find master from available Sentinels...");
        Iterator var5 = sentinels.iterator();

        String sentinel;
        HostAndPort hap;
        //遍历sentinel节点，获取master节点信息，拿到就break
        while(var5.hasNext()) {
            sentinel = (String)var5.next();
            hap = HostAndPort.parseString(sentinel);
            this.log.debug("Connecting to Sentinel {}", hap);
            Jedis jedis = null;

            try {
                jedis = new Jedis(hap);
                //通过Protocol.SENTINEL_GET_MASTER_ADDR_BY_NAME获得master节点信息
                //{host,port}形式
                List<String> masterAddr = jedis.sentinelGetMasterAddrByName(masterName);
                sentinelAvailable = true;
                if(masterAddr != null && masterAddr.size() == 2) {
                    master = this.toHostAndPort(masterAddr);
                    this.log.debug("Found Redis master at {}", master);
                    break;
                }

                this.log.warn("Can not get master addr, master name: {}. Sentinel: {}", masterName, hap);
            } catch (JedisException var13) {
                this.log.warn("Cannot get master address from sentinel running @ {}. Reason: {}. Trying next one.", hap, var13.toString());
            } finally {
                if(jedis != null) {
                    jedis.close();
                }

            }
        }

        if(master == null) {
            if(sentinelAvailable) {
                throw new JedisException("Can connect to sentinel, but " + masterName + " seems to be not monitored...");
            } else {
                throw new JedisConnectionException("All sentinels down, cannot determine where is " + masterName + " master is running...");
            }
        } else {
            this.log.info("Redis master running at " + master + ", starting Sentinel listeners...");
            var5 = sentinels.iterator();
			//监听每个sentinel，以发现故障修复重新获得master
            while(var5.hasNext()) {
                sentinel = (String)var5.next();
                hap = HostAndPort.parseString(sentinel);
                JedisSentinelPool.MasterListener masterListener = new JedisSentinelPool.MasterListener(masterName, hap.getHost(), hap.getPort());
                masterListener.setDaemon(true);
                this.masterListeners.add(masterListener);
                masterListener.start();
            }

            return master;
        }
    }
```



#### cluster模式调用

```java
package com.gupao;

import redis.clients.jedis.*;

import java.util.HashSet;
import java.util.Set;

public class JedisClientDemo {

    public static void main(String[] args) {

        //cluster模式调用
        Set<HostAndPort> hostAndPorts = new HashSet<>();
        hostAndPorts.add(new HostAndPort("119.23.201.241",7000));
        hostAndPorts.add(new HostAndPort("119.23.201.241",7001));
        hostAndPorts.add(new HostAndPort("119.23.201.241",7002));
        hostAndPorts.add(new HostAndPort("119.23.201.241",7003));
        hostAndPorts.add(new HostAndPort("119.23.201.241",7004));
        hostAndPorts.add(new HostAndPort("119.23.201.241",7005));
        JedisCluster jedisCluster = new JedisCluster(hostAndPorts,6000);
        jedisCluster.set("name","tom");
        System.out.println(jedisCluster.get("name"));

    }
}

```

原理

程序启动初始化集群环境

1. 读取配置文件中的节点配置，无论是主从，无论多少个，只拿第一个，获取redis连接实例
2. 用获取的redis连接实例执行clusterNodes()方法，实际执行redis服务端cluster nodes命令，获取主从配置信
   息
3. 解析主从配置信息，先把所有节点存放到nodes的map集合中，key为节点的ip:port，value为当前节点的
   jedisPool
4. 解析主节点分配的slots区间段，把slot对应的索引值作为key，第三步中拿到的jedisPool作为value，存储在
   slots的map集合中就实现了slot槽索引值与jedisPool的映射，这个jedisPool包含了master的节点信息，所以槽和几点是对应的，与redis服务端一致



从集群环境存取值

1. 把key作为参数，执行CRC16算法，获取key对应的slot值
2. 通过该slot值，去slots的map集合中获取jedisPool实例 
3. 通过jedisPool实例获取jedis实例，最终完成redis数据存取工作 



### Redisson API调用

```java
Config config=new Config();
config.useClusterServers().setScanInterval(2000).
addNodeAddress("redis://192.168.11.153:7000",
"redis://192.168.11.153:7001",
"redis://192.168.11.154:7003","redis://192.168.11.157:7006");
RedissonClient redissonClient= Redisson.create(config);
RBucket<String> rBucket=redissonClient.getBucket("mic");
System.out.println(rBucket.get());
```

#### 常规操作命令

```properties
getBucket-> 获取字符串对象；
getMap -> 获取map对象
getSortedSet->获取有序集合
getSet -> 获取集合
getList ->获取列表
```



## redis实现分布式锁

​	在传统的java编程中的锁一般都可以通过进程获得，因为这些数据在进程中是可以控制为共享的，但是再分布式结构中，进程之间是隔离的，这时需要有个地方存储共享数据并且能够保证原子操作，于是redis正好可以用于实现。

#### 通过jedis API自定义实现

```java
package com.gupao;


import redis.clients.jedis.Jedis;
import redis.clients.jedis.JedisPool;
import redis.clients.jedis.JedisPoolConfig;

/**
 * jetis连接工具类
 */
public class JedisConnectionUtils {
    private static JedisPool jedisPool = null;
    static {
        JedisPoolConfig jedisPoolConfig = new JedisPoolConfig();
        jedisPoolConfig.setMaxTotal(50);
        jedisPool = new JedisPool(jedisPoolConfig,"119.23.201.241");
    }

    public static Jedis getJedis(){
        return jedisPool.getResource();
    }
}

```

```java
package com.gupao;

import redis.clients.jedis.Jedis;
import redis.clients.jedis.Transaction;

import java.util.UUID;

/**
 * 通过redis实现分布式锁
 */
public class DistributeLock {

    /**
     * 尝试获得锁
     * @param lockName 锁的名称
    * @param acquireTimeout 获得锁的超时时间
     * @param lockTimeout 锁本身的过期时间
     * @return
     */
    public String acquireLock(String lockName,long acquireTimeout,long lockTimeout){
        String identifier = UUID.randomUUID().toString();
        String lockKey = "lock"+lockName;
        int lockExpire = (int)(lockTimeout/1000);
        Jedis jedis= null;
        try {
            jedis = JedisConnectionUtils.getJedis();
            long end = System.currentTimeMillis() + acquireTimeout;
            while (System.currentTimeMillis() < end){
                if(jedis.setnx(lockKey,identifier) == 1){
                    jedis.expire(lockKey,lockExpire);
                    return identifier;
                }

                if(jedis.ttl(lockKey) == -1){
                    jedis.expire(lockKey,lockExpire);
                }

                try {
                    //等待片刻后再进行重试获得锁
                    Thread.sleep(100);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        } finally {
            jedis.close();
        }
        return null;
    }

    /**
     * 释放锁
     * @param lockName  锁名称
     * @param identifier 标识
     * @return
     */
    public boolean releaseLock(String lockName,String identifier){
        System.out.println("开始释放锁："+lockName+"="+identifier);
        String lockKey = "lock"+lockName;
        boolean isRelease = false;

        Jedis jedis = null;
        try{
            jedis = JedisConnectionUtils.getJedis();
            while(true){
                //watch 监视一个key，如果事务执行之前这个key被其他命令修改，则事务被打断 总是返回OK
                jedis.watch(lockKey);
                if(identifier.equals(jedis.get(lockKey))){
                    //muti标记事务的开始 总是返回OK
                    //exec 原子性的执行事务
                    Transaction transaction = jedis.multi();
                    transaction.del(lockKey);
                    if(transaction.exec().isEmpty()){
                        continue;
                    }

                    isRelease = true;
                }

                //异常
                jedis.unwatch();
                break;
            }
        } finally {
            jedis.close();
        }

        return isRelease;
    }

    /**
     * 通过lua表达式释放锁
     * @param lockName
     * @param identifier
     * @return
     */
    public boolean releaseLockWithLua(String lockName,String identifier){
        System.out.println("开始释放锁："+identifier);
        String lockKey = "lock"+lockName;
        Jedis jedis = JedisConnectionUtils.getJedis();
        String lua = "if redis.call(\"get\",KEYS[1]) == ARGV[1] then"
                +" return redis.call(\"del\",KEYS[1]) else return 0 end";
        Long rs = (Long) jedis.eval(lua,1,new String[]{lockKey,identifier});
        if(rs > 0){
            return true;
        }

        return false;
    }
}

```

```java
package com.gupao;

import com.sun.deploy.util.StringUtils;

/**
 * 单元测试类
 */
public class UnitTest {
    public static void main(String[] args) {
        Runnable runnable = UnitTest::run;
        /*Runnable runnable = ()->{
            System.out.println();
        };*/

        for(int i = 0; i< 10; i++){
            Thread thread = new Thread(runnable);
            thread.start();
        }
    }

    public static void run(){
        while(true){
            DistributeLock distributeLock = new DistributeLock();
            String identifier = distributeLock.acquireLock("updateOrder",2000,5000);
            if(identifier != null){
                System.out.println(Thread.currentThread()+"成功获得锁"+identifier);
                try {
                    Thread.sleep(1000);
                    //distributeLock.releaseLock("updateOrder",identifier);
                    distributeLock.releaseLockWithLua("updateOrder",identifier);
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
                break;
            }
        }

    }
}

```



#### redisson实现分布式锁

​	redisson除了使用redis常规的操作功能之外，还基于redis封装了一些功能，如：

- 分布式锁
- 原子操作
- 布隆过滤器
- 队列

分布式锁示例

```java
package com.gupao;


import org.redisson.Redisson;
import org.redisson.api.RLock;
import org.redisson.api.RedissonClient;
import org.redisson.config.Config;

import java.util.concurrent.TimeUnit;

public class RedissonClientDemo {

    public static void main(String[] args) throws InterruptedException {
        Config config = new Config();
        config.useSingleServer().setAddress("redis://119.23.201.241:6379");
        RedissonClient client = Redisson.create(config);
        RLock lock = client.getLock("updateOrder");
        //最多等待100s、上锁10s后自动解锁
        if(lock.tryLock(100,10, TimeUnit.SECONDS)){
            System.out.println("获得锁成功");
        }
        lock.unlock();
        System.out.println("释放锁成功");
    }

}

```

原理分析

tryLock

```java
public boolean tryLock(long waitTime, long leaseTime, TimeUnit unit) throws InterruptedException {
        long time = unit.toMillis(waitTime);
        long current = System.currentTimeMillis();
        long threadId = Thread.currentThread().getId();
    	//申请锁，返回锁的过期时间
        Long ttl = this.tryAcquire(leaseTime, unit, threadId);
        if(ttl == null) {
            return true;
        } else {
            time -= System.currentTimeMillis() - current;
            if(time <= 0L) {
                this.acquireFailed(threadId);
                return false;
            } else {
```

tryAcquire:

```java
private <T> RFuture<Long> tryAcquireAsync(long leaseTime, TimeUnit unit, long threadId) {
        //
    	if(leaseTime != -1L) {
            return this.tryLockInnerAsync(leaseTime, unit, threadId, RedisCommands.EVAL_LONG);
        } else {
            RFuture<Long> ttlRemainingFuture = this.tryLockInnerAsync(this.commandExecutor.getConnectionManager().getCfg().getLockWatchdogTimeout(), TimeUnit.MILLISECONDS, threadId, RedisCommands.EVAL_LONG);
            ttlRemainingFuture.onComplete((ttlRemaining, e) -> {
                if(e == null) {
                    if(ttlRemaining == null) {
                        this.scheduleExpirationRenewal(threadId);
                    }

                }
            });
            return ttlRemainingFuture;
        }
    }
```



### 多主机redis安全的分布式锁-redlock