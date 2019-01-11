## Makefile的规则

```makefile
target...: prerequisites...
	command
...
...
```

或

```makefile
targets: prerequisites; command
command
...
```

这是一个文件的依赖关系，也就是说target这个一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。通俗点就是如果prerequisites中有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。

值得注意是，`command`如果不与`target: prerequisites`在一行，那么必须以tab键开头。如果在一行，那么可以用分号作为分隔。

### make是如何工作的

```makefile
#MakeFile
objects = main.o tool.o

main:$(objects)
	cc -o main $(objects)
main.o:main.c tool.h
	cc -c main.c
tool.o:tool.c tool.h
	cc -c tool.c
   
.PHONY:clean
clean:
	rm main *.o
```

跟据上述的makefile文件，在默认情况下

1. make会在当前目录下找到名字为"Makefile"或者"makefile"文件；
2. 如果找到，它会找到文件中的第一个目标文件main，并将其作为最终的目标文件.
3. 如果main不存在，或者main所依赖的后面.o文件的修改时间要比main这个文件要新，那么它就会执行后面所定义的命令来生成main这个文件；
4. 如果main所依赖的.o文件存在，那么make会在当前文件中找目标为.o文件的依赖性，如果找到再根据那一规则生成.o文件；
5. 最后会将.o文件生成最后的文件main。

在寻找的过程中，如果出现错误，比如最后被依赖的文件找不到，那么make就会直接退出，并报错。

### make的自动推导

GNU的make可以自动推到文件以及文件依赖关系后面的命令。

所以上面的makefile可以简化为

```makefile
objects = main.o tool.o
main: $(objects)
	cc -o main $(objects)
main.o: tool.h
tool.o: tool.h 

.PHONY:clean
clean:
	rm main $(objects)
```

这种方式为make的“隐晦规则”. 其中“.PHONY”表示clean是个伪目标文件。



## makefile总述

1. 显式规则

   最具体的写法

2. 隐晦规则

   make的自动推导功能

3. 变量的定义

   类似于C语言的宏，变量在执行时扩展到相应的引用位置

4. 文件指示

   1. 一个makefile引用另一个makefile;
   2. 根据某些情况指定makefile中的有效部分；
   3. 定义多行命令.

5. 注释

   注释用"#"字符。

   ​	

### makefile的文件名

默认情况下，make命令会在当前目录下寻找以"Makefile"或"makefile"命名的文件。

如果是使用重命名了make文件为"filename"则使用:

```makefile
make -f filename
```

引用其它的Makefile

可以通过`include` 关键字把别的Makefile包含进来。

### 环境变量Makefiles

如果当前环境定义了环境变量makefiles，那么make会把这个变量中的值做一个类似于`include` 的动作。

### make的工作方式

GNU的make工作的执行步骤：

1. 读入所有的makefile;
2. 读入被include的其它Makefile；
3. 初始化文件中的变量；
4. 推导隐晦规则，并分析所有规则；
5. 为所有的目标文件创建依赖关系链；
6. 根据依赖关系，决定哪些目标要重新生成；
7. 执行生成命令。



### 通配符

make支持三种通配符： `*`、`?`、`[...]`

通配符代替了你一系列的文件，如`*.c`表示所有后缀为c的文件。



### 文件搜寻

方法1：

Makefile文件中的特殊变量`VPATH`来完成，如

`VPATH = src:../headers` make会按照"src"和"../headers"这两个目录去搜索。（目录由冒号分隔）

方法2：

使用make的`vpath`关键字（注意是全小写）

1.  vpath <pattern> <directories>

    对符合<pattern>的文件指定搜索目录<directories>

2. vpath <pattern>

    清除符合<pattern>的文件指定搜索目录

3. vpath

   清除所有已被设置好了的文件搜索目录



```	
|-- Makefile
|-- include
|   `-- hello.h
`-- src
    |-- hello.cpp
    ` -- main.cpp
```

首先是Makefile 文件和include文件夹还有src文件夹在同一个目录下，头文件hello.h在include目录下，源文件main.cpp和hello.cpp在src目录下

在执行make命令的时候，根据makefile执行步骤，首先读入所有的makefile文件，那么

`VPATH = include：src`       //指定了makefile的搜索路径

或

`vpath %.h include`    //指定.h类型文件的搜索路径是include。

`vpath %.cpp src`      //指定.cpp类型文件的搜索路径是src。



### 伪目标

因为我们不能生成"clean"这个文件，所以我们会指定一个标签来对应“伪目标”。当然“伪目标”的取名不能和文件名重名。所以为了避免和文件重名的这种情况，我们可以使用一个特殊的标记`.PHONY`来显示地指明一个目标是伪目标，向make说明，不管是否有这个文件，这个目标就是"伪目标"。

```makefile
.PHONY: clean
clean:
	rm *.o temp
```

伪目标也可以成为依赖，如下

```makefile
.PHONY: cleanall cleanobj cleandiff
cleanall: cleanobj cleandiff
	rm program
cleanobj:
	rm *.o
cleandiff:
	rm *.diff
```

我们可以通过输入不同的命令来达到清除不同种类文件的目的。



### 多目标

使用自动化变量`$@`  表示目标集

还有`$<` 表示依赖目标集



### 静态模式

静态模式可以更加容易地定义多目标的规则，可以让我们的规则变得更加的有弹性和灵活。

```makefile
objects =foo.o bar.o

all:$(objects)

$(objects):%.o :%.c
	$(CC) -c $(CFLAGS) $< -o $@
```

如果我们有几百个"%.o" ，那么使用这种“静态模式”就可以简单地写完一堆规则。

### 自动生成依赖

当`main.c`文件中有`include "defs.h"`时，那么makefile的依赖关系会是：`main.o: main.c defs.h`。

但是如果一个大型的工程为了避免这种繁重而容易出错的事情，我们可以使用C/C++编译的一个功能。即自动寻找源文件包含的头文件：`-M`。

所以，当我们执行`cc -M main.c ` 其输出就是`main.o: main.c defs.h`。GNU的C/C++需要使用`-MM`参数，不然会把标准库的头文件也包含进来。









