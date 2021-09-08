# 操作系统内存管理学习笔记(Linux内核学习)

## 1.Linux内核整体架构及子系统

<img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210908184625839.png" alt="image-20210908184625839" style="zoom:50%;" />

内核对下管理硬件,对上通过运行时库对应用提供服务

![image-20210908200143443](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210908200143443.png)

1. 用户空间

   使用`malloc()`分配内存通过`free()`释放内存 

2. 内核空间

   虚拟进程负责从进程的虚拟地址空间分配虚拟页,`sys_brk`来扩大或收缩堆,`sys_mmap`负责在内存映射区分配虚拟页,

   页分配器负责分配物理页

   不连续内存分配器提供分配内存的接口`vmalloc`和释放内存接口`vfree`,申请连续的物理页的成功率比较低,可以申请不连续的物理页,r然后映射到连续的虚拟页,及虚拟地址连续而物理地址不连续

   **内存控制组来控制进程占用的资源,当内存碎片化的时候,找不到连续的物理页,内存碎片整理通过迁移方式得到连续的物理页,当内存不足的时候,负责回收物理页**

3. 硬件

   MMU^1^包含一个页表缓存,保存最近使用过的页表映射,避免每次都要查询虚拟页对应的物理页,解决了CPU执行速度与内存速度不匹配的问题



![image-20210908203424221](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210908203424221.png)

来个例子:

```cpp
#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
constexpr int MAX = 1024;

int main(int argc, char const *argv[])
{
    /*
    sbrk函数在内核的管理下将虚拟地址空间映射到内存，供malloc函数使用。
    */
    void *p = sbrk(0);
    void *old = p;

    p = (int *)sbrk(MAX * MAX);
    if (p == (void *)(-1))
    {
        std::cout << "sbrk error\n";
        exit(0);
    }
    printf("old:%p\tp=%p\n", p, old);

    void *new_ = sbrk(0);
    printf("new:%p", new_);
    sbrk(-MAX * MAX);

    return 0;
}

```

*输出*

```
old:0x55555557a000      p=0x55555557a000
new:0x55555567a000
```

可见成功分配了新的内存.

那我们如何更加具体的看到是否获取了内存呢?

更改代码如下:

```cpp
#include <iostream>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
constexpr int MAX = 1024;

int main(int argc, char const *argv[])
{
    /*
    sbrk函数在内核的管理下将虚拟地址空间映射到内存，供malloc函数使用。
    */
    void *p = sbrk(0);
    void *old = p;

    p = (int *)sbrk(MAX * MAX);
    if (p == (void *)(-1))
    {
        std::cout << "sbrk error\n";
        exit(0);
    }
    printf("old:%p\tp=%p\n", p, old);

    void *new_ = sbrk(0);
    printf("new:%p\n", new_);

    printf("pid= %d\n", getpid());

    while (true)
    {
    }

    sbrk(-MAX * MAX);

    return 0;
}

```

运行,不要停止

输出为:

```
old:0x55555557a000      p=0x55555557a000
new:0x55555567a000
pid= 1193
```

打开新终端输入`cat /proc/1193/maps `(一切皆文件)查看内存情况:

![image-20210908211016646](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210908211016646.png)

后续就不在演示了(懒)

## 2.虚拟内存空间内存架构

![image-20210908211415941](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210908211415941.png)



*linux-4.4.4\arch\arm64\include\asm\memory.h*

```cpp
#define TASK_SIZE_64		(UL(1) << VA_BITS)//64位操作系统  //VA_BITS 编译内核的时候选择的虚拟地址的位数

#ifdef CONFIG_COMPAT
#define TASK_SIZE_32		UL(0x100000000)    //32位操作系统
```

进程的用户虚拟地址空间包含区域:代码段、数据段、未初始化数据段;
动态库的代码段、数据段和未初始化数据段;存放动态生成的数据的堆;
存放局部变量和实现函数调用的栈;
把文件区间映射到虚拟地址空间的内存映射区域;存放在栈底部的环境变量和参数字符串。

部分内核代码:

```cpp
struct mm_struct {
	struct vm_area_struct *mmap;//虚拟内存区域链表		/* list of VMAs */
	struct rb_root mm_rb;//虚拟内存区红黑树
	u32 vmacache_seqnum;                   /* per-thread vmacache */
#ifdef CONFIG_MMU    //在内存映射区找到一个没有映射的区域
	unsigned long (*get_unmapped_area) (struct file *filp,
				unsigned long addr, unsigned long len,
				unsigned long pgoff, unsigned long flags);
    unsigned long mmap_base;	//内存映射区起始地址	/* base of mmap area */
	unsigned long mmap_legacy_base;         /* base of mmap area in bottom-up allocations */
	unsigned long task_size;	//用户虚拟空间长度	/* size of task vm space */
	unsigned long highest_vm_end;		/* highest vma end address */
	pgd_t * pgd;
	atomic_t mm_users;	//共享一个用户虚拟空间的进程数量,进程包含的线程的数量		/* How many users with user space? */
	atomic_t mm_count;			/* How many references to "struct mm_struct" (users count as 1) */
	atomic_long_t nr_ptes;	
    unsigned long hiwater_rss;//进程所拥有的最大页框数	/* High-watermark of RSS usage */
	unsigned long hiwater_vm;	// 最大页数/* High-water virtual memory usage */

	unsigned long total_vm;	//进程页数	/* Total pages mapped */
	unsigned long locked_vm;//锁住不能交换的页数	/* Pages that have PG_mlocked set */
	unsigned long pinned_vm;	/* Refcount permanently increased */
	unsigned long shared_vm;	/* Shared pages (files) */
	unsigned long exec_vm;		/* VM_EXEC & ~VM_WRITE */
	unsigned long stack_vm;		/* VM_GROWSUP/DOWN */
	unsigned long def_flags;
	unsigned long start_code, end_code, start_data, end_data;//代码段起始结束,数据段起始结束
	unsigned long start_brk, brk, start_stack;//堆 栈的起始地址
```



![image-20210908215430545](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210908215430545.png)



![image-20210908220500685](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210908220500685.png)



![image-20210908220535224](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210908220535224.png)

![image-20210908220623757](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210908220623757.png)



![image-20210908220734028](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210908220734028.png)

# 未完待续

[^1]: MMU是Memory Management Unit的缩写，中文名是[内存管理](https://baike.baidu.com/item/内存管理)单元，有时称作**分页内存管理单元**（英语：**paged memory management unit**，缩写为**PMMU**）。它是一种负责处理[中央处理器](https://baike.baidu.com/item/中央处理器)（CPU）的[内存](https://baike.baidu.com/item/内存)访问请求的[计算机硬件](https://baike.baidu.com/item/计算机硬件)。它的功能包括[虚拟地址](https://baike.baidu.com/item/虚拟地址)到[物理地址](https://baike.baidu.com/item/物理地址)的转换（即[虚拟内存](https://baike.baidu.com/item/虚拟内存)管理）、内存保护、中央处理器[高速缓存](https://baike.baidu.com/item/高速缓存)的控制，在较为简单的计算机体系结构中，负责[总线](https://baike.baidu.com/item/总线)的[仲裁](https://baike.baidu.com/item/仲裁)以及存储体切换（bank switching，尤其是在8位的系统上）。

