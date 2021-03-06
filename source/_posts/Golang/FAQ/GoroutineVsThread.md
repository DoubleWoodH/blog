---
title: Goroutine vs Thread
---

Goroutine 相对于 Thread 的一些优点

- 在一台典型的操作系统上，你可以运行更多的 goroutine
- 有可生长的分段栈
- 启动更快
- 内置基元使得 channel 之间的通信更加安全
- 允许你在共享数据结构的时候避免使用互斥锁
- 在一小部分的线程上可以多路复用，而不是 1:1 的映射
- 你可以编写大量并发的服务器，而不必借助于事件编程

### 运行更多的 goroutine 

与线程相比，goroutines 的堆栈大小只有几 KB，而且堆栈可根据应用程序的需要增长和缩小，而在线程的情况下，堆栈大小必须指定并固定，一般都是几 MB，所以，系统可以运行成千上万甚至上百万的 goroutines，也不会崩溃

例如，Java 的线程直接映射到操作系统的线程，并且相对重量级，部分原因是他们相当大的固定堆栈大小。 由于这会增加内存开销，因此限制了可以在单个虚拟机中运行的数量

goroutines 被多路复用到少量的 OS 线程。有数千 goroutines 的程序中可能只有一个线程。如果该线程中的任何goroutine 阻止了等待用户输入，则会创建另一个 OS 线程，并将剩余的 goroutine 移至新的 OS 线程

goroutine 被称为“绿色线程”，因为 Go runtime 执行调度，而不是操作系统。runtime 在一个操作系统的线程上多路复用 goroutines，数量由 `GOMAXPROCS` 控制。通常情况下，你需要将其设置为系统的核数，以最大限度地提高潜在的相关性

### 避免进入锁地狱

线程编程最大的缺点之一是许多使用线程来实现高并发的代码的复杂性和脆弱性。可能存在潜在的死锁和竞争条件，并且几乎不可能对代码进行推理

Go 提供基元，让你避免完全锁定。它的口头禅是 ***不通过共享内存来通信，而通过通信来共享内存***。换句话说，如果两个 goroutine 需要共享数据，他们可以通过 channel 安全地进行。Go 为你处理所有的同步操作，并且还要处理像死锁这样更难的问题

### 没有回调意大利面

也有一些其他的办法使用一小部分线程来实现高并发。`Python Twisted` 就是较早的很受关注的办法之一。Node.js 是现在最突出的事件框架

而这些事件框架有一个问题，就是代码很复杂，而且很难推理。程序员不是“直线”编码，而是被迫链接回调，而这些回调与错误处理交错在一起。尽管重构可以起到一部分作用，但它仍然是一个问题

### 参考

- [Goroutines vs Threads - Seven Story Rabbit Hole](http://tleyden.github.io/blog/2014/10/30/goroutines-vs-threads/)
- [第21部分：Goroutines](https://golangbot.com/goroutines/)