# hash算法的作用
为了Node节点落点的一个前戏运算
默认数组大小，位移运算速度快
```
 /**
     * The default initial capacity - MUST be a power of two.
     */
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
```
计算数组下标：
 1. 通过hash算法获得一个整数
 2. 控制在0-15之间

hash方法

作用：

# PUT的过程
 1. 计算hash(key)
```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```
 2. table为空，通过resize()方法初始化table
```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
            ....
    }
```
resize：
```java
else {               // zero initial threshold signifies using defaults
            newCap = DEFAULT_INITIAL_CAPACITY;
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
        }
```
初始化数组，长度默认16
```java
Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
```

 3. 根据key value组装Node节点
a.若数组下标位置数据为空，加入新节点
```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        //根据key value组装Node节点，找到数组下标位置数据为空，加入新节点
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        ......
    }
```

b.如果数组对应索引位置数据不为空，走else逻辑
```java
        final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            Node<K,V> e; K k;
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            else if (p instanceof TreeNode) //红黑树
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else { //链表
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        p.next = newNode(hash, key, value, null);
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);//链表提升为红黑树
                        break;
                    }
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        //拓容，threshold为托容因子DEFAULT_LOAD_FACTOR*DEFAULT_INITIAL_CAPACITY的值，即12。resize还可以用来拓容
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
(1)key值相同，把value值覆盖
(2)红黑树
不能让链表过长，当链表长度超过8时，链表转为红黑树形式
(3)链表的方式

拓容，2倍拓容，容量值为2的n次幂，为什么？保证hash()方法算出的结果尽量分散，怎么分散，结果取决于key的hashCode

对新的数组填充，即将原数组的部分数据移动到新数组中

 - 555
```java
        final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            //容量不为0时，进行双倍拓容
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else {               // zero initial threshold signifies using defaults
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
            Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
        table = newTab;
        //拓容之后的数据移动操作
        if (oldTab != null) {
            for (int j = 0; j < oldCap; ++j) {
                Node<K,V> e;
                if ((e = oldTab[j]) != null) { //数组某个索引位置有元素
                    oldTab[j] = null;
                    if (e.next == null) //只有单个元素
                        newTab[e.hash & (newCap - 1)] = e;
                    else if (e instanceof TreeNode) //红黑树结构
                        ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                    else { // preserve order //链表结构
                        Node<K,V> loHead = null, loTail = null;
                        Node<K,V> hiHead = null, hiTail = null;
                        Node<K,V> next;
                        do {
                            next = e.next;
                            if ((e.hash & oldCap) == 0) { //判断倒数第五位是否为0
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
链表中的数据自动后最终存在的结果（根据倒数第五位是否为0）
 - 原来位置
 - 原来位置+oldCup