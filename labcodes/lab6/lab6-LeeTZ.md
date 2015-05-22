操作系统Lab6实验报告
===================
计14班 李铁峥 2011011268

-------------------

1. 练习1：理解并分析sched_calss中各个函数指针的用法，并接合Round  Robin 调度算法描ucore的调度执行过程

首先我们关注sched_class的定义：

```
struct sched_class {
    // the name of sched_class
    const char *name;
    // Init the run queue
    void (*init)(struct run_queue *rq);
    // put the proc into runqueue, and this function must be called with rq_lock
    void (*enqueue)(struct run_queue *rq, struct proc_struct *proc);
    // get the proc out runqueue, and this function must be called with rq_lock
    void (*dequeue)(struct run_queue *rq, struct proc_struct *proc);
    // choose the next runnable task
    struct proc_struct *(*pick_next)(struct run_queue *rq);
    // dealer of the time-tick
    void (*proc_tick)(struct run_queue *rq, struct proc_struct *proc);
};
```

由定义知各函数作用如下：
init函数初始化进程的运行队列
enqueue函数将进程放入运行队列中，在进程的need_resched = 1 时调用，进程放弃时间片进入等待下一次调度。
dequeue函数将进程弹出队列，进程将进入running状态。
pick_next用于选取下一个要被执行的进程，在schedule函数中调用，具体使用情况依赖于所用的调度算法。
proc_tick函数在每个clock tick调用，用于计时，进行进程的切换。

Round Robin调度算法的实现过程如下：
1. 进程进入队列。enqueue操作时为每个进程设置time_slice，在进程RUNNING时proc_tick函数不断递减time_slice的值，当其减为0时，将need_resched置为1，由schedule函数将该进程进行回收，进入run_list队列的尾部等待下一次被分配时间片。然后选择队列中的下一个进程进入RUNNINNG态。


1. 练习1：如何设计实现”多级反馈队列调度算法“，给出概要设计。

需要维护多个队列来多级反馈队列调度算法，由一个链表管理，该链表中的元素是某一个队列的头指针，按照队列优先级的顺序组织链表的顺序。
选择进程时遍历链表，按照优先级从高到低查看队列中是否存在进程，如果有进程就执行，并记录下进程所处的队列指针。
在进程用完固定的时间片后如果还没有结束，就将进程放置到下一级队列的尾部。


 2. 练习2：实现  Stride  Scheduling  调度算法

与RR类似，实现如下几个函数：

stride_init:初始化运行队列，将专用的lab6_run_pool设置好。
stride_enqueue:将进程的lab6_run_pool插入队列中，使用优先队列的接口。同时检查进程的时间片使用情况。
stride_dequeue:将进程移出优先队列。
stride_pick_next:由于优先队列的结构特性，可以直接从头部取出stride值最小的进程，同时更新stride的值，加上的pass值为BIG_STRIDE / priority。
stride_proc_tick:用于计时。


## 与参考答案的对比

trap.c中，注释指出处理时间中断的地方需要更新，然而trap.c中无法调用另一个文件中的静态函数。答案中也没有对齐进行调用。

## 实验中的知识点

本次实验重要的就是如何调度，什么时候调度。不同的调度算法对之有各自的处理。

## 原理课中的知识点

只选取了课程中的两个来实现，其余的没有用代码来体现。



