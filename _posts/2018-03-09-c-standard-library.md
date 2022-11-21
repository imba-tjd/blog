---
title: C标准函数库
---

string.h
--------

strlen比较两个字符串的长度要用if(strlen(x)>=strlen(y))而非if(strlen(x)-strlen(y)>=0)，后者永远为真

因为返回值的缘故，连接字符串可以嵌套调用：strcat(strcpy(dst, a), b);如果源字符串和目标字符串重叠，结果未定义；小心溢出；如果目标未初始化，用strcpy而不是strcat，否则会有垃圾值。

strncpy：如果源数组复制完了，指定数目的剩下的空间用'\0'填充；如果达到了指定数目还没有复制完，不会添加'\0'。
strncat：最多复制指定数目的字符，并添加一个'\0'。
strncmp：不要把返回值当作布尔值。

在一个字符串中查找特定字符，区分大小写，如果不存在则返回NULL，获取索引可以减去str。第二个函数查找最后一次出现的位置，第三个查找匹配group中任意一个字符的字符：
char* strchr(const char* str, char ch)
char* strrchr(const char* str, char ch)
char* strpbrk(const char* str, const char* group)

char* strstr(const char* s1, const char* s1)：在s1中查找是否有包含s2的子串。如果s2为空，返回s1；如果找不到，返回NULL。

查找一个字符串前缀：
size_t strspn(const char* str, const char* group)
size_t strcspn(const char* str, const char* group)
group指示一个或多个字符，strspn返回str起始部分匹配group中任意字符的数目，空格等空白字符和标点也有效。str + strspn可获得指针。strcspn对不符合group的字符计数。
如：

```c
strspn("25,142,330,abc", "0123456789"); // 2
strspn("25,142,330,abc", ",0123456789"); // 11
```

分隔字符串：
char* strtok(char* str, const char* sep);
sep指示用作分隔符的字符集合。当strtok函数执行时，它将会修改它所处理的字符串，如果源字符串不可修改，需要自己先复制一份，再传递给strtok函数。当函数第一次调用时，它会保存局部状态信息，所以不能用它同时解析两个字符串。当第一个参数为NULL时，函数会在保存好的字符串中继续查找;第二个参数可以变化，这样调用它的时候会查找不同的字符集合。

```c
void print_tokens(char* line)
{
    static char whitespace[] = " \t\f\r\v\n";

    for(char* token = strtok(line, whitespace);
        token != NULL;
        token = strtok(NULL, whitespace)
    )
        printf("%s", token ）;
}
```

复制一个字符串：
char* strdup(char* str);
该函数非标准库函数，返回str的拷贝。内部使用strlen计算需要分配内存的大小，malloc分配空间（一般实现）；而strcpy需要自己分配空间。用完后需要free释放，否则会内存泄漏。因为用了strlen，所有str参数不能为NULL，否则会报段错误；因为是非标准的，某些C++库如果重写它使用了new，则释放就要用delete了。

字符串常量：
*"xyz" == 'x';
*("xyz" + 1) == 'y';
"xyz"[2] == 'z';

十进制转换为十六进制：
putchar("0123456789ABCDEF"[value%16]);

内存操作函数（位于string.h中）
------------------------------

与字符串的那些函数类似，但是不会以'\0'来结束。

* void* memcpy(void* dst, const void* src, size_t len); // 不能重叠否则未定义，可能会从后往前复制；如果类型相同，len可以为sizeof(src)；如果需要指定长度，count * sizeof(src[0])
* void* memmove(void* dst, const void* src, size_t len); // 可以重叠（src会先复制到临时位置）
* void* memcmp(const void* a, const void* b, size_t len); // 按照无符号字符逐字节比较
* void* memchr(const void* a, int ch, size_t len); // 从a的位置开始查找字符ch第一次出现的位置，并返回指向该位置的指针
* void* memset(void* a, int ch, size_t len); // 把从a开始的len个字节设置为字符ch，ch只会取8位，不能用于int数组的初始化
* alloca是非标准函数，memory.h是非标准函数库
* C23：memccpy找到并复制了c后停止

### 注意事项

* 应该对malloc**等**返回的指针进行检查，如果可用内存不够，它会返回NULL
* realloc可以将内存块扩大或缩小。如果是扩大，且后部有足够的空间，新内存会被直接添加到后边，且未初始化；如果是缩小，该内存块尾部的空间会直接被废弃掉
* 如果原先的内存块无法改变大小，或是扩大时后部控件不够，realloc将分配另一块正确大小的内存，并把原来内存的内容复制到新的块上，再自动free原来的部分。因此，在使用realloc后，就不应该再使用指向旧内存的指针
* 也要避免`p = realloc(p,2048);`这种写法，本来分配内存失败后原来的数据还是存在的，这样写就找不到了
* 如果realloc第一个参数为NULL，它的行为就和malloc一样
* free的参数只能是NULL或从malloc、calloc、realloc返回的指针，不能是指针+-后的结果，因为分配了多少内存是记录在某个偏移的地方的。也不能二次free之前已经释放过的
* realloc(p,0)未定义

## 可移植类型：stdint.h和inttypes.h

* 精确宽度类型：例如int32_t表示32位有符号类型，在有些32位int的系统中作为int的别名
* 最小宽度类型：比如在某些不提供int8_t类型的系统中，可用int_least8_t，最终可能实现为int16_t
* 最快最小宽度类型：int_fast8_t
* 最大整数类型：intmax_t，可能比long long还大，用jd打印
* 表示int32_t类型的常量：INT32_C(445566)
* 要打印这些可移植类型，inttypes.h中提供了字符串宏。例如PRId32代表打印32位有符号值的说明符，在scanf中是SCNd32，使用示例：`printf("a=%" PRId32 "\n", a);`

明示常量：limits.h和float.h
---------------------------

分别提供了整数与浮点数类型大小限制的信息。

* CHAR_BIT：char类型的位数
* LLONG_MAX：long long类型的最大值
* FLT_MANTz_DIG：float类型的尾数位数
* FLT_DIG：float类型的十进制最少有效数字位数
* FLT_MIN_10_EXP：以10为底带全部有效数字的float类型的最小负指数
* FLT_MIN：保留全部精度的float类型最小正数
* FLT_EPSILON：1.00和比1.00大的最小float类型值之间的差值

宽字符支持：wchar.h和wctype.h
-----------------------------

wchar.h提供类似stdio.h的函数，在标准输入输出返回EOF的时候返回WEOF：

* fwprintf/fwscanf
* swprintf/swscanf
* wprintf/wscanf
* wint_t fgetwc
* wchar_t* fgetws
* wint_t fputwc
* getwc/getwchar/putwc/putwchar/ungetwc

wchar.h参照string.h提供了字符串控制函数，一般而言，用wcs代替str。几个内存处理函数前加w。

wctype.h提供与ctype.h类似的宽字符函数：

* iswalpha
* ...

wtypes.h和WCHAR是Windows的头文件。

在C++下，C的流是有“方向”的，最初流是无方向的，一旦使用了窄系函数，就会确定下来，就不能再用宽系函数了，只有freopen才能更改。

## TODO

* asprintf：非标准库，好像也不是posix，是GNU的
* itoa：非标准库，可以把数字转换成字符串，可指定进制
