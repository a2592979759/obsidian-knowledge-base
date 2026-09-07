---
tags:
  - Qt
  - 工具相关
  - 高质量
---
# 串口调试助手 (`comtool`)

> 串口调试助手

## 效果图

![[QWidgetDemo_assert/comtool.jpg]]

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/comtool`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QString QStringList QColor QByteArray QMessageBox QFile QRect QWidget QFileDialog QList QComboBox QNetworkInterface QApplication QTimer QLabel QSize QVector QPushButton QSettings QTime

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./head.h`

```cpp
﻿#include <QtCore>
#include <QtGui>
#include <QtNetwork>

#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
#include <QtWidgets>
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
#include <QtCore5Compat>
#endif

#pragma execution_character_set("utf-8")
#define TIMEMS qPrintable(QTime::currentTime().toString("HH:mm:ss zzz"))
#define STRDATETIME qPrintable(QDateTime::currentDateTime().toString("yyyy-MM-dd-HH-mm-ss"))

#include "appconfig.h"
#include "appdata.h"
```

### `./main.cpp`

```cpp
﻿#include "frmcomtool.h"
#include "qthelper.h"

int main(int argc, char *argv[])
{
    QtHelper::initMain();
    QApplication a(argc, argv);
    a.setWindowIcon(QIcon(":/main.ico"));

    //设置编码以及加载中文翻译文件
    QtHelper::initAll();
    //读取配置文件
    AppConfig::ConfigFile = QString("%1/%2.ini").arg(QtHelper::appPath()).arg(QtHelper::appName());
    AppConfig::readConfig();

    AppData::Intervals << "1" << "10" << "20" << "50" << "100" << "200" << "300" << "500" << "1000" << "1500" << "2000" << "3000" << "5000" << "10000";
    AppData::readSendData();
    AppData::readDeviceData();

    frmComTool w;
    w.setWindowTitle("串口调试助手 V2024 (QQ: 517216493 WX: feiyangqingyun)");
    w.resize(900, 650);
    QtHelper::setFormInCenter(&w);
    w.show();

    return a.exec();
}
```

### `api/appconfig.cpp`

```cpp
﻿#include "appconfig.h"
#include "qthelper.h"

QString AppConfig::ConfigFile = "config.ini";
QString AppConfig::SendFileName = "send.txt";
QString AppConfig::DeviceFileName = "device.txt";

QString AppConfig::PortName = "COM1";
int AppConfig::BaudRate = 9600;
int AppConfig::DataBit = 8;
QString AppConfig::Parity = QString::fromUtf8("无");
double AppConfig::StopBit = 1;

bool AppConfig::HexSend = false;
bool AppConfig::HexReceive = false;
bool AppConfig::Debug = false;
bool AppConfig::AutoClear = false;

bool AppConfig::AutoSend = false;
int AppConfig::SendInterval = 1000;
bool AppConfig::AutoSave = false;
int AppConfig::SaveInterval = 5000;

QString AppConfig::Mode = "Tcp_Client";
QString AppConfig::ServerIP = "127.0.0.1";
int AppConfig::ServerPort = 6000;
int AppConfig::ListenPort = 6000;
int AppConfig::SleepTime = 100;
bool AppConfig::AutoConnect = false;

void AppConfig::readConfig()
{
    QSettings set(AppConfig::ConfigFile, QSettings::IniFormat);

    set.beginGroup("ComConfig");
    AppConfig::PortName = set.value("PortName", AppConfig::PortName).toString();
    AppConfig::BaudRate = set.value("BaudRate", AppConfig::BaudRate).toInt();
    AppConfig::DataBit = set.value("DataBit", AppConfig::DataBit).toInt();
    AppConfig::Parity = set.value("Parity", AppConfig::Parity).toString();
    AppConfig::StopBit = set.value("StopBit", AppConfig::StopBit).toInt();

    AppConfig::HexSend = set.value("HexSend", AppConfig::HexSend).toBool();
    AppConfig::HexReceive = set.value("HexReceive", AppConfig::HexReceive).toBool();
    AppConfig::Debug = set.value("Debug", AppConfig::Debug).toBool();
    AppConfig::AutoClear = set.value("AutoClear", AppConfig::AutoClear).toBool();

    AppConfig::AutoSend = set.value("AutoSend", AppConfig::AutoSend).toBool();
    AppConfig::SendInterval = set.value("SendInterval", AppConfig::SendInterval).toInt();
    AppConfig::AutoSave = set.value("AutoSave", AppConfig::AutoSave).toBool();
    AppConfig::SaveInterval = set.value("SaveInterval", AppConfig::SaveInterval).toInt();
    set.endGroup();

    set.beginGroup("NetConfig");
    AppConfig::Mode = set.value("Mode", AppConfig::Mode).toString();
    AppConfig::ServerIP = set.value("ServerIP", AppConfig::ServerIP).toString();
    AppConfig::ServerPort = set.value("ServerPort", AppConfig::ServerPort).toInt();
    AppConfig::ListenPort = set.value("ListenPort", AppConfig::ListenPort).toInt();
    AppConfig::SleepTime = set.value("SleepTime", AppConfig::SleepTime).toInt();
    AppConfig::AutoConnect = set.value("AutoConnect", AppConfig::AutoConnect).toBool();
    set.endGroup();

    //配置文件不存在或者不全则重新生成
    if (!QtHelper::checkIniFile(AppConfig::ConfigFile)) {
        writeConfig();
        return;
    }
}

void AppConfig::writeConfig()
{    
    QSettings set(AppConfig::ConfigFile, QSettings::IniFormat);

    set.beginGroup("ComConfig");
    set.setValue("PortName", AppConfig::PortName);
    set.setValue("BaudRate", AppConfig::BaudRate);
    set.setValue("DataBit", AppConfig::DataBit);
    set.setValue("Parity", AppConfig::Parity);
    set.setValue("StopBit", AppConfig::StopBit);

    set.setValue("HexSend", AppConfig::HexSend);
    set.setValue("HexReceive", AppConfig::HexReceive);
    set.setValue("Debug", AppConfig::Debug);
    set.setValue("AutoClear", AppConfig::AutoClear);

    set.setValue("AutoSend", AppConfig::AutoSend);
    set.setValue("SendInterval", AppConfig::SendInterval);
    set.setValue("AutoSave", AppConfig::AutoSave);
    set.setValue("SaveInterval", AppConfig::SaveInterval);
    set.endGroup();

    set.beginGroup("NetConfig");
    set.setValue("Mode", AppConfig::Mode);
    set.setValue("ServerIP", AppConfig::ServerIP);
    set.setValue("ServerPort", AppConfig::ServerPort);
    set.setValue("ListenPort", AppConfig::ListenPort);
    set.setValue("SleepTime", AppConfig::SleepTime);
    set.setValue("AutoConnect", AppConfig::AutoConnect);
    set.endGroup();
}
```

### `api/appconfig.h`

```cpp
﻿#ifndef APPCONFIG_H
#define APPCONFIG_H

#include "head.h"

class AppConfig
{
public:
    static QString ConfigFile;          //配置文件路径
    static QString SendFileName;        //发送配置文件名
    static QString DeviceFileName;      //模拟设备数据文件名

    static QString PortName;            //串口号
    static int BaudRate;                //波特率
    static int DataBit;                 //数据位
    static QString Parity;              //校验位
    static double StopBit;              //停止位

    static bool HexSend;                //16进制发送
    static bool HexReceive;             //16进制接收
    static bool Debug;                  //模拟设备
    static bool AutoClear;              //自动清空

    static bool AutoSend;               //自动发送
    static int SendInterval;            //自动发送间隔
    static bool AutoSave;               //自动保存
    static int SaveInterval;            //自动保存间隔

    static QString Mode;                //转换模式
    static QString ServerIP;            //服务器IP
    static int ServerPort;              //服务器端口
    static int ListenPort;              //监听端口
    static int SleepTime;               //延时时间
    static bool AutoConnect;            //自动重连

    //读写配置参数
    static void readConfig();           //读取配置参数
    static void writeConfig();          //写入配置参数
};

#endif // APPCONFIG_H
```

### `api/appdata.cpp`

```cpp
﻿#include "appdata.h"
#include "qthelper.h"

QStringList AppData::Intervals = QStringList();
QStringList AppData::Datas = QStringList();
QStringList AppData::Keys = QStringList();
QStringList AppData::Values = QStringList();

QString AppData::SendFileName = "send.txt";
void AppData::readSendData()
{
    //读取发送数据列表
    AppData::Datas.clear();
    QString fileName = QString("%1/%2").arg(QtHelper::appPath()).arg(AppData::SendFileName);
    QFile file(fileName);
    if (file.size() > 0 && file.open(QFile::ReadOnly | QIODevice::Text)) {
        while (!file.atEnd()) {
            QString line = file.readLine();
            line = line.trimmed();
            line = line.replace("\r", "");
            line = line.replace("\n", "");
            if (!line.isEmpty()) {
                AppData::Datas.append(line);
            }
        }

        file.close();
    }

    //没有的时候主动添加点免得太空
    if (AppData::Datas.count() == 0) {
        AppData::Datas << "16 FF 01 01 E0 E1" << "16 FF 01 01 E1 E2";
    }
}

QString AppData::DeviceFileName = "device.txt";
void AppData::readDeviceData()
{
    //读取转发数据列表
    AppData::Keys.clear();
    AppData::Values.clear();
    QString fileName = QString("%1/%2").arg(QtHelper::appPath()).arg(AppData::DeviceFileName);
    QFile file(fileName);
    if (file.size() > 0 && file.open(QFile::ReadOnly | QIODevice::Text)) {
        while (!file.atEnd()) {
            QString line = file.readLine();
            line = line.trimmed();
            line = line.replace("\r", "");
            line = line.replace("\n", "");
            if (!line.isEmpty()) {
                QStringList list = line.split(";");
                QString key = list.at(0);
                QString value;
                for (int i = 1; i < list.count(); i++) {
                    value += QString("%1;").arg(list.at(i));
                }

                //去掉末尾分号
                value = value.mid(0, value.length() - 1);
                AppData::Keys.append(key);
                AppData::Values.append(value);
            }
        }

        file.close();
    }
}

void AppData::saveData(const QString &data)
{
    if (data.length() <= 0) {
        return;
    }

    QString fileName = QString("%1/%2.txt").arg(QtHelper::appPath()).arg(STRDATETIME);
    QFile file(fileName);
    if (file.open(QFile::WriteOnly | QFile::Text)) {
        file.write(data.toUtf8());
        file.close();
    }
}

void AppData::loadIP(QComboBox *cbox)
{
    //获取本机所有IP
    static QStringList ips;
    if (ips.count() == 0) {
#ifdef emsdk
        ips << "127.0.0.1";
#else
        QList<QNetworkInterface> netInterfaces = QNetworkInterface::allInterfaces();
        foreach (const QNetworkInterface  &netInterface, netInterfaces) {
            //移除虚拟机和抓包工具的虚拟网卡
            QString humanReadableName = netInterface.humanReadableName().toLower();
            if (humanReadableName.startsWith("vmware network adapter") || humanReadableName.startsWith("npcap loopback adapter")) {
                continue;
            }

            //过滤当前网络接口
            bool flag = (netInterface.flags() == (QNetworkInterface::IsUp | QNetworkInterface::IsRunning | QNetworkInterface::CanBroadcast | QNetworkInterface::CanMulticast));
            if (flag) {
                QList<QNetworkAddressEntry> addrs = netInterface.addressEntries();
                foreach (QNetworkAddressEntry addr, addrs) {
                    //只取出IPV4的地址
                    if (addr.ip().protocol() == QAbstractSocket::IPv4Protocol) {
                        QString ip4 = addr.ip().toString();
                        if (ip4 != "127.0.0.1") {
                            ips << ip4;
                        }
                    }
                }
            }
        }
#endif
    }

    cbox->clear();
    cbox->addItems(ips);
    if (!ips.contains("127.0.0.1")) {
        cbox->addItem("127.0.0.1");
    }
}
```

### `api/appdata.h`

```cpp
﻿#ifndef APPDATA_H
#define APPDATA_H

#include "head.h"

class AppData
{
public:
    //全局变量
    static QStringList Intervals;
    static QStringList Datas;
    static QStringList Keys;
    static QStringList Values;

    //读取发送数据列表
    static QString SendFileName;
    static void readSendData();

    //读取转发数据列表
    static QString DeviceFileName;
    static void readDeviceData();

    //保存数据到文件
    static void saveData(const QString &data);

    //添加网卡IP地址到下拉框
    static void loadIP(QComboBox *cbox);
};

#endif // APPDATA_H
```

### `api/qthelper.cpp`

```cpp
﻿#include "qthelper.h"
#include "qnetworkinterface.h"
#include "qnetworkproxy.h"

#define TIMEMS qPrintable(QTime::currentTime().toString("HH:mm:ss zzz"))

bool QtHelper::useRatio = true;

//full指宽屏/就是将所有屏幕拼接在一起
QList<QRect> QtHelper::getScreenRects(bool available, bool full)
{
    QRect rect;
    QList<QRect> rects;
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    QList<QScreen *> screens = qApp->screens();
    int screenCount = screens.count();
    for (int i = 0; i < screenCount; ++i) {
        QScreen *screen = screens.at(i);
        if (full) {
            rect = (available ? screen->availableVirtualGeometry() : screen->virtualGeometry());
        } else {
            rect = (available ? screen->availableGeometry() : screen->geometry());
        }

        //需要根据缩放比来重新调整宽高
        qreal ratio = (QtHelper::useRatio ? screen->devicePixelRatio() : 1);
        rect.setWidth(rect.width() * ratio);
        rect.setHeight(rect.height() * ratio);
        rects << rect;
    }
#else
    QDesktopWidget *desk = qApp->desktop();
    int screenCount = desk->screenCount();
    for (int i = 0; i < screenCount; ++i) {
        if (full) {
            rect = (available ? desk->geometry() : desk->geometry());
        } else {
            rect = (available ? desk->availableGeometry(i) : desk->screenGeometry(i));
        }

        rects << rect;
    }
#endif
    return rects;
}

int QtHelper::getScreenIndex()
{
    //需要对多个屏幕进行处理
    int screenIndex = 0;
    QList<QRect> rects = getScreenRects(false);
    int count = rects.count();
    for (int i = 0; i < count; ++i) {
        //找到当前鼠标所在屏幕
        QPoint pos = QCursor::pos();
        if (rects.at(i).contains(pos)) {
            screenIndex = i;
            break;
        }
    }

    return screenIndex;
}

QRect QtHelper::getScreenRect(bool available, bool full)
{
    int screenIndex = getScreenIndex();
    QList<QRect> rects = getScreenRects(available, full);
    return rects.at(screenIndex);
}

qreal QtHelper::getScreenRatio(int index, bool devicePixel)
{
    qreal ratio = 1.0;
    //索引-1则自动获取鼠标所在当前屏幕
    int screenIndex = (index == -1 ? getScreenIndex() : index);
#if (QT_VERSION >= QT_VERSION_CHECK(5,5,0))
    QScreen *screen = qApp->screens().at(screenIndex);
    if (devicePixel) {
        //需要开启 AA_EnableHighDpiScaling 属性才能正常获取
        ratio = screen->devicePixelRatio() * 96;
    } else {
        ratio = screen->logicalDotsPerInch();
    }
#else
    //Qt4不能动态识别缩放更改后的值
    ratio = qApp->desktop()->screen(screenIndex)->logicalDpiX();
#endif
    return ratio / 96;
}

QRect QtHelper::checkCenterRect(QRect &rect, bool available)
{
    QRect deskRect = QtHelper::getScreenRect(available);
    int formWidth = rect.width();
    int formHeight = rect.height();
    int deskWidth = deskRect.width();
    int deskHeight = deskRect.height();
    int formX = deskWidth / 2 - formWidth / 2 + deskRect.x();
    int formY = deskHeight / 2 - formHeight / 2;
    rect = QRect(formX, formY, formWidth, formHeight);
    return deskRect;
}

int QtHelper::deskWidth()
{
    return getScreenRect().width();
}

int QtHelper::deskHeight()
{
    return getScreenRect().height();
}

QSize QtHelper::deskSize()
{
    return getScreenRect().size();
}

QWidget *QtHelper::centerBaseForm = 0;
void QtHelper::setFormInCenter(QWidget *form)
{
    int formWidth = form->width();
    int formHeight = form->height();

    //如果=0表示采用系统桌面屏幕为参照
    QRect rect;
    if (centerBaseForm == 0) {
        rect = getScreenRect();
    } else {
        rect = centerBaseForm->geometry();
    }

    int deskWidth = rect.width();
    int deskHeight = rect.height();
    QPoint movePoint(deskWidth / 2 - formWidth / 2 + rect.x(), deskHeight / 2 - formHeight / 2 + rect.y());
    form->move(movePoint);
}

void QtHelper::showForm(QWidget *form)
{
    setFormInCenter(form);
    form->show();

    //判断宽高是否超过了屏幕分辨率,超过了则最大化显示
    //qDebug() << TIMEMS << form->size() << deskSize();
    if (form->width() + 20 > deskWidth() || form->height() + 50 > deskHeight()) {
        QMetaObject::invokeMethod(form, "showMaximized", Qt::QueuedConnection);
    }
}

QString QtHelper::appName()
{
    //没有必要每次都获取,只有当变量为空时才去获取一次
    static QString name;
    if (name.isEmpty()) {
        name = qApp->applicationFilePath();
        //下面的方法主要为了过滤安卓的路径 lib程序名_armeabi-v7a/lib程序名_arm64-v8a
        QStringList list = name.split("/");
        name = list.at(list.count() - 1).split(".").at(0);
        name.replace("_armeabi-v7a", "");
        name.replace("_arm64-v8a", "");
    }

    return name;
}

QString QtHelper::appPath()
{
    static QString path;
    if (path.isEmpty()) {
#ifdef Q_OS_ANDROID
        //默认安卓根目录
        path = "/storage/emulated/0";
        //带上程序名称作为目录 前面加个0方便排序
        path = path + "/0" + appName();
#else
        path = qApp->applicationDirPath();
#endif
    }

    return path;
}

void QtHelper::getCurrentInfo(char *argv[], QString &path, QString &name)
{
    //必须用fromLocal8Bit保证中文路径正常
    QString argv0 = QString::fromLocal8Bit(argv[0]);
    QFileInfo file(argv0);
    path = file.path();
    name = file.baseName();
}

QString QtHelper::getIniValue(const QString &fileName, const QString &key)
{
    QString value;
    QFile file(fileName);
    if (file.open(QFile::ReadOnly | QFile::Text)) {
        while (!file.atEnd()) {
            QString line = file.readLine();
            if (line.startsWith(key)) {
                line = line.replace("\n", "");
                line = line.trimmed();
                value = line.split("=").last();
                break;
            }
        }
    }
    return value;
}

QString QtHelper::getIniValue(char *argv[], const QString &key, const QString &dir, const QString &file)
{
    QString path, name;
    QtHelper::getCurrentInfo(argv, path, name);
    //指定了名称则取指定的名称/防止程序重命名
    if (!file.isEmpty()) {
        name = file;
    }

    QString fileName = QString("%1/%2%3.ini").arg(path).arg(dir).arg(name);
    return getIniValue(fileName, key);
}

QStringList QtHelper::getLocalIPs()
{
    static QStringList ips;
    if (ips.count() == 0) {
#ifdef Q_OS_WASM
        ips << "127.0.0.1";
#else
        QList<QNetworkInterface> netInterfaces = QNetworkInterface::allInterfaces();
        foreach (QNetworkInterface netInterface, netInterfaces) {
            //移除虚拟机和抓包工具的虚拟网卡
            QString humanReadableName = netInterface.humanReadableName().toLower();
            if (humanReadableName.startsWith("vmware network adapter") || humanReadableName.startsWith("npcap loopback adapter")) {
                continue;
            }

            //过滤当前网络接口
            bool flag = (netInterface.flags() == (QNetworkInterface::IsUp | QNetworkInterface::IsRunning | QNetworkInterface::CanBroadcast | QNetworkInterface::CanMulticast));
            if (!flag) {
                continue;
            }

            QList<QNetworkAddressEntry> addrs = netInterface.addressEntries();
            foreach (QNetworkAddressEntry addr, addrs) {
                //只取出IPV4的地址
                if (addr.ip().protocol() != QAbstractSocket::IPv4Protocol) {
                    continue;
                }

                QString ip4 = addr.ip().toString();
                if (ip4 != "127.0.0.1") {
                    ips << ip4;
                }
            }
        }
#endif
    }

    return ips;
}

void QtHelper::initLocalIPs(QComboBox *cbox, const QString &defaultIP, bool local127)
{
    QStringList ips;
    if (local127) {
        ips << "127.0.0.1";
    }

    //添加本地网卡地址集合
    ips << QtHelper::getLocalIPs();

    //不在网卡地址列表中则取第一个
    QString ip = defaultIP;
    if (ips.count() > 0) {
        ip = ips.contains(ip) ? ip : ips.first();
    }

    //设置当前下拉框索引
    int index = ips.indexOf(ip);
    cbox->addItems(ips);
    cbox->setCurrentIndex(index < 0 ? 0 : index);

    //如果有文本框还要设置文本框的值
    if (cbox->lineEdit()) {
        cbox->lineEdit()->setText(ip);
    }
}

QList<QColor> QtHelper::colors = QList<QColor>();
QList<QColor> QtHelper::getColorList()
{
    //备用颜色集合 可以自行添加
    if (colors.count() == 0) {
        colors << QColor(0, 176, 180) << QColor(0, 113, 193) << QColor(255, 192, 0);
        colors << QColor(72, 103, 149) << QColor(185, 87, 86) << QColor(0, 177, 125);
        colors << QColor(214, 77, 84) << QColor(71, 164, 233) << QColor(34, 163, 169);
        colors << QColor(59, 123, 156) << QColor(162, 121, 197) << QColor(72, 202, 245);
        colors << QColor(0, 150, 121) << QColor(111, 9, 176) << QColor(250, 170, 20);
    }

    return colors;
}

QStringList QtHelper::getColorNames()
{
    QList<QColor> colors = getColorList();
    QStringList colorNames;
    foreach (QColor color, colors) {
        colorNames << color.name();
    }
    return colorNames;
}

QColor QtHelper::getRandColor()
{
    QList<QColor> colors = getColorList();
    int index = getRandValue(0, colors.count(), true);
    return colors.at(index);
}

void QtHelper::initRand()
{
    //初始化随机数种子
    QTime t = QTime::currentTime();
    srand(t.msec() + t.second() * 1000);
}

float QtHelper::getRandFloat(float min, float max)
{
    double diff = fabs(max - min);
    double value = (double)(rand() % 100) / 100;
    value = min + value * diff;
    return value;
}

double QtHelper::getRandValue(int min, int max, bool contansMin, bool contansMax)
{
    int value;
#if (QT_VERSION <= QT_VERSION_CHECK(5,10,0))
    //通用公式 a是起始值,n是整数的范围
    //int value = a + rand() % n;
    if (contansMin) {
        if (contansMax) {
            value = min + 0 + (rand() % (max - min + 1));
        } else {
            value = min + 0 + (rand() % (max - min + 0));
        }
    } else {
        if (contansMax) {
            value = min + 1 + (rand() % (max - min + 0));
        } else {
            value = min + 1 + (rand() % (max - min - 1));
        }
    }
#else
    if (contansMin) {
        if (contansMax) {
            value = QRandomGenerator::global()->bounded(min + 0, max + 1);
        } else {
            value = QRandomGenerator::global()->bounded(min + 0, max + 0);
        }
    } else {
        if (contansMax) {
            value = QRandomGenerator::global()->bounded(min + 1, max + 1);
        } else {
            value = QRandomGenerator::global()->bounded(min + 1, max + 0);
        }
    }
#endif
    return value;
}

QStringList QtHelper::getRandPoint(int count, float mainLng, float mainLat, float dotLng, float dotLat)
{
    //随机生成点坐标
    QStringList points;
    for (int i = 0; i < count; ++i) {
        //0.00881415 0.000442928
#if (QT_VERSION >= QT_VERSION_CHECK(5,10,0))
        float lngx = QRandomGenerator::global()->bounded(dotLng);
        float latx = QRandomGenerator::global()->bounded(dotLat);
#else
        float lngx = getRandFloat(dotLng / 100, dotLng);
        float latx = getRandFloat(dotLat / 100, dotLat);
#endif
        //需要先用精度转换成字符串
        QString lng2 = QString::number(mainLng + lngx, 'f', 8);
        QString lat2 = QString::number(mainLat + latx, 'f', 8);
        QString point = QString("%1,%2").arg(lng2).arg(lat2);
        points << point;
    }

    return points;
}

int QtHelper::getRangeValue(int oldMin, int oldMax, int oldValue, int newMin, int newMax)
{
    return (((oldValue - oldMin) * (newMax - newMin)) / (oldMax - oldMin)) + newMin;
}

QString QtHelper::getUuid()
{
    QString uuid = QUuid::createUuid().toString();
    uuid.replace("{", "");
    uuid.replace("}", "");
    return uuid;
}

QString QtHelper::checkPath(const QString &dirName)
{
    //相对路径需要补全完整路径
    QString path = dirName;
    if (path.startsWith("./")) {
        path.replace(".", "");
        path = QtHelper::appPath() + path;
    } else if (!path.startsWith("/") && !path.contains(":/")) {
        path = QtHelper::appPath() + "/" + path;
    }

    //目录不存在则新建
    QDir dir(path);
    if (!dir.exists()) {
        dir.mkpath(path);
    }

    return path;
}

QString QtHelper::checkFile(const QString &fileName)
{
    //将相对路径转换成完整路径
    QString name = fileName;
    if (name.startsWith("./")) {
        name = QtHelper::appPath() + name.mid(1, name.length());
    }

    return name;
}

void QtHelper::sleep(int msec, bool exec)
{
    if (msec <= 0) {
        return;
    }

    if (exec) {
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
        //阻塞方式延时(如果在主线程会卡住主界面)
        QThread::msleep(msec);
#else
        //非阻塞方式延时(不会卡住主界面/据说可能有问题)
        QTime endTime = QTime::currentTime().addMSecs(msec);
        while (QTime::currentTime() < endTime) {
            QCoreApplication::processEvents(QEventLoop::AllEvents, 100);
        }
#endif
    } else {
        //非阻塞方式延时(现在很多人推荐的方法)
        QEventLoop loop;
        QTimer::singleShot(msec, &loop, SLOT(quit()));
        loop.exec();
    }
}

void QtHelper::checkRun()
{
#ifdef Q_OS_WIN
    //延时1秒钟,等待程序释放完毕
    QtHelper::sleep(1000);
    //创建共享内存,判断是否已经运行程序
    static QSharedMemory mem(QtHelper::appName());
    if (!mem.create(1)) {
        QtHelper::showMessageBoxError("程序已运行, 软件将自动关闭!", 5, true);
        exit(0);
    }
#endif
}

void QtHelper::setStyle()
{
    //打印下所有内置风格的名字
    //qDebug() << TIMEMS << "QStyleFactory::keys" << QStyleFactory::keys();

    //设置内置风格
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    qApp->setStyle("Fusion");
#else
    qApp->setStyle("Cleanlooks");
#endif

    //设置指定颜色
    QPalette palette;
    palette.setBrush(QPalette::Window, QColor("#F0F0F0"));
    //qApp->setPalette(palette);
}

QFont QtHelper::addFont(const QString &fontFile, const QString &fontName)
{
    //判断图形字体是否存在,不存在则加入
    QFontDatabase fontDb;
    if (!fontDb.families().contains(fontName)) {
        int fontId = fontDb.addApplicationFont(fontFile);
        QStringList listName = fontDb.applicationFontFamilies(fontId);
        if (listName.count() == 0) {
            qDebug() << QString("load %1 error").arg(fontName);
        }
    }

    //再次判断是否包含字体名称防止加载失败
    QFont font;
    if (fontDb.families().contains(fontName)) {
        font = QFont(fontName);
#if (QT_VERSION >= QT_VERSION_CHECK(4,8,0))
        font.setHintingPreference(QFont::PreferNoHinting);
#endif
    }

    return font;
}

void QtHelper::setFont(int fontSize)
{
    //安卓套件在有些手机上默认字体不好看需要主动设置字体
    //网页套件需要主动加载中文字体才能正常显示中文
#if (defined Q_OS_ANDROID) || (defined Q_OS_WASM)
    QString fontFile = ":/font/DroidSansFallback.ttf";
    QString fontName = "Droid Sans Fallback";
    qApp->setFont(addFont(fontFile, fontName));
    return;
#endif

#ifdef __arm__
    fontSize = 25;
#endif

    QFont font;
    font.setFamily("MicroSoft Yahei");
    font.setPixelSize(fontSize);
    qApp->setFont(font);
}

void QtHelper::setCode(bool utf8)
{
    QTextCodec *codec = QTextCodec::codecForName("utf-8");
#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
    QTextCodec::setCodecForCStrings(codec);
    QTextCodec::setCodecForTr(codec);
#endif

    //如果想要控制台打印信息中文正常就注释掉这个设置/setCodecForLocale会影响toLocal8Bit函数
    if (utf8) {
        QTextCodec::setCodecForLocale(codec);
    }
}

void QtHelper::setTranslator(const QString &qmFile)
{
    //过滤下不存在的就不用设置了
    if (!QFile(qmFile).exists()) {
        return;
    }

    QTranslator *translator = new QTranslator(qApp);
    if (translator->load(qmFile)) {
        qApp->installTranslator(translator);
    }
}

#ifdef Q_OS_ANDROID
#if (QT_VERSION < QT_VERSION_CHECK(6,0,0))
#include <QtAndroidExtras>
#else
//Qt6中将相关类移到了core模块而且名字变了
#include <QtCore/private/qandroidextras_p.h>
#endif
#endif

bool QtHelper::checkPermission(const QString &permission)
{
#ifdef Q_OS_ANDROID
#if (QT_VERSION >= QT_VERSION_CHECK(5,10,0) && QT_VERSION < QT_VERSION_CHECK(6,0,0))
    QtAndroid::PermissionResult result = QtAndroid::checkPermission(permission);
    if (result == QtAndroid::PermissionResult::Denied) {
        QtAndroid::requestPermissionsSync(QStringList() << permission);
        result = QtAndroid::checkPermission(permission);
        if (result == QtAndroid::PermissionResult::Denied) {
            return false;
        }
    }
#else
    QFuture<QtAndroidPrivate::PermissionResult> result = QtAndroidPrivate::requestPermission(permission);
    if (result.resultAt(0) == QtAndroidPrivate::PermissionResult::Denied) {
        return false;
    }
#endif
#endif
    return true;
}

void QtHelper::initAndroidPermission()
{
    //可以把所有要动态申请的权限都写在这里
    checkPermission("android.permission.CALL_PHONE");
    checkPermission("android.permission.SEND_SMS");
    checkPermission("android.permission.CAMERA");
    checkPermission("android.permission.READ_EXTERNAL_STORAGE");
    checkPermission("android.permission.WRITE_EXTERNAL_STORAGE");

    checkPermission("android.permission.ACCESS_COARSE_LOCATION");
    checkPermission("android.permission.INTERNET");
    checkPermission("android.permission.BLUETOOTH");
    checkPermission("android.permission.BLUETOOTH_SCAN");
    checkPermission("android.permission.BLUETOOTH_CONNECT");
    checkPermission("android.permission.BLUETOOTH_ADVERTISE");
}

void QtHelper::initAll(bool utf8, bool style, bool tabCenter, int fontSize)
{
    //初始化安卓权限
    QtHelper::initAndroidPermission();
    //初始化随机数种子
    QtHelper::initRand();
    //设置编码
    QtHelper::setCode(utf8);
    //设置字体
    QtHelper::setFont(fontSize);

    //设置样式风格
    if (style) {
        QtHelper::setStyle();
    }

    //选项卡居中
    if (tabCenter) {
        qApp->setStyleSheet("QTabWidget::tab-bar{alignment:center;}");
    }

    //设置翻译文件支持多个
    QtHelper::setTranslator(":/qm/widgets.qm");
    QtHelper::setTranslator(":/qm/qt_zh_CN.qm");
    QtHelper::setTranslator(":/qm/designer_zh_CN.qm");

    //设置不使用本地系统环境代理配置
    QNetworkProxyFactory::setUseSystemConfiguration(false);
    //设置当前目录为程序可执行文件所在目录
    QDir::setCurrent(QtHelper::appPath());
    //Qt4中默认没有程序名称需要主动设置
#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
    qApp->setApplicationName(QtHelper::appName());
#endif
}

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
#ifdef webengine
#include "qquickwindow.h"
#endif
#endif

void QtHelper::initMain(bool desktopSettingsAware, bool use96Dpi, bool logCritical)
{
#ifdef Q_OS_LINUX
#ifndef Q_OS_ANDROID
    //Qt6开始默认用wayland/由于没有坐标系统导致无边框窗体不可用
    //qputenv("QT_QPA_PLATFORM", "xcb");
#endif
#endif

#ifdef webengine
    //谷歌浏览器禁用沙箱和安全策略以便跨域请求/否则可能报错 Access-Control-Allow-Origin
    qputenv("QTWEBENGINE_DISABLE_SANDBOX", "1");
    qputenv("QTWEBENGINE_CHROMIUM_FLAGS", "--disable-web-security");
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    //设置是否应用操作系统设置比如字体
    QApplication::setDesktopSettingsAware(desktopSettingsAware);
#endif

    //安卓必须启用高分屏
#ifdef Q_OS_ANDROID
    use96Dpi = false;
#endif

    QtHelper::useRatio = use96Dpi;
#if (QT_VERSION >= QT_VERSION_CHECK(5,6,0) && QT_VERSION < QT_VERSION_CHECK(6,0,0))
    //开启高分屏缩放支持
    if (!use96Dpi) {
        QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
        QApplication::setAttribute(Qt::AA_UseHighDpiPixmaps);
    }
#endif

#ifdef Q_OS_WIN
    if (use96Dpi) {
        //Qt6中AA_Use96Dpi没效果必须下面方式设置强制指定缩放DPI
        qputenv("QT_FONT_DPI", "96");
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
        //不应用任何缩放
        QApplication::setAttribute(Qt::AA_Use96Dpi);
#endif
    }
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(5,14,0))
    //高分屏缩放策略
    QApplication::setHighDpiScaleFactorRoundingPolicy(Qt::HighDpiScaleFactorRoundingPolicy::PassThrough);
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    //下面这行表示不打印Qt内部类的警告提示信息
    if (!logCritical) {
        QLoggingCategory::setFilterRules("*.critical=false\n*.warning=false");
    }
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(5,4,0))
    //设置opengl共享上下文
    QApplication::setAttribute(Qt::AA_ShareOpenGLContexts);
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
#ifdef webengine
    //修复openglwidget和webengine共存出现黑屏的bug
    QQuickWindow::setGraphicsApi(QSGRendererInterface::OpenGL);
#endif
#endif
}

void QtHelper::initOpenGL(quint8 type, bool checkCardEnable, bool checkVirtualSystem)
{
#if (QT_VERSION >= QT_VERSION_CHECK(5,4,0))
    //设置opengl模式 AA_UseDesktopOpenGL(默认) AA_UseOpenGLES AA_UseSoftwareOpenGL
    //在一些很旧的设备上或者对opengl支持很低的设备上需要使用AA_UseOpenGLES表示禁用硬件加速
    //如果开启的是AA_UseOpenGLES则无法使用硬件加速比如ffmpeg的dxva2
    if (type == 1) {
        QApplication::setAttribute(Qt::AA_UseDesktopOpenGL);
    } else if (type == 2) {
        QApplication::setAttribute(Qt::AA_UseOpenGLES);
    } else if (type == 3) {
        QApplication::setAttribute(Qt::AA_UseSoftwareOpenGL);
    }

    //检测显卡是否被禁用
    if (checkCardEnable && !isVideoCardEnable()) {
        QApplication::setAttribute(Qt::AA_UseOpenGLES);
    }

    //检测是否是虚拟机系统
    if (checkVirtualSystem && isVirtualSystem()) {
        QApplication::setAttribute(Qt::AA_UseOpenGLES);
    }
#endif
}

QString QtHelper::getStyle(const QString &qssFile)
{
    QString qss;
    QFile file(qssFile);
    if (file.open(QFile::ReadOnly)) {
#if 0
        qss = QLatin1String(file.readAll());
#else
        //用下面这种方式读取可以不用区分文件编码
        QStringList list;
        QTextStream stream(&file);
        while (!stream.atEnd()) {
            QString line;
            stream >> line;
            list << line;
        }
        qss = list.join("\n");
#endif
    }

    return qss.trimmed();
}

void QtHelper::setStyle(const QString &qssFile)
{
    QString qss = QtHelper::getStyle(qssFile);
    if (!qss.isEmpty()) {
        QString paletteColor = qss.mid(20, 7);
        qApp->setPalette(QPalette(QColor(paletteColor)));
        qApp->setStyleSheet(qss);
    }
}

QString QtHelper::doCmd(const QString &program, const QStringList &arguments, int timeout)
{
    QString result;
#ifndef Q_OS_WASM
    QProcess p;
    p.start(program, arguments);
    p.waitForFinished(timeout);
    result = QString::fromLocal8Bit(p.readAllStandardOutput());
    result.replace("\r", "");
    result.replace("\n", "");
    result = result.simplified();
    result = result.trimmed();
#endif
    return result;
}

bool QtHelper::isVideoCardEnable()
{
    QString result;
    bool videoCardEnable = true;

#if defined(Q_OS_WIN)
    QStringList args;
    //wmic path win32_VideoController get name,Status
    args << "path" << "win32_VideoController" << "get" << "name,Status";
    result = doCmd("wmic", args);
#endif

    //Name Status Intel(R) UHD Graphics 630 OK
    //Name Status Intel(R) UHD Graphics 630 Error
    if (result.contains("Error")) {
        videoCardEnable = false;
    }

    return videoCardEnable;
}

bool QtHelper::isVirtualSystem()
{
    QString result;
    bool virtualSystem = false;

#if defined(Q_OS_WIN)
    QStringList args;
    //wmic computersystem get Model
    args << "computersystem" << "get" << "Model";
    result = doCmd("wmic", args);
#elif defined(Q_OS_LINUX)
    QStringList args;
    //还有个命令需要root权限运行 dmidecode -s system-product-name 执行结果和win一样
    result = doCmd("lscpu", args);
#endif

    //Model MS-7C00
    //Model VMWare Virtual Platform
    //Model VirtualBox Virtual Platform
    //Model Alibaba Cloud ECS
    if (result.contains("VMware") || result.contains("VirtualBox") || result.contains("Alibaba")) {
        virtualSystem = true;
    }

    return virtualSystem;
}

bool QtHelper::replaceCRLF = true;
QVector<int> QtHelper::msgTypes = QVector<int>() << 0 << 1 << 2 << 3 << 4;
QVector<QString> QtHelper::msgKeys = QVector<QString>() << QString::fromUtf8("发送") << QString::fromUtf8("接收") << QString::fromUtf8("解析") << QString::fromUtf8("错误") << QString::fromUtf8("提示");
QVector<QColor> QtHelper::msgColors = QVector<QColor>() << QColor("#3BA372") << QColor("#EE6668") << QColor("#9861B4") << QColor("#FA8359") << QColor("#22A3A9");
QString QtHelper::appendMsg(QTextEdit *textEdit, int type, const QString &data, int maxCount, int &currentCount, bool clear, bool pause)
{
    if (clear) {
        textEdit->clear();
        currentCount = 0;
        return QString();
    }

    if (pause) {
        return QString();
    }

    if (currentCount >= maxCount) {
        textEdit->clear();
        currentCount = 0;
    }

    //不同类型不同颜色显示
    QString strType;
    int index = msgTypes.indexOf(type);
    if (index >= 0) {
        strType = msgKeys.at(index);
        textEdit->setTextColor(msgColors.at(index));
    }

    //过滤回车换行符
    QString strData = data;
    if (replaceCRLF) {
        strData.replace("\r", "");
        strData.replace("\n", "");
    }

    strData = QString("时间[%1] %2: %3").arg(TIMEMS).arg(strType).arg(strData);
    textEdit->append(strData);
    currentCount++;
    return strData;
}

void QtHelper::setFramelessForm(QWidget *widgetMain, bool tool, bool top, bool menu)
{
    widgetMain->setProperty("form", true);
    widgetMain->setProperty("canMove", true);

    //根据设定逐个追加属性
#ifdef __arm__
    widgetMain->setWindowFlags(Qt::FramelessWindowHint | Qt::X11BypassWindowManagerHint);
#else
    widgetMain->setWindowFlags(Qt::FramelessWindowHint);
#endif
    if (tool) {
        widgetMain->setWindowFlags(widgetMain->windowFlags() | Qt::Tool);
    }
    if (top) {
        widgetMain->setWindowFlags(widgetMain->windowFlags() | Qt::WindowStaysOnTopHint);
    }
    if (menu) {
        //如果是其他系统比如neokylin会产生系统边框
#ifdef Q_OS_WIN
        widgetMain->setWindowFlags(widgetMain->windowFlags() | Qt::WindowSystemMenuHint | Qt::WindowMinMaxButtonsHint);
#endif
    }
}

int QtHelper::showMessageBox(const QString &text, int type, int closeSec, bool exec)
{
    int result = 0;
    if (type == 0) {
        showMessageBoxInfo(text, closeSec, exec);
    } else if (type == 1) {
        showMessageBoxError(text, closeSec, exec);
    } else if (type == 2) {
        result = showMessageBoxQuestion(text);
    }

    return result;
}

void QtHelper::showMessageBoxInfo(const QString &text, int closeSec, bool exec)
{
    QMessageBox box(QMessageBox::Information, "提示", text);
    box.setStandardButtons(QMessageBox::Yes);
    box.button(QMessageBox::Yes)->setText("确 定");
    box.exec();
    //QMessageBox::information(0, "提示", info, QMessageBox::Yes);
}

void QtHelper::showMessageBoxError(const QString &text, int closeSec, bool exec)
{
    QMessageBox box(QMessageBox::Critical, "错误", text);
    box.setStandardButtons(QMessageBox::Yes);
    box.button(QMessageBox::Yes)->setText("确 定");
    box.exec();
    //QMessageBox::critical(0, "错误", info, QMessageBox::Yes);
}

int QtHelper::showMessageBoxQuestion(const QString &text)
{
    QMessageBox box(QMessageBox::Question, "询问", text);
    box.setStandardButtons(QMessageBox::Yes | QMessageBox::No);
    box.button(QMessageBox::Yes)->setText("确 定");
    box.button(QMessageBox::No)->setText("取 消");
    return box.exec();
    //return QMessageBox::question(0, "询问", info, QMessageBox::Yes | QMessageBox::No);
}

void QtHelper::initDialog(QFileDialog *dialog, const QString &title, const QString &acceptName,
                          const QString &dirName, bool native, int width, int height)
{
    //设置标题
    dialog->setWindowTitle(title);
    //设置标签文本
    dialog->setLabelText(QFileDialog::Accept, acceptName);
    dialog->setLabelText(QFileDialog::Reject, "取消(&C)");
    dialog->setLabelText(QFileDialog::LookIn, "查看");
    dialog->setLabelText(QFileDialog::FileName, "名称");
    dialog->setLabelText(QFileDialog::FileType, "类型");

    //设置默认显示目录
    if (!dirName.isEmpty()) {
        dialog->setDirectory(dirName);
    }

    //设置对话框宽高
    if (width > 0 && height > 0) {
#ifdef Q_OS_ANDROID
        bool horizontal = (QtHelper::deskWidth() > QtHelper::deskHeight());
        if (horizontal) {
            width = QtHelper::deskWidth() / 2;
            height = QtHelper::deskHeight() - 50;
        } else {
            width = QtHelper::deskWidth() - 10;
            height = QtHelper::deskHeight() / 2;
        }
#endif
        dialog->setFixedSize(width, height);
    }

    //设置是否采用本地对话框
    dialog->setOption(QFileDialog::DontUseNativeDialog, !native);
    //设置只读可以取消右上角的新建按钮
    //dialog->setReadOnly(true);
}

QString QtHelper::getDialogResult(QFileDialog *dialog)
{
    QString result;
    if (dialog->exec() == QFileDialog::Accepted) {
        result = dialog->selectedFiles().first();
        if (!result.contains(".")) {
            //自动补全拓展名 保存文件(*.txt *.exe)
            QString filter = dialog->selectedNameFilter();
            if (filter.contains("*.")) {
                filter = filter.split("(").last();
                filter = filter.mid(0, filter.length() - 1);
                //取出第一个作为拓展名
                if (!filter.contains("*.*")) {
                    filter = filter.split(" ").first();
                    result = result + filter.mid(1, filter.length());
                }
            }
        }
    }
    return result;
}

QString QtHelper::getOpenFileName(const QString &filter, const QString &dirName, const QString &fileName,
                                  bool native, int width, int height)
{
    QFileDialog dialog;
    initDialog(&dialog, "打开文件", "选择(&S)", dirName, native, width, height);

    //设置文件类型
    if (!filter.isEmpty()) {
        dialog.setNameFilter(filter);
    }

    //设置默认文件名称
    dialog.selectFile(fileName);
    return getDialogResult(&dialog);
}

QString QtHelper::getSaveFileName(const QString &filter, const QString &dirName, const QString &fileName,
                                  bool native, int width, int height)
{
    QFileDialog dialog;
    initDialog(&dialog, "保存文件", "保存(&S)", dirName, native, width, height);

    //设置文件类型
    if (!filter.isEmpty()) {
        dialog.setNameFilter(filter);
    }

    //设置默认文件名称
    dialog.selectFile(fileName);
    //设置模态类型允许输入
    dialog.setWindowModality(Qt::WindowModal);
    //设置置顶显示
    dialog.setWindowFlags(dialog.windowFlags() | Qt::WindowStaysOnTopHint);
    return getDialogResult(&dialog);
}

QString QtHelper::getExistingDirectory(const QString &dirName, bool native, int width, int height)
{
    QFileDialog dialog;
    initDialog(&dialog, "选择目录", "选择(&S)", dirName, native, width, height);
    dialog.setOption(QFileDialog::ReadOnly);
    //设置只显示目录
#if (QT_VERSION < QT_VERSION_CHECK(6,0,0))
    dialog.setFileMode(QFileDialog::DirectoryOnly);
#else
    dialog.setFileMode(QFileDialog::Directory);
#endif
    dialog.setOption(QFileDialog::ShowDirsOnly);
    return getDialogResult(&dialog);
}

QString QtHelper::getXorEncryptDecrypt(const QString &value, char key)
{
    //矫正范围外的数据
    if (key < 0 || key >= 127) {
        key = 127;
    }

    //大概从5.9版本输出的加密密码字符串前面会加上 @String 字符
    QString result = value;
    if (result.startsWith("@String")) {
        result = result.mid(8, result.length() - 9);
    }

    for (int i = 0; i < result.length(); ++i) {
        result[i] = QChar(result.at(i).toLatin1() ^ key);
    }
    return result;
}

quint8 QtHelper::getOrCode(const QByteArray &data)
{
    int len = data.length();
    quint8 result = 0;
    for (int i = 0; i < len; ++i) {
        result ^= data.at(i);
    }

    return result;
}

quint8 QtHelper::getCheckCode(const QByteArray &data)
{
    int len = data.length();
    quint8 temp = 0;
    for (int i = 0; i < len; ++i) {
        temp += data.at(i);
    }

    return temp % 256;
}

void QtHelper::initTableView(QTableView *tableView, int rowHeight, bool headVisible, bool edit, bool stretchLast)
{
    //设置弱属性用于应用qss特殊样式
    tableView->setProperty("model", true);
    //取消自动换行
    tableView->setWordWrap(false);
    //超出文本不显示省略号
    tableView->setTextElideMode(Qt::ElideNone);
    //奇数偶数行颜色交替
    tableView->setAlternatingRowColors(false);
    //垂直表头是否可见
    tableView->verticalHeader()->setVisible(headVisible);
    //选中一行表头是否加粗
    tableView->horizontalHeader()->setHighlightSections(false);
    //最后一行拉伸填充
    tableView->horizontalHeader()->setStretchLastSection(stretchLast);
    //行标题最小宽度尺寸
    tableView->horizontalHeader()->setMinimumSectionSize(0);
    //行标题最小高度,等同于和默认行高一致
    tableView->horizontalHeader()->setFixedHeight(rowHeight);
    //默认行高
    tableView->verticalHeader()->setDefaultSectionSize(rowHeight);
    //选中时一行整体选中
    tableView->setSelectionBehavior(QAbstractItemView::SelectRows);
    //只允许选择单个
    tableView->setSelectionMode(QAbstractItemView::SingleSelection);

    //表头不可单击
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    tableView->horizontalHeader()->setSectionsClickable(false);
#else
    tableView->horizontalHeader()->setClickable(false);
#endif

    //鼠标按下即进入编辑模式
    if (edit) {
        tableView->setEditTriggers(QAbstractItemView::CurrentChanged | QAbstractItemView::DoubleClicked);
    } else {
        tableView->setEditTriggers(QAbstractItemView::NoEditTriggers);
    }
}

void QtHelper::openFile(const QString &fileName, const QString &msg)
{
#ifdef __arm__
    return;
#endif
    //文件不存在则不用处理
    if (!QFile(fileName).exists()) {
        return;
    }
    if (QtHelper::showMessageBoxQuestion(msg + "成功, 确定现在就打开吗?") == QMessageBox::Yes) {
        QString url = QString("file:///%1").arg(fileName);
        QDesktopServices::openUrl(QUrl(url, QUrl::TolerantMode));
    }
}

bool QtHelper::checkIniFile(const QString &iniFile)
{
    //如果配置文件大小为0,则以初始值继续运行,并生成配置文件
    QFile file(iniFile);
    if (file.size() == 0) {
        return false;
    }

    //如果配置文件不完整,则以初始值继续运行,并生成配置文件
    if (file.open(QFile::ReadOnly)) {
        bool ok = true;
        while (!file.atEnd()) {
            QString line = file.readLine();
            line.replace("\r", "");
            line.replace("\n", "");
            QStringList list = line.split("=");

            if (list.count() == 2) {
                QString key = list.at(0);
                QString value = list.at(1);
                if (value.isEmpty()) {
                    qDebug() << TIMEMS << "ini node no value" << key;
                    ok = false;
                    break;
                }
            }
        }

        if (!ok) {
            return false;
        }
    } else {
        return false;
    }

    return true;
}

QString QtHelper::cutString(const QString &text, int len, int left, int right, bool file, const QString &mid)
{
    //如果指定了字符串分割则表示是文件名需要去掉拓展名
    QString result = text;
    if (file && result.contains(".")) {
        int index = result.lastIndexOf(".");
        result = result.mid(0, index);
    }

    //最终字符串格式为 前缀字符...后缀字符
    if (result.length() > len) {
        result = QString("%1%2%3").arg(result.left(left)).arg(mid).arg(result.right(right));
    }

    return result;
}

QRect QtHelper::getCenterRect(const QSize &imageSize, const QRect &widgetRect, int borderWidth, int scaleMode)
{
    QSize newSize = imageSize;
    QSize widgetSize = widgetRect.size() - QSize(borderWidth * 1, borderWidth * 1);

    if (scaleMode == 0) {
        if (newSize.width() > widgetSize.width() || newSize.height() > widgetSize.height()) {
            newSize.scale(widgetSize, Qt::KeepAspectRatio);
        }
    } else if (scaleMode == 1) {
        newSize.scale(widgetSize, Qt::KeepAspectRatio);
    } else {
        newSize = widgetSize;
    }

    int x = widgetRect.center().x() - newSize.width() / 2;
    int y = widgetRect.center().y() - newSize.height() / 2;
    //不是2的倍数需要偏移1像素
    x += (x % 2 == 0 ? 1 : 0);
    y += (y % 2 == 0 ? 1 : 0);
    return QRect(x, y, newSize.width(), newSize.height());
}

void QtHelper::getScaledImage(QImage &image, const QSize &widgetSize, int scaleMode, bool fast)
{
    Qt::TransformationMode mode = fast ? Qt::FastTransformation : Qt::SmoothTransformation;
    if (scaleMode == 0) {
        if (image.width() > widgetSize.width() || image.height() > widgetSize.height()) {
            image = image.scaled(widgetSize, Qt::KeepAspectRatio, mode);
        }
    } else if (scaleMode == 1) {
        image = image.scaled(widgetSize, Qt::KeepAspectRatio, mode);
    } else {
        image = image.scaled(widgetSize, Qt::IgnoreAspectRatio, mode);
    }
}

QString QtHelper::getTimeString(qint64 time)
{
    time = time / 1000;
    QString min = QString("%1").arg(time / 60, 2, 10, QChar('0'));
    QString sec = QString("%2").arg(time % 60, 2, 10, QChar('0'));
    return QString("%1:%2").arg(min).arg(sec);
}

QString QtHelper::getTimeString(QElapsedTimer timer)
{
    return QString::number((float)timer.elapsed() / 1000, 'f', 3);
}

QString QtHelper::getSizeString(quint64 size)
{
    float num = size;
    QStringList list;
    list << "KB" << "MB" << "GB" << "TB";

    QString unit("bytes");
    QStringListIterator i(list);
    while (num >= 1024.0 && i.hasNext()) {
        unit = i.next();
        num /= 1024.0;
    }

    return QString("%1 %2").arg(QString::number(num, 'f', 2)).arg(unit);
}

//setSystemDateTime("2022", "07", "01", "12", "22", "55");
void QtHelper::setSystemDateTime(const QString &year, const QString &month, const QString &day, const QString &hour, const QString &min, const QString &sec)
{
#ifdef Q_OS_WIN
    QProcess p;
    //先设置日期
    p.start("cmd", QStringList());
    p.waitForStarted();
    p.write(QString("date %1-%2-%3\n").arg(year).arg(month).arg(day).toLatin1());
    p.closeWriteChannel();
    p.waitForFinished(1000);
    p.close();
    //再设置时间
    p.start("cmd", QStringList());
    p.waitForStarted();
    p.write(QString("time %1:%2:%3.00\n").arg(hour).arg(min).arg(sec).toLatin1());
    p.closeWriteChannel();
    p.waitForFinished(1000);
    p.close();
#else
    QString cmd = QString("date %1%2%3%4%5.%6").arg(month).arg(day).arg(hour).arg(min).arg(year).arg(sec);
    //设置日期时间
    system(cmd.toLatin1());
    //硬件时钟同步
    system("hwclock -w");
#endif
}

void QtHelper::runWithSystem(bool autoRun)
{
    QtHelper::runWithSystem(qApp->applicationName(), qApp->applicationFilePath(), autoRun);
}

void QtHelper::runWithSystem(const QString &fileName, const QString &filePath, bool autoRun)
{
#ifdef Q_OS_WIN
    //要转换成本地文件路径(不启动则文件路径为空即可)
    QSettings reg("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run", QSettings::NativeFormat);
    reg.setValue(fileName, autoRun ? QDir::toNativeSeparators(filePath) : "");
#endif
}

void QtHelper::start(const QString &path, const QString &name, bool bin)
{
#ifdef Q_OS_WIN
    QString cmd1 = "tasklist";
    QString cmd2 = QString("%1/%2%3").arg(path).arg(name).arg(bin ? ".exe" : "");
#else
    QString cmd1 = "ps -aux";
    QString cmd2 = QString("%1/%2").arg(path).arg(name);
#endif

#ifndef Q_OS_WASM
    QProcess p;
    p.start(cmd1, QStringList());
    if (p.waitForFinished()) {
        QString result = p.readAll();
        if (result.contains(name)) {
            return;
        }
    }

    //加上引号可以兼容打开带空格的目录(Program Files)
    if (cmd2.contains(" ")) {
        cmd2 = "\"" + cmd2 + "\"";
    }

    //切换到当前目录
    QDir::setCurrent(path);
    //QProcess::execute(cmd2, QStringList());
    QProcess::startDetached(cmd2, QStringList());
    //执行完成后切换回默认目录
    QDir::setCurrent(QtHelper::appPath());
#endif
}
```

### `api/qthelper.h`

```cpp
﻿#ifndef QTHELPER_H
#define QTHELPER_H

#include "head.h"

class QtHelper
{
public:
    //获取所有屏幕区域/当前鼠标所在屏幕索引/区域尺寸/缩放系数
    static bool useRatio;
    static QList<QRect> getScreenRects(bool available = true, bool full = false);
    static int getScreenIndex();
    static QRect getScreenRect(bool available = true, bool full = false);
    static qreal getScreenRatio(int index = -1, bool devicePixel = false);

    //矫正当前鼠标所在屏幕居中尺寸
    static QRect checkCenterRect(QRect &rect, bool available = true);

    //获取桌面宽度高度+居中显示
    static int deskWidth();
    static int deskHeight();
    static QSize deskSize();

    //居中显示窗体
    //定义标志位指定是以桌面为参照还是主程序界面为参照
    static QWidget *centerBaseForm;
    static void setFormInCenter(QWidget *form);
    static void showForm(QWidget *form);

    //程序文件名称和当前所在路径
    static QString appName();
    static QString appPath();

    //程序最前面获取应用程序路径和名称
    static void getCurrentInfo(char *argv[], QString &path, QString &name);
    //程序最前面读取配置文件节点的值
    static QString getIniValue(const QString &fileName, const QString &key);
    static QString getIniValue(char *argv[], const QString &key, const QString &dir = QString(), const QString &file = QString());

    //获取本地网卡IP集合
    static QStringList getLocalIPs();
    //添加网卡集合并根据默认值设置当前项
    static void initLocalIPs(QComboBox *cbox, const QString &defaultIP, bool local127 = true);

    //获取内置颜色集合
    static QList<QColor> colors;
    static QList<QColor> getColorList();
    static QStringList getColorNames();
    //随机获取颜色集合中的颜色
    static QColor getRandColor();

    //初始化随机数种子
    static void initRand();
    //获取随机小数
    static float getRandFloat(float min, float max);
    //获取随机数,指定最小值和最大值
    static double getRandValue(int min, int max, bool contansMin = false, bool contansMax = false);
    //获取范围值随机经纬度集合
    static QStringList getRandPoint(int count, float mainLng, float mainLat, float dotLng, float dotLat);
    //根据旧的范围值和值计算新的范围值对应的值
    static int getRangeValue(int oldMin, int oldMax, int oldValue, int newMin, int newMax);

    //获取uuid
    static QString getUuid();
    //校验目录
    static QString checkPath(const QString &dirName);
    //转换成完整路径
    static QString checkFile(const QString &fileName);
    //通用延时函数(支持Qt4 Qt5 Qt6)
    static void sleep(int msec, bool exec = true);
    //检查程序是否已经运行
    static void checkRun();

    //设置Qt自带样式
    static void setStyle();
    //设置字体
    static QFont addFont(const QString &fontFile, const QString &fontName);
    static void setFont(int fontSize = 12);
    //设置编码
    static void setCode(bool utf8 = true);
    //设置翻译文件
    static void setTranslator(const QString &qmFile);

    //动态设置权限
    static bool checkPermission(const QString &permission);
    //申请安卓权限
    static void initAndroidPermission();

    //一次性设置所有包括编码样式字体等
    static void initAll(bool utf8 = true, bool style = true, bool tabCenter = true, int fontSize = 13);
    //初始化main函数最前面执行的一段代码
    static void initMain(bool desktopSettingsAware = false, bool use96Dpi = false, bool logCritical = true);
    //初始化opengl类型(1=AA_UseDesktopOpenGL 2=AA_UseOpenGLES 3=AA_UseSoftwareOpenGL)
    static void initOpenGL(quint8 type = 0, bool checkCardEnable = false, bool checkVirtualSystem = false);

    //读取qss文件获取样式表内容
    static QString getStyle(const QString &qssFile);
    //设置qss文件到全局样式
    static void setStyle(const QString &qssFile);

    //执行命令行返回执行结果
    static QString doCmd(const QString &program, const QStringList &arguments, int timeout = 1000);
    //获取显卡是否被禁用
    static bool isVideoCardEnable();
    //获取是否在虚拟机环境
    static bool isVirtualSystem();

    //插入消息
    static bool replaceCRLF;
    static QVector<int> msgTypes;
    static QVector<QString> msgKeys;
    static QVector<QColor> msgColors;
    static QString appendMsg(QTextEdit *textEdit, int type, const QString &data,
                             int maxCount, int &currentCount,
                             bool clear = false, bool pause = false);

    //设置无边框
    static void setFramelessForm(QWidget *widgetMain, bool tool = false, bool top = false, bool menu = true);

    //弹出框
    static int showMessageBox(const QString &text, int type = 0, int closeSec = 0, bool exec = false);
    //弹出消息框
    static void showMessageBoxInfo(const QString &text, int closeSec = 0, bool exec = false);
    //弹出错误框
    static void showMessageBoxError(const QString &text, int closeSec = 0, bool exec = false);
    //弹出询问框
    static int showMessageBoxQuestion(const QString &text);

    //为什么还要自定义对话框因为可控宽高和汉化对应文本等
    //初始化对话框文本
    static void initDialog(QFileDialog *dialog, const QString &title, const QString &acceptName,
                           const QString &dirName, bool native, int width, int height);
    //拿到对话框结果
    static QString getDialogResult(QFileDialog *dialog);
    //选择文件对话框
    static QString getOpenFileName(const QString &filter = QString(),
                                   const QString &dirName = QString(),
                                   const QString &fileName = QString(),
                                   bool native = false, int width = 900, int height = 600);
    //保存文件对话框
    static QString getSaveFileName(const QString &filter = QString(),
                                   const QString &dirName = QString(),
                                   const QString &fileName = QString(),
                                   bool native = false, int width = 900, int height = 600);
    //选择目录对话框
    static QString getExistingDirectory(const QString &dirName = QString(),
                                        bool native = false, int width = 900, int height = 600);

    //异或加密-只支持字符,如果是中文需要将其转换base64编码
    static QString getXorEncryptDecrypt(const QString &value, char key);
    //异或校验
    static quint8 getOrCode(const QByteArray &data);
    //计算校验码
    static quint8 getCheckCode(const QByteArray &data);

    //初始化表格
    static void initTableView(QTableView *tableView, int rowHeight = 25,
                              bool headVisible = false, bool edit = false,
                              bool stretchLast = true);
    //打开文件带提示框
    static void openFile(const QString &fileName, const QString &msg);

    //检查ini配置文件
    static bool checkIniFile(const QString &iniFile);

    //首尾截断字符串显示
    static QString cutString(const QString &text, int len, int left, int right, bool file, const QString &mid = "...");

    //传入图片尺寸和窗体区域及边框大小返回居中区域(scaleMode: 0-自动调整 1-等比缩放 2-拉伸填充)
    static QRect getCenterRect(const QSize &imageSize, const QRect &widgetRect, int borderWidth = 2, int scaleMode = 0);
    //传入图片尺寸和窗体尺寸及缩放策略返回合适尺寸(scaleMode: 0-自动调整 1-等比缩放 2-拉伸填充)
    static void getScaledImage(QImage &image, const QSize &widgetSize, int scaleMode = 0, bool fast = true);

    //毫秒数转时间 00:00
    static QString getTimeString(qint64 time);
    //用时时间转秒数
    static QString getTimeString(QElapsedTimer timer);
    //文件大小转 KB MB GB TB
    static QString getSizeString(quint64 size);

    //设置系统时间
    static void setSystemDateTime(const QString &year, const QString &month, const QString &day,
                                  const QString &hour, const QString &min, const QString &sec);
    //设置开机自启动
    static void runWithSystem(bool autoRun = true);
    static void runWithSystem(const QString &fileName, const QString &filePath, bool autoRun = true);

    //启动运行程序(已经在运行则不启动)
    static void start(const QString &path, const QString &name, bool bin = true);
};

#endif // QTHELPER_H
```

### `api/qthelperdata.cpp`

```cpp
﻿#include "qthelperdata.h"
#include "qthelper.h"

int QtHelperData::strHexToDecimal(const QString &strHex)
{
    bool ok;
    return strHex.toInt(&ok, 16);
}

int QtHelperData::strDecimalToDecimal(const QString &strDecimal)
{
    bool ok;
    return strDecimal.toInt(&ok, 10);
}

int QtHelperData::strBinToDecimal(const QString &strBin)
{
    bool ok;
    return strBin.toInt(&ok, 2);
}

QString QtHelperData::strHexToStrBin(const QString &strHex)
{
    quint8 decimal = strHexToDecimal(strHex);
    QString bin = QString::number(decimal, 2);
    quint8 len = bin.length();

    if (len < 8) {
        for (int i = 0; i < 8 - len; ++i) {
            bin = "0" + bin;
        }
    }

    return bin;
}

QString QtHelperData::decimalToStrBin1(int decimal)
{
    QString bin = QString::number(decimal, 2);
    quint8 len = bin.length();
    if (len <= 8) {
        for (int i = 0; i < 8 - len; ++i) {
            bin = "0" + bin;
        }
    }

    return bin;
}

QString QtHelperData::decimalToStrBin2(int decimal)
{
    QString bin = QString::number(decimal, 2);
    quint8 len = bin.length();
    if (len <= 16) {
        for (int i = 0; i < 16 - len; ++i) {
            bin = "0" + bin;
        }
    }

    return bin;
}

QString QtHelperData::decimalToStrHex(int decimal)
{
    QString temp = QString::number(decimal, 16);
    if (temp.length() == 1) {
        temp = "0" + temp;
    }

    return temp;
}

QByteArray QtHelperData::intToByte(int data, bool reverse)
{
    quint8 data1 = (quint8)(0x000000ff & data);
    quint8 data2 = (quint8)((0x0000ff00 & data) >> 8);
    quint8 data3 = (quint8)((0x00ff0000 & data) >> 16);
    quint8 data4 = (quint8)((0xff000000 & data) >> 24);

    QByteArray result;
    result.resize(4);
    if (reverse) {
        result[0] = data1;
        result[1] = data2;
        result[2] = data3;
        result[3] = data4;
    } else {
        result[0] = data4;
        result[1] = data3;
        result[2] = data2;
        result[3] = data1;
    }
    return result;
}

int QtHelperData::byteToInt(const QByteArray &data, bool reverse)
{
    int result = 0;
    if (reverse) {
        result = data.at(0) & 0x000000ff;
        result |= ((data.at(1) << 8) & 0x0000ff00);
        result |= ((data.at(2) << 16) & 0x00ff0000);
        result |= ((data.at(3) << 24) & 0xff000000);
    } else {
        result = data.at(3) & 0x000000ff;
        result |= ((data.at(2) << 8) & 0x0000ff00);
        result |= ((data.at(1) << 16) & 0x00ff0000);
        result |= ((data.at(0) << 24) & 0xff000000);
    }
    return result;
}

QByteArray QtHelperData::ushortToByte(int data, bool reverse)
{
    quint8 data1 = (quint8)(0x000000ff & data);
    quint8 data2 = (quint8)((0x0000ff00 & data) >> 8);

    QByteArray result;
    result.resize(2);
    if (reverse) {
        result[0] = data1;
        result[1] = data2;
    } else {
        result[0] = data2;
        result[1] = data1;
    }
    return result;
}

int QtHelperData::byteToShort(const QByteArray &data, bool reverse)
{
    int result = 0;
    if (reverse) {
        result = data.at(0) & 0x000000ff;
        result |= ((data.at(1) << 8) & 0x0000ff00);
    } else {
        result = data.at(1) & 0x000000ff;
        result |= ((data.at(0) << 8) & 0x0000ff00);
    }
    if (result >= 32768) {
        result = result - 65536;
    }
    return result;
}

QString QtHelperData::getValue(quint8 value)
{
    QString result = QString::number(value);
    if (result.length() <= 1) {
        result = QString("0%1").arg(result);
    }
    return result;
}

QString QtHelperData::trimmed(const QString &text, int type)
{
    QString temp = text;
    QString pattern;
    if (type == -1) {
        pattern = "^ +\\s*";
    } else if (type == 0) {
        pattern = "\\s";
        //temp.replace(" ", "");
    } else if (type == 1) {
        pattern = "\\s* +$";
    } else if (type == 2) {
        temp = temp.trimmed();
    } else if (type == 3) {
        temp = temp.simplified();
    }

    //调用正则表达式移除空格
    if (!pattern.isEmpty()) {
#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
        temp.remove(QRegularExpression(pattern));
#else
        temp.remove(QRegExp(pattern));
#endif
    }

    return temp;
}

QString QtHelperData::getXorEncryptDecrypt(const QString &value, char key)
{
    //矫正范围外的数据
    if (key < 0 || key >= 127) {
        key = 127;
    }

    //大概从5.9版本输出的加密密码字符串前面会加上 @String 字符
    QString result = value;
    if (result.startsWith("@String")) {
        result = result.mid(8, result.length() - 9);
    }

    for (int i = 0; i < result.length(); ++i) {
        result[i] = QChar(result.at(i).toLatin1() ^ key);
    }

    return result;
}

quint8 QtHelperData::getOrCode(const QByteArray &data)
{
    quint8 result = 0;
    int len = data.length();
    for (int i = 0; i < len; ++i) {
        result ^= (quint8)data.at(i);
    }

    return result;
}

quint8 QtHelperData::getCheckCode(const QByteArray &data)
{
    quint8 result = 0;
    int len = data.length();
    for (int i = 0; i < len; ++i) {
        result += (quint8)data.at(i);
    }

    return result % 256;
}

void QtHelperData::getFullData(QByteArray &buffer)
{
    //计算校验码
    quint8 checkCode = getCheckCode(buffer);
    //尾部插入校验码
    buffer.append(checkCode);
    //头部插入固定帧头
    buffer.insert(0, 0x16);
}

//函数功能：计算CRC16
//参数1：*data 16位CRC校验数据，
//参数2：len   数据流长度
//参数3：init  初始化值
//参数4：table 16位CRC查找表

//正序CRC计算
quint16 QtHelperData::getCrc16(quint8 *data, int len, quint16 init, const quint16 *table)
{
    quint16 crc_16 = init;
    quint8 temp;
    while (len-- > 0) {
        temp = crc_16 & 0xff;
        crc_16 = (crc_16 >> 8) ^ table[(temp ^ *data++) & 0xff];
    }

    return crc_16;
}

//逆序CRC计算
quint16 QtHelperData::getCrc16Rec(quint8 *data, int len, quint16 init, const quint16 *table)
{
    quint16 crc_16 = init;
    quint8 temp;
    while (len-- > 0) {
        temp = crc_16 >> 8;
        crc_16 = (crc_16 << 8) ^ table[(temp ^ *data++) & 0xff];
    }

    return crc_16;
}

//Modbus CRC16校验
quint16 QtHelperData::getModbus16(quint8 *data, int len)
{
    //MODBUS CRC-16表 8005 逆序
    const quint16 table_16[256] = {
        0x0000, 0xC0C1, 0xC181, 0x0140, 0xC301, 0x03C0, 0x0280, 0xC241,
        0xC601, 0x06C0, 0x0780, 0xC741, 0x0500, 0xC5C1, 0xC481, 0x0440,
        0xCC01, 0x0CC0, 0x0D80, 0xCD41, 0x0F00, 0xCFC1, 0xCE81, 0x0E40,
        0x0A00, 0xCAC1, 0xCB81, 0x0B40, 0xC901, 0x09C0, 0x0880, 0xC841,
        0xD801, 0x18C0, 0x1980, 0xD941, 0x1B00, 0xDBC1, 0xDA81, 0x1A40,
        0x1E00, 0xDEC1, 0xDF81, 0x1F40, 0xDD01, 0x1DC0, 0x1C80, 0xDC41,
        0x1400, 0xD4C1, 0xD581, 0x1540, 0xD701, 0x17C0, 0x1680, 0xD641,
        0xD201, 0x12C0, 0x1380, 0xD341, 0x1100, 0xD1C1, 0xD081, 0x1040,
        0xF001, 0x30C0, 0x3180, 0xF141, 0x3300, 0xF3C1, 0xF281, 0x3240,
        0x3600, 0xF6C1, 0xF781, 0x3740, 0xF501, 0x35C0, 0x3480, 0xF441,
        0x3C00, 0xFCC1, 0xFD81, 0x3D40, 0xFF01, 0x3FC0, 0x3E80, 0xFE41,
        0xFA01, 0x3AC0, 0x3B80, 0xFB41, 0x3900, 0xF9C1, 0xF881, 0x3840,
        0x2800, 0xE8C1, 0xE981, 0x2940, 0xEB01, 0x2BC0, 0x2A80, 0xEA41,
        0xEE01, 0x2EC0, 0x2F80, 0xEF41, 0x2D00, 0xEDC1, 0xEC81, 0x2C40,
        0xE401, 0x24C0, 0x2580, 0xE541, 0x2700, 0xE7C1, 0xE681, 0x2640,
        0x2200, 0xE2C1, 0xE381, 0x2340, 0xE101, 0x21C0, 0x2080, 0xE041,
        0xA001, 0x60C0, 0x6180, 0xA141, 0x6300, 0xA3C1, 0xA281, 0x6240,
        0x6600, 0xA6C1, 0xA781, 0x6740, 0xA501, 0x65C0, 0x6480, 0xA441,
        0x6C00, 0xACC1, 0xAD81, 0x6D40, 0xAF01, 0x6FC0, 0x6E80, 0xAE41,
        0xAA01, 0x6AC0, 0x6B80, 0xAB41, 0x6900, 0xA9C1, 0xA881, 0x6840,
        0x7800, 0xB8C1, 0xB981, 0x7940, 0xBB01, 0x7BC0, 0x7A80, 0xBA41,
        0xBE01, 0x7EC0, 0x7F80, 0xBF41, 0x7D00, 0xBDC1, 0xBC81, 0x7C40,
        0xB401, 0x74C0, 0x7580, 0xB541, 0x7700, 0xB7C1, 0xB681, 0x7640,
        0x7200, 0xB2C1, 0xB381, 0x7340, 0xB101, 0x71C0, 0x7080, 0xB041,
        0x5000, 0x90C1, 0x9181, 0x5140, 0x9301, 0x53C0, 0x5280, 0x9241,
        0x9601, 0x56C0, 0x5780, 0x9741, 0x5500, 0x95C1, 0x9481, 0x5440,
        0x9C01, 0x5CC0, 0x5D80, 0x9D41, 0x5F00, 0x9FC1, 0x9E81, 0x5E40,
        0x5A00, 0x9AC1, 0x9B81, 0x5B40, 0x9901, 0x59C0, 0x5880, 0x9841,
        0x8801, 0x48C0, 0x4980, 0x8941, 0x4B00, 0x8BC1, 0x8A81, 0x4A40,
        0x4E00, 0x8EC1, 0x8F81, 0x4F40, 0x8D01, 0x4DC0, 0x4C80, 0x8C41,
        0x4400, 0x84C1, 0x8581, 0x4540, 0x8701, 0x47C0, 0x4680, 0x8641,
        0x8201, 0x42C0, 0x4380, 0x8341, 0x4100, 0x81C1, 0x8081, 0x4040
    };

    return getCrc16(data, len, 0xFFFF, table_16);
}

//CRC16校验
QByteArray QtHelperData::getCrcCode(const QByteArray &data)
{
    quint16 result = getModbus16((quint8 *)data.data(), data.length());
    return QtHelperData::ushortToByte(result, true);
}

static QMap<char, QString> listChar;
void QtHelperData::initAscii()
{
    //0x20为空格,空格以下都是不可见字符
    if (listChar.count() == 0) {
        listChar.insert(0, "\\NUL");
        listChar.insert(1, "\\SOH");
        listChar.insert(2, "\\STX");
        listChar.insert(3, "\\ETX");
        listChar.insert(4, "\\EOT");
        listChar.insert(5, "\\ENQ");
        listChar.insert(6, "\\ACK");
        listChar.insert(7, "\\BEL");
        listChar.insert(8, "\\BS");
        listChar.insert(9, "\\HT");
        listChar.insert(10, "\\LF");
        listChar.insert(11, "\\VT");
        listChar.insert(12, "\\FF");
        listChar.insert(13, "\\CR");
        listChar.insert(14, "\\SO");
        listChar.insert(15, "\\SI");
        listChar.insert(16, "\\DLE");
        listChar.insert(17, "\\DC1");
        listChar.insert(18, "\\DC2");
        listChar.insert(19, "\\DC3");
        listChar.insert(20, "\\DC4");
        listChar.insert(21, "\\NAK");
        listChar.insert(22, "\\SYN");
        listChar.insert(23, "\\ETB");
        listChar.insert(24, "\\CAN");
        listChar.insert(25, "\\EM");
        listChar.insert(26, "\\SUB");
        listChar.insert(27, "\\ESC");
        listChar.insert(28, "\\FS");
        listChar.insert(29, "\\GS");
        listChar.insert(30, "\\RS");
        listChar.insert(31, "\\US");
        listChar.insert(0x5C, "\\");
        listChar.insert(0x7F, "\\DEL");
    }
}

QString QtHelperData::byteArrayToAsciiStr(const QByteArray &data)
{
    //先初始化字符表
    initAscii();

    QString temp;
    int len = data.length();
    for (int i = 0; i < len; ++i) {
        int byte = data.at(i);
        QString value = listChar.value(byte);
        if (!value.isEmpty()) {
        } else if (byte >= 0 && byte <= 0x7F) {
            value = QString("%1").arg(byte);
        } else {
            value = decimalToStrHex(byte);
            value = QString("\\x%1").arg(value.toUpper());
        }

        temp += value;
    }

    return temp.trimmed();
}

QByteArray QtHelperData::asciiStrToByteArray(const QString &data)
{
    //先初始化字符表
    initAscii();

    QByteArray buffer;
    QStringList list = data.split("\\");

    int count = list.count();
    for (int i = 1; i < count; ++i) {
        QString str = list.at(i);
        int key = 0;
        if (str.contains("x")) {
            key = strHexToDecimal(str.mid(1, 2));
        } else {
            key = listChar.key("\\" + str);
        }

        buffer.append(key);
    }

    //可能是纯字符串不带控制字符
    if (buffer.length() == 0) {
        buffer = data.toUtf8();
    }

    return buffer;
}

char QtHelperData::hexStrToChar(char data)
{
    if ((data >= '0') && (data <= '9')) {
        return data - 0x30;
    } else if ((data >= 'A') && (data <= 'F')) {
        return data - 'A' + 10;
    } else if ((data >= 'a') && (data <= 'f')) {
        return data - 'a' + 10;
    } else {
        return (-1);
    }
}

QByteArray QtHelperData::hexStrToByteArray(const QString &data)
{
    QByteArray senddata;
    int hexdata, lowhexdata;
    int hexdatalen = 0;
    int len = data.length();
    senddata.resize(len / 2);
    char lstr, hstr;

    for (int i = 0; i < len;) {
        hstr = data.at(i).toLatin1();
        if (hstr == ' ') {
            i++;
            continue;
        }

        i++;
        if (i >= len) {
            break;
        }

        lstr = data.at(i).toLatin1();
        hexdata = hexStrToChar(hstr);
        lowhexdata = hexStrToChar(lstr);

        if ((hexdata == 16) || (lowhexdata == 16)) {
            break;
        } else {
            hexdata = hexdata * 16 + lowhexdata;
        }

        i++;
        senddata[hexdatalen] = (char)hexdata;
        hexdatalen++;
    }

    senddata.resize(hexdatalen);
    return senddata;
}

QString QtHelperData::byteArrayToHexStr(const QByteArray &data)
{
    QString temp = "";
    QString hex = data.toHex();
    for (int i = 0; i < hex.length(); i = i + 2) {
        temp += hex.mid(i, 2) + " ";
    }

    return temp.trimmed().toUpper();
}
```

### `api/qthelperdata.h`

```cpp
﻿#ifndef QTHELPERDATA_H
#define QTHELPERDATA_H

#include <QObject>

class QtHelperData
{
public:
    //16进制字符串转10进制
    static int strHexToDecimal(const QString &strHex);
    //10进制字符串转10进制
    static int strDecimalToDecimal(const QString &strDecimal);
    //2进制字符串转10进制
    static int strBinToDecimal(const QString &strBin);

    //16进制字符串转2进制字符串
    static QString strHexToStrBin(const QString &strHex);
    //10进制转2进制字符串一个字节
    static QString decimalToStrBin1(int decimal);
    //10进制转2进制字符串两个字节
    static QString decimalToStrBin2(int decimal);
    //10进制转16进制字符串,补零.
    static QString decimalToStrHex(int decimal);

    //int和字节数组互转
    static QByteArray intToByte(int data, bool reverse = false);
    static int byteToInt(const QByteArray &data, bool reverse = false);

    //ushort和字节数组互转
    static QByteArray ushortToByte(int data, bool reverse = false);
    static int byteToShort(const QByteArray &data, bool reverse = false);

    //字符串补全
    static QString getValue(quint8 value);
    //字符串去空格 -1=移除左侧空格 0=移除所有空格 1=移除右侧空格 2=移除首尾空格 3=首尾清除中间留一个空格
    static QString trimmed(const QString &text, int type);

    //异或加密-只支持字符,如果是中文需要将其转换base64编码
    static QString getXorEncryptDecrypt(const QString &value, char key);
    //异或校验
    static quint8 getOrCode(const QByteArray &data);

    //公司专用-计算校验码
    static quint8 getCheckCode(const QByteArray &data);
    //公司专用-加上桢头和校验码完整数据
    static void getFullData(QByteArray &buffer);

    //CRC校验
    static quint16 getCrc16Rec(quint8 *data, int len, quint16 init, const quint16 *table);
    static quint16 getCrc16(quint8 *data, int len, quint16 init, const quint16 *table);
    static quint16 getModbus16(quint8 *data, int len);
    static QByteArray getCrcCode(const QByteArray &data);

    //字节数组与Ascii字符串互转
    static void initAscii();
    static QString byteArrayToAsciiStr(const QByteArray &data);
    static QByteArray asciiStrToByteArray(const QString &data);

    //16进制字符串与字节数组互转
    static char hexStrToChar(char data);
    static QByteArray hexStrToByteArray(const QString &data);
    static QString byteArrayToHexStr(const QByteArray &data);
};

#endif // QTHELPERDATA_H
```

### `form/frmcomtool.cpp`

```cpp
﻿#include "frmcomtool.h"
#include "ui_frmcomtool.h"
#include "qthelper.h"
#include "qthelperdata.h"

frmComTool::frmComTool(QWidget *parent) : QWidget(parent), ui(new Ui::frmComTool)
{
    ui->setupUi(this);
    this->initForm();
    this->initConfig();
    QtHelper::setFormInCenter(this);
}

frmComTool::~frmComTool()
{
    delete ui;
}

void frmComTool::initForm()
{
    comOk = false;
    com = 0;
    sleepTime = 10;
    receiveCount = 0;
    sendCount = 0;
    isShow = true;

    ui->cboxSendInterval->addItems(AppData::Intervals);
    ui->cboxData->addItems(AppData::Datas);

    //读取数据
    timerRead = new QTimer(this);
    timerRead->setInterval(100);
    connect(timerRead, SIGNAL(timeout()), this, SLOT(readData()));

    //发送数据
    timerSend = new QTimer(this);
    connect(timerSend, SIGNAL(timeout()), this, SLOT(sendData()));
    connect(ui->btnSend, SIGNAL(clicked()), this, SLOT(sendData()));

    //保存数据
    timerSave = new QTimer(this);
    connect(timerSave, SIGNAL(timeout()), this, SLOT(saveData()));
    connect(ui->btnSave, SIGNAL(clicked()), this, SLOT(saveData()));

    ui->tabWidget->setCurrentIndex(0);
    changeEnable(false);

    tcpOk = false;
    socket = new QTcpSocket(this);
    socket->abort();
    connect(socket, SIGNAL(readyRead()), this, SLOT(readDataNet()));
#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
    connect(socket, SIGNAL(errorOccurred(QAbstractSocket::SocketError)), this, SLOT(readErrorNet()));
#else
    connect(socket, SIGNAL(error(QAbstractSocket::SocketError)), this, SLOT(readErrorNet()));
#endif

    timerConnect = new QTimer(this);
    connect(timerConnect, SIGNAL(timeout()), this, SLOT(connectNet()));
    timerConnect->setInterval(3000);
    timerConnect->start();

#ifdef __arm__
    ui->widgetRight->setFixedWidth(280);
#endif
}

void frmComTool::initConfig()
{
    QStringList comList;
    for (int i = 1; i <= 20; i++) {
        comList << QString("COM%1").arg(i);
    }

    comList << "ttyUSB0" << "ttyS0" << "ttyS1" << "ttyS2" << "ttyS3" << "ttyS4";
    comList << "ttymxc1" << "ttymxc2" << "ttymxc3" << "ttymxc4";
    comList << "ttySAC1" << "ttySAC2" << "ttySAC3" << "ttySAC4";
    ui->cboxPortName->addItems(comList);
    ui->cboxPortName->setCurrentIndex(ui->cboxPortName->findText(AppConfig::PortName));
    connect(ui->cboxPortName, SIGNAL(currentIndexChanged(int)), this, SLOT(saveConfig()));

    QStringList baudList;
    baudList << "50" << "75" << "100" << "134" << "150" << "200" << "300" << "600" << "1200"
             << "1800" << "2400" << "4800" << "9600" << "14400" << "19200" << "38400"
             << "56000" << "57600" << "76800" << "115200" << "128000" << "256000";

    ui->cboxBaudRate->addItems(baudList);
    ui->cboxBaudRate->setCurrentIndex(ui->cboxBaudRate->findText(QString::number(AppConfig::BaudRate)));
    connect(ui->cboxBaudRate, SIGNAL(currentIndexChanged(int)), this, SLOT(saveConfig()));

    QStringList dataBitsList;
    dataBitsList << "5" << "6" << "7" << "8";

    ui->cboxDataBit->addItems(dataBitsList);
    ui->cboxDataBit->setCurrentIndex(ui->cboxDataBit->findText(QString::number(AppConfig::DataBit)));
    connect(ui->cboxDataBit, SIGNAL(currentIndexChanged(int)), this, SLOT(saveConfig()));

    QStringList parityList;
    parityList << "无" << "奇" << "偶";
#ifdef Q_OS_WIN
    parityList << "标志";
#endif
    parityList << "空格";

    ui->cboxParity->addItems(parityList);
    ui->cboxParity->setCurrentIndex(ui->cboxParity->findText(AppConfig::Parity));
    connect(ui->cboxParity, SIGNAL(currentIndexChanged(int)), this, SLOT(saveConfig()));

    QStringList stopBitsList;
    stopBitsList << "1";
#ifdef Q_OS_WIN
    stopBitsList << "1.5";
#endif
    stopBitsList << "2";

    ui->cboxStopBit->addItems(stopBitsList);
    ui->cboxStopBit->setCurrentIndex(ui->cboxStopBit->findText(QString::number(AppConfig::StopBit)));
    connect(ui->cboxStopBit, SIGNAL(currentIndexChanged(int)), this, SLOT(saveConfig()));

    ui->ckHexSend->setChecked(AppConfig::HexSend);
    connect(ui->ckHexSend, SIGNAL(stateChanged(int)), this, SLOT(saveConfig()));

    ui->ckHexReceive->setChecked(AppConfig::HexReceive);
    connect(ui->ckHexReceive, SIGNAL(stateChanged(int)), this, SLOT(saveConfig()));

    ui->ckDebug->setChecked(AppConfig::Debug);
    connect(ui->ckDebug, SIGNAL(stateChanged(int)), this, SLOT(saveConfig()));

    ui->ckAutoClear->setChecked(AppConfig::AutoClear);
    connect(ui->ckAutoClear, SIGNAL(stateChanged(int)), this, SLOT(saveConfig()));

    ui->ckAutoSend->setChecked(AppConfig::AutoSend);
    connect(ui->ckAutoSend, SIGNAL(stateChanged(int)), this, SLOT(saveConfig()));

    ui->ckAutoSave->setChecked(AppConfig::AutoSave);
    connect(ui->ckAutoSave, SIGNAL(stateChanged(int)), this, SLOT(saveConfig()));

    QStringList sendInterval;
    QStringList saveInterval;
    sendInterval << "100" << "300" << "500";

    for (int i = 1000; i <= 10000; i = i + 1000) {
        sendInterval << QString::number(i);
        saveInterval << QString::number(i);
    }

    ui->cboxSendInterval->addItems(sendInterval);
    ui->cboxSaveInterval->addItems(saveInterval);

    ui->cboxSendInterval->setCurrentIndex(ui->cboxSendInterval->findText(QString::number(AppConfig::SendInterval)));
    connect(ui->cboxSendInterval, SIGNAL(currentIndexChanged(int)), this, SLOT(saveConfig()));
    ui->cboxSaveInterval->setCurrentIndex(ui->cboxSaveInterval->findText(QString::number(AppConfig::SaveInterval)));
    connect(ui->cboxSaveInterval, SIGNAL(currentIndexChanged(int)), this, SLOT(saveConfig()));

    timerSend->setInterval(AppConfig::SendInterval);
    timerSave->setInterval(AppConfig::SaveInterval);

    if (AppConfig::AutoSend) {
        timerSend->start();
    }

    if (AppConfig::AutoSave) {
        timerSave->start();
    }

    //串口转网络部分
    ui->cboxMode->setCurrentIndex(ui->cboxMode->findText(AppConfig::Mode));
    connect(ui->cboxMode, SIGNAL(currentIndexChanged(int)), this, SLOT(saveConfig()));

    ui->txtServerIP->setText(AppConfig::ServerIP);
    connect(ui->txtServerIP, SIGNAL(textChanged(QString)), this, SLOT(saveConfig()));

    ui->txtServerPort->setText(QString::number(AppConfig::ServerPort));
    connect(ui->txtServerPort, SIGNAL(textChanged(QString)), this, SLOT(saveConfig()));

    ui->txtListenPort->setText(QString::number(AppConfig::ListenPort));
    connect(ui->txtListenPort, SIGNAL(textChanged(QString)), this, SLOT(saveConfig()));

    QStringList values;
    values << "0" << "10" << "50";

    for (int i = 100; i < 1000; i = i + 100) {
        values << QString("%1").arg(i);
    }

    ui->cboxSleepTime->addItems(values);

    ui->cboxSleepTime->setCurrentIndex(ui->cboxSleepTime->findText(QString::number(AppConfig::SleepTime)));
    connect(ui->cboxSleepTime, SIGNAL(currentIndexChanged(int)), this, SLOT(saveConfig()));

    ui->ckAutoConnect->setChecked(AppConfig::AutoConnect);
    connect(ui->ckAutoConnect, SIGNAL(stateChanged(int)), this, SLOT(saveConfig()));
}

void frmComTool::saveConfig()
{
    AppConfig::PortName = ui->cboxPortName->currentText();
    AppConfig::BaudRate = ui->cboxBaudRate->currentText().toInt();
    AppConfig::DataBit = ui->cboxDataBit->currentText().toInt();
    AppConfig::Parity = ui->cboxParity->currentText();
    AppConfig::StopBit = ui->cboxStopBit->currentText().toDouble();

    AppConfig::HexSend = ui->ckHexSend->isChecked();
    AppConfig::HexReceive = ui->ckHexReceive->isChecked();
    AppConfig::Debug = ui->ckDebug->isChecked();
    AppConfig::AutoClear = ui->ckAutoClear->isChecked();

    AppConfig::AutoSend = ui->ckAutoSend->isChecked();
    AppConfig::AutoSave = ui->ckAutoSave->isChecked();

    int sendInterval = ui->cboxSendInterval->currentText().toInt();
    if (sendInterval != AppConfig::SendInterval) {
        AppConfig::SendInterval = sendInterval;
        timerSend->setInterval(AppConfig::SendInterval);
    }

    int saveInterval = ui->cboxSaveInterval->currentText().toInt();
    if (saveInterval != AppConfig::SaveInterval) {
        AppConfig::SaveInterval = saveInterval;
        timerSave->setInterval(AppConfig::SaveInterval);
    }

    AppConfig::Mode = ui->cboxMode->currentText();
    AppConfig::ServerIP = ui->txtServerIP->text().trimmed();
    AppConfig::ServerPort = ui->txtServerPort->text().toInt();
    AppConfig::ListenPort = ui->txtListenPort->text().toInt();
    AppConfig::SleepTime = ui->cboxSleepTime->currentText().toInt();
    AppConfig::AutoConnect = ui->ckAutoConnect->isChecked();

    AppConfig::writeConfig();
}

void frmComTool::changeEnable(bool b)
{
    ui->cboxBaudRate->setEnabled(!b);
    ui->cboxDataBit->setEnabled(!b);
    ui->cboxParity->setEnabled(!b);
    ui->cboxPortName->setEnabled(!b);
    ui->cboxStopBit->setEnabled(!b);
    ui->btnSend->setEnabled(b);
    ui->ckAutoSend->setEnabled(b);
    ui->ckAutoSave->setEnabled(b);
}

void frmComTool::append(int type, const QString &data, bool clear)
{
    static int currentCount = 0;
    static int maxCount = 100;

    if (clear) {
        ui->txtMain->clear();
        currentCount = 0;
        return;
    }

    if (currentCount >= maxCount) {
        ui->txtMain->clear();
        currentCount = 0;
    }

    if (!isShow) {
        return;
    }

    //过滤回车换行符
    QString strData = data;
    strData = strData.replace("\r", "");
    strData = strData.replace("\n", "");

    //不同类型不同颜色显示
    QString strType;
    if (type == 0) {
        strType = "串口发送 >>";
        ui->txtMain->setTextColor(QColor("dodgerblue"));
    } else if (type == 1) {
        strType = "串口接收 <<";
        ui->txtMain->setTextColor(QColor("red"));
    } else if (type == 2) {
        strType = "处理延时 >>";
        ui->txtMain->setTextColor(QColor("gray"));
    } else if (type == 3) {
        strType = "正在校验 >>";
        ui->txtMain->setTextColor(QColor("green"));
    } else if (type == 4) {
        strType = "网络发送 >>";
        ui->txtMain->setTextColor(QColor(24, 189, 155));
    } else if (type == 5) {
        strType = "网络接收 <<";
        ui->txtMain->setTextColor(QColor(255, 107, 107));
    } else if (type == 6) {
        strType = "提示信息 >>";
        ui->txtMain->setTextColor(QColor(100, 184, 255));
    }

    strData = QString("时间[%1] %2 %3").arg(TIMEMS).arg(strType).arg(strData);
    ui->txtMain->append(strData);
    currentCount++;
}

void frmComTool::readData()
{
    if (com->bytesAvailable() <= 0) {
        return;
    }

    QtHelper::sleep(sleepTime);
    QByteArray data = com->readAll();
    int dataLen = data.length();
    if (dataLen <= 0) {
        return;
    }

    if (isShow) {
        QString buffer;
        if (ui->ckHexReceive->isChecked()) {
            buffer = QtHelperData::byteArrayToHexStr(data);
        } else {
            //buffer = QtHelperData::byteArrayToAsciiStr(data);
            buffer = QString::fromLocal8Bit(data);
        }

        //启用调试则模拟调试数据
        if (ui->ckDebug->isChecked()) {
            int count = AppData::Keys.count();
            for (int i = 0; i < count; i++) {
                if (buffer.startsWith(AppData::Keys.at(i))) {
                    sendData(AppData::Values.at(i));
                    break;
                }
            }
        }

        append(1, buffer);
        receiveCount = receiveCount + data.size();
        ui->btnReceiveCount->setText(QString("接收 : %1 字节").arg(receiveCount));

        //启用网络转发则调用网络发送数据
        if (tcpOk) {
            socket->write(data);
            append(4, QString(buffer));
        }
    }
}

void frmComTool::sendData()
{
    QString str = ui->cboxData->currentText();
    if (str.isEmpty()) {
        ui->cboxData->setFocus();
        return;
    }

    sendData(str);

    if (ui->ckAutoClear->isChecked()) {
        ui->cboxData->setCurrentIndex(-1);
        ui->cboxData->setFocus();
    }
}

void frmComTool::sendData(QString data)
{
    if (com == 0 || !com->isOpen()) {
        return;
    }

    //短信猫调试
    if (data.startsWith("AT")) {
        data += "\r";
    }

    QByteArray buffer;
    if (ui->ckHexSend->isChecked()) {
        buffer = QtHelperData::hexStrToByteArray(data);
    } else {
        buffer = QtHelperData::asciiStrToByteArray(data);
    }

    com->write(buffer);
    append(0, data);
    sendCount = sendCount + buffer.size();
    ui->btnSendCount->setText(QString("发送 : %1 字节").arg(sendCount));
}

void frmComTool::saveData()
{
    QString tempData = ui->txtMain->toPlainText();
    if (tempData.isEmpty()) {
        return;
    }

    QDateTime now = QDateTime::currentDateTime();
    QString name = now.toString("yyyy-MM-dd-HH-mm-ss");
    QString fileName = QString("%1/%2.txt").arg(QtHelper::appPath()).arg(name);

    QFile file(fileName);
    file.open(QFile::WriteOnly | QIODevice::Text);
    QTextStream out(&file);
    out << tempData;
    file.close();

    on_btnClear_clicked();
}

void frmComTool::on_btnOpen_clicked()
{
    if (ui->btnOpen->text() == "打开串口") {
        com = new QextSerialPort(ui->cboxPortName->currentText(), QextSerialPort::Polling);
        comOk = com->open(QIODevice::ReadWrite);

        if (comOk) {
            //清空缓冲区
            com->flush();
            //设置波特率
            com->setBaudRate((BaudRateType)ui->cboxBaudRate->currentText().toInt());
            //设置数据位
            com->setDataBits((DataBitsType)ui->cboxDataBit->currentText().toInt());
            //设置校验位
            com->setParity((ParityType)ui->cboxParity->currentIndex());
            //设置停止位
            com->setStopBits((StopBitsType)ui->cboxStopBit->currentIndex());
            com->setFlowControl(FLOW_OFF);
            com->setTimeout(10);

            changeEnable(true);
            ui->btnOpen->setText("关闭串口");
            timerRead->start();
        }
    } else {
        timerRead->stop();
        com->close();
        com->deleteLater();

        changeEnable(false);
        ui->btnOpen->setText("打开串口");
        on_btnClear_clicked();
        comOk = false;
    }
}

void frmComTool::on_btnSendCount_clicked()
{
    sendCount = 0;
    ui->btnSendCount->setText("发送 : 0 字节");
}

void frmComTool::on_btnReceiveCount_clicked()
{
    receiveCount = 0;
    ui->btnReceiveCount->setText("接收 : 0 字节");
}

void frmComTool::on_btnStopShow_clicked()
{
    if (ui->btnStopShow->text() == "停止显示") {
        isShow = false;
        ui->btnStopShow->setText("开始显示");
    } else {
        isShow = true;
        ui->btnStopShow->setText("停止显示");
    }
}

void frmComTool::on_btnData_clicked()
{
    QString fileName = QString("%1/%2").arg(QtHelper::appPath()).arg("send.txt");
    QFile file(fileName);
    if (!file.exists()) {
        return;
    }

    if (ui->btnData->text() == "管理数据") {
        ui->txtMain->setReadOnly(false);
        ui->txtMain->clear();
        file.open(QFile::ReadOnly | QIODevice::Text);
        QTextStream in(&file);
        ui->txtMain->setText(in.readAll());
        file.close();
        ui->btnData->setText("保存数据");
    } else {
        ui->txtMain->setReadOnly(true);
        file.open(QFile::WriteOnly | QIODevice::Text);
        QTextStream out(&file);
        out << ui->txtMain->toPlainText();
        file.close();
        ui->txtMain->clear();
        ui->btnData->setText("管理数据");
        AppData::readSendData();
    }
}

void frmComTool::on_btnClear_clicked()
{
    append(0, "", true);
}

void frmComTool::on_btnStart_clicked()
{
    if (ui->btnStart->text() == "启动") {
        if (AppConfig::ServerIP == "" || AppConfig::ServerPort == 0) {
            append(6, "IP地址和远程端口不能为空");
            return;
        }

        socket->connectToHost(AppConfig::ServerIP, AppConfig::ServerPort);
        if (socket->waitForConnected(100)) {
            ui->btnStart->setText("停止");
            append(6, "连接服务器成功");
            tcpOk = true;
        }
    } else {
        socket->disconnectFromHost();
        if (socket->state() == QAbstractSocket::UnconnectedState || socket->waitForDisconnected(100)) {
            ui->btnStart->setText("启动");
            append(6, "断开服务器成功");
            tcpOk = false;
        }
    }
}

void frmComTool::on_ckAutoSend_stateChanged(int arg1)
{
    if (arg1 == 0) {
        ui->cboxSendInterval->setEnabled(false);
        timerSend->stop();
    } else {
        ui->cboxSendInterval->setEnabled(true);
        timerSend->start();
    }
}

void frmComTool::on_ckAutoSave_stateChanged(int arg1)
{
    if (arg1 == 0) {
        ui->cboxSaveInterval->setEnabled(false);
        timerSave->stop();
    } else {
        ui->cboxSaveInterval->setEnabled(true);
        timerSave->start();
    }
}

void frmComTool::connectNet()
{
    if (!tcpOk && AppConfig::AutoConnect && ui->btnStart->text() == "启动") {
        if (AppConfig::ServerIP != "" && AppConfig::ServerPort != 0) {
            socket->connectToHost(AppConfig::ServerIP, AppConfig::ServerPort);
            if (socket->waitForConnected(100)) {
                ui->btnStart->setText("停止");
                append(6, "连接服务器成功");
                tcpOk = true;
            }
        }
    }
}

void frmComTool::readDataNet()
{
    if (socket->bytesAvailable() > 0) {
        QtHelper::sleep(AppConfig::SleepTime);
        QByteArray data = socket->readAll();

        QString buffer;
        if (ui->ckHexReceive->isChecked()) {
            buffer = QtHelperData::byteArrayToHexStr(data);
        } else {
            buffer = QtHelperData::byteArrayToAsciiStr(data);
        }

        append(5, buffer);

        //将收到的网络数据转发给串口
        if (comOk) {
            com->write(data);
            append(0, buffer);
        }
    }
}

void frmComTool::readErrorNet()
{
    ui->btnStart->setText("启动");
    append(6, QString("连接服务器失败,%1").arg(socket->errorString()));
    socket->disconnectFromHost();
    tcpOk = false;
}
```

### `form/frmcomtool.h`

```cpp
﻿#ifndef FRMCOMTOOL_H
#define FRMCOMTOOL_H

#include <QWidget>
#include "qtcpsocket.h"
#include "qextserialport.h"

namespace Ui
{
class frmComTool;
}

class frmComTool : public QWidget
{
    Q_OBJECT

public:
    explicit frmComTool(QWidget *parent = 0);
    ~frmComTool();

private:
    Ui::frmComTool *ui;

    bool comOk;                 //串口是否打开
    QextSerialPort *com;        //串口通信对象
    QTimer *timerRead;          //定时读取串口数据
    QTimer *timerSend;          //定时发送串口数据
    QTimer *timerSave;          //定时保存串口数据

    int sleepTime;              //接收延时时间
    int sendCount;              //发送数据计数
    int receiveCount;           //接收数据计数
    bool isShow;                //是否显示数据

    bool tcpOk;                 //网络是否正常
    QTcpSocket *socket;         //网络连接对象
    QTimer *timerConnect;       //定时器重连

private slots:
    void initForm();            //初始化窗体数据
    void initConfig();          //初始化配置文件
    void saveConfig();          //保存配置文件
    void readData();            //读取串口数据
    void sendData();            //发送串口数据
    void sendData(QString data);//发送串口数据带参数
    void saveData();            //保存串口数据

    void changeEnable(bool b);  //改变状态
    void append(int type, const QString &data, bool clear = false);

private slots:
    void connectNet();
    void readDataNet();
    void readErrorNet();

private slots:
    void on_btnOpen_clicked();
    void on_btnStopShow_clicked();
    void on_btnSendCount_clicked();
    void on_btnReceiveCount_clicked();

    void on_btnClear_clicked();
    void on_btnData_clicked();
    void on_btnStart_clicked();

    void on_ckAutoSend_stateChanged(int arg1);
    void on_ckAutoSave_stateChanged(int arg1);
};

#endif // FRMCOMTOOL_H
```

### `form/frmcomtool.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmComTool</class>
 <widget class="QWidget" name="frmComTool">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <layout class="QGridLayout" name="gridLayout_6">
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
   <item row="0" column="0">
    <widget class="QTextEdit" name="txtMain">
     <property name="enabled">
      <bool>true</bool>
     </property>
     <property name="frameShape">
      <enum>QFrame::StyledPanel</enum>
     </property>
     <property name="verticalScrollBarPolicy">
      <enum>Qt::ScrollBarAsNeeded</enum>
     </property>
     <property name="horizontalScrollBarPolicy">
      <enum>Qt::ScrollBarAsNeeded</enum>
     </property>
     <property name="readOnly">
      <bool>true</bool>
     </property>
    </widget>
   </item>
   <item row="0" column="1" rowspan="2">
    <widget class="QWidget" name="widgetRight" native="true">
     <property name="minimumSize">
      <size>
       <width>180</width>
       <height>0</height>
      </size>
     </property>
     <property name="maximumSize">
      <size>
       <width>180</width>
       <height>16777215</height>
      </size>
     </property>
     <layout class="QVBoxLayout" name="verticalLayout">
      <property name="leftMargin">
       <number>0</number>
      </property>
      <property name="topMargin">
       <number>0</number>
      </property>
      <property name="rightMargin">
       <number>0</number>
      </property>
      <property name="bottomMargin">
       <number>0</number>
      </property>
      <item>
       <widget class="QFrame" name="frameTop">
        <property name="frameShape">
         <enum>QFrame::Box</enum>
        </property>
        <property name="frameShadow">
         <enum>QFrame::Sunken</enum>
        </property>
        <layout class="QGridLayout" name="gridLayout_3">
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
         <item row="0" column="0">
          <widget class="QLabel" name="labPortName">
           <property name="text">
            <string>串口号</string>
           </property>
          </widget>
         </item>
         <item row="0" column="1">
          <widget class="QComboBox" name="cboxPortName">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
             <horstretch>1</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="editable">
            <bool>true</bool>
           </property>
          </widget>
         </item>
         <item row="1" column="0">
          <widget class="QLabel" name="labBaudRate">
           <property name="text">
            <string>波特率</string>
           </property>
          </widget>
         </item>
         <item row="1" column="1">
          <widget class="QComboBox" name="cboxBaudRate">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
             <horstretch>1</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="editable">
            <bool>true</bool>
           </property>
          </widget>
         </item>
         <item row="2" column="0">
          <widget class="QLabel" name="labDataBit">
           <property name="text">
            <string>数据位</string>
           </property>
          </widget>
         </item>
         <item row="2" column="1">
          <widget class="QComboBox" name="cboxDataBit">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
             <horstretch>1</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
          </widget>
         </item>
         <item row="3" column="0">
          <widget class="QLabel" name="labParity">
           <property name="text">
            <string>校验位</string>
           </property>
          </widget>
         </item>
         <item row="3" column="1">
          <widget class="QComboBox" name="cboxParity">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
             <horstretch>1</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
          </widget>
         </item>
         <item row="4" column="0">
          <widget class="QLabel" name="labStopBit">
           <property name="text">
            <string>停止位</string>
           </property>
          </widget>
         </item>
         <item row="4" column="1">
          <widget class="QComboBox" name="cboxStopBit">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
             <horstretch>1</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
          </widget>
         </item>
         <item row="5" column="0" colspan="2">
          <widget class="QPushButton" name="btnOpen">
           <property name="text">
            <string>打开串口</string>
           </property>
          </widget>
         </item>
        </layout>
       </widget>
      </item>
      <item>
       <widget class="QTabWidget" name="tabWidget">
        <property name="tabPosition">
         <enum>QTabWidget::South</enum>
        </property>
        <property name="currentIndex">
         <number>0</number>
        </property>
        <widget class="QWidget" name="tab">
         <attribute name="title">
          <string>串口配置</string>
         </attribute>
         <layout class="QGridLayout" name="gridLayout_5">
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
          <item row="0" column="0">
           <widget class="QCheckBox" name="ckHexSend">
            <property name="text">
             <string>Hex发送</string>
            </property>
           </widget>
          </item>
          <item row="0" column="1">
           <widget class="QCheckBox" name="ckHexReceive">
            <property name="text">
             <string>Hex接收</string>
            </property>
           </widget>
          </item>
          <item row="1" column="0">
           <widget class="QCheckBox" name="ckDebug">
            <property name="text">
             <string>模拟设备</string>
            </property>
           </widget>
          </item>
          <item row="1" column="1">
           <widget class="QCheckBox" name="ckAutoClear">
            <property name="text">
             <string>自动清空</string>
            </property>
           </widget>
          </item>
          <item row="2" column="0" colspan="2">
           <layout class="QGridLayout" name="gridLayout">
            <item row="0" column="0">
             <widget class="QCheckBox" name="ckAutoSend">
              <property name="text">
               <string>自动发送</string>
              </property>
             </widget>
            </item>
            <item row="0" column="1">
             <widget class="QComboBox" name="cboxSendInterval">
              <property name="sizePolicy">
               <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                <horstretch>0</horstretch>
                <verstretch>0</verstretch>
               </sizepolicy>
              </property>
             </widget>
            </item>
            <item row="1" column="0">
             <widget class="QCheckBox" name="ckAutoSave">
              <property name="text">
               <string>自动保存</string>
              </property>
             </widget>
            </item>
            <item row="1" column="1">
             <widget class="QComboBox" name="cboxSaveInterval">
              <property name="sizePolicy">
               <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                <horstretch>0</horstretch>
                <verstretch>0</verstretch>
               </sizepolicy>
              </property>
             </widget>
            </item>
           </layout>
          </item>
          <item row="3" column="0" colspan="2">
           <widget class="QPushButton" name="btnSendCount">
            <property name="text">
             <string>发送 : 0 字节</string>
            </property>
           </widget>
          </item>
          <item row="4" column="0" colspan="2">
           <widget class="QPushButton" name="btnReceiveCount">
            <property name="text">
             <string>接收 : 0 字节</string>
            </property>
           </widget>
          </item>
          <item row="5" column="0" colspan="2">
           <widget class="QPushButton" name="btnStopShow">
            <property name="text">
             <string>停止显示</string>
            </property>
           </widget>
          </item>
          <item row="6" column="0" colspan="2">
           <widget class="QPushButton" name="btnSave">
            <property name="text">
             <string>保存数据</string>
            </property>
           </widget>
          </item>
          <item row="7" column="0" colspan="2">
           <widget class="QPushButton" name="btnData">
            <property name="text">
             <string>管理数据</string>
            </property>
           </widget>
          </item>
          <item row="8" column="0" colspan="2">
           <widget class="QPushButton" name="btnClear">
            <property name="text">
             <string>清空数据</string>
            </property>
           </widget>
          </item>
          <item row="9" column="0" colspan="2">
           <spacer name="verticalSpacer">
            <property name="orientation">
             <enum>Qt::Vertical</enum>
            </property>
            <property name="sizeHint" stdset="0">
             <size>
              <width>20</width>
              <height>2</height>
             </size>
            </property>
           </spacer>
          </item>
         </layout>
        </widget>
        <widget class="QWidget" name="tab_2">
         <attribute name="title">
          <string>网络配置</string>
         </attribute>
         <layout class="QGridLayout" name="gridLayout_2">
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
          <item row="0" column="0" colspan="2">
           <layout class="QGridLayout" name="gridLayout_4">
            <item row="3" column="0">
             <widget class="QLabel" name="labListenPort">
              <property name="text">
               <string>监听端口</string>
              </property>
             </widget>
            </item>
            <item row="1" column="0">
             <widget class="QLabel" name="labServerIP">
              <property name="text">
               <string>远程地址</string>
              </property>
             </widget>
            </item>
            <item row="4" column="1">
             <widget class="QComboBox" name="cboxSleepTime"/>
            </item>
            <item row="1" column="1">
             <widget class="QLineEdit" name="txtServerIP">
              <property name="text">
               <string/>
              </property>
             </widget>
            </item>
            <item row="2" column="0">
             <widget class="QLabel" name="labServerPort">
              <property name="text">
               <string>远程端口</string>
              </property>
             </widget>
            </item>
            <item row="2" column="1">
             <widget class="QLineEdit" name="txtServerPort">
              <property name="text">
               <string/>
              </property>
             </widget>
            </item>
            <item row="4" column="0">
             <widget class="QLabel" name="labSleepTime">
              <property name="text">
               <string>延时时间</string>
              </property>
             </widget>
            </item>
            <item row="3" column="1">
             <widget class="QLineEdit" name="txtListenPort"/>
            </item>
            <item row="0" column="0">
             <widget class="QLabel" name="labMode">
              <property name="text">
               <string>转换模式</string>
              </property>
             </widget>
            </item>
            <item row="0" column="1">
             <widget class="QComboBox" name="cboxMode">
              <item>
               <property name="text">
                <string>Tcp_Client</string>
               </property>
              </item>
              <item>
               <property name="text">
                <string>Tcp_Server</string>
               </property>
              </item>
              <item>
               <property name="text">
                <string>Udp_Client</string>
               </property>
              </item>
              <item>
               <property name="text">
                <string>Udp_Server</string>
               </property>
              </item>
             </widget>
            </item>
           </layout>
          </item>
          <item row="2" column="0" colspan="2">
           <widget class="QPushButton" name="btnStart">
            <property name="text">
             <string>启动</string>
            </property>
           </widget>
          </item>
          <item row="3" column="0" colspan="2">
           <spacer name="verticalSpacer_2">
            <property name="orientation">
             <enum>Qt::Vertical</enum>
            </property>
            <property name="sizeHint" stdset="0">
             <size>
              <width>20</width>
              <height>59</height>
             </size>
            </property>
           </spacer>
          </item>
          <item row="1" column="0" colspan="2">
           <widget class="QCheckBox" name="ckAutoConnect">
            <property name="text">
             <string>自动重连网络</string>
            </property>
           </widget>
          </item>
         </layout>
        </widget>
       </widget>
      </item>
     </layout>
    </widget>
   </item>
   <item row="1" column="0">
    <widget class="QWidget" name="widget" native="true">
     <layout class="QHBoxLayout" name="horizontalLayout">
      <property name="leftMargin">
       <number>0</number>
      </property>
      <property name="topMargin">
       <number>0</number>
      </property>
      <property name="rightMargin">
       <number>0</number>
      </property>
      <property name="bottomMargin">
       <number>0</number>
      </property>
      <item>
       <widget class="QComboBox" name="cboxData">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Ignored" vsizetype="Fixed">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="editable">
         <bool>true</bool>
        </property>
        <property name="duplicatesEnabled">
         <bool>false</bool>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnSend">
        <property name="minimumSize">
         <size>
          <width>80</width>
          <height>0</height>
         </size>
        </property>
        <property name="maximumSize">
         <size>
          <width>80</width>
          <height>16777215</height>
         </size>
        </property>
        <property name="text">
         <string>发送</string>
        </property>
       </widget>
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

### `api/api.pri`

```makefile
HEADERS += \
    $$PWD/appconfig.h \
    $$PWD/appdata.h \
    $$PWD/qthelper.h \
    $$PWD/qthelperdata.h

SOURCES += \
    $$PWD/appconfig.cpp \
    $$PWD/appdata.cpp \
    $$PWD/qthelper.cpp \
    $$PWD/qthelperdata.cpp
```

### `form/form.pri`

```makefile
FORMS += \
    $$PWD/frmcomtool.ui

HEADERS += \
    $$PWD/frmcomtool.h

SOURCES += \
    $$PWD/frmcomtool.cpp
```
