# JVM



## 

## 垃圾回收

- GCROOT
- 分代收集理论
- CMS
- G1

## JVM结构

## JMM

## 调优

### case：

- 排查CPU飙高
  - top 命令(找出CPU占比最高的进程和线程(转换为16进制))
    - load average
    - usertime-us idletime-id watingtime-wa
    - 物理内存使用情况
  - jstack pid > pid.txt
  - 查找到该线程的堆栈信息，定位到代码
  - case
    - while死循环
    - 频繁gc
    - 线程过多状态的频繁切换
  - 真实case
    - 一次logback 的排查
- 内存飙高
- GC 调优
- arthas 使用
- 压测工具