---
tags:
  - Qt
  - 界面美化
---
# 核心辅助库 (`core_helper`)

## 分类

- 类型: 界面美化 `ui`
- 源码目录: `ui/core_helper`

## 技术要点

**用到的 Qt 类**: QString QList QColor QWidget QAbstractButton QStringList QRect QMessageBox QObject QFont QEvent QFileDialog QPixmap QImage QByteArray QApplication QSlider QFile QSize QPalette

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./appdata.cpp`

```cpp
﻿#include "appdata.h"
#include "qthelper.h"

QString AppData::TitleFlag = "(QQ: 517216493 WX: feiyangqingyun)";
int AppData::RowHeight = 25;
int AppData::RightWidth = 250;
int AppData::FormWidth = 1200;
int AppData::FormHeight = 750;

void AppData::checkRatio()
{
    //根据分辨率设定宽高
    int width = QtHelper::deskWidth();
    if (width >= 1440) {
        RowHeight = RowHeight < 25 ? 25 : RowHeight;
        RightWidth = RightWidth < 220 ? 220 : RightWidth;
        FormWidth = FormWidth < 1200 ? 1200 : FormWidth;
        FormHeight = FormHeight < 800 ? 800 : FormHeight;
    }
}
```

### `./appdata.h`

```cpp
﻿#ifndef APPDATA_H
#define APPDATA_H

#include "head.h"

class AppData
{
public:
    static QString TitleFlag;       //标题标识
    static int RowHeight;           //行高
    static int RightWidth;          //右侧宽度
    static int FormWidth;           //窗体宽度
    static int FormHeight;          //窗体高度

    static void checkRatio();       //校验分辨率    
};

#endif // APPDATA_H
```

### `./appinit.cpp`

```cpp
﻿#include "appinit.h"
#include "qmutex.h"
#include "qapplication.h"
#include "qevent.h"
#include "qwidget.h"
#include "qdebug.h"

SINGLETON_IMPL(AppInit)
AppInit::AppInit(QObject *parent) : QObject(parent)
{
}

bool AppInit::eventFilter(QObject *watched, QEvent *event)
{ 
    QWidget *w = (QWidget *)watched;
    if (!w->property("canMove").toBool()) {
        return QObject::eventFilter(watched, event);
    }

    static QPoint mousePoint;
    static bool mousePressed = false;

    int type = event->type();
    QMouseEvent *mouseEvent = static_cast<QMouseEvent *>(event);
    if (type == QEvent::MouseButtonPress) {
        if (mouseEvent->button() == Qt::LeftButton) {
            mousePressed = true;
            mousePoint = mouseEvent->globalPos() - w->pos();
        }
    } else if (type == QEvent::MouseButtonRelease) {
        mousePressed = false;
    } else if (type == QEvent::MouseMove) {
        if (mousePressed) {
            w->move(mouseEvent->globalPos() - mousePoint);
            return true;
        }
    }

    return QObject::eventFilter(watched, event);
}

void AppInit::start()
{
    qApp->installEventFilter(this);
}
```

### `./appinit.h`

```cpp
﻿#ifndef APPINIT_H
#define APPINIT_H

#include <QObject>
#include "singleton.h"

class AppInit : public QObject
{
    Q_OBJECT SINGLETON_DECL(AppInit)
public:
    explicit AppInit(QObject *parent = 0);

protected:
    bool eventFilter(QObject *watched, QEvent *event);

public slots:
    void start();
};

#endif // APPINIT_H
```

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

### `./customstyle.cpp`

```cpp
﻿#include "customstyle.h"
#include "qapplication.h"
#include "qpalette.h"

void CustomStyle::initStyle(int fontSize, int radioButtonSize, int checkBoxSize, int sliderHeight)
{
    if (fontSize <= 0) {
        return;
    }

    QStringList list;
    //全局字体
    list << QString("*{font-size:%1px;}").arg(fontSize);
    //单选框
    list << QString("QRadioButton::indicator{width:%1px;height:%1px;}").arg(radioButtonSize);
    //复选框
    list << QString("QCheckBox::indicator,QGroupBox::indicator,QTreeWidget::indicator,QListWidget::indicator{width:%1px;height:%1px;}").arg(checkBoxSize);

    //滑块颜色
#if 0
    QString normalColor = "#e3e3e3";
    QString grooveColor = "#0078d7";
    QString handleColor = "#FFFFFF";
    QString borderColor = "#9B9B9B";
#else
    QPalette palette;
    for (int i = 0; i < 21; ++i) {
        //qDebug() << i << palette.color((QPalette::ColorRole)i).name();
    }

    QString normalColor = palette.color(QPalette::Midlight).name();
    QString grooveColor = palette.color(QPalette::Highlight).name();
    QString handleColor = palette.color(QPalette::Light).name();
    QString borderColor = palette.color(QPalette::Shadow).name();
#endif
    int sliderRadius = sliderHeight / 2;
    int handleWidth = (sliderHeight * 3) / 2 + (sliderHeight / 5);
    int handleRadius = handleWidth / 2 + 1;
    int handleOffset = handleRadius / 2;

    //横向滑块
    list << QString("QSlider::horizontal{min-height:%1px;}").arg(sliderHeight * 2);
    list << QString("QSlider::groove:horizontal{background:%1;height:%2px;border-radius:%3px;}")
         .arg(normalColor).arg(sliderHeight).arg(sliderRadius);
    list << QString("QSlider::add-page:horizontal{background:%1;height:%2px;border-radius:%3px;}")
         .arg(normalColor).arg(sliderHeight).arg(sliderRadius);
    list << QString("QSlider::sub-page:horizontal{background:%1;height:%2px;border-radius:%3px;}")
         .arg(grooveColor).arg(sliderHeight).arg(sliderRadius);
    list << QString("QSlider::handle:horizontal{border:1px solid %5;width:%2px;margin-top:-%3px;margin-bottom:-%3px;border-radius:%4px;"
                    "background:qradialgradient(spread:pad,cx:0.5,cy:0.5,radius:0.5,fx:0.5,fy:0.5,stop:0.6 #FFFFFF,stop:0.8 %1);}")
         .arg(handleColor).arg(handleWidth).arg(handleOffset).arg(handleRadius).arg(borderColor);

    //垂直滑块
    list << QString("QSlider::vertical{min-width:%1px;}").arg(sliderHeight * 2);
    list << QString("QSlider::groove:vertical{background:%1;width:%2px;border-radius:%3px;}")
         .arg(normalColor).arg(sliderHeight).arg(sliderRadius);
    list << QString("QSlider::add-page:vertical{background:%1;width:%2px;border-radius:%3px;}")
         .arg(grooveColor).arg(sliderHeight).arg(sliderRadius);
    list << QString("QSlider::sub-page:vertical{background:%1;width:%2px;border-radius:%3px;}")
         .arg(normalColor).arg(sliderHeight).arg(sliderRadius);
    list << QString("QSlider::handle:vertical{border:1px solid %5;height:%2px;margin-left:-%3px;margin-right:-%3px;border-radius:%4px;"
                    "background:qradialgradient(spread:pad,cx:0.5,cy:0.5,radius:0.5,fx:0.5,fy:0.5,stop:0.6 #FFFFFF,stop:0.8 %1);}")
         .arg(handleColor).arg(handleWidth).arg(handleOffset).arg(handleRadius).arg(borderColor);

    qApp->setStyleSheet(list.join(""));
}
```

### `./customstyle.h`

```cpp
﻿#ifndef CUSTOMSTYLE_H
#define CUSTOMSTYLE_H

#include <QObject>

class CustomStyle
{
public:
    //全局样式比如放大选择器
    static void initStyle(int fontSize = 15, int radioButtonSize = 18, int checkBoxSize = 16, int sliderHeight = 13);
};

#endif // CUSTOMSTYLE_H
```

### `./delegate.cpp`

```cpp
﻿#include "delegate.h"
#include "qcombobox.h"
#include "qdebug.h"

DelegateComboBox::DelegateComboBox(const QStringList &delegateValue, QObject *parent) : QStyledItemDelegate(parent)
{
    this->delegateValue = delegateValue;
}

QWidget *DelegateComboBox::createEditor(QWidget *parent, const QStyleOptionViewItem &option, const QModelIndex &index) const
{
    QComboBox *cbox = new QComboBox(parent);
    cbox->addItems(delegateValue);
    return cbox;
}

void DelegateComboBox::setEditorData(QWidget *editor, const QModelIndex &index) const
{
    QString data = index.data(Qt::DisplayRole).toString();
    QComboBox *cbox = static_cast<QComboBox *>(editor);
    cbox->setCurrentIndex(cbox->findText(data));
}

void DelegateComboBox::setModelData(QWidget *editor, QAbstractItemModel *model, const QModelIndex &index) const
{
    QComboBox *cbox = static_cast<QComboBox *>(editor);
    QString data = cbox->currentText();
    model->setData(index, data);
}
```

### `./delegate.h`

```cpp
﻿#ifndef DELEGATE_H
#define DELEGATE_H

#include <QStyledItemDelegate>

class DelegateComboBox : public QStyledItemDelegate
{
    Q_OBJECT

public:
    explicit DelegateComboBox(const QStringList &delegateValue, QObject *parent = 0);

protected:
    QWidget *createEditor(QWidget *parent, const QStyleOptionViewItem &option, const QModelIndex &index) const;
    void setEditorData(QWidget *editor, const QModelIndex &index) const;
    void setModelData(QWidget *editor, QAbstractItemModel *model, const QModelIndex &index) const;

private:
    QStringList delegateValue;
};

#endif // DELEGATE_H
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

### `./qthelper.cpp`

```cpp
﻿#include "qthelper.h"
#include "qnetworkinterface.h"
#include "qnetworkproxy.h"

#define TIMEMS qPrintable(QTime::currentTime().toString("HH:mm:ss zzz"))

bool QtHelper::useRatio = true;

//full指宽屏/就是将所有屏幕拼接在一起
QList<QRect> QtHelper::getScreenRects(bool available, bool full)
{
    QRect rect;
    QList<QRect> rects;
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    QList<QScreen *> screens = qApp->screens();
    int screenCount = screens.count();
    for (int i = 0; i < screenCount; ++i) {
        QScreen *screen = screens.at(i);
        if (full) {
            rect = (available ? screen->availableVirtualGeometry() : screen->virtualGeometry());
        } else {
            rect = (available ? screen->availableGeometry() : screen->geometry());
        }

        //需要根据缩放比来重新调整宽高
        qreal ratio = (QtHelper::useRatio ? screen->devicePixelRatio() : 1);
        rect.setWidth(rect.width() * ratio);
        rect.setHeight(rect.height() * ratio);
        rects << rect;
    }
#else
    QDesktopWidget *desk = qApp->desktop();
    int screenCount = desk->screenCount();
    for (int i = 0; i < screenCount; ++i) {
        if (full) {
            rect = (available ? desk->geometry() : desk->geometry());
        } else {
            rect = (available ? desk->availableGeometry(i) : desk->screenGeometry(i));
        }

        rects << rect;
    }
#endif
    return rects;
}

int QtHelper::getScreenIndex()
{
    //需要对多个屏幕进行处理
    int screenIndex = 0;
    QList<QRect> rects = getScreenRects(false);
    int count = rects.count();
    for (int i = 0; i < count; ++i) {
        //找到当前鼠标所在屏幕
        QPoint pos = QCursor::pos();
        if (rects.at(i).contains(pos)) {
            screenIndex = i;
            break;
        }
    }

    return screenIndex;
}

QRect QtHelper::getScreenRect(bool available, bool full)
{
    int screenIndex = getScreenIndex();
    QList<QRect> rects = getScreenRects(available, full);
    return rects.at(screenIndex);
}

qreal QtHelper::getScreenRatio(int index, bool devicePixel)
{
    qreal ratio = 1.0;
    //索引-1则自动获取鼠标所在当前屏幕
    int screenIndex = (index == -1 ? getScreenIndex() : index);
#if (QT_VERSION >= QT_VERSION_CHECK(5,5,0))
    QScreen *screen = qApp->screens().at(screenIndex);
    if (devicePixel) {
        //需要开启 AA_EnableHighDpiScaling 属性才能正常获取
        ratio = screen->devicePixelRatio() * 96;
    } else {
        ratio = screen->logicalDotsPerInch();
    }
#else
    //Qt4不能动态识别缩放更改后的值
    ratio = qApp->desktop()->screen(screenIndex)->logicalDpiX();
#endif
    return ratio / 96;
}

QRect QtHelper::checkCenterRect(QRect &rect, bool available)
{
    QRect deskRect = QtHelper::getScreenRect(available);
    int formWidth = rect.width();
    int formHeight = rect.height();
    int deskWidth = deskRect.width();
    int deskHeight = deskRect.height();
    int formX = deskWidth / 2 - formWidth / 2 + deskRect.x();
    int formY = deskHeight / 2 - formHeight / 2;
    rect = QRect(formX, formY, formWidth, formHeight);
    return deskRect;
}

int QtHelper::deskWidth()
{
    return getScreenRect().width();
}

int QtHelper::deskHeight()
{
    return getScreenRect().height();
}

QSize QtHelper::deskSize()
{
    return getScreenRect().size();
}

QWidget *QtHelper::centerBaseForm = 0;
void QtHelper::setFormInCenter(QWidget *form)
{
    int formWidth = form->width();
    int formHeight = form->height();

    //如果=0表示采用系统桌面屏幕为参照
    QRect rect;
    if (centerBaseForm == 0) {
        rect = getScreenRect();
    } else {
        rect = centerBaseForm->geometry();
    }

    int deskWidth = rect.width();
    int deskHeight = rect.height();
    QPoint movePoint(deskWidth / 2 - formWidth / 2 + rect.x(), deskHeight / 2 - formHeight / 2 + rect.y());
    form->move(movePoint);
}

void QtHelper::showForm(QWidget *form)
{
    setFormInCenter(form);
    form->show();

    //判断宽高是否超过了屏幕分辨率,超过了则最大化显示
    //qDebug() << TIMEMS << form->size() << deskSize();
    if (form->width() + 20 > deskWidth() || form->height() + 50 > deskHeight()) {
        QMetaObject::invokeMethod(form, "showMaximized", Qt::QueuedConnection);
    }
}

QString QtHelper::appName()
{
    //没有必要每次都获取,只有当变量为空时才去获取一次
    static QString name;
    if (name.isEmpty()) {
        name = qApp->applicationFilePath();
        //下面的方法主要为了过滤安卓的路径 lib程序名_armeabi-v7a/lib程序名_arm64-v8a
        QStringList list = name.split("/");
        name = list.at(list.count() - 1).split(".").at(0);
        name.replace("_armeabi-v7a", "");
        name.replace("_arm64-v8a", "");
    }

    return name;
}

QString QtHelper::appPath()
{
    static QString path;
    if (path.isEmpty()) {
#ifdef Q_OS_ANDROID
        //默认安卓根目录
        path = "/storage/emulated/0";
        //带上程序名称作为目录 前面加个0方便排序
        path = path + "/0" + appName();
#else
        path = qApp->applicationDirPath();
#endif
    }

    return path;
}

void QtHelper::getCurrentInfo(char *argv[], QString &path, QString &name)
{
    //必须用fromLocal8Bit保证中文路径正常
    QString argv0 = QString::fromLocal8Bit(argv[0]);
    QFileInfo file(argv0);
    path = file.path();
    name = file.baseName();
}

QString QtHelper::getIniValue(const QString &fileName, const QString &key)
{
    QString value;
    QFile file(fileName);
    if (file.open(QFile::ReadOnly | QFile::Text)) {
        while (!file.atEnd()) {
            QString line = file.readLine();
            if (line.startsWith(key)) {
                line = line.replace("\n", "");
                line = line.trimmed();
                value = line.split("=").last();
                break;
            }
        }
    }
    return value;
}

QString QtHelper::getIniValue(char *argv[], const QString &key, const QString &dir, const QString &file)
{
    QString path, name;
    QtHelper::getCurrentInfo(argv, path, name);
    //指定了名称则取指定的名称/防止程序重命名
    if (!file.isEmpty()) {
        name = file;
    }

    QString fileName = QString("%1/%2%3.ini").arg(path).arg(dir).arg(name);
    return getIniValue(fileName, key);
}

QStringList QtHelper::getLocalIPs()
{
    static QStringList ips;
    if (ips.count() == 0) {
#ifdef Q_OS_WASM
        ips << "127.0.0.1";
#else
        QList<QNetworkInterface> netInterfaces = QNetworkInterface::allInterfaces();
        foreach (QNetworkInterface netInterface, netInterfaces) {
            //移除虚拟机和抓包工具的虚拟网卡
            QString humanReadableName = netInterface.humanReadableName().toLower();
            if (humanReadableName.startsWith("vmware network adapter") || humanReadableName.startsWith("npcap loopback adapter")) {
                continue;
            }

            //过滤当前网络接口
            bool flag = (netInterface.flags() == (QNetworkInterface::IsUp | QNetworkInterface::IsRunning | QNetworkInterface::CanBroadcast | QNetworkInterface::CanMulticast));
            if (!flag) {
                continue;
            }

            QList<QNetworkAddressEntry> addrs = netInterface.addressEntries();
            foreach (QNetworkAddressEntry addr, addrs) {
                //只取出IPV4的地址
                if (addr.ip().protocol() != QAbstractSocket::IPv4Protocol) {
                    continue;
                }

                QString ip4 = addr.ip().toString();
                if (ip4 != "127.0.0.1") {
                    ips << ip4;
                }
            }
        }
#endif
    }

    return ips;
}

void QtHelper::initLocalIPs(QComboBox *cbox, const QString &defaultIP, bool local127)
{
    QStringList ips;
    if (local127) {
        ips << "127.0.0.1";
    }

    //添加本地网卡地址集合
    ips << QtHelper::getLocalIPs();

    //不在网卡地址列表中则取第一个
    QString ip = defaultIP;
    if (ips.count() > 0) {
        ip = ips.contains(ip) ? ip : ips.first();
    }

    //设置当前下拉框索引
    int index = ips.indexOf(ip);
    cbox->addItems(ips);
    cbox->setCurrentIndex(index < 0 ? 0 : index);

    //如果有文本框还要设置文本框的值
    if (cbox->lineEdit()) {
        cbox->lineEdit()->setText(ip);
    }
}

QList<QColor> QtHelper::colors = QList<QColor>();
QList<QColor> QtHelper::getColorList()
{
    //备用颜色集合 可以自行添加
    if (colors.count() == 0) {
        colors << QColor(0, 176, 180) << QColor(0, 113, 193) << QColor(255, 192, 0);
        colors << QColor(72, 103, 149) << QColor(185, 87, 86) << QColor(0, 177, 125);
        colors << QColor(214, 77, 84) << QColor(71, 164, 233) << QColor(34, 163, 169);
        colors << QColor(59, 123, 156) << QColor(162, 121, 197) << QColor(72, 202, 245);
        colors << QColor(0, 150, 121) << QColor(111, 9, 176) << QColor(250, 170, 20);
    }

    return colors;
}

QStringList QtHelper::getColorNames()
{
    QList<QColor> colors = getColorList();
    QStringList colorNames;
    foreach (QColor color, colors) {
        colorNames << color.name();
    }
    return colorNames;
}

QColor QtHelper::getRandColor()
{
    QList<QColor> colors = getColorList();
    int index = getRandValue(0, colors.count(), true);
    return colors.at(index);
}

void QtHelper::initRand()
{
    //初始化随机数种子
    QTime t = QTime::currentTime();
    srand(t.msec() + t.second() * 1000);
}

float QtHelper::getRandFloat(float min, float max)
{
    double diff = fabs(max - min);
    double value = (double)(rand() % 100) / 100;
    value = min + value * diff;
    return value;
}

double QtHelper::getRandValue(int min, int max, bool contansMin, bool contansMax)
{
    int value;
#if (QT_VERSION <= QT_VERSION_CHECK(5,10,0))
    //通用公式 a是起始值,n是整数的范围
    //int value = a + rand() % n;
    if (contansMin) {
        if (contansMax) {
            value = min + 0 + (rand() % (max - min + 1));
        } else {
            value = min + 0 + (rand() % (max - min + 0));
        }
    } else {
        if (contansMax) {
            value = min + 1 + (rand() % (max - min + 0));
        } else {
            value = min + 1 + (rand() % (max - min - 1));
        }
    }
#else
    if (contansMin) {
        if (contansMax) {
            value = QRandomGenerator::global()->bounded(min + 0, max + 1);
        } else {
            value = QRandomGenerator::global()->bounded(min + 0, max + 0);
        }
    } else {
        if (contansMax) {
            value = QRandomGenerator::global()->bounded(min + 1, max + 1);
        } else {
            value = QRandomGenerator::global()->bounded(min + 1, max + 0);
        }
    }
#endif
    return value;
}

QStringList QtHelper::getRandPoint(int count, float mainLng, float mainLat, float dotLng, float dotLat)
{
    //随机生成点坐标
    QStringList points;
    for (int i = 0; i < count; ++i) {
        //0.00881415 0.000442928
#if (QT_VERSION >= QT_VERSION_CHECK(5,10,0))
        float lngx = QRandomGenerator::global()->bounded(dotLng);
        float latx = QRandomGenerator::global()->bounded(dotLat);
#else
        float lngx = getRandFloat(dotLng / 100, dotLng);
        float latx = getRandFloat(dotLat / 100, dotLat);
#endif
        //需要先用精度转换成字符串
        QString lng2 = QString::number(mainLng + lngx, 'f', 8);
        QString lat2 = QString::number(mainLat + latx, 'f', 8);
        QString point = QString("%1,%2").arg(lng2).arg(lat2);
        points << point;
    }

    return points;
}

int QtHelper::getRangeValue(int oldMin, int oldMax, int oldValue, int newMin, int newMax)
{
    return (((oldValue - oldMin) * (newMax - newMin)) / (oldMax - oldMin)) + newMin;
}

QString QtHelper::getUuid()
{
    QString uuid = QUuid::createUuid().toString();
    uuid.replace("{", "");
    uuid.replace("}", "");
    return uuid;
}

QString QtHelper::checkPath(const QString &dirName)
{
    //相对路径需要补全完整路径
    QString path = dirName;
    if (path.startsWith("./")) {
        path.replace(".", "");
        path = QtHelper::appPath() + path;
    } else if (!path.startsWith("/") && !path.contains(":/")) {
        path = QtHelper::appPath() + "/" + path;
    }

    //目录不存在则新建
    QDir dir(path);
    if (!dir.exists()) {
        dir.mkpath(path);
    }

    return path;
}

QString QtHelper::checkFile(const QString &fileName)
{
    //将相对路径转换成完整路径
    QString name = fileName;
    if (name.startsWith("./")) {
        name = QtHelper::appPath() + name.mid(1, name.length());
    }

    return name;
}

void QtHelper::sleep(int msec, bool exec)
{
    if (msec <= 0) {
        return;
    }

    if (exec) {
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
        //阻塞方式延时(如果在主线程会卡住主界面)
        QThread::msleep(msec);
#else
        //非阻塞方式延时(不会卡住主界面/据说可能有问题)
        QTime endTime = QTime::currentTime().addMSecs(msec);
        while (QTime::currentTime() < endTime) {
            QCoreApplication::processEvents(QEventLoop::AllEvents, 100);
        }
#endif
    } else {
        //非阻塞方式延时(现在很多人推荐的方法)
        QEventLoop loop;
        QTimer::singleShot(msec, &loop, SLOT(quit()));
        loop.exec();
    }
}

void QtHelper::checkRun()
{
#ifdef Q_OS_WIN
    //延时1秒钟,等待程序释放完毕
    QtHelper::sleep(1000);
    //创建共享内存,判断是否已经运行程序
    static QSharedMemory mem(QtHelper::appName());
    if (!mem.create(1)) {
        QtHelper::showMessageBoxError("程序已运行, 软件将自动关闭!", 5, true);
        exit(0);
    }
#endif
}

void QtHelper::setStyle()
{
    //打印下所有内置风格的名字
    //qDebug() << TIMEMS << "QStyleFactory::keys" << QStyleFactory::keys();

    //设置内置风格
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    qApp->setStyle("Fusion");
#else
    qApp->setStyle("Cleanlooks");
#endif

    //设置指定颜色
    QPalette palette;
    palette.setBrush(QPalette::Window, QColor("#F0F0F0"));
    //qApp->setPalette(palette);
}

QFont QtHelper::addFont(const QString &fontFile, const QString &fontName)
{
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
    QFont font;
    if (fontDb.families().contains(fontName)) {
        font = QFont(fontName);
#if (QT_VERSION >= QT_VERSION_CHECK(4,8,0))
        font.setHintingPreference(QFont::PreferNoHinting);
#endif
    }

    return font;
}

void QtHelper::setFont(int fontSize)
{
    //安卓套件在有些手机上默认字体不好看需要主动设置字体
    //网页套件需要主动加载中文字体才能正常显示中文
#if (defined Q_OS_ANDROID) || (defined Q_OS_WASM)
    QString fontFile = ":/font/DroidSansFallback.ttf";
    QString fontName = "Droid Sans Fallback";
    qApp->setFont(addFont(fontFile, fontName));
    return;
#endif

#ifdef __arm__
    fontSize = 25;
#endif

    QFont font;
    font.setFamily("MicroSoft Yahei");
    font.setPixelSize(fontSize);
    qApp->setFont(font);
}

void QtHelper::setCode(bool utf8)
{
    QTextCodec *codec = QTextCodec::codecForName("utf-8");
#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
    QTextCodec::setCodecForCStrings(codec);
    QTextCodec::setCodecForTr(codec);
#endif

    //如果想要控制台打印信息中文正常就注释掉这个设置/setCodecForLocale会影响toLocal8Bit函数
    if (utf8) {
        QTextCodec::setCodecForLocale(codec);
    }
}

void QtHelper::setTranslator(const QString &qmFile)
{
    //过滤下不存在的就不用设置了
    if (!QFile(qmFile).exists()) {
        return;
    }

    QTranslator *translator = new QTranslator(qApp);
    if (translator->load(qmFile)) {
        qApp->installTranslator(translator);
    }
}

#ifdef Q_OS_ANDROID
#if (QT_VERSION < QT_VERSION_CHECK(6,0,0))
#include <QtAndroidExtras>
#else
//Qt6中将相关类移到了core模块而且名字变了
#include <QtCore/private/qandroidextras_p.h>
#endif
#endif

bool QtHelper::checkPermission(const QString &permission)
{
#ifdef Q_OS_ANDROID
#if (QT_VERSION >= QT_VERSION_CHECK(5,10,0) && QT_VERSION < QT_VERSION_CHECK(6,0,0))
    QtAndroid::PermissionResult result = QtAndroid::checkPermission(permission);
    if (result == QtAndroid::PermissionResult::Denied) {
        QtAndroid::requestPermissionsSync(QStringList() << permission);
        result = QtAndroid::checkPermission(permission);
        if (result == QtAndroid::PermissionResult::Denied) {
            return false;
        }
    }
#else
    QFuture<QtAndroidPrivate::PermissionResult> result = QtAndroidPrivate::requestPermission(permission);
    if (result.resultAt(0) == QtAndroidPrivate::PermissionResult::Denied) {
        return false;
    }
#endif
#endif
    return true;
}

void QtHelper::initAndroidPermission()
{
    //可以把所有要动态申请的权限都写在这里
    checkPermission("android.permission.CALL_PHONE");
    checkPermission("android.permission.SEND_SMS");
    checkPermission("android.permission.CAMERA");
    checkPermission("android.permission.READ_EXTERNAL_STORAGE");
    checkPermission("android.permission.WRITE_EXTERNAL_STORAGE");

    checkPermission("android.permission.ACCESS_COARSE_LOCATION");
    checkPermission("android.permission.INTERNET");
    checkPermission("android.permission.BLUETOOTH");
    checkPermission("android.permission.BLUETOOTH_SCAN");
    checkPermission("android.permission.BLUETOOTH_CONNECT");
    checkPermission("android.permission.BLUETOOTH_ADVERTISE");
}

void QtHelper::initAll(bool utf8, bool style, bool tabCenter, int fontSize)
{
    //初始化安卓权限
    QtHelper::initAndroidPermission();
    //初始化随机数种子
    QtHelper::initRand();
    //设置编码
    QtHelper::setCode(utf8);
    //设置字体
    QtHelper::setFont(fontSize);

    //设置样式风格
    if (style) {
        QtHelper::setStyle();
    }

    //选项卡居中
    if (tabCenter) {
        qApp->setStyleSheet("QTabWidget::tab-bar{alignment:center;}");
    }

    //设置翻译文件支持多个
    QtHelper::setTranslator(":/qm/widgets.qm");
    QtHelper::setTranslator(":/qm/qt_zh_CN.qm");
    QtHelper::setTranslator(":/qm/designer_zh_CN.qm");

    //设置不使用本地系统环境代理配置
    QNetworkProxyFactory::setUseSystemConfiguration(false);
    //设置当前目录为程序可执行文件所在目录
    QDir::setCurrent(QtHelper::appPath());
    //Qt4中默认没有程序名称需要主动设置
#if (QT_VERSION < QT_VERSION_CHECK(5,0,0))
    qApp->setApplicationName(QtHelper::appName());
#endif
}

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
#ifdef webengine
#include "qquickwindow.h"
#endif
#endif

void QtHelper::initMain(bool desktopSettingsAware, bool use96Dpi, bool logCritical)
{
#ifdef Q_OS_LINUX
#ifndef Q_OS_ANDROID
    //Qt6开始默认用wayland/由于没有坐标系统导致无边框窗体不可用
    //qputenv("QT_QPA_PLATFORM", "xcb");
#endif
#endif

#ifdef webengine
    //谷歌浏览器禁用沙箱和安全策略以便跨域请求/否则可能报错 Access-Control-Allow-Origin
    qputenv("QTWEBENGINE_DISABLE_SANDBOX", "1");
    qputenv("QTWEBENGINE_CHROMIUM_FLAGS", "--disable-web-security");
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    //设置是否应用操作系统设置比如字体
    QApplication::setDesktopSettingsAware(desktopSettingsAware);
#endif

    //安卓必须启用高分屏
#ifdef Q_OS_ANDROID
    use96Dpi = false;
#endif

    QtHelper::useRatio = use96Dpi;
#if (QT_VERSION >= QT_VERSION_CHECK(5,6,0) && QT_VERSION < QT_VERSION_CHECK(6,0,0))
    //开启高分屏缩放支持
    if (!use96Dpi) {
        QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
        QApplication::setAttribute(Qt::AA_UseHighDpiPixmaps);
    }
#endif

#ifdef Q_OS_WIN
    if (use96Dpi) {
        //Qt6中AA_Use96Dpi没效果必须下面方式设置强制指定缩放DPI
        qputenv("QT_FONT_DPI", "96");
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
        //不应用任何缩放
        QApplication::setAttribute(Qt::AA_Use96Dpi);
#endif
    }
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(5,14,0))
    //高分屏缩放策略
    QApplication::setHighDpiScaleFactorRoundingPolicy(Qt::HighDpiScaleFactorRoundingPolicy::PassThrough);
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    //下面这行表示不打印Qt内部类的警告提示信息
    if (!logCritical) {
        QLoggingCategory::setFilterRules("*.critical=false\n*.warning=false");
    }
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(5,4,0))
    //设置opengl共享上下文
    QApplication::setAttribute(Qt::AA_ShareOpenGLContexts);
#endif

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
#ifdef webengine
    //修复openglwidget和webengine共存出现黑屏的bug
    QQuickWindow::setGraphicsApi(QSGRendererInterface::OpenGL);
#endif
#endif
}

void QtHelper::initOpenGL(quint8 type, bool checkCardEnable, bool checkVirtualSystem)
{
#if (QT_VERSION >= QT_VERSION_CHECK(5,4,0))
    //设置opengl模式 AA_UseDesktopOpenGL(默认) AA_UseOpenGLES AA_UseSoftwareOpenGL
    //在一些很旧的设备上或者对opengl支持很低的设备上需要使用AA_UseOpenGLES表示禁用硬件加速
    //如果开启的是AA_UseOpenGLES则无法使用硬件加速比如ffmpeg的dxva2
    if (type == 1) {
        QApplication::setAttribute(Qt::AA_UseDesktopOpenGL);
    } else if (type == 2) {
        QApplication::setAttribute(Qt::AA_UseOpenGLES);
    } else if (type == 3) {
        QApplication::setAttribute(Qt::AA_UseSoftwareOpenGL);
    }

    //检测显卡是否被禁用
    if (checkCardEnable && !isVideoCardEnable()) {
        QApplication::setAttribute(Qt::AA_UseOpenGLES);
    }

    //检测是否是虚拟机系统
    if (checkVirtualSystem && isVirtualSystem()) {
        QApplication::setAttribute(Qt::AA_UseOpenGLES);
    }
#endif
}

QString QtHelper::getStyle(const QString &qssFile)
{
    QString qss;
    QFile file(qssFile);
    if (file.open(QFile::ReadOnly)) {
#if 0
        qss = QLatin1String(file.readAll());
#else
        //用下面这种方式读取可以不用区分文件编码
        QStringList list;
        QTextStream stream(&file);
        while (!stream.atEnd()) {
            QString line;
            stream >> line;
            list << line;
        }
        qss = list.join("\n");
#endif
    }

    return qss.trimmed();
}

void QtHelper::setStyle(const QString &qssFile)
{
    QString qss = QtHelper::getStyle(qssFile);
    if (!qss.isEmpty()) {
        QString paletteColor = qss.mid(20, 7);
        qApp->setPalette(QPalette(QColor(paletteColor)));
        qApp->setStyleSheet(qss);
    }
}

QString QtHelper::doCmd(const QString &program, const QStringList &arguments, int timeout)
{
    QString result;
#ifndef Q_OS_WASM
    QProcess p;
    p.start(program, arguments);
    p.waitForFinished(timeout);
    result = QString::fromLocal8Bit(p.readAllStandardOutput());
    result.replace("\r", "");
    result.replace("\n", "");
    result = result.simplified();
    result = result.trimmed();
#endif
    return result;
}

bool QtHelper::isVideoCardEnable()
{
    QString result;
    bool videoCardEnable = true;

#if defined(Q_OS_WIN)
    QStringList args;
    //wmic path win32_VideoController get name,Status
    args << "path" << "win32_VideoController" << "get" << "name,Status";
    result = doCmd("wmic", args);
#endif

    //Name Status Intel(R) UHD Graphics 630 OK
    //Name Status Intel(R) UHD Graphics 630 Error
    if (result.contains("Error")) {
        videoCardEnable = false;
    }

    return videoCardEnable;
}

bool QtHelper::isVirtualSystem()
{
    QString result;
    bool virtualSystem = false;

#if defined(Q_OS_WIN)
    QStringList args;
    //wmic computersystem get Model
    args << "computersystem" << "get" << "Model";
    result = doCmd("wmic", args);
#elif defined(Q_OS_LINUX)
    QStringList args;
    //还有个命令需要root权限运行 dmidecode -s system-product-name 执行结果和win一样
    result = doCmd("lscpu", args);
#endif

    //Model MS-7C00
    //Model VMWare Virtual Platform
    //Model VirtualBox Virtual Platform
    //Model Alibaba Cloud ECS
    if (result.contains("VMware") || result.contains("VirtualBox") || result.contains("Alibaba")) {
        virtualSystem = true;
    }

    return virtualSystem;
}

bool QtHelper::replaceCRLF = true;
QVector<int> QtHelper::msgTypes = QVector<int>() << 0 << 1 << 2 << 3 << 4;
QVector<QString> QtHelper::msgKeys = QVector<QString>() << QString::fromUtf8("发送") << QString::fromUtf8("接收") << QString::fromUtf8("解析") << QString::fromUtf8("错误") << QString::fromUtf8("提示");
QVector<QColor> QtHelper::msgColors = QVector<QColor>() << QColor("#3BA372") << QColor("#EE6668") << QColor("#9861B4") << QColor("#FA8359") << QColor("#22A3A9");
QString QtHelper::appendMsg(QTextEdit *textEdit, int type, const QString &data, int maxCount, int &currentCount, bool clear, bool pause)
{
    if (clear) {
        textEdit->clear();
        currentCount = 0;
        return QString();
    }

    if (pause) {
        return QString();
    }

    if (currentCount >= maxCount) {
        textEdit->clear();
        currentCount = 0;
    }

    //不同类型不同颜色显示
    QString strType;
    int index = msgTypes.indexOf(type);
    if (index >= 0) {
        strType = msgKeys.at(index);
        textEdit->setTextColor(msgColors.at(index));
    }

    //过滤回车换行符
    QString strData = data;
    if (replaceCRLF) {
        strData.replace("\r", "");
        strData.replace("\n", "");
    }

    strData = QString("时间[%1] %2: %3").arg(TIMEMS).arg(strType).arg(strData);
    textEdit->append(strData);
    currentCount++;
    return strData;
}

void QtHelper::setFramelessForm(QWidget *widgetMain, bool tool, bool top, bool menu)
{
    widgetMain->setProperty("form", true);
    widgetMain->setProperty("canMove", true);

    //根据设定逐个追加属性
#ifdef __arm__
    widgetMain->setWindowFlags(Qt::FramelessWindowHint | Qt::X11BypassWindowManagerHint);
#else
    widgetMain->setWindowFlags(Qt::FramelessWindowHint);
#endif
    if (tool) {
        widgetMain->setWindowFlags(widgetMain->windowFlags() | Qt::Tool);
    }
    if (top) {
        widgetMain->setWindowFlags(widgetMain->windowFlags() | Qt::WindowStaysOnTopHint);
    }
    if (menu) {
        //如果是其他系统比如neokylin会产生系统边框
#ifdef Q_OS_WIN
        widgetMain->setWindowFlags(widgetMain->windowFlags() | Qt::WindowSystemMenuHint | Qt::WindowMinMaxButtonsHint);
#endif
    }
}

int QtHelper::showMessageBox(const QString &text, int type, int closeSec, bool exec)
{
    int result = 0;
    if (type == 0) {
        showMessageBoxInfo(text, closeSec, exec);
    } else if (type == 1) {
        showMessageBoxError(text, closeSec, exec);
    } else if (type == 2) {
        result = showMessageBoxQuestion(text);
    }

    return result;
}

void QtHelper::showMessageBoxInfo(const QString &text, int closeSec, bool exec)
{
    QMessageBox box(QMessageBox::Information, "提示", text);
    box.setStandardButtons(QMessageBox::Yes);
    box.button(QMessageBox::Yes)->setText("确 定");
    box.exec();
    //QMessageBox::information(0, "提示", info, QMessageBox::Yes);
}

void QtHelper::showMessageBoxError(const QString &text, int closeSec, bool exec)
{
    QMessageBox box(QMessageBox::Critical, "错误", text);
    box.setStandardButtons(QMessageBox::Yes);
    box.button(QMessageBox::Yes)->setText("确 定");
    box.exec();
    //QMessageBox::critical(0, "错误", info, QMessageBox::Yes);
}

int QtHelper::showMessageBoxQuestion(const QString &text)
{
    QMessageBox box(QMessageBox::Question, "询问", text);
    box.setStandardButtons(QMessageBox::Yes | QMessageBox::No);
    box.button(QMessageBox::Yes)->setText("确 定");
    box.button(QMessageBox::No)->setText("取 消");
    return box.exec();
    //return QMessageBox::question(0, "询问", info, QMessageBox::Yes | QMessageBox::No);
}

void QtHelper::initDialog(QFileDialog *dialog, const QString &title, const QString &acceptName,
                          const QString &dirName, bool native, int width, int height)
{
    //设置标题
    dialog->setWindowTitle(title);
    //设置标签文本
    dialog->setLabelText(QFileDialog::Accept, acceptName);
    dialog->setLabelText(QFileDialog::Reject, "取消(&C)");
    dialog->setLabelText(QFileDialog::LookIn, "查看");
    dialog->setLabelText(QFileDialog::FileName, "名称");
    dialog->setLabelText(QFileDialog::FileType, "类型");

    //设置默认显示目录
    if (!dirName.isEmpty()) {
        dialog->setDirectory(dirName);
    }

    //设置对话框宽高
    if (width > 0 && height > 0) {
#ifdef Q_OS_ANDROID
        bool horizontal = (QtHelper::deskWidth() > QtHelper::deskHeight());
        if (horizontal) {
            width = QtHelper::deskWidth() / 2;
            height = QtHelper::deskHeight() - 50;
        } else {
            width = QtHelper::deskWidth() - 10;
            height = QtHelper::deskHeight() / 2;
        }
#endif
        dialog->setFixedSize(width, height);
    }

    //设置是否采用本地对话框
    dialog->setOption(QFileDialog::DontUseNativeDialog, !native);
    //设置只读可以取消右上角的新建按钮
    //dialog->setReadOnly(true);
}

QString QtHelper::getDialogResult(QFileDialog *dialog)
{
    QString result;
    if (dialog->exec() == QFileDialog::Accepted) {
        result = dialog->selectedFiles().first();
        if (!result.contains(".")) {
            //自动补全拓展名 保存文件(*.txt *.exe)
            QString filter = dialog->selectedNameFilter();
            if (filter.contains("*.")) {
                filter = filter.split("(").last();
                filter = filter.mid(0, filter.length() - 1);
                //取出第一个作为拓展名
                if (!filter.contains("*.*")) {
                    filter = filter.split(" ").first();
                    result = result + filter.mid(1, filter.length());
                }
            }
        }
    }
    return result;
}

QString QtHelper::getOpenFileName(const QString &filter, const QString &dirName, const QString &fileName,
                                  bool native, int width, int height)
{
    QFileDialog dialog;
    initDialog(&dialog, "打开文件", "选择(&S)", dirName, native, width, height);

    //设置文件类型
    if (!filter.isEmpty()) {
        dialog.setNameFilter(filter);
    }

    //设置默认文件名称
    dialog.selectFile(fileName);
    return getDialogResult(&dialog);
}

QString QtHelper::getSaveFileName(const QString &filter, const QString &dirName, const QString &fileName,
                                  bool native, int width, int height)
{
    QFileDialog dialog;
    initDialog(&dialog, "保存文件", "保存(&S)", dirName, native, width, height);

    //设置文件类型
    if (!filter.isEmpty()) {
        dialog.setNameFilter(filter);
    }

    //设置默认文件名称
    dialog.selectFile(fileName);
    //设置模态类型允许输入
    dialog.setWindowModality(Qt::WindowModal);
    //设置置顶显示
    dialog.setWindowFlags(dialog.windowFlags() | Qt::WindowStaysOnTopHint);
    return getDialogResult(&dialog);
}

QString QtHelper::getExistingDirectory(const QString &dirName, bool native, int width, int height)
{
    QFileDialog dialog;
    initDialog(&dialog, "选择目录", "选择(&S)", dirName, native, width, height);
    dialog.setOption(QFileDialog::ReadOnly);
    //设置只显示目录
#if (QT_VERSION < QT_VERSION_CHECK(6,0,0))
    dialog.setFileMode(QFileDialog::DirectoryOnly);
#else
    dialog.setFileMode(QFileDialog::Directory);
#endif
    dialog.setOption(QFileDialog::ShowDirsOnly);
    return getDialogResult(&dialog);
}

QString QtHelper::getXorEncryptDecrypt(const QString &value, char key)
{
    //矫正范围外的数据
    if (key < 0 || key >= 127) {
        key = 127;
    }

    //大概从5.9版本输出的加密密码字符串前面会加上 @String 字符
    QString result = value;
    if (result.startsWith("@String")) {
        result = result.mid(8, result.length() - 9);
    }

    for (int i = 0; i < result.length(); ++i) {
        result[i] = QChar(result.at(i).toLatin1() ^ key);
    }
    return result;
}

quint8 QtHelper::getOrCode(const QByteArray &data)
{
    int len = data.length();
    quint8 result = 0;
    for (int i = 0; i < len; ++i) {
        result ^= data.at(i);
    }

    return result;
}

quint8 QtHelper::getCheckCode(const QByteArray &data)
{
    int len = data.length();
    quint8 temp = 0;
    for (int i = 0; i < len; ++i) {
        temp += data.at(i);
    }

    return temp % 256;
}

void QtHelper::initTableView(QTableView *tableView, int rowHeight, bool headVisible, bool edit, bool stretchLast)
{
    //设置弱属性用于应用qss特殊样式
    tableView->setProperty("model", true);
    //取消自动换行
    tableView->setWordWrap(false);
    //超出文本不显示省略号
    tableView->setTextElideMode(Qt::ElideNone);
    //奇数偶数行颜色交替
    tableView->setAlternatingRowColors(false);
    //垂直表头是否可见
    tableView->verticalHeader()->setVisible(headVisible);
    //选中一行表头是否加粗
    tableView->horizontalHeader()->setHighlightSections(false);
    //最后一行拉伸填充
    tableView->horizontalHeader()->setStretchLastSection(stretchLast);
    //行标题最小宽度尺寸
    tableView->horizontalHeader()->setMinimumSectionSize(0);
    //行标题最小高度,等同于和默认行高一致
    tableView->horizontalHeader()->setFixedHeight(rowHeight);
    //默认行高
    tableView->verticalHeader()->setDefaultSectionSize(rowHeight);
    //选中时一行整体选中
    tableView->setSelectionBehavior(QAbstractItemView::SelectRows);
    //只允许选择单个
    tableView->setSelectionMode(QAbstractItemView::SingleSelection);

    //表头不可单击
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    tableView->horizontalHeader()->setSectionsClickable(false);
#else
    tableView->horizontalHeader()->setClickable(false);
#endif

    //鼠标按下即进入编辑模式
    if (edit) {
        tableView->setEditTriggers(QAbstractItemView::CurrentChanged | QAbstractItemView::DoubleClicked);
    } else {
        tableView->setEditTriggers(QAbstractItemView::NoEditTriggers);
    }
}

void QtHelper::openFile(const QString &fileName, const QString &msg)
{
#ifdef __arm__
    return;
#endif
    //文件不存在则不用处理
    if (!QFile(fileName).exists()) {
        return;
    }
    if (QtHelper::showMessageBoxQuestion(msg + "成功, 确定现在就打开吗?") == QMessageBox::Yes) {
        QString url = QString("file:///%1").arg(fileName);
        QDesktopServices::openUrl(QUrl(url, QUrl::TolerantMode));
    }
}

bool QtHelper::checkIniFile(const QString &iniFile)
{
    //如果配置文件大小为0,则以初始值继续运行,并生成配置文件
    QFile file(iniFile);
    if (file.size() == 0) {
        return false;
    }

    //如果配置文件不完整,则以初始值继续运行,并生成配置文件
    if (file.open(QFile::ReadOnly)) {
        bool ok = true;
        while (!file.atEnd()) {
            QString line = file.readLine();
            line.replace("\r", "");
            line.replace("\n", "");
            QStringList list = line.split("=");

            if (list.count() == 2) {
                QString key = list.at(0);
                QString value = list.at(1);
                if (value.isEmpty()) {
                    qDebug() << TIMEMS << "ini node no value" << key;
                    ok = false;
                    break;
                }
            }
        }

        if (!ok) {
            return false;
        }
    } else {
        return false;
    }

    return true;
}

QString QtHelper::cutString(const QString &text, int len, int left, int right, bool file, const QString &mid)
{
    //如果指定了字符串分割则表示是文件名需要去掉拓展名
    QString result = text;
    if (file && result.contains(".")) {
        int index = result.lastIndexOf(".");
        result = result.mid(0, index);
    }

    //最终字符串格式为 前缀字符...后缀字符
    if (result.length() > len) {
        result = QString("%1%2%3").arg(result.left(left)).arg(mid).arg(result.right(right));
    }

    return result;
}

QRect QtHelper::getCenterRect(const QSize &imageSize, const QRect &widgetRect, int borderWidth, int scaleMode)
{
    QSize newSize = imageSize;
    QSize widgetSize = widgetRect.size() - QSize(borderWidth * 1, borderWidth * 1);

    if (scaleMode == 0) {
        if (newSize.width() > widgetSize.width() || newSize.height() > widgetSize.height()) {
            newSize.scale(widgetSize, Qt::KeepAspectRatio);
        }
    } else if (scaleMode == 1) {
        newSize.scale(widgetSize, Qt::KeepAspectRatio);
    } else {
        newSize = widgetSize;
    }

    int x = widgetRect.center().x() - newSize.width() / 2;
    int y = widgetRect.center().y() - newSize.height() / 2;
    //不是2的倍数需要偏移1像素
    x += (x % 2 == 0 ? 1 : 0);
    y += (y % 2 == 0 ? 1 : 0);
    return QRect(x, y, newSize.width(), newSize.height());
}

void QtHelper::getScaledImage(QImage &image, const QSize &widgetSize, int scaleMode, bool fast)
{
    Qt::TransformationMode mode = fast ? Qt::FastTransformation : Qt::SmoothTransformation;
    if (scaleMode == 0) {
        if (image.width() > widgetSize.width() || image.height() > widgetSize.height()) {
            image = image.scaled(widgetSize, Qt::KeepAspectRatio, mode);
        }
    } else if (scaleMode == 1) {
        image = image.scaled(widgetSize, Qt::KeepAspectRatio, mode);
    } else {
        image = image.scaled(widgetSize, Qt::IgnoreAspectRatio, mode);
    }
}

QString QtHelper::getTimeString(qint64 time)
{
    time = time / 1000;
    QString min = QString("%1").arg(time / 60, 2, 10, QChar('0'));
    QString sec = QString("%2").arg(time % 60, 2, 10, QChar('0'));
    return QString("%1:%2").arg(min).arg(sec);
}

QString QtHelper::getTimeString(QElapsedTimer timer)
{
    return QString::number((float)timer.elapsed() / 1000, 'f', 3);
}

QString QtHelper::getSizeString(quint64 size)
{
    float num = size;
    QStringList list;
    list << "KB" << "MB" << "GB" << "TB";

    QString unit("bytes");
    QStringListIterator i(list);
    while (num >= 1024.0 && i.hasNext()) {
        unit = i.next();
        num /= 1024.0;
    }

    return QString("%1 %2").arg(QString::number(num, 'f', 2)).arg(unit);
}

//setSystemDateTime("2022", "07", "01", "12", "22", "55");
void QtHelper::setSystemDateTime(const QString &year, const QString &month, const QString &day, const QString &hour, const QString &min, const QString &sec)
{
#ifdef Q_OS_WIN
    QProcess p;
    //先设置日期
    p.start("cmd", QStringList());
    p.waitForStarted();
    p.write(QString("date %1-%2-%3\n").arg(year).arg(month).arg(day).toLatin1());
    p.closeWriteChannel();
    p.waitForFinished(1000);
    p.close();
    //再设置时间
    p.start("cmd", QStringList());
    p.waitForStarted();
    p.write(QString("time %1:%2:%3.00\n").arg(hour).arg(min).arg(sec).toLatin1());
    p.closeWriteChannel();
    p.waitForFinished(1000);
    p.close();
#else
    QString cmd = QString("date %1%2%3%4%5.%6").arg(month).arg(day).arg(hour).arg(min).arg(year).arg(sec);
    //设置日期时间
    system(cmd.toLatin1());
    //硬件时钟同步
    system("hwclock -w");
#endif
}

void QtHelper::runWithSystem(bool autoRun)
{
    QtHelper::runWithSystem(qApp->applicationName(), qApp->applicationFilePath(), autoRun);
}

void QtHelper::runWithSystem(const QString &fileName, const QString &filePath, bool autoRun)
{
#ifdef Q_OS_WIN
    //要转换成本地文件路径(不启动则文件路径为空即可)
    QSettings reg("HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Run", QSettings::NativeFormat);
    reg.setValue(fileName, autoRun ? QDir::toNativeSeparators(filePath) : "");
#endif
}

void QtHelper::start(const QString &path, const QString &name, bool bin)
{
#ifdef Q_OS_WIN
    QString cmd1 = "tasklist";
    QString cmd2 = QString("%1/%2%3").arg(path).arg(name).arg(bin ? ".exe" : "");
#else
    QString cmd1 = "ps -aux";
    QString cmd2 = QString("%1/%2").arg(path).arg(name);
#endif

#ifndef Q_OS_WASM
    QProcess p;
    p.start(cmd1, QStringList());
    if (p.waitForFinished()) {
        QString result = p.readAll();
        if (result.contains(name)) {
            return;
        }
    }

    //加上引号可以兼容打开带空格的目录(Program Files)
    if (cmd2.contains(" ")) {
        cmd2 = "\"" + cmd2 + "\"";
    }

    //切换到当前目录
    QDir::setCurrent(path);
    //QProcess::execute(cmd2, QStringList());
    QProcess::startDetached(cmd2, QStringList());
    //执行完成后切换回默认目录
    QDir::setCurrent(QtHelper::appPath());
#endif
}
```

### `./qthelper.h`

```cpp
﻿#ifndef QTHELPER_H
#define QTHELPER_H

#include "head.h"

class QtHelper
{
public:
    //获取所有屏幕区域/当前鼠标所在屏幕索引/区域尺寸/缩放系数
    static bool useRatio;
    static QList<QRect> getScreenRects(bool available = true, bool full = false);
    static int getScreenIndex();
    static QRect getScreenRect(bool available = true, bool full = false);
    static qreal getScreenRatio(int index = -1, bool devicePixel = false);

    //矫正当前鼠标所在屏幕居中尺寸
    static QRect checkCenterRect(QRect &rect, bool available = true);

    //获取桌面宽度高度+居中显示
    static int deskWidth();
    static int deskHeight();
    static QSize deskSize();

    //居中显示窗体
    //定义标志位指定是以桌面为参照还是主程序界面为参照
    static QWidget *centerBaseForm;
    static void setFormInCenter(QWidget *form);
    static void showForm(QWidget *form);

    //程序文件名称和当前所在路径
    static QString appName();
    static QString appPath();

    //程序最前面获取应用程序路径和名称
    static void getCurrentInfo(char *argv[], QString &path, QString &name);
    //程序最前面读取配置文件节点的值
    static QString getIniValue(const QString &fileName, const QString &key);
    static QString getIniValue(char *argv[], const QString &key, const QString &dir = QString(), const QString &file = QString());

    //获取本地网卡IP集合
    static QStringList getLocalIPs();
    //添加网卡集合并根据默认值设置当前项
    static void initLocalIPs(QComboBox *cbox, const QString &defaultIP, bool local127 = true);

    //获取内置颜色集合
    static QList<QColor> colors;
    static QList<QColor> getColorList();
    static QStringList getColorNames();
    //随机获取颜色集合中的颜色
    static QColor getRandColor();

    //初始化随机数种子
    static void initRand();
    //获取随机小数
    static float getRandFloat(float min, float max);
    //获取随机数,指定最小值和最大值
    static double getRandValue(int min, int max, bool contansMin = false, bool contansMax = false);
    //获取范围值随机经纬度集合
    static QStringList getRandPoint(int count, float mainLng, float mainLat, float dotLng, float dotLat);
    //根据旧的范围值和值计算新的范围值对应的值
    static int getRangeValue(int oldMin, int oldMax, int oldValue, int newMin, int newMax);

    //获取uuid
    static QString getUuid();
    //校验目录
    static QString checkPath(const QString &dirName);
    //转换成完整路径
    static QString checkFile(const QString &fileName);
    //通用延时函数(支持Qt4 Qt5 Qt6)
    static void sleep(int msec, bool exec = true);
    //检查程序是否已经运行
    static void checkRun();

    //设置Qt自带样式
    static void setStyle();
    //设置字体
    static QFont addFont(const QString &fontFile, const QString &fontName);
    static void setFont(int fontSize = 12);
    //设置编码
    static void setCode(bool utf8 = true);
    //设置翻译文件
    static void setTranslator(const QString &qmFile);

    //动态设置权限
    static bool checkPermission(const QString &permission);
    //申请安卓权限
    static void initAndroidPermission();

    //一次性设置所有包括编码样式字体等
    static void initAll(bool utf8 = true, bool style = true, bool tabCenter = true, int fontSize = 13);
    //初始化main函数最前面执行的一段代码
    static void initMain(bool desktopSettingsAware = false, bool use96Dpi = false, bool logCritical = true);
    //初始化opengl类型(1=AA_UseDesktopOpenGL 2=AA_UseOpenGLES 3=AA_UseSoftwareOpenGL)
    static void initOpenGL(quint8 type = 0, bool checkCardEnable = false, bool checkVirtualSystem = false);

    //读取qss文件获取样式表内容
    static QString getStyle(const QString &qssFile);
    //设置qss文件到全局样式
    static void setStyle(const QString &qssFile);

    //执行命令行返回执行结果
    static QString doCmd(const QString &program, const QStringList &arguments, int timeout = 1000);
    //获取显卡是否被禁用
    static bool isVideoCardEnable();
    //获取是否在虚拟机环境
    static bool isVirtualSystem();

    //插入消息
    static bool replaceCRLF;
    static QVector<int> msgTypes;
    static QVector<QString> msgKeys;
    static QVector<QColor> msgColors;
    static QString appendMsg(QTextEdit *textEdit, int type, const QString &data,
                             int maxCount, int &currentCount,
                             bool clear = false, bool pause = false);

    //设置无边框
    static void setFramelessForm(QWidget *widgetMain, bool tool = false, bool top = false, bool menu = true);

    //弹出框
    static int showMessageBox(const QString &text, int type = 0, int closeSec = 0, bool exec = false);
    //弹出消息框
    static void showMessageBoxInfo(const QString &text, int closeSec = 0, bool exec = false);
    //弹出错误框
    static void showMessageBoxError(const QString &text, int closeSec = 0, bool exec = false);
    //弹出询问框
    static int showMessageBoxQuestion(const QString &text);

    //为什么还要自定义对话框因为可控宽高和汉化对应文本等
    //初始化对话框文本
    static void initDialog(QFileDialog *dialog, const QString &title, const QString &acceptName,
                           const QString &dirName, bool native, int width, int height);
    //拿到对话框结果
    static QString getDialogResult(QFileDialog *dialog);
    //选择文件对话框
    static QString getOpenFileName(const QString &filter = QString(),
                                   const QString &dirName = QString(),
                                   const QString &fileName = QString(),
                                   bool native = false, int width = 900, int height = 600);
    //保存文件对话框
    static QString getSaveFileName(const QString &filter = QString(),
                                   const QString &dirName = QString(),
                                   const QString &fileName = QString(),
                                   bool native = false, int width = 900, int height = 600);
    //选择目录对话框
    static QString getExistingDirectory(const QString &dirName = QString(),
                                        bool native = false, int width = 900, int height = 600);

    //异或加密-只支持字符,如果是中文需要将其转换base64编码
    static QString getXorEncryptDecrypt(const QString &value, char key);
    //异或校验
    static quint8 getOrCode(const QByteArray &data);
    //计算校验码
    static quint8 getCheckCode(const QByteArray &data);

    //初始化表格
    static void initTableView(QTableView *tableView, int rowHeight = 25,
                              bool headVisible = false, bool edit = false,
                              bool stretchLast = true);
    //打开文件带提示框
    static void openFile(const QString &fileName, const QString &msg);

    //检查ini配置文件
    static bool checkIniFile(const QString &iniFile);

    //首尾截断字符串显示
    static QString cutString(const QString &text, int len, int left, int right, bool file, const QString &mid = "...");

    //传入图片尺寸和窗体区域及边框大小返回居中区域(scaleMode: 0-自动调整 1-等比缩放 2-拉伸填充)
    static QRect getCenterRect(const QSize &imageSize, const QRect &widgetRect, int borderWidth = 2, int scaleMode = 0);
    //传入图片尺寸和窗体尺寸及缩放策略返回合适尺寸(scaleMode: 0-自动调整 1-等比缩放 2-拉伸填充)
    static void getScaledImage(QImage &image, const QSize &widgetSize, int scaleMode = 0, bool fast = true);

    //毫秒数转时间 00:00
    static QString getTimeString(qint64 time);
    //用时时间转秒数
    static QString getTimeString(QElapsedTimer timer);
    //文件大小转 KB MB GB TB
    static QString getSizeString(quint64 size);

    //设置系统时间
    static void setSystemDateTime(const QString &year, const QString &month, const QString &day,
                                  const QString &hour, const QString &min, const QString &sec);
    //设置开机自启动
    static void runWithSystem(bool autoRun = true);
    static void runWithSystem(const QString &fileName, const QString &filePath, bool autoRun = true);

    //启动运行程序(已经在运行则不启动)
    static void start(const QString &path, const QString &name, bool bin = true);
};

#endif // QTHELPER_H
```

### `./singleton.h`

```cpp
﻿#ifndef SINGLETON_H
#define SINGLETON_H

#include <QScopedPointer>
#include <QMutex>

#define SINGLETON_DECL(Class) \
    public: \
        static Class *Instance(); \
    private: \
        Q_DISABLE_COPY(Class) \
        static QScopedPointer<Class> self;

#define SINGLETON_IMPL(Class) \
    QScopedPointer<Class> Class::self; \
    Class *Class::Instance() { \
        if (self.isNull()) { \
            static QMutex mutex; \
            QMutexLocker locker(&mutex); \
            if (self.isNull()) { \
                self.reset(new Class); \
            } \
        } \
        return self.data(); \
    }

#endif // SINGLETON_H
```

### `./wasmhelper.cpp`

```cpp
﻿#include "wasmhelper.h"
#include "qrect.h"
#include "emscripten.h"
#include "emscripten/html5.h"

//弹出js信息框
EM_JS(void, showMessageJs, (const char *text), {
    alert(UTF8ToString(text));
})

//弹出js输入框
EM_JS(const char *, getInputJs, (const char *title, const char *defaultText), {
    var result = prompt(UTF8ToString(title), UTF8ToString(defaultText));
    if (!result)
    {
        result = "";
    }
    return stringToNewUTF8(result);
})

//打开iframe窗体
EM_JS(void, openIframeJs, (const char *flag, const char *url, const char *style), {
    //如果存在则只移动位置
    var id = UTF8ToString(flag);
    var iframe = document.getElementById(id);
    if (iframe)
    {
        iframe.style = UTF8ToString(style);
        return;
    }

    iframe = document.createElement('iframe');
    iframe.id = id;
    iframe.src = UTF8ToString(url);
    iframe.style = UTF8ToString(style);
    iframe.scrolling = 'no';
    document.body.appendChild(iframe);
})

//加载iframe窗体
EM_JS(void, reloadIframeJs, (const char *flag, const char *url), {
    var iframe = document.getElementById(UTF8ToString(flag));
    if (iframe)
    {
        iframe.src = UTF8ToString(url);
    }
})

//移动iframe窗体
EM_JS(void, moveIframeJs, (const char *flag, const char *style), {
    var iframe = document.getElementById(UTF8ToString(flag));
    if (iframe)
    {
        iframe.style = UTF8ToString(style);
    }
})

//隐藏iframe窗体
EM_JS(void, hideIframeJs, (const char *flag), {
    var iframe = document.getElementById(UTF8ToString(flag));
    if (iframe)
    {
        iframe.style = "display:none";
    }
})

void WasmHelper::showMessage(const QString &text)
{
    showMessageJs(text.toUtf8().constData());
}

QString WasmHelper::getInput(const QString &title, const QString &text)
{
    return getInputJs(title.toUtf8().constData(), text.toUtf8().constData());
}

QString WasmHelper::getIframeStyle(const QRect &rect)
{
    QString style = QString("border:0px;position:absolute;margin:0px;padding:0px;z-index:10000;opacity:1.0;");
    style += QString("left:%1px;").arg(rect.x());
    style += QString("top:%1px;").arg(rect.y());
    style += QString("width:%1px;").arg(rect.width());
    style += QString("height:%1px;").arg(rect.height());
    return style;
}

void WasmHelper::openIframe(const QString &flag, const QString &url, const QRect &rect)
{
    QString style = getIframeStyle(rect);
    openIframeJs(flag.toUtf8().constData(), url.toUtf8().constData(), style.toUtf8().constData());
}

void WasmHelper::reloadIframe(const QString &flag, const QString &url)
{
    reloadIframeJs(flag.toUtf8().constData(), url.toUtf8().constData());
}

void WasmHelper::moveIframe(const QString &flag, const QRect &rect)
{
    QString style = getIframeStyle(rect);
    moveIframeJs(flag.toUtf8().constData(), style.toUtf8().constData());
}

void WasmHelper::hideIframe(const QString &flag)
{
    hideIframeJs(flag.toUtf8().constData());
}
```

### `./wasmhelper.h`

```cpp
﻿#ifndef WASMHELPER_H
#define WASMHELPER_H

#include <QObject>

class WasmHelper
{    
public:
    //弹出js信息框
    static void showMessage(const QString &text);
    //弹出js输入框
    static QString getInput(const QString &title, const QString &text);

    //获取iframe样式
    static QString getIframeStyle(const QRect &rect);
    //打开iframe窗体
    static void openIframe(const QString &flag, const QString &url, const QRect &rect);
    //重新加载iframe窗体
    static void reloadIframe(const QString &flag, const QString &url);
    //移动iframe窗体
    static void moveIframe(const QString &flag, const QRect &rect);
    //隐藏iframe窗体
    static void hideIframe(const QString &flag);
};

#endif // WASMHELPER_H
```

### `./core_helper.pri`

```makefile
QT *= network
greaterThan(QT_MAJOR_VERSION, 4) {
lessThan(QT_MAJOR_VERSION, 6) {
android {QT *= androidextras}
} else {
QT *= core-private
}}

#指定编译产生的文件分门别类放到对应目录
MOC_DIR     = temp/moc
RCC_DIR     = temp/rcc
UI_DIR      = temp/ui
OBJECTS_DIR = temp/obj

#指定编译生成的可执行文件放到源码上一级目录下的bin目录
!android:!ios {
DESTDIR = $$PWD/../bin
}

#把所有警告都关掉眼不见为净
CONFIG += warn_off
#开启大资源支持
CONFIG += resources_big
#开启后会将打印信息用控制台输出
#CONFIG += console
#开启后不会生成空的 debug release 目录
#CONFIG -= debug_and_release

include ($$PWD/core_util.pri)

#将当前目录加入到头文件路径
INCLUDEPATH += $$PWD
HEADERS += $$PWD/singleton.h

HEADERS += $$PWD/appdata.h
SOURCES += $$PWD/appdata.cpp

HEADERS += $$PWD/appinit.h
SOURCES += $$PWD/appinit.cpp

HEADERS += $$PWD/base64helper.h
SOURCES += $$PWD/base64helper.cpp

HEADERS += $$PWD/customstyle.h
SOURCES += $$PWD/customstyle.cpp

HEADERS += $$PWD/delegate.h
SOURCES += $$PWD/delegate.cpp

HEADERS += $$PWD/iconhelper.h
SOURCES += $$PWD/iconhelper.cpp

HEADERS += $$PWD/qthelper.h
SOURCES += $$PWD/qthelper.cpp

#可以指定不加载对应的资源文件
!contains(DEFINES, no_qrc_image) {
RESOURCES += $$PWD/qrc/image.qrc
}

!contains(DEFINES, no_qrc_qm) {
RESOURCES += $$PWD/qrc/qm.qrc
}

!contains(DEFINES, no_qrc_font) {
RESOURCES += $$PWD/qrc/font.qrc
}

wasm {
HEADERS += $$PWD/wasmhelper.h
SOURCES += $$PWD/wasmhelper.cpp
RESOURCES += $$PWD/qrc/wasm.qrc
}
```

### `./core_util.pri`

```makefile
#定义复制文件到目录的函数
#为什么上面不加win{}这种
#因为还有在win/linux上的安卓套件/他并不是win/linux套件
#所以两种命令都执行保证任意系统可用
defineTest(copyToDestDir) {
#取出对应的参数变量
srcFile = $$1
dstPath = $$2

#linux和mac系统拷贝
system($$QMAKE_COPY $$srcFile $$dstPath)

#win上需要转换路径
srcFile2 = $$srcFile
dstPath2 = $$dstPath
srcFile2 ~= s,/,\\,g
dstPath2 ~= s,/,\\,g
system($$QMAKE_COPY $$srcFile2 $$dstPath2)
}

#新建目录/在win上目录不存在的话需要主动新建/linux会自动
defineTest(newPath) {
path = $$1
win32 {
path ~= s,/,\\,g
}
system(mkdir $$path)
}

#引入全志H3芯片依赖(不需要的用户可以删除)
unix:!macx {
contains(QT_ARCH, arm) {
contains(DEFINES, arma7) {
INCLUDEPATH += /usr/local/openssl-1.0.2m-h3-gcc-4.9.2/include
LIBS += -L/usr/local/openssl-1.0.2m-h3-gcc-4.9.2/lib -lssl -lcrypto
LIBS += -L/usr/local/h3_rootfsv -lXdmcp
}}}
```
