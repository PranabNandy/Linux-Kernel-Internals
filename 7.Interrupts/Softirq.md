Softirqs 
============
- One softirq `cannot be scheduled` twice on the same processor
- One softirq may run in parallel on other **at the same time.**

- Softirqs are bottom halves that run at a high priority but with hardware interrupts enabled

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


softirq methods
===================

Registering softirq handlers:
-------------------------

- Software interrupts must be registered before the kernel can execute them.

- **open_softirq** is used for associating the softirq instance with the corresponding bottom half routine.
```c++
void open_softirq(int nr, void (*action)(struct softirq_action *))
{
        softirq_vec[nr].action = action;
}
```
It is being called for example from networking subsystem.

net/core/dev.c:
---------------
-        open_softirq(NET_TX_SOFTIRQ, net_tx_action);
-        open_softirq(NET_RX_SOFTIRQ, net_rx_action);

Execution of softirq:
----------------------

The kernel maintains a per-CPU bitmask indicating which softirqs need processing at any given time
**irq_stat[smp_processor_id].__softirq_pending.**

Drivers can signal the execution of soft irq handlers using a function **raise_softirq().**
This function takes the index of the softirq as argument.
```C++
void raise_softirq(unsigned int nr)
{
        unsigned long flags;

        local_irq_save(flags);
        raise_softirq_irqoff(nr);
        local_irq_restore(flags);
}
```

- local_irq_save 		--> Disables interrupts on the current processor where code is running
- raise_softirq_irqoff 	--> sets the corresponding bit in the local CPUs softirq bitmask to mark the specified softirq as pending
- local_irq_restore	--> Enables the interrupts

raise_softirq_irqoff if executed in non-interrupt context, will invoke wakeup_softirqd(), to wake up, if necessary the ksoftirqd kernel thread of that local CPU

What is the benefit of per-CPU Bitmask?
========================================

By using a processors specific bitmap, the kernel ensures that several softIRQs — even identical ones — can be executed on different CPUs at the same time.

Executing Softirqs
=====================

- The actual execution of softirqs is managed by do_softirq()

Implementation : kernel/softirq.c

- do_softirq() will call __do_softirq(), if any bit in the local softirq bit mask is set.

- __do_softirq() then iterates over the softirq bit mask (least signicant bit) and invokes scheduled softirq handlers.


Creating a new softirq
-----------------------

You declare softirqs statically at compile time via an enum in <linux/interrupt.h>.

Creating a new softirq includes **adding a new entry to this enum.**

The index is used by the kernel as priority.

Softirqs with the lowest numerical priority execute before those with a higher numerical priority

Insert the new entry depending on the priority you want to give it.
<img width="1240" height="585" alt="image" src="https://github.com/user-attachments/assets/f45cc69c-a3f8-4456-8091-bc41388cb114" />


Registering your handler
------------------------

Soft irq is registered at runtime via open_softirq().

It takes two parameters:

	a) Index
	b) Handler Function.

Raising your softirq
---------------------

To mark it pending, so it is run at the next invocation of do_softirq(), call raise_softirq()

Softirqs are most often raised from within interrupt handlers.



Other Details
-----------------
The softirq handlers run with interrupts enabled and cannot sleep.

While a handler runs, softirqs on the current processor are disabled.

Another processor, can however execute another softirq.

If the same softirq is raised again while it is executing, another processor can run in it simultaneously.

This means that any shared data even global data used only within the soft irq handler needs proper locking.

most softirq handlers resort to per-processor data (data unique to each processor and thus not requiring locking) and other tricks to avoid explicit locking and provide excellent scalability.



<img width="1288" height="562" alt="image" src="https://github.com/user-attachments/assets/89769339-164f-4929-a8a2-a00d71f42184" />

```c++
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/gpio.h>
#include <linux/delay.h>
#include <linux/interrupt.h>
#define GPIO_BASE               0x3f200000      // GPIO controller 
#define GPIO_SIZE               0xb4
#define GPFSEL0_OFFSET          0x00
#define GPSET0_OFFSET           0x07
#define GPCLR0_OFFSET           0x0A
#define GPPUD_OFFSET		0x25
#define GPPUDCLK0_OFFSET	0x26

static unsigned int irq_number;
static unsigned int gpio_button = 15;
MODULE_LICENSE("GPL");
uint32_t *mem;

void set_gpio_pulldown(unsigned int gpio)
{
	int register_index = gpio/32;
	unsigned int value = (1 << (gpio % 32));
	mem = (uint32_t *)ioremap(GPIO_BASE, GPIO_SIZE);
	iowrite32(0x01, mem + GPPUD_OFFSET); //enable pull down
	// Wait 150 cycles
	udelay(2000);
	//Write to GPPUDCLK0/1 to clock the control signal into the GPIO pads you wish to modify
	iowrite32(value, mem + GPPUDCLK0_OFFSET + register_index);	
	// Wait 150 cycles
	udelay(2000);
	//Write to GPPUD to remove the control signal
	iowrite32(0x00, mem + GPPUD_OFFSET);
	//Write to GPPUDCLK0/1 to remove the clock
	iowrite32(0x00, mem + GPPUDCLK0_OFFSET + register_index);	
	iounmap(mem);
}
void print_context(void)
{
        if (in_irq()) {
                pr_info("Code is running in hard irq context\n");
        } else {
                pr_info("Code is not running in hard irq context\n");
        }
}
// Both softirq and hardirq are executed in same processor (i.e cpu_2)
void my_action(struct softirq_action *h)  // softirq handler function  --> running in interrupt context
{                                         // IRQ Enabled (hardirq handler can preemmpt it) -->  Not running in hard irq context
        pr_info("my_action\n");
}
static irqreturn_t  button_handler(int irq, void *dev_id)  // hardirq handler --> interrupt context
{                                                          // IRQ disable  -->  running in hard irq context
        pr_info("irq:%d\n", irq);
	    raise_softirq(MY_SOFTIRQ);
        return IRQ_HANDLED;
}
static int test_hello_init(void)
{
	pr_info("%s: In init\n", __func__);
	if (!gpio_is_valid(gpio_button)){
		pr_info("Invalid GPIO:%d\n", gpio_button);
		return -ENODEV;
	}

	pr_info("gpio button:%d is valid\n", gpio_button);
	if (gpio_request(gpio_button, "my_button")) {
		pr_info("GPIO Request Failed on gpio:%d\n", gpio_button);
		return -EINVAL;
	}
	pr_info("GPIO Request successful on gpio:%d\n", gpio_button);

	gpio_direction_input(gpio_button);
	gpio_set_debounce(gpio_button, 1000);      // Debounce the button with a delay of 1000ms
	set_gpio_pulldown(gpio_button);
	irq_number = gpio_to_irq(gpio_button);
        pr_info("irq number:%d\n", irq_number);
	open_softirq(MY_SOFTIRQ, my_action);
	return request_irq(irq_number, button_handler,
			IRQF_TRIGGER_FALLING,
			"button_interrupt",
			NULL);
}

static void test_hello_exit(void)
{
	pr_info("%s: In exit\n", __func__);
	free_irq(irq_number, NULL);
	gpio_free(gpio_button);
}

module_init(test_hello_init);
module_exit(test_hello_exit);
```


<img width="1311" height="643" alt="image" src="https://github.com/user-attachments/assets/0fb0adb8-fc42-486e-b1aa-bcee97f15d81" />

<img width="1250" height="581" alt="image" src="https://github.com/user-attachments/assets/aaee7d50-58cc-46eb-a4a2-c50abc76e2f7" />

ksoftirqd
============

Softirqs are executed as long as the processor-local softirq bitmap is set.

Since softirqs are bottom halves and thus remain interruptible during execution, 

the system can find itself in a state where it does nothing else than 
-	serving interrupts and 
-	softirqs

incoming interrupts may schedule softirqs what leads to another iteration over the bitmap.

Such processor-time monopolization by softirqs is acceptable under high workloads (e.g., high IO or network traffic), but it is generally undesirable for a longer period of time since (user) processes cannot be executed.

Solution to above problem by kernel
======================================

After the tenth iteration(**MAX_SOFTIRQ_RESTART** == 10) over the softirq bitmap, the kernel schedules the so-called **ksoftirqd** kernel thread, which takes control over the execution of softirqs.

Each processor has its own kernel thread called `ksoftirqd/n`, where n is the number of the processor

This processor-local kernel thread then executes softirqs as long as any bit in the softirq bitmap is set.

Therefore mentioned processor-monopolization is thus avoided by deferring softirq execution into process context (i.e., kernel thread), so that the ksoftirqd can be preempted by any other (user) process.

``` ps -ef | grep ksoftirqd/ ```

```c++
The spawn_ksoftirqd function starts these threads.

It is called early in the boot process.

static __init int spawn_ksoftirqd(void)
{
        cpuhp_setup_state_nocalls(CPUHP_SOFTIRQ_DEAD, "softirq:dead", NULL,
                                  takeover_tasklets);
        BUG_ON(smpboot_register_percpu_thread(&softirq_threads));

        return 0;
}
early_initcall(spawn_ksoftirqd);
```
File: kernel/softirq.c
```c++
Each ksoftirqd/n kernel thread runs the run_ksoftirqd()

static void run_ksoftirqd(unsigned int cpu)
{
        local_irq_disable();
        if (local_softirq_pending()) {
                /*
                 * We can safely run softirq on inline stack, as we are not deep
                 * in the task stack here.
                 */
                __do_softirq();
                local_irq_enable();
                cond_resched();
                return;
        }
        local_irq_enable();
}
```
In the existing code
```c++
void my_action(struct softirq_action *h)
{
	static int i = 0;
        pr_info("my_action\n");
	pr_info("current pid : %d , current process : %s\n",current->pid, current->comm);
	if (i++ <= 10)
		raise_softirq(MY_SOFTIRQ);
}
```
<img width="1077" height="532" alt="image" src="https://github.com/user-attachments/assets/85765ba4-9b0f-4dab-aa34-1a8770d5b5d6" />



