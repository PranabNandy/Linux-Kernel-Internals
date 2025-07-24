<img width="792" height="507" alt="Screenshot 2025-07-24 221115" src="https://github.com/user-attachments/assets/ea8a6eb6-911d-4d7d-adac-4e50a73f5ecf" />

<img width="1430" height="482" alt="Screenshot 2025-07-24 225202" src="https://github.com/user-attachments/assets/6a988797-aab2-4277-91c6-998dc1099279" />

<img width="1725" height="858" alt="image" src="https://github.com/user-attachments/assets/30597fca-f11b-4180-8ac0-3e0cd2b3c3b1" />



## Linux List API ( linux/list.h )
<img width="1653" height="648" alt="image" src="https://github.com/user-attachments/assets/950b6d85-1fb3-4d5a-95fc-7c4580f8e9cb" />


```c++
#include <linux/init.h>
#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/list.h>

static LIST_HEAD(head);   // 1st API

struct bookkeeping {
    int id;
    int long_id;
    struct list_head list_meta;    // 2nd API
};

static int __init dog_init(void) {
    printk(KERN_INFO "linked list experiments\n");
    struct bookkeeping *data1, *data2;

    data1 = kmalloc(sizeof(struct bookkeeping), GFP_KERNEL);
    if (!data1) {
        printk(KERN_INFO "memory allocation failed!\n");
        return -1;
    }

    data1->id = 10;
    data1->long_id = 123410;

    list_add(&(data1->list_meta), &head);   // 3rd API

    data2 = kmalloc(sizeof(struct bookkeeping), GFP_KERNEL);
    if (!data2) {
        printk(KERN_INFO "memory allocation failed!\n");
        return -1;
    }

    data2->id = 20;
    data2->long_id = 100020;

    list_add_tail(&(data2->list_meta), &head);   // 4th API

    struct bookkeeping *cur;
    list_for_each_entry(cur, &head, list_meta) {    // 5th API
        printk(KERN_INFO "node id: %d\n", cur->id);
    }

    printk(KERN_INFO "removing %d\n", data2->id);
    list_del(&(data2->list_meta));   // 6th API
    kfree(data2);

    list_for_each_entry(cur, &head, list_meta) {
        printk(KERN_INFO "node id: %d\n", cur->id);
    }
    return 0;
}

static void __exit dog_exit(void) {
    struct bookkeeping *cur;
    printk(KERN_INFO "driver unload\n");
    list_for_each_entry(cur, &head, list_meta) {
        printk(KERN_INFO "node id: %d\n", cur->id);
        kfree(cur);
    }
    printk(KERN_INFO "module unloaded!\n");
}

module_init(dog_init);
module_exit(dog_exit);
```
