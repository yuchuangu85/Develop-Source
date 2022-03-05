<h1 align="center">HashMap 的 loadFactor 为什么是 0.75</h1>

[TOC]

## 1.答案一

之前看各大面经的时候搜索到了这个问题，切实感觉到如果刨根问底的问，自己还真不能抵挡住这种攻势，现在闲暇时间又心血来潮地想起来这个问题，就打算好好弄懂弄透，也希望能在将来面试的时候做好准备。

本文基于[这个 StackOverflow 回答](https://stackoverflow.com/a/31401836)进一步推导，并给出详细解答步骤



###  1. loadFactor 是什么

loadFactor 翻译为 **负载因子**，是 HashMap 负载程度的一个度量，所谓负载程度即 HashMap 持有的元素数量和 HashMap 大小的比值

当 HashMap 中的元素数量大于 `capacity * loadFactor` 时，HashMap 就要扩容，并进行重新 hash

那么，我们可以得出一个重要结论，`loadFactor` 是为了让 HashMap 尽可能 **不满** 而存在的

众所周知，HashMap 越空越好，这样插入和查找都能尽可能接近常数级别

那么接下来的一个重要问题就是：HashMap 什么时候是空的？通过这个问题，我们就可以一步一步推导出 `loadFactor` 的值

###  2. HashMap 什么时候不是很满

我们调整 loadFactor 的根本目标在于，要让元素的插入时间缩短到最少，也就是说，**元素最好不要发生碰撞**

**只要元素在插入时不发生碰撞，那么我们的 HashMap 就不算特别的满**

这是一个很重要的结论，通过它，我们成功地把 HashMap 满不满的问题，转换到了插入元素是否碰撞的问题

###  3. 插入元素的碰撞几率

插入元素是否碰撞，这是一个概率事件，有可能碰撞，也可能不碰撞

对于一个未知的元素，它有可能插入到 HashMap 的任何一个位置，因此，对于未知的元素插入，碰撞是**等可能的**，而一个元素是否碰撞和它之后的元素是否碰撞并无关系，因此是 **独立的**

> 为什么是独立的？因为 HashMap 采用拉链法解决碰撞，碰撞的元素不占用数组空间，因此一个元素是否碰撞和它前一个元素是否碰撞没有关系

在这里，我们要引入一个假设，上面我们提到的 HashMap 不是很满，但是 loadFactor 也不应该让一个 HashMap 过于空，太空的 HashMap 会造成空间的浪费；
 假如一个元素的插入正好导致它碰撞，那么说明这个 HashMap 肯定不是特别空旷，而且当元素插入就碰撞时，恰好说明我们需要扩大 HashMap，而不是修改元素的 `hash()` 方法

因此，我们有单个元素插入碰撞的概率为

​						$$p = \frac{1}{s} $$

###  4. HashMap 在插入过程中不发生碰撞的概率

得到单个元素插入发生碰撞的概率之后，我们来考虑整个 HashMap 在插入过程中不发生碰撞的概率

对于一个 $s$ 大小的 Hashmap，我们插入 $n$ 个元素，这个操作属于等可能独立事件的**重复操作**，满足 **二项分布**，因此我们可以得出，在这个重复插入操作中，没有碰撞的概率为：

​						$$P(0)=C^{0}_{n}\times (\frac{1}{s})^0 \times (1-\frac{1}{s})^{n-0}$$

​							$$=(1-\frac{1}{s})^n$$

###  5. 什么叫 “HashMap 很可能不满”

假如一个 HashMap，它在插入元素的过程中，如果它一次碰撞都没有发生，说明它没有满；

上面我们得到了这个事件的概率，如果这个事件发生的概率很大，那么就说明 HashMap **很可能不满**

所以，若 $$P(0)≤0.5$$，则说明 HashMap 很可能没有满

则有

​							$(1 - \frac{1}{s})^n \ge \frac{1}{2}$

其中 $n$ 代表 HashMap 中元素的个数，$s$ 代表 HashMap 的数组大小

###  6. loadFactor 的计算过程

HashMap 中 `loadFactor` 即为 n/s，首先求出 n，对于上面的式子取对数，则有

​				$n\ln(1-\frac{1}{s})≥-\ln{2}$

​						$n≤{\frac{-ln2}{\ln(1-\frac{1}{s})}}$

​						$n≤{\frac{-ln2}{\ln(\frac{s-1}{s})}}$

​						$n≤{\frac{-ln2}{\ln(\frac{s}{s-1})}}$

所以，当 $n≤\frac{ln2}{ln(\frac{2s}{2s-1})}$ 时，HashMap **很可能**不满，所以

​					$\frac{n}{s}≤\frac{\ln2}{s\ln\frac{s}{s-1}}$

当 $s→∞$ 时，有

​					$$loadFactor=\lim\limits_{s\to\infty}{\frac{\ln2}{s\ln{\frac{s}{s-1}}}}$$

对于分母的式子有

​					$\lim\limits_{s\to\infty}{s\ln{\frac{s}{s-1}}}$

从形式上来看，当 $s\to\infty$ 时，$\frac{2s}{2s-1}\to0$ ，则上式为 $\infty \cdot 0$ 型，应转换为 $\frac{0}{\infty}$ 或者 $\frac{\infty}{0}$ 计算，对 $s$ 取倒数，有：
 令 $s=\frac{1}{t}$：

​					$\lim\limits_{t\to0}{\frac{1}{t}}{\ln{\frac{\frac{1}{t}}{\frac{1}{t}-1}}}$

​						$=\lim\limits_{t\to0}{\frac{1}{t}}\ln{\frac{\frac{1}{t}}{\frac{1-t}{t}}}$

​						$=\lim\limits_{t\to0}{\frac{1}{t}}\ln{\frac{1}{1-t}}$

​						$=\lim\limits_{t\to0}{\frac{\ln{\frac{1}{1-t}}}{t}}⋯(⋆)$

遇见 $x\to0$ ，就要想 **等价无穷小**，对于 $ln$⁡ 可以构造 $ln⁡(1+x) \sim x$，则有：

​					$\ln{\frac{1}{1-t}}$

​					$=\ln{\frac{1-t+t}{1-t}}$

​					$=\ln({1+\frac{t}{1-t}})$

则 $(⋆)$ 式则有

​					$\lim\limits_{t\to0}{\frac{\frac{t}{1-t}}{t}}$

​					$=\lim\limits_{t\to0}{\frac{1}{1-t}}$

​					$=1$



​		$∴loadFactor=\lim\limits_{s\to\infty}{\frac{\ln2}{s\ln{\frac{s}{s-1}}}}$

​					$=\lim\limits_{s\to\infty}{\frac{\ln2}{1}}$

​					$=\ln2$

​					$\sim0.693$



###  7. 为什么是 0.75

从上面的计算来看，`loadFactor` 取 $ln⁡2$ 时，能够让 HashMap 尽可能不满

但是在实际中，HashMap 碰撞与否，其实是与 `hashCode()` 的设计有很大关系，因此 JDK 设计者在平衡空间利用和性能方面给了一个更高的经验数字。

###  8. 总结

当然，这只是一家之言，你也可以从其他方面解释 0.75 这个值如何如何；

其实这种刨根问底的问题，终究希望考察你的 **能力** 而不是 **记忆**，只要你能给出自己的解释，而不是被问住，呆若木鸡，就能通过面试。

## 2.StackOverflow上的答案

HashMap实例有两个影响其性能的参数:初始容量和负载系数。容量是哈希表中的桶数，初始容量只是创建哈希表时的容量。负载系数是一种衡量在自动增加容量之前哈希表允许的满度的度量。当哈希表中的条目数超过负载因子和当前容量的乘积时，哈希表被重新哈希(也就是说，重新构建内部数据结构)，以便哈希表的桶数大约是桶数的两倍。

作为一般规则，默认负载系数(0.75)在时间和空间成本之间提供了很好的权衡。较高的值会减少空间开销，但会增加查找成本(反映在HashMap类的大多数操作中，包括get和put)。在设置map的初始容量时，应该考虑map的预期条目数和它的负载因子，以减少重新哈希操作的数量。如果初始容量大于条目的最大数量除以负载系数，则不会发生重新哈希操作。

## 3.Why is the loading factor of HashMap 0.75?

There are a lot of things that I didn't pay much attention to before , I'm also reviewing HashMap I found that there are many problems that can be studied in detail , It will eventually return to Mathematics , Such as HashMap Why is the loading factor of 0.75？

This paper mainly introduces the following contents ：

- Why? HashMap Need to load factor ？
- What's the solution to the conflict ？
- Why the loading factor must be 0.75？ instead of 0.8,0.6？

### Why? HashMap Need to load factor ？

HashMap At the bottom of is the hash table , Is the type of structure that stores key value pairs , It needs some calculation to determine the storage location of data in the hash table ：

```
static final int hash(Object key) {
int h;
return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
// AbstractMap
public int hashCode() {
int h = 0;
Iterator<Entry<K,V>> i = entrySet().iterator();
while (i.hasNext())
h += i.next().hashCode();
return h;
}
```

General data structure , Either query fast or insert fast ,HashMap It's a slow insertion 、 Query fast data structure .

But this data structure is prone to two problems ：① If space utilization is high , When the hash algorithm is used to calculate the storage location , You'll find that many storage locations already have data （ Hash Collisions ）;② If you want to avoid hash conflicts , Increase the size of the array , It will lead to low space utilization .

And the loading factor means Hash The degree to which the elements in the table are filled .

Load factor = The number of elements in the table / The length of the hash table

The larger the load factor , The more elements you fill , The higher the space utilization , But the chances of conflict are bigger ;

The smaller the load factor , The fewer elements you fill , The chance of conflict diminishes , But more space is wasted , And it's going to increase capacity rehash Number of operations .

The greater the chance of conflict , Explain that the data to be searched needs another way to find , The higher the cost of searching . therefore , Must be in “ Opportunities for conflict ” And “ Space utilization ” Between , Looking for a balance and compromise .

So we can also know , These are the main factors that affect the efficiency of search ：

- Whether the hash function can hash the data in the hash table evenly ？
- How to deal with conflict ？
- How to select the loading factor of hash table ？

This paper mainly introduces the latter two questions .

### What's the solution to the conflict ？

#### 1. Open addressing

Hi = (H(key) + di) MOD m, among i=1,2,…,k(k<=m-1) H(key) Is the hash function ,m For hash table length ,di For incremental sequence ,i For the number of conflicts that have occurred . among , Open addressing method can be divided into 3 Kind of ：

##### 1.1 Linear exploration （Linear Probing）：di = 1,2,3,…,m-1

In short , Starting from the current conflict position , In steps of 1 Loop search , Until you find an empty place , If you don't have a place after the cycle , That means the container is full . Take a chestnut , It's like you're eating in the street at lunch point , Go from house to house to see if there is a same place .

##### 1.2 Square detection （Quadratic Probing）：di = ±12, ±22,±32,…,±k2（k≤m/2）

As opposed to linear probing , This is equivalent to a step size of di = i2 To loop through , Until we find an empty place . Take the example above , Now you're not going from house to house to see if there's a place , It's a cell phone i2 stores , Then go and ask if the store has a location .

##### 1.3 Pseudo random detection ：di = A sequence of pseudo-random numbers

This is to take a random number as the step size . Use the example above , This time, I went to choose a shop according to my mood and asked if there was a place .

But open addressing has these disadvantages ：

The hash table created by this method , When there are many conflicts, the data is easy to pile together , It's not friendly to search at this time ; When deleting a node, you can't simply empty the space of the node , Otherwise, it will truncate the synonym node search path after it fills in the hash table . So if you want to delete a node , Only delete mark can be added on the deleted node , Instead of deleting the nodes ; If the hash table is full , You also need to create an overflow table , To store the extra elements .

#### 2. Then the hash method

Hi = RHi(key), among i=1,2,…,k RHi() Functions are different from H() Hash function of , Used when a synonym has an address conflict , Calculate the address of another hash function , Until there are no conflicting positions . This method is not easy to generate heaps , But it will increase the calculation time .

So the drawback of hashifa is ： Increased computing time .

#### 3. Create a common overflow area

Suppose that the value range of the hash function is [0, m-1], A vector HashTable[0,…,m-1] For the basic table , Each component holds a record , In addition, the vector is set up OverTable[0,…,v] For overflow table . The base table stores a record of keywords , In case of conflict , No matter what the hash address they get from the hash function , Fill in the overflow table .

But the disadvantage of this method is that ： When looking for conflicting data , You need to traverse the overflow table to get the data .

#### 4. Chain address （ Zipper method ）

Construct the elements in the conflict position into a linked list . When adding data , If the hash address conflicts with an element on the hash table , It's on the list at this location .

The advantages of the zipper method ：

The way to deal with conflict is simple , And there is no heap phenomenon , Non synonyms never conflict , So the average search length is shorter ; Because the node space of each chain list in zipper method is applied dynamically , So it's more suitable for situations where the table length cannot be determined before making a table ; Deleting nodes is easy to implement , Simply delete the corresponding node on the linked list . Disadvantages of zipper method ： Additional storage required .

from HashMap We can see that ,HashMap It's an array + Linked list / A combination of red and black trees as the underlying structure , Open address method + Chain address method to achieve HashMap.![ Insert picture description here ](media/20210309145127601H_0.png)

### Why? HashMap The loading factor must be 0.75？ instead of 0.8,0.6？

We know from the above ,HashMap The underlying layer of is actually a hash table （ Hash table ）, The way to solve the conflict is chain address method .HashMap The default initial capacity size of is 16, In order to reduce the probability of conflict , When HashMap When the array length reaches a critical value , It will trigger the expansion , Put all the elements rehash And then put it in the expanded container , It's a fairly time-consuming operation .

The critical value is determined by the load factor and the current container size ：

critical value = DEFAULT_INITIAL_CAPACITY * DEFAULT_LOAD_FACTOR

By default, it is 16x0.75=12 when , It will trigger the expansion operation .

So why did you choose 0.75 As HashMap What about the loading factor ？ This is a very important principle in statistics —— Poisson distribution is related to .

Poisson distribution is a common discrete probability distribution in statistics and probability , It can be used to describe the probability distribution of the number of random events in unit time . Interested readers can take a look at Wikipedia or this article by Ruan Yifeng ： Poisson distribution and exponential distribution ：10 Minute tutorial [1]

![ Insert picture description here ](media/20210309145127601H_1.png)

To the left of the equal sign ,P Representation probability ,N Denotes a functional relationship ,t Time ,n It means quantity . To the right of the equal sign ,λ Indicates the frequency of the event .

stay HashMap There is such a comment in the source code of ：

```
* Ideally, under random hashCodes, the frequency of
* nodes in bins follows a Poisson distribution
* (http://en.wikipedia.org/wiki/Poisson_distribution) with a
* parameter of about 0.5 on average for the default resizing
* threshold of 0.75, although with a large variance because of
* resizing granularity. Ignoring variance, the expected
* occurrences of list size k are (exp(-0.5) * pow(0.5, k) /
* factorial(k)). The first values are:
* 0: 0.60653066
* 1: 0.30326533
* 2: 0.07581633
* 3: 0.01263606
* 4: 0.00157952
* 5: 0.00015795
* 6: 0.00001316
* 7: 0.00000094
* 8: 0.00000006
* more: less than 1 in ten million
```

In an ideal situation , Use random hash code , At the expansion threshold （ Load factor ） by 0.75 Under the circumstances , Nodes appear in the frequency of Hash bucket （ surface ） The average of the following parameters is 0.5 The Poisson distribution of . Ignore variance , namely X = λt,P(λt = k), among λt = 0.5 The situation of , According to the formula ：

![ Insert picture description here ](media/20210309145127601H_2.png)

The calculation results are shown in the table above , When one bin The length of the linked list in the 8 When it's an element , The probability of 0.00000006, It's almost an impossible event .

So we can know , In fact, the constant 0.5 It is calculated by substituting the Poisson distribution as a parameter , And the loading factor 0.75 As a condition , When HashMap The length is length/size ≥ 0.75 And then expand it , Under this condition , The length and probability of the zipper after the conflict is ：

```
0: 0.60653066
1: 0.30326533
2: 0.07581633
3: 0.01263606
4: 0.00157952
5: 0.00015795
6: 0.00001316
7: 0.00000094
8: 0.00000006
```

### So why can't it be 0.8 perhaps 0.6 Well ？

HashMap In addition to the hash algorithm , There are two parameters that affect performance ： Initial capacity and loading factor . The initial capacity is the capacity of the hash table when it is created , The load factor is a measure of how full a hash table can reach before its capacity is automatically expanded .

Describe the loading factor in Wikipedia ：

For open addressing , The loading factor is particularly important , It should be strictly limited to 0.7-0.8 following . exceed 0.8, Look up the table CPU Cache miss （cache missing） Follow the exponential curve . therefore , Some use open addressing hash library , Such as Java The loading factor is limited to 0.75, Exceeding this value will resize Hash table .

When setting the initial capacity, you should consider the number of entries required in the mapping and its load factor , In order to minimize the expansion rehash Operating frequency , therefore , Generally in use HashMap It is recommended to set the initial capacity according to the estimated value , In order to reduce the expansion operation .

choice 0.75 As the default load factor , It's a trade-off between time and space costs .





## 参考

* https://wafer.li/Interview/hashmap-%E7%9A%84-loadfactor-%E4%B8%BA%E4%BB%80%E4%B9%88%E6%98%AF-0-75/
* https://stackoverflow.com/questions/10901752/what-is-the-significance-of-load-factor-in-hashmap/31401836#31401836
* https://javamana.com/2021/03/20210325004843480y.html
* [HashMap默认加载因子为什么选择0.75？(阿里)](https://www.cnblogs.com/aspirant/p/11470928.html)
* [泊松分布和指数分布：10分钟教程](http://www.ruanyifeng.com/blog/2015/06/poisson-distribution.html#comment-356111)
* 

