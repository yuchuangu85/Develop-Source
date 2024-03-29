<h1 align="center">为什么两个数组不能直接赋值？const详解</h1>

[toc]

## **指针 关于const**

### const

### 数组变量 是 const 的指针

在初学数组时，我们都有这样的思考：既然变量可以互相赋值，那么 `数组` 可以相互赋值吗？
比如说：

```c
int a = 1;
int b = 2;
int arr1[3] = {1, 2, 3};
int arr2[3] = {0};
b = a;//ok
arr2 = arr1;//error
```

一但这么些程序就会报错，为什么会这样呢？

> 这是因为，以上面的为例：`int arr2[3] = {0}`在编译器看来其实是这样的：`int* const arr2` 上一篇我们也学到了，const在 * 后 const 修饰的是地址 arr2，因此arr2是不能被改变的

### `int* const arr` 与 `int arr[]`是否可以划等号？

我们先来看下面这个程序：

```c
int main() {

    int arr[3] = { 1, 2, 3 };
    int* const q = arr;

    printf("arr  = %p\n", arr);
    printf("&arr = %p\n", &arr);

    printf("q    = %p\n", q);
    printf("&q   = %p\n", &q);

}
```

这个程序里，arr 的值与 q 的值相同我们应该是提前都会想到的。问题就是 这个 arr 的地址 与 q 的地址问题。他们会相同吗？虽然他们都指向 arr，但是这是两个不同的指针变量，所以他们的地址肯定是不会相同的。请看在我的机器上输出结果：

```c
arr  = 004FF824
&arr = 004FF824
q    = 004FF824
&q   = 004FF818
```

`&arr` 竟然与 `arr` 与 `q` 是一样的！ 为什么会 这样？`&arr` 与 `arr` 有什么区别？请看下面的程序：

```c
int main() {

    int arr[3] = { 1, 2, 3 };
    int* const q = arr;

    printf("arr      =   %p\n", arr);
    printf("&arr     =   %p\n", &arr);

    printf("arr + 1  =   %p\n", arr + 1);
    printf("&arr + 1 =   %p\n", &arr + 1);

    printf("%d\n", ((int)(&arr + 1) - (int)(&arr)));//将指针转变为int，看地址相差多少

    printf("q        =   %p\n", q);
    printf("&q       =   %p\n", &q);

    printf("&q + 1   =   %p\n", &q + 1);

}

arr      =   0020F860
&arr     =   0020F860
arr + 1  =   0020F864
&arr + 1 =   0020F86C
12
q        =   0020F860
&q       =   0020F854
&q + 1   =   0020F858
```

`&arr + 1` 和 `arr + 1`差了 12 个字节， 刚好是一整个arr数组的长度。这意味着什么？

> 取数组的地址 实际上 取走的是 `整个数组` 的 地址，它将整个数组视为整体，对它进行加减，大小是整个数组的大小，而`&q + 1`得值仅仅变化了 4 个字节 ，就是一个指针的大小

## 来源

https://zhuanlan.zhihu.com/p/104578649