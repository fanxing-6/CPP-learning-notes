# 代码风格

![image-20210813183714206](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210813183714206.png)

<img src="https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/original_363728fcf8b6b056188a2bba6a3e87ed.jpg" alt="original_363728fcf8b6b056188a2bba6a3e87ed" style="zoom:200%;" />

​    #include 用来包含文件,不只是头文件

<div class="se-32a24e79 " data-slate-type="paragraph" data-slate-object="block" data-key="778"><span data-slate-object="text" data-key="779"><span data-slate-leaf="true" data-offset-key="779:0" data-first-offset="true"><span data-slate-string="true">比如说，有一个用于数值计算的大数组，里面有成百上千个数，放在文件里占了很多地方，特别“碍眼”：</span></span></span></div>

```cpp

static uint32_t  calc_table[] = {  // 非常大的一个数组，有几十行
    0x00000000, 0x77073096, 0xee0e612c, 0x990951ba,
    0x076dc419, 0x706af48f, 0xe963a535, 0x9e6495a3,
    0x0edb8832, 0x79dcb8a4, 0xe0d5e91e, 0x97d2d988,
    0x09b64c2b, 0x7eb17cbd, 0xe7b82d07, 0x90bf1d91,
    ...                          
};
```

这个时候，你就可以把它单独摘出来，另存为一个“*.inc”文件，然后再用“#include”替换原来的大批数字。这样就节省了大量的空间，让代码更加整洁。

```cpp

static uint32_t  calc_table[] = {
#  include "calc_values.inc"        // 非常大的一个数组，细节被隐藏
};
```

![img](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/6739782414607164bdbe20fca7fd5fb3.jpg)

![img](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/6ec0c53ee9917795c0e2a494cfe70014.png)

![img](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/bdd9bb369fcbe65a8c879f37995a77dd.jpg)

![img](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/e5298af2501d0156fcc50d50cdb82351.jpg)

异常是个很重要的东西。
涉及磁盘操作的最好使用异常+调用栈，
涉及业务逻辑的最好利用日志+调用栈，
涉及指针和内存分配的还是用日志+调用栈吧，这种coredump一般是内存泄露和内存不够引起的

![img](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/5ac283e096d87e582fed017597ba4e0d.jpg)

![img](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/3301d0231ebb46c0e70d726af3cbc858.jpg)

![img](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/a39719e615f1124d60b5b9ca51b88cf1.png)

![img](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/3e07516e87c61172f9b2ddc317c74add.jpg)

![img](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/9b2d2c8285643a9202d822639fffe8e9.png)

