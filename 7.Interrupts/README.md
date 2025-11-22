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
