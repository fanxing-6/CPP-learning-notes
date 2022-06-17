# The Ranges Library --- C++20

`ranges`可以让我们更加舒服的写代码了, 不用再敲那么多的代码

之前我们需要这样标准库的算法对容器的操作

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <algorithm>

int main()
{
	std::vector vec{ 1,5,67,8 };
	std::sort(vec.begin(), vec.end());
	for (auto value : vec)
	{
		std::cout << value << std::endl;
	}
}

```

有了`std::ranges`就可以

## 直接作用于容器

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <algorithm>

int main()
{
	std::vector vec{ 1,5,67,8 };
	std::ranges::sort(vec);
	for (auto value : vec)
	{
		std::cout << value << std::endl;
	}
}

```

`std::ranges::sort`可以直接作用于容器

## 懒执行

`std::ranges::iota`用于生成一个序列

示例如下:

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <algorithm>

using namespace std;

int main()
{
	for (auto value : std::views::iota(1, 10))
	{
		cout << value << " ";
	}

	cout << endl;
	cout << endl;

	for (auto value : std::views::iota(1) | std::views::take(10)) // 取10个数
	{
		cout << value << " ";
	}

	cout << endl;
	cout << endl;

	for (auto value : std::views::iota(1) | std::views::take_while([](auto i) { return i < 9; }))
	{
		cout << value << " ";
	}

	cout << endl;
	cout << endl;
}

```

## 函数组合

我们可以使用`std::views::filter`实现更复杂的逻辑

比如求素数

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <algorithm>

using namespace std;


bool isPrime(int i)
{
	for (int j = 2; j * j <= i; ++j)
	{
		if (i % j == 0) return false;
	}
	return true;
}


int main()
{
	auto add = [](int i) { return i % 2 == 1; };

	for (int i : std::views::iota(1)
	     | std::views::filter(add)
	     | std::views::filter(isPrime)
	     | std::views::take(100))
	{
		cout << i << "  ";
	}
	cout << "\n";
}

```

`std::views::iota(1)` 从 1 开始--->` std::views::filter(add)`是奇数---> `std::views::filter(isPrime)`是素数--->`std::views::take(100)`取前 100 个数

数据的流动是怎么开始的呢? 是从右到左, `std::views::take(10)` 希望有下一个值，因此需要询问其前,直到`for-loop`产生符合要求的数字

## 比较`std`与`std::ranges`算法

比较一下`std::sort`和`std::ranges::sort`

- `std::sort`

```cpp
template< class RandomIt >
constexpr void sort( RandomIt first, RandomIt last );

template< class ExecutionPolicy, class RandomIt >
void sort( ExecutionPolicy&& policy,
           RandomIt first, RandomIt last );

template< class RandomIt, class Compare >
constexpr void sort( RandomIt first, RandomIt last, Compare comp );

template< class ExecutionPolicy, class RandomIt, class Compare >
void sort( ExecutionPolicy&& policy,
           RandomIt first, RandomIt last, Compare comp );
```

- `std::ranges::sort`

```cpp
template <std::random_access_iterator I, std::sentinel_for<I> S,
         class Comp = ranges::less, class Proj = std::identity>
requires std::sortable<I, Comp, Proj>
constexpr I sort(I first, S last, Comp comp = {}, Proj proj = {});

template <ranges::random_access_range R, class Comp = ranges::less, 
          class Proj = std::identity>
requires std::sortable<ranges::iterator_t<R>, Comp, Proj>
constexpr ranges::borrowed_iterator_t<R> sort(R&& r, Comp comp = {}, Proj proj = {});
```

预知后事如何, 请听下回分解





















