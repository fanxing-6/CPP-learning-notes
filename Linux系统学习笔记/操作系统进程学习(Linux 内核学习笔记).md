# 操作系统进程学习(Linux 内核学习笔记)

## 进程优先级

并非所有进程都具有相同的重要性。除了大多数我们所熟悉的进程优先级之外，进程还有不同的关键度类别，以满足不同需求。首先进程比较粗糙的划分，进程可以分为**实时进程** 和**非实时进程（普通进程）**。 



实时进程优先级（0-99）都比普通 进程的优先级（100-139）高。当系统中有实时进程运行时，普通进程几乎无法分到时间片（只能分到5%的CPU时间）。

![通过时间片运行进程](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210903161607827.png)



## 进程系统调用

​	讨论fork和exec系列系统调用的实现。通常这些调用不是由应用程序直接发出的，而是通过一个中间层调用，即负责与内核通信的C标准库。从用户状态切换到核心态的方法，依不同的体系结构而各有不同

用户发起一个新进程之后,CPU为进程分配资源,并将硬盘数据读到内存中去,但是用户进程是一个应用级程序,无法直接与CPU进行交互,所以通过系统调用(system_call),加载完成内核就会退出CPU,用户进程执行完毕后,内核进入CPU将用户进程移除,下一个同上![image-20210903163111488](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210903163111488.png)

1. 进程复制

   (1) fork是重量级调用，因为它建立了父进程的一个完整副本，然后作为子进程执行。 为减少与该调用相关的工作量，Linux使用了写时复制（copy-on-write)^1^技术。 

   (2) vfork类似于fork，但并不创建父进程数据的副本。相反，父子进程之间共享数据。 这节省了大量CPU时间（如果一个进程操纵共享数据，则另一个会自动注意到）。 

   (3) clone产生线程，可以对父子进程之间的共享、复制进行精确控制。

## 调度器分析

1. 调度器及其功能

   内核中用来安排进程执行的模块称为调度器（scheduler），它可以切换进程状态 （process state）。例如执行、可中断睡眠、不可中断睡眠、退出、暂停等。 调度器是CPU中央处理器的管理员，主要负责完成做两件事情：一、选择某些就绪进 程来执行，二是打断某些执行的进程让它们变为就绪状态。调度器分配CPU时间的基本依据就 是进程的优先级。上下文 切换（context switch）：将进程在CPU中切换执行的过程，内核承担 此任务，负责重建和存储被切换掉之前的CPU状态

主要有两个函数可以获得线程设置的最高和最低优先级:

```cpp
int sched_get_priority_max(int); // 获取实时优先级最大值
int sched_get_priority_min(int); // 获取实时优先级最小值
```

设置与获取优先级各个主要函数:

```cpp
int pthread_attr_setschedparam(pthread_attr_t*,const struct sched_param);//创建线程优先级
int pthread_attr_getschedparam(pthread_attr_t*,const struct sched_param);//获取线程优先级
```



**实战:**

```cpp
#include <assert.h>
#include <iostream>
#include <pthread.h>
#include <sched.h>
#include <stdio.h>

static int get_thread_policy(pthread_attr_t *attr) //获取线程调度策略
{
    int plicy;
    int rs = pthread_attr_getschedpolicy(attr, &plicy);
    assert(rs == 0);
    switch (plicy)
    {
    case SCHED_FIFO:
        printf("policy = FIFO");
        break;
    case SCHED_RR:
        printf("policy = RR");
        break;
    case SCHED_OTHER:
        printf("policy=OTHER");
        break;
    default:
        printf("policy = unknown");
        break;
    }
    return plicy;
}

static void show_thread_priority(pthread_attr_t *attr, int policy)
{
    int priority = sched_get_priority_max(policy);
    assert(priority != -1);
    printf("max_priority=%d\n", priority);

    priority = sched_get_priority_min(policy);
    assert(priority != -1);
    printf("min_priority=%d\n", priority);
}

static int get_thread_priority(pthread_attr_t *attr) //获取线程优先级
{
    sched_param param;
    int rs = pthread_attr_getschedparam(attr, &param);
    assert(rs == 0);
    printf("priority = %d", param.sched_priority);
    return param.sched_priority;
}
static void set_thread_policy(pthread_attr_t *attr, int policy)
{
    int rs = pthread_attr_setschedpolicy(attr, policy);
    assert(rs == 0);

    get_thread_policy(attr);
}

int main(int argc, char const *argv[])
{
    pthread_attr_t attr;
    struct sched_param sched;

    int rs = pthread_attr_init(&attr);
    assert(rs == 0);

    int plicy = get_thread_policy(&attr);
    printf("输出进程优先级\n");
    show_thread_priority(&attr, plicy);

    printf("输出FIFO优先级");
    show_thread_priority(&attr, SCHED_FIFO);

    printf("输出RR优先级\n");
    show_thread_priority(&attr, SCHED_RR);
    printf("输出当前线程优先级");

    int priority = get_thread_priority(&attr);
    printf("设置线程策略\n");
    printf("设置FIFO策略");

    set_thread_policy(&attr, SCHED_FIFO);
    printf("设置RR策略");
    set_thread_policy(&attr, SCHED_RR);

    printf("还原线程属性\n");
    set_thread_policy(&attr, plicy);

    rs = pthread_attr_destroy(&attr);
    assert(rs == 0);

    return 0;
}

```

**实战2**

```cpp
#include <pthread.h>
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>

void threadfunc1()
{
    sleep(1);
    int policy;
    struct sched_param praram;
    pthread_getschedparam(pthread_self(), &policy, &praram);

    if (policy == SCHED_OTHER)
        printf("SCHED_OTHER.\n");
    if (policy == SCHED_RR)
        ;
    printf("SCHED_RR 1.\n");
    if (policy == SCHED_FIFO)
        printf("SCHED_FIFO.\n");

    for (int i = 1; i <= 10; i++)
    {
        for (int j = 1; j < 4000000; j++)
        {
        }
        printf("Threadfunc1.\n");
    }
    printf("pthreadfunc1 EXIT.\n");
}

void threadfunc2()
{
    sleep(1);
    int policy;
    struct sched_param praram;
    pthread_getschedparam(pthread_self(), &policy, &praram);

    if (policy == SCHED_OTHER)
        printf("SCHED_OTHER.\n");
    if (policy == SCHED_RR)
        ;
    printf("SCHED_RR 1.\n");
    if (policy == SCHED_FIFO)
        printf("SCHED_FIFO.\n");

    for (int i = 1; i <= 10; i++)
    {
        for (int j = 1; j < 4000000; j++)
        {
        }
        printf("Threadfunc2.\n");
    }
    printf("pthreadfunc2 EXIT.\n");
}

void threadfunc3()
{
    sleep(1);
    int policy;
    struct sched_param praram;
    pthread_getschedparam(pthread_self(), &policy, &praram);

    if (policy == SCHED_OTHER)
        printf("SCHED_OTHER.\n");
    if (policy == SCHED_RR)
        ;
    printf("SCHED_RR 1.\n");
    if (policy == SCHED_FIFO)
        printf("SCHED_FIFO.\n");

    for (int i = 1; i <= 10; i++)
    {
        for (int j = 1; j < 4000000; j++)
        {
        }
        printf("Threadfunc3.\n");
    }
    printf("pthreadfunc3 EXIT.\n");
}

int main()
{
    int i;
    i = getuid();

    if (i == 0)
        printf("the current user is root.\n");
    else
        printf("the current user is not root.\n");

    pthread_t ppid1, ppid2, ppid3;
    struct sched_param param;

    pthread_attr_t attr1, attr2, attr3;

    pthread_attr_init(&attr2);
    pthread_attr_init(&attr1);
    pthread_attr_init(&attr3);

    param.sched_priority = 51;

    pthread_attr_setschedpolicy(&attr3, SCHED_RR);
    pthread_attr_setschedparam(&attr3, &param);
    pthread_attr_setinheritsched(&attr3, PTHREAD_EXPLICIT_SCHED);

    param.sched_priority = 22;
    pthread_attr_setschedpolicy(&attr2, SCHED_RR);
    pthread_attr_setschedparam(&attr2, &param);
    pthread_attr_setinheritsched(&attr2, PTHREAD_EXPLICIT_SCHED);

    pthread_create(&ppid3, &attr1, (void *)threadfunc3, NULL);
    pthread_create(&ppid2, &attr2, (void *)threadfunc2, NULL);
    pthread_create(&ppid1, &attr3, (void *)threadfunc1, NULL);

    pthread_join(ppid3, NULL);
    pthread_join(ppid2, NULL);
    pthread_join(ppid1, NULL);

    pthread_attr_destroy(&attr3);
    pthread_attr_destroy(&attr2);
    pthread_attr_destroy(&attr1);

    return 0;
}

```

## RCU机制

​	RCU英文全称为Read-Copy-Update，顾名思义就是 “读 - 拷贝-更新”，是内核中重要 的同步机制。Linux内核已有原子操作、读写信号量等等锁机制，为何会单独设计一个比较复杂的新机制？ 

**RCU原理**

​	RCU记录所有指向共享数据的指针的使用者，当要修改该共享数据时，首先创建一个 副本，在副本中修改。所有读访问线程都离开读临界区之后 ，指针指向新的修改后副本的指针,并且删除旧数据

1. 写者修改对象的过程:首先生成一个副本,让后更新副本,最后使用新的对象替换旧的对象,在写者执行复制更新的时候读者可以读数据
2. 写者删除对象:必须等到所有访问的读者访问结束,才能执行销毁操作,RCU技术的关键:**如何判断所有的读者已经结束访问**,等待所有读者访问时间结束的时间称为**宽限期**
3. RCU读者并不需要与写进行同步,读者和写者能够并发执行,RCU目标最大程序来减少读者的开销,因为也经常使用于读者性能要求高的场合

RCU优点:读者开销少,不需要锁,原子指令,内存屏障等,无死锁问题,无内存泄露风险

RCU缺点:写者同步开销比较大,写着之间需要互斥处理,同步机制复杂

![image-20210905142136565](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210905142136565.png)



## 内核内存布局

​	ARM64架构处理器采用48位物理寻址机制，最大可寻找256TB的物理地址空间。对于 目前应用完全足够，不需要扩展到64位的物理寻址。虚拟地址也同样最大支持48位寻址，所以 在处理器架构设计上，把虚拟地址空间划分为两个空间，每个空间最大支持256TB，linux内核 在大多数体系结构上都把两个地址划分为：用户空间和内核空间。 

用户空间：0x0000_0000_0000_0000至0x0000_ffff_ffff_ffff。 

内核空间：0xffff_0000_0000_0000至0xffff_ffff_ffff_ffff



[^1]: ![image-20210903182906481](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210903182906481.png)



