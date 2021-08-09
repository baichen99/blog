---
title: python 命令行参数
date: 2020-12-17 22:44:40
tags:
- python2
- 命令行
- 参数
- commandline
- arguments
categories:
- python
---

本分翻译自 https://realpython.com/python-command-line-arguments



python处理命令行参数的功能为一些**基于文本命令行**的程序提供了一个用户友好的界面。这类似于图形用户界面（一个由图形元素或部件操作的可视化应用程序）。

python提供了可以获取、提取命令行参数的机制。这些参数可以用来更改程序的行为。举个例子，假如你的程序要处理一个文件中的数据，那么你可以把文件路径传给程序，而不是在源代码中写死。

**通过此教程，你将会了解：**

- python命令行参数的起源
- python命令行参数的底层支持
- 设计命令行界面的指导标准
- 手动定制以及处理命令行参数的简单方法
- 使用python中的库简化复杂命令行界面的开发

如果你想在不使用专用库的情况下开发一个用户友好的命令行界面，或者是想更好的理解现有的python命令行参数库的一些共识，那么请继续阅读！

## 命令行界面 The Command Line Interface

**命令行界面(CLI)** 为用户提供了一种方式，使用户可以和运行在**基于文本的shell解释器**中的程序进行交互。

shell解释器有Linux上的bash，windows上的命令提示行等。命令行界面由能显示命令提示符的shell解释器支持。它一般有以下几个要素：

- 一个命令或一段程序
- 0或多个命令行参数
- 一个输出，代表命令结果
- 使用或帮助的参考文档

不是每个命令行界面都提供以上要素，这些也不是命令行界面的全部特点。命令行的复杂性表现在从传递单个参数到多个参数和选项，很像领域专用语言(Domain Specific Language)。举个例子，一些程序可能会从命令行启动web版的文档，或者像python那样打开一个交互解释器。

下面两个python命令的例子展示出了命令行界面的样子：

```shell
$ python -c "print(Real Python)"
Real Python
```

在这个例子里，python解释器接收`-c`参数，它表示将在选项`-c`之后的参数作为Python程序执行。

下面这个例子展示了使用`-h`调用python来显示`help`信息。

```shell
$ python -h
usage: python3 [option] ... [-c cmd | -m mod | file | -] [arg] ...
Options and arguments (and corresponding environment variables):
-b     : issue warnings about str(bytes_instance), str(bytearray_instance)
         and comparing bytes/bytearray with str. (-bb: issue errors)
[ ... complete help text not shown ... ]
```



## C语言历史遗留

Python命令行参数是从c语言继承而来的。在[Guido Van Rossum](https://en.wikipedia.org/wiki/Guido_van_Rossum) 于1993年写的[写给Unix/C程序员的Python介绍](http://citeseerx.ist.psu.edu/viewdoc/summary?doi=10.1.1.47.4180)中提到了，C对Python是有很大的影响的。Guido提及了字面量的定义，标识符，操作符以及像`break`, `continue`,`return`之类的语句。Python命令行参数也很大程度上受到了C语言的影响。

为了说明两个语言之间的相似性，情况下面的C语言程序：

```c
// main.c
#include <stdio.h>

int main(int argc, char *argv[]) {
    printf("Arguments count: %d\n", argc);
    for (int i = 0; i < argc; i++) {
        printf("Argument %6d: %s\n", i, argv[i]);
    }
    return 0;
}
```

第四行定义了`main()`, c程序的入口函数。下面的参数要记好笔记：

1. **argc** 是代表参数个数的一个整数

2. **argv** 是是一个字符指针的数组，包含程序名(第一个参数)，后面是其他参数(如果有的话)

你可以在linux环境下使用`gcc -o main main.c`上面的代码，然后用`./main`执行得到下面的结果。

```shell
$ gcc -o main main.c
$ ./main
Arguments count: 1
Argument      0: ./main
```

如果没有用`-o`选项指明，gcc编译器将默认使用`a.out`作为输出可执行文件名。它代表**汇编输出(assembler output)**, 让人想起在旧的UNIX系统上生成的可执行文件。而且观察到，可执行文件的名称./main是唯一的参数。
让我们使用同一个程序，并传递几个Python命令行参数：

```shell
$ ./main Python Command Line Arguments
Arguments count: 5
Argument      0: ./main
Argument      1: Python
Argument      2: Command
Argument      3: Line
Argument      4: Arguments
```

输出结果显示参数的个数是5，参数列表包括程序名main，后面是在命令行传递的“Python Command Line Arguments”的每个单词。

> **注意**: `argc` 代表 **argument count**, 而 `argv` 代表 **argument vector**. 想知道更多内容, 可以查看 [A Little C Primer/C Command Line Arguments](https://en.wikibooks.org/wiki/A_Little_C_Primer/C_Command_Line_Arguments).

上述编译`main.c`文件假定你是用的是Linux或Mac OS系统。在Windows上你也可以用以下几个方法来编译C程序。

- [**Windows Subsystem for Linux (WSL):**](https://en.wikipedia.org/wiki/Windows_Subsystem_for_Linux) It’s available in a few Linux distributions, like [Ubuntu](https://ubuntu.com/), [OpenSUSE](https://www.opensuse.org/), and [Debian](https://www.debian.org/), among others. You can install it from the Microsoft Store.
- [**Windows Build Tools:**](https://visualstudio.microsoft.com/downloads/#build-tools-for-visual-studio-2019) This includes the Windows command line build tools, the Microsoft C/C++ compiler [`cl.exe`](https://docs.microsoft.com/en-us/cpp/build/walkthrough-compiling-a-cpp-cli-program-on-the-command-line?view=vs-2019), and a compiler front end named [`clang.exe`](https://en.wikipedia.org/wiki/Clang) for C/C++.
- [**Microsoft Visual Studio:**](https://visualstudio.microsoft.com/downloads/) This is the main Microsoft integrated development environment (IDE). To learn more about IDEs that can be used for both Python and C on various operating systems, including Windows, check out [Python IDEs and Code Editors (Guide)](https://realpython.com/python-ides-code-editors-guide/).
- [**mingw-64 project:**](http://mingw-w64.org/) This supports the [GCC compiler](https://gcc.gnu.org/) on Windows.

如果你已经安装了Microsoft Visual Studio或者Windows Build Tools，那么你可以用下面的方法编译main.c:

```shell
C:/>cl main.c
```

 你将会获得一个名为`main.exe`的可执行文件，可以用下面命令执行：

```
C:/>main
Arguments count: 1
Argument      0: main
```

你可以实现一个Python程序，main.py，这和上面的C程序main.c是一样的：

```python
# main.py
import sys

if __name__ == "__main__":
    print(f"Arguments count: {len(sys.argv)}")
    for i, arg in enumerate(sys.argv):
        print(f"Argument {i:>6}: {arg}")  
```

在代码中并没有看到像C语言中argc的变量，他在Python中并不存在，因为`sys.argv`已经够用了。你可以不用知道参数列表长度，来解析`sys.argv`的命令行参数，如果你的程序需要知道参数长度的话，也可以调用内置`len()`函数。

请注意`enumerate()`，当应用于一个可迭代对象时，他返回一个可枚举对象(enumerate object)，该对象可以同时返回sys.argv元素的索引和它对应的值。这让我们可以遍历sys.argv，而不用维护一个计数变量。

main.py运行结果如下：

```shell
$ python main.py Python Command Line Arguments
Arguments count: 5
Argument      0: main.py
Argument      1: Python
Argument      2: Command
Argument      3: Line
Argument      4: Arguments
```

sys.argv包含了和C语言中argv相同的信息:

- **程序名称**`main.py`是参数列表第一个参数
- `Python`, `Command`, `Line`, and `Arguments`是剩下的参数
- 译者注：代码中:>6是让变量i代表的字符串占据6个字符宽度并且右对齐，类似于C语言中的"%6d"

通过对C语言比较难懂部分的一些简短的介绍，你现在可以去学习更多关于Python命令行参数的内容了。

## 两个Unix实用工具

为了在本教程中使用Python命令行参数，你将实现Unix生态系统中两个实用程序的部分功能。

1. [sha1sum](https://en.wikipedia.org/wiki/Sha1sum)
2. [seq](https://en.wikipedia.org/wiki/Seq_(Unix))

在下面几个部分中，你将会熟悉这些unix工具。

### sha1sum

sha1sum 计算SHA-1哈希值，它通常用来验证文件的完整性。给定一个输入，哈希函数返回相同的值。任何对输入的改变都将会导致输出不同的哈希值。在使用带有具体参数的实用工具之前，你可以先打印一下help信息。

> 译者注：**SHA-1**（英语：Secure Hash Algorithm 1，中文名：安全散列算法1）是一种[密码散列函数](https://baike.baidu.com/item/密码散列函数)。SHA-1可以生成一个被称为消息摘要的160位散列值，通常为40个十六进制数。（1个十六进制数是4位二进制数）

```shell
$ sha1sum --help
Usage: sha1sum [OPTION]... [FILE]...
Print or check SHA1 (160-bit) checksums.

With no FILE, or when FILE is -, read standard input.

  -b, --binary         read in binary mode
  -c, --check          read SHA1 sums from the FILEs and check them
      --tag            create a BSD-style checksum
  -t, --text           read in text mode (default)
  -z, --zero           end each output line with NUL, not newline,
                       and disable file name escaping
[ ... complete help text not shown ... ]
```

显示命令行程序的帮助信息是命令行界面提供的常用功能。

要计算一个文件内容的SHA-1 hash值，可以按如下操作进行：

```shell
$ sha1sum main.c
125a0f900ff6f164752600550879cbfabb098bc3  main.c
```

返回的结果第一部分显示的是SHA-1哈希值，第二部分是文件名。而且此命令可以接受多个文件名作为参数：

```shell
$ sha1sum main.c main.py
125a0f900ff6f164752600550879cbfabb098bc3  main.c
d84372fc77a90336b6bb7c5e959bcb1b24c608b4  main.py
```

由于Unix终端的通配符扩展特性，也可以提供带有通配符字符的命令行参数。这其中一个字符是asterisk，或者可以称作星号(*)：

```shell
$ sha1sum main.*
3f6d5274d6317d580e2ffc1bf52beee0d94bf078  main.c
f41259ea5835446536d2e71e566075c1c1bfc111  main.py
```

shell把main.\*转换成main.c和main.py并且把他们传入sha1sum，因为当前文件夹下的这两个文件，都和main.\*所匹配。该程序会计算参数列表的每个文件的SHA1值。在Windows上你会发现他的行为有所不同。Windows没有通配符扩展，所以程序可能必须得适应这一点。你可能需要在内部扩展通配符。

在没有参数的情况下，sha1sum从标准输入中读取。你可以通过在键盘上输入字符来传入数据。输入的字符流包括任何字符，包括回车键。要终止输入，必须用Enter发出文件结束的信号，然后输入Ctrl+D。

结果是为文本`Real\nPython\n`生成的SHA1哈希值。文件的名称是-。这是一个表示标准输入的惯例。当你执行以下命令时，会得到相同的哈希值。

```shell
$ python -c "print('Real\nPython\n', end='')" | sha1sum
87263a73c98af453d68ee4aab61576b331f8d9d6  -
$ python -c "print('Real\nPython')" | sha1sum
87263a73c98af453d68ee4aab61576b331f8d9d6  -
$ printf "Real\nPython\n" | sha1sum
87263a73c98af453d68ee4aab61576b331f8d9d6  -
```

接下来，你将会阅读的是对**seq**的简单介绍 。

### seq

[seq](https://en.wikipedia.org/wiki/Seq_%28Unix%29)可以生成数字**序列**。在最基本的形式中，比如生成1-5的序列，你可以依照如下指令执行：

```shell
$ seq 5
1
2
3
4
5
```

为了seq提供的所有功能，你可以在命令行打印出帮助信息：

```sh
$ seq --help
Usage: seq [OPTION]... LAST
  or:  seq [OPTION]... FIRST LAST
  or:  seq [OPTION]... FIRST INCREMENT LAST
Print numbers from FIRST to LAST, in steps of INCREMENT.

Mandatory arguments to long options are mandatory for short options too.
  -f, --format=FORMAT      use printf style floating-point FORMAT
  -s, --separator=STRING   use STRING to separate numbers (default: \n)
  -w, --equal-width        equalize width by padding with leading zeroes
      --help     display this help and exit
      --version  output version information and exit
[ ... complete help text not shown ... ]
```

在此教程中，你将会编写一些sha1sum和seq的简单变体。在每个例子中，你将会了解到Python命令行参数不同的特点和功能。

在Mac OS和Linux上，sha1sum和seq需要预先安装，尽管不同的系统或发布版本的特性和帮助信息可能会有所不同。如果你正在使用Windows 10，那么运行sha1sum和seq的最简单的方法则是在[WSL](https://docs.microsoft.com/en-us/windows/wsl/install-win10)中安装Linux环境。如果你无法访问提供标准Unix实用程序的终端，你或许可以使用一些在线终端：

- 在[PythonAnywhere](https://www.pythonanywhere.com/)上创建一个免费账户，然后启动一个Bash控制台。
- 在[repl.it](https://repl.it/languages)上创建一个临时Bash。

以上只是两个例子，你也可以找到其他的替代品。

## sys.argv.Arrary

在探索一些公认的约定和了解如何处理Python命令行参数之前，你需要知道对所有Python命令行参数的底层支持是由[`sys.argv`](https://docs.python.org/library/sys.html?highlight=sys argv#sys.argv)所提供的。 以下各节中的示例将向你展示如何处理存储在sys.argv中的Python命令行参数以及处理当访问它时所遇到的典型的错误。在这章你将会了解：

- 如何**访问**sys.argv的内容
- 如何**消除**sys.argv的全局特性所带来的副作用
- 如何在Pythobn命令行参数中**处理**空格
- 如何在访问Python命令行参数时**处理**错误 
- 如何**接收**以字节形式传递的Python命令行参数的原始格式

让我们开始吧！

### 显示参数

sys模块提供了一个列表叫做argv，它包括：

1. argv[0] 包含当前Python程序的名称
2. argv[1:]，列表剩余部分，包含所有传给这个程序的Python命令行参数

下面这个例子展示了sys.argv的内容：

```python
# argv.py
import sys

print(f"Name of the script      : {sys.argv[0]=}")
print(f"Arguments of the script : {sys.argv[1:]=}")
```

下面是代码解释：

- 第2行倒入了Python的内部模块sys。
- 第4行通过访问sys.argv的第一个元素提取了程序名称。
- 第5行通过获取sys.argv的剩余元素，显示了Python命令行参数。

> **注意：**在上面argv.py中使用的[f-string](https://realpython.com/python-f-strings/)语法使用了Python 3.8中的新调试说明符。 要了解有关此f字符串新功能和其他功能的更多信息，请查看Python 3.8中的新功能。
>
> 如果你的Python版本低于3.8，那么只需要移除两个f-string中的=使程序得以运行。输出结果回展示变量名而不是它们的名字。

使用任意参数列表执行上面的脚本argv.py：

```shell
$ python argv.py un deux trois quatre
Name of the script      : sys.argv[0]='argv.py'
Arguments of the script : sys.argv[1:]=['un', 'deux', 'trois', 'quatre']
```

### 反转首个参数
现在你对sys.argv有了足够的了解，接下来你将会对命令行传递的参数进行操作。示例程序example.py把在命令行传递的第一个参数逆转了过来。

```python
# reverse.py

import sys

arg = sys.argv[1]
print(arg[::-1])
```

在reverse.py中程序通过以下几个步骤反转第一个参数：

- 第5行获取程序存储在`sys.argv`中的index为1的参数。记住程序名存储在sys.argv的index 0的位置。
- 第6行打印了反转字符串。 args[::-1]是python中使用分片操作来反转列表的方法。

按照下面运行脚本：

```shell
$ python reverse.py "Real Python"
nohtyP laeR
```

如预期的那样，`reverse.py`对"Real Python"进行处理，并且反转了唯一参数并输出"nohtyP laeR"。注意多个单词"Real Python"周围的引号确保解释器将其处理为唯一参数而不是两个。你将在后面的部分中深入探讨参数分隔符。

### 转换 sys.argv

sys.argv对正在运行的Python程序全局可用。 在执行过程中导入的所有模块都可以直接访问sys.argv。 这种全局访问可能很方便，但是sys.argv并非一成不变。 您可能想要实现一种更可靠的机制，以*将程序参数提供给Python程序中的不同模块*，尤其是在具有多文件的复杂程序中。

观察如果你随意使用`sys.argv`会发生什么：

```python
# argv_pop.py

import sys

print(sys.argv)
sys.argv.pop()
print(sys.argv)
```

调用[`.pop()`](https://docs.python.org/tutorial/datastructures.html#more-on-lists)方法移除`sys.argv`中最后一个元素。

执行上面的脚本：

```shell
$ python argv_pop.py un deux trois quatre
['argv_pop.py', 'un', 'deux', 'trois', 'quatre']
['argv_pop.py', 'un', 'deux', 'trois']
```

注意，第四个参数从sys.argv中被移除。

在上述简短的脚本中，您可以安全地依靠对sys.argv的全局访问，但是在较大规模的程序中，您可能希望将参数存储在单独的变量中。 前面的示例可以进行如下修改：

```python
# argv_var_pop.py

import sys

print(sys.argv)
args = sys.argv[1:]
print(args)
sys.argv.pop()
print(sys.argv)
print(args)
```

这次尽管`sys.argv`移除了最后一个元素，但`args`仍然保持不变。`args`不是全局的，你可以将其传递给程序，以根据程序的逻辑来解析参数。

```python
def main(args=None):
    if args is None:
        args = sys.argv[1:]
```

在此摘录自[`pip`](https://realpython.com/what-is-pip/) 源码中，main()将sys.argv切片保存到args中，该片仅包含参数而不包含文件名。 sys.argv保持不变，并且对sys.argv的任何更改都不会影响arg。

### 转义空格字符

### 错误处理

### 计算sha1sum



#####  剖析python命令行参数

### 标准

### 选项 option

### 参数 arguments

### 子命令 subcommand

### 窗口

### 视觉



## 一些解析python命令行参数的方法

### 正则表达式

### 文件处理

### 标准输入

### 标准输出和标准错误

### 自定义解析器



## 一些验证python命令行参数的方法

### 使用python data classes进行类型验证

### 自定义验证

## python标准库

### argparse

### getopt

## 第三方库

### Click

### Python Prompt Toolkit

## 总结

## 其他资源








```

```