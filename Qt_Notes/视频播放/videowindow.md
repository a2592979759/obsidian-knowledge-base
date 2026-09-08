---
tags:
  - Qt
  - 视频播放
---
# 视频监控控件 (`videowindow`)

> 视频监控控件

## 效果图

![[QWidgetDemo_assert/videowindow.jpg]]

## 分类

- 类型: 视频播放 `video`
- 源码目录: `video/videowindow`

## 技术要点

**用到的 Qt 类**: QString QColor QImage QWidget QPainter QTextCodec QDateTime QList QPoint QApplication QTimer QSize QRect QPushButton QFont QStyle QEvent QPixmap QFrame QHBoxLayout

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmvideowindow.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmvideowindow.h"
#include "ui_frmvideowindow.h"

frmVideoWindow::frmVideoWindow(QWidget *parent) : QWidget(parent), ui(new Ui::frmVideoWindow)
{
    ui->setupUi(this);
    this->initForm();
}

frmVideoWindow::~frmVideoWindow()
{
    delete ui;
}

void frmVideoWindow::initForm()
{
    ui->videoWindow1->setFlowEnable(true);
    ui->videoWindow2->setFlowEnable(true);
    ui->videoWindow3->setFlowEnable(true);
    ui->videoWindow4->setFlowEnable(true);

    connect(ui->videoWindow1, SIGNAL(btnClicked(QString)), this, SLOT(btnClicked(QString)));
    connect(ui->videoWindow2, SIGNAL(btnClicked(QString)), this, SLOT(btnClicked(QString)));
    connect(ui->videoWindow3, SIGNAL(btnClicked(QString)), this, SLOT(btnClicked(QString)));
    connect(ui->videoWindow4, SIGNAL(btnClicked(QString)), this, SLOT(btnClicked(QString)));
}

void frmVideoWindow::btnClicked(const QString &objName)
{
    VideoWindow *videoWindow = (VideoWindow *)sender();
    QString str = QString("当前单击了控件 %1 的按钮 %2").arg(videoWindow->objectName()).arg(objName);
    ui->label->setText(str);
}
```

### `./frmvideowindow.h`

```cpp
﻿#ifndef FRMVIDEOWINDOW_H
#define FRMVIDEOWINDOW_H

#include <QWidget>

namespace Ui {
class frmVideoWindow;
}

class frmVideoWindow : public QWidget
{
    Q_OBJECT

public:
    explicit frmVideoWindow(QWidget *parent = 0);
    ~frmVideoWindow();

private:
    Ui::frmVideoWindow *ui;

private slots:
    void initForm();
    void btnClicked(const QString &objName);
};

#endif // FRMVIDEOWINDOW_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmvideowindow.h"
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

    frmVideoWindow w;
    w.setWindowTitle("视频监控控件 (QQ: 517216493 WX: feiyangqingyun)");
    w.resize(800, 600);
    w.show();

    return a.exec();
}
```

### `./videowindow.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "videowindow.h"
#include "qfontdatabase.h"
#include "qpushbutton.h"
#include "qtreewidget.h"
#include "qlayout.h"
#include "qtimer.h"
#include "qdir.h"
#include "qpainter.h"
#include "qevent.h"
#include "qmimedata.h"
#include "qurl.h"
#include "qdebug.h"

VideoWindow::VideoWindow(QWidget *parent) : QWidget(parent)
{
    //设置强焦点
    setFocusPolicy(Qt::StrongFocus);
    //设置支持拖放
    setAcceptDrops(true);

    //定时器校验视频
    timerCheck = new QTimer(this);
    timerCheck->setInterval(10 * 1000);
    connect(timerCheck, SIGNAL(timeout()), this, SLOT(checkVideo()));

    image = QImage();
    copyImage = false;
    checkLive = true;
    drawImage = true;
    fillImage = true;

    flowEnable = false;
    flowBgColor = "#000000";
    flowPressColor = "#5EC7D9";

    timeout = 20;
    borderWidth = 5;
    borderColor = "#000000";
    focusColor = "#22A3A9";
    bgColor = Qt::transparent;
    bgText = "实时视频";
    bgImage = QImage();

    osd1Visible = false;
    osd1FontSize = 12;
    osd1Text = "时间";
    osd1Color = "#FF0000";
    osd1Image = QImage();
    osd1Format = OSDFormat_DateTime;
    osd1Position = OSDPosition_Right_Top;

    osd2Visible = false;
    osd2FontSize = 12;
    osd2Text = "通道名称";
    osd2Color = "#FF0000";
    osd2Image = QImage();
    osd2Format = OSDFormat_Text;
    osd2Position = OSDPosition_Left_Bottom;

    //初始化解码线程
    this->initThread();
    //初始化悬浮条
    this->initFlowPanel();
    //初始化悬浮条样式
    this->initFlowStyle();
}

void VideoWindow::initThread()
{

}

void VideoWindow::initFlowPanel()
{
    //顶部工具栏,默认隐藏,鼠标移入显示移除隐藏
    flowPanel = new QWidget(this);
    flowPanel->setObjectName("flowPanel");
    flowPanel->setVisible(false);

    //用布局顶住,左侧弹簧
    QHBoxLayout *layout = new QHBoxLayout;
    layout->setSpacing(2);
    layout->setContentsMargins(0, 0, 0, 0);
    layout->addStretch();
    flowPanel->setLayout(layout);

    //按钮集合名称,如果需要新增按钮则在这里增加即可
    QList<QString> btns;
    btns << "btnFlowVideo" << "btnFlowSnap" << "btnFlowSound" << "btnFlowAlarm" << "btnFlowClose";

    //有多种办法来设置图片,qt内置的图标+自定义的图标+图形字体
    //既可以设置图标形式,也可以直接图形字体设置文本
#if 0
    QList<QIcon> icons;
    icons << QApplication::style()->standardIcon(QStyle::SP_ComputerIcon);
    icons << QApplication::style()->standardIcon(QStyle::SP_FileIcon);
    icons << QApplication::style()->standardIcon(QStyle::SP_DirIcon);
    icons << QApplication::style()->standardIcon(QStyle::SP_DialogOkButton);
    icons << QApplication::style()->standardIcon(QStyle::SP_DialogCancelButton);
#else
    QList<int> icons;
    icons << 0xe68d << 0xe672 << 0xe674 << 0xea36 << 0xe74c;

    //判断图形字体是否存在,不存在则加入
    QFont iconFont;
    QFontDatabase fontDb;
    if (!fontDb.families().contains("iconfont")) {
        int fontId = fontDb.addApplicationFont(":/font/iconfont.ttf");
        QStringList fontName = fontDb.applicationFontFamilies(fontId);
        if (fontName.count() == 0) {
            qDebug() << "load iconfont.ttf error";
        }
    }

    if (fontDb.families().contains("iconfont")) {
        iconFont = QFont("iconfont");
        iconFont.setPixelSize(17);
#if (QT_VERSION >= QT_VERSION_CHECK(4,8,0))
        iconFont.setHintingPreference(QFont::PreferNoHinting);
#endif
    }
#endif

    //循环添加顶部按钮
    for (int i = 0; i < btns.count(); ++i) {
        QPushButton *btn = new QPushButton;
        //绑定按钮单击事件,用来发出信号通知
        connect(btn, SIGNAL(clicked(bool)), this, SLOT(btnClicked()));
        //设置标识,用来区别按钮
        btn->setObjectName(btns.at(i));
        //设置固定宽度
        btn->setFixedWidth(20);
        //设置拉伸策略使得填充
        btn->setSizePolicy(QSizePolicy::Preferred, QSizePolicy::Expanding);
        //设置焦点策略为无焦点,避免单击后焦点跑到按钮上
        btn->setFocusPolicy(Qt::NoFocus);

#if 0
        //设置图标大小和图标
        btn->setIconSize(QSize(16, 16));
        btn->setIcon(icons.at(i));
#else
        btn->setFont(iconFont);
        btn->setText((QChar)icons.at(i));
#endif

        //将按钮加到布局中
        layout->addWidget(btn);
    }
}

void VideoWindow::initFlowStyle()
{
    //设置样式以便区分,可以自行更改样式,也可以不用样式
    QStringList qss;
    QString rgba = QString("rgba(%1,%2,%3,150)").arg(flowBgColor.red()).arg(flowBgColor.green()).arg(flowBgColor.blue());
    qss.append(QString("#flowPanel{background:%1;border:none;}").arg(rgba));
    qss.append(QString("QPushButton{border:none;padding:0px;background:rgba(0,0,0,0);}"));
    qss.append(QString("QPushButton:pressed{color:%1;}").arg(flowPressColor.name()));
    flowPanel->setStyleSheet(qss.join(""));
}

VideoWindow::~VideoWindow()
{
    if (timerCheck->isActive()) {
        timerCheck->stop();
    }

    close();
}

void VideoWindow::resizeEvent(QResizeEvent *)
{
    //重新设置顶部工具栏的位置和宽高,可以自行设置顶部显示或者底部显示
    int height = 20;
    flowPanel->setGeometry(borderWidth, borderWidth, this->width() - (borderWidth * 2), height);
    //flowPanel->setGeometry(borderWidth, this->height() - height - borderWidth, this->width() - (borderWidth * 2), height);
}

void VideoWindow::enterEvent(EnterEvent *)
{
    //这里还可以增加一个判断,是否获取了焦点的才需要显示
    //if (this->hasFocus()) {}
    if (flowEnable) {
        flowPanel->setVisible(true);
    }
}

void VideoWindow::leaveEvent(QEvent *)
{
    if (flowEnable) {
        flowPanel->setVisible(false);
    }
}

void VideoWindow::dropEvent(QDropEvent *event)
{
    //拖放完毕鼠标松开的时候执行
    //判断拖放进来的类型,取出文件,进行播放
    QString url;
    if (event->mimeData()->hasUrls()) {
        url = event->mimeData()->urls().first().toLocalFile();
    } else if (event->mimeData()->hasFormat("application/x-qabstractitemmodeldatalist")) {
        QTreeWidget *treeWidget = (QTreeWidget *)event->source();
        if (treeWidget) {
            url = treeWidget->currentItem()->data(0, Qt::UserRole).toString();
        }
    }

    if (!url.isEmpty()) {
        Q_EMIT fileDrag(url);
        this->restart(url);
    }
}

void VideoWindow::dragEnterEvent(QDragEnterEvent *event)
{
    //拖曳进来的时候先判断下类型,非法类型则不处理
    if (event->mimeData()->hasFormat("application/x-qabstractitemmodeldatalist")) {
        event->setDropAction(Qt::CopyAction);
        event->accept();
    } else if (event->mimeData()->hasFormat("text/uri-list")) {
        event->setDropAction(Qt::LinkAction);
        event->accept();
    } else {
        event->ignore();
    }
}

void VideoWindow::paintEvent(QPaintEvent *)
{
    //如果不需要绘制
    if (!drawImage) {
        return;
    }

    //qDebug() << TIMEMS << "paintEvent" << objectName();
    QPainter painter(this);
    painter.setRenderHints(QPainter::Antialiasing);

    //绘制边框
    drawBorder(&painter);
    if (!image.isNull()) {
        //绘制背景图片
        drawImg(&painter, image);
        //绘制标签
        drawOSD(&painter, osd1Visible, osd1FontSize, osd1Text, osd1Color, osd1Image, osd1Format, osd1Position);
        drawOSD(&painter, osd2Visible, osd2FontSize, osd2Text, osd2Color, osd2Image, osd2Format, osd2Position);
    } else {
        //绘制背景
        drawBg(&painter);
    }
}

void VideoWindow::drawBorder(QPainter *painter)
{
    painter->save();

    QPen pen;
    pen.setWidth(borderWidth);
    pen.setColor(hasFocus() ? focusColor : borderColor);
    //边框宽度=0则不绘制边框
    painter->setPen(borderWidth == 0 ? Qt::NoPen : pen);
    //顺带把背景颜色这里也一并处理
    if (bgColor != Qt::transparent) {
        painter->setBrush(bgColor);
    }
    painter->drawRect(rect());

    painter->restore();
}

void VideoWindow::drawBg(QPainter *painter)
{
    painter->save();

    //背景图片为空则绘制文字,否则绘制背景图片
    if (bgImage.isNull()) {
        painter->setFont(this->font());
        painter->setPen(palette().windowText().color());
        painter->drawText(rect(), Qt::AlignCenter, bgText);
    } else {
        //居中绘制
        int x = rect().center().x() - bgImage.width() / 2;
        int y = rect().center().y() - bgImage.height() / 2;
        QPoint point(x, y);
        painter->drawImage(point, bgImage);
    }

    painter->restore();
}

void VideoWindow::drawImg(QPainter *painter, QImage img)
{
    painter->save();

    int offset = borderWidth * 1 + 0;
    if (fillImage) {
        QRect rect(offset / 2, offset / 2, width() - offset, height() - offset);
        painter->drawImage(rect, img);
    } else {
        //按照比例自动居中绘制
        img = img.scaled(width() - offset, height() - offset, Qt::KeepAspectRatio);
        int x = rect().center().x() - img.width() / 2;
        int y = rect().center().y() - img.height() / 2;
        QPoint point(x, y);
        painter->drawImage(point, img);
    }

    painter->restore();
}

void VideoWindow::drawOSD(QPainter *painter,
                          bool osdVisible,
                          int osdFontSize,
                          const QString &osdText,
                          const QColor &osdColor,
                          const QImage &osdImage,
                          const VideoWindow::OSDFormat &osdFormat,
                          const VideoWindow::OSDPosition &osdPosition)
{
    if (!osdVisible) {
        return;
    }

    painter->save();

    //标签位置尽量偏移多一点避免遮挡
    QRect osdRect(rect().x() + (borderWidth * 2), rect().y() + (borderWidth * 2), width() - (borderWidth * 5), height() - (borderWidth * 5));
    int flag = Qt::AlignLeft | Qt::AlignTop;
    QPoint point = QPoint(osdRect.x(), osdRect.y());

    if (osdPosition == OSDPosition_Left_Top) {
        flag = Qt::AlignLeft | Qt::AlignTop;
        point = QPoint(osdRect.x(), osdRect.y());
    } else if (osdPosition == OSDPosition_Left_Bottom) {
        flag = Qt::AlignLeft | Qt::AlignBottom;
        point = QPoint(osdRect.x(), osdRect.height() - osdImage.height());
    } else if (osdPosition == OSDPosition_Right_Top) {
        flag = Qt::AlignRight | Qt::AlignTop;
        point = QPoint(osdRect.width() - osdImage.width(), osdRect.y());
    } else if (osdPosition == OSDPosition_Right_Bottom) {
        flag = Qt::AlignRight | Qt::AlignBottom;
        point = QPoint(osdRect.width() - osdImage.width(), osdRect.height() - osdImage.height());
    }

    if (osdFormat == OSDFormat_Image) {
        painter->drawImage(point, osdImage);
    } else {
        QDateTime now = QDateTime::currentDateTime();
        QString text = osdText;
        if (osdFormat == OSDFormat_Date) {
            text = now.toString("yyyy-MM-dd");
        } else if (osdFormat == OSDFormat_Time) {
            text = now.toString("HH:mm:ss");
        } else if (osdFormat == OSDFormat_DateTime) {
            text = now.toString("yyyy-MM-dd HH:mm:ss");
        }

        //设置颜色及字号
        QFont font;
        font.setPixelSize(osdFontSize);
        painter->setPen(osdColor);
        painter->setFont(font);

        painter->drawText(osdRect, flag, text);
    }

    painter->restore();
}

QImage VideoWindow::getImage() const
{
    return this->image;
}

QPixmap VideoWindow::getPixmap() const
{
    return QPixmap();
}

QString VideoWindow::getUrl() const
{
    return this->property("url").toString();
}

QDateTime VideoWindow::getLastTime() const
{
    return QDateTime::currentDateTime();
}

bool VideoWindow::getCallback() const
{
    return false;
}

bool VideoWindow::getIsPlaying() const
{
    return false;
}

bool VideoWindow::getIsRtsp() const
{
    return false;
}

bool VideoWindow::getIsUsbCamera() const
{
    return false;
}

bool VideoWindow::getCopyImage() const
{
    return this->copyImage;
}

bool VideoWindow::getCheckLive() const
{
    return this->checkLive;
}

bool VideoWindow::getDrawImage() const
{
    return this->drawImage;
}

bool VideoWindow::getFillImage() const
{
    return this->fillImage;
}

bool VideoWindow::getFlowEnable() const
{
    return this->flowEnable;
}

QColor VideoWindow::getFlowBgColor() const
{
    return this->flowBgColor;
}

QColor VideoWindow::getFlowPressColor() const
{
    return this->flowPressColor;
}

int VideoWindow::getTimeout() const
{
    return this->timeout;
}

int VideoWindow::getBorderWidth() const
{
    return this->borderWidth;
}

QColor VideoWindow::getBorderColor() const
{
    return this->borderColor;
}

QColor VideoWindow::getFocusColor() const
{
    return this->focusColor;
}

QColor VideoWindow::getBgColor() const
{
    return this->bgColor;
}

QString VideoWindow::getBgText() const
{
    return this->bgText;
}

QImage VideoWindow::getBgImage() const
{
    return this->bgImage;
}

bool VideoWindow::getOSD1Visible() const
{
    return this->osd1Visible;
}

int VideoWindow::getOSD1FontSize() const
{
    return this->osd1FontSize;
}

QString VideoWindow::getOSD1Text() const
{
    return this->osd1Text;
}

QColor VideoWindow::getOSD1Color() const
{
    return this->osd1Color;
}

QImage VideoWindow::getOSD1Image() const
{
    return this->osd1Image;
}

VideoWindow::OSDFormat VideoWindow::getOSD1Format() const
{
    return this->osd1Format;
}

VideoWindow::OSDPosition VideoWindow::getOSD1Position() const
{
    return this->osd1Position;
}

bool VideoWindow::getOSD2Visible() const
{
    return this->osd2Visible;
}

int VideoWindow::getOSD2FontSize() const
{
    return this->osd2FontSize;
}

QString VideoWindow::getOSD2Text() const
{
    return this->osd2Text;
}

QColor VideoWindow::getOSD2Color() const
{
    return this->osd2Color;
}

QImage VideoWindow::getOSD2Image() const
{
    return this->osd2Image;
}

VideoWindow::OSDFormat VideoWindow::getOSD2Format() const
{
    return this->osd2Format;
}

VideoWindow::OSDPosition VideoWindow::getOSD2Position() const
{
    return this->osd2Position;
}

int VideoWindow::getFaceBorder() const
{
    return this->faceBorder;
}

QColor VideoWindow::getFaceColor() const
{
    return this->faceColor;
}

QList<QRect> VideoWindow::getFaceRects() const
{
    return this->faceRects;
}

QSize VideoWindow::sizeHint() const
{
    return QSize(400, 300);
}

QSize VideoWindow::minimumSizeHint() const
{
    return QSize(40, 30);
}

void VideoWindow::updateImage(const QImage &image)
{
    //拷贝图片有个好处,当处理器比较差的时候,图片不会产生断层,缺点是占用时间
    //默认QImage类型是浅拷贝,可能正在绘制的时候,那边已经更改了图片的上部分数据
    this->image = copyImage ? image.copy() : image;
    this->update();
}

void VideoWindow::checkVideo()
{
    QDateTime now = QDateTime::currentDateTime();
    QDateTime lastTime = now;
    int sec = lastTime.secsTo(now);
    if (sec >= timeout) {
        restart(this->getUrl());
    }
}

void VideoWindow::btnClicked()
{
    QPushButton *btn = (QPushButton *)sender();
    Q_EMIT btnClicked(btn->objectName());
}

uint VideoWindow::getLength()
{
    return 0;
}

uint VideoWindow::getPosition()
{
    return 0;
}

void VideoWindow::setPosition(int position)
{

}

bool VideoWindow::getMuted()
{
    return false;
}

void VideoWindow::setMuted(bool muted)
{

}

int VideoWindow::getVolume()
{
    return 0;
}

void VideoWindow::setVolume(int volume)
{

}

void VideoWindow::setInterval(int interval)
{

}

void VideoWindow::setSleepTime(int sleepTime)
{

}

void VideoWindow::setCheckTime(int checkTime)
{

}

void VideoWindow::setCheckConn(bool checkConn)
{

}

void VideoWindow::setUrl(const QString &url)
{
    this->setProperty("url", url);
}

void VideoWindow::setCallback(bool callback)
{

}

void VideoWindow::setHardware(const QString &hardware)
{

}

void VideoWindow::setTransport(const QString &transport)
{

}

void VideoWindow::setSaveFile(bool saveFile)
{

}

void VideoWindow::setSaveInterval(int saveInterval)
{

}

void VideoWindow::setFileFlag(const QString &fileFlag)
{

}

void VideoWindow::setSavePath(const QString &savePath)
{
    //如果目录不存在则新建
    QDir dir(savePath);
    if (!dir.exists()) {
        dir.mkpath(savePath);
    }
}

void VideoWindow::setFileName(const QString &fileName)
{

}

void VideoWindow::setCopyImage(bool copyImage)
{
    this->copyImage = copyImage;
}

void VideoWindow::setCheckLive(bool checkLive)
{
    this->checkLive = checkLive;
}

void VideoWindow::setDrawImage(bool drawImage)
{
    this->drawImage = drawImage;
}

void VideoWindow::setFillImage(bool fillImage)
{
    this->fillImage = fillImage;
}

void VideoWindow::setFlowEnable(bool flowEnable)
{
    this->flowEnable = flowEnable;
}

void VideoWindow::setFlowBgColor(const QColor &flowBgColor)
{
    if (this->flowBgColor != flowBgColor) {
        this->flowBgColor = flowBgColor;
        this->initFlowStyle();
    }
}

void VideoWindow::setFlowPressColor(const QColor &flowPressColor)
{
    if (this->flowPressColor != flowPressColor) {
        this->flowPressColor = flowPressColor;
        this->initFlowStyle();
    }
}

void VideoWindow::setTimeout(int timeout)
{
    this->timeout = timeout;
}

void VideoWindow::setBorderWidth(int borderWidth)
{
    this->borderWidth = borderWidth;
    this->update();
}

void VideoWindow::setBorderColor(const QColor &borderColor)
{
    this->borderColor = borderColor;
    this->update();
}

void VideoWindow::setFocusColor(const QColor &focusColor)
{
    this->focusColor = focusColor;
    this->update();
}

void VideoWindow::setBgColor(const QColor &bgColor)
{
    this->bgColor = bgColor;
    this->update();
}

void VideoWindow::setBgText(const QString &bgText)
{
    this->bgText = bgText;
    this->update();
}

void VideoWindow::setBgImage(const QImage &bgImage)
{
    this->bgImage = bgImage;
    this->update();
}

void VideoWindow::setOSD1Visible(bool osdVisible)
{
    this->osd1Visible = osdVisible;
    this->update();
}

void VideoWindow::setOSD1FontSize(int osdFontSize)
{
    this->osd1FontSize = osdFontSize;
    this->update();
}

void VideoWindow::setOSD1Text(const QString &osdText)
{
    this->osd1Text = osdText;
    this->update();
}

void VideoWindow::setOSD1Color(const QColor &osdColor)
{
    this->osd1Color = osdColor;
    this->update();
}

void VideoWindow::setOSD1Image(const QImage &osdImage)
{
    this->osd1Image = osdImage;
    this->update();
}

void VideoWindow::setOSD1Format(const VideoWindow::OSDFormat &osdFormat)
{
    this->osd1Format = osdFormat;
    this->update();
}

void VideoWindow::setOSD1Position(const VideoWindow::OSDPosition &osdPosition)
{
    this->osd1Position = osdPosition;
    this->update();
}

void VideoWindow::setOSD2Visible(bool osdVisible)
{
    this->osd2Visible = osdVisible;
    this->update();
}

void VideoWindow::setOSD2FontSize(int osdFontSize)
{
    this->osd2FontSize = osdFontSize;
    this->update();
}

void VideoWindow::setOSD2Text(const QString &osdText)
{
    this->osd2Text = osdText;
    this->update();
}

void VideoWindow::setOSD2Color(const QColor &osdColor)
{
    this->osd2Color = osdColor;
    this->update();
}

void VideoWindow::setOSD2Image(const QImage &osdImage)
{
    this->osd2Image = osdImage;
    this->update();
}

void VideoWindow::setOSD2Format(const VideoWindow::OSDFormat &osdFormat)
{
    this->osd2Format = osdFormat;
    this->update();
}

void VideoWindow::setOSD2Position(const VideoWindow::OSDPosition &osdPosition)
{
    this->osd2Position = osdPosition;
    this->update();
}

void VideoWindow::setOSD1Format(quint8 osdFormat)
{
    setOSD1Format((VideoWindow::OSDFormat)osdFormat);
}

void VideoWindow::setOSD2Format(quint8 osdFormat)
{
    setOSD2Format((VideoWindow::OSDFormat)osdFormat);
}

void VideoWindow::setOSD1Position(quint8 osdPosition)
{
    setOSD1Position((VideoWindow::OSDPosition)osdPosition);
}

void VideoWindow::setOSD2Position(quint8 osdPosition)
{
    setOSD2Position((VideoWindow::OSDPosition)osdPosition);
}

void VideoWindow::setFaceBorder(int faceBorder)
{
    this->faceBorder = faceBorder;
    this->update();
}

void VideoWindow::setFaceColor(const QColor &faceColor)
{
    this->faceColor = faceColor;
    this->update();
}

void VideoWindow::setFaceRects(const QList<QRect> &faceRects)
{
    this->faceRects = faceRects;
    this->update();
}

void VideoWindow::open()
{
    //qDebug() << TIMEMS << "open video" << objectName();
    clear();

    //如果是图片则只显示图片就行
    image = QImage(this->property("url").toString());
    if (!image.isNull()) {
        this->update();
        return;
    }

    //thread->play();
    //thread->start();

    if (checkLive) {
        timerCheck->start();
    }

    this->setProperty("isPause", false);
}

void VideoWindow::pause()
{
    if (!this->property("isPause").toBool()) {
        //thread->pause();
        this->setProperty("isPause", true);
    }
}

void VideoWindow::next()
{
    if (this->property("isPause").toBool()) {
        //thread->next();
        this->setProperty("isPause", false);
    }
}

void VideoWindow::close()
{
    if (checkLive) {
        timerCheck->stop();
    }

    this->clear();
    //QTimer::singleShot(5, this, SLOT(clear()));
}

void VideoWindow::restart(const QString &url, int delayOpen)
{
    //qDebug() << TIMEMS << "restart video" << objectName();
    //关闭视频
    close();
    //重新设置播放地址
    setUrl(url);

    //打开视频
    if (delayOpen > 0) {
        QTimer::singleShot(delayOpen, this, SLOT(open()));
    } else {
        open();
    }
}

void VideoWindow::clear()
{
    image = QImage();
    this->update();
}

void VideoWindow::snap(const QString &fileName)
{

}
```

### `./videowindow.h`

```cpp
﻿#ifndef VIDEOWINDOW_H
#define VIDEOWINDOW_H

/**
 * 通用视频播放控件 作者:feiyangqingyun(QQ:517216493) 2018-05-01
 * 1. 可设置边框大小。
 * 2. 可设置边框颜色。
 * 3. 可设置两路OSD标签。
 * 4. 可设置是否绘制OSD标签。
 * 5. 可设置标签文本或图片。
 * 6. 可设置OSD位置 左上角、左下角、右上角、右下角。
 * 7. 可设置OSD风格 文本、日期、时间、日期时间、图片。
 * 8. 自定义半透明悬浮窗体，一排按钮。
 * 9. 悬浮按钮可自定义设置，包括背景颜色+按下颜色。
 * 10. 发送信号通知单击了哪个悬浮按钮。
 * 11. 能够识别拖进来的文件，通知url。
 * 12. 提供open close pause等接口。
 */

#include <QWidget>
#include <QDateTime>
class QTimer;

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
#define EnterEvent QEnterEvent
#else
#define EnterEvent QEvent
#endif

#ifdef quc
class Q_DECL_EXPORT VideoWindow : public QWidget
#else
class VideoWindow : public QWidget
#endif

{
    Q_OBJECT
    Q_ENUMS(OSDFormat)
    Q_ENUMS(OSDPosition)

    Q_PROPERTY(bool copyImage READ getCopyImage WRITE setCopyImage)
    Q_PROPERTY(bool checkLive READ getCheckLive WRITE setCheckLive)
    Q_PROPERTY(bool drawImage READ getDrawImage WRITE setDrawImage)
    Q_PROPERTY(bool fillImage READ getFillImage WRITE setFillImage)

    Q_PROPERTY(bool flowEnable READ getFlowEnable WRITE setFlowEnable)
    Q_PROPERTY(QColor flowBgColor READ getFlowBgColor WRITE setFlowBgColor)
    Q_PROPERTY(QColor flowPressColor READ getFlowPressColor WRITE setFlowPressColor)

    Q_PROPERTY(int timeout READ getTimeout WRITE setTimeout)
    Q_PROPERTY(int borderWidth READ getBorderWidth WRITE setBorderWidth)
    Q_PROPERTY(QColor borderColor READ getBorderColor WRITE setBorderColor)
    Q_PROPERTY(QColor focusColor READ getFocusColor WRITE setFocusColor)
    Q_PROPERTY(QColor bgColor READ getBgColor WRITE setBgColor)
    Q_PROPERTY(QString bgText READ getBgText WRITE setBgText)
    Q_PROPERTY(QImage bgImage READ getBgImage WRITE setBgImage)

    Q_PROPERTY(bool osd1Visible READ getOSD1Visible WRITE setOSD1Visible)
    Q_PROPERTY(int osd1FontSize READ getOSD1FontSize WRITE setOSD1FontSize)
    Q_PROPERTY(QString osd1Text READ getOSD1Text WRITE setOSD1Text)
    Q_PROPERTY(QColor osd1Color READ getOSD1Color WRITE setOSD1Color)
    Q_PROPERTY(QImage osd1Image READ getOSD1Image WRITE setOSD1Image)
    Q_PROPERTY(OSDFormat osd1Format READ getOSD1Format WRITE setOSD1Format)
    Q_PROPERTY(OSDPosition osd1Position READ getOSD1Position WRITE setOSD1Position)

    Q_PROPERTY(bool osd2Visible READ getOSD2Visible WRITE setOSD2Visible)
    Q_PROPERTY(int osd2FontSize READ getOSD2FontSize WRITE setOSD2FontSize)
    Q_PROPERTY(QString osd2Text READ getOSD2Text WRITE setOSD2Text)
    Q_PROPERTY(QColor osd2Color READ getOSD2Color WRITE setOSD2Color)
    Q_PROPERTY(QImage osd2Image READ getOSD2Image WRITE setOSD2Image)
    Q_PROPERTY(OSDFormat osd2Format READ getOSD2Format WRITE setOSD2Format)
    Q_PROPERTY(OSDPosition osd2Position READ getOSD2Position WRITE setOSD2Position)

public:
    //标签格式
    enum OSDFormat {
        OSDFormat_Text = 0,             //文本
        OSDFormat_Date = 1,             //日期
        OSDFormat_Time = 2,             //时间
        OSDFormat_DateTime = 3,         //日期时间
        OSDFormat_Image = 4             //图片
    };

    //标签位置
    enum OSDPosition {
        OSDPosition_Left_Top = 0,       //左上角
        OSDPosition_Left_Bottom = 1,    //左下角
        OSDPosition_Right_Top = 2,      //右上角
        OSDPosition_Right_Bottom = 3    //右下角
    };

    explicit VideoWindow(QWidget *parent = 0);
    ~VideoWindow();

protected:
    void resizeEvent(QResizeEvent *);
    void enterEvent(EnterEvent *);
    void leaveEvent(QEvent *);
    void dropEvent(QDropEvent *event);
    void dragEnterEvent(QDragEnterEvent *event);
    void paintEvent(QPaintEvent *);
    void drawBorder(QPainter *painter);
    void drawBg(QPainter *painter);
    void drawImg(QPainter *painter, QImage img);
    void drawOSD(QPainter *painter,
                 bool osdVisible,
                 int osdFontSize,
                 const QString &osdText,
                 const QColor &osdColor,
                 const QImage &osdImage,
                 const OSDFormat &osdFormat,
                 const OSDPosition &osdPosition);

private:
    QTimer *timerCheck;             //定时器检查设备是否在线
    QImage image;                   //要显示的图片
    QWidget *flowPanel;             //悬浮条面板

    bool copyImage;                 //是否拷贝图片
    bool checkLive;                 //检测是否活着
    bool drawImage;                 //是否绘制图片
    bool fillImage;                 //自动拉伸填充

    bool flowEnable;                //是否显示悬浮条
    QColor flowBgColor;             //悬浮条背景颜色
    QColor flowPressColor;          //悬浮条按下颜色

    int timeout;                    //超时时间
    int borderWidth;                //边框宽度
    QColor borderColor;             //边框颜色
    QColor focusColor;              //有焦点边框颜色
    QColor bgColor;                 //背景颜色
    QString bgText;                 //默认无图像显示文字
    QImage bgImage;                 //默认无图像背景图片

    bool osd1Visible;               //显示标签1
    int osd1FontSize;               //标签1字号
    QString osd1Text;               //标签1文本
    QColor osd1Color;               //标签1颜色
    QImage osd1Image;               //标签1图片
    OSDFormat osd1Format;           //标签1文本格式
    OSDPosition osd1Position;       //标签1位置

    bool osd2Visible;               //显示标签2
    int osd2FontSize;               //标签2字号
    QString osd2Text;               //标签2文本
    QColor osd2Color;               //标签2颜色
    QImage osd2Image;               //标签2图片
    OSDFormat osd2Format;           //标签2文本格式
    OSDPosition osd2Position;       //标签2位置

    int faceBorder;                 //人脸框粗细
    QColor faceColor;               //人脸框颜色
    QList<QRect> faceRects;         //人脸框集合

private:
    //初始化解码线程
    void initThread();
    //初始化悬浮条
    void initFlowPanel();
    //初始化悬浮条样式
    void initFlowStyle();

public:
    QImage getImage()               const;
    QPixmap getPixmap()             const;
    QString getUrl()                const;
    QDateTime getLastTime()         const;

    bool getCallback()              const;
    bool getIsPlaying()             const;
    bool getIsRtsp()                const;
    bool getIsUsbCamera()           const;

    bool getCopyImage()             const;
    bool getCheckLive()             const;
    bool getDrawImage()             const;
    bool getFillImage()             const;

    bool getFlowEnable()            const;
    QColor getFlowBgColor()         const;
    QColor getFlowPressColor()      const;

    int getTimeout()                const;
    int getBorderWidth()            const;
    QColor getBorderColor()         const;
    QColor getFocusColor()          const;
    QColor getBgColor()             const;
    QString getBgText()             const;
    QImage getBgImage()             const;

    bool getOSD1Visible()           const;
    int getOSD1FontSize()           const;
    QString getOSD1Text()           const;
    QColor getOSD1Color()           const;
    QImage getOSD1Image()           const;
    OSDFormat getOSD1Format()       const;
    OSDPosition getOSD1Position()   const;

    bool getOSD2Visible()           const;
    int getOSD2FontSize()           const;
    QString getOSD2Text()           const;
    QColor getOSD2Color()           const;
    QImage getOSD2Image()           const;
    OSDFormat getOSD2Format()       const;
    OSDPosition getOSD2Position()   const;

    int getFaceBorder()             const;
    QColor getFaceColor()           const;
    QList<QRect> getFaceRects()     const;

    QSize sizeHint()                const;
    QSize minimumSizeHint()         const;

private slots:
    //接收图像并绘制
    void updateImage(const QImage &image);
    //校验设备
    void checkVideo();
    //处理按钮单击
    void btnClicked();

Q_SIGNALS:
    //播放成功
    void receivePlayStart();
    //播放失败
    void receivePlayError();
    //播放结束
    void receivePlayFinsh();

    //总时长
    void fileLengthReceive(qint64 length);
    //当前播放时长
    void filePositionReceive(qint64 position);

    //收到图片信号
    void receiveImage(const QImage &image);

    //接收到拖曳文件
    void fileDrag(const QString &url);

    //工具栏单击
    void btnClicked(const QString &objName);

public Q_SLOTS:
    //获取长度
    uint getLength();
    //获取当前播放位置
    uint getPosition();
    //设置播放位置
    void setPosition(int position);

    //获取静音状态
    bool getMuted();
    //设置静音
    void setMuted(bool muted);

    //获取音量
    int getVolume();
    //设置音量
    void setVolume(int volume);

    //设置显示间隔
    void setInterval(int interval);
    //设置休眠时间
    void setSleepTime(int sleepTime);
    //设置检测连接超时
    void setCheckTime(int checkTime);
    //设置是否检测连接
    void setCheckConn(bool checkConn);

    //设置视频流地址
    void setUrl(const QString &url);
    //设置是否采用回调
    void setCallback(bool callback);
    //设置硬件解码器名称
    void setHardware(const QString &hardware);
    //设置通信协议
    void setTransport(const QString &transport);

    //设置是否保存文件
    void setSaveFile(bool saveFile);
    //设置保存间隔
    void setSaveInterval(int saveInterval);
    //设置定时保存文件唯一标识符
    void setFileFlag(const QString &fileFlag);
    //设置保存文件夹
    void setSavePath(const QString &savePath);
    //设置保存文件名称
    void setFileName(const QString &fileName);

    //设置是否拷贝图片
    void setCopyImage(bool copyImage);
    //设置是否检测活着
    void setCheckLive(bool checkLive);
    //设置是否实时绘制图片
    void setDrawImage(bool drawImage);
    //设置是否拉伸填充
    void setFillImage(bool fillImage);

    //设置是否启用悬浮条
    void setFlowEnable(bool flowEnable);
    //设置悬浮条背景颜色
    void setFlowBgColor(const QColor &flowBgColor);
    //设置悬浮条按下颜色
    void setFlowPressColor(const QColor &flowPressColor);

    //设置超时时间
    void setTimeout(int timeout);
    //设置边框宽度
    void setBorderWidth(int borderWidth);
    //设置边框颜色
    void setBorderColor(const QColor &borderColor);
    //设置有焦点边框颜色
    void setFocusColor(const QColor &focusColor);
    //设置背景颜色
    void setBgColor(const QColor &bgColor);
    //设置无图像文字
    void setBgText(const QString &bgText);
    //设置无图像背景图
    void setBgImage(const QImage &bgImage);

    //设置标签1是否可见
    void setOSD1Visible(bool osdVisible);
    //设置标签1文字字号
    void setOSD1FontSize(int osdFontSize);
    //设置标签1文本
    void setOSD1Text(const QString &osdText);
    //设置标签1文字颜色
    void setOSD1Color(const QColor &osdColor);
    //设置标签1图片
    void setOSD1Image(const QImage &osdImage);
    //设置标签1格式
    void setOSD1Format(const OSDFormat &osdFormat);
    //设置标签1位置
    void setOSD1Position(const OSDPosition &osdPosition);

    //设置标签2是否可见
    void setOSD2Visible(bool osdVisible);
    //设置标签2文字字号
    void setOSD2FontSize(int osdFontSize);
    //设置标签2文本
    void setOSD2Text(const QString &osdText);
    //设置标签2文字颜色
    void setOSD2Color(const QColor &osdColor);
    //设置标签2图片
    void setOSD2Image(const QImage &osdImage);
    //设置标签2格式
    void setOSD2Format(const OSDFormat &osdFormat);
    //设置标签2位置
    void setOSD2Position(const OSDPosition &osdPosition);

    //设置值自动进行枚举转换
    void setOSD1Format(quint8 osdFormat);
    void setOSD2Format(quint8 osdFormat);
    void setOSD1Position(quint8 osdPosition);
    void setOSD2Position(quint8 osdPosition);

    //设置人脸框粗细
    void setFaceBorder(int faceBorder);
    //设置人脸框颜色
    void setFaceColor(const QColor &faceColor);
    //设置人脸框区域集合
    void setFaceRects(const QList<QRect> &faceRects);

    //打开设备
    void open();
    //暂停播放
    void pause();
    //继续播放
    void next();
    //关闭设备
    void close();
    //重新加载
    void restart(const QString &url, int delayOpen = 500);
    //清空图片
    void clear();
    //截图快照
    void snap(const QString &fileName);
};

#endif // VIDEOWINDOW_H
```

### `./frmvideowindow.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmVideoWindow</class>
 <widget class="QWidget" name="frmVideoWindow">
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
   <item row="0" column="1">
    <widget class="VideoWindow" name="videoWindow2" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
    </widget>
   </item>
   <item row="0" column="0">
    <widget class="VideoWindow" name="videoWindow1" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
    </widget>
   </item>
   <item row="1" column="1">
    <widget class="VideoWindow" name="videoWindow4" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
    </widget>
   </item>
   <item row="1" column="0">
    <widget class="VideoWindow" name="videoWindow3" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
    </widget>
   </item>
   <item row="2" column="0" colspan="2">
    <widget class="QLabel" name="label">
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
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>VideoWindow</class>
   <extends>QWidget</extends>
   <header>videowindow.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
