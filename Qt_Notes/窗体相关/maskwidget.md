---
tags:
  - Qt
  - 窗体相关
---
# 通用遮罩层 (`maskwidget`)

> 通用遮罩层

## 效果图

![[QWidgetDemo_assert/maskwidget.jpg]]

## 分类

- 类型: 窗体相关 `widget`
- 源码目录: `widget/maskwidget`

## 技术要点

**用到的 Qt 类**: QWidget QTextCodec QEvent QStringList QColor QObject QTimer QApplication QScopedPointer QPalette QShowEvent QDialog QPushButton QFont QMutex QMutexLocker

**特性/模块**: Q_OBJECT

## 完整源码

### `./frmmaskwidget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmaskwidget.h"
#include "ui_frmmaskwidget.h"
#include "maskwidget.h"
#include "qdialog.h"
#include "qtimer.h"
#include "qdebug.h"

frmMaskWidget::frmMaskWidget(QWidget *parent) : QWidget(parent), ui(new Ui::frmMaskWidget)
{
    ui->setupUi(this);
    QTimer::singleShot(1000, this, SLOT(initForm()));
}

frmMaskWidget::~frmMaskWidget()
{
    delete ui;
}

void frmMaskWidget::initForm()
{
    MaskWidget::Instance()->setMainWidget(this->topLevelWidget());
    MaskWidget::Instance()->setDialogNames(QStringList() << "frmTest");
}

void frmMaskWidget::on_pushButton_clicked()
{
    QDialog d;
    d.setObjectName("frmTest");
    d.setWindowTitle("遮罩层弹出窗体");
    d.resize(400, 300);
    d.exec();
}
```

### `./frmmaskwidget.h`

```cpp
﻿#ifndef FRMMASKWIDGET_H
#define FRMMASKWIDGET_H

#include <QWidget>

namespace Ui {
class frmMaskWidget;
}

class frmMaskWidget : public QWidget
{
    Q_OBJECT

public:
    explicit frmMaskWidget(QWidget *parent = 0);
    ~frmMaskWidget();

private:
    Ui::frmMaskWidget *ui;

private slots:
    void initForm();
    void on_pushButton_clicked();
};

#endif // FRMMASKWIDGET_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmaskwidget.h"
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

    frmMaskWidget w;
    w.setWindowTitle("遮罩层窗体 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./maskwidget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "maskwidget.h"
#include "qmutex.h"
#include "qapplication.h"
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

QScopedPointer<MaskWidget> MaskWidget::self;
MaskWidget *MaskWidget::Instance()
{
    if (self.isNull()) {
        static QMutex mutex;
        QMutexLocker locker(&mutex);
        if (self.isNull()) {
            self.reset(new MaskWidget);
        }
    }

    return self.data();
}

MaskWidget::MaskWidget(QWidget *parent) : QWidget(parent)
{
    mainWidget = 0;
    setOpacity(0.7);
    setBgColor(QColor(0, 0, 0));

    //不设置主窗体则遮罩层大小为默认桌面大小
    this->setGeometry(deskGeometry);
    this->setWindowFlags(Qt::FramelessWindowHint | Qt::Tool);

    //绑定全局事件,过滤弹窗窗体进行处理
    qApp->installEventFilter(this);
}

void MaskWidget::setMainWidget(QWidget *mainWidget)
{
    if (this->mainWidget != mainWidget) {
        this->mainWidget = mainWidget;
    }
}

void MaskWidget::setDialogNames(const QStringList &dialogNames)
{
    if (this->dialogNames != dialogNames) {
        this->dialogNames = dialogNames;
    }
}

void MaskWidget::setOpacity(double opacity)
{
    this->setWindowOpacity(opacity);
}

void MaskWidget::setBgColor(const QColor &bgColor)
{
    QPalette palette = this->palette();
    palette.setBrush(QPalette::Window, bgColor);
    this->setPalette(palette);
}

void MaskWidget::showEvent(QShowEvent *)
{
    if (mainWidget) {
        this->setGeometry(mainWidget->geometry());
    }
}

bool MaskWidget::eventFilter(QObject *obj, QEvent *event)
{
    int type = event->type();
    if (type == QEvent::Show) {
        if (dialogNames.contains(obj->objectName())) {
            this->show();
            this->activateWindow();
            QWidget *w = (QWidget *)obj;
            w->activateWindow();
        }
    } else if (type == QEvent::Hide) {
        if (dialogNames.contains(obj->objectName())) {
            this->hide();
        }
    } else if (type == QEvent::WindowActivate) {
        //当主窗体激活时,同时激活遮罩层
        if (mainWidget) {
            if (obj->objectName() == mainWidget->objectName()) {
                if (this->isVisible()) {
                    this->activateWindow();
                }
            }
        }
    }

    return QObject::eventFilter(obj, event);
}
```

### `./maskwidget.h`

```cpp
﻿#ifndef MASKWIDGET_H
#define MASKWIDGET_H

/**
 * 弹窗遮罩层控件 作者:feiyangqingyun(QQ:517216493) 2016-12-26
 * 1. 可设置需要遮罩的主窗体，自动跟随主窗体位置显示遮罩面积。
 * 2. 只需要将弹窗窗体的名称一开始传入队列即可，足够简单。
 * 3. 可设置透明度。
 * 4. 可设置遮罩层颜色。
 * 5. 不阻塞消息循坏。
 */

#include <QWidget>

#ifdef quc
class Q_DECL_EXPORT MaskWidget : public QWidget
#else
class MaskWidget : public QWidget
#endif

{
    Q_OBJECT
public:
    static MaskWidget *Instance();
    explicit MaskWidget(QWidget *parent = 0);

protected:
    void showEvent(QShowEvent *);
    bool eventFilter(QObject *obj, QEvent *event);

private:
    static QScopedPointer<MaskWidget> self;

    //需要遮罩的主窗体
    QWidget *mainWidget;
    //需要弹窗的窗体对象名称集合链表
    QStringList dialogNames;

public Q_SLOTS:
    //设置需要遮罩的主窗体
    void setMainWidget(QWidget *mainWidget);
    //设置需要弹窗的窗体对象名称集合链表
    void setDialogNames(const QStringList &dialogNames);

    //设置遮罩颜色
    void setBgColor(const QColor &bgColor);
    //设置颜色透明度
    void setOpacity(double opacity);
};

#endif // MASKWIDGET_H
```

### `./frmmaskwidget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmMaskWidget</class>
 <widget class="QWidget" name="frmMaskWidget">
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
