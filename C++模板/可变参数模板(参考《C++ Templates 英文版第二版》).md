# 可变参数模板(参考《C++ Templates 英文版第二版》)

## Chapter 4 可变参数模板

自从C++11,模板可以接受可变数量的参数

### 4.1 可变参数模板

可以定义模板,去接受无限数量的模板参数

这种行为的模板叫做**可变参数模板**

#### 4.1.1 例子

```cpp
#include <iostream>

template<typename T>
void print(T arg)
{
	std::cout << arg << std::endl;
}

template<typename T, typename... Types>
void print(T firstArg, Types... args)
{
	std::cout << firstArg << '\n';  // print first argument
	print(args...);                 // call print() for remaining arguments
}

int main(int argc, char* argv[])
{
	print(1, 4, 7, "妙");
}
```

#### 4.1.3 运算符`sizeof`

C++11 之后,`sizeof`操作符对于可变参数模板有新的用法`sizeof...`,他返回参数包里面包含多少个元素

```cpp
template<typename T, typename... Types>
void print(T firstArg, Types... args)
{
	std::cout << sizeof... (Types) << std::endl;
	std::cout << sizeof... (args) << std::endl;
}
```

### 4.2 折叠表达式

C++11 提供了可变模板参数包, 使函数可以接受任意数量的参数. 但在 C++11中展开参数包稍显麻烦, 而 C++17 的折叠表达式使得展开参数包变得容易,其基本语法是使用 `(...)` 的语法形式进行展开.

# 支持的操作符

折叠表达式支持 32 个操作符: `+`, `-`, `*`, `/`, `%`, `^`, `&`, `|`, `=`, `<`,`>`, `<<`, `>>`, `+=`, `-=`, `*=`, `/=`, `%=`, `^=`, `&=`, `|=`, `<<=`,`>>=`,`==`, `!=`, `<=`, `>=`, `&&`, `||`, `,`, `.*`, `->*`

- 对于一元右折叠 `(E op …)` 具体展开为 `E1 op (… op (EN-1 op EN))`.
- 对于一元左折叠 `(… op E)` 具体展开为 `(( E1 op E2) op …) op En`.
- 对于二元右折叠 `(E op … op I)` 具体展开为 `E1 op (… op (EN-1 op (EN op I)))`.
- 对于二元左折叠 `(I op … op E)` 具体展开为 `(((I op E1) op E2) op …) op E2`.

```cpp
// define binary tree structure and traverse helpers:
struct Node {
  int value;
  Node* left;
  Node* right;
  Node(int i=0) : value(i), left(nullptr), right(nullptr) {
  }
  //...
};
auto left = &Node::left;
auto right = &Node::right;

// traverse tree, using fold expression:
template<typename T, typename... TP>
Node* traverse (T np, TP... paths) {
  return (np ->* ... ->* paths);      // np ->* paths1 ->* paths2 ...
}

int main()
{
  // init binary tree structure:
  Node* root = new Node{0};
  root->left = new Node{1};
  root->left->right = new Node{2};
  //...
  // traverse binary tree:
  Node* node = traverse(root, left, right);
  //...
}
```

