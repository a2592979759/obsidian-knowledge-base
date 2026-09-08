---
tags:
  - Qt
  - 控件相关
---
# CPU内存控件 (`cpumemorylabel`)

> CPU内存控件

## 效果图

![[QWidgetDemo_assert/cpumemorylabel.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/cpumemorylabel`

## 技术要点

**用到的 Qt 类**: QString QTextCodec QLabel QWidget QTimer QProcess QSize QStringList QSysInfo QLatin1String QFont QApplication QVBoxLayout

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./cpumemorylabel.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "cpumemorylabel.h"
#include "qtimer.h"
#include "qprocess.h"
#include "qdebug.h"

#ifdef Q_OS_WIN
#ifndef _WIN32_WINNT
#define _WIN32_WINNT 0x502
#endif
#include "windows.h"
#endif

#define MB (1024 * 1024)
#define KB (1024)

CpuMemoryLabel::CpuMemoryLabel(QWidget *parent) : QLabel(parent)
{
    totalNew = idleNew = totalOld = idleOld = 0;

    cpuPercent = 0;
    memoryPercent = 0;
    memoryAll = 0;
    memoryUse = 0;

    //获取CPU占用情况定时器
    timerCPU = new QTimer(this);
    connect(timerCPU, SIGNAL(timeout()), this, SLOT(getCPU()));

    //获取内存占用情况定时器
    timerMemory = new QTimer(this);
    connect(timerMemory, SIGNAL(timeout()), this, SLOT(getMemory()));

    //执行命令获取
#ifndef Q_OS_WASM
    process = new QProcess(this);
    connect(process, SIGNAL(readyRead()), this, SLOT(readData()));
#endif

    showText = true;
}

CpuMemoryLabel::~CpuMemoryLabel()
{
    this->stop();
}

void CpuMemoryLabel::start(int interval)
{
    this->getCPU();
    this->getMemory();

    if (!timerCPU->isActive()) {
        timerCPU->start(interval);
    }
    if (!timerMemory->isActive()) {
        timerMemory->start(interval + 1000);
    }
}

void CpuMemoryLabel::stop()
{
#ifndef Q_OS_WASM
    process->close();
#endif

    if (timerCPU->isActive()) {
        timerCPU->stop();
    }
    if (timerMemory->isActive()) {
        timerMemory->stop();
    }
}

void CpuMemoryLabel::getCPU()
{
#ifdef Q_OS_WIN
#if 0
    static FILETIME lastIdleTime;
    static FILETIME lastKernelTime;
    static FILETIME lastUserTime;

    FILETIME newIdleTime;
    FILETIME newKernelTime;
    FILETIME newUserTime;

    //采用GetSystemTimes获取的CPU占用和任务管理器的不一致
    GetSystemTimes(&newIdleTime, &newKernelTime, &newUserTime);

    int offset = 31;
    quint64 a, b;
    quint64 idle, kernel, user;

    a = (lastIdleTime.dwHighDateTime << offset) | lastIdleTime.dwLowDateTime;
    b = (newIdleTime.dwHighDateTime << offset) | newIdleTime.dwLowDateTime;
    idle = b - a;

    a = (lastKernelTime.dwHighDateTime << offset) | lastKernelTime.dwLowDateTime;
    b = (newKernelTime.dwHighDateTime << offset) | newKernelTime.dwLowDateTime;
    kernel = b - a;

    a = (lastUserTime.dwHighDateTime << offset) | lastUserTime.dwLowDateTime;
    b = (newUserTime.dwHighDateTime << offset) | newUserTime.dwLowDateTime;
    user = b - a;

    cpuPercent = float(kernel + user - idle) * 100 / float(kernel + user);

    lastIdleTime = newIdleTime;
    lastKernelTime = newKernelTime;
    lastUserTime = newUserTime;
    this->setData();
#else
    //获取系统版本区分win10
    bool win10 = false;
#if (QT_VERSION >= QT_VERSION_CHECK(5,4,0))
    win10 = (QSysInfo::productVersion().mid(0, 2).toInt() >= 10);
#else
    win10 = (QSysInfo::WindowsVersion >= 192);
#endif

    QString cmd = "\\Processor(_Total)\\% Processor Time";
    if (win10) {
        cmd = "\\Processor Information(_Total)\\% Processor Utility";
    }

    if (process->state() == QProcess::NotRunning) {
        process->start("typeperf", QStringList() << cmd);
    }
#endif

#elif defined(Q_OS_UNIX) && !defined(Q_OS_WASM)
    if (process->state() == QProcess::NotRunning) {
        totalNew = idleNew = 0;
        process->start("cat", QStringList() << "/proc/stat");
    }
#endif
}

void CpuMemoryLabel::getMemory()
{
#ifdef Q_OS_WIN
    MEMORYSTATUSEX statex;
    statex.dwLength = sizeof(statex);
    GlobalMemoryStatusEx(&statex);
    memoryPercent = statex.dwMemoryLoad;
    memoryAll = statex.ullTotalPhys / MB;
    memoryFree = statex.ullAvailPhys / MB;
    memoryUse = memoryAll - memoryFree;
    this->setData();

#elif defined(Q_OS_UNIX) && !defined(Q_OS_WASM)
    if (process->state() == QProcess::NotRunning) {
        process->start("cat", QStringList() << "/proc/meminfo");
    }
#endif
}

void CpuMemoryLabel::readData()
{
#ifdef Q_OS_WIN
    while (!process->atEnd()) {
        QString s = QLatin1String(process->readLine());
        s = s.split(",").last();
        s.replace("\r", "");
        s.replace("\n", "");
        s.replace("\"", "");
        if (!s.isEmpty() && s.length() < 10) {
            cpuPercent = qRound(s.toFloat());
        }
    }
#elif defined(Q_OS_UNIX) && !defined(Q_OS_WASM)
    while (!process->atEnd()) {
        QString s = QLatin1String(process->readLine());
        if (s.startsWith("cpu")) {
            QStringList list = s.split(" ");
            idleNew = list.at(5).toUInt();
            foreach (QString value, list) {
                totalNew += value.toUInt();
            }

            quint64 total = totalNew - totalOld;
            quint64 idle = idleNew - idleOld;
            cpuPercent = 100 * (total - idle) / total;
            totalOld = totalNew;
            idleOld = idleNew;
            break;
        } else if (s.startsWith("MemTotal")) {
            s.replace(" ", "");
            s = s.split(":").at(1);
            memoryAll = s.left(s.length() - 3).toUInt() / KB;
        } else if (s.startsWith("MemFree")) {
            s.replace(" ", "");
            s = s.split(":").at(1);
            memoryFree = s.left(s.length() - 3).toUInt() / KB;
        } else if (s.startsWith("Buffers")) {
            s.replace(" ", "");
            s = s.split(":").at(1);
            memoryFree += s.left(s.length() - 3).toUInt() / KB;
        } else if (s.startsWith("Cached")) {
            s.replace(" ", "");
            s = s.split(":").at(1);
            memoryFree += s.left(s.length() - 3).toUInt() / KB;
            memoryUse = memoryAll - memoryFree;
            memoryPercent = 100 * memoryUse / memoryAll;
            break;
        }
    }
#endif
    this->setData();
}

void CpuMemoryLabel::setData()
{
    //cpuPercent = (cpuPercent < 0 ? 0 : cpuPercent);
    cpuPercent = (cpuPercent > 100 ? 0 : cpuPercent);
    QString msg = QString("CPU %1%  Mem %2% ( 已用 %3 MB / 共 %4 MB )").arg(cpuPercent).arg(memoryPercent).arg(memoryUse).arg(memoryAll);
    if (showText) {
        this->setText(msg);
    }

    Q_EMIT textChanged(msg);
    Q_EMIT valueChanged(cpuPercent, memoryPercent, memoryAll, memoryUse, memoryFree);
}

QSize CpuMemoryLabel::sizeHint() const
{
    return QSize(300, 30);
}

QSize CpuMemoryLabel::minimumSizeHint() const
{
    return QSize(30, 10);
}

bool CpuMemoryLabel::getShowText() const
{
    return this->showText;
}

void CpuMemoryLabel::setShowText(bool showText)
{
    this->showText = showText;
}
```

### `./cpumemorylabel.h`

```cpp
﻿#ifndef CPUMEMORYLABEL_H
#define CPUMEMORYLABEL_H

/**
 * CPU内存显示控件 作者:feiyangqingyun(QQ:517216493) 2016-11-18
 * 1. 实时显示当前CPU占用率。
 * 2. 实时显示内存使用情况。
 * 3. 包括共多少内存、已使用多少内存。
 * 4. 全平台通用，包括windows、linux、ARM。
 * 5. 发出信号通知占用率和内存使用情况等，以便自行显示到其他地方。
 */

#include <QLabel>

class QProcess;

#ifdef quc
class Q_DECL_EXPORT CpuMemoryLabel : public QLabel
#else
class CpuMemoryLabel : public QLabel
#endif

{
    Q_OBJECT

    Q_PROPERTY(bool showText READ getShowText WRITE setShowText)

public:
    explicit CpuMemoryLabel(QWidget *parent = 0);
    ~CpuMemoryLabel();

private:
    quint64 totalNew, idleNew, totalOld, idleOld;

    quint64 cpuPercent;     //CPU百分比
    quint64 memoryPercent;  //内存百分比
    quint64 memoryAll;      //所有内存
    quint64 memoryUse;      //已用内存
    quint64 memoryFree;     //空闲内存

    QTimer *timerCPU;       //定时器获取CPU信息
    QTimer *timerMemory;    //定时器获取内存信息
    QProcess *process;      //执行命令行

    bool showText;          //自己显示值

private slots:
    void getCPU();          //获取cpu
    void getMemory();       //获取内存
    void readData();        //读取数据
    void setData();         //设置数据

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //获取和设置是否显示文本
    bool getShowText() const;
    void setShowText(bool showText);

public Q_SLOTS:
    //开始启动服务
    void start(int interval);
    //停止服务
    void stop();

Q_SIGNALS:
    //文本改变信号
    void textChanged(const QString &text);
    //cpu和内存占用情况数值改变信号
    void valueChanged(quint64 cpuPercent, quint64 memoryPercent, quint64 memoryAll, quint64 memoryUse, quint64 memoryFree);
};

#endif // CPUMEMORYLABEL_H
```

### `./frmcpumemorylabel.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmcpumemorylabel.h"
#include "ui_frmcpumemorylabel.h"

frmCpuMemoryLabel::frmCpuMemoryLabel(QWidget *parent) : QWidget(parent), ui(new Ui::frmCpuMemoryLabel)
{
    ui->setupUi(this);
    this->initForm();
}

frmCpuMemoryLabel::~frmCpuMemoryLabel()
{
    delete ui;
}

void frmCpuMemoryLabel::initForm()
{
    QFont font;
    font.setPixelSize(16);
    setFont(font);

    QString qss1 = QString("QLabel{background-color:rgb(0,0,0);color:rgb(%1);}").arg("100,184,255");
    QString qss2 = QString("QLabel{background-color:rgb(0,0,0);color:rgb(%1);}").arg("255,107,107");
    QString qss3 = QString("QLabel{background-color:rgb(0,0,0);color:rgb(%1);}").arg("24,189,155");

    ui->label1->setStyleSheet(qss1);
    ui->label2->setStyleSheet(qss2);
    ui->label3->setStyleSheet(qss3);

    connect(ui->label1, SIGNAL(textChanged(QString)), ui->label2, SLOT(setText(QString)));
    connect(ui->label1, SIGNAL(valueChanged(quint64, quint64, quint64, quint64, quint64)),
            this, SLOT(valueChanged(quint64, quint64, quint64, quint64, quint64)));
    ui->label1->start(1000);
}

void frmCpuMemoryLabel::valueChanged(quint64 cpuPercent, quint64 memoryPercent, quint64 memoryAll, quint64 memoryUse, quint64 memoryFree)
{
    //重新组织文字
    QString msg = QString("CPU: %1%  内存: %2% ( %3 MB / %4 MB )").arg(cpuPercent).arg(memoryPercent).arg(memoryUse).arg(memoryAll);
    ui->label3->setText(msg);
}
```

### `./frmcpumemorylabel.h`

```cpp
﻿#ifndef FRMCPUMEMORYLABEL_H
#define FRMCPUMEMORYLABEL_H

#include <QWidget>

namespace Ui {
class frmCpuMemoryLabel;
}

class frmCpuMemoryLabel : public QWidget
{
    Q_OBJECT

public:
    explicit frmCpuMemoryLabel(QWidget *parent = 0);
    ~frmCpuMemoryLabel();

private:
    Ui::frmCpuMemoryLabel *ui;

private slots:
    void initForm();
    //cpu和内存占用情况数值改变信号
    void valueChanged(quint64 cpuPercent, quint64 memoryPercent, quint64 memoryAll, quint64 memoryUse, quint64 memoryFree);
};

#endif // FRMCPUMEMORYLABEL_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmcpumemorylabel.h"
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

    frmCpuMemoryLabel w;
    w.setWindowTitle("CPU内存显示控件 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmcpumemorylabel.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmCpuMemoryLabel</class>
 <widget class="QWidget" name="frmCpuMemoryLabel">
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
   <property name="spacing">
    <number>6</number>
   </property>
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
    <widget class="CpuMemoryLabel" name="label1">
     <property name="text">
      <string/>
     </property>
     <property name="alignment">
      <set>Qt::AlignCenter</set>
     </property>
    </widget>
   </item>
   <item>
    <widget class="QLabel" name="label2">
     <property name="text">
      <string/>
     </property>
     <property name="alignment">
      <set>Qt::AlignCenter</set>
     </property>
    </widget>
   </item>
   <item>
    <widget class="QLabel" name="label3">
     <property name="text">
      <string/>
     </property>
     <property name="alignment">
      <set>Qt::AlignCenter</set>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>CpuMemoryLabel</class>
   <extends>QLabel</extends>
   <header>cpumemorylabel.h</header>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
