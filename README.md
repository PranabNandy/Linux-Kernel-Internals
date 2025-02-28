Linux Device Driver book : https://github.com/PranabNandy/Linux-Kernel-Internals/blob/main/1.LDD/joibsn.md



### OS =
**Linux Kernel + Device Driver + Bootloader + Terminal + Basic fs + Other UI**
### Linux Kernel = 
**Interrupt Handler + Scheduler + Memory Management + System Services (such as Networking & IPC)**

#### Normal build , 
`$ make -j32`
#### Silencing the output,  
`$ make -j32 > /dev/null`
`Here only error messages will get printed`
# Linux-Kernel-Internals
- [ ] Linker Scripts
- [ ] Makefile
- [ ] Device Drivers  `(Done)`
- [ ] Bottom Halves Interrupts
- [ ] Timing Subsystem
- [ ] Linux Kernel Debugging
- Synchronization
- System Calls
- ELF file breakdown
- Exception in Armv8-A and Kernel Crash debugging
- Timers, PWM, CAN, Low Power(MCU)
- Custom Bootloader Development

### There are 4 types of device file
<img width="553" alt="Screenshot 2024-09-21 080105" src="https://github.com/user-attachments/assets/3f334eca-5c4f-418c-add1-617b3bbbd7e9">

### How to create and apply patches in "git" via Gerrit/ Github?
- $ git diff > s2r_poip.patch
- git cherry-pick 
