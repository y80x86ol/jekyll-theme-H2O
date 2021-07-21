---
layout: post
title: 'golang的goroutine原理(一)'
date: 2021-07-21
author: ken
postPatterns: 'ticTacToe'
tags: goroutine
---

> golang的goroutine原理理解，了解go的MPG模型，具体如何进行协程的调度，如果从并发和并行使go的性能最高效

go语言一个大的语言特色就是goroutine协程，而和很多同事沟通的时候，他们都认为goroutine很快，今天我们就来看一看goroutine是如何运行的。


### MPG模型
go使用的是MPG模型，意思是通过一个全局的调度器来实现goroutine协程的调度，来达到通过分配平均使用CPU资源。

go的调度器有3个重要的结构，M(OS线程)、P(协程调度器),G(goroutine协程)

- M(OS线程)：是操作系统的线程，一个程序可以模拟出多个线程。

- P(逻辑处理器or协程调度器)：这个一个专门调度goroutine协程的逻辑处理器，或者称为协程调度器都可以。

- G(goroutine协程)：goroutine协程。

用户空间线程和内核空间线程之间的映射关系有：N:1、1:1和M:N

- N:1，多个（N）用户线程始终在一个内核线程上跑，context上下文切换确实很快，但是无法真正的利用多核。

- 1:1，一个用户线程就只在一个内核线程上跑，这时可以利用多核，但是上下文switch很慢。

- M:N，多个goroutine在多个内核线程上跑，这个看似可以集齐上面两者的优势，但是无疑增加了调度的难度。

而go使用的就是M:N这种映射关系。

### 实现原理

当go启动一个进程的时候，会默认创建一个线程，这个线程会有一个逻辑处理器，通过具体的逻辑处理器处理goroutine协程。如下图所示：

![](https://raw.githubusercontent.com/y80x86ol/img/main/2021/20210721112655.png)

> 我们假设当前线程为M2，逻辑处理器为P0,有4个协程G1、G2、G3、G4等待执行，当前正在执行G1

假如当G1有执行文件阻塞的操作，这个时候逻辑处理器P0会将G1分离处理，同时与线程M2分离，如下图：

![](https://raw.githubusercontent.com/y80x86ol/img/main/2021/20210721112847.png)

这个时候原有的逻辑处理器P0下面还有很多协程需要执行，于是会生成一个新的线程M3来继续进行当前逻辑处理器P0的处理，此时顺序执行G2。而原有的线程M2和G1则等待文件阻塞操作的完成。

![](https://raw.githubusercontent.com/y80x86ol/img/main/2021/20210721113053.png)

此时M3正常执行goroutine，过了一段时间，原有的G1阻塞操作完成，等待被继续执行，而此时G2也执行完成，G1会被重新分配回到逻辑处理器P0进行执行。

![](https://raw.githubusercontent.com/y80x86ol/img/main/2021/20210721113413.png)

而这个时候，线程M2并不会立即销毁，而是等待被下次利用。这样一个简单的goroutine协程调度流程就完成了。下面来一张完整图：

![完整图](https://raw.githubusercontent.com/y80x86ol/img/main/2021/20210721113701.png)

但是这个只是针对系统文件的I/O操作的情况。如果是针对网络I/O的情况，稍微有一点不一样。

涉及网络I/O操作的时候，会使用网络轮询器来进行操作，对这个不太了解，有想深入了解的同学可以查阅相关资料。

但是原理是一样的，都是通过将阻塞的G放到其他线程处理，也放一张完整图：

![网络i/O完整图](https://raw.githubusercontent.com/y80x86ol/img/main/2021/20210721113853.png)

至此，一个单一的MPG模型就完成了，但是如果实现上面的M:N模型乃？

那就是多线程操作了。上面的列子只是说明一个单一的线程在执行goroutie调度。go启动的时候默认是启动4个线程M，4个线程M都都是一个MPG。图示如下：

![](https://raw.githubusercontent.com/y80x86ol/img/main/2021/20210721114302.png)

当我们要分配很多goroutine协程的时候，会被平均分配到各个线程M上，这样就实现了并行处理操作。

### 并发与并行

从上面的简单原理我们可以看到，go采用的是线程+协程的模式，并不是单一的协程。

- 并发：同时管理很多事情，这些事情可能只做了一半就被暂停去做别的事情了。
- 并行：同时做很多事情。

协程实现的是并发，线程实现的是并行。将并发和并行的结合，会做到全程的高性能。

### 小结

无论go语言如何变种，都逃不过操作系统的进程、线程。而我们所谓的go协程很快，是因为实现了一个复杂的调度器，来同时使用线程和goroutine协程，这样即可以充分利用多核CPU资源。

当然，协程的一个优点是开销非常小，要相对于大规模的线程，协程开销的确非常小。

每个 goroutine (协程) 默认占用内存远比 Java 、C 的线程少（goroutine：2KB ，线程：8MB）

所以这也是其中一个重要原因。

### 参考资料
1. 《GO语言实战》
2. [Golang 的 goroutine 是如何实现的？](https://www.zhihu.com/question/20862617)
3. [Goroutine(协程)的理解](https://studygolang.com/articles/19654?fr=sidebar)
4. [Go goroutine理解](https://segmentfault.com/a/1190000018150987)