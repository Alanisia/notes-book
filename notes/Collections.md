# 容器

## Map

### HashMap

#### 实现原理

HashMap底层为Entry数组+链表(JDK7)/Node数组+链表+红黑树(JDK8)。

`put(Object key, Object value)`：先计算key的hashCode，然后将其与数组长度-1的差值相与得到hash值，该hash值确认在数组的存储位置。若该位置无元素，直接插入；否则遍历该位置下的链表/红黑树，依次比较各个元素的key和hashCode，如果两个key相等则用新的value覆盖原来的value；否则将该节点插入链表/红黑树。如果此时的数组元素数量已经达到了容量与负载因子的乘积，应先扩容，再插入元素。

`get(Object key)`：先计算key的hashCode，然后将其与数组长度-1的差值相与得到hash值，确认其在数组的位置，在那个位置找到与key相等的元素，返回元素的value。

`resize()`：数组容量翻倍。扩容后，原数组中的所有元素需要找到在新数组中的位置。

_注：JDK7以前使用拉链法解决冲突，JDK8以后改为当链表长度大于阈值（默认为8）且数组长度大于64时（小于64时会选择将数组进行扩容）转化为红黑树。_

相关问题：

1. 为什么HashMap的数组长度须为2的整数次幂？

    为了能让HashMap存储高效，尽量减少碰撞，尽量让数组分配均匀。利用key的散列值对数组的长度进行取模运算可以得到数组的下标，当数组长度为2的整数次幂时，key的散列值对数组长度取余等价于key的散列值与数组长度减1的值相与。由于采用二进制位与操作，相对于%能够提高效率。

2. 为什么默认加载因子为0.75？

    加载因子表示哈希表中元素的填满程度。加载因子越大，填满的元素越多，空间利用率高，但发生冲突的几率也增加；加载因子越小，填满的元素越少，冲突发生几率减小，但空间浪费多，还会提高扩容时rehash的次数。

    在理想情况下，使用随机哈希码，在加载因子为0.75的情况下，节点出现的频率在哈希表中遵循参数平均为0.5的泊松分布。

    选择0.75作为加载因子，完全是时间和空间成本上寻求的一种折衷选择。

3. 为什么说HashMap线程不安全？

    主要原因在于并发条件下的rehash会形成一个循环链表。JDK8解决了此问题，不过在并发条件下使用HashMap还是会存在其他问题诸如数据丢失。

4. 为什么选择将链表长度为8作为将链表转换为红黑树的阈值？

    通常如果哈希算法正常，那么链表长度也不会很长，那么红黑树也不会带来明显的查询时间上的优势，反而会增加空间负担，所以通常情况下没有必要转化为红黑树，所以就选择了概率非常小，也就是长度为8的概率（泊松分布），将长度为8作为转换阈值。

#### HashMap与HashTable的区别

1. HashMap线程不安全，HashTable线程安全；
2. 因为线程安全问题，HashMap效率高于HashTable；
3. HashMap可以存储值为null的key和value，但null的key只能有一个，null的value可以有多个；HashTable不允许出现值为null的key或
value（会抛出NullPointerException）；
4. JDK8以后的HashMap在链表长度大于阈值时将链表转化为红黑树，HashTable没有这样的机制；
5. 若不指定初始容量，HashMap会默认初始容量为16，以后每次扩容容量翻倍，而HashTable默认初始容量为11，以后每次扩容容量变为原来的2n+1；若指定初始容量，HashTable会直接使用指定的容量，而HashMap会将其扩充为2的整数次幂。

_注：不要在代码中使用HashTable，如果需要使用能保证并发安全的HashMap请使用ConcurrentHashMap。_

### HashSet

底层是HashMap。如何检查重复：

当把元素加入HashSet时，HashSet会先计算对象的hashCode值来判断元素加入的位置，同时也会与其他元素的hashCode值相比较。若没有相同的hashCode，hashSet会假设元素没有重复出现，否则调用equals方法来检查hashCode相等的对象是否真的相等。若两者相等则HashSet不会让加入操作成功。

### `hashCode()`和`equals()`相关

1. 如果两个对象相等，则hashCode相等
2. 两个对象相等，`equals()`返回`true`
3. 两个对象hashCode相等，两个对象不一定相等
4. 综上，`equals()`被覆盖，`hashCode()`必须被覆盖
5. `hashCode()`默认行为是对堆上的对象产生独特值。若不重写`hashCode()`则两个对象无论如何都不会相等

### ConcurrentHashMap

#### 实现原理

JDK7和JDK8以后的ConcurrentHashMap实现底层不同。

- JDK7

    首先将数组分割成一段一段，每段配一把锁，当一个线程访问其中一个段的数据时，其他段的数据能被其他线程访问。

    ConcurrentHashMap由Segment数组和HashEntry数据结构构成。Segment实现了ReentrantLock，所以Segment是一种可重入锁，HashEntry用于存储键值对数据：

    ```java
    static class Segment<K, V> extends ReentrantLock implements Serializable {}
    ```

    一个ConcurrentHashMap包含一个Segment数组，Segment结构与HashMap类似，是一种数组+链表的结构，一个Segment包含一个HashEntry数组，每个HashEntry是链表结构的元素，每个Segment守护着一个HashEntry数组的元素，当对HashEntry数组进行修改时须获得对应的Segment的锁。

- JDK8

    ConcurrentHashMap取消了Segment分段锁，采用CAS和synchronized来保证并发安全，数据结构与JDK8的HashMap类似，都是数组+链表/红黑树。在链表长度大于阈值（默认为8）时将链表转换为红黑树。

    synchronized只锁当前的链表或者红黑树首节点。只要不发生hash冲突，就不会产生并发。

#### ConcurrentHashMap与HashTable的区别

1. HashTable底层采用数组+链表实现，JDK7的ConcurrentHashMap底层采用分段数组+链表实现，JDK8以后改为Node数组+链表+红黑树。

2. 实现线程安全的方式不同：

    1. JDK7的ConcurrentHashMap采用分段锁(`Segment`)对整个桶数组进行分割分段，每一把锁只锁容器其中一部分数据，多线程访问容器里不同数据段的数据，就不会存在锁竞争，提高并发访问率。但JDK8以后弃用分段锁的概念，而直接使用Node数组+链表+红黑树的数据结构来实现，并发控制使用synchronized和CAS来操作，整个看起来就像是优化过且线程安全的HashMap。即使在JDK8中还能看到Segment，但已经简化了属性，只是为了兼容旧版本。
    2. HashTable使用synchronized来确保线程安全，但效率低下。当一个线程访问同步方法时，其他线程也访问同步方法可能会进入阻塞或轮询的状态（对整张表加锁），竞争激烈，并发效率低。

## List

### ArrayList，Vector，LinkedList区别

1. ArrayList底层基于数组实现，查找快，增删慢；LinkedList底层基于双向链表实现（JDK6以前是双向循环链表，JDK7以后取消循环），查找慢，增删快；

2. ArrayList、LinkedList皆为线程不安全集合，Vector线程安全。

### ArrayList与数组的区别

1. 数组可以容纳基本数据类型的数据和对象，ArrayList只能容纳对象；
2. 数组大小可以自行指定，ArrayList不能。

### RandomAccess接口

ArrayList实现了该接口，而LinkedList没有实现。

ArrayList实现该接口的作用：

实现该接口的集合支持快速随机访问。实现了该接口的集合一般采用for循环遍历，而未实现该接口的集合一般采用迭代器进行遍历。ArrayList用for遍历要比用迭代器快，而LinkedList则是反过来。

### ArrayList扩容机制

***TODO***

## 迭代器（Iterator）

***TODO***

## fail-fast

***TODO***

## fail-safe

***TODO***