---
tags:
  - Qt
  - 工具相关
---
# 秘钥测试程序 (`keydemo`)

> 秘钥测试程序

## 效果图

![[QWidgetDemo_assert/keydemo.jpg]]

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/keydemo`

## 技术要点

**用到的 Qt 类**: QTextCodec QString QWidget QObject QTimer QDateTime QMessageBox QFrame QFile QApplication QMutex QMutexLocker QStringList QDate QByteArray QLatin1String QHBoxLayout QListWidget QVBoxLayout QLineEdit

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./appkey.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "appkey.h"
#include "qmutex.h"
#include "qfile.h"
#include "qtimer.h"
#include "qdatetime.h"
#include "qapplication.h"
#include "qmessagebox.h"

AppKey *AppKey::self = NULL;
AppKey *AppKey::Instance()
{
    if (!self) {
        QMutex mutex;
        QMutexLocker locker(&mutex);
        if (!self) {
            self = new AppKey;
        }
    }

    return self;
}

AppKey::AppKey(QObject *parent) : QObject(parent)
{
    keyData = "";
    keyUseDate = false;
    keyDate = "2017-01-01";
    keyUseRun = false;
    keyRun = 1;
    keyUseCount = false;
    keyCount = 10;

    timer = new QTimer(this);
    timer->setInterval(1000);
    connect(timer, SIGNAL(timeout()), this, SLOT(checkTime()));
    startTime = QDateTime::currentDateTime();
}

void AppKey::start()
{
    //判断密钥文件是否存在,不存在则从资源文件复制出来,同时需要设置文件写权限
    QString keyName = qApp->applicationDirPath() + "/key.db";
    QFile keyFile(keyName);
    if (!keyFile.exists() || keyFile.size() == 0) {
        QMessageBox::critical(0, "错误", "密钥文件丢失,请联系供应商!");
        exit(0);
    }

    //读取密钥文件
    keyFile.open(QFile::ReadOnly);
    keyData = keyFile.readLine();
    keyFile.close();

    //将从注册码文件中的密文解密,与当前时间比较是否到期
    keyData = getXorEncryptDecrypt(keyData, 110);
    QStringList data = keyData.split("|");

    if (data.count() != 6) {
        QMessageBox::critical(0, "错误", "注册码文件已损坏,程序将自动关闭!");
        exit(0);
    }

    keyUseDate = (data.at(0) == "1");
    keyDate = data.at(1);
    keyUseRun = (data.at(2) == "1");
    keyRun = data.at(3).toInt();
    keyUseCount = (data.at(4) == "1");
    keyCount = data.at(5).toInt();

    //如果启用了时间限制
    if (keyUseDate) {
        QString nowDate = QDate::currentDate().toString("yyyy-MM-dd");
        if (nowDate > keyDate) {
            QMessageBox::critical(0, "错误", "软件已到期,请联系供应商更新注册码!");
            exit(0);
        }
    }

    //如果启用了运行时间显示
    if (keyUseRun) {
        timer->start();
    }
}

void AppKey::stop()
{
    timer->stop();
}

void AppKey::checkTime()
{
    //找出当前时间与首次启动时间比较
    QDateTime now = QDateTime::currentDateTime();
    if (startTime.secsTo(now) >= (keyRun * 60)) {
        QMessageBox::critical(0, "错误", "试运行时间已到,请联系供应商更新注册码!");
        exit(0);
    }
}

QString AppKey::getXorEncryptDecrypt(const QString &data, char key)
{
    //采用异或加密,也可以自行更改算法
    QByteArray buffer = data.toLatin1();
    int size = buffer.size();
    for (int i = 0; i < size; i++) {
        buffer[i] = buffer.at(i) ^ key;
    }

    return QLatin1String(buffer);
}

bool AppKey::checkCount(int count)
{
    if (keyUseCount) {
        if (count >= keyCount) {
            QMessageBox::critical(0, "错误", "设备数量超过限制,请联系供应商更新注册码!");
            return false;
        }
    }

    return true;
}
```

### `./appkey.h`

```cpp
﻿#ifndef APPKEY_H
#define APPKEY_H

#include <QObject>
#include <QDateTime>

class QTimer;

class AppKey : public QObject
{
    Q_OBJECT
public:
    static AppKey *Instance();
    explicit AppKey(QObject *parent = 0);

private:
    static AppKey *self;

    QString keyData;            //注册码密文
    bool keyUseDate;            //是否启用运行日期时间限制
    QString keyDate;            //到期时间字符串
    bool keyUseRun;             //是否启用可运行时间限制
    int keyRun;                 //可运行时间
    bool keyUseCount;           //是否启用设备数量限制
    int keyCount;               //设备限制数量

    QTimer *timer;              //定时器判断是否运行超时
    QDateTime startTime;        //程序启动时间

private slots:
    void checkTime();
    QString getXorEncryptDecrypt(const QString &data, char key);

public slots:
    void start();
    void stop();
    bool checkCount(int count);
};

#endif // APPKEY_H
```

### `./frmmain.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
#include "ui_frmmain.h"
#include "appkey.h"

frmMain::frmMain(QWidget *parent) : QWidget(parent), ui(new Ui::frmMain)
{
    ui->setupUi(this);
}

frmMain::~frmMain()
{
    delete ui;
}

void frmMain::on_btnAdd_clicked()
{
    QString name = ui->lineEdit->text().trimmed();
    ui->listWidget->addItem(name);

    //计算当前设备数量多少
    AppKey::Instance()->checkCount(ui->listWidget->count());
}
```

### `./frmmain.h`

```cpp
﻿#ifndef FRMMAIN_H
#define FRMMAIN_H

#include <QWidget>

namespace Ui {
class frmMain;
}

class frmMain : public QWidget
{
    Q_OBJECT

public:
    explicit frmMain(QWidget *parent = 0);
    ~frmMain();

private:
    Ui::frmMain *ui;

private slots:
    void on_btnAdd_clicked();
};

#endif // FRMMAIN_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
#include "appkey.h"
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

    //启动密钥服务类
    AppKey::Instance()->start();

    frmMain w;
    w.setWindowTitle("密钥使用示例 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmmain.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmMain</class>
 <widget class="QWidget" name="frmMain">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>frmMain</string>
  </property>
  <layout class="QHBoxLayout" name="horizontalLayout">
   <item>
    <widget class="QListWidget" name="listWidget"/>
   </item>
   <item>
    <widget class="QFrame" name="frame">
     <property name="minimumSize">
      <size>
       <width>150</width>
       <height>0</height>
      </size>
     </property>
     <property name="maximumSize">
      <size>
       <width>150</width>
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
       <widget class="QLineEdit" name="lineEdit">
        <property name="text">
         <string>测试设备</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnAdd">
        <property name="text">
         <string>添加</string>
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
          <height>486</height>
         </size>
        </property>
       </spacer>
      </item>
     </layout>
    </widget>
   </item>
  </layout>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <resources/>
 <connections/>
</ui>
```
