# SpringCloud && SpringBoot && Spring

## Spring

- IOC
- AOP
  - 入口：doCreateBean()，IOC容器getBean()对象是经过BeanWrapper包装的，若有AOP的逻辑，则返回的是AOP创建的代理对象。
  - 代理生成阶段
    - 完成bean的创建和依赖注入后，会调用initializeBean()方法
    - initialzeBean() 对Bean信息进行业务初始化，执行用户自定的业务比如Aware，BeanPostProcessor，initializingBean接口和initMethod业务，最后调用applyBeanPost ProcessorsAfterInitialization()方法
    - 初始化业务完成后，开始代理的生成，这里经过postProcessAfterInitialization()方法进入到wrapIf Necessary() 进入代理创建，创建代理前，会通过getAdvicesAndAdvisorsForBean() 来获得代理类各个通知方法组成的链条
    - 接着用createProxy()方法或JDK或Cglib生成代理对象
  - 代理执行阶段
    - 代理生成后保存到BeanWrapper 包装类里面，getBean() 实际上是代理对象
    - JDK
      - 代理对象每次执行方法，首先会进入JdkDynamicAopProxy.invoke()方法，这个方法会过滤掉不需要代理的一些对象原生方法，如equals，hashcode
      - 通过getInterceptorsAndDynamicInterceptionAdvice() 获取<u>**代理生成阶段**</u>生成的执行方法链，这里面包括了通知的方法，和他们的执行顺序
      - 拿到方法链后，通过ReflectiveMethodInvocation类的proceed()方法来循环调用执行方法链的方法，执行完成，整个流程结束
- 循环依赖和三级缓存
  - 
- Bean 的生命周期 && 扩展接口

## SpringBoot

- starter 设计

## SpringCloud

- Hystrix 线程隔离

  - 命令模式

    - 每一个请求封装在一个命令类，这些命令类实现一个共同的接口execute()，命令调用类用来接受这些命令类并在其中依次执行

    - 要素：命令接口、请求类、实现命令接口的实体类、命令调用类。利用命令调用类接受和执行命令。

  - 线程池隔离

  - 信号量隔离

    - 使用一个原子计数器（或信号量）来记录当前有多少个线程在运行，当请求进来时先判断计数 器的数值，若超过设置的最大线程个数则拒绝该请求，若不超过则通行，这时候计数器+1，请求返 回成功后计数器-1。与线程池隔离最大不同在于执行依赖代码的线程依然是请求线程 
      tips：信号量的大小可以动态调整, 线程池大小不可以

  - |  隔离方式  | 是否支持超时                                                 | 是否支持熔断                                                 | 隔离原理                                               | 是否异步调用                           |                                     资源消耗 |
    | :--------: | ------------------------------------------------------------ | ------------------------------------------------------------ | ------------------------------------------------------ | -------------------------------------- | -------------------------------------------: |
    | 线程池隔离 | 支持，可直接返回                                             | 支持，当线程池到达maxSize后，再请求会触发fallback接口进行熔断 | 每个服务单独用线程池，请求线程与转发处理线程不是同一个 | 可以是异步，也可以是同步。看调用的方法 | 大，大量线程的上下文切换，容易造成机器负载高 |
    | 信号量隔离 | 不支持，如果阻塞，只能通过调用协议（如：socket超时才能返回） | 支持，当信号量达到maxConcurrentRequests后。再请求会触发fallback | 通过信号量的计数器，请求线程与转发处理线程是同一个     | 同步调用，不支持异步                   |                             小，只是个计数器 |

