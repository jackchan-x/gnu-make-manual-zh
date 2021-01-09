# GNU Make Manual

[toc]

# 1. make 概述

*make* 可以自动确定一个大型软件中的哪些部分需要重新编译，并且能够自动执行相关命令去重新编译。本手册描述了的 *GNU make* 由 Richard Stallman 和 Roladn McGrath 开发。从 3.76 版后，由 Paul D. Smith 接管开发工作。

*GNU make* 符合 *IEEE Standard 1003.2-1992 (POSIX.2)* 第 6.2 节。

在本文例子，笔者使用了 C 语言程序，这是因为 C 语言程序最为流行，但读者也可以用 *make* 来完成任何编程语言的编译工作，前提是这种编程语言的编译器能在 *shell* 下运行。当然，*make* 的应用不局限于程序。如果某些文件必须随着其他一些文件的变化而更新，那么读者也可以使用 *make* 来完成这些任务。

在使用 *make* 前，读者需要写好一个名为 *makefile* 的文件，该文件需要描述清楚程序文件之间的关系，还要给出更新每个文件的命令。通常，一个程序的可执行文件由目标文件来更新，而这些目标文件则由编译源码产生。

一旦有了合适的 *makefile* 文件，在修改了一些源码文件后，一个简单的 *shell* 命令：

```bash
make
```

便足以完成所有必须的重编译工作。*make* 通过 *makefile* 数据库以及文件的最后修改时间来判断哪些文件需要更新。对于每个需要更新的文件，*make* 会用数据库中记录的命令来对其操作。

读者可以在执行 *make* 命令的时候添加命令行参数，来告诉 *make* 哪些文件需要重新编译以及如何编译。参见 [9. 如何运行 make](#9. 如何运行 make)。

## 1.1 如何阅读本手册

如果读者是个 *make* 新手，或者想阅读总体介绍，那么只需要阅读每章的前几节，后面几节可以略过。每章的前几节会有该章简介和总体信息，而后几节则会包含一些专业的或技术性的信息。[2. Makefiles 简介](#2. Makefiles 简介) 是个例外，整章都是介绍性的内容。

如果读者对其他 *make* 程序非常熟悉的话，请参阅 [14. GNU make 的特色功能](#14. GNU make 的特色功能)，该章列出了 *GNU make* 的增强功能。另外请参阅  [15. 不兼容性和缺失功能](#15. 不兼容性和缺失功能)，该章解释了一小部分其他 *make* 拥有而 *GNU make* 缺少的功能。

如果想阅读概要，请参阅 [9.7 选项概要](#9.7 选项概要)，[附录 A 快速索引](#附录 A 快速索引) 以及 [4.8 特殊内置目标](#4.8 特殊的内置目标)。

## 1.2 问题和漏洞

如果读者对 *GNU make* 有些疑问，或者认为发现了一些漏洞，那么请向开发者报告。我们不做任何承诺，但是我们会尽力去解决。

在报告漏洞之前，请读者确保漏洞的真实性。另外请仔细反复阅读文档，确保文档描述的方法在实际应用中有效。如果文档没有阐释清楚你能够做什么或者不能做什么，那么也请提交报告，这些都是文档的漏洞。

在提交漏洞报告或者自己解决漏洞之前，请将 *makefile* 文件简化至最小且能体现漏洞问题的程度，然后连同准确结果，包括任何出错和告警信息一起发给我们。请确保用以生成最简 *makefile* 文件的命令中不使用非自由软件或者不常用的工具（其实对于这种工具，读者随时都可以用简单的几句 *shell* 命令来测试）。最后，请详述 *makefile* 预期结果，以便于我们判断问题是否出在文档上。

如果读者发现了一个如假包换的错误，那么可以通过以下两种途径提交报告：发送电子邮件到：

​	bug-make@gnu.org

或者使用我们的在线项目管理工具，网址是：

​	http://savannah.gnu.org/projects/make/

除了上述的信息以外，请将使用的 *make* 版本号一并发给我们。读者可以通过使用命令 *make --version* 来获取版本信息。请确保报告中包含 *make* 所运行机器的硬件信息与操作系统信息。顺便提一句，获取这些信息的一种方法是使用命令 *make --help*，一切尽在命令的最后一行输出中。

# 2. Makefiles 简介

你需要编写一个名为 *makefile* 的文件来告诉 *make* 去做什么。最常见的情况是，让 *makefile* 告诉 *make* 如何进行编译和链接程序。

本章我们将会讨论一个简单的 *makefile* ，其描述了如何编译链接一个由 8 个 C 源码文件和 3 个头文件的文本编辑器。*GNU make* 有能力使能依赖（仅针对依赖）的二次展开，对于 *makefile* 里定义的某些或是所有目标。也可以通过*makefile* 显式要求 *make* 运行各种命令（例如，移除特定文件的清理动作）。想要查看更复杂的 *makefile* 例子，请参阅 [附录 C 复杂 Makefile](#附录 C 复杂 Makefile)。

*make* 重新编译编辑器的过程中，每个修改过的 C 源文件都会被重新编译。如果头文件修改了，每个包含该头文件的 C 源文件都会被重新编译。每个编译产生一个与源文件对应的目标文件。最终，如果源文件被重新编译，那么所有的目标文件，无论是新生成的还是之前编译生成的，都要被重新链接到一起产生一个新的可执行编辑器。

## 2.1 规则是什么样的

一个简单的 *makefile* 包含类似下面这样的规则：

```makefile
target ... : prerequisites ...
		recipe
		...
		...
```

**target** 通常是程序生成的文件的名字，本章例子的目标是可执行文件和目标文件。**target** 也可以是要执行的动作的名字，例如 *clean* （请参阅 [4.5 伪目标](4.5 伪目标)）.

**prerequisites** 是创建 **target** 所需要输入的文件。**target** 通常会依赖多个文件。

**recipe** 是 *make* 执行的命令。**recipe** 可能会包含多条命令，可以在同一行，也可以每个命令单独一行。请注意：每个 **recipe** 命令行都要以 *tab* 字符开头！这里特别需要注意。如果你想用其他字符开头，可以将 *.RECIPEPREFIX* 变量设置为你想要的字符（请参阅 [6.14 特殊变量](#6.14 特殊变量)）。

通常，**recipe** 位于带有依赖的规则中，它的作用是当任何依赖变化时创建目标文件。不过，依赖对规则来说不是必须的。例如，与目标 *clean* 相关的规则，一般会包含删除命令，它就没有依赖。

规则解释了如何以及何时产生特定的文件，这些文件是特定规则的目标。*make* 会根据依赖执行命令，来创建或更新目标。规则也可以说明如何以及何时执行动作。请参阅 [4. 编写规则](#4. 编写规则)。

makefile 可以包含规则意外的文本，但是一个简洁的 *makefile* 仅需要包含规则。也许规则看起来比样例要复杂一点，但所有规则多多少少都符合这种样式。

## 2.2 一个简单的 Makefile

下面直接上 *makefile* ，它描述了一个名为 *edit* 的可执行文件如何依赖 8 个目标文件，而这些目标文件又依赖 8 个C 源码文件和 3 个头文件。

本例中，所有 C 源文件都包含 *defs.h* ，但只有那些定义了编辑命令的源文件包含 *command.h*。同时，只有改变编辑器 *buffer* 的底层文件包含 *buffer.h*。

```makefile
edit : main.o kbd.o command.o display.o \
	   insert.o search.o files.o utils.o
		cc -o edit main.o kbd.o command.o display.o \
				insert.o search.o files.o utils.o
				   
main.o : main.c defs.h
		cc -c main.c
kbd.o : kbd.c defs.h command.h
		cc -c kbd.c
command.o : command.c defs.h command.h
		cc -c command.c
display.o : display.c defs.h buffer.h
		cc -c display.c
insert.o : insert.c defs.h buffer.h
		cc -c insert.c
search.o : search.c defs.h buffer.h
		cc -c search.c
files.o : files.c defs.h buffer.h command.h
		cc -c files.c
utils.o : utils.c defs.h
		cc -c utils.c
clean :
		rm edit main.o kbd.o command.o display.o \
			insert.o search.o files.o utils.o
```

通过使用反斜杠 '\\' 将较长的单行分为两行，这和使用较长的单行效果一样，但是更容易阅读。请参阅 [3.11 分割长行](#3.11 分割长行)。

要通过 *makefile* 创建可执行文件 *edit*，可以执行：

```bash
make
```

要通过 *makefile* 删除可执行文件和当前目录的所有目标文件，可以执行：

```bash
make clean
```

例子中，目标（target）包括可执行文件 *edit* 和 *object* 文件 *main.o* 和 *kbd.o* 。*main.c* 和 *defs.h* 是依赖。实际上，每个 *.o* 文件既是目标（target），同时也是依赖。命令为 *cc -c main.c* 和 *cc -c kbd.c*。

当目标（target）是一个文件的，任意一个依赖发生变化，目标都需要重编译或者重链接。此外，自动生成的依赖应当被首先更新。本例中，*edit* 依赖八个 *object* 文件。*main.o* 依赖源文件 *main.c* 和头文件 *defs.h*。

命令应当跟在每个包含目标（target）和依赖的行之后。这些命令阐释了如何更新目标（target）文件。命令行的行首必须是 *tab* 字符（或者是通过 *.RECIPEPREFIX* 变量指定的字符。请参阅 [6.14 特殊变量](#6.14 特殊变量)），以此来区分出 *makefile* 中的命令行。（请记住 *make* 不清楚命令是如何工作的。能否恰当的更新目标（target）文件取决于你提供的命令。目标（target）文件需要更新是， *make* 做的所有事情就是执行你指定的命令。）

*clean* 不是一个文件，仅仅是一个动作的名字。正常情况下，你并不想执行 *clean* 这个动作，所以 *clean* 没有被写入其他任何规则的依赖。因此，*make* 永远不会运行 *clean*，除非你指定。注意，*clean* 不仅不是其他规则的依赖，也不包含依赖，其仅有的目的就行运行特定的命令。只运行特定命令而不涉及文件的目标称为伪目标。想要了解伪目标，请参阅 [4.5 伪目标](#4.5 伪目标) 。想要了解如何让 *make* 忽略 *rm* 或者其他命令的错误，请参阅 [5.5 命令中的错误](#[5.5 命令中的错误) 。

## 2.3 make 是如何处理 Makefile

# 3. 编写 Makefile

## 3.7 make 如何读取 Makefile

## 3.9 二次展开

通过前面的内容，我们知道 *GNU make* 工作在两种不同的状态：读取状态和目标更新状态（请参阅 [3.7 make 如何读取 Makefile](#3.7 make 如何读取 Makefile)）。

## 3.11 分割长行

# 4. 编写规则

## 4.5 伪目标

伪目标不是一个真实的文件名。当用 *make* 显式指定请求时，伪目标仅仅是将要执行的命令的名字。有两个理由去使用伪目标：一是避免和相同的文件名产生冲突；二是提升性能。

如果你写了一条规则，而该规则的命令不会实际创建目标文件，那么 *make* 每次到达该目标时，其命令总会被执行。例如：

```makefile
clean:
		rm *.o temp
```

因为 *rm* 命令不会创建名为 *clean* 的文件，也许永远也不存在这样的文件。因此，每次执行 *make clean* 的时候，*rm* 命令都会被执行。

上面的例子中，如果 *Makefile* 同目录下存在名为 *clean* 的文件，*clean* 目标的执行将会出现问题。因为没有依赖文件，*clean* 总是被认为是最新的，所以 *clean* 的命令永远不会被执行。为了避免这种问题，可以显式的将其声明为伪目标，方法是让它成为特殊目标 *.PHONY* 的依赖目标，如下所示： 

```makefile
.PHONY: clean
clean:
		rm *.o temp
```

一旦将 *clean* 指定为伪目标，无论名为 *clean* 的文件是否存在，命令总会被执行。

伪目标在串联 *make* 的递归调用时也非常有用（请参阅 [5.7 make 的递归调用](#5.7 make 的递归调用)）。在这种情形下，*makefile* 总会包含一个变量，该变量列出了多个将要被构建的子目录。一种简单的处理方法是定义一个规则，其命令通过循环遍历子目录，如下：

```makefile
SUBDIRS = foo bar baz

subdirs:
		for dir in $(SUBDIRS); do \
			$(MAKE) -C $$dir; \
		done
```

然而，这种方法存在很多问题。首先，子目录的构建中， *make* 检测到的任何错误都会被忽略，也就是说一条命令执行失败时，将会继续构建其余的目录。当然，通过添加 *shell* 命令可以解决这个问题，但不幸的是，这会使得 *make* 的 *-k* 参数失效。此外，更重要的一点是，你将无法利用 *make* 并行构建（请参阅 [5.4 并行执行](#5.4 并行执行)）的能力，因为只有一条规则。

你可以通过将子目录声明为伪目标（你应当这么做，因为很显然子目录总会存在，否则将不会构建）解决这些问题。

```makefile
SUBDIRS = foo bar baz

.PHONY: subdirs $(SUBDIRS)

subdirs: $(SUBDIRS)

$(SUBDIRS):
		$(MAKE) -C $@
		
foo: baz
```
这里我们做了这样的声明，*baz* 子目录完成构建前，*foo* 子目录不会构建。在并行构建中，这种关系声明非常重要。

对于 *.PHONY* 目标，隐式规则（请参阅 [10. 隐式规则](#10. 隐式规则)）的搜索过程将会被跳过。这就是 *.PHONY* 能提升性能的原因，甚至你根本就不用关心真实文件是否存在。

伪目标不应当是一个真实目标文件的依赖，如果是，那么每次 *make* 更新这个文件的时候，伪目标的命令都会被执行。只要伪目标不是一个真实目标的依赖，那么只有伪目标被指定为最终目标时，其命令才会被执行（请参阅 [9.2 通过参数指定最终目标](#9.2 通过参数指定最终目标)）。

伪目标也可以有依赖。当一个目录下包含多个程序时，通过一个 *./Makefile* 描述所有程序将非常便利。因为 *makefile* 里面的第一个目标时默认执行的目标，所以通常将命名为 *all* 的伪目标作为第一个目标，并将所有独立的程序作为其依赖。例如：

```makefile
all: prog1 prog2 prog3
.PHONY: all

prog1: prog1.o utils.o
		cc -o prog1 prog1.o utils.o
		
prog2: prog2.o
		cc -o prog2 prog2.o
		
prog1: prog1.o sort.o utils.o
		cc -o prog3 prog3.o sort.o utils.o
```
你可以通过 *make* 执行者三个程序，或者将他们中的部分指定为 *make* 参数（如：*make prog1 prog3*）。伪目标不能继承，也就是说其依赖本身不是伪目标，除非显式指定。

当一个伪目标是另一个伪目标的依赖时，其将成为另一个伪目标的子路径。例如，下面的 *make cleanall* 将会删除目标文件，差异文件和程序文件。

```makefile
.PHONY: cleanall cleanobj cleandiff

cleanall: cleanobj cleandiff
		rm program
		
cleanobj:
		rm *.o
		
cleandiff:
		rm *.diff
```



## 4.8 特殊内置目标

一些特定的名字作为目标时，具有特殊的含义。

.PHONY

*.PHONY* 的依赖目标将被视为伪目标。*make* 遇到伪目标时，将无条件运行其命令，无论伪目标同名称的文件是否存在，也不管该文件最后是什么时间修改的。 请参阅 [4.5 伪目标](#4.5 伪目标)。

.SECONDEXPANSION

如果 *.SECONDEXPANSION* 在 *makefile* 中被指定为目标，无论在什么位置，在所有的 *makefile* 被读取后， *.SECONDEXPANSION* 之后定义的所有依赖列表都将会进行二次展开。请参阅 [二次展开](#3.9 二次展开)。

# 5. 编写规则的命令

## 5.4 并行执行 

## 5.5 命令中的错误

## 5.7 make 的递归调用

# 6. 使用变量

## 6.14 特殊变量

# 9. 如何运行 make

## 9.2 通过参数指定最终目标

## 9.7 选项概要

# 10. 隐式规则

# 14. GNU make 的特色功能

# 15. 不兼容性和缺失功能

# 附录 A 快速索引

# 附录 C 复杂 Makefile

# 参考文献

[1] [GNU make](https://www.gnu.org/software/make/manual/make.html#Overview)

[2] [维基教科书 - GNU make](https://zh.m.wikibooks.org/wiki/GNU_make)

