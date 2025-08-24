
## 1. How many gear modes are there in UFS M-PHY?

UFS uses M-PHY in both **High-Speed** (HS) and `Low-Speed` (LS) modes:

**High-Speed (HS) Gears:**
- Supports Gear 1, 2, 3, and extends up to Gear 4 and Gear 5 in later versions (M-PHY v5.0) 

**PWM (Type-I LS) Gears:**
- These cover Gear 0 through Gear 7 


**SYS (Type-II LS):**
- Unlike the PWM gearing, SYS mode isn't subdivided into typical “gears”—it’s a standalone mode, usually used for low-speed operation up to hundreds of Mbps


## 2. What are Series A and Series B for HS Gears?

Each HS gear defines two sub-rates:

Rate A and Rate B.

- These are roughly double each other
- Rate B is higher than Rate A

- `HS-Gear3-Series A` = **5 Gbps**
- `HS-Gear3-Series B` = **5.83 Gbps**
- PWM/ SYS are in **576 Mb/s**
```c++
struct uic_pwr_mode mode = {
    .lane = 2,        // 2 lanes
    .gear = 3,        // Gear 3
    .mode = 2,        // HS (not PWM)
    .hs_series = 2    // HS Series B
};
```
➡️ 2-lane, HS-G3B → ~11.6 Gbps aggregate throughput (2 × 5.8 Gbps).

**u8 mode**

- Selects which signaling mode is active:

  - `2 → Fast/High Speed (HS)`

  - `1 → PWM (Pulse Width Modulation)`

  - Sometimes you may also see SYS (Type II LS) in spec, but UFS generally uses HS or PWM only
 

✅ So struct uic_pwr_mode is just a software-side representation of UFS UniPro attributes, and the UFS host driver translates it into UIC DME_SET commands during link startup.

`ufshcd_change_power_mode()` → `ufshcd_dme_set()`

```scss
DME_SET(PA_ActiveTxDataLanes, 2)
DME_SET(PA_ActiveRxDataLanes, 2)
DME_SET(PA_TxGear, 3)
DME_SET(PA_RxGear, 3)
DME_SET(PA_TxMode, 2)   // HS
DME_SET(PA_RxMode, 2)   // HS
DME_SET(PA_HSSeries, 2) // HS-B
```




-------------------------------------------------------------------------------------------------------------

| Layer       | Role                                        | Interfaces With    |
| ----------- | ------------------------------------------- | ------------------ |
| Block Layer | Provides `read/write/erase` for filesystems | Calls SCSI Layer   |
| SCSI Layer  | Converts block ops to SCSI commands         | Uses UFS transport |
| UFS Layer   | Sends SCSI over UTP commands                | Talks to Host Ctrl |
| Host Ctrl   | Controls UFS PHY + registers                | Platform HW        |

## UFS Struct -> SCSI cmd struct -> SCSI device struct -> Block struct
```C++
target_init()
  └─► ufs_init(stage=1, host_id)
        └─► ufs_host_init()
              └─► ufs_hcd_init()         ← Initializes UFS Host Controller
                      └─► link startup
                      └─► descriptor setup
                      └─► device ready

  └─► ufs_init(stage=2, host_id)
        └─► scsi_alloc_device()
              └─► send INQUIRY
              └─► send READ CAPACITY
              └─► allocates bdev_t
              └─► bdev_register("scsi0")

```
## The UFS driver:

- Builds a UTP Transfer Request Descriptor (UTRD).

- Fills in the UFS Command Descriptor (UCD) with SCSI CDB bytes from cmd->cdb.

- Sets PRDT entries for data transfer.

- Rings the Doorbell to tell the UFS HC to start.
```C++

load_domain_binaries() function ultimately leads to bio_read(), but only indirectly 
through a well-abstracted, multi-layered call chain involving:

1.domain loading logic (*_load_binaries)
2.binary loading policy
3.partition policy
4.I/O variant (SCSI, SPI, etc.)

This abstraction allows the same upper-layer loading logic to work for multiple 
boot targets (Linux, QNX, RTOS) and storage devices (UFS, eMMC)
```

# 1. Key Components to Initialize in ufs_init()
Think of the ufs_init() process in four stages:

## A. Host Controller Hardware Bring-up
Before we even touch SCSI, we must bring the UFS Host Controller (UFS HC) to an operational state.

1.Enable UFS HC Clock & Power

- Turn on clocks (HCLK, PCLK, UFS core, etc.).

- Power up PHY and link.

- Enable required regulators (VCC, VCCQ, VCCQ2).

2.Reset the UFS HC

- Hardware reset (via reset controller or HCE bit in REG_CONTROLLER_ENABLE).

- Software reset for UTP (UTRL, UTMRL reset bits).

3.Program Host Controller Registers

- Capabilities (REG_CONTROLLER_CAPABILITIES).

- Interrupt enable (REG_INTERRUPT_ENABLE).

- UTP base addresses for transfer and task management request lists.

4.Link Startup

- Configure UniPro parameters (PA, DL, NL layers).

- Issue DME_LINKSTARTUP command and wait for the link to be active.

- Make sure the UIC link state is UP.

## B. UTP (UTP Transfer Protocol) Setup
UFS uses two queues internally:

- UTRL (UTP Transfer Request List) — For normal SCSI I/O (READ/WRITE/UNMAP…).

- UTMRL (UTP Task Management Request List) — For SCSI task management commands (ABORT, RESET).

In ufs_init():

- Allocate and zero UTRD (UTP Request Descriptors).

- Allocate UCD (Command Descriptors) and PRDT memory **(for DMA data buffers).**

- Write UTRLBA / UTRLBAU registers with physical addresses of UTRD array.

- Do the same for UTMRLBA / UTMRLBAU for TM commands.

## C. Interrupt Configuration
You must enable interrupts after queues are set up.

Enable Host Controller Interrupts:

- UTR Completion (UTRCS).

- UTMR Completion.

- UIC Command Completion.

- Error events (Fatal, Non-fatal, etc.).
Program REG_INTERRUPT_ENABLE with the correct mask.

## D. SCSI Layer Registration
Finally, after hardware is ready:

Create a scsi_transport_ops struct pointing to your:

- queue_cmd() (for normal commands)

- abort_cmd() / reset() (for task management)

- Call scsi_add_lun() to register with the SCSI layer so higher layers can start sending commands.

## 2. Interrupt Handling in UFS
How Interrupts Are Handled In LK:

You register an IRQ handler for the UFS HC IRQ line.

Inside the handler:

- Read REG_INTERRUPT_STATUS.

- Mask out events you care about.

- Clear the status bits you’ve handled (write-1-to-clear).

- Call specific sub-handlers (UTRL, UTMR, UIC, etc.).

| **Interrupt Type**                  | **Register Bits**    | **Purpose**                                                          |
| ----------------------------------- | -------------------- | -------------------------------------------------------------------- |
| **UTP Transfer Request Completion** | UTRCS                | Signals a normal SCSI command completed. Driver calls `scsi_done()`. |
| **UTP Task Management Completion**  | UTMRCS               | Signals a TM command completed (RESET/ABORT).                        |
| **UIC Command Completion**          | UICCMD               | Signals a UniPro/PHY config command is done (e.g., link startup).    |
| **Error Handling**                  | UECPA/UECDL/UECNL... | Protocol, Data Link, Network Layer errors. Requires recovery.        |
| **Fatal Error**                     | HCFES                | Host Controller fatal condition — might require reset.               |

#### Typical IRQ Flow
- IRQ fires.

- Driver reads Interrupt Status Register.

- If UTRL completion:

  - Find which slot finished (**UTRCS bitmask**).

  - Read the completion descriptor.

  - Mark the SCSI command as done and free the slot.

- If UTMR completion:

  - Process TM response, unblock waiting thread.

- If UIC command completion:

  - Mark UIC cmd complete so link training or DME operations can proceed.

- If Error:

  - Log it, possibly issue UIC reset or HC reset.


### PRDT and DMA
- UFS uses DMA to transfer data between system memory (PRDT) and the UFS device.
- PRDT always contains the physical address
- Instead of CPU copying data manually, the HC’s UTP engine reads the PRDT, then directly fetches or writes data from/to the memory addresses in those entries.
- Writes PRDT length in UTRD (UTP Transfer Request Descriptor).
  

```C++
Thread (exynosboot or app)
      |
      v
bio_read(dev, buf, offset, len)        [lib/bio/bio.c]
      |
      +--> dev->read(dev, buf, offset, len)
                |
                v
        scsi_read(dev, buf, offset, len)     [dev/scsi/scsi.c]
                |
                v
        scsi_exec_cmd()  -->  scsi_command_meta (CDB = READ10)
                |
                v
        ufs_send_scsi_cmd() / ufs_scsi_rw()   [dev/scsi/ufs.c]
                |
                v
        Build UTRD + UPIU + PRDT
        Write Doorbell bit for free slot
                |
                v
        UFS Host Controller DMA transfer  { In this mode, Bulk amount of data transferred from PRDT region }
                |
                v
        Device returns data → interrupt
                |
                v
        Complete SCSI cmd → signal thread

```

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


