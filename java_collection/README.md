<!-- TOC -->

- [概述](#概述)
    - [Collection](#collection)
        - [1. Set](#1-set)
        - [2. List](#2-list)
        - [3. Queue](#3-queue)
    - [Map](#map)
- [容器中设计模式](#容器中设计模式)
    - [迭代器模式](#迭代器模式)
    - [适配器模式](#适配器模式)
- [源码分析](#源码分析)
    - [ArrayList](#arraylist)
        - [1.概述](#1概述)
        - [2.扩容](#2扩容)
        - [3.删除元素](#3删除元素)
        - [4.序列化](#4序列化)
        - [5.Fail-Fast](#5fail-fast)
    - [Vector](#vector)
        - [1.同步](#1同步)
        - [2.扩容](#2扩容-1)
        - [3.与ArrayList比较](#3与arraylist比较)
        - [替代方案](#替代方案)
    - [CopyOnWriteArrayList](#copyonwritearraylist)
        - [1.读写分离](#1读写分离)
        - [2.适用场景](#2适用场景)
    - [LinkedList](#linkedlist)
        - [1.概述](#1概述-1)
        - [2.与ArrayList比较](#2与arraylist比较)
    - [HashMap](#hashmap)
        - [1. 存储结构](#1-存储结构)
        - [2.拉链法工作原理](#2拉链法工作原理)
        - [3.put操作](#3put操作)
        - [4.确定桶下标](#4确定桶下标)
        - [5.扩容原理](#5扩容原理)
        - [6.扩容-重新计算桶下标](#6扩容-重新计算桶下标)
        - [7.计算数组容量](#7计算数组容量)
        - [8.链表转红黑树](#8链表转红黑树)
        - [9.与Hashtable比较](#9与hashtable比较)
    - [ConcurrentHashMap](#concurrenthashmap)
        - [1.存储结构](#1存储结构)
        - [2.size操作](#2size操作)
        - [3.JDK 1.8改动](#3jdk-18改动)
    - [LinkedHashMap](#linkedhashmap)
        - [1.存储结构](#1存储结构-1)
        - [2.afterNodeAccess()](#2afternodeaccess)
        - [3.afterNodeInsertion()](#3afternodeinsertion)
        - [LRU缓存](#lru缓存)
    - [WeakHashMap](#weakhashmap)
        - [1.存储结构](#1存储结构-2)
        - [2.ConcurrentCache](#2concurrentcache)

<!-- /TOC -->
# 概述
容器主要包括 Collection 和 Map 两种，Collection 存储着对象的集合，而 Map 存储着键值对（两个对象）的映射表。
## Collection
- Collection
    - Set
        - SortedSet
            - TreeSet
        - HashSet
        - LinkedHashSet
    - List
        - ArrayList
        - LinkedList
        - Vector
    - Queue
        - LinkedList
        - PrioityQueue
### 1. Set
- TreeSet：基于红黑树实现，支持有序性操作，例如根据一个范围查找元素的操作。但是查找效率不如 HashSet，HashSet 查找的时间复杂度为 O(1)，TreeSet 则为 O(logN)。

- HashSet：基于哈希表实现，支持快速查找，但不支持有序性操作。并且失去了元素的插入顺序信息，也就是说使用 Iterator 遍历 HashSet 得到的结果是不确定的。

- LinkedHashSet：具有 HashSet 的查找效率，并且内部使用双向链表维护元素的插入顺序。

### 2. List
- ArrayList：基于动态数组实现，支持随机访问。
- Vector：和 ArrayList 类似，但它是线程安全的。
- LinkedList：基于双向链表实现，只能顺序访问，但是可以快速地在链表中间插入和删除元素。不仅如此，LinkedList 还可以用作栈、队列和双向队列。

[SynchronizedList与Vector区别](https://github.com/hollischuang/toBeTopJavaer/blob/master/basics/java-basic/synchronizedlist-vector.md)

### 3. Queue
- LinkedList：可以用它来实现双向队列。
- PriorityQueue：基于堆结构实现，可以用它来实现优先队列。

## Map
- Map
    - SortedMap
        - TreeMap
    - HashMap
    - HashTable
    - LinkedHashMap

- TreeMap：基于红黑树实现。
- HashMap：基于哈希表实现。
- HashTable：和 HashMap 类似，但它是线程安全的，这意味着同一时刻多个线程同时写入 HashTable 不会导致数据不一致。它是遗留类，不应该去使用它，而是使用 ConcurrentHashMap 来支持线程安全，ConcurrentHashMap 的效率会更高，因为 ConcurrentHashMap 引入了分段锁。
- LinkedHashMap：使用双向链表来维护元素的顺序，顺序为插入顺序或者最近最少使用（LRU）顺序。
# 容器中设计模式
## 迭代器模式
Collection 继承了 Iterable 接口，其中的 iterator() 方法能够产生一个 Iterator 对象，通过这个对象就可以迭代遍历 Collection 中的元素。

## 适配器模式

# 源码分析
JDK 1.8版本
## ArrayList
### 1.概述
ArrayList基于数组实现的饿，支持快速随机访问。ArrayList实现了RandomAccess接口标识着支持快速随机访问。

数组默认大小为10。

### 2.扩容
添加元素时使用 ```ensureCapacityInternal()``` 方法来保证容量足够，如果不够时，需要使用 grow() 方法进行扩容，新容量的大小为```oldCapacity + (oldCapacity >> 1)```，也就是旧容量的 1.5 倍。

扩容操作需要调用 ```Arrays.copyOf()``` 把原数组整个复制到新数组中，这个操作代价很高，因此最好在创建 ArrayList 对象时就指定大概的容量大小，减少扩容操作的次数。

### 3.删除元素
会调用```System.arraycopy()```将index+1后面饿元素都复制到index位置上，该操作单饿时间复杂度为O(N)，可以看到ArrayList删除元素的代价是非常高的。
```
/**
     * Removes the element at the specified position in this list.
     * Shifts any subsequent elements to the left (subtracts one from their
     * indices).
     *
     * @param index the index of the element to be removed
     * @return the element that was removed from the list
     * @throws IndexOutOfBoundsException {@inheritDoc}
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

        return oldValue;
    }
```
### 4.序列化
ArrayList基于数组实现的，而且可以动态扩容，所以保存的元素并不一定都会被使用，就没有必要全部都序列化。
```
transient Object[] elementData; // non-private to simplify nested class access
```
其中实现中用的elementData被transient修饰，默认数组被声明不会被序列化。

ArrayList 实现了 writeObject() 和 readObject() 来控制只序列化数组中有元素填充那部分内容。

### 5.Fail-Fast
[java集合中常见异常fail-fast（面试常问）](https://baijiahao.baidu.com/s?id=1638201147057831295&wfr=spider&for=pc)

## Vector
### 1.同步
使用了synchronized同步
### 2.扩容
扩容翻倍
### 3.与ArrayList比较
- Vector 是同步的，因此开销就比 ArrayList 要大，访问速度更慢。最好使用 ArrayList 而不是 Vector，因为同步操作完全可以由程序员自己来控制；

- Vector 每次扩容请求其大小的 2 倍（也可以通过构造函数设置增长的容量），而 ArrayList 是 1.5 倍。

### 替代方案
- Collections.synchronizedList()
- CopyOnWriteArrayList
## CopyOnWriteArrayList
参考

[CopyOnWriteArrayList](https://www.jianshu.com/p/ce7f203731af)


### 1.读写分离
读在原始数组中进行，写在复制的数组上进行，读写分离互不影响。

写操作小加锁，防止并发写入时导致写入数据丢失

写操作结束后需要把原始数组指向新的复制数组
### 2.适用场景
CopyOnWriteArrayList 在写操作的同时允许读操作，大大提高了读操作的性能，因此很适合读多写少的应用场景。

但是 CopyOnWriteArrayList 有其缺陷：

- 内存占用：在写操作时需要复制一个新的数组，使得内存占用为原来的两倍左右；
- 数据不一致：读操作不能读取实时性的数据，因为部分写操作的数据还未同步到读数组中。

所以 CopyOnWriteArrayList 不适合内存敏感以及对实时性要求很高的场景。

## LinkedList
### 1.概述
双向链表实现，使用Node存储链表节点信息
```
private static class Node<E> {
    E item;
    Node<E> next;
    Node<E> prev;
}
```
每个链表存储了first和last指针
```
transient Node<E> first;
transient Node<E> last;
```
### 2.与ArrayList比较
ArrayList基于动态数组实现，LinkedList基于双向链表实现。
- 数组支持随机访问，但插入删除的代价很大，需要移动大量元素
- 链表不支持随机访问，但插入删除只需要改变指针

## HashMap
### 1. 存储结构
内部包含了一个 Entry 类型的数组 table。Entry 存储着键值对。它包含了四个字段，从 next 字段我们可以看出 Entry 是一个链表。即数组中的每个位置被当成一个桶，一个桶存放一个链表。HashMap 使用拉链法来解决冲突，同一个链表中存放哈希值和散列桶取模运算结果相同的 Entry。
```
transient Entry[] table;
```
```
static class Entry<K,V> implements Map.Entry<K,V> {
    final K key;
    V value;
    Entry<K,V> next;
    int hash;

    Entry(int h, K k, V v, Entry<K,V> n) {
        value = v;
        next = n;
        key = k;
        hash = h;
    }

    public final K getKey() {
        return key;
    }

    public final V getValue() {
        return value;
    }

    public final V setValue(V newValue) {
        V oldValue = value;
        value = newValue;
        return oldValue;
    }

    public final boolean equals(Object o) {
        if (!(o instanceof Map.Entry))
            return false;
        Map.Entry e = (Map.Entry)o;
        Object k1 = getKey();
        Object k2 = e.getKey();
        if (k1 == k2 || (k1 != null && k1.equals(k2))) {
            Object v1 = getValue();
            Object v2 = e.getValue();
            if (v1 == v2 || (v1 != null && v1.equals(v2)))
                return true;
        }
        return false;
    }

    public final int hashCode() {
        return Objects.hashCode(getKey()) ^ Objects.hashCode(getValue());
    }

    public final String toString() {
        return getKey() + "=" + getValue();
    }
}
```
### 2.拉链法工作原理
```
HashMap<String, String> map = new HashMap<>();
map.put("K1", "V1");
map.put("K2", "V2");
map.put("K3", "V3");
```
链表的插入是以头插法方式进行的。

查找需要分成两步进行：
- 计算键值对所在的桶；
- 在链表上顺序查找，时间复杂度显然和链表的长度成正比。
### 3.put操作
- 处理null的key
- 确定桶下标
- 判断是否存才key的键值对，存在的话更新键值对
- 插入键值对

HashMap 允许插入键为 null 的键值对。但是因为无法调用 null 的 hashCode() 方法，也就无法确定该键值对的桶下标，只能通过强制指定一个桶下标来存放。HashMap 使用第 0 个桶存放键为 null 的键值对。

使用链表的头插法，也就是新的键值对插在链表的头部，而不是链表的尾部。
### 4.确定桶下标
1、计算hash值

2、取模

### 5.扩容原理
### 6.扩容-重新计算桶下标
### 7.计算数组容量
### 8.链表转红黑树
JDK 1.8开始，一个桶存储的链表大于等于8时会将链表转换为红黑树

### 9.与Hashtable比较
- Hashtable 使用 synchronized 来进行同步。
- HashMap 可以插入键为 null 的 Entry。
- HashMap 的迭代器是 fail-fast 迭代器。
- HashMap 不能保证随着时间的推移 Map 中的元素次序是不变的。

[面试必备：HashMap、Hashtable、ConcurrentHashMap的原理与区别](https://www.cnblogs.com/heyonggang/p/9112731.html)

[吊打面试官HashMap、ConcurrentHashMap](https://msd.misuland.com/pd/3691885065085649392?page=1)

## ConcurrentHashMap
### 1.存储结构
ConcurrentHashMap 和 HashMap 实现上类似，最主要的差别是 ConcurrentHashMap 采用了分段锁（Segment），每个分段锁维护着几个桶（HashEntry），多个线程可以同时访问不同分段锁上的桶，从而使其并发度更高（并发度就是 Segment 的个数）。

Segment 继承自 ReentrantLock。

```
static final class Segment<K,V> extends ReentrantLock implements Serializable {

    private static final long serialVersionUID = 2249069246763182397L;

    static final int MAX_SCAN_RETRIES =
        Runtime.getRuntime().availableProcessors() > 1 ? 64 : 1;

    transient volatile HashEntry<K,V>[] table;

    transient int count;

    transient int modCount;

    transient int threshold;

    final float loadFactor;
}
```
```
final Segment<K,V>[] segments;
```
默认的并发级别为 16，也就是说默认创建 16 个 Segment。
```
static final int DEFAULT_CONCURRENCY_LEVEL = 16;
```
### 2.size操作
每个 Segment 维护了一个 count 变量来统计该 Segment 中的键值对个数。

### 3.JDK 1.8改动
JDK 1.7 使用分段锁机制来实现并发更新操作，核心类为 Segment，它继承自重入锁 ReentrantLock，并发度与 Segment 数量相等。

JDK 1.8 使用了 CAS 操作来支持更高的并发度，在 CAS 操作失败时使用内置锁 synchronized。

并且 JDK 1.8 的实现也在链表过长时会转换为红黑树。

## LinkedHashMap
### 1.存储结构
继承自 HashMap，因此具有和 HashMap 一样的快速查找特性。
```
public class LinkedHashMap<K,V> extends HashMap<K,V> implements Map<K,V>
```
双向链表，用来维护插入顺序或者 LRU 顺序。
```
/**
 * The head (eldest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> head;

/**
 * The tail (youngest) of the doubly linked list.
 */
transient LinkedHashMap.Entry<K,V> tail;
```
accessOrder 决定了顺序，默认为 false，此时维护的是插入顺序。
```
final boolean accessOrder;
```
LinkedHashMap 最重要的是以下用于维护顺序的函数，它们会在 put、get 等方法中调用。
```
void afterNodeAccess(Node<K,V> p) { }
void afterNodeInsertion(boolean evict) { }
```
### 2.afterNodeAccess()
当一个节点被访问时，如果 accessOrder 为 true，则会将该节点移到链表尾部。也就是说指定为 LRU 顺序之后，在每次访问一个节点时，会将这个节点移到链表尾部，保证链表尾部是最近访问的节点，那么链表首部就是最近最久未使用的节点。
### 3.afterNodeInsertion()
在 put 等操作之后执行，当 removeEldestEntry() 方法返回 true 时会移除最晚的节点，也就是链表首部节点 first。

evict 只有在构建 Map 的时候才为 false，在这里为 true。
```
void afterNodeInsertion(boolean evict) { // possibly remove eldest
    LinkedHashMap.Entry<K,V> first;
    if (evict && (first = head) != null && removeEldestEntry(first)) {
        K key = first.key;
        removeNode(hash(key), key, null, false, true);
    }
}
```
removeEldestEntry() 默认为 false，如果需要让它为 true，需要继承 LinkedHashMap 并且覆盖这个方法的实现，这在实现 LRU 的缓存中特别有用，通过移除最近最久未使用的节点，从而保证缓存空间足够，并且缓存的数据都是热点数据。
```
protected boolean removeEldestEntry(Map.Entry<K,V> eldest) {
    return false;
}
```
### LRU缓存
使用LinkedHashMap实现一个LRU缓存

## WeakHashMap
### 1.存储结构
WeakHashMap 的 Entry 继承自 WeakReference，被 WeakReference 关联的对象在下一次垃圾回收时会被回收。

WeakHashMap 主要用来实现缓存，通过使用 WeakHashMap 来引用缓存对象，由 JVM 对这部分缓存进行回收。
```
private static class Entry<K,V> extends WeakReference<Object> implements Map.Entry<K,V>
```
### 2.ConcurrentCache
Tomcat 中的 ConcurrentCache 使用了 WeakHashMap 来实现缓存功能。

ConcurrentCache 采取的是分代缓存：

- 经常使用的对象放入 eden 中，eden 使用 ConcurrentHashMap 实现，不用担心会被回收（伊甸园）；

- 不常用的对象放入 longterm，longterm 使用 WeakHashMap 实现，这些老对象会被垃圾收集器回收。

- 当调用 get() 方法时，会先从 eden 区获取，如果没有找到的话再到 longterm 获取，当从 longterm 获取到就把对象放入 eden 中，从而保证经常被访问的节点不容易被回收。

- 当调用 put() 方法时，如果 eden 的大小超过了 size，那么就将 eden 中的所有对象都放入 longterm 中，利用虚拟机回收掉一部分不经常使用的对象。






