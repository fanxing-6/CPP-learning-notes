# 函数编程(参考《C++ Templates 英文版第二版》)

## Chapter 1 函数模板

### 1.1 一窥函数模板

```cpp
template<class T>
T max(T a, T b)
{
	return b < a ? a : b;
}
```

```cpp
#include "max1.hpp"
#include <iostream>
#include <string>
#include <format>

int main(int argc, char* argv[])
{
	
	int i = 42;
	std::cout << std::format("max(7,i):{}\n", ::max(7, i)); // C++20

	double f1 = 3.4;
	double f2 = -9.3;
	std::cout << std::format("max(f1,f2):{}\n", ::max(f1, f2));

	std::string s1 = "i am very intelligent";
	std::string s2 = "i am intelligent";
	/*
	 * 如果不使用::限定max为全局
	 * 则默认使用std::max(),对于在std空间内的类型
	 */
	std::cout << std::format("max(s1,s2):{}\n", ::max(s1, s2)); 
	
}

```

输出:

```cpp
max(7,i):42
max(f1,f2):3.4
max(s1,s2):i am very intelligent
```

**模板不会被编译成一个可以处理任何类型的实例,而是不同的类型使用模板时,模板会生成不同的实例**

通过确定的参数代替模板参数的过程叫**实例化**

### 1.1.3 两阶段Translation

为了一个不支持相应操作的类型实例化模板会导致错误

例如,复数比较大小

因此,模板的"编译"分两步:

1. 在没有实例化前:
   - 检查语法错误
   - 使用不依赖模板参数的名字
   - 检查不依赖于模板参数的静态断言
2. 在实例化阶段,模板代码被检查是否有效,所有依赖于模板的代码都将被检查



```cpp
template <typename T>
void f(T x) {
  undeclared();   // first-phase compile-time error if undeclared() unknown
  undeclared(x);  // second-phase compile-time error if undeclared(T) unknown
  static_assert(sizeof(int) > 10,
                "int too small");  // always fails if sizeof(int) <= 10
  static_assert(sizeof(T) > 10,
                "T too small");  // fails if instantiated for T with size <= 10
}
```

有些编译器不进行完全的第一阶段检查,你有可能观察不到上述现象



### 1.2 函数参数推断(Template Argument Deduction)

当我们调用函数模板时,模板参数将由我们传入的参数确定

但是,`T` 可能只是参数的"一部分",例如,我们我们使用常量引用声明`max()`

```cpp

template<typename T>
T amx(T const& a,T const& b)
{
	return a < b ? b : a;
}
```

**在类型推断中的类型转换**

在类型推断中类型转换是有限的

- 当使用引用传参时,参数类型必须完全匹配,编译器不会进行任何微小的转换
- 当使用值传参时,支持退化的转换,`const` `voaltile`限定符会被忽略,引用转换为引用类型,数组,函数将会被转换成指针类型,两个参数退化(`std::decayed`)[^1]后的类型必须匹配

```cpp
template <typename T>
T max(T a, T b) {
  return b < a ? a : b;
}

int a = 1;
const int b = 42;
int& c = a;
int arr[4];
std::string s;

::max(a, b);        // OK：T 推断为 int
::max(b, b);        // OK：T 推断为 int
::max(a, c);        // OK：T 推断为 int
::max(&a, arr);     // OK：T 推断为 int*
::max(1, 3.14);     // 错误：T 推断为 int 和 double
::max("hello", s);  // 错误：T 推断为 const char[6] 和 std::string

```

有三个方法解决这个问题:

1. 强制转换参数使他们都匹配

   ```cpp
   std::cout << std::format("max1(1,4.4):{}", ::max(static_cast<double>(1), 4.4));
   ```

2. 显示指定参数类型

   ```cpp
   std::cout << std::format("max1(1,4.4):{}", ::max<double>(1, 4.4));
   ```
   

**默认参数的类型推断**

```cpp
template<typename T>
void f(T = "")
{
}
f(1);  // OK：T 推断为 int，调用 f<int>(1)
f();   // 错误：不能推断 T
```

如果想要支持这个例子,你也可以声明一个默认参数类型

```cpp
f();  // OK
```

### 1.3 多种参数模板

函数模板有两种不同的参数

- `template parameters 模板参数` , 尖括号里面的就是模板参数

- `call parameters 调用参数`,就是参数列表里面的

  ```cpp
  template<typename T> // 模板参数
  T max(T a,T b)       // a,b就是调用参数
  ```

  你可以使用多个模板参数
  
  ```cpp
  
  template<typename T1,typename T2>
  T1 max(T1 a,T2 b)
  {
  	return a < b ? b : a;
  }
  
  ```
  
  这个例子有一些问题,虽然他很有可能正常工作,但是他可能违背程序员的意愿,将返回值强制转换成T1,例如 double->int
  
  有三种方式可以避免这个问题:
  
  - 再加一个模板参数作为返回值类型
  - 让编译器清楚返回值类型
  - 把返回值声明成两个模板参数的共同类型
  
  我们接下来讨论:
  
  #### 1.3.1 模板参数作为返回值
  
  我们可以明确的指定参数类型
  
  ```cpp
  template<typename T>
  T max(T a, T b)
  {
  	return a < b ? b : a;
  }
  ::max<int>(1,2.2);
  ```
  
  但是很多情况下,参数与返回值类型并没有关系,我们就可以声明第三个模板参数作为返回值
  
  ```cpp
  template<typename T1, typename T2,typename  RT>
  RT max(T1 a, T2 b)
  {
  	return a < b ? b : a;
  }
  ```
  
  但是**模板参数推断并不会考虑到返回值**,所也这么做是不行的,除非你手动指定返回值类型
  
  ```cpp
  ::max<int,double,double>(1,3.3)
  ```
  
  显然,这样做并没有什么优势
  
  #### 1.3.2 推断返回值
  
  ```cpp
  
  template<typename T1,typename T2>
  auto max(T1 a,T2 b)
  {
  	return a < b ? b : a;
  }
  ```
  
  `auto`使得真正返回值必须从函数体中推断出来
  
  然而,在一些情况下,函数传入参数是引用类型,你想返回值类型,这时你就需要使用`std::decay`[^1]去掉修饰符
  
  ```cpp
  template<typename T1, typename T2>
  auto max(T1 a, T2 b) ->typename std::decay<decltype(true?a:b)>::type
  {
  	return a < b ? b : a;
  }
  ```
  
  #### 1.3.3 返回公共类型(Common Type)
  
  上面的例子可以直接这么写
  
  ```cpp
  #include <type_traits>
  
  template<typename T1, typename T2>
  std::common_type_t<T1,T2> max(T1 a, T2 b) 
  {
  	return a < b ? b : a;
  }
  
  ```

### 1.4 默认模板参数

你也可以定义默认的模板参数

1. 我们可以使用之前的模板参数

   ```cpp
   template<typename T1, typename T2,typename RT = std::decay_t<decltype(true?T1():T2())>>
   RT max(T1 a, T2 b)
   {
   	return a < b ? b : a;
   }
   ```

   `std::decay_t`确保不会返回引用

2. 我们可以使用`std::common_type_t<>`与`std::decay`相同的效果

   ```cpp
   template<typename T1, typename T2, typename RT =std::common_type_t<T1,T2>>
   RT max(T1 a, T2 b)
   {
   	return a < b ? b : a;
   }
   ```

### 1.5 函数模板重载

```cpp
// maximum of two int values:
int max (int a, int b)
{
  return  b < a ? a : b;
}

// maximum of two values of any type:
template<typename T>
T max (T a, T b)
{
  return  b < a ? a : b;
}

int main()
{
  ::max(7, 42);          // calls the nontemplate for two ints
  ::max(7.0, 42.0);      // calls max<double> (by argument deduction)
  ::max('a', 'b');       // calls max<char> (by argument deduction)
  ::max<>(7, 42);        // calls max<int> (by argument deduction)
  ::max<double>(7, 42);  // calls max<double> (no argument deduction)
  ::max('a', 42.7);      // calls the nontemplate for two ints
}
```

总之,确保只有一个函数匹配

```cpp
#include <cstring>
#include <string>

// maximum of two values of any type:
template<typename T>
T max (T a, T b)
{
  return  b < a ? a : b;
}

// maximum of two pointers:
template<typename T>
T* max (T* a, T* b)
{
  return  *b < *a  ? a : b;
}

// maximum of two C-strings:
char const* max (char const* a, char const* b)
{
  return  std::strcmp(b,a) < 0  ? a : b;
}

int main ()
{
  int a = 7;
  int b = 42;
  auto m1 = ::max(a,b);     // max() for two values of type int

  std::string s1 = "hey";
  std::string s2 = "you";
  auto m2 = ::max(s1,s2);   // max() for two values of type std::string

  int* p1 = &b;
  int* p2 = &a;
  auto m3 = ::max(p1,p2);   // max() for two pointers

  char const* x = "hello";
  char const* y = "world";
  auto m4 = ::max(x,y);     // max() for two C-strings
}

```

如果用传引用实现模板，再用传值重载 C 字符串版本，不能用三个实参版本计算三个 C 字符串的最大值

```cpp
#include <cstring>

// maximum of two values of any type (call-by-reference)
template<typename T>
T const& max (T const& a, T const& b)
{
  return  b < a ? a : b;
}

// maximum of two C-strings (call-by-value)
char const* max (char const* a, char const* b)
{
  return  std::strcmp(b,a) < 0  ? a : b;
}

// maximum of three values of any type (call-by-reference)
template<typename T>
T const& max (T const& a, T const& b, T const& c)
{
  return max (max(a,b), c);       // error if max(a,b) uses call-by-value
}

int main ()
{
  auto m1 = ::max(7, 42, 68);     // OK

  char const* s1 = "frederic";
  char const* s2 = "anica";
  char const* s3 = "lucas";
  auto m2 = ::max(s1, s2, s3);    // run-time ERROR
}
```

错误原因是 `max(max(a, b), c)` 中，`max(a, b)` 产生了一个临时对象的引用，这个引用在计算完就马上失效了

![image-20210918182940269](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202109181829343.png)

**重载版本必须在函数调用前声明才可见**



**第一章结束**

# 未完待续

[^1]: Applies lvalue-to-rvalue, array-to-pointer, and function-to-pointer implicit conversions to the type `T`, removes cv-qualifiers, and defines the resulting type as the member typedef `type`. Formally:If `T` names the type "array of `U`" or "reference to array of `U`", the member typedef `type` is U*.Otherwise, if `T` is a function type `F` or a reference thereto, the member typedef `type` is [std::add_pointer](http://en.cppreference.com/w/cpp/types/add_pointer)<F>::type.Otherwise, the member typedef `type` is [std::remove_cv](http://en.cppreference.com/w/cpp/types/remove_cv)<[std::remove_reference](http://en.cppreference.com/w/cpp/types/remove_reference)<T>::type>::type.These conversions model the type conversion applied to all function arguments when passed by value.The behavior of a program that adds specializations for `decay` is undefined.

