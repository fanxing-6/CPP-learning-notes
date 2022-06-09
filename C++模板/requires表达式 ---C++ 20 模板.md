# `requires`表达式 ---C++ 20 模板



`requires`还可以接一个表达式,该表达式也是一个**纯右值表达式**,表达式为`true`时满足约束条件,`false`则不满足约束条件

**`requires`表达式的判定标准:对`requires`表达式进行模板实参的替换,如果替换之后出现无效类型,或者违反约束条件,则值为`false`,反之为`true`**

```cpp
template <class T>
concept Check = requires {
	T().clear();
};

template <Check T>
struct G {};

G<std::vector<char>> x;      // 成功
G<std::string> y;            // 成功
G<std::array<char, 10>> z;   // 失败
```

失败原因

![image-20220519101032861](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205191010964.png)

由于`std::array`没有`clear`操作,所以编译失败

除此之外,我们还可以使用更灵活的方式,进行更多的限定:

```cpp

template <class T>
concept CheckType = requires(T a, T b)
{
	a.clear();
	a + b;
};

template <class CheckType>
struct A
{
};

```

在上面的requires表达式中，`a.clear()`和`a + b`可以说是对模板实参的两个要求，这些要求在C++标准中称为要求序列`（requirement-seq）`。要求序列分为4种，包括简单要求、类型要求、复合要求以及嵌套要求

## 简单要求  **simple requirements**

只要语法正确就行,编译器不会计算其结果

```cpp
template <class T>
concept Check = requires(T a, T b) {
	a + b;                         // 并不要求满足object+object ,即使传入array也可以通过
};

```



```cpp
template<class T>
concept C= requires(T a){
    std::is_pointer<T>::value;      //事实上,并不需要是一个指针
    a++;
};
```

## 类型要求 **type requirements**

类型要求是以`typename`关键字开始的要求，紧跟`typename`的是一个类型名，通常可以用来检查嵌套类型、类模板以及别名模板特化的有效性。如果模板实参替换失败，则要求表达式的计算结果为`false`

```cpp
template <typename T, typename T::type = 0>
struct S;
template <typename T>
using Ref = T&;
template <typename T> concept C = requires
{
	typename T::inner; // 要求嵌套类型
	typename S<T>; // 要求类模板特化
	typename Ref<T>; // 要求别名模板特化
};

template <C c>
struct M {};

struct H {
	using type = int;
	using inner = double;
};

M<H> m;
```

概念`C`中有3个类型要求，分别为`T::inner、S<T>`和`Ref<T>`，它们各自对应的是对嵌套类型、类模板特化和别名模板特化的检查。请注意代码中的类模板声明S，它不是一个完整类型，缺少了类模板定义。但是编译器仍然可以编译成功，因为标准明确指出类型要求中的命名类模板特化不需要该类型是完整的。

## 复合要求 **compound requirements**

```cpp
template <class T>
concept Check = requires(T a, T b) {
  {a.clear()} noexcept; // 支持clear,且不抛异常
  {a + b} noexcept -> std::same_as<int>; // std::same_as<decltype((a + b)), int>
};
```

```cpp
template<typename T> concept C =
requires(T x) {
    {*x} ;   // *x有意义                                               
{x + 1} -> std::same_as<int>; // x + 1有意义且std::same_as<decltype((x + 1)), int>，即x+1是int类型
{x * 1} -> std::convertible_to<T>; // x * 1 有意义且std::convertible_to< decltype((x *1),T>
};
```

## 嵌套要求 **nested requirements**

由若干条requires构成，每一条都需要满足。

```cpp
template <class T>
concept Check = requires(T a, T b) {
  requires std::same_as<decltype((a + b)), int>;
};
```

等同于:

```cpp
template <class T>
concept Check = requires(T a, T b) {
  {a + b} -> std::same_as<int>;
};
```

