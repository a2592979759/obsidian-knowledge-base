---
tags:
  - Qt
  - 其他相关
---
# NTP校时 (`ntpclient`)

> NTP校时

## 效果图

![[QWidgetDemo_assert/ntpclient.jpg]]

## 分类

- 类型: 其他相关 `other`
- 源码目录: `other/ntpclient`

## 技术要点

**用到的 Qt 类**: QDateTime QTextCodec QWidget QObject QUdpSocket QDate QTime QByteArray QString QLabel QLineEdit QApplication QScopedPointer QGridLayout QPushButton QFont QMutex QMutexLocker

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmntpclient.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmntpclient.h"
#include "ui_frmntpclient.h"
#include "ntpclient.h"
#include "qdebug.h"

frmNtpClient::frmNtpClient(QWidget *parent) : QWidget(parent), ui(new Ui::frmNtpClient)
{
    ui->setupUi(this);
    ui->txtNtpIP->setText("ntp1.aliyun.com");
    connect(NtpClient::Instance(), SIGNAL(receiveTime(QDateTime)), this, SLOT(receiveTime(QDateTime)));
}

frmNtpClient::~frmNtpClient()
{
    delete ui;
}

void frmNtpClient::on_btnGetTime_clicked()
{
    NtpClient::Instance()->setNtpIP(ui->txtNtpIP->text().trimmed());
    NtpClient::Instance()->getDateTime();
}

void frmNtpClient::receiveTime(const QDateTime &dateTime)
{
    ui->txtTime->setText(dateTime.toString("yyyy-MM-dd HH:mm:ss zzz"));
}
```

### `./frmntpclient.h`

```cpp
﻿#ifndef FRMNTPCLIENT_H
#define FRMNTPCLIENT_H

#include <QWidget>
#include <QDateTime>

namespace Ui {
class frmNtpClient;
}

class frmNtpClient : public QWidget
{
    Q_OBJECT

public:
    explicit frmNtpClient(QWidget *parent = 0);
    ~frmNtpClient();

private:
    Ui::frmNtpClient *ui;

private slots:
    void on_btnGetTime_clicked();
    void receiveTime(const QDateTime &dateTime);
};

#endif // FRMNTPCLIENT_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmntpclient.h"
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

    frmNtpClient w;
    w.setWindowTitle("Ntp校时 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./ntpclient.cpp`

```cpp
﻿#include "ntpclient.h"
#include "qmutex.h"
#include "qudpsocket.h"
#include "qdebug.h"

QScopedPointer<NtpClient> NtpClient::self;
NtpClient *NtpClient::Instance()
{
    if (self.isNull()) {
        static QMutex mutex;
        QMutexLocker locker(&mutex);
        if (self.isNull()) {
            self.reset(new NtpClient);
        }
    }

    return self.data();
}

NtpClient::NtpClient(QObject *parent) : QObject(parent)
{
    ntpIP = "ntp1.aliyun.com";

    udpSocket = new QUdpSocket(this);
    connect(udpSocket, SIGNAL(connected()), this, SLOT(sendData()));
    connect(udpSocket, SIGNAL(readyRead()), this, SLOT(readData()));
}

void NtpClient::sendData()
{
    qint8 LI = 0;
    qint8 VN = 3;
    qint8 MODE = 3;
    qint8 STRATUM = 0;
    qint8 POLL = 4;
    qint8 PREC = -6;
    QDateTime epoch(QDate(1900, 1, 1), QTime(0, 0, 0));
    qint32 second = quint32(epoch.secsTo(QDateTime::currentDateTime()));

    qint32 temp = 0;
    QByteArray timeRequest(48, 0);
    timeRequest[0] = (LI << 6) | (VN << 3) | (MODE);
    timeRequest[1] = STRATUM;
    timeRequest[2] = POLL;
    timeRequest[3] = PREC & 0xff;
    timeRequest[5] = 1;
    timeRequest[9] = 1;
    timeRequest[40] = (temp = (second & 0xff000000) >> 24);
    temp = 0;
    timeRequest[41] = (temp = (second & 0x00ff0000) >> 16);
    temp = 0;
    timeRequest[42] = (temp = (second & 0x0000ff00) >> 8);
    temp = 0;
    timeRequest[43] = ((second & 0x000000ff));

    udpSocket->write(timeRequest);
}

void NtpClient::setTime_t(uint secsSince1Jan1970UTC)
{

}

void NtpClient::readData()
{
    QByteArray newTime;
    QDateTime epoch(QDate(1900, 1, 1), QTime(0, 0, 0));
    QDateTime unixStart(QDate(1970, 1, 1), QTime(0, 0, 0));

    while (udpSocket->hasPendingDatagrams()) {
        newTime.resize(udpSocket->pendingDatagramSize());
        udpSocket->read(newTime.data(), newTime.size());
    };

    QByteArray transmitTimeStamp ;
    transmitTimeStamp = newTime.right(8);
    quint32 seconds = transmitTimeStamp.at(0);
    quint8 temp = 0;

    for (int i = 1; i <= 3; ++i) {
        seconds = (seconds << 8);
        temp = transmitTimeStamp.at(i);
        seconds = seconds + temp;
    }

    QDateTime dateTime;
    uint secs = seconds - epoch.secsTo(unixStart);

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
    dateTime.setSecsSinceEpoch(secs);
#else
    dateTime.setTime_t(secs);
#endif

#if defined(__arm__) || defined(__aarch64__)
#ifdef arma9
    dateTime = dateTime.addSecs(8 * 60 * 60);
#endif
#endif
    udpSocket->disconnectFromHost();

    //有些时候返回的数据可能有误或者解析不正确,导致填充的时间不正确
    if (dateTime.isValid()) {
        Q_EMIT receiveTime(dateTime);
    }
}

void NtpClient::setNtpIP(const QString &ntpIP)
{
    if (this->ntpIP != ntpIP) {
        this->ntpIP = ntpIP;
    }
}

void NtpClient::getDateTime()
{
    udpSocket->abort();
    udpSocket->connectToHost(ntpIP, 123);
}
```

### `./ntpclient.h`

```cpp
﻿#ifndef NTPCLIENT_H
#define NTPCLIENT_H

/**
 * Ntp校时类 作者:feiyangqingyun(QQ:517216493) 2017-02-16
 * 1. 可设置Ntp服务器IP地址。
 * 2. 推荐用默认的阿里云时间服务器 ntp1.aliyun.com
 * 3. 收到时间信号发出。
 * 4. 时间精确到秒。
 */

#include <QObject>
#include <QDateTime>
class QUdpSocket;

#ifdef quc
class Q_DECL_EXPORT NtpClient : public QObject
#else
class NtpClient : public QObject
#endif

{
    Q_OBJECT
public:
    static NtpClient *Instance();
    explicit NtpClient(QObject *parent = 0);

private:
    static QScopedPointer<NtpClient> self;
    QString ntpIP;
    QUdpSocket *udpSocket;

private slots:
    void readData();
    void sendData();
    void setTime_t(uint secsSince1Jan1970UTC);

public Q_SLOTS:
    //设置Ntp服务器IP
    void setNtpIP(const QString &ntpIP);
    //获取日期时间
    void getDateTime();

Q_SIGNALS:
    //收到时间返回
    void receiveTime(const QDateTime &dateTime);
};

#endif // NTPCLIENT_H
```

### `./frmntpclient.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmNtpClient</class>
 <widget class="QWidget" name="frmNtpClient">
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
  <widget class="QWidget" name="layoutWidget">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>10</y>
     <width>381</width>
     <height>51</height>
    </rect>
   </property>
   <layout class="QGridLayout" name="gridLayout">
    <item row="0" column="0">
     <widget class="QLabel" name="labNtpIP">
      <property name="text">
       <string>服务地址</string>
      </property>
     </widget>
    </item>
    <item row="0" column="1">
     <widget class="QLineEdit" name="txtNtpIP">
      <property name="text">
       <string/>
      </property>
     </widget>
    </item>
    <item row="0" column="2" rowspan="2">
     <widget class="QPushButton" name="btnGetTime">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>获取时间</string>
      </property>
     </widget>
    </item>
    <item row="1" column="0">
     <widget class="QLabel" name="labTime">
      <property name="text">
       <string>返回时间</string>
      </property>
     </widget>
    </item>
    <item row="1" column="1">
     <widget class="QLineEdit" name="txtTime"/>
    </item>
   </layout>
  </widget>
 </widget>
 <resources/>
 <connections/>
</ui>
```
