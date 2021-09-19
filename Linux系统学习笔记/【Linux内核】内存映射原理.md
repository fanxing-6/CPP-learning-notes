# 【Linux内核】内存映射原理

## 物理地址空间

1. 物理地址是处理器在总线上能看到的地址,使用RISC(Reduced Instruction Set Computing精简指令集)的处理器通常只实现一个物理地址空间,外围设备和物理内存使用统一的物理空间,

   有些架构的处理器把分配给外围设备的物理地址称为设备内存

   处理器通过外围设备控制器里面的寄存器来访问外围设备,寄存器分为控制寄存器,状态寄存器和数据寄存器

   外围设备的寄存器通常被常备连续的编址,处理器对外围设备寄存器的编址方式分为两种:I/O映射方式,内存映射方式

   - IO映射方式:x86处理器专门为外围设备单独提供一个空间,通过单独的指令(如in out)来访问这个空间的地址
   - 内存映射方式:外围设备和物理内存使用同一个物理空间,处理器以相同方式访问物理内存和外围设备,那么应用如何访问外围设备呢? 操作系统提供单独的函数将外围设备对应的地址映射到应用的虚拟内存中,从而使应用能访问设备

   ARM64架构分为两种内存类型:

   - 正常内存(Normal Memory) :包括物理内存和只读存在器(ROM) ;

     `对于正常内存来讲可以设置共享属性,包括缓冲区的共享属性,共享属性分为不可共享,内部共享和外部共享.不可共享:制备处理器一个核心使用,内部共享:可以被多个核使用,外部共享:可以被DMI(DMI是指Direct Media InterfaceI(直接媒体接口))等共享`[^1]

   - 设备内存(Device Memory)∶指分配给外围设备寄存器的物理地址区域

     `设备内存的共享属性总是外部共享`

## 内存映射原理

​	创建内存映射时,在进程的用户虚拟地址空间中分配一个虚拟内存区域,内核采用一个延迟物理内存分配策略,在进程第一次访问虚拟页的时候产生缺页异常[^2]

​	如果是文件映射,那么分配物理页把文件指定的区域数据读到物理页当中,然后把应用的虚拟页表映射到物理页,

​	如果是匿名映射,就分配物理页,让后把虚拟页映射到物理页

**内存映射即在进程的虚拟地址空间创建一个映射,分为两种:**

- 文件映射:文件支持内存映射,把文件的一段区域映射进程的虚拟空间,文件源式存储设备上的文件

  `两个进程可以使用共享的文件映射实现共享内存`

- 匿名映射[^3]:没有文件支持的映射,把物理内存映射到进程里的虚拟空间,无数据源

  `匿名映射通常是私有映射,只可能出现在父进程和子进程之间`

在进程的虚拟地址空间中,代码段和数据段是私有文件映射

未初始化的数据段,堆栈是私有的匿名映射

**线程启动映射过程,并且在虚拟地址空间中为内存映射创建一个映射区,先在用户空间调用mmap函数,并且在内存虚拟空间中找到一段连续空闲的符合要求的虚拟地址,对这个区域初始化,并插入进程虚拟空间地址的链表,让后再内核中系统调用,实现文件的物理地址和进程虚拟地址之间一一对应的关系,进程开始查询文件内容,这是虽然进程虚拟内存与文件物理地址已经建立映射关系,但由于文件相关区域还没有加载到物理内存中,这是就会引发缺页中断,请求将磁盘内容调入内存当中,先在进程缓存空间swapcathe^4^中进行查找,如果没有找到,就通过nopage()把缺页从磁盘调入内存,如果你对内存进行写入改变内存内容,一段时间后,系统就会将改变的脏页写入硬盘,也可以使用函数强制及时同步.**



## 数据结构

```cpp
struct vm_area_struct {
	/* The first cache line has the info for VMA tree walking. */

	unsigned long vm_start;		/* Our start address within vm_mm. */
	unsigned long vm_end;		/* The first byte after our end address
					   within vm_mm. */

	/* linked list of VM areas per task, sorted by address */
	struct vm_area_struct *vm_next, *vm_prev;//虚拟内存连接
//红黑树实现
	struct rb_node vm_rb;

	/*
	 * Largest free memory gap in bytes to the left of this VMA.
	 * Either between this VMA and vma->vm_prev, or between one of the
	 * VMAs below us in the VMA rbtree and its ->vm_prev. This helps
	 * get_unmapped_area find a free area of the right size.
	 */
	unsigned long rb_subtree_gap;

	/* Second cache line starts here. */

	struct mm_struct *vm_mm;//指向内存描述符,虚拟内存区域所属的用户虚拟空间	/* The address space we belong to. */
	pgprot_t vm_page_prot;	//权限呗	/* Access permissions of this VMA. */
	unsigned long vm_flags;		/* Flags, see mm.h. */
}
struct vm_operations_struct {
    //虚拟内存操作集合
	void (*open)(struct vm_area_struct * area);//创建虚拟内存区域
	void (*close)(struct vm_area_struct * area);//关闭虚拟内存区域
	int (*mremap)(struct vm_area_struct * area);//移动虚拟内存区域是调用
	int (*fault)(struct vm_area_struct *vma, struct vm_fault *vmf);//缺页异常处理,将文件加载到内存中
	int (*pmd_fault)(struct vm_area_struct *, unsigned long address,
						pmd_t *, unsigned int flags);
	void (*map_pages)(struct vm_area_struct *vma, struct vm_fault *vmf);

	/* notification that a previously read-only page is about to become
	 * writable, if an error is returned it will cause a SIGBUS */
	int (*page_mkwrite)(struct vm_area_struct *vma, struct vm_fault *vmf);

	/* same as page_mkwrite when using VM_PFNMAP|VM_MIXEDMAP */
	int (*pfn_mkwrite)(struct vm_area_struct *vma, struct vm_fault *vmf);
}
```

![image-20210909224459409](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210909224459409.png)



## 系统调用

​	应用程序通常使用C标准库提供的函数malloc()申请内存,glibc库的内存分配器ptmalloc使用brk或mmap向内核以页为单位申请虚拟内存然后把页划分为小块共应用程序使用

​	应用程序可以使用mmap向内核申请虚拟内存



1、`mmap()`----创建内存映射 

```cpp
#include <sys/mman.h> 

void *mmap(void *addr，size_t length，int prot，int flags，int fd，off_t offset);
```

系统调用`mmap()`：进程创建匿名的内存映射，把内存的物理页映射到进程的虚拟地址空间。进程把文件映射到进程的虚拟地址空间，可以像访问内存一样访问文件，不需要调用系统调用read()/write()访问文件，从而避免用户模式和内核模式之间的切换，提高读写文件速度。 两个进程针对同一个文件创建共享的内存映射，实现共享内存。 

2、`munmap()`----删除内存映射 

```cpp
#include <sys/mman.h> 

int munmap(void *addr, size_t len); 
```



3、`mprotect()`----设置虚拟内存区域的访问权限 

```cpp
#include <sys/mman.h> 

int mprotect(void *addr, size_t len, int prot);
```

1、进程启动映射过程，并且在虚拟地址空间中为映射创建虚拟映射区域;
2、调用内核空间的系统调用函数mmap(不同于用户空间函数)，实现文件物理地址和进程虚拟的一一映射关系;
3、进程发起对这片映射空间的访问，引发缺页异常，实现文件内容到物理内存（主存）的拷贝。

上代码:

```cpp
#include <errno.h>
#include <fcntl.h>
#include <stdio.h>
#include <string.h>
#include <sys/mman.h>
#include <sys/types.h>
#include <unistd.h>

typedef struct
{
    /* data */
    char name[4];
    int age;
} people;

void main(int argc, char **argv)
{
    int fd, i;
    people *p_map;
    char temp;
    fd = open(argv[1], O_CREAT | O_RDWR | O_TRUNC, 00777);

    lseek(fd, sizeof(people) * 5 - 1, SEEK_SET);
    write(fd, "", 1);

    p_map = (people *)mmap(NULL, sizeof(people) * 10, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    if (p_map == (void *)-1)
    {
        fprintf(stderr, "mmap : %s \n", strerror(errno));
        return;
    }
    close(fd);

    temp = 'A';
    for (i = 0; i < 10; i++)
    {
        temp = temp + 1;
        (*(p_map + i)).name[1] = '\0';
        memcpy((*(p_map + i)).name, &temp, 1);
        (*(p_map + i)).age = 30 + i;
    }

    printf("Initialize.\n");

    sleep(15);

    munmap(p_map, sizeof(people) * 10);

    printf("UMA OK.\n");
}

```



```cpp
#include <sys/mman.h>
#include <sys/types.h>
#include <fcntl.h>
#include <string.h>
#include <stdio.h>
#include <unistd.h>
#include <errno.h>

typedef struct 
{
    /* data */
    char name[4];
    int age;
}people;

void main(int argc,char**argv)
{
    int fd,i;
    people *p_map;

    fd=open(argv[1],O_CREAT|O_RDWR,00777);
    p_map=(people*)mmap(NULL,sizeof(people)*10,PROT_READ|PROT_WRITE,MAP_SHARED,fd,0);
    if(p_map==(void*)-1)
    {
        fprintf(stderr,"mmap : %s \n",strerror(errno));
        return ;
    }

    for(i=0;i<10;i++)
    {
        printf("name:%s age:%d\n",(*(p_map+i)).name,(*(p_map+i)).age);
    }

    munmap(p_map,sizeof(people)*10);   

}



```



[^1]: DMI是Intel(英特尔)公司开发用于连接主板南北桥的总线，取代了以前的Hub-Link总线。DMI采用点对点的连接方式，时钟频率为100MHz，由于它是基于PCI-Express总线，因此具有PCI-E总线的优势。DMI实现了上行与下行各1GB/s的数据传输率，总带宽达到2GB/s，这个高速接口集成了高级优先服务，允许并发通讯和真正的同步传输能力。它的基本功能对于软件是完全透明的，因此早期的软件也可以正常操作。
[^2]: 缺页异常，页缺失Page fault，指的是硬错误、硬中断、分页错误、寻页缺失、缺页中断、页故障等)指的是当软件试图访问已映射在虚拟地址空间中，但是目前并未被加载在物理内存中的一个分页时，由中央处理器的内存管理单元所发出的中断。通常情况下，用于处理此中断的程序是操作系统的一部分。如果操作系统判断此次访问是有效的，那么操作系统会尝试将相关的分页从硬盘上的虚拟内存文件中调入内存。而如果访问是不被允许的，那么操作系统通常会结束相关的进程。虽然其名为“页缺失”错误，但实际上这并不一定是一种错误。而且这一机制对于利用虚拟内存来增加程序可用内存空间的操作系统(比如Microsoft Windows和各种类 Unix 系统)中都是常见且有必要的。
[^3]: mmap是一种内存映射文件的方法，即将一个文件或者其它对象映射到进程的地址空间，实现文件磁盘地址和进程虚拟地址空间中一段虚拟地址的一一对映关系。实现这样的映射关系后，进程就可以采用指针的方式读写操作这一段内存，而系统会自动回写脏页面到对应的文件磁盘上，即完成了对文件的操作而不必再调用read,write等系统调用函数。相反，内核空间对这段区域的修改也直接反映用户空间，从而可以实现不同进程间的文件共享。进程在用户空间调用库函数mmap，原型：void *mmap(void *start, size_t length, int prot, int flags, int fd, off_t offset);来实现内存映射。从原型可知，存在一个参数为fd，根据fd，存在一种情况叫匿名映射，所谓匿名映射，表示不存在fd这么个真实的文件。实现匿名映射的方式主要有以下两种： 1、BSD 提供匿名映射的办法是fd =-1，同时 flag 指定为MAP_SHARE|MAP_ANON。    ptr = mmap（NULL，sizeof（int），PROT_READ|PROT_WRITE，          MAP_SHARED|MAP_ANON，-1,0）；2、SVR4 提供匿名映射的办法是 open  /dev/zero设备文件，把返回的文件描述符，作为mmap的fd参数。    fd = open（"/dev/zero",O_RDWR）; /dev/zero 是一个特殊的文件，当你读它的时候，它会提供无限的空字符(NULL, ASCII NUL, 0x00)  一个作用是用它作为源，产生一个特定大小的空白文件。匿名内存映射适用于具有亲属关系的进程之间；由于父子进程之间的这种特殊的父子关系，在父进程中先调用mmap()，然后调用fork()，那么，在调用fork() 之后，子进程继承了父进程的所有资源，当然也包括匿名映射后的地址空间和mmap()返回的地址，这样父子进程就可以通过映射区域进行通信了；这里不是一般的继承关系，一般来说，子进程单独维护从父进程继承下来的一些变量，而mmap()返回的地址却是由父子进程共同维护的；对于具有亲属关系的进程之间实现共享内存的最好方式应该是采用匿名映射的方式。此时，不必指定具体的条件，只要设置相应的标志即可。
[^4]: 安装Linux时需要两个分区，一个是根目录，另外一个就是swap(内存交换空间)。swap的功能就是在应付物理内存不足的情况下所造成的内存扩展记录的功能。一般来说，如果硬件的配备足够的话，那么swap应该不会被我们的系统所使用到。　　CPU所读取的数据都来自内存，当内存不足的时候，为了让后续的程序可以顺利运行，因此在内存中暂不使用的程序与数据就会被挪到swap中了。此时内存就会空出来给需要执行的程序加载。swap是用硬盘来暂时放置内存中的信息。

