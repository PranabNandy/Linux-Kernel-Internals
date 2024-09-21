## SCSI 
Small Computer Systems Interface (SCSI) defined both a `parallel I/O bus` and `a data protocol` to connect a wide variety of peripherals 
(disk drives, tape drives, modems, printers, scanners, optical drives, test equipment, and medical devices) to a host computer.

Although the old parallel (fast/wide/ultra) SCSI bus has largely `fallen out of use`, the **SCSI command set** is more widely used than ever to communicate with devices over a number of different busses.

The SCSI protocol is a big-endian peer-to-peer `packet based protocol`. SCSI commands are **6, 10, 12, or 16** bytes long, often followed by an associated data payload.

SCSI commands can be transported over just about `any kind of bus`, and are the default protocol for storage devices attached to USB, SATA, SAS, Fibre Channel, FireWire, and ATAPI devices. SCSI packets are also commonly exchanged over Infiniband, I20, TCP/IP (iSCSI), even Parallel ports.

<img width="957" alt="scsi" src="https://github.com/user-attachments/assets/0f3b1929-fb6c-4c6f-bebe-1b4ae55a881c">

### Main file for the SCSI midlayer.

`drivers/scsi/scsicam.c`
- **scsi_bios_ptable** — Read PC partition table out of first sector of device.
- **scsicam_bios_param** — Determine geometry of a disk in cylinders/heads/sectors.
- **scsi_partsize** — Parse cylinders/heads/sectors from PC partition table

### SCSI sysfs interface routines.
`drivers/scsi/scsi_sysfs.c`
- **scsi_remove_device** — unregister a device from the scsi bus
- **scsi_remove_target** — try to remove a target and all its devices

### mid to lowlevel SCSI driver interface
`drivers/scsi/hosts.c`
- scsi_host_set_state — Take the given host through the host state model.
- scsi_remove_host — remove a scsi host
- scsi_add_host — add a scsi host
- scsi_host_alloc — register a scsi host adapter instance.
- scsi_host_lookup — get a reference to a Scsi_Host by host no
- scsi_host_get — inc a Scsi_Host ref count
- scsi_host_put — dec a Scsi_Host ref count
- scsi_queue_work — Queue work to the Scsi_Host workqueue.
- scsi_flush_work — Flush a Scsi_Host's workqueue.

`drivers/scsi/scsi_scan.c`

Scan a host to determine which (if any) devices are attached. The general scanning/probing algorithm is as follows, exceptions are made to it depending on device specific flags, compilation options, and global variable (boot or module load time) settings. A specific LUN is scanned via an INQUIRY command; if the LUN has a device attached, **a scsi_device is allocated and setup for it**. 

**For every id of every channel on the given host**, start by scanning LUN 0. Skip hosts that don't respond at all to a scan of LUN 0. Otherwise, if LUN 0 has a device attached, allocate and setup a scsi_device for it. If target is SCSI-3 or up, issue a REPORT LUN, and scan all of the LUNs returned by the REPORT LUN; else, sequentially scan LUNs up until some maximum is reached, or a LUN is seen that cannot have a device attached to it.


Two main structures `(Scsi_Host and Scsi_Cmnd)` are used to communicate between the high-level code and the low-level code. The next two sections provide detailed information about these structures and the requirements of the low-level driver.

### Requesting the DMA channel
Some SCSI **host adapters use DMA** to access large blocks of data in memory. Since the CPU does not have to deal with the individual DMA requests, data transfers are faster than CPU-mediated transfers and allow the CPU to do other useful work during a block transfer (assuming interrupts are enabled).

The host adapter will use a specific DMA channel. This DMA channel will be determined by the `detect()` function and requested from the kernel with the `request_dma()` function. This function takes the DMA channel number as its only parameter and returns zero if the DMA channel was successfully allocated. Non-zero results may be interpreted as follows:

- EINVAL :-
The DMA channel number requested was larger than 7.
- EBUSY :-
The requested DMA channel has already been allocated. This is a very serious situation, and will probably cause any SCSI requests to fail. It is worthy of a call to panic().

#### TEST UNIT READY
This command is used to test the target's status. If the target can accept a medium-access command (e.g., a READ or a WRITE), the command returns with a GOOD status. Otherwise, the command returns with a CHECK CONDITION status and a sense key of NOT READY. This response usually indicates that the target is completing power-on self-tests.
#### INQUIRY
This command returns the target's make, model, and device type. The high-level Linux code uses this command to differentiate among magnetic disks, optical disks, and tape drives (the high-level code currently does not support printers, processors, or juke boxes).
#### READ and WRITE
These commands are used to transfer data from and to the target. You should be sure your driver can support simpler commands, such as TEST UNIT READY and INQUIRY, before attempting to use the READ and WRITE commands.

#### unchecked_isa_dma
Some host adapters use Direct Memory Access (DMA) to read and write blocks of data directly from or to the computer's main memory. Linux is a virtual memory operating system that can use more than **16 MB of physical memory**. Unfortunately, on machines using the ISA bus (the so-called ``Industry Standard Architecture'' bus was introduced with the IBM PC/XT and IBM PC/AT computers), DMA is limited to the low 16 MB of physical memory.

If the unchecked_isa_dma bit is set, the high-level code will provide data buffers which are guaranteed to be in the low 16 MB of the physical address space. Drivers written for host adapters that do not use DMA should set this bit to zero. Drivers specific to EISA bus (the ``Extended Industry Standard Architecture'' bus is a non-proprietary 32-bit bus for 386 and i486 machines) machines should also set this bit to zero, since EISA bus machines allow unrestricted DMA access.

#### detect()
The detect() function's only argument is the host number, an index into the scsi_hosts variable (an array of type struct Scsi_Host). The detect() function should return a non-zero value if the host adapter is detected, and should return zero otherwise.

Host adapter detection must be done carefully. Usually the process begins by looking in the ROM area for the ``BIOS signature'' of the host adapter. On PC/AT-compatible computers, the use of the address space between 0xc0000 and 0xfffff is fairly well defined. For example, the video BIOS on most machines starts at 0xc0000 and the hard disk BIOS, if present, starts at 0xc8000. When a PC/AT-compatible computer boots, every 2-kilobyte block from 0xc0000 to 0xf8000 is examined for the 2-byte signature (0x55aa) which indicates that a valid BIOS extension is present.

Usually, each host adapter has a series of I/O port addresses which are used for communications. Sometimes these addresses will be hard coded into the driver, forcing all Linux users who have this host adapter to use a specific set of I/O port addresses. Other drivers are more flexible, and find the current I/O port address by scanning all possible port addresses. Usually each host adapter will allow 3 or 4 sets of addresses, which are selectable via hardware jumpers on the host adapter card.
