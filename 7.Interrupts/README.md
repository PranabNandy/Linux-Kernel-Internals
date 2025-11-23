### Interrupts
- PIC ==> x86 machine
- GIC ==> ARM machine

### Interrupt Context
When executing a interrupt handler, the kernel is in interrupt context

We know **process context** is the mode of operation the kernel is in while it is executing on behalf of a process.

- Eg. Executing a system call.

As interrupt context is not backed with process, you cannot sleep in interrupt context.

If a function sleeps, you cannot use it from your interrupt handler 

- Examples: kmalloc with GFP_KERNEL, ssleep.


```c++
#define SHARED_IRQ 12    // For Mouse interrupt
static int irq = SHARED_IRQ, my_dev_id, irq_counter = 0;
module_param(irq, int, S_IRUGO);

void print_context(void)
{
	if (in_interrupt()) {
		pr_info("Code is running in interrupt context\n");
	} else {
		pr_info("Code is running in process context\n");
	}
}
static irqreturn_t my_interrupt(int irq, void *dev_id)
{
	print_context();
	return IRQ_NONE;	/* we return IRQ_NONE because we are just observing */
}

static int __init my_init(void)
{
	print_context();
	if (request_irq
	    (irq, my_interrupt, IRQF_SHARED, "my_interrupt", &my_dev_id)) {
		pr_info("Failed to reserve irq %d\n", irq);
		return -1;
	}
	pr_info("Successfully loading ISR handler\n");
	return 0;
}

static void __exit my_exit(void)
{
	print_context();
	synchronize_irq(irq);
	free_irq(irq, &my_dev_id);
	pr_info("Successfully unloading,  irq_counter = %d\n", irq_counter);
}

```
--------------------------------------
### Memory allocation in inerrupt context
```c++
#define SHARED_IRQ 12
static int irq = SHARED_IRQ, my_dev_id, irq_counter = 0;
module_param(irq, int, S_IRUGO);


void *alloc_mem(unsigned int size)
{
	if (in_interrupt()) {
		return kmalloc(size, GFP_ATOMIC);
	} else {
		return kmalloc(size, GFP_KERNEL);
	}

}
static irqreturn_t my_interrupt(int irq, void *dev_id)
{
	void *mem = alloc_mem(1024);
	kfree(mem);
	return IRQ_NONE;	/* we return IRQ_NONE because we are just observing */
}

static int __init my_init(void)
{
	void *mem = alloc_mem(1024);
	kfree(mem);
	if (request_irq
	    (irq, my_interrupt, IRQF_SHARED, "my_interrupt", &my_dev_id)) {
		pr_info("Failed to reserve irq %d\n", irq);
		return -1;
	}
	pr_info("Successfully loading ISR handler\n");
	return 0;
}


```
### Printing call stack in interrupt handler
- Function name : dump_stack();

<img width="883" height="347" alt="image" src="https://github.com/user-attachments/assets/03115b7d-f3b2-485d-a49c-2d3fb331c59a" />


### Can we use the current MACRO inside the interrupt handler
- Basically it points to the interrupted process
```c++
static irqreturn_t my_interrupt(int irq, void *dev_id)
{
	irq_counter++;
	pr_info("In the ISR: counter = %d\n", irq_counter);
	pr_info("current pid : %d , current process : %s\n",current->pid, current->comm);
	return IRQ_NONE;	/* we return IRQ_NONE because we are just observing */
}
```


Top Half and Bottom Half
================================

Two important goals of interrupt handler are:

Execution Time
-----------------

- Handler of an interrupt must execute quickly

- long running handlers can slow down the system and may also lead to losing interrupts

- The faster the handler returns, the lower the interrupt latencies in the kernel, which is especially important for real-time systems

Execution Context
------------------

- Interrupt handlers are executed in hard-interrupt context – CPU-local interrupts remain disabled
		
- locking is undesirable and sleeping must be avoided
- large amount of work cannot be performed in interrupt handler


Both limitations lead to the fact that most interrupt handlers execute only a small amount of code and defer the rest of the work to a later point in time

Handling of interrupts is divided into two parts:

1. Top Half (Hard IRQ)
- Acknowledge the interrupt
- Copy the necessary stuff from the device
- schedule the bottom half

2. Bottom Half (Soft IRQ)
- Remaining pending work

	







