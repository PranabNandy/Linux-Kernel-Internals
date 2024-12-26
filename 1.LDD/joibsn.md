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
