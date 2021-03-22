# 										       							    							 						AWK简介

### 前言

我们通常会遇到大量的数据处理工作，比如日志分析、数据清洗、文本过滤、数据统计等。计算机用户经常把大量的时间花费在简单, 机械化的数据处理工作中 — 改变数据格式, 验证数据的有效性, 搜索特定的数据项, 求和, 打印报表等. 这些工作完全可以自动化地完成, 但是如果每碰到一个这样的任务,就用一门标准的编程语言 (比如 C 或 Pascal) 写一个专用的程序来解决它, 未免也太麻烦了。AWK是一个处理文本文件的Linux工具，同时它也是一门编程语言，不过99%的应用场景都可以

### 基本用法

AWK会每次读取一个输入行, 对读取到的每一行, 按顺序检查每一个模式. 对每一个与当前行匹配的模式, 对应的动作就会执行,一个缺失的模式匹配每一个输入行

<img src="https://wetalk-1300208549.cos.ap-nanjing.myqcloud.com/drawImage/awk%E6%B5%81%E7%A8%8B%E5%9B%BE.png" alt="AWK" style="zoom: 67%;" />

```shell
### AWK会每次读取一个输入行, 对读取到的每一行, 按顺序检查每一个模式. 对每一个与当前行匹配的模式, 对应的动作就会执行,一个缺失的模式匹配每一个输入行,
### awk 的基本操作是在由输入行组成的序列中, 陆续地扫描每一行, 搜索可以被模式 匹配 (match) 的行. “匹配” 的精确含义依赖于问题中的模式
awk 'program' input files
awk  'pattern {action}'  input files
```

awk 自动地扫描输入文件, 并把每一个输入行切分成字段。许多工作都是自动完成的：包括输入, 字段分割, 存储管理, 初始化。与传统语言编写的程序相比, awk 程序简短得多. Awk 最常用的用途就是前面提到的那些工作. 因为 awk 程序一般都很短, 所以人们经常这样使用它: 通过键盘在命令行中输入程序代码 (只有一两行), 执行, 然后把代码丢弃。实际上, awk 是一个通用编程工具, 许多专用工具都可以用它来替代。

```shell
### awk 的基本操作是在由输入行组成的序列中, 陆续地扫描每一行, 搜索可以被模式 匹配 (match) 的行. “匹配” 的精确含义依赖于问题中的模式
awk  'pattern {action}'  input files

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
```

### 模式

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

### 动作

一个动作就是一个语句序列, 语句之间用分号或换行符分开

```shell

```

### 输出到文件

### 输出到管道

### 命令行参数

### 内建函数

### 内建变量

### 附录

```shell
### 参考 表2.4: 模式
### 表 2.5: 内建变量
### 表 2.6: 内建算术函数
### 表 2.7: 内建字符串函数
```

