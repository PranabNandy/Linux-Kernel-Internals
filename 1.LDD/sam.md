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
