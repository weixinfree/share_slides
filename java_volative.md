title: 简答的程序理解(一)
speaker: 王伟

[slide]

# 线程安全的单例

[slide]

```java

public class Singleton {

    private Singleton() {}

    private static [volatile] Singleton outInstance = null;

    public Singleton getInstance() {
        if (outInstance == null) {
            synchronized(Singleton.this) {
                if (outInstance == null) {
                    outInstance = new Singleton();
                }
            }
        }

        return outInstance;
    }
}
```

[slide]

## 问题
1. 为什么要double check ?
2. 有没有volatile关键字，有区别吗 ？
3. 线程安全吗？能确保多线程下只有一个单例吗？

[slide]

# 三个不同的解读视角

[slide]

# 存储器的结构 👀

[slide]

## 硬件存储器结构

[slide]
![hhh](/cpu_arch/i5_cpu.jpg)

[slide]
![hhh](/cpu_arch/i7_cpu.jpg)

[slide]
![hhh](/cpu_arch/i7_cpu2.jpg)

[slide]

## 思考
1. 为什么多级cache ?
2. 带来的潜在问题 ?
3. 解决方案 ?

----

## 硬件的缓存一致性协议

[内存模型 和 缓存一致性](https://ljalphabeta.gitbooks.io/a-primer-on-memory-consistency-and-cache-coherenc/content/%E7%AC%AC%E4%B8%80%E7%AB%A0-consistency.html)

[slide]

## Java 内存模型 (JMM)

[slide]

![hhh](/cpu_arch/jmm_1.png)

[slide]

[java内存模型与线程](http://foolchild.cn/2016/07/31/jvm_memory_thread)

![hhh](/cpu_arch/jmm_2.png)

[slide]

![hhh](/cpu_arch/jmm_3.png)


[slide]

## 执行顺序视角 👀

1. 重排序
2. `intra-thread semantics`
3. happens-before

[slide]

# 重排序

![hhh](/cpu_arch/reorder.png)

[slide]

## 思考
1. 重排序 why?
2. 带来的潜在问题
3. 解决方案

[slide]

## 在执行程序时为了提高性能，编译器和处理器常常会对指令做重排序。重排序分三种类型：

1. 编译器优化的重排序。编译器在不改变单线程程序语义的前提下，可以重新安排语句的执行顺序。
2. 指令级并行的重排序。现代处理器采用了指令级并行技术（Instruction-Level Parallelism， ILP）来将多条指令重叠执行。如果不存在数据依赖性，处理器可以改变语句对应机器指令的执行顺序。
3. 内存系统的重排序。由于处理器使用缓存和读/写缓冲区，这使得加载和存储操作看上去可能是在乱序执行。

[slide]

1. 1属于编译器重排序，2和3属于处理器重排序, 这些重排序都可能会导致多线程程序出现内存可见性问题。
2. 对于编译器，JMM的编译器重排序规则会禁止特定类型的编译器重排序（不是所有的编译器重排序都要禁止）。
3. 对于处理器重排序，JMM的处理器重排序规则会要求java编译器在生成指令序列时，插入特定类型的内存屏障（memory barriers，intel称之为memory fence）指令，通过内存屏障指令来禁止特定类型的处理器重排序（不是所有的处理器重排序都要禁止）

[slide]

## 参考
[深入理解Java内存模型](http://www.infoq.com/cn/articles/java-memory-model-1)

[slide]

## WTF? 这么复杂还怎么愉快的玩耍？

![wtf](/cpu_arch/wtf.jpeg)

[slide]

## intra-thread semantics

![](/cpu_arch/god.jpeg)

[slide]

## The Java® Language Specification

> The memory model determines what values can be read at every point in the program. The actions of each thread in isolation must behave as governed by the semantics of that thread, with the exception that the values seen by each read are determined by the memory model. When we refer to this, we say that the program obeys `intra-thread semantics`. Intra-thread semantics are the semantics for single-threaded programs, and allow the complete prediction of the behavior of a thread based on the values seen by read actions within the thread. To determine if the actions of thread t in an execution are legal, we simply evaluate the implementation of thread t as it would be performed in a single-threaded context, as defined in the rest of this specification.

[slide]

## 参考

[java 语言规范](https://docs.oracle.com/javase/specs/)
[Chapter 17. Threads and Locks](https://docs.oracle.com/javase/specs/jls/se10/html/jls-17.html#jls-17.4.3)

[slide]

## 单线程是ok了？多线程呢？

![wtf](/cpu_arch/wtf.jpeg)

[slide]

## Happens-before

[slide]

### 17.4.5. Happens-before Order

>Two actions can be ordered by a happens-before relationship. If one action happens-before another, then the first is visible to and ordered before the second.

[slide]


>If we have two actions x and y, we write hb(x, y) to indicate that x happens-before y.

>1. If x and y are actions of the same thread and x comes before y in program order, then hb(x, y).

>2. There is a happens-before edge from the end of a constructor of an object to the start of a finalizer (§12.6) for that object.

>3. If an action x synchronizes-with a following action y, then we also have hb(x, y).

>4. If hb(x, y) and hb(y, z), then hb(x, z).

[slide]

```java

public class Singleton {

    private Singleton() {}

    private static [volatile] Singleton outInstance = null;

    public Singleton getInstance() {
        if (outInstance == null) {
            synchronized(Singleton.this) {
                if (outInstance == null) {
                    outInstance = new Singleton();
                }
            }
        }

        return outInstance;
    }
}
```

[slide]

1. An unlock on a monitor happens-before every subsequent lock on that monitor.
2. A write to a volatile field (§8.3.1.4) happens-before every subsequent read of that field.
3. A call to start() on a thread happens-before any actions in the started thread.
4. All actions in a thread happen-before any other thread successfully returns from a join() on that thread.
5. The default initialization of any object happens-before any other actions (other than default-writes) of a program.

[slide]

1. 程序次序规则：在一个单独的线程中，按照程序代码的执行流顺序，（时间上）先执行的操作happen—before（时间上）后执行的操作。
2. 管理锁定规则：一个unlock操作happen—before后面（时间上的先后顺序，下同）对同一个锁的lock操作。
3. volatile变量规则：对一个volatile变量的写操作happen—before后面对该变量的读操作。
4. 线程启动规则：Thread对象的start（）方法happen—before此线程的每一个动作。
5. 线程终止规则：线程的所有操作都happen—before对此线程的终止检测，可以通过Thread.join（）方法结束、Thread.isAlive（）的返回值等手段检测到线程已经终止执行。
6. 对象终结规则：一个对象的初始化完成（构造函数执行结束）happen—before它的finalize（）方法的开始。
7. 传递性：如果操作A happen—before操作B，操作B happen—before操作C，那么可以得出A happen—before操作C。

[slide]

![hhh](/cpu_arch/total.png)

[slide]

## 抽象视角 👀

[slide]

### 多核cpu/cache体系 对代码的影响

1. 原子性
2. 可见性
3. 有序性

[slide]

## Question: 下面这行代码是线程安全的吗？

```java
private static void demo() {
    int a = 0;
    a++;
}
```

[slide]

> When a program contains two conflicting accesses (§17.4.1) that are not ordered by a happens-before relationship, it is said to contain a data race.

[slide]

## JMM 和 原子性
1. 非数据库的原子性，强调作为一个整体发生
2. JMM 中定义的几个原子指令：read，load，use，assign, store, write, lock, unlock
3. monitorenter，monitorexit <=> (Syncronized)
4. CAS <=> AtomicInteger/Boolean

[slide]

## JMM 和 可见性，有序性

1. happens-before 关系定义的就是 可见性和有序性
2. 数学上的偏序关系, 秩序 => 时间，对称破缺

[slide]

## 回到最初的问题

1. 为什么要double check ?
2. 有没有volatile关键字，有区别吗 ？
3. 线程安全吗？能确保多线程下只有一个单例吗？


[双重检查锁定与延迟初始化](http://www.infoq.com/cn/articles/double-checked-locking-with-delay-initialization)

[slide]

## 废话了这么多，你还要说什么？

![](/cpu_arch/feihua.png)

[slide]

1. 早期jvm -> 锁优化，如何衡量锁消耗
2. JDK 5
3. 永远不要以为多线程简单
4. 使用有把握的那部分特性。kiss 原则的另一个体现
5. 理解一个程序是一件系统性，多层次的事情

[slide]

[jsr33 Java 内存](http://www.cs.umd.edu/~pugh/java/memoryModel/jsr133.pdf)
[java 语言规范](https://docs.oracle.com/javase/specs/jls/se10/html/jls-17.html#jls-17.4.3)
