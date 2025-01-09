File system structure is stored in the first block of the disk. Root of the file system structure contains how many files are there in the system. 

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

