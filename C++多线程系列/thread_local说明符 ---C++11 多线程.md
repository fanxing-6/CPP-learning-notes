# `thread_local`说明符 ---C++11 多线程

`thread_local`说明符可以声明线程生命周期的对象,它能与`static`或`extern`结合,分别指定内部连接或者外部链接

额外的`static`说明符并不影响线程局部存储

了解了线程局部存储的意义后,我们发现**线程局部存储只是定义了对象的生命周期,并没有对访问进行限制**,所以我们在其他线程依旧可以访问另一线程的由`thread_local`关键字标记的对象,并修改.

下面写个代码演示一下:

```cpp
#include <iostream>
#include <string>
#include <thread>
#include <mutex>
#include <format>
using namespace std;

struct WarpInt;
WarpInt* warp_int_ptr = nullptr; // 为了简单就采用这种不严谨的方式了...


struct WarpInt
{
	int i;

	WarpInt() : i(0)
	{
		cout << "i:" << i << endl;
		cout << "thread id:" << this_thread::get_id << ": 构造WarpInt" << endl;
	};

	~WarpInt()
	{
		cout << "i:" << i << endl;
		cout << "thread id:" << this_thread::get_id << ": 析构WarpInt" << endl;
	}

	void addi()
	{
		i++;
	}
};

void func1()
{
	thread_local WarpInt Int1;
	warp_int_ptr = &Int1;
	this_thread::sleep_for(5s); // 暂停5秒
}

void func2()
{
	this_thread::sleep_for(1s); // 暂停一秒等待t1将WarpInt构造完毕并获取地址
	WarpInt* warp_int2 = warp_int_ptr;
	warp_int2->addi();
}


int main(int argc, char* argv[])
{
	thread t1(func1);
	thread t2(func2);
	t1.join();
	t2.join();
}
```

最后`i`编程`1`了说明`t2`成功获取到了`t1`的`WarpInt`并修改

```cpp
i:0
thread id:00007FF7791213CA: 构造WarpInt
i:1
thread id:00007FF7791213CA: 析构WarpInt
```



**同一个线程中,一个线程局部变量只会初始化一次,即使被某个函数多次调用,也只会销毁一次,一般在线程结束时销毁**

例子如下:

```cpp
#include <iostream>
#include <string>
#include <thread>
#include <mutex>
#include <format>
using namespace std;

mutex g_out_lock;

struct RefCount
{
	int i;
	string func;

	RefCount(const char* f): i(0), func(f)
	{
		lock_guard<mutex> lock(g_out_lock);
		std::cout << std::this_thread::get_id()
			<< "|" << func
			<< " : ctor i(" << i << ")" << std::endl;
	}

	~RefCount()
	{
		std::lock_guard<mutex> lock(g_out_lock);
		std::cout << std::this_thread::get_id()
			<< "|" << func
			<< " : dtor i(" << i << ")" << std::endl;

	}

	void inc()
	{
		std::lock_guard<mutex> lock(g_out_lock);
		std::cout << std::this_thread::get_id()
			<< "|" << func
			<< " : ref count add 1 to  i(" << i << ")" << std::endl;
		i++;
	}
};

RefCount* lp_ptr = nullptr;

void foo(const char* f)
{
	string func(f);
	thread_local RefCount tv(func.append("#foo").c_str());
	tv.inc();
}

void bar(const char* f)
{
	string func(f);
	thread_local RefCount tv(func.append("#bar").c_str());
	tv.inc();
}

void threadfunc1()
{
	const char* func = "threadfunc1";
	foo(func);
	foo(func);
	foo(func);
}

void threadfunc2()
{
	const char* func = "threadfunc2";
	foo(func);
	foo(func);
	foo(func);
}

void threadfunc3()
{
	const char* func = "threadfunc3";
	foo(func);
	bar(func);
	bar(func);
}

int main(int argc, char* argv[])
{
	thread t1(threadfunc1);
	thread t2(threadfunc2);
	thread t3(threadfunc3);

	t1.join(); 
	t2.join();
	t3.join();
}

```



输出:

```
21884|threadfunc2#foo : ctor i(0)
21884|threadfunc2#foo : ref count add 1 to  i(0)
21884|threadfunc2#foo : ref count add 1 to  i(1)
21884|threadfunc2#foo : ref count add 1 to  i(2)
22812|threadfunc3#foo : ctor i(0)
22812|threadfunc3#foo : ref count add 1 to  i(0)
22812|threadfunc3#bar : ctor i(0)
22812|threadfunc3#bar : ref count add 1 to  i(0)
22812|threadfunc3#bar : ref count add 1 to  i(1)
26216|threadfunc1#foo : ctor i(0)
26216|threadfunc1#foo : ref count add 1 to  i(0)
26216|threadfunc1#foo : ref count add 1 to  i(1)
26216|threadfunc1#foo : ref count add 1 to  i(2)
21884|threadfunc2#foo : dtor i(3)
22812|threadfunc3#bar : dtor i(2)
22812|threadfunc3#foo : dtor i(1)
26216|threadfunc1#foo : dtor i(3)
```

从结果可以看出，线程`threadfunc1`和`threadfunc2`分别只调用了一次构造和析构函数，而且引用计数的递增也不会互相干扰，也就是说两个线程中线程局部存储对象是独立存在的。对于线程`threadfunc3`，它进行了两次线程局部存储对象的构造和析构，这两次分别对应`foo`和`bar`函数里的线程局部存储对象`tv`。可以发现，虽然这两个对象具有相同的对象名，但是由于不在同一个函数中，因此也应该认为是相同线程中不同的线程局部存储对象，它们的引用计数的递增同样不会相互干扰。