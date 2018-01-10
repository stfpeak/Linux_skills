## 编写一个简单的内核驱动

### hello.c
```c
#include "linux/init.h"
#include "linux/module.h"
#include "linux/kernel.h"

MODULE_LICENSE("Dual BSD/GPL");

static int hello_init(void)
{
    printk(KERN_ALERT "hello,I am dafeng/n");
    return 0;
}


static void hello_exit(void)

{
    printk(KERN_ALERT "goodbye,kernel/n");
}


module_init(hello_init);
module_exit(hello_exit);



MODULE_AUTHOR("dafeng");
MODULE_DESCRIPTION("This is a simple driver example!/n");
MODULE_ALIAS("A simplest driver example");

```

#### Makefile
```c
obj-m += hello.o
#generate the path
CURRENT_PATH:=$(shell pwd)
#the current kernel version number
LINUX_KERNEL:=$(shell uname -r)
#the absolute path
LINUX_KERNEL_PATH:=/usr/src/linux-headers-$(LINUX_KERNEL)
#complie object
all:
    make -C $(LINUX_KERNEL_PATH) M=$(CURRENT_PATH) modules
#clean
clean:
    make -C $(LINUX_KERNEL_PATH) M=$(CURRENT_PATH) clean
```

> ps: Makefile 要格外注意每一行的空格和tab，可以使用vim打开查看

#### 查看
* 编译ko文件
$ sudo make 

* 加载内核文件 
$ sudo insmod hello.ko 

* 查看内核文件是否加载成功 
$ sudo dmesg | tail -n 10
 
* 查看内核模块信息 
$ sudo modinfo hello.ko 
 
* 卸载内核模块 
$ sudo rmmod hello.ko 
