<h1 align="center">C语言：结构体</h1>

[toc]

## 1.结构体的定义方式

C语言结构体（Struct）从本质上讲是一种自定义的数据类型，只不过这种数据类型比较复杂，是由 int、char、float 等基本类型组成的。你可以认为结构体是一种聚合类型。

在C语言中，可以使用**结构体（Struct）**来存放一组不同类型的数据。结构体的定义形式为：

```c
struct 结构体名{
  结构体所包含的变量或数组
};
```

结构体是一种集合，它里面包含了多个变量或数组，它们的类型可以相同，也可以不同，每个这样的变量或数组都称为结构体的成员（Member）。请看下面的一个例子：

```c
struct stu{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在学习小组
    float score;  //成绩
};
```

stu 为结构体名，它包含了 5 个成员，分别是 name、num、age、group、score。结构体成员的定义方式与变量和数组的定义方式相同，只是不能初始化。

> 注意大括号后面的分号`;`不能少，这是一条完整的语句。

结构体也是一种数据类型，它由程序员自己定义，可以包含多个其他类型的数据。

像 int、float、char 等是由C语言本身提供的数据类型，不能再进行分拆，我们称之为基本数据类型；而结构体可以包含多个基本类型的数据，也可以包含其他的结构体，我们将它称为复杂数据类型或构造数据类型。

### 1) 先定义结构体类型，再定义结构体变量。

```c
struct student{
    char no[20];       //学号
    char name[20];    //姓名
     char sex[5];    //性别
    int age;          //年龄
};             
struct student stu1,stu2;
```

* 此时stu1,stu2为student结构体变量

### 2) 定义结构体类型的同时定义结构体变量。

```c
struct student{
    char no[20];        //学号
    char name[20];     //姓名
    char sex[5];      //性别
    int age;            //年龄
} stu1,stu2;
```

* 此时还可以继续定义student结构体变量，如：struct student stu3;

### 3) 不指定类型名而直接定义结构体变量

```c
struct{
    char no[20];        //学号
    char name[20];      //姓名
    char sex[5];      //性别
    int age;          //年龄
} stu1,stu2;
```

* 一般不使用这种方法，因为直接定义结构体变量stu1、stu2之后，就不能再继续定义该类型的变量。

### 4) 用typedef定义结构体变量

```c
typedef struct student
{
       char name[20];
       int age;
}student_t;
```

* 上面的代码，定义了一个结构体变量类型，这个类型有2个名字：第一个名字是struct student；第二个类型名字是student_t.
* 定义了这个之后，下面有2中方法可以定义结构体变量
* 第一种： struct student student_1;   
* 第二种：student_t student_1 
* 推荐在实际代码中使用第四种方法定义结构体变量。

## 2.用结构体实现父类，子类模拟HAL层的代码

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>

void say() {
    printf("good mornin\n");
}

void work() {
    printf("I am working\n");
}

void cook() {
    printf("I am cooking\n");
}

typedef struct person {
    void (*say)();
} person;

typedef struct man {
    person p;

    void (*work)();
} man;

typedef struct woman {
    person p;

    void (*cook)();
} woman;

void get_person(int i, struct person **person) {
    if (i == 0) {
        man *m;
        m = (man *) malloc(sizeof(*m));
        memset(m, 0, sizeof(m));
        m->p.say = say;
        m->work = work;
        *person = &m->p;
    } else {
        woman *w;
        w = (woman *) malloc(sizeof(*w));
        memset(w, 0, sizeof(w));
        w->p.say = say;
        w->cook = cook;
        *person = &w->p;
    }
}

void get_man(struct man **m) {
    get_person(0, (struct person **) m);
}

void get_woman(struct woman **w) {
    get_person(1, (struct person **) w);
}

int main(int argc __unused, char **argv __unused) {
    man *m;
    get_man(&m);
    ((person *) m)->say();//向上转型成父类
    m->work();

    woman *w;
    get_woman(&w);
    ((person *) m)->say();
    w->cook();
}
```

## 3.结构体（struct）变量

### 结构体变量

既然结构体是一种数据类型，那么就可以用它来定义变量。例如：

```c
struct stu stu1, stu2;
```

定义了两个变量 stu1 和 stu2，它们都是 stu 类型，都由 5 个成员组成。注意关键字`struct`不能少。

stu 就像一个“模板”，定义出来的变量都具有相同的性质。也可以将结构体比作“图纸”，将结构体变量比作“零件”，根据同一张图纸生产出来的零件的特性都是一样的。

你也可以在定义结构体的同时定义结构体变量：

```c
struct stu{
    char *name;  // 姓名
    int num;  // 学号
    int age;  // 年龄
    char group;  // 所在学习小组
    float score;  // 成绩
} stu1, stu2;// 变量放在后面
```

将变量放在结构体定义的最后即可。

如果只需要 stu1、stu2 两个变量，后面不需要再使用结构体名定义其他变量，那么在定义时也可以不给出结构体名，如下所示：

```c
struct{  // 没有写 stu
    char *name;  // 姓名
    int num;  // 学号
    int age;  // 年龄
    char group;  // 所在学习小组
    float score;  // 成绩
} stu1, stu2;
```

这样做书写简单，但是因为没有结构体名，后面就没法用该结构体定义新的变量。

理论上讲结构体的各个成员在内存中是**连续存储**的，和数组非常类似，例如上面的结构体变量 stu1、stu2 的内存分布如下图所示，共占用 4+4+4+1+4 = 17 个字节。

![img](./media/150GQ243-0.jpg)


但是在编译器的具体实现中，各个成员之间可能会存在缝隙，对于 stu1、stu2，成员变量 group 和 score 之间就存在 3 个字节的空白填充（见下图）。这样算来，stu1、stu2 其实占用了 17 + 3 = 20 个字节。

![img](./media/150GUE0-1.jpg)

关于成员变量之间存在“裂缝”的原因，我们将在《[C语言内存精讲](http://c.biancheng.net/c/140/)》专题中的《[C语言内存对齐，提高寻址效率](http://c.biancheng.net/view/vip_2093.html)》一节中详细讲解。

### 成员的获取和赋值

结构体和数组类似，也是一组数据的集合，整体使用没有太大的意义。数组使用下标`[ ]`获取单个元素，结构体使用点号`.`获取单个成员。获取结构体成员的一般格式为：

```c
结构体变量名.成员名;
```

通过这种方式可以获取成员的值，也可以给成员赋值：

```c
#include <stdio.h>
int main(){
    struct{
        char *name;  //姓名
        int num;  //学号
        int age;  //年龄
        char group;  //所在小组
        float score;  //成绩
    } stu1;
    //给结构体成员赋值
    stu1.name = "Tom";
    stu1.num = 12;
    stu1.age = 18;
    stu1.group = 'A';
    stu1.score = 136.5;
    //读取结构体成员的值
    printf("%s的学号是%d，年龄是%d，在%c组，今年的成绩是%.1f！\n", stu1.name, stu1.num, stu1.age, stu1.group, stu1.score);
    return 0;
}
```

运行结果：
Tom的学号是12，年龄是18，在A组，今年的成绩是136.5！

除了可以对成员进行逐一赋值，也可以在定义时整体赋值，例如：

```c
struct{
    char *name;  //姓名
    int num;  //学号
    int age;  //年龄
    char group;  //所在小组
    float score;  //成绩
} stu1, stu2 = { "Tom", 12, 18, 'A', 136.5 };// 直接赋值
```

不过整体赋值仅限于定义结构体变量的时候，在使用过程中只能对成员逐一赋值，这和数组的赋值非常类似。

## 参考

作者：王小二的Android站
链接：https://www.jianshu.com/p/65e80b946649
来源：简书
著作权归作者所有。商业转载请联系作者获得授权，非商业转载请注明出处。