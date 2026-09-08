---
tags:
  - Qt
  - 其他相关
---
# 鼠标十字线 (`mouseline`)

> 鼠标十字线

## 效果图

![[QWidgetDemo_assert/mouseline.jpg]]

## 分类

- 类型: 其他相关 `other`
- 源码目录: `other/mouseline`

## 技术要点

**用到的 Qt 类**: QTextCodec QWidget QMouseEvent QApplication QPaintEvent QPainter QFont QPen QPoint

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
    w.setWindowTitle("鼠标十字线 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./widget.cpp`

```cpp
﻿#include "widget.h"
#include "ui_widget.h"
#include "qpainter.h"
#include "qevent.h"
#include "qdebug.h"

Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget)
{
    ui->setupUi(this);
    this->setMouseTracking(true);
}

Widget::~Widget()
{
    delete ui;
}

void Widget::mouseMoveEvent(QMouseEvent *event)
{
    lastPos = event->pos();
    update();
}

void Widget::mouseReleaseEvent(QMouseEvent *event)
{
    //这里是鼠标按下的坐标,自己存到数据库
    lastPos = event->pos();
    update();
    qDebug() << lastPos;
}

void Widget::paintEvent(QPaintEvent *)
{
    QPainter painter(this);

    QPen pen;
    pen.setWidth(5);
    pen.setColor(Qt::red);
    painter.setPen(pen);

    //绘制横向线
    painter.drawLine(0, lastPos.y(), width(), lastPos.y());
    //绘制纵向线
    painter.drawLine(lastPos.x(), 0, lastPos.x(), height());
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
    void mouseMoveEvent(QMouseEvent *event);
    void mouseReleaseEvent(QMouseEvent *event);
    void paintEvent(QPaintEvent *);

private:
    Ui::Widget *ui;
    QPoint lastPos;
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
