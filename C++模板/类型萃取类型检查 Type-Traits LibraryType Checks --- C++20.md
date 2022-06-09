# 类型萃取:类型检查 Type-Traits Library:Type Checks --- C++20

**Type-Traits library** 在`C++11`的时候就已经发布,但依然随着C++版本在不断更新

## 类型检查 Type Checks

每种类型就是十四种主要类型之一

### 主要类型

```cpp
template <class T> struct is_void;
template <class T> struct is_integral;
template <class T> struct is_floating_point;
template <class T> struct is_array;
template <class T> struct is_pointer;
template <class T> struct is_null_pointer;
template <class T> struct is_member_object_pointer;
template <class T> struct is_member_function_pointer;
template <class T> struct is_enum;
template <class T> struct is_union;
template <class T> struct is_class;
template <class T> struct is_function;
template <class T> struct is_lvalue_reference;
template <class T> struct is_rvalue_reference;
```

例子:

```cpp
#include <iostream>
#include <type_traits>
#include <iostream>
#include <type_traits>

struct A
{
	int a;
	int f(int) { return 2011; }
};

enum E
{
	e = 1,
};

union U
{
	int u;
};


int main()
{
	using namespace std;

	cout << boolalpha << '\n'; // boolalpha: bool 以 true 或 false输出

	cout << is_void<void>::value << '\n'; // true                           
	cout << is_integral<short>::value << '\n'; // true
	cout << is_floating_point<double>::value << '\n'; // true
	cout << is_array<int[]>::value << '\n'; // true
	cout << is_pointer<int*>::value << '\n'; // true
	cout << is_null_pointer<nullptr_t>::value << '\n'; // true
	cout << is_member_object_pointer<int A::*>::value << '\n'; // true
	cout << is_member_function_pointer<int (A::*)(int)>::value << '\n'; // true
	cout << is_enum<E>::value << '\n'; // true
	cout << is_union<U>::value << '\n'; // true 
	cout << is_class<string>::value << '\n'; // true
	cout << is_function<int*(double)>::value << '\n'; // true	
	cout << is_lvalue_reference<int&>::value << '\n'; // true
	cout << is_rvalue_reference<int&&>::value << '\n'; // true
}

```

让我们试着实验一下这个魔法

```cpp
#include <iostream>
#include <type_traits>
#include <iostream>
#include <type_traits>

namespace fx
{
	template <class T, T v>
	struct intergral_constant
	{
		static constexpr T value = v;
		using value_type = T;
		using type = intergral_constant;

		constexpr operator value_type() const noexcept
		{
			return value;
		}

		constexpr value_type operator()() const noexcept
		{
			return value;
		}
	};

	using true_type = intergral_constant<bool, true>;
	using false_type = intergral_constant<bool, false>;

	template <class T>
	struct is_integral : public false_type
	{
	};

	template <>
	struct is_integral<bool> : public true_type
	{
	};

	template <>
	struct is_integral<char> : public true_type
	{
	};

	template <>
	struct is_integral<signed char> : public true_type
	{
	};

	template <>
	struct is_integral<unsigned char> : public true_type
	{
	};

	template <>
	struct is_integral<wchar_t> : public true_type
	{
	};

	template <>
	struct is_integral<short> : public true_type
	{
	};

	template <>
	struct is_integral<int> : public true_type
	{
	};

	template <>
	struct is_integral<long> : public true_type
	{
	};

	template <>
	struct is_integral<long long> : public true_type
	{
	};

	template <>
	struct is_integral<unsigned short> : public true_type
	{
	};

	template <>
	struct is_integral<unsigned int> : public true_type
	{
	};

	template <>
	struct is_integral<unsigned long> : public true_type
	{
	};

	template <>
	struct is_integral<unsigned long long> : public true_type
	{
	};
}


int main(int argc, char* argv[])
{
	
	std::cout << fx::is_integral<int>::value << std::endl;


}

```



`fx::is_integral<>::value`作为返回值,这是元函数的命名约定

自从C++17之后,有了一个更便捷的方式:

```cpp
template< class T >
inline constexpr bool is_integral_v = is_integral<T>::value
```

这样可以使用`std::integral_v<T>`代替`std::integral<int>::value`

## 复合类型 Composite Type Categories

[Composite data type - Wikipedia](https://en.wikipedia.org/wiki/Composite_data_type)

![image-20220525233832046](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205252338601.png)

对于主要类型和复合类型,类型萃取库提供类型属性和类型属性查询

### 类型属性

```cpp
template <class T> struct is_const;
template <class T> struct is_volatile;
template <class T> struct is_trivial;
template <class T> struct is_trivially_copyable;
template <class T> struct is_standard_layout;
template <class T> struct is_empty;
template <class T> struct is_polymorphic;
template <class T> struct is_abstract;
template <class T> struct is_final;
template <class T> struct is_aggregate;
 
template <class T> struct is_signed;
template <class T> struct is_unsigned;
template <class T> struct is_bounded_array;
template <class T> struct is_unbounded_array;
template <class T> struct is_scoped_enum;
 
template <class T, class... Args> struct is_constructible;
template <class T> struct is_default_constructible;
template <class T> struct is_copy_constructible;
template <class T> struct is_move_constructible;
 
template <class T, class U> struct is_assignable;
template <class T> struct is_copy_assignable;
template <class T> struct is_move_assignable;
 
template <class T, class U> struct is_swappable_with;
template <class T> struct is_swappable;
 
template <class T> struct is_destructible;
 
template <class T, class... Args> struct is_trivially_constructible;
template <class T> struct is_trivially_default_constructible;
template <class T> struct is_trivially_copy_constructible;
template <class T> struct is_trivially_move_constructible;
 
template <class T, class U> struct is_trivially_assignable;
template <class T> struct is_trivially_copy_assignable;
template <class T> struct is_trivially_move_assignable;
template <class T> struct is_trivially_destructible;
 
template <class T, class... Args> struct is_nothrow_constructible;
template <class T> struct is_nothrow_default_constructible;
template <class T> struct is_nothrow_copy_constructible;
template <class T> struct is_nothrow_move_constructible;
 
template <class T, class U> struct is_nothrow_assignable;
template <class T> struct is_nothrow_copy_assignable;
template <class T> struct is_nothrow_move_assignable;
 
template <class T, class U> struct is_nothrow_swappable_with;
template <class T> struct is_nothrow_swappable;
 
template <class T> struct is_nothrow_destructible;
 
template <class T> struct has_virtual_destructor;
 
template <class T> struct has_unique_object_representations;
```

### 类型属性查询

```cpp
template <class T> struct alignment_of;
template <class T> struct rank;
template <class T, unsigned I = 0> struct extent;
```

