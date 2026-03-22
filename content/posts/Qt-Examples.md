---
title: "Qt Examples"
date: '2026-03-20T16:18:01+08:00'
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

## 计算器示例注释
`caculator.h`
```cpp
#pragma once
#include <QWidget>

QT_BEGIN_NAMESPACE
class QLineEdit;
QT_END_NAMESPACE
class Button;

//! [0]
class Calculator : public QWidget
{
    Q_OBJECT

public:
    Calculator(QWidget *parent = nullptr);

private slots:
    void digitClicked();
    void unaryOperatorClicked();
    void additiveOperatorClicked();
    void multiplicativeOperatorClicked();
    void equalClicked();
    void pointClicked();
    void changeSignClicked();
    void backspaceClicked();
    void clear();

    // 将sumSoFar 重置为零
    void clearAll();
    // MC
    void clearMemory();
    // MR
    void readMemory();

    // MS
    void setMemory();
    // M+
    void addToMemory();
//! [0]

//! [1]
private:
//! [1] //! [2]
    template<typename PointerToMemberFunction>
    Button *createButton(const QString &text, const PointerToMemberFunction &member);
    void abortOperation();


    bool calculate(double rightOperand, const QString &pendingOperator);

    // 含存储在计算器内存中的数值（使用MS,M+, 或MC ）。
    double sumInMemory;

    // 存储当前累积的数值。当用户点击= 时，sumSoFar 将被重新计算并显示在显示屏上。
    double sumSoFar;

    // 在进行乘除运算时存储临时值。
    double factorSoFar;

    // 存储用户最后点击的加法运算符?
    QString pendingAdditiveOperator;

    // 存储用户最后点击的乘法运算符?
    QString pendingMultiplicativeOperator;

    // 当计算器等待用户输入操作数时，true 。
    bool waitingForOperand;



    QLineEdit *display;


    enum { NumDigitButtons = 10 };
    Button *digitButtons[NumDigitButtons];
};
```
`caculator.cpp`
```cpp
#include "calculator.h"
#include "button.h"

#include <QGridLayout>
#include <QLineEdit>
#include <QtMath>

//! [0]
Calculator::Calculator(QWidget *parent)
    : QWidget(parent), sumInMemory(0.0), sumSoFar(0.0)
    , factorSoFar(0.0), waitingForOperand(true)
{

    // 显示屏初始化
    display = new QLineEdit("0");
    display->setReadOnly(true);
    display->setAlignment(Qt::AlignRight);
    display->setMaxLength(15);

    // 设置字体大小
    QFont font = display->font();
    font.setPointSize(font.pointSize() + 8);
    display->setFont(font);

    // 添加数字按钮（绑定到digitClicked槽）
    for (int i = 0; i < NumDigitButtons; ++i)
        digitButtons[i] = createButton(QString::number(i), &Calculator::digitClicked);


    // 添加功能按钮
    Button *pointButton = createButton(tr("."), &Calculator::pointClicked);
    Button *changeSignButton = createButton(tr("\302\261"), &Calculator::changeSignClicked);

    Button *backspaceButton = createButton(tr("Backspace"), &Calculator::backspaceClicked);
    Button *clearButton = createButton(tr("Clear"), &Calculator::clear);
    Button *clearAllButton = createButton(tr("Clear All"), &Calculator::clearAll);

    Button *clearMemoryButton = createButton(tr("MC"), &Calculator::clearMemory);
    Button *readMemoryButton = createButton(tr("MR"), &Calculator::readMemory);
    Button *setMemoryButton = createButton(tr("MS"), &Calculator::setMemory);
    Button *addToMemoryButton = createButton(tr("M+"), &Calculator::addToMemory);

    Button *divisionButton = createButton(tr("\303\267"), &Calculator::multiplicativeOperatorClicked);
    Button *timesButton = createButton(tr("\303\227"), &Calculator::multiplicativeOperatorClicked);
    Button *minusButton = createButton(tr("-"), &Calculator::additiveOperatorClicked);
    Button *plusButton = createButton(tr("+"), &Calculator::additiveOperatorClicked);

    Button *squareRootButton = createButton(tr("Sqrt"), &Calculator::unaryOperatorClicked);
    Button *powerButton = createButton(tr("x\302\262"), &Calculator::unaryOperatorClicked);
    Button *reciprocalButton = createButton(tr("1/x"), &Calculator::unaryOperatorClicked);
    Button *equalButton = createButton(tr("="), &Calculator::equalClicked);



    // 添加主网格布局
    QGridLayout *mainLayout = new QGridLayout;

    // 设置尺寸约束
    mainLayout->setSizeConstraint(QLayout::SetFixedSize);
    // 显示屏
    mainLayout->addWidget(display, 0, 0, 1, 6);

    // 三个需要跨列的按钮
    mainLayout->addWidget(backspaceButton, 1, 0, 1, 2);
    mainLayout->addWidget(clearButton, 1, 2, 1, 2);
    mainLayout->addWidget(clearAllButton, 1, 4, 1, 2);

    // MC MR MS M+
    mainLayout->addWidget(clearMemoryButton, 2, 0);
    mainLayout->addWidget(readMemoryButton, 3, 0);
    mainLayout->addWidget(setMemoryButton, 4, 0);
    mainLayout->addWidget(addToMemoryButton, 5, 0);

    // 数字按钮布局
    for (int i = 1; i < NumDigitButtons; ++i) {
        int row = ((9 - i) / 3) + 2;
        int column = ((i - 1) % 3) + 1;
        mainLayout->addWidget(digitButtons[i], row, column);
    }
    mainLayout->addWidget(digitButtons[0], 5, 1);


    mainLayout->addWidget(pointButton, 5, 2);
    mainLayout->addWidget(changeSignButton, 5, 3);

    mainLayout->addWidget(divisionButton, 2, 4);
    mainLayout->addWidget(timesButton, 3, 4);
    mainLayout->addWidget(minusButton, 4, 4);
    mainLayout->addWidget(plusButton, 5, 4);

    mainLayout->addWidget(squareRootButton, 2, 5);
    mainLayout->addWidget(powerButton, 3, 5);
    mainLayout->addWidget(reciprocalButton, 4, 5);
    mainLayout->addWidget(equalButton, 5, 5);


    // 设置布局
    setLayout(mainLayout);
    // 设置窗口标题
    setWindowTitle(tr("Calculator"));
}




// 数字点击的槽函数
void Calculator::digitClicked()
{
    Button *clickedButton = qobject_cast<Button *>(sender());
    int digitValue = clickedButton->text().toInt();
    if (display->text() == "0" && digitValue == 0.0)
        return;

    // 当等待新的操作数时
    if (waitingForOperand) {
        display->clear();
        waitingForOperand = false;
    }

    // 将后面的数字拼接到之前的数字文本上
    display->setText(display->text() + QString::number(digitValue));
}



void Calculator::unaryOperatorClicked()
{
    Button *clickedButton = qobject_cast<Button *>(sender());
    QString clickedOperator = clickedButton->text();
    double operand = display->text().toDouble();
    double result = 0.0;

    
    if (clickedOperator == tr("Sqrt")) {
        if (operand < 0.0) {
            abortOperation();
            return;
        }
        // 开根
        result = std::sqrt(operand);
    } else if (clickedOperator == tr("x\302\262")) {
        // 平方
        result = std::pow(operand, 2.0);
    } else if (clickedOperator == tr("1/x")) {
        if (operand == 0.0) {
            abortOperation();
            return;
        }
        // 倒数
        result = 1.0 / operand;
    }

    // 设置结果
    display->setText(QString::number(result));
    waitingForOperand = true;
}



void Calculator::additiveOperatorClicked()
//! [10] //! [11]
{
    Button *clickedButton = qobject_cast<Button *>(sender());
    if (!clickedButton)
      return;
    QString clickedOperator = clickedButton->text();
    double operand = display->text().toDouble();

    if (!pendingMultiplicativeOperator.isEmpty()) {
        if (!calculate(operand, pendingMultiplicativeOperator)) {
            abortOperation();
            return;
        }

        // 显示相乘的结果
        display->setText(QString::number(factorSoFar));
        // 保存相乘的结果
        operand = factorSoFar;
        // 清空乘法结果
        factorSoFar = 0.0;
        // 清空乘法运算符
        pendingMultiplicativeOperator.clear();
    }


    // 有待执行的加法运算符
    if (!pendingAdditiveOperator.isEmpty()) {
        // 计算和
        if (!calculate(operand, pendingAdditiveOperator)) {
            abortOperation();
            return;
        }
        // 显示加起来的和
        display->setText(QString::number(sumSoFar));
    } else {
        sumSoFar = operand;
    }

    // 待执行的加法运算符
    pendingAdditiveOperator = clickedOperator;
    // 等待输入新的操作数
    waitingForOperand = true;
}




void Calculator::multiplicativeOperatorClicked()
{
    Button *clickedButton = qobject_cast<Button *>(sender());
    if (!clickedButton)
      return;
    QString clickedOperator = clickedButton->text();
    double operand = display->text().toDouble();

    // 有待执行的乘法运算符
    if (!pendingMultiplicativeOperator.isEmpty()) {
        if (!calculate(operand, pendingMultiplicativeOperator)) {
            abortOperation();
            return;
        }
        display->setText(QString::number(factorSoFar));
    } else {
        // 将显示屏数字因子放进去
        factorSoFar = operand;
    }

    pendingMultiplicativeOperator = clickedOperator;

    // 等待输入新操作数
    waitingForOperand = true;
}



// 点击等于
void Calculator::equalClicked()
{
    double operand = display->text().toDouble();


    // 先乘除
    if (!pendingMultiplicativeOperator.isEmpty()) {
        if (!calculate(operand, pendingMultiplicativeOperator)) {
            abortOperation();
            return;
        }
        operand = factorSoFar;
        factorSoFar = 0.0;
        pendingMultiplicativeOperator.clear();
    }

    // 后加减
    if (!pendingAdditiveOperator.isEmpty()) {
        if (!calculate(operand, pendingAdditiveOperator)) {
            abortOperation();
            return;
        }
        pendingAdditiveOperator.clear();
    } else {
        sumSoFar = operand;
    }

    display->setText(QString::number(sumSoFar));
    sumSoFar = 0.0;
    waitingForOperand = true;
}




void Calculator::pointClicked()
{
    if (waitingForOperand)
        display->setText("0");
    if (!display->text().contains('.'))
        display->setText(display->text() + tr("."));

    // 配合第一个if。如果先按下了点，表示已经输入了新的操作数
    waitingForOperand = false;
}



// 按下取反按钮
void Calculator::changeSignClicked()
{
    QString text = display->text();
    double value = text.toDouble();

    if (value > 0.0) {
        text.prepend(tr("-"));
    } else if (value < 0.0) {
        text.remove(0, 1);
    }
    display->setText(text);
}




// 退格键按下
void Calculator::backspaceClicked()
{
    if (waitingForOperand)
        return;

    QString text = display->text();

    // 删除末尾的字符
    text.chop(1);
    if (text.isEmpty()) {
        // 删除完了后  设置为0，并重新等待输入操作数
        text = "0";

        // 防止输入新的操作数时出现的前面附加0
        waitingForOperand = true;
    }
    display->setText(text);
}




// 清除操作数
void Calculator::clear()
{
    if (waitingForOperand)
        return;

    display->setText("0");
    waitingForOperand = true;
}




// 清空所有输入记录
void Calculator::clearAll()
{
    sumSoFar = 0.0;
    factorSoFar = 0.0;
    pendingAdditiveOperator.clear();
    pendingMultiplicativeOperator.clear();
    display->setText("0");
    waitingForOperand = true;
}


// MC
void Calculator::clearMemory()
{
    sumInMemory = 0.0;
}



// MR
void Calculator::readMemory()
{
    display->setText(QString::number(sumInMemory));
    waitingForOperand = true;
}

// MS
void Calculator::setMemory()
{
    equalClicked();
    sumInMemory = display->text().toDouble();
}


// M+
void Calculator::addToMemory()
{
    equalClicked();
    sumInMemory += display->text().toDouble();
}


// 创建按钮的模板函数。用member接受槽函数
template<typename PointerToMemberFunction>
Button *Calculator::createButton(const QString &text, const PointerToMemberFunction &member)
{
    Button *button = new Button(text);
    connect(button, &Button::clicked, this, member);
    return button;
}



// 中断输入
void Calculator::abortOperation()
{
    clearAll();
    display->setText(tr("####"));
}

// 计算函数
bool Calculator::calculate(double rightOperand, const QString &pendingOperator)
{
    if (pendingOperator == tr("+")) {
        sumSoFar += rightOperand;
    } else if (pendingOperator == tr("-")) {
        sumSoFar -= rightOperand;
    } else if (pendingOperator == tr("\303\227")) {
        factorSoFar *= rightOperand;
    } else if (pendingOperator == tr("\303\267")) {
        if (rightOperand == 0.0)
            return false;
        factorSoFar /= rightOperand;
    }
    return true;
}
```
`button.h`
```cpp
#include <QToolButton>
class Button : public QToolButton
{
    Q_OBJECT

public:
    explicit Button(const QString &text, QWidget *parent = nullptr);

    QSize sizeHint() const override;
};
```

`button.cpp`
```cpp
#include "button.h"
Button::Button(const QString &text, QWidget *parent)
    : QToolButton(parent)
{
    // 水平扩展  垂直推荐
    setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Preferred);
    setText(text);
}

QSize Button::sizeHint() const
{

    // 根据内容来生成推荐尺寸
    QSize size = QToolButton::sizeHint();
    //高度增加20
    size.rheight() += 20;
    size.rwidth() = qMax(size.width(), size.height());
    return size;
}
```





