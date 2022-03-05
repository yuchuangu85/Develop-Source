<h1 align="center">List</h1>

[TOC]

## 常见实现类

* ArrayList
* LinkedList
* CopyOnWriteArrayList



## 详解

### ArrayList：

* int newCapacity = oldCapacity + (oldCapacity >> 1);//1.5倍(15 >> 1=7) 扩容是大约1.5倍扩容，HashMap则是刚好2倍扩容。
* add(int index, E element)；将当前处于该位置的元素（如果有的话）和所有后续元素向后移动（其索引加 1）【System.arraycopy】。
* trimToSize()去掉预留元素的位置，返回一个新数组，新数组不含null，数组的size和elementData.length相等，以节省空间。此函数可避免size很小但elementData.length很大的情况。
* 不建议使用contains(Object o)方法，看源码就知道了，调用其内置的indexOf方法，for循环一个个equals，这效率只能呵呵哒了，建议使用hashcode。
* remove： 首先判断要remove的元素是null还是非null，然后for循环查找，核心是fastRemove(index)方法。 fastRemove并不返回被移除的元素。  elementData[--size] = null;因为arraycopy方法是将elementData的index+1处开始的元素往前复制，也就是说最后一个数本该消除，但还在那里，所以需要置空。
* subList方法得到的subList将和原来的list互相影响，不管你改哪一个，另一个都会随之改变，而且当父list结构改变时，子list会抛ConcurrentModificationException异常。解决方案：List<String> subListNew = new ArrayList(parentList.subList(1, 3));//【类似Arrays.asList()方法】
* ArrayList 底层是一个动态扩容的Object数组结构，默认容量是10，每次扩容需要增加到原来的1.5倍的容量
* ArrayList 扩容底层是通过Arrays.CopyOf和System.arraycopy来实现的。每次都会产生新的数组，和数组中内容的拷贝，所以会耗费性能，所以在多增删的操作的情况可优先考虑 LinkedList 而不是 ArrayList。
* ArrayList 的 toArray 方法重载方法的使用。
* 允许存放（不止一个） null 元素，
* 允许存放重复数据，存储顺序按照元素的添加顺序
* ArrayList 并不是一个线程安全的集合。如果集合的增删操作需要保证线程的安全性，可以考虑使用CopyOnWriteArrayList或者使Collections.synchronizedList(List)函数返回一个线程安全的ArrayList类.
* 调用add()方法时，add()方法首先调用ensureCapacityInternal()来判断elementData数组容量是否足够，ensureCapacityInternal()之所以能够判断，是因为它内部调用了ensureExplicitCapacity()方法，这个方法才是真正判断elementData数组容量是否够用的关键方法。如果容量足够，则直接将元素添加到ArrayList中；如果容量不够，则ensureExplicityCapacity()方法内部会调用grow()方法来对数组进行扩容。扩容成功之后，再将元素添加到ArrayList扩容之后的新数组中。**注意**：如何扩容呢？会先创建一个原来数组1.5倍大小的新数组，然后将数据拷贝到新数组中。



## 对比

### 1、Vector和ArrayList

| 名称       | 结构     | 增   | 删   | 查   | 线程是否安全 | 效率 | 是否存null   |
| ---------- | -------- | ---- | ---- | ---- | ------------ | ---- | ------------ |
| ArrayList  | 数组     | 慢   | 慢   | 快   | 不安全       | 高   | 可以（多个） |
| LinkedList | 双向链表 | 快   | 快   | 慢   | 不安全       | 高   | 不可以       |
| Vector     | 数组     | 慢   | 慢   | 快   | 安全         | 低   | 可以（多个） |

1. vector是线程同步的，所以它也是线程安全的，而arraylist是线程异步的，是不安全的。如果不考虑到线程的安全因素，一般用arraylist效率比较高。

2. Vector底层数据结构为Object数组，增删慢、查询快，线程安全，效率低，每次扩容为原来数组的2倍。

3. 如果集合中的元素的数目大于目前集合数组的长度时，vector增长率为目前数组长度的100%,而arraylist增长率为目前数组长度的50%.如过在集合中使用数据量比较大的数据，用vector有一定的优势。

4. 如果查找一个指定位置的数据，vector和arraylist使用的时间是相同的，都是0(1),这个时候使用vector和arraylist都可以。而如果移动一个指定位置的数据花费的时间为0(n-i)n为总长度，这个时候就应该考虑到使用linklist,因为它移动一个指定位置的数据所花费的时间为0(1),而查询一个指定位置的数据时花费的时间为0(i)。

5. 是否保证线程安全： ArrayList 和 LinkedList 都是不同步的，也就是不保证线程安全；

6. 底层数据结构： Arraylist 底层使用的是Object数组；LinkedList 底层使用的是双向循环链表数据结构；

7. 插入和删除是否受元素位置的影响：
   ① ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响。 比如：执行add(E e)方法的时候， ArrayList 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是$O(1)$。但是如果要在指定位置 i 插入和删除元素的话（add(int index, E element)）时间复杂度就为 $O(n-i)$。因为在进行上述操作的时候集合中第 i 和第 i 个元素之后的$(n-i)$个元素都要执行向后位/向前移一位的操作。

    ② LinkedList 采用链表存储，所以插入，删除元素时间复杂度不受元素位置的影响，都是近似 $O(1)$而数组为近似 $O(n)$。

8. 是否支持快速随机访问： LinkedList 不支持高效的随机元素访问，而ArrayList 实现了RandmoAccess 接口，所以有随机访问功能。快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index)方法)。

9. 内存空间占用： ArrayList的空间浪费主要体现在在list列表的结尾会预留一定的容量空间，而LinkedList的空间花费则体现在它的每一个元素都需要消耗比ArrayList更多的空间（因为要存放直接后继和直接前驱以及数据）。

10. Vector类的所有方法都是同步的。可以由两个线程安全地访问一个Vector对象、但是一个线程访问Vector的话代码要在同步操作上耗费大量的时间。

11. Vector底层数据结构为数组，增删慢、查询快，线程安全，效率低，每次扩容为原来数组的2倍。

12. ArrayList区别于数组的地方在于能够自动扩展大小，每次扩容1.5倍。

13. 当对数据的主要操作为索引或只在集合的末端增加、删除数据时，使用ArrayList效率比较高；当对数据的操作主要为指定位置的插入或删除操作时，使用LinkedList效率比较高。

14. 为什么ArrayList在增、删的时候效率低？**答**：因为在增、删的过程中会涉及到数组的复制，效率低

15. ArrayList的时间复杂度是多少？**答**：当修改、查询或者只在数组末尾增、删时，时间复杂度为O(1)；对指定位置的元素进行增、删时，时间复杂度为O(n)。

ArrayList 和Vector是采用数组方式存储数据，此数组元素数大于实际存储的数据以便增加和插入元素，都允许直接序号索引元素，但是插入数据要设计到数组元素移动等内存操作，所以索引数据快插入数据慢，Vector由于使用了synchronized方法（线程安全）所以性能上比ArrayList要差，LinkedList使用双向链表实现存储，按序号索引数据需要进行向前或向后遍历，但是插入数据时只需要记录本项的前后项即可，所以插入数度较快！
    

### 2、AarrayList和LinkedList

1. ArrayList是实现了基于动态数组的数据结构，LinkedList基于链表的数据结构。
2. 对于随机访问get和set，ArrayList觉得优于LinkedList，因为LinkedList要移动指针。
3. 对于新增和删除操作add和remove，LinedList比较占优势，因为ArrayList要移动数据。

这一点要看实际情况的。若只对单条数据插入或删除，ArrayList的速度反而优于LinkedList。但若是批量随机的插入删除数据，LinkedList的速度大大优于ArrayList. 因为ArrayList每插入一条数据，要移动插入点及之后的所有数据。