# 模板元编程(二) Template Metaprogramming ---C++ 20

现在我们介绍**参数与模板参数混合使用**

先看一下例子:

```cpp
#include <iostream>

int power(int m, int n)
{
	int r = 1;
	for (int k = 1; k <= n; ++k) r *= m;
	return r;
}

template <int m, int n>
struct Power
{
	static int const value = m * Power<m, n - 1>::value;
};

template <int m>
struct Power<m, 0>
{
	static int const value = 1;
};

int main()
{
	std::cout << '\n';

	std::cout << "power(2, 10)= " << power(2, 10) << '\n';
	std::cout << "Power<2,10>::value= " << Power<2, 10>::value << '\n';

	std::cout << '\n';
}

```

下面我们来看一个更好的例子:

```cpp
#include <iostream>

template <int n>
int Power(int m)
{
	return m * Power<n - 1>(m);
}

template <>
int Power<0>(int m)
{
	return 1;
}

int main()
{
	std::cout << '\n';

	std::cout << "Power<0>(10): " << Power<0>(20) << '\n';
	std::cout << "Power<1>(10): " << Power<1>(10) << '\n';
	std::cout << "Power<2>(10): " << Power<2>(10) << '\n';


	std::cout << '\n';
}

```

这个例子更加清晰明了



## Hybrid Programming (参数与模板参数混合编程)

`Power<0>(10)`既有`()`中的函数参数也有`<>`中的模板参数,所以**`Power`同时是一个函数和元函数**

深入了解一下,我们可以用模板参数`2`实例化`Power`,并在循环中使用:

```cpp
#include <iostream>

template <int n>
int Power(int m)
{
	return m * Power<n - 1>(m);
}

template <>
int Power<0>(int m)
{
	return 1;
}


int main(int argc, char* argv[])
{
	std::cout << std::endl;
	const auto Power2of = Power<2>;
	for (int i = 0; i <= 20; ++i)
	{
		std::cout << "Power2of(" << i << ")= "
			<< Power2of(i) << '\n';
	}

	std::cout << '\n';
}

```

![image-20220523221550705](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205232215816.png)

可以使用这种方式计算每个数的平方

**不可以在循环中给`Power`可变的模板参数**像这样:

```cpp
for (int i = 0; i <= 20; ++i)
	{
		std::cout << "Power<" << i << ">(2)= " << Power<i>(2) << '\n';
	}
```

## 使用模板创建全新的类型

**每个模板实例化之后都会创建一个新的类型**

例如:

```cpp
auto res1 = Power<2>(10);                       // (1)
auto res2 = Power<2>(11);                       // (2)
auto rest3 = Power<3>(10);                      // (3)
```



------

前文的例子存在很多缺陷,比如传入`Power<-1>(10)`传入一个不合适的数,`Power<2000>(10)`运行时发生溢出

第一个问题可以使用`concept`解决,第二个问题暂时没有好的解决办法

但我们现在使用`static_assert`解决第一个问题

```cpp
#include <iostream>

template <int n>

int Power(int m)
{
	static_assert(n >= 0, "exponent must be >= 0");
	return m * Power<n - 1>(m);
}

template <>
int Power<0>(int m)
{
	return 1;
}


int main(int argc, char* argv[])
{
	std::cout << std::endl;
	const auto Power2of = Power<-1>;
	for (int i = 0; i <= 20; ++i)
	{
		std::cout << "Power2of(" << i << ")= "
			<< Power2of(i) << '\n';
	}

	std::cout << '\n';
}

```

![image-20220523222808692](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205232228725.png)



## 不要在编译阶段使用模板元编程

现在都是为了跟好的理解模板在编译器是如何工作的,**更好的选择是`constexpr`和`consteval`**

现在都是为了跟好的理解模板在编译器是如何工作的,**更好的选择是`constexpr`和`consteval`**

现在都是为了跟好的理解模板在编译器是如何工作的,**更好的选择是`constexpr`和`consteval`**

现在都是为了跟好的理解模板在编译器是如何工作的,**更好的选择是`constexpr`和`consteval`**

