---
title: "C++笔记"
date: '2026-03-22T10:38:58+08:00'
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

## 左值引用
### const的左值引用
非const的引用不可以绑定到const变量上
```cpp
int main()
{
    const int x { 5 }; // x is a non-modifiable (const) lvalue
    int& ref { x }; // error: ref can not bind to non-modifiable lvalue

    return 0;
}
```
const 的左值引用也可以绑定到可修改的左值
```cpp
#include <iostream>
int main()
{
    int x { 5 };          
    const int& ref { x }; // 不可以通过ref修改x
    return 0;
}
```
绑定到临时对象的常量引用延长了临时对象的生命周期
```cpp
#include <iostream>

int main()
{
    const int& ref { 5 }; 
    // int& ref { 5 } 是不合法的，因为非const左值引用只能绑定到左值

    std::cout << ref << '\n'; // Therefore, we can safely use it here

    return 0;
} // Both ref and the temporary object die here
```
### 通过左值引用传递
一些类类型复制成本高昂，特别是当我们几乎会立即销毁这些副本时。例如：
```cpp
#include <iostream>
#include <string>

void printValue(std::string y)
{
    std::cout << y << '\n';
} // y在这里被销毁

int main()
{
    std::string x { "Hello, world!" };
    printValue(x); // 从x拷贝到了函数参数y
    return 0;
}
```
#### 按引用传递
调用函数时避免对参数进行昂贵复制的一种方法是使用按引用传递而不是按值传递。
```cpp
#include <iostream>
#include <string>

void printValue(std::string& y) // type changed to std::string&
{
    std::cout << y << '\n';
} // y is destroyed here

int main()
{
    std::string x { "Hello, world!" };

    printValue(x); // 传递了引用

    return 0;
}
```
### 按 const 左值引用传递
与非 const 引用（**只能绑定到可修改的左值**）不同，**const 引用可以绑定到可修改的左值、不可修改的左值和右值**。因此，如果我们将引用参数设为 const，它就能够绑定到任何类型的实参。
```cpp
#include <iostream>

void printRef(const int& y) // y is a const reference
{
    std::cout << y << '\n';
}

int main()
{
    int x { 5 };
    printRef(x);   // 可修改的左值
    const int z { 5 };
    printRef(z);   // 不可修改的左值
    printRef(5);   // 右值

    return 0;
}
```
## 按引用返回和按地址返回
### 按引用返回

```cpp
#include <iostream>
#include <string>

const std::string& getProgramName() // returns a const reference
{
    static const std::string s_programName { "Calculator" }; // 声明了一个静态的常量，以延长声明周期

    return s_programName;
}

int main()
{
    std::cout << "This program is named " << getProgramName();
    return 0;
}
```
### 按引用返回的对象在函数返回后必须存在
```cpp
#include <iostream>
#include <string>

const std::string& getProgramName()
{
    const std::string programName { "Calculator" }; // 非静态，到函数末尾会被销毁
    return programName;
}

int main()
{
    std::cout << "This program is named " << getProgramName(); // 未定义的行为

    return 0;
}
```
### 生命周期延长不跨函数边界工作
```cpp
#include <iostream>

const int& returnByConstReference()
{
    return 5; // returns const reference to temporary object
}

int main()
{
    const int& ref { returnByConstReference() };

    std::cout << ref; // undefined behavior

    return 0;
}
```
在上述程序中，returnByConstReference() 返回一个整数字面量，但函数的返回类型是 const int&。这导致创建并返回一个绑定到持有值 5 的临时对象的临时引用。此返回的引用被复制到调用者范围内的临时引用中。然后临时对象超出作用域，使调用者范围内的临时引用悬空。

这是一个不那么明显的、同样不起作用的例子。
```cpp
#include <iostream>

const int& returnByConstReference(const int& ref)
{
    return ref;
}

int main()
{
    // case 1: direct binding
    const int& ref1 { 5 }; // extends lifetime
    std::cout << ref1 << '\n'; // okay

    // case 2: indirect binding
    const int& ref2 { returnByConstReference(5) }; // binds to dangling reference
    std::cout << ref2 << '\n'; // undefined behavior

    return 0;
}
```
在情况 2 中，创建了一个临时对象来保存值 5，函数参数 ref 绑定到该对象。函数只是将此引用返回给调用者，然后调用者使用该引用初始化 ref2。因为这不是与临时对象的直接绑定（因为引用是通过函数跳过的），所以生命周期延长不适用。这导致 ref2 悬空，其后续使用是未定义的行为

### 不要按引用返回非 const 静态局部变量
```cpp
#include <iostream>
#include <string>

const int& getNextId()
{
    static int s_x{ 0 }; // note: variable is non-const
    ++s_x; // generate the next id
    return s_x; // and return a reference to it
}

int main()
{
    const int& id1 { getNextId() }; // id1 是一个引用
    const int& id2 { getNextId() }; // id2 是一个引用

    std::cout << id1 << id2 << '\n';

    return 0;
}
```
发生这种情况是因为 id1 和 id2 引用同一个对象（静态变量 s_x），所以当任何东西（例如 getNextId()）修改该值时，所有引用现在都访问修改后的值。

上述示例可以通过将 id1 和 id2 设为普通变量（而不是引用）来修复，以便它们保存返回值的副本而不是对 s_x 的引用。


### 返回 std::optional

```cpp
#include <iostream>
#include <optional> // for std::optional (C++17)

// Our function now optionally returns an int value
std::optional<int> doIntDivision(int x, int y)
{
    if (y == 0)
        return {}; // or return std::nullopt
    return x / y;
}

int main()
{
    std::optional<int> result1 { doIntDivision(20, 5) };
    if (result1) // if the function returned a value
        std::cout << "Result 1: " << *result1 << '\n'; // get the value
    else
        std::cout << "Result 1: failed\n";

    std::optional<int> result2 { doIntDivision(5, 0) };

    if (result2)
        std::cout << "Result 2: " << *result2 << '\n';
    else
        std::cout << "Result 2: failed\n";

    return 0;
}
```


使用 `std::optional` 非常简单。我们可以使用有值或无值的方式构造一个 `std::optional<T>`
```cpp
std::optional<int> o1 { 5 };            // initialize with a value
std::optional<int> o2 {};               // initialize with no value
std::optional<int> o3 { std::nullopt }; // initialize with no value
```
要查看 `std::optional` 是否有值，我们可以选择以下之一
```cpp
if (o1.has_value()) // call has_value() to check if o1 has a value
if (o2)             // use implicit conversion to bool to check if o2 has a value
```
要从 `std::optional` 获取值，我们可以选择以下之一
```cpp
std::cout << *o1;             // dereference to get value stored in o1 (undefined behavior if o1 does not have a value)
std::cout << o2.value();      // call value() to get value stored in o2 (throws std::bad_optional_access exception if o2 does not have a value)
std::cout << o3.value_or(42); // call value_or() to get value stored in o3 (or value `42` if o3 doesn't have a value)
```

## 类
### 成员初始化列表与默认成员初始化器
成员可以通过几种不同的方式初始化
- 如果成员在成员初始化列表中列出，则使用该初始化值
- 否则，如果成员具有默认成员初始化器，则使用该初始化值
- 否则，成员将进行默认初始化。
这意味着如果一个成员既有默认成员初始化器又在构造函数的成员初始化列表中列出，则成员初始化列表的值优先。

这是一个显示所有三种初始化方法的示例
```cpp
#include <iostream>

class Foo
{
private:
    int m_x {};    // default member initializer (will be ignored)
    int m_y { 2 }; // default member initializer (will be used)
    int m_z;      // no initializer

public:
    Foo(int x)
        : m_x { x } // member initializer list
    {
        std::cout << "Foo constructed\n";
    }

    void print() const
    {
        std::cout << "Foo(" << m_x << ", " << m_y << ", " << m_z << ")\n";
    }
};

int main()
{
    Foo foo { 6 };
    foo.print();

    return 0;
}
```
这是正在发生的事情。当 `foo` 被构造时，只有 `m_x` 出现在成员初始化列表中，所以 `m_x` 首先被初始化为 6。`m_y` 不在成员初始化列表中，但它有一个默认成员初始化器，所以它被初始化为 2。`m_z` 既不在成员初始化列表中，也没有默认成员初始化器，所以它被默认初始化（对于基本类型，这意味着它未初始化）。因此，当我们打印`m_z` 的值时，我们得到未定义行为。
### 默认构造函数和默认实参
```cpp
#include <iostream>

class Foo
{
public:
    Foo() // default constructor
    {
        std::cout << "Foo default constructed\n";
    }
};

int main()
{
    Foo foo{}; // No initialization values, calls Foo's default constructor

    return 0;
}
```
#### 类类型的值初始化与默认初始化
如果一个类类型有默认构造函数，则值初始化和默认初始化都会调用默认构造函数。因此，对于像上面示例中的 Foo 类这样的类，以下是基本等效的
```cpp
Foo foo{}; // 值初始化（优先选择）, calls Foo() default constructor
Foo foo2;  // 默认初始化, calls Foo() default constructor
```



## C++移动语义
左值：有名字，可以取地址
```cpp
int x = 5;
x = 10;
```
`x`是左值
___
右值：临时对象
```cpp
5
a + b
std::string("hello")
```
___
移动语义依赖**右值引用**
```cpp
std::string&& s = std::string("hello"); // 将右值绑定到右值引用
////////////////////
std::string a{"Hello"};
std::string&& sb = std::move(a); // 将左值绑定到右值引用
```
___
移动构造函数
```cpp
class String
{
public:

    String(String&& other) noexcept
    {
        data = other.data;
        other.data = nullptr;
    }

private:
    char* data;
};
```
移动构造函数可以移动资源，例如：
```cpp
std::string a = "hello";
std::string b = std::move(a);
///
std::string b(std::move(a)); // 直接初始化
/// 
std::string b{std::move(a)}; // 直接列表初始化
```



