# windows网络编程

## TCP编程

### 服务端

这里我们有几点需要注意:

- 使用`WSAStartup`初始化网络库,即将与`socket`函数相关`dll`文件加载到进程地址空间中
- 退出时,使用`WSACleanup()`卸载相关`dll`文件
- 与**Linux**使用`close`函数关闭`socket`不同,**windows**需要使用`closesocket`函数关闭`socket`



`WSAStartup函数和WSACleanup`函数是线程相关的,任何一个线程都可以调用,对于`WSAStartup`函数,某一个线程调用后,其他线程不需要调用,也可以使用;

如果某个线程不在调用`socket`相关函数,调用`WSACleanup`函数,那么其他需要调用`socket`相关函数的线程将无法调用,所以我们应该在整个程序退时调用`WACleanup`函数



```cpp
#include <WinSock2.h>
#include <stdio.h>
#include <stdlib.h>
#pragma comment(lib, "ws2_32.lib")
int main()
{
	printf("Server\n");
	//1 初始化网络库 // 加载套接字库
	WORD wVersionRequested;
	WSADATA wsaData;
	int err;
	wVersionRequested = MAKEWORD(2, 2);
	// 1、初始化套接字库
	err = WSAStartup(wVersionRequested, &wsaData);
	if (err != 0)
	{
		printf("WSAStartup errorNum = %d\n", GetLastError());
		return err;
	}
	if (LOBYTE(wsaData.wVersion) != 2 || HIBYTE(wsaData.wVersion) != 2)
	{
		printf("LOBYTE errorNum = %d\n", GetLastError());
		WSACleanup();
		return -1;
	}
	// 2 安装电话机 // 新建套接字
	SOCKET sockSrv = socket(AF_INET, SOCK_STREAM, 0);
	if (INVALID_SOCKET == sockSrv)
	{
		printf("socket errorNum = %d\n", GetLastError());
		return -1;
	} //给变量配置电话号码 IP 任何 端口 6000
	SOCKADDR_IN addrSrv;
	addrSrv.sin_addr.S_un.S_addr = htonl(INADDR_ANY);
	addrSrv.sin_family = AF_INET;
	addrSrv.sin_port = htons(6000); // 3 分配电话号码 // 绑定套接字到本地 IP 地址，端口号 6000
	if (SOCKET_ERROR == bind(sockSrv, (SOCKADDR*)&addrSrv, sizeof(SOCKADDR)))
	{
		printf("bind errorNum = %d\n", GetLastError());
		return -1;
	} // 4、监听 listen
	if (SOCKET_ERROR == listen(sockSrv, 5))
	{
		printf("listen errorNum = %d\n", GetLastError());
		return -1;
	} // 5、拿起话筒，准备通话
	SOCKADDR_IN addrCli;
	int len = sizeof(SOCKADDR);
	while (TRUE)
	{
		//6、分配一台分机去服务
		SOCKET sockConn = accept(sockSrv, (SOCKADDR*)&addrCli, &len);
		char sendBuf[100] = {0};
		sprintf_s(sendBuf, 100, "Welcome %s to bingo!", inet_ntoa(addrCli.sin_addr));
		//发送数据
		int iLen = send(sockConn, sendBuf, strlen(sendBuf) + 1, 0);
		if (iLen < 0)
		{
			printf("send errorNum = %d\n", GetLastError());
			return -1;
		}
		char recvBuf[100] = {0}; //接收数据
		iLen = recv(sockConn, recvBuf, 100, 0);
		if (iLen < 0)
		{
			printf("recv errorNum = %d\n", GetLastError());
			return -1;
		} //打印接收的数据
		printf("recvBuf = %s\n", recvBuf);
		closesocket(sockConn);
	}
	//7 关闭总机
	closesocket(sockSrv);
	WSACleanup();
	system("pause");
	return 0;
}

```

### 客户端

```cpp
#include <iostream>
#include <WinSock2.h>
#include <stdio.h>
#include <stdlib.h>
#pragma comment(lib, "ws2_32.lib")
int main()
{
	printf("Client\n");
	//1 初始化网络库 // 加载套接字库
	WORD wVersionRequested;
	WSADATA wsaData;
	int err;
	wVersionRequested = MAKEWORD(2, 2);
	// 1、初始化套接字库
	err = WSAStartup(wVersionRequested, &wsaData);
	if (err != 0)
	{
		printf("WSAStartup errorNum = %d\n", GetLastError());
		return err;
	}
	if (LOBYTE(wsaData.wVersion) != 2 || HIBYTE(wsaData.wVersion) != 2)
	{
		printf("LOBYTE errorNum = %d\n", GetLastError());
		WSACleanup();
		return -1;
	}
	// 2 安装电话机 // 新建套接字
	SOCKET sockCli = socket(AF_INET, SOCK_STREAM, 0);
	if (INVALID_SOCKET == sockCli)
	{
		printf("socket errorNum = %d\n", GetLastError());
		return -1;
	}
	//配置要连接服务器
	SOCKADDR_IN addrSrv;
	addrSrv.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");
	addrSrv.sin_family = AF_INET;
	addrSrv.sin_port = htons(6000);
	//连接服务器

	if (SOCKET_ERROR == connect(sockCli, (SOCKADDR*)&addrSrv, sizeof(SOCKADDR)))
	{
		printf("connect errorNum = %d\n", GetLastError());
		return -1;
	}
	//收发数据
	char recvBuf[100] = { 0 };
	int iLen = recv(sockCli, recvBuf, 100, 0);
	std::cout << recvBuf << std::endl;
	char sendBuf[100] = "hello";
	iLen = send(sockCli, sendBuf, 100, 0);

	closesocket(sockCli);
	WSACleanup();
}
```

### 调试小技巧

可以使用`GetLastError()`获取最新错误码(其实不只错误码),通过visual studio中的工具查询含义

![image-20210710001034175](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210710001034175.png)

![image-20210710001051256](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210710001051256.png)

输入错误码,就可以获取错误信息啦

### vs F1快捷键 快速查阅msdn文档

选中函数,按F1键,就会自动打开默认浏览器,进入msdn文档搜索

