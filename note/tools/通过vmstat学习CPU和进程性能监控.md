# [通过vmstat学习CPU和进程性能监控]( http://www.talkwithtrend.com/Article/161357 )

vmstat是一个很全面的性能分析工具，可以观察到系统的进程状态、虚拟内存使用、磁盘的IO、中断、上下文切换、CPU使用等情况。在操作系统性能分析中，能100%理解vmstat输出的含义并灵活应用，是性能分析必备的基本能力。
本次学习从vmstat下手，研究CPU的三个重要运行指标：上下文切换（context switchs）、运行队列（Run queue）和使用率（utilization），日常运维过程中经常根据这三个指标来判断CPU的性能和系统整体性能，但这三个指标不是孤立的，它们有一个共同的联系纽带：进程。
[![img](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481871951.png)](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481871951.png)
图1vmstat运行截图

# 1 进程工作模式和上下文切换

进程是在操作系统中运行的特定程序或执行的任务。进程是程序的运行过程，是随执行过程不断变化的实体。和程序包含指令和数据一样，进程也包含程序计数器和所有CPU寄存器的值，同时它的堆栈中存储着子程序参数、返回地址以及变量等临时数据。

## 1.1 进程的两种工作模式

内核态和用户态是现代操作系统进程的两种工作模式，内核态运行在内核空间，而用户态应用程序运行在用户空间。它们代表不同的级别，而对系统资源具有不同的访问权限。内核态运行在最高级别，这个级下所有的操作都受系统信任；而用户态运行在较低级别。在内核态时，处理器控制着对硬件的直接访问以及对内存的非授权访问。内核态和用户态有自己的内存映射，即自己的地址空间。
对应进程的两种工作模式，处理器总处于以下状态中的一种：
a、 内核态，运行于进程上下文，内核代表进程运行于内核空间；
b、 内核态，运行于中断上下文，内核代表硬件运行于内核空间；
c、 用户态，运行于用户空间。
进程的用户态切换到内核态有3种方式，但这三种方式只是进程内部的模式切换。
a、 系统调用：这是用户态进程主动要求切换到内核态的一种方式，用户态进程通过系统调用申请使用操作系统提供的服务程序完成工作，如创建新进程。系统调用的机制核心还是使用了操作系统为用户特别开放的一个中断来实现。
b、 异常中断：当CPU在执行运行在用户态下的程序时，发生了某些事先不可知的异常，这时会触发由当前运行进程切换到处理此异常的内核相关程序中，也就转到了内核态，比如缺页异常。
c、 外围设备的中断：当外围设备完成用户请求的操作后，会向CPU发出相应的中断信号，这时CPU会暂停执行下一条即将要执行的指令转而去执行与中断信号对应的处理程序，如果先前执行的指令是用户态下的程序，那么这个转换的过程自然也就发生了由用户态到内核态的切换。如硬盘读写操作完成，系统会切换到硬盘读写的中断处理程序中执行后续操作等。

## 1.2 上下文切换

进程上下文：就是一个进程在执行的时候，CPU的所有寄存器中的值、进程的状态、堆栈上的内容、进程打开的文件以及内存信息等，是进程运行的环境。
进程的上下文可以分为三个部分:用户级上下文、寄存器上下文以及系统级上下文。
a、 用户级上下文: 正文、数据、用户堆栈以及共享存储区；
b、 寄存器上下文: 通用寄存器、程序寄存器、CPU状态寄存器、栈指针；
c、 系统级上下文: 进程控制块、内存管理信息、内核栈
中断上下文：硬件通过触发信号，导致内核调用中断处理程序，进入内核空间。这个过程中，硬件的一些变量和参数也要传递给内核，内核通过这些参数进行中断处理。中断上下文可以看作就是硬件传递过来的这些参数和内核需要保存的一些其他环境（主要是当前被中断的进程环境）
上下文切换(context switch)可以分为三种情况：
a、 当内核需要从一个进程内核态切换到另一个进程内核态时，它需要保存当前进程的所有状态然后加载下一个进程的状态，即保存当前进程的进程上下文，以便再次执行该进程时，能够恢复切换时的状态，继续执行。
b、 进程由用户态切换到内核态，由用户空间转化为内核空间。可能由系统调用或设备中断引起，但不是所有的系统调用或设备中断都会触发上下文切换，那些在内核态中的系统调用和设备中断是不会引起上下文切换的。
c、 有时一个硬件中断的产生，也可能导致内核收到中断信号后由进程上下文切换到中断上下文。
无论哪种上下文切换，只要切换次数多都会影响CPU性能，这时线程就有非常大的优势。

## 1.3 线程

线程是一种轻量进程，实际上在内核中，两者几乎没有差别，除了一点：线程并不产生新的地址空间和资源描述符表，而是复用父进程的。但是无论如何，线程的调度和进程一样，必须切换到内核态。
进程优点是业务隔离，一个进程出现错误不会影响整个系统。如Oracle数据库服务器传统上就是进程模型。进程缺点是进程的分配和释放有非常高的成本。因此Oracle数据库需要连接池来保持连接减少新建和释放，同时尽量复用连接而不是随意的新建连接。
线程优点是更轻量，建立和释放速度更快，而且多个上下文间的通讯速度非常快。Web服务器传统上就是线程模型。线程缺点是一个线程出现问题容易将整个系统搞崩溃。

## 1.4 Vmstat工具faults栏（系统中断/切换）参数

熟悉了以上概念，对vmstat指令中faults栏就会有直观的理解了（见图1）：

1. in：(non clock) device interrupts 设备中断，IO繁忙的系统（数据库系统），进程经常性等待IO中断进入内核态运行，所以设备中断比较高，

2. sy：system calls系统调用，统计进程内部模式切换的系统调用和进程内核态内部系统调用的总数。Linux系统vmstat中没有这一项，认为系统调用也是中断。

3. cs：context switch上下文切换，统计进程间（线程间）上下文切换、进程（线程）模式切换和硬件中断的总数。CPU繁忙的系统（web系统）上下文切换较多，可以调整进程/线程数比或者通过进程复用的方法来降低上下文切换总量。
   如果CPU在满负荷运行，应该符合下列分布：
   User Time：65%～70%；System Time：30%～35%；Idle：0%～5%
   上下文切换要结合CPU使用率来看，如果CPU使用率满足上述分布，那么大量的上下文切换也是可以接受的。

   # 2 CPU利用率

   在使用vmstat时一般认为CPU栏sy是系统进程CPU占用率，us是用户进程CPU占用率；但学习了进程工作模式和上下文切换后，发现这种认识是错误的，真实表述如下：
   [![img](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872116.png)](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872116.png)
   图2 Linux系统vmstat运行截图

4. us列显示了所有进程用户态消耗CPU的时间百分比。us值比较高时，说明进程用户态消耗的CPU时间多，如果长期大于50%，需要考虑优化应用程序。

5. sy列显示了所有进程内核态消耗CPU的时间百分比。sy值比较高时，说明进程内核态消耗的CPU时间多；如果us+sy超过80%，就表明CPU资源存在不足。

6. id列显示了CPU处在空闲状态的时间百分比；

7. wa列表示进程IO等待所占CPU时间百分比。wa值越高，说明IO等待越严重。如果wa值超过20%，说明IO等待严重。wa仅在Linux系统vmstat中显示。

8. st列代表虚拟机占用CPU时间百分比。st仅在Linux系统vmstat中显示。
   除此以外，还有ni、hi、si三个CPU参数，这三个参数是在top工具中展示（也可以通过dstat -c查看），它们对操作系统性能监控有很大作用：
   [![img](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872159.png)](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872159.png)
   图3 Linux系统top截图

9. ni：用做nice加权的进程分配的用户态cpu时间百分比

10. hi：硬中断消耗CPU时间百分比

11. si：软中断消耗CPU时间百分比

12. st：虚拟机使用CPU时间百分比
    3 进程生命周期、状态和进程队列

    ## 3.1 生命周期

    每个进程都经历了创建、运行和死亡的周期，但最精彩的始终是运行部分。
    [![img](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872266.png)](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872266.png)
    图4 进程生命周期

## 3.2 进程状态及状态转换

下面是Linux内核中对进程状态的定义（其它类Unix系统的进程状态类似）：

```
 #define TASK_RUNNING            0 #define TASK_INTERRUPTIBLE      1 #define TASK_UNINTERRUPTIBLE    2 #define __TASK_STOPPED          4 #define __TASK_TRACED           8
```

/ *in tsk->exit_state* /进程的退出状态

```
 #define EXIT_ZOMBIE             16 #define EXIT_DEAD               32
```

/ *in tsk->state again* /理解为进程的唤醒状态

```
#define TASK_DEAD               64 #define TASK_WAKEKILL           128 #define TASK_WAKING             256 #define TASK_STATE_TO_CHAR_STR "RSDTtZXxKW"
```

下图是Linux和HPUX系统进程状态切换图，两个系统相似但又有细微不同：
[![img](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872339.png)](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872339.png)
图5 linux和HPUX进程状态变换图

下表是常见进程状态详解：
[![img](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872372.png)](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872372.png)
图6进程状态详解

大家会注意到在HPUX系统top中，进程的“睡眠状态”和“不可中断等待状态”都显示为sleep，但实际上这两种状态有很大区别；如果不可中断等待状态持续时间长，则代表系统存在性能问题。与top不同，HPUX工具glance中对进程所有状态都进行了细分（见下图），从中我们可以了解HPUX详细进程的运行情况。
[![img](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872413.png)](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872413.png)
图7 glance中所有阻塞状态
[![img](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872442.png)](http://www.talkwithtrend.com/home/attachment/201612/16/926467_1481872442.png)
图8 glance运行截图

需要注意glance工具和top工具监控进程都是每5秒刷新一次状态，如果持续刷新3次以上进程还是保持异常状态，才能判断此进程或系统真有异常。

## 3.3 vmstat中procs进程统计

通过以上介绍，我们可以详细解释vmstat工具procs进程统计部分三个参数含义：

1. r：代表CPU运行队列中进程数。原则上1核的CPU的运行队列不要超过2，整个系统的运行队列不能超过总核数的2倍，否则代表系统压力过大。

2. b：代表处于等待IO、内存页等资源的被阻塞的进程数，在Linux系统top中就是处于D状态（不可中断等待状态）的进程数；在HPUX系统glance工具中统计处于CDFS、VM和IO状态的进程数。

3. w：代表可以进入运行队列，但由于进程切换被切换出内核态的进程数。此参数只在HPUX系统的vmstat工具中存在。 如果系统存在阻塞的进程时，我们需要对这些进程进行密切关注，在Linux系统中可以使用strace/ltrace工具，在HPUX中可以使用tusc/kiinfo工具，限于篇幅就不详细介绍这些工具的使用方法。

   # 4 总结

   性能监控和优化是一个庞大而又严谨的体系，要深入研究只能通过原理、实现和工具三方面结合，本文只是管中窥豹学习了CPU调度和进程管理，希望对大家的运维工作有所帮助。