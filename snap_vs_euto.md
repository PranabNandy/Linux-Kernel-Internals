- What is beyond dcvs and what MPMM cover over it?
- How is SCP sending a packet for the first time?
- I need to understand the CacheDump flow?


# auto vs snap
![Screenshot from 2024-11-17 17-01-47](https://github.com/user-attachments/assets/3ba78ccf-95f1-41a2-9fee-5f5246c28a29)

## CPU
- **vsys** : channel to supply voltage
- **pdsys** : channel to supply power

### S2MPU 
- When separate VM has separate address space
### MPU 
- When we need separate address space for the process

### ASIC vs MCU
- ASIC is the application specific IC.
- It is useful for small application oriented task like USB controller
- MCU is the generic IC

### MPMM
- I_min : threshold I if crosses then mpmm hits up
- V_min : threshold V if crosses then mpmm hits up
- 
- **CPU utilization**
- Power Consumption
- Thermal limits
- Latency requirement
 
## Idle Power Management
![Screenshot from 2024-11-17 17-25-48](https://github.com/user-attachments/assets/80ec1976-458b-49b9-80c8-6916770a6e61)

![Screenshot from 2024-11-17 17-37-17](https://github.com/user-attachments/assets/7f90f0af-8f4a-4f4a-aead-50e1e45f47e0)
![Screenshot from 2024-11-17 17-46-15](https://github.com/user-attachments/assets/b3719f76-43f0-41d8-bccb-4e38b8d71cb1)

## SCMI 
![Screenshot from 2024-11-17 18-22-54](https://github.com/user-attachments/assets/d58bb98d-5ecc-476a-97f6-0ffccff6ea4e)

## cpu_freq Governor
![Screenshot from 2024-11-17 18-34-40](https://github.com/user-attachments/assets/50ab89d7-01ee-4351-b9dd-0772b0791e19)

## PMU ( Power Management Unit)
![Screenshot from 2024-11-17 18-50-59](https://github.com/user-attachments/assets/b63c5e88-85b7-43d9-a365-c1eb0d08d1c9)
![Screenshot from 2024-11-23 19-48-46](https://github.com/user-attachments/assets/4a559667-26ba-4bbb-81cf-1ee93c044ce8)

## Power Consumption
![Screenshot from 2024-11-23 20-19-50](https://github.com/user-attachments/assets/fab60184-188a-42d6-8c77-35d190f318f6)



