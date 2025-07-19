### 1.What does SMU_CDB_INVALID_INFO mean?
It means:

 The UFSHC attempted to access a memory address (CDB buffer) which is:

- Outside of the permitted SMU region OR

- Not mapped correctly in the DDR/IPMMU/IOMMU OR

- Violating some access control policy

This results in:

- A `SMU violation interrupt` or exception

- The SMU_CDB_INVALID_INFO log message (from SMU or UFS driver/BSP log)




✅ How to Debug This
Check your CDB buffer allocation:

- Use `dma_alloc_coherent()` or `dma_alloc_attrs()` with **DMA_TO_DEVICE** for CDB

- Avoid kmalloc() or vmalloc() unless you’re mapping manually

Inspect SMU configuration:

- Is the UFSHC master ID allowed access to the DDR region of CDB?

- Check platform SMU config in QNX BSP or device tree (on Linux)

### 2. Write Booster feature from UFS 3.0 onwards

### 3. VH can send UFS Query for Device Management/ Device Control
