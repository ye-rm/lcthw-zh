# 练习4：Valgrind 介绍

> 原文：[Exercise 4: Introducing Valgrind](http://c.learncodethehardway.org/book/ex4.html)

> 译者：[飞龙](https://github.com/wizardforcel)

现在是介绍另一个工具的时间了，在你学习C的过程中，你会时时刻刻用到它，它就是 `Valgrind`。我现在就向你介绍 `Valgrind`，是因为从现在开始你将会在“如何使它崩溃”一节中用到它。`Valgrind`是一个运行你的程序的程序，并且随后会报告所有你犯下的可怕错误。它是一款相当棒的自由软件，我在编写C代码时一直使用它。

回忆一下在上一章中，我让你移除`printf`的一个参数，来使你的代码崩溃。它打印出了一些奇怪的结果，但我并没有告诉你为什么它会这样打印。这个练习中我们要使用`Valgrind`来搞清楚为什么。

> 注

> 这本书的前几章讲解了一小段代码，同时掺杂了一些必要的工具，它们在本书的剩余章节会用到。这样做的原因是，阅读这本书的大多数人都不熟悉编译语言，也必然不熟悉自动化的辅助工具。通过先让你懂得如何使用`make`和`Valgrind`，我可以在后面使用它们更快地教你C语言，以及帮助你尽早找出所有的bug。

> 这一章之后我就不再介绍更多的工具了，每章的内容大部分是代码，以及少量的语法。然而，我也会提及少量工具，我们可以用它来真正了解发生了什么，以及更好地了解常见的错误和问题。

## 安装 Valgrind

你可以用OS上的包管理器来安装`Valgrind`，但是我想让你学习如何从源码安装程序。这涉及到下面几个步骤：

+ 下载源码的归档文件来获得源码
+ 解压归档文件，将文件提取到你的电脑上
+ 运行`./configure`来建立构建所需的配置
+ 运行`make`来构建源码，就像之前所做的那样
+ 运行`sudo make install`来将它安装到你的电脑

下面是执行以上步骤的脚本，我想让你复制它：

```sh
# 1) Download it (use wget if you don't have curl)
curl -O http://valgrind.org/downloads/valgrind-3.6.1.tar.bz2

# use md5sum to make sure it matches the one on the site
md5sum valgrind-3.6.1.tar.bz2

# 2) Unpack it.
tar -xjvf valgrind-3.6.1.tar.bz2

# cd into the newly created directory
cd valgrind-3.6.1

# 3) configure it
./configure

# 4) make it
make

# 5) install it (need root)
sudo make install
```

按照这份脚本，但是如果 `Valgrind` 有新的版本请更新它。如果它不能正常执行，也请试着深入研究原因。

## 使用 Valgrind

使用 `Valgrind` 十分简单，只要执行`valgrind theprogram`，它就会运行你的程序，随后打印出你的程序运行时出现的所有错误。在这个练习中，我们会崩溃在一个错误输出上，然后会修复它。

首先，这里有一个`ex3.c`的故意出错的版本，叫做`ex4.c`。出于练习目的，将它再次输入到文件中：

```c
#include <stdio.h>

/* Warning: This program is wrong on purpose. */

int main()
{
    int age = 10;
    int height;

    printf("I am %d years old.\n");
    printf("I am %d inches tall.\n", height);

    return 0;
}
```

你会发现，除了两个经典的错误外，其余部分都相同：

+ 没有初始化`height`变量
+ 没有将`age`变量传入第一个`printf`函数

## 你会看到什么

现在我们像通常一样构建它，但是不要直接运行，而是使用`Valgrind`来运行它（见源码："使用Valgrind构建并运行 ex4.c"）：

```sh
$ make ex4
cc -Wall -g    ex4.c   -o ex4
ex4.c: In function 'main':
ex4.c:10: warning: too few arguments for format
ex4.c:7: warning: unused variable 'age'
ex4.c:11: warning: 'height' is used uninitialized in this function
$ valgrind ./ex4
==3082== Memcheck, a memory error detector
==3082== Copyright (C) 2002-2010, and GNU GPL'd, by Julian Seward et al.
==3082== Using Valgrind-3.6.0.SVN-Debian and LibVEX; rerun with -h for copyright info
==3082== Command: ./ex4
==3082==
I am -16775432 years old.
==3082== Use of uninitialised value of size 8
==3082==    at 0x4E730EB: _itoa_word (_itoa.c:195)
==3082==    by 0x4E743D8: vfprintf (vfprintf.c:1613)
==3082==    by 0x4E7E6F9: printf (printf.c:35)
==3082==    by 0x40052B: main (ex4.c:11)
==3082==
==3082== Conditional jump or move depends on uninitialised value(s)
==3082==    at 0x4E730F5: _itoa_word (_itoa.c:195)
==3082==    by 0x4E743D8: vfprintf (vfprintf.c:1613)
==3082==    by 0x4E7E6F9: printf (printf.c:35)
==3082==    by 0x40052B: main (ex4.c:11)
==3082==
==3082== Conditional jump or move depends on uninitialised value(s)
==3082==    at 0x4E7633B: vfprintf (vfprintf.c:1613)
==3082==    by 0x4E7E6F9: printf (printf.c:35)
==3082==    by 0x40052B: main (ex4.c:11)
==3082==
==3082== Conditional jump or move depends on uninitialised value(s)
==3082==    at 0x4E744C6: vfprintf (vfprintf.c:1613)
==3082==    by 0x4E7E6F9: printf (printf.c:35)
==3082==    by 0x40052B: main (ex4.c:11)
==3082==
I am 0 inches tall.
==3082==
==3082== HEAP SUMMARY:
==3082==     in use at exit: 0 bytes in 0 blocks
==3082==   total heap usage: 0 allocs, 0 frees, 0 bytes allocated
==3082==
==3082== All heap blocks were freed -- no leaks are possible
==3082==
==3082== For counts of detected and suppressed errors, rerun with: -v
==3082== Use --track-origins=yes to see where uninitialised values come from
==3082== ERROR SUMMARY: 4 errors from 4 contexts (suppressed: 4 from 4)
$
```

> 注

> 如果你运行了`Valgrind`，它显示一些类似于`by 0x4052112: (below main) (libc-start.c:226)`的东西，而不是`main.c`中的行号，你需要使用`valgrind --track-origins=yes ./ex4`命令来运行你的`Valgrind`。由于某些原因，`valgrind`的Debian和Ubuntu上的版本会这样，但是其它的不会。

上面那段输出非常长，因为`Valgrind`在明确地告诉你程序中的每个错误都在哪儿。让我们从开头逐行分析一下（行号在左边，你可以参照）：

1

你执行了通常的`make ex4`来构建它。确保你看到的`cc`命令和它一样，并且带有`-g`选项，否则`Valgrind`的输出不会带上行号。

2~6

要注意编译器也会向你报告源码的错误，它警告你“向格式化函数传入了过少的变量”，因为你忘记包含`age`变量。

7

然后使用`valgrind ./ex4`来运行程序。

8

之后`Valgrind`变得十分奇怪，并向你报错：

　　14~18

　　在`main (ex4.c:11)`（意思是文件`ex4.c`的`main`函数的第11行）的那行中，有“大小为8的未初始化的值”。你通过查看错误找到了它，并且在它下面看到了“栈踪迹”。最开始看到的那行`(ex4.c:11)`在最下面，如果你不明白哪里出错了，你可以向上看，比如`printf.c:35`。通常最下面的一行最重要（这个例子中是第18行）。

　　20~24

　　下一个错误位于 `main` 函数中的 `ex4.c:11`。`Valgrind`不喜欢这一行，它说的是一些 if 语句或者 while 循环基于一个未初始化的值，在这个例子中是`height`。

　　25~35

　　剩下的错误都大同小异，因为这个值还在继续使用。

37~46

最后程序退出了，`Valgrind`显示出一份摘要，告诉你程序有多烂。

这段信息读起来会相当多，下面是你的处理方法：

+ 无论什么时候你运行C程序并且使它工作，都应该使用`Valgrind`重新运行它来检查。
+ 对于得到的每个错误，找到“源码:行数”提示的位置，然后修复它。你可以上网搜索错误信息，来弄清楚它的意思。
+ 一旦你的程序在`Valgrind`下不出现任何错误信息，应该就好了。你可能学会了如何编写代码的一些技巧。

在这个练习中我并不期待你马上完全掌握`Valgrind`，但是你应该安装并且学会如何快速使用它，以便我们将它用于后面的练习。

## 附加题

+ 按照上面的指导，使用`Valgrind`和编译器修复这个程序。
+ 在互联网上查询`Valgrind`相关的资料。
+ 下载另一个程序并手动构建它。尝试一些你已经使用，但从来没有手动构建的程序。
+ 看看`Valgrind`的源码是如何在目录下组织的，并且阅读它的Makefile文件。不要担心，这对我来说没有任何意义。

# 

> [ye-rm](https://github.com/ye-rm)在2025/3/12重校，考虑到这两节内容还是有用的，从PDF版本中加入，练习序号为书中序号

# 练习 4. 使用调试器

> 原作者的视频是没有的，我建议自己找一个视频学一学

这是一个以视频讲解为主的练习，我会在视频里向你展示如何使用你电脑自带的调试器来调试程序、检测错误，甚至调试正在运行的进程。请观看配套视频以深入了解这个主题。

#### GDB 技巧

以下是你可以用 GNU 调试器（GDB）实现的一些简单技巧：

- `gdb --args`：通常情况下，gdb 会把你提供的参数当作是给它自己的。使用 `--args` 可以将这些参数传递给要调试的程序。
- `thread apply all bt`：打印所有线程的调用栈回溯信息。这非常有用。
- `gdb --batch --ex r --ex bt --ex q --args`：运行程序，这样如果程序崩溃，你就能得到调用栈回溯信息。

#### GDB 快速参考

视频有助于你学习如何使用调试器，但在实际操作时，你需要随时查阅相关命令。以下是我在视频中使用过的 GDB 命令的快速参考，以便你在后续学习本书内容时使用：

- `run [args]`：使用指定的参数 `[args]` 启动你的程序。
- `break [file:]function`：在指定的文件（可选）和函数处设置断点。你也可以用 `b` 作为简写。
- `backtrace`：打印当前调用栈的回溯信息。简写为 `bt`。
- `print expr`：打印表达式 `expr` 的值。简写为 `p`。
- `continue`：继续运行程序。简写为 `c`。
- `next`：执行下一行代码，但会跳过函数调用。简写为 `n`。
- `step`：执行下一行代码，如果是函数调用则进入函数内部。简写为 `s`。
- `quit`：退出 GDB。
- `help`：列出命令的类型。之后你可以获取某类命令或特定命令的帮助信息。
- `cd`、`pwd`、`make`：这些命令的使用方式和在 shell 中一样。
- `shell`：快速启动一个 shell，以便你可以执行其他操作。
- `clear`：清除一个断点。
- `info break`、`info watch`：显示断点和观察点的相关信息。
- `attach pid`：附加到一个正在运行的进程，以便对其进行调试。
- `detach`：从进程中分离。
- `list`：列出接下来的十行源代码。加上 `-` 则列出前十行代码。

#### LLDB 快速参考

在 OS X 系统中，你不再使用 GDB，而是要使用一个类似的程序，即 LLDB 调试器（LLDB）。其命令几乎相同，以下是 LLDB 的快速参考：

- `run [args]`：使用指定的参数 `[args]` 启动你的程序。
- `breakpoint set --name [file:]function`：在指定的文件（可选）和函数处设置断点。你也可以用 `b`，这个更简便。
- `thread backtrace`：打印当前调用栈的回溯信息。简写为 `bt`。
- `print expr`：打印表达式 `expr` 的值。简写为 `p`。
- `continue`：继续运行程序。简写为 `c`。
- `next`：执行下一行代码，但会跳过函数调用。简写为 `n`。
- `step`：执行下一行代码，如果是函数调用则进入函数内部。简写为 `s`。
- `quit`：退出 LLDB。
- `help`：列出命令的类型。之后你可以获取某类命令或特定命令的帮助信息。
- `cd`、`pwd`、`make`：使用方式和在 shell 中一样。
- `shell`：快速启动一个 shell，以便你可以执行其他操作。
- `clear`：清除一个断点。
- `info break`、`info watch`：显示断点和观察点的相关信息。
- `attach -p pid`：附加到一个正在运行的进程，以便对其进行调试。
- `detach`：从进程中分离。
- `list`：列出接下来的十行源代码。加上 `-` 则列出前十个源代码行。

你也可以在网上搜索 GDB 和 LLDB 的快速参考卡片和教程。



> 建议搜索 C 语言运算符优先级，相关的图片有很多，下文图片暂缺

# 练习 5. 记住 C 语言运算符

当你学习第一门编程语言时，很可能是通过阅读一本书，输入一些你不太理解的代码，然后试图弄清楚它是如何工作的。我写的其他大部分书也是采用这种方式，这对初学者来说非常有效。一开始，在你理解所有符号和单词的含义之前，有一些复杂的主题需要掌握，所以这是一种简单的学习方法。

然而，一旦你已经掌握了一门编程语言，这种通过潜移默化摸索来学习语法的方法就不是学习一门新语言的最有效方式了。这种方法虽然可行，但有一种更快的方法可以提升你使用该语言的技能和信心。这种学习编程语言的方法可能看起来像魔法，但你得相信我，它的效果出奇地好。

我希望你学习 C 语言的方式是，先记住所有基本符号和语法，然后通过一系列练习来应用它们。这种方法与你学习人类语言的方式非常相似，即先记住单词和语法，然后在对话中应用你所记住的内容。一开始只需付出一点记忆的努力，你就能获得基础知识，并且在阅读和编写 C 代码时会更轻松。

#### 警告！

有些人坚决反对死记硬背。通常，他们声称这会让你变得缺乏创造力且无趣。我就是一个例子，证明记住东西并不会让你变得缺乏创造力和无趣。我会画画、弹奏和制作吉他、唱歌、编程、写书，而且我记住了很多东西。这种观点完全没有根据，并且对高效学习有害。请忽略任何跟你说这种话的人。

#### 如何记忆

记忆东西的最佳方法其实相当简单：

1. 制作一套闪卡，一面写上符号，另一面写上符号的描述。你也可以使用一款名为 Anki 的程序在电脑上完成这个任务。我更喜欢自己制作，因为在制作的过程中就有助于我记忆。
2. 将闪卡打乱顺序，然后开始看一面。尽力在不看另一面的情况下回忆出内容。
3. 如果你记不起另一面的内容，就看一下，然后自己重复答案，再把这张卡放到一个单独的堆里。
4. 当你看完所有卡片后，会得到两堆卡片：一堆是你能快速回忆起来的，另一堆是你没能回忆起来的。拿起没记住的那堆卡片，只针对这些卡片进行强化记忆。
5. 在每次学习时段（通常为 15 - 30 分钟）结束时，你会有一些怎么都记不住的卡片。无论你走到哪里都带上这些卡片，有空的时候就练习记忆它们。

记忆东西还有很多其他技巧，但我发现这是让你能对需要立即使用的内容形成即时记忆的最佳方法。C 语言的符号、关键字和语法都是你需要即时回忆起来的内容，所以这种方法非常适合这个任务。

还要记住，你需要对闪卡的两面都进行学习。你应该既能根据描述知道对应的符号，也能说出符号的描述。

最后，在记忆这些运算符的过程中你不必停下来。最好的方法是将记忆和本书中的练习结合起来，这样你就可以应用你所记住的内容。关于这一点，可参考下一个练习。

#### 运算符列表

首先是算术运算符，它们与几乎所有其他编程语言中的算术运算符非常相似。制作闪卡时，描述面应说明它是算术运算符以及它的作用。

关系运算符用于测试值是否相等，同样，它们在编程语言中非常常见。

逻辑运算符用于执行逻辑测试，你应该已经知道它们的作用了。唯一比较特别的是逻辑三目运算符，你会在本书后面学到它。

位运算符在现代代码中你可能不常遇到。它们以各种方式改变组成字节和其他数据类型的位。我在本书中不会详细介绍，但在处理某些底层系统时，它们非常有用。

赋值运算符只是将表达式赋值给变量，但 C 语言将大量其他运算符与赋值操作结合在一起。所以当我说 “按位与并赋值” 时，指的是位运算符，而不是逻辑运算符。

我把这些称为数据运算符，但实际上它们处理的是 C 语言中指针、成员访问和数据结构的各个方面。

最后，有一些杂项符号，它们要么经常用于不同的用途（如逗号 `,`），要么由于各种原因不属于前面提到的任何类别。

在继续学习本书内容的同时，学习你的闪卡。如果你每天在学习前花 15 - 30 分钟，睡前再花 15 - 30 分钟，那么你很可能在几周内记住所有这些内容。
