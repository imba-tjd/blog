---
title: C标准函数库
---

> 《C和指针》、《C Primer Plus》
> 别人的整理，未读：http://c.biancheng.net/ref/

stdio.h
-------

* FOPEN_MAX：最多能同时打开这么多个文件
* FILENAME_MAX：略
* 已经从流读取了数据，则在写之前要调用其中一个定位函数；已经写入了数据，在读取之前要用fflush或定位函数
* freopen可以用于重新打开流，它会先尝试关闭流。可以改变读写方式。可以改变标准流。
* fclose会刷新缓冲区。它也有可能失败，比如流是NULL。
* fgetc和fputc都是正真的函数，getc、putc、getchar、putchar都是宏
* int ungetc( char character, FILE* stream )：把一个字符返回到流中。每个流都允许至少一个字符被退回。退入流和写入流不同，如果用了定位函数，退回的字符会丢失
* sscanf和sprintf可以从字符串读取信息或把信息转换为字符串。sprintf没有指定缓冲区大小
* 其他人的笔记：http://www.cnblogs.com/likebeta/archive/2012/06/16/2551780.html

### 定位函数

* long ftell( FILE* stream )：返回该流的当前位置距离起始位置的偏移量。
* int fseek( FILE* stream, long offset, int from )：from可以是SEEK_SET起始位置、SEEK_CUR当前位置、SEEK_END尾部位置。起始位置的offset必非负，后两者的偏移可正可负；可以是ftell返回的值，但必须是同一文件。定位到文件尾后会返回EOF。在二进制中，SEEK_END可能不被支持。移动到末尾可以把from设为SEEK_END，offset设为0L

> 之所以存在这种限制，部分原因是文本流所执行的行末字符映射，文本文件的字节数可能和程序写入的字节数不同。因此，一个可移植的程序不能根据实际写入字符数的计算结果定位到文本流的一个位置。

* void rewind( FILE* stream )：将读/写指针设置回指定流的起始位置，同时清除流的错误提示标志；相当于`(void)fseek(stream, 0L, SEEK_SET)`，但rewind没有返回值，无法确定操作是否成功。
* int fgetpos( FILE* stream, fpos_t* position )：ftell的替代方案，把位置放到position上
* int fsetpos( FILE* stream, const fpos_t* position )：fseek的替代方案

fpos_t储存当前位置，但用它表示位置的方式不是标准定义的。因此唯一安全的方法就是作为上面那两个函数的参数。这样可以解决用long储存文件位置不够长的问题。

### 改变缓冲方式

* void setbuf( FILE* stream, char* buf )
* int setvbuf( FILE* stream, char* buf, int mode, sisze_t size )

setfub设置的缓冲数组大小必须为BUFSIZ。小心使用自动类型的数组，确保代码块结束前流已被关闭，否则就分配到堆上。如果buf为NULL，会关闭缓冲，但操作系统可能有自己的缓冲方式。
setvbuf更常用。mode指定缓冲的类型：_IOFBF指定一个完全缓冲的流，_IONBF不缓冲，_IOLBF行缓冲。buf和size指定缓冲区，size应该为BUFSIZ的整数倍。如果buf为NULL，size应为0.

### 流错误函数

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

### 排序和查找

* qsort
* bsearch

### gets的替代品

#### fgets

* char* fgets( char* str, int buffer_size, FILE* stream );，返回指向str的指针或NULL
* fgets读取直到遇到的第一个换行符，并且储存它，然后储存'\0'
* fgets的第二个参数指明了读入字符的最大数量，如果为n，它最多读入n-1个字符，然后储存'\0'
* 出错和到末尾都会返回NULL？可使用feof判断指定流是否到末尾
* fputs会把字符串直到'\0'按原样输出到流，不会在最后换行；所以即使fgets读满了，下一次fputs也可以继续断的地方输出

#### gets_s

* 它只从标准输入中读取数据
* 如果遇到换行符，丢弃它并储存'\0'
* 如果读满了，它会把目标数组首字符设为'\0'、读取并丢弃标准输入直到遇到换行符或EOF，然后返回空指针、调用其他“处理函数”

stdlib.h
--------

### 算数

* int abs( int value )
* long labs ( long value )
* div_t div( int numerator, int denominator )：用第二个参数除以第一个参数
* ldiv_t ldiv( long numer, long denom )
* double floor( double x )：返回不大于其参数的最大整数值，double是出于范围考虑
* double ceil( double x )：返回不小于其参数的最小整数值
* double fabs( double x )
* double fmod( double x, double y )：返回x整除y所产生的余数。

#### div_t

* int quot：商
* int rem：余数

### 随机数

* int rand( void )
* void srand( unsigned int seed )：srand( (unsigned)time(0) )

### 字符串转换

* int atoi( const char* str)
* long atol( const char* str)
* double atof( const char* str)
* long strtol( const char* str, char** unused, int base)
* unsigned long strtoul( const char* str, char** unused, int base)
* double strtod( const char* str, char** unused)

把字符串转换成数值，前导空白字符会被跳过。unused（如果不为NULL）保存一个指向字符串转换后第一个字符的指针，方便后续处理。base为基数，如果为0，八进制、十进制、十六进制都会被接受；否则应该在2到36的范围内（不含36）。

如果没有合法数值，函数返回0。如果太大（太小），返回LONG_MAX（LONG_MIN）、ULONG_MAX、HUGE_VAL（0），并在errno中储存ERANGE这个值。

### 终止执行

* void abort(void)
* void exit (int status)：main函数结束的时候会隐式调用
* void atexit( void(* func)(void) )：用于把**一些**函数注册为退出函数，当程序正常终止时（exit或main结束），那些退出函数将按照注册的反序被调用。不要在那些函数里使用exit

### 环境变量

* char* getenv(const char* name)：在环境变量里查找name，如果找到，返回路径，否则返回NULL
* 标准并未定义putenv函数

math.h
------

### 三角函数/双曲函数/对数和指数函数/幂函数

* double sin( double angle )
* double cos( double angle )
* double tan( double angle )
* double asin( double angle )
* double acos( double angle )
* double atan( double angle )
* double atan2( double x, double y )
* double sinh( double angle )
* double cosh( double angle )
* double tanh( double angle )
* double exp( double x )：e的x次方
* double log( double x )：自然对数
* double log10( double x )
* double pow( double x, double y )：在计算时可能要用对数，所以如果x为负**且**y为整数时会报错
* double sqrt( double angle )

### 浮点表示形式

* double frexp( double value, int* exponent )：fraction × 2^exponent = value，提供value；返回小数(fraction)，在0.5到1之间；把exponent存到第二个参数的内存里
* double ldexp( double fraction, int exponent )：返回fraction × 2^exponent，也就是上面的value
* double modf( double value, int* ipart )：把一个浮点数分成整数和小数两个部分，符号和原数一样。返回的是小数部分。

### tgmath.h

* 定义了泛型类型宏，名称与原来的一样
* 如果要使用函数而不是宏，可以把调用的函数名括起来：(sqrt)(x)
* _Generic表达式是实现tgmath.h最简单的方式

time.h
------

* clock_t clock( void )：返回从程序开始执行起处理器所消耗的时间（不是经过的时间），除以CLOCKS_PER_SEC以后就得出消耗的秒数。如果无法表示，返回-1
* time_t time( time_t* return_value )：如果参数不为NULL，也会储存在这个指针里。如果无法表示（2038年时会溢出？），返回-1。标准未规定time_t的编码方式，常见的为1970年1月1日0点0分0秒
* char* ctime( const time_t* time_value )：输出固定格式的时间Sun Jul 4 04:02:48 1976\n\0。下一次调用时，这个字符串会被覆盖。ctime实际上可能以asctime( localtime( time_value ) )实现
* double difftime( time_t time1, time_t time12 )：计算time1 - time2，结果为秒
* struct tm* gmtime( const time_t* time_value )：把时间值转换为UTC，以前被叫做GMT
* struct tm* localtime( const time_t* time_value )：把时间值转换为当地时间
* char* asctime( const struct tm* tm_ptr )：输出固定格式的时间，参见上面的ctime，注意参数区别
* size_t strftime( char* string, size_t maxsize, const char* format, const struct tm* tm_ptr )：把时间格式化成字符串，使用了和printf类似的原理，具体替换码略。如果maxsize不够大，函数返回-1且字符串内容未定义
* time_t mktime( struct tm* tm_ptr )：把tm转换为time_t，tm_wday和tm_yday会被忽略，其他字段的值的范围也无需原来的限制。转换之后tm会进行规格化，各字段的值转化到通常范围内。可以用于判断某个特定的日期属于星期几

### tm结构

* tm_sec：秒数，范围为0-60，允许”闰秒”
* tm_min：0-59
* tm_hour：0-23
* tm_mday：当月的日期
* tm_mon：0-11
* tm_year：1900之后的年数。所以要加上1900才是真正的年数
* tm_wday：星期，0为星期天，6为星期六
* tm_yday：天数，0-365
* tm_isdat：夏令时标识

非本地跳转setjmp.h
------------------

* int setjmp( jmp_buf state )：没有使用过的时候调用它永远返回0
* void longjmp( jump_buf state, int value )：跳转到最开始setjmp(state)的位置，并且让setjmp(state)以后每次调用都返回value值
* 顶层函数返回后再调用longjmp很可能会失败

```c
jmp_buf restart;

int mian(void)
{
    int value;
    value = setjmp(restart);
    switch(setjmp(restart)){
    default:严重错误
    case 1:小错误
    case 0:正常的处理过程。如果中间发生错误，使用longjmp
}
```

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

ctype.h
-------

* iscntrl：任何控制字符
* isspace：空白字符，包括空格、换页\f、换行\n、回车\r、制表符\t、垂直制表符\v
* isdigit：十进制数字0-9
* isxdigit：十六进制数字，包括十进制数字、小写字母a-f、大写字母A-F
* islower：小写字母a-z
* isupper：大写字母A-Z
* isalpha：字母a-z或A-Z
* isalnum：字母或数字，0-9、a-z、A-Z
* ispunct：标点符号，任何不属于数字或字母的图形字符（可打印符号）
* isgraph：任何图形字符
* isprint：可打印符号，包括图形字符和空白字符
* int tolower( int ch )
* int toupper( int ch )

直接测试或操纵字符会降低可移植性：if( ch\>= 'A' && ch \<= 'Z')在使用ASCII字符集的机器上能正常允许，但在使用EBCDIC字符集的机器上将会失败。而isupper总是可以成功。

输出错误信息errno.h
-------------------

* errno：当标准库函数发生错误时，会在一个外部整型变量errno中保存错误代码
* char* strerror( int error_number )：根据错误值返回预设的错误信息（位于\<string.h\>）
* void perror( const char* messsage)：先输出自定义信息，再输出strerror的信息；相当于printf("%s %s", message, strerror(errno))（位于 \<stdio.h\>）

signal.h
--------

### 预定义信号

* SIGABRT：由abort函数引发的信号
* SIGFPE：发生一个算数错误，比如溢出或除以0。
* SIGILL：非法指令、把数据当成代码
* SIGSEGV：访问非法内存（野指针或越界）
* SIGINT：按下ctrl + c时触发
* SIGTERM：也是终止程序的请求
* SIGKILL、SIGSTOP：不能被捕捉也不会被忽略，终止程序
* SIGALRM：由alarm函数结束后产生
* SIG_DFL：函数指针，表示默认的处理函数
* SIG_IGN：函数指针，忽视信号

### 函数

* int raise( int sig )：引发一个信号
* signal( int sig, void ( * handler )(int) )设置指定信号的回调函数，返回该信号前一个回调函数的函数指针

### 使用方式

* 当一个已经设置了信号处理函数的信号发生时，系统首先恢复对该信号的缺省行为，这样是为了防止如果信号处理函数内部也发生这个信号可能导致的无限循环。然后调用信号处理函数。所以如果想要多次捕捉同一个信号，要在调用信号处理函数返回前再用一次signal函数
* 如果信号是异步的，也就是不是由于调用abort或raise函数引起的，信号处理函数便不应调用除signal之外的任何库函数（除了exit和非SIGABRT里的abort），否则结果未定义。而且，信号处理函数可能只能向类型为volatile sig_atomic_t的变量赋值，无法访问其他任何静态数据
* sig_atomic_t定义了一种CPU可以原子访问的数据类型
* 从信号处理函数返回会从信号发生的地点恢复执行，除了SIGFPE
* 进程收到一个信号后不会被立即处理，而是在恰当时机进行处理，比如内核态返回用户态之前

stdarg.h
--------

* 宏无法判断实际存在的参数的数量
* 这些宏无法判断每个参数的类型，如果指定了错误的类型，结果是不可预测的
* 许多库实现了getopt函数用于处理命令行参数，但它不是标准库函数
* `##`加在函数参数的`__VA_ARGS__`前，如果可变参数没有传参，这个符号就把前面的`,`去掉，这样编译就不会出错

```c
#include <stdarg.h>

double average(int num,...)
{
    va_list var_arg;
    double sum = 0;

    va_start(var_arg, num);
    for (int i = 0; i < num; i++)
        sum += va_arg(var_arg, int);
    va_end(var_arg);

    return sum / num;
}
```

### 有关的函数（位于stdio.h中）

* int vprintf( const char *format, va_list vlist )
* int vfprintf( FILE *stream, const char *format, va_list vlist )
* int vsprintf( char *buffer, const char *format, va_list vlist )

### 变参宏

```c
#define PR(...) printf(__VA_ARGS__)
PR("weight = %d", wt); // 可以接受多个宏参数
```

locale.h
--------

### 改变locale的效果

* 可能向正在执行的程序所使用的字符集增加字符
* 打印的方向可能会改变
* 小数点符号可能会改变
* isalpha、islower、isspace和isupper可能包含更多的字符
* 对照序列可能会改变
* 时间和日期的格式可能会改变

可移植类型：stdint.h和inttypes.h
--------------------------------

C99提供了与平台无关的类型，定义在stdint.h中。这些实现是可选的。

* 精确宽度类型：例如int32_t表示32位有符号类型，在有些32位int的系统中作为int的别名
* 最小宽度类型：比如在某些不提供int8_t类型的系统中，可用int_least8_t，最终可能实现为int16_t
* 最快最小宽度类型：int_fast8_t
* 最大整数类型：intmax_t，可能比long long还大

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

Assert.h
--------

* static_assert宏。不包含的时候要用_Static_assert
* assert()

unistd.h
--------

* 不是标准库的，而是posix函数库
* sleep(int seconds)

## TODO

* asprintf：非标准库，好像也不是posix，是GNU的
* https://github.com/nothings/stb https://github.com/srdja/Collections-C https://github.com/faragon/libsrt https://github.com/P-p-H-d/mlib https://github.com/troydhanson/uthash
* libuv
