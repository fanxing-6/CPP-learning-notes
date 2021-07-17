# 自定义对象如何支持Range-based循环语法

至少实现以下两种语法:

```cpp
//返回第一个迭代子的位置
Iterator begin()
//返回最后一个迭代子的下一个位置
Iterator end()
```

迭代子需要支持如下三种方法:

- operator++(自增)
- operator!= (判不等)
- operator* (解引用)

```cpp
#include <iostream>
using namespace std;
template<typename T,size_t N>
class A
{
public:
	A()
	{
		for (size_t i =0 ;i<N;++i)
		{
			m_elements[i] = i;
		}
	}
	~A()
	{
		
	}
	T* begin()
	{
		return m_elements + 0;
	}
	T* end()
	{
		return m_elements + N;
	}
private:
	T m_elements[N];
};
int main()
{
	A<int, 10> a;
	for (auto iter: a)
	{
		std::cout << iter << endl;
	}
}
```

![image-20210702153940729](https://lzx-figure-bed.obs.dualstack.cn-north-4.myhuaweicloud.com/Figurebed/image-20210702153940729.png)

