---
tags:
  - Qt
  - 第三方类
  - 高质量
---
# 精美图表qcustomplot示例 (`qcustomplotdemo`)

> 精美图表qcustomplot示例

## 效果图

![[QWidgetDemo_assert/qcustomplotdemo1.jpg]]
![[QWidgetDemo_assert/qcustomplotdemo2.jpg]]
![[QWidgetDemo_assert/qcustomplotdemo3.jpg]]
![[QWidgetDemo_assert/qcustomplotdemo4.jpg]]
![[QWidgetDemo_assert/qcustomplotdemo5.jpg]]
![[QWidgetDemo_assert/qcustomplotdemo6.jpg]]
![[QWidgetDemo_assert/qcustomplotdemo7.jpg]]

## 分类

- 类型: 第三方类 `third`
- 源码目录: `third/qcustomplotdemo`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QWidget QColor QCPAxis QPen QCP QString QCustomPlot QFont QAbstractButton QCPScatterStyle QList QVector QCPRange QCPGraph QVBoxLayout QBrush QPixmap QCPBars QMouseEvent QCPItemText

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
#include <QApplication>
#include <QTextCodec>
#include <QFontDatabase>
#include <QTime>
#include <QDebug>

int main(int argc, char *argv[])
{
#if (QT_VERSION >= QT_VERSION_CHECK(5,14,0))
    QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
    QApplication::setHighDpiScaleFactorRoundingPolicy(Qt::HighDpiScaleFactorRoundingPolicy::PassThrough);
#endif

    QApplication a(argc, argv);
    QFont font;
#ifdef Q_OS_WASM
    QString fontFile = ":/font/DroidSansFallback.ttf";
    QString fontName = "Droid Sans Fallback";

    //判断图形字体是否存在,不存在则加入
    QFontDatabase fontDb;
    if (!fontDb.families().contains(fontName)) {
        int fontId = fontDb.addApplicationFont(fontFile);
        QStringList listName = fontDb.applicationFontFamilies(fontId);
        if (listName.count() == 0) {
            qDebug() << QString("load %1 error").arg(fontName);
        }
    }

    //再次判断是否包含字体名称防止加载失败
    if (fontDb.families().contains(fontName)) {
        font = QFont(fontName);
#if (QT_VERSION >= QT_VERSION_CHECK(4,8,0))
        font.setHintingPreference(QFont::PreferNoHinting);
#endif
    }
#else
    font.setFamily("Microsoft Yahei");
    font.setPixelSize(13);
#endif
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

    //初始化随机数种子
    QTime t = QTime::currentTime();
    srand(t.msec() + t.second() * 1000);

    frmMain w;
    w.setWindowTitle("qcustomplot示例 (QQ: 517216493 WX: feiyangqingyun)");
    w.resize(1000, 650);
    w.show();

    return a.exec();
}
```

### `frmcustom/frmmain.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
#include "ui_frmmain.h"
#include "iconhelper.h"
#include "qdebug.h"

#include "frmquadratic.h"
#include "frmsimple.h"
#include "frmsincscatter.h"
#include "frmscatterstyle.h"
#include "frmscatterpixmap.h"
#include "frmlinestyle.h"
#include "frmdate.h"
#include "frmtexturebrush.h"
#include "frmmultiaxis.h"
#include "frmlogarithmic.h"
#include "frmrealtimedata.h"
#include "frmparametriccurve.h"
#include "frmbarchart.h"
#include "frmstatistical.h"
#include "frmsimpleitem.h"
#include "frmitem.h"
#include "frmstyled.h"
#include "frmadvancedaxes.h"
#include "frmcolormap.h"
#include "frmfinancial.h"
#include "frmpolarplot.h"

#include "frmaxistag.h"
#include "frminteraction.h"
#include "frmscrollbar.h"

frmMain::frmMain(QWidget *parent) : QWidget(parent), ui(new Ui::frmMain)
{
    ui->setupUi(this);
    this->initForm();
    this->initWidget();
    this->initNav();
    this->initIcon();
}

frmMain::~frmMain()
{
    delete ui;
}

void frmMain::showEvent(QShowEvent *)
{
    //滚动到底部
    QScrollBar *bar = ui->scrollArea->verticalScrollBar();
    bar->setValue(bar->maximum());
}

QList<QColor> frmMain::colors = QList<QColor>();
void frmMain::initForm()
{
    //颜色集合供其他界面使用
    colors << QColor(211, 78, 78) << QColor(29, 185, 242) << QColor(170, 162, 119) << QColor(255, 192, 1);
    colors << QColor(0, 176, 180) << QColor(0, 113, 193) << QColor(255, 192, 0);
    colors << QColor(72, 103, 149) << QColor(185, 87, 86) << QColor(0, 177, 125);
    colors << QColor(214, 77, 84) << QColor(71, 164, 233) << QColor(34, 163, 169);
    colors << QColor(59, 123, 156) << QColor(162, 121, 197) << QColor(72, 202, 245);
    colors << QColor(0, 150, 121) << QColor(111, 9, 176) << QColor(250, 170, 20);

    ui->scrollArea->setFixedWidth(170);
    ui->widgetLeft->setProperty("flag", "left");
}

void frmMain::initWidget()
{
    //ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
    ui->stackedWidget->addWidget(new frmQuadratic);
    ui->stackedWidget->addWidget(new frmSimple);
    ui->stackedWidget->addWidget(new frmSincScatter);
    ui->stackedWidget->addWidget(new frmScatterStyle);
    ui->stackedWidget->addWidget(new frmScatterPixmap);

    ui->stackedWidget->addWidget(new frmLineStyle);
    ui->stackedWidget->addWidget(new frmDate);
    ui->stackedWidget->addWidget(new frmTextureBrush);
    ui->stackedWidget->addWidget(new frmMultiAxis);
    ui->stackedWidget->addWidget(new frmLogarithmic);

    ui->stackedWidget->addWidget(new frmRealtimeData);
    ui->stackedWidget->addWidget(new frmParametricCurve);
    ui->stackedWidget->addWidget(new frmBarChart);
    ui->stackedWidget->addWidget(new frmStatistical);
    ui->stackedWidget->addWidget(new frmSimpleItem);

    ui->stackedWidget->addWidget(new frmItem);
    ui->stackedWidget->addWidget(new frmStyled);
    ui->stackedWidget->addWidget(new frmAdvancedAxes);
    ui->stackedWidget->addWidget(new frmColorMap);
    ui->stackedWidget->addWidget(new frmFinancial);
    ui->stackedWidget->addWidget(new frmPolarPlot);

    ui->stackedWidget->addWidget(new frmAxisTag);
    ui->stackedWidget->addWidget(new frmInterAction);
    ui->stackedWidget->addWidget(new frmScrollBar);
}

void frmMain::initNav()
{
    //按钮文字集合
    QStringList names;
    names << "平方二次元" << "简单曲线图" << "正弦散点图" << "数据点样式" << "数据点图片"
          << "曲线条样式" << "日期数据图" << "纹理画刷图" << "双坐标曲线" << "对数曲线图"
          << "动态正弦图" << "环形曲线图" << "垂直柱状图" << "箱形盒须图" << "静态指示线"
          << "动态指示线" << "曲线样式图" << "高级坐标轴" << "颜色热力图" << "金融曲线图"
          << "南丁格尔图" << "坐标轴指示" << "相互作用图" << "滚动条曲线";

    //自动生成按钮
    for (int i = 0; i < names.count(); i++) {
        QToolButton *btn = new QToolButton;
        //设置按钮固定高度
        btn->setFixedHeight(35);
        //设置按钮的文字
        btn->setText(QString("%1. %2").arg(i + 1, 2, 10, QChar('0')).arg(names.at(i)));
        //设置按钮可选中按下类似复选框的功能
        btn->setCheckable(true);
        //设置按钮图标在左侧文字在右侧
        btn->setToolButtonStyle(Qt::ToolButtonTextBesideIcon);
        //设置按钮拉伸策略为横向填充
        btn->setSizePolicy(QSizePolicy::Expanding, QSizePolicy::Preferred);
        //关联按钮单击事件
        connect(btn, SIGNAL(clicked(bool)), this, SLOT(buttonClicked()));
        //将按钮加入到布局
        ui->widgetLeft->layout()->addWidget(btn);
        //可以指定不同的图标
        icons << 0xf061;
        btns << btn;
    }

    //底部加个弹簧
    QSpacerItem *verticalSpacer = new QSpacerItem(20, 40, QSizePolicy::Minimum, QSizePolicy::Expanding);
    ui->widgetLeft->layout()->addItem(verticalSpacer);
    btns.at(names.count() - 1)->click();
}

void frmMain::initIcon()
{
    //左侧导航样式,可以设置各种牛逼的参数,超级棒
    IconHelper::StyleColor styleColor;
    styleColor.defaultBorder = true;
    IconHelper::setStyle(ui->widgetLeft, btns, icons, styleColor);
}

void frmMain::buttonClicked()
{
    QAbstractButton *b = (QAbstractButton *)sender();
    int count = btns.count();
    int index = btns.indexOf(b);
    ui->stackedWidget->setCurrentIndex(index);

    for (int i = 0; i < count; i++) {
        QAbstractButton *btn = btns.at(i);
        btn->setChecked(btn == b);
    }
}
```

### `frmcustom/frmmain.h`

```cpp
﻿#ifndef FRMMAIN_H
#define FRMMAIN_H

#include <QWidget>
class QAbstractButton;

namespace Ui {
class frmMain;
}

class frmMain : public QWidget
{
    Q_OBJECT

public:
    static QList<QColor> colors;
    explicit frmMain(QWidget *parent = 0);
    ~frmMain();

protected:
    void showEvent(QShowEvent *);

private:
    Ui::frmMain *ui;

    //左侧导航栏图标+按钮集合
    QList<int> icons;
    QList<QAbstractButton *> btns;

private slots:
    void initForm();        //初始化界面数据
    void initWidget();      //初始化子窗体
    void initNav();         //初始化导航按钮
    void initIcon();        //初始化导航按钮图标
    void buttonClicked();   //导航按钮单击事件
};

#endif // FRMMAIN_H
```

### `frmcustom/iconhelper.cpp`

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

### `frmcustom/iconhelper.h`

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

### `frmexample/frmadvancedaxes.cpp`

```cpp
﻿#include "frmadvancedaxes.h"
#include "ui_frmadvancedaxes.h"
#include "qdebug.h"

frmAdvancedAxes::frmAdvancedAxes(QWidget *parent) : QWidget(parent), ui(new Ui::frmAdvancedAxes)
{
    ui->setupUi(this);
    this->initForm();
}

frmAdvancedAxes::~frmAdvancedAxes()
{
    delete ui;
}

void frmAdvancedAxes::initForm()
{
    // configure axis rect:
    ui->customPlot->plotLayout()->clear(); // clear default axis rect so we can start from scratch

    QCPAxisRect *wideAxisRect = new QCPAxisRect(ui->customPlot);
    wideAxisRect->setupFullAxesBox(true);
    wideAxisRect->axis(QCPAxis::atRight, 0)->setTickLabels(true);
    wideAxisRect->addAxis(QCPAxis::atLeft)->setTickLabelColor(QColor("#6050F8")); // add an extra axis on the left and color its numbers

    QCPLayoutGrid *subLayout = new QCPLayoutGrid;
    ui->customPlot->plotLayout()->addElement(0, 0, wideAxisRect); // insert axis rect in first row
    ui->customPlot->plotLayout()->addElement(1, 0, subLayout); // sub layout in second row (grid layout will grow accordingly)
    //ui->customPlot->plotLayout()->setRowStretchFactor(1, 2);

    // prepare axis rects that will be placed in the sublayout:
    QCPAxisRect *subRectLeft = new QCPAxisRect(ui->customPlot, false); // false means to not setup default axes
    QCPAxisRect *subRectRight = new QCPAxisRect(ui->customPlot, false);
    subLayout->addElement(0, 0, subRectLeft);
    subLayout->addElement(0, 1, subRectRight);
    subRectRight->setMaximumSize(100, 100); // make bottom right axis rect size fixed 100x100
    subRectRight->setMinimumSize(100, 100); // make bottom right axis rect size fixed 100x100

    // setup axes in sub layout axis rects:
    subRectLeft->addAxes(QCPAxis::atBottom | QCPAxis::atLeft);
    subRectRight->addAxes(QCPAxis::atBottom | QCPAxis::atRight);
    subRectLeft->axis(QCPAxis::atLeft)->ticker()->setTickCount(2);
    subRectRight->axis(QCPAxis::atRight)->ticker()->setTickCount(2);
    subRectRight->axis(QCPAxis::atBottom)->ticker()->setTickCount(2);
    subRectLeft->axis(QCPAxis::atBottom)->grid()->setVisible(true);

    // synchronize the left and right margins of the top and bottom axis rects:
    QCPMarginGroup *marginGroup = new QCPMarginGroup(ui->customPlot);
    subRectLeft->setMarginGroup(QCP::msLeft, marginGroup);
    subRectRight->setMarginGroup(QCP::msRight, marginGroup);
    wideAxisRect->setMarginGroup(QCP::msLeft | QCP::msRight, marginGroup);

    // move newly created axes on "axes" layer and grids on "grid" layer:
    foreach (QCPAxisRect *rect, ui->customPlot->axisRects()) {
        foreach (QCPAxis *axis, rect->axes()) {
            axis->setLayer("axes");
            axis->grid()->setLayer("grid");
        }
    }

    // prepare data:
    QVector<QCPGraphData> dataCos(21), dataGauss(50), dataRandom(100);
    QVector<double> x3, y3;
    std::srand(3);
    for (int i = 0; i < dataCos.size(); ++i) {
        dataCos[i].key = i / (double)(dataCos.size() - 1) * 10 - 5.0;
        dataCos[i].value = qCos(dataCos[i].key);
    }
    for (int i = 0; i < dataGauss.size(); ++i) {
        dataGauss[i].key = i / (double)dataGauss.size() * 10 - 5.0;
        dataGauss[i].value = qExp(-dataGauss[i].key * dataGauss[i].key * 0.2) * 1000;
    }
    for (int i = 0; i < dataRandom.size(); ++i) {
        dataRandom[i].key = i / (double)dataRandom.size() * 10;
        dataRandom[i].value = std::rand() / (double)RAND_MAX - 0.5 + dataRandom[qMax(0, i - 1)].value;
    }
    x3 << 1 << 2 << 3 << 4;
    y3 << 2 << 2.5 << 4 << 1.5;

    // create and configure plottables:
    QCPGraph *mainGraphCos = ui->customPlot->addGraph(wideAxisRect->axis(QCPAxis::atBottom), wideAxisRect->axis(QCPAxis::atLeft));
    mainGraphCos->data()->set(dataCos);
    mainGraphCos->valueAxis()->setRange(-1, 1);
    mainGraphCos->rescaleKeyAxis();
    mainGraphCos->setScatterStyle(QCPScatterStyle(QCPScatterStyle::ssCircle, QPen(Qt::black), QBrush(Qt::white), 6));
    mainGraphCos->setPen(QPen(QColor(120, 120, 120), 2));

    QCPGraph *mainGraphGauss = ui->customPlot->addGraph(wideAxisRect->axis(QCPAxis::atBottom), wideAxisRect->axis(QCPAxis::atLeft, 1));
    mainGraphGauss->data()->set(dataGauss);
    mainGraphGauss->setPen(QPen(QColor("#8070B8"), 2));
    mainGraphGauss->setBrush(QColor(110, 170, 110, 30));
    mainGraphCos->setChannelFillGraph(mainGraphGauss);
    mainGraphCos->setBrush(QColor(255, 161, 0, 50));
    mainGraphGauss->valueAxis()->setRange(0, 1000);
    mainGraphGauss->rescaleKeyAxis();

    QCPGraph *subGraphRandom = ui->customPlot->addGraph(subRectLeft->axis(QCPAxis::atBottom), subRectLeft->axis(QCPAxis::atLeft));
    subGraphRandom->data()->set(dataRandom);
    subGraphRandom->setLineStyle(QCPGraph::lsImpulse);
    subGraphRandom->setPen(QPen(QColor("#FFA100"), 1.5));
    subGraphRandom->rescaleAxes();

    QCPBars *subBars = new QCPBars(subRectRight->axis(QCPAxis::atBottom), subRectRight->axis(QCPAxis::atRight));
    subBars->setWidth(3 / (double)x3.size());
    subBars->setData(x3, y3);
    subBars->setPen(QPen(Qt::black));
    subBars->setAntialiased(false);
    subBars->setAntialiasedFill(false);
    subBars->setBrush(QColor("#705BE8"));
    subBars->keyAxis()->setSubTicks(false);
    subBars->rescaleAxes();

    // setup a ticker for subBars key axis that only gives integer ticks:
    QSharedPointer<QCPAxisTickerFixed> intTicker(new QCPAxisTickerFixed);
    intTicker->setTickStep(1.0);
    intTicker->setScaleStrategy(QCPAxisTickerFixed::ssMultiples);
    subBars->keyAxis()->setTicker(intTicker);
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmadvancedaxes.h`

```cpp
﻿#ifndef FRMADVANCEDAXES_H
#define FRMADVANCEDAXES_H

#include <QWidget>

namespace Ui {
class frmAdvancedAxes;
}

class frmAdvancedAxes : public QWidget
{
    Q_OBJECT

public:
    explicit frmAdvancedAxes(QWidget *parent = 0);
    ~frmAdvancedAxes();

private:
    Ui::frmAdvancedAxes *ui;

private slots:
    void initForm();
};

#endif // FRMADVANCEDAXES_H
```

### `frmexample/frmbarchart.cpp`

```cpp
﻿#include "frmbarchart.h"
#include "ui_frmbarchart.h"
#include "qdebug.h"

frmBarChart::frmBarChart(QWidget *parent) : QWidget(parent), ui(new Ui::frmBarChart)
{
    ui->setupUi(this);
    this->initForm();
}

frmBarChart::~frmBarChart()
{
    delete ui;
}

void frmBarChart::initForm()
{
    // set dark background gradient:
    QLinearGradient gradient(0, 0, 0, 400);
    gradient.setColorAt(0, QColor(90, 90, 90));
    gradient.setColorAt(0.38, QColor(105, 105, 105));
    gradient.setColorAt(1, QColor(70, 70, 70));
    ui->customPlot->setBackground(QBrush(gradient));

    // create empty bar chart objects:
    QCPBars *regen = new QCPBars(ui->customPlot->xAxis, ui->customPlot->yAxis);
    QCPBars *nuclear = new QCPBars(ui->customPlot->xAxis, ui->customPlot->yAxis);
    QCPBars *fossil = new QCPBars(ui->customPlot->xAxis, ui->customPlot->yAxis);

    regen->setAntialiased(false); // gives more crisp, pixel aligned bar borders
    nuclear->setAntialiased(false);
    fossil->setAntialiased(false);
    regen->setStackingGap(1);
    nuclear->setStackingGap(1);
    fossil->setStackingGap(1);

    // set names and colors:
    fossil->setName("Fossil fuels");
    fossil->setPen(QPen(QColor(111, 9, 176).lighter(170)));
    fossil->setBrush(QColor(111, 9, 176));
    nuclear->setName("Nuclear");
    nuclear->setPen(QPen(QColor(250, 170, 20).lighter(150)));
    nuclear->setBrush(QColor(250, 170, 20));
    regen->setName("Regenerative");
    regen->setPen(QPen(QColor(0, 168, 140).lighter(130)));
    regen->setBrush(QColor(0, 168, 140));

    // stack bars on top of each other:
    nuclear->moveAbove(fossil);
    regen->moveAbove(nuclear);

    // prepare x axis with country labels:
    QVector<double> ticks;
    QVector<QString> labels;
    ticks << 1 << 2 << 3 << 4 << 5 << 6 << 7;
    labels << "USA" << "Japan" << "Germany" << "France" << "UK" << "Italy" << "Canada";

    QSharedPointer<QCPAxisTickerText> textTicker(new QCPAxisTickerText);
    textTicker->addTicks(ticks, labels);
    ui->customPlot->xAxis->setTicker(textTicker);
    ui->customPlot->xAxis->setTickLabelRotation(60);
    ui->customPlot->xAxis->setSubTicks(false);
    ui->customPlot->xAxis->setTickLength(0, 4);
    ui->customPlot->xAxis->setRange(0, 8);
    ui->customPlot->xAxis->setBasePen(QPen(Qt::white));
    ui->customPlot->xAxis->setTickPen(QPen(Qt::white));
    ui->customPlot->xAxis->grid()->setVisible(true);
    ui->customPlot->xAxis->grid()->setPen(QPen(QColor(130, 130, 130), 0, Qt::DotLine));
    ui->customPlot->xAxis->setTickLabelColor(Qt::white);
    ui->customPlot->xAxis->setLabelColor(Qt::white);

    // prepare y axis:
    ui->customPlot->yAxis->setRange(0, 12.1);
    ui->customPlot->yAxis->setPadding(5); // a bit more space to the left border
    ui->customPlot->yAxis->setLabel("Power Consumption in\nKilowatts per Capita (2007)");
    ui->customPlot->yAxis->setBasePen(QPen(Qt::white));
    ui->customPlot->yAxis->setTickPen(QPen(Qt::white));
    ui->customPlot->yAxis->setSubTickPen(QPen(Qt::white));
    ui->customPlot->yAxis->grid()->setSubGridVisible(true);
    ui->customPlot->yAxis->setTickLabelColor(Qt::white);
    ui->customPlot->yAxis->setLabelColor(Qt::white);
    ui->customPlot->yAxis->grid()->setPen(QPen(QColor(130, 130, 130), 0, Qt::SolidLine));
    ui->customPlot->yAxis->grid()->setSubGridPen(QPen(QColor(130, 130, 130), 0, Qt::DotLine));

    // Add data:
    QVector<double> fossilData, nuclearData, regenData;
    fossilData  << 0.86 * 10.5 << 0.83 * 5.5 << 0.84 * 5.5 << 0.52 * 5.8 << 0.89 * 5.2 << 0.90 * 4.2 << 0.67 * 11.2;
    nuclearData << 0.08 * 10.5 << 0.12 * 5.5 << 0.12 * 5.5 << 0.40 * 5.8 << 0.09 * 5.2 << 0.00 * 4.2 << 0.07 * 11.2;
    regenData   << 0.06 * 10.5 << 0.05 * 5.5 << 0.04 * 5.5 << 0.06 * 5.8 << 0.02 * 5.2 << 0.07 * 4.2 << 0.25 * 11.2;
    fossil->setData(ticks, fossilData);
    nuclear->setData(ticks, nuclearData);
    regen->setData(ticks, regenData);

    // setup legend:
    ui->customPlot->legend->setVisible(true);
    ui->customPlot->axisRect()->insetLayout()->setInsetAlignment(0, Qt::AlignTop | Qt::AlignHCenter);
    ui->customPlot->legend->setBrush(QColor(255, 255, 255, 100));
    ui->customPlot->legend->setBorderPen(Qt::NoPen);
    QFont legendFont = font();
    legendFont.setPointSize(10);
    ui->customPlot->legend->setFont(legendFont);
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmbarchart.h`

```cpp
﻿#ifndef FRMBARCHART_H
#define FRMBARCHART_H

#include <QWidget>

namespace Ui {
class frmBarChart;
}

class frmBarChart : public QWidget
{
    Q_OBJECT

public:
    explicit frmBarChart(QWidget *parent = 0);
    ~frmBarChart();

private:
    Ui::frmBarChart *ui;

private slots:
    void initForm();
};

#endif // FRMBARCHART_H
```

### `frmexample/frmcolormap.cpp`

```cpp
﻿#include "frmcolormap.h"
#include "ui_frmcolormap.h"
#include "qdebug.h"

frmColorMap::frmColorMap(QWidget *parent) : QWidget(parent), ui(new Ui::frmColorMap)
{
    ui->setupUi(this);
    this->initForm();
}

frmColorMap::~frmColorMap()
{
    delete ui;
}

void frmColorMap::initForm()
{
    // configure axis rect:
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom); // this will also allow rescaling the color scale by dragging/zooming
    ui->customPlot->axisRect()->setupFullAxesBox(true);
    ui->customPlot->xAxis->setLabel("x");
    ui->customPlot->yAxis->setLabel("y");

    // set up the QCPColorMap:
    QCPColorMap *colorMap = new QCPColorMap(ui->customPlot->xAxis, ui->customPlot->yAxis);
    int nx = 200;
    int ny = 200;
    colorMap->data()->setSize(nx, ny); // we want the color map to have nx * ny data points
    colorMap->data()->setRange(QCPRange(-4, 4), QCPRange(-4, 4)); // and span the coordinate range -4..4 in both key (x) and value (y) dimensions

    // now we assign some data, by accessing the QCPColorMapData instance of the color map:
    double x, y, z;
    for (int xIndex = 0; xIndex < nx; ++xIndex) {
        for (int yIndex = 0; yIndex < ny; ++yIndex) {
            colorMap->data()->cellToCoord(xIndex, yIndex, &x, &y);
            double r = 3 * qSqrt(x * x + y * y) + 1e-2;
            z = 2 * x * (qCos(r + 2) / r - qSin(r + 2) / r); // the B field strength of dipole radiation (modulo physical constants)
            colorMap->data()->setCell(xIndex, yIndex, z);
        }
    }

    // add a color scale:
    QCPColorScale *colorScale = new QCPColorScale(ui->customPlot);
    ui->customPlot->plotLayout()->addElement(0, 1, colorScale); // add it to the right of the main axis rect
    colorScale->setType(QCPAxis::atRight); // scale shall be vertical bar with tick/axis labels right (actually atRight is already the default)
    colorMap->setColorScale(colorScale); // associate the color map with the color scale
    colorScale->axis()->setLabel("Magnetic Field Strength");

    // set the color gradient of the color map to one of the presets:
    colorMap->setGradient(QCPColorGradient::gpPolar);
    // we could have also created a QCPColorGradient instance and added own colors to
    // the gradient, see the documentation of QCPColorGradient for what's possible.

    // rescale the data dimension (color) such that all data points lie in the span visualized by the color gradient:
    colorMap->rescaleDataRange();

    // make sure the axis rect and color scale synchronize their bottom and top margins (so they line up):
    QCPMarginGroup *marginGroup = new QCPMarginGroup(ui->customPlot);
    ui->customPlot->axisRect()->setMarginGroup(QCP::msBottom | QCP::msTop, marginGroup);
    colorScale->setMarginGroup(QCP::msBottom | QCP::msTop, marginGroup);

    // rescale the key (x) and value (y) axes so the whole color map is visible:
    ui->customPlot->rescaleAxes();
}
```

### `frmexample/frmcolormap.h`

```cpp
﻿#ifndef FRMCOLORMAP_H
#define FRMCOLORMAP_H

#include <QWidget>

namespace Ui {
class frmColorMap;
}

class frmColorMap : public QWidget
{
    Q_OBJECT

public:
    explicit frmColorMap(QWidget *parent = 0);
    ~frmColorMap();

private:
    Ui::frmColorMap *ui;

private slots:
    void initForm();
};

#endif // FRMCOLORMAP_H
```

### `frmexample/frmdate.cpp`

```cpp
﻿#include "frmdate.h"
#include "ui_frmdate.h"
#include "qdebug.h"

frmDate::frmDate(QWidget *parent) : QWidget(parent), ui(new Ui::frmDate)
{
    ui->setupUi(this);
    this->initForm();
}

frmDate::~frmDate()
{
    delete ui;
}

void frmDate::initForm()
{
    // set locale to english, so we get english month names:
    ui->customPlot->setLocale(QLocale(QLocale::English, QLocale::UnitedKingdom));

    // seconds of current time, we'll use it as starting point in time for data:
    double now = QDateTime::currentDateTime().toMSecsSinceEpoch() / 1000.0;
    srand(8); // set the random seed, so we always get the same random data
    // create multiple graphs:
    for (int gi = 0; gi < 5; ++gi) {
        ui->customPlot->addGraph();
        QColor color(20 + 200 / 4.0 * gi, 70 * (1.6 - gi / 4.0), 150, 150);
        ui->customPlot->graph()->setLineStyle(QCPGraph::lsLine);
        ui->customPlot->graph()->setPen(QPen(color.lighter(200)));
        ui->customPlot->graph()->setBrush(QBrush(color));
        // generate random walk data:
        QVector<QCPGraphData> timeData(250);
        for (int i = 0; i < 250; ++i) {
            timeData[i].key = now + 24 * 3600 * i;
            if (i == 0) {
                timeData[i].value = (i / 50.0 + 1) * (rand() / (double)RAND_MAX - 0.5);
            } else {
                timeData[i].value = qFabs(timeData[i - 1].value) * (1 + 0.02 / 4.0 * (4 - gi)) + (i / 50.0 + 1) * (rand() / (double)RAND_MAX - 0.5);
            }
        }
        ui->customPlot->graph()->data()->set(timeData);
    }

    // configure bottom axis to show date instead of number:
    QSharedPointer<QCPAxisTickerDateTime> dateTicker(new QCPAxisTickerDateTime);
    dateTicker->setDateTimeFormat("d. MMMM\nyyyy");
    ui->customPlot->xAxis->setTicker(dateTicker);

    // configure left axis text labels:
    QSharedPointer<QCPAxisTickerText> textTicker(new QCPAxisTickerText);
    textTicker->addTick(10, "a bit\nlow");
    textTicker->addTick(50, "quite\nhigh");
    ui->customPlot->yAxis->setTicker(textTicker);

    // set a more compact font size for bottom and left axis tick labels:
    ui->customPlot->xAxis->setTickLabelFont(QFont(QFont().family(), 8));
    ui->customPlot->yAxis->setTickLabelFont(QFont(QFont().family(), 8));

    // set axis labels:
    ui->customPlot->xAxis->setLabel("Date");
    ui->customPlot->yAxis->setLabel("Random wobbly lines value");

    // make top and right axes visible but without ticks and labels:
    ui->customPlot->xAxis2->setVisible(true);
    ui->customPlot->yAxis2->setVisible(true);
    ui->customPlot->xAxis2->setTicks(false);
    ui->customPlot->yAxis2->setTicks(false);
    ui->customPlot->xAxis2->setTickLabels(false);
    ui->customPlot->yAxis2->setTickLabels(false);

    // set axis ranges to show all data:
    ui->customPlot->xAxis->setRange(now, now + 24 * 3600 * 249);
    ui->customPlot->yAxis->setRange(0, 60);

    // show legend with slightly transparent background brush:
    ui->customPlot->legend->setVisible(true);
    ui->customPlot->legend->setBrush(QColor(255, 255, 255, 150));
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmdate.h`

```cpp
﻿#ifndef FRMDATE_H
#define FRMDATE_H

#include <QWidget>

namespace Ui {
class frmDate;
}

class frmDate : public QWidget
{
    Q_OBJECT

public:
    explicit frmDate(QWidget *parent = 0);
    ~frmDate();

private:
    Ui::frmDate *ui;

private slots:
    void initForm();
};

#endif // FRMDATE_H
```

### `frmexample/frmfinancial.cpp`

```cpp
﻿#include "frmfinancial.h"
#include "ui_frmfinancial.h"
#include "qdebug.h"

frmFinancial::frmFinancial(QWidget *parent) : QWidget(parent), ui(new Ui::frmFinancial)
{
    ui->setupUi(this);
    this->initForm();
}

frmFinancial::~frmFinancial()
{
    delete ui;
}

void frmFinancial::initForm()
{
    ui->customPlot->legend->setVisible(true);

    // generate two sets of random walk data (one for candlestick and one for ohlc chart):
    int n = 500;
    QVector<double> time(n), value1(n), value2(n);
    QDateTime start(QDate(2022, 1, 1), QTime(0, 0));
    start.setTimeSpec(Qt::UTC);
    double startTime = start.toMSecsSinceEpoch() / 1000.0;
    double binSize = 3600 * 24; // bin data in 1 day intervals
    time[0] = startTime;
    value1[0] = 60;
    value2[0] = 20;
    std::srand(9);
    for (int i = 1; i < n; ++i) {
        time[i] = startTime + 3600 * i;
        value1[i] = value1[i - 1] + (std::rand() / (double)RAND_MAX - 0.5) * 10;
        value2[i] = value2[i - 1] + (std::rand() / (double)RAND_MAX - 0.5) * 3;
    }

    // create candlestick chart:
    QCPFinancial *candlesticks = new QCPFinancial(ui->customPlot->xAxis, ui->customPlot->yAxis);
    candlesticks->setName("Candlestick");
    candlesticks->setChartStyle(QCPFinancial::csCandlestick);
    candlesticks->data()->set(QCPFinancial::timeSeriesToOhlc(time, value1, binSize, startTime));
    candlesticks->setWidth(binSize * 0.9);
    candlesticks->setTwoColored(true);
    candlesticks->setBrushPositive(QColor(245, 245, 245));
    candlesticks->setBrushNegative(QColor(40, 40, 40));
    candlesticks->setPenPositive(QPen(QColor(0, 0, 0)));
    candlesticks->setPenNegative(QPen(QColor(0, 0, 0)));

    // create ohlc chart:
    QCPFinancial *ohlc = new QCPFinancial(ui->customPlot->xAxis, ui->customPlot->yAxis);
    ohlc->setName("OHLC");
    ohlc->setChartStyle(QCPFinancial::csOhlc);
    ohlc->data()->set(QCPFinancial::timeSeriesToOhlc(time, value2, binSize / 3.0, startTime)); // divide binSize by 3 just to make the ohlc bars a bit denser
    ohlc->setWidth(binSize * 0.2);
    ohlc->setTwoColored(true);

    // create bottom axis rect for volume bar chart:
    QCPAxisRect *volumeAxisRect = new QCPAxisRect(ui->customPlot);
    ui->customPlot->plotLayout()->addElement(1, 0, volumeAxisRect);
    volumeAxisRect->setMaximumSize(QSize(QWIDGETSIZE_MAX, 100));
    volumeAxisRect->axis(QCPAxis::atBottom)->setLayer("axes");
    volumeAxisRect->axis(QCPAxis::atBottom)->grid()->setLayer("grid");

    // bring bottom and main axis rect closer together:
    ui->customPlot->plotLayout()->setRowSpacing(0);
    volumeAxisRect->setAutoMargins(QCP::msLeft | QCP::msRight | QCP::msBottom);
    volumeAxisRect->setMargins(QMargins(0, 0, 0, 0));

    // create two bar plottables, for positive (green) and negative (red) volume bars:
    ui->customPlot->setAutoAddPlottableToLegend(false);
    QCPBars *volumePos = new QCPBars(volumeAxisRect->axis(QCPAxis::atBottom), volumeAxisRect->axis(QCPAxis::atLeft));
    QCPBars *volumeNeg = new QCPBars(volumeAxisRect->axis(QCPAxis::atBottom), volumeAxisRect->axis(QCPAxis::atLeft));
    for (int i = 0; i < n / 5; ++i) {
        int v = std::rand() % 20000 + std::rand() % 20000 + std::rand() % 20000 - 10000 * 3;
        (v < 0 ? volumeNeg : volumePos)->addData(startTime + 3600 * 5.0 * i, qAbs(v)); // add data to either volumeNeg or volumePos, depending on sign of v
    }

    volumePos->setWidth(3600 * 4);
    volumePos->setPen(Qt::NoPen);
    volumePos->setBrush(QColor(100, 180, 110));
    volumeNeg->setWidth(3600 * 4);
    volumeNeg->setPen(Qt::NoPen);
    volumeNeg->setBrush(QColor(180, 90, 90));

    // interconnect x axis ranges of main and bottom axis rects:
    connect(ui->customPlot->xAxis, SIGNAL(rangeChanged(QCPRange)), volumeAxisRect->axis(QCPAxis::atBottom), SLOT(setRange(QCPRange)));
    connect(volumeAxisRect->axis(QCPAxis::atBottom), SIGNAL(rangeChanged(QCPRange)), ui->customPlot->xAxis, SLOT(setRange(QCPRange)));

    // configure axes of both main and bottom axis rect:
    QSharedPointer<QCPAxisTickerDateTime> dateTimeTicker(new QCPAxisTickerDateTime);
    dateTimeTicker->setDateTimeSpec(Qt::UTC);
    dateTimeTicker->setDateTimeFormat("dd. MMMM");
    volumeAxisRect->axis(QCPAxis::atBottom)->setTicker(dateTimeTicker);
    volumeAxisRect->axis(QCPAxis::atBottom)->setTickLabelRotation(15);
    ui->customPlot->xAxis->setBasePen(Qt::NoPen);
    ui->customPlot->xAxis->setTickLabels(false);
    ui->customPlot->xAxis->setTicks(false); // only want vertical grid in main axis rect, so hide xAxis backbone, ticks, and labels
    ui->customPlot->xAxis->setTicker(dateTimeTicker);
    ui->customPlot->rescaleAxes();
    ui->customPlot->xAxis->scaleRange(1.025, ui->customPlot->xAxis->range().center());
    ui->customPlot->yAxis->scaleRange(1.1, ui->customPlot->yAxis->range().center());

    // make axis rects' left side line up:
    QCPMarginGroup *group = new QCPMarginGroup(ui->customPlot);
    ui->customPlot->axisRect()->setMarginGroup(QCP::msLeft | QCP::msRight, group);
    volumeAxisRect->setMarginGroup(QCP::msLeft | QCP::msRight, group);
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmfinancial.h`

```cpp
﻿#ifndef FRMFINANCIAL_H
#define FRMFINANCIAL_H

#include <QWidget>

namespace Ui {
class frmFinancial;
}

class frmFinancial : public QWidget
{
    Q_OBJECT

public:
    explicit frmFinancial(QWidget *parent = 0);
    ~frmFinancial();

private:
    Ui::frmFinancial *ui;

private slots:
    void initForm();
};

#endif // FRMFINANCIAL_H
```

### `frmexample/frmitem.cpp`

```cpp
﻿#include "frmitem.h"
#include "ui_frmitem.h"
#include "qdebug.h"

frmItem::frmItem(QWidget *parent) : QWidget(parent), ui(new Ui::frmItem)
{
    ui->setupUi(this);
    this->initForm();
}

frmItem::~frmItem()
{
    delete ui;
}

void frmItem::showEvent(QShowEvent *)
{
    dataTimer.start(0);
}

void frmItem::hideEvent(QHideEvent *)
{
    dataTimer.stop();
}

void frmItem::initForm()
{
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
    QCPGraph *graph = ui->customPlot->addGraph();
    int n = 500;
    double phase = 0;
    double k = 3;
    QVector<double> x(n), y(n);
    for (int i = 0; i < n; ++i) {
        x[i] = i / (double)(n - 1) * 34 - 17;
        y[i] = qExp(-x[i] * x[i] / 20.0) * qSin(k * x[i] + phase);
    }

    graph->setData(x, y);
    graph->setPen(QPen(Qt::blue));
    graph->rescaleKeyAxis();
    ui->customPlot->yAxis->setRange(-1.45, 1.65);
    ui->customPlot->xAxis->grid()->setZeroLinePen(Qt::NoPen);

    // add the bracket at the top:
    QCPItemBracket *bracket = new QCPItemBracket(ui->customPlot);
    bracket->left->setCoords(-8, 1.1);
    bracket->right->setCoords(8, 1.1);
    bracket->setLength(13);

    // add the text label at the top:
    QCPItemText *wavePacketText = new QCPItemText(ui->customPlot);
    wavePacketText->position->setParentAnchor(bracket->center);
    wavePacketText->position->setCoords(0, -10); // move 10 pixels to the top from bracket center anchor
    wavePacketText->setPositionAlignment(Qt::AlignBottom | Qt::AlignHCenter);
    wavePacketText->setText("Wavepacket");
    wavePacketText->setFont(QFont(font().family(), 10));

    // add the phase tracer (red circle) which sticks to the graph data (and gets updated in bracketDataSlot by timer event):
    phaseTracer = new QCPItemTracer(ui->customPlot);
    phaseTracer->setGraph(graph);
    phaseTracer->setGraphKey((M_PI * 1.5 - phase) / k);
    phaseTracer->setInterpolating(true);
    phaseTracer->setStyle(QCPItemTracer::tsCircle);
    phaseTracer->setPen(QPen(Qt::red));
    phaseTracer->setBrush(Qt::red);
    phaseTracer->setSize(7);

    // add label for phase tracer:
    QCPItemText *phaseTracerText = new QCPItemText(ui->customPlot);
    phaseTracerText->position->setType(QCPItemPosition::ptAxisRectRatio);
    phaseTracerText->setPositionAlignment(Qt::AlignRight | Qt::AlignBottom);
    phaseTracerText->position->setCoords(1.0, 0.95); // lower right corner of axis rect
    phaseTracerText->setText("Points of fixed\nphase define\nphase velocity vp");
    phaseTracerText->setTextAlignment(Qt::AlignLeft);
    phaseTracerText->setFont(QFont(font().family(), 9));
    phaseTracerText->setPadding(QMargins(8, 0, 0, 0));

    // add arrow pointing at phase tracer, coming from label:
    QCPItemCurve *phaseTracerArrow = new QCPItemCurve(ui->customPlot);
    phaseTracerArrow->start->setParentAnchor(phaseTracerText->left);
    phaseTracerArrow->startDir->setParentAnchor(phaseTracerArrow->start);
    phaseTracerArrow->startDir->setCoords(-40, 0); // direction 30 pixels to the left of parent anchor (tracerArrow->start)
    phaseTracerArrow->end->setParentAnchor(phaseTracer->position);
    phaseTracerArrow->end->setCoords(10, 10);
    phaseTracerArrow->endDir->setParentAnchor(phaseTracerArrow->end);
    phaseTracerArrow->endDir->setCoords(30, 30);
    phaseTracerArrow->setHead(QCPLineEnding::esSpikeArrow);
    phaseTracerArrow->setTail(QCPLineEnding(QCPLineEnding::esBar, (phaseTracerText->bottom->pixelPosition().y() - phaseTracerText->top->pixelPosition().y()) * 0.85));

    // add the group velocity tracer (green circle):
    QCPItemTracer *groupTracer = new QCPItemTracer(ui->customPlot);
    groupTracer->setGraph(graph);
    groupTracer->setGraphKey(5.5);
    groupTracer->setInterpolating(true);
    groupTracer->setStyle(QCPItemTracer::tsCircle);
    groupTracer->setPen(QPen(Qt::green));
    groupTracer->setBrush(Qt::green);
    groupTracer->setSize(7);

    // add label for group tracer:
    QCPItemText *groupTracerText = new QCPItemText(ui->customPlot);
    groupTracerText->position->setType(QCPItemPosition::ptAxisRectRatio);
    groupTracerText->setPositionAlignment(Qt::AlignRight | Qt::AlignTop);
    groupTracerText->position->setCoords(1.0, 0.20); // lower right corner of axis rect
    groupTracerText->setText("Fixed positions in\nwave packet define\ngroup velocity vg");
    groupTracerText->setTextAlignment(Qt::AlignLeft);
    groupTracerText->setFont(QFont(font().family(), 9));
    groupTracerText->setPadding(QMargins(8, 0, 0, 0));

    // add arrow pointing at group tracer, coming from label:
    QCPItemCurve *groupTracerArrow = new QCPItemCurve(ui->customPlot);
    groupTracerArrow->start->setParentAnchor(groupTracerText->left);
    groupTracerArrow->startDir->setParentAnchor(groupTracerArrow->start);
    groupTracerArrow->startDir->setCoords(-40, 0); // direction 30 pixels to the left of parent anchor (tracerArrow->start)
    groupTracerArrow->end->setCoords(5.5, 0.4);
    groupTracerArrow->endDir->setParentAnchor(groupTracerArrow->end);
    groupTracerArrow->endDir->setCoords(0, -40);
    groupTracerArrow->setHead(QCPLineEnding::esSpikeArrow);
    groupTracerArrow->setTail(QCPLineEnding(QCPLineEnding::esBar, (groupTracerText->bottom->pixelPosition().y() - groupTracerText->top->pixelPosition().y()) * 0.85));

    // add dispersion arrow:
    QCPItemCurve *arrow = new QCPItemCurve(ui->customPlot);
    arrow->start->setCoords(1, -1.1);
    arrow->startDir->setCoords(-1, -1.3);
    arrow->endDir->setCoords(-5, -0.3);
    arrow->end->setCoords(-10, -0.2);
    arrow->setHead(QCPLineEnding::esSpikeArrow);

    // add the dispersion arrow label:
    QCPItemText *dispersionText = new QCPItemText(ui->customPlot);
    dispersionText->position->setCoords(-6, -0.9);
    dispersionText->setRotation(40);
    dispersionText->setText("Dispersion with\nvp < vg");
    dispersionText->setFont(QFont(font().family(), 10));

    // setup a timer that repeatedly calls MainWindow::bracketDataSlot:
    connect(&dataTimer, SIGNAL(timeout()), this, SLOT(bracketDataSlot()));    
}

void frmItem::bracketDataSlot()
{
    double secs = QCPAxisTickerDateTime::dateTimeToKey(QDateTime::currentDateTime());

    // update data to make phase move:
    int n = 500;
    double phase = secs * 5;
    double k = 3;
    QVector<double> x(n), y(n);
    for (int i = 0; i < n; ++i) {
        x[i] = i / (double)(n - 1) * 34 - 17;
        y[i] = qExp(-x[i] * x[i] / 20.0) * qSin(k * x[i] + phase);
    }

    ui->customPlot->graph()->setData(x, y);
    phaseTracer->setGraphKey((8 * M_PI + fmod(M_PI * 1.5 - phase, 6 * M_PI)) / k);
    ui->customPlot->replot();

    // calculate frames per second:
    double key = secs;
    static double lastFpsKey;
    static int frameCount;
    ++frameCount;
    if (key - lastFpsKey > 2) { // average fps over 2 seconds
//        ui->statusBar->showMessage(
//            QString("%1 FPS, Total Data points: %2")
//            .arg(frameCount / (key - lastFpsKey), 0, 'f', 0)
//            .arg(ui->customPlot->graph(0)->data()->size())
//            , 0);
        lastFpsKey = key;
        frameCount = 0;
    }
}
```

### `frmexample/frmitem.h`

```cpp
﻿#ifndef FRMITEM_H
#define FRMITEM_H

#include <QWidget>
#include <QTimer>
class QCPItemTracer;

namespace Ui {
class frmItem;
}

class frmItem : public QWidget
{
    Q_OBJECT

public:
    explicit frmItem(QWidget *parent = 0);
    ~frmItem();

protected:
    void showEvent(QShowEvent *);
    void hideEvent(QHideEvent *);

private:
    Ui::frmItem *ui;
    QTimer dataTimer;
    QCPItemTracer *phaseTracer;

private slots:
    void initForm();
    void bracketDataSlot();
};

#endif // FRMITEM_H
```

### `frmexample/frmlinestyle.cpp`

```cpp
﻿#include "frmlinestyle.h"
#include "ui_frmlinestyle.h"
#include "qdebug.h"

frmLineStyle::frmLineStyle(QWidget *parent) : QWidget(parent), ui(new Ui::frmLineStyle)
{
    ui->setupUi(this);
    this->initForm();
}

frmLineStyle::~frmLineStyle()
{
    delete ui;
}

void frmLineStyle::initForm()
{
    ui->customPlot->legend->setVisible(true);
    ui->customPlot->legend->setFont(QFont("Helvetica", 9));

    QPen pen;
    QStringList lineNames;
    lineNames << "lsNone" << "lsLine" << "lsStepLeft" << "lsStepRight" << "lsStepCenter" << "lsImpulse";

    // add graphs with different line styles:
    for (int i = QCPGraph::lsNone; i <= QCPGraph::lsImpulse; ++i) {
        ui->customPlot->addGraph();
        pen.setColor(QColor(qSin(i * 1 + 1.2) * 80 + 80, qSin(i * 0.3 + 0) * 80 + 80, qSin(i * 0.3 + 1.5) * 80 + 80));
        ui->customPlot->graph()->setPen(pen);
        ui->customPlot->graph()->setName(lineNames.at(i - QCPGraph::lsNone));
        ui->customPlot->graph()->setLineStyle((QCPGraph::LineStyle)i);
        ui->customPlot->graph()->setScatterStyle(QCPScatterStyle(QCPScatterStyle::ssCircle, 5));

        // generate data:
        QVector<double> x(15), y(15);
        for (int j = 0; j < 15; ++j) {
            x[j] = j / 15.0 * 5 * 3.14 + 0.01;
            y[j] = 7 * qSin(x[j]) / x[j] - (i - QCPGraph::lsNone) * 5 + (QCPGraph::lsImpulse) * 5 + 2;
        }

        ui->customPlot->graph()->setData(x, y);
        ui->customPlot->graph()->rescaleAxes(true);
    }

    // zoom out a bit:
    ui->customPlot->yAxis->scaleRange(1.1, ui->customPlot->yAxis->range().center());
    ui->customPlot->xAxis->scaleRange(1.1, ui->customPlot->xAxis->range().center());

    // set blank axis lines:
    ui->customPlot->xAxis->setTicks(false);
    ui->customPlot->yAxis->setTicks(true);
    ui->customPlot->xAxis->setTickLabels(false);
    ui->customPlot->yAxis->setTickLabels(true);

    // make top right axes clones of bottom left axes:
    ui->customPlot->axisRect()->setupFullAxesBox();
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmlinestyle.h`

```cpp
﻿#ifndef FRMLINESTYLE_H
#define FRMLINESTYLE_H

#include <QWidget>

namespace Ui {
class frmLineStyle;
}

class frmLineStyle : public QWidget
{
    Q_OBJECT

public:
    explicit frmLineStyle(QWidget *parent = 0);
    ~frmLineStyle();

private:
    Ui::frmLineStyle *ui;

private slots:
    void initForm();
};

#endif // FRMLINESTYLE_H
```

### `frmexample/frmlogarithmic.cpp`

```cpp
﻿#include "frmlogarithmic.h"
#include "ui_frmlogarithmic.h"
#include "qdebug.h"

frmLogarithmic::frmLogarithmic(QWidget *parent) : QWidget(parent), ui(new Ui::frmLogarithmic)
{
    ui->setupUi(this);
    this->initForm();
}

frmLogarithmic::~frmLogarithmic()
{
    delete ui;
}

void frmLogarithmic::initForm()
{
    ui->customPlot->setNoAntialiasingOnDrag(true); // more performance/responsiveness during dragging
    ui->customPlot->addGraph();

    QPen pen;
    pen.setColor(QColor(255, 170, 100));
    pen.setWidth(2);
    pen.setStyle(Qt::DotLine);
    ui->customPlot->graph(0)->setPen(pen);
    ui->customPlot->graph(0)->setName("x");

    ui->customPlot->addGraph();
    ui->customPlot->graph(1)->setPen(QPen(Qt::red));
    ui->customPlot->graph(1)->setBrush(QBrush(QColor(255, 0, 0, 20)));
    ui->customPlot->graph(1)->setName("-sin(x)exp(x)");

    ui->customPlot->addGraph();
    ui->customPlot->graph(2)->setPen(QPen(Qt::blue));
    ui->customPlot->graph(2)->setBrush(QBrush(QColor(0, 0, 255, 20)));
    ui->customPlot->graph(2)->setName(" sin(x)exp(x)");

    ui->customPlot->addGraph();
    pen.setColor(QColor(0, 0, 0));
    pen.setWidth(1);
    pen.setStyle(Qt::DashLine);
    ui->customPlot->graph(3)->setPen(pen);
    ui->customPlot->graph(3)->setBrush(QBrush(QColor(0, 0, 0, 15)));
    ui->customPlot->graph(3)->setLineStyle(QCPGraph::lsStepCenter);
    ui->customPlot->graph(3)->setName("x!");

    const int dataCount = 200;
    const int dataFactorialCount = 21;
    QVector<QCPGraphData> dataLinear(dataCount), dataMinusSinExp(dataCount), dataPlusSinExp(dataCount), dataFactorial(dataFactorialCount);
    for (int i = 0; i < dataCount; ++i) {
        dataLinear[i].key = i / 10.0;
        dataLinear[i].value = dataLinear[i].key;
        dataMinusSinExp[i].key = i / 10.0;
        dataMinusSinExp[i].value = -qSin(dataMinusSinExp[i].key) * qExp(dataMinusSinExp[i].key);
        dataPlusSinExp[i].key = i / 10.0;
        dataPlusSinExp[i].value = qSin(dataPlusSinExp[i].key) * qExp(dataPlusSinExp[i].key);
    }

    for (int i = 0; i < dataFactorialCount; ++i) {
        dataFactorial[i].key = i;
        dataFactorial[i].value = 1.0;
        for (int k = 1; k <= i; ++k) {
            dataFactorial[i].value *= k;    // factorial
        }
    }

    ui->customPlot->graph(0)->data()->set(dataLinear);
    ui->customPlot->graph(1)->data()->set(dataMinusSinExp);
    ui->customPlot->graph(2)->data()->set(dataPlusSinExp);
    ui->customPlot->graph(3)->data()->set(dataFactorial);

    ui->customPlot->yAxis->grid()->setSubGridVisible(true);
    ui->customPlot->xAxis->grid()->setSubGridVisible(true);
    ui->customPlot->yAxis->setScaleType(QCPAxis::stLogarithmic);
    ui->customPlot->yAxis2->setScaleType(QCPAxis::stLogarithmic);

    QSharedPointer<QCPAxisTickerLog> logTicker(new QCPAxisTickerLog);
    ui->customPlot->yAxis->setTicker(logTicker);
    ui->customPlot->yAxis2->setTicker(logTicker);
    ui->customPlot->yAxis->setNumberFormat("eb"); // e = exponential, b = beautiful decimal powers
    ui->customPlot->yAxis->setNumberPrecision(0); // makes sure "1*10^4" is displayed only as "10^4"
    ui->customPlot->xAxis->setRange(0, 19.9);
    ui->customPlot->yAxis->setRange(1e-2, 1e10);    

    // make top right axes clones of bottom left axes:
    ui->customPlot->axisRect()->setupFullAxesBox();
    // connect signals so top and right axes move in sync with bottom and left axes:
    connect(ui->customPlot->xAxis, SIGNAL(rangeChanged(QCPRange)), ui->customPlot->xAxis2, SLOT(setRange(QCPRange)));
    connect(ui->customPlot->yAxis, SIGNAL(rangeChanged(QCPRange)), ui->customPlot->yAxis2, SLOT(setRange(QCPRange)));

    ui->customPlot->legend->setVisible(true);
    ui->customPlot->legend->setBrush(QBrush(QColor(255, 255, 255, 150)));
    ui->customPlot->axisRect()->insetLayout()->setInsetAlignment(0, Qt::AlignLeft | Qt::AlignTop); // make legend align in top left corner or axis rect
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmlogarithmic.h`

```cpp
﻿#ifndef FRMLOGARITHMIC_H
#define FRMLOGARITHMIC_H

#include <QWidget>

namespace Ui {
class frmLogarithmic;
}

class frmLogarithmic : public QWidget
{
    Q_OBJECT

public:
    explicit frmLogarithmic(QWidget *parent = 0);
    ~frmLogarithmic();

private:
    Ui::frmLogarithmic *ui;

private slots:
    void initForm();
};

#endif // FRMLOGARITHMIC_H
```

### `frmexample/frmmultiaxis.cpp`

```cpp
﻿#include "frmmultiaxis.h"
#include "ui_frmmultiaxis.h"
#include "qdebug.h"

frmMultiAxis::frmMultiAxis(QWidget *parent) : QWidget(parent), ui(new Ui::frmMultiAxis)
{
    ui->setupUi(this);
    this->initForm();
}

frmMultiAxis::~frmMultiAxis()
{
    delete ui;
}

void frmMultiAxis::initForm()
{
    ui->customPlot->setLocale(QLocale(QLocale::English, QLocale::UnitedKingdom)); // period as decimal separator and comma as thousand separator
    ui->customPlot->legend->setVisible(true);
    QFont legendFont = font();  // start out with MainWindow's font..
    legendFont.setPointSize(9); // and make a bit smaller for legend
    ui->customPlot->legend->setFont(legendFont);
    ui->customPlot->legend->setBrush(QBrush(QColor(255, 255, 255, 230)));
    // by default, the legend is in the inset layout of the main axis rect. So this is how we access it to change legend placement:
    ui->customPlot->axisRect()->insetLayout()->setInsetAlignment(0, Qt::AlignBottom | Qt::AlignRight);

    // setup for graph 0: key axis left, value axis bottom
    // will contain left maxwell-like function
    ui->customPlot->addGraph(ui->customPlot->yAxis, ui->customPlot->xAxis);
    ui->customPlot->graph(0)->setPen(QPen(QColor(255, 100, 0)));
    ui->customPlot->graph(0)->setBrush(QBrush(QPixmap("./balboa.jpg"))); // fill with texture of specified image
    ui->customPlot->graph(0)->setLineStyle(QCPGraph::lsLine);
    ui->customPlot->graph(0)->setScatterStyle(QCPScatterStyle(QCPScatterStyle::ssDisc, 5));
    ui->customPlot->graph(0)->setName("Left maxwell function");

    // setup for graph 1: key axis bottom, value axis left (those are the default axes)
    // will contain bottom maxwell-like function with error bars
    ui->customPlot->addGraph();
    ui->customPlot->graph(1)->setPen(QPen(Qt::red));
    ui->customPlot->graph(1)->setBrush(QBrush(QPixmap("./balboa.jpg"))); // same fill as we used for graph 0
    ui->customPlot->graph(1)->setLineStyle(QCPGraph::lsStepCenter);
    ui->customPlot->graph(1)->setScatterStyle(QCPScatterStyle(QCPScatterStyle::ssCircle, Qt::red, Qt::white, 7));
    ui->customPlot->graph(1)->setName("Bottom maxwell function");

    QCPErrorBars *errorBars = new QCPErrorBars(ui->customPlot->xAxis, ui->customPlot->yAxis);
    errorBars->removeFromLegend();
    errorBars->setDataPlottable(ui->customPlot->graph(1));

    // setup for graph 2: key axis top, value axis right
    // will contain high frequency sine with low frequency beating:
    ui->customPlot->addGraph(ui->customPlot->xAxis2, ui->customPlot->yAxis2);
    ui->customPlot->graph(2)->setPen(QPen(Qt::blue));
    ui->customPlot->graph(2)->setName("High frequency sine");

    // setup for graph 3: same axes as graph 2
    // will contain low frequency beating envelope of graph 2
    ui->customPlot->addGraph(ui->customPlot->xAxis2, ui->customPlot->yAxis2);
    QPen blueDotPen;
    blueDotPen.setColor(QColor(30, 40, 255, 150));
    blueDotPen.setStyle(Qt::DotLine);
    blueDotPen.setWidthF(4);
    ui->customPlot->graph(3)->setPen(blueDotPen);
    ui->customPlot->graph(3)->setName("Sine envelope");

    // setup for graph 4: key axis right, value axis top
    // will contain parabolically distributed data points with some random perturbance
    ui->customPlot->addGraph(ui->customPlot->yAxis2, ui->customPlot->xAxis2);
    ui->customPlot->graph(4)->setPen(QColor(50, 50, 50, 255));
    ui->customPlot->graph(4)->setLineStyle(QCPGraph::lsNone);
    ui->customPlot->graph(4)->setScatterStyle(QCPScatterStyle(QCPScatterStyle::ssCircle, 4));
    ui->customPlot->graph(4)->setName("Some random data around\na quadratic function");

    // generate data, just playing with numbers, not much to learn here:
    QVector<double> x0(25), y0(25);
    QVector<double> x1(15), y1(15), y1err(15);
    QVector<double> x2(250), y2(250);
    QVector<double> x3(250), y3(250);
    QVector<double> x4(250), y4(250);

    for (int i = 0; i < 25; ++i) { // data for graph 0
        x0[i] = 3 * i / 25.0;
        y0[i] = qExp(-x0[i] * x0[i] * 0.8) * (x0[i] * x0[i] + x0[i]);
    }

    for (int i = 0; i < 15; ++i) { // data for graph 1
        x1[i] = 3 * i / 15.0;;
        y1[i] = qExp(-x1[i] * x1[i]) * (x1[i] * x1[i]) * 2.6;
        y1err[i] = y1[i] * 0.25;
    }

    for (int i = 0; i < 250; ++i) { // data for graphs 2, 3 and 4
        x2[i] = i / 250.0 * 3 * M_PI;
        x3[i] = x2[i];
        x4[i] = i / 250.0 * 100 - 50;
        y2[i] = qSin(x2[i] * 12) * qCos(x2[i]) * 10;
        y3[i] = qCos(x3[i]) * 10;
        y4[i] = 0.01 * x4[i] * x4[i] + 1.5 * (rand() / (double)RAND_MAX - 0.5) + 1.5 * M_PI;
    }

    // pass data points to graphs:
    ui->customPlot->graph(0)->setData(x0, y0);
    ui->customPlot->graph(1)->setData(x1, y1);
    errorBars->setData(y1err);
    ui->customPlot->graph(2)->setData(x2, y2);
    ui->customPlot->graph(3)->setData(x3, y3);
    ui->customPlot->graph(4)->setData(x4, y4);

    // activate top and right axes, which are invisible by default:
    ui->customPlot->xAxis2->setVisible(true);
    ui->customPlot->yAxis2->setVisible(true);

    // set ranges appropriate to show data:
    ui->customPlot->xAxis->setRange(0, 2.7);
    ui->customPlot->yAxis->setRange(0, 2.6);
    ui->customPlot->xAxis2->setRange(0, 3.0 * M_PI);
    ui->customPlot->yAxis2->setRange(-70, 35);

    // set pi ticks on top axis:
    ui->customPlot->xAxis2->setTicker(QSharedPointer<QCPAxisTickerPi>(new QCPAxisTickerPi));
    // add title layout element:
    ui->customPlot->plotLayout()->insertRow(0);
    ui->customPlot->plotLayout()->addElement(0, 0, new QCPTextElement(ui->customPlot, "Way too many graphs in one plot", QFont("sans", 12, QFont::Bold)));

    // set labels:
    ui->customPlot->xAxis->setLabel("Bottom axis with outward ticks");
    ui->customPlot->yAxis->setLabel("Left axis label");
    ui->customPlot->xAxis2->setLabel("Top axis label");
    ui->customPlot->yAxis2->setLabel("Right axis label");
    // make ticks on bottom axis go outward:
    ui->customPlot->xAxis->setTickLength(0, 5);
    ui->customPlot->xAxis->setSubTickLength(0, 3);
    // make ticks on right axis go inward and outward:
    ui->customPlot->yAxis2->setTickLength(3, 3);
    ui->customPlot->yAxis2->setSubTickLength(1, 1);
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmmultiaxis.h`

```cpp
﻿#ifndef FRMMULTIAXIS_H
#define FRMMULTIAXIS_H

#include <QWidget>

namespace Ui {
class frmMultiAxis;
}

class frmMultiAxis : public QWidget
{
    Q_OBJECT

public:
    explicit frmMultiAxis(QWidget *parent = 0);
    ~frmMultiAxis();

private:
    Ui::frmMultiAxis *ui;

private slots:
    void initForm();
};

#endif // FRMMULTIAXIS_H
```

### `frmexample/frmparametriccurve.cpp`

```cpp
﻿#include "frmparametriccurve.h"
#include "ui_frmparametriccurve.h"
#include "qdebug.h"

frmParametricCurve::frmParametricCurve(QWidget *parent) : QWidget(parent), ui(new Ui::frmParametricCurve)
{
    ui->setupUi(this);
    this->initForm();
}

frmParametricCurve::~frmParametricCurve()
{
    delete ui;
}

void frmParametricCurve::initForm()
{
    // create empty curve objects:
    QCPCurve *fermatSpiral1 = new QCPCurve(ui->customPlot->xAxis, ui->customPlot->yAxis);
    QCPCurve *fermatSpiral2 = new QCPCurve(ui->customPlot->xAxis, ui->customPlot->yAxis);
    QCPCurve *deltoidRadial = new QCPCurve(ui->customPlot->xAxis, ui->customPlot->yAxis);

    // generate the curve data points:
    const int pointCount = 500;
    QVector<QCPCurveData> dataSpiral1(pointCount), dataSpiral2(pointCount), dataDeltoid(pointCount);
    for (int i = 0; i < pointCount; ++i) {
        double phi = i / (double)(pointCount - 1) * 8 * M_PI;
        double theta = i / (double)(pointCount - 1) * 2 * M_PI;
        dataSpiral1[i] = QCPCurveData(i, qSqrt(phi) * qCos(phi), qSqrt(phi) * qSin(phi));
        dataSpiral2[i] = QCPCurveData(i, -dataSpiral1[i].key, -dataSpiral1[i].value);
        dataDeltoid[i] = QCPCurveData(i, 2 * qCos(2 * theta) + qCos(1 * theta) + 2 * qSin(theta), 2 * qSin(2 * theta) - qSin(1 * theta));
    }

    // pass the data to the curves; we know t (i in loop above) is ascending, so set alreadySorted=true (saves an extra internal sort):
    fermatSpiral1->data()->set(dataSpiral1, true);
    fermatSpiral2->data()->set(dataSpiral2, true);
    deltoidRadial->data()->set(dataDeltoid, true);

    // color the curves:
    fermatSpiral1->setPen(QPen(Qt::blue));
    fermatSpiral1->setBrush(QBrush(QColor(0, 0, 255, 20)));
    fermatSpiral2->setPen(QPen(QColor(255, 120, 0)));
    fermatSpiral2->setBrush(QBrush(QColor(255, 120, 0, 30)));

    QRadialGradient radialGrad(QPointF(310, 180), 200);
    radialGrad.setColorAt(0, QColor(170, 20, 240, 100));
    radialGrad.setColorAt(0.5, QColor(20, 10, 255, 40));
    radialGrad.setColorAt(1, QColor(120, 20, 240, 10));
    deltoidRadial->setPen(QPen(QColor(170, 20, 240)));
    deltoidRadial->setBrush(QBrush(radialGrad));

    // set some basic ui->customPlot config:
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom | QCP::iSelectPlottables);
    ui->customPlot->axisRect()->setupFullAxesBox();
    ui->customPlot->rescaleAxes();
}
```

### `frmexample/frmparametriccurve.h`

```cpp
﻿#ifndef FRMPARAMETRICCURVE_H
#define FRMPARAMETRICCURVE_H

#include <QWidget>

namespace Ui {
class frmParametricCurve;
}

class frmParametricCurve : public QWidget
{
    Q_OBJECT

public:
    explicit frmParametricCurve(QWidget *parent = 0);
    ~frmParametricCurve();

private:
    Ui::frmParametricCurve *ui;

private slots:
    void initForm();
};

#endif // FRMPARAMETRICCURVE_H
```

### `frmexample/frmpolarplot.cpp`

```cpp
﻿#include "frmpolarplot.h"
#include "ui_frmpolarplot.h"
#include "qdebug.h"

frmPolarPlot::frmPolarPlot(QWidget *parent) : QWidget(parent), ui(new Ui::frmPolarPlot)
{
    ui->setupUi(this);
    this->initForm();
}

frmPolarPlot::~frmPolarPlot()
{
    delete ui;
}

void frmPolarPlot::initForm()
{
    //南丁格尔图示 2.1 版本新增加的
#ifdef qcustomplot_v2_1
    // Warning: Polar plots are a still a tech preview

    ui->customPlot->plotLayout()->clear();
    QCPPolarAxisAngular *angularAxis = new QCPPolarAxisAngular(ui->customPlot);
    ui->customPlot->plotLayout()->addElement(0, 0, angularAxis);
    /* This is how we could set the angular axis to show pi symbols instead of degree numbers:
    QSharedPointer<QCPAxisTickerPi> ticker(new QCPAxisTickerPi);
    ticker->setPiValue(180);
    ticker->setTickCount(8);
    polarAxis->setTicker(ticker);
    */

    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
    angularAxis->setRangeDrag(false);
    angularAxis->setTickLabelMode(QCPPolarAxisAngular::lmUpright);

    angularAxis->radialAxis()->setTickLabelMode(QCPPolarAxisRadial::lmUpright);
    angularAxis->radialAxis()->setTickLabelRotation(0);
    angularAxis->radialAxis()->setAngle(45);

    angularAxis->grid()->setAngularPen(QPen(QColor(200, 200, 200), 0, Qt::SolidLine));
    angularAxis->grid()->setSubGridType(QCPPolarGrid::gtAll);

    QCPPolarGraph *g1 = new QCPPolarGraph(angularAxis, angularAxis->radialAxis());
    QCPPolarGraph *g2 = new QCPPolarGraph(angularAxis, angularAxis->radialAxis());
    g2->setPen(QPen(QColor(255, 150, 20)));
    g2->setBrush(QColor(255, 150, 20, 50));
    g1->setScatterStyle(QCPScatterStyle::ssDisc);

    for (int i = 0; i < 100; ++i) {
        g1->addData(i / 100.0 * 360.0, qSin(i / 100.0 * M_PI * 8) * 8 + 1);
        g2->addData(i / 100.0 * 360.0, qSin(i / 100.0 * M_PI * 6) * 2);
    }

    angularAxis->setRange(0, 360);
    angularAxis->radialAxis()->setRange(-10, 10);
#endif
}
```

### `frmexample/frmpolarplot.h`

```cpp
﻿#ifndef FRMPOLARPLOT_H
#define FRMPOLARPLOT_H

#include <QWidget>

namespace Ui {
class frmPolarPlot;
}

class frmPolarPlot : public QWidget
{
    Q_OBJECT

public:
    explicit frmPolarPlot(QWidget *parent = 0);
    ~frmPolarPlot();

private:
    Ui::frmPolarPlot *ui;

private slots:
    void initForm();
};

#endif // FRMPOLARPLOT_H
```

### `frmexample/frmquadratic.cpp`

```cpp
﻿#include "frmquadratic.h"
#include "ui_frmquadratic.h"
#include "qdebug.h"

frmQuadratic::frmQuadratic(QWidget *parent) : QWidget(parent), ui(new Ui::frmQuadratic)
{
    ui->setupUi(this);
    this->initForm();
}

frmQuadratic::~frmQuadratic()
{
    delete ui;
}

void frmQuadratic::initForm()
{
    // generate some data:
    QVector<double> x(101), y(101); // initialize with entries 0..100
    for (int i = 0; i < 101; ++i) {
        x[i] = i / 50.0 - 1; // x goes from -1 to 1
        y[i] = x[i] * x[i]; // let's plot a quadratic function
    }

    // create graph and assign data to it:
    ui->customPlot->addGraph();
    ui->customPlot->graph(0)->setData(x, y);
    // give the axes some labels:
    ui->customPlot->xAxis->setLabel("x");
    ui->customPlot->yAxis->setLabel("y");
    // set axes ranges, so we see all data:
    ui->customPlot->xAxis->setRange(-1, 1);
    ui->customPlot->yAxis->setRange(0, 1);
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmquadratic.h`

```cpp
﻿#ifndef FRMQUADRATIC_H
#define FRMQUADRATIC_H

#include <QWidget>

namespace Ui {
class frmQuadratic;
}

class frmQuadratic : public QWidget
{
    Q_OBJECT

public:
    explicit frmQuadratic(QWidget *parent = 0);
    ~frmQuadratic();

private:
    Ui::frmQuadratic *ui;

private slots:
    void initForm();
};

#endif // FRMQUADRATIC_H
```

### `frmexample/frmrealtimedata.cpp`

```cpp
﻿#include "frmrealtimedata.h"
#include "ui_frmrealtimedata.h"
#include "qdebug.h"

frmRealtimeData::frmRealtimeData(QWidget *parent) : QWidget(parent), ui(new Ui::frmRealtimeData)
{
    ui->setupUi(this);
    this->initForm();
}

frmRealtimeData::~frmRealtimeData()
{
    delete ui;
}

void frmRealtimeData::showEvent(QShowEvent *)
{
    timeStart = QTime::currentTime();
    dataTimer.start(0);
}

void frmRealtimeData::hideEvent(QHideEvent *)
{
    dataTimer.stop();
}

void frmRealtimeData::initForm()
{
    // include this section to fully disable antialiasing for higher performance:
    /*
    ui->customPlot->setNotAntialiasedElements(QCP::aeAll);
    QFont font;
    font.setStyleStrategy(QFont::NoAntialias);
    ui->customPlot->xAxis->setTickLabelFont(font);
    ui->customPlot->yAxis->setTickLabelFont(font);
    ui->customPlot->legend->setFont(font);
    */

    ui->customPlot->addGraph(); // blue line
    ui->customPlot->graph(0)->setPen(QPen(QColor(40, 110, 255)));
    ui->customPlot->addGraph(); // red line
    ui->customPlot->graph(1)->setPen(QPen(QColor(255, 110, 40)));

    QSharedPointer<QCPAxisTickerTime> timeTicker(new QCPAxisTickerTime);
    timeTicker->setTimeFormat("%h:%m:%s");
    ui->customPlot->xAxis->setTicker(timeTicker);
    ui->customPlot->axisRect()->setupFullAxesBox();
    ui->customPlot->yAxis->setRange(-1.2, 1.2);

    // make left and bottom axes transfer their ranges to right and top axes:
    connect(ui->customPlot->xAxis, SIGNAL(rangeChanged(QCPRange)), ui->customPlot->xAxis2, SLOT(setRange(QCPRange)));
    connect(ui->customPlot->yAxis, SIGNAL(rangeChanged(QCPRange)), ui->customPlot->yAxis2, SLOT(setRange(QCPRange)));

    // setup a timer that repeatedly calls MainWindow::realtimeDataSlot:
    connect(&dataTimer, SIGNAL(timeout()), this, SLOT(realtimeDataSlot()));
}

void frmRealtimeData::realtimeDataSlot()
{
    // calculate two new data points:
    double key = timeStart.msecsTo(QTime::currentTime()) / 1000.0; // time elapsed since start of demo, in seconds
    static double lastPointKey = 0;
    if (key - lastPointKey > 0.002) { // at most add point every 2 ms
        // add data to lines:
        ui->customPlot->graph(0)->addData(key, qSin(key) + std::rand() / (double)RAND_MAX * 1 * qSin(key / 0.3843));
        ui->customPlot->graph(1)->addData(key, qCos(key) + std::rand() / (double)RAND_MAX * 0.5 * qSin(key / 0.4364));
        // rescale value (vertical) axis to fit the current data:
        //ui->customPlot->graph(0)->rescaleValueAxis();
        //ui->customPlot->graph(1)->rescaleValueAxis(true);
        lastPointKey = key;
    }

    // make key axis range scroll with the data (at a constant range size of 8):
    ui->customPlot->xAxis->setRange(key, 8, Qt::AlignRight);
    ui->customPlot->replot();

    // calculate frames per second:
    static double lastFpsKey;
    static int frameCount;
    ++frameCount;
    if (key - lastFpsKey > 2) { // average fps over 2 seconds
        QString fps = QString("%1").arg(frameCount / (key - lastFpsKey), 0, 'f', 0);
        int count = ui->customPlot->graph(0)->data()->size() + ui->customPlot->graph(1)->data()->size();
        QString info = QString("%1 FPS, Total Data points: %2").arg(fps).arg(count);
        ui->label->setText(info);
        lastFpsKey = key;
        frameCount = 0;
    }
}
```

### `frmexample/frmrealtimedata.h`

```cpp
﻿#ifndef FRMREALTIMEDATA_H
#define FRMREALTIMEDATA_H

#include <QWidget>
#include <QDateTime>
#include <QTimer>

namespace Ui {
class frmRealtimeData;
}

class frmRealtimeData : public QWidget
{
    Q_OBJECT

public:
    explicit frmRealtimeData(QWidget *parent = 0);
    ~frmRealtimeData();

protected:
    void showEvent(QShowEvent *);
    void hideEvent(QHideEvent *);

private:
    Ui::frmRealtimeData *ui;
    QTimer dataTimer;
    QTime timeStart;

private slots:
    void initForm();
    void realtimeDataSlot();
};

#endif // FRMREALTIMEDATA_H
```

### `frmexample/frmscatterpixmap.cpp`

```cpp
﻿#include "frmscatterpixmap.h"
#include "ui_frmscatterpixmap.h"
#include "qdebug.h"

frmScatterPixmap::frmScatterPixmap(QWidget *parent) : QWidget(parent), ui(new Ui::frmScatterPixmap)
{
    ui->setupUi(this);
    this->initForm();
}

frmScatterPixmap::~frmScatterPixmap()
{
    delete ui;
}

void frmScatterPixmap::initForm()
{
    ui->customPlot->axisRect()->setBackground(QPixmap(":/image/bg1.jpg"));
    ui->customPlot->addGraph();
    ui->customPlot->graph()->setLineStyle(QCPGraph::lsLine);

    QPen pen;
    pen.setColor(QColor(255, 200, 20, 200));
    pen.setStyle(Qt::DashLine);
    pen.setWidthF(2.5);
    ui->customPlot->graph()->setPen(pen);
    ui->customPlot->graph()->setBrush(QBrush(QColor(255, 200, 20, 70)));
    ui->customPlot->graph()->setScatterStyle(QCPScatterStyle(QPixmap(":/image/data.png")));
    // set graph name, will show up in legend next to icon:
    ui->customPlot->graph()->setName("Data from Photovoltaic\nenergy barometer 2011");

    // set data:
    QVector<double> year, value;
    year  << 2005 << 2006 << 2007 << 2008  << 2009  << 2010 << 2011;
    value << 2.17 << 3.42 << 4.94 << 10.38 << 15.86 << 29.33 << 52.1;
    ui->customPlot->graph()->setData(year, value);

    // set title of plot:
    ui->customPlot->plotLayout()->insertRow(0);
    ui->customPlot->plotLayout()->addElement(0, 0, new QCPTextElement(ui->customPlot, "Regenerative Energies", QFont("sans", 12, QFont::Bold)));
    // axis configurations:
    ui->customPlot->xAxis->setLabel("Year");
    ui->customPlot->yAxis->setLabel("Installed Gigawatts of\nphotovoltaic in the European Union");
    ui->customPlot->xAxis2->setVisible(true);
    ui->customPlot->yAxis2->setVisible(true);
    ui->customPlot->xAxis2->setTickLabels(false);
    ui->customPlot->yAxis2->setTickLabels(false);
    ui->customPlot->xAxis2->setTicks(false);
    ui->customPlot->yAxis2->setTicks(false);
    ui->customPlot->xAxis2->setSubTicks(false);
    ui->customPlot->yAxis2->setSubTicks(false);
    ui->customPlot->xAxis->setRange(2004.5, 2011.5);
    ui->customPlot->yAxis->setRange(0, 52);

    // setup legend:
    ui->customPlot->legend->setFont(QFont(font().family(), 7));
    ui->customPlot->legend->setIconSize(50, 20);
    ui->customPlot->legend->setVisible(true);
    ui->customPlot->axisRect()->insetLayout()->setInsetAlignment(0, Qt::AlignLeft | Qt::AlignTop);
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmscatterpixmap.h`

```cpp
﻿#ifndef FRMSCATTERPIXMAP_H
#define FRMSCATTERPIXMAP_H

#include <QWidget>

namespace Ui {
class frmScatterPixmap;
}

class frmScatterPixmap : public QWidget
{
    Q_OBJECT

public:
    explicit frmScatterPixmap(QWidget *parent = 0);
    ~frmScatterPixmap();

private:
    Ui::frmScatterPixmap *ui;

private slots:
    void initForm();
};

#endif // FRMSCATTERPIXMAP_H
```

### `frmexample/frmscatterstyle.cpp`

```cpp
﻿#include "frmscatterstyle.h"
#include "ui_frmscatterstyle.h"
#include "qmetaobject.h"
#include "qdebug.h"

frmScatterStyle::frmScatterStyle(QWidget *parent) : QWidget(parent), ui(new Ui::frmScatterStyle)
{
    ui->setupUi(this);
    this->initForm();
}

frmScatterStyle::~frmScatterStyle()
{
    delete ui;
}

void frmScatterStyle::initForm()
{
    ui->customPlot->legend->setVisible(true);
    ui->customPlot->legend->setFont(QFont("Helvetica", 9));
    ui->customPlot->legend->setRowSpacing(-3);

    QVector<QCPScatterStyle::ScatterShape> shapes;
    shapes << QCPScatterStyle::ssCross;
    shapes << QCPScatterStyle::ssPlus;
    shapes << QCPScatterStyle::ssCircle;
    shapes << QCPScatterStyle::ssDisc;
    shapes << QCPScatterStyle::ssSquare;
    shapes << QCPScatterStyle::ssDiamond;
    shapes << QCPScatterStyle::ssStar;
    shapes << QCPScatterStyle::ssTriangle;
    shapes << QCPScatterStyle::ssTriangleInverted;
    shapes << QCPScatterStyle::ssCrossSquare;
    shapes << QCPScatterStyle::ssPlusSquare;
    shapes << QCPScatterStyle::ssCrossCircle;
    shapes << QCPScatterStyle::ssPlusCircle;
    shapes << QCPScatterStyle::ssPeace;
    shapes << QCPScatterStyle::ssCustom;

    QPen pen;
    // add graphs with different scatter styles:
    for (int i = 0; i < shapes.size(); ++i) {
        ui->customPlot->addGraph();
        pen.setColor(QColor(qSin(i * 0.3) * 100 + 100, qSin(i * 0.6 + 0.7) * 100 + 100, qSin(i * 0.4 + 0.6) * 100 + 100));
        // generate data:
        QVector<double> x(10), y(10);
        for (int k = 0; k < 10; ++k) {
            x[k] = k / 10.0 * 4 * 3.14 + 0.01;
            y[k] = 7 * qSin(x[k]) / x[k] + (shapes.size() - i) * 5;
        }
        ui->customPlot->graph()->setData(x, y);
        ui->customPlot->graph()->rescaleAxes(true);
        ui->customPlot->graph()->setPen(pen);

        //枚举值转字符串
        QMetaObject metaObject = QCPScatterStyle::staticMetaObject;
        QMetaEnum metaEnum = metaObject.enumerator(metaObject.indexOfEnumerator("ScatterShape"));
        QString name = metaEnum.valueToKey(shapes.at(i));
        ui->customPlot->graph()->setName(name);

        ui->customPlot->graph()->setLineStyle(QCPGraph::lsLine);
        // set scatter style:
        if (shapes.at(i) != QCPScatterStyle::ssCustom) {
            ui->customPlot->graph()->setScatterStyle(QCPScatterStyle(shapes.at(i), 10));
        } else {
            QPainterPath customScatterPath;
            for (int i = 0; i < 3; ++i) {
                customScatterPath.cubicTo(qCos(2 * M_PI * i / 3.0) * 9, qSin(2 * M_PI * i / 3.0) * 9, qCos(2 * M_PI * (i + 0.9) / 3.0) * 9, qSin(2 * M_PI * (i + 0.9) / 3.0) * 9, 0, 0);
            }
            ui->customPlot->graph()->setScatterStyle(QCPScatterStyle(customScatterPath, QPen(Qt::black, 0), QColor(40, 70, 255, 50), 10));
        }
    }

    // set blank axis lines:
    ui->customPlot->rescaleAxes();
    ui->customPlot->xAxis->setTicks(false);
    ui->customPlot->yAxis->setTicks(false);
    ui->customPlot->xAxis->setTickLabels(false);
    ui->customPlot->yAxis->setTickLabels(false);
    // make top right axes clones of bottom left axes:
    ui->customPlot->axisRect()->setupFullAxesBox();
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmscatterstyle.h`

```cpp
﻿#ifndef FRMSCATTERSTYLE_H
#define FRMSCATTERSTYLE_H

#include <QWidget>

namespace Ui {
class frmScatterStyle;
}

class frmScatterStyle : public QWidget
{
    Q_OBJECT

public:
    explicit frmScatterStyle(QWidget *parent = 0);
    ~frmScatterStyle();

private:
    Ui::frmScatterStyle *ui;

private slots:
    void initForm();
};

#endif // FRMSCATTERSTYLE_H
```

### `frmexample/frmsimple.cpp`

```cpp
﻿#include "frmsimple.h"
#include "ui_frmsimple.h"
#include "qdebug.h"

frmSimple::frmSimple(QWidget *parent) : QWidget(parent), ui(new Ui::frmSimple)
{
    ui->setupUi(this);
    this->initForm();
}

frmSimple::~frmSimple()
{
    delete ui;
}

void frmSimple::initForm()
{
    // add two new graphs and set their look:
    ui->customPlot->addGraph();
    ui->customPlot->graph(0)->setPen(QPen(Qt::blue)); // line color blue for first graph
    ui->customPlot->graph(0)->setBrush(QBrush(QColor(0, 0, 255, 20))); // first graph will be filled with translucent blue
    ui->customPlot->addGraph();
    ui->customPlot->graph(1)->setPen(QPen(Qt::red)); // line color red for second graph

    // generate some points of data (y0 for first, y1 for second graph):
    QVector<double> x(251), y0(251), y1(251);
    for (int i = 0; i < 251; ++i) {
        x[i] = i;
        y0[i] = qExp(-i / 150.0) * qCos(i / 10.0); // exponentially decaying cosine
        y1[i] = qExp(-i / 150.0);            // exponential envelope
    }

    // configure right and top axis to show ticks but no labels:
    // (see QCPAxisRect::setupFullAxesBox for a quicker method to do this)
    ui->customPlot->xAxis2->setVisible(true);
    ui->customPlot->xAxis2->setTickLabels(false);
    ui->customPlot->yAxis2->setVisible(true);
    ui->customPlot->yAxis2->setTickLabels(false);

    // make left and bottom axes always transfer their ranges to right and top axes:
    connect(ui->customPlot->xAxis, SIGNAL(rangeChanged(QCPRange)), ui->customPlot->xAxis2, SLOT(setRange(QCPRange)));
    connect(ui->customPlot->yAxis, SIGNAL(rangeChanged(QCPRange)), ui->customPlot->yAxis2, SLOT(setRange(QCPRange)));

    // pass data points to graphs:
    ui->customPlot->graph(0)->setData(x, y0);
    ui->customPlot->graph(1)->setData(x, y1);
    // let the ranges scale themselves so graph 0 fits perfectly in the visible area:
    ui->customPlot->graph(0)->rescaleAxes();
    // same thing for graph 1, but only enlarge ranges (in case graph 1 is smaller than graph 0):
    ui->customPlot->graph(1)->rescaleAxes(true);
    // Note: we could have also just called customPlot->rescaleAxes(); instead
    // Allow user to drag axis ranges with mouse, zoom with mouse wheel and select graphs by clicking:
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom | QCP::iSelectPlottables);
}
```

### `frmexample/frmsimple.h`

```cpp
﻿#ifndef FRMSIMPLE_H
#define FRMSIMPLE_H

#include <QWidget>

namespace Ui {
class frmSimple;
}

class frmSimple : public QWidget
{
    Q_OBJECT

public:
    explicit frmSimple(QWidget *parent = 0);
    ~frmSimple();

private:
    Ui::frmSimple *ui;

private slots:
    void initForm();
};

#endif // FRMSIMPLE_H
```

### `frmexample/frmsimpleitem.cpp`

```cpp
﻿#include "frmsimpleitem.h"
#include "ui_frmsimpleitem.h"
#include "qdebug.h"

frmSimpleItem::frmSimpleItem(QWidget *parent) : QWidget(parent), ui(new Ui::frmSimpleItem)
{
    ui->setupUi(this);
    this->initForm();
}

frmSimpleItem::~frmSimpleItem()
{
    delete ui;
}

void frmSimpleItem::initForm()
{
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);

    // add the text label at the top:
    QCPItemText *textLabel = new QCPItemText(ui->customPlot);
    textLabel->setPositionAlignment(Qt::AlignTop|Qt::AlignHCenter);
    textLabel->position->setType(QCPItemPosition::ptAxisRectRatio);
    textLabel->position->setCoords(0.5, 0); // place position at center/top of axis rect
    textLabel->setText("Text Item Demo");
    textLabel->setFont(QFont(font().family(), 16)); // make font a bit larger
    textLabel->setPen(QPen(Qt::black)); // show black border around text

    // add the arrow:
    QCPItemLine *arrow = new QCPItemLine(ui->customPlot);
    arrow->start->setParentAnchor(textLabel->bottom);
    arrow->end->setCoords(4, 1.6); // point to (4, 1.6) in x-y-plot coordinates
    arrow->setHead(QCPLineEnding::esSpikeArrow);
}
```

### `frmexample/frmsimpleitem.h`

```cpp
﻿#ifndef FRMSIMPLEITEM_H
#define FRMSIMPLEITEM_H

#include <QWidget>

namespace Ui {
class frmSimpleItem;
}

class frmSimpleItem : public QWidget
{
    Q_OBJECT

public:
    explicit frmSimpleItem(QWidget *parent = 0);
    ~frmSimpleItem();

private:
    Ui::frmSimpleItem *ui;

private slots:
    void initForm();
};

#endif // FRMSIMPLEITEM_H
```

### `frmexample/frmsincscatter.cpp`

```cpp
﻿#include "frmsincscatter.h"
#include "ui_frmsincscatter.h"
#include "qdebug.h"

frmSincScatter::frmSincScatter(QWidget *parent) : QWidget(parent), ui(new Ui::frmSincScatter)
{
    ui->setupUi(this);
    this->initForm();
}

frmSincScatter::~frmSincScatter()
{
    delete ui;
}

void frmSincScatter::initForm()
{
    ui->customPlot->legend->setVisible(true);
    ui->customPlot->legend->setFont(QFont("Helvetica", 9));
    // set locale to english, so we get english decimal separator:
    ui->customPlot->setLocale(QLocale(QLocale::English, QLocale::UnitedKingdom));
    // add confidence band graphs:
    ui->customPlot->addGraph();

    QPen pen;
    pen.setStyle(Qt::DotLine);
    pen.setWidth(1);
    pen.setColor(QColor(180, 180, 180));
    ui->customPlot->graph(0)->setName("Confidence Band 68%");
    ui->customPlot->graph(0)->setPen(pen);
    ui->customPlot->graph(0)->setBrush(QBrush(QColor(255, 50, 30, 20)));
    ui->customPlot->addGraph();
    ui->customPlot->legend->removeItem(ui->customPlot->legend->itemCount() - 1); // don't show two confidence band graphs in legend
    ui->customPlot->graph(1)->setPen(pen);
    ui->customPlot->graph(0)->setChannelFillGraph(ui->customPlot->graph(1));

    // add theory curve graph:
    ui->customPlot->addGraph();
    pen.setStyle(Qt::DashLine);
    pen.setWidth(2);
    pen.setColor(Qt::red);
    ui->customPlot->graph(2)->setPen(pen);
    ui->customPlot->graph(2)->setName("Theory Curve");

    // add data point graph:
    ui->customPlot->addGraph();
    ui->customPlot->graph(3)->setPen(QPen(Qt::blue));
    ui->customPlot->graph(3)->setName("Measurement");
    ui->customPlot->graph(3)->setLineStyle(QCPGraph::lsNone);
    ui->customPlot->graph(3)->setScatterStyle(QCPScatterStyle(QCPScatterStyle::ssCross, 4));

    // add error bars:
    QCPErrorBars *errorBars = new QCPErrorBars(ui->customPlot->xAxis, ui->customPlot->yAxis);
    errorBars->removeFromLegend();
    errorBars->setAntialiased(false);
    errorBars->setDataPlottable(ui->customPlot->graph(3));
    errorBars->setPen(QPen(QColor(180, 180, 180)));

    // generate ideal sinc curve data and some randomly perturbed data for scatter plot:
    QVector<double> x0(250), y0(250);
    QVector<double> yConfUpper(250), yConfLower(250);
    for (int i = 0; i < 250; ++i) {
        x0[i] = (i / 249.0 - 0.5) * 30 + 0.01; // by adding a small offset we make sure not do divide by zero in next code line
        y0[i] = qSin(x0[i]) / x0[i]; // sinc function
        yConfUpper[i] = y0[i] + 0.15;
        yConfLower[i] = y0[i] - 0.15;
        x0[i] *= 1000;
    }

    QVector<double> x1(50), y1(50), y1err(50);
    for (int i = 0; i < 50; ++i) {
        // generate a gaussian distributed random number:
        double tmp1 = rand() / (double)RAND_MAX;
        double tmp2 = rand() / (double)RAND_MAX;
        double r = qSqrt(-2 * qLn(tmp1)) * qCos(2 * M_PI * tmp2); // box-muller transform for gaussian distribution
        // set y1 to value of y0 plus a random gaussian pertubation:
        x1[i] = (i / 50.0 - 0.5) * 30 + 0.25;
        y1[i] = qSin(x1[i]) / x1[i] + r * 0.15;
        x1[i] *= 1000;
        y1err[i] = 0.15;
    }

    // pass data to graphs and let Qui->customPlot determine the axes ranges so the whole thing is visible:
    ui->customPlot->graph(0)->setData(x0, yConfUpper);
    ui->customPlot->graph(1)->setData(x0, yConfLower);
    ui->customPlot->graph(2)->setData(x0, y0);
    ui->customPlot->graph(3)->setData(x1, y1);
    errorBars->setData(y1err);
    ui->customPlot->graph(2)->rescaleAxes();
    ui->customPlot->graph(3)->rescaleAxes(true);

    // setup look of bottom tick labels:
    ui->customPlot->xAxis->setTickLabelRotation(30);
    ui->customPlot->xAxis->ticker()->setTickCount(9);
    ui->customPlot->xAxis->setNumberFormat("ebc");
    ui->customPlot->xAxis->setNumberPrecision(1);
    ui->customPlot->xAxis->moveRange(-10);

    // make top right axes clones of bottom left axes. Looks prettier:
    ui->customPlot->axisRect()->setupFullAxesBox();
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmsincscatter.h`

```cpp
﻿#ifndef FRMSINCSCATTER_H
#define FRMSINCSCATTER_H

#include <QWidget>

namespace Ui {
class frmSincScatter;
}

class frmSincScatter : public QWidget
{
    Q_OBJECT

public:
    explicit frmSincScatter(QWidget *parent = 0);
    ~frmSincScatter();

private:
    Ui::frmSincScatter *ui;

private slots:
    void initForm();
};

#endif // FRMSINCSCATTER_H
```

### `frmexample/frmstatistical.cpp`

```cpp
﻿#include "frmstatistical.h"
#include "ui_frmstatistical.h"
#include "qdebug.h"

frmStatistical::frmStatistical(QWidget *parent) : QWidget(parent), ui(new Ui::frmStatistical)
{
    ui->setupUi(this);
    this->initForm();
}

frmStatistical::~frmStatistical()
{
    delete ui;
}

void frmStatistical::initForm()
{
    QCPStatisticalBox *statistical = new QCPStatisticalBox(ui->customPlot->xAxis, ui->customPlot->yAxis);
    QBrush boxBrush(QColor(60, 60, 255, 100));
    boxBrush.setStyle(Qt::Dense6Pattern); // make it look oldschool
    statistical->setBrush(boxBrush);

    // specify data:
    statistical->addData(1, 1.1, 1.9, 2.25, 2.7, 4.2);
    statistical->addData(2, 0.8, 1.6, 2.2, 3.2, 4.9, QVector<double>() << 0.7 << 0.34 << 0.45 << 6.2 << 5.84); // provide some outliers as QVector
    statistical->addData(3, 0.2, 0.7, 1.1, 1.6, 2.9);

    // prepare manual x axis labels:
    ui->customPlot->xAxis->setSubTicks(false);
    ui->customPlot->xAxis->setTickLength(0, 4);
    ui->customPlot->xAxis->setTickLabelRotation(20);
    QSharedPointer<QCPAxisTickerText> textTicker(new QCPAxisTickerText);
    textTicker->addTick(1, "Sample 1");
    textTicker->addTick(2, "Sample 2");
    textTicker->addTick(3, "Control Group");
    ui->customPlot->xAxis->setTicker(textTicker);

    // prepare axes:
    ui->customPlot->yAxis->setLabel(QString::fromUtf8("O? Absorption [mg]"));
    ui->customPlot->rescaleAxes();
    ui->customPlot->xAxis->scaleRange(1.7, ui->customPlot->xAxis->range().center());
    ui->customPlot->yAxis->setRange(0, 7);
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmstatistical.h`

```cpp
﻿#ifndef FRMSTATISTICAL_H
#define FRMSTATISTICAL_H

#include <QWidget>

namespace Ui {
class frmStatistical;
}

class frmStatistical : public QWidget
{
    Q_OBJECT

public:
    explicit frmStatistical(QWidget *parent = 0);
    ~frmStatistical();

private:
    Ui::frmStatistical *ui;

private slots:
    void initForm();
};

#endif // FRMSTATISTICAL_H
```

### `frmexample/frmstyled.cpp`

```cpp
﻿#include "frmstyled.h"
#include "ui_frmstyled.h"
#include "qdebug.h"

frmStyled::frmStyled(QWidget *parent) : QWidget(parent), ui(new Ui::frmStyled)
{
    ui->setupUi(this);
    this->initForm();
}

frmStyled::~frmStyled()
{
    delete ui;
}

void frmStyled::initForm()
{
    // prepare data:
    QVector<double> x1(20), y1(20);
    QVector<double> x2(100), y2(100);
    QVector<double> x3(20), y3(20);
    QVector<double> x4(20), y4(20);

    for (int i = 0; i < x1.size(); ++i) {
        x1[i] = i / (double)(x1.size() - 1) * 10;
        y1[i] = qCos(x1[i] * 0.8 + qSin(x1[i] * 0.16 + 1.0)) * qSin(x1[i] * 0.54) + 1.4;
    }

    for (int i = 0; i < x2.size(); ++i) {
        x2[i] = i / (double)(x2.size() - 1) * 10;
        y2[i] = qCos(x2[i] * 0.85 + qSin(x2[i] * 0.165 + 1.1)) * qSin(x2[i] * 0.50) + 1.7;
    }

    for (int i = 0; i < x3.size(); ++i) {
        x3[i] = i / (double)(x3.size() - 1) * 10;
        y3[i] = 0.05 + 3 * (0.5 + qCos(x3[i] * x3[i] * 0.2 + 2) * 0.5) / (double)(x3[i] + 0.7) + std::rand() / (double)RAND_MAX * 0.01;
    }

    for (int i = 0; i < x4.size(); ++i) {
        x4[i] = x3[i];
        y4[i] = (0.5 - y3[i]) + ((x4[i] - 2) * (x4[i] - 2) * 0.02);
    }

    // create and configure plottables:
    QCPGraph *graph1 = ui->customPlot->addGraph();
    graph1->setData(x1, y1);
    graph1->setScatterStyle(QCPScatterStyle(QCPScatterStyle::ssCircle, QPen(Qt::black, 1.5), QBrush(Qt::white), 9));
    graph1->setPen(QPen(QColor(120, 120, 120), 2));

    QCPGraph *graph2 = ui->customPlot->addGraph();
    graph2->setData(x2, y2);
    graph2->setPen(Qt::NoPen);
    graph2->setBrush(QColor(200, 200, 200, 20));
    graph2->setChannelFillGraph(graph1);

    QCPBars *bars1 = new QCPBars(ui->customPlot->xAxis, ui->customPlot->yAxis);
    bars1->setWidth(9 / (double)x3.size());
    bars1->setData(x3, y3);
    bars1->setPen(Qt::NoPen);
    bars1->setBrush(QColor(10, 140, 70, 160));

    QCPBars *bars2 = new QCPBars(ui->customPlot->xAxis, ui->customPlot->yAxis);
    bars2->setWidth(9 / (double)x4.size());
    bars2->setData(x4, y4);
    bars2->setPen(Qt::NoPen);
    bars2->setBrush(QColor(10, 100, 50, 70));
    bars2->moveAbove(bars1);

    // move bars above graphs and grid below bars:
    ui->customPlot->addLayer("abovemain", ui->customPlot->layer("main"), QCustomPlot::limAbove);
    ui->customPlot->addLayer("belowmain", ui->customPlot->layer("main"), QCustomPlot::limBelow);
    graph1->setLayer("abovemain");
    ui->customPlot->xAxis->grid()->setLayer("belowmain");
    ui->customPlot->yAxis->grid()->setLayer("belowmain");

    // set some pens, brushes and backgrounds:
    ui->customPlot->xAxis->setBasePen(QPen(Qt::white, 1));
    ui->customPlot->yAxis->setBasePen(QPen(Qt::white, 1));
    ui->customPlot->xAxis->setTickPen(QPen(Qt::white, 1));
    ui->customPlot->yAxis->setTickPen(QPen(Qt::white, 1));
    ui->customPlot->xAxis->setSubTickPen(QPen(Qt::white, 1));
    ui->customPlot->yAxis->setSubTickPen(QPen(Qt::white, 1));
    ui->customPlot->xAxis->setTickLabelColor(Qt::white);
    ui->customPlot->yAxis->setTickLabelColor(Qt::white);

    ui->customPlot->xAxis->grid()->setPen(QPen(QColor(140, 140, 140), 1, Qt::DotLine));
    ui->customPlot->yAxis->grid()->setPen(QPen(QColor(140, 140, 140), 1, Qt::DotLine));
    ui->customPlot->xAxis->grid()->setSubGridPen(QPen(QColor(80, 80, 80), 1, Qt::DotLine));
    ui->customPlot->yAxis->grid()->setSubGridPen(QPen(QColor(80, 80, 80), 1, Qt::DotLine));
    ui->customPlot->xAxis->grid()->setSubGridVisible(true);
    ui->customPlot->yAxis->grid()->setSubGridVisible(true);
    ui->customPlot->xAxis->grid()->setZeroLinePen(Qt::NoPen);
    ui->customPlot->yAxis->grid()->setZeroLinePen(Qt::NoPen);

    ui->customPlot->xAxis->setUpperEnding(QCPLineEnding::esSpikeArrow);
    ui->customPlot->yAxis->setUpperEnding(QCPLineEnding::esSpikeArrow);

    QLinearGradient plotGradient;
    plotGradient.setStart(0, 0);
    plotGradient.setFinalStop(0, 350);
    plotGradient.setColorAt(0, QColor(80, 80, 80));
    plotGradient.setColorAt(1, QColor(50, 50, 50));
    ui->customPlot->setBackground(plotGradient);

    QLinearGradient axisRectGradient;
    axisRectGradient.setStart(0, 0);
    axisRectGradient.setFinalStop(0, 350);
    axisRectGradient.setColorAt(0, QColor(80, 80, 80));
    axisRectGradient.setColorAt(1, QColor(30, 30, 30));
    ui->customPlot->axisRect()->setBackground(axisRectGradient);

    ui->customPlot->rescaleAxes();
    ui->customPlot->yAxis->setRange(0, 2);
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmstyled.h`

```cpp
﻿#ifndef FRMSTYLED_H
#define FRMSTYLED_H

#include <QWidget>

namespace Ui {
class frmStyled;
}

class frmStyled : public QWidget
{
    Q_OBJECT

public:
    explicit frmStyled(QWidget *parent = 0);
    ~frmStyled();

private:
    Ui::frmStyled *ui;

private slots:
    void initForm();
};

#endif // FRMSTYLED_H
```

### `frmexample/frmtexturebrush.cpp`

```cpp
﻿#include "frmtexturebrush.h"
#include "ui_frmtexturebrush.h"
#include "qdebug.h"

frmTextureBrush::frmTextureBrush(QWidget *parent) : QWidget(parent), ui(new Ui::frmTextureBrush)
{
    ui->setupUi(this);
    this->initForm();
}

frmTextureBrush::~frmTextureBrush()
{
    delete ui;
}

void frmTextureBrush::initForm()
{
    // add two graphs with a textured fill:
    ui->customPlot->addGraph();
    QPen redDotPen;
    redDotPen.setStyle(Qt::DotLine);
    redDotPen.setColor(QColor(170, 100, 100, 180));
    redDotPen.setWidthF(2);
    ui->customPlot->graph(0)->setPen(redDotPen);
    ui->customPlot->graph(0)->setBrush(QBrush(QPixmap(":/image/bg2.jpg"))); // fill with texture of specified image

    ui->customPlot->addGraph();
    ui->customPlot->graph(1)->setPen(QPen(Qt::red));

    // activate channel fill for graph 0 towards graph 1:
    ui->customPlot->graph(0)->setChannelFillGraph(ui->customPlot->graph(1));

    // generate data:
    QVector<double> x(250);
    QVector<double> y0(250), y1(250);
    for (int i = 0; i < 250; ++i) {
        // just playing with numbers, not much to learn here
        x[i] = 3 * i / 250.0;
        y0[i] = 1 + qExp(-x[i] * x[i] * 0.8) * (x[i] * x[i] + x[i]);
        y1[i] = 1 - qExp(-x[i] * x[i] * 0.4) * (x[i] * x[i]) * 0.1;
    }

    // pass data points to graphs:
    ui->customPlot->graph(0)->setData(x, y0);
    ui->customPlot->graph(1)->setData(x, y1);
    // activate top and right axes, which are invisible by default:
    ui->customPlot->xAxis2->setVisible(true);
    ui->customPlot->yAxis2->setVisible(true);
    // make tick labels invisible on top and right axis:
    ui->customPlot->xAxis2->setTickLabels(false);
    ui->customPlot->yAxis2->setTickLabels(false);
    // set ranges:
    ui->customPlot->xAxis->setRange(0, 2.5);
    ui->customPlot->yAxis->setRange(0.9, 1.6);
    // assign top/right axes same properties as bottom/left:
    ui->customPlot->axisRect()->setupFullAxesBox();
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);
}
```

### `frmexample/frmtexturebrush.h`

```cpp
﻿#ifndef FRMTEXTUREBRUSH_H
#define FRMTEXTUREBRUSH_H

#include <QWidget>

namespace Ui {
class frmTextureBrush;
}

class frmTextureBrush : public QWidget
{
    Q_OBJECT

public:
    explicit frmTextureBrush(QWidget *parent = 0);
    ~frmTextureBrush();

private:
    Ui::frmTextureBrush *ui;

private slots:
    void initForm();
};

#endif // FRMTEXTUREBRUSH_H
```

### `frmexample2/axistag.cpp`

```cpp
/***************************************************************************
**                                                                        **
**  QCustomPlot, an easy to use, modern plotting widget for Qt            **
**  Copyright (C) 2011-2021 Emanuel Eichhammer                            **
**                                                                        **
**  This program is free software: you can redistribute it and/or modify  **
**  it under the terms of the GNU General Public License as published by  **
**  the Free Software Foundation, either version 3 of the License, or     **
**  (at your option) any later version.                                   **
**                                                                        **
**  This program is distributed in the hope that it will be useful,       **
**  but WITHOUT ANY WARRANTY; without even the implied warranty of        **
**  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the         **
**  GNU General Public License for more details.                          **
**                                                                        **
**  You should have received a copy of the GNU General Public License     **
**  along with this program.  If not, see http://www.gnu.org/licenses/.   **
**                                                                        **
****************************************************************************
**           Author: Emanuel Eichhammer                                   **
**  Website/Contact: http://www.qcustomplot.com/                          **
**             Date: 29.03.21                                             **
**          Version: 2.1.0                                                **
****************************************************************************/

#include "axistag.h"

AxisTag::AxisTag(QCPAxis *parentAxis) :
    QObject(parentAxis),
    mAxis(parentAxis)
{
    // The dummy tracer serves here as an invisible anchor which always sticks to the right side of
    // the axis rect
    mDummyTracer = new QCPItemTracer(mAxis->parentPlot());
    mDummyTracer->setVisible(false);
    mDummyTracer->position->setTypeX(QCPItemPosition::ptAxisRectRatio);
    mDummyTracer->position->setTypeY(QCPItemPosition::ptPlotCoords);
    mDummyTracer->position->setAxisRect(mAxis->axisRect());
    mDummyTracer->position->setAxes(0, mAxis);
    mDummyTracer->position->setCoords(1, 0);

    // the arrow end (head) is set to move along with the dummy tracer by setting it as its parent
    // anchor. Its coordinate system (setCoords) is thus pixels, and this is how the needed horizontal
    // offset for the tag of the second y axis is achieved. This horizontal offset gets dynamically
    // updated in AxisTag::updatePosition. the arrow "start" is simply set to have the "end" as parent
    // anchor. It is given a horizontal offset to the right, which results in a 15 pixel long arrow.
    mArrow = new QCPItemLine(mAxis->parentPlot());
    mArrow->setLayer("overlay");
    mArrow->setClipToAxisRect(false);
    mArrow->setHead(QCPLineEnding::esSpikeArrow);
    mArrow->end->setParentAnchor(mDummyTracer->position);
    mArrow->start->setParentAnchor(mArrow->end);
    mArrow->start->setCoords(15, 0);

    // The text label is anchored at the arrow start (tail) and has its "position" aligned at the
    // left, and vertically centered to the text label box.
    mLabel = new QCPItemText(mAxis->parentPlot());
    mLabel->setLayer("overlay");
    mLabel->setClipToAxisRect(false);
    mLabel->setPadding(QMargins(3, 0, 3, 0));
    mLabel->setBrush(QBrush(Qt::white));
    mLabel->setPen(QPen(Qt::blue));
    mLabel->setPositionAlignment(Qt::AlignLeft | Qt::AlignVCenter);
    mLabel->position->setParentAnchor(mArrow->start);
}

AxisTag::~AxisTag()
{
    if (mDummyTracer) {
        mDummyTracer->parentPlot()->removeItem(mDummyTracer);
    }
    if (mArrow) {
        mArrow->parentPlot()->removeItem(mArrow);
    }
    if (mLabel) {
        mLabel->parentPlot()->removeItem(mLabel);
    }
}

void AxisTag::setPen(const QPen &pen)
{
    mArrow->setPen(pen);
    mLabel->setPen(pen);
}

void AxisTag::setBrush(const QBrush &brush)
{
    mLabel->setBrush(brush);
}

void AxisTag::setText(const QString &text)
{
    mLabel->setText(text);
}

void AxisTag::updatePosition(double value)
{
    // since both the arrow and the text label are chained to the dummy tracer (via anchor
    // parent-child relationships) it is sufficient to update the dummy tracer coordinates. The
    // Horizontal coordinate type was set to ptAxisRectRatio so to keep it aligned at the right side
    // of the axis rect, it is always kept at 1. The vertical coordinate type was set to
    // ptPlotCoordinates of the passed parent axis, so the vertical coordinate is set to the new
    // value.
    mDummyTracer->position->setCoords(1, value);

    // We want the arrow head to be at the same horizontal position as the axis backbone, even if
    // the axis has a certain offset from the axis rect border (like the added second y axis). Thus we
    // set the horizontal pixel position of the arrow end (head) to the axis offset (the pixel
    // distance to the axis rect border). This works because the parent anchor of the arrow end is
    // the dummy tracer, which, as described earlier, is tied to the right axis rect border.
    mArrow->end->setCoords(mAxis->offset(), 0);
}
```

### `frmexample2/axistag.h`

```cpp
/***************************************************************************
**                                                                        **
**  QCustomPlot, an easy to use, modern plotting widget for Qt            **
**  Copyright (C) 2011-2021 Emanuel Eichhammer                            **
**                                                                        **
**  This program is free software: you can redistribute it and/or modify  **
**  it under the terms of the GNU General Public License as published by  **
**  the Free Software Foundation, either version 3 of the License, or     **
**  (at your option) any later version.                                   **
**                                                                        **
**  This program is distributed in the hope that it will be useful,       **
**  but WITHOUT ANY WARRANTY; without even the implied warranty of        **
**  MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the         **
**  GNU General Public License for more details.                          **
**                                                                        **
**  You should have received a copy of the GNU General Public License     **
**  along with this program.  If not, see http://www.gnu.org/licenses/.   **
**                                                                        **
****************************************************************************
**           Author: Emanuel Eichhammer                                   **
**  Website/Contact: http://www.qcustomplot.com/                          **
**             Date: 29.03.21                                             **
**          Version: 2.1.0                                                **
****************************************************************************/

#ifndef AXISTAG_H
#define AXISTAG_H

#include <QObject>
#include "qcustomplot.h"

class AxisTag : public QObject
{
  Q_OBJECT
public:
  explicit AxisTag(QCPAxis *parentAxis);
  virtual ~AxisTag();
  
  // setters:
  void setPen(const QPen &pen);
  void setBrush(const QBrush &brush);
  void setText(const QString &text);
  
  // getters:
  QPen pen() const { return mLabel->pen(); }
  QBrush brush() const { return mLabel->brush(); }
  QString text() const { return mLabel->text(); }
  
  // other methods:
  void updatePosition(double value);
  
protected:
  QCPAxis *mAxis;
  QPointer<QCPItemTracer> mDummyTracer;
  QPointer<QCPItemLine> mArrow;
  QPointer<QCPItemText> mLabel;
};


#endif // AXISTAG_H
```

### `frmexample2/frmaxistag.cpp`

```cpp
﻿#include "frmaxistag.h"
#include "ui_frmaxistag.h"
#include "qdebug.h"

frmAxisTag::frmAxisTag(QWidget *parent) : QWidget(parent), ui(new Ui::frmAxisTag)
{
    ui->setupUi(this);
    this->initForm();
}

frmAxisTag::~frmAxisTag()
{
    delete ui;
}

void frmAxisTag::showEvent(QShowEvent *)
{
    dataTimer.start(50);
}

void frmAxisTag::hideEvent(QHideEvent *)
{
    dataTimer.stop();
}

void frmAxisTag::initForm()
{
    // configure plot to have two right axes:
    ui->customPlot->yAxis->setTickLabels(false);
    connect(ui->customPlot->yAxis2, SIGNAL(rangeChanged(QCPRange)), ui->customPlot->yAxis, SLOT(setRange(QCPRange))); // left axis only mirrors inner right axis
    ui->customPlot->yAxis2->setVisible(true);
    ui->customPlot->axisRect()->addAxis(QCPAxis::atRight);
    ui->customPlot->axisRect()->axis(QCPAxis::atRight, 0)->setPadding(30); // add some padding to have space for tags
    ui->customPlot->axisRect()->axis(QCPAxis::atRight, 1)->setPadding(30); // add some padding to have space for tags

    // create graphs:
    mGraph1 = ui->customPlot->addGraph(ui->customPlot->xAxis, ui->customPlot->axisRect()->axis(QCPAxis::atRight, 0));
    mGraph2 = ui->customPlot->addGraph(ui->customPlot->xAxis, ui->customPlot->axisRect()->axis(QCPAxis::atRight, 1));
    mGraph1->setPen(QPen(QColor(250, 120, 0)));
    mGraph2->setPen(QPen(QColor(0, 180, 60)));

    // create tags with newly introduced AxisTag class (see axistag.h/.cpp):
    mTag1 = new AxisTag(mGraph1->valueAxis());
    mTag1->setPen(mGraph1->pen());
    mTag2 = new AxisTag(mGraph2->valueAxis());
    mTag2->setPen(mGraph2->pen());

    connect(&dataTimer, SIGNAL(timeout()), this, SLOT(timerSlot()));
}

void frmAxisTag::timerSlot()
{
    // calculate and add a new data point to each graph:
    int count1 = mGraph1->dataCount();
    mGraph1->addData(count1, qSin(count1 / 50.0) + qSin(count1 / 50.0 / 0.3843) * 0.25);
    int count2 = mGraph2->dataCount();
    mGraph2->addData(count2, qCos(count2 / 50.0) + qSin(count2 / 50.0 / 0.4364) * 0.15);

    // make key axis range scroll with the data:
    ui->customPlot->xAxis->rescale();
    mGraph1->rescaleValueAxis(false, true);
    mGraph2->rescaleValueAxis(false, true);
    ui->customPlot->xAxis->setRange(ui->customPlot->xAxis->range().upper, 100, Qt::AlignRight);

    // update the vertical axis tag positions and texts to match the rightmost data point of the graphs:
    double graph1Value = mGraph1->dataMainValue(mGraph1->dataCount() - 1);
    double graph2Value = mGraph2->dataMainValue(mGraph2->dataCount() - 1);
    mTag1->updatePosition(graph1Value);
    mTag2->updatePosition(graph2Value);
    mTag1->setText(QString::number(graph1Value, 'f', 2));
    mTag2->setText(QString::number(graph2Value, 'f', 2));

    ui->customPlot->replot();
}
```

### `frmexample2/frmaxistag.h`

```cpp
﻿#ifndef FRMAXISTAG_H
#define FRMAXISTAG_H

#include <QWidget>
#include "qcustomplot.h"
#include "axistag.h"

namespace Ui {
class frmAxisTag;
}

class frmAxisTag : public QWidget
{
    Q_OBJECT

public:
    explicit frmAxisTag(QWidget *parent = 0);
    ~frmAxisTag();

protected:
    void showEvent(QShowEvent *);
    void hideEvent(QHideEvent *);

private:
    Ui::frmAxisTag *ui;
    QPointer<QCPGraph> mGraph1;
    QPointer<QCPGraph> mGraph2;
    AxisTag *mTag1;
    AxisTag *mTag2;
    QTimer dataTimer;

private slots:
    void initForm();
    void timerSlot();
};

#endif // FRMAXISTAG_H
```

### `frmexample2/frminteraction.cpp`

```cpp
﻿#include "frminteraction.h"
#include "ui_frminteraction.h"
#include "qmenu.h"
#include "qdebug.h"

frmInterAction::frmInterAction(QWidget *parent) : QWidget(parent), ui(new Ui::frmInterAction)
{
    ui->setupUi(this);
    this->initForm();
}

frmInterAction::~frmInterAction()
{
    delete ui;
}

void frmInterAction::initForm()
{
    std::srand(QDateTime::currentDateTime().toMSecsSinceEpoch() / 1000.0);

    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom | QCP::iSelectAxes | QCP::iSelectLegend | QCP::iSelectPlottables);
    ui->customPlot->xAxis->setRange(-8, 8);
    ui->customPlot->yAxis->setRange(-5, 5);
    ui->customPlot->axisRect()->setupFullAxesBox();

    ui->customPlot->plotLayout()->insertRow(0);
    QCPTextElement *title = new QCPTextElement(ui->customPlot, "Interaction Example", QFont("sans", 17, QFont::Bold));
    ui->customPlot->plotLayout()->addElement(0, 0, title);

    ui->customPlot->xAxis->setLabel("x Axis");
    ui->customPlot->yAxis->setLabel("y Axis");
    ui->customPlot->legend->setVisible(true);
    QFont legendFont = font();
    legendFont.setPointSize(10);
    ui->customPlot->legend->setFont(legendFont);
    ui->customPlot->legend->setSelectedFont(legendFont);
    //ui->customPlot->legend->setSelectableParts(QCPLegend::spItems); // legend box shall not be selectable, only legend items

    addRandomGraph();
    addRandomGraph();
    addRandomGraph();
    addRandomGraph();
    ui->customPlot->rescaleAxes();

    //  // connect slot that ties some axis selections together (especially opposite axes):
    connect(ui->customPlot, SIGNAL(selectionChangedByUser()), this, SLOT(selectionChanged()));
    //  // connect slots that takes care that when an axis is selected, only that direction can be dragged and zoomed:
    connect(ui->customPlot, SIGNAL(mousePress(QMouseEvent *)), this, SLOT(mousePress()));
    connect(ui->customPlot, SIGNAL(mouseWheel(QWheelEvent *)), this, SLOT(mouseWheel()));

    // make bottom and left axes transfer their ranges to top and right axes:
    connect(ui->customPlot->xAxis, SIGNAL(rangeChanged(QCPRange)), ui->customPlot->xAxis2, SLOT(setRange(QCPRange)));
    connect(ui->customPlot->yAxis, SIGNAL(rangeChanged(QCPRange)), ui->customPlot->yAxis2, SLOT(setRange(QCPRange)));

    // connect some interaction slots:
    connect(ui->customPlot, SIGNAL(axisDoubleClick(QCPAxis *, QCPAxis::SelectablePart, QMouseEvent *)), this, SLOT(axisLabelDoubleClick(QCPAxis *, QCPAxis::SelectablePart)));
    connect(ui->customPlot, SIGNAL(legendDoubleClick(QCPLegend *, QCPAbstractLegendItem *, QMouseEvent *)), this, SLOT(legendDoubleClick(QCPLegend *, QCPAbstractLegendItem *)));
    connect(title, SIGNAL(doubleClicked(QMouseEvent *)), this, SLOT(titleDoubleClick(QMouseEvent *)));

    // connect slot that shows a message in the status bar when a graph is clicked:
    connect(ui->customPlot, SIGNAL(plottableClick(QCPAbstractPlottable *, int, QMouseEvent *)), this, SLOT(graphClicked(QCPAbstractPlottable *, int)));

    // setup policy and connect slot for context menu popup:
    ui->customPlot->setContextMenuPolicy(Qt::CustomContextMenu);
    connect(ui->customPlot, SIGNAL(customContextMenuRequested(QPoint)), this, SLOT(contextMenuRequest(QPoint)));
}

void frmInterAction::titleDoubleClick(QMouseEvent *event)
{
    Q_UNUSED(event)
    if (QCPTextElement *title = qobject_cast<QCPTextElement *>(sender())) {
        // Set the plot title by double clicking on it
        bool ok;
        QString newTitle = QInputDialog::getText(this, "QCustomPlot example", "New plot title:", QLineEdit::Normal, title->text(), &ok);
        if (ok) {
            title->setText(newTitle);
            ui->customPlot->replot();
        }
    }
}

void frmInterAction::axisLabelDoubleClick(QCPAxis *axis, QCPAxis::SelectablePart part)
{
    // Set an axis label by double clicking on it
    if (part == QCPAxis::spAxisLabel) { // only react when the actual axis label is clicked, not tick label or axis backbone
        bool ok;
        QString newLabel = QInputDialog::getText(this, "QCustomPlot example", "New axis label:", QLineEdit::Normal, axis->label(), &ok);
        if (ok) {
            axis->setLabel(newLabel);
            ui->customPlot->replot();
        }
    }
}

void frmInterAction::legendDoubleClick(QCPLegend *legend, QCPAbstractLegendItem *item)
{
    // Rename a graph by double clicking on its legend item
    Q_UNUSED(legend)
    if (item) { // only react if item was clicked (user could have clicked on border padding of legend where there is no item, then item is 0)
        QCPPlottableLegendItem *plItem = qobject_cast<QCPPlottableLegendItem *>(item);
        bool ok;
        QString newName = QInputDialog::getText(this, "QCustomPlot example", "New graph name:", QLineEdit::Normal, plItem->plottable()->name(), &ok);
        if (ok) {
            plItem->plottable()->setName(newName);
            ui->customPlot->replot();
        }
    }
}

void frmInterAction::selectionChanged()
{
    /*
     normally, axis base line, axis tick labels and axis labels are selectable separately, but we want
     the user only to be able to select the axis as a whole, so we tie the selected states of the tick labels
     and the axis base line together. However, the axis label shall be selectable individually.

     The selection state of the left and right axes shall be synchronized as well as the state of the
     bottom and top axes.

     Further, we want to synchronize the selection of the graphs with the selection state of the respective
     legend item belonging to that graph. So the user can select a graph by either clicking on the graph itself
     or on its legend item.
    */

    // make top and bottom axes be selected synchronously, and handle axis and tick labels as one selectable object:
    if (ui->customPlot->xAxis->selectedParts().testFlag(QCPAxis::spAxis) || ui->customPlot->xAxis->selectedParts().testFlag(QCPAxis::spTickLabels) ||
        ui->customPlot->xAxis2->selectedParts().testFlag(QCPAxis::spAxis) || ui->customPlot->xAxis2->selectedParts().testFlag(QCPAxis::spTickLabels)) {
        ui->customPlot->xAxis2->setSelectedParts(QCPAxis::spAxis | QCPAxis::spTickLabels);
        ui->customPlot->xAxis->setSelectedParts(QCPAxis::spAxis | QCPAxis::spTickLabels);
    }
    // make left and right axes be selected synchronously, and handle axis and tick labels as one selectable object:
    if (ui->customPlot->yAxis->selectedParts().testFlag(QCPAxis::spAxis) || ui->customPlot->yAxis->selectedParts().testFlag(QCPAxis::spTickLabels) ||
        ui->customPlot->yAxis2->selectedParts().testFlag(QCPAxis::spAxis) || ui->customPlot->yAxis2->selectedParts().testFlag(QCPAxis::spTickLabels)) {
        ui->customPlot->yAxis2->setSelectedParts(QCPAxis::spAxis | QCPAxis::spTickLabels);
        ui->customPlot->yAxis->setSelectedParts(QCPAxis::spAxis | QCPAxis::spTickLabels);
    }

    // synchronize selection of graphs with selection of corresponding legend items:
    for (int i = 0; i < ui->customPlot->graphCount(); ++i) {
        QCPGraph *graph = ui->customPlot->graph(i);
        QCPPlottableLegendItem *item = ui->customPlot->legend->itemWithPlottable(graph);
        if (item->selected() || graph->selected()) {
            item->setSelected(true);
            graph->setSelection(QCPDataSelection(graph->data()->dataRange()));
        }
    }
}

void frmInterAction::mousePress()
{
    // if an axis is selected, only allow the direction of that axis to be dragged
    // if no axis is selected, both directions may be dragged

    if (ui->customPlot->xAxis->selectedParts().testFlag(QCPAxis::spAxis)) {
        ui->customPlot->axisRect()->setRangeDrag(ui->customPlot->xAxis->orientation());
    } else if (ui->customPlot->yAxis->selectedParts().testFlag(QCPAxis::spAxis)) {
        ui->customPlot->axisRect()->setRangeDrag(ui->customPlot->yAxis->orientation());
    } else {
        ui->customPlot->axisRect()->setRangeDrag(Qt::Horizontal | Qt::Vertical);
    }
}

void frmInterAction::mouseWheel()
{
    // if an axis is selected, only allow the direction of that axis to be zoomed
    // if no axis is selected, both directions may be zoomed

    if (ui->customPlot->xAxis->selectedParts().testFlag(QCPAxis::spAxis)) {
        ui->customPlot->axisRect()->setRangeZoom(ui->customPlot->xAxis->orientation());
    } else if (ui->customPlot->yAxis->selectedParts().testFlag(QCPAxis::spAxis)) {
        ui->customPlot->axisRect()->setRangeZoom(ui->customPlot->yAxis->orientation());
    } else {
        ui->customPlot->axisRect()->setRangeZoom(Qt::Horizontal | Qt::Vertical);
    }
}

void frmInterAction::addRandomGraph()
{
    int n = 50; // number of points in graph
    double xScale = (std::rand() / (double)RAND_MAX + 0.5) * 2;
    double yScale = (std::rand() / (double)RAND_MAX + 0.5) * 2;
    double xOffset = (std::rand() / (double)RAND_MAX - 0.5) * 4;
    double yOffset = (std::rand() / (double)RAND_MAX - 0.5) * 10;
    double r1 = (std::rand() / (double)RAND_MAX - 0.5) * 2;
    double r2 = (std::rand() / (double)RAND_MAX - 0.5) * 2;
    double r3 = (std::rand() / (double)RAND_MAX - 0.5) * 2;
    double r4 = (std::rand() / (double)RAND_MAX - 0.5) * 2;
    QVector<double> x(n), y(n);
    for (int i = 0; i < n; i++) {
        x[i] = (i / (double)n - 0.5) * 10.0 * xScale + xOffset;
        y[i] = (qSin(x[i] * r1 * 5) * qSin(qCos(x[i] * r2) * r4 * 3) + r3 * qCos(qSin(x[i]) * r4 * 2)) * yScale + yOffset;
    }

    ui->customPlot->addGraph();
    ui->customPlot->graph()->setName(QString("New graph %1").arg(ui->customPlot->graphCount() - 1));
    ui->customPlot->graph()->setData(x, y);
    ui->customPlot->graph()->setLineStyle((QCPGraph::LineStyle)(std::rand() % 5 + 1));
    if (std::rand() % 100 > 50) {
        ui->customPlot->graph()->setScatterStyle(QCPScatterStyle((QCPScatterStyle::ScatterShape)(std::rand() % 14 + 1)));
    }

    QPen graphPen;
    graphPen.setColor(QColor(std::rand() % 245 + 10, std::rand() % 245 + 10, std::rand() % 245 + 10));
    graphPen.setWidthF(std::rand() / (double)RAND_MAX * 2 + 1);
    ui->customPlot->graph()->setPen(graphPen);
    ui->customPlot->replot();
}

void frmInterAction::removeSelectedGraph()
{
    if (ui->customPlot->selectedGraphs().size() > 0) {
        ui->customPlot->removeGraph(ui->customPlot->selectedGraphs().first());
        ui->customPlot->replot();
    }
}

void frmInterAction::removeAllGraphs()
{
    ui->customPlot->clearGraphs();
    ui->customPlot->replot();
}

void frmInterAction::contextMenuRequest(QPoint pos)
{
    QMenu *menu = new QMenu(this);
    menu->setAttribute(Qt::WA_DeleteOnClose);

    if (ui->customPlot->legend->selectTest(pos, false) >= 0) { // context menu on legend requested
        menu->addAction("Move to top left", this, SLOT(moveLegend()))->setData((int)(Qt::AlignTop | Qt::AlignLeft));
        menu->addAction("Move to top center", this, SLOT(moveLegend()))->setData((int)(Qt::AlignTop | Qt::AlignHCenter));
        menu->addAction("Move to top right", this, SLOT(moveLegend()))->setData((int)(Qt::AlignTop | Qt::AlignRight));
        menu->addAction("Move to bottom right", this, SLOT(moveLegend()))->setData((int)(Qt::AlignBottom | Qt::AlignRight));
        menu->addAction("Move to bottom left", this, SLOT(moveLegend()))->setData((int)(Qt::AlignBottom | Qt::AlignLeft));
    } else { // general context menu on graphs requested
        menu->addAction("Add random graph", this, SLOT(addRandomGraph()));
        if (ui->customPlot->selectedGraphs().size() > 0) {
            menu->addAction("Remove selected graph", this, SLOT(removeSelectedGraph()));
        }
        if (ui->customPlot->graphCount() > 0) {
            menu->addAction("Remove all graphs", this, SLOT(removeAllGraphs()));
        }
    }

    menu->popup(ui->customPlot->mapToGlobal(pos));
}

void frmInterAction::moveLegend()
{
    if (QAction *contextAction = qobject_cast<QAction *>(sender())) { // make sure this slot is really called by a context menu action, so it carries the data we need
        bool ok;
        int dataInt = contextAction->data().toInt(&ok);
        if (ok) {
            ui->customPlot->axisRect()->insetLayout()->setInsetAlignment(0, (Qt::Alignment)dataInt);
            ui->customPlot->replot();
        }
    }
}

void frmInterAction::graphClicked(QCPAbstractPlottable *plottable, int dataIndex)
{
    // since we know we only have QCPGraphs in the plot, we can immediately access interface1D()
    // usually it's better to first check whether interface1D() returns non-zero, and only then use it.
    double dataValue = plottable->interface1D()->dataMainValue(dataIndex);
    QString message = QString("Clicked on graph '%1' at data point #%2 with value %3.").arg(plottable->name()).arg(dataIndex).arg(dataValue);
    //ui->statusBar->showMessage(message, 2500);
}
```

### `frmexample2/frminteraction.h`

```cpp
﻿#ifndef FRMINTERACTION_H
#define FRMINTERACTION_H

#include <QWidget>
#include <QInputDialog>
#include "qcustomplot.h"

namespace Ui {
class frmInterAction;
}

class frmInterAction : public QWidget
{
    Q_OBJECT

public:
    explicit frmInterAction(QWidget *parent = 0);
    ~frmInterAction();

private:
    Ui::frmInterAction *ui;

private slots:
    void initForm();
    void titleDoubleClick(QMouseEvent *event);
    void axisLabelDoubleClick(QCPAxis* axis, QCPAxis::SelectablePart part);
    void legendDoubleClick(QCPLegend* legend, QCPAbstractLegendItem* item);
    void selectionChanged();
    void mousePress();
    void mouseWheel();
    void addRandomGraph();
    void removeSelectedGraph();
    void removeAllGraphs();
    void contextMenuRequest(QPoint pos);
    void moveLegend();
    void graphClicked(QCPAbstractPlottable *plottable, int dataIndex);
};

#endif // FRMINTERACTION_H
```

### `frmexample2/frmscrollbar.cpp`

```cpp
﻿#include "frmscrollbar.h"
#include "ui_frmscrollbar.h"
#include "qdebug.h"

frmScrollBar::frmScrollBar(QWidget *parent) : QWidget(parent), ui(new Ui::frmScrollBar)
{
    ui->setupUi(this);
    this->initForm();
}

frmScrollBar::~frmScrollBar()
{
    delete ui;
}

void frmScrollBar::initForm()
{
    // The following plot setup is mostly taken from the plot demos:
    ui->customPlot->addGraph();
    ui->customPlot->graph()->setPen(QPen(Qt::blue));
    ui->customPlot->graph()->setBrush(QBrush(QColor(0, 0, 255, 20)));
    ui->customPlot->addGraph();
    ui->customPlot->graph()->setPen(QPen(Qt::red));

    QVector<double> x(500), y0(500), y1(500);
    for (int i = 0; i < 500; ++i) {
        x[i] = (i / 499.0 - 0.5) * 10;
        y0[i] = qExp(-x[i] * x[i] * 0.25) * qSin(x[i] * 5) * 5;
        y1[i] = qExp(-x[i] * x[i] * 0.25) * 5;
    }

    ui->customPlot->graph(0)->setData(x, y0);
    ui->customPlot->graph(1)->setData(x, y1);
    ui->customPlot->axisRect()->setupFullAxesBox(true);
    ui->customPlot->setInteractions(QCP::iRangeDrag | QCP::iRangeZoom);

    // configure scroll bars:
    // Since scroll bars only support integer values, we'll set a high default range of -500..500 and
    // divide scroll bar position values by 100 to provide a scroll range -5..5 in floating point
    // axis coordinates. if you want to dynamically grow the range accessible with the scroll bar,
    // just increase the minimum/maximum values of the scroll bars as needed.
    ui->horizontalScrollBar->setRange(-500, 500);
    ui->verticalScrollBar->setRange(-500, 500);

    // create connection between axes and scroll bars:
    connect(ui->horizontalScrollBar, SIGNAL(valueChanged(int)), this, SLOT(horzScrollBarChanged(int)));
    connect(ui->verticalScrollBar, SIGNAL(valueChanged(int)), this, SLOT(vertScrollBarChanged(int)));
    connect(ui->customPlot->xAxis, SIGNAL(rangeChanged(QCPRange)), this, SLOT(xAxisChanged(QCPRange)));
    connect(ui->customPlot->yAxis, SIGNAL(rangeChanged(QCPRange)), this, SLOT(yAxisChanged(QCPRange)));

    // initialize axis range (and scroll bar positions via signals we just connected):
    ui->customPlot->xAxis->setRange(0, 6, Qt::AlignCenter);
    ui->customPlot->yAxis->setRange(0, 10, Qt::AlignCenter);
}

void frmScrollBar::horzScrollBarChanged(int value)
{
    if (qAbs(ui->customPlot->xAxis->range().center() - value / 100.0) > 0.01) { // if user is dragging plot, we don't want to replot twice
        ui->customPlot->xAxis->setRange(value / 100.0, ui->customPlot->xAxis->range().size(), Qt::AlignCenter);
        ui->customPlot->replot();
    }
}

void frmScrollBar::vertScrollBarChanged(int value)
{
    if (qAbs(ui->customPlot->yAxis->range().center() + value / 100.0) > 0.01) { // if user is dragging plot, we don't want to replot twice
        ui->customPlot->yAxis->setRange(-value / 100.0, ui->customPlot->yAxis->range().size(), Qt::AlignCenter);
        ui->customPlot->replot();
    }
}

void frmScrollBar::xAxisChanged(QCPRange range)
{
    ui->horizontalScrollBar->setValue(qRound(range.center() * 100.0)); // adjust position of scroll bar slider
    ui->horizontalScrollBar->setPageStep(qRound(range.size() * 100.0)); // adjust size of scroll bar slider
}

void frmScrollBar::yAxisChanged(QCPRange range)
{
    ui->verticalScrollBar->setValue(qRound(-range.center() * 100.0)); // adjust position of scroll bar slider
    ui->verticalScrollBar->setPageStep(qRound(range.size() * 100.0)); // adjust size of scroll bar slider
}
```

### `frmexample2/frmscrollbar.h`

```cpp
﻿#ifndef FRMSCROLLBAR_H
#define FRMSCROLLBAR_H

#include <QWidget>
#include "qcustomplot.h"

namespace Ui {
class frmScrollBar;
}

class frmScrollBar : public QWidget
{
    Q_OBJECT

public:
    explicit frmScrollBar(QWidget *parent = 0);
    ~frmScrollBar();

private:
    Ui::frmScrollBar *ui;

private slots:
    void initForm();
    void horzScrollBarChanged(int value);
    void vertScrollBarChanged(int value);
    void xAxisChanged(QCPRange range);
    void yAxisChanged(QCPRange range);
};

#endif // FRMSCROLLBAR_H
```

### `frmcustom/frmmain.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmMain</class>
 <widget class="QWidget" name="frmMain">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>635</width>
    <height>429</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
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
    <widget class="QScrollArea" name="scrollArea">
     <property name="minimumSize">
      <size>
       <width>180</width>
       <height>0</height>
      </size>
     </property>
     <property name="maximumSize">
      <size>
       <width>180</width>
       <height>16777215</height>
      </size>
     </property>
     <property name="widgetResizable">
      <bool>true</bool>
     </property>
     <widget class="QWidget" name="widgetLeft">
      <property name="geometry">
       <rect>
        <x>0</x>
        <y>0</y>
        <width>178</width>
        <height>427</height>
       </rect>
      </property>
      <layout class="QVBoxLayout" name="verticalLayout">
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
      </layout>
     </widget>
    </widget>
   </item>
   <item>
    <widget class="QStackedWidget" name="stackedWidget"/>
   </item>
  </layout>
 </widget>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmadvancedaxes.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmAdvancedAxes</class>
 <widget class="QWidget" name="frmAdvancedAxes">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmbarchart.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmBarChart</class>
 <widget class="QWidget" name="frmBarChart">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmcolormap.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmColorMap</class>
 <widget class="QWidget" name="frmColorMap">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmdate.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmDate</class>
 <widget class="QWidget" name="frmDate">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmfinancial.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmFinancial</class>
 <widget class="QWidget" name="frmFinancial">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmitem.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmItem</class>
 <widget class="QWidget" name="frmItem">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmlinestyle.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmLineStyle</class>
 <widget class="QWidget" name="frmLineStyle">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmlogarithmic.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmLogarithmic</class>
 <widget class="QWidget" name="frmLogarithmic">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmmultiaxis.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmMultiAxis</class>
 <widget class="QWidget" name="frmMultiAxis">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmparametriccurve.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmParametricCurve</class>
 <widget class="QWidget" name="frmParametricCurve">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmpolarplot.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmPolarPlot</class>
 <widget class="QWidget" name="frmPolarPlot">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmquadratic.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmQuadratic</class>
 <widget class="QWidget" name="frmQuadratic">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmrealtimedata.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmRealtimeData</class>
 <widget class="QWidget" name="frmRealtimeData">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
   <item>
    <widget class="QLabel" name="label">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Preferred" vsizetype="Fixed">
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
    </widget>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmscatterpixmap.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmScatterPixmap</class>
 <widget class="QWidget" name="frmScatterPixmap">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmscatterstyle.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmScatterStyle</class>
 <widget class="QWidget" name="frmScatterStyle">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmsimple.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmSimple</class>
 <widget class="QWidget" name="frmSimple">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmsimpleitem.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmSimpleItem</class>
 <widget class="QWidget" name="frmSimpleItem">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmsincscatter.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmSincScatter</class>
 <widget class="QWidget" name="frmSincScatter">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmstatistical.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmStatistical</class>
 <widget class="QWidget" name="frmStatistical">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmstyled.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmStyled</class>
 <widget class="QWidget" name="frmStyled">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample/frmtexturebrush.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmTextureBrush</class>
 <widget class="QWidget" name="frmTextureBrush">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample2/frmaxistag.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmAxisTag</class>
 <widget class="QWidget" name="frmAxisTag">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header>qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample2/frminteraction.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmInterAction</class>
 <widget class="QWidget" name="frmInterAction">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="QCustomPlot" name="customPlot" native="true"/>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header>qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmexample2/frmscrollbar.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmScrollBar</class>
 <widget class="QWidget" name="frmScrollBar">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>400</width>
    <height>300</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QGridLayout" name="gridLayout">
   <item row="0" column="0">
    <widget class="QCustomPlot" name="customPlot" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Preferred" vsizetype="MinimumExpanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
    </widget>
   </item>
   <item row="0" column="1">
    <widget class="QScrollBar" name="verticalScrollBar">
     <property name="orientation">
      <enum>Qt::Vertical</enum>
     </property>
    </widget>
   </item>
   <item row="1" column="0">
    <widget class="QScrollBar" name="horizontalScrollBar">
     <property name="orientation">
      <enum>Qt::Horizontal</enum>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QCustomPlot</class>
   <extends>QWidget</extends>
   <header location="global">qcustomplot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `frmcustom/frmcustom.pri`

```makefile
FORMS += \
    $$PWD/frmmain.ui

HEADERS += \
    $$PWD/frmmain.h \
    $$PWD/iconhelper.h

SOURCES += \
    $$PWD/frmmain.cpp \
    $$PWD/iconhelper.cpp
```

### `frmexample/frmexample.pri`

```makefile
FORMS += \
    $$PWD/frmadvancedaxes.ui \
    $$PWD/frmbarchart.ui \
    $$PWD/frmcolormap.ui \
    $$PWD/frmdate.ui \
    $$PWD/frmfinancial.ui \
    $$PWD/frmitem.ui \
    $$PWD/frmlinestyle.ui \
    $$PWD/frmlogarithmic.ui \
    $$PWD/frmmultiaxis.ui \
    $$PWD/frmparametriccurve.ui \
    $$PWD/frmpolarplot.ui \
    $$PWD/frmquadratic.ui \
    $$PWD/frmrealtimedata.ui \
    $$PWD/frmscatterpixmap.ui \
    $$PWD/frmscatterstyle.ui \
    $$PWD/frmsimple.ui \
    $$PWD/frmsimpleitem.ui \
    $$PWD/frmsincscatter.ui \
    $$PWD/frmstatistical.ui \
    $$PWD/frmstyled.ui \
    $$PWD/frmtexturebrush.ui

HEADERS += \
    $$PWD/frmadvancedaxes.h \
    $$PWD/frmbarchart.h \
    $$PWD/frmcolormap.h \
    $$PWD/frmdate.h \
    $$PWD/frmfinancial.h \
    $$PWD/frmitem.h \
    $$PWD/frmlinestyle.h \
    $$PWD/frmlogarithmic.h \
    $$PWD/frmmultiaxis.h \
    $$PWD/frmparametriccurve.h \
    $$PWD/frmpolarplot.h \
    $$PWD/frmquadratic.h \
    $$PWD/frmrealtimedata.h \
    $$PWD/frmscatterpixmap.h \
    $$PWD/frmscatterstyle.h \
    $$PWD/frmsimple.h \
    $$PWD/frmsimpleitem.h \
    $$PWD/frmsincscatter.h \
    $$PWD/frmstatistical.h \
    $$PWD/frmstyled.h \
    $$PWD/frmtexturebrush.h

SOURCES += \
    $$PWD/frmadvancedaxes.cpp \
    $$PWD/frmbarchart.cpp \
    $$PWD/frmcolormap.cpp \
    $$PWD/frmdate.cpp \
    $$PWD/frmfinancial.cpp \
    $$PWD/frmitem.cpp \
    $$PWD/frmlinestyle.cpp \
    $$PWD/frmlogarithmic.cpp \
    $$PWD/frmmultiaxis.cpp \
    $$PWD/frmparametriccurve.cpp \
    $$PWD/frmpolarplot.cpp \
    $$PWD/frmquadratic.cpp \
    $$PWD/frmrealtimedata.cpp \
    $$PWD/frmscatterpixmap.cpp \
    $$PWD/frmscatterstyle.cpp \
    $$PWD/frmsimple.cpp \
    $$PWD/frmsimpleitem.cpp \
    $$PWD/frmsincscatter.cpp \
    $$PWD/frmstatistical.cpp \
    $$PWD/frmstyled.cpp \
    $$PWD/frmtexturebrush.cpp
```

### `frmexample2/frmexample2.pri`

```makefile
FORMS += \
    $$PWD/frmaxistag.ui \
    $$PWD/frminteraction.ui \
    $$PWD/frmscrollbar.ui

HEADERS += \
    $$PWD/axistag.h \
    $$PWD/frmaxistag.h \
    $$PWD/frminteraction.h \
    $$PWD/frmscrollbar.h

SOURCES += \
    $$PWD/axistag.cpp \
    $$PWD/frmaxistag.cpp \
    $$PWD/frminteraction.cpp \
    $$PWD/frmscrollbar.cpp
```
