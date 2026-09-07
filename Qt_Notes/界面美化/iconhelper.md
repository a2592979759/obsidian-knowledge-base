---
tags:
  - Qt
  - 界面美化
  - 高质量
---
# 超级图形字体 (`iconhelper`)

> 超级图形字体

## 效果图

![[QWidgetDemo_assert/iconhelper1.jpg]]
![[QWidgetDemo_assert/iconhelper2.jpg]]

## 分类

- 类型: 界面美化 `ui`
- 源码目录: `ui/iconhelper`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QWidget QString QAbstractButton QList QToolButton QFrame QLabel QPixmap QFont QEvent QButtonGroup QIcon QTextCodec QObject QVBoxLayout QSize QPushButton QColor QRadioButton QChar

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmiconhelper.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmiconhelper.h"
#include "ui_frmiconhelper.h"
#include "iconhelper.h"

frmIconHelper::frmIconHelper(QWidget *parent) : QWidget(parent), ui(new Ui::frmIconHelper)
{
    ui->setupUi(this);
    this->initForm();
    this->initWidget1();
    this->initWidget2();
    this->initWidget3();
    this->initWidget4();
    this->initWidget5();
    this->initWidget6();
    QTimer::singleShot(100, this, SLOT(initPanel()));
}

frmIconHelper::~frmIconHelper()
{
    delete ui;
}

bool frmIconHelper::eventFilter(QObject *watched, QEvent *event)
{
    if (event->type() == QEvent::Enter) {
        QLabel *lab = (QLabel *)watched;
        if (lab != 0) {
            //由于有图形字体的范围值冲突需要手动切换索引
            if (ui->rbtnFontAwesome6->isChecked()) {
                IconHelper::setIconFontIndex(2);
            } else if (ui->rbtnFontWeather->isChecked()) {
                IconHelper::setIconFontIndex(3);
            } else {
                IconHelper::setIconFontIndex(-1);
            }

            //对应图形字体的16进制值已经赋值给了 toolTip
            QString value = lab->toolTip();
            ui->labValue->setText(value);
            int icon = value.toInt(NULL, 16);

            IconHelper::setIcon(ui->labIcon, icon, iconSize);
            IconHelper::setIcon(ui->btnIcon, icon, iconSize);

            //万能大法直接从指定标识获取图片文件
            QPixmap pix = IconHelper::getPixmap("#753775", icon, iconSize, iconSize, iconSize);
            ui->labImage->setPixmap(pix);

            //设置图标  以下方法二选一都可以
            //ui->btnImage->setIcon(QIcon(pix));
            IconHelper::setPixmap(ui->btnImage, "#FD8B28", icon, iconSize, iconSize, iconSize);

            //取出对应图形字体类
            QFont font = IconHelper::getIconFontAwesome();
            if (ui->rbtnFontAliBaBa->isChecked()) {
                font = IconHelper::getIconFontAliBaBa();
            } else if (ui->rbtnFontAwesome6->isChecked()) {
                font = IconHelper::getIconFontAwesome6();
            } else if (ui->rbtnFontWeather->isChecked()) {
                font = IconHelper::getIconFontWeather();
            }

            //直接设置图标+文本到按钮
            font.setPixelSize(15);
            ui->btnTest1->setFont(font);
            ui->btnTest1->setText(QString("%1 测试图标").arg((QChar)icon));

            //分别设置图标+文本到按钮
            ui->btnTest2->setIcon(QIcon(pix));
        }
    }

    return QWidget::eventFilter(watched, event);
}

void frmIconHelper::initForm()
{
    iconSize = 40;
    //图标对应图形字体值
    icons << 0xf2ba << 0xf002 << 0xf013 << 0xf021;

    //设置样式表
    QStringList qss;
    qss << QString("#labIcon{color:#32B9CF;}");
    qss << QString("#btnIcon{color:#C13256;}");
    qss << QString("#labValue,#labCount,#labInfo1,#labInfo2,#labInfo3,#labInfo4{font-weight:bold;font-size:20px;}");
    qss << QString("QWidget#widget1 QAbstractButton{min-height:%1px;max-height:%1px;}").arg(35);
    qss << QString("QWidget#widget2 QAbstractButton{min-height:%1px;max-height:%1px;}").arg(35);
    qss << QString("QWidget#widget3 QAbstractButton{min-height:%1px;max-height:%1px;}").arg(70);
    qss << QString("QWidget#widget4 QAbstractButton{min-height:%1px;max-height:%1px;}").arg(70);
    qss << QString("QWidget#widget5 QAbstractButton{min-width:%1px;max-width:%1px;}").arg(90);
    qss << QString("QWidget#widget6 QAbstractButton{min-width:%1px;max-width:%1px;}").arg(90);
    this->setStyleSheet(qss.join(""));

    //设置图形字体
    IconHelper::setIcon(ui->labIcon, 0xf067, iconSize);
    IconHelper::setIcon(ui->btnIcon, 0xf067, iconSize);
    QPixmap pix = IconHelper::getPixmap("#753775", 0xf067, iconSize, iconSize, iconSize);
    ui->labImage->setPixmap(pix);
    ui->btnImage->setIconSize(QSize(iconSize, iconSize));
    ui->btnImage->setIcon(QIcon(pix));

    //关联单选框切换
    QList<QRadioButton *> rbtns = ui->widgetRight->findChildren<QRadioButton *>();
    foreach (QRadioButton *rbtn, rbtns) {
        connect(rbtn, SIGNAL(toggled(bool)), this, SLOT(toggled(bool)));
    }

    ui->tabWidget->setCurrentIndex(0);
}

void frmIconHelper::initPanel()
{
    //清空原有对象
    qDeleteAll(labs);
    labs.clear();

    //选择不同的图形字体
    int start = 0xf000;
    int end = 0xf2e0;
    QFont iconFont = IconHelper::getIconFontAwesome();
    IconHelper::setIconFontIndex(-1);

    if (ui->rbtnFontAliBaBa->isChecked()) {
        start = 0xe500;
        end = 0xec00;
        iconFont = IconHelper::getIconFontAliBaBa();
    } else if (ui->rbtnFontAwesome6->isChecked()) {
        start = 0xe000;
        end = 0xf8ff;
        iconFont = IconHelper::getIconFontAwesome6();
        IconHelper::setIconFontIndex(2);
    } else if (ui->rbtnFontWeather->isChecked()) {
        start = 0xe900;
        end = 0xe9cf;
        iconFont = IconHelper::getIconFontWeather();
        IconHelper::setIconFontIndex(3);
    }

    //设置字体大小
    iconFont.setPixelSize(15);

    //加载图形字体面板
    QStringList list;
    for (int icon = start; icon <= end; icon++) {
        //中间有一段是空的,可以自行屏蔽下面这段代码查看这段空的值对应的文字
        if (ui->rbtnFontAliBaBa->isChecked()) {
            if (icon >= 0xe76c && icon <= 0xe8f8) {
                continue;
            }
        } else if (ui->rbtnFontAwesome6->isChecked()) {
            if (icon >= 0xe33d && icon <= 0xefff) {
                continue;
            }
        }

        QString tip = "0x" + QString::number(icon, 16);
        if (!checkIcon(icon)) {
            list << tip;
            continue;
        }

        QLabel *lab = new QLabel;
        lab->installEventFilter(this);
        lab->setAlignment(Qt::AlignCenter);
        lab->setFont(iconFont);
        lab->setText((QChar)icon);
        lab->setToolTip(tip);
        lab->setMinimumSize(30, 30);
        labs << lab;
    }

    //qDebug() << "no text font" << list.count() << list;
    ui->widgetFontPanel->setAutoWidth(true);
    ui->widgetFontPanel->setMargin(3);
    ui->widgetFontPanel->setSpace(3);
    ui->widgetFontPanel->setColumnCount(18);
    ui->widgetFontPanel->setWidgets(labs);

    //设置图形字体面板鼠标悬停时的效果
    QString qss = QString("QLabel:hover,QLabel:focus{color:%1;border:1px solid %1;}").arg("#00BB9E");
    ui->widgetFontPanel->setStyleSheet(qss);

    int count = end - start + 1;
    ui->labCount->setText(QString("%1/%2").arg(labs.count()).arg(count));
}

bool frmIconHelper::checkIcon(int icon)
{
    //从图形字体对应值生成一个指定颜色的图片
    QPixmap pix = IconHelper::getPixmap("#FF0000", icon, 120, 120, 120);
    QImage img = pix.toImage();
    int width = img.width();
    int height = img.height();

    //过滤不存在的图形字体
    //对该图片逐个扫描像素点,都是空白则意味着当前图形字体不存在
    for (int i = 0; i < height; ++i) {
        uchar *lineByte = img.scanLine(i);
        for (int j = 0; j < width; j++) {
            uchar tp = lineByte[j];
            if (tp > 0x00) {
                return true;
            }
        }
    }

    //保存下图片看下
    //QString fileName = QString("%1/icon/%2.jpg").arg(qApp->applicationDirPath()).arg(icon);
    //pix.save(fileName, "jpg");
    return false;
}

void frmIconHelper::toggled(bool checked)
{
    //单选框按下后自动重新加载对应的图形字体
    if (checked) {
        initPanel();
    }
}

void frmIconHelper::initBtn(QButtonGroup *btnGroup, bool textBesideIcon)
{
    QList<QAbstractButton *> btns = btnGroup->buttons();
    foreach (QAbstractButton *btn, btns) {
        QToolButton *b = (QToolButton *)btn;
        //关联按钮单击事件
        connect(b, SIGNAL(clicked(bool)), this, SLOT(btnClicked()));
        b->setCheckable(true);
        b->setToolButtonStyle(textBesideIcon ? Qt::ToolButtonTextBesideIcon : Qt::ToolButtonTextUnderIcon);
    }
}

void frmIconHelper::btnClicked()
{
    QAbstractButton *btn = (QAbstractButton *)sender();
    QString objName = btn->parent()->objectName();
    if (objName == "widget1") {
        ui->labInfo1->setText(btn->text());
    } else if (objName == "widget2") {
        ui->labInfo2->setText(btn->text());
    } else if (objName == "widget3") {
        ui->labInfo3->setText(btn->text());
    } else if (objName == "widget4") {
        ui->labInfo4->setText(btn->text());
    }
}

void frmIconHelper::initWidget1()
{
    //加入按钮组自动互斥
    QButtonGroup *btnGroup = new QButtonGroup(this);
    btnGroup->addButton(ui->btn11);
    btnGroup->addButton(ui->btn12);
    btnGroup->addButton(ui->btn13);
    btnGroup->addButton(ui->btn14);

    //设置按钮可选中以及图标样式
    initBtn(btnGroup, true);
    //设置弱属性以便应用样式
    ui->widget1->setProperty("flag", "left");

    IconHelper::StyleColor styleColor;
    styleColor.defaultBorder = true;
    styleColor.position = "left";
    styleColor.iconSize = 18;
    styleColor.iconWidth = 30;
    styleColor.iconHeight = 25;
    styleColor.borderWidth = 4;
    IconHelper::setStyle(ui->widget1, btnGroup->buttons(), icons, styleColor);

    //默认选中某个按钮
    ui->btn11->click();
}

void frmIconHelper::initWidget2()
{
    //加入按钮组自动互斥
    QButtonGroup *btnGroup = new QButtonGroup(this);
    btnGroup->addButton(ui->btn21);
    btnGroup->addButton(ui->btn22);
    btnGroup->addButton(ui->btn23);
    btnGroup->addButton(ui->btn24);

    //设置按钮可选中以及图标样式
    initBtn(btnGroup, true);
    //设置弱属性以便应用样式
    ui->widget2->setProperty("flag", "right");

    IconHelper::StyleColor styleColor;
    styleColor.defaultBorder = true;
    styleColor.position = "right";
    styleColor.iconSize = 18;
    styleColor.iconWidth = 25;
    styleColor.iconHeight = 20;
    styleColor.borderWidth = 4;
    styleColor.borderColor = "#32B9CF";
    styleColor.setColor("#187294", "#B6D7E3", "#145C75", "#F0F0F0");
    IconHelper::setStyle(ui->widget2, btnGroup->buttons(), icons, styleColor);

    //默认选中某个按钮
    ui->btn22->click();
}

void frmIconHelper::initWidget3()
{
    //加入按钮组自动互斥
    QButtonGroup *btnGroup = new QButtonGroup(this);
    btnGroup->addButton(ui->btn31);
    btnGroup->addButton(ui->btn32);
    btnGroup->addButton(ui->btn33);
    btnGroup->addButton(ui->btn34);

    //设置按钮可选中以及图标样式
    initBtn(btnGroup, false);
    //设置弱属性以便应用样式
    ui->widget3->setProperty("flag", "left");

    IconHelper::StyleColor styleColor;
    styleColor.position = "left";
    styleColor.iconSize = 30;
    styleColor.iconWidth = 40;
    styleColor.iconHeight = 40;
    styleColor.borderWidth = 3;
    styleColor.borderColor = "#609EE9";
    IconHelper::setStyle(ui->widget3, btnGroup->buttons(), icons, styleColor);

    //默认选中某个按钮
    ui->btn33->click();
}

void frmIconHelper::initWidget4()
{
    //加入按钮组自动互斥
    QButtonGroup *btnGroup = new QButtonGroup(this);
    btnGroup->addButton(ui->btn41);
    btnGroup->addButton(ui->btn42);
    btnGroup->addButton(ui->btn43);
    btnGroup->addButton(ui->btn44);

    //设置按钮可选中以及图标样式
    initBtn(btnGroup, false);
    //设置弱属性以便应用样式
    ui->widget4->setProperty("flag", "right");

    IconHelper::StyleColor styleColor;
    styleColor.position = "right";
    styleColor.iconSize = 30;
    styleColor.iconWidth = 40;
    styleColor.iconHeight = 40;
    styleColor.borderWidth = 3;
    styleColor.borderColor = "#F7AE13";
    styleColor.setColor("#FCDC97", "#54626F", "#FFF0BC", "#54626F");
    IconHelper::setStyle(ui->widget4, btnGroup->buttons(), icons, styleColor);

    //默认选中某个按钮
    ui->btn44->click();
}

void frmIconHelper::initWidget5()
{
    //加入按钮组自动互斥
    QButtonGroup *btnGroup = new QButtonGroup(this);
    btnGroup->addButton(ui->btn51);
    btnGroup->addButton(ui->btn52);
    btnGroup->addButton(ui->btn53);
    btnGroup->addButton(ui->btn54);

    //设置按钮可选中以及图标样式
    initBtn(btnGroup, false);
    //设置弱属性以便应用样式
    ui->widget5->setProperty("flag", "top");

    //设置整体按钮组样式
    IconHelper::StyleColor styleColor;
    styleColor.defaultBorder = true;
    styleColor.position = "top";
    styleColor.iconSize = 25;
    styleColor.iconWidth = 25;
    styleColor.iconHeight = 25;
    styleColor.borderWidth = 3;
    IconHelper::setStyle(ui->widget5, btnGroup->buttons(), icons, styleColor);

    //默认选中某个按钮
    ui->btn51->click();
}

void frmIconHelper::initWidget6()
{
    //加入按钮组自动互斥
    QButtonGroup *btnGroup = new QButtonGroup(this);
    btnGroup->addButton(ui->btn61);
    btnGroup->addButton(ui->btn62);
    btnGroup->addButton(ui->btn63);
    btnGroup->addButton(ui->btn64);

    //设置按钮可选中以及图标样式
    initBtn(btnGroup, false);
    //设置弱属性以便应用样式
    ui->widget6->setProperty("flag", "bottom");

    //设置整体按钮组样式
    IconHelper::StyleColor styleColor;
    styleColor.defaultBorder = true;
    styleColor.position = "bottom";
    styleColor.iconSize = 25;
    styleColor.iconWidth = 25;
    styleColor.iconHeight = 25;
    styleColor.borderWidth = 3;
    styleColor.borderColor = "#A279C5";
    styleColor.setColor("#292929", "#B6D7E3", "#10689A", "#F0F0F0");
    IconHelper::setStyle(ui->widget6, btnGroup->buttons(), icons, styleColor);

    //默认选中某个按钮
    ui->btn63->click();
}
```

### `./frmiconhelper.h`

```cpp
﻿#ifndef FRMICONHELPER_H
#define FRMICONHELPER_H

#include <QWidget>
class QButtonGroup;

namespace Ui {
class frmIconHelper;
}

class frmIconHelper : public QWidget
{
    Q_OBJECT

public:
    explicit frmIconHelper(QWidget *parent = 0);
    ~frmIconHelper();

protected:
    bool eventFilter(QObject *watched, QEvent *event);

private:
    Ui::frmIconHelper *ui;
    int iconSize;
    QList<int> icons;
    QList<QWidget *> labs;

private slots:
    void initForm();
    void initPanel();
    bool checkIcon(int icon);
    void toggled(bool checked);

    void initBtn(QButtonGroup *btnGroup, bool textBesideIcon);
    void btnClicked();

    void initWidget1();
    void initWidget2();
    void initWidget3();
    void initWidget4();
    void initWidget5();
    void initWidget6();
};

#endif // FRMICONHELPER_H
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

#include "frmiconhelper.h"
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

    frmIconHelper w;
    w.setWindowTitle("图形字体示例 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./panelwidget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "panelwidget.h"
#include "qscrollarea.h"
#include "qframe.h"
#include "qboxlayout.h"
#include "qdebug.h"

PanelWidget::PanelWidget(QWidget *parent) : QWidget(parent)
{
    scrollArea = new QScrollArea(this);
    scrollArea->setObjectName("scrollAreaMain");
    scrollArea->setWidgetResizable(true);

    scrollAreaContents = new QWidget();
    scrollAreaContents->setGeometry(QRect(0, 0, 100, 100));

    verticalLayout = new QVBoxLayout(scrollAreaContents);
    verticalLayout->setSpacing(0);
    verticalLayout->setContentsMargins(0, 0, 0, 0);

    frame = new QFrame(scrollAreaContents);
    frame->setObjectName("frameMain");

    gridLayout = new QGridLayout(frame);
    gridLayout->setSpacing(0);
    gridLayout->setContentsMargins(0, 0, 0, 0);

    verticalLayout->addWidget(frame);
    scrollArea->setWidget(scrollAreaContents);
    frame->setStyleSheet("QFrame#frameMain{border-width:0px;}");

    hSpacer = new QSpacerItem(1, 1, QSizePolicy::Expanding, QSizePolicy::Minimum);
    vSpacer = new QSpacerItem(1, 1, QSizePolicy::Minimum, QSizePolicy::Expanding);

    margin = 0;
    space = 0;
    autoWidth = false;
    autoHeight = false;
}

void PanelWidget::resizeEvent(QResizeEvent *)
{
    scrollArea->resize(this->size());
}

QSize PanelWidget::sizeHint() const
{
    return QSize(300, 200);
}

QSize PanelWidget::minimumSizeHint() const
{
    return QSize(20, 20);
}

void PanelWidget::setMargin(int left, int top, int right, int bottom)
{
    gridLayout->setContentsMargins(left, top, right, bottom);
}

int PanelWidget::getMargin() const
{
    return this->margin;
}

void PanelWidget::setMargin(int margin)
{
    if (this->margin != margin) {
        setMargin(margin, margin, margin, margin);
    }
}

int PanelWidget::getSpace() const
{
    return this->space;
}

void PanelWidget::setSpace(int space)
{
    if (this->space != space) {
        gridLayout->setSpacing(space);
    }
}

bool PanelWidget::getAutoWidth() const
{
    return this->autoWidth;
}

void PanelWidget::setAutoWidth(bool autoWidth)
{
    if (this->autoWidth != autoWidth) {
        this->autoWidth = autoWidth;
    }
}

bool PanelWidget::getAutoHeight() const
{
    return this->autoHeight;
}

void PanelWidget::setAutoHeight(bool autoHeight)
{
    if (this->autoHeight != autoHeight) {
        this->autoHeight = autoHeight;
    }
}

int PanelWidget::getColumnCount() const
{
    return this->columnCount;
}

void PanelWidget::setColumnCount(int columnCount)
{
    if (this->columnCount != columnCount) {
        this->columnCount = columnCount;
    }
}

QList<QWidget *> PanelWidget::getWidgets() const
{
    return this->widgets;
}

void PanelWidget::setWidgets(QList<QWidget *> widgets)
{
    this->widgets = widgets;
    this->loadWidgets();
}

void PanelWidget::loadWidgets()
{
    int row = 0;
    int column = 0;
    int index = 0;

    //先把之前的所有移除并不可见    
    foreach (QWidget *widget, widgets) {
        gridLayout->removeWidget(widget);
        widget->setVisible(false);
    }

    //移除所有弹簧
    gridLayout->removeItem(hSpacer);
    gridLayout->removeItem(vSpacer);

    //重新添加到布局中并可见
    foreach (QWidget *widget, widgets) {
        gridLayout->addWidget(widget, row, column);
        widget->setVisible(true);

        column++;
        index++;
        if (index % columnCount == 0) {
            row++;
            column = 0;
        }
    }

    row++;

    //设置右边弹簧
    if (!autoWidth) {
        gridLayout->addItem(hSpacer, 0, gridLayout->columnCount());
    }

    //设置底边弹簧
    if (!autoHeight) {
        gridLayout->addItem(vSpacer, row, 0);
    }
}

void PanelWidget::insertWidget(int index, QWidget *widget)
{
    this->widgets.insert(index, widget);
    this->loadWidgets();
}

void PanelWidget::removeWidget(QWidget *widget)
{
    this->widgets.removeOne(widget);
    this->loadWidgets();
}

void PanelWidget::clearWidgets()
{
    qDeleteAll(this->widgets);
    this->widgets.clear();
}
```

### `./panelwidget.h`

```cpp
﻿#ifndef PANELWIDGET_H
#define PANELWIDGET_H

/**
 * 面板容器控件 作者:feiyangqingyun(QQ:517216493) 2016-11-20
 * 1. 支持所有widget子类对象，自动产生滚动条。
 * 2. 支持自动拉伸自动填充。
 * 3. 提供接口获取容器内的所有对象的指针。
 * 4. 可设置是否自动拉伸宽度高度。
 * 5. 可设置设备面板之间的间距和边距。
 */

#include <QWidget>

class QScrollArea;
class QFrame;
class QVBoxLayout;
class QGridLayout;
class QSpacerItem;

#ifdef quc
class Q_DECL_EXPORT PanelWidget : public QWidget
#else
class PanelWidget : public QWidget
#endif

{
    Q_OBJECT

    Q_PROPERTY(int margin READ getMargin WRITE setMargin)
    Q_PROPERTY(int space READ getSpace WRITE setSpace)
    Q_PROPERTY(bool autoWidth READ getAutoWidth WRITE setAutoWidth)
    Q_PROPERTY(bool autoHeight READ getAutoHeight WRITE setAutoHeight)
    Q_PROPERTY(int columnCount READ getColumnCount WRITE setColumnCount)

public:
    explicit PanelWidget(QWidget *parent = 0);

protected:
    void resizeEvent(QResizeEvent *);

private:
    QScrollArea *scrollArea;    //滚动区域
    QWidget *scrollAreaContents;//滚动区域载体
    QFrame *frame;              //放置设备的框架,自动变宽变高
    QVBoxLayout *verticalLayout;//设备面板总布局
    QGridLayout *gridLayout;    //设备表格布局
    QSpacerItem *hSpacer;       //横向弹簧
    QSpacerItem *vSpacer;       //垂直弹簧

    int margin;                 //边距
    int space;                  //设备之间的间隔
    bool autoWidth;             //宽度自动拉伸
    bool autoHeight;            //高度自动拉伸

    int columnCount;            //面板列数
    QList<QWidget *> widgets;   //设备面板对象集合

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //设置边距
    void setMargin(int left, int top, int right, int bottom);

    //获取和设置边距
    int getMargin() const;
    void setMargin(int margin);

    //获取和设置间距
    int getSpace() const;
    void setSpace(int space);

    //获取和设置自动填充宽度
    bool getAutoWidth() const;
    void setAutoWidth(bool autoWidth);

    //获取和设置自自动填充高度
    bool getAutoHeight() const;
    void setAutoHeight(bool autoHeight);

    //获取和设置列数
    int getColumnCount() const;
    void setColumnCount(int columnCount);

    //获取和设置窗体集合
    QList<QWidget *> getWidgets() const;
    void setWidgets(QList<QWidget *> widgets);

    //载入窗体集合
    void loadWidgets();
    //指定位置插入新窗体
    void insertWidget(int index, QWidget *widget);
    //移除指定的窗体
    void removeWidget(QWidget *widget);
    //清空窗体
    void clearWidgets();
};

#endif // PANELWIDGET_H
```

### `./frmiconhelper.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmIconHelper</class>
 <widget class="QWidget" name="frmIconHelper">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Dialog</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout_6">
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
    <widget class="QTabWidget" name="tabWidget">
     <property name="currentIndex">
      <number>1</number>
     </property>
     <widget class="QWidget" name="tabNav">
      <attribute name="title">
       <string>使用示例</string>
      </attribute>
      <layout class="QVBoxLayout" name="verticalLayout_5">
       <item>
        <widget class="QWidget" name="widget" native="true">
         <property name="sizePolicy">
          <sizepolicy hsizetype="Preferred" vsizetype="Expanding">
           <horstretch>0</horstretch>
           <verstretch>0</verstretch>
          </sizepolicy>
         </property>
         <layout class="QGridLayout" name="gridLayout">
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
          <item row="0" column="0">
           <widget class="QWidget" name="widget1" native="true">
            <property name="minimumSize">
             <size>
              <width>120</width>
              <height>0</height>
             </size>
            </property>
            <property name="maximumSize">
             <size>
              <width>120</width>
              <height>16777215</height>
             </size>
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
             <item>
              <widget class="QToolButton" name="btn11">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>访客登记</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn12">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>记录查询</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn13">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>系统设置</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn14">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>系统重启</string>
               </property>
              </widget>
             </item>
             <item>
              <spacer name="verticalSpacer1">
               <property name="orientation">
                <enum>Qt::Vertical</enum>
               </property>
               <property name="sizeHint" stdset="0">
                <size>
                 <width>20</width>
                 <height>366</height>
                </size>
               </property>
              </spacer>
             </item>
            </layout>
           </widget>
          </item>
          <item row="0" column="1">
           <widget class="QWidget" name="widget2" native="true">
            <property name="minimumSize">
             <size>
              <width>120</width>
              <height>0</height>
             </size>
            </property>
            <property name="maximumSize">
             <size>
              <width>120</width>
              <height>16777215</height>
             </size>
            </property>
            <layout class="QVBoxLayout" name="verticalLayout_2">
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
              <widget class="QToolButton" name="btn21">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>访客登记</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn22">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>记录查询</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn23">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>系统设置</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn24">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>系统重启</string>
               </property>
              </widget>
             </item>
             <item>
              <spacer name="verticalSpacer2">
               <property name="orientation">
                <enum>Qt::Vertical</enum>
               </property>
               <property name="sizeHint" stdset="0">
                <size>
                 <width>20</width>
                 <height>366</height>
                </size>
               </property>
              </spacer>
             </item>
            </layout>
           </widget>
          </item>
          <item row="0" column="2">
           <widget class="QWidget" name="widget3" native="true">
            <property name="minimumSize">
             <size>
              <width>120</width>
              <height>0</height>
             </size>
            </property>
            <property name="maximumSize">
             <size>
              <width>120</width>
              <height>16777215</height>
             </size>
            </property>
            <layout class="QVBoxLayout" name="verticalLayout_3">
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
              <widget class="QToolButton" name="btn31">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>访客登记</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn32">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>记录查询</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn33">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>系统设置</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn34">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>系统重启</string>
               </property>
              </widget>
             </item>
             <item>
              <spacer name="verticalSpacer3">
               <property name="orientation">
                <enum>Qt::Vertical</enum>
               </property>
               <property name="sizeHint" stdset="0">
                <size>
                 <width>20</width>
                 <height>366</height>
                </size>
               </property>
              </spacer>
             </item>
            </layout>
           </widget>
          </item>
          <item row="0" column="3">
           <widget class="QWidget" name="widget4" native="true">
            <property name="minimumSize">
             <size>
              <width>120</width>
              <height>0</height>
             </size>
            </property>
            <property name="maximumSize">
             <size>
              <width>120</width>
              <height>16777215</height>
             </size>
            </property>
            <layout class="QVBoxLayout" name="verticalLayout_4">
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
              <widget class="QToolButton" name="btn41">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>访客登记</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn42">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>记录查询</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn43">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>系统设置</string>
               </property>
              </widget>
             </item>
             <item>
              <widget class="QToolButton" name="btn44">
               <property name="sizePolicy">
                <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
                 <horstretch>0</horstretch>
                 <verstretch>0</verstretch>
                </sizepolicy>
               </property>
               <property name="text">
                <string>系统重启</string>
               </property>
              </widget>
             </item>
             <item>
              <spacer name="verticalSpacer4">
               <property name="orientation">
                <enum>Qt::Vertical</enum>
               </property>
               <property name="sizeHint" stdset="0">
                <size>
                 <width>20</width>
                 <height>366</height>
                </size>
               </property>
              </spacer>
             </item>
            </layout>
           </widget>
          </item>
          <item row="0" column="4">
           <spacer name="horizontalSpacer">
            <property name="orientation">
             <enum>Qt::Horizontal</enum>
            </property>
            <property name="sizeHint" stdset="0">
             <size>
              <width>261</width>
              <height>20</height>
             </size>
            </property>
           </spacer>
          </item>
          <item row="1" column="0">
           <widget class="QLabel" name="labInfo1">
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
          <item row="1" column="1">
           <widget class="QLabel" name="labInfo2">
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
          <item row="1" column="2">
           <widget class="QLabel" name="labInfo3">
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
          <item row="1" column="3">
           <widget class="QLabel" name="labInfo4">
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
         </layout>
        </widget>
       </item>
       <item>
        <widget class="QWidget" name="widget5" native="true">
         <property name="minimumSize">
          <size>
           <width>0</width>
           <height>75</height>
          </size>
         </property>
         <property name="maximumSize">
          <size>
           <width>16777215</width>
           <height>75</height>
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
           <widget class="QToolButton" name="btn51">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>访客登记</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="btn52">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>记录查询</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="btn53">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>系统设置</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="btn54">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>系统重启</string>
            </property>
           </widget>
          </item>
          <item>
           <spacer name="horizontalSpacer1">
            <property name="orientation">
             <enum>Qt::Horizontal</enum>
            </property>
            <property name="sizeHint" stdset="0">
             <size>
              <width>413</width>
              <height>20</height>
             </size>
            </property>
           </spacer>
          </item>
         </layout>
        </widget>
       </item>
       <item>
        <widget class="QWidget" name="widget6" native="true">
         <property name="minimumSize">
          <size>
           <width>0</width>
           <height>75</height>
          </size>
         </property>
         <property name="maximumSize">
          <size>
           <width>16777215</width>
           <height>75</height>
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
           <spacer name="horizontalSpacer2">
            <property name="orientation">
             <enum>Qt::Horizontal</enum>
            </property>
            <property name="sizeHint" stdset="0">
             <size>
              <width>413</width>
              <height>20</height>
             </size>
            </property>
           </spacer>
          </item>
          <item>
           <widget class="QToolButton" name="btn61">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>访客登记</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="btn62">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>记录查询</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="btn63">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>系统设置</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="btn64">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Fixed" vsizetype="Expanding">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="text">
             <string>系统重启</string>
            </property>
           </widget>
          </item>
         </layout>
        </widget>
       </item>
      </layout>
     </widget>
     <widget class="QWidget" name="tabFont">
      <attribute name="title">
       <string>图形字体</string>
      </attribute>
      <layout class="QHBoxLayout" name="horizontalLayout_3">
       <item>
        <widget class="PanelWidget" name="widgetFontPanel" native="true">
         <property name="sizePolicy">
          <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
           <horstretch>0</horstretch>
           <verstretch>0</verstretch>
          </sizepolicy>
         </property>
        </widget>
       </item>
       <item>
        <widget class="QWidget" name="widgetRight" native="true">
         <property name="minimumSize">
          <size>
           <width>130</width>
           <height>0</height>
          </size>
         </property>
         <layout class="QVBoxLayout" name="verticalLayout_7">
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
           <widget class="QRadioButton" name="rbtnFontAwesome">
            <property name="text">
             <string>FontAwesome</string>
            </property>
            <property name="checked">
             <bool>true</bool>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QRadioButton" name="rbtnFontAwesome6">
            <property name="text">
             <string>FontAwesome6</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QRadioButton" name="rbtnFontAliBaBa">
            <property name="text">
             <string>FontAliBaBa</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QRadioButton" name="rbtnFontWeather">
            <property name="text">
             <string>FontWeather</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QLabel" name="labCount">
            <property name="minimumSize">
             <size>
              <width>0</width>
              <height>35</height>
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
          <item>
           <widget class="QLabel" name="labValue">
            <property name="minimumSize">
             <size>
              <width>0</width>
              <height>35</height>
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
          <item>
           <widget class="QLabel" name="labIcon">
            <property name="minimumSize">
             <size>
              <width>0</width>
              <height>70</height>
             </size>
            </property>
            <property name="frameShape">
             <enum>QFrame::Box</enum>
            </property>
            <property name="frameShadow">
             <enum>QFrame::Sunken</enum>
            </property>
            <property name="text">
             <string>标签文字</string>
            </property>
            <property name="alignment">
             <set>Qt::AlignCenter</set>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QLabel" name="labImage">
            <property name="minimumSize">
             <size>
              <width>0</width>
              <height>70</height>
             </size>
            </property>
            <property name="frameShape">
             <enum>QFrame::Box</enum>
            </property>
            <property name="frameShadow">
             <enum>QFrame::Sunken</enum>
            </property>
            <property name="text">
             <string>标签图片</string>
            </property>
            <property name="alignment">
             <set>Qt::AlignCenter</set>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QPushButton" name="btnIcon">
            <property name="minimumSize">
             <size>
              <width>0</width>
              <height>70</height>
             </size>
            </property>
            <property name="text">
             <string>按钮文字</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QToolButton" name="btnImage">
            <property name="sizePolicy">
             <sizepolicy hsizetype="Preferred" vsizetype="Preferred">
              <horstretch>0</horstretch>
              <verstretch>0</verstretch>
             </sizepolicy>
            </property>
            <property name="minimumSize">
             <size>
              <width>0</width>
              <height>70</height>
             </size>
            </property>
            <property name="text">
             <string>按钮图标</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QPushButton" name="btnTest1">
            <property name="text">
             <string>测试图标</string>
            </property>
           </widget>
          </item>
          <item>
           <widget class="QPushButton" name="btnTest2">
            <property name="text">
             <string>测试图标</string>
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
              <height>105</height>
             </size>
            </property>
           </spacer>
          </item>
         </layout>
        </widget>
       </item>
      </layout>
     </widget>
    </widget>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>PanelWidget</class>
   <extends>QWidget</extends>
   <header location="global">panelwidget.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
