# `constexpr` 函数 --- C++ 20

```cpp
constexpr double pi = 3.14;
```

`constexpr`允许你在编译时使用典型的C++函数语法进行编程,但这并不意味之`constexpr`只和编译期有关

**`constexpr`函数可以在编译期运行,也可以在运行时运行**

但在以下情况`constexpr`函数必须在编译期运行:

- `constexpr`函数在编译的上下文中需要,比如数组初始化,类型萃取库,断言表达式
- `constexpr`函数的值被`constexpr`的值需要,比如`constexpr int a = func();`

## 与模板元编程相似

![image-20220531223244872](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205312232915.png)

- 模板元程序一定在编译期运行,但是`constexpr`函数可以在编译期或运行时运行
- 模板元程序的参数可以使类型,非类型,模板
- 编译期是没有状态的,这意味着模板元程序是纯粹的函数式风格
  - 在模板元程序中,不会修改值,只会在每次返回值
  - 在编译期,实现一个循环所需要的可变变量是不可能实现的,所有的循环都依赖递归
  - 在模板元程序中,条件执行都被模板的特化取代

## 对比

1. `constexpr`参数与模板元函数的参数对比

![image-20220531224725257](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205312247309.png)

2. `constexpr`函数可以有可变的变量并修改,但是模板元函数无

![image-20220531225041140](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205312250189.png)

3. 模板元函数使用递归模拟循环

![image-20220531225139346](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205312251393.png)

4. 模板元函数使用专门化或者模板结束循环

![image-20220531225443877](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205312254924.png)

5. 模板元函数不会修改变量,而是在每次递归中返回一个新值

![image-20220531225615714](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205312256756.png)

6. 模板元函数没有返回值,直接使用值作为返回值

![image-20220531225746373](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202205312257423.png)

## 实例化

```cpp
template <typename T>
bool isSmaller(T fir, T sec)
{
	return fir < sec;
}

int main(int argc, char* argv[])
{
	isSmaller(5, 10); // (1)

	std::unordered_set<int> set1;
	std::unordered_set<int> set2;
	isSmaller(set1, set2); // (2)	
}

```

1. 检查模板语法,通常由编译器完成
2. 为每个不同类型的模板参数实例化一个模板函数,第二个函数会失败,因为`std::unordered_set<int>`不支持`<`

`constexpr`与此相似



------

什么是函数式编程思维？ - 用心阁的回答 - 知乎 https://www.zhihu.com/question/28292740/answer/40336090

