# 类型萃取:类型比较 Type-Traits Library:type comparisons --- C++20

不涉及`runtime`只在编译期

## Comparing types 类型比较

`C++11` 支持三种类型:

- `is_base_of<Base, Derived>` 
- `is_convertible<From, To>`
- `is_same<T, U>`

`C++20`新加了几种:

- `is_pointer_interconvertible_with_class<From, To>`

  检查一个类型的对象是否与该类型的指定对象指针可互转换

- `is_pointer_interconvertible_base_of<Base, Derived>`

  检查一个类型的对象是否与该类型的指定子对象指针可互转换



```cpp
#include <iostream>
#include <type_traits>
using namespace std;


class BaseClass
{
public:
	int a;
};

class DerivedClass : public BaseClass
{
public:
	int b;
};

int main(int argc, char* argv[])
{
	cout << boolalpha << endl;

	cout << is_base_of<BaseClass, DerivedClass>::value << endl;
	cout << is_convertible<int, double>::value << endl;
	cout << is_convertible<int, string>::value << endl;
	cout << is_same<int, int>::value << endl;

	
}
```

```cpp

#include <type_traits>
#include <iostream>
 
struct Foo { int x; };
struct Bar { int y; };
 
struct Baz : Foo, Bar {}; // 非标准布局
 
int main()
{
    std::cout << std::boolalpha
        << std::is_same_v<decltype(&Baz::x), int Baz::*>
        << std::is_pointer_interconvertible_with_class(&Baz::x) << '\n'
        << std::is_pointer_interconvertible_with_class<Baz>(&Baz::x) << '\n';
}
```

[std::is_pointer_interconvertible_with_class - cppreference.com](https://zh.cppreference.com/w/cpp/types/is_pointer_interconvertible_with_class)

## 可能的实现方式

### `std::is_same`

```cpp
#include <iostream>
#include <type_traits>

namespace rgr {

  template<class T, T v>
  struct integral_constant {
      static constexpr T value = v;
      typedef T value_type;
      typedef integral_constant type;
      constexpr operator value_type() const noexcept { return value; }
      constexpr value_type operator()() const noexcept { return value; } //since c++14
  };

  typedef integral_constant<bool, true> true_type;                      // (2)              
  typedef integral_constant<bool, false> false_type;
  
  template<class T, class U>
  struct is_same : false_type {};                                       // (3)
 
  template<class T>
  struct is_same<T, T> : true_type {};

}

int main() {

    std::cout << '\n';

    std::cout << std::boolalpha;

    std::cout << "rgr::is_same<int, const int>::value: " 
              << rgr::is_same<int, const int>::value << '\n';          // (1)
    std::cout << "rgr::is_same<int, volatile int>::value: " 
              << rgr::is_same<int, volatile int>::value << '\n';
    std::cout << "rgr::is_same<int, int>::value: "  
              << rgr::is_same<int, int>::value << '\n';

    std::cout << '\n';

    std::cout << "std::is_same<int, const int>::value: " 
              << std::is_same<int, const int>::value << '\n';
    std::cout << "std::is_same<int, volatile int>::value: " 
              << std::is_same<int, volatile int>::value << '\n';
    std::cout << "std::is_same<int, int>::value: "  
              << std::is_same<int, int>::value << '\n';

    std::cout << '\n';

}
```

调用函数模板`rgr::is_same<int, const int>`就是调用模板函数`false_type::value`,因为`is_same : false_type {};   `

忽略`const`和`volatile`版本

```cpp
#include <iostream>
#include <type_traits>

namespace rgr {

  template<class T, T v>
  struct integral_constant {
      static constexpr T value = v;
      typedef T value_type;
      typedef integral_constant type;
      constexpr operator value_type() const noexcept { return value; }
      constexpr value_type operator()() const noexcept { return value; } //since c++14
  };

  typedef integral_constant<bool, true> true_type;                       
  typedef integral_constant<bool, false> false_type;

  template<class T, class U>
  struct is_same : false_type {};
 
  template<class T>
  struct is_same<T, T> : true_type {};
  
  template<typename T, typename U>                                    // (1)
  struct isSameIgnoringConstVolatile: rgr::integral_constant<
         bool,
         rgr::is_same<typename std::remove_cv<T>::type, 
                      typename std::remove_cv<U>::type>::value  
     > {};

}

int main() {

    std::cout << '\n';

    std::cout << std::boolalpha;

    std::cout << "rgr::isSameIgnoringConstVolatile<int, const int>::value: " 
              << rgr::isSameIgnoringConstVolatile<int, const int>::value << '\n';
    std::cout << "rgr::is_same<int, volatile int>::value: " 
              << rgr::isSameIgnoringConstVolatile<int, volatile int>::value << '\n';
    std::cout << "rgr::isSameIgnoringConstVolatile<int, int>::value: "  
              << rgr::isSameIgnoringConstVolatile<int, int>::value << '\n';

    std::cout << '\n';

}
```



使用`std::remove_cv`移除`const`和`volatile`



### `std::is_base_of`

```cpp
namespace details {
    template <typename B>
    std::true_type test_pre_ptr_convertible(const volatile B*);
    template <typename>
    std::false_type test_pre_ptr_convertible(const volatile void*);
 
    template <typename, typename>
    auto test_pre_is_base_of(...) -> std::true_type;
    template <typename B, typename D>
    auto test_pre_is_base_of(int) ->
        decltype(test_pre_ptr_convertible<B>(static_cast<D*>(nullptr)));
}
 
template <typename Base, typename Derived>
struct is_base_of :
    std::integral_constant<
        bool,
        std::is_class<Base>::value && std::is_class<Derived>::value &&
        decltype(details::test_pre_is_base_of<Base, Derived>(0))::value
    > { };
```

### `std::is_convertible`

```cpp
namespace detail {
 
template<class T>
auto test_returnable(int) -> decltype(
    void(static_cast<T(*)()>(nullptr)), std::true_type{}
);
template<class>
auto test_returnable(...) -> std::false_type;
 
template<class From, class To>
auto test_implicitly_convertible(int) -> decltype(
    void(std::declval<void(&)(To)>()(std::declval<From>())), std::true_type{}
);
template<class, class>
auto test_implicitly_convertible(...) -> std::false_type;
 
} // namespace detail
 
template<class From, class To>
struct is_convertible : std::integral_constant<bool,
    (decltype(detail::test_returnable<To>(0))::value &&
     decltype(detail::test_implicitly_convertible<From, To>(0))::value) ||
    (std::is_void<From>::value && std::is_void<To>::value)
> {};
```

这东西好难,智商太低......

------

[最全C++11/14/17/20/23 的新特性代码案例 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/389895793?utm_medium=social&utm_oi=776449486972030976)

[The Type-Traits Library: Type Comparisons - ModernesCpp.com](https://www.modernescpp.com/index.php/the-type-traits-library-type-comparisons)