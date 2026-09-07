---
tags:
  - Qt
  - 控件相关
---
# 高亮按钮 (`lightbutton`)

> 高亮按钮

## 效果图

![[QWidgetDemo_assert/lightbutton.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/lightbutton`

## 技术要点

**用到的 Qt 类**: QColor QPainter QWidget QTextCodec QTimer QEvent QSize QString QFont QLinearGradient QPainterPath QObject QMouseEvent QPaintEvent QApplication QHBoxLayout QRect QPoint

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmlightbutton.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmlightbutton.h"
#include "ui_frmlightbutton.h"
#include "qdatetime.h"
#include "qtimer.h"

frmLightButton::frmLightButton(QWidget *parent) : QWidget(parent), ui(new Ui::frmLightButton)
{
    ui->setupUi(this);
    this->initForm();
}

frmLightButton::~frmLightButton()
{
    delete ui;
}

void frmLightButton::initForm()
{
    ui->lightButton2->setBgColor(QColor(255, 107, 107));
    ui->lightButton3->setBgColor(QColor(24, 189, 155));

    type = 0;

    QTimer *timer = new QTimer(this);
    timer->setInterval(1000);
    connect(timer, SIGNAL(timeout()), this, SLOT(updateValue()));
    timer->start();
    updateValue();

    //以下方法启动报警
    //ui->lightButton1->setAlarmColor(QColor(255, 0, 0));
    //ui->lightButton1->setNormalColor(QColor(0, 0, 0));
    //ui->lightButton1->startAlarm();
}

void frmLightButton::updateValue()
{
    if (type == 0) {
        ui->lightButton1->setLightGreen();
        ui->lightButton2->setLightRed();
        ui->lightButton3->setLightBlue();
        type = 1;
    } else if (type == 1) {
        ui->lightButton1->setLightBlue();
        ui->lightButton2->setLightGreen();
        ui->lightButton3->setLightRed();
        type = 2;
    } else if (type == 2) {
        ui->lightButton1->setLightRed();
        ui->lightButton2->setLightBlue();
        ui->lightButton3->setLightGreen();
        type = 0;
    }
}
```

### `./frmlightbutton.h`

```cpp
﻿#ifndef FRMLIGHTBUTTON_H
#define FRMLIGHTBUTTON_H

#include <QWidget>

namespace Ui {
class frmLightButton;
}

class frmLightButton : public QWidget
{
    Q_OBJECT

public:
    explicit frmLightButton(QWidget *parent = 0);
    ~frmLightButton();

private:
    Ui::frmLightButton *ui;
    int type;

private slots:
    void initForm();
    void updateValue();
};

#endif // FRMLIGHTBUTTON_H
```

### `./lightbutton.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "lightbutton.h"
#include "qpainter.h"
#include "qpainterpath.h"
#include "qevent.h"
#include "qtimer.h"
#include "qdebug.h"

LightButton::LightButton(QWidget *parent) : QWidget(parent)
{
    text = "";
    textColor = QColor(255, 255, 255);
    alarmColor = QColor(255, 107, 107);
    normalColor = QColor(10, 10, 10);

    borderOutColorStart = QColor(255, 255, 255);
    borderOutColorEnd = QColor(166, 166, 166);

    borderInColorStart = QColor(166, 166, 166);
    borderInColorEnd = QColor(255, 255, 255);

    bgColor = QColor(100, 184, 255);

    showRect = false;
    showOverlay = true;
    overlayColor = QColor(255, 255, 255);

    canMove = false;
    pressed = false;
    this->installEventFilter(this);

    isAlarm = false;
    timerAlarm = new QTimer(this);
    connect(timerAlarm, SIGNAL(timeout()), this, SLOT(alarm()));
    timerAlarm->setInterval(500);
}

bool LightButton::eventFilter(QObject *watched, QEvent *event)
{
    int type = event->type();
    QMouseEvent *mouseEvent = (QMouseEvent *)event;
    if (type == QEvent::MouseButtonPress) {
        if (this->rect().contains(mouseEvent->pos()) && (mouseEvent->button() == Qt::LeftButton)) {
            lastPoint = mouseEvent->pos();
            pressed = true;
        }
    } else if (type == QEvent::MouseMove && pressed) {
        if (canMove) {
            int dx = mouseEvent->pos().x() - lastPoint.x();
            int dy = mouseEvent->pos().y() - lastPoint.y();
            this->move(this->x() + dx, this->y() + dy);
        }
    } else if (type == QEvent::MouseButtonRelease && pressed) {
        pressed = false;
        Q_EMIT clicked();
    }

    return QWidget::eventFilter(watched, event);
}

void LightButton::paintEvent(QPaintEvent *)
{
    int width = this->width();
    int height = this->height();
    int side = qMin(width, height);

    //绘制准备工作,启用反锯齿,平移坐标轴中心,等比例缩放
    QPainter painter(this);
    painter.setRenderHints(QPainter::Antialiasing | QPainter::TextAntialiasing);

    if (showRect) {
        //绘制矩形区域
        painter.setPen(Qt::NoPen);
        painter.setBrush(bgColor);
        painter.drawRoundedRect(this->rect(), 5, 5);

        //绘制文字
        if (!text.isEmpty()) {
            QFont font;
            font.setPixelSize(side - 20);
            painter.setFont(font);
            painter.setPen(textColor);
            painter.drawText(this->rect(), Qt::AlignCenter, text);
        }
    } else {
        painter.translate(width / 2, height / 2);
        painter.scale(side / 200.0, side / 200.0);

        //绘制外边框
        drawBorderOut(&painter);
        //绘制内边框
        drawBorderIn(&painter);
        //绘制内部指示颜色
        drawBg(&painter);
        //绘制居中文字
        drawText(&painter);
        //绘制遮罩层
        drawOverlay(&painter);
    }
}

void LightButton::drawBorderOut(QPainter *painter)
{
    int radius = 99;
    painter->save();
    painter->setPen(Qt::NoPen);
    QLinearGradient borderGradient(0, -radius, 0, radius);
    borderGradient.setColorAt(0, borderOutColorStart);
    borderGradient.setColorAt(1, borderOutColorEnd);
    painter->setBrush(borderGradient);
    painter->drawEllipse(-radius, -radius, radius * 2, radius * 2);
    painter->restore();
}

void LightButton::drawBorderIn(QPainter *painter)
{
    int radius = 90;
    painter->save();
    painter->setPen(Qt::NoPen);
    QLinearGradient borderGradient(0, -radius, 0, radius);
    borderGradient.setColorAt(0, borderInColorStart);
    borderGradient.setColorAt(1, borderInColorEnd);
    painter->setBrush(borderGradient);
    painter->drawEllipse(-radius, -radius, radius * 2, radius * 2);
    painter->restore();
}

void LightButton::drawBg(QPainter *painter)
{
    int radius = 80;
    painter->save();
    painter->setPen(Qt::NoPen);
    painter->setBrush(bgColor);
    painter->drawEllipse(-radius, -radius, radius * 2, radius * 2);
    painter->restore();
}

void LightButton::drawText(QPainter *painter)
{
    if (text.isEmpty()) {
        return;
    }

    int radius = 100;
    painter->save();

    QFont font;
    font.setPixelSize(85);
    painter->setFont(font);
    painter->setPen(textColor);
    QRect rect(-radius, -radius, radius * 2, radius * 2);
    painter->drawText(rect, Qt::AlignCenter, text);
    painter->restore();
}

void LightButton::drawOverlay(QPainter *painter)
{
    if (!showOverlay) {
        return;
    }

    int radius = 80;
    painter->save();
    painter->setPen(Qt::NoPen);

    QPainterPath smallCircle;
    QPainterPath bigCircle;
    radius -= 1;
    smallCircle.addEllipse(-radius, -radius, radius * 2, radius * 2);
    radius *= 2;
    bigCircle.addEllipse(-radius, -radius + 140, radius * 2, radius * 2);

    //高光的形状为小圆扣掉大圆的部分
    QPainterPath highlight = smallCircle - bigCircle;

    QLinearGradient linearGradient(0, -radius / 2, 0, 0);
    overlayColor.setAlpha(100);
    linearGradient.setColorAt(0.0, overlayColor);
    overlayColor.setAlpha(30);
    linearGradient.setColorAt(1.0, overlayColor);
    painter->setBrush(linearGradient);
    painter->rotate(-20);
    painter->drawPath(highlight);

    painter->restore();
}

QSize LightButton::sizeHint() const
{
    return QSize(100, 100);
}

QSize LightButton::minimumSizeHint() const
{
    return QSize(10, 10);
}

QString LightButton::getText() const
{
    return this->text;
}

void LightButton::setText(const QString &text)
{
    if (this->text != text) {
        this->text = text;
        this->update();
    }
}

QColor LightButton::getTextColor() const
{
    return this->textColor;
}

void LightButton::setTextColor(const QColor &textColor)
{
    if (this->textColor != textColor) {
        this->textColor = textColor;
        this->update();
    }
}

QColor LightButton::getAlarmColor() const
{
    return this->alarmColor;
}

void LightButton::setAlarmColor(const QColor &alarmColor)
{
    if (this->alarmColor != alarmColor) {
        this->alarmColor = alarmColor;
        this->update();
    }
}

QColor LightButton::getNormalColor() const
{
    return this->normalColor;
}

void LightButton::setNormalColor(const QColor &normalColor)
{
    if (this->normalColor != normalColor) {
        this->normalColor = normalColor;
        this->update();
    }
}

QColor LightButton::getBorderOutColorStart() const
{
    return this->borderOutColorStart;
}

void LightButton::setBorderOutColorStart(const QColor &borderOutColorStart)
{
    if (this->borderOutColorStart != borderOutColorStart) {
        this->borderOutColorStart = borderOutColorStart;
        this->update();
    }
}

QColor LightButton::getBorderOutColorEnd() const
{
    return this->borderOutColorEnd;
}

void LightButton::setBorderOutColorEnd(const QColor &borderOutColorEnd)
{
    if (this->borderOutColorEnd != borderOutColorEnd) {
        this->borderOutColorEnd = borderOutColorEnd;
        this->update();
    }
}

QColor LightButton::getBorderInColorStart() const
{
    return this->borderInColorStart;
}

void LightButton::setBorderInColorStart(const QColor &borderInColorStart)
{
    if (this->borderInColorStart != borderInColorStart) {
        this->borderInColorStart = borderInColorStart;
        this->update();
    }
}

QColor LightButton::getBorderInColorEnd() const
{
    return this->borderInColorEnd;
}

void LightButton::setBorderInColorEnd(const QColor &borderInColorEnd)
{
    if (this->borderInColorEnd != borderInColorEnd) {
        this->borderInColorEnd = borderInColorEnd;
        this->update();
    }
}

QColor LightButton::getBgColor() const
{
    return this->bgColor;
}

void LightButton::setBgColor(const QColor &bgColor)
{
    if (this->bgColor != bgColor) {
        this->bgColor = bgColor;
        this->update();
    }
}

bool LightButton::getCanMove() const
{
    return this->canMove;
}

void LightButton::setCanMove(bool canMove)
{
    if (this->canMove != canMove) {
        this->canMove = canMove;
        this->update();
    }
}

bool LightButton::getShowRect() const
{
    return this->showRect;
}

void LightButton::setShowRect(bool showRect)
{
    if (this->showRect != showRect) {
        this->showRect = showRect;
        this->update();
    }
}

bool LightButton::getShowOverlay() const
{
    return this->showOverlay;
}

void LightButton::setShowOverlay(bool showOverlay)
{
    if (this->showOverlay != showOverlay) {
        this->showOverlay = showOverlay;
        this->update();
    }
}

QColor LightButton::getOverlayColor() const
{
    return this->overlayColor;
}

void LightButton::setOverlayColor(const QColor &overlayColor)
{
    if (this->overlayColor != overlayColor) {
        this->overlayColor = overlayColor;
        this->update();
    }
}

void LightButton::setGreen()
{
    textColor = QColor(255, 255, 255);
    setBgColor(QColor(0, 166, 0));
}

void LightButton::setRed()
{
    textColor = QColor(255, 255, 255);
    setBgColor(QColor(255, 0, 0));
}

void LightButton::setYellow()
{
    textColor = QColor(25, 50, 7);
    setBgColor(QColor(238, 238, 0));
}

void LightButton::setBlack()
{
    textColor = QColor(255, 255, 255);
    setBgColor(QColor(10, 10, 10));
}

void LightButton::setGray()
{
    textColor = QColor(255, 255, 255);
    setBgColor(QColor(129, 129, 129));
}

void LightButton::setBlue()
{
    textColor = QColor(255, 255, 255);
    setBgColor(QColor(0, 0, 166));
}

void LightButton::setLightBlue()
{
    textColor = QColor(255, 255, 255);
    setBgColor(QColor(100, 184, 255));
}

void LightButton::setLightRed()
{
    textColor = QColor(255, 255, 255);
    setBgColor(QColor(255, 107, 107));
}

void LightButton::setLightGreen()
{
    textColor = QColor(255, 255, 255);
    setBgColor(QColor(24, 189, 155));
}

void LightButton::startAlarm()
{
    if (!timerAlarm->isActive()) {
        timerAlarm->start();
    }
}

void LightButton::stopAlarm()
{
    if (timerAlarm->isActive()) {
        timerAlarm->stop();
    }
}

void LightButton::alarm()
{
    if (isAlarm) {
        textColor = QColor(255, 255, 255);
        bgColor = normalColor;
    } else {
        textColor = QColor(255, 255, 255);
        bgColor = alarmColor;
    }

    this->update();
    isAlarm = !isAlarm;
}
```

### `./lightbutton.h`

```cpp
﻿#ifndef LIGHTBUTTON_H
#define LIGHTBUTTON_H

/**
 * 高亮发光按钮控件 作者:feiyangqingyun(QQ:517216493) 2016-10-16
 * 1. 可设置文本，居中显示。
 * 2. 可设置文本颜色。
 * 3. 可设置外边框渐变颜色。
 * 4. 可设置里边框渐变颜色。
 * 5. 可设置背景色。
 * 6. 可直接调用内置的设置 绿色、红色、黄色、黑色、蓝色 等公有槽函数。
 * 7. 可设置是否在容器中可移动，当成一个对象使用。
 * 8. 可设置是否显示矩形。
 * 9. 可设置报警颜色、非报警颜色。
 * 10. 可控制启动报警和停止报警，报警时闪烁。
 */

#include <QWidget>

#ifdef quc
class Q_DECL_EXPORT LightButton : public QWidget
#else
class LightButton : public QWidget
#endif

{
    Q_OBJECT

    Q_PROPERTY(QString text READ getText WRITE setText)
    Q_PROPERTY(QColor textColor READ getTextColor WRITE setTextColor)
    Q_PROPERTY(QColor alarmColor READ getAlarmColor WRITE setAlarmColor)
    Q_PROPERTY(QColor normalColor READ getNormalColor WRITE setNormalColor)

    Q_PROPERTY(QColor borderOutColorStart READ getBorderOutColorStart WRITE setBorderOutColorStart)
    Q_PROPERTY(QColor borderOutColorEnd READ getBorderOutColorEnd WRITE setBorderOutColorEnd)
    Q_PROPERTY(QColor borderInColorStart READ getBorderInColorStart WRITE setBorderInColorStart)
    Q_PROPERTY(QColor borderInColorEnd READ getBorderInColorEnd WRITE setBorderInColorEnd)
    Q_PROPERTY(QColor bgColor READ getBgColor WRITE setBgColor)

    Q_PROPERTY(bool canMove READ getCanMove WRITE setCanMove)
    Q_PROPERTY(bool showRect READ getShowRect WRITE setShowRect)
    Q_PROPERTY(bool showOverlay READ getShowOverlay WRITE setShowOverlay)
    Q_PROPERTY(QColor overlayColor READ getOverlayColor WRITE setOverlayColor)

public:
    explicit LightButton(QWidget *parent = 0);

protected:
    bool eventFilter(QObject *watched, QEvent *event);
    void paintEvent(QPaintEvent *);
    void drawBorderOut(QPainter *painter);
    void drawBorderIn(QPainter *painter);
    void drawBg(QPainter *painter);
    void drawText(QPainter *painter);
    void drawOverlay(QPainter *painter);

private:
    QString text;               //文本
    QColor textColor;           //文字颜色
    QColor alarmColor;          //报警颜色
    QColor normalColor;         //正常颜色

    QColor borderOutColorStart; //外边框渐变开始颜色
    QColor borderOutColorEnd;   //外边框渐变结束颜色
    QColor borderInColorStart;  //里边框渐变开始颜色
    QColor borderInColorEnd;    //里边框渐变结束颜色
    QColor bgColor;             //背景颜色

    bool showRect;              //显示成矩形
    bool canMove;               //是否能够移动
    bool showOverlay;           //是否显示遮罩层
    QColor overlayColor;        //遮罩层颜色

    bool pressed;               //鼠标是否按下
    QPoint lastPoint;           //鼠标最后按下坐标

    bool isAlarm;               //是否报警
    QTimer *timerAlarm;         //定时器切换颜色

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //获取和设置文本
    QString getText() const;
    void setText(const QString &text);

    //获取和设置文本颜色
    QColor getTextColor() const;
    void setTextColor(const QColor &textColor);

    //获取和设置报警颜色
    QColor getAlarmColor() const;
    void setAlarmColor(const QColor &alarmColor);

    //获取和设置正常颜色
    QColor getNormalColor() const;
    void setNormalColor(const QColor &normalColor);

    //获取和设置外边框渐变颜色
    QColor getBorderOutColorStart() const;
    void setBorderOutColorStart(const QColor &borderOutColorStart);

    QColor getBorderOutColorEnd() const;
    void setBorderOutColorEnd(const QColor &borderOutColorEnd);

    //获取和设置里边框渐变颜色
    QColor getBorderInColorStart() const;
    void setBorderInColorStart(const QColor &borderInColorStart);

    QColor getBorderInColorEnd() const;
    void setBorderInColorEnd(const QColor &borderInColorEnd);

    //获取和设置背景色
    QColor getBgColor() const;
    void setBgColor(const QColor &bgColor);

    //获取和设置是否可移动
    bool getCanMove() const;
    void setCanMove(bool canMove);

    //获取和设置是否显示矩形
    bool getShowRect() const;
    void setShowRect(bool showRect);

    //获取和设置是否显示遮罩层
    bool getShowOverlay() const;
    void setShowOverlay(bool showOverlay);

    //获取和设置遮罩层颜色
    QColor getOverlayColor() const;
    void setOverlayColor(const QColor &overlayColor);

public Q_SLOTS:
    //设置为绿色
    void setGreen();
    //设置为红色
    void setRed();
    //设置为黄色
    void setYellow();
    //设置为黑色
    void setBlack();
    //设置为灰色
    void setGray();
    //设置为蓝色
    void setBlue();

    //设置为淡蓝色
    void setLightBlue();
    //设置为淡红色
    void setLightRed();
    //设置为淡绿色
    void setLightGreen();

    //设置报警闪烁
    void startAlarm();
    void stopAlarm();
    void alarm();

Q_SIGNALS:
    //单击信号
    void clicked();
};

#endif // LIGHTBUTTON_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmlightbutton.h"
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

    frmLightButton w;
    w.setWindowTitle("高亮发光按钮 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmlightbutton.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmLightButton</class>
 <widget class="QWidget" name="frmLightButton">
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
  <layout class="QHBoxLayout" name="horizontalLayout">
   <item>
    <widget class="LightButton" name="lightButton1"/>
   </item>
   <item>
    <widget class="LightButton" name="lightButton2"/>
   </item>
   <item>
    <widget class="LightButton" name="lightButton3"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>LightButton</class>
   <extends>QWidget</extends>
   <header>lightbutton.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
