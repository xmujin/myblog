---
title: "C++输入输出"
date: '2026-03-14T15:53:32+08:00'
# weight: 1
# aliases: ["/first"]
tags: ["C++"]
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
## 通过cin(istream)读取输入
```cpp
char buf[10]{};
std::cin >> buf; // 如果输入超过10个字符会缓冲区溢出
```
可以使用setw限制输入，以下代码会读取前9个字符，最后一个字符用于\0。
*需要注意的是cin通过>>读取遇到空白字符会结束读取*。
```cpp
#include <iomanip>
char buf[10]{};
std::cin >> std::setw(10) >> buf;
```
### 提取和空白符

```cpp
int main()
{
    char ch{};
    while (std::cin >> ch)
        std::cout << ch;

    return 0;
}
```
当用户输入以下内容时 

`Hello my name is Alex`

提取运算符会跳过空格和换行符。因此，输出是

`HellomynameisAlex`

### 通过get函数读取
```cpp
int main()
{
    char ch{};
    while (std::cin.get(ch))
        std::cout << ch;

    return 0;
}
```
当用户输入以下内容时 

`Hello my name is Alex`

不会跳过空格和换行符。输出是

`Hello my name is Alex`

get() 还有一个字符串版本，它接受要读取的最大字符数
```cpp
int main()
{
    char strBuf[11]{};
    // Read up to 10 characters
    std::cin.get(strBuf, 11);
    std::cout << strBuf << '\n';
    
    // 如果上述读取到换行，则下面读取不到
    // Read up to 10 more characters
    std::cin.get(strBuf, 11);
    std::cout << strBuf << '\n';
    return 0;
}
```
与读取`cin.get(ch)`不同的是，cin.get(strBuf, 11)不能读取换行符
，这可能会导致一些意想不到的结果，但是同样能够读取空格。

### 通过getline函数读取

还有一个名为 getline() 的函数，其工作方式类似于 get(strBuf, 11)，但会提取（并丢弃）分隔符(默认是换行)。
```cpp
int main()
{
    char strBuf[11]{};
    // Read up to 10 characters
    std::cin.getline(strBuf, 11);
    std::cout << strBuf << '\n';

    // Read up to 10 more characters
    std::cin.getline(strBuf, 11);
    std::cout << strBuf << '\n';
    return 0;
}
```

#### std::string 的特殊版本 getline()
有一个特殊版本的 getline() 存在于 istream 类之外，用于读取 std::string 类型的变量。这个特殊版本不是 ostream 或 istream 的成员，并包含在 string 头文件中。以下是其使用示例

```cpp
#include <string>
#include <iostream>

int main()
{
    std::string strBuf{};
    std::getline(std::cin, strBuf);
    std::cout << strBuf << '\n';

    return 0;
}
```

您可能还需要使用一些其他有用的输入函数

ignore() 丢弃流中的第一个字符。

ignore(int nCount) 丢弃前 nCount 个字符。

peek() 允许您从流中读取一个字符，而无需将其从流中删除。

unget() 将最后读取的字符返回到流中，以便下一次调用可以再次读取它。

putback(char ch) 允许您将自己选择的字符放回流中，以便下一次调用读取。

## 通过ostream 和 ios 进行输出
通过setf()函数设置标志。
```cpp
std::cout.setf(std::ios::showpos); // turn on the std::ios::showpos flag
std::cout << 27 << '\n';
```
这会输出
`+27`

要关闭标志，使用 unsetf() 函数
```cpp
std::cout.setf(std::ios::showpos); // turn on the std::ios::showpos flag
std::cout << 27 << '\n';
std::cout.unsetf(std::ios::showpos); // turn off the std::ios::showpos flag
std::cout << 28 << '\n';
```
**标志组**
|组|标志|含义|
|-|-|-|
|std::ios::basefield|std::ios::dec|以十进制打印值（默认）|
|std::ios::basefield|std::ios::dec|以十六进制打印值|
|std::ios::basefield|std::ios::dec|以八进制打印值|

**操纵符**
|操纵符|含义|
|-|-|
|std::dec|以十进制打印值|
|std::hex|以十六进制打印值|
|std::oct|以八进制打印值|

**示例**
```cpp
std::cout << 27 << '\n';

std::cout.setf(std::ios::dec, std::ios::basefield);
std::cout << 27 << '\n';

std::cout.setf(std::ios::oct, std::ios::basefield);
std::cout << 27 << '\n';

std::cout.setf(std::ios::hex, std::ios::basefield);
std::cout << 27 << '\n';

std::cout << std::dec << 27 << '\n';
std::cout << std::oct << 27 << '\n';
std::cout << std::hex << 27 << '\n';
```
输出
```
27
27
33
1b
27
33
1b
```

### 精度、表示法和小数点

|组|标志|含义|
|-|-|-|
|std::ios::floatfield|std::ios::fixed|	浮点数使用十进制表示法|
|std::ios::floatfield|std::ios::scientific|	浮点数使用科学记数法|
|std::ios::floatfield|（无）|	对于位数少的数字使用 fixed，否则使用 scientific|
|std::ios::floatfield|std::ios::showpoint|	始终显示小数点和浮点值的尾随 0|

|操纵符|含义|
|-|-|
|std::fixed|	值使用十进制表示法|
|std::scientific|	值使用科学记数法|
|std::showpoint|	显示小数点和浮点值的尾随 0|
|std::noshowpoint|	不显示小数点和浮点值的尾随 0|
|std::setprecision(int)|	设置浮点数的精度（定义在 iomanip 头文件中）|

|成员函数|含义|
|-|-|
|std::ios_base::precision()|	返回浮点数的当前精度|
|std::ios_base::precision(int)|	设置浮点数的精度并返回旧精度|

**如果使用 fixed 或 scientific 表示法，精度决定了小数部分显示多少位小数。请注意，如果精度小于有效数字的数量，则数字将被四舍五入。**

```cpp
std::cout << std::fixed << '\n';
std::cout << std::setprecision(3) << 123.456 << '\n';
std::cout << std::setprecision(4) << 123.456 << '\n';
std::cout << std::setprecision(5) << 123.456 << '\n';
std::cout << std::setprecision(6) << 123.456 << '\n';
std::cout << std::setprecision(7) << 123.456 << '\n';

std::cout << std::scientific << '\n';
std::cout << std::setprecision(3) << 123.456 << '\n';
std::cout << std::setprecision(4) << 123.456 << '\n';
std::cout << std::setprecision(5) << 123.456 << '\n';
std::cout << std::setprecision(6) << 123.456 << '\n';
std::cout << std::setprecision(7) << 123.456 << '\n';
```
结果
```cpp
123.456
123.4560
123.45600
123.456000
123.4560000

1.235e+002
1.2346e+002
1.23456e+002
1.234560e+002
1.2345600e+002
```
**如果既不使用 fixed 也不使用 scientific，则精度决定应显示多少有效数字。同样，如果精度小于有效数字的数量，则数字将被四舍五入。**
```cpp
std::cout << std::setprecision(3) << 123.456 << '\n';
std::cout << std::setprecision(4) << 123.456 << '\n';
std::cout << std::setprecision(5) << 123.456 << '\n';
std::cout << std::setprecision(6) << 123.456 << '\n';
std::cout << std::setprecision(7) << 123.456 << '\n';
```
结果
```cpp
123
123.5
123.46
123.456
123.456
```
使用 showpoint 操纵符或标志，可以使流写入小数点和尾随零。
```cpp
std::cout << std::showpoint << '\n';
std::cout << std::setprecision(3) << 123.456 << '\n';
std::cout << std::setprecision(4) << 123.456 << '\n';
std::cout << std::setprecision(5) << 123.456 << '\n';
std::cout << std::setprecision(6) << 123.456 << '\n';
std::cout << std::setprecision(7) << 123.456 << '\n';
```
### 宽度、填充字符和对齐方式

|组|标志|含义|
|-|-|-|
|std::ios::adjustfield|	std::ios::internal|	数字的符号左对齐，值右对齐|
|std::ios::adjustfield|	std::ios::left|	符号和值左对齐|
|std::ios::adjustfield|	std::ios::right|	符号和值右对齐（默认）|

|操纵符|含义|
|-|-|
|std::internal|	数字的符号左对齐，值右对齐|
|std::left|	符号和值左对齐|
|std::right|	符号和值右对齐|
|std::setfill(char)|	设置参数作为填充字符（定义在 iomanip 头文件中）|
|std::setw(int)|	将输入和输出的字段宽度设置为参数（定义在 iomanip 头文件中）|

|成员函数|	含义|
|-|-|
|std::basic_ostream::fill()|	返回当前填充字符|
|std::basic_ostream::fill(char)|	设置填充字符并返回旧填充字符|
|std::ios_base::width()	|返回当前字段宽度|
|std::ios_base::width(int)|	设置当前字段宽度并返回旧字段宽度|

为了使用这些格式化器中的任何一个，我们必须首先设置字段宽度。这可以通过 width(int) 成员函数或 setw() 操纵符完成。请注意，右对齐是默认设置。

```cpp
std::cout << -12345 << '\n'; // print default value with no field width
std::cout << std::setw(10) << -12345 << '\n'; // print default with field width
std::cout << std::setw(10) << std::left << -12345 << '\n'; // print left justified
std::cout << std::setw(10) << std::right << -12345 << '\n'; // print right justified
std::cout << std::setw(10) << std::internal << -12345 << '\n'; // print internally justified
```
这会产生结果
```cpp
-12345
    -12345
-12345
    -12345
-    12345
```

**需要注意的是，setw() 和 width() 只影响下一个输出语句。它们不像其他一些标志/操纵符那样是持久的。**

现在，让我们设置一个填充字符并执行相同的示例

```cpp
std::cout.fill('*');
std::cout << -12345 << '\n'; // print default value with no field width
std::cout << std::setw(10) << -12345 << '\n'; // print default with field width
std::cout << std::setw(10) << std::left << -12345 << '\n'; // print left justified
std::cout << std::setw(10) << std::right << -12345 << '\n'; // print right justified
std::cout << std::setw(10) << std::internal << -12345 << '\n'; // print internally justified
```
这会产生输出
```cpp
-12345
****-12345
-12345****
****-12345
-****12345
```
