# ioctrl
1.Define ioctl first

```cpp
#define "ioctl name" __IOX("magic number","command number","argument type")
```
where IOX can be :

- “IO“: an ioctl with no parameters
- “IOW“: an ioctl with write parameters (copy_from_user)
- “IOR“: an ioctl with read parameters (copy_to_user)
- “IOWR“: an ioctl with both write and read parameters

- The Magic Number is a unique number or character that will differentiate our set of ioctl calls from the other ioctl calls. some times the major number for the device is used here.

- Command Number is the number that is assigned to the ioctl. This is used to differentiate the commands from one another.

- The last is the type of data.
```cpp
#include <linux/ioctl.h>
#define WR_VALUE _IOW('a','a',int32_t*)
#define RD_VALUE _IOR('a','b',int32_t*)
```
2.Write IOCTL Function in the Driver:
```cpp
int  ioctl(struct inode *inode,struct file *file,unsigned int cmd,unsigned long arg)
```

- <inode>: is the inode number of the file being worked on.
- <file>: is the file pointer to the file that was passed by the application.
- <cmd>: is the ioctl command that was called from the userspace.
- <arg>: are the arguments passed from the userspace

- Within the function “ioctl” we need to implement all the commands that we defined above (WR_VALUE, RD_VALUE). We need to use the same commands in the switch statement which is defined above.
- Then we need to inform the kernel that the ioctl calls are implemented in the function “etx_ioctl“. This is done by making the fops pointer “unlocked_ioctl” to point to “etx_ioctl” as shown below.

```cpp

/*
** This function will be called when we write IOCTL on the Device file
*/
static long etx_ioctl(struct file *file, unsigned int cmd, unsigned long arg)
{
         switch(cmd) {
                case WR_VALUE:
                        if( copy_from_user(&value ,(int32_t*) arg, sizeof(value)) )
                        {
                                pr_err("Data Write : Err!\n");
                        }
                        pr_info("Value = %d\n", value);
                        break;
                case RD_VALUE:
                        if( copy_to_user((int32_t*) arg, &value, sizeof(value)) )
                        {
                                pr_err("Data Read : Err!\n");
                        }
                        break;
                default:
                        pr_info("Default\n");
                        break;
        }
        return 0;
}
/*
** File operation sturcture
*/
static struct file_operations fops =
{
        .owner          = THIS_MODULE,
        .read           = etx_read,
        .write          = etx_write,
        .open           = etx_open,
        .unlocked_ioctl = etx_ioctl,
        .release        = etx_release,
};
```

3. Create IOCTL Command in a Userspace Application

Just define the ioctl command like how we defined it in the driver.

```cpp
#define WR_VALUE _IOW('a','a',int32_t*)
#define RD_VALUE _IOR('a','b',int32_t*)

```
4. Use IOCTL System Call in Userspace

Include the header file <sys/ioctl.h>.Now we need to call the new ioctl command from a user application.

```cpp
long ioctl( "file descriptor","ioctl command","Arguments");

```

Where,

- <file descriptor>: This the open file on which the ioctl command needs to be executed, which would generally be device files.
- <ioctl command>: ioctl command which is implemented to achieve the desired functionality
- <arguments>: The arguments need to be passed to the ioctl command.

```cpp
ioctl(fd, WR_VALUE, (int32_t*) &number); 
ioctl(fd, RD_VALUE, (int32_t*) &value);
```


=================================================================
## main.c
```cpp
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/types.h>
#include <sys/stat.h>
#include <fcntl.h>
#include <unistd.h>
#include<sys/ioctl.h>
 
#define WR_VALUE _IOW('a','a',int32_t*)
#define RD_VALUE _IOR('a','b',int32_t*)
 
int main()
{
        int fd;
        int32_t value, number;
        printf("*********************************\n");
        printf("*******WWW.EmbeTronicX.com*******\n");
 
        printf("\nOpening Driver\n");
        fd = open("/dev/etx_device", O_RDWR);
        if(fd < 0) {
                printf("Cannot open device file...\n");
                return 0;
        }
 
        printf("Enter the Value to send\n");
        scanf("%d",&number);
        printf("Writing Value to Driver\n");
        ioctl(fd, WR_VALUE, (int32_t*) &number); 
 
        printf("Reading Value from Driver\n");
        ioctl(fd, RD_VALUE, (int32_t*) &value);
        printf("Value is %d\n", value);
 
        printf("Closing Driver\n");
        close(fd);
}
```



