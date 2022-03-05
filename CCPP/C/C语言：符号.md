<h1 align="center">C语言：符号</h1>

[toc]

## c语言"->"

`foo->bar` is equivalent to `(*foo).bar`, i.e. it gets the member called `bar` from the struct that `foo` points to.

## C语言"-->"

Example:

```c
#include <stdio.h>
int main()
{
    int x = 10;
    while (x --> 0) // x goes to 0
    {
        printf("%d ", x);
    }
}
```

`-->` is not an operator. It is in fact two separate operators, `--` and `>`.

The conditional's code decrements `x`, while returning `x`'s original (not decremented) value, and then compares the original value with `0` using the `>` operator.

**To better understand, the statement could be written as follows:**

```c
while( (x--) > 0 )
```

