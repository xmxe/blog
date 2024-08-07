---
title: ArrayList&CopyOnWriteArrayList
categories: Java
index_img: /assert/list.jpg
img: https://pic2.zhimg.com/v2-afbb5fb285f0775146b89fe018dbbc93_r.jpg

---

## ArrayList

ArrayList的底层实现是一个Object数组,ArrayList的无参构造函数为底层的Object数组也就是elementData赋值了一个默认的空数组DEFAULTCAPACITY_EMPTY_ELEMENTDATA。也就是说，使用无参构造函数初始化ArrayList后，它当时的数组容量为0,只有当我们真正对数据进行添加操作add时，才会给数组分配一个默认的初始容量DEFAULT_CAPACITY = 10,ArrayList的有参构造函数就是按照用户传入的大小开辟数组空间。

ArrayList的底层是数组队列，相当于动态数组。与Java中的数组相比，它的容量能动态增长。在添加大量元素前，应用程序可以使用ensureCapacity操作来增加ArrayList实例的容量。这可以减少递增式再分配的数量。

ArrayList继承于AbstractList,实现了**List**,**RandomAccess**,**Cloneable**,**java.io.Serializable**这些接口。

```java
public class ArrayList<E> extends AbstractList<E> implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
}
```

- **RandomAccess**是一个标志接口，表明实现这个接口的List集合是支持快速随机访问的。在ArrayList中，我们即可以通过元素的序号快速获取元素对象，这就是快速随机访问。
- ArrayList实现了**Cloneable接口**，即覆盖了函数clone()，能被克隆。
- ArrayList实现了**java.io.Serializable**接口，这意味着ArrayList支持序列化，能通过序列化去传输。

### Arraylist和Vector的区别

1. ArrayList是List的主要实现类，底层使用Object[]存储，适用于频繁的查找工作，线程不安全。
2. Vector是List的古老实现类，底层使用Object[]存储，线程安全的。

### Arraylist与LinkedList区别

(1) **是否保证线程安全**：ArrayList和LinkedList都是不同步的，也就是ArrayList和LinkedList都不是线程安全的，以add为例,源码如下
```java
elementData[size++] = e;
// 它由两步操作构成
elementData[size] = e;
size = size + 1;
```
在单线程执行这两条代码时，那当然没有任何问题，但是当多线程环境下执行时，可能就会发生一个线程添加的值覆盖另一个线程添加的值。举个例子：假设size = 0，我们要往这个数组的末尾添加元素，线程A开始添加一个元素，值为A。此时它执行第一条操作，将A放在了数组elementData下标为0的位置上，接着线程B刚好也要开始添加一个值为B的元素，且走到了第一步操作。此时线程B获取到的size值依然为0，于是它将B也放在了elementData下标为0的位置上，线程A开始增加size的值，size = 1,线程B开始增加size的值，size = 2,这样，线程A、B 都执行完毕后，理想的情况应该是size = 2，elementData[0] = A，elementData[1] = B。而实际情况变成了size = 2，elementData[0] = B（线程B覆盖了线程A的操作），下标1的位置上什么都没有。并且后续除非我们使用set方法修改下标为1的值，否则这个位置上将一直为null，因为在末尾添加元素时将会从size = 2的位置上开始。

(2) **底层数据结构**：Arraylist底层使用的是Object数组；LinkedList底层使用的是双向链表数据结构（JDK1.6之前为循环链表，JDK1.7取消了循环。注意双向链表和双向循环链表的区别）

(3) **插入和删除是否受元素位置的影响**：

   ①ArrayList采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。比如：执行`add(E e)`方法的时候，ArrayList会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是O(1)。但是如果要在指定位置i插入和删除元素的话（add(int index, E element)）时间复杂度就为O(n-i)。因为在进行上述操作的时候集合中第i和第i个元素之后的(n-i)个元素都要执行向后位/向前移一位的操作。

   ②LinkedList采用链表存储，所以对于`add(E e)`方法的插入，删除元素时间复杂度不受元素位置的影响，近似O(1)，如果是要在指定位置i插入和删除元素的话（(add(int index, E element)）时间复杂度近似为o(n))因为需要先移动到指定位置再插入。

(4) **是否支持快速随机访问**：LinkedList不支持高效的随机元素访问，而ArrayList支持。快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index)方法)。

(5) **内存空间占用**：ArrayList的空间浪费主要体现在在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗比ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）。

### ArrayList核心源码解读

```java
package java.util;

import java.util.function.Consumer;
import java.util.function.Predicate;
import java.util.function.UnaryOperator;

public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable{
    
    private static final long serialVersionUID = 8683452581122892189L;
    /**
     * 默认初始容量大小
     */
    private static final int DEFAULT_CAPACITY = 10;
    /**
     * 空数组（用于空实例）。
     */
    private static final Object[] EMPTY_ELEMENTDATA = {};
    
     //用于默认大小空实例的共享空数组实例。
     //我们把它从EMPTY_ELEMENTDATA数组中区分出来，以知道在添加第一个元素时容量需要增加多少。
    private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

    /**
     * 保存ArrayList数据的数组
     */
    transient Object[] elementData; // non-private to simplify nested class access

    /**
     * ArrayList所包含的元素个数
     */
    private int size;

    /**
     * 带初始容量参数的构造函数（用户可以在创建ArrayList对象时自己指定集合的初始大小）
     */
    public ArrayList(int initialCapacity) {
        if (initialCapacity > 0) {
            //如果传入的参数大于0，创建initialCapacity大小的数组
            this.elementData = new Object[initialCapacity];
        } else if (initialCapacity == 0) {
            //如果传入的参数等于0，创建空数组
            this.elementData = EMPTY_ELEMENTDATA;
        } else {
            //其他情况，抛出异常
            throw new IllegalArgumentException("Illegal Capacity: "+
                                               initialCapacity);
        }
    }

    /**
     *默认无参构造函数
     *DEFAULTCAPACITY_EMPTY_ELEMENTDATA为0.初始化为10，也就是说初始其实是空数组,当添加第一个元素的时候数组容量才变成10
     */
    public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }

    /**
     * 构造一个包含指定集合的元素的列表，按照它们由集合的迭代器返回的顺序。
     */
    public ArrayList(Collection<? extends E> c) {
        //将指定集合转换为数组
        elementData = c.toArray();
        //如果elementData数组的长度不为0
        if ((size = elementData.length) != 0) {
            // 如果elementData不是Object类型数据（c.toArray可能返回的不是Object类型的数组所以加上下面的语句用于判断）
            if (elementData.getClass() != Object[].class)
                //将原来不是Object类型的elementData数组的内容，赋值给新的Object类型的elementData数组
                elementData = Arrays.copyOf(elementData, size, Object[].class);
        } else {
            // 其他情况，用空数组代替
            this.elementData = EMPTY_ELEMENTDATA;
        }
    }

    /**
     * 修改这个ArrayList实例的容量是列表的当前大小。应用程序可以使用此操作来最小化ArrayList实例的存储。
     */
    public void trimToSize() {
        modCount++;
        if (size < elementData.length) {
            elementData = (size == 0)
              ? EMPTY_ELEMENTDATA
              : Arrays.copyOf(elementData, size);
        }
    }
    
    //下面是ArrayList的扩容机制
    //ArrayList的扩容机制提高了性能，如果每次只扩充一个，
    //那么频繁的插入会导致频繁的拷贝，降低性能，而ArrayList的扩容机制避免了这种情况。
    /**
     * 如有必要，增加此ArrayList实例的容量，以确保它至少能容纳元素的数量
     * @param   minCapacity   所需的最小容量
     */
    public void ensureCapacity(int minCapacity) {
        //如果是true，minExpand的值为0，如果是false,minExpand的值为10
        int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
            // any size if not default element table
            ? 0
            // larger than default for default empty table. It's already
            // supposed to be at default size.
            : DEFAULT_CAPACITY;
        //如果最小容量大于已有的最大容量
        if (minCapacity > minExpand) {
            ensureExplicitCapacity(minCapacity);
        }
    }
   //1.得到最小扩容量
   //2.通过最小容量扩容
    private void ensureCapacityInternal(int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
              // 获取“默认的容量”和“传入参数”两者之间的最大值
            minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
        }

        ensureExplicitCapacity(minCapacity);
    }
  //判断是否需要扩容
    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;

        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            //调用grow方法进行扩容，调用此方法代表已经开始扩容了
            grow(minCapacity);
    }

    /**
     * 要分配的最大数组大小
     */
    private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

    /**
     * ArrayList扩容的核心方法。
     */
    private void grow(int minCapacity) {
        // oldCapacity为旧容量，newCapacity为新容量
        int oldCapacity = elementData.length;
        //将oldCapacity右移一位，其效果相当于oldCapacity/2，
        //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
        int newCapacity = oldCapacity + (oldCapacity >> 1);
        //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        //再检查新容量是否超出了ArrayList所定义的最大容量，
        //若超出了，则调用hugeCapacity()来比较minCapacity和MAX_ARRAY_SIZE，
        //如果minCapacity大于MAX_ARRAY_SIZE，则新容量则为Interger.MAX_VALUE，否则，新容量大小则为MAX_ARRAY_SIZE。
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
    //比较minCapacity和MAX_ARRAY_SIZE
    private static int hugeCapacity(int minCapacity) {
        if (minCapacity < 0) // overflow
            throw new OutOfMemoryError();
        return (minCapacity > MAX_ARRAY_SIZE) ?
            Integer.MAX_VALUE :
            MAX_ARRAY_SIZE;
    }

    /**
     *返回此列表中的元素数。
     */
    public int size() {
        return size;
    }

    /**
     * 如果此列表不包含元素，则返回true。
     */
    public boolean isEmpty() {
        //注意=和==的区别
        return size == 0;
    }

    /**
     * 如果此列表包含指定的元素，则返回true。
     */
    public boolean contains(Object o) {
        //indexOf()方法：返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1
        return indexOf(o) >= 0;
    }

    /**
     *返回此列表中指定元素的首次出现的索引，如果此列表不包含此元素，则为-1
     */
    public int indexOf(Object o) {
        if (o == null) {
            for (int i = 0; i < size; i++)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = 0; i < size; i++)
                //equals()方法比较
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此列表中指定元素的最后一次出现的索引，如果此列表不包含元素，则返回-1。.
     */
    public int lastIndexOf(Object o) {
        if (o == null) {
            for (int i = size-1; i >= 0; i--)
                if (elementData[i]==null)
                    return i;
        } else {
            for (int i = size-1; i >= 0; i--)
                if (o.equals(elementData[i]))
                    return i;
        }
        return -1;
    }

    /**
     * 返回此ArrayList实例的浅拷贝。（元素本身不被复制。）
     */
    public Object clone() {
        try {
            ArrayList<?> v = (ArrayList<?>) super.clone();
            //Arrays.copyOf功能是实现数组的复制，返回复制后的数组。参数是被复制的数组和复制的长度
            v.elementData = Arrays.copyOf(elementData, size);
            v.modCount = 0;
            return v;
        } catch (CloneNotSupportedException e) {
            // 这不应该发生，因为我们是可以克隆的
            throw new InternalError(e);
        }
    }

    /**
     * 以正确的顺序（从第一个到最后一个元素）返回一个包含此列表中所有元素的数组。
     * 返回的数组将是“安全的”，因为该列表不保留对它的引用。（换句话说，这个方法必须分配一个新的数组）。
     * 因此，调用者可以自由地修改返回的数组。此方法充当基于阵列和基于集合的API之间的桥梁。
     */
    public Object[] toArray() {
        return Arrays.copyOf(elementData, size);
    }

    /**
     * 以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）;
     * 返回的数组的运行时类型是指定数组的运行时类型。如果列表适合指定的数组，则返回其中。
     * 否则，将为指定数组的运行时类型和此列表的大小分配一个新数组。
     * 如果列表适用于指定的数组，其余空间（即数组的列表数量多于此元素），则紧跟在集合结束后的数组中的元素设置为null。
     * （这仅在调用者知道列表不包含任何空元素的情况下才能确定列表的长度。）
     */
    @SuppressWarnings("unchecked")
    public <T> T[] toArray(T[] a) {
        if (a.length < size)
            // 新建一个运行时类型的数组，但是ArrayList数组的内容
            return (T[]) Arrays.copyOf(elementData, size, a.getClass());
            //调用System提供的arraycopy()方法实现数组之间的复制
        System.arraycopy(elementData, 0, a, 0, size);
        if (a.length > size)
            a[size] = null;
        return a;
    }

    // Positional Access Operations

    @SuppressWarnings("unchecked")
    E elementData(int index) {
        return (E) elementData[index];
    }

    /**
     * 返回此列表中指定位置的元素。
     */
    public E get(int index) {
        rangeCheck(index);

        return elementData(index);
    }

    /**
     * 用指定的元素替换此列表中指定位置的元素。
     */
    public E set(int index, E element) {
        //对index进行界限检查
        rangeCheck(index);

        E oldValue = elementData(index);
        elementData[index] = element;
        //返回原来在这个位置的元素
        return oldValue;
    }

    /**
     * 将指定的元素追加到此列表的末尾。
     */
    public boolean add(E e) {
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //这里看到ArrayList添加元素的实质就相当于为数组赋值
        elementData[size++] = e;
        return true;
    }

    /**
     * 在此列表中的指定位置插入指定的元素。
     * 先调用rangeCheckForAdd对index进行界限检查；然后调用ensureCapacityInternal方法保证capacity足够大；
     * 再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
     */
    public void add(int index, E element) {
        rangeCheckForAdd(index);

        ensureCapacityInternal(size + 1);  // Increments modCount!!
        //arraycopy()这个实现数组之间复制的方法一定要看一下，下面就用到了arraycopy()方法实现数组自己复制自己
        System.arraycopy(elementData, index, elementData, index + 1,
                         size - index);
        elementData[index] = element;
        size++;
    }

    /**
     * 删除该列表中指定位置的元素。将任何后续元素移动到左侧（从其索引中减去一个元素）。
     */
    public E remove(int index) {
        rangeCheck(index);

        modCount++;
        E oldValue = elementData(index);

        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
      //从列表中删除的元素
        return oldValue;
    }

    /**
     * 从列表中删除指定元素的第一个出现（如果存在）。如果列表不包含该元素，则它不会更改。
     * 返回true，如果此列表包含指定的元素
     */
    public boolean remove(Object o) {
        if (o == null) {
            for (int index = 0; index < size; index++)
                if (elementData[index] == null) {
                    fastRemove(index);
                    return true;
                }
        } else {
            for (int index = 0; index < size; index++)
                if (o.equals(elementData[index])) {
                    fastRemove(index);
                    return true;
                }
        }
        return false;
    }

    /*
     * Private remove method that skips bounds checking and does not
     * return the value removed.
     */
    private void fastRemove(int index) {
        modCount++;
        int numMoved = size - index - 1;
        if (numMoved > 0)
            System.arraycopy(elementData, index+1, elementData, index,
                             numMoved);
        elementData[--size] = null; // clear to let GC do its work
    }

    /**
     * 从列表中删除所有元素。
     */
    public void clear() {
        modCount++;

        // 把数组中所有的元素的值设为null
        for (int i = 0; i < size; i++)
            elementData[i] = null;

        size = 0;
    }

    /**
     * 按指定集合的Iterator返回的顺序将指定集合中的所有元素追加到此列表的末尾。
     */
    public boolean addAll(Collection<? extends E> c) {
        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount
        System.arraycopy(a, 0, elementData, size, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 将指定集合中的所有元素插入到此列表中，从指定的位置开始。
     */
    public boolean addAll(int index, Collection<? extends E> c) {
        rangeCheckForAdd(index);

        Object[] a = c.toArray();
        int numNew = a.length;
        ensureCapacityInternal(size + numNew);  // Increments modCount

        int numMoved = size - index;
        if (numMoved > 0)
            System.arraycopy(elementData, index, elementData, index + numNew,
                             numMoved);

        System.arraycopy(a, 0, elementData, index, numNew);
        size += numNew;
        return numNew != 0;
    }

    /**
     * 从此列表中删除所有索引为fromIndex（含）和toIndex之间的元素。
     * 将任何后续元素移动到左侧（减少其索引）。
     */
    protected void removeRange(int fromIndex, int toIndex) {
        modCount++;
        int numMoved = size - toIndex;
        System.arraycopy(elementData, toIndex, elementData, fromIndex,
                         numMoved);

        // clear to let GC do its work
        int newSize = size - (toIndex-fromIndex);
        for (int i = newSize; i < size; i++) {
            elementData[i] = null;
        }
        size = newSize;
    }

    /**
     * 检查给定的索引是否在范围内。
     */
    private void rangeCheck(int index) {
        if (index >= size)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * add和addAll使用的rangeCheck的一个版本
     */
    private void rangeCheckForAdd(int index) {
        if (index > size || index < 0)
            throw new IndexOutOfBoundsException(outOfBoundsMsg(index));
    }

    /**
     * 返回IndexOutOfBoundsException细节信息
     */
    private String outOfBoundsMsg(int index) {
        return "Index: "+index+", Size: "+size;
    }

    /**
     * 从此列表中删除指定集合中包含的所有元素。
     */
    public boolean removeAll(Collection<?> c) {
        Objects.requireNonNull(c);
        //如果此列表被修改则返回true
        return batchRemove(c, false);
    }

    /**
     * 仅保留此列表中包含在指定集合中的元素。
     *换句话说，从此列表中删除其中不包含在指定集合中的所有元素。
     */
    public boolean retainAll(Collection<?> c) {
        Objects.requireNonNull(c);
        return batchRemove(c, true);
    }

    /**
     * 从列表中的指定位置开始，返回列表中的元素（按正确顺序）的列表迭代器。
     * 指定的索引表示初始调用将返回的第一个元素为next。初始调用previous将返回指定索引减1的元素。
     * 返回的列表迭代器是fail-fast。
     */
    public ListIterator<E> listIterator(int index) {
        if (index < 0 || index > size)
            throw new IndexOutOfBoundsException("Index: "+index);
        return new ListItr(index);
    }

    /**
     * 返回列表中的列表迭代器（按适当的顺序）。
     * 返回的列表迭代器是fail-fast。
     */
    public ListIterator<E> listIterator() {
        return new ListItr(0);
    }

    /**
     * 以正确的顺序返回该列表中的元素的迭代器。
     * 返回的迭代器是fail-fast。
     */
    public Iterator<E> iterator() {
        return new Itr();
    }
}
```

### ArrayList扩容机制分析

扩容后的数组长度 = 当前数组长度 + 当前数组长度 / 2,即扩容成原来的1.5倍。最后使用Arrays.copyOf方法直接把原数组中的数组copy过来，需要注意的是，Arrays.copyOf方法会创建一个新数组然后再进行拷贝。

> [ArrayList的扩容机制](https://mp.weixin.qq.com/s/GY7RLE-yIF7jPAjqu5V9Cg)

**先从ArrayList的构造函数说起，（JDK8）ArrayList有三种方式来初始化，构造方法源码如下**：

```java
/**
  * 默认初始容量大小
  */
private static final int DEFAULT_CAPACITY = 10;

private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

/**
  * 默认构造函数，使用初始容量10构造一个空列表(无参数构造)
  */
public ArrayList() {
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}

/**
  * 带初始容量参数的构造函数。（用户自己指定容量）
  */
public ArrayList(int initialCapacity) {
    if (initialCapacity > 0) {//初始容量大于0
        //创建initialCapacity大小的数组
        this.elementData = new Object[initialCapacity];
    } else if (initialCapacity == 0) {//初始容量等于0
        //创建空数组
        this.elementData = EMPTY_ELEMENTDATA;
    } else {//初始容量小于0，抛出异常
        throw new IllegalArgumentException("Illegal Capacity: "+
                                           initialCapacity);
    }
}
/**
  *构造包含指定collection元素的列表，这些元素利用该集合的迭代器按顺序返回
  *如果指定的集合为null，throws NullPointerException。
  */
public ArrayList(Collection<? extends E> c) {
    elementData = c.toArray();
    if ((size = elementData.length) != 0) {
        // c.toArray might (incorrectly) not return Object[] (see 6260652)
        if (elementData.getClass() != Object[].class)
            elementData = Arrays.copyOf(elementData, size, Object[].class);
    } else {
        // replace with empty array.
        this.elementData = EMPTY_ELEMENTDATA;
    }
}
```

细心的同学一定会发现：以无参数构造方法创建ArrayList时，实际上初始化赋值的是一个空数组。当真正对数组进行添加元素操作时，才真正分配容量。即向数组中添加第一个元素时，数组容量扩为10。

> 补充：JDK6 new无参构造的ArrayList对象时，直接创建了长度是10的Object[]数组elementData。

**一步一步分析ArrayList扩容机制**

这里以无参构造函数创建的ArrayList为例分析

**add()方法**

```java
/**
  * 将指定的元素追加到此列表的末尾。
  */
public boolean add(E e) {
    //添加元素之前，先调用ensureCapacityInternal方法
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //这里看到ArrayList添加元素的实质就相当于为数组赋值
    elementData[size++] = e;
    return true;
}
```

> **注意**：JDK11移除了ensureCapacityInternal()和ensureExplicitCapacity()方法

添加数据前会先判断一下是否需要扩容,先讲下add(int index, E element) 这个方法的含义，就是在指定索引index处插入元素element。比如说ArrayList.add(0, 3)，意思就是在头部插入元素3。再来看看add方法的核心System.arraycopy，这个方法有5个参数：
```java
elementData //源数组
index //从源数组中的哪个位置开始复制
elementData //目标数组
toIndex //复制到目标数组中的哪个位置
length //要复制的源数组中数组元素的数量
```
举个例子，我们想要在index = 5的位置插入元素(数组大小为10)，首先，我们会复制一遍源数组elementData，然后把源数组中从index = 5的位置开始到数组末尾的元素，放到新数组的index + 1 = 6的位置上,于是，这就给我们要新增的元素腾出了位置，然后在新数组index = 5的位置放入元素element就完成了添加的操作,显然ArrayList的将数据插入到指定位置的操作性能非常低下，因为要开辟新数组复制元素，要是涉及到扩容那就更慢了。另外，ArrayList还内置了一个直接在末尾添加元素的add方法，不用复制数组，直接size++就好，这个方法应该是我们最常使用的，即直接add(element);

**ensureCapacityInternal()方法**

（JDK7）可以看到add方法首先调用了ensureCapacityInternal(size + 1)

```java
// 得到最小扩容量
private void ensureCapacityInternal(int minCapacity) {
    if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
        // 获取默认的容量和传入参数的较大值
        minCapacity = Math.max(DEFAULT_CAPACITY, minCapacity);
    }
    ensureExplicitCapacity(minCapacity);
}
```

**当要add进第1个元素时，minCapacity为1，在Math.max()方法比较后，minCapacity为10。**

> 此处和后续JDK8代码格式化略有不同，核心代码基本一样。

**ensureExplicitCapacity()方法**

如果调用ensureCapacityInternal()方法就一定会进入（执行）这个方法，下面我们来研究一下这个方法的源码！

```java
// 判断是否需要扩容
private void ensureExplicitCapacity(int minCapacity) {
    modCount++;

    // overflow-conscious code
    if (minCapacity - elementData.length > 0)
        //调用grow方法进行扩容，调用此方法代表已经开始扩容了
        grow(minCapacity);
}
```

我们来仔细分析一下：

- 当我们要add进第1个元素到ArrayList时，elementData.length为0（因为还是一个空的list），因为执行了ensureCapacityInternal()方法，所以minCapacity此时为10。此时minCapacity - elementData.length > 0成立，所以会进入grow(minCapacity)方法。
- 当add第2个元素时，minCapacity为2，此时elementData.length(容量)在添加第一个元素后扩容成10了。此时，minCapacity - elementData.length > 0不成立，所以不会进入（执行）grow(minCapacity)方法。
- 添加第3、4···到第10个元素时，依然不会执行grow方法，数组容量都为10。

直到添加第11个元素，minCapacity(为11)比elementData.length（为10）要大。进入grow方法进行扩容。

**grow()方法**

```java
/**
  * 要分配的最大数组大小
  */
private static final int MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;

/**
  * ArrayList扩容的核心方法。
  */
private void grow(int minCapacity) {
    // oldCapacity为旧容量，newCapacity为新容量
    int oldCapacity = elementData.length;
    //将oldCapacity 右移一位，其效果相当于oldCapacity /2，
    //我们知道位运算的速度远远快于整除运算，整句运算式的结果就是将新容量更新为旧容量的1.5倍，
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //然后检查新容量是否大于最小需要容量，若还是小于最小需要容量，那么就把最小需要容量当作数组的新容量，
    if (newCapacity - minCapacity < 0)
        newCapacity = minCapacity;
    // 如果新容量大于MAX_ARRAY_SIZE,进入(执行)`hugeCapacity()`方法来比较minCapacity和MAX_ARRAY_SIZE，
    //如果minCapacity大于最大容量，则新容量则为`Integer.MAX_VALUE`，否则，新容量大小则为MAX_ARRAY_SIZE即为`Integer.MAX_VALUE - 8`。
    if (newCapacity - MAX_ARRAY_SIZE > 0)
        newCapacity = hugeCapacity(minCapacity);
    // minCapacity is usually close to size, so this is a win:
    elementData = Arrays.copyOf(elementData, newCapacity);
}
```

**int newCapacity = oldCapacity + (oldCapacity >> 1),所以ArrayList每次扩容之后容量都会变为原来的1.5倍左右（oldCapacity为偶数就是1.5倍，否则是1.5倍左右）**。奇偶不同，比如：10+10/2 = 15,33+33/2=49。如果是奇数的话会丢掉小数.

> ">>"（移位运算符）：>>1右移一位相当于除2，右移n位相当于除以2的n次方。这里oldCapacity明显右移了1位所以相当于oldCapacity/2。对于大数据的2进制运算,位移运算符比那些普通运算符的运算要快很多,因为程序仅仅移动一下而已,不去计算,这样提高了效率,节省了资源

**我们再来通过例子探究一下grow()方法**：

- 当add第1个元素时，oldCapacity为0，经比较后第一个if判断成立，newCapacity = minCapacity(为10)。但是第二个if判断不会成立，即newCapacity不比MAX_ARRAY_SIZE大，则不会进入hugeCapacity方法。数组容量为10，add方法中return true,size增为1。
- 当add第11个元素进入grow方法时，newCapacity为15，比minCapacity（为11）大，第一个if判断不成立。新容量没有大于数组最大size，不会进入hugeCapacity方法。数组容量扩为15，add方法中return true,size增为11。
- 以此类推······

**这里补充一点比较重要，但是容易被忽视掉的知识点**：

- java中的length属性是针对数组说的,比如说你声明了一个数组,想知道这个数组的长度则用到了length这个属性。
- java中的length()方法是针对字符串说的,如果想看这个字符串的长度则用到length()这个方法。
- java中的size()方法是针对泛型集合说的,如果想看这个泛型有多少个元素,就调用此方法来查看。

**hugeCapacity()方法**

从上面grow()方法源码我们知道：如果新容量大于MAX_ARRAY_SIZE,进入(执行)hugeCapacity()方法来比较minCapacity和MAX_ARRAY_SIZE，如果minCapacity大于最大容量，则新容量则为Integer.MAX_VALUE，否则，新容量大小则为MAX_ARRAY_SIZE即为Integer.MAX_VALUE - 8。

```java
private static int hugeCapacity(int minCapacity) {
    if (minCapacity < 0) // overflow
        throw new OutOfMemoryError();
    //对minCapacity和MAX_ARRAY_SIZE进行比较
    //若minCapacity大，将Integer.MAX_VALUE作为新数组的大小
    //若MAX_ARRAY_SIZE大，将MAX_ARRAY_SIZE作为新数组的大小
    //MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
    return (minCapacity > MAX_ARRAY_SIZE) ?
        Integer.MAX_VALUE :
    MAX_ARRAY_SIZE;
}
```

**System.arraycopy()和Arrays.copyOf()方法**

阅读源码的话，我们就会发现ArrayList中大量调用了这两个方法。比如：我们上面讲的扩容操作以及add(int index, E element)、toArray()等方法中都用到了该方法

**System.arraycopy()方法**

源码：

```java
// 我们发现arraycopy是一个native方法,接下来我们解释一下各个参数的具体意义
/**
  * 复制数组
  * @param src 源数组
  * @param srcPos 源数组中的起始位置
  * @param dest 目标数组
  * @param destPos 目标数组中的起始位置
  * @param length 要复制的数组元素的数量
  */
public static native void arraycopy(Object src,  int  srcPos,
                                    Object dest, int destPos,
                                    int length);
```

场景：

```java
/**
   * 在此列表中的指定位置插入指定的元素。
   * 先调用rangeCheckForAdd对index进行界限检查；然后调用ensureCapacityInternal方法保证capacity足够大；
   * 再将从index开始之后的所有成员后移一个位置；将element插入index位置；最后size加1。
   */
public void add(int index, E element) {
    rangeCheckForAdd(index);
    
    ensureCapacityInternal(size + 1);  // Increments modCount!!
    //arraycopy()方法实现数组自己复制自己
    //elementData:源数组;index:源数组中的起始位置;elementData：目标数组；index + 1：目标数组中的起始位置；size - index：要复制的数组元素的数量；
    System.arraycopy(elementData, index, elementData, index + 1, size - index);
    elementData[index] = element;
    size++;
}
```

我们写一个简单的方法测试以下：

```java
public class ArraycopyTest {
	public static void main(String[] args) {
		// TODO Auto-generated method stub
		int[] a = new int[10];
		a[0] = 0;
		a[1] = 1;
		a[2] = 2;
		a[3] = 3;
		System.arraycopy(a, 2, a, 3, 3);
		a[2]=99;
		for (int i = 0; i < a.length; i++) {
			System.out.print(a[i] + " ");
		}
	}
}
```

结果：0 1 99 2 3 0 0 0 0 0

**Arrays.copyOf()方法**

源码：
```java
public static int[] copyOf(int[] original, int newLength) {
    // 申请一个新的数组
    int[] copy = new int[newLength];
    // 调用System.arraycopy,将源数组中的数据进行拷贝,并返回新的数组
    System.arraycopy(original, 0, copy, 0,
                     Math.min(original.length, newLength));
    return copy;
}
```

场景：
```java
/**
  * 以正确的顺序返回一个包含此列表中所有元素的数组（从第一个到最后一个元素）;返回的数组的运行时类型是指定数组的运行时类型。
  */
public Object[] toArray() {
    //elementData：要复制的数组；size：要复制的长度
    return Arrays.copyOf(elementData, size);
}
```

个人觉得使用Arrays.copyOf()方法主要是为了给原有数组扩容，测试代码如下：

```java
public class ArrayscopyOfTest {
	public static void main(String[] args) {
		int[] a = new int[3];
		a[0] = 0;
		a[1] = 1;
		a[2] = 2;
		int[] b = Arrays.copyOf(a, 10);
		System.out.println("b.length"+b.length);
	}
}
```

结果：10

**两者联系和区别**

**联系**：
看两者源代码可以发现`copyOf()`内部实际调用了`System.arraycopy()`方法

**区别**：
`arraycopy()`需要目标数组，将原数组拷贝到你自己定义的数组里或者原数组，而且可以选择拷贝的起点和长度以及放入新数组中的位置,`copyOf()`是系统自动在内部新建一个数组，并返回该数组。

**ensureCapacity()方法**

ArrayList源码中有一个ensureCapacity方法，这个方法ArrayList内部没有被调用过，所以很显然是提供给用户调用的，那么这个方法有什么作用呢？

```java
/**
  * 如有必要，增加此ArrayList实例的容量，以确保它至少可以容纳由minimum capacity参数指定的元素数。
  * @param   minCapacity   所需的最小容量
  */
public void ensureCapacity(int minCapacity) {
    int minExpand = (elementData != DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
        // any size if not default element table
        ? 0
        // larger than default for default empty table. It's already
        // supposed to be at default size.
        : DEFAULT_CAPACITY;

    if (minCapacity > minExpand) {
        ensureExplicitCapacity(minCapacity);
    }
}
```
理论上来说，最好在向ArrayList添加大量元素之前用ensureCapacity方法，以减少增量重新分配的次数。我们通过下面的代码实际测试以下这个方法的效果：

```java
public class EnsureCapacityTest {
	public static void main(String[] args) {
		ArrayList<Object> list = new ArrayList<Object>();
		final int N = 10000000;
		long startTime = System.currentTimeMillis();
		for (int i = 0; i < N; i++) {
			list.add(i);
		}
		long endTime = System.currentTimeMillis();
		System.out.println("使用ensureCapacity方法前："+(endTime - startTime));

	}
}
```
运行结果：
```text
使用ensureCapacity方法前：2158
```

```java
public class EnsureCapacityTest {
    public static void main(String[] args) {
        ArrayList<Object> list = new ArrayList<Object>();
        final int N = 10000000;
        long startTime1 = System.currentTimeMillis();
        list.ensureCapacity(N);
        for (int i = 0; i < N; i++) {
            list.add(i);
        }
        long endTime1 = System.currentTimeMillis();
        System.out.println("使用ensureCapacity方法后："+(endTime1 - startTime1));
    }
}
```

运行结果：
```text
使用ensureCapacity方法后：1773
```
通过运行结果，我们可以看出向ArrayList添加大量元素之前使用ensureCapacity方法可以提升性能。不过，这个性能差距几乎可以忽略不计。而且，实际项目根本也不可能往ArrayList里面添加这么多元素。

**remove()方法**

假设我们要删除数组的index = 5的元素，首先，我们会复制一遍源数组，然后把源数组中从 index + 1 = 6的位置开始到数组末尾的元素，放到新数组的index = 5的位置上,也就是说 index = 5的元素直接被覆盖掉了，给了你被删除的感觉。同样的，它的效率自然也是十分低下的

> [原文链接](https://javaguide.cn/java/collection/arraylist-source-code.html)

**总结扩容**

- 默认容量：ArrayList的默认初始容量是10。这意味着当你创建一个新的ArrayList实例而不指定初始容量时，它将能够存储最多10个元素，而不需要进行扩容。
- 扩容规则：当需要扩容时，ArrayList会将当前数组的大小增加到“旧容量 + (旧容量 / 2)”，也就是说，新的容量将是旧容量的1.5倍。这个计算方式确保了ArrayList的容量以指数级的速度增长，但每次扩容的成本（即复制元素到新数组）也会相应增加。
- 最大容量：ArrayList有一个MAX_ARRAY_SIZE的常量，这个常量限制了ArrayList可以扩容到的最大容量。在JDK 8中，这个值通常是`Integer.MAX_VALUE - 8`，因为数组还需要一些额外的空间来存储元数据（如数组长度等）。这意味着在大多数情况下，ArrayList可以扩展到非常大的容量，但实际上受限于JVM的堆内存大小。

## CopyOnWriteArrayList

### CopyOnWriteArrayList简介

CopyOnWriteArrayList实际上是ArrayList一个线程安全的操作类,是一个典型的读写分离的动态数组操作类.从它的名字可以看出，CopyOnWrite是在写入的时候，不修改原内容，而是将原来的内容复制一份到新的数组，然后向新数组写完数据之后，再移动内存指针，将目标指向最新的位置。CopyOnWriteArraySet主要针对集，CopyOnWriteArraySet可以理解为HashSet线程安全的操作类，我们都知道HashSet基于散列表HashMap实现，但是CopyOnWriteArraySet并不是基于散列表实现，而是基于CopyOnWriteArrayList动态数组实现。两者最大的不同点是，CopyOnWriteArrayList可以允许元素重复，而CopyOnWriteArraySet不允许有重复的元素。CopyOnWriteArrayList在写入数据的时候，将旧数组内容复制一份出来，然后向新的数组写入数据，最后将新的数组内存地址返回给数组变量；移除操作也类似，只是方式是移除元素而不是添加元素；而查询方法，因为不涉及线程操作，所以并没有加锁出来！因为CopyOnWriteArrayList读取内容没有加锁，在写入数据的时候同时也可以进行读取数据操作，因此性能得到很大的提升，但是也有缺陷，**对于边读边写的情况，不一定能实时的读到最新的数据,因此CopyOnWriteArrayList很适合读多写少的应用场景**


```java
public class CopyOnWriteArrayList<E> extends Object implements List<E>, RandomAccess, Cloneable, Serializable
```

在很多应用场景中，读操作可能会远远大于写操作。由于读操作根本不会修改原有的数据，因此对于每次读取都进行加锁其实是一种资源浪费。我们应该允许多个线程同时访问List的内部数据，毕竟读取操作是安全的。这和我们之前提到过的ReentrantReadWriteLock读写锁的思想非常类似，也就是读读共享、写写互斥、读写互斥、写读互斥。JDK中提供了CopyOnWriteArrayList类比相比于在读写锁的思想又更进一步。为了将读取的性能发挥到极致，CopyOnWriteArrayList读取是完全不用加锁的，并且更厉害的是：写入也不会阻塞读取操作。只有写入和写入之间需要进行同步等待。这样一来，读操作的性能就会大幅度提升。

### CopyOnWriteArrayList是如何做到的？

CopyOnWriteArrayList类的所有可变操作（add，set等等）都是通过创建底层数组的新副本来实现的。当List需要被修改的时候，我并不修改原有内容，而是对原有数据进行一次复制，将修改的内容写入副本。写完之后，再将修改完的副本替换原来的数据，这样就可以保证写操作不会影响读操作了。从CopyOnWriteArrayList的名字就能看出CopyOnWriteArrayList是满足CopyOnWrite的。所谓CopyOnWrite也就是说：在计算机，如果你想要对一块内存进行修改时，我们不在原有内存块中进行写操作，而是将内存拷贝一份，在新的内存中进行写操作，写完之后呢，就将指向原来内存指针指向新的内存，原来的内存就可以被回收掉了。

### CopyOnWriteArrayList读取和写入源码简单分析

**CopyOnWriteArrayList读取操作的实现**

读取操作没有任何同步控制和锁操作，理由就是内部数组array不会发生修改，只会被另外一个array替换，因此可以保证数据安全。


```java
/** The array, accessed only via getArray/setArray. */
private transient volatile Object[] array;
public E get(int index) {
    return get(getArray(), index);
}
@SuppressWarnings("unchecked")
private E get(Object[] a, int index) {
    return (E) a[index];
}
final Object[] getArray() {
    return array;
}
```

**CopyOnWriteArrayList写入操作的实现**

CopyOnWriteArrayList写入操作add()方法在添加集合的时候加了锁，保证了同步，避免了多线程写的时候会copy出多个副本出来。


```java
/**
  * Appends the specified element to the end of this list.
  * @param e element to be appended to this list
  * @return {@code true} (as specified by {@link Collection#add})
  */
public boolean add(E e) {
    final ReentrantLock lock = this.lock;
    lock.lock();//加锁
    try {
        Object[] elements = getArray();
        int len = elements.length;
        Object[] newElements = Arrays.copyOf(elements, len + 1);//拷贝新数组
        newElements[len] = e;
        setArray(newElements);
        return true;
    } finally {
        lock.unlock();//释放锁
    }
}
```

### 常用方法

**add()**

add()方法是CopyOnWriteArrayList的添加元素的入口,CopyOnWriteArrayList之所以能保证多线程下安全操作，add()方法功不可没,操作步骤如下：
- 获得对象锁
- 获取数组内容
- 将原数组内容复制到新数组
- 写入数据
- 将array数组变量地址指向新数组
- 释放对象锁
CopyOnWriteArrayList使用了ReentrantLock这种可重入锁，保证了线程操作安全，同时数组变量array使用volatile保证多线程下数据的可见性

**remove()**

remove()方法是CopyOnWriteArrayList的移除元素的入口
操作类似添加方法，步骤如下：
- 获得对象锁
- 获取数组内容
- 判断移除的元素是否为数组最后的元素，如果是最后的元素，直接将旧元素内容复制到新数组，并重新设置array值
- 如果是中间元素，以index为分界点，分两节复制
- 将array数组变量地址指向新数组
- 释放对象锁
当然，移除的方法还有基于对象的remove(Object o)，原理也是一样的，先找到元素的下标，然后执行移除操作。

**get()**

get()方法是CopyOnWriteArrayList的查询元素的入口,源码如下：

```java
public E get(int index) {
    //获取数组内容，通过下标直接获取
    return get(getArray(), index);
}
```
查询因为不涉及到数据操作，所以无需使用锁进行处理！

**iterator()**

CopyOnWriteArrayList在使用迭代器遍历的时候，操作的都是原数组，没有像上面那样进行修改次数判断，所以不会抛异常,当然，从源码上也可以得出，使用CopyOnWriteArrayList的迭代器进行遍历元素的时候，不能调用remove()方法移除元素，因为不支持此操作,如果想要移除元素，只能使用CopyOnWriteArrayList提供的remove()方法，而不是迭代器的remove()方法，这个需要注意一下

