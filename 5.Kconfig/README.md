The Linux Kernel Build System has four main components: 
- `Config symbols`: Two kinds of symbols are used for conditional compilation: boolean and tristate. Tristate symbols, on the other hand, can take three different values: yes (y : built in), no(n) or module(m). 

Not everything in the kernel can be compiled as a module. Many features are so intrusive that you have to decide at compilation time whether the kernel will support them. For example, you can't add **Symmetric Multi-Processing (SMP)** or **kernel preemption** support to a running kernel. 

`autoconf.h`: Note that although kernel source code may contain preprocessor directives (e.g. #ifdef for conditional compilation) utilizing CONFIG_xxx symbols that appear identical to the .config symbols, these symbols are not equivalent.

Kernel source code does not read or use the .config file.
Rather the .config file is converted into a C header file of preprocessor statements with a #define for each enabled CONFIG_xxx line, and stored in **include/generated/autoconf.h** (the exact path has changed for older versions).
It is the autoconf.h file that defines the CONFIG_xxx symbols used in kernel source code.
Note that a tristate configuration item that is assigned **m in the .config becomes #define CONFIG_xxx_MODULE 1** in the autoconf.h file, rather than just #define CONFIG_xxx 1
- `Kconfig files`: Compilation targets that construct configuration menus of kernel compile options, such as `make menuconfig`, read these files to build the tree-like structure. Every directory in the kernel has one Kconfig that includes the Kconfig files of its subdirectories. On top of the kernel source code directory, there is a Kconfig file that is the root of the options tree
- **`.config file`**: All config symbol values are saved in a special file called .config. Every time you want to change a kernel compile configuration, you execute a make target, such as menuconfig or xconfig.
- `Makefiles`: The last component of the kbuild system is the Makefiles. These are used to build the kernel image and modules. Like the Kconfig files, each subdirectory has a Makefile that compiles only the files in its directory. 

