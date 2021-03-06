---
title: 垃圾回收机制
---

### 垃圾回收

传统的系统级编程语言（主要是 C/C++），如果对内存的管理稍有不慎，就会出现内存泄露的情况，导致程序的崩溃，所以需要程序员小心地进行内存的申请与释放，所以之后的语言（如 java，python，php，js，go等）都引入了自动内存管理，也就是语言的使用者只需关注内存的申请而不必关心内存的释放，内存释放由虚拟机（virtual machine）或运行时（runtime）来自动进行管理。而这种对不再使用的内存资源进行自动回收的行为就被称为垃圾回收

### 常见的垃圾回收方法

#### 引用计数（reference counting）

最简单的一种，对每个对象维护一个引用计数，假设对象 P 的引用计数为 S，而有一个新的对象 A 引用了对象 P，则 S 加 1，当对象 A 被销毁或者 对象 A 更改引用为对象 Q，则 S 减 1，最后当 S 为 0 时，自动回收对象 P

**优点**

- 这种回收方法实现简单，对内存的回收也很及时

**缺点**

- 频繁地更新引用计数降低了性能。一种简单的解决方法就是编译器将相邻的引用计数更新操作合并到一次更新；还有一种方法是针对频繁发生的临时变量引用不进行计数，而是在引用达到 0 时通过扫描堆栈确认是否还有临时对象引用而决定是否释放，等等
- 循环引用。当对象间发生循环引用的问题时，循环链中的对象都无法释放。最明显的解决办法就是避免产生循环引用，如 Cocoa 引入了 strong 指针和 weak 指针两种指针类型。或者系统检测循环引用并主动打破循环链。当然这都增加了垃圾回收的复杂度

#### 标记 - 清除（mark and sweep）

从根开始，遍历所有对象，给每个可达对象打上“标记”，标记完进行清除操作，对没有标记的对象进行内存回收（同时可能伴有碎片整理操作)

**优点**

- 不存在处理循环引用，需要维护指针

**缺点**

- 每次启动垃圾回收时，需要 STW（Stop The World)，即需要中断正常程序一段时间来回收垃圾，如果对象多的话需要的时间就会更长一些，使系统响应能力大大降低，不过后续许多方法优化了这个算法，如**三色标记法**

#### 节点复制（copying）

也是基于追踪的算法，它将内存空间一分为二，分别记为 FromSpace 和 ToSpace，新对象的内存都是在 FromSpace 分配，而当 FromSpace 满时，同样从根出发，遍历所有对象，把可达对象复制到 ToSpace 中，然后对调 FromSpace 和 ToSpace 的角色。回收过程中需要 STW

**优点**

- 节点复制时相当于做了一次内存整理的紧缩工作，不存在内存碎片的问题
- 所有的对象在内存中永远都是紧密排列的，所以分配内存极为简单，只要移动一个指针即可，对于内存分配频繁的环境来说，性能优势很大

**缺点**

- 导致一半的内存空间处于空闲状态

#### 分代收集（generation）

基于追踪的垃圾回收算法（标记-清扫、节点复制)，一个主要问题是在生命周期较长的对象上浪费时间（长生命周期的对象是不需要频繁扫描的）。同时，内存分配存在这么一个事实 “most object die young”，基于这两点，分代收集算法将内存空间划分为两个（或更多）区域，这些区域就是代（generation)，一般为新生代和老一代。为新对象分配内存时，从新生代分配，如果后期多次回收发现该对象都没被清除，便将其放入老一代，这个过程叫做 promote。随着不断 promote，最后新生代的内存大小在整个内存空间的比例不会很大，对新生代的对象执行垃圾回收相对频繁，老生代频率相对较低。回收过程中需要 STW

**优点**

- 相对于全区域扫描，分代提高了扫描的效率，STW 时间也会更短

**缺点**

- 实现复杂

### Golang 中的垃圾回收机制

下面是随着 Golang 版本发布，GC 比较重要的改动

- v1.1    STW
- v1.3    Mark - Sweep 和 STW 并行（并发清理，即垃圾回收和用户逻辑并发执行）
- v1.5    三色标记法
- v1.8    hybrid write barrier

#### 三色标记法

它是**标记 - 清除**的一个变种，有三种颜色：白色，灰色，黑色

标记过程：

1. 所有对象最初都是白色
2. 将所有初始的可达对象，即全局对象或者栈对象(root集合)，标记为灰色
3. 从中任意取出一个对象，将所有引用它的白色对象标记为灰色，最后标记它自身为黑色
4. 重复上一步，直到灰色集合为空
5. 此时，剩下的对象不是白色就是黑色，白色不可达，黑色可达
6. 回收所有白色对象

白色表示不可达对象；灰色表示对象本身被引用到，但是它的孩子结点还没处理完；黑色表示本身被引用，并且已处理对象中的子引用

最终，标记算法完成后，不会存在灰色对象，黑色表示活跃的对象，而标记白色的对象将会被回收

如果是STW的，上面没什么问题。但是如果允许用户代码跟垃圾回收同时运行，需要维护一条约束条件：

- **黑色对象绝对不能引用白色对象**

为什么不能让黑色引用白色？因为黑色对象是活跃对象，它引用的对象是也应该属于活跃的，不应该被清理。但是，由于在三色标记算法中，黑色对象已经处理完毕，它不会被重复扫描。那么，这个对象引用的白色对象将没有机会被着色，最终会被误当作垃圾清理。

STW 中，一个对象，只有它引用的对象全标记后才会标记为黑色。所以黑色对象要么被黑色对象引用，要么被灰色对象引用，不会出现黑色引用白色对象。

对于垃圾回收和用户代码并行的场景，用户代码可能会修改已经标记为黑色的对象，让它引用白色对象。看一个例子来说明这个问题：

```wiki
stack -> A.ref -> B
```

A 是从栈对象直接可达，将它标记为灰色。此时B是白色对象。假设这个时候用户代码执行：

```go
localRef = A.ref
A.ref = NULL
```

localRef 是栈上面的一个黑色对象，前一行赋值语句使得它引用到 B 对象。后一行 A.ref 被置为空之后，A 将不再引用到 B。A 是灰色但是不再引用到 B 了，B 不会着色。localRef 是黑色，已经处理完毕，引用了 B ，但不会被重复扫描。于是 B 将永远不会再有机会被标记，会被误当作垃圾清理掉

### 参考资料

- [Golang 垃圾回收剖析](https://studygolang.com/articles/10175)
- [golang 垃圾回收 gc](https://studygolang.com/articles/7366)
- [垃圾回收面面观](https://studygolang.com/articles/7558)