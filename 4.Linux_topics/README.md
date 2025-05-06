There are many ways to Communicate between the **Userspace** and **Kernel Space**, they are:
- IOCTL
- Procfs
- Sysfs
- Configfs
- Debugfs
- Sysctl
- UDP Sockets
- Netlink Sockets

## Difference between HLOS and RTOS
- HLOS has Virtual Memory(VM) which provides the Process Isolation ( this is kind of System Level engg)
- MCU CPU simply fetch the instruction and executes
- CPU Processor ( Cortex-A) has Layer of Isolation ( ELx)
- In RTOS, we can directly write the application whereas in HLOS we generally don't do that. Application developers mostly write that.

## FW engg vs System Engg
- In FW engg, there is no certain rule to write any driver
- In System Engg, 90% rules are defined regarding how to write a driver, you have to write the remaining 10% of it to achieve the goal i.e, manipulating a HW by also following the datasheet
- System Engg involve :
    - Application cLass CPU i.e has the concept of VM
    - HLOS
    - Kernel Device Driver
