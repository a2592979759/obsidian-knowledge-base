---
tags:
  - Qt
  - 其他相关
---
# 随机大量矩形 (`drawrect`)

> 随机大量矩形

## 效果图

![[QWidgetDemo_assert/drawrect.jpg]]

## 分类

- 类型: 其他相关 `other`
- 源码目录: `other/drawrect`

## 技术要点

**用到的 Qt 类**: QTextCodec QWidget QTimer QApplication QPaintEvent QPainter QFont QString QPen

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
    //w.setWindowTitle("随机大量矩形 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./widget.cpp`

```cpp
﻿#include "widget.h"
#include "ui_widget.h"
#include "qpainter.h"
#include "qtimer.h"
#include "qdebug.h"

Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget)
{
    ui->setupUi(this);

    interval = 20;
    count = 500;
    this->setWindowTitle(QString("随机大量矩形  帧率: %1  数量: %2 (QQ: 517216493 WX: feiyangqingyun)").arg(1000 / interval).arg(count));

    //定时器测试速度
    QTimer *timer = new QTimer(this);
    connect(timer, SIGNAL(timeout()), this, SLOT(update()));
    timer->start(interval);
}

Widget::~Widget()
{
    delete ui;
}

void Widget::paintEvent(QPaintEvent *)
{
    int width = this->width();
    int height = this->height();
    QPainter painter(this);

    QPen pen;
    pen.setWidth(2);
    pen.setColor(Qt::red);
    painter.setPen(pen);

    for (int i = 0; i < count; ++i) {
        int x = rand() % width;
        int y = rand() % height;
        painter.drawRect(x, y, 30, 30);
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

protected:
    void paintEvent(QPaintEvent *);

private:
    Ui::Widget *ui;
    int interval;
    int count;
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
   <string/>
  </property>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <resources/>
 <connections/>
</ui>
```
