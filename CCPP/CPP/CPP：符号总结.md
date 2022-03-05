<h1 align="center">C++ 符号总结</h1>

[toc]

## 1.符号 " :: " 

#### 1.作用域符号：前面一般是类名，后面一般是成员名称

例如：A,B表示两个类，在A,B中都有成员变量member。那么

```c++
A::member就表示A中的成员member
B::member就表示B中的成员member
```

#### 2.全局作用域符号：当全局变量在局部函数中与其中某个变量重名，就可以用::来区分，如下：

```c++
char zhou;  // 全局变量
void sleep()
{
    char zhou;  //  局部变量
    char(局部变量) = char(局部变量) *char(局部变量);
    ::char(全局变量) =::char(全局变量) *char(局部变量)
}
```

#### 3.::是C++里的“作用域分解运算符”。比如声明一个类，类A里声明了一个成员函数voidf()，但是没有在类的声明里给出 f 的定义，那么在类外定义f时，就要写成voidA:f()，表示这个f()函数是类A的成员函数。例如：

```c++
class CA {
public:
    int ca_var;
    int add(int a, int b);
    int add(int a);
}

// 那么在实现函数时，必须这样写：
int CA::add(int a, int b)
{
    return a + b;
}

// 另外，双冒号也常常用于在类变量内部作为当前类实例的元素进行表示，比如：
int CA::add(int a)
{
    return a + ::ca_var;
}

// 表示当前类实例中的变量ca_var

```
