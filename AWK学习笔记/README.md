# 										       							    							 					    						 									AWK

## 前言

哈喽，大家好，今天给大家介绍一个非常之牛逼的Unix工具  **AWK** ！！！

AWK是1977年贝尔实验室的三个哥们( [Alfred Aho](http://en.wikipedia.org/wiki/Alfred_Aho)、[Peter Weinberger](http://en.wikipedia.org/wiki/Peter_J._Weinberger)、 [Brian Kernighan](http://en.wikipedia.org/wiki/Brian_Kernighan) )搞出来的文本分析工具，这三个哥们的首字母拼起来就是AWK的名字了。

AWK虽然是上个世纪的产物，但是它的简洁和丰富的功能可以称之为**神器**！！！

它处理文本就像其他语言处理数值一样方便，所以经常被应用在文本处理领域，比如日志分析、数据清洗、文本过滤、数据统计等。

同时AWK也是一门编程语言，不过它的命令行用法就可以满足大多数的应用场景，我们通常可以使用一行AWK命令完成一个脚本的任务！！！

AWK所适用的文本处理通常都有一些共同&显著的特点：

**(1)     输入数据格式统一** 

比如日志，为了对日志进行上报、监控、统计分析，我们通常会采用一些分割手段来记录日志 (或者json等易于统计的格式)。

例如下面采用"|"来分割日志。

```shell
# 日志格式：{服务}|{日期}|{业务}|{请求URL}|{返回状态}|{请求耗时}|{请求参数}|{返回参数}...
```

比如CSV文件，采用","来分割。

```shell
# CSV格式：field1,field2,field3...
```

如果输入数据不是固定格式，通常会使用sed、grep等工具来过滤、清洗为awk可以处理的形式。

**(2)    每一列代表固定含义，便于数据分析**

通常统计文件每一行的相同列类型相同，如果每一列含义不同，那就失去了数据分析的意义。

比如本文演示数据，第一列表示地区，第二列表示总人口等。

> 演示数据来源于国家统计局。
>
> [各地区户口登记地在外乡镇街道的人口状况](https://github.com/jinjideyuyan/wetalk/blob/main/AWK%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/population.txt)
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

本文及相关数据集均上传至GitHub：https://github.com/jinjideyuyan/wetalk/tree/main/AWK%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0



**Let's  start ！！！** 



## 基本用法

一个AWK程序的组成非常简单，它的核心内容是：一个或多个 "模式–动作" 语句序列。

"模式–动作"  序列用单引号包起来，动作放在花括号里，在传入输入文件即可。

```shell
### 一个 模式-动作
awk 'pattern {action}' input_files

### 多个 模式-动作
awk 'pattern1 {action1} pattern2 {action2} pattern3 {action3} ...' input_files
```

AWK会每次读取一个输入行，对读取到的每一行，按顺序检查每一个模式。

如果当前行符合模式，则执行对应动作。

所以AWK的工作原理就是按顺序执行模式然后执行动作，可以想象到AWK伪代码大概长这样，我猜的(\*^_^\*)。

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

AWK在自动扫描输入文件的同时, 也会按照分隔符(默认空格/Tab)把每一个输入行切分成字段。

其中$0表示整行，$1,$2...$n对应表示第一列，第二列...第N列。

大致的流程图如下：

<img src="https://wetalk-1300208549.cos.ap-nanjing.myqcloud.com/drawImage/awk%E6%B5%81%E7%A8%8B%E5%9B%BE.png" alt="AWK" style="zoom: 67%;" />

大部分的工作都是AWK自动完成的：包括按行输入，字段分割，存储管理，初始化。

所以我们只需要给出 "模式–动作" 序列就可以完成对文件的操作！！！

来个 Hello World 吧，输出 "hello"  和 整行 ($0)。

> print函数使用逗号分隔不同的参数，打印结果用空格符分隔，并且会自动换行。(类似于各大语言println函数)。
>
> 不写模式就是匹配所有行。

```shell
$ awk '{print "hello",$0}' population.txt|head -n 5
hello 地区      合计      本县/市/区  本省其他县/市/区     省外
hello 全国    260937942   90372599     84689006       85876337
hello 北京    10498288    1582574      1871181        7044533
hello 天津    4952225     1095282      865442         2991501
hello 河北    8297279     4263957      2628649        1404673
```

AWK提供了很多有用的内置变量，如：

NR  (Number Of Record) ：表示读取到的记录数，即当前行号。

FILENAME ：表示当前输入的文件名。

NF (Number Of Field) ：表示当前记录的字段个数，即总共多少列。

我们通常用 $NF 提取当前行的最后一列。

如下例子所示，总共有5列，$NF代表的就是第五列的值，等价于$5，$(NF-1)表示倒数第二列的值。

```shell
$ awk '{print FILENAME,NR,$1,$3,NF,$NF}' population.txt|head -n 5
population.txt 1 地区 本县/市/区 5 省外
population.txt 2 全国 90372599 5 85876337
population.txt 3 北京 1582574 5 7044533
population.txt 4 天津 1095282 5 2991501
population.txt 5 河北 4263957 5 1404673
```

常见的内建变量可以去附录查阅：[常见的内建变量](#常见的内建变量) 。

AWK也提供了格式化输出函数，跟C语言的printf用法一样。

```shell
$ awk '{printf "%s的外地总人口有:%d,省外人口有:%0.2f\n",$1,$2,$NF}' population.txt|tail -n 5
陕西的外地总人口有:5894416,省外人口有:974362.00
甘肃的外地总人口有:3112722,省外人口有:432833.00
青海的外地总人口有:1140954,省外人口有:318435.00
宁夏的外地总人口有:1534482,省外人口有:368451.00
新疆的外地总人口有:4276951,省外人口有:1791642.00
```

格式化规则可以参考：https://www.gnu.org/software/gawk/manual/html_node/Control-Letters.html 。



## 模式过滤

上面介绍了动作的使用，动作通常用来输出展示。

模式用来过滤我们想要的记录。

如下筛选（行号>1 且 第二列大于11074525）的行。

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

AWK的字符串拼接跟shell一样简单粗暴，不需要使用任何运算符。

将两个字符串并排放在一起就能实现拼接。

```shell
$ awk 'NR>1 {print NR,"开始_"$1"_结束"}' population.txt|head -n 5
2 开始_全国_结束
3 开始_北京_结束
4 开始_天津_结束
5 开始_河北_结束
6 开始_山西_结束
```

AWK还提供了很多有用的内置函数。

length(s)：用来计算字符串s 中字符的个数。

```shell
### 我的系统编码&文件编码均为UTF-8
$ awk 'length($1) > 6 {print $1,"占用长度：",length($1)}' population.txt
内蒙古 占用长度： 9
黑龙江 占用长度： 9
```

substr(s,p)：求字符串s的子串，从位置p开始到末尾。

```shell
$ awk '{print $1,substr($1,4)}' population.txt|head -n 5
地区 区
全国 国
北京 京
天津 津
河北 北
```

常见的内建函数可以去附录查阅：[常见的内建函数](#常见的内建函数) 。

AWK还提供了一些特殊的模式，比如 BEGIN 和 END。这两个模式不匹配任何输入行。

当 awk读取数据前，BEGIN 的语句开始执行，通常用于初始化。

如下我们可以用BEGIN来给输出打印一个表头。

```shell
### 多个 "模式-动作" 并排写就行。
$ awk 'BEGIN{print "AREA TOTAL LOCAL OTHER OUTLAND"} NR>2{print}' population.txt|head -n 5
AREA TOTAL LOCAL OTHER OUTLAND
北京    10498288    1582574      1871181        7044533
天津    4952225     1095282      865442         2991501
河北    8297279     4263957      2628649        1404673
山西    6764665     3643627      2189385        931653
```

当所有输入行被处理完毕，END 的语句开始执行。通常用来收尾。

如下我们可以统计一下第二列大于262005的国家，并在END进行打印。

```shell
$ awk 'NR>2 && $2>262005{count += 1} END{print count"个大于262005的国家"}' population.txt
30个大于262005的国家
```

同一个动作里的多个语句之间使用分号或者换行进行分割。

如下在BEGIN的动作中先指定输出分隔符，接着打印表头。

> OFS (Output Formmat Separate) 也是一个内建变量：指定输出字段分割符。
>
> 如下指定输出时字段采用逗号进行分割

```shell
$ awk 'BEGIN{OFS=",";print "AREA,TOTAL,LOCAL,OTHER,OUTLAND"} NR>2{print $1,$2,$3,$4,$5}' population.txt|head -n 5
AREA,TOTAL,LOCAL,OTHER,OUTLAND
北京,10498288,1582574,1871181,7044533
天津,4952225,1095282,865442,2991501
河北,8297279,4263957,2628649,1404673
山西,6764665,3643627,2189385,931653
```

AWK提供了范围模式可以根据一个区间来匹配多个输入行。

范围模式由两个被逗号分开的模式组成。

```shell
awk 'pattern1,pattern2 {action}' input_file
```

AWK从符合 pattern1 的行开始，到符合 pattern2 的行结束 (包括这两行)，对这其中的每一行执行action。

如下提取第五行到第十行之间地区的数据。

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

前文提到了AWK也是一门编程语言，所以它支持很多编程语言特性，与C语言使用类似。

比如流程控制语句 if-else 、循环(for，while)。

比如数据结构数组等。

它们只能用在动作里。

如下示例使用if-else统计第二列大于4462177 和小于4462177的分别有多少行。

```shell
$ awk 'NR>2{if($2>4462177) more+=1; else less+=1} END{print "more:",more,"less:",less}' population.txt
more: 24 less: 7
```

上面这个例子也可以拆分成多个"模式-动作"来实现。

```shell
$ awk 'NR>2 && $2>4462177{more+=1} NR>2 && $2<=4462177{less+=1} END{print "more:",more,"less:",less}' population.txt
more: 24 less: 7
```

再来看个for循环的例子，打印AWK的命令行参数。

命令行参数在输入文件后追加就可以传入。

```shell
$ awk 'BEGIN {for(i=0;i<ARGC;i++) printf "%s\t",ARGV[i]; print ""}' population.txt abc def cdg
awk	population.txt	abc	def	cdg
```

ARGC和ARGV也是AWK的内建变量，跟C语言的参数结构差不多。

ARGC：命令行参数的个数。

ARGV：命令行参数数组。

```c
// 等价于C语言
int main(int argc, char *argv[])
```

AWK也支持使用数组进行数据存储。

如下示例将对输入行进行倒序输出。

```shell
$ awk 'NR>2{addr[NR]=$1} END{i=NR; while(i>2){print i,addr[i];i-=1}}' population.txt|head -n 5
33 新疆
32 宁夏
31 青海
30 甘肃
29 陕西
```



## 正则表达式

AWK 提供了对正则表达式的支持，正则表达式放在一对斜杠里：/regexpr/ 。

AWK使用 "\~" 符号表示字符串匹配，"!\~" 符号表示不匹配。

所以我们可以在模式中判断一个字符串是否匹配一个正则表达式。

如下示例对  第一列含有  “北”  且第二列不包含  “88”  的行 进行打印。

```shell
$ awk '$1 ~ /北/ {print}' population.txt
北京    10498288    1582574      1871181        7044533
河北    8297279     4263957      2628649        1404673
湖北    9250228     4445565      3791051        1013612

$ awk '$1 ~ /北/ && $2 !~ /88/ {print}' population.txt
河北    8297279     4263957      2628649        1404673
湖北    9250228     4445565      3791051        1013612
```

如果判断整行是否匹配，可以省略 "~" 的左值，如下所示。

```shell
###  /regexpr/ 等价于 $0  ~ /regexpr/
### !/regexpr/ 等价于 $0 !~ /regexpr/
$ awk '!/西/ && /88/ {print}' population.txt
北京    10498288    1582574      1871181        7044533
内蒙古  7170889     2732591      2994117         1444181
福建    11074525    3162036      3598887        4313602
湖南    7898815     4170436      3003397        724982
海南    1843430     586432       668535         588463
青海    1140954     351988       470531         318435
```

正则表达式的语法细节本文不过多说明。

使用  [here documents](http://zh.wikipedia.org/wiki/Here文档)   快速验证几个例子。

```shell
### 匹配小写字母开头的字符串
$ awk '/^[a-z]/' <<< "`echo -e "apple333\n1999fds\nhaode3232\n4343...\nhaoya328"`"
apple333
haode3232
haoya328

### 验证是否是11位国内手机号码
$ awk '/^1[3584][0-9]{9}$/' <<< "`echo -e "18894465939\n1364483882\n13644838825\n23443243432\n1334funny"`"
18894465939
13644838825
```



## 进阶用法

接下来换个内容丰富的数据集来演示。

以下是    [豆瓣电影评分Top250](https://github.com/jinjideyuyan/wetalk/blob/main/AWK%E5%AD%A6%E4%B9%A0%E7%AC%94%E8%AE%B0/douban_top250.csv)    的 CSV数据集。

```shell
### 数据格式：排行,电影名,评分,年份,导演,标签,星级
$ cat douban_top250.csv|head -n 5
rank,title,rating_num,year,director,quote,star
1,肖申克的救赎,9.7,1994,弗兰克·德拉邦特 Frank Darabont,希望让人自由。,2304569
2,霸王别姬,9.6,1993,陈凯歌 Kaige Chen,风华绝代。,1709820
3,阿甘正传,9.5,1994,罗伯特·泽米吉斯 Robert Zemeckis,一部美国近现代史。,1733112
4,这个杀手不太冷,9.4,1994,吕克·贝松 Luc Besson,怪蜀黍和小萝莉不得不说的故事。,1913405
```

AWK默认按照  空格/Tab 对每一个输入行进行切分。

我们可以使用  -F 参数进行指定分隔符，也支持多个分隔符。

```shell
### 指定分隔符
$ awk -F',' '{print $1,$2,$3}' douban_top250.csv|head -n 3
rank title rating_num
1 肖申克的救赎 9.7
2 霸王别姬 9.6

### 多个分隔符 可以看到评分被切分了
$ awk -F'[,.]' '{print $1,$2,$3,$4}' douban_top250.csv|head -n 3
rank title rating_num year
1 肖申克的救赎 9 7
2 霸王别姬 9 6
```

AWK支持使用shell重定向运算符  >  和  >>  ，可以对文件进行拆分。

评分9以上的另存为douban_more_9.csv，评分9以下的为douban_less_9.csv。

```shell
$ awk -F',' 'NR>1 && $3>=9 {print $0 > "douban_more_9.csv"} NR >1 && $3<9 {print $0 > "douban_less_9.csv"}' douban_top250.csv

$ cat douban_less_9.csv|head -n 5
61,让子弹飞,8.9,2010,姜文 Wen Jiang,你给我翻译翻译，神马叫做TMD的惊喜。,1294845
63,绿皮书,8.9,2018,彼得·法雷里 Peter Farrelly,去除成见，需要勇气。,1245160
65,本杰明·巴顿奇事,8.9,2008,大卫·芬奇 David Fincher,在时间之河里感受溺水之苦。,788815
68,看不见的客人,8.8,2016,奥里奥尔·保罗 Oriol Paulo,你以为你以为的就是你以为的。,965038
69,西西里的美丽传说,8.9,2000,朱塞佩·托纳多雷 Giuseppe Tornatore,美丽无罪。,781719

$ cat douban_more_9.csv|head -n 5
1,肖申克的救赎,9.7,1994,弗兰克·德拉邦特 Frank Darabont,希望让人自由。,2304569
2,霸王别姬,9.6,1993,陈凯歌 Kaige Chen,风华绝代。,1709820
3,阿甘正传,9.5,1994,罗伯特·泽米吉斯 Robert Zemeckis,一部美国近现代史。,1733112
4,这个杀手不太冷,9.4,1994,吕克·贝松 Luc Besson,怪蜀黍和小萝莉不得不说的故事。,1913405
5,泰坦尼克号,9.4,1997,詹姆斯·卡梅隆 James Cameron,失去的才是永恒的。,1695453
```

AWK也支持三目表达式，上面语句等价于下面。

```shell
$ awk -F',' 'NR>1 {print $0 > ($3>=9 ? "douban_more_9.csv":"douban_less_9.csv")}' douban_top250.csv
```

同时我们可以对文件进行批量处理。

比如下面提取第二列和最后一列进行MySQL入库。

这在数据量大的时候很管用。

比如几万、几亿的数据可以快速转化为SQL语句。

```shell
### 注意 双引号只需要斜杠转义：\"
### 单引号除了斜杠转义还要用''包围起来： '\''
$ awk -F',' 'NR>1 {print "insert into `movie` (name,star) values ('\''"$2"'\'','\''"$NF"'\'');" > "movie.sql"}' douban_top250.csv

cat movie.sql|head -n 5
insert into `movie` (name,star) values ('肖申克的救赎','2304569');
insert into `movie` (name,star) values ('霸王别姬','1709820');
insert into `movie` (name,star) values ('阿甘正传','1733112');
insert into `movie` (name,star) values ('这个杀手不太冷','1913405');
insert into `movie` (name,star) values ('泰坦尼克号','1695453');
```

统计Top250里各个评分所占数量。

```shell
$ awk -F',' 'NR>1{count[$3]++} END{for(i in count) print "豆瓣电影Top250里评分",i,"的电影有",count[i],"个"}' douban_top250.csv
豆瓣电影Top250里评分 9.0 的电影有 20 个
豆瓣电影Top250里评分 9.1 的电影有 23 个
豆瓣电影Top250里评分 9.2 的电影有 19 个
豆瓣电影Top250里评分 9.3 的电影有 17 个
豆瓣电影Top250里评分 9.4 的电影有 6 个
豆瓣电影Top250里评分 9.5 的电影有 4 个
豆瓣电影Top250里评分 9.6 的电影有 2 个
豆瓣电影Top250里评分 9.7 的电影有 1 个
豆瓣电影Top250里评分 8.3 的电影有 1 个
豆瓣电影Top250里评分 8.4 的电影有 3 个
豆瓣电影Top250里评分 8.5 的电影有 11 个
豆瓣电影Top250里评分 8.6 的电影有 25 个
豆瓣电影Top250里评分 8.7 的电影有 42 个
豆瓣电影Top250里评分 8.8 的电影有 38 个
豆瓣电影Top250里评分 8.9 的电影有 38 个
```

找出Top250里拍过多个电影的导演。

```shell
$ awk -F',' 'NR>1{print $5}' douban_top250.csv|sort|uniq -c|sort -rn|head -n 5
   8 宫崎骏 Hayao Miyazaki
   7 克里斯托弗·诺兰 Christopher Nolan
   6 史蒂文·斯皮尔伯格 Steven Spielberg
   5 王家卫 Kar Wai Wong
   5 李安 Ang Lee
```

找出Top250里即拍过评分9以上 又拍过9分以下的导演。

即求 douban_less_9.csv 和 douban_more_9.csv 两个文件的交集。

```shell
$ awk -F',' 'NR==FNR{map[$5]++} NR>FNR{if($5 in map)print $5}' douban_less_9.csv douban_more_9.csv|sort|uniq -c
   1 Chris Columbus
   2 李安 Ang Lee
   1 姜文 Wen Jiang
   1 大卫·芬奇 David Fincher
   1 罗伯·莱纳 Rob Reiner
   1 刘伟强 / 麦兆辉
   1 黑泽明 Akira Kurosawa
   1 杨德昌 Edward Yang
   4 宫崎骏 Hayao Miyazaki
   2 刘镇伟 Jeffrey Lau
   1 詹姆斯·卡梅隆 James Cameron
   2 朱塞佩·托纳多雷 Giuseppe Tornatore
   3 史蒂文·斯皮尔伯格 Steven Spielberg
   1 是枝裕和 Hirokazu Koreeda
   2 弗朗西斯·福特·科波拉 Francis Ford Coppola
   3 克里斯托弗·诺兰 Christopher Nolan
```

数组的key可以字符串拼接，这样可以间接实现二维数组的逻辑。

```shell
$ awk -F',' 'NR==2,NR==5{a[$1"-"$2]=$3} END {for (i in a) print i, a[i]}' douban_top250.csv
1-肖申克的救赎 9.7
3-阿甘正传 9.5
4-这个杀手不太冷 9.4
2-霸王别姬 9.6
```

数据统计的大部分需求都可以用AWK快速的实现。

比如：过滤、统计、聚合、并集、交集、差集等。

快试试吧！

## 附录

### 常见的内建变量

| 内建变量 | 补充默认值 含义                                              |
| -------- | :----------------------------------------------------------- |
| NF       | 当前记录的字段个数，即总共多少列                             |
| NR       | 读取到的记录数，即当前行号                                   |
| FNR      | 当前输入文件的记录个数，区别于NR，NR表示整体的记录数，FNR表示当前文件 |
| ARGC     | 命令行参数的个数                                             |
| ARGV     | 命令行参数数组                                               |
| FS       | 指定输入行的字段分割符                                       |
| FILENAME | 当前输入文件名                                               |
| OFS      | 指定输出字段分割符                                           |
| ORS      | 指定输出的记录分割符   默认是换行 "\n"                       |
| RS       | 指定输入行的记录分割符   默认是换行 "\n"                     |

### 常见的内建函数

| 函数                      | 含义                                                |
| ------------------------- | --------------------------------------------------- |
| length(s)                 | 字符串s长度                                         |
| tolower(s)                | 把字符串转为小写                                    |
| substr(s, p)              | 字符串s的子串，从位置p开始到末尾                    |
| split(s, a, fs)           | 把字符串s根据fs进行分割，存到数组a中                |
| sprintf(fmt,expr-list)    | 跟C语言sprintf一样，用于字符串格式化                |
| int(x)                    | 取x 的整数部分                                      |
| sin(x) / cos(x) / sqrt(x) | 正弦 / 余弦 / 平方根                                |
| rand()                    | 随机数  配合 srand(x)使用  x 是 rand() 的随机数种子 |

