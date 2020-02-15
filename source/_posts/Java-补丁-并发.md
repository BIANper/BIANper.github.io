---
title: Java 补丁-并发
date: 2020-01-18 20:10:17
tags:
- 线程
- 同步

categories: Java
---

并发得专门开个分类 x

到时候把这篇删掉 x

# 零碎

所有实例域、静态域和数组元素都储存在堆内存中，堆内存在线程之前共享。
JMM 定义了：线程之间的共享变量储存在主内存中，每个线程都有一个私有的本地内存，本地内存储存了该线程已读写共享变量的副本。

- 线程 A 把本地内存 A 中的共享变量副本刷新到主内存中。
- 线程 B 去读取主内存中线程 A 刷新过的共享变量。
- 从整体来看，这两个步骤实质上是线程A向线程B发送消息，而通信必须经过主内存。JMM 通过控制主内存与每个线程的本地内存之间的交互，来提供内存可见性的保证。

# 线程状态

**新创建线程 New**

如 `new Thread(r)` ，该线程还没有开始运行。

**可运行线程 Runnable**

调用 start 方法便处于 runnable 状态，可能正在运行也可能没有运行。

**被阻塞线程和等待线程 Blocked Waiting**

线程处于被阻塞或等待状态时，会暂时不活动，直到被线程调度器重新激活。

当一个线程试图获取一个内部的对象锁，而该锁被其他进程持有则进入阻塞状态。直到所有其他线程释放该锁且被线程调度器允许持有。

**被终止线程 Terminated**

因为 run 方法正常退出而自然死亡。或一个没有捕获的异常终止了 run 方法而意外死亡。（过时方法 stop 通过抛出 ThreadDeath 错误对象杀死线程）

# 中断

线程随着 thread.start() 开始启动 到 run() 方法执行完毕 结束。通过 Thread.interrupted() 方法中断线程。
中断可以理解为线程的一个标识位属性，它表示一个运行中的线程是否被其他线程进行了中断操作。
线程通过检查自身是否被中断来进行响应，线程通过方法 isInterrupted() 来进行判断是否被中断，也可以调用静态方法 Thread.interrupted() 对当前线程的中断标识位进行复位。如果该线程已经处于终结状态，即使该线程被中断过，在调用该线程对象的 isInterrupted() 时依旧会返回 false。
许多声明抛出 InterruptedException 的方法（例如 Thread.sleep(long millis) 方法）这些方法在抛出 InterruptedException 之前，Java 虚拟机会先将该线程的中断标识位清除，然后抛出 InterruptedException，此时调用isInterrupted() 方法将会返回 false。
在线程 sleep 的时候，调用 isInterrupted() 会导致 sleep interrupted 异常，并且中断标记也被清除了。

# 线程属性

**线程优先级**

默认继承父线程的优先级。使用 setPriority 方法修改为 1 `MIN_PRIORITY` 到 10 `MAX_PRIORITY` 之间的任何值。

线程优先级高度依赖于系统，即会映射到宿主机平台的优先级上。

**守护线程**

在线程启动前 `t.setDaemon(true)` 将线程转换为守护线程，为其他线程提供服务，如计时。当只剩下守护线程时虚拟机就退出了。守护线程不应该访问固有资源如文件，数据库，要考虑 shutdown 动作。

**未捕获异常处理器**

线程的 run 方法不能抛出任何受查异常，但非受查异常会导致线程终止。实际不需要 catch 子句来处理，在线程死亡之前异常被传播到一个用于未捕获异常的处理器。

该处理器必须属于一个实现 Thread.UncaughtExceptionHandler 接口的类，只有一个 `void uncaughtException(Thread t, Throwable e)` 。用 `setUncaughtExceptionHandler` 为任何线程安装一个处理器，或者用 `Thread.setDefaultUncaughtExceptionHandler`为所有线程安装默认的处理器。

若不安装处理器则使用该线程的 ThreadGroup 对象，其 uncaughtException 方法如下：

1.  if 有父线程组，调用父线程组的 uncaughtException 方法
2.  else 若 `Thread.setDefaultUncaughtExceptionHandler` 返回非空的处理器（即有默认的处理器），则使用
3.  else 若 Throwable 是 ThreadDeath 的一个实例，则什么也不做
4.  else 线程名及 Throwable 的栈轨迹被输出到 System.err 

# 同步

**ReentrantLock**

```java
private Lock myLock = new ReentrantLock();
// implements the Lock interface
myLock.lock(); // a ReentrantLock object
try{ critical section }
finally { myLock.unLock(); }
```

确保任何时刻只有一个线程进入临界区。一旦线程封锁了锁对象，任何线程都无法通过 lock 语句，将被阻塞至锁对象释放。

不要将获取锁的过程写在try块中，因为如果在获取锁（自定义锁的实现）时发生了异常，异常抛出的同时，也会导致锁无故释放。为确保临界区代码抛出异常后会将锁释放，必须将解锁置于 finally 子句。 

可重入：锁保持一个持有计数跟踪对 lock 方法的嵌套调用。被锁保护的代码可以调用另一个使用相同的锁的方法。

-   transfer 方法中调用 getTotal 方法，此时 Lock 对象持有计数 2，当 getToal 方法退出时，计数变为 1；当 transfer 方法退出计数变为 0，线程释放锁。

`ReentrantLock(boolean fair)` 是一个公平锁，偏向等待时间最长的线程。会极大降低性能。

Lock 接口提供的 synchronized 关键字所不具备的主要特性：

- 尝试非阻塞性获取锁：当前线程尝试获取锁，如果此时没有其他线程占用此锁，则成功获取到锁。
- 能被中断的获取锁： 当获取到锁的线程被中断时，中断异常会抛出并且会释放锁。
- 超时获取锁： 在指定时间内获取锁，如果超过时间还没获取，则返回。

Lock api
void lockInterruptibly() throws InterruptedException ：可中断的获取锁
boolean tryLock() ：尝试非阻塞的获取锁
boolean tryLock(long time, TimeUnit unit) throws InterruptedException ： 超时获取锁。 超时时间结束，未获得锁，返回 false。

**条件对象**

用于管理已经获得一个锁但在满足某一条件后才执行的线程。

```java
Condition newCondition()
void await()
void signalAll()
void signal()
```

await 与 waiting 不同在于进入的是对应条件的等待集，当锁可用时仍处于阻塞状态，直到另一个线程调用同一条件上的 signalAll 方法。

signalAll 激活由对应条件而等待的所有线程（或者说接触线程的阻塞，通过竞争实现访问），线程应该再次测试该条件以确保被满足。

**synchronized**
使用 synchronized 关键字声明方法，对象的内部锁将保护整个方法。

```java
public synchronized void method() { ... }
```

等价于

```java
public void method() {
		this.intrinsicLock.lock();
		try { ... }
		finally {
			this.intrinsicLock.unnlock();
		}
}
```

内部对象锁只有一个条件，wait 方法添加线程到等待集中（`void wait(long millis)` `void wait(long millis, int nanos)`）， notifyAll/notify 方法解除等待线程的阻塞状态。

对于普通同步方法，锁是当前实例对象。
对于静态同步方法，锁是当前类的Class对象。
对于同步方法块，锁是 Synchonized 括号里配置的对象。

等待方原则：

```java
synchronized (对象) {
    while (条件不满足) {
        对象.wait();
    }
    对应的处理逻辑
}
```

通知方原则：

```java
synchronized (对象) {
    改变条件
    对象.notifyAll();
}
```

**同步阻塞**
就是同步方法块。

**volatile**
为实力的同步访问提供免锁方案。不提供原子性。

> 如果向一个变量写入值，而将会被另一个线程读取。或者从一个变量读取值，而之前可能被另一个线程写入，必须使用同步。

```java
private volatile boolean bo = true;
public void flipBoolean() { bo = !bo; }
// 不保证读取，写入 ，反转不被中断
```

**final**
将一个域声明为 final 时，其他线程在其完成构造后才能看到对应变量。

```java
final Map<String, Double> acounts = new HashMap<>();
// 其他线程不会获得 acounts = null
// 本身并不线程安全，读写仍需同步
```

**Thread.join()**
主线程生成并起动了子线程，如果子线程里要进行大量的耗时的运算，主线程往往将于子线程之前结束，但是如果主线程处理完其他的事务后，需要用到子线程的处理结果，即主线程需要等待子线程执行完成之后再结束。
join()的作用是：主线程等待该子线程的终止。即在子线程调用了join()方法后面的代码，只有等到子线程结束了才能执行。

**ThreadLocal**
线程局部变量，以 ThreadLocal 对象为键 、任意对象为值的存储结构。
这个结构被附带在线程上，所以一个线程可以根据一个ThreadLocal对象查询到绑定在这个线程上的一个值。set(T t) 设置，get() 获取。