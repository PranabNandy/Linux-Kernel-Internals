

# Linux Device Driver

![Screenshot from 2024-08-12 08-08-53](https://github.com/user-attachments/assets/ed37ec5f-aef7-4ac5-a40a-0691f443488d)

### boot flow

![Screenshot from 2024-08-07 07-40-58](https://github.com/user-attachments/assets/f02f72c6-7078-4cc6-847c-69f951f58530)


Device Drivers to some extent develop the **System Calls**
2 types of Drivers:
1. Boot time drivers / built-in driver
2. Loading drivers in run time i.e when the system is live / Out of tree module
![Screenshot from 2024-08-10 16-44-03](https://github.com/user-attachments/assets/021a53dc-7edb-4781-a299-65e501f84ec9)


what are these Macros do "`module_init`" and "`module_exit`"?
This is the process using which, basically during the compilation process
in **.ko** file, 2 symbols are exported 
**pranab_init**
**pranab_exit**
by consuming the .ko file kernel got to know which symbol to invoke during adding a driver and removing a driver.

![Screenshot from 2024-08-11 11-37-19](https://github.com/user-attachments/assets/2316849e-0301-4c58-9690-b48ed2bde567)
![Screenshot from 2024-08-11 11-40-38](https://github.com/user-attachments/assets/2f4daf8a-a378-4e0d-92af-aefd48855175)
![Screenshot from 2024-08-11 12-29-10](https://github.com/user-attachments/assets/86f562e0-7c55-4c4d-957c-52f020e33f35)



make is an **Uility**
It is used for build system as **recipe**

![Screenshot from 2024-08-11 13-58-21](https://github.com/user-attachments/assets/f3bf448c-e6f7-468b-8c9d-26f03f4d7660)

If the function returns **-ve number** then, the kernel returns an message to `insmod` that I can not insert this module and `lsmod` will not this module.

![Screenshot from 2024-08-11 15-36-24](https://github.com/user-attachments/assets/cb8dd508-f3a2-4850-9697-7b42f5c40fc6)

### insmod
- what happens behind the scene
![Screenshot from 2024-08-11 17-04-34](https://github.com/user-attachments/assets/973e6922-60c7-48ee-9b63-7920fe04de45)

### rmmod
- what happens behind the scene
- ![Screenshot from 2024-08-11 22-30-12](https://github.com/user-attachments/assets/e6093d52-13b0-417f-9265-0c39c682d32b)

### modinfo
- what kind of information is getting added in module
![Screenshot from 2024-08-12 07-26-18](https://github.com/user-attachments/assets/6d57616d-d08d-467b-829f-2073b0a859de)

### proc fs
- It is a vfs i.e logically exist but physically does not exist
- /dev/ is a real file system (char files and block files)

![Screenshot from 2024-08-12 07-58-33](https://github.com/user-attachments/assets/06aefbd8-36ac-4017-983d-0d35dc9eb5da)
![Screenshot from 2024-08-12 07-59-22](https://github.com/user-attachments/assets/a1cc97ad-db71-4579-a106-f84d05994966)

![Screenshot from 2024-08-12 08-05-54](https://github.com/user-attachments/assets/cb351bd8-778b-4ad2-a085-2ae7532d7647)
![Screenshot from 2024-08-12 08-03-08](https://github.com/user-attachments/assets/ab00970a-06c9-4390-8703-b6556b528828)

- **file * is a very important cmd**
- but proc file system won't show the type of the file

![Screenshot from 2024-08-12 08-38-25](https://github.com/user-attachments/assets/76167379-8164-43d0-88cf-594f4e32970b)



## driver with proc_fs
![Screenshot from 2024-08-13 08-29-04](https://github.com/user-attachments/assets/25d2e760-0a2c-40a6-b9db-c706024f09c3)


