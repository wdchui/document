
# Linux 文本处理技巧

当我们需要处理json格式或者csv格式的日志文件时，可以使用Linux命令来完成一些简单的处理操作，减少编程时间，提高工作效率。列举几种我在工作中遇到的场景
    
 - 确认昨天上报的几十G日志中是否有上报某个字段
 - 提取日志上报中某个字段的值
 - 统计日志上报中某个字段上报值的和

以上3种场景我们可以通过Linux文本处理三剑客grep、sed、awk来处理。首先我们简单介绍下三个命令的使用

## grep

一款强大的文本搜索工具，支持正则表达式搜索。它从标准输入、管道、文件读取内容，将匹配的结果输出到标准输出。

#### Usage: grep [option]...pattern [file]...

##### 常用OPTION

 - -e 指定匹配模式，可指定多个，只有一个匹配模式时可省略
 - -f 指定匹配模式文件，文件内容为一个或多个匹配模式，每个匹配模式放一行
 - -i 忽略匹配文本的大小写
 - -v 反向匹配
 - -c 计算匹配次数
 - -r 递归搜索子目录
 - -w 全词匹配，类比Mysql中的精准匹配
 - -l 多文件搜索时只显示匹配的文件名，而不输出文件匹配内容

#### 使用举例

- 查找/opt/case/txt_data日志目录下所有文件中是否有上报ST_01字段
```shell
	grep -r 'ST_01' /opt/case/txt_data
```

- 查找demo.log日志文件中上报ST_01或ST_02的记录数
```shell
	grep -c -e 'ST_01' -e 'ST_02' demo.log
```

- 查找demo.log日志文件中上报ST_01且ST_02的记录数
```shell
	grep 'ST_01'  demo.log | grep -ce 'ST_02'
```

- 查找demo.log日志文件中未上报ST_01的记录数
```shell
	grep -v 'ST_01' demo.log
```

## sed
文本替换的利器。可从标准输入、管道、文件中读取内容，每次处理一行内容，并将读取的内容输出到标准输出。
#### Usage: sed [option]... {script} [file]...

##### 常用OPTION

 - -e 命令行指定要执行的script，可指定多个
 - -n 静默模式，只输出处理的行，一般与script中的指令'p'配合使用
 - -f 指定要执行的script文件
 - -i[suffix] 修改覆盖源文件，指定suffix时生成以suffix结尾的备份文件

##### script
script 由地址和指令构成，地址指定要操作的行，指令为要执行的动作。
###### 地址
 - 1 首行
 - n 第n行
 - $ 尾行
 - n1,n2 从n1行到n2行
 - n1,+10 从n1行起之后的10行
 - /pattern/ 匹配pattern模式的行，支持正则
 - n1~n2 以n1为基准，以n2为间隔处理行
###### 指令
 - p 打印指令，常与 -n 一起使用
 - d 删除指令
 - a[text] 向下追加指令，追加text内容到下一行
 - i[text] 向上追加指令, 追加text内容到上一行
 - c[text] 文本替换指令, 替换本行内容为text
 - s 查找替换指令 格式：'s/查找串/替换串/'

#### 使用举例
 - 替换标准输入中的hello word 为hello sed
```shell
	echo hello word | sed  's/word/sed/'
```

 - 替换demo.txt文件中的hello word 为 hello sed
```shell
 	sed 's/hello word/hello sed/g' demo.txt
```

 - 替换demo.txt文件中的hello word 为 hello sed, 并将修改应用到文件
```shell
	sed -i 's/hello word/hello sed/g' demo.txt
```

 - 提取日志文件demo.txt文件中A0001字段的值，文件每行内容示例：{"count":{"ST_01":2}, "common":{"A0001":"ABC123", "A0002":"124"}
```shell
	sed 's/.*A0001\":\"\([^",]*\)\".*/\1/g' demo.txt
```
对于json格式的文件内容还可以使用jq命令处理，jq的使用详细可参照https://www.ibm.com/developerworks/cn/linux/1612_chengg_jq/index.html

## awk
awk是一种编程语言，用于在linux/unix下对文本和数据进行处理。数据可以来自标准输入(stdin)、一个或多个文件，或其它命令的输出。它支持用户自定义函数和动态正则表达式等先进功能，是linux/unix下的一个强大编程工具。它在命令行中使用，但更多是作为脚本来使用。awk有很多内建的功能，比如数组、函数等，这是它和C语言的相同之处，灵活性是awk最大的优势。
#### Usage: awk [option]... {script} [file]...

##### 常用OPTION
 - F 指定输入分隔符
 - v 定义变量，可将外部变量传入awk使用
 - f 指定script脚本

##### script
script 由模式和操作组成
###### 模式
模式可以是以下任意一个：

 - /正则表达式/：使用通配符的扩展集。
 - 关系表达式：使用运算符进行操作，可以是字符串或数字的比较测试。
 - 模式匹配表达式：用运算符~（匹配）和~!（不匹配）。
 - BEGIN语句块、pattern语句块、END语句块

###### 操作
操作由一个或多个命令、函数、表达式组成，之间由换行符或分号隔开，并位于大括号内，主要部分是：

 - 变量或数组赋值
 - 输出命令
 - 内置函数
 - 控制流语句

##### 基本结构 awk 'BEGIN{ print "start" } pattern{ commands } END{ print "end" }' file
一个awk脚本通常由：BEGIN语句块、能够使用模式匹配的通用语句块、END语句块3部分组成，这三个部分是可选的。任意一个部分都可以不出现在脚本中，脚本通常是被单引号或双引号中

##### 常用的内置变量
 - $n 当前记录的第n个字段，比如n为1表示第一个字段，n为2表示第二个字段。 
 - $0 这个变量包含执行过程中当前行的文本内容。
 - NF 表示字段数，在执行过程中对应于当前的字段数。
 - NR 表示记录数，在执行过程中对应于当前的行号。

#### 使用举例
 - 输出CSV文件中的第2列数据
```shell
	awk -F ',' '{print $2}' file
```

 - 统计/opt/case/txt_data目录下所有文件的总大小
```shell
	ll /opt/case/txt_data | awk '/.txt/{sum +=$2}END{print sum}'
```

 - 提取日志文件demo.txt文件中ST_01字段的值，并做累加和，文件每行内容示例: {"count":{"ST_01":2}, "common":{"A0001":"ABC123", "A0002":"124"}
```shell
	cat demo.txt  |  grep 'ST_01' | sed 's/.*\(\"ST_01\":[^,]*\),.*/\1/g' | awk -F ':' '{sum+=$2}END{print sum}'
```

## 其他常用命令

- sort 排序 详见http://man.linuxde.net/sort
- uniq 去重 详见http://man.linuxde.net/uniq
- comm 文件比较 详见http://man.linuxde.net/comm
- xargs 参数传递 详见http://man.linuxde.net/xargs