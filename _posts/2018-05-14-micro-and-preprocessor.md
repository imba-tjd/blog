---
title: C的宏和预处理器
---

## 宏定义

### 优点

* 宏是类型无关的，比如求最大值的宏
* 一些任务无法用函数实现，比如#define MALLOC(n, type) ...

### 特性

* 宏定义可以分成几行，除了最后一行外，每行末尾都需要加一个反斜杠
* 可以利用相邻字符串自动拼接的特性
* 宏参数前加一个#，可以把替换后的宏参数转换为字符串
* ##可以把它两边的符号连成一个符号。这样可以在同一程序中实现不同类型的泛型，不会发生函数名字冲突的问题；需要把整个ADT作为宏体，宏参数里加上类型。

### 警告

* 宏不可以递归
* 只要一条宏定义语句里包含有操作符，就应该用括号把它括起来
* 宏定义越紧凑越好；表达式比语句好，单条语句比多条语句好
* 注意避免使用会导致二义性或副作用（i++、getchar()）的表达式或其他C语言元素
* 一定要让对宏进行扩展而得到的字符串成为一个完整的C语言元素
* 宏越简单越好，如果无法得到一个简单的宏，就应该把它定义成一个函数

### 多条语句

如果宏定义里有分号，某些情况下是空语句，没什么问题。但扩展后是多条语句，则在有些情况下有问题。

```c#
#define NL putchar('\n')
#define PRINT(a) printf(#a " = %d", (int) (a)); NL
#define TRACE if(1) printf("Done")

#define PR (fmt, val) printf(#val " = %" #fmt "\t", (val))
#define PRINT1 (f, x1) PR(f, x1), NL
#define PRINT2 (f, x1, x2) PR(f, mx1), PRINT1(f, x2)
#define PRINT3 (F, x1, x2, x3) PR(f, x1), PRINT2(f, x2, x3)

int main(void)
{
    if(0)
        TRACE;
    else // 这个else其实是和TRACE(X)里的if对应的
        puts("Not yet");

    for(int i = 0;i<2;i++)
        PRINT(i); // 这里只在最后换了一行，因为只有printf在循环体内
}
```

C和指针的解决办法：把PRINT宏定义里的分号换成逗号；在TRACE宏的末尾加上一条空else语句（而且这个加花括号没用）。

Milo Yip的解决办法：用 `do { ... } while(0)` 包裹成单个语句，换行还是要用反斜杠的。但这样不可以作为表达式。

### 获取编译器所有的预定义宏

* `gcc/clang -dM -E -x c /dev/null`

### 判断win和linux

* 都有的：linux下定义了linux和unix
* mingw定义了WIN32
* clang定义了__clang__
* 但win下的clang没有定义WIN32

```c
#if defined __clang__ && !(defined WIN32 || defined linux || defined unix)
#define WIN32
#endif
// 或者直接#ifdef linux 做与linux有关的操作 #else 做与win有关的操作
```

## 预处理器

### 预定义符号

* __FILE__：源文件名
* __LINE__：文件当前的行号。一般要配合宏使用而不能是函数，否则每次调用都相同
* __DATE__：被编译的日期
* __TIME__：被编译的时间
* __STDC__：如果编译器遵循ANSI C，其值为1，否则未定义
* __STDC_VERSION__：如果支持C99，值为199901L；如果支持C11，为201112L
* __func__：函数的定义块内可用，是函数名字的const char[]

### 预处理指令

* #define
* #undef
* #if：后面跟的是表达式，非零值为真
* #elif
* #else
* #endif
* #ifdef(#if defined)
* #ifndef(#if !defined)
* #include：可以使用绝对路径
* #line
* #progma：允许一些编译器选项或其他非标准的处理方式
* #error
* #：后面不跟东西，和空行效果一样
* 多个条件的#if不能用#ifdef和#ifndef，可见上面那个判读win的例子
* #：把后面的内容变成字符串
* ##：把两边的标识符连起来

```c
#define M(ch) putchar('\ch') // 无法做单引号和双引号内部的字符替换
#define M(X) ((#X)[0]) // 虽然M(a)可以返回'a'，但还不如直接要求输入'a'
```

### 命令行定义

* -Dname（仅仅定义标识符`name`）
* -Dname=stuff

### #pragma

C99提供_Pragma宏，比如它可以把_Pragma("abc 123")变为#pragma abc 123。这样就可以自定义含参宏，调用_Pragma宏，再展开成#pragma。

以下来自于《C Primer Plus》，但我觉得是错的。

```c
#define PRAGMA(X) _Pragma(#X) // 井号把X变成字符串，但难道不是只对STDC有效吗
#define LIMRG(X) PRAGMA(STDC CX_LIMITED_RANGE X)
LIMRG(ON) // 最终变为#pragma STDC CX_LIMITED_RANGE ON？
// 不能是#define LIMRG(X) _Pragma( "STDC CX_LIMITED_RANGE " #X )，因为预处理之后才会串联字符串？
```

## 三字母词

源代码中的“三字母词”，在编译阶段会被替换为“对应的字符”。对于以“?”开头的字符序列，如果不能与那9个匹配，编译器将保持原状；一旦匹配，编译器就会做替换。但是现在的编译器默认不开启了，gcc要加`-ansi`或者`-trigraphs`。

双字符（digraph）：与三字符不同的是，它不会替换双引号中的字符。

## 泛型选择（C11）

* _Generic ( x, int:0, double:1, default: 2 )
* x不会被求值，只判断类型，返回的值为匹配的标签后的值
* 如果没有default，且类型不匹配，会在编译期失败

```c
#define MYTYPE(X) _Generic((X),\
    int: "int",\
    double: "double",\
    default: "other"\
)
```

```c
#define SQRT(X) _Generic((X),\
   long double: sqrtl,\
   default: sqrt,\ // 可以是int和double
   float: sqrtf\
)(X)
```

## 静态断言

_Static_assert()可以在编译期检查表达式，如果为假则终止编译并打印错误信息。

```c
#include <limits.h>
_Static_assert(CHAR_BIT == 16, "16-bit char falsesly assumed");
```

## 参考

* 《C和指针》
* 《C Primer Plus》
