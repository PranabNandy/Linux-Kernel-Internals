
![unnamed (2)](https://github.com/user-attachments/assets/6484172d-55ed-4fb7-a58e-0993359375c3)
**Deriving details** - Get into the mode of separating out the how's and the why's. Ask the right questions, get to the specifics, don't get into the implementation details just yet - that inhibits free thinking and clutters judgement.

## General Concepts
FPGA might sound like an odd item in the list. These are one of the goto solutions for the ASIC (Application Specific Integrated Circuit) 
designers and having a working knowledge of these will make you a better Embedded systems software engineer.


<img width="368" alt="Screenshot 2024-09-22 075014" src="https://github.com/user-attachments/assets/66a5b1df-33ca-4c86-b908-d6afb8ae62f4">

You can think of FPGA as a CHIP with lot of unconnected logic gates (AND, NOT, OR etc) within them. Connecting these gates in 
certain fashion will lead to creation of a digital circuit. Much like how a firmware is written, the code for FPGAs is written in a 
languages referred to as HDL (Hardware Description Languages). We'll learn more about HDLs in later emails.

The HDL help describe the circuit in a textual form which is then converted to logic gate connection details for a given FPGA 
using the build tools that are provided by the FPGA vendor.

## Bare minimum SoC setup
![unnamed (1)](https://github.com/user-attachments/assets/f361766f-7b18-445b-8a1d-c6f3b3833816)

![unnamed](https://github.com/user-attachments/assets/ea90fa8b-1233-4995-add7-91e2cd7393e8)
- The fist thing embedded software engineers do is to write a **bare minimum firmware code** that boots from single memory and writes debug messages to the UART. Notice how only three hardware pieces have to be functional to achieve this - the CPU, Memory (SRAM) and the UART hardware



# CPU boot flow
- Two things will be given to CPU
    - Power to the cpu
    - clock to the cpu
 
- Both are taken care by Power Management circuit

- In ARM Cortex-M processors, the first two words of memory (at `0x00000000` and `0x00000004`) are used to initialize:

    - SP (Stack Pointer): Used for managing the stack, which is crucial for function calls, **interrupt handling**, and **exception handling**.
    - PC (Program Counter): Used to point to the first instruction to execute after the processor resets.

- In RISC-V processors, only the PC (Program Counter) is loaded with the reset vector address, and it does not automatically load the SP (Stack Pointer).

- Only loads the PC because it is designed to be more flexible, allowing **software (like the bootloader or OS)** to initialize the stack pointer (SP) later based on system requirements. 

- It is the responsibility of the programmer to place the `initial code` in right memory location (which is pointed by **PC**) using `Linker Script`.

### RISC-V
![1](https://github.com/user-attachments/assets/f33d3db7-5a71-4d9c-83d1-003a797524bc)
- **start.S code**
- Line 1 and 2 are the directives that will hint the assembler that line 4-6 need to be placed in `.init` section.
- We need to set the PC to this location so that we can set the SP before jumping into the main program.
- We place the start section at the address `0x8000 0000` location. 

### Address Space/Map in Memory mapped IO
![1](https://github.com/user-attachments/assets/d1b5367a-a6fe-4fbd-bce4-bf4e166b7f0b)
![2](https://github.com/user-attachments/assets/d4d473b7-6a8f-45f0-bbd8-9c793bf626a8)


### RTC

It stands for Real Time Clock. It is just a bunch of register, ticking up based on system level digital clock. 

### PLIC and CLINT

These are interrupt management related block.

### UART
- All transmiiting data via `tx` line will be written to `thx` register
- All the receiving data via `rx` line will be written to `rhx` register
- We need 3 codes to enable UART communication
![1](https://github.com/user-attachments/assets/44b0428e-c2f1-4743-8d47-e274e5bb6d70)

## main.ld code
![2](https://github.com/user-attachments/assets/77906a59-7d48-4cae-9dc2-cd063ec4dec7)

## main.c 
![3](https://github.com/user-attachments/assets/63ef2562-7c60-4406-b215-10407c0e3861)


## Line 46 is the black magic in embedded System C programming 
![5](https://github.com/user-attachments/assets/6aee6de8-3593-4e72-9f9c-64f2ca09d005)

## volatile
```c++
int foo(){
    volatile int x=0;
    x++;
    return x;
}
```
- volatile means never cache the variable value
- Not even in CPU register
- Just directly read it from Memory, modify it (if needed) and then write it back to the memory location

## const
- we can not change the value of const variable
- Direct modification is not possible but indirect write can happen using pointer
```c++
int foo(){
    const int x=851;
    int *ptr=(int*)&x;
    *ptr=560;
    return x;
}
```

## CSR register
- So we have mapped a **C structure** to CSR of some pheripheral device
![11](https://github.com/user-attachments/assets/41aec02c-e442-4b2b-afc8-612737caaae2)

## Compilation Process
### Linking steps
![Capture](https://github.com/user-attachments/assets/c2f0d4a5-dde9-4d9e-9146-5563697cf8af)

### All the Sys call made by compiler to compile a program
![Capture](https://github.com/user-attachments/assets/19de576f-8773-4d79-b34a-1922b7c0f880)

### meaning of each arguments in compilation
![Capture](https://github.com/user-attachments/assets/753fc47a-677a-484a-b6fe-f195f14b2eb9)
![Capture](https://github.com/user-attachments/assets/ac53851f-5c28-4943-8b2d-e3dfd8d1b8e8)


### Assembly code for Risc-v architecture 
![Capture](https://github.com/user-attachments/assets/b81eabdd-6915-432f-88cb-dc7a2da5a381)
![Capture](https://github.com/user-attachments/assets/452e65c5-fb09-4753-bdbd-f82901dbed07)

