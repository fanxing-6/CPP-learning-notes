# Linux守护进程的创建(结合nginx框架)

先介绍几个相关函数:

`int dup2(arg1,arg2)`:参数一指向的内容赋给参数二,shi的参数二也能访问参数一所指向的内容,并返回新的描述符

`int fork()`创建子进程,返回值-1:创建失败  返回值0:子进程  返回其他:父进程

`setsid()`调用成功后，返回新的会话的ID，调用setsid函数的进程成为新的会话的领头进程，并与其父进程的会话组和进程组脱离

`unmask()`:umask可用来设定[权限掩码]。[权限掩码]是由3个八进制的数字所组成，将现有的存取权限减掉权限掩码后，即可产生建立文件时预设的权限,**咱们现在不用管,设置成0就可以了**

代码:

```cpp
#include <fcntl.h>
#include <iostream>
#include <signal.h>
#include <stdlib.h>
#include <sys/stat.h>
#include <unistd.h>
using std::cout;
using std::endl;
int ngx_doemon
{
    int fd;

    switch (fork())
    {
    case -1:
        return -1;
    case 0:
        break;
    default:
        exit(0);
    }

    if (setsid() == -1)
    {
        return -1;
    }

    umask(0);
    fd = open("dev/null", O_RDWR);
    if (fd == -1)
    {
        return -1;
    }
    if (dup2(fd, STDIN_FILENO) == -1)
    {
        return -1;
    }
    if (dup2(df, STDOUT_FILENO))
    {
        return -1;
    }
    if (fd > STDERR_FILENO)
    {
        if (close(fd) == -1)
            return -1;
    }
    return 1;
}
int main(int argc, char const *argv[])
{
    if (ngx_doemon != 1)
    {
        //创建守护进程失败,可以做失败后的处理
        return -1;
    }
    else
    {
        //创建守护进程成功,执行守护进程中要做的工作
        for (;;)
        {
            sleep(1);
        }
    }

    return 0;
}

```

