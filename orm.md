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

![image-20210704180151045](orm.assets\image-20210704180151045.png)



## 基础支持层

### 解析器模块

XML解析

- DOM
- SAX
- StAX（jdk6开始提供）

#### XPATH

![image-20210708222020269](orm.assets\image-20210708222020269.png)

#### 反射工具箱

- org.apache.ibatis.reflection.Reflector
- org.apache.ibatis.reflection.ReflectorFactory
  - org.apache.ibatis.reflection.DefaultReflectorFactory（mybatis默认提供实现，可在config文件中自定义配置）



##### TypeParameterResolver

![image-20210710210239659](orm.assets\image-20210710210239659.png)

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
package org.apache.ibatis.reflection.factory;

import java.util.List;
import java.util.Properties;

/**
 * MyBatis uses an ObjectFactory to create all needed new Objects.
 *
 * @author Clinton Begin
 */
public interface ObjectFactory {

  /**
   * Sets configuration properties.
   * @param properties configuration properties
   */
  default void setProperties(Properties properties) {
    // NOP
  }

  /**
   * Creates a new object with default constructor.
   *
   * @param <T>
   *          the generic type
   * @param type
   *          Object type
   * @return the t
   */
  <T> T create(Class<T> type);

  /**
   * Creates a new object with the specified constructor and params.
   *
   * @param <T>
   *          the generic type
   * @param type
   *          Object type
   * @param constructorArgTypes
   *          Constructor argument types
   * @param constructorArgs
   *          Constructor argument values
   * @return the t
   */
  <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs);

  /**
   * Returns true if this object can have a set of other objects.
   * It's main purpose is to support non-java.util.Collection objects like Scala collections.
   *
   * @param <T>
   *          the generic type
   * @param type
   *          Object type
   * @return whether it is a collection or not
   * @since 3.1.0
   */
  <T> boolean isCollection(Class<T> type);

}

```

- org.apache.ibatis.reflection.factory.DefaultObjectFactory

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
  package org.apache.ibatis.reflection.factory;
  
  import java.io.Serializable;
  import java.lang.reflect.Constructor;
  import java.util.ArrayList;
  import java.util.Collection;
  import java.util.Collections;
  import java.util.HashMap;
  import java.util.HashSet;
  import java.util.List;
  import java.util.Map;
  import java.util.Optional;
  import java.util.Set;
  import java.util.SortedSet;
  import java.util.TreeSet;
  import java.util.stream.Collectors;
  
  import org.apache.ibatis.reflection.ReflectionException;
  import org.apache.ibatis.reflection.Reflector;
  
  /**
   * @author Clinton Begin
   */
  public class DefaultObjectFactory implements ObjectFactory, Serializable {
  
    private static final long serialVersionUID = -8855120656740914948L;
  
    @Override
    public <T> T create(Class<T> type) {
      return create(type, null, null);
    }
  
    @SuppressWarnings("unchecked")
    @Override
    public <T> T create(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
      Class<?> classToCreate = resolveInterface(type);
      // we know types are assignable
      return (T) instantiateClass(classToCreate, constructorArgTypes, constructorArgs);
    }
  
    private  <T> T instantiateClass(Class<T> type, List<Class<?>> constructorArgTypes, List<Object> constructorArgs) {
      try {
        Constructor<T> constructor;
        if (constructorArgTypes == null || constructorArgs == null) {
          constructor = type.getDeclaredConstructor();
          try {
            return constructor.newInstance();
          } catch (IllegalAccessException e) {
            if (Reflector.canControlMemberAccessible()) {
              constructor.setAccessible(true);
              return constructor.newInstance();
            } else {
              throw e;
            }
          }
        }
        constructor = type.getDeclaredConstructor(constructorArgTypes.toArray(new Class[0]));
        try {
          return constructor.newInstance(constructorArgs.toArray(new Object[0]));
        } catch (IllegalAccessException e) {
          if (Reflector.canControlMemberAccessible()) {
            constructor.setAccessible(true);
            return constructor.newInstance(constructorArgs.toArray(new Object[0]));
          } else {
            throw e;
          }
        }
      } catch (Exception e) {
        String argTypes = Optional.ofNullable(constructorArgTypes).orElseGet(Collections::emptyList)
            .stream().map(Class::getSimpleName).collect(Collectors.joining(","));
        String argValues = Optional.ofNullable(constructorArgs).orElseGet(Collections::emptyList)
            .stream().map(String::valueOf).collect(Collectors.joining(","));
        throw new ReflectionException("Error instantiating " + type + " with invalid types (" + argTypes + ") or values (" + argValues + "). Cause: " + e, e);
      }
    }
  
    protected Class<?> resolveInterface(Class<?> type) {
      Class<?> classToCreate;
      if (type == List.class || type == Collection.class || type == Iterable.class) {
        classToCreate = ArrayList.class;
      } else if (type == Map.class) {
        classToCreate = HashMap.class;
      } else if (type == SortedSet.class) { // issue #510 Collections Support
        classToCreate = TreeSet.class;
      } else if (type == Set.class) {
        classToCreate = HashSet.class;
      } else {
        classToCreate = type;
      }
      return classToCreate;
    }
  
    @Override
    public <T> boolean isCollection(Class<T> type) {
      return Collection.class.isAssignableFrom(type);
    }
  
  }
  
  ```

  



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

![image-20210711174800382](orm.assets\image-20210711174800382.png)

核心API

![image-20210711175722369](orm.assets\image-20210711175722369.png)

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

核心接口：org.apache.ibatis.type.TypeHandler

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

- setParameter：将java类型参数加入java.sql.PreparedStatement参数中
- getResult：获取java类型的结果

如：org.apache.ibatis.type.IntegerTypeHandler实现

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

org.apache.ibatis.type.TypeHandlerRegistry中主要维护了JdbcType、JavaType与TypeHandler的映射集合

```java
 
  private final Map<JdbcType, TypeHandler<?>>  jdbcTypeHandlerMap = new EnumMap<>(JdbcType.class);
  private final Map<Type, Map<JdbcType, TypeHandler<?>>> typeHandlerMap = new ConcurrentHashMap<>();
  private final TypeHandler<Object> unknownTypeHandler;
  private final Map<Class<?>, TypeHandler<?>> allTypeHandlersMap = new HashMap<>();
```

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

构造方法中有初始化：

```java
  public TypeHandlerRegistry(Configuration configuration) {
    this.unknownTypeHandler = new UnknownTypeHandler(configuration);

    register(Boolean.class, new BooleanTypeHandler());
    register(boolean.class, new BooleanTypeHandler());
    register(JdbcType.BOOLEAN, new BooleanTypeHandler());
    register(JdbcType.BIT, new BooleanTypeHandler());

    register(Byte.class, new ByteTypeHandler());
    register(byte.class, new ByteTypeHandler());
    register(JdbcType.TINYINT, new ByteTypeHandler());

    register(Short.class, new ShortTypeHandler());
    register(short.class, new ShortTypeHandler());
    register(JdbcType.SMALLINT, new ShortTypeHandler());
      ...
  }
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

![哪个台](orm.assets\image-20210711230258746.png)

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




#### Resources

org.apache.ibatis.io.Resources

提供一些解析资源的方法：

![image-20210721112401299](orm.assets\image-20210721112401299.png)



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

基础接口：javax.sql.DataSource

```java
package javax.sql;

import java.sql.Connection;
import java.sql.SQLException;
import java.sql.Wrapper;

/**
 * <p>A factory for connections to the physical data source that this
 * {@code DataSource} object represents.  An alternative to the
 * {@code DriverManager} facility, a {@code DataSource} object
 * is the preferred means of getting a connection. An object that implements
 * the {@code DataSource} interface will typically be
 * registered with a naming service based on the
 * Java&trade; Naming and Directory (JNDI) API.
 * <P>
 * The {@code DataSource} interface is implemented by a driver vendor.
 * There are three types of implementations:
 * <OL>
 *   <LI>Basic implementation -- produces a standard {@code Connection}
 *       object
 *   <LI>Connection pooling implementation -- produces a {@code Connection}
 *       object that will automatically participate in connection pooling.  This
 *       implementation works with a middle-tier connection pooling manager.
 *   <LI>Distributed transaction implementation -- produces a
 *       {@code Connection} object that may be used for distributed
 *       transactions and almost always participates in connection pooling.
 *       This implementation works with a middle-tier
 *       transaction manager and almost always with a connection
 *       pooling manager.
 * </OL>
 * <P>
 * A {@code DataSource} object has properties that can be modified
 * when necessary.  For example, if the data source is moved to a different
 * server, the property for the server can be changed.  The benefit is that
 * because the data source's properties can be changed, any code accessing
 * that data source does not need to be changed.
 * <P>
 * A driver that is accessed via a {@code DataSource} object does not
 * register itself with the {@code DriverManager}.  Rather, a
 * {@code DataSource} object is retrieved though a lookup operation
 * and then used to create a {@code Connection} object.  With a basic
 * implementation, the connection obtained through a {@code DataSource}
 * object is identical to a connection obtained through the
 * {@code DriverManager} facility.
 * <p>
 * An implementation of {@code DataSource} must include a public no-arg
 * constructor.
 *
 * @since 1.4
 */

public interface DataSource  extends CommonDataSource, Wrapper {

  /**
   * <p>Attempts to establish a connection with the data source that
   * this {@code DataSource} object represents.
   *
   * @return  a connection to the data source
   * @exception SQLException if a database access error occurs
   * @throws java.sql.SQLTimeoutException  when the driver has determined that the
   * timeout value specified by the {@code setLoginTimeout} method
   * has been exceeded and has at least tried to cancel the
   * current database connection attempt
   */
  Connection getConnection() throws SQLException;

  /**
   * <p>Attempts to establish a connection with the data source that
   * this {@code DataSource} object represents.
   *
   * @param username the database user on whose behalf the connection is
   *  being made
   * @param password the user's password
   * @return  a connection to the data source
   * @exception SQLException if a database access error occurs
   * @throws java.sql.SQLTimeoutException  when the driver has determined that the
   * timeout value specified by the {@code setLoginTimeout} method
   * has been exceeded and has at least tried to cancel the
   * current database connection attempt
   * @since 1.4
   */
  Connection getConnection(String username, String password)
    throws SQLException;
}

```

mybatis提供实现：

- org.apache.ibatis.datasource.unpooled.UnpooledDataSource

  主要属性：

  ```java
    private ClassLoader driverClassLoader;
    private Properties driverProperties;
    //注册的驱动
    private static Map<String, Driver> registeredDrivers = new ConcurrentHashMap<>();
  
    private String driver;
    private String url;
    private String username;
    private String password;
  
    private Boolean autoCommit;
    private Integer defaultTransactionIsolationLevel;
    private Integer defaultNetworkTimeout;
  ```

  初始化：

  ```java
    static {
      Enumeration<Driver> drivers = DriverManager.getDrivers();
      while (drivers.hasMoreElements()) {
        Driver driver = drivers.nextElement();
        registeredDrivers.put(driver.getClass().getName(), driver);
      }
    }
  ```

  > 初始化时会把DriverManager中已注册的驱动复制一份到registeredDrivers中

  在JDBC中，创建数据库连接前需要向DriverManager中注册驱动。如mysql的Driver实现类com.mysql.cj.jdbc.Driver中：

  ```
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

  UnpooledDataSource实现了DataSource的connect方法，其中调用的核心方法是：

  ```java
    private Connection doGetConnection(Properties properties) throws SQLException {
      initializeDriver();
      Connection connection = DriverManager.getConnection(url, properties);
      configureConnection(connection);
      return connection;
    }
  ```

  org.apache.ibatis.datasource.unpooled.UnpooledDataSource#initializeDriver：初始化驱动

  ```java
    private synchronized void initializeDriver() throws SQLException {
      if (!registeredDrivers.containsKey(driver)) {
        Class<?> driverType;
        try {
          //获取驱动的Class对象
          if (driverClassLoader != null) {
            driverType = Class.forName(driver, true, driverClassLoader);
          } else {
            driverType = Resources.classForName(driver);
          }
          // DriverManager requires the driver to be loaded via the system ClassLoader.
          // http://www.kfu.com/~nsayer/Java/dyn-jdbc.html
          //初始化驱动
          Driver driverInstance = (Driver) driverType.getDeclaredConstructor().newInstance();
          //注册驱动 DriverProxy是UnpooledDataSource的内部类
          DriverManager.registerDriver(new DriverProxy(driverInstance));
          registeredDrivers.put(driver, driverInstance);
        } catch (Exception e) {
          throw new SQLException("Error setting driver on UnpooledDataSource. Cause: " + e);
        }
      }
    }
  ```

  org.apache.ibatis.datasource.unpooled.UnpooledDataSource#configureConnection：配置连接属性

  ```java
    private void configureConnection(Connection conn) throws SQLException {
      if (defaultNetworkTimeout != null) {
        conn.setNetworkTimeout(Executors.newSingleThreadExecutor(), defaultNetworkTimeout);
      }
      if (autoCommit != null && autoCommit != conn.getAutoCommit()) {
        conn.setAutoCommit(autoCommit);
      }
      if (defaultTransactionIsolationLevel != null) {
        conn.setTransactionIsolation(defaultTransactionIsolationLevel);
      }
    }
  ```

  setNetworkTimeout是设置网络连接时长，即session的socket超时时间，具体代码实现：com.mysql.cj.jdbc.ConnectionImpl.NetworkTimeoutSetter

  ```java
      private static class NetworkTimeoutSetter implements Runnable {
          private final WeakReference<JdbcConnection> connRef;
          private final int milliseconds;
  
          public NetworkTimeoutSetter(JdbcConnection conn, int milliseconds) {
              this.connRef = new WeakReference<>(conn);
              this.milliseconds = milliseconds;
          }
  
          @Override
          public void run() {
              JdbcConnection conn = this.connRef.get();
              if (conn != null) {
                  synchronized (conn.getConnectionMutex()) {
                      ((NativeSession) conn.getSession()).setSocketTimeout(this.milliseconds);
                  }
              }
          }
      }
  ```

- org.apache.ibatis.datasource.pooled.PooledDataSource

  总体类图：

  ![image-20210721225817084](D:\BaiduNetdiskDownload\markdown笔记\image-20210721225817084.png)

  首先查看获取连接方法：org.apache.ibatis.datasource.pooled.PooledDataSource#getConnection()

  ```java
  @Override
  public Connection getConnection() throws SQLException {
    return popConnection(dataSource.getUsername(), dataSource.getPassword()).getProxyConnection();
  }
  ```

  即从连接池中获取连接，连接池是通过org.apache.ibatis.datasource.pooled.PoolState实现的：

  ```java
  package org.apache.ibatis.datasource.pooled;
  
  import java.util.ArrayList;
  import java.util.List;
  
  /**
   * @author Clinton Begin
   */
  public class PoolState {
  
    protected PooledDataSource dataSource;
  
    protected final List<PooledConnection> idleConnections = new ArrayList<>();
    protected final List<PooledConnection> activeConnections = new ArrayList<>();
    protected long requestCount = 0;
    protected long accumulatedRequestTime = 0;
    protected long accumulatedCheckoutTime = 0;
    protected long claimedOverdueConnectionCount = 0;
    protected long accumulatedCheckoutTimeOfOverdueConnections = 0;
    protected long accumulatedWaitTime = 0;
    protected long hadToWaitCount = 0;
    protected long badConnectionCount = 0;
  
    public PoolState(PooledDataSource dataSource) {
      this.dataSource = dataSource;
    }
  
    public synchronized long getRequestCount() {
      return requestCount;
    }
  
    public synchronized long getAverageRequestTime() {
      return requestCount == 0 ? 0 : accumulatedRequestTime / requestCount;
    }
  
    public synchronized long getAverageWaitTime() {
      return hadToWaitCount == 0 ? 0 : accumulatedWaitTime / hadToWaitCount;
  
    }
  
    public synchronized long getHadToWaitCount() {
      return hadToWaitCount;
    }
  
    public synchronized long getBadConnectionCount() {
      return badConnectionCount;
    }
  
    public synchronized long getClaimedOverdueConnectionCount() {
      return claimedOverdueConnectionCount;
    }
  
    public synchronized long getAverageOverdueCheckoutTime() {
      return claimedOverdueConnectionCount == 0 ? 0 : accumulatedCheckoutTimeOfOverdueConnections / claimedOverdueConnectionCount;
    }
  
    public synchronized long getAverageCheckoutTime() {
      return requestCount == 0 ? 0 : accumulatedCheckoutTime / requestCount;
    }
  
    public synchronized int getIdleConnectionCount() {
      return idleConnections.size();
    }
  
    public synchronized int getActiveConnectionCount() {
      return activeConnections.size();
    }
  
    @Override
    public synchronized String toString() {
      StringBuilder builder = new StringBuilder();
      builder.append("\n===CONFINGURATION==============================================");
      builder.append("\n jdbcDriver                     ").append(dataSource.getDriver());
      builder.append("\n jdbcUrl                        ").append(dataSource.getUrl());
      builder.append("\n jdbcUsername                   ").append(dataSource.getUsername());
      builder.append("\n jdbcPassword                   ").append(dataSource.getPassword() == null ? "NULL" : "************");
      builder.append("\n poolMaxActiveConnections       ").append(dataSource.poolMaximumActiveConnections);
      builder.append("\n poolMaxIdleConnections         ").append(dataSource.poolMaximumIdleConnections);
      builder.append("\n poolMaxCheckoutTime            ").append(dataSource.poolMaximumCheckoutTime);
      builder.append("\n poolTimeToWait                 ").append(dataSource.poolTimeToWait);
      builder.append("\n poolPingEnabled                ").append(dataSource.poolPingEnabled);
      builder.append("\n poolPingQuery                  ").append(dataSource.poolPingQuery);
      builder.append("\n poolPingConnectionsNotUsedFor  ").append(dataSource.poolPingConnectionsNotUsedFor);
      builder.append("\n ---STATUS-----------------------------------------------------");
      builder.append("\n activeConnections              ").append(getActiveConnectionCount());
      builder.append("\n idleConnections                ").append(getIdleConnectionCount());
      builder.append("\n requestCount                   ").append(getRequestCount());
      builder.append("\n averageRequestTime             ").append(getAverageRequestTime());
      builder.append("\n averageCheckoutTime            ").append(getAverageCheckoutTime());
      builder.append("\n claimedOverdue                 ").append(getClaimedOverdueConnectionCount());
      builder.append("\n averageOverdueCheckoutTime     ").append(getAverageOverdueCheckoutTime());
      builder.append("\n hadToWait                      ").append(getHadToWaitCount());
      builder.append("\n averageWaitTime                ").append(getAverageWaitTime());
      builder.append("\n badConnectionCount             ").append(getBadConnectionCount());
      builder.append("\n===============================================================");
      return builder.toString();
    }
  
  }
  
  ```

  同时这里获取的Connection是在PooledConnection中维护的java.sql.Connection的一个代理对象

  ```java
    public PooledConnection(Connection connection, PooledDataSource dataSource) {
      this.hashCode = connection.hashCode();
      this.realConnection = connection;
      this.dataSource = dataSource;
      this.createdTimestamp = System.currentTimeMillis();
      this.lastUsedTimestamp = System.currentTimeMillis();
      this.valid = true;
      this.proxyConnection = (Connection) Proxy.newProxyInstance(Connection.class.getClassLoader(), IFACES, this);
    }
  ```

  org.apache.ibatis.datasource.pooled.PooledConnection实现了java.lang.reflect.InvocationHandler接口：

  ```java
  
    /**
     * Required for InvocationHandler implementation.
     *
     * @param proxy
     *          - not used
     * @param method
     *          - the method to be executed
     * @param args
     *          - the parameters to be passed to the method
     * @see java.lang.reflect.InvocationHandler#invoke(Object, java.lang.reflect.Method, Object[])
     */
    @Override
    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
      String methodName = method.getName();
      if (CLOSE.equals(methodName)) {
        dataSource.pushConnection(this);
        return null;
      }
      try {
        if (!Object.class.equals(method.getDeclaringClass())) {
          // issue #579 toString() should never fail
          // throw an SQLException instead of a Runtime
          checkConnection();
        }
        return method.invoke(realConnection, args);
      } catch (Throwable t) {
        throw ExceptionUtil.unwrapThrowable(t);
      }
  
    }
  ```

  > 这里只对java.sql.Connection#close做了特殊处理，注意这个除了并没有真正关闭连接

  org.apache.ibatis.datasource.pooled.PooledDataSource#popConnection

  ```java
    private PooledConnection popConnection(String username, String password) throws SQLException {
      boolean countedWait = false;
      PooledConnection conn = null;
      long t = System.currentTimeMillis();
      int localBadConnectionCount = 0;
  
      while (conn == null) {
        synchronized (state) {
          if (!state.idleConnections.isEmpty()) {
            // Pool has available connection 池中有空闲连接，直接获取
            conn = state.idleConnections.remove(0);
            if (log.isDebugEnabled()) {
              log.debug("Checked out connection " + conn.getRealHashCode() + " from pool.");
            }
          } else {
            // Pool does not have available connection //池中没有空闲连接且当前池中的连接数量未达到上限，则创建新的连接
            if (state.activeConnections.size() < poolMaximumActiveConnections) {
              // Can create new connection
              conn = new PooledConnection(dataSource.getConnection(), this);
              if (log.isDebugEnabled()) {
                log.debug("Created connection " + conn.getRealHashCode() + ".");
              }
            } else {
              // Cannot create new connection //不能创建新的连接，则获取最早创建的连接，判断是否超时了，超时就把其删掉，并基于其创建新的PooledConnection（realdConnection不变）
              PooledConnection oldestActiveConnection = state.activeConnections.get(0);
              long longestCheckoutTime = oldestActiveConnection.getCheckoutTime();
              if (longestCheckoutTime > poolMaximumCheckoutTime) {
                // Can claim overdue connection
                state.claimedOverdueConnectionCount++;
                state.accumulatedCheckoutTimeOfOverdueConnections += longestCheckoutTime;
                state.accumulatedCheckoutTime += longestCheckoutTime;
                state.activeConnections.remove(oldestActiveConnection);
                if (!oldestActiveConnection.getRealConnection().getAutoCommit()) {
                  try {
                    oldestActiveConnection.getRealConnection().rollback();
                  } catch (SQLException e) {
                    /*
                       Just log a message for debug and continue to execute the following
                       statement like nothing happened.
                       Wrap the bad connection with a new PooledConnection, this will help
                       to not interrupt current executing thread and give current thread a
                       chance to join the next competition for another valid/good database
                       connection. At the end of this loop, bad {@link @conn} will be set as null.
                     */
                    log.debug("Bad connection. Could not roll back");
                  }
                }
                conn = new PooledConnection(oldestActiveConnection.getRealConnection(), this);
                conn.setCreatedTimestamp(oldestActiveConnection.getCreatedTimestamp());
                conn.setLastUsedTimestamp(oldestActiveConnection.getLastUsedTimestamp());
                oldestActiveConnection.invalidate();
                if (log.isDebugEnabled()) {
                  log.debug("Claimed overdue connection " + conn.getRealHashCode() + ".");
                }
              } else {
                // Must wait
                try {
                  if (!countedWait) {
                    state.hadToWaitCount++;
                    countedWait = true;
                  }
                  if (log.isDebugEnabled()) {
                    log.debug("Waiting as long as " + poolTimeToWait + " milliseconds for connection.");
                  }
                  long wt = System.currentTimeMillis();
                  state.wait(poolTimeToWait);
                  state.accumulatedWaitTime += System.currentTimeMillis() - wt;
                } catch (InterruptedException e) {
                  break;
                }
              }
            }
          }
          if (conn != null) {
            // ping to server and check the connection is valid or not 判断连接是否有效
            if (conn.isValid()) {
              if (!conn.getRealConnection().getAutoCommit()) {
                conn.getRealConnection().rollback();
              }
              conn.setConnectionTypeCode(assembleConnectionTypeCode(dataSource.getUrl(), username, password));
              conn.setCheckoutTimestamp(System.currentTimeMillis());
              conn.setLastUsedTimestamp(System.currentTimeMillis());
              state.activeConnections.add(conn);
              state.requestCount++;
              state.accumulatedRequestTime += System.currentTimeMillis() - t;
            } else {
              if (log.isDebugEnabled()) {
                log.debug("A bad connection (" + conn.getRealHashCode() + ") was returned from the pool, getting another connection.");
              }
              state.badConnectionCount++;
              localBadConnectionCount++;
              conn = null;
              if (localBadConnectionCount > (poolMaximumIdleConnections + poolMaximumLocalBadConnectionTolerance)) {
                if (log.isDebugEnabled()) {
                  log.debug("PooledDataSource: Could not get a good connection to the database.");
                }
                throw new SQLException("PooledDataSource: Could not get a good connection to the database.");
              }
            }
          }
        }
  
      }
  
      if (conn == null) {
        if (log.isDebugEnabled()) {
          log.debug("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
        }
        throw new SQLException("PooledDataSource: Unknown severe error condition.  The connection pool returned a null connection.");
      }
  
      return conn;
    }
  ```

  当connection.close()被调用时会调用org.apache.ibatis.datasource.pooled.PooledDataSource#pushConnection，即将Connection放入连接池中

  ```java
    protected void pushConnection(PooledConnection conn) throws SQLException {
  
      synchronized (state) {
        state.activeConnections.remove(conn);
        if (conn.isValid()) {
          if (state.idleConnections.size() < poolMaximumIdleConnections && conn.getConnectionTypeCode() == expectedConnectionTypeCode) {
            // 能放入空闲队列中就放入
            state.accumulatedCheckoutTime += conn.getCheckoutTime();
            if (!conn.getRealConnection().getAutoCommit()) {
              conn.getRealConnection().rollback();
            }
            PooledConnection newConn = new PooledConnection(conn.getRealConnection(), this);
            state.idleConnections.add(newConn);
            newConn.setCreatedTimestamp(conn.getCreatedTimestamp());
            newConn.setLastUsedTimestamp(conn.getLastUsedTimestamp());
            conn.invalidate();
            if (log.isDebugEnabled()) {
              log.debug("Returned connection " + newConn.getRealHashCode() + " to pool.");
            }
            state.notifyAll();
          } else {
            //不能放入空闲队列则获取真正的连接进行关闭操作
            state.accumulatedCheckoutTime += conn.getCheckoutTime();
            if (!conn.getRealConnection().getAutoCommit()) {
              conn.getRealConnection().rollback();
            }
            conn.getRealConnection().close();
            if (log.isDebugEnabled()) {
              log.debug("Closed connection " + conn.getRealHashCode() + ".");
            }
            conn.invalidate();
          }
        } else {
          if (log.isDebugEnabled()) {
            log.debug("A bad connection (" + conn.getRealHashCode() + ") attempted to return to the pool, discarding connection.");
          }
          state.badConnectionCount++;
        }
      }
    }
  ```

  

DataSource的创建使用了工厂方法模式

- org.apache.ibatis.datasource.DataSourceFactory

  ```java
package org.apache.ibatis.datasource;
  
  ```

import java.util.Properties;

  import javax.sql.DataSource;

  /**
   * @author Clinton Begin
      */
    public interface DataSourceFactory {

    void setProperties(Properties props);
      
    DataSource getDataSource();

  }
  ```
  
  - org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory
  
  ```java
    /**
     *    Copyright 2009-2015 the original author or authors.
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
    package org.apache.ibatis.datasource.unpooled;
    
    import java.util.Properties;
    
    import javax.sql.DataSource;
    
    import org.apache.ibatis.datasource.DataSourceException;
    import org.apache.ibatis.datasource.DataSourceFactory;
    import org.apache.ibatis.reflection.MetaObject;
    import org.apache.ibatis.reflection.SystemMetaObject;
    
    /**
     * @author Clinton Begin
     */
    public class UnpooledDataSourceFactory implements DataSourceFactory {
    
      private static final String DRIVER_PROPERTY_PREFIX = "driver.";
      private static final int DRIVER_PROPERTY_PREFIX_LENGTH = DRIVER_PROPERTY_PREFIX.length();
    
      protected DataSource dataSource;
    
      public UnpooledDataSourceFactory() {
        this.dataSource = new UnpooledDataSource();
      }
    
      /**
      * 设置UnpooledDataSource的属性
      */
      @Override
      public void setProperties(Properties properties) {
        Properties driverProperties = new Properties();
        MetaObject metaDataSource = SystemMetaObject.forObject(dataSource);
        for (Object key : properties.keySet()) {
          String propertyName = (String) key;
          if (propertyName.startsWith(DRIVER_PROPERTY_PREFIX)) {
            String value = properties.getProperty(propertyName);
            driverProperties.setProperty(propertyName.substring(DRIVER_PROPERTY_PREFIX_LENGTH), value);
          } else if (metaDataSource.hasSetter(propertyName)) {
            String value = (String) properties.get(propertyName);
            Object convertedValue = convertValue(metaDataSource, propertyName, value);
            metaDataSource.setValue(propertyName, convertedValue);
          } else {
            throw new DataSourceException("Unknown DataSource property: " + propertyName);
          }
        }
        if (driverProperties.size() > 0) {
          metaDataSource.setValue("driverProperties", driverProperties);
        }
      }
    
      @Override
      public DataSource getDataSource() {
        return dataSource;
      }
    
      private Object convertValue(MetaObject metaDataSource, String propertyName, String value) {
        Object convertedValue = value;
        Class<?> targetType = metaDataSource.getSetterType(propertyName);
        if (targetType == Integer.class || targetType == int.class) {
          convertedValue = Integer.valueOf(value);
        } else if (targetType == Long.class || targetType == long.class) {
          convertedValue = Long.valueOf(value);
        } else if (targetType == Boolean.class || targetType == boolean.class) {
          convertedValue = Boolean.valueOf(value);
        }
        return convertedValue;
      }
    
    }
    
  ```

​    

  - org.apache.ibatis.datasource.pooled.PooledDataSourceFactory：相比UnpooledDataSourceFactory，只是将dataSource属性改为PooledDataSource类型：
  
    ```java
    package org.apache.ibatis.datasource.pooled;
    
    import org.apache.ibatis.datasource.unpooled.UnpooledDataSourceFactory;
    
    /**
     * @author Clinton Begin
     */
    public class PooledDataSourceFactory extends UnpooledDataSourceFactory {
    
      public PooledDataSourceFactory() {
        this.dataSource = new PooledDataSource();
      }
    
    }
    ```
    
  - org.apache.ibatis.datasource.jndi.JndiDataSourceFactory：依赖JNDI从容器中获取用户配置的Datasource
  
    ```java
    package org.apache.ibatis.datasource.jndi;
    
    import java.util.Map.Entry;
    import java.util.Properties;
    
    import javax.naming.Context;
    import javax.naming.InitialContext;
    import javax.naming.NamingException;
    import javax.sql.DataSource;
    
    import org.apache.ibatis.datasource.DataSourceException;
    import org.apache.ibatis.datasource.DataSourceFactory;
    
    /**
     * @author Clinton Begin
     */
    public class JndiDataSourceFactory implements DataSourceFactory {
    
      public static final String INITIAL_CONTEXT = "initial_context";
      public static final String DATA_SOURCE = "data_source";
      public static final String ENV_PREFIX = "env.";
    
      private DataSource dataSource;
    
      @Override
      public void setProperties(Properties properties) {
        try {
          InitialContext initCtx;
          Properties env = getEnvProperties(properties);
          if (env == null) {
            initCtx = new InitialContext();
          } else {
            initCtx = new InitialContext(env);
          }
    
          if (properties.containsKey(INITIAL_CONTEXT) && properties.containsKey(DATA_SOURCE)) {
            Context ctx = (Context) initCtx.lookup(properties.getProperty(INITIAL_CONTEXT));
            dataSource = (DataSource) ctx.lookup(properties.getProperty(DATA_SOURCE));
          } else if (properties.containsKey(DATA_SOURCE)) {
            dataSource = (DataSource) initCtx.lookup(properties.getProperty(DATA_SOURCE));
          }
    
        } catch (NamingException e) {
          throw new DataSourceException("There was an error configuring JndiDataSourceTransactionPool. Cause: " + e, e);
        }
      }
    
      @Override
      public DataSource getDataSource() {
        return dataSource;
      }
    
      private static Properties getEnvProperties(Properties allProps) {
        final String PREFIX = ENV_PREFIX;
        Properties contextProperties = null;
        for (Entry<Object, Object> entry : allProps.entrySet()) {
          String key = (String) entry.getKey();
          String value = (String) entry.getValue();
          if (key.startsWith(PREFIX)) {
            if (contextProperties == null) {
              contextProperties = new Properties();
            }
            contextProperties.put(key.substring(PREFIX.length()), value);
          }
        }
        return contextProperties;
      }
    
    }
    
    
    ```
    
    



### Transation

- org.apache.ibatis.transaction.Transaction
  - org.apache.ibatis.transaction.jdbc.JdbcTransaction
  - org.apache.ibatis.transaction.managed.ManagedTransaction：生命周期是交给容器管理的

跟DataSource一样，使用了工厂方法模式：

- org.apache.ibatis.transaction.TransactionFactory
  - org.apache.ibatis.transaction.jdbc.JdbcTransactionFactory
  - org.apache.ibatis.transaction.managed.ManagedTransactionFactory



### Binding模块

#### MapperRegistry

org.apache.ibatis.binding.MapperRegistry主要维护mapper类与MapperProxyFactory的映射关系

```java
  private final Configuration config;
  private final Map<Class<?>, MapperProxyFactory<?>> knownMappers = new HashMap<>();
```

增加Mapper方法：org.apache.ibatis.binding.MapperRegistry#addMapper

```java
  public <T> void addMapper(Class<T> type) {
    if (type.isInterface()) {
      if (hasMapper(type)) {
        throw new BindingException("Type " + type + " is already known to the MapperRegistry.");
      }
      boolean loadCompleted = false;
      try {
        knownMappers.put(type, new MapperProxyFactory<>(type));
        // It's important that the type is added before the parser is run
        // otherwise the binding may automatically be attempted by the
        // mapper parser. If the type is already known, it won't try.
        MapperAnnotationBuilder parser = new MapperAnnotationBuilder(config, type);
        parser.parse();
        loadCompleted = true;
      } finally {
        if (!loadCompleted) {
          knownMappers.remove(type);
        }
      }
    }
  }
```

org.apache.ibatis.builder.annotation.MapperAnnotationBuilder#parse:

```java
  public void parse() {
    String resource = type.toString();
    if (!configuration.isResourceLoaded(resource)) {
      loadXmlResource();
      configuration.addLoadedResource(resource);
      assistant.setCurrentNamespace(type.getName());
      parseCache();
      parseCacheRef();
      for (Method method : type.getMethods()) {
        if (!canHaveStatement(method)) {
          continue;
        }
        if (getAnnotationWrapper(method, false, Select.class, SelectProvider.class).isPresent()
            && method.getAnnotation(ResultMap.class) == null) {
          parseResultMap(method);
        }
        try {
          parseStatement(method);
        } catch (IncompleteElementException e) {
          configuration.addIncompleteMethod(new MethodResolver(this, method));
        }
      }
    }
    parsePendingMethods();
  }
```

其中loadXmlResource()触发了解析mapper映射文件，生成org.apache.ibatis.mapping.MappedStatement

#### ParamNameResolver

参数名称解析类org.apache.ibatis.reflection.ParamNameResolver



### 缓存模块

#### 装饰器模式

核心角色类图

![image-20210720212649275](orm.assets\image-20210720212649275.png)

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

![image-20210720212936017](orm.assets\image-20210720212936017.png)

- org.apache.ibatis.cache.decorators.BlockingCache



#### CacheKey

缓存key，在mybatis中难以通过一个字符串来作为key，其中涉及到动态sql等方面因素，所以使用一个对象作为key

cache重写了equals和hashcode方法

```java
public boolean equals(Object object) {
    if (this == object) {
        return true;
    } else if (!(object instanceof CacheKey)) {
        return false;
    } else {
        CacheKey cacheKey = (CacheKey)object;
        if (this.hashcode != cacheKey.hashcode) {
            return false;
        } else if (this.checksum != cacheKey.checksum) {
            return false;
        } else if (this.count != cacheKey.count) {
            return false;
        } else {
            for(int i = 0; i < this.updateList.size(); ++i) {
                Object thisObject = this.updateList.get(i);
                Object thatObject = cacheKey.updateList.get(i);
                if (!ArrayUtil.equals(thisObject, thatObject)) {
                    return false;
                }
            }

            return true;
        }
    }
}

public int hashCode() {
    return this.hashcode;
}
```

从中看出确定是否同一个缓存的属性为：

```java
    private int hashcode;
    private long checksum;
    private int count;
    private List<Object> updateList;
```

update方法会对updateList更新，并以此计算出其他值，可见最终确定唯一性的字段是updateList中的对象

```java
public void update(Object object) {
    int baseHashCode = object == null ? 1 : ArrayUtil.hashCode(object);
    ++this.count;
    this.checksum += (long)baseHashCode;
    baseHashCode *= this.count;
    this.hashcode = this.multiplier * this.hashcode + baseHashCode;
    this.updateList.add(object);
}
```

![image-20210721092837981](orm.assets\image-20210721092837981.png)





## 核心处理层

从xml启动的方式查找入口：

```java
package com.mybatis.test;


import com.mybatis.mapper.BlogMapper;
import org.apache.ibatis.io.Resources;
import org.apache.ibatis.session.SqlSession;
import org.apache.ibatis.session.SqlSessionFactory;
import org.apache.ibatis.session.SqlSessionFactoryBuilder;
import java.io.InputStream;
import java.util.HashMap;
import java.util.Map;

/**
 * 通过xml配置启动使用mybatis
 */
public class StartWithXml {

    public static void main(String[] args) throws Exception {
        String resource = "mybatis-config.xml";
        InputStream stream = Resources.getResourceAsStream(resource);
        //加载config配置文件，并创建SqlSessionFactory对象
        SqlSessionFactory sqlSessionFactory = new SqlSessionFactoryBuilder().build(stream);
        //创建SqlSession对象
        SqlSession sqlSession = sqlSessionFactory.openSession();
        Map<String, Object> param = new HashMap<>();
        param.put("id", 1);
        //Blog blog = (Blog) sqlSession.selectOne("com.mybatis.mapper.BlogMapper.selectBlogDetail", param);
        //System.out.println(blog);

        BlogMapper mapper = sqlSession.getMapper(BlogMapper.class);
        System.out.println(mapper.getTime());
    }

}

```

org.apache.ibatis.session.SqlSessionFactoryBuilder#build(java.io.InputStream, java.lang.String, java.util.Properties)：

```java
public SqlSessionFactory build(InputStream inputStream, String environment, Properties properties) {
  try {
    XMLConfigBuilder parser = new XMLConfigBuilder(inputStream, environment, properties);
    return build(parser.parse());
  } catch (Exception e) {
    throw ExceptionFactory.wrapException("Error building SqlSession.", e);
  } finally {
    ErrorContext.instance().reset();
    try {
      inputStream.close();
    } catch (IOException e) {
      // Intentionally ignore. Prefer previous error.
    }
  }
}
```

org.apache.ibatis.builder.xml.XMLConfigBuilder#parse

```java
  public Configuration parse() {
    if (parsed) {
      throw new BuilderException("Each XMLConfigBuilder can only be used once.");
    }
    parsed = true;
    parseConfiguration(parser.evalNode("/configuration"));
    return configuration;
  }
```

org.apache.ibatis.builder.xml.XMLConfigBuilder#parseConfiguration

```java
  private void parseConfiguration(XNode root) {
    try {
      // issue #117 read properties first
      propertiesElement(root.evalNode("properties"));
      Properties settings = settingsAsProperties(root.evalNode("settings"));
      loadCustomVfs(settings);
      loadCustomLogImpl(settings);
      typeAliasesElement(root.evalNode("typeAliases"));
      pluginElement(root.evalNode("plugins"));
      objectFactoryElement(root.evalNode("objectFactory"));
      objectWrapperFactoryElement(root.evalNode("objectWrapperFactory"));
      reflectorFactoryElement(root.evalNode("reflectorFactory"));
      settingsElement(settings);
      // read it after objectFactory and objectWrapperFactory issue #631
      environmentsElement(root.evalNode("environments"));
      databaseIdProviderElement(root.evalNode("databaseIdProvider"));
      typeHandlerElement(root.evalNode("typeHandlers"));
      mapperElement(root.evalNode("mappers"));
    } catch (Exception e) {
      throw new BuilderException("Error parsing SQL Mapper Configuration. Cause: " + e, e);
    }
  }
```

这里主要是解析mybatis-config文件，然后组装成org.apache.ibatis.session.Configuration配置



org.apache.ibatis.mapping.DatabaseIdProvider使用：

mybatis-config.xml中增加DatabaseIdProvider配置：

```java
    <databaseIdProvider type="DB_VENDOR">
        <property name="Oracle" value="oracle"/>
        <property name="MySQL" value="mysql"/>
    </databaseIdProvider>
```

org.apache.ibatis.mapping.VendorDatabaseIdProvider#getDatabaseName会决策出使用哪个databaseId（根据当前连接的数据库）

mapper配置文件中：

```java
    <select id="getTime" resultType="String" databaseId="mysql">
        select now() from dual
    </select>

    <select id="getTime" resultType="String" databaseId="oracle">
        select  'oralce'||to_char(sysdate,'yyyy-mm-dd hh24:mi:ss')  from dual
    </select>
```

这样下来，根据链接配置，会决定使用哪个语句





### XMLMapperBuilder

org.apache.ibatis.builder.xml.XMLMapperBuilder主要是解析mapper映射文件

org.apache.ibatis.builder.xml.XMLMapperBuilder#parse

```java
  public void parse() {
    if (!configuration.isResourceLoaded(resource)) {
      configurationElement(parser.evalNode("/mapper"));
      configuration.addLoadedResource(resource);
      //绑定mapper class和映射文件
      bindMapperForNamespace();
    }
	//重新解析失败的内容
    parsePendingResultMaps();
    parsePendingCacheRefs();
    parsePendingStatements();
  }
```





### XMLStatementBuilder

org.apache.ibatis.builder.xml.XMLStatementBuilder用来解析mapper文件中的SQL节点

mybatis中使用org.apache.ibatis.mapping.SqlSource表示映射文件或者注解中的SQL语句

```java
package org.apache.ibatis.mapping;

/**
 * Represents the content of a mapped statement read from an XML file or an annotation.
 * It creates the SQL that will be passed to the database out of the input parameter received from the user.
 *
 * @author Clinton Begin
 */
public interface SqlSource {

  BoundSql getBoundSql(Object parameterObject);

}
```

mybatis中使用org.apache.ibatis.mapping.**MappedStatement**表示映射文件中定义的SQL节点，其中属性：

```java
  private String resource; 
  private Configuration configuration;
  private String id;
  private Integer fetchSize;
  private Integer timeout;
  private StatementType statementType;
  private ResultSetType resultSetType;
  private SqlSource sqlSource; //对应一条SQL语句
  private Cache cache;
  private ParameterMap parameterMap;
  private List<ResultMap> resultMaps;
  private boolean flushCacheRequired;
  private boolean useCache;
  private boolean resultOrdered;
  private SqlCommandType sqlCommandType; //SQL类型 INSERT SELECT DELETE UPDATE
  private KeyGenerator keyGenerator;
  private String[] keyProperties;
  private String[] keyColumns;
  private boolean hasNestedResultMaps;
  private String databaseId;
  private Log statementLog;
  private LanguageDriver lang;
  private String[] resultSets;
```



解析SQL节点入口：org.apache.ibatis.builder.xml.**XMLStatementBuilder**#parseStatementNode

```java
  public void parseStatementNode() {
    String id = context.getStringAttribute("id");
    String databaseId = context.getStringAttribute("databaseId");

    if (!databaseIdMatchesCurrent(id, databaseId, this.requiredDatabaseId)) {
      return;
    }

    String nodeName = context.getNode().getNodeName();
    SqlCommandType sqlCommandType = SqlCommandType.valueOf(nodeName.toUpperCase(Locale.ENGLISH));
    boolean isSelect = sqlCommandType == SqlCommandType.SELECT;
    boolean flushCache = context.getBooleanAttribute("flushCache", !isSelect);
    boolean useCache = context.getBooleanAttribute("useCache", isSelect);
    boolean resultOrdered = context.getBooleanAttribute("resultOrdered", false);

    // Include Fragments before parsing
    XMLIncludeTransformer includeParser = new XMLIncludeTransformer(configuration, builderAssistant);
    includeParser.applyIncludes(context.getNode());

    String parameterType = context.getStringAttribute("parameterType");
    Class<?> parameterTypeClass = resolveClass(parameterType);

    String lang = context.getStringAttribute("lang");
    LanguageDriver langDriver = getLanguageDriver(lang);

    // Parse selectKey after includes and remove them.
    processSelectKeyNodes(id, parameterTypeClass, langDriver);

    // Parse the SQL (pre: <selectKey> and <include> were parsed and removed)
    KeyGenerator keyGenerator;
    String keyStatementId = id + SelectKeyGenerator.SELECT_KEY_SUFFIX;
    keyStatementId = builderAssistant.applyCurrentNamespace(keyStatementId, true);
    if (configuration.hasKeyGenerator(keyStatementId)) {
      keyGenerator = configuration.getKeyGenerator(keyStatementId);
    } else {
      keyGenerator = context.getBooleanAttribute("useGeneratedKeys",
          configuration.isUseGeneratedKeys() && SqlCommandType.INSERT.equals(sqlCommandType))
          ? Jdbc3KeyGenerator.INSTANCE : NoKeyGenerator.INSTANCE;
    }

    SqlSource sqlSource = langDriver.createSqlSource(configuration, context, parameterTypeClass);
    StatementType statementType = StatementType.valueOf(context.getStringAttribute("statementType", StatementType.PREPARED.toString()));
    Integer fetchSize = context.getIntAttribute("fetchSize");
    Integer timeout = context.getIntAttribute("timeout");
    String parameterMap = context.getStringAttribute("parameterMap");
    String resultType = context.getStringAttribute("resultType");
    Class<?> resultTypeClass = resolveClass(resultType);
    String resultMap = context.getStringAttribute("resultMap");
    String resultSetType = context.getStringAttribute("resultSetType");
    ResultSetType resultSetTypeEnum = resolveResultSetType(resultSetType);
    if (resultSetTypeEnum == null) {
      resultSetTypeEnum = configuration.getDefaultResultSetType();
    }
    String keyProperty = context.getStringAttribute("keyProperty");
    String keyColumn = context.getStringAttribute("keyColumn");
    String resultSets = context.getStringAttribute("resultSets");

    builderAssistant.addMappedStatement(id, sqlSource, statementType, sqlCommandType,
        fetchSize, timeout, parameterMap, parameterTypeClass, resultMap, resultTypeClass,
        resultSetTypeEnum, flushCache, useCache, resultOrdered,
        keyGenerator, keyProperty, keyColumn, databaseId, langDriver, resultSets);
  }
```

最终会往Configuration中增加解析后的MappedStatement对象

```java
  protected final Map<String, MappedStatement> mappedStatements = new StrictMap<MappedStatement>("Mapped Statements collection")
      .conflictMessageProducer((savedValue, targetValue) ->
          ". please check " + savedValue.getResource() + " and " + targetValue.getResource());  
  public void addMappedStatement(MappedStatement ms) {
    mappedStatements.put(ms.getId(), ms);
  }
```



### ResultSetHandler

```java
package org.apache.ibatis.executor.resultset;

import java.sql.CallableStatement;
import java.sql.SQLException;
import java.sql.Statement;
import java.util.List;

import org.apache.ibatis.cursor.Cursor;

/**
 * @author Clinton Begin
 */
public interface ResultSetHandler {
  //处理结果集，生成相应的结果对象集合
  <E> List<E> handleResultSets(Statement stmt) throws SQLException;
  //处理结果集，返回相应的游标对象
  <E> Cursor<E> handleCursorResultSets(Statement stmt) throws SQLException;
  //处理存储过程的输出参数
  void handleOutputParameters(CallableStatement cs) throws SQLException;

}

```

