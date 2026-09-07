---
tags:
  - Qt
  - 控件相关
  - 高质量
---
# 导航按钮 (`navbutton`)

> 导航按钮

## 效果图

![[QWidgetDemo_assert/navbutton.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/navbutton`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QColor QPixmap QWidget QList QString QAbstractButton QSize QPainter QFont QEvent QTextCodec QPushButton QBrush QIcon QPoint QObject QRect QLabel QToolButton QVBoxLayout

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmnavbutton.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmnavbutton.h"
#include "ui_frmnavbutton.h"
#include "navbutton.h"
#include "iconhelper.h"
#include "qdebug.h"

frmNavButton::frmNavButton(QWidget *parent) : QWidget(parent), ui(new Ui::frmNavButton)
{
    ui->setupUi(this);
    this->initForm();
    this->initBtn1();
    this->initBtn2();
    this->initBtn3();
    this->initBtn4();
    this->initBtn5();
    this->initBtn6();
    this->initBtn7();
}

frmNavButton::~frmNavButton()
{
    delete ui;
}

void frmNavButton::initForm()
{
    icons << 0xf17b << 0xf002 << 0xf013 << 0xf021 << 0xf0e0 << 0xf135;

    ui->navButton11->setChecked(true);
    ui->navButton23->setChecked(true);
    ui->navButton31->setChecked(true);
    ui->navButton44->setChecked(true);
    ui->navButton53->setChecked(true);
    ui->navButton61->setChecked(true);
    ui->navButton75->setChecked(true);

    //设置整体圆角
    ui->widgetNav5->setStyleSheet(".QWidget{background:#292929;border:1px solid #292929;border-radius:20px;}");
}

void frmNavButton::initBtn1()
{
    quint32 size = 15;
    quint32 pixWidth = 15;
    quint32 pixHeight = 15;

    //从图形字体获得图片,也可以从资源文件或者路径文件获取
    int icon = 0xf061;
    QPixmap iconNormal = IconHelper::getPixmap(QColor(100, 100, 100).name(), icon, size, pixWidth, pixHeight);
    QPixmap iconHover = IconHelper::getPixmap(QColor(255, 255, 255).name(), icon, size, pixWidth, pixHeight);
    QPixmap iconCheck = IconHelper::getPixmap(QColor(255, 255, 255).name(), icon, size, pixWidth, pixHeight);

    btns1 << ui->navButton11 << ui->navButton12 << ui->navButton13 << ui->navButton14;
    for (int i = 0; i < btns1.count(); i++) {
        NavButton *btn = btns1.at(i);
        btn->setPaddingLeft(32);
        btn->setLineSpace(6);

        btn->setShowIcon(true);
        btn->setIconSpace(15);
        btn->setIconSize(QSize(10, 10));
        btn->setIconNormal(iconNormal);
        btn->setIconHover(iconHover);
        btn->setIconCheck(iconCheck);

        connect(btn, SIGNAL(clicked(bool)), this, SLOT(buttonClick1()));
    }
}

void frmNavButton::initBtn2()
{
    quint32 size = 15;
    quint32 pixWidth = 20;
    quint32 pixHeight = 20;

    QColor normalBgColor = QColor("#2D9191");
    QColor hoverBgColor = QColor("#187294");
    QColor checkBgColor = QColor("#145C75");
    QColor normalTextColor = QColor("#FFFFFF");
    QColor hoverTextColor = QColor("#FFFFFF");
    QColor checkTextColor = QColor("#FFFFFF");

    btns2 << ui->navButton21 << ui->navButton22 << ui->navButton23 << ui->navButton24;
    for (int i = 0; i < btns2.count(); i++) {
        NavButton *btn = btns2.at(i);
        btn->setPaddingLeft(35);
        btn->setLineSpace(0);
        btn->setLineWidth(8);
        btn->setLineColor(QColor(255, 107, 107));
        btn->setShowTriangle(true);

        btn->setShowIcon(true);
        btn->setIconSpace(10);
        btn->setIconSize(QSize(22, 22));

        //分开设置图标
        int icon = icons.at(i);
        QPixmap iconNormal = IconHelper::getPixmap(normalTextColor.name(), icon, size, pixWidth, pixHeight);
        QPixmap iconHover = IconHelper::getPixmap(hoverTextColor.name(), icon, size, pixWidth, pixHeight);
        QPixmap iconCheck = IconHelper::getPixmap(checkTextColor.name(), icon, size, pixWidth, pixHeight);

        btn->setIconNormal(iconNormal);
        btn->setIconHover(iconHover);
        btn->setIconCheck(iconCheck);

        btn->setNormalBgColor(normalBgColor);
        btn->setHoverBgColor(hoverBgColor);
        btn->setCheckBgColor(checkBgColor);
        btn->setNormalTextColor(normalTextColor);
        btn->setHoverTextColor(hoverTextColor);
        btn->setCheckTextColor(checkTextColor);

        connect(btn, SIGNAL(clicked(bool)), this, SLOT(buttonClick2()));
    }
}

void frmNavButton::initBtn3()
{
    quint32 size = 15;
    quint32 pixWidth = 20;
    quint32 pixHeight = 20;

    QColor normalBgColor = QColor("#292F38");
    QColor hoverBgColor = QColor("#1D2025");
    QColor checkBgColor = QColor("#1D2025");
    QColor normalTextColor = QColor("#54626F");
    QColor hoverTextColor = QColor("#FDFDFD");
    QColor checkTextColor = QColor("#FDFDFD");

    btns3 << ui->navButton31 << ui->navButton32 << ui->navButton33 << ui->navButton34;
    for (int i = 0; i < btns3.count(); i++) {
        NavButton *btn = btns3.at(i);
        btn->setPaddingLeft(35);
        btn->setLineWidth(10);
        btn->setLineColor(QColor("#029FEA"));
        btn->setShowTriangle(true);
        btn->setTextAlign(NavButton::TextAlign_Left);
        btn->setTrianglePosition(NavButton::TrianglePosition_Left);
        btn->setLinePosition(NavButton::LinePosition_Right);

        btn->setShowIcon(true);
        btn->setIconSpace(10);
        btn->setIconSize(QSize(22, 22));

        //分开设置图标
        int icon = icons.at(i);
        QPixmap iconNormal = IconHelper::getPixmap(normalTextColor.name(), icon, size, pixWidth, pixHeight);
        QPixmap iconHover = IconHelper::getPixmap(hoverTextColor.name(), icon, size, pixWidth, pixHeight);
        QPixmap iconCheck = IconHelper::getPixmap(checkTextColor.name(), icon, size, pixWidth, pixHeight);

        btn->setIconNormal(iconNormal);
        btn->setIconHover(iconHover);
        btn->setIconCheck(iconCheck);

        btn->setNormalBgColor(normalBgColor);
        btn->setHoverBgColor(hoverBgColor);
        btn->setCheckBgColor(checkBgColor);
        btn->setNormalTextColor(normalTextColor);
        btn->setHoverTextColor(hoverTextColor);
        btn->setCheckTextColor(checkTextColor);

        connect(btn, SIGNAL(clicked(bool)), this, SLOT(buttonClick3()));
    }
}

void frmNavButton::initBtn4()
{
    quint32 size = 15;
    quint32 pixWidth = 15;
    quint32 pixHeight = 15;

    int icon = 0xf105;
    QPixmap iconNormal = IconHelper::getPixmap(QColor(100, 100, 100).name(), icon, size, pixWidth, pixHeight);
    QPixmap iconHover = IconHelper::getPixmap(QColor(255, 255, 255).name(), icon, size, pixWidth, pixHeight);
    QPixmap iconCheck = IconHelper::getPixmap(QColor(255, 255, 255).name(), icon, size, pixWidth, pixHeight);

    btns4 << ui->navButton41 << ui->navButton42 << ui->navButton43 << ui->navButton44;
    for (int i = 0; i < btns4.count(); i++) {
        NavButton *btn = btns4.at(i);
        btn->setLineSpace(10);
        btn->setLineWidth(10);
        btn->setPaddingRight(35);
        btn->setShowTriangle(true);
        btn->setTextAlign(NavButton::TextAlign_Right);
        btn->setTrianglePosition(NavButton::TrianglePosition_Left);
        btn->setLinePosition(NavButton::LinePosition_Right);

        btn->setShowIcon(true);
        btn->setIconSpace(20);
        btn->setIconSize(QSize(15, 15));
        btn->setIconNormal(iconNormal);
        btn->setIconHover(iconHover);
        btn->setIconCheck(iconCheck);

        connect(btn, SIGNAL(clicked(bool)), this, SLOT(buttonClick4()));
    }
}

void frmNavButton::initBtn5()
{
    QFont font;
    font.setPixelSize(15);
    font.setBold(true);

    quint32 size = 15;
    quint32 pixWidth = 20;
    quint32 pixHeight = 20;

    QColor normalBgColor = QColor("#292929");
    QColor hoverBgColor = QColor("#064077");
    QColor checkBgColor = QColor("#10689A");
    QColor normalTextColor = QColor("#FFFFFF");
    QColor hoverTextColor = Qt::yellow;
    QColor checkTextColor = QColor("#FFFFFF");

    btns5 << ui->navButton51 << ui->navButton52 << ui->navButton53 << ui->navButton54 << ui->navButton55;
    for (int i = 0; i < btns5.count(); i++) {
        NavButton *btn = btns5.at(i);
        btn->setFont(font);
        btn->setPaddingLeft(20);
        btn->setShowLine(false);
        btn->setTextAlign(NavButton::TextAlign_Center);
        btn->setLinePosition(NavButton::LinePosition_Bottom);

        btn->setShowIcon(true);
        btn->setIconSpace(15);
        btn->setIconSize(QSize(22, 22));

        //分开设置图标
        int icon = icons.at(i);
        QPixmap iconNormal = IconHelper::getPixmap(normalTextColor.name(), icon, size, pixWidth, pixHeight);
        QPixmap iconHover = IconHelper::getPixmap(hoverTextColor.name(), icon, size, pixWidth, pixHeight);
        QPixmap iconCheck = IconHelper::getPixmap(checkTextColor.name(), icon, size, pixWidth, pixHeight);

        btn->setIconNormal(iconNormal);
        btn->setIconHover(iconHover);
        btn->setIconCheck(iconCheck);

        btn->setNormalBgColor(normalBgColor);
        btn->setHoverBgColor(hoverBgColor);
        btn->setCheckBgColor(checkBgColor);
        btn->setNormalTextColor(normalTextColor);
        btn->setHoverTextColor(hoverTextColor);
        btn->setCheckTextColor(checkTextColor);

        connect(btn, SIGNAL(clicked(bool)), this, SLOT(buttonClick5()));
    }
}

void frmNavButton::initBtn6()
{
    QFont font;
    font.setPixelSize(15);
    font.setBold(true);

    quint32 size = 15;
    quint32 pixWidth = 20;
    quint32 pixHeight = 20;

    QColor normalBgColor = QColor("#E6393D");
    QColor hoverBgColor = QColor("#EE0000");
    QColor checkBgColor = QColor("#A40001");
    QColor normalTextColor = QColor("#FFFFFF");
    QColor hoverTextColor = QColor("#FFFFFF");
    QColor checkTextColor = QColor("#FFFFFF");

    btns6 << ui->navButton61 << ui->navButton62 << ui->navButton63 << ui->navButton64 << ui->navButton65;
    for (int i = 0; i < btns6.count(); i++) {
        NavButton *btn = btns6.at(i);
        btn->setFont(font);
        btn->setPaddingLeft(20);
        btn->setShowLine(false);
        btn->setTextAlign(NavButton::TextAlign_Center);
        btn->setLinePosition(NavButton::LinePosition_Bottom);

        btn->setShowIcon(true);
        btn->setIconSpace(15);
        btn->setIconSize(QSize(22, 22));

        //分开设置图标
        int icon = icons.at(i);
        QPixmap iconNormal = IconHelper::getPixmap(normalTextColor.name(), icon, size, pixWidth, pixHeight);
        QPixmap iconHover = IconHelper::getPixmap(hoverTextColor.name(), icon, size, pixWidth, pixHeight);
        QPixmap iconCheck = IconHelper::getPixmap(checkTextColor.name(), icon, size, pixWidth, pixHeight);

        btn->setIconNormal(iconNormal);
        btn->setIconHover(iconHover);
        btn->setIconCheck(iconCheck);

        btn->setNormalBgColor(normalBgColor);
        btn->setHoverBgColor(hoverBgColor);
        btn->setCheckBgColor(checkBgColor);
        btn->setNormalTextColor(normalTextColor);
        btn->setHoverTextColor(hoverTextColor);
        btn->setCheckTextColor(checkTextColor);

        connect(btn, SIGNAL(clicked(bool)), this, SLOT(buttonClick6()));
    }
}

void frmNavButton::initBtn7()
{
    QFont font;
    font.setPixelSize(15);
    font.setBold(true);

    QColor normalTextColor = QColor("#FFFFFF");
    QColor hoverTextColor = QColor("#FFFFFF");
    QColor checkTextColor = QColor("#FFFFFF");

    //设置背景色为画刷
    QLinearGradient normalBgBrush(0, 0, 0, ui->navButton71->height());
    normalBgBrush.setColorAt(0.0, QColor("#3985BF"));
    normalBgBrush.setColorAt(0.5, QColor("#2972A9"));
    normalBgBrush.setColorAt(1.0, QColor("#1C6496"));

    QLinearGradient hoverBgBrush(0, 0, 0, ui->navButton71->height());
    hoverBgBrush.setColorAt(0.0, QColor("#4897D1"));
    hoverBgBrush.setColorAt(0.5, QColor("#3283BC"));
    hoverBgBrush.setColorAt(1.0, QColor("#3088C3"));

    btns7 << ui->navButton71 << ui->navButton72 << ui->navButton73 << ui->navButton74 << ui->navButton75 << ui->navButton76;
    for (int i = 0; i < btns7.count(); i++) {
        NavButton *btn = btns7.at(i);
        btn->setFont(font);
        btn->setPaddingLeft(0);
        btn->setLineSpace(0);
        btn->setShowTriangle(true);
        btn->setTextAlign(NavButton::TextAlign_Center);
        btn->setTrianglePosition(NavButton::TrianglePosition_Bottom);
        btn->setLinePosition(NavButton::LinePosition_Top);

        btn->setNormalTextColor(normalTextColor);
        btn->setHoverTextColor(hoverTextColor);
        btn->setCheckTextColor(checkTextColor);

        btn->setNormalBgBrush(normalBgBrush);
        btn->setHoverBgBrush(hoverBgBrush);
        btn->setCheckBgBrush(hoverBgBrush);

        connect(btn, SIGNAL(clicked(bool)), this, SLOT(buttonClick7()));
    }
}

void frmNavButton::buttonClick1()
{
    NavButton *b = (NavButton *)sender();
    qDebug() << "当前按下" << b->text();
    for (int i = 0; i < btns1.count(); i++) {
        NavButton *btn = btns1.at(i);
        btn->setChecked(b == btn);
    }
}

void frmNavButton::buttonClick2()
{
    NavButton *b = (NavButton *)sender();
    qDebug() << "当前按下" << b->text();
    for (int i = 0; i < btns2.count(); i++) {
        NavButton *btn = btns2.at(i);
        btn->setChecked(b == btn);
    }
}

void frmNavButton::buttonClick3()
{
    NavButton *b = (NavButton *)sender();
    qDebug() << "当前按下" << b->text();
    for (int i = 0; i < btns3.count(); i++) {
        NavButton *btn = btns3.at(i);
        btn->setChecked(b == btn);
    }
}

void frmNavButton::buttonClick4()
{
    NavButton *b = (NavButton *)sender();
    qDebug() << "当前按下" << b->text();
    for (int i = 0; i < btns4.count(); i++) {
        NavButton *btn = btns4.at(i);
        btn->setChecked(b == btn);
    }
}

void frmNavButton::buttonClick5()
{
    NavButton *b = (NavButton *)sender();
    qDebug() << "当前按下" << b->text();
    for (int i = 0; i < btns5.count(); i++) {
        NavButton *btn = btns5.at(i);
        btn->setChecked(b == btn);
    }
}

void frmNavButton::buttonClick6()
{
    NavButton *b = (NavButton *)sender();
    qDebug() << "当前按下" << b->text();
    for (int i = 0; i < btns6.count(); i++) {
        NavButton *btn = btns6.at(i);
        btn->setChecked(b == btn);
    }
}

void frmNavButton::buttonClick7()
{
    NavButton *b = (NavButton *)sender();
    qDebug() << "当前按下" << b->text();
    for (int i = 0; i < btns7.count(); i++) {
        NavButton *btn = btns7.at(i);
        btn->setChecked(b == btn);
    }
}
```

### `./frmnavbutton.h`

```cpp
﻿#ifndef FRMNAVBUTTON_H
#define FRMNAVBUTTON_H

#include <QWidget>

class NavButton;

namespace Ui {
class frmNavButton;
}

class frmNavButton : public QWidget
{
    Q_OBJECT

public:
    explicit frmNavButton(QWidget *parent = 0);
    ~frmNavButton();

private:
    Ui::frmNavButton *ui;
    QList<int> icons;
    QList<NavButton *> btns1;
    QList<NavButton *> btns2;
    QList<NavButton *> btns3;
    QList<NavButton *> btns4;
    QList<NavButton *> btns5;
    QList<NavButton *> btns6;
    QList<NavButton *> btns7;

private slots:
    void initForm();
    void initBtn1();
    void initBtn2();
    void initBtn3();
    void initBtn4();
    void initBtn5();
    void initBtn6();
    void initBtn7();

    void buttonClick1();
    void buttonClick2();
    void buttonClick3();
    void buttonClick4();
    void buttonClick5();
    void buttonClick6();
    void buttonClick7();
};

#endif // FRMNAVBUTTON_H
```

### `./iconhelper.cpp`

```cpp
﻿#include "iconhelper.h"

IconHelper *IconHelper::iconFontAliBaBa = 0;
IconHelper *IconHelper::iconFontAwesome = 0;
IconHelper *IconHelper::iconFontAwesome6 = 0;
IconHelper *IconHelper::iconFontWeather = 0;
int IconHelper::iconFontIndex = -1;

void IconHelper::initFont()
{
    static bool isInit = false;
    if (!isInit) {
        isInit = true;
        if (iconFontAliBaBa == 0) {
            iconFontAliBaBa = new IconHelper(":/font/iconfont.ttf", "iconfont");
        }
        if (iconFontAwesome == 0) {
            iconFontAwesome = new IconHelper(":/font/fontawesome-webfont.ttf", "FontAwesome");
        }
        if (iconFontAwesome6 == 0) {
            iconFontAwesome6 = new IconHelper(":/font/fa-regular-400.ttf", "Font Awesome 6 Pro Regular");
        }
        if (iconFontWeather == 0) {
            iconFontWeather = new IconHelper(":/font/pe-icon-set-weather.ttf", "pe-icon-set-weather");
        }
    }
}

void IconHelper::setIconFontIndex(int index)
{
    iconFontIndex = index;
}

QFont IconHelper::getIconFontAliBaBa()
{
    initFont();
    return iconFontAliBaBa->getIconFont();
}

QFont IconHelper::getIconFontAwesome()
{
    initFont();
    return iconFontAwesome->getIconFont();
}

QFont IconHelper::getIconFontAwesome6()
{
    initFont();
    return iconFontAwesome6->getIconFont();
}

QFont IconHelper::getIconFontWeather()
{
    initFont();
    return iconFontWeather->getIconFont();
}

IconHelper *IconHelper::getIconHelper(int icon)
{
    initFont();

    //指定了字体索引则取对应索引的字体类
    //没指定则自动根据不同的字体的值选择对应的类
    //由于部分值范围冲突所以可以指定索引来取
    //fontawesome   0xf000-0xf2e0
    //fontawesome6  0xe000-0xe33d 0xf000-0xf8ff
    //iconfont      0xe501-0xe793 0xe8d5-0xea5d 0xeb00-0xec00
    //weather       0xe900-0xe9cf

    IconHelper *iconHelper = iconFontAwesome;
    if (iconFontIndex < 0) {
        if ((icon >= 0xe501 && icon <= 0xe793) || (icon >= 0xe8d5 && icon <= 0xea5d) || (icon >= 0xeb00 && icon <= 0xec00)) {
            iconHelper = iconFontAliBaBa;
        }
    } else if (iconFontIndex == 0) {
        iconHelper = iconFontAliBaBa;
    } else if (iconFontIndex == 1) {
        iconHelper = iconFontAwesome;
    } else if (iconFontIndex == 2) {
        iconHelper = iconFontAwesome6;
    } else if (iconFontIndex == 3) {
        iconHelper = iconFontWeather;
    }

    return iconHelper;
}

void IconHelper::setIcon(QLabel *lab, int icon, quint32 size)
{
    getIconHelper(icon)->setIcon1(lab, icon, size);
}

void IconHelper::setIcon(QAbstractButton *btn, int icon, quint32 size)
{
    getIconHelper(icon)->setIcon1(btn, icon, size);
}

void IconHelper::setPixmap(QAbstractButton *btn, const QColor &color, int icon, quint32 size,
                           quint32 width, quint32 height, int flags)
{
    getIconHelper(icon)->setPixmap1(btn, color, icon, size, width, height, flags);
}

QPixmap IconHelper::getPixmap(const QColor &color, int icon, quint32 size,
                              quint32 width, quint32 height, int flags)
{
    return getIconHelper(icon)->getPixmap1(color, icon, size, width, height, flags);
}

void IconHelper::setStyle(QWidget *widget, QList<QPushButton *> btns,
                          QList<int> icons, const IconHelper::StyleColor &styleColor)
{
    int icon = icons.first();
    getIconHelper(icon)->setStyle1(widget, btns, icons, styleColor);
}

void IconHelper::setStyle(QWidget *widget, QList<QToolButton *> btns,
                          QList<int> icons, const IconHelper::StyleColor &styleColor)
{
    int icon = icons.first();
    getIconHelper(icon)->setStyle1(widget, btns, icons, styleColor);
}

void IconHelper::setStyle(QWidget *widget, QList<QAbstractButton *> btns,
                          QList<int> icons, const IconHelper::StyleColor &styleColor)
{
    int icon = icons.first();
    getIconHelper(icon)->setStyle1(widget, btns, icons, styleColor);
}


IconHelper::IconHelper(const QString &fontFile, const QString &fontName, QObject *parent) : QObject(parent)
{
    //判断图形字体是否存在,不存在则加入
    //这里暂时限制在同一个项目中只加载一次字体文件
    QFontDatabase fontDb;
    bool exist = false;//fontDb.families().contains(fontName);
    if (!exist && QFile(fontFile).exists()) {
        int fontId = fontDb.addApplicationFont(fontFile);
        QStringList listName = fontDb.applicationFontFamilies(fontId);
        if (listName.count() == 0) {
            qDebug() << QString("load %1 error").arg(fontName);
        }
    }

    //再次判断是否包含字体名称防止加载失败
    if (fontDb.families().contains(fontName)) {
        iconFont = QFont(fontName);
#if (QT_VERSION >= QT_VERSION_CHECK(4,8,0))
        iconFont.setHintingPreference(QFont::PreferNoHinting);
#endif
    }
}

bool IconHelper::eventFilter(QObject *watched, QEvent *event)
{
    //根据不同的
    if (watched->inherits("QAbstractButton")) {
        QAbstractButton *btn = (QAbstractButton *)watched;
        int index = btns.indexOf(btn);
        if (index >= 0) {
            //不同的事件设置不同的图标,同时区分选中的和没有选中的
            int type = event->type();
            if (btn->isChecked()) {
                if (type == QEvent::MouseButtonPress) {
                    QMouseEvent *mouseEvent = (QMouseEvent *)event;
                    if (mouseEvent->button() == Qt::LeftButton) {
                        btn->setIcon(QIcon(pixChecked.at(index)));
                    }
                } else if (type == QEvent::Enter) {
                    btn->setIcon(QIcon(pixChecked.at(index)));
                } else if (type == QEvent::Leave) {
                    btn->setIcon(QIcon(pixChecked.at(index)));
                }
            } else {
                if (type == QEvent::MouseButtonPress) {
                    QMouseEvent *mouseEvent = (QMouseEvent *)event;
                    if (mouseEvent->button() == Qt::LeftButton) {
                        btn->setIcon(QIcon(pixPressed.at(index)));
                    }
                } else if (type == QEvent::Enter) {
                    btn->setIcon(QIcon(pixHover.at(index)));
                } else if (type == QEvent::Leave) {
                    btn->setIcon(QIcon(pixNormal.at(index)));
                }
            }
        }
    }

    return QObject::eventFilter(watched, event);
}

void IconHelper::toggled(bool checked)
{
    //选中和不选中设置不同的图标
    QAbstractButton *btn = (QAbstractButton *)sender();
    int index = btns.indexOf(btn);
    if (checked) {
        btn->setIcon(QIcon(pixChecked.at(index)));
    } else {
        btn->setIcon(QIcon(pixNormal.at(index)));
    }
}

QFont IconHelper::getIconFont()
{
    return this->iconFont;
}

void IconHelper::setIcon1(QLabel *lab, int icon, quint32 size)
{
    iconFont.setPixelSize(size);
    lab->setFont(iconFont);
    lab->setText((QChar)icon);
}

void IconHelper::setIcon1(QAbstractButton *btn, int icon, quint32 size)
{
    iconFont.setPixelSize(size);
    btn->setFont(iconFont);
    btn->setText((QChar)icon);
}

void IconHelper::setPixmap1(QAbstractButton *btn, const QColor &color, int icon, quint32 size,
                            quint32 width, quint32 height, int flags)
{
    btn->setIcon(getPixmap1(color, icon, size, width, height, flags));
}

QPixmap IconHelper::getPixmap1(const QColor &color, int icon, quint32 size,
                               quint32 width, quint32 height, int flags)
{
    //主动绘制图形字体到图片
    QPixmap pix(width, height);
    pix.fill(Qt::transparent);

    QPainter painter;
    painter.begin(&pix);
    painter.setRenderHints(QPainter::Antialiasing | QPainter::TextAntialiasing);
    painter.setPen(color);

    iconFont.setPixelSize(size);
    painter.setFont(iconFont);
    painter.drawText(pix.rect(), flags, (QChar)icon);
    painter.end();
    return pix;
}

void IconHelper::setStyle1(QWidget *widget, QList<QPushButton *> btns, QList<int> icons, const IconHelper::StyleColor &styleColor)
{
    QList<QAbstractButton *> list;
    foreach (QPushButton *btn, btns) {
        list << btn;
    }

    setStyle(widget, list, icons, styleColor);
}

void IconHelper::setStyle1(QWidget *widget, QList<QToolButton *> btns, QList<int> icons, const IconHelper::StyleColor &styleColor)
{
    QList<QAbstractButton *> list;
    foreach (QToolButton *btn, btns) {
        list << btn;
    }

    setStyle(widget, list, icons, styleColor);
}

void IconHelper::setStyle1(QWidget *widget, QList<QAbstractButton *> btns, QList<int> icons, const IconHelper::StyleColor &styleColor)
{
    int btnCount = btns.count();
    int iconCount = icons.count();
    if (btnCount <= 0 || iconCount <= 0 || btnCount != iconCount) {
        return;
    }

    QString position = styleColor.position;
    quint32 btnWidth = styleColor.btnWidth;
    quint32 btnHeight = styleColor.btnHeight;
    quint32 iconSize = styleColor.iconSize;
    quint32 iconWidth = styleColor.iconWidth;
    quint32 iconHeight = styleColor.iconHeight;
    quint32 borderWidth = styleColor.borderWidth;

    //根据不同的位置计算边框
    QString strBorder;
    if (position == "top") {
        strBorder = QString("border-width:%1px 0px 0px 0px;padding-top:%1px;padding-bottom:%2px;")
                    .arg(borderWidth).arg(borderWidth * 2);
    } else if (position == "right") {
        strBorder = QString("border-width:0px %1px 0px 0px;padding-right:%1px;padding-left:%2px;")
                    .arg(borderWidth).arg(borderWidth * 2);
    } else if (position == "bottom") {
        strBorder = QString("border-width:0px 0px %1px 0px;padding-bottom:%1px;padding-top:%2px;")
                    .arg(borderWidth).arg(borderWidth * 2);
    } else if (position == "left") {
        strBorder = QString("border-width:0px 0px 0px %1px;padding-left:%1px;padding-right:%2px;")
                    .arg(borderWidth).arg(borderWidth * 2);
    }

    //如果图标是左侧显示则需要让没有选中的按钮左侧也有加深的边框,颜色为背景颜色
    //如果图标在文字上面而设置的边框是 top bottom 也需要启用加深边框
    QStringList qss;
    if (styleColor.defaultBorder) {
        qss << QString("QWidget[flag=\"%1\"] QAbstractButton{border-style:solid;border-radius:0px;%2border-color:%3;color:%4;background:%5;}")
            .arg(position).arg(strBorder).arg(styleColor.normalBgColor).arg(styleColor.normalTextColor).arg(styleColor.normalBgColor);
    } else {
        qss << QString("QWidget[flag=\"%1\"] QAbstractButton{border-style:none;border-radius:0px;padding:5px;color:%2;background:%3;}")
            .arg(position).arg(styleColor.normalTextColor).arg(styleColor.normalBgColor);
    }

    //悬停+按下+选中
    qss << QString("QWidget[flag=\"%1\"] QAbstractButton:hover{border-style:solid;%2border-color:%3;color:%4;background:%5;}")
        .arg(position).arg(strBorder).arg(styleColor.borderColor).arg(styleColor.hoverTextColor).arg(styleColor.hoverBgColor);
    qss << QString("QWidget[flag=\"%1\"] QAbstractButton:pressed{border-style:solid;%2border-color:%3;color:%4;background:%5;}")
        .arg(position).arg(strBorder).arg(styleColor.borderColor).arg(styleColor.pressedTextColor).arg(styleColor.pressedBgColor);
    qss << QString("QWidget[flag=\"%1\"] QAbstractButton:checked{border-style:solid;%2border-color:%3;color:%4;background:%5;}")
        .arg(position).arg(strBorder).arg(styleColor.borderColor).arg(styleColor.checkedTextColor).arg(styleColor.checkedBgColor);

    //窗体背景颜色+按钮背景颜色
    qss << QString("QWidget#%1{background:%2;}")
        .arg(widget->objectName()).arg(styleColor.normalBgColor);
    qss << QString("QWidget>QAbstractButton{border-width:0px;background-color:%1;color:%2;}")
        .arg(styleColor.normalBgColor).arg(styleColor.normalTextColor);
    qss << QString("QWidget>QAbstractButton:hover{background-color:%1;color:%2;}")
        .arg(styleColor.hoverBgColor).arg(styleColor.hoverTextColor);
    qss << QString("QWidget>QAbstractButton:pressed{background-color:%1;color:%2;}")
        .arg(styleColor.pressedBgColor).arg(styleColor.pressedTextColor);
    qss << QString("QWidget>QAbstractButton:checked{background-color:%1;color:%2;}")
        .arg(styleColor.checkedBgColor).arg(styleColor.checkedTextColor);

    //按钮宽度高度
    if (btnWidth > 0) {
        qss << QString("QWidget>QAbstractButton{min-width:%1px;}").arg(btnWidth);
    }
    if (btnHeight > 0) {
        qss << QString("QWidget>QAbstractButton{min-height:%1px;}").arg(btnHeight);
    }

    //设置样式表
    widget->setStyleSheet(qss.join(""));

    //可能会重复调用设置所以先要移除上一次的
    for (int i = 0; i < btnCount; ++i) {
        for (int j = 0; j < this->btns.count(); j++) {
            if (this->btns.at(j) == btns.at(i)) {
                disconnect(btns.at(i), SIGNAL(toggled(bool)), this, SLOT(toggled(bool)));
                this->btns.at(j)->removeEventFilter(this);
                this->btns.removeAt(j);
                this->pixNormal.removeAt(j);
                this->pixHover.removeAt(j);
                this->pixPressed.removeAt(j);
                this->pixChecked.removeAt(j);
                break;
            }
        }
    }

    //存储对应按钮对象,方便鼠标移上去的时候切换图片
    int checkedIndex = -1;
    for (int i = 0; i < btnCount; ++i) {
        int icon = icons.at(i);
        QPixmap pixNormal = getPixmap1(styleColor.normalTextColor, icon, iconSize, iconWidth, iconHeight);
        QPixmap pixHover = getPixmap1(styleColor.hoverTextColor, icon, iconSize, iconWidth, iconHeight);
        QPixmap pixPressed = getPixmap1(styleColor.pressedTextColor, icon, iconSize, iconWidth, iconHeight);
        QPixmap pixChecked = getPixmap1(styleColor.checkedTextColor, icon, iconSize, iconWidth, iconHeight);

        //记住最后选中的按钮
        QAbstractButton *btn = btns.at(i);
        if (btn->isChecked()) {
            checkedIndex = i;
        }

        btn->setIcon(QIcon(pixNormal));
        btn->setIconSize(QSize(iconWidth, iconHeight));
        btn->installEventFilter(this);
        connect(btn, SIGNAL(toggled(bool)), this, SLOT(toggled(bool)));

        this->btns << btn;
        this->pixNormal << pixNormal;
        this->pixHover << pixHover;
        this->pixPressed << pixPressed;
        this->pixChecked << pixChecked;
    }

    //主动触发一下选中的按钮
    if (checkedIndex >= 0) {
        QMetaObject::invokeMethod(btns.at(checkedIndex), "toggled", Q_ARG(bool, true));
    }
}
```

### `./iconhelper.h`

```cpp
﻿#ifndef ICONHELPER_H
#define ICONHELPER_H

/**
 * 超级图形字体类 作者:feiyangqingyun(QQ:517216493) 2016-11-23
 * 1. 可传入多种图形字体文件，一个类通用所有图形字体。
 * 2. 默认已经内置了阿里巴巴图形字体FontAliBaBa、国际知名图形字体FontAwesome、天气图形字体FontWeather。
 * 3. 可设置 QLabel、QAbstractButton 文本为图形字体。
 * 4. 可设置图形字体作为 QAbstractButton 按钮图标。
 * 5. 内置万能的方法 getPixmap 将图形字体值转换为图片。
 * 6. 无论是设置文本、图标、图片等都可以设置图标的大小、尺寸、颜色等参数。
 * 7. 内置超级导航栏样式设置，将图形字体作为图标设置到按钮。
 * 8. 支持各种颜色设置比如正常颜色、悬停颜色、按下颜色、选中颜色。
 * 9. 可设置导航的位置为 left、right、top、bottom 四种。
 * 10. 可设置导航加深边框颜色和粗细大小。
 * 11. 导航面板的各种切换效果比如鼠标悬停、按下、选中等都自动处理掉样式设置。
 * 12. 全局静态方法，接口丰富，使用极其简单方便。
 */

#include <QtGui>
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
#include <QtWidgets>
#endif

#ifdef quc
class Q_DECL_EXPORT IconHelper : public QObject
#else
class IconHelper : public QObject
#endif

{
    Q_OBJECT

private:
    //阿里巴巴图形字体类
    static IconHelper *iconFontAliBaBa;
    //FontAwesome图形字体类
    static IconHelper *iconFontAwesome;
    //FontAwesome6图形字体类
    static IconHelper *iconFontAwesome6;
    //天气图形字体类
    static IconHelper *iconFontWeather;
    //图形字体索引
    static int iconFontIndex;

public:
    //样式颜色结构体
    struct StyleColor {
        QString position;           //位置 left right top bottom
        bool defaultBorder;         //默认有边框

        quint32 btnWidth;           //按钮宽度
        quint32 btnHeight;          //按钮高度

        quint32 iconSize;           //图标字体尺寸
        quint32 iconWidth;          //图标图片宽度
        quint32 iconHeight;         //图标图片高度

        quint32 borderWidth;        //边框宽度
        QString borderColor;        //边框颜色

        QString normalBgColor;      //正常背景颜色
        QString normalTextColor;    //正常文字颜色
        QString hoverBgColor;       //悬停背景颜色
        QString hoverTextColor;     //悬停文字颜色
        QString pressedBgColor;     //按下背景颜色
        QString pressedTextColor;   //按下文字颜色
        QString checkedBgColor;     //选中背景颜色
        QString checkedTextColor;   //选中文字颜色

        StyleColor() {
            position = "left";
            defaultBorder = false;

            btnWidth = 0;
            btnHeight = 0;

            iconSize = 12;
            iconWidth = 15;
            iconHeight = 15;

            borderWidth = 3;
            borderColor = "#029FEA";

            normalBgColor = "#292F38";
            normalTextColor = "#54626F";
            hoverBgColor = "#40444D";
            hoverTextColor = "#FDFDFD";
            pressedBgColor = "#404244";
            pressedTextColor = "#FDFDFD";
            checkedBgColor = "#44494F";
            checkedTextColor = "#FDFDFD";
        }

        //设置常规颜色 普通状态+加深状态
        void setColor(const QString &normalBgColor,
                      const QString &normalTextColor,
                      const QString &darkBgColor,
                      const QString &darkTextColor) {
            this->normalBgColor = normalBgColor;
            this->normalTextColor = normalTextColor;
            this->hoverBgColor = darkBgColor;
            this->hoverTextColor = darkTextColor;
            this->pressedBgColor = darkBgColor;
            this->pressedTextColor = darkTextColor;
            this->checkedBgColor = darkBgColor;
            this->checkedTextColor = darkTextColor;
        }
    };


    //初始化图形字体
    static void initFont();
    //设置引用图形字体文件索引
    static void setIconFontIndex(int index);

    //获取图形字体
    static QFont getIconFontAliBaBa();
    static QFont getIconFontAwesome();
    static QFont getIconFontAwesome6();
    static QFont getIconFontWeather();

    //根据值获取图形字体类
    static IconHelper *getIconHelper(int icon);

    //设置图形字体到标签
    static void setIcon(QLabel *lab, int icon, quint32 size = 12);
    //设置图形字体到按钮
    static void setIcon(QAbstractButton *btn, int icon, quint32 size = 12);

    //设置图形字体到图标
    static void setPixmap(QAbstractButton *btn, const QColor &color,
                          int icon, quint32 size = 12,
                          quint32 width = 15, quint32 height = 15,
                          int flags = Qt::AlignCenter);
    //获取指定图形字体,可以指定文字大小,图片宽高,文字对齐
    static QPixmap getPixmap(const QColor &color, int icon, quint32 size = 12,
                             quint32 width = 15, quint32 height = 15,
                             int flags = Qt::AlignCenter);

    //指定导航面板样式,带图标和效果切换+悬停颜色+按下颜色+选中颜色
    static void setStyle(QWidget *widget, QList<QPushButton *> btns, QList<int> icons, const StyleColor &styleColor);
    static void setStyle(QWidget *widget, QList<QToolButton *> btns, QList<int> icons, const StyleColor &styleColor);
    static void setStyle(QWidget *widget, QList<QAbstractButton *> btns, QList<int> icons, const StyleColor &styleColor);

    //默认构造函数,传入字体文件+字体名称
    explicit IconHelper(const QString &fontFile, const QString &fontName, QObject *parent = 0);

protected:
    bool eventFilter(QObject *watched, QEvent *event);

private:
    QFont iconFont;                 //图形字体
    QList<QAbstractButton *> btns;  //按钮队列
    QList<QPixmap> pixNormal;       //正常图片队列
    QList<QPixmap> pixHover;        //悬停图片队列
    QList<QPixmap> pixPressed;      //按下图片队列
    QList<QPixmap> pixChecked;      //选中图片队列

private slots:
    //按钮选中状态切换处理
    void toggled(bool checked);

public:
    //获取图形字体
    QFont getIconFont();

    //设置图形字体到标签
    void setIcon1(QLabel *lab, int icon, quint32 size = 12);
    //设置图形字体到按钮
    void setIcon1(QAbstractButton *btn, int icon, quint32 size = 12);

    //设置图形字体到图标
    void setPixmap1(QAbstractButton *btn, const QColor &color,
                    int icon, quint32 size = 12,
                    quint32 width = 15, quint32 height = 15,
                    int flags = Qt::AlignCenter);
    //获取指定图形字体,可以指定文字大小,图片宽高,文字对齐
    QPixmap getPixmap1(const QColor &color, int icon, quint32 size = 12,
                       quint32 width = 15, quint32 height = 15,
                       int flags = Qt::AlignCenter);

    //指定导航面板样式,带图标和效果切换+悬停颜色+按下颜色+选中颜色
    void setStyle1(QWidget *widget, QList<QPushButton *> btns, QList<int> icons, const StyleColor &styleColor);
    void setStyle1(QWidget *widget, QList<QToolButton *> btns, QList<int> icons, const StyleColor &styleColor);
    void setStyle1(QWidget *widget, QList<QAbstractButton *> btns, QList<int> icons, const StyleColor &styleColor);
};

#endif // ICONHELPER_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmnavbutton.h"
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

    frmNavButton w;
    w.setWindowTitle("导航按钮控件 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./navbutton.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "navbutton.h"
#include "qpainter.h"
#include "qdebug.h"

NavButton::NavButton(QWidget *parent) : QPushButton(parent)
{
    paddingLeft = 20;
    paddingRight = 5;
    paddingTop = 5;
    paddingBottom = 5;
    textAlign = TextAlign_Left;

    showTriangle = false;
    triangleLen = 5;
    trianglePosition = TrianglePosition_Right;
    triangleColor = QColor(255, 255, 255);

    showIcon = false;
    iconSpace = 10;
    iconSize = QSize(16, 16);
    iconNormal = QPixmap();
    iconHover = QPixmap();
    iconCheck = QPixmap();

    showLine = true;
    lineSpace = 0;
    lineWidth = 5;
    linePosition = LinePosition_Left;
    lineColor = QColor(0, 187, 158);

    normalBgColor = QColor(230, 230, 230);
    hoverBgColor = QColor(130, 130, 130);
    checkBgColor = QColor(80, 80, 80);
    normalTextColor = QColor(100, 100, 100);
    hoverTextColor = QColor(255, 255, 255);
    checkTextColor = QColor(255, 255, 255);

    normalBgBrush = Qt::NoBrush;
    hoverBgBrush = Qt::NoBrush;
    checkBgBrush = Qt::NoBrush;

    hover = false;
    setCheckable(true);
    setText("导航按钮");
}

void NavButton::enterEvent(EnterEvent *)
{
    hover = true;
    this->update();
}

void NavButton::leaveEvent(QEvent *)
{
    hover = false;
    this->update();
}

void NavButton::paintEvent(QPaintEvent *)
{
    //绘制准备工作,启用反锯齿
    QPainter painter(this);
    painter.setRenderHints(QPainter::Antialiasing | QPainter::TextAntialiasing);

    //绘制背景
    drawBg(&painter);
    //绘制文字
    drawText(&painter);
    //绘制图标
    drawIcon(&painter);
    //绘制边框线条
    drawLine(&painter);
    //绘制倒三角
    drawTriangle(&painter);
}

void NavButton::drawBg(QPainter *painter)
{
    painter->save();
    painter->setPen(Qt::NoPen);

    int width = this->width();
    int height = this->height();

    QRect bgRect;
    if (linePosition == LinePosition_Left) {
        bgRect = QRect(lineSpace, 0, width - lineSpace, height);
    } else if (linePosition == LinePosition_Right) {
        bgRect = QRect(0, 0, width - lineSpace, height);
    } else if (linePosition == LinePosition_Top) {
        bgRect = QRect(0, lineSpace, width, height - lineSpace);
    } else if (linePosition == LinePosition_Bottom) {
        bgRect = QRect(0, 0, width, height - lineSpace);
    }

    //如果画刷存在则取画刷
    QBrush bgBrush;
    if (isChecked()) {
        bgBrush = checkBgBrush;
    } else if (hover) {
        bgBrush = hoverBgBrush;
    } else {
        bgBrush = normalBgBrush;
    }

    if (bgBrush != Qt::NoBrush) {
        painter->setBrush(bgBrush);
    } else {
        //根据当前状态选择对应颜色
        QColor bgColor;
        if (isChecked()) {
            bgColor = checkBgColor;
        } else if (hover) {
            bgColor = hoverBgColor;
        } else {
            bgColor = normalBgColor;
        }

        painter->setBrush(bgColor);
    }

    painter->drawRect(bgRect);

    painter->restore();
}

void NavButton::drawText(QPainter *painter)
{
    painter->save();
    painter->setBrush(Qt::NoBrush);

    //根据当前状态选择对应颜色
    QColor textColor;
    if (isChecked()) {
        textColor = checkTextColor;
    } else if (hover) {
        textColor = hoverTextColor;
    } else {
        textColor = normalTextColor;
    }

    QRect textRect = QRect(paddingLeft, paddingTop, width() - paddingLeft - paddingRight, height() - paddingTop - paddingBottom);
    painter->setPen(textColor);
    painter->drawText(textRect, textAlign | Qt::AlignVCenter, text());

    painter->restore();
}

void NavButton::drawIcon(QPainter *painter)
{
    if (!showIcon) {
        return;
    }

    painter->save();

    QPixmap pix;
    if (isChecked()) {
        pix = iconCheck;
    } else if (hover) {
        pix = iconHover;
    } else {
        pix = iconNormal;
    }

    if (!pix.isNull()) {
        //等比例平滑缩放图标
        pix = pix.scaled(iconSize, Qt::KeepAspectRatio, Qt::SmoothTransformation);
        painter->drawPixmap(iconSpace, (height() - iconSize.height()) / 2, pix);
    }

    painter->restore();
}

void NavButton::drawLine(QPainter *painter)
{
    if (!showLine) {
        return;
    }

    if (!isChecked()) {
        return;
    }

    painter->save();

    QPen pen;
    pen.setWidth(lineWidth);
    pen.setColor(lineColor);
    painter->setPen(pen);

    //根据线条位置设置线条坐标
    QPoint pointStart, pointEnd;
    if (linePosition == LinePosition_Left) {
        pointStart = QPoint(0, 0);
        pointEnd = QPoint(0, height());
    } else if (linePosition == LinePosition_Right) {
        pointStart = QPoint(width(), 0);
        pointEnd = QPoint(width(), height());
    } else if (linePosition == LinePosition_Top) {
        pointStart = QPoint(0, 0);
        pointEnd = QPoint(width(), 0);
    } else if (linePosition == LinePosition_Bottom) {
        pointStart = QPoint(0, height());
        pointEnd = QPoint(width(), height());
    }

    painter->drawLine(pointStart, pointEnd);

    painter->restore();
}

void NavButton::drawTriangle(QPainter *painter)
{
    if (!showTriangle) {
        return;
    }

    //选中或者悬停显示
    if (!hover && !isChecked()) {
        return;
    }

    painter->save();
    painter->setPen(Qt::NoPen);
    painter->setBrush(triangleColor);

    //绘制在右侧中间,根据设定的倒三角的边长设定三个点位置
    int width = this->width();
    int height = this->height();
    int midWidth = width / 2;
    int midHeight = height / 2;

    QPolygon pts;
    if (trianglePosition == TrianglePosition_Left) {
        pts.setPoints(3, triangleLen, midHeight, 0, midHeight - triangleLen, 0, midHeight + triangleLen);
    } else if (trianglePosition == TrianglePosition_Right) {
        pts.setPoints(3, width - triangleLen, midHeight, width, midHeight - triangleLen, width, midHeight + triangleLen);
    } else if (trianglePosition == TrianglePosition_Top) {
        pts.setPoints(3, midWidth, triangleLen, midWidth - triangleLen, 0, midWidth + triangleLen, 0);
    } else if (trianglePosition == TrianglePosition_Bottom) {
        pts.setPoints(3, midWidth, height - triangleLen, midWidth - triangleLen, height, midWidth + triangleLen, height);
    }

    painter->drawPolygon(pts);

    painter->restore();
}

QSize NavButton::sizeHint() const
{
    return QSize(100, 30);
}

QSize NavButton::minimumSizeHint() const
{
    return QSize(20, 10);
}

int NavButton::getPaddingLeft() const
{
    return this->paddingLeft;
}

void NavButton::setPaddingLeft(int paddingLeft)
{
    if (this->paddingLeft != paddingLeft) {
        this->paddingLeft = paddingLeft;
        this->update();
    }
}

int NavButton::getPaddingRight() const
{
    return this->paddingRight;
}

void NavButton::setPaddingRight(int paddingRight)
{
    if (this->paddingRight != paddingRight) {
        this->paddingRight = paddingRight;
        this->update();
    }
}

int NavButton::getPaddingTop() const
{
    return this->paddingTop;
}

void NavButton::setPaddingTop(int paddingTop)
{
    if (this->paddingTop != paddingTop) {
        this->paddingTop = paddingTop;
        this->update();
    }
}

int NavButton::getPaddingBottom() const
{
    return this->paddingBottom;
}

void NavButton::setPaddingBottom(int paddingBottom)
{
    if (this->paddingBottom != paddingBottom) {
        this->paddingBottom = paddingBottom;
        this->update();
    }
}

void NavButton::setPadding(int padding)
{
    setPadding(padding, padding, padding, padding);
}

void NavButton::setPadding(int paddingLeft, int paddingRight, int paddingTop, int paddingBottom)
{
    this->paddingLeft = paddingLeft;
    this->paddingRight = paddingRight;
    this->paddingTop = paddingTop;
    this->paddingBottom = paddingBottom;
    this->update();
}

NavButton::TextAlign NavButton::getTextAlign() const
{
    return this->textAlign;
}

void NavButton::setTextAlign(const NavButton::TextAlign &textAlign)
{
    if (this->textAlign != textAlign) {
        this->textAlign = textAlign;
        this->update();
    }
}

bool NavButton::getShowTriangle() const
{
    return this->showTriangle;
}

void NavButton::setShowTriangle(bool showTriangle)
{
    if (this->showTriangle != showTriangle) {
        this->showTriangle = showTriangle;
        this->update();
    }
}

int NavButton::getTriangleLen() const
{
    return this->triangleLen;
}

void NavButton::setTriangleLen(int triangleLen)
{
    if (this->triangleLen != triangleLen) {
        this->triangleLen = triangleLen;
        this->update();
    }
}

NavButton::TrianglePosition NavButton::getTrianglePosition() const
{
    return this->trianglePosition;
}

void NavButton::setTrianglePosition(const NavButton::TrianglePosition &trianglePosition)
{
    if (this->trianglePosition != trianglePosition) {
        this->trianglePosition = trianglePosition;
        this->update();
    }
}

QColor NavButton::getTriangleColor() const
{
    return this->triangleColor;
}

void NavButton::setTriangleColor(const QColor &triangleColor)
{
    if (this->triangleColor != triangleColor) {
        this->triangleColor = triangleColor;
        this->update();
    }
}

bool NavButton::getShowIcon() const
{
    return this->showIcon;
}

void NavButton::setShowIcon(bool showIcon)
{
    if (this->showIcon != showIcon) {
        this->showIcon = showIcon;
        this->update();
    }
}

int NavButton::getIconSpace() const
{
    return this->iconSpace;
}

void NavButton::setIconSpace(int iconSpace)
{
    if (this->iconSpace != iconSpace) {
        this->iconSpace = iconSpace;
        this->update();
    }
}

QSize NavButton::getIconSize() const
{
    return this->iconSize;
}

void NavButton::setIconSize(const QSize &iconSize)
{
    if (this->iconSize != iconSize) {
        this->iconSize = iconSize;
        this->update();
    }
}

QPixmap NavButton::getIconNormal() const
{
    return this->iconNormal;
}

void NavButton::setIconNormal(const QPixmap &iconNormal)
{
    this->iconNormal = iconNormal;
    this->update();
}

QPixmap NavButton::getIconHover() const
{
    return this->iconHover;
}

void NavButton::setIconHover(const QPixmap &iconHover)
{
    this->iconHover = iconHover;
    this->update();
}

QPixmap NavButton::getIconCheck() const
{
    return this->iconCheck;
}

void NavButton::setIconCheck(const QPixmap &iconCheck)
{
    this->iconCheck = iconCheck;
    this->update();
}

bool NavButton::getShowLine() const
{
    return this->showLine;
}

void NavButton::setShowLine(bool showLine)
{
    if (this->showLine != showLine) {
        this->showLine = showLine;
        this->update();
    }
}

int NavButton::getLineSpace() const
{
    return this->lineSpace;
}

void NavButton::setLineSpace(int lineSpace)
{
    if (this->lineSpace != lineSpace) {
        this->lineSpace = lineSpace;
        this->update();
    }
}

int NavButton::getLineWidth() const
{
    return this->lineWidth;
}

void NavButton::setLineWidth(int lineWidth)
{
    if (this->lineWidth != lineWidth) {
        this->lineWidth = lineWidth;
        this->update();
    }
}

NavButton::LinePosition NavButton::getLinePosition() const
{
    return this->linePosition;
}

void NavButton::setLinePosition(const NavButton::LinePosition &linePosition)
{
    if (this->linePosition != linePosition) {
        this->linePosition = linePosition;
        this->update();
    }
}

QColor NavButton::getLineColor() const
{
    return this->lineColor;
}

void NavButton::setLineColor(const QColor &lineColor)
{
    if (this->lineColor != lineColor) {
        this->lineColor = lineColor;
        this->update();
    }
}

QColor NavButton::getNormalBgColor() const
{
    return this->normalBgColor;
}

void NavButton::setNormalBgColor(const QColor &normalBgColor)
{
    if (this->normalBgColor != normalBgColor) {
        this->normalBgColor = normalBgColor;
        this->update();
    }
}

QColor NavButton::getHoverBgColor() const
{
    return this->hoverBgColor;
}

void NavButton::setHoverBgColor(const QColor &hoverBgColor)
{
    if (this->hoverBgColor != hoverBgColor) {
        this->hoverBgColor = hoverBgColor;
        this->update();
    }
}

QColor NavButton::getCheckBgColor() const
{
    return this->checkBgColor;
}

void NavButton::setCheckBgColor(const QColor &checkBgColor)
{
    if (this->checkBgColor != checkBgColor) {
        this->checkBgColor = checkBgColor;
        this->update();
    }
}

QColor NavButton::getNormalTextColor() const
{
    return this->normalTextColor;
}

void NavButton::setNormalTextColor(const QColor &normalTextColor)
{
    if (this->normalTextColor != normalTextColor) {
        this->normalTextColor = normalTextColor;
        this->update();
    }
}


QColor NavButton::getHoverTextColor() const
{
    return this->hoverTextColor;
}

void NavButton::setHoverTextColor(const QColor &hoverTextColor)
{
    if (this->hoverTextColor != hoverTextColor) {
        this->hoverTextColor = hoverTextColor;
        this->update();
    }
}

QColor NavButton::getCheckTextColor() const
{
    return this->checkTextColor;
}

void NavButton::setCheckTextColor(const QColor &checkTextColor)
{
    if (this->checkTextColor != checkTextColor) {
        this->checkTextColor = checkTextColor;
        this->update();
    }
}

void NavButton::setNormalBgBrush(const QBrush &normalBgBrush)
{
    if (this->normalBgBrush != normalBgBrush) {
        this->normalBgBrush = normalBgBrush;
        this->update();
    }
}

void NavButton::setHoverBgBrush(const QBrush &hoverBgBrush)
{
    if (this->hoverBgBrush != hoverBgBrush) {
        this->hoverBgBrush = hoverBgBrush;
        this->update();
    }
}

void NavButton::setCheckBgBrush(const QBrush &checkBgBrush)
{
    if (this->checkBgBrush != checkBgBrush) {
        this->checkBgBrush = checkBgBrush;
        this->update();
    }
}
```

### `./navbutton.h`

```cpp
﻿#ifndef NAVBUTTON_H
#define NAVBUTTON_H

/**
 * 导航按钮控件 作者:feiyangqingyun(QQ:517216493) 2017-12-19
 * 1. 可设置文字的左侧、右侧、顶部、底部间隔。
 * 2. 可设置文字对齐方式。
 * 3. 可设置显示倒三角、倒三角边长、倒三角位置、倒三角颜色。
 * 4. 可设置显示图标、图标间隔、图标尺寸、正常状态图标、悬停状态图标、选中状态图标。
 * 5. 可设置显示边框线条、线条宽度、线条间隔、线条位置、线条颜色。
 * 6. 可设置正常背景颜色、悬停背景颜色、选中背景颜色。
 * 7. 可设置正常文字颜色、悬停文字颜色、选中文字颜色。
 * 8. 可设置背景颜色为画刷颜色。
 */

#include <QPushButton>

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
#define EnterEvent QEnterEvent
#else
#define EnterEvent QEvent
#endif

#ifdef quc
class Q_DECL_EXPORT NavButton : public QPushButton
#else
class NavButton : public QPushButton
#endif

{
    Q_OBJECT
    Q_ENUMS(TextAlign)
    Q_ENUMS(TrianglePosition)
    Q_ENUMS(LinePosition)
    Q_ENUMS(IconPosition)

    Q_PROPERTY(int paddingLeft READ getPaddingLeft WRITE setPaddingLeft)
    Q_PROPERTY(int paddingRight READ getPaddingRight WRITE setPaddingRight)
    Q_PROPERTY(int paddingTop READ getPaddingTop WRITE setPaddingTop)
    Q_PROPERTY(int paddingBottom READ getPaddingBottom WRITE setPaddingBottom)
    Q_PROPERTY(TextAlign textAlign READ getTextAlign WRITE setTextAlign)

    Q_PROPERTY(bool showTriangle READ getShowTriangle WRITE setShowTriangle)
    Q_PROPERTY(int triangleLen READ getTriangleLen WRITE setTriangleLen)
    Q_PROPERTY(TrianglePosition trianglePosition READ getTrianglePosition WRITE setTrianglePosition)
    Q_PROPERTY(QColor triangleColor READ getTriangleColor WRITE setTriangleColor)

    Q_PROPERTY(bool showIcon READ getShowIcon WRITE setShowIcon)
    Q_PROPERTY(int iconSpace READ getIconSpace WRITE setIconSpace)
    Q_PROPERTY(QSize iconSize READ getIconSize WRITE setIconSize)
    Q_PROPERTY(QPixmap iconNormal READ getIconNormal WRITE setIconNormal)
    Q_PROPERTY(QPixmap iconHover READ getIconHover WRITE setIconHover)
    Q_PROPERTY(QPixmap iconCheck READ getIconCheck WRITE setIconCheck)

    Q_PROPERTY(bool showLine READ getShowLine WRITE setShowLine)
    Q_PROPERTY(int lineSpace READ getLineSpace WRITE setLineSpace)
    Q_PROPERTY(int lineWidth READ getLineWidth WRITE setLineWidth)
    Q_PROPERTY(LinePosition linePosition READ getLinePosition WRITE setLinePosition)
    Q_PROPERTY(QColor lineColor READ getLineColor WRITE setLineColor)

    Q_PROPERTY(QColor normalBgColor READ getNormalBgColor WRITE setNormalBgColor)
    Q_PROPERTY(QColor hoverBgColor READ getHoverBgColor WRITE setHoverBgColor)
    Q_PROPERTY(QColor checkBgColor READ getCheckBgColor WRITE setCheckBgColor)
    Q_PROPERTY(QColor normalTextColor READ getNormalTextColor WRITE setNormalTextColor)
    Q_PROPERTY(QColor hoverTextColor READ getHoverTextColor WRITE setHoverTextColor)
    Q_PROPERTY(QColor checkTextColor READ getCheckTextColor WRITE setCheckTextColor)

public:
    enum TextAlign {
        TextAlign_Left = 0x0001,    //左侧对齐
        TextAlign_Right = 0x0002,   //右侧对齐
        TextAlign_Top = 0x0020,     //顶部对齐
        TextAlign_Bottom = 0x0040,  //底部对齐
        TextAlign_Center = 0x0004   //居中对齐
    };

    enum TrianglePosition {
        TrianglePosition_Left = 0,  //左侧
        TrianglePosition_Right = 1, //右侧
        TrianglePosition_Top = 2,   //顶部
        TrianglePosition_Bottom = 3 //底部
    };

    enum IconPosition {
        IconPosition_Left = 0,      //左侧
        IconPosition_Right = 1,     //右侧
        IconPosition_Top = 2,       //顶部
        IconPosition_Bottom = 3     //底部
    };

    enum LinePosition {
        LinePosition_Left = 0,      //左侧
        LinePosition_Right = 1,     //右侧
        LinePosition_Top = 2,       //顶部
        LinePosition_Bottom = 3     //底部
    };

    explicit NavButton(QWidget *parent = 0);

protected:
    void enterEvent(EnterEvent *);
    void leaveEvent(QEvent *);
    void paintEvent(QPaintEvent *);
    void drawBg(QPainter *painter);
    void drawText(QPainter *painter);
    void drawIcon(QPainter *painter);
    void drawLine(QPainter *painter);
    void drawTriangle(QPainter *painter);

private:
    int paddingLeft;            //文字左侧间隔
    int paddingRight;           //文字右侧间隔
    int paddingTop;             //文字顶部间隔
    int paddingBottom;          //文字底部间隔
    TextAlign textAlign;        //文字对齐

    bool showTriangle;          //显示倒三角
    int triangleLen;            //倒三角边长
    TrianglePosition trianglePosition;//倒三角位置
    QColor triangleColor;       //倒三角颜色

    bool showIcon;              //显示图标
    int iconSpace;              //图标间隔
    QSize iconSize;             //图标尺寸
    QPixmap iconNormal;         //正常图标
    QPixmap iconHover;          //悬停图标
    QPixmap iconCheck;          //选中图标

    bool showLine;              //显示线条
    int lineSpace;              //线条间隔
    int lineWidth;              //线条宽度
    LinePosition linePosition;  //线条位置
    QColor lineColor;           //线条颜色

    QColor normalBgColor;       //正常背景颜色
    QColor hoverBgColor;        //悬停背景颜色
    QColor checkBgColor;        //选中背景颜色
    QColor normalTextColor;     //正常文字颜色
    QColor hoverTextColor;      //悬停文字颜色
    QColor checkTextColor;      //选中文字颜色

    QBrush normalBgBrush;       //正常背景画刷
    QBrush hoverBgBrush;        //悬停背景画刷
    QBrush checkBgBrush;        //选中背景画刷

    bool hover;                 //悬停标志位

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //获取和设置文字左侧间隔
    int getPaddingLeft() const;
    void setPaddingLeft(int paddingLeft);

    //获取和设置文字左侧间隔
    int getPaddingRight() const;
    void setPaddingRight(int paddingRight);

    //获取和设置文字顶部间隔
    int getPaddingTop() const;
    void setPaddingTop(int paddingTop);

    //获取和设置文字底部间隔
    int getPaddingBottom() const;
    void setPaddingBottom(int paddingBottom);

    //设置边距
    void setPadding(int padding);
    void setPadding(int paddingLeft, int paddingRight, int paddingTop, int paddingBottom);

    //获取和设置文字对齐
    TextAlign getTextAlign() const;
    void setTextAlign(const TextAlign &textAlign);

    //获取和设置显示倒三角
    bool getShowTriangle() const;
    void setShowTriangle(bool showTriangle);

    //获取和设置倒三角边长
    int getTriangleLen() const;
    void setTriangleLen(int triangleLen);

    //获取和设置倒三角位置
    TrianglePosition getTrianglePosition() const;
    void setTrianglePosition(const TrianglePosition &trianglePosition);

    //获取和设置倒三角颜色
    QColor getTriangleColor() const;
    void setTriangleColor(const QColor &triangleColor);

    //获取和设置显示图标
    bool getShowIcon() const;
    void setShowIcon(bool showIcon);

    //获取和设置图标间隔
    int getIconSpace() const;
    void setIconSpace(int iconSpace);

    //获取和设置图标尺寸
    QSize getIconSize() const;
    void setIconSize(const QSize &iconSize);

    //获取和设置正常图标
    QPixmap getIconNormal() const;
    void setIconNormal(const QPixmap &iconNormal);

    //获取和设置悬停图标
    QPixmap getIconHover() const;
    void setIconHover(const QPixmap &iconHover);

    //获取和设置按下图标
    QPixmap getIconCheck() const;
    void setIconCheck(const QPixmap &iconCheck);

    //获取和设置显示线条
    bool getShowLine() const;
    void setShowLine(bool showLine);

    //获取和设置线条间隔
    int getLineSpace() const;
    void setLineSpace(int lineSpace);

    //获取和设置线条宽度
    int getLineWidth() const;
    void setLineWidth(int lineWidth);

    //获取和设置线条位置
    LinePosition getLinePosition() const;
    void setLinePosition(const LinePosition &linePosition);

    //获取和设置线条颜色
    QColor getLineColor() const;
    void setLineColor(const QColor &lineColor);

    //获取和设置正常背景颜色
    QColor getNormalBgColor() const;
    void setNormalBgColor(const QColor &normalBgColor);

    //获取和设置悬停背景颜色
    QColor getHoverBgColor() const;
    void setHoverBgColor(const QColor &hoverBgColor);

    //获取和设置选中背景颜色
    QColor getCheckBgColor() const;
    void setCheckBgColor(const QColor &checkBgColor);

    //获取和设置正常文字颜色
    QColor getNormalTextColor() const;
    void setNormalTextColor(const QColor &normalTextColor);

    //获取和设置悬停文字颜色
    QColor getHoverTextColor() const;
    void setHoverTextColor(const QColor &hoverTextColor);

    //获取和设置选中文字颜色
    QColor getCheckTextColor() const;
    void setCheckTextColor(const QColor &checkTextColor);

    //设置正常背景画刷
    void setNormalBgBrush(const QBrush &normalBgBrush);
    //设置悬停背景画刷
    void setHoverBgBrush(const QBrush &hoverBgBrush);
    //设置选中背景画刷
    void setCheckBgBrush(const QBrush &checkBgBrush);
};

#endif // NAVBUTTON_H
```

### `./frmnavbutton.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmNavButton</class>
 <widget class="QWidget" name="frmNavButton">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>605</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <widget class="QWidget" name="widgetNav7" native="true">
   <property name="geometry">
    <rect>
     <x>11</x>
     <y>245</y>
     <width>511</width>
     <height>40</height>
    </rect>
   </property>
   <property name="minimumSize">
    <size>
     <width>0</width>
     <height>40</height>
    </size>
   </property>
   <property name="maximumSize">
    <size>
     <width>16777215</width>
     <height>40</height>
    </size>
   </property>
   <layout class="QHBoxLayout" name="horizontalLayout_2">
    <property name="spacing">
     <number>0</number>
    </property>
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
    <item>
     <widget class="NavButton" name="navButton71">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>首页</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton72">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>论坛</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton73">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>Qt下载</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton74">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>作品展</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton75">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>群组</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton76">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>个人中心</string>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
  <widget class="QWidget" name="widgetNav1" native="true">
   <property name="geometry">
    <rect>
     <x>11</x>
     <y>11</y>
     <width>120</width>
     <height>133</height>
    </rect>
   </property>
   <layout class="QVBoxLayout" name="verticalLayout">
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
    <item>
     <widget class="NavButton" name="navButton11">
      <property name="text">
       <string>学生管理</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton12">
      <property name="text">
       <string>教师管理</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton13">
      <property name="text">
       <string>成绩管理</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton14">
      <property name="text">
       <string>记录查询</string>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
  <widget class="QWidget" name="widgetNav2" native="true">
   <property name="geometry">
    <rect>
     <x>140</x>
     <y>11</y>
     <width>120</width>
     <height>133</height>
    </rect>
   </property>
   <layout class="QVBoxLayout" name="verticalLayout_2">
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
    <item>
     <widget class="NavButton" name="navButton21">
      <property name="text">
       <string>访客登记</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton22">
      <property name="text">
       <string>记录查询</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton23">
      <property name="text">
       <string>系统设置</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton24">
      <property name="text">
       <string>系统重启</string>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
  <widget class="QWidget" name="widgetNav5" native="true">
   <property name="geometry">
    <rect>
     <x>11</x>
     <y>151</y>
     <width>511</width>
     <height>40</height>
    </rect>
   </property>
   <property name="minimumSize">
    <size>
     <width>0</width>
     <height>40</height>
    </size>
   </property>
   <property name="maximumSize">
    <size>
     <width>16777215</width>
     <height>40</height>
    </size>
   </property>
   <layout class="QHBoxLayout" name="horizontalLayout">
    <property name="spacing">
     <number>0</number>
    </property>
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
    <item>
     <widget class="NavButton" name="navButton51">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>首页</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton52">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>论坛</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton53">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>作品</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton54">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>群组</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton55">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>帮助</string>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
  <widget class="QWidget" name="widgetNav6" native="true">
   <property name="geometry">
    <rect>
     <x>11</x>
     <y>198</y>
     <width>511</width>
     <height>40</height>
    </rect>
   </property>
   <property name="minimumSize">
    <size>
     <width>0</width>
     <height>40</height>
    </size>
   </property>
   <property name="maximumSize">
    <size>
     <width>16777215</width>
     <height>40</height>
    </size>
   </property>
   <layout class="QHBoxLayout" name="horizontalLayout_3">
    <property name="spacing">
     <number>0</number>
    </property>
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
    <item>
     <widget class="NavButton" name="navButton61">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>首页</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton62">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>论坛</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton63">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>作品</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton64">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>群组</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton65">
      <property name="sizePolicy">
       <sizepolicy hsizetype="Minimum" vsizetype="Expanding">
        <horstretch>0</horstretch>
        <verstretch>0</verstretch>
       </sizepolicy>
      </property>
      <property name="text">
       <string>帮助</string>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
  <widget class="QWidget" name="widgetNav3" native="true">
   <property name="geometry">
    <rect>
     <x>270</x>
     <y>11</y>
     <width>120</width>
     <height>133</height>
    </rect>
   </property>
   <layout class="QVBoxLayout" name="verticalLayout_3">
    <property name="spacing">
     <number>6</number>
    </property>
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
    <item>
     <widget class="NavButton" name="navButton31">
      <property name="text">
       <string>学生管理</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton32">
      <property name="text">
       <string>教师管理</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton33">
      <property name="text">
       <string>成绩管理</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton34">
      <property name="text">
       <string>记录查询</string>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
  <widget class="QWidget" name="widgetNav4" native="true">
   <property name="geometry">
    <rect>
     <x>400</x>
     <y>11</y>
     <width>120</width>
     <height>133</height>
    </rect>
   </property>
   <layout class="QVBoxLayout" name="verticalLayout_4">
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
    <item>
     <widget class="NavButton" name="navButton41">
      <property name="text">
       <string>学生管理</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton42">
      <property name="text">
       <string>教师管理</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton43">
      <property name="text">
       <string>成绩管理</string>
      </property>
     </widget>
    </item>
    <item>
     <widget class="NavButton" name="navButton44">
      <property name="text">
       <string>记录查询</string>
      </property>
     </widget>
    </item>
   </layout>
  </widget>
 </widget>
 <customwidgets>
  <customwidget>
   <class>NavButton</class>
   <extends>QPushButton</extends>
   <header>navbutton.h</header>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
