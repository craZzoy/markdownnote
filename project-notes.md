# 交易服务整合（HFSI）子系统

## 服务器地址信息

HFSI地址：
128.192.119.1
cenfrt/ceb7@ZvT

资源路径：





Digester开源组件进行装配







adapter

interceptor



ThreadLocal



reminder-conf.xml

技术：

1. 生产禁用远程监控，而使用JVMTI、JPDA等
2. 传输敏感数据时使用SSLSocket，而非Socket，但SSLSocket性能开销较大
3. SecureRandom是安全的，而Random是伪随机数，其还是可以被预测。对于敏感数据处理应使用SecureRandom（java8以后使用java.security.SecureRandom#getInstanceStrong，之前使用其构造器）
4. 加密算法的使用
5. 应用服务器weblogic、Tuxedo
6. HAProxy、keepalived
7. Hadoop、HBase
8. 分布式缓存组件coherence
   - 用途：共享一个运用的对象（如一个会话java）或者数据（如数据库中的数据）
9. JMX(Java Management Extension)
   - 支持协议：SNMP、JAVA RMI、HTTP
10. java -jar xxx.jar par...，给指定的jar中传递参数并执行。
11. properties文件的读取，spring中的实现
12. jenkin、nocas、eruka





处理流程：

1. 数据映射
   - 渠道级映射channel-special.xml
     - 请求映射
     - 响应映射
     - 错误映射
2. 





# 服务管理

- 简单服务

- 原子服务

  - provider.xml

    

  - 





# 公共组件

## compositeData数据结构

- 域：最小数据单元
- 数组
- 结构：即key-value结构



## 流控组件





# 代码、配置

## 渠道

### 配置

- ../dist/conf/channel
  - `channel.xml`：配置该渠道的渠道处理器和渠道特殊处理器
  - 渠道处理器：处理请求
  - 渠道特殊处理器：拆包、打包操作
  - `channel-special.xml`：配置渠道级（该渠道下所有交易）字段映射，包括请求、响应、错误等



数据库表：flat_trcd-> trad_conv->trad_rout（根据唯一的平台客户号trad_no关联）

​					渠道交易码->平台交易码->定位到平台服务





交易相关配置：

- dist/rules目录
  - provider-rule.xml(每个服务目录下)：provider.xml文件装载规则
  - service-rule.xml(每个服务目录下)：service.xml文件装载规则
- dist/conf目录
  - service.xml：dist/conf/atom-service/{serviceName}目录下，配置拆包模式，包括请求响应等
  - provider.xml：dist/conf/atom-service/{serviceName}目录下，配置组包模式，包括请求、响应、错误报文
  - mode.xml：gloabl目录下，每个模式对应一个PackageConverter实现类，
    - pack方法：用于打包操作：CompositeData->OutputPacket
    - unpack方法：用于拆包操作：InputPacket->CompositeData







# 业务相关知识

会计、科目

借、贷



技术组件

Cassandra：一种nosql数据库，用于存储账号-分行索引。。。





# 编码规范

## 输入数据校验

防止SQL注入

- 使用参数化查询
- 对不可信数据进行校验





Hession系列化