---
tags:
  - Qt
  - 控件相关
  - 高质量
---
# 运行时间记录 (`saveruntime`)

> 运行时间记录

## 效果图

![[QWidgetDemo_assert/saveruntime.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/saveruntime`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QString QFile QDateTime QTextCodec QWidget QObject QStringList QTimer QDate QFrame QPushButton QTextStream QApplication QScopedPointer QHBoxLayout QTextEdit QVBoxLayout QCheckBox QFont QMutex

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmsaveruntime.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmsaveruntime.h"
#include "ui_frmsaveruntime.h"
#include "qfile.h"
#include "saveruntime.h"
#include "qdebug.h"

frmSaveRunTime::frmSaveRunTime(QWidget *parent) : QWidget(parent), ui(new Ui::frmSaveRunTime)
{
    ui->setupUi(this);
    //设置文件存储目录
    SaveRunTime::Instance()->setPath(qApp->applicationDirPath() + "/log");
}

frmSaveRunTime::~frmSaveRunTime()
{
    delete ui;
}

void frmSaveRunTime::on_checkBox_stateChanged(int arg1)
{
    if (arg1 == 0) {
        SaveRunTime::Instance()->stop();
    } else {
        SaveRunTime::Instance()->start();
    }
    on_btnOpen_clicked();
}

void frmSaveRunTime::on_btnAppend_clicked()
{
    SaveRunTime::Instance()->initLog();
    SaveRunTime::Instance()->appendLog();
    on_btnOpen_clicked();
}

void frmSaveRunTime::on_btnUpdate_clicked()
{
    SaveRunTime::Instance()->saveLog();
    on_btnOpen_clicked();
}

void frmSaveRunTime::on_btnOpen_clicked()
{
    QString path = qApp->applicationDirPath();
    QString name = qApp->applicationFilePath();
    QStringList list = name.split("/");
    name = list.at(list.count() - 1).split(".").at(0);

    QString fileName = QString("%1/log/%2_runtime_%3.txt").arg(path).arg(name).arg(QDate::currentDate().year());
    QFile file(fileName);
    if (file.open(QFile::ReadOnly | QFile::Text)) {
        ui->txtMain->setText(file.readAll());
    }
}
```

### `./frmsaveruntime.h`

```cpp
﻿#ifndef FRMSAVERUNTIME_H
#define FRMSAVERUNTIME_H

#include <QWidget>

namespace Ui {
class frmSaveRunTime;
}

class frmSaveRunTime : public QWidget
{
    Q_OBJECT

public:
    explicit frmSaveRunTime(QWidget *parent = 0);
    ~frmSaveRunTime();

private:
    Ui::frmSaveRunTime *ui;

private slots:
    void on_checkBox_stateChanged(int arg1);
    void on_btnAppend_clicked();
    void on_btnUpdate_clicked();
    void on_btnOpen_clicked();
};

#endif // FRMSAVERUNTIME_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmsaveruntime.h"
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

    frmSaveRunTime w;
    w.setWindowTitle("运行时间记录示例 V2022 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./saveruntime.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "saveruntime.h"
#include "qmutex.h"
#include "qdir.h"
#include "qfile.h"
#include "qapplication.h"
#include "qtimer.h"
#include "qtextstream.h"
#include "qstringlist.h"

QScopedPointer<SaveRunTime> SaveRunTime::self;
SaveRunTime *SaveRunTime::Instance()
{
    if (self.isNull()) {
        static QMutex mutex;
        QMutexLocker locker(&mutex);
        if (self.isNull()) {
            self.reset(new SaveRunTime);
        }
    }

    return self.data();
}

SaveRunTime::SaveRunTime(QObject *parent) : QObject(parent)
{
    path = qApp->applicationDirPath();
    QString str = qApp->applicationFilePath();
    QStringList list = str.split("/");
    name = list.at(list.count() - 1).split(".").at(0);

    saveInterval = 1 * 60 * 1000;
    startTime = QDateTime::currentDateTime();

    //存储运行时间定时器
    timerSave = new QTimer(this);
    timerSave->setInterval(saveInterval);
    connect(timerSave, SIGNAL(timeout()), this, SLOT(saveLog()));
}

SaveRunTime::~SaveRunTime()
{
    this->stop();
}

void SaveRunTime::getDiffValue(const QDateTime &startTime, const QDateTime &endTime, int &day, int &hour, int &minute)
{
    qint64 totalSeconds = startTime.secsTo(endTime);
    minute = (totalSeconds / 60) % 60;
    hour = totalSeconds / 3600;
    day = totalSeconds / (24 * 3600);
}

void SaveRunTime::start()
{
    if (timerSave->isActive()) {
        return;
    }

    //开始时间变量必须在这,在部分嵌入式系统上开机后的时间不准确比如是1970,而后会变成1999或者其他时间
    //会在getDiffValue函数执行很久很久
    startTime = QDateTime::currentDateTime();
    timerSave->start();

    initLog();
    appendLog();
    saveLog();
}

void SaveRunTime::stop()
{
    if (!timerSave->isActive()) {
        return;
    }

    timerSave->stop();
}

void SaveRunTime::newPath()
{
    //检查目录是否存在,不存在则先新建
    QDir dir(path);
    if (!dir.exists()) {
        dir.mkpath(path);
    }
}

void SaveRunTime::initLog()
{
    //判断当前年份的记事本文件是否存在,不存在则新建并且写入标题
    //存在则自动读取最后一行的id号  记事本文件格式内容
    //幢号    开始时间                结束时间                已运行时间
    //1      2016-01-01 12:33:33    2016-02-05 12:12:12     day: 0  hour: 0  minute: 0

    newPath();
    logFile = QString("%1/%2_runtime_%3.txt").arg(path).arg(name).arg(QDate::currentDate().year());
    QFile file(logFile);

    if (file.size() == 0) {
        if (file.open(QFile::WriteOnly | QFile::Text)) {
            QString strID = QString("%1\t").arg("编号");
            QString strStartTime = QString("%1\t\t").arg("开始时间");
            QString strEndTime = QString("%1\t\t").arg("结束时间");
            QString strRunTime = QString("%1").arg("已运行时间");
            QString line = strID + strStartTime + strEndTime + strRunTime;

            QTextStream stream(&file);
            stream << line << "\n";
            file.close();
            lastID = 0;
        }
    } else {
        if (file.open(QFile::ReadOnly)) {
            QString lastLine;
            while (!file.atEnd()) {
                lastLine = file.readLine();
            }

            file.close();
            QStringList list = lastLine.split("\t");
            lastID = list.at(0).toInt();
        }
    }

    lastID++;
}

void SaveRunTime::appendLog()
{
    newPath();
    logFile = QString("%1/%2_runtime_%3.txt").arg(path).arg(name).arg(QDate::currentDate().year());
    QFile file(logFile);

    //写入当前首次运行时间
    if (file.open(QFile::WriteOnly | QFile::Append | QFile::Text)) {
        QString strID = QString("%1\t").arg(lastID);
        QString strStartTime = QString("%1\t").arg(startTime.toString("yyyy-MM-dd HH:mm:ss"));
        QString strEndTime = QString("%1\t").arg(QDateTime::currentDateTime().toString("yyyy-MM-dd HH:mm:ss"));

        int day, hour, minute;
        getDiffValue(startTime, QDateTime::currentDateTime(), day, hour, minute);
        QString strRunTime = QString("%1 天 %2 时 %3 分").arg(day).arg(hour).arg(minute);
        QString line = strID + strStartTime + strEndTime + strRunTime;

        QTextStream stream(&file);
        stream << line << "\n";
        file.close();
    }
}

void SaveRunTime::saveLog()
{
    //每次保存都是将之前的所有文本读取出来,然后替换最后一行即可
    newPath();
    logFile = QString("%1/%2_runtime_%3.txt").arg(path).arg(name).arg(QDate::currentDate().year());
    QFile file(logFile);

    //如果日志文件不存在,则初始化一个日志文件
    if (file.size() == 0) {
        initLog();
        appendLog();
        return;
    }

    if (file.open(QFile::ReadWrite)) {
        //一行行读取到链表
        QStringList content;
        while (!file.atEnd()) {
            content.append(file.readLine());
        }

        //重新清空文件
        file.resize(0);
        //如果行数小于2则返回
        if (content.count() < 2) {
            file.close();
            return;
        }

        QString lastLine = content.last();
        QStringList list = lastLine.split("\t");

        //计算已运行时间
        int day, hour, minute;
        getDiffValue(startTime, QDateTime::currentDateTime(), day, hour, minute);
        QString strRunTime = QString("%1 天 %2 时 %3 分").arg(day).arg(hour).arg(minute);

        //重新拼接最后一行
        list[2] = QDateTime::currentDateTime().toString("yyyy-MM-dd HH:mm:ss");
        list[3] = strRunTime;
        lastLine = list.join("\t");

        //重新替换最后一行并写入新的数据
        content[content.count() - 1] = lastLine;

        QTextStream stream(&file);
        stream << content.join("") << "\n";
        file.close();
    }
}

void SaveRunTime::setPath(const QString &path)
{
    this->path = path;
}

void SaveRunTime::setName(const QString &name)
{
    this->name = name;
}

void SaveRunTime::setSaveInterval(int saveInterval)
{
    if (this->saveInterval != saveInterval) {
        this->saveInterval = saveInterval;
        timerSave->setInterval(saveInterval);
    }
}
```

### `./saveruntime.h`

```cpp
﻿#ifndef SAVERUNTIME_H
#define SAVERUNTIME_H

/**
 * 运行时间记录 作者:feiyangqingyun(QQ:517216493) 2016-12-16
 * 1. 可以启动和停止服务，在需要的时候启动。
 * 2. 可以指定日志文件存放目录。
 * 3. 可以指定时间日志输出间隔。
 * 4. 可以单独追加一条记录到日志文件。
 * 5. 日志为文本格式，清晰明了。
 */

#include <QObject>
#include <QDateTime>

class QTimer;

#ifdef quc
class Q_DECL_EXPORT SaveRunTime : public QObject
#else
class SaveRunTime : public QObject
#endif

{
    Q_OBJECT
public:
    static SaveRunTime *Instance();
    explicit SaveRunTime(QObject *parent = 0);
    ~SaveRunTime();

private:
    static QScopedPointer<SaveRunTime> self;

    //日志文件路径
    QString path;
    //日志文件名称
    QString name;

    //最后的编号
    int lastID;
    //保存间隔
    int saveInterval;
    //开始时间
    QDateTime startTime;
    //日志文件
    QString logFile;
    //保存文件定时器
    QTimer *timerSave;

private:
    //比较两个时间差值
    void getDiffValue(const QDateTime &startTime, const QDateTime &endTime, int &day, int &hour, int &minute);

public Q_SLOTS:
    //启动服务
    void start();
    //停止服务
    void stop();

    //新建目录
    void newPath();
    //初始化日志文件
    void initLog();
    //追加一条记录到日志文件
    void appendLog();
    //保存运行时间到日志文件
    void saveLog();

    //设置文件保存目录
    void setPath(const QString &path);
    //设置文件名称
    void setName(const QString &name);
    //设置保存间隔
    void setSaveInterval(int saveInterval);
};

#endif // SAVERUNTIME_H
```

### `./frmsaveruntime.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmSaveRunTime</class>
 <widget class="QWidget" name="frmSaveRunTime">
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
   <item>
    <widget class="QTextEdit" name="txtMain"/>
   </item>
   <item>
    <widget class="QFrame" name="frame">
     <property name="minimumSize">
      <size>
       <width>200</width>
       <height>0</height>
      </size>
     </property>
     <property name="maximumSize">
      <size>
       <width>200</width>
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
      <item>
       <widget class="QCheckBox" name="checkBox">
        <property name="text">
         <string>启动运行时间记录</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnAppend">
        <property name="minimumSize">
         <size>
          <width>130</width>
          <height>0</height>
         </size>
        </property>
        <property name="text">
         <string>插入一条记录</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnUpdate">
        <property name="minimumSize">
         <size>
          <width>130</width>
          <height>0</height>
         </size>
        </property>
        <property name="text">
         <string>更新一条记录</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnOpen">
        <property name="minimumSize">
         <size>
          <width>130</width>
          <height>0</height>
         </size>
        </property>
        <property name="text">
         <string>打开记录文件</string>
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
