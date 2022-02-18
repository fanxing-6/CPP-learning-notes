# std::packaged_task() ---C++17 并发编程

`std::packaged_task<>`连结了`future`对象与函数（或可调用对象）。

`std::packaged_task<>`对象在执行任务时，会调用关联的函数（或可调用对象），把返回值保存为`future`的内部数据，并令`future`准备就绪。它可作为线程池的构件单元，亦可用于其他任务管理方案。

例如，为各个任务分别创建专属的独立运行的线程，或者在某个特定的后台线程上依次执行全部任务。若一项庞杂的操作能分解为多个子任务，则可把它们分别包装到多个`std::packaged_task<>`实例之中，再传递给任务调度器或线程池。这就隐藏了细节，使任务抽象化，让调度器得以专注处理`std::packaged_task<>`实例，无须纠缠于形形色色的任务函数。
`std::packaged_task<>`是类模板，其模板参数是函数签名`（function signature）`：譬如，`void()`表示一个函数，不接收参数，也没有返回值；又如，`int(std::string&,double*`)代表某函数，它接收两个参数并返回`int`值，其中，第一个参数是非`const`引用，指向`std::string`对象，第二个参数是`double`类型的指针。

假设，我们要构建`std::packaged_task<>`实例，那么，由于模板参数先行指定了函数签名，因此传入的函数（或可调用对象）必须与之相符，即它应接收指定类型的参数，返回值也必须可以转换为指定类型。
这些类型不必严格匹配，若某函数接收`int`类型参数并返回`float`值，我们则可以为其构建`std::packaged_task<double(double)>`的实例，因为对应的类型可进行隐式转换。
类模板`std::packaged_task<>`具有成员函数`get_future()`，它返回`std::future<>`实例，该`future`的特化类型取决于函数签名所指定的返回值。
`std::packaged_task<>`还具备函数调用操作符，它的参数取决于函数签名的参数列表。

`std::packaged_task`对象是可调用对象，我们可以直接调用，还可以将其包装在`std::function`对象内，当作线程函数传递给`std::thread`对象，也可以传递给需要可调用对象的函数。

若`std::packaged_task`作为函数对象而被调用，它就会通过函数调用操作符接收参数，并将其进一步传递给包装在内的任务函数，由其异步运行得出结果，并将结果保存到`std::future`对象内部，再通过`get_future()`获取此对象。因此，为了在未来的适当时刻执行某项任务，我们可以将其包装在`std::packaged_task`对象内，取得对应的`future`之后，才把该对象传递给其他线程，由它触发任务执行。

等到需要使用结果时，我们静候`future`准备就绪即可。

举一个不是很恰当的例子:

```c++
#pragma once
#include <deque>
#include <future>
#include <mutex>
#include <thread>
#include <utility>
#include <iostream>

using namespace std;

/*
 * 模拟多线程加载 GUI 界面
 */
std::mutex mt;
std::deque<std::packaged_task<std::string()>> tasks;
std::vector<future<std::string>> futures;

void gui_thread()
{
	for (int i = 0; i < 3; ++i)
	{
		std::packaged_task<std::string()> task;
		{
			std::unique_lock<std::mutex> lk(mt);
			if (tasks.empty())
				continue;
			task = std::move(tasks.front());
			tasks.pop_front();
		}
		task();
		
	}
	
}


template <typename Func>
std::future<std::string> post_task_for_gui(Func f)
{
	std::packaged_task<std::string()> task(f);
	std::future<std::string> res = task.get_future();
	std::lock_guard<std::mutex> lk(mt);
	tasks.push_back(std::move(task));
	return res;
}

void start_422()
{
	for (int i = 0; i < 3; ++i)
	{
		futures.push_back(post_task_for_gui([=]()-> std::string
			{
				std::this_thread::sleep_for(3s);
				return std::format("第 {} 任务执行完毕", i);
			})
		);
		
	}
	cout << "任务提交完成" << endl;


	std::thread gui_background_thread(gui_thread);
	if (gui_background_thread.joinable())
		gui_background_thread.detach(); //后台执行


	for (auto& value : futures)
	{
		cout << value.get() << endl;
	}
}

```

本例采用`std::packaged_task<void()>`表示任务，包装某个函数（或可调用对象），它不接收参数，返回`void`（倘若真正的任务函数返回任何其他类型的值，则会被丢弃）。

这里，我们采用最简单的任务举例，但前文已提过，`std::packaged_task`也能用于更复杂的情况。针对不同的任务函数，`std::packaged_task`的函数调用操作符须就此修改参数，保存于相关的`future`实例内的返回值类型也须变动，而我们只要通过模板参数，指定对应任务的函数签名即可。

我们可轻松扩展上例，改动那些只准许在GUI线程上运行的任务，令其接收参数，并凭借`std::future`返回结果，`std::future`不再局限于充当指标，示意任务是否完成。