---
title: 进程 - 控制
---

### 进程创建

#### 进程图



#### 引起创建进程的事件

一个进程去创建另一个进程

- **用户登录。**分时系统中，合法用户登录，系统为该终端创建进程并插入就绪队列
- **作业调度。**批处理系统中，作业调度算法调度到某作业时，将其装入内存，分配资源，创建进程，插入就绪队列
- **提供服务。**运行中的用户程序提出某种请求时，创建进程来满足这种请求，例如打印
- **应用请求。**创建新进程以并发运行方式完成任务

#### 创建进程

操作系统发现要求创建新进程的事件后，调用原语 **`Create`** 创建进程

- **申请空白 PCB。**申请获得唯一标识符，从 PCB 集合中索取一个空白 PCB
- **为新进程分配资源。**分配必要的内存空间，内存大小由用户提供，而交互型作业，可由系统自行分配；如果是共享内存，必须建立链接
- **初始化进程控制块**
  - 标识信息（标识符、父进程标识符）
  - 处理机状态信息。程序计数器指向程序入口，栈指针指向站定
  - 处理机控制信息。状态设置为就绪或者静止就绪，优先级设置为最低优先级，用户可显式提高优先级
- **将进程插入就绪队列，**在进程就绪队列可接纳新进程的前提下

---

### 进程终止

#### 引起进程终止的事件

- **正常结束。**批处理系统中，程序最后安排一条 Holt（产生一个中断） 指令或终止的系统调用；分时系统中，使用 Logs off 表示进程运行完毕，同样产生一个中断

- **异常结束**

  - 越界错误。程序访问的存储区越出该进程区域

  - 保护错。进程（以不恰当的方式）访问不允许访问的资源。例如，写一个只读文件
  - 非法指令。执行不存在的指令
  - 特权指令错。执行只允许 OS 执行的指令
  - 运行超时。执行时间超过规定的最大值
  - 等待超时。等待某事件的时间超过规定的最大值
  - 算术运算错。执行一个被禁止的运算。例如，被 0 除
  - I/O 故障。

- **外界干预**

  - 操作员或操作系统干预。例如，死锁情况
  - 父进程请求。父进程终止子孙进程
  - 父进程终止，所有子孙都将终止

#### 进程的终止过程

---

### 进程的阻塞与唤醒

#### 引起进程阻塞和唤醒的事件

- **请求系统服务**
- **启动某种操作**
- **新数据尚未到达**
- **无新工作各做**

#### 进程阻塞过程

#### 进程唤醒过程

---

### 进程的挂起与激活

#### 进程的挂起

用户进程请求自己挂起或父进程请求将某个子进程挂起

- 检查状态
- 活动就绪 -> 静止就绪，活动阻塞 -> 静止阻塞
- 将该进程 PCB 复制到指定内存区域，方便用户或父进程考察其运行情况
- 若被挂起的进程正在执行，则调度程序重新调度

#### 进程的激活过程

父进程或用户进程请求激活孩指定进程，若该进程驻留在外村而内存有足够的空间时，可将其换如内存

- 将其调入内存，检查状态
- 静止就绪 -> 活动就绪，静止阻塞 -> 活动阻塞
- 当有新进程进入就绪队列，检查是否需要重新调度