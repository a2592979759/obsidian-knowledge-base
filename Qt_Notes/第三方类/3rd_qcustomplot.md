---
tags:
  - Qt
  - 第三方类
---
# QCustomPlot 图表库 (`3rd_qcustomplot`)

## 分类

- 类型: 第三方类 `third`
- 源码目录: `third/3rd_qcustomplot`

## 技术要点

**用到的 Qt 类**: QPointF QVector QPainterPath QPainter QObject

## 完整源码

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

### `./3rd_qcustomplot.pri`

```makefile
greaterThan(QT_MAJOR_VERSION, 4): QT += printsupport
greaterThan(QT_MAJOR_VERSION, 4): CONFIG += c++11
#lessThan(QT_MAJOR_VERSION, 5): QMAKE_CXXFLAGS += -std=c++11

#下面用于开启opengl
#DEFINES += QCUSTOMPLOT_USE_OPENGL
#LIBS += -lopengl32 -lglu32

#将当前目录加入到头文件路径
INCLUDEPATH += $$PWD
DEFINES += qcustomplot

#引入平滑曲线类
HEADERS += $$PWD/smoothcurve.h
SOURCES += $$PWD/smoothcurve.cpp

#没有定义任何版本则默认采用2.0
!contains(DEFINES, qcustomplot_v1_3) {
!contains(DEFINES, qcustomplot_v2_0) {
!contains(DEFINES, qcustomplot_v2_1) {
DEFINES += qcustomplot_v2_0
}}}

#定义了2.0版本在Qt5以上采用2.1
contains(DEFINES, qcustomplot_v2_0) {
!contains(DEFINES, qcustomplot_v2_1) {
greaterThan(QT_MAJOR_VERSION, 4) {
DEFINES -= qcustomplot_v1_3
DEFINES -= qcustomplot_v2_0
DEFINES += qcustomplot_v2_1
}}}

#根据定义的版本引入文件
contains(DEFINES, qcustomplot_v1_3) {
INCLUDEPATH += $$PWD/v1_3
HEADERS += $$PWD/v1_3/qcustomplot.h
SOURCES += $$PWD/v1_3/qcustomplot.cpp
} else {
contains(DEFINES, qcustomplot_v2_0) {
INCLUDEPATH += $$PWD/v2_0
HEADERS += $$PWD/v2_0/qcustomplot.h
SOURCES += $$PWD/v2_0/qcustomplot.cpp
} else {
INCLUDEPATH += $$PWD/v2_1
#引入对应修复不支持Qt6的头文件
greaterThan(QT_MAJOR_VERSION, 5) {
HEADERS += $$PWD/v2_1_6/qcustomplot.h
} else {
HEADERS += $$PWD/v2_1/qcustomplot.h
}
SOURCES += $$PWD/v2_1/qcustomplot.cpp
}}

#修复debug套件编译失败
greaterThan(QT_MAJOR_VERSION, 5) {
mingw {
QMAKE_CXXFLAGS += -Wa,-mbig-obj
}
}
```
