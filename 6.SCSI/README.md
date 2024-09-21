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
