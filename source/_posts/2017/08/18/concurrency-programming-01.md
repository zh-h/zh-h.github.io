---
title: 并发编程词义
date: 2017-08-18
tags: [Java,Linux]
categories: [Java]
---

## 并发 (concurrency) 与并行 (parallelism)

### 并发
两个或多个事件在同一时间间隔发生，微观上串行；

### 并行
两个或者多个事件在同一时刻发生，充分利用多核处理器。

## 原子性 (atomicity) (CAS)
- 原子性不论是多核还是单核，具有原子性的量，同一时刻只能有一个线程来对它进行操作；
- i++ 运算不具有原子性。

## 可见性 (visibility)

多个线程对一个共享变量进行操作时，由于编译器或者硬件优化的缘故，a 线程修改了变量的值，但是 b 线程缓存了变量原来的值，读取的就是 cache 中或者寄存器里的数据。(volatile)

## 进程、线程和协程

### 进程
“程序执行的一个实例” ，担当分配系统资源的实体。进程创建必须分配一个完整的独立地址空间；

### 线程
线程创建的开销主要取决于为线程堆栈的建立而分配内存的开销，这些开销并不大。线程上下文切换发生在两个线程需要同步的时候，比如进入共享数据段，由操作系统进行调度；

### 协程
与线程一样共享堆，不共享栈，协程调度切换时，将寄存器上下纹和栈保存起来，协程由程序定义调度(goto)。

