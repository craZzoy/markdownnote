# 消息中间件的设计原则

## 基本功能

- 支持消息的发送和接收，涉及到网路通讯（NIO）
- 消息中心的持久化功能
- 消息的系列化和反系列化
- 是否跨语言
- 消息的确认机制，如何避免消息重发

## 高级功能

- 消息的有序性
- 是否支持事务消息
- 消息收发的性能，对高并发大数据量的支持
- 是否支持集群
- 消息的可靠性存储
- 是否支持多协议

# 简介



## 名词解析





# QuickStart

1. 下载kafka_2.12-2.5.0.tgz（自己选择版本）

   解压：tar -xzf  kafka_2.12-2.5.0.tgz

2. 启动服务

   1. 需先启动zookeeper

      ```shell
      # windows
      bin\windows\zookeeper-server-start.bat config\zookeeper.properties
      # linux
      bin/zookeeper-server-start.sh -damoen config/zookeeper.properties
      ```

   2. 启动服务

      ```shell
      # windows
      bin\windows\kafka-server-start.bat config\server.properties
      # linux
      bin/kafka-server-start.sh -damoen config/server.properties
      ```

3. 创建一个topic

   ```shell
   # linux
   bin/kafka-topics.sh --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
   # windows
   bin\windows\kafka-topics.bat --create --bootstrap-server localhost:9092 --replication-factor 1 --partitions 1 --topic test
   # 查看topic信息
   D:\dev-software\kafka_2.12-2.5.0>bin\windows\kafka-topics.bat --list --bootstrap-server localhost:9092
   __consumer_offsets
   log.business
   test
   ```

   - --replication-factor：表示给topic需要在不同的broker中保留几份，1表示在两个broker中保存两份
   - partitions：分区数

4. 发送消息

   ```shell
   > bin/kafka-console-producer.sh --bootstrap-server localhost:9092 --topic test
   This is a message
   This is another message
   ```

5. 消费消息

   ```shell
   > bin/kafka-console-consumer.sh --bootstrap-server localhost:9092 --topic test --from-beginning
   This is a message
   This is another message
   ```

6. 

