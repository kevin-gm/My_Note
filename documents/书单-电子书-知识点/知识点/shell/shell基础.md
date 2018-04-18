### 一、导读
shell是跟计算机交互的中间桥梁

Shell 除了能解释用户输入的命令，将它传递给内核，还可以：

- 调用其他程序，给其他程序传递数据或参数，并获取程序的处理结果；
- 在多个程序之间传递数据，把一个程序的输出作为另一个程序的输入；
- Shell 本身也可以被其他程序调用。

由此可见，Shell 是将内核、程序和用户连接了起来。

常见的 Shell 有 sh、bash、csh、tcsh、ash 等。

#### sh
sh 的全称是 Bourne shell，由 AT&T 公司的 Steve Bourne开发，为了纪念他，就用他的名字命名了。

sh 是 UNIX 上的标准 shell，很多 UNIX 版本都配有 sh。sh 是第一个流行的 Shell。
#### csh
sh 之后另一个广为流传的 shell 是由柏克莱大学的 Bill Joy 设计的，这个 shell 的语法有点类似C语言，所以才得名为 C shell ，简称为 csh。

Bill Joy 是一个风云人物，他创立了 BSD 操作系统，开发了 vi 编辑器，还是 Sun 公司的创始人之一。
> BSD 是 UNIX 的一个重要分支，后人在此基础上发展出了很多现代的操作系统，最著名的有 FreeBSD、OpenBSD 和 NetBSD，就连 Mac OS X 在很大程度上也基于BSD。
#### tcsh
tcsh 是 csh 的增强版，加入了命令补全功能，提供了更加强大的语法支持。
#### ash
一个简单的轻量级的 Shell，占用资源少，适合运行于低内存环境，但是与下面讲到的 bash shell 完全兼容。
#### bash
bash shell 是 Linux 的默认 shell，本教程也基于 bash 编写。

bash 由 GNU 组织开发，保持了对 sh shell 的兼容性，是各种 Linux 发行版默认配置的 shell。
> bash 兼容 sh 意味着，针对 sh 编写的 Shell 代码可以不加修改地在 bash 中运行。

尽管如此，bash 和 sh 还是有一些不同之处：

- 一方面，bash 扩展了一些命令和参数；
- 另一方面，bash 并不完全和 sh 兼容，它们有些行为并不一致，但在大多数企业运维的情况下区别不大，特殊场景可以使用 bash 代替 sh。


### 查看 Shell
Shell 是一个程序，一般都是放在 ```/bin``` 或者 ```/user/bin``` 目录下，当前 Linux 系统可用的 Shell 都记录在 ```/etc/shells``` 文件中

	[root@VM_0_14_centos ~]# cat /etc/shells
	/bin/sh
	/bin/bash
	/sbin/nologin
	/usr/bin/sh
	/usr/bin/bash
	/usr/sbin/nologin
	/bin/tcsh
	/bin/csh

在现代的 Linux 上，sh 已经被 bash 代替，```/bin/sh``` 往往是指向 ```/bin/bash``` 的符号链接

### 查看默认shell

	[root@VM_0_14_centos ~]# echo $SHELL
	/bin/bash


### Shell提示符(CentOS7)
对于普通用户，Base shell 默认的提示符是美元符号 ```$```；对于超级用户（root 用户），Bash Shell 默认的提示符是井号 ```#```

	[ec2-user@ip-172-31-8-238 ~]$ su root
	Password: 
	[root@ip-172-31-8-238 ec2-user]#

- 当前shell用户名，ec2-user
- 本地主机名称，ip-172-31-8-238
- 当前目录

Shell 通过PS1和PS2两个环境变量来控制提示符格式：

- PS1 控制最外层命令行的提示符格式。
- PS2 控制第二层命令行的提示符格式。

所谓命令提示符，就是提示用户接下来可以或者需要输入命令，进行进一步的操作。

在 Shell 中初次输入命令，使用的是 PS1 指定的提示符格式；如果输入一个命令后还需要输入附加信息，Shell 就使用 PS2 指定的提示符格式。请看下面的例子：

	[mozhiyan@localhost ~]$ echo "C语言中文网"
	C语言中文网
	[mozhiyan@localhost ~]$ echo "http://c.biancheng.net"
	http://c.biancheng.net
	[mozhiyan@localhost ~]$ echo "
	> yan
	> chang
	> sheng
	> "
	yan
	chang
	sheng
	[mozhiyan@localhost ~]$ 

echo 是一个输出命令，可以用来输出数字、变量、字符串等；本例中，我们使用 echo 来输出字符串。

字符串是一组由" "包围起来的字符序列，echo 将第一个"作为字符串的开端，将第二个"作为字符串的结尾。此处的字符串就可以看做 echo 命令的附加信息。

本例中，前两次使用 echo 命令时都是在后面紧跟字符串，一行之内输入了完整的附加信息。第三次使用 echo 时，将字符串分成多行，echo 遇到第一个 ```"``` 认为是不完整的附加信息，所以会继续等待用户输入，直到遇见第二个 ```"``` 。输入的附加信息就是第二层命令，所以使用 ```>``` 作为提示符。

要显示提示符的当前格式，可以使用 echo 输出 PS1 和 PS2：

	[mozhiyan@localhost ~]$ echo $PS1
	[\u@\h \W]\$
	[mozhiyan@localhost ~]$ echo $PS2
	>
	[mozhiyan@localhost ~]$ 

Shell 使用以\为前导的特殊字符来表示命令提示符中包含的要素，这使得 PS1 和 PS2 的格式看起来可能有点奇怪。下表展示了可以在 PS1 和 PS2 中使用的特殊字符。



| 字符 | 描述 |
| - | - |
| \a | 铃声字符 |
| \d | 格式为“日 月 年”的日期
| \e | ASCII转义字符
| \h | 本地主机名
| \H | 完全合格的限定域主机名
| \j | shell当前管理的作业数
| \1 | shell终端设备名的基本名称
| \n | ASCII换行字符
| \r | ASCII回车
| \s | shell的名称
| \t | 格式为“小时:分钟:秒”的24小时制的当前时间
| \T | 格式为“小时:分钟:秒”的12小时制的当前时间
| \@ | 格式为am/pm的12小时制的当前时间
| \u | 当前用户的用户名
| \v | bash shell的版本
| \V | bash shell的发布级别
| \w | 当前工作目录
| \W | 当前工作目录的基本名称
| \! | 该命令的bash shell历史数
| \# | 该命令的命令数量
| \$ | 如果是普通用户，则为美元符号$；如果超级用户（root 用户），则为井号#。
| \nnn | 对应于八进制值 nnn 的字符
| \\ | 斜杠
| \[ | 控制码序列的开头
| \] | 控制码序列的结尾

注意，所有的特殊字符均以反斜杠\开头，目的是与普通字符区分开来。您可以在命令提示符中使用以上任何特殊字符的组合。

我们可以通过修改 PS1 变量来修改提示符格式，例如：

	[mozhiyan@localhost ~]$ PS1="[\t][\u]\$ "
	[17:27:34][mozhiyan]$ 


### 执行shell
新建一个文件，扩展名为sh(sh代表shell)。扩展名不影响脚本执行，扩展名为php也没有关系。

在文件中输入一些内容，比如：

	#!/bin/bash
	echo "Hello World !"

\#! 是一个约定的标记，它告诉系统这个脚本需要什么解释器来执行，即使用哪一种Shell。echo命令用于向窗口输出文本。

运行Shell脚本有两种方法。

#### 作为可执行程序

	chmod +x ./test.sh  #使脚本具有执行权限
	./test.sh  #执行脚本

注意，一定要写成 ```./test.sh```，而不是 ```test.sh``` 。运行其它二进制的程序也一样，直接写test.sh，linux系统会去PATH里寻找有没有叫test.sh的，而只有/bin, /sbin, /usr/bin，/usr/sbin等在PATH里，你的当前目录通常不在PATH里，所以写成test.sh是会找不到命令的，要用./test.sh告诉系统说，就在当前目录找。

#### 作为解释器参数
这种运行方式是，直接运行解释器，其参数就是shell脚本的文件名，如：

	/bin/sh test.sh

可以使用 ```read``` 从 ```stdin``` 获取输入

	#!/bin/bash
	# Author : mozhiyan
	# Copyright (c) http://see.xidian.edu.cn/cpp/linux/
	# Script follows here:
	echo "What is your name?"
	read PERSON
	echo "Hello, $PERSON"

运行脚本：

	chmod +x ./test.sh
	./test.sh
	What is your name?
	mozhiyan
	Hello, mozhiyan