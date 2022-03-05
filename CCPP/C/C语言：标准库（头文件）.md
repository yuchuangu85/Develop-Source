<h1 align="center">C语言这些常用的标准库（头文件）</h1>

[toc]

有很多工程师喜欢自己封装一些标准库已有的函数，其实自己封装的函数，并不一定比标准库好，有时候反而代码更冗余，且有bug。

下面小编就来分享一下C语言常见的一些标准库。

**标准头文件包括：**

> <asset.h><ctype.h><errno.h><float.h><limits.h>
> <locale.h><math.h><stdio.h><signal.h><time.h>
> <stddef.h><stdlib.h><string.h><stdarg.h><setjmp.h>

### **一、标准定义（<stddef.h>）**

文件<stddef.h>里包含了标准库的一些常用定义，无论我们包含哪个标准头文件，<stddef.h>都会被自动包含进来。

这个文件里定义：

**● 类型size_t**（sizeof运算符的结果类型，是某个无符号整型）；

**● 类型ptrdiff_t**（两个指针相减运算的结果类型，是某个有符号整型）；

**● 类型wchar_t**（宽字符类型，是一个整型，其中足以存放本系统所支持的所有本地环境中的字符集的所有编码值。这里还保证空字符的编码值为0）；

**● 符号常量NULL**（空指针值）；

**● 宏offsetot** （这是一个带参数的宏，第一个参数应是一个结构类型，第二个参数应是结构成员名。

> offsetot(s,m)

求出成员m在结构类型t的变量里的偏移量）。

> **注：**其中有些定义也出现在其他头文件里（如NULL）。



### **二、错误信息（<errno.h>）**

<errno.h>定义了一个int类型的表达式errno，可以看作一个变量，其初始值为0，一些标准库函数执行中出错时将它设为非0值，但任何标准库函数都设置它为0。

<errno.h>里还定义了两个宏EDOM和ERANGE，都是非0的整数值。数学函数执行中遇到参数错误，就会将errno置为EDOM，如出现值域错误就会将errno置为ERANGE。

### **三、输入输出函数（<stdio.h>）**

文件打开和关闭：

![img](https://pic3.zhimg.com/80/v2-4c0a3db4924c02bcda5e5a2f7424af5e_1440w.png)

字符输入输出：

![img](https://pic4.zhimg.com/80/v2-0f958ce0564f6f7b543cc63e03050213_1440w.png)

**getc**和**putc**与这两个函数类似，但通过宏定义实现。通常有下面定义：

![img](https://pic4.zhimg.com/80/v2-125d764f8f53aeb4ec3b9fef7501fc7b_1440w.png)

格式化输入输出：

![img](https://pic4.zhimg.com/80/v2-3ec75dade982c1803996507f737f4c23_1440w.jpg)

行式输入输出：

![img](https://pic1.zhimg.com/80/v2-e832d01959f1a5ea8972bb5ebb077684_1440w.png)

直接输入输出：

![img](https://pic2.zhimg.com/80/v2-deaaf7b7b67f37f699de5edce42e0371_1440w.png)



### **四、数学函数（<math.h>）**

**1.三角函数：**

![img](https://pic4.zhimg.com/80/v2-7ddd7018dace658eaa18f522acbc80b3_1440w.jpg)

**2.指数和对数函数：**

![img](https://pic3.zhimg.com/80/v2-18b69e0a84359669bdf3d7e65b16ea56_1440w.jpg)

**3.其他函数：**

![img](https://pic2.zhimg.com/80/v2-aaf02072c1332ee8ec76a06103d7d95d_1440w.jpg)

> **注：**所有上面未给出类型特征的函数都取一个参数，其参数与返回值都是double类型。

下面函数返回双精度值（包括函数ceil和floor）。在下表里，除其中有特别说明的参数之外，所有函数的其他参数都是double类型。

**函数原型意义解释：**

![img](https://pic2.zhimg.com/80/v2-797b9285ae67e71dafd14cebf05ebf39_1440w.jpg)



### **五、字符处理函数（<ctype.h>）**

见下表：

![img](https://pic3.zhimg.com/80/v2-603ffc89dbe433d5f28c21043606ad12_1440w.jpg)

> **注：**条件成立时这些函数返回非0值。最后两个转换函数对于非字母参数返回原字符。

### **六、字符串函数（<string.h>）**

**1.字符串函数**

所有字符串函数列在下表里，函数描述采用如下约定：s、t表示 (char *)类型的参数，cs、ct表示(const char*)类型的参数（它们都应表示字符串）。

n表示size_t类型的参数（size_t是一个无符号的整数类型），c是整型参数（在函数里转换到char）：

**函数原型意义解释：**

![img](https://pic1.zhimg.com/80/v2-aad794a3f5ae576e218ff28378c0d7bc_1440w.jpg)

**2.存储区操作**

<string.h>还有一组字符数组操作函数（存储区操作函数），名字都以mem开头，以某种高效方式实现。

在下面原型中，参数s和t的类型是(void *)，cs和ct的类型是(const void *)，n的类型是size_t，c的类型是int（转换为unsigned char）。

函数原型意义解释：

![img](https://pic2.zhimg.com/80/v2-9a0882ebb6d07fd28d3506de8c6288e9_1440w.jpg)



### **七、功能函数（<stdlib.h>）**

**1.随机数函数：**

函数原型意义解释

![img](https://pic2.zhimg.com/80/v2-407cbb4f11c5d640787aaa455ff824e5_1440w.jpg)

**2.动态存储分配函数：**

函数原型意义解释

![img](https://pic1.zhimg.com/80/v2-985c9414547c532bdb1293da83b630f8_1440w.jpg)

**3.几个整数函数**

几个简单的整数函数见下表，div_t和ldiv_t是两个预定义结构类型，用于存放整除时得到的商和余数。

div_t类型的成分是int类型的quot和rem，ldiv_t类型的成分是long类型的quot和rem。

函数原型意义解释

![img](https://pic2.zhimg.com/80/v2-94fa07b760c8138686a2f90f98028a09_1440w.jpg)

**4.数值转换**

函数原型意义解释

![img](https://pic4.zhimg.com/80/v2-c6e94fe9466e4acc2535c4f289521227_1440w.jpg)

**5.执行控制**

**1）**非正常终止函数abort。

原型是：

![img](https://pic3.zhimg.com/80/v2-b71dbfea86afd23bdee955b66ecd585a_1440w.png)

**2）**正常终止函数exit。

原型是：

![img](https://pic4.zhimg.com/80/v2-cac7786378211bfa017b180a12b610df_1440w.png)

导致程序按正常方式立即终止。status作为送给执行环境的出口值，0表示成功结束，两个可用的常数为EXIT_SUCCESS，EXIT_FAILURE。

**3）**正常终止注册函数atexit。

原型是：

![img](https://pic2.zhimg.com/80/v2-cf6b2c4c24555f2ed9dbae14a73012f9_1440w.png)

可用本函数把一些函数注册为结束动作。被注册函数应当是无参无返回值的函数。注册正常完成时atexit返回值0，否则返回非零值。

**6.与执行环境交互**

**1）**向执行环境传送命令的函数system。

原型是：

![img](https://pic1.zhimg.com/80/v2-68bf718b5d2386b92cdf9a02c92b31d0_1440w.png)

把串s传递给程序的执行环境要求作为系统命令执行。如以NULL为参数调用，函数返回非0表示环境里有命令解释器。如果s不是NULL，返回值由实现确定。

**2）**访问执行环境的函数getenv。

原型是：

![img](https://pic2.zhimg.com/80/v2-54685cec02b41a3d448712f31e70ac31_1440w.png)

从执行环境中取回与字符串s相关联的环境串。如果找不到就返回NULL。本函数的具体结果由实现确定。在许多执行环境里，可以用这个函数去查看“环境变量”的值。

**7.常用函数bsearch和qsort**

**1）**二分法查找函数bsearch：

![img](https://pic3.zhimg.com/80/v2-0f6df47084fa0d5d5c0eb4d3dd681836_1440w.png)

函数指针参数cmp的实参应是一个与字符串比较函数strcmp类似的函数，确定排序的顺序，当第一个参数keyval比第二个参数datum大、相等或小时分别返回正、零或负值。

**2）**快速排序函数qsort：

qsort对于比较函数cmp的要求与bsearch一样。设有数组base[0],...,base[n-1]，元素大小为size。用qsort可以把这个数组的元素按cmp确定的上升顺序重新排列。

![img](https://pic4.zhimg.com/80/v2-5e2935fbc1c93bbfa9173f77ecf4107b_1440w.png)

最后，不管你是转行也好，初学也罢，进阶也可，如果你想学编程~

[C语言这些常用的标准库（头文件），你不得不知道... - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/339231872)