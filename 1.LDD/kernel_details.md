![Screenshot 2025-01-14 190850](https://github.com/user-attachments/assets/2eedee99-1ac3-436c-ad4a-68cc43c998be)
![Screenshot 2025-01-14 191710](https://github.com/user-attachments/assets/43335845-225a-4b0c-a081-20aa66c95e37)

# What is the responsibility of kernel ?

![Screenshot 2025-01-14 220618](https://github.com/user-attachments/assets/4ecc628a-0a07-4f82-994c-2167e4509e0c)

# Need for Platform Drivre
## Driver
- Hardware Descriptions
- Operations

## Comes with Device Model
- Hardware Descriptions (Deivce)
-  ||| Bus ||| ( PCI bus, USB bus )
- Operations (Driver)
  ===================================
- Hardware Descriptions (Platform Deivce)
-  ||| Bus ||| ( Platform bus )
- Operations (Platform Driver)


![Screenshot 2025-01-14 224908](https://github.com/user-attachments/assets/c270b7af-ddcb-4598-a594-5254e2a2d8c5)
- When we insert the driver then no device file was created
- As soon as, we inserted the device.ko then only device files were created

## Crux of file system 
- Algorithm to maintain the meta data of the files
- There are different algorithm designed in different fs like ext4, NTFS, FAT32 etc/
![Screenshot 2025-01-14 230505](https://github.com/user-attachments/assets/5f16eb3a-7c04-4222-abf1-ee81e7e1e170)

  

