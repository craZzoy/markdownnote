# 分布式redis

## 集群

为解决单点故障，可使用集群。

## 主从复制

​	主从复制就是我们常说的master-slave模式，主数据库可进行读写操作，并把数据同步到slave节点。在一般情况下，从数据库是只读的，并接受master节点同步过来的数据。一个master可以有多个slave

![1568464491877](D:\BaiduNetdiskDownload\markdown笔记\redis(二).assets\1568464491877.png)

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

![1568465474929](D:\BaiduNetdiskDownload\markdown笔记\redis(二).assets\1568465474929.png)

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

![1568470362898](D:\BaiduNetdiskDownload\markdown笔记\redis(二).assets\1568470362898.png)

​	为了解决master选举问题引入了哨兵，此时哨兵也可能出现单点问题，这就需要对哨兵做集群操作。此时的哨兵不仅会监控master、slave节点，哨兵之间还会互相监控。这种方式叫哨兵集群，哨兵集群需要解决故障发现和master决策的协商问题

![1568470615527](D:\BaiduNetdiskDownload\markdown笔记\redis(二).assets\1568470615527.png)



### sentinel（哨兵）之间的互相感知

​	sentinel节点之间因为共同监视同一个master而产生关联，一个新加入的sentinel节点需要和其他监视相同master节点的sentinel相互感知，其机制如下图所示

![1568471049467](D:\BaiduNetdiskDownload\markdown笔记\redis(二).assets\1568471049467.png)

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

![1568513279241](D:\BaiduNetdiskDownload\markdown笔记\redis(二).assets\1568513279241.png)

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

​	在redis中引入了HashTags的概念

- codis
- twenproxy
- redis-cluster
- 

## 问题

1. 哨兵模式下，客户端应该连接到哪个redis-server
2. 集群模式中，为什么会有moved的error



# redis应用实战