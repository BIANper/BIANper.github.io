---
title: Java 补丁-异常、断言和日志
date: 2019-12-23 19:27:09
tags:
- 异常
- 断言
categories: Java
---
# 异常部分
**概念**
非受查 unchecked 异常：派生于 Error 类或 RuntimeException 类的所有异常，其它的异常称为受查 checked 异常。

**创建异常类**
派生于 Exception 或其子类。应该包含两个构造器：默认构造器和带描述信息的构造器（超类 Throwable 的 toString 方法会打印出详细信息）。
```
class FileFormatException extends IOException {
public FileFormatException() {}
public FileFormatException(String gripe) { super(gripe) };
}
```

**finally 子句**
建议解耦合 try/catch 和 try/finally 语句块，以提高清晰度（且会报告 finaly 子句的错误）。
```
try {
	try {
	}
	finally {
	}
}
catch (Exception e) {
}
```
finally 子句中的 return 语句将会覆盖 try 中 return 的值。

**带资源的 try 语句**
```
try (Resource res = ...; ...) {
	work with res
}
```
可以指定多个资源，try 块正常退出或存在异常时会自动调用 res.close()。catch 子句和 finally 子句将会在关闭资源后执行。

**堆栈轨迹元素**
堆栈轨迹 stack trace：方法调用过程的列表。

# 断言部分
**概念**
```
assert 条件 ;
assert 条件 : 表达式 ;
```
测试期间向代码插入一些检查语句，发布时会自动移走。
对条件进行检测，如果结果是 false 则抛出 AssertionError 异常。第二种还会将表达式转换成消息字符串。

**启用和禁用**
运行程序时用 -ea -da 选项，可以指定某个类或某个包。默认禁用。
程序中使用 `setDefaultAssertionSatus (boolean b)` 等完成。

# 日志部分
占坑