---
tags:
  - Qt
  - 工具相关
  - 高质量
---
# 图片文字转base (`base64helper`)

> 图片文字转base

## 效果图

![[QWidgetDemo_assert/base64helper.jpg]]

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/base64helper`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QString QImage QTextCodec QByteArray QWidget QPushButton QFrame QPixmap QSize QFile QElapsedTimer QLineEdit QLabel QApplication QBuffer QFileDialog QGridLayout QTextEdit QFont

**特性/模块**: Q_OBJECT

## 完整源码

### `./base64helper.cpp`

```cpp
﻿#include "base64helper.h"
#include "qbuffer.h"
#include "qdebug.h"

QString Base64Helper::imageToBase64(const QImage &image)
{
    return QString(imageToBase64x(image));
}

QByteArray Base64Helper::imageToBase64x(const QImage &image)
{
    //这个转换可能比较耗时建议在线程中执行
    QByteArray data;
    QBuffer buffer(&data);
    image.save(&buffer, "JPG");
    data = data.toBase64();
    return data;
}

QImage Base64Helper::base64ToImage(const QString &data)
{
    return base64ToImagex(data.toUtf8());
}

QImage Base64Helper::base64ToImagex(const QByteArray &data)
{
    //这个转换可能比较耗时建议在线程中执行
    QImage image;
    image.loadFromData(QByteArray::fromBase64(data));
    return image;
}

QString Base64Helper::textToBase64(const QString &text)
{
    return QString(text.toUtf8().toBase64());
}

QString Base64Helper::base64ToText(const QString &text)
{
    return QString(QByteArray::fromBase64(text.toUtf8()));
}
```

### `./base64helper.h`

```cpp
﻿#ifndef BASE64HELPER_H
#define BASE64HELPER_H

/**
 * base64编码转换类 作者:feiyangqingyun(QQ:517216493) 2016-12-16
 * 1. 图片转base64字符串。
 * 2. base64字符串转图片。
 * 3. 字符转base64字符串。
 * 4. base64字符串转字符。
 * 5. 后期增加数据压缩。
 * 6. Qt6对base64编码转换进行了重写效率提升至少200%。
 */

#include <QImage>

#ifdef quc
class Q_DECL_EXPORT Base64Helper
#else
class Base64Helper
#endif

{
public:
    //图片转base64字符串
    static QString imageToBase64(const QImage &image);
    static QByteArray imageToBase64x(const QImage &image);

    //base64字符串转图片
    static QImage base64ToImage(const QString &data);
    static QImage base64ToImagex(const QByteArray &data);

    //字符串与base64互转
    static QString textToBase64(const QString &text);
    static QString base64ToText(const QString &text);
};

#endif // BASE64HELPER_H
```

### `./frmbase64helper.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmbase64helper.h"
#include "ui_frmbase64helper.h"
#include "base64helper.h"
#include "qfiledialog.h"
#include "qdebug.h"

frmBase64Helper::frmBase64Helper(QWidget *parent) : QWidget(parent), ui(new Ui::frmBase64Helper)
{
    ui->setupUi(this);
}

frmBase64Helper::~frmBase64Helper()
{
    delete ui;
}

void frmBase64Helper::showTime(qint64 size1, qint64 size2)
{
    //统计用时
#if (QT_VERSION >= QT_VERSION_CHECK(4,8,0))
    double elapsed = (double)timer.nsecsElapsed() / 1000000;
#else
    double elapsed = (double)timer.elapsed();
#endif
    QString time = QString::number(elapsed, 'f', 3);
    ui->labInfo->setText(QString("用时: %1 毫秒  大小: %2 -> %3").arg(time).arg(size1).arg(size2));
}

void frmBase64Helper::on_btnOpen_clicked()
{
    QString fileName = QFileDialog::getOpenFileName(this, "选择文件", "", "图片(*.png *.bmp *.jpg)");
    if (!fileName.isEmpty()) {
        ui->txtFile->setText(fileName);
        QPixmap pix(fileName);
        pix = pix.scaled(ui->labImage->size() - QSize(4, 4), Qt::KeepAspectRatio);
        ui->labImage->setPixmap(pix);
    }
}

void frmBase64Helper::on_btnClear_clicked()
{
    ui->txtFile->clear();
    ui->txtText->clear();
    ui->txtBase64->clear();
    ui->labImage->clear();
}

void frmBase64Helper::on_btnImageToBase64_clicked()
{
    QString fileName = ui->txtFile->text().trimmed();
    if (fileName.isEmpty()) {
        return;
    }

    timer.restart();
    QImage image(fileName);
    QString text = Base64Helper::imageToBase64(image);
    showTime(QFile(fileName).size(), text.size());
    ui->txtBase64->setText(text);
}

void frmBase64Helper::on_btnBase64ToImage_clicked()
{
    QString fileName = ui->txtFile->text().trimmed();
    QString text = ui->txtBase64->toPlainText().trimmed();
    if (text.isEmpty()) {
        return;
    }

    timer.restart();
    QImage image = Base64Helper::base64ToImage(text);
    showTime(text.size(), QFile(fileName).size());
    QPixmap pix = QPixmap::fromImage(image);
    pix = pix.scaled(ui->labImage->size() - QSize(4, 4), Qt::KeepAspectRatio);
    ui->labImage->setPixmap(pix);
}

void frmBase64Helper::on_btnTextToBase64_clicked()
{
    QString text = ui->txtText->text().trimmed();
    if (text.isEmpty()) {
        return;
    }

    timer.restart();
    QString result = Base64Helper::textToBase64(text);
    showTime(text.size(), result.size());
    ui->txtBase64->setText(result);
}

void frmBase64Helper::on_btnBase64ToText_clicked()
{
    QString text = ui->txtBase64->toPlainText().trimmed();
    if (text.isEmpty()) {
        return;
    }

    timer.restart();
    QString result = Base64Helper::base64ToText(text);
    showTime(text.size(), result.size());
    ui->txtText->setText(result);
}
```

### `./frmbase64helper.h`

```cpp
﻿#ifndef FRMBASE64HELPER_H
#define FRMBASE64HELPER_H

#include <QWidget>
#include <QElapsedTimer>

namespace Ui {
class frmBase64Helper;
}

class frmBase64Helper : public QWidget
{
    Q_OBJECT

public:
    explicit frmBase64Helper(QWidget *parent = 0);
    ~frmBase64Helper();

private:
    Ui::frmBase64Helper *ui;
    QElapsedTimer timer;

private slots:
    void showTime(qint64 size1, qint64 size2);
    void on_btnOpen_clicked();
    void on_btnClear_clicked();

    void on_btnImageToBase64_clicked();
    void on_btnBase64ToImage_clicked();

    void on_btnTextToBase64_clicked();
    void on_btnBase64ToText_clicked();
};

#endif // FRMBASE64HELPER_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmbase64helper.h"
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

    frmBase64Helper w;
    w.setWindowTitle("图片文字base64编码互换 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmbase64helper.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmBase64Helper</class>
 <widget class="QWidget" name="frmBase64Helper">
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
   <item row="0" column="0">
    <widget class="QLineEdit" name="txtFile">
     <property name="text">
      <string>E:/myFile/美女图片/2.jpg</string>
     </property>
    </widget>
   </item>
   <item row="0" column="1">
    <widget class="QPushButton" name="btnOpen">
     <property name="minimumSize">
      <size>
       <width>120</width>
       <height>0</height>
      </size>
     </property>
     <property name="text">
      <string>打开文件</string>
     </property>
    </widget>
   </item>
   <item row="0" column="2">
    <widget class="QPushButton" name="btnImageToBase64">
     <property name="minimumSize">
      <size>
       <width>120</width>
       <height>0</height>
      </size>
     </property>
     <property name="text">
      <string>图片转base64</string>
     </property>
    </widget>
   </item>
   <item row="0" column="3">
    <widget class="QPushButton" name="btnBase64ToImage">
     <property name="minimumSize">
      <size>
       <width>120</width>
       <height>0</height>
      </size>
     </property>
     <property name="text">
      <string>base64转图片</string>
     </property>
    </widget>
   </item>
   <item row="1" column="0">
    <widget class="QLineEdit" name="txtText">
     <property name="text">
      <string>游龙 feiyangqingyun QQ: 517216493</string>
     </property>
    </widget>
   </item>
   <item row="1" column="1">
    <widget class="QPushButton" name="btnClear">
     <property name="minimumSize">
      <size>
       <width>120</width>
       <height>0</height>
      </size>
     </property>
     <property name="text">
      <string>清空数据</string>
     </property>
    </widget>
   </item>
   <item row="1" column="2">
    <widget class="QPushButton" name="btnTextToBase64">
     <property name="minimumSize">
      <size>
       <width>120</width>
       <height>0</height>
      </size>
     </property>
     <property name="text">
      <string>文字转base64</string>
     </property>
    </widget>
   </item>
   <item row="1" column="3">
    <widget class="QPushButton" name="btnBase64ToText">
     <property name="minimumSize">
      <size>
       <width>120</width>
       <height>0</height>
      </size>
     </property>
     <property name="text">
      <string>base64转文字</string>
     </property>
    </widget>
   </item>
   <item row="2" column="0">
    <widget class="QLabel" name="labInfo">
     <property name="frameShape">
      <enum>QFrame::Box</enum>
     </property>
     <property name="frameShadow">
      <enum>QFrame::Sunken</enum>
     </property>
     <property name="text">
      <string/>
     </property>
    </widget>
   </item>
   <item row="3" column="0" colspan="4">
    <widget class="QLabel" name="labImage">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Preferred" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
     <property name="frameShape">
      <enum>QFrame::Box</enum>
     </property>
     <property name="frameShadow">
      <enum>QFrame::Sunken</enum>
     </property>
     <property name="text">
      <string/>
     </property>
     <property name="alignment">
      <set>Qt::AlignCenter</set>
     </property>
    </widget>
   </item>
   <item row="4" column="0" colspan="4">
    <widget class="QTextEdit" name="txtBase64"/>
   </item>
  </layout>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <tabstops>
  <tabstop>txtFile</tabstop>
  <tabstop>txtText</tabstop>
  <tabstop>btnOpen</tabstop>
  <tabstop>btnClear</tabstop>
  <tabstop>btnImageToBase64</tabstop>
  <tabstop>btnBase64ToImage</tabstop>
  <tabstop>btnTextToBase64</tabstop>
  <tabstop>btnBase64ToText</tabstop>
  <tabstop>txtBase64</tabstop>
 </tabstops>
 <resources/>
 <connections/>
</ui>
```
