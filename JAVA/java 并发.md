# 并发

## volatile

- 指令重排序
- 内存屏障
- 总线风暴

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

## Synchronized & ReentrantLock & CountDownLatch & CyclicBarrier



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