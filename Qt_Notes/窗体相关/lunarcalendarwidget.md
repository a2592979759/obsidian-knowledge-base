---
tags:
  - Qt
  - 窗体相关
  - 高质量
---
# 农历控件 (`lunarcalendarwidget`)

> 农历控件

## 效果图

![[QWidgetDemo_assert/lunarcalendarwidget.jpg]]

## 分类

- 类型: 窗体相关 `widget`
- 源码目录: `widget/lunarcalendarwidget`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QColor QString QWidget QDate QList QPainter QSize QTextCodec QToolButton QSizePolicy QLabel QComboBox QFont QObject QHBoxLayout QEvent QRect QMouseEvent QStringList QChar

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmlunarcalendarwidget.cpp`

```cpp
﻿#include "frmlunarcalendarwidget.h"
#include "ui_frmlunarcalendarwidget.h"

frmLunarCalendarWidget::frmLunarCalendarWidget(QWidget *parent) : QWidget(parent), ui(new Ui::frmLunarCalendarWidget)
{
    ui->setupUi(this);
    this->initForm();
}

frmLunarCalendarWidget::~frmLunarCalendarWidget()
{
    delete ui;
}

void frmLunarCalendarWidget::initForm()
{
    ui->cboxWeekNameFormat->setCurrentIndex(2);
}

void frmLunarCalendarWidget::on_cboxCalendarStyle_currentIndexChanged(int index)
{
    ui->lunarCalendarWidget->setCalendarStyle((LunarCalendarWidget::CalendarStyle)index);
}

void frmLunarCalendarWidget::on_cboxSelectType_currentIndexChanged(int index)
{
    ui->lunarCalendarWidget->setSelectType((LunarCalendarWidget::SelectType)index);
}

void frmLunarCalendarWidget::on_cboxWeekNameFormat_currentIndexChanged(int index)
{
    ui->lunarCalendarWidget->setWeekNameFormat((LunarCalendarWidget::WeekNameFormat)index);
}

void frmLunarCalendarWidget::on_ckShowLunar_stateChanged(int arg1)
{
    ui->lunarCalendarWidget->setShowLunar(arg1 != 0);
}
```

### `./frmlunarcalendarwidget.h`

```cpp
﻿#ifndef FRMLUNARCALENDARWIDGET_H
#define FRMLUNARCALENDARWIDGET_H

#include <QWidget>

namespace Ui {
class frmLunarCalendarWidget;
}

class frmLunarCalendarWidget : public QWidget
{
    Q_OBJECT

public:
    explicit frmLunarCalendarWidget(QWidget *parent = 0);
    ~frmLunarCalendarWidget();

private:
    Ui::frmLunarCalendarWidget *ui;

private slots:
    void initForm();
    void on_cboxCalendarStyle_currentIndexChanged(int index);
    void on_cboxSelectType_currentIndexChanged(int index);
    void on_cboxWeekNameFormat_currentIndexChanged(int index);
    void on_ckShowLunar_stateChanged(int arg1);
};

#endif // FRMLUNARCALENDARWIDGET_H
```

### `./lunarcalendarinfo.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "lunarcalendarinfo.h"
#include "qmutex.h"
#include "qdebug.h"

#define year_2099

QScopedPointer<LunarCalendarInfo> LunarCalendarInfo::self;
LunarCalendarInfo *LunarCalendarInfo::Instance()
{
    if (self.isNull()) {
        static QMutex mutex;
        QMutexLocker locker(&mutex);
        if (self.isNull()) {
            self.reset(new LunarCalendarInfo);
        }
    }

    return self.data();
}

LunarCalendarInfo::LunarCalendarInfo(QObject *parent) : QObject(parent)
{
    //农历查表
    lunarCalendarTable << 0x04AE53 << 0x0A5748 << 0x5526BD << 0x0D2650 << 0x0D9544 << 0x46AAB9 << 0x056A4D << 0x09AD42 << 0x24AEB6 << 0x04AE4A; //1901-1910
    lunarCalendarTable << 0x6A4DBE << 0x0A4D52 << 0x0D2546 << 0x5D52BA << 0x0B544E << 0x0D6A43 << 0x296D37 << 0x095B4B << 0x749BC1 << 0x049754; //1911-1920
    lunarCalendarTable << 0x0A4B48 << 0x5B25BC << 0x06A550 << 0x06D445 << 0x4ADAB8 << 0x02B64D << 0x095742 << 0x2497B7 << 0x04974A << 0x664B3E; //1921-1930
    lunarCalendarTable << 0x0D4A51 << 0x0EA546 << 0x56D4BA << 0x05AD4E << 0x02B644 << 0x393738 << 0x092E4B << 0x7C96BF << 0x0C9553 << 0x0D4A48; //1931-1940
    lunarCalendarTable << 0x6DA53B << 0x0B554F << 0x056A45 << 0x4AADB9 << 0x025D4D << 0x092D42 << 0x2C95B6 << 0x0A954A << 0x7B4ABD << 0x06CA51; //1941-1950
    lunarCalendarTable << 0x0B5546 << 0x555ABB << 0x04DA4E << 0x0A5B43 << 0x352BB8 << 0x052B4C << 0x8A953F << 0x0E9552 << 0x06AA48 << 0x6AD53C; //1951-1960
    lunarCalendarTable << 0x0AB54F << 0x04B645 << 0x4A5739 << 0x0A574D << 0x052642 << 0x3E9335 << 0x0D9549 << 0x75AABE << 0x056A51 << 0x096D46; //1961-1970
    lunarCalendarTable << 0x54AEBB << 0x04AD4F << 0x0A4D43 << 0x4D26B7 << 0x0D254B << 0x8D52BF << 0x0B5452 << 0x0B6A47 << 0x696D3C << 0x095B50; //1971-1980
    lunarCalendarTable << 0x049B45 << 0x4A4BB9 << 0x0A4B4D << 0xAB25C2 << 0x06A554 << 0x06D449 << 0x6ADA3D << 0x0AB651 << 0x093746 << 0x5497BB; //1981-1990
    lunarCalendarTable << 0x04974F << 0x064B44 << 0x36A537 << 0x0EA54A << 0x86B2BF << 0x05AC53 << 0x0AB647 << 0x5936BC << 0x092E50 << 0x0C9645; //1991-2000
    lunarCalendarTable << 0x4D4AB8 << 0x0D4A4C << 0x0DA541 << 0x25AAB6 << 0x056A49 << 0x7AADBD << 0x025D52 << 0x092D47 << 0x5C95BA << 0x0A954E; //2001-2010
    lunarCalendarTable << 0x0B4A43 << 0x4B5537 << 0x0AD54A << 0x955ABF << 0x04BA53 << 0x0A5B48 << 0x652BBC << 0x052B50 << 0x0A9345 << 0x474AB9; //2011-2020
    lunarCalendarTable << 0x06AA4C << 0x0AD541 << 0x24DAB6 << 0x04B64A << 0x69573D << 0x0A4E51 << 0x0D2646 << 0x5E933A << 0x0D534D << 0x05AA43; //2021-2030
    lunarCalendarTable << 0x36B537 << 0x096D4B << 0xB4AEBF << 0x04AD53 << 0x0A4D48 << 0x6D25BC << 0x0D254F << 0x0D5244 << 0x5DAA38 << 0x0B5A4C; //2031-2040
    lunarCalendarTable << 0x056D41 << 0x24ADB6 << 0x049B4A << 0x7A4BBE << 0x0A4B51 << 0x0AA546 << 0x5B52BA << 0x06D24E << 0x0ADA42 << 0x355B37; //2041-2050
    lunarCalendarTable << 0x09374B << 0x8497C1 << 0x049753 << 0x064B48 << 0x66A53C << 0x0EA54F << 0x06B244 << 0x4AB638 << 0x0AAE4C << 0x092E42; //2051-2060
    lunarCalendarTable << 0x3C9735 << 0x0C9649 << 0x7D4ABD << 0x0D4A51 << 0x0DA545 << 0x55AABA << 0x056A4E << 0x0A6D43 << 0x452EB7 << 0x052D4B; //2061-2070
    lunarCalendarTable << 0x8A95BF << 0x0A9553 << 0x0B4A47 << 0x6B553B << 0x0AD54F << 0x055A45 << 0x4A5D38 << 0x0A5B4C << 0x052B42 << 0x3A93B6; //2071-2080
    lunarCalendarTable << 0x069349 << 0x7729BD << 0x06AA51 << 0x0AD546 << 0x54DABA << 0x04B64E << 0x0A5743 << 0x452738 << 0x0D264A << 0x8E933E; //2081-2090
    lunarCalendarTable << 0x0D5252 << 0x0DAA47 << 0x66B53B << 0x056D4F << 0x04AE45 << 0x4A4EB9 << 0x0A4D4C << 0x0D1541 << 0x2D92B5;             //2091-2099

    //每年春节对应的公历日期
    springFestival << 130 << 217 << 206;                                                                // 1968 1969 1970
    springFestival << 127 << 215 << 203 << 123 << 211 << 131 << 218 << 207 << 128 << 216;               // 1971--1980
    springFestival << 205 << 125 << 213 << 202 << 220 << 209 << 219 << 217 << 206 << 127;               // 1981--1990
    springFestival << 215 << 204 << 123 << 210 << 131 << 219 << 207 << 128 << 216 << 205;               // 1991--2000
    springFestival << 124 << 212 << 201 << 122 << 209 << 129 << 218 << 207 << 126 << 214;               // 2001--2010
    springFestival << 203 << 123 << 210 << 131 << 219 << 208 << 128 << 216 << 205 << 125;               // 2011--2020
    springFestival << 212 << 201 << 122 << 210 << 129 << 217 << 206 << 126 << 213 << 203;               // 2021--2030
    springFestival << 123 << 211 << 131 << 219 << 208 << 128 << 215 << 204 << 124 << 212;               // 2031--2040

    //16--18位表示闰几月 0--12位表示农历每月的数据 高位表示1月 低位表示12月(农历闰月就会多一个月)
    lunarData << 461653 << 1386 << 2413;                                                                // 1968 1969 1970
    lunarData << 330077 << 1197 << 2637 << 268877 << 3365 << 531109 << 2900 << 2922 << 398042 << 2395;  // 1971--1980
    lunarData << 1179 << 267415 << 2635 << 661067 << 1701 << 1748 << 398772 << 2742 << 2391 << 330031;  // 1981--1990
    lunarData << 1175 << 1611 << 200010 << 3749 << 527717 << 1452 << 2742 << 332397 << 2350 << 3222;    // 1991--2000
    lunarData << 268949 << 3402 << 3493 << 133973 << 1386 << 464219 << 605 << 2349 << 334123 << 2709;   // 2001--2010
    lunarData << 2890 << 267946 << 2773 << 592565 << 1210 << 2651 << 395863 << 1323 << 2707 << 265877;  // 2011--2020
    lunarData << 1706 << 2773 << 133557 << 1206 << 398510 << 2638 << 3366 << 335142 << 3411 << 1450;    // 2021--2030
    lunarData << 200042 << 2413 << 723293 << 1197 << 2637 << 399947 << 3365 << 3410 << 334676 << 2906;  // 2031--2040

    //二十四节气表
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x69 << 0x78 << 0x87; // 1970
    chineseTwentyFourData << 0x96 << 0xB4 << 0x96 << 0xA6 << 0x97 << 0x97 << 0x78 << 0x79 << 0x79 << 0x69 << 0x78 << 0x77; // 1971
    chineseTwentyFourData << 0x96 << 0xA4 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 1972
    chineseTwentyFourData << 0xA5 << 0xB5 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x78 << 0x78 << 0x78 << 0x87 << 0x87; // 1973
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x69 << 0x78 << 0x87; // 1974
    chineseTwentyFourData << 0x96 << 0xB4 << 0x96 << 0xA6 << 0x97 << 0x97 << 0x78 << 0x79 << 0x78 << 0x69 << 0x78 << 0x77; // 1975
    chineseTwentyFourData << 0x96 << 0xA4 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x88 << 0x89 << 0x88 << 0x78 << 0x87 << 0x87; // 1976
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 1977
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x79 << 0x78 << 0x87; // 1978
    chineseTwentyFourData << 0x96 << 0xB4 << 0x96 << 0xA6 << 0x96 << 0x97 << 0x78 << 0x79 << 0x78 << 0x69 << 0x78 << 0x77; // 1979
    chineseTwentyFourData << 0x96 << 0xA4 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 1980
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x77 << 0x87; // 1981
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x79 << 0x77 << 0x87; // 1982
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x78 << 0x79 << 0x78 << 0x69 << 0x78 << 0x77; // 1983
    chineseTwentyFourData << 0x96 << 0xB4 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 1984
    chineseTwentyFourData << 0xA5 << 0xB4 << 0xA6 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 1985
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x79 << 0x77 << 0x87; // 1986
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x79 << 0x78 << 0x69 << 0x78 << 0x87; // 1987
    chineseTwentyFourData << 0x96 << 0xB4 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 1988
    chineseTwentyFourData << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 1989
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x79 << 0x77 << 0x87; // 1990
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x86 << 0x97 << 0x88 << 0x78 << 0x78 << 0x69 << 0x78 << 0x87; // 1991
    chineseTwentyFourData << 0x96 << 0xB4 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 1992
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 1993
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x78 << 0x87 << 0x87; // 1994
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x76 << 0x78 << 0x69 << 0x78 << 0x87; // 1995
    chineseTwentyFourData << 0x96 << 0xB4 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 1996
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 1997
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x78 << 0x87 << 0x87; // 1998
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x69 << 0x78 << 0x87; // 1999
    chineseTwentyFourData << 0x96 << 0xB4 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2000
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2001
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x78 << 0x87 << 0x87; // 2002
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x69 << 0x78 << 0x87; // 2003
    chineseTwentyFourData << 0x96 << 0xB4 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2004
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2005
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2006
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x69 << 0x78 << 0x87; // 2007
    chineseTwentyFourData << 0x96 << 0xB4 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x87 << 0x78 << 0x87 << 0x86; // 2008
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2009
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2010
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x79 << 0x78 << 0x87; // 2011
    chineseTwentyFourData << 0x96 << 0xB4 << 0xA5 << 0xB5 << 0xA5 << 0xA6 << 0x87 << 0x88 << 0x87 << 0x78 << 0x87 << 0x86; // 2012
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2013
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2014
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x79 << 0x77 << 0x87; // 2015
    chineseTwentyFourData << 0x95 << 0xB4 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x87 << 0x88 << 0x87 << 0x78 << 0x87 << 0x86; // 2016
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2017
    chineseTwentyFourData << 0xA5 << 0xB4 << 0xA6 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2018
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x79 << 0x77 << 0x87; // 2019
    chineseTwentyFourData << 0x95 << 0xB4 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x86; // 2020
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2021
    chineseTwentyFourData << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2022
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x79 << 0x77 << 0x87; // 2023
    chineseTwentyFourData << 0x95 << 0xB4 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2024
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2025
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2026
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x78 << 0x87 << 0x87; // 2027
    chineseTwentyFourData << 0x95 << 0xB4 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2028
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2029
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2030
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x78 << 0x87 << 0x87; // 2031
    chineseTwentyFourData << 0x95 << 0xB4 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2032
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2033
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x78 << 0x88 << 0x78 << 0x87 << 0x87; // 2034
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2035
    chineseTwentyFourData << 0x95 << 0xB4 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2036
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2037
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2038
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2039
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x69 << 0x78 << 0x87; // 2040
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA5 << 0xA6 << 0x87 << 0x88 << 0x87 << 0x78 << 0x87 << 0x86; // 2041
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2042
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2043
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x97 << 0x88 << 0x78 << 0x78 << 0x79 << 0x78 << 0x87; // 2044
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x87 << 0x88 << 0x87 << 0x78 << 0x87 << 0x86; // 2045
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2046
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2047
    chineseTwentyFourData << 0x95 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x79 << 0x77 << 0x87; // 2048
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x86; // 2049
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2050
    chineseTwentyFourData << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2051
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x79 << 0x77 << 0x87; // 2052
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x86; // 2053
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2054
    chineseTwentyFourData << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2055
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x79 << 0x77 << 0x87; // 2056
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2057
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2058
    chineseTwentyFourData << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2059
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x78 << 0x87 << 0x87; // 2060
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2061
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2062
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2063
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0x96 << 0x96 << 0x88 << 0x78 << 0x78 << 0x78 << 0x87 << 0x87; // 2064
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2065
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2066
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2067
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2068
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2069
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA5 << 0xA6 << 0x87 << 0x88 << 0x87 << 0x78 << 0x87 << 0x86; // 2070
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2071
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2072
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x88 << 0x87 << 0x96; // 2073
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA5 << 0xA6 << 0x87 << 0x88 << 0x87 << 0x78 << 0x87 << 0x86; // 2074
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2075
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2076
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x88 << 0x87 << 0x96; // 2077
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x88 << 0x87 << 0x78 << 0x87 << 0x86; // 2078
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2079
    chineseTwentyFourData << 0xA5 << 0xB4 << 0x96 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x78 << 0x78 << 0x87 << 0x87; // 2080
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0x97 << 0x87 << 0x87 << 0x88 << 0x86 << 0x96; // 2081
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x86; // 2082
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2083
    chineseTwentyFourData << 0xA5 << 0xB4 << 0xA6 << 0xA5 << 0xA6 << 0x96 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2084
    chineseTwentyFourData << 0xB4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0x97 << 0x87 << 0x87 << 0x88 << 0x86 << 0x96; // 2085
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x86; // 2086
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2087
    chineseTwentyFourData << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2088
    chineseTwentyFourData << 0xB4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0x97 << 0x87 << 0x87 << 0x88 << 0x96 << 0x96; // 2089
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2090
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2091
    chineseTwentyFourData << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2092
    chineseTwentyFourData << 0xB4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0x97 << 0x87 << 0x87 << 0x87 << 0x96 << 0x96; // 2093
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2094
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2095
    chineseTwentyFourData << 0xA5 << 0xB3 << 0xA5 << 0xA5 << 0xA6 << 0xA6 << 0x88 << 0x88 << 0x88 << 0x78 << 0x87 << 0x87; // 2096
    chineseTwentyFourData << 0xB4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA5 << 0x97 << 0x97 << 0x87 << 0x87 << 0x96 << 0x96; // 2097
    chineseTwentyFourData << 0xA4 << 0xC3 << 0xA5 << 0xB4 << 0xA5 << 0xA6 << 0x97 << 0x87 << 0x87 << 0x78 << 0x87 << 0x96; // 2098
    chineseTwentyFourData << 0xA5 << 0xC3 << 0xA5 << 0xB5 << 0xA6 << 0xA6 << 0x87 << 0x88 << 0x88 << 0x78 << 0x87 << 0x86; // 2099

    //公历每月前面的天数
    monthAdd << 0 << 31 << 59 << 90 << 120 << 151 << 181 << 212 << 243 << 273 << 304 << 334;

    //农历日期名称
    listDayName << "*" << "初一" << "初二" << "初三" << "初四" << "初五" << "初六" << "初七" << "初八" << "初九" << "初十"
                << "十一" << "十二" << "十三" << "十四" << "十五" << "十六" << "十七" << "十八" << "十九" << "二十"
                << "廿一" << "廿二" << "廿三" << "廿四" << "廿五" << "廿六" << "廿七" << "廿八" << "廿九" << "三十";

    //农历月份名称
    listMonthName << "*" << "正月" << "二月" << "三月" << "四月" << "五月" << "六月" << "七月" << "八月" << "九月" << "十月" << "冬月" << "腊月";

    //二十四节气名称
    listSolarTerm << "小寒" << "大寒" << "立春" << "雨水" << "惊蛰" << "春分" << "清明" << "谷雨"
                  << "立夏" << "小满" << "芒种" << "夏至" << "小暑" << "大暑" << "立秋" << "处暑"
                  << "白露" << "秋分" << "寒露" << "霜降" << "立冬" << "小雪" << "大雪" << "冬至";

    //天干
    listTianGan << "甲" << "乙" << "丙" << "丁" << "戊" << "己" << "庚" << "辛" << "壬" << "癸";

    //地支
    listDiZhi << "子" << "丑" << "寅" << "卯" << "辰" << "巳" << "午" << "未" << "申" << "酉" << "戌" << "亥";

    //属相
    listShuXiang << "鼠" << "牛" << "虎" << "兔" << "龙" << "蛇" << "马" << "羊" << "猴" << "鸡" << "狗" << "猪";
}

//计算是否闰年
bool LunarCalendarInfo::isLoopYear(int year)
{
    return (((0 == (year % 4)) && (0 != (year % 100))) || (0 == (year % 400)));
}

//计算指定年月该月共多少天
int LunarCalendarInfo::getMonthDays(int year, int month)
{
    int countDay = 0;
    int loopDay = isLoopYear(year) ? 1 : 0;

    switch (month) {
        case 1:
            countDay = 31;
            break;
        case 2:
            countDay = 28 + loopDay;
            break;
        case 3:
            countDay = 31;
            break;
        case 4:
            countDay = 30;
            break;
        case 5:
            countDay = 31;
            break;
        case 6:
            countDay = 30;
            break;
        case 7:
            countDay = 31;
            break;
        case 8:
            countDay = 31;
            break;
        case 9:
            countDay = 30;
            break;
        case 10:
            countDay = 31;
            break;
        case 11:
            countDay = 30;
            break;
        case 12:
            countDay = 31;
            break;
        default:
            countDay = 30;
            break;
    }

    return countDay;
}

//计算指定年月对应到该月共多少天
int LunarCalendarInfo::getTotalMonthDays(int year, int month)
{
    int countDay = 0;
    int loopDay = isLoopYear(year) ? 1 : 0;

    switch (month) {
        case 1:
            countDay = 0;
            break;
        case 2:
            countDay = 31;
            break;
        case 3:
            countDay = 59 + loopDay;
            break;
        case 4:
            countDay = 90 + loopDay;
            break;
        case 5:
            countDay = 120 + loopDay;
            break;
        case 6:
            countDay = 151 + loopDay;
            break;
        case 7:
            countDay = 181 + loopDay;
            break;
        case 8:
            countDay = 212 + loopDay;
            break;
        case 9:
            countDay = 243 + loopDay;
            break;
        case 10:
            countDay = 273 + loopDay;
            break;
        case 11:
            countDay = 304 + loopDay;
            break;
        case 12:
            countDay = 334 + loopDay;
            break;
        default:
            countDay = 0;
            break;
    }

    return countDay;
}

//计算指定年月对应星期几
int LunarCalendarInfo::getFirstDayOfWeek(int year, int month)
{
    int week = 0;
    week = (year + (year - 1) / 4 - (year - 1) / 100 + (year - 1) / 400) % 7;
    week += getTotalMonthDays(year, month);
    week = week % 7;
    return week;
}

//计算国际节日
QString LunarCalendarInfo::getHoliday(int month, int day)
{
    int temp = (month << 8) | day;
    QString strHoliday;

    switch (temp) {
        case 0x0101:
            strHoliday = "元旦";
            break;
        case 0x020E:
            strHoliday = "情人节";
            break;
        case 0x0303:
            strHoliday = "爱耳日";
            break;
        case 0x0305:
            strHoliday = "志愿者服务日";
            break;
        case 0x0308:
            strHoliday = "妇女节";
            break;
        case 0x0309:
            strHoliday = "保护母亲河";
            break;
        case 0x030C:
            strHoliday = "植树节";
            break;
        case 0x030F:
            strHoliday = "消费者权益日";
            break;
        case 0x0401:
            strHoliday = "愚人节";
            break;
        case 0x0501:
            strHoliday = "劳动节";
            break;
        case 0x0504:
            strHoliday = "青年节";
            break;
        case 0x0601:
            strHoliday = "儿童节";
            break;
        case 0x0606:
            strHoliday = "全国爱眼日";
            break;
        case 0x0701:
            strHoliday = "建党节";
            break;
        case 0x0707:
            strHoliday = "抗战纪念日";
            break;
        case 0x0801:
            strHoliday = "建军节";
            break;
        case 0x090A:
            strHoliday = "教师节";
            break;
        case 0x0910:
            strHoliday = "脑健康日";
            break;
        case 0x0914:
            strHoliday = "爱牙日";
            break;
        case 0x0A01:
            strHoliday = "国庆节";
            break;
        case 0x0A0A:
            strHoliday = "高血压日";
            break;
        case 0x0A1C:
            strHoliday = "男性健康日";
            break;
        case 0x0B08:
            strHoliday = "记者节";
            break;
        case 0x0B09:
            strHoliday = "消防宣传日";
            break;
        case 0x0C04:
            strHoliday = "法制宣传日";
            break;
        case 0x0C18:
            strHoliday = "平安夜";
            break;
        case 0x0C19:
            strHoliday = "圣诞节";
            break;
        default:
            break;
    }

    return strHoliday;
}

//计算二十四节气
QString LunarCalendarInfo::getSolarTerms(int year, int month, int day)
{
    QString strSolarTerms;
    int dayTemp = 0;
    //24节气对应表在1970年以后指定?
    int index = (year - 1970) * 12 + month - 1;
    if (index < 0) {
        return "";
    }

    if (day < 15) {
        dayTemp = 15 - day;
        if ((chineseTwentyFourData.at(index) >> 4) == dayTemp) {
            strSolarTerms = listSolarTerm.at(2 * (month - 1));
        }
    } else if (day > 15) {
        dayTemp = day - 15;
        if ((chineseTwentyFourData.at(index) & 0x0f) == dayTemp) {
            strSolarTerms = listSolarTerm.at(2 * (month - 1) + 1);
        }
    }

    return strSolarTerms;
}

//计算农历节日(必须传入农历年份月份)
QString LunarCalendarInfo::getLunarFestival(int month, int day)
{
    int temp = (month << 8) | day;
    QString strFestival;

    switch (temp) {
        case 0x0101:
            strFestival = "春节";
            break;
        case 0x010F:
            strFestival = "元宵节";
            break;
        case 0x0202:
            strFestival = "龙抬头";
            break;
        case 0x0505:
            strFestival = "端午节";
            break;
        case 0x0707:
            strFestival = "七夕节";
            break;
        case 0x080F:
            strFestival = "中秋节";
            break;
        case 0x0909:
            strFestival = "重阳节";
            break;
        case 0x0C08:
            strFestival = "腊八节";
            break;
        case 0x0C1E:
            strFestival = "除夕";
            break;
        default:
            break;
    }

    return strFestival;
}

//计算农历年 天干+地支+生肖
QString LunarCalendarInfo::getLunarYear(int year)
{
    QString strYear;
    if (year > 1924) {
        int temp = year - 1924;
        strYear.append(listTianGan.at(temp % 10));
        strYear.append(listDiZhi.at(temp % 12));
        strYear.append(listShuXiang.at(temp % 12));
    }

    return strYear;
}

void LunarCalendarInfo::getLunarCalendarInfo(int year, int month, int day,
        QString &strHoliday,
        QString &strSolarTerms,
        QString &strLunarFestival,
        QString &strLunarYear,
        QString &strLunarMonth,
        QString &strLunarDay)
{
    //过滤不在范围内的年月日
    if (year < 1901 || year > 2099 || month < 1 || month > 12 || day < 1 || day > 31) {
        return;
    }

    strHoliday = getHoliday(month, day);
    strSolarTerms = getSolarTerms(year, month, day);

#ifndef year_2099
    //现在计算农历:获得当年春节的公历日期(比如：2015年春节日期为(2月19日))
    //以此为分界点,2.19前面的农历是2014年农历(用2014年农历数据来计算)
    //2.19以及之后的日期，农历为2015年农历(用2015年农历数据来计算)
    int temp, tempYear, tempMonth, isEnd, k, n;
    tempMonth = year - 1968;
    int springFestivalMonth = springFestival.at(tempMonth) / 100;
    int springFestivalDaty = springFestival.at(tempMonth) % 100;

    if (month < springFestivalMonth) {
        tempMonth--;
        tempYear = 365 * 1 + day + monthAdd.at(month - 1) - 31 * ((springFestival.at(tempMonth) / 100) - 1) - springFestival.at(tempMonth) % 100 + 1;

        if ((!(year % 4)) && (month > 2)) {
            tempYear = tempYear + 1;
        }

        if ((!((year - 1) % 4))) {
            tempYear = tempYear + 1;
        }
    } else if (month == springFestivalMonth) {
        if (day < springFestivalDaty) {
            tempMonth--;
            tempYear = 365 * 1 + day + monthAdd.at(month - 1) - 31 * ((springFestival.at(tempMonth) / 100) - 1) - springFestival.at(tempMonth) % 100 + 1;

            if ((!(year % 4)) && (month > 2)) {
                tempYear = tempYear + 1;
            }

            if ((!((year - 1) % 4))) {
                tempYear = tempYear + 1;
            }
        } else {
            tempYear = day + monthAdd.at(month - 1) - 31 * ((springFestival.at(tempMonth) / 100) - 1) - springFestival.at(tempMonth) % 100 + 1;

            if ((!(year % 4)) && (month > 2)) {
                tempYear = tempYear + 1;
            }
        }
    } else {
        tempYear = day + monthAdd.at(month - 1) - 31 * ((springFestival.at(tempMonth) / 100) - 1) - springFestival.at(tempMonth) % 100 + 1;

        if ((!(year % 4)) && (month > 2)) {
            tempYear = tempYear + 1;
        }
    }

    //计算农历天干地支月日
    isEnd = 0;
    while (isEnd != 1)  {
        if (lunarData.at(tempMonth) < 4095) {
            k = 11;
        } else {
            k = 12;
        }

        n = k;

        while (n >= 0)   {
            temp = lunarData.at(tempMonth);
            temp = temp >> n;
            temp = temp % 2;

            if (tempYear <= (29 + temp))    {
                isEnd = 1;
                break;
            }

            tempYear = tempYear - 29 - temp;
            n = n - 1;
        }

        if (isEnd) {
            break;
        }

        tempMonth = tempMonth + 1;
    }

    //农历的年月日
    year = 1969 + tempMonth - 1;
    month = k - n + 1;
    day = tempYear;

    if (k == 12)  {
        if (month == (lunarData.at(tempMonth) / 65536) + 1) {
            month = 1 - month;
        } else if (month > (lunarData.at(tempMonth) / 65536) + 1) {
            month = month - 1;
        }
    }

    strLunarYear = getLunarYear(year);

    if (month < 1 && (1 == day)) {
        month = month * -1;
        strLunarMonth = "闰" + listMonthName.at(month);
    } else {
        strLunarMonth = listMonthName.at(month);
    }

    strLunarDay = listDayName.at(day);
    strLunarFestival = getLunarFestival(month, day);
#else
    //记录春节离当年元旦的天数
    int springOffset = 0;
    //记录阳历日离当年元旦的天数
    int newYearOffset = 0;
    //记录大小月的天数 29或30
    int monthCount = 0;

    if (((lunarCalendarTable.at(year - 1901) & 0x0060) >> 5) == 1) {
        springOffset = (lunarCalendarTable.at(year - 1901) & 0x001F) - 1;
    } else {
        springOffset = (lunarCalendarTable.at(year - 1901) & 0x001F) - 1 + 31;
    }

    //如果是不闰年且不是2月份 +1
    newYearOffset = monthAdd.at(month - 1) + day - 1;
    if ((!(year % 4)) && (month > 2)) {
        newYearOffset++;
    }

    //记录从哪个月开始来计算
    int index = 0;
    //用来对闰月的特殊处理
    int flag = 0;

    //判断阳历日在春节前还是春节后,计算农历的月/日
    if (newYearOffset >= springOffset) {
        newYearOffset -= springOffset;
        month = 1;
        index = 1;
        flag = 0;

        if ((lunarCalendarTable.at(year - 1901) & (0x80000 >> (index - 1))) == 0) {
            monthCount = 29;
        } else {
            monthCount = 30;
        }

        while (newYearOffset >= monthCount) {
            newYearOffset -= monthCount;
            index++;
            if (month == ((lunarCalendarTable.at(year - 1901) & 0xF00000) >> 20)) {
                flag = ~flag;
                if (flag == 0) {
                    month++;
                }
            } else {
                month++;
            }

            if ((lunarCalendarTable.at(year - 1901) & (0x80000 >> (index - 1))) == 0) {
                monthCount = 29;
            } else {
                monthCount = 30;
            }
        }

        day = newYearOffset + 1;
    } else {
        springOffset -= newYearOffset;
        year--;
        month = 12;
        flag = 0;

        if (((lunarCalendarTable.at(year - 1901) & 0xF00000) >> 20) == 0) {
            index = 12;
        } else {
            index = 13;
        }

        if ((lunarCalendarTable.at(year - 1901) & (0x80000 >> (index - 1))) == 0) {
            monthCount = 29;
        } else {
            monthCount = 30;
        }

        while (springOffset > monthCount) {
            springOffset -= monthCount;
            index--;

            if (flag == 0) {
                month--;
            }

            if (month == ((lunarCalendarTable.at(year - 1901) & 0xF00000) >> 20)) {
                flag = ~flag;
            }

            if ((lunarCalendarTable.at(year - 1901) & (0x80000 >> (index - 1))) == 0) {
                monthCount = 29;
            } else {
                monthCount = 30;
            }
        }

        day = monthCount - springOffset + 1;
    }

    //计算农历的索引配置
    int temp = 0;
    temp |= day;
    temp |= (month << 6);

    //转换农历的年月
    month = (temp & 0x3C0) >> 6;
    day = temp & 0x3F;

    strLunarYear = getLunarYear(year);

    if ((month == ((lunarCalendarTable.at(year - 1901) & 0xF00000) >> 20)) && (1 == day)) {
        strLunarMonth = "闰" + listMonthName.at(month);
    } else {
        strLunarMonth = listMonthName.at(month);
    }

    strLunarDay = listDayName.at(day);
    strLunarFestival = getLunarFestival(month, day);
#endif
}

QString LunarCalendarInfo::getLunarInfo(int year, int month, int day, bool yearInfo, bool monthInfo, bool dayInfo)
{
    QString strHoliday, strSolarTerms, strLunarFestival, strLunarYear, strLunarMonth, strLunarDay;

    LunarCalendarInfo::Instance()->getLunarCalendarInfo(year, month, day,
            strHoliday, strSolarTerms, strLunarFestival,
            strLunarYear, strLunarMonth, strLunarDay);

    //农历节日优先,其次农历节气,然后公历节日,最后才是农历月份名称
    if (!strLunarFestival.isEmpty()) {
        strLunarDay = strLunarFestival;
    } else if (!strSolarTerms.isEmpty()) {
        strLunarDay = strSolarTerms;
    } else if (!strHoliday.isEmpty()) {
        strLunarDay = strHoliday;
    }

    QString info = QString("%1%2%3")
                   .arg(yearInfo ? strLunarYear + "年" : "")
                   .arg(monthInfo ? strLunarMonth : "")
                   .arg(dayInfo ? strLunarDay : "");

    return info;
}

QString LunarCalendarInfo::getLunarYearMonthDay(int year, int month, int day)
{
    return getLunarInfo(year, month, day, true, true, true);
}

QString LunarCalendarInfo::getLunarMonthDay(int year, int month, int day)
{
    return getLunarInfo(year, month, day, false, true, true);
}

QString LunarCalendarInfo::getLunarDay(int year, int month, int day)
{
    return getLunarInfo(year, month, day, false, false, true);
}
```

### `./lunarcalendarinfo.h`

```cpp
﻿#ifndef LUNARCALENDARINFO_H
#define LUNARCALENDARINFO_H

/**
 * 农历信息类 作者:倪大侠 整理:feiyangqingyun(QQ:517216493) 2016-12-10
 * 1. 计算是否闰年。
 * 2. 计算国际节日。
 * 3. 计算二十四节气。
 * 4. 计算农历年 天干、地支、生肖。
 * 5. 计算指定年月日农历信息，包括公历节日和农历节日及二十四节气。
 */

#include <QObject>

#ifdef quc
class Q_DECL_EXPORT LunarCalendarInfo : public QObject
#else
class LunarCalendarInfo : public QObject
#endif

{
    Q_OBJECT
public:
    static LunarCalendarInfo *Instance();
    explicit LunarCalendarInfo(QObject *parent = 0);

    //计算是否闰年
    bool isLoopYear(int year);

    //计算指定年月该月共多少天
    int getMonthDays(int year, int month);

    //计算指定年月对应到该月共多少天
    int getTotalMonthDays(int year, int month);

    //计算指定年月对应星期几
    int getFirstDayOfWeek(int year, int month);

    //计算国际节日
    QString getHoliday(int month, int day);

    //计算二十四节气
    QString getSolarTerms(int year, int month, int day);

    //计算农历节日(必须传入农历年份月份)
    QString getLunarFestival(int month, int day);

    //计算农历年 天干+地支+生肖
    QString getLunarYear(int year);

    //计算指定年月日农历信息,包括公历节日和农历节日及二十四节气
    void getLunarCalendarInfo(int year, int month, int day,
                              QString &strHoliday,
                              QString &strSolarTerms,
                              QString &strLunarFestival,
                              QString &strLunarYear,
                              QString &strLunarMonth,
                              QString &strLunarDay);

    //获取指定年月日农历信息
    QString getLunarInfo(int year, int month, int day, bool yearInfo, bool monthInfo, bool dayInfo);
    QString getLunarYearMonthDay(int year, int month, int day);
    QString getLunarMonthDay(int year, int month, int day);
    QString getLunarDay(int year, int month, int day);

private:
    static QScopedPointer<LunarCalendarInfo> self;

    QList<int> lunarCalendarTable;      //农历年表
    QList<int> springFestival;          //春节公历日期
    QList<int> lunarData;               //农历每月数据
    QList<int> chineseTwentyFourData;   //农历二十四节气数据
    QList<int> monthAdd;                //公历每月前面的天数

    QList<QString> listDayName;         //农历日期名称集合
    QList<QString> listMonthName;       //农历月份名称集合
    QList<QString> listSolarTerm;       //二十四节气名称集合

    QList<QString> listTianGan;         //天干名称集合
    QList<QString> listDiZhi;           //地支名称集合
    QList<QString> listShuXiang;        //属相名称集合
};

#endif // LUNARCALENDARINFO_H
```

### `./lunarcalendaritem.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "lunarcalendaritem.h"
#include "qpainter.h"
#include "qevent.h"
#include "qdatetime.h"
#include "qdebug.h"

LunarCalendarItem::LunarCalendarItem(QWidget *parent) : QWidget(parent)
{
    hover = false;
    pressed = false;
    listDayName << "*" << "初一" << "初二" << "初三" << "初四" << "初五" << "初六" << "初七" << "初八" << "初九" << "初十"
                << "十一" << "十二" << "十三" << "十四" << "十五" << "十六" << "十七" << "十八" << "十九" << "二十"
                << "廿一" << "廿二" << "廿三" << "廿四" << "廿五" << "廿六" << "廿七" << "廿八" << "廿九" << "三十";

    select = false;
    showLunar = true;
    bgImage = ":/image/bg_calendar.png";
    selectType = SelectType_Rect;

    date = QDate::currentDate();
    lunar = "初一";
    dayType = DayType_MonthCurrent;

    borderColor = QColor(180, 180, 180);
    weekColor = QColor(255, 0, 0);
    superColor = QColor(255, 129, 6);
    lunarColor = QColor(55, 156, 238);

    currentTextColor = QColor(0, 0, 0);
    otherTextColor = QColor(200, 200, 200);
    selectTextColor = QColor(255, 255, 255);
    hoverTextColor = QColor(250, 250, 250);

    currentLunarColor = QColor(150, 150, 150);
    otherLunarColor = QColor(200, 200, 200);
    selectLunarColor = QColor(255, 255, 255);
    hoverLunarColor = QColor(250, 250, 250);

    currentBgColor = QColor(255, 255, 255);
    otherBgColor = QColor(240, 240, 240);
    selectBgColor = QColor(208, 47, 18);
    hoverBgColor = QColor(204, 183, 180);
}

void LunarCalendarItem::enterEvent(EnterEvent *)
{
    hover = true;
    this->update();
}

void LunarCalendarItem::leaveEvent(QEvent *)
{
    hover = false;
    this->update();
}

void LunarCalendarItem::mousePressEvent(QMouseEvent *)
{
    pressed = true;
    this->update();

    Q_EMIT clicked(date, dayType);
}

void LunarCalendarItem::mouseReleaseEvent(QMouseEvent *)
{
    pressed = false;
    this->update();
}

void LunarCalendarItem::paintEvent(QPaintEvent *)
{
    //绘制准备工作,启用反锯齿
    QPainter painter(this);
    painter.setRenderHints(QPainter::Antialiasing | QPainter::TextAntialiasing);

    //绘制背景和边框
    drawBg(&painter);

    //优先绘制选中状态,其次绘制悬停状态
    if (select) {
        drawBgCurrent(&painter, selectBgColor);
    } else if (hover) {
        drawBgCurrent(&painter, hoverBgColor);
    }

    //绘制日期
    drawDay(&painter);

    //绘制农历信息
    drawLunar(&painter);
}

void LunarCalendarItem::drawBg(QPainter *painter)
{
    painter->save();

    //根据当前类型选择对应的颜色
    QColor bgColor = currentBgColor;
    if (dayType == DayType_MonthPre || dayType == DayType_MonthNext) {
        bgColor = otherBgColor;
    }

    painter->setPen(borderColor);
    painter->setBrush(bgColor);
    painter->drawRect(rect());

    painter->restore();
}

void LunarCalendarItem::drawBgCurrent(QPainter *painter, const QColor &color)
{
    int width = this->width();
    int height = this->height();
    int side = qMin(width, height);

    painter->save();

    painter->setPen(Qt::NoPen);
    painter->setBrush(color);

    //根据设定绘制背景样式
    if (selectType == SelectType_Rect) {
        painter->drawRect(rect());
    } else if (selectType == SelectType_Circle) {
        int radius = side / 2 - 3;
        painter->drawEllipse(QPointF(width / 2, height / 2), radius, radius);
    } else if (selectType == SelectType_Triangle) {
        int radius = side / 3;
        QPolygon pts;
        pts.setPoints(3, 1, 1, radius, 1, 1, radius);
        painter->drawRect(rect());
        painter->setBrush(superColor);
        painter->drawConvexPolygon(pts);
    } else if (selectType == SelectType_Image) {
        //等比例缩放居中绘制
        QImage img(bgImage);
        if (!img.isNull()) {
            img = img.scaled(this->size(), Qt::KeepAspectRatio, Qt::SmoothTransformation);
            int x = (width - img.width()) / 2;
            int y = (height - img.height()) / 2;
            painter->drawImage(x, y, img);
        }
    }

    painter->restore();
}

void LunarCalendarItem::drawDay(QPainter *painter)
{
    int width = this->width();
    int height = this->height();
    int side = qMin(width, height);

    painter->save();

    //根据当前类型选择对应的颜色
    QColor color = currentTextColor;
    if (dayType == DayType_MonthPre || dayType == DayType_MonthNext) {
        color = otherTextColor;
    } else if (dayType == DayType_WeekEnd) {
        color = weekColor;
    }

    if (select) {
        color = selectTextColor;
    } else if (hover) {
        color = hoverTextColor;
    }

    painter->setPen(color);

    if (showLunar) {
        QFont font;
        font.setPixelSize(side / 2.7);
        painter->setFont(font);

        QRect dayRect = QRect(0, 0, width, height / 1.7);
        painter->drawText(dayRect, Qt::AlignHCenter | Qt::AlignBottom, QString::number(date.day()));
    } else {
        QFont font;
        font.setPixelSize(side / 2);
        painter->setFont(font);

        QRect dayRect = QRect(0, 0, width, height);
        painter->drawText(dayRect, Qt::AlignCenter, QString::number(date.day()));
    }

    painter->restore();
}

void LunarCalendarItem::drawLunar(QPainter *painter)
{
    if (!showLunar) {
        return;
    }

    int width = this->width();
    int height = this->height();
    int side = qMin(width, height);

    painter->save();

    //判断当前农历文字是否节日,是节日且是当月则用农历节日颜色显示
    bool exist = (!listDayName.contains(lunar) && dayType != DayType_MonthPre && dayType != DayType_MonthNext);

    //根据当前类型选择对应的颜色
    QColor color = currentLunarColor;
    if (dayType == DayType_MonthPre || dayType == DayType_MonthNext) {
        color = otherLunarColor;
    }

    if (select) {
        color = selectTextColor;
    } else if (hover) {
        color = hoverTextColor;
    } else if (exist) {
        color = lunarColor;
    }

    painter->setPen(color);

    QFont font;
    font.setPixelSize(side / 5);
    painter->setFont(font);

    QRect lunarRect(0, height / 2, width, height / 2);
    painter->drawText(lunarRect, Qt::AlignCenter, lunar);

    painter->restore();
}

QSize LunarCalendarItem::sizeHint() const
{
    return QSize(100, 100);
}

QSize LunarCalendarItem::minimumSizeHint() const
{
    return QSize(20, 20);
}

bool LunarCalendarItem::getSelect() const
{
    return this->select;
}

void LunarCalendarItem::setSelect(bool select)
{
    if (this->select != select) {
        this->select = select;
        this->update();
    }
}

bool LunarCalendarItem::getShowLunar() const
{
    return this->showLunar;
}

void LunarCalendarItem::setShowLunar(bool showLunar)
{
    if (this->showLunar != showLunar) {
        this->showLunar = showLunar;
        this->update();
    }
}

QString LunarCalendarItem::getBgImage() const
{
    return this->bgImage;
}

void LunarCalendarItem::setBgImage(const QString &bgImage)
{
    if (this->bgImage != bgImage) {
        this->bgImage = bgImage;
        this->update();
    }
}

LunarCalendarItem::SelectType LunarCalendarItem::getSelectType() const
{
    return this->selectType;
}

void LunarCalendarItem::setSelectType(const LunarCalendarItem::SelectType &selectType)
{
    if (this->selectType != selectType) {
        this->selectType = selectType;
        this->update();
    }
}

QDate LunarCalendarItem::getDate() const
{
    return this->date;
}

void LunarCalendarItem::setDate(const QDate &date)
{
    if (this->date != date) {
        this->date = date;
        this->update();
    }
}

QString LunarCalendarItem::getLunar() const
{
    return this->lunar;
}

void LunarCalendarItem::setLunar(const QString &lunar)
{
    if (this->lunar != lunar) {
        this->lunar = lunar;
        this->update();
    }
}

LunarCalendarItem::DayType LunarCalendarItem::getDayType() const
{
    return this->dayType;
}

void LunarCalendarItem::setDayType(const LunarCalendarItem::DayType &dayType)
{
    if (this->dayType != dayType) {
        this->dayType = dayType;
        this->update();
    }
}

void LunarCalendarItem::setDate(const QDate &date, const QString &lunar, const DayType &dayType)
{
    this->date = date;
    this->lunar = lunar;
    this->dayType = dayType;
    this->update();
}

QColor LunarCalendarItem::getBorderColor() const
{
    return this->borderColor;
}

void LunarCalendarItem::setBorderColor(const QColor &borderColor)
{
    if (this->borderColor != borderColor) {
        this->borderColor = borderColor;
        this->update();
    }
}


QColor LunarCalendarItem::getWeekColor() const
{
    return this->weekColor;
}

void LunarCalendarItem::setWeekColor(const QColor &weekColor)
{
    if (this->weekColor != weekColor) {
        this->weekColor = weekColor;
        this->update();
    }
}

QColor LunarCalendarItem::getSuperColor() const
{
    return this->superColor;
}

void LunarCalendarItem::setSuperColor(const QColor &superColor)
{
    if (this->superColor != superColor) {
        this->superColor = superColor;
        this->update();
    }
}

QColor LunarCalendarItem::getLunarColor() const
{
    return this->lunarColor;
}

void LunarCalendarItem::setLunarColor(const QColor &lunarColor)
{
    if (this->lunarColor != lunarColor) {
        this->lunarColor = lunarColor;
        this->update();
    }
}

QColor LunarCalendarItem::getCurrentTextColor() const
{
    return this->currentTextColor;
}

void LunarCalendarItem::setCurrentTextColor(const QColor &currentTextColor)
{
    if (this->currentTextColor != currentTextColor) {
        this->currentTextColor = currentTextColor;
        this->update();
    }
}

QColor LunarCalendarItem::getOtherTextColor() const
{
    return this->otherTextColor;
}

void LunarCalendarItem::setOtherTextColor(const QColor &otherTextColor)
{
    if (this->otherTextColor != otherTextColor) {
        this->otherTextColor = otherTextColor;
        this->update();
    }
}

QColor LunarCalendarItem::getSelectTextColor() const
{
    return this->selectTextColor;
}

void LunarCalendarItem::setSelectTextColor(const QColor &selectTextColor)
{
    if (this->selectTextColor != selectTextColor) {
        this->selectTextColor = selectTextColor;
        this->update();
    }
}

QColor LunarCalendarItem::getHoverTextColor() const
{
    return this->hoverTextColor;
}

void LunarCalendarItem::setHoverTextColor(const QColor &hoverTextColor)
{
    if (this->hoverTextColor != hoverTextColor) {
        this->hoverTextColor = hoverTextColor;
        this->update();
    }
}

QColor LunarCalendarItem::getCurrentLunarColor() const
{
    return this->currentLunarColor;
}

void LunarCalendarItem::setCurrentLunarColor(const QColor &currentLunarColor)
{
    if (this->currentLunarColor != currentLunarColor) {
        this->currentLunarColor = currentLunarColor;
        this->update();
    }
}


QColor LunarCalendarItem::getOtherLunarColor() const
{
    return this->otherLunarColor;
}

void LunarCalendarItem::setOtherLunarColor(const QColor &otherLunarColor)
{
    if (this->otherLunarColor != otherLunarColor) {
        this->otherLunarColor = otherLunarColor;
        this->update();
    }
}

QColor LunarCalendarItem::getSelectLunarColor() const
{
    return this->selectLunarColor;
}

void LunarCalendarItem::setSelectLunarColor(const QColor &selectLunarColor)
{
    if (this->selectLunarColor != selectLunarColor) {
        this->selectLunarColor = selectLunarColor;
        this->update();
    }
}

QColor LunarCalendarItem::getHoverLunarColor() const
{
    return this->hoverLunarColor;
}

void LunarCalendarItem::setHoverLunarColor(const QColor &hoverLunarColor)
{
    if (this->hoverLunarColor != hoverLunarColor) {
        this->hoverLunarColor = hoverLunarColor;
        this->update();
    }
}

QColor LunarCalendarItem::getCurrentBgColor() const
{
    return this->currentBgColor;
}

void LunarCalendarItem::setCurrentBgColor(const QColor &currentBgColor)
{
    if (this->currentBgColor != currentBgColor) {
        this->currentBgColor = currentBgColor;
        this->update();
    }
}


QColor LunarCalendarItem::getOtherBgColor() const
{
    return this->otherBgColor;
}

void LunarCalendarItem::setOtherBgColor(const QColor &otherBgColor)
{
    if (this->otherBgColor != otherBgColor) {
        this->otherBgColor = otherBgColor;
        this->update();
    }
}

QColor LunarCalendarItem::getSelectBgColor() const
{
    return this->selectBgColor;
}

void LunarCalendarItem::setSelectBgColor(const QColor &selectBgColor)
{
    if (this->selectBgColor != selectBgColor) {
        this->selectBgColor = selectBgColor;
        this->update();
    }
}

QColor LunarCalendarItem::getHoverBgColor() const
{
    return this->hoverBgColor;
}

void LunarCalendarItem::setHoverBgColor(const QColor &hoverBgColor)
{
    if (this->hoverBgColor != hoverBgColor) {
        this->hoverBgColor = hoverBgColor;
        this->update();
    }
}
```

### `./lunarcalendaritem.h`

```cpp
﻿#ifndef LUNARCALENDARITEM_H
#define LUNARCALENDARITEM_H

#include <QWidget>
#include <QDate>

#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
#define EnterEvent QEnterEvent
#else
#define EnterEvent QEvent
#endif

#ifdef quc
class Q_DECL_EXPORT LunarCalendarItem : public QWidget
#else
class LunarCalendarItem : public QWidget
#endif

{
    Q_OBJECT
    Q_ENUMS(DayType)
    Q_ENUMS(SelectType)

    Q_PROPERTY(bool select READ getSelect WRITE setSelect)
    Q_PROPERTY(bool showLunar READ getShowLunar WRITE setShowLunar)
    Q_PROPERTY(QString bgImage READ getBgImage WRITE setBgImage)
    Q_PROPERTY(SelectType selectType READ getSelectType WRITE setSelectType)

    Q_PROPERTY(QDate date READ getDate WRITE setDate)
    Q_PROPERTY(QString lunar READ getLunar WRITE setLunar)
    Q_PROPERTY(DayType dayType READ getDayType WRITE setDayType)

    Q_PROPERTY(QColor borderColor READ getBorderColor WRITE setBorderColor)
    Q_PROPERTY(QColor weekColor READ getWeekColor WRITE setWeekColor)
    Q_PROPERTY(QColor superColor READ getSuperColor WRITE setSuperColor)
    Q_PROPERTY(QColor lunarColor READ getLunarColor WRITE setLunarColor)

    Q_PROPERTY(QColor currentTextColor READ getCurrentTextColor WRITE setCurrentTextColor)
    Q_PROPERTY(QColor otherTextColor READ getOtherTextColor WRITE setOtherTextColor)
    Q_PROPERTY(QColor selectTextColor READ getSelectTextColor WRITE setSelectTextColor)
    Q_PROPERTY(QColor hoverTextColor READ getHoverTextColor WRITE setHoverTextColor)

    Q_PROPERTY(QColor currentLunarColor READ getCurrentLunarColor WRITE setCurrentLunarColor)
    Q_PROPERTY(QColor otherLunarColor READ getOtherLunarColor WRITE setOtherLunarColor)
    Q_PROPERTY(QColor selectLunarColor READ getSelectLunarColor WRITE setSelectLunarColor)
    Q_PROPERTY(QColor hoverLunarColor READ getHoverLunarColor WRITE setHoverLunarColor)

    Q_PROPERTY(QColor currentBgColor READ getCurrentBgColor WRITE setCurrentBgColor)
    Q_PROPERTY(QColor otherBgColor READ getOtherBgColor WRITE setOtherBgColor)
    Q_PROPERTY(QColor selectBgColor READ getSelectBgColor WRITE setSelectBgColor)
    Q_PROPERTY(QColor hoverBgColor READ getHoverBgColor WRITE setHoverBgColor)

public:
    enum DayType {
        DayType_MonthPre = 0,       //上月剩余天数
        DayType_MonthNext = 1,      //下个月的天数
        DayType_MonthCurrent = 2,   //当月天数
        DayType_WeekEnd = 3         //周末
    };

    enum SelectType {
        SelectType_Rect = 0,        //矩形背景
        SelectType_Circle = 1,      //圆形背景
        SelectType_Triangle = 2,    //带三角标
        SelectType_Image = 3        //图片背景
    };

    explicit LunarCalendarItem(QWidget *parent = 0);

protected:
    void enterEvent(EnterEvent *);
    void leaveEvent(QEvent *);
    void mousePressEvent(QMouseEvent *);
    void mouseReleaseEvent(QMouseEvent *);
    void paintEvent(QPaintEvent *);
    void drawBg(QPainter *painter);
    void drawBgCurrent(QPainter *painter, const QColor &color);
    void drawDay(QPainter *painter);
    void drawLunar(QPainter *painter);

private:
    bool hover;                 //鼠标是否悬停
    bool pressed;               //鼠标是否按下
    QStringList listDayName;    //农历日期

    bool select;                //是否选中
    bool showLunar;             //显示农历
    QString bgImage;            //背景图片
    SelectType selectType;      //选中模式

    QDate date;                 //当前日期
    QString lunar;              //农历信息
    DayType dayType;            //当前日类型

    QColor borderColor;         //边框颜色
    QColor weekColor;           //周末颜色
    QColor superColor;          //角标颜色
    QColor lunarColor;          //农历节日颜色

    QColor currentTextColor;    //当前月文字颜色
    QColor otherTextColor;      //其他月文字颜色
    QColor selectTextColor;     //选中日期文字颜色
    QColor hoverTextColor;      //悬停日期文字颜色

    QColor currentLunarColor;   //当前月农历文字颜色
    QColor otherLunarColor;     //其他月农历文字颜色
    QColor selectLunarColor;    //选中日期农历文字颜色
    QColor hoverLunarColor;     //悬停日期农历文字颜色

    QColor currentBgColor;      //当前月背景颜色
    QColor otherBgColor;        //其他月背景颜色
    QColor selectBgColor;       //选中日期背景颜色
    QColor hoverBgColor;        //悬停日期背景颜色

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //获取和设置是否选中
    bool getSelect() const;
    void setSelect(bool select);

    //获取和设置是否显示农历信息
    bool getShowLunar() const;
    void setShowLunar(bool showLunar);

    //获取和设置背景图片
    QString getBgImage() const;
    void setBgImage(const QString &bgImage);

    //获取和设置选中背景样式
    SelectType getSelectType() const;
    void setSelectType(const SelectType &selectType);

    //获取和设置日期
    QDate getDate() const;
    void setDate(const QDate &date);

    //获取和设置农历
    QString getLunar() const;
    void setLunar(const QString &lunar);

    //获取和设置类型
    DayType getDayType() const;
    void setDayType(const DayType &dayType);

    //设置日期/农历/类型
    void setDate(const QDate &date, const QString &lunar, const DayType &dayType);

    //获取和设置边框颜色
    QColor getBorderColor() const;
    void setBorderColor(const QColor &borderColor);

    //获取和设置周末颜色
    QColor getWeekColor() const;
    void setWeekColor(const QColor &weekColor);

    //获取和设置角标颜色
    QColor getSuperColor() const;
    void setSuperColor(const QColor &superColor);

    //获取和设置农历节日颜色
    QColor getLunarColor() const;
    void setLunarColor(const QColor &lunarColor);

    //获取和设置当前月文字颜色
    QColor getCurrentTextColor() const;
    void setCurrentTextColor(const QColor &currentTextColor);

    //获取和设置其他月文字颜色
    QColor getOtherTextColor() const;
    void setOtherTextColor(const QColor &otherTextColor);

    //获取和设置选中日期文字颜色
    QColor getSelectTextColor() const;
    void setSelectTextColor(const QColor &selectTextColor);

    //获取和设置悬停日期文字颜色
    QColor getHoverTextColor() const;
    void setHoverTextColor(const QColor &hoverTextColor);

    //获取和设置当前月农历文字颜色
    QColor getCurrentLunarColor() const;
    void setCurrentLunarColor(const QColor &currentLunarColor);

    //获取和设置其他月农历文字颜色
    QColor getOtherLunarColor() const;
    void setOtherLunarColor(const QColor &otherLunarColor);

    //获取和设置选中日期农历文字颜色
    QColor getSelectLunarColor() const;
    void setSelectLunarColor(const QColor &selectLunarColor);

    //获取和设置悬停日期农历文字颜色
    QColor getHoverLunarColor() const;
    void setHoverLunarColor(const QColor &hoverLunarColor);

    //获取和设置当前月背景颜色
    QColor getCurrentBgColor() const;
    void setCurrentBgColor(const QColor &currentBgColor);

    //获取和设置其他月背景颜色
    QColor getOtherBgColor() const;
    void setOtherBgColor(const QColor &otherBgColor);

    //获取和设置选中日期背景颜色
    QColor getSelectBgColor() const;
    void setSelectBgColor(const QColor &selectBgColor);

    //获取和设置悬停日期背景颜色
    QColor getHoverBgColor() const;
    void setHoverBgColor(const QColor &hoverBgColor);

Q_SIGNALS:
    void clicked(const QDate &date, const LunarCalendarItem::DayType &dayType);
};

#endif // LUNARCALENDARITEM_H
```

### `./lunarcalendarwidget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "lunarcalendarwidget.h"
#include "qfontdatabase.h"
#include "qdatetime.h"
#include "qlayout.h"
#include "qlabel.h"
#include "qpushbutton.h"
#include "qtoolbutton.h"
#include "qcombobox.h"
#include "qdebug.h"

LunarCalendarWidget::LunarCalendarWidget(QWidget *parent) : QWidget(parent)
{
    //判断图形字体是否存在,不存在则加入
    QFontDatabase fontDb;
    if (!fontDb.families().contains("FontAwesome")) {
        int fontId = fontDb.addApplicationFont(":/font/fontawesome-webfont.ttf");
        QStringList fontName = fontDb.applicationFontFamilies(fontId);
        if (fontName.count() == 0) {
            qDebug() << "load fontawesome-webfont.ttf error";
        }
    }

    if (fontDb.families().contains("FontAwesome")) {
        iconFont = QFont("FontAwesome");
#if (QT_VERSION >= QT_VERSION_CHECK(4,8,0))
        iconFont.setHintingPreference(QFont::PreferNoHinting);
#endif
    }

    btnClick = false;

    calendarStyle = CalendarStyle_Red;
    weekNameFormat = WeekNameFormat_Short;
    date = QDate::currentDate();

    weekTextColor = QColor(255, 255, 255);
    weekBgColor = QColor(22, 160, 134);

    showLunar = true;
    bgImage = ":/image/bg_calendar.png";
    selectType = SelectType_Rect;

    borderColor = QColor(180, 180, 180);
    weekColor = QColor(255, 0, 0);
    superColor = QColor(255, 129, 6);
    lunarColor = QColor(55, 156, 238);

    currentTextColor = QColor(0, 0, 0);
    otherTextColor = QColor(200, 200, 200);
    selectTextColor = QColor(255, 255, 255);
    hoverTextColor = QColor(250, 250, 250);

    currentLunarColor = QColor(150, 150, 150);
    otherLunarColor = QColor(200, 200, 200);
    selectLunarColor = QColor(255, 255, 255);
    hoverLunarColor = QColor(250, 250, 250);

    currentBgColor = QColor(255, 255, 255);
    otherBgColor = QColor(240, 240, 240);
    selectBgColor = QColor(208, 47, 18);
    hoverBgColor = QColor(204, 183, 180);

    initWidget();
    initStyle();
    initDate();
}

LunarCalendarWidget::~LunarCalendarWidget()
{
}

void LunarCalendarWidget::initWidget()
{
    setObjectName("lunarCalendarWidget");

    //顶部widget
    QWidget *widgetTop = new QWidget;
    widgetTop->setObjectName("widgetTop");
    widgetTop->setMinimumHeight(35);

    //上一年按钮
    QToolButton *btnPrevYear = new QToolButton;
    btnPrevYear->setObjectName("btnPrevYear");
    btnPrevYear->setFixedWidth(35);
    btnPrevYear->setSizePolicy(QSizePolicy::Minimum, QSizePolicy::Expanding);
    btnPrevYear->setFont(iconFont);
    btnPrevYear->setText(QChar(0xf060));

    //下一年按钮
    QToolButton *btnNextYear = new QToolButton;
    btnNextYear->setObjectName("btnNextYear");
    btnNextYear->setFixedWidth(35);
    btnNextYear->setSizePolicy(QSizePolicy::Minimum, QSizePolicy::Expanding);
    btnNextYear->setFont(iconFont);
    btnNextYear->setText(QChar(0xf061));

    //上个月按钮
    QToolButton *btnPrevMonth = new QToolButton;
    btnPrevMonth->setObjectName("btnPrevMonth");
    btnPrevMonth->setFixedWidth(35);
    btnPrevMonth->setSizePolicy(QSizePolicy::Preferred, QSizePolicy::Expanding);
    btnPrevMonth->setFont(iconFont);
    btnPrevMonth->setText(QChar(0xf060));

    //下个月按钮
    QToolButton *btnNextMonth = new QToolButton;
    btnNextMonth->setObjectName("btnNextMonth");
    btnNextMonth->setFixedWidth(35);
    btnNextMonth->setSizePolicy(QSizePolicy::Preferred, QSizePolicy::Expanding);
    btnNextMonth->setFont(iconFont);
    btnNextMonth->setText(QChar(0xf061));

    //转到今天
    QPushButton *btnToday = new QPushButton;
    btnToday->setObjectName("btnToday");
    btnToday->setSizePolicy(QSizePolicy::Preferred, QSizePolicy::Expanding);
    btnToday->setText("转到今天");

    //年份下拉框
    cboxYear = new QComboBox;
    cboxYear->setObjectName("cboxYear");
    for (int i = 1901; i <= 2099; ++i) {
        cboxYear->addItem(QString("%1年").arg(i));
    }

    //月份下拉框
    cboxMonth = new QComboBox;
    cboxMonth->setObjectName("cboxMonth");
    for (int i = 1; i <= 12; ++i) {
        cboxMonth->addItem(QString("%1月").arg(i));
    }

    //中间用个空widget隔开
    QWidget *widgetBlank1 = new QWidget;
    widgetBlank1->setFixedWidth(5);
    QWidget *widgetBlank2 = new QWidget;
    widgetBlank2->setFixedWidth(5);

    //顶部横向布局
    QHBoxLayout *layoutTop = new QHBoxLayout(widgetTop);
    layoutTop->setContentsMargins(0, 0, 0, 9);
    layoutTop->addWidget(btnPrevYear);
    layoutTop->addWidget(cboxYear);
    layoutTop->addWidget(btnNextYear);
    layoutTop->addWidget(widgetBlank1);

    layoutTop->addWidget(btnPrevMonth);
    layoutTop->addWidget(cboxMonth);
    layoutTop->addWidget(btnNextMonth);
    layoutTop->addWidget(widgetBlank2);
    layoutTop->addWidget(btnToday);

    //星期widget
    QWidget *widgetWeek = new QWidget;
    widgetWeek->setObjectName("widgetWeek");
    widgetWeek->setMinimumHeight(30);

    //星期布局
    QHBoxLayout *layoutWeek = new QHBoxLayout(widgetWeek);
    layoutWeek->setContentsMargins(0, 0, 0, 0);
    layoutWeek->setSpacing(0);

    for (int i = 0; i < 7; ++i) {
        QLabel *lab = new QLabel;
        lab->setAlignment(Qt::AlignCenter);
        layoutWeek->addWidget(lab);
        labWeeks.append(lab);
    }

    setWeekNameFormat(WeekNameFormat_Long);

    //日期标签widget
    QWidget *widgetBody = new QWidget;
    widgetBody->setObjectName("widgetBody");

    //日期标签布局
    QGridLayout *layoutBody = new QGridLayout(widgetBody);
    layoutBody->setContentsMargins(1, 1, 1, 1);
    layoutBody->setHorizontalSpacing(0);
    layoutBody->setVerticalSpacing(0);

    //逐个添加日标签
    for (int i = 0; i < 42; ++i) {
        LunarCalendarItem *item = new LunarCalendarItem;
        connect(item, SIGNAL(clicked(QDate, LunarCalendarItem::DayType)), this, SLOT(clicked(QDate, LunarCalendarItem::DayType)));
        layoutBody->addWidget(item, i / 7, i % 7);
        items << item;
    }

    //主布局
    QVBoxLayout *verLayoutCalendar = new QVBoxLayout(this);
    verLayoutCalendar->setContentsMargins(0, 0, 0, 0);
    verLayoutCalendar->setSpacing(0);
    verLayoutCalendar->addWidget(widgetTop);
    verLayoutCalendar->addWidget(widgetWeek);
    verLayoutCalendar->addWidget(widgetBody, 1);

    //绑定按钮和下拉框信号
    connect(btnPrevYear, SIGNAL(clicked(bool)), this, SLOT(showPreviousYear()));
    connect(btnNextYear, SIGNAL(clicked(bool)), this, SLOT(showNextYear()));
    connect(btnPrevMonth, SIGNAL(clicked(bool)), this, SLOT(showPreviousMonth()));
    connect(btnNextMonth, SIGNAL(clicked(bool)), this, SLOT(showNextMonth()));
    connect(btnToday, SIGNAL(clicked(bool)), this, SLOT(showToday()));
    connect(cboxYear, SIGNAL(currentIndexChanged(int)), this, SLOT(yearChanged(int)));
    connect(cboxMonth, SIGNAL(currentIndexChanged(int)), this, SLOT(monthChanged(int)));
}

void LunarCalendarWidget::initStyle()
{
    //设置样式
    QStringList qss;

    //星期名称样式
    qss.append(QString("QLabel{background:%1;color:%2;}").arg(weekBgColor.name()).arg(weekTextColor.name()));

    //边框
    qss.append(QString("QWidget#widgetBody{border:1px solid %1;}").arg(borderColor.name()));

    //顶部下拉框
    qss.append(QString("QToolButton{padding:0px;background:none;border:none;border-radius:5px;}"));
    qss.append(QString("QToolButton:hover{background:#16A085;color:#FFFFFF;}"));

    //转到今天
    qss.append(QString("QPushButton{background:#16A085;color:#FFFFFF;border-radius:5px;}"));

    //自定义日控件颜色
    QString strSelectType;
    if (selectType == SelectType_Rect) {
        strSelectType = "SelectType_Rect";
    } else if (selectType == SelectType_Circle) {
        strSelectType = "SelectType_Circle";
    } else if (selectType == SelectType_Triangle) {
        strSelectType = "SelectType_Triangle";
    } else if (selectType == SelectType_Image) {
        strSelectType = "SelectType_Image";
    }

    qss.append(QString("LunarCalendarItem{qproperty-showLunar:%1;}").arg(showLunar));
    qss.append(QString("LunarCalendarItem{qproperty-bgImage:%1;}").arg(bgImage));
    qss.append(QString("LunarCalendarItem{qproperty-selectType:%1;}").arg(strSelectType));
    qss.append(QString("LunarCalendarItem{qproperty-borderColor:%1;}").arg(borderColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-weekColor:%1;}").arg(weekColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-superColor:%1;}").arg(superColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-lunarColor:%1;}").arg(lunarColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-currentTextColor:%1;}").arg(currentTextColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-otherTextColor:%1;}").arg(otherTextColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-selectTextColor:%1;}").arg(selectTextColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-hoverTextColor:%1;}").arg(hoverTextColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-currentLunarColor:%1;}").arg(currentLunarColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-otherLunarColor:%1;}").arg(otherLunarColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-selectLunarColor:%1;}").arg(selectLunarColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-hoverLunarColor:%1;}").arg(hoverLunarColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-currentBgColor:%1;}").arg(currentBgColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-otherBgColor:%1;}").arg(otherBgColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-selectBgColor:%1;}").arg(selectBgColor.name()));
    qss.append(QString("LunarCalendarItem{qproperty-hoverBgColor:%1;}").arg(hoverBgColor.name()));

    this->setStyleSheet(qss.join(""));
}

//初始化日期面板
void LunarCalendarWidget::initDate()
{
    int year = date.year();
    int month = date.month();
    int day = date.day();

    //设置为今天,设置变量防止重复触发
    btnClick = true;
    cboxYear->setCurrentIndex(cboxYear->findText(QString("%1年").arg(year)));
    cboxMonth->setCurrentIndex(cboxMonth->findText(QString("%1月").arg(month)));
    btnClick = false;

    //首先判断当前月的第一天是星期几
    int week = LunarCalendarInfo::Instance()->getFirstDayOfWeek(year, month);
    //当前月天数
    int countDay = LunarCalendarInfo::Instance()->getMonthDays(year, month);
    //上月天数
    int countDayPre = LunarCalendarInfo::Instance()->getMonthDays(1 == month ? year - 1 : year, 1 == month ? 12 : month - 1);

    //如果上月天数上月刚好一周则另外处理
    int startPre, endPre, startNext, endNext, index, tempYear, tempMonth, tempDay;
    if (0 == week) {
        startPre = 0;
        endPre = 7;
        startNext = 0;
        endNext = 42 - (countDay + 7);
    } else {
        startPre = 0;
        endPre = week;
        startNext = week + countDay;
        endNext = 42;
    }

    //纠正1月份前面部分偏差,1月份前面部分是上一年12月份
    tempYear = year;
    tempMonth = month - 1;
    if (tempMonth < 1) {
        tempYear--;
        tempMonth = 12;
    }

    //显示上月天数
    for (int i = startPre; i < endPre; ++i) {
        index = i;
        tempDay = countDayPre - endPre + i + 1;

        QDate date(tempYear, tempMonth, tempDay);
        QString lunar = LunarCalendarInfo::Instance()->getLunarDay(tempYear, tempMonth, tempDay);
        items.at(index)->setDate(date, lunar, LunarCalendarItem::DayType_MonthPre);
    }

    //纠正12月份后面部分偏差,12月份后面部分是下一年1月份
    tempYear = year;
    tempMonth = month + 1;
    if (tempMonth > 12) {
        tempYear++;
        tempMonth = 1;
    }

    //显示下月天数
    for (int i = startNext; i < endNext; ++i) {
        index = 42 - endNext + i;
        tempDay = i - startNext + 1;

        QDate date(tempYear, tempMonth, tempDay);
        QString lunar = LunarCalendarInfo::Instance()->getLunarDay(tempYear, tempMonth, tempDay);
        items.at(index)->setDate(date, lunar, LunarCalendarItem::DayType_MonthNext);
    }

    //重新置为当前年月
    tempYear = year;
    tempMonth = month;

    //显示当前月
    for (int i = week; i < (countDay + week); ++i) {
        index = (0 == week ? (i + 7) : i);
        tempDay = i - week + 1;

        QDate date(tempYear, tempMonth, tempDay);
        QString lunar = LunarCalendarInfo::Instance()->getLunarDay(tempYear, tempMonth, tempDay);
        if (0 == (i % 7) || 6 == (i % 7)) {
            items.at(index)->setDate(date, lunar, LunarCalendarItem::DayType_WeekEnd);
        } else {
            items.at(index)->setDate(date, lunar, LunarCalendarItem::DayType_MonthCurrent);
        }
    }

    dayChanged(this->date);
}

void LunarCalendarWidget::yearChanged(int)
{
    //如果是单击按钮切换的日期变动则不需要触发
    if (btnClick) {
        return;
    }

    QString arg1 = cboxYear->currentText();
    int year = arg1.mid(0, arg1.length() - 1).toInt();
    int month = date.month();
    int day = date.day();
    dateChanged(year, month, day);
}

void LunarCalendarWidget::monthChanged(int)
{
    //如果是单击按钮切换的日期变动则不需要触发
    if (btnClick) {
        return;
    }

    QString arg1 = cboxMonth->currentText();
    int year = date.year();
    int month = arg1.mid(0, arg1.length() - 1).toInt();
    int day = date.day();
    dateChanged(year, month, day);
}

void LunarCalendarWidget::clicked(const QDate &date, const LunarCalendarItem::DayType &dayType)
{
    if (LunarCalendarItem::DayType_MonthPre == dayType) {
        showPreviousMonth();
    } else if (LunarCalendarItem::DayType_MonthNext == dayType) {
        showNextMonth();
    } else {
        this->date = date;
        dayChanged(this->date);
    }
}

void LunarCalendarWidget::dayChanged(const QDate &date)
{
    //计算星期几,当前天对应标签索引=日期+星期几-1
    int year = date.year();
    int month = date.month();
    int day = date.day();
    int week = LunarCalendarInfo::Instance()->getFirstDayOfWeek(year, month);
    //qDebug() << QString("%1-%2-%3").arg(year).arg(month).arg(day);

    //选中当前日期,其他日期恢复,这里还有优化空间,比方说类似单选框机制
    for (int i = 0; i < 42; ++i) {
        //当月第一天是星期天要另外计算
        int index = day + week - 1;
        if (week == 0) {
            index = day + 6;
        }

        items.at(i)->setSelect(i == index);
    }

    //发送日期单击信号
    Q_EMIT clicked(date);
    //发送日期更新信号
    Q_EMIT selectionChanged();
}

void LunarCalendarWidget::dateChanged(int year, int month, int day)
{
    //如果原有天大于28则设置为1,防止出错
    date.setDate(year, month, day > 28 ? 1 : day);
    initDate();
}

QSize LunarCalendarWidget::sizeHint() const
{
    return QSize(600, 500);
}

QSize LunarCalendarWidget::minimumSizeHint() const
{
    return QSize(200, 150);
}

LunarCalendarWidget::CalendarStyle LunarCalendarWidget::getCalendarStyle() const
{
    return this->calendarStyle;
}

void LunarCalendarWidget::setCalendarStyle(const LunarCalendarWidget::CalendarStyle &calendarStyle)
{
    if (this->calendarStyle != calendarStyle) {
        this->calendarStyle = calendarStyle;
    }
}

LunarCalendarWidget::WeekNameFormat LunarCalendarWidget::getWeekNameFormat() const
{
    return this->weekNameFormat;
}

void LunarCalendarWidget::setWeekNameFormat(const LunarCalendarWidget::WeekNameFormat &weekNameFormat)
{
    if (this->weekNameFormat != weekNameFormat) {
        this->weekNameFormat = weekNameFormat;

        QStringList listWeek;
        if (weekNameFormat == WeekNameFormat_Short) {
            listWeek << "日" << "一" << "二" << "三" << "四" << "五" << "六";
        } else if (weekNameFormat == WeekNameFormat_Normal) {
            listWeek << "周日" << "周一" << "周二" << "周三" << "周四" << "周五" << "周六";
        } else if (weekNameFormat == WeekNameFormat_Long) {
            listWeek << "星期天" << "星期一" << "星期二" << "星期三" << "星期四" << "星期五" << "星期六";
        } else if (weekNameFormat == WeekNameFormat_En) {
            listWeek << "Sun" << "Mon" << "Tue" << "Wed" << "Thu" << "Fri" << "Sat";
        }

        //逐个添加日期文字
        for (int i = 0; i < 7; ++i) {
            labWeeks.at(i)->setText(listWeek.at(i));
        }
    }
}

QDate LunarCalendarWidget::getDate() const
{
    return this->date;
}

void LunarCalendarWidget::setDate(const QDate &date)
{
    if (this->date != date) {
        this->date = date;
        initDate();
    }
}

QColor LunarCalendarWidget::getWeekTextColor() const
{
    return this->weekTextColor;
}

void LunarCalendarWidget::setWeekTextColor(const QColor &weekTextColor)
{
    if (this->weekTextColor != weekTextColor) {
        this->weekTextColor = weekTextColor;
        this->initStyle();
    }
}


QColor LunarCalendarWidget::getWeekBgColor() const
{
    return this->weekBgColor;
}

void LunarCalendarWidget::setWeekBgColor(const QColor &weekBgColor)
{
    if (this->weekBgColor != weekBgColor) {
        this->weekBgColor = weekBgColor;
        this->initStyle();
    }
}

bool LunarCalendarWidget::getShowLunar() const
{
    return this->showLunar;
}

void LunarCalendarWidget::setShowLunar(bool showLunar)
{
    if (this->showLunar != showLunar) {
        this->showLunar = showLunar;
        this->initStyle();
    }
}

QString LunarCalendarWidget::getBgImage() const
{
    return this->bgImage;
}

void LunarCalendarWidget::setBgImage(const QString &bgImage)
{
    if (this->bgImage != bgImage) {
        this->bgImage = bgImage;
        this->initStyle();
    }
}

LunarCalendarWidget::SelectType LunarCalendarWidget::getSelectType() const
{
    return this->selectType;
}

void LunarCalendarWidget::setSelectType(const LunarCalendarWidget::SelectType &selectType)
{
    if (this->selectType != selectType) {
        this->selectType = selectType;
        this->initStyle();
    }
}


QColor LunarCalendarWidget::getBorderColor() const
{
    return this->borderColor;
}

void LunarCalendarWidget::setBorderColor(const QColor &borderColor)
{
    if (this->borderColor != borderColor) {
        this->borderColor = borderColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getWeekColor() const
{
    return this->weekColor;
}

void LunarCalendarWidget::setWeekColor(const QColor &weekColor)
{
    if (this->weekColor != weekColor) {
        this->weekColor = weekColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getSuperColor() const
{
    return this->superColor;
}

void LunarCalendarWidget::setSuperColor(const QColor &superColor)
{
    if (this->superColor != superColor) {
        this->superColor = superColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getLunarColor() const
{
    return this->lunarColor;
}

void LunarCalendarWidget::setLunarColor(const QColor &lunarColor)
{
    if (this->lunarColor != lunarColor) {
        this->lunarColor = lunarColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getCurrentTextColor() const
{
    return this->currentTextColor;
}

void LunarCalendarWidget::setCurrentTextColor(const QColor &currentTextColor)
{
    if (this->currentTextColor != currentTextColor) {
        this->currentTextColor = currentTextColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getOtherTextColor() const
{
    return this->otherTextColor;
}

void LunarCalendarWidget::setOtherTextColor(const QColor &otherTextColor)
{
    if (this->otherTextColor != otherTextColor) {
        this->otherTextColor = otherTextColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getSelectTextColor() const
{
    return this->selectTextColor;
}

void LunarCalendarWidget::setSelectTextColor(const QColor &selectTextColor)
{
    if (this->selectTextColor != selectTextColor) {
        this->selectTextColor = selectTextColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getHoverTextColor() const
{
    return this->hoverTextColor;
}

void LunarCalendarWidget::setHoverTextColor(const QColor &hoverTextColor)
{
    if (this->hoverTextColor != hoverTextColor) {
        this->hoverTextColor = hoverTextColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getCurrentLunarColor() const
{
    return this->currentLunarColor;
}

void LunarCalendarWidget::setCurrentLunarColor(const QColor &currentLunarColor)
{
    if (this->currentLunarColor != currentLunarColor) {
        this->currentLunarColor = currentLunarColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getOtherLunarColor() const
{
    return this->otherLunarColor;
}

void LunarCalendarWidget::setOtherLunarColor(const QColor &otherLunarColor)
{
    if (this->otherLunarColor != otherLunarColor) {
        this->otherLunarColor = otherLunarColor;
        this->initStyle();
    }
}


QColor LunarCalendarWidget::getSelectLunarColor() const
{
    return this->selectLunarColor;
}

void LunarCalendarWidget::setSelectLunarColor(const QColor &selectLunarColor)
{
    if (this->selectLunarColor != selectLunarColor) {
        this->selectLunarColor = selectLunarColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getHoverLunarColor() const
{
    return this->hoverLunarColor;
}

void LunarCalendarWidget::setHoverLunarColor(const QColor &hoverLunarColor)
{
    if (this->hoverLunarColor != hoverLunarColor) {
        this->hoverLunarColor = hoverLunarColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getCurrentBgColor() const
{
    return this->currentBgColor;
}

void LunarCalendarWidget::setCurrentBgColor(const QColor &currentBgColor)
{
    if (this->currentBgColor != currentBgColor) {
        this->currentBgColor = currentBgColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getOtherBgColor() const
{
    return this->otherBgColor;
}

void LunarCalendarWidget::setOtherBgColor(const QColor &otherBgColor)
{
    if (this->otherBgColor != otherBgColor) {
        this->otherBgColor = otherBgColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getSelectBgColor() const
{
    return this->selectBgColor;
}

void LunarCalendarWidget::setSelectBgColor(const QColor &selectBgColor)
{
    if (this->selectBgColor != selectBgColor) {
        this->selectBgColor = selectBgColor;
        this->initStyle();
    }
}

QColor LunarCalendarWidget::getHoverBgColor() const
{
    return this->hoverBgColor;
}

void LunarCalendarWidget::setHoverBgColor(const QColor &hoverBgColor)
{
    if (this->hoverBgColor != hoverBgColor) {
        this->hoverBgColor = hoverBgColor;
        this->initStyle();
    }
}

//显示上一年
void LunarCalendarWidget::showPreviousYear()
{
    int year = date.year();
    int month = date.month();
    int day = date.day();
    if (year <= 1901) {
        return;
    }

    year--;
    dateChanged(year, month, day);
}

//显示下一年
void LunarCalendarWidget::showNextYear()
{
    int year = date.year();
    int month = date.month();
    int day = date.day();
    if (year >= 2099) {
        return;
    }

    year++;
    dateChanged(year, month, day);
}

//显示上月日期
void LunarCalendarWidget::showPreviousMonth()
{
    int year = date.year();
    int month = date.month();
    int day = date.day();
    if (year <= 1901 && month == 1) {
        return;
    }

    month--;
    if (month < 1) {
        month = 12;
        year--;
    }

    dateChanged(year, month, day);
}

//显示下月日期
void LunarCalendarWidget::showNextMonth()
{
    int year = date.year();
    int month = date.month();
    int day = date.day();
    if (year >= 2099 && month == 12) {
        return;
    }

    month++;
    if (month > 12) {
        month = 1;
        year++;
    }

    dateChanged(year, month, day);
}

//转到今天
void LunarCalendarWidget::showToday()
{
    date = QDate::currentDate();
    initDate();
    dayChanged(date);
}
```

### `./lunarcalendarwidget.h`

```cpp
﻿#ifndef LUNARCALENDARWIDGET_H
#define LUNARCALENDARWIDGET_H

/**
 * 自定义农历控件 作者:倪大侠 整理:feiyangqingyun(QQ:517216493) 2017-11-17
 * 1. 可设置边框颜色、周末颜色、角标颜色、农历节日颜色。
 * 2. 可设置当前月文字颜色、其他月文字颜色、选中日期文字颜色、悬停日期文字颜色。
 * 3. 可设置当前月农历文字颜色、其他月农历文字颜色、选中日期农历文字颜色、悬停日期农历文字颜色。
 * 4. 可设置当前月背景颜色、其他月背景颜色、选中日期背景颜色、悬停日期背景颜色。
 * 5. 可设置三种选中背景模式，矩形背景、圆形背景、图片背景。
 * 6. 可直接切换到上一年、下一年、上一月、下一月、转到今天。
 * 7. 可设置是否显示农历信息，不显示则当做正常的日历使用。
 * 8. 支持1901年-2099年范围。
 * 9. 很方便改成多选日期。
 */

#include <QWidget>
#include <QDate>

#include "lunarcalendarinfo.h"
#include "lunarcalendaritem.h"

class QLabel;
class QComboBox;
class LunarCalendarItem;

#ifdef quc
class Q_DECL_EXPORT LunarCalendarWidget : public QWidget
#else
class LunarCalendarWidget : public QWidget
#endif

{
    Q_OBJECT
    Q_ENUMS(CalendarStyle)
    Q_ENUMS(WeekNameFormat)
    Q_ENUMS(SelectType)

    Q_PROPERTY(CalendarStyle calendarStyle READ getCalendarStyle WRITE setCalendarStyle)
    Q_PROPERTY(WeekNameFormat weekNameFormat READ getWeekNameFormat WRITE setWeekNameFormat)
    Q_PROPERTY(QDate date READ getDate WRITE setDate)

    Q_PROPERTY(QColor weekTextColor READ getWeekTextColor WRITE setWeekTextColor)
    Q_PROPERTY(QColor weekBgColor READ getWeekBgColor WRITE setWeekBgColor)

    Q_PROPERTY(bool showLunar READ getShowLunar WRITE setShowLunar)
    Q_PROPERTY(QString bgImage READ getBgImage WRITE setBgImage)
    Q_PROPERTY(SelectType selectType READ getSelectType WRITE setSelectType)

    Q_PROPERTY(QColor borderColor READ getBorderColor WRITE setBorderColor)
    Q_PROPERTY(QColor weekColor READ getWeekColor WRITE setWeekColor)
    Q_PROPERTY(QColor superColor READ getSuperColor WRITE setSuperColor)
    Q_PROPERTY(QColor lunarColor READ getLunarColor WRITE setLunarColor)

    Q_PROPERTY(QColor currentTextColor READ getCurrentTextColor WRITE setCurrentTextColor)
    Q_PROPERTY(QColor otherTextColor READ getOtherTextColor WRITE setOtherTextColor)
    Q_PROPERTY(QColor selectTextColor READ getSelectTextColor WRITE setSelectTextColor)
    Q_PROPERTY(QColor hoverTextColor READ getHoverTextColor WRITE setHoverTextColor)

    Q_PROPERTY(QColor currentLunarColor READ getCurrentLunarColor WRITE setCurrentLunarColor)
    Q_PROPERTY(QColor otherLunarColor READ getOtherLunarColor WRITE setOtherLunarColor)
    Q_PROPERTY(QColor selectLunarColor READ getSelectLunarColor WRITE setSelectLunarColor)
    Q_PROPERTY(QColor hoverLunarColor READ getHoverLunarColor WRITE setHoverLunarColor)

    Q_PROPERTY(QColor currentBgColor READ getCurrentBgColor WRITE setCurrentBgColor)
    Q_PROPERTY(QColor otherBgColor READ getOtherBgColor WRITE setOtherBgColor)
    Q_PROPERTY(QColor selectBgColor READ getSelectBgColor WRITE setSelectBgColor)
    Q_PROPERTY(QColor hoverBgColor READ getHoverBgColor WRITE setHoverBgColor)

public:
    enum CalendarStyle {
        CalendarStyle_Red = 0
    };

    enum WeekNameFormat {
        WeekNameFormat_Short = 0,   //短名称
        WeekNameFormat_Normal = 1,  //普通名称
        WeekNameFormat_Long = 2,    //长名称
        WeekNameFormat_En = 3       //英文名称
    };

    enum SelectType {
        SelectType_Rect = 0,        //矩形背景
        SelectType_Circle = 1,      //圆形背景
        SelectType_Triangle = 2,    //带三角标
        SelectType_Image = 3        //图片背景
    };

    explicit LunarCalendarWidget(QWidget *parent = 0);
    ~LunarCalendarWidget();

private:
    QFont iconFont;                 //图形字体
    bool btnClick;                  //按钮单击,避开下拉选择重复触发
    QComboBox *cboxYear;            //年份下拉框
    QComboBox *cboxMonth;           //月份下拉框
    QList<QLabel *> labWeeks;       //顶部星期名称
    QList<LunarCalendarItem *> items;//日期元素

    CalendarStyle calendarStyle;    //整体样式
    WeekNameFormat weekNameFormat;  //星期名称格式
    QDate date;                     //当前日期

    QColor weekTextColor;           //星期名称文字颜色
    QColor weekBgColor;             //星期名称背景色

    bool showLunar;                 //显示农历
    QString bgImage;                //背景图片
    SelectType selectType;          //选中模式

    QColor borderColor;             //边框颜色
    QColor weekColor;               //周末颜色
    QColor superColor;              //角标颜色
    QColor lunarColor;              //农历节日颜色

    QColor currentTextColor;        //当前月文字颜色
    QColor otherTextColor;          //其他月文字颜色
    QColor selectTextColor;         //选中日期文字颜色
    QColor hoverTextColor;          //悬停日期文字颜色

    QColor currentLunarColor;       //当前月农历文字颜色
    QColor otherLunarColor;         //其他月农历文字颜色
    QColor selectLunarColor;        //选中日期农历文字颜色
    QColor hoverLunarColor;         //悬停日期农历文字颜色

    QColor currentBgColor;          //当前月背景颜色
    QColor otherBgColor;            //其他月背景颜色
    QColor selectBgColor;           //选中日期背景颜色
    QColor hoverBgColor;            //悬停日期背景颜色

private slots:
    void initWidget();
    void initStyle();
    void initDate();
    void yearChanged(int);
    void monthChanged(int);
    void clicked(const QDate &date, const LunarCalendarItem::DayType &dayType);
    void dayChanged(const QDate &date);
    void dateChanged(int year, int month, int day);

public:
    //默认尺寸和最小尺寸
    QSize sizeHint() const;
    QSize minimumSizeHint() const;

    //获取和设置整体样式
    CalendarStyle getCalendarStyle() const;
    void setCalendarStyle(const CalendarStyle &calendarStyle);

    //获取和设置星期名称格式
    WeekNameFormat getWeekNameFormat() const;
    void setWeekNameFormat(const WeekNameFormat &weekNameFormat);

    //获取和设置日期
    QDate getDate() const;
    void setDate(const QDate &date);

    //获取和设置顶部星期名称文字颜色
    QColor getWeekTextColor() const;
    void setWeekTextColor(const QColor &weekTextColor);

    //获取和设置顶部星期名称文字背景色
    QColor getWeekBgColor() const;
    void setWeekBgColor(const QColor &weekBgColor);

    //获取和设置是否显示农历信息
    bool getShowLunar() const;
    void setShowLunar(bool showLunar);

    //获取和设置背景图片
    QString getBgImage() const;
    void setBgImage(const QString &bgImage);

    //获取和设置选中背景样式
    SelectType getSelectType() const;
    void setSelectType(const SelectType &selectType);

    //获取和设置边框颜色
    QColor getBorderColor() const;
    void setBorderColor(const QColor &borderColor);

    //获取和设置周末颜色
    QColor getWeekColor() const;
    void setWeekColor(const QColor &weekColor);

    //获取和设置角标颜色
    QColor getSuperColor() const;
    void setSuperColor(const QColor &superColor);

    //获取和设置农历节日颜色
    QColor getLunarColor() const;
    void setLunarColor(const QColor &lunarColor);

    //获取和设置当前月文字颜色
    QColor getCurrentTextColor() const;
    void setCurrentTextColor(const QColor &currentTextColor);

    //获取和设置其他月文字颜色
    QColor getOtherTextColor() const;
    void setOtherTextColor(const QColor &otherTextColor);

    //获取和设置选中日期文字颜色
    QColor getSelectTextColor() const;
    void setSelectTextColor(const QColor &selectTextColor);

    //获取和设置悬停日期文字颜色
    QColor getHoverTextColor() const;
    void setHoverTextColor(const QColor &hoverTextColor);

    //获取和设置当前月农历文字颜色
    QColor getCurrentLunarColor() const;
    void setCurrentLunarColor(const QColor &currentLunarColor);

    //获取和设置其他月农历文字颜色
    QColor getOtherLunarColor() const;
    void setOtherLunarColor(const QColor &otherLunarColor);

    //获取和设置选中日期农历文字颜色
    QColor getSelectLunarColor() const;
    void setSelectLunarColor(const QColor &selectLunarColor);

    //获取和设置悬停日期农历文字颜色
    QColor getHoverLunarColor() const;
    void setHoverLunarColor(const QColor &hoverLunarColor);

    //获取和设置当前月背景颜色
    QColor getCurrentBgColor() const;
    void setCurrentBgColor(const QColor &currentBgColor);

    //获取和设置其他月背景颜色
    QColor getOtherBgColor() const;
    void setOtherBgColor(const QColor &otherBgColor);

    //获取和设置选中日期背景颜色
    QColor getSelectBgColor() const;
    void setSelectBgColor(const QColor &selectBgColor);

    //获取和设置悬停日期背景颜色
    QColor getHoverBgColor() const;
    void setHoverBgColor(const QColor &hoverBgColor);

public Q_SLOTS:
    //转到上一年
    void showPreviousYear();
    //转到下一年
    void showNextYear();

    //转到上一月
    void showPreviousMonth();
    //转到下一月
    void showNextMonth();

    //转到今天
    void showToday();

Q_SIGNALS:
    void clicked(const QDate &date);
    void selectionChanged();
};

#endif // LUNARCALENDARWIDGET_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmlunarcalendarwidget.h"
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

    frmLunarCalendarWidget w;
    w.setWindowTitle("农历控件 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./frmlunarcalendarwidget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmLunarCalendarWidget</class>
 <widget class="QWidget" name="frmLunarCalendarWidget">
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
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="LunarCalendarWidget" name="lunarCalendarWidget" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Preferred" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
    </widget>
   </item>
   <item>
    <widget class="QWidget" name="widgetBottom" native="true">
     <layout class="QHBoxLayout" name="horizontalLayout">
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
       <widget class="QLabel" name="labCalendarStyle">
        <property name="text">
         <string>整体样式</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QComboBox" name="cboxCalendarStyle">
        <property name="minimumSize">
         <size>
          <width>90</width>
          <height>0</height>
         </size>
        </property>
        <item>
         <property name="text">
          <string>红色风格</string>
         </property>
        </item>
       </widget>
      </item>
      <item>
       <widget class="QLabel" name="labSelectType">
        <property name="text">
         <string>选中样式</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QComboBox" name="cboxSelectType">
        <property name="minimumSize">
         <size>
          <width>90</width>
          <height>0</height>
         </size>
        </property>
        <item>
         <property name="text">
          <string>矩形背景</string>
         </property>
        </item>
        <item>
         <property name="text">
          <string>圆形背景</string>
         </property>
        </item>
        <item>
         <property name="text">
          <string>角标背景</string>
         </property>
        </item>
        <item>
         <property name="text">
          <string>图片背景</string>
         </property>
        </item>
       </widget>
      </item>
      <item>
       <widget class="QLabel" name="labWeekNameFormat">
        <property name="text">
         <string>星期格式</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QComboBox" name="cboxWeekNameFormat">
        <property name="minimumSize">
         <size>
          <width>90</width>
          <height>0</height>
         </size>
        </property>
        <item>
         <property name="text">
          <string>短名称</string>
         </property>
        </item>
        <item>
         <property name="text">
          <string>普通名称</string>
         </property>
        </item>
        <item>
         <property name="text">
          <string>长名称</string>
         </property>
        </item>
        <item>
         <property name="text">
          <string>英文名称</string>
         </property>
        </item>
       </widget>
      </item>
      <item>
       <widget class="QCheckBox" name="ckShowLunar">
        <property name="text">
         <string>显示农历</string>
        </property>
        <property name="checked">
         <bool>true</bool>
        </property>
       </widget>
      </item>
      <item>
       <spacer name="horizontalSpacer">
        <property name="orientation">
         <enum>Qt::Horizontal</enum>
        </property>
        <property name="sizeHint" stdset="0">
         <size>
          <width>40</width>
          <height>20</height>
         </size>
        </property>
       </spacer>
      </item>
     </layout>
    </widget>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>LunarCalendarWidget</class>
   <extends>QWidget</extends>
   <header>lunarcalendarwidget.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
