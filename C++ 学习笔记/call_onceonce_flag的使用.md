# call_once/once_flag的使用

保证在多线程环境中某个函数仅仅被调用一次,可以使用`std::call_once`函数,并且需要一个入参`once_flag`类型的入参

```cpp
#include<iostream>
#include <string>
#include <tuple>
#include <mutex>
#include <thread>
#include <list>
#include <condition_variable>
using namespace std;

std::once_flag flag;

void only_do_once()
{
	std::call_once(flag, []() {std::cout << "只调用一次哦"; });

}

int main()
{
	std::thread t1(only_do_once); 
	std::thread t2(only_do_once); 
	std::thread t3(only_do_once);
	t1.join();
	t2.join();
	t3.join();
}
```

看结果,这个函数`only_do_once`只被调用了一次

