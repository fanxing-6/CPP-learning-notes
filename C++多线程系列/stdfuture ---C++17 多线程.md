# `std::future ` ---C++17 多线程

## `std::future`

C++标准程序库使用future来模拟这类一次性事件：若线程需等待某个特定的一次性事件发生，则会以恰当的方式取得一个future，它代表目标事件；接着，该线程就能一边执行其他任务（光顾机场茶座），一边在future上等待；同时，它以短暂的间隔反复查验目标事件是否已经发生（查看出发时刻表）。这个线程也可以转换运行模式，先不等目标事件发生，直接暂缓当前任务，而切换到别的任务，及至必要时，才回头等待future准备就绪。future可能与数据关联（如航班的登机口），也可能未关联。一旦目标事件发生，其future即进入就绪状态，无法重置。

总之,先保存一个事件(创建`future`对象),在未来获取事件的结果(`get`)

```cpp
#pragma once
#include <future>
#include <iostream>
#include <format>
using namespace std;

std::string get_answer_from_hard_question()
{
	std::this_thread::sleep_for(2s);
	cout << "id: " << std::this_thread::get_id << endl;
	return string("答案在这\n");
}
void do_something()
{
	std::this_thread::sleep_for(5s);
	cout << "id: " << std::this_thread::get_id << endl;
	cout << "结束任务\n";
}

void start()
{
	std::future<std::string> the_ans = std::async(std::launch::async,get_answer_from_hard_question);
	do_something();
	std::cout << std::format("答案是:{}", the_ans.get());
}
```

### 创建`future`对象的方法

```cpp
#include <string>
#include <future>
struct X
{
    void foo(int,std::string const&);
    std::string bar(std::string const&);
};
X x;    
auto f1=std::async(&X::foo,&x,42,"hello");    ⇽---  ①调用p->foo(42,"hello")，其中p的值是&x，即x的地址
auto f2=std::async(&X::bar,x,"goodbye");    ⇽---  ②调用tmpx.bar("goodbye")，其中tmpx是x的副本
struct Y                                
{
    double operator()(double);
}; 
Y y;
auto f3=std::async(Y(),3.141);    ⇽---  ③调用tmpy(3.141)。其中，由Y()生成一个匿名变量，传递给std::async()，进而发生移动构造。在std::async()内部产生对象tmpy，在tmpy上执行Y::operator()(3.141) 
auto f4=std::async(std::ref(y),2.718);    ⇽---  ④调用y(2.718)
X baz(X&);
std::async(baz,std::ref(x));    ⇽---  ⑤调用baz(x)
class move_only
{
public:
    move_only();
    move_only(move_only&&)
    move_only(move_only const&) = delete;
    move_only& operator=(move_only&&);
    move_only& operator=(move_only const&) = delete;
    void operator()();
}; 
auto f5=std::async(move_only());    ⇽---  ⑥调用tmp()，其中tmp等价于std::move (move_only())，它的产生过程与③相似
```

按默认情况下，`std::async()`的具体实现会自行决定——等待`future`时，是启动新线程，还是同步执行任务。大多数情况下，我们正希望如此。不过，我们还能够给`std::async()`补充一个参数，以指定采用哪种运行方式。参数的类型是`std::launch`，其值可以是`std::launch::deferred`或`std::launch::async`。前者指定在当前线程上延后调用任务函数，等到在`future`上调用了`wait()`或`get()`，任务函数才会执行；后者指定必须另外开启专属的线程，在其上运行任务函数。该参数的值还可以是`std::launch::deferred | std::launch:: async`，表示由`std::async()`的实现自行选择运行方式。最后这项是参数的默认值。若延后调用任务函数，则任务函数有可能永远不会运行。举例如下。

```cpp
auto f6=std::async(std::launch::async,Y(),1.2);    ⇽---  ①运行新线程

auto f7=std::async(std::launch::deferred,baz,std::ref(x));    ⇽---  ②在wait()或get()内部运行任务函数
auto f8=std::async(    ⇽---  
   std::launch::deferred | std::launch::async,
   baz,std::ref(x));
auto f9=std::async(baz,std::ref(x));     ⇽---  ③交由实现自行选择运行方式
f7.wait();    ⇽---  ④前面②处的任务函数调用被延后，到这里才运行
```


