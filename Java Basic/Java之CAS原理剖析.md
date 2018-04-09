# Java之CAS原理剖析

`CAS(Compare and Swap)`，即比较并交换，是实现并发算法常用的一种技术。

`CAS`的思想很简单：三个参数，一个当前内存值`V`、旧的预期值`A`、即将更新的值`B`，当且仅当预期值`A`和内存值`V`相同时，将内存值修改为`B`并返回`true`，否则什么都不做，并返回`false`。

## 问题

一个`n++`的问题。

```java
public class Case {

    public volatile int n;

    public void add() {
        n++;
    }
}
```
`n++`其实是被拆分成了几个指令：

1. 执行getfield拿到原始n；
2. 执行iadd进行加1操作；
3. 执行putfield写把累加后的值写回n；

通过`volatile`修饰的变量可以保证线程之间的可见性，但并不能保证这3个指令的原子执行，在多线程并发执行下，无法做到线程安全，得到正确的结果。

## 如何解决

在add方法加上`synchronized`修饰解决。

```java
public class Case {

    public volatile int n;

    public synchronized void add() {
        n++;
    }
}
```
但是`synchronized`属于**重量级锁**，很多时候会引起性能问题，像`synchronized`这种独占锁属于**悲观锁**，它是在假设一定会发生冲突的，那么加锁恰到好处，除此之外，还有**乐观锁**，乐观锁的含义就是假设没有发生冲突，那么我正好可以进行某项操作，如果要是发生冲突呢，那我就重试直到成功，乐观锁最常见的就是`CAS`。

再来看一段代码：

```java
public int a = 1;
public boolean compareAndSwapInt(int b) {
    if (a == 1) {
        a = b;
        return true;
    }
    return false;
}
```
以上代码在并发下执行，结果是无法符合预期的，无法确认a的最终值。

同样可以在`compareAndSwapInt`方法加锁同步，变成一个原子操作，同一时刻只有一个线程能够修改变量a。

除了低性能的加锁方案，我们可以使用JDK自带的`CAS`方案，在`CAS`中，比较和替换是一组原子操作，不会被外部打断，且在性能上更占优势。

下面以`AutomicInteger`的实现为例，分析一下`CAS`是如何实现的。

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;
    public final int get() {return value;}
}
```
1. `Unsafe`，是`CAS`的核心类，由于Java方法无法直接访问底层系统，需要通过本地(native)方法来访问，`Unsafe`相当于一个后门，基于该类可以直接操作特定内存的数据。
2. 变量`valueOffset`，表示该变量值在内存中的偏移地址，因为`Unsafe`就是根据内存偏移地址获取数据的。
3. 变量`value`用`volatile`修饰，保证了多线程之间的内存可见性。

看看`AutomicInteger`如何实现并发下的累加操作：

```java
public final int getAndAdd(int delta) {    
    return unsafe.getAndAddInt(this, valueOffset, delta);
}

//unsafe.getAndAddInt
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));
    return var5;
}
```

假设线程A和线程B同时执行`getAndAdd`操作：

1. `AutomicInteger`里面的`value`原始值为3，即主内存中`AutomicInteger`的`value`为3，根据Java内存模型，线程A和线程B各自持有一份`value`的副本，值为3。
2. 线程A通过`getIntVolatile(var1, var2)`拿到`value`值3，这时线程A被挂起。
3. 线程B也通过`getIntVolatile(var1, var2)`方法获取到`value`值3，运气好，线程B没有被挂起，并执行`compareAndSwapInt`方法比较内存值也为3，成功修改内存值为2。
4. 这时线程A恢复，执行`compareAndSwapInt`方法比较，发现自己手里的值(3)和内存的值(2)不一致，说明该值已经被其它线程提前修改过了，那只能重新来一遍了。
5. 重新获取`value`值，因为变量`value`被`volatile`修饰，所以其它线程对它的修改，线程A总是能够看到，线程A继续执行`compareAndSwapInt`进行比较替换，直到成功。

整个过程中，利用`CAS`保证了对于`value`的修改的并发安全，继续深入看看`Unsafe`类中的`compareAndSwapInt`方法实现。

```java
public final native boolean compareAndSwapInt(Object var1, long var2, int var4, int var5);
```
`Unsafe`类中的`compareAndSwapInt`，是一个本地方法，该方法的实现位于`unsafe.cpp`中。

`compareAndSwapInt(var1, var2, var5, var5 + var4)`其实换成`compareAndSwapInt(obj, offset, expect, update)`比较清楚，意思就是如果`obj`内的`value`和`expect`相等，就证明没有其他线程改变过这个变量，那么就更新它为`update`，如果这一步的`CAS`没有成功，那就采用自旋的方式继续进行`CAS`操作。

## CAS的问题

### ABA问题

`CAS`需要在操作值的时候检查下值有没有发生变化，如果没有发生变化则更新，但是如果一个值原来是A，变成了B，又变成了A，那么使用`CAS`进行检查时会发现它的值没有发生变化，但是实际上却变化了。这就是`CAS`的ABA问题。常见的解决思路是使用版本号。在变量前面追加上版本号，每次变量更新的时候把版本号加一，那么`A-B-A`就会变成`1A-2B-3A`。目前在JDK的automic包里提供了一个类`AutomicStampedReference`来解决ABA问题。这个类的`compareAndSet`方法作用是首先检查当前引用是否等于预期引用，并且当前标志是否等于预期标志，如果全部相等，则以原子方式将该引用和该标志的值设置为给定的更新值。

### 循环时间长开销大

如果`CAS`不成功，则会原地自旋，如果长时间自旋会给CPU带来非常大的执行开销。

### 只能保证一个共享变量的原子操作

当对一个共享变量执行操作时，我们可以使用循环`CAS`的方式来保证原子操作，但是对多个共享变量操作时，循环`CAS`就无法保证操作的原子性，这个时候就可以用锁，或者有一个取巧的办法，就是把多个共享变量合并成一个共享变量来操作。从Java 1.5开始JDK提供了`AutomicReference`类来保证引用对象之间的原子性，你可以把多个变量放到一个对象里来进行`CAS`操作。

## 总结

* `synchronized`属于重量级锁，悲观锁，可以修饰代码块，修饰方法，修饰静态方法。

* `CAS`是非阻塞的，轻量级锁，乐观锁。

*  悲观锁机制存在的问题：

> 在多线程竞争下，加锁、释放锁会导致比较多的上下文切换和调度延时，引起性能问题。
> 
> 一个线程持有锁会导致其他所有需要此锁的线程挂起。
> 
> 如果一个优先级高的线程等待一个优先级低的线程释放锁会导致优先级倒置，引起性能风险。

*  `CAS`在竞争激烈的时候会长时间自旋，引起性能问题。

## 参考资料

* [深入浅出CAS](https://www.jianshu.com/p/fb6e91b013cc)
* [Java CAS 原理剖析](https://juejin.im/post/5a73cbbff265da4e807783f5)