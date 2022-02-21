##### 1、synchronized 的使用小例子?

```java
public class SynchronizedTest {
 
    public static volatile int race = 0;
 
    private static CountDownLatch countDownLatch = new CountDownLatch(2);
 
    public static void main(String[] args) throws InterruptedException {
        // 循环开启2个线程来计数
        for (int i = 0; i < 2; i++) {
            new Thread(() -> {
                // 每个线程累加1万次
                for (int j = 0; j < 10000; j++) {
                    race++;
                }
                countDownLatch.countDown();
            }).start();
        }
        // 等待 ，直到所有线程处理结束才放行【直到计数器变为0的时候】
        countDownLatch.await();
        // 期望输出 2万（2*1万）
        System.out.println(race);
    }
}
```

熟悉的2个线程计数的例子，每个线程自增1万次，预期的结果是2万，但是实际运行结果总是一个小于等于2万的数字，为什么会这样了？

race++在我们看来可能只是1个操作，但是在底层其实是由多个操作组成的，所以在并发下会有如下的场景：

![image-20220221144334121](images/image-20220221144334121.png)

为了得到正确的结果，此时我们可以将 race++ 使用 synchronized 来修饰，如下：

```java
synchronized (SynchronizedTest.class) {
    race++;
}
```

加了 synchronized 后，只有抢占到锁才能对 race 进行操作，此时的流程会变成如下：

![img](images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3YxMjM0MTE3Mzk=,size_16,color_FFFFFF,t_70-16454139810177.png)

##### 2.synchronized 各种加锁场景？

* 作用于非静态方法，锁住的是对象实例（this），每一个**对象实例**有一个锁

```java
public synchronized void method() {}
```

* 作用于静态方法，锁住的是类的 Class 对象，Class 对象全局只有一份，因此静态方法锁相当于类的一个全局锁，会锁所有调用该方法的线程

```java
public static synchronized void method() {}
```

* 作用于 Lock.class，锁住的是 Lock 的 Class 对象，也是全局只有一个。这个Lock也可以是其他的类

```java
synchronized (Lock.class) {}
```

* 作用于 this，锁住的是对象实例，每一个对象实例有一个锁。

```java
synchronized (this) {}
```

* 作用于静态成员变量，锁住的是该静态成员变量对象，由于是静态变量，因此全局只有一个。

```java
public static Object monitor = new Object(); 
synchronized (monitor) {}
```

有些同学可能会搞混，但是其实很容易记，记住以下两点:

1.必须有“对象”来充当“锁”的角色

2.对于同一个类来说，通常只有两种对象来充当锁：实例对象、Class 对象（一个类全局只有一份）。

Class 对象：静态相关的都是属于 Class 对象，还有一种直接指定 Lock.class。

实例对象：非静态相关的都是属于实例对象。

##### 3.为什么调用 Object 的 wait/notify/notifyAll 方法，需要加 synchronized 锁？

这个问题说难也难，说简单也简单。说简单是因为，大家应该都记得有道题目：“sleep 和 wait 的区别”，答案中非常重要的一项是：“**wait会释放对象锁，sleep不会**”，既然要释放锁，那必然要先获取锁。

说难是因为如果没有联想到这个题目并且没有了解的底层原理，可能就完全没头绪了。

究其原因，因为这3个方法都会操作锁对象，所以需要先获取锁对象，而加 synchronized 锁可以让我们获取到锁对象。

来看一个例子：

```java
public class SynchronizedTest {
    private static final Object lock = new Object();
     
    public static void testWait() throws InterruptedException {
        lock.wait();
    }
     
    public static void testNotify() throws InterruptedException {
        lock.notify();
    }
}
```

在这个例子中，wait 会释放 lock 锁对象，notify/notifyAll 会唤醒其他正在等待获取 lock 锁对象的线程来抢占 lock 锁对象。

既然你想要操作 lock 锁对象，那必然你就得先获取 lock 锁对象。就像你想把苹果让给其他同学，那你必须先拿到苹果。

再来看一个反例：

```java
public class SynchronizedTest {
    private static final Object lock = new Object();
    
    public static synchronized void getLock() throws InterruptedException {
        lock.wait();
    }
}
```

该方法运行后会抛出 IllegalMonitorStateException，为什么了，我们明明加了 synchronized 来获取锁对象了？

因为在 getLock 静态方法中加 synchronized 方法获取到的是 SynchronizedTest.class 的锁对象，而我们的 wait() 方法是要释放 lock 的锁对象。

这就相当于你想让给其他同学一个苹果（lock），但是你只有一个梨子（SynchronizedTest.class）。

##### 4.synchronize 底层维护了几个列表存放被阻塞的线程？

这题是紧接着上一题的，很明显面试官想看看我是不是真的对 synchronize 底层原理有所了解。

synchronized 底层对应的 JVM 模型为 objectMonitor，使用了3个双向链表来存放被阻塞的线程：_cxq（Contention queue 竞争队列）、_EntryList（EntryList）、_WaitSet（WaitSet）。

当线程**获取锁失败**进入阻塞后，**首先**会被加入到 _**cxq 链表**，_**cxq 链表**的节点会在**某个时刻被进一步转移到 _EntryList 链表**。

具体转移的时刻？见题目30。

当持有锁的线程释放锁后，**_EntryList 链表头结点**的线程会**被唤醒**，该线程称为 successor（**假定继承者**），然后该线程**会尝试抢占锁**。

当我们调用 **wait()** 时，线程会被放入 **_WaitSet**，直到调用了 **notify()/notifyAll()** 后，线程**才被重新放入 _cxq 或 _EntryList**，**默认放入 _cxq 链表头部**。

objectMonitor 的整体流程如下图：
![img](images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3YxMjM0MTE3Mzk=,size_16,color_FFFFFF,t_70-16454246181169.png)

##### 5、为什么释放锁时被唤醒的线程会称为“假定继承者”？被唤醒的线程一定能获取到锁吗？

因为被唤醒的线程并不是就一定获取到锁了，该线程仍然需要去**竞争锁**，而且可能会失败，所以该线程并不是就**一定会**成为锁的“继承者”，而**只是有机会成为**，所以我们称它为**假定**的。

这也是 synchronized 为什么是**非公平锁**的一个原因。

##### 6.synchronized 为什么是非公平锁？非公平体现在哪些地方？

synchronized 的非公平其实在源码中应该有不少地方，因为设计者就没按公平锁来设计，核心有以下几个点：

1.当持有锁的线程释放锁时，该线程会执行以下两个重要操作：

* 先将锁的**持有者 owner 属性赋值为 null**
* **唤醒**等待链表中的一个线程（**假定继承者**）

在上面两点之间，**如果有其他线程刚好在尝试获取锁（例如自旋），则可以马上获取到锁**。

2.当线程尝试获取锁失败，**进入阻塞时，放入链表的顺序，和最终被唤醒的顺序是不一致的**，也就是说你先进入链表，不代表你就会先被唤醒。

##### 7.既然加了 synchronized 锁，那当某个线程调用了 wait 的时候明明还在 synchronized 块里，其他线程怎么进入到 synchronized 里去执行 notify 的？

如下例子：调用 lock.wait() 时，线程就阻塞在这边了，此时代码执行应该还在 synchronized 块里，其他线程为什么就能进入 synchronized 块去执行 notify() ？

```java
public class SynchronizedTest {

    private static final Object lock = new Object();
     
    public static void testWait() throws InterruptedException {
        synchronized (lock) {
            // 阻塞住，被唤醒之前不会输出aa，也就是还没离开synchronized
            lock.wait();
            System.out.println("aa");
        }
    }
     
    public static void testNotify() throws InterruptedException {
        synchronized (lock) {
            lock.notify();
            System.out.println("bb");
        }
    }
}
```

只看代码确实会给人题目中的这种错觉，这也是 Object 的 wait() 和 notify() 方法很多人用不好的原因，包括我也是用的不太好。

这个题需要从底层去看，当线程进入 synchronized 时，需要获取 lock 锁，但是在调用 lock.wait() 的时候，此时虽然线程还在 synchronized 块里，但是其实已经释放掉了 lock 锁。

所以，其他线程此时可以获取 lock 锁进入到 synchronized 块，从而去执行 lock.notify()。

##### 8.如果有多个线程都进入 wait 状态，那某个线程调用 notify 唤醒线程时是否按照进入 wait 的顺序去唤醒？

答案是否定的。上面在介绍 synchronized 为什么是非公平锁时也介绍了**不会按照顺序去唤醒**。

调用 wait 时，节点进入**_WaitSet** 链表的尾部。

调用 notify 时，根据不同的策略，节点可能被移动到 cxq 头部、cxq 尾部、EntryList 头部、EntryList 尾部等多种情况。

所以，**唤醒的顺序并不一定是进入 wait 时的顺序**。

##### 9.notifyAll 是怎么实现全唤起的？

**nofity** 是获取 **WaitSet 的头结点，执行唤起操作**。

**nofityAll** 的流程，**可以简单的理解为就是循环遍历 WaitSet 的所有节点，对每个节点执行 notify 操作**。

##### 10.JVM 做了哪些锁优化？

偏向锁、轻量级锁、自旋锁、自适应自旋、锁消除、锁粗化

##### 11.为什么要引入偏向锁和轻量级锁？为什么重量级锁开销大？

重量级锁底层依赖于**系统的同步函数来实现**，在 linux 中使用 pthread_mutex_t（**互斥锁**）来实现。

这些**底层的同步函数**操作会涉及到：操作系统**用户态**和**内核态**的切换、**进程的上下文切换**，而这些操作都是比较耗时的，因此**重量级锁操作的开销比较大**。

而在很多情况下，**可能获取锁时只有一个线程**，**或者是多个线程交替获取锁**，在这种情况下，使用重量级锁就不划算了，因此**引入了偏向锁和轻量级锁**来**降低没有并发竞争时的锁开销**。

##### 12.偏向锁有撤销、膨胀，性能损耗这么大为什么要用呢？

偏向锁的好处是在只有一个线程获取锁的情况下，只需要通过**一次 CAS** 操作修改 **markword** ，之后每次进行简单的判断即可，**避免了轻量级锁每次获取释放锁时的 CAS 操作**。

如果确定同步代码块**会被多个线程访问或者竞争较大**，可以通过 -XX:-UseBiasedLocking 参数关闭偏向锁。

##### 13.偏向锁、轻量级锁、重量级锁分别对应了什么使用场景？todo？

1）偏向锁

适用于只有一个线程获取锁。当第二个线程尝试获取锁时，即使此时第一个线程已经释放了锁，此时还是会升级为轻量级锁。

但是有一种特例，如果出现**偏向锁的重偏向**，则此时第二个线程可以尝试获取偏向锁。

2）轻量级锁

适用于**多个线程交替获取锁**。跟偏向锁的区别在于可以**有多个线程**来获取锁，但是**必须没有竞争**，如果**有则会升级会重量级锁**。有同学可能会说，不是有自旋，请继续往下看。

3）重量级锁

适用于多个线程**同时**获取锁。

##### 14.自旋发生在哪个阶段？

网上99.99%的说法，自旋都是发生在轻量级锁阶段，但是实际看了源码（JDK8）之后，并不是这样的。

**轻量级锁阶段并没有自旋操作**，在轻量级锁阶段，**只要发生竞争**，**就是直接膨胀成重量级锁**。

而在重量级锁阶段，如果获取锁失败，则会尝试**自旋去获取锁**。

##### 15.为什么要设计自旋操作？

因为**重量级锁的挂起开销太大**。todo

一般来说，同步代码块内的代码应该很快就执行结束，这时候竞争锁的线程自旋一段时间是很容易拿到锁的，这样就可以节省了重量级锁挂起的开销。

##### 16.自适应自旋是如何体现自适应的？

自适应自旋锁有自旋次数限制，范围在：1000~5000。

如果当次自旋获取锁成功，则会奖励自旋次数100次，如果当次自旋获取锁失败，则会惩罚扣掉次数200次。

所以如果自旋一直成功，则JVM认为自旋的成功率很高，值得多自旋几次，因此增加了自旋的尝试次数。

相反的，如果自旋一直失败，则JVM认为自旋只是在浪费时间，则尽量减少自旋。

##### 17.synchronized 锁能降级吗？

答案是可以的。

具体的触发时机：在全局安全点（safepoint）中，**执行清理任务的时候**会触发尝试降级锁。

当锁降级时，主要进行了以下操作：

1）**恢复锁对象的 markword 对象头**；

2）**重置 ObjectMonitor**，然后将该 ObjectMonitor 放入全局空闲列表，等待后续使用。

##### 18.synchronized 和 ReentrantLock 的区别

1）底层实现：synchronized 是 Java 中的关键字，是 JVM 层面的锁；ReentrantLock 是 JDK 层次的锁实现。

2）是否需要手动释放：synchronized 不需要手动获取锁和释放锁，在发生异常时，会自动释放锁，因此不会导致死锁现象发生；ReentrantLock 在发生异常时，如果没有主动通过 unLock() 去释放锁，很可能会造成死锁现象，因此使用 ReentrantLock 时需要在 finally 块中释放锁。

3）锁的公平性：synchronized 是非公平锁；ReentrantLock 默认是非公平锁，但是可以通过参数选择公平锁。

4）是否可中断：synchronized 是不可被中断的；ReentrantLock 则可以被中断。

5）灵活性：使用 synchronized 时，等待的线程会一直等待下去，直到获取到锁；ReentrantLock 的使用更加灵活，有立即返回是否成功的，有响应中断、有超时时间等。

6）性能上：随着近些年 synchronized 的不断优化，ReentrantLock 和 synchronized 在性能上已经没有很明显的差距了，所以性能不应该成为我们选择两者的主要原因。官方推荐尽量使用 synchronized，除非 synchronized 无法满足需求时，则可以使用 Lock。

##### 19.认识 Java 对象头

Java 对象头最多由三部分构成：

1. `MarkWord`
2. ClassMetadata Address  [对象的Class信息，存储对象的class地址【元空间中】]
3. Array Length （**如果对象是数组才会有这部分**）

其中 `Markword` 是保存锁状态的关键，对象锁状态可以从偏向锁升级到轻量级锁，再升级到重量级锁，加上初始的无锁状态，可以理解为有 4 种状态。想在一个对象中表示这么多信息自然就要用`位`存储，在 64 位操作系统中，是这样存储的（**注意颜色标记**）,下面是对象头的结构

![img](images/02fb2dfdec534a0ca4a7a8fe24c2b097-164543499216614.png)

##### 20.synchronized 锁升级流程？

核心流程如下图所示，请保存后放大查看，有一些概念看不懂很正常，会在文章后面陆续介绍，请继续往下看。

![img](images/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3YxMjM0MTE3Mzk=,size_16,color_FFFFFF,t_70-164543435632711.png)
