---
tags:
  - Qt
  - 窗体相关
  - 高质量
---
# 通用无边框窗体 (`framelesswidget`)

> 通用无边框窗体

## 效果图

![[QWidgetDemo_assert/framelesswidget.jpg]]

## 分类

- 类型: 窗体相关 `widget`
- 源码目录: `widget/framelesswidget`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QWidget QEvent QRect QPoint QPushButton QObject QByteArray QTextCodec QDialog QHoverEvent QMouseEvent QList QHBoxLayout QMainWindow QShowEvent QCheckBox QApplication QTime QStringList QVBoxLayout

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./framelesswidget2.cpp`

```cpp
﻿#include "framelesswidget2.h"
#include "qapplication.h"
#include "qevent.h"
#include "qdebug.h"

FramelessWidget2::FramelessWidget2(QObject *parent) : QObject(parent)
{
    padding = 8;
    moveEnable = true;
    resizeEnable = true;
    widget = NULL;

    mousePressed = false;
    mousePoint = QPoint(0, 0);
    mouseRect = QRect(0, 0, 0, 0);

    for (int i = 0; i < 8; ++i) {
        pressedArea << false;
        pressedRect << QRect(0, 0, 0, 0);
    }

    //如果父类是窗体则直接设置
    if (parent->isWidgetType()) {
        setWidget((QWidget *)parent);
    }
}

bool FramelessWidget2::eventFilter(QObject *watched, QEvent *event)
{
    if (widget && watched == widget) {
        int type = event->type();
        if (type == QEvent::WindowStateChange) {
            //解决mac系统上无边框最小化失效的bug
#ifdef Q_OS_MACOS
            if (widget->windowState() & Qt::WindowMinimized) {
                isMin = true;
            } else {
                if (isMin) {
                    //设置无边框属性
                    widget->setWindowFlags(flags | Qt::FramelessWindowHint);
                    widget->setVisible(true);
                    isMin = false;
                }
            }
#endif
        } else if (type == QEvent::Resize) {
            //重新计算八个描点的区域,描点区域的作用还有就是计算鼠标坐标是否在某一个区域内
            int width = widget->width();
            int height = widget->height();

            //左侧描点区域
            pressedRect[0] = QRect(0, padding, padding, height - padding * 2);
            //右侧描点区域
            pressedRect[1] = QRect(width - padding, padding, padding, height - padding * 2);
            //上侧描点区域
            pressedRect[2] = QRect(padding, 0, width - padding * 2, padding);
            //下侧描点区域
            pressedRect[3] = QRect(padding, height - padding, width - padding * 2, padding);

            //左上角描点区域
            pressedRect[4] = QRect(0, 0, padding, padding);
            //右上角描点区域
            pressedRect[5] = QRect(width - padding, 0, padding, padding);
            //左下角描点区域
            pressedRect[6] = QRect(0, height - padding, padding, padding);
            //右下角描点区域
            pressedRect[7] = QRect(width - padding, height - padding, padding, padding);
        } else if (type == QEvent::HoverMove) {
            //设置对应鼠标形状,这个必须放在这里而不是下面,因为可以在鼠标没有按下的时候识别
            QHoverEvent *hoverEvent = (QHoverEvent *)event;
            QPoint point = hoverEvent->pos();
            if (resizeEnable) {
                if (pressedRect.at(0).contains(point)) {
                    widget->setCursor(Qt::SizeHorCursor);
                } else if (pressedRect.at(1).contains(point)) {
                    widget->setCursor(Qt::SizeHorCursor);
                } else if (pressedRect.at(2).contains(point)) {
                    widget->setCursor(Qt::SizeVerCursor);
                } else if (pressedRect.at(3).contains(point)) {
                    widget->setCursor(Qt::SizeVerCursor);
                } else if (pressedRect.at(4).contains(point)) {
                    widget->setCursor(Qt::SizeFDiagCursor);
                } else if (pressedRect.at(5).contains(point)) {
                    widget->setCursor(Qt::SizeBDiagCursor);
                } else if (pressedRect.at(6).contains(point)) {
                    widget->setCursor(Qt::SizeBDiagCursor);
                } else if (pressedRect.at(7).contains(point)) {
                    widget->setCursor(Qt::SizeFDiagCursor);
                } else {
                    widget->setCursor(Qt::ArrowCursor);
                }
            }

            //根据当前鼠标位置,计算XY轴移动了多少
            int offsetX = point.x() - mousePoint.x();
            int offsetY = point.y() - mousePoint.y();

            //根据按下处的位置判断是否是移动控件还是拉伸控件
            if (moveEnable && mousePressed) {
                widget->move(widget->x() + offsetX, widget->y() + offsetY);
            }

            if (resizeEnable) {
                int rectX = mouseRect.x();
                int rectY = mouseRect.y();
                int rectW = mouseRect.width();
                int rectH = mouseRect.height();

                if (pressedArea.at(0)) {
                    int resizeW = widget->width() - offsetX;
                    if (widget->minimumWidth() <= resizeW) {
                        widget->setGeometry(widget->x() + offsetX, rectY, resizeW, rectH);
                    }
                } else if (pressedArea.at(1)) {
                    widget->setGeometry(rectX, rectY, rectW + offsetX, rectH);
                } else if (pressedArea.at(2)) {
                    int resizeH = widget->height() - offsetY;
                    if (widget->minimumHeight() <= resizeH) {
                        widget->setGeometry(rectX, widget->y() + offsetY, rectW, resizeH);
                    }
                } else if (pressedArea.at(3)) {
                    widget->setGeometry(rectX, rectY, rectW, rectH + offsetY);
                } else if (pressedArea.at(4)) {
                    int resizeW = widget->width() - offsetX;
                    int resizeH = widget->height() - offsetY;
                    if (widget->minimumWidth() <= resizeW) {
                        widget->setGeometry(widget->x() + offsetX, widget->y(), resizeW, resizeH);
                    }
                    if (widget->minimumHeight() <= resizeH) {
                        widget->setGeometry(widget->x(), widget->y() + offsetY, resizeW, resizeH);
                    }
                } else if (pressedArea.at(5)) {
                    int resizeW = rectW + offsetX;
                    int resizeH = widget->height() - offsetY;
                    if (widget->minimumHeight() <= resizeH) {
                        widget->setGeometry(widget->x(), widget->y() + offsetY, resizeW, resizeH);
                    }
                } else if (pressedArea.at(6)) {
                    int resizeW = widget->width() - offsetX;
                    int resizeH = rectH + offsetY;
                    if (widget->minimumWidth() <= resizeW) {
                        widget->setGeometry(widget->x() + offsetX, widget->y(), resizeW, resizeH);
                    }
                    if (widget->minimumHeight() <= resizeH) {
                        widget->setGeometry(widget->x(), widget->y(), resizeW, resizeH);
                    }
                } else if (pressedArea.at(7)) {
                    int resizeW = rectW + offsetX;
                    int resizeH = rectH + offsetY;
                    widget->setGeometry(widget->x(), widget->y(), resizeW, resizeH);
                }
            }
        } else if (type == QEvent::MouseButtonPress) {
            //必须是鼠标左键
            if (qApp->mouseButtons() == Qt::LeftButton) {
                //记住鼠标按下的坐标和窗体区域
                QMouseEvent *mouseEvent = (QMouseEvent *)event;
                mousePoint = mouseEvent->pos();
                mouseRect = widget->geometry();

                //判断按下的手柄的区域位置
                if (pressedRect.at(0).contains(mousePoint)) {
                    pressedArea[0] = true;
                } else if (pressedRect.at(1).contains(mousePoint)) {
                    pressedArea[1] = true;
                } else if (pressedRect.at(2).contains(mousePoint)) {
                    pressedArea[2] = true;
                } else if (pressedRect.at(3).contains(mousePoint)) {
                    pressedArea[3] = true;
                } else if (pressedRect.at(4).contains(mousePoint)) {
                    pressedArea[4] = true;
                } else if (pressedRect.at(5).contains(mousePoint)) {
                    pressedArea[5] = true;
                } else if (pressedRect.at(6).contains(mousePoint)) {
                    pressedArea[6] = true;
                } else if (pressedRect.at(7).contains(mousePoint)) {
                    pressedArea[7] = true;
                } else {
                    mousePressed = true;
                }
            }
        } else if (type == QEvent::MouseMove) {
            //改成用HoverMove识别
        } else if (type == QEvent::MouseButtonRelease) {
            //恢复所有
            widget->setCursor(Qt::ArrowCursor);
            mousePressed = false;
            for (int i = 0; i < 8; ++i) {
                pressedArea[i] = false;
            }
        }
    }

    return QObject::eventFilter(watched, event);
}

void FramelessWidget2::setPadding(int padding)
{
    this->padding = padding;
}

void FramelessWidget2::setMoveEnable(bool moveEnable)
{
    this->moveEnable = moveEnable;
}

void FramelessWidget2::setResizeEnable(bool resizeEnable)
{
    this->resizeEnable = resizeEnable;
}

void FramelessWidget2::setMousePressed(bool mousePressed)
{
    this->mousePressed = mousePressed;
}

void FramelessWidget2::setWidget(QWidget *widget)
{
    if (this->widget == 0) {
        this->widget = widget;
        //设置鼠标追踪为真
        this->widget->setMouseTracking(true);
        //绑定事件过滤器
        this->widget->installEventFilter(this);
        //设置悬停为真,必须设置这个,不然当父窗体里边还有子窗体全部遮挡了识别不到MouseMove,需要识别HoverMove
        this->widget->setAttribute(Qt::WA_Hover, true);

        isMin = false;
        flags = widget->windowFlags();
    }
}
```

### `./framelesswidget2.h`

```cpp
﻿#ifndef FRAMELESSWIDGET2_H
#define FRAMELESSWIDGET2_H

/**
 * 无边框窗体类 作者:feiyangqingyun(QQ:517216493) 2019-10-03
 * 1. 可以指定需要无边框的widget。
 * 2. 边框四周八个方位都可以自由拉伸。
 * 3. 可设置对应位置的边距，以便识别更大区域。
 * 4. 可设置是否允许拖动。
 * 5. 可设置是否允许拉伸。
 */

#include <QWidget>

#ifdef quc
class Q_DECL_EXPORT FramelessWidget2 : public QObject
#else
class FramelessWidget2 : public QObject
#endif

{
    Q_OBJECT
public:
    explicit FramelessWidget2(QObject *parent = 0);

protected:
    bool eventFilter(QObject *watched, QEvent *event);

private:
    //边距+可移动+可拉伸
    int padding;
    bool moveEnable;
    bool resizeEnable;

    //无边框窗体
    QWidget *widget;

    //鼠标是否按下+按下坐标+按下时窗体区域
    bool mousePressed;
    QPoint mousePoint;
    QRect mouseRect;

    //鼠标是否按下某个区域+按下区域的大小
    //依次为 左侧+右侧+上侧+下侧+左上侧+右上侧+左下侧+右下侧
    QList<bool> pressedArea;
    QList<QRect> pressedRect;

    //记录是否最小化
    bool isMin;
    //存储窗体默认的属性
    Qt::WindowFlags flags;

public Q_SLOTS:
    //设置边距
    void setPadding(int padding);
    //设置是否可拖动+拉伸
    void setMoveEnable(bool moveEnable);
    void setResizeEnable(bool resizeEnable);
    //修复部分控件不能自动识别 MouseButtonRelease 的bug
    void setMousePressed(bool mousePressed);

    //设置要无边框的窗体
    void setWidget(QWidget *widget);
};

#endif // FRAMELESSWIDGET2_H
```

### `./frmframelesswidget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmframelesswidget.h"
#include "ui_frmframelesswidget.h"
#include "qpushbutton.h"
#include "qcheckbox.h"
#include "qdebug.h"
#include "framelesswidget2.h"

#include "framelessform/dialog.h"
#include "framelessform/widget.h"
#include "framelessform/mainwindow.h"

frmFramelessWidget::frmFramelessWidget(QWidget *parent) : QWidget(parent), ui(new Ui::frmFramelessWidget)
{
    ui->setupUi(this);
    this->initForm();
}

frmFramelessWidget::~frmFramelessWidget()
{
    delete ui;
}

void frmFramelessWidget::initForm()
{
    widget = 0;
    frameless = 0;

    connect(ui->btnDialog, SIGNAL(clicked(bool)), this, SLOT(buttonClicked()));
    connect(ui->btnWidget, SIGNAL(clicked(bool)), this, SLOT(buttonClicked()));
    connect(ui->btnMainWindow, SIGNAL(clicked(bool)), this, SLOT(buttonClicked()));
}

void frmFramelessWidget::initWidget(QWidget *w)
{
    //设置无边框属性
    w->setWindowFlags(Qt::WindowStaysOnTopHint | Qt::FramelessWindowHint);
    //w->setWindowFlags(Qt::FramelessWindowHint | Qt::WindowSystemMenuHint | Qt::WindowMinMaxButtonsHint);
    w->setWindowTitle("自由拉伸无边框窗体");
    w->setMinimumSize(200, 120);
    w->resize(480, 320);

    //设置下背景颜色区别看
    QPalette palette = w->palette();
    palette.setBrush(QPalette::Window, QColor(162, 121, 197));
    w->setPalette(palette);

    QPushButton *btn = new QPushButton(w);
    btn->setText("关闭");
    btn->setGeometry(10, 10, 130, 25);
    connect(btn, SIGNAL(clicked(bool)), w, SLOT(close()));

    QCheckBox *cboxMove = new QCheckBox(w);
    cboxMove->setText("可移动");
    cboxMove->setChecked(true);
    cboxMove->setGeometry(10, 40, 70, 25);
    connect(cboxMove, SIGNAL(stateChanged(int)), this, SLOT(stateChanged1(int)));

    QCheckBox *cboxResize = new QCheckBox(w);
    cboxResize->setText("可拉伸");
    cboxResize->setChecked(true);
    cboxResize->setGeometry(80, 40, 70, 25);
    connect(cboxResize, SIGNAL(stateChanged(int)), this, SLOT(stateChanged2(int)));
}

void frmFramelessWidget::on_pushButton_clicked()
{
    if (widget == 0) {
        widget = new QWidget;
        this->initWidget(widget);
        frameless = new FramelessWidget2(widget);
        frameless->setWidget(widget);
    }

    //widget->resize(800, 600);
    widget->show();
}

void frmFramelessWidget::stateChanged1(int arg1)
{
    if (frameless != 0) {
        frameless->setMoveEnable(arg1 != 0);
    }
}

void frmFramelessWidget::stateChanged2(int arg1)
{
    if (frameless != 0) {
        frameless->setResizeEnable(arg1 != 0);
    }
}

void frmFramelessWidget::buttonClicked()
{
#ifndef Q_CC_MSVC
    QString objName = sender()->objectName();
    if (objName == "btnDialog") {
        Dialog dialog;
        dialog.exec();
    } else if (objName == "btnWidget") {
        Widget *widget = new Widget;
        widget->show();
    } else if (objName == "btnMainWindow") {
        MainWindow *window = new MainWindow;
        window->show();
    }
#endif
}
```

### `./frmframelesswidget.h`

```cpp
﻿#ifndef FRMFRAMELESSWIDGET_H
#define FRMFRAMELESSWIDGET_H

#include <QWidget>
class FramelessWidget2;

namespace Ui {
class frmFramelessWidget;
}

class frmFramelessWidget : public QWidget
{
    Q_OBJECT

public:
    explicit frmFramelessWidget(QWidget *parent = 0);
    ~frmFramelessWidget();

private:
    Ui::frmFramelessWidget *ui;
    QWidget *widget;
    FramelessWidget2 *frameless;

private slots:
    void initForm();
    void initWidget(QWidget *w);
    void on_pushButton_clicked();
    void stateChanged1(int arg1);
    void stateChanged2(int arg1);
    void buttonClicked();
};

#endif // FRMFRAMELESSWIDGET_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmframelesswidget.h"
#include <QApplication>
#include <QTextCodec>
#include <QDebug>

int main(int argc, char *argv[])
{
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    //QApplication::setAttribute(Qt::AA_Use96Dpi);
    QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
#endif
#if (QT_VERSION >= QT_VERSION_CHECK(5,14,0))
    QGuiApplication::setHighDpiScaleFactorRoundingPolicy(Qt::HighDpiScaleFactorRoundingPolicy::PassThrough);
#endif

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

    frmFramelessWidget w;    
    w.setWindowTitle("无边框窗体 (QQ: 517216493 WX: feiyangqingyun)");
    w.resize(800, 600);
    w.show();

    return a.exec();
}
```

### `framelesscore/framelessdialog.cpp`

```cpp
﻿#include "framelessdialog.h"
#include "qdatetime.h"
#include "qevent.h"
#include "qdebug.h"

#ifdef Q_OS_WIN
#include "windows.h"
#include "windowsx.h"
#pragma comment (lib,"user32.lib")
#endif

#define TIMEMS qPrintable(QTime::currentTime().toString("HH:mm:ss zzz"))

FramelessDialog::FramelessDialog(QWidget *parent) : QDialog(parent)
{
    padding = 8;
    moveEnable = true;
    resizeEnable = true;

    mousePressed = false;
    mousePoint = QPoint(0, 0);
    mouseRect = QRect(0, 0, 0, 0);

    for (int i = 0; i < 8; ++i) {
        pressedArea << false;
        pressedRect << QRect(0, 0, 0, 0);
    }

    isMin = false;
    flags = this->windowFlags();
    titleBar = 0;

    //设置背景透明 官方在5.3以后才彻底修复 WA_TranslucentBackground+FramelessWindowHint 并存不绘制的bug
#if (QT_VERSION >= QT_VERSION_CHECK(5,3,0))
    this->setAttribute(Qt::WA_TranslucentBackground);
#endif
    //设置无边框属性
    this->setWindowFlags(flags | Qt::FramelessWindowHint);
    //安装事件过滤器识别拖动
    this->installEventFilter(this);

    //设置属性产生win窗体效果,移动到边缘半屏或者最大化等
    //设置以后会产生标题栏需要在下面拦截消息重新去掉
#ifdef Q_OS_WIN
    HWND hwnd = (HWND)this->winId();
    DWORD style = ::GetWindowLong(hwnd, GWL_STYLE);
    ::SetWindowLong(hwnd, GWL_STYLE, style | WS_MAXIMIZEBOX | WS_THICKFRAME | WS_CAPTION);
#endif
}

void FramelessDialog::showEvent(QShowEvent *event)
{
    //解决有时候窗体重新显示的时候假死不刷新的bug
    setAttribute(Qt::WA_Mapped);
    QDialog::showEvent(event);
}

void FramelessDialog::doWindowStateChange(QEvent *event)
{
    //非最大化才能移动和拖动大小
    if (windowState() == Qt::WindowNoState) {
        moveEnable = true;
        resizeEnable = true;
    } else {
        moveEnable = false;
        resizeEnable = false;
    }

    //发出最大化最小化等改变事件,以便界面上更改对应的信息比如右上角图标和文字
    Q_EMIT windowStateChange(!moveEnable);

    //解决mac系统上无边框最小化失效的bug
#ifdef Q_OS_MACOS
    if (windowState() & Qt::WindowMinimized) {
        isMin = true;
    } else {
        if (isMin) {
            //设置无边框属性
            this->setWindowFlags(flags | Qt::FramelessWindowHint);
            this->setVisible(true);
            isMin = false;
        }
    }
#endif
}

void FramelessDialog::doResizeEvent(QEvent *event)
{
    //非win系统的无边框拉伸,win系统上已经采用了nativeEvent来处理拉伸
    //为何不统一用计算的方式因为在win上用这个方式往左拉伸会发抖妹的
#ifndef Q_OS_WIN
    int type = event->type();
    if (type == QEvent::Resize) {
        //重新计算八个描点的区域,描点区域的作用还有就是计算鼠标坐标是否在某一个区域内
        int width = this->width();
        int height = this->height();

        //左侧描点区域
        pressedRect[0] = QRect(0, padding, padding, height - padding * 2);
        //右侧描点区域
        pressedRect[1] = QRect(width - padding, padding, padding, height - padding * 2);
        //上侧描点区域
        pressedRect[2] = QRect(padding, 0, width - padding * 2, padding);
        //下侧描点区域
        pressedRect[3] = QRect(padding, height - padding, width - padding * 2, padding);

        //左上角描点区域
        pressedRect[4] = QRect(0, 0, padding, padding);
        //右上角描点区域
        pressedRect[5] = QRect(width - padding, 0, padding, padding);
        //左下角描点区域
        pressedRect[6] = QRect(0, height - padding, padding, padding);
        //右下角描点区域
        pressedRect[7] = QRect(width - padding, height - padding, padding, padding);
    } else if (type == QEvent::HoverMove) {
        //设置对应鼠标形状,这个必须放在这里而不是下面,因为可以在鼠标没有按下的时候识别
        QHoverEvent *hoverEvent = (QHoverEvent *)event;
        QPoint point = hoverEvent->pos();
        if (resizeEnable) {
            if (pressedRect.at(0).contains(point)) {
                this->setCursor(Qt::SizeHorCursor);
            } else if (pressedRect.at(1).contains(point)) {
                this->setCursor(Qt::SizeHorCursor);
            } else if (pressedRect.at(2).contains(point)) {
                this->setCursor(Qt::SizeVerCursor);
            } else if (pressedRect.at(3).contains(point)) {
                this->setCursor(Qt::SizeVerCursor);
            } else if (pressedRect.at(4).contains(point)) {
                this->setCursor(Qt::SizeFDiagCursor);
            } else if (pressedRect.at(5).contains(point)) {
                this->setCursor(Qt::SizeBDiagCursor);
            } else if (pressedRect.at(6).contains(point)) {
                this->setCursor(Qt::SizeBDiagCursor);
            } else if (pressedRect.at(7).contains(point)) {
                this->setCursor(Qt::SizeFDiagCursor);
            } else {
                this->setCursor(Qt::ArrowCursor);
            }
        }

        //根据当前鼠标位置,计算XY轴移动了多少
        int offsetX = point.x() - mousePoint.x();
        int offsetY = point.y() - mousePoint.y();

        //根据按下处的位置判断是否是移动控件还是拉伸控件
        if (moveEnable && mousePressed) {
            this->move(this->x() + offsetX, this->y() + offsetY);
        }

        if (resizeEnable) {
            int rectX = mouseRect.x();
            int rectY = mouseRect.y();
            int rectW = mouseRect.width();
            int rectH = mouseRect.height();

            if (pressedArea.at(0)) {
                int resizeW = this->width() - offsetX;
                if (this->minimumWidth() <= resizeW) {
                    this->setGeometry(this->x() + offsetX, rectY, resizeW, rectH);
                }
            } else if (pressedArea.at(1)) {
                this->setGeometry(rectX, rectY, rectW + offsetX, rectH);
            } else if (pressedArea.at(2)) {
                int resizeH = this->height() - offsetY;
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(rectX, this->y() + offsetY, rectW, resizeH);
                }
            } else if (pressedArea.at(3)) {
                this->setGeometry(rectX, rectY, rectW, rectH + offsetY);
            } else if (pressedArea.at(4)) {
                int resizeW = this->width() - offsetX;
                int resizeH = this->height() - offsetY;
                if (this->minimumWidth() <= resizeW) {
                    this->setGeometry(this->x() + offsetX, this->y(), resizeW, resizeH);
                }
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(this->x(), this->y() + offsetY, resizeW, resizeH);
                }
            } else if (pressedArea.at(5)) {
                int resizeW = rectW + offsetX;
                int resizeH = this->height() - offsetY;
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(this->x(), this->y() + offsetY, resizeW, resizeH);
                }
            } else if (pressedArea.at(6)) {
                int resizeW = this->width() - offsetX;
                int resizeH = rectH + offsetY;
                if (this->minimumWidth() <= resizeW) {
                    this->setGeometry(this->x() + offsetX, this->y(), resizeW, resizeH);
                }
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(this->x(), this->y(), resizeW, resizeH);
                }
            } else if (pressedArea.at(7)) {
                int resizeW = rectW + offsetX;
                int resizeH = rectH + offsetY;
                this->setGeometry(this->x(), this->y(), resizeW, resizeH);
            }
        }
    } else if (type == QEvent::MouseButtonPress) {
        //记住鼠标按下的坐标+窗体区域
        QMouseEvent *mouseEvent = (QMouseEvent *)event;
        mousePoint = mouseEvent->pos();
        mouseRect = this->geometry();

        //判断按下的手柄的区域位置
        if (pressedRect.at(0).contains(mousePoint)) {
            pressedArea[0] = true;
        } else if (pressedRect.at(1).contains(mousePoint)) {
            pressedArea[1] = true;
        } else if (pressedRect.at(2).contains(mousePoint)) {
            pressedArea[2] = true;
        } else if (pressedRect.at(3).contains(mousePoint)) {
            pressedArea[3] = true;
        } else if (pressedRect.at(4).contains(mousePoint)) {
            pressedArea[4] = true;
        } else if (pressedRect.at(5).contains(mousePoint)) {
            pressedArea[5] = true;
        } else if (pressedRect.at(6).contains(mousePoint)) {
            pressedArea[6] = true;
        } else if (pressedRect.at(7).contains(mousePoint)) {
            pressedArea[7] = true;
        } else {
            mousePressed = true;
        }
    } else if (type == QEvent::MouseMove) {
        //改成用HoverMove识别
    } else if (type == QEvent::MouseButtonRelease) {
        //恢复所有
        this->setCursor(Qt::ArrowCursor);
        mousePressed = false;
        for (int i = 0; i < 8; ++i) {
            pressedArea[i] = false;
        }
    }
#endif
}

bool FramelessDialog::eventFilter(QObject *watched, QEvent *event)
{
    int type = event->type();
    if (watched == this) {
        if (type == QEvent::WindowStateChange) {
            doWindowStateChange(event);
        } else {
            doResizeEvent(event);
        }
    } else if (watched == titleBar) {
        //双击标题栏发出双击信号给主界面
        //下面的 *result = HTCAPTION; 标志位也会自动识别双击标题栏
#ifndef Q_OS_WIN
        if (type == QEvent::MouseButtonDblClick) {
            Q_EMIT titleDblClick();
        } else if (type == QEvent::NonClientAreaMouseButtonDblClick) {
            Q_EMIT titleDblClick();
        }
#endif
    }

    return QDialog::eventFilter(watched, event);
}

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
bool FramelessDialog::nativeEvent(const QByteArray &eventType, void *message, qintptr *result)
#else
bool FramelessDialog::nativeEvent(const QByteArray &eventType, void *message, long *result)
#endif
{
    if (eventType == "windows_generic_MSG") {
#ifdef Q_OS_WIN
        MSG *msg = static_cast<MSG *>(message);
        //qDebug() << TIMEMS << "nativeEvent" << msg->wParam << msg->message;

        //不同的消息类型和参数进行不同的处理
        if (msg->message == WM_NCCALCSIZE) {
            *result = 0;
            return true;
        } else if (msg->message == WM_SYSKEYDOWN) {
            //屏蔽alt键按下
        } else if (msg->message == WM_SYSKEYUP) {
            //屏蔽alt键松开
        } else if (msg->message == WM_NCHITTEST) {
            //计算鼠标对应的屏幕坐标
            //这里最开始用的 LOWORD HIWORD 在多屏幕的时候会有问题
            //官方说明在这里 https://docs.microsoft.com/zh-cn/windows/win32/inputdev/wm-nchittest
            qreal ratio = 1.0;
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
            ratio = this->devicePixelRatioF();
#endif
            long x = GET_X_LPARAM(msg->lParam) / ratio;
            long y = GET_Y_LPARAM(msg->lParam) / ratio;
            QPoint pos = mapFromGlobal(QPoint(x, y));

            //判断当前鼠标位置在哪个区域
            bool left = pos.x() < padding;
            bool right = pos.x() > width() - padding;
            bool top = pos.y() < padding;
            bool bottom = pos.y() > height() - padding;

            //鼠标移动到四个角,这个消息是当鼠标移动或者有鼠标键按下时候发出的
            *result = 0;
            if (resizeEnable) {
                if (left && top) {
                    *result = HTTOPLEFT;
                } else if (left && bottom) {
                    *result = HTBOTTOMLEFT;
                } else if (right && top) {
                    *result = HTTOPRIGHT;
                } else if (right && bottom) {
                    *result = HTBOTTOMRIGHT;
                } else if (left) {
                    *result = HTLEFT;
                } else if (right) {
                    *result = HTRIGHT;
                } else if (top) {
                    *result = HTTOP;
                } else if (bottom) {
                    *result = HTBOTTOM;
                }
            }

            //先处理掉拉伸
            if (0 != *result) {
                return true;
            }

            //识别标题栏拖动产生半屏全屏效果
            if (titleBar && titleBar->rect().contains(pos)) {
                QWidget *child = titleBar->childAt(pos);
                if (!child) {
                    *result = HTCAPTION;
                    return true;
                }
            }
        } else if (msg->wParam == PBT_APMSUSPEND && msg->message == WM_POWERBROADCAST) {
            //系统休眠的时候自动最小化可以规避程序可能出现的问题
            this->showMinimized();
        } else if (msg->wParam == PBT_APMRESUMEAUTOMATIC) {
            //休眠唤醒后自动打开
            this->showNormal();
        }
#endif
    } else if (eventType == "NSEvent") {
#ifdef Q_OS_MACOS
#endif
    } else if (eventType == "xcb_generic_event_t") {
#ifdef Q_OS_LINUX
#endif
    }
    return false;
}

#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
#ifdef Q_OS_WIN
bool FramelessDialog::winEvent(MSG *message, long *result)
{
    return nativeEvent("windows_generic_MSG", message, result);
}
#endif
#endif

void FramelessDialog::setPadding(int padding)
{
    this->padding = padding;
}

void FramelessDialog::setMoveEnable(bool moveEnable)
{
    this->moveEnable = moveEnable;
}

void FramelessDialog::setResizeEnable(bool resizeEnable)
{
    this->resizeEnable = resizeEnable;
}

void FramelessDialog::setTitleBar(QWidget *titleBar)
{
    this->titleBar = titleBar;
    this->titleBar->installEventFilter(this);
}
```

### `framelesscore/framelessdialog.h`

```cpp
﻿#ifndef FRAMELESSDIALOG_H
#define FRAMELESSDIALOG_H

/**
 * 无边框窗体类 作者:feiyangqingyun(QQ:517216493) 2021-07-27
 * 1. 同时支持Qt4-Qt6，亲测Qt4.7到Qt6.2。
 * 2. 同时支持mingw、msvc、gcc等。
 * 3. 同时支持windows、linux、mac。
 * 4. 同时支持QMainWindow、QWidget、QDialog。
 * 5. 使用方法极其简单，只需要将继承类修改即可。
 * 6. 自动识别双击标题栏响应。
 * 7. 无边框拉伸在windows下不抖动。
 * 8. 在windows下具有移动到边缘半屏、移动到顶部全屏特性。
 * 9. 解决mac系统上无边框最小化最大化失效的bug。
 * 10. 解决系统休眠后再次启动程序懵逼的bug。
 * 11. 解决有时候窗体重新显示的时候假死不刷新的bug。
 * 12. 轻量级，1个代码文件，核心代码行数不到300行。
 * 13. 注释详细，示例完美，非常适合阅读和学习。
 * 14. 开源开箱即用，保证任意Qt版本可正常编译运行，无需任何调整。
 */

#include <QDialog>

#ifdef quc
class Q_DECL_EXPORT FramelessDialog : public QDialog
#else
class FramelessDialog : public QDialog
#endif

{
    Q_OBJECT
public:
    explicit FramelessDialog(QWidget *parent = 0);

protected:
    //窗体显示的时候触发
    void showEvent(QShowEvent *event);

    //事件过滤器识别拖动拉伸等
    void doWindowStateChange(QEvent *event);
    void doResizeEvent(QEvent *event);
    bool eventFilter(QObject *watched, QEvent *event);

    //拦截系统事件用于修复系统休眠后唤醒程序的bug
#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
    bool nativeEvent(const QByteArray &eventType, void *message, qintptr *result);
#else
    bool nativeEvent(const QByteArray &eventType, void *message, long *result);
#endif

    //Qt4的写法
#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
#ifdef Q_OS_WIN
    bool winEvent(MSG *message, long *result);
#endif
#endif

private:
    //边距+可移动+可拉伸
    int padding;
    bool moveEnable;
    bool resizeEnable;

    //标题栏控件
    QWidget *titleBar;

    //鼠标是否按下+按下坐标+按下时窗体区域
    bool mousePressed;
    QPoint mousePoint;
    QRect mouseRect;

    //鼠标是否按下某个区域+按下区域的大小
    //依次为 左侧+右侧+上侧+下侧+左上侧+右上侧+左下侧+右下侧
    QList<bool> pressedArea;
    QList<QRect> pressedRect;

    //记录是否最小化
    bool isMin;
    //存储窗体默认的属性
    Qt::WindowFlags flags;

public Q_SLOTS:
    //设置边距+可拖动+可拉伸
    void setPadding(int padding);
    void setMoveEnable(bool moveEnable);
    void setResizeEnable(bool resizeEnable);

    //设置标题栏控件
    void setTitleBar(QWidget *titleBar);

Q_SIGNALS:
    void titleDblClick();
    void windowStateChange(bool max);
};

#endif // FRAMELESSDIALOG_H
```

### `framelesscore/framelessmainwindow.cpp`

```cpp
﻿#include "framelessmainwindow.h"
#include "qdatetime.h"
#include "qevent.h"
#include "qdebug.h"

#ifdef Q_OS_WIN
#include "windows.h"
#include "windowsx.h"
#pragma comment (lib,"user32.lib")
#endif

#define TIMEMS qPrintable(QTime::currentTime().toString("HH:mm:ss zzz"))

FramelessMainWindow::FramelessMainWindow(QWidget *parent) : QMainWindow(parent)
{
    padding = 8;
    moveEnable = true;
    resizeEnable = true;

    mousePressed = false;
    mousePoint = QPoint(0, 0);
    mouseRect = QRect(0, 0, 0, 0);

    for (int i = 0; i < 8; ++i) {
        pressedArea << false;
        pressedRect << QRect(0, 0, 0, 0);
    }

    isMin = false;
    flags = this->windowFlags();
    titleBar = 0;

    //设置背景透明 官方在5.3以后才彻底修复 WA_TranslucentBackground+FramelessWindowHint 并存不绘制的bug
#if (QT_VERSION >= QT_VERSION_CHECK(5,3,0))
    this->setAttribute(Qt::WA_TranslucentBackground);
#endif
    //设置无边框属性
    this->setWindowFlags(flags | Qt::FramelessWindowHint);
    //安装事件过滤器识别拖动
    this->installEventFilter(this);

    //设置属性产生win窗体效果,移动到边缘半屏或者最大化等
    //设置以后会产生标题栏,需要在下面拦截消息WM_NCCALCSIZE重新去掉
#ifdef Q_OS_WIN
    HWND hwnd = (HWND)this->winId();
    DWORD style = ::GetWindowLong(hwnd, GWL_STYLE);
    ::SetWindowLong(hwnd, GWL_STYLE, style | WS_MAXIMIZEBOX | WS_THICKFRAME | WS_CAPTION);
#endif
}

void FramelessMainWindow::showEvent(QShowEvent *event)
{
    //解决有时候窗体重新显示的时候假死不刷新的bug
    setAttribute(Qt::WA_Mapped);
    QMainWindow::showEvent(event);
}

void FramelessMainWindow::doWindowStateChange(QEvent *event)
{
    //非最大化才能移动和拖动大小
    if (windowState() == Qt::WindowNoState) {
        moveEnable = true;
        resizeEnable = true;
    } else {
        moveEnable = false;
        resizeEnable = false;
    }

    //发出最大化最小化等改变事件,以便界面上更改对应的信息比如右上角图标和文字
    Q_EMIT windowStateChange(!moveEnable);

    //解决mac系统上无边框最小化失效的bug
#ifdef Q_OS_MACOS
    if (windowState() & Qt::WindowMinimized) {
        isMin = true;
    } else {
        if (isMin) {
            //设置无边框属性
            this->setWindowFlags(flags | Qt::FramelessWindowHint);
            this->setVisible(true);
            isMin = false;
        }
    }
#endif
}

void FramelessMainWindow::doResizeEvent(QEvent *event)
{
    //非win系统的无边框拉伸,win系统上已经采用了nativeEvent来处理拉伸
    //为何不统一用计算的方式因为在win上用这个方式往左拉伸会发抖妹的
#ifndef Q_OS_WIN
    int type = event->type();
    if (type == QEvent::Resize) {
        //重新计算八个描点的区域,描点区域的作用还有就是计算鼠标坐标是否在某一个区域内
        int width = this->width();
        int height = this->height();

        //左侧描点区域
        pressedRect[0] = QRect(0, padding, padding, height - padding * 2);
        //右侧描点区域
        pressedRect[1] = QRect(width - padding, padding, padding, height - padding * 2);
        //上侧描点区域
        pressedRect[2] = QRect(padding, 0, width - padding * 2, padding);
        //下侧描点区域
        pressedRect[3] = QRect(padding, height - padding, width - padding * 2, padding);

        //左上角描点区域
        pressedRect[4] = QRect(0, 0, padding, padding);
        //右上角描点区域
        pressedRect[5] = QRect(width - padding, 0, padding, padding);
        //左下角描点区域
        pressedRect[6] = QRect(0, height - padding, padding, padding);
        //右下角描点区域
        pressedRect[7] = QRect(width - padding, height - padding, padding, padding);
    } else if (type == QEvent::HoverMove) {
        //设置对应鼠标形状,这个必须放在这里而不是下面,因为可以在鼠标没有按下的时候识别
        QHoverEvent *hoverEvent = (QHoverEvent *)event;
        QPoint point = hoverEvent->pos();
        if (resizeEnable) {
            if (pressedRect.at(0).contains(point)) {
                this->setCursor(Qt::SizeHorCursor);
            } else if (pressedRect.at(1).contains(point)) {
                this->setCursor(Qt::SizeHorCursor);
            } else if (pressedRect.at(2).contains(point)) {
                this->setCursor(Qt::SizeVerCursor);
            } else if (pressedRect.at(3).contains(point)) {
                this->setCursor(Qt::SizeVerCursor);
            } else if (pressedRect.at(4).contains(point)) {
                this->setCursor(Qt::SizeFDiagCursor);
            } else if (pressedRect.at(5).contains(point)) {
                this->setCursor(Qt::SizeBDiagCursor);
            } else if (pressedRect.at(6).contains(point)) {
                this->setCursor(Qt::SizeBDiagCursor);
            } else if (pressedRect.at(7).contains(point)) {
                this->setCursor(Qt::SizeFDiagCursor);
            } else {
                this->setCursor(Qt::ArrowCursor);
            }
        }

        //根据当前鼠标位置,计算XY轴移动了多少
        int offsetX = point.x() - mousePoint.x();
        int offsetY = point.y() - mousePoint.y();

        //根据按下处的位置判断是否是移动控件还是拉伸控件
        if (moveEnable && mousePressed) {
            this->move(this->x() + offsetX, this->y() + offsetY);
        }

        if (resizeEnable) {
            int rectX = mouseRect.x();
            int rectY = mouseRect.y();
            int rectW = mouseRect.width();
            int rectH = mouseRect.height();

            if (pressedArea.at(0)) {
                int resizeW = this->width() - offsetX;
                if (this->minimumWidth() <= resizeW) {
                    this->setGeometry(this->x() + offsetX, rectY, resizeW, rectH);
                }
            } else if (pressedArea.at(1)) {
                this->setGeometry(rectX, rectY, rectW + offsetX, rectH);
            } else if (pressedArea.at(2)) {
                int resizeH = this->height() - offsetY;
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(rectX, this->y() + offsetY, rectW, resizeH);
                }
            } else if (pressedArea.at(3)) {
                this->setGeometry(rectX, rectY, rectW, rectH + offsetY);
            } else if (pressedArea.at(4)) {
                int resizeW = this->width() - offsetX;
                int resizeH = this->height() - offsetY;
                if (this->minimumWidth() <= resizeW) {
                    this->setGeometry(this->x() + offsetX, this->y(), resizeW, resizeH);
                }
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(this->x(), this->y() + offsetY, resizeW, resizeH);
                }
            } else if (pressedArea.at(5)) {
                int resizeW = rectW + offsetX;
                int resizeH = this->height() - offsetY;
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(this->x(), this->y() + offsetY, resizeW, resizeH);
                }
            } else if (pressedArea.at(6)) {
                int resizeW = this->width() - offsetX;
                int resizeH = rectH + offsetY;
                if (this->minimumWidth() <= resizeW) {
                    this->setGeometry(this->x() + offsetX, this->y(), resizeW, resizeH);
                }
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(this->x(), this->y(), resizeW, resizeH);
                }
            } else if (pressedArea.at(7)) {
                int resizeW = rectW + offsetX;
                int resizeH = rectH + offsetY;
                this->setGeometry(this->x(), this->y(), resizeW, resizeH);
            }
        }
    } else if (type == QEvent::MouseButtonPress) {
        //记住鼠标按下的坐标+窗体区域
        QMouseEvent *mouseEvent = (QMouseEvent *)event;
        mousePoint = mouseEvent->pos();
        mouseRect = this->geometry();

        //判断按下的手柄的区域位置
        if (pressedRect.at(0).contains(mousePoint)) {
            pressedArea[0] = true;
        } else if (pressedRect.at(1).contains(mousePoint)) {
            pressedArea[1] = true;
        } else if (pressedRect.at(2).contains(mousePoint)) {
            pressedArea[2] = true;
        } else if (pressedRect.at(3).contains(mousePoint)) {
            pressedArea[3] = true;
        } else if (pressedRect.at(4).contains(mousePoint)) {
            pressedArea[4] = true;
        } else if (pressedRect.at(5).contains(mousePoint)) {
            pressedArea[5] = true;
        } else if (pressedRect.at(6).contains(mousePoint)) {
            pressedArea[6] = true;
        } else if (pressedRect.at(7).contains(mousePoint)) {
            pressedArea[7] = true;
        } else {
            mousePressed = true;
        }
    } else if (type == QEvent::MouseMove) {
        //改成用HoverMove识别
    } else if (type == QEvent::MouseButtonRelease) {
        //恢复所有
        this->setCursor(Qt::ArrowCursor);
        mousePressed = false;
        for (int i = 0; i < 8; ++i) {
            pressedArea[i] = false;
        }
    }
#endif
}

bool FramelessMainWindow::eventFilter(QObject *watched, QEvent *event)
{
    int type = event->type();
    if (watched == this) {
        if (type == QEvent::WindowStateChange) {
            doWindowStateChange(event);
        } else {
            doResizeEvent(event);
        }
    } else if (watched == titleBar) {
        //双击标题栏发出双击信号给主界面
        //下面的 *result = HTCAPTION; 标志位也会自动识别双击标题栏
#ifndef Q_OS_WIN
        if (type == QEvent::MouseButtonDblClick) {
            Q_EMIT titleDblClick();
        } else if (type == QEvent::NonClientAreaMouseButtonDblClick) {
            Q_EMIT titleDblClick();
        }
#endif
    }

    return QMainWindow::eventFilter(watched, event);
}

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
bool FramelessMainWindow::nativeEvent(const QByteArray &eventType, void *message, qintptr *result)
#else
bool FramelessMainWindow::nativeEvent(const QByteArray &eventType, void *message, long *result)
#endif
{
    if (eventType == "windows_generic_MSG") {
#ifdef Q_OS_WIN
        MSG *msg = static_cast<MSG *>(message);
        //qDebug() << TIMEMS << "nativeEvent" << msg->wParam << msg->message;

        //不同的消息类型和参数进行不同的处理
        if (msg->message == WM_NCCALCSIZE) {
            *result = 0;
            return true;
        } else if (msg->message == WM_SYSKEYDOWN) {
            //屏蔽alt键按下
        } else if (msg->message == WM_SYSKEYUP) {
            //屏蔽alt键松开
        } else if (msg->message == WM_NCHITTEST) {
            //计算鼠标对应的屏幕坐标
            //这里最开始用的 LOWORD HIWORD 在多屏幕的时候会有问题
            //官方说明在这里 https://docs.microsoft.com/zh-cn/windows/win32/inputdev/wm-nchittest
            qreal ratio = 1.0;
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
            ratio = this->devicePixelRatioF();
#endif
            long x = GET_X_LPARAM(msg->lParam) / ratio;
            long y = GET_Y_LPARAM(msg->lParam) / ratio;
            QPoint pos = mapFromGlobal(QPoint(x, y));

            //判断当前鼠标位置在哪个区域
            bool left = pos.x() < padding;
            bool right = pos.x() > width() - padding;
            bool top = pos.y() < padding;
            bool bottom = pos.y() > height() - padding;

            //鼠标移动到四个角,这个消息是当鼠标移动或者有鼠标键按下时候发出的
            *result = 0;
            if (resizeEnable) {
                if (left && top) {
                    *result = HTTOPLEFT;
                } else if (left && bottom) {
                    *result = HTBOTTOMLEFT;
                } else if (right && top) {
                    *result = HTTOPRIGHT;
                } else if (right && bottom) {
                    *result = HTBOTTOMRIGHT;
                } else if (left) {
                    *result = HTLEFT;
                } else if (right) {
                    *result = HTRIGHT;
                } else if (top) {
                    *result = HTTOP;
                } else if (bottom) {
                    *result = HTBOTTOM;
                }
            }

            //先处理掉拉伸
            if (0 != *result) {
                return true;
            }

            //识别标题栏拖动产生半屏全屏效果
            if (titleBar && titleBar->rect().contains(pos)) {
                QWidget *child = titleBar->childAt(pos);
                if (!child) {
                    *result = HTCAPTION;
                    return true;
                }
            }
        } else if (msg->wParam == PBT_APMSUSPEND && msg->message == WM_POWERBROADCAST) {
            //系统休眠的时候自动最小化可以规避程序可能出现的问题
            this->showMinimized();
        } else if (msg->wParam == PBT_APMRESUMEAUTOMATIC) {
            //休眠唤醒后自动打开
            this->showNormal();
        }
#endif
    } else if (eventType == "NSEvent") {
#ifdef Q_OS_MACOS
#endif
    } else if (eventType == "xcb_generic_event_t") {
#ifdef Q_OS_LINUX
#endif
    }
    return false;
}

#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
#ifdef Q_OS_WIN
bool FramelessMainWindow::winEvent(MSG *message, long *result)
{
    return nativeEvent("windows_generic_MSG", message, result);
}
#endif
#endif

void FramelessMainWindow::setPadding(int padding)
{
    this->padding = padding;
}

void FramelessMainWindow::setMoveEnable(bool moveEnable)
{
    this->moveEnable = moveEnable;
}

void FramelessMainWindow::setResizeEnable(bool resizeEnable)
{
    this->resizeEnable = resizeEnable;
}

void FramelessMainWindow::setTitleBar(QWidget *titleBar)
{
    this->titleBar = titleBar;
    this->titleBar->installEventFilter(this);
}
```

### `framelesscore/framelessmainwindow.h`

```cpp
﻿#ifndef FRAMELESSMAINWINDOW_H
#define FRAMELESSMAINWINDOW_H

/**
 * 无边框窗体类 作者:feiyangqingyun(QQ:517216493) 2021-07-27
 * 1. 同时支持Qt4-Qt6，亲测Qt4.7到Qt6.2。
 * 2. 同时支持mingw、msvc、gcc等。
 * 3. 同时支持windows、linux、mac。
 * 4. 同时支持QMainWindow、QWidget、QDialog。
 * 5. 使用方法极其简单，只需要将继承类修改即可。
 * 6. 自动识别双击标题栏响应。
 * 7. 无边框拉伸在windows下不抖动。
 * 8. 在windows下具有移动到边缘半屏、移动到顶部全屏特性。
 * 9. 解决mac系统上无边框最小化最大化失效的bug。
 * 10. 解决系统休眠后再次启动程序懵逼的bug。
 * 11. 解决有时候窗体重新显示的时候假死不刷新的bug。
 * 12. 轻量级，1个代码文件，核心代码行数不到300行。
 * 13. 注释详细，示例完美，非常适合阅读和学习。
 * 14. 开源开箱即用，保证任意Qt版本可正常编译运行，无需任何调整。
 */

#include <QMainWindow>

#ifdef quc
class Q_DECL_EXPORT FramelessMainWindow : public QMainWindow
#else
class FramelessMainWindow : public QMainWindow
#endif

{
    Q_OBJECT
public:
    explicit FramelessMainWindow(QWidget *parent = 0);

protected:
    //窗体显示的时候触发
    void showEvent(QShowEvent *event);

    //事件过滤器识别拖动拉伸等
    void doWindowStateChange(QEvent *event);
    void doResizeEvent(QEvent *event);
    bool eventFilter(QObject *watched, QEvent *event);

    //拦截系统事件用于修复系统休眠后唤醒程序的bug
#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
    bool nativeEvent(const QByteArray &eventType, void *message, qintptr *result);
#else
    bool nativeEvent(const QByteArray &eventType, void *message, long *result);
#endif

    //Qt4的写法
#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
#ifdef Q_OS_WIN
    bool winEvent(MSG *message, long *result);
#endif
#endif

private:
    //边距+可移动+可拉伸
    int padding;
    bool moveEnable;
    bool resizeEnable;

    //标题栏控件
    QWidget *titleBar;

    //鼠标是否按下+按下坐标+按下时窗体区域
    bool mousePressed;
    QPoint mousePoint;
    QRect mouseRect;

    //鼠标是否按下某个区域+按下区域的大小
    //依次为 左侧+右侧+上侧+下侧+左上侧+右上侧+左下侧+右下侧
    QList<bool> pressedArea;
    QList<QRect> pressedRect;

    //记录是否最小化
    bool isMin;
    //存储窗体默认的属性
    Qt::WindowFlags flags;

public Q_SLOTS:
    //设置边距+可拖动+可拉伸
    void setPadding(int padding);
    void setMoveEnable(bool moveEnable);
    void setResizeEnable(bool resizeEnable);

    //设置标题栏控件
    void setTitleBar(QWidget *titleBar);

Q_SIGNALS:
    void titleDblClick();
    void windowStateChange(bool max);
};

#endif // FRAMELESSMAINWINDOW_H
```

### `framelesscore/framelesswidget.cpp`

```cpp
﻿#include "framelesswidget.h"
#include "qdatetime.h"
#include "qevent.h"
#include "qdebug.h"

#ifdef Q_OS_WIN
#include "windows.h"
#include "windowsx.h"
#pragma comment (lib,"user32.lib")
#endif

#define TIMEMS qPrintable(QTime::currentTime().toString("HH:mm:ss zzz"))

FramelessWidget::FramelessWidget(QWidget *parent) : QWidget(parent)
{
    padding = 8;
    moveEnable = true;
    resizeEnable = true;

    mousePressed = false;
    mousePoint = QPoint(0, 0);
    mouseRect = QRect(0, 0, 0, 0);

    for (int i = 0; i < 8; ++i) {
        pressedArea << false;
        pressedRect << QRect(0, 0, 0, 0);
    }

    isMin = false;
    flags = this->windowFlags();
    titleBar = 0;

    //设置背景透明 官方在5.3以后才彻底修复 WA_TranslucentBackground+FramelessWindowHint 并存不绘制的bug
#if (QT_VERSION >= QT_VERSION_CHECK(5,3,0))
    this->setAttribute(Qt::WA_TranslucentBackground);
#endif
    //设置无边框属性
    this->setWindowFlags(flags | Qt::FramelessWindowHint);
    //安装事件过滤器识别拖动
    this->installEventFilter(this);

    //设置属性产生win窗体效果,移动到边缘半屏或者最大化等
    //设置以后会产生标题栏需要在下面拦截消息重新去掉
#ifdef Q_OS_WIN
    HWND hwnd = (HWND)this->winId();
    DWORD style = ::GetWindowLong(hwnd, GWL_STYLE);
    ::SetWindowLong(hwnd, GWL_STYLE, style | WS_MAXIMIZEBOX | WS_THICKFRAME | WS_CAPTION);
#endif
}

void FramelessWidget::showEvent(QShowEvent *event)
{
    //解决有时候窗体重新显示的时候假死不刷新的bug
    setAttribute(Qt::WA_Mapped);
    QWidget::showEvent(event);
}

void FramelessWidget::doWindowStateChange(QEvent *event)
{
    //非最大化才能移动和拖动大小
    if (windowState() == Qt::WindowNoState) {
        moveEnable = true;
        resizeEnable = true;
    } else {
        moveEnable = false;
        resizeEnable = false;
    }

    //发出最大化最小化等改变事件,以便界面上更改对应的信息比如右上角图标和文字
    Q_EMIT windowStateChange(!moveEnable);

    //解决mac系统上无边框最小化失效的bug
#ifdef Q_OS_MACOS
    if (windowState() & Qt::WindowMinimized) {
        isMin = true;
    } else {
        if (isMin) {
            //设置无边框属性
            this->setWindowFlags(flags | Qt::FramelessWindowHint);
            this->setVisible(true);
            isMin = false;
        }
    }
#endif
}

void FramelessWidget::doResizeEvent(QEvent *event)
{
    //非win系统的无边框拉伸,win系统上已经采用了nativeEvent来处理拉伸
    //为何不统一用计算的方式因为在win上用这个方式往左拉伸会发抖妹的
#ifndef Q_OS_WIN
    int type = event->type();
    if (type == QEvent::Resize) {
        //重新计算八个描点的区域,描点区域的作用还有就是计算鼠标坐标是否在某一个区域内
        int width = this->width();
        int height = this->height();

        //左侧描点区域
        pressedRect[0] = QRect(0, padding, padding, height - padding * 2);
        //右侧描点区域
        pressedRect[1] = QRect(width - padding, padding, padding, height - padding * 2);
        //上侧描点区域
        pressedRect[2] = QRect(padding, 0, width - padding * 2, padding);
        //下侧描点区域
        pressedRect[3] = QRect(padding, height - padding, width - padding * 2, padding);

        //左上角描点区域
        pressedRect[4] = QRect(0, 0, padding, padding);
        //右上角描点区域
        pressedRect[5] = QRect(width - padding, 0, padding, padding);
        //左下角描点区域
        pressedRect[6] = QRect(0, height - padding, padding, padding);
        //右下角描点区域
        pressedRect[7] = QRect(width - padding, height - padding, padding, padding);
    } else if (type == QEvent::HoverMove) {
        //设置对应鼠标形状,这个必须放在这里而不是下面,因为可以在鼠标没有按下的时候识别
        QHoverEvent *hoverEvent = (QHoverEvent *)event;
        QPoint point = hoverEvent->pos();
        if (resizeEnable) {
            if (pressedRect.at(0).contains(point)) {
                this->setCursor(Qt::SizeHorCursor);
            } else if (pressedRect.at(1).contains(point)) {
                this->setCursor(Qt::SizeHorCursor);
            } else if (pressedRect.at(2).contains(point)) {
                this->setCursor(Qt::SizeVerCursor);
            } else if (pressedRect.at(3).contains(point)) {
                this->setCursor(Qt::SizeVerCursor);
            } else if (pressedRect.at(4).contains(point)) {
                this->setCursor(Qt::SizeFDiagCursor);
            } else if (pressedRect.at(5).contains(point)) {
                this->setCursor(Qt::SizeBDiagCursor);
            } else if (pressedRect.at(6).contains(point)) {
                this->setCursor(Qt::SizeBDiagCursor);
            } else if (pressedRect.at(7).contains(point)) {
                this->setCursor(Qt::SizeFDiagCursor);
            } else {
                this->setCursor(Qt::ArrowCursor);
            }
        }

        //根据当前鼠标位置,计算XY轴移动了多少
        int offsetX = point.x() - mousePoint.x();
        int offsetY = point.y() - mousePoint.y();

        //根据按下处的位置判断是否是移动控件还是拉伸控件
        if (moveEnable && mousePressed) {
            this->move(this->x() + offsetX, this->y() + offsetY);
        }

        if (resizeEnable) {
            int rectX = mouseRect.x();
            int rectY = mouseRect.y();
            int rectW = mouseRect.width();
            int rectH = mouseRect.height();

            if (pressedArea.at(0)) {
                int resizeW = this->width() - offsetX;
                if (this->minimumWidth() <= resizeW) {
                    this->setGeometry(this->x() + offsetX, rectY, resizeW, rectH);
                }
            } else if (pressedArea.at(1)) {
                this->setGeometry(rectX, rectY, rectW + offsetX, rectH);
            } else if (pressedArea.at(2)) {
                int resizeH = this->height() - offsetY;
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(rectX, this->y() + offsetY, rectW, resizeH);
                }
            } else if (pressedArea.at(3)) {
                this->setGeometry(rectX, rectY, rectW, rectH + offsetY);
            } else if (pressedArea.at(4)) {
                int resizeW = this->width() - offsetX;
                int resizeH = this->height() - offsetY;
                if (this->minimumWidth() <= resizeW) {
                    this->setGeometry(this->x() + offsetX, this->y(), resizeW, resizeH);
                }
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(this->x(), this->y() + offsetY, resizeW, resizeH);
                }
            } else if (pressedArea.at(5)) {
                int resizeW = rectW + offsetX;
                int resizeH = this->height() - offsetY;
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(this->x(), this->y() + offsetY, resizeW, resizeH);
                }
            } else if (pressedArea.at(6)) {
                int resizeW = this->width() - offsetX;
                int resizeH = rectH + offsetY;
                if (this->minimumWidth() <= resizeW) {
                    this->setGeometry(this->x() + offsetX, this->y(), resizeW, resizeH);
                }
                if (this->minimumHeight() <= resizeH) {
                    this->setGeometry(this->x(), this->y(), resizeW, resizeH);
                }
            } else if (pressedArea.at(7)) {
                int resizeW = rectW + offsetX;
                int resizeH = rectH + offsetY;
                this->setGeometry(this->x(), this->y(), resizeW, resizeH);
            }
        }
    } else if (type == QEvent::MouseButtonPress) {
        //记住鼠标按下的坐标+窗体区域
        QMouseEvent *mouseEvent = (QMouseEvent *)event;
        mousePoint = mouseEvent->pos();
        mouseRect = this->geometry();

        //判断按下的手柄的区域位置
        if (pressedRect.at(0).contains(mousePoint)) {
            pressedArea[0] = true;
        } else if (pressedRect.at(1).contains(mousePoint)) {
            pressedArea[1] = true;
        } else if (pressedRect.at(2).contains(mousePoint)) {
            pressedArea[2] = true;
        } else if (pressedRect.at(3).contains(mousePoint)) {
            pressedArea[3] = true;
        } else if (pressedRect.at(4).contains(mousePoint)) {
            pressedArea[4] = true;
        } else if (pressedRect.at(5).contains(mousePoint)) {
            pressedArea[5] = true;
        } else if (pressedRect.at(6).contains(mousePoint)) {
            pressedArea[6] = true;
        } else if (pressedRect.at(7).contains(mousePoint)) {
            pressedArea[7] = true;
        } else {
            mousePressed = true;
        }
    } else if (type == QEvent::MouseMove) {
        //改成用HoverMove识别
    } else if (type == QEvent::MouseButtonRelease) {
        //恢复所有
        this->setCursor(Qt::ArrowCursor);
        mousePressed = false;
        for (int i = 0; i < 8; ++i) {
            pressedArea[i] = false;
        }
    }
#endif
}

bool FramelessWidget::eventFilter(QObject *watched, QEvent *event)
{
    int type = event->type();
    if (watched == this) {
        if (type == QEvent::WindowStateChange) {
            doWindowStateChange(event);
        } else {
            doResizeEvent(event);
        }
    } else if (watched == titleBar) {
        //双击标题栏发出双击信号给主界面
        //下面的 *result = HTCAPTION; 标志位也会自动识别双击标题栏
#ifndef Q_OS_WIN
        if (type == QEvent::MouseButtonDblClick) {
            Q_EMIT titleDblClick();
        } else if (type == QEvent::NonClientAreaMouseButtonDblClick) {
            Q_EMIT titleDblClick();
        }
#endif
    }

    return QWidget::eventFilter(watched, event);
}

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
bool FramelessWidget::nativeEvent(const QByteArray &eventType, void *message, qintptr *result)
#else
bool FramelessWidget::nativeEvent(const QByteArray &eventType, void *message, long *result)
#endif
{
    if (eventType == "windows_generic_MSG") {
#ifdef Q_OS_WIN
        MSG *msg = static_cast<MSG *>(message);
        //qDebug() << TIMEMS << "nativeEvent" << msg->wParam << msg->message;

        //不同的消息类型和参数进行不同的处理
        if (msg->message == WM_NCCALCSIZE) {
            *result = 0;
            return true;
        } else if (msg->message == WM_SYSKEYDOWN) {
            //屏蔽alt键按下
        } else if (msg->message == WM_SYSKEYUP) {
            //屏蔽alt键松开
        } else if (msg->message == WM_NCHITTEST) {
            //计算鼠标对应的屏幕坐标
            //这里最开始用的 LOWORD HIWORD 在多屏幕的时候会有问题
            //官方说明在这里 https://docs.microsoft.com/zh-cn/windows/win32/inputdev/wm-nchittest
            qreal ratio = 1.0;
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
            ratio = this->devicePixelRatioF();
#endif
            long x = GET_X_LPARAM(msg->lParam) / ratio;
            long y = GET_Y_LPARAM(msg->lParam) / ratio;
            QPoint pos = mapFromGlobal(QPoint(x, y));

            //判断当前鼠标位置在哪个区域
            bool left = pos.x() < padding;
            bool right = pos.x() > width() - padding;
            bool top = pos.y() < padding;
            bool bottom = pos.y() > height() - padding;

            //鼠标移动到四个角,这个消息是当鼠标移动或者有鼠标键按下时候发出的
            *result = 0;
            if (resizeEnable) {
                if (left && top) {
                    *result = HTTOPLEFT;
                } else if (left && bottom) {
                    *result = HTBOTTOMLEFT;
                } else if (right && top) {
                    *result = HTTOPRIGHT;
                } else if (right && bottom) {
                    *result = HTBOTTOMRIGHT;
                } else if (left) {
                    *result = HTLEFT;
                } else if (right) {
                    *result = HTRIGHT;
                } else if (top) {
                    *result = HTTOP;
                } else if (bottom) {
                    *result = HTBOTTOM;
                }
            }

            //先处理掉拉伸
            if (0 != *result) {
                return true;
            }

            //识别标题栏拖动产生半屏全屏效果
            if (titleBar && titleBar->rect().contains(pos)) {
                QWidget *child = titleBar->childAt(pos);
                if (!child) {
                    *result = HTCAPTION;
                    return true;
                }
            }
        } else if (msg->wParam == PBT_APMSUSPEND && msg->message == WM_POWERBROADCAST) {
            //系统休眠的时候自动最小化可以规避程序可能出现的问题
            this->showMinimized();
        } else if (msg->wParam == PBT_APMRESUMEAUTOMATIC) {
            //休眠唤醒后自动打开
            this->showNormal();
        }
#endif
    } else if (eventType == "NSEvent") {
#ifdef Q_OS_MACOS
#endif
    } else if (eventType == "xcb_generic_event_t") {
#ifdef Q_OS_LINUX
#endif
    }
    return false;
}

#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
#ifdef Q_OS_WIN
bool FramelessWidget::winEvent(MSG *message, long *result)
{
    return nativeEvent("windows_generic_MSG", message, result);
}
#endif
#endif

void FramelessWidget::setPadding(int padding)
{
    this->padding = padding;
}

void FramelessWidget::setMoveEnable(bool moveEnable)
{
    this->moveEnable = moveEnable;
}

void FramelessWidget::setResizeEnable(bool resizeEnable)
{
    this->resizeEnable = resizeEnable;
}

void FramelessWidget::setTitleBar(QWidget *titleBar)
{
    this->titleBar = titleBar;
    this->titleBar->installEventFilter(this);
}
```

### `framelesscore/framelesswidget.h`

```cpp
﻿#ifndef FRAMELESSWIDGET_H
#define FRAMELESSWIDGET_H

/**
 * 无边框窗体类 作者:feiyangqingyun(QQ:517216493) 2021-07-27
 * 1. 同时支持Qt4-Qt6，亲测Qt4.7到Qt6.2。
 * 2. 同时支持mingw、msvc、gcc等。
 * 3. 同时支持windows、linux、mac。
 * 4. 同时支持QMainWindow、QWidget、QDialog。
 * 5. 使用方法极其简单，只需要将继承类修改即可。
 * 6. 自动识别双击标题栏响应。
 * 7. 无边框拉伸在windows下不抖动。
 * 8. 在windows下具有移动到边缘半屏、移动到顶部全屏特性。
 * 9. 解决mac系统上无边框最小化最大化失效的bug。
 * 10. 解决系统休眠后再次启动程序懵逼的bug。
 * 11. 解决有时候窗体重新显示的时候假死不刷新的bug。
 * 12. 轻量级，1个代码文件，核心代码行数不到300行。
 * 13. 注释详细，示例完美，非常适合阅读和学习。
 * 14. 开源开箱即用，保证任意Qt版本可正常编译运行，无需任何调整。
 */

#include <QWidget>

#ifdef quc
class Q_DECL_EXPORT FramelessWidget : public QWidget
#else
class FramelessWidget : public QWidget
#endif

{
    Q_OBJECT
public:
    explicit FramelessWidget(QWidget *parent = 0);

protected:
    //窗体显示的时候触发
    void showEvent(QShowEvent *event);

    //事件过滤器识别拖动拉伸等
    void doWindowStateChange(QEvent *event);
    void doResizeEvent(QEvent *event);
    bool eventFilter(QObject *watched, QEvent *event);

    //拦截系统事件用于修复系统休眠后唤醒程序的bug
#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
    bool nativeEvent(const QByteArray &eventType, void *message, qintptr *result);
#else
    bool nativeEvent(const QByteArray &eventType, void *message, long *result);
#endif

    //Qt4的写法
#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
#ifdef Q_OS_WIN
    bool winEvent(MSG *message, long *result);
#endif
#endif

private:
    //边距+可移动+可拉伸
    int padding;
    bool moveEnable;
    bool resizeEnable;

    //标题栏控件
    QWidget *titleBar;

    //鼠标是否按下+按下坐标+按下时窗体区域
    bool mousePressed;
    QPoint mousePoint;
    QRect mouseRect;

    //鼠标是否按下某个区域+按下区域的大小
    //依次为 左侧+右侧+上侧+下侧+左上侧+右上侧+左下侧+右下侧
    QList<bool> pressedArea;
    QList<QRect> pressedRect;

    //记录是否最小化
    bool isMin;
    //存储窗体默认的属性
    Qt::WindowFlags flags;

public Q_SLOTS:
    //设置边距+可拖动+可拉伸
    void setPadding(int padding);
    void setMoveEnable(bool moveEnable);
    void setResizeEnable(bool resizeEnable);

    //设置标题栏控件
    void setTitleBar(QWidget *titleBar);

Q_SIGNALS:
    void titleDblClick();
    void windowStateChange(bool max);
};

#endif // FRAMELESSWIDGET_H
```

### `framelessform/dialog.cpp`

```cpp
﻿#include "dialog.h"
#include "ui_dialog.h"
#include "qdebug.h"

#pragma execution_character_set("utf-8")
Dialog::Dialog(QWidget *parent) : FramelessDialog(parent), ui(new Ui::Dialog)
{
    ui->setupUi(this);
    this->initForm();
}

Dialog::~Dialog()
{
    delete ui;
}

void Dialog::initForm()
{
    //设置标题栏控件
    ui->labTitle->setText("无边框窗体示例-支持win、linux、mac等系统 (QQ: 517216493 WX: feiyangqingyun)");
    this->setWindowTitle(ui->labTitle->text());
    this->setTitleBar(ui->labTitle);

    //关联信号
    connect(this, SIGNAL(titleDblClick()), this, SLOT(titleDblClick()));
    connect(this, SIGNAL(windowStateChange(bool)), this, SLOT(windowStateChange(bool)));

    //设置样式表
    QStringList list;
    list << "#titleBar{background:#BBBBBB;}";
    list << "#titleBar{border-top-left-radius:8px;border-top-right-radius:8px;}";
    list << "#widgetMain{border:2px solid #BBBBBB;background:#FFFFFF;}";
    //list << "#widgetMain{border-bottom-left-radius:8px;border-bottom-right-radius:8px;}";
    this->setStyleSheet(list.join(""));
}

void Dialog::titleDblClick()
{
    on_btnMenu_Max_clicked();
}

void Dialog::windowStateChange(bool max)
{
    ui->btnMenu_Max->setText(max ? "还原" : "最大");
}

void Dialog::on_btnMenu_Min_clicked()
{
#ifdef Q_OS_MACOS
    this->setWindowFlags(this->windowFlags() & ~Qt::FramelessWindowHint);
#endif
    this->showMinimized();
}

void Dialog::on_btnMenu_Max_clicked()
{
    if (this->isMaximized()) {
        this->showNormal();
        ui->btnMenu_Max->setText("最大");
    } else {
        this->showMaximized();
        ui->btnMenu_Max->setText("还原");
    }
}

void Dialog::on_btnMenu_Close_clicked()
{
    this->close();
}
```

### `framelessform/dialog.h`

```cpp
﻿#ifndef DIALOG_H
#define DIALOG_H

#include "framelessdialog.h"

namespace Ui {
class Dialog;
}

class Dialog : public FramelessDialog
{
    Q_OBJECT

public:
    explicit Dialog(QWidget *parent = 0);
    ~Dialog();

private:
    Ui::Dialog *ui;

private slots:
    void initForm();
    void titleDblClick();
    void windowStateChange(bool max);

private slots:
    void on_btnMenu_Min_clicked();
    void on_btnMenu_Max_clicked();
    void on_btnMenu_Close_clicked();
};

#endif // DIALOG_H
```

### `framelessform/mainwindow.cpp`

```cpp
﻿#include "mainwindow.h"
#include "ui_mainwindow.h"
#include "qdebug.h"

#pragma execution_character_set("utf-8")
MainWindow::MainWindow(QWidget *parent) : FramelessMainWindow(parent), ui(new Ui::MainWindow)
{
    ui->setupUi(this);
    this->initForm();
}

MainWindow::~MainWindow()
{
    delete ui;
}

void MainWindow::initForm()
{
    //设置标题栏控件
    ui->labTitle->setText("无边框窗体示例-支持win、linux、mac等系统 (QQ: 517216493 WX: feiyangqingyun)");
    this->setWindowTitle(ui->labTitle->text());
    this->setTitleBar(ui->labTitle);

    //关联信号
    connect(this, SIGNAL(titleDblClick()), this, SLOT(titleDblClick()));
    connect(this, SIGNAL(windowStateChange(bool)), this, SLOT(windowStateChange(bool)));

    //设置样式表
    QStringList list;
    list << "#titleBar{background:#BBBBBB;}";
    list << "#titleBar{border-top-left-radius:8px;border-top-right-radius:8px;}";
    list << "#widgetMain{border:2px solid #BBBBBB;background:#FFFFFF;}";
    //list << "#widgetMain{border-bottom-left-radius:8px;border-bottom-right-radius:8px;}";
    this->setStyleSheet(list.join(""));
}

void MainWindow::titleDblClick()
{
    on_btnMenu_Max_clicked();
}

void MainWindow::windowStateChange(bool max)
{
    ui->btnMenu_Max->setText(max ? "还原" : "最大");
}

void MainWindow::on_btnMenu_Min_clicked()
{
#ifdef Q_OS_MACOS
    this->setWindowFlags(this->windowFlags() & ~Qt::FramelessWindowHint);
#endif
    this->showMinimized();
}

void MainWindow::on_btnMenu_Max_clicked()
{
    if (this->isMaximized()) {
        this->showNormal();
        ui->btnMenu_Max->setText("最大");
    } else {
        this->showMaximized();
        ui->btnMenu_Max->setText("还原");
    }
}

void MainWindow::on_btnMenu_Close_clicked()
{
    this->close();
}
```

### `framelessform/mainwindow.h`

```cpp
﻿#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include "framelessmainwindow.h"

namespace Ui {
class MainWindow;
}

class MainWindow : public FramelessMainWindow
{
    Q_OBJECT

public:
    explicit MainWindow(QWidget *parent = 0);
    ~MainWindow();

private:
    Ui::MainWindow *ui;

private slots:
    void initForm();
    void titleDblClick();
    void windowStateChange(bool max);

private slots:
    void on_btnMenu_Min_clicked();
    void on_btnMenu_Max_clicked();
    void on_btnMenu_Close_clicked();
};

#endif // MAINWINDOW_H
```

### `framelessform/widget.cpp`

```cpp
﻿#include "widget.h"
#include "ui_widget.h"
#include "qdebug.h"

#pragma execution_character_set("utf-8")
Widget::Widget(QWidget *parent) : FramelessWidget(parent), ui(new Ui::Widget)
{
    ui->setupUi(this);
    this->initForm();
}

Widget::~Widget()
{
    delete ui;
}

void Widget::initForm()
{
    //设置标题栏控件
    ui->labTitle->setText("无边框窗体示例-支持win、linux、mac等系统 (QQ: 517216493 WX: feiyangqingyun)");
    this->setWindowTitle(ui->labTitle->text());
    this->setTitleBar(ui->labTitle);

    //关联信号
    connect(this, SIGNAL(titleDblClick()), this, SLOT(titleDblClick()));
    connect(this, SIGNAL(windowStateChange(bool)), this, SLOT(windowStateChange(bool)));

    //设置样式表
    QStringList list;
    list << "#titleBar{background:#BBBBBB;}";
    list << "#titleBar{border-top-left-radius:8px;border-top-right-radius:8px;}";
    list << "#widgetMain{border:2px solid #BBBBBB;background:#FFFFFF;}";
    //list << "#widgetMain{border-bottom-left-radius:8px;border-bottom-right-radius:8px;}";
    this->setStyleSheet(list.join(""));
}

void Widget::titleDblClick()
{
    on_btnMenu_Max_clicked();
}

void Widget::windowStateChange(bool max)
{
    ui->btnMenu_Max->setText(max ? "还原" : "最大");
}

void Widget::on_btnMenu_Min_clicked()
{
#ifdef Q_OS_MACOS
    this->setWindowFlags(this->windowFlags() & ~Qt::FramelessWindowHint);
#endif
    this->showMinimized();
}

void Widget::on_btnMenu_Max_clicked()
{
    if (this->isMaximized()) {
        this->showNormal();
        ui->btnMenu_Max->setText("最大");
    } else {
        this->showMaximized();
        ui->btnMenu_Max->setText("还原");
    }
}

void Widget::on_btnMenu_Close_clicked()
{
    this->close();
}
```

### `framelessform/widget.h`

```cpp
﻿#ifndef WIDGET_H
#define WIDGET_H

#include "framelesswidget.h"

namespace Ui {
class Widget;
}

class Widget : public FramelessWidget
{
    Q_OBJECT

public:
    explicit Widget(QWidget *parent = 0);
    ~Widget();

private:
    Ui::Widget *ui;

private slots:
    void initForm();
    void titleDblClick();
    void windowStateChange(bool max);

private slots:
    void on_btnMenu_Min_clicked();
    void on_btnMenu_Max_clicked();
    void on_btnMenu_Close_clicked();
};

#endif // WIDGET_H
```

### `./frmframelesswidget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmFramelessWidget</class>
 <widget class="QWidget" name="frmFramelessWidget">
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
  <widget class="QWidget" name="">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>10</y>
     <width>491</width>
     <height>30</height>
    </rect>
   </property>
   <layout class="QHBoxLayout" name="horizontalLayout">
    <item>
     <widget class="QPushButton" name="pushButton">
      <property name="text">
       <string>弹出</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="QPushButton" name="btnDialog">
      <property name="text">
       <string>Dialog</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="QPushButton" name="btnWidget">
      <property name="text">
       <string>Widget</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="QPushButton" name="btnMainWindow">
      <property name="text">
       <string>MainWindow</string>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
 </widget>
 <resources/>
 <connections/>
</ui>
```

### `framelessform/dialog.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>Dialog</class>
 <widget class="FramelessDialog" name="Dialog">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Dialog</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <property name="spacing">
    <number>0</number>
   </property>
   <property name="leftMargin">
    <number>0</number>
   </property>
   <property name="topMargin">
    <number>0</number>
   </property>
   <property name="rightMargin">
    <number>0</number>
   </property>
   <property name="bottomMargin">
    <number>0</number>
   </property>
   <item>
    <widget class="QWidget" name="titleBar" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
     <property name="minimumSize">
      <size>
       <width>0</width>
       <height>40</height>
      </size>
     </property>
     <layout class="QHBoxLayout" name="horizontalLayout_2">
      <property name="spacing">
       <number>0</number>
      </property>
      <property name="topMargin">
       <number>0</number>
      </property>
      <property name="rightMargin">
       <number>6</number>
      </property>
      <property name="bottomMargin">
       <number>0</number>
      </property>
      <item>
       <widget class="QLabel" name="labTitle">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QWidget" name="widgetMenu" native="true">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="maximumSize">
         <size>
          <width>16777215</width>
          <height>28</height>
         </size>
        </property>
        <layout class="QHBoxLayout" name="horizontalLayout">
         <property name="spacing">
          <number>0</number>
         </property>
         <property name="leftMargin">
          <number>0</number>
         </property>
         <property name="topMargin">
          <number>0</number>
         </property>
         <property name="rightMargin">
          <number>0</number>
         </property>
         <property name="bottomMargin">
          <number>0</number>
         </property>
         <item>
          <widget class="QPushButton" name="btnMenu_Min">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="minimumSize">
            <size>
             <width>60</width>
             <height>0</height>
            </size>
           </property>
           <property name="maximumSize">
            <size>
             <width>60</width>
             <height>16777215</height>
            </size>
           </property>
           <property name="cursor">
            <cursorShape>ArrowCursor</cursorShape>
           </property>
           <property name="focusPolicy">
            <enum>Qt::NoFocus</enum>
           </property>
           <property name="toolTip">
            <string>最小化</string>
           </property>
           <property name="text">
            <string>最小</string>
           </property>
          </widget>
         </item>
         <item>
          <widget class="QPushButton" name="btnMenu_Max">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="minimumSize">
            <size>
             <width>60</width>
             <height>0</height>
            </size>
           </property>
           <property name="maximumSize">
            <size>
             <width>60</width>
             <height>16777215</height>
            </size>
           </property>
           <property name="focusPolicy">
            <enum>Qt::NoFocus</enum>
           </property>
           <property name="text">
            <string>最大</string>
           </property>
          </widget>
         </item>
         <item>
          <widget class="QPushButton" name="btnMenu_Close">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="minimumSize">
            <size>
             <width>60</width>
             <height>0</height>
            </size>
           </property>
           <property name="maximumSize">
            <size>
             <width>60</width>
             <height>16777215</height>
            </size>
           </property>
           <property name="cursor">
            <cursorShape>ArrowCursor</cursorShape>
           </property>
           <property name="focusPolicy">
            <enum>Qt::NoFocus</enum>
           </property>
           <property name="toolTip">
            <string>关闭</string>
           </property>
           <property name="text">
            <string>关闭</string>
           </property>
          </widget>
         </item>
        </layout>
       </widget>
      </item>
     </layout>
    </widget>
   </item>
   <item>
    <widget class="QWidget" name="widgetMain" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Preferred" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>FramelessDialog</class>
   <extends>QWidget</extends>
   <header>framelessdialog.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `framelessform/mainwindow.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>MainWindow</class>
 <widget class="QMainWindow" name="MainWindow">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>MainWindow</string>
  </property>
  <widget class="QWidget" name="centralwidget">
   <property name="geometry">
    <rect>
     <x>0</x>
     <y>0</y>
     <width>207</width>
     <height>59</height>
    </rect>
   </property>
   <layout class="QVBoxLayout" name="verticalLayout" stretch="0,0">
    <property name="spacing">
     <number>0</number>
    </property>
    <property name="leftMargin">
     <number>0</number>
    </property>
    <property name="topMargin">
     <number>0</number>
    </property>
    <property name="rightMargin">
     <number>0</number>
    </property>
    <property name="bottomMargin">
     <number>0</number>
    </property>
    <item>
     <widget class="QWidget" name="titleBar" native="true">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="minimumSize">
       <size>
        <width>0</width>
        <height>40</height>
       </size>
      </property>
      <layout class="QHBoxLayout" name="horizontalLayout_2">
       <property name="spacing">
        <number>0</number>
       </property>
       <property name="topMargin">
        <number>0</number>
       </property>
       <property name="rightMargin">
        <number>6</number>
       </property>
       <property name="bottomMargin">
        <number>0</number>
       </property>
       <item>
        <widget class="QLabel" name="labTitle">
         <property name="sizePolicy">
          <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
           <horstretch>0</horstretch>
           <verstretch>0</verstretch>
          </sizepolicy>
         </property>
        </widget>
       </item>
       <item>
        <widget class="QWidget" name="widgetMenu" native="true">
         <property name="sizePolicy">
          <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
           <horstretch>0</horstretch>
           <verstretch>0</verstretch>
          </sizepolicy>
         </property>
         <property name="maximumSize">
          <size>
           <width>16777215</width>
           <height>28</height>
          </size>
         </property>
         <layout class="QHBoxLayout" name="horizontalLayout">
          <property name="spacing">
           <number>0</number>
          </property>
          <property name="leftMargin">
           <number>0</number>
          </property>
          <property name="topMargin">
           <number>0</number>
          </property>
          <property name="rightMargin">
           <number>0</number>
          </property>
          <property name="bottomMargin">
           <number>0</number>
          </property>
          <item>
           <widget class="QPushButton" name="btnMenu_Min">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="minimumSize">
             <size>
              <width>60</width>
              <height>0</height>
             </size>
            </property>
            <property name="maximumSize">
             <size>
              <width>60</width>
              <height>16777215</height>
             </size>
            </property>
            <property name="cursor">
             <cursorShape>ArrowCursor</cursorShape>
            </property>
            <property name="focusPolicy">
             <enum>Qt::NoFocus</enum>
            </property>
            <property name="toolTip">
             <string>最小化</string>
            </property>
            <property name="text">
             <string>最小</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QPushButton" name="btnMenu_Max">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="minimumSize">
             <size>
              <width>60</width>
              <height>0</height>
             </size>
            </property>
            <property name="maximumSize">
             <size>
              <width>60</width>
              <height>16777215</height>
             </size>
            </property>
            <property name="focusPolicy">
             <enum>Qt::NoFocus</enum>
            </property>
            <property name="text">
             <string>最大</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QPushButton" name="btnMenu_Close">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="minimumSize">
             <size>
              <width>60</width>
              <height>0</height>
             </size>
            </property>
            <property name="maximumSize">
             <size>
              <width>60</width>
              <height>16777215</height>
             </size>
            </property>
            <property name="cursor">
             <cursorShape>ArrowCursor</cursorShape>
            </property>
            <property name="focusPolicy">
             <enum>Qt::NoFocus</enum>
            </property>
            <property name="toolTip">
             <string>关闭</string>
            </property>
            <property name="text">
             <string>关闭</string>
            </property>
           </widget>
          </item>
         </layout>
        </widget>
       </item>
      </layout>
     </widget>
    </item>
    <item>
     <widget class="QWidget" name="widgetMain" native="true">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Preferred" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
 </widget>
 <customwidgets>
  <customwidget>
   <class>FramelessMainWindow</class>
   <extends>QWidget</extends>
   <header>framelessmainwindow.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `framelessform/widget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>Widget</class>
 <widget class="FramelessWidget" name="Widget">
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
  <layout class="QVBoxLayout" name="verticalLayout">
   <property name="spacing">
    <number>0</number>
   </property>
   <property name="leftMargin">
    <number>0</number>
   </property>
   <property name="topMargin">
    <number>0</number>
   </property>
   <property name="rightMargin">
    <number>0</number>
   </property>
   <property name="bottomMargin">
    <number>0</number>
   </property>
   <item>
    <widget class="QWidget" name="titleBar" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
     <property name="minimumSize">
      <size>
       <width>0</width>
       <height>40</height>
      </size>
     </property>
     <layout class="QHBoxLayout" name="horizontalLayout_2">
      <property name="spacing">
       <number>0</number>
      </property>
      <property name="topMargin">
       <number>0</number>
      </property>
      <property name="rightMargin">
       <number>6</number>
      </property>
      <property name="bottomMargin">
       <number>0</number>
      </property>
      <item>
       <widget class="QLabel" name="labTitle">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QWidget" name="widgetMenu" native="true">
        <property name="sizePolicy">
         <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
          <horstretch>0</horstretch>
          <verstretch>0</verstretch>
         </sizepolicy>
        </property>
        <property name="maximumSize">
         <size>
          <width>16777215</width>
          <height>28</height>
         </size>
        </property>
        <layout class="QHBoxLayout" name="horizontalLayout">
         <property name="spacing">
          <number>0</number>
         </property>
         <property name="leftMargin">
          <number>0</number>
         </property>
         <property name="topMargin">
          <number>0</number>
         </property>
         <property name="rightMargin">
          <number>0</number>
         </property>
         <property name="bottomMargin">
          <number>0</number>
         </property>
         <item>
          <widget class="QPushButton" name="btnMenu_Min">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="minimumSize">
            <size>
             <width>60</width>
             <height>0</height>
            </size>
           </property>
           <property name="maximumSize">
            <size>
             <width>60</width>
             <height>16777215</height>
            </size>
           </property>
           <property name="cursor">
            <cursorShape>ArrowCursor</cursorShape>
           </property>
           <property name="focusPolicy">
            <enum>Qt::NoFocus</enum>
           </property>
           <property name="toolTip">
            <string>最小化</string>
           </property>
           <property name="text">
            <string>最小</string>
           </property>
          </widget>
         </item>
         <item>
          <widget class="QPushButton" name="btnMenu_Max">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="minimumSize">
            <size>
             <width>60</width>
             <height>0</height>
            </size>
           </property>
           <property name="maximumSize">
            <size>
             <width>60</width>
             <height>16777215</height>
            </size>
           </property>
           <property name="focusPolicy">
            <enum>Qt::NoFocus</enum>
           </property>
           <property name="text">
            <string>最大</string>
           </property>
          </widget>
         </item>
         <item>
          <widget class="QPushButton" name="btnMenu_Close">
           <property name="sizePolicy">
            <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
             <horstretch>0</horstretch>
             <verstretch>0</verstretch>
            </sizepolicy>
           </property>
           <property name="minimumSize">
            <size>
             <width>60</width>
             <height>0</height>
            </size>
           </property>
           <property name="maximumSize">
            <size>
             <width>60</width>
             <height>16777215</height>
            </size>
           </property>
           <property name="cursor">
            <cursorShape>ArrowCursor</cursorShape>
           </property>
           <property name="focusPolicy">
            <enum>Qt::NoFocus</enum>
           </property>
           <property name="toolTip">
            <string>关闭</string>
           </property>
           <property name="text">
            <string>关闭</string>
           </property>
          </widget>
         </item>
        </layout>
       </widget>
      </item>
     </layout>
    </widget>
   </item>
   <item>
    <widget class="QWidget" name="widgetMain" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Preferred" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>FramelessWidget</class>
   <extends>QWidget</extends>
   <header>framelesswidget.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `framelesscore/framelesscore.pri`

```makefile
HEADERS += \
    $$PWD/framelessdialog.h \
    $$PWD/framelessmainwindow.h \
    $$PWD/framelesswidget.h

SOURCES += \
    $$PWD/framelessdialog.cpp \
    $$PWD/framelessmainwindow.cpp \
    $$PWD/framelesswidget.cpp
```

### `framelessform/framelessform.pri`

```makefile
FORMS += \
    $$PWD/dialog.ui \
    $$PWD/mainwindow.ui \
    $$PWD/widget.ui

HEADERS += \
    $$PWD/dialog.h \
    $$PWD/mainwindow.h \
    $$PWD/widget.h

SOURCES += \
    $$PWD/dialog.cpp \
    $$PWD/mainwindow.cpp \
    $$PWD/widget.cpp
```
