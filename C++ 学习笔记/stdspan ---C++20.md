# `std::span` ---C++20

*`std::span`的定义*

```cpp
template<
    class T,
    std::size_t Extent = std::dynamic_extent
> class span;
```

`std::span`是指向一组连续的对象的对象,  是一个视图`view`, 不是一个拥有者`owner`

**一组连续的对象可以是 C 数组, 带着大小的指针, `std::array`, 或者` std::string`**

`std::span`可以有两种范围:

- 静态范围`static extend`
- 动态范围`dynamic extend`

## 静态范围, 动态范围

**静态范围`static extent`: **编译期就可以确定大小

**动态范围`dynamic extend`: ** 由指向第一个对象的指针和连续对象的大小组成

```cpp
#include <ranges>
#include <vector>
#include <iostream>
#include <span>
#include <format>

void printSpan(std::span<int> container)
{
	std::cout << std::format("container size: {} \n", container.size());
	for (auto ele : container)
	{
		std::cout << ele << " ";
	}
	std::cout << std::endl;
	std::cout << std::endl;
}


int main()
{
	std::vector v1{1, 2, 3, 4, 5, 8};
	std::vector v2{9, 2, 4, 2, 6, 78};

	std::span<int> dynamicSpan(v1);
	std::span<int, 6> staticSpan(v2);

	printSpan(dynamicSpan);
	printSpan(staticSpan);
}

```

## 构造

- 默认构造 `template< class T, std::size_t Extent = std::dynamic_extent > class span;`
- 使用迭代器(`std::contiguous_iterator`)
- 迭代器  +  大小
- C-stye数组

```cpp

int main()
{
	std::vector v1{1, 2, 3, 4, 5, 8};
	std::vector v2{9, 2, 4, 2, 6, 78};

	std::span<int> dynamicSpan(v1); // 默认构造
	std::span<int, 6> staticSpan(v2);

	std::span<int, 2> undefineSpan(v2); // 未定义行为 2 != 6

	std::span<int> iteratorSpan1(v1.begin(), v1.end());
	std::span<int> iteratorSpan2(v1.begin()+1, v1.end());

	std::span<int> iteratorsizeSpan3(v1.begin(), 6); // iterator + size

	int cArray[5]{ 1, 2, 3, 4, 5 };
	std::array<int, 5> cppArray{ 1, 2, 3, 4, 5 };

	std::span<int> cArrSpan(cArray);
	std::span<int> cppArrSpan{cppArrSpan};
	std::span<int,5> cppArrSpan1{cppArrSpan};
}

```



- **`std::span` 在构造时不支持隐式类型转换**
- `std::span`可以推断C-stye数组的大小

## 修改

可以修改`std::span`的指向,或者修改子集

```cpp
#include <ranges>
#include <vector>
#include <iostream>
#include <span>
#include <format>
#include <array>

void printSpan(std::span<int> container)
{
	std::cout << std::format("container size: {} \n", container.size());
	for (auto ele : container)
	{
		std::cout << ele << " ";
	}
	std::cout << std::endl;
	std::cout << std::endl;
}


int main()
{
	std::cout << '\n';

	std::vector vec{1, 2, 3, 4, 5, 6, 7, 8, 9, 10};
	printSpan(vec);

	std::span span1(vec);
	std::span span2{span1.subspan(1, span1.size() - 2)};


	std::transform(span2.begin(), span2.end(),
	               span2.begin(),
	               [](int i) { return i * i; });


	printSpan(vec);
	printSpan(span1);
}

```



















------

[std::span - cppreference.com](https://en.cppreference.com/w/cpp/container/span)

[std::basic_string_view - cppreference.com](https://en.cppreference.com/w/cpp/string/basic_string_view)
