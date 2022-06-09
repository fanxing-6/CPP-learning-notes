# 模板元编程(一) Template Metaprogramming--- C++ 20

## 在编译期进行类型操作

举个例子:

`std::move`在概念上应该这样实现(实际并不是这么做的):

```cpp
static_cast<std::remove_reference<decltype(arg)>::type&&>(arg);
```

意义上,`std::move`首先获取它的参数`arg`,推断出其类型,移除引用,最后转换为右值引用,移动语义就生效了.



如何移除参数的`const`呢?

```cpp
#include <iostream>
#include <type_traits>

template <typename T>
struct removeConst
{
	using type = T; // (1)
};

template <typename T>
struct removeConst<const T>
{
	using type = T; // (2)
};

int main()
{
	std::cout << std::boolalpha;
	std::cout << std::is_same_v<int, removeConst<int>::type> << '\n'; // true    
	std::cout << std::is_same_v<int, removeConst<const int>::type> << '\n'; // true
}

```

`std::is_same_v<int, removeConst<int>::type> `等同`std::is_same<int, removeConst<int>::type>::value`,其中`::type`表示编译器推断出来的模板类型

传入`int`时,应用`removeConst`

传入`const int`,应用`removeConst<const T>`,这样就移除了`const`,很好理解



## Metadata

**`metadata`是编译阶段元函数使用的数据**

一共有三种类型:

- 类型参数:`int`,`double`
- 非类型参数,例如:数字类型,枚举类型...
- 模板,例如`std::vector`

以后会详细解释这部分内容

## Metafunctions

**元函数是在编译期执行的函数**

元函数:

```cpp
template <int a , int b>
struct Product {
    static int const value = a * b;
};

template<typename T >
struct removeConst<const T> {
    using type = T;
};
```



### 函数 vs 元函数

```cpp
#include <iostream>

int power(const int m, const int n)
{
	int r = 1;
	for (int k = 1; k <= n; ++k) r *= m;
	return r;
}

template <int M, int N>
struct Power
{
	static int const value = M * Power<M, N - 1>::value;
};


template <int M>
struct Power<M, 0>
{
	static int constexpr value = 1;
};

int main()
{
	std::cout << '\n';

	std::cout << "power(2, 10)= " << power(2, 10) << '\n';
	std::cout << "Power<2,10>::value= " << Power<2, 10>::value << '\n';

	std::cout << '\n';
}

```



![image-20220522212804860](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205222128980.png)

- 参数:函数的参数在圆括号里面`(...)`,元函数的参数在尖括号里面`<...>`

- 返回值:函数返回一个语句,元函数返回一个静态常量值

  以后会介绍`constexpr`和`consteval`

## 混合编程 Hybrid Programming 

例子:

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

下一篇文章介绍

------

[What does “::value, ::type” mean in C++? - Quora](https://www.quora.com/What-does-value-type-mean-in-C)