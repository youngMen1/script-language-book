# 第05讲：Shell 编程核心技术（上）

在本课时我们主要讲解 Shell 编程核心技术，也就是如何编写Shell脚本。

在我们的日常工作中经常需要编写一些 Shell 逻辑脚本来批量处理一些任务，比如读取输入数据进行相关处理，将任务放入一个脚本进行自动化运行等场景，这些都涉及 Shell 编程，其实 Shell 编程也非常简单，就像 Java、Python 这些大众化的编程语言一样，也具备编程语言的特性，我们来看下 Shell 编程主要涉及的几个方面的内容。

* 变量

* 逻辑控制

* Shell 环境

* 脚本应用

* 自动化

## 变量

### 变量定义

学习一门编程技术，首先需要掌握如何使用变量，在 Shell 中定义一个变量非常简单，它与在 Python 中定义一个变量非常相似，你可以在任意位置定义一个变量并给它赋值，而 Shell 比 Python 更简洁的地方在于不强制输入单引号或双引号去指明内容。

![](/static/image/CgotOV3fosGAOY1tAANwPox6f7I970.png)

举个例子，我们定义一个 x=1 的变量，然后输入 echo $x 指令，其中 $x 表示可以引用这个变量。

![](/static/image/CgoB5l3fosKAcC0AAANOqJ5Rmsk957.png)

又比如输入 hello 字符串赋值给变量 x，然后再打印 echo $x。

![](/static/image/CgotOV3fosKAVC89AAOEgykUuPg498.png)

如果输入 x=hello world 呢，这时系统运行会报错，因为 x=hello 会被认为是一个变量赋值，而 world 会被认为是另外一个独立的命令。

![](/static/image/CgoB5l3fosKAdqgFAAOGtJpRb8Q155.png)

那我们应该怎么做呢？和 Python 一样，我们只需要对字符串加上单引号或双引号。然后再打印这个变量 echo $x 就可以了。

![](/static/image/CgotOV3fosKAcDsaAAN5G0JEAsA725.png)

而使用单引号和双引号的区别在于是否支持转义，双引号可以对转义符号进行处理。

在变量的定义中，有几点需要我们注意。

* = 左右不要有空格；

* 如果内容有空格，需要使用单引号或双引号；

* 双引号支持转义符，$ 开头的变量会被自动替换。

在变量的使用中还有几个方法需要注意，echo $a、echo "$a" 都可以表示变量输出，但如果你想使用中更严谨建议使用双引号，还有就是多个字符串串联使用时，使用 {} 对内容括起来表示该内容是变量，避免与后面的其他字符混淆。

## 预定义变量

在 Shell 编程中，系统还提供了几个预定义变量。比如：

* PWD 表示当前目录；

* USER 表示当前用户；

* HOME 表示当前用户的主目录，HOME 还可以用 ~ 简写；

* PATH 表示当前所有可执行程序；

* RANDOM 可以产生一个随机数。

![](/static/image/CgoB5l3fosOAVQSoAAUa3CT95gI876.png)

其中，我们需要特别注意 PATH 变量，比如输入 echo $PATH 指令，你可以看到 PATH 变量定义了所有可执行程序，定义在 PATH 中的程序可以直接调用程序名执行而不必再输入程序的全部路径。

![](/static/image/CgotOV3fosSATo1pAASkBuOkl-M111.png)

比如使用 which ls 指令，你会发现 ls 在 /bin/ls 目录下，打印 echo $PATH 变量，你会发现里面有对应的 /bin 目录，但如果程序没有定义在 PATH 变量中，就需要你打印全部路径来执行程序 。

## 特殊符号的使用

除了这些之外，还需要掌握一些特殊的符号，比如：

* 双引号用于括起一段字符串，支持 $var 形式的变量替换；

* 单引号也表示其内容是字符串，但不支持转义；

* $'\n' 表示 ANSI-C 引用；

*  反斜线，某些情况下表示转义；

* \(\(a=a+3\)\) 是整数扩展，双括号里面的变量作为整数处理；

* $\(ls\) 执行命令并把结果保存为变量，简写为\`\`；

* {1..10} 等价于 seq 1 10，表示 1~10 数字；

* seq 1 3 10 表示生成一个步进为 3 的 1~10 的数字。

![](/static/image/CgoB5l3fosSAeiWNAAN3mkLaCuI573.png)

其中，需要重点介绍整数扩展，你可以把数学表达式放到双括号中进行相应的计算，比如输入 echo $\(\(10000/3\)\)，它的计算结果是 3333，这里为什么没有小数位呢？是因为 Shell 中目前仅处理整数，如果你需要精确到小数位可以使用 awk 指令。Shell的整数计算不使用 $ 也是可以的，但如果你想引用具体的结果值，就需要使用 $。

![](/static/image/CgotOV3fosWAUtBiAAMX01YRWG0720.png)

比如输入 x=$\(echo xxxx\) 指令将字符串 xxxx 存入 x 变量中，这时输入 echo $x 指令，你会看到 $\(\) 会把括号内的命令执行然后把输出结果作为值传递给变量。

![](/static/image/CgoB5l3fosWAKUt6AAMqCZTcpAk900.png)

还有一个需要注意的是序列，比如我们想从 1~10 获取一个序列，可以输入 echo {1..10} 指令来打印 1~10 的数字。

![](/static/image/CgotOV3fosWAUSepAAJPWmR_Pm8293.png)

还可以使用 seq 1 10，它们是等价的。

## 变量类型

然后是变量类型，在 Shell 中是不区分变量类型的，这一点和 Python 很像，所有的变量都是基础类型，只在运行时做动态解析。其中字符串、数字、布尔是比较常用的。

首先是字符串，字符串常用于一些类似掐头、去尾、替换的操作，课后你可以自己去练习，这里不再具体演示。

然后是布尔类型，布尔的基本表示表现形式是 true 和 false，需要注意的是在 Shell 中有一个特殊的用法，如果是某个命令返回的是 0，则表示这个进程是正常工作的。

![](/static/image/CgoB5l3fosWADT10AAIM0bYzP1o997.png)

比如输入 ls /tmp/hello.txt ;echo $? 指令，你可以看到返回值是 0，表示该进程运行正确，如果返回其他值就表示进程运行错误。

![](/static/image/CgotOV3fosaAaPu1AAJ8n7xBAno779.png)

比如输入 ls /tmp/hello.txtdddd ;echo $? 指令运行一个不存在的文件，输出为 1 表示运行错误。

## 判断

判断主要包括算数判断，与或非的逻辑判断，还有一些 Shell 中内置的判断三个方面。首先来看下算数判断，通常算数判断可以用来比较两个变量间的关系，比如两个数字的大小比较，字符串的匹配关系，等等。

除了简单的条件判断之外，Shell 还支持复杂的与、或、非逻辑判断。

除此之外，Shell 还提供了一些内置判断，比如：

* -e file 表示如果文件存在，则结果为真；

* -d file 表示如果文件是一个子目录，则结果为真；

* -f file 表示如果文件是一个普通文件，则结果为真；

* -r file 表示如果文件可读，则结果为真；

* -s file 表示如果文件的长度不为 0，则结果为真；

* -w file 表示如果文件可写，则结果为真；

* -x file 表示如果文件可执行，则结果为真。

因为数组不经常使用，这里就不再详细讲解，如果你感兴趣可以课后自己练习。

## 逻辑控制

学完变量的相关知识，我们继续学习逻辑控制，基于数据可以设计一些逻辑，如下所示：

* 条件判断 if；

* 分支判断 case、select，根据不同的条件进行不同的处理；

* 循环 for、while、until；

* break 和 continue，退出循环。

你可以看到整个逻辑控制和 Python 是很相似的。

## if 判断

首先看下 if 条件判断，if 首先检测判断条件是否成立，如果成立则执行 then 语句块内的逻辑，else 执行判断不成立的逻辑，还有 elif...if...，它类似于 Python，当条件都不满足时去判断下一个条件。

## for 循环

然后是 for 循环，for 循环在 Shell 中有两种用法，第一种类似 Java 或 Python ，从 1~10 进行循环，这个时候可以使用 for\(\(i=0;i&lt;10;i++\)\) 实现，这种用法是根据基数进行精准循环次数的一个判断。

第二种用法是 for 遍历循环，你可以使用 for...in...语句块。

![](/static/image/CgoB5l3fosaAOw4CAAHflNfAfkU288.png)

比如输入 for i in $\(seq 1 3 10\) 指令，然后在 do 语句块中打印 echo $i 的值，do 语句块以 done 结束。

![](/static/image/CgotOV3fosaALQ6mAAHflNfAfkU836.png)

你可以看到打印了 1、4、7、10，这个就是 for 循环的遍历用法。

## while 循环

最后是 while 循环，和 for 循环很像，while 首先判断条件，条件成立则在 do 语句块中执行操作。

![](/static/image/CgoB5l3foseAOrrnAAHk4lHDC9s130.png)

举个例子，定义 i=0，然后输入 while \(\(i&lt;3\)\);do\(\(i=i+1\)\);sleep 1;echo $i;done 指令。

你可以看到，输出结果每隔 1 秒打印一个 i 的值，而 i 的值逐渐增加，直到等于 3 时不再满足条件，退出循环。

而 while 还有一个很常用的功能，就是通过 while read line 循环读取文件的每一行。

![](/static/image/CgoB5l3foseADrLRAAIeM1hcmns056.png)

比如输入 while read line；do echo $line;done&lt; /tmp/hello.txt 指令，它就会打印出文件的每一行信息。

![](/static/image/CgotOV3fosiAdS1TAAIysEIC3Y0290.png)

除此之外，使用管道也是可以的，输入 cat /tmp/hello.txt \| while read line;do echo $line;done 指令，输出效果是一样的。

## 退出控制

而有一些复杂的条件需要适时推出，这个时候就需要我们掌握控制推出的语句，比如：

* return 函数返回；

* exit 脚本进程推出；

* break 退出当前循环；

* continue 跳出当前循环，进入下一次循环。



