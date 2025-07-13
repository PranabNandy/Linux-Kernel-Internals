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
