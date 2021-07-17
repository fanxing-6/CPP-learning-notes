# 侯捷 C++标准库体系结构与内核源码分析

STL分为六大组件:

- 容器(Containers)
- 分配器(Allocators)
- 算法(Algorithms)
- 迭代器(Iterators)
- 仿函数(Functors)

![image-20210717184423546](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210717184423546.png)

​	![image-20210717224534884](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210717224534884.png)

只有在大量数据情况下才会体现

## 容器

![image-20210717224703171](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210717224703171.png)

其中,红色框框为`C++11`新增