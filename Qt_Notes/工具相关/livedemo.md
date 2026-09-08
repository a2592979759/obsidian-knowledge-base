---
tags:
  - Qt
  - 工具相关
---
# 程序启动示例 (`livedemo`)

> 程序启动示例

## 效果图

![[QWidgetDemo_assert/livedemo.jpg]]

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/livedemo`

## 技术要点

**用到的 Qt 类**: QTextCodec QWidget QObject QString QUdpSocket QScopedPointer QApplication QTime QMutex QMutexLocker QStringList QByteArray QHostAddress QLatin1String QGridLayout QFont

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./applive.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "applive.h"
#include "qmutex.h"
#include "qudpsocket.h"
#include "qstringlist.h"
#include "qapplication.h"
#include "qdatetime.h"
#include "qdebug.h"

#define TIMEMS qPrintable(QTime::currentTime().toString("HH:mm:ss zzz"))

QScopedPointer<AppLive> AppLive::self;
AppLive *AppLive::Instance()
{
    if (self.isNull()) {
        QMutex mutex;
        QMutexLocker locker(&mutex);
        if (self.isNull()) {
            self.reset(new AppLive);
        }
    }

    return self.data();
}

AppLive::AppLive(QObject *parent) : QObject(parent)
{
    udpServer  = new QUdpSocket(this);

    QString name = qApp->applicationFilePath();
    QStringList list = name.split("/");
    appName = list.at(list.count() - 1).split(".").at(0);
}

void AppLive::readData()
{
    QByteArray tempData;

    do {
        tempData.resize(udpServer->pendingDatagramSize());
        QHostAddress sender;
        quint16 senderPort;
        udpServer->readDatagram(tempData.data(), tempData.size(), &sender, &senderPort);
        QString data = QLatin1String(tempData);

        if (data == "hello") {
            udpServer->writeDatagram(QString("%1OK").arg(appName).toLatin1(), sender, senderPort);
        }
    } while (udpServer->hasPendingDatagrams());
}

bool AppLive::start(int port)
{
    bool ok = udpServer->bind(port);
    if (ok) {
        connect(udpServer, SIGNAL(readyRead()), this, SLOT(readData()));
        qDebug() << TIMEMS << "Start AppLive Ok";
    }

    return ok;
}

void AppLive::stop()
{
    udpServer->abort();
    disconnect(udpServer, SIGNAL(readyRead()), this, SLOT(readData()));
}
```

### `./applive.h`

```cpp
#ifndef APPLIVE_H
#define APPLIVE_H

#include <QObject>

class QUdpSocket;

class AppLive : public QObject
{
	Q_OBJECT
public:
    static AppLive *Instance();
    explicit AppLive(QObject *parent = 0);	

private:
    static QScopedPointer<AppLive> self;
	QUdpSocket *udpServer;
    QString appName;

private slots:
    void readData();

public slots:
    bool start(int port);
    void stop();
};

#endif // APPLIVE_H
```

### `./frmmain.cpp`

```cpp
﻿#include "frmmain.h"
#include "ui_frmmain.h"

frmMain::frmMain(QWidget *parent) : QWidget(parent), ui(new Ui::frmMain)
{
    ui->setupUi(this);
}

frmMain::~frmMain()
{
    delete ui;
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
};

#endif // FRMMAIN_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
#include "applive.h"
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

    //启动守护程序服务类
    AppLive::Instance()->start(6666);

    frmMain w;
    w.setWindowTitle("守护程序使用示例 (QQ: 517216493 WX: feiyangqingyun)");
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
  <layout class="QGridLayout" name="gridLayout"/>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <resources/>
 <connections/>
</ui>
```
