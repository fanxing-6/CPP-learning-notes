# 【Linux内核】物理内存组织结构

## 系统调用mmap

![1-pdf-系统调用sys_mmap过程_00](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/48533643.png)

![image-20210911191801841](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202109111918945.png)

## 物理内存组织结构

### 体系结构

**目前多处理器系统有两种体系结构：** 

1）非一致内存访问（Non-Unit Memory Access，NUMA）：指内存被划分成多个 内存节点的多处理器系统。访问一个内存节点花费的时间取决于处理器和内存节点的距离。 

2）对称多处理器（Symmetric Multi-Processor，SMP）：即一致内存访问 （Uniform Memory Access，UMA），所有处理器访问内存花费的时间是相同。

![image-20210911192149473](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202109111921542.png)

![image-20210911192123271](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202109111921333.png)

### 内存模型

内存模型是从处理器角度看到的物理内存分布，内核管理不同内存模型的方式存差异。 

内存管理子系统支持3种内存模型： 

1） 平坦内存（Flat Memory）：内存的物理地址空间是连续的，没有空洞。 

2） 不连续内存（Discontiguous Memory）：内存的物理地址空间存在空洞，这种模 型可以高效地处理空洞。 

3） 稀疏内存（Space Memory）：内存的物理地址空间存在空洞，如果要支持内存热 插拔，只能选择稀疏内存模型。

