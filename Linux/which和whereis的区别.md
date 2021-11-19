### 前言

一般而言，倘若我们要在 Linux 操作系统上开发或者移植一个功能或脚本时，对于功能中涉及的命令或者可执行文件需要使用绝对路径。

而要在 Linux 操作系统上搜索一条命令的绝对路径，你一定会选择 `which` 或者 `whereis` 两个命令中的一个来辅助你查找可执行命令的路径。当然，`which` 和 `whereis` 也可以满足你的觉大部分需求。

但同时作为查找命令的绝对路径，`which` 和 `whereis` 有什么区别呢？什么场景下更推荐 `which`，什么场景下推荐 whereis 呢？

### 分析

在看 `which` 和 `whereis` 的区别之前，先来看看两个命令官方给的解释。

#### which

```bash
#man which
NAME
       which - shows the full path of (shell) commands.

SYNOPSIS
       which [options] [--] programname [...]

DESCRIPTION
       Which takes one or more arguments. For each of its arguments it prints to stdout the full path of the executables that would have been executed when this argument had been entered at the shell prompt. It does this by searching for an executable or script in
       the directories listed in the environment variable PATH using the same algorithm as bash(1).

       This man page is generated from the file which.texinfo
```

`which` 命令用来展示shell命令的完整路径，它接收一个或多个参数，执行时所有的参数会展示在shell提示并根据传入的参数将可执行命令的完整路径输出到 `stdout` 中。

当你在 Linux 终端下试图查找一条命令的完整路径时， 终端将输出查找到的匹配的第一条可执行命令的绝对路径，如下

```bash
#which tree
/bin/tree
```

若需要查找命令对应的所有完整路径，你可以在加上 `-a` 参数

```bash
#which -a tree
/bin/tree
/usr/bin/tree
```

那么，`which` 查找命令时是从哪里查找的？

我们在终端输入一条不存在的命令，来看看 which 会返回什么。

```bash
#which initialization
/usr/bin/which: no initialization in (/usr/lib64/qt-3.3/bin:/root/perl5/bin:/usr/local/sbin:/usr/local/bin:/sbin:/bin:/usr/sbin:/usr/bin:/opt/ibutils/bin:/opt/dell/srvadmin/bin:/opt/dell/srvadmin/sbin:/root/bin)
```

当输入一个不存在的命令时，终端返回了类似 `no {command} in (xxxx)` 这样的提示，看到这些 `/usr/local/bin` 等路径你是否会觉得熟悉。没错，这就是 Linux 操作系统上配置的环境变量。

也即是说， `which` 查找一条命令是根据当前 Linux 的环境变量来查找的。

被查找的命令在环境变量配置的对应目录下，则该命令可以被查找到并返回该命令的绝对路径；倘若环境变量配置的所有目录下没有找到对应的命令，则会出现上面的提示。

那么，如何通过 `which` 能查找到 `initialization` 这个可执行文件呢？你可以将 `initialization` 可执行文件移动到环境变量所配置的目录下，或者将 `initialization` 文件当前的路径添加到环境变量中。不过，针对单一命令，你应该尽可能使用第一种方法。毕竟，随意添加环境变量并不总是好的。

#### whereis

```bash
#man whereis
WHEREIS(1)                                                                                                                    User Commands                                                                                                                    WHEREIS(1)

NAME
       whereis - locate the binary, source, and manual page files for a command

SYNOPSIS
       whereis [options] [-BMS directory... -f] name...

DESCRIPTION
       whereis  locates  the binary, source and manual files for the specified command names.  The supplied names are first stripped of leading pathname components and any (single) trailing extension of the form .ext (for example: .c) Prefixes of s.  resulting from
       use of source code control are also dealt with.  whereis then attempts to locate the desired program in the standard Linux places, and in the places specified by $PATH and $MANPATH.
```

`whereis` 查找命令时会返回查找到的二进制文件、源码文件、和命令对应的手册信息，搜索时 whereis 会将提供的文件名前面的路径与后面的扩展名去掉，并找到与之匹配的文件。

当使用 `whereis` 查找命令的绝对路径及手册时，终端会返回可以被匹配到的命令的绝对路径、源文件及手册，如下

```bash
#whereis tree
tree: /usr/bin/tree /usr/share/man/man1/tree.1.gz
```

`whereis` 搜索时提供了一些参数，用来根据需要查找。例如，

通过添加 `-b` 参数只查找可执行文件的路径

```bash
#whereis -b tree
tree: /usr/bin/tree
```

通过添加 `-m` 参数只查找命令相关的手册

```bash
#whereis -m tree
tree: /usr/share/man/man1/tree.1.gz
```

通过添加 `-s` 参数查找对应的源文件

```bash
#whereis -s tree
tree:
```

正如你所看到的，`whereis` 查找到的内容不存在时，并不会出现报错信息，你可以通过 `echo $?` 来验证这一点。

```bash
#whereis initialization
initialization:

#echo $?
0
```

那么，`whereis` 查找信息是从哪里查找的呢？

事实上，`whereis` 命令提供了 `-l` 参数用来查询当前 `whereis` 查找命令的所有有效路径

```bash
#whereis -l
bin: /usr/bin
bin: /usr/sbin
bin: /usr/lib
bin: /usr/lib64
...
man: /usr/share/man/pt_PT
man: /usr/share/man/ro
man: /usr/share/man/zh
src: /usr/src/debug
src: /usr/src/kernel
```

可以看到，`whereis` 查找信息主要分为3部分，其中可执行文件的路径查找在 `bin: ` 后的所有目录下查找；手册查找在 `man: `后的所有目录下查找；源文件在 `src: ` 后的所有目录下查找。

事实上，`bin: ` 后的所有目录不正是 Linux的环境变量吗？

最后，如果你仅仅只需要查找命令的绝对路径，那么使用 `which` 则更便于程序解析，当命令不存在时，你也能第一时间反应到文件不存在；

若你需要命令的源程序信息，那么你只能使用 `whereis` 了。

