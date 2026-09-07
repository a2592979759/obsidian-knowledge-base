---
tags:
  - Qt
  - 工具相关
---
# 图片警告去除工具 (`pngtool`)

> 图片警告去除工具

## 效果图

![[QWidgetDemo_assert/pngtool.jpg]]

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/pngtool`

## 技术要点

**用到的 Qt 类**: QTextCodec QString QWidget QStringList QPushButton QFileDialog QLineEdit QApplication QTime QDir QImage QGridLayout QProgressBar QTextEdit QFont

**特性/模块**: Q_OBJECT

## 完整源码

### `./frmpngtool.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")
#include "frmpngtool.h"
#include "ui_frmpngtool.h"
#include "qfile.h"
#include "qfiledialog.h"
#include "qdatetime.h"
#include "qdebug.h"

#define TIMEMS QTime::currentTime().toString("hh:mm:ss zzz")

frmPngTool::frmPngTool(QWidget *parent) : QWidget(parent), ui(new Ui::frmPngTool)
{
    ui->setupUi(this);
    ui->progress->setRange(0, 0);
    ui->progress->setValue(0);
}

frmPngTool::~frmPngTool()
{
    delete ui;
}

void frmPngTool::on_btnFile_clicked()
{
    QString file = QFileDialog::getOpenFileName(this, "选择png文件", qApp->applicationDirPath(), "png图片文件(*.png)");
    if (!file.isEmpty()) {
        ui->txtFile->setText(file);
        ui->progress->setValue(0);
    }
}

void frmPngTool::on_btnDir_clicked()
{
    QString dir = QFileDialog::getExistingDirectory(this, "选择目录");
    if (!dir.isEmpty()) {
        ui->txtDir->setText(dir);
        ui->progress->setValue(0);
    }
}

void frmPngTool::on_btnOk_clicked()
{
    files.clear();

    //将单个文件加入队列
    QString currentFile = ui->txtFile->text().trimmed();
    if (!currentFile.isEmpty()) {
        files.append(currentFile);
    }

    //将该目录下的所有png文件存入链表
    QString currentDir = ui->txtDir->text().trimmed();
    if (!currentDir.isEmpty()) {
        QDir imagePath(currentDir);
        QStringList filter;
        filter << "*.png";
        QStringList list = imagePath.entryList(filter);
        foreach (QString str, list) {
            files.append(currentDir + "/" + str);
        }
    }

    ui->progress->setRange(0, files.count());
    ui->progress->setValue(0);

    ui->txtMain->clear();
    int count = 0;
    foreach (QString file, files) {
        ui->txtMain->append(QString("%1 -> %2").arg(TIMEMS).arg(file));
        QImage image(file);
        image.save(file, "png");
        count++;
        ui->progress->setValue(count);
        qApp->processEvents();
    }

    ui->txtMain->append(QString("%1 -> 处理完成, 共 %2 个文件").arg(TIMEMS).arg(files.count()));
}
```

### `./frmpngtool.h`

```cpp
﻿#ifndef FRMPNGTOOL_H
#define FRMPNGTOOL_H

#include <QWidget>

namespace Ui {
class frmPngTool;
}

class frmPngTool : public QWidget
{
    Q_OBJECT

public:
    explicit frmPngTool(QWidget *parent = 0);
    ~frmPngTool();

private slots:
    void on_btnFile_clicked();
    void on_btnDir_clicked();
    void on_btnOk_clicked();

private:
    Ui::frmPngTool *ui;
    QStringList files;
};

#endif // FRMPNGTOOL_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmpngtool.h"
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

    frmPngTool w;
    w.setWindowTitle("PNG图片警告去除工具 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmpngtool.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmPngTool</class>
 <widget class="QWidget" name="frmPngTool">
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
  <layout class="QGridLayout" name="gridLayout">
   <item row="0" column="0">
    <widget class="QLineEdit" name="txtFile"/>
   </item>
   <item row="2" column="1">
    <widget class="QPushButton" name="btnOk">
     <property name="text">
      <string>执行转换</string>
     </property>
    </widget>
   </item>
   <item row="1" column="0">
    <widget class="QLineEdit" name="txtDir"/>
   </item>
   <item row="1" column="1">
    <widget class="QPushButton" name="btnDir">
     <property name="text">
      <string>选择目录</string>
     </property>
    </widget>
   </item>
   <item row="0" column="1">
    <widget class="QPushButton" name="btnFile">
     <property name="text">
      <string>选择文件</string>
     </property>
    </widget>
   </item>
   <item row="2" column="0">
    <widget class="QProgressBar" name="progress">
     <property name="value">
      <number>0</number>
     </property>
     <property name="alignment">
      <set>Qt::AlignCenter</set>
     </property>
    </widget>
   </item>
   <item row="3" column="0" colspan="2">
    <widget class="QTextEdit" name="txtMain"/>
   </item>
  </layout>
 </widget>
 <resources/>
 <connections/>
</ui>
```
