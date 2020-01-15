---
title: Java 补丁-接口、lambda 和内部类
date: 2019-12-23 19:26:36
tags:
- 接口
- 内部类
- lambda
- 泛型
categories: Java
---
# 接口部分
**特性**
可以使用 instanceof 检查一个对象是否实现某个接口：
```
if (anObject instanceof Comparable) { ... }
```
接口中不能包含实例域或静态方法（ Java 8 支持静态方法），允许常量。接口中的方法默认为 public，域默认为 public static final。
 
 **default 默认方法**
与超类冲突：超类的方法和接口的默认方法冲突时，超类优先。
与接口冲突：只要有一个接口有默认方法，就需要提供一个实现覆盖掉。若接口们都没有提供默认方法，则不存在冲突 - 全部实现或不实现（使自身抽象）。
> 所以不要用接口的默认方法重新定义 Object 类中的方法如 equals 等，因为超类优先规则默认方法会永远被忽略。

**clone 方法**
copy 一个新对象而不是引用。clone 方法是 Objects 的一个 protected 方法，子类只能克隆它自己的对象，会产生问题：若拷贝的是相同子对象的引用，原对象和克隆对象仍会共享一些信息。默认的拷贝是浅拷贝，若共享的子对象不可变（如 String 类），在生命周期里没有更改器改变它，也没有方法生成它的引用，则是安全的。

若默认 clone 方法满足需求，需要实现 Cloneable 接口（标记接口），调用 `super.clone()`；若要在可变的子对象上调用 clone 进行修补，则还重新定义一个 public 的 clone 方法。

# lambda 表达式部分 *
**规范**
一个可传递的代码块，以及其变量规范。即使 lambda 表达式没有参数仍然要提供空括号：`() -> { ... }` ；如果可以推导出参数类型则可以忽略其类型：`Comparator<S
tring> comp = (first, second) -> ... ;` ；如果方法只有一个参数，且可以推导出类型，可以省略小括号：`ActionListener listener = event -> ... ;` 。
lambda 表达式的返回类型会由上下文推导得出；只在某些分支返回一个值是不合法的。

**函数式接口**
对于只有一个抽象方法的接口，需要其对象时，可以提供一个 lambda 表达式。Java 中对 lambda 表达式所能做的只是转换为函数式接口。

**引用方法**
使用现有方法代替要传递的内容，用 :: 分隔方法名和对象名，允许使用 this 和 super。
- object::instanceMethod
- Class::staticMethod
- Class::instanceMethod

前两种情况中方法引用等价于提供方法参数的 lambda 表达式，如 `System.out::println` 等价于 `x -> System.out.println(x)` 。第三种情况第一个参数会成为方法的目标，如 `String::compareToIgnoreCase` 等价于 `(x,y) -> x.compareToIgnoreCase(y)` 。

**构造器引用**
构造器引用类似于方法引用，不过方法名为 new。编译器通过上下文选择构造器。
可以用数组类型建立构造器引用，如 `int[]::new` 有一个数组长度参数，等价于 `x -> new int[x]`。但无法构造泛型类型 T 的数组，通过 Stream 接口的 toArray 方法调用构造器获得某类型的数组。

**变量作用域**
lambda 表达式可以捕获外围方法或类中变量的值，即可以存储自由变量（非参数且不在表达式中定义）的值。
不能改变引用值，或在外部改变引用变量。即捕获的变量必须实际上是最终变量，初始化后就不会再为它赋新值。
this 关键字是指创建这个 lambda 表达式的方法的 this 参数。

# 内部类部分
**规则**
内部类的对象总有一个隐式引用（OuterClass.this）指向外部类对象，故可以访问创建它的外围类对象的实例域。
每个外部对象会分别有一个内部类实例，故内部类的静态域要加 final 确保唯一。而其方法不允许 static，可能因为弊大于利。

**局部内部类**
定义在方法中，不能用 public 和 private 声明。对外部世界完全隐藏，外部类也不能访问它。

**匿名内部类**
```
new SuperType(construction parameters)
	{
		inner class methods and data
	}
```

若构造一个数组列表并将其传递到方法后就不再需要，可以作为匿名列表：
```
invite(new ArrayList<String>() {{ add("A"; add("B"); )}});
```
外层括号建立了 ArrayList 的一个匿名子类，内层括号为子类添加元素。

调用 getClass() 是实际是 this.getClass() 而静态方法没有 this，所以应该：` new Object(){}.getClass().getEnclosingClass()` 。`new Object(){}` 会建立 Object 的一个匿名子类的匿名对象，`getEnclosingClass()` 则得到其外围类即包含对应静态方法的类。

**静态内部类**
当使用内部类只是为了隐藏在另一个类内部，不引用外围类对象，可以声明成 static 以便取消产生的引用。（有时内部类对象在静态方法中构造）
可以有静态域和方法。