## General Concepts
FPGA might sound like an odd item in the list. These are one of the goto solutions for the ASIC (Application Specific Integrated Circuit) 
designers and having a working knowledge of these will make you a better Embedded systems software engineer.


<img width="368" alt="Screenshot 2024-09-22 075014" src="https://github.com/user-attachments/assets/66a5b1df-33ca-4c86-b908-d6afb8ae62f4">

You can think of FPGA as a CHIP with lot of unconnected logic gates (AND, NOT, OR etc) within them. Connecting these gates in 
certain fashion will lead to creation of a digital circuit. Much like how a firmware is written, the code for FPGAs is written in a 
languages referred to as HDL (Hardware Description Languages). We'll learn more about HDLs in later emails.



The HDL help describe the circuit in a textual form which is then converted to logic gate connection details for a given FPGA 
using the build tools that are provided by the FPGA vendor.


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
