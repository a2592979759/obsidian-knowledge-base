---
tags:
  - Qt
  - 网友提供
---
# 滑块图片验证码 (`slidepuzzlewidget`)

> 滑块图片验证码

## 效果图

![[QWidgetDemo_assert/sliderpuzzlewidget.jpg]]

## 分类

- 类型: 网友提供 `netfriend`
- 源码目录: `netfriend/slidepuzzlewidget`

## 技术要点

**用到的 Qt 类**: QWidget QTextCodec QString QPainter QTimer QPainterPath QColor QTime QPixmap QSlider QVBoxLayout QApplication QPoint QPaintEvent QPen QMessageBox QFont QRect QSize

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmslidepuzzlewidget.cpp`

```cpp
﻿#include "frmslidepuzzlewidget.h"

FrmSlidePuzzleWidget::FrmSlidePuzzleWidget(QWidget *parent)
	: QWidget(parent)
{
	ui.setupUi(this);
	this->initForm();
}

FrmSlidePuzzleWidget::~FrmSlidePuzzleWidget()
{

}

void FrmSlidePuzzleWidget::initForm()
{

}
```

### `./frmslidepuzzlewidget.h`

```cpp
﻿#ifndef FRMSLIDEPUZZLEWIDGET_H
#define FRMSLIDEPUZZLEWIDGET_H

#include <QtWidgets/QWidget>
#include "ui_frmslidepuzzlewidget.h"

class FrmSlidePuzzleWidget : public QWidget
{
	Q_OBJECT

public:
	FrmSlidePuzzleWidget(QWidget *parent = 0);
	~FrmSlidePuzzleWidget();

private:
	void initForm();

private:
	Ui::FrmSlidePuzzleWidgetClass ui;
};

#endif // FRMSLIDEPUZZLEWIDGET_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmslidepuzzlewidget.h"
#include <QApplication>
#include <QTextCodec>

int main(int argc, char *argv[])
{
	QApplication a(argc, argv);
	a.setFont(QFont("Microsoft Yahei", 10));

#if (QT_VERSION <= QT_VERSION_CHECK(5,0,0))
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

	FrmSlidePuzzleWidget w;
	w.setWindowTitle("滑块图片验证码");
	w.show();
	return a.exec();
}
```

### `./puzzlewidget.cpp`

```cpp
﻿#include "puzzlewidget.h"

#include <QPainter>
#include <QPainterPath>
#include <QTime>
#include <QTimer>

const int squarewidth = 46;
const int squareradius = 20;
PuzzleWidget::PuzzleWidget(QWidget *parent)
	: QWidget(parent)
{
	m_value = 0;
	m_offsetPoint = QPoint(0, 0);
    srand(QTime(0, 0, 0).secsTo(QTime::currentTime()));
}

PuzzleWidget::~PuzzleWidget()
{

}

void PuzzleWidget::setPixmap(const QString& pixmap)
{
	m_pixmap = pixmap;
	QTimer::singleShot(10, this, SLOT(onUpdatePixmap()));
}

void PuzzleWidget::onUpdatePixmap()
{
    m_offsetPoint.rx() = qBound(0, rand() % this->width() + squarewidth + squareradius, this->width() - squarewidth - squareradius);
    m_offsetPoint.ry() = qBound(0, rand() % this->height() + squarewidth + squareradius, this->height() - squarewidth - squareradius);
	update();
}

void PuzzleWidget::setValue(int value)
{
	m_value = qBound(0, value, this->width() - squarewidth - squareradius + m_offsetPoint.x());
	update();
}

void PuzzleWidget::paintEvent(QPaintEvent *event)
{
	QPainter painter(this);
	painter.setRenderHints(QPainter::Antialiasing);
	QPainterPath clippath;
	clippath.addRoundedRect(this->rect(), 4, 4);
	painter.setClipPath(clippath);
	const QPixmap& pixmap = QPixmap(m_pixmap).scaled(this->width(), this->height(), Qt::IgnoreAspectRatio, Qt::SmoothTransformation);
	painter.drawPixmap(0, 0, this->width(), this->height(), pixmap);

	QPainterPath cutoutpath;
	cutoutpath.setFillRule(Qt::WindingFill);
	QRect rect(m_offsetPoint, QSize(squarewidth, squarewidth));
	cutoutpath.addRoundedRect(rect, 2, 2);
	cutoutpath.addEllipse(rect.center().x() - squareradius / 2, rect.top() - squareradius + 6, squareradius, squareradius);
	QPainterPath subellipseparh;
	subellipseparh.addEllipse(rect.right() - squareradius + 6, rect.center().y() - squareradius / 2, squareradius, squareradius);
	cutoutpath -= subellipseparh;

	painter.setPen(QPen(QColor(80, 80, 80), 1));
	painter.setBrush(QColor(100, 100, 100, 220));
	painter.drawPath(cutoutpath);

	QPixmap puzzlePixmap(this->size());
	puzzlePixmap.fill(Qt::transparent);
	QPainter puzzlePainter(&puzzlePixmap);
	puzzlePainter.setRenderHints(QPainter::Antialiasing);
	puzzlePainter.setClipPath(cutoutpath);
	puzzlePainter.setPen(QPen(QColor(80, 80, 80), 2));
	puzzlePainter.setBrush(QColor(200, 200, 200, 100));
	puzzlePainter.drawPixmap(0, 0, this->width(), this->height(), pixmap);
	puzzlePainter.drawPath(cutoutpath);

	painter.drawPixmap(-m_offsetPoint.x() + m_value, 0, this->width(), this->height(), puzzlePixmap);
}

bool PuzzleWidget::isOverlap()
{
	return qAbs(-m_offsetPoint.x() + m_value) < 5;
}
```

### `./puzzlewidget.h`

```cpp
﻿#ifndef PUZZLEWIDGET_H
#define PUZZLEWIDGET_H

#include <QWidget>

class PuzzleWidget : public QWidget
{
	Q_OBJECT
	Q_PROPERTY(QString pixmap READ getPixmap WRITE setPixmap)

public:
	PuzzleWidget(QWidget *parent);
	~PuzzleWidget();

public:
	QString getPixmap() const { return m_pixmap; };
	void setPixmap(const QString& pixmap);

	void setValue(int value);
	bool isOverlap();

private slots:
	void onUpdatePixmap();

protected:
	void paintEvent(QPaintEvent *event);

private:
	int m_value;
	QString m_pixmap;
	QPoint m_offsetPoint;
};

#endif // PUZZLEWIDGET_H
```

### `./sliderpuzzlewidget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")
#include "sliderpuzzlewidget.h"

#include <QTimer>
#include <QMessageBox>

SliderPuzzleWidget::SliderPuzzleWidget(QWidget *parent)
	: QWidget(parent)
{
	ui.setupUi(this);
	this->initForm();
}

SliderPuzzleWidget::~SliderPuzzleWidget()
{

}

void SliderPuzzleWidget::initForm()
{
	QTimer::singleShot(10, this, SLOT(onUpdateWidget()));
	connect(ui.horizontalSlider, &QSlider::valueChanged, this, &SliderPuzzleWidget::onSliderValueChanged);
	connect(ui.horizontalSlider, &QSlider::sliderReleased, this, &SliderPuzzleWidget::onSliderReleased);
	ui.puzzlewidget->setPixmap(":/FrmSlidePuzzleWidget/Resources/back1.png");
}

void SliderPuzzleWidget::onUpdateWidget()
{
	ui.horizontalSlider->setRange(0, this->width());
}

void SliderPuzzleWidget::onSliderValueChanged(int value)
{
	ui.puzzlewidget->setValue(value);
}

void SliderPuzzleWidget::onSliderReleased()
{
	QString content = ui.puzzlewidget->isOverlap() ? "验证成功！" : "验证失败！";
	QMessageBox msgBox;
	msgBox.setWindowTitle("滑块图片验证码");
	msgBox.setText(content);
	msgBox.exec();

	static int testIndex = 1;
	testIndex = testIndex + 1 > 4 ? 1 : testIndex + 1;
	ui.horizontalSlider->setValue(0);
	ui.puzzlewidget->setPixmap(QString(":/FrmSlidePuzzleWidget/Resources/back%1.png").arg(testIndex));
}
```

### `./sliderpuzzlewidget.h`

```cpp
﻿#ifndef SLIDERPUZZLEWIDGET_H
#define SLIDERPUZZLEWIDGET_H

#include <QWidget>
#include "ui_sliderpuzzlewidget.h"

class SliderPuzzleWidget : public QWidget
{
	Q_OBJECT

public:
	SliderPuzzleWidget(QWidget *parent = 0);
	~SliderPuzzleWidget();

private:
	void initForm();

private slots:
	void onUpdateWidget();
	void onSliderValueChanged(int value);
	void onSliderReleased();

private:
	Ui::SliderPuzzleWidget ui;
};

#endif // SLIDERPUZZLEWIDGET_H
```

### `./frmslidepuzzlewidget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>FrmSlidePuzzleWidgetClass</class>
 <widget class="QWidget" name="FrmSlidePuzzleWidgetClass">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>600</width>
    <height>400</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>FrmSlidePuzzleWidget</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <property name="spacing">
    <number>20</number>
   </property>
   <property name="leftMargin">
    <number>20</number>
   </property>
   <property name="topMargin">
    <number>20</number>
   </property>
   <property name="rightMargin">
    <number>20</number>
   </property>
   <property name="bottomMargin">
    <number>20</number>
   </property>
   <item>
    <widget class="SliderPuzzleWidget" name="sliderpuzzleWidget" native="true"/>
   </item>
  </layout>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <customwidgets>
  <customwidget>
   <class>SliderPuzzleWidget</class>
   <extends>QWidget</extends>
   <header>sliderpuzzlewidget.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources>
  <include location="frmslidepuzzlewidget.qrc"/>
 </resources>
 <connections/>
</ui>
```

### `./sliderpuzzlewidget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>SliderPuzzleWidget</class>
 <widget class="QWidget" name="SliderPuzzleWidget">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>717</width>
    <height>320</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>SliderPuzzleWidget</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <property name="spacing">
    <number>14</number>
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
    <widget class="PuzzleWidget" name="puzzlewidget" native="true"/>
   </item>
   <item>
    <widget class="QSlider" name="horizontalSlider">
     <property name="orientation">
      <enum>Qt::Horizontal</enum>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <customwidgets>
  <customwidget>
   <class>PuzzleWidget</class>
   <extends>QWidget</extends>
   <header>puzzlewidget.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
