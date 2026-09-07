---
tags:
  - Qt
  - 其他相关
---
# 多对象共用槽 (`multobj2slot`)

> 多对象共用槽

## 效果图

![[QWidgetDemo_assert/multobj2slot.jpg]]

## 分类

- 类型: 其他相关 `other`
- 源码目录: `other/multobj2slot`

## 技术要点

**用到的 Qt 类**: QTextCodec QString QWidget QPushButton QSignalMapper QApplication QFont QTime

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

#if (QT_VERSION <= QT_VERSION_CHECK(5,0,0))
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
    w.setWindowTitle("多对象共用槽 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./widget.cpp`

```cpp
﻿#include "widget.h"
#include "ui_widget.h"
#include "qpushbutton.h"
#include "qsignalmapper.h"
#include "qdatetime.h"
#include "qdebug.h"

#define TIMEMS QTime::currentTime().toString("hh:mm:ss zzz")
Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget)
{
    ui->setupUi(this);
    this->initBtn();
}

Widget::~Widget()
{
    delete ui;
}

void Widget::initBtn()
{
    QSignalMapper *signMap = new QSignalMapper(this);
    connect(signMap, SIGNAL(mapped(QString)), this, SLOT(doBtn(QString)));

    int x = 5, y = -25;
    for (int i = 0; i < 1000; ++i) {
        //动态设置坐标
        x += 80;
        if (i % 10 == 0) {
            x = 5;
            y += 30;
        }

        QPushButton *btn = new QPushButton(this);
        btn->setObjectName(QString("btn_%1").arg(i + 1));
        btn->setText(QString("text_%1").arg(i + 1));
        btn->setGeometry(x, y, 75, 25);

        //方法0: 每个按钮关联到一个独立的槽,代码量大不可取放弃
        //方式1: 绑定到一个槽函数
        connect(btn, SIGNAL(clicked(bool)), this, SLOT(doBtn()));
        //方式2: 通过 QSignalMapper 转发信号
        connect(btn, SIGNAL(clicked(bool)), signMap, SLOT(map()));
        signMap->setMapping(btn, btn->objectName());
        //方法3: 用 lambda 表达式
#if (QT_VERSION >= QT_VERSION_CHECK(5,6,0))
        connect(btn, &QPushButton::clicked, [btn] {
            QString name = btn->objectName();
            qDebug() << TIMEMS << "doBtn3" << name;
        });

        connect(btn, &QPushButton::clicked, [=]() {
            QString name = btn->objectName();
            qDebug() << TIMEMS << "doBtn3" << name;
        });
#endif
    }
}

void Widget::doBtn()
{
    QPushButton *btn = (QPushButton *)sender();
    QString name = btn->objectName();
    qDebug() << TIMEMS << "doBtn1" << name;
}

void Widget::doBtn(const QString &name)
{
    qDebug() << TIMEMS << "doBtn2" << name;
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
    void initBtn();
    void doBtn();
    void doBtn(const QString &name);
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
