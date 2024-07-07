---
description: Record the content learned in the book "The Art of Java Concurrency".
---

# Java并发的艺术

## synchronized的实现原理与应用

Java中的每一个对象都可以作为锁。具体表现为以下3种形式。

* 对于普通同步方法，锁是当前实例对象。
* 对于静态同步方法，锁是当前类的Class对象。
* 对于同步方法块，锁是Synchonized括号里配置的对象。

当一个线程试图访问同步代码块时，它首先必须得到锁，退出或抛出异常时必须释放锁。那么锁到底存在哪里呢？锁里面会存储什么信息呢？

从JVM规范中可以看到Synchonized在JVM里的实现原理，JVM基于进入和退出Monitor对象来实现方法同步和代码块同步，但两者的实现细节不一样。代码块同步是使用monitorenter和monitorexit指令实现的，而方法同步是使用另外一种方式实现的，细节在JVM规范里并没有详细说明。但是，方法的同步同样可以使用这两个指令来实现。

monitorenter指令是在编译后插入到同步代码块的开始位置，而monitorexit是插入到方法结束处和异常处，JVM要保证每个monitorenter必须有对应的monitorexit与之配对。任何对象都有一个monitor与之关联，当且一个monitor被持有后，它将处于锁定状态。线程执行到monitorenter指令时，将会尝试获取对象所对应的monitor的所有权，即尝试获得对象的锁。

<figure><img src=".gitbook/assets/image (3).png" alt=""><figcaption></figcaption></figure>

<figure><img src=".gitbook/assets/image (4).png" alt=""><figcaption></figcaption></figure>

### **Java对象头**

synchronized用的锁是存在Java对象头里的。如果对象是数组类型，则虚拟机用3个字宽（Word）存储对象头，如果对象是非数组类型，则用2字宽存储对象头。在32位虚拟机中，1字宽等于4字节，即32bit。

<figure><img src=".gitbook/assets/image (5).png" alt=""><figcaption></figcaption></figure>

Java对象头里的Mark Word里默认存储对象的HashCode、分代年龄和锁标记位。

<figure><img src=".gitbook/assets/image (6).png" alt=""><figcaption></figcaption></figure>

在运行期间，Mark Word里存储的数据会随着锁标志位的变化而变化。

<figure><img src=".gitbook/assets/image (7).png" alt=""><figcaption></figcaption></figure>

在64位虚拟机下，Mark Word是64bit大小的。

<figure><img src=".gitbook/assets/image (8).png" alt=""><figcaption></figcaption></figure>

### 锁的升级与对比

锁一共有4种状态，级别从低到高依次是：无锁状态、偏向锁状态、轻量级锁状态和重量级锁状态，这几个状态会随着竞争情况逐渐升级。锁可以升级但不能降级，意味着偏向锁升级成轻量级锁后不能降级成偏向锁。这种锁升级却不能降级的策略，目的是为了提高获得锁和释放锁的效率，下文会详细分析。

#### 偏向锁

HotSpot\[1]的作者经过研究发现，大多数情况下，锁不仅不存在多线程竞争，而且总是由同一线程多次获得，为了让线程获得锁的代价更低而引入了偏向锁。当一个线程访问同步块并获取锁时，会在对象头和栈帧中的锁记录里存储锁偏向的线程ID，以后该线程在进入和退出同步块时不需要进行CAS操作来加锁和解锁，只需简单地测试一下对象头的Mark Word里是否 存储着指向当前线程的偏向锁。如果测试成功，表示线程已经获得了锁。如果测试失败，则需 要再测试一下Mark Word中偏向锁的标识是否设置成1（表示当前是偏向锁）：如果没有设置，则 使用CAS竞争锁；如果设置了，则尝试使用CAS将对象头的偏向锁指向当前线程。

偏向锁使用了一种等到竞争出现才释放锁的机制，所以当其他线程尝试竞争偏向锁时，持有偏向锁的线程才会释放锁。偏向锁的撤销，需要等待全局安全点（在这个时间点上没有正在执行的字节码）。它会首先暂停拥有偏向锁的线程，然后检查持有偏向锁的线程是否活着，如果线程不处于活动状态，则将对象头设置成无锁状态；如果线程仍然活着，拥有偏向锁的栈会被执行，遍历偏向对象的锁记录，栈中的锁记录和对象头的Mark Word要么重新偏向于其他线程，要么恢复到无锁或者标记对象不适合作为偏向锁，最后唤醒暂停的线程。下图中的线程1演示了偏向锁初始化的流程，线程2演示了偏向锁撤销的流程

<figure><img src=".gitbook/assets/image (9).png" alt=""><figcaption></figcaption></figure>

#### 轻量级锁

1. 轻量级锁加锁

线程在执行同步块之前，JVM会先在当前线程的栈桢中创建用于存储锁记录的空间，并将对象头中的Mark Word复制到锁记录中，官方称为Displaced Mark Word。然后线程尝试使用CAS将对象头中的Mark Word替换为指向锁记录的指针。如果成功，当前线程获得锁，如果失败，表示其他线程竞争锁，当前线程便尝试使用自旋来获取锁。

2. 轻量级锁解锁

轻量级解锁时，会使用原子的CAS操作将Displaced Mark Word替换回到对象头，如果成功，则表示没有竞争发生。如果失败，表示当前锁存在竞争，锁就会膨胀成重量级锁。图2-2是两个线程同时争夺锁，导致锁膨胀的流程图。

<figure><img src=".gitbook/assets/image (10).png" alt=""><figcaption></figcaption></figure>

因为自旋会消耗CPU，为了避免无用的自旋（比如获得锁的线程被阻塞住了），一旦锁升级成重量级锁，就不会再恢复到轻量级锁状态。当锁处于这个状态下，其他线程试图获取锁时，都会被阻塞住，当持有锁的线程释放锁之后会唤醒这些线程，被唤醒的线程就会进行新一轮的夺锁之争。

#### 锁的优缺点对比

<figure><img src=".gitbook/assets/image (11).png" alt=""><figcaption></figcaption></figure>

## 关于volatile你应该知道的事

> 在最新的Java语言规范中这样定义volatile：
>
> The Java programming language allows threads to access shared variables ([§17.1](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.1)). As a rule, to ensure that shared variables are consistently and reliably updated, a thread should ensure that it has exclusive use of such variables by obtaining a lock that, conventionally, enforces mutual exclusion for those shared variables.
>
> The Java programming language provides a second mechanism, `volatile` fields, that is more convenient than locking for some purposes.
>
> A field may be declared `volatile`, in which case the Java Memory Model ensures that all threads see a consistent value for the variable ([§17.4](https://docs.oracle.com/javase/specs/jls/se8/html/jls-17.html#jls-17.4)).
>
> 翻译后：
>
> Java编程语言允许线程访问共享变量。通常，为了确保共享变量得到一致和可靠的更新，线程应该通过获得一个锁来确保它对这些共享变量具有独占使用权，该锁通常会强制执行这些共享变量的互斥。
>
> Java编程语言提供了第二种机制，即易失性字段，它在某些方面比锁定更方便。
>
> 字段可以被声明为volatile，在这种情况下，Java内存模型确保所有线程都能看到变量的一致值。

### 缓存一致性

在了解volatile实现原理之前，我们先来看下与其实现原理相关的CPU术语与说明。表2-1 是CPU术语的定义。

<figure><img src=".gitbook/assets/image.png" alt=""><figcaption></figcaption></figure>

volatile是如何来保证可见性的呢？让我们在X86处理器下通过工具获取JIT编译器生成的汇编指令来查看对volatile进行写操作时CPU会做什么事情。

Java代码如下：

<figure><img src=".gitbook/assets/image (1).png" alt=""><figcaption></figcaption></figure>

转变为汇编代码，如下：

<figure><img src=".gitbook/assets/image (2).png" alt=""><figcaption></figcaption></figure>

有volatile变量修饰的共享变量进行写操作的时候会多出第二行汇编代码，通过查IA-32架构软件开发者手册可知，Lock前缀的指令在多核处理器下会引发了两件事情\[1]。

1. **将当前处理器缓存行的数据写回到系统内存**。
2. **这个写回内存的操作会使在其他CPU里缓存了该内存地址的数据无效**。

为了提高处理速度，处理器不直接和内存进行通信，而是先将系统内存的数据读到内部缓存（L1，L2或其他）后再进行操作，但操作完不知道何时会写到内存。如果对声明了volatile的变量进行写操作，JVM就会向处理器发送一条Lock前缀的指令，将这个变量所在缓存行的数据写回到系统内存。但是，就算写回到内存，如果其他处理器缓存的值还是旧的，再执行计算操作就会有问题。所以，在多处理器下，为了保证各个处理器的缓存是一致的，就会实现缓存一致性协议，每个处理器通过嗅探在总线上传播的数据来检查自己缓存的值是不是过期了，当处理器发现自己缓存行对应的内存地址被修改，就会将当前处理器的缓存行设置成无效状态，当处理器对这个数据进行修改操作的时候，会重新从系统内存中把数据读到处理器缓存里。

### 指令重排序

关于 `volatile` 防止指令重排序的情况，这是因为它为变量引入了一种被称作 "内存屏障" 或 "内存栅栏" 的机制：\\

1. **写内存屏障（Write Barrier）**：当一个变量被声明为 `volatile` 并且对它进行写操作时，内存屏障确保了这个写操作之前的所有操作（在当前线程中）都不会被重排序到写操作之后。
2. **读内存屏障（Read Barrier）**：类似地，当进行 `volatile` 变量的读操作时，内存屏障确保了这个读操作之后的所有操作（在当前线程中）都不会被重排序到读操作之前。

\
这样一来，`volatile` 所提供的 "happens-before" 关系就保证了在一个线程中的写操作在另一个线程读取同一个 `volatile` 变量后是可见的，而且这之间的操作不会发生乱序。简而言之，它确保了对 volatile 变量的读写操作在所有处理器中都具有一致性，并且在一个处理器内部保证了不会发生乱序执行，从而提高了多线程程序的可靠性。
