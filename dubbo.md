# Dubbo入门

dubbo是一个服务协调的框架，其中包括服务发现、服务治理、服务监控等服务。



## Hello World

基于dubbo编写一个hello world程序，先对其有一个整体的理解。

简单架构图：

![1569424435857](C:\Users\zwz\Documents\dubbo.assets\1569424435857.png)

### dubbo-pay-api接口提供

在dubbo-pay-api中定义了一些接口：

```java
package com.gupao;

public interface IQueryService {
    String queryPay(String info);
}
```

```java
package com.gupao;

public interface Iservice {
    String pay(String name);
}

```

通过将项目打包安装到本地仓库中待用。



## dubbo-pay-server

相关依赖：

```xml
            <!--将dubbo-pay-api引入-->
            <dependency>
                <groupId>com.gupao</groupId>
                <artifactId>dubbo-pay-api</artifactId>
                <version>1.0-SNAPSHOT</version>
            </dependency>

            <!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo -->
            <dependency>
                <groupId>org.apache.dubbo</groupId>
                <artifactId>dubbo</artifactId>
                <version>2.7.3</version>
            </dependency>
            <!--slf4j-->
            <dependency>
                <groupId>org.slf4j</groupId>
                <artifactId>slf4j-api</artifactId>
                <version>1.7.26</version>
            </dependency>
            <dependency>
                <groupId>ch.qos.logback</groupId>
                <artifactId>logback-classic</artifactId>
                <version>1.2.3</version>
            </dependency>

            <!--curator zk客户端实现，用于支持zookeeper注册中心-->
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-framework</artifactId>
                <version>4.0.1</version>
            </dependency>
            <dependency>
                <groupId>org.apache.curator</groupId>
                <artifactId>curator-recipes</artifactId>
                <version>4.0.1</version>
            </dependency>

            <!--rest协议支持：JAX-RS  CXF / Jersey /RESTEasy-->
            <dependency>
                <groupId>org.jboss.resteasy</groupId>
                <artifactId>resteasy-jaxrs</artifactId>
                <version>3.8.1.Final</version>
            </dependency>
            <dependency>
                <groupId>org.jboss.resteasy</groupId>
                <artifactId>resteasy-client</artifactId>
                <version>4.0.0.Final</version>
            </dependency>

            <!--jetty 容器-->
            <dependency>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-server</artifactId>
                <version>9.4.19.v20190610</version>
            </dependency>
            <dependency>
                <groupId>org.eclipse.jetty</groupId>
                <artifactId>jetty-servlet</artifactId>
                <version>9.4.19.v20190610</version>
            </dependency>

            <!--webservice支持-->
            <dependency>
                <groupId>org.apache.cxf</groupId>
                <artifactId>cxf-rt-frontend-simple</artifactId>
                <version>3.3.2</version>
            </dependency>
            <dependency>
                <groupId>org.apache.cxf</groupId>
                <artifactId>cxf-rt-transports-http</artifactId>
                <version>3.3.2</version>
            </dependency>

```

将前面pay-api包引入后实现接口用于服务：

```java
package com.gupao;

public class IServiceImpl implements Iservice{
    @Override
    public String pay(String name) {
        return "Hello "+name;
    }
}

```

```java
package com.gupao;

import javax.ws.rs.GET;
import javax.ws.rs.Path;
import javax.ws.rs.PathParam;

/**
 * 基于java-ws实现的rest服务，其实也类似spring中的rest实现
 */
@Path("/service")
public class IQueryServiceImpl implements IQueryService{

    @GET
    @Path("/query/{info}") //GetMapping
    @Override
    public String queryPay(@PathParam("info") String info) {
        return "Hello Dubbo:"+info;
    }
}

```

配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <!--当前应用信息-->
    <dubbo:application name="dubbo-pay-service"/>

    <!--配置注册中心，可配置多个注册中心-->
    <dubbo:registry id="rg1" address="zookeeper://119.23.201.241:2181"/>
    <!--N/A标识本地注册中心-->
    <dubbo:registry id="rg2" address="N/A"/>

    <!--多协议支持-->
    <dubbo:protocol name="dubbo" port="20880"/>

    <dubbo:protocol name="webservice" port="8080" server="jetty"/>

    <dubbo:protocol name="rest" port="8888" server="jetty"/>

    <!--服务注册-->
    <dubbo:service interface="com.gupao.IQueryService" ref="iQueryServiceImpl" protocol="rest" registry="rg1"/>

    <dubbo:service interface="com.gupao.Iservice" ref="iserviceImpl" protocol="dubbo,webservice" registry="rg1"/>

    <bean id="iserviceImpl" class="com.gupao.IServiceImpl"/>

    <bean id="iQueryServiceImpl" class="com.gupao.IQueryServiceImpl"/>

</beans>
```

启动程序：

```java
package com.gupao;

import org.apache.dubbo.container.Main;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.io.IOException;

/**
 * Hello world!
 *
 */
public class App 
{
    public static void main( String[] args ) throws IOException {
        Main.main(new String[]{"spring","log4j"}); //Dubbo提供的启动类方法，它会启动dobbu中配置的多个container


       /* //System.out.println( "Hello World!" );
        ClassPathXmlApplicationContext context =
                new ClassPathXmlApplicationContext(new String[]{"META-INF/application.xml"});
        context.start();
        //阻塞，按任意键退出
        System.in.read();*/
    }
}

```



此时zookeeper根节点下有一个dubbo命名空间，dubbo节点中是发布的服务信息：

```properties
[zk: 127.0.0.1:2181(CONNECTED) 9] ls /
[dubbo, zookeeper]
[zk: 127.0.0.1:2181(CONNECTED) 10] ls /dubbo
[com.gupao.IQueryService, com.gupao.Iservice]
```

可通过ZooInspector查看节点内容：

![1569423236633](C:\Users\zwz\Documents\dubbo.assets\1569423236633.png)

```properties
dubbo%3A%2F%2F169.254.238.92%3A20880%2Fcom.gupao.Iservice%3Fanyhost%3Dtrue%26application%3Ddubbo-pay-service%26bean.name%3Dcom.gupao.Iservice%26deprecated%3Dfalse%26dubbo%3D2.0.2%26dynamic%3Dtrue%26generic%3Dfalse%26interface%3Dcom.gupao.Iservice%26methods%3Dpay%26pid%3D30764%26register%3Dtrue%26release%3D2.7.3%26side%3Dprovider%26timestamp%3D1569321431208
```

```properties
webservice%3A%2F%2F169.254.238.92%3A8080%2Fcom.gupao.Iservice%3Fanyhost%3Dtrue%26application%3Ddubbo-pay-service%26bean.name%3Dcom.gupao.Iservice%26deprecated%3Dfalse%26dubbo%3D2.0.2%26dynamic%3Dtrue%26generic%3Dfalse%26interface%3Dcom.gupao.Iservice%26methods%3Dpay%26pid%3D30764%26register%3Dtrue%26release%3D2.7.3%26server%3Djetty%26side%3Dprovider%26timestamp%3D1569321431389
```

```properties
rest%3A%2F%2F169.254.238.92%3A8888%2Fcom.gupao.IQueryService%3Fanyhost%3Dtrue%26application%3Ddubbo-pay-service%26bean.name%3Dcom.gupao.IQueryService%26deprecated%3Dfalse%26dubbo%3D2.0.2%26dynamic%3Dtrue%26generic%3Dfalse%26interface%3Dcom.gupao.IQueryService%26methods%3DqueryPay%26pid%3D30764%26register%3Dtrue%26release%3D2.7.3%26server%3Djetty%26side%3Dprovider%26timestamp%3D1569321427544
```



此时可查看本机中的相关tcp服务：

```properties
#rest服务
C:\Users\zwz>netstat -ano|findstr "8888"
  TCP    0.0.0.0:8888           0.0.0.0:0              LISTENING       30764
  TCP    [::]:8888              [::]:0                 LISTENING       30764
#webvices服务
C:\Users\zwz>netstat -ano|findstr "8080"
  TCP    0.0.0.0:8080           0.0.0.0:0              LISTENING       30764
  TCP    [::]:8080              [::]:0                 LISTENING       30764
#dubbo服务
C:\Users\zwz>netstat -ano|findstr "20880"
  TCP    0.0.0.0:20880          0.0.0.0:0              LISTENING       30764
  TCP    [::]:20880             [::]:0                 LISTENING       30764
```



## dubbo-order-service

依赖与dubbo-pay-service类似

dubbo配置文件：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:dubbo="http://dubbo.apache.org/schema/dubbo"
       xsi:schemaLocation="http://www.springframework.org/schema/beans        http://www.springframework.org/schema/beans/spring-beans-4.3.xsd        http://dubbo.apache.org/schema/dubbo        http://dubbo.apache.org/schema/dubbo/dubbo.xsd">

    <dubbo:application name="dubbo-order-service"/>

    <!--注册中心-->
    <!--<dubbo:registry address="N/A"/>-->
    <dubbo:registry id="rg1" address="zookeeper://119.23.201.241:2181"/>

    <!--协议-->
    <dubbo:protocol name="dubbo" port="20880"/>

    <!--生成远程服务代理，从注册中心rg1中取-->
    <!--此时可以像使用本地bean一样-->
    <dubbo:reference id="iserviceImpl" interface="com.gupao.Iservice" registry="rg1"/>


</beans>
```



dubbo协议服务验证：

```java
package com.gupao;

import org.springframework.context.support.ClassPathXmlApplicationContext;

/**
 * Hello world!
 *
 */
public class App 
{
    public static void main( String[] args )
    {
        //System.out.println(Float.valueOf(12));
        //System.exit(0);
        ClassPathXmlApplicationContext context
                = new ClassPathXmlApplicationContext(new String[]{"application.xml"});
        context.start();
        Iservice iserviceImpl = (Iservice) context.getBean("iserviceImpl");
        System.out.println(iserviceImpl.pay("Dubbo"));
    }
}

```

结果：

```
Hello Dubbo
```



rest服务验证：

http://169.254.238.92:8888/service/query/tom

![1569321607055](C:\Users\zwz\Documents\dubbo.assets\1569321607055.png)



webservice服务验证：

http://169.254.238.92:8080/com.gupao.Iservice?wsdl

![1569424251664](C:\Users\zwz\Documents\dubbo.assets\1569424251664.png)



## XML配置文件标签解释

| 标签                                                         | 用途         | 解释                                                         |
| ------------------------------------------------------------ | ------------ | ------------------------------------------------------------ |
| `<dubbo:service/>`                                           | 服务配置     | 用于暴露一个服务，定义服务的元信息，一个服务可以用多个协议暴露，一个服务也可以注册到多个注册中心 |
| `<dubbo:reference/>`[[2\]](http://dubbo.apache.org/zh-cn/docs/user/configuration/xml.html#fn2) | 引用配置     | 用于创建一个远程服务代理，一个引用可以指向多个注册中心       |
| `<dubbo:protocol/>`                                          | 协议配置     | 用于配置提供服务的协议信息，协议由提供方指定，消费方被动接受 |
| `<dubbo:application/>`                                       | 应用配置     | 用于配置当前应用信息，不管该应用是提供者还是消费者           |
| `<dubbo:module/>`                                            | 模块配置     | 用于配置当前模块信息，可选                                   |
| `<dubbo:registry/>`                                          | 注册中心配置 | 用于配置连接注册中心相关信息                                 |
| `<dubbo:monitor/>`                                           | 监控中心配置 | 用于配置连接监控中心相关信息，可选                           |
| `<dubbo:provider/>`                                          | 提供方配置   | 当 ProtocolConfig 和 ServiceConfig 某属性没有配置时，采用此缺省值，可选 |
| `<dubbo:consumer/>`                                          | 消费方配置   | 当 ReferenceConfig 某属性没有配置时，采用此缺省值，可选      |
| `<dubbo:method/>`                                            | 方法配置     | 用于 ServiceConfig 和 ReferenceConfig 指定方法级的配置信息   |
| `<dubbo:argument/>`                                          | 参数配置     | 用于指定方法参数配置                                         |







# dubbo服务治理的体现

## Springboot+Dubbo

1. 生态
2. 标准化

### 服务提供方

包依赖：

```properties
<!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo-spring-boot-starter -->
		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo-spring-boot-starter</artifactId>
			<version>2.7.3</version>
		</dependency>

		<dependency>
			<groupId>com.gupao</groupId>
			<artifactId>dubbo-pay-api</artifactId>
			<version>1.0-SNAPSHOT</version>
		</dependency>

		<!-- https://mvnrepository.com/artifact/org.apache.dubbo/dubbo -->
		<dependency>
			<groupId>org.apache.dubbo</groupId>
			<artifactId>dubbo</artifactId>
			<version>2.7.3</version>
		</dependency>

		<!--curator zk客户端实现，用于支持zookeeper注册中心-->
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-framework</artifactId>
			<version>4.0.1</version>
		</dependency>
		<dependency>
			<groupId>org.apache.curator</groupId>
			<artifactId>curator-recipes</artifactId>
			<version>4.0.1</version>
		</dependency>
```

dubbo相关配置：

```properties
spring.application.name=springboot-dubbo-privider
server.port=8090

dubbo.application.name=springboot-dubbo-privider
#注册中心
dubbo.registry.address=zookeeper://119.23.201.241:2181
dubbo.scan.base-packages=com.example

```

通过注解发布服务

```java
package com.example.impl;

import com.gupao.IHelloService;
import org.apache.dubbo.config.annotation.Service;

@Service
public class IHelloServiceImpl implements IHelloService {


    @Override
    public String hello(String info) {
        return "hello "+info;
    }
}

```



### 服务调用方

配置：

```properties
server.port=8090
dubbo.application.name=springboot-dubbo-consumer
dubbo.registry.address=zookeeper://119.23.201.241:2181
dubbo.scan.base-packages=com.example
```

注册引用服务：

```java
package com.example.demo;

import com.gupao.IHelloService;
import org.apache.dubbo.config.annotation.Reference;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class DubboController {

    @Reference
    IHelloService helloService;

    @GetMapping("/hello")
    public String hello(){
        System.out.println("service running");
        return helloService.hello("Dubbo");
    }
}

```

访问http://localhost:8090/hello调用结果成功



## 负载均衡

当服务被发布为多个节点时，客户端需要根据负载均衡算法而将其请求路由到服务的某个节点，负载均衡分为硬件负载和软件负载，硬件负载成本较高，一般用的软件负载。



### 演示

可根据vm参数在不同的端口发布服务：

```properties
-Ddubbo.protocol.port=20881
```

加入分配随机策略

```java
@Reference(loadbalance = "ramdom")
IHelloService helloService;
```

运行结果：

![1569493895260](C:\Users\zwz\Documents\dubbo.assets\1569493895260.png)

![1569493906862](C:\Users\zwz\Documents\dubbo.assets\1569493906862.png)



服务方和调用方都可以配置：

```java
@Service(loadbalance = "ramdom")
public class IHelloServiceImpl implements IHelloService {
    @Override
    public String hello(String info) {
        System.out.println("i am server");
        return "hello "+info;
    }
}
```



### 配置优先级

官网中以timeout为例子，retries、loadBlance、actives等类似：

- 方法级>接口级>全局配置
- 如果级别（方法级别、接口级别、全局）一样，则消费方优先，提供方次之

![1569494298381](C:\Users\zwz\Documents\dubbo.assets\1569494298381.png)



### 负载均衡算法

1. random：权重随机算法，根据权重值进行随机负载
2. roundrobin：加权轮询。默认对应每个服务器都落入公平的请求。假如有三个节点提供服务，加权比为1:3:6，那么10次请求分别分发1个、3个、6个。
3. leastActive：最少活跃调用数算法。相同活跃数随机，活跃数指调用前后计算差。活跃数越小值越大（积累的请求数多）
4. consistentHash：一致性hash。相同参数的请求总是发到同一提供者



## 集群容错

容错，即容忍错误的情况。在dubbo中通过service的参数cluster设置。缺省参数是failsafe

cluster参数：

1.  failover：当前节点失效时，重试其它服务器，默认是三次，retries=2（加上第一次是三次）
2. failfast：快速失败，不重试，只发送一次调用，失败立即报错。通常用于非幂等性的写操作，比如新增操作。
3. failback：失败自动恢复，后台记录失败请求，定时重发。通常用于消息通知
4. failsafe：失败安全，忽略错误。通常会记录日志
5. forking：并行调用多个服务，只要一个成功就返回。通常用于实时性要求比较高的读操作，但需要浪费更多服务器资源。可通过forks=“2”参数来设置最大并行数
6. broadcast：广播调用所有提供者，这个调用，任意一台不可用则报错。通常用于通知所有提供者更新缓存或日志等本地资源信息。



## 服务降级

当某个不重要的功能出现错误时，可以通过降级功能来屏蔽这个服务。

一些概念：

- 人工降级：通过人工关闭某些功能，如限时活动
- 异常降级：异常时返回兜底数据
- 限流、熔断降级：请求量达到某个阈值时，降低以减少请求数来限制流量。

dubbo通过mock机制来实现降级：

```java
    @Reference(loadbalance = "ramdom",timeout=2000,
            cluster = "failfast",mock = "com.example.demo.IHelloServiceMock")
    IHelloService helloService;
```

```java
package com.example.demo;

import com.gupao.IHelloService;

public class IHelloServiceMock implements IHelloService{
    @Override
    public String hello(String info) {
        return "服务端异常了，服务降级，返回兜底数据";
    }
}

```

当出现错误时，此时不是返回错误，还是返回一个指定的兜底数据：

![1569507179071](C:\Users\zwz\Documents\dubbo.assets\1569507179071.png)

## 启动时检查

​	dubbo缺省会在启动时检查依赖的服务是否可用，不可用时会抛出异常，阻止spring的初始化完成，默认`check=“true”`。

​	可通过`check=“false”`关闭检查，如测试时，有些服务不太关心，后者出现循环依赖，必须有一方先启动的情况。

示例：

spring配置文件：

- 关闭某个服务的启动时检查（没有提供者报错）

  ```xml
  <dubbo:reference interface="com.foo.BarService" check="false" />
  ```

- 关闭所有服务的启动时检查（没有提供者报错）

  ```xml
  <dubbo:consumer check="false" />
  ```

- 关闭注册中心启动时检查（注册订阅失败时报错）
  ```xml
  <dubbo:registry check="false" />
  ```

通过dubbo.properties文件，该配置文件一般放在classpath路径下

```properties
dubbo.reference.com.foo.BarService.check=false
dubbo.reference.check=false
dubbo.consumer.check=false
dubbo.registry.check=false
```

通过-D参数：

```properties
java -Ddubbo.reference.com.foo.BarService.check=false
java -Ddubbo.reference.check=false
java -Ddubbo.consumer.check=false 
java -Ddubbo.registry.check=false
```



配置优先级：

1. JVM-D参数
2. xml配置文件（spring）
3. properties文件（dubbo.properties）：Dubbo可以自动加载classpath根目录下的dubbo.properties，但是你同样可以使用JVM参数来指定路径：`-Ddubbo.properties.file=xxx.properties`。



### 主机绑定

缺省主机 IP 查找顺序：

- 通过 `LocalHost.getLocalHost()` 获取本机地址。
- 如果是 `127.*` 等 loopback 地址，则扫描各网卡，获取网卡 IP。



#### 主机配置

```properties
#主机配置
dubbo.protocol.host=127.0.0.1
dubbo.protocol.name=dubbo
dubbo.protocol.port=20880
```



#### 端口配置

缺省端口配置：

| 协议       | 端口  |
| ---------- | ----- |
| dubbo      | 20880 |
| rmi        | 1099  |
| http       | 80    |
| hessian    | 80    |
| webservice | 80    |
| memcached  | 11211 |
| redis      | 6379  |



## 动态配置中心

默认配置中心优先级高

```properties
#外部化配置
#是否外部化配置优先，默认true
dubbo.config-center.highest-priority=true
dubbo.config-center.address=zookeeper://119.23.201.241:2181
```

zookeeper中的实现原理：

![1569587266167](C:\Users\zwz\Documents\dubbo.assets\1569587266167.png)



## 元数据中心

```properties
dubbo.metadata-report.address=zookeeper://119.23.201.241:2181
#注册到注册中心的URL是否采用精简模式
dubbo.registry.simplified=true
```

simplified参数作用：

```
#简化后
dubbo%3A%2F%2F169.254.238.92%3A20882%2Fcom.gupao.IHelloService%3Fapplication%3Dspringboot-dubbo-privider%26deprecated%3Dfalse%26dubbo%3D2.0.2%26release%3D2.7.3%26timestamp%3D1569587722130
#简化前
dubbo%3A%2F%2F169.254.238.92%3A20882%2Fcom.gupao.IHelloService%3Fanyhost%3Dtrue%26application%3Dspringboot-dubbo-privider%26bean.name%3DServiceBean%3Acom.gupao.IHelloService%26deprecated%3Dfalse%26dubbo%3D2.0.2%26dynamic%3Dtrue%26generic%3Dfalse%26interface%3Dcom.gupao.IHelloService%26methods%3Dhello%26pid%3D13824%26register%3Dtrue%26release%3D2.7.3%26side%3Dprovider%26timestamp%3D1569587767191
```

