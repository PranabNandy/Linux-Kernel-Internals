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
