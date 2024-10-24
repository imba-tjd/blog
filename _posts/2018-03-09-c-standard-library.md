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
        printf("%s", token);
}
```

字符串常量：
*"xyz" == 'x';
*("xyz" + 1) == 'y';
"xyz"[2] == 'z';

十进制转换为十六进制：
putchar("0123456789ABCDEF"[value%16]);

内存操作函数（位于string.h中）
------------------------------

与字符串的那些函数类似，但是不会以'\0'来结束。长度都是字节数

* void* memcpy(void* dst, const void* src, size_t len); // 不能重叠否则未定义，可能会从后往前复制；如果类型相同，len可以为sizeof(src)；如果需要指定长度，count * sizeof(src[0])
* void* memmove(void* dst, const void* src, size_t len); // 可以重叠（src会先复制到临时位置）
* void* memcmp(const void* a, const void* b, size_t len); // 按照无符号字符逐字节比较
* void* memchr(const void* a, int ch, size_t len); // 从a的位置开始查找字符ch第一次出现的位置，并返回指向该位置的指针
* void* memset(void* a, int ch, size_t len); // 把从a开始的len个字节设置为字符ch，ch只会取8位，不能用于int数组的初始化
* alloca是非标准函数，memory.h是非标准函数库
* C23：memccpy找到并复制了c后停止、memset_explicit用于清空秘密信息，防止编译器优化

### 注意事项

* 应该对malloc等返回的指针进行检查，如果可用内存不够，它会返回NULL
  * 对于32位程序，运行内存达到1.6GB后，分配10MB以上内存就可能失败了
* realloc可以将内存块扩大或缩小。如果是扩大，且后部有足够的空间，新内存会被直接添加到后边，且未初始化；如果是缩小，该内存块尾部的空间会直接被废弃掉
* 如果原先的内存块无法改变大小，或是扩大时后部控件不够，realloc将分配另一块正确大小的内存，并把原来内存的内容复制到新的块上，再自动free原来的部分。因此，在使用realloc后，就不应该再使用指向旧内存的指针
* 也要避免`p = realloc(p,2048);`这种写法，本来分配内存失败后原来的数据还是存在的，这样写就找不到了
* 如果realloc第一个参数为NULL，它的行为就和malloc一样
* free的参数只能是NULL或从malloc、calloc、realloc返回的指针，不能是指针+-后的结果，因为分配了多少内存是记录在某个偏移的地方的。也不能二次free之前已经释放过的
* realloc(p,0)未定义
* malloc和free是线程安全的，但不可重入，不能在信号处理函数中用

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

## TODO

* asprintf：非标准库，好像也不是posix，是GNU的
* itoa：非标准库，可以把数字转换成字符串，可指定进制
