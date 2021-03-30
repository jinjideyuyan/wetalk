# 										       							    							 					    						 上古神器AWK

## 前言

哈喽，大家好，今天给大家介绍一个非常之牛逼的Unix工具AWK。AWK是1977年贝尔实验室的三个哥们( [Alfred Aho](http://en.wikipedia.org/wiki/Alfred_Aho)、[Peter Weinberger](http://en.wikipedia.org/wiki/Peter_J._Weinberger)、 [Brian Kernighan](http://en.wikipedia.org/wiki/Brian_Kernighan) )搞出来的文本分析工具，这三个哥们的首字母拼起来就是AWK的名字了。AWK虽然是上个世纪的产物，但是它的简洁和丰富的功能可以称之为**神器**！！！ 它处理文本就像其他语言处理数值一样方便, 所以经常被应用在文本处理领域，比如日志分析、数据清洗、文本过滤、数据统计等。同时它也是一门编程语言，不过它的命令行用法就可以覆盖大多数的应用场景，我们通常可以使用一行AWK命令完成一个脚本的任务！！！

这些数据处理工作都有一些共同&显著的特点：

**(1)     输入数据格式统一** 

比如日志，为了对日志进行统计和上报，我们通常会采用一些分割手段来记录日志(或者json等易于统计的格式)，如下日志采用"|"来分割。

```shell
# 日志格式：{服务}|{日期}|{业务}|{请求URL}|{返回状态}|{请求耗时}|{请求参数}|{返回参数}...
```

比如CSV文件，采用","来分割。

```shell
# CSV格式：field1,field2,field3...
```

**(2)    每一列代表固定含义，便于统计**

跟关系型数据库类似，统计文件的每一行的相同列代表同一个含义的数据，这样才具有数据分析意义，比如本文演示数据，第一列表示地区，第二列表示总人口等。

> 演示数据来源于国家统计局：各地区户口登记地在外乡镇街道的人口状况。由于演示数据文件行数太多占用篇幅较长，以下演示均只展示前几条数据。
>

```shell
$ cat population.txt|head -n 10
地区      合计      本县/市/区  本省其他县/市/区     省外
全国    260937942   90372599     84689006       85876337
北京    10498288    1582574      1871181        7044533
天津    4952225     1095282      865442         2991501
河北    8297279     4263957      2628649        1404673
山西    6764665     3643627      2189385        931653
内蒙古  7170889     2732591      2994117        1444181
辽宁    9310058     3899728      3623800        1786530
吉林    4462177     2604239      1401439        456499
黑龙江  5557828     2800727      2250704         506397
```

## 基本用法

一个AWK程序的组成非常简单，它是由一个或多个 模式–动作 语句组成的序列。

```shell
### 一个 模式-动作
awk 'pattern {action}' input_files

### 多个 模式-动作
awk 'pattern {action} pattern {action} pattern {action} ...' input_files
```

AWK会每次读取一个输入行，对读取到的每一行，按顺序检查每一个模式。如果当前行与模式匹配，则执行对应动作。所以AWK的工作原理就是按顺序执行模式然后执行动作，可以想象到AWK伪代码大概长这样，我猜的(\*^_^\*)。

```shell
### AWK伪代码  我猜的 (*^_^*)
while(getline(inputfile))
{
	if(模式1 == true)
	{
		动作1;
	}
	if(模式2 == true)
	{
		动作2;
	}
	....
}
```

由于许多工作都是自动完成的：包括输入, 字段分割, 存储管理, 初始化，所以我们只需要给出串行的"模式–动作"就可以完成对文件的操作！！！大致的流程图如下：

<img src="https://wetalk-1300208549.cos.ap-nanjing.myqcloud.com/drawImage/awk%E6%B5%81%E7%A8%8B%E5%9B%BE.png" alt="AWK" style="zoom: 67%;" />

AWK在自动扫描输入文件的同时, 也会按照分隔符(默认空格/Tab)把每一个输入行切分成字段。其中$0表示整行，$1,$2...$n对应表示第一列，第二列...第N列。我们可以用print语句来打印验证。

> 使用print函数由逗号分隔不同的参数，打印结果用空格符分隔，并且会自动换行。(类似于各大语言println函数)。

```shell
$ awk '{print "hello",$0}' population.txt|head -n 5
hello 地区      合计      本县/市/区  本省其他县/市/区     省外
hello 全国    260937942   90372599     84689006       85876337
hello 北京    10498288    1582574      1871181        7044533
hello 天津    4952225     1095282      865442         2991501
hello 河北    8297279     4263957      2628649        1404673
```

AWK还提供了很多有用的内置变量，如 

NR  (Number Of Record) ：表示读取到的记录数，即当前行号 

FILENAME ：表示当前输入的文件名

NF (Number Of Field) ：表示当前记录的字段个数，即总共多少列，我们通常用这个变量提取一行的最后一列，如下例子所示，总共有5列，$NF代表的就是第五列的值，等价于$5，$(NF-1)表示倒数第二列的值。

```shell
$ awk '{print FILENAME,NR,$1,$3,NF,$NF}' population.txt|head -n 5
population.txt 1 地区 本县/市/区 5 省外
population.txt 2 全国 90372599 5 85876337
population.txt 3 北京 1582574 5 7044533
population.txt 4 天津 1095282 5 2991501
population.txt 5 河北 4263957 5 1404673
```

**常用的内置变量在附录总结**，方便大家查阅。

AWK也提供了格式化输出，等价于C语言的printf，格式化规则可以参考：https://en.cppreference.com/w/c/io/fprintf

```shell
# printf 格式化输出  printf(format, value1, value2, ... , valuen)
# todo 补充例子
```

## 模式过滤

上面介绍了动作的使用，动作通常用来用来输出展示，模式用来过滤我们想要的记录。
如下筛选（行号>1 且 第一列大于11074525）的行。

```shell
### AWK的变量也可以自由进行算术运算(加减乘除)，比如 $2-$3
$ awk 'NR>1 && $2>11074525 {print NR,$1,$2,$2-$3}' population.txt
2 全国 260937942 170565343
11 上海 12685316 11016029
12 江苏 18226819 13681789
13 浙江 19900863 15274032
17 山东 13698321 7123530
21 广东 36806649 31390437
25 四川 11735152 6913850
```

AWK的字符串拼接跟shell一样简单粗暴，不需要使用任何运算符，将两个字符串并排放在一起就能实现拼接。

```shell
$ awk 'NR>1 {print NR,"pre_"$1"_end"}' population.txt|head -n 5
2 pre_全国_end
3 pre_北京_end
4 pre_天津_end
5 pre_河北_end
6 pre_山西_end
```

内建函数  多展示几个。。。

```shell
### 我的系统编码&文件编码均为UTF-8
$ awk 'length($1) > 6 {print $1,"占用长度：",length($1)}' population.txt
内蒙古 占用长度： 9
黑龙江 占用长度： 9
```

AWK还提供了一些特殊的模式，比如 BEGIN 和 END。这两个模式不匹配任何输入行。
当 awk读取数据前，BEGIN 的语句开始执行，通常用于初始化。如下我们可以用BEGIN来给输出打印一个表头。

```shell
### 多个 "模式-动作" 并排写就行。
$ awk 'BEGIN{print "AREA TOTAL LOCAL OTHER OUTLAND"} NR>2{print}' population.txt|head -n 5
AREA TOTAL LOCAL OTHER OUTLAND
北京    10498288    1582574      1871181        7044533
天津    4952225     1095282      865442         2991501
河北    8297279     4263957      2628649        1404673
山西    6764665     3643627      2189385        931653
```

当所有输入被读取完毕，END 的语句开始执行。通常用来收尾。如下我们可以统计一下第二列大于262005的国家，并在END进行打印。

```shell
$ awk 'NR>2 && $2>262005{count += 1} END{print count"个大于262005的国家"}' population.txt
30个大于262005的国家
```

同一个动作里的多个语句之间使用分号或者换行进行分割。如下在BEGIN的动作中先指定输出分隔符，接着打印表头。

> OFS (Output Formmat Separate) 也是一个内建变量：指定输出字段分割符，如下指定输出时字段采用逗号进行分割

```shell
$ awk 'BEGIN{OFS=",";print "AREA,TOTAL,LOCAL,OTHER,OUTLAND"} NR>2{print $1,$2,$3,$4,$5}' population.txt|head -n 5
AREA,TOTAL,LOCAL,OTHER,OUTLAND
北京,10498288,1582574,1871181,7044533
天津,4952225,1095282,865442,2991501
河北,8297279,4263957,2628649,1404673
山西,6764665,3643627,2189385,931653
```

AWK提供了范围模式可以根据一个区间来匹配多个输入行，范围模式由两个被逗号分开的模式组成。

```shell
awk 'pattern1,pattern2 {action}' input_file
```

AWK从符合 pattern1 的行开始，到符合 pattern2 的行结束 (包括这两行)，对这其中的每一行执行action。如下提取第五行到第十行之间地区的数据。

```shell
$ awk 'NR==5,NR==10" {print NR,$0}' population.txt
5 河北    8297279     4263957      2628649        1404673
6 山西    6764665     3643627      2189385        931653
7 内蒙古  7170889     2732591      2994117        1444181
8 辽宁    9310058     3899728      3623800        1786530
9 吉林    4462177     2604239      1401439        456499
10 黑龙江  5557828     2800727      2250704        506397
```

## 流程控制

前文提到了AWK也是一门编程语言，所以它支持很多编程语言特性(源于C语言)，比如流程控制语句 if-else 、循环(for，while)；比如数据结构数组等。它们只能用在动作里。

```shell
### 流程控制
awk 'NR>2{if($2>4462177) more+=1; else less+=1} END{print "more:",more,"less:",less}' population.txt
awk 'NR>2 && $2>4462177{more+=1} NR>2 && $2<=4462177{less+=1}END{print "more:",more,"less:",less}' population.txt
awk 'BEGIN {for(i=0;i<ARGC;i++) printf "%s\t",ARGV[i]; print ""}' population.txt abc def cdg
### 数组演示 倒序输出
awk 'NR>2{addr[NR]=$1} END{i=NR; while(i>2){print addr[i];i-=1}}' population.txt
```

## 正则表达式

```shell
# 字符串匹配模式 (string-matching pattern) 测试一个字符串是否包含一段可以被正则 表达式匹配的子字符串
# 4. /regular expression/ { statements}
# 当碰到这样一个输入行时, statements 就执行: 输入行含有一段字符串, 而该字符串可 以被 regular expression 匹配.
# 最简单的正则表达式是仅由数字与字母组成的字符串, 就像 Asia, 它匹配的就是它本身. 为了将一个正则表达式切换成一个模式, 只需要用一对斜杠包围起来即可:
# /regexpr/
# expression ~ /regexpr/ 
# expression !~ /regexpr/
# 这个模式匹配那些含有子字符串 Asia 的输入行, 例如 Asia, Asian, 或 Pan-Asiatic

###正则表达式验证
awk '/f.n/' <<< "`echo -e "cat\nbat\nfun\nfin\nfan"`"

### 这里代办 搞一堆例子 简单介绍正则表达式
### 演示一下范围模式  FNR演示
```

## 内建函数

```shell
### 内建算数函数 例子演示
### 内建字符串函数 例子演示
```

## 命令行参数

## 输出到文件/管道

## 文件拆分&分析

```shell
### CSV
# 过滤、统计、查找、聚合、分组，类似于mysql 中 where,count(),group by等单表操作
# 并集、交集、差集、子集，类比于mysql 中 join、where in 操作等
```

## 附录

### 常见的内建变量

| 内建变量 | 补充默认值 含义                |
| -------- | :----------------------------- |
| NF       | 当前记录的字段个数             |
| NR       | 到目前为止读的记录数量         |
| FNR      | 当前输入文件的记录个数         |
| ARGC     | 命令行参数的个数               |
| ARGV     | 命令行参数数组                 |
| FS       | 控制着输入行的字段分割符       |
| FILENAME | 当前输入文件名                 |
| OFS      | 输出字段分割符                 |
| ORS      | 输出的记录的分割符   "\n"      |
| RS       | 控制着输入行的记录分割符  "\n" |

### 常见的内建函数

| 函数                                                     | 含义                                                         |
| -------------------------------------------------------- | ------------------------------------------------------------ |
| length(s)                                                | 返回字符串长度                                               |
| tolower(s)                                               | 字符转为小写。                                               |
| substr(s,p,n)                                            |                                                              |
| match(s,r)                                               |                                                              |
| split(s,a) split(s,a,fs) sprintf(fmt,expr-list) sub(r,s) |                                                              |
| gsub(r,s) gsub(r,s,t ) index(s,t)                        |                                                              |
| int(x)                                                   | x 的整数部分; 当 x 大于 0 时, 向 0 取整 log(x) x 的自然对数 (以 e 为底) |
| sin(x)/cos(x)/sqrt(x)                                    | 正弦/余弦/平方根                                             |
| rand()                                                   | 随机数  配合 srand(x)使用  x 是 rand() 的新的随机数种子      |



