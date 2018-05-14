
# linux调度类和调度策略


~~~
#define SCHED_OTHER  0
#define SCHED_FIFO   1
#define SCHED_RR     2
~~~

* STOP
    * No policy
        * migration
* Deadline
    * SCHED_DEADLINE
        * Video encoding/decoding
* Real TIME
    * SCHED_FIFO
        * IRQ threads
    * SCHED_RR
* FAIR
    * SCHED_NORMAL
    * SCHED_BATCH
    * SCHED_IDLE
* IDLE
    * No policy
        * swapd

# 什么是OS?

对计算机所有硬件资源的统一抽象，忽略硬件的差异性，方便管理这些资源。
