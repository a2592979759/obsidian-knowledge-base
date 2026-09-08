---
tags:
  - Qt
  - 控件相关
---
# 平滑曲线 (`smoothcurve`)

> 平滑曲线

## 效果图

![[QWidgetDemo_assert/smoothcurve.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/smoothcurve`

## 技术要点

**用到的 Qt 类**: QPointF QVector QPainterPath QTextCodec QWidget QPainter QRadioButton QPaintEvent QColor QApplication QTime QDateTime QPen QList QGridLayout QCheckBox QFont QObject

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmsmoothcurve.cpp`

```cpp
﻿#include "frmsmoothcurve.h"
#include "ui_frmsmoothcurve.h"
#include "smoothcurve.h"
#include "qpainter.h"
#include "qdatetime.h"
#include "qdebug.h"

#define TIMEMS QTime::currentTime().toString("hh:mm:ss zzz")

frmSmoothCurve::frmSmoothCurve(QWidget *parent) : QWidget(parent), ui(new Ui::frmSmoothCurve)
{
    ui->setupUi(this);

    //初始化随机数种子
    srand(QDateTime::currentDateTime().toMSecsSinceEpoch());

    //随机生成曲线上的点
    int x = -300;
    while (x < 300) {
        datas << QPointF(x, rand() % 300 - 100);
        x += qMin(rand() % 30 + 5, 300);
    }

    //正常曲线
    pathNormal.moveTo(datas.at(0));
    for (int i = 1; i < datas.size(); ++i) {
        pathNormal.lineTo(datas.at(i));
    }

    //平滑曲线1
    //qDebug() << TIMEMS << "createSmoothCurve start";
    pathSmooth1 = SmoothCurve::createSmoothCurve(datas);
    //qDebug() << TIMEMS << "createSmoothCurve stop";

    //平滑曲线2
    //qDebug() << TIMEMS << "createSmoothCurve2 start";
    pathSmooth2 = SmoothCurve::createSmoothCurve2(datas);
    //qDebug() << TIMEMS << "createSmoothCurve2 stop";

    ui->ckShowPoint->setChecked(true);
    connect(ui->ckShowPoint, SIGNAL(clicked(bool)), this, SLOT(update()));
    connect(ui->rbtnPathNormal, SIGNAL(clicked(bool)), this, SLOT(update()));
    connect(ui->rbtnPathSmooth1, SIGNAL(clicked(bool)), this, SLOT(update()));
    connect(ui->rbtnPathSmooth2, SIGNAL(clicked(bool)), this, SLOT(update()));
}

frmSmoothCurve::~frmSmoothCurve()
{
    delete ui;
}

void frmSmoothCurve::paintEvent(QPaintEvent *)
{
    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);
    painter.translate(width() / 2, height() / 2);
    painter.scale(1, -1);

    //画坐标轴
    painter.setPen(QColor(180, 180, 180));
    painter.drawLine(-250, 0, 250, 0);
    painter.drawLine(0, 150, 0, -150);

    //根据选择绘制不同的曲线路径
    painter.setPen(QPen(QColor(80, 80, 80), 2));
    if (ui->rbtnPathSmooth1->isChecked()) {
        painter.drawPath(pathSmooth1);
    } else if (ui->rbtnPathSmooth2->isChecked()) {
        painter.drawPath(pathSmooth2);
    } else {
        painter.drawPath(pathNormal);
    }

    //如果曲线上的点可见则显示出来
    if (ui->ckShowPoint->isChecked()) {
        painter.setPen(Qt::black);
        painter.setBrush(Qt::gray);
        foreach (QPointF point, datas) {
            painter.drawEllipse(point, 3, 3);
        }
    }
}
```

### `./frmsmoothcurve.h`

```cpp
﻿#ifndef FRMSMOOTHCURVE_H
#define FRMSMOOTHCURVE_H

#include <QWidget>
#include <QList>
#include <QPointF>
#include <QPainterPath>

namespace Ui {
class frmSmoothCurve;
}

class frmSmoothCurve : public QWidget
{
    Q_OBJECT

public:
    explicit frmSmoothCurve(QWidget *parent = 0);
    ~frmSmoothCurve();

protected:
    void paintEvent(QPaintEvent *event);

private:
    Ui::frmSmoothCurve *ui;
    QVector<QPointF> datas;     //曲线上的点
    QPainterPath pathNormal;    //正常曲线
    QPainterPath pathSmooth1;   //平滑曲线1
    QPainterPath pathSmooth2;   //平滑曲线2
};

#endif // FRMSMOOTHCURVE_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmsmoothcurve.h"
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

    frmSmoothCurve w;
    w.setWindowTitle("平滑曲线 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./smoothcurve.cpp`

```cpp
﻿#include "smoothcurve.h"
#include "qdebug.h"

QPainterPath SmoothCurve::createSmoothCurve(const QVector<QPointF> &points)
{
    QPainterPath path;
    int len = points.count();
    if (len < 2) {
        return path;
    }

    QVector<QPointF> firstControlPoints;
    QVector<QPointF> secondControlPoints;
    calculateControlPoints(points, &firstControlPoints, &secondControlPoints);
    path.moveTo(points[0].x(), points[0].y());

    for (int i = 0; i < len - 1; ++i) {
        path.cubicTo(firstControlPoints[i], secondControlPoints[i], points[i + 1]);
    }

    return path;
}

QPainterPath SmoothCurve::createSmoothCurve2(const QVector<QPointF> &points)
{
    //采用Qt原生方法不做任何处理
    int count = points.count();
    if (count == 0) {
        return QPainterPath();
    }

    QPainterPath path(points.at(0));
    for (int i = 0; i < count - 1; ++i) {
        //控制点的 x 坐标为 sp 与 ep 的 x 坐标和的一半
        //第一个控制点 c1 的 y 坐标为起始点 sp 的 y 坐标
        //第二个控制点 c2 的 y 坐标为结束点 ep 的 y 坐标
        QPointF sp = points.at(i);
        QPointF ep = points.at(i + 1);
        QPointF c1 = QPointF((sp.x() + ep.x()) / 2, sp.y());
        QPointF c2 = QPointF((sp.x() + ep.x()) / 2, ep.y());
        path.cubicTo(c1, c2, ep);
    }

    return path;
}

void SmoothCurve::calculateFirstControlPoints(double *&result, const double *rhs, int n)
{
    result = new double[n];
    double *tmp = new double[n];
    double b = 2.0;
    result[0] = rhs[0] / b;

    for (int i = 1; i < n; ++i) {
        tmp[i] = 1 / b;
        b = (i < n - 1 ? 4.0 : 3.5) - tmp[i];
        result[i] = (rhs[i] - result[i - 1]) / b;
    }

    for (int i = 1; i < n; ++i) {
        result[n - i - 1] -= tmp[n - i] * result[n - i];
    }

    delete tmp;
}

void SmoothCurve::calculateControlPoints(const QVector<QPointF> &datas,
                                         QVector<QPointF> *firstControlPoints,
                                         QVector<QPointF> *secondControlPoints)
{
    int n = datas.count() - 1;
    for (int i = 0; i < n; ++i) {
        firstControlPoints->append(QPointF());
        secondControlPoints->append(QPointF());
    }

    if (n == 1) {
        (*firstControlPoints)[0].rx() = (2 * datas[0].x() + datas[1].x()) / 3;
        (*firstControlPoints)[0].ry() = (2 * datas[0].y() + datas[1].y()) / 3;
        (*secondControlPoints)[0].rx() = 2 * (*firstControlPoints)[0].x() - datas[0].x();
        (*secondControlPoints)[0].ry() = 2 * (*firstControlPoints)[0].y() - datas[0].y();
        return;
    }

    double *xs = 0;
    double *ys = 0;
    double *rhsx = new double[n];
    double *rhsy = new double[n];

    for (int i = 1; i < n - 1; ++i) {
        rhsx[i] = 4 * datas[i].x() + 2 * datas[i + 1].x();
        rhsy[i] = 4 * datas[i].y() + 2 * datas[i + 1].y();
    }

    rhsx[0] = datas[0].x() + 2 * datas[1].x();
    rhsx[n - 1] = (8 * datas[n - 1].x() + datas[n].x()) / 2.0;
    rhsy[0] = datas[0].y() + 2 * datas[1].y();
    rhsy[n - 1] = (8 * datas[n - 1].y() + datas[n].y()) / 2.0;

    calculateFirstControlPoints(xs, rhsx, n);
    calculateFirstControlPoints(ys, rhsy, n);

    for (int i = 0; i < n; ++i) {
        (*firstControlPoints)[i].rx() = xs[i];
        (*firstControlPoints)[i].ry() = ys[i];

        if (i < n - 1) {
            (*secondControlPoints)[i].rx() = 2 * datas[i + 1].x() - xs[i + 1];
            (*secondControlPoints)[i].ry() = 2 * datas[i + 1].y() - ys[i + 1];
        } else {
            (*secondControlPoints)[i].rx() = (datas[n].x() + xs[n - 1]) / 2;
            (*secondControlPoints)[i].ry() = (datas[n].y() + ys[n - 1]) / 2;
        }
    }

    delete xs;
    delete ys;
    delete rhsx;
    delete rhsy;
}
```

### `./smoothcurve.h`

```cpp
﻿#ifndef SMOOTHCURVE_H
#define SMOOTHCURVE_H

#include <QObject>
#include <QVector>
#include <QPointF>
#include <QPainterPath>

#ifdef quc
class Q_DECL_EXPORT SmoothCurve
#else
class SmoothCurve
#endif

{
public:
    //创建平滑曲线路径
    static QPainterPath createSmoothCurve(const QVector<QPointF> &points);
    static QPainterPath createSmoothCurve2(const QVector<QPointF> &points);

private:
    static void calculateFirstControlPoints(double *&result, const double *rhs, int n);
    static void calculateControlPoints(const QVector<QPointF> &datas,
                                       QVector<QPointF> *firstControlPoints,
                                       QVector<QPointF> *secondControlPoints);
};

#endif // SMOOTHCURVE_H
```

### `./frmsmoothcurve.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmSmoothCurve</class>
 <widget class="QWidget" name="frmSmoothCurve">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Widget</string>
  </property>
  <layout class="QGridLayout" name="gridLayout">
   <item row="0" column="5">
    <spacer name="verticalSpacer">
     <property name="orientation">
      <enum>Qt::Vertical</enum>
     </property>
     <property name="sizeHint" stdset="0">
      <size>
       <width>20</width>
       <height>473</height>
      </size>
     </property>
    </spacer>
   </item>
   <item row="1" column="2">
    <widget class="QRadioButton" name="rbtnPathSmooth1">
     <property name="text">
      <string>平滑曲线1</string>
     </property>
    </widget>
   </item>
   <item row="1" column="3">
    <widget class="QRadioButton" name="rbtnPathSmooth2">
     <property name="text">
      <string>平滑曲线2</string>
     </property>
    </widget>
   </item>
   <item row="1" column="0">
    <widget class="QCheckBox" name="ckShowPoint">
     <property name="text">
      <string>显示坐标点</string>
     </property>
    </widget>
   </item>
   <item row="1" column="4">
    <spacer name="horizontalSpacer">
     <property name="orientation">
      <enum>Qt::Horizontal</enum>
     </property>
     <property name="sizeHint" stdset="0">
      <size>
       <width>549</width>
       <height>20</height>
      </size>
     </property>
    </spacer>
   </item>
   <item row="1" column="1">
    <widget class="QRadioButton" name="rbtnPathNormal">
     <property name="text">
      <string>正常曲线</string>
     </property>
     <property name="checked">
      <bool>true</bool>
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
