## Linux内核调试技术——jprobe使用与实现

### jprobe使用实例
使用jprobe探测函数的入参值，需要编写内核模块。同kprobe一样，内核同样提供了jprobe的实例程序jprobe_example.c（位于sample/kprobes目录），该程序实现了探测do_fork函数入参的功能，用户可以以它为模板来探测其他函数（当然不是说什么函数都能探测的，限制同kprobe一样，另外需要注意的是一个被探测函数只能注册一个jprobe）。在分析jprobe_example.c之前先熟悉一下jprobe的基本结构与API接口。

#### jprobe结构体与API介绍
struct jprobe结构体定义如下：
```c
/*
 * Special probe type that uses setjmp-longjmp type tricks to resume
 * execution at a specified entry with a matching prototype corresponding
 * to the probed function - a trick to enable arguments to become
 * accessible seamlessly by probe handling logic.
 * Note:
 * Because of the way compilers allocate stack space for local variables
 * etc upfront, regardless of sub-scopes within a function, this mirroring
 * principle currently works only for probes placed on function entry points.
 */  
struct jprobe {  
    struct kprobe kp;  
    void *entry;    /* probe handling code to jump to */  
};  
```

该结构非常的简单，仅包含了一个kprobe结构（因为它是基于kprobe实现的）和一个entry指针，它保存的是探测点执行回调函数的地址，当触发调用被探测函数时，保存到该指针的地址会作为目标地址跳转执行（probe handling code to jump to），因此用户指定的探测函数得以执行。

相关的API如下：
```c
int register_jprobe(struct jprobe *jp)      //向内核注册jprobe探测点    
void unregister_jprobe(struct jprobe *jp)   //卸载jprobe探测点    
int register_jprobes(struct jprobe **jps, int num)     //注册探测函数向量，包含多个不同探测点    
void unregister_jprobes(struct jprobe **jps, int num)  //卸载探测函数向量，包含多个不同探测点    
int disable_jprobe(struct jprobe *jp)       //临时暂停指定探测点的探测    
int enable_jprobe(struct jprobe *jp)        //恢复指定探测点的探测  
```

#### 示例jprobe_example分析与演示
示例程序仍以do_fork作为被探测函数进行探测。当创建进程时，探测函数会调用它打印出do_fork函数的入参值。程序定义了一个struct jprobe实例my_jprobe，指定被探测函数的名字是do_fork（可以修改它以达到探测其他函数的目的），然后探测回调函数为jdo_fork。在模块的初始化函数中，调用register_jprobe函数向kprobe子系统注册my_jprobe，这样jprobe探测默认就启用了，最后在exit函数中调用unregister_jprobe函数卸载。 下面详细分析：
```c
/* console whenever _do_fork() is invoked to create a new process.
 * (Some messages may be suppressed if syslogd is configured to
 * eliminate duplicate messages.)
 */

#include <linux/kernel.h>
#include <linux/module.h>
#include <linux/kprobes.h>

/*
 * Jumper probe for _do_fork.
 * Mirror principle enables access to arguments of the probed routine
 * from the probe handler.
 */

/* Proxy routine having the same arguments as actual _do_fork() routine */
static long j_do_fork(unsigned long clone_flags, unsigned long stack_start,
	      unsigned long stack_size, int __user *parent_tidptr,
	      int __user *child_tidptr)
{
    //TO-DO
    // if(xxxx && xxx)
    //     printk();
    // dump_stack();

	pr_info("jprobe: clone_flags = 0x%lx, stack_start = 0x%lx "
		"stack_size = 0x%lx\n", clone_flags, stack_start, stack_size);

	/* Always end with a call to jprobe_return(). */
	jprobe_return();
	return 0;
}

static struct jprobe my_jprobe = {
	.entry			= j_do_fork,
	.kp = {
		.symbol_name	= "_do_fork",
	},
};

static int __init jprobe_init(void)
{
	int ret;

	ret = register_jprobe(&my_jprobe);
	if (ret < 0) {
		printk(KERN_INFO "register_jprobe failed, returned %d\n", ret);
		return -1;
	}
	printk(KERN_INFO "Planted jprobe at %p, handler addr %p\n",
	       my_jprobe.kp.addr, my_jprobe.entry);
	return 0;
}

static void __exit jprobe_exit(void)
{
	unregister_jprobe(&my_jprobe);
	printk(KERN_INFO "jprobe at %p unregistered\n", my_jprobe.kp.addr);
}

module_init(jprobe_init)
module_exit(jprobe_exit)
MODULE_LICENSE("GPL");

```

Makefile
```c

```

jdo_fork函数也仅仅打印出了在调用do_fork函数时传入的clone_flags、stack_start和stack_size这三个入参值，整个实现非常简单直观，但是有两点需要注意：

1）探测回调函数的入参必须同被探测函数的一致，否则无法达到探测函数入参的目的，例如此处的jdo_fork函数入参unsigned long clone_flags、unsigned long stack_start、unsigned long stack_size、int __user *parent_tidptr和int __user *child_tidptr同do_fork函数是完全一致的（注意返回值固定为long类型）。

2）在回调函数执行完毕以后，必须调用jprobe_return函数（注释中也有强调），否则执行流程就回不到正常的执行流程中了，这一点后文会详细分析。

### 参考
* [Linux内核调试技术——jprobe使用与实现 ](http://blog.csdn.net/luckyapple1028/article/details/54350410#)
