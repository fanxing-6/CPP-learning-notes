# The Ranges Library (2) --- C++20

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



你观察一下第二个重载,你会发现, 他需一个支持`ranges::random_access_range`的容器

|                                  | std::forward_list | std::list | std::deque | std::array | std::vector |
| -------------------------------- | ----------------- | --------- | ---------- | ---------- | ----------- |
| std::ranges::random_access_range |                   |           | ✅          | ✅          | ✅           |

一个谓词 `Comp`(返回`bool`的可调用对象),一个`std::identity`,`Comp`默认是`less`,`Prog`是[std::identity ](https://en.cppreference.com/w/cpp/utility/functional/identity):就是参数本身,本质是使用完美转发, 按照`Prog`将一个集合映射到子集

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <algorithm>

using namespace std;


struct PhoneBookEntry
{
	std::string name;
	int number;
};

void printPhoneBook(const std::vector<PhoneBookEntry>& phoneBook)
{
	for (const auto& entry : phoneBook) std::cout << "(" << entry.name << ", " << entry.number << ")";
	std::cout << "\n\n";
}

int main()
{
	std::cout << '\n';

	std::vector<PhoneBookEntry> phoneBook{
		{"Brown", 111}, {"Smith", 444},
		{"Grimm", 666}, {"Butcher", 222},
		{"Taylor", 555}, {"Wilson", 333}
	};

	std::ranges::sort(phoneBook, {}, &PhoneBookEntry::name); // 按姓名升序
	printPhoneBook(phoneBook);

	std::ranges::sort(phoneBook, std::ranges::greater(),
	                  &PhoneBookEntry::name); //按姓名降序
	printPhoneBook(phoneBook);

	std::ranges::sort(phoneBook, {}, &PhoneBookEntry::number); // 按号码升序
	printPhoneBook(phoneBook);

	std::ranges::sort(phoneBook, std::ranges::greater(),
	                  &PhoneBookEntry::number); // 按号码降序
	printPhoneBook(phoneBook);
}

```

![image-20220616153237923](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206161532058.png)

如果你有更加复杂的要求,可以使用`lambda`函数这样的可调用对象

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <algorithm>
#include <string>
using namespace std;


struct PhoneBookEntry
{
	std::string name;
	int number;
};

void printPhoneBook(const std::vector<PhoneBookEntry>& phoneBook)
{
	for (const auto& entry : phoneBook) std::cout << "(" << entry.name << ", " << entry.number << ")";
	std::cout << "\n\n";
}

int main()
{
	std::cout << '\n';

	std::vector<PhoneBookEntry> phoneBook{
		{"Brown", 111}, {"Smith", 444},
		{"Grimm", 666}, {"Butcher", 222},
		{"Taylor", 555}, {"Wilson", 333}
	};

	std::ranges::sort(phoneBook, {}, [](auto p) { return p.name; });
	printPhoneBook(phoneBook);

	std::ranges::sort(phoneBook, [](auto p, auto p2)
	{
		return std::to_string(p.number) + p.name <
			std::to_string(p2.number) + p2.name;
	});
	printPhoneBook(phoneBook);
}

```

## 直接查看key和value

可以直接查看`map`中的 key 和 value 

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <algorithm>
#include <string>
#include <unordered_map>

using namespace std;


int main()
{
	std::unordered_map<std::string, int> m{{"李宁", 12}, {"张三", 34}, {"张杰", 23}};
	auto names = std::views::keys(m);
	cout << "keys: ";
	for (auto name : names)
	{
		cout << name << " ";
	}
	cout << endl;

	auto ages = std::views::values(m);
	cout << "values: ";
	for (auto age : ages)
	{
		cout << age << " ";
	}
	cout << " ";
}

```

![image-20220616162131843](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206161621886.png)

但我们还有更加灵活的方式,使用`lambda`函数来完成一些更复杂的逻辑

## **Function Composition**

上一篇文章中有提及, 但不是很复杂, 我们实现一些更复杂的逻辑



```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <algorithm>
#include <string>
#include <unordered_map>

using namespace std;


int main()
{
	std::unordered_map<std::string, int> m{{"李宁", 12}, {"张三", 34}, {"张杰", 23}, {"西施", 18}, {"修勾", 4}};
	auto names = std::views::keys(m);
	cout << "keys: ";
	for (auto name : names)
	{
		cout << name << " ";
	}
	cout << endl;

	auto ages = std::views::values(m);
	cout << "values: ";
	for (auto age : ages)
	{
		cout << age << " ";
	}
	cout << " " << endl;

	std::cout << "张 开头的姓名: ";
	auto fistZ = [](const std::string& name) { return name.find("张", 0) == 0; };
	for (const auto& name : std::views::keys(m) | std::views::filter(fistZ))
	{
		cout << name << " ";
	}
	cout << endl;
}

```

![image-20220616163534876](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206161635921.png)



`|` 是一个语法糖, `C(R)`等价于 `R | C`

```cpp
auto rev1 = std::views::reverse(std::views::keys(m));
auto rev2 = std::views::keys(m) | std::views::reverse;
auto rev3 = freqWord | std::views::keys | std::views::reverse;
```

## **Lazy Evaluation** 惰性执行

上一篇文章已经介绍了

## 自定义`View`

```cpp
#include <ranges>
#include <vector>
#include <iostream>

template <class T, class A>
class VectorView : public std::ranges::view_interface<VectorView<T, A>>
{
public:
	VectorView() = default;

	VectorView(const std::vector<T, A>& vec) :
		m_begin(vec.cbegin()), m_end(vec.cend())
	{
	}

	auto begin() const { return m_begin; }
	auto end() const { return m_end; }
private:
	typename std::vector<T, A>::const_iterator m_begin{}, m_end{};
};

int main()
{
	std::vector<int> v = {1, 4, 9, 16};

	VectorView view_over_v{v};

	// We can iterate with begin() and end().
	for (int n : view_over_v)
	{
		std::cout << n << ' ';
	}
	std::cout << '\n';

	// We get operator[] for free when inheriting from view_interface
	// since we satisfy the random_access_range concept.
	for (std::ptrdiff_t i = 0; i < view_over_v.size(); i++)
	{
		std::cout << "v[" << i << "] = " << view_over_v[i] << '\n';
	}
}

```

![image-20220616173540337](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206161735401.png)



