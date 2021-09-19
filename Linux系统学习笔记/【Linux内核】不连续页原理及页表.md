# 【Linux内核】不连续页原理及页表

## 不连续页分配器

当操作系统连续运行之后,很难找到连续的物理内存进行内存分配,这是就需要进行进行**不连续页分配**,使用不连续的物理内存,产生连续的虚拟内存

1. 不连续内存分配系统接口

   ```cpp
   void *vmalloc(unsigned long size); //分配不连续的物理页,并且把其映射到连续的虚拟空间\
   
   void vfree (const void * addr); //释放vmalloc分配的物理页和虚拟地址空间
   
   void *vmap(struct type **pages,unsigned int count,unsigned long flags,pgprot_t prot); //把已经分配的不连续的物理页映射到连续的虚拟地址空间
   
   void vunmap(const void *addr);//释放使用vmap分配的虚拟地址空间
   ```

2. 内核接口

   ```cpp
   void *kvmalloc(size_t size,gfp_t flags); //使用kmalloc分配连续的内存块,如果失败,使用vmalloc分配不连续的物理页
   
   void kvfree (const void * addr)//如果内存块使用vmalloc分配,那就使用vfree释放,否则使用kfree释放
   ```

数据结构

![image-20210915220744998](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202109152207101.png)

## 页表

​	**页表是一种特殊的数据结构，放在系统空间的页表区，存放逻辑页与物理页帧的对应 关系。 每一个进程都拥有一个自己的页表，PCB表中有指针指向页表。 **



