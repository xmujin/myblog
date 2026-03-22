---
title: "C++基础"
date: '2026-03-21T10:09:15+08:00'
# weight: 1
# aliases: ["/first"]
# tags: ["nothing"]
# categories: ["日常"]
author: "向洵"
# author: ["Me", "You"] # multiple authors
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
# description: "添加描述文本"
canonicalURL: "https://canonical.url/to/page"
disableHLJS: true # to disable highlightjs
disableShare: false
hideSummary: false
searchHidden: true
ShowReadingTime: true
ShowBreadCrumbs: true
ShowPostNavLinks: true
ShowWordCount: true
ShowRssButtonInSectionTermList: true
UseHugoToc: true
---

## 变量赋值和初始化
### 变量赋值
```cpp
int width; // 定义一个被命名为width的整型变量
width = 5; // 将5赋值到width上
```
默认情况下，赋值会将`=`运算符右侧的值复制到运算符左侧的变量中。这称为**复制赋值**。
### 变量初始化
#### 不同形式的初始化
```cpp
int a;         // 默认初始化

// 传统初始化风格
int b = 5;     // 复制初始化
int c ( 6 );   // 直接初始化

// 现代初始化风格
int d { 7 };   // 列表初始化
int e {};      // value-initialization (empty braces)
```
#### 默认初始化
当没有提供初始化器时（例如上面的变量 a），这称为默认初始化。在许多情况下，默认初始化不执行任何初始化，并使变量具有**不确定值**
#### 复制初始化
继承自C语言风格
#### 直接初始化
没啥卵用
```cpp
int width(5);
```
#### 列表初始化
列表初始化有两种形式：
```cpp
int width { 5 };    // 推荐使用
int height = { 6 }; // 使用少
```
#### 值初始化和零初始化
当使用一组空的花括号初始化变量时，会发生一种特殊的列表初始化形式，称为值初始化。在大多数情况下，值初始化会隐式将变量初始化为零（或给定类型最接近零的值）。在发生置零的情况下，这称为零初始化。
```cpp
int width {}; // 值初始化和零初始化
```
问：我应该用 { 0 } 还是 {} 来初始化？
当实际使用初始值时，使用直接列表初始化
```cpp
int x { 0 };    // 用0进行列表初始化
std::cout << x; // we're using that 0 value here
```
当对象的值是临时的并且将被替换时，使用值初始化
```cpp
int x {};      // 值初始化
std::cin >> x; // 立即被替换，显式写0没有意义
```
实际上直接写{}就行，多写个0麻烦
### [[maybe_unused]]属性`C++17`
它允许我们告诉编译器，我们不介意变量未使用。编译器不会为此类变量生成未使用变量警告。
```cpp
#include <iostream>

int main()
{
    [[maybe_unused]] double pi { 3.14159 };  // Don't complain if pi is unused
    [[maybe_unused]] double gravity { 9.8 }; // Don't complain if gravity is unused
    [[maybe_unused]] double phi { 1.61803 }; // Don't complain if phi is unused

    std::cout << pi << '\n';
    std::cout << phi << '\n';

    // The compiler will no longer warn about gravity not being used

    return 0;
}
```
## iostream 简介：cout、cin 和 endl
### std::cout是缓冲的
`std::cout`输出通常不会立即发送到控制台。相反，请求的输出“排队”，并存储在一块专门用于收集此类请求的内存区域中（称为缓冲区）。定期地，缓冲区会刷新，这意味着缓冲区中收集的所有数据都会传输到其目的地（在本例中为控制台）。
### `std::endl`与`\n`
使用 std::endl 通常效率低下，因为它实际上做了两件事：它输出一个换行符（将光标移动到控制台的下一行），并且它会刷新缓冲区（这很慢）。如果我们输出多行以 std::endl 结尾的文本，我们将得到多次刷新，这很慢并且可能不必要。

将文本输出到控制台时，我们通常不需要自己显式刷新缓冲区。C++ 的输出系统设计为定期自刷新，让它自行刷新既简单又高效。

要在不刷新输出缓冲区的情况下输出换行符，我们使用 \n（在单引号或双引号内），它是一个特殊符号，编译器将其解释为换行符。\n 将光标移动到控制台的下一行，而不会导致刷新，因此它通常会表现得更好。\n 也更简洁，并且可以嵌入到现有的双引号文本中。

这是一个以几种不同方式使用 \n 的示例

```cpp
#include <iostream> // for std::cout

int main()
{
    int x{ 5 };
    std::cout << "x is equal to: " << x << '\n'; // single quoted (by itself) (conventional)
    std::cout << "Yep." << "\n";                 // double quoted (by itself) (unconventional but okay)
    std::cout << "And that's all, folks!\n";     // between double quotes in existing text (conventional)
    return 0;
}
```
## 函数和文件
### 函数定义
```cpp
returnType functionName() // This is the function header (tells the compiler about the existence of the function)
{
    // This is the function body (tells the compiler what the function does)
}
```
### 函数参数和实参简介
#### 参数和实参如何协同工作
当调用函数时，函数的所有参数都作为变量创建，并且每个实参的值都被**复制**到匹配的参数中（使用复制初始化）。这个过程称为**按值传递**。利用按值传递的函数参数称为**值参数**。
```cpp
#include <iostream>

// This function has two integer parameters, one named x, and one named y
// The values of x and y are passed in by the caller
void printValues(int x, int y)
{
    std::cout << x << '\n';
    std::cout << y << '\n';
}

int main()
{
    printValues(6, 7); // This function call has two arguments, 6 and 7

    return 0;
}
```
#### 未引用参数和未命名参数
在函数定义中，函数参数的名称是可选的。因此，在函数参数需要存在但未在函数体中使用的情况下，您可以直接省略名称。没有名称的参数称为**未命名参数**。
```cpp
void doSomething(int) // ok: unnamed parameter will not generate warning
{
}
```
> 作者注(bushi)
> 
> 你可能想知道我们为什么要写一个带有未使用参数值的函数。这种情况最常发生在以下类似情况中：
假设我们有一个带有一个参数的函数。后来，函数以某种方式更新，并且不再需要该参数的值。如果现在未使用的函数参数被简单地移除，那么对该函数的每个现有调用都将中断（因为函数调用将提供比函数可接受的更多参数）。这将要求我们找到对该函数的每个调用并移除不需要的参数。这可能需要大量工作（并需要大量重新测试）。甚至可能无法做到（在我们无法控制所有调用函数的代码的情况下）。因此，我们可以保留参数不变，只是让它什么都不做。

### 多代码文件时的前向声明
add.cpp
```cpp
int add(int x, int y)
{
    return x + y;
}
```
main.cpp
```cpp
#include <iostream>

int add(int x, int y); // needed so main.cpp knows that add() is a function defined elsewhere

int main()
{
    std::cout << "The sum of 3 and 4 is: " << add(3, 4) << '\n'; // compile error
    return 0;
}
```
现在，当编译器编译 `main.cpp` 时，它会知道标识符 `add` 是什么。链接器会将 `main.cpp` 中对 `add` 的函数调用连接到 `add.cpp` 中函数 `add` 的定义。
使用这种方法，我们可以让文件访问存在于另一个文件中的函数。

### 全局命名空间
```cpp
#include <iostream> // 导入声明

// 以下所有语句都是全局命名空间的一部分

void foo();    // 可以: 函数前置声明
int x;         // 不推荐: 非常量的全局变量定义（没有初始化）
int y { 5 };   // 不推荐: 非常量的全局变量定义（初始化）
x = 5;         // 编译错误❌: 不允许在命名空间执行语句

int main()     // 可以: 函数定义
{
    return 0;
}

void goo();    // 可以: 函数前置声明
```

#### 使用命名空间 std（以及为何避免使用它）
using 指令允许我们访问命名空间中的名称而无需使用命名空间前缀。因此，在上面的示例中，当编译器确定标识符 cout 是什么时，它将与 std::cout 匹配，由于 using 指令，std::cout 可以直接作为 cout 访问。

许多教材、教程甚至一些 IDE 都推荐或在程序顶部使用 using-directive。但是，以这种方式使用，这是一种不好的做法，强烈不鼓励。

考虑以下程序
```cpp
#include <iostream> // imports the declaration of std::cout into the global scope

using namespace std; // makes std::cout accessible as "cout"

int cout() // defines our own "cout" function in the global namespace
{
    return 5;
}

int main()
{
    cout << "Hello, world!"; // Compile error!  Which cout do we want here?  The one in the std namespace or the one we defined above?

    return 0;
}
```

上面的程序无法编译，因为编译器现在无法区分我们要的是我们定义的 cout 函数，还是 std::cout。

以这种方式使用 using-directive 时，我们定义的任何标识符都可能与 std 命名空间中任何同名标识符冲突。更糟糕的是，虽然标识符名称今天可能不冲突，但它可能与未来语言修订版中添加到 std 命名空间的新标识符冲突。这正是最初将标准库中的所有标识符移到 std 命名空间中的全部目的！

### 头文件
通常，头文件用于将一批相关的前向声明传播到代码文件中。
#### 头文件保护

```cpp
#ifndef ADD_H
#define ADD_H

int add(int x, int y);

#endif
```
或者
```cpp
#pragma once 
int add(int x, int y);

```
## 常量和字符串
### Constexpr变量
们可以借助编译器的帮助来确保我们在需要时获得编译时常量变量。为此，我们在变量声明中Instead of const，我们使用 constexpr 关键字（“constant expression”的缩写）。**constexpr** 变量始终是编译时常量。因此，constexpr 变量必须用常量表达式初始化，否则将导致编译错误。

例如
```cpp
#include <iostream>

// The return value of a non-constexpr function is not constexpr
int five()
{
    return 5;
}

int main()
{
    constexpr double gravity { 9.8 }; // ok: 9.8 is a constant expression
    constexpr int sum { 4 + 5 };      // ok: 4 + 5 is a constant expression
    constexpr int something { sum };  // ok: sum is a constant expression

    std::cout << "Enter your age: ";
    int age{};
    std::cin >> age;

    constexpr int myAge { age };      // compile error: age is not a constant expression
    constexpr int f { five() };       // compile error: return value of five() is not constexpr

    return 0;
}
```
### std::string和std::string_view
```cpp
int age;
std::string name;

std::cin >> age;          // 读取数字后，换行符留在缓冲区
std::cin >> std::ws;      // 跳过换行符/空格
std::getline(std::cin, name); // 正确读取完整行
```
#### 使用std::string_view
```cpp

```

## 作用域、生命周期和链接
### 定义您自己的命名空间
命名空间的语法如下
```cpp
namespace NamespaceIdentifier
{
    // content of namespace here
}
```
以大写字母开头的命名空间名称的一些原因：
- 它有助于防止与系统提供或库提供的其他小写名称发生命名冲突。
### 内部链接
#### 具有内部链接的全局变量
```cpp
#include <iostream>

static int g_x{}; // 不能被外部文件访问

const int g_y{ 1 }; // 常量默认具有内部链接
constexpr int g_z{ 2 }; // constexpr globals have internal linkage by default

int main()
{
    std::cout << g_x << ' ' << g_y << ' ' << g_z << '\n';
    return 0;
}
```

### 外部链接和变量前向声明
#### 函数默认具有外部链接
#### 具有外部链接的全局变量
具有外部链接的全局变量有时称为外部变量。为了使全局变量外部化（从而可由其他文件访问），我们可以使用extern关键字来实现
```cpp
int g_x { 2 }; // 非常量的全局变量默认具有外部链接non-constant globals are external by default (no need to use extern)

extern const int g_y { 3 }; // 常量全局变量可以通过extern暴露
extern constexpr int g_z { 3 }; // constexpr globals can be defined as extern, making them external (but this is pretty useless, see the warning in the next section)

int main()
{
    return 0;
}
```
#### 通过 extern 关键字进行变量前向声明
要实际使用在另一个文件中定义的外部全局变量，您还必须在任何其他希望使用该变量的文件中放置该全局变量的前向声明。对于变量，创建前向声明也通过extern关键字（**不带初始化值**）完成。
```cpp
#include <iostream>
// 注意不要就行初始化，在其他地方初始化
extern int g_x;       // this extern is a forward declaration of a variable named g_x that is defined somewhere else
extern const int g_y; // this extern is a forward declaration of a const variable named g_y that is defined somewhere else

int main()
{
    std::cout << g_x << ' ' << g_y << '\n'; // prints 2 3

    return 0;
}
```

### 内联函数和变量
#### 内联展开
```cpp
#include <iostream>

inline int min(int x, int y) // inline keyword means this function is an inline function
{
    return (x < y) ? x : y;
}

int main()
{
    std::cout << min(5, 6) << '\n';
    std::cout << min(3, 2) << '\n';
    return 0;
}
```
#### 内联变量`c17`
```cpp
#ifndef CONSTANTS_H
#define CONSTANTS_H

// define your own namespace to hold constants
namespace constants
{
    inline constexpr double pi { 3.14159 }; // note: now inline constexpr
    inline constexpr double avogadro { 6.0221413e23 };
    inline constexpr double myGravity { 9.2 }; // m/s^2 -- gravity is light on this planet
    // ... other related constants
}
#endif
``` 
### 静态局部变量
```cpp
#include <iostream>

void incrementAndPrint()
{
    static int s_value{ 1 }; // static duration via static keyword.  This initializer is only executed once.
    ++s_value;
    std::cout << s_value << '\n';
} // s_value is not destroyed here, but becomes inaccessible because it goes out of scope

int main()
{
    incrementAndPrint();
    incrementAndPrint();
    incrementAndPrint();

    return 0;
}
```
### using 声明和 using 指令
你可能在许多教科书和教程中见过这个程序
```cpp
#include <iostream>

using namespace std;

int main()
{
    cout << "Hello world!\n";

    return 0;
}
```
####  using 声明
```cpp
#include <iostream>

int main()
{
   using std::cout; // this using declaration tells the compiler that cout should resolve to std::cout
   cout << "Hello world!\n"; // so no std:: prefix is needed here!

   return 0;
} // the using declaration expires at the end of the current scope
```
使用声明从声明点到其声明所在作用域的末尾都有效。
## 控制流
### 随机数使用

## 错误检测和处理



