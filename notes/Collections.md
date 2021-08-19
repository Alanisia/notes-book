# 容器

## Map

### HashMap

#### 实现原理

HashMap底层为Entry数组+链表(JDK7)/Node数组+链表+红黑树(JDK8)。

`put(Object key, Object value)`：先计算key的hashCode，然后将其与数组长度-1的差值相与得到hash值，该hash值确认在数组的存储位置。若该位置无元素，直接插入；否则遍历该位置下的链表/红黑树，依次比较各个元素的key和hashCode，如果两个key相等则用新的value覆盖原来的value；否则将该节点插入链表/红黑树。如果此时的数组元素数量已经达到了容量与负载因子的乘积，应先扩容，再插入元素。

`get(Object key)`：先计算key的hashCode，然后将其与数组长度-1的差值相与得到hash值，确认其在数组的位置，在那个位置找到与key相等的元素，返回元素的value。

`resize()`：数组容量翻倍。扩容后，原数组中的所有元素需要找到在新数组中的位置。

#### HashMap与HashTable的区别

1. HashMap线程不安全，HashTable线程安全；
2. 因为线程安全问题，HashMap效率高于HashTable；
3. HashMap可以存储值为null的key和value，但null的key只能有一个，null的value可以有多个；HashTable不允许出现值为null的key或
value（会抛出NullPointerException）；
4. JDK8以后的HashMap在链表长度大于阙值时将链表转化为红黑树，HashTable没有这样的机制；
5. 若不指定初始容量，HashMap会默认初始容量为16，以后每次扩容容量翻倍，而HashTable默认初始容量为11，以后每次扩容容量变为原来的2n+1；若指定初始容量，HashTable会直接使用指定的容量，而HashMap会将其扩充为2的整数次幂。

_注：不要在代码中使用HashTable，如果需要使用能保证并发安全的HashMap请使用ConcurrentHashMap。_

### HashSet

底层是HashMap。


### ConcurrentHashMap

#### 实现原理

#### ConcurrentHashMap与HashTable的区别

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

## 迭代器（Iterator）

## fail-fast

## fail-safe

