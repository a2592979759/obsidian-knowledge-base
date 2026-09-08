---
tags:
  - Qt
  - 界面美化
  - 高质量
---
# 扁平化主界面 (`uidemo10`)

> 扁平化主界面

## 效果图

![[QWidgetDemo_assert/uidemo10.jpg]]

## 分类

- 类型: 界面美化 `ui`
- 源码目录: `ui/uidemo10`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QToolButton QWidget QList QString QApplication QSize QPixmap QIcon QGridLayout

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./head.h`

```cpp
﻿#include <QtCore>
#include <QtGui>

#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
#include <QtWidgets>
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
#include <QtCore5Compat>
#endif

#pragma execution_character_set("utf-8")
```

### `./main.cpp`

```cpp
﻿#include "frmmain.h"
#include "appinit.h"
#include "qthelper.h"

int main(int argc, char *argv[])
{
    QtHelper::initMain();
    QApplication a(argc, argv);
    AppInit::Instance()->start();

    QtHelper::setFont();
    QtHelper::setCode();

    frmMain w;
    w.setWindowTitle("metro风格主界面 (QQ: 517216493 WX: feiyangqingyun)");
    QtHelper::setFormInCenter(&w);
    w.show();

    return a.exec();
}
```

### `form/frmmain.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
#include "ui_frmmain.h"
#include "iconhelper.h"

frmMain::frmMain(QWidget *parent) : QWidget(parent), ui(new Ui::frmMain)
{
    ui->setupUi(this);
    this->initForm();   
}

frmMain::~frmMain()
{
    delete ui;
}

void frmMain::initForm()
{
    int iconSize = 100;
    int iconWidth = 150;
    int iconHeight = 130;

    QList<QString> listColorBg;
    listColorBg << "#FF3739" << "#1A9FE0" << "#41BB1A" << "#1570A5" << "#FE781F" << "#9B59BB";
    QList<QString> listColorText;
    listColorText << "#FEFEFE" << "#FEFEFE" << "#FEFEFE" << "#FEFEFE" << "#FEFEFE" << "#FEFEFE";

    QList<int> icons;
    icons << 0xf2ba << 0xf002 << 0xf2c2 << 0xf02f << 0xf013 << 0xf021;
    QList<QString> names;
    names << "访客登记" << "记录查询" << "证件扫描" << "信息打印" << "系统设置" << "系统重启";

    QList<QToolButton *> btns = this->findChildren<QToolButton *>();
    for (int i = 0; i < btns.count(); ++i) {
        QToolButton *btn = btns.at(i);
        btn->setToolButtonStyle(Qt::ToolButtonTextUnderIcon);
        btn->setIconSize(QSize(iconWidth, iconHeight));

        QPixmap pix = IconHelper::getPixmap(listColorText.at(i), icons.at(i), iconSize, iconWidth, iconHeight);
        btn->setIcon(QIcon(pix));
        btn->setText(names.at(i));
        btn->setStyleSheet(QString("QToolButton{font:%1px;color:%2;background-color:%3;border:none;border-radius:0px;}")
                           .arg(iconSize / 2).arg(listColorText.at(i)).arg(listColorBg.at(i)));

        connect(btn, SIGNAL(clicked(bool)), this, SLOT(buttonClicked()));
    }
}

void frmMain::buttonClicked()
{
    QToolButton *btn = (QToolButton *)sender();
    QString text = btn->text();
    qDebug() << text;

    if (text == "系统重启") {
        close();
    }
}
```

### `form/frmmain.h`

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
    void initForm();
    void buttonClicked();
};

#endif // FRMMAIN_H
```

### `form/frmmain.ui`

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
    <height>480</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Dialog</string>
  </property>
  <layout class="QGridLayout" name="gridLayout">
   <item row="0" column="0">
    <widget class="QToolButton" name="btn1">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="text">
      <string/>
     </property>
     <property name="toolButtonStyle">
      <enum>Qt::ToolButtonTextUnderIcon</enum>
     </property>
    </widget>
   </item>
   <item row="0" column="1">
    <widget class="QToolButton" name="btn2">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="text">
      <string/>
     </property>
     <property name="toolButtonStyle">
      <enum>Qt::ToolButtonTextUnderIcon</enum>
     </property>
    </widget>
   </item>
   <item row="0" column="2">
    <widget class="QToolButton" name="btn3">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="text">
      <string/>
     </property>
     <property name="toolButtonStyle">
      <enum>Qt::ToolButtonTextUnderIcon</enum>
     </property>
    </widget>
   </item>
   <item row="1" column="0">
    <widget class="QToolButton" name="btn4">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="text">
      <string/>
     </property>
     <property name="toolButtonStyle">
      <enum>Qt::ToolButtonTextUnderIcon</enum>
     </property>
    </widget>
   </item>
   <item row="1" column="1">
    <widget class="QToolButton" name="btn5">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="text">
      <string/>
     </property>
     <property name="toolButtonStyle">
      <enum>Qt::ToolButtonTextUnderIcon</enum>
     </property>
    </widget>
   </item>
   <item row="1" column="2">
    <widget class="QToolButton" name="btn6">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="text">
      <string/>
     </property>
     <property name="toolButtonStyle">
      <enum>Qt::ToolButtonTextUnderIcon</enum>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <resources/>
 <connections/>
</ui>
```

### `form/form.pri`

```makefile
FORMS += \
    $$PWD/frmmain.ui

HEADERS += \
    $$PWD/frmmain.h

SOURCES += \
    $$PWD/frmmain.cpp
```
