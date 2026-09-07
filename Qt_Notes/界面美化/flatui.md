---
tags:
  - Qt
  - 界面美化
---
# 扁平化风格 (`flatui`)

> 扁平化风格

## 效果图

![[QWidgetDemo_assert/flatui.jpg]]

## 分类

- 类型: 界面美化 `ui`
- 源码目录: `ui/flatui`

## 技术要点

**用到的 Qt 类**: QString QScrollBar QSlider QTextCodec QPushButton QRadioButton QTableWidgetItem QLineEdit QWidget QStringList QProgressBar QAbstractItemView QApplication QObject QCheckBox QDateTime QGridLayout QTableWidget QFont

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./flatui.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "flatui.h"
#include "qpushbutton.h"
#include "qlineedit.h"
#include "qprogressbar.h"
#include "qslider.h"
#include "qradiobutton.h"
#include "qcheckbox.h"
#include "qscrollbar.h"
#include "qdebug.h"

QString FlatUI::setPushButtonQss(QPushButton *btn, int radius, int padding,
                                 const QString &normalColor,
                                 const QString &normalTextColor,
                                 const QString &hoverColor,
                                 const QString &hoverTextColor,
                                 const QString &pressedColor,
                                 const QString &pressedTextColor)
{
    QStringList list;
    list.append(QString("QPushButton{border-style:none;padding:%1px;border-radius:%2px;color:%3;background:%4;}")
                .arg(padding).arg(radius).arg(normalTextColor).arg(normalColor));
    list.append(QString("QPushButton:hover{color:%1;background:%2;}")
                .arg(hoverTextColor).arg(hoverColor));
    list.append(QString("QPushButton:pressed{color:%1;background:%2;}")
                .arg(pressedTextColor).arg(pressedColor));

    QString qss = list.join("");
    btn->setStyleSheet(qss);
    return qss;
}

QString FlatUI::setLineEditQss(QLineEdit *txt, int radius, int borderWidth,
                               const QString &normalColor,
                               const QString &focusColor)
{
    QStringList list;
    list.append(QString("QLineEdit{border-style:none;padding:3px;border-radius:%1px;border:%2px solid %3;}")
                .arg(radius).arg(borderWidth).arg(normalColor));
    list.append(QString("QLineEdit:focus{border:%1px solid %2;}")
                .arg(borderWidth).arg(focusColor));

    QString qss = list.join("");
    txt->setStyleSheet(qss);
    return qss;
}

QString FlatUI::setProgressQss(QProgressBar *bar, int barHeight,
                               int barRadius, int fontSize,
                               const QString &normalColor,
                               const QString &chunkColor)
{

    QStringList list;
    list.append(QString("QProgressBar{font:%1px;background:%2;max-height:%3px;border-radius:%4px;text-align:center;border:1px solid %2;}")
                .arg(fontSize).arg(normalColor).arg(barHeight).arg(barRadius));
    list.append(QString("QProgressBar:chunk{border-radius:%2px;background-color:%1;}")
                .arg(chunkColor).arg(barRadius));

    QString qss = list.join("");
    bar->setStyleSheet(qss);
    return qss;
}

QString FlatUI::setSliderQss(QSlider *slider, int sliderHeight,
                             const QString &normalColor,
                             const QString &grooveColor,
                             const QString &handleBorderColor,
                             const QString &handleColor)
{
    int sliderRadius = sliderHeight / 2;
    int handleWidth = (sliderHeight * 3) / 2 + (sliderHeight / 5);
    int handleRadius = handleWidth / 2;
    int handleOffset = handleRadius / 2;

    QStringList list;
    list.append(QString("QSlider::horizontal{min-height:%1px;}").arg(sliderHeight * 2));
    list.append(QString("QSlider::groove:horizontal{background:%1;height:%2px;border-radius:%3px;}")
                .arg(normalColor).arg(sliderHeight).arg(sliderRadius));
    list.append(QString("QSlider::add-page:horizontal{background:%1;height:%2px;border-radius:%3px;}")
                .arg(normalColor).arg(sliderHeight).arg(sliderRadius));
    list.append(QString("QSlider::sub-page:horizontal{background:%1;height:%2px;border-radius:%3px;}")
                .arg(grooveColor).arg(sliderHeight).arg(sliderRadius));
    list.append(QString("QSlider::handle:horizontal{width:%3px;margin-top:-%4px;margin-bottom:-%4px;border-radius:%5px;"
                        "background:qradialgradient(spread:pad,cx:0.5,cy:0.5,radius:0.5,fx:0.5,fy:0.5,stop:0.6 %1,stop:0.8 %2);}")
                .arg(handleColor).arg(handleBorderColor).arg(handleWidth).arg(handleOffset).arg(handleRadius));

    //偏移一个像素
    handleWidth = handleWidth + 1;
    list.append(QString("QSlider::vertical{min-width:%1px;}").arg(sliderHeight * 2));
    list.append(QString("QSlider::groove:vertical{background:%1;width:%2px;border-radius:%3px;}")
                .arg(normalColor).arg(sliderHeight).arg(sliderRadius));
    list.append(QString("QSlider::add-page:vertical{background:%1;width:%2px;border-radius:%3px;}")
                .arg(grooveColor).arg(sliderHeight).arg(sliderRadius));
    list.append(QString("QSlider::sub-page:vertical{background:%1;width:%2px;border-radius:%3px;}")
                .arg(normalColor).arg(sliderHeight).arg(sliderRadius));
    list.append(QString("QSlider::handle:vertical{height:%3px;margin-left:-%4px;margin-right:-%4px;border-radius:%5px;"
                        "background:qradialgradient(spread:pad,cx:0.5,cy:0.5,radius:0.5,fx:0.5,fy:0.5,stop:0.6 %1,stop:0.8 %2);}")
                .arg(handleColor).arg(handleBorderColor).arg(handleWidth).arg(handleOffset).arg(handleRadius));

    QString qss = list.join("");
    slider->setStyleSheet(qss);
    return qss;
}

QString FlatUI::setRadioButtonQss(QRadioButton *rbtn, int indicatorRadius,
                                  const QString &normalColor,
                                  const QString &checkColor)
{
    int indicatorWidth = indicatorRadius * 2;

    QStringList list;
    list.append(QString("QRadioButton::indicator{border-radius:%1px;width:%2px;height:%2px;}")
                .arg(indicatorRadius).arg(indicatorWidth));
    list.append(QString("QRadioButton::indicator::unchecked{background:qradialgradient(spread:pad,cx:0.5,cy:0.5,radius:0.5,fx:0.5,fy:0.5,"
                        "stop:0.6 #FFFFFF,stop:0.7 %1);}").arg(normalColor));
    list.append(QString("QRadioButton::indicator::checked{background:qradialgradient(spread:pad,cx:0.5,cy:0.5,radius:0.5,fx:0.5,fy:0.5,"
                        "stop:0 %1,stop:0.3 %1,stop:0.4 #FFFFFF,stop:0.6 #FFFFFF,stop:0.7 %1);}").arg(checkColor));

    QString qss = list.join("");
    rbtn->setStyleSheet(qss);
    return qss;
}

QString FlatUI::setScrollBarQss(QWidget *scroll, int radius, int min, int max,
                                const QString &bgColor,
                                const QString &handleNormalColor,
                                const QString &handleHoverColor,
                                const QString &handlePressedColor)
{
    //滚动条离背景间隔
    int padding = 0;

    QStringList list;

    //handle:指示器,滚动条拉动部分 add-page:滚动条拉动时增加的部分 sub-page:滚动条拉动时减少的部分 add-line:递增按钮 sub-line:递减按钮

    //横向滚动条部分
    list.append(QString("QScrollBar:horizontal{background:%1;padding:%2px;border-radius:%3px;min-height:%4px;max-height:%4px;}")
                .arg(bgColor).arg(padding).arg(radius).arg(max));
    list.append(QString("QScrollBar::handle:horizontal{background:%1;min-width:%2px;border-radius:%3px;}")
                .arg(handleNormalColor).arg(min).arg(radius));
    list.append(QString("QScrollBar::handle:horizontal:hover{background:%1;}")
                .arg(handleHoverColor));
    list.append(QString("QScrollBar::handle:horizontal:pressed{background:%1;}")
                .arg(handlePressedColor));
    list.append(QString("QScrollBar::add-page:horizontal{background:none;}"));
    list.append(QString("QScrollBar::sub-page:horizontal{background:none;}"));
    list.append(QString("QScrollBar::add-line:horizontal{background:none;}"));
    list.append(QString("QScrollBar::sub-line:horizontal{background:none;}"));

    //纵向滚动条部分
    list.append(QString("QScrollBar:vertical{background:%1;padding:%2px;border-radius:%3px;min-width:%4px;max-width:%4px;}")
                .arg(bgColor).arg(padding).arg(radius).arg(max));
    list.append(QString("QScrollBar::handle:vertical{background:%1;min-height:%2px;border-radius:%3px;}")
                .arg(handleNormalColor).arg(min).arg(radius));
    list.append(QString("QScrollBar::handle:vertical:hover{background:%1;}")
                .arg(handleHoverColor));
    list.append(QString("QScrollBar::handle:vertical:pressed{background:%1;}")
                .arg(handlePressedColor));
    list.append(QString("QScrollBar::add-page:vertical{background:none;}"));
    list.append(QString("QScrollBar::sub-page:vertical{background:none;}"));
    list.append(QString("QScrollBar::add-line:vertical{background:none;}"));
    list.append(QString("QScrollBar::sub-line:vertical{background:none;}"));

    QString qss = list.join("");
    scroll->setStyleSheet(qss);
    return qss;
}
```

### `./flatui.h`

```cpp
﻿#ifndef FLATUI_H
#define FLATUI_H

/**
 * FlatUI辅助类 作者:feiyangqingyun(QQ:517216493) 2016-12-16
 * 1. 按钮样式设置。
 * 2. 文本框样式设置。
 * 3. 进度条样式。
 * 4. 滑块条样式。
 * 5. 单选框样式。
 * 6. 滚动条样式。
 * 7. 可自由设置对象的高度宽度大小等。
 * 8. 自带默认参数值。
 */

#include <QObject>

class QPushButton;
class QLineEdit;
class QProgressBar;
class QSlider;
class QRadioButton;
class QCheckBox;
class QScrollBar;

#ifdef quc
class Q_DECL_EXPORT FlatUI
#else
class FlatUI
#endif

{
public:
    //设置按钮样式
    static QString setPushButtonQss(QPushButton *btn,                               //按钮对象
                                    int radius = 5,                                 //圆角半径
                                    int padding = 8,                                //间距
                                    const QString &normalColor = "#34495E",         //正常颜色
                                    const QString &normalTextColor = "#FFFFFF",     //文字颜色
                                    const QString &hoverColor = "#4E6D8C",          //悬停颜色
                                    const QString &hoverTextColor = "#F0F0F0",      //悬停文字颜色
                                    const QString &pressedColor = "#2D3E50",        //按下颜色
                                    const QString &pressedTextColor = "#B8C6D1");   //按下文字颜色

    //设置文本框样式
    static QString setLineEditQss(QLineEdit *txt,                                   //文本框对象
                                  int radius = 3,                                   //圆角半径
                                  int borderWidth = 2,                              //边框大小
                                  const QString &normalColor = "#DCE4EC",           //正常颜色
                                  const QString &focusColor = "#34495E");           //选中颜色

    //设置进度条样式
    static QString setProgressQss(QProgressBar *bar,
                                  int barHeight = 8,                                //进度条高度
                                  int barRadius = 5,                                //进度条半径
                                  int fontSize = 12,                                //文字字号
                                  const QString &normalColor = "#E8EDF2",           //正常颜色
                                  const QString &chunkColor = "#E74C3C");           //进度颜色

    //设置滑块条样式
    static QString setSliderQss(QSlider *slider,                                    //滑动条对象
                                int sliderHeight = 8,                               //滑动条高度
                                const QString &normalColor = "#E8EDF2",             //正常颜色
                                const QString &grooveColor = "#1ABC9C",             //滑块颜色
                                const QString &handleBorderColor = "#1ABC9C",       //指示器边框颜色
                                const QString &handleColor = "#FFFFFF");            //指示器颜色

    //设置单选框样式
    static QString setRadioButtonQss(QRadioButton *rbtn,                            //单选框对象
                                     int indicatorRadius = 8,                       //指示器圆角角度
                                     const QString &normalColor = "#D7DBDE",        //正常颜色
                                     const QString &checkColor = "#34495E");        //选中颜色

    //设置滚动条样式
    static QString setScrollBarQss(QWidget *scroll,                                 //滚动条对象
                                   int radius = 6,                                  //圆角角度
                                   int min = 120,                                   //指示器最小长度
                                   int max = 12,                                    //滚动条最大长度
                                   const QString &bgColor = "#606060",              //背景色
                                   const QString &handleNormalColor = "#34495E",    //指示器正常颜色
                                   const QString &handleHoverColor = "#1ABC9C",     //指示器悬停颜色
                                   const QString &handlePressedColor = "#E74C3C");  //指示器按下颜色
};

#endif // FLATUI_H
```

### `./frmflatui.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmflatui.h"
#include "ui_frmflatui.h"
#include "flatui.h"
#include "qdebug.h"
#include "qdatetime.h"

frmFlatUI::frmFlatUI(QWidget *parent) : QWidget(parent), ui(new Ui::frmFlatUI)
{
    ui->setupUi(this);
    this->initForm();
}

frmFlatUI::~frmFlatUI()
{
    delete ui;
}

void frmFlatUI::initForm()
{
    ui->bar1->setRange(0, 100);
    ui->bar2->setRange(0, 100);
    ui->slider1->setRange(0, 100);
    ui->slider2->setRange(0, 100);

    connect(ui->slider1, SIGNAL(valueChanged(int)), ui->bar1, SLOT(setValue(int)));
    connect(ui->slider2, SIGNAL(valueChanged(int)), ui->bar2, SLOT(setValue(int)));
    ui->slider1->setValue(30);
    ui->slider2->setValue(30);

    this->setStyleSheet("*{outline:0px;}QWidget#frmFlatUI{background:#FFFFFF;}");

    FlatUI::setPushButtonQss(ui->btn1);
    FlatUI::setPushButtonQss(ui->btn2, 5, 8, "#1ABC9C", "#E6F8F5", "#2EE1C1", "#FFFFFF", "#16A086", "#A7EEE6");
    FlatUI::setPushButtonQss(ui->btn3, 5, 8, "#3498DB", "#FFFFFF", "#5DACE4", "#E5FEFF", "#2483C7", "#A0DAFB");
    FlatUI::setPushButtonQss(ui->btn4, 5, 8, "#E74C3C", "#FFFFFF", "#EC7064", "#FFF5E7", "#DC2D1A", "#F5A996");

    FlatUI::setLineEditQss(ui->txt1);
    FlatUI::setLineEditQss(ui->txt2, 5, 2, "#DCE4EC", "#1ABC9C");
    FlatUI::setLineEditQss(ui->txt3, 3, 1, "#DCE4EC", "#3498DB");
    FlatUI::setLineEditQss(ui->txt4, 3, 1, "#DCE4EC", "#E74C3C");

    FlatUI::setProgressQss(ui->bar1);
    FlatUI::setProgressQss(ui->bar2, 8, 5, 9, "#E8EDF2", "#1ABC9C");

    FlatUI::setSliderQss(ui->slider1);
    FlatUI::setSliderQss(ui->slider2, 10, "#E8EDF2", "#E74C3C", "#E74C3C");
    FlatUI::setSliderQss(ui->slider3, 10, "#E8EDF2", "#34495E", "#34495E");

    FlatUI::setRadioButtonQss(ui->rbtn1);
    FlatUI::setRadioButtonQss(ui->rbtn2, 8, "#D7DBDE", "#1ABC9C");
    FlatUI::setRadioButtonQss(ui->rbtn3, 8, "#D7DBDE", "#3498DB");
    FlatUI::setRadioButtonQss(ui->rbtn4, 8, "#D7DBDE", "#E74C3C");

    FlatUI::setScrollBarQss(ui->horizontalScrollBar);
    FlatUI::setScrollBarQss(ui->verticalScrollBar, 8, 120, 20, "#606060", "#34495E", "#1ABC9C", "#E74C3C");

    //设置列数和列宽
    int width = 1920;
    ui->tableWidget->setColumnCount(5);
    ui->tableWidget->setColumnWidth(0, width * 0.06);
    ui->tableWidget->setColumnWidth(1, width * 0.10);
    ui->tableWidget->setColumnWidth(2, width * 0.06);
    ui->tableWidget->setColumnWidth(3, width * 0.10);
    ui->tableWidget->setColumnWidth(4, width * 0.20);
    ui->tableWidget->verticalHeader()->setDefaultSectionSize(25);

    QStringList headText;
    headText << "设备编号" << "设备名称" << "设备地址" << "告警内容" << "告警时间";
    ui->tableWidget->setHorizontalHeaderLabels(headText);
    ui->tableWidget->setSelectionBehavior(QAbstractItemView::SelectRows);
    ui->tableWidget->setEditTriggers(QAbstractItemView::NoEditTriggers);
    ui->tableWidget->setSelectionMode(QAbstractItemView::SingleSelection);
    ui->tableWidget->setAlternatingRowColors(true);
    ui->tableWidget->verticalHeader()->setVisible(false);
    ui->tableWidget->horizontalHeader()->setStretchLastSection(true);

    //设置行高
    ui->tableWidget->setRowCount(300);

    for (int i = 0; i < 300; i++) {
        ui->tableWidget->setRowHeight(i, 24);
        QTableWidgetItem *itemDeviceID = new QTableWidgetItem(QString::number(i + 1));
        QTableWidgetItem *itemDeviceName = new QTableWidgetItem(QString("测试设备%1").arg(i + 1));
        QTableWidgetItem *itemDeviceAddr = new QTableWidgetItem(QString::number(i + 1));
        QTableWidgetItem *itemContent = new QTableWidgetItem("防区告警");
        QTableWidgetItem *itemTime = new QTableWidgetItem(QDateTime::currentDateTime().toString("yyyy-MM-dd hh:mm:ss"));

        ui->tableWidget->setItem(i, 0, itemDeviceID);
        ui->tableWidget->setItem(i, 1, itemDeviceName);
        ui->tableWidget->setItem(i, 2, itemDeviceAddr);
        ui->tableWidget->setItem(i, 3, itemContent);
        ui->tableWidget->setItem(i, 4, itemTime);
    }
}
```

### `./frmflatui.h`

```cpp
﻿#ifndef FRMFLATUI_H
#define FRMFLATUI_H

#include <QWidget>

namespace Ui {
class frmFlatUI;
}

class frmFlatUI : public QWidget
{
    Q_OBJECT

public:
    explicit frmFlatUI(QWidget *parent = 0);
    ~frmFlatUI();

private:
    Ui::frmFlatUI *ui;

private slots:
    void initForm();
};

#endif // FRMFLATUI_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmflatui.h"
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

    frmFlatUI w;
    w.setWindowTitle("FlatUI控件集合 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmflatui.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmFlatUI</class>
 <widget class="QWidget" name="frmFlatUI">
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
  <property name="styleSheet">
   <string notr="true"/>
  </property>
  <layout class="QGridLayout" name="gridLayout">
   <item row="7" column="0">
    <widget class="QRadioButton" name="rbtn1">
     <property name="text">
      <string>语文</string>
     </property>
     <property name="checked">
      <bool>true</bool>
     </property>
    </widget>
   </item>
   <item row="0" column="2">
    <widget class="QPushButton" name="btn3">
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="text">
      <string>测试按钮</string>
     </property>
    </widget>
   </item>
   <item row="1" column="2">
    <widget class="QLineEdit" name="txt3"/>
   </item>
   <item row="0" column="1">
    <widget class="QPushButton" name="btn2">
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="text">
      <string>测试按钮</string>
     </property>
    </widget>
   </item>
   <item row="0" column="4" rowspan="9">
    <widget class="QSlider" name="slider3">
     <property name="orientation">
      <enum>Qt::Vertical</enum>
     </property>
     <property name="invertedControls">
      <bool>false</bool>
     </property>
     <property name="tickPosition">
      <enum>QSlider::NoTicks</enum>
     </property>
    </widget>
   </item>
   <item row="7" column="1">
    <widget class="QRadioButton" name="rbtn2">
     <property name="text">
      <string>英语</string>
     </property>
    </widget>
   </item>
   <item row="2" column="2" colspan="2">
    <widget class="QProgressBar" name="bar2">
     <property name="minimumSize">
      <size>
       <width>0</width>
       <height>16</height>
      </size>
     </property>
     <property name="value">
      <number>24</number>
     </property>
    </widget>
   </item>
   <item row="5" column="0" colspan="4">
    <widget class="QScrollBar" name="horizontalScrollBar">
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="orientation">
      <enum>Qt::Horizontal</enum>
     </property>
    </widget>
   </item>
   <item row="7" column="3">
    <widget class="QRadioButton" name="rbtn4">
     <property name="text">
      <string>历史</string>
     </property>
    </widget>
   </item>
   <item row="7" column="2">
    <widget class="QRadioButton" name="rbtn3">
     <property name="text">
      <string>数学</string>
     </property>
    </widget>
   </item>
   <item row="0" column="3">
    <widget class="QPushButton" name="btn4">
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="text">
      <string>测试按钮</string>
     </property>
    </widget>
   </item>
   <item row="1" column="1">
    <widget class="QLineEdit" name="txt2"/>
   </item>
   <item row="1" column="3">
    <widget class="QLineEdit" name="txt4"/>
   </item>
   <item row="0" column="0">
    <widget class="QPushButton" name="btn1">
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="text">
      <string>测试按钮</string>
     </property>
    </widget>
   </item>
   <item row="1" column="0">
    <widget class="QLineEdit" name="txt1"/>
   </item>
   <item row="2" column="0" colspan="2">
    <widget class="QProgressBar" name="bar1">
     <property name="minimumSize">
      <size>
       <width>0</width>
       <height>16</height>
      </size>
     </property>
     <property name="value">
      <number>24</number>
     </property>
    </widget>
   </item>
   <item row="0" column="5" rowspan="9">
    <widget class="QScrollBar" name="verticalScrollBar">
     <property name="orientation">
      <enum>Qt::Vertical</enum>
     </property>
    </widget>
   </item>
   <item row="3" column="0" colspan="2">
    <widget class="QSlider" name="slider1">
     <property name="minimumSize">
      <size>
       <width>0</width>
       <height>20</height>
      </size>
     </property>
     <property name="orientation">
      <enum>Qt::Horizontal</enum>
     </property>
    </widget>
   </item>
   <item row="3" column="2" colspan="2">
    <widget class="QSlider" name="slider2">
     <property name="minimumSize">
      <size>
       <width>0</width>
       <height>20</height>
      </size>
     </property>
     <property name="maximum">
      <number>255</number>
     </property>
     <property name="orientation">
      <enum>Qt::Horizontal</enum>
     </property>
    </widget>
   </item>
   <item row="9" column="0" colspan="6">
    <widget class="QTableWidget" name="tableWidget">
     <property name="styleSheet">
      <string notr="true"/>
     </property>
     <property name="gridStyle">
      <enum>Qt::DashDotLine</enum>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <resources/>
 <connections/>
</ui>
```
