# Windows编程 UDP客户端 UDP服务端

服务端

```cpp
#include <WinSock2.h>
#include <iostream>
#pragma comment(lib, "ws2_32.lib")
int main()
{
	//初始化套接字
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
	//创建套接字

	SOCKET sockSrv = socket(AF_INET, SOCK_DGRAM, 0/*默认参数*/);
	if(INVALID_SOCKET == sockSrv)
	{
		printf("socket error= % d\n", GetLastError());
		return -1;
	}

	//分配地址协议族端口
	SOCKADDR_IN addrSer{};
	addrSer.sin_family = AF_INET;
	addrSer.sin_addr.S_un.S_addr = htonl(INADDR_ANY);//host to net long
	addrSer.sin_port = htons(6000);//host to net short
	
	if (SOCKET_ERROR == bind(sockSrv, (SOCKADDR*)&addrSer, sizeof(SOCKADDR)))
	{
		printf("bind error= % d\n", GetLastError());
		return -1;
	}
	//等待接收数据
	SOCKADDR_IN addrCli;//客户端地址族
	int len = sizeof(SOCKADDR_IN);
	char recvBuf[100] = { 0 };
	char sendBuf[100] = { 0 };

	while (true)
	{
		recvfrom(sockSrv, recvBuf, 100, 0, (SOCKADDR*)&addrCli, &len);
		std::cout <<recvBuf<<std::endl;

		sprintf_s(sendBuf, 100, "返回:%s", recvBuf);
		sendto(sockSrv, sendBuf, strlen(sendBuf) + 1, 0, (SOCKADDR*)&addrCli, len);
		
	}
	//关闭套接字
	closesocket(sockSrv);
	WSACleanup();
}
```

客户端

```cpp
#include <WinSock2.h>
#include <iostream>
#pragma comment(lib, "ws2_32.lib")
int main()
{

	//初始化套接字
	printf("UDP_Client\n");
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
	//创建套接字
	SOCKET sockCli = socket(AF_INET, SOCK_DGRAM, 0/*默认参数*/);
	if (INVALID_SOCKET == sockCli)
	{
		printf("socket error= % d\n", GetLastError());
		return -1;
	}
	//分配地址协议族端口
	SOCKADDR_IN addrSer{};
	addrSer.sin_family = AF_INET;
	addrSer.sin_addr.S_un.S_addr = inet_addr("127.0.0.1");//host to net long
	addrSer.sin_port = htons(6000);//host to net short

	int len = sizeof(SOCKADDR_IN);
	char sendBuf[100] = "hello server\n";
	char recvBuf[100] = { 0 };

	//发送数据
	sendto(sockCli, sendBuf, strlen(sendBuf) + 1, 0, (SOCKADDR*)&addrSer, len);
	//接收数据
	recvfrom(sockCli, recvBuf, 100, 0, (SOCKADDR*)&addrSer, &len);
	std::cout << recvBuf<<std::endl;
	closesocket(sockCli);
	WSACleanup();
}
```

