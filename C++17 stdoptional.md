# C++17 std::optional<>

**The class template `std::optional` manages an *optional* contained value, i.e. a value that may or may not be present.
A common use case for `optional` is the return value of a function that may fail. As opposed to other approaches, such as [std::pair](https://link.zhihu.com/?target=http%3A//en.cppreference.com/w/cpp/utility/pair)<T,bool>, `optional` handles expensive-to-construct objects well and is more readable, as the intent is expressed explicitly.**

**类模板`std::optional`管理着一个可选的容纳值即可以存在也可以不存在的值。
一种常见的 `optional` 使用情况是一个可能失败的函数的返回值。与其他手段，如 [std::pair](https://link.zhihu.com/?target=http%3A//zh.cppreference.com/w/cpp/utility/pair)<T,bool> 相比， `optional` 良好地处理构造开销高昂的对象，并更加可读，因为它显式表达意图。**

以下一段程序演示使用std::optional<>`作为返回值

```cpp
#include<iostream>
#include <optional>
#include <string>

std::optional<int> toInt(const std::string& s)
{
	try
	{
		return std::stoi(s);
	}
	catch (...)
	{
		return std::nullopt;
	}
}

int main()
{
	for (auto a: { "42", " 077", "hello", "0x33" })
	{
		std::optional<int> oi = toInt(a);
		if (oi)
		{
			std::cout << "convert " << a << " to int: " << *oi << "\n";
		}
		else
		{
			std::cout << "can`t convert " << a << " to int\n";
		}
	}
}

```

在头文件<optional>,C++标准库定义了`std::optional<>`类:

```cpp
namespace std {
	template<typename T> class optional;
}
```

此外,以下的类型和对象被定义

- `std::nullopt_t`类型的`nullopt`作为类型optional没有值时的值

-  从 std::exception 派生的std::bad_optional_access 异常类，当无值时候访问值将会抛出该异常。

  可选对象也使用了 <utility> 头文件中定义的 std::in_place 对象（类型是 std::in_place_t）来初始

  化多个参数的可选对象（见下文）。

