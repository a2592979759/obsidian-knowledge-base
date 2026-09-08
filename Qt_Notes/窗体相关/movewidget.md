---
tags:
  - Qt
  - 窗体相关
---
# 通用控件移动 (`movewidget`)

> 通用控件移动

## 效果图

![[QWidgetDemo_assert/movewidget.jpg]]

## 分类

- 类型: 窗体相关 `widget`
- 源码目录: `widget/movewidget`

## 技术要点

**用到的 Qt 类**: QWidget QTextCodec QObject QEvent QPushButton QProgressBar QApplication QPoint QMouseEvent QFont

**特性/模块**: Q_OBJECT

## 完整源码

### `./frmmovewidget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmovewidget.h"
#include "ui_frmmovewidget.h"
#include "qpushbutton.h"
#include "qprogressbar.h"
#include "movewidget.h"

frmMoveWidget::frmMoveWidget(QWidget *parent) : QWidget(parent), ui(new Ui::frmMoveWidget)
{
    ui->setupUi(this);
    this->initForm();
}

frmMoveWidget::~frmMoveWidget()
{
    delete ui;
}

void frmMoveWidget::initForm()
{
    QPushButton *btn1 = new QPushButton(this);
    btn1->setGeometry(10, 10, 250, 25);
    btn1->setText("按住我拖动(仅限左键拖动)");
    MoveWidget *moveWidget1 = new MoveWidget(this);
    moveWidget1->setWidget(btn1);

    QPushButton *btn2 = new QPushButton(this);
    btn2->setGeometry(10, 40, 250, 25);
    btn2->setText("按住我拖动(支持右键拖动)");
    MoveWidget *moveWidget2 = new MoveWidget(this);
    moveWidget2->setWidget(btn2);
    moveWidget2->setLeftButton(false);

    QProgressBar *bar = new QProgressBar(this);
    bar->setGeometry(10, 70, 250, 25);
    bar->setRange(0, 0);
    bar->setTextVisible(false);
    MoveWidget *moveWidget3 = new MoveWidget(this);
    moveWidget3->setWidget(bar);
}
```

### `./frmmovewidget.h`

```cpp
﻿#ifndef FRMMOVEWIDGET_H
#define FRMMOVEWIDGET_H

#include <QWidget>

namespace Ui {
class frmMoveWidget;
}

class frmMoveWidget : public QWidget
{
    Q_OBJECT

public:
    explicit frmMoveWidget(QWidget *parent = 0);
    ~frmMoveWidget();

private:
    Ui::frmMoveWidget *ui;

private slots:
    void initForm();
};

#endif // FRMMOVEWIDGET_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmovewidget.h"
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

    frmMoveWidget w;
    w.setWindowTitle("通用移动类 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./movewidget.cpp`

```cpp
﻿#include "movewidget.h"
#include "qevent.h"
#include "qdebug.h"

MoveWidget::MoveWidget(QObject *parent) : QObject(parent)
{
    lastPoint = QPoint(0, 0);
    pressed = false;
    leftButton = true;
    inControl = true;
    widget = 0;
}

bool MoveWidget::eventFilter(QObject *watched, QEvent *event)
{
    if (widget && watched == widget) {
        int type = event->type();
        QMouseEvent *mouseEvent = (QMouseEvent *)event;
        if (type == QEvent::MouseButtonPress) {
            //如果限定了只能鼠标左键拖动则判断当前是否是鼠标左键
            if (leftButton && mouseEvent->button() != Qt::LeftButton) {
                return false;
            }

            //判断控件的区域是否包含了当前鼠标的坐标
            if (widget->rect().contains(mouseEvent->pos())) {
                lastPoint = mouseEvent->pos();
                pressed = true;
            }
        } else if (type == QEvent::MouseMove && pressed) {
            //计算坐标偏移值,调用move函数移动过去
            int offsetX = mouseEvent->pos().x() - lastPoint.x();
            int offsetY = mouseEvent->pos().y() - lastPoint.y();
            int x = widget->x() + offsetX;
            int y = widget->y() + offsetY;
            if (inControl) {
                //可以自行调整限定在容器中的范围,这里默认保留20个像素在里面
                int offset = 20;
                bool xyOut = (x + widget->width() < offset || y + widget->height() < offset);
                bool whOut = false;
                QWidget *w = (QWidget *)widget->parent();
                if (w) {
                    whOut = (w->width() - x < offset || w->height() - y < offset);
                }
                if (xyOut || whOut) {
                    return false;
                }
            }

            widget->move(x, y);
        } else if (type == QEvent::MouseButtonRelease && pressed) {
            pressed = false;
        }
    }

    return QObject::eventFilter(watched, event);
}

void MoveWidget::setLeftButton(bool leftButton)
{
    this->leftButton = leftButton;
}

void MoveWidget::setInControl(bool inControl)
{
    this->inControl = inControl;
}

void MoveWidget::setWidget(QWidget *widget)
{
    if (this->widget == 0) {
        this->widget = widget;
        this->widget->installEventFilter(this);
    }
}
```

### `./movewidget.h`

```cpp
﻿#ifndef MOVEWIDGET_H
#define MOVEWIDGET_H

/**
 * 通用控件移动类 作者:feiyangqingyun(QQ:517216493) 2019-09-28
 * 1. 可以指定需要移动的widget。
 * 2. 可设置是否限定鼠标左键拖动。
 * 3. 支持任意widget控件。
 */

#include <QWidget>

#ifdef quc
class Q_DECL_EXPORT MoveWidget : public QObject
#else
class MoveWidget : public QObject
#endif

{
    Q_OBJECT
public:
    explicit MoveWidget(QObject *parent = 0);

protected:
    bool eventFilter(QObject *watched, QEvent *event);

private:
    QPoint lastPoint;   //最后按下的坐标
    bool pressed;       //鼠标是否按下
    bool leftButton;    //限定鼠标左键
    bool inControl;     //限定在容器内
    QWidget *widget;    //移动的控件

public Q_SLOTS:
    //设置是否限定鼠标左键
    void setLeftButton(bool leftButton);
    //设置是否限定不能移出容器外面
    void setInControl(bool inControl);
    //设置要移动的控件
    void setWidget(QWidget *widget);
};

#endif // MOVEWIDGET_H
```

### `./frmmovewidget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmMoveWidget</class>
 <widget class="QWidget" name="frmMoveWidget">
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
 </widget>
 <resources/>
 <connections/>
</ui>
```
