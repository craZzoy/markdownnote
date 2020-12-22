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
- Bean行为配置元素，如作用域、自动绑定的方式、声明周期的回调等
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

BeanDefinition创建方式

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



学习：深入到规范，如JSR规范、Servlet规范等