---
title: Java 补丁-泛型程序设计
date: 2019-12-24 15:10:20
tags:
- 类型擦除
- 继承
- 通配符
- 泛型约束
categories: Java
---
**类型变量限定**
对类型变量 T 设置限定 `<T extends BoundingType>`。T 的绑定类型可以是类也可以是接口，只是 extends 关键词更接近子类概念，并非表达继承的意思。
一个类型变量或通配符可以有多个限定，限定类型用 & 分隔，类型变量用逗号分隔。但限定中至多有一个类，并是限定列表的第一个。

**继承规则**
Pair<Manager> 不是 Pair<Employee> 的子类，继承不延续进泛型类。
泛型类可以扩展或实现其他的泛型类。

# 虚拟机部分
**类型擦除**
虚拟机没有泛型类型对象，无论何时定义泛型类型都会自动提供相应的原始类型，即擦除类型变量并替换成限定类型（用第一个限定的类型变量替换，无限定的用 Object）。

**翻译泛型表达式**
程序调用泛型方法时，擦除返回类型插入强制类型转换。存取泛型域时也会插入强制类型转换。
```
Pair<Employee> buddies = ... ;
Employeee buddy = buddies.gettFirst();
```
编译器将方法调用翻译成两条虚拟机指令：
- 对原始方法 Pair.getFirst 的调用。
- 将返回的 Object 类型强制转换为 Employee 类型。

**翻译泛型方法**
```
class Father<T> {
	public void fun(T x) {
	}
}
class Son extends Father<String> {
	public void fun(String x) {
	}
}
```
Father 类和 Son 类经过类型擦除后变成：
```
class Father<Object> {
	public void fun(Object x) {
	}
}
class Son extends Father {
	public void fun(String x) {
	}
}
```
很显然，子类 Son 是想覆盖 Father 的 fun 方法，但是却出现了问题。
当出现：
```
Father<String> f = new Son();
f.fun(x);
```
变量 f 声明为类型 Father<String> 而此类型的方法是 fun(Object)，虚拟机用 f 引用的对象调用此方法，即 Son 的 fun(Object)，这个方法是虚拟机合成的桥方法，会调用 Son 的 fun(String) 方法。
总之就是保证了不会调用之前被覆盖的超类方法。

对于 get 方法等无参数的，子类会出现两个方法名相同，都无参数的方法：
```
String get() {}
Object get() {}
```
在编写中是不合法的，但虚拟机用参数类型和_返回类型_确定一个方法，故能正确处理。

# 约束与局限部分
**不能用基本类型实例化类型参数**
没有诸如 <double> 而应该是 <Double>，因为类型擦除后 Object 无法存储基本类型。

**类型查询的是原始类型**
虚拟机中的对象总是一个非泛型类型，比如 `if (a instanceof Pair<String>)` 仅仅测试 a 是否是任意类型的 Pair。getClass 方法总是返回原始类型，强制类型转换将 warning。
```
Pair<String> stringPair = ... ;
Pair<Employee> employeePair = ... ;
if (stringPair.getClass() == employeePair.getClass()) // true
```

**不能创建泛型数组** 1.8 中存疑
类型擦除后使检查元素类型的机制无效，所以不允许创建泛型数组。声明泛型数组是合法的，但不能通过创建泛型数组来对其初始化。使用类似 ArrayList:ArrayList<Pair<String>>。

**Varargs 警告**
```
public static <T> void addAll(Collection<T> coll, T... ts) {
	for (t : ts) coll.add(t);
}
```
当：
```
Collection<Pair<String>> table = ... ;
Pair<String> Pair1 = ... ;
Pair<String> Pair2 = ... ;
addAll (table, pair1, pair2);
```
虚拟机不得不建立 Pair<String > 数组，违反了上一条。但这种情况规则有所放松只会得到警告，用 @SafeVarargs 标注来抑制。

**不能实例化类型变量**
诸如 `new T(...)` `T.class` 是非法的。类型擦除将 T 改变成 Object 从而违反了本意。
通过反射调用 Class.newInstance 方法来构造泛型对象，例：
```
public static <T> Pair<T> makePair(Class<T> cl) {
	try { return new Pair<>(cl.newInstance(), cl.newInstance()); }
	catch (Exception ex) { return null; }
}

Pair<String> p = Pair.makePair(String.class);
```
Class 类本身是泛型，例 `String.calss` 是 `Class<String>` 的唯一实例，makePair 以此推断 Pair 类型。

**泛型类的静态域和方法无效**
不能在静态域或方法中使用泛型，经过类型擦除后只有一个非泛型域，会引起歧义。

**不能抛出或捕获泛型类的实例**
泛型类扩展 Throwable 都是不合法的，catch 子句中不能使用类型变量。
```
public class Problem<T> extends Exception {} // Error

catch (T e) {} //Error
```
在异常规范中使用是允许的：
```
public static <T extends Throwable> void doWork(T t) throws T {}
```

**擦除后的冲突**
当：
```
public class Pair<T> {
	public boolean equals(T value) { return first.equals(value) && second.equals(value); }
	...
}
```
比如对于 Pai<String> 有两个 equals 方法：
- boolean equals(String) 由 Pair<T> 定义
- boolean equals(Object) 继承自 Object

方法擦除 `boolean equals(T)` 后即 `boolean equals(Object)` 覆盖了继承的方法。

一个类或类型变量不能同时成为两个接口类型的子类，这两个接口是同一接口的不同参数化（可能与桥方法产生冲突）：
```
class Employee implements Comparable<Employee> { ... }
class mangeer extends Employee implements Comparable<Manager> { ... }
// Error
```

# 通配符部分
**子类型限定**
例：Pair<Manager> 是 Pair<? extends Employee> 的子类型，即将 Pair 的类型限定于 Employee 及其子类。通配符的引用不会引起破坏：
```
Pair<Manager> managerBuddies = new Pair<>(ceo, cfo);
Pair<? extends Employee> wildcardBuddies = managerBuddies; // OK
wildcardBuddies.setFirst(lowlyEmployee); // error
```
因为对于方法而言：
```
? extends Employee getFirst() {}
void setFirst(? extends Employee) {}
```
编译器只知道 setFirst 需要某个 Employee 的子类型，不知道具体是什么类型，拒绝任何特定的类型。而对于 getFrist，将返回值赋给 Employee 的引用是合法的。

**超类型限定**
诸如 `<? super Manger>` 。
与子类型限定相反，可以为方法提供参数但不能使用返回值。

带有超类型限定的通配符可以向泛型对象写入，带有子类型限定的通配符可以从泛型读取。

**无限定通配符**
get 的返回值只能赋给一个 Object 。setFirst 只允许 null 值（子类型限定同）。

# 反射部分
对象是泛型类实例时，因为擦除，反射得不到太多信息。
详见 API