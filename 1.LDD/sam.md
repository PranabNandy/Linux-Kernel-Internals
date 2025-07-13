### What happens when we loads any driver
If you build it as a module without being embedded in the kernel (select **'m'** in the kernel's menuconfig) 
and then load it by the insmod command in the user's area. 

The insmod command loads the driver module and calls the **driver_init** to register the driver. 

Again, if the device for this is already loaded and can be **matched**, it **binds** and immediately **probes** the probe() function.
