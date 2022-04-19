### 简介
Makefile是在Linux环境下 C/C++ 程序开发必须要掌握的一个工程管理文件。当你使用make命令去编译一个工程项目时，make工具会首先到这个项目的根目录下去寻找Makefile文件，然后才能根据这个文件去编译程序。

GCC编译器，在程序的安装目录下面会有各种二进制可执行文件：

   - cpp：预处理器
   - ccl：编译器
   - as：汇编器
   - ld：链接器
   - ar：静态库制作工具

### Makefile规则
规则通常由目标、目标依赖和命令三部分构成：
```
目标：目标依赖
    命令
a.out: hello.c
    gcc -o a.out hello.c
```

- 默认目标

  一个Makefile文件里通常会有多个目标，一般会选择第一个作为默认目标。
- 伪目标

  设置一个目标，并不是真正生成这个文件，如clean目标，仅仅为了执行某个操作。
```
.PHONY: clean
clean:
    rm -f a.out hello.o
```

### 目标依赖

  自动生成hello.c文件的头文件依赖关系
```
gcc -M hello.c
```

### Makefile命令
命令一般由shell命令（echo、ls）和编译器的一些工具（gcc、ld、ar、objcopy等）组成，使用tab键缩进。每一个命令，make会开一个进程执行，每条命令执行完，make会监测每个命令的返回码。若命令返回成功，make继续执行下一个命令；若命令执行出错，make会终止执行当前的规则，退出编译流程。
- 在make编译的时候打印正在执行的执行，可以在每条命令的前面加一个@可以不打印命令
```
echo "start compiling..."
@gcc -o a.out hello.c
```

### Makefile变量
Makefile中定义一个变量val，使用使用 $(val) 或 ${val} 的形式去引用它。当项目中需要添加更多的源文件时，你只需要更改OBJS的值就可以，便于修改和维护。
- 条件赋值
```
条件赋值：?=
```
条件赋值是指一个变量如果没有被定义过，就直接给它赋值；如果之前被定义过，那么这条赋值语句就什么都不做。
```
CC = gcc
CC ?= arm-linux-gnueabi-gcc
```

- 追加赋值
```
追加赋值：+=
```
追加赋值是指一个变量，以前已经被赋值，现在想给它增加新的值，此时可以使用+=追加赋值。
```
OBJS = hello.c
OBJS += module.c
```

- 立即变量

  立即变量使用 := 操作符进行赋值，在解析阶段就直接展开。
```
a = 1
val_a := $(a)
a = 10
test:
    echo $(val_a)
```
结果： echo 1
立即展开变量一般用在规则中的目标、目标依赖中,make在解析Makefile阶段，需要这些变量有确切的值来构建依赖关系树。

- 延迟变量

  延迟变量则是使用 = 操作符进行赋值，在make解析Makefile阶段不会立即展开，而是等到实际使用这个变量时才展开。
```
b = 2
val_b  = $(b)
b = 20
test:
    echo $(val_b)
```
结果： echo 20
延迟展开变量一般用在规则的命令行中，这些变量在make编译过程中被引用到才会展开，获得其实际的值。

- 自动变量

  自动变量是局部变量，作用域范围在当前的规则内。
  - $@：目标
  - $^：所有目标依赖
  - $<：目标依赖列表中的第一个依赖
  - $?：所有目标依赖中被修改过的文件
  - $%：当规则的目标是一个静态库文件时，$%代表静态库的一个成员名
  - $+：类似$^，但是保留了依赖文件中重复出现的文件
  - $*：在模式匹配和静态模式规则中，代表目标模式中%的部分。比如hello.c，当匹配模式为%.c时，$*表示hello
  - $(@D)：表示目标文件的目录部分
  - $(@F)：表示目标文件的文件名部分
  - $(*D)：在模式匹配中，表示目标模式中%的目录部分
  - $(*F)：在模式匹配中，表示目标模式中%的文件名部分
  - -: ：告诉make在编译时忽略所有的错误
  - @: ：告诉make在执行命令前不要显示命令

### Makefile递归执行
在工程项目中，各个源文件通常存放在各个不同的目录中，make在编译工程项目时，会依次遍历各个不同的子目录，编译每个子目录下的源文件。
```
# make -C subdir1 subdir2 subdir3 ...
```
make会依次到subdir1、subdir2、subdir3子目录下去执行make程序。
```
# cd subdir1 && $(MAKE)
# cd subdir2 && $(MAKE)
# cd subdir3 && $(MAKE)
```
在subdir1、subdir2、subdir3子目录下，需要有对应的Makefile文件，否则make就会运行报错。每个Makefile文件内容如下：
```
#subdir1/makefile:
all:
    echo "make in subdir1, do some thing"
```
在工程的根目录下，Makefile内容如下：
```
.PHONY:all
all:
    @echo "make start"
    make -C subdir1
    make -C subdir2
    make -C subdir3
    @echo "make done"
```

#### Makefile递归传递变量
make依次遍历到各个子目录下解析新的Makefile时，需要项目顶层目录的主Makefile定义的一些变量。使用export可以将主Makefile变量传递给子Makefile。
```
# 主Makefile
export WEB = zhaixue.cc
all:
    @echo "make start"
    @echo "WEB = $(WEB)"
    make -C subdir1
    make -C subdir2
    make -C subdir3
    @echo "make done"
```

```
# 子Makefile
#subdir1/makefile:
all:
    @echo "make in subdir1"
    @echo "sundir1:WEB = $(WEB)"
```

### 条件判断
使用条件判断，可以让make在编译程序时，根据不同的情况，执行不同的分支。ifeq、ifneq、ifdef、ifndef 等关键字来进行条件判断。
- ifeq

  ifeq关键字用来判断两个参数是否相等。ifeq一般和变量结合使用。
```
mode = debug
hello: hello.c
ifeq ($(mode),debug)
    @echo "debug mode"
    gcc -g -o hello hello.c
else
  @echo "release mode"
  gcc -o hello hello.c
endif
```

- ifneq

  ifneq 关键字和ifeq关键字恰恰相反，用来判断参数是否不相等。

- ifdef

  ifdef 关键字用来判断一个变量是否已经定义。如果变量的值非空（在Makefile中，没有定义的变量的值为空），表达式为真。
```
mode =
hello: hello.c
ifdef mode
    @echo "debug mode"
    gcc -g -o hello hello.c
else
    @echo "release mode"
    gcc -o hello hello.c
endif
```

- ifndef

  ifndef关键字和ifdef相反，如果一个变量没有定义，表达式为真。

### Makefile函数
函数的使用和变量引用的展开方式相同：
```
$(function arguments)
${function arguments}
```
函数的使用格式，有以下需要注意:

  - 函数主要分为两类：make内嵌函数和用户自定义函数。对于 GNU make内嵌的函数，直接引用就可以了；对于用户自定义的函数，要通过make的call函数来间接调用。
  - 函数和参数列表之间要用空格隔开，多个参数之间使用逗号隔开。
  - 如果在参数中引用了变量，变量的引用建议和函数引用使用统一格式：要么是一对小括号，要么是一对大括号。

想要获取某个目录下所有的C文件列表，可以使用扩展通配符函数：wildcard
```
SRC  = $(wildcard *.c)
HEAD = $(wildcard *.h)
all:
    @echo "SRC = $(SRC)"
    @echo "HEAD = $(HEAD)"
```

用户自定义函数以define开头，endif结束，给函数传递的参数在函数中使用$(0)、$(1)引用，分别表示第1个参数、第2个参数…对于用户自定义函数，在Makefile中要使用call函数间接调用，各个参数之间使用空格隔开。
```

define func
    @echo "pram1 = $(0)"
    @echo "pram2 = $(1)"
endif
all:
    $(call func, hello zhaixue.cc)
```

### Makefile文本处理函数
文本处理函数：subst、patsubst、strip、findstring、filter、filer-out、sort、word、wordlist、words、fistword。
- subst

  subst函数用来实现字符串的替换，将字符串text中的old替换为new。
```
$(subst old,new,text)
```
- patsubst

  patsubst函数主要用来模式替换：使用通配符 % 代表一个单词中的若干字符，在PATTERN和REPLACEMENT如果都包含这个通配符，表示两者表示的是相同的若干个字符，并执行替换操作。
```
$(patsubst PATTERN, REPLACEMENT, TEXT)
```
- strip

  strip函数是一个去空格函数：一个字符串通常有多个单词，单词之间使用一个或多个空格进行分割，strip函数用来将多个连续的空字符合并成一个，并去掉字符串开头、末尾的空字符。空字符包括：空格、多个空格、tab等不可显示的字符。
```
STR =     hello a    b   c   
STRIP_STR = $(strip $(STR))
all:
    @echo "STR = $(STR)"
    @echo "STRIP_STR = $(STRIP_STR)"
```
- findstring

  findstring函数用来查找一个字符串。
```
$(findstring FIND, IN)
```
findstring函数会在字符串IN中查找“FIND”字符串，如果找到，则返回字符串FIND，否则，返回空。
- filter

  filter函数用来过滤掉一个指定的字符串，使用格式如下：
```
$(filter PATTERN…,TEXT)
```
filter函数用来过滤掉字符串TEXT中所有不符合PATTERN模式的单词，只留下符合PATTERN格式的单词。
- filter-out

  filer-out函数是一个反过滤函数，功能和filter函数相反：该函数会过滤掉所有符合PATTERN模式的单词，保留所有不符合此模式的单词。
- sort

  单词排序
```
$(sort LIST)
```
- word

  取单词,word函数的作用是从一个字符串TEXT中，按照指定的数目N取单词。
```
 $(word N,TEXT)
```
- wordlist

  wordlist函数用来从一个字符串TEXT中取出从N到M之间的一个单词串。
```
$(wordlist N, M, TEXT)
```
- words

  统计一个字符串TEXT中单词的个数。
```
$(words TEXT)
```
- firstword

  取一个字符串中的首个单词
```
$(firstword NAMES…)
```

### Makefile文件名处理

- dir

  取路径名的目录
```
$(dir NAMES…)
```

- notdir

  取文件名

- suffix

  取文件名后缀

- basename

  取文件名前缀

- addsuffix

  给文件名加后缀,给文件列表中的每个文件名添加后缀SUFFIX。
```
$(addsuffix SUFFIX, NAMES…)
```

- addprefix

  给文件名加前缀
```
$(addprefix PREFIX, NAMES…)
```

- wildcard

  列出所有符号匹配模式的文件,PATTREN可以使用shell能识别的通配符：？、*等。
```
$(wildcard PATTERN)
```

### foreach

  做一些循环或遍历操作。
```
$(foreach VAR,LIST,TEXT)
```
  Sample:
```
.PHONY: all
dirs = lcd usb media keyboard
srcs = $(foreach dir, $(dirs), $(wildcard $(dir)/*.c))
objs = $(foreach src, $(srcs), $(subst .c,.o,$(src)))
all:
    @echo "srcs = $(srcs)"
    @echo "objs = $(objs)"

Result：
srcs =  lcd/lcd.c  usb/usb.c  media/decode.c  keyboard/key.c
objs =  lcd/lcd.o  usb/usb.o  media/decode.o  keyboard/key.o
```

### if

  在一个函数上下文中实现条件判断的功能，类似于ifeq关键字。
```
$(if CONDITION,THEN-PART)
$(if CONDITION,THEN-PART[,ELSE-PART])
```
if 函数的第一个参数 CONDITION表示条件判断，展开后如果非空，则条件为真，执行 THEN-PART部分；否则，如果有ELSE-PART部分，则执行ELSE-PART部分。

### call

  调用用户自定义的函数，则只能使用call函数来间接调用。
```
PHONY: all
define func
    @echo "pram1 = $(0)"
    @echo "pram2 = $(1)"
endef
all:
    $(call func, hello zhaixue.cc)
```

### origin

  确定变量是从哪里来的。
```
$(origin <variable>)
```
如果变量没有定义，origin函数的返回值为：undefined，不同的返回值代表变量的类型不同。常见的返回值如下;
  - default：变量是一个默认的定义，比如 CC 这个变量
  - file：这个变量被定义在Makefile中
  - command line：这个变量是被命令行定义的
  - override：这个变量是被override指示符重新定义过的
  - automatic：一个命令运行中的自动化变量

### shell

  在Makefile中运行shell命令，可以使用 shell 函数来完成这个功能。
```
current_path = $(shell pwd)
```

### error and warning

  make提供了两个可以控制make运行方式的函数：error和warning。如果这两个函数在Makefile中使用，当make执行过程中检测到某些错误，就可以给用户提供一些信息，并且可以控制make的是否继续执行下去。
  - error

  ```
$(error TEXT…)
  ```
  Sample:
```
.PHONY: all
all:
    @echo "make command start..."
    $(error find a error)
    @echo "make command end..."
```
  注意的是：error函数是在函数被调用时才会提示信息并终止make的继续执行。如果函数出现在命令中，或者出现在一个递归变量的定义里，在读取Makefile时不会出现错误。而只有包含error函数引用的命令执行时，或者包含这个函数的定义变量被展开时，才会提示错误信息TEXT，并终止make的运行。

  - warning

  warning函数不会终止make的运行，make会继续运行下去。
```
$(warning TEXT…)
```
  Sample:
```
.PHONY: all
all:
    @echo "make command start..."
    $(warning find a error)
    @echo "make command end..."
```

### 通配符

  - 在Makefile中可以使用的通配符有：* 、? 、 […]，还有经常使用的几个自动变量也可以看做特殊通配符：

   - $@：所有目标文件
   - $^：目标依赖的所有文件
   - $<：第一个依赖文件
   - $?：所有更新过的依赖文件

  - 通配符主要用在两个场合：

  在规则的目标和依赖中：make在读取Makefile时会自动对其进行匹配处理（通配符展开）。
```
   test: *.o
       gcc -o $@ $^
   *.o: *.c
       gcc -c $^
```   
在规则的命令中：通配符的通配处理在shell执行命令时完成。
```
clean:
    rm -f *.o
```
