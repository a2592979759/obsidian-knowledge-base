---
tags:
  - Qt
  - 第三方类
---
# 全局热键2 (`shortcut`)

> 全局热键2

## 效果图

![[QWidgetDemo_assert/shortcut.jpg]]

## 分类

- 类型: 第三方类 `third`
- 源码目录: `third/shortcut`

## 技术要点

**用到的 Qt 类**: QTextCodec QWidget QApplication QKeySequence QTime QVBoxLayout QLabel QFont

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmshortcut.cpp`

```cpp
﻿#include "frmshortcut.h"
#include "ui_frmshortcut.h"
#include "qxtglobalshortcut.h"
#include "qdatetime.h"
#include "qdebug.h"

frmShortCut::frmShortCut(QWidget *parent) : QWidget(parent), ui(new Ui::frmShortCut)
{
    ui->setupUi(this);
    this->initForm();
}

frmShortCut::~frmShortCut()
{
    delete ui;
}

void frmShortCut::initForm()
{
    //实例化热键类
    QxtGlobalShortcut *shortcut = new QxtGlobalShortcut(QKeySequence("ctrl+x"), this);
    connect(shortcut, SIGNAL(activated()), this, SLOT(shortcut()));
}

void frmShortCut::shortcut()
{
#if 1
    //如果是最小化则显示,否则最小化
    if (this->isMinimized()) {
        this->showNormal();
        this->activateWindow();
    } else {
        this->showMinimized();
    }
#else
    ui->label->setText("activated  " + QTime::currentTime().toString("hh:mm:ss zzz"));
#endif
}
```

### `./frmshortcut.h`

```cpp
﻿#ifndef FRMSHORTCUT_H
#define FRMSHORTCUT_H

#include <QWidget>

namespace Ui {
class frmShortCut;
}

class frmShortCut : public QWidget
{
    Q_OBJECT

public:
    explicit frmShortCut(QWidget *parent = 0);
    ~frmShortCut();

private:
    Ui::frmShortCut *ui;

private slots:
    void initForm();
    void shortcut();
};

#endif // FRMSHORTCUT_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmshortcut.h"
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

    frmShortCut w;
    w.setWindowTitle("全局热键示例 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmshortcut.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmShortCut</class>
 <widget class="QWidget" name="frmShortCut">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>全局热键示例</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QLabel" name="label">
     <property name="text">
      <string>按 ctrl+x 最小化,再次按显示</string>
     </property>
     <property name="alignment">
      <set>Qt::AlignCenter</set>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <resources/>
 <connections/>
</ui>
```
