---
tags:
  - Qt
  - 其他相关
---
# 文本框回车自动跳转 (`lineeditnext`)

> 文本框回车自动跳转

## 效果图

![[QWidgetDemo_assert/lineeditnext.jpg]]

## 分类

- 类型: 其他相关 `other`
- 源码目录: `other/lineeditnext`

## 技术要点

**用到的 Qt 类**: QTextCodec QWidget QLineEdit QApplication QFont

**特性/模块**: Q_OBJECT、connect

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
    w.setWindowTitle("回车自动跳转 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./widget.cpp`

```cpp
﻿#include "widget.h"
#include "ui_widget.h"
#include "qlineedit.h"

Widget::Widget(QWidget *parent) :
    QWidget(parent),
    ui(new Ui::Widget)
{
    ui->setupUi(this);
    connect(ui->lineEdit1, SIGNAL(returnPressed()), this, SLOT(next()));
    connect(ui->lineEdit2, SIGNAL(returnPressed()), this, SLOT(next()));
    connect(ui->lineEdit3, SIGNAL(returnPressed()), this, SLOT(next()));
}

Widget::~Widget()
{
    delete ui;
}

void Widget::next()
{
    QLineEdit *lineEdit = (QLineEdit *)sender();
    if (lineEdit == ui->lineEdit1) {
        ui->lineEdit2->setFocus();
    } else if (lineEdit == ui->lineEdit2) {
        ui->lineEdit3->setFocus();
    } else if (lineEdit == ui->lineEdit3) {
        ui->lineEdit1->setFocus();
    }
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
    void next();
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
  <widget class="QLineEdit" name="lineEdit1">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>10</y>
     <width>181</width>
     <height>20</height>
    </rect>
   </property>
   <property name="text">
    <string>Qt自定义控件大全</string>
   </property>
  </widget>
  <widget class="QLineEdit" name="lineEdit2">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>40</y>
     <width>181</width>
     <height>20</height>
    </rect>
   </property>
   <property name="text">
    <string>QQ：517216493</string>
   </property>
  </widget>
  <widget class="QLineEdit" name="lineEdit3">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>70</y>
     <width>181</width>
     <height>20</height>
    </rect>
   </property>
   <property name="text">
    <string>WX：feiyangqingyun</string>
   </property>
  </widget>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <resources/>
 <connections/>
</ui>
```
