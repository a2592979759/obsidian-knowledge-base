---
tags:
  - Qt
  - 工具相关
---
# 存款利息计算器 (`moneytool`)

> 存款利息计算器

## 效果图

![[QWidgetDemo_assert/moneytool.jpg]]

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/moneytool`

## 技术要点

**用到的 Qt 类**: QLabel QLineEdit QTextCodec QWidget QString QDateTime QGridLayout QApplication QGroupBox QComboBox QPushButton QRadioButton QDateEdit QFont QMessageBox

**特性/模块**: Q_OBJECT

## 完整源码

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "widget.h"
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

    Widget w;
    w.setWindowTitle("存款/贷款 计算器 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./widget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "widget.h"
#include "ui_widget.h"
#include "qmessagebox.h"
#include "qdebug.h"

Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget)
{
    ui->setupUi(this);
    this->initForm();
}

Widget::~Widget()
{
    delete ui;
}

void Widget::initForm()
{
    QDateTime now = QDateTime::currentDateTime();
    ui->dateStart->setDate(now.date());
    ui->dateEnd->setDate(now.date().addYears(1));
}

void Widget::on_btnCalc_clicked()
{
    //当前多少钱
    int moneyCurrent = ui->txtMoneyCurrent->text().toInt();
    //利息
    float rate = ui->txtRate->text().toFloat();
    //定期期限
    int year = ui->cboxYear->currentText().left(1).toInt();
    //总年份 必须是定期期限的倍数
    int years = ui->txtYears->text().toInt();
    //最终多少钱
    int moneyAll = 0;

    if (years % year != 0) {
        ui->txtYears->setFocus();
        QMessageBox::critical(this, "错误", "总年份必须是期限的整数倍数!");
        return;
    }

    if (ui->cboxType->currentIndex() == 0) {
        //傻瓜场景 直接计算
        moneyAll = moneyCurrent + (moneyCurrent * rate * years);
    } else {
        //真实场景 复利计算
        int count = years / year;
        for (int i = 0; i < count; ++i) {
            moneyCurrent = moneyCurrent + (moneyCurrent * rate * year);
        }
        moneyAll = moneyCurrent;
    }

    //计算下来3年期定期存款30年总金额翻2番到最初本金3倍 100W本金3年期自动续期30年=321W
    QString value = QString::number(moneyAll);
    ui->txtMoneyAll->setText(value);

    //拷贝到其他地方
    if (ui->rbtn1->isChecked()) {
        ui->txtValue1->setText(value);
    } else {
        ui->txtValue2->setText(value);
    }

    //计算两种存款方式的差额 比如1年期存3年和3年期存3年
    QString value1 = ui->txtValue1->text().trimmed();
    QString value2 = ui->txtValue2->text().trimmed();
    if (!value1.isEmpty() && !value2.isEmpty()) {
        int value = qAbs(value1.toInt() - value2.toInt());
        ui->txtValue->setText(QString::number(value));
    }
}

void Widget::on_btnCalc2_clicked()
{
    //计算天数
    QDateTime dateStart = ui->dateStart->dateTime();
    QDateTime dateEnd = ui->dateEnd->dateTime();
    int day = dateStart.daysTo(dateEnd);
    int money = ui->txtMoney2->text().toInt();
    float rate = ui->txtRate2->text().toFloat();
    int result = money * rate * day;
    ui->txtResult2->setText(QString::number(result));
    qDebug() << day;
}
```

### `./widget.h`

```cpp
﻿#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>

namespace Ui {
class Widget;
}

class Widget : public QWidget
{
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();

private:
    Ui::Widget *ui;

private slots:
    void initForm();
    void on_btnCalc_clicked();
    void on_btnCalc2_clicked();
};

#endif // WIDGET_H
```

### `./widget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>Widget</class>
 <widget class="QWidget" name="Widget">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>存款计算器</string>
  </property>
  <layout class="QGridLayout" name="gridLayout_3">
   <item row="0" column="0">
    <widget class="QGroupBox" name="groupBox">
     <property name="title">
      <string>存款计算</string>
     </property>
     <layout class="QGridLayout" name="gridLayout_2">
      <item row="0" column="0">
       <widget class="QWidget" name="widget" native="true">
        <property name="maximumSize">
         <size>
          <width>600</width>
          <height>16777215</height>
         </size>
        </property>
        <layout class="QGridLayout" name="gridLayout">
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
         <item row="2" column="3">
          <widget class="QLineEdit" name="txtMoneyAll"/>
         </item>
         <item row="1" column="0">
          <widget class="QLabel" name="labYear">
           <property name="text">
            <string>期限</string>
           </property>
          </widget>
         </item>
         <item row="0" column="2">
          <widget class="QLabel" name="labRate">
           <property name="text">
            <string>利率</string>
           </property>
          </widget>
         </item>
         <item row="2" column="1">
          <widget class="QComboBox" name="cboxType">
           <property name="currentIndex">
            <number>1</number>
           </property>
           <item>
            <property name="text">
             <string>单利</string>
            </property>
           </item>
           <item>
            <property name="text">
             <string>复利</string>
            </property>
           </item>
          </widget>
         </item>
         <item row="2" column="6">
          <widget class="QLineEdit" name="txtValue"/>
         </item>
         <item row="0" column="1">
          <widget class="QLineEdit" name="txtMoneyCurrent">
           <property name="text">
            <string>1000000</string>
           </property>
          </widget>
         </item>
         <item row="0" column="4" rowspan="3">
          <widget class="QPushButton" name="btnCalc">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="text">
            <string>计算</string>
           </property>
          </widget>
         </item>
         <item row="0" column="6">
          <widget class="QLineEdit" name="txtValue1"/>
         </item>
         <item row="0" column="0">
          <widget class="QLabel" name="labMoneyCurrent">
           <property name="text">
            <string>本金</string>
           </property>
          </widget>
         </item>
         <item row="1" column="1">
          <widget class="QComboBox" name="cboxYear">
           <property name="currentIndex">
            <number>1</number>
           </property>
           <item>
            <property name="text">
             <string>1年</string>
            </property>
           </item>
           <item>
            <property name="text">
             <string>3年</string>
            </property>
           </item>
           <item>
            <property name="text">
             <string>5年</string>
            </property>
           </item>
          </widget>
         </item>
         <item row="1" column="2">
          <widget class="QLabel" name="labYears">
           <property name="text">
            <string>年限</string>
           </property>
          </widget>
         </item>
         <item row="2" column="0">
          <widget class="QLabel" name="labType">
           <property name="text">
            <string>方式</string>
           </property>
          </widget>
         </item>
         <item row="1" column="5">
          <widget class="QRadioButton" name="rbtn2">
           <property name="text">
            <string>总计2</string>
           </property>
          </widget>
         </item>
         <item row="1" column="6">
          <widget class="QLineEdit" name="txtValue2"/>
         </item>
         <item row="0" column="3">
          <widget class="QLineEdit" name="txtRate">
           <property name="text">
            <string>0.04125</string>
           </property>
          </widget>
         </item>
         <item row="0" column="5">
          <widget class="QRadioButton" name="rbtn1">
           <property name="text">
            <string>总计1</string>
           </property>
           <property name="checked">
            <bool>true</bool>
           </property>
          </widget>
         </item>
         <item row="2" column="2">
          <widget class="QLabel" name="labMoneyAll">
           <property name="text">
            <string>总计</string>
           </property>
          </widget>
         </item>
         <item row="2" column="5">
          <widget class="QLabel" name="labValue">
           <property name="text">
            <string>总计差额</string>
           </property>
           <property name="alignment">
            <set>Qt::AlignRight|Qt::AlignTrailing|Qt::AlignVCenter</set>
           </property>
          </widget>
         </item>
         <item row="1" column="3">
          <widget class="QLineEdit" name="txtYears">
           <property name="text">
            <string>30</string>
           </property>
          </widget>
         </item>
        </layout>
       </widget>
      </item>
     </layout>
    </widget>
   </item>
   <item row="0" column="1">
    <spacer name="horizontalSpacer">
     <property name="orientation">
      <enum>Qt::Horizontal</enum>
     </property>
     <property name="sizeHint" stdset="0">
      <size>
       <width>19</width>
       <height>20</height>
      </size>
     </property>
    </spacer>
   </item>
   <item row="1" column="0">
    <widget class="QGroupBox" name="groupBox_2">
     <property name="minimumSize">
      <size>
       <width>0</width>
       <height>200</height>
      </size>
     </property>
     <property name="title">
      <string>贷款计算</string>
     </property>
     <layout class="QGridLayout" name="gridLayout_4">
      <item row="1" column="2">
       <widget class="QLabel" name="labRate2">
        <property name="text">
         <string>贷款利率</string>
        </property>
       </widget>
      </item>
      <item row="0" column="2">
       <widget class="QLabel" name="labMoney2">
        <property name="text">
         <string>贷款金额</string>
        </property>
       </widget>
      </item>
      <item row="0" column="6" rowspan="2">
       <widget class="QPushButton" name="btnCalc2">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="text">
         <string>计算</string>
        </property>
       </widget>
      </item>
      <item row="0" column="5">
       <widget class="QLineEdit" name="txtRate3">
        <property name="text">
         <string>0.0003</string>
        </property>
       </widget>
      </item>
      <item row="1" column="0">
       <widget class="QLabel" name="labEnd">
        <property name="text">
         <string>到期日期</string>
        </property>
       </widget>
      </item>
      <item row="0" column="4">
       <widget class="QLabel" name="labRate3">
        <property name="text">
         <string>逾期利率</string>
        </property>
       </widget>
      </item>
      <item row="1" column="5">
       <widget class="QLineEdit" name="txtRate4">
        <property name="text">
         <string>0.0003</string>
        </property>
       </widget>
      </item>
      <item row="0" column="1">
       <widget class="QDateEdit" name="dateStart">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="displayFormat">
         <string>yyyy-MM-dd</string>
        </property>
        <property name="calendarPopup">
         <bool>true</bool>
        </property>
       </widget>
      </item>
      <item row="1" column="1">
       <widget class="QDateEdit" name="dateEnd">
        <property name="displayFormat">
         <string>yyyy-MM-dd</string>
        </property>
        <property name="calendarPopup">
         <bool>true</bool>
        </property>
       </widget>
      </item>
      <item row="0" column="3">
       <widget class="QLineEdit" name="txtMoney2">
        <property name="text">
         <string>100000</string>
        </property>
       </widget>
      </item>
      <item row="1" column="3">
       <widget class="QLineEdit" name="txtRate2">
        <property name="text">
         <string>0.0003</string>
        </property>
       </widget>
      </item>
      <item row="0" column="0">
       <widget class="QLabel" name="labStart">
        <property name="text">
         <string>贷款日期</string>
        </property>
       </widget>
      </item>
      <item row="1" column="4">
       <widget class="QLabel" name="labRate4">
        <property name="text">
         <string>复利利率</string>
        </property>
       </widget>
      </item>
      <item row="2" column="0">
       <widget class="QLabel" name="labValue2">
        <property name="text">
         <string>贷款利息</string>
        </property>
       </widget>
      </item>
      <item row="2" column="1">
       <widget class="QLineEdit" name="txtResult2"/>
      </item>
     </layout>
    </widget>
   </item>
   <item row="2" column="0">
    <spacer name="verticalSpacer">
     <property name="orientation">
      <enum>Qt::Vertical</enum>
     </property>
     <property name="sizeHint" stdset="0">
      <size>
       <width>20</width>
       <height>489</height>
      </size>
     </property>
    </spacer>
   </item>
  </layout>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <tabstops>
  <tabstop>txtMoneyCurrent</tabstop>
  <tabstop>txtRate</tabstop>
  <tabstop>cboxYear</tabstop>
  <tabstop>txtYears</tabstop>
  <tabstop>cboxType</tabstop>
  <tabstop>txtMoneyAll</tabstop>
  <tabstop>btnCalc</tabstop>
  <tabstop>rbtn1</tabstop>
  <tabstop>txtValue1</tabstop>
  <tabstop>rbtn2</tabstop>
  <tabstop>txtValue2</tabstop>
  <tabstop>txtValue</tabstop>
 </tabstops>
 <resources/>
 <connections/>
</ui>
```
