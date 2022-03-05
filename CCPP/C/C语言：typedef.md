<h1 align="center">C语言：关键字typedef</h1>

[toc]

## **typedef常见应用场景**

### **为特定含义的类型取别名**

例如，假设速度是整型值：

```text
typedef int SpeedType;
```

那么你就可以像下面这样使用了：

```c
#include<stdio.h>
typedef int SpeedType;
int main(void)
{
    SpeedType s = 10;
    printf("speed is %d m/s",s);
    return 0;
}
```

在main函数中，你可以直接使用SpeedType作为一种类型来定义变量了。有人可能问了，为什么要这样，直接使用int不是更好吗？那么如果你的代码中很多地方都用到了这个，但是突然有一天不再使用int，而是使用long呢？是不是直接修改typedef部分就可以了？（当然打印的地方也需要变，可自定义打印函数），另外一方面，通过SpeedType这个名字就可以非常直接的读懂变量的含义。

事实上，size_t，socklen_t等类型都是类似的定义。

说到typedef，就需要提一下define了，define只是一个字符串简单替换。当然下面这样的例子你可能见过很多次了：

```c
#define PIONTER int*
PIONTER a,b; //等同于int* a,b;
typedef int* POINTER1
POINTER1 c,d;//等同于int *c;int *d;
```

### **为结构体取别名**

这个也比较常见，不过有的人认为，为结构体取别名并不是一个明智的选择，因为它在使用的时候不能直观看到它是结构体类型了。

```c
struct info
{
    char name[128];
    int length;
};
```

那么你在声明变量的时候，需要带上struct，即像下面这样使用：

```text
struct info var;
```

但是如果你用typedef取个别名呢？

```c
typedef struct info
{
    char name[128];
    int length;
}Info;
```

你就可以像下面这样使用了：

```c
Info var;
```

### **声明函数指针类型**

前面的都很好理解，那么来看看函数指针：

```c
typedef void*(*Fun)(int,int);
```

这里将返回类型为void *，入参为int的函数类型命名为Fun，那么在其他地方，就可以像下面这样使用啦：

```c
//来源：公众号【编程珠玑】，博客地址：https://www.yanbinghu.com
#include <stdio.h>

typedef void*(*Fun)(int,int);

void *test(int a,int b)
{
    printf("%d,%d\n",a,b);
    //do something
    return NULL;
}

int main(void)
{
    Fun myfun = test;//这里的Fun已经是一种类型名了
    myfun(1,1);
    return 0;
}
```

是不是发现跟前面的不一样了呢？类型别名的位置飘忽不定，有的在最后，有的在中间。

关于函数指针应用方面的介绍这里不展开，有兴趣的可以参数《[高级指针话题-函数指针](https://link.zhihu.com/?target=http%3A//mp.weixin.qq.com/s%3F__biz%3DMzI2OTA3NTk3Ng%3D%3D%26mid%3D2649284302%26idx%3D1%26sn%3Dff641fe387304221983823cecf98d05c%26chksm%3Df2f9ada9c58e24bfb3b7de4bebfb3b4c87809f10a8633ad4daa09a6ed7f9450c3a3448fd3312%26scene%3D21%23wechat_redirect)》。

当然typedef的场景并不限于以上几种，这里仅仅是举例。

> 来源：公众号【编程珠玑】，博客地址：[https://www.yanbinghu.com](https://link.zhihu.com/?target=https%3A//www.yanbinghu.com)

### **一句话理解**

我不知道你是不是已经完全理解了前面的场景，无论理解与否，这句话都能很好的帮助你再次理解前面的内容：
**typedef中声明的类型在变量名的位置出现**。

什么意思呢，我们回头来看。我们是怎么声明int类型变量的？

```c
int Typename;
```

像上面这样，对不对？那么用typedef之后呢？把变量名的位置替换为别名：

```c
typedef int Typename;
```

好了，你现在已经把为int取别名为Typename。

再来看结构体，声明普通结构体变量：

```c
struct info
{
    char name[128];
    int length;
};
struct info Typename;
```

用typedef取别名，别名取代变量名的位置：

```c
struct info
{
    char name[128];
    int length;
};
typedef struct info Typename;
```

好了，你现在已经为struct info取别名为Typename。
当然这可能我们平常通常使用下面这种写法：

```c
typedef struct info
{
    char name[128];
    int length;
}Typename;
```

再来看函数指针类型，我们平常是如何声明函数的？

```c
void *function(int,int);
```

那么使用typedef取别名呢？用别名取代函数名的位置即可：

```c
void *(*Fun)(int,int);
```

不过这里需要注意用括号将这个别名括起来，并在前面加*号。





## 参考

* [一句话帮你理解typedef的用法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/81221267)
* [typedef函数指针用法_Liam Q的专栏-CSDN博客](https://blog.csdn.net/qll125596718/article/details/6891881)
* [typedef 函数类型 详解_xiaorenwuzyh的专栏-CSDN博客_typedef 模板函数](https://blog.csdn.net/xiaorenwuzyh/article/details/48997767)