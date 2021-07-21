# ORM框架

Object Relational Mapping

- Mybatis
- Hibernate
- Spring JDBC



关于java.io.Serializable接口

- 用于存储的类
- 用于传输的类，如分布式调用中

为什么要系列化

- 提供一种简单又可扩展的对象保存恢复机制。即要保存状态
- 对于远程调用，能方便对对象进行编码和解码，就像实现对象直接传输。
- 可以将对象持久化到介质中，就像实现对象直接存储。
- 允许对象自定义外部存储的格式。

为什么domain中实体不实现java.io.Serializable也可以？

domain中实体一般都是无状态的。只是对应映射数据库数据，而并不是需要把整个类的信息持久化。





# Mybatis

基于版本：3.5.7

## 入门

### mybatis整体架构

![image-20210704180151045](D:\BaiduNetdiskDownload\markdown笔记\orm.assets\image-20210704180151045.png)



## 基础支持层

### 解析器模块

XML解析

- DOM
- SAX
- StAX（jdk6开始提供）

#### XPATH

![image-20210708222020269](D:\BaiduNetdiskDownload\markdown笔记\orm.assets\image-20210708222020269.png)

#### 反射工具箱

- org.apache.ibatis.reflection.Reflector
- org.apache.ibatis.reflection.ReflectorFactory
  - org.apache.ibatis.reflection.DefaultReflectorFactory（mybatis默认提供实现，可在config文件中自定义配置）



##### TypeParameterResolver

![image-20210710210239659](D:\BaiduNetdiskDownload\markdown笔记\orm.assets\image-20210710210239659.png)

java.lang.reflect.Type类型

- java.lang.Class：原始类型。表示JVM中的一个类或者接口，数组也被映射为Class对象，所有元素相同且维数相同的数组都被映射为t同一个Class对象。

  - Object.getClass()
  - Class.forName()

  ```java
  package com.mybatis.reflector.type;
  
  
  public class ClassTypeTest {
  
      public static void main(String[] args) {
          String[] strs = new String[]{"key","name"};
          Integer[] ints = new Integer[]{1,2,3};
          Integer[] ints1 = new Integer[]{1,2,3};
          System.out.println(strs.getClass());
          System.out.println(ints.getClass());
          System.out.println(ints1.getClass());
      }
  
  }
  
  ```

  ```txt
  class [Ljava.lang.String;
  class [Ljava.lang.Integer;
  class [Ljava.lang.Integer;
  ```

- java.lang.reflect.GenericArrayType：数组类型且组成元素是java.lang.reflect.ParameterizedType或者java.lang.reflect.TypeVariable。例如List<String>[]或者T[]。

  - java.lang.reflect.GenericArrayType#getGenericComponentType

- java.lang.reflect.ParameterizedType：参数化类型。例如List<String>，Map<Integer, String>，Service<User>这种带泛型的参数

  ```java
  package java.lang.reflect;
  
  
  /**
   * ParameterizedType represents a parameterized type such as
   * Collection&lt;String&gt;.
   *
   * <p>A parameterized type is created the first time it is needed by a
   * reflective method, as specified in this package. When a
   * parameterized type p is created, the generic type declaration that
   * p instantiates is resolved, and all type arguments of p are created
   * recursively. See {@link java.lang.reflect.TypeVariable
   * TypeVariable} for details on the creation process for type
   * variables. Repeated creation of a parameterized type has no effect.
   *
   * <p>Instances of classes that implement this interface must implement
   * an equals() method that equates any two instances that share the
   * same generic type declaration and have equal type parameters.
   *
   * @since 1.5
   */
  public interface ParameterizedType extends Type {
      /**
       * 返回参数化类型的类型变量或是实际类型列表。如Map<String,Integer>实际类型列表是String和Integer
       * Returns an array of {@code Type} objects representing the actual type
       * arguments to this type.
       *
       * <p>Note that in some cases, the returned array be empty. This can occur
       * if this type represents a non-parameterized type nested within
       * a parameterized type.
       *
       * @return an array of {@code Type} objects representing the actual type
       *     arguments to this type
       * @throws TypeNotPresentException if any of the
       *     actual type arguments refers to a non-existent type declaration
       * @throws MalformedParameterizedTypeException if any of the
       *     actual type parameters refer to a parameterized type that cannot
       *     be instantiated for any reason
       * @since 1.5
       */
      Type[] getActualTypeArguments();
  
      /**
       * 返回原始类型，如List<String>返回List
       * Returns the {@code Type} object representing the class or interface
       * that declared this type.
       * @since 1.5
       */
      Type getRawType();
  
      /**
       * 返回类型所属的类型，一般用在内部类
       * Returns a {@code Type} object representing the type that this type
       * is a member of.  For example, if this type is {@code O<T>.I<S>},
       * return a representation of {@code O<T>}.
       *
       * <p>If this type is a top-level type, {@code null} is returned.
       *
       * @return a {@code Type} object representing the type that
       *     this type is a member of. If this type is a top-level type,
       *     {@code null} is returned
       * @throws TypeNotPresentException if the owner type
       *     refers to a non-existent type declaration
       * @throws MalformedParameterizedTypeException if the owner type
       *     refers to a parameterized type that cannot be instantiated
       *     for any reason
       * @since 1.5
       */
      Type getOwnerType();
  }
  
  ```

- java.lang.reflect.TypeVariable：类型变量，它用来反映在JVM编译该泛型前的信息。例如List<T>中T就是类型变量

  - java.lang.reflect.TypeVariable#getBounds：获取类型变量的上边界，默认是Object。如class Test<K extends Person>中K的上界就是Person
  - java.lang.reflect.TypeVariable#getGenericDeclaration：获取声明该类型变量的原始变量。如class Test<K extends Person>中的原始类型就是Test
  - java.lang.reflect.TypeVariable#getName：获取在源码中定义的名字。如上例中的K

- java.lang.reflect.WildcardType：通配符类型。例如? extends Number和? super Integer

  - java.lang.reflect.WildcardType#getUpperBounds
  - java.lang.reflect.WildcardType#getLowerBounds

```java
package com.mybatis.reflector.type;

import org.apache.ibatis.reflection.TypeParameterResolver;
import sun.reflect.generics.reflectiveObjects.ParameterizedTypeImpl;

import java.lang.reflect.Field;
import java.lang.reflect.ParameterizedType;
import java.lang.reflect.Type;

public class TypeTest {

    SubClassA<Long> as = new SubClassA<>();

    public static void main(String[] args) throws Exception {

        Field f = ClassA.class.getDeclaredField("map");
        System.out.println(f.getGenericType()); // java.util.Map<K, V>
        System.out.println(f.getGenericType() instanceof ParameterizedType); // true

        Type type = TypeParameterResolver.resolveFieldType(f,
                ParameterizedTypeImpl.make(SubClassA.class, new Type[]{Long.class}, TypeTest.class));
        Type type1 = TypeParameterResolver.resolveFieldType(f, TypeTest.class.getDeclaredField("as").getGenericType());
        System.out.println(type.getClass()); //class org.apache.ibatis.reflection.TypeParameterResolver$ParameterizedTypeImpl
        System.out.println(type1.getClass()); //class org.apache.ibatis.reflection.TypeParameterResolver$ParameterizedTypeImpl

        ParameterizedType p = (ParameterizedType) type;
        System.out.println(p.getOwnerType()); // null
        System.out.println(p.getRawType()); // interface java.util.Map
        for (Type t : p.getActualTypeArguments()){
            System.out.println(t);
            //class java.lang.Long
            //class java.lang.Long
        }

    }

}
```

源码学习参考：

org.apache.ibatis.reflection.TypeParameterResolver#resolveFieldType



##### ObjectFactory

用于实例化对象

org.apache.ibatis.reflection.factory.ObjectFactory

- org.apache.ibatis.reflection.factory.DefaultObjectFactory



##### Property工具集

- org.apache.ibatis.reflection.property.PropertyTokenizer：解析属性表达式，如解析orders[0].items[0].name

- org.apache.ibatis.reflection.property.PropertyNamer：方法名到属性名的转换

  ```java
  /**
   *    Copyright 2009-2019 the original author or authors.
   *
   *    Licensed under the Apache License, Version 2.0 (the "License");
   *    you may not use this file except in compliance with the License.
   *    You may obtain a copy of the License at
   *
   *       http://www.apache.org/licenses/LICENSE-2.0
   *
   *    Unless required by applicable law or agreed to in writing, software
   *    distributed under the License is distributed on an "AS IS" BASIS,
   *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   *    See the License for the specific language governing permissions and
   *    limitations under the License.
   */
  package org.apache.ibatis.reflection.property;
  
  import java.util.Locale;
  
  import org.apache.ibatis.reflection.ReflectionException;
  
  /**
   * @author Clinton Begin
   */
  public final class PropertyNamer {
  
    private PropertyNamer() {
      // Prevent Instantiation of Static Class
    }
  
    public static String methodToProperty(String name) {
      if (name.startsWith("is")) {
        name = name.substring(2);
      } else if (name.startsWith("get") || name.startsWith("set")) {
        name = name.substring(3);
      } else {
        throw new ReflectionException("Error parsing property name '" + name + "'.  Didn't start with 'is', 'get' or 'set'.");
      }
  
      if (name.length() == 1 || (name.length() > 1 && !Character.isUpperCase(name.charAt(1)))) {
        name = name.substring(0, 1).toLowerCase(Locale.ENGLISH) + name.substring(1);
      }
  
      return name;
    }
  
    public static boolean isProperty(String name) {
      return isGetter(name) || isSetter(name);
    }
  
    public static boolean isGetter(String name) {
      return (name.startsWith("get") && name.length() > 3) || (name.startsWith("is") && name.length() > 2);
    }
  
    public static boolean isSetter(String name) {
      return name.startsWith("set") && name.length() > 3;
    }
  
  }
  
  ```

- org.apache.ibatis.reflection.property.PropertyCopier：属性值拷贝

  ```java
  /**
   *    Copyright 2009-2019 the original author or authors.
   *
   *    Licensed under the Apache License, Version 2.0 (the "License");
   *    you may not use this file except in compliance with the License.
   *    You may obtain a copy of the License at
   *
   *       http://www.apache.org/licenses/LICENSE-2.0
   *
   *    Unless required by applicable law or agreed to in writing, software
   *    distributed under the License is distributed on an "AS IS" BASIS,
   *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   *    See the License for the specific language governing permissions and
   *    limitations under the License.
   */
  package org.apache.ibatis.reflection.property;
  
  import java.lang.reflect.Field;
  
  import org.apache.ibatis.reflection.Reflector;
  
  /**
   * @author Clinton Begin
   */
  public final class PropertyCopier {
  
    private PropertyCopier() {
      // Prevent Instantiation of Static Class
    }
  
    public static void copyBeanProperties(Class<?> type, Object sourceBean, Object destinationBean) {
      Class<?> parent = type;
      while (parent != null) {
        final Field[] fields = parent.getDeclaredFields();
        for (Field field : fields) {
          try {
            try {
              field.set(destinationBean, field.get(sourceBean));
            } catch (IllegalAccessException e) {
              if (Reflector.canControlMemberAccessible()) {
                field.setAccessible(true);
                field.set(destinationBean, field.get(sourceBean));
              } else {
                throw e;
              }
            }
          } catch (Exception e) {
            // Nothing useful to do, will only fail on final fields, which will be ignored.
          }
        }
        parent = parent.getSuperclass();
      }
    }
  
  }
  
  ```



##### MetaClass

org.apache.ibatis.reflection.MetaClass结合org.apache.ibatis.reflection.ReflectorFactory和org.apache.ibatis.reflection.Reflector，可用于解析复杂的表达式



##### ObjectWrapper

MetaClass是对类级别的元信息的封装和处理。ObjectWrapper接口是对对象的包装，抽象了对象的属性信息，它定义了一系列查询对象属性信息的方法，以及更新属性的方法。

org.apache.ibatis.reflection.wrapper.ObjectWrapper

```java
/**
 *    Copyright 2009-2019 the original author or authors.
 *
 *    Licensed under the Apache License, Version 2.0 (the "License");
 *    you may not use this file except in compliance with the License.
 *    You may obtain a copy of the License at
 *
 *       http://www.apache.org/licenses/LICENSE-2.0
 *
 *    Unless required by applicable law or agreed to in writing, software
 *    distributed under the License is distributed on an "AS IS" BASIS,
 *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *    See the License for the specific language governing permissions and
 *    limitations under the License.
 */
package org.apache.ibatis.reflection.wrapper;

import java.util.List;

import org.apache.ibatis.reflection.MetaObject;
import org.apache.ibatis.reflection.factory.ObjectFactory;
import org.apache.ibatis.reflection.property.PropertyTokenizer;

/**
 * @author Clinton Begin
 */
public interface ObjectWrapper {

  Object get(PropertyTokenizer prop);

  void set(PropertyTokenizer prop, Object value);

  String findProperty(String name, boolean useCamelCaseMapping);

  String[] getGetterNames();

  String[] getSetterNames();

  Class<?> getSetterType(String name);

  Class<?> getGetterType(String name);

  boolean hasSetter(String name);

  boolean hasGetter(String name);

  MetaObject instantiatePropertyValue(String name, PropertyTokenizer prop, ObjectFactory objectFactory);

  boolean isCollection();

  void add(Object element);

  <E> void addAll(List<E> element);

}

```

- org.apache.ibatis.reflection.wrapper.BaseWrapper
  - org.apache.ibatis.reflection.wrapper.BeanWrapper
  - org.apache.ibatis.reflection.wrapper.MapWrapper
- org.apache.ibatis.reflection.wrapper.CollectionWrapper



##### MetaObject

实例对象的元数据，用于对属性表达式的解析



#### 类型转换

用于javaType和jdbcType类型之间的转换

![image-20210711174800382](D:\BaiduNetdiskDownload\markdown笔记\orm.assets\image-20210711174800382.png)

核心API

![image-20210711175722369](D:\BaiduNetdiskDownload\markdown笔记\orm.assets\image-20210711175722369.png)

- org.apache.ibatis.type.TypeHandler：定义设置参数（javaType转jdbcType）方法，获取结果方法（jdbcType转javaType）

  ```java
  /**
   *    Copyright 2009-2020 the original author or authors.
   *
   *    Licensed under the Apache License, Version 2.0 (the "License");
   *    you may not use this file except in compliance with the License.
   *    You may obtain a copy of the License at
   *
   *       http://www.apache.org/licenses/LICENSE-2.0
   *
   *    Unless required by applicable law or agreed to in writing, software
   *    distributed under the License is distributed on an "AS IS" BASIS,
   *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   *    See the License for the specific language governing permissions and
   *    limitations under the License.
   */
  package org.apache.ibatis.type;
  
  import java.sql.CallableStatement;
  import java.sql.PreparedStatement;
  import java.sql.ResultSet;
  import java.sql.SQLException;
  
  /**
   * @author Clinton Begin
   */
  public interface TypeHandler<T> {
  
    void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;
  
    /**
     * Gets the result.
     *
     * @param rs
     *          the rs
     * @param columnName
     *          Colunm name, when configuration <code>useColumnLabel</code> is <code>false</code>
     * @return the result
     * @throws SQLException
     *           the SQL exception
     */
    T getResult(ResultSet rs, String columnName) throws SQLException;
  
    T getResult(ResultSet rs, int columnIndex) throws SQLException;
  
    T getResult(CallableStatement cs, int columnIndex) throws SQLException;
  
  }
  
  ```

  - org.apache.ibatis.type.BaseTypeHandler：提供默认实现

    ```java
    /**
     *    Copyright 2009-2020 the original author or authors.
     *
     *    Licensed under the Apache License, Version 2.0 (the "License");
     *    you may not use this file except in compliance with the License.
     *    You may obtain a copy of the License at
     *
     *       http://www.apache.org/licenses/LICENSE-2.0
     *
     *    Unless required by applicable law or agreed to in writing, software
     *    distributed under the License is distributed on an "AS IS" BASIS,
     *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
     *    See the License for the specific language governing permissions and
     *    limitations under the License.
     */
    package org.apache.ibatis.type;
    
    import java.sql.CallableStatement;
    import java.sql.PreparedStatement;
    import java.sql.ResultSet;
    import java.sql.SQLException;
    
    import org.apache.ibatis.executor.result.ResultMapException;
    import org.apache.ibatis.session.Configuration;
    
    /**
     * The base {@link TypeHandler} for references a generic type.
     * <p>
     * Important: Since 3.5.0, This class never call the {@link ResultSet#wasNull()} and
     * {@link CallableStatement#wasNull()} method for handling the SQL {@code NULL} value.
     * In other words, {@code null} value handling should be performed on subclass.
     * </p>
     *
     * @author Clinton Begin
     * @author Simone Tripodi
     * @author Kzuki Shimizu
     */
    public abstract class BaseTypeHandler<T> extends TypeReference<T> implements TypeHandler<T> {
    
      /**
       * @deprecated Since 3.5.0 - See https://github.com/mybatis/mybatis-3/issues/1203. This field will remove future.
       */
      @Deprecated
      protected Configuration configuration;
    
      /**
       * Sets the configuration.
       *
       * @param c
       *          the new configuration
       * @deprecated Since 3.5.0 - See https://github.com/mybatis/mybatis-3/issues/1203. This property will remove future.
       */
      @Deprecated
      public void setConfiguration(Configuration c) {
        this.configuration = c;
      }
    
      @Override
      public void setParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException {
        if (parameter == null) {
          if (jdbcType == null) {
            throw new TypeException("JDBC requires that the JdbcType must be specified for all nullable parameters.");
          }
          try {
            ps.setNull(i, jdbcType.TYPE_CODE);
          } catch (SQLException e) {
            throw new TypeException("Error setting null for parameter #" + i + " with JdbcType " + jdbcType + " . "
                  + "Try setting a different JdbcType for this parameter or a different jdbcTypeForNull configuration property. "
                  + "Cause: " + e, e);
          }
        } else {
          try {
            setNonNullParameter(ps, i, parameter, jdbcType);
          } catch (Exception e) {
            throw new TypeException("Error setting non null for parameter #" + i + " with JdbcType " + jdbcType + " . "
                  + "Try setting a different JdbcType for this parameter or a different configuration property. "
                  + "Cause: " + e, e);
          }
        }
      }
    
      @Override
      public T getResult(ResultSet rs, String columnName) throws SQLException {
        try {
          return getNullableResult(rs, columnName);
        } catch (Exception e) {
          throw new ResultMapException("Error attempting to get column '" + columnName + "' from result set.  Cause: " + e, e);
        }
      }
    
      @Override
      public T getResult(ResultSet rs, int columnIndex) throws SQLException {
        try {
          return getNullableResult(rs, columnIndex);
        } catch (Exception e) {
          throw new ResultMapException("Error attempting to get column #" + columnIndex + " from result set.  Cause: " + e, e);
        }
      }
    
      @Override
      public T getResult(CallableStatement cs, int columnIndex) throws SQLException {
        try {
          return getNullableResult(cs, columnIndex);
        } catch (Exception e) {
          throw new ResultMapException("Error attempting to get column #" + columnIndex + " from callable statement.  Cause: " + e, e);
        }
      }
    
      public abstract void setNonNullParameter(PreparedStatement ps, int i, T parameter, JdbcType jdbcType) throws SQLException;
    
      /**
       * Gets the nullable result.
       *
       * @param rs
       *          the rs
       * @param columnName
       *          Colunm name, when configuration <code>useColumnLabel</code> is <code>false</code>
       * @return the nullable result
       * @throws SQLException
       *           the SQL exception
       */
      public abstract T getNullableResult(ResultSet rs, String columnName) throws SQLException;
    
      public abstract T getNullableResult(ResultSet rs, int columnIndex) throws SQLException;
    
      public abstract T getNullableResult(CallableStatement cs, int columnIndex) throws SQLException;
    
    }
    
    ```

    - org.apache.ibatis.type.IntegerTypeHandler：Integer类型转换实现

      ```java
      /**
       *    Copyright 2009-2018 the original author or authors.
       *
       *    Licensed under the Apache License, Version 2.0 (the "License");
       *    you may not use this file except in compliance with the License.
       *    You may obtain a copy of the License at
       *
       *       http://www.apache.org/licenses/LICENSE-2.0
       *
       *    Unless required by applicable law or agreed to in writing, software
       *    distributed under the License is distributed on an "AS IS" BASIS,
       *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
       *    See the License for the specific language governing permissions and
       *    limitations under the License.
       */
      package org.apache.ibatis.type;
      
      import java.sql.CallableStatement;
      import java.sql.PreparedStatement;
      import java.sql.ResultSet;
      import java.sql.SQLException;
      
      /**
       * @author Clinton Begin
       */
      public class IntegerTypeHandler extends BaseTypeHandler<Integer> {
      
        @Override
        public void setNonNullParameter(PreparedStatement ps, int i, Integer parameter, JdbcType jdbcType)
            throws SQLException {
          ps.setInt(i, parameter);
        }
      
        @Override
        public Integer getNullableResult(ResultSet rs, String columnName)
            throws SQLException {
          int result = rs.getInt(columnName);
          return result == 0 && rs.wasNull() ? null : result;
        }
      
        @Override
        public Integer getNullableResult(ResultSet rs, int columnIndex)
            throws SQLException {
          int result = rs.getInt(columnIndex);
          return result == 0 && rs.wasNull() ? null : result;
        }
      
        @Override
        public Integer getNullableResult(CallableStatement cs, int columnIndex)
            throws SQLException {
          int result = cs.getInt(columnIndex);
          return result == 0 && cs.wasNull() ? null : result;
        }
      }
      
      ```

    - 。。。



##### TypeHandlerRegistry

注册管理对应jdbcType和javaType对应的TypeHandler



##### TypeAliasRegistry

维护类的别名，构造方法提供了一些默认的类的别名：

```java
public class TypeAliasRegistry {

  private final Map<String, Class<?>> typeAliases = new HashMap<>();

  public TypeAliasRegistry() {
    registerAlias("string", String.class);

    registerAlias("byte", Byte.class);
    registerAlias("long", Long.class);
    registerAlias("short", Short.class);
    registerAlias("int", Integer.class);
    registerAlias("integer", Integer.class);
    registerAlias("double", Double.class);
    registerAlias("float", Float.class);
    registerAlias("boolean", Boolean.class);
      ...
  }
}
...
```



### 日志模块

常用日志框架：

- Log4j
- Log4j2
- Apache Commons Log
- java.util.logging
- slf4j



mybatis提供的统一日志适配接口：

org.apache.ibatis.logging.Log

```java
public interface Log {

  boolean isDebugEnabled();

  boolean isTraceEnabled();

  void error(String s, Throwable e);

  void error(String s);

  void debug(String s);

  void trace(String s);

  void warn(String s);

}
```

适配各种日志框架：

![哪个台](D:\BaiduNetdiskDownload\markdown笔记\orm.assets\image-20210711230258746.png)

org.apache.ibatis.logging.LogFactory的静态代码块会尝试初始化使用哪种适配的日志框架

```java
public final class LogFactory {

  /**
   * Marker to be used by logging implementations that support markers.
   */
  public static final String MARKER = "MYBATIS";

  private static Constructor<? extends Log> logConstructor;

  static {
    tryImplementation(LogFactory::useSlf4jLogging);
    tryImplementation(LogFactory::useCommonsLogging);
    tryImplementation(LogFactory::useLog4J2Logging);
    tryImplementation(LogFactory::useLog4JLogging);
    tryImplementation(LogFactory::useJdkLogging);
    tryImplementation(LogFactory::useNoLogging);
  }
    ...
}
```

```java
  private static void tryImplementation(Runnable runnable) {
    if (logConstructor == null) {
      try {
        runnable.run();
      } catch (Throwable t) {
        // ignore
      }
    }
  }
```

```java
  public static synchronized void useJdkLogging() {
    setImplementation(org.apache.ibatis.logging.jdk14.Jdk14LoggingImpl.class);
  }
```

```java
  private static void setImplementation(Class<? extends Log> implClass) {
    try {
      Constructor<? extends Log> candidate = implClass.getConstructor(String.class);
      Log log = candidate.newInstance(LogFactory.class.getName());
      if (log.isDebugEnabled()) {
        log.debug("Logging initialized using '" + implClass + "' adapter.");
      }
      logConstructor = candidate;
    } catch (Throwable t) {
      throw new LogException("Error setting Log implementation.  Cause: " + t, t);
    }
  }
```



#### 代理模式

jdk动态代理与cglib动态代理



#### jdbc调试

- org.apache.ibatis.logging.jdbc.BaseJdbcLogger
  - org.apache.ibatis.logging.jdbc.ConnectionLogger
  - org.apache.ibatis.logging.jdbc.PreparedStatementLogger
  - org.apache.ibatis.logging.jdbc.ResultSetLogger
  - org.apache.ibatis.logging.jdbc.StatementLogger

通过对核心的jdbc处理类进行代理，结合org.apache.ibatis.logging.Log打印调试信息，如输入参数、执行sql、返回结果等。如ConnectionLogger：

```java
package org.apache.ibatis.logging.jdbc;

import java.lang.reflect.InvocationHandler;
import java.lang.reflect.Method;
import java.lang.reflect.Proxy;
import java.sql.Connection;
import java.sql.PreparedStatement;
import java.sql.Statement;

import org.apache.ibatis.logging.Log;
import org.apache.ibatis.reflection.ExceptionUtil;

/**
 * Connection proxy to add logging.
 *
 * @author Clinton Begin
 * @author Eduardo Macarron
 *
 */
public final class ConnectionLogger extends BaseJdbcLogger implements InvocationHandler {

  private final Connection connection;

  private ConnectionLogger(Connection conn, Log statementLog, int queryStack) {
    super(statementLog, queryStack);
    this.connection = conn;
  }

  @Override
  public Object invoke(Object proxy, Method method, Object[] params)
      throws Throwable {
    try {
      if (Object.class.equals(method.getDeclaringClass())) {
        return method.invoke(this, params);
      }
      if ("prepareStatement".equals(method.getName()) || "prepareCall".equals(method.getName())) {
        if (isDebugEnabled()) {
          debug(" Preparing: " + removeExtraWhitespace((String) params[0]), true);
        }
        PreparedStatement stmt = (PreparedStatement) method.invoke(connection, params);
        stmt = PreparedStatementLogger.newInstance(stmt, statementLog, queryStack);
        return stmt;
      } else if ("createStatement".equals(method.getName())) {
        Statement stmt = (Statement) method.invoke(connection, params);
        stmt = StatementLogger.newInstance(stmt, statementLog, queryStack);
        return stmt;
      } else {
        return method.invoke(connection, params);
      }
    } catch (Throwable t) {
      throw ExceptionUtil.unwrapThrowable(t);
    }
  }

  /**
   * Creates a logging version of a connection.
   *
   * @param conn
   *          the original connection
   * @param statementLog
   *          the statement log
   * @param queryStack
   *          the query stack
   * @return the connection with logging
   */
  public static Connection newInstance(Connection conn, Log statementLog, int queryStack) {
    InvocationHandler handler = new ConnectionLogger(conn, statementLog, queryStack);
    ClassLoader cl = Connection.class.getClassLoader();
    return (Connection) Proxy.newProxyInstance(cl, new Class[]{Connection.class}, handler);
  }

  /**
   * return the wrapped connection.
   *
   * @return the connection
   */
  public Connection getConnection() {
    return connection;
  }

}

```

ConnectionLogger会被org.apache.ibatis.executor.BaseExecutor#getConnection方法调用：

```java
  protected Connection getConnection(Log statementLog) throws SQLException {
    Connection connection = transaction.getConnection();
    if (statementLog.isDebugEnabled()) {
      return ConnectionLogger.newInstance(connection, statementLog, queryStack);
    } else {
      return connection;
    }
  }
```



### 资源加载

#### ClassLoaderWrapper

ClassLoader的包装器，其中维护多个ClassLoader，查找资源时有序的调用其中的多个ClassLoader查找资源。如org.apache.ibatis.io.ClassLoaderWrapper#getResourceAsStream(java.lang.String, java.lang.ClassLoader[])

```java
  /**
   * Try to get a resource from a group of classloaders
   *
   * @param resource    - the resource to get
   * @param classLoader - the classloaders to examine
   * @return the resource or null
   */
  InputStream getResourceAsStream(String resource, ClassLoader[] classLoader) {
    for (ClassLoader cl : classLoader) {
      if (null != cl) {

        // try to find the resource as passed
        InputStream returnValue = cl.getResourceAsStream(resource);

        // now, some class loaders want this leading "/", so we'll add it and try again if we didn't find the resource
        if (null == returnValue) {
          returnValue = cl.getResourceAsStream("/" + resource);
        }

        if (null != returnValue) {
          return returnValue;
        }
      }
    }
    return null;
  }
```



#### ResolverUtil

查找指定包下的类

核心方法：org.apache.ibatis.io.ResolverUtil#find

```java
  /**
   * Scans for classes starting at the package provided and descending into subpackages.
   * Each class is offered up to the Test as it is discovered, and if the Test returns
   * true the class is retained.  Accumulated classes can be fetched by calling
   * {@link #getClasses()}.
   *
   * @param test
   *          an instance of {@link Test} that will be used to filter classes
   * @param packageName
   *          the name of the package from which to start scanning for classes, e.g. {@code net.sourceforge.stripes}
   * @return the resolver util
   */
  public ResolverUtil<T> find(Test test, String packageName) {
    String path = getPackagePath(packageName);

    try {
      List<String> children = VFS.getInstance().list(path);
      for (String child : children) {
        if (child.endsWith(".class")) {
          addIfMatching(test, child);
        }
      }
    } catch (IOException ioe) {
      log.error("Could not read package: " + packageName, ioe);
    }

    return this;
  }
```

条件接口：

- org.apache.ibatis.io.ResolverUtil.Test

  ```java
    /**
     * A simple interface that specifies how to test classes to determine if they
     * are to be included in the results produced by the ResolverUtil.
     */
    public interface Test {
  
      /**
       * Will be called repeatedly with candidate classes. Must return True if a class
       * is to be included in the results, false otherwise.
       *
       * @param type
       *          the type
       * @return true, if successful
       */
      boolean matches(Class<?> type);
    }
  ```

  - org.apache.ibatis.io.ResolverUtil.IsA：继承某个类或者实现某个接口

    ```java
      /**
       * A Test that checks to see if each class is assignable to the provided class. Note
       * that this test will match the parent type itself if it is presented for matching.
       */
      public static class IsA implements Test {
    
        /** The parent. */
        private Class<?> parent;
    
        /**
         * Constructs an IsA test using the supplied Class as the parent class/interface.
         *
         * @param parentType
         *          the parent type
         */
        public IsA(Class<?> parentType) {
          this.parent = parentType;
        }
    
        /** Returns true if type is assignable to the parent type supplied in the constructor. */
        @Override
        public boolean matches(Class<?> type) {
          return type != null && parent.isAssignableFrom(type);
        }
    
        @Override
        public String toString() {
          return "is assignable to " + parent.getSimpleName();
        }
      }
    ```

  - org.apache.ibatis.io.ResolverUtil.AnnotatedWith：包含指定注解

    ```java
      /**
       * A Test that checks to see if each class is annotated with a specific annotation. If it
       * is, then the test returns true, otherwise false.
       */
      public static class AnnotatedWith implements Test {
    
        /** The annotation. */
        private Class<? extends Annotation> annotation;
    
        /**
         * Constructs an AnnotatedWith test for the specified annotation type.
         *
         * @param annotation
         *          the annotation
         */
        public AnnotatedWith(Class<? extends Annotation> annotation) {
          this.annotation = annotation;
        }
    
        /** Returns true if the type is annotated with the class provided to the constructor. */
        @Override
        public boolean matches(Class<?> type) {
          return type != null && type.isAnnotationPresent(annotation);
        }
    
        @Override
        public String toString() {
          return "annotated with @" + annotation.getSimpleName();
        }
      }
    ```

    

#### VFS

virtual file system，用来查找指定路径下的资源

- org.apache.ibatis.io.VFS

  - 常用方法
    - org.apache.ibatis.io.VFS#list(java.net.URL, java.lang.String)：指定URL和包下的资源
    - org.apache.ibatis.io.VFS#list(java.lang.String)：指定URL下的资源

  - 实现类
    - org.apache.ibatis.io.DefaultVFS
    - org.apache.ibatis.io.JBoss6VFS

值得注意的是VFS单例是通过静态内部类实现的

```java
  /** Singleton instance holder. */
  private static class VFSHolder {
    static final VFS INSTANCE = createVFS();

    @SuppressWarnings("unchecked")
    static VFS createVFS() {
      // Try the user implementations first, then the built-ins
      List<Class<? extends VFS>> impls = new ArrayList<>();
      impls.addAll(USER_IMPLEMENTATIONS);
      impls.addAll(Arrays.asList((Class<? extends VFS>[]) IMPLEMENTATIONS));

      // Try each implementation class until a valid one is found
      VFS vfs = null;
      for (int i = 0; vfs == null || !vfs.isValid(); i++) {
        Class<? extends VFS> impl = impls.get(i);
        try {
          vfs = impl.getDeclaredConstructor().newInstance();
          if (!vfs.isValid() && log.isDebugEnabled()) {
            log.debug("VFS implementation " + impl.getName()
                + " is not valid in this environment.");
          }
        } catch (InstantiationException | IllegalAccessException | NoSuchMethodException | InvocationTargetException e) {
          log.error("Failed to instantiate " + impl, e);
          return null;
        }
      }

      if (log.isDebugEnabled()) {
        log.debug("Using VFS adapter " + vfs.getClass().getName());
      }

      return vfs;
    }
  }
```

org.apache.ibatis.io.VFS#getInstance:

```java

  /**
   * Get the singleton {@link VFS} instance. If no {@link VFS} implementation can be found for the current environment,
   * then this method returns null.
   *
   * @return single instance of VFS
   */
  public static VFS getInstance() {
    return VFSHolder.INSTANCE;
  }
```



### DataSource

未记录（需重点分析）



### Transation

- org.apache.ibatis.transaction.Transaction
  - org.apache.ibatis.transaction.jdbc.JdbcTransaction
  - org.apache.ibatis.transaction.managed.ManagedTransaction：生命周期是交给容器管理的

跟DataSource一样，使用了工厂方法模式：

- org.apache.ibatis.transaction.TransactionFactory
  - org.apache.ibatis.transaction.jdbc.JdbcTransactionFactory
  - org.apache.ibatis.transaction.managed.ManagedTransactionFactory



### Binding模块



#### ParamNameResolver

参数名称解析类org.apache.ibatis.reflection.ParamNameResolver



### 缓存模块

#### 装饰器模式

核心角色类图

![image-20210720212649275](D:\BaiduNetdiskDownload\markdown笔记\orm.assets\image-20210720212649275.png)

- Component：组件
- ConcreteComponent：具体组件实现类
- Decorator：装饰器，所有装饰器的父类，它也实现了Component接口，并在其中封装了Component对象
- ConcreteDecorator：具体装饰器

如java.io.BufferedInputStream就是java.io.InputStream的装饰对象



#### Cache接口

mybatis中cache接口及相关实现使用了装饰者模式的变种

组件接口：org.apache.ibatis.cache.Cache

组件实现类：org.apache.ibatis.cache.impl.PerpetualCache，此类通过HashMap提供了cache接口最简单的实现

```java
/**
 *    Copyright 2009-2019 the original author or authors.
 *
 *    Licensed under the Apache License, Version 2.0 (the "License");
 *    you may not use this file except in compliance with the License.
 *    You may obtain a copy of the License at
 *
 *       http://www.apache.org/licenses/LICENSE-2.0
 *
 *    Unless required by applicable law or agreed to in writing, software
 *    distributed under the License is distributed on an "AS IS" BASIS,
 *    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 *    See the License for the specific language governing permissions and
 *    limitations under the License.
 */
package org.apache.ibatis.cache.impl;

import java.util.HashMap;
import java.util.Map;

import org.apache.ibatis.cache.Cache;
import org.apache.ibatis.cache.CacheException;

/**
 * @author Clinton Begin
 */
public class PerpetualCache implements Cache {

  private final String id;

  private final Map<Object, Object> cache = new HashMap<>();

  public PerpetualCache(String id) {
    this.id = id;
  }

  @Override
  public String getId() {
    return id;
  }

  @Override
  public int getSize() {
    return cache.size();
  }

  @Override
  public void putObject(Object key, Object value) {
    cache.put(key, value);
  }

  @Override
  public Object getObject(Object key) {
    return cache.get(key);
  }

  @Override
  public Object removeObject(Object key) {
    return cache.remove(key);
  }

  @Override
  public void clear() {
    cache.clear();
  }

  @Override
  public boolean equals(Object o) {
    if (getId() == null) {
      throw new CacheException("Cache instances require an ID.");
    }
    if (this == o) {
      return true;
    }
    if (!(o instanceof Cache)) {
      return false;
    }

    Cache otherCache = (Cache) o;
    return getId().equals(otherCache.getId());
  }

  @Override
  public int hashCode() {
    if (getId() == null) {
      throw new CacheException("Cache instances require an ID.");
    }
    return getId().hashCode();
  }

}

```

装饰器：

![image-20210720212936017](D:\BaiduNetdiskDownload\markdown笔记\orm.assets\image-20210720212936017.png)

- org.apache.ibatis.cache.decorators.BlockingCache