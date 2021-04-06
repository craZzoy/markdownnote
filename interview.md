

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



# mysql中的事务和spring中的事务



# HashMap(JDK8)相关原理初探



## 哈希表数据结构
首先了解下hashTable的数据结构



## 内部数据结构
hashmap的底层数据结构其实是数组加单向链表的形式
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
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and load factor.
     *
     * @param  initialCapacity the initial capacity
     * @param  loadFactor      the load factor
     * @throws IllegalArgumentException if the initial capacity is negative
     *         or the load factor is nonpositive
     */
     //initialCapacity初始化容量，
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
        this.threshold = tableSizeFor(initialCapacity);
    }

    /**
     * Constructs an empty <tt>HashMap</tt> with the specified initial
     * capacity and the default load factor (0.75).
     *
     * @param  initialCapacity the initial capacity.
     * @throws IllegalArgumentException if the initial capacity is negative.
     */
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
        else { //4、定位到的table数字位置数据不为空
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k)))) //4a、数组
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
        //5b、托容之后的数据移动操作
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
为什么是2^n？
因为此时cap-1的值低n位都是1，最终结果取决于hash值得低n位，这样能够使算出来的索引尽量分散，并且结果实在0至(2^n)-1之间。


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





