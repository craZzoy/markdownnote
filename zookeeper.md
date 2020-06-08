# zookeeper入门

zookeeper是一个分布式协调服务中间件



## 安装

### 首先需在安装jdk

1. 下载tar包，解压：tar -xvf 

2. 配置环境变量：vim /ect/profile

   ```shell
   export JAVA_HOME=/usr/sofeware/jdk-13 
   export JRE_HOME=${JAVA_HOME}/jre  
   export CLASSPATH=.:${JAVA_HOME}/lib:${JRE_HOME}/lib  
   export  PATH=${JAVA_HOME}/bin:$PATH
   ```

   jdk13中默认没有jre，进入JAVA_HOME目录，执行：

   ```properties
   bin\jlink.sh --module-path jmods --add-modules java.desktop --output jre
   ```

3. 生效环境变量：source /ect/profile

4. 查看有没安装成功：java -version



### zookeeper安装

1. 官网下载安装包，解压：tar -zxvf zookeeper-3.4.10.tar.gz 

2. 在zookeeper的conf目录中复制zoo_sample.cfg到zoo.cfg，zoo.cfg为zookeeper默认的配置文件。

3. 修改zoo.cfg配置文件

   ```properties
   
   # The number of milliseconds of each tick
   tickTime=2000
   # The number of ticks that the initial 
   # synchronization phase can take
   initLimit=10
   # The number of ticks that can pass between 
   # sending a request and getting an acknowledgement
   syncLimit=5
   # the directory where the snapshot is stored.
   # do not use /tmp for storage, /tmp here is just 
   # example sakes.
   dataDir=/var/lib/zookeeper
   # the port at which the clients will connect
   clientPort=2181
   # the maximum number of client connections.
   # increase this if you need to handle more clients
   #maxClientCnxns=60
   #
   # Be sure to read the maintenance section of the 
   # administrator guide before turning on autopurge.
   #
   # http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
   #
   # The number of snapshots to retain in dataDir
   #autopurge.snapRetainCount=3
   # Purge task interval in hours
   # Set to "0" to disable auto purge feature
   #autopurge.purgeInterval=1                         
   ```

   参数说明：

   - tickTime=2000：基本事件单位，以毫秒为单位，用来控制心跳和超时，默认情况下最小的会话超时时间为两倍的 tickTime。
   - initLimit=10：初始化最大心跳次数（时间为10*2000ms）限制。*
   - syncLimit=5：表示leader和follower之间同步信息允许最大心跳次数（时间为5*2000ms）。
   - dataDir=/home/kkxmoye/local/zookeeper/data：数据存储路径。

4. 启动zkServer

   ```properties
   bin/zkServer.sh start
   ```

5. 客户端启动

   ```properties
   $ bin/zkCli.sh -server 127.0.0.1:2181
   ```

6. 简单命令

   - help：命令提示
   - ls
   - create 
   - get
   - set
   - delete

   ```properties
   [zk: 127.0.0.1:2181(CONNECTED) 6] help
   ZooKeeper -server host:port cmd args
   	addauth scheme auth
   	close 
   	config [-c] [-w] [-s]
   	connect host:port
   	create [-s] [-e] [-c] [-t ttl] path [data] [acl]
   	delete [-v version] path
   	deleteall path
   	delquota [-n|-b] path
   	get [-s] [-w] path
   	getAcl [-s] path
   	history 
   	listquota path
   	ls [-s] [-w] [-R] path
   	ls2 path [watch]
   	printwatches on|off
   	quit 
   	reconfig [-s] [-v version] [[-file path] | [-members serverID=host:port1:port2;port3[,...]*]] | [-add serverId=host:port1:port2;port3[,...]]* [-remove serverId[,...]*]
   	redo cmdno
   	removewatches path [-c|-d|-a] [-l]
   	rmr path
   	set [-s] [-v version] path data
   	setAcl [-s] [-v version] [-R] path acl
   	setquota -n|-b val path
   	stat [-w] path
   	sync path
   Command not found: Command not found help
   [zk: 127.0.0.1:2181(CONNECTED) 7] ls /
   [zookeeper]
   #创建一个节点my_data与zk_test关联
   [zk: 127.0.0.1:2181(CONNECTED) 8] create /zk_test my_data
   Created /zk_test
   [zk: 127.0.0.1:2181(CONNECTED) 9] ls
   ls [-s] [-w] [-R] path
   [zk: 127.0.0.1:2181(CONNECTED) 10] ls /
   [zk_test, zookeeper]
   #获取节点
   [zk: 127.0.0.1:2181(CONNECTED) 12] get zk_test
   Path must start with / character
   [zk: 127.0.0.1:2181(CONNECTED) 13] get /zk_test
   my_data
   [zk: 127.0.0.1:2181(CONNECTED) 14] get /zookeeper
   
   #更新节点
   [zk: 127.0.0.1:2181(CONNECTED) 15] set /zk_test junk
   [zk: 127.0.0.1:2181(CONNECTED) 16] get /zk_test
   junk
   [zk: 127.0.0.1:2181(CONNECTED) 17] 
   #删除节点
   [zk: 127.0.0.1:2181(CONNECTED) 18] delete /zk_test
   [zk: 127.0.0.1:2181(CONNECTED) 19] ls /
   [zookeeper]
   [zk: 127.0.0.1:2181(CONNECTED) 20] 
   
   ```

   



## zookeeper集群配置

1. 官网下载安装包，解压：tar -zxvf zookeeper-3.4.10.tar.gz 
2. 在zookeeper的conf目录中复制zoo_sample.cfg到zoo.cfg，zoo.cfg为zookeeper默认的配置文件。
3. 修改zoo.cfg文件，每个server端的zookeeper的server配置要相同：

```properties
# The number of milliseconds of each tick
tickTime=2000
# The number of ticks that the initial 
# synchronization phase can take
initLimit=10
# The number of ticks that can pass between 
# sending a request and getting an acknowledgement
syncLimit=5
# the directory where the snapshot is stored.
# do not use /tmp for storage, /tmp here is just 
# example sakes.
dataDir=/home/kkxmoye/local/zookeeper/data
# the port at which the clients will connect
clientPort=2181
# the maximum number of client connections.
# increase this if you need to handle more clients
#maxClientCnxns=60
#
# Be sure to read the maintenance section of the 
# administrator guide before turning on autopurge.
#
# http://zookeeper.apache.org/doc/current/zookeeperAdmin.html#sc_maintenance
#
# The number of snapshots to retain in dataDir
#autopurge.snapRetainCount=3
# Purge task interval in hours
# Set to "0" to disable auto purge feature
#autopurge.purgeInterval=1
server.0=192.168.180.129:2888:3888
server.1=192.168.180.130:2888:3888
server.2=192.168.180.132:2888:3888
```

配置文件参数说明：

server.A=B： C： D：

A：是一个数字，表示第几号服务器。

B：服务器的ip地址。

C：表示这个服务器与集群中的leader服务器交换信息的端口。

D：表示万一集群中leader服务器挂了，用这个端口与其他服务器交互信息选举出新的leader。



4. 在dataDir目录下创建myid文件，只有一行，值为对应的server数值，如上面文件中的0或1或2。如server1中：

![img](https://img-blog.csdn.net/20180812173619929?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM1NTAwMDYz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![拖曳以移動](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

5. 启动zookeeper：进入zookeeper bin目录，执行bash zkServer start命令，启动后，我们可以查看bin目录中的zookeeper.out文件查看日志，其中可能会出错：

![img](https://img-blog.csdn.net/20180812173902313?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM1NTAwMDYz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![拖曳以移動](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

我遇到这个错误，查找资料后得知是防火墙没有关闭，通过systemctl stop firewalld.service（生产环境肯定不能这样搞）关闭防火墙，相关命令：

systemctl stop firewalld.service  关闭防火墙

systemctl disable firewalld.service  禁用防火墙

systemctl status firewalld.service  查看当前防火墙状态

最后，配置成功：

server0:

![img](https://img-blog.csdn.net/20180812174411343?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM1NTAwMDYz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![拖曳以移動](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

server1:

![img](https://img-blog.csdn.net/20180812174529790?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM1NTAwMDYz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![拖曳以移動](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

server2:

![img](https://img-blog.csdn.net/20180812174648893?watermark/2/text/aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3NpbmF0XzM1NTAwMDYz/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70)![拖曳以移動](data:image/gif;base64,R0lGODlhAQABAPABAP///wAAACH5BAEKAAAALAAAAAABAAEAAAICRAEAOw==)

结果leader是server1，因为我是先启动的是server0和server1，最后启动的server3肯定是follower





一致性级别

- 强一致性
- 弱一致性
  - 会话一致性
  - 用户一致性
- 最终一致性

​	