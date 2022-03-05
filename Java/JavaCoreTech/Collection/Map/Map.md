<h1 align="center">Map</h1>

[TOC]

## 常用实现类

* HashMap
* ConcurrentHashMap
* LinkedHashMap
* HashTable
* TreeMap

## 详解

### HashMap：

* 当某个桶中的键值对数量大于8个【9个起】，且桶数量大于等于64，则将底层实现从链表转为红黑树 ；
* int threshold; // 新的扩容resize临界值,当实际大小(容量*填充比)大于临界值时，会进行2倍扩容；
* key是有可能是null的，并且会在0桶位位置；
* tableSizeFor(int cap) {//计算下次需要调整大小的扩容resize临界值；结果为>=cap的最小2的自然数幂（64-》64；65-》128）
* length为2的整数幂保证了length - 1 最后一位（二进制表示）为1，从而保证了索引位置index即（ hash &length-1）的最后一位同时有为0和为1的可能性，保证了散列的均匀性。length为2的幂保证了按位与最后一位的有效性，使哈希表散列更均匀。
* resize 时【链表】的变化： 元素位置在【原位置】或【原位置+oldCap】，链表转红黑树后，【仅在扩容resize时】若树变短，会恢复为链表。
* 当链表长度太长（默认超过8）时，链表就转换为红黑树。

### ConcurrentHashMap：

* CAS算法；unsafe.compareAndSwapInt(this, valueOffset, expect, update);  CAS(Compare And Swap)，意思是如果valueOffset位置包含的值与expect值相同，则更新valueOffset位置的值为update，并返回true，否则不更新，返回false。
* 与Java8的HashMap有相通之处，底层依然由“哈希数组”+链表+红黑树；底层结构存放的是TreeBin对象，而不是TreeNode对象；
* CAS作为知名无锁算法，那ConcurrentHashMap就没用锁了么？当然不是，hash值相同的链表的头结点还是会synchronized上锁。 

```java
private transient volatile int sizeCtl;
```

​		`sizeCtl` 是控制标识符，不同的值表示不同的意义。

* 负数代表正在进行初始化或扩容操作
* -1代表正在初始化
* -N 表示有N-1个线程正在进行扩容操作
* 正数或0代表hash表还没有被初始化，这个数值表示初始化或下一次进行扩容的大小，类似于扩容阈值。它的值始终是当前ConcurrentHashMap容量的0.75倍，这与loadfactor是对应的。实际容量>=sizeCtl，则扩容。

 1、concurrencyLevel：
    能够同时更新ConccurentHashMap且不产生锁竞争的最大线程数，在Java8之前实际上就是ConcurrentHashMap中的分段锁个数，即Segment[]的数组长度。正确地估计很重要，当低估，数据结构将根据额外的竞争，从而导致线程试图写入当前锁定的段时阻塞；相反，如果高估了并发级别，你遇到过大的膨胀，由于段的不必要的数量; 这种膨胀可能会导致性能下降，由于高数缓存未命中。
    在Java8里，仅仅是为了兼容旧版本而保留。唯一的作用就是保证构造map时初始容量不小于concurrencyLevel。
ForwardingNode：
    `并不是我们传统的包含key-value的节点，只是一个标志节点，并且指向nextTable，提供find方法而已。生命周期：仅存活于扩容操作且bin不为null时，一定会出现在每个bin的首位。`

3个原子操作（调用频率很高）

```java
    static final <K,V> Node<K,V> tabAt(Node<K,V>[] tab, int i) { // 获取索引i处Node
        return (Node<K,V>)U.getObjectVolatile(tab, ((long)i << ASHIFT) + ABASE);
    }
    
    // 利用CAS算法设置i位置上的Node节点（将c和table[i]比较，相同则插入v）。
    static final <K,V> boolean casTabAt(Node<K,V>[] tab, int i,
                                        Node<K,V> c, Node<K,V> v) {
        return U.compareAndSwapObject(tab, ((long)i << ASHIFT) + ABASE, c, v);
    }
    
    // 设置节点位置的值，仅在上锁区被调用
    static final <K,V> void setTabAt(Node<K,V>[] tab, int i, Node<K,V> v) {
        U.putObjectVolatile(tab, ((long)i << ASHIFT) + ABASE, v);
    }
```

ConcurrentHashMap无锁多线程扩容，减少扩容时的时间消耗。
transfer扩容操作：单线程构建两倍容量的nextTable；允许多线程复制原table元素到nextTable。

1. 为每个内核均分任务，并保证其不小于16；
2. 若nextTab为null，则初始化其为原table的2倍；
3. 死循环遍历，直到finishing。

* 节点为空，则插入ForwardingNode；
* 链表节点（fh>=0），分别插入nextTable的i和i+n的位置；【逆序链表？？】
* TreeBin节点（fh<0），判断是否需要untreefi，分别插入nextTable的i和i+n的位置；【逆序树？？】
* finishing时，nextTab赋给table，更新sizeCtl为新容量的0.75倍 ，完成扩容。

以上说的都是单线程，多线程又是如何实现的呢？
    遍历到ForwardingNode节点((fh = f.hash) == MOVED)，说明此节点被处理过了，直接跳过。这是控制并发扩容的核心 。由于给节点上了锁，只允许当前线程完成此节点的操作，处理完毕后，将对应值设为ForwardingNode（fwd），其他线程看到forward，直接向后遍历。如此便完成了多线程的复制工作，也解决了线程安全问题。

2、 put相关：

理一下put的流程：
	① 判空：null直接抛空指针异常；
	② hash：计算h=key.hashcode；调用spread计算hash=(h ^(h >>>16))& HASH_BITS；
	③ 遍历table

    * 若table为空，则初始化，仅设置相关参数；
    * @@@计算当前key存放位置，即table的下标i=(n - 1) & hash；
    * 若待存放位置为null，casTabAt无锁插入；
    * 若是forwarding nodes（检测到正在扩容），则helpTransfer（帮助其扩容）；
    * else（待插入位置非空且不是forward节点，即碰撞了），将头节点上锁（保证了线程安全）：区分链表节点和树节点，分别插入（遇到hash值与key值都与新节点一致的情况，只需要更新value值即可。否则依次向后遍历，直到链表尾插入这个结点）；
    * 若链表长度>8，则treeifyBin转树（Note：若length<64,直接tryPresize,两倍table.length;不转树）。

④ addCount(1L, binCount)。
Note：
	1、put操作共计两次hash操作，再利用“与&”操作计算Node的存放位置。
	2、ConcurrentHashMap不允许key或value为null。
	3、addCount(longx, intcheck)方法：
   	 ① 利用CAS快速更新baseCount的值；
   	 ② check>=0.则检验是否需要扩容；if sizeCtl<0（正在进行初始化或扩容操作）【nexttable null等情况break；如果有线程正在扩容，则协助扩容】；else if 仅当前线程在扩容，调用协助扩容函数，注其参数

### LinkedHashMap：

* remove后再put，集合结构变化：只要未冲突，table不改变（想想put原理就好理解了）；但链表改变，新元素始终在tail。
* 显式地指定为access order后【前提】，调用get()方法，导致对应的entry移动到双向链表的最后位置（tail），但是table未没变。
* So LinkedHashMap元素有序存放，但并不保证其迭代顺序一直不变
* LinkedHashMap的每个bucket都存了这个bucket的before和after，且每个before(after)又存储了自身的前驱后继，直到null。

​		迭代：

```java
Iterator<Map.Entry> iterl = map.entrySet().iterator();
利用ArrayList的【ListIterator】向前迭代：
ListIterator<Map.Entry> iterpre = new ArrayList<Map.Entry>(map.entrySet()).listIterator(map.size());
while (iterpre.hasPrevious()) {……}
```



## 对比

### HashMap与TreeMap

1. HashMap通过hashcode对其内容进行快速查找，而TreeMap中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用TreeMap（HashMap中元素的排列顺序是不固定的）。HashMap中元素的排列顺序是不固定的）。
2. HashMap通过hashcode对其内容进行快速查找，而TreeMap中所有的元素都保持着某种固定的顺序，如果你需要得到一个有序的结果你就应该使用TreeMap（HashMap中元素的排列顺序是不固定的）。集合框架提供两种常规的Map实现：HashMap和TreeMap (TreeMap实现SortedMap接口)。
3. 在Map 中插入、删除和定位元素，HashMap 是最好的选择。但如果您要按自然顺序或自定义顺序遍历键，那么TreeMap会更好。使用HashMap要求添加的键类明确定义了hashCode()和 equals()的实现。 这个TreeMap没有调优选项，因为该树总处于平衡状态。

### Hashtable与HashMap

1. 历史原因:Hashtable是基于陈旧的Dictionary类的，HashMap是Java 1.2引进的Map接口的一个实现 。
2. 同步性:Hashtable是线程安全的，也就是说是同步的，而HashMap是线程序不安全的，不是同步的 。
3. 值：只有HashMap可以让你将空值作为一个表的条目的key或value 。

### ConurrentHashMap和CopyOnWriteArrayList的区别

为什么我们需要ConcurrentHashMap和CopyOnWriteArrayList

同步的集合类（Hashtable和Vector），同步的封装类（使用Collections.synchronizedMap()方法和Collections.synchronizedList()方法返回的对象）可以创建出线程安全的Map和List。但是有些因素使得它们不适合高并发的系统。它们仅有单个锁，对整个集合加锁，以及为了防止ConcurrentModificationException异常经常要在迭代的时候要将集合锁定一段时间，这些特性对可扩展性来说都是障碍。

ConcurrentHashMap和CopyOnWriteArrayList保留了线程安全的同时，也提供了更高的并发性。ConcurrentHashMap和CopyOnWriteArrayList并不是处处都需要用，大部分时候你只需要用到HashMap和ArrayList，它们用于应对一些普通的情况。

### ConcurrentHashMap和Hashtable的区别

Hashtable和ConcurrentHashMap有什么分别呢？它们都可以用于多线程的环境，但是当Hashtable的大小增加到一定的时候，性能会急剧下降，因为迭代时需要被锁定很长的时间。因为ConcurrentHashMap引入了分割(segmentation)，不论它变得多么大，仅仅需要锁定map的某个部分，而其它的线程不需要等到迭代完成才能访问map。简而言之，在迭代的过程中，ConcurrentHashMap仅仅锁定map的某个部分，而Hashtable则会锁定整个map。

### HashMap和Hashtable的区别

1. HashMap几乎可以等价于Hashtable，除了HashMap是非synchronized的，并可以接受null(HashMap可以接受为null的键值(key)和值(value)，而Hashtable则不行)。
2. HashMap是非synchronized，而Hashtable是synchronized，这意味着Hashtable是线程安全的，多个线程可以共享一个Hashtable；而如果没有正确的同步的话，多个线程是不能共享HashMap的。Java 5提供了ConcurrentHashMap，它是HashTable的替代，比HashTable的扩展性更好。
3. 另一个区别是HashMap的迭代器(Iterator)是fail-fast迭代器，而Hashtable的enumerator迭代器不是fail-fast的。所以当有其它线程改变了HashMap的结构（增加或者移除元素），将会抛出ConcurrentModificationException，但迭代器本身的remove()方法移除元素则不会抛出ConcurrentModificationException异常。但这并不是一个一定发生的行为，要看JVM。这条同样也是Enumeration和Iterator的区别。
4. 由于Hashtable是线程安全的也是synchronized，所以在单线程环境下它比HashMap要慢。如果你不需要同步，只需要单一线程，那么使用HashMap性能要好过Hashtable。
5. HashMap不能保证随着时间的推移Map中的元素次序是不变的。