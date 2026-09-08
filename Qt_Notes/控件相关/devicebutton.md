---
tags:
  - Qt
  - 控件相关
  - 高质量
---
# 设备按钮 (`devicebutton`)

> 设备按钮

## 效果图

![[QWidgetDemo_assert/devicebutton.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/devicebutton`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QString QPushButton QWidget QTextCodec QEvent QRectF QSize QFrame QTimer QPainter QPoint QPaintEvent QFont QObject QMouseEvent QList QApplication QImage QMetaObject QHBoxLayout

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./devicebutton.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "devicebutton.h"
#include "qpainter.h"
#include "qevent.h"
#include "qtimer.h"
#include "qdebug.h"

DeviceButton::DeviceButton(QWidget *parent) : QWidget(parent)
{
    canMove = false;
    text = "1";

    colorNormal = "black";
    colorAlarm = "red";

    buttonStyle = ButtonStyle_Police;
    buttonColor = ButtonColor_Green;

    isPressed = false;
    lastPoint = QPoint();

    type = "police";
    imgPath = ":/image/devicebutton/devicebutton";
    imgName = QString("%1_green_%2.png").arg(imgPath).arg(type);

    isDark = false;
    timer = new QTimer(this);
    timer->setInterval(500);
    connect(timer, SIGNAL(timeout()), this, SLOT(checkAlarm()));

    this->installEventFilter(this);
}

DeviceButton::~DeviceButton()
{
    if (timer->isActive()) {
        timer->stop();
    }
}

void DeviceButton::paintEvent(QPaintEvent *)
{
    double width = this->width();
    double height = this->height();

    QPainter painter(this);
    painter.setRenderHint(QPainter::Antialiasing);

    //绘制背景图
    QImage img(imgName);
    if (!img.isNull()) {
        img = img.scaled(width, height, Qt::IgnoreAspectRatio, Qt::SmoothTransformation);
        painter.drawImage(0, 0, img);
    }

    //计算字体
    QFont font;
    font.setPixelSize(height * 0.37);
    font.setBold(true);

    //自动计算文字绘制区域,绘制防区号
    QRectF rect = this->rect();
    if (buttonStyle == ButtonStyle_Police) {
        double y = (30 * height / 60);
        rect = QRectF(0, y, width, height - y);
    } else if (buttonStyle == ButtonStyle_Bubble) {
        double y = (8 * height / 60);
        rect = QRectF(0, 0, width, height - y);
    } else if (buttonStyle == ButtonStyle_Bubble2) {
        double y = (13 * height / 60);
        rect = QRectF(0, 0, width, height - y);
        font.setPixelSize(width * 0.33);
    } else if (buttonStyle == ButtonStyle_Msg) {
        double y = (17 * height / 60);
        rect = QRectF(0, 0, width, height - y);
    } else if (buttonStyle == ButtonStyle_Msg2) {
        double y = (17 * height / 60);
        rect = QRectF(0, 0, width, height - y);
    }

    //绘制文字标识
    painter.setFont(font);
    painter.setPen(Qt::white);
    painter.drawText(rect, Qt::AlignCenter, text);
}

bool DeviceButton::eventFilter(QObject *watched, QEvent *event)
{
    //识别鼠标 按下+移动+松开+双击 等事件
    int type = event->type();
    QMouseEvent *mouseEvent = static_cast<QMouseEvent *>(event);
    if (type == QEvent::MouseButtonPress) {
        //限定鼠标左键
        if (mouseEvent->button() == Qt::LeftButton) {
            lastPoint = mouseEvent->pos();
            isPressed = true;
            Q_EMIT clicked();
            return true;
        }
    } else if (type == QEvent::MouseMove) {
        //允许拖动并且鼠标按下准备拖动
        if (canMove && isPressed) {
            int dx = mouseEvent->pos().x() - lastPoint.x();
            int dy = mouseEvent->pos().y() - lastPoint.y();
            this->move(this->x() + dx, this->y() + dy);            
            return true;
        }
    } else if (type == QEvent::MouseButtonRelease) {
        isPressed = false;
        //增加刷新父窗体/防止残影
        if (this->parent()) {
            QMetaObject::invokeMethod(this->parent(), "update");
        }
    } else if (type == QEvent::MouseButtonDblClick) {
        Q_EMIT doubleClicked();
    }

    return QWidget::eventFilter(watched, event);
}

QSize DeviceButton::sizeHint() const
{
    return QSize(50, 50);
}

QSize DeviceButton::minimumSizeHint() const
{
    return QSize(10, 10);
}

void DeviceButton::checkAlarm()
{
    if (isDark) {
        imgName = QString("%1_%2_%3.png").arg(imgPath).arg(colorNormal).arg(type);
    } else {
        imgName = QString("%1_%2_%3.png").arg(imgPath).arg(colorAlarm).arg(type);
    }

    isDark = !isDark;
    this->update();
}

bool DeviceButton::getCanMove() const
{
    return this->canMove;
}

void DeviceButton::setCanMove(bool canMove)
{
    this->canMove = canMove;
}

QString DeviceButton::getText() const
{
    return this->text;
}

void DeviceButton::setText(const QString &text)
{
    if (this->text != text) {
        this->text = text;
        this->update();
    }
}

QString DeviceButton::getColorNormal() const
{
    return this->colorNormal;
}

void DeviceButton::setColorNormal(const QString &colorNormal)
{
    if (this->colorNormal != colorNormal) {
        this->colorNormal = colorNormal;
        this->update();
    }
}

QString DeviceButton::getColorAlarm() const
{
    return this->colorAlarm;
}

void DeviceButton::setColorAlarm(const QString &colorAlarm)
{
    if (this->colorAlarm != colorAlarm) {
        this->colorAlarm = colorAlarm;
        this->update();
    }
}

DeviceButton::ButtonStyle DeviceButton::getButtonStyle() const
{
    return this->buttonStyle;
}

void DeviceButton::setButtonStyle(const DeviceButton::ButtonStyle &buttonStyle)
{
    this->buttonStyle = buttonStyle;
    if (buttonStyle == ButtonStyle_Circle) {
        type = "circle";
    } else if (buttonStyle == ButtonStyle_Police) {
        type = "police";
    } else if (buttonStyle == ButtonStyle_Bubble) {
        type = "bubble";
    } else if (buttonStyle == ButtonStyle_Bubble2) {
        type = "bubble2";
    } else if (buttonStyle == ButtonStyle_Msg) {
        type = "msg";
    } else if (buttonStyle == ButtonStyle_Msg2) {
        type = "msg2";
    } else {
        type = "circle";
    }

    setButtonColor(buttonColor);
}

DeviceButton::ButtonColor DeviceButton::getButtonColor() const
{
    return this->buttonColor;
}

void DeviceButton::setButtonColor(const DeviceButton::ButtonColor &buttonColor)
{
    //先停止定时器
    this->buttonColor = buttonColor;
    isDark = false;
    if (timer->isActive()) {
        timer->stop();
    }

    QString color;
    if (buttonColor == ButtonColor_Green) {
        color = "green";
    } else if (buttonColor == ButtonColor_Blue) {
        color = "blue";
    } else if (buttonColor == ButtonColor_Gray) {
        color = "gray";
    } else if (buttonColor == ButtonColor_Black) {
        color = "black";
    } else if (buttonColor == ButtonColor_Purple) {
        color = "purple";
    } else if (buttonColor == ButtonColor_Yellow) {
        color = "yellow";
    } else if (buttonColor == ButtonColor_Red) {
        color = "red";
    } else {
        color = "green";
    }

    //如果和报警颜色一致则主动启动定时器切换报警颜色
    imgName = QString("%1_%2_%3.png").arg(imgPath).arg(color).arg(type);
    if (color == colorAlarm) {
        checkAlarm();
        if (!timer->isActive()) {
            timer->start();
        }
    }

    this->update();
}
```

### `./devicebutton.h`

```cpp
﻿#ifndef DEVICEBUTTON_H
#define DEVICEBUTTON_H

/**
 * 设备按钮控件 作者:feiyangqingyun(QQ:517216493) 2018-07-02
 * 1. 可设置按钮样式 圆形、警察、气泡、气泡2、消息、消息2。
 * 2. 可设置按钮颜色 布防、撤防、报警、旁路、故障。
 * 3. 可设置报警切换及对应报警切换的颜色。
 * 4. 可设置显示的防区号。
 * 5. 可设置是否可鼠标拖动。
 * 6. 发出单击和双击信号。
 */

#include <QWidget>

#ifdef quc
class Q_DECL_EXPORT DeviceButton : public QWidget
#else
class DeviceButton : public QWidget
#endif

{
    Q_OBJECT
    Q_ENUMS(ButtonStyle)
    Q_ENUMS(ButtonColor)

    Q_PROPERTY(bool canMove READ getCanMove WRITE setCanMove)
    Q_PROPERTY(QString text READ getText WRITE setText)

    Q_PROPERTY(QString colorNormal READ getColorNormal WRITE setColorNormal)
    Q_PROPERTY(QString colorAlarm READ getColorAlarm WRITE setColorAlarm)

    Q_PROPERTY(ButtonStyle buttonStyle READ getButtonStyle WRITE setButtonStyle)
    Q_PROPERTY(ButtonColor buttonColor READ getButtonColor WRITE setButtonColor)

public:
    //设备按钮样式
    enum ButtonStyle {
        ButtonStyle_Circle = 0,     //圆形
        ButtonStyle_Police = 1,     //警察
        ButtonStyle_Bubble = 2,     //气泡
        ButtonStyle_Bubble2 = 3,    //气泡2
        ButtonStyle_Msg = 4,        //消息
        ButtonStyle_Msg2 = 5        //消息2
    };

    //设备按钮颜色
    enum ButtonColor {
        ButtonColor_Green = 0,      //绿色(激活状态)
        ButtonColor_Blue = 1,       //蓝色(在线状态)
        ButtonColor_Red = 2,        //红色(报警状态)
        ButtonColor_Gray = 3,       //灰色(离线状态)
        ButtonColor_Black = 4,      //黑色(故障状态)
        ButtonColor_Purple = 5,     //紫色(其他状态)
        ButtonColor_Yellow = 6      //黄色(其他状态)
    };

    explicit DeviceButton(QWidget *parent = 0);
    ~DeviceButton();

protected:
    void paintEvent(QPaintEvent *);
    bool eventFilter(QObject *watched, QEvent *event);

private:
    bool canMove;           //是否可移动
    QString text;           //显示文字

    QString colorNormal;    //正常颜色
    QString colorAlarm;     //报警颜色

    ButtonStyle buttonStyle;//按钮样式
    ButtonColor buttonColor;//按钮颜色

    bool isPressed;         //鼠标是否按下
    QPoint lastPoint;       //鼠标按下最后坐标

    QString type;           //图片末尾类型
    QString imgPath;        //背景图片路径
    QString imgName;        //背景图片名称

    bool isDark;            //是否加深报警
    QTimer *timer;          //报警闪烁定时器

private slots:
    void checkAlarm();      //切换报警状态

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //获取和设置可移动
    bool getCanMove() const;
    void setCanMove(bool canMove);

    //获取和设置显示文字
    QString getText() const;
    void setText(const QString &text);

    //获取和设置正常颜色
    QString getColorNormal() const;
    void setColorNormal(const QString &colorNormal);

    //获取和设置报警颜色
    QString getColorAlarm() const;
    void setColorAlarm(const QString &colorAlarm);

    //获取和设置样式
    ButtonStyle getButtonStyle() const;
    void setButtonStyle(const ButtonStyle &buttonStyle);

    //获取和设置颜色
    ButtonColor getButtonColor() const;
    void setButtonColor(const ButtonColor &buttonColor);

Q_SIGNALS:
    //鼠标单击双击事件
    void clicked();
    void doubleClicked();
};

#endif //DEVICEBUTTON_H
```

### `./frmdevicebutton.cpp`

```cpp
﻿#include "frmdevicebutton.h"
#include "ui_frmdevicebutton.h"
#include "devicebutton.h"
#include "qdebug.h"

frmDeviceButton::frmDeviceButton(QWidget *parent) : QWidget(parent), ui(new Ui::frmDeviceButton)
{
    ui->setupUi(this);
    this->initForm();
}

frmDeviceButton::~frmDeviceButton()
{
    delete ui;
}

void frmDeviceButton::initForm()
{
    //设置背景地图
    ui->labMap->setStyleSheet("border-image:url(:/image/bg_call.jpg);");

    btn1 = new DeviceButton(ui->labMap);
    btn1->setText("#1");
    btn1->setGeometry(5, 5, 35, 35);

    btn2 = new DeviceButton(ui->labMap);
    btn2->setText("#2");
    btn2->setGeometry(45, 5, 35, 35);

    btn3 = new DeviceButton(ui->labMap);
    btn3->setText("#3");
    btn3->setGeometry(85, 5, 35, 35);

    btnStyle << ui->btnCircle << ui->btnPolice << ui->btnBubble << ui->btnBubble2 << ui->btnMsg << ui->btnMsg2;
    foreach (QPushButton *btn, btnStyle) {
        connect(btn, SIGNAL(clicked(bool)), this, SLOT(changeStyle()));
    }

    btnColor << ui->btnGreen << ui->btnBlue << ui->btnRed << ui->btnGray << ui->btnBlack << ui->btnPurple << ui->btnYellow;
    foreach (QPushButton *btn, btnColor) {
        connect(btn, SIGNAL(clicked(bool)), this, SLOT(changeColor()));
    }
}

void frmDeviceButton::changeStyle()
{
    QPushButton *btn = (QPushButton *)sender();
    int index = btnStyle.indexOf(btn);
    DeviceButton::ButtonStyle style = (DeviceButton::ButtonStyle)index;
    btn1->setButtonStyle(style);
    btn2->setButtonStyle(style);
    btn3->setButtonStyle(style);
}

void frmDeviceButton::changeColor()
{
    QPushButton *btn = (QPushButton *)sender();
    int index = btnColor.indexOf(btn);
    DeviceButton::ButtonColor style = (DeviceButton::ButtonColor)index;
    btn1->setButtonColor(style);
    btn2->setButtonColor(style);
    btn3->setButtonColor(style);
}

void frmDeviceButton::on_ckCanMove_stateChanged(int arg1)
{
    bool canMove = (arg1 != 0);
    btn1->setCanMove(canMove);
    btn2->setCanMove(canMove);
    btn3->setCanMove(canMove);
}
```

### `./frmdevicebutton.h`

```cpp
﻿#ifndef FRMDEVICEBUTTON_H
#define FRMDEVICEBUTTON_H

#include <QWidget>

class DeviceButton;
class QPushButton;

namespace Ui {
class frmDeviceButton;
}

class frmDeviceButton : public QWidget
{
    Q_OBJECT

public:
    explicit frmDeviceButton(QWidget *parent = 0);
    ~frmDeviceButton();

private slots:
    void initForm();
    void changeStyle();
    void changeColor();
    void on_ckCanMove_stateChanged(int arg1);

private:
    Ui::frmDeviceButton *ui;
    DeviceButton *btn1;
    DeviceButton *btn2;
    DeviceButton *btn3;
    QList<QPushButton *> btnStyle;
    QList<QPushButton *> btnColor;
};

#endif // FRMDEVICEBUTTON_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmdevicebutton.h"
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

    frmDeviceButton w;
    w.setWindowTitle("设备按钮控件 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmdevicebutton.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmDeviceButton</class>
 <widget class="QWidget" name="frmDeviceButton">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>900</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QHBoxLayout" name="horizontalLayout">
   <item>
    <widget class="QLabel" name="labMap">
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
   <item>
    <widget class="QFrame" name="frame">
     <property name="maximumSize">
      <size>
       <width>100</width>
       <height>16777215</height>
      </size>
     </property>
     <property name="frameShape">
      <enum>QFrame::Box</enum>
     </property>
     <property name="frameShadow">
      <enum>QFrame::Sunken</enum>
     </property>
     <layout class="QVBoxLayout" name="verticalLayout">
      <item>
       <widget class="QPushButton" name="btnCircle">
        <property name="text">
         <string>圆形</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnPolice">
        <property name="text">
         <string>警察</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnBubble">
        <property name="text">
         <string>气泡</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnBubble2">
        <property name="text">
         <string>气泡2</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnMsg">
        <property name="text">
         <string>消息</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnMsg2">
        <property name="text">
         <string>消息2</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="Line" name="line">
        <property name="orientation">
         <enum>Qt::Horizontal</enum>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnGreen">
        <property name="text">
         <string>绿色</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnBlue">
        <property name="text">
         <string>蓝色</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnRed">
        <property name="text">
         <string>红色</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnGray">
        <property name="text">
         <string>灰色</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnBlack">
        <property name="text">
         <string>黑色</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnPurple">
        <property name="text">
         <string>紫色</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnYellow">
        <property name="text">
         <string>黄色</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QCheckBox" name="ckCanMove">
        <property name="text">
         <string>可移动</string>
        </property>
       </widget>
      </item>
      <item>
       <spacer name="verticalSpacer">
        <property name="orientation">
         <enum>Qt::Vertical</enum>
        </property>
        <property name="sizeHint" stdset="0">
         <size>
          <width>20</width>
          <height>40</height>
         </size>
        </property>
       </spacer>
      </item>
     </layout>
    </widget>
   </item>
  </layout>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <resources/>
 <connections/>
</ui>
```
