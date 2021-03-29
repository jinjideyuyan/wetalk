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

> 演示数据来源于国家统计局：各地区户口登记地在外乡镇街道的人口状况
>
> 由于演示数据文件行数太多占用篇幅较长，以下演示均只展示前几条数据。

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

AWK在自动地扫描输入文件的同时, 也会按照分隔符(默认空格/Tab)把每一个输入行切分成字段。其中$0表示整行，$1,$2...$n对应表示第一列，第二列...第N列。

```shell
$ awk '{print "hello",$0}' population.txt|head -n 5
hello 地区      合计      本县/市/区  本省其他县/市/区     省外
hello 全国    260937942   90372599     84689006       85876337
hello 北京    10498288    1582574      1871181        7044533
hello 天津    4952225     1095282      865442         2991501
hello 河北    8297279     4263957      2628649        1404673
```

AWK还提供了很多有用的内置变量，如 NR

```shell
$ awk '{print NR,$1,$3}' population.txt|head -n 5
1 地区 本县/市/区
2 全国 90372599
3 北京 1582574
4 天津 1095282
5 河北 4263957
```

## 上手使用

接下来怎么办呢

```shell
awk '{print NR,$0}' population.txt
awk 'NR>2 {print NR,$1,$2-$3}' population.txt
awk 'NR>2 {print NR,$1,$2-$3,$NF}' population.txt
awk 'NR>2 && $2>5302276 {print NR,$1,$2}' population.txt
awk 'NR==1 || $2>5302276 {print NR,$1,$2}' population.txt
awk 'NR==1 || $2-5302276>0{print NR,$1,$2}' population.txt
awk 'length($1) > 6 {print length($1)}' population.txt
awk 'NR>2 {print "pre_"$1"_end"}' population.txt

### 演示printf
### 内建变量表格列举
```



```shell
### 文件结束标志 (Unix 系统是组合键 Control-d
### 演示标准输入

### 例子
awk '$3 > 0 { print $1, $2 * $3 }' emp.data Kathy 40
Mark 100
Mary 121
Susie 76.5
```

```shell
# 同一行多语句采用;分隔
# print  默认用一个空格符分隔  等于println  例子 print ""
# printf 格式化输出  printf(format, value1, value2, ... , valuen)
# 模式控制着动作的执行: 当模式匹配时, 相应的动作便会执行
```

```shell
### 特殊模式
### BEGIN 与 END
awk 'BEGIN {print "AREA TOTAL LOCAL OTHER OUTLAND"} NR>2{print}' population.txt
awk 'BEGIN{OFS=",";print "AREA,TOTAL,LOCAL,OTHER,OUTLAND"} NR>2{print $1,$2,$3,$4,$5}' population.txt
      
### 计数
awk 'NR>2 && $2>262005{count += 1} END{print count"个大于262005的国家"}' population.txt
### 流程控制
awk 'NR>2{if($2>4462177) more+=1; else less+=1} END{print "more:",more,"less:",less}' population.txt
awk 'NR>2 && $2>4462177{more+=1} NR>2 && $2<=4462177{less+=1}END{print "more:",more,"less:",less}' population.txt
awk 'BEGIN {for(i=0;i<ARGC;i++) printf "%s\t",ARGV[i]; print ""}' population.txt abc def cdg
### 数组演示 倒序输出
awk 'NR>2{addr[NR]=$1} END{i=NR; while(i>2){print addr[i];i-=1}}' population.txt
```

## 模式

```shell
# 模式汇总 
# 1. BEGIN{ statements}
# 在输入被读取之前, statements 执行一次.
# 2. END{ statements}
# 当所有输入被读取完毕之后, statements 执行一次.
# 3. expression{ statements}
# 每碰到一个使 expression 为真的输入行, statements 就执行. expression 为真指的是其
# 值非零或非空.
# 4. /regular expression/ { statements}
# 当碰到这样一个输入行时, statements 就执行: 输入行含有一段字符串, 而该字符串可 以被 regular expression 匹配.
# 5. compound pattern { statements}
# 一个复合模式将表达式用 &&(AND), ||(OR), !(NOT), 以及括号组合起来; 当 com-
# pound pattern 为真时, statements 执行.
# 6. pattern1, pattern2 { statements}
# 一个范围模式匹配多个输入行, 这些输入行从匹配 pattern1 的行开始, 到匹配 pattern2 的行结束 (包括这两行), 对这其中的每一行执行 statements.a
# BEGIN 与 END 不与其他模式组合. 一个范围模式不能是其他模式的一部分. BEGIN 与 END 是唯一两个不能省略动作的模式.
# pattern1 与 pattern2 可以匹配同一行. — 译者注
```

```shell
# 字符串匹配模式 (string-matching pattern) 测试一个字符串是否包含一段可以被正则 表达式匹配的子字符串
# 正则表达式
```

## 动作

一个动作就是一个语句序列, 语句之间用分号或换行符分开

```shell

```

## 正则表达式

```shell
###字符串匹配模式
## 一个字符串匹配模式测试一个字符串是否包含一段可以被正则 表达式匹配的子字符串.
# 最简单的正则表达式是仅由数字与字母组成的字符串, 就像 Asia, 它匹配的就是它本身. 为了将一个正则表达式切换成一个模式, 只需要用一对斜杠包围起来即可:
# /regexpr/
# expression ~ /regexpr/ 
# expression !~ /regexpr/
# 这个模式匹配那些含有子字符串 Asia 的输入行, 例如 Asia, Asian, 或 Pan-Asiatic

###正则表达式验证
awk '/f.n/' <<< `echo -e "cat\nbat\nfun\nfin\nfan"`

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

### AWK常见的内建变量

| 内建变量 | 补充默认值 含义                |
| -------- | ------------------------------ |
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

```shell
### 参考 表2.4: 模式
### 表 2.5: 内建变量
### 表 2.6: 内建算术函数
### 表 2.7: 内建字符串函数
```

