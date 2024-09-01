# Linux fs
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

<img width="464" alt="Screenshot 2024-09-01 132835" src="https://github.com/user-attachments/assets/58537897-24e2-4d83-8c87-c095d057a4a6">
