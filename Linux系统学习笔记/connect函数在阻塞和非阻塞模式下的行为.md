# `connect`函数在阻塞和非阻塞模式下的行为

当`socket`使用阻塞模式时,`connect`函数会阻塞到有明确结果才会返回,如果网络环境较差,可能要等一会,影响体验,

为了解决这个问题,我们使用**异步`connect`技术**

1. 创建`socket`,将`socket`设置为非阻塞模式

2. 调用`connect`函数,此时无论`connect`函数是否连接成功,都会立即返回,如果返回-1,不一定表示连接出错,如果此时错误码为`EINPROGRESS`表示正在尝试连接

3. 调用`select`函数,在指定时间内判断该`socket`是否可写,可写说明连接成功,反之,连接失败

   上述流程代码

   ```cpp
   #include <sys/types.h> 
   #include <sys/socket.h>
   #include <arpa/inet.h>
   #include <unistd.h>
   #include <iostream>
   #include <string.h>
   #include <stdio.h>
   #include <fcntl.h>
   #include <errno.h>
   
   #define SERVER_ADDRESS  "127.0.0.1"
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
   	
   	//将clientfd设置成非阻塞模式
   	int oldSocketFlag = fcntl(clientfd, F_GETFL, 0);
   	int newSocketFlag = oldSocketFlag | O_NONBLOCK;
   	if (fcntl(clientfd, F_SETFL,  newSocketFlag) == -1)
   	{
   		close(clientfd);
   		std::cout << "set socket to nonblock error." << std::endl;
   		return -1;
   	}
   	
   	//2.连接服务器
   	struct sockaddr_in serveraddr;
   	serveraddr.sin_family = AF_INET;
   	serveraddr.sin_addr.s_addr = inet_addr(SERVER_ADDRESS);
   	serveraddr.sin_port = htons(SERVER_PORT);
   	for (;;)
   	{
   		int ret = connect(clientfd, (struct sockaddr *)&serveraddr, sizeof(serveraddr));
   		if (ret == 0)
   		{
   			std::cout << "connect to server successfully." << std::endl;
   			close(clientfd);
   			return 0;
   		} 
   		else if (ret == -1) 
   		{
   			if (errno == EINTR)
   			{
   				//connect 动作被信号中断，重试connect
   				std::cout << "connecting interruptted by signal, try again." << std::endl;
   				continue;
   			} 
   			else if (errno == EINPROGRESS)
   			{
   				//连接正在尝试中
   				break;
   			} 
   			else
   			{
   				//真的出错了，
   				close(clientfd);
   				return -1;
   			}
   		}
   	}
   	
   	fd_set writeset;
   	FD_ZERO(&writeset);
   	FD_SET(clientfd, &writeset);
   	struct timeval tv;
   	tv.tv_sec = 3;  
   	tv.tv_usec = 0;
   	//3.调用select函数判断socket是否可写
   	if (select(clientfd + 1, NULL, &writeset, NULL, &tv) == 1)
   	{
   		std::cout << "[select] connect to server successfully." << std::endl;
   	} 
   	else 
   	{
   		std::cout << "[select] connect to server error." << std::endl;
   	}
   
   	close(clientfd);
   	
   	return 0;
   }
   ```

   首先先用`nc`命令启动一个服务端程序并执行

   ```shell
   nc -v -l -n 0.0.0.0 3000
   ```

   然后运行程序,我用的clion

   ![image-20210706233125990](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210706233125990.png)

   把服务端关掉,在重新启动客户端,一看结果,还是

   ![](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210706233125990.png)

   为什么连接不上也会输出同样的结果?原因如下:

   - 在Windows上,一个`socket`没有建立连接之前,我们用`select`检测是否可写,是可以得到正确结果的,即不可写;连接成功后在检测,就会变为可写
   
   - 在Linux上一个`socket`没有建立连接之前,用`select`函数检测是否可写,我们也会得到可写的结果,**所以,在Linux上,我们不仅要用`select`检测`socket`是否可写还要用`getsocketopt`检测`socket`此时是否出错
   
     ```cpp
     #include <sys/types.h>
     #include <sys/socket.h>
     #include <arpa/inet.h>
     #include <unistd.h>
     #include <iostream>
     #include <string.h>
     #include <stdio.h>
     #include <fcntl.h>
     #include <errno.h>
     
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
     
         //将clientfd设置成非阻塞模式
         int oldSocketFlag = fcntl(clientfd, F_GETFL, 0);
         int newSocketFlag = oldSocketFlag | O_NONBLOCK;
         if (fcntl(clientfd, F_SETFL,  newSocketFlag) == -1)
         {
             close(clientfd);
             std::cout << "set socket to nonblock error." << std::endl;
             return -1;
         }
     
         //2.连接服务器
         struct sockaddr_in serveraddr;
         serveraddr.sin_family = AF_INET;
         serveraddr.sin_addr.s_addr = inet_addr(SERVER_ADDRESS);
         serveraddr.sin_port = htons(SERVER_PORT);
         for (;;)
         {
             int ret = connect(clientfd, (struct sockaddr *)&serveraddr, sizeof(serveraddr));
             if (ret == 0)
             {
                 std::cout << "connect to server successfully." << std::endl;
                 close(clientfd);
                 return 0;
             }
             else if (ret == -1)
             {
                 if (errno == EINTR)
                 {
                     //connect 动作被信号中断，重试connect
                     std::cout << "connecting interruptted by signal, try again." << std::endl;
                     continue;
                 }
                 else if (errno == EINPROGRESS)
                 {
                     //连接正在尝试中
                     break;
                 }
                 else
                 {
                     //真的出错了，
                     close(clientfd);
                     return -1;
                 }
             }
         }
     
         fd_set writeset;
         FD_ZERO(&writeset);
         FD_SET(clientfd, &writeset);
         struct timeval tv;
         tv.tv_sec = 3;
         tv.tv_usec = 0;
         //3.调用select函数判断socket是否可写
         if (select(clientfd + 1, NULL, &writeset, NULL, &tv) != 1)
         {
             std::cout << "[select] connect to server error." << std::endl;
             close(clientfd);
             return -1;
         }
     
         int err;
         socklen_t len = static_cast<socklen_t>(sizeof err);
         //4.调用getsockopt检测此时socket是否出错
         if (::getsockopt(clientfd, SOL_SOCKET, SO_ERROR, &err, &len) < 0)
         {
             close(clientfd);
             return -1;
         }
     
         if (err == 0)
             std::cout << "connect to server successfully." << std::endl;
         else
             std::cout << "connect to server error." << std::endl;
     
         close(clientfd);
     
         return 0;
     }
     ```
   
     
   
     
   
     
   
     
   
     