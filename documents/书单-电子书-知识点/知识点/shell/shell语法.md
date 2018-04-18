## 变量
shell是一种弱语言，也就是定义变量时，不需要指明变量类型。

在 Bash shell 中，每一个变量的值都是字符串，无论你给变量赋值时有没有使用引号，值都会以字符串的形式存储。

这意味着，Bash shell 在默认情况下不会区分变量类型，即使你将整数和小数赋值给变量，它们也会被视为字符串

当然，如果有必要，你也可以使用 declare 关键字显式定义变量的类型，但在一般情况下没有这个需求，Shell 开发者在编写代码时自行注意值的类型即可。

Shell 变量的命名规范和大部分编程语言都一样：

+ 变量名由数字、字母、下划线组成；
+ 必须以字母或者下划线开头；
+ 不能使用 Shell 里的关键字（通过 help 命令可以查看保留关键字）。

使用一个定义过的变量，只要在变量名前面加美元符号 ```$``` 即可

	# 定义变量 url
	url=http://c.biancheng.net
	# 使用变量
	# {}是可选的，加上，可以确认变量边界
	echo ${url}
	echo "hello ${url}world"

注意，赋值号的周围不能有空格，这可能和你熟悉的大部分编程语言都不一样。

#### 单引号和双引号的区别

	#!/bin/bash
	url="http://c.biancheng.net"
	website1='C语言中文网：${url}'
	website2="C语言中文网：${url}"
	echo $website1
	echo $website2

运行结果：

	C语言中文网：${url}
	C语言中文网：http://c.biancheng.net

以单引号' '包围变量的值时，单引号里面是什么就输出什么，即使内容中有变量和命令（命令需要反引起来）也会把它们原样输出

#### 将命令的结果赋值给变量

Shell 也支持将命令的执行结果赋值给变量，常见的有以下两种方式：

	variable=`command`
	variable=$(command) # 推荐

示例

	[mozhiyan@localhost code]$ log=$(cat log.txt)
	[mozhiyan@localhost code]$ echo $log

#### 只读变量
使用 readonly 命令可以将变量定义为只读变量，只读变量的值不能被改变

	myUrl="http://see.xidian.edu.cn/cpp/shell/"
	readonly myUrl

#### 删除变量
使用 unset 命令可以删除变量

	unset variable_name
变量被删除后不能再次使用；unset 命令不能删除只读变量。

#### 变量类型

运行shell时，会同时存在三种变量：

**1) 局部变量**
>局部变量在脚本或命令中定义，仅在当前shell实例中有效，其他shell启动的程序不能访问局部变量。

2) 环境变量
>所有的程序，包括shell启动的程序，都能访问环境变量，有些程序需要环境变量来保证其正常运行。必要的时候shell脚本也可以定义环境变量。

3) shell变量
>shell变量是由shell程序设置的特殊变量。shell变量中有一部分是环境变量，有一部分是局部变量，这些变量保证了shell的正常运行

## Shell特殊变量

|变量|含义|
|-|-|
|$0|当前脚本的文件名
|$n|	传递给脚本或函数的参数。n 是一个数字，表示第几个参数。例如，第一个参数是$1，第二个参数是$2。
|$# |	传递给脚本或函数的参数个数。
|$* |	传递给脚本或函数的所有参数。
|$@ |	传递给脚本或函数的所有参数。被双引号(" ")包含时，与 $* 稍有不同，下面将会讲到。
|$? |	上个命令的退出状态，或函数的返回值。
|$$ |	当前Shell进程ID。对于 Shell 脚本，就是这些脚本所在的进程ID。

#### 命令行参数
运行脚本时传递给脚本的参数称为命令行参数。命令行参数用 $n 表示，例如，$1 表示第一个参数，$2 表示第二个参数，依次类推。

请看下面的脚本：

	#!/bin/bash
	echo "File Name: $0"
	echo "First Parameter : $1"
	echo "First Parameter : $2"
	echo "Quoted Values: $@"
	echo "Quoted Values: $*"
	echo "Total Number of Parameters : $#"
运行结果：

	$./test.sh Zara Ali
	File Name : ./test.sh
	First Parameter : Zara
	Second Parameter : Ali
	Quoted Values: Zara Ali
	Quoted Values: Zara Ali
	Total Number of Parameters : 2

#### $* 和 $@ 的区别

$* 和 $@ 都表示传递给函数或脚本的所有参数，不被双引号(" ")包含时，都以"$1" "$2" … "$n" 的形式输出所有参数。

但是当它们被双引号(" ")包含时，"$*" 会将所有的参数作为一个整体，以"$1 $2 … $n"的形式输出所有参数；"$@" 会将各个参数分开，以"$1" "$2" … "$n" 的形式输出所有参数。

	#!/bin/bash
	echo "\$*=" $*
	echo "\"\$*\"=" "$*"
	echo "\$@=" $@
	echo "\"\$@\"=" "$@"
	echo "print each param from \$*"
	for var in $*
	do
	    echo "$var"
	done
	echo "print each param from \$@"
	for var in $@
	do
	    echo "$var"
	done
	echo "print each param from \"\$*\""
	for var in "$*"
	do
	    echo "$var"
	done
	echo "print each param from \"\$@\""
	for var in "$@"
	do
	    echo "$var"
	done
执行 ./test.sh "a" "b" "c" "d"，看到下面的结果：

	$*=  a b c d
	"$*"= a b c d
	$@=  a b c d
	"$@"= a b c d
	print each param from $*
	a
	b
	c
	d
	print each param from $@
	a
	b
	c
	d
	print each param from "$*"
	a b c d
	print each param from "$@"
	a
	b
	c
	d

#### 退出状态
$? 可以获取上一个命令的退出状态。所谓退出状态，就是上一个命令执行后的返回结果。

退出状态是一个数字，一般情况下，大部分命令执行成功会返回 0，失败返回 1。

不过，也有一些命令返回其他值，表示不同类型的错误。