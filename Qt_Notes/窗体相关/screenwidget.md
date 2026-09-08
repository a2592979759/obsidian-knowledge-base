---
tags:
  - Qt
  - 窗体相关
---
# 屏幕截图 (`screenwidget`)

> 屏幕截图

## 效果图

![[QWidgetDemo_assert/screenwidget.jpg]]

## 分类

- 类型: 窗体相关 `widget`
- 源码目录: `widget/screenwidget`

## 技术要点

**用到的 Qt 类**: QPoint QWidget QTextCodec QString QPixmap QMouseEvent QApplication QSize QMenu QPainter QScopedPointer QShowEvent QPaintEvent QContextMenuEvent QFileDialog QPushButton QFont QDateTime QMutex QMutexLocker

**特性/模块**: Q_OBJECT

## 完整源码

### `./frmscreenwidget.cpp`

```cpp
﻿#include "frmscreenwidget.h"
#include "ui_frmscreenwidget.h"
#include "screenwidget.h"

frmScreenWidget::frmScreenWidget(QWidget *parent) : QWidget(parent), ui(new Ui::frmScreenWidget)
{
    ui->setupUi(this);
}

frmScreenWidget::~frmScreenWidget()
{
    delete ui;
    exit(0);
}

void frmScreenWidget::on_pushButton_clicked()
{
    ScreenWidget::Instance()->showFullScreen();
}
```

### `./frmscreenwidget.h`

```cpp
﻿#ifndef FRMSCREENWIDGET_H
#define FRMSCREENWIDGET_H

#include <QWidget>

namespace Ui {
class frmScreenWidget;
}

class frmScreenWidget : public QWidget
{
    Q_OBJECT

public:
    explicit frmScreenWidget(QWidget *parent = 0);
    ~frmScreenWidget();

private slots:
    void on_pushButton_clicked();

private:
    Ui::frmScreenWidget *ui;
};

#endif // FRMSCREENWIDGET_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmscreenwidget.h"
#include <QApplication>
#include <QTextCodec>

int main(int argc, char *argv[])
{
    //qputenv("QT_FONT_DPI", "96");
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

    frmScreenWidget w;
    w.setWindowTitle("屏幕截图 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./screenwidget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "screenwidget.h"
#include "qmutex.h"
#include "qapplication.h"
#include "qpainter.h"
#include "qfiledialog.h"
#include "qevent.h"
#include "qdatetime.h"
#include "qstringlist.h"
#include "qdebug.h"

#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
#include "qscreen.h"
#define deskGeometry qApp->primaryScreen()->geometry()
#define deskGeometry2 qApp->primaryScreen()->availableGeometry()
#else
#include "qdesktopwidget.h"
#define deskGeometry qApp->desktop()->geometry()
#define deskGeometry2 qApp->desktop()->availableGeometry()
#endif

#define STRDATETIME qPrintable (QDateTime::currentDateTime().toString("yyyy-MM-dd-HH-mm-ss"))

Screen::Screen(QSize size)
{
    maxWidth = size.width();
    maxHeight = size.height();

    startPos = QPoint(-1, -1);
    endPos = startPos;
    leftUpPos = startPos;
    rightDownPos = startPos;
    status = SELECT;
}

int Screen::width()
{
    return maxWidth;
}

int Screen::height()
{
    return maxHeight;
}

Screen::STATUS Screen::getStatus()
{
    return status;
}

void Screen::setStatus(STATUS status)
{
    this->status = status;
}

void Screen::setEnd(QPoint pos)
{
    endPos = pos;
    leftUpPos = startPos;
    rightDownPos = endPos;
    cmpPoint(leftUpPos, rightDownPos);
}

void Screen::setStart(QPoint pos)
{
    startPos = pos;
}

QPoint Screen::getEnd()
{
    return endPos;
}

QPoint Screen::getStart()
{
    return startPos;
}

QPoint Screen::getLeftUp()
{
    return leftUpPos;
}

QPoint Screen::getRightDown()
{
    return rightDownPos;
}

bool Screen::isInArea(QPoint pos)
{
    if (pos.x() > leftUpPos.x() && pos.x() < rightDownPos.x() && pos.y() > leftUpPos.y() && pos.y() < rightDownPos.y()) {
        return true;
    }

    return false;
}

void Screen::move(QPoint p)
{
    int lx = leftUpPos.x() + p.x();
    int ly = leftUpPos.y() + p.y();
    int rx = rightDownPos.x() + p.x();
    int ry = rightDownPos.y() + p.y();

    if (lx < 0) {
        lx = 0;
        rx -= p.x();
    }

    if (ly < 0) {
        ly = 0;
        ry -= p.y();
    }

    if (rx > maxWidth)  {
        rx = maxWidth;
        lx -= p.x();
    }

    if (ry > maxHeight) {
        ry = maxHeight;
        ly -= p.y();
    }

    leftUpPos = QPoint(lx, ly);
    rightDownPos = QPoint(rx, ry);
    startPos = leftUpPos;
    endPos = rightDownPos;
}

void Screen::cmpPoint(QPoint &leftTop, QPoint &rightDown)
{
    QPoint l = leftTop;
    QPoint r = rightDown;

    if (l.x() <= r.x()) {
        if (l.y() <= r.y()) {
        } else {
            leftTop.setY(r.y());
            rightDown.setY(l.y());
        }
    } else {
        if (l.y() < r.y()) {
            leftTop.setX(r.x());
            rightDown.setX(l.x());
        } else {
            QPoint tmp;
            tmp = leftTop;
            leftTop = rightDown;
            rightDown = tmp;
        }
    }
}

QScopedPointer<ScreenWidget> ScreenWidget::self;
ScreenWidget *ScreenWidget::Instance()
{
    if (self.isNull()) {
        static QMutex mutex;
        QMutexLocker locker(&mutex);
        if (self.isNull()) {
            self.reset(new ScreenWidget);
        }
    }

    return self.data();
}

ScreenWidget::ScreenWidget(QWidget *parent) : QWidget(parent)
{
    //this->setWindowFlags(Qt::Tool | Qt::WindowStaysOnTopHint | Qt::FramelessWindowHint | Qt::X11BypassWindowManagerHint);

    menu = new QMenu(this);
    menu->addAction("保存当前截图", this, SLOT(saveScreen()));
    menu->addAction("保存全屏截图", this, SLOT(saveFullScreen()));
    menu->addAction("截图另存为", this, SLOT(saveScreenOther()));
    menu->addAction("全屏另存为", this, SLOT(saveFullOther()));
    menu->addAction("退出截图", this, SLOT(hide()));

    //取得屏幕大小
    screen = new Screen(deskGeometry.size());
    //保存全屏图像
    fullScreen = new QPixmap();
}

QPixmap ScreenWidget::getSelectPixmap(int *rectX, int *rectY, int *rectW, int *rectH, int *pixX, int *pixY)
{
    int x = screen->getLeftUp().x();
    int y = screen->getLeftUp().y();
    int w = screen->getRightDown().x() - x;
    int h = screen->getRightDown().y() - y;

    qreal dpi = 1.0;
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    dpi = this->devicePixelRatioF();
#endif

    int x2 = x * dpi;
    int y2 = y * dpi;
    int w2 = w * dpi;
    int h2 = h * dpi;

    if (rectX) {
        (*rectX) = x;
    }
    if (rectY) {
        (*rectY) = y;
    }
    if (rectW) {
        (*rectW) = w;
    }
    if (rectH) {
        (*rectH) = h;
    }
    if (pixX) {
        (*pixX) = x2;
    }
    if (pixY) {
        (*pixY) = y2;
    }

    return fullScreen->copy(x2, y2, w2, h2);
}

void ScreenWidget::showEvent(QShowEvent *)
{
    QPoint point(-1, -1);
    screen->setStart(point);
    screen->setEnd(point);

#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    QScreen *pscreen = QApplication::primaryScreen();
    *fullScreen = pscreen->grabWindow(0, 0, 0, screen->width(), screen->height());
#else
    *fullScreen = fullScreen->grabWindow(0, 0, 0, screen->width(), screen->height());
#endif

    //设置透明度实现模糊背景
    QPixmap pix(screen->width(), screen->height());
    pix.fill((QColor(160, 160, 160, 200)));
    bgScreen = new QPixmap(*fullScreen);
    QPainter p(bgScreen);
    p.drawPixmap(0, 0, pix);
}

void ScreenWidget::paintEvent(QPaintEvent *)
{
    QPen pen;
    pen.setColor(Qt::green);
    pen.setWidth(2);
    pen.setStyle(Qt::DotLine);

    QPainter painter(this);
    painter.setPen(pen);
    painter.drawPixmap(0, 0, *bgScreen);

    int x, y, w, h, x2, y2;
    QPixmap pix = this->getSelectPixmap(&x, &y, &w, &h, &x2, &y2);
    painter.drawRect(x, y, w, h);
    if (w > 1 && h > 1) {
        painter.drawPixmap(x, y, pix);
    }

    int pixW = pix.width();
    int pixH = pix.height();
    pen.setColor(Qt::yellow);
    painter.setPen(pen);
    painter.drawText(x + 2, y - 8, QString("截图范围：( %1 x %2 ) - ( %3 x %4 )  图片大小：( %5 x %6 )")
                     .arg(x2).arg(y2).arg(x2 + pixW).arg(y2 + pixH).arg(pixW).arg(pixH));
}

void ScreenWidget::contextMenuEvent(QContextMenuEvent *)
{
    this->setCursor(Qt::ArrowCursor);
    menu->exec(cursor().pos());
}

void ScreenWidget::mousePressEvent(QMouseEvent *e)
{
    int status = screen->getStatus();
    if (status == Screen::SELECT) {
        screen->setStart(e->pos());
    } else if (status == Screen::MOV) {
        if (screen->isInArea(e->pos()) == false) {
            screen->setStart(e->pos());
            screen->setStatus(Screen::SELECT);
        } else {
            movPos = e->pos();
            this->setCursor(Qt::SizeAllCursor);
        }
    }

    this->update();
}

void ScreenWidget::mouseMoveEvent(QMouseEvent *e)
{
    if (screen->getStatus() == Screen::SELECT) {
        screen->setEnd(e->pos());
    } else if (screen->getStatus() == Screen::MOV) {
        QPoint p(e->x() - movPos.x(), e->y() - movPos.y());
        screen->move(p);
        movPos = e->pos();
    }

    this->update();
}

void ScreenWidget::mouseReleaseEvent(QMouseEvent *)
{
    if (screen->getStatus() == Screen::SELECT) {
        screen->setStatus(Screen::MOV);
    } else if (screen->getStatus() == Screen::MOV) {
        this->setCursor(Qt::ArrowCursor);
    }
}

void ScreenWidget::saveScreen()
{
    QString fileName = QString("%1/screen_%2.png").arg(qApp->applicationDirPath()).arg(STRDATETIME);
    this->getSelectPixmap().save(fileName, "png");
    this->close();
}

void ScreenWidget::saveFullScreen()
{
    QString fileName = QString("%1/full_%2.png").arg(qApp->applicationDirPath()).arg(STRDATETIME);
    fullScreen->save(fileName, "png");
    this->close();
}

void ScreenWidget::saveScreenOther()
{
    QString name = QString("%1.png").arg(STRDATETIME);
    QString fileName = QFileDialog::getSaveFileName(this, "保存图片", name, "png Files (*.png)");
    if (!fileName.endsWith(".png")) {
        fileName += ".png";
    }

    if (!fileName.isEmpty()) {
        this->getSelectPixmap().save(fileName, "png");
        this->close();
    }
}

void ScreenWidget::saveFullOther()
{
    QString name = QString("%1.png").arg(STRDATETIME);
    QString fileName = QFileDialog::getSaveFileName(this, "保存图片", name, "png Files (*.png)");
    if (!fileName.endsWith(".png")) {
        fileName += ".png";
    }

    if (!fileName.isEmpty()) {
        fullScreen->save(fileName, "png");
        this->close();
    }
}
```

### `./screenwidget.h`

```cpp
﻿#ifndef SCREENWIDGET_H
#define SCREENWIDGET_H

/**
 * 全局截屏控件 作者:feiyangqingyun(QQ:517216493) 2016-11-11
 * 1. 鼠标右键弹出菜单。
 * 2. 支持全局截屏。
 * 3. 支持局部截屏。
 * 4. 支持截图区域拖动。
 * 5. 支持图片另存为。
 */

#include <QWidget>
#include <QMenu>
#include <QPoint>
#include <QSize>

class Screen
{
public:
    enum STATUS {SELECT, MOV, SET_W_H};
    Screen() {}
    Screen(QSize size);

    void setStart(QPoint pos);
    void setEnd(QPoint pos);
    QPoint getStart();
    QPoint getEnd();

    QPoint getLeftUp();
    QPoint getRightDown();

    STATUS getStatus();
    void setStatus(STATUS status);

    int width();
    int height();

    //检测坐标点是否在截图区域内
    bool isInArea(QPoint pos);
    //按坐标移动截图区域
    void move(QPoint p);

private:
    //记录 截图区域 左上角、右下角
    QPoint leftUpPos, rightDownPos;
    //记录 鼠标开始位置、结束位置
    QPoint startPos, endPos;
    //记录屏幕大小
    int maxWidth, maxHeight;
    //三个状态: 选择区域、移动区域、设置width height
    STATUS status;

    //比较两位置，判断左上角、右下角
    void cmpPoint(QPoint &s, QPoint &e);
};

#ifdef quc
class Q_DECL_EXPORT ScreenWidget : public QWidget
#else
class ScreenWidget : public QWidget
#endif

{
    Q_OBJECT
public:
    static ScreenWidget *Instance();
    explicit ScreenWidget(QWidget *parent = 0);

private:
    static QScopedPointer<ScreenWidget> self;

    //右键菜单
    QMenu *menu;
    //截屏对象
    Screen *screen;
    //全屏图像
    QPixmap *fullScreen;
    //模糊背景
    QPixmap *bgScreen;
    //移动坐标
    QPoint movPos;

    //获取选中图像
    QPixmap getSelectPixmap(int *rectX = NULL, int *rectY = NULL, int *rectW = NULL, int *rectH = NULL, int *pixX = NULL, int *pixY = NULL);

protected:
    void showEvent(QShowEvent *);
    void paintEvent(QPaintEvent *);
    void contextMenuEvent(QContextMenuEvent *);

    void mousePressEvent(QMouseEvent *);
    void mouseMoveEvent(QMouseEvent *);
    void mouseReleaseEvent(QMouseEvent *);

private slots:
    void saveScreen();
    void saveFullScreen();
    void saveScreenOther();
    void saveFullOther();
};

#endif // SCREENWIDGET_H
```

### `./frmscreenwidget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmScreenWidget</class>
 <widget class="QWidget" name="frmScreenWidget">
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
  <widget class="QPushButton" name="pushButton">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>10</y>
     <width>92</width>
     <height>28</height>
    </rect>
   </property>
   <property name="text">
    <string>弹出</string>
   </property>
  </widget>
 </widget>
 <resources/>
 <connections/>
</ui>
```
