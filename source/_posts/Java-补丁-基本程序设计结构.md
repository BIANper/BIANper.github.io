---
title: Java 补丁-基本程序设计结构
date: 2019-12-01 19:16:02
tags:
- 数据类型
- 控制流程
- 数组
- 运算符
categories: Java
---
# 数据类型部分
**整型**
Java7 开始：允许通过加前缀 0b 写二进制数；允许为数字字面量加下划线：
- 1_000_000
- 0b1111_0000_0100

**浮点型**
由于二进制系统无法精确表示分数 1/10，故浮点数值不适用金融计算，此时应使用 BigDecimal 类。

|   常量   |   数值   |
| :--: | :--: |
|   Double.POSITIVE_INFINITY   |   正无穷   |
|   Double.NEGATIVE_INFINITY   |   负无穷   |
|   Double.NaN   |   非数字   |
判断一个特定值是否为 Double.NaN ：
```
if(x == Double.NaN) // never true
if(Double.isNaN(x)) // √
```

**char 类型**
char 数据类型是一个采用 UTF-16 编码表示 Unicode 码点的代码单元。
Unicode 转义序列会在解析代码前处理：
- "\u0022+\u0022" 会得到 ""+"" 即空串
- // \u00A0 is a newline 会替换成换行符
- // c:\users 会产生语法错误

**枚举类型**
枚举类型的变量只能储存这个类型声明中给定的某个枚举值，或没有设置值 null
```
enum Size { SAMLL, MEDIUM, LARGE };
Size s = Size.SMALL;
```
所有枚举类型都是 Enum 的子类，继承其方法。因为实际上是一个类，故比较时直接用 == 就好，也可以在枚举类型中添加构造器 方法 域。

**相关：大数值**
java.math 包的 BigInteger 和 BigDecimal 类可以处理任意长度数字序列的数值，分别对应整数和浮点数运算。

---

# 运算符部分
**数学函数**
StrictMath 类使用 fdlibm 库实现算法，确保在所有平台上得到相同结果。

**数值类型转换**
二元操作中的类型转换，以 double float long 的顺序进行判断，若都不是则两个操作数都转换成 int 型。
强制类型转换通过截断小数部分将浮点值转换为整型，若要得到最接近整型的数，使用 Math.round 对浮点数进行舍入：
- int newx = (int) Math.round(x); // 返回的是 long 类型

强制转换中若超出目标类型的表示范围，结果会截断成完全不同的值。
不要在 boolean 与任何数值类型间强制类型转换，转换为数值时可以使用条件表达式 boolean ? 1:0

**自增自减和运算符级别**
自增自减中，用于表达式时前缀形式会先完成加减一：
```
int m = 7;
int n = 7;
int a = 2 * ++m; // a is 16, m is 8
int b = 2 * n++; // b is 14, n is 8
```

+= 是右结合运算符，所以 `a += b += c` 等价于 `a += (b += c)`

**位运算符**

|   位运算符   |      |
| :--: | :--: |
|   &   |   and   |
|   &#124;   |   or   |
|   ^   |   xor   |
|   ~   |   not   |

```
int fourthBit = (n & 0b1000) / 0b1000;
```

---

# 字符串部分
**String API**
提取子串： substring 方法。
多个字符串放在一起用一个定界符分隔： String.join 方法。

**不可变字符串**
String 类没有修改字符串的方法，可以先提取再拼接；编译器可以让字符串共享，即复制一个字符串变量，原始字符串与复制的字符串共享相同的字符。
但不要使用 == （此运算符确定是否在同一位置上）检测两个字符串是否相等，虚拟机实际上只有字符串常量是共享的，而 + 或 substring 等操作产生的结果并不共享。使用 equals equalsIgnoreCase 方法。

**构建字符串**
用许多小段字符串构建时，字符串连接的方式效率较低，每次连接字符串都会构建一个新的 String 对象，应该 new 个空的字符串构造器：StringBuilder 类，相对 StringBuffer 类效率更高，但后者允许多线程方式执行操作。

**Null 串**
null 上调用方法会出现错误，故检查字符串 null 和空 `if (str != null && str.length() != 0)` ，空用 equals 也行。

**码点和代码单元**
length 方法返回采用 UTF-16 编码表示的给定字符串所需要的代码单元数量（配合 char 数据类型食用）。大多数常用字符使用一个代码单元表示，辅助字符需要一对代码单元表示。故实际长度应该是码点数量：使用 codePointCount 方法获取。
如字符 𝕆 U+1D546 需要两个代码单元，若使用 `char ch = sentence.charAt(1)` 会返回第二个代码单元而非空格，所以此类不要用 char 类型。charAt 方法返回位置 n 的代码单元。

---

# 输入输出部分
**格式化输出**
```
System.out.printf("%,.2f", 10000.0 / 3.0);
// 3,333.33
```
用于 printf 的转换符：

|   转换符   |   类型   |
| :--: | :--: |
|   d   |   十进制整数   |
|   x   |   十六进制整数   |
|   o   |   八进制整数   |
|   f   |   定点浮点数   |
|   e   |   指数浮点值   |
|   g   |   通用浮点数   |
|   a   |   十六进制浮点数   |
|   s   |   字符串   |
|   c   |   字符   |
|   b   |   布尔   |
|   h   |   散列码   |
|   %   |   百分号   |
|   n   |   平台有关的行分隔符   |

用于 printf 的标志：

|   标志   |   目的   |   举例   |
| :--: | :--: | :--: |
|   +   |   打印正负数符号   |   +333   |
|   空格   |   在正数前加空格   |      |
|   0   |   数字前面补 0   |   00333.3   |
|   -   |   左对齐   |      |
|   (   |   将负数括在括号内   |   (3.3)   |
|   ,   |   添加分组分隔符   |   3,333   |
|   #（f）   |   包含小数点   |   333.   |
|   #   |   添加前缀 0x 或 0   |   0xcafe   |

**文件输入和输出**
通过相对位置指定文件时，是 Java 虚拟机启动路径的相对位置。
记得捕获输入输出异常，以免文件不存在或不能被创建。

读取也是构造 Scanner 对象，路径中要转义反斜杠；写入则构造 PrintWriter 对象。
```
Scanner in = new Scanner(Paths.get("c:\\my\\file"), "UTF-8");
Scanner in = new Scanner("c:\\my\\file"); // 允许但会变成字符数据
```

---

# 控制流程部分
**switch 语句**
Java7 开始，case 标签可以是字符串字面量。
```
String input = ...;
switch (input.toLowerCase()){
    case "yes":
}
```
**中断语句**
带标签 break 语句用于跳出多重循环。标签放在最外层循环之前，并紧跟冒号，break 将跳转到带标签的语句块结尾。只能跳出，不能跳入。
带标签 break 甚至可以用于 if 语句或语句块中：
```
label:
{
    if (condition) break label;
}
```

continue 语句将控制转移到最内层循环的首部，带标签 continue 语句将跳到与标签匹配的循环首部。

---

# 数组部分
数字数组初始化为 0，boolean 数组初始化为 false，对象数组则为 null。数组创建后不能改变大小，需要扩展应选择数组列表 array list。
Java 允许数组长度为 0，与 null 不同。

**for each 循环**
```
for (variable : collection) statement
```
collection 可以是数组或实现了 Iterable 接口的类对象。
打印数组所有值可以用 Arrays.toString 方法，元素被放在 [] 内，用逗号分隔。

**初始化和匿名**
不创建变量初始化一个数组：
```
arr = new int[] {1, 2, 3};
// 即下列语句简写
int[] temp = {1, 2, 3};
arr = temp;
```

**拷贝**
将一个数组变量拷贝给另一个数组变量后，两个变量引用的是同一个数组，即更改在两者生效。若要拷贝到新数组，使用 Arrays.copyOf 方法，这个方法也用于增加数组大小：`arr = Arrays.copyOf(arr, 2 * arr.length)` 。

**命令行参数**
main 的 String[] args 用于存储命令行参数，不包含程序名。

**不规则数组**
Java 实际上只有一维数组，多维数组定义为“数组的数组”。所以可以让两行交换，或是每一行有不同的长度：
```
int[][] odds = new int[NMAX +1][];
//new 一个具有行数的数组
for (int n = 0; n <= NMAX; n++)
    odds[n] = new int[n + 1];
    //然后为每行添加元素
```