# 并发

## volatile

## synchronized

## ReentrantLock

## ThreadPool

## ThreadLocal

## AQS

- 为什么要设计 Lock ？1. 可中断 2. 非阻塞获取锁 3. 支持超时

- 模版模式：

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

    - 当线程获取同步状态失败时，就会将当前线程以及等待状态等信息构造成一个 Node 节点，将其加入到同步队列中尾部，阻塞该线程
    - 当同步状态被释放时，会唤醒同步队列中“首节点”的线程获取同步状态

设计思路：从 AQS 的类名称和修饰上来看，这是一个抽象类，所以从设计模式的角度来看同步器一定是基于【模版模式】来设计的，使用者需要继承同步器，实现自定义同步器，并重写指定方法，随后将同步器组合在自定义的同步组件中，并调用同步器的模版方法，而这些模版方法又回调用使用者重写的方法









## ConcurrentHashMap

## BlockingQueue

## 



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