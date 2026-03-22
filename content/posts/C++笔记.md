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


