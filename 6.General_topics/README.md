## LDD-Communication_with_HW
https://github.com/PranabNandy/Linux-Kernel-Internals/blob/main/6.General_topics/4.LLD-Communication_with_HW.md

## Pointers
https://github.com/PranabNandy/Linux-Kernel-Internals/blob/main/6.General_topics/1.Preparation.md
## Ftrace
https://github.com/PranabNandy/Linux-Kernel-Internals/blob/main/6.General_topics/3.Ftrace.md
## Custom Scheduler
https://github.com/PranabNandy/Linux-Kernel-Internals/blob/main/6.General_topics/2.Scheduler_without_RTOS.md

## What is ABI?
- Application Binary Interface (ABI) is a set of rules and conventions that define how software components (communicates effectively) interact with each other at the machine code level.
- Operating systems play a crucial role in defining and enforcing ABIs. 
The ABI for a particular operating system, such as Windows, macOS, or Linux, determines how software components interact with that system. 
This ensures that applications can run consistently across different hardware platforms.

### A Real-World Example:
Let's imagine you're building a Lego castle. The bricks are like software components, and the instructions are like the ABI. 
If you follow the instructions correctly, you can build a sturdy and stable castle. 
If you don't, the castle might collapse. Similarly, if software components don't adhere to the ABI, they might not work together as expected.

-------------------------------------------------------------------------
In an embedded system (like a smart device with sensors and motors), different parts (like a sensor, a fan, and a screen) need to talk to each other. Instead of making them talk directly, they send messages to the mediator (the leader). The **mediator** tells each part what to do.

Example
- Sensor: "The temperature is too high!"
- Mediator: "Okay, let me tell the fan to turn on and the screen to show the temperature."
![Screenshot 2024-10-16 135402](https://github.com/user-attachments/assets/02de4d9c-4b84-4812-b736-9788af607edc)

--------------------------------------------------------------------------
## Caches
- Spacial Locality
- Temporal Locality

Spacial Locality = Example of the acccessing array of 1000 elements in a loop

Temporal Locality = 

--------------------------------------------------------------------------

## SoC vs MCU

![Screenshot 2024-10-16 150225](https://github.com/user-attachments/assets/28cfa769-4aa0-468b-880e-3ad82ddf6fcb)

MCUs are much simpler that SoCs.

#### The Memory Sizes
When I suggested that the memories are external to the SoC and that they are huge **(like in GBs)**, what I should have mentioned is - a **memory controller** is required to manage such memories. 

An MCU has a very simple interconnect/bus infrastructure to which the memories that it has **(small memories in Kilo and Mega bytes)** are connected. For an SoC, the interconnect is designed for extremely high bandwidth (in GBps) and has a dedicated memory controller that manages the external DRAM. What the presence of memory controller really suggests is - the level of complexity in the design.

#### MCU inside in GPU
Consider the GPU. It has too many ALUs (arithmetic and logic units) to perform Multiply-Add-Accumulate operations. The given graphics data doesn't automatically get processed, there is usually an MCU in the GPU that takes the incoming data (image rendering related data), understands the computation to be done and distributes the work to the ALUs in the GPU. Someone to manage the incoming and outgoing data so to speak...



