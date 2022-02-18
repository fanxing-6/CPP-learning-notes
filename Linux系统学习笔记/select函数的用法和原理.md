# select函数的用法和原理 

## Linux上的select函数

select函数用于检测一组`socket`中是否有事件就绪.这里的事件为以下三类:

1. 读事件就绪
   - 在`socket`内核中,接收缓冲区中的字节数大于或者等于低水位标记`SO_RCVLOWAT`,此时调用`rec`或`read`函数可以无阻塞的读取该文件描述符,并且返回值大于零
   - TCP连接的对端关闭连接,此时本端调用r`recv`或`read`函数对`socket`进行读操作,`recv`或`read`函数返回0
   - 在监听的`socket`上有新的连接请求
   - 在`socket`尚有未处理的错误
2. 写事件就绪
   - 在`socket`内核中,发送缓冲区中的可用字节数大于等于低水位标记时,可以无阻塞的写,并且返回值大于0
   - `socket`的写操作被关闭时,对一个写操作被关闭的`socket`进行写操作,会触发SIGPIPE信号
   - `socket`使用非阻塞`connect`连接成功或失败时
3. 异常事件就绪

`select()`如下:

```cpp
#include <sys/select.h>   
    int select(int maxfdp1, fd_set *readset, fd_set *writeset, fd_set *exceptset,struct timeval *timeout);
```

参数说明

| nfds:      | Linux上的`socket`也叫作fd,将这个参数的值设置为所有需要使用select函数检测事件的fd中的最大值加1即` nfds=max(fd1,fd2,...,fdn)+1` |
| ---------- | ------------------------------------------------------------ |
| readfds:   | 需要监听可读事件的fd集合                                     |
| writefds:  | 需要监听可写事件fd的集合                                     |
| exceptfds: | 需要监听异常事件的fd集合                                     |
| timeout:   | 超时时间,即在这个参数设定的时间内检测这些fd的事件,超过这个时间后,`select`函数立即返回,这是一个`timeval`结构体 |

其定义如下:

```cpp
struct timeval{      
        long tv_sec;   /*秒 */
        long tv_usec;  /*微秒 */   
    }
```

参数`readfds,writefds,exceptfds`的类型都是`fd_set`,这是一个结构体信息

定义如下

```cpp
//#define __FD_SETSIZE		1024
#define __NFDBITS	(8 * (int) sizeof (__fd_mask))
#define	__FD_ELT(d)	((d) / __NFDBITS)
#define	__FD_MASK(d)	((__fd_mask) (1UL << ((d) % __NFDBITS)))

/* fd_set for select and pselect.  */
typedef struct
  {
    /* XPG4.2 requires this member name.  Otherwise avoid the name
       from the global namespace.  */
#ifdef __USE_XOPEN
    //typedef long int __fd_mask;
    __fd_mask fds_bits[__FD_SETSIZE / __NFDBITS];
# define __FDS_BITS(set) ((set)->fds_bits)
#endif
  } fd_set;

/* 最大数量`fd_set'.  */
#define	FD_SETSIZE		__FD_SETSIZE
```

假设未定义`__USE_XOPEN`整理一年

```cpp
typedef struct
  {
//typedef long int __fd_mask;
    long int fds_bits[__FD_SETSIZE / __NFDBITS];
  } fd_set;
```

将一个fd添加到`fd_set`这个集合中时需要使用`FD_SET`宏,其定义如下:

```cpp
void FD_SET(fd, fdsetp)
```

实现如下:

```cpp
#define	FD_SET(fd, fdsetp)	__FD_SET (fd, fdsetp)
```

`__FD_SET (fd, fdsetp)`实现如下:

```cpp
/* We don't use `memset' because this would require a prototype and
   the array isn't too big.  */
# define __FD_ZERO(set)  \
  do {									      \
    unsigned int __i;							      \
    fd_set *__arr = (set);						      \
    for (__i = 0; __i < sizeof (fd_set) / sizeof (__fd_mask); ++__i)	      \
      __FDS_BITS (__arr)[__i] = 0;					      \
  } while (0)

#endif	/* GNU CC */

#define __FD_SET(d, set) \
  ((void) (__FDS_BITS (set)[__FD_ELT (d)] |= __FD_MASK (d)))
```

举个例子,假设现在fd的值为43,,那么在数组下表为0的元素中第43个bit被置为1

------

再Linux上,向fd_set集合中添加新的fd时,采用位图法确定位置;在windows中添加fd至fd_set的实现规则依次从数组第0个位置开始向后递增

------

也就是说,`FD_SET`宏本质上是在一个有1024个连续`bit`的数组的第`fd`位置置`1`.

同理,`FD_CLR`删除一个`fd`的原理,也就是将数组的第`fd`位置置为`0`

![image-20210706144042275](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210706144042275.png)





实例;

```cpp
#include <sys/types.h>
#include <sys/socket.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <iostream>
#include <cstring>
#include <sys/time.h>
#include <vector>
#include <cerrno>

//Customize the value representing invalid fd
#pragma clang diagnostic push
#pragma ide diagnostic ignored "EndlessLoop"
#define INVALID_FD -1
int main(int argc,char * argv[])
{
    //create a listen socket
    int listenfd = socket(AF_INET,SOCK_STREAM,0);
    if(listenfd == INVALID_FD)
    {
        printf("创建监听socket失败");
        return -1;
    }
    //init server addr
    sockaddr_in bindaddr{};
    bindaddr.sin_family = AF_INET;
    bindaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    bindaddr.sin_port= htons(3000);
    if(bind(listenfd,(struct sockaddr*) &bindaddr, sizeof(bindaddr)) == -1)
    {
        printf("绑定socket失败");
        close(listenfd);
        return -1;
    }
    //start listen
    if(listen(listenfd,SOMAXCONN) == -1)
    {
        printf("监听失败！");
        close(listenfd);
        return -1;
    }
    //Store the client's socket data
    std::vector<int> clientfds;
    int maxfd;
    while(true)
    {
        fd_set readset;
        FD_ZERO(&readset);
        FD_SET(listenfd,&readset);
        maxfd = listenfd;
        unsigned long clientfdslength = clientfds.size();
        for (int i = 0; i < clientfdslength; ++i)
        {
            if(clientfds[i] != INVALID_FD)
            {
                FD_SET(clientfds[i],&readset);
                if(maxfd<clientfds[i])
                    maxfd = clientfds[i];
            }
        }
        timeval tm{};
        tm.tv_sec = 1;
        tm.tv_usec =0;
        int  ret = select(maxfd+1,&readset, nullptr, nullptr,&tm);
        if(ret == -1)
        {
            if (errno != EINTR)
                break;
        }
        //time out
        else if (ret ==0 )
        {
            continue;
        }
        else
        {
            //event detected on a socket
            if (FD_ISSET(listenfd,&readset))
            {
                sockaddr_in clientaddr{};
                socklen_t  clientaddrlen = sizeof(clientaddr);
                //accept client connection
                int clientfd = accept(listenfd,(struct sockaddr *)&clientaddr,&clientaddrlen);
                if (clientfd == INVALID_FD)
                {
                    break;
                }
                std::cout<<"接受到客户端连接，fd："<<clientfd<<std::endl;
                clientfds.push_back(clientfd);
            }
            else
            {
                //Assume that the data length sent by the client is not greater than 63
                char recvbuf[64];
                unsigned long clientfdslength = clientfds.size();
                for (int i = 0; i < clientfdslength; ++i)
                {
                    if(clientfds[i] != INVALID_FD && FD_ISSET(clientfds[i],&readset))
                    {
                        memset(recvbuf,0, sizeof(recvbuf));
                        //accept data
                        int length = recv(clientfds[i],recvbuf,64,0);
                        //recv的返回值等于0,表示客户端关闭了连接
                        if (length <=0 )
                        {
                            //error
                            std::cout<<"error"<<clientfds[i]<<std::endl;
                            close(clientfds[i]);
                            clientfds[i] == INVALID_FD;
                            continue;
                        }
                        std::cout<<"clientfd: "<<clientfds[i]<<", recv data:"<<recvbuf<<std::endl;
                    }
                }
            }
        }
    }
    //close all client socket
    int clientfdslength = clientfds.size();
    for (int i = 0; i < clientfdslength; ++i)
    {
        if(clientfds[i] != INVALID_FD)
        {
            close(clientfds[i]);
        }
    }
    //close socket
    close(listenfd);
    return 0;
}
#pragma clang diagnostic pop
```

使用`nc -v 127.0.0.1 3000`来模拟客户端,打开三个终端

![image-20210706165026755](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210706165026755.png)

关于以上代码,需要注意以下几点:

1. `select`函数在调用前后可能会修改`readfds,writefds,exceptfds`所以想在下次调用`select`函数时服用这些`fd_set`变量需要重新清零,添加内容

   ```cpp
   for (int i = 0; i < clientfdslength; ++i)
           {
               if(clientfds[i] != INVALID_FD)
               {
                   FD_SET(clientfds[i],&readset);
                   if(maxfd<clientfds[i])
                       maxfd = clientfds[i];
               }
           }
   ```

   

2. `select`函数也会修改`timeval`结构体的值,如果想复用这些变量,需要重新设置

   ```cpp
   timeval tm{};
           tm.tv_sec = 1;
           tm.tv_usec =0;
   ```

3. 如果将`select`的`timeval`参数设置为`NULL`,则`select`函数会一直阻塞下去

## windows上的socket函数

在windows上,`select`函数结束后,不会修改`timeval`函数



