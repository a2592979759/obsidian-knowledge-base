---
tags:
  - Qt
  - 控件相关
---
# 图片开关 (`imageswitch`)

> 图片开关

## 效果图

![[QWidgetDemo_assert/imageswitch.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/imageswitch`

## 技术要点

**用到的 Qt 类**: QWidget QTextCodec QSize QPainter QString QMouseEvent QPaintEvent QApplication QGridLayout QImage QPoint QFont

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmimageswitch.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmimageswitch.h"
#include "ui_frmimageswitch.h"
#include "qdebug.h"

frmImageSwitch::frmImageSwitch(QWidget *parent) : QWidget(parent), ui(new Ui::frmImageSwitch)
{
    ui->setupUi(this);
    this->initForm();
}

frmImageSwitch::~frmImageSwitch()
{
    delete ui;
}

void frmImageSwitch::initForm()
{
    ui->imageSwitch1->setChecked(true);
    ui->imageSwitch2->setChecked(true);
    ui->imageSwitch3->setChecked(true);

    ui->imageSwitch1->setFixedSize(87, 30);
    ui->imageSwitch2->setFixedSize(87, 30);
    ui->imageSwitch3->setFixedSize(87, 30);

    ui->imageSwitch1->setButtonStyle(ImageSwitch::ButtonStyle_1);
    ui->imageSwitch2->setButtonStyle(ImageSwitch::ButtonStyle_2);
    ui->imageSwitch3->setButtonStyle(ImageSwitch::ButtonStyle_3);

    //绑定选中切换信号
    connect(ui->imageSwitch1, SIGNAL(checkedChanged(bool)), this, SLOT(checkedChanged(bool)));
    connect(ui->imageSwitch2, SIGNAL(checkedChanged(bool)), this, SLOT(checkedChanged(bool)));
    connect(ui->imageSwitch3, SIGNAL(checkedChanged(bool)), this, SLOT(checkedChanged(bool)));
}

void frmImageSwitch::checkedChanged(bool checked)
{
    qDebug() << sender() << checked;
}
```

### `./frmimageswitch.h`

```cpp
﻿#ifndef FRMIMAGESWITCH_H
#define FRMIMAGESWITCH_H

#include <QWidget>

namespace Ui {
class frmImageSwitch;
}

class frmImageSwitch : public QWidget
{
    Q_OBJECT

public:
    explicit frmImageSwitch(QWidget *parent = 0);
    ~frmImageSwitch();

private:
    Ui::frmImageSwitch *ui;

private slots:
    void initForm();
    void checkedChanged(bool checked);
};

#endif // FRMIMAGESWITCH_H
```

### `./imageswitch.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "imageswitch.h"
#include "qpainter.h"
#include "qdebug.h"

ImageSwitch::ImageSwitch(QWidget *parent) : QWidget(parent)
{
    isChecked = false;
    buttonStyle = ButtonStyle_2;

    imgOffFile = ":/image/imageswitch/btncheckoff2.png";
    imgOnFile = ":/image/imageswitch/btncheckon2.png";
    imgFile = imgOffFile;
}

void ImageSwitch::mousePressEvent(QMouseEvent *)
{
    imgFile = isChecked ? imgOffFile : imgOnFile;
    isChecked = !isChecked;
    Q_EMIT checkedChanged(isChecked);
    this->update();
}

void ImageSwitch::paintEvent(QPaintEvent *)
{
    QPainter painter(this);
    painter.setRenderHints(QPainter::SmoothPixmapTransform);
    QImage img(imgFile);
    img = img.scaled(this->size(), Qt::KeepAspectRatio, Qt::SmoothTransformation);

    //按照比例自动居中绘制
    int pixX = rect().center().x() - img.width() / 2;
    int pixY = rect().center().y() - img.height() / 2;
    QPoint point(pixX, pixY);
    painter.drawImage(point, img);
}

QSize ImageSwitch::sizeHint() const
{
    return QSize(87, 28);
}

QSize ImageSwitch::minimumSizeHint() const
{
    return QSize(87, 28);
}

bool ImageSwitch::getChecked() const
{
    return isChecked;
}

void ImageSwitch::setChecked(bool isChecked)
{
    if (this->isChecked != isChecked) {
        this->isChecked = isChecked;
        imgFile = isChecked ? imgOnFile : imgOffFile;
        this->update();
    }
}

ImageSwitch::ButtonStyle ImageSwitch::getButtonStyle() const
{
    return this->buttonStyle;
}

void ImageSwitch::setButtonStyle(const ImageSwitch::ButtonStyle &buttonStyle)
{
    if (this->buttonStyle != buttonStyle) {
        this->buttonStyle = buttonStyle;

        if (buttonStyle == ButtonStyle_1) {
            imgOffFile = ":/image/imageswitch/btncheckoff1.png";
            imgOnFile = ":/image/imageswitch/btncheckon1.png";
            this->resize(87, 28);
        } else if (buttonStyle == ButtonStyle_2) {
            imgOffFile = ":/image/imageswitch/btncheckoff2.png";
            imgOnFile = ":/image/imageswitch/btncheckon2.png";
            this->resize(87, 28);
        } else if (buttonStyle == ButtonStyle_3) {
            imgOffFile = ":/image/imageswitch/btncheckoff3.png";
            imgOnFile = ":/image/imageswitch/btncheckon3.png";
            this->resize(96, 38);
        }

        imgFile = isChecked ? imgOnFile : imgOffFile;
        setChecked(isChecked);
        this->update();
        updateGeometry();
    }
}
```

### `./imageswitch.h`

```cpp
﻿#ifndef IMAGESWITCH_H
#define IMAGESWITCH_H

/**
 * 图片开关控件 作者:feiyangqingyun(QQ:517216493) 2016-11-25
 * 1. 自带三种开关按钮样式。
 * 2. 可自定义开关图片。
 */

#include <QWidget>

#ifdef quc
class Q_DECL_EXPORT ImageSwitch : public QWidget
#else
class ImageSwitch : public QWidget
#endif

{
    Q_OBJECT
    Q_ENUMS(ButtonStyle)

    Q_PROPERTY(bool isChecked READ getChecked WRITE setChecked)
    Q_PROPERTY(ButtonStyle buttonStyle READ getButtonStyle WRITE setButtonStyle)

public:
    enum ButtonStyle {
        ButtonStyle_1 = 0,  //开关样式1
        ButtonStyle_2 = 1,  //开关样式2
        ButtonStyle_3 = 2   //开关样式3
    };

    explicit ImageSwitch(QWidget *parent = 0);

protected:
    void mousePressEvent(QMouseEvent *);
    void paintEvent(QPaintEvent *event);

private:
    bool isChecked;             //是否选中
    ButtonStyle buttonStyle;    //按钮样式

    QString imgOffFile;         //关闭图片
    QString imgOnFile;          //开启图片
    QString imgFile;            //当前图片

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //获取和设置是否选中
    bool getChecked() const;
    void setChecked(bool isChecked);

    //获取和设置按钮样式
    ButtonStyle getButtonStyle() const;
    void setButtonStyle(const ImageSwitch::ButtonStyle &buttonStyle);

Q_SIGNALS:
    void checkedChanged(bool checked);
};

#endif // IMAGESWITCH_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmimageswitch.h"
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

    frmImageSwitch w;
    w.setWindowTitle("图片背景开关 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmimageswitch.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmImageSwitch</class>
 <widget class="QWidget" name="frmImageSwitch">
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
  <layout class="QGridLayout" name="gridLayout">
   <item row="0" column="2">
    <widget class="ImageSwitch" name="imageSwitch3" native="true"/>
   </item>
   <item row="0" column="0">
    <widget class="ImageSwitch" name="imageSwitch1" native="true"/>
   </item>
   <item row="0" column="1">
    <widget class="ImageSwitch" name="imageSwitch2" native="true"/>
   </item>
   <item row="1" column="0">
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
   <item row="0" column="3">
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
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>ImageSwitch</class>
   <extends>QWidget</extends>
   <header>imageswitch.h</header>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
