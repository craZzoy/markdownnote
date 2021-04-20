# 数组

- 优点
  - 是快速查询，即通过索引查询
- 缺点
  - 创建时数组容量就已经确定，不能更改
  - 容易引起数组溢出错误
  - 不能动态地操作一个数组，即没有增删方法
- 索引最好有意义



# Map

## HashMap、HashTable、ConcurrentHashMap

- HashMap

  - key和value都能为空
  - 线程不安全

- HashTable

  - key能为空，value不能为空

    ```java
    public synchronized V put(K key, V value) {
            // Make sure the value is not null
            if (value == null) {
                throw new NullPointerException();
            }
        ...
    }
    ```

  - 多线程安全，使用synchronized实现多线程安全

- ConcurrentHashMap

  - key和value都不能为空
  - 多线程安全，JDK7中使用分段锁实现线程安全