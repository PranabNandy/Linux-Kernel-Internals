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

# Coin Driver
```bash
root@localhost:~# cat /dev/coin
tail
root@localhost:~# cat /dev/coin
head
root@sauron:/# cat /sys/kernel/debug/coin/stats
head=6 tail=4
```
```cpp
drivers/char/Kconfig  |   16 +++++++++
drivers/char/Makefile |    1 +
drivers/char/coin.c   |   89 ++++++++++++++++++++++++++++++++++++++++
3 files changed, 106 insertions(+), 0 deletions(-)
```

### coin.c
```cpp
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/fs.h>
#include <linux/uaccess.h>
#include <linux/device.h>
#include <linux/random.h>
#include <linux/debugfs.h>

#define DEVNAME "coin"
#define LEN  20
enum values {HEAD, TAIL};

struct dentry *dir, *file;
int file_value;
int stats[2] = {0, 0};
char *msg[2] = {"head\n", "tail\n"};

static int major;
static struct class *class_coin;
static struct device *dev_coin;

static ssize_t r_coin(struct file *f, char __user *b,
                      size_t cnt, loff_t *lf)
{
        char *ret;
        u32 value = random32() % 2;
        ret = msg[value];
        stats[value]++;
        return simple_read_from_buffer(b, cnt,
                                       lf, ret,
                                       strlen(ret));
}

static struct file_operations fops = { .read = r_coin };

#ifdef CONFIG_COIN_STAT
static ssize_t r_stat(struct file *f, char __user *b,
                         size_t cnt, loff_t *lf)
{
        char buf[LEN];
        snprintf(buf, LEN, "head=%d tail=%d\n",
                 stats[HEAD], stats[TAIL]);
        return simple_read_from_buffer(b, cnt,
                                       lf, buf,
                                       strlen(buf));
}

static struct file_operations fstat = { .read = r_stat };
#endif

int init_module(void)
{
        void *ptr_err;
        major = register_chrdev(0, DEVNAME, &fops);
        if (major < 0)
                return major;

        class_coin = class_create(THIS_MODULE,
                                  DEVNAME);
        if (IS_ERR(class_coin)) {
                ptr_err = class_coin;
                goto err_class;
        }

        dev_coin = device_create(class_coin, NULL,
                                 MKDEV(major, 0),
                                 NULL, DEVNAME);
        if (IS_ERR(dev_coin))
                goto err_dev;

#ifdef CONFIG_COIN_STAT
        dir = debugfs_create_dir("coin", NULL);
        file = debugfs_create_file("stats", 0644,
                                   dir, &file_value,
                                   &fstat);
#endif

        return 0;
err_dev:
        ptr_err = class_coin;
        class_destroy(class_coin);
err_class:
        unregister_chrdev(major, DEVNAME);
        return PTR_ERR(ptr_err);
}

void cleanup_module(void)
{
#ifdef CONFIG_COIN_STAT
        debugfs_remove(file);
        debugfs_remove(dir);
#endif

        device_destroy(class_coin, MKDEV(major, 0));
        class_destroy(class_coin);
        return unregister_chrdev(major, DEVNAME);
}
```

### Kconfig
```cpp

#
# Character device configuration
#

menu "Character devices"

config COIN
       tristate "Coin char device support"
       help
         Say Y here if you want to add support for the 
         coin char device.

         If unsure, say N.

         To compile this driver as a module, choose M here: 
         the module will be called coin.

config COIN_STAT
        bool "flipping statistics"
        depends on COIN
       help
        Say Y here if you want to enable statistics about 
        the coin char device.
```
### Makefile
```
obj-$(CONFIG_COIN)    += coin.o
```

