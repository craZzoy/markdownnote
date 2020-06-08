# spring boot初体验

构建项目：从官网下载

SOA（Service-oriented architecture）：面向服务架构

微服务：更细粒度的SOA

## Spring Boot Actuator

​	springboot用于监控和管理运用的组件，需引入spring mvc才有用，actuator组件也许添加依赖

```xml
<dependencies>
<dependency>
<groupId>org.springframework.boot</groupId>
<artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
</dependencies>
```

### EndPoints

​	如下面例子，更多查看官方文档

- /dump：查看线程信息

- /beans：查看Spring管理的类

- /metrics：一些指标值，如总内存，空闲内存等

  ...



# Spring Boot-Web

## 静态web内容

基本解释：

HTTP请求内容由web服务器文件系统提供，常见静态web内容如：HTML、JS、CSS、JPG、Flash等等

特点：

- 计算类型：I/O类型
- 交互方式：单一
- 资源内容：基本相同（可能后天会有权限相关处理返回不一样的内容）
- 资源路径：物理路径（文件、目录）
- 请求方式：GET为主



### 常见场景

- 信息展示
- 样式文件（CSS）
- 脚本文件（JS）
- 图片（JPG、GIF）
- 多媒体（Flash、Movie）
- 文件下载



### 常见web服务器

- Apache HTTP Server
- Nginx
- Microsoft IIS
- GWS



为什么java web Server不是常用Web Server？

- 内存占用
  - 类型：如java中int数据类型是32位，c语言中更省内存
  - 分配：java中的对象创建时可能会分配更多的内存
- 垃圾回收
  - 被动回收
  - 停顿：full gc的触发可能会引起停顿
- 并发处理
  - 线程池
  - 线程开销

### 标准化技术

- 资源变化

  - 响应头：Last-Modified
  - 请求头：if-Modified-Since

  ```
  Last-Modified: Wed, 03 Sep 2014 10:00:27 GMT
  If-Modified-Since: Wed, 03 Sep 2014 10:00:27 GMT
  ```

  

- 资源缓存

  - 响应头：ETag
  - 请求头：If-None-Match

  ```
  Etag: "1ec5-502264e2ae4c0"
  If-None-Match: "1ec5-502264e2ae4c0"
  ```

  

### 示例

Spring默认静态资源路径配置在

```java
public class ResourceProperties implements ResourceLoaderAware, InitializingBean {
 
private static final String[] CLASSPATH_RESOURCE_LOCATIONS = new String[]{"classpath:/META-INF/resources/", "classpath:/resources/", "classpath:/static/", "classpath:/public/"};
}
```

jsp页面获取静态资源：

```jsp
<br>
<a href="/text.txt">text</a>
<br>
<img src="/images/test.jpg" />
```

http请求获取静态资源：

<http://localhost:8080/text.txt>

可自定义配置：

```properties
#匹配路径
spring.mvc.static-path-pattern=/resources/**
#静态资源存储路径
spring.resources.static-locations=classpath:/META-INF/resources/,classpath:/resources/,classpath:/static/,classpath:/public/,classpath:/mystatic/
```



JSR：Java Specification Requests，java规范提案



实例：构建一个支持JSP的Spring Boot应用。

```java
package sample.jsp;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.builder.SpringApplicationBuilder;
import org.springframework.boot.web.support.SpringBootServletInitializer;

@SpringBootApplication
public class SampleWebJspApplication extends SpringBootServletInitializer {

    @Override
    protected SpringApplicationBuilder configure(SpringApplicationBuilder application) {
        return application.sources(SampleWebJspApplication.class);
    }

    public static void main(String[] args) throws Exception {
        SpringApplication.run(SampleWebJspApplication.class, args);
    }

}
```

踩坑：<https://blog.csdn.net/qq496013218/article/details/76241027>



## 动态WEB内容

基本解释：与静态内容不一样，需通过服务器计算而来

特点：

- 计算类型：混合方式（I/O、CPU、内存等）
- 交互方式：丰富（用户输入、客户端特征等）
- 资源内容：多样性
- 资源路径：逻辑路径（虚拟）
- 请求方法：GET、Head、PUT、POST



### 常见场景

- 页面渲染
- 表单交互（From）
- AJAX
- XML
- JSON/JSONP
- Web Services(SOAP、WSDL)
- WebSocket



### 流行服务器

- Servlet容器
  - Tomcat
  - Jetty
- 非Servlet容器
  - Undertow



### 技术/架构演进

- CGI（Common Gate Way）

- Servlet

- JSP(Java Server page)

- Model1(JSP+JavaBeans)

  - model1结构中，JSP充当了MVC中VC的角色，所有请求通过转发到JSP、业务逻辑也在JSP处理，耦合性很高。提升后的Model1在JSP中也处理Java Bean。

  ![1567049006480](SpringBoot.assets\1567049006480.png)

- Model2(MVC)

  - 分为了三层，各司其职。

    ![1567049157769](SpringBoot.assets\1567049157769.png)

  - Struts Web Mvc

    ![1567049336361](SpringBoot.assets\1567049336361.png)

  - Spring Web MVC

    ![1567049374742](SpringBoot.assets\1567049374742.png)



### Model2和MVC的细微差异

- Model2为面向Web服务的架构，MVC则是面向所有应用场景（比如：PC应用、无线应用）
- 相对于MVC，Model2中Controller细化为Font Controller（FC）和Application Controller（AC），前者（FC）负责路由到后者（AC），后者负责跳转视图（View）。

### 模板引擎

- JSP

- Velocity

- Thymeleaf

  ...



问题一：Spring MVC中的Font Controller是DispatherServlet，@Controller实际是指Application Controller或者Command Controller（strut）

问题二：js、图片等封装在JAR包中，性能下降体现在哪些方面？每次请求都会创建新的线程？

springboot默认打包方式是jar -0（不使用任何zip压缩），所以并没有解压的消耗，不过每次url请求中都会从服务器的相关文件系统中获取文件。有没有创建新的线程取决于实现的线程池，超过设定的不会创建新的线程。

问题三：什么时候用模板语言、js渲染页面

模板语言是后端语言，需服务器计算。js是在客户端执行的

问题四：jsp、velocity、Thymeleaf

- Jsp：翻译式的语言，jsp->java->class，后端执行，相对比velocity快，编译需花费更多时间。但java代码耦合在jSP中，职责不分明。例外，JSP还支持HTML、XHTML、DHTML、XML等。
- velocity：解析执行语言（AST解析），相对JSP较慢

性能：jsp>velocity>Thymeleaf

可读性：jsp<velocity<Thymeleaf



## @AliasFor标签

参考文章<https://blog.csdn.net/wolfcode_cn/article/details/80654730>

使用org.springframework.core.annotation.AnnotationUtils#getAnnotation(java.lang.annotation.Annotation, java.lang.Class<A>)是的自定义注解支持@AliasFor注解



## Rest理论基础

### 基本概念

​	REST（RESTFUL）：Representational state transfer,is one way of providing interoperability between computer systems on the Internet.即它是一种软件风格，目的是便于不同软件/程序在网络中互相传递信息，它是基于HTTP协议之上而确定的一种约束和属性，注意它只是一种设计风格而不是标准（更多维基百科）。



### 对比其他web服务实现方案

- REST：相对更加简洁
- WSDL、SOAP
- XML-RPC



### 架构属性/约束

- C/S架构（Client-Server）：B/S架构可视为C/S架构
- 无状态（stateless）：即每个请求是相互独立不依赖的
- 可缓存（Cacheable）：如Cookie
- 分层结构（Layer System）:分层的系统，客户端不知道所连接的是不是最终服务器
- 按需代码（Code on demand）：服务端可以将能力拓展到客户端，如果客户端可以执行的话。这个功能是可选择的。
- 统一接口（Uniform interface）
  - 资源识别（Identification of resources）
    - URI（Uniform Resources identifier）:区别URL，URI包括URL，URL更具体，注意identifier和location的重点，结构你的名字（Identifier）和住址（Location）理解。
  - 资源操作（Manipulation of resources through representations）
    - GET：查询，幂等（相对）
    - PUT：更新，幂等（相对）
    - POST：插入，非幂等，实现时需考虑这点
    - DELETE：删除，幂等（相对）
  - 自描述信息（Self-descriptive messages）
    - Content-Type：浏览器返回的内容类型，Request Header中的Accept指定了浏览器可解析的类型
    - MIME-TYPE：媒体类型，HTTP中体现在Content-Type
    - Media-Type：application/JavaScript、text/html
  - 超媒体（HATEOAS）
    - Hypermedia As The Engine Of Application State



## Rest服务端实战

### Spring Boot Rest实战

### 核心接口

- 定义相关

  - @Controller
  - @RestController：@Controller+@ResponseBody，不加入@ResponseBody的方法返回会被模板引擎解析

  ```java
  //
  // Source code recreated from a .class file by IntelliJ IDEA
  // (powered by Fernflower decompiler)
  //
  
  package org.springframework.web.bind.annotation;
  
  import java.lang.annotation.Documented;
  import java.lang.annotation.ElementType;
  import java.lang.annotation.Retention;
  import java.lang.annotation.RetentionPolicy;
  import java.lang.annotation.Target;
  import org.springframework.stereotype.Controller;
  
  @Target({ElementType.TYPE})
  @Retention(RetentionPolicy.RUNTIME)
  @Documented
  @Controller
  @ResponseBody
  public @interface RestController {
      String value() default "";
  }
  ```

  ```java
  package src.controller;
  
  
  import org.springframework.web.bind.annotation.GetMapping;
  import org.springframework.web.bind.annotation.ResponseBody;
  import org.springframework.web.bind.annotation.RestController;
  
  @RestController
  //@Controller
  public class HTMLRestController {
  
      /**
       * 使用@Controller不加@ResponseBody会报404
       * @return
       */
      @GetMapping(path = {"/html/demo"})
      @ResponseBody
      public String html(){
          return "<html><body>Hello,World</body></html>";
      }
  
  }
  ```

  请求效果：

  ![1567089041831](SpringBoot.assets\1567089041831.png)

  注意看响应头信息，即返回的类型是html类型

  ```
  Content-Type: text/html;charset=UTF-8
  ```

- 映射相关

  - @RequestMapping

    ```java
    @GetMapping(path = {"/html/demo"})  //等于@RequestMapping(path = {"/html/demo"},method = RequestMethod.GET)
    @PostMapping(path = {"/html/demo1"})
    //@ResponseBody
    public String html(){
        return "<html><body>Hello,World</body></html>";
    }
    ```

  - @PathVariable

    ```java
    @GetMapping(path = {"/html/demo/{message}"})
    public String html(@PathVariable(value = "message") String msg){
        return "<html><body>"+msg+"</body></html>";
    }
    ```

- 请求相关

  - @RequestParam

    ```java
    /**
     * 可以通过HttpServletRequest获取参数
     * @RequestParam提供了自动转型的功能，以及默认值等等特性
     * @param name
     * @param age
     * @return
     */
    @GetMapping(path = {"/html/demo/param"})
    public String htmlParam(@RequestParam(value = "name",required = false,defaultValue = "Empty") String name,
                            @RequestParam(value = "age",required = false,defaultValue = "0") Integer age){
        return "<html><body> Request Parameter1 value name: " + name
                + " , parameter2 value age:" + age +
                " </body></html>";
    }
    ```

  - @RequestHeader

    ```java
        @GetMapping(path = {"/html/demo/header"})
        public String htmlHeader(@RequestHeader(value = "Accept") String accept
                , @CookieValue(value = "cookie",defaultValue = "Empty", required = false) String c
                , RequestEntity entity){
            System.out.println(entity);
            return "<html><body>"+"Accept:"+accept+"<br>cookie:"+c+"</body></html>";
        }
    ```

  - @CookieValue

  - RequestEntity

- 响应相关

  - @ResponseBody

  - ResponseEntity

    ```java
        @GetMapping(path = {"/html/demo/response/entity"})
        public ResponseEntity<String> htmlResponseEntity(){
            HttpHeaders httpHeaders = new HttpHeaders();
            httpHeaders.put("selfHeader", Arrays.asList("Hello,World"));
            ResponseEntity responseEntity = new ResponseEntity(
                    "<html><body>htmlResponseEntity</body></html>",
                    httpHeaders,
                    HttpStatus.OK);
            return responseEntity;
        }
    ```



### 返回JSON数据

看如下方法

```java
@GetMapping("/json/user")
public User user(){
    return user;
}
```

自动返回了application/json类型：

![1567170210604](SpringBoot.assets\1567170210604.png)

```java
package src.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.beans.factory.annotation.Qualifier;
import org.springframework.context.annotation.Bean;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.annotation.Resource;

@RestController
public class JSONRestController {

    @Autowired
    @Qualifier("currentUser")
    //@Resource(name = "currentUser")
    private User user;

    @Bean
    public User currentUser(){
        User user = new User();
        user.setName("JSON");
        user.setAge(20);
        return user;
    }


    @GetMapping("/json/user")
    public User user(){
        return user;
    }

    @GetMapping("/json/user/set/name")
    public User setUserName(@RequestParam String name){
        user.setName(name);
        return user;
    }

    @GetMapping("/json/user/set/age")
    public User setAge(@RequestParam Integer age){
        user.setAge(age);
        return user;
    }
}
```

### 返回XML数据

引入依赖：

```xml
<!--json转xml组件-->
<dependency>
    <groupId>com.fasterxml.jackson.dataformat</groupId>
    <artifactId>jackson-dataformat-xml</artifactId>
</dependency>
```

controller：

```java
package src.controller;

import org.springframework.http.MediaType;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;


@RestController
public class XMLRestController {

    /**
     * 默认为xml，可省略produces application/xml
     * 通过produces指定返回媒体类型
     * @return
     */
    @GetMapping(path="/xml/user",produces = MediaType.APPLICATION_XML_VALUE)
    public User user(){
        User user = new User();
        user.setName("XML");
        user.setAge(30);
        return user;
    }
}

```

此时效果：

![1567171465284](SpringBoot.assets\1567171465284.png)

其实JSON数据被处理成了XML格式：

![1567171918145](SpringBoot.assets\1567171918145.png)

同样，可通过produces指定返回类型

```java
@GetMapping(path="/json/user",produces = MediaType.APPLICATION_JSON_VALUE)
public User user(){
    return user;
}
```



### HATEOAS 

​	HATEOAS = Hypermedia As The Engine Of Application State，前面有提到endpoint的应用，用于发现自己服务

引入依赖：

```xml
<!--引入制动器，用于监控管理-->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-actuator</artifactId>
</dependency>
```

1.5.x版本默认是权限敏感的，需设置：

```properties
endpoints.enabled=true
endpoints.sensitive=false
```



此外，还可以通过HATEOAS自定义把用户自定义的API信息暴露出去：

引入依赖：

```xml
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-hateoas</artifactId>
</dependency>
```

实体类继承ResourceSupport：

```java
package src.controller;

import org.springframework.hateoas.ResourceSupport;

public class User extends ResourceSupport{
    private String name;
    private Integer age;

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }
}
```

请求处理：

```java
@GetMapping(path="/json/user",produces = MediaType.APPLICATION_JSON_VALUE)
public User user(){
    user.add(ControllerLinkBuilder
            .linkTo(ControllerLinkBuilder.methodOn
                    (JSONRestController.class).setUserName(user.getName()))
            .withSelfRel());
    user.add(ControllerLinkBuilder
            .linkTo(ControllerLinkBuilder.methodOn
                    (JSONRestController.class).setAge(user.getAge()))
            .withSelfRel());
    return user;
}
```

效果：

```json
{
    name: "tom",
    age: 20,
    _links: {
    self: [
            {
            href: "http://localhost:8080/json/user/set/name?name=JSON"
            },
            {
            href: "http://localhost:8080/json/user/set/age?age=20"
            },
            {
            href: "http://localhost:8080/json/user/set/name?name=JSON"
            },
            {
            href: "http://localhost:8080/json/user/set/age?age=20"
            },
            {
            href: "http://localhost:8080/json/user/set/name?name=tom"
            },
            {
            href: "http://localhost:8080/json/user/set/age?age=20"
            }
        ]
	}
}
```



### 对比技术

​	Web Servie：WSDL



GET（查询）、PUT（更新）、DELETE（删除）相对比较幂等

POST（插入）非幂等，实现时需考虑



## Rest客户端实战

### WEB浏览器

如上



### Apache HttpClient

引入依赖：

```xml
<dependency>
   <groupId>org.apache.httpcomponents</groupId>
   <artifactId>httpclient</artifactId>
</dependency>
```

```java
package src.client;

import org.apache.http.client.HttpClient;
import org.apache.http.impl.client.HttpClientBuilder;
import org.springframework.http.client.HttpComponentsClientHttpRequestFactory;
import org.springframework.web.client.RestTemplate;
import src.controller.User;

public class RestClient {
    public static void main(String[] args) {

        HttpClientBuilder httpClientBuilder = HttpClientBuilder.create();
        HttpClient httpClient = httpClientBuilder.build();
        HttpComponentsClientHttpRequestFactory factory = new HttpComponentsClientHttpRequestFactory(httpClient);
        RestTemplate restTemplate = new RestTemplate(factory);


        //RestTemplate restTemplate = new RestTemplate();
        User user = restTemplate.getForObject("http://localhost:8080/json/user", User.class);
        System.out.println(user);
    }
}
```

结果：

```
User{name='tom', age=20}
```

同样，HATEOAS 中的link信息未获取到



### Spring RestTemple

```java
package src.client;

import org.springframework.web.client.RestTemplate;
import src.controller.User;

public class RestClient {
    public static void main(String[] args) {
        RestTemplate restTemplate = new RestTemplate();
        User user = restTemplate.getForObject("http://localhost:8080/json/user", User.class);
        System.out.println(user);
    }
}
```

结果：

```
User{name='tom', age=20}
```

注：HATEOAS 中的link信息未取到



## 传统Servlet

### 什么是Servlet？

​	Servlet是一种基于java技术的Web组件，用于生成动态内容，由容器管理（Servlet容器），类似于其他java技术组件，Servlet是由平台无关的java组成，并且由java web服务器负责执行。

### 什么是Servlet容器？

​	Servlet容器，又称Servlet引擎，作为Web服务器或应用服务器的一部分。通过请求和响应对话，提供Web客户端与Servlet交互的能力。容器管理Servlet示例以及它们的生命周期。

### 核心接口

#### Servlet3.0前时代

##### 服务组件

- javax.servlet.Servlet：servlet默认是线程不安全的

  - javax.servlet.GenericServlet
  - javax.servlet.http.HttpServlet：一般的Servlet实现此接口（HTTP）

- javax.servlet.Filter（@since Servlet 2.3）

  ```java
  /*
   * Licensed to the Apache Software Foundation (ASF) under one or more
   * contributor license agreements.  See the NOTICE file distributed with
   * this work for additional information regarding copyright ownership.
   * The ASF licenses this file to You under the Apache License, Version 2.0
   * (the "License"); you may not use this file except in compliance with
   * the License.  You may obtain a copy of the License at
   *
   *     http://www.apache.org/licenses/LICENSE-2.0
   *
   * Unless required by applicable law or agreed to in writing, software
   * distributed under the License is distributed on an "AS IS" BASIS,
   * WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   * See the License for the specific language governing permissions and
   * limitations under the License.
   */
  package javax.servlet;
  
  import java.io.IOException;
  
  /**
   * A filter is an object that performs filtering tasks on either the request to
   * a resource (a servlet or static content), or on the response from a resource,
   * or both. <br>
   * <br>
   * Filters perform filtering in the <code>doFilter</code> method. Every Filter
   * has access to a FilterConfig object from which it can obtain its
   * initialization parameters, a reference to the ServletContext which it can
   * use, for example, to load resources needed for filtering tasks.
   * <p>
   * Filters are configured in the deployment descriptor of a web application
   * <p>
   * Examples that have been identified for this design are<br>
   * 1) Authentication Filters <br>
   * 2) Logging and Auditing Filters <br>
   * 3) Image conversion Filters <br>
   * 4) Data compression Filters <br>
   * 5) Encryption Filters <br>
   * 6) Tokenizing Filters <br>
   * 7) Filters that trigger resource access events <br>
   * 8) XSL/T filters <br>
   * 9) Mime-type chain Filter <br>
   *
   * @since Servlet 2.3
   */
  public interface Filter {
  
      public void init(FilterConfig filterConfig) throws ServletException;
  
      public void doFilter(ServletRequest request, ServletResponse response,
              FilterChain chain) throws IOException, ServletException;
      
      public void destroy();
  
  }
  ```

##### 上下文组件

- javax.servlet.ServletContext：由容器供应商负责实现
- javax.servlet.http.HttpSession
- javax.servlet.http.HttpServletRequest
- javax.servlet.http.HttpServletResponse
- javax.servlet.http.Cookie（客户端）

##### 配置

- javax.servlet.ServletConfig
- javax.servlet.FilterConfig（since Servlet 2.3）

##### 输入输出

- javax.servlet.ServletInputStream：继承了InPutStream
- javax.servlet.ServletOutputStream：继承了OutPutStream

##### 异常

- javax.servlet.ServletException

##### 事件（用于对应监听器参数传入）

- 声明周期类型
  - javax.servlet.ServletContextEvent
  - javax.servlet.http.HttpSessionEvent
  - java.servlet.ServletRequestEvent
- 属性上下文类型（对应某个上下文组件属性改变触发）
  - javax.servlet.ServletContextAttributeEvent
  - javax.servlet.http.HttpSessionBindingEvent
  - javax.servlet.ServletRequestAttributeEvent

##### 监听器

- 生命周期类型
  - javax.servlet.ServletContextListener
  - javax.servlet.http.HttpSessionListener
  - javax.servlet.http.HttpSessionActivationListener
  - javax.servlet.ServletRequestListener
- 属性上下文类型
  - javax.servlet.ServletContextAttributeListener
  - javax.servlet.http.HttpSessionAttributeListener
  - javax.servlet.http.HttpSessionBindingListener
  - javax.servlet.ServletRequestAttributeListener



#### Servlet3后时代

- 组件声明注解
  - @javax.servlet.annotation.WebServlet
  - @javax.servlet.annotation.WebFilter
  - @javax.servlet.annotation.WebListener
  - @javax.servlet.annotation.ServletSecurity
  - @javax.servlet.annotation.HttpMethodConstraint
  - @javax.servlet.annotation.HttpConstraint
- 配置声明
  - @javax.servlet.annotation.WebInitParam
- 上下文
  - javax.servlet.AsyncContext
- 事件
  - javax.servlet.AsyncEvent
- 监听器
  - javax.servlet.AsyncListener
- Servlet组件注册
  - javax.servlet.ServletContext#addServlet()
  - javax.servlet.ServletRegistration
- Filter组件注册
  - javax.servlet.ServletContext#addFilter()
  - javax.servlet.FilterRegistration
- 监听器注册
  - javax.servlet.ServletContext#addListener()
  - javax.servlet.AsyncListener
- 自动装配
  - 初始器
    - javax.servlet.ServletContainerInitializer
  - 类型过滤
    - @javax.servlet.annotation.HandlesTypes



### Servlet生命周期

1. 构造器：调用一次，Servlet类实例化，单例
2. init()：一次，构造方法执行后，初始化Servlet。
3. service()：多次，处理请求。
4. destroy()：servlet对象销毁后，调用一次。重新部署网站或者停止服务器触发。



### Filter生命周期

1. 构造器：实例化
2. init()：初始化
3. doFilter()：拦截请求，Servlet#service方法调用前执行
4. destroy()



### SPI 

​	service provider interface，是java提供的一套用来被第三方实现或者拓展的API，它可以用来启动框架和替换组件。如springboot中就使用了SPI的机制实现了javax.servlet.ServletContainerInitializer接口。

 参考：<https://www.jianshu.com/p/46b42f7f593c>



## Servlet on SpringBoot

### Servlet组件扫描和注解方式注册

1. 添加@org.springframework.boot.web.servlet.ServletComponentScan扫描Servlet组件

   - 指定包路径扫描
     - String[] value() default {}
     - String[] basePackages() default {}
   - 指定类扫描
     - Class<?>[] basePackageClasses() default {}

2. 注解方式注册组件

   - Servlet组件

     1. 扩展 javax.servlet.Servlet
        - javax.servlet.http.HttpServlet（servlet标准）
        - org.springframework.web.servlet.FrameworkServlet（Spring实现）
     2. 标记 @javax.servlet.annotation.WebServlet

     ```java
     package src.servlet;
     
     import javax.servlet.ServletConfig;
     import javax.servlet.ServletContext;
     import javax.servlet.ServletException;
     import javax.servlet.annotation.WebInitParam;
     import javax.servlet.annotation.WebServlet;
     import javax.servlet.http.HttpServlet;
     import javax.servlet.http.HttpServletRequest;
     import javax.servlet.http.HttpServletResponse;
     import java.io.IOException;
     import java.io.Writer;
     
     @WebServlet(
             name = "myServlet",
             urlPatterns = "/myServlet",
             initParams = {
                     @WebInitParam(name="myname",value = "myname")
             }
     )
     public class MyServlet extends HttpServlet{
     
         private String value;
     
         @Override
         public void init(ServletConfig config) throws ServletException {
             super.init(config);
             value = config.getInitParameter("myname");
         }
     
         @Override
         protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
             Writer writer = resp.getWriter();
             ServletContext servletContext = getServletContext();
             servletContext.log("myServlet doGet");
             writer.write("<html><body>Hello,World,my value="+this.value+"</body></html>");
         }
     
     
     }
     ```

   - Filter组件

     1. 实现javax.servlet.Filter
        - org.springframework.web.filter.OncePerRequestFilter（Spring接口）
     2. 标记 @javax.servlet.annotation.WebFilter

     ```java
     package src.servlet;
     
     import org.springframework.web.filter.OncePerRequestFilter;
     
     import javax.servlet.Filter;
     import javax.servlet.FilterChain;
     import javax.servlet.ServletContext;
     import javax.servlet.ServletException;
     import javax.servlet.annotation.WebFilter;
     import javax.servlet.http.HttpServletRequest;
     import javax.servlet.http.HttpServletResponse;
     import java.io.IOException;
     
     
     @WebFilter(urlPatterns = "/myServlet")
     //@WebFilter(servletNames = "myServlet") //可指定过滤某个Servlet
     public class MyFilter extends OncePerRequestFilter{
     
         @Override
         protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
             ServletContext servletContext = request.getServletContext();
             servletContext.log("myFilter doFilterInternal");
             filterChain.doFilter(request,response);
         }
     }
     ```

   - 监听器组件

     1. 实现Listener接口

        - javax.servlet.ServletContextListener

        - javax.servlet.http.HttpSessionListener

        - javax.servlet.http.HttpSessionActivationListener

        - javax.servlet.ServletRequestListener

          ```java
          package src.servlet;
          
          import javax.servlet.ServletContext;
          import javax.servlet.ServletRequestEvent;
          import javax.servlet.ServletRequestListener;
          import javax.servlet.annotation.WebListener;
          import javax.servlet.http.HttpServletRequest;
          
          @WebListener
          public class MyServletRequestListener implements ServletRequestListener{
          
              @Override
              public void requestInitialized(ServletRequestEvent sre) {
                  HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
                  ServletContext servletContext = request.getServletContext();
                  servletContext.log("MyServletRequestListener requestInitialized...");
              }
          
              @Override
              public void requestDestroyed(ServletRequestEvent sre) {
                  HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
                  ServletContext servletContext = request.getServletContext();
                  servletContext.log("MyServletRequestListener requestDestroyed...");
              }
          
          
          }
          ```

        - javax.servlet.ServletContextAttributeListener

        - javax.servlet.http.HttpSessionAttributeListener

        - javax.servlet.http.HttpSessionBindingListener

        - javax.servlet.ServletRequestAttributeListener

     2. 标记@javax.servlet.annotation.WebListener

从执行结果可看出这些组件结合起来的生命周期是：

- 监听器->Filter->Servlet

```txt
2019-09-01 19:54:33.471  INFO 17736 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : MyServletRequestListener requestInitialized...
2019-09-01 19:54:33.484  INFO 17736 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : myFilter doFilterInternal
2019-09-01 19:54:33.485  INFO 17736 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : myServlet doGet
2019-09-01 19:54:33.485  INFO 17736 --- [nio-8080-exec-1] o.a.c.c.C.[Tomcat].[localhost].[/]       : MyServletRequestListener requestDestroyed...
```



### Springboot API方式注入

组件定义：

```java
package src.spring;

import javax.servlet.ServletConfig;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebInitParam;
import javax.servlet.annotation.WebServlet;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;
import java.io.Writer;

/*@WebServlet(
        name = "myServlet",
        urlPatterns = "/myServlet",
        initParams = {
                @WebInitParam(name="myname",value = "myname")
        }
)*/
public class MyServletForSpring extends HttpServlet{

    private String value;

    @Override
    public void init(ServletConfig config) throws ServletException {
        super.init(config);
        value = config.getInitParameter("myname");
    }

    @Override
    protected void doGet(HttpServletRequest req, HttpServletResponse resp) throws ServletException, IOException {
        Writer writer = resp.getWriter();
        ServletContext servletContext = getServletContext();
        servletContext.log("myServlet doGet");
        writer.write("<html><body>Hello,World,my value="+this.value+"</body></html>");
    }


}
```

```java
package src.spring;

import org.springframework.web.filter.OncePerRequestFilter;

import javax.servlet.FilterChain;
import javax.servlet.ServletContext;
import javax.servlet.ServletException;
import javax.servlet.annotation.WebFilter;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import java.io.IOException;


//@WebFilter(urlPatterns = "/myServlet")
//@WebFilter(servletNames = "myServlet") //可指定过滤某个Servlet
public class MyFilterForSpring extends OncePerRequestFilter{

    @Override
    protected void doFilterInternal(HttpServletRequest request, HttpServletResponse response, FilterChain filterChain) throws ServletException, IOException {
        ServletContext servletContext = request.getServletContext();
        servletContext.log("myFilter doFilterInternal");
        filterChain.doFilter(request,response);
    }
}
```

```java
package src.spring;

import javax.servlet.ServletContext;
import javax.servlet.ServletRequestEvent;
import javax.servlet.ServletRequestListener;
import javax.servlet.annotation.WebListener;
import javax.servlet.http.HttpServletRequest;

//@WebListener
public class MyServletRequestListenerForSpring implements ServletRequestListener{

    @Override
    public void requestInitialized(ServletRequestEvent sre) {
        HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
        ServletContext servletContext = request.getServletContext();
        servletContext.log("MyServletRequestListenerForSpring requestInitialized...");
    }

    @Override
    public void requestDestroyed(ServletRequestEvent sre) {
        HttpServletRequest request = (HttpServletRequest) sre.getServletRequest();
        ServletContext servletContext = request.getServletContext();
        servletContext.log("MyServletRequestListenerForSpring requestDestroyed...");
    }


}
```

暴露Bean：

```java
package src;

import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.web.servlet.FilterRegistrationBean;
import org.springframework.boot.web.servlet.ServletComponentScan;
import org.springframework.boot.web.servlet.ServletListenerRegistrationBean;
import org.springframework.boot.web.servlet.ServletRegistrationBean;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.ComponentScan;
import src.spring.MyFilterForSpring;
import src.spring.MyServletForSpring;
import src.spring.MyServletRequestListenerForSpring;

import java.util.Arrays;

@SpringBootApplication
@ServletComponentScan(basePackages = {"src.servlet"})
public class LessonFourDemoStarter {
    public static void main(String[] args) {
        SpringApplication.run(LessonFourDemoStarter.class,args);
    }

    /**
     * 注册一个Servlet
     * Spring Boot 1.4.0 开始支持org.springframework.boot.web.servlet.ServletRegistrationBean
     * Spring Boot  1.4.0 之前org.springframework.boot.context.embedded.ServletRegistrationBean
     * @return
     */
    @Bean
    public static ServletRegistrationBean registrationMyServlet(){
        ServletRegistrationBean servletRegistrationBean = new ServletRegistrationBean();
        servletRegistrationBean.setServlet(new MyServletForSpring());
        servletRegistrationBean.setName("myServletForSpring");
        servletRegistrationBean.setUrlMappings(Arrays.asList("/myServletForSpring"));
        return servletRegistrationBean;
    }

    /**
     * 注册一个Filter
     * Spring Boot 1.4.0 开始org.springframework.boot.web.servlet.FilterRegistrationBean
     * Spring Boot  1.4.0 之前 org.springframework.boot.context.embedded.FilterRegistrationBean
     * @return
     */
    @Bean
    public static FilterRegistrationBean registrationMyFilter(){
        FilterRegistrationBean filterRegistrationBean = new FilterRegistrationBean();
        filterRegistrationBean.setFilter(new MyFilterForSpring());
        filterRegistrationBean.setName("myFilterForSpring");
        filterRegistrationBean.setUrlPatterns(Arrays.asList("/myServletForSpring"));
        return filterRegistrationBean;
    }

    /**
     * 注册一个监听器
     * Spring Boot 1.4.0 开始org.springframework.boot.web.servlet.ServletListenerRegistrationBean
     * Spring Boot  1.4.0 之前org.springframework.boot.context.embedded.ServletListenerRegistrationBean
     * @return
     */
    @Bean
    public static ServletListenerRegistrationBean registrationMyListener(){
        ServletListenerRegistrationBean servletListenerRegistrationBean = new ServletListenerRegistrationBean();
        servletListenerRegistrationBean.setListener(new MyServletRequestListenerForSpring());
        return servletListenerRegistrationBean;
    }

}
```



ThreadLocal的运用



## JSP on SpringBoot

1. 引入依赖

   ```properties
   <!-- JSTL -->
   <dependency>
       <groupId>javax.servlet</groupId>
       <artifactId>jstl</artifactId>
   </dependency>
   
   <!--tomcat嵌入式容器-->
   <dependency>
       <groupId>org.springframework.boot</groupId>
       <artifactId>spring-boot-starter-tomcat</artifactId>
       <scope>compile</scope>
   </dependency>
   
   <!--JSP渲染引擎-->
   <dependency>
       <groupId>org.apache.tomcat.embed</groupId>
       <artifactId>tomcat-embed-jasper</artifactId>
       <scope>compile</scope>
   </dependency>
   ```

2. 激活传统Servlet Web部署

   - org.springframework.boot.web.support.SpringBootServletInitializer（since1.4.0）

3. 组装 org.springframework.boot.builder.SpringApplicationBuilder

   ```java
   package src;
   
   import org.springframework.boot.SpringApplication;
   import org.springframework.boot.autoconfigure.SpringBootApplication;
   import org.springframework.boot.builder.SpringApplicationBuilder;
   import org.springframework.boot.web.servlet.FilterRegistrationBean;
   import org.springframework.boot.web.servlet.ServletComponentScan;
   import org.springframework.boot.web.servlet.ServletListenerRegistrationBean;
   import org.springframework.boot.web.servlet.ServletRegistrationBean;
   import org.springframework.boot.web.support.SpringBootServletInitializer;
   import org.springframework.context.annotation.Bean;
   import org.springframework.context.annotation.ComponentScan;
   import src.spring.MyFilterForSpring;
   import src.spring.MyServletForSpring;
   import src.spring.MyServletRequestListenerForSpring;
   
   import java.util.Arrays;
   
   @SpringBootApplication
   @ServletComponentScan(basePackages = {"src.servlet"})
   public class LessonFourDemoStarter extends SpringBootServletInitializer{
       public static void main(String[] args) {
           SpringApplication.run(LessonFourDemoStarter.class,args);
       }
   
       @Override
       protected SpringApplicationBuilder configure(SpringApplicationBuilder builder){
           builder.sources(LessonFourDemoStarter.class);
           return builder;
       }
   
   }
   ```

4. 配置JSP视图

注意，如果是idea中module，需配置工作目录$MODULE_DIR$

![1567351361776](SpringBoot.assets\1567351361776.png)





问题

Spring和Spring MVC中父子容器AOP失效问题？Springboot不会，为什么？

ContextLoaderListener

Spring MVC中通过DispatherServlet构建上下文

Springboot中DispatherServlet作为一个Bean，DispatcherServletAutoConfiguration



1、ClassPathXmlApplicationContext



# SpringBoot嵌入式容器

## 传统servlet容器

### Eclipse Jetty

​	Eclipse Jetty provides a Web server and javax.servlet container, plus support for HTTP/2, WebSocket, OSGi, JMX, JNDI, JAAS and many other integrations. These components are open source and available for commercial use and distribution.

- Asynchronous HTTP Server
- Standards based Servlet Container
- websocket server
- http/2 server
- Asynchronous Client (http/1.1, http/2, websocket)
- OSGI, JNDI, JMX, JASPI, AJP support



### Apache Tomcat

​	Tomcat是由Apache软件基金会下属的Jakarta项目开发的一个Servlet容器，按照Sun Microsystems提供的技术规范，实现了对Servlet和JavaServer Page（JSP）的支持，并提供了作为Web服务器的一些特有功能，如Tomcat管理和控制平台、安全域管理和Tomcat阀等。由于Tomcat本身也内含了一个HTTP服务器，它也可以被视作一个单独的Web服务器。

#### 特性

- 标准实现：
  - Servlet
  - JSP
  - Expression Language
  - WebSocket

- 核心组件

  - Engine

  - Host

  - Context

    - config路径下的context.xml文件定义了Context初始化信息通过两个xml文件配置，WEB-INF/web.xml（应用）会覆盖tomcat中的全局配置。

      ```xml
      <?xml version="1.0" encoding="UTF-8"?>
      <!--
        Licensed to the Apache Software Foundation (ASF) under one or more
        contributor license agreements.  See the NOTICE file distributed with
        this work for additional information regarding copyright ownership.
        The ASF licenses this file to You under the Apache License, Version 2.0
        (the "License"); you may not use this file except in compliance with
        the License.  You may obtain a copy of the License at
      
            http://www.apache.org/licenses/LICENSE-2.0
      
        Unless required by applicable law or agreed to in writing, software
        distributed under the License is distributed on an "AS IS" BASIS,
        WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
        See the License for the specific language governing permissions and
        limitations under the License.
      --><!-- The contents of this file will be loaded for each web application --><Context>
      
          <!-- Default set of monitored resources. If one of these changes, the    -->
          <!-- web application will be reloaded.                                   -->
          <WatchedResource>WEB-INF/web.xml</WatchedResource>
          <WatchedResource>${catalina.base}/conf/web.xml</WatchedResource>
      
          <!-- Uncomment this to disable session persistence across Tomcat restarts -->
          <!--
          <Manager pathname="" />
          -->
      </Context>
      ```

      

- 静态资源处理

  - Tomcat实现中处理静态资源的默认实现是DefaultServlet，config目录下web.xml有如下配置：

    ```xml
    <servlet>
        <servlet-name>default</servlet-name>
        <servlet-class>org.apache.catalina.servlets.DefaultServlet</servlet-class>
        <init-param>
            <param-name>debug</param-name>
            <param-value>0</param-value>
        </init-param>
        <!--   listings            Should directory listings be produced if there -->
      <!--                       is no welcome file in this directory?  [false] -->
      <!--                       WARNING: Listings for directories with many    -->
      <!--                       entries can be slow and may consume            -->
      <!--                       significant proportions of server resources.   -->
        <!--listings属性，true时如果访问路径下没有欢迎页面，则显示该文件下的列表-->
        <init-param>
            <param-name>listings</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>1</load-on-startup>
    </servlet>
    <servlet-mapping>
         <servlet-name>default</servlet-name>
         <url-pattern>/</url-pattern>
    </servlet-mapping>
    ```

  - 证明访问的是静态页面：304 no-modified

  - 注意：springboot实现中加载静态页面并未经过DefaultServlet。

- JSP处理

  - Tomcat中处理JSP资源的默认实现是JspServlet，同样配置在config目录下的web.xml中：

    ```xml
    <servlet>
        <servlet-name>jsp</servlet-name>
        <servlet-class>org.apache.jasper.servlet.JspServlet</servlet-class>
        <init-param>
            <param-name>fork</param-name>
            <param-value>false</param-value>
        </init-param>
        <init-param>
            <param-name>xpoweredBy</param-name>
            <param-value>false</param-value>
        </init-param>
        <load-on-startup>3</load-on-startup>
    </servlet>
    <!-- The mappings for the JSP servlet -->
    <servlet-mapping>
        <servlet-name>jsp</servlet-name>
        <url-pattern>*.jsp</url-pattern>
        <url-pattern>*.jspx</url-pattern>
    </servlet-mapping>
    ```

- 欢迎页面：默认配置（web.xml）

  ```xml
  <!-- ==================== Default Welcome File List ===================== -->
    <!-- When a request URI refers to a directory, the default servlet looks  -->
    <!-- for a "welcome file" within that directory and, if present, to the   -->
    <!-- corresponding resource URI for display.                              -->
    <!-- If no welcome files are present, the default servlet either serves a -->
    <!-- directory listing (see default servlet configuration on how to       -->
    <!-- customize) or returns a 404 status, depending on the value of the    -->
    <!-- listings setting.                                                    -->
    <!--                                                                      -->
    <!-- If you define welcome files in your own application's web.xml        -->
    <!-- deployment descriptor, that list *replaces* the list configured      -->
    <!-- here, so be sure to include any of the default values that you wish  -->
    <!-- to use within your application.                                       -->
  
      <welcome-file-list>
          <welcome-file>index.html</welcome-file>
          <welcome-file>index.htm</welcome-file>
          <welcome-file>index.jsp</welcome-file>
      </welcome-file-list>
  ```

  可在应用中的web.xml文件中增加配置：

  ```xml
  <welcome-file-list>
      <welcome-file>index.html</welcome-file>
      <welcome-file>index.htm</welcome-file>
      <welcome-file>index.jsp</welcome-file>
      <welcome-file>default.html</welcome-file>
      <welcome-file>default.htm</welcome-file>
      <welcome-file>default.jsp</welcome-file>
    </welcome-file-list>
  ```

  新增一个test目录，新建一个default.xml文件：

  ![1567486753262](SpringBoot.assets\1567486753262.png)

  访问<http://localhost:8080/lesson-five/test/>，欢迎页面被加载，资源是静态资源

  ![1567486816607](SpringBoot.assets\1567486816607.png)

- 类加载：在原有jdk标准实现中tomcat增加了自己的两个启动类Common ClassLoader和Webapp Classloader，即用户类加载器（tomcat8.5测试结果）。

  - BootStrap Classloader
  - Extension ClassLoader
  - System ClassLoader
  - Common ClassLoader：java.net.URLClassLoader实现，加载类路径配置在$CATALINA_BASE/conf/catalina.properties
  - Webapp Classloader：默认org.apache.catalina.loader.ParallelWebappClassLoader实现，加载/WEB-INF/classes或/WEB-INF/lib中的文件
  
  可通过监听器查看类加载结构：
  
  ```java
  package com.lesson5.servlet;
  
  import javax.servlet.ServletContext;
  import javax.servlet.ServletContextEvent;
  import javax.servlet.ServletContextListener;
  
  public class ServletContextListenerImpl implements ServletContextListener{
  
  	@Override
  	public void contextInitialized(ServletContextEvent sce) {
  		ServletContext context = sce.getServletContext();
  		
  		ClassLoader c = context.getClassLoader();
  		
  		while(true) {
  			System.out.println(c.getClass().getName());
  			c = c.getParent();
  			if(c == null) {
  				break;
  			}
  		}
  	}
  
  	@Override
  	public void contextDestroyed(ServletContextEvent sce) {
  		// TODO Auto-generated method stub
  		
  	}
  
  }
  
  ```
  
  ```
  org.apache.catalina.loader.ParallelWebappClassLoader
  java.net.URLClassLoader
  sun.misc.Launcher$AppClassLoader
  sun.misc.Launcher$ExtClassLoader
  ```



### JDBC数据源

在config/context.xml中以JNDI形式配置jdbc数据源：

```xml
<Context>
    <Resource name="jdbc/TestDB" anth="Container" type="javax.sql.DataSource"
            username="root" password="123456" driverClassName="com.mysql.jdbc.Driver"
            url="jdbc:mysql://localhost:3306/?useSSL=false"/>
</Context>
```



### JNDI

​	The **Java Naming and Directory Interface** (**JNDI**) is a Java [API](https://en.wikipedia.org/wiki/Application_programming_interface) for a [directory service](https://en.wikipedia.org/wiki/Directory_service) that allows Java software clients to discover and look up data and resources (in the form of Java [objects](https://en.wikipedia.org/wiki/Object_(computer_science))) via a name. Like all [Java](https://en.wikipedia.org/wiki/Java_(programming_language)) APIs that interface with host systems, JNDI is independent of the underlying implementation. Additionally, it specifies a [service provider interface](https://en.wikipedia.org/wiki/Service_provider_interface) (SPI) that allows [directory service](https://en.wikipedia.org/wiki/Directory_service) implementations to be plugged into the framework.[[1\]](https://en.wikipedia.org/wiki/Java_Naming_and_Directory_Interface#cite_note-1) The information looked up via JNDI may be supplied by a server, a flat file, or a database; the choice is up to the implementation used.

#### 基本类型

- 资源（Resource）
- 环境（Environment）

#### 配置方式

- context.xml配置
- web.xml配置

#### 示例

新建两个作为资源：

```java
package com.lesson5.bean;

public class MyBean {
	private String foo = "Default Foo";
	private int bar = 0;
	public String getFoo() {
		return foo;
	}
	public void setFoo(String foo) {
		this.foo = foo;
	}
	public int getBar() {
		return bar;
	}
	public void setBar(int bar) {
		this.bar = bar;
	}
	@Override
	public String toString() {
		return "MyBean [foo=" + foo + ", bar=" + bar + "]";
	}
	
}

```

```java
package com.lesson5.bean;

import java.net.InetAddress;
import java.net.UnknownHostException;

public class MyBean2 {

	private InetAddress local = null;
	private InetAddress remote = null;

	public InetAddress getLocal() {
		return local;
	}

	public void setLocal(InetAddress ip) {
		local = ip;
	}

	public void setLocal(String localHost) {
		try {
			local = InetAddress.getByName(localHost);
		} catch (UnknownHostException ex) {
		}
	}

	public InetAddress getRemote() {
		return remote;
	}

	public void setRemote(InetAddress ip) {
		remote = ip;
	}

	public void host(String remoteHost) {
		try {
			remote = InetAddress.getByName(remoteHost);
		} catch (UnknownHostException ex) {
		}
	}

	@Override
	public String toString() {
		return "MyBean2 [local=" + local + ", remote=" + remote + "]";
	}
	
	

}

```

在config/context.xml中定义资源：

```xml
   	<Resource name="jdbc/TestDB" auth="Container" type="javax.sql.DataSource"
   		username="root" password="123456" driverClassName="com.mysql.jdbc.Driver"
   		url="jdbc:mysql://localhost:3306/?useSSL=false"/>
   		
   	<Resource name="bean/MyBeanFactory" auth="Container"
   	 type="com.lesson5.bean.MyBean" factory="org.apache.naming.factory.BeanFactory" bar="23"/>
   	 
   	 <!-- 赋值方法默认取跟字段同类型的赋值方法，会抛出NamingException-->
   	 <!-- forceString可指定赋值方法（参数为String类型），只写字段名称方法为get方法 -->
   	 <Resource name="bean2/MyBeanFactory" auth="Container"
   	 type="com.lesson5.bean.MyBean2" factory="org.apache.naming.factory.BeanFactory"
   	 forceString="local,remote=host" local="localhost" remote="tomcat.apache.org"/>
```

应用web.xml文件中增加文档说明：

```xml
<!-- 可不配置，文档描述作用 --> 
<!-- Resource -->
  <resource-ref>
  	<res-ref-name>jdbc/TestDB</res-ref-name>
  	<res-type>javax.sql.DataSource</res-type>
  	<res-auth>Container</res-auth>
  </resource-ref>
  
  <!-- Environment -->
  <env-entry>
  	<env-entry-name>Bean</env-entry-name>
  	<env-entry-type>java.lang.String</env-entry-type>
  	<env-entry-value>Hello,World</env-entry-value>
  </env-entry>
  
<!-- 同样起描述作用,与resource-ref类似 -->
  <resource-env-ref>
  	<description>Object Factory for MyBean Instance</description>
  	<resource-env-ref-name>bean/MyBeanFactory</resource-env-ref-name>
  	<resource-env-ref-type>com.lesson5.bean.MyBean</resource-env-ref-type>
  </resource-env-ref>
```

应用程序中调用资源：

```java
package com.lesson5.servlet;

import java.io.IOException;
import java.io.Writer;
import java.sql.Connection;
import java.sql.ResultSet;
import java.sql.SQLException;
import java.sql.Statement;

import javax.naming.Context;
import javax.naming.InitialContext;
import javax.naming.NamingException;
import javax.servlet.ServletConfig;
import javax.servlet.ServletException;
import javax.servlet.http.HttpServlet;
import javax.servlet.http.HttpServletRequest;
import javax.servlet.http.HttpServletResponse;
import javax.sql.DataSource;

import com.lesson5.bean.MyBean;
import com.lesson5.bean.MyBean2;

public class JdbcTestServlet extends HttpServlet{

	/**
	 * 
	 */
	private static final long serialVersionUID = -4822714901050365617L;
	
	private DataSource dataSource;
	
	@Override
    public void init(ServletConfig config) throws ServletException {
        try {
			Context context = new InitialContext();
			Context evnContext = (Context)context.lookup("java:comp/env");
			dataSource = (DataSource) evnContext.lookup("jdbc/TestDB");
			//dataSource = (DataSource) context.lookup("java:comp/env/jdbc/TestDB");
			
			//String bean = (String) context.lookup("java:comp/env/Bean");
			String bean = (String) evnContext.lookup("Bean");
			System.out.println(bean);
			
			MyBean myBean = (MyBean) evnContext.lookup("bean/MyBeanFactory");
			System.out.println(myBean);
			
			MyBean2 myBean2 = (MyBean2) evnContext.lookup("bean2/MyBeanFactory");
			System.out.println(myBean2);
			
		} catch (NamingException e) {
			throw new ServletException(e);
		}
    }
	
	public void service(HttpServletRequest req, HttpServletResponse resp)
	        throws ServletException, IOException {
		Writer writer = resp.getWriter();
		resp.setContentType("text/html;charset=UFT-8");
		try {
			Connection con = dataSource.getConnection();
			Statement st = con.createStatement();
			ResultSet rs = st.executeQuery("show databases;");
			while(rs.next()) {
				String dataName = rs.getString(1);
				writer.write(dataName);
				writer.write("<br/>");
				writer.flush();
			}
		} catch (SQLException e) {
			throw new ServletException(e);
		}
	}

}

```

注意：此处的dataSource最终类型是org.apache.tomcat.dbcp.dbcp2.BasicDataSource，tomcat8.5 lib目录下由tomcat-dbcp.jar，可见这是它的默认实现

### 连接器Connectors

- 端口（port）
- 协议（protocol）
- 线程池（Thread pool）
- 超时时间（Timeout）

```
    <!--
    <Executor name="tomcatThreadPool" namePrefix="catalina-exec-"
        maxThreads="150" minSpareThreads="4"/>
    -->
<Connector connectionTimeout="20000" port="8080" protocol="HTTP/1.1" redirectPort="8443"/>
```



## SpringBoot嵌入式web容器



SpringBoot中支持的web嵌入式容器：

- servlet容器
  - tomcat
  - jetty
- 非servlet容器
  - undertow

自定义tomcat嵌入式容器：

```java
package com.segmentfault;

import org.apache.catalina.Context;
import org.apache.catalina.connector.Connector;
import org.apache.coyote.http11.Http11Nio2Protocol;
import org.springframework.boot.SpringApplication;
import org.springframework.boot.autoconfigure.SpringBootApplication;
import org.springframework.boot.context.embedded.ConfigurableEmbeddedServletContainer;
import org.springframework.boot.context.embedded.EmbeddedServletContainerCustomizer;
import org.springframework.boot.context.embedded.tomcat.TomcatConnectorCustomizer;
import org.springframework.boot.context.embedded.tomcat.TomcatContextCustomizer;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainer;
import org.springframework.boot.context.embedded.tomcat.TomcatEmbeddedServletContainerFactory;
import org.springframework.context.annotation.Bean;

@SpringBootApplication
public class LessonFiveDemoStarter {

    public static void main(String[] args) {
        SpringApplication.run(LessonFiveDemoStarter.class,args);
    }

    @Bean
    public static EmbeddedServletContainerCustomizer embeddedServletContainerCustomizer(){
        return new EmbeddedServletContainerCustomizer() {
            @Override
            public void customize(ConfigurableEmbeddedServletContainer container) {
                if(container instanceof TomcatEmbeddedServletContainerFactory){
                    TomcatEmbeddedServletContainerFactory factory
                            = TomcatEmbeddedServletContainerFactory.class.cast(container);
                    factory.addContextCustomizers(new TomcatContextCustomizer() {
                        @Override
                        public void customize(Context context) {
                            context.setPath("lesson-five");
                        }
                    });
                    factory.addConnectorCustomizers(new TomcatConnectorCustomizer() {
                        @Override
                        public void customize(Connector connector) {
                            connector.setPort(8090);
                            //connector.setProtocol(Http11Nio2Protocol.class.getName());
                        }
                    });
                }
            }
        };
    }



}
```

问题

1. servletContext与Spring上下文的关系

   在传统非嵌入式容器（Spring）中，ServletContext初始化后，Spring上下文是通过ContextLoaderListener加载的，而在嵌入式容器实现（Springboot）中，ServletContext作为Spring中的一个Bean初始化的。

3. Servlet为什么要实现系列化

ribbin



# Springboot数据库JDBC

jdbc参考博客：<https://blog.csdn.net/qq_34626097/article/list/3?t=1&>

## JDBC4.0(JSR-221)核心接口

- 驱动接口：java.sql.Drvier
- 驱动管理：java.sql.DriverManager
- 数据源：javax.sql.DataSource
- 数据连接：java.sql.Connection
- 执行语句：java.sql.Statement
- 查询结果集：java.sql.ResultSet
- 元数据接口：java.sql.DatabaseMetaData、java.sql.ResultSetMetaData

## 数据源（DataSource）

​	数据源是数据库连接的来源，通过DataSourced接口获取

### 类型

- 通用型数据源（javax.sql.DataSource）
  - 主要使用场景：通用性数据库，本地事务，一般用socket连接
- 分布式数据源（javax.sql.XADataSource）
  - 主要使用场景：通用型数据库，分布式事务，一般用socket连接
- 嵌入式数据源（org.springframework.jdbc.datasource.embedded.EmbeddedDatabase）
  - 主要使用场景：本地文件系统数据库，如HSQL、H2、Derby等
  - 枚举：org.springframework.jdbc.datasource.embedded.EmbeddedDatabaseType

### 示例

配置数据源信息：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

#### 通过Driver获得连接

```java
package com.segfault.controller;


import java.sql.Connection;
import java.sql.Driver;
import java.sql.DriverManager;
import java.util.Properties;

/**
 * JDBC连接测试
 */
public class JdbcUtils {

    /**
     * 通过Driver连接数据库
     * @return
     * @throws Exception
     */
    public static Connection getConnectionFordDriver() throws Exception{
        Properties properties = new Properties();
        properties.load(
                JdbcUtils.class.getClassLoader()
                        .getResourceAsStream("application.properties"));
        //通过反射实现Driver接口（这里是mysql的实现）
        Driver  driver = (Driver)Class.forName(properties.getProperty("spring.datasource.driver-class-name")).newInstance();
        Properties info = new Properties();
        info.setProperty("user",properties.getProperty("spring.datasource.username"));
        info.setProperty("password",properties.getProperty("spring.datasource.password"));
        //?useSSL=false,mysql5.x版本需要ssl校验
        Connection con = driver.connect(
                properties.getProperty("spring.datasource.url")+"?useSSL=false"
                ,info);
        return con;
    }

}

```

#### 通过DriverManager获得连接

```java
package com.segfault.controller;


import java.sql.Connection;
import java.sql.Driver;
import java.sql.DriverManager;
import java.util.Properties;

/**
 * JDBC连接测试
 */
public class JdbcUtils {

    /**
     * 通过获取DriverManager
     * com.mysql.jdbc.JDBC4Connection
     * @return
     * @throws Exception
     */
    public static Connection getConnectionForDriverManager() throws Exception{
        Properties properties = new Properties();
        properties.load(
                JdbcUtils.class.getClassLoader()
                        .getResourceAsStream("application.properties"));
        //往DriverManager注册Driver
        //可注册多个驱动
        Class.forName(properties.getProperty("spring.datasource.driver-class-name"));
        Connection con = DriverManager.getConnection(
                properties.getProperty("spring.datasource.url")+"?useSSL=false",
                properties.getProperty("spring.datasource.username"),
                properties.getProperty("spring.datasource.password"));
        return con;
    }

    public static void main(String[] args) throws Exception {
        Connection con = JdbcUtils.getConnectionForDriverManager();
        System.out.println(con);
    }
}

```

这里Class.forName怎么把Driver注册到DriverManager的，原因是Driver的静态初始块代码：

```java
//
// Register ourselves with the DriverManager
//
static {
    try {
        java.sql.DriverManager.registerDriver(new Driver());
    } catch (SQLException E) {
        throw new RuntimeException("Can't register driver!");
    }
}
```

#### 通过DataSource获得连接和DatabaseMetaData使用

配置DataSource连接信息：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
```

controller注入DataSource：

```java
package com.segfault.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RestController;

import javax.sql.DataSource;
import java.sql.Connection;
import java.sql.DatabaseMetaData;
import java.sql.SQLException;
import java.util.HashMap;
import java.util.Map;

@RestController
public class JdbcController {

    @Autowired
    private DataSource dataSource;

    /**
     * 获取一些数据库元信息
     * @return
     */
    @GetMapping("/jdbc/meta")
    public Map<String,Object> metaData(){
        Connection connection = null;
        Map<String,Object> map = new HashMap<>();
        try {
            connection = dataSource.getConnection();
            DatabaseMetaData metaData = connection.getMetaData();
            map.put("supportsTransactions",metaData.supportsTransactions());
            map.put("username",metaData.getUserName());
            map.put("url",metaData.getURL());
            map.put("maxConnections",metaData.getMaxConnections());
        } catch (SQLException e) {
            throw new RuntimeException(e);
        }
        return map;
    }
}
```

请求结果：

```json
{
    supportsTransactions: true,
    url: "jdbc:mysql://localhost:3306/test",
    username: "root@localhost",
    maxConnections: 0
}
```

​	默认org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration.Tomcat#dataSource负责配置DataSource，实际的DataSource是org.apache.tomcat.jdbc.pool.DataSource类型。此类在org.apache.tomcat.jdbc包中，tomcat8.5 lib目录下有这个包。

```java
/**
 * Tomcat Pool DataSource configuration.
 */
@Configuration
@ConditionalOnClass(org.apache.tomcat.jdbc.pool.DataSource.class)
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.tomcat.jdbc.pool.DataSource",
      matchIfMissing = true)
static class Tomcat {

   @Bean
   @ConfigurationProperties(prefix = "spring.datasource.tomcat")
   public org.apache.tomcat.jdbc.pool.DataSource dataSource(DataSourceProperties properties) {
      org.apache.tomcat.jdbc.pool.DataSource dataSource = createDataSource(properties,
            org.apache.tomcat.jdbc.pool.DataSource.class);
      DatabaseDriver databaseDriver = DatabaseDriver.fromJdbcUrl(properties.determineUrl());
      String validationQuery = databaseDriver.getValidationQuery();
      if (validationQuery != null) {
         dataSource.setTestOnBorrow(true);
         dataSource.setValidationQuery(validationQuery);
      }
      return dataSource;
   }

}
```

```java
@Configuration
@Conditional(PooledDataSourceCondition.class)
@ConditionalOnMissingBean({ DataSource.class, XADataSource.class })
@Import({ DataSourceConfiguration.Tomcat.class, DataSourceConfiguration.Hikari.class,
      DataSourceConfiguration.Dbcp.class, DataSourceConfiguration.Dbcp2.class,
      DataSourceConfiguration.Generic.class })
@SuppressWarnings("deprecation")
protected static class PooledDataSourceConfiguration {

}
```



#### 实现dbcp数据连接池

增加dbcp连接池配置：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
##默认datasource是org.apache.tomcat.jdbc.pool.DataSource（tomcat实现的连接池）
##指定dbcp的datasource实现
spring.datasource.type=org.apache.commons.dbcp.BasicDataSource
spring.datasource.dbcp.max-active=20
spring.datasource.dbcp.max-idle=5
spring.datasource.dbcp.validation-query=select * from user;
```

此时dataSource类型为org.apache.commons.dbcp.BasicDataSource

初始化方法：

org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration.Dbcp#dataSource

```java
	/**
	 * DBCP DataSource configuration.
	 *
	 * @deprecated as of 1.5 in favor of DBCP2
	 */
	@Configuration
	@ConditionalOnClass(org.apache.commons.dbcp.BasicDataSource.class)
	@ConditionalOnMissingBean(DataSource.class)
	@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.commons.dbcp.BasicDataSource",
			matchIfMissing = true)
	@Deprecated
	static class Dbcp {

		@Bean
		@ConfigurationProperties(prefix = "spring.datasource.dbcp")
		public org.apache.commons.dbcp.BasicDataSource dataSource(DataSourceProperties properties) {
			org.apache.commons.dbcp.BasicDataSource dataSource = createDataSource(properties,
					org.apache.commons.dbcp.BasicDataSource.class);
			DatabaseDriver databaseDriver = DatabaseDriver.fromJdbcUrl(properties.determineUrl());
			String validationQuery = databaseDriver.getValidationQuery();
			if (validationQuery != null) {
				dataSource.setTestOnBorrow(true);
				dataSource.setValidationQuery(validationQuery);
			}
			return dataSource;
		}

	}
```

设置为dbcp2连接池：

```properties
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
##默认datasource是org.apache.tomcat.jdbc.pool.DataSource（tomcat实现的连接池）
##指定dbcp的datasource实现
#spring.datasource.type=org.apache.commons.dbcp.BasicDataSource
#spring.datasource.dbcp.max-active=20
#spring.datasource.dbcp.max-idle=5
#spring.datasource.dbcp.validation-query=select * from user;
##指定dbcp2的datasource实现
spring.datasource.type=org.apache.commons.dbcp2.BasicDataSource
spring.datasource.dbcp2.max-conn-lifetime-millis=20
spring.datasource.dbcp2.max-idle=5
spring.datasource.dbcp2.validation-query=select * from user;
```

此时dataSource为org.apache.commons.dbcp2.BasicDataSource，初始化方法为org.springframework.boot.autoconfigure.jdbc.DataSourceConfiguration.Dbcp2#dataSource：

```java
/**
 * DBCP DataSource configuration.
 */
@Configuration
@ConditionalOnClass(org.apache.commons.dbcp2.BasicDataSource.class)
@ConditionalOnMissingBean(DataSource.class)
@ConditionalOnProperty(name = "spring.datasource.type", havingValue = "org.apache.commons.dbcp2.BasicDataSource",
      matchIfMissing = true)
static class Dbcp2 {

   @Bean
   @ConfigurationProperties(prefix = "spring.datasource.dbcp2")
   public org.apache.commons.dbcp2.BasicDataSource dataSource(DataSourceProperties properties) {
      return createDataSource(properties, org.apache.commons.dbcp2.BasicDataSource.class);
   }

}
```

#### 使用JdbcTemplate和java.sql.ResultSetMetaData使用

```java
package com.segfault.controller;

import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.StatementCallback;
import org.springframework.web.bind.annotation.GetMapping;
import org.springframework.web.bind.annotation.RequestParam;
import org.springframework.web.bind.annotation.RestController;

import javax.sql.DataSource;
import java.sql.*;
import java.util.*;

@RestController
public class JdbcController {

    @Autowired
    private DataSource dataSource;

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @GetMapping("/users")
    public List<Map<String,Object>> users(){
        return jdbcTemplate.execute(new StatementCallback<List<Map<String, Object>>>() {
            @Override
            public List<Map<String, Object>> doInStatement(Statement stmt) throws SQLException, DataAccessException {
                ResultSet resultSet = stmt.executeQuery("SELECT * FROM USER ");
                ResultSetMetaData metaData = resultSet.getMetaData();
                ArrayList<String> columnNames = new ArrayList<>(metaData.getColumnCount());
                for(int i = 1; i <= metaData.getColumnCount(); i++){
                    columnNames.add(metaData.getColumnName(i));
                }

                List<Map<String,Object>> result = new LinkedList<>();

                while(resultSet.next()){
                    Map<String,Object> map = new HashMap<>();
                    for(String columnName:columnNames){
                        Object obj = resultSet.getObject(columnName);
                        map.put(columnName,obj);
                    }
                    result.add(map);
                }

                return result;
            }
        });
    }

}
```

访问<http://localhost:8080/users>结果：

```json
[
    {
    name: "tom",
    id: 1,
    age: 10
    },
    {
    name: "tom",
    id: 2,
    age: 10
    },
    {
    name: "Joy",
    id: 3,
    age: 20
    },
    {
    name: "Jetty",
    id: 4,
    age: 10
    }
]
```



## 事务

​	事务用于提供数据完整性，并在并发访问下确保数据视图的一致性。

### 重要概念

- 自动提交模式（Auto-commit mode）

  - 触发时机
    - DML（data manipulation language）执行后，如insert、delete
    - DDL（data definition language）执行后，如create、alter、drop
    - select查询后结果集关闭后
    - 存储过程执行后（如果返回结果集，等其关闭后）

- 事务四大特征和

  - 原子性
  - 一致性
  - 隔离性
    - red uncommitted
    - red committed
    - repeatable-read
    - serializable
    - 事务并发可能的影响
      - 脏读（dirty reads）
      - 不可重复读（nonrepeatable reads）
      - 幻读（phantom reads
  - 持久性

- 保护点（Savepoints）

  - 保护点是在事务中创建，提供细粒度的事务控制
  - 使用场景
    - 部分事务回滚
    - 选择性释放

  如下面的伪代码，执行完后只有一行数据被插入

  ```java
  conn.createStatement();
  int rows = stmt.executeUpdate("INSERT INTO TAB1 (COL1) VALUES " +
  "(’FIRST’)");
  // set savepoint
  Savepoint svpt1 = conn.setSavepoint("SAVEPOINT_1");
  rows = stmt.executeUpdate("INSERT INTO TAB1 (COL1) " + 
  "VALUES (’SECOND’)");
  ...
  conn.rollback(svpt1);
  ...
  conn.commit();
  ```

  

### 示例：

domin领域对象：

```java
package com.segfault.domin;

public class User {
    private int id;
    private String name;
    private int age;

    public int getId() {
        return id;
    }

    public void setId(int id) {
        this.id = id;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }
}
```

服务接口：

```java
package com.segfault.service;


import com.segfault.domin.User;

public interface UserService {
    boolean save(User user);

    boolean save2(User user);
}
```

在服务实现中使用事务：

```java
package com.segfault.service;


import com.segfault.domin.User;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.dao.DataAccessException;
import org.springframework.jdbc.core.JdbcTemplate;
import org.springframework.jdbc.core.PreparedStatementCallback;
import org.springframework.stereotype.Service;
import org.springframework.transaction.PlatformTransactionManager;
import org.springframework.transaction.TransactionStatus;
import org.springframework.transaction.annotation.EnableTransactionManagement;
import org.springframework.transaction.annotation.Transactional;
import org.springframework.transaction.support.DefaultTransactionDefinition;

import java.sql.PreparedStatement;
import java.sql.SQLException;

@Service
@EnableTransactionManagement
public class UserServiceImpl implements UserService{

    @Autowired
    private JdbcTemplate jdbcTemplate;

    @Autowired
    private PlatformTransactionManager platformTransactionManager;

    @Override
    /**
     * 声明式事务
     */
    @Transactional(rollbackFor = Exception.class)
    public boolean save(User user) {
        return jdbcTemplate.execute("INSERT INTO USER (name,age) values(?,?);", new PreparedStatementCallback<Boolean>() {
            @Override
            public Boolean doInPreparedStatement(PreparedStatement ps) throws SQLException, DataAccessException {
                ps.setString(1,user.getName());
                ps.setInt(2,user.getAge());
                return ps.executeUpdate() > 0;
            }
        });
    }

    @Override
    //@Transactional
    /**
     * 手写，通过spring的事务管理平台PlatformTransactionManager控制事务
     */
    public boolean save2(User user) {
        Boolean result = false;
        DefaultTransactionDefinition transactionDefinition = new DefaultTransactionDefinition();
        TransactionStatus transaction = platformTransactionManager.getTransaction(transactionDefinition);
        result = save(user);
        try{
            platformTransactionManager.commit(transaction);
        } catch (Exception e){
            platformTransactionManager.rollback(transaction);
        }
        return result;
    }
}
```

控制层：

```java
@PostMapping("/user/add")
@ResponseBody
public Map<String,Object> save(@RequestBody User user){
    Map<String,Object> map = new HashMap<>();
    map.put("success",userService.save(user));
    return map;
}

@PostMapping("/user/add2")
@ResponseBody
public Map<String,Object> save2(@RequestBody User user){
    Map<String,Object> map = new HashMap<>();
    map.put("success",userService.save2(user));
    return map;
}
```



savepoint示例：

```java
/**
 * savepoint示例
 * @return
 */
@GetMapping("/jdbc/savepoint")
public Map<String,Object> savePoint(){
    Connection connection = null;
    Map<String,Object> map = new HashMap<>();
    Savepoint savepoint = null;
    try {
        connection = dataSource.getConnection();
        connection.setAutoCommit(false);
        savepoint = connection.setSavepoint();

        DatabaseMetaData metaData = connection.getMetaData();
        map.put("supportsTransactions",metaData.supportsTransactions());
        map.put("username",metaData.getUserName());
        map.put("url",metaData.getURL());
        map.put("maxConnections",metaData.getMaxConnections());
        connection.commit();
    } catch (SQLException e) {
        if(connection != null){
            try {
                connection.rollback(savepoint);
            } catch (SQLException e1) {
                e1.printStackTrace();
            }
        }
        throw new RuntimeException(e);
    } finally {
        try {
            connection.setAutoCommit(true);
            connection.close();
        } catch (SQLException e) {
            e.printStackTrace();
        }
    }
    return map;
}
```



reactive

ScriptEngine

@Transationl 和TransationInterceptor

TransationAspectSupport



问题

c3p0和dbcp

- c3p0有自动关闭空闲连接机制，dbcp没有

分布式事务

代理



# springboot-MyBaties

## 简介

​	MyBatis is a first class persistence framework with support for custom SQL, stored procedures and advanced mappings. MyBatis eliminates almost all of the JDBC code and manual setting of parameters and retrieval of results. MyBatis can use simple XML or Annotations for configuration and map primitives, Map interfaces and Java POJOs (Plain Old Java Objects) to database records.

​	mybatis前身是ibatis。

## 通过读取xml文件中定义的sql查询

## 通过mybatis-generator逆向工程生成的Temple查询

### mybatis-generator逆向工程

​	就是通过逆向工程，自动生成mybatis所需的一些实体，mapper类等。

官网：http://www.mybatis.org/generator/index.html

引入maven插件

```xml
    <build>
        <plugins>
            <plugin>
                <groupId>org.mybatis.generator</groupId>
                <artifactId>mybatis-generator-maven-plugin</artifactId>
                <version>1.3.7</version>
            </plugin>
        </plugins>
    </build>
```



## 通过Annotion定义查询



## SpringBoot集成mybatis

添加依赖：

```properties
<dependency>
    <groupId>org.mybatis.spring.boot</groupId>
    <artifactId>mybatis-spring-boot-starter</artifactId>
    <version>1.3.0</version>
</dependency>
```

配置：

```properties
server.port=8080
mybatis.config-location= classpath:/mybatis/mybatis-config.xml
mybatis.check-config-location=true
mybatis.executor-type=SIMPLE
spring.datasource.driver-class-name=com.mysql.jdbc.Driver
spring.datasource.url=jdbc:mysql://localhost:3306/test
spring.datasource.username=root
spring.datasource.password=123456
```

controller调用

```java
package com.segfault.controller;

import com.segfault.entity.User;
import org.apache.ibatis.session.SqlSessionFactory;
import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.web.bind.annotation.PathVariable;
import org.springframework.web.bind.annotation.RequestMapping;
import org.springframework.web.bind.annotation.RestController;

@RestController
public class MybatisController {

    //通过spring组装SqlSessionTemplate
    @Autowired
    private SqlSessionTemplate sqlSessionTemplate;

    @RequestMapping("/mybatis/user/{id}")
    public User getUser(@PathVariable int id){
        User user = sqlSessionTemplate.selectOne("mybatis.mapper.UserMapper.selectUser",id);
        return user;
    }
}

```



## pligins(拦截器）

- Executor 
- ParameterHandler 
- ResultSetHandler 
- StatementHandler 

如：

```java
// ExamplePlugin.java
@Intercepts({@Signature(
  type= Executor.class,
  method = "update",
  args = {MappedStatement.class,Object.class})})
public class ExamplePlugin implements Interceptor {
  private Properties properties = new Properties();
  public Object intercept(Invocation invocation) throws Throwable {
    // implement pre processing if need
    Object returnObject = invocation.proceed();
    // implement post processing if need
    return returnObject;
  }
  public void setProperties(Properties properties) {
    this.properties = properties;
  }
}
```

```xml
<!-- mybatis-config.xml -->
<plugins>
  <plugin interceptor="org.mybatis.example.ExamplePlugin">
    <property name="someProperty" value="100"/>
  </plugin>
</plugins>
```

通过plugins自定义分页：

```java
package com.segfault.interceptor;


import org.apache.ibatis.executor.parameter.ParameterHandler;
import org.apache.ibatis.executor.statement.StatementHandler;
import org.apache.ibatis.mapping.MappedStatement;
import org.apache.ibatis.plugin.*;
import org.apache.ibatis.reflection.MetaObject;
import org.apache.ibatis.reflection.SystemMetaObject;
import org.mybatis.spring.SqlSessionTemplate;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.context.annotation.Configuration;

import java.lang.reflect.ParameterizedType;
import java.sql.Connection;
import java.util.Map;
import java.util.Properties;

/**
 * 分页处理拦截器
 * @Intercepts 声明是拦截器
 * @Signature 签名配置
 *  type:
 */
@Intercepts({@Signature(type = StatementHandler.class, method = "prepare", args = {Connection.class,Integer.class})})
public class PageInterceptor implements Interceptor {


    private Integer pageSize;
    private String dbType;

    @Override
    public Object intercept(Invocation invocation) throws Throwable {

        //获得StatementHandler
        StatementHandler statementHandler = (StatementHandler)invocation.getTarget();
        //StatementHandler包装类
        MetaObject metaObjectHandler = SystemMetaObject.forObject(statementHandler);

        while(metaObjectHandler.hasGetter("h")){
            Object obj = metaObjectHandler.getValue("h");
            metaObjectHandler = SystemMetaObject.forObject(obj);
        }
        while(metaObjectHandler.hasGetter("target")){
            Object obj = metaObjectHandler.getValue("target");
            metaObjectHandler = SystemMetaObject.forObject(obj);
        }

        //获取查询接口映射的相关信息
        MappedStatement mappedStatement =
                (MappedStatement) metaObjectHandler.getValue("delegate.mappedStatement");
        String mapId = mappedStatement.getId();

        if(mapId.matches(".+ByPageParam$")){
            ParameterHandler parameterHandler =
                    (ParameterHandler)metaObjectHandler.getValue("delegate.parameterHandler");
            Map param = (Map) parameterHandler.getParameterObject();
            int start = (Integer) param.get("start");
            int limit = (Integer) param.get("limit");
            String sql = ((String)metaObjectHandler.getValue("delegate.boundSql.sql")).trim();
            sql = sql + " limit " + start + "," + limit;
            metaObjectHandler.setValue("delegate.boundSql.sql",sql);
        }

        return invocation.proceed();
    }

    /**
     * 获取代理对象
     * @param target
     * @return
     */
    @Override
    public Object plugin(Object target) {
        return Plugin.wrap(target,this);
    }

    @Override
    public void setProperties(Properties properties) {
        String limit1 = properties.getProperty("limit", "10");
        this.pageSize = Integer.valueOf(limit1);
        this.dbType = properties.getProperty("dbType", "mysql");
    }
}

```



## 分页

- 内存分页：即先查询到内存中，再分页，效率较低，如RowBounds
- 物理分页：sql语句中加入分页语句，效率仅仅取决于sql。

批量插入

一对多，多对多查询

常见面试题：







问题

包扫描方式



# SpringBoot-JPA

## Java Persistence API

### 介绍

- JPA1.0整合查询语言（Query）和对象关系映射（ORM）元数据定义。
- JPA2.0在1.0的基础上，增加了Criteria、元数据API以及校验支持。

### 实体相关属性配置

- 实体（Entity）：轻量级持久化域对象

- 实体类（Entity Class）：实体类可能用于辅助类或者状态
  - 约束
    - 实体类支持继承、多态关联以及多态查询
    - 实体类必须用@Entity标注或者XML描述
    - 实体类必须包含一个默认构造器，并且构造器是public或者protected
    - 实体类必须是顶级类，不能是枚举或者接口
    - 实体类禁止是final类
  
- 实体持久字段和属性

  - 实体持久状态由字段（Field）或者属性（Properties）表示
    - 字段：实例的属性或者变量
    - 属性：JavaBean实例的setter或getter方法
  - 实例属性的访问性必须是private、protedted或者包可见；属性的可见性必须是public或者protedted
  - 字段或属性可能是单一类型值或者集合类型值

- 持久字段和属性类型

  - 原生类型
  - java serializable类型
  - 自定义类型（实现serializable接口）
  - 枚举
  - 实体类型（包括集合实体类型）
  - 嵌入类型

- 字段和属性访问类型（Access Type）

  - 默认访问类型

    - 非transient或者非@Transient字段
    - 非@Transient属性

  - 显示访问类型

    - 注解类型

      - 实体类
      - 映射类
      - 嵌套类

    - 注解

      - @Access(value = AccessType.FIELD)字段

      - @Access(value = AccessType.PROPERTY)属性

- 实体主键：每个实体必须存在主键，主键必须定义在实体类

  - 简单主键

    - @Id

  - 复合主键

    - @Embeddable

      ```java
      @Embeddable
      public class EmployeeId {
       String firstName;
       String lastName;
       ...
      }
      @Entity
      public class Employee {
       @EmbeddedId EmployeeId empId;
       ...
      }
      ```

    - @IdClass

    ```java
    package com.segfault.entity;
    
    import javax.persistence.Entity;
    import javax.per sistence.Id;
    
    @Entity
    public class Employee {
    
        @Id long enmId;
    
    
        String empName;
    
    }
    
    ```

    ```java
    package com.segfault.entity;
    
    import javax.persistence.Entity;
    import javax.persistence.Id;
    import javax.persistence.IdClass;
    import javax.persistence.ManyToOne;
    import java.io.Serializable;
    
    
    @Entity
    @IdClass(DependentId.class)
    public class Dependent {
        @Id
        String name;
    
        @Id
        @ManyToOne
        Employee emp;
    }
    
    class DependentId implements Serializable{
        //映射到具体类字段
        String name;
        long emp;
    }
    ```

    生成语句：

    ```sql
    CREATE TABLE `dependent` (
      `name` varchar(255) NOT NULL,
      `emp_enm_id` bigint(20) NOT NULL,
      PRIMARY KEY (`emp_enm_id`,`name`),
      CONSTRAINT `FK12ab73h9ykm360drrkfpxonjx` FOREIGN KEY (`emp_enm_id`) REFERENCES `employee` (`enm_id`)
    ) ENGINE=InnoDB DEFAULT CHARSET=latin1
    ```

```java
package com.segfault.entity;


import javax.persistence.*;

@Entity
@Access(value = AccessType.FIELD)
@Table(name = "customers")
public class Customer {

    @Id
    @GeneratedValue
    private Long id;

    @Column(length = 64)
    private String name;


/*    @Id
    @GeneratedValue*/
    //对应AccessType.PROPERTY
    public Long getId() {
        return id;
    }


    public void setId(Long id) {
        this.id = id;
    }

//    @Column(length = 64)
    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}
```

- 实体关系：可表示一对一、一对多、多对一、多对多，这些关系是多态的。可以是单向的或者双向的。

  - 注解表示方式

    - @OnoToOne
    - @OneToMany
    - @ManyToOne
    - @ManyToMany

  - XML表示方式

    - EmbeddedId
    - IdClass

  - 实体双向关系：实体双向关系是指两实体之间不仅存在拥有方（owning），也存在倒转方（inverse）。主方决定了更新级联关系到数据库

    - 规则

      - 倒转必须通过@OneToOne、@OneToMany、@ManyToMany中的mappedBy属性方法关联到**拥有方**的字段或者属性。
      - 一对多、多对一双向关系中的多方必须是主方，因此@ManyToOne注解不能指定mappedBy注解
      - 双向一对一关系中，主方相当于包含外键的一方
      - 双向多对多关系中，任何一方可能是拥有方

    - 一对一（OneToOne）

      - 假设
        - 实体A引用单个实体B的实例
        - 实体B引用单个实体A的实例
        - 实体A在关系中处于拥有方
      - 默认映射
        - 实体A被映射为数据库一张表
        - 实体B被映射为数据库一张表
        - 表A中包含一个外键关联B
      - 举例：客户、信用卡，客户为拥有方

      ```java
      package com.segfault.entity;
      
      
      import javax.persistence.*;
      
      /**
       * 客户类
       */
      @Entity
      @Access(value = AccessType.FIELD)
      @Table(name = "customers")
      public class Customer {
      
          @Id
          @GeneratedValue
          private Long id;
      
          @Column(length = 64)
          private String name;
      
          @OneToOne
          private CreditCard creditCard;
      
      
      /*    @Id
          @GeneratedValue*/
          //对应AccessType.PROPERTY
          public Long getId() {
              return id;
          }
      
      
          public void setId(Long id) {
              this.id = id;
          }
      
      //    @Column(length = 64)
          public String getName() {
              return name;
          }
      
          public void setName(String name) {
              this.name = name;
          }
      
          public CreditCard getCreditCard() {
              return creditCard;
          }
      
          public void setCreditCard(CreditCard creditCard) {
              this.creditCard = creditCard;
          }
      }
      
      ```

      ```java
      package com.segfault.entity;
      
      import javax.persistence.*;
      import java.util.Date;
      
      /**
       * 信用卡类
       */
      @Entity
      @Table(name = "credit_cards")
      public class CreditCard {
          @Id
          @GeneratedValue
          private Long id;
      
          @Column(length = 128)
          private String number;
      
          @Column(name = "register_date")
          private Date regiteredDate;
      
          @OneToOne(mappedBy = "creditCard",cascade = CascadeType.REMOVE)
          private Customer customer;
      
          public Long getId() {
              return id;
          }
      
          public void setId(Long id) {
              this.id = id;
          }
      
          public String getNumber() {
              return number;
          }
      
          public void setNumber(String number) {
              this.number = number;
          }
      
          public Date getRegiteredDate() {
              return regiteredDate;
          }
      
          public void setRegiteredDate(Date regiteredDate) {
              this.regiteredDate = regiteredDate;
          }
      
          public Customer getCustomer() {
              return customer;
          }
      
          public void setCustomer(Customer customer) {
              this.customer = customer;
          }
      }
      
      ```

      生成的语句：

      ```sql
      CREATE TABLE `customers` (
        `id` bigint(20) NOT NULL AUTO_INCREMENT,
        `name` varchar(64) DEFAULT NULL,
        `credit_card_id` bigint(20) DEFAULT NULL,
        PRIMARY KEY (`id`),
        KEY `FKboqlm1lu6kfqo3xxmstxrwd3d` (`credit_card_id`),
        CONSTRAINT `FKboqlm1lu6kfqo3xxmstxrwd3d` FOREIGN KEY (`credit_card_id`) REFERENCES `credit_cards` (`id`)
      ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
      
      
      CREATE TABLE `credit_cards` (
        `id` bigint(20) NOT NULL AUTO_INCREMENT,
        `number` varchar(128) DEFAULT NULL,
        `register_date` datetime DEFAULT NULL,
        PRIMARY KEY (`id`)
      ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
      ```

      > 主方即是创建外键那一方

    - 多对一（ManyToOne）/一对多（OneToMany）

      - 假设
        - 实体A引用实体B单个实例
        - 实体B引用实体A多个实例
        - 实体A在关系中处于拥有方
      - 默认映射
        - 实体A被映射为数据库一张表
        - 实体B被映射为数据库一张表
        - 表A中包含一个外键关联B
      - 举例：店铺（一）与客户（多）的关系

      ```java
      package com.segfault.entity;
      import javax.persistence.*;
      
      /**
       * 客户类
       */
      @Entity
      @Access(value = AccessType.FIELD)
      @Table(name = "customers")
      public class Customer {
      
          @Id
          @GeneratedValue
          private Long id;
      
          @Column(length = 64)
          private String name;
      
          @OneToOne
          private CreditCard creditCard;
      
          @ManyToOne
          private Store store;
      
      
      /*    @Id
          @GeneratedValue*/
          //对应AccessType.PROPERTY
          public Long getId() {
              return id;
          }
      
      
          public void setId(Long id) {
              this.id = id;
          }
      
      //    @Column(length = 64)
          public String getName() {
              return name;
          }
      
          public void setName(String name) {
              this.name = name;
          }
      
          public CreditCard getCreditCard() {
              return creditCard;
          }
      
          public void setCreditCard(CreditCard creditCard) {
              this.creditCard = creditCard;
          }
      
          public Store getStore() {
              return store;
          }
      
          public void setStore(Store store) {
              this.store = store;
          }
      }
      
      ```

      ```java
      package com.segfault.entity;
      
      import javax.persistence.*;
      import java.util.Collection;
      
      /**
       * 店铺类
       */
      @Entity
      @Table(name = "stores")
      public class Store {
          @Id
          @GeneratedValue
          private Long id;
      
          private String name;
      
          @OneToMany(mappedBy = "store")
          private Collection<Customer> customers;
      
          public Long getId() {
              return id;
          }
      
          public void setId(Long id) {
              this.id = id;
          }
      
          public String getName() {
              return name;
          }
      
          public void setName(String name) {
              this.name = name;
          }
      
          public Collection<Customer> getCustomers() {
              return customers;
          }
      
          public void setCustomers(Collection<Customer> customers) {
              this.customers = customers;
          }
      }
      
      ```

      生成语句：

      ```sql
      CREATE TABLE `customers` (
        `id` bigint(20) NOT NULL AUTO_INCREMENT,
        `name` varchar(64) DEFAULT NULL,
        `credit_card_id` bigint(20) DEFAULT NULL,
        `store_id` bigint(20) DEFAULT NULL,
        PRIMARY KEY (`id`),
        KEY `FKboqlm1lu6kfqo3xxmstxrwd3d` (`credit_card_id`),
        KEY `FKfo2nbino9sutm98o6hbac74ue` (`store_id`),
        CONSTRAINT `FKboqlm1lu6kfqo3xxmstxrwd3d` FOREIGN KEY (`credit_card_id`) REFERENCES `credit_cards` (`id`),
        CONSTRAINT `FKfo2nbino9sutm98o6hbac74ue` FOREIGN KEY (`store_id`) REFERENCES `stores` (`id`)
      ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
      
      
      CREATE TABLE `stores` (
        `id` bigint(20) NOT NULL AUTO_INCREMENT,
        `name` varchar(255) DEFAULT NULL,
        PRIMARY KEY (`id`)
      ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
      ```

    - 多对多（ManyToMany）

      - 假设
        - 实体A引用实体B多个实例
        - 实体B引用实体A多个实例
        - 实体A在关系中处于拥有方
      - 默认映射
        - 实体A被映射到一张表A
        - 实体B被映射到一张表B
        - 存在一个名为A_B的数据表（拥有方表名为前缀），其中包含两个外键列，一个映射到A表的主键列，一个映射到B表的主键列。
      - 举例

      ```java
      package com.segfault.entity;
      import javax.persistence.*;
      import java.util.Collection;
      
      /**
       * 客户类
       */
      @Entity
      @Access(value = AccessType.FIELD)
      @Table(name = "customers")
      public class Customer {
      
          @Id
          @GeneratedValue
          private Long id;
      
          @Column(length = 64)
          private String name;
      
          @OneToOne
          private CreditCard creditCard;
      
          @ManyToOne
          private Store store;
      
          @ManyToMany
          private Collection<Book> books;
      
      
      /*    @Id
          @GeneratedValue*/
          //对应AccessType.PROPERTY
          public Long getId() {
              return id;
          }
      
      
          public void setId(Long id) {
              this.id = id;
          }
      
      //    @Column(length = 64)
          public String getName() {
              return name;
          }
      
          public void setName(String name) {
              this.name = name;
          }
      
          public CreditCard getCreditCard() {
              return creditCard;
          }
      
          public void setCreditCard(CreditCard creditCard) {
              this.creditCard = creditCard;
          }
      
          public Store getStore() {
              return store;
          }
      
          public void setStore(Store store) {
              this.store = store;
          }
      }
      
      ```

      ```java
      package com.segfault.entity;
      
      import javax.persistence.*;
      import java.util.Collection;
      
      /**
       * 店铺类
       */
      @Entity
      @Table(name = "stores")
      public class Store {
          @Id
          @GeneratedValue
          private Long id;
      
          private String name;
      
          @OneToMany(mappedBy = "store")
          private Collection<Customer> customers;
      
          public Long getId() {
              return id;
          }
      
          public void setId(Long id) {
              this.id = id;
          }
      
          public String getName() {
              return name;
          }
      
          public void setName(String name) {
              this.name = name;
          }
      
          public Collection<Customer> getCustomers() {
              return customers;
          }
      
          public void setCustomers(Collection<Customer> customers) {
              this.customers = customers;
          }
      }
      
      ```

      生成语句：

      ```sql
      
      CREATE TABLE `customers` (
        `id` bigint(20) NOT NULL AUTO_INCREMENT,
        `name` varchar(64) DEFAULT NULL,
        `credit_card_id` bigint(20) DEFAULT NULL,
        `store_id` bigint(20) DEFAULT NULL,
        PRIMARY KEY (`id`),
        KEY `FKboqlm1lu6kfqo3xxmstxrwd3d` (`credit_card_id`),
        KEY `FKfo2nbino9sutm98o6hbac74ue` (`store_id`),
        CONSTRAINT `FKboqlm1lu6kfqo3xxmstxrwd3d` FOREIGN KEY (`credit_card_id`) REFERENCES `credit_cards` (`id`),
        CONSTRAINT `FKfo2nbino9sutm98o6hbac74ue` FOREIGN KEY (`store_id`) REFERENCES `stores` (`id`)
      ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
      
      
      CREATE TABLE `books` (
        `id` bigint(20) NOT NULL,
        `name` varchar(255) DEFAULT NULL,
        PRIMARY KEY (`id`)
      ) ENGINE=InnoDB DEFAULT CHARSET=latin1;
      
      CREATE TABLE `customers_books` (
        `customers_id` bigint(20) NOT NULL,
        `books_id` bigint(20) NOT NULL,
        KEY `FKjg5i5btvybmhsrm4y8sj1axxk` (`books_id`),
        KEY `FKs9thot7dft06jal77j312isnt` (`customers_id`),
        CONSTRAINT `FKjg5i5btvybmhsrm4y8sj1axxk` FOREIGN KEY (`books_id`) REFERENCES `books` (`id`),
        CONSTRAINT `FKs9thot7dft06jal77j312isnt` FOREIGN KEY (`customers_id`) REFERENCES `customers` (`id`)
      ) ENGINE=InoDB DEFAULT CHARSET=latin1;
      ```

  - 实体单向关系
    - 一对一
      -  假设：
        - 实体A引用单个实体B的实例
        - 实体B没有引用单个实体A的实例
        - 实体A在关系中处于拥有方
      - 默认映射
        - 实体A被映射到数据表A
        - 实体B被映射到数据表B
        - 表A包含一个外键关联B
      - 举例：客户（Customer）与信用卡（Credit Card）的关系
    - 一对多
      - 假设
        - 实体A引用多个实体B的实例
        - 实体B没有引用单个实体A的实例
        - 实体A在关系中处于拥有方
      - 默认映射
        - 实体A被映射到数据表A
        - 实体B被映射到数据表B
        - 存在一个名为A_B的关联表（拥有方表名为前缀），其中包含两个外键列，一列关联表A的主键，另外一列关联表B的主键。
    - 多对多
      - 假设
        - 实体A引用多个实体B的实例
        - 实体B没有引用单个实体A的实例
        - 实体A在关系中处于拥有方
      - 默认映射
        - 实体A被映射到数据表A
        - 实体B被映射到数据表B
        - 存在一个名为A_B的关联表（拥有方表名为前缀），其中包含两个外键列，一列关联表A的主键，另外一列关联表B的主键。

- 实体继承：实体可继承其他实体。实体之间支持继承、多态关联、多态查询

  - 继承方式
    - 继承抽象实体类
      - @Inheritance
    - 继承已映射的父类型
      - @MappedSuperclass
      - @AssociationOverride
    - 继承非实体类型

- 实体操作

  - 实体管理器EntityManager

  ```java
  package com.segfault.service;
  
  import com.segfault.entity.Customer;
  import org.springframework.beans.factory.annotation.Autowired;
  import org.springframework.stereotype.Service;
  import org.springframework.transaction.annotation.Transactional;
  
  import javax.persistence.EntityManager;
  import javax.persistence.PersistenceContext;
  
  @Service
  public class CustomerService {
  
      //JPA标准实现
      //持久化上下文，type默认PersistenceContextType.TRANSACTION
      @PersistenceContext
      private EntityManager entityManager;
  
      /**
       * 添加客户
       * @param customer
       */
      @Transactional(rollbackFor = Exception.class)
      public void addCustomer(Customer customer){
          entityManager.persist(customer);
      }
  
      /**
       * 查询客户
       * @param id
       * @return
       */
      public Customer findCustomer(Long id){
          return entityManager.find(Customer.class,id);
      }
  
  }
  
  ```



### 实体实例的生命周期

- 创建
- 持久化
- 移除
- 同步到数据库
- 刷新实例
- 淘汰



### 持久化上下文使用期限

类型

```java
public enum PersistenceContextType {

	/**
	 * Transaction-scoped persistence context
	 */
	TRANSACTION,

	/**
	 * Extended persistence context
	 */
	EXTENDED
}
```

- 事务类型（默认）
- 拓展类型

阶段

- 事务提交阶段
  - 事务类型：实体状态->托管（即实体实例会在PersistenceContext中删除）
  - 拓展类型：实体状态->继续维持（不删除实体实例）
- 事务回滚阶段
  - 实体状态->托管（删除实体实例）

#### 对比Hiberbate中对象的四种状态

1. 临时状态（新建状态）：没保存在数据库中之前
2. 持久化状态（托管状态）：数据库中存在，session中存在
3. 游离状态（脱管）：数据库中有，session中不存在
4. 删除状态：数据库中没有，session缓存中没有

![1568087722049](SpringBoot.assets\1568087722049.png)



### 实体监听器和回调方法

- 实体监听器

  - @EntityListeners

- 回调方法

  - @PrePersist
  - @PostPersist
  - @PreRemove
  - @PostRemove
  - @PreUpdate
  - @PostUpdate
  - @PostLoad

- 例子：

  ```java
  package com.segfault.entity.listener;
  
  
  import javax.persistence.PostPersist;
  import javax.persistence.PrePersist;
  
  public class CustomerListener {
  
      @PrePersist
      public void perPersist(Object source){
          System.out.println("@perPersist"+source);
      }
  
      @PostPersist
      public void postPersist(Object source){
          System.out.println("@perPersist"+source);
      }
  }
  
  ```

  使用@EntityListeners注入实体：

  ```java
  package com.segfault.entity;
  import com.segfault.entity.listener.CustomerListener;
  
  import javax.persistence.*;
  import java.util.Collection;
  
  /**
   * 客户类
   */
  @Entity
  @Access(value = AccessType.FIELD)
  @Table(name = "customers")
  @EntityListeners(value = {CustomerListener.class})
  public class Customer {
  
      @Id
      @GeneratedValue
      private Long id;
  
      @Column(length = 64)
      private String name;
  
      @OneToOne
      private CreditCard creditCard;
  
      @ManyToOne
      private Store store;
  
      @ManyToMany
      private Collection<Book> books;
  
  }
  
  ```



## Spring Data JPA

### 缓存（Caching）

JPA2.0中支持一级缓存和二级缓存，1.0只支持一级缓存。

- 一级缓存：持久化上下文（PersistenceContext）就是一级缓存，属于请求级别的缓存。
- 二级缓存：二级缓存是跨越上下文的，可被多个持久化上下文共享。

#### mybais中的缓存：

- 一级缓存：sqlSession级别，即本地缓存

  ```java
  SqlSession sqlSession = sqlSessionFactory.openSession();
  //清空本地缓存
  sqlSession.clearCache();
  ```

  可配置本地缓存的范围：`org.apache.ibatis.session.Configuration`

  ```java
    protected LocalCacheScope localCacheScope = LocalCacheScope.SESSION;
  ```

  - `LocalCacheScope.STATEMENT`：仅在语句执行时有效
  - `LocalCacheScope.SESSION`：整个session中有效，但是修改返回的数据（如list）会影响本地缓存的值（同个引用），所以不要对mybatis返回的对象进行修改

- 二级缓存：mapper级别，需配置**cacheEnabled=true**，配置后获取缓存顺序：**二级缓存→一级缓存→数据库**



#### Hibernate中的缓存

- 一级缓存：即Session的缓存
- 二级缓存：SessionFactory的缓存



### 查询API（Query API）

```java
public List findWithName(String name) {
 return em.createQuery(
"SELECT c FROM Customer c WHERE c.name LIKE :custName")
 .setParameter("custName", name)
 .setMaxResults(10)
 .getResultList();
}
```

### Criteria API

即面向对象的方式，具体参考JPA文档

### 元模型API（Metamodel API）

### Spring Data Repository

核心接口：

- org.springframework.data.repository.Repository

- org.springframework.data.repository.CrudRepository

- org.springframework.data.jpa.repository.JpaRepository

- @NoRepositoryBean：告诉sping不要为此repository创建代理bean，真正需要代理的是具体实现，如JpaRepository、CrudRepository等接口都被声明为@NoRepositoryBean，若不声明哪个，spring可能会为其创建代理类并报错，而具体实现类SimpleJpaRepository则不需要，因为它是具体实现。

  ```java
  @Repository
  @Transactional(readOnly = true)
  public class SimpleJpaRepository<T, ID extends Serializable>
  		implements JpaRepository<T, ID>, JpaSpecificationExecutor<T> {
  		...
  		}
  ```

  ```java
  @NoRepositoryBean
  public interface JpaRepository<T, ID extends Serializable>
  		extends PagingAndSortingRepository<T, ID>, QueryByExampleExecutor<T> {
  		...
  }
  ```

  ```java
  @NoRepositoryBean
  public interface CrudRepository<T, ID extends Serializable> extends Repository<T, ID> {
  ...
  }
  ```

运用示例：

```java
package com.segfault.repository;

import com.segfault.entity.Customer;
import org.hibernate.annotations.ManyToAny;
import org.springframework.beans.factory.annotation.Autowired;
import org.springframework.data.jpa.repository.support.SimpleJpaRepository;
import org.springframework.stereotype.Repository;
import org.springframework.transaction.annotation.Transactional;

import javax.persistence.EntityManager;

/**
 * 客户仓储
 */
@Repository
@Transactional(readOnly = false)
public class CustomerRepository extends SimpleJpaRepository<Customer,Long>{

    @Autowired
    public CustomerRepository(EntityManager em) {
        super(Customer.class, em);
    }
}

```

激活JPA Repository

- @EnableJpaRepositories

  ```java
  package com.segfault;
  
  
  import org.springframework.boot.SpringApplication;
  import org.springframework.boot.autoconfigure.SpringBootApplication;
  import org.springframework.data.jpa.repository.config.EnableJpaRepositories;
  import org.springframework.transaction.annotation.EnableTransactionManagement;
  
  @SpringBootApplication
  @EnableJpaRepositories(basePackages = {"com.segfault.entity"})
  @EnableTransactionManagement(proxyTargetClass = true)
  public class LessonEightStarter {
      public static void main(String[] args) {
          SpringApplication.run(LessonEightStarter.class,args);
      }
  }
  
  ```

  

问题

1、实现jdk动态代理和cglib动态代理

需自己实现ProxyFactoryBean

2、domain与entity区别

- domain：更关注逻辑状态描述
  - full domain：包含状态和操作
  - pool domain：基本的POJO、javaBean
- entity：javaBean+id，更关注于持久化

3、缓存









