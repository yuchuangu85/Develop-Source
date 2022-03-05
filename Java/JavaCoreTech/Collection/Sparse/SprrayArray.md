<h1 align="center">SparseArray</h1>

[toc]

## SparseArray 原理

SparseArray，通常来讲是 Android 中用来替代 HashMap 的一个数据结构。 准确来讲，是用来替换key为 Integer 类型，value为Object 类型的HashMap。需要注意的是 SparseArray 仅仅实现了 Cloneable 接口，所以不能用Map来声明。 从内部结构来讲，SparseArray 内部由两个数组组成，一个是 `int[]`类型的 mKeys，用来存放所有的键；另一个是 `Object[]`类型的 mValues，用来存放所有的值。 最常见的是拿 SparseArray 跟HashMap 来做对比，由于 SparseArray 内部组成是两个数组，所以占用内存比 HashMap 要小。我们都知道，增删改查等操作都首先需要找到相应的键值对，而 SparseArray 内部是通过二分查找来寻址的，效率很明显要低于 HashMap 的常数级别的时间复杂度。提到二分查找，这里还需要提一下的是二分查找的前提是数组已经是排好序的，没错，SparseArray 中就是按照key进行升序排列的。 综合起来来说，SparseArray 所占空间优于 HashMap，而效率低于 HashMap，是典型的时间换空间，适合较小容量的存储。 从源码角度来说，我认为需要注意的是 `SparseArray的remove()、put()`和 `gc()`方法。

- **`remove()`。** SparseArray 的 `remove()` 方法并不是直接删除之后再压缩数组，而是将要删除的 value 设置为 DELETE 这个 SparseArray 的静态属性，这个 DELETE 其实就是一个 Object 对象，同时会将 SparseArray 中的 mGarbage 这个属性设置为 true，这个属性是便于在合适的时候调用自身的 `gc()`方法压缩数组来避免浪费空间。这样可以提高效率，如果将来要添加的key等于删除的key，那么会将要添加的 value 覆盖 DELETE。
- **`gc()。`** SparseArray 中的 `gc()` 方法跟 JVM 的 GC 其实完全没有任何关系。``gc()` 方法的内部实际上就是一个for循环，将 value 不为 DELETE 的键值对往前移动覆盖value 为DELETE的键值对来实现数组的压缩，同时将 mGarbage 置为 false，避免内存的浪费。
- **`put()。`** put 方法是这么一个逻辑，如果通过二分查找 在 mKeys 数组中找到了 key，那么直接覆盖 value 即可。如果没有找到，会拿到与数组中与要添加的 key 最接近的 key 索引，如果这个索引对应的 value 为 DELETE，则直接把新的 value 覆盖 DELET 即可，在这里可以避免数组元素的移动，从而提高了效率。如果 value 不为 DELETE，会判断 mGarbage，如果为 true，则会调用 `gc()`方法压缩数组，之后会找到合适的索引，将索引之后的键值对后移，插入新的键值对，这个过程中可能会触发数组的扩容。