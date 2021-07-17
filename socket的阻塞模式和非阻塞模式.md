# `socket`的阻塞模式和非阻塞模式

无论是Windows还是Linux,默认创建`socket`都是阻塞模式的

在Linux中,可以再创建`socket`是直接将它设置为非阻塞模式

```cpp
int socket (int __domain, int __type, int __protocol)
```

将`__type`增加`SOCK_NOBLOCK`

不仅如此,在Linux上直接利用`accept`函数返回的代表与客户端通信的`socket`也提供了一个拓展函数`accept4`,直接将`accept4`返回的`socket`设置为非阻塞的

## `send`和`recv`函数在阻塞和非阻塞模式下的表现

`send`和`recv`函数并不是直接向网络上发送数据和接收数据

`send`函数是将应用层发送缓冲区的数据拷贝到内核缓冲区中

`recv`函数是将内核缓冲区的数据拷贝到应用缓冲区

可以用下面这张图来描述:

![image-20210706212341938](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210706212341938.png)

通过上图我们可以知道,不同的程序进行网络通信时,发送的一方会将内核缓冲区的数据通过网络传输给接收方的内核缓冲区。

在应用程序A与应用程序B建立TCP连接后,假设A不断调用`send`函数,会将数据不断拷贝到对应的内核缓冲区,如果应用程序不调用`recv`函数,那么在应用程序B的内核缓冲区被填满后,A的缓冲区也随后被填满,此时如果A继续调用`send`函数会有什么后果呢?

- 当`socket`处于阻塞模式时,继续调用`send/recv`函数,程序会阻塞在`send/recv`调用处
- 当`socket`处于非阻塞模式时,继续调用`send/recv`函数,会返回错误码

1. `socket`阻塞模式下`send`函数的表现

   **代码来自《C++服务器开发精髓》**

   服务端代码:

   ```cpp
   #include <sys/types.h> 
   #include <sys/socket.h>
   #include <arpa/inet.h>
   #include <unistd.h>
   #include <iostream>
   #include <string.h>
   
   int main(int argc, char* argv[])
   {
       //1.创建一个侦听socket
       int listenfd = socket(AF_INET, SOCK_STREAM, 0);
       if (listenfd == -1)
       {
           std::cout << "create listen socket error." << std::endl;
           return -1;
       }
   
       //2.初始化服务器地址
       struct sockaddr_in bindaddr;
       bindaddr.sin_family = AF_INET;
       bindaddr.sin_addr.s_addr = htonl(INADDR_ANY);
       bindaddr.sin_port = htons(3000);
       if (bind(listenfd, (struct sockaddr *)&bindaddr, sizeof(bindaddr)) == -1)
       {
           std::cout << "bind listen socket error." << std::endl;
   		close(listenfd);
           return -1;
       }
   
   	//3.启动侦听
       if (listen(listenfd, SOMAXCONN) == -1)
       {
           std::cout << "listen error." << std::endl;
   		close(listenfd);
           return -1;
       }
   
       while (true)
       {
           struct sockaddr_in clientaddr;
           socklen_t clientaddrlen = sizeof(clientaddr);
   		//4. 接受客户端连接
           int clientfd = accept(listenfd, (struct sockaddr *)&clientaddr, &clientaddrlen);
           if (clientfd != -1)
           {         	
   			//只接受连接，不调用recv收取任何数据
   			std:: cout << "accept a client connection." << std::endl;
           }
       }
   	
   	//7.关闭侦听socket
   	close(listenfd);
   
       return 0;
   }
   
   ```

   客户端代码:

   ```cpp
   /**
    * 验证阻塞模式下send函数的行为，client端
    * zhangyl 2018.12.17
    */
   #include <sys/types.h> 
   #include <sys/socket.h>
   #include <arpa/inet.h>
   #include <unistd.h>
   #include <iostream>
   #include <string.h>
   
   #define SERVER_ADDRESS "127.0.0.1"
   #define SERVER_PORT     3000
   #define SEND_DATA       "helloworld"
   
   int main(int argc, char* argv[])
   {
       //1.创建一个socket
       int clientfd = socket(AF_INET, SOCK_STREAM, 0);
       if (clientfd == -1)
       {
           std::cout << "create client socket error." << std::endl;
           return -1;
       }
   
       //2.连接服务器
       struct sockaddr_in serveraddr;
       serveraddr.sin_family = AF_INET;
       serveraddr.sin_addr.s_addr = inet_addr(SERVER_ADDRESS);
       serveraddr.sin_port = htons(SERVER_PORT);
       if (connect(clientfd, (struct sockaddr *)&serveraddr, sizeof(serveraddr)) == -1)
       {
           std::cout << "connect socket error." << std::endl;
           close(clientfd);
           return -1;
       }
   
       //3. 不断向服务器发送数据，或者出错退出
       int count = 0;
       while (true)
       {
           int ret = send(clientfd, SEND_DATA, strlen(SEND_DATA), 0);
           if (ret != strlen(SEND_DATA))
           {
               std::cout << "send data error." << std::endl;
               break;
           } 
           else
           {
               count ++;
               std::cout << "send data successfully, count = " << count << std::endl;
           }
       }
   
       //5. 关闭socket
       close(clientfd);
   
       return 0;
   }
   
   ```

   先启动server在启动client,客户端会不断向服务端发送helloworld,每次发送成功后会打印计数器,运行一顿时间后,停止打印,计数器不再增加

   

![20210706214738](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/20210706214738.jpg)

当程序不再有输出,说明阻塞在某个函数gdb看一看

```shell
(gdb) bt
#0  0x00007ffff7d03690 in __libc_send (fd=3, buf=0x555555556045, len=10, 
    flags=0) at ../sysdeps/unix/sysv/linux/send.c:28
#1  0x00005555555553bb in main (argc=1, argv=0x7fffffffdf28) at client.cpp:42
(gdb) 
```

果然是`send`函数



上面这个例子证明了如果一端一直发送数据,另一端不接收数据,内核缓冲区很快就会被填满,发生阻塞. 其实这里所说的内核缓冲区就是**TCP窗口**

我们现在利用`tcpdump`工具查看一下这种情况下TCP窗口的大小

```cpp
22:01:57.543364 IP 127.0.0.1.53382 > 127.0.0.1.3000: Flags [S], seq 1832090129, win 65495, options [mss 65495,sackOK,TS val 451488646 ecr 0,nop,wscale 7], length 0
22:01:57.543379 IP 127.0.0.1.3000 > 127.0.0.1.53382: Flags [S.], seq 1797517498, ack 1832090130, win 65483, options [mss 65495,sackOK,TS val 451488646 ecr 451488646,nop,wscale 7], length 0
22:01:57.543386 IP 127.0.0.1.53382 > 127.0.0.1.3000: Flags [.], ack 1797517499, win 512, options [nop,nop,TS val 451488646 ecr 451488646], length 0
...
22:02:11.342670 IP 127.0.0.1.3000 > 127.0.0.1.53382: Flags [.], ack 1832177322, win 0, options [nop,nop,TS val 451502445 ecr 451488936], length 0

```

win就是TCP窗口的大小可以看出,逐渐减小最后变为零



2. `socket`非阻塞模式下`send`函数的表现

   就是返回一个错误码,不阻塞了,略...

3. `socket`阻塞模式下`recv`函数的表现

   阻塞了就...

4. `socket`非阻塞模式下`recv`函数的表现

   `recv`在没有数据可读的情况下,会立即返回,返回值为-1



### 非阻塞模式下`send`和`recv`函数返回值总结

| 返回值n | 含义                                                         |
| ------- | ------------------------------------------------------------ |
| 大于0   | 成功发送或接受n字节                                          |
| 等于零  | 对方关闭连接                                                 |
| 小于零  | 出错,被信号中断,TCP窗口太小导致数据发送不出去,或者当前网卡缓冲区已经无数据可以接受 |

详细介绍:

1. 返回值大于0

   当`send`和`recv`函数返回值大于0时,表示发送或者接收多少字节.需要注意的是,在这种情况下,**判断`send`返回值是否等于要发送的字节数,而不是简单地判断返回值是否大于零

   

   ```cpp
   int n = send(socketfd,buf,buf_length,0);
       if (n>0)
       {
           printf("send successfully");
       }
   ```

   很多新手就会写出以上的代码(比如我...)虽然返回值大于零,但由于对端TCP窗口已满,搜易我们所期望发送的字节,并没有全部被对方接收,所以`n`的大小在区间(0,buf_length)内

   **解决办法:**

   - 在返回值等于`buf_length`时才认为正确
   - 在一个循环中调用`send`函数,如果一次性发送不完,记录偏移量,接着从偏移量处发送

2. 返回值等于0

   - 对端关闭了连接
   - 特殊情况:`send`函数主动发送了0字节

3. 小于零

   出错啦呗

