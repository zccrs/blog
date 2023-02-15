---
title: "用Flex和Bison自制“五仁”语言"
date: 2016-05-04T14:00:00+08:00
draft: false
authors: ["zccrs"]
tags: ["Bison", "Flex"]
---

## 先介绍一下“五仁”语言的特性。

* “五仁”官方名称：zScript
* 变量使用关键字"var"进行声明，变量本身无类型

<!--more-->

* 变量声明可放在源码中任何位置，可在任何位置被调用，例如，var a在源码中最后一行，也可以在第一行使用变量a
* 每条语句的结尾必须是';'或换行符(后面准备改成强制使用';'结尾)
* 目前支持的数据类型：int、double、bool、string、list、tuple、object 、undefined
* 函数也是object类型
* string的定义为 '' 或 "" 中所包裹的内容，支持C和JavaScript风格的字符转义
* 支持的运算符： =、+、-、*、/、%、&、|、~、^、&&、||、+=、-=、*=、/=、%=、&=、|=、~=、^=、&&=、||=、!、>、<、=、==、===、!=、!==、++、--
* 支持匿名函数，函数闭包，多值返回
* 支持的分支结构：if else
* 支持的循环结构：goto while for（后面准备加入do while、foreach）
* while for 循环结构中使用break跳出循环，continue立马开始执行下一次循环，break和continue可组合使用
例如：break, break 则可以直接跳出两层循环，组合使用时continue只能放在末尾
* 支持switch case语句，可枚举int bool string undefined
## Flex(Lexical Analyzar)

* Lex是一个产生词法分析器的工具(最早是Eric Emerson Schmidt和Mike Lesk制作）是许多UNIX系统的标准词法分析器（lexical analyzer）产生程序，而且这个工具所作的行为被详列为POSIX标准的一部分。而Flex就是由Vern Paxon实现的Lex
* FLex读进一个代表词法分析器规则的输入字符串流，然后输出C/C++语言的词法分析器源代码。
* wiki：[https://](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[en](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[.wik](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[i](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[p](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[e](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[dia.o](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[r](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[g/w](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[i](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[ki/F](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[l](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[ex](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[_](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[%28lex](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[ical](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[_](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[an](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[a](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[ly](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[s](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[er](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[_g](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[e](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[n](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[erat](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[o](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[r](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)[%29](https://en.wikipedia.org/wiki/Flex_%28lexical_analyser_generator%29)
### Flex的文件结构

* 文件分成三个区块，均以一个只有两个百分比符号（%%）的单行来分隔，如下：
> 定义区域
%%
规则区域
%%
C/C++代码区
* 定义区块是用来定义宏以及导入C写成的头文件所在区块。在这里面也可以写一些C代码，这一些代码会被复制到产生出来的C源代码的开头部分。
* 规则区块是最重要的区块；这里将样式与C的陈述（statement）串连在一起。这一些样式都是正则表达式。当lexer看到输入里面有合乎给定的样式时，则会操作相对应的C代码。这就是flex运作的基础。
* C代码区块的内容会原封不动的照搬到产生出来的C源代码里面。
### 小例子

```plain
%{
 #include 
%}
number [0-9]+
%option noyywrap
%% // 第一部分结束
{number} {
    printf("number: %s\n", yytext);
}
%% // 第二部分结束
int main()
{
    yylex();
    return 0;
}
```
* %option noyywrap用于指明未定义yywrap函数，此函数用于在文件（或输入）的末尾调用。如果函数的返回值是1，就停止解析。否则等待输入后继续解析
注：正则表达式是使用单个字符串来描述、匹配一系列字符串的规则

wiki：[h](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[ttp](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[s://zh.wiki](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[p](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[e](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[di](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[a.](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[o](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[rg/](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[w](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[iki/%E6%](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[AD](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[%A3%E5%88%99%](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[E](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[8%A1%A8%](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[E](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)[8%BE%BE%E5%BC%8F](https://zh.wikipedia.org/wiki/%E6%AD%A3%E5%88%99%E8%A1%A8%E8%BE%BE%E5%BC%8F)

## Bison

* Bison是GNU版本的Yacc（Lexical Analyzar），用于做语法分析，它与Yacc兼容
* 参考手册：[ht](http://www.gnu.org/software/bison/manual/bison.html)[t](http://www.gnu.org/software/bison/manual/bison.html)[p://www.gn](http://www.gnu.org/software/bison/manual/bison.html)[u](http://www.gnu.org/software/bison/manual/bison.html)[.o](http://www.gnu.org/software/bison/manual/bison.html)[r](http://www.gnu.org/software/bison/manual/bison.html)[g](http://www.gnu.org/software/bison/manual/bison.html)[/](http://www.gnu.org/software/bison/manual/bison.html)[so](http://www.gnu.org/software/bison/manual/bison.html)[f](http://www.gnu.org/software/bison/manual/bison.html)[twa](http://www.gnu.org/software/bison/manual/bison.html)[re](http://www.gnu.org/software/bison/manual/bison.html)[/biso](http://www.gnu.org/software/bison/manual/bison.html)[n](http://www.gnu.org/software/bison/manual/bison.html)[/manual/b](http://www.gnu.org/software/bison/manual/bison.html)[i](http://www.gnu.org/software/bison/manual/bison.html)[so](http://www.gnu.org/software/bison/manual/bison.html)[n](http://www.gnu.org/software/bison/manual/bison.html)[.html](http://www.gnu.org/software/bison/manual/bison.html)
中译版：[htt](http://blog.csdn.net/sirouni2003/article/details/400672)[p](http://blog.csdn.net/sirouni2003/article/details/400672)[://bl](http://blog.csdn.net/sirouni2003/article/details/400672)[o](http://blog.csdn.net/sirouni2003/article/details/400672)[g.c](http://blog.csdn.net/sirouni2003/article/details/400672)[s](http://blog.csdn.net/sirouni2003/article/details/400672)[dn.ne](http://blog.csdn.net/sirouni2003/article/details/400672)[t](http://blog.csdn.net/sirouni2003/article/details/400672)[/s](http://blog.csdn.net/sirouni2003/article/details/400672)[i](http://blog.csdn.net/sirouni2003/article/details/400672)[r](http://blog.csdn.net/sirouni2003/article/details/400672)[o](http://blog.csdn.net/sirouni2003/article/details/400672)[u](http://blog.csdn.net/sirouni2003/article/details/400672)[ni](http://blog.csdn.net/sirouni2003/article/details/400672)[2003/](http://blog.csdn.net/sirouni2003/article/details/400672)[ar](http://blog.csdn.net/sirouni2003/article/details/400672)[ticl](http://blog.csdn.net/sirouni2003/article/details/400672)[e](http://blog.csdn.net/sirouni2003/article/details/400672)[/d](http://blog.csdn.net/sirouni2003/article/details/400672)[e](http://blog.csdn.net/sirouni2003/article/details/400672)[tail](http://blog.csdn.net/sirouni2003/article/details/400672)[s](http://blog.csdn.net/sirouni2003/article/details/400672)[/400672](http://blog.csdn.net/sirouni2003/article/details/400672)
* wiki：[h](https://en.wikipedia.org/wiki/GNU_bison)[t](https://en.wikipedia.org/wiki/GNU_bison)[tps://en.](https://en.wikipedia.org/wiki/GNU_bison)[wi](https://en.wikipedia.org/wiki/GNU_bison)[kipe](https://en.wikipedia.org/wiki/GNU_bison)[d](https://en.wikipedia.org/wiki/GNU_bison)[ia.](https://en.wikipedia.org/wiki/GNU_bison)[o](https://en.wikipedia.org/wiki/GNU_bison)[rg/](https://en.wikipedia.org/wiki/GNU_bison)[wi](https://en.wikipedia.org/wiki/GNU_bison)[k](https://en.wikipedia.org/wiki/GNU_bison)[i](https://en.wikipedia.org/wiki/GNU_bison)[/GNU_bis](https://en.wikipedia.org/wiki/GNU_bison)[on](https://en.wikipedia.org/wiki/GNU_bison)
#### Bison的文件结构

* 和Flex一样，也分为三个部分
* 分别是声明、语法规则和代码段，使用%%分开
> 定义区域
%%
规则区域
%%
C/C++代码区
* 第一部分和Flex一样，用于存放终结符的声明、符号优先级和符号类型等定义
* 第二部分用来定义文法规则
* 第三部分也是用来存放C/C++代码
#### 来个小例子

```plain
%{
#include 
#define YYSTYPE int

void yyerror(char const *);
int yylex(void);
%}

%token NUMBER

%debug

%%

start:  { printf("yylval = %d\n", yylval);}
    | NUMBER {
        printf("NUMBER %d\n", yylval);
    }
    start NUMBER {
        printf("start NUMBER %d\n", yylval);
    }
    ;

%%

void yyerror(char const *msg)
{
    printf("error: %s\n", msg);
}

int yylex()
{
    printf("&gt;&gt;&gt;");

    char str[10] = {};

if(scanf("%s\n", str) == EOF)
    return 0;

int number = atoi(str);

printf("%d\n", number);

yylval = number;

return NUMBER;
}

int main()
{
return yyparse();
}
```
* %token用于定义一个终结符
* %debug指定bison在生成的代码中加入debug输出（和bison的-t参数一样）
* yyerror在分析过程中有语法错误时被调用
* yylex用于给分析器返回token（在生成的代码中自动被调用）
注：终结符是指文法规则中不能再被分解的最小单位，非终结符则是相反的概念。

例如上面的文法规则中“start”就是非终结符，NUMBER就是一个终结符。

wiki：[h](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[tt](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[ps://zh.w](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[i](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[k](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[ipe](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[d](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[i](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[a.](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[o](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[rg/wik](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[i](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[/%E7%B5%82%E7%B5%90%](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[E](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[7%AC%](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[A](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[6%](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[E](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[8%88%87%](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[E](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[9%9](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[D](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[%9](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[E](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[%E7%BB%88%E7%B5%90%](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[E](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)[7%AC%A6#](https://zh.wikipedia.org/wiki/%E7%B5%82%E7%B5%90%E7%AC%A6%E8%88%87%E9%9D%9E%E7%BB%88%E7%B5%90%E7%AC%A6#)

## 语法介绍

### 变量的定义

变量使用关键字“var”进行定义。

zScript提供了两种形式的变量定义语句。

1.直接定义的形式，该语句的一般形式为：

```plain
var 标识符;
```
该语句的含义是：定义变量名为“标识符”的变量。例如：
```plain
var a;
```
该语句的作用是：定义一个名为“a”的变量。
2.定义同时给变量初始化赋值，该语句的一般形式为：

```plain
var 标识符 = 表达式;
```
该语句的含义是：定义变量名为“标识符”的变量，且给变量赋值为表达式的结果。例如：
```plain
var a = 1 + 2;
```
该语句的作用是：定义一个名为“a”的变量，且赋值为3。
“var”关键字可同时定义多个变量，每个变量之间使用符号“,”分割。

### 匿名函数的定义

匿名函数定义的一般形式为：

```plain
(形参列表) {
    语句组
}
```
该语句的含义是：生成一个函数对象，例如：
```plain
(a) {
    console.log(a);
}
```
该语句的作用是：定义一个函数，它有一个名为“a”的形参，执行此函数会将a的值打印到屏幕上。
形参列表，可为空、一个、多个，多个形参之间使用“,”分割。

### 数组的定义

数组定义的一般形式为：

```plain
var a = [表达式1, 表达式2, ...];
```
该语句的含义是：生成一个数组，数组元素为中括号中表达式的结果。例如：
```plain
var a = [1, 2, 3];
```
该语句的作用是：定义一个包含三个元素的数组，元素的值分别为1、2、3，然后赋值给变量a；
数组使用“[”“]”定义，中间内容可以为空、一个表达式、多个表达式，多个表达式之间使用“,”隔开。

```plain
a = a[0];
```
取数组元素语法和C语言一样，都是中括号中写入数组下标。
### 选择控制语句

一个选择结构，包括一组或若干组操作，每组操作称为一个分支。通过选择控制语句可以实现选择结构。选择控制语句包括if语句、switch语句及起辅助控制作用的break语句。

If语句用于计算给定的表达式，根据表达式的值是否为假，决定是否执行某一组操作。

Switch语句首先求解一个表达式，然后根据计算结果的值，从哈希表中查询该从哪一组操作开始执行。

Break语句用于switch结构中，用于终止当前switch结构的执行。

#### if语句

zScript提供了两种形式的if语句。

1.单if子句的if语句。该if语句的一般形式为：

```plain
if(表达式)
{
    语句组
}
```
该语句的含义是：只有表达式的值为非零值时，才执行其内部的语句组。例如：
```plain
if ( a &gt; b )
{
    console.log(“hello”);
}
```
该语句的作用是：当a的值大于b的值时（此时，“a>b”的值为真，为非假值），在屏幕上显示“hello”；否则，不显示“hello”。
2.带else子句的if语句。该if语句的一般形式为：

```plain
if ( 表达式 )
{
    语句组1
} else {
    语句组2
}
```
该语句的含义是：当表达式的值为非假时，执行语句组1，而不执行语句组2；否则，即表达式的值为假时，执行语句组2，而不执行语句组1。例如：
```plain
if ( a &gt; b )
{
    console.log(“hello1”);
} else {
    console.log(“hello2”);
}
```
该语句的作用是：若a的值大于b的值（此时“a>b”为真，为非假值），则在屏幕上显示“hello1”，而不显示“hello2”；否则，即表达式的值为假时，显示“hello2”，而不显示“hello1”。
#### switch语句

Switch语句与if语句一样，也可以实现分支选择。但if语句是判断一个表达式的值是否为假，决定是否执行某个分支；而switch语句是计算一个表达式的值，根据计算结果，从哈希表查询从哪个分支开始执行代码。Switch语句的一般形式为：

```plain
switch( 表达式 )
{
    case 常量1：
    语句组1
    case 常量2：
    语句组2
    ...
    case 常量n：
    语句组n
    default:
    语句组 n + 1
}
```
switch语句的执行过程：
* 1.求解“表达式”的值；

* 2.如果“表达式”的值与某个case后面的“常量”的值相同，则从这里开始顺序执行语句。结果switch执行有两种形式：一是遇到break语句为止；二是未遇到break语句，则程序依次执行完所有语句组。

* 3.如果“表达式”的值与任何一个case后面的“常量”的值都不相同，当有default子句时，则执行default后面的语句，如果没有default子句，则结束switch。

其中break的一般形式为

```plain
break;
```
### 循环控制语句

#### while语句

while语句的一般形式为

```plain
while( 表达式 )
{
    循环体
}
```
while语句的执行过程：
* 1.求解小括号中表达式的值。如果表达式的值为真，转第2步；否则转第3步。

* 2.执行循环体，然后转第1步。

* 3.执行while语句后面的语句。

小括号中表达式的值是否为假，决定着循环体是终止还是继续循环。因此，该表达式的值为循环条件。while循环语句的执行特点是，先判断循环条件是否成立，然后决定是否执行循环体。

当while语句的循环体只包含一条语句时，包含该循环体的“{}”可以省略。

#### for语句

在两种循环语句中，for语句最为灵活。for语句的一般形式为：

```plain
for (表达式1; 表达式2; 表达式3)
{
    循环体
}
```
for语句的执行过程：
* 1.求解表达式1的值。

* 2.求解表达式2的值，若其值为假，则结束循环，转第4步；若其值为真，则执行循环体。

* 3.求解表达式3。

* 4.结束循环。

### 对象的定义

对象定义的一般形式为：

```plain
{
    属性名1: 属性的值,
    属性名2: 属性的值,
    ...
    属性名n: 属性的值,
    属性名n+1: 属性的值
}
```
例如：
```plain
var object = {
    name: "张三",
    age: 18
};
```
变量object即是一个对象，它包含两个属性。
注：当对一个对象不存在的属性赋值时会将此属性加入到对象中，例如：

```plain
object.sex = '男';
```
对象object中就会多出一个“sex”属性。
## 推荐

* [GNU](http://www.gnu.org/software/bison/manual/bison.html)[ ](http://www.gnu.org/software/bison/manual/bison.html)[B](http://www.gnu.org/software/bison/manual/bison.html)[i](http://www.gnu.org/software/bison/manual/bison.html)[so](http://www.gnu.org/software/bison/manual/bison.html)[n](http://www.gnu.org/software/bison/manual/bison.html)[手册](http://www.gnu.org/software/bison/manual/bison.html)
* [中译版](http://blog.csdn.net/sirouni2003/article/details/400672#SEC103)
* [zScript项目地址](https://github.com/zccrs/zScript)

