---
tags:
  - Qt
  - 界面美化
---
# 界面美化基础示例 (`uidemo01`)

> 界面美化基础示例

## 效果图

![[QWidgetDemo_assert/uidemo01.jpg]]

## 分类

- 类型: 界面美化 `ui`
- 源码目录: `ui/uidemo01`

## 技术要点

**用到的 Qt 类**: QWidget QToolButton QLabel QString QEvent QPushButton QVBoxLayout QDialog QScrollBar QObject QComboBox QDateEdit QTimeEdit QDateTimeEdit QList QHBoxLayout QApplication QFont QStringList QRadioButton

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
    w.resize(800, 600);
    QtHelper::setFormInCenter(&w);
    w.show();

    return a.exec();
}
```

### `form/frmmain.cpp`

```cpp
﻿#include "frmmain.h"
#include "ui_frmmain.h"
#include "iconhelper.h"
#include "qthelper.h"

frmMain::frmMain(QWidget *parent) : QDialog(parent), ui(new Ui::frmMain)
{
    ui->setupUi(this);
    this->initForm();
}

frmMain::~frmMain()
{
    delete ui;
}

bool frmMain::eventFilter(QObject *watched, QEvent *event)
{
    if (event->type() == QEvent::MouseButtonDblClick) {
        if (watched == ui->widgetTitle) {
            on_btnMenu_Max_clicked();
            return true;
        }
    }

    return QWidget::eventFilter(watched, event);
}

void frmMain::initForm()
{
    //设置无边框
    QtHelper::setFramelessForm(this);
    //设置图标
    IconHelper::setIcon(ui->labIco, 0xf099, 35);
    IconHelper::setIcon(ui->btnMenu_Min, 0xf068);
    IconHelper::setIcon(ui->btnMenu_Max, 0xf067);
    IconHelper::setIcon(ui->btnMenu_Close, 0xf00d);

    //ui->widgetMenu->setVisible(false);
    ui->widgetTitle->installEventFilter(this);
    ui->widgetTitle->setProperty("form", "title");
    ui->widgetTop->setProperty("nav", "top");

    QFont font;
    font.setPixelSize(25);
    ui->labTitle->setFont(font);
    ui->labTitle->setText("智能访客管理平台");
    this->setWindowTitle(ui->labTitle->text());

    ui->stackedWidget->setStyleSheet("QLabel{font:60px;}");

    //单独设置指示器大小
    int addWidth = 20;
    int addHeight = 10;
    int rbtnWidth = 15;
    int ckWidth = 13;
    int scrWidth = 12;
    int borderWidth = 3;

    QStringList qss;
    qss << QString("QComboBox::drop-down,QDateEdit::drop-down,QTimeEdit::drop-down,QDateTimeEdit::drop-down{width:%1px;}").arg(addWidth);
    qss << QString("QComboBox::down-arrow,QDateEdit[calendarPopup=\"true\"]::down-arrow,QTimeEdit[calendarPopup=\"true\"]::down-arrow,"
                   "QDateTimeEdit[calendarPopup=\"true\"]::down-arrow{width:%1px;height:%1px;right:2px;}").arg(addHeight);
    qss << QString("QRadioButton::indicator{width:%1px;height:%1px;}").arg(rbtnWidth);
    qss << QString("QCheckBox::indicator,QGroupBox::indicator,QTreeWidget::indicator,QListWidget::indicator{width:%1px;height:%1px;}").arg(ckWidth);
    qss << QString("QScrollBar:horizontal{min-height:%1px;border-radius:%2px;}QScrollBar::handle:horizontal{border-radius:%2px;}"
                   "QScrollBar:vertical{min-width:%1px;border-radius:%2px;}QScrollBar::handle:vertical{border-radius:%2px;}").arg(scrWidth).arg(scrWidth / 2);
    qss << QString("QWidget#widget_top>QToolButton:pressed,QWidget#widget_top>QToolButton:hover,"
                   "QWidget#widget_top>QToolButton:checked,QWidget#widget_top>QLabel:hover{"
                   "border-width:0px 0px %1px 0px;}").arg(borderWidth);
    qss << QString("QWidget#widgetleft>QPushButton:checked,QWidget#widgetleft>QToolButton:checked,"
                   "QWidget#widgetleft>QPushButton:pressed,QWidget#widgetleft>QToolButton:pressed{"
                   "border-width:0px 0px 0px %1px;}").arg(borderWidth);
    this->setStyleSheet(qss.join(""));

    QSize icoSize(32, 32);
    int icoWidth = 85;

    //设置顶部导航按钮
    QList<QToolButton *> tbtns = ui->widgetTop->findChildren<QToolButton *>();
    foreach (QToolButton *btn, tbtns) {
        btn->setIconSize(icoSize);
        btn->setMinimumWidth(icoWidth);
        btn->setCheckable(true);
        connect(btn, SIGNAL(clicked()), this, SLOT(buttonClick()));
    }

    ui->btnMain->click();
}

void frmMain::buttonClick()
{
    QToolButton *b = (QToolButton *)sender();
    QString name = b->text();

    QList<QToolButton *> tbtns = ui->widgetTop->findChildren<QToolButton *>();
    foreach (QToolButton *btn, tbtns) {
        btn->setChecked(btn == b);
    }

    if (name == "主界面") {
        ui->stackedWidget->setCurrentIndex(0);
    } else if (name == "系统设置") {
        ui->stackedWidget->setCurrentIndex(1);
    } else if (name == "警情查询") {
        ui->stackedWidget->setCurrentIndex(2);
    } else if (name == "调试帮助") {
        ui->stackedWidget->setCurrentIndex(3);
    } else if (name == "用户退出") {
        exit(0);
    }
}

void frmMain::on_btnMenu_Min_clicked()
{
    showMinimized();
}

void frmMain::on_btnMenu_Max_clicked()
{
    static bool max = false;
    static QRect location = this->geometry();

    if (max) {
        this->setGeometry(location);
    } else {
        location = this->geometry();
        this->setGeometry(QtHelper::getScreenRect());
    }

    this->setProperty("canMove", max);
    max = !max;
}

void frmMain::on_btnMenu_Close_clicked()
{
    close();
}
```

### `form/frmmain.h`

```cpp
﻿#ifndef FRMMAIN_H
#define FRMMAIN_H

#include <QDialog>

namespace Ui {
class frmMain;
}

class frmMain : public QDialog
{
    Q_OBJECT

public:
    explicit frmMain(QWidget *parent = 0);
    ~frmMain();

protected:
    bool eventFilter(QObject *watched, QEvent *event);

private:
    Ui::frmMain *ui;

private slots:
    void initForm();
    void buttonClick();

private slots:
    void on_btnMenu_Min_clicked();
    void on_btnMenu_Max_clicked();
    void on_btnMenu_Close_clicked();
};

#endif // UIDEMO01_H
```

### `form/frmmain.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmMain</class>
 <widget class="QDialog" name="frmMain">
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
    <number>0</number>
   </property>
   <property name="leftMargin">
    <number>1</number>
   </property>
   <property name="topMargin">
    <number>1</number>
   </property>
   <property name="rightMargin">
    <number>1</number>
   </property>
   <property name="bottomMargin">
    <number>1</number>
   </property>
   <item>
    <widget class="QWidget" name="widgetTitle" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
     <layout class="QHBoxLayout" name="horizontalLayout_2">
      <property name="spacing">
       <number>10</number>
      </property>
      <property name="leftMargin">
       <number>10</number>
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
       <widget class="QLabel" name="labIco">
        <property name="text">
         <string/>
        </property>
        <property name="alignment">
         <set>Qt::AlignCenter</set>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QLabel" name="labTitle">
        <property name="styleSheet">
         <string notr="true"/>
        </property>
        <property name="text">
         <string/>
        </property>
        <property name="alignment">
         <set>Qt::AlignLeading|Qt::AlignLeft|Qt::AlignVCenter</set>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QWidget" name="widgetTop" native="true">
        <layout class="QHBoxLayout" name="horizontalLayout_3">
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
          <widget class="QToolButton" name="btnMain">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="text">
            <string>主界面</string>
           </property>
           <property name="icon">
            <iconset resource="../other/main.qrc">
             <normaloff>:/image/main_main.png</normaloff>:/image/main_main.png</iconset>
           </property>
           <property name="toolButtonStyle">
            <enum>Qt::ToolButtonTextUnderIcon</enum>
           </property>
          </widget>
         </item>
         <item>
          <widget class="QToolButton" name="btnConfig">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="text">
            <string>系统设置</string>
           </property>
           <property name="icon">
            <iconset resource="../other/main.qrc">
             <normaloff>:/image/main_config.png</normaloff>:/image/main_config.png</iconset>
           </property>
           <property name="toolButtonStyle">
            <enum>Qt::ToolButtonTextUnderIcon</enum>
           </property>
          </widget>
         </item>
         <item>
          <widget class="QToolButton" name="btnData">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="text">
            <string>警情查询</string>
           </property>
           <property name="icon">
            <iconset resource="../other/main.qrc">
             <normaloff>:/image/main_data.png</normaloff>:/image/main_data.png</iconset>
           </property>
           <property name="checked">
            <bool>false</bool>
           </property>
           <property name="toolButtonStyle">
            <enum>Qt::ToolButtonTextUnderIcon</enum>
           </property>
          </widget>
         </item>
         <item>
          <widget class="QToolButton" name="btnHelp">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="styleSheet">
            <string notr="true"/>
           </property>
           <property name="text">
            <string>调试帮助</string>
           </property>
           <property name="icon">
            <iconset resource="../other/main.qrc">
             <normaloff>:/image/main_person.png</normaloff>:/image/main_person.png</iconset>
           </property>
           <property name="toolButtonStyle">
            <enum>Qt::ToolButtonTextUnderIcon</enum>
           </property>
          </widget>
         </item>
         <item>
          <widget class="QToolButton" name="btnExit">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="styleSheet">
            <string notr="true"/>
           </property>
           <property name="text">
            <string>用户退出</string>
           </property>
           <property name="icon">
            <iconset resource="../other/main.qrc">
             <normaloff>:/image/main_exit.png</normaloff>:/image/main_exit.png</iconset>
           </property>
           <property name="toolButtonStyle">
            <enum>Qt::ToolButtonTextUnderIcon</enum>
           </property>
          </widget>
         </item>
        </layout>
       </widget>
      </item>
      <item>
       <spacer name="horizontalSpacer">
        <property name="orientation">
         <enum>Qt::Horizontal</enum>
        </property>
        <property name="sizeHint" stdset="0">
         <size>
          <width>40</width>
          <height>20</height>
         </size>
        </property>
       </spacer>
      </item>
      <item>
       <widget class="QWidget" name="widgetMenu" native="true">
        <layout class="QGridLayout" name="gridLayout_2">
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
         <property name="spacing">
          <number>0</number>
         </property>
         <item row="1" column="1" colspan="3">
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
         <item row="0" column="1">
          <widget class="QPushButton" name="btnMenu_Min">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="minimumSize">
            <size>
             <width>30</width>
             <height>30</height>
            </size>
           </property>
           <property name="cursor">
            <cursorShape>ArrowCursor</cursorShape>
           </property>
           <property name="focusPolicy">
            <enum>Qt::NoFocus</enum>
           </property>
           <property name="toolTip">
            <string>最小化</string>
           </property>
           <property name="text">
            <string/>
           </property>
          </widget>
         </item>
         <item row="0" column="3">
          <widget class="QPushButton" name="btnMenu_Close">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="minimumSize">
            <size>
             <width>30</width>
             <height>30</height>
            </size>
           </property>
           <property name="cursor">
            <cursorShape>ArrowCursor</cursorShape>
           </property>
           <property name="focusPolicy">
            <enum>Qt::NoFocus</enum>
           </property>
           <property name="toolTip">
            <string>关闭</string>
           </property>
           <property name="text">
            <string/>
           </property>
          </widget>
         </item>
         <item row="0" column="2">
          <widget class="QPushButton" name="btnMenu_Max">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="minimumSize">
            <size>
             <width>30</width>
             <height>30</height>
            </size>
           </property>
           <property name="focusPolicy">
            <enum>Qt::NoFocus</enum>
           </property>
           <property name="text">
            <string/>
           </property>
          </widget>
         </item>
        </layout>
       </widget>
      </item>
     </layout>
    </widget>
   </item>
   <item>
    <widget class="QStackedWidget" name="stackedWidget">
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="currentIndex">
      <number>0</number>
     </property>
     <widget class="QWidget" name="page1">
      <layout class="QVBoxLayout" name="verticalLayout_2">
       <item>
        <widget class="QLabel" name="lab1">
         <property name="styleSheet">
          <string notr="true"/>
         </property>
         <property name="text">
          <string>主界面</string>
         </property>
         <property name="alignment">
          <set>Qt::AlignCenter</set>
         </property>
        </widget>
       </item>
      </layout>
     </widget>
     <widget class="QWidget" name="page2">
      <layout class="QVBoxLayout" name="verticalLayout_3">
       <item>
        <widget class="QLabel" name="lab2">
         <property name="text">
          <string>系统设置</string>
         </property>
         <property name="alignment">
          <set>Qt::AlignCenter</set>
         </property>
        </widget>
       </item>
      </layout>
     </widget>
     <widget class="QWidget" name="page3">
      <layout class="QVBoxLayout" name="verticalLayout_5">
       <item>
        <widget class="QLabel" name="lab3">
         <property name="text">
          <string>警情查询</string>
         </property>
         <property name="alignment">
          <set>Qt::AlignCenter</set>
         </property>
        </widget>
       </item>
      </layout>
     </widget>
     <widget class="QWidget" name="page4">
      <layout class="QVBoxLayout" name="verticalLayout_4">
       <item>
        <widget class="QLabel" name="lab4">
         <property name="text">
          <string>调试帮助</string>
         </property>
         <property name="alignment">
          <set>Qt::AlignCenter</set>
         </property>
        </widget>
       </item>
      </layout>
     </widget>
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
