# 容器

## Map

### HashMap

#### 实现原理

HashMap底层为Entry数组+链表(JDK7)/Node数组+链表+红黑树(JDK8)。

初始化默认容量为16，加载因子为0.75。若初始化指定容量，则容量大小会扩充为大于该容量值的2的最小整数幂。

```java
public Object put(Object key, Object value);
```

JDK1.7：

- 判断数组是否为空，为空则进行初始化；
- 若不为空则计算key的hash值，通过`h & (length - 1)`计算应当存放在数组中的下标`index`；
- 查看`table[index]`是否存在数据，没有数据就构造一个`Entry`节点存放在`table[index]`中；
- 存在数据，说明发生了hash冲突，继续判断key是否相等，若相等，用新的value替换原数据；
- 若不相等，创建`Entry`节点加入链表中（头插法）；
- 插入前判断当前节点数是否大于阈值（数组容量与加载因子的乘积），如果是则开始扩容为原数组的2倍，再插入节点。

JDK1.8：

- 判断数组是否为空，为空则进行初始化；
- 若不为空则计算key的hash值，通过`h & (length - 1)`计算应当存放在数组中的下标`index`；
- 查看`table[index]`是否存在数据，没有数据就构造一个`Node`节点存放在`table[index]`中；
- 存在数据，说明发生了hash冲突，继续判断key是否相等，若相等，用新的value替换原数据（`onlyIfAbsent`为`false`）；
- 若不相等，判断当前节点类型是否为红黑树节点，如果是则创建红黑树节点插入到树中；
- 如果不是红黑树节点则创建普通`Node`加入链表中（尾插法）；
- 判断链表长度是否大于8并且数组长度是否大于64，若是则将链表转为红黑树；
- 插入完成后判断当前节点数是否大于阈值（数组容量与加载因子的乘积），如果是则开始扩容为原数组的2倍。

```java
public Object get(Object key);
```

- 计算key的hash值，通过`h & (length - 1)`计算应当存放在数组中的下标`index`；
- 对链表/红黑树进行遍历，使用`equals()`方法查找其中相等的key对应的value。

`resize()`：数组容量翻倍。扩容后，原数组中的所有元素需要找到在新数组中的位置。

相关问题：

1. 为什么HashMap的数组长度须为2的整数次幂？

    为了能让HashMap存储高效，尽量减少碰撞，尽量让数组分配均匀。利用key的散列值对数组的长度进行取模运算可以得到数组的下标，当数组长度为2的整数次幂时，key的散列值对数组长度取余等价于key的散列值与数组长度减1的值相与。由于采用二进制位与操作，相对于%能够提高效率。

2. 为什么默认加载因子为0.75？

    加载因子表示哈希表中元素的填满程度。加载因子越大，填满的元素越多，空间利用率高，但发生冲突的几率也增加；加载因子越小，填满的元素越少，冲突发生几率减小，但空间浪费多，还会提高扩容时rehash的次数。

    在理想情况下，使用随机哈希码，在加载因子为0.75的情况下，节点出现的频率在哈希表中遵循参数平均为0.5的泊松分布。

    选择0.75作为加载因子，完全是时间和空间成本上寻求的一种折衷选择。

3. 为什么说HashMap线程不安全？

    主要原因在于并发条件下，由于链表插入节点采用头插法，扩容时会导致链表反转形成一个循环链表，后续在调用`get()`时会进入死循环。JDK8解决了此问题，改用尾插法插入节点，此时扩容转移前后链表顺序不变，保持之前节点的引用关系。不过在并发条件下使用HashMap还是会存在其他问题诸如数据丢失。

    _头插法：新值会作为链表的头部替换原来的值，原来的值会被顺推到链表当中。设计者认为后来插入的值被查找的概率较高，使用头插法可以提高查找的效率。_

4. JDK8为什么增加红黑树？选择红黑树而不是二叉查找树？

    当链表长度过长时，查找效率降低（时间复杂度`O(n)`）；红黑树可以提高数据检索的速度（时间复杂度`O(logn)`）。

    二叉树在特殊情况下会退化成链表，使用红黑树可以解决二叉树的缺陷。

5. 为什么不一直使用红黑树？

    红黑树在插入时需要通过旋转、变色等操作来保持平衡，为了维持这种平衡需要付出代价。当链表很短时不必要使用红黑树，否则会导致效率更低；当链表很长时使用红黑树，保持平衡所消耗的资源要远小于遍历链表所消耗的效率，所以设定一个阈值来判断链表转为红黑树的时机。

6. 为什么选择将链表长度为8作为将链表转换为红黑树的阈值？

    通常如果哈希算法正常，那么链表长度也不会很长，那么红黑树也不会带来明显的查询时间上的优势，反而会增加空间负担，所以通常情况下没有必要转化为红黑树，所以就选择了概率非常小，也就是长度为8的概率（泊松分布），将长度为8作为转换阈值。

7. 哈希函数的实现？

    ```java
    // JDK1.8
    static final int hash(Object key) {
        int h;
        return key == null ? 0 : (h = key.hashCode()) ^ (h >>> 16);
    }
    //JDK 1.7
    static int hash(int h) {
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
    ```

    使用扰动函数可以对一些实现比较差的`hashCode()`方法进行扰动以减少碰撞。

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

**`hashCode()`和`equals()`相关**

1. 如果两个对象相等，则`hashCode()`相等
2. 两个对象相等，`equals()`返回`true`
3. 两个对象`hashCode()`相等，两个对象不一定相等
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

### fail-fast & fail-safe

- fail-fast（快速失败）：直接在容器上遍历，在遍历过程中一旦发现容器中的数据被修改了，会立刻抛出`ConcurrentModificationException`导致遍历失败

    常见的集合如ArrayList、HashMap都使用fail-fast机制，在单线程或多线程环境下都有可能出现快速失败。

    原理：迭代器在遍历时直接访问集合中的内容，并且在遍历过程中使用一个`modCount`变量，集合在被遍历期间如果内容发生变化就会改变`modCount`的值，每当迭代器使用`hashNext()`/`next()`遍历下一个元素之前，都会检测`modCount`变量和`expectedModCount`是否相等，若相等就返回遍历，否则抛出`ConcurrentModificationException`，终止遍历。

    避免措施：使用迭代器上的修改数据的方法而不是集合内置的方法，但该方法不能指定元素，存在局限性。
- fail-safe（安全失败）：这种遍历基于容器的一个克隆，因此对容器内容的修改不影响遍历

    JUC下的容器均为安全失败，可以在多线程下并发使用、修改，常见的使用fail-safe方式遍历的容器有ConcurrentHashMap、CopyOnWriteArrayList等。

    原理：遍历时不是直接在集合内容上访问，而是先复制原有集合内容，在拷贝的集合上进行遍历，由于迭代时是对原集合的拷贝进行遍历，所以在遍历过程中对原集合所做的修改不能被迭代器检测到，所以不会触发`ConcurrentModificationException`。

    缺点：迭代器不能访问到修改后的内容，即迭代器遍历的是开始遍历那一刻拿到的集合拷贝，在遍历期间原集合发生的修改迭代器是不知道的。
## 选用集合

根据集合的特点进行选用：

- 需要根据键值对获取元素值：Map
    - 需要排序：TreeMap
    - 不需要排序：HashMap
    - 线程安全：ConcurrentHashMap
- 需要存放元素值：Collection
    - 需要保证元素唯一：Set
        - 有序：TreeSet
        - 无序：HashSet
    - 不需要保证元素唯一：List
        - 数组（查询快，插入/删除慢）：ArrayList
        - 链表（查询慢，插入/删除快）：LinkedList
