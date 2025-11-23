Softirqs 
============

Softirqs are bottom halves that run at a high priority but with hardware interrupts enabled

Header File: <linux/softirq.h>

Data structures
---------------

Softirqs are represented by the softirq_action structure.
```
struct softirq_action
{
        void    (*action)(struct softirq_action *);
};
```

A 10 entry array of this structure is declared in kernel/softirq.c
```
static struct softirq_action softirq_vec[NR_SOFTIRQS];

	two for tasklet processing (HI_SOFTIRQ and TASKLET_SOFTIRQ),
	two for send and receive operations in networking (NET_TX_SOFTIRQ and NET_RX_SOFTIRQ),
	two for the block layer (asynchronous request completions),
	two for timers, and 
	one each for the scheduler and 
	read-copy-update processing
```
```c++
From include/linux/interrupt.h
enum
{
        HI_SOFTIRQ=0,
        TIMER_SOFTIRQ,
        NET_TX_SOFTIRQ,
        NET_RX_SOFTIRQ,
        BLOCK_SOFTIRQ,
        IRQ_POLL_SOFTIRQ,
        TASKLET_SOFTIRQ,
        SCHED_SOFTIRQ,
        HRTIMER_SOFTIRQ, /* Unused, but kept as tools rely on the
                            numbering. Sigh! */
        RCU_SOFTIRQ,    /* Preferable RCU should always be the last softirq */

        NR_SOFTIRQS
};
```

The number of registered softirqs is statically determined and cannot be changed dynamically.


Preeemption
-------------
A softirq never preempts another softirq . The only event that can preempt a softirq is interrupt handler.

Another softirq even the **same one can run on another processor.**
