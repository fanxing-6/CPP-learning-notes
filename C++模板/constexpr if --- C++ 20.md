# `constexpr if` --- C++ 20

`constexpr if` 可以让我们实现条件编译

```cpp
template <typename T>
auto getResult(T t)
{
	if constexpr (std::is_integral_v<T>)
		return *t;

	else
		return t;
}
```

如果`T`是`intergral`类型,执行第一个分支,否则执行第二个分支

还记得前文写过的模板元编程递归求N次方吗? 使用`constexpr if`可以使我们的代码更加的优雅

```cpp
template <int N> 
constexpr int factorial()
{
	if constexpr (N >= 2)
		return N * factorial<N - 1>();
	else
		return N;
}


int main()
{
	cout << factorial<10>() << endl;
}
```

`constexpr if` 也会使得斐波那契数列更加的优雅

```cpp

template <int N>
constexpr int fibonacci()
{
	if constexpr (N >= 2)
		return fibonacci<N - 1>() + fibonacci<N - 2>();
	else
		return N;
}
```

