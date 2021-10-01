# 非类型模板参数(参考《C++ Templates 英文版第二版》)

## Chapter 3

### 3.1 非类型类模板参数

与前几章的简单例子不同,你也可以通过`std::array`实例化一个固定大小的栈,这样做的优点在于内存管理,

```cpp
#include <array>
#include <cassert>

template<typename T, std::size_t Maxsize>
class Stack {
  private:
    std::array<T,Maxsize> elems; // elements
    std::size_t numElems;        // current number of elements
  public:
    Stack();                   // constructor
    void push(T const& elem);  // push element
    void pop();                // pop element
    T const& top() const;      // return top element
    bool empty() const {       // return whether the stack is empty
        return numElems == 0;
    }
    std::size_t size() const { // return current number of elements
        return numElems;
    }
};

template<typename T, std::size_t Maxsize>
Stack<T,Maxsize>::Stack ()
 : numElems(0)                 // start with no elements
{
    // nothing else to do
}

template<typename T, std::size_t Maxsize>
void Stack<T,Maxsize>::push (T const& elem)
{
    assert(numElems < Maxsize);
    elems[numElems] = elem;    // append element
    ++numElems;                // increment number of elements
}

template<typename T, std::size_t Maxsize>
void Stack<T,Maxsize>::pop ()
{
    assert(!elems.empty());
    --numElems;                // decrement number of elements
}

template<typename T, std::size_t Maxsize>
T const& Stack<T,Maxsize>::top () const
{
    assert(!elems.empty());
    return elems[numElems-1];  // return last element
}

```

运行一下:

```cpp
#include "stacknontype.hpp"
#include <iostream>
#include <string>

int main()
{
	Stack<int, 20>         int20Stack;     // stack of up to 20 ints
	Stack<int, 40>         int40Stack;     // stack of up to 40 ints
	Stack<std::string, 40> stringStack;    // stack of up to 40 strings

	// manipulate stack of up to 20 ints
	int20Stack.push(7);
	std::cout << int20Stack.top() << '\n';
	int20Stack.pop();

	// manipulate stack of up to 40 strings
	stringStack.push("hello");
	std::cout << stringStack.top() << '\n';
	stringStack.pop();
}
```

你也可以设置默认参数

`template<typename T, std::size_t Maxsize = 100>`,但最好不要这样做,因为栈大小最好还是有程序员自己控制

### 3.2 非类型函数模板参数

你也可以为函数定义非类型模板参数

```cpp
template<int val,typename  T>
int addval(T a)
{
	return a + val;
}
int main()
{
	std::cout << addval<10>(10) << std::endl;
}
```

这种函数是很有用的,可以将它作为参数.

例如,如果你可以使用STL,你可以传入一个函数模板的实例,使集合中的每个数加一个值

```cpp
	std::vector<int> source{ 1,4,5 };
	std::vector<int> dest{0,0,0};
	std::transform(source.begin(), source.end(), dest.begin(), addval<10,int>);
	for (auto value : dest)
	{
		std::cout << value << std::endl;
	}
```

如果你制定非类型函数模板参数是`int`那么就无法使用其他类型,是有什么方法可以做到自动推导其他类型呢?

其实,你可以指定一个根据之前的模板参数推断出来的模板参数

```cpp
template<auto val,typename  T = decltype(val)>
T addval(T a)
{
	return a + val;
}
```

### 3.3 非类型模板参数的限制

 非类型模板参数有一些限制,例如,它们只能是整数,(对象,函数,成员)指针,(对象,函数,成员)引用,`std::nullptr`

浮点数指针和类对象不可以作为非类型模板参数

### 3.4 模板参数类型`auto`

C++17 ,你可以定义一个非类型模板模板参数去普遍的接收任何类型,使用这个特性,你可以定义一个更加一般的固定大小栈

```cpp
#include <array>
#include <cassert>

template<typename T, auto Maxsize>
class Stack {
  public:
    using size_type = decltype(Maxsize);
  private:
    std::array<T,Maxsize> elems; // elements
    size_type numElems;          // current number of elements
  public:
    Stack();                   // constructor
    void push(T const& elem);  // push element
    void pop();                // pop element 
    T const& top() const;      // return top element
    bool empty() const {       // return whether the stack is empty
        return numElems == 0;
    }
    size_type size() const {   // return current number of elements
        return numElems;
    }
};

// constructor
template<typename T, auto Maxsize>
Stack<T,Maxsize>::Stack ()
 : numElems(0)                 // start with no elements
{
    // nothing else to do
}

template<typename T, auto Maxsize>
void Stack<T,Maxsize>::push (T const& elem)
{
    assert(numElems < Maxsize);
    elems[numElems] = elem;    // append element
    ++numElems;                // increment number of elements
}

template<typename T, auto Maxsize>
void Stack<T,Maxsize>::pop ()
{
    assert(!elems.empty());
    --numElems;                // decrement number of elements
}

template<typename T, auto Maxsize>
T const& Stack<T,Maxsize>::top () const
{
    assert(!elems.empty());
    return elems[numElems-1];  // return last element
}

```

运行:

```cpp
Stack<int, 20u>        int20Stack;     // stack of up to 20 ints
    Stack<std::string, 40> stringStack;    // stack of up to 40 strings

    // manipulate stack of up to 20 ints
    int20Stack.push(7);
    std::cout << int20Stack.top() << '\n';
    auto size1 = int20Stack.size();

    // manipulate stack of up to 40 strings
    stringStack.push("hello");
    std::cout << stringStack.top() << '\n';
    auto size2 = stringStack.size();

    if (!std::is_same_v<decltype(size1), decltype(size2)>) {
        std::cout << "size types differ" << '\n';
    }
```



