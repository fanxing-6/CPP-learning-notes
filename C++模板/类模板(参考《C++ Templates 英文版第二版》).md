# 类模板(参考《C++ Templates 英文版第二版》)

## Chapter 1 类模板

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

# 未完待续,9.20晚继续更新





------



**我所有的学习笔记都在Github仓库里:https://github.com/fanxing-6/CPP-learning-notes **

**,如果访问Github有问题也可以访问Gitee:[CPP-learning-notes: C++学习笔记 (gitee.com)](https://gitee.com/fanxingin/CPP-learning-notes)**

**本人能力有限,笔记难免有疏漏,如果有错误,欢迎各位关注公众号与我交流,一定会及时回复**

![gfsdf](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202109192255474.png)

------



[^1]: 参考解答: [GotW #8: CHALLENGE EDITION: Exception Safety](http://www.gotw.ca/gotw/008.htm)
[^2]: C++17之后,如果参数类型可以从构造函数推断出来,可以跳过写模板参数即<T>