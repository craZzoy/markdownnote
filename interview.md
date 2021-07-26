# 基础

1、常见网络协议

- HTTP/HTTPS：超文本传输协议
- FTP：文件传输协议
- SMTP：简单邮件协议
- TELNET：远程终端协议
- POP3：邮件读取协议

2、什么是JVM，java虚拟机包括什么？

JVM：Java virtual machine，java虚拟机。运用硬件或者软件的手段实现的虚拟的计算机。java虚拟机包括：寄存器、堆栈、处理器

3、JDK、JRE

- JDK：java development kit，java开发工具包，是java开发人员所需安装的环境
- JRE：java runtime environment，java运行环境，java程序运行所需安装的环境

4、java基本数据类型

boolean、byte、char、short、int、long、float、double

- 整型：byte（8）、short（16）、int（32）、long（64）
- 浮点数：float（32）、double（64）
- 字符型：char（16）
- 布尔型：boolean（8）

5、java的数据结构有哪些？

- 线性表（ArrayList）
- 链表（LinkedList）
- 栈（Stack）
- 队列（Queue）
- 图（Map）
- 数（Tree）

6、Char类型能不能转为int类型？能不能转为String类型，能不能转为double类型

```java
package com.basicvar;
public class CharTest {

    public static void main(String[] args) {
        char c = '汉';
        byte byteV = (byte) c;
        short shortV = (short) c;
        int intV = c;
        long longV = c;
        //不能强转为String
        //String strV = (String) c;
        String strV = c + "";
        System.out.println(c);
        System.out.printf("byteV：%d\n",byteV);
        System.out.printf("shortV：%d\n",shortV);
        System.out.printf("intV：%d\n",intV);
        System.out.printf("longV：%d\n",longV);
        System.out.printf("strV：%s\n",strV);
    }

}

```

```txt
汉
byteV：73
shortV：27721
intV：27721
longV：27721
strV：汉
```

7、char类型可以存储汉字不，char是什么编码

可以，char类型在JAVA中是用Unicode编码存储的

8、java中是值传递还是引用传递？

理论上说，java都是引用传递。

- 对于基本类型，传递的是值的副本，而不是值本身。
- 对于对象而言，传递的是对象的引用，当一个方法操作传递参数的时候，实际操作的是引用所指的对象。

```java
package com.base;

import com.domain.User;

import java.lang.reflect.Field;

/**
 * 参数传递
 */
public class ParameterPass {

    public static void main(String[] args) throws Exception {
        int a = 4;
        Integer inta = 290;
        String s = "Hello";
        User user = new User("Jack", 29, "jack@163.com");
        testPass(a, s, user, inta);
        System.out.println(a);
        System.out.println(s);
        System.out.println(user);
        System.out.println(inta);
    }

    private static void testPass(int a, String s, User user, Integer inta) throws Exception {
        a = 5;
        s = "World";
        user.setAge(30);
        //inta = 23;
        Field value = inta.getClass().getDeclaredField("value");
        value.setAccessible(true);
        value.set(inta, 350);
    }

}

```

```txt
4
Hello
User{username='Jack', age=30, email='jack@163.com', address=null}
350
```

```java
package com.base;

import com.domain.User;

import java.lang.reflect.Field;

/**
 * 参数传递
 */
public class ParameterPass {

    public static void main(String[] args) throws Exception {
        int a = 4;
        Integer inta = 290;
        String s = "Hello";
        User user = new User("Jack", 29, "jack@163.com");
        testPass(a, s, user, inta);
        System.out.println(a);
        System.out.println(s);
        System.out.println(user);
        System.out.println(inta);
    }

    private static void testPass(int a, String s, User user, Integer inta) throws Exception {
        a = 5;
        s = "World";
        user.setAge(30);
        inta = 23;
        Field value = inta.getClass().getDeclaredField("value");
        value.setAccessible(true);
        value.set(inta, 350);
    }

}

```

```txt
4
Hello
User{username='Jack', age=30, email='jack@163.com', address=null}
290
```

9、内部类和静态类的区别

![image-20210724183857298](D:\BaiduNetdiskDownload\markdown笔记\interview.assets\image-20210724183857298.png)

10、静态类可以被继承不？肯定可以啦，不是final修饰的都可以。

11、java修饰符

private（当前类可见）<default（当前包可见）<protected（当前包或子类可见）<public（所有类可见）

12、接口有什么特点

- 接口中声明的全是public static final修饰的常量（jdk8后可以不是）

- 接口中所有方法都是抽象方法（java 8中有default）

- 接口没有构造方法

- 接口不能直接实例化

- 接口可以多继承

  ```java
  package com.base;
  
  import java.io.Serializable;
  
  public class InterfaceTest {
  
  
      interface InterfaceA extends Serializable, Cloneable {
  
          default void echo(){
              System.out.println("echo");
          }
  
      }
  
  }
  
  ```

13、接口和抽象类有什么区别

- 抽象类有构造方法，接口没有
- 抽象类只能单继承，接口可以多继承
- 抽象类可以有普通方法，接口中的所有方法都是抽象方法（Jdk8后可以有默认实现）
- 接口的属性都是public static final修饰的，而抽象的不是

14、说出常见的运行时异常

![image-20210724220302380](D:\BaiduNetdiskDownload\markdown笔记\interview.assets\image-20210724220302380.png)

15、Java中的集合框架有几个

- Collection
- Map

16、List（线性表）接口和Set（无序集合）接口分别有什么特点？

- 顺序存储，可以有重复值
- 无序存储，不可以有重复值

17、ArrayList和LinkedList有什么区别？

- ArrayList是基于数组实现的线性表。尾端插入和查询性能较高
- LinkedArrayList是基于双向链表实现的线性表。插入和删除性能较高，查找性能低。

18、JDBC操作的步骤

1. 加载驱动
2. 打开数据库连接
3. 执行语句
4. 处理结果
5. 关闭资源

19、JDBC中怎么防止sql注入？

使用PrepareStatement（可以强制转换参数）而不是Statement

20、连接池的好处？

对于客户端来说，可以很好的管理、分配、释放数据库连接资源。而对于数据库来说，连接池可以控制连接的数量，以免数据库因为连接过多而导致性能问题。

21、数据源技术有哪些？好处是什么

DBCP、C3P0等，用的C3P0较多，因为其比较稳定，安全。

通过使用数据源，可以在配置文件中配置数据库相关的配置，而不是通过硬编码的方式。

22、Java中的IO分为哪两种

- 按功能
  - 输出流
  - 输入流
- 按类型
  - 字节流
  - 字符流

23、常用的IO类有哪些？

![image-20210724223825370](D:\BaiduNetdiskDownload\markdown笔记\interview.assets\image-20210724223825370.png)

24、字节流和字符流的区别

- 字节流：以字节为单位，按8位传输
- 字符流：以字符为单位，按16位传输

25、进程和线程

- 进程：操作系统进行资源分配和调度的基本单位。如exe就是以进程的运行的。
- 线程：轻量级进程。程序执行的最小单元。

1、springbean的生命周期
2、bean的作用域
3、分布式事务
4、springmvc的流程
5、es索引结构
6、es内存存储状态

7、LinkedHashMap实现LRU
8、AtomicInteger如何保证线程安全
9、spring何如解决循环依赖
10、前缀树
11、非主键索引的存储
12、zookeeper
13、kafka
14、es数据类型
15、Future 于CompalteableFuture的区别
16、间隙锁
17、spring事物的失效场景
18、redis zset存储结构
19、redis红锁
20、分布式锁
21、springboot的启动流程
22、mysql的事务实现原理
23、Kafka isr
24、在什么情况会指令重拍
25、cms、g1
26、KAFKA如何保证
27、mysql主从同步延迟时间过长，如何保证
28、mysql如何实现持久化
29、redis事务
30、mysql主键索引与普通索引存储结构的区别是什么
31、MyIsam与innodb的区别
32、volatile内存屏障实现原理
33、线程池
34、Synchronized与Reentrantlock区别
35、mysql都有什么类型的锁

36、线程池怎么知道线程运行完了

1. java.util.concurrent.ExecutorService#isTerminated接口
2. CountDownLatch
3. 使用重入锁，维护公共系数
4. 借助java.util.concurrent.ExecutorService#submit(java.util.concurrent.Callable<T>)返回的Future



面试题记录

1. 整体描述Spring

2. Spring中的单例是否线程安全

   分情况：

   - 无状态Bean：线程安全
   - 有状态Bean：有状态Bean是指是否用了实例属性保存数据，这种情况线程不安全，需要自己保证多线程安全：
     - 使用锁，Class对象锁，静态字段锁
     - 分布式锁（杀鸡用牛刀，这种使用于分布式集群场景）
     - Bean作用域设为prototype

3. 线程池创建方式，提交任务，submit方法和execute()方法区别

4. mybatis #和$区别

5. 浅谈分布式架构

6. Spring作用域

7. mq消息保证



# 多线程相关
# mysql中的事务和spring中的事务
线程之间的通信方式

- 共享内存
- 消息传递
# HashMap(JDK8)相关原理初探



## 哈希表数据结构
首先了解下hashTable的数据结构



## 内部数据结构
hashmap的底层数据结构其实是**数组**加**单向链表**的形式
证明：
hashmap有这个属性

```java
transient Node<K,V>[] table;
```
Node是一个内部类，其中有一个next属性指向一个Node，可见其是单向的
```java
    static class Node<K,V> implements Map.Entry<K,V> {
    	//hash(key)计算出来的结果
        final int hash;
        final K key;
        V value;
        Node<K,V> next;

        Node(int hash, K key, V value, Node<K,V> next) {
            this.hash = hash;
            this.key = key;
            this.value = value;
            this.next = next;
        }

        public final K getKey()        { return key; }
        public final V getValue()      { return value; }
        public final String toString() { return key + "=" + value; }

        public final int hashCode() {
            return Objects.hashCode(key) ^ Objects.hashCode(value);
        }

        public final V setValue(V newValue) {
            V oldValue = value;
            value = newValue;
            return oldValue;
        }

        public final boolean equals(Object o) {
            if (o == this)
                return true;
            if (o instanceof Map.Entry) {
                Map.Entry<?,?> e = (Map.Entry<?,?>)o;
                if (Objects.equals(key, e.getKey()) &&
                    Objects.equals(value, e.getValue()))
                    return true;
            }
            return false;
        }
    }
```



## 看看构造方法

无参构造器：这里值初始化了一个loadFactor参数，DEFAULT_LOAD_FACTOR值为0.75，先记住这个参数，我们叫它拓容因子，具体后面会提到
```java
  ...
  static final float DEFAULT_LOAD_FACTOR = 0.75f;
  ...
  /**
 * 无参构造器
  */
  public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; // all other fields defaulted
    }
```
这里的构造器仅仅初始化了一个属性，其他属性呢？table的大小呢？什么时候初始化的，暂时先放着，再看看其他的构造器
```java
    /**
     * The next size value at which to resize (capacity * load factor).
     * 下次调整大小时的容量
     *
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;

	/**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
     //指定初始容量和加载因子
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        //下一次调整后的数组容量大小，返回2的n次幂的值
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
	//指定初始容量
    public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```
假如我们使用Map map = new HashMap(11)语句初始化HashMap，11用来初始化threshold这个属性，其它threshold保存的就是table数组的容量，后面会提到。
```java
    /**
     * The next size value at which to resize (capacity * load factor).
     *
     * @serial
     */
    // (The javadoc description is true upon serialization.
    // Additionally, if the table array has not been allocated, this
    // field holds the initial array capacity, or zero signifying
    // DEFAULT_INITIAL_CAPACITY.)
    int threshold;
```
接着我们再看看this.threshold = tableSizeFor(initialCapacity)这里干了什么，从注释里可以看出，这里根据给出的值返回一个2的n次幂的值，比如给的11返回16，给的17返回32，给的33返回64。
```java
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```
先记住现有问题
 * DEFAULT_LOAD_FACTOR(0.75)赋值给了loadFactor属性，这是干嘛用的？
 * threshold属性是什么，用来初始化table的大小?
 * threshold这个值最终为什么是2^n?



### PUT的过程
首先我们看看put方法具体干了什么
```java
    /**
     * Associates the specified value with the specified key in this map.
     * If the map previously contained a mapping for the key, the old
     * value is replaced.
     *
     * @param key key with which the specified value is to be associated
     * @param value value to be associated with the specified key
     * @return the previous value associated with <tt>key</tt>, or
     *         <tt>null</tt> if there was no mapping for <tt>key</tt>.
     *         (A <tt>null</tt> return can also indicate that the map
     *         previously associated <tt>null</tt> with <tt>key</tt>.)
     */
    public V put(K key, V value) {
    	//1、hash(key)
        return putVal(hash(key), key, value, false, true);
    }
        /**
     * Implements Map.put and related methods
     *
     * @param hash hash for key
     * @param key the key
     * @param value the value to put
     * @param onlyIfAbsent if true, don't change existing value
     * @param evict if false, the table is in creation mode.
     * @return previous value, or null if none
     */
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        //2、数组为空，初始化数组
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //3、定位到table数组的某个位置，数据是空的
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else { //4、定位到的table数字位置数据不为空，hash冲突了
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k)))) //4a、数组（判断是否与数组内第一个元素hash冲突和key是否相等）
                e = p;
            else if (p instanceof TreeNode) //4b、节点是红黑树时赋值操作
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else { //4c、链表
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        //4ca、当链表长度大于8时，链表晋升为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            //4d、若key冲突，覆盖原来的value，返回旧value
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        //当前Hashmap结构上的改变次数，包括改变其中的mapping的数量和rehash
        ++modCount;
        //5、拓容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
put中给putVal传递了key、value、hash(key)等值，先关注下hash(key)干了什么
1、hash(key)方法

```java
static final int hash(Object key) {
        int h;
        return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
```
可以看出，这里做的是：key为空返回0，不为空返回key的hashCode的低16位和其高16位异或结果，是int类型。假如key=name，那么算出hash("name")=3373752，二进制为1100110111101010111000

2、table数组为空时，通过resize()初始化

```java
    /**
     * Initializes or doubles table size.  If null, allocates in
     * accord with initial capacity target held in field threshold.
     * Otherwise, because we are using power-of-two expansion, the
     * elements from each bin must either stay at same index, or move
     * with a power of two offset in the new table.
     *
     * @return the table
     */
    final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            //超过最大容量1 << 30，指定容量为0x7fffffff
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //5a 拓容
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {
            //2a、oldCap=0，即table为空
            // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
        @SuppressWarnings({"rawtypes","unchecked"})
        //2b、新生成的table，长度为newCap
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        //5b、拓容之后的数据移动操作
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) {
                    oldTab[j] = null;
                    if (e.next == null) //5ba、单个元素移动，按原有公式hash&cap-1进行计算位置
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode) //5bb、红黑树数据移动
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order //5bc、链表数据移动
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) {
                                if (loTail == null)
                                    loHead = e;
                                else
                                    loTail.next = e;
                                loTail = e;
                            }
                            else {
                                if (hiTail == null)
                                    hiHead = e;
                                else
                                    hiTail.next = e;
                                hiTail = e;
                            }
                        } while ((e = next) != null);
                        if (loTail != null) {
                            loTail.next = null;
                            newTab[j] = loHead;
                        }
                        if (hiTail != null) {
                            hiTail.next = null;
                            newTab[j + oldCap] = hiHead;
                        }
                    }
                }
            }
        }
        return newTab;
    }

```
根据Map map = new HashMap()初始化创建hashmap对象的前提下，我们可以模拟出它的执行流程，先看(2a)这块
```java
//2a、oldCap=0，即table为空
            // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
```
这里计算结果newCap = 16，newThr=12
再看(2b)这一块
```java
//2b、给属性threshold赋值，生成新table，长度为newCap
		threshold = newThr;
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
```
根据前面的计算结果，threshold = 12，table初始化为长度为16的Node数组

3、接着定位到(3)块的内容

```java
//3、定位到table数组的某个位置，数据是空的
if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
```
这里逻辑是定位到table数组的某个位置，数据是空的，根据key value组装Node节点。hash值是前面hash(key)的值，即1100110111101010111000，tab[i = (n - 1) & hash]根据hash值定位了存取数据的数据具体位置
0000000000000000001111
&
1100110111101010111000
可见最终值一定介于0000-1111之间，即0-15之间，到此我们随机地将数据放到了table数组中，并且不会越界。其实hash%16与hash&15效果是等价的，但是&运算效率更高。

**为什么是2^n？**
因为此时cap-1的值低n位都是1，**最终结果取决于hash值**得到低n位，这样能够使算出来的索引尽量分散，并且结果是在0至(2^n)-1之间。

4、定位到的table数字位置数据不为空，即hash值冲突

 - (4a)块：数组
 - (4b)块：节点是红黑树时赋值操作
 - (4c)块：节点是链表时赋值操作，当链表长度大于8时，链表晋升为红黑树


5、拓容
```java
//5、拓容
        if (++size > threshold)
            resize();
```
当数组中不为空的元素数量大于DEFAULT_LOAD_FACTOR*DEFAULT_INITIAL_CAPACITY的值，即12时，会调用resize()托容。resize不仅用来初始化table数组，还可以用来对其拓容。
我们继续定位到(5a)这一块内容

```java
if (oldCap > 0) {
            //超过最大容量1 << 30，指定容量为0x7fffffff
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //5a 拓容
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
```
可见这里实现的是双倍拓容
继续往下走，我们到了(5b)这一块，这里是拓容之后的数据移动操作

 - (5ba)块：单个元素移动，按原有公式hash&cap-1进行计算位置
 - (5bb)块：红黑树数据移动
 - (5bc)块：链表数据移动，其移动依据这个条件(e.hash & oldCap) == 0，其实根据oldCap为1的比特位位置，找到e.hash对应位数的值，由它决定数据的去留，只有两种可能
0：newTab[j] = loHead；即放在数组原来位置
1：newTab[j + oldCap] = hiHead；即放在原索引位置+oldCap



# 测试相关

1、rpc怎么标记流量是压测流量

2、普通的代码压测的时候怎么样才能尽可能的仿真

https://www.infoq.cn/article/O9i6BfecBm3TI5htfM1r





# 时间日期

## 常用时区

- UTC：世界协调时
  - UTC+08：提前8小时，中国标准时间。
  - CST：UTC+08知名名称之一





# Bitmap和布隆过滤器

## 布隆算法

布隆算法是以bitmap集合为基础的排重算法

使用场景：url去重、垃圾邮箱过滤

问题：字符串hashcode可能重复，不同url的hashcode可能是相同的

解决方式：对一个url进行hash多次计算（一般8次），讲生成值对应bitmap位置置1，这种方式还是会误判，hash计算次数越多，错误率越低。

对应一定不能误判的业务，可对误判的值维护一个白名单。

bitmap：https://mp.weixin.qq.com/s?__biz=MzIxMjE5MTE1Nw==&mid=2653191272&idx=1&sn=9bbcd172b611b455ebfc4b7fb9a6a55e&chksm=8c990eb2bbee87a486c55572a36c577a48df395e13e74314846d221cbcfd364d44c280250234&scene=21#wechat_redirect

布隆算法：https://mp.weixin.qq.com/s/RmR5XmLeMvk35vgjwxANFQ



# 题目记录

1. sql中in和exists区别

   参考：https://www.cnblogs.com/liyasong/p/sql_in_exists.html

   https://www.cnblogs.com/weifeng123/p/9530758.html

   执行流程对比：

   - select a.* from A a where a.bid in (select distinct id from B)；

     流程：in后面语句先执行-->根据前面语句

   - select a.* from A a where exists (select 1 from b B where )

     流程：先执行exists前面语句-->再根据exists判断数据是否符合要求

   一般认为：A表数据量大时适合in，B表数据量大时适合exists。实际情况还需结合sql执行计划进行判断。

   其他区别：

   - in和exists都能尝试命中索引
   - not in不能命中索引，not exists可以

2. spring中的事务机制

   https://zhuanlan.zhihu.com/p/148504094

   spring中的事务传播级别

   ```java
   public enum Propagation {
       REQUIRED(0),
       SUPPORTS(1),
       MANDATORY(2),
       REQUIRES_NEW(3),
       NOT_SUPPORTED(4),
       NEVER(5),
       NESTED(6);
   
       private final int value;
   
       private Propagation(int value) {
           this.value = value;
       }
   
       public int value() {
           return this.value;
       }
   }
   ```

   - REQUIRED：没有事务创建新事务，有则使用原有事务。
   - SUPPORTS：有事务使用原事务，没有则不使用事务执行。
   - MANDATORY：有事务使用原事务，没有则抛出异常
   - REQUIRED_NEW：新增一个平行的事务，挂起已存在的事务。一个事务的回滚不会引起另外个事务的回滚。
   - NOT_SUPPORTED：不以事务执行，挂起已有事务。
   - NEVER：不以事务执行，有事务则抛出异常。
   - NESTED：嵌套性事务。嵌套性事务是指在原有事务基础上嵌套一个子事务。子事务回滚并不会引起父事务回滚，而父事务回滚同时会引起子事务回滚。

3. @Transational事务不生效情况

   https://www.cnblogs.com/javastack/p/12160464.html

   - 数据库引擎不支持事务

   - 类没有被Spring管理

     ```java
     // @Service
     public class TestService {
     
         @Transactional
         public void Test(Order order) {
             // update order
         }
     
     }
     ```

     > 类未声明Service注解，即是未交给Spring管理

   - 方法不是public的

     官方文档中有一下描述：

     ```java
     Method visibility and @Transactional
     When you use proxies, you should apply the @Transactional annotation only to methods with
     public visibility. If you do annotate protected, private or package-visible methods with the
     @Transactional annotation, no error is raised, but the annotated method does not exhibit the
     configured transactional settings. If you need to annotate non-public methods, consider using
     AspectJ (described later).
     ```

   - 调用错误

     ```java
     @Service
     public class OrderServiceImpl implements OrderService {
     
         public void update(Order order) {
             updateOrder(order);
         }
     
         @Transactional
         public void updateOrder(Order order) {
             // update order
         }
     
     }
     ```

     > update方法中未声明@Transactional注解，updateOrder中声明的不会生效

     ```java
     @Service
     public class OrderServiceImpl implements OrderService {
     
         @Transactional
         public void update(Order order) {
             updateOrder(order);
         }
     
         @Transactional(propagation = Propagation.REQUIRES_NEW)
         public void updateOrder(Order order) {
             // update order
         }
     
     }
     ```

     > updateOrder方法中声明了生成新的并行事务

   

   - 数据源没有配置事务管理器

     ```java
     @Bean
     public PlatformTransactionManager transactionManager(DataSource dataSource) {
         return new DataSourceTransactionManager(dataSource);
     }
     ```

   - 不支持事务

     ```java
     @Service
     public class OrderServiceImpl implements OrderService {
     
         @Transactional
         public void update(Order order) {
             updateOrder(order);
         }
     
         @Transactional(propagation = Propagation.NOT_SUPPORTED)
         public void updateOrder(Order order) {
             // update order
         }
     
     }
     ```

     > updateOrder不会被纳入事务管理

   - 异常被抓走了

     ```java
     // @Service
     public class OrderServiceImpl implements OrderService {
     
         @Transactional
         public void updateOrder(Order order) {
             try {
                 // update order
             } catch {
     
             }
         }
     
     }
     ```

     > 异常被抓走导致事务无法回滚

   - 异常类型错误

     ```java
     // @Service
     public class OrderServiceImpl implements OrderService {
     
         @Transactional
         public void updateOrder(Order order) {
             try {
                 // update order
             } catch {
                 throw new Exception("更新错误");
             }
         }
     
     }
     ```

     @Transactional默认回滚的是RuntimeException，Exception是RuntimeException的父类，导致事务回滚不生效。需如下处理：

     ```java
     @Transactional(rollbackFor = Exception.class)
     ```

     

4. sql优化

5. zookeeper中ZAB协议和Raft协议

6. 接口怎么保证幂等性

7. 

8. 666





## 一般幂等技术方案有这几种:

- 查询操作
- 唯一索引
- token机制，防止重复提交
- 数据库的delete/update操作
- 乐观锁
- 悲观锁
- Redis、zookeeper 分布式锁（以前抢红包需求，用了Redis分布式锁）
- 状态机幂等