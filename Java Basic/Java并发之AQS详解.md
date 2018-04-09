# Java并发之AQS详解

谈到并发，不得不谈`AbstractQueuedSynchronizer`(`AQS`)。

类如其名，抽象的队列式的同步器，`AQS`定义了一套多线程访问共享资源的同步器框架，许多同步类实现都依赖于它，如常用的`ReentrantLock`、`Semaphore`、`CountDownLatch`等。

## AQS数据结构

![](https://github.com/maoyunfei/static-sources/blob/master/AQS_1.jpeg?raw=true)
![](https://github.com/maoyunfei/static-sources/blob/master/AQS_2.png?raw=true)

## AQS简介

`AQS`提供了一个基于`FIFO`队列，可以用于构建锁或者其他相关同步装置的基础框架。该同步器利用了一个`int`来表示状态，期望它能够成为实现大部分同步需求的基础。使用的方法是继承，子类通过继承同步器并需要实现它的方法来管理其状态，管理的方式就是通过类似`acquire`和`release`的方式来操纵状态。然而多线程环境中对状态的操纵必须确保原子性，因此子类对于状态的把握，需要使用这个同步器提供的以下三个方法对状态进行操作：

* `java.util.concurrent.locks.AbstractQueuedSynchronizer.getState()`
* `java.util.concurrent.locks.AbstractQueuedSynchronizer.setState(int)`
* `java.util.concurrent.locks.AbstractQueuedSynchronizer.compareAndSetState(int, int)`

`AQS`定义两种资源共享方式：`Exclusive`(独占模式，只有一个线程能执行，如`ReentrantLock`)和`Share`(共享模式，多个线程可同时执行，如`Semaphore`/`CountDownLatch`)。

## 同步器与锁

**同步器是实现锁的关键，利用同步器将锁的语义实现，然后在锁的实现中聚合同步器**。可以这样理解：锁的API是面向使用者的，它定义了与锁交互的公共行为，而每个锁需要完成特定的操作也是透过这些行为来完成的（比如：可以允许两个线程进行加锁，排除两个以上的线程），但是实现是依托给同步器来完成；同步器面向的是线程访问和资源控制，它定义了线程对资源是否能够获取以及线程的排队等操作。**锁和同步器很好的隔离了二者所需要关注的领域，严格意义上讲，同步器可以适用于除了锁以外的其他同步设施上(包括锁)**。

同步器的开始提到了其实现依赖于一个`FIFO`队列，那么队列中的元素`Node`就是保存着线程引用和线程状态的容器，每个线程对同步器的访问，都可以看做是队列中的一个节点。Node的主要包含以下成员变量：

```java
Node {
    int waitStatus;
    Node prev;
    Node next;
    Node nextWaiter;
    Thread thread;
}
```
以上五个成员变量主要负责保存该节点的线程引用，同步等待队列(`sync`队列)的前驱和后继节点，同时也包括了同步状态。

|    属性名称    | 描述 |
| ---------- | --- |
| `int waitStatus`	 |  表示节点的状态。其中包含的状态有：<br/>1. `CANCELLED`，值为`1`，表示当前的线程被取消；<br/>2. `SIGNAL`，值为`-1`，表示当前节点的后继节点包含的线程需要运行，也就是`unpark`；<br/>3. `CONDITION`，值为`-2`，表示当前节点在等待`condition`，也就是在`condition`队列中；<br/>4. `PROPAGATE`，值为`-3`，表示当前场景下后续的`acquireShared`能够得以执行；<br/>5. 值为`0`，表示当前节点在`sync`队列中，等待着获取锁。 |
| `Node prev`	| 前驱节点，比如当前节点被取消，那就需要前驱节点和后继节点来完成连接。|
| `Node next`  | 后继节点。|
| `Node nextWaiter`  | 存储`condition`队列中的后继节点。|
| `Thread thread`  | 入队列时的当前线程。|

节点成为`sync`队列和`condition`队列构建的基础，在同步器中就包含了`sync`队列。同步器拥有三个成员变量：`sync`队列的头结点`head`、`sync`队列的尾节点`tail`和状态`state`。对于锁的获取，请求形成节点，将其挂载在尾部，而锁资源的转移(释放再获取)是从头部开始向后进行。对于同步器维护的状态`state`，多个线程对其的获取将会产生一个链式的结构。

![](https://github.com/maoyunfei/static-sources/blob/master/AQS_3.jpeg?raw=true)

## API说明

实现自定义同步器时，需要使用同步器提供的getState()、setState()和compareAndSetState()方法来操纵状态的变迁。

|    方法名称    | 描述 |
| ---------- | --- |
| `protected boolean tryAcquire(int arg)`| 独占方式。尝试获取资源，成功则返回`true`，失败则返回`false`。|
| `protected boolean tryRelease(int arg)` | 独占方式。尝试释放资源，成功则返回`true`，失败则返回`false`。|
| `protected int tryAcquireShared(int arg)` | 共享方式。尝试获取资源。负数表示失败；`0`表示成功，但没有剩余可用资源；正数表示成功，且有剩余资源。|
| `protected boolean tryReleaseShared(int arg)` | 共享方式。尝试释放资源，成功则返回`true`，失败则返回`false`。|
| `protected boolean isHeldExclusively()` | 该线程是否正在独占资源。只有用到`condition`才需要去实现它。|

实现这些方法必须是非阻塞而且是线程安全的，推荐使用该同步器的父类`java.util.concurrent.locks.AbstractOwnableSynchronizer`来设置当前的线程。

## 总结

`AQS`核心是通过一个共享变量来同步状态，变量的状态由子类去维护，而`AQS`框架做的是：

* 线程阻塞队列的维护
* 线程阻塞和唤醒

共享变量的修改都是通过`Unsafe`类提供的`CAS`操作完成的。

`AbstractQueuedSynchronizer`类的主要方法是`acquire`和`release`，典型的模板方法，下面这4个方法由子类去实现：

```java
protected boolean tryAcquire(int arg)
protected boolean tryRelease(int arg)
protected int tryAcquireShared(int arg)
protected boolean tryReleaseShared(int arg)
```

`acquire`方法用来获取锁，返回`true`说明线程获取成功继续执行，一旦返回`false`则线程加入到等待队列中，等待被唤醒，`release`方法用来释放锁。 一般来说实现的时候这两个方法被封装为`lock`和`unlock`方法。

## 参考资料

* [AbstractQueuedSynchronizer的介绍和原理分析](http://ifeve.com/introduce-abstractqueuedsynchronizer/)
* [Java并发之AQS详解](http://www.cnblogs.com/waterystone/p/4920797.html)














