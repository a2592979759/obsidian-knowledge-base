---
tags:
  - Qt
  - 控件相关
  - 高质量
---
# 电池电量 (`battery`)

> 电池电量

## 效果图

![[QWidgetDemo_assert/battery.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/battery`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QColor QWidget QPainter QTextCodec QPointF QSize QTimer QRectF QPaintEvent QLinearGradient QApplication QPen QSlider QFont

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./battery.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "battery.h"
#include "qpainter.h"
#include "qtimer.h"
#include "qdebug.h"

Battery::Battery(QWidget *parent) : QWidget(parent)
{
    minValue = 0;
    maxValue = 100;
    value = 0;
    alarmValue = 30;

    animation = true;
    animationStep = 0.5;

    borderWidth = 5;
    borderRadius = 8;
    bgRadius = 5;
    headRadius = 3;

    borderColorStart = QColor(100, 100, 100);
    borderColorEnd = QColor(80, 80, 80);
    alarmColorStart = QColor(250, 118, 113);
    alarmColorEnd = QColor(204, 38, 38);
    normalColorStart = QColor(50, 205, 51);
    normalColorEnd = QColor(60, 179, 133);

    isForward = false;
    currentValue = 0;

    timer = new QTimer(this);
    timer->setInterval(10);
    connect(timer, SIGNAL(timeout()), this, SLOT(updateValue()));
}

Battery::~Battery()
{
    if (timer->isActive()) {
        timer->stop();
    }
}

void Battery::paintEvent(QPaintEvent *)
{
    //绘制准备工作,启用反锯齿
    QPainter painter(this);
    painter.setRenderHints(QPainter::Antialiasing | QPainter::TextAntialiasing);

    //绘制边框
    drawBorder(&painter);
    //绘制背景
    drawBg(&painter);
    //绘制头部
    drawHead(&painter);
}

void Battery::drawBorder(QPainter *painter)
{
    painter->save();

    double headWidth = width() / 15;
    double batteryWidth = width() - headWidth;

    //绘制电池边框
    QPointF topLeft(borderWidth, borderWidth);
    QPointF bottomRight(batteryWidth, height() - borderWidth);
    batteryRect = QRectF(topLeft, bottomRight);

    painter->setPen(QPen(borderColorStart, borderWidth));
    painter->setBrush(Qt::NoBrush);
    painter->drawRoundedRect(batteryRect, borderRadius, borderRadius);

    painter->restore();
}

void Battery::drawBg(QPainter *painter)
{
    if (value == minValue) {
        return;
    }

    painter->save();

    QLinearGradient batteryGradient(QPointF(0, 0), QPointF(0, height()));
    if (currentValue <= alarmValue) {
        batteryGradient.setColorAt(0.0, alarmColorStart);
        batteryGradient.setColorAt(1.0, alarmColorEnd);
    } else {
        batteryGradient.setColorAt(0.0, normalColorStart);
        batteryGradient.setColorAt(1.0, normalColorEnd);
    }

    int margin = qMin(width(), height()) / 20;
    double unit = (batteryRect.width() - (margin * 2)) / (maxValue - minValue);
    double width = currentValue * unit;
    QPointF topLeft(batteryRect.topLeft().x() + margin, batteryRect.topLeft().y() + margin);
    QPointF bottomRight(width + margin + borderWidth, batteryRect.bottomRight().y() - margin);
    QRectF rect(topLeft, bottomRight);

    painter->setPen(Qt::NoPen);
    painter->setBrush(batteryGradient);
    painter->drawRoundedRect(rect, bgRadius, bgRadius);

    painter->restore();
}

void Battery::drawHead(QPainter *painter)
{
    painter->save();

    QPointF headRectTopLeft(batteryRect.topRight().x(), height() / 3);
    QPointF headRectBottomRight(width(), height() - height() / 3);
    QRectF headRect(headRectTopLeft, headRectBottomRight);

    QLinearGradient headRectGradient(headRect.topLeft(), headRect.bottomLeft());
    headRectGradient.setColorAt(0.0, borderColorStart);
    headRectGradient.setColorAt(1.0, borderColorEnd);

    painter->setPen(Qt::NoPen);
    painter->setBrush(headRectGradient);
    painter->drawRoundedRect(headRect, headRadius, headRadius);

    painter->restore();
}

void Battery::updateValue()
{
    if (isForward) {
        currentValue -= animationStep;
        if (currentValue <= value) {
            currentValue = value;
            timer->stop();
        }
    } else {
        currentValue += animationStep;
        if (currentValue >= value) {
            currentValue = value;
            timer->stop();
        }
    }

    this->update();
}

QSize Battery::sizeHint() const
{
    return QSize(150, 80);
}

QSize Battery::minimumSizeHint() const
{
    return QSize(30, 10);
}

void Battery::setRange(double minValue, double maxValue)
{
    //如果最小值大于或者等于最大值则不设置
    if (minValue >= maxValue) {
        return;
    }

    this->minValue = minValue;
    this->maxValue = maxValue;

    //如果目标值不在范围值内,则重新设置目标值
    //值小于最小值则取最小值,大于最大值则取最大值
    if (value < minValue) {
        setValue(minValue);
    } else if (value > maxValue) {
        setValue(maxValue);
    }

    this->update();
}

void Battery::setRange(int minValue, int maxValue)
{
    setRange((double)minValue, (double)maxValue);
}

double Battery::getMinValue() const
{
    return this->minValue;
}

void Battery::setMinValue(double minValue)
{
    setRange(minValue, maxValue);
}

double Battery::getMaxValue() const
{
    return this->maxValue;
}

void Battery::setMaxValue(double maxValue)
{
    setRange(minValue, maxValue);
}

double Battery::getValue() const
{
    return this->value;
}

void Battery::setValue(double value)
{
    //值和当前值一致则无需处理
    if (value == this->value) {
        return;
    }

    //值小于最小值则取最小值,大于最大值则取最大值
    if (value < minValue) {
        value = minValue;
    } else if (value > maxValue) {
        value = maxValue;
    }

    if (value > currentValue) {
        isForward = false;
    } else if (value < currentValue) {
        isForward = true;
    } else {
        this->value = value;
        this->update();
        return;
    }

    this->value = value;
    Q_EMIT valueChanged(value);
    if (animation) {
        timer->stop();
        timer->start();
    } else {
        this->currentValue = value;
        this->update();
    }
}

void Battery::setValue(int value)
{
    setValue((double)value);
}

double Battery::getAlarmValue() const
{
    return this->alarmValue;
}

void Battery::setAlarmValue(double alarmValue)
{
    if (this->alarmValue != alarmValue) {
        this->alarmValue = alarmValue;
        this->update();
    }
}

void Battery::setAlarmValue(int alarmValue)
{
    setAlarmValue((double)alarmValue);
}


bool Battery::getAnimation() const
{
    return this->animation;
}

void Battery::setAnimation(bool animation)
{
    if (this->animation != animation) {
        this->animation = animation;
        this->update();
    }
}

double Battery::getAnimationStep() const
{
    return this->animationStep;
}

void Battery::setAnimationStep(double animationStep)
{
    if (this->animationStep != animationStep) {
        this->animationStep = animationStep;
        this->update();
    }
}

int Battery::getBorderWidth() const
{
    return this->borderWidth;
}

void Battery::setBorderWidth(int borderWidth)
{
    if (this->borderWidth != borderWidth) {
        this->borderWidth = borderWidth;
        this->update();
    }
}

int Battery::getBorderRadius() const
{
    return this->borderRadius;
}

void Battery::setBorderRadius(int borderRadius)
{
    if (this->borderRadius != borderRadius) {
        this->borderRadius = borderRadius;
        this->update();
    }
}

int Battery::getBgRadius() const
{
    return this->bgRadius;
}

void Battery::setBgRadius(int bgRadius)
{
    if (this->bgRadius != bgRadius) {
        this->bgRadius = bgRadius;
        this->update();
    }
}

int Battery::getHeadRadius() const
{
    return this->headRadius;
}

void Battery::setHeadRadius(int headRadius)
{
    if (this->headRadius != headRadius) {
        this->headRadius = headRadius;
        this->update();
    }
}

QColor Battery::getBorderColorStart() const
{
    return this->borderColorStart;
}

void Battery::setBorderColorStart(const QColor &borderColorStart)
{
    if (this->borderColorStart != borderColorStart) {
        this->borderColorStart = borderColorStart;
        this->update();
    }
}

QColor Battery::getBorderColorEnd() const
{
    return this->borderColorEnd;
}

void Battery::setBorderColorEnd(const QColor &borderColorEnd)
{
    if (this->borderColorEnd != borderColorEnd) {
        this->borderColorEnd = borderColorEnd;
        this->update();
    }
}

QColor Battery::getAlarmColorStart() const
{
    return this->alarmColorStart;
}

void Battery::setAlarmColorStart(const QColor &alarmColorStart)
{
    if (this->alarmColorStart != alarmColorStart) {
        this->alarmColorStart = alarmColorStart;
        this->update();
    }
}

QColor Battery::getAlarmColorEnd() const
{
    return this->alarmColorEnd;
}

void Battery::setAlarmColorEnd(const QColor &alarmColorEnd)
{
    if (this->alarmColorEnd != alarmColorEnd) {
        this->alarmColorEnd = alarmColorEnd;
        this->update();
    }
}

QColor Battery::getNormalColorStart() const
{
    return this->normalColorStart;
}

void Battery::setNormalColorStart(const QColor &normalColorStart)
{
    if (this->normalColorStart != normalColorStart) {
        this->normalColorStart = normalColorStart;
        this->update();
    }
}

QColor Battery::getNormalColorEnd() const
{
    return this->normalColorEnd;
}

void Battery::setNormalColorEnd(const QColor &normalColorEnd)
{
    if (this->normalColorEnd != normalColorEnd) {
        this->normalColorEnd = normalColorEnd;
        this->update();
    }
}
```

### `./battery.h`

```cpp
﻿#ifndef BATTERY_H
#define BATTERY_H

/**
 * 电池电量控件 作者:feiyangqingyun(QQ:517216493) 2016-10-23
 * 1. 可设置电池电量，动态切换电池电量变化。
 * 2. 可设置电池电量警戒值。
 * 3. 可设置电池电量正常颜色和报警颜色。
 * 4. 可设置边框渐变颜色。
 * 5. 可设置电量变化时每次移动的步长。
 * 6. 可设置边框圆角角度、背景进度圆角角度、头部圆角角度。
 */

#include <QWidget>

#ifdef quc
class Q_DECL_EXPORT Battery : public QWidget
#else
class Battery : public QWidget
#endif

{
    Q_OBJECT

    Q_PROPERTY(double minValue READ getMinValue WRITE setMinValue)
    Q_PROPERTY(double maxValue READ getMaxValue WRITE setMaxValue)
    Q_PROPERTY(double value READ getValue WRITE setValue)
    Q_PROPERTY(double alarmValue READ getAlarmValue WRITE setAlarmValue)

    Q_PROPERTY(bool animation READ getAnimation WRITE setAnimation)
    Q_PROPERTY(double animationStep READ getAnimationStep WRITE setAnimationStep)

    Q_PROPERTY(int borderWidth READ getBorderWidth WRITE setBorderWidth)
    Q_PROPERTY(int borderRadius READ getBorderRadius WRITE setBorderRadius)
    Q_PROPERTY(int bgRadius READ getBgRadius WRITE setBgRadius)
    Q_PROPERTY(int headRadius READ getHeadRadius WRITE setHeadRadius)

    Q_PROPERTY(QColor borderColorStart READ getBorderColorStart WRITE setBorderColorStart)
    Q_PROPERTY(QColor borderColorEnd READ getBorderColorEnd WRITE setBorderColorEnd)

    Q_PROPERTY(QColor alarmColorStart READ getAlarmColorStart WRITE setAlarmColorStart)
    Q_PROPERTY(QColor alarmColorEnd READ getAlarmColorEnd WRITE setAlarmColorEnd)

    Q_PROPERTY(QColor normalColorStart READ getNormalColorStart WRITE setNormalColorStart)
    Q_PROPERTY(QColor normalColorEnd READ getNormalColorEnd WRITE setNormalColorEnd)

public:
    explicit Battery(QWidget *parent = 0);
    ~Battery();

protected:
    void paintEvent(QPaintEvent *);
    void drawBorder(QPainter *painter);
    void drawBg(QPainter *painter);
    void drawHead(QPainter *painter);

private slots:
    void updateValue();

private:
    double minValue;        //最小值
    double maxValue;        //最大值
    double value;           //目标电量
    double alarmValue;      //电池电量警戒值

    bool animation;         //是否启用动画显示
    double animationStep;   //动画显示时步长

    int borderWidth;        //边框粗细
    int borderRadius;       //边框圆角角度
    int bgRadius;           //背景进度圆角角度
    int headRadius;         //头部圆角角度

    QColor borderColorStart;//边框渐变开始颜色
    QColor borderColorEnd;  //边框渐变结束颜色

    QColor alarmColorStart; //电池低电量时的渐变开始颜色
    QColor alarmColorEnd;   //电池低电量时的渐变结束颜色

    QColor normalColorStart;//电池正常电量时的渐变开始颜色
    QColor normalColorEnd;  //电池正常电量时的渐变结束颜色

    bool isForward;         //是否往前移
    double currentValue;    //当前电量
    QRectF batteryRect;     //电池主体区域
    QTimer *timer;          //绘制定时器

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //设置范围值
    void setRange(double minValue, double maxValue);
    void setRange(int minValue, int maxValue);

    //获取和设置最小值
    double getMinValue() const;
    void setMinValue(double minValue);

    //获取和设置最大值
    double getMaxValue() const;
    void setMaxValue(double maxValue);

    //获取和设置电池电量值
    double getValue() const;
    void setValue(double value);

    //获取和设置电池电量警戒值
    double getAlarmValue() const;
    void setAlarmValue(double alarmValue);

    //获取和设置是否启用动画显示
    bool getAnimation() const;
    void setAnimation(bool animation);

    //获取和设置动画显示的步长
    double getAnimationStep() const;
    void setAnimationStep(double animationStep);

    //获取和设置边框粗细
    int getBorderWidth() const;
    void setBorderWidth(int borderWidth);

    //获取和设置边框圆角角度
    int getBorderRadius() const;
    void setBorderRadius(int borderRadius);

    //获取和设置背景圆角角度
    int getBgRadius() const;
    void setBgRadius(int bgRadius);

    //获取和设置头部圆角角度
    int getHeadRadius() const;
    void setHeadRadius(int headRadius);

    //获取和设置边框渐变颜色
    QColor getBorderColorStart() const;
    void setBorderColorStart(const QColor &borderColorStart);

    QColor getBorderColorEnd() const;
    void setBorderColorEnd(const QColor &borderColorEnd);

    //获取和设置电池电量报警时的渐变颜色
    QColor getAlarmColorStart() const;
    void setAlarmColorStart(const QColor &alarmColorStart);

    QColor getAlarmColorEnd() const;
    void setAlarmColorEnd(const QColor &alarmColorEnd);

    //获取和设置电池电量正常时的渐变颜色
    QColor getNormalColorStart() const;
    void setNormalColorStart(const QColor &normalColorStart);

    QColor getNormalColorEnd() const;
    void setNormalColorEnd(const QColor &normalColorEnd);

public Q_SLOTS:
    void setValue(int value);
    void setAlarmValue(int alarmValue);

Q_SIGNALS:
    void valueChanged(double value);
};

#endif // BATTERY_H
```

### `./frmbattery.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmbattery.h"
#include "ui_frmbattery.h"

frmBattery::frmBattery(QWidget *parent) : QWidget(parent), ui(new Ui::frmBattery)
{
    ui->setupUi(this);
    this->initForm();
}

frmBattery::~frmBattery()
{
    delete ui;
}

void frmBattery::initForm()
{
    connect(ui->horizontalSlider, SIGNAL(valueChanged(int)), ui->battery, SLOT(setValue(int)));
    ui->horizontalSlider->setValue(30);
}
```

### `./frmbattery.h`

```cpp
﻿#ifndef FRMBATTERY_H
#define FRMBATTERY_H

#include <QWidget>

namespace Ui {
class frmBattery;
}

class frmBattery : public QWidget
{
    Q_OBJECT

public:
    explicit frmBattery(QWidget *parent = 0);
    ~frmBattery();

private:
    Ui::frmBattery *ui;

private slots:
    void initForm();
};

#endif // FRMBATTERY_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmbattery.h"
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

    frmBattery w;
    w.setWindowTitle("电池电量控件 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmbattery.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmBattery</class>
 <widget class="QWidget" name="frmBattery">
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
  <widget class="Battery" name="battery" native="true">
   <property name="geometry">
    <rect>
     <x>9</x>
     <y>9</y>
     <width>482</width>
     <height>257</height>
    </rect>
   </property>
  </widget>
  <widget class="QSlider" name="horizontalSlider">
   <property name="geometry">
    <rect>
     <x>9</x>
     <y>272</y>
     <width>481</width>
     <height>19</height>
    </rect>
   </property>
   <property name="maximum">
    <number>100</number>
   </property>
   <property name="orientation">
    <enum>Qt::Horizontal</enum>
   </property>
   <property name="invertedControls">
    <bool>false</bool>
   </property>
  </widget>
 </widget>
 <customwidgets>
  <customwidget>
   <class>Battery</class>
   <extends>QWidget</extends>
   <header>battery.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
