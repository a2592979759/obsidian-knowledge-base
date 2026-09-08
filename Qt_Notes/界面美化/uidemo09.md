---
tags:
  - Qt
  - 界面美化
---
# 九宫格主界面 (`uidemo09`)

> 九宫格主界面

## 效果图

![[QWidgetDemo_assert/uidemo09.jpg]]

## 分类

- 类型: 界面美化 `ui`
- 源码目录: `ui/uidemo09`

## 技术要点

**用到的 Qt 类**: QToolButton QWidget QString QApplication QList QVBoxLayout QGridLayout

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
    //加载样式表
    QtHelper::setStyle(":/qss/blacksoft.css");

    frmMain w;
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
    this->setWindowTitle("九宫格主界面");

    bg = "main2.jpg";
    QList<QToolButton *> btns = this->findChildren<QToolButton *>();

    foreach (QToolButton *btn, btns) {
        connect(btn, SIGNAL(clicked()), this, SLOT(buttonClick()));
    }
}

void frmMain::buttonClick()
{
    QToolButton *btn = (QToolButton *)sender();
    QString objName = btn->objectName();

    if (objName == "btnCOMTool") {
        if (bg == "main1.jpg") {
            bg = "main2.jpg";
        } else if (bg == "main2.jpg") {
            bg = "main3.jpg";
        } else if (bg == "main3.jpg") {
            bg = "main4.jpg";
        } else if (bg == "main4.jpg") {
            bg = "main5.jpg";
        } else if (bg == "main5.jpg") {
            bg = "main1.jpg";
        }

        QString qss = QString("QWidget#frm{background-image: url(:/image/%1);}").arg(bg);
        qss += "QToolButton{color:#E7ECF0;background-color:rgba(0,0,0,0);border-style:none;}";
        this->setStyleSheet(qss);
    } else if (objName == "btnAddressTool") {

    } else if (objName == "btnTCPTool") {
        this->close();
    } else if (objName == "btnCOMTCPTool") {

    } else if (objName == "btnDefence") {

    } else if (objName == "btnHostTool") {

    } else if (objName == "btnLinkTool") {

    } else if (objName == "btnMaiChongTool") {

    } else if (objName == "btnPlot") {

    } else if (objName == "btnZhangLi4Tool") {

    } else if (objName == "btnZhangLi5Tool") {

    } else if (objName == "btnZhangLiTool") {

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
    QString bg;

private slots:
    void initForm();
    void buttonClick();
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
  <property name="styleSheet">
   <string notr="true">QWidget#frm{
background-image: url(:/image/main2.jpg);
}
QToolButton{
font: 10pt &quot;微软雅黑&quot;;
color:#E7ECF0;
background-color:rgba(0,0,0,0);
border-style:none;
}</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <property name="spacing">
    <number>0</number>
   </property>
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
    <widget class="QWidget" name="frm" native="true">
     <layout class="QGridLayout" name="gridLayout">
      <property name="leftMargin">
       <number>30</number>
      </property>
      <property name="topMargin">
       <number>30</number>
      </property>
      <property name="rightMargin">
       <number>30</number>
      </property>
      <property name="bottomMargin">
       <number>30</number>
      </property>
      <property name="spacing">
       <number>6</number>
      </property>
      <item row="0" column="0">
       <widget class="QToolButton" name="btnCOMTool">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>更换背景</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG112.png</normaloff>:/image/PNG112.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="0" column="1">
       <widget class="QToolButton" name="btnTCPTool">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>退出系统</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG100.png</normaloff>:/image/PNG100.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="0" column="2">
       <widget class="QToolButton" name="btnCOMTCPTool">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>串口转网络工具</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG99.png</normaloff>:/image/PNG99.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="0" column="3">
       <widget class="QToolButton" name="btnAddressTool">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>设备调试工具</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG101.png</normaloff>:/image/PNG101.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="1" column="0">
       <widget class="QToolButton" name="btnZhangLiTool">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>张力批量配置工具</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG103.png</normaloff>:/image/PNG103.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="1" column="1">
       <widget class="QToolButton" name="btnZhangLi5Tool">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>5道张力调试工具</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG104.png</normaloff>:/image/PNG104.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="1" column="2">
       <widget class="QToolButton" name="btnMaiChongTool">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>脉冲主机调试工具</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG105.png</normaloff>:/image/PNG105.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="1" column="3">
       <widget class="QToolButton" name="btnHostTool">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>主机内部调试工具</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG108.png</normaloff>:/image/PNG108.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="2" column="0">
       <widget class="QToolButton" name="btnLinkTool">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>联动模块调试工具</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG109.png</normaloff>:/image/PNG109.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="2" column="1">
       <widget class="QToolButton" name="btnPlot">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>张力曲线图</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG110.png</normaloff>:/image/PNG110.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="2" column="2">
       <widget class="QToolButton" name="btnDefence">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>防区管理</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG111.png</normaloff>:/image/PNG111.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
      <item row="2" column="3">
       <widget class="QToolButton" name="btnConfig">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>系统配置</string>
        </property>
        <property name="icon">
         <iconset resource="../other/main.qrc">
          <normaloff>:/image/PNG114.png</normaloff>:/image/PNG114.png</iconset>
        </property>
        <property name="iconSize">
         <size>
          <width>100</width>
          <height>100</height>
         </size>
        </property>
        <property name="toolButtonStyle">
         <enum>Qt::ToolButtonTextUnderIcon</enum>
        </property>
       </widget>
      </item>
     </layout>
    </widget>
   </item>
  </layout>
 </widget>
 <resources>
  <include location="../other/main.qrc"/>
 </resources>
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
