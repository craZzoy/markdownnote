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


​            
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
