---
title: "QtWidgets指南"
date: '2026-03-15T15:22:43+08:00'
# weight: 1
# aliases: ["/first"]
tags: ["Qt"]
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
## 创建一个简易的窗口
```cpp
int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    QWidget w;
    w.resize(800, 600);
    w.setWindowTitle("你好  服了");
    w.show();
    return a.exec();
}
```
## 使用布局

```cpp
int main(int argc, char *argv[])
{
    QApplication app(argc, argv);
    QWidget window;
    QLabel *label = new QLabel(QApplication::translate("windowlayout", "Name:"));
    QLineEdit *lineEdit = new QLineEdit();

    QHBoxLayout *layout = new QHBoxLayout();
    layout->addWidget(label);
    layout->addWidget(lineEdit);
    window.setLayout(layout);
    window.setWindowTitle("草了");
    window.show();
    return app.exec();
}
```
上面在window上设置布局`window.setLayout(layout)`时，两个部件和布局本身都被 "重新父对象化"，成为窗口的子窗口。
## qt嵌套布局

```cpp
#include <QtWidgets>

int main(int argc, char *argv[])
{
    QApplication app(argc, argv);
    QWidget window;

    QLabel *queryLabel = new QLabel(
        QApplication::translate("nestedlayouts", "Query:"));
    QLineEdit *queryEdit = new QLineEdit();
    QTableView *resultView = new QTableView();

    QHBoxLayout *queryLayout = new QHBoxLayout();、
    // 这里将queryLabel和queryEdit加入到布局中，但此时这些组件还没有父组件
    queryLayout->addWidget(queryLabel);
    queryLayout->addWidget(queryEdit);


    QVBoxLayout *mainLayout = new QVBoxLayout();

    // 将布局添加到另外一个布局时，则子布局的父对象是是其父布局
    mainLayout->addLayout(queryLayout);
    mainLayout->addWidget(resultView);

    // 当给一个组件设置布局时，其布局内的所有组件都成为其子组件
    // 但queryLayout的父组件还是mainLayout，这个关系没有发生改变
    // mainLayout由于没有父布局，则其父组件变为了window
    window.setLayout(mainLayout);

    // Set up the model and configure the view...
    window.setWindowTitle(
        QApplication::translate("nestedlayouts", "Nested layouts"));
    window.show();
    return app.exec();
}
```

## Qt元对象系统
元对象系统基于以下三个方面：
1. QObject 类为可以利用元对象系统的对象提供了一个基类。
2. Q_OBJECT 宏用于启用元对象功能，如动态属性、信号和槽。
宏 
3. Meta-Object Compiler(moc) 为每个QObject 子类提供实现元对象功能所需的代码。
