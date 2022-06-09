# `Curiously Recurring Template Pattern `奇怪的模板递归 --- C++20

我们都知道`C++`有静态多态和动态多态,动态多态通过虚函数表实现,他的缺点就是对效率产生一点点影响

可以用`CRTP`解决这个问题

我们先举一个动态多态的例子:

```cpp
#include <iostream>
using namespace std;

class Base
{
public:
	virtual void invoking()
	{
		cout << "Base" << endl;
	}
};

class Derived1 final : public Base
{
public:
	virtual void invoking() override final
	{
		cout << "Derived1" << endl;
	}
};

class Derived2 final : public Base
{
public:
	virtual void invoking() override final
	{
		cout << "Derived2" << endl;
	}
};

int main()
{
	Derived1* derived1 = new Derived1;
	Derived2* derived2 = new Derived2;

	derived1->invoking();
	derived2->invoking();

}
```

会在运行时查找虚函数表,影响运行时效率

## 使用`CRTP`编译器多态

特点:**子类继承基类,且基类有一个模板参数,并以子类类型为参数**

```cpp
#include <iostream>
using namespace std;


template <typename Derived>
class Base
{
public:
	void interface()
	{
		static_cast<Derived*>(this)->implementation();
	}

	void implementation()
	{
		cout << "Implementation Base" << endl;
	}
};

class Derived1 :public  Base<Derived1>
{
public:
	void implementation()
	{
		cout << "Implementation Derived1" << endl;
	}
};

class Derived2 : public  Base<Derived2>
{
public:
	void implementation()
	{
		cout << "Implementation Derived2" << endl;
	}
};

class Derived3 : public Base<Derived3>
{
public:
};

template <typename T>
void execute(T& base)
{
	base.interface();
}

int main(int argc, char* argv[])
{
	Derived1 d1;
	execute(d1);

	Derived2 d2;
	execute(d2);

	Derived3 d3;
	execute(d3);
}

```

每个基类都调用 `base.interface();`	然后调用`static_cast<Derived*>(this)->implementation();`

这样就在编译器实现了多态,避免了动态多态

```
1>class Base<class Derived1>	size(1):
1>	+---
1>	+---
1>class Base<class Derived2>	size(1):
1>	+---
1>	+---
1>class Base<class Derived3>	size(1):
1>	+---
1>	+---
```

![image-20220609223403879](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206092234000.png)

```cpp
#include <chrono>
#include <iostream>

auto start = std::chrono::steady_clock::now();

void writeElapsedTime() {
    auto now = std::chrono::steady_clock::now();
    std::chrono::duration<double> diff = now - start;

    std::cerr << diff.count() << " sec. elapsed: ";
}

template <typename ConcreteMessage>                        // (1)
struct MessageSeverity {
    void writeMessage() {                                     // (2)
        static_cast<ConcreteMessage*>(this)->writeMessageImplementation();
    }
    void writeMessageImplementation() const {
        std::cerr << "unexpected" << std::endl;
    }
};

struct MessageInformation : MessageSeverity<MessageInformation> {
    void writeMessageImplementation() const {               // (3)
        std::cerr << "information" << std::endl;
    }
};

struct MessageWarning : MessageSeverity<MessageWarning> {
    void writeMessageImplementation() const {               // (4)
        std::cerr << "warning" << std::endl;
    }
};

struct MessageFatal : MessageSeverity<MessageFatal> {};     // (5)

template <typename T>
void writeMessage(T& messServer) {

    writeElapsedTime();
    messServer.writeMessage();                            // (6)

}

int main() {

    std::cout << std::endl;

    MessageInformation messInfo;
    writeMessage(messInfo);

    MessageWarning messWarn;
    writeMessage(messWarn);

    MessageFatal messFatal;
    writeMessage(messFatal);

    std::cout << std::endl;

}
```

![image-20220609224336869](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206092243908.png)