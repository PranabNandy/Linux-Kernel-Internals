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
