<h1 align="center">C语言标准库函数</h1>

[toc]

标准头文件包括：

*<asset.h> <ctype.h> <errno.h> <float.h>*

*<limits.h> <locale.h> <math.h> <setjmp.h>*

*<signal.h> <stdarg.h> <stddef.h> <stdlib.h>*

*<stdio.h> <string.h> <time.h>*

## ***一、标准定义（<stddef.h>）***

文件<stddef.h>里包含了标准库的一些常用定义，无论我们包含哪个标准头文件，<stddef.h>都会被自动包含进来。

这个文件里定义：

l 类型size_t （sizeof运算符的结果类型，是某个无符号整型）；

l 类型ptrdiff_t（两个指针相减运算的结果类型，是某个有符号整型）；

l 类型wchar_t （宽字符类型，是一个整型，其中足以存放本系统所支持的所有本地环境中的字符集的所有编码值。这里还保证空字符的编码值为0）；

l 符号常量NULL （空指针值）；

l 宏offsetor （这是一个带参数的宏，第一个参数应是一个结构类型，第二个参数应是结构成员名。

offsetor(s,m)求出成员m在结构类型t的变量里的偏移量）。

注：其中有些定义也出现在其他头文件里（如NULL）。

## **二、错误信息（\*<errno.h>\*）**

<errno.h>定义了一个*int*类型的表达式*errno*，可以看作一个变量，其初始值为*0*，一些标准库函数执行中出错时将它设为非*0*值，但任何标准库函数都设置它为*0*。

*<errno.h>*里还定义了两个宏*EDOM*和*ERANGE*，都是非*0*的整数值。数学函数执行中遇到参数错误，就会将*errno*置为*EDOM*，如出现值域错误就会将*errno*置为*ERANGE*。

## **三、输入输出函数（\*<stdio.h>\*）**

**文件打开和关闭：**

*FILE \*fopen(const char \*filename, const char \*mode);*

*int fclose(FILE \* stream);*

**字符输入输出：**

*int fgetc(FILE \*fp);*

*int fputc(int c, FILE \*fp);*

getc和putc与这两个函数类似，但通过宏定义实现。通常有下面定义：

*#define getchar() getc(stdin)*

*#define putchar(c) putc(c, stdout)*

*int ungetc(int c, FILE\* stream);//把字符 c 退回流 stream*

**格式化输入输出：**

*int scanf(const char \*format, ...);*

*int printf(const char \*format, ...);*

*int fscanf(FILE \*stream, const char \*format, ...);*

*int fprintf(FILE \*stream, const char \*format, ...);*

*int sscanf(char \*s, const char \*format, ...);*

*int sprintf(char \*s, const char \*format, ...);*

**行式输入输出：**

*char \*fgets(char \*buffer, int n, FILE \*stream);*

*int fputs(const char \*buffer, FILE \*stream);*

*char \*gets(char \*s);*

*int puts(const char \*s);*

**直接输入输出：**

*size_t fread(void \*pointer, size_t size, size_t num, FILE \*stream);*

*size_t fwrite(const void \*pointer, size_t size, size_t num, FILE \*stream);*

## **四、数学函数（\*<math.h>\*）**

三角函数：

| 三角函数   | sin  | cos  | tan  |
| ---------- | ---- | ---- | ---- |
| 反三角函数 | asin | acos | atan |
| 双曲函数   | sinh | cosh | tanh |



指数和对数函数：

| 以e为底的指数函数  | exp   |
| ------------------ | ----- |
| 自然对数函数       | log   |
| 以10为底的对数函数 | log10 |



其他函数：

| 平方根                                 | sqrt                        |
| -------------------------------------- | --------------------------- |
| 绝对值                                 | fabs                        |
| 乘幂，第一个参数作为底，第二个是指数   | double pow(double, double)  |
| 实数的余数，两个参数分别是被除数和除数 | double fmod(double, double) |



注：所有上面未给出类型特征的函数都取一个参数，其参数与返回值都是double类型。

下面函数返回双精度值（包括函数ceil和floor）。在下表里，除其中有特别说明的参数之外，所有函数的其他参数都是double类型。

| 函数原型           | 意义解释                                                     |
| ------------------ | ------------------------------------------------------------ |
| ceil(x)            | 求出不小于x的最小整数（返回与这个整数对应的double值）        |
| floor(x)           | 求出不大于x的最大整数（返回与这个整数对应的double值）        |
| atan2(y, x)        | 求出 tan-1(y/x)，其值的范围是[-pai,pai]                      |
| ldexp(x, int n)    | 求出x*2n                                                     |
| frexp(x, int *exp) | 把 x分解为 y*2n， 是位于区间 [1/2,1)里的一个小数，作为函数结果返回，整数n 通过指针*exp返回（应提供一个int变量地址）。当x 为0时这两个结果的值都是0 |
| modf(x, double*ip) | 把x分解为小数部分和整数部分，小数部分作为函数返回值，整数部分通过指针*ip返回。 |



## **五、字符处理函数（\*<ctype.h>\*）**

见下表：

| int isalpha(c)     | c是字母字符                                    |
| ------------------ | ---------------------------------------------- |
| int isdigit(c)     | c是数字字符                                    |
| int isalnum(c)     | c是字母或数字字符                              |
| int isspace(c)     | c是空格、制表符、换行符                        |
| int isupper(c)     | c是大写字母                                    |
| int islower(c)     | c是小写字母                                    |
| int iscntrl(c)     | c是控制字符                                    |
| int isprint(c)     | c是可打印字符，包括空格                        |
| int isgraph(c)     | c是可打印字符，不包括空格                      |
| int isxdigit(c)    | c是十六进制数字字符                            |
| int ispunct(c)     | c是标点符号                                    |
| int tolower(int c) | 当c是大写字母时返回对应小写字母，否则返回c本身 |
| int toupper(int c) | 当c是小写字母时返回对应大写字母，否则返回c本身 |

注：条件成立时这些函数返回非*0*值。最后两个转换函数对于非字母参数返回原字符。



## **六、字符串函数（\*<string.h>\*）**

### **字符串函数**

所有字符串函数列在下表里，函数描述采用如下约定：s、t表示 (char *)类型的参数，cs、ct表示(const char*)类型的参数（它们都应表示字符串）。n表示size_t类型的参数（size_t是一个无符号的整数类型），c是整型参数（在函数里转换到char）：

| 函数原型              | 意义解释                                                     |
| --------------------- | ------------------------------------------------------------ |
| size_t strlen(cs)     | 求出cs的长度                                                 |
| char *strcpy(s,ct)    | 把ct复制到s。要求s指定足够大的字符数组                       |
| char *strncpy(s,ct,n) | 把ct里的至多n个字符复制到s。要求s指定一个足够大的字符数组。如果ct里的字符不够n个，就在s里填充空字符。 |
| char *strcat(s,ct)    | 把ct里的字符复制到s里已有的字符串之后。s应指定一个保存着字符串，而且足够大的字符数组。 |
| char *strncat(s,ct,n) | 把ct里的至多n个字符复制到s里已有的字符串之后。s应指定一个保存着字符串，而且足够大的字符数组。 |
| int strcmp(cs,ct)     | 比较字符串cs和ct的大小，在cs大于、等于、小于ct时分别返回正值、0、负值。 |
| int strncmp(cs,ct,n)  | 比较字符串cs和ct的大小，至多比较n个字符。在cs大于、等于、小于ct时分别返回正值、0、负值。 |
| char *strchr(cs,c)    | 在cs中查寻c并返回c第一个出现的位置，用指向这个位置的指针表示。当cs里没有c时返回值NULL |
| char *strrchr(cs,c)   | 在cs中查寻c并返回c最后一个出现的位置，没有时返回NULL         |
| size_t strspn(cs,ct)  | 由cs起确定一段全由ct里的字符组成的序列，返回其长度           |
| size_t strcspn(cs,ct) | 由cs起确定一段全由非ct里的字符组成的序列，返回其长度         |
| char *strpbrk(cs,ct)  | 在cs里查寻ct里的字符，返回第一个满足条件的字符出现的位置，没有时返回NULL |
| char *strstr(cs,ct)   | 在cs中查寻串ct（查询子串），返回ct作为cs的子串的第一个出现的位置，ct未出现在cs里时返回NULL |
| char *strerror(n)     | 返回与错误编号n相关的错误信息串（指向该错误信息串的指针）    |
| char *strtok(s,ct)    | 在s中查寻由ct中的字符作为分隔符而形成的单词                  |



### **存储区操作**

<string.h>还有一组字符数组操作函数（存储区操作函数），名字都以mem开头，以某种高效方式实现。在下面原型中，参数s和t的类型是(void *)，cs和ct的类型是(const void *)，n的类型是size_t，c的类型是int（转换为unsigned char）。

| 函数原型              | 意义解释                                                     |
| --------------------- | ------------------------------------------------------------ |
| void *memcpy(s,ct,n)  | 从ct处复制n个字符到s处，返回s                                |
| void *memmove(s,ct,n) | 从ct处复制n个字符到s处，返回s，这里的两个段允许重叠          |
| int memcmp(cs,ct,n)   | 比较由cs和ct开始的n个字符，返回值定义同strcmp                |
| void *memchr(cs,c,n)  | 在n个字符的范围内查寻c在cs中的第一次出现，如果找到，返回该位置的指针值，否则返回NULL |
| void *memset(s,c,n)   | 将s的前n个字符设置为c，返回s                                 |



## **七、功能函数（<stdlib.h>）**

随机数函数：

| 函数原型                  | 意义解释                           |
| ------------------------- | ---------------------------------- |
| int rand(void)            | 生成一个0到RAND_MAX的随机整数      |
| void srand(unsigned seed) | 用seed为随后的随机数生成设置种子值 |



动态存储分配函数：

| 函数原型                            | 意义解释                                                     |
| ----------------------------------- | ------------------------------------------------------------ |
| void *calloc(size_t n, size_t size) | 分配一块存储，其中足以存放n个大小为size的对象，并将所有字节用0字符填充。返回该存储块的地址。不能满足时返回NULL |
| void *malloc(size_t size)           | 分配一块足以存放大小为size的存储，返回该存储块的地址，不能满足时返回NULL |
| void *realloc(void *p, size_t size) | 将p所指存储块调整为大小size，返回新块的地址。如能满足要求，新块的内容与原块一致；不能满足要求时返回NULL，此时原块不变 |
| void free(void *p)                  | 释放以前分配的动态存储块                                     |



### **几个整数函数**

几个简单的整数函数见下表，div_t和ldiv_t是两个预定义结构类型，用于存放整除时得到的商和余数。div_t类型的成分是int类型的quot和rem，ldiv_t类型的成分是long类型的quot和rem。

| 函数原型                    | 意义解释                                      |
| --------------------------- | --------------------------------------------- |
| int abs(int n)              | 求整数的绝对值                                |
| long labs(long n)           | 求长整数的绝对值                              |
| div_t div(int n, int m)     | 求n/m，商和余数分别存放到结果结构的对应成员里 |
| ldiv_t ldiv(long n, long m) | 同上，参数为长整数                            |



### **数值转换**

| 函数原型                   | 意义解释              |
| -------------------------- | --------------------- |
| double atof(const char *s) | 由串s构造一个双精度值 |
| int atoi(const char *s)    | 由串s构造一个整数值   |
| long atol(const char *s)   | 由串s构造一个长整数值 |



### **执行控制**

*1*）非正常终止函数abort。

原型是： *void abort(void);*

*2*）正常终止函数exit。

原型是：*void exit(int status);*

导致程序按正常方式立即终止。status作为送给执行环境的出口值，*0*表示成功结束，两个可用的常数为EXIT_SUCCESS，EXIT_FAILURE。

*3*）正常终止注册函数atexit。

原型是：*int atexit(void (\*fcn)(void))*

可用本函数把一些函数注册为结束动作。被注册函数应当是无参无返回值的函数。注册正常完成时atexit返回值*0*，否则返回非零值。

**与执行环境交互**

*1*）向执行环境传送命令的函数system。

原型是：*int system(const char \*s);*

把串s传递给程序的执行环境要求作为系统命令执行。如以NULL为参数调用，函数返回非*0*表示环境里有命令解释器。如果s不是NULL，返回值由实现确定。

*2*）访问执行环境的函数getenv。

原型是：*char \*getenv(const char \*s);*

从执行环境中取回与字符串s相关联的环境串。如果找不到就返回NULL。本函数的具体结果由实现确定。在许多执行环境里，可以用这个函数去查看“环境变量”的值。

### **常用函数bsearch和qsort**

*1*）二分法查找函数bsearch：

*void \*bsearch(const void \*key, const void \*base, size_t n, size_t size, int (\*cmp)(const void \*keyval, const void \*datum));*

函数指针参数cmp的实参应是一个与字符串比较函数strcmp类似的函数，确定排序的顺序，当第一个参数keyval比第二个参数datum大、相等或小时分别返回正、零或负值。

*2*）快速排序函数qsort：

*void qsort(void \*base, size_t n, size_t size, int (\*cmp)(const void \*, const void \*));*

qsort对于比较函数cmp的要求与bsearch一样。设有数组base[0],...,base[n-1]，元素大小为size。用qsort可以把这个数组的元素按cmp确定的上升顺序重新排列。



[C语言标准库函数 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/341336093)

