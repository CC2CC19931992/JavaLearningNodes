#### 1.volatile关键字

1.可见性

.MESI
多线程提高效率，本地缓存数据，造成数据修改不可见。
要想保证可见性，要么触发同步指令，要么加上volatile。被volatile修饰的内存，只要有修改，则马上同步到每个线程

2.禁止指令重排序

3.线程的 as-if-serial 看上去是顺序执行的(单线程)

单个线程，两条语句，未必是按顺序执行。

单线程的重排序，必须保证最终一致性。

4.CPU的乱序执行

为什么乱序执行？为了提高效率

5.使用内存屏障组织乱序执行

内存屏障是特殊指令：看到这种指令，前面的必须执行完，后面的才能执行。

intel cpu内存屏障指令:ifence sfence mfence  (CPU特有指令)

6.JVM中的内存屏障

所有实现JVM规范的虚拟机，必须实现四种屏障
LoadLoadBarrier(LL)  LoadStoreBarrier(LS)    StoreLoadBarrier(SL)    StoreStoreBarrier(SS)

load是读 store是写
load load 是读和读指令              load store 是读和写指令 ······

7.volatile禁止指令重排序的底层实现

volatile修饰的内存不可以重排序，对volatile修饰变量的读写访问都不可以换顺序

8.DCL单例要不要加volatile（要） double-check-locking
https://www.bilibili.com/video/BV1Vo4y1U76c?p=8

一般dcl单例的写法

```java
public class Singleton{
    private static Singleton singleton;
    
    private int m = 8;
    
    private Singleton(){}
    
    public static Singleton getInstance(){
        if(singleton==null){
            synchronized(Singleton.class){
                if(singleton==null)  singleton=new Singleton();
            }
              
        }
        return singleton;
    } 
}
```

这个写法会有问题 
1.singleton = new Singleton()的时候,,在字节码里会发生如下操作

```
    LINENUMBER 15 L0
    NEW java/lang/Object    申请内存块(在内存中开辟一个空间) 还未去初始化建立关联,这个时候成员变量m为	0(初始值)
    DUP
    INVOKESPECIAL java/lang/Object.<init> ()V    调用构造方法，这时候singleton才有内容
    ASTORE 2                将singleton 和开辟的内存块建立关联
```

而上面指令 可能会发生重排序
如在线程1中：进入到了  singleton=new Singleton()    执行
 ASTORE  在  INVOKESPECIAL前面执行,这个时候还没走INVOKESPECIAL命令，singleton已经和内存块建立了关联singleton不为空，**但是没有初始化完成**，成员变量m=0
 当其他线程进来发现singleton不为空了，那么就会拿到这个时候的的singleton去使用(**这个singleton未执行构造方法**)。我们需要的成员变量是m=8，显然不符合我们的初衷。
那么 这个时候得到的其他线程得到的对象就是**未执行构造方法的对象**。
所以这就是以上这段代码会出现的问题。

这个时候我们加个volatile关键字 禁止指令重排序 就能解决这个问题 如下

```java
public class Singleton{
    private volatile static Singleton singleton;
    
    private int m = 8;
    
    private Singleton(){}
    
    public static Singleton getInstance(){
        if(singleton==null){
            synchronized(Singleton.class){
                if(singleton==null)  singleton=new Singleton();
            }
        }
        return singleton;
    } 
}
```

