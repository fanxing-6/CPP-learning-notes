# `constexpr`和`consteval` --- C++ 20

## 标准库容器和算法库对`constexpr` 的应用

C++20 中大量的算法和容器可以使用`constexpr`,这意味着你甚至可以再编译期`vector<int>`进行排序

[Algorithms library - cppreference.com](https://en.cppreference.com/w/cpp/algorithm)

如下:

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <unordered_set>
#include <algorithm>
#include <format>

constexpr int maxElement()
{
	std::vector myVec{1, 4, 5, 7, 23, 4};
	std::sort(myVec.begin(), myVec.end());
	return myVec.back();
}


int main(int argc, char* argv[])
{
	constexpr int maxValue1 = []()-> int
	{
		std::vector myVec = {1, 2, 4, 3}; 
		std::sort(myVec.begin(), myVec.end());
		return myVec.back();
	}(); //  immediately-invoked lambda


	std::cout << maxValue1 << std::endl;


	constexpr int maxValue = maxElement();
	std::cout << std::format("maxElement: {}", maxValue);
}

```

![image-20220601215824311](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202206012158414.png)

- immediately-invoked lambda : 即调用函数表达式：先创建Lambda表达式，并不分配给任何闭包对象，然后它被( )调用

## Transient Allocation (瞬时分配内存)

`Transient Allocation`: 编译期申请的内存也会在编译期释放

C++不支持` non-transient constexpr allocation`:编译期申请的内存提升为静态在运行时继续使用

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <unordered_set>
#include <algorithm>
#include <format>
#include <memory>


#include <memory>

constexpr auto correctRelease() {
    auto* p = new int[2020];
    
    delete[] p;
    return 2020;
}

constexpr auto forgottenRelease() { // (1)
    auto* p = new int[2020];
    return 2020;
}

constexpr auto falseRelease() {     // (3)
    auto* p = new int[2020];
    delete p;                       // (2)
    return 2020;
}

int main() {

    constexpr int res1 = correctRelease();
    // constexpr int res2 = forgottenRelease();
    // constexpr int res3 = falseRelease();

}
```

注释掉的函数编译失败,因为内存没有成对的申请和释放

`constexpr`有个缺点:无法确定是在编译期还是运行时执行

```cpp
#include <iostream>
#include <ranges>
#include <vector>
#include <unordered_set>
#include <algorithm>
#include <format>
#include <memory>


#include <memory>

constexpr int constexprFunction(int arg)
{
	return arg * arg;
}


int main()
{
	static_assert(constexprFunction(10) == 100); // (1)
	int arrayNewWithConstExpressiomFunction[constexprFunction(100)]; // (2)
	constexpr int prod = constexprFunction(100); // (3)

	int a = 100;
	int runTime = constexprFunction(a); // (4)

	int runTimeOrCompiletime = constexprFunction(100); // (5) 编译期和运行时都可以执行
}

```

所以C++20 就有了 `consteval`,一定在编译期执行

## `consteval`

**只能在编译期执行**

```cpp
consteval int sqr(int n) {
    return n * n;
}
```

- 每次调用即时函数都会创建一个编译期常量

- 不能应用于析构函数,或者申请或释放内存的函数

- 满足`constexpr`的所有要求

  

