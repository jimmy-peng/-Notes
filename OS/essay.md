
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
