1. 小马哥spring编程思想
2. 网络编程
   1. Java_TCPIP_Socket 网络编程.pd  
   2. Java 网络编程  
   3. NIO、mina、netty
3. Java rpc机制
   1. rmi
   2. hession
   3. thrift
   4. webservice



[MyBatis Generator](http://mybatis.org/generator/index.html)原理研究



# 学习资源

Python

链接: https://pan.baidu.com/s/1NhSJb97UaN1dn0TvN6SrNw 提取码: h1m2
链接: https://pan.baidu.com/s/1lRBvpRtx2LjkAkvd0sG4kQ 提取码: zh45
链接: https://pan.baidu.com/s/1L4pMwA8b2l6SQCNLuspB8w 提取码: 38dj
链接: https://pan.baidu.com/s/1-KWm88SJwEfmtFV2SDTn4w 提取码: ozi6
链接: https://pan.baidu.com/s/1yyoBL-Cf2ZtY4035cKHgzw 提取码: y628
链接: https://pan.baidu.com/s/1zfF5h3RUzNwoX1wMwloffQ 提取码: 8l99[/hide]



springCloud Alibaba

https://developer.aliyun.com/article/762296?utm_content=g_1000128689



groovy语言入门

https://www.jianshu.com/p/e8dec95c4326



spring cloud alibaba

https://start.aliyun.com/article/sca7lesson/rpc



# 定时任务

## quartz定时任务框架



## 分布式定时任务框架

- quartz集群解决方案：基于数据库悲观锁策略实现
  - 缺点：只解决了高可用的问题，并未解决任务分片的问题，存在单机性能瓶颈
- TBSchedule
- elastic-job：https://shardingsphere.apache.org/elasticjob/index_zh.html
- Saturn
- xxl-job





# HTTP调用方式

- JDK URLConnection
- Apache HTTP Client
- Netty异步HTTP client
- OkHttp
- Spring 
  - RestTemplate：同步调用
  - WebClient：支持同步、异步调用



# IO流

HttpConnection阻塞等待问题



# 流量统计

## 常见概念

- PV：page view，页面访问量。PV=QPS\*24\*60*60
- QPS：query per second，每秒查询（请求）数
- TPS：transation per second，每秒事务数，每个事务里可能包含多个请求



# JSON解析工具

- fastjson
- gson
- org.json
- net.sf.json
- json-simple
- jackson





数据库字符集、排序规则。





# 单元测试



## Junit4



## Junit5

参考文档：https://junit.org/junit5/docs/current/user-guide

### 编写测试

#### 常用注解

| Annotation               | Description                                                  |
| :----------------------- | :----------------------------------------------------------- |
| `@Test`                  | Denotes that a method is a test method. Unlike JUnit 4’s `@Test` annotation, this annotation does not declare any attributes, since test extensions in JUnit Jupiter operate based on their own dedicated annotations. Such methods are *inherited* unless they are *overridden*. |
| `@ParameterizedTest`     | Denotes that a method is a [parameterized test](https://junit.org/junit5/docs/current/user-guide/#writing-tests-parameterized-tests). Such methods are *inherited* unless they are *overridden*. |
| `@RepeatedTest`          | Denotes that a method is a test template for a [repeated test](https://junit.org/junit5/docs/current/user-guide/#writing-tests-repeated-tests). Such methods are *inherited* unless they are *overridden*. |
| `@TestFactory`           | Denotes that a method is a test factory for [dynamic tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests-dynamic-tests). Such methods are *inherited* unless they are *overridden*. |
| `@TestTemplate`          | Denotes that a method is a [template for test cases](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-templates) designed to be invoked multiple times depending on the number of invocation contexts returned by the registered [providers](https://junit.org/junit5/docs/current/user-guide/#extensions-test-templates). Such methods are *inherited* unless they are *overridden*. |
| `@TestMethodOrder`       | Used to configure the [test method execution order](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-execution-order) for the annotated test class; similar to JUnit 4’s `@FixMethodOrder`. Such annotations are *inherited*. |
| `@TestInstance`          | Used to configure the [test instance lifecycle](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle) for the annotated test class. Such annotations are *inherited*. |
| `@DisplayName`           | Declares a custom [display name](https://junit.org/junit5/docs/current/user-guide/#writing-tests-display-names) for the test class or test method. Such annotations are not *inherited*. |
| `@DisplayNameGeneration` | Declares a custom [display name generator](https://junit.org/junit5/docs/current/user-guide/#writing-tests-display-name-generator) for the test class. Such annotations are *inherited*. |
| `@BeforeEach`            | Denotes that the annotated method should be executed *before* **each** `@Test`, `@RepeatedTest`, `@ParameterizedTest`, or `@TestFactory` method in the current class; analogous to JUnit 4’s `@Before`. Such methods are *inherited* unless they are *overridden*. |
| `@AfterEach`             | Denotes that the annotated method should be executed *after* **each** `@Test`, `@RepeatedTest`, `@ParameterizedTest`, or `@TestFactory` method in the current class; analogous to JUnit 4’s `@After`. Such methods are *inherited* unless they are *overridden*. |
| `@BeforeAll`             | Denotes that the annotated method should be executed *before* **all** `@Test`, `@RepeatedTest`, `@ParameterizedTest`, and `@TestFactory` methods in the current class; analogous to JUnit 4’s `@BeforeClass`. Such methods are *inherited* (unless they are *hidden* or *overridden*) and must be `static` (unless the "per-class" [test instance lifecycle](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle) is used). |
| `@AfterAll`              | Denotes that the annotated method should be executed *after* **all** `@Test`, `@RepeatedTest`, `@ParameterizedTest`, and `@TestFactory` methods in the current class; analogous to JUnit 4’s `@AfterClass`. Such methods are *inherited* (unless they are *hidden* or *overridden*) and must be `static` (unless the "per-class" [test instance lifecycle](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle) is used). |
| `@Nested`                | Denotes that the annotated class is a non-static [nested test class](https://junit.org/junit5/docs/current/user-guide/#writing-tests-nested). `@BeforeAll` and `@AfterAll` methods cannot be used directly in a `@Nested` test class unless the "per-class" [test instance lifecycle](https://junit.org/junit5/docs/current/user-guide/#writing-tests-test-instance-lifecycle) is used. Such annotations are not *inherited*. |
| `@Tag`                   | Used to declare [tags for filtering tests](https://junit.org/junit5/docs/current/user-guide/#writing-tests-tagging-and-filtering), either at the class or method level; analogous to test groups in TestNG or Categories in JUnit 4. Such annotations are *inherited* at the class level but not at the method level. |
| `@Disabled`              | Used to [disable](https://junit.org/junit5/docs/current/user-guide/#writing-tests-disabling) a test class or test method; analogous to JUnit 4’s `@Ignore`. Such annotations are not *inherited*. |
| `@Timeout`               | Used to fail a test, test factory, test template, or lifecycle method if its execution exceeds a given duration. Such annotations are *inherited*. |
| `@ExtendWith`            | Used to [register extensions declaratively](https://junit.org/junit5/docs/current/user-guide/#extensions-registration-declarative). Such annotations are *inherited*. |
| `@RegisterExtension`     | Used to [register extensions programmatically](https://junit.org/junit5/docs/current/user-guide/#extensions-registration-programmatic) via fields. Such fields are *inherited* unless they are *shadowed*. |
| `@TempDir`               | Used to supply a [temporary directory](https://junit.org/junit5/docs/current/user-guide/#writing-tests-built-in-extensions-TempDirectory) via field injection or parameter injection in a lifecycle method or test method; located in the `org.junit.jupiter.api.io` package. |



#### 测试类和方法

- 测试类
- 测试方法
- 声明周期方法
  - @BeforeAll
  - @AfterAll
  - @BeforeEach
  - @AfterEach





#### Assertions

```java
package com.example;

import org.junit.jupiter.api.Assertions;
import org.junit.jupiter.api.Test;

import java.util.concurrent.CountDownLatch;

import static java.time.Duration.*;
import static org.junit.jupiter.api.Assertions.*;

/**
 * @Description: 断言demo
 * @Date : 14:58 2020/7/7
 */
public class AssertionsDemo {

    private final Calculator calculator = new Calculator();

    private final Person person = new Person("Jane", "Doe");

    @Test
    void standardAssertions() {
        Assertions.assertEquals(2, calculator.add(1, 1));
        Assertions.assertEquals(4, calculator.multiply(2, 2), "The optional failure message is now the last parameter");
        Assertions.assertTrue('a' < 'b', () -> "Assertion messages can be lazily evaluated -- "
                + "to avoid constructing complex messages unnecessarily.");
    }

    @Test
    void groupedAssertions() {
        //所有失败会统一报告
        Assertions.assertAll("person",
                () -> Assertions.assertEquals("Jane", person.getFirstName()),
                () -> Assertions.assertEquals("Doe", person.getLastName()));
    }

    @Test
    void dependentAssertions() {
        //发生错误时同一块代码块后续代码会跳过
        assertAll("properties",
                () -> {
                    String firstName = person.getFirstName();
                    assertNotNull(firstName);
                    // Executed only if the previous assertion is valid.
                    System.out.println(firstName);
                    assertAll("first name",
                            () -> assertTrue(firstName.startsWith("J")),
                            () -> assertTrue(firstName.endsWith("e"))
                    );
                },
                () -> {
                    System.out.println("second...");
                    // Grouped assertion, so processed independently
                    // of results of first name assertions.
                    String lastName = person.getLastName();
                    assertNotNull(lastName);

                    // Executed only if the previous assertion is valid.
                    assertAll("last name",
                            () -> assertTrue(lastName.startsWith("D")),
                            () -> assertTrue(lastName.endsWith("e"))
                    );
                });
    }

    /**
     * 异常
     */
    @Test
    void exceptionTesting(){
        Exception exception = assertThrows(ArithmeticException.class, () ->
                calculator.divide(1, 0));
        assertEquals("/ by zero", exception.getMessage());
    }

    /**
     * 执行超时
     */
    @Test
    void timeoutNotExceeded() {
        // The following assertion succeeds.
        assertTimeout(ofSeconds(2), () -> {
            // Perform task that takes less than 2 minutes.
            Thread.sleep(1000);
        });
    }

    /**
     * 有返回值的执行超时
     */
    @Test
    void timeoutNotExceededWithResult() {
        // The following assertion succeeds, and returns the supplied object.
        String actualResult = assertTimeout(ofSeconds(2), () -> {
            Thread.sleep(1000);
            return "a result";
        });
        assertEquals("a result", actualResult);
    }

    /**
     * 有返回值的执行超时（执行方法超时）
     */
    @Test
    void timeoutNotExceededWithMethod() {
        // The following assertion invokes a method reference and returns an object.
        String actualGreeting = assertTimeout(ofMinutes(2), AssertionsDemo::greeting);
        assertEquals("Hello, World!", actualGreeting);
    }


    @Test
    void timeoutExceeded() {
        // The following assertion fails with an error message similar to:
        // execution exceeded timeout of 10 ms by 91 ms
        assertTimeout(ofMillis(10), () -> {
            // Simulate task that takes more than 10 ms.
            //Thread.sleep(100);
        });
    }


    @Test
    void timeoutExceededWithPreemptiveTermination() {
        // The following assertion fails with an error message similar to:
        // execution timed out after 10 ms
        assertTimeoutPreemptively(ofMillis(10), () -> {
            // Simulate task that takes more than 10 ms.
            new CountDownLatch(1).await();
        });
    }


    private static String greeting() {
        return "Hello, World!";
    }

}

```



##### 第三方Assertion包

- AssertJ

- Hamcrest

  ```java
  package com.example;
  
  import org.hamcrest.MatcherAssert;
  import org.junit.jupiter.api.Test;
  
  import static org.hamcrest.CoreMatchers.is;
  import static org.hamcrest.CoreMatchers.equalTo;
  
  
  /**
   * @Description: Hamcrest框架示例
   * @Date : 15:57 2020/7/7
   */
  public class HamcrestAssertionsDemo {
  
      private Calculator calculator = new Calculator();
  
      @Test
      void assertWithHamcrestMatcher() {
          MatcherAssert.assertThat(calculator.subtract(4, 1), is(equalTo(3)));
      }
  
  }
  
  ```

- Truth



#### Assumptions假设

```java
package com.example;

import org.junit.jupiter.api.Test;

import static org.junit.jupiter.api.Assertions.assertEquals;
import static org.junit.jupiter.api.Assumptions.assumeTrue;
import static org.junit.jupiter.api.Assumptions.assumingThat;

/**
 * @Description: 假设
 * @Date : 16:13 2020/7/7
 */
public class AssumptionsDemo {

    private final Calculator calculator = new Calculator();

    @Test
    void testOnlyOnCiServer() {
        assumeTrue("CI".equals(System.getenv("ENV")));
        // remainder of test
    }

    @Test
    void testOnlyOnDeveloperWorkstation() {
        assumeTrue("DEV".equals(System.getenv("ENV")),
                () -> "Aborting test: not on developer workstation");
        // remainder of test
    }

    @Test
    void testInAllEnvironments() {
        assumingThat("CI".equals(System.getenv("ENV")),
                () -> {
                    // perform these assertions only on the CI server
                    assertEquals(2, calculator.divide(4, 2));
                });

        // perform these assertions in all environments
        assertEquals(42, calculator.multiply(6, 7));
    }

}
```



#### disable

```java
package com.example;

import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

/**
 * @Description: disable class 一般需在@Disabled中注明为何原因
 * @Date : 16:17 2020/7/7
 */
@Disabled("Disabled until bug #99 has been fixed")
public class DisabledClassDemo {

    @Test
    void testWillBeSkipped() {
    }

}

```

```java
package com.example;

import org.junit.jupiter.api.Disabled;
import org.junit.jupiter.api.Test;

/**
 * @Description: disable method
 * @Date : 16:19 2020/7/7
 */
public class DisabledTestsDemo {

    @Disabled("Disabled until bug #42 has been resolved")
    @Test
    void testWillBeSkipped() {
    }

    @Test
    void testWillBeExecuted() {
    }

}

```



#### 有条件执行

1. 操作系统条件

   - [@EnabledOnOs](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/condition/EnabledOnOs.html)
   - [@DisabledOnOs](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/condition/DisabledOnOs.html)

   ```java
   @Test
   @EnabledOnOs(MAC)
   void onlyOnMacOs() {
       // ...
   }
   
   @TestOnMac
   void testOnMac() {
       // ...
   }
   
   @Test
   @EnabledOnOs({ LINUX, MAC })
   void onLinuxOrMac() {
       // ...
   }
   
   @Test
   @DisabledOnOs(WINDOWS)
   void notOnWindows() {
       // ...
   }
   
   @Target(ElementType.METHOD)
   @Retention(RetentionPolicy.RUNTIME)
   @Test
   @EnabledOnOs(MAC)
   @interface TestOnMac {
   }
   ```

2. java运行环境条件

   - [@EnabledOnJre](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/condition/EnabledOnJre.html)
   - [@DisabledForJreRange](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/condition/DisabledForJreRange.html)

   ```java
   @Test
   @EnabledOnJre(JAVA_8)
   void onlyOnJava8() {
       // ...
   }
   
   @Test
   @EnabledOnJre({ JAVA_9, JAVA_10 })
   void onJava9Or10() {
       // ...
   }
   
   @Test
   @EnabledForJreRange(min = JAVA_9, max = JAVA_11)
   void fromJava9to11() {
       // ...
   }
   
   @Test
   @EnabledForJreRange(min = JAVA_9)
   void fromJava9toCurrentJavaFeatureNumber() {
       // ...
   }
   
   @Test
   @EnabledForJreRange(max = JAVA_11)
   void fromJava8To11() {
       // ...
   }
   
   @Test
   @DisabledOnJre(JAVA_9)
   void notOnJava9() {
       // ...
   }
   
   @Test
   @DisabledForJreRange(min = JAVA_9, max = JAVA_11)
   void notFromJava9to11() {
       // ...
   }
   
   @Test
   @DisabledForJreRange(min = JAVA_9)
   void notFromJava9toCurrentJavaFeatureNumber() {
       // ...
   }
   
   @Test
   @DisabledForJreRange(max = JAVA_11)
   void notFromJava8to11() {
       // ...
   }
   ```

3. 系统参数条件

   - [@EnabledIfSystemProperty](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/condition/EnabledIfSystemProperty.html)
   - [@DisabledIfSystemProperty](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/condition/DisabledIfSystemProperty.html)

   ```java
   @Test
   @EnabledIfSystemProperty(named = "os.arch", matches = ".*64.*")
   void onlyOn64BitArchitectures() {
       // ...
   }
   
   @Test
   @DisabledIfSystemProperty(named = "ci-server", matches = "true")
   void notOnCiServer() {
       // ...
   }
   ```

4. 环境变量条件

   - [@EnabledIfEnvironmentVariable](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/condition/EnabledIfEnvironmentVariable.html)
   - [@DisabledIfEnvironmentVariable](https://junit.org/junit5/docs/current/api/org.junit.jupiter.api/org/junit/jupiter/api/condition/DisabledIfEnvironmentVariable.html)

   ```java
   @Test
   @EnabledIfEnvironmentVariable(named = "ENV", matches = "staging-server")
   void onlyOnStagingServer() {
       // ...
   }
   
   @Test
   @DisabledIfEnvironmentVariable(named = "ENV", matches = ".*development.*")
   void notOnDeveloperWorkstation() {
       // ...
   }
   ```



#### 标签和过滤

@Tags用于打标签，常用于过滤

```java
import org.junit.jupiter.api.Tag;
import org.junit.jupiter.api.Test;

@Tag("fast")
@Tag("model")
class TaggingDemo {

    @Test
    @Tag("taxes")
    void testingTaxCalculation() {
    }

}
```



#### 按顺序执行测试

```java
package com.example;

import org.junit.jupiter.api.MethodOrderer;
import org.junit.jupiter.api.Order;
import org.junit.jupiter.api.Test;
import org.junit.jupiter.api.TestMethodOrder;

/**
 * @Description: 按顺序执行测试
 * @Date : 9:13 2020/7/8
 */
@TestMethodOrder(MethodOrderer.OrderAnnotation.class)
public class OrderTestDemo {

    @Test
    @Order(1)
    void test1(){
        System.out.println("test1...");
    }

    @Test
    @Order(3)
    void test2(){
        System.out.println("test2...");
    }

    @Test
    @Order(2)
    void test3(){
        System.out.println("test3...");
    }

}
```

默认有三种类型的排序

- org.junit.jupiter.api.MethodOrderer.Random：随机
- org.junit.jupiter.api.MethodOrderer.OrderAnnotation：根据@Order排序
- org.junit.jupiter.api.MethodOrderer.Alphanumeric：根据名字和正式参数列表排序



#### 测试实例声明周期

- org.junit.jupiter.api.TestInstance.Lifecycle#PER_METHOD：创建多个测试实例

  ```java
  package com.example;
  
  import org.junit.jupiter.api.Order;
  import org.junit.jupiter.api.Test;
  import org.junit.jupiter.api.TestInstance;
  
  /**
   * @Description: 生命周期
   * @Date : 9:43 2020/7/8
   */
  @TestInstance(TestInstance.Lifecycle.PER_METHOD)
  public class LifeCycleDemo {
  
      @Test
      void test1(){
          System.out.println("current instance:" + this);
      }
  
      @Test
      void test2(){
          System.out.println("current instance:" + this);
      }
  
      @Test
      void test3(){
          System.out.println("current instance:" + this);
      }
  
  }
  ```

  ```proper
  current instance:com.example.LifeCycleDemo@d35dea7
  
  
  current instance:com.example.LifeCycleDemo@66d18979
  
  
  current instance:com.example.LifeCycleDemo@bccb269
  ```

- org.junit.jupiter.api.TestInstance.Lifecycle#PER_CLASS：只创建一个测试实例

  ```JAVA
  package com.example;
  
  import org.junit.jupiter.api.Order;
  import org.junit.jupiter.api.Test;
  import org.junit.jupiter.api.TestInstance;
  
  /**
   * @Description: 生命周期
   * @Date : 9:43 2020/7/8
   */
  @TestInstance(TestInstance.Lifecycle.PER_CLASS)
  public class LifeCycleDemo {
  
      @Test
      void test1(){
          System.out.println("current instance:" + this);
      }
  
      @Test
      void test2(){
          System.out.println("current instance:" + this);
      }
  
      @Test
      void test3(){
          System.out.println("current instance:" + this);
      }
  
  }
  
  ```

  ```properties
  current instance:com.example.LifeCycleDemo@24313fcc
  
  
  current instance:com.example.LifeCycleDemo@24313fcc
  
  
  current instance:com.example.LifeCycleDemo@24313fcc
  ```

  

#### nested

常用于标识测试类的层次性

```java
package com.example;

import org.junit.jupiter.api.*;

import java.util.EmptyStackException;
import java.util.Stack;

import static org.junit.jupiter.api.Assertions.*;

/**
 * @Description: nested test
 * @Date : 10:38 2020/7/8
 */
@DisplayName("A stack")
public class TestingAStackDemo {

    Stack<Object> stack;

    @Test
    @DisplayName("is instantiated with new Stack()")
    void isInstantiatedWithNew() {
        new Stack<>();
    }

    @Nested
    @DisplayName("when new")
    class WhenNew {

        /*@BeforeAll
        void beforAll(){
            System.out.println("before all...");
        }*/

        /**
         * 每个Test方法执行前都会执行
         */
        @BeforeEach
        @DisplayName("createNewStack")
        void createNewStack() {
            assertTrue(true);
            System.out.println("new stack");
            stack = new Stack<>();
        }

        @Test
        @DisplayName("is empty")
        void isEmpty() {
            assertTrue(stack.isEmpty());
        }

        @Test
        @DisplayName("throws EmptyStackException when popped")
        void throwsExceptionWhenPopped() {
            assertThrows(EmptyStackException.class, stack::pop);
        }

        @Test
        @DisplayName("throws EmptyStackException when peeked")
        void throwsExceptionWhenPeeked() {
            assertThrows(EmptyStackException.class, stack::peek);
        }

        @Nested
        @DisplayName("after pushing an element")
        class AfterPushing {

            String anElement = "an element";

            @BeforeEach
            void pushAnElement() {
                stack.push(anElement);
            }

            @Test
            @DisplayName("it is no longer empty")
            void isNotEmpty() {
                assertFalse(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when popped and is empty")
            void returnElementWhenPopped() {
                assertEquals(anElement, stack.pop());
                assertTrue(stack.isEmpty());
            }

            @Test
            @DisplayName("returns the element when peeked but remains not empty")
            void returnElementWhenPeeked() {
                assertEquals(anElement, stack.peek());
                assertFalse(stack.isEmpty());
            }
        }
    }

}

```

> 注意beforeAll和afterAll不能作用在内部类中，因为java中内部类不允许调用静态方法



#### 构造器和方法的依赖注入



### 运行测试

idea中使用：

```xml
<!-- Only needed to run tests in a version of IntelliJ IDEA that bundles older versions -->
<dependency>
    <groupId>org.junit.platform</groupId>
    <artifactId>junit-platform-launcher</artifactId>
    <version>1.6.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.jupiter</groupId>
    <artifactId>junit-jupiter-engine</artifactId>
    <version>5.6.2</version>
    <scope>test</scope>
</dependency>
<dependency>
    <groupId>org.junit.vintage</groupId>
    <artifactId>junit-vintage-engine</artifactId>
    <version>5.6.2</version>
    <scope>test</scope>
</dependency>
```



maven中需引用：

```xml
<build>
    <plugins>
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.2</version>
        </plugin>
        <plugin>
            <artifactId>maven-failsafe-plugin</artifactId>
            <version>2.22.2</version>
        </plugin>
    </plugins>
</build>
<!-- ... -->
<dependencies>
    <!-- ... -->
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-api</artifactId>
        <version>5.6.2</version>
        <scope>test</scope>
    </dependency>
    <dependency>
        <groupId>org.junit.jupiter</groupId>
        <artifactId>junit-jupiter-engine</artifactId>
        <version>5.6.2</version>
        <scope>test</scope>
    </dependency>
    <!-- ... -->
</dependencies>
<!-- ... -->
```



#### maven surefire相关支持

参考https://maven.apache.org/surefire/maven-surefire-plugin/examples/inclusion-exclusion.html

##### inclusions

默认会执行测试的类：

- `"**/Test*.java"` - includes all of its subdirectories and all Java filenames that start with "Test".
- `"**/*Test.java"` - includes all of its subdirectories and all Java filenames that end with "Test".
- `"**/*Tests.java"` - includes all of its subdirectories and all Java filenames that end with "Tests".
- `"**/*TestCase.java"` - includes all of its subdirectories and all Java filenames that end with "TestCase".

若命名不符合上述规范，也可以特殊设置：

```xml
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.0.0-M5</version>
        <configuration>
          <includes>
            <include>Sample.java</include>
          </includes>
        </configuration>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
```



##### Exclusions

配置排除类：

```xml
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.0.0-M5</version>
        <configuration>
          <excludes>
            <exclude>**/TestCircle.java</exclude>
            <exclude>**/TestSquare.java</exclude>
          </excludes>
        </configuration>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
```



##### 正则表达式支持

```xml
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.0.0-M5</version>
        <configuration>
          <includes>
            <include>%regex[.*(Cat|Dog).*Test.*]</include>
          </includes>
        </configuration>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
```



##### 一次使用多个表达式

As of Surefire Plugin 2.19, a complex syntax is supported in one parameter (JUnit 4, JUnit 4.7+, TestNG):

```xml
  [...]
          <include>%regex[.*(Cat|Dog).*], !%regex[pkg.*Slow.*.class], pkg/**/*Fast*.java, Basic????, !Unstable*</include>
  [...]
          <exclude>%regex[pkg.*Slow.*.class], Unstable*</exclude>
  [...]
```

This syntax can be used in parameters: `test`, `includes`, `excludes`, `includesFile`, `excludesFile`. Exclamation mark (!) excludes tests. The syntax in parameter `excludes` and `excludesFile` should not use (!). The character (?) within non-regex pattern replaces one character in file name or path. The file extensions are not mandatory in non-regex patterns, and packages with slash can be used. The regex validates fully qualified class file. The regex supports '.class' file extension only. Note the regex comments, marked by (#) character, are unsupported.



##### 全限定类名匹配

As of Surefire Plugin 2.19.1, the syntax with fully qualified class names or packages can be used, e.g.:

```xml
  [...]
          <include>my.package.*, another.package.*</include>
  [...]
          <exclude>my.package.???ExcludedTest, another.package.*ExcludedTest</exclude>
  [...]
```

The character (?) replaces single character and (*) represents zero or more characters. Multiple formats can be additionally combined. This syntax can be used in parameters: `test`, `includes`, `excludes`, `includesFile`, `excludesFile`.



##### 从依赖中选择测试

By default, Surefire will only scan for test classes to execute in the configured `testSourceDirectory`. To have the plugin scan dependencies to find test classes to execute, use the `dependenciesToScan` configuration. Dependencies can be specified using the `groupId[:artifactId[:type[:classifier][:version]]]` format, and must already be `dependency` elements in scope.

*Note:* Support for version, type and classifier was introduced in version `3.0.0-M4`. When using earlier versions, Surefire will fail with an `IllegalArgumentException` if more than groupId and artifactId are specified.

```xml
<project>
  [...]
  <build>
    <plugins>
      <plugin>
        <groupId>org.apache.maven.plugins</groupId>
        <artifactId>maven-surefire-plugin</artifactId>
        <version>3.0.0-M5</version>
        <configuration>
          <dependenciesToScan>
            <dependency>org.acme:project-a</dependency>
            <dependency>org.acme:project-b:test-jar</dependency>
            <dependency>org.acme:project-c:*:tests-jdk15</dependency>
          </dependenciesToScan>
        </configuration>
      </plugin>
    </plugins>
  </build>
  [...]
</project>
```

The configuration above will search for test classes in:

- All dependencies with the groupId `org.acme` and artifactId `project-a`
- All dependencies with the groupId `org.acme` and artifactId `project-b` and type `test-jar`
- All dependencies with the groupId `org.acme` and artifactId `project-c` and classifier `tests-jdk15`



##### 通过tags过滤

- include：使用group
- exclude：使用excludedGroups：

```xml
<!-- ... -->
<build>
    <plugins>
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.2</version>
            <configuration>
                <groups>acceptance | !feature-a</groups>
                <excludedGroups>integration, regression</excludedGroups>
            </configuration>
        </plugin>
    </plugins>
</build>
<!-- ... -->
```



##### 配置参数

可通过设置Junit平台的参数来影响test流程的发现和执行过程

- 在pom文件中配置（如下）
- 在junit-platform.properties中设置

```xml
<!-- ... -->
<build>
    <plugins>
        <plugin>
            <artifactId>maven-surefire-plugin</artifactId>
            <version>2.22.2</version>
            <configuration>
                <properties>
                    <configurationParameters>
                        junit.jupiter.conditions.deactivate = *
                        junit.jupiter.extensions.autodetection.enabled = true
                        junit.jupiter.testinstance.lifecycle.default = per_class
                    </configurationParameters>
                </properties>
            </configuration>
        </plugin>
    </plugins>
</build>
<!-- ... -->
```





## testNG



# 文件系统

- fastDFS





# FRP内网穿透

nginx frps frpc nginx



# JSR #380 Bean Validation 2.0

> Bean Validation 2.0 是JSR第380号标准。该标准连接如下：https://www.jcp.org/en/egc/view?id=380
> Bean Validation的主页：[http://beanvalidation.org](http://beanvalidation.org/)
> Bean Validation的参考实现：https://github.com/hibernate/hibernate-validator







# 阿里编码规范

## 正确创建线程池

手动创建线程池效果会更好哦。 
 Inspection info: 
线程池不允许使用Executors去创建，而是通过ThreadPoolExecutor的方式，这样的处理方式让写的同学更加明确线程池的运行规则，规避资源耗尽的风险。 说明：Executors返回的线程池对象的弊端如下：
1）FixedThreadPool和SingleThreadPool:
  允许的请求队列长度为Integer.MAX_VALUE，可能会堆积大量的请求，从而导致OOM。
2）CachedThreadPool:
  允许的创建线程数量为Integer.MAX_VALUE，可能会创建大量的线程，从而导致OOM。
            
Positive example 1：
    //org.apache.commons.lang3.concurrent.BasicThreadFactory
    ScheduledExecutorService executorService = new ScheduledThreadPoolExecutor(1,
        new BasicThreadFactory.Builder().namingPattern("example-schedule-pool-%d").daemon(true).build());
       
        
            
Positive example 2：
    ThreadFactory namedThreadFactory = new ThreadFactoryBuilder()
        .setNameFormat("demo-pool-%d").build();

    //Common Thread Pool
    ExecutorService pool = new ThreadPoolExecutor(5, 200,
        0L, TimeUnit.MILLISECONDS,
        new LinkedBlockingQueue<Runnable>(1024), namedThreadFactory, new ThreadPoolExecutor.AbortPolicy());
    
    pool.execute(()-> System.out.println(Thread.currentThread().getName()));
    pool.shutdown();//gracefully shutdown

Positive example 3：
​    <bean id="userThreadPool"
​        class="org.springframework.scheduling.concurrent.ThreadPoolTaskExecutor">
​        <property name="corePoolSize" value="10" />
​        <property name="maxPoolSize" value="100" />
​        <property name="queueCapacity" value="2000" />

    <property name="threadFactory" value= threadFactory />
        <property name="rejectedExecutionHandler">
            <ref local="rejectedExecutionHandler" />
        </property>
    </bean>
    //in code
    userThreadPool.execute(thread);





# JWT（Json Web Token）



## 使用场景

- 授权
- 信息交换



## 数据接口

JSON web token通过“.”分割为三部分：

- Header：通常只包含两部分

  - 类型：JWT
  - 签名算法：HMAC、SHA256、RSA等

  ```json
  {
    "alg": "HS256",
    "typ": "JWT"
  }
  ```

  > 接着使用**Base64Url** 编码生成JWT的第一部分

- Payload：包含多个claims，claims是一些关于实体的描述（如：user）和一些额外信息，有三种claims

  - Registered claims：一些推荐使用的预定义的claims，如iss(issuer)、exp(expiration time)、sub(subject)、aud(audience)等等
  - Public claims：
  - private claims：用户自定义的，用来保存可分享信息的claims

  ```json
  {
    "sub": "1234567890",
    "name": "John Doe",
    "admin": true
  }
  ```

  

- Signatute：使用header中的算法对编码的header、payload和秘钥进行签名，如：

  ```json
  HMACSHA256(
    base64UrlEncode(header) + "." +
    base64UrlEncode(payload),
    secret)
  ```

  



# Spring EL表达式

类似统一EL表达式，但在其基础上增加了额外的特性

Spring EL表达式支持功能：

- Literal expressions
- Boolean and relational operators
- Regular expressions
- Class expressions
- Accessing properties, arrays, lists, and maps
- Method invocation
- Relational operators
- Assignment
- Calling constructors
- Bean references
- Array construction
- Inline lists
- Inline maps
- Ternary operator
- Variables
- User-defined functions
- Collection projection
- Collection selection
- Templated expressions



## 编译

可修改编译模式用于提升性能：`org.springframework.expression.spel.SpelCompilerMode`

- `OFF`

- `IMMEDIATE`

- `MIXED`

  



# Spring Cache

核心接口：

org.springframework.cache.Cache

org.springframework.cache.CacheManager



## 注解

声明性缓存注解

- `@Cacheable`：触发缓存入口
- `@CacheEvict`：触发缓存删除
- `@CachePut`：在不干扰方法执行情况下更新缓存
- `@Caching`：
- `@CacheConfig`：缓存配置，可指定cacheName等



### @Cacheable

Key生成器

org.springframework.cache.interceptor.KeyGenerator

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

package org.springframework.cache.interceptor;

import java.lang.reflect.Method;

/**
 * Cache key generator. Used for creating a key based on the given method
 * (used as context) and its parameters.
 *
 * @author Costin Leau
 * @author Chris Beams
 * @author Phillip Webb
 * @since 3.1
 */
@FunctionalInterface
public interface KeyGenerator {

	/**
	 * Generate a key for the given method and its parameters.
	 * @param target the target instance
	 * @param method the method being called
	 * @param params the method parameters (with any var-args expanded)
	 * @return a generated key
	 */
	Object generate(Object target, Method method, Object... params);

}

```

默认实现：org.springframework.cache.interceptor.SimpleKeyGenerator

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

package org.springframework.cache.interceptor;

import java.lang.reflect.Method;

/**
 * Simple key generator. Returns the parameter itself if a single non-null
 * value is given, otherwise returns a {@link SimpleKey} of the parameters.
 *
 * <p>No collisions will occur with the keys generated by this class.
 * The returned {@link SimpleKey} object can be safely used with a
 * {@link org.springframework.cache.concurrent.ConcurrentMapCache}, however,
 * might not be suitable for all {@link org.springframework.cache.Cache}
 * implementations.
 *
 * @author Phillip Webb
 * @author Juergen Hoeller
 * @since 4.0
 * @see SimpleKey
 * @see org.springframework.cache.annotation.CachingConfigurer
 */
public class SimpleKeyGenerator implements KeyGenerator {

	@Override
	public Object generate(Object target, Method method, Object... params) {
		return generateKey(params);
	}

	/**
	 * Generate a key based on the specified parameters.
	 */
	public static Object generateKey(Object... params) {
		if (params.length == 0) {
			return SimpleKey.EMPTY;
		}
		if (params.length == 1) {
			Object param = params[0];
			if (param != null && !param.getClass().isArray()) {
				return param;
			}
		}
		return new SimpleKey(params);
	}

}

```





API网关：

鉴权访问者模式、门面模式





# mongodb



## 启动

../bin/mongod

mongod常用启动参数（windows下）

查看更多参数：mangod --help

| 参数                 | 描述                                                         |
| :------------------- | :----------------------------------------------------------- |
| --bind_ip            | 绑定服务IP，若绑定127.0.0.1，则只能本机访问，不指定默认本地所有IP |
| --logpath            | 定义MongoDB日志文件，注意是指定文件不是目录                  |
| --logappend          | 使用追加的方式写日志                                         |
| --dbpath             | 指定数据库路径                                               |
| --port               | 指定服务端口号，默认端口27017                                |
| --serviceName        | 指定服务名称                                                 |
| --serviceDisplayName | 指定服务名称，有多个mongodb服务时执行。                      |
| --install            | 指定作为一个Windows服务安装。                                |



## 连接

- Shell连接
- PHP连接

../bin/mongo

### 参数选项

- 标准格式：mongodb://[username:password@]host1[:port1][,host2[:port2],...[,hostN[:portN]]][/[database][?options]]

- 常用参数选项

  | 选项                | 描述                                                         |
  | :------------------ | :----------------------------------------------------------- |
  | replicaSet=name     | 验证replica set的名称。 Impliesconnect=replicaSet.           |
  | slaveOk=true\|false | true:在connect=direct模式下，驱动会连接第一台机器，即使这台服务器不是主。在connect=replicaSet模式下，驱动会发送所有的写请求到主并且把读取操作分布在其他从服务器。false: 在 connect=direct模式下，驱动会自动找寻主服务器. 在connect=replicaSet 模式下，驱动仅仅连接主服务器，并且所有的读写命令都连接到主服务器。 |
  | safe=true\|false    | true: 在执行更新操作之后，驱动都会发送getLastError命令来确保更新成功。(还要参考 wtimeoutMS).false: 在每次更新之后，驱动不会发送getLastError来确保更新成功。 |
  | w=n                 | 驱动添加 { w : n } 到getLastError命令. 应用于safe=true。     |
  | wtimeoutMS=ms       | 驱动添加 { wtimeout : ms } 到 getlasterror 命令. 应用于 safe=true. |
  | fsync=true\|false   | true: 驱动添加 { fsync : true } 到 getlasterror 命令.应用于 safe=true.false: 驱动不会添加到getLastError命令中。 |
  | journal=true\|false | 如果设置为 true, 同步到 journal (在提交到数据库前写入到实体中). 应用于 safe=true |
  | connectTimeoutMS=ms | 可以打开连接的时间。                                         |
  | socketTimeoutMS=ms  | 发送和接受sockets的时间。                                    |





## 语句操作

### 数据增删

#### insert()方法

语法：

```sql
 db.COLLECTION_NAME.insert(document)  
```



#### update()方法

语法格式：

```sql
db.collection.update(    
	<query>, 
	<update>, 
	{       
		upsert: <boolean>,   
		multi: <boolean>,  
		writeConcern: <document>
	}
)
```

参数说明：

- **query** : update的查询条件，类似sql update查询内where后面的。
- **update** : update的对象和一些更新的操作符（如$,$inc...）等，也可以理解为sql update查询内set后面的
- **upsert** : 可选，这个参数的意思是，如果不存在update的记录，是否插入objNew,true为插入，默认是false，不插入。
- **multi** : 可选，mongodb 默认是false,只更新找到的第一条记录，如果这个参数为true,就把按条件查出来多条记录全部更新。
- **writeConcern** :可选，抛出异常的级别。

例：

```sql
>db.col.insert({  
	title: 'Mongodb 教程',  
	description: 'MongoDB 是一个 Nosql 数据库',   
	by: 'Mongodb中文网',   
	url: 'http://www.mongodb.org.cn',   
	tags: ['mongodb', 'database', 'NoSQL'],  
	likes: 100  
})

>db.col.update({'title':'MongoDB 教程'},{$set:{'title':'MongoDB'}})
WriteResult({ "nMatched" : 1, "nUpserted" : 0, "nModified" : 1 })   # 输出信息  

> db.col.find().pretty()  
{         
	"_id" : ObjectId("56064f89ade2f21f36b03136"),  
	"title" : "MongoDB",   
	"description" : "MongoDB 是一个 Nosql 数据库",  
	"by" : "Mongodb中文网",      
	"url" : "http://www.mongodb.org.cn", 
	"tags" : [   
		"mongodb",   
		"database",   
		"NoSQL"      
	],      
	"likes" : 100  
}  

>  
```



#### save()方法

save() 方法通过传入的文档来替换已有文档。语法格式如下：

```sql
db.collection.save(    
	<document>,     
	{      
		writeConcern: <document> 
	}  
)  
```

参数说明：

- **document** : 文档数据。
- **writeConcern** :可选，抛出异常的级别。



#### remove()方法

2.6版本之前：

```sql
db.collection.remove( 
	<query>,     
	<justOne> 
)  
```

2.6版本之后：

```sql
db.collection.remove(     
	<query>,     
	{       
		justOne: <boolean>,
		writeConcern: <document> 
	} 
)
```



### 数据查询

#### find()方法

语法：

```sql
db.collection.find(query, projection)
```

- **query** ：可选，使用查询操作符指定查询条件
- **projection** ：可选，使用投影操作符指定返回的键。查询时返回文档中所有键值， 只需省略该参数即可（默认省略）。

```sql
>db.COLLECTION_NAME.find()
>db.COLLECTION_NAME.find().pretty()
/**只返回一个文档*/
>db.COLLECTION_NAME.findOne()
```



#### MongoDB 与 RDBMS Where 语句比较

| 操作       | 格式                     | 范例                                        | RDBMS中的类似语句       |
| :--------- | :----------------------- | :------------------------------------------ | :---------------------- |
| 等于       | `{<key>:<value>`}        | `db.col.find({"by":"菜鸟教程"}).pretty()`   | `where by = '菜鸟教程'` |
| 小于       | `{<key>:{$lt:<value>}}`  | `db.col.find({"likes":{$lt:50}}).pretty()`  | `where likes < 50`      |
| 小于或等于 | `{<key>:{$lte:<value>}}` | `db.col.find({"likes":{$lte:50}}).pretty()` | `where likes <= 50`     |
| 大于       | `{<key>:{$gt:<value>}}`  | `db.col.find({"likes":{$gt:50}}).pretty()`  | `where likes > 50`      |
| 大于或等于 | `{<key>:{$gte:<value>}}` | `db.col.find({"likes":{$gte:50}}).pretty()` | `where likes >= 50`     |
| 不等于     | `{<key>:{$ne:<value>}}`  | `db.col.find({"likes":{$ne:50}}).pretty()`  | `where likes != 50`     |



#### AND条件

```sql
 >db.col.find({key1:value1, key2:value2}).pretty()  
```



#### OR条件

```sql
>db.col.find(     {        $or: [  	     {key1: value1}, {key2:value2}        ]     }  ).pretty()  
```



### mongodb条件符

- (>) 大于 - $gt
- (<) 小于 - $lt
- (>=) 大于等于 - $gte
- (<= ) 小于等于 - $lte



### $type操作符

$type操作符是基于BSON类型来检索集合中匹配的类型，并返回结果。mongodb中可使用的类型：

| **类型**                | **数字** | **备注**         |
| :---------------------- | :------- | :--------------- |
| Double                  | 1        |                  |
| String                  | 2        |                  |
| Object                  | 3        |                  |
| Array                   | 4        |                  |
| Binary data             | 5        |                  |
| Undefined               | 6        | 已废弃。         |
| Object id               | 7        |                  |
| Boolean                 | 8        |                  |
| Date                    | 9        |                  |
| Null                    | 10       |                  |
| Regular Expression      | 11       |                  |
| JavaScript              | 13       |                  |
| Symbol                  | 14       |                  |
| JavaScript (with scope) | 15       |                  |
| 32-bit integer          | 16       |                  |
| Timestamp               | 17       |                  |
| 64-bit integer          | 18       |                  |
| Min key                 | 255      | Query with `-1`. |
| Max key                 | 127      |                  |

如获取“col”集合中“title”类型为String的数据

```sql
db.col.find({"title" : {$type : 2}})
```



### limit()方法

传入一个数字值，用于指定返回的数据的条数

```sql
  >db.COLLECTION_NAME.find().limit(NUMBER)  
```



### skip()方法

传入一个数字值，用于指定跳过的数据条数

```sql
  >db.COLLECTION_NAME.find().limit(NUMBER).skip(NUMBER)  
```

> 结合limit()方法实现分页的功能



### sort()方法

指定排序的字段，1为升序排列，-1为降序排列

```sql
  >db.COLLECTION_NAME.find().sort({KEY:1})  
```

实例：

如有数据：

```sql
  { "_id" : ObjectId("56066542ade2f21f36b0313a"), "title" : "PHP 教程", "description" : "PHP 是一种创建动态交互性站点的强有力的服务器端脚本语言。", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "php" ], "likes" : 200 }  { "_id" : ObjectId("56066549ade2f21f36b0313b"), "title" : "Java 教程", "description" : "Java 是由Sun Microsystems公司于1995年5月推出的高级程序设计语言。", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "java" ], "likes" : 150 }  { "_id" : ObjectId("5606654fade2f21f36b0313c"), "title" : "MongoDB 教程", "description" : "MongoDB 是一个 Nosql 数据库", "by" : "菜鸟教程", "url" : "http://www.runoob.com", "tags" : [ "mongodb" ], "likes" : 100 }  
```

按“likes”降序排列

```sql

>db.col.find({},{"title":1,_id:0}).sort({"likes":-1})
  { "title" : "PHP 教程" }  { "title" : "Java 教程" }  { "title" : "MongoDB 教程" }  >  
```



### 索引

创建索引语句createIndex()语法：

```sql
>db.collection.createIndex(keys, options)
```

实例：创建“title”为索引字段，1为按升序创建索引，-1为按降序创建索引

```sql
>db.col.createIndex({"title":1})
```

使用多个字段创建索引：

```sql
>db.col.createIndex({"title":1,"description":-1})
```



可选参数：

| Parameter          | Type          | Description                                                  |
| :----------------- | :------------ | :----------------------------------------------------------- |
| background         | Boolean       | 建索引过程会阻塞其它数据库操作，background可指定以后台方式创建索引，即增加 "background" 可选参数。 "background" 默认值为**false**。 |
| unique             | Boolean       | 建立的索引是否唯一。指定为true创建唯一索引。默认值为**false**. |
| name               | string        | 索引的名称。如果未指定，MongoDB的通过连接索引的字段名和排序顺序生成一个索引名称。 |
| dropDups           | Boolean       | **3.0+版本已废弃。**在建立唯一索引时是否删除重复记录,指定 true 创建唯一索引。默认值为 **false**. |
| sparse             | Boolean       | 对文档中不存在的字段数据不启用索引；这个参数需要特别注意，如果设置为true的话，在索引字段中不会查询出不包含对应字段的文档.。默认值为 **false**. |
| expireAfterSeconds | integer       | 指定一个以秒为单位的数值，完成 TTL设定，设定集合的生存时间。 |
| v                  | index version | 索引的版本号。默认的索引版本取决于mongod创建索引时运行的版本。 |
| weights            | document      | 索引权重值，数值在 1 到 99,999 之间，表示该索引相对于其他索引字段的得分权重。 |
| default_language   | string        | 对于文本索引，该参数决定了停用词及词干和词器的规则的列表。 默认为英语 |
| language_override  | string        | 对于文本索引，该参数指定了包含在文档中的字段名，语言覆盖默认的language，默认值为 language. |



### 聚合操作

MongoDB 中聚合(aggregate)主要用于处理数据(诸如统计平均值，求和等)，并返回计算后的数据结果。

有点类似 **SQL** 语句中的 **count(\*)**。

------

#### aggregate() 方法

MongoDB中聚合的方法使用aggregate()。

##### 语法

aggregate() 方法的基本语法格式如下所示：

```
>db.COLLECTION_NAME.aggregate(AGGREGATE_OPERATION)
```

##### 实例

集合中的数据如下：

```
{
   _id: ObjectId(7df78ad8902c)
   title: 'MongoDB Overview', 
   description: 'MongoDB is no sql database',
   by_user: 'runoob.com',
   url: 'http://www.runoob.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 100
},
{
   _id: ObjectId(7df78ad8902d)
   title: 'NoSQL Overview', 
   description: 'No sql database is very fast',
   by_user: 'runoob.com',
   url: 'http://www.runoob.com',
   tags: ['mongodb', 'database', 'NoSQL'],
   likes: 10
},
{
   _id: ObjectId(7df78ad8902e)
   title: 'Neo4j Overview', 
   description: 'Neo4j is no sql database',
   by_user: 'Neo4j',
   url: 'http://www.neo4j.com',
   tags: ['neo4j', 'database', 'NoSQL'],
   likes: 750
},
```

现在我们通过以上集合计算每个作者所写的文章数，使用aggregate()计算结果如下：

```
> db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : 1}}}])
{
   "result" : [
      {
         "_id" : "runoob.com",
         "num_tutorial" : 2
      },
      {
         "_id" : "Neo4j",
         "num_tutorial" : 1
      }
   ],
   "ok" : 1
}
>
```

以上实例类似sql语句：

```
 select by_user, count(*) from mycol group by by_user
```

在上面的例子中，我们通过字段 by_user 字段对数据进行分组，并计算 by_user 字段相同值的总和。

下表展示了一些聚合的表达式:

| 表达式    | 描述                                           | 实例                                                         |
| :-------- | :--------------------------------------------- | :----------------------------------------------------------- |
| $sum      | 计算总和。                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$sum : "$likes"}}}]) |
| $avg      | 计算平均值                                     | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$avg : "$likes"}}}]) |
| $min      | 获取集合中所有文档对应值得最小值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$min : "$likes"}}}]) |
| $max      | 获取集合中所有文档对应值得最大值。             | db.mycol.aggregate([{$group : {_id : "$by_user", num_tutorial : {$max : "$likes"}}}]) |
| $push     | 在结果文档中插入值到一个数组中。               | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$push: "$url"}}}]) |
| $addToSet | 在结果文档中插入值到一个数组中，但不创建副本。 | db.mycol.aggregate([{$group : {_id : "$by_user", url : {$addToSet : "$url"}}}]) |
| $first    | 根据资源文档的排序获取第一个文档数据。         | db.mycol.aggregate([{$group : {_id : "$by_user", first_url : {$first : "$url"}}}]) |
| $last     | 根据资源文档的排序获取最后一个文档数据         | db.mycol.aggregate([{$group : {_id : "$by_user", last_url : {$last : "$url"}}}]) |

------

#### 管道的概念

管道在Unix和Linux中一般用于将当前命令的输出结果作为下一个命令的参数。

MongoDB的聚合管道将MongoDB文档在一个管道处理完毕后将结果传递给下一个管道处理。管道操作是可以重复的。

表达式：处理输入文档并输出。表达式是无状态的，只能用于计算当前聚合管道的文档，不能处理其它的文档。

这里我们介绍一下聚合框架中常用的几个操作：

- $project：修改输入文档的结构。可以用来重命名、增加或删除域，也可以用于创建计算结果以及嵌套文档。
- $match：用于过滤数据，只输出符合条件的文档。$match使用MongoDB的标准查询操作。
- $limit：用来限制MongoDB聚合管道返回的文档数。
- $skip：在聚合管道中跳过指定数量的文档，并返回余下的文档。
- $unwind：将文档中的某一个数组类型字段拆分成多条，每条包含数组中的一个值。
- $group：将集合中的文档分组，可用于统计结果。
- $sort：将输入文档排序后输出。
- $geoNear：输出接近某一地理位置的有序文档。

##### 管道操作符实例

1、$project实例



```
db.article.aggregate(
    { $project : {
        title : 1 ,
        author : 1 ,
    }}
 );
```

这样的话结果中就只还有_id,tilte和author三个字段了，默认情况下_id字段是被包含的，如果要想不包含_id话可以这样:

```
db.article.aggregate(
    { $project : {
        _id : 0 ,
        title : 1 ,
        author : 1
    }});
```

2.$match实例

```
db.articles.aggregate( [
                        { $match : { score : { $gt : 70, $lte : 90 } } },
                        { $group: { _id: null, count: { $sum: 1 } } }
                       ] );
```

$match用于获取分数大于70小于或等于90记录，然后将符合条件的记录送到下一阶段$group管道操作符进行处理。

3.$skip实例

```
db.article.aggregate(
    { $skip : 5 });
```

经过$skip管道操作符处理后，前五个文档被"过滤"掉。





## 复制（副本集）



## 分片



## 备份和恢复



## 监控





## Mongodb Java

官方网址：https://mongodb.github.io/mongo-java-driver/





Mongodb索引可以随时建立？

有数量限制吗？

对性能有什么影响？

Mangodb事务支持





## Mongodb高级操作

### 高级索引





# 压测工具

## AB

ab,Apache HTTP server benchmarking tool

## Jmeter



https://www.cnblogs.com/cjsblog/p/9038838.html