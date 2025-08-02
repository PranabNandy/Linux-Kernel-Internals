### why kernel also need "system.map" and "initrd" along with vmlinux ?
![Screenshot from 2024-09-08 13-49-45](https://github.com/user-attachments/assets/a34e7abf-bdd7-437b-82b5-a55a0118f161)

- There are some external modules present in `ext4 fs` **(present in Hard drive)** which kernel is not aware of that. But some of those modules are necessary for booting up the procedure. Some of those modules are present in `initrd` ( **init ram disk**)
- `initrd` contains essential drivers and tools needed to **mount the real root file system.** The `initrd` allows the kernel to boot and load critical modules (e.g., **for accessing disks or filesystems**) before the main file system is mounted.
- The `System.map` file is a symbol table for the Linux kernel. It contains a mapping between symbol names (e.g., functions and variables) and their corresponding memory addresses in the kernel.
- `vmLinux` won't know how to load few system call. How to access those calls in `initrd`. It will be mapped by `system.map`. 

- In modern monolithic kernel, we can not put everything in kernel images. There must be some external module or loadable module.
- When we do **make module**, the modules will be generated in local location(where we are executing the make module). We need to put it inside the `/lib/module/'kernel_version'/` , in case we want to upgrade our kernel.
- To do the second part, we execute `make modules_install`
- Before loading kernel, grub boot loader will load and execute. We have to tell this bootloader that I have a new kernel. Can you please load OUR NEW KERNEL??
 

## Dynamically Loadable Modules (or External Modules)
![Screenshot from 2024-09-08 15-54-20](https://github.com/user-attachments/assets/81b0655c-1bcd-4f45-abc8-92fc175bf7bb)
## Different between lsmod and modprobe 
- Even though we are trying to load Custom module from our Custom Kernel (4.4.72)
![Screenshot from 2024-09-08 17-27-45](https://github.com/user-attachments/assets/7df1a743-59da-4208-96ab-bc46f8fdd1d5)

<img width="464" alt="Screenshot 2024-09-01 132835" src="https://github.com/user-attachments/assets/58537897-24e2-4d83-8c87-c095d057a4a6">
# Linux fs
<img width="1755" height="2487" alt="111" src="https://github.com/user-attachments/assets/094ddcb5-5fdb-475d-b7c4-93e57220c079" />

![Linux-storage-stack-diagram_v4 10](https://github.com/user-attachments/assets/f27c1c5f-48f2-4e5a-970c-c7b298e97c84)



- **`/ (Root File System):`** The root file system is the top-level directory in Linux. All other directories and file systems are mounted under this.

- **`/dev/ (Device File System):`** Contains device nodes that represent hardware and virtual devices. For example, /dev/sda represents a storage device.

- **`/proc/ (Process Information File System):`** A virtual file system that contains runtime system information, such as system memory, devices mounted, hardware configuration, etc. For example, /proc/cpuinfo provides information about the CPU.

- **`/sys/ (Sysfs):`** Provides information about the devices and drivers present in the system. It exposes kernel objects and their attributes, such as device information, kernel modules, and more.

- /tmp/ (Temporary File System): Used to store temporary files by applications and the system. Files here are typically cleared on reboot.

- /run/: A temporary file system that contains information about the system since the last boot. It is used to store transient runtime data.

- /mnt/ and /media/: Commonly used for mounting external file systems, such as USB drives, CD-ROMs, or network file systems.

- /boot/: Contains the boot loader files, including the Linux kernel and GRUB files.

- /home/: User home directories are stored here. Each user has a separate directory under /home/, such as /home/username/.

- /var/ (Variable Files): Contains variable data like logs, spool files, and transient data. For example, system logs are often stored in /var/log/.

- **`/etc/: `** Contains **configuration files and shell scripts** needed to boot and configure the system.

- /usr/: Contains user programs, libraries, documentation, and other files that are not part of the core system. /usr/bin/ contains executable binaries, and /usr/lib/ contains libraries.

- /opt/: Used to install optional software packages.

- /lib/: Contains shared libraries needed to boot the system and execute commands in the root file system.

- /sbin/: Contains system binaries essential for booting and system recovery.

- /srv/: Contains data for services provided by the system, such as web or FTP servers.

## Virtual File Systems:

- tmpfs: A temporary file system that keeps files in volatile memory (RAM) instead of a persistent storage disk.
- **`debugfs:`** A file system used for debugging the Linux kernel, typically mounted at /sys/kernel/debug.
- cgroup: A file system used to control and monitor resources allocated to process groups.

## Network File Systems:
- **`NFS (Network File System):`** Allows a system to share directories and files with others over a network.
- CIFS/SMB: Common Internet File System/Server Message Block, used mainly for file sharing on Windows networks.
- AFS: Andrew File System, a distributed network file system.


