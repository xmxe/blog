---
title: HashMap&ConcurrentHashMap
categories: Java
index_img: /assert/hashmap.jpg
img: https://pica.zhimg.com/v2-617196c5dc7927460c726d1477174464_1440w.jpg

---

## HashMap

### HashMap简介

HashMap主要用来存放键值对，它基于哈希表的Map接口实现，是常用的Java集合之一，是非线程安全的。HashMap可以存储null的key和value，但null作为键只能有一个，null作为值可以有多个。

JDK1.8之前HashMap由数组+链表组成的，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的（“拉链法”解决冲突）。JDK1.8以后的HashMap在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）（将链表转换成红黑树前会判断，如果当前数组的长度小于64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

HashMap默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。并且，HashMap总是使用2的幂作为哈希表的大小。

### HashMap的底层实现

#### JDK1.8之前

JDK1.8之前HashMap底层是**数组和链表**结合在一起使用也就是**链表散列**。HashMap通过key的hashcode经过扰动函数处理过后得到hash值，然后通过(n-1)&hash判断当前元素存放的位置（这里的n指的是数组的长度），如果当前位置存在元素的话，就判断该元素与要存入的元素的hash值以及key是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。所谓扰动函数指的就是HashMap的hash方法。使用hash方法也就是扰动函数是为了防止一些实现比较差的hashCode()方法,换句话说使用扰动函数之后可以减少碰撞。

**JDK1.8HashMap的hash方法源码**：

JDK1.8的hash方法相比于JDK1.7hash方法更加简化，但是原理不变。
```java
static final int hash(Object key) {
    int h;
    // key.hashCode()：返回散列值也就是hashcode
    // ^ ：按位异或
    // >>>:无符号右移，忽略符号位，空位都以0补齐
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
```

对比一下JDK1.7的HashMap的hash方法源码.

```java
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

相比于JDK1.8的hash方法，JDK1.7的hash方法的性能会稍差一点点，因为毕竟扰动了4次。
所谓**拉链法**就是：将链表和数组相结合。也就是说创建一个链表数组，数组中每一格就是一个链表。若遇到哈希冲突，则将冲突的值加到链表中即可。

![jdk1.8之前的内部结构-HashMap](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/jdk1.7_hashmap.png)

#### JDK1.8之后

相比于之前的版本，JDK1.8之后在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）（将链表转换成红黑树前会判断，如果当前数组的长度小于64，那么会选择先进行数组扩容，而不是转换为红黑树）时，将链表转化为红黑树，以减少搜索时间。

![jdk1.8之后的内部结构-HashMap](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/jdk1.8_hashmap.png)

> TreeMap、TreeSet以及JDK1.8之后的HashMap底层都用到了红黑树。红黑树就是为了解决二叉查找树的缺陷，因为二叉查找树在某些情况下会退化成一个线性结构。

我们来结合源码分析一下HashMap链表到红黑树的转换。

**1、putVal方法中执行链表转红黑树的判断逻辑。**

链表的长度大于8的时候，就执行treeifyBin（转换红黑树）的逻辑。

```java
// 遍历链表
for (int binCount = 0; ; ++binCount) {
    // 遍历到链表最后一个节点
    if ((e = p.next) == null) {
        p.next = newNode(hash, key, value, null);
        // 如果链表元素个数大于等于TREEIFY_THRESHOLD（8）
        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
            // 红黑树转换（并不会直接转换成红黑树）
            treeifyBin(tab, hash);
        break;
    }
    if (e.hash == hash &&
        ((k = e.key) == key || (key != null && key.equals(k))))
        break;
    p = e;
}
```

**2、treeifyBin方法中判断是否真的转换为红黑树。**

```java
final void treeifyBin(Node<K,V>[] tab, int hash) {
    int n, index; Node<K,V> e;
    // 判断当前数组的长度是否小于 64
    if (tab == null || (n = tab.length) < MIN_TREEIFY_CAPACITY)
        // 如果当前数组的长度小于 64，那么会选择先进行数组扩容
        resize();
    else if ((e = tab[index = (n - 1) & hash]) != null) {
        // 否则才将列表转换为红黑树

        TreeNode<K,V> hd = null, tl = null;
        do {
            TreeNode<K,V> p = replacementTreeNode(e, null);
            if (tl == null)
                hd = p;
            else {
                p.prev = tl;
                tl.next = p;
            }
            tl = p;
        } while ((e = e.next) != null);
        if ((tab[index] = hd) != null)
            hd.treeify(tab);
    }
}
```

将链表转换成红黑树前会判断，如果当前数组的长度小于64，那么会选择先进行数组扩容，而不是转换为红黑树。当链表长度大于阈值（默认为8）时，会首先调用treeifyBin()方法。这个方法会根据HashMap数组来决定是否转换为红黑树。只有当数组长度大于或者等于64的情况下，才会执行转换红黑树操作，以减少搜索时间。否则，就是只是执行resize()方法对数组扩容。相关源码这里就不贴了，重点关注treeifyBin()方法即可！

![img](https://oscimg.oschina.net/oscnet/up-bba283228693dae74e78da1ef7a9a04c684.png)

**类的属性**：

```java
public class HashMap<K,V> extends AbstractMap<K,V> implements Map<K,V>, Cloneable, Serializable {
    // 序列号
    private static final long serialVersionUID = 362498820763181265L;
    // 默认的初始容量是16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4;
    // 最大容量
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认的填充因子
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 当桶(bucket)上的结点数大于这个值时会转成红黑树
    static final int TREEIFY_THRESHOLD = 8;
    // 当桶(bucket)上的结点数小于这个值时树转链表
    static final int UNTREEIFY_THRESHOLD = 6;
    // 桶中结构转化为红黑树对应的table的最小容量
    static final int MIN_TREEIFY_CAPACITY = 64;
    // 存储元素的数组，总是2的幂次倍
    transient Node<k,v>[] table;
    // 存放具体元素的集
    transient Set<map.entry<k,v>> entrySet;
    // 存放元素的个数，注意这个不等于数组的长度。
    transient int size;
    // 每次扩容和更改map结构的计数器
    transient int modCount;
    // 临界值(容量*填充因子) 当实际大小超过临界值时，会进行扩容
    int threshold;
    // 加载因子
    final float loadFactor;
}
```

- **loadFactor加载因子**
loadFactor加载因子是控制数组存放数据的疏密程度，loadFactor越趋近于1，那么数组中存放的数据(entry)也就越多，也就越密，也就是会让链表的长度增加，loadFactor越小，也就是趋近于0，数组中存放的数据(entry)也就越少，也就越稀疏。**loadFactor太大导致查找元素效率低，太小导致数组的利用率低，存放的数据会很分散。loadFactor的默认值为0.75f是官方给出的一个比较好的临界值**。给定的默认容量为16，负载因子为0.75。Map在使用过程中不断的往里面存放数据，当数量达到了16*0.75=12就需要将当前16的容量进行扩容，而扩容这个过程涉及到rehash、复制数据等操作，所以非常消耗性能。

- **threshold**
**threshold=capacity\*loadFactor**，**当Size>=threshold**的时候，那么就要考虑对数组的扩增了，也就是说，这个的意思就是衡量数组是否需要扩增的一个标准。

**Node节点类源码:**

```java
// 继承自Map.Entry<K,V>
static class Node<K,V> implements Map.Entry<K,V> {
    final int hash;// 哈希值，存放元素到hashmap中时用来与其他元素hash值比较
    final K key;//键
    V value;//值
    // 指向下一个节点
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
    // 重写hashCode()方法
    public final int hashCode() {
        return Objects.hashCode(key) ^ Objects.hashCode(value);
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }
    // 重写equals()方法
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

**树节点类源码:**

```java
static final class TreeNode<K,V> extends LinkedHashMap.Entry<K,V> {
    TreeNode<K,V> parent;  // 父
    TreeNode<K,V> left;    // 左
    TreeNode<K,V> right;   // 右
    TreeNode<K,V> prev;    // needed to unlink next upon deletion
    boolean red;           // 判断颜色
    TreeNode(int hash, K key, V val, Node<K,V> next) {
        super(hash, key, val, next);
    }
    // 返回根节点
    final TreeNode<K,V> root() {
        for (TreeNode<K,V> r = this, p;;) {
            if ((p = r.parent) == null)
                return r;
            r = p;
        }
    }
}
```

### HashMap源码分析

#### 构造方法

HashMap中有四个构造方法，它们分别如下：
```java
// 默认构造函数。
public HashMap() {
    this.loadFactor = DEFAULT_LOAD_FACTOR; // all   other fields defaulted
}

// 包含另一个“Map”的构造函数
public HashMap(Map<? extends K, ? extends V> m) {
    this.loadFactor = DEFAULT_LOAD_FACTOR;
    putMapEntries(m, false);//下面会分析到这个方法
}

// 指定“容量大小”的构造函数
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}

// 指定“容量大小”和“加载因子”的构造函数
public HashMap(int initialCapacity, float loadFactor) {
    if (initialCapacity < 0)
        throw new IllegalArgumentException("Illegal initial capacity: " + initialCapacity);
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    if (loadFactor <= 0 || Float.isNaN(loadFactor))
        throw new IllegalArgumentException("Illegal load factor: " + loadFactor);
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

**putMapEntries方法：**
```java
final void putMapEntries(Map<? extends K, ? extends V> m, boolean evict) {
    int s = m.size();
    if (s > 0) {
        // 判断table是否已经初始化
        if (table == null) { // pre-size
            // 未初始化，s为m的实际元素个数
            float ft = ((float)s / loadFactor) + 1.0F;
            int t = ((ft < (float)MAXIMUM_CAPACITY) ?
                    (int)ft : MAXIMUM_CAPACITY);
            // 计算得到的t大于阈值，则初始化阈值
            if (t > threshold)
                threshold = tableSizeFor(t);
        }
        // 已初始化，并且m元素个数大于阈值，进行扩容处理
        else if (s > threshold)
            resize();
        // 将m中的所有元素添加至HashMap中
        for (Map.Entry<? extends K, ? extends V> e : m.entrySet()) {
            K key = e.getKey();
            V value = e.getValue();
            putVal(hash(key), key, value, false, evict);
        }
    }
}
```

#### put方法

HashMap只提供了put用于添加元素，putVal方法只是给put方法调用的一个方法，并没有提供给用户使用。

**对putVal方法添加元素的分析如下**：

1. 如果定位到的数组位置没有元素就直接插入。
2. 如果定位到的数组位置有元素就和要插入的key比较，如果key相同就直接覆盖，如果key不相同，就判断p是否是一个树节点，如果是就调用`e = ((TreeNode<K,V>)p).putTreeVal(this,tab,hash,key,value)`将元素添加进入。如果不是就遍历链表插入(插入的是链表尾部)。

![](https://my-blog-to-use.oss-cn-beijing.aliyuncs.com/2019-7/put方法.png)

说明:上图有两个小问题：

- 直接覆盖之后应该就会return，不会有后续操作。参考JDK8 HashMap.java658行（[issue#608](https://github.com/Snailclimb/JavaGuide/issues/608)）。
- 当链表长度大于阈值（默认为8）并且HashMap数组长度超过64的时候才会执行链表转红黑树的操作，否则就只是对数组扩容。参考HashMap的treeifyBin()方法（[issue#1087](https://github.com/Snailclimb/JavaGuide/issues/1087)）。

```java
public V put(K key, V value) {
    return putVal(hash(key), key, value, false, true);
}

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,boolean evict) {
    Node<K,V>[] tab; Node<K,V> p; int n, i;
    // table未初始化或者长度为0，进行扩容
    if ((tab = table) == null || (n = tab.length) == 0)
        n = (tab = resize()).length;
    // (n - 1) & hash 确定元素存放在哪个桶中，桶为空，新生成结点放入桶中(此时，这个结点是放在数组中)
    if ((p = tab[i = (n - 1) & hash]) == null)
        tab[i] = newNode(hash, key, value, null);
    // 桶中已经存在元素（处理hash冲突）
    else {
        Node<K,V> e; K k;
        // 判断table[i]中的元素是否与插入的key一样，若相同那就直接使用插入的值p替换掉旧的值e。
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
        // 判断插入的是否是红黑树节点
        else if (p instanceof TreeNode)
            // 放入树中
            e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
        // 不是红黑树节点则说明为链表结点
        else {
            // 在链表最末插入结点
            for (int binCount = 0; ; ++binCount) {
                // 到达链表的尾部
                if ((e = p.next) == null) {
                    // 在尾部插入新结点
                    p.next = newNode(hash, key, value, null);
                    // 结点数量达到阈值(默认为 8 )，执行 treeifyBin 方法
                    // 这个方法会根据 HashMap 数组来决定是否转换为红黑树。
                    // 只有当数组长度大于或者等于 64 的情况下，才会执行转换红黑树操作，以减少搜索时间。否则，就是只是对数组扩容。
                    if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                        treeifyBin(tab, hash);
                    // 跳出循环
                    break;
                }
                // 判断链表中结点的key值与插入的元素的key值是否相等
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    // 相等，跳出循环
                    break;
                // 用于遍历桶中的链表，与前面的e = p.next组合，可以遍历链表
                p = e;
            }
        }
        // 表示在桶中找到key值、hash值与插入元素相等的结点
        if (e != null) {
            // 记录e的value
            V oldValue = e.value;
            // onlyIfAbsent为false或者旧值为null
            if (!onlyIfAbsent || oldValue == null)
                //用新值替换旧值
                e.value = value;
            // 访问后回调
            afterNodeAccess(e);
            // 返回旧值
            return oldValue;
        }
    }
    // 结构性修改
    ++modCount;
    // 实际大小大于阈值则扩容
    if (++size > threshold)
        resize();
    // 插入后回调
    afterNodeInsertion(evict);
    return null;
}
```

**我们再来对比一下JDK1.7put方法的代码，对于put方法的分析如下**：

①如果定位到的数组位置没有元素就直接插入。
②如果定位到的数组位置有元素，遍历以这个元素为头结点的链表，依次和插入的key比较，如果key相同就直接覆盖，不同就采用头插法插入元素。

```java
public V put(K key, V value)
    if (table == EMPTY_TABLE) {
    	inflateTable(threshold);
	}
    if (key == null)
        return putForNullKey(value);
    int hash = hash(key);
    int i = indexFor(hash, table.length);
    for (Entry<K,V> e = table[i]; e != null; e = e.next) { // 先遍历
        Object k;
        if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
            V oldValue = e.value;
            e.value = value;
            e.recordAccess(this);
            return oldValue;
        }
    }

    modCount++;
    addEntry(hash, key, value, i);  // 再插入
    return null;
}
```

#### get方法

```java
public V get(Object key) {
    Node<K,V> e;
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (first = tab[(n - 1) & hash]) != null) {
        // 数组元素相等
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        // 桶中不止一个节点
        if ((e = first.next) != null) {
            // 在树中get
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 在链表中get
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

#### resize方法

进行扩容，会伴随着一次重新hash分配，并且会遍历hash表中所有的元素，是非常耗时的。在编写程序中，要尽量避免resize。

```java
final Node<K,V>[] resize() {
    Node<K,V>[] oldTab = table;
    int oldCap = (oldTab == null) ? 0 : oldTab.length;
    int oldThr = threshold;
    int newCap, newThr = 0;
    if (oldCap > 0) {
        // 超过最大值就不再扩充了，就只好随你碰撞去吧
        if (oldCap >= MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return oldTab;
        }
        // 没超过最大值，就扩充为原来的2倍
        else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY && oldCap >= DEFAULT_INITIAL_CAPACITY)
            newThr = oldThr << 1; // double threshold
    }
    else if (oldThr > 0) // initial capacity was placed in threshold
        newCap = oldThr;
    else {
        // signifies using defaults
        newCap = DEFAULT_INITIAL_CAPACITY;
        newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);
    }
    // 计算新的resize上限
    if (newThr == 0) {
        float ft = (float)newCap * loadFactor;
        newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ? (int)ft : Integer.MAX_VALUE);
    }
    threshold = newThr;
    @SuppressWarnings({"rawtypes","unchecked"})
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
    if (oldTab != null) {
        // 把每个bucket都移动到新的buckets中
        for (int j = 0; j < oldCap; ++j) {
            Node<K,V> e;
            if ((e = oldTab[j]) != null) {
                oldTab[j] = null;
                if (e.next == null)
                    newTab[e.hash & (newCap - 1)] = e;
                else if (e instanceof TreeNode)
                    ((TreeNode<K,V>)e).split(this, newTab, j, oldCap);
                else {
                    Node<K,V> loHead = null, loTail = null;
                    Node<K,V> hiHead = null, hiTail = null;
                    Node<K,V> next;
                    do {
                        next = e.next;
                        // 原索引
                        if ((e.hash & oldCap) == 0) {
                            if (loTail == null)
                                loHead = e;
                            else
                                loTail.next = e;
                            loTail = e;
                        }
                        // 原索引+oldCap
                        else {
                            if (hiTail == null)
                                hiHead = e;
                            else
                                hiTail.next = e;
                            hiTail = e;
                        }
                    } while ((e = next) != null);
                    // 原索引放到bucket里
                    if (loTail != null) {
                        loTail.next = null;
                        newTab[j] = loHead;
                    }
                    // 原索引+oldCap放到bucket里
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

### HashMap常用方法测试

```java
package map;

import java.util.Collection;
import java.util.HashMap;
import java.util.Set;

public class HashMapDemo {
    public static void main(String[] args) {
        HashMap<String, String> map = new HashMap<String, String>();
        // 键不能重复，值可以重复
        map.put("san", "张三");
        map.put("si", "李四");
        map.put("wu", "王五");
        map.put("wang", "老王");
        map.put("wang", "老王2");// 老王被覆盖
        map.put("lao", "老王");
        System.out.println("-------直接输出hashmap:-------");
        System.out.println(map);
        /**
         * 遍历HashMap
         */
        // 1.获取Map中的所有键
        System.out.println("-------foreach获取Map中所有的键:------");
        Set<String> keys = map.keySet();
        for (String key : keys) {
            System.out.print(key+"  ");
        }
        System.out.println();//换行
        // 2.获取Map中所有值
        System.out.println("-------foreach获取Map中所有的值:------");
        Collection<String> values = map.values();
        for (String value : values) {
            System.out.print(value+"  ");
        }
        System.out.println();//换行
        // 3.得到key的值的同时得到key所对应的值
        System.out.println("-------得到key的值的同时得到key所对应的值:-------");
        Set<String> keys2 = map.keySet();
        for (String key : keys2) {
            System.out.print(key + "：" + map.get(key)+"   ");

        }
        /**
         * 如果既要遍历key又要value，那么建议这种方式，因为如果先获取keySet然后再执行map.get(key)，map内部会执行两次遍历。
         * 一次是在获取keySet的时候，一次是在遍历所有key的时候。
         */
        // 当我调用put(key,value)方法的时候，首先会把key和value封装到
        // Entry这个静态内部类对象中，把Entry对象再添加到数组中，所以我们想获取
        // map中的所有键值对，我们只要获取数组中的所有Entry对象，接下来
        // 调用Entry对象中的getKey()和getValue()方法就能获取键值对了
        Set<java.util.Map.Entry<String, String>> entrys = map.entrySet();
        for (java.util.Map.Entry<String, String> entry : entrys) {
            System.out.println(entry.getKey() + "--" + entry.getValue());
        }

        /**
         * HashMap其他常用方法
         */
        System.out.println("after map.size()："+map.size());
        System.out.println("after map.isEmpty()："+map.isEmpty());
        System.out.println(map.remove("san"));
        System.out.println("after map.remove()："+map);
        System.out.println("after map.get(si)："+map.get("si"));
        System.out.println("after map.containsKey(si)："+map.containsKey("si"));
        System.out.println("after containsValue(李四)："+map.containsValue("李四"));
        System.out.println(map.replace("si", "李四2"));
        System.out.println("after map.replace(si, 李四2):"+map);
    }

}
```

### HashMap常见问题

#### HashMap的长度为什么是2的幂次方

答：HashMap的默认长度为16和规定数组长度为2的幂,是为了降低hash碰撞的几率

为了能让HashMap存取高效，尽量较少碰撞，也就是要尽量把数据分配均匀。我们上面也讲到了过了，Hash值的范围值-2147483648到2147483647，前后加起来大概40亿的映射空间，只要哈希函数映射得比较均匀松散，一般应用是很难出现碰撞的。但问题是一个40亿长度的数组，内存是放不下的。所以这个散列值是不能直接拿来用的。用之前还要先做对数组的长度取模运算，得到的余数才能用来要存放的位置也就是对应的数组下标。这个数组下标的计算方法是(n - 1) & hash。（n代表数组长度）。这也就解释了HashMap的长度为什么是2的幂次方。

**这个算法应该如何设计呢？**

我们首先可能会想到采用%取余的操作来实现。但是，重点来了：

> “取余(%)操作中如果除数是2的幂次则等价于与其除数减一的与(&)操作（也就是说hash%length==hash&(length-1)的前提是length是2的n次方；）”。并且采用二进制位操作&，相对于%能够提高运算效率，这就解释了HashMap的长度为什么是2的幂次方。

#### 加载因子为什么0.75，而不是其他值？
答：可以说是一个经过考量的经验值。加载因子涉及扩容，下次扩容的阈值=数组桶的大小\*加载因子，如果加载因子太小，这就会导致阈值太小，这就会导致比较容易发生扩容。如果加载因子太大，那就会导致阈值太大，可能冲突会很多，导致查找效率下降。负载因子为什么是0.75，如果负载因子为0.5甚至更低的可能的话，最后得到的临时阈值明显会很小，这样的情况就会造成分配的内存的浪费，存在多余的没用的内存空间，也不满足了哈希表均匀分布的情况。如果负载因子达到了1的情况，也就是Entry数组存满了才发生扩容，这样会出现大量的哈希冲突的情况，出现链表过长，因此造成get查询数据的效率。因此选择了0.5~1的折中数也就是0.75，均衡解决了上面出现的情况。
> [面试官竟然问我为啥HashMap的负载因子不设置成1！](https://mp.weixin.qq.com/s/kbLASf0lcF4PDJ3qBsFyUg)
> [面试官：为什么HashMap的加载因子是0.75？](https://mp.weixin.qq.com/s/ZxwU2qSXvdEZVAIbY_5EPQ)

#### 为什么不能将实数作为HashMap的key？

答：java中浮点数的表示比较复杂，特别是牵涉到-0.0,NaN,正负无穷这种，所以不适宜用来作为Map的key。

#### HashMap多线程操作导致死循环问题

答：准确的讲应该是JDK1.7的HashMap链表会有死循环的可能，因为JDK1.7是采用的头插法，在多线程环境下有可能会使链表形成环状，从而导致死循环。JDK1.8做了改进，用的是尾插法，不会产生死循环。
我们从`put()`方法开始，最终找到线程不安全的那个方法。这里省略中间不重要的过程，我只把方法的跳转流程贴出来：
> //添加元素方法->添加新节点方法->扩容方法->把原数组元素重新分配到新数组中
> put()-->addEntry()-->resize()-->transfer()

现在，有两个线程都执行transfer方法。每个线程都会在它们自己的工作内存生成一个newTable的数组，用于存储变化后的链表，它们互不影响（这里互不影响，指的是两个新数组本身互不影响）。但是，需要注意的是，它们操作的数据却是同一份。一番扩容操作后出现环形链表，这时，有的同学可能就会问了，就算他们成环了，又怎样，跟死循环有什么关系？我们看下get()方法（最终调用getEntry方法），可以看到查找元素时，只要e不为空，就会一直循环查找下去。若有某个元素C的hash值也落在了和A，B元素同一个桶中，则会由于，A，B互相指向，e.next永远不为空，就会形成死循环。主要原因在于并发下的Rehash会造成元素之间会形成一个循环链表。不过，jdk1.8后解决了这个问题，但是还是不建议在多线程下使用HashMap,因为多线程下使用HashMap还是会存在其他问题比如数据丢失。并发环境下推荐使用ConcurrentHashMap。

| [JAVA HASHMAP的死循环](https://coolshell.cn/articles/9606.html) | [美团面试题：HashMap是如何形成死循环的？（最完整的配图讲解）](https://mp.weixin.qq.com/s/5FdDjDo5H-nDZhFxo7H6QQ) | [多线程环境下，HashMap为什么会出现死循环？](https://mp.weixin.qq.com/s/gAw9K6yd-w9ZyP90xyvTwg) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| [听说过HashMap会导致CPU飙升100%吗？](https://mp.weixin.qq.com/s/-IvCgU8fC0Au21BPxPyo-g) |                                                              |                                                              |

#### HashMap有哪几种常见的遍历方式?

- [HashMap的7种遍历方式与性能分析](https://mp.weixin.qq.com/s/zQBN3UvJDhRTKP6SzcZFKw)

> [原文链接](https://javaguide.cn/java/collection/java-collection-questions-02.html)

### HashMap相关文章

| [面试官再问你HashMap底层原理，就把这篇文章甩给他看](https://mp.weixin.qq.com/s/8Nl9dv_ywubW7Wc45--pgw) | [HashMap面试21问，这次要跪了！](https://mp.weixin.qq.com/s/WyPnPAKZfA58eX7qSBcP8Q) | [为什么HashMap不能一边遍历一边删除](https://mp.weixin.qq.com/s/JZiSfe7IIIJCstTucH5tlw) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |


## ConcurrentHashMap

jdk1.7 ConcurrentHashMap类所采用的是分段锁的思想，将HashMap进行切割，把HashMap中的哈希数组切分成小数组，这个小数组名叫Segment,每个Segment有n个HashEntry组成，其中Segment继承自ReentrantLock（可重入锁）,在操作的时候给Segment赋予了一个对象锁，从而保证多线程环境下并发操作安全。

JDK1.8对HashMap做了改造，当冲突链表长度大于8时，会将链表转变成红黑树结构,JDK1.8中ConcurrentHashMap类取消了Segment分段锁，采用CAS+synchronized来保证并发安全，数据结构跟jdk1.8中HashMap结构类似，都是数组+链表（当链表长度大于8时，链表结构转为红黑二叉树）结构。ConcurrentHashMap中synchronized只锁定当前链表或红黑二叉树的首节点，只要节点hash不冲突，就不会产生并发，相比JDK1.7的ConcurrentHashMap效率又提升了N倍！

### ConcurrentHashMap源码&底层数据结构分析

#### ConcurrentHashMap 1.7

##### 1.存储结构

![Java7 ConcurrentHashMap存储结构](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/java7_concurrenthashmap.png)

Java7中ConcurrentHashMap的存储结构如上图，ConcurrnetHashMap由很多个Segment组合，而每一个Segment是一个类似于HashMap的结构，所以每一个HashMap的内部可以进行扩容。但是Segment的个数一旦**初始化就不能改变**，默认Segment的个数是16个，你也可以认为ConcurrentHashMap默认支持最多16个线程并发。

##### 2.初始化

通过ConcurrentHashMap的无参构造探寻ConcurrentHashMap的初始化流程。

```java
/**
  * Creates a new, empty map with a default initial capacity (16),
  * load factor (0.75) and concurrencyLevel (16).
  */
public ConcurrentHashMap() {
    this(DEFAULT_INITIAL_CAPACITY, DEFAULT_LOAD_FACTOR, DEFAULT_CONCURRENCY_LEVEL);
}
```

无参构造中调用了有参构造，传入了三个参数的默认值，他们的值是。

```java
/**
  * 默认初始化容量
  */
static final int DEFAULT_INITIAL_CAPACITY = 16;

/**
  * 默认负载因子
  */
static final float DEFAULT_LOAD_FACTOR = 0.75f;

/**
  * 默认并发级别
  */
static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```

接着看下这个有参构造函数的内部实现逻辑。

```java
@SuppressWarnings("unchecked")
public ConcurrentHashMap(int initialCapacity,float loadFactor, int concurrencyLevel) {
    // 参数校验
    if (!(loadFactor > 0) || initialCapacity < 0 || concurrencyLevel <= 0)
        throw new IllegalArgumentException();
    // 校验并发级别大小，大于1<<16，重置为65536
    if (concurrencyLevel > MAX_SEGMENTS)
        concurrencyLevel = MAX_SEGMENTS;
    // Find power-of-two sizes best matching arguments
    // 2的多少次方
    int sshift = 0;
    int ssize = 1;
    // 这个循环可以找到concurrencyLevel之上最近的2的次方值
    while (ssize < concurrencyLevel) {
        ++sshift;
        ssize <<= 1;
    }
    // 记录段偏移量
    this.segmentShift = 32 - sshift;
    // 记录段掩码
    this.segmentMask = ssize - 1;
    // 设置容量
    if (initialCapacity > MAXIMUM_CAPACITY)
        initialCapacity = MAXIMUM_CAPACITY;
    // c=容量/ssize，默认16/16=1，这里是计算每个Segment中的类似于HashMap的容量
    int c = initialCapacity / ssize;
    if (c * ssize < initialCapacity)
        ++c;
    int cap = MIN_SEGMENT_TABLE_CAPACITY;
    //Segment中的类似于HashMap的容量至少是2或者2的倍数
    while (cap < c)
        cap <<= 1;
    // create segments and segments[0]
    // 创建Segment数组，设置segments[0]
    Segment<K,V> s0 = new Segment<K,V>(loadFactor, (int)(cap * loadFactor),
                         (HashEntry<K,V>[])new HashEntry[cap]);
    Segment<K,V>[] ss = (Segment<K,V>[])new Segment[ssize];
    UNSAFE.putOrderedObject(ss, SBASE, s0); // ordered write of segments[0]
    this.segments = ss;
}
```

总结一下在Java7中ConcurrnetHashMap的初始化逻辑。

1. 必要参数校验。
2. 校验并发级别concurrencyLevel大小，如果大于最大值，重置为最大值。无参构造**默认值是16**。
3. 寻找并发级别concurrencyLevel之上最近的**2的幂次方**值，作为初始化容量大小，**默认是16**。
4. 记录segmentShift偏移量，这个值为【容量=2的N次方】中的N，在后面Put时计算位置时会用到。**默认是32-sshift=28**。
5. 记录segmentMask，默认是ssize-1=16-1=15.
6. **初始化segments[0]**，**默认大小为2**，**负载因子0.75**，**扩容阀值是2\*0.75=1.5**，插入第二个值时才会进行扩容。

##### 3.put

接着上面的初始化参数继续查看put方法源码。

```java
/**
 * Maps the specified key to the specified value in this table.
 * Neither the key nor the value can be null.
 *
 * <p> The value can be retrieved by calling the <tt>get</tt> method
 * with a key that is equal to the original key.
 *
 * @param key key with which the specified value is to be associated
 * @param value value to be associated with the specified key
 * @return the previous value associated with <tt>key</tt>, or
 *         <tt>null</tt> if there was no mapping for <tt>key</tt>
 * @throws NullPointerException if the specified key or value is null
 */
public V put(K key, V value) {
    Segment<K,V> s;
    if (value == null)
        throw new NullPointerException();
    int hash = hash(key);
    // hash值无符号右移28位（初始化时获得），然后与segmentMask=15做与运算
    // 其实也就是把高4位与segmentMask（1111）做与运算
    int j = (hash >>> segmentShift) & segmentMask;
    if ((s = (Segment<K,V>)UNSAFE.getObject          // nonvolatile; recheck
         (segments, (j << SSHIFT) + SBASE)) == null) //  in ensureSegment
        // 如果查找到的Segment为空，初始化
        s = ensureSegment(j);
    return s.put(key, hash, value, false);
}

/**
 * Returns the segment for the given index, creating it and
 * recording in segment table (via CAS) if not already present.
 *
 * @param k the index
 * @return the segment
 */
@SuppressWarnings("unchecked")
private Segment<K,V> ensureSegment(int k) {
    final Segment<K,V>[] ss = this.segments;
    long u = (k << SSHIFT) + SBASE; // raw offset
    Segment<K,V> seg;
    // 判断u位置的Segment是否为null
    if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) {
        Segment<K,V> proto = ss[0]; // use segment 0 as prototype
        // 获取0号segment里的HashEntry<K,V>初始化长度
        int cap = proto.table.length;
        // 获取0号segment里的hash表里的扩容负载因子，所有的segment的loadFactor是相同的
        float lf = proto.loadFactor;
        // 计算扩容阀值
        int threshold = (int)(cap * lf);
        // 创建一个cap容量的HashEntry数组
        HashEntry<K,V>[] tab = (HashEntry<K,V>[])new HashEntry[cap];
        if ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u)) == null) { // recheck
            // 再次检查u位置的Segment是否为null，因为这时可能有其他线程进行了操作
            Segment<K,V> s = new Segment<K,V>(lf, threshold, tab);
            // 自旋检查u位置的Segment是否为null
            while ((seg = (Segment<K,V>)UNSAFE.getObjectVolatile(ss, u))
                   == null) {
                // 使用CAS赋值，只会成功一次
                if (UNSAFE.compareAndSwapObject(ss, u, null, seg = s))
                    break;
            }
        }
    }
    return seg;
}
```

上面的源码分析了ConcurrentHashMap在put一个数据时的处理流程，下面梳理下具体流程。

1. 计算要put的key的位置，获取指定位置的Segment。

2. 如果指定位置的Segment为空，则初始化这个Segment。**初始化Segment流程**：

   1. 检查计算得到的位置的Segment是否为null.
   2. 为null继续初始化，使用Segment[0]的容量和负载因子创建一个HashEntry数组。
   3. 再次检查计算得到的指定位置的Segment是否为null.
   4. 使用创建的HashEntry数组初始化这个Segment.
   5. 自旋判断计算得到的指定位置的Segment是否为null，使用CAS在这个位置赋值为Segment.
   
3. Segment.put插入key,value值。

上面探究了获取Segment段和初始化Segment段的操作。最后一行的Segment的put方法还没有查看，继续分析。

```java
final V put(K key, int hash, V value, boolean onlyIfAbsent) {
    // 获取ReentrantLock独占锁，获取不到，scanAndLockForPut获取。
    HashEntry<K,V> node = tryLock() ? null : scanAndLockForPut(key, hash, value);
    V oldValue;
    try {
        HashEntry<K,V>[] tab = table;
        // 计算要put的数据位置
        int index = (tab.length - 1) & hash;
        // CAS获取index坐标的值
        HashEntry<K,V> first = entryAt(tab, index);
        for (HashEntry<K,V> e = first;;) {
            if (e != null) {
                // 检查是否key已经存在，如果存在，则遍历链表寻找位置，找到后替换value
                K k;
                if ((k = e.key) == key ||
                    (e.hash == hash && key.equals(k))) {
                    oldValue = e.value;
                    if (!onlyIfAbsent) {
                        e.value = value;
                        ++modCount;
                    }
                    break;
                }
                e = e.next;
            }
            else {
                // first有值没说明index位置已经有值了，有冲突，链表头插法。
                if (node != null)
                    node.setNext(first);
                else
                    node = new HashEntry<K,V>(hash, key, value, first);
                int c = count + 1;
                // 容量大于扩容阀值，小于最大容量，进行扩容
                if (c > threshold && tab.length < MAXIMUM_CAPACITY)
                    rehash(node);
                else
                    // index位置赋值node，node可能是一个元素，也可能是一个链表的表头
                    setEntryAt(tab, index, node);
                ++modCount;
                count = c;
                oldValue = null;
                break;
            }
        }
    } finally {
        unlock();
    }
    return oldValue;
}
```

由于Segment继承了ReentrantLock，所以Segment内部可以很方便的获取锁，put流程就用到了这个功能。

1. tryLock()获取锁，获取不到使用**scanAndLockForPut**方法继续获取。
2. 计算put的数据要放入的index位置，然后获取这个位置上的HashEntry。
3. 遍历put新元素，为什么要遍历？因为这里获取的HashEntry可能是一个空元素，也可能是链表已存在，所以要区别对待。如果这个位置上的**HashEntry不存在**：

   1. 如果当前容量大于扩容阀值，小于最大容量，**进行扩容**。
   2. 直接头插法插入。
   
   如果这个位置上的**HashEntry存在**：

   1. 判断链表当前元素key和hash值是否和要put的key和hash值一致。一致则替换值
   2. 不一致，获取链表下一个节点，直到发现相同进行值替换，或者链表表里完毕没有相同的。
      1. 如果当前容量大于扩容阀值，小于最大容量，**进行扩容**。
      2. 直接链表头插法插入。
   
4. 如果要插入的位置之前已经存在，替换后返回旧值，否则返回null。

这里面的第一步中的scanAndLockForPut操作这里没有介绍，这个方法做的操作就是不断的自旋tryLock()获取锁。当自旋次数大于指定次数时，使用lock()阻塞获取锁。在自旋时顺表获取下hash位置的HashEntry。

```java
private HashEntry<K,V> scanAndLockForPut(K key, int hash, V value) {
    HashEntry<K,V> first = entryForHash(this, hash);
    HashEntry<K,V> e = first;
    HashEntry<K,V> node = null;
    int retries = -1; // negative while locating node
    // 自旋获取锁
    while (!tryLock()) {
        HashEntry<K,V> f; // to recheck first below
        if (retries < 0) {
            if (e == null) {
                if (node == null) // speculatively create node
                    node = new HashEntry<K,V>(hash, key, value, null);
                retries = 0;
            }
            else if (key.equals(e.key))
                retries = 0;
            else
                e = e.next;
        }
        else if (++retries > MAX_SCAN_RETRIES) {
            // 自旋达到指定次数后，阻塞等到只到获取到锁
            lock();
            break;
        }
        else if ((retries & 1) == 0 &&
                 (f = entryForHash(this, hash)) != first) {
            e = first = f; // re-traverse if entry changed
            retries = -1;
        }
    }
    return node;
}
```

##### 4.扩容rehash

ConcurrentHashMap的扩容只会扩容到原来的两倍。老数组里的数据移动到新的数组时，位置要么不变，要么变为index+oldSize，参数里的node会在扩容之后使用链表**头插法**插入到指定位置。

```java
private void rehash(HashEntry<K,V> node) {
    HashEntry<K,V>[] oldTable = table;
    // 老容量
    int oldCapacity = oldTable.length;
    // 新容量，扩大两倍
    int newCapacity = oldCapacity << 1;
    // 新的扩容阀值 
    threshold = (int)(newCapacity * loadFactor);
    // 创建新的数组
    HashEntry<K,V>[] newTable = (HashEntry<K,V>[]) new HashEntry[newCapacity];
    // 新的掩码，默认2扩容后是4，-1是3，二进制就是11。
    int sizeMask = newCapacity - 1;
    for (int i = 0; i < oldCapacity ; i++) {
        // 遍历老数组
        HashEntry<K,V> e = oldTable[i];
        if (e != null) {
            HashEntry<K,V> next = e.next;
            // 计算新的位置，新的位置只可能是不便或者是老的位置+老的容量。
            int idx = e.hash & sizeMask;
            if (next == null)   //  Single node on list
                // 如果当前位置还不是链表，只是一个元素，直接赋值
                newTable[idx] = e;
            else { // Reuse consecutive sequence at same slot
                // 如果是链表了
                HashEntry<K,V> lastRun = e;
                int lastIdx = idx;
                // 新的位置只可能是不便或者是老的位置+老的容量。
                // 遍历结束后，lastRun后面的元素位置都是相同的
                for (HashEntry<K,V> last = next; last != null; last = last.next) {
                    int k = last.hash & sizeMask;
                    if (k != lastIdx) {
                        lastIdx = k;
                        lastRun = last;
                    }
                }
                // lastRun后面的元素位置都是相同的，直接作为链表赋值到新位置。
                newTable[lastIdx] = lastRun;
                // Clone remaining nodes
                for (HashEntry<K,V> p = e; p != lastRun; p = p.next) {
                    // 遍历剩余元素，头插法到指定k位置。
                    V v = p.value;
                    int h = p.hash;
                    int k = h & sizeMask;
                    HashEntry<K,V> n = newTable[k];
                    newTable[k] = new HashEntry<K,V>(h, p.key, v, n);
                }
            }
        }
    }
    // 头插法插入新的节点
    int nodeIndex = node.hash & sizeMask; // add the new node
    node.setNext(newTable[nodeIndex]);
    newTable[nodeIndex] = node;
    table = newTable;
}
```

有些同学可能会对最后的两个for循环有疑惑，这里第一个for是为了寻找这样一个节点，这个节点后面的所有next节点的新位置都是相同的。然后把这个作为一个链表赋值到新位置。第二个for循环是为了把剩余的元素通过头插法插入到指定位置链表。这样实现的原因可能是基于概率统计，有深入研究的同学可以发表下意见。

##### 5.get

到这里就很简单了，get方法只需要两步即可。

1. 计算得到key的存放位置。
2. 遍历指定位置查找相同key的value值。

```java
public V get(Object key) {
    Segment<K,V> s; // manually integrate access methods to reduce overhead
    HashEntry<K,V>[] tab;
    int h = hash(key);
    long u = (((h >>> segmentShift) & segmentMask) << SSHIFT) + SBASE;
    // 计算得到key的存放位置
    if ((s = (Segment<K,V>)UNSAFE.getObjectVolatile(segments, u)) != null &&
        (tab = s.table) != null) {
        for (HashEntry<K,V> e = (HashEntry<K,V>) UNSAFE.getObjectVolatile
                 (tab, ((long)(((tab.length - 1) & h)) << TSHIFT) + TBASE);
             e != null; e = e.next) {
            // 如果是链表，遍历查找到相同key的value。
            K k;
            if ((k = e.key) == key || (e.hash == h && key.equals(k)))
                return e.value;
        }
    }
    return null;
}
```

#### ConcurrentHashMap 1.8

##### 1.存储结构

![Java8 ConcurrentHashMap存储结构（图片来自javadoop）](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/java8_concurrenthashmap.png)

可以发现Java8的ConcurrentHashMap相对于Java7来说变化比较大，不再是之前的**Segment数组+HashEntry数组+链表**，而是**Node数组+链表/红黑树**。当冲突链表达到一定长度时，链表会转换成红黑树。

##### 2.初始化initTable


```java
/**
 * Initializes table, using the size recorded in sizeCtl.
 */
private final Node<K,V>[] initTable() {
    Node<K,V>[] tab; int sc;
    while ((tab = table) == null || tab.length == 0) {
        //　如果sizeCtl < 0,说明另外的线程执行CAS成功，正在进行初始化。
        if ((sc = sizeCtl) < 0)
            // 让出CPU使用权
            Thread.yield(); // lost initialization race; just spin
        else if (U.compareAndSwapInt(this, SIZECTL, sc, -1)) {
            try {
                if ((tab = table) == null || tab.length == 0) {
                    int n = (sc > 0) ? sc : DEFAULT_CAPACITY;
                    @SuppressWarnings("unchecked")
                    Node<K,V>[] nt = (Node<K,V>[])new Node<?,?>[n];
                    table = tab = nt;
                    sc = n - (n >>> 2);
                }
            } finally {
                sizeCtl = sc;
            }
            break;
        }
    }
    return tab;
}
```

从源码中可以发现ConcurrentHashMap的初始化是通过**自旋和CAS**操作完成的。里面需要注意的是变量sizeCtl，它的值决定着当前的初始化状态。

1. -1说明正在初始化
2. -N说明有N-1个线程正在进行扩容
3. 0表示table初始化大小，如果table没有初始化
4. \>0表示table扩容的阈值，如果table已经初始化。

##### 3.put

直接过一遍put源码。

```java
public V put(K key, V value) {
    return putVal(key, value, false);
}

/** Implementation for put and putIfAbsent */
final V putVal(K key, V value, boolean onlyIfAbsent) {
    // key和value不能为空
    if (key == null || value == null) throw new NullPointerException();
    int hash = spread(key.hashCode());
    int binCount = 0;
    for (Node<K,V>[] tab = table;;) {
        // f=目标位置元素
        Node<K,V> f; int n, i, fh;// fh后面存放目标位置的元素hash值
        if (tab == null || (n = tab.length) == 0)
            // 数组桶为空，初始化数组桶（自旋+CAS)
            tab = initTable();
        else if ((f = tabAt(tab, i = (n - 1) & hash)) == null) {
            // 桶内为空，CAS放入，不加锁，成功了就直接break跳出
            if (casTabAt(tab, i, null,new Node<K,V>(hash, key, value, null)))
                break;  // no lock when adding to empty bin
        }
        else if ((fh = f.hash) == MOVED)
            tab = helpTransfer(tab, f);
        else {
            V oldVal = null;
            // 使用synchronized加锁加入节点
            synchronized (f) {
                if (tabAt(tab, i) == f) {
                    // 说明是链表
                    if (fh >= 0) {
                        binCount = 1;
                        // 循环加入新的或者覆盖节点
                        for (Node<K,V> e = f;; ++binCount) {
                            K ek;
                            if (e.hash == hash &&
                                ((ek = e.key) == key ||
                                 (ek != null && key.equals(ek)))) {
                                oldVal = e.val;
                                if (!onlyIfAbsent)
                                    e.val = value;
                                break;
                            }
                            Node<K,V> pred = e;
                            if ((e = e.next) == null) {
                                pred.next = new Node<K,V>(hash, key,
                                                          value, null);
                                break;
                            }
                        }
                    }
                    else if (f instanceof TreeBin) {
                        // 红黑树
                        Node<K,V> p;
                        binCount = 2;
                        if ((p = ((TreeBin<K,V>)f).putTreeVal(hash, key,
                                                       value)) != null) {
                            oldVal = p.val;
                            if (!onlyIfAbsent)
                                p.val = value;
                        }
                    }
                }
            }
            if (binCount != 0) {
                if (binCount >= TREEIFY_THRESHOLD)
                    treeifyBin(tab, i);
                if (oldVal != null)
                    return oldVal;
                break;
            }
        }
    }
    addCount(1L, binCount);
    return null;
}
```

1. 根据key计算出hashcode。
2. 判断是否需要进行初始化。
3. 即为当前key定位出的Node，如果为空表示当前位置可以写入数据，利用CAS尝试写入，失败则自旋保证成功。
4. 如果当前位置的hashcode==MOVED==-1,则需要进行扩容。
5. 如果都不满足，则利用synchronized锁写入数据。
6. 如果数量大于TREEIFY_THRESHOLD则要执行树化方法，在treeifyBin中会首先判断当前数组长度≥64时才会将链表转换为红黑树。

##### 4.get

get流程比较简单，直接过一遍源码。

```java
public V get(Object key) {
    Node<K,V>[] tab; Node<K,V> e, p; int n, eh; K ek;
    // key所在的hash位置
    int h = spread(key.hashCode());
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (e = tabAt(tab, (n - 1) & h)) != null) {
        // 如果指定位置元素存在，头结点hash值相同
        if ((eh = e.hash) == h) {
            if ((ek = e.key) == key || (ek != null && key.equals(ek)))
                // key hash值相等，key值相同，直接返回元素value
                return e.val;
        }
        else if (eh < 0)
            // 头结点hash值小于0，说明正在扩容或者是红黑树，find查找
            return (p = e.find(h, key)) != null ? p.val : null;
        while ((e = e.next) != null) {
            // 是链表，遍历查找
            if (e.hash == h &&
                ((ek = e.key) == key || (ek != null && key.equals(ek))))
                return e.val;
        }
    }
    return null;
}
```

总结一下get过程：

1. 根据hash值计算位置。
2. 查找到指定位置，如果头节点就是要找的，直接返回它的value.
3. 如果头节点hash值小于0，说明正在扩容或者是红黑树，查找之。
4. 如果是链表，遍历查找之。

总的来说ConcurrentHashMap在Java8中相对于Java7来说变化还是挺大的。

#### 总结

Java7中ConcurrentHashMap使用的分段锁，也就是每一个Segment上同时只有一个线程可以操作，每一个Segment都是一个类似HashMap数组的结构，它可以扩容，它的冲突会转化为链表。但是Segment的个数一但初始化就不能改变。

Java8中的ConcurrentHashMap使用的Synchronized锁加CAS的机制。结构也由Java7中的**Segment数组+HashEntry数组+链表**进化成了**Node数组+链表/红黑树**，Node是类似于一个HashEntry的结构。它的冲突再达到一定大小时会转化成红黑树，在冲突小于一定数量时又退回链表。

有些同学可能对Synchronized的性能存在疑问，其实Synchronized锁自从引入锁升级策略后，性能不再是问题，有兴趣的同学可以自己了解下Synchronized的锁升级。


### ConcurrentHashMap和Hashtable的区别

ConcurrentHashMap和Hashtable的区别主要体现在实现线程安全的方式上不同。

- **底层数据结构**：JDK1.7的ConcurrentHashMap底层采用**分段的数组+链表**实现，JDK1.8采用的数据结构跟HashMap1.8的结构一样，数组+链表/红黑二叉树。Hashtable和JDK1.8之前的HashMap的底层数据结构类似都是采用**数组+链表**的形式，数组是HashMap的主体，链表则是主要为了解决哈希冲突而存在的；
- **实现线程安全的方式（重要）**：
  - 在JDK1.7的时候，ConcurrentHashMap对整个桶数组进行了分割分段(Segment，分段锁)，每一把锁只锁容器其中一部分数据（下面有示意图），多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。
  - 到了JDK1.8的时候，ConcurrentHashMap已经摒弃了Segment的概念，而是直接用Node数组+链表+红黑树的数据结构来实现，并发控制使用synchronized和CAS来操作。（JDK1.6以后synchronized锁做了很多优化）整个看起来就像是优化过且线程安全的HashMap，虽然在JDK1.8中还能看到Segment的数据结构，但是已经简化了属性，只是为了兼容旧版本；
  - Hashtable(同一把锁):使用synchronized来保证线程安全，效率非常低下。当一个线程访问同步方法时，其他线程也访问同步方法，可能会进入阻塞或轮询状态，如使用put添加元素，另一个线程不能使用put添加元素，也不能使用get，竞争会越来越激烈效率越低。

下面，我们再来看看两者底层数据结构的对比图。

**Hashtable**:

![Hashtable的内部结构](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/jdk1.7_hashmap.png)

**JDK1.7的ConcurrentHashMap**：

![Java7 ConcurrentHashMap存储结构](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/java7_concurrenthashmap.png)

ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成。

Segment数组中的每个元素包含一个HashEntry数组，每个HashEntry数组属于链表结构。

**JDK1.8的ConcurrentHashMap**：

![Java8 ConcurrentHashMap存储结构](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/java8_concurrenthashmap.png)

JDK1.8的ConcurrentHashMap不再是**Segment数组+HashEntry数组+链表**，而是**Node数组+链表/红黑树**。不过，Node只能用于链表的情况，红黑树的情况需要使用**TreeNode**。当冲突链表达到一定长度时，链表会转换成红黑树。

TreeNode是存储红黑树节点，被TreeBin包装。TreeBin通过root属性维护红黑树的根结点，因为红黑树在旋转的时候，根结点可能会被它原来的子节点替换掉，在这个时间点，如果有其他线程要写这棵红黑树就会发生线程不安全问题，所以在ConcurrentHashMap中TreeBin通过waiter属性维护当前使用这棵红黑树的线程，来防止其他线程的进入。

```java
static final class TreeBin<K,V> extends Node<K,V> {
    TreeNode<K,V> root;
    volatile TreeNode<K,V> first;
    volatile Thread waiter;
    volatile int lockState;
    // values for lockState
    static final int WRITER = 1; // set while holding write lock
    static final int WAITER = 2; // set when waiting for write lock
    static final int READER = 4; // increment value for setting read lock
...
}
```

### ConcurrentHashMap线程安全的具体实现方式/底层具体实现

#### JDK1.8之前

![Java7 ConcurrentHashMap存储结构](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/java7_concurrenthashmap.png)

首先将数据分为一段一段（这个“段”就是Segment）的存储，然后给每一段数据配一把锁，当一个线程占用锁访问其中一个段数据时，其他段的数据也能被其他线程访问。

**ConcurrentHashMap是由Segment数组结构和HashEntry数组结构组成**。

Segment继承了ReentrantLock,所以Segment是一种可重入锁，扮演锁的角色。HashEntry用于存储键值对数据。

```java
static class Segment<K,V> extends ReentrantLock implements Serializable {
}
```

一个ConcurrentHashMap里包含一个Segment数组，Segment的个数一旦**初始化就不能改变**。Segment数组的大小默认是16，也就是说默认可以同时支持16个线程并发写。

Segment的结构和HashMap类似，是一种数组和链表结构，一个Segment包含一个HashEntry数组，每个HashEntry是一个链表结构的元素，每个Segment守护着一个HashEntry数组里的元素，当对HashEntry数组的数据进行修改时，必须首先获得对应的Segment的锁。也就是说，对同一Segment的并发写入会被阻塞，不同Segment的写入是可以并发执行的。

#### JDK1.8之后

![Java8 ConcurrentHashMap存储结构](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/java8_concurrenthashmap.png)

Java8几乎完全重写了ConcurrentHashMap，代码量从原来Java7中的1000多行，变成了现在的6000多行。

**ConcurrentHashMap取消了Segment分段锁，采用Node+CAS+synchronized来保证并发安全**。数据结构跟HashMap1.8的结构类似，数组+链表/红黑二叉树。Java8在链表长度超过一定阈值（8）时将链表（寻址时间复杂度为O(N)）转换为红黑树（寻址时间复杂度为O(log(N))）。

Java8中，锁粒度更细，synchronized只锁定当前链表或红黑二叉树的首节点，这样只要hash不冲突，就不会产生并发，就不会影响其他Node的读写，效率大幅提升。

### JDK1.7和JDK1.8的ConcurrentHashMap实现有什么不同？

- **线程安全实现方式**：JDK1.7采用Segment分段锁来保证安全，Segment是继承自ReentrantLock。JDK1.8放弃了Segment分段锁的设计，采用Node+CAS+synchronized保证线程安全，锁粒度更细，synchronized只锁定当前链表或红黑二叉树的首节点。
- **Hash碰撞解决方法**：JDK1.7采用拉链法，JDK1.8采用拉链法结合红黑树（链表长度超过一定阈值时，将链表转换为红黑树）。
- **并发度**：JDK1.7最大并发度是Segment的个数，默认是16。JDK1.8最大并发度是Node数组的大小，并发度更大。


### ConcurrentHashMap的哪些操作需要加锁？

答：只有写入操作才需要加锁，读取操作不需要加锁

### ConcurrentHashMap的无锁读是如何实现的？

答：首先HashEntry中的value和next都是有volatile修饰的，其次在写入操作的时候通过调用UNSAFE库延迟同步了主存，保证了数据的一致性

### 在多线程的场景下调用size()方法获取ConcurrentHashMap的大小有什么挑战？ConcurrentHashMap是怎么解决的？

答：size()具有全局的语义，如何能保证在不加全局锁的情况下读取到全局状态的值是一个很大的挑战，ConcurrentHashMap通过查看两次无锁读中间是否发生了写入操作来决定读取到的size()是否可信，如果写入操作频繁，则再退化为全局加锁读取。

### 在有Segment存在的前提下，是如何扩容的？

答：segment数组的大小在一开始初始化的时候就已经决定了，扩容主要扩的是HashEntry数组，基本的思路与HashTable一致，但这是一个线程不安全方法，调用之前需要加锁。

### 为什么JDK8舍弃掉了分段锁呢？

1. 每个锁控制的是一段，当分段很多，并且加锁的分段不连续的时候，内存空间的浪费比较严重。
2. 如果某个分段特别的大，那么就会影响效率，耽误时间。

### HashMap可以存null，ConcurrentHashMap不可以，为什么？

关于这个问题，其实最有发言权的就是ConcurrentHashMap的作者——Doug Lea
ConcurrentMap(如ConcurrentHashMap、ConcurrentSkipListMap)不允许使用null值的主要原因是在非并发的Map中(如HashMap)是可以容忍模糊性(二义性)的，而在并发Map中是无法容忍的。假如说，所有的Map都支持null的话，那么map.get(key)就可以返回null，但是这时候就会存在一个不确定性，当你拿到null的时候，你是不知道他是因为本来就存了一个null进去还是说就是因为没找到而返回了null。在HashMap中，因为它的设计就是给单线程用的，所以当我们map.get(key)返回null的时候，我们是可以通过map.contains(key)检查来进行检测的，如果它返回true，则认为是存了一个null，否则就是因为没找到而返回了null。但是像ConcurrentHashMap，它是为并发而生的，它是要用在并发场景中的，当我们map.get(key)返回null的时候，是没办法通过通过map.contains(key)检查来准确的检测，因为在检测过程中可能会被其他线程锁修改，而导致检测结果并不可靠。所以，为了让ConcurrentHashMap的语义更加准确，不存在二义性的问题，他就不支持null。

### ConcurrentHashMap相关文章

| [那些年你啃过的ConcurrentHashMap](https://mp.weixin.qq.com/s/ufoKhs4VRXhE8_PT2rXoeg) | [面试必问之ConcurrentHashMap线程安全的具体实现方式](https://mp.weixin.qq.com/s?__biz=MzkzODE3OTI0Ng==&mid=2247491116&idx=1&sn=30ee6196266dab2cbf46cf7f98d99120&source=41#wechat_redirect) | [ConcurrentHashMap是如何保证线程安全的](https://mp.weixin.qq.com/s/Hb7JCkMrri0VDiPcl8HnBQ) |
| :----------------------------------------------------------: | :----------------------------------------------------------: | :----------------------------------------------------------: |


## 面试题

### HashMap和Hashtable的区别

- **线程是否安全**：HashMap是非线程安全的，Hashtable是线程安全的,因为Hashtable内部的方法基本都经过synchronized修饰。（如果你要保证线程安全的话就使用ConcurrentHashMap）；
- **效率**：因为线程安全的问题，HashMap要比Hashtable效率高一点。另外，Hashtable基本被淘汰，不要在代码中使用它；
- **对Null key和Null value的支持**：HashMap可以存储null的key和value，但null作为键只能有一个，null作为值可以有多个；Hashtable不允许有null键和null值，否则会抛出NullPointerException。
- **初始容量大小和每次扩充容量大小的不同**：①创建时如果不指定容量初始值，Hashtable默认的初始大小为11，之后每次扩充，容量变为原来的2n+1。HashMap默认的初始化大小为16。之后每次扩充，容量变为原来的2倍。②创建时如果给定了容量初始值，那么Hashtable会直接使用你给定的大小，而HashMap会将其扩充为2的幂次方大小（HashMap中的tableSizeFor()方法保证，下面给出了源代码）。也就是说HashMap总是使用2的幂作为哈希表的大小,后面会介绍到为什么是2的幂次方。
- **底层数据结构**：JDK1.8以后的HashMap在解决哈希冲突时有了较大的变化，当链表长度大于阈值（默认为8）时，将链表转化为红黑树（将链表转换成红黑树前会判断，如果当前数组的长度小于64，那么会选择先进行数组扩容，而不是转换为红黑树），以减少搜索时间（后文中我会结合源码对这一过程进行分析）。Hashtable没有这样的机制。

**HashMap中带有初始容量的构造函数**：

```java
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
public HashMap(int initialCapacity) {
    this(initialCapacity, DEFAULT_LOAD_FACTOR);
}
```
下面这个方法保证了HashMap总是使用2的幂作为哈希表的大小。

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

### HashMap和HashSet区别

如果你看过HashSet源码的话就应该知道：HashSet底层就是基于HashMap实现的。（HashSet的源码非常非常少，因为除了clone()、writeObject()、readObject()是HashSet自己不得不实现之外，其他方法都是直接调用HashMap中的方法。

|               HashMap                |                          HashSet                           |
| :------------------------------------: | :----------------------------------------------------------: |
|           实现了Map接口            |                       实现Set接口                        |
|               存储键值对               |                          仅存储对象                          |
|     调用put()向map中添加元素      |             调用add()方法向Set中添加元素              |
| HashMap使用键（Key）计算hashcode | HashSet使用成员对象来计算hashcode值，对于两个对象来说hashcode可能相同，所以equals()方法用来判断对象的相等性 |

### HashMap和TreeMap区别

TreeMap和HashMap都继承自**AbstractMap**，但是需要注意的是TreeMap它还实现了**NavigableMap**接口和**SortedMap**接口。

![TreeMap继承关系图](https://guide-blog-images.oss-cn-shenzhen.aliyuncs.com/github/javaguide/java/collection/treemap_hierarchy.png)

实现**NavigableMap**接口让TreeMap有了对集合内元素的搜索的能力。

实现**SortedMap**接口让TreeMap有了对集合中的元素根据键排序的能力。默认是按key的升序排序，不过我们也可以指定排序的比较器。示例代码如下：
```java
/**
 * @author shuang.kou
 * @createTime 2020年06月15日 17:02:00
 */
public class Person {
    private Integer age;

    public Person(Integer age) {
        this.age = age;
    }

    public Integer getAge() {
        return age;
    }


    public static void main(String[] args) {
        TreeMap<Person, String> treeMap = new TreeMap<>(new Comparator<Person>() {
            @Override
            public int compare(Person person1, Person person2) {
                int num = person1.getAge() - person2.getAge();
                return Integer.compare(num, 0);
            }
        });
        treeMap.put(new Person(3), "person1");
        treeMap.put(new Person(18), "person2");
        treeMap.put(new Person(35), "person3");
        treeMap.put(new Person(16), "person4");
        treeMap.entrySet().stream().forEach(personStringEntry -> {
            System.out.println(personStringEntry.getValue());
        });
    }
}
```

输出:
```text
person1
person4
person2
person3
```

可以看出，TreeMap中的元素已经是按照Person的age字段的升序来排列了。

上面，我们是通过传入匿名内部类的方式实现的，你可以将代码替换成Lambda表达式实现的方式：

```java
TreeMap<Person, String> treeMap = new TreeMap<>((person1, person2) -> {
  int num = person1.getAge() - person2.getAge();
  return Integer.compare(num, 0);
});
```

**综上，相比于HashMap来说TreeMap主要多了对集合中的元素根据键排序的能力以及对集合内元素的搜索的能力**。

### HashSet如何检查重复?

> 当你把对象加入HashSet时，HashSet会先计算对象的hashcode值来判断对象加入的位置，同时也会与其他加入的对象的hashcode值作比较，如果没有相符的hashcode，HashSet会假设对象没有重复出现。但是如果发现有相同hashcode值的对象，这时会调用equals()方法来检查hashcode相等的对象是否真的相同。如果两者相同，HashSet就不会让加入操作成功。

在JDK1.8中，HashSet的add()方法只是简单的调用了HashMap的put()方法，并且判断了一下返回值以确保是否有重复元素。直接看一下HashSet中的源码：
```java
// Returns: true if this set did not already contain the specified element
// 返回值：当set中没有包含add的元素时返回真
public boolean add(E e) {
    return map.put(e, PRESENT)==null;
}
```

而在HashMap的putVal()方法中也能看到如下说明：

```java
// Returns : previous value, or null if none
// 返回值：如果插入位置没有元素返回null，否则返回上一个元素
final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
...
}
```

也就是说，在JDK1.8中，实际上无论HashSet中是否已经存在了某元素，HashSet都会直接插入，只是会在add()方法的返回值处告诉我们插入前是否存在相同元素。