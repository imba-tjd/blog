---
title: Makefile和gcc
---

## Makefile

* 缩进只能用tab

```makefile
include a.mk # 引用其他的makefile，可以写绝对路径，可以使用通配符和变量

# 变量定义
objects = main.o kbd.o command.o display.o \ # 用反斜杠换行，教程错了= =
    insert.o search.o files.o utils.o
objects2 := $(wildcard *.o) # 使用wildcard关键字会进行扩展；如果直接用*.o，那就是普通的*.o，（效果应该是在command中当作shell命令会生效，但makefile自动推导无效）
obj = $(patsubst %.c ,%.o ,$(src)) # 用于从src目录中找到所有.c 结尾的文件，并将其替换为.o文件，并赋值给obj

    @echo 正在编译 # 单独的命令最前面必须使用tab
    cd /etc; pwd # 不同命令之间独立，如果不用分号而是换行，pwd不会显示cd的结果
    cd subdir && make # 进入子文件夹make；父makefile定义的变量手动用export命令可以传递到子makefile中，但SHELL和MAKEFLAGS变量会自动传递

# 显式规则
edit : $(objects) # 第一条为最终目标；使用变量用$()
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

1. gcc -E 进行预处理，一般以.i为后缀，如果不加-o会直接输出到终端里
2. gcc -S -masm=intel可生成intel风格的汇编，否则是AT&T风格的；加-fverbose-asm可生成与源代码对应的注释
3. gcc -c 生成二进制代码；-shared -fPIC生成动态库
4. ar -crv libname.a ... 生成静态库；-t显示包含哪些.o
5. `-Wl,-rpath=/...` 可以指定**运行时**要搜索的动态库目录
6. 与库相关的东西见《C语言笔记》第55点

### 其它

* gcc -o如果没有后缀，会自动加exe；touch也是这样
* 链接过程中，需要进行符号解析，并且是按照顺序解析；如果库链接在前，就可能出现库中的符号不会被需要，链接器不会把它加到未解析的符号集合中，那么后面引用这个符号的目标文件就不能解析该引用，导致最后链接失败。因此链接库的一般准则是将它们放在命令行的结尾
* ccache可以缓存编译信息
* 增强安全性的参数：https://gist.github.com/jrelo/f5c976fdc602688a0fd40288fde6d886 https://security.stackexchange.com/questions/24444

## 参考

* https://blog.csdn.net/haoel/article/details/2886
* 未读：https://zhuanlan.zhihu.com/p/78091632 https://www.zhihu.com/question/58949190
