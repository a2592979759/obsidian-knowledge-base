---
tags:
  - Qt
  - 界面美化
  - 高质量
---
# 界面美化入门示例 (`uidemo08`)

> 界面美化入门示例

## 效果图

![[QWidgetDemo_assert/uidemo08.jpg]]

## 分类

- 类型: 界面美化 `ui`
- 源码目录: `ui/uidemo08`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QString QToolButton QWidget QAbstractButton QLabel QList QEvent QVBoxLayout QHBoxLayout QPushButton QObject QApplication QFont QSize QPalette QColor QRect QSS QGridLayout QStackedWidget

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
#include "qthelper.h"

frmMain::frmMain(QWidget *parent) : QWidget(parent), ui(new Ui::frmMain)
{
    ui->setupUi(this);
    this->initForm();
    this->initStyle();
    this->initLeftMain();
    this->initLeftConfig();
}

frmMain::~frmMain()
{
    delete ui;
}

bool frmMain::eventFilter(QObject *watched, QEvent *event)
{
    if (watched == ui->widgetTitle) {
        if (event->type() == QEvent::MouseButtonDblClick) {
            on_btnMenu_Max_clicked();
        }
    }
    return QWidget::eventFilter(watched, event);
}

void frmMain::getQssColor(const QString &qss, const QString &flag, QString &color)
{
    int index = qss.indexOf(flag);
    if (index >= 0) {
        color = qss.mid(index + flag.length(), 7);
    }
    //qDebug() << TIMEMS << flag << color;
}

void frmMain::getQssColor(const QString &qss, QString &textColor, QString &panelColor,
                          QString &borderColor, QString &normalColorStart, QString &normalColorEnd,
                          QString &darkColorStart, QString &darkColorEnd, QString &highColor)
{
    getQssColor(qss, "TextColor:", textColor);
    getQssColor(qss, "PanelColor:", panelColor);
    getQssColor(qss, "BorderColor:", borderColor);
    getQssColor(qss, "NormalColorStart:", normalColorStart);
    getQssColor(qss, "NormalColorEnd:", normalColorEnd);
    getQssColor(qss, "DarkColorStart:", darkColorStart);
    getQssColor(qss, "DarkColorEnd:", darkColorEnd);
    getQssColor(qss, "HighColor:", highColor);
}

void frmMain::initForm()
{
    //设置无边框
    QtHelper::setFramelessForm(this);
    //设置图标
    IconHelper::setIcon(ui->labIco, 0xf073, 30);
    IconHelper::setIcon(ui->btnMenu_Min, 0xf068);
    IconHelper::setIcon(ui->btnMenu_Max, 0xf067);
    IconHelper::setIcon(ui->btnMenu_Close, 0xf00d);

    //ui->widgetMenu->setVisible(false);
    ui->widgetTitle->setProperty("form", "title");
    //关联事件过滤器用于双击放大
    ui->widgetTitle->installEventFilter(this);
    ui->widgetTop->setProperty("nav", "top");

    QFont font;
    font.setPixelSize(25);
    ui->labTitle->setFont(font);
    ui->labTitle->setText("智能访客管理平台");
    this->setWindowTitle(ui->labTitle->text());

    ui->stackedWidget->setStyleSheet("QLabel{font:60px;}");

    QSize icoSize(32, 32);
    int icoWidth = 85;

    //设置顶部导航按钮
    QList<QAbstractButton *> tbtns = ui->widgetTop->findChildren<QAbstractButton *>();
    foreach (QAbstractButton *btn, tbtns) {
        btn->setIconSize(icoSize);
        btn->setMinimumWidth(icoWidth);
        btn->setCheckable(true);
        connect(btn, SIGNAL(clicked()), this, SLOT(buttonClick()));
    }

    ui->btnMain->click();

    ui->widgetLeftMain->setProperty("flag", "left");
    ui->widgetLeftConfig->setProperty("flag", "left");
    ui->page1->setStyleSheet(QString("QWidget[flag=\"left\"] QAbstractButton{min-height:%1px;max-height:%1px;}").arg(60));
    ui->page2->setStyleSheet(QString("QWidget[flag=\"left\"] QAbstractButton{min-height:%1px;max-height:%1px;}").arg(25));
}

void frmMain::initStyle()
{
    //加载样式表
    QString qss = QtHelper::getStyle(":/qss/blacksoft.css");
    if (!qss.isEmpty()) {
        QString paletteColor = qss.mid(20, 7);
        qApp->setPalette(QPalette(QColor(paletteColor)));
        qApp->setStyleSheet(qss);
    }

    //先从样式表中取出对应的颜色
    QString textColor, panelColor, borderColor, normalColorStart, normalColorEnd, darkColorStart, darkColorEnd, highColor;
    getQssColor(qss, textColor, panelColor, borderColor, normalColorStart, normalColorEnd, darkColorStart, darkColorEnd, highColor);

    //将对应颜色设置到控件
    this->borderColor = highColor;
    this->normalBgColor = normalColorStart;
    this->darkBgColor = panelColor;
    this->normalTextColor = textColor;
    this->darkTextColor = normalTextColor;
}

void frmMain::buttonClick()
{
    QAbstractButton *b = (QAbstractButton *)sender();
    QString name = b->text();

    QList<QAbstractButton *> tbtns = ui->widgetTop->findChildren<QAbstractButton *>();
    foreach (QAbstractButton *btn, tbtns) {
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

void frmMain::initLeftMain()
{
    iconsMain << 0xf030 << 0xf03e << 0xf247;
    btnsMain << ui->tbtnMain1 << ui->tbtnMain2 << ui->tbtnMain3;

    for (int i = 0; i < btnsMain.count(); ++i) {
        QToolButton *btn = (QToolButton *)btnsMain.at(i);
        btn->setCheckable(true);
        btn->setToolButtonStyle(Qt::ToolButtonTextUnderIcon);
        connect(btn, SIGNAL(clicked(bool)), this, SLOT(leftMainClick()));
    }

    IconHelper::StyleColor styleColor;
    styleColor.position = "left";
    styleColor.iconSize = 18;
    styleColor.iconWidth = 35;
    styleColor.iconHeight = 25;
    styleColor.borderWidth = 4;
    styleColor.borderColor = borderColor;
    styleColor.setColor(normalBgColor, normalTextColor, darkBgColor, darkTextColor);
    IconHelper::setStyle(ui->widgetLeftMain, btnsMain, iconsMain, styleColor);
    ui->tbtnMain1->click();
}

void frmMain::initLeftConfig()
{
    iconsConfig << 0xf031 << 0xf036 << 0xf249 << 0xf055 << 0xf05a << 0xf249;
    btnsConfig << ui->tbtnConfig1 << ui->tbtnConfig2 << ui->tbtnConfig3 << ui->tbtnConfig4 << ui->tbtnConfig5 << ui->tbtnConfig6;

    for (int i = 0; i < btnsConfig.count(); ++i) {
        QToolButton *btn = (QToolButton *)btnsConfig.at(i);
        btn->setCheckable(true);
        btn->setToolButtonStyle(Qt::ToolButtonTextBesideIcon);
        connect(btn, SIGNAL(clicked(bool)), this, SLOT(leftConfigClick()));
    }

    IconHelper::StyleColor styleColor;
    styleColor.position = "left";
    styleColor.iconSize = 16;
    styleColor.iconWidth = 20;
    styleColor.iconHeight = 20;
    styleColor.borderWidth = 3;
    styleColor.borderColor = borderColor;
    styleColor.setColor(normalBgColor, normalTextColor, darkBgColor, darkTextColor);
    IconHelper::setStyle(ui->widgetLeftConfig, btnsConfig, iconsConfig, styleColor);
    ui->tbtnConfig1->click();
}

void frmMain::leftMainClick()
{
    QAbstractButton *b = (QAbstractButton *)sender();
    QString name = b->text();
    for (int i = 0; i < btnsMain.count(); ++i) {
        QAbstractButton *btn = btnsMain.at(i);
        btn->setChecked(btn == b);
    }

    ui->lab1->setText(name);
}

void frmMain::leftConfigClick()
{
    QToolButton *b = (QToolButton *)sender();
    QString name = b->text();
    for (int i = 0; i < btnsConfig.count(); ++i) {
        QAbstractButton *btn = btnsConfig.at(i);
        btn->setChecked(btn == b);
    }

    ui->lab2->setText(name);
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

#include <QWidget>

class QAbstractButton;

namespace Ui {
class frmMain;
}

class frmMain : public QWidget
{
    Q_OBJECT

public:
    explicit frmMain(QWidget *parent = 0);
    ~frmMain();

protected:
    bool eventFilter(QObject *watched, QEvent *event);

private:
    Ui::frmMain *ui;

    QList<int> iconsMain;
    QList<QAbstractButton *> btnsMain;

    QList<int> iconsConfig;
    QList<QAbstractButton *> btnsConfig;

private:
    //根据QSS样式获取对应颜色值
    QString borderColor;
    QString normalBgColor;
    QString darkBgColor;
    QString normalTextColor;
    QString darkTextColor;

    void getQssColor(const QString &qss, const QString &flag, QString &color);
    void getQssColor(const QString &qss, QString &textColor,
                     QString &panelColor, QString &borderColor,
                     QString &normalColorStart, QString &normalColorEnd,
                     QString &darkColorStart, QString &darkColorEnd,
                     QString &highColor);

private slots:
    void initForm();
    void initStyle();
    void buttonClick();
    void initLeftMain();
    void initLeftConfig();
    void leftMainClick();
    void leftConfigClick();

private slots:
    void on_btnMenu_Min_clicked();
    void on_btnMenu_Max_clicked();
    void on_btnMenu_Close_clicked();
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
    <width>810</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Dialog</string>
  </property>
  <property name="sizeGripEnabled" stdset="0">
   <bool>false</bool>
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
        <property name="minimumSize">
         <size>
          <width>100</width>
          <height>0</height>
         </size>
        </property>
        <property name="maximumSize">
         <size>
          <width>100</width>
          <height>16777215</height>
         </size>
        </property>
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
      <number>1</number>
     </property>
     <widget class="QWidget" name="page1">
      <layout class="QHBoxLayout" name="horizontalLayout">
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
        <widget class="QWidget" name="widgetLeftMain" native="true">
         <property name="maximumSize">
          <size>
           <width>90</width>
           <height>16777215</height>
          </size>
         </property>
         <layout class="QVBoxLayout" name="verticalLayout_2">
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
           <widget class="QToolButton" name="tbtnMain1">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>视频监控</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="tbtnMain2">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>地图监控</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="tbtnMain3">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>设备监控</string>
            </property>
           </widget>
          </item>
          <item>
           <spacer name="verticalSpacer_2">
            <property name="orientation">
             <enum>Qt::Vertical</enum>
            </property>
            <property name="sizeHint" stdset="0">
             <size>
              <width>20</width>
              <height>471</height>
             </size>
            </property>
           </spacer>
          </item>
         </layout>
        </widget>
       </item>
       <item>
        <widget class="QLabel" name="lab1">
         <property name="sizePolicy">
          <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
           <horstretch>0</horstretch>
           <verstretch>0</verstretch>
          </sizepolicy>
         </property>
         <property name="text">
          <string/>
         </property>
         <property name="alignment">
          <set>Qt::AlignCenter</set>
         </property>
        </widget>
       </item>
      </layout>
     </widget>
     <widget class="QWidget" name="page2">
      <layout class="QHBoxLayout" name="horizontalLayout_4">
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
        <widget class="QWidget" name="widgetLeftConfig" native="true">
         <property name="maximumSize">
          <size>
           <width>110</width>
           <height>16777215</height>
          </size>
         </property>
         <layout class="QVBoxLayout" name="verticalLayout_3">
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
           <widget class="QToolButton" name="tbtnConfig1">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>基本设置</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="tbtnConfig2">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>转发设置</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="tbtnConfig3">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>用户设置</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="tbtnConfig4">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>防区设置</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="tbtnConfig5">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>设备设置</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="tbtnConfig6">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>其他设置</string>
            </property>
           </widget>
          </item>
          <item>
           <spacer name="verticalSpacer_3">
            <property name="orientation">
             <enum>Qt::Vertical</enum>
            </property>
            <property name="sizeHint" stdset="0">
             <size>
              <width>20</width>
              <height>417</height>
             </size>
            </property>
           </spacer>
          </item>
         </layout>
        </widget>
       </item>
       <item>
        <widget class="QLabel" name="lab2">
         <property name="sizePolicy">
          <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
           <horstretch>0</horstretch>
           <verstretch>0</verstretch>
          </sizepolicy>
         </property>
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
