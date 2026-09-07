---
tags:
  - Qt
  - 窗体相关
---
# 颜色拾取器 (`colorwidget`)

> 颜色拾取器

## 效果图

![[QWidgetDemo_assert/colorwidget.jpg]]

## 分类

- 类型: 窗体相关 `widget`
- 源码目录: `widget/colorwidget`

## 技术要点

**用到的 Qt 类**: QWidget QLabel QTextCodec QLineEdit QSizePolicy QPushButton QString QTimer QMouseEvent QGridLayout QVBoxLayout QSize QCursor QApplication QPixmap QColor QChar QFont QFrame QMutex

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./colorwidget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "colorwidget.h"
#include "qmutex.h"
#include "qgridlayout.h"
#include "qlabel.h"
#include "qlineedit.h"
#include "qpushbutton.h"
#include "qapplication.h"
#include "qtimer.h"
#include "qevent.h"
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

ColorWidget *ColorWidget::instance = 0;
ColorWidget *ColorWidget::Instance()
{
    if (!instance) {
        static QMutex mutex;
        QMutexLocker locker(&mutex);
        if (!instance) {
            instance = new ColorWidget;
        }
    }

    return instance;
}

ColorWidget::ColorWidget(QWidget *parent) : QWidget(parent)
{
    gridLayout = new QGridLayout(this);
    gridLayout->setSpacing(6);
    gridLayout->setContentsMargins(11, 11, 11, 11);

    verticalLayout = new QVBoxLayout();
    verticalLayout->setSpacing(0);

    labColor = new QLabel(this);
    labColor->setText("+");
    labColor->setStyleSheet("background-color: rgb(255, 107, 107);color: rgb(250, 250, 250);");
    labColor->setAlignment(Qt::AlignCenter);
    QFont font;
    font.setPixelSize(35);
    font.setBold(true);
    labColor->setFont(font);

    QSizePolicy sizePolicy(QSizePolicy::Preferred, QSizePolicy::Expanding);
    sizePolicy.setHorizontalStretch(0);
    sizePolicy.setVerticalStretch(0);
    sizePolicy.setHeightForWidth(labColor->sizePolicy().hasHeightForWidth());
    labColor->setSizePolicy(sizePolicy);
    labColor->setMinimumSize(QSize(80, 70));
    labColor->setMaximumSize(QSize(80, 70));
    labColor->setCursor(QCursor(Qt::CrossCursor));
    labColor->setFrameShape(QFrame::StyledPanel);
    labColor->setFrameShadow(QFrame::Sunken);

    verticalLayout->addWidget(labColor);

    QLabel *labName = new QLabel(this);
    labName->setMinimumSize(QSize(0, 18));
    labName->setStyleSheet("background-color: rgb(0, 0, 0);color: rgb(200, 200, 200);");
    labName->setAlignment(Qt::AlignCenter);

    verticalLayout->addWidget(labName);
    gridLayout->addLayout(verticalLayout, 0, 0, 3, 1);

    QLabel *labWeb = new QLabel(this);
    gridLayout->addWidget(labWeb, 0, 1, 1, 1);

    txtWeb = new QLineEdit(this);
    gridLayout->addWidget(txtWeb, 0, 2, 1, 1);

    QLabel *labRgb = new QLabel(this);
    gridLayout->addWidget(labRgb, 1, 1, 1, 1);

    txtRgb = new QLineEdit(this);
    gridLayout->addWidget(txtRgb, 1, 2, 1, 1);

    QLabel *labPoint = new QLabel(this);
    gridLayout->addWidget(labPoint, 2, 1, 1, 1);

    txtPoint = new QLineEdit(this);
    gridLayout->addWidget(txtPoint, 2, 2, 1, 1);

    //底部增加按钮
    QPushButton *btn = new QPushButton;
    connect(btn, SIGNAL(clicked(bool)), this, SLOT(buttonClicked()));
    btn->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Expanding);
    btn->setText("开始拾色");
    gridLayout->addWidget(btn, 0, 3, 3, 1);

    labName->setText("当前颜色");
    labWeb->setText("web值:");
    labRgb->setText("rgb值:");
    labPoint->setText("坐标值:");

    this->setLayout(gridLayout);
    this->setWindowTitle("屏幕拾色器");
    this->setFixedSize(400, 108);

    cp = QApplication::clipboard();
    pressed = false;

    timer = new QTimer(this);
    timer->setInterval(100);
    connect(timer, SIGNAL(timeout()), this, SLOT(showColorValue()));
    timer->start();
}

ColorWidget::~ColorWidget()
{
}

void ColorWidget::mousePressEvent(QMouseEvent *e)
{
    if (labColor->rect().contains(e->pos())) {
        pressed = true;
    }
}

void ColorWidget::mouseReleaseEvent(QMouseEvent *)
{
    pressed = false;
}

void ColorWidget::showColorValue()
{
    if (!pressed) {
        return;
    }

    int x = QCursor::pos().x();
    int y = QCursor::pos().y();
    txtPoint->setText(tr("x:%1  y:%2").arg(x).arg(y));

#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    QScreen *screen = qApp->primaryScreen();
    QPixmap pixmap = screen->grabWindow(0, x, y, 2, 2);
#else
    QPixmap pixmap = QPixmap::grabWindow(qApp->desktop()->winId(), x, y, 2, 2);
#endif

    int red, green, blue;
    QString strDecimalValue, strHex;
    if (pixmap.isNull()) {
        return;
    }

    QImage image = pixmap.toImage();
    if (image.valid(0, 0)) {
        QColor color = image.pixel(0, 0);
        red = color.red();
        green = color.green();
        blue = color.blue();
        QString strRed = tr("%1").arg(red & 0xFF, 2, 16, QChar('0'));
        QString strGreen = tr("%1").arg(green & 0xFF, 2, 16, QChar('0'));
        QString strBlue = tr("%1").arg(blue & 0xFF, 2, 16, QChar('0'));

        strDecimalValue = tr("%1, %2, %3").arg(red).arg(green).arg(blue);
        strHex = tr("#%1%2%3").arg(strRed.toUpper()).arg(strGreen.toUpper()).arg(strBlue.toUpper());
    }

    //根据背景色自动计算合适的前景色
    QColor color(red, green, blue);
    double gray = (0.299 * color.red() + 0.587 * color.green() + 0.114 * color.blue()) / 255;
    QColor textColor = gray > 0.5 ? Qt::black : Qt::white;

    QString str = tr("background:rgb(%1);color:%2").arg(strDecimalValue).arg(textColor.name());
    labColor->setStyleSheet(str);
    txtRgb->setText(strDecimalValue);
    txtWeb->setText(strHex);
}

void ColorWidget::buttonClicked()
{
    QPushButton *btn = (QPushButton *)sender();
    if (btn->text() == "开始拾色") {
        btn->setText("停止拾色");
        pressed = true;
    } else {
        btn->setText("开始拾色");
        pressed = false;
    }
}
```

### `./colorwidget.h`

```cpp
﻿#ifndef COLORWIDGET_H
#define COLORWIDGET_H

/**
 * 屏幕拾色器 作者:feiyangqingyun(QQ:517216493) 2016-11-11
 * 1. 鼠标按下实时采集鼠标处的颜色。
 * 2. 实时显示颜色值。
 * 3. 支持16进制格式和rgb格式。
 * 4. 实时显示预览颜色。
 * 5. 根据背景色自动计算合适的前景色。
 */

#include <QWidget>

class QGridLayout;
class QVBoxLayout;
class QLabel;
class QLineEdit;

#ifdef quc
class Q_DECL_EXPORT ColorWidget : public QWidget
#else
class ColorWidget : public QWidget
#endif

{
    Q_OBJECT
public:
    static ColorWidget *Instance();
    explicit ColorWidget(QWidget *parent = 0);
    ~ColorWidget();

protected:
    void mousePressEvent(QMouseEvent *);
    void mouseReleaseEvent(QMouseEvent *);

private:
    static ColorWidget *instance;
    QClipboard *cp;
    bool pressed;
    QTimer *timer;

    QGridLayout *gridLayout;
    QVBoxLayout *verticalLayout;

    QLabel *labColor;
    QLineEdit *txtWeb;
    QLineEdit *txtRgb;
    QLineEdit *txtPoint;

private Q_SLOTS:
    void showColorValue();
    void buttonClicked();
};

#endif // COLORWIDGET_H
```

### `./frmcolorwidget.cpp`

```cpp
﻿#include "frmcolorwidget.h"
#include "ui_frmcolorwidget.h"
#include "colorwidget.h"

frmColorWidget::frmColorWidget(QWidget *parent) : QWidget(parent), ui(new Ui::frmColorWidget)
{
    ui->setupUi(this);
}

frmColorWidget::~frmColorWidget()
{
    delete ui;
}

void frmColorWidget::on_pushButton_clicked()
{
    ColorWidget::Instance()->show();
}
```

### `./frmcolorwidget.h`

```cpp
﻿#ifndef FRMCOLORWIDGET_H
#define FRMCOLORWIDGET_H

#include <QWidget>

namespace Ui {
class frmColorWidget;
}

class frmColorWidget : public QWidget
{
    Q_OBJECT

public:
    explicit frmColorWidget(QWidget *parent = 0);
    ~frmColorWidget();

private slots:
    void on_pushButton_clicked();

private:
    Ui::frmColorWidget *ui;
};

#endif // FRMCOLORWIDGET_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmcolorwidget.h"
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

    frmColorWidget w;
    w.setWindowTitle("屏幕拾色器 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmcolorwidget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmColorWidget</class>
 <widget class="QWidget" name="frmColorWidget">
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
