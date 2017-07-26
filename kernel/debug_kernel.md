## 内核调试手段
* jprobe
* gdb + qemu
* printk

## 开发一个内核调试器
利用 DRn 和 int 1中断， 然后利用ioctl 和 read函数来获取内核寄存器、调用栈等信息
