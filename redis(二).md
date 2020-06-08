# 分布式redis

## 集群

为解决单点故障，可使用集群。

## 主从复制

​	主从复制就是我们常说的master-slave模式，主数据库可进行读写操作，并把数据同步到slave节点。在一般情况下，从数据库是只读的，并接受master节点同步过来的数据。一个master可以有多个slave

![1568464491877](redis(二).assets\1568464491877.png)

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

![1568465474929](redis(二).assets\1568465474929.png)

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

![1568470362898](redis(二).assets\1568470362898.png)

​	为了解决master选举问题引入了哨兵，此时哨兵也可能出现单点问题，这就需要对哨兵做集群操作。此时的哨兵不仅会监控master、slave节点，哨兵之间还会互相监控。这种方式叫哨兵集群，哨兵集群需要解决故障发现和master决策的协商问题

![1568470615527](redis(二).assets\1568470615527.png)



### sentinel（哨兵）之间的互相感知

​	sentinel节点之间因为共同监视同一个master而产生关联，一个新加入的sentinel节点需要和其他监视相同master节点的sentinel相互感知，其机制如下图所示

![1568471049467](redis(二).assets\1568471049467.png)

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

![1568513279241](redis(二).assets\1568513279241.png)

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

![1568550489500](redis(二).assets\1568550489500.png)

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

   ![1568595770744](redis(二).assets\1568595770744.png)

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