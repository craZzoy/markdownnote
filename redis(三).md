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

