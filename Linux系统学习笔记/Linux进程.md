# Linux进程

## 僵尸进程

首先,我们创建一个僵尸进程:

```cpp
#include <iostream>
#include <unistd.h>
#include <sys/wait.h>
int main()
{
    pid_t  pid = fork();
    if (pid  > 0)
    {
        sleep(30);
        int status;
        waitpid(pid,&status,0);
    }
    else
    {
        printf("%s(%d):%s\n",__FILE__,__LINE__,__FUNCTION__ );
        exit(-1);
    }
}
```

运行一下,使用`ps -aux`查看进程状态

![image-20210607195437115](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210607195437115.png)

Z就说明子进程是僵尸进程,父进程处于休眠状态

## 信号处理

那我们如何终止子进程呢? 我们向**操作系统**求助,**让操作系统通知父进程子进程已经结束**

Linux系统信号机制最简单的接口是`signal`函数

```cpp
#include<signal.h>
void (*signal(int signo,void (*func))(int))(int)
```

具体参见《UNIX环境高级编程(第三版)》P256

一个简单示例:

```cpp
#include <iostream>
#include <unistd.h>
#include <sys/wait.h>
#include <csignal>
using std::cout;
void signal_func(int sig)
{
    switch (sig)
    {
        case SIGALRM:
        {
            cout << "tid:" << pthread_self() << "pid: " << getpid() << std::endl;
            alarm(2);
            break;
        }
        case SIGINT:
        {
            cout << "Ctrl + C press" << std::endl;
            exit(0);
            break;
        }

    }
}
int main()
{
    cout << "****MAIN THREAD:" << "tid:" << pthread_self() << "pid: " << getpid() << std::endl;
    signal(SIGALRM, signal_func);
    signal(SIGINT, signal_func);
    alarm(1);
    while (true)
    {
        cout << "****MAIN THREAD:" << "tid:" << pthread_self() << "pid: " << getpid() << std::endl;
        sleep(3);
    }

}
```

输出:

![image-20210607220411049](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210607220411049.png)

## 管道

单向管道

```cpp
#include <iostream>
#include <unistd.h>
#include <sys/wait.h>
#include <csignal>
#include <cstdlib>
#include <string>
#include <cstdio>
#include <unistd.h>
#include <csignal>
#include <sys/wait.h>
#include <arpa/inet.h>
#include <sys/socket.h>
using namespace std;
int main()
{
    int fds[2];
    char str[999] = "send by sub process!!\n";
    char buf[999] {};
    pipe(fds);
    pid_t pid = fork();
    if (pid == 0)
    {
        write(fds[1],str, sizeof(str));
    } else
    {
        read(fds[0],buf, sizeof(buf));
        cout<<buf<<endl;
        
    }

}
```

## FIFO(命名管道)

```cpp
int main()
{
    mkfifo("./a.fifo",0666);
    pid_t  pid =fork();
    if (pid == 0)
    {

        char buffer[999];
        int fd = open("./a.fifo",O_RDONLY);
        ssize_t len = read(fd,buffer, sizeof(buffer));
        cout<<__LINE__<<buffer<<endl;
        close(fd);
    }
    else
    {
        int fd = open("./a.fifo",O_WRONLY);
        write(fd,"hello boy!",11);
        close(fd);
    }

}
```

## 共享内存

IPC形式最快的,不需要内核的系统调用来复制信息



```cpp
#include <iostream>
#include <sys/wait.h>
#include <sys/ipc.h> //进程间通信
#include <sys/shm.h>
#include <sys/stat.h>
#include <sys/types.h>
#include <unistd.h>
#include <cstring>
typedef struct
{
  int signal = 1324;
  int id;
  char name[128];
  int age;
  bool sex;
}STUDENT,*PSTUDEND;

int main()
{
    pid_t pid =fork();
    if (pid>0)
    {
        int shm_id = shmget(ftok(".",1),sizeof(STUDENT),IPC_CREAT | 0666);
        if (shm_id == -1)
        {
            std::cout<<"fail to create!"<<std::endl;
            return 0;
        }
        auto pStu = (PSTUDEND) shmat(shm_id,nullptr,0);
        pStu->id = 666;
        strcpy(pStu->name,"seagvesrfvsejgehsgb");
        pStu->age =18;
        pStu->sex = true;
        pStu->signal =99;
        if (pStu->signal == 0)
        {
            shmdt(pStu);
            shmctl(shm_id, IPC_RMID, nullptr);
        }
    }
    else
    {
        usleep(50000000);//1000 means one millisecond(毫秒)
        int shm_id = shmget(ftok(".",1),sizeof(STUDENT),IPC_CREAT | 0666);
        if (shm_id == -1)
        {
            std::cout<<"fail to create!"<<std::endl;
            return 0;
        }
        auto pStu = (PSTUDEND) shmat(shm_id,nullptr,0);
        while(pStu->signal != 99)
        {
            usleep(1);
        }

        std::cout<<pStu->name<<pStu->age<<std::endl;
        pStu->signal = 0;
        shmdt(pStu);//Disconnect shared memory link
        shmctl(shm_id,IPC_RMID, nullptr);
    }
}

```

### 信号量

```cpp

```

