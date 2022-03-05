<h1 align="center">C语言：关键字extern、static</h1>

[toc]

## 关键字extern

> 如果要在外部变量的定义之前使用该变量，或者外部变量的定义与变量的使用不在同一个源文件，则必须在相应的变量声明中强制性使用关键字extern
>
> 外部变量的定义中必须指定数组的长度，但是extern声明则不一定要指定数组的长度

### 1.引用同一个文件中的变量

```c
#include<stdio.h>
int func();

int main()
{
  func(); //1
  printf("%d", num);//2
  return 0;
}

int num = 3;
 
int func() 
{
  printf("%d\n", num);
}
```

如果按照这个顺序，变量 num在main函数的后边进行声明和初始化的话，那么在main函数中是不能直接引用num这个变量的，因为当编译器编译到这一句话的时候，找不到num这个变量的声明，但是在func函数中是可以正常使用，因为func对num的调用是发生在num的声明和初始化之后。

如果我不想改变num的声明的位置，但是想在main函数中直接使用num这个变量，怎么办呢？可以使用extern这个关键字。像下面这一段代码，利用extern关键字先声明一下num变量，告诉编译器num这个变量是存在的，但是不是在这之前声明的，你到别的地方找找吧，果然，这样就可以顺利通过编译啦。但是你要是想欺骗编译器也是不行的，比如你声明了extern int num；但是在后面却没有真正的给出num变量的声明，那么编译器去别的地方找了，但是没找到还是不行的。

下面的程序就是利用extern关键字，使用在后边定义的变量。

```c
#include<stdio.h>

int func();

int main()
{
  func(); //1
  extern int num;
  printf("%d", num);//2
  return 0;
}

int num = 3;
 
int func() 
{
  printf("%d\n", num);
}
```

### 引用另一个文件中的变量

如果extern这个关键字就这点功能，那么这个关键字就显得多余了，因为上边的程序可以通过将num变量在main函数的上边声明，使得在main函数中也可以使用。
extern这个关键字的真正的作用是引用不在同一个文件中的变量或者函数。

main.c

```c
#include<stdio.h>

int main()
{
  extern int num;
  printf("%d", num);
  return 0;
}
```

b.c

```c
#include<stdio.h>

int num = 5;

void func()
{
  printf("fun in a.c");
}
```

例如，这里b.c中定义了一个变量num，如果main.c中想要引用这个变量，那么可以使用extern这个关键字，注意这里能成功引用的原因是，num这个关键字在b.c中是一个全局变量，也就是说只有当一个变量是一个全局变量时，extern变量才会起作用，像下面这样是`不行`的。

main.c

```c
#include<stdio.h>

int main()
{
  extern int num;
  printf("%d", num);
  return 0;
}
```

b.c

```c
#include<stdio.h>

void func()
{
  int num = 5;
  printf("fun in a.c");
}
```

另外，extern关键字只需要指明类型和变量名就行了，不能再重新赋值，初始化需要在原文件所在处进行，如果不进行初始化的话，全局变量会被编译器自动初始化为0。像这种写法是`不行`的。

```c
extern int num = 4;
```

但是在声明之后就可以使用变量名进行修改了，像这样：

```c
#include<stdio.h>

int main()
{
  extern int num;
  num = 1;
  printf("%d", num);
  return 0;
}
```

如果不想这个变量被修改可以使用const关键字进行修饰，写法如下：
mian.c

```c
#include<stdio.h>

int main()
{
  extern const int num;
  printf("%d", num);
  return 0;
}
```

b.c

```c
#include<stdio.h>

const int num = 5;

void func()
{
  printf("fun in a.c");
}
```

使用include将另一个文件全部包含进去可以引用另一个文件中的变量，但是这样做的结果就是，被包含的文件中的所有的变量和方法都可以被这个文件使用，这样就变得不安全，如果只是希望一个文件使用另一个文件中的某个变量还是使用extern关键字更好。

### 引用另一个文件中的函数

extern除了引用另一个文件中的变量外，还可以引用另一个文件中的函数，引用方法和引用变量相似。

mian.c

```c
#include<stdio.h>

int main()
{
  extern void func();
  func();
  return 0;
}
```

b.c

```c
#include<stdio.h>

const int num = 5;

void func()
{
  printf("fun in a.c");
}
```

这里main函数中引用了b.c中的函数func。因为所有的函数都是全局的，所以对函数的extern用法和对全局变量的修饰基本相同，需要注意的就是，需要指明返回值的类型和参数。



## 关键字static

C语言 static 关键字的常见用法有三种：

- 用于局部变量的修饰符；
- 用于全局变量的修饰符；
- 用于函数的修饰符。

### 1、用于局部变量的修饰符

当 static 用于修饰局部变量时，通常是在某个函数体内，只能在该函数内被调用。

这样定义的变量通常被称为**局部静态变量**，它的值不会因为函数调用的结束而被清除，当函数再次被调用时，它的值是上一次调用结束后的值。

如下面这段代码[所示，变量 x 是局部变量，变量 y 是静态局部变量。在调用函数后，变量 x 的值会被清除，而变量 y 的值则](https://link.zhihu.com/?target=https%3A//jq.qq.com/%3F_wv%3D1027%26k%3D5PrT9Dq)会被保留。多次调用该函数，变量 x 每次都会从新初始化，而变量 y 的值则不会。

```c
#include <stdio.h>
#include <string.h>

void my_function()
{
  int x = 0;
  static int y = 0;
  
  printf("x: %d, y: %d\n", x, y);
  
  x = x + 5;
  y = y + 5;
}

int main()
{
  my_function();
  my_function();
  my_function();
  return 0;
}
```

输出结果：

```
x: 0, y: 0
x: 0, y: 5
x: 0, y: 10
```

静态局部变量的特性：

- 存储位置：处于静态存储区，当用 static 修饰局部变量的时候，它就改变了局部变量的存储位置，从原来的栈中存放改为静态存储区；
- 初始化操作：未经初始化的局部静态变量会被程序自动初始化为0（自动对象的值是任意的，除非他被显示初始化）；
- 作用域：为局部作用域，即当定义它的函数结束的时候，作用域随之结束（不能被访问）。但是静态局部变量在离开作用域之后，并没有被销毁，而是仍然保存在内存当中，直到程序结束。

### 2、用于全局变量的修饰符

关键字 static 还可用于修饰全局变量，该变量在某一个文件中变量，但不属于任何一个函数内，这样的变量通常称为**静态全局变量**。

静态全局变量的存储位置、初始化操作同静态局部变量的特性，但其作用域有所不同：静态全局变量可以被该文件内的所有函数访问，但不能被其它文件内的函数访问。

### 3、用于函数的修饰符

关键字 static 还可以用于修饰一个函数，这样的函数称之为静态函数。

定义一个静态函数就是在函数的返回类型前加上 static 关键字。

静态函数的作用域仅限于本文件，不能被其它文件调用。

```c
#include <stdio.h>
#include <string.h>

static void my_function()
{
  int x = 0;
  static int y = 0;
  
  printf("x: %d, y: %d\n", x, y);
  
  x = x + 5;
  y = y + 5;
}

int main()
{
  my_function();
  my_function();
  my_function();
  return 0;
}
```

### static

* 隐藏与隔离的作用
  * 如果我们希望全局变量仅限于在本源文件中使用，在其他源文件中不能引用，也就是说限制其作用域只在定义该变量的源文件内有效，而在同一源程序的其他源文件中不能使用。这时，就可以通过在全局变量之前加上关键字 static 来实现，使全局变量被定义成为一个静态全局变量。
  * 所有未加 static 前缀的全局变量和函数都具有全局可见性，其它的源文件也能访问。
* 保持变量内容的持久性
  * 存储在静态数据区的变量会在程序刚开始运行时就完成初始化，也是唯一的一次初始化。共有两种变量存储在静态存储区：全局变量和 static 变量，只不过和全局变量比起来，static 可以控制变量的可见范围，说到底 static 还是用来隐藏的。
* 默认初始化为 0
  * 在静态数据区，内存中所有的字节默认值都是 0x00。静态变量与全局变量也一样，它们都存储在静态数据区中，因此其变量的值默认也为 0。



## 参考

* [C语言正确使用extern关键字_xingjiarong的专栏-CSDN博客_c extern用法](https://blog.csdn.net/xingjiarong/article/details/47656339)
* [C语言关键字 static 的用法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/112027143)
* [static](https://www.runoob.com/w3cnote/c-static-effect.html)
* [static](http://c.biancheng.net/view/301.html)

