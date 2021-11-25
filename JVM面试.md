##### 1.JVM体系概述

![image-20211118144251334](images/image-20211118144251334.png)

GC作用域是方法区和堆

##### 2.四种常见垃圾回收算法

引用计数法，复制算法，标记清除算法，标记压缩算法

##### 3.GCRoots的理解

**什么是垃圾？**

简单的说就是内存中已经不再被使用到的空间就是垃圾

**要进行垃圾回收，如何判断一个对象是否可以被回收?**

* 引用计数法（目前已经不用了，难解决对象之间相互循环引用的问题）
* 枚举根节点做可达性分析(根搜索路径)GC Roots

**GC Roots解释**：基本思路就是通过一系列名为”GC Roots”的对象作为起始点，从这个被称为GC Roots的对象开始向下搜索，如果一个对象到GC Roots没有任何引用链相连时，则说明此对象不可用。也即给定一个集合的引用作为根出发，通过引用关系遍历对象图，能被遍历到的（可到达的）对象就被判定为存活;没有被遍历到的就自然被判定为死亡。

![image-20211118150112401](images/image-20211118150112401.png)

**Java中可以作为GC Roots的对象**：

- 虚拟机栈（栈帧中的局部变量区，也叫做局部变量表）中引用的对象。[方法局部变量]
- 方法区中的类静态属性引用的对象。[类中static修饰的对象]
- 方法区中常量引用的对象。[static final修饰的对象]
- 本地方法栈中JNI(Native方法)引用的对象。[]

##### 4.JVM的参数类型(三类)

* 标配参数
  - java -version
  - java -help
* X参数
  - -Xint：解释执行
  - -Xcomp：第一次使用就编译成本地代码
  - -Xmixed：混合模式
* XX参数【重点】

JVM

-Xms
-Xmx
-Xss
......

**其中Xms和Xmx最好配置成一样，防止频繁GC**

##### 5.XX参数--布尔类型

**公式**：-XX:+ 或者 - 某个属性值（+表示开启，-表示关闭）

如何查看一个正在运行中的java程序，它的某个jvm参数是否开启？具体值是多少？

jps -l 查看一个正在运行中的java程序，得到Java程序号。
命令【jinfo -flag PrintGCDetails  java进程号 】查看它的某个jvm参数（如PrintGCDetails ）是否开启。
jinfo -flags (Java程序号 )查看它的所有jvm参数。

**Case：**
是否打印GC收集细节

* -XX:-PrintGCDetails
* -XX:+PrintGCDetails

是否使用串行垃圾回收器

* -XX:-UseSerialGC
* -XX:+UserSerialGC

##### 6.XX参数--KV键值对类型

公式：`-XX:属性key=属性值value`

Case

- -XX:MetaspaceSize=128m 设置元空间大小
  注意 查看的时候 这里的单位是byte。1M=1024✖1KB = 1024✖1024byte
  ![image-20211118164931601](images/image-20211118164931601.png)
- -XX:MaxTenuringThreshold=15 

##### 7.JVM的XX参数之Xms和Xmx坑题

- -Xms等价于-XX:InitialHeapSize，初始的堆内存大小，默认物理内存1/64

- -Xmx等价于-XX:MaxHeapSize，最大堆分配内存，默认为物理内存1/4

  **Xms和Xmx最好配置成一样，防止频繁GC**

##### 8.JVM盘点家底查看初始默认值

* **查看初始默认参数值**：-XX:+PrintFlagsInitial

  公式：`java -XX:+PrintFlagsInitial`

  这个命令**不需要运行java程序**就能看到jvm的出厂默认参数。

  ![image-20211119100011364](images/image-20211119100011364.png)

* **查看修改更新的参数**：-XX:+PrintFlagsFinal

  公式：`java -XX:+PrintFlagsFinal`

  ![image-20211119100421153](images/image-20211119100421153.png)

  =表示默认，:=表示修改过的。

  T是一个.java的文件

  ![image-20211119101956443](images/image-20211119101956443.png)

  运行如下命令:
  java -XX:+PrintFlagsFinal -XX:MetaspaceSize=512m T

  可以找到修改后的MetaspaceSize的修改后的值
  ![image-20211119102101779](images/image-20211119102101779.png)

* **打印命令行参数**：-XX:+PrintCommandLineFlags

  运行程序的时候会在(idea是debug窗口)命令行上出来
  公式：java -XX:+PrintCommandLineFlags 

![image-20211119102646076](images/image-20211119102646076.png)

##### 9.JVM常见参数之 堆

-Xmn 是Young Gen的配置(年轻代内存大小)

-Xms 和-Xmx上面有讲过，是JVM Heap的内存大小配置

##### 10.堆内存

在Java8中，永久代已经被移除，被一个称为**元空间**的区域所取代。**元空间的本质和永久代类似**。

元空间(Java8)与永久代(Java7)之间最大的区别在于：永久带使用的JVM的堆内存，但是Java8以后的**元空间并不在虚拟机中而是使用本机物理内存**。

![image-20211119110943226](images/image-20211119110943226.png)

##### 11.元空间(永久代)与方法区

**元空间(永久带)**是方法区的一种实现，其中存储类模板信息，常量以及静态变量。

元空间中一般包含：
	1 类的方法(字节码...)
	2 类名(Sring对象)
	3 .class文件读到的常量信息
	4 class对象相关的对象列表和类型列表 (e.g., 方法对象的array).
	5 JVM创建的内部对象
	6 JIT编译器优化用的信息

**由于元空间是使用的本机物理内存，所以垃圾回收对元空间无作用**

##### 12.Xss参数讲解

-Xss等价于 -XX:ThreadStackSize
设置单个线程栈的大小，一般默认为512k~1024K

##### 13.Xmn参数

设置年轻代大小，一般是用默认就行

##### 14.MetaspaceSize

-XX:MetaspaceSize 设置元空间大小

元空间的本质和永久代类似，都是对JVM规范中方法区的实现。不过元空间与永久代之间最大的区别在于：**元空间并不在虚拟机中，而是使用本地内存**。因此，默认情况下，元空间的大小仅受本地内存限制。

##### 15.参数典型设置案例

-Xms128m -Xmx4096m -Xss1024k -XX:MetaspaceSize=512m -XX:+PrintCommandLineFlags -XX:+PrintGCDetails-XX:+UseSerialGC

##### 16.PrintGCDetails参数调试

-XX:+PrintGCDetails 输出详细GC收集日志信息

设置参数 -Xms10m -Xmx10m -XX:+PrintGCDetails 运行以下程序

```java
import java.util.concurrent.TimeUnit;

public class PrintGCDetailsDemo {

public static void main(String[] args) throws 	 		InterruptedException {
		byte[] byteArray = new byte[10 * 1024 * 1024];
		
		TimeUnit.SECONDS.sleep(Integer.MAX_VALUE);
	}
}
```

![image-20211119121753804](images/image-20211119121753804.png)

关于普通GC的分析如下图：

![image-20211119152846757](images/image-20211119152846757.png)

关于Full GC的分析 如下图：

![image-20211119121718612](images/image-20211119121718612.png)

关于垃圾回收器的

![image-20211125113723139](images/image-20211125113723139.png)

##### 17.SurvivorRatio参数

![img](images/5799217d4beece00818129f315ac4dab.png)

调节**新生代**中 eden 和 S0、S1的空间比例，默认为 -XX:SuriviorRatio=8，Eden:S0:S1 = 8:1:1

假如设置成 -XX:SurvivorRatio=4，则为 Eden:S0:S1 = 4:1:1

SurvivorRatio值就是设置eden区的比例占多少，S0和S1相同.

**from区和to区 谁空谁是to**

##### 18.NewRatio参数

配置年轻代new 和老年代old 在堆结构的占比

默认：-XX:NewRatio=2 新生代占1，老年代2，年轻代占整个堆的1/3

-XX:NewRatio=4：新生代占1，老年代占4，年轻代占整个堆的1/5，

NewRadio值就是设置老年代的占比，剩下的1个新生代。

新生代特别小，会造成频繁的进行GC收集。

##### 19.MaxTenuringThreshold参数

晋升到老年代的对象年龄。

在一次minorGC时，SurvivorTo和SurvivorFrom互换，原SurvivorTo成为下一次GC时的SurvivorFrom区，部分对象会在From和To区域中复制来复制去，如此交换15次[15次minorGC]（由JVM参数MaxTenuringThreshold决定，这个参数默认为15），最终如果还是存活，就存入老年代。

在MinorGC开始的时候，对象只会存在于Eden区和名为“From”的Survivor区，Survivor区“To”是空的。紧接着进行GC，Eden区中所有存活的对象都会被复制到“To”，而在“From”区中，仍存活的对象会根据他们的年龄值来决定去向。年龄达到一定值(年龄阈值，可以通过-XX:MaxTenuringThreshold来设置)的对象会被移动到年老代中，没有达到阈值的对象会被复制到“To”区域。经过这次GC后，Eden区和From区已经被清空。这个时候，**“From”和“To”会交换他们的角色**，也就是新的“To”就是上次GC前的“From”，新的“From”就是上次GC前的“To”。不管怎样，都会保证名为To的Survivor区域是空的。Minor GC会一直重复这样的过程，直到“To”区被填满，“To”区被填满之后，会将所有对象移动到年老代中。

这里就是调整这个次数的，**默认是15**，并且**设置的值一定要在 0~15之间**。

-XX:MaxTenuringThreshold=0：设置垃圾最大年龄。如果设置为0的话，则年轻对象不经过Survivor区，直接进入老年代。对于年老代比较多的应用，可以提高效率。如果将此值设置为一个较大的值，则年轻代对象会在Survivor区进行多次复制，这样可以增加对象再年轻代的存活时间，增加在年轻代即被回收的概念

##### 20.MaxDirectMemorySize参数

公式：-XX:MaxDirectMemorySize=5m

解释：给java虚拟机堆之外的直接内存分配【元空间就是在这块上面】，NIO中的ByteBuffer.allocateDirect(capability)方法也是分配OS本地内存，不属于GC管辖范围，由于不需要内存拷贝所以速度相对较快。





##### 21.垃圾收集器种类

GC算法(引用计数/复制(新生代)/标清(老年代)/标整(老年代))是内存回收的方法论，垃圾收集器就是算法落地实现

因为目前为止还没有完美的收集器出现，更加没有万能的收集器，只是针对具体应用最合适的收集器，进行分代收集

4种**主要**垃圾收集器Serial[串行]，Parallel[并行回收]，CMS[并发标记清楚]，G1

![img](images/66a41f86a94641626e78e1278b7b2de0.png)

* 串行垃级回收器(Serial) - 它为单线程环境设计且值使用**一个线程**进行垃圾收集，会**暂停所有的用户线程**，**只有当垃圾回收完成时，才会重新唤醒主线程继续执行**。所以**不适合**服务器环境。会产生STW
* 并行垃圾回收器(Parallel) - **多个垃圾收集线程**并行工作，**此时用户线程也是阻塞的**，适用于科学计算 / 大数据处理等弱交互场景，也就是说Serial 和 Parallel其实是类似的，不过是多了几个线程进行垃圾收集，但是主线程都会被暂停，但是**并行垃圾收集器处理时间，肯定比串行的垃圾收集器要更短**。会产生STW
* 并发垃圾回收器(CMS) - **用户线程和垃圾收集线程同时执行**（不一定是并行，可能是交替执行），不需要停顿用户线程，互联网公司都在使用，适用于响应时间有要求的场景。并发标记清楚，会产生内存碎片
* G1垃圾回收器 - G1垃圾回收器将堆内存分割成不同的区域然后并发的对其进行垃圾回收。
* ZGC（Java 11的，了解）
  ![img](images/35b6e31d8ca498dd287e890bc0f362ea.png)



##### 22.如何查看默认的垃圾收集器

java -XX:+PrintCommandLineFlags -version 

输出结果

```
C:\Users\abc>java -XX:+PrintCommandLineFlags -version
-XX:InitialHeapSize=266613056 -XX:MaxHeapSize=4265808896 -XX:+PrintCommandLineFlags -XX:+UseCompressedClassPointers -XX:+UseCompressedOops -XX:-UseLargePagesIndividualAllocation -XX:+UseParallelGC
java version "1.8.0_251"
Java(TM) SE Runtime Environment (build 1.8.0_251-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.251-b08, mixed mode)
```

从结果看到-XX:+UseParallelGC，也就是说默认的垃圾收集器是并行垃圾回收器。

或者用 jps -l

然后得出Java程序号 

再用这个命令 jinfo -flags (Java程序号)

##### 23.JVM默认的垃圾收集器有哪些

Java中一共有7大垃圾收集器

年轻代GC：

* UserSerialGC：串行垃圾收集器
* UserParallelGC：并行垃圾收集器
* UseParNewGC：年轻代的并行垃圾回收器

老年代GC：

* UserSerialOldGC：串行老年代垃圾收集器（已经被移除）
* UseParallelOldGC：老年代的并行垃圾回收器
* UseConcMarkSweepGC：（CMS）并发标记清除

老嫩通吃：

* UseG1GC：G1垃圾收集器

不同厂商、不同版本的虚拟机实现差别很大，HotSpot中包含的收集器如下图所示：

![image-20211124152240631](images/image-20211124152240631.png)

上图中打红×的年轻代和老年代是不推荐使用的组合。

##### 24.GC之约定参数说明 以及 Server/Client模式

- DefNew：Default New Generation  新生代默认GC  也就是 串行GC
- Tenured：Old   (老年代默认GC,也就是Serial Old)
- ParNew：Parallel New Generation
- PSYoungGen：Parallel Scavenge
- ParOldGen：Parallel Old Generation

Server/Client模式分别是什么意思？

Server启动慢，运行快；Client启动快，运行慢

使用范围：一般使用Server模式，Client模式基本不会使用

操作系统：

* 32位的Window操作系统，不论硬件如何都默认使用Client的JVM模式

* 32位的其它操作系统，2G内存同时有2个cpu以上用Server模式，低于该配置还是Client模式

* 64位只有Server模式

```
C:\Users\abc>java -version
java version "1.8.0_251"
Java(TM) SE Runtime Environment (build 1.8.0_251-b08)
Java HotSpot(TM) 64-Bit Server VM (build 25.251-b08, mixed mode)
```

##### 25.GC之Serial收集器

一句话：一个单线程的收集器，在进行垃圾收集时候，必须暂停其他所有的工作线程直到它收集结束。

![img](images/a2f9de5e9b485247a7daa7b343b70264.png)

STW: Stop The World

串行收集器是最古老，最稳定以及效率高的收集器，只使用一个线程去回收但其在进行垃圾收集过程中可能会产生较长的停顿（Stop-The-World状态)。虽然在收集垃圾过程中需要暂停所有其他的工作线程，但是它简单高效，对于限定单个CPU环境来说，**没有线程交互的开销可以获得最高的单线程垃圾收集效率**，因此Serial垃圾收集器依然是java虚拟机运行在Client模式下默认的新生代垃圾收集器。

**对应JVM参数是：-XX:+UseSerialGC**

**开启后会使用：Serial(Young区用) + Serial Old(Old区用)的收集器组合**

表示：新生代、老年代都会使用串行回收收集器，新生代使用复制算法，老年代使用标记-整理算法

```java
public class GCDemo {

	public static void main(String[] args) throws InterruptedException {
		
		Random rand = new Random(System.nanoTime());
		
		try {
			String str = "Hello, World";
			while(true) {
				str += str + rand.nextInt(Integer.MAX_VALUE) + rand.nextInt(Integer.MAX_VALUE);
			}
		}catch (Throwable e) {
			e.printStackTrace();
		}
		
	}

}
```

VM参数：（启用UseSerialGC）

-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseSerialGC

上面程序运行的结果：

![image-20211124171858054](images/image-20211124171858054.png)

##### 26.GC之ParNew收集器

一句话：使用多线程进行垃圾回收，在垃圾收集时，会Stop-The-World暂停其他所有的工作线程直到它收集结束。

![img](images/25405deebf37fd1517467ef47cc2a1ea.png)

ParNew收集器其实就是Serial收集器**新生代**的并行多线程版本，最常见的应用场景是配合老年代的**CMS** GC工作，其余的行为和Seria收集器完全一样，ParNew垃圾收集器在垃圾收集过程中同样也要**暂停所有其他的工作线程**。它是很多Java虚拟机运行在Server模式下**新生代**的**默认垃圾收集器**。

**常用对应JVM参数**：-XX:+UseParNewGC启用ParNew收集器，只影响新生代的收集，不影响老年代。

**开启上述参数后**，会使用：ParNew(Young区)+ Serial Old的收集器组合，新生代使用复制算法，老年代采用标记-整理算法

但是，ParNew+Tenured这样的搭配，**Java8已经不再被推荐**

备注：-XX:ParallelGCThreads限制开启的GC线程数量，默认开启和CPU数目相同的线程数。

vm参数：-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseParNewGC 

![image-20211124181723020](images/image-20211124181723020.png)



##### 27.GC之Parallel收集器

![img](images/42f9a757b25c0d6f8f1d86d5eaddf398.png)

Parallel Scavenge收集器类似ParNew也是一个新生代垃圾收集器，**使用复制算法[新生代]**，也是一个并行的多线程的垃圾收集器，俗称吞吐量优先收集器。一句话：**串行收集器在新生代和老年代的并行化**。

它重点关注的是：可控制的吞吐量(Thoughput=运行用户代码时间(运行用户代码时间+垃圾收集时间),也即比如程序运行100分钟，垃圾收集时间1分钟，吞吐量就是99% )。高吞吐量意味着**高效利用CPU的时间**，它多用于在**后台运算而不需要太多交互的任务**。

自适应调节策略也是ParallelScavenge收集器与ParNew收集器的一个重要区别。(自适应调节策略:虚拟机会根据当前系统的运行情况收集性能监控信息，动态调整这些参数以提供最合适的停顿时间（-XX:MaxGCPauseMillis）或最大的吞吐量。

常用JVM参数：-XX:+UseParallelGC或-XX:+UseParallelOldGC（**可互相激活**）使用Parallel Scanvenge收集器。

开启该参数后：新生代使用复制算法，老年代使用标记-整理算法。

多说一句：-XX:ParallelGCThreads=数字N 表示启动多少个GC线程

* cpu>8 N= 5/8
* cpu<8 N=实际个数

-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseParallelGC

##### 28.GC之ParallelOld收集器

Parallel Old收集器是Parallel Scavenge的老年代版本，使用多线程的**标记-整理算法**，Parallel Old收集器在JDK1.6才开始提供。

在JDK1.6之前，新生代使用ParallelScavenge收集器只能搭配年老代的Serial Old收集器，只能保证新生代的吞吐量优先，无法保证整体的吞吐量。在JDK1.6之前（Parallel Scavenge + Serial Old )

Parallel Old 正是为了在年老代同样提供吞吐量优先的垃圾收集器，如果系统对吞吐量要求比较高，JDK1.8后可以优先考虑新生代Parallel Scavenge和年老代Parallel Old收集器的搭配策略。在JDK1.8及后〈Parallel Scavenge + Parallel Old )

JVM常用参数：-XX:+UseParallelOldGC 或 -XX:+UseParallelGC （**可互相激活**）使用Parallel Old收集器，设置该参数后，新生代Parallel+老年代Parallel Old。

-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseParallelOldGC

##### 29.GC之CMS收集器

CMS收集器(Concurrent Mark Sweep：**并发标记清除**）**是一种以获取最短回收停顿时间为目标的收集器**。

适合应用在互联网站或者**B/S**系统的服务器上，这类应用尤其**重视服务器的响应速度，希望系统停顿时间最短**。

CMS非常适合堆内存大、CPU核数多的服务器端应用，也是G1出现之前大型应用的首选收集器。
![img](images/59c716f3ebb5070f67062ea8094a5266.png)

Concurrent Mark Sweep并发标记清除，**并发收集低停顿,**并发指的是与用户线程一起执行
开启该收集器的JVM参数：-XX:+UseConcMarkSweepGC开启该参数后会**自动**将-XX:+UseParNewGC打开。

开启该参数后，使用**ParNew（Young区用）+ CMS（Old区用）+ Serial Old**的收集器组合，**Serial Old将作为CMS出错的后备收集器**。

###### 四步过程：

初始标记（CMS initial mark） - 只是标记一下GC Roots能直接关联的对象，**速度很快**，**仍然需要暂停所有的工作线程**。

并发标记（CMS concurrent mark）和用户线程一起 - 进行GC Roots跟踪的过程，和用户线程一起工作，**不需要暂停工作线程**。主要标记过程，标记全部对象。

重新标记（CMS remark）- **为了修正在并发标记期间，因用户程序继续运行而导致标记产生变动的那一部分对象的标记记录**，仍然**需要暂停所有的工作线程**。由于并发标记时，用户线程依然运行，因此在正式清理前，再做修正。就怕第二个过程中(并发标记)用户线程让初始标记要清楚的对象又用了起来，所以需要重新检查标记下。

并发清除（CMS concurrent sweep） - **清除GCRoots不可达对象，和用户线程一起工作**，**不需要暂停工作线程**。基于标记结果，直接清理对象，由于耗时最长的并发标记和并发清除过程中，**垃圾收集线程可以和用户现在一起并发工作**，所以总体上来看**CMS 收集器的内存回收和用户线程是一起并发地执行**。

![img](images/232c6da9405df5933336be71228c2bb9.png)

优点：并发收集低停顿。

缺点：并发执行，对CPU资源压力大，采用的标记清除算法会导致大量碎片。

由于并发进行，CMS在收集与应用线程会同时会增加对堆内存的占用，也就是说，CMS必须要在老年代堆内存用尽之前完成垃圾回收，否则CMS回收失败时，将触发担保机制，串行老年代收集器将会以STW的方式进行一次GC，从而造成较大停顿时间。

**标记清除算法无法整理空间碎片**，老年代空间会随着应用时长被逐步耗尽，最后将不得不通过**担保机制(Serial Old垃圾收集器)**对堆内存进行压缩。CMS也提供了参数-XX:CMSFullGCsBeForeCompaction(默认O，即每次都进行内存整理)来指定多少次CMS收集之后，进行一次压缩的Full GC。

复用之前的GCDemo

vm参数启动CMS垃圾收集：-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseConcMarkSweepGC 

##### 30.GC之SerialOld收集器

Serial Old是Serial垃圾收集器老年代版本，它同样是个单线程的收集器，使用标记-整理算法，这个收集器也主要是运行在 Client默认的java虚拟机默认的年老代垃圾收集器。

在Server模式下，主要有两个用途(了解，版本已经到8及以后):

* 在JDK1.5之前版本中与新生代的Parallel Scavenge 收集器搭配使用。(Parallel Scavenge + Serial Old )
* 作为老年代版中使用CMS收集器的后备垃圾收集方案。

vm参数：-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseSerialOldGC 

注意：**在Java8中，-XX:+UseSerialOldGC不起作用**

##### 31.GC之如何选择垃圾收集器

组合的选择：

* 单CPU或者小内存，单机程序
  * -XX:+UseSerialGC
* 多CPU，需要最大的吞吐量，如后台计算型应用
  * -XX:+UseParallelGC（这两个相互激活）
  * -XX:+UseParallelOldGC
* 多CPU，追求低停顿时间，需要快速响应如互联网应用
  * -XX:+UseConcMarkSweepGC
  * -XX:+ParNewGC

| **参数**                                 | 新生代垃圾收集器         | 新生代算法                | 老年代垃圾收集器                                             | 老年代算法 |
| ---------------------------------------- | ------------------------ | ------------------------- | ------------------------------------------------------------ | ---------- |
| -XX:+UseSerialGC                         | SerialGC                 | 复制                      | SerialOldGC                                                  | 标记整理   |
| -XX:+UseParNewGC                         | ParNew                   | 复制                      | SerialOldGC                                                  | 标记整理   |
| -XX:+UseParallelGC/-XX:UserParallelOldGC | Parallel Scavenge        | 复制                      | Parallel Old                                                 | 标记整理   |
| -XX:+UseConcMarkSweepGC                  | ParNew                   | 复制                      | CMS + Serial Old的收集器组合，Serial Old作为CMS出错的后备收集器 | 标记清除   |
| -XX:+UseG1GC                             | G1整体上采用标记整理算法 | 局部复制,不会产生内存随便 |                                                              |            |

##### 32.G1垃圾回收器概述

借用之前的CGDemo

将启动参数变更为：-Xms10m -Xmx10m -XX:+PrintGCDetails -XX:+PrintCommandLineFlags -XX:+UseG1GC

注意控制台输出的结果：

只有一个堆信息和元空间 不像之前的垃圾回收器有区分新生代和老年代以及元空间

![image-20211125151225847](images/image-20211125151225847.png)

###### 以前收集器特点：

- 年轻代和老年代是各自独立且连续的内存块；
- 年轻代收集使用单eden+s0+s1进行复机算法；
- 老年代收集必须扫描整个老年代区域；
- 都是以尽可能少而快速地执行GC为设计原则。

###### G1是什么

G1 (Garbage-First）收集器，是一款面向服务端应用的收集器：

从官网的描述中，我们知道G1是一种服务器端的垃圾收集器，应用在多处理器和大容量内存环境中，在实现**高吞吐量**的同时，尽可能的**满足垃圾收集暂停时间变短**的要求。另外，它还具有以下特性:

* 像CMS收集器一样，能与应用程序线程并发执行。

* 整理空闲空间更快。

* 需要更多的时间来预测GC停顿时间。

* 不希望牺牲大量的吞吐性能。

* 不需要更大的Java Heap。

**G1收集器的设计目标是取代CMS收集器**，它同CMS相比，在以下方面表现的更出色：

* G1是一个有整理内存过程的垃圾收集器，不会产生很多内存碎片。
* G1的Stop The World(STW)更可控，**G1在停顿时间上添加了预测机制**，**用户可以指定期望停顿时间**。

CMS垃圾收集器虽然减少了暂停应用程序的运行时间，但是它还是存在着内存碎片问题。于是，为了去除内存碎片问题，同时又保留CMS垃圾收集器低暂停时间的优点，JAVA7发布了一个新的垃圾收集器-G1垃圾收集器。

G1是在2012年才在jdk1.7u4中可用。oracle官方计划在JDK9中将G1变成默认的垃圾收集器以替代CMS。它是一款面向服务端应用的收集器，主要应用在多CPU和大内存服务器环境下，极大的减少垃圾收集的停顿时间，全面提升服务器的性能，逐步替换java8以前的CMS收集器。

主要改变是Eden，Survivor和Tenured等内存区域**不再是连续的了**，而是变成了一个个大小一样的region ,每个region从1M到32M不等。**一个region有可能属于Eden，Survivor或者Tenured内存区域**。

**特点：**

* G1能充分利用多CPU、多核环境硬件优势，尽量缩短STW。
* G1整体上采用**标记-整理算法**，**局部是通过复制算法**，**不会产生内存碎片**。
* **宏观上**看G1之中**不再区分年轻代和老年代**。把内存划分成多个独立的子区域(Region)，可以近似理解为一个围棋的棋盘。
* G1收集器里面讲整个的内存区都混合在一起了，**但其本身依然在小范围内要进行年轻代和老年代的区分**，保留了新生代和老年代，**但它们不再是物理隔离的**，而是一部分Region的集合且不需要Region是连续的，**也就是说依然会采用不同的GC方式来处理不同的区域**。
* **G1虽然也是分代收集器，但整个内存分区不存在物理上的年轻代与老年代的区别**，也不需要完全独立的**survivor(to space)**堆做复制准备。**G1只有逻辑上的分代概念**，或者说每**个分区都可能随G1的运行在不同代之间前后切换**。

##### 33.G1的底层原理

Region区域化垃圾收集器 - 最大好处是化整为零，避免全内存扫描，只需要按照区域来进行扫描即可。

区域化内存划片Region，整体编为了一些列不连续的内存区域，**避免了全内存区的GC操作**。

核心思想是**将整个堆内存区域分成大小相同的子区域(Region)**，在JVM启动时会自动设置这些子区域的大小，在堆的使用上，G1并不要求对象的存储一定是物理上连续的只要逻辑上连续即可，每个分区也不会固定地为某个代服务，可以按需在年轻代和老年代之间切换。**启动时可以通过参数-XX:G1HeapRegionSize=n可指定分区大小（1MB~32MB，且必须是2的幂），默认将整堆划分为2048个分区**。

**大小范围在1MB~32MB，最多能设置2048个区域**，也即能够支持的最大内存为：32 M B ∗ 2048 = 65536 M B = 64 G 32MB*2048=65536MB=64G32MB∗2048=65536MB=64G内存

![img](images/b804a13751168e17c42652f585b12772.png)

G1算法将堆划分为若干个区域(Region），它仍然属于**分代收集器**。

这些Region的一部分包含新生代，新生代的垃圾收集**依然采用暂停所有应用线程的方式**，**将存活对象拷贝到老年代或者Survivor空间**。

这些Region的一部分包含老年代，G1收集器通过将对象从一个区域复制到另外一个区域，完成了清理工作。这就意味着，在正常的处理过程中，G1完成了堆的压缩（至少是部分堆的压缩），这样也就不会有CMS内存碎片问题的存在了。

在G1中，还有一种特殊的区域，叫**Humongous区域**。

如果一个对象占用的空间超过了**分区容量50%以上**，G1收集器就认为这是一个巨型对象。这些巨型对象默认直接会被分配在年老代，但是如果它是一个短期存在的巨型对象，就会对垃圾收集器造成负面影响。

为了解决这个问题，G1划分了一个**Humongous区**，它用来专门存放**巨型对象**。**如果一个H区装不下一个巨型对象，那么G1会寻找连续的H分区来存储**。为了能找到连续的H区，有时候不得不启动**Full GC**。

###### 回收步骤：

1.G1收集器下的Young GC

针对Eden区进行收集，Eden区耗尽后会被触发，主要是小区域收集＋形成连续的内存块，避免内存碎片

* **Eden区的数据移动到Survivor区，假如出现Survivor区空间不够，Eden区数据会部会晋升到Old区**。
* **Survivor区的数据移动到新的Survivor区，部会数据晋升到Old区。**
* 最后Eden区收拾干净了，GC结束，用户的应用程序继续执行
  

![img](images/4a747a577ee9ffb9d4f22b3ad883ca48.png)

![img](images/868f99e06ca822ccc04d197f88067969.png)

###### 四步过程：

1. 初始标记：只标记GC Roots能直接关联到的对象
2. 并发标记：进行GC Roots Tracing的过程
3. 最终标记：修正并发标记期间，因程序运行导致标记发生变化的那一部分对象
4. 筛选回收：根据时间来进行价值最大化的回收

![img](images/388dfd48e99e82bd025f6b33f4e41ff5.png)



##### 34.G1参数配置及和CMS的比较

* -XX:+UseG1GC
* -XX:G1HeapRegionSize=n：设置的G1区域的大小。值是2的幂，范围是1MB到32MB。目标是根据最小的Java堆大小划分出约2048个区域。
* -XX:MaxGCPauseMillis=n：最大GC停顿时间，这是个软目标，JVM将尽可能（但不保证）停顿小于这个时间。
* -XX:InitiatingHeapOccupancyPercent=n：堆占用了多少的时候就触发GC，默认为45。
* -XX:ConcGCThreads=n：并发GC使用的线程数。
* -XX:G1ReservePercent=n：设置作为空闲空间的预留内存百分比，以降低目标空间溢出的风险，默认值是10%。

开发人员仅仅需要声明以下参数即可：

三步归纳：开始G1+设置最大内存+设置最大停顿时间

* -XX:+UseG1GC
* -Xmx32g
* -XX:MaxGCPauseMillis=100

-XX:MaxGCPauseMillis=n：最大GC停顿时间单位毫秒，这是个软目标，JVM将尽可能（但不保证）停顿小于这个时间

**G1和CMS比较**

* G1不会产生内碎片
* 是可以精准控制停顿。该收集器是把整个堆（新生代、老年代）划分成多个固定大小的区域，每次根据允许停顿的时间去收集垃圾最多的区域。

##### 35.垃圾回收小总结：

**1.垃圾对象的判断**：引用计数器；可达性分析(虚拟机栈-局部变量，方法区类属性和常量所引用的对象，本地方法栈所引用的，都可作为GCroot)

**2.回收策略**：标记清除(效率差，存在内存碎片),复制(没有碎片，但浪费空间),标记整理(没有碎片，需要移动对象),分代().

**3.垃圾回收器**：Serial(单线程GC,会STW), ParNew(多线程的GC,会STW), Parallel-Scavenge(多线程同parNew,不过它关心的是吞吐量，用户代码time/(用户time+GC time)), CMS(三阶段a初始标记,STW，b并行标记，c重新标记并清除，STW-----占用cpu,空间碎片，并发异常),G1(并发和并行，分代，空间整合，可预测停顿),ZGC(jdk11).

##### 36.JVMGC结合SpringBoot微服务优化简介

1. IDEA开发微服务工程。
2. Maven进行clean package。
3. 要求微服务启动的时候，同时配置我们的JVM/GC的调优参数。
4. 公式：`java -server jvm的各种参数 -jar 第1步上面的jar/war包名`。
