# C++ 17 inline 内联定义静态变量

正在学习`C++20`新标准,突然看到`C++17`拓展`inline`变量,突然想到可不可以在类内部直接初始化静态变量,整个**单例模式**呢

- **不需要在类外部初始化静态变量**

- 实现懒加载,需要的时候才加载
- 线程安全
- 外部无法调用构造函数,析构函数

代码如下:

```cpp
// 单例模式
class singleton_pattern
{
private:
	
	inline static singleton_pattern* _instance_ptr{nullptr};// C++ 17 inline static 直接初始化

private:
	singleton_pattern()
	{
		cout << "constructor called" << endl;
	}
	
	singleton_pattern(singleton_pattern&) = delete;
	singleton_pattern& operator=(const singleton_pattern&) = delete;

	
public:
	~singleton_pattern()
	{
		cout << "destructor called" << endl;
	}

	static singleton_pattern* get_instance()
	{
		static  std::once_flag init_flag;//多线程条件下只执行一次

		std::call_once(init_flag, []()
			{
				if (_instance_ptr == nullptr)
					_instance_ptr = new singleton_pattern;
			});
		return _instance_ptr;
	}

	void print_addr()
	{
		cout << std::format("address: {} \n", (void*)_instance_ptr);
	}
	
};
```

如果有不对的地方还请纠正

![my](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/202201210103511.png)