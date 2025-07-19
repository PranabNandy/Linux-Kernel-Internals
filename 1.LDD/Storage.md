### 1.What does SMU_CDB_INVALID_INFO mean?
It means:

 The UFSHC attempted to access a memory address (CDB buffer) which is:

- Outside of the permitted SMU region OR

- Not mapped correctly in the **`DDR/IPMMU/IOMMU`** OR

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

🔄 Relation to UFSHC
UFSHC performs DMA for commands and data.

Both SMU and S2MPU ensure that the UFSHC only accesses memory it's allowed to, based on:

- Which VM is using it

- Which memory region was allocated for the buffer

If **CDB/PRDT** is not mapped properly, you will hit:

- `SMU_CDB_INVALID_INFO` (SMU violation)

- or **Stage-2 MMU fault** (S2MPU violation)

🧪 Example (Simplified)
If QNX is VM0 and Android is VM2:

- S2MPU will be configured such that:

    - UFSHC can access only **the CDB/PRDT memory allocated in VM0 when QNX owns UFS**

    - If Android tries to reuse the same memory region (outside its allowed S2MPU window), it gets blocked

- SMU, on the other hand, might block all access to `0x9000_0000–0x9FFF_FFFF` unless whitelisted for UFSHC


### 2. Write Booster feature from UFS 3.0 onwards

### 3. VH can send UFS Query for Device Management/ Device Control
