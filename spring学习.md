项目：D:\projects\github\personal\spring-learn\think-in-spring



# 总览

## spring中的编程模型



Aware回调

BeanPostProcessor后置处理

```java
//
// Source code recreated from a .class file by IntelliJ IDEA
// (powered by Fernflower decompiler)
//

package org.springframework.beans.factory.config;

import org.springframework.beans.BeansException;
import org.springframework.lang.Nullable;

public interface BeanPostProcessor {
    @Nullable
    default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }

    @Nullable
    default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        return bean;
    }
}

```

## 面试题



# IOC

IOC主要实现策略

- service locator pattern:服务定位模式，如通过JNDI获得即JAVA EE组件



JAVA EE

J2EE



实现策略主要分为两种类型

- 依赖查找：callback机制？
- 依赖注入
  - 构造器
  - 方法
  - 属性
  - 接口



## IOC容器的职责



## 传统IOC容器的实现

### Java Bean作为IOC容器





# IOC容器概述

## Spring IOC依赖查找

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <bean id="user" class="com.geekbang.ioc.overview.dependency.domain.User">
        <property name="id" value="1" />
        <property name="age" value="18" />
        <property name="name" value="Jack"/>
    </bean>


    <bean id="superUser" class="com.geekbang.ioc.overview.dependency.domain.SuperUser">
        <property name="address" value="SHENZHEN" />
    </bean>


    <bean id="objectFactory" class="org.springframework.beans.factory.config.ObjectFactoryCreatingFactoryBean">
        <property name="targetBeanName" value="user" />
    </bean>

</beans>

```

- 根据Bean名称查找
  - 实时查找
  - 延迟查找
- 根据Bean类型查找
  - 单个Bean
  - 集合Bean
- 根据Bean名称+类型查找
- 根据java注解查找
  - 单个Bean
  - 集合Bean

```java
package com.geekbang.ioc.overview.dependency.lookup;

import com.geekbang.ioc.overview.dependency.annotation.Super;
import com.geekbang.ioc.overview.dependency.domain.User;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.beans.factory.ListableBeanFactory;
import org.springframework.beans.factory.ObjectFactory;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.util.Map;

public class DependencyLookupDemo {


    public static void main(String[] args) {
        BeanFactory factory = new ClassPathXmlApplicationContext("classpath:/META-INF/dependency-lookup-context.xml");

        lookupInRealTime(factory);

        lookupInLazy(factory);

        //lookupByType(factory);

        lookupByNameAndType(factory);

        lookupCollectionByType(factory);


        lookByAnnotationType(factory);

    }

    /**
     * 根据名称和类型查找
     * @param factory
     */
    private static void lookupByNameAndType(BeanFactory factory) {
        User user = factory.getBean("user", User.class);
        System.out.println(user);
    }

    /**
     * 根据类型查找集合
     * @param factory
     */
    private static void lookupCollectionByType(BeanFactory factory) {

        if(factory instanceof ListableBeanFactory){
            ListableBeanFactory listableBeanFactory = (ListableBeanFactory) factory;
            Map<String, User> beansOfType = listableBeanFactory.getBeansOfType(User.class);
            System.out.println(beansOfType);
        }

    }

    /**
     * 根据类型查找
     * @param factory
     */
    private static void lookupByType(BeanFactory factory) {
        User bean = factory.getBean(User.class);
        System.out.println(bean);
    }

    /**
     * 根据注解查找
     * @param factory
     */
    private static void lookByAnnotationType(BeanFactory factory) {
        if (factory instanceof ListableBeanFactory){
            ListableBeanFactory beanFactory = (ListableBeanFactory) factory;
            Map<String, Object> beansWithAnnotation = beanFactory.getBeansWithAnnotation(Super.class);
            System.out.println("根据Super注解找到的class：" + beansWithAnnotation);
        }
    }

    /**
     * 延迟查找
     * @param factory
     */
    private static void lookupInLazy(BeanFactory factory) {
        ObjectFactory<User> objectFactory = (ObjectFactory<User>) factory.getBean("objectFactory");
        User object = objectFactory.getObject();
        System.out.println(object);
    }

    /**
     * 实时查找
     * @param factory
     */
    private static void lookupInRealTime(BeanFactory factory) {
        User bean = (User) factory.getBean("user");
        System.out.println(bean);
    }
}

```





ObjectFactory没有生成新的Bean？而FactoryBean有？



## Spring IOC依赖注入

- 根据 Bean 名称注入
- 根据 Bean 类型注入
  - 单个 Bean 对象
  - 集合 Bean 对象
- 注入容器內建 Bean 对象
- 注入非 Bean 对象（内建依赖）
- 注入类型
  - 实时注入
  - 延迟注入  



## Spring IOC依赖来源

- 自定义Bean
- 容器内建Bean
- 容器内建依赖

示例代码：

```java
package com.geekbang.ioc.overview.dependency.repository;

import com.geekbang.ioc.overview.dependency.domain.User;
import org.springframework.beans.factory.BeanFactory;

import java.util.Collection;

/**
 * 用户信息仓库
 */
public class UserRepository {

    private Collection<User> users; //自定义bean

    private BeanFactory beanFactory;


    public Collection<User> getUsers() {
        return users;
    }

    public void setUsers(Collection<User> users) {
        this.users = users;
    }

    public BeanFactory getBeanFactory() {
        return beanFactory;
    }

    public void setBeanFactory(BeanFactory beanFactory) {
        this.beanFactory = beanFactory;
    }
}

```



```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:util="http://www.springframework.org/schema/util"
       xsi:schemaLocation="
        http://www.springframework.org/schema/beans https://www.springframework.org/schema/beans/spring-beans.xsd
        http://www.springframework.org/schema/util https://www.springframework.org/schema/util/spring-util.xsd">

    <import resource="dependency-lookup-context.xml" />
    
    <bean id="userRepository" class="com.geekbang.ioc.overview.dependency.repository.UserRepository"
    autowire="byType"> <!-- 根据类型自动注入-->
        <!-- 手动配置（硬编码） -->
        <!--<property name="users">
            <util:list>
                <ref bean="user" />
                <ref bean="superUser" />
            </util:list>
        </property>-->
    </bean>

</beans>

```

```java
package com.geekbang.ioc.overview.dependency.injection;

import com.geekbang.ioc.overview.dependency.repository.UserRepository;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.context.ApplicationContext;
import org.springframework.context.support.ClassPathXmlApplicationContext;
import org.springframework.core.env.Environment;

/**
 * 依赖注入
 */
public class DependencyInjectionDemo {

    public static void main(String[] args) {
        BeanFactory beanFactory = new ClassPathXmlApplicationContext("classpath:/META-INF/dependency-injection-context.xml");

        //依赖来源一：自定义对象
        UserRepository userRepository = beanFactory.getBean("userRepository", UserRepository.class);
        System.out.println(userRepository.getUsers());

        //依赖来源二：依赖注入（内建依赖）: 依赖注入的beanFactory和用于依赖查找的BeanFactory并非同一个
        //解答26节
        System.out.println(userRepository.getBeanFactory());
        System.out.println(beanFactory);
        System.out.println(beanFactory == userRepository.getBeanFactory());

        //错误查找，不用在依赖查找中找到BeanFactory
        //System.out.println(beanFactory.getBean(BeanFactory.class));

        //依赖来源三、容器内建Bean
        Environment bean = beanFactory.getBean(Environment.class);
        System.out.println("获取到的Environment类型的Bean：" + bean);

    }

}

```



## Spring IOC配置元信息

- Bean定义配置

  - 基于XML文件
  - 基于properties文件
  - 基于Java注解
  - 基于Java API（专题）
- IOC容器配置
- 基于XML文件
  - 基于Java文件
- 基于Java API（专题）
- 外部化Java配置
  - 基于Java注解



## Spring IOC容器

ApplicationContext和BeanFactory谁才是IOC容器

![image-20201129104413223](spring学习.assets\image-20201129104413223.png)

查看ApplicationContext的继承实现图，可见ApplicationContext实现了BeanFactory接口，可以说两者都是IOC容器，ApplicationContext在BeanFactory的基础上做了一些拓展，其除了作为IOC容器角色外，还提供：

- 面向切面（AOP）
- 配置元信息（Configuration Metadata）
- 资源管理（Resource）
- 事件（Event）
- 国际化（i18n）
- 注解（Annotation）
- Environment抽象（Environment Abstraction）

但是，实际上ApplicationContext依赖查找时，其还是通过其内建实现的一个组合BeanFactory其查找的，毕竟对于Bean的操作基本上包含在BeanFactory及其实现类接口中。

注意ApplicationContext的GetBean方法，其中组建了一个组合BeanFactory，查找依赖时是使用其查找的



## 使用Spring IOC容器

- BeanFactory作为IOC容器

  ```java
  package com.geekbang.ioc.overview.dependency.container;
  
  import com.geekbang.ioc.overview.dependency.domain.User;
  import org.springframework.beans.factory.BeanFactory;
  import org.springframework.beans.factory.ListableBeanFactory;
  import org.springframework.beans.factory.support.DefaultListableBeanFactory;
  import org.springframework.beans.factory.xml.XmlBeanDefinitionReader;
  
  import java.util.Map;
  
  /**
   * {@link BeanFactory} BeanFactory作为IOC容器
   */
  public class BeanFactoryAsIocContainerDemo {
  
      public static void main(String[] args) {
          DefaultListableBeanFactory beanFactory = new DefaultListableBeanFactory();
          XmlBeanDefinitionReader reader = new XmlBeanDefinitionReader(beanFactory);
          String location = "classpath:/META-INF/dependency-lookup-context.xml";
          //加载配置
          int i = reader.loadBeanDefinitions(location);
          System.out.println("查找到的Bean定义数量：" + i);
          //依赖查找集合对象
          lookupCollectionByType(beanFactory);
      }
  
  
      /**
       * 根据类型查找集合
       * @param factory
       */
      private static void lookupCollectionByType(BeanFactory factory) {
  
          if(factory instanceof ListableBeanFactory){
              ListableBeanFactory listableBeanFactory = (ListableBeanFactory) factory;
              Map<String, User> beansOfType = listableBeanFactory.getBeansOfType(User.class);
              System.out.println(beansOfType);
          }
  
      }
  
  }
  
  ```

- ApplicationContext作为IOC容器

  ```java
  package com.geekbang.ioc.overview.dependency.container;
  
  import com.geekbang.ioc.overview.dependency.domain.User;
  import org.springframework.beans.factory.BeanFactory;
  import org.springframework.beans.factory.ListableBeanFactory;
  import org.springframework.context.annotation.AnnotationConfigApplicationContext;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  
  import java.util.Map;
  
  /**
   * {@link org.springframework.context.ApplicationContext} 作为IOC容器
   */
  @Configuration
  public class ApplicationContextAsIocContainerDemo {
  
      public static void main(String[] args) {
          //创建BeanFactory容器
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
          //注册配置类
          context.register(ApplicationContextAsIocContainerDemo.class);
          //启动上下文
          context.refresh();
          lookupCollectionByType(context);
          context.close();
      }
  
      @Bean
      public User user (){
          User user = new User();
          user.setName("测试用户");
          user.setAge(18);
          return user;
      }
  
      /**
       * 根据类型查找集合
       * @param factory
       */
      private static void lookupCollectionByType(BeanFactory factory) {
  
          if(factory instanceof ListableBeanFactory){
              ListableBeanFactory listableBeanFactory = (ListableBeanFactory) factory;
              Map<String, User> beansOfType = listableBeanFactory.getBeansOfType(User.class);
              System.out.println(beansOfType);
          }
  
      }
  
  }
  
  ```

  



IDEA UML类图使用

https://blog.csdn.net/zj420964597/article/details/87856758

https://www.cnblogs.com/LDZZDL/p/9061603.html



## Spring IOC容器的生命周期

1. 启动：org.springframework.context.support.AbstractApplicationContext#refresh

   ```java
   	@Override
   	public void refresh() throws BeansException, IllegalStateException {
   		synchronized (this.startupShutdownMonitor) {
   			// Prepare this context for refreshing.
   			prepareRefresh();
   
   			// Tell the subclass to refresh the internal bean factory.
   			ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
   
   			// Prepare the bean factory for use in this context.
   			prepareBeanFactory(beanFactory);
   
   			try {
   				// Allows post-processing of the bean factory in context subclasses.
   				postProcessBeanFactory(beanFactory);
   
   				// Invoke factory processors registered as beans in the context.
   				invokeBeanFactoryPostProcessors(beanFactory);
   
   				// Register bean processors that intercept bean creation.
   				registerBeanPostProcessors(beanFactory);
   
   				// Initialize message source for this context.
   				initMessageSource();
   
   				// Initialize event multicaster for this context.
   				initApplicationEventMulticaster();
   
   				// Initialize other special beans in specific context subclasses.
   				onRefresh();
   
   				// Check for listener beans and register them.
   				registerListeners();
   
   				// Instantiate all remaining (non-lazy-init) singletons.
   				finishBeanFactoryInitialization(beanFactory);
   
   				// Last step: publish corresponding event.
   				finishRefresh();
   			}
   
   			catch (BeansException ex) {
   				if (logger.isWarnEnabled()) {
   					logger.warn("Exception encountered during context initialization - " +
   							"cancelling refresh attempt: " + ex);
   				}
   
   				// Destroy already created singletons to avoid dangling resources.
   				destroyBeans();
   
   				// Reset 'active' flag.
   				cancelRefresh(ex);
   
   				// Propagate exception to caller.
   				throw ex;
   			}
   
   			finally {
   				// Reset common introspection caches in Spring's core, since we
   				// might not ever need metadata for singleton beans anymore...
   				resetCommonCaches();
   			}
   		}
   	}
   ```

   

2. 运行

3. 停止：org.springframework.context.support.AbstractApplicationContext#close

   ```java
   	@Override
   	public void close() {
   		synchronized (this.startupShutdownMonitor) {
   			doClose();
   			// If we registered a JVM shutdown hook, we don't need it anymore now:
   			// We've already explicitly closed the context.
   			if (this.shutdownHook != null) {
   				try {
   					Runtime.getRuntime().removeShutdownHook(this.shutdownHook);
   				}
   				catch (IllegalStateException ex) {
   					// ignore - VM is already shutting down
   				}
   			}
   		}
   	}
   ```

   > applicationContextd关闭时调用了通用关闭资源的方法，还删除了当前应用中关闭钩子，关于关闭钩子，可参考https://www.cnblogs.com/maxstack/p/9112711.html

   



## 面试题

1. 什么是Spring IOC容器
2. BeanFactory和FactoryBean
3. Spring IOC容器启动时做了哪些准备



# Spring Bean基础

## BeanDefinition接口

beanDefinition接口是Spring FrameWork提供的定义Bean的配置元信息接口，其中包含信息：

- Bean名称
- Bean行为配置元素，如作用域、自动绑定的方式、生命周期的回调等
- 其他Bean引用
  - 合作者（collaborators）
  - 依赖（dependencies）
- 配置设置，如Bean属性（properties）
- 其他

![image-20201216101927357](spring学习.assets/image-20201216101927357.png)



## BeanDefinition元信息

| 属性（Property）         | 说明                                          |
| ------------------------ | --------------------------------------------- |
| Class                    | Bean 全类名，必须是具体类，不能用抽象类或接口 |
| Name                     | Bean 的名称或者 ID                            |
| Scope                    | Bean 的作用域（如：singleton、 prototype 等） |
| Constructor arguments    | Bean 构造器参数（用于依赖注入）               |
| Properties               | Bean 属性设置（用于依赖注入）                 |
| Autowiring mode          | Bean 自动绑定模式（如：通过名称 byName）      |
| Lazy initialization mode | Bean 延迟初始化模式（延迟和非延迟）           |
| Initialization method    | Bean 初始化回调方法名称                       |
| Destruction method       | Bean 销毁回调方法名称                         |

### 原生BeanDefinition创建方式

- 通过 BeanDefinitionBuilder  
- 通过 AbstractBeanDefinition 以及派生类 

```java
package org.geekbang.thinking.in.spring.bean.definition;

import org.geekbang.thinking.in.spring.bean.domain.User;
import org.springframework.beans.MutablePropertyValues;
import org.springframework.beans.factory.config.BeanDefinition;
import org.springframework.beans.factory.support.AbstractBeanDefinition;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.GenericBeanDefinition;

/**
 * @Description: BeanDefinition创建demo
 *
 */
public class BeanDefinitionGenerationDemo {

    public static void main(String[] args) {
        //1、通过BeanDefinitionBuilder构建
        BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(User.class);
        //设置Bean属性
        beanDefinitionBuilder.addPropertyValue("id", 1);
        beanDefinitionBuilder.addPropertyValue("name", "JackMa");
        //获取BeanDefinition
        BeanDefinition beanDefinition = beanDefinitionBuilder.getBeanDefinition();
        System.out.println(beanDefinition.getBeanClassName());


        //2、通过AbstractBeanDefinition构建
        final GenericBeanDefinition genericBeanDefinition = new GenericBeanDefinition();
        genericBeanDefinition.setBeanClass(User.class);
        MutablePropertyValues mutablePropertyValues = new MutablePropertyValues();
        mutablePropertyValues.addPropertyValue("id", 1);
        mutablePropertyValues.addPropertyValue("name", "JackMa");
        genericBeanDefinition.setPropertyValues(mutablePropertyValues);
    }

}
```



### 使用Annotation使上下文自动创建BeanDefinition

```java
package org.geekbang.thinking.in.spring.bean.definition;

import com.geekbang.ioc.overview.dependency.domain.User;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Import;
import org.springframework.stereotype.Component;

/**
 * @Description: 注解BeanDefinition demo
 * @Author :
 * @Date : 15:00 2020/12/16
 */
@Import(AnnotationBeanDefinitionDemo.Config.class)
public class AnnotationBeanDefinitionDemo {

    public static void main(String[] args) {
        //创建Bean容器
        final AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        //注册配置Bean
        context.register(AnnotationBeanDefinitionDemo.class);
        //启动应用上下文
        context.refresh();
        System.out.println("user 类型的所有Bean：" + context.getBeansOfType(User.class));
        //关闭上下文
        context.close();

    }


    @Component
    public static class Config {

        @Bean
        public User user (){
            User user1 = new User();
            user1.setName("杰克马");
            user1.setAge(35);
            return user1;
        }
        
    }

}

```







## 命名Spring Bean

### Bean名称

每个 Bean 拥有一个或多个标识符（identifiers），这些标识符在 Bean 所在的容器必须是唯一
的。通常，一个 Bean 仅有一个标识符，如果需要额外的，可考虑使用别名（Alias）来扩充。
在基于 XML 的配置元信息中，开发人员可用 id 或者 name 属性来规定 Bean 的 标识符。通常
Bean 的 标识符由字母组成，允许出现特殊字符。如果要想引入 Bean 的别名的话，可在
name 属性使用半角逗号（“,”）或分号（“;”) 来间隔。
Bean 的 id 或 name 属性并非必须制定，如果留空的话，容器会为 Bean 自动生成一个唯一的
名称。Bean 的命名尽管没有限制，不过官方建议采用驼峰的方式，更符合 Java 的命名约定  

通过BeanDefinition生成命名和非命名Bean：

```java
package org.geekbang.thinking.in.spring.bean.definition;

import com.geekbang.ioc.overview.dependency.domain.User;
import org.springframework.beans.factory.support.BeanDefinitionBuilder;
import org.springframework.beans.factory.support.BeanDefinitionReaderUtils;
import org.springframework.beans.factory.support.BeanDefinitionRegistry;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.util.StringUtils;

/**
 * @Description: 命名Bean Demo
 * @Author : 
 * @Date : 10:05 2020/12/21
 */
public class NamedBeanDemo {

    public static void main(String[] args) {
        //创建Bean容器
        final AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        //通过BeanDefinition API注册Bean
        //命名注册Bean
        registerUserBeanDefinition(context, "naming-user");
        //非命名注册Bean
        registerUserBeanDefinition(context, null);
        //启动应用上下文
        context.refresh();
        System.out.println("user 类型的所有Bean：" + context.getBeansOfType(User.class));
        //关闭上下文
        context.close();

    }


    public static void registerUserBeanDefinition(BeanDefinitionRegistry registry, String name){
        final BeanDefinitionBuilder beanDefinitionBuilder = BeanDefinitionBuilder.genericBeanDefinition(User.class);
        beanDefinitionBuilder
                .addPropertyValue("id", 1)
                .addPropertyValue("name", "JackMa");
        if (StringUtils.hasText(name)){
            //命名注册Bean
            registry.registerBeanDefinition(name,beanDefinitionBuilder.getBeanDefinition());
        } else {
            //非命名注册Bean
            BeanDefinitionReaderUtils.registerWithGeneratedName(beanDefinitionBuilder.getBeanDefinition(), registry);
        }
    }

}

```

> user 类型的所有Bean：{naming-user=User{id=1, name='JackMa', age=null}, com.geekbang.ioc.overview.dependency.domain.User#0=User{id=1, name='JackMa', age=null}}



### Bean名称生成器

- org.springframework.beans.factory.support.BeanNameGenerato（since 2.0.3）：

  ```java
  /*
   * Copyright 2002-2007 the original author or authors.
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
  
  package org.springframework.beans.factory.support;
  
  import org.springframework.beans.factory.config.BeanDefinition;
  
  /**
   * Strategy interface for generating bean names for bean definitions.
   *
   * @author Juergen Hoeller
   * @since 2.0.3
   */
  public interface BeanNameGenerator {
  
  	/**
  	 * Generate a bean name for the given bean definition.
  	 * @param definition the bean definition to generate a name for
  	 * @param registry the bean definition registry that the given definition
  	 * is supposed to be registered with
  	 * @return the generated bean name
  	 */
  	String generateBeanName(BeanDefinition definition, BeanDefinitionRegistry registry);
  
  }
  
  ```

  其中generateBeanName()方法中使用的BeanDefinitionRegistry类提供了对容器中BeanDefinition做一些操作的API：

  ![image-20201221110359652](spring学习.assets/image-20201221110359652.png)

  > 一般IOC容器（BeanFactory和ApplicationContext）实现都会实现这个接口

  - org.springframework.beans.factory.support.DefaultBeanNameGenerator（since 2.0.3）：适用于普通的Beandefinition，未指定名称时调用
  - org.springframework.context.annotation.AnnotationBeanNameGenerator（since 2.5）：适用于org.springframework.beans.factory.annotation.AnnotatedBeanDefinition类型的Beandefinition，注解生成的Bean调用



## Bean别名

- Bean 别名（Alias）的价值

  - 复用现有的 BeanDefinition

  - 更具有场景化的命名方法，比如：
    <alias name="myApp-dataSource" alias="subsystemA-dataSource"/>
    <alias name="myApp-dataSource" alias="subsystemB-dataSource"/>  

    ```java
    @Bean(name = {"myApp-dataSource", "subsystemA-dataSource"})
    ```

```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
        https://www.springframework.org/schema/beans/spring-beans.xsd">

    <import resource="classpath:/META-INF/dependency-lookup-context.xml"/>
    
    <alias name="user" alias="alias-test-user"/>
    <alias name="user" alias="alias-test-user1" />

    <bean class="com.geekbang.ioc.overview.dependency.domain.User" />

</beans>

```

```java
package org.geekbang.thinking.in.spring.bean.definition;

import com.geekbang.ioc.overview.dependency.domain.User;
import org.springframework.beans.factory.BeanFactory;
import org.springframework.context.support.ClassPathXmlApplicationContext;

import java.util.Arrays;

/**
 * @Description: 别名Bean demo
 * @Author : 
 * @Date : 10:08 2020/12/21
 */
public class BeanAliasDemo {

    public static void main(String[] args) {
        BeanFactory beanFactory = new ClassPathXmlApplicationContext("classpath:/META-INF/bean-definitions-context.xml");
        final User user = beanFactory.getBean("user", User.class);
        //通过别名获取Bean
        final User user1 = beanFactory.getBean("alias-test-user", User.class);
        System.out.println(Arrays.asList(beanFactory.getAliases("user")));
        System.out.println("user == alias-test-user : " + (user == user1));
    }

}

```

> [alias-test-user, alias-test-user1]
> user == alias-test-user : true





## 注册Spring Bean

- Beandefinition注册

  - XML 配置元信息

    -  <bean name=”...” ... />

  -  Java 注解配置元信息

    - @Bean
    - @Component
    - @Import

  -  Java API 配置元信息

    - 命名方式：BeanDefinitionRegistry#registerBeanDefinition(String,BeanDefinition)

    - 非命名方式：BeanDefinitionReaderUtils#registerWithGeneratedName(AbstractBeanDefinition,Be
      anDefinitionRegistry)

    - 配置类方式：AnnotatedBeanDefinitionReader#register(Class...)  

      ```java
      package org.geekbang.thinking.in.spring.bean.definition;
      
      import com.geekbang.ioc.overview.dependency.domain.User;
      import org.springframework.context.annotation.AnnotatedBeanDefinitionReader;
      import org.springframework.context.annotation.AnnotationConfigApplicationContext;
      import org.springframework.context.annotation.Bean;
      import org.springframework.stereotype.Component;
      
      import java.util.Map;
      
      /**
       * AnnotatedBeanDefinitionReader demo
       */
      public class AnnotatedBeanDefinitionReaderDemo {
      
          public static void main(String[] args) {
              AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
              AnnotatedBeanDefinitionReader beanDefinitionReader = new AnnotatedBeanDefinitionReader(context);
              beanDefinitionReader.register(Config.class);
              context.refresh();
              Map<String, User> beansOfType = context.getBeansOfType(User.class);
              System.out.println(beansOfType);
              context.close();
          }
      
      
          @Component
          public static class Config {
      
              @Bean(name = {"user", "user-alias"})
              public User user (){
                  User user1 = new User();
                  user1.setName("杰克马");
                  user1.setAge(35);
                  return user1;
              }
          }
      }
      
      ```

  > {user=User{id=null, name='杰克马', age=35}}

- 外部单例对象注册

  - Java API 配置元信息
    
    - SingletonBeanRegistry#registerSingleton  
    
    ```java
    package org.geekbang.thinking.in.spring.bean.definition;
    
    
    import org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory;
    import org.geekbang.thinking.in.spring.bean.factory.UserFactory;
    import org.springframework.beans.factory.config.ConfigurableListableBeanFactory;
    import org.springframework.beans.factory.config.SingletonBeanRegistry;
    import org.springframework.context.annotation.AnnotationConfigApplicationContext;
    
    /**
     * 外部化单例Bean注册 demo
     * 即先手动创建Bean，然后注册到Spring上下文中
     */
    public class SingleBeanRegistionDemo {
    
        public static void main(String[] args) {
            AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
            SingletonBeanRegistry beanFactory = context.getBeanFactory();
            UserFactory userFactory = new DefaultUserFactory();
    
            //注册外部单例对象，外部单例指外部手动创建的、不由IOC控制创建（通过BeanDefinition创建）过程的Bean
            beanFactory.registerSingleton("userFactory", userFactory);
    
            context.refresh();
    
            UserFactory bean = context.getBean(UserFactory.class);
            System.out.println(bean);
            System.out.println("bean == userFactory : " + (bean == userFactory));
    
            context.close();
        }
    
    }
    
    ```
    
    > org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory@150c158
    > bean == userFactory : true



## 实例化Bean

- 常规方式

  - 通过构造器（配置元信息：XML、Java 注解和 Java API  )
    - XML方式
    - JAVA注解
    - JAVA API
  - 通过静态工厂方法（配置元信息：XML和 Java API  )
  - 通过Bean（实例化）工厂方法（配置元信息：XML和 Java API  )
  - 通过FactoryBean（配置元信息：XML、Java 注解和 Java API  )

  User.class：

  ```java
  package com.geekbang.ioc.overview.dependency.domain;
  
  public class User {
  
      private Integer id;
  
      private String name;
  
      private Integer age;
  
      public Integer getId() {
          return id;
      }
  
      public void setId(Integer id) {
          this.id = id;
      }
  
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
  
      public static User createUser(){
          User user = new User();
          user.setName("Tom Cat");
          user.setId(10);
          user.setAge(12);
          return user;
      }
  
      @Override
      public String toString() {
          return "User{" +
                  "id=" + id +
                  ", name='" + name + '\'' +
                  ", age=" + age +
                  '}';
      }
  }
  
  ```

  UserFactory接口：

  ```java
  package org.geekbang.thinking.in.spring.bean.factory;
  
  import com.geekbang.ioc.overview.dependency.domain.User;
  
  public interface UserFactory {
  
      default User createUser(){
          return User.createUser();
      }
  
  }
  
  ```

  UserFactory接口实现类：

  ```java
  package org.geekbang.thinking.in.spring.bean.factory;
  
  import com.geekbang.ioc.overview.dependency.domain.User;
  
  public class DefaultUserFactory implements UserFactory {
  }
  
  ```

  

  UserFactoryBean：

  ```java
  package org.geekbang.thinking.in.spring.bean.factory;
  
  import com.geekbang.ioc.overview.dependency.domain.User;
  import org.springframework.beans.factory.FactoryBean;
  
  public class UserFactoryBean implements FactoryBean<User> {
      @Override
      public User getObject() throws Exception {
          return User.createUser();
      }
  
      @Override
      public Class<?> getObjectType() {
          return User.class;
      }
  }
  
  ```

  

  

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <!-- 构造器实例化 -->
      <bean id="user-by-constructor-method" class="com.geekbang.ioc.overview.dependency.domain.User"/>
  
      <!-- 静态工厂实例化 -->
      <bean id = "user-by-static-method" class="com.geekbang.ioc.overview.dependency.domain.User" factory-method="createUser" />
  
      <!-- 实例（bean）方法实例化bean -->
      <bean id="user-by-instance-bean" factory-bean="userFactory" factory-method="createUser" />
  
      <!-- 根据factorybean实例化 -->
      <bean id = "user-by-factory-bean" class="org.geekbang.thinking.in.spring.bean.factory.UserFactoryBean" />
  
  
      <bean id="userFactory" class="org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory" />
  
  
  </beans>
  ```

  ```java
  package org.geekbang.thinking.in.spring.bean.definition;
  
  import com.geekbang.ioc.overview.dependency.domain.User;
  import org.springframework.beans.factory.BeanFactory;
  import org.springframework.context.support.ClassPathXmlApplicationContext;
  
  /**
   * Bean实例化 demo
   */
  public class BeanInstantiationDemo {
  
      public static void main(String[] args) {
          BeanFactory factory = new ClassPathXmlApplicationContext
                  ("classpath:/META-INF/bean-instantination-context.xml");
  
          User userByConstructorMethod = factory.getBean("user-by-constructor-method", User.class);
          User userByStaticMethod = factory.getBean("user-by-static-method", User.class);
          User userByInstanceBean = factory.getBean("user-by-instance-bean", User.class);
          User userByFactoryBean = factory.getBean("user-by-factory-bean", User.class);
  
          System.out.println(userByConstructorMethod);
          System.out.println(userByStaticMethod);
          System.out.println(userByInstanceBean);
          System.out.println(userByFactoryBean);
  
          System.out.println(userByStaticMethod == userByInstanceBean);
          System.out.println(userByStaticMethod == userByFactoryBean);
  
      }
  
  }
  
  ```

  >User{id=null, name='null', age=null}
  >User{id=10, name='Tom Cat', age=12}
  >User{id=10, name='Tom Cat', age=12}
  >User{id=10, name='Tom Cat', age=12}
  >false
  >false

- 特殊方式

  - 通过 ServiceLoaderFactoryBean（配置元信息：XML、Java 注解和 Java API ）  
  - 通过 AutowireCapableBeanFactory#createBean(java.lang.Class, int, boolean)  
  - 通过 BeanDefinitionRegistry#registerBeanDefinition(String,BeanDefinition)  

  示例：

  classpath:/META-INF/services路径下新增文件org.geekbang.thinking.in.spring.bean.factory.UserFactory：

  ```txt
  org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory
  org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory
  org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactoryTwo
  ```

  special-bean-instantination-context.xml：

  ```xml
  <?xml version="1.0" encoding="UTF-8"?>
  <beans xmlns="http://www.springframework.org/schema/beans"
         xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
         xsi:schemaLocation="http://www.springframework.org/schema/beans
          https://www.springframework.org/schema/beans/spring-beans.xsd">
  
      <bean id = "userFactoryServiceLoader" class="org.springframework.beans.factory.serviceloader.ServiceLoaderFactoryBean" >
          <property name="serviceType" value="org.geekbang.thinking.in.spring.bean.factory.UserFactory"/>
          <property name="singleton" value="true" />
      </bean>
  
  
  </beans>
  
  ```

  ```java
  package org.geekbang.thinking.in.spring.bean.definition;
  
  import org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory;
  import org.geekbang.thinking.in.spring.bean.factory.UserFactory;
  import org.springframework.beans.factory.config.AutowireCapableBeanFactory;
  import org.springframework.context.ApplicationContext;
  import org.springframework.context.support.ClassPathXmlApplicationContext;
  
  import java.util.Iterator;
  import java.util.ServiceLoader;
  
  /**
   * 特殊Bean实例化demo
   */
  public class SpecialBeanInstantiationDemo {
  
      public static void main(String[] args) {
  
          //通过jdk api @{java.util.ServiceLoader} 查找 ServiceLoader
          ServiceLoader serviceLoader = demoServiceLoader();
  
          //将ServiceLoader交给Spring IOC管理
          ApplicationContext context = new ClassPathXmlApplicationContext("classpath:/META-INF/special-bean-instantination-context.xml");
  
          ServiceLoader<UserFactory> userFactoryServiceLoader = context.getBean("userFactoryServiceLoader", ServiceLoader.class);
          ServiceLoader<UserFactory> userFactoryServiceLoaderTwo = context.getBean("userFactoryServiceLoader", ServiceLoader.class);
          System.out.println(userFactoryServiceLoader == userFactoryServiceLoaderTwo);
          displayServiceLoader(userFactoryServiceLoader);
  
          System.out.println(serviceLoader == userFactoryServiceLoader);
  
  
          //通过 AutowireCapableBeanFactory#createBean(java.lang.Class, int, boolean)实例化
          AutowireCapableBeanFactory autowireCapableBeanFactory = context.getAutowireCapableBeanFactory();
          UserFactory bean = autowireCapableBeanFactory.createBean(DefaultUserFactory.class);
          System.out.println(bean.createUser());
  
      }
  
      private static ServiceLoader demoServiceLoader() {
          ServiceLoader<UserFactory> serviceLoader = ServiceLoader.load(UserFactory.class, Thread.currentThread().getContextClassLoader());
          displayServiceLoader(serviceLoader);
          return serviceLoader;
      }
  
      private static void displayServiceLoader(ServiceLoader<UserFactory> userFactoryServiceLoader) {
  
          Iterator<UserFactory> iterator = userFactoryServiceLoader.iterator();
          while (iterator.hasNext()){
              UserFactory next = iterator.next();
              System.out.println(next.createUser());
          }
  
      }
  
  }
  
  ```

  >User{id=10, name='Tom Cat', age=12}
  >User{id=10, name='Tom Cat', age=12}
  >true
  >User{id=10, name='Tom Cat', age=12}
  >User{id=10, name='Tom Cat', age=12}
  >false
  >User{id=10, name='Tom Cat', age=12}

  

## 初始化Bean

- Bean初始化

  - @PostConstruct（javax.annotation.PostConstruct）标注方法：在完成依赖注入后执行，java11中已被废弃（具体从什么版本废弃有待查阅）
  - 实现`org.springframework.beans.factory.InitializingBean`接口的afterPropertiesSet()   方法
  - 自定义初始化方法
    - XML 配置：<bean init-method=”init” ... />  
    - Java 注解：@Bean(initMethod=”init”)  
    - Java API：AbstractBeanDefinition#setInitMethodName(String)  

  思考：假设以上三种方式均在同一 Bean 中定义，那么这些方法的执行顺序是怎样？  

- Bean延迟初始化

  - XML 配置：<bean lazy-init=”true” ... />  
  - Java 注解：@Lazy(true)  

  思考：当某个 Bean 定义为延迟初始化，那么，Spring 容器返回的对象与非延迟的对象存在怎样的差异？  （延迟初始化，在获取Bean之前Bean是否已经实例化？是。见源码`org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#doCreateBean`）

  ```java
  package org.geekbang.thinking.in.spring.bean.factory;
  
  import com.geekbang.ioc.overview.dependency.domain.User;
  import org.springframework.beans.factory.InitializingBean;
  
  import javax.annotation.PostConstruct;
  
  public class DefaultUserFactory implements UserFactory, InitializingBean {
  
      @PostConstruct
      public void init(){
          System.out.println("@PostConstruct：UserFactory初始化中...");
      }
  
      /**
       * 自定义初始化方法
       */
      public void initUserFactory(){
          System.out.println("自定义初始化方法initUserFactory()：UserFactory初始化中...");
      }
  
      @Override
      public void afterPropertiesSet() throws Exception {
          System.out.println("InitializingBean#afterPropertiesSet：UserFactory初始化中...");
      }
  }
  
  ```

  ```java
  package org.geekbang.thinking.in.spring.bean.definition;
  
  import org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory;
  import org.geekbang.thinking.in.spring.bean.factory.UserFactory;
  import org.springframework.context.annotation.AnnotationConfigApplicationContext;
  import org.springframework.context.annotation.Bean;
  import org.springframework.context.annotation.Configuration;
  import org.springframework.context.annotation.Lazy;
  
  /**
   * Bean初始化 demo
   */
  @Configuration
  public class InitializationDemo {
  
      public static void main(String[] args) {
          AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
          context.register(InitializationDemo.class);
  
          context.refresh();
          System.out.println("spring应用上下文已启动...");
  
          UserFactory bean = context.getBean(UserFactory.class);
          System.out.println(bean);
  
          System.out.println("spring上下文准备关闭");
          context.close();
          System.out.println("spring上下文已关闭");
      }
  
      @Bean(initMethod = "initUserFactory")
      @Lazy(value = true)
      public UserFactory userFactory(){
          return new DefaultUserFactory();
      }
  
  }
  
  ```

  延迟初始化效果：
  
  ```txt
  spring应用上下文已启动...
  @PostConstruct：UserFactory初始化中...
  InitializingBean#afterPropertiesSet：UserFactory初始化中...
  自定义初始化方法initUserFactory()：UserFactory初始化中...
  org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory@709ba3fb
  spring上下文准备关闭
  spring上下文已关闭
  ```
  
  非延迟初始化效果：
  
  ```txt
  @PostConstruct：UserFactory初始化中...
  InitializingBean#afterPropertiesSet：UserFactory初始化中...
  自定义初始化方法initUserFactory()：UserFactory初始化中...
  spring应用上下文已启动...
  org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory@7364985f
  spring上下文准备关闭
  spring上下文已关闭
  ```
  
  > 可见延迟初始化时，在获取Bean的时候初始化方法才会执行，在初始化方法中未执行。而非延迟情况下，在注册Bean且在上下文启动时初始化方法就执行了

详细源码参照：

- org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean
- org.springframework.beans.factory.support.AbstractAutowireCapableBeanFactory#initializeBean(java.lang.String, java.lang.Object, org.springframework.beans.factory.support.RootBeanDefinition)

```java
	protected Object initializeBean(final String beanName, final Object bean, @Nullable RootBeanDefinition mbd) {
		if (System.getSecurityManager() != null) {
			AccessController.doPrivileged((PrivilegedAction<Object>) () -> {
				invokeAwareMethods(beanName, bean);
				return null;
			}, getAccessControlContext());
		}
		else {
			invokeAwareMethods(beanName, bean);
		}

		Object wrappedBean = bean;
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsBeforeInitialization(wrappedBean, beanName);
		}

		try {
			invokeInitMethods(beanName, wrappedBean, mbd);
		}
		catch (Throwable ex) {
			throw new BeanCreationException(
					(mbd != null ? mbd.getResourceDescription() : null),
					beanName, "Invocation of init method failed", ex);
		}
		if (mbd == null || !mbd.isSynthetic()) {
			wrappedBean = applyBeanPostProcessorsAfterInitialization(wrappedBean, beanName);
		}

		return wrappedBean;
	}
```

最终调用方法：

```java
	protected void invokeInitMethods(String beanName, final Object bean, @Nullable RootBeanDefinition mbd)
			throws Throwable {

		boolean isInitializingBean = (bean instanceof InitializingBean);
		if (isInitializingBean && (mbd == null || !mbd.isExternallyManagedInitMethod("afterPropertiesSet"))) {
			if (logger.isTraceEnabled()) {
				logger.trace("Invoking afterPropertiesSet() on bean with name '" + beanName + "'");
			}
			if (System.getSecurityManager() != null) {
				try {
					AccessController.doPrivileged((PrivilegedExceptionAction<Object>) () -> {
						((InitializingBean) bean).afterPropertiesSet();
						return null;
					}, getAccessControlContext());
				}
				catch (PrivilegedActionException pae) {
					throw pae.getException();
				}
			}
			else {
				((InitializingBean) bean).afterPropertiesSet();
			}
		}

		if (mbd != null && bean.getClass() != NullBean.class) {
			String initMethodName = mbd.getInitMethodName();
			if (StringUtils.hasLength(initMethodName) &&
					!(isInitializingBean && "afterPropertiesSet".equals(initMethodName)) &&
					!mbd.isExternallyManagedInitMethod(initMethodName)) {
				invokeCustomInitMethod(beanName, bean, mbd);
			}
		}
	}
```

从源码可见init方法调用顺序：InitializingBean的实现方法 -> 自定义初始化方法



## 销毁Spring Bean

- @PreDestroy 标注方法
- 实现 DisposableBean 接口的 destroy() 方法
- 自定义销毁方法
  - XML 配置：<bean destroy=”destroy” ... />
  - Java 注解：@Bean(destroy=”destroy”)
  - Java API：AbstractBeanDefinition#setDestroyMethodName(String)

DefaultFactory中增加销毁方法：

```java
package org.geekbang.thinking.in.spring.bean.factory;

import com.geekbang.ioc.overview.dependency.domain.User;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.SmartInitializingSingleton;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class DefaultUserFactory implements UserFactory, InitializingBean, SmartInitializingSingleton, DisposableBean {

    @PostConstruct
    public void init(){
        System.out.println("@PostConstruct：UserFactory初始化中...");
    }

    /**
     * 自定义初始化方法
     */
    public void initUserFactory(){
        System.out.println("自定义初始化方法initUserFactory()：UserFactory初始化中...");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean#afterPropertiesSet：UserFactory初始化中...");
    }

    @Override
    public void afterSingletonsInstantiated() {
        System.out.println("SmartInitializingSingleton#afterSingletonsInstantiated: UserFactoryy已经初始化中");
    }

    @PreDestroy
    public void destory(){
        System.out.println("@PreDestroy：UserFactory销毁中...");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean#destroy：UserFactory销毁中...");
    }

    public void destroyMethod(){
        System.out.println("DefaultUserFactory#destroyMethod：UserFactory销毁中...");
    }

}

```

```java
package org.geekbang.thinking.in.spring.bean.definition;

import org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory;
import org.geekbang.thinking.in.spring.bean.factory.UserFactory;
import org.springframework.context.annotation.AnnotationConfigApplicationContext;
import org.springframework.context.annotation.Bean;
import org.springframework.context.annotation.Configuration;
import org.springframework.context.annotation.Lazy;

/**
 * Bean初始化 demo
 */
@Configuration
public class InitializationDemo {

    public static void main(String[] args) {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(InitializationDemo.class);

        context.refresh();
        System.out.println("spring应用上下文已启动...");

        UserFactory bean = context.getBean(UserFactory.class);
        System.out.println(bean);

        System.out.println("spring上下文准备关闭");
        context.close();
        System.out.println("spring上下文已关闭");
    }

    @Bean(initMethod = "initUserFactory", destroyMethod = "destroyMethod")
    @Lazy(value = false)
    public UserFactory userFactory(){
        return new DefaultUserFactory();
    }

}

```

运行结果：

```txt
@PostConstruct：UserFactory初始化中...
InitializingBean#afterPropertiesSet：UserFactory初始化中...
自定义初始化方法initUserFactory()：UserFactory初始化中...
SmartInitializingSingleton#afterSingletonsInstantiated: UserFactoryy已经初始化中
spring应用上下文已启动...
org.geekbang.thinking.in.spring.bean.factory.DefaultUserFactory@3d36e4cd
spring上下文准备关闭
@PreDestroy：UserFactory销毁中...
DisposableBean#destroy：UserFactory销毁中...
DefaultUserFactory#destroyMethod：UserFactory销毁中...
spring上下文已关闭
```

思考：假设以上三种方式均在同一 Bean 中定义，那么这些方法的执行顺序是怎样？  

> 可见，类似初始方法：@PreDestroy -> DisposableBean接口实现方法 -> 自定义销毁方法



## 垃圾回收Bean（GC）

1. 关闭 Spring 容器（应用上下文）
2. 执行 GC
3. Spring Bean 覆盖的 finalize() 方法被回调  

```java
package org.geekbang.thinking.in.spring.bean.factory;

import com.geekbang.ioc.overview.dependency.domain.User;
import org.springframework.beans.factory.DisposableBean;
import org.springframework.beans.factory.InitializingBean;
import org.springframework.beans.factory.SmartInitializingSingleton;

import javax.annotation.PostConstruct;
import javax.annotation.PreDestroy;

public class DefaultUserFactory implements UserFactory, InitializingBean, SmartInitializingSingleton, DisposableBean {

    @PostConstruct
    public void init(){
        System.out.println("@PostConstruct：UserFactory初始化中...");
    }

    /**
     * 自定义初始化方法
     */
    public void initUserFactory(){
        System.out.println("自定义初始化方法initUserFactory()：UserFactory初始化中...");
    }

    @Override
    public void afterPropertiesSet() throws Exception {
        System.out.println("InitializingBean#afterPropertiesSet：UserFactory初始化中...");
    }

    @Override
    public void afterSingletonsInstantiated() {
        System.out.println("SmartInitializingSingleton#afterSingletonsInstantiated: UserFactoryy已经初始化中");
    }

    @PreDestroy
    public void destory(){
        System.out.println("@PreDestroy：UserFactory销毁中...");
    }

    @Override
    public void destroy() throws Exception {
        System.out.println("DisposableBean#destroy：UserFactory销毁中...");
    }

    public void destroyMethod(){
        System.out.println("DefaultUserFactory#destroyMethod：UserFactory销毁中...");
    }

    @Override
    protected void finalize() throws Throwable {
        System.out.println("当前对象DefaultUserFactory正在回收中.......");
    }
}

```

```java
package org.geekbang.thinking.in.spring.bean.definition;

import org.springframework.context.annotation.AnnotationConfigApplicationContext;

/**
 * Bean 垃圾回收示例
 */
public class BeanGarbageCollectionDemo {

    public static void main(String[] args) throws InterruptedException {
        AnnotationConfigApplicationContext context = new AnnotationConfigApplicationContext();
        context.register(InitializationDemo.class);
        context.refresh();
        context.close();
        Thread.sleep(5000L);
        //强制gc，确保finalize()方法能被执行
        System.gc();
        Thread.sleep(5000L);
    }

}

```

> 注意finalize()方法只是有概率执行，不确保一定执行



## 面试题

### 如何注册一个 Spring Bean？  

答：通过 BeanDefinition 和外部单体对象来注册  



### 什么是 Spring BeanDefinition？  

答：回顾“定义 Spring Bean” 和 “BeanDefinition 元信息”



### Spring 容器是怎样管理注册 Bean  

答：答案将在后续专题章节详细讨论，如：IoC 配置元信息读取和解析、依赖
查找和注入以及 Bean 生命周期等。  





学习：深入到规范，如JSR规范、Servlet规范等
