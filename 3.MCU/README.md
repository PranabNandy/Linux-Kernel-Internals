# MCU
## ✅ Modes in ARM Cortex-M3
Cortex-M3 supports two modes only:

### Thread Mode

- The mode in which normal application code runs.

- Can operate at:

    - Privileged level (full access to system resources)

    - Unprivileged level (restricted access, for safety and security)

- Entered on reset.

- After exception return, execution can return to Thread Mode.

### Handler Mode
- Used when servicing exceptions and interrupts (like IRQ, SVC, faults).

- Always Privileged.

- Includes:

    - Exception handlers (IRQ, SVC, SysTick)

    - Fault handlers (HardFault, MemManage, BusFault, UsageFault)
-----------------------------------------------------------------------------------------------------------------
## ✅ CONTROL Register ( Switching of Modes) 
CONTROL is a special register that determines privilege and stack usage in Thread Mode.

Bit	Meaning
- CONTROL[0] `~~~~`	`0 = Privileged, 1 = Unprivileged` (Thread Mode only)
- CONTROL[1] `~~~~`	`0 = Main Stack Pointer (MSP), 1 = Process Stack Pointer (PSP)`

Checking privilege
```c++
uint32_t control;
__asm volatile ("MRS %0, CONTROL" : "=r" (control) );
if (control & 0x01) {
    // Unprivileged
} else {
    // Privileged
}
```

✅ Switching Privilege Level
1. Privileged(Thread Mode) → Unprivileged (Thread Mode)

- Only possible from Privileged Thread Mode.

- Set CONTROL[0] = 1 and execute ISB (Instruction Synchronization Barrier):

```c++
__asm volatile (
    "MRS R0, CONTROL\n"     // Read CONTROL
    "ORR R0, R0, #1\n"      // Set bit 0 (unprivileged)
    "MSR CONTROL, R0\n"     // Write back
    "ISB\n"                 // Flush pipeline
);
```
2.1. Unprivileged(Thread Mode) → Privileged(Handler Mode)

- Cannot modify CONTROL directly (privileged instruction).

- Must trigger SVC (Supervisor Call) to request OS/Kernel to switch back:
```c++
__asm volatile ("SVC #0");
```
2.2. When SVC occurs, CPU enters Handler Mode (Privileged):
```c++
void SVC_Handler(void) {
    uint32_t control;
    __asm volatile ("MRS %0, CONTROL" : "=r" (control) );
    control &= ~0x01;    // Clear bit 0 → Privileged
    __asm volatile ("MSR CONTROL, %0" :: "r" (control) );
    __asm volatile ("ISB");
}
```
2.3. Return from SVC:

CPU returns to **Thread Mode, now privileged.**


✅ Flow:
- Thread Mode (Privileged) → set CONTROL → becomes Unprivileged.

- Unprivileged task → calls SVC → kernel regains Privileged (in Handler Mode) → can restore Privileged Thread Mode if needed.

```c++
/* Vector table placed at the start of flash (0x08000000 for STM32) */
__attribute__((section(".isr_vector")))
const void *vector_table[] = {
    (void *)STACK_TOP,    /* Initial stack pointer */
    Reset_Handler,        /* Reset handler */
    Default_Handler,      /* NMI */
    Default_Handler,      /* HardFault */
    Default_Handler,      /* MemManage */
    Default_Handler,      /* BusFault */
    Default_Handler,      /* UsageFault */
    0, 0, 0, 0,           /* Reserved */
    SVC_Handler,          /* SVC Handler */
    Default_Handler,      /* Debug Monitor */
    0,                    /* Reserved */
    Default_Handler,      /* PendSV */
    Default_Handler       /* SysTick */
};
```
-----------------------------------------------------------------------------------------------------------------

## Why 2 Handlers Instead of One? ( SysTick Handler & PendSV Handler)
- SysTick should run quickly → just marks PendSV pending.

- PendSV does the heavy lifting (saving/restoring registers) at the lowest priority.

✅ **PendSV (Pending Supervisor Call)**
Purpose:
- PendSV is an interrupt with the lowest priority in Cortex-M.
- It’s designed to defer work until all higher-priority interrupts are done.

Main Use Case:

- Context Switching in an RTOS (FreeRTOS, RTX, etc.).

- After a SysTick interrupt triggers a scheduling decision, the PendSV handler is set pending.

- When CPU finishes all other ISRs, PendSV executes → performs thread switch by saving current task context and restoring next task context.

Why PendSV?
- Context switching is not time-critical → Run it at the lowest priority.

- Avoids preempting high-priority ISRs with heavy operations.

-----------------------------------------------------------------------------------------------------------------
## ✅ SysTick
Purpose:
- A built-in 24-bit timer that generates periodic interrupts (e.g., every 1 ms).

Main Use Case:

- System Tick Timer for an OS → provides time base for:

- Preemptive scheduling.

- Delays, timeouts.

- In bare-metal, it can provide regular periodic tasks.

- How it works in RTOS:

        - SysTick fires every 1 ms.

        - SysTick ISR updates system tick count.
    
        - Sets PendSV interrupt pending if a context switch is needed.

-----------------------------------------------------------------------------------------------------------------

## ✅ Why update the OS tick count?

**The tick count** is a global variable maintained by the RTOS to **keep track of time in ticks** (e.g., every 1 ms if SysTick runs at 1 kHz). This is essential for:

### 1. Task Scheduling and Time-Slicing
- In preemptive multitasking, tasks share the CPU.

- SysTick interrupt fires periodically (e.g., every 1 ms).
- Each SysTick interrupt, we increase **The Tick count.**

- RTOS uses tick count to decide:

    - Whether the currently running task exceeded its time slice.

    - If so, trigger PendSV for context switching to another ready task.

Benefit:
- Without tick count, tasks could hog CPU indefinitely → no fairness.


```c++
+--------------------+         +-------------------+         +------------------+
|  User Application  | ----->  |   SVC Exception   | ----->  | Switch to Priv.  |
|  (Thread Mode,     |         | (for kernel entry)|         |  Thread Mode     |
|  Unprivileged)     |         +-------------------+         +------------------+
|                    |                                               |
|  System Calls via SVC (e.g., xTaskCreate, vTaskDelay, etc.)       |
+--------------------+                                               |
                                                                     ↓
                                                        +--------------------------+
                                                        |        Scheduler          |
                                                        |      (in RTOS kernel)     |
                                                        +--------------------------+
                                                                    |
                           +----------------------------------------+----------------------------------------+
                           |                                                                                 |
                    (Timer Interrupt)                                                                (Context Switch)
                          ↓                                                                                  ↓
                +-------------------+                                                        +----------------------------+
                |   SysTick Handler |-------------------------------------------------------> |   PendSV Handler           |
                | (Triggers Tick ISR)|                                                        | (Low-priority context sw.) |
                +-------------------+                                                        +----------------------------+
                                                                                                       ↓
                                                                                         +-------------------------+
                                                                                         | Save Current Context    |
                                                                                         | Load Next Task Context  |
                                                                                         +-------------------------+
                                                                                                   ↓
                                                                                        +--------------------------+
                                                                                        | Resume Next Thread       |
                                                                                        | (Thread Mode)            |
                                                                                        +--------------------------+

```

## Architecture
- what a car can do like go straight, go backward, go left, go right
- It basically specifies what a system should do 

## Micro-architecture
- How to implement all of these features in a real car
- Depending on the design and manufacturing company, car design might vary
- The basic functionality of the car should be the same 


![Screenshot from 2024-08-17 22-18-25](https://github.com/user-attachments/assets/9364f1a3-e0db-4d6a-800d-399c6450aa6a)

## To master CPU
- Programmers' model: details out the mode of operation and internal register that CPU has
  - How can a programmer think about CPU to do the computation
- Memory model: It specifies how the CPU interacts with the memory
  - like cpu stores some result in registers and sent it out in memory 
- Exception Model : 
- Debug Model : Like external HW is connected to debugging circuit of cpu
  - like **gdb** debugger has some power and limitation in terms of debugging the cpu

## Exception vs Interrupts
- **Exception** happens internal to the CPU (like accessing wrong memory address)
- **Interrupt** happens external to the CPU

![Screenshot from 2024-08-19 22-50-39](https://github.com/user-attachments/assets/832913e2-da6c-4a7a-a8b3-e654d25a13e6)
![Screenshot from 2024-08-19 22-50-16](https://github.com/user-attachments/assets/8bfb1bba-f751-4bb6-9344-3b84b9f93bc7)
![Screenshot from 2024-08-19 22-49-06](https://github.com/user-attachments/assets/e8c5d1bc-f404-4055-b4aa-ff70ce31f1ae)


## Different modes
- **Un-Privilege thread** mode generally made for application program execution.
- Suppose inside the foo() function
  - we called first bar()
  - then we will be calling bar2()
  - So we are calling bar() then we need to **store the return address in LR** i.e address of bar2()
![Screenshot from 2024-08-19 23-13-29](https://github.com/user-attachments/assets/7f209e28-7008-4abe-b29e-45325f401d81)
![Screenshot from 2024-08-19 23-25-29](https://github.com/user-attachments/assets/de869072-99af-47b8-8a4f-61c8114e5d47)

## what happens when CPU boots up
- first **MSP initial value** will be copied to **SP** (i.e R13 register)
- **Reset vector** value will be copied to **PC**
- PC will start fetching the instruction from that location **(mem[0x000004] = 0x000f9c)**
- Depending on the interrupt type, the respective **offset value** in the vector table will be decided and the respective **interrupt handler** will be called 
![Screenshot from 2024-08-20 06-56-44](https://github.com/user-attachments/assets/bee12b32-7b9e-42e0-81d8-6732873950c0)
