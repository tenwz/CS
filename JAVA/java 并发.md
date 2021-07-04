# 并发

## JMM

- 对象头中的并发结构
  - Mark Word
    - 无锁：对象HashCode，分代年龄，偏向锁标识位0，锁标识位01
    - 偏向锁：线程ID，Epoch，分代年龄，偏向锁标识位1，锁标识位01 
    - 轻量级锁：指向栈中锁记录的指针，锁标识位00
    - 重量级锁：指向互斥量（重量级锁）的指针（指向monitor对象地址（每个对象都有ObjectMonitor与之关联）），锁标识位10
    - GC标记：空，锁标识位 11
  - Klass Pointer

## Synchronized

```c++
//反映出Java中任意对象可以作为锁的原因,同时也是notify/notifyAll/wait等方法存在于顶级对象Object中的原因
ObjectMonitor() {
    _count        = 0; //记录数
    _recursions   = 0; //锁的重入次数
    _owner        = NULL; //指向持有ObjectMonitor对象的线程 
    _WaitSet      = NULL; //调用wait后，线程会被加入到_WaitSet
    _EntryList    = NULL ; //等待获取锁的线程，会被加入到该列表
}
```

- 获取过程

  - monitorenter 和 monitorexit ， monitorenter 指向同步代码块的开始位置，monitorexit指向结束位置。 执行 monitorenter 指令时，线程试图获取 monitor对象
  - 同步方法：flags中：ACC_SYNCHRONIZED 标明同步方法

- 竞争过程

  - <u>新建(new)、就绪(runnable)、运行(running)、堵塞(blocked)、死亡(dead)</u>。
  - 多线程竞争，失败线程放进EntryList队列，线程处于blocked状态
  - 线程获取到对象的monitor，进入running状态，执行方法，ObjectMonitor对象的/owner指向当前线程，count加1表示当前对象锁被一个线程获取
  - running状态的线程调用wait()方法，当前线程释放monitor对象，进入waiting状态，ObjectMonitor对象的/owner变为null，count减1，线程进入WaitSet队列，直到有线程调用notify()方法唤醒，该线程进入EntryList队列，竞争到锁再进入Owner区
  - 如果当前线程执行完毕，那么也释放monitor对象，ObjectMonitor对象的/_owner变为null，_count减1

- 锁升级过程

  - 偏向锁（背景：一个线程多次获得同一个锁）

    - 偏向锁 <u>不主动释放锁</u>，竞争锁对象时，先比较当前先线程threadId 和 对象头中 threadId 是否一致。一致，进入；不一致，判断对象头线程是否存活。不存活，置为无锁，重新竞争；存活，查找这个占用线程的栈帧信息，判断是否持有这个对象。持有，暂停占用线程，撤销偏向锁，升级为轻量级锁；不持有，设为无锁，重新竞争。

    - 人话：如果锁正被其他线程使用，则竞争失败，升级为轻量级锁

  - 轻量级锁（背景：自选一会儿等待释放）

    - 复制锁对象的整个Mark Word 到该线程的栈帧中（DisplacedMarkWord），CAS把对象头的内容替换为，该线程 DisplacedMarkWord 的地址。CAS失败，不断CAS自旋。自旋次数过多 || 自旋时又有线程来竞争，升级为重量级锁（阻塞所有线程）

- 锁消除

  - 消除锁是虚拟机另外一种锁的优化，这种优化更彻底，Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间，如下StringBuffer的append是一个同步方法，但是在add方法中的StringBuffer属于一个局部变量，并且不会被其他线程所使用，因此StringBuffer不可能存在共享资源竞争的情景，JVM会自动将其锁消除。

- case ：为什么需要可重入 ？TODO

## volatile

- happen-before
  - volatile变量规则：对一个变量的写操作先行发生于后面对这个变量的读操作（分布式场景？）
- 原理
  - volatile的变量进行写操作时，会在前面加上lock指令前缀。
    - lock 指令：先对总线/缓存加锁，然后执行后面的指令，最后释放锁后把高速缓存中的脏数据全部刷新回主内存（可见性）
      - 嗅探机制：每个处理器会通过<u>**嗅探器**</u>来监控总线上的数据来检查自己缓存内的数据是否过期，如果发现自己缓存行对应的地址被修改了，就会将此缓存行置为无效。当处理器对此数据进行操作时，就会重新从主内存中读取数据到缓存行。
      - 缓存一致性协议：modified（修改）、exclusive（互斥）、share（共享）、invalid（无效）E -> S -> M -> I
      - 缓存一致性流量：处理器都是靠一条总线来和主内存进行数据交互。某一处理器更新了此共享数据后，通过总线触发嗅探机制来通知其他处理器将自己高速缓存内的共享数据置为无效，在下次使用时重新从主内存加载最新数据。这种通过总线来进行通信则称之为”缓存一致性流量“。
      - 总线风暴：CAS 在多核CPU下，指令为 lock +  cmpxchg ;volatile和cas使用过多，导致缓存一致性流量增大，称为总线风暴。
    - 内存屏障 (防止指令重排)
      -  load：作用于工作内存，主内存传递来的值赋给工作内存工作变量
      -  store：作用于工作内存的变量，工作内存工作变量传送到主内存中。
      - 在每个volatile写操作前插入StoreStore屏障，在写操作后插入StoreLoad屏障
        在每个volatile读操作前插入LoadLoad屏障，在读操作后插入LoadStore屏障；
- case
  - 指令重排序
    - 双重检查锁
      - new 步骤 
        - 分配内存空间 2. 初始化对象 3. 将引用指向对象内存地址（存疑）
        - 重排可能出现 132 
        - 第二次判断时因为内存地址已分配，导致可能直接返回一个未初始化完成的对象引用。
  - 可见性
    - synchronized 亦能保证
    - 场景：<u>while重循环</u>（JIT）无法判断多线程下条件flag更改，引入 volatile

## ThreadPool

## ThreadLocal

### 设计

1. ThreadLocal实例本身不存值，它只提供一个在当前线程中找到副本值的key。
2. 每个线程以ThreadLocal作为引用，在自己的map里找对应的key，从而实现了线程隔离

源码：Thread#ThreadLocalMap(本质是操作 Entry数组) -> key : ThreadLocal value: Entry key:ThreadLocal(WeakReference) realValue

- 父子线程共享数据
  - interitableThreadLocals

### 问题

- 内存泄漏：
  - 弱引用：被弱引用关联的对象只能生存到下一次垃圾收集发生之前
    - 使用弱引用使 threadlocal 对象不发生内存泄漏（强引用使threadlocal置null依然无法回收）
    - 若线程不销毁（线程池），Entry 中的 key 为 null 的 Value 对象不会被回收（存在强引用），会导致内存泄漏
  - 解决办法：
    - set，get，remove 都会清除此线程 ThreadLocalMap 里 Entry 数组中所有 Key 为 null 的 Value
    - 线程使用完 threadlocal 后，通过调用 ThreadLocal 的 remove 方法清除，降低内存泄漏的风险

### 场景

## AQS

- 为什么要设计 Lock ？1. 可中断 2. 非阻塞获取锁 3. 支持超时

- 设计思路：从 AQS 的类名称和修饰上来看，这是一个抽象类，所以从设计模式的角度来看同步器一定是基于【模版模式】来设计的，使用者需要继承同步器，实现自定义同步器，并重写指定方法，随后将同步器组合在自定义的同步组件中，并调用同步器的模版方法，而这些模版方法又回调用使用者重写的方法

- 同步器可重写的方法
  - 独占式：tryAcquire tryRelease
  - 共享式：tryAcquireShared tryReleaseShared
  - isHeldExclusively
  
- getState setState compareAndSetState

- 模版方法：(acquire acquireInterruptibly tryAcquireNanos release) (acquireShared acquireSharedInterruptibly tryAcquireSharedNanos releaseShared)

- Node 节点 (Node prev,next;Thread thread;int waitStatus;Node nextWaiter)

  - ```java
    public final void  acquire( int arg) {
        if(!tryAcquire(arg) && acquireQueued(addWaiter(Node.EXCLUSIVE), arg)){
          	selfInterrupt();
        }    
    }
    ```

  - AQS 内部维护了一个同步队列，用于管理同步状态。

    - 线程获取同步状态失败，将当前线程以及等待状态等信息构造成 Node （独占）节点
    
    - 加入到同步队列(双向队列head-tail-哨兵Node)中尾部(CAS+自旋(enq))
    
    - 结点进入队尾后，争取资源（prev节点是head）成功则升为head，返回中断信息，失败则找到安全休息点(判断前驱节点状态是否为Signnal)
    
    - 调用park()进入waiting状态，等待unpark()或interrupt()唤醒自己
    
    - 被唤醒后，进入自旋判断是否能争抢到资源。如果拿到，head指向当前结点，并返回从入队到拿到号的整个过程中是否被中断过；如果没拿到，则继续自旋流程
    
    - 如果线程在等待过程中被中断过，它是不响应的。只是获取资源后才再进行自我中断selfInterrupt()，将中断补上。
    
    - release :用unpark()唤醒等待队列中最前边的那个未放弃线程
    
    - ``` java
      public final boolean release(int arg) {
      if (tryRelease(arg)) {
               Node h = head;//找到头结点
               if (h != null && h.waitStatus != 0)
                   unparkSuccessor(h);//唤醒等待队列里的下一个线程
               return true;
           }
           return false;
      }
      ```

## ReentrantLock & CountDownLatch & CyclicBarrier

## Atomic

AtomicInteger 类主要利用 CAS + **<u>volatile</u>** 和 native 方法来保证原子操作

- 问题：
  - CAS ABA
    - AtomicStampedReference：维护一个Pair对象，Pair对象存储我们的对象引用和一个stamp值（版本号）。每次CAS比较的是两个Pair对象
  - long double 非原子性
    - 32位机 单次次操作能处理的最长长度为32bit，而long类型8字节64bit，所以对long的读写都要两条指令才能完成（即每次读写64bit中的32bit）。如果JVM要保证long和double读写的原子性，势必要做额外的处理（**volatile**）。

## ConcurrentHashMap

## BlockingQueue



- volatile 如何保障可见性，了解过 happen before 吗？
- volatile i i+1这是一个原子性操作吗？ 如果不是原子性怎么解决？ 为什么long 和 double 不是原子性的 i=66.66 是原子操作吗？
- 什么是指令重排序
- java 内存模型
- 用过syshronized吗 讲一下核心的原理和无锁编程有什么区别、静态方法和实例方法的区别？
- 创建线程的方式？
- 线程池的参数？
- 进程线程协诚的区别
- 线程有哪几种状态分别怎么转换
- 知道哪些原子类 ？