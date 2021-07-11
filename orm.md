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

