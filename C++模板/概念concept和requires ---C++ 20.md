# 概念`concept`和`requires` ---C++ 20

## concept

`concept`简化了模板编程的难度

我们可以使用**`concept`定义模板形参的约束条件T** 模板实力替换T后必须满足`std::is_integral_v<C>;`为`true`

例子:

![image-20220518224709352](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205182247486.png)



`requires`关键字可以直接约束模板形参`T`

如下:

![image-20220518225045944](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205182250993.png)



```cpp
template <class C>
concept IntegerType = std::is_integral_v<C>
```

其中`IntegerType`是概念名,这里的`std::is_integral_v<C>`称为**约束表达式,是一个纯右值的常量表达式,在编译期确定**并且支持合取和析取

如果约束表达式为`true`,就表示任意类型都可以

如下

```cpp

template <typename T>
concept Type = true;

template <Type T>
struct Z
{
};


int main(int argc, char* argv[])
{
	Z z = Z<int>();

}

```

##  requires

```cpp
constexpr bool bar() { return true; }

template <class T>
requires (bar()) // bar()不是初等表达式，不符合语法规则,所以加个括号
struct X {};
```

requires子句除了能出现在模板形参列表尾部，还可以出现在函数模板声明尾部，所以下面的用法都是正确的:

```cpp
template <class T> requires std::is_integral_v<T>
void foo();

template <class T>
void foo() requires std::is_integral_v<T>;
```

那么如何确定多种约束的优先级呢?

例如:

```cpp
template <class C>
concept ConstType = std::is_const_v<C>;

template <class C>
concept IntegralType = std::is_integral_v<C>;

template <ConstType T>
	requires std::is_pointer_v<T>
void foo(IntegralType auto) requires std::is_same_v<T, char* const>
{
}

int main(int argc, char* argv[])
{
	foo<int>(1.5);
}

```

![image-20220518233412354](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205182334439.png)

编译器究竟应该用什么顺序检查约束条件呢？事实上，标准文档给出了明确的答案，编译器应该按照以下顺序检查各个约束条件。

1．模板形参列表中的形参的约束表达式，其中检查顺序就是形参出现的顺序。也就是说使用`concept`定义的概念约束的形参会被优先检查，放到刚刚的例子中`foo<int>()`;最先不符合的是`ConstType`的约束表达式`std::is_const_v<C>`。

2．模板形参列表之后的`requires`子句中的约束表达式。这意味着，如果`foo`的模板实参通过了前一个约束检查后将会面临`std::is_pointer_v<T>`的检查。

3．简写函数模板声明中每个拥有受约束`auto`占位符类型的形参所引入的约束表达式。还是放到例子中看，如果前两个约束条件已经满足，编译器则会检查函数实参是否满足`IntegralType`的约束。

4．函数模板声明尾部`requires`子句中的约束表达式。所以例子中最后检查的是`std::is_same_v<T,char * const>`

## 原子约束

`concept`格式

```cpp
template < template-parameter-list >
concept  concept-name = constraint-expression;
```

原子约束由一个表达式E，以及E的参数映射(parameter mappings)。E的参数映射的意思是 约束的实体的跟表达式E相关的模板参数和模板类型。

原子约束是在编译器进行`constraint normalization`时生成的。表达式E中必然不包含 AND 或者OR的逻辑，否则它就会被分割为2个原子约束。

原子约束是表达式和表达式中模板形参到模板实参映射的组合（简称为形参映射）。比较两个原子约束是否相同的方法很特殊，除了比较代码上是否有相同的表现，还需要比较形参映射是否相同，也就是说功能上相同的原子约束可能是不同的原子约束

```cpp
template <int N> constexpr bool Atomic = true;
template <int N> concept C = Atomic<N>;
template <int N> concept Add1 = C<N + 1>;
template <int N> concept AddOne = C<N + 1>;
template <int M> void f()
	requires Add1<2 * M> {};
template <int M> void f()
	requires AddOne<2 * M> && true {};

int main(int argc, char* argv[])
{
	f<0>(); 
}
```

`Add1`和`AddOne`其实是一样的约束,原子约束都是`concept C = Atomic<N>`,形参映射为`N~2*M+1`

![image-20220519001014510](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205190010584.png)

```cpp
template <class T> concept sad = false;
template <class T> int f1(T) requires (!sad<T>) { return 1; };
template <class T> int f1(T) requires (!sad<T>) && true {return 2; };

f1(0); // 编译失败
```

逻辑否定表达式是一个原子约束。所以以上代码会产生二义性,修改:

```
template <class T> concept not_sad = !sad<T>;
template <class T> int f2(T) requires (not_sad<T>) { return 3; };
template <class T> int f2(T) requires (not_sad<T>) && true  { return 4; };

f2(0);
```

还有点没搞明白,稍后再看看吧