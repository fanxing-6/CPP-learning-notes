#  std::function和std::bind

## `std::function`

1. 可调用对象
   - 是一个函数指针
   - 是一个具有operator()成员函数的类和对象
   - 可被转换成函数指针的类对象；
   - 一个类成员函数指针；

C++中**可调用对象**的虽然都有一个比较统一的操作形式，但是定义方法五花八门，这样就导致使用统一的方式保存可调用对象或者传递可调用对象时，会十分繁琐。C++11中提供了std::function和std::bind统一了可调用对象的各种操作。

```cpp
// 普通函数
int add(int a, int b){return a+b;} 

// lambda表达式
auto mod = [](int a, int b){ return a % b;}

// 函数对象类
struct divide{
    int operator()(int denominator, int divisor){
        return denominator/divisor;
    }
};
```

上述表达式虽然不同但是是同一种调用形

```cpp
int(int,int)
```

`std::function`就可以将上述类型保存下来

```cpp
std::function<int(int ,int)>  a = add; 
std::function<int(int ,int)>  b = mod ; 
std::function<int(int ,int)>  c = divide(); 
```



## `std::bind`

可将std::bind函数看作一个通用的函数适配器，它接受一个可调用对象，生成一个新的可调用对象来“适应”原对象的参数列表。

std::bind将可调用对象与其参数一起进行绑定，绑定后的结果可以使用std::function保存。std::bind主要有以下两个作用：

- 将可调用对象和其参数绑定成一个防函数；
- 只绑定部分参数，减少可调用对象传入的参数。

### `std::bind`绑定普通函数

```cpp
double my_divide (double x, double y) {return x/y;}
auto fn_half = std::bind (my_divide,placeholders::_1,2);  
std::cout << fn_half(10) << '\n';                      
```

- `bind`的第一个参数是函数名，回隐式转换为函数指针，所以写成`&my_divide`也可以；
- 第二个参数是占位符，`placeholders::_1`自动匹配第一个参数
- 占位符可以有多个
- 第三个参数是默认值，调用时第二个参数默认是2；（如果调用时第二个参数你指定了，那也没有用，还是使用2);



