---
tags:
  - Qt
  - 第三方类
---
# 全局热键1 (`hotkey`)

> 全局热键1

## 效果图

![[QWidgetDemo_assert/hotkey.jpg]]

## 分类

- 类型: 第三方类 `third`
- 源码目录: `third/hotkey`

## 技术要点

**用到的 Qt 类**: QTextCodec QWidget QHotkey QApplication QKeySequence QTime QVBoxLayout QLabel QFont

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmhotkey.cpp`

```cpp
﻿#include "frmhotkey.h"
#include "ui_frmhotkey.h"
#include "qhotkey.h"
#include "qdatetime.h"
#include "qdebug.h"

frmHotKey::frmHotKey(QWidget *parent) : QWidget(parent), ui(new Ui::frmHotKey)
{
    ui->setupUi(this);
    this->initForm();
}

frmHotKey::~frmHotKey()
{
    delete ui;
}

void frmHotKey::initForm()
{
    //this->setWindowFlags(Qt::FramelessWindowHint);

    //实例化热键类 支持各种组合形式比如 ctrl+a alt+a f2
    QHotkey *hotkey = new QHotkey(QKeySequence("ctrl+x"), true, this);
    connect(hotkey, SIGNAL(activated()), this, SLOT(shortcut()));
}

void frmHotKey::shortcut()
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

### `./frmhotkey.h`

```cpp
﻿#ifndef FRMHOTKEY_H
#define FRMHOTKEY_H

#include <QWidget>

namespace Ui {
class frmHotKey;
}

class frmHotKey : public QWidget
{
    Q_OBJECT

public:
    explicit frmHotKey(QWidget *parent = 0);
    ~frmHotKey();

private:
    Ui::frmHotKey *ui;

private slots:
    void initForm();
    void shortcut();
};

#endif // FRMHOTKEY_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmhotkey.h"
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

    frmHotKey w;
    w.setWindowTitle("全局热键示例 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmhotkey.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmHotKey</class>
 <widget class="QWidget" name="frmHotKey">
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
