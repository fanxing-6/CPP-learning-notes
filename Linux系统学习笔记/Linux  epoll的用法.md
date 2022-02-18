# Linux  `epoll`的用法

### `epollfd_create`函数

```cpp
#include <sys/epoll.h>
	int epoll_create (int __size)
```

| 参数   | 含义                                                     |
| ------ | -------------------------------------------------------- |
| __size | 此参数从Linux 2.6.8后就不再使用了,但必须设置成大于零的值 |



| 返回值 | 含义          |
| ------ | ------------- |
| >0     | 可用的epollfd |
| -1     | 调用失败      |

### `epollfd_ctl`函数

有了`epollfd`,我们需要将要检测事件的fd绑定到这个`epollfd`上,或者修改或者移除,使用`epollfd_ctl`完成

```cpp
int epoll_ctl (int __epfd, int __op, int __fd,
		      struct epoll_event *__event) 
```



| 参数                 | 含义                                                         |
| -------------------- | ------------------------------------------------------------ |
| __epfd               | 上文中的epollfd                                              |
| __op                 | 操作类型1.(EPOLLFD_CTL_ADD)添加2.(EPOLLFD_CTL_MOD)修改3.(EPOLLFD_CTL_DEL)移除 |
| __fd                 | 需要被操作的描述符fd                                         |
| epoll_event *__event | 这是一个epollfd_event结构体地址,下文解释                     |



```cpp
struct epoll_event
{
  uint32_t events;	/* 需要检测fd事件标志 */
  epoll_data_t data;	/* 用户自定义的数据 */
}
```

其中的`epoll_data_t`

```cpp
typedef union epoll_data
{
  void *ptr;
  int fd;
  uint32_t u32;
  uint64_t u64;
} epoll_data_t;
```



| 返回值 | 含义 |
| ------ | ---- |
| 0      | 成功 |
| -1     | 失败 |

### `epollfd_wait`函数

```cpp
int epoll_wait (int __epfd, struct epoll_event *__events,
		       int __maxevents, int __timeout);
```

| 参数                  | 含义                                                         |
| --------------------- | ------------------------------------------------------------ |
| epoll_event *__events | 输出参数,在函数调用成功后,`events`中存放的是与就绪事件相关的`epoll_event`结构体数组 |
| __maxevents           | 上述数组中元素的个数                                         |

| 返回值 | 含义             |
| ------ | ---------------- |
| >0     | 有事件的fd的数量 |
| 0      | 超时             |
| -1     | 失败             |

示例:

```cpp
int main()
{
    epoll_event epoll_events[1024];
    int n = epoll_wait(epollfd,epoll_events,1024,1000);
    if(n<0) 
    {
        //被信号中断
        if(errno == EINTR)
        {
            //...
        }
    }
    else if(n ==0)
    {
        //...
    }
    for (size_t i = 0; i < n; ++i) 
    {
        if(epoll_events[i].events & EPOLLIN)
        {
            //处理可读事件
        } else if (epoll_events[i].events & EPOLLOUT)
        {
            //处理可写事件
        } else if (epoll_events[i].events & EPOLLERR)
        {
            //处理出错事件
        }
    }
}
```

### `poll`与`epoll_wait`函数的区别

![image-20210708153552180](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210708153552180.png)

## 边缘触发模式(ET) 和 水平触发模式 (LT)

水平触发模式:一个事件只要有,就会一直触发

边缘触发模式:一个事件从无到有才会触发

想不出好例子,摘抄一个吧

**水平触发**
儿子：妈妈，我收到了500元的压岁钱。
妈妈：嗯，省着点花。
儿子：妈妈，我今天花了200元买了个变形金刚。
妈妈：以后不要乱花钱。
儿子：妈妈，我今天买了好多好吃的，还剩下100元。
妈妈：用完了这些钱，我可不会再给你钱了。
儿子：妈妈，那100元我没花，我攒起来了
妈妈：这才是明智的做法！
儿子：妈妈，那100元我还没花，我还有钱的。
妈妈：嗯，继续保持。
儿子：妈妈，我还有100元钱。
妈妈：…

接下来的情形就是没完没了了：只要儿子一直有钱，他就一直会向他的妈妈汇报。LT模式下，只要内核缓冲区中还有未读数据，就会一直返回描述符的就绪状态，即不断地唤醒应用进程。在上面的例子中，儿子是缓冲区，钱是数据，妈妈则是应用进程了解儿子的压岁钱状况（读操作）。

**边缘触发**
儿子：妈妈，我收到了500元的压岁钱。
妈妈：嗯，省着点花。
（儿子使用压岁钱购买了变形金刚和零食。）
儿子：
妈妈：儿子你倒是说话啊？压岁钱呢？

这个就是ET模式，儿子只在第一次收到压岁钱时通知妈妈，接下来儿子怎么把压岁钱花掉并没有通知妈妈。即儿子从没钱变成有钱，需要通知妈妈，接下来钱变少了，则不会再通知妈妈了。在ET模式下， 缓冲区从不可读变成可读，会唤醒应用进程，缓冲区数据变少的情况，则不会再唤醒应用进程。



```cpp
/** 
 * 验证epoll的LT与ET模式的区别，epoll_server.cpp
 * zhangyl 2019.04.01
 */
#include<sys/types.h>
#include<sys/socket.h>
#include<arpa/inet.h>
#include<unistd.h>
#include<fcntl.h>
#include<sys/epoll.h>
#include<poll.h>
#include<iostream>
#include<string.h>
#include<vector>
#include<errno.h>
#include<iostream>

int main()
{
    //创建一个监听socket
    int listenfd = socket(AF_INET, SOCK_STREAM, 0);
    if (listenfd == -1)
    {
        std::cout << "create listen socket error" << std::endl;
        return -1;
    }

    //设置重用ip地址和端口号
    int on = 1;
    setsockopt(listenfd, SOL_SOCKET, SO_REUSEADDR, (char*)&on, sizeof(on));
    setsockopt(listenfd, SOL_SOCKET, SO_REUSEPORT, (char*)&on, sizeof(on));


    //将监听socker设置为非阻塞的
    int oldSocketFlag = fcntl(listenfd, F_GETFL, 0);
    int newSocketFlag = oldSocketFlag | O_NONBLOCK;
    if (fcntl(listenfd, F_SETFL, newSocketFlag) == -1)
    {
        close(listenfd);
        std::cout << "set listenfd to nonblock error" << std::endl;
        return -1;
    }

    //初始化服务器地址
    struct sockaddr_in bindaddr;
    bindaddr.sin_family = AF_INET;
    bindaddr.sin_addr.s_addr = htonl(INADDR_ANY);
    bindaddr.sin_port = htons(3000);

    if (bind(listenfd, (struct sockaddr*)&bindaddr, sizeof(bindaddr)) == -1)
    {
        std::cout << "bind listen socker error." << std::endl;
        close(listenfd);
        return -1;
    }

    //启动监听
    if (listen(listenfd, SOMAXCONN) == -1)
    {
        std::cout << "listen error." << std::endl;
        close(listenfd);
        return -1;
    }


    //创建epollfd
    int epollfd = epoll_create(1);
    if (epollfd == -1)
    {
        std::cout << "create epollfd error." << std::endl;
        close(listenfd);
        return -1;
    }

    epoll_event listen_fd_event;
    listen_fd_event.data.fd = listenfd;
    listen_fd_event.events = EPOLLIN;
    //取消注释掉这一行，则使用ET模式
    //listen_fd_event.events |= EPOLLET;

    //将监听sokcet绑定到epollfd上去
    if (epoll_ctl(epollfd, EPOLL_CTL_ADD, listenfd, &listen_fd_event) == -1)
    {
        std::cout << "epoll_ctl error" << std::endl;
        close(listenfd);
        return -1;
    }

    int n;
    while (true)
    {
        epoll_event epoll_events[1024];
        n = epoll_wait(epollfd, epoll_events, 1024, 1000);
        if (n < 0)
        {
            //被信号中断
            if (errno == EINTR) 
                continue;

            //出错,退出
            break;
        }
        else if (n == 0)
        {
            //超时,继续
            continue;
        }
		
        for (size_t i = 0; i < n; ++i)
        {
            //事件可读
            if (epoll_events[i].events & EPOLLIN)
            {
                if (epoll_events[i].data.fd == listenfd)
                {
                    //侦听socket,接受新连接
                    struct sockaddr_in clientaddr;
                    socklen_t clientaddrlen = sizeof(clientaddr);
                    int clientfd = accept(listenfd, (struct sockaddr*)&clientaddr, &clientaddrlen);
                    if (clientfd != -1)
                    {
                        int oldSocketFlag = fcntl(clientfd, F_GETFL, 0);
                        int newSocketFlag = oldSocketFlag | O_NONBLOCK;
                        if (fcntl(clientfd, F_SETFL, newSocketFlag) == -1)
                        {
                            close(clientfd);
                            std::cout << "set clientfd to nonblocking error." << std::endl;
                        }
                        else
                        {
                            epoll_event client_fd_event;
                            client_fd_event.data.fd = clientfd;
                            client_fd_event.events = EPOLLIN;
                            //取消注释这一行，则使用ET模式
                            //client_fd_event.events |= EPOLLET; 
                            if (epoll_ctl(epollfd, EPOLL_CTL_ADD, clientfd, &client_fd_event) != -1)
                            {
                                std::cout << "new client accepted,clientfd: " << clientfd << std::endl;
                            }
                            else
                            {
                                std::cout << "add client fd to epollfd error" << std::endl;
                                close(clientfd);
                            }
                        }
                    }
                }
                else
                {
                    std::cout << "client fd: " << epoll_events[i].data.fd << " recv data." << std::endl;
                    //普通clientfd
                    char ch;
                    //每次只收一个字节
                    int m = recv(epoll_events[i].data.fd, &ch, 1, 0);
                    if (m == 0)
                    {
                        //对端关闭了连接，从epollfd上移除clientfd
                        if (epoll_ctl(epollfd, EPOLL_CTL_DEL, epoll_events[i].data.fd, NULL) != -1)
                        {
                            std::cout << "client disconnected,clientfd:" << epoll_events[i].data.fd << std::endl;
                        }
                        close(epoll_events[i].data.fd);
                    }
                    else if (m < 0)
                    {
                        //出错
                        if (errno != EWOULDBLOCK && errno != EINTR)
                        {
                            if (epoll_ctl(epollfd, EPOLL_CTL_DEL, epoll_events[i].data.fd, NULL) != -1)
                            {
                                std::cout << "client disconnected,clientfd:" << epoll_events[i].data.fd << std::endl;
                            }
                            close(epoll_events[i].data.fd);
                        }
                    }
                    else
                    {
                        //正常收到数据
                        std::cout << "recv from client:" << epoll_events[i].data.fd << ", " << ch << std::endl;
                    }
                }
            }
            else if (epoll_events[i].events & POLLERR)
            {
                // TODO 暂不处理
            }
        }
    }

    close(listenfd);
    return 0;
}

```



![image-20210708163455964](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210708163455964.png)

现在采用的是一个水平模式,只要有数据可读,就会触发事件

现在采用边缘触发模式

![image-20210708163804287](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210708163804287.png)



采用边缘触发模式,只有有新数据到来才会触发,所以就有了上面的现象













