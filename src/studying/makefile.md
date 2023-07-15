# Makefile

## gcc

`main.c`

```c
#include<stdio.h>

int main()
{
    printf("Hello world\n");
    return 0;
}
```

### 1.预处理

```shell
gcc -E main.c -o main.i
```

* `-E`：预处理；
* `-o` : 将结果输出到指定文件中；

### 2. 编译，生成汇编语言

```shell
gcc -S mian.c -o main.s
```

* `-S`：进行预处理和汇编操作；

### 3. 生成目标文件

```shell
gcc -c mian.c -o mian.o
```

- `-c`：生成目标文件

### 4.生成可执行文件

```shell
gcc mian.c -o main
```

## 静态文件库

三个.c文件：`main.c` , `add.c`, `minus.c`

**目标**：将`add.c`, `minus.c`编译为静态文件库

`mian.c`

```c
#include<stdio.h>

int add(int a, int b);
int minus(int a, int b);

int main()
{
    printf("Hello world\n");
    printf("add  = %d\n", add(7, 5));
    printf("minus= %d\n", minus(7, 5));
    return 0;
}
```

`add.c`

```c
int add(int a, int b)
{
    return a + b;
}
```

`minus.c`

```c
int minus(int a, int b)
{
    return a - b;
}
```

```shell
# 1.编译成.o文件
gcc -c [.c] [.c] ...
# example
gcc -c add.c  minus.c
# 2. 编译静态库
ar -r [lib自定义库名.a] [.o] [.o] ...
# example
ar -r  liboperation.a add.o minus.o
# 3. 链接为可执行文件
gcc [.c] [.a] -o [自定义输出文件名]
or
gcc [.c] -o [自定义输出文件名] -l[库名] -L[库路径]
# example
gcc main.c liboperation.a -o main
```

总结：

1. 静态库以`.a` 为后缀；
2. 静态库名字以`lib`作为前缀；

## 动态文件库

```shell
# 1. 编译为.o文件
gcc -c -fpic [.c] [.c]...
# example
gcc -c -fpic add.c minus.c
# 编译为静态库
gcc -shared [.o] [.o]... -o [lib自定义库名.so]
# example
gcc -shared add.o minus.o -o liboperation.so
# 链接
gcc [.c/.cpp] -o [自定义可执行文件名]  [动态库路径]
or
gcc [.c] -o [自定义输出文件名] -l[库名] -L[库路径] -WL,rpath=[库路径]
# example
gcc main.c -o main ./liboperation.so 

```

* `-fpic`:生成与位置无关的代码（Position Independent Code）
* 动态文件库后缀为`.so`

## Makefile

基本格式

```shell
targets : prerequisties
[tab键]command
```

* targets:目标文件
* prerequisties：依赖文件
* command：执行命令

### 变量

* `$@`: 目标(target)的完整名称
* `$<`: 第一个依赖文件（prerequisties）的名称
* `$^`: 所有的依赖文件（prerequisties），以空格分开，不包含重复的依赖文件

```makefile
#定义变量
obj := main.o
cfile := main.c

${obj}: ${cfile}
	gcc -c $^ -o $@

debug:
	@echo ${obj}
	@echo ${cfile}
clear:
	rm *.o
.PHONY: clear
```

* `.PHONY`伪目标
* `@`可以隐藏命令，终端不输出

### 常用符号

### =

- 简单的赋值运算符
- 用于将右边的值分配给左边的变量
- 如果在后面的语句中重新定义了该变量，则将使用新的值

#### :=

- 立即赋值运算符
- 用于在定义变量时立即求值
- 该值在定义后不再更改
- 即使在后面的语句中重新定义了该变量

#### ?=

- 默认赋值运算符
- 如果该变量已经定义，则不进行任何操作
- 如果该变量尚未定义，则求值并分配

#### +=

* 累加

#### *和% 

- `*`: 通配符表示匹配任意字符串，可以用在目录名或文件名中
- `%`: 通配符表示匹配任意字符串，并将匹配到的字符串作为变量使用

### 常用函数

```makefile
$(fn arguments) or ${fn arguments}
```

* fn: 函数名
* arguments: 函数参数，参数间以逗号 `,` 分隔，而函数名和参数之间以“空格”分隔

 #### 1. shell: 调用shell命令

```makefile
$(shell <command> <arguments>)
```

- 功能：调用 shell 命令 command
- 返回：函数返回 shell 命令 command 的执行结果

```makefile
#获取src下的所有.c文件
c_src := $(shell find  src -name "*.c")
debug:
	@echo $(c_src)

.PHONY:debug
```

#### 2 subst：字符串替换

```makefile
$(subst <from>,<to>,<text>)
```

- 功能：把字串 <text> 中的 <from> 字符串替换成 <to>
- 返回：函数返回被替换过后的字符串

```makefile
#获取src下的所有.c文件
c_src := $(shell find  src -name "*.c")
# 将src文件夹更换为obj文件夹
c_obj := $(subst src, obj, $(c_src))
debug:
	@echo $(c_src)
	@echo $(c_obj)

.PHONY:debug
```

#### 3. patsubst: 字符串替换

- 功能：通配符 `%`，表示任意长度的字串，从 text 中取出 patttern， 替换成 replacement
- 返回：函数返回被替换过后的字符串

```makefile
#获取src下的所有.c文件
c_src := $(shell find  src -name "*.c")
# 将src文件夹更换为obj文件夹并将后缀替换为.o
c_obj := $(patsubst src%.c, obj%.o, $(c_src))
debug:
	@echo $(c_src)
	@echo $(c_obj)

.PHONY:debug
```

#### 4. foreach:循环函数

```makefile
$(foreach <var>,<list>,<text>)
```

- 功能：把字串<list>中的元素逐一取出来，执行<text>包含的表达式
- 返回：<text>所返回的每个字符串所组成的整个字符串（以空格分隔）

```makefile
# .头文件的地址
library_paths := ./inc \
				/usr/local/include
# 将头文件前面加上-L
library_paths := $(foreach iterm, $(library_paths), -L$(library_paths))
debug:
	@echo $(library_paths)

.PHONY:debug
```

#### 5 dir：返回子目录

```makefile
$(dir <names...>)
```

* 功能：从文件名序列中取出目录部分。目录部分是指最后一个反斜杠（“/”）之前 的部分。如果没有反斜杠，那么返回“./”。
* 返回：返回文件名序列的目录部分。

```makefile
#获取src下的所有.c文件
c_src := $(shell find  src -name "*.c")
# 获取src文件夹路径
src_path := $(dir $(c_src))
debug:
	@echo $(src_path)

.PHONY:debug
```

#### 6 notdir : 去掉文件绝对路径，保留文件名

```makefile
$(notdir <names...>)
```

#### 7. filter:过滤

```makefile
$(filter <names...>)
```

#### 8 basename:去掉后缀

```makefile
$(basename <names...>)
```

```makefile
# 获取静态库和动态库名称

#获取/usr/lib 下以lib为前缀的路径
lib_path := $(shell find  /usr/lib -name "lib*")
# 去掉文件路径，获取文件名
lib_name := $(notdir $(lib_path))
# 过滤出静态库和动态库
a_libs := $(filter %.a, $(lib_name))
so_libs := $(filter %.so, $(lib_name))
# 获取库文件名称
a_libs := $(basename $(a_libs))
so_libs := $(basename $(a_libs))
# 去掉前缀lib
a_libs := $(subst lib, , $(a_libs)) 
so_libs := $(subst lib, , $(so_libs)) 
debug:
	@echo $(so_libs)

.PHONY:debug
```

### 编译选项

- `-std=`: 指定编译标准，例如：-std=c++11、-std=c++14
- `-g`: 包含调试信息
- `-w`: 不显示警告
- `-O`: 优化等级，通常使用：-O3
- `-I`: 加在头文件路径前
- `fPIC`: (Position-Independent Code), 产生的没有绝对地址，全部使用相对地址，代码可以被加载到内存的任意位置，且可以正确的执行。这正是共享库所要求的，共享库被加载时，在内存的位置不是固定的

### 连接选项

- `-l`: 加在库名前面
- `-L`: 加在库路径前面
- `-Wl,<选项>`: 将逗号分隔的 <选项> 传递给链接器
- `-rpath=`: "运行" 的时候，去找的目录。运行的时候，要找 .so 文件，会从这个选项里指定的地方去找

## example

文件夹结构

```she
.
├── inc
│   ├── add.h
│   └── minus.h
├── Makefile
└── src
    ├── add.c
    ├── main.c
    └── minus.c
```

`add.h`

```c
#ifndef ADD_H
#define ADD_H

int add(int a, int b);

#endif 

```

`minus.h`

```c
#ifndef MINUX_H
#define MINUX_H

int minus(int a, int b);

#endif 
```

`add.c`

```c
# include"add.h"
int add(int a, int b)
{
    return a + b;
}
```

`minus.c`

```c
# include"minus.h"
int minus(int a, int b)
{
    return a - b;
}
```

`main.c`

```c
#include<stdio.h>
#include "add.h"
#include "minus.h"

int main()
{
    printf("Hello world\n");
    printf("add  = %d\n", add(7, 5));
    printf("minus= %d\n", minus(7, 5));
    return 0;
}
```

将add和minus编译成静态库，然后链接成可执行文件

`Makefile`

```makefile
# 库源文件
lib_srcs := $(filter-out src/main.c, $(shell find src -name "*.c"))
lib_objs := $(patsubst src%.c, objs%.o, $(lib_srcs))
# 头文件
include_path := ./inc
# 编译选项
I_flag := $(include_path:%=-I%)
compile_flag := -g -O3 $(I_flag)
# 链接选项
library_path := ./lib
linking_lib := XXX
l_flag := $(linking_lib:%=-l%)
L_flag := $(library_path:%=-L%)

linking_flag := $(l_flag) $(L_flag)

# 加上编译选项
objs/%.o: src/%.c
	@mkdir -p $(dir $@)
	gcc -c $^ -o $@ $(compile_flag)

# 编译静态库
lib/libXXX.a: $(lib_objs)
	@mkdir -p $(dir $@)
	ar -r $@ $^

static_lib: lib/libXXX.a

# 链接静态库
objs/main.o: src/main.c
	@mkdir -p $(dir $@)
	gcc -c $< -o $@ $(compile_flag)

# 加上链接选项
workspace/exec: objs/main.o
	@mkdir -p $(dir $@)
	gcc  $^ -o $@ $(linking_flag)

run: workspace/exec
	@./$<

debug:
	@echo $(lib_srcs)
	@echo $(compile_flag)

clear:
	rm -rf objs workspace lib

.PHONY: clear debug run
```

编译成动态库，然后链接为可执行文件

`Makefile`

```makefile
# 库源文件
lib_srcs := $(filter-out src/main.c, $(shell find src -name "*.c"))
lib_objs := $(patsubst src%.c, objs%.o, $(lib_srcs))
# 头文件
include_path := ./inc
# 编译选项
I_flag := $(include_path:%=-I%)
compile_flag := -g -O3 -fpic $(I_flag)
# 链接选项
library_path := ./lib
linking_lib := XXX
l_flag := $(linking_lib:%=-l%)
L_flag := $(library_path:%=-L%)

# 加上链接选项-Wl,-rpath=[库路径]
linking_flag := $(l_flag) $(L_flag)  -Wl,-rpath=$(library_path)

# 加上编译选项
objs/%.o: src/%.c
	@mkdir -p $(dir $@)
	gcc -c $^ -o $@ $(compile_flag)

# 编译动态库： gcc -shared
lib/libXXX.so: $(lib_objs)
	@mkdir -p $(dir $@)
	gcc -shared  $^ -o $@

share_lib: lib/libXXX.so

# 链接动态库
objs/main.o: src/main.c
	@mkdir -p $(dir $@)
	gcc -c $< -o $@ $(compile_flag)

# 加上链接选项
workspace/exec: objs/main.o
	@mkdir -p $(dir $@)
	gcc  $^ -o $@ $(linking_flag)

run: workspace/exec
	@./$<

debug:
	@echo $(lib_srcs)
	@echo $(r_flag)

clear:
	rm -rf objs workspace lib

.PHONY: clear debug run
```

