---
tags:
  - Qt
  - 其他相关
  - 高质量
---
# 通用数据库翻页 (`dbpage`)

> 通用数据库翻页

## 效果图

![[QWidgetDemo_assert/dbpage.jpg]]

## 分类

- 类型: 其他相关 `other`
- 源码目录: `other/dbpage`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QString QLabel QList QAbstractButton QFrame QTableView QTextCodec QObject QSqlDatabase QWidget QComboBox QPushButton QVariant QThread QDateTime QSqlQueryModel QSqlQuery QModelIndex QColor QStringList

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./dbpage.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")
#include "dbpage.h"

SqlQueryModel::SqlQueryModel(QObject *parent) : QSqlQueryModel(parent)
{
    allCenter = false;
    alignCenterColumn.clear();
    alignRightColumn.clear();
}

QVariant SqlQueryModel::data(const QModelIndex &index, int role) const
{
    QVariant value = QSqlQueryModel::data(index, role);

    if (allCenter) {
        if (role == Qt::TextAlignmentRole) {
            value = Qt::AlignCenter;
        }
    } else {
        //逐个从列索引中查找是否当前列在其中
        int column = index.column();
        bool existCenter = alignCenterColumn.contains(column);
        bool existRight = alignRightColumn.contains(column);

        if (role == Qt::TextAlignmentRole) {
            if (existCenter) {
                value = Qt::AlignCenter;
            }

            if (existRight) {
                value = (QVariant)(Qt::AlignVCenter | Qt::AlignRight);
            }
        }
    }

    //实现鼠标经过整行换色,如果设置了hoverRow才需要处理
    if (property("hoverRow").isValid()) {
        int row = property("hoverRow").toInt();
        if (row == index.row()) {
            if (role == Qt::BackgroundRole) {
                value = QColor(property("hoverBgColor").toString());
            } else if (role == Qt::ForegroundRole) {
                value = QColor(property("hoverTextColor").toString());
            }
        }
    }

    //实现隐藏部分显示,指定列和替换字符
    if (property("hideColumn").isValid()) {
        int column = property("hideColumn").toInt();
        if (column == index.column()) {
            if (role == Qt::DisplayRole) {
                QString letter = property("hideLetter").toString();
                int start = property("hideStart").toInt();
                int end = property("hideEnd").toInt();
                QString str = value.toString();

                QStringList list;
                for (int i = 0; i < str.length(); i++) {
                    if (i >= start && i <= end) {
                        list << letter;
                    } else {
                        list << str.at(i);
                    }
                }

                value = list.join("");
            }
        }
    }

    return value;
}

void SqlQueryModel::setAllCenter(bool allCenter)
{
    this->allCenter = allCenter;
}

void SqlQueryModel::setAlignCenterColumn(const QList<int> &alignCenterColumn)
{
    this->alignCenterColumn = alignCenterColumn;
}

void SqlQueryModel::setAlignRightColumn(const QList<int> &alignRightColumn)
{
    this->alignRightColumn = alignRightColumn;
}


DbCountThread::DbCountThread(QObject *parent) : QThread(parent)
{
    connName = "qt_sql_default_connection";
    sql = "select 1";

    connect(this, SIGNAL(finished()), this, SLOT(deleteLater()));
}

void DbCountThread::run()
{
    select();
}

void DbCountThread::setConnName(const QString &connName)
{
    this->connName = connName;
}

void DbCountThread::setSql(const QString &sql)
{
    this->sql = sql;
}

void DbCountThread::select()
{
    //计算用时
    QDateTime dtStart = QDateTime::currentDateTime();
    int count = 0;
    QSqlQuery query(QSqlDatabase::database(connName));
    if (query.exec(sql)) {
        if (query.next()) {
            count = query.value(0).toUInt();
        }
    }

    QDateTime dtEnd = QDateTime::currentDateTime();
    double msec = dtStart.msecsTo(dtEnd);
    emit receiveCount(count, msec);
}


DbPage::DbPage(QObject *parent) : QObject(parent)
{
    startIndex = 0;
    queryModel = new SqlQueryModel;

    pageCurrent = 1;
    pageTotal = 0;
    recordsTotal = 0;
    recordsPerpage = 0;

    labPageTotal = 0;
    labPageCurrent = 0;
    labRecordsTotal = 0;
    labRecordsPerpage = 0;
    labSelectTime = 0;
    labSelectInfo = 0;

    tableView = 0;
    btnFirst = 0;
    btnPrevious = 0;
    btnNext = 0;
    btnLast = 0;

    countName = "*";
    connName = "qt_sql_default_connection";
    dbType = DbType_Sqlite;

    pageCurrent = 0;
    pageTotal = 0;
    recordsTotal = 0;
    recordsPerpage = 30;

    tableName = "";
    selectColumn = "*";
    orderSql = "";
    whereSql = "";
    columnNames.clear();
    columnWidths.clear();

    insertColumnIndex = -1;
    insertColumnName = "";
    insertColumnWidth = 50;
}

void DbPage::bindData(const QString &columnName, const QString &orderColumn, const QString &tableName,
                      QComboBox *cbox, const QString &connName)
{
    QSqlQuery query(QSqlDatabase::database(connName));
    query.exec("select " + columnName + " from " + tableName + " order by " + orderColumn + " asc");
    while (query.next()) {
        cbox->addItem(query.value(0).toString());
    }
}

void DbPage::bindData(const QString &columnName, const QString &orderColumn, const QString &tableName,
                      QList<QComboBox *> cboxs, const QString &connName)
{
    QSqlQuery query(QSqlDatabase::database(connName));
    query.exec("select " + columnName + " from " + tableName + " order by " + orderColumn + " asc");
    while (query.next()) {
        foreach (QComboBox *cbox, cboxs) {
            cbox->addItem(query.value(0).toString());
        }
    }
}

void DbPage::bindData(const QString &sql)
{
    queryModel->setQuery(sql, QSqlDatabase::database(connName));
    tableView->setModel(queryModel);

    //依次设置列标题列宽
    int columnCount = tableView->model()->columnCount();
    int nameCount = columnNames.count();
    columnCount = columnCount > nameCount ? nameCount : columnCount;

    QList<QString> columnNames = this->columnNames;
    QList<int> columnWidths = this->columnWidths;

    //根据设置添加新列,将对应新列的标题名称和宽度按照索引位置插
    if (insertColumnIndex >= 0) {
        columnCount++;
        columnNames.insert(insertColumnIndex, insertColumnName);
        columnWidths.insert(insertColumnIndex, insertColumnWidth);
        queryModel->insertColumn(insertColumnIndex);
    }

    //设置列标题和列宽度
    for (int i = 0; i < columnCount; i++) {
        queryModel->setHeaderData(i, Qt::Horizontal, columnNames.at(i));
        tableView->setColumnWidth(i, columnWidths.at(i));
    }

    if (labPageCurrent != 0) {
        labPageCurrent->setText(QString("第 %1 页").arg(pageCurrent));
    }

    if (labPageTotal != 0) {
        labPageTotal->setText(QString("共 %1 页").arg(pageTotal));
    }

    if (labRecordsTotal != 0) {
        labRecordsTotal->setText(QString("共 %1 条").arg(recordsTotal));
    }

    if (labRecordsPerpage != 0) {
        labRecordsPerpage->setText(QString("每页 %1 条").arg(recordsPerpage));
    }

    if (labSelectInfo != 0) {
        //labSelectInfo->setText(QString("共 %1 条  每页 %2 条  共 %3 页  第 %4 页").arg(recordsTotal).arg(recordsPerpage).arg(pageTotal).arg(pageCurrent));
        labSelectInfo->setText(QString("第 %1 页  每页 %2 条  共 %3 页  共 %4 条").arg(pageCurrent).arg(recordsPerpage).arg(pageTotal).arg(recordsTotal));
    }

    //发送结果信号
    if (recordsTotal != recordsPerpage) {
        emit receivePage(pageCurrent, pageTotal, recordsTotal, recordsPerpage);
        //qDebug() << TIMEMS << startIndex << pageCurrent << pageTotal << recordsTotal << recordsPerpage;
    }

    changeBtnEnable();
}

QString DbPage::getPageSql()
{
    //组织分页SQL语句,不同的数据库分页语句不一样
    QString sql = QString("select %1 from %2 %3 order by %4").arg(selectColumn).arg(tableName).arg(whereSql).arg(orderSql);
    if (dbType == DbType_PostgreSQL || dbType == DbType_KingBase) {
        sql = QString("%1 limit %3 offset %2;").arg(sql).arg(startIndex).arg(recordsPerpage);
    } else if (dbType == DbType_SqlServer) {
        //取第m条到第n条记录：select top (n-m+1) id from tablename where id not in (select top m-1 id from tablename)
        //sql = QString("select %1 from %2 %3 order by %4").arg(selectColumn).arg(tableName).arg(whereSql).arg(orderSql);
    } else if (dbType == DbType_Oracle) {
        //暂时没有找到好办法
    } else {
        sql = QString("%1 limit %2,%3;").arg(sql).arg(startIndex).arg(recordsPerpage);
    }

    return sql;
}

void DbPage::slot_receiveCount(quint32 count, double msec)
{
    if (labSelectTime != 0) {
        labSelectTime->setText(QString("查询用时 %1 秒").arg(QString::number(msec / 1000, 'f', 3)));
    }

    recordsTotal = count;
    int yushu = recordsTotal % recordsPerpage;

    //不存在余数,说明是整行,例如300%5==0
    if (yushu == 0) {
        if (recordsTotal > 0 && recordsTotal < recordsPerpage) {
            pageTotal = 1;
        } else {
            pageTotal = recordsTotal / recordsPerpage;
        }
    } else {
        pageTotal = (recordsTotal / recordsPerpage) + 1;
    }

    bindData(getPageSql());
}

//设置显示数据的表格控件,当前翻页信息的标签控件等
void DbPage::setControl(QTableView *tableView,
                        QLabel *labPageTotal, QLabel *labPageCurrent,
                        QLabel *labRecordsTotal, QLabel *labRecordsPerpage,
                        QLabel *labSelectTime, QLabel *labSelectInfo,
                        QAbstractButton *btnFirst, QAbstractButton *btnPrevious,
                        QAbstractButton *btnNext, QAbstractButton *btnLast,
                        const QString &countName, const QString &connName)
{
    this->tableView = tableView;
    this->labPageTotal = labPageTotal;
    this->labPageCurrent = labPageCurrent;
    this->labRecordsTotal = labRecordsTotal;
    this->labRecordsPerpage = labRecordsPerpage;
    this->labSelectTime = labSelectTime;
    this->labSelectInfo = labSelectInfo;

    this->btnFirst = btnFirst;
    this->btnPrevious = btnPrevious;
    this->btnNext = btnNext;
    this->btnLast = btnLast;

    this->countName = countName;
    this->connName = connName;
    this->tableView->setSelectionBehavior(QAbstractItemView::SelectRows);

    if (btnFirst == 0 || btnPrevious == 0 || btnNext == 0 || btnLast == 0) {
        return;
    }

    //挂载翻页按钮事件
    connect(btnFirst, SIGNAL(clicked()), this, SLOT(first()));
    connect(btnPrevious, SIGNAL(clicked()), this, SLOT(previous()));
    connect(btnNext, SIGNAL(clicked()), this, SLOT(next()));
    connect(btnLast, SIGNAL(clicked()), this, SLOT(last()));
}

void DbPage::setControl(QTableView *tableView,
                        QLabel *labPageTotal, QLabel *labPageCurrent,
                        QLabel *labRecordsTotal, QLabel *labRecordsPerpage,
                        QLabel *labSelectTime, QLabel *labSelectInfo,
                        const QString &countName, const QString &connName)
{
    setControl(tableView, labPageTotal, labPageCurrent, labRecordsTotal, labRecordsPerpage, labSelectTime, labSelectInfo, 0, 0, 0, 0, countName, connName);
}

void DbPage::setControl(QTableView *tableView,
                        QAbstractButton *btnFirst, QAbstractButton *btnPrevious,
                        QAbstractButton *btnNext, QAbstractButton *btnLast,
                        const QString &countName, const QString &connName)
{
    setControl(tableView, 0, 0, 0, 0, 0, 0, btnFirst, btnPrevious, btnNext, btnLast, countName, connName);
}

void DbPage::setControl(QTableView *tableView, const QString &countName, const QString &connName)
{
    setControl(tableView, 0, 0, 0, 0, 0, 0, 0, 0, 0, 0, countName, connName);
}

void DbPage::setConnName(const QString &connName)
{
    this->connName = connName;
}

void DbPage::setDbType(const DbPage::DbType &dbType)
{
    this->dbType = dbType;
}

void DbPage::setTableName(const QString &tableName)
{
    this->tableName = tableName;
}

void DbPage::setSelectColumn(const QString &selectColumn)
{
    this->selectColumn = selectColumn;
}

void DbPage::setOrderSql(const QString &orderSql)
{
    this->orderSql = orderSql;
}

void DbPage::setWhereSql(const QString &whereSql)
{
    this->whereSql = whereSql;
}

void DbPage::setRecordsPerpage(int recordsPerpage)
{
    this->recordsPerpage = recordsPerpage;
}

void DbPage::setColumnNames(const QList<QString> &columnNames)
{
    this->columnNames = columnNames;
}

void DbPage::setColumnWidths(const QList<int> &columnWidths)
{
    this->columnWidths = columnWidths;
}

void DbPage::setAllCenter(bool allCenter)
{
    queryModel->setAllCenter(allCenter);
}

void DbPage::setAlignCenterColumn(const QList<int> &alignCenterColumn)
{
    queryModel->setAlignCenterColumn(alignCenterColumn);
}

void DbPage::setAlignRightColumn(const QList<int> &alignRightColumn)
{
    queryModel->setAlignRightColumn(alignRightColumn);
}

void DbPage::setInsertColumnIndex(int insertColumnIndex)
{
    this->insertColumnIndex = insertColumnIndex;
}

void DbPage::setInsertColumnName(const QString &insertColumnName)
{
    this->insertColumnName = insertColumnName;
}

void DbPage::setInsertColumnWidth(int insertColumnWidth)
{
    this->insertColumnWidth = insertColumnWidth;
}

void DbPage::changeBtnEnable()
{
    if (btnFirst == 0 || btnPrevious == 0 || btnNext == 0 || btnLast == 0) {
        return;
    }

    //下面默认对上一页下一页按钮禁用
    //也可以取消注释对第一页末一页同样处理
    //因为到了第一页就可以不必再单击第一页和上一页
    if (pageTotal <= 1) {
        //如果只有一页数据则翻页按钮不可用
        btnFirst->setEnabled(false);
        btnLast->setEnabled(false);
        btnPrevious->setEnabled(false);
        btnNext->setEnabled(false);
    } else {
        //判断是否在首页末页禁用按钮
        bool first = (pageCurrent == 1);
        bool last = (pageCurrent == pageTotal);
        btnFirst->setEnabled(!first);
        btnLast->setEnabled(!last);
        btnPrevious->setEnabled(!first);
        btnNext->setEnabled(!last);
    }
}

void DbPage::select()
{
    //重置开始索引
    startIndex = 0;
    pageCurrent = 1;
    pageTotal = 1;
    changeBtnEnable();

    //假设只有一页
    slot_receiveCount(recordsPerpage, 0);

    //文本显示正在查询...
    QString info = "正在查询...";
    if (labSelectInfo != 0) {
        labSelectInfo->setText(info);
    }
    if (labPageCurrent != 0) {
        labPageCurrent->setText(info);
    }
    if (labPageTotal != 0) {
        labPageTotal->setText(info);
    }
    if (labRecordsTotal != 0) {
        labRecordsTotal->setText(info);
    }
    if (labRecordsPerpage != 0) {
        labRecordsPerpage->setText(info);
    }
    if (labSelectTime != 0) {
        labSelectTime->setText(info);
    }

    //开始分页绑定数据前,计算好总数据量以及行数
    QString sql = QString("select count(%1) from %2 %3").arg(countName).arg(tableName).arg(whereSql);

    //采用线程执行查询复合条件的记录行数
    DbCountThread *dbCountThread = new DbCountThread(this);
    //绑定查询结果信号槽,一旦收到结果则立即执行
    connect(dbCountThread, SIGNAL(receiveCount(quint32, double)), this, SIGNAL(receiveCount(quint32, double)));
    connect(dbCountThread, SIGNAL(receiveCount(quint32, double)), this, SLOT(slot_receiveCount(quint32, double)));

    //设置数据库连接名称和查询语句,并启动线程
    dbCountThread->setConnName(connName);
    dbCountThread->setSql(sql);
    //从5.10开始不支持数据库在线程中执行
#if (QT_VERSION <= QT_VERSION_CHECK(5,10,0))
    dbCountThread->start();
#else
    dbCountThread->select();
#endif
}

void DbPage::selectPage(int page)
{
    //必须小于总页数+不是当前页
    if (page >= 1 && page <= pageTotal && page != pageCurrent) {
        //计算指定页对应开始的索引
        startIndex = (page - 1) * recordsPerpage;
        pageCurrent = page;
        bindData(getPageSql());
    }
}

void DbPage::first()
{
    //当前页不是第一页才能切换到第一页
    if (pageTotal > 1 && pageCurrent != 1) {
        startIndex = 0;
        pageCurrent = 1;
        bindData(getPageSql());
    }
}

void DbPage::previous()
{
    //当前页不是第一页才能上一页
    if (pageCurrent > 1) {
        pageCurrent--;
        startIndex -= recordsPerpage;
        bindData(getPageSql());
    }
}

void DbPage::next()
{
    //当前页小于总页数才能下一页
    if (pageCurrent < pageTotal) {
        pageCurrent++;
        startIndex += recordsPerpage;
        bindData(getPageSql());
    }
}

void DbPage::last()
{
    //当前页不是末尾页才能切换到末尾页
    if (pageTotal > 1 && pageCurrent != pageTotal) {
        startIndex = (pageTotal - 1) * recordsPerpage;
        pageCurrent = pageTotal;
        bindData(getPageSql());
    }
}
```

### `./dbpage.h`

```cpp
﻿#ifndef DBPAGE_H
#define DBPAGE_H

/**
 * 数据库通用翻页类 作者:feiyangqingyun(QQ:517216493) 2017-1-15
 * 1:自动按照设定的每页多少行数据分页
 * 2:只需要传入表名/字段集合/每页行数/翻页指示按钮/文字指示标签
 * 3:提供公共静态方法绑定字段数据到下拉框
 * 4:建议条件字段用数字类型的主键,速度极快
 * 5:增加线程查询符合条件的记录总数,数据量巨大时候不会卡主界面
 * 6:提供查询结果返回信号,包括当前页/总页数/总记录数/查询用时
 * 7:可设置所有列或者某一列对齐样式例如居中或者右对齐
 * 8:可设置增加一列,列的位置,标题,宽度
 * 9:可设置要查询的字段集合
 */

#include <QtGui>
#include <QtSql>
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
#include <QtWidgets>
#endif

//自定义模型设置列居中和右对齐
class SqlQueryModel: public QSqlQueryModel
{
public:
    explicit SqlQueryModel(QObject *parent = 0);

protected:
    QVariant data(const QModelIndex &index, int role) const;

private:
    bool allCenter;                 //所有居中
    QList<int> alignCenterColumn;   //居中对齐列
    QList<int> alignRightColumn;    //右对齐列

public:
    //设置所有列居中
    void setAllCenter(bool allCenter);
    //设置居中对齐列索引集合
    void setAlignCenterColumn(const QList<int> &alignCenterColumn);
    //设置右对齐列索引集合
    void setAlignRightColumn(const QList<int> &alignRightColumn);
};

//计算复合条件的记录总行数,以便分页
class DbCountThread : public QThread
{
    Q_OBJECT
public:
    explicit DbCountThread(QObject *parent = 0);

private:
    QString connName;   //数据库连接名称
    QString sql;        //要执行的查询语句

protected:
    void run();

signals:
    void receiveCount(quint32 count, double msec);

public slots:
    //设置数据库连接名称
    void setConnName(const QString &connName);
    //设置要执行的查询语句
    void setSql(const QString &sql);
    //查询行数
    void select();
};

class DbPage : public QObject
{
    Q_OBJECT
public:
    enum DbType {
        DbType_ODBC = 0,        //odbc数据源
        DbType_Sqlite = 1,      //sqlite数据库
        DbType_MySql = 2,       //mysql数据库
        DbType_PostgreSQL = 3,  //postgresql数据库
        DbType_SqlServer = 4,   //sqlserver数据库
        DbType_Oracle = 5,      //oracle数据库
        DbType_KingBase = 6,    //人大金仓数据库
        DbType_Other = 255      //其他数据库
    };

    explicit DbPage(QObject *parent = 0);

    //绑定数据到下拉框
    static void bindData(const QString &columnName, const QString &orderColumn, const QString &tableName,
                         QComboBox *cbox, const QString &connName = "qt_sql_default_connection");
    static void bindData(const QString &columnName, const QString &orderColumn, const QString &tableName,
                         QList<QComboBox *> cboxs, const QString &connName = "qt_sql_default_connection");

private:
    int startIndex;             //分页开始索引,每次翻页都变动
    SqlQueryModel *queryModel;  //查询模型

    QLabel *labPageTotal;       //总页数标签
    QLabel *labPageCurrent;     //当前页标签
    QLabel *labRecordsTotal;    //总记录数标签
    QLabel *labRecordsPerpage;  //每页记录数标签
    QLabel *labSelectTime;      //显示查询用时标签
    QLabel *labSelectInfo;      //总页数当前页总记录数每页记录数

    QTableView *tableView;      //显示数据的表格对象
    QAbstractButton *btnFirst;  //第一页按钮对象
    QAbstractButton *btnPrevious;//上一页按钮对象
    QAbstractButton *btnNext;   //下一页按钮对象
    QAbstractButton *btnLast;   //末一页按钮对象

    QString countName;          //统计表行数用字段
    QString connName;           //所使用的数据库连接名
    DbType dbType;              //数据库类型

    quint32 pageCurrent;        //当前第几页
    quint32 pageTotal;          //总页数
    quint32 recordsTotal;       //总记录数
    quint32 recordsPerpage;     //每页显示记录数

    QString tableName;          //表名
    QString selectColumn;       //要查询的字段集合
    QString orderSql;           //排序语句
    QString whereSql;           //条件语句
    QList<QString> columnNames; //列名集合
    QList<int> columnWidths;    //列宽集合

    int insertColumnIndex;      //插入的列的索引位置
    QString insertColumnName;   //插入的列的标题
    int insertColumnWidth;      //插入的列的宽度

private slots:
    //绑定sql语句到表格
    void bindData(const QString &sql);
    //生成分页sql语句
    QString getPageSql();

    //收到记录行数
    void slot_receiveCount(quint32 count, double msec);

signals:
    //将翻页后的页码信息发出去可能其他地方要用到
    void receivePage(quint32 pageCurrent, quint32 pageTotal, quint32 recordsTotal, quint32 recordsPerpage);
    void receiveCount(quint32 count, double msec);

public slots:
    //设置需要显示数据的表格,数据翻页对应的按钮
    void setControl(QTableView *tableView,
                    QLabel *labPageTotal, QLabel *labPageCurrent,
                    QLabel *labRecordsTotal, QLabel *labRecordsPerpage,
                    QLabel *labSelectTime, QLabel *labSelectInfo,
                    QAbstractButton *btnFirst, QAbstractButton *btnPrevious,
                    QAbstractButton *btnNext, QAbstractButton *btnLast,
                    const QString &countName, const QString &connName = "qt_sql_default_connection");
    void setControl(QTableView *tableView,
                    QLabel *labPageTotal, QLabel *labPageCurrent,
                    QLabel *labRecordsTotal, QLabel *labRecordsPerpage,
                    QLabel *labSelectTime, QLabel *labSelectInfo,
                    const QString &countName, const QString &connName = "qt_sql_default_connection");
    void setControl(QTableView *tableView,
                    QAbstractButton *btnFirst, QAbstractButton *btnPrevious,
                    QAbstractButton *btnNext, QAbstractButton *btnLast,
                    const QString &countName, const QString &connName = "qt_sql_default_connection");
    void setControl(QTableView *tableView,
                    const QString &countName, const QString &connName = "qt_sql_default_connection");

    //设置数据库连接名称
    void setConnName(const QString &connName);
    //设置数据库类型
    void setDbType(const DbType &dbType);

    //设置要查询的表名
    void setTableName(const QString &tableName);
    //设置要查询的字段列名集合
    void setSelectColumn(const QString &selectColumn);

    //设置排序sql
    void setOrderSql(const QString &orderSql);
    //设置条件sql
    void setWhereSql(const QString &whereSql);

    //设置每页显示多少行数据
    void setRecordsPerpage(int recordsPerpage);
    //设置列名称集合
    void setColumnNames(const QList<QString> &columnNames);
    //设置列宽度集合
    void setColumnWidths(const QList<int> &columnWidths);

    //设置所有列居中
    void setAllCenter(bool allCenter);
    //设置居中对齐列索引集合
    void setAlignCenterColumn(const QList<int> &alignCenterColumn);
    //设置右对齐列索引集合
    void setAlignRightColumn(const QList<int> &alignRightColumn);

    //设置插入的列的索引
    void setInsertColumnIndex(int insertColumnIndex);
    //设置插入的列的标题
    void setInsertColumnName(const QString &insertColumnName);
    //设置插入的列的宽度
    void setInsertColumnWidth(int insertColumnWidth);

    //根据首页末页禁用按钮
    void changeBtnEnable();
    //执行查询
    void select();
    //指定页跳转
    void selectPage(int page);

    //翻页 第一页+上一页+下一页+末一页
    void first();
    void previous();
    void next();
    void last();
};

#endif // DBPAGE_H
```

### `./frmdbpage.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmdbpage.h"
#include "ui_frmdbpage.h"
#include "dbpage.h"

frmDbPage::frmDbPage(QWidget *parent) : QWidget(parent), ui(new Ui::frmDbPage)
{
    ui->setupUi(this);
    this->initForm();
    on_btnSelect_clicked();
}

frmDbPage::~frmDbPage()
{
    delete ui;
}

void frmDbPage::initForm()
{
    columnNames.clear();
    columnWidths.clear();

    tableName = "LogInfo";
    countName = "rowid";

    columnNames.append("防区号");
    columnNames.append("防区名称");
    columnNames.append("设备IP");
    columnNames.append("日志类型");
    columnNames.append("事件内容");
    columnNames.append("触发时间");
    columnNames.append("告警详情");
    columnNames.append("告警数据");
    columnNames.append("告警图像");

    columnWidths.append(70);
    columnWidths.append(120);
    columnWidths.append(120);
    columnWidths.append(80);
    columnWidths.append(150);
    columnWidths.append(160);
    columnWidths.append(160);
    columnWidths.append(160);
    columnWidths.append(160);

    //设置需要显示数据的表格和翻页的按钮
    dbPage = new DbPage(this);
    //设置所有列居中显示
    dbPage->setAllCenter(true);
    dbPage->setControl(ui->tableMain, ui->labPageTotal, ui->labPageCurrent, ui->labRecordsTotal, ui->labRecordsPerpage,
                       ui->labSelectTime, 0, ui->btnFirst, ui->btnPreVious, ui->btnNext, ui->btnLast, countName);
    ui->tableMain->horizontalHeader()->setStretchLastSection(true);
    ui->tableMain->verticalHeader()->setDefaultSectionSize(25);
}

void frmDbPage::on_btnSelect_clicked()
{
    //绑定数据到表格
    QString sql = "where 1=1";
    dbPage->setTableName(tableName);
    dbPage->setOrderSql(QString("%1 %2").arg(countName).arg("asc"));
    dbPage->setWhereSql(sql);
    dbPage->setRecordsPerpage(20);
    dbPage->setColumnNames(columnNames);
    dbPage->setColumnWidths(columnWidths);
    dbPage->select();
}
```

### `./frmdbpage.h`

```cpp
﻿#ifndef FRMDBPAGE_H
#define FRMDBPAGE_H

#include <QWidget>

class DbPage;

namespace Ui {
class frmDbPage;
}

class frmDbPage : public QWidget
{
    Q_OBJECT

public:
    explicit frmDbPage(QWidget *parent = 0);
    ~frmDbPage();

private:
    Ui::frmDbPage *ui;

    QList<QString> columnNames; //字段名集合
    QList<int> columnWidths;    //字段宽度集合
    DbPage *dbPage;             //数据库翻页类

    QString tableName;          //表名称
    QString countName;          //统计行数字段名称

private slots:
    void initForm();

private slots:
    void on_btnSelect_clicked();
};

#endif // FRMDBPAGE_H
```

### `./head.h`

```cpp
﻿#include <QtCore>
#include <QtGui>
#include <QtSql>
#if (QT_VERSION > QT_VERSION_CHECK(5,0,0))
#include <QtWidgets>
#endif

#pragma execution_character_set("utf-8")
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmdbpage.h"
#include "qapplication.h"
#include "qtextcodec.h"
#include "qsqldatabase.h"
#include "qsqlquery.h"
#include "qdebug.h"

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    QFont font;
    font.setFamily("Microsoft Yahei");
    font.setPixelSize(13);
    a.setFont(font);

#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
#if _MSC_VER
    QTextCodec *codec = QTextCodec::codecForName("gbk");
#else
    QTextCodec *codec = QTextCodec::codecForName("utf-8");
#endif
    QTextCodec::setCodecForLocale(codec);
    QTextCodec::setCodecForCStrings(codec);
    QTextCodec::setCodecForTr(codec);
#else
    QTextCodec *codec = QTextCodec::codecForName("utf-8");
    QTextCodec::setCodecForLocale(codec);
#endif

    //打开数据库,整个应用程序可用
    QSqlDatabase dbConn = QSqlDatabase::addDatabase("QSQLITE");
    dbConn.setDatabaseName(qApp->applicationDirPath() + "/TA.db");
    if (dbConn.open()) {
        qDebug() << "连接数据库成功!";
    } else {
        qDebug() << "连接数据库失败!";
    }

    frmDbPage w;
    w.setWindowTitle("数据库分页示例 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmdbpage.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmDbPage</class>
 <widget class="QWidget" name="frmDbPage">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QHBoxLayout" name="horizontalLayout">
   <property name="leftMargin">
    <number>6</number>
   </property>
   <property name="topMargin">
    <number>6</number>
   </property>
   <property name="rightMargin">
    <number>6</number>
   </property>
   <property name="bottomMargin">
    <number>6</number>
   </property>
   <item>
    <widget class="QSplitter" name="splitter">
     <property name="orientation">
      <enum>Qt::Vertical</enum>
     </property>
     <property name="opaqueResize">
      <bool>false</bool>
     </property>
     <widget class="QTableView" name="tableMain">
      <property name="styleSheet">
       <string notr="true"/>
      </property>
     </widget>
    </widget>
   </item>
   <item>
    <widget class="QFrame" name="frameBottom">
     <property name="minimumSize">
      <size>
       <width>190</width>
       <height>0</height>
      </size>
     </property>
     <property name="maximumSize">
      <size>
       <width>190</width>
       <height>16777215</height>
      </size>
     </property>
     <property name="frameShape">
      <enum>QFrame::Box</enum>
     </property>
     <property name="frameShadow">
      <enum>QFrame::Sunken</enum>
     </property>
     <layout class="QVBoxLayout" name="verticalLayout">
      <property name="leftMargin">
       <number>6</number>
      </property>
      <property name="topMargin">
       <number>6</number>
      </property>
      <property name="rightMargin">
       <number>6</number>
      </property>
      <property name="bottomMargin">
       <number>6</number>
      </property>
      <item>
       <widget class="QPushButton" name="btnSelect">
        <property name="minimumSize">
         <size>
          <width>80</width>
          <height>0</height>
         </size>
        </property>
        <property name="text">
         <string>查询</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnPreVious">
        <property name="minimumSize">
         <size>
          <width>50</width>
          <height>0</height>
         </size>
        </property>
        <property name="text">
         <string>上一页</string>
        </property>
        <property name="iconSize">
         <size>
          <width>18</width>
          <height>18</height>
         </size>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnNext">
        <property name="minimumSize">
         <size>
          <width>50</width>
          <height>0</height>
         </size>
        </property>
        <property name="text">
         <string>下一页</string>
        </property>
        <property name="iconSize">
         <size>
          <width>18</width>
          <height>18</height>
         </size>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnFirst">
        <property name="minimumSize">
         <size>
          <width>50</width>
          <height>0</height>
         </size>
        </property>
        <property name="text">
         <string>第一页</string>
        </property>
        <property name="iconSize">
         <size>
          <width>18</width>
          <height>18</height>
         </size>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnLast">
        <property name="minimumSize">
         <size>
          <width>50</width>
          <height>0</height>
         </size>
        </property>
        <property name="text">
         <string>末一页</string>
        </property>
        <property name="iconSize">
         <size>
          <width>18</width>
          <height>18</height>
         </size>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QLabel" name="labPageTotal">
        <property name="minimumSize">
         <size>
          <width>0</width>
          <height>25</height>
         </size>
        </property>
        <property name="frameShape">
         <enum>QFrame::Box</enum>
        </property>
        <property name="frameShadow">
         <enum>QFrame::Sunken</enum>
        </property>
        <property name="text">
         <string/>
        </property>
        <property name="alignment">
         <set>Qt::AlignCenter</set>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QLabel" name="labRecordsTotal">
        <property name="minimumSize">
         <size>
          <width>0</width>
          <height>25</height>
         </size>
        </property>
        <property name="frameShape">
         <enum>QFrame::Box</enum>
        </property>
        <property name="frameShadow">
         <enum>QFrame::Sunken</enum>
        </property>
        <property name="text">
         <string/>
        </property>
        <property name="alignment">
         <set>Qt::AlignCenter</set>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QLabel" name="labPageCurrent">
        <property name="minimumSize">
         <size>
          <width>0</width>
          <height>25</height>
         </size>
        </property>
        <property name="frameShape">
         <enum>QFrame::Box</enum>
        </property>
        <property name="frameShadow">
         <enum>QFrame::Sunken</enum>
        </property>
        <property name="text">
         <string/>
        </property>
        <property name="alignment">
         <set>Qt::AlignCenter</set>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QLabel" name="labRecordsPerpage">
        <property name="minimumSize">
         <size>
          <width>0</width>
          <height>25</height>
         </size>
        </property>
        <property name="frameShape">
         <enum>QFrame::Box</enum>
        </property>
        <property name="frameShadow">
         <enum>QFrame::Sunken</enum>
        </property>
        <property name="text">
         <string/>
        </property>
        <property name="alignment">
         <set>Qt::AlignCenter</set>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QLabel" name="labSelectTime">
        <property name="minimumSize">
         <size>
          <width>0</width>
          <height>25</height>
         </size>
        </property>
        <property name="frameShape">
         <enum>QFrame::Box</enum>
        </property>
        <property name="frameShadow">
         <enum>QFrame::Sunken</enum>
        </property>
        <property name="text">
         <string/>
        </property>
        <property name="alignment">
         <set>Qt::AlignCenter</set>
        </property>
       </widget>
      </item>
      <item>
       <spacer name="verticalSpacer">
        <property name="orientation">
         <enum>Qt::Vertical</enum>
        </property>
        <property name="sizeHint" stdset="0">
         <size>
          <width>20</width>
          <height>40</height>
         </size>
        </property>
       </spacer>
      </item>
     </layout>
    </widget>
   </item>
  </layout>
 </widget>
 <resources/>
 <connections/>
</ui>
```
