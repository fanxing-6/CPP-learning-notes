# 第一个 Win32 窗口程序

```cpp
程序骨架 int WinMain(){ 
    // 设计窗口外观及交互响应，注册，申请专利
    RegisterClass(...) ;
        // 生产窗口 
    CreateWindow(...); 
    // 展示窗口 
    ShowWindow(...); 
    // 粉刷窗口 
    UpdateWindow(...);
    // 进入消息循环 
    while (GetMessage(...));
    { // 消息转换 
        TranslateMessage(...); 
        // 消息分发 
        DispatchMessage(...); 
    }
```

可以直接运行

```cpp
#include <stdio.h>
#include <windows.h>

//设计一个窗口类,填入参数
//注册窗口类
//创建窗口
//显示以及更新窗口
//循环等待消息
LPCTSTR clsName = "My";
LPCTSTR msgName = "欢迎学习";

//回调函数
LRESULT CALLBACK MyWinProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam);

INT WINAPI WinMain(HINSTANCE hInstance, HINSTANCE hPrevInstance,PSTR lpCmdLine, INT nCmdShow)
{
// 	typedef struct tagWNDCLASSA {
// 		UINT      style;
// 		WNDPROC   lpfnWndProc;
// 		int       cbClsExtra;
// 		int       cbWndExtra;
// 		HINSTANCE hInstance;
// 		HICON     hIcon;
// 		HCURSOR   hCursor;
// 		HBRUSH    hbrBackground;
// 		LPCSTR    lpszMenuName;
// 		LPCSTR    lpszClassName;
// 	} WNDCLASSA, * PWNDCLASSA, * NPWNDCLASSA, * LPWNDCLASSA;
	//定义一个窗口对象
    WNDCLASS wndcls{};
	wndcls.cbClsExtra = NULL;
	wndcls.cbWndExtra = NULL;
	wndcls.hbrBackground = (HBRUSH)GetStockObject(WHITE_BRUSH);
	wndcls.hCursor = LoadCursor(NULL, IDC_ARROW);
	wndcls.hIcon = LoadIcon(NULL, IDI_APPLICATION);
	wndcls.hInstance = hInstance;
	//定义交互响应
	wndcls.lpfnWndProc = MyWinProc;
	//定义窗口代号
	wndcls.lpszClassName = clsName;
	wndcls.lpszMenuName = NULL;
	wndcls.style = CS_HREDRAW | CS_VREDRAW;

	//注册窗口类
	RegisterClass(&wndcls);
	//创建窗口
	HWND hwnd;
	hwnd = CreateWindow(clsName, msgName, WS_OVERLAPPEDWINDOW, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, CW_USEDEFAULT, NULL, NULL, hInstance, NULL);


	//显示和刷新窗口
	ShowWindow(hwnd, SW_SHOWNORMAL);
	UpdateWindow(hwnd);

	//消息循环
	MSG msg;
	while (GetMessage(&msg,NULL,NULL,NULL))
	{
		TranslateMessage(&msg);
		DispatchMessage(&msg);
	}
	return msg.wParam;
	
}
LRESULT __stdcall MyWinProc(HWND hwnd, UINT uMsg, WPARAM wParam, LPARAM lParam)
{
	//uMsg消息类型
	int ret;
	HDC hdc;
	switch (uMsg)
	{
	case WM_CHAR:
		char szChar[20];
		sprintf_s(szChar, "您刚才按下了:%c", wParam);
		MessageBox(hwnd, szChar, "char", NULL);
		break;
	case WM_LBUTTONDOWN:
		MessageBox(hwnd,"检测鼠标左键按下","msg",NULL);
		break;
	case WM_PAINT:
		PAINTSTRUCT ps;
		hdc = BeginPaint(hwnd, &ps);
		TextOut(hdc, 0, 0, "www.baidu.com", strlen("www.baidu.com"));
		EndPaint(hwnd, &ps);
		MessageBox(hwnd, "重新绘制", "msg", NULL);
		break;
	 case WM_CLOSE:
		 ret = MessageBox(hwnd, "是否真要退出!", "msg", MB_YESNO);
		if(ret == IDYES)
		{
			DestroyWindow(hwnd);
		}
	 break;
	case WM_DESTROY:
		PostQuitMessage(0);
		break;
	default:
		return DefWindowProc(hwnd,uMsg,wParam,lParam);
	}
	return 0;
}

```

