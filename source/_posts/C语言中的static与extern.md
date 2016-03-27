---
title: C语言中的static与extern
date: 2016-03-26 22:15:25
tags: C
---

最近在学习Operating System，作业需要使用C语言在一个toy OS中进行编程与学习。 在原始提供的源码中，发现有许多static和extern的关键字，故在此做一下总结与整理。

# extern
首先来看一下`extern`。其最主要的作用就是可以引用工程的其他文件中定义的变量或函数。
假设有两个文件foo1.c和foo2.c：
foo1.c
{% codeblock lang:c %}
int var;
{% endcodeblock %}
foo2.c
{% codeblock lang:c %}
extern int var;
{% endcodeblock %}
最终生成可执行文件时，需要分别编译compile和链接link：
{% codeblock %}
gcc -c foo1.c
gcc -c foo2.c
gcc foo1.o foo2.o -o foo  // link
{% endcodeblock %}
在编译过程中，编译器只会为foo2.c中的var分配内存，在foo1.c中仅仅生成一个名为var的symbol。而后在link过程中，链接器会将该symbol替换为foo2.c中的var的内存地址，从而达到指向同一变量的目的。

`extern`的实际应用场景主要在编写库library。打个比方，假设有名为Mylib的库，包含Mylib.c和Mylib.h。
MyLib.c:
{% codeblock lang:c %}
int Variable;
{% endcodeblock %}
MyLib.h:
{% codeblock lang:c %}
extern int Variable;
{% endcodeblock %}
main.c:
{% codeblock lang:c %}
#include "MyLib.h"

int main(void)
{
    Variable = 10;
    printf("%d\n", Variable);
    return 0;
}
{% endcodeblock %}
在.h文件中使用`extern`来引用.c库文件中定义的变量后，用户即可通过include头文件来使用库中的变量。

# static
回到之前提到的例子，如果foo2.c中的`extern`去掉又会怎样呢。
对于未初始化的变量，GCC会默认将它们视为同一个变量，效果与extern相同。而如果变量被初始化，在链接的过程中，gcc会报错：
```
/tmp/ccFN6SQZ.o:(.data+0x0): multiple definition of `var'
/tmp/ccbc0T4O.o:(.data+0x0): first defined here
```
假设foo1.c与foo2.c分别来自于两个不同的库，如果用户想要同时include这两种库时，就会产生multiple definition的错误。作为库的开发者，有责任防止这种多重定义的情况的发生，因此需要用到`static`。
`static`保证了定义的变量和函数只存在该文件范围内，其他的文件无法link这些变量和函数，从而避免了多重定义的问题。

除此之外，定义在函数中`static`变量在函数被调用后可维持该变量的值。
来看一下下面的例子：
{% codeblock lang:c %}
void foo()
{
    int a = 10;
    static int sa = 10;

    a += 5;
    sa += 5;

    printf("a = %d, sa = %d\n", a, sa);
}


int main()
{
    int i;

    for (i = 0; i < 10; ++i)
        foo();
}
{% endcodeblock %}
这种情况下，main函数调用foo函数，在foo执行完退出后，sa变量仍然保持其退出前的值。当foo被在此调用的时候，sa会在原来的基础上再加5。
当我们需用保留调用函数的某种状态而又不想创建全局变量时，这种方法就显得比较有用。然而，这种方法的滥用会导致程序的可读性和线程安全，所以需要谨慎使用。

在C++中，static还可用于class属性，在此不再赘述。


参考：
1. http://stackoverflow.com/questions/572547/what-does-static-mean-in-a-c-program/572550#572550
2. https://www.quora.com/What-is-the-deep-difference-between-static-extern-declaration-in-C-C++-programming
