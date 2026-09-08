---
tags:
  - Qt
  - 工具相关
  - 高质量
---
# 秘钥生成器 (`keytool`)

> 秘钥生成器

## 效果图

![[QWidgetDemo_assert/keytool.jpg]]

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/keytool`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QString QTextCodec QWidget QMessageBox QApplication QCheckBox QStringList QFile QPushButton QComboBox QDate QProcess QByteArray QLatin1String QIODevice QGridLayout QDateEdit QLabel QFont

**特性/模块**: Q_OBJECT

## 完整源码

### `./frmmain.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
#include "ui_frmmain.h"
#include "qmessagebox.h"
#include "qfile.h"
#include "qprocess.h"
#include "qdebug.h"

frmMain::frmMain(QWidget *parent) : QWidget(parent), ui(new Ui::frmMain)
{
    ui->setupUi(this);
    this->initForm();
    qDebug() << this->getCpuName() << this->getCpuId() << this->getDiskNum();
}

frmMain::~frmMain()
{
    delete ui;
}

void frmMain::initForm()
{
    QStringList min;
    min << "1" << "5" << "10" << "20" << "30";
    for (int i = 1; i <= 24; i++) {
        min << QString::number(i * 60);
    }

    ui->cboxMin->addItems(min);
    ui->cboxMin->setCurrentIndex(1);
    ui->dateEdit->setDate(QDate::currentDate());

    for (int i = 5; i <= 150; i = i + 5) {
        ui->cboxCount->addItem(QString("%1").arg(i));
    }
}

QString frmMain::getWMIC(const QString &cmd)
{
    //获取cpu名称：wmic cpu get Name
    //获取cpu核心数：wmic cpu get NumberOfCores
    //获取cpu线程数：wmic cpu get NumberOfLogicalProcessors
    //查询cpu序列号：wmic cpu get processorid
    //查询主板序列号：wmic baseboard get serialnumber
    //查询BIOS序列号：wmic bios get serialnumber
    //查看硬盘：wmic diskdrive get serialnumber
    QProcess p;
    p.start(cmd);
    p.waitForFinished();
    QString result = QString::fromLocal8Bit(p.readAllStandardOutput());
    QStringList list = cmd.split(" ");
    result = result.remove(list.last(), Qt::CaseInsensitive);
    result = result.replace("\r", "");
    result = result.replace("\n", "");
    result = result.simplified();
    return result;
}

QString frmMain::getCpuName()
{
    return getWMIC("wmic cpu get name");
}

QString frmMain::getCpuId()
{
    return getWMIC("wmic cpu get processorid");
}

QString frmMain::getDiskNum()
{
    return getWMIC("wmic diskdrive where index=0 get serialnumber");
}

QString frmMain::getXorEncryptDecrypt(const QString &data, char key)
{
    //采用异或加密,也可以自行更改算法
    QByteArray buffer = data.toLatin1();
    int size = buffer.size();
    for (int i = 0; i < size; i++) {
        buffer[i] = buffer.at(i) ^ key;
    }

    return QLatin1String(buffer);
}

void frmMain::on_btnOk_clicked()
{
    bool useDate = ui->ckDate->isChecked();
    bool useRun = ui->ckRun->isChecked();
    bool useCount = ui->ckCount->isChecked();

    if (!useDate && !useRun && !useCount) {
        if (QMessageBox::question(this, "询问", "确定要生成没有任何限制的密钥吗?") != QMessageBox::Yes) {
            return;
        }
    }

    QString strDate = ui->dateEdit->date().toString("yyyy-MM-dd");
    QString strRun = ui->cboxMin->currentText();
    QString strCount = ui->cboxCount->currentText();
    QString key = QString("%1|%2|%3|%4|%5|%6").arg(useDate).arg(strDate).arg(useRun).arg(strRun).arg(useCount).arg(strCount);

    QFile file(QApplication::applicationDirPath() + "/key.db");
    file.open(QFile::WriteOnly | QIODevice::Text);
    file.write(getXorEncryptDecrypt(key, 110).toLatin1());
    file.close();
    QMessageBox::information(this, "提示", "生成密钥成功,将 key.db 文件拷贝到对应目录即可!");
}

void frmMain::on_btnClose_clicked()
{
    this->close();
}
```

### `./frmmain.h`

```cpp
﻿#ifndef FRMMAIN_H
#define FRMMAIN_H

#include <QWidget>

namespace Ui {
class frmMain;
}

class frmMain : public QWidget
{
    Q_OBJECT

public:
    explicit frmMain(QWidget *parent = 0);
    ~frmMain();

private:
    Ui::frmMain *ui;

private slots:
    void initForm();
    QString getWMIC(const QString &cmd);
    QString getCpuName();
    QString getCpuId();
    QString getDiskNum();
    QString getXorEncryptDecrypt(const QString &data, char key);

private slots:
    void on_btnOk_clicked();
    void on_btnClose_clicked();
};

#endif // FRMMAIN_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
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

    frmMain w;
    w.setWindowTitle("密钥生成工具 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmmain.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmMain</class>
 <widget class="QWidget" name="frmMain">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <layout class="QGridLayout" name="gridLayout">
   <item row="0" column="1" rowspan="2" colspan="2">
    <widget class="QDateEdit" name="dateEdit">
     <property name="focusPolicy">
      <enum>Qt::NoFocus</enum>
     </property>
     <property name="displayFormat">
      <string>yyyy年MM月dd日</string>
     </property>
     <property name="calendarPopup">
      <bool>true</bool>
     </property>
    </widget>
   </item>
   <item row="0" column="3" rowspan="2">
    <widget class="QPushButton" name="btnOk">
     <property name="minimumSize">
      <size>
       <width>80</width>
       <height>0</height>
      </size>
     </property>
     <property name="focusPolicy">
      <enum>Qt::NoFocus</enum>
     </property>
     <property name="text">
      <string>生成</string>
     </property>
     <property name="iconSize">
      <size>
       <width>20</width>
       <height>20</height>
      </size>
     </property>
    </widget>
   </item>
   <item row="1" column="0">
    <widget class="QCheckBox" name="ckDate">
     <property name="focusPolicy">
      <enum>Qt::NoFocus</enum>
     </property>
     <property name="text">
      <string>时间限制</string>
     </property>
    </widget>
   </item>
   <item row="2" column="0">
    <widget class="QCheckBox" name="ckCount">
     <property name="focusPolicy">
      <enum>Qt::NoFocus</enum>
     </property>
     <property name="text">
      <string>数量限制</string>
     </property>
    </widget>
   </item>
   <item row="2" column="1" colspan="2">
    <widget class="QComboBox" name="cboxCount">
     <property name="editable">
      <bool>true</bool>
     </property>
    </widget>
   </item>
   <item row="2" column="3">
    <widget class="QPushButton" name="btnClose">
     <property name="minimumSize">
      <size>
       <width>80</width>
       <height>0</height>
      </size>
     </property>
     <property name="focusPolicy">
      <enum>Qt::NoFocus</enum>
     </property>
     <property name="text">
      <string>关闭</string>
     </property>
     <property name="iconSize">
      <size>
       <width>20</width>
       <height>20</height>
      </size>
     </property>
    </widget>
   </item>
   <item row="3" column="0">
    <widget class="QCheckBox" name="ckRun">
     <property name="focusPolicy">
      <enum>Qt::NoFocus</enum>
     </property>
     <property name="text">
      <string>运行限制</string>
     </property>
    </widget>
   </item>
   <item row="3" column="1">
    <widget class="QComboBox" name="cboxMin">
     <property name="editable">
      <bool>true</bool>
     </property>
    </widget>
   </item>
   <item row="3" column="2">
    <widget class="QLabel" name="label">
     <property name="text">
      <string>分钟自动关闭</string>
     </property>
    </widget>
   </item>
   <item row="4" column="0">
    <spacer name="verticalSpacer">
     <property name="orientation">
      <enum>Qt::Vertical</enum>
     </property>
     <property name="sizeHint" stdset="0">
      <size>
       <width>20</width>
       <height>24</height>
      </size>
     </property>
    </spacer>
   </item>
   <item row="0" column="4" rowspan="5">
    <spacer name="horizontalSpacer">
     <property name="orientation">
      <enum>Qt::Horizontal</enum>
     </property>
     <property name="sizeHint" stdset="0">
      <size>
       <width>1</width>
       <height>108</height>
      </size>
     </property>
    </spacer>
   </item>
  </layout>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <resources/>
 <connections/>
</ui>
```
