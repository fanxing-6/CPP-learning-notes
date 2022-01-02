# lambda表达式浅析【C++学习笔记】

## 基本用法:

```cpp
auto f = [/*捕获列表*/](/*参数*/)->int /*后置返回值类型*/
	{
		/*
		 * 函数体
		 */
	};
```

**捕获列表:**

- [] : 不捕获任何变量

- [变量名] : 表示值捕获,不可修改
- [=] :按值捕获所有变量,不可修改
- [&] : 按引用捕获可以修改
- [this] : 在类中捕获,捕获当前类的`this`指针,如果使用 = & 捕获,则默认捕获`this`指针
- \[& 变量名\] :按引用捕获该变量
- \[ = , & 变量名\] ; 按值捕获所有变量,但是按引用捕获变量名变量,按引用捕获的变量,每个前面都有写一个&
- \[ &,变量名\] : 按引用捕获所有变量,但是按值捕获变量名变量

## lambda表达式延迟调用易错点

```cpp
int a = 12;
	auto f = [=]() ->int 
	{
		return a;
	};
	a = 99;
	cout << f() << endl;
```

输出:

![image-20211102201025117](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202111022010213.png)

为什么输出不是99呢?

**因为在遇到`auto f = [=]() ->int `这一行时,`a`的值就已经被复制到lambda表达式中了**

要避免这个错误可以使用按引用捕获

## lambda表达式类型

lambda表达式是闭包类型,可以理解成函数中的函数

编译器会为每个lambda表达式生成一个类,和一个可调用类对象

## lambda表达式用法介绍

```cpp
vector<int> vec{ 12,23,435,56,76 };
	int isnums = 9;
	for_each(vec.begin(), vec.end(), [=](int& val)
		{
			val -= isnums;
		});
	for (auto value : vec)
	{
		cout << value << endl;
	}
```

## 广义lambda捕获

解决lambda捕获依赖于类对象问题

将m_object复制到闭包里面来

```cpp
[ temp = m_object]()
{
    return temp;

};
```

