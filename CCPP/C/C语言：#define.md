<h1 align="center">C语言：#define</h1>

[toc]

## #define概念

* 定义：定义一个标识符来表示一个常量。其特点是：定义的标识符不占内存，只是一个临时的符号，预编译后这个符号就不存在了。
* 形式：`#define 标识符 常量`  //注意, 最后没有分号
* \#define又称宏定义，标识符为所定义的宏名，简称宏。标识符的命名规则与前面讲的变量的命名规则是一样的。#define 的功能是将标识符定义为其后的常量。一经定义，程序中就可以直接用标识符来表示这个常量。

1. 程序中什么时候会使用宏定义呢？用宏定义有什么好处呢？我们直接写数字不行吗？为什么要用一个标识符表示数字呢？

   宏定义最大的好处是“方便程序的修改”。使用宏定义可以用宏代替一个在程序中经常使用的常量。注意，是“经常”使用的。这样，当需要改变这个常量的值时，就不需要对整个程序一个一个进行修改，只需修改宏定义中的常量即可。且当常量比较长时，使用宏就可以用较短的有意义的标识符来代替它，这样编程的时候就会更方便，不容易出错。因此，宏定义的优点就是方便和易于维护。

2. 那么程序在预编译的时候是怎么处理宏定义的呢？或者说是怎么处理预处理指令的呢？

   其实预编译所执行的操作就是简单的“文本”替换。对宏定义而言，预编译的时候会将程序中所有出现“标识符”的地方全部用这个“常量”替换，称为“宏替换”或“宏展开”。替换完了之后再进行正式的编译。所以说当单击“编译”的时候实际上是执行了两个操作，即先预编译，然后才正式编译。#include<stdio.h>也是这样的，即在预处理的时候先单纯地用头文件stdio.h中所有的“文本”内容替换程序中#include<stdio.h>这一行，然后再进行正式编译。

   **需要注意的是，预处理指令不是语句，所以后面不能加分号。这是很多新手经常犯的错误。#include 后面也没有加分号。**

宏定义 `#define `一般都写在函数外面，与 #include 写在一起。当然，写在函数里面也没有语法错误，但通常不那么写。#define 的作用域为自` #define` 那一行起到源程序结束。如果要终止其作用域可以使用 #undef 命令，格式为：

```c
#undef 标识符
```

undef 后面的标识符表示你所要终止的宏。比如前面在程序开头用 define 定义了一个宏 M，它原本的作用范围是一直到程序结束，但如果现在在程序中某个位置加了一句：

```
#undef  M
```

那么这个宏的作用范围到此就结束了。#undef 用得不多，但大家要了解。

注意：\#define 和 #include 一样，也是以“#”开头的。凡是以“#”开头的均为预处理指令，#define也不例外。

例子：

```c
# include <stdio.h>
# define PI 3.14159
int main(void)
{
    double r, s;
    printf("请输入圆的半径:");
    scanf("%lf", &r);  //scanf中, double只能用%lf
    s = PI * r * r;
    printf("s=PI*r^2 = %.6f\n", s);  //PI不会被宏替换
    return 0;
}
```

输出结果是：

请输入圆的半径:1

s=PI*r^2 = 3.141590

PI 是定义的宏，它表示的是其后的常量（而不是变量）。此外，程序中用双引号括起来的宏在预处理的时候是不会被宏替换的。因为在 C 语言中，用双引号括起来表示的是字符串。

## #define用法

在#define中，标准只定义了 `#` 和 `##` 两种操作。`#` 用来把参数转换成字符串，`##` 则用来连接前后两个参数，把它们变成一个字符串。

```c
#include<stdio.h>
#define paster(n) printf("token"#n"=%d\n",token##n)

int main(int argc,char *argv[])
{
    int token9=10;
    paster(9);
    return 0;
}
输出为:token 9 = 10
```

无参宏定义的一般形式为：**#define 标识符 字符串**

　　其中的“#”表示这是一条预处理命令。凡是以“#”开头的均为预处理命令。“define”为宏定义命令。“标识符”为所定义的宏名。“字符串”可以是常数、表达式、格式串等。

　　**例如：**　**#define M (a+b)**　它的作用是指定标识符M来代替表达式(a+b)。在编写源程序时，所有的(a+b)都可由M代替，而对源程序作编译时，将先由预处理程序进行宏代换，即用(a+b)表达式去置换所有的宏名M，然后再进行编译。

```c
#include<stdio.h>
#defineM(a+b)

int main(intargc,char*argv[])
{
    ints,a,b;
    printf("inputnumbera&b:");
    scanf("%d%d",&a,&b);
    s=M*M;
    printf("s=%d\n",s);
}
```

带参的宏定义一般形式为： 　**#define 宏名(形参**表) 字符串

```c
#include<stdio.h>
#define MAX(a,b) ((a>b)?(a):(b))

int main(intargc,char*argv[])
{
    intx,y,max;
    printf("inputtwonumbers:");
    scanf("%d%d",&x,&y);
    max=MAX(x,y);
    printf("max=%d\n",max);
    return0;
}
```

## #undef及其用法

简  介：在后面取消以前定义的宏定义

在此程序中，我们将取消在先前程序中对预处理器的定义。

```c
#include <stdio.h>
int main( void )
{
　　#define MAX 200
    　　printf("MAX= %d\n",MAX);
　　#undef MAX
　　#define MAX 300
    　　printf("MAX= %d\n",MAX);
    return 0;
}
```



## 参考

* [C语言宏的定义和宏的使用方法（#define）](http://c.biancheng.net/view/446.html)
* [#define及其用法](https://www.cnblogs.com/wangqw/p/4230915.html)
* [#undef及其用法](https://www.cnblogs.com/wangqw/p/4230918.html)

