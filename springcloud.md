基于springcloud的微服务解决方案



基于dubbo的微服务解决方案



# Cloud native Application



# spring-cloud-config

Spring Cloud config支持多种存储配置信息的形式：

- jdbc
- Vault
- Native
- svn
- git



## Spring-cloud-config-client

### 预备知识

#### 发布订阅模式模式

- `java.util.Observable`：发布者
- `java.util.Observer`：订阅者
- 关系
  - `1:1`
  - `1:N`
- 特性
  - 推模式：被动获取，对比Iterator的迭代（可认为是拉模式，是主动获取的）
    - `java.util.Observable#notifyObservers(java.lang.Object)`

```java
package com.gupao.springcloudconfigclient.demo;

import java.util.*;

public class ObserverDemo {

    public static void main(String[] args) {

        MyObservable observable = new MyObservable();
        // 增加订阅者
        observable.addObserver(new Observer() {
            @Override
            public void update(Observable o, Object value) {
                System.out.println(value);
            }
        });

        observable.setChanged();
        // 发布者通知，订阅者是被动感知（推模式）
        observable.notifyObservers("Hello,World");

        echoIterator();

    }

    private static void echoIterator(){
        List<Integer> values = Arrays.asList(1,2,3,4,5);
        Iterator<Integer> integerIterator = values.iterator();
        while(integerIterator.hasNext()){ // 通过循环，主动去获取
            System.out.println(integerIterator.next());
        }
    }



    public static class MyObservable extends Observable {

        public void setChanged() {
            super.setChanged();
        }
    }
}

```



#### 事件/监听模式

- `java.util.EventObject`：事件对象

  - 事件对象总是关联事件源source

    ```java
        /**
         * The object on which the Event initially occurred.
         */
        protected transient Object  source;
    ```

- `java.util.EventListener`：事件监听接口



#### Spring事件/监听

- ApplicationEvent：应用事件，继承了java api中EventObject
- ApplicationEventListener：应用监听器，继承了Java api中的EventListener

![1569944408650](springcloud.assets\1569944408650.png)

```java
package demo;


import org.springframework.context.ApplicationContext;
import org.springframework.context.ApplicationEvent;
import org.springframework.context.ApplicationListener;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;

public class SpringEventListenerDemo {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context =
                new AnnotationConfigApplicationContext();
        context.addApplicationListener(new ApplicationListener<MyApplicationEvent>() {
            /**
             * 监听器得到事件
             * @param event
             */
            @Override
            public void onApplicationEvent(MyApplicationEvent event) {
                System.out.println("监听器得到事件："+event.getSource()+"@"+event.getApplicationContext());
            }
        });
        context.refresh();
        //发布事件，即触发事件发生
        context.publishEvent(new MyApplicationEvent(context,"Hello,World"));
        context.publishEvent(new MyApplicationEvent(context,1));
        context.publishEvent(new MyApplicationEvent(context,System.currentTimeMillis()));

    }

    private static class MyApplicationEvent extends ApplicationEvent{

        private final ApplicationContext context;

        /**
         * Create a new ApplicationEvent.
         *
         * @param source the object on which the event initially occurred (never {@code null})
         */
        public MyApplicationEvent(ApplicationContext context, Object source) {
            super(source);
            this.context = context;
        }

        public ApplicationContext getApplicationContext(){
            return this.context;
        }
    }
}

```

context.publishEvent实际调用方法：`org.springframework.context.event.SimpleApplicationEventMulticaster#multicastEvent(org.springframework.context.ApplicationEvent, org.springframework.core.ResolvableType)`

```java
	@Override
	public void multicastEvent(final ApplicationEvent event, @Nullable ResolvableType eventType) {
		ResolvableType type = (eventType != null ? eventType : resolveDefaultEventType(event));
		Executor executor = getTaskExecutor();
		for (ApplicationListener<?> listener : getApplicationListeners(event, type)) {
			if (executor != null) {
				executor.execute(() -> invokeListener(listener, event));
			}
			else {
				invokeListener(listener, event);
			}
		}
	}
```

最终在`org.springframework.context.event.SimpleApplicationEventMulticaster#doInvokeListener`中执行了`org.springframework.context.ApplicationListener#onApplicationEvent`方法

```java
	@SuppressWarnings({"rawtypes", "unchecked"})
	private void doInvokeListener(ApplicationListener listener, ApplicationEvent event) {
		try {
			listener.onApplicationEvent(event);
		}
		catch (ClassCastException ex) {
			String msg = ex.getMessage();
			if (msg == null || matchesClassCastMessage(msg, event.getClass())) {
				// Possibly a lambda-defined listener which we could not resolve the generic event type for
				// -> let's suppress the exception and just log a debug message.
				Log logger = LogFactory.getLog(getClass());
				if (logger.isTraceEnabled()) {
					logger.trace("Non-matching event type for listener: " + listener, ex);
				}
			}
			else {
				throw ex;
			}
		}
	}
```



#### Spring Boot事件监听

类似java中的SPI，java.util.ServiceLoader，对于ApplicationListener，spring boot中也定义了相应的SPI。

相对于 ClassPath ： /META-INF/spring.factories

```properties
# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.boot.ClearCachesApplicationListener,\
org.springframework.boot.builder.ParentContextCloserApplicationListener,\
org.springframework.boot.context.FileEncodingApplicationListener,\
org.springframework.boot.context.config.AnsiOutputApplicationListener,\
org.springframework.boot.context.config.ConfigFileApplicationListener,\
org.springframework.boot.context.config.DelegatingApplicationListener,\
org.springframework.boot.context.logging.ClasspathLoggingApplicationListener,\
org.springframework.boot.context.logging.LoggingApplicationListener,\
org.springframework.boot.liquibase.LiquibaseServiceLocatorApplicationListener
```

关注下`org.springframework.boot.context.config.ConfigFileApplicationListener`的作用，它主要用于管理配置文件，比如

- application-{profile}.properties
- application.poperties

通过读取这些文件构建PropertySource加入到某个Enviroment中的PropertySource列表中

它主要监听了两种ApplicationEvent：

```java
	@Override
	public void onApplicationEvent(ApplicationEvent event) {
		if (event instanceof ApplicationEnvironmentPreparedEvent) {
			onApplicationEnvironmentPreparedEvent((ApplicationEnvironmentPreparedEvent) event);
		}
		if (event instanceof ApplicationPreparedEvent) {
			onApplicationPreparedEvent(event);
		}
	}
```

`org.springframework.boot.context.config.ConfigFileApplicationListener#onApplicationEnvironmentPreparedEvent`方法用于管理配置文件



#### Spring Cloud事件监听

##### BootstrapApplicationListener

Spring Cloud SPI“ /META-INF/spring.factories”

```properties
# Application Listeners
org.springframework.context.ApplicationListener=\
org.springframework.cloud.bootstrap.BootstrapApplicationListener,\
org.springframework.cloud.bootstrap.LoggingSystemShutdownListener,\
org.springframework.cloud.context.restart.RestartListener
```

1. 负责加载`bootstrap.properties` 或者 `bootstrap.yaml`

2. 负责初始化BootStrap ApplicationContext（ConfigurableApplicationContext实现类），其是跟应用上下文，parent是null

   ```java
                       context = this.bootstrapServiceContext(environment, event.getSpringApplication(), configName);
   ```

BootStrap ApplicationContext默认名字为bootstrap，在BootstrapApplicationListener中有写明：

```java
                String configName = environment.resolvePlaceholders("${spring.cloud.bootstrap.name:bootstrap}");
```

通过在application.properties中设置名字：

```properties
server.port=8090
management.endpoint.env.enabled=true
#更改bootstrap context名字
spring.cloud.bootstrap.name=abc
```

原因：

`BootstrapApplicationListener ` 加载顺序比`ConfigFileApplicationListener` 优先，即`ConfigFileApplicationListener` 读取application.xml中的参数时，`BootstrapApplicationListener ` 已经初始化了bootstrap context，所有并不能改变其名字

`BootstrapApplicationListener ` order：

```java
public static final int DEFAULT_ORDER = -2147483643;
```

`ConfigFileApplicationListener` order：

```java
public static final int DEFAULT_ORDER = Ordered.HIGHEST_PRECEDENCE + 10; //-2147483628
```

可通过program arguments设置：

```properties
--spring.cloud.bootstrap.name=abc
```



org.springframework.context.support.AbstractApplicationContext理解



#### Env端点：EnvironmentEndpoint

Environment可以关联多个带名称的PropertySource，其有两种实现方式：

- 普通类型：`StandardEnvironment`
- web类型：`StandardServletEnvironment`

如spring中的`AbstractRefreshableWebApplicationContext`：

```java
protected void initPropertySources() {
  ConfigurableEnvironment env = getEnvironment();
  if (env instanceof ConfigurableWebEnvironment) {
    ((ConfigurableWebEnvironment) env).initPropertySources(this.servletContext, this.servletConfig);
  }
}
```

Enviroment 关联着一个`PropertySources` 实例

`PropertySources` 关联着多个`PropertySource`，并且有优先级

其中比较常用的`PropertySource` 实现：

Java System#getProperties 实现：  名称"systemProperties"，对应的内容 `System.getProperties()`

Java System#getenv 实现(环境变量）：  名称"systemEnvironment"，对应的内容 `System.getProperties()`

springboot中`PropertySource` 的加载顺序：

1. [Devtools global settings properties](https://docs.spring.io/spring-boot/docs/2.0.0.BUILD-SNAPSHOT/reference/htmlsingle/#using-boot-devtools-globalsettings) on your home directory (`~/.spring-boot-devtools.properties` when devtools is active).
2. [`@TestPropertySource`](https://docs.spring.io/spring/docs/5.0.4.RELEASE/javadoc-api/org/springframework/test/context/TestPropertySource.html) annotations on your tests.
3. [`@SpringBootTest#properties`](https://docs.spring.io/spring-boot/docs/2.0.0.BUILD-SNAPSHOT/api/org/springframework/boot/test/context/SpringBootTest.html) annotation attribute on your tests.
4. Command line arguments.
5. Properties from `SPRING_APPLICATION_JSON` (inline JSON embedded in an environment variable or system property).
6. `ServletConfig` init parameters.
7. `ServletContext` init parameters.
8. JNDI attributes from `java:comp/env`.
9. Java System properties (`System.getProperties()`).
10. OS environment variables.
11. A `RandomValuePropertySource` that has properties only in `random.*`.
12. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/2.0.0.BUILD-SNAPSHOT/reference/htmlsingle/#boot-features-external-config-profile-specific-properties) outside of your packaged jar (`application-{profile}.properties` and YAML variants).
13. [Profile-specific application properties](https://docs.spring.io/spring-boot/docs/2.0.0.BUILD-SNAPSHOT/reference/htmlsingle/#boot-features-external-config-profile-specific-properties) packaged inside your jar (`application-{profile}.properties` and YAML variants).
14. Application properties outside of your packaged jar (`application.properties` and YAML variants).
15. Application properties packaged inside your jar (`application.properties` and YAML variants).
16. [`@PropertySource`](https://docs.spring.io/spring/docs/5.0.4.RELEASE/javadoc-api/org/springframework/context/annotation/PropertySource.html) annotations on your `@Configuration` classes.
17. Default properties (specified by setting `SpringApplication.setDefaultProperties`).



#### 自定义实现配置

1. 暴露配置Bean

   ```java
       @Configuration
       @Order(Ordered.HIGHEST_PRECEDENCE)
       public static class MyPropertySourceLoader implements PropertySourceLocator {
   
           public PropertySource<?> locate(Environment environment) {
               Map<String,Object> source = new HashMap<String, Object>();
               source.put("server.port","9090");
               MapPropertySource mapPropertySource =
                       new MapPropertySource("my-property-source",source);
               return mapPropertySource;
           }
       }
   ```

2. 在classpath中添加META-INF/spring.factories，即springcloud SPI

   ```properties
   org.springframework.cloud.bootstrap.BootstrapConfiguration=\
   com.SpringBootCloudClientStarter.MyPropertySourceLoader
   ```

> 注意：Order数值越小，越优先

> 注：org.springframework.boot.context.config.ConfigFileApplicationListener#onApplicationEvent监听到了这事件，org.springframework.cloud.bootstrap.BootstrapApplicationListener#onApplicationEvent没有，可认为不在Bootstrap Context中？





注意事项：

`Environment` 允许出现同名的配置，不过优先级高的胜出

内部实现：`MutablePropertySources` 关联代码：

```java
List<PropertySource<?>> propertySourceList = new CopyOnWriteArrayList<PropertySource<?>>();
```

propertySourceList FIFO，它有顺序

可以通过 MutablePropertySources#addFirst 提高到最优先，相当于调用：

`List#add(0,PropertySource);`



### 问题

1. .yml和.yaml是啥区别？

   答：没有区别，就是文件扩展名不同

2. 自定义的配置在平时使用的多吗 一般是什么场景

   答：不多，一般用于中间件的开发

3. Spring 里面有个`@EventListener`和`ApplicationListener`什么区别

   答：没有区别，前者是 Annotation 编程模式，后者 接口编程

4. 小马哥 可以讲课的时候简单的实现一个小项目，在讲原理和源码吧，直接上源码，感觉讲得好散，听起来好累

   答：从第三节开始直接开始从功能入

5. `/env` 端点的使用场景 是什么

   答：用于排查问题，比如要分析`@Value("${server.port}")`里面占位符的具体值

6. Spring cloud 会用这个实现一个整合起来的高可用么

   答：Spring Cloud 整体达到一个目标，把 Spring Cloud 的技术全部整合到一个项目，比如负载均衡、短路、跟踪、服务调用等

7. 怎样防止Order一样

   答：Spring Boot 和 Spring Cloud 里面没有办法，在 Spring Security 通过异常实现的。

8. 服务监控跟鹰眼一样吗

   答：类似

9. bootstrapApplicationListener是引入cloud组件来有的吗

   答：是的

10. pom.xml引入哪个cloud组件了？

    答：

    ```xml
    <dependency>
       <groupId>org.springframework.cloud</groupId>
       <artifactId>spring-cloud-starter-config</artifactId>
    </dependency>
    ```

    

    



## Spring-cloud-config-server

```properties
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-config-server</artifactId>
</dependency>
```

### 构建 Spring Cloud 配置服务器

#### 实现步骤

1. 在 Configuration Class 标记`@EnableConfigServer`

2. 配置文件目录（基于 git）

   1. gupao.properties （默认） // 默认环境，跟着代码仓库

   2. gupao-dev.properties ( profile = "dev") // 开发环境

   3. gupao-test.properties ( profile = "test") // 测试环境

   4. gupao-staging.properties ( profile = "staging") // 预发布环境

   5. gupao-prod.properties ( profile =  "prod") // 生产环境

      ```
      十月 25 20:46 gupao.properties
      十月 25 20:52 gupao-dev.properties
      十月 25 20:53 gupao-prod.properties
      十月 25 20:52 gupao-staging.properties
      十月 25 20:47 gupao-test.properties
      ```

3. 服务端配置配置版本仓库（本地）

   ```properties
   spring.cloud.config.server.git.uri = \
     file:///E:/Google%20Driver/Private/Lessons/gupao/xiaomage-space/
   ```

> 注意：放在存有`.git`的根目录

完整的配置项：

```properties
### 配置服务器配置项
spring.application.name = config-server
### 定义HTTP服务端口
server.port = 9090
### 本地仓库的GIT URI 配置
spring.cloud.config.server.git.uri = \
  file:///E:/Google%20Driver/Private/Lessons/gupao/xiaomage-space/

### 全局关闭 Actuator 安全
# management.security.enabled = false
### 细粒度的开放 Actuator Endpoints
### sensitive 关注是敏感性，安全
endpoints.env.sensitive = false
```



### 构建 Spring Cloud 配置客户端

```properties
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-config</artifactId>
</dependency>
```

#### 实现步骤

1. 创建`bootstrap.properties` 或者 `bootstrap.yml`文件

2. `bootstrap.properties` 或者 `bootstrap.yml`文件中配置客户端信息

   ```properties
   ### bootstrap 上下文配置
   # 配置服务器 URI
   spring.cloud.config.uri = http://localhost:9090/
   # 配置客户端应用名称:{application}
   spring.cloud.config.name = gupao
   # profile 是激活配置
   spring.cloud.config.profile = prod
   # label 在Git中指的分支名称
   spring.cloud.config.label = master 
   ```

3. 设置关键 Endpoints 的敏感性

   ```properties
   ### 配置客户端配置项
   spring.application.name = config-client
   
   #暴露/refresh endpoint
   management.endpoints.enabled-by-default=true
   management.endpoints.web.exposure.include=refresh,health,env,beans
   ```

这样，client端就可以读取到server端提供的配置：

### @RefreshScope 用法

```java
@RestController
@RefreshScope//
public class EchoController {

    @Value("${my.name}")
    private String myName;

    @GetMapping("/my-name")
    public String getName(){
        return myName;
    }

}
```

当server值改变后，通过调用`/refresh` Endpoint 控制客户端配置更新的，开关、阈值、`@RefreshScope`常用于文案等等场景



### 实现定时更新客户端

除调用/refresh endpoint外，还可用定时刷新配置：

```java
package com.demo;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.context.refresh.ContextRefresher;
import org.springframework.core.env.Environment;
import org.springframework.scheduling.annotation.Scheduled;

import java.util.Set;

@SpringBootApplication
public class ConfigClientStarter {

    public static void main(String[] args) {
        SpringApplication.run(ConfigClientStarter.class,args);
    }

    private final Environment environment;
    private final ContextRefresher refresher;

    public ConfigClientStarter(Environment environment, ContextRefresher refresher) {
        this.environment = environment;
        this.refresher = refresher;
    }

    /**
     * 通过refresher定时刷新配置
     */
    @Scheduled(fixedRate = 5000, initialDelay = 3000)
    public void autoRefresh(){
        Set<String> strings = refresher.refresh();
        strings.stream().forEach(s -> {
            System.err.printf("[Thread :%s] 当前配置已经更新，key：%s，value：%s \n",
                    Thread.currentThread().getName(),
                    s,
                    environment.getProperty(s));
        });

    }
}

```



### 结合@RefreshScope手动更新配置

![1585143487382](springcloud.assets\1585143487382.png)

代码示例见ch11-1config-client、ch11-1config-server



### 结合Spring Cloud Bus热刷新

![1585451146326](springcloud.assets\1585451146326.png)

代码示例见ch11-3-config-client-bus、ch11-3-config-server-bus

server端推送配置：http://localhost:9090/actuator/bus-refresh（POST），若不想手动触发，可以在git上配置Webhooks



springcloud config下篇未看



## 健康检查

### 意义

比如应用可以任意地输出业务健康、系统健康等指标



端点URI：`/health`

实现类：`HealthEndpoint`

健康指示器：`HealthIndicator`，

`HealthEndpoint`：`HealthIndicator` ，一对多



### 自定义实现`HealthIndicator`

1. 实现`AbstractHealthIndicator`

   ```java
   public class MyHealthIndicator extends AbstractHealthIndicator {
   
       @Override
       protected void doHealthCheck(Health.Builder builder)
               throws Exception {
           builder.up().withDetail("MyHealthIndicator","Day Day Up");
       }
   }
   ```

   

2. 暴露 `MyHealthIndicator` 为 `Bean`

   ```java
   @Bean
   public MyHealthIndicator myHealthIndicator(){
     return new MyHealthIndicator();
   }	
   ```

3. 关闭安全控制

   ```properties
   management.security.enabled = false
   ```

   



#### 其他内容



REST API = /users , /withdraw

HATEOAS =  REST 服务器发现的入口，类似 UDDI (Universal Description Discovery and Integration)

HAL

/users
/withdraw
...



Spring Boot 激活 `actuator` 需要增加 Hateoas 的依赖：

```xml
<dependency>
  <groupId>org.springframework.hateoas</groupId>
  <artifactId>spring-hateoas</artifactId>
</dependency>
```

以客户端为例：

```json
{
    "links": [{
        "rel": "self",
        "href": "http://localhost:8080/actuator"
    }, {
        "rel": "heapdump",
        "href": "http://localhost:8080/heapdump"
    }, {
        "rel": "beans",
        "href": "http://localhost:8080/beans"
    }, {
        "rel": "resume",
        "href": "http://localhost:8080/resume"
    }, {
        "rel": "autoconfig",
        "href": "http://localhost:8080/autoconfig"
    }, {
        "rel": "refresh",
        "href": "http://localhost:8080/refresh"
    }, {
        "rel": "env",
        "href": "http://localhost:8080/env"
    }, {
        "rel": "auditevents",
        "href": "http://localhost:8080/auditevents"
    }, {
        "rel": "mappings",
        "href": "http://localhost:8080/mappings"
    }, {
        "rel": "info",
        "href": "http://localhost:8080/info"
    }, {
        "rel": "dump",
        "href": "http://localhost:8080/dump"
    }, {
        "rel": "loggers",
        "href": "http://localhost:8080/loggers"
    }, {
        "rel": "restart",
        "href": "http://localhost:8080/restart"
    }, {
        "rel": "metrics",
        "href": "http://localhost:8080/metrics"
    }, {
        "rel": "health",
        "href": "http://localhost:8080/health"
    }, {
        "rel": "configprops",
        "href": "http://localhost:8080/configprops"
    }, {
        "rel": "pause",
        "href": "http://localhost:8080/pause"
    }, {
        "rel": "features",
        "href": "http://localhost:8080/features"
    }, {
        "rel": "trace",
        "href": "http://localhost:8080/trace"
    }]
}
```



## 问答

1. 小马哥，你们服务是基于啥原因采用的springboot 的， 这么多稳定性的问题？

   答：Spring Boot 业界比较稳定的微服务中间件，不过它使用是易学难精！

2. 小马哥 为什么要把配置项放到 git上，为什么不放到具体服务的的程序里边 ；git在这里扮演什么样的角色 ;是不是和 zookeeper 一样

   答：Git 文件存储方式、分布式的管理系统，Spring Cloud 官方实现基于 Git，它达到的理念和 ZK 一样。

3. 一个DB配置相关的bean用@RefreshScope修饰时，config service修改了db的配置，比如mysql的url，那么这个Bean会不会刷新？如果刷新了是不是获取新的连接的时候url就变了？

   如果发生了配置变更，我的解决方案是重启 Spring Context。@RefreshScope 最佳实践用于 配置Bean，比如：开关、阈值、文案等等

   A B C
   1 1 1

   A* B C
   0  1 1

   A* B* C
   1  0  1

   A* B* C
   1  1  0

   A* B* C*
   1  1  1

4. 如果这样是不是动态刷新就没啥用了吧

   答：不能一概而论，@RefreshScope 开关、阈值、文案等等场景使用比较多



通过/xxx.properties访问配置文件（xxx-default）

/refresh刷新配置

## java中的配置

### 字符类型配置

- 通用

  - java系统属性：java.lang.System#getProperty(java.lang.String)

    ```java
    package configuration;
    
    public class GenericConfiguration {
        public static void main(String[] args) {
            System.out.println(System.getProperty("user.home"));
            System.out.println(System.getProperty("user.age"));
            //不加入需加入jdk参数：-Duser.age=25，返回0
            System.out.println(Integer.getInteger("user.age",0));
        }
    }
    
    ```

  - OS环境变量：java.lang.System#getenv()

    ```java
    package configuration;
    
    import java.util.Map;
    
    public class GenericConfiguration {
        public static void main(String[] args) {
            Map<String, String> getenv = System.getenv();
            System.out.println(getenv);
            System.out.println(System.getenv("JAVA8_HOME"));
        }
    }
    
    ```

- 特别

  - XML（JDK API可处理）

    - spring dom api
      - org.w3c.dom.Document
    - JAXB（Spring XML系列化）
    - SAX（Simple APl for XML）
    - XML Stream
    - xStream

  - .ini（JDK API可处理）

  - Properties（JDK API可处理）

    - key-value配置

    - XML配置

      ```java
      package configuration;
      
      import java.io.IOException;
      import java.util.Properties;
      
      public class PropertiesConfiguration {
      
          public static void main(String[] args) throws IOException {
              Properties properties = new Properties();
              properties.setProperty("name","jack");
              properties.setProperty("age","30");
              properties.storeToXML(System.out,"UTF-8");
          }
      }
      
      ```

      ```xml
      <?xml version="1.0" encoding="UTF-8" standalone="no"?>
      <!DOCTYPE properties SYSTEM "http://java.sun.com/dtd/properties.dtd">
      <properties>
      <comment>UTF-8</comment>
      <entry key="age">30</entry>
      <entry key="name">jack</entry>
      </properties>
      ```

      - springboot解决不了中文乱码问题

  - JSON（第三方处理）

  - YAML（第三方）

  - CSV（自定义、第三方）

  - Hard Code(硬编码)

### 非字符类型配置拓展

- 通用

  - Integer

    ```java
    package configuration;
    
    
    public class GenericConfiguration {
        public static void main(String[] args) {
          	//不加入需加入jdk参数：-Duser.age=25，返回0
            System.out.println(Integer.getInteger("user.age",0));
        }
    }
    
    ```

    输出：

    `25`

    通过查看源码，其实是通过System.getProperties获取的参数

    ```java
        public static Integer getInteger(String nm, Integer val) {
            String v = null;
            try {
                v = System.getProperty(nm);
            } catch (IllegalArgumentException | NullPointerException e) {
            }
            if (v != null) {
                try {
                    return Integer.decode(v);
                } catch (NumberFormatException e) {
                }
            }
            return val;
        }
    ```

  - Boolean

  - Long

> 结论：JDK中提供底层配置源的存储方式，没有具体抽象配置API，仅提供了一些零散的配置类型信息。





## Apache Commons Configuration

### API

apache common通用工程，对JDK API做了一定的补充

- commons-lang StringUtils 
- commons-collection CollectionUtils 
- commons-dbcp BasicDataSource 

commons-configure提供了统一配置API，提供数据类型转换：Integer、Long、BigDecimel



### 核心接口

- org.apache.commons.configuration.Configuration 

  - 关系型数据库-org.apache.commons.configuration.DatabaseConfiguration 

  - Properties配置文件-org.apache.commons.configuration.PropertiesConfiguratio
    n 

  - XML配置-org.apache.commons.configuration2.XMLConfiguration

  - OS环境配置-org.apache.commons.configuration2.EnvironmentConfiguration

    ```java
    package configuration;
    
    
    import org.apache.commons.configuration2.Configuration;
    import org.apache.commons.configuration2.EnvironmentConfiguration;
    
    public class Test {
        public static void main(String[] args) {
            Configuration envConfig = new EnvironmentConfiguration();
            System.out.println("JAVA_HOME=" + envConfig.getString("JAVA_HOME"));
        }
    }
    
    ```
    构造方法：

    ```java
        /**
         * Create a Configuration based on the environment variables.
         *
         * @see System#getenv()
         */
        public EnvironmentConfiguration()
        {
            super(new HashMap<String, Object>(System.getenv()));
        }
    ```

    最终getString方法通过调用org.apache.commons.configuration2.MapConfiguration#getPropertyInternal获得env系统环境变量

    ```java
        @Override
        protected Object getPropertyInternal(final String key)
        {
            final Object value = map.get(key);
            if (value instanceof String)
            {
                final Collection<String> list = getListDelimiterHandler().split((String) value, !isTrimmingDisabled());
                return list.size() > 1 ? list : list.iterator().next();
            }
            return value;
        }
    ```

  - java System Prooerties-org.apache.commons.configuration.SystemConfiguration

    - 实现类似EnvironmentConfiguration

  - 特殊组合实现-org.apache.commons.configuration.CompositeConfiguratio
    n 

    - 其本身不提供任何数据源，组合其他来源，并且控制优先级顺序

      ```java
      public class CompositeConfiguration extends AbstractConfiguration
      implements Cloneable
      {
          /** List holding all the configuration */
          private List<Configuration> configList = new LinkedList<>();
          ...
      }    
      ```

    - 举例

      - CompositeConfiguration ( List 有序)

        - DatabaseConfiguration
          - user.age = 34
        - PropertiesConfiguration
          - user.age = 30
        - XMLConfiguration
        - EnvironmentConfiguration
          - user.age = 19
          - SystemConfiguration 

      - CompositeConfiguration -> Configuration.getInteger("user.age") ->两种结果

        - 19
        - 34

        查看org.apache.commons.configuration2.CompositeConfiguration#getPropertyInternal可知道结论，先插入的Configuration优先。

> 以上事例均由外部化配置提供来源



### 提供的机制

- 读取配置源
- 合并配置源
- 定义顺序（CompositeConfiguration）

> 结论：Apache commons-configuration 提供一套完整配置 API，并且能够通过组合模式组合多种不同的配置来源，来实现统一的配置读取，不支持动态更新和写入 



## Netfix Archaius

netfix archaius拓展了Apache Common Configuration，实现动态配置更新和写入操作。

### 核心API

- com.netflix.config.DynamicConfiguration 

> 结论：弥补了Apache Common Configuration的不足，不过尚未提供类型转换的API



## Spring FrameWork Enviroment抽象

### Enviroment接口

#### 语义

- 存储配置
- 存储profile
- 转换类型



#### API

- 只读API-org.springframework.core.env.Environment 

  - PropertyResolver
  - 读取profile

  ```java
  /*
   * Copyright 2002-2018 the original author or authors.
   *
   * Licensed under the Apache License, Version 2.0 (the "License");
   * you may not use this file except in compliance with the License.
   * You may obtain a copy of the License at
   *
   *      https://www.apache.org/licenses/LICENSE-2.0
   *
   * Unless required by applicable law or agreed to in writing, software
   * distributed under the License is distributed on an "AS IS" BASIS,
   * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   * See the License for the specific language governing permissions and
   * limitations under the License.
   */
  
  package org.springframework.core.env;
  
  /**
   * Interface representing the environment in which the current application is running.
   * Models two key aspects of the application environment: <em>profiles</em> and
   * <em>properties</em>. Methods related to property access are exposed via the
   * {@link PropertyResolver} superinterface.
   *
   * <p>A <em>profile</em> is a named, logical group of bean definitions to be registered
   * with the container only if the given profile is <em>active</em>. Beans may be assigned
   * to a profile whether defined in XML or via annotations; see the spring-beans 3.1 schema
   * or the {@link org.springframework.context.annotation.Profile @Profile} annotation for
   * syntax details. The role of the {@code Environment} object with relation to profiles is
   * in determining which profiles (if any) are currently {@linkplain #getActiveProfiles
   * active}, and which profiles (if any) should be {@linkplain #getDefaultProfiles active
   * by default}.
   *
   * <p><em>Properties</em> play an important role in almost all applications, and may
   * originate from a variety of sources: properties files, JVM system properties, system
   * environment variables, JNDI, servlet context parameters, ad-hoc Properties objects,
   * Maps, and so on. The role of the environment object with relation to properties is to
   * provide the user with a convenient service interface for configuring property sources
   * and resolving properties from them.
   *
   * <p>Beans managed within an {@code ApplicationContext} may register to be {@link
   * org.springframework.context.EnvironmentAware EnvironmentAware} or {@code @Inject} the
   * {@code Environment} in order to query profile state or resolve properties directly.
   *
   * <p>In most cases, however, application-level beans should not need to interact with the
   * {@code Environment} directly but instead may have to have {@code ${...}} property
   * values replaced by a property placeholder configurer such as
   * {@link org.springframework.context.support.PropertySourcesPlaceholderConfigurer
   * PropertySourcesPlaceholderConfigurer}, which itself is {@code EnvironmentAware} and
   * as of Spring 3.1 is registered by default when using
   * {@code <context:property-placeholder/>}.
   *
   * <p>Configuration of the environment object must be done through the
   * {@code ConfigurableEnvironment} interface, returned from all
   * {@code AbstractApplicationContext} subclass {@code getEnvironment()} methods. See
   * {@link ConfigurableEnvironment} Javadoc for usage examples demonstrating manipulation
   * of property sources prior to application context {@code refresh()}.
   *
   * @author Chris Beams
   * @since 3.1
   * @see PropertyResolver
   * @see EnvironmentCapable
   * @see ConfigurableEnvironment
   * @see AbstractEnvironment
   * @see StandardEnvironment
   * @see org.springframework.context.EnvironmentAware
   * @see org.springframework.context.ConfigurableApplicationContext#getEnvironment
   * @see org.springframework.context.ConfigurableApplicationContext#setEnvironment
   * @see org.springframework.context.support.AbstractApplicationContext#createEnvironment
   */
  public interface Environment extends PropertyResolver {
  
  	/**
  	 * Return the set of profiles explicitly made active for this environment. Profiles
  	 * are used for creating logical groupings of bean definitions to be registered
  	 * conditionally, for example based on deployment environment. Profiles can be
  	 * activated by setting {@linkplain AbstractEnvironment#ACTIVE_PROFILES_PROPERTY_NAME
  	 * "spring.profiles.active"} as a system property or by calling
  	 * {@link ConfigurableEnvironment#setActiveProfiles(String...)}.
  	 * <p>If no profiles have explicitly been specified as active, then any
  	 * {@linkplain #getDefaultProfiles() default profiles} will automatically be activated.
  	 * @see #getDefaultProfiles
  	 * @see ConfigurableEnvironment#setActiveProfiles
  	 * @see AbstractEnvironment#ACTIVE_PROFILES_PROPERTY_NAME
  	 */
  	String[] getActiveProfiles();
  
  	/**
  	 * Return the set of profiles to be active by default when no active profiles have
  	 * been set explicitly.
  	 * @see #getActiveProfiles
  	 * @see ConfigurableEnvironment#setDefaultProfiles
  	 * @see AbstractEnvironment#DEFAULT_PROFILES_PROPERTY_NAME
  	 */
  	String[] getDefaultProfiles();
  
  	/**
  	 * Return whether one or more of the given profiles is active or, in the case of no
  	 * explicit active profiles, whether one or more of the given profiles is included in
  	 * the set of default profiles. If a profile begins with '!' the logic is inverted,
  	 * i.e. the method will return {@code true} if the given profile is <em>not</em> active.
  	 * For example, {@code env.acceptsProfiles("p1", "!p2")} will return {@code true} if
  	 * profile 'p1' is active or 'p2' is not active.
  	 * @throws IllegalArgumentException if called with zero arguments
  	 * or if any profile is {@code null}, empty, or whitespace only
  	 * @see #getActiveProfiles
  	 * @see #getDefaultProfiles
  	 * @see #acceptsProfiles(Profiles)
  	 * @deprecated as of 5.1 in favor of {@link #acceptsProfiles(Profiles)}
  	 */
  	@Deprecated
  	boolean acceptsProfiles(String... profiles);
  
  	/**
  	 * Return whether the {@linkplain #getActiveProfiles() active profiles}
  	 * match the given {@link Profiles} predicate.
  	 */
  	boolean acceptsProfiles(Profiles profiles);
  
  }
  
  ```

- 写入API-org.springframework.core.env.ConfigurableEnvironment

  - ConfigurablePropertyResolver
  - 存取profile
  - 获取配置源-org.springframework.core.env.ConfigurableEnvironment#getPropertySources

  ```java
  /*
   * Copyright 2002-2018 the original author or authors.
   *
   * Licensed under the Apache License, Version 2.0 (the "License");
   * you may not use this file except in compliance with the License.
   * You may obtain a copy of the License at
   *
   *      https://www.apache.org/licenses/LICENSE-2.0
   *
   * Unless required by applicable law or agreed to in writing, software
   * distributed under the License is distributed on an "AS IS" BASIS,
   * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   * See the License for the specific language governing permissions and
   * limitations under the License.
   */
  
  package org.springframework.core.env;
  
  import java.util.Map;
  
  /**
   * Configuration interface to be implemented by most if not all {@link Environment} types.
   * Provides facilities for setting active and default profiles and manipulating underlying
   * property sources. Allows clients to set and validate required properties, customize the
   * conversion service and more through the {@link ConfigurablePropertyResolver}
   * superinterface.
   *
   * <h2>Manipulating property sources</h2>
   * <p>Property sources may be removed, reordered, or replaced; and additional
   * property sources may be added using the {@link MutablePropertySources}
   * instance returned from {@link #getPropertySources()}. The following examples
   * are against the {@link StandardEnvironment} implementation of
   * {@code ConfigurableEnvironment}, but are generally applicable to any implementation,
   * though particular default property sources may differ.
   *
   * <h4>Example: adding a new property source with highest search priority</h4>
   * <pre class="code">
   * ConfigurableEnvironment environment = new StandardEnvironment();
   * MutablePropertySources propertySources = environment.getPropertySources();
   * Map&lt;String, String&gt; myMap = new HashMap&lt;&gt;();
   * myMap.put("xyz", "myValue");
   * propertySources.addFirst(new MapPropertySource("MY_MAP", myMap));
   * </pre>
   *
   * <h4>Example: removing the default system properties property source</h4>
   * <pre class="code">
   * MutablePropertySources propertySources = environment.getPropertySources();
   * propertySources.remove(StandardEnvironment.SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME)
   * </pre>
   *
   * <h4>Example: mocking the system environment for testing purposes</h4>
   * <pre class="code">
   * MutablePropertySources propertySources = environment.getPropertySources();
   * MockPropertySource mockEnvVars = new MockPropertySource().withProperty("xyz", "myValue");
   * propertySources.replace(StandardEnvironment.SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, mockEnvVars);
   * </pre>
   *
   * When an {@link Environment} is being used by an {@code ApplicationContext}, it is
   * important that any such {@code PropertySource} manipulations be performed
   * <em>before</em> the context's {@link
   * org.springframework.context.support.AbstractApplicationContext#refresh() refresh()}
   * method is called. This ensures that all property sources are available during the
   * container bootstrap process, including use by {@linkplain
   * org.springframework.context.support.PropertySourcesPlaceholderConfigurer property
   * placeholder configurers}.
   *
   * @author Chris Beams
   * @since 3.1
   * @see StandardEnvironment
   * @see org.springframework.context.ConfigurableApplicationContext#getEnvironment
   */
  public interface ConfigurableEnvironment extends Environment, ConfigurablePropertyResolver {
  
  	/**
  	 * Specify the set of profiles active for this {@code Environment}. Profiles are
  	 * evaluated during container bootstrap to determine whether bean definitions
  	 * should be registered with the container.
  	 * <p>Any existing active profiles will be replaced with the given arguments; call
  	 * with zero arguments to clear the current set of active profiles. Use
  	 * {@link #addActiveProfile} to add a profile while preserving the existing set.
  	 * @throws IllegalArgumentException if any profile is null, empty or whitespace-only
  	 * @see #addActiveProfile
  	 * @see #setDefaultProfiles
  	 * @see org.springframework.context.annotation.Profile
  	 * @see AbstractEnvironment#ACTIVE_PROFILES_PROPERTY_NAME
  	 */
  	void setActiveProfiles(String... profiles);
  
  	/**
  	 * Add a profile to the current set of active profiles.
  	 * @throws IllegalArgumentException if the profile is null, empty or whitespace-only
  	 * @see #setActiveProfiles
  	 */
  	void addActiveProfile(String profile);
  
  	/**
  	 * Specify the set of profiles to be made active by default if no other profiles
  	 * are explicitly made active through {@link #setActiveProfiles}.
  	 * @throws IllegalArgumentException if any profile is null, empty or whitespace-only
  	 * @see AbstractEnvironment#DEFAULT_PROFILES_PROPERTY_NAME
  	 */
  	void setDefaultProfiles(String... profiles);
  
  	/**
  	 * Return the {@link PropertySources} for this {@code Environment} in mutable form,
  	 * allowing for manipulation of the set of {@link PropertySource} objects that should
  	 * be searched when resolving properties against this {@code Environment} object.
  	 * The various {@link MutablePropertySources} methods such as
  	 * {@link MutablePropertySources#addFirst addFirst},
  	 * {@link MutablePropertySources#addLast addLast},
  	 * {@link MutablePropertySources#addBefore addBefore} and
  	 * {@link MutablePropertySources#addAfter addAfter} allow for fine-grained control
  	 * over property source ordering. This is useful, for example, in ensuring that
  	 * certain user-defined property sources have search precedence over default property
  	 * sources such as the set of system properties or the set of system environment
  	 * variables.
  	 * @see AbstractEnvironment#customizePropertySources
  	 */
  	MutablePropertySources getPropertySources();
  
  	/**
  	 * Return the value of {@link System#getProperties()} if allowed by the current
  	 * {@link SecurityManager}, otherwise return a map implementation that will attempt
  	 * to access individual keys using calls to {@link System#getProperty(String)}.
  	 * <p>Note that most {@code Environment} implementations will include this system
  	 * properties map as a default {@link PropertySource} to be searched. Therefore, it is
  	 * recommended that this method not be used directly unless bypassing other property
  	 * sources is expressly intended.
  	 * <p>Calls to {@link Map#get(Object)} on the Map returned will never throw
  	 * {@link IllegalAccessException}; in cases where the SecurityManager forbids access
  	 * to a property, {@code null} will be returned and an INFO-level log message will be
  	 * issued noting the exception.
  	 */
  	Map<String, Object> getSystemProperties();
  
  	/**
  	 * Return the value of {@link System#getenv()} if allowed by the current
  	 * {@link SecurityManager}, otherwise return a map implementation that will attempt
  	 * to access individual keys using calls to {@link System#getenv(String)}.
  	 * <p>Note that most {@link Environment} implementations will include this system
  	 * environment map as a default {@link PropertySource} to be searched. Therefore, it
  	 * is recommended that this method not be used directly unless bypassing other
  	 * property sources is expressly intended.
  	 * <p>Calls to {@link Map#get(Object)} on the Map returned will never throw
  	 * {@link IllegalAccessException}; in cases where the SecurityManager forbids access
  	 * to a property, {@code null} will be returned and an INFO-level log message will be
  	 * issued noting the exception.
  	 */
  	Map<String, Object> getSystemEnvironment();
  
  	/**
  	 * Append the given parent environment's active profiles, default profiles and
  	 * property sources to this (child) environment's respective collections of each.
  	 * <p>For any identically-named {@code PropertySource} instance existing in both
  	 * parent and child, the child instance is to be preserved and the parent instance
  	 * discarded. This has the effect of allowing overriding of property sources by the
  	 * child as well as avoiding redundant searches through common property source types,
  	 * e.g. system environment and system properties.
  	 * <p>Active and default profile names are also filtered for duplicates, to avoid
  	 * confusion and redundant storage.
  	 * <p>The parent environment remains unmodified in any case. Note that any changes to
  	 * the parent environment occurring after the call to {@code merge} will not be
  	 * reflected in the child. Therefore, care should be taken to configure parent
  	 * property sources and profile information prior to calling {@code merge}.
  	 * @param parent the environment to merge with
  	 * @since 3.1.2
  	 * @see org.springframework.context.support.AbstractApplicationContext#setParent
  	 */
  	void merge(ConfigurableEnvironment parent);
  
  }
  
  ```

  增加`MutablePropertySources`例子：

  ```java
  /*
   * Copyright 2002-2019 the original author or authors.
   *
   * Licensed under the Apache License, Version 2.0 (the "License");
   * you may not use this file except in compliance with the License.
   * You may obtain a copy of the License at
   *
   *      https://www.apache.org/licenses/LICENSE-2.0
   *
   * Unless required by applicable law or agreed to in writing, software
   * distributed under the License is distributed on an "AS IS" BASIS,
   * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   * See the License for the specific language governing permissions and
   * limitations under the License.
   */
  
  package org.springframework.core.env;
  
  /**
   * {@link Environment} implementation suitable for use in 'standard' (i.e. non-web)
   * applications.
   *
   * <p>In addition to the usual functions of a {@link ConfigurableEnvironment} such as
   * property resolution and profile-related operations, this implementation configures two
   * default property sources, to be searched in the following order:
   * <ul>
   * <li>{@linkplain AbstractEnvironment#getSystemProperties() system properties}
   * <li>{@linkplain AbstractEnvironment#getSystemEnvironment() system environment variables}
   * </ul>
   *
   * That is, if the key "xyz" is present both in the JVM system properties as well as in
   * the set of environment variables for the current process, the value of key "xyz" from
   * system properties will return from a call to {@code environment.getProperty("xyz")}.
   * This ordering is chosen by default because system properties are per-JVM, while
   * environment variables may be the same across many JVMs on a given system.  Giving
   * system properties precedence allows for overriding of environment variables on a
   * per-JVM basis.
   *
   * <p>These default property sources may be removed, reordered, or replaced; and
   * additional property sources may be added using the {@link MutablePropertySources}
   * instance available from {@link #getPropertySources()}. See
   * {@link ConfigurableEnvironment} Javadoc for usage examples.
   *
   * <p>See {@link SystemEnvironmentPropertySource} javadoc for details on special handling
   * of property names in shell environments (e.g. Bash) that disallow period characters in
   * variable names.
   *
   * @author Chris Beams
   * @since 3.1
   * @see ConfigurableEnvironment
   * @see SystemEnvironmentPropertySource
   * @see org.springframework.web.context.support.StandardServletEnvironment
   */
  public class StandardEnvironment extends AbstractEnvironment {
  
  	/** System environment property source name: {@value}. */
  	public static final String SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME = "systemEnvironment";
  
  	/** JVM system properties property source name: {@value}. */
  	public static final String SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME = "systemProperties";
  
  
  	/**
  	 * Customize the set of property sources with those appropriate for any standard
  	 * Java environment:
  	 * <ul>
  	 * <li>{@value #SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME}
  	 * <li>{@value #SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME}
  	 * </ul>
  	 * <p>Properties present in {@value #SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME} will
  	 * take precedence over those in {@value #SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME}.
  	 * @see AbstractEnvironment#customizePropertySources(MutablePropertySources)
  	 * @see #getSystemProperties()
  	 * @see #getSystemEnvironment()
  	 */
  	@Override
  	protected void customizePropertySources(MutablePropertySources propertySources) {
  		propertySources.addLast(
  				new PropertiesPropertySource(SYSTEM_PROPERTIES_PROPERTY_SOURCE_NAME, getSystemProperties()));
  		propertySources.addLast(
  				new SystemEnvironmentPropertySource(SYSTEM_ENVIRONMENT_PROPERTY_SOURCE_NAME, getSystemEnvironment()));
  	}
  
  }
  
  ```

  

### 配置源

#### API

- 单配置源-org.springframework.core.env.PropertySource

  ```java
  public abstract class PropertySource<T> {
  
  	protected final Log logger = LogFactory.getLog(getClass());
  
  	protected final String name;
  
  	protected final T source;
  
  
  	/**
  	 * Create a new {@code PropertySource} with the given name and source object.
  	 */
  	public PropertySource(String name, T source) {
  		Assert.hasText(name, "Property source name must contain at least one character");
  		Assert.notNull(source, "Property source must not be null");
  		this.name = name;
  		this.source = source;
  	}
  	...
  }
  ```

  例如：

  ```java
          PropertySource source = new PropertiesPropertySource("java_system",System.getProperties());
          System.out.println(source.getProperty("file.encoding"));
  ```

- 多配置源-org.springframework.core.env.PropertySources

  ```java
  /*
   * Copyright 2002-2018 the original author or authors.
   *
   * Licensed under the Apache License, Version 2.0 (the "License");
   * you may not use this file except in compliance with the License.
   * You may obtain a copy of the License at
   *
   *      https://www.apache.org/licenses/LICENSE-2.0
   *
   * Unless required by applicable law or agreed to in writing, software
   * distributed under the License is distributed on an "AS IS" BASIS,
   * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   * See the License for the specific language governing permissions and
   * limitations under the License.
   */
  
  package org.springframework.core.env;
  
  import java.util.stream.Stream;
  import java.util.stream.StreamSupport;
  
  import org.springframework.lang.Nullable;
  
  /**
   * Holder containing one or more {@link PropertySource} objects.
   *
   * @author Chris Beams
   * @author Juergen Hoeller
   * @since 3.1
   * @see PropertySource
   */
  public interface PropertySources extends Iterable<PropertySource<?>> {
  
  	/**
  	 * Return a sequential {@link Stream} containing the property sources.
  	 * @since 5.1
  	 */
  	default Stream<PropertySource<?>> stream() {
  		return StreamSupport.stream(spliterator(), false);
  	}
  
  	/**
  	 * Return whether a property source with the given name is contained.
  	 * @param name the {@linkplain PropertySource#getName() name of the property source} to find
  	 */
  	boolean contains(String name);
  
  	/**
  	 * Return the property source with the given name, {@code null} if not found.
  	 * @param name the {@linkplain PropertySource#getName() name of the property source} to find
  	 */
  	@Nullable
  	PropertySource<?> get(String name);
  
  }
  
  ```

  - 实现类-org.springframework.core.env.MutablePropertySources：存取多配置源

    - 存取容器

      ```java
      	private final List<PropertySource<?>> propertySourceList = new CopyOnWriteArrayList<>();
      
      ```

      `CopyOnWriteArrayList`是一个线程安全的列表，即写时操作的是新复制的一份数据

    - 支持增加配置源方式

      - org.springframework.core.env.MutablePropertySources#addFirst
      - addLast
      - addBefore
      - addAfter

#### 注解

- 单配置源-@org.springframework.context.annotation.PropertySource
- 多配置源-@org.springframework.context.annotation.PropertySources



### Spring profile

profile用于指定Enviroment的界限，如dev的Enviroment、production的profile，同时可以给Bean指定一个profile，用于不同的环境

```java
@Configuration
public class AppConfig {

    @Bean("dataSource")
    @Profile("development") 
    public DataSource standaloneDataSource() {
        return new EmbeddedDatabaseBuilder()
            .setType(EmbeddedDatabaseType.HSQL)
            .addScript("classpath:com/bank/config/sql/schema.sql")
            .addScript("classpath:com/bank/config/sql/test-data.sql")
            .build();
    }

    @Bean("dataSource")
    @Profile("production") 
    public DataSource jndiDataSource() throws Exception {
        Context ctx = new InitialContext();
        return (DataSource) ctx.lookup("java:comp/env/jdbc/datasource");
    }
}
```

#### API

- 读
  - org.springframework.core.env.AbstractEnvironment#getActiveProfiles 
  - org.springframework.core.env.AbstractEnvironment#getDefaultProfiles（默认）
- 写
  - org.springframework.core.env.AbstractEnvironment#setActiveProfiles
  - org.springframework.core.env.AbstractEnvironment#setDefaultProfiles  

### 类型转换

PropertyResolver对属性的解析就是通过类型转换类去实现的

#### API

- org.springframework.core.convert.ConversionService

  如org.springframework.core.env.AbstractPropertyResolver#convertValueIfNecessary：

  ```java
  	/**
  	 * Convert the given value to the specified target type, if necessary.
  	 * @param value the original property value
  	 * @param targetType the specified target type for property retrieval
  	 * @return the converted value, or the original value if no conversion
  	 * is necessary
  	 * @since 4.3.5
  	 */
  	@SuppressWarnings("unchecked")
  	@Nullable
  	protected <T> T convertValueIfNecessary(Object value, @Nullable Class<T> targetType) {
  		if (targetType == null) {
  			return (T) value;
  		}
  		ConversionService conversionServiceToUse = this.conversionService;
  		if (conversionServiceToUse == null) {
  			// Avoid initialization of shared DefaultConversionService if
  			// no standard type conversion is needed in the first place...
  			if (ClassUtils.isAssignableValue(targetType, value)) {
  				return (T) value;
  			}
  			conversionServiceToUse = DefaultConversionService.getSharedInstance();
  		}
  		return conversionServiceToUse.convert(value, targetType);
  	}
  ```

- org.springframework.core.convert.converter.Converter 

  Converter之声明了一个方法，用于实现类实现转换的逻辑

  ```java
  /*
   * Copyright 2002-2016 the original author or authors.
   *
   * Licensed under the Apache License, Version 2.0 (the "License");
   * you may not use this file except in compliance with the License.
   * You may obtain a copy of the License at
   *
   *      https://www.apache.org/licenses/LICENSE-2.0
   *
   * Unless required by applicable law or agreed to in writing, software
   * distributed under the License is distributed on an "AS IS" BASIS,
   * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   * See the License for the specific language governing permissions and
   * limitations under the License.
   */
  
  package org.springframework.core.convert.converter;
  
  import org.springframework.lang.Nullable;
  
  /**
   * A converter converts a source object of type {@code S} to a target of type {@code T}.
   *
   * <p>Implementations of this interface are thread-safe and can be shared.
   *
   * <p>Implementations may additionally implement {@link ConditionalConverter}.
   *
   * @author Keith Donald
   * @since 3.0
   * @param <S> the source type
   * @param <T> the target type
   */
  @FunctionalInterface
  public interface Converter<S, T> {
  
  	/**
  	 * Convert the source object of type {@code S} to target type {@code T}.
  	 * @param source the source object to convert, which must be an instance of {@code S} (never {@code null})
  	 * @return the converted object, which must be an instance of {@code T} (potentially {@code null})
  	 * @throws IllegalArgumentException if the source cannot be converted to the desired target type
  	 */
  	@Nullable
  	T convert(S source);
  
  }
  
  ```

  如org.springframework.core.convert.support.EnumToIntegerConverter实现类：

  ```java
  /*
   * Copyright 2002-2016 the original author or authors.
   *
   * Licensed under the Apache License, Version 2.0 (the "License");
   * you may not use this file except in compliance with the License.
   * You may obtain a copy of the License at
   *
   *      https://www.apache.org/licenses/LICENSE-2.0
   *
   * Unless required by applicable law or agreed to in writing, software
   * distributed under the License is distributed on an "AS IS" BASIS,
   * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   * See the License for the specific language governing permissions and
   * limitations under the License.
   */
  
  package org.springframework.core.convert.support;
  
  import org.springframework.core.convert.ConversionService;
  import org.springframework.core.convert.converter.Converter;
  
  /**
   * Calls {@link Enum#ordinal()} to convert a source Enum to a Integer.
   * This converter will not match enums with interfaces that can be converted.
   *
   * @author Yanming Zhou
   * @since 4.3
   */
  final class EnumToIntegerConverter extends AbstractConditionalEnumConverter implements Converter<Enum<?>, Integer> {
  
  	public EnumToIntegerConverter(ConversionService conversionService) {
  		super(conversionService);
  	}
  
  	@Override
  	public Integer convert(Enum<?> source) {
  		return source.ordinal();
  	}
  
  }
  
  ```



## Spring Boot配置实现

### 配置来源

核心实现-org.springframework.boot.context.config.ConfigFileApplicationListener 



## Spring Cloud配置实现

### 配置来源

核心实现-org.springframework.cloud.bootstrap.BootstrapApplicationListener 



怎么保证外部化配置比本地配置优先：

BootstrapApplicationListene 

org.springframework.boot.context.config.ConfigFileApplicationListener



# 服务注册与发现



## 服务发现协议

通过服务发现协议的实现可以实现自动发现设备或者服务，常见协议：

- Java：JINI（Apache River）
- REST：HATEOAS
- Web Service：UDDI（Universal Description DiscoDiscover and Integration）



## 服务注册

为了治理多个设备或者服务，这些设备或者服务主动或者被动注册到管理中心，服务的发现和消费通过调用注册中心去实现，常用服务注册中心：

- Apache Zookeeper
- Netflix Eureka
- Consul

注册中心有两种模型：

- 中心化：服务注册到单个节点中，服务的注册与发现通过其实现
- 去中心化：每个节点都是注册中心

![1570193856565](springcloud.assets\1570193856565.png)



## Spring Cloud Netflix Eureka

eureka是有netflix公司发明的一个服务发现中间件，包括服务发现服务端和客户端

### Eureka Server

eureka server是eureka client的**注册服务中心**，它用于管理所有注册服务、以及其实例信息和状态

#### 使用

依赖：

```properties
<dependency>
	<groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-server</artifactId>
</dependency>
```

配置：

- `org.springframework.cloud.netflix.eureka.EurekaClientConfigBean`

```properties
spring:
  application:
    name: eureka-server

server:
  port: 8080

#eureka服务端配置，提供给客户端用于注册服务
eureka:
  client:
    service-url:
      defaultZone: http://127.0.0.1:${server.port}/eureka
    registryFetchIntervalSeconds: 5 # 5 秒轮训一次


```

激活： `@EnableEurekaServer`

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.netflix.eureka.server.EnableEurekaServer;

public class EurekaServerStarter {

    @SpringBootApplication
    @EnableEurekaServer
    public static class EurekaServerConfiguration{

    }

    public static void main(String[] args) {
        SpringApplication.run(EurekaServerConfiguration.class,args);
    }
}

```

访问http://localhost:8080/是eureka服务端的管理界面，可以看到当前注册的服务相关信息

常用API：

![1583742775894](springcloud.assets\1583742775894.png)

- 列出所有服务信息（一个服务只一个应用）

  ```java
  http://localhost:8080/eureka/apps
  http://${host}:${port}/eureka/apps
  ```

- 列出某个应用信息（根据spring.application.name判断是否是同一应用）

  ```java
  http://${host}:${port}/eureka/apps/${application.nam}
  ```

- 列出某个应用实例信息

  ```java
  http://${host}:${port}/eureka/apps/EUREKA-SERVER/windows10.microdone.cn:eureka-server:8080
  http://${host}:${port}/eureka/apps/${application.nam}/${instance_id}
  ```

eureka缺点：

- 会把自己注册为一个服务(后面版本已经解决)
- 不能多注册中心，只有单个注册中心

- 随机端口时并不会显示真正的端口

  ![1570151324898](springcloud.assets\1570151324898.png)

- 注册列表达到5w时页面受不了



### Eureka Client

eureka client为当前服务提供注册、同步、查找服务以及其实例信息或状态等功能

#### 使用

依赖：

```properties
<!--eureka服务注册与发现客户端依赖-->
<dependency>
    <groupId>org.springframework.cloud</groupId>
    <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>
```

配置：

- `org.springframework.cloud.netflix.eureka.EurekaClientConfigBean`

```properties
--- #eureka profiles
spring:
  profiles: eureka

#eureka客户端配置
eureka:
  server:
    host: 127.0.0.1  #自定义配置
    port: 8080
  client:
    service-url:
      defaultZone: http://${eureka.server.host}:${eureka.server.port}/eureka
    enabled: true
```

通过设置program arguments--spring.profile.include指定profiles

#### 激活

- `@EnableDiscoveryClient`
- `@EnableEurekaClient`

```java
package com.example;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.xml.ws.Service;
import java.util.LinkedHashSet;
import java.util.LinkedList;
import java.util.List;
import java.util.Set;

//@EnableEurekaClient
//@SpringBootApplication
@EnableAutoConfiguration
@EnableDiscoveryClient
@RestController
public class EurekaClientBootstrap {

    @Autowired
    private DiscoveryClient discoveryClient;

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientBootstrap.class,args);
    }

}

```

### client端查看服务相关信息

核心接口：

- `DiscoveryClient`
  - `org.springframework.cloud.netflix.eureka.EurekaDiscoveryClient`
  - `org.springframework.cloud.zookeeper.discovery.ZookeeperDiscoveryClient`
  - `org.springframework.cloud.consul.discovery.ConsulDiscoveryClient`

```java
package com.example;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.EnableAutoConfiguration;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.ServiceInstance;
import org.springframework.cloud.client.discovery.DiscoveryClient;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.netflix.eureka.EnableEurekaClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.xml.ws.Service;
import java.util.LinkedHashSet;
import java.util.LinkedList;
import java.util.List;
import java.util.Set;

//@EnableEurekaClient
@SpringBootApplication
//@EnableAutoConfiguration
@EnableDiscoveryClient
@RestController
public class EurekaClientBootstrap {

    @Autowired
    private DiscoveryClient discoveryClient;

    public static void main(String[] args) {
        SpringApplication.run(EurekaClientBootstrap.class,args);
    }

    /**
     * 返回所有服务
     * @return
     */
    @GetMapping("/services")
    public Set<String> getService(){
        return new LinkedHashSet<>(discoveryClient.getServices());
    }

    /**
     * 返回某个服务的所有实例
     * @param serviceName
     * @return
     */
    @GetMapping("/services/{serviceName}")
    public List<ServiceInstance> getServiceInstances(
            @PathVariable(value = "serviceName") String serviceName){
        return new LinkedList<>(discoveryClient.getInstances(serviceName));
    }

    /**
     * 返回具体某个实例信息
     * @param serviceName
     * @param instanceId
     * @return
     */
    @GetMapping("/services/{serviceName}/{instanceId}")
    public ServiceInstance getInstance(@PathVariable(value = "serviceName") String serviceName,
                                       @PathVariable(value = "instanceId") String instanceId){
        return getServiceInstances(serviceName)
                .stream()
                .filter(serviceInstance -> instanceId.equals(serviceInstance.getInstanceId()))
                .findFirst()
                .orElseThrow(() -> new RuntimeException("No Suck Service Instance"));

    }
}

```



### Eureka中的Zone和Region设计

- 一个Region下可以有多个zone，包括availableZone和defaultZone
- 实例信息默认只会在同个region中共享
- 若有defaultZone，则优先请求同个zone下的实例



### 实战示例

#### eureka在线拓容

即通过springcloud-config的refresh刷新配置属性，在线不停应用修改defaultZone，代码示例见ch3-1-config-server,ch3-1-eureka-client,ch3-1-eureka-server



#### 构建Muti Zone Eureka Server

目的：演示多个zone下的eureka server，观察eureka client的注册情况，代码参考：ch3-2-config-server,ch3-2-eureka-client,ch3-2-zuul-gateway

- 查看不同zone下的client env：

![1586163783575](springcloud.assets\1586163783575.png)

![1586163768133](springcloud.assets\1586163768133.png)

- 通过测试gateway的client/actuator/env，访问的是client的/actuator/env接口，可以看出ZoneAffinity特性，即优先请求同个zone下的client。

gateway未测试成功，后面补上



#### 支持Remote Zone



### 客户端原理分析（太难了）

- eureka：轮询把所有的服务列表都加载进来，明显受内存限制，示例过多时内存可能会不够用
  - `org.springframework.cloud.netflix.eureka.EurekaDiscoveryClient`
  - com.netflix.discovery.DiscoveryClient#refreshRegistry
- zookeeper：监听服务节点的变化，被动更新列表的方式
  - `org.springframework.cloud.zookeeper.discovery.ZookeeperDiscoveryClient`
- consul：类似eureka，也是轮询，同时轮询比较频繁
  - `org.springframework.cloud.consul.discovery.ConsulDiscoveryClient`



## 其他注册中心实现

### zookeeper

依赖：

```properties
<!--zookeeper服务注册与发现客户端依赖-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
</dependency>
```

配置：

- `org.springframework.cloud.zookeeper.ZookeeperProperties`

```properties
--- #zookeeper profiles
spring:
  profiles: zookeeper
  cloud:
    zookeeper:
      enabled: true
      connect-string: 119.23.201.241:2181 #连接zookeeper
```

zk注册中心的效果：

```properties
[zk: 127.0.0.1:2181(CONNECTED) 0] ls /
[dubbo, services, zookeeper]
[zk: 127.0.0.1:2181(CONNECTED) 1] get /services/eureka-client/156ca38d-461e-4e4c-ba97-e1c8babe6b2c
{"name":"eureka-client","id":"156ca38d-461e-4e4c-ba97-e1c8babe6b2c","address":"windows10.microdone.cn","port":8091,"sslPort":null,"payload":{"@class":"org.springframework.cloud.zookeeper.discovery.ZookeeperInstance","id":"application-1","name":"eureka-client","metadata":{}},"registrationTimeUTC":1570186702159,"serviceType":"DYNAMIC","uriSpec":{"parts":[{"value":"scheme","variable":true},{"value":"://","variable":false},{"value":"address","variable":true},{"value":":","variable":false},{"value":"port","variable":true}]}}
[zk: 127.0.0.1:2181(CONNECTED) 2] ^C[root@izwz9c61wsgboaq9aoxis9z apache-zookeeper-3.5.5-bin]# ^C

```

```json
{
	"name": "eureka-client",
	"id": "156ca38d-461e-4e4c-ba97-e1c8babe6b2c",
	"address": "windows10.microdone.cn",
	"port": 8091,
	"sslPort": null,
	"payload": {
		"@class": "org.springframework.cloud.zookeeper.discovery.ZookeeperInstance",
		"id": "application-1",
		"name": "eureka-client",
		"metadata": {}
	},
	"registrationTimeUTC": 1570186702159,
	"serviceType": "DYNAMIC",
	"uriSpec": {
		"parts": [{
			"value": "scheme",
			"variable": true
		}, {
			"value": "://",
			"variable": false
		}, {
			"value": "address",
			"variable": true
		}, {
			"value": ":",
			"variable": false
		}, {
			"value": "port",
			"variable": true
		}]
	}
}
```



### consul

依赖：

```properties
<!-- Consul 服务发现与注册客户端依赖 -->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-consul-discovery</artifactId>
</dependency>
```

配置：

- `org.springframework.cloud.consul.discovery.ConsulDiscoveryProperties`

```properties
---  #consul profiles
spring:
  profiles: consul
  cloud:
    consul:
      discovery:
        enabled: true
        ip-address: 127.0.0.1
        port: 8500
      enabled: true
```

通过http://localhost:8500查看consul管理页面



## 多个注册中心比较

| 注册中心  | CAP特性 | 推荐规模（实例数） |
| --------- | :------ | ------------------ |
| eureka    | AP      | <30k               |
| zookeeper | CP      | <30k               |
| consul    | AP/CP   | <5k                |
| nocas     | AP/CP   | 100k-300k          |

- C：consistency，一致性，即各个节点的数据一致，各节点数据不一致之前不给请求返回结果
- A：Available，可用性，用户的请求能一直的到正常的返回结果，而不会出现系统访问失败、超时等情况
- P：Partition tolerance，分区容错性，某个节点出错时，不影响系统的整体使用。某个节点出错，意味着数据在有限时间内不能达到一致性。这时必须就C和A中做出选择

三者不可兼得，取两者：

- CA：优先保证一致性和可用性。意味着放弃系统的拓展性，而变为了非分布式系统
- AP：保证可用性和分区容灾，放弃强一致性，每个节点同一时刻的数据可能不一致
- CP：保证一致性和分区容灾，放弃可用性，即当数据不一致时，系统不保证提供可靠的服务，可能会返回错误，超时等

BASE理论

- Basically available：基本可用
- Soft state：数据允许一致、不一致外，还允许一个中间状态
- Eventually Consistence：最终一致性，即有限时间内数据达到一致





# Spring Cloud Feign使用和拓展

- Open Feign
- Spring Cloud Open Fiegn



# 服务调用与熔断

## 核心概念

### RPC

​	Remote Procedure Call，远程过程调用，是一个计算机通信协议。该协议允许运行于一台计算机的[程序](https://zh.wikipedia.org/wiki/程序)调用另一台计算机的[子程序](https://zh.wikipedia.org/wiki/子程序)，而程序员无需额外地为这个交互作用编程。如果涉及的软件采用[面向对象编程](https://zh.wikipedia.org/wiki/面向对象编程)，那么远程过程调用亦可称作**远程调用**或**远程方法调用**

例如：

- Java RMI（二进制协议）
- WebService（文本协议）
  - XML约束
    - DTD
    - XSD（schema）：强类型表达

常用框架：

- Dubbo
- grpc



### 消息传递

RPC是一种请求-响应协议。一次 RPC在客户端初始化，再由客户端将请求消息传递到远程的服务器，执⾏指定的带有参数的过程。经过远程服务器器执⾏过程后，将结果作为响应内容返回到客户端。 

模型：

- 请求-响应
- 事件驱动，异步

### 存根

指一次分布式计算RPC过程中，客户端和服务器转化参数的一段代码。由于存根的参数转化， RPC执行过程如同本地执⾏函数调⽤。存根必须在客户端和服务器器两端均装载，并且保持兼容。 



实现java.io.Serializable接口：有状态，用字段标识状态

协议对比：https://github.com/RuedigerMoeller/fast-serialization/wiki/Benchmark



## Spring Cloud Open Feign

### Feign

​	feign是一个java到http客户端的绑定器，它受Retrofix、JAXRS-2.0和webservice启发。它的首要目标就是简化HTTP APIs的复杂性，不管该客户端是不是restful风格的。

feign：https://github.com/search?q=feign	

#### 常用rest框架注解驱动对比

| REST 框架               | 使用场景           | 请求映射注解    | 请求参数      |
| ----------------------- | ------------------ | --------------- | ------------- |
| Feign                   | 客户端声明         | @RequestLine    | @Param        |
| Spring Cloud Open Feign | 客户端声明         | @ReqeustMapping | @RequestParam |
| JAX-RS                  | 客户端、服务端声明 | @Path           | @*Param       |
| Spring Web MVC          | 服务端声明         | @ReqeustMapping | @RequestParam |

> Spring Cloud Open Fiegn利用Feign高拓展性，使用标准Spring MVC来声明客户端java接口

- Feign
  - 注解拓展性
  - HTTP请求处理
  - REST请求源信息解析
- Spring Cloud Open Feign
  - 提供Spring Web MVC注解处理
  - 提供Feign自动装配

> Spring Cloud Open Feign是通过java接口的方式来声明REST服务提供者的请求元信息，通过调用java接口的方式来实现HTTP/REST通信





#### 支持拓展

- 内建Feign注解
- JAX-RS 1/2注解
  - JAX_RX标准实现：
    - RestEasy
    - Jersey
- JAXB
- OkHttp
- ...

实现：

通过Contract 接口实现

- 默认实现
  - feign.Contract.Default
- Spring Cloud open Feign实现
  - org.springframework.cloud.openfeign.support.SpringMvcContract
    - 通过org.springframework.cloud.openfeign.AnnotatedParameterProcessor联系Fiegn接口



feign的延迟，发现服务，负载均衡



## 代码实现

- 注册中心（eureka server、zk、consul等）发现应用
- feign调用服务、负载均衡
- ribbin熔断

### 服务提供方

依赖

```xml
<!--eureka服务注册与发现客户端依赖-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<!--zookeeper服务注册与发现客户端依赖-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
</dependency>
```

配置：

bootstrap.yaml

```properties
spring:
  cloud:
    zookeeper:
      enabled: false
    consul:
      enabled: false


#停用eureka
eureka:
  client:
    enabled: false

--- #eureka profiles
spring:
  profiles: eureka

#eureka客户端配置
eureka:
  server:
    host: 127.0.0.1  #自定义配置
    port: 8080
  client:
    service-url:
      defaultZone: http://${eureka.server.host}:${eureka.server.port}/eureka #连接注册eureka server中心
    enabled: true
    registryFetchIntervalSeconds: 5 # 5 秒轮训一次

--- #zookeeper profiles
spring:
  profiles: zookeeper
  cloud:
    zookeeper:
      enabled: true
      connect-string: 119.23.201.241:2181

---  #consul profiles
spring:
  profiles: consul
  cloud:
    consul:
      discovery:
        enabled: true
        ip-address: 127.0.0.1
        port: 8500
      enabled: true
```

application.yaml

```properties
spring:
  application:
    name: service-invoke-provider

server:
  port: 0
```

通过--spring.profiles.active启动

启动类：

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;

@SpringBootApplication
@EnableDiscoveryClient
public class ServiceInvokeProviderStarter {

    public static void main(String[] args) {
        SpringApplication.run(ServiceInvokeProviderStarter.class,args);
    }
}

```

服务示例：

```java
package com.example;

import org.springframework.beans.factory.annotation.Value;
import org.springframework.core.env.Environment;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class EchoController {

    private final Environment environment;

    public EchoController(Environment environment) {
        this.environment = environment;
    }

    /**
     * 获取实际运行的端口
     * @return
     */
    private String getLocalServerPort(){
        return environment.getProperty("local.server.port");
    }
    
    @Value("${server.port}")
    private int port;

    @GetMapping(value = "/echo/{message}")
    String echo(@PathVariable(value = "message") String message){
        System.out.println("i am service provider...");
        return "Echo "+ getLocalServerPort()+ " " + message;
    }
}

```



### 服务调用方

依赖：

```xml
<!--spring cloud open feign依赖-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-openfeign</artifactId>
</dependency>
<!--eureka服务注册与发现客户端依赖-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
</dependency>

<!--zookeeper服务注册与发现客户端依赖-->
<dependency>
	<groupId>org.springframework.cloud</groupId>
	<artifactId>spring-cloud-starter-zookeeper-discovery</artifactId>
</dependency>
```

依赖：类似服务提供方

启动类/服务调用类：

```java
package com.example;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.cloud.client.discovery.EnableDiscoveryClient;
import org.springframework.cloud.openfeign.EnableFeignClients;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@SpringBootApplication
@EnableDiscoveryClient
@EnableFeignClients
@RestController
public class ServiceInvokeClientBootstrap {

    private final EchoServiceClient serviceClient;

    public ServiceInvokeClientBootstrap(EchoServiceClient serviceClient) {
        this.serviceClient = serviceClient;
    }

    public static void main(String[] args) {
        SpringApplication.run(ServiceInvokeClientBootstrap.class,args);
    }

    @GetMapping("/call/echo/{message}")
    public String callEcho(@PathVariable("message") String msg){
        return serviceClient.echo(msg);
    }
}

```

```java
package com.example;

import org.springframework.cloud.openfeign.FeignClient;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.PathVariable;

/**
 * 服务调用类
 * 声明了服务所在应用
 * 定义实际调用方法接口
 */
@FeignClient("service-invoke-provider")
public interface EchoServiceClient {

    @GetMapping(value="/echo/{message}")
    String echo(@PathVariable(value = "message") String message);
}

```



### 实现时常见的坑

- @Value("${server.port}")不一定靠谱，若设置了0随机端口，读取不到
- `@LocalServerPort` 也不一定靠谱，因为注入阶段`lacal.server.port`也不一定存在
- Spring Cloud+Netflix Ribbon有一个30s的延迟
  - com.netflix.loadbalancer.DynamicServerListLoadBalancer
    - 通过`ServerListUpdater`更新服务列表
      - 非eureka实现：com.netflix.loadbalancer.PollingServerListUpdater
      - eureka实现：com.netflix.niws.loadbalancer.EurekaNotificationServerListUpdater



## Spring Cloud Feign实例细节

### 猜想

- Java 接口（以及方法）与 REST 提供者元信息如何映射
- @FeignClient 注解所指定的应用（服务）名称可能用到了服务发现，一个服务可以对应多个服务实例（HOST:PORT)
- @EnableFeignClients 注解是如何感知或加载标注@FeignClient 的配置类（Bean）
- Feign 请求和响应的内容是如何序列化和反序列化对应的 POJO 的 



### @EnableFeignClients

实现策略：Enable模块驱动

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.cloud.openfeign;

import java.lang.annotation.Documented;
import java.lang.annotation.ElementType;
import java.lang.annotation.Retention;
import java.lang.annotation.RetentionPolicy;
import java.lang.annotation.Target;
import org.springframework.context.annotation.Import;

@Retention(RetentionPolicy.RUNTIME)
@Target({ElementType.TYPE})
@Documented
@Import({FeignClientsRegistrar.class})
public @interface EnableFeignClients {
    String[] value() default {};

    String[] basePackages() default {};

    Class<?>[] basePackageClasses() default {};

    Class<?>[] defaultConfiguration() default {};

    Class<?>[] clients() default {};
}

```

可见，导入了`org.springframework.cloud.openfeign.FeignClientsRegistrar`类，其主要功能为：

- 注册默认配置

  - `org.springframework.cloud.openfeign.FeignClientsRegistrar#registerDefaultConfiguration`

- 注册所有标注@FeignClient配置类

  - `org.springframework.cloud.openfeign.FeignClientsRegistrar#registerFeignClients`

    - 通过ClassPathScanningCandidateComponentProvider删选basePackage（由EnableFeignClients.class获得）包下的声明为`@FeignClient`的类，获得了类定义信息AnnotatedBeanDefinition，然后通过BeanDefinitionBuilder生成FeignClientFactoryBean将其注册到Spring中管理

      ```java
      BeanDefinitionBuilder definition = BeanDefinitionBuilder
      				.genericBeanDefinition(FeignClientFactoryBean.class);
      ```

  - FeignClientFactoryBean会生成代理对象
    - org.springframework.cloud.openfeign.FeignClientFactoryBean#getTarget
      - 其中重要的类feign.Feign.Builder
      - 最终生成代理对象：feign.ReflectiveFeign#newInstance

> @EnableFeignClients->@FeignClient元信息->标记接口定义的FeignClientFactoryBean->形成被标注接口的代理对象 



### 大致流程

1. Spring Web MVC 注解元信息解析 
   - Contract
   - Feign 内建实现
     - @RequestLine
     - @Headers
   - JAX-RS 1/2 注解
     - @Path
     - @PathParam
     - @HeaderParam
   - Spring Web MVC ( Spring Cloud Open Feign 扩展) 
2. 通过 @FeignClient 所生成代理对象的方法调用实现 HTTP 调用 
3. 通过 SpringDecoder 实现 Response 与接口返回类型的反序列化 
   - feign.Feign.Builder
4. 负载均衡 
   - Spring Cloud 替换了 Client 实现 -LoadBalancerFeignClient 
5. 服务重试
   - Spring Cloud 在外部包装 Feign 接口 
6. 熔断
   - feign.hystrix.HystrixFeign 



## Enable模块驱动

- org.springframework.context.annotation.ImportBeanDefinitionRegistrar

  - @EnableFeignClients

    ```java
    class FeignClientsRegistrar
    		implements ImportBeanDefinitionRegistrar, ResourceLoaderAware, EnvironmentAware {
    		...
    		}
    ```

- org.springframework.context.annotation.ImportSelector

  - @EnableAsync

- @Configuration 类

  - @EnableWebMvc 





# Spring Cloud Ribbon

负载均衡：

- 客户端负载均衡（进程内负载均衡）

  ![image-20210214231352998](D:\BaiduNetdiskDownload\markdown笔记\springcloud.assets\image-20210214231352998.png)

- 服务端负载均衡（集中式负载均衡）

  ![image-20210214231340361](D:\BaiduNetdiskDownload\markdown笔记\springcloud.assets\image-20210214231340361.png)

  - Nginx
  - Lvs
  - F5

官网描述：

![image-20210214231444618](D:\BaiduNetdiskDownload\markdown笔记\springcloud.assets\image-20210214231444618.png)



# Spring Cloud Zuul

应用方面：动态路由、监控、弹性、服务治理、编排、安全。zuul可以简单看成应用的前置处理器：

![1586173862973](springcloud.assets\1586173862973.png)

其具备以下功能：

- 认证和鉴权
- 压力控制
- 金丝雀测试
- 负载削减
- 静态响应处理
- 主动流量管理

入门示例：ch7-2-zuul-server，ch7-2-client-a，eureka-server-demo

![1589380298240](springcloud.assets\1589380298240.png)



## Zuul典型配置

### 路由配置

#### 路由配置简化及规则

- 单实例serviceId映射

  前面示例中的配置如下：

  ```yaml
  zuul:
    routes:
      client-a:
        path: /client/** #以此开头的请求映射到client-a(application-name)
        serviceId: client-a
  ```

  这是一个`/client/**`到`client-a`服务的映射规则，其可以简化为

  ```yaml
  zuul:
    routes:
      client-a: /client/**
  ```

  前面甚至还可以简化为：

  ```yaml
  zuul:
    routes:
      client-a: 
  ```

  这种情况下，zuul默认会为client-a服务添加一个默认的映射规则：/client-a/**，相当于：

  ```yaml
  zuul:
    routes:
      client-a: /client-a/**
  ```

- 单实例url映射

  ```yaml
  zuul:
    routes:
      client-a:
        path: /client/**
        url: http://localhost:7070
  ```

- 多实例路由

- 



#### 路由通配符

| 规则 | 释义               | 例子                 |
| ---- | ------------------ | -------------------- |
| /**  | 匹配任意字符和路径 | /client,/client/a    |
| /*   | 只匹配任意字符     | /aaa,/bbb            |
| /?   | 匹配单个字符       | /client/a, /client/b |



### 功能配置





# Spring cloud认证和鉴权

## JWT

Jwt(json web token)，是目前最流行的跨域认证解决方案。Jwt数据结构分为三部分，

- Header（头部）
- Payload（负载）
- Signature（签名）

三部分由符号.分隔，`Header.Payload.Signature`

http://www.ruanyifeng.com/blog/2018/07/json_web_token-tutorial.html





# Spring Cloud Gateway