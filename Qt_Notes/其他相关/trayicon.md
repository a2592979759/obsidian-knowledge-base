---
tags:
  - Qt
  - 其他相关
---
# 通用托盘效果 (`trayicon`)

> 通用托盘效果

## 效果图

![[QWidgetDemo_assert/trayicon.jpg]]

## 分类

- 类型: 其他相关 `other`
- 源码目录: `other/trayicon`

## 技术要点

**用到的 Qt 类**: QSystemTrayIcon QTextCodec QWidget QString QObject QMenu QPushButton QApplication QScopedPointer QFont QMutex QMutexLocker QIcon

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmtrayicon.cpp`

```cpp
﻿#include "frmtrayicon.h"
#include "ui_frmtrayicon.h"
#include "trayicon.h"

frmTrayIcon::frmTrayIcon(QWidget *parent) : QWidget(parent), ui(new Ui::frmTrayIcon)
{
    ui->setupUi(this);
    TrayIcon::Instance()->setIcon(":/main.ico");
    TrayIcon::Instance()->setMainWidget(this);
}

frmTrayIcon::~frmTrayIcon()
{
    TrayIcon::Instance()->setVisible(false);
    delete ui;
}

void frmTrayIcon::on_btnShow_clicked()
{
    TrayIcon::Instance()->setVisible(true);
    TrayIcon::Instance()->showMessage("自定义控件大全", "已经最小化到托盘,双击打开!");
}

void frmTrayIcon::on_btnHide_clicked()
{
    TrayIcon::Instance()->setVisible(false);
}
```

### `./frmtrayicon.h`

```cpp
﻿#ifndef FRMTRAYICON_H
#define FRMTRAYICON_H

#include <QWidget>

namespace Ui {
class frmTrayIcon;
}

class frmTrayIcon : public QWidget
{
    Q_OBJECT

public:
    explicit frmTrayIcon(QWidget *parent = 0);
    ~frmTrayIcon();

private:
    Ui::frmTrayIcon *ui;

private slots:
    void on_btnShow_clicked();
    void on_btnHide_clicked();
};

#endif // FRMTRAYICON_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmtrayicon.h"
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

    frmTrayIcon w;
    w.setWindowTitle("托盘图标 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

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

### `./frmtrayicon.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmTrayIcon</class>
 <widget class="QWidget" name="frmTrayIcon">
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
  <widget class="QPushButton" name="btnShow">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>10</y>
     <width>92</width>
     <height>28</height>
    </rect>
   </property>
   <property name="text">
    <string>显示托盘</string>
   </property>
  </widget>
  <widget class="QPushButton" name="btnHide">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>50</y>
     <width>92</width>
     <height>28</height>
    </rect>
   </property>
   <property name="text">
    <string>隐藏托盘</string>
   </property>
  </widget>
 </widget>
 <resources/>
 <connections/>
</ui>
```
