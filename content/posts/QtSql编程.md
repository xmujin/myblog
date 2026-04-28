---
title: "QtSql编程"
date: '2026-04-03T21:49:10+08:00'
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

## SQL编程概述
这些类提供了访问SQL数据库的接口。

|           类名           |                      描述                       |
| :----------------------: | :---------------------------------------------: |
|           QSql           |    包含了在整个Qt SQL模块中使用的各种标识符     |
|       QSqlDatabase       |                处理数据库的连接                 |
|        QSqlDriver        |        用于访问特定 SQL 数据库的抽象基类        |
|    QSqlDriverCreator     | 提供特定驱动程序类型的 SQL 驱动程序工厂的模板类 |
|  QSqlDriverCreatorBase   |             SQL 驱动程序工厂的基类              |
|        QSqlError         |                SQL数据库错误信息                |
|        QSqlField         |         操作 SQL 数据库表和视图中的字段         |
|        QSqlIndex         |         用于操作和描述数据库索引的函数          |
|        QSqlQuery         |            执行和操作 SQL 语句的方法            |
|      QSqlQueryModel      |                  QSql查询模型                   |
|        QSqlRecord        |                 f封装数据库记录                 |
| QSqlRelationalTableModel |     具有外键支持的单个数据库表的可编辑数据      |
|        QSqlResult        |     用于访问特定 SQL 数据库中数据的抽象接口     |
|      QSqlTableModel      |          单个数据库表的可编辑数据模型           |


SQL类分为以下三层：
### 驱动层
包含以下类 QSqlDriver, QSqlDriverCreator, QSqlDriverCreatorBase, QSqlDriverPlugin, 和 QSqlResult。

这一层提供了特定数据库和 SQL API 之间的底层桥梁。

### SQL API层
这些类提供对数据库的访问。数据库连接通过 QSqlDatabase 类建立。数据库交互则通过 QSqlQuery 类实现。除了 QSqlDatabase 和 QSqlQuery 之外，SQL API 层还支持 QSqlError、QSqlField、QSqlIndex 和 QSqlRecord。

### 用户接口层
这些类将数据库中的数据与数据感知型控件连接起来。它们包括 QSqlQueryModel、QSqlTableModel 和 QSqlRelationalTableModel。这些类旨在与 Qt 的模型/视图框架配合使用。

请注意，在使用这些类之前，必须先实例化一个 QCoreApplication 对象。

## 连接到数据库
为了通过 QSqlQuery、QSqlQueryModel访问数据库，需要创建并打开一个或多个的数据库连接。数据库连接通常由连接名称标识，而不是数据库的名字。对于一个数据库，你可以同时拥有其多个数据库连接，QSqlDatabase也支持默认连接的概念，即未指定连接名称的连接。当调用 QSqlQuery 或 QSqlQueryModel的成员方法时要传递一个连接名参数，如果没有传递连接名参数，则使用默认连接。当你的应用只需要一个数据库连接，创建一个默认连接更为便利。

请注意创建连接和打开连接之间的区别。创建连接涉及创建 QSqlDatabase 类的实例。连接必须打开后才能使用。以下代码片段展示了如何创建默认连接并将其打开：
```cpp
QSqlDatabase db = QSqlDatabase::addDatabase("QMYSQL");
    db.setHostName("bigblue");
    db.setDatabaseName("flightdb");
    db.setUserName("acarlson");
    db.setPassword("1uTbSbAs");
    bool ok = db.open();
```

第一行创建连接对象，最后一行打开。在中间，我们初始化一些连接信息，包括数据库名、主机名、用户名、密码。"QMYSQL"参数传递到addDatabase()指定了要使用的连接的数据库驱动的类型。数据库驱动包含在[表格](https://doc.qt.io/qt-6/sql-driver.html#supported-databases)中。


可以指定连接名，例如：
```cpp
    QSqlDatabase firstDB = QSqlDatabase::addDatabase("QMYSQL", "first");
    QSqlDatabase secondDB = QSqlDatabase::addDatabase("QMYSQL", "second");
```

在创建连接并打开后，如果失败，`open()`将返回false.在这种情况下，调用`QSqlDatabase::lastError`获取错误信息。

一旦建立连接，就可以用 QSqlDatabase::database("连接名") 获取该连接的指针。不传参数则获取默认连接。


```cpp
    QSqlDatabase defaultDB = QSqlDatabase::database();
    QSqlDatabase firstDB = QSqlDatabase::database("first");
    QSqlDatabase secondDB = QSqlDatabase::database("second");

```

要移除数据库连接，首先使用 QSqlDatabase::close() 关闭数据库，然后使用静态方法 QSqlDatabase::removeDatabase() 将其移除。


## 执行SQL语句
### 执行查询
```cpp
    QSqlQuery query;
    query.exec("SELECT name, salary FROM employee WHERE salary > 50000");
```

QSqlQuery 构造函数接受一个可选的 QSqlDatabase 对象参数，用于指定要使用的数据库连接。在上面的示例中，我们没有指定任何连接，因此使用了默认连接。

如果发生错误，exec() 将返回 false。错误信息可通过 QSqlQuery::lastError() 获取。

### 导航结果集
QSqlQuery 提供对结果集的逐条记录访问。调用 exec() 后，QSqlQuery 的内部指针位于第一条记录之前的位置。我们必须先调用一次 QSqlQuery::next() 来前进到第一条记录，然后反复调用 next() 以访问其他记录，直到它返回 false。以下是一个典型的循环，按顺序遍历所有记录：

```cpp
    while (query.next()) {
        QString name = query.value(0).toString();
        int salary = query.value(1).toInt();
        qDebug() << name << salary;
    }
```
QSqlQuery::value() 函数返回当前记录中某个字段的值。字段通过从零开始的索引指定。QSqlQuery::value() 返回一个 QVariant，这是一种可以容纳各种 C++ 和 Qt 核心数据类型（如 int、QString 和 QByteArray）的类型。不同的数据库类型会自动映射到最接近的 Qt 等效类型。在代码片段中，我们调用 QVariant::toString() 和 QVariant::toInt() 来将 QVariant 转换为 QString 和 int。

有关 Qt 支持的数据库推荐使用的类型，请参考此表。

您可以使用 `QSqlQuery::next()`、`QSqlQuery::previous()`、`QSqlQuery::first()`、`QSqlQuery::last()` 和 `QSqlQuery::seek()` 在数据集中导航。当前行的索引由 `QSqlQuery::at()` 返回，结果集中的总行数可通过 `QSqlQuery::size()` 获取（如果数据库支持此功能）。

要确定数据库驱动程序是否支持特定功能，请使用 `QSqlDriver::hasFeature()`。在以下示例中，如果底层数据库支持该功能，我们调用 `QSqlQuery::size()` 来确定结果集的大小；否则，我们导航到最后一条记录，并使用查询的位置来告诉我们有多少条记录。

```cpp
QSqlQuery query;
    int numRows;
    query.exec("SELECT name, salary FROM employee WHERE salary > 50000");

    QSqlDatabase defaultDB = QSqlDatabase::database();
    if (defaultDB.driver()->hasFeature(QSqlDriver::QuerySize)) {
        numRows = query.size();
    } else {
        // this can be very slow
        query.last();
        numRows = query.at() + 1;
    }
```

如果在结果集中导航，并且仅使用 `next()` 和 `seek()` 向前浏览，可以在调用 `exec()` 之前调用 `QSqlQuery::setForwardOnly(true)`。这是一个简单的优化，当处理大型结果集时，可以显著加快查询速度。


### 插入，更新和删除记录

QSqlQuery 可以执行任意 SQL 语句，而不仅仅是 SELECT。以下示例使用 INSERT 向表中插入一条记录：


```cpp
    QSqlQuery query;
    query.exec("INSERT INTO employee (id, name, salary) "
               "VALUES (1001, 'Thad Beaumont', 65000)");

```

如果想同时插入多条记录，通常更高效的做法是将查询语句与实际要插入的值分开。这可以通过占位符来实现。Qt 支持两种占位符语法：命名绑定和位置绑定。以下是命名绑定的示例：


```cpp
    QSqlQuery query;
    query.prepare("INSERT INTO employee (id, name, salary) "
                  "VALUES (:id, :name, :salary)");
    query.bindValue(":id", 1001);
    query.bindValue(":name", "Thad Beaumont");
    query.bindValue(":salary", 65000);
    query.exec();

```

以下是位置绑定的示例：


```cpp
    QSqlQuery query;
    query.prepare("INSERT INTO employee (id, name, salary) "
                  "VALUES (?, ?, ?)");
    query.addBindValue(1001);
    query.addBindValue("Thad Beaumont");
    query.addBindValue(65000);
    query.exec();

```


这两种语法都适用于 Qt 提供的所有数据库驱动程序。如果数据库本身支持该语法，Qt 会直接将查询转发给数据库管理系统；否则，Qt 会通过预处理查询来模拟占位符语法。最终由数据库管理系统执行的实际查询可以通过 QSqlQuery::executedQuery() 获取。

当插入多条记录时，只需调用一次 QSqlQuery::prepare()。然后可以根据需要多次调用 bindValue() 或 addBindValue()，再调用 exec()。

除了性能优势外，使用占位符的另一个优点是，可以轻松指定任意值，而无需担心转义特殊字符。

更新记录与向表中插入记录类似：

```cpp
    QSqlQuery query;
    query.exec("UPDATE employee SET salary = 70000 WHERE id = 1003");

```

你也可以使用命名绑定或位置绑定将参数与实际值关联起来。

最后，以下是一个 DELETE 语句的示例：

```cpp
    QSqlQuery query;
    query.exec("DELETE FROM employee WHERE id = 1007");

```

### 事务处理
如果底层数据库引擎支持事务，`QSqlDriver::hasFeature(QSqlDriver::Transactions)` 将返回 `true`。你可以使用 `QSqlDatabase::transaction()` 来启动一个事务，然后执行你想在该事务上下文中运行的 SQL 命令，最后调用 `QSqlDatabase::commit()` 或 `QSqlDatabase::rollback()`。在使用事务时，必须在创建查询之前启动事务。

例如：
```cpp
    ::database().transaction();
    QSqlQuery query;
    query.exec("SELECT id FROM employee WHERE name = 'Torild Halvorsen'");
    if (query.next()) {
        int employeeId = query.value(0).toInt();
        query.exec("INSERT INTO project (id, name, ownerid) "
                   "VALUES (201, 'Manhattan Project', "
                   + QString::number(employeeId) + ')');
    }
    QSqlDatabase::database().commit();

```

事务可用于确保复杂操作的原子性（例如，查找外键并创建记录），或提供中途取消复杂更改的方法。


 
## 使用SQL模型类SQL 

除了QSqlQuery,Qt提供了三个更高级的类用于访问数据库。

| 类名                | 描述                             |
| ------------------- | -------------------------------- |
| QSqlQueryModel      | 基于任意SQL查询的只读模型类      |
| QSqlTableModel      | 基于单个表的可读写模型类         |
| QSqlRelationalModel | 一个支持外键的QSqlTableModel子类 |

这些类派生自QAbstractTableModel(它继承自QAbstractItemModel)，使得他们很轻松的将数据库中的数据展示在Qt的视图类中（如QListView和QTableView）。

使用这些类的另一个优点是，它可以让你的代码更容易适应其他数据源。例如，如果你使用了 QSqlTableModel，之后决定使用 XML 文件而不是数据库来存储数据，那么本质上只需要将一个数据模型替换为另一个数据模型即可。

### QSqlQueryModel

它是只读的sql查询模型。

如：

```cpp
    QSqlQueryModel model;
    model.setQuery("SELECT * FROM employee");

    for (int i = 0; i < model.rowCount(); ++i) {
        int id = model.record(i).value("id").toInt();
        QString name = model.record(i).value("name").toString();
        qDebug() << id << name;
    }

```
使用 `QSqlQueryModel::setQuery()` 设置查询后，您可以使用 `QSqlQueryModel::record(int)` 访问各个记录。您还可以使用 `QSqlQueryModel::data()` 以及从 `QAbstractItemModel` 继承的任何其他函数。

此外，还有一个 `setQuery()` 重载函数，它接受一个 `QSqlQuery` 对象并对其结果集进行操作。这使您可以使用 `QSqlQuery` 的任何功能来设置查询（例如，预处理查询）。

### SQL Table Model

QSqlTableModel 提供了一种读写模型，一次只能处理一个 SQL 表。

如：

```cpp
    QSqlTableModel model;
    model.setTable("employee");
    model.setFilter("salary > 50000");
    model.setSort(2, Qt::DescendingOrder);
    model.select();

    for (int i = 0; i < model.rowCount(); ++i) {
        QString name = model.record(i).value("name").toString();
        int salary = model.record(i).value("salary").toInt();
        qDebug() << name << salary;
    }

```

QSqlTableModel 是 QSqlQuery 的高级替代方案，用于浏览和修改单个 SQL 表。它通常代码量更少，并且无需了解 SQL 语法。

使用 QSqlTableModel::record() 可以检索表中的一行，使用 QSqlTableModel::setRecord() 可以修改该行。例如，以下代码会将每位员工的工资提高 10%：

```cpp
    for (int i = 0; i < model.rowCount(); ++i) {
        QSqlRecord record = model.record(i);
        double salary = record.value("salary").toInt();
        salary *= 1.1;
        record.setValue("salary", salary);
        model.setRecord(i, record);
    }
    model.submitAll();
```

您还可以使用继承自 QAbstractItemModel 的 QSqlTableModel::data() 和 QSqlTableModel::setData() 来访问数据。例如，以下是如何使用 setData() 更新记录：

```cpp
    model.setData(model.index(row, column), 75000);
    model.submitAll();

以下是如何插入一行并填充数据：
```cpp
    model.insertRows(row, 1);
    model.setData(model.index(row, 0), 1013);
    model.setData(model.index(row, 1), "Peter Gordon");
    model.setData(model.index(row, 2), 68500);
    model.submitAll();

以下是如何删除连续五行的方法：
```cpp
    model.removeRows(row, 5);
    model.submitAll();

    

