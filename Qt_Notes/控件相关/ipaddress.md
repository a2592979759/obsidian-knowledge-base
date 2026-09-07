---
tags:
  - Qt
  - 控件相关
---
# IP地址输入框 (`ipaddress`)

> IP地址输入框

## 效果图

![[QWidgetDemo_assert/ipaddress.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/ipaddress`

## 技术要点

**用到的 Qt 类**: QString QWidget QLineEdit QTextCodec QLabel QSizePolicy QSize QEvent QFrame QPushButton QRegularExpressionValidator QRegularExpression QRegExp QRegExpValidator QVBoxLayout QHBoxLayout QObject QKeyEvent QStringList QApplication

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmipaddress.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmipaddress.h"
#include "ui_frmipaddress.h"
#include "qdebug.h"

frmIPAddress::frmIPAddress(QWidget *parent) : QWidget(parent), ui(new Ui::frmIPAddress)
{
    ui->setupUi(this);
    //ui->widgetIP->setBgColor("#ff0000");
    on_btnSetIP_clicked();
}

frmIPAddress::~frmIPAddress()
{
    delete ui;
}

void frmIPAddress::on_btnSetIP_clicked()
{
    ui->widgetIP->setIP("192.168.1.56");
}

void frmIPAddress::on_btnGetIP_clicked()
{
    qDebug() << ui->widgetIP->getIP();
}

void frmIPAddress::on_btnClear_clicked()
{
    ui->widgetIP->clear();
}
```

### `./frmipaddress.h`

```cpp
﻿#ifndef FRMADDRESS_H
#define FRMADDRESS_H

#include <QWidget>

namespace Ui {
class frmIPAddress;
}

class frmIPAddress : public QWidget
{
    Q_OBJECT

public:
    explicit frmIPAddress(QWidget *parent = 0);
    ~frmIPAddress();

private:
    Ui::frmIPAddress *ui;

private slots:
    void on_btnSetIP_clicked();
    void on_btnGetIP_clicked();
    void on_btnClear_clicked();
};

#endif // FRMADDRESS_H
```

### `./ipaddress.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "ipaddress.h"
#include "qlabel.h"
#include "qlineedit.h"
#include "qboxlayout.h"
#include "qregexp.h"
#include "qvalidator.h"
#include "qevent.h"
#include "qdebug.h"

IPAddress::IPAddress(QWidget *parent) : QWidget(parent)
{
    bgColor = "#FFFFFF";
    borderColor = "#A6B5B8";
    borderRadius = 3;

    //用于显示小圆点的标签/居中对齐
    labDot1 = new QLabel;
    labDot1->setAlignment(Qt::AlignCenter);
    labDot1->setText(".");

    labDot2 = new QLabel;
    labDot2->setAlignment(Qt::AlignCenter);
    labDot2->setText(".");

    labDot3 = new QLabel;
    labDot3->setAlignment(Qt::AlignCenter);
    labDot3->setText(".");

    //用于输入IP地址的文本框/居中对齐
    txtIP1 = new QLineEdit;
    txtIP1->setObjectName("txtIP1");
    txtIP1->setAlignment(Qt::AlignCenter);
    txtIP1->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Expanding);
    connect(txtIP1, SIGNAL(textChanged(QString)), this, SLOT(textChanged(QString)));

    txtIP2 = new QLineEdit;
    txtIP2->setObjectName("txtIP2");
    txtIP2->setAlignment(Qt::AlignCenter);
    txtIP2->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Expanding);
    connect(txtIP2, SIGNAL(textChanged(QString)), this, SLOT(textChanged(QString)));

    txtIP3 = new QLineEdit;
    txtIP3->setObjectName("txtIP3");
    txtIP3->setAlignment(Qt::AlignCenter);
    txtIP3->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Expanding);
    connect(txtIP3, SIGNAL(textChanged(QString)), this, SLOT(textChanged(QString)));

    txtIP4 = new QLineEdit;
    txtIP4->setObjectName("txtIP4");
    txtIP4->setAlignment(Qt::AlignCenter);
    txtIP4->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Expanding);
    connect(txtIP4, SIGNAL(textChanged(QString)), this, SLOT(textChanged(QString)));

    //设置IP地址校验过滤
    QString pattern = "(2[0-5]{2}|2[0-4][0-9]|1?[0-9]{1,2})";
    //确切的说 QRegularExpression QRegularExpressionValidator 从5.0 5.1开始就有
#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
    QRegularExpression regExp(pattern);
    QRegularExpressionValidator *validator = new QRegularExpressionValidator(regExp, this);
#else
    QRegExp regExp(pattern);
    QRegExpValidator *validator = new QRegExpValidator(regExp, this);
#endif

    txtIP1->setValidator(validator);
    txtIP2->setValidator(validator);
    txtIP3->setValidator(validator);
    txtIP4->setValidator(validator);

    //绑定事件过滤器/识别键盘按下
    txtIP1->installEventFilter(this);
    txtIP2->installEventFilter(this);
    txtIP3->installEventFilter(this);
    txtIP4->installEventFilter(this);

    frame = new QFrame;
    frame->setObjectName("frameIP");

    QVBoxLayout *verticalLayout = new QVBoxLayout(this);
    verticalLayout->setContentsMargins(0, 0, 0, 0);
    verticalLayout->setSpacing(0);
    verticalLayout->addWidget(frame);

    //将控件按照横向布局排列
    QHBoxLayout *layout = new QHBoxLayout(frame);
    layout->setContentsMargins(0, 0, 0, 0);
    layout->setSpacing(0);
    layout->addWidget(txtIP1);
    layout->addWidget(labDot1);
    layout->addWidget(txtIP2);
    layout->addWidget(labDot2);
    layout->addWidget(txtIP3);
    layout->addWidget(labDot3);
    layout->addWidget(txtIP4);

    this->initStyle();
}

bool IPAddress::eventFilter(QObject *watched, QEvent *event)
{
    if (event->type() == QEvent::KeyPress) {
        QLineEdit *txt = (QLineEdit *)watched;
        if (txt == txtIP1 || txt == txtIP2 || txt == txtIP3 || txt == txtIP4) {
            QKeyEvent *key = (QKeyEvent *)event;

            //如果当前按下了小数点/则移动焦点到下一个输入框
            if (key->text() == ".") {
                this->focusNextChild();
            }

            //如果按下了退格键/并且当前文本框已经没有了内容/则焦点往前移
            if (key->key() == Qt::Key_Backspace) {
                if (txt->text().length() <= 1) {
                    this->focusNextPrevChild(false);
                }
            }
        }
    }

    return QWidget::eventFilter(watched, event);
}

void IPAddress::initStyle()
{
    QStringList qss;
    qss << QString("QFrame#frameIP{border:1px solid %1;border-radius:%2px;}").arg(borderColor).arg(borderRadius);
    qss << QString("QLabel{min-width:15px;background-color:%1;}").arg(bgColor);
    qss << QString("QLineEdit{background-color:%1;border:none;}").arg(bgColor);
    qss << QString("QLineEdit#txtIP1{border-top-left-radius:%1px;border-bottom-left-radius:%1px;}").arg(borderRadius);
    qss << QString("QLineEdit#txtIP4{border-top-right-radius:%1px;border-bottom-right-radius:%1px;}").arg(borderRadius);
    frame->setStyleSheet(qss.join(""));
}

void IPAddress::textChanged(const QString &text)
{
    int len = text.length();
    int value = text.toInt();

    //判断当前是否输入完成一个网段/是的话则自动移动到下一个输入框
    if (len == 3) {
        if (value >= 100 && value <= 255) {
            this->focusNextChild();
        }
    }

    //拼接成完整IP地址
    ip = QString("%1.%2.%3.%4").arg(txtIP1->text()).arg(txtIP2->text()).arg(txtIP3->text()).arg(txtIP4->text());
}

QSize IPAddress::sizeHint() const
{
    return QSize(250, 20);
}

QSize IPAddress::minimumSizeHint() const
{
    return QSize(30, 10);
}

QString IPAddress::getIP() const
{
    return this->ip;
}

void IPAddress::setIP(const QString &ip)
{
    //先检测IP地址是否合法
    QRegExp regExp("((2[0-4]\\d|25[0-5]|[01]?\\d\\d?)\\.){3}(2[0-4]\\d|25[0-5]|[01]?\\d\\d?)");
    if (!regExp.exactMatch(ip)) {
        return;
    }

    if (this->ip != ip) {
        this->ip = ip;

        //将IP地址填入各个网段
        QStringList list = ip.split(".");
        txtIP1->setText(list.at(0));
        txtIP2->setText(list.at(1));
        txtIP3->setText(list.at(2));
        txtIP4->setText(list.at(3));
    }
}

void IPAddress::clear()
{
    txtIP1->clear();
    txtIP2->clear();
    txtIP3->clear();
    txtIP4->clear();
    txtIP1->setFocus();
}

void IPAddress::setBgColor(const QString &bgColor)
{
    if (this->bgColor != bgColor) {
        this->bgColor = bgColor;
        this->initStyle();
    }
}

void IPAddress::setBorderColor(const QString &borderColor)
{
    if (this->borderColor != borderColor) {
        this->borderColor = borderColor;
        this->initStyle();
    }
}

void IPAddress::setBorderRadius(int borderRadius)
{
    if (this->borderRadius != borderRadius) {
        this->borderRadius = borderRadius;
        this->initStyle();
    }
}
```

### `./ipaddress.h`

```cpp
﻿#ifndef IPADDRESS_H
#define IPADDRESS_H

/**
 * IP地址输入框控件 作者:feiyangqingyun(QQ:517216493) 2017-08-11
 * 1. 可设置IP地址，自动填入框。
 * 2. 可清空IP地址。
 * 3. 支持按下小圆点自动切换。
 * 4. 支持退格键自动切换。
 * 5. 支持IP地址过滤。
 * 6. 可设置背景色、边框颜色、边框圆角角度。
 */

#include <QWidget>
class QFrame;
class QLabel;
class QLineEdit;

#ifdef quc
class Q_DECL_EXPORT IPAddress : public QWidget
#else
class IPAddress : public QWidget
#endif

{
    Q_OBJECT

    Q_PROPERTY(QString ip READ getIP WRITE setIP)

public:
    explicit IPAddress(QWidget *parent = 0);

protected:
    bool eventFilter(QObject *watched, QEvent *event);

private:
    QFrame *frame;      //外框
    QLabel *labDot1;    //第一个小圆点
    QLabel *labDot2;    //第二个小圆点
    QLabel *labDot3;    //第三个小圆点

    QLineEdit *txtIP1;  //IP地址网段输入框1
    QLineEdit *txtIP2;  //IP地址网段输入框2
    QLineEdit *txtIP3;  //IP地址网段输入框3
    QLineEdit *txtIP4;  //IP地址网段输入框4

    QString ip;         //IP地址
    QString bgColor;    //背景颜色
    QString borderColor;//边框颜色
    int borderRadius;   //边框圆角角度

private slots:
    void initStyle();
    void textChanged(const QString &text);

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //获取和设置IP地址
    QString getIP() const;
    void setIP(const QString &ip);

    //清空
    void clear();

    //设置背景颜色
    void setBgColor(const QString &bgColor);
    //设置边框颜色
    void setBorderColor(const QString &borderColor);
    //设置边框圆角角度
    void setBorderRadius(int borderRadius);
};

#endif // IPADDRESS_H
```

### `./main.cpp`

```cpp
﻿#include "frmipaddress.h"
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

    frmIPAddress w;
    w.setWindowTitle("IP地址控件 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmipaddress.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmIPAddress</class>
 <widget class="QWidget" name="frmIPAddress">
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
  <widget class="QWidget" name="layoutWidget">
   <property name="geometry">
    <rect>
     <x>10</x>
     <y>10</y>
     <width>281</width>
     <height>71</height>
    </rect>
   </property>
   <layout class="QGridLayout" name="gridLayout">
    <property name="spacing">
     <number>6</number>
    </property>
    <item row="0" column="0" colspan="3">
     <widget class="IPAddress" name="widgetIP" native="true"/>
    </item>
    <item row="1" column="0">
     <widget class="QPushButton" name="btnSetIP">
      <property name="text">
       <string>填入IP</string>
      </property>
     </widget>
    </item>
    <item row="1" column="1">
     <widget class="QPushButton" name="btnGetIP">
      <property name="text">
       <string>获取IP</string>
      </property>
     </widget>
    </item>
    <item row="1" column="2">
     <widget class="QPushButton" name="btnClear">
      <property name="text">
       <string>清空</string>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <customwidgets>
  <customwidget>
   <class>IPAddress</class>
   <extends>QWidget</extends>
   <header>ipaddress.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
