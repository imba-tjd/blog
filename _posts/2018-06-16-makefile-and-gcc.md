---
title: Makefile和gcc
---

## Makefile

* 缩进只能用tab
* “目标”其实是要生成的文件名，如果存在那个文件且依赖最新，就自动不运行。依赖当有另一个目标时就执行对应的目标，当没有时仍视为文件
* 变量赋值：=会自动推导最终赋值作为结果，与使用它的位置无关；:=是覆盖式赋值，与一般语言的赋值类似；+=相当于字符串拼接；?=当变量未定义时赋值，已定义时什么也不做
* 多线程：`-j"$(nproc 2> /dev/null || sysctl -n hw.ncpu)"`，单纯使用-j将不限制
* 依赖要加上.h，但这不是人类能完成的事，因为.h之间可以存在依赖。gcc有选项生成，但不如用别的构建工具了

```makefile
include a.mk # 引用其他的makefile，可以写绝对路径，可以使用通配符和变量

# 变量定义，可在命令行中用key="val"重写，用$()使用
objects = main.o \ # 用反斜杠换行
	utils.o

# 显式规则
prog: $(objects) # 如果make时没有指定目标就用第一个目标
	cc -o prog $(objects)

# 隐式规则自动推导：.o会自动把.c加入依赖
main.o: defs.h
files.o: defs.h buffer.h command.h

# 多个目标的公共依赖，另一种隐式推导：
$(objects): defs.h
command.o files.o: command.h
search.o files.o: buffer.h

# 伪目标，不是文件，只是标签；不会因为存在一个叫做clean的文件而认为已是最新不执行，所有依赖总会被执行
.PHONY: clean cleandiff
clean: cleandiff
	-rm prog $(objects) # 减号表示出现错误也继续执行。Win下也用rm

TARGET = hello
OBJ = lib.o
CC = gcc
CFLAGS = -Wall
$(TARGET): $(OBJ)
	$(CC) $(CFLAGS) -o $@ $^  # $@表示目标，$^表示依赖。其它$变量：https://blog.csdn.net/qu1993/article/details/88871799
%.o: %.c  # 某种通配
	$(CC) $(CFLAGS) -o $@ -c $<  # $<表示第一个依赖，此处每次只有一个依赖

ifeq (, $(shell which curl))
	$(error "No curl in $$PATH, please install")
endif

$(wildcard *.o) # wildcard关键字进行扩展
$(patsubst %.c ,%.o ,$(src)) # 表示从src变量（列表）中找到所有.c，替换为.o
$(foreach item, 以空格分隔的列表, 含有$(item)的拼接字符串结果)
$(dir xxx/yyy) 取目录部分，此处返回值为xxx/。$(notdir)去掉目录部分。$(basename)去掉扩展名
$(filter 模式, 列表)

	@echo 正在编译 # 单独的命令，也要用tab。@表示不显示命令本身，类似于bat的
	cd /etc; pwd # 不同命令之间独立，如果不用分号而是换行，又会回到cd前的地方
	cd subdir && make # 进入子文件夹make；父makefile定义的变量手动用export命令可以传递到子makefile中，但SHELL和MAKEFLAGS变量会自动传递

多目标、静态模式：如果command相似，可以使用此功能，会进行一些扩展。见https://blog.csdn.net/haoel/article/details/2890
```

## CMake

* pip install cmake ninja
* CMakeLists.txt
  * 参数（列表）也可以用分号隔开，分号和空格是等价的
* cmake -B build -G "MinGW Makefiles"; cmake --build build -v/--verbose -j/--parallel -t/--target 子项目 -- -传递给make或ninja的参数
  * -DCMAKE_BUILD_TYPE=Debug Release RelWithDebInfo MinSizeRel，适用于Makefile。对于VS用--build --config Release
  * -G -D等只要用第一次，之后会保留。如果要刷新，可删除CMakeCache.txt
  * --install build、-t clean
* 另一种方式：mkdir build; cd build; cmake ..; make -j VERBOSE=1; make install; make clean
* 项目组织形式：include与src同级。include/项目名/xxx.h放公开接口，被install后会放在/usr/local/include里因此要加项目名，使用者用<项目名/xxx.h>
* 各个版本新增的特性：https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/chapters/intro/newcmake.html

```cmake
cmake_minimum_required(VERSION 3.5)
set(CMAKE_CXX_STANDARD 17) set(CMAKE_CXX_STANDARD_REQUIRED ON) # 后者不设置时若编译器不支持会自动降低版本
set(CMAKE_CXX_EXTENSIONS OFF) # 默认on，表示启用GUN扩展。这些需要在project之前设置
project(hello VERSION 1.0)  # 产生PROJECT_NAME、PROJECT_SOURCE_DIR。还有CMAKE_SOURCE_DIR表示根目录，BINARY_DIR一般就是build。CURRENT表示当前目录，若当前不存在project语句时PROJECT目录就为上层的。LANGUAGE默认为C和CXX。还有VERSION x.y.z DESCRIPTION HOMEPAGE_URL

add_executable(${PROJECT_NAME} main.cpp utils.cpp)  # 生成exe，第一个参数是文件名

if(WIN32) set(CMAKE CMAKE_SHARED_LIBRARY_PREFIX "") endif() # 使得Win下的dll不加lib前缀
add_library(hello_library STATIC或SHARED  # 生成库。默认STATIC，或用BUILD_SHARED_LIBS:BOOL=ON表示默认动态库。header-only库用INTERFACE。SHARED会自动-D库名_EXPORT
    src/Hello.cpp
)
add_library(Foo::Bar ALIAS Bar)

file(GLOB或GLOB_RECURSE SOURCES "src/*.cpp") # 之后可用${SOURCES}表示所有源文件。但不推荐这样做，因为添加了cpp后本文件无法反映变化，缓解办法是再加CONFIGURE_DEPENDS
aux_source_directory(src SOURCES) # 另一种方式，不会递归包含子目录。可以多次对同一个结果变量使用来添加


target_include_directories(hello_library  # 相当于-I。第一个参数是目标
    PUBLIC  # PRIVATE表示生成目标时添加本语句，INTERFACE表示目标被link时添加本语句，PUBLIC表示二者都是且默认。一般目标是库就用PUBLIC，是header-only库就用INTERFACE
        ${PROJECT_SOURCE_DIR}/include
)

target_link_libraries(hello_binary  # 相当于-l。target不存在时会寻找系统库
    PRIVATE  # 目标是exe时一般用PRIVATE，是库时如果依赖在头文件里出现了则用PUBLIC，只在cpp里出现则用PRIVATE
        hello_library # 会自动引入它的PUBLIC和INTERFACE的-I的内容
)

target_compile_definitions(hello
    PRIVATE MYMACRO=1  # 相当于-DMYMACRO=1；此条也兼容加-D
)
target_compile_options
set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DEX2" CACHE STRING "Set C++ Compiler Flags" FORCE)  # 全局参数，FORCE表示忽略命令行调用时-D的覆盖，STRING是类型，后面那个是注释
# 还有CMAKE_C_FLAGS CMAKE_LINKER_FLAGS。可以在cmake命令行时加-D设定


find_package(Boost 1.46.1 REQUIRED COMPONENTS filesystem system) # 另一个支持CMAKE且被install了的包
源码依赖：add_subdirectory()
下载GitHub的内容：https://cmake.org/cmake/help/latest/module/FetchContent.html

添加预编译的库：
add_library(foo SHARED IMPORTED) 如果位置不在项目中再加GLOBAL
set_property(TARGET foo PROPERTY IMPORTED_LOCATION "/dir-of-libfoo")
再对mylib加target_include_directories，必为INTERFACE
或者直接使用者link加绝对路径。或者配合find_library和find_path在多个地方寻找指定文件保存到变量里再使用
还能把后者逻辑放到cmake/FindXxx.cmake里，名称按约定的，使用时先set(CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake;${CMAKE_MODULE_PATH}")，就可以find_package()了。或者用include()添加文件，手动加


if(Boost_FOUND) # 这里面使用变量无需${}。运算符支持MATCHES、STREQUAL。一元NOT、NOT DEFINED、TARGET是否存在指定目标
    message ("boost found")
    include_directories(${Boost_INCLUDE_DIRS})
else()  # 不用REQUIRED时手动处理不存在的场景
    message (FATAL_ERROR "Cannot find Boost")
endif()

option (USE_MYMATH  # 是set BOOL类型的简写，另外能在图形化配置或cmake -LH中显示出来
       "Use provided math implementation" ON)
if (USE_MYMATH) ... endif()

foreach(var IN ITEMS foo bar baz) ...
foreach(var IN LISTS my_list) ...
foreach(var IN LISTS my_list ITEMS foo bar baz) ...

function(my_func ret_var_name) # 没有返回值，使用者要把期待接受返回值的变量名传进去
    set(${ret_var_name} "return value" PARENT_SCOPE)
endfunction()

target_compile_options(demo PRIVATE
    $<$<OR:$<CXX_COMPILER_ID:GNU>,$<CXX_COMPILER_ID:Clang>>:"-Wall">
    $<$<CXX_COMPILER_ID:MSVC>:"/W4">)
$<$<BOOL:${WIN32}>:
    # for Windows
>
$<$<NOT:$<BOOL:${WIN32}>>:
    # for POSIX
>
# 以上实际一般用IF (WIN32)、UNIX、MSVC

find_program(CCACHE ccache)

set_target_properties(tgt PROPERTEIS
  属性名1 值1
  属性名2 值2
)

$ENV{PATH}

https://cmake.org/cmake/help/latest/guide/tutorial/index.html
https://modern-cmake-cn.github.io/Modern-CMake-zh_CN/
https://github.com/Akagi201/learning-cmake
https://www.youtube.com/watch?v=y7ndUhdQuU8 https://www.youtube.com/watch?v=y9kSr5enrSk
https://github.com/onqtam/awesome-cmake
https://cmake.org/cmake/help/latest/command/target_sources.html
install: https://github.com/ttroy50/cmake-examples/blob/master/01-basic/E-installing/CMakeLists.txt https://github.com/BrightXiaoHan/CMakeTutorial/blob/master/Installation/README.md
```

## gcc

* gcc -o如果没有后缀，会自动加exe；touch也是这样
* 链接过程中，需要进行符号解析，并且是按照顺序解析；如果库链接在前，就可能出现库中的符号不会被需要，链接器不会把它加到未解析的符号集合中，那么后面引用这个符号的目标文件就不能解析该引用，导致最后链接失败。因此链接库的一般准则是将它们放在命令行的结尾
* [ccache](https://github.com/ccache/ccache)可以缓存编译信息
* -Ofast可开启最高优化，包含O3和ffast-math等，但可能产生不符合标准的行为
* /bin/gcc-10、/bin/gcc、/bin/x86_64-linux-gnu-gcc
* -flto：编译和链接都要用，与make -j同时用或自动多线程时加=auto，-fno-fat-lto-objects能减少生成时间但无法进行普通链接，只需在编译时用，没看懂默认是否启用。clang支持=thin比普通的更好
* -march指定代码能运行的最小CPU，默认x86-64；设为native可能就等于skylake这样，就可能不能运行在其它机器上。-mtune默认generic，改成native可生成为本机优化的代码
* -g等于-g2，-g3还会包含宏定义体积更大，-g0禁用前面的-g，-ggdb(3)产生仅限于gdb的信息，-Og保留调试信息且优化。-gsplit-dwarf能减少一些体积，把信息放到dwo文件中，能提升链接速度，但无法与-flto一起使用
* -###为dry-run，能显示具体编译用到的命令
* --help=xxx能显示更多选项帮助，在前面加-Q改为看是否启用
* -fuse-ld=gold比普通的ld快，MinGW不自带
* -s：去掉符号信息
* --include：相当于`#include`，与-I无关
* -rdynamic：使得可执行程序也导出符号，只在Linux下有效

### 编译步骤

1. gcc -E 或 cpp 进行预处理，一般以.i为后缀，如果不加-o会直接输出到终端里
2. gcc -S 或 cc1 生成汇编（称为编译）
3. gcc -c 或 as 生成目标文件(.o)二进制代码
4. 生成动态库：gcc -shared -fPIC 或 ld
  * fPIC是必须的，其实应在-c时使用。fpic产生的代码更小更快但在某些平台有限制一般不用，或者好像x86_64上二者相同PowerPC上才不同。fPIE用于可执行文件，可在编译时都用pic，只在最后链接时用pie
  * Win下-mdll或--dll现在与shared相同，简化起见永远使用shared。Win下是默认PIC的，不需要加
5. ar rcsv libname.a src.o ... 生成静态库。-t显示包含哪些.o
  * 静态库转动态库：-Wl,--whole-archive -lxxx。此参数本意是包含所有静态库中的符号，不管是否用到；默认使用静态库只会载入用到的

### 库

* Linux
  * .a是静态库，由多个.o组成，编译时当作.o附加到参数中就是
  * .so是动态库，需要-L指定库存在的文件夹，当前目录也不可省，再-l库名
  * -l也支持静态库但优先用动态的，可在前面加-Wl,-Bstatic优先用静态的
  * 一般库都以lib开头，-l时省略前缀和后缀。-l:可精确指定名字
  * 指定产物运行时要搜索的动态库目录：`-Wl,-rpath=.`，即Linux默认不会寻找CWD的链接库。另外还可用LD_LIBRARY_PATH和LD_PRELOAD
  * 假如两个库ab，a依赖b，在编译a时可以完全不管b，只在编译可执行文件时再去指定b；也可以编译a时链接b，则编译可执行文件时就不再需要指定b
* Windows
  * .lib是静态库，.dll是动态库
  * 另有一种dll，函数符号在lib中，虽然类型为T但没有实现，还会再生成一个`__imp_`开头的I符号，实现在dll中。MinGW产生用-o example.dll -Wl,--out-implib=libexample.a。使用时将它当作.o编译
    * 对于现在的MinGW，作为使用者，用上面这条、把dll当作.o、用-l，产生的效果一样
    * 假如两个库ab，a依赖b，在编译a时可以自己生成.o，但必须要引用b才能生成dll
  * MinGW的-lxxx的搜索顺序：libxxx.dll.a xxx.dll.a libxxx.a cygxxx.dll libxxx.dll xxx.dll
* 工具
  * 查看程序所依赖的共享库：ldd
  * 查看库导出的符号，但必须有调试符号：nm -C，其中-C能解码C++符号，-l列出源文件行号。类型T是本库实现的，U是引用外部的
  * 上面两条都支持：objdump -p
* 理论上MinGW可以直接链接.lib的，但32和64不能通用。lib转a可以见：https://stackoverflow.com/questions/11793370/how-can-i-convert-a-vsts-lib-to-a-mingw-a ，但我试了一下无效
* 增强安全性的参数：https://gist.github.com/jrelo/f5c976fdc602688a0fd40288fde6d886 https://security.stackexchange.com/questions/24444
* -ftrapv在linux下整数溢出时会触发core dump，会减慢速度
* 现在的编译器对未定义行为优化得太多了，但写底层代码时又时又无法避免。此时就要加-fno-strict-aliasing和-fwrapv
* Linux允许多个库存在相同的符号，会使用先链接的那一个，即命令中的链接顺序会影响结果。Win会报错
* 减少体积
  * -Wl,--as-needed
  * -Wl,--strip-all
  * -Wl,-dead_strip 好像只有lld支持
  * -ffunction-sections -fdata-sections -Wl,--gc-sections
  * strip -s或--strip-unneeded

### 超级静态的编译

```
-static-libgcc -static-libstdc++ -Wl,-Bstatic,--whole-archive -lwinpthread -Wl,--no-whole-archive
```

* MinGW32编译出来的程序可能依赖libgcc_s_sjlj-1.dll等，使用-static-libgcc就可避免依赖。好像MinGW64不会。另外g++必须用shared-libgcc

### 临时禁用警告

```c
#pragma GCC diagnostic push  // 如果在整个本文件中忽略，也可不用它
#pragma GCC diagnostic ignored "-Wunused-value"
...
#pragma GCC diagnostic pop
```

## 编译器

* 平台：指令集体系结构(ISA) - os - libc（一般只有Linux区分，其它os自带）。如x86_64-linux-gnu。ISA还有aarch64。libc还有musl、android(bionic)。Win下libc分为gnu和msvc。不同平台之间的程序不通用
  * ISA后还可能有厂商，win下一般是unknown或者pc，linux下可以是ubuntu
* 构建(build) - 宿主(host) - 目标(target)。host是运行编译器的平台，target是编译器生成的程序运行的平台。
  * build和host不同，称为加拿大编译(Canadian)。host和target不同，称为交叉编译。当构建和目标相同但host不同时又称为反向编译(Crossback)
  * clang（在编译编译器本身时）不区分target

### Clang

* 安装：https://apt.llvm.org/
* Win: https://github.com/mstorsjo/llvm-mingw 有ucrt，但支持太多的target导致可执行文件有点多
* https://gitee.com/qabeowjbtkwb/windows-hosted-llvm-clang

### MinGW

* https://github.com/brechtsanders/winlibs_mingw/releases 下x86_64-posix-seh-*.7z 没有pretty-printer，有ucrt
* http://www.equation.com/servlet/equation.cmd?fa=fortran 线程模式为win32。安装必须用它的程序，可以自己解压但不能直接复制，因为内部用了bzip2，env文件控制自动添加PATH
* https://gcc-mcf.lhmouse.com/ 小文件太多；有ucrt
* https://github.com/niXman/mingw-builds-binaries https://github.com/RoEdAl/ucrt-mingw-builds
* https://jmeubank.github.io/tdm-gcc/ 自动添加系统级别的PATH，目前最新10.3
* https://nuwen.net/mingw.html
* https://osdn.net/projects/mingw/releases/ MinGW32，只能用mingw-get-setup.exe这个在线安装器，因为各个组件都分散了。不如用TDM-GCC-32
* https://packages.msys2.org/group/mingw-w64-ucrt-x86_64-toolchain 下载对应包的File，解压tar.zst。只下gcc的还不够，也许下gcc的Dependencies就行了
* https://gitee.com/qabeowjbtkwb/x86_64-w64-mingw32-gcc-native-toolchain 也有Linux下运行的编译到Win的
* https://musl.cc/
* https://www.ed-x.cc/manual.html 优化了某些工具的性能
* __MINGW64_VERSION_STR定义了它自己的版本
* 线程模式：posix提供std::thread std::mutex，依赖libwinpthreads
* Linux下运行编译到Win的：gcc-mingw-w64-x86-64-win32，Ubuntu需要2204，Debian要bullseye(11)，命令行为x86_64-w64-mingw32-gcc

### [TCC](https://download.savannah.gnu.org/releases/tinycc/)

* -run 直接运行，支持-从stdin中读取
* -b 进行边界检查，隐含-g
* 支持C99
* 不支持-O

### MSVC

* #pragma comment(lib, "emapi") 相当于-lemapi

### 其他

* https://github.com/swig/cccl 把Unix编译器参数转换为cl的参数（cl的wrapper）
* https://github.com/rui314/chibicc 小型C编译器
* ICC/ICX：https://www.intel.cn/content/www/cn/zh/developer/articles/tool/oneapi-standalone-components.html#dpcpp-cpp 大小超过1G
* https://github.com/jart/cosmopolitan 编译出可以在Linux和Win上运行的程序

## gdb

* gdb a.out [coredump]。-q不显示startup-text，--args 命令行参数（也可以run时传），-ex一进入就执行的命令
* run(r)、continue(c)
* breakpoint(br) 'src.c'::line/func if(...)、delete、clear
* step(s)、s func、next(n)
* print(p) 'src.c'::arr 支持字符串、数组
  * /x以十六进制显示值
  * 打印指针对应的数组：p *arrp@n，先解引用，再显示那个地址往后n个元素
  * 打印指针变量的值及其指向的值，或打印地址里的值：x p/add。有选项修改打印的数量和格式
* list(l) line/func 显示源码，无参默认为当前函数的
* info(i) registers(r)/functions等
* quit(q)
* backtrace(bt)
* set var=v
* !xxx 执行shell命令
* 在VSC的Cpptools的DebugConsole中，要在前面加-exec或反引号才能调用

## 参考

* https://blog.csdn.net/haoel/article/details/2886

### TODO

* https://zhuanlan.zhihu.com/p/78091632
* https://zhuanlan.zhihu.com/p/100964932
* https://www.ruanyifeng.com/blog/2015/02/make.html
* https://github.com/seisman/how-to-write-makefile
* https://nullprogram.com/blog/2017/08/20/
* https://reactos.org/wiki/Building_MINGW-w64
* -Wno-excessive-errors
* https://zhuanlan.zhihu.com/p/296191493
* https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/
* https://github.com/rui314/mold
* https://zhuanlan.zhihu.com/p/163287897 九图记住Makefile
* https://github.com/hellogcc/100-gcc-tips/blob/master/src/index.md
* https://github.com/rr-debugger/rr
* https://makefiletutorial.com/
* scan-build
* crosstool-NG
* https://github.com/jrfonseca/drmingw JIT-debugger
