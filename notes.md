## Makefile的规则

```makefile
target...: prerequisites...
	command
...
...
```

​	这是一个文件的依赖关系，也就是说target这个一个或多个的目标文件依赖于prerequisites中的文件，其生成规则定义在command中。通俗点就是如果prerequisites中有一个以上的文件比target文件要新的话，command所定义的命令就会被执行。

值得注意是，命令都是以tab键开始。

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
