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
- MCU CPU simply fetch the instruction and execute
- CPU Processor ( Cortex-A) has Layer of Isolation ( ELx)
- In RTOS, we can directly write the application whereas in HLOS we generally 
