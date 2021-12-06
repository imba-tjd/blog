---
title: Makefile和gcc
---

## Makefile

* 缩进只能用tab

```makefile
include a.mk # 引用其他的makefile，可以写绝对路径，可以使用通配符和变量

# 变量定义，可以在命令行中用key="val"重写
objects = main.o kbd.o command.o display.o \ # 用反斜杠换行，教程错了= =
    insert.o search.o files.o utils.o
objects2 := $(wildcard *.o) # 使用wildcard关键字会进行扩展；如果直接用*.o，那就是普通的*.o，（效果应该是在command中当作shell命令会生效，但makefile自动推导无效）
obj = $(patsubst %.c ,%.o ,$(src)) # 用于从src目录中找到所有.c 结尾的文件，并将其替换为.o文件，并赋值给obj

    @echo 正在编译 # 单独的命令最前面必须使用tab
    cd /etc; pwd # 不同命令之间独立，如果不用分号而是换行，pwd不会显示cd的结果
    cd subdir && make # 进入子文件夹make；父makefile定义的变量手动用export命令可以传递到子makefile中，但SHELL和MAKEFLAGS变量会自动传递

# 显式规则
edit : $(objects) # 如果没有指定目标就用第一个目标；使用变量用$()
    cc -o edit $(objects) # 此处的edit为程序名字

# 隐式规则，自动推导：.o会自动把.c加入依赖
main.o : defs.h
kbd.o : defs.h command.h
command.o : defs.h command.h
display.o : defs.h buffer.h
insert.o : defs.h buffer.h
search.o : defs.h buffer.h
files.o : defs.h buffer.h command.h
utils.o : defs.h

# 发现.h有很多重复的，另一种隐式推导：
$(objects) : defs.h
kbd.o command.o files.o : command.h
display.o insert.o search.o files.o : buffer.h

.PHONY : clean cleandiff # 伪目标，不是一个文件，只是一个标签；只有显式使用make clean才会生效
clean : cleandiff # 伪目标也可以有依赖
    -rm edit $(objects) # 减号表示出现错误也继续执行
cleandiff :
    rm *.diff

ifeq (, $(shell which curl))
    $(error "No curl in $$PATH, please install")
endif

EMPTY:=
SPACE:=$(EMPTY) $(EMPTY)
COMMA:=$(EMPTY),$(EMPTY)
```

### 文件搜寻

* 在一些大的工程中，有大量的源文件。通常的做法是把这许多的源文件分类，并存放在不同的目录中。所以，当make需要去找寻文件的依赖关系时，你可以在文件前加上路径。但最好的方法是把一个路径告诉make，让make在自动去找：VPATH = src:../headers（src和headers是目录名）
* 当然，当前目录永远是最高优先搜索的地方

还可以使用vpath关键字：

1. vpath pattern directories ：为符合模式的文件指定搜索目录。
2. vpath pattern：清除符合模式的文件的搜索目录。
3. vpath：清除所有已被设置好了的文件搜索目录。

pattern中%表示0或任意字符；路径可以用冒号分隔多个

### 一次生成多个文件

```
# 伪目标的所有依赖总会被执行
all : prog1 prog2 prog3
.PHONY : all

prog1 : prog1.o utils.o
    cc -o prog1 prog1.o utils.o
prog2 : prog2.o
    cc -o prog2 prog2.o
prog3 : prog3.o sort.o utils.o
    cc -o prog3 prog3.o sort.o utils.o
```

### 多目标、静态模式

如果command相似，可以使用此功能，会进行一些扩展。见《[跟我一起写 Makefile（五）](https://blog.csdn.net/haoel/article/details/2890)》

### 其它

* 自动生成依赖性：太复杂……

## gcc

* gcc -o如果没有后缀，会自动加exe；touch也是这样
* 链接过程中，需要进行符号解析，并且是按照顺序解析；如果库链接在前，就可能出现库中的符号不会被需要，链接器不会把它加到未解析的符号集合中，那么后面引用这个符号的目标文件就不能解析该引用，导致最后链接失败。因此链接库的一般准则是将它们放在命令行的结尾
* [ccache](https://github.com/ccache/ccache)可以缓存编译信息
* -Ofast可开启最高优化，包含O3和ffast-math等，但可能产生不符合标准的行为
* /bin/gcc-10、/bin/gcc、/bin/x86_64-linux-gnu-gcc
* -flto=thin可进行一些优化，编译和链接步骤都要用，除非手动用lld-link
* -march指定代码能运行的最小CPU，默认x86-64；设为native可能就等于skylake这样，就可能不能运行在其它机器上。-mtune默认generic，改成native可生成为本机优化的代码
* -g等于-g2，-g3的体积更大；-gsplit-dwarf能减少一些体积，把信息放到dwo文件中，能提升链接速度。但无法与-flto一起使用
* -###为dry-run，能显示具体编译用到的命令
* --help=xxx能显示更多选项帮助，在前面加-Q改为看是否启用

### 编译步骤

1. gcc -E 进行预处理，一般以.i为后缀，如果不加-o会直接输出到终端里
2. gcc -S -masm=intel可生成intel风格的汇编，否则是AT&T风格的；加-fverbose-asm可生成与源代码对应的注释
3. gcc -c 生成二进制代码(.o)
4. -shared -fPIC生成动态库(.so)，fpic产生的代码更小更快但在某些平台有限制一般不用；fPIE用于可执行文件；Win下的-mdll或--dll现在的效果与shared相同
5. ar -crv libname.a src.o ... 生成静态库；-t显示包含哪些.o

### 库

* Linux下，.a是静态库，由多个.o组成，编译时当作.o附加到参数中就是；.so是动态库，需要-L指定库存在的文件夹，当前目录可省，再-l库名，也支持静态库但优先用动态的，可在前面加-Wl,-Bstatic优先用静态的；一般库都以lib开头，-l:库名.a可精确指定名字
* Windows下，.lib是静态库，.dll是动态库；但与so相比，dll可不包含符号，此时符号在.lib中，即同一个后缀两个不同的作用
* MinGW的-lxxx的搜索顺序：libxxx.dll.a xxx.dll.a libxxx.a cygxxx.dll libxxx.dll xxx.dll
* ldd命令能查看程序所依赖的共享库
* `-Wl,-rpath=/...` 可以指定**运行时**要搜索的动态库目录
* 理论上MinGW可以直接链接.lib的，但32和64不能通用。lib转a可以见：https://stackoverflow.com/questions/11793370/how-can-i-convert-a-vsts-lib-to-a-mingw-a ，但我试了一下无效
* 增强安全性的参数：https://gist.github.com/jrelo/f5c976fdc602688a0fd40288fde6d886 https://security.stackexchange.com/questions/24444
* -ftrapv在linux下整数溢出时会触发core dump，会减慢速度
* 现在的编译器对未定义行为优化得太多了，但写底层代码时又时又无法避免。此时就要加-fno-strict-aliasing和-fwrapv

### 超级静态的编译

```
-static-libgcc -static-libstdc++ -Wl,-Bstatic,--whole-archive -lwinpthread -Wl,--no-whole-archive
```

### 临时禁用警告

```c
#pragma GCC diagnostic push  // 如果在整个本文件中忽略，也可不用它
#pragma GCC diagnostic ignored "-Wunused-value"
...
#pragma GCC diagnostic pop
```

## Clang

* 安装：https://apt.llvm.org/
* Win: https://github.com/mstorsjo/llvm-mingw 有ucrt，但支持太多的target导致可执行文件有点多

## MinGW

* https://github.com/brechtsanders/winlibs_mingw/releases 下x86_64-posix-seh-*.7z 没有pretty-printer
* http://www.equation.com/servlet/equation.cmd?fa=fortran 线程模式为win32。安装必须用它的程序，可以自己解压但不能直接复制，env文件控制自动添加PATH
* https://gcc-mcf.lhmouse.com/ 小文件太多；有ucrt
* https://github.com/Guyutongxue/mingw-release
* https://jmeubank.github.io/tdm-gcc/ 自动添加系统级别的PATH，目前最新10.3
* https://nuwen.net/mingw.html 目前最新11.2
* __MINGW64_VERSION_MAJOR定义了它自己的版本
* https://osdn.net/projects/mingw/releases/ MinGW32，官网都挂了

## 参考

* https://blog.csdn.net/haoel/article/details/2886

### TODO

* https://zhuanlan.zhihu.com/p/78091632
* CMake：https://www.zhihu.com/question/58949190
* https://zhuanlan.zhihu.com/p/100964932
* https://www.ruanyifeng.com/blog/2015/02/make.html
* https://github.com/seisman/how-to-write-makefile
* https://nullprogram.com/blog/2017/08/20/
* https://reactos.org/wiki/Building_MINGW-w64
* -Wno-excessive-errors
* https://zhuanlan.zhihu.com/p/296191493
* https://developers.redhat.com/blog/2018/03/21/compiler-and-linker-flags-gcc/
* CMake: https://zhuanlan.zhihu.com/p/361123818
