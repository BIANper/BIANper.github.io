---
title: Java 补丁-集合
date: 2019-12-25 17:00:45
tags:
- 集合
- 队列
- 散列
- 映射
categories: Java
---

# 具体集合部分

**链表 LinkedList**
List 接口描述有序集合，有两种访问元素的协议，迭代器或 get set 方法，后者更适合数组。链表对于删除插入元素很有效，允许对象有重复的值。
Java 中链表都是双向链接的，即每个结点存放前驱结点和下一个结点的引用。如果链表有 n 个元素则有 n+1 个位置可以插入新元素。
链表只跟踪结构性修改，set 方法不被视为结构性修改。
避免使用整数索引表示链表中位置的方法，如果需要对集合进行指定访问，就使用数组或 ArrayList 而非链表。

**数组链表 ArrayList**
也实现了 List 接口，封装了动态再分配的对象数组。不需要同步时使用，若有多个线程访问，应使用 Vector类。

**散列集 HashSet**
Set 的集合里不允许对象有重复的值。HashSet 仅仅存储对象（HashMap 储存键值对），基于 HashMap 实现。
散列表为每一个对象计算一个整数（hash code），Java 中用链表数组实现，每个表称为桶。
HashSet 类实现了基于散列表的集，散列集迭代器将依次访问所有桶，由于元素分散在表的各个位置上（散列码与桶的总数取余），所以访问的顺序是随机的。
对与集中的元素，如果元素的散列码发生了改变，在数据结构中的位置也会发生变化。

**树集 TreeSet**
TreeSet 是一个有序集合，用红黑树实现，任意顺序将元素插入集合遍历时自动排序输出。树的排序必须是全序即任意两个元素是可比的，并且元素必须实现 Comparable 接口，或构造集时提供一个 Comparator 。
 添加元素比散列表慢但是检查重复元素快很多。

 **双端队列**
 可以在头部和尾部同时添加或删除元素，不支持在队列中间添加元素。Deque 接口由 ArrayDeque 和 LinkedList 类实现，必要时可增加队列长度。

 **优先级队列**
 使用堆。可以保存实现了 Comparable 接口的类对象，或在构造器中提供的 Comparator 对象。

# 映射部分
**基本**
HashMap 和 TreeMap 类都实现了 Map 接口。散列映射对键进行散列，树映射用键的整体顺序对元素进行排序并组织成搜索树。
不能对同一个键存放两个值，若调用两次 put 方法，第二个值会取代第一个值并返回存储的上一个值。若要迭代处理映射的键值，可以使用 forEach 方法配合 lambda 表达式。
TreeMap保存了对象的排列次序。HashMap允许键和值为null。

**更新映射项**
获得键关联的原值，更新后再放回如：`counts.put(word, counts.get(word) + 1)` 。当 word 没有预先存在的话，get 将返回 null 而 put 的值不能为 null 。可以使用：

- counts.put(word, counts.getOrDefault(word, 0) + 1);
- counts.putIfAbsent(word, 0); counts.put(word, counts.get(word) + 1);
- counts.merge(word, 1, Integer::sum);

**映射视图**

一共分为三种：

-   键集 Set\<K>  keySet()
-   值集合 Collection\<V>  values()
-   键值对集 Set<Map.Entry<K,V>>  entrySet()

**弱散列映射**

WeakHashMap 类，当对键的唯一引用来自散列条目，将删除键值对。使用弱引用保存键，将引用保存到散列键对象中，垃圾回收会用特有方式处理此类对象。

>   新建WeakHashMap，将“键值对”添加到WeakHashMap中。 实际上，WeakHashMap是通过数组table保存Entry(键值对)；每一个Entry实际上是一个单向链表，即Entry是键值对链表。
>
>   当某“弱键”不再被其它对象引用，并被GC回收时。在GC回收该“弱键”时，这个“弱键”也同时会被添加到ReferenceQueue(queue)队列中。
>
>   当下一次需要操作WeakHashMap时，会先同步table和queue。table中保存了全部的键值对，而queue中保存被GC回收的键值对；同步它们，就是删除table中被GC回收的键值对。

**链接散列集和映射**

LinkedHashSet 和 LinkedHashMap 类会保留插入元素项的顺序。LinkedHashSet 是对 LinkedHashMap 的简单包装。LinkedHashMap 是 HashMap 的直接子类，二者唯一的区别是 LinkedHashMap 在 HashMap 的基础上，采用双向链表的形式将所有 entry 连接起来，保证元素的迭代顺序跟插入顺序相同。
会将 get 或 put 影响的条目移至链表尾部。

**枚举集与映射**
EnumSet 内部用位序列实现，对应值在集中则相应位被置 1。使用静态方法构造这个集：`enum Weekday { ... };`，使用 Set 接口的方法修改 EnumSet。
EnumMap 是以键类型为枚举类型的映射，需要在构造时指定：`EunmMap<Weekday, Employee> test = new EnumMap<>{Weekday.class}`。

**标识散列映射**
IdentityHsahMap 中键的散列值用 System.identityHashCode 方法计算，即根据内存地址计算。比较两对象时，IdentityHashMap 类使用 == 。
故不同的键对象即使内容相同，也视为不同对象。


# 视图与包装器

视图可以获得其他实现了 Collection 接口和 Map 接口的对象，如 keySet 方法并没有创建新集，而是返回了一个实现 Set 接口的类对象，其方法对原映射进行操作。

**轻量集合包装器**
Arrrays 类的静态方法 asList 返回一个包装了普通 Java 数组的 List 包装器而非 ArrayList 对象：
```java
Card[] cardDeck = new Card[52];
...
List<Card> cardList = Arrays.asList(cardDeck);
```
改变数组大小的方法将抛出异常。

**子范围视图**
如从列表 staff 中取出第 10-19 个元素：
```java
List group = staff.subList(10, 20);
```
对于有序集和映射，可以使用排序顺序而非元素位置建立子范围。如 `SortedSet` `SortMap`

**不可修改的视图**
Collection 中一些方法用于产生不可修改视图，运行时检查到修改则抛出一个异常。

**同步视图**
类库设计者使用视图确保集合的线程安全，而非实现线程安全的集合类。如将任何一个映射表转换成具有同步访问方法的 Map 视图：
```java
Map<String,, Employee> map = Collections.synchronizedMap(new HashMap<String, Employee>());
```

