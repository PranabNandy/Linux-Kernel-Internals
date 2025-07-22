## How to add a custom device driver for Beagle Bone black in BSP ?
Some files to be added
- drivers/pranab/pcd.c
- drivers/pranab/Kconfig
- drivers/pranab/Makefile ( Kbuild for Kernel 6.1 onwards)
  
Then we have to mention these 2 things in drivers set up

- drivers/Makefile
``` obj-$(CONFIG_PRANAB) += pranab.o ```
- drivers/Kconfig
``` source "drivers/pranab/Kconfig" ```

Then for platform driver we have to have *.dtsi file
- arch/arm64/boot/dts/pranab/pranab-ev2.dtsi
``` #include "pranab-test.dtsi" ```
- arch/arm64/boot/dts/pranab/pranab-test.dtsi
```bash
/ {
      pranab_test {
                compatible = "pranab,my-test-driver";
                  base-address = <0x16CC8000>;
      };
};
```
drivers/pranab/Kconfig
```c++
config PRANAB
    tristate "Pranab pseudo device driver"
    default m
    help
      A simple test driver for BeagleBone Black.
```
drivers/pranab/Makefile
```c++
obj-$(CONFIG_PRANAB) +=pcd.o
```
- drivers/pranab/pcd.c
```c++
#include <linux/module.h>
#include <linux/platform_device.h>
#include <linux/of.h>  // For device tree matching

static int pdrv_probe(struct platform_device *pdev)
{
    pr_info("Platform Driver Probed: %s\n", dev_name(&pdev->dev));
    return 0;
}

static int pdrv_remove(struct platform_device *pdev)
{
    pr_info("Platform Driver Removed: %s\n", dev_name(&pdev->dev));
    return 0;
}

static const struct of_device_id pdrv_of_match[] = {
    { .compatible = "pranab,my-test-driver", },
    {},
};
MODULE_DEVICE_TABLE(of, pdrv_of_match);

static struct platform_driver my_platform_driver = {
    .driver = {
        .name = "my_platform_driver",
        .of_match_table = pdrv_of_match,
    },
    .probe  = pdrv_probe,
    .remove = pdrv_remove,
};

// module_platform_driver(my_platform_driver);  // This macro actaully expands to this 
static struct platform_device *pdev;

static int __init pdev_init(void)
{
    pdev = platform_device_register_simple(&my_platform_driver);
    return 0;
}

static void __exit pdev_exit(void)
{
    platform_device_unregister(pdev);
}

module_init(pdev_init);
module_exit(pdev_exit);


MODULE_LICENSE("GPL");
MODULE_AUTHOR("Pranab");
MODULE_DESCRIPTION("Basic Platform Driver Example");
```

### What happens when we loads any driver
- If you build it as a module without being embedded in the kernel (select **'m'** in the kernel's menuconfig) 
and then load it by the insmod command in the user's area. 

- The insmod command loads the driver module and calls the **driver_init** to register the driver. 

- Again, if the `device for this is already loaded` and can be **matched**, it **binds** and immediately **probes** the probe() function.


### About init.h 
- `init.h` has a variety of initcall functions.
- Let's take core_initcall as an example. `core_initcall(pranab_foo) `
- The macro is responsible for registering the `pranab_foo` (registering the init_func of the device).
- It is responsible for registering the function in the kernel when the system is booted, and there are other conditions for initcalls. In other words, you can set when to register a function via initcall.


### Why should I use platform_driver?
- In the early days, Linux users told the kernel that there was a specific device before using the hardware system.
- However, a huge number of devices do not automatically recognize the CPU. (More and more devices.) So, you have to tell the kernel what the hardware is like.
- Therefore, in order to prevent `hard coding inside the kernel`, we will run a **platform_device** and use it.

### What kind of flow does it have?
- Kernel initialization process creates platform bus Platform Device Registration
-  → At this time, the dev included in the platform Device is connected to the Bus.
-  When registering a platform driver, → the driver included in the platform driver is connected to the bus.
-  Platform Bus compares **the name of the platform Device** with the `name included in the driver` of the platform Driver.
-  If name is the same, probe() is called If the platform device or platform driver is later removed, remove() is called

  ### Probe function
- Usually, a probe function initializes a specific data structure or assigns a data structure.


```bash
/ {
compatible = "QC, pranab-sadk"
cpus {
        cpu@0{
              compatible = "arm, cortex-a9";
        };
        cpu@1{
              compatible = "arm, cortex-a9";
        };
};
serial@101F0000{
        compatible = "arm,pl011";
};
serial@101F2000{
        compatible = "arm,pl011";
};
gpio@101F3000{
        compatible = "arm,pl061";
};

interrupt-controller@10140000{
        compatible = "arm,pl190";
};
spi@10115000{
        compatible = "arm,pl022";
};

external-bus {
        ethernet@0,0{
              compatible = "smc, smc91c111";
        };
        i2c@1,0{
              compatible = "smc, smc91c111";
              rtc@58{
                    compatible = "maxim, ds1338";
            };
        };

        flash@2,0{
              compatible = "samsung, k8f1315ebm", "cfi-flash";
        };
};
}; // close of /

```


- The addition of CPU is expressed using a node with the node name "cpus".
- The compatible attribute is always promised to be expressed in string form in the order of "make, model".
- All nodes are expressed in the form of name@deviceaddress.

If there are no nodes that represent the same device or the same content, they can be omitted.

- Such is the case with "/" or "cpus".

Also, if there are multiple serial devices, it is common to distinguish them by the address of the serial device.

- (ex) serial@101f1000, serial@101f2000

The presence of an external bus can be described as an external-bus.

- You can see that the external buses are Ethernet, I2C bus, and Flash.

- Each also represents a companion as a child node.


The important thing to note here is that a comparable is not just one string, but a list of multiple strings.

- Also, you don't have to follow this format except for the first string.

If the same name is used, the operating system cannot distinguish between the devices and boards it is trying to handle.

- This problem is called a "**namespace conflict problem**," and it is very important to accurately describe the device name in order to resolve it.

Device Tree source path
- The source (script) of DTS is located in the arch/arm/boot/dts or arch/arm64/boot/dts path of the kernel.

- The source script files are organized by architecture.



There are also two files in it, which are as follows:

- DTB is a data base file, and it is created when the DTS file is compiled into DTC. (autocompiled on kernel load)

- DTS is a file that describes information about hardware devices, and DTSi is a file that is included in DTS. In other words, it uses DTS files and DTSi files to create DTB files.


## Makefile & Kconfig (Kernel 5.15 ) -> Kbuild & Kconfig ( kernel 6.6)


Make file is literally a script that generates a file.

```bash
obj-y += target.o --> built into the kernel
obj-m += target.o  --> module
obj -$(CONFIG_TEST) += target.o  --> The command that says it can be compiled as a module or as a built-in depending on the result of menuconfig

```
In the practice, I had to understand menuconfig because I was dealing with the form of registering and using drivers in menuconfig.


- menuconfig is created through a config file, which is automatically generated by referencing a file called kConfig.

<img width="927" height="285" alt="image" src="https://github.com/user-attachments/assets/0da0a1ff-c4b2-4a00-a8fa-f6768a97f5dc" />

- Therefore, you need to register the config in kConfig.

- Now, let's take a look at how to register Make file and kConfig, respectively.

<img width="682" height="332" alt="Screenshot 2025-07-13 133637" src="https://github.com/user-attachments/assets/80354ea4-b42f-4f68-8621-bd1616b59ab8" />

[1] Config --

- This is the most common command in Kconfig, and it is used to define menu entry.

- Usually after config is the name of ARM's feature. If you create a kernel option item called config ARM, it will eventually be a feature that will be created.

- #define defined as a statement, and any code can be inserted/removed depending on the situation. The kernel also uses this method to include only the necessary items.

[2] bool

- A bool can specify two states. The type can be tristate/ string / hex/ int.

- A bool has two states, and can be specified as y/n, etc.

[3] tristate

- tristate is in three states: Y/M/N, kernel insertion, module insertion, and subtraction.

[4] Prompt

- Every menu entry has a single prompt, which is a user-visible string (which is printed to the screen).

- prompt can be omitted, so

- bool "Exam Menu" → bool prompt "Exam Menu"

[5] default

- It is a command that sets the default value of the menu entry, e.g. y, n, etc.

[6] depends on

- This is a command about setting dependencies. This means that if another feature is selected, the current menu entry will be activated.

[7] select

- When the current menu is selected, other menu items are also selected.
