---
tags:
  - Qt
  - 工具相关
  - 高质量
---
# 代码行数统计 (`countcode`)

> 代码行数统计工具

## 效果图

![[QWidgetDemo_assert/countcode.jpg]]

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/countcode`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QString QTableWidgetItem QLabel QTextCodec QLineEdit QWidget QStringList QFrame QFileDialog QAbstractItemView QFileInfo QChar QPushButton QFont QFile QGridLayout QApplication QList QDir QFileInfoList

**特性/模块**: Q_OBJECT

## 完整源码

### `./frmcountcode.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmcountcode.h"
#include "ui_frmcountcode.h"
#include "qfile.h"
#include "qtextstream.h"
#include "qfiledialog.h"
#include "qfileinfo.h"
#include "qdebug.h"

frmCountCode::frmCountCode(QWidget *parent) : QWidget(parent), ui(new Ui::frmCountCode)
{
    ui->setupUi(this);
    this->initForm();
    on_btnClear_clicked();
}

frmCountCode::~frmCountCode()
{
    delete ui;
}

void frmCountCode::initForm()
{
    QStringList headText;
    headText << "文件名" << "类型" << "大小" << "总行数" << "代码行数" << "注释行数" << "空白行数" << "路径";
    QList<int> columnWidth;
    columnWidth << 130 << 50 << 70 << 80 << 70 << 70 << 70 << 150;

    int columnCount = headText.count();
    ui->tableWidget->setColumnCount(columnCount);
    ui->tableWidget->setHorizontalHeaderLabels(headText);
    ui->tableWidget->setSelectionBehavior(QAbstractItemView::SelectRows);
    ui->tableWidget->setEditTriggers(QAbstractItemView::NoEditTriggers);
    ui->tableWidget->setSelectionMode(QAbstractItemView::SingleSelection);
    ui->tableWidget->verticalHeader()->setVisible(false);
    ui->tableWidget->horizontalHeader()->setStretchLastSection(true);
    ui->tableWidget->horizontalHeader()->setHighlightSections(false);
    ui->tableWidget->verticalHeader()->setDefaultSectionSize(20);
    ui->tableWidget->verticalHeader()->setHighlightSections(false);

    for (int i = 0; i < columnCount; i++) {
        ui->tableWidget->setColumnWidth(i, columnWidth.at(i));
    }

    //设置前景色
    ui->txtCount->setStyleSheet("color:#17A086;");
    ui->txtSize->setStyleSheet("color:#CA5AA6;");
    ui->txtRow->setStyleSheet("color:#CD1B19;");
    ui->txtCode->setStyleSheet("color:#22A3A9;");
    ui->txtNote->setStyleSheet("color:#D64D54;");
    ui->txtBlank->setStyleSheet("color:#A279C5;");

    //设置字体加粗
    QFont font;
    font.setBold(true);
    if (font.pointSize() > 0) {
        font.setPointSize(font.pointSize() + 1);
    } else {
        font.setPixelSize(font.pixelSize() + 2);
    }

    ui->txtCount->setFont(font);
    ui->txtSize->setFont(font);
    ui->txtRow->setFont(font);
    ui->txtCode->setFont(font);
    ui->txtNote->setFont(font);
    ui->txtBlank->setFont(font);

#if (QT_VERSION > QT_VERSION_CHECK(4,7,0))
    ui->txtFilter->setPlaceholderText("中间空格隔开,例如 *.h *.cpp *.c");
#endif
}

bool frmCountCode::checkFile(const QString &fileName)
{
    if (fileName.startsWith("moc_") || fileName.startsWith("ui_") || fileName.startsWith("qrc_")) {
        return false;
    }

    QFileInfo file(fileName);
    QString suffix = "*." + file.suffix();
    QString filter = ui->txtFilter->text().trimmed();
    QStringList filters = filter.split(" ");
    return filters.contains(suffix);
}

void frmCountCode::countCode(const QString &filePath)
{
    QDir dir(filePath);
    QFileInfoList fileInfos = dir.entryInfoList();
    foreach (QFileInfo fileInfo, fileInfos) {
        QString fileName = fileInfo.fileName();
        if (fileInfo.isFile()) {
            if (checkFile(fileName)) {
                listFile << fileInfo.filePath();
            }
        } else {
            if (fileName == "." || fileName == "..") {
                continue;
            }

            //递归找出文件
            countCode(fileInfo.absoluteFilePath());
        }
    }
}

void frmCountCode::countCode(const QStringList &files)
{
    int lineCode;
    int lineBlank;
    int lineNotes;
    int count = files.count();
    on_btnClear_clicked();
    ui->tableWidget->setRowCount(count);

    quint32 totalLines = 0;
    quint32 totalBytes = 0;
    quint32 totalCodes = 0;
    quint32 totalNotes = 0;
    quint32 totalBlanks = 0;

    for (int i = 0; i < count; i++) {
        QFileInfo fileInfo(files.at(i));
        countCode(fileInfo.filePath(), lineCode, lineBlank, lineNotes);
        int lineAll = lineCode + lineBlank + lineNotes;

        QTableWidgetItem *itemName = new QTableWidgetItem;
        itemName->setText(fileInfo.fileName());

        QTableWidgetItem *itemSuffix = new QTableWidgetItem;
        itemSuffix->setText(fileInfo.suffix());

        QTableWidgetItem *itemSize = new QTableWidgetItem;
        itemSize->setText(QString::number(fileInfo.size()));

        QTableWidgetItem *itemLine = new QTableWidgetItem;
        itemLine->setText(QString::number(lineAll));

        QTableWidgetItem *itemCode = new QTableWidgetItem;
        itemCode->setText(QString::number(lineCode));

        QTableWidgetItem *itemNote = new QTableWidgetItem;
        itemNote->setText(QString::number(lineNotes));

        QTableWidgetItem *itemBlank = new QTableWidgetItem;
        itemBlank->setText(QString::number(lineBlank));

        QTableWidgetItem *itemPath = new QTableWidgetItem;
        itemPath->setText(fileInfo.filePath());

        itemSuffix->setTextAlignment(Qt::AlignCenter);
        itemSize->setTextAlignment(Qt::AlignCenter);
        itemLine->setTextAlignment(Qt::AlignCenter);
        itemCode->setTextAlignment(Qt::AlignCenter);
        itemNote->setTextAlignment(Qt::AlignCenter);
        itemBlank->setTextAlignment(Qt::AlignCenter);

        ui->tableWidget->setItem(i, 0, itemName);
        ui->tableWidget->setItem(i, 1, itemSuffix);
        ui->tableWidget->setItem(i, 2, itemSize);
        ui->tableWidget->setItem(i, 3, itemLine);
        ui->tableWidget->setItem(i, 4, itemCode);
        ui->tableWidget->setItem(i, 5, itemNote);
        ui->tableWidget->setItem(i, 6, itemBlank);
        ui->tableWidget->setItem(i, 7, itemPath);

        totalBytes  += fileInfo.size();
        totalLines  += lineAll;
        totalCodes  += lineCode;
        totalNotes  += lineNotes;
        totalBlanks += lineBlank;

        if (i % 100 == 0) {
            qApp->processEvents();
        }
    }

    //显示统计结果
    listFile.clear();
    ui->txtCount->setText(QString::number(count));
    ui->txtSize->setText(QString::number(totalBytes));
    ui->txtRow->setText(QString::number(totalLines));
    ui->txtCode->setText(QString::number(totalCodes));
    ui->txtNote->setText(QString::number(totalNotes));
    ui->txtBlank->setText(QString::number(totalBlanks));

    //计算百分比
    double percent = 0.0;
    //代码行所占百分比
    percent = ((double)totalCodes / totalLines) * 100;
    ui->labPercentCode->setText(QString("%1%").arg(percent, 5, 'f', 2, QChar(' ')));
    //注释行所占百分比
    percent = ((double)totalNotes / totalLines) * 100;
    ui->labPercentNote->setText(QString("%1%").arg(percent, 5, 'f', 2, QChar(' ')));
    //空行所占百分比
    percent = ((double)totalBlanks / totalLines) * 100;
    ui->labPercentBlank->setText(QString("%1%").arg(percent, 5, 'f', 2, QChar(' ')));
}

void frmCountCode::countCode(const QString &fileName, int &lineCode, int &lineBlank, int &lineNotes)
{
    lineCode = lineBlank = lineNotes = 0;
    QFile file(fileName);
    if (file.open(QFile::ReadOnly)) {
        QTextStream out(&file);
        QString line;
        bool isNote = false;
        while (!out.atEnd()) {
            line = out.readLine();

            //移除前面的空行
            if (line.startsWith(" ")) {
                line.remove(" ");
            }

            //判断当前行是否是注释
            if (line.startsWith("/*")) {
                isNote = true;
            }

            //注释部分
            if (isNote) {
                lineNotes++;
            } else {
                if (line.startsWith("//")) {    //注释行
                    lineNotes++;
                } else if (line.isEmpty()) {    //空白行
                    lineBlank++;
                } else {                        //代码行
                    lineCode++;
                }
            }

            //注释结束
            if (line.endsWith("*/")) {
                isNote = false;
            }
        }
    }
}

void frmCountCode::on_btnOpenFile_clicked()
{
    QString filter = QString("代码文件(%1)").arg(ui->txtFilter->text().trimmed());
    QStringList files = QFileDialog::getOpenFileNames(this, "选择文件", "./", filter);
    if (files.size() > 0) {
        ui->txtFile->setText(files.join("|"));
        countCode(files);
    }
}

void frmCountCode::on_btnOpenPath_clicked()
{
    QString path = QFileDialog::getExistingDirectory(this, "选择目录", "./",  QFileDialog::ShowDirsOnly | QFileDialog::DontResolveSymlinks);
    if (!path.isEmpty()) {
        ui->txtPath->setText(path);
        listFile.clear();
        countCode(path);
        countCode(listFile);
    }
}

void frmCountCode::on_btnClear_clicked()
{
    ui->txtCount->setText("0");
    ui->txtSize->setText("0");
    ui->txtRow->setText("0");

    ui->txtCode->setText("0");
    ui->txtNote->setText("0");
    ui->txtBlank->setText("0");

    ui->labPercentCode->setText("0%");
    ui->labPercentNote->setText("0%");
    ui->labPercentBlank->setText("0%");
    ui->tableWidget->setRowCount(0);
}
```

### `./frmcountcode.h`

```cpp
﻿#ifndef FRMCOUNTCODE_H
#define FRMCOUNTCODE_H

#include <QWidget>

namespace Ui {
class frmCountCode;
}

class frmCountCode : public QWidget
{
    Q_OBJECT

public:
    explicit frmCountCode(QWidget *parent = 0);
    ~frmCountCode();

private:
    Ui::frmCountCode *ui;
    QStringList listFile;

private:
    void initForm();
    bool checkFile(const QString &fileName);
    void countCode(const QString &filePath);
    void countCode(const QStringList &files);
    void countCode(const QString &fileName, int &lineCode, int &lineBlank, int &lineNotes);

private slots:
    void on_btnOpenFile_clicked();
    void on_btnOpenPath_clicked();
    void on_btnClear_clicked();
};

#endif // FRMCOUNTCODE_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmcountcode.h"
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

    frmCountCode w;
    w.setWindowTitle("代码行数统计 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmcountcode.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmCountCode</class>
 <widget class="QWidget" name="frmCountCode">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout_2">
   <item>
    <widget class="QTableWidget" name="tableWidget"/>
   </item>
   <item>
    <widget class="QWidget" name="widget" native="true">
     <layout class="QGridLayout" name="gridLayout_2">
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
      <item row="0" column="0" rowspan="3">
       <layout class="QGridLayout" name="gridLayout">
        <item row="0" column="3">
         <widget class="QLineEdit" name="txtCode">
          <property name="maximumSize">
           <size>
            <width>80</width>
            <height>16777215</height>
           </size>
          </property>
          <property name="alignment">
           <set>Qt::AlignCenter</set>
          </property>
          <property name="readOnly">
           <bool>true</bool>
          </property>
         </widget>
        </item>
        <item row="2" column="1">
         <widget class="QLineEdit" name="txtRow">
          <property name="sizePolicy">
           <sizepolicy hsizetype="Ignored" vsizetype="Fixed">
            <horstretch>0</horstretch>
            <verstretch>0</verstretch>
           </sizepolicy>
          </property>
          <property name="alignment">
           <set>Qt::AlignCenter</set>
          </property>
          <property name="readOnly">
           <bool>true</bool>
          </property>
         </widget>
        </item>
        <item row="1" column="3">
         <widget class="QLineEdit" name="txtNote">
          <property name="sizePolicy">
           <sizepolicy hsizetype="Ignored" vsizetype="Fixed">
            <horstretch>0</horstretch>
            <verstretch>0</verstretch>
           </sizepolicy>
          </property>
          <property name="alignment">
           <set>Qt::AlignCenter</set>
          </property>
          <property name="readOnly">
           <bool>true</bool>
          </property>
         </widget>
        </item>
        <item row="2" column="3">
         <widget class="QLineEdit" name="txtBlank">
          <property name="sizePolicy">
           <sizepolicy hsizetype="Ignored" vsizetype="Fixed">
            <horstretch>0</horstretch>
            <verstretch>0</verstretch>
           </sizepolicy>
          </property>
          <property name="alignment">
           <set>Qt::AlignCenter</set>
          </property>
          <property name="readOnly">
           <bool>true</bool>
          </property>
         </widget>
        </item>
        <item row="0" column="1">
         <widget class="QLineEdit" name="txtCount">
          <property name="maximumSize">
           <size>
            <width>80</width>
            <height>16777215</height>
           </size>
          </property>
          <property name="alignment">
           <set>Qt::AlignCenter</set>
          </property>
          <property name="readOnly">
           <bool>true</bool>
          </property>
         </widget>
        </item>
        <item row="0" column="4">
         <widget class="QLabel" name="labPercentCode">
          <property name="minimumSize">
           <size>
            <width>60</width>
            <height>0</height>
           </size>
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
        <item row="2" column="2">
         <widget class="QLabel" name="labBlank">
          <property name="text">
           <string>空白行数</string>
          </property>
         </widget>
        </item>
        <item row="1" column="1">
         <widget class="QLineEdit" name="txtSize">
          <property name="sizePolicy">
           <sizepolicy hsizetype="Ignored" vsizetype="Fixed">
            <horstretch>0</horstretch>
            <verstretch>0</verstretch>
           </sizepolicy>
          </property>
          <property name="alignment">
           <set>Qt::AlignCenter</set>
          </property>
          <property name="readOnly">
           <bool>true</bool>
          </property>
         </widget>
        </item>
        <item row="2" column="5">
         <widget class="QLabel" name="labFilter">
          <property name="text">
           <string>过滤</string>
          </property>
         </widget>
        </item>
        <item row="0" column="5">
         <widget class="QLabel" name="labFile">
          <property name="text">
           <string>文件</string>
          </property>
         </widget>
        </item>
        <item row="1" column="6">
         <widget class="QLineEdit" name="txtPath">
          <property name="readOnly">
           <bool>true</bool>
          </property>
         </widget>
        </item>
        <item row="0" column="6">
         <widget class="QLineEdit" name="txtFile">
          <property name="readOnly">
           <bool>true</bool>
          </property>
         </widget>
        </item>
        <item row="1" column="5">
         <widget class="QLabel" name="labPath">
          <property name="text">
           <string>目录</string>
          </property>
         </widget>
        </item>
        <item row="2" column="6">
         <widget class="QLineEdit" name="txtFilter">
          <property name="text">
           <string>*.h *.cpp *.c *.cs *.java *.js</string>
          </property>
         </widget>
        </item>
        <item row="1" column="4">
         <widget class="QLabel" name="labPercentNote">
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
        <item row="2" column="4">
         <widget class="QLabel" name="labPercentBlank">
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
        <item row="0" column="0">
         <widget class="QLabel" name="labCount">
          <property name="text">
           <string>文件数</string>
          </property>
         </widget>
        </item>
        <item row="0" column="2">
         <widget class="QLabel" name="labCode">
          <property name="text">
           <string>代码行数</string>
          </property>
         </widget>
        </item>
        <item row="1" column="2">
         <widget class="QLabel" name="labNote">
          <property name="text">
           <string>注释行数</string>
          </property>
         </widget>
        </item>
        <item row="1" column="0">
         <widget class="QLabel" name="labSize">
          <property name="text">
           <string>字节数</string>
          </property>
         </widget>
        </item>
        <item row="2" column="0">
         <widget class="QLabel" name="labRow">
          <property name="text">
           <string>总行数</string>
          </property>
         </widget>
        </item>
       </layout>
      </item>
      <item row="0" column="1">
       <widget class="QPushButton" name="btnOpenFile">
        <property name="text">
         <string>打开文件</string>
        </property>
       </widget>
      </item>
      <item row="1" column="1">
       <widget class="QPushButton" name="btnOpenPath">
        <property name="text">
         <string>打开目录</string>
        </property>
       </widget>
      </item>
      <item row="2" column="1">
       <widget class="QPushButton" name="btnClear">
        <property name="text">
         <string>清空结果</string>
        </property>
       </widget>
      </item>
     </layout>
    </widget>
   </item>
  </layout>
  <action name="actionOpen">
   <property name="icon">
    <iconset>
     <normaloff>:/images/toolbar/ic_files.png</normaloff>:/images/toolbar/ic_files.png</iconset>
   </property>
   <property name="text">
    <string>选择文件</string>
   </property>
   <property name="shortcut">
    <string>Ctrl+O</string>
   </property>
  </action>
  <action name="actionOpenDir">
   <property name="icon">
    <iconset>
     <normaloff>:/images/toolbar/ic_folder.png</normaloff>:/images/toolbar/ic_folder.png</iconset>
   </property>
   <property name="text">
    <string>选择目录</string>
   </property>
   <property name="shortcut">
    <string>Ctrl+Shift+O</string>
   </property>
  </action>
  <action name="actionAbout">
   <property name="icon">
    <iconset>
     <normaloff>:/images/toolbar/ic_about.png</normaloff>:/images/toolbar/ic_about.png</iconset>
   </property>
   <property name="text">
    <string>关于</string>
   </property>
  </action>
  <action name="actionClearModel">
   <property name="icon">
    <iconset>
     <normaloff>:/images/toolbar/ic_clean.png</normaloff>:/images/toolbar/ic_clean.png</iconset>
   </property>
   <property name="text">
    <string>清空列表</string>
   </property>
  </action>
  <action name="actionClearModelLine">
   <property name="icon">
    <iconset>
     <normaloff>:/images/toolbar/ic_delete.png</normaloff>:/images/toolbar/ic_delete.png</iconset>
   </property>
   <property name="text">
    <string>删除选中行</string>
   </property>
  </action>
  <action name="actionChinese">
   <property name="checkable">
    <bool>true</bool>
   </property>
   <property name="checked">
    <bool>true</bool>
   </property>
   <property name="text">
    <string>中文</string>
   </property>
  </action>
  <action name="actionEnglish">
   <property name="checkable">
    <bool>true</bool>
   </property>
   <property name="text">
    <string>English</string>
   </property>
  </action>
  <action name="actionUTF8">
   <property name="checkable">
    <bool>true</bool>
   </property>
   <property name="checked">
    <bool>true</bool>
   </property>
   <property name="text">
    <string>UTF8</string>
   </property>
  </action>
  <action name="actionGB18030">
   <property name="checkable">
    <bool>true</bool>
   </property>
   <property name="text">
    <string>GB18030</string>
   </property>
  </action>
  <action name="actionQuit">
   <property name="text">
    <string>退出</string>
   </property>
   <property name="shortcut">
    <string>Ctrl+Q</string>
   </property>
  </action>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <tabstops>
  <tabstop>btnOpenFile</tabstop>
  <tabstop>btnOpenPath</tabstop>
  <tabstop>btnClear</tabstop>
  <tabstop>tableWidget</tabstop>
  <tabstop>txtCount</tabstop>
  <tabstop>txtSize</tabstop>
  <tabstop>txtRow</tabstop>
  <tabstop>txtCode</tabstop>
  <tabstop>txtNote</tabstop>
  <tabstop>txtBlank</tabstop>
  <tabstop>txtFile</tabstop>
  <tabstop>txtPath</tabstop>
  <tabstop>txtFilter</tabstop>
 </tabstops>
 <resources/>
 <connections/>
</ui>
```
