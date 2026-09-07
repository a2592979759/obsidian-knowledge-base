---
tags:
  - Qt
  - 控件相关
---
# 磁盘容量 (`devicesizetable`)

> 磁盘容量

## 效果图

![[QWidgetDemo_assert/devicesizetable.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/devicesizetable`

## 技术要点

**用到的 Qt 类**: QColor QString QTextCodec QProgressBar QWidget QTableWidgetItem QTableWidget QStringList QSize QProcess QAbstractItemView QTimer QApplication QMetaObject QLatin1String QFileInfoList QDir QFileInfo QVBoxLayout QFont

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./devicesizetable.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "devicesizetable.h"
#include "qprocess.h"
#include "qtablewidget.h"
#include "qheaderview.h"
#include "qfileinfo.h"
#include "qdir.h"
#include "qprogressbar.h"
#include "qtimer.h"
#include "qdebug.h"

#ifdef Q_OS_WIN
#include "windows.h"
#endif
#define GB (1024 * 1024 * 1024)
#define MB (1024 * 1024)
#define KB (1024)

DeviceSizeTable::DeviceSizeTable(QWidget *parent) : QTableWidget(parent)
{
    bgColor = QColor(255, 255, 255);

    chunkColor1 = QColor(100, 184, 255);
    chunkColor2 = QColor(24, 189, 155);
    chunkColor3 = QColor(255, 107, 107);

    textColor1 = QColor(10, 10, 10);
    textColor2 = QColor(255, 255, 255);
    textColor3 = QColor(255, 255, 255);

#if defined(Q_OS_UNIX) && !defined(Q_OS_WASM)
    process = new QProcess(this);
    connect(process, SIGNAL(readyRead()), this, SLOT(readData()));
#endif

    this->clear();

    //设置列数和列宽
    this->setColumnCount(5);
    this->setColumnWidth(0, 100);
    this->setColumnWidth(1, 120);
    this->setColumnWidth(2, 120);
    this->setColumnWidth(3, 120);
    this->setColumnWidth(4, 120);

    this->setStyleSheet("QTableWidget::item{padding:0px;}");

    QStringList headText;
    headText << "盘符" << "已用空间" << "可用空间" << "总大小" << "已用百分比" ;

    this->setHorizontalHeaderLabels(headText);
    this->setSelectionBehavior(QAbstractItemView::SelectRows);
    this->setEditTriggers(QAbstractItemView::NoEditTriggers);
    this->setSelectionMode(QAbstractItemView::SingleSelection);
    this->verticalHeader()->setVisible(true);
    this->horizontalHeader()->setStretchLastSection(true);
    QMetaObject::invokeMethod(this, "load", Qt::QueuedConnection);
    //QTimer::singleShot(10, this, SLOT(load()));
}

void DeviceSizeTable::readData()
{
#if defined(Q_OS_UNIX) && !defined(Q_OS_WASM)
    while (!process->atEnd()) {
        QString result = QLatin1String(process->readLine());
#if defined(__arm__) || defined(__aarch64__)
        if (result.startsWith("/dev/root")) {
            checkSize(result, "本地存储");
        } else if (result.startsWith("/dev/mmcblk")) {
            checkSize(result, "本地存储");
        } else if (result.startsWith("/dev/mmcblk1p")) {
            checkSize(result, "SD卡");
            QStringList list = result.split(" ");
            Q_EMIT sdcardReceive(list.at(0));
        } else if (result.startsWith("/dev/sd")) {
            checkSize(result, "U盘");
            QStringList list = result.split(" ");
            Q_EMIT udiskReceive(list.at(0));
        }
#else
        if (result.startsWith("/dev/sd")) {
            checkSize(result, "");
            QStringList list = result.split(" ");
            Q_EMIT udiskReceive(list.at(0));
        }
#endif
    }
#endif
}

void DeviceSizeTable::checkSize(const QString &result, const QString &name)
{
    QString dev, use, free, all;
    int percent = 0;
    QStringList list = result.split(" ");
    int index = 0;

    for (int i = 0; i < list.count(); ++i) {
        QString s = list.at(i).trimmed();
        if (s.isEmpty()) {
            continue;
        }

        index++;
        if (index == 1) {
            dev = s;
        } else if (index == 2) {
            all = s;
        } else if (index == 3) {
            use = s;
        } else if (index == 4) {
            free = s;
        } else if (index == 5) {
            percent = s.left(s.length() - 1).toInt();
            break;
        }
    }

    if (name.length() > 0) {
        dev = name;
    }

    insertSize(dev, use, free, all, percent);
}

void DeviceSizeTable::insertSize(const QString &name, const QString &use, const QString &free, const QString &all, int percent)
{
    int row = this->rowCount();
    this->insertRow(row);

    QTableWidgetItem *itemname = new QTableWidgetItem(name);
    QTableWidgetItem *itemuse = new QTableWidgetItem(use);
    itemuse->setTextAlignment(Qt::AlignCenter);
    QTableWidgetItem *itemfree = new QTableWidgetItem(free);
    itemfree->setTextAlignment(Qt::AlignCenter);
    QTableWidgetItem *itemall = new QTableWidgetItem(all);
    itemall->setTextAlignment(Qt::AlignCenter);

    this->setItem(row, 0, itemname);
    this->setItem(row, 1, itemuse);
    this->setItem(row, 2, itemfree);
    this->setItem(row, 3, itemall);

    QProgressBar *bar = new QProgressBar;
    bar->setRange(0, 100);
    bar->setValue(percent);

    QString qss = QString("QProgressBar{background:%1;border-width:0px;border-radius:0px;text-align:center;}"
                          "QProgressBar::chunk{border-radius:0px;}").arg(bgColor.name());

    if (percent < 50) {
        qss += QString("QProgressBar{color:%1;}QProgressBar::chunk{background:%2;}").arg(textColor1.name()).arg(chunkColor1.name());
    } else if (percent < 90) {
        qss += QString("QProgressBar{color:%1;}QProgressBar::chunk{background:%2;}").arg(textColor2.name()).arg(chunkColor2.name());
    } else {
        qss += QString("QProgressBar{color:%1;}QProgressBar::chunk{background:%2;}").arg(textColor3.name()).arg(chunkColor3.name());
    }

    bar->setStyleSheet(qss);
    this->setCellWidget(row, 4, bar);
}

QSize DeviceSizeTable::sizeHint() const
{
    return QSize(500, 300);
}

QSize DeviceSizeTable::minimumSizeHint() const
{
    return QSize(200, 150);
}

QColor DeviceSizeTable::getBgColor() const
{
    return this->bgColor;
}

void DeviceSizeTable::setBgColor(const QColor &bgColor)
{
    if (this->bgColor != bgColor) {
        this->bgColor = bgColor;
        this->load();
    }
}

QColor DeviceSizeTable::getChunkColor1() const
{
    return this->chunkColor1;
}

void DeviceSizeTable::setChunkColor1(const QColor &chunkColor1)
{
    if (this->chunkColor1 != chunkColor1) {
        this->chunkColor1 = chunkColor1;
        this->load();
    }
}

QColor DeviceSizeTable::getChunkColor2() const
{
    return this->chunkColor2;
}

void DeviceSizeTable::setChunkColor2(const QColor &chunkColor2)
{
    if (this->chunkColor2 != chunkColor2) {
        this->chunkColor2 = chunkColor2;
        this->load();
    }
}

QColor DeviceSizeTable::getChunkColor3() const
{
    return this->chunkColor3;
}

void DeviceSizeTable::setChunkColor3(const QColor &chunkColor3)
{
    if (this->chunkColor3 != chunkColor3) {
        this->chunkColor3 = chunkColor3;
        this->load();
    }
}

QColor DeviceSizeTable::getTextColor1() const
{
    return this->textColor1;
}

void DeviceSizeTable::setTextColor1(const QColor &textColor1)
{
    if (this->textColor1 != textColor1) {
        this->textColor1 = textColor1;
        this->load();
    }
}

QColor DeviceSizeTable::getTextColor2() const
{
    return this->textColor2;
}

void DeviceSizeTable::setTextColor2(const QColor &textColor2)
{
    if (this->textColor2 != textColor2) {
        this->textColor2 = textColor2;
        this->load();
    }
}

QColor DeviceSizeTable::getTextColor3() const
{
    return this->textColor3;
}

void DeviceSizeTable::setTextColor3(const QColor &textColor3)
{
    if (this->textColor3 != textColor3) {
        this->textColor3 = textColor3;
        this->load();
    }
}

void DeviceSizeTable::load()
{
    //清空原有数据
    int row = this->rowCount();
    for (int i = 0; i < row; ++i) {
        this->removeRow(0);
    }

#ifdef Q_OS_WIN
    QFileInfoList list = QDir::drives();
    foreach (QFileInfo dir, list) {
        QString dirName = dir.absolutePath();
        LPCWSTR lpcwstrDriver = (LPCWSTR)dirName.utf16();
        ULARGE_INTEGER liFreeBytesAvailable, liTotalBytes, liTotalFreeBytes;

        if (GetDiskFreeSpaceEx(lpcwstrDriver, &liFreeBytesAvailable, &liTotalBytes, &liTotalFreeBytes)) {
            QString use = QString::number((double)(liTotalBytes.QuadPart - liTotalFreeBytes.QuadPart) / GB, 'f', 1);
            use += "G";
            QString free = QString::number((double) liTotalFreeBytes.QuadPart / GB, 'f', 1);
            free += "G";
            QString all = QString::number((double) liTotalBytes.QuadPart / GB, 'f', 1);
            all += "G";
            int percent = 100 - ((double)liTotalFreeBytes.QuadPart / liTotalBytes.QuadPart) * 100;
            insertSize(dirName, use, free, all, percent);
        }
    }
#elif defined(Q_OS_UNIX) && !defined(Q_OS_WASM)
    process->start("df", QStringList() << "-h");
#endif
}
```

### `./devicesizetable.h`

```cpp
﻿#ifndef DEVICESIZETABLE_H
#define DEVICESIZETABLE_H

/**
 * 本地存储空间大小控件 作者:feiyangqingyun(QQ:517216493) 2016-11-30
 * 1. 可自动加载本地存储设备的总容量/已用容量。
 * 2. 进度条显示已用容量。
 * 3. 支持所有操作系统。
 * 4. 增加U盘或者SD卡到达信号。
 */

#include <QTableWidget>

class QProcess;

#ifdef quc
class Q_DECL_EXPORT DeviceSizeTable : public QTableWidget
#else
class DeviceSizeTable : public QTableWidget
#endif

{
    Q_OBJECT

    Q_PROPERTY(QColor bgColor READ getBgColor WRITE setBgColor)
    Q_PROPERTY(QColor chunkColor1 READ getChunkColor1 WRITE setChunkColor1)
    Q_PROPERTY(QColor chunkColor2 READ getChunkColor2 WRITE setChunkColor2)
    Q_PROPERTY(QColor chunkColor3 READ getChunkColor3 WRITE setChunkColor3)

    Q_PROPERTY(QColor textColor1 READ getTextColor1 WRITE setTextColor1)
    Q_PROPERTY(QColor textColor2 READ getTextColor2 WRITE setTextColor2)
    Q_PROPERTY(QColor textColor3 READ getTextColor3 WRITE setTextColor3)

public:
    explicit DeviceSizeTable(QWidget *parent = 0);

private:
    QProcess *process;      //执行命令进程

    QColor bgColor;         //背景颜色
    QColor chunkColor1;     //进度颜色1
    QColor chunkColor2;     //进度颜色2
    QColor chunkColor3;     //进度颜色3

    QColor textColor1;      //文字颜色1
    QColor textColor2;      //文字颜色2
    QColor textColor3;      //文字颜色3

private slots:
    void readData();
    void checkSize(const QString &result, const QString &name);
    void insertSize(const QString &name, const QString &use, const QString &free, const QString &all, int percent);

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //获取和设置背景颜色
    QColor getBgColor() const;
    void setBgColor(const QColor &bgColor);

    //获取和设置进度颜色1
    QColor getChunkColor1() const;
    void setChunkColor1(const QColor &chunkColor1);

    //获取和设置进度颜色2
    QColor getChunkColor2() const;
    void setChunkColor2(const QColor &chunkColor2);

    //获取和设置进度颜色3
    QColor getChunkColor3() const;
    void setChunkColor3(const QColor &chunkColor3);

    //获取和设置文字颜色1
    QColor getTextColor1() const;
    void setTextColor1(const QColor &textColor1);

    //获取和设置文字颜色2
    QColor getTextColor2() const;
    void setTextColor2(const QColor &textColor2);

    //获取和设置文字颜色3
    QColor getTextColor3() const;
    void setTextColor3(const QColor &textColor3);

public Q_SLOTS:
    //载入容量
    void load();

Q_SIGNALS:
    void sdcardReceive(const QString &sdcardName);
    void udiskReceive(const QString &udiskName);
};

#endif // DEVICESIZETABLE_H
```

### `./frmdevicesizetable.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmdevicesizetable.h"
#include "ui_frmdevicesizetable.h"

frmDeviceSizeTable::frmDeviceSizeTable(QWidget *parent) : QWidget(parent), ui(new Ui::frmDeviceSizeTable)
{
    ui->setupUi(this);
    ui->tableWidget->verticalHeader()->setDefaultSectionSize(25);
}

frmDeviceSizeTable::~frmDeviceSizeTable()
{
    delete ui;
}
```

### `./frmdevicesizetable.h`

```cpp
﻿#ifndef FRMDEVICESIZETABLE_H
#define FRMDEVICESIZETABLE_H

#include <QWidget>

namespace Ui {
class frmDeviceSizeTable;
}

class frmDeviceSizeTable : public QWidget
{
    Q_OBJECT

public:
    explicit frmDeviceSizeTable(QWidget *parent = 0);
    ~frmDeviceSizeTable();

private:
    Ui::frmDeviceSizeTable *ui;
};

#endif // FRMDEVICESIZETABLE_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmdevicesizetable.h"
#include <QApplication>
#include <QTextCodec>

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

    frmDeviceSizeTable w;
    w.setWindowTitle("磁盘容量 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmdevicesizetable.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmDeviceSizeTable</class>
 <widget class="QWidget" name="frmDeviceSizeTable">
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
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="DeviceSizeTable" name="tableWidget">
     <row/>
     <row/>
     <row/>
     <row/>
     <row/>
     <column/>
     <column/>
     <column/>
     <column/>
     <column/>
     <item row="0" column="0"/>
     <item row="0" column="1">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="0" column="2">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="0" column="3">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="1" column="0"/>
     <item row="1" column="1">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="1" column="2">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="1" column="3">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="2" column="0"/>
     <item row="2" column="1">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="2" column="2">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="2" column="3">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="3" column="0"/>
     <item row="3" column="1">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="3" column="2">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="3" column="3">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="4" column="0"/>
     <item row="4" column="1">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="4" column="2">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
     <item row="4" column="3">
      <property name="textAlignment">
       <set>AlignCenter</set>
      </property>
     </item>
    </widget>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>DeviceSizeTable</class>
   <extends>QTableWidget</extends>
   <header>devicesizetable.h</header>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
