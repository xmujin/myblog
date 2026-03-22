---
title: "C++算法训练"
date: '2026-03-22T14:40:46+08:00'
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
description: "来自于www.codewars.com"
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

## Calculate average 计算平均值
### 描述
Write a function which calculates the average of the numbers in a given array.

Note: Empty arrays should return 0.
### 我的解决方案
```cpp
#include <vector>

double calc_average(const std::vector<double>& values)
{
    if(values.empty())
      return 0.0;
    double sum{};
    for(const auto& v : values)
    {
      sum += v;
    }
    return sum / values.size(); 
}
```
### 另外的解决方案
```cpp
#include <vector>
#include <numeric>
double calc_average(const std::vector<double>& values)
{
    if(values.empty())
      return 0.0;
    double res = std::accumulate(vec.begin(), vec.end(), 0) / values.size();
    return res
}
```
```cpp
double calc_average(const std::vector<double>& values)
{
    return values.empty() ? 0.0 : accumulate(values.begin(), values.end(), 0.0) / values.size();
}
```
### 要点与总结
vector可以通过`empty()`函数判断是否为空(或使用`size()`方法与0比较)

迭代vector有一下几种方式：

```cpp
    std::vector<int> vec = {1, 2, 3, 4, 5};

    // 使用下标迭代
    for (size_t i = 0; i < vec.size(); ++i) {
        std::cout << vec[i] << " ";
    }

    // 使用范围for
    for (int value : vec) {
        std::cout << value << " ";
    }
    // 使用迭代器
    for (auto it = vec.begin(); it != vec.end(); ++it) {
        std::cout << *it << " ";
    }
    // 使用常量迭代器
    for (auto it = vec.cbegin(); it != vec.cend(); ++it) {
        std::cout << *it << " ";
    }
    // 使用 std::for_each 和 Lambda 表达式
    std::for_each(vec.begin(), vec.end(), [](int value) {
        std::cout << value << " ";
    });


```

## String ends with?
### 描述
Complete the solution so that it returns true if the first argument(string) passed in ends with the 2nd argument (also a string).
如果传入的第二个字符串是第一个字符串的结尾，返回true。

Examples:
```cpp
Inputs: "abc", "bc"
Output: true

Inputs: "abc", "d"
Output: false
```

### 我的解决方案
```cpp
#include <string>
bool solution(std::string const &str, std::string const &ending) {
    if(ending.empty())
      return true;
    if(ending.length() > str.length())
      return false;
    return str.compare(str.length() - ending.length(), ending.length(), ending) == 0;
}
```

### 另外的解决方案

```cpp
#include <string>

bool solution(const std::string& str, const std::string& ending) {
  return str.size() >= ending.size() && str.compare(str.size() - ending.size(), std::string::npos, ending) == 0;
}
```
### 要点与总结
通过std::string::compare进行字符串比较。
#### compare函数的用法
```cpp
string str = "apple";
int result = str.compare("banana");
// result < 0: "apple" < "banana"
// result = 0: 相等
// result > 0: 大于

// 比较子串 这里第一个参数表示比较开始的位置，第二个参数
str.compare(str.length() - ending.length(), ending.length(), ending)
// 在比较后缀时，这种与其一致（std::string::npos表示一直匹配到str的末尾）
str.compare(str.size() - ending.size(), std::string::npos, ending)

// 比较子串和子串（用得少）
s.compare(pos, count, str, pos2, count2); // 子串 vs 子串
```

## Convert string to camel case转为驼峰命名法
### 描述
Complete the method/function so that it converts dash/underscore delimited words into camel casing. The first word within the output should be capitalized only if the original word was capitalized (known as Upper Camel Case, also often referred to as Pascal case). The next words should be always capitalized.
### 我的解决方案
```cpp
#include <string>
#include <cctype>  // toupper
std::string to_camel_case(std::string text) {
  if(text.empty())
    return "";
  while(text.find('-') != std::string::npos)
  {
    int pos = text.find('-');
    text[pos + 1] = std::toupper(text[pos + 1]);  // "heLlo"
    text.erase(pos, 1);
    
  }
  while(text.find('_') != std::string::npos)
  {
    int pos = text.find('_');
    text[pos + 1] = std::toupper(text[pos + 1]);  // "heLlo"
    text.erase(pos, 1);
    
  }
  return text;
}
```
### 另外的解决方案
```cpp
#include <string>


std::string to_camel_case(std::string text)
{
  for(int i = 0; i < text.size(); ++i)
  {
    if(text[i] == '-' || text[i] == '_') // 直接找到分隔符，先删除，然后将其位置处改为大写
    {
      text.erase(i,1);
      text[i] = toupper(text[i]);
    }
  }
  return text;
}
```
### 要点与总结
通过`erase`函数来删除string中的元素，并通过`toupper`来将字母转为大写。

## Reverse words 反转字符串中的单词
### 描述
Complete the function that accepts a string parameter, and reverses each word in the string. All spaces in the string should be retained.
Examples
```cpp
"This is an example!" ==> "sihT si na !elpmaxe"
"double  spaces"      ==> "elbuod  secaps"
```
### 我的解决方案
```cpp
#include <string>

std::string reverse_words(std::string str)
{
  int start = 0;
  for(int i = 0; i < str.length(); i++) {
    if(str[i] == ' ') {
      std::reverse(str.begin() + start, str.begin() + i);
      while(str[i] == ' ')
        i++;
      start = i;
    }
  }
  std::reverse(str.begin() + start, str.end());
  return str;
}
```
### 另外的解决方案

```cpp
#include <iostream>
#include <string>
std::string reverse_words(std::string str)
{
    auto begin = str.begin();
    for (auto it = str.begin(); it != str.end(); ++it)
    {
        if (*it == ' ') 
        {
            auto end = it;
            std::reverse(begin, end);
            begin = it + 1; // 这一步将begin放到end后面，用于跳过多余的空格
        }
    }
    std::reverse(begin, str.end());
    return str;
}
```
### 要点与总结
通过函数`std::reverse`翻转字符串


## Disemvowel Trolls 去词喷子
### 描述
Trolls are attacking your comment section!

A common way to deal with this situation is to remove all of the vowels from the trolls' comments, neutralizing the threat.

Your task is to write a function that takes a string and return a new string with all vowels removed.

For example, the string "This website is for losers LOL!" would become "Ths wbst s fr lsrs LL!".

Note: for this kata y isn't considered a vowel.

喷子正在攻击你的评论区！
解决这种情况的常见方法是删除喷子评论中的所有元音，从而消除威胁。
你的任务是写一个函数，取字符串返回一个去除所有元音的新字符串。
例如，字符串“This website is for losers LOL！”会变成“Ths wbst s fr lsrs LL！”。
注意：在这种情况下，`y`不被视为元音。

### 我的解决方案
```cpp
#include <string>

std::string disemvowel(const std::string& str) {
    std::string result;
    std::string vowels = "aeiouAEIOU";
    
    for (char c : str) {
        if (vowels.find(c) == std::string::npos) {
            result += c;  // 不是元音就保留
        }
    }
    return result;
}

```
### 另外的解决方案
```cpp
# include <string>
# include <regex>
std::string disemvowel(std::string str)
{
    // 创建正则表达式，匹配任意元音字母（大小写）
    std::regex vowels("[aeiouAEIOU]");
    // 将匹配到的所有元音替换为空字符串（即删除）
    return std::regex_replace(str, vowels, "");
}
```
### 要点与总结
可以通过正则表达式解决。

## Equal Sides Of An Array
### 描述
You are going to be given an array of integers. Your job is to take that array and find an index N where the sum of the integers to the left of N is equal to the sum of the integers to the right of N.
你会得到一个整数数组。你的任务是取这个数组，找到一个索引 N，使得 N 左侧整数的和等于 N 右侧整数的和。

If there is no index that would make this happen, return -1.
如果没有能实现这一点的索引，返回 -1。

For example:  例如：
Let's say you are given the array {1,2,3,4,3,2,1}:
假设你给定数组 {1,2,3,4,3,2,1}：
Your function will return the index 3, because the sum of left side of the index ({1,2,3}) and the sum of the right side of the index ({3,2,1}) both equal 6.
你的函数会返回索引 3，因为索引左边（{1,2,3}）和右边的和（{3,2,1}）都等于 6。

Let's look at another one.
我们再看看另一个。
You are given the array {1,100,50,-51,1,1}:
给出数组 {1,100,50，-51,1,1}：
Your function will return the index 1, because the sum of left side of the index ({1}) and the sum of the right side of the index ({50,-51,1,1}) both equal 1.
你的函数会返回索引 1，因为索引左边（{1}）和右边的和（{50，-51,1,1}）都等于 1。

Last one:  最后一个：
You are given the array {20,10,-80,10,10,15,35}
你会得到数组 {20,10，-80,10,10,15,35}
At index 0 the left side is {}
在索引 0 处，左边为 {}
The right side is {10,-80,10,10,15,35}
右边为 {10，-80,10,10,15,35}
They both are equal to 0 when added. (Empty arrays are equal to 0 in this problem)
它们相加后都等于 0。（本问题中空数组等于 0）
Index 0 is the place where the left side and right side are equal.
索引0是左右两边相等的位置。

Note: Please remember that in most languages the index of an array starts at 0.
注意：请记住，在大多数语言中，数组的索引从0开始。

Input  输入
An integer array of length 0 < arr < 1000. The numbers in the array can be any integer positive or negative.
一个长度为 0 < arr < 1000 的整数组。数组中的数字可以是任何整数正数或负数。

Output  输出
The lowest index N where the side to the left of N is equal to the side to the right of N. If you do not find an index that fits these rules, then you will return -1.
最低的指标 N，其中 N 左边等于 N 右边。如果你找不到符合这些规则的索引，那么你将返回 -1。

Note  注释
If you are given an array with multiple answers, return the lowest correct index.
如果给出一个包含多个答案的数组，返回最低的正确索引。
### 我的解决方案
```cpp
#include <vector>
#include <numeric>
using namespace std;

int find_even_index (const vector <int> numbers) {
  int left{};
  int right{};
  for(auto it = numbers.begin(); it != numbers.end(); ++it) {
    left = accumulate(numbers.begin(), it, 0);
    right = accumulate(it + 1, numbers.end(), 0);
    if(left == right)
      return std::distance(numbers.begin(), it); // 获取迭代器所指向元素的索引
  }
  return -1;
}
```
### 另外的解决方案
```cpp
#include <vector>
#include <numeric>
using namespace std;

int find_even_index (const vector <int> numbers) {
  for (int index = 0; index < numbers.size(); index++)
  {
    int left_sum = std::accumulate(numbers.begin(), numbers.begin() + index, 0);
    int right_sum = std::accumulate(numbers.begin() + index + 1, numbers.end(), 0);
    if (left_sum == right_sum)
      return index;
  }
  return -1;
}
```
### 要点与总结


## Calculating with Functions函数计算
### 描述
This time we want to write calculations using functions and get the results. Let's have a look at some examples:
这次我们想用函数写计算并得到结果。让我们看看一些例子：

seven(times(five()));    //  must return 35
four(plus(nine()));      //  must return 13
eight(minus(three()));   //  must return 5
six(divided_by(two()));  //  must return 3
Requirements:  要求：

There must be a function for each number from 0 ("zero") to 9 ("nine")
每个数字必须有一个函数，范围从0（“零”）到9（“九”
There must be a function for each of the following mathematical operations: plus, minus, times, divided_by
每个数学运算都必须有一个函数：加、减、乘、divided_by
Each calculation consist of exactly one operation and two numbers
### 我的解决方案
```cpp
#include <functional>


template<int N>
auto make_func() {
  return [](std::function<int(int)> func = nullptr) {
    if(func)
      return func(N);
    return N;
  };
}

auto zero = make_func<0>();
auto one = make_func<1>();
auto two = make_func<2>();
auto three = make_func<3>();
auto four = make_func<4>();
auto five = make_func<5>();
auto six = make_func<6>();
auto seven = make_func<7>();
auto eight = make_func<8>();
auto nine = make_func<9>();

auto plus(int a) {
  return [a](int b) {
    return a + b;
  };
}
auto minus(int a) {
  return [a](int b) {
    return b - a;
  };
}
auto times(int a) {
  return [a](int b) {
    return a * b;
  };
}
auto divided_by(int a) {
  return [a](int b) {
    return b / a;
  };
}
```
### 另外的解决方案
```cpp
#include <functional>
using op = std::function<int(int)>; // 类型别名语法
int id(int n) { return n; }

int zero (op func = id) { return func(0); }
int one  (op func = id) { return func(1); }
int two  (op func = id) { return func(2); }
int three(op func = id) { return func(3); }
int four (op func = id) { return func(4); }
int five (op func = id) { return func(5); }
int six  (op func = id) { return func(6); }
int seven(op func = id) { return func(7); }
int eight(op func = id) { return func(8); }
int nine (op func = id) { return func(9); }

// 这里的“=”表示按值捕获
op plus      (int n) { return [=](int m) { return m + n; }; } 
op minus     (int n) { return [=](int m) { return m - n; }; }
op times     (int n) { return [=](int m) { return m * n; }; }
op divided_by(int n) { return [=](int m) { return m / n; }; }
```
### 要点与总结
主要是通过lamda函数完成计算。