---
title: C标准函数库
---

stdio.h
-------

* FOPEN_MAX：最多能同时打开这么多个文件
* FILENAME_MAX：最长文件名
* 已经从流读取了数据，则在写之前要调用其中一个定位函数；已经写入了数据，在读取之前要用fflush或定位函数
* freopen：会先尝试关闭流。可以改变读写方式，可以改变标准流
* fclose：会刷新缓冲区。它也有可能失败，比如流是NULL
* fgetc和fputc是真正的函数，getc、putc、getchar、putchar都是宏；非标准有getch无回显无缓冲输入，getche有回显无缓冲输入
* int ungetc( char character, FILE* stream )：把一个字符返回到流中。每个流都允许至少一个字符被退回。退入流和写入流不同，如果用了定位函数，退回的字符会丢失
* sscanf和sprintf可以从字符串读取信息或把信息转换为字符串。sprintf没有指定缓冲区大小
* 其他人的笔记：http://www.cnblogs.com/likebeta/archive/2012/06/16/2551780.html
* EXIT_FAILURE
* 不用回车从终端读取字符：kbhit是非阻塞式的，但是是非标准的。conio.h中，getche是回显无缓冲的输入，getch是无回显无缓冲的输入。unix可用ioctl指定输入类型再用getchar（属于unix库）

fflush(NULL)刷新所有输出流

### 定位函数

* ftell：返回该流的当前位置距离起始位置的偏移量，理论上是当前位置距文件开始处的字节数，实际文本模式下会把`\r\n`算作一个字节
* int fseek( FILE* stream, long offset, int from )：from可以是SEEK_SET起始位置、SEEK_CUR当前位置、SEEK_END尾部位置。起始位置的offset必非负，后两者的偏移可正可负；可以是ftell返回的值，但必须是同一文件。定位到文件尾后会返回EOF。二进制模式下可能不支持SEEK_END。移动到末尾可以把from设为SEEK_END，offset设为0L

> 之所以存在这种限制，部分原因是文本流所执行的行末字符映射，文本文件的字节数可能和程序写入的字节数不同。因此，一个可移植的程序不能根据实际写入字符数的计算结果定位到文本流的一个位置。

* void rewind( FILE* stream )：将读/写指针设置回指定流的起始位置，同时清除流的错误提示标志；相当于`(void)fseek(stream, 0L, SEEK_SET)`，但rewind没有返回值，无法确定操作是否成功。
* fgetpos和fsetpos：分别是ftell和fseek的替代方案，用于解决文件长度超过long的范围（20亿字节）

### 改变缓冲方式

* void setbuf( FILE* stream, char* buf )
* int setvbuf( FILE* stream, char* buf, int mode, sisze_t size )

setfub设置的缓冲数组大小必须为BUFSIZ。小心使用自动类型的数组，确保代码块结束前流已被关闭，否则就分配到堆上。如果buf为NULL，会关闭缓冲，但操作系统可能有自己的缓冲方式。
setvbuf更常用。mode指定缓冲的类型：_IOFBF指定一个完全缓冲的流，_IONBF不缓冲，_IOLBF行缓冲。buf和size指定缓冲区，size应该为BUFSIZ的整数倍。如果buf为NULL，size应为0.
标准输入默认是行缓冲，标准错误是无缓冲。setbuf(stdin, NULL)可以清除输入流？

### 流错误函数

* 出现读取错误时，函数也会返回EOF
* int feof( FILE* stream )：如果流处于文件尾，返回非零值（正数）；但只有读函数刚好返回（完全处于）EOF才会这样，如果循环是先用feof判断，再读取，就会多循环一次，feof也不是这样用的。执行fseek、frewin或fsetpos可以清除
* int ferror( FILE* stream )：如果出现读/写错误就为真；读函数返回EOF时也有可能是出错，就使用这俩函数判断
* void clearerr( FILE* stream )：对指定流的错误标志重置

### 临时文件

* FILE* tmpfile(void)：该文件以wb+模式打开，当文件被关闭或程序终止时自动删除。
* char* tmpnam( char* name )：产生随机的字符串，参数为保存字符串的数组，长度至少为L_tmpnam。如果参数为NULL，函数会返回指向静态数组的指针。只要调用次数不超过TMP_MAX次，函数都能产生新的名字。

### 文件操纵函数

* int remove( const char* filename )
* int rename( const char* oldname, const char* newname )

### 二进制读写

* size_t fread( void *restrict buffer, size_t size, size_t count,
    FILE *restrict stream );
* size_t fwrite( const void *restrict buffer, size_t size, size_t count,
    FILE *restrict stream );

前者把stream读到buffer里，后者把buffer写到stream里。buffer实际是unsigned char*类型的。返回成功读写的个数，如果发生错误，不会读写count次。

stdlib.h

### 随机数

* int rand()
* void srand( unsigned int seed )：srand( (unsigned)time(0) )

### 终止执行

* void abort(void)
* void exit (int status)：main函数结束的时候会隐式调用
* void atexit( void(* func)(void) )：用于把**一些**函数注册为退出函数，当程序正常终止时（exit或main结束），那些退出函数将按照注册的反序被调用。不要在那些函数里使用exit

### 环境变量

* char* getenv(const char* name)：在环境变量里查找name，如果找到，返回路径，否则返回NULL
* 标准并未定义putenv函数

string.h
--------

strlen返回一个类型为size_t的值，而无符号整数的运算结果永远不可能小于零。所以以下两条语句不相等，第一条是正确的。
if( strlen(x) \>= strlen(y) )
if( strlen(x) - strlen(y) \>= 0 )
而无符号整数比有符号整数高级，所以if( strlen(x) - 100 \>= 0 )同样永远为真。

因为返回值的缘故，连接字符串可以嵌套调用：strcat( strcpy( dst, a), b);如果源字符串和目标字符串重叠，结果未定义；小心溢出；如果目标未初始化，用strcpy而不是strcat，否则会有垃圾值。

strncpy：如果源数组复制完了，指定数目的剩下的空间用'\0'填充；如果达到了指定数目还没有复制完，不会添加'\0'。
strncat：最多复制指定数目的字符，并添加一个'\0'。
strncmp：不要把返回值当作布尔值。

在一个字符串中查找特定字符，区分大小写，如果不存在则返回NULL，获取索引可以减去str。第二个函数查找最后一次出现的位置，第三个查找匹配group中任意一个字符的字符：
char* strchr( const char* str, char ch)
char* strrchr( const char* str, char ch)
char* strpbrk( const char* str, const char* group)

char* strstr( const char* s1, const char* s1)：在s1中查找是否有包含s2的子串。如果s2为空，返回s1；如果找不到，返回NULL。

查找一个字符串前缀：
size_t strspn( const char* str, const char* group)
size_t strcspn( const char* str, const char* group)
group指示一个或多个字符，strspn返回str起始部分匹配group中任意字符的数目，空格等空白字符和标点也有效。str + strspn可获得指针。strcspn对不符合group的字符计数。
如：

```c
strspn( "25,142,330,abc", "0123456789" ); // 2
strspn( "25,142,330,abc", ",0123456789" ); // 11
```

分隔字符串：
char* strtok( char* str, const char* sep );
sep指示用作分隔符的字符集合。当strtok函数执行时，它将会修改它所处理的字符串，如果源字符串不可修改，需要自己先复制一份，再传递给strtok函数。当函数第一次调用时，它会保存局部状态信息，所以不能用它同时解析两个字符串。当第一个参数为NULL时，函数会在保存好的字符串中继续查找;第二个参数可以变化，这样调用它的时候会查找不同的字符集合。

```c
void print_tokens( char* line )
{
    static char whitespace[] = " \t\f\r\v\n";

    for( char* token = strtok( line, whitespace );
        token != NULL;
        token = strtok( NULL, whitespace )
    )
        printf( "%s", token ）;
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

* void* memcpy( void* dst, const void* src, size_t length); // 不能重叠，如果类型相同，length可以为sizeof( src )；如果需要指定长度，count * sizeof( src[0] )
* void* memmove( void* dst, const void* src, size_t length); // 可以重叠（src会先复制到临时位置）
* void* memcmp( const void* a, const void* b, size_t length); // 按照无符号字符逐字节比较
* void* memchr( const void* a, int ch, size_t length); // 从a的位置开始查找字符ch第一次出现的位置，并返回指向该位置的指针
* void* memset( void* a, int ch, size_t length); // 把从a开始的length个字节设置为字符ch，ch只会取8位，不能用于int数组的初始化
* alloca是非标准函数，memory.h是非标准函数库

### 注意事项

* 应该对malloc**等**返回的指针进行检查，如果可用内存不够，它会返回NULL
* realloc可以将内存块扩大或缩小。如果是扩大，且后部有足够的空间，新内存会被直接添加到后边，且未初始化；如果是缩小，该内存块尾部的空间会直接被废弃掉
* 如果原先的内存块无法改变大小，或是扩大时后部控件不够，realloc将分配另一块正确大小的内存，并把原来内存的内容复制到新的块上，再自动free原来的部分。因此，在使用realloc后，就不应该再使用指向旧内存的指针
* 也要避免`p = realloc(p,2048);`这种写法，本来分配内存失败后原来的数据还是存在的，这样写就找不到了
* 如果realloc第一个参数为NULL，它的行为就和malloc一样
* free的参数只能是NULL或从malloc、calloc、realloc返回的指针，不能是指针+-后的结果，因为分配了多少内存是记录在某个偏移的地方的。

可移植类型：stdint.h和inttypes.h
--------------------------------

C99提供了与平台无关的类型，定义在stdint.h中。这些实现是可选的。

* 精确宽度类型：例如int32_t表示32位有符号类型，在有些32位int的系统中作为int的别名
* 最小宽度类型：比如在某些不提供int8_t类型的系统中，可用int_least8_t，最终可能实现为int16_t
* 最快最小宽度类型：int_fast8_t
* 最大整数类型：intmax_t，可能比long long还大，用jd打印

要打印这些可移植类型，在inttypes.h中提供了字符串宏。例如PRId32代表打印32位有符号值的说明符，在scanf中是SCNd32，比如可能被"d"替换。所以实际使用是要写成：`printf("a = %" PRId32 "\n", a);`

### 扩展的整形常量

如何表示int32_t类型的常量？INT32_C(445566)即可

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

## TODO

* asprintf：非标准库，好像也不是posix，是GNU的
* itoa：非标准库，可以把数字转换成字符串，可指定进制
