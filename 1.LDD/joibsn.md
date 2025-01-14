# File System
File system structure is stored in the `first block of the disk`. Root of the file system structure contains how many files are there in the system. 

#### FAT 16 file system
- This is powered by Microsoft
- First 3 bytes of boot sector `usually a short jump`
- `Between` first line and short jump level, it contains the `fs structure`
  
{ `First sector` = boot sector | reserved sector | reserved sector ....| File Allocation table | file allocation table(backup) | root directory | Data regions }

========================================================
- Files do not exist on disk (only holds data blocks called sectors ) It is an OS concepts.
- Disk only contains the file system structures. This is special data structure that explains the files.
- It is upto the kernel to read this data structure correctly.
- Kernel must have file system driver to read the files.
- Disk is not responsilbe to implement the fs.
- Implementing the file system requires the kernel programmers to create file system driver for target file systems.

## file system structure
- contains a header which holds information like `how many files` are there in the disk, where `root fs` is located on disk etc.
- When you `format any disk` (USB) with a file system structure then the `new file system structure strored in the first sector` of the disk (USB)
- Your OS has to be programmed to understand the fs (USB new fs) you want to read.
- You can not store data using `NTFS file system` if your OS supports only `FAT 32 fs`.

## VFS
- It allows multiple number of underlying fs layer
- It allows file system funtionality to be loaded and unloaded in kernel at any time
  ![Screenshot 2025-01-11 181401](https://github.com/user-attachments/assets/02f62f68-83be-4abe-8b8b-f7c0d0377aeb)
![WhatsApp Image 2025-01-11 at 6 13 21 PM](https://github.com/user-attachments/assets/108a6d1f-6f08-445b-b0f4-e1cf299f5be1)

### What happens when we insert a new disk in the system?
- Kernel will poll each file system and check whatever file system, `the disk holds` whether it(i.e OS) can manage or not
- for example FAT fs will read the first block ( it varies depending on the fs) and maps `FAT fs driver` for that disk

### How VFS Works
- **High-Level API:** Applications interact with the VFS using system calls like **open, read, write, and close**.
- **File System Abstraction:** The VFS maps these calls to the appropriate file system driver (e.g., ext4, NTFS) based on the mounted file systems.
```c++
 mount -t qnx4 /dev/hd0.ms.84 /
```
- **Uniform Namespace:** All file systems are presented as part of a single directory tree, regardless of their physical location or type.
- **Interfacing with Drivers:** The VFS communicates with  `file system drivers` followed by `storage device drivers` to perform the actual file operations.


## What is kernel panic ?
- When kernel enter into a state from where it can not recover
- Then we call the kernel panic handler
- what it will do ?
- It will print the messages into the buffer or on the screen and then
- Halt the kernel

## Important directory in Linux ( Top 5)
- `bin` : contains all binary
- `sbin` : contains init process to jump from kernel space to user space
- `lib` : Contains all library + all SO + all Modules i.e Dynamically Loadable Kernel Driver like (/lib/Modules/..)
- `dev` : contains all hardware devices in form of files
- `etc` : contains all the configurations and rules. Here only, we create our user name and password


==================================================================================
# LDD

Device driver communicates with device for reading and writing 
- through a bus to Device pheripheral Register (HCI)
- Bus is a data path between device controller(UFSHC) and processor
- [Processor] <----------bus-----------> [UFS HC] <-------MIPI--------> [UFS Device]
- pheripheral controller = UFS HC  ~~~~ & & ~~~~~  Pheripheral Device = UFS Device


Device driver is called as **Module** as per kernel language.

Module uses kernel services
- INT
- memory allocation
- wait queue

**insmod** is a user space application

## Chapter 6

printk - Using this function, string will get loaded in kernel **log_buffer**

printk has 8 log level - defines the importance of message

## Chapter 7 - Paremeter Passing 

```C++

#include<linux/kernel.h>
#include<linux/init.h>
#include<linux/module.h>
#include<linux/moduleparam.h>
 
int valueETX, arr_valueETX[4];
char *nameETX;
int cb_valueETX = 0;
 
module_param(valueETX, int, S_IRUSR|S_IWUSR);                      //integer value
module_param(nameETX, charp, S_IRUSR|S_IWUSR);                     //String
module_param_array(arr_valueETX, int, NULL, S_IRUSR|S_IWUSR);      //Array of integers
 

static int __init hello_world_init(void)
{
        int i;
        printk(KERN_INFO "ValueETX = %d  \n", valueETX);
        printk(KERN_INFO "cb_valueETX = %d  \n", cb_valueETX);
        printk(KERN_INFO "NameETX = %s \n", nameETX);
        for (i = 0; i < (sizeof arr_valueETX / sizeof (int)); i++) {
                printk(KERN_INFO "Arr_value[%d] = %d\n", i, arr_valueETX[i]);
        }
        printk(KERN_INFO "Kernel Module Inserted Successfully...\n");
    return 0;
}

static void __exit hello_world_exit(void)
{
    printk(KERN_INFO "Kernel Module Removed Successfully...\n");
}
 
module_init(hello_world_init);
module_exit(hello_world_exit);
 
MODULE_LICENSE("GPL");
MODULE_AUTHOR("EmbeTronicX <embetronicx@gmail.com>");
MODULE_DESCRIPTION("A simple hello world driver");
MODULE_VERSION("1.0");
```

```c
sudo insmod hello_world_module.ko valueETX=14 nameETX="EmbeTronicX" arr_valueETX=100,102,104,106
```

## Chapter - 7
![Screenshot 2024-12-26 080447](https://github.com/user-attachments/assets/3320e6a3-4326-4674-ae64-73633a0ac4a8)

## Chapter - 8

- ls -l /dev/ | grep "crw"
- ls -l /dev/ | grep "brw"
  
![Screenshot 2024-12-26 081329](https://github.com/user-attachments/assets/033880a4-b8c7-4c1c-82c7-61a91910f302)

## Chapter - 9

There are 2 things
- Device creation in **/proc/devices**  { Here registering our device with a Major number }
- Device file creation **in /dev/~~~~**   { Accessing our device with device file operation defined in driver code }

![Screenshot 2024-12-26 082844](https://github.com/user-attachments/assets/dd1a3125-1f2a-4aba-99eb-cc22a65bd7da)

## Chapter 10

read function()
- read function actually writes **"len"** bytes of data from kernel space to user **"buf".**
- SO that user application can the data from **"buf"** and use it for further task.


open method
- It will first check whether the device is ready ( from Device Driver module function)
- Then if it is opening for the first time then initialize the device and Update the **f_op** pointer.
- Reserved the space for **file->private_data** structure and keep the data in allocated space.


## Chapter 11 - PCI device driver

PCIe - Pheripheral Components Interconnect ( Express)

PCI bus is used to connect GPU Card, LAN Card etc.

PCI Devices have 3 address region:
- Configuration Space 
- Input Output Space
- Device Memory


In order to send data from system to LCD screen, user needs to write pixel data in **frame buffer**, then LCD controller connects with it in timely basis and shows the fresh data in the screen. This memory space in the system called **DMA buffer.** 

![Screenshot 2025-01-01 082326](https://github.com/user-attachments/assets/e4e25a27-0805-4120-a45a-313d7f82f7f6)

## Chapter 12 - USB device driver
![Screenshot 2024-12-29 224409](https://github.com/user-attachments/assets/a1bdcf4d-17d8-4a3c-9d5f-35cf68446270)


## Chapter 13 - structure of USB device driver
four part of usb enpoints
- Control Endpoints := Used for configuration, command, and status exchanges between the host and the device.
- Bulk Endpoints := For large, non-time-critical data transfers
- Interrupt Endpoints := For small, time-sensitive data transfers that need regular polling
- Isochronous Endpoints := For real-time data streams where timely delivery is critical.

In USB device drivers, a URB (USB Request Block) is a data structure used to communicate with USB devices. It encapsulates all the information necessary for performing USB data transfers between the host and a USB device. The URB acts as a container for transfer requests and their associated parameters, allowing the USB core and drivers to manage USB communications efficiently.

![Screenshot 2025-01-01 135421](https://github.com/user-attachments/assets/834e7514-a658-43cf-8e92-b481e12e5564)

## Chapter 14 - USB device file operation

The gadget devices in USB subsystem has 2 types
- Storage
- Network


## Chapter 15 - Complete USB device driver program

## Chapter 16 - USB 3.0 special

