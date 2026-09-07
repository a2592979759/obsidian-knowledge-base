---
tags:
  - Qt
  - 工具相关
---
# 程序启动器 (`livetool`)

> 程序启动器

## 效果图

![[QWidgetDemo_assert/livetool.jpg]]

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/livetool`

## 技术要点

**用到的 Qt 类**: QString QSystemTrayIcon QTextCodec QWidget QProcess QObject QTimer QSettings QEvent QUdpSocket QFrame QPushButton QMenu QFile QStringList QStandardPaths QDesktopServices QMessageBox QLabel QApplication

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./app.cpp`

```cpp
﻿#include "app.h"
#include "qsettings.h"
#include "qfile.h"

QString App::ConfigFile = "config.ini";
QString App::TargetAppName = "livedemo";
int App::TargetAppPort = 6666;
bool App::ReStartExplorer = false;
int App::TimeoutCount = 3;
int App::ReStartCount = 0;
QString App::ReStartLastTime = "2019-01-01 00:00:00";

void App::readConfig()
{
    if (!checkConfig()) {
        return;
    }

    QSettings set(App::ConfigFile, QSettings::IniFormat);
    set.beginGroup("BaseConfig");
    App::TargetAppName = set.value("TargetAppName", App::TargetAppName).toString();
    App::TargetAppPort = set.value("TargetAppPort", App::TargetAppPort).toInt();
    App::ReStartExplorer = set.value("ReStartExplorer", App::ReStartExplorer).toBool();
    App::TimeoutCount = set.value("TimeoutCount", App::TimeoutCount).toInt();
    App::ReStartCount = set.value("ReStartCount", App::ReStartCount).toInt();
    App::ReStartLastTime = set.value("ReStartLastTime", App::ReStartLastTime).toString();
    set.endGroup();
}

void App::writeConfig()
{
    QSettings set(App::ConfigFile, QSettings::IniFormat);
    set.beginGroup("BaseConfig");
    set.setValue("TargetAppName", App::TargetAppName);
    set.setValue("TargetAppPort", App::TargetAppPort);
    set.setValue("ReStartExplorer", App::ReStartExplorer);
    set.setValue("TimeoutCount", App::TimeoutCount);
    set.setValue("ReStartCount", App::ReStartCount);
    set.setValue("ReStartLastTime", App::ReStartLastTime);
    set.endGroup();
}

bool App::checkConfig()
{
    //如果配置文件大小为0,则以初始值继续运行,并生成配置文件
    QFile file(App::ConfigFile);
    if (file.size() == 0) {
        writeConfig();
        return false;
    }

    //如果配置文件不完整,则以初始值继续运行,并生成配置文件
    if (file.open(QFile::ReadOnly)) {
        bool ok = true;
        while (!file.atEnd()) {
            QString line = file.readLine();
            line = line.replace("\r", "");
            line = line.replace("\n", "");
            QStringList list = line.split("=");

            if (list.count() == 2) {
                if (list.at(1) == "") {
                    ok = false;
                    break;
                }
            }
        }

        if (!ok) {
            writeConfig();
            return false;
        }
    } else {
        writeConfig();
        return false;
    }

    return true;
}
```

### `./app.h`

```cpp
﻿#ifndef APP_H
#define APP_H

#include <QStringList>

class App
{
public:
    static QString ConfigFile;          //配置文件文件路径及名称
    static QString TargetAppName;       //目标软件程序名称
    static int TargetAppPort;           //目标软件通信端口
    static bool ReStartExplorer;        //是否需要重启桌面
    static int TimeoutCount;            //超时次数
    static int ReStartCount;            //已重启次数
    static QString ReStartLastTime;     //最后一次重启时间

    static void readConfig();           //读取配置文件,在main函数最开始加载程序载入
    static void writeConfig();          //写入配置文件,在更改配置文件程序关闭时调用
    static bool checkConfig();          //校验配置文件    

};

#endif // APP_H
```

### `./frmmain.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
#include "ui_frmmain.h"
#include "qtimer.h"
#include "qudpsocket.h"
#include "qsharedmemory.h"
#include "qprocess.h"
#include "qdatetime.h"
#include "qapplication.h"
#include "qdesktopservices.h"
#include "qmessagebox.h"
#if (QT_VERSION > QT_VERSION_CHECK(5,0,0))
#include "qstandardpaths.h"
#endif

#include "app.h"

frmMain::frmMain(QWidget *parent) : QWidget(parent), ui(new Ui::frmMain)
{
    ui->setupUi(this);
    this->initForm();
}

frmMain::~frmMain()
{
    delete ui;
}

void frmMain::changeEvent(QEvent *event)
{
    //隐藏当前界面,最小化到托盘
    if(event->type() == QEvent::WindowStateChange) {
        if(windowState() & Qt::WindowMinimized) {
            hide();
        }
    }

    QWidget::changeEvent(event);
}

void frmMain::initForm()
{
    count = 0;
    ok = false;

    //每秒钟定时询问心跳
    timerHeart = new QTimer(this);
    timerHeart->setInterval(2000);
    connect(timerHeart, SIGNAL(timeout()), this, SLOT(sendHearData()));

    //从6050端口开始,如果绑定失败则将端口加1,直到绑定成功
    udp = new QUdpSocket(this);
    int port = 6050;
    while(!udp->bind(port)) {
        port++;
    }

    connect(udp, SIGNAL(readyRead()), this, SLOT(readData()));

    if (App::TargetAppName.isEmpty()) {
        ui->btnStart->setText("启动");
        ui->btnStart->setEnabled(false);
        timerHeart->stop();
    } else {
        ui->btnStart->setText("暂停");
        ui->btnStart->setEnabled(true);
        timerHeart->start();
    }

    ui->txtAppName->setText(App::TargetAppName);
    ui->txtAppName->setFocus();
}

void frmMain::sendHearData()
{
    udp->writeDatagram("hello", QHostAddress::LocalHost, App::TargetAppPort);

    //判断当前是否没有回复
    if (!ok) {
        count++;
    } else {
        count = 0;
        ok = false;
    }

    //如果超过规定次数没有收到心跳回复,则超时重启
    if (count >= App::TimeoutCount) {
        timerHeart->stop();

        QSharedMemory mem(App::TargetAppName);
        if (!mem.create(1)) {
            killApp();
        }

        QTimer::singleShot(1000 , this, SLOT(killOther()));
        QTimer::singleShot(3000 , this, SLOT(startApp()));
        QTimer::singleShot(4000 , this, SLOT(startExplorer()));
    }
}

void frmMain::killApp()
{
    QProcess *p = new QProcess;
    p->start(QString("taskkill /im %1.exe /f").arg(App::TargetAppName));
}

void frmMain::killOther()
{
    QProcess *p = new QProcess;
    p->start(QString("taskkill /im %1.exe /f").arg("WerFault"));

    //重建缓存,彻底清除托盘图标
    if (App::ReStartExplorer) {
        QProcess *p1 = new QProcess;
        p1->start("taskkill /f /im explorer.exe");
    }
}

void frmMain::startApp()
{
    if (ui->btnStart->text() == "开始" || ui->btnStart->text() == "启动") {
        count = 0;
        return;
    }

    QProcess *p = new QProcess;
    p->start(QString("\"%1/%2.exe\"").arg(qApp->applicationDirPath()).arg(App::TargetAppName));

    count = 0;
    ok = true;
    timerHeart->start();

    App::ReStartCount++;
    App::ReStartLastTime = QDateTime::currentDateTime().toString("yyyy-MM-dd HH:mm:ss");
    App::writeConfig();

    ui->labCount->setText(QString("已重启 %1 次").arg(App::ReStartCount));
    ui->labInfo->setText(QString("最后一次重启在 %1").arg(App::ReStartLastTime));
}

void frmMain::startExplorer()
{
    //取得操作系统目录路径,指定操作系统目录下的explorer程序,采用绝对路径,否则在64位操作系统下无效
#if (QT_VERSION > QT_VERSION_CHECK(5,0,0))
    QString str = QStandardPaths::writableLocation(QStandardPaths::ApplicationsLocation);
#else
    QString str = QDesktopServices::storageLocation(QDesktopServices::ApplicationsLocation);
#endif

    if (App::ReStartExplorer) {
        str = QString("%1\\Windows\\explorer.exe").arg(str.mid(0, 2));
        QProcess *p = new QProcess(this);
        p->start(str);
    }
}

void frmMain::readData()
{
    QByteArray tempData;
    do {
        tempData.resize(udp->pendingDatagramSize());
        udp->readDatagram(tempData.data(), tempData.size());
        QString data = QLatin1String(tempData);
        if (data.right(2) == "OK") {
            count = 0;
            ok = true;
        }
    } while (udp->hasPendingDatagrams());
}

void frmMain::on_btnOk_clicked()
{
    App::TargetAppName = ui->txtAppName->text();
    if (App::TargetAppName == "") {
        QMessageBox::critical(this, "提示", "应用程序名称不能为空!");
        ui->txtAppName->setFocus();
        return;
    }

    App::writeConfig();
    ui->btnStart->setEnabled(true);
}

void frmMain::on_btnStart_clicked()
{
    count = 0;
    if (ui->btnStart->text() == "暂停") {
        timerHeart->stop();
        ui->btnStart->setText("开始");
    } else {
        timerHeart->start();
        ui->btnStart->setText("暂停");
    }
}

void frmMain::on_btnReset_clicked()
{
    App::ReStartCount = 0;
    App::ReStartLastTime = "2019-01-01 12:00:00";
    App::writeConfig();

    ui->txtAppName->setText(App::TargetAppName);
    ui->labCount->setText(QString("已重启 %1 次").arg(App::ReStartCount));
    ui->labInfo->setText(QString("最后一次重启在 %1").arg(App::ReStartLastTime));
    QMessageBox::information(this, "提示", "重置配置文件成功!");
}
```

### `./frmmain.h`

```cpp
﻿#ifndef FRMMAIN_H
#define FRMMAIN_H

#include <QWidget>

class QUdpSocket;

namespace Ui
{
    class frmMain;
}

class frmMain : public QWidget
{
    Q_OBJECT

public:
    explicit frmMain(QWidget *parent = 0);
    ~frmMain();

protected:
    void changeEvent(QEvent *event);

private:
    Ui::frmMain *ui;
    QTimer *timerHeart;     //心跳定时器
    QUdpSocket *udp;        //UDP通信对象
    int count;              //计数
    bool ok;                //是否正常

private slots:
    void initForm();
    void sendHearData();
    void readData();
    void killApp();
    void killOther();
    void startApp();
    void startExplorer();

private slots:
    void on_btnOk_clicked();
    void on_btnStart_clicked();
    void on_btnReset_clicked();
};

#endif // FRMMAIN_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
#include "trayicon.h"
#include "app.h"
#include <QApplication>
#include <QTextCodec>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    a.setWindowIcon(QIcon(":/main.ico"));

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

    App::ConfigFile = qApp->applicationDirPath() + "/livetool.ini";
    App::readConfig();

    frmMain w;
    w.setWindowTitle("程序启动器 (QQ: 517216493 WX: feiyangqingyun)");
    w.setFixedSize(w.sizeHint());

    //启动托盘类
    TrayIcon::Instance()->setMainWidget(&w);
    TrayIcon::Instance()->setIcon(":/main.ico");
    TrayIcon::Instance()->setToolTip(w.windowTitle());
    TrayIcon::Instance()->setVisible(true);
    QObject::connect(&w, SIGNAL(destroyed(QObject *)), TrayIcon::Instance(), SLOT(closeAll()));
    App::TargetAppName.isEmpty() ? w.show() : w.hide();

    return a.exec();
}
```

### `./trayicon.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "trayicon.h"
#include "qmutex.h"
#include "qmenu.h"
#include "qapplication.h"
#include "qdebug.h"

QScopedPointer<TrayIcon> TrayIcon::self;
TrayIcon *TrayIcon::Instance()
{
    if (self.isNull()) {
        static QMutex mutex;
        QMutexLocker locker(&mutex);
        if (self.isNull()) {
            self.reset(new TrayIcon);
        }
    }

    return self.data();
}

TrayIcon::TrayIcon(QObject *parent) : QObject(parent)
{
    mainWidget = 0;
    trayIcon = new QSystemTrayIcon(this);
    connect(trayIcon, SIGNAL(activated(QSystemTrayIcon::ActivationReason)),
            this, SLOT(iconIsActived(QSystemTrayIcon::ActivationReason)));
    menu = new QMenu;
    exitDirect = true;
}

void TrayIcon::iconIsActived(QSystemTrayIcon::ActivationReason reason)
{
    switch (reason) {
        case QSystemTrayIcon::Trigger:
        case QSystemTrayIcon::DoubleClick: {
            this->showMainWidget();
            break;
        }

        default:
            break;
    }
}

void TrayIcon::setExitDirect(bool exitDirect)
{
    if (this->exitDirect != exitDirect) {
        this->exitDirect = exitDirect;
    }
}

void TrayIcon::setMainWidget(QWidget *mainWidget)
{
    this->mainWidget = mainWidget;
    menu->addAction("主界面", this, SLOT(showMainWidget()));

    if (exitDirect) {
        menu->addAction("退出", this, SLOT(closeAll()));
    } else {
        menu->addAction("退出", this, SIGNAL(trayIconExit()));
    }

    trayIcon->setContextMenu(menu);
}

void TrayIcon::showMainWidget()
{
    if (mainWidget) {
        mainWidget->showNormal();
        mainWidget->activateWindow();
    }
}

void TrayIcon::showMessage(const QString &title, const QString &msg, QSystemTrayIcon::MessageIcon icon, int msecs)
{
    trayIcon->showMessage(title, msg, icon, msecs);
}

void TrayIcon::setIcon(const QString &strIcon)
{
    trayIcon->setIcon(QIcon(strIcon));
}

void TrayIcon::setToolTip(const QString &tip)
{
    trayIcon->setToolTip(tip);
}

bool TrayIcon::getVisible() const
{
    return trayIcon->isVisible();
}

void TrayIcon::setVisible(bool visible)
{
    trayIcon->setVisible(visible);
}

void TrayIcon::closeAll()
{
    trayIcon->hide();
    trayIcon->deleteLater();
    qApp->exit();
}
```

### `./trayicon.h`

```cpp
﻿#ifndef TRAYICON_H
#define TRAYICON_H

/**
 * 托盘图标控件 作者:feiyangqingyun(QQ:517216493) 2017-01-08
 * 1. 可设置托盘图标对应所属主窗体。
 * 2. 可设置托盘图标。
 * 3. 可设置提示信息。
 * 4. 自带右键菜单。
 */

#include <QObject>
#include <QSystemTrayIcon>

class QMenu;

#ifdef quc
class Q_DECL_EXPORT TrayIcon : public QObject
#else
class TrayIcon : public QObject
#endif

{
    Q_OBJECT
public:
    static TrayIcon *Instance();
    explicit TrayIcon(QObject *parent = 0);

private:
    static QScopedPointer<TrayIcon> self;
    QWidget *mainWidget;            //对应所属主窗体
    QSystemTrayIcon *trayIcon;      //托盘对象
    QMenu *menu;                    //右键菜单
    bool exitDirect;                //是否直接退出

private slots:
    void iconIsActived(QSystemTrayIcon::ActivationReason reason);

public:
    //设置是否直接退出,如果不是直接退出则发送信号给主界面
    void setExitDirect(bool exitDirect);

    //设置所属主窗体
    void setMainWidget(QWidget *mainWidget);    

    //显示消息
    void showMessage(const QString &title, const QString &msg,
                     QSystemTrayIcon::MessageIcon icon = QSystemTrayIcon::Information, int msecs = 5000);

    //设置图标
    void setIcon(const QString &strIcon);
    //设置提示信息
    void setToolTip(const QString &tip);

    //获取和设置是否可见
    bool getVisible() const;
    void setVisible(bool visible);

public Q_SLOTS:
    //退出所有
    void closeAll();
    //显示主窗体
    void showMainWidget();

Q_SIGNALS:
    void trayIconExit();
};

#endif // TRAYICON_H
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
    <width>288</width>
    <height>125</height>
   </rect>
  </property>
  <property name="sizeGripEnabled" stdset="0">
   <bool>false</bool>
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
    <widget class="QFrame" name="frame">
     <property name="frameShape">
      <enum>QFrame::Box</enum>
     </property>
     <property name="frameShadow">
      <enum>QFrame::Sunken</enum>
     </property>
     <layout class="QGridLayout" name="gridLayout">
      <item row="3" column="0">
       <widget class="QPushButton" name="btnOk">
        <property name="minimumSize">
         <size>
          <width>70</width>
          <height>0</height>
         </size>
        </property>
        <property name="text">
         <string>应用</string>
        </property>
       </widget>
      </item>
      <item row="3" column="1">
       <widget class="QPushButton" name="btnStart">
        <property name="minimumSize">
         <size>
          <width>70</width>
          <height>0</height>
         </size>
        </property>
        <property name="text">
         <string>暂停</string>
        </property>
       </widget>
      </item>
      <item row="3" column="2">
       <widget class="QPushButton" name="btnReset">
        <property name="text">
         <string>重置</string>
        </property>
       </widget>
      </item>
      <item row="2" column="0" colspan="3">
       <widget class="QLineEdit" name="txtAppName"/>
      </item>
      <item row="1" column="0" colspan="3">
       <widget class="QLabel" name="labInfo">
        <property name="text">
         <string>最后一次重启在</string>
        </property>
       </widget>
      </item>
      <item row="0" column="0" colspan="3">
       <widget class="QLabel" name="labCount">
        <property name="text">
         <string>已重启 0 次</string>
        </property>
       </widget>
      </item>
     </layout>
    </widget>
   </item>
  </layout>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <tabstops>
  <tabstop>txtAppName</tabstop>
  <tabstop>btnOk</tabstop>
  <tabstop>btnStart</tabstop>
  <tabstop>btnReset</tabstop>
 </tabstops>
 <resources/>
 <connections/>
</ui>
```
