---
title: Java 补丁-继承
date: 2019-12-11 19:29:45
tags:
- 继承
- 泛型
- 数组
categories: Java
---

# 超类子类部分
**this 和 super**

|  this  |  super  |
| --- | --- |
|  引用隐式参数  |  调用超类方法  |
|  调用该类其他构造器  |  调用超类的构造器  |

但是：super 不是一个对象的引用，只是一个指示编译器调用超类方法的_关键词_

**is-a 规则**
出现超类对象的任何地方都可以通过子类对象置换。即置换法则。

**方法调用**
假设调用 x.f(args)，x 是类 C 的一个对象：
1. 编译器查看对象的声明类型和方法名。编译器会一一列举所有 C 类中名为 f 的方法和其超类中声明为 public 且名为 f 的方法。
2. 编译器查看调用方法时提供的参数类型，选择第一步中与提供的参数类型完全匹配的方法。即重载解析。这个过程允许类型转换，若没有找到或找到多个则报告错误。<-- 此时已获得需要调用的方法名和参数类型
3. 如果是 private static final 方法或者构造器，编译器已知道应该调用的方法，即静态绑定。若调用的方法依赖隐式参数的实际类型，称动态绑定。此例中编译器通过动态绑定确定调用 f(String) 方法。
4. 采用动态绑定调用方法时，虚拟机会调用与 x 所引对象_实际类型_最符合的那个类的方法。若 x 实际类型为 D，是 C 的子类。若 D 定义了 f(String) 方法就调用，否则在 D 的超类中寻找。

为了应对搜索方法的时间开销，虚拟机预先为每个类创建方法表，列出所有方法签名和实际调用的方法。

**绑定**
_静态绑定_：在程序执行以前已经被绑定（即在编译过程中就已经知道这个方法到底是哪个类中的方法）。
_动态绑定_：在运行时期根据具体对象的类型进行绑定。
Java 中重载的方法使用静态绑定，重写的方法使用动态绑定。

**protected**
仅对本类可见，包括子类。谨慎使用于数据域，因为他人可能由这个类派生出新类并访问 protected，若要对此类进行修改就必须通知所有相关人员。更适合使用于方法。

# Objects 部分
**覆盖 equals**
1. 检测 this 与显式参数是否引用同一个对象。`if (this == otherObj) return true;`
2. 检测显式参数是否为 null，因为 null 调用方法会报错。 `if (otherObj == null) return false;`
3. 比较 this 与显式参数是否为同一类。为了保证 equals 的对称性，应该 `if (getClass() != otherObj.getclass()) return false;` 但是例如 AbstractSet 类拥有 TreeSet 和 HashSet 两个具体子类，实现了同样的操作。此时应该 `if (!(otherObj instanceof ClassName)) return false;`
4. 将 otherObj 转换为相应的类类型变量。 `ClassName other = (ClassName) otherObj;`
5. 对需要比较的域进行比较，== 比较基本域，Objects.equals 比较对象域。 `return field1 == other.field1 $$ Objects.equals(field2, other.field2);`
如果在子类中重新定义了 equals，先调用 `super.equals(other);` 比较，再比较子类中的实例域。
覆盖 Objects 类的 equals 方法就不要弄错显式参数类型！可以用 @Override 进行标记。
对于数组类型，使用静态 Arrays.equals 方法。

**覆盖 hashCode**
不同对象的散列码基本不会相同，最好使用 null 安全的 Objects.hashCode，参数为 null 时返回 0；使用静态方法避免创建对象。
```
public int hashCode(){
	return 7 * Objects.hashCode(name) + 11* Double.hashCode(salary);
	// 或者
	return Objects.hashCode(name, salary);
}
```
**覆盖 toString**
Objects 的 toString 输出对象所属的类名和散列码。有时候直接输出这个大概是设计者没覆盖...
比如数组继承了 Objects 的方法，所以用静态方法 Arrays.toString 比较好。

# 泛型数组列表部分
**数组长度**
确认数组列表不再发生变化时使用 trimToSize 方法，将存储区域的大小调整为当前，回收多余存储空间。不影响之后再添加元素，但和扩展一样，会自动创建更大的数组并拷贝进去。

**参数数量可变方法**
... 代表当前方法可以接收任意数量的对象，相当于接收了对象数组。
```
public static double max(double... values){}
// 基本类型会自动装箱
```


