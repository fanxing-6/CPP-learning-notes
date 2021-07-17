# 循环接收/发送 更好的TCP接收/发送文件写法

```cpp
#include <cerrno>
#include <cstdio>
#include <WinSock2.h>

int MySocketRecv0(int sock, char* buf, int dateSize)
{
	int numsRecvSoFar = 0; int numsRemainingToRecv = dateSize; printf("enter MySocketRecv0\n");
	while (1)
	{
		int bytesRead = recv(sock, &buf[numsRecvSoFar], numsRemainingToRecv, 0);
		printf("###bytesRead = %d,numsRecvSoFar = %d, numsRemainingToRecv = %d\n", bytesRead, numsRecvSoFar,
		       numsRemainingToRecv);
		if (bytesRead == numsRemainingToRecv) { return 0; }
		else if (bytesRead > 0)
		{
			numsRecvSoFar += bytesRead;
			numsRemainingToRecv -= bytesRead;
			continue;
		}
		else if ((bytesRead < 0) && (errno == EAGAIN)) { continue; }
		else { return -1; }
	}
}

int MySocketSend0(int socketNum, unsigned char* data, unsigned dataSize)
{
	unsigned numBytesSentSoFar = 0;
	unsigned numBytesRemainingToSend = dataSize;
	while (1)
	{
		int bytesSend = send(socketNum, (char const*)(&data[numBytesSentSoFar]), numBytesRemainingToSend, 0/*flags*/);
		if (bytesSend == numBytesRemainingToSend) { return 0; }
		else if (bytesSend > 0)
		{
			numBytesSentSoFar += bytesSend;
			numBytesRemainingToSend -= bytesSend;
			continue;
		}
		else if ((bytesSend < 0) && (errno == 11)) { continue; }
		else { return -1; }
	}
}

```

