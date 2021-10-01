# 类模板(参考《C++ Templates 英文版第二版》)

## Chapter 2 类模板

与函数相似,类也可以被一个或者多个类型参数化

在这章,我们使用栈作为例子

### 2.1 类模板`stack`的实现

```cpp
#include <vector>
#include <cassert>

template<typename  T>
class Stack
{
private:
	std::vector<T> elems;
    
public:
	void push(T const& elem);
	void pop();
	T const& top() const;
	bool empty() const
	{
		return elems.empty();
	}
};

template <typename T>
void Stack<T>::push(T const& elem)
{
	elems.push_back(elem);
}

template <typename T>
void Stack<T>::pop()
{
	assert(!elems.empty());
	elems.pop_back();
}

template <typename T>
T const& Stack<T>::top() const
{
	assert(!elems.empty());
	return elems.back();
}
```

这个类模板通过一个STL里面的类模板`vector<>`实现.这样,我们就不用去实现内存管理,拷贝构造,赋值运算符等内容,专注于类的实现



#### 2.1.2 成员函数的实现

如果你要定义一个类模板的成员函数,你必须去指定他是一个模板,并且满足类模板的全部类型,就像下面这样:

```cpp
void Stack<T>::push(T const& elem)
{
	elems.push_back(elem);
}
```

在这种情况下,`push_back()`被调用,向`vector`添加一个`elem`

注意:`pop_back()`函数只移除最后一个元素,但是不返回它,只是因为这种行为(只移除)是异常安全的,是不可能实现一个移除并返回最后一个元素的异常安全函数的[^1]

### 2.2 `stack`实用类模板

```cpp
#include "max1.hpp"
#include <iostream>
#include <string>
#include <format>
#include <type_traits>
#include "stack1.hpp"
#include <iostream>
#include <string>

int main()
{
	Stack<int>         intStack;       // stack of ints
	Stack<std::string> stringStack;    // stack of strings

	// manipulate int stack
	intStack.push(7);
	std::cout << intStack.top() << '\n';

	// manipulate string stack
	stringStack.push("hello");
	std::cout << stringStack.top() << '\n';
	stringStack.pop();
}
```

**C++17可以这么写[^2]**

```cpp
#include <vector>
int main()
{
	std::vector intVector{ 1,2 }; //省略<>
}
```



注意:**类模板只会实例化被调用的函数**,再上个例子中,`int string`的`top()`,`push()`都被实例化,但是`pop()`只在传入`string`实例化一次

### 2.3 类模板的部分使用(Partial Usage of Class Templates)

一个类模板通常对模板参数提供多种操作,这可能会给你错觉:类模板必须提供模板参数所有成员函数的操作. 

但事实上不是这样,**类模板只会提供被模板参数用到的成员函数**

在上文中加入以下代码:

```cpp
void printOn(std::ostream& strm)
    {
	    for (T const& elem : elems)
	    {
            strm << elem << std::endl;
	    }
    }
```

运行:

```cpp
Stack<std::pair<int, int>> ps;
	ps.push({ 2,5 });
	ps.push({ 3,5 });
	std::cout << ps.top().first << std::endl; //正确
	//ps.printOn(std::cout); // 错误
```

只有当你调用`printOn()`时,才会报错,说明**类模板只会实例化需要的成员函数**,

#### 2.3.1  概念(Concepts)

[类模板](https://zh.cppreference.com/w/cpp/language/class_template)、[函数模板](https://zh.cppreference.com/w/cpp/language/function_template)及非模板函数（常为类模板成员）可以与*制约*关联，制约指定模板实参上的要求，这能用于选择最准确的函数重载和模板特化。

制约亦可用于限制变量声明和函数返回类型中的自动类型推导，为只有满足指定要求的类型。

这种要求的具名集合被称为*概念*。每个概念都是谓词，于编译时求值，并成为模板接口的一部分，它在其中用作制约：

```cpp
#include <locale>
#include <string>
using namespace std::literals;

// 概念 "EqualityComparable" 的声明，任何有该类型值 a 和 b ，
// 而表达式 a==b 可编译而其结果可转换为 bool 的 T 类型满足它
template <typename T>
concept bool EqualityComparable = requires(T a, T b)
{
    {
        a == b
        } -> bool;
};

void f(EqualityComparable&&);  // 有制约函数模板的声明
// template<typename T>
// void f(T&&) requires EqualityComparable<T>; // 相同的长形式

int main()
{
    f("abc"s);  // OK ： std::string 为 EqualityComparable
    f(std::use_facet<std::ctype<char>>(
        std::locale{}));  // 错误：非 EqualityComparable
}
```

更多请参见:[制约与概念 - cppreference.com](https://zh.cppreference.com/w/cpp/experimental/constraints)

### 2.4 友元

与其使用printOn函数打印元素，不如重载`operator<<`，然而通常`operator<<`会实现为非成员函数。下面在类内定义友元，它是一个普通函数

```cpp
template<typename T> 
class Stack {
  ...
  void printOn(std::ostream& os) const;
  friend std::ostream& operator<<(std::ostream& os, const Stack<T>& stack)
  {
    stack.printOn(os); 
    return os;
  }
};
```

如果在类外定义友元，类模板参数不可见，事情会复杂很多

```cpp
template<typename T> 
class Stack {
  ...
  friend std::ostream& operator<<(std::ostream&, const Stack<T>);
};

std::ostream& operator<<(std::ostream& os,
  const Stack<T>& stack) // 错误：类模板参数T不可见
{
  stack.printOn(os);
  return os;
}
```

有两个解决方案:

- 一是隐式声明一个新的函数模板，并使用不同的模板参数

  ```cpp
  template<typename T> 
  class Stack {
    … 
    template<typename U> 
    friend std::ostream& operator<<(std::ostream&, const Stack<U>&);
  };
  
  // 类外定义
  template<typename U>
  std::ostream& operator<<(std::ostream& os, const Stack<U>& stack)
  {
    stack.printOn(os);
    return os;
  }
  ```

- 二是将友元前置声明为模板，而友元参数中包含类模板，这样就必须先前置声明类模板

  ```cpp
  template<typename T> // operator<<中参数中要求Stack模板可见
  class Stack;
  
  template<typename T>
  std::ostream& operator<<(std::ostream&, const Stack<T>&);
  
  // 随后就可以将其声明为友元
  template<typename T> 
  class Stack {
     …
     friend std::ostream& operator<< <T> (std::ostream&, const Stack<T>&);
  };
  
  // 类外定义
  template<typename T>
  std::ostream& operator<<(std::ostream& os, const Stack<T>& stack)
  {
    stack.printOn(os);
    return os;
  }
  
  ```

同样，函数只有被调用到时才实例化，元素没有定义`operator<<`时也可以使用这个类，只有调用`operator<<`时才会出错

```cpp
Stack<std::pair<int, int>> s; // std::pair没有定义operator<<
s.push({1, 2}); // OK
s.push({3, 4}); // OK
std::cout << s.top().first << s.top().second; // 34
std::cout << s << '\n'; // 错误：元素类型不支持operator<<
```

### 2.5 类模板特化

模板的实际应用中，有一些概念和应用很容易让人混淆，现在就分析一下模板的特化和实例化。编写模板的代码，最终的目的是应用，而在实际应用的过程中，大家最经常使用的是模板的实例化。也就是说模板说一族类或函数的抽象，那么要使用它，就需要把它应用到某个具体的类或者函数上。
另外还有一个绕不开的就是：**特化（specialization）**,从目前的教材来看有两种理解：

- 凡是把模板用具体的值来替代的过程都叫特化。如果这么理解，实例化也是特化的一种。
- 特化是普通模板通过具体的值来替换后不能满足一些特定的情况下的要求，需要对其进行特别的处理，包括偏特化和全特化。

全特化写法

```cpp
template<>
class Stack<std::string> 
{
    ...
};


```

实例:

```cpp
#include "stack1.hpp"
#include <deque>
#include <string>
#include <cassert>

template<>
class Stack<std::string> {
  private:
    std::deque<std::string> elems;  // elements

  public:
    void push(std::string const&);  // push element
    void pop();                     // pop element
    std::string const& top() const; // return top element
    bool empty() const {            // return whether the stack is empty
        return elems.empty();
    }
};

void Stack<std::string>::push (std::string const& elem)
{
    elems.push_back(elem);    // append copy of passed elem
}

void Stack<std::string>::pop ()
{
    assert(!elems.empty());
    elems.pop_back();         // remove last element
}

std::string const& Stack<std::string>::top () const
{
    assert(!elems.empty());
    return elems.back();      // return copy of last element
}

```

当我们使用`std::string`作为模板参数时,就会实例化这个用`std::string`特化的类.

我的理解:模板的特化就是为了处理一些非一般情况

### 2.6 偏特化

**函数是没有偏特化的。所以这里只是介绍类的偏特化。所谓偏特化，又叫局部特化或者部分特化，也就是在特定的条件下使用特定的对象来替换模板参数，但又不能完全替换**

```cpp
#include "stack1.hpp"

// partial specialization of class Stack<> for pointers:
template<typename T>
class Stack<T*> {
  private:
    std::vector<T*> elems;    // elements

  public:
    void push(T*);            // push element
    T* pop();                 // pop element
    T* top() const;           // return top element
    bool empty() const {      // return whether the stack is empty
        return elems.empty();
    }
};

template<typename T>
void Stack<T*>::push (T* elem)
{
    elems.push_back(elem);    // append copy of passed elem
}

template<typename T>
T* Stack<T*>::pop ()
{
    assert(!elems.empty());
    T* p = elems.back();
    elems.pop_back();         // remove last element
    return p;                 // and return it (unlike in the general case)
}

template<typename T>
T* Stack<T*>::top () const
{
    assert(!elems.empty());
    return elems.back();      // return copy of last element
}

```

使用

```cpp
template<typename T>
class Stack<T*>
{
    
}
```

我们定义了一个类模板

`T`仍然是模板参数,但是为了`T*`特化

**具有多个参数的部分特化**

```cpp
template<typename T1, typename T2> 
class MyClass 
{ … };
```

```cpp
// partial specialization: both template parameters have same type
template<typename T>
class MyClass<T,T>
{ … };
// partial specialization: second type is int
template<typename T>
class MyClass<T,int>
{ … };// partial specialization: both template parameters are pointer types
template<typename T1, typename T2>
class MyClass<T1*,T2*>
{ … };
```



```cpp
MyClass< int, float> mif; // uses MyClass<T1,T2> 
MyClass< float, float> mff; // uses MyClass<T,T> 
MyClass< float, int> mfi; // uses MyClass<T,int> 
MyClass< int*, float*> mp; // uses MyClass<T1*,T2*>
```

如果有多个模板匹配,就会歧义:

```cpp
MyClass< int, int> m; // ERROR: matches MyClass<T,T> and MyClass<T,int> 
MyClass< int*, int*> m; // ERROR: matches MyClass<T,T> and MyClass<T1*,T2*>
```

### 2.7 默认模板参数

对类模板,你可以设置一个默认模板参数,例如,对于`Stack`你可以设置一个默认模板参数管理内容

```cpp
template<typename T, typename Cont = std::vector<T>>
class Stack {
  private:
    Cont elems;                // elements

  public:
    void push(T const& elem);  // push element
    void pop();                // pop element
    T const& top() const;      // return top element
    bool empty() const {       // return whether the stack is empty
        return elems.empty();
    }
};

template<typename T, typename Cont>
void Stack<T,Cont>::push (T const& elem)
{
    elems.push_back(elem);     // append copy of passed elem
}

template<typename T, typename Cont>
void Stack<T,Cont>::pop ()
{
    assert(!elems.empty());
    elems.pop_back();          // remove last element
}

template<typename T, typename Cont>
T const& Stack<T,Cont>::top () const
{
    assert(!elems.empty());
    return elems.back();       // return copy of last element
}

```

这个例子就是使用`std::vector`作为默认内容管理器

**注意:这个类现在有两个模板参数,所以每个成员函数也应该有两个模板参数**

```cpp
int main()
{
  // stack of ints:
  Stack<int> intStack;

  // stack of doubles using a std::deque<> to manage the elements
  Stack<double,std::deque<double>> dblStack;

  // manipulate int stack
  intStack.push(7);
  std::cout << intStack.top() << '\n';
  intStack.pop();

  // manipulate double stack
  dblStack.push(42.42);
  std::cout << dblStack.top() << '\n';
  dblStack.pop();
}

```

这样` Stack<int> intStack;`使用`std::vector`管理元素,你也可以自定义管理器`Stack<double,std::deque<double>> dblStack;`



### 2.8 类型别名

为整个类型定义一个新名字让类模板更方便使用

通过使用`using`

```cpp
using IntStack = Stack <int>; // alias declaration 
void foo (IntStack const& s); // s is stack of ints 
IntStack istack[10]; // istack is array of 10 stacks of ints
```



```cpp
using IntStack = Stack <int>;
```

这样你就可以为整个类型定义一个别名,更方便使用

由于模板不是一个类型，所以不能定义一个typedef引用一个模板，但是新标准(since C++11)允许使用using为类模板定义一个别名

以`Stack使用`std::deque`管理元素为例

```cpp
template<typename T> 
using DequeStack = Stack<T, std::deque<T>>;
```

这样我们就可以使用`DequeStack<int>` 代替 `Stack<int,std::deque<int>>`,这两个表达的完全相同 

### 2.9 类模板参数推断

在C++17之前,你必须传入所有的模板参数类型,但是,C++17之后,这个限制放松了,**如果能通过构造函数推断出模板参数类型**,你可以不用明确的指定参数类型

```cpp
Stack< int> intStack1; // stack of strings 
Stack< int> intStack2 = intStack1; // OK in all versions 
Stack intStack3 = intStack1; // OK since C++17
```

构造函数:

```cpp
template<typename T> class Stack
{
private: std::vector<T> elems; // elements
	public: Stack () = default; Stack (T const& elem) // initialize stack with one element
	: elems({elem}) { }…
};
```

你可以这样声明一个`Stack`:

```cpp
Stack intStack = 0; // Stack<int> deduced since C++17
```

通过用整初始化`Stack`，推断出模板参数`T`为`int`，从而实例化一个`Stack<int>`

原则上也可以传递字符串字面值常量，但这样会造成许多麻烦。用引用传递模板类型T的实参时，模板参数不会decay，最终得到的类型是原始数组类型

```cpp
Stack stringStack = "bottom"; // Stack<char const[7]> deduced since C++17
```

传值的话则不会有这种问题，模板实参会decay，原始数组类型会转换为指针

```cpp
template<typename T> 
class Stack {
 public:
  Stack(T x) : v({x}) {}
 private:
  std::vector<T> v;
};

Stack stringStack = "bottom"; // Stack<const char*> deduced since C++17
```

传值时最好使用[std::move](https://en.cppreference.com/w/cpp/utility/move)以避免不必要的拷贝

```cpp
template<typename T> 
class Stack {
 public:
  Stack(T x) : v({std::move(x)}) {}
 private:
  std::vector<T> v;
};
```

**推断指引(**Deduction Guides)

如果构造函数不想使用传值方式声明,也有其他的解决办法,

你可以定义一个专用的**类型指引**,将C字符串推断为`std::string`

```cpp
Stack( char const*) -> Stack<std::string>;
```

这个指引必须出现在类定义的块,或者命名空间里

```cpp
template<typename T>
Stack(const char*) -> Stack<std::string>;
Stack stringStack{"bottom"}; // OK: Stack<std::string> deduced since C++17
```

但由于语法限制,下面这种方法不可以

```cpp
Stack stringStack = "bottom"; // Stack<std::string> deduced, but still not valid
```

因为不能使用拷贝构造(=) 传递一个字符串去构造一个`std::string`.

你可以这样:

```cpp
Stack stack2{stringStack}; // Stack<std::string> deduced 
Stack stack3(stringStack); // Stack<std::string> deduced 
Stack stack4 = {stringStack}; // Stack<std::string> deduced
```



### 2.10 模板化聚合（Templatized Aggregates）

聚合类也能作为模板

```cpp
template<typename T> 
struct A {
  T x;
  std::string s;
};
```

这样可以为了参数化值而定义一个聚合，它可以像其他类模板一样声明对象，同时当作一个聚合使用

```cpp
A<int> a;
a.x = 42;
a.s = "initial value";
```

C++17中可以为聚合类模板定义deduction guide

```cpp
template<typename T> 
struct A {
  T x;
  std::string s;
};

A(const char*, const char*) -> A<std::string>;

int main()
{
  A a = { "hi", "initial value" };
  std::cout << a.x; // hi
}
```

没有deduction guide，初始化就无法进行，因为A没有构造函数来推断。[std::array](https://en.cppreference.com/w/cpp/container/array)也是一个聚合，元素类型和大小都是参数化的，C++17为其定义了一个deduction guide

```cpp
namespace std {
template<typename T, typename... U> array(T, U...)
  -> array<enable_if_t<(is_same_v<T, U> && ...), T>, (1 + sizeof...(U))>;
}

std::array a{ 1, 2, 3, 4 };
// 等价于
std::array<int, 4> a{ 1, 2, 3, 4 };
```





[^1]: 参考解答: [GotW #8: CHALLENGE EDITION: Exception Safety](http://www.gotw.ca/gotw/008.htm)
[^2]: C++17之后,如果参数类型可以从构造函数推断出来,可以跳过写模板参数即<T>