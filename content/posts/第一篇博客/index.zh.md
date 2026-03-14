---
title: "我的第一篇博客"
date: '2026-03-13T19:54:56+08:00'
# weight: 1
# aliases: ["/first"]
# tags: ["nothing"]
categories: ["日常"]
author: "向洵"
showToc: true
TocOpen: true
draft: false
hidemeta: false
comments: false
description: "服了"
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

## hugo的使用命令
```C++
hugo new posts/文章名字.md # 添加文章
hugo new posts/文章名字/index.md # 用于叶子资源捆绑
hugo server -D # 启动本地服务器
```

```cpp
int main() {
    return 0;
}
```
## Markdown语法
### 标题语法
# 这是一级标题
## 这是二级标题
### 这是三级标题

一级标题可选语法
===============
二级标题可选语法
---------------
### 段落语法
Don't put tabs or spaces in front of your paragraphs.

Keep lines left-aligned like this.
### 换行语法
First line with two spaces after.  
And the next line.
### 强调语法
这是**粗体**
这是*斜体*
### 引用语法
> 我是一个引用
### 列表语法
#### 有序列表
1. 第一项
2. 第二项
3. 第三项
#### 无序列表
- 我
- 还
- 爱
- 她
    - 天
    - 人
    - 合
    - 一

### 代码语法
#### 行内代码
At the command prompt, type `我是行内代码`.
#### 代码块
```
我是代码块
```
### 分隔线语法
***
---
_________________
### 链接语法
链接文本放在中括号内，链接地址放在后面的括号中，链接title可选。

超链接Markdown语法代码：[超链接显示名](超链接地址 "超链接title")

对应的HTML代码：`<a href="超链接地址" title="超链接title">超链接显示名</a>`

### 图片语法
插入图片Markdown语法代码：`![图片alt](图片链接 "图片title")`。

对应的HTML代码：`<img src="图片链接" alt="图片alt" title="图片title">`
![显示不了哦](test.jpg "小可爱哦")

### 表格语法
| Syntax      | Description |
| ----------- | ----------- |
| Header      | Title       |
| Paragraph   | Text        |
### 脚注
Here's a simple footnote,[^1] and here's a longer one.[^2]

[^1]: 不知道写什么.

[^2]: 我其实服了

### 标题编号
### My Great Heading {#custom-id}


[回到markdown第一个语法](#标题语法)

### 删除线
~~世界是平坦的。~~ 我们现在知道世界是圆的。
### 任务列表
- [x] Write the press release
- [ ] Update the website
- [ ] Contact the media
### 使用表情
去露营了！ :tent: 很快回来。

真好笑！ :joy: 😅

[推荐表情网站](https://www.webfx.com/tools/emoji-cheat-sheet/)