---
tags:
  - Qt
  - 网友提供
---
# 动态StackWidget (`astackwidget`)

> 动态StackWidget

## 效果图

![[QWidgetDemo_assert/astackwidget.jpg]]

## 分类

- 类型: 网友提供 `netfriend`
- 源码目录: `netfriend/astackwidget`

## 技术要点

**用到的 Qt 类**: QWidget QTextCodec QPropertyAnimation QPushButton QList QButtonGroup QLabel QString QResizeEvent QVariant QApplication QDebug QAbstractAnimation QVBoxLayout QHBoxLayout QFont QStringLiteral

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./astackwidget.cpp`

```cpp
﻿#include "astackwidget.h"

#include <QDebug>
#include <QPropertyAnimation>

AStackWidget::AStackWidget(QWidget *parent)
	: QWidget(parent)
{
	m_offset = 0;
	m_curIndex = 0;
    m_lastIndex = 0;
	m_duration = 500;
	m_moveAnimation = new QPropertyAnimation(this, "");
	m_moveAnimation->setDuration(m_duration);
	connect(m_moveAnimation, &QPropertyAnimation::valueChanged, this, &AStackWidget::onValueChanged);
	connect(m_moveAnimation, &QPropertyAnimation::finished, this, &AStackWidget::onMoveFinished);
}

AStackWidget::~AStackWidget()
{

}

int AStackWidget::count() const
{
	return m_widgetLst.size();
}

int AStackWidget::currentIndex() const
{
	return m_curIndex;
}

void AStackWidget::setDuration(int duration)
{
	m_duration = duration;
}

int AStackWidget::addWidget(QWidget * widget)
{
	int index = indexOf(widget);
	if (index >= 0){
		return index;
	}
	widget->setParent(this);
	m_widgetLst.append(widget);
	return count() - 1;
}

int AStackWidget::indexOf(QWidget * widget) const
{
	return m_widgetLst.indexOf(widget);
}

int AStackWidget::insertWidget(int index, QWidget * widget)
{
	int curindex = indexOf(widget);
	if (curindex >= 0) {
		return curindex;
	}
	widget->setParent(this);
	m_widgetLst.insert(index, widget);
	return index;
}

QWidget * AStackWidget::currentWidget() const
{
	if (m_curIndex >= 0 && m_curIndex < count()){
		return m_widgetLst.at(m_curIndex);
	}
    return 0;
}

QWidget * AStackWidget::widget(int index) const
{
	if (index >= 0 && index < count()) {
		return m_widgetLst.at(index);
	}
    return 0;
}

void AStackWidget::removeWidget(QWidget * widget)
{
	int index = indexOf(widget);
	if (index >= 0) {
		m_widgetLst.removeAll(widget);
		emit widgetRemoved(index);
	}
}

void AStackWidget::setCurrentWidget(QWidget * widget)
{
	int index = indexOf(widget);
	if (index >= 0 && m_curIndex != index) {
		setCurrentIndex(index);
	}
}

void AStackWidget::setCurrentIndex(int index)
{
	if (index >= 0 && m_curIndex != index) {
		m_lastIndex = m_curIndex;
		m_curIndex = index;	
		moveAnimationStart();
		emit currentChanged(index);
	}
}

void AStackWidget::resizeEvent(QResizeEvent *event)
{
	QWidget::resizeEvent(event);

	int size = count();
	for (int i = 0; i < size; i++) {
		m_widgetLst.at(i)->resize(this->width(), this->height());
	}

	if (m_moveAnimation->state() == QAbstractAnimation::Running) {
		moveAnimationStart();
	}
	else {
		setWidgetsVisible();
        onValueChanged(0);
	}
}

void AStackWidget::onValueChanged(const QVariant &value)
{
	m_offset = value.toInt();
	m_widgetLst.at(m_curIndex)->move(m_offset, 0);
	if (m_curIndex > m_lastIndex) {
		m_widgetLst.at(m_lastIndex)->move(m_offset - this->width(), 0);
    } else if (m_curIndex < m_lastIndex){
		m_widgetLst.at(m_lastIndex)->move(this->width() + m_offset, 0);
	}
}

void AStackWidget::moveAnimationStart()
{
	m_moveAnimation->stop();
	setWidgetsVisible();
	int startOffset = m_offset;
	if (m_curIndex > m_lastIndex) {
		if (startOffset == 0) startOffset = this->width();
		else startOffset = this->width() - qAbs(startOffset);
	}
	else {
		if (startOffset == 0) startOffset = -this->width();
		else startOffset = qAbs(startOffset) - this->width();
	}
	m_moveAnimation->setDuration(qAbs(startOffset) * m_duration / this->width());
	m_moveAnimation->setStartValue(startOffset);
	m_moveAnimation->setEndValue(0);
	m_moveAnimation->start();
}

void AStackWidget::setWidgetsVisible()
{
	int size = count();
	for (int i = 0; i < size; i++) {
		if (m_lastIndex == i || m_curIndex == i)
			m_widgetLst.at(i)->setVisible(true);
		else {
			m_widgetLst.at(i)->setVisible(false);
		}
	}
}

void AStackWidget::onMoveFinished()
{

}
```

### `./astackwidget.h`

```cpp
﻿#include <QWidget>

class QPropertyAnimation;
class AStackWidget : public QWidget
{
	Q_OBJECT

public:
	AStackWidget(QWidget *parent);
	~AStackWidget();

signals:
	void currentChanged(int index);
	void widgetRemoved(int index);

public:
	int count() const;
	int currentIndex() const;
	int addWidget(QWidget *widget);
	int indexOf(QWidget *widget) const;
	int insertWidget(int index, QWidget *widget);
	
	QWidget *currentWidget() const;	
	QWidget *widget(int index) const;

	void removeWidget(QWidget *widget);
	void setDuration(int duration);

public slots:
	void setCurrentIndex(int index);
	void setCurrentWidget(QWidget *widget);

private slots:
	void onValueChanged(const QVariant &);
	void onMoveFinished();

private:
	void moveAnimationStart();
	void setWidgetsVisible();

protected:
	void resizeEvent(QResizeEvent *event);

private:
	int m_offset;
	int m_curIndex;
	int m_lastIndex;

	int m_duration;
	QPropertyAnimation *m_moveAnimation;
	QList<QWidget *> m_widgetLst;
};
```

### `./frmastackwidget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "FrmAStackWidget.h"

#include <QButtonGroup>
#include <QLabel>

FrmAStackWidget::FrmAStackWidget(QWidget *parent)
    : QWidget(parent)
{
    ui.setupUi(this);

    QList<QString> colorlst;
    colorlst << "#1abc9c";
    colorlst << "#2ecc71";
    colorlst << "#3498db";
    colorlst << "#9b59b6";
    colorlst << "#e74c3c";

    QList<QPushButton *> btnlst;
    btnlst << ui.pushButton_1;
    btnlst << ui.pushButton_2;
    btnlst << ui.pushButton_3;
    btnlst << ui.pushButton_4;
    btnlst << ui.pushButton_5;

    QButtonGroup *btnGroup = new QButtonGroup(this);
#if (QT_VERSION >= QT_VERSION_CHECK(6,0,0))
    connect(btnGroup, SIGNAL(idClicked(int)), ui.aStackwidget, SLOT(setCurrentIndex(int)));
#else
    connect(btnGroup, SIGNAL(buttonClicked(int)), ui.aStackwidget, SLOT(setCurrentIndex(int)));
#endif

    for (int i = 0; i < 5; i++) {
        QLabel *label = new QLabel(ui.aStackwidget);
        label->setStyleSheet(QString("background-color:%1;color:#ffffff;").arg(colorlst.at(i)));
        label->setText(QString::number(i + 1));
        label->setAlignment(Qt::AlignCenter);
        int index = ui.aStackwidget->addWidget(label);
        btnGroup->addButton(btnlst.at(i), index);
    }
}
```

### `./frmastackwidget.h`

```cpp
﻿#include <QWidget>
#include "ui_FrmAStackWidget.h"

class FrmAStackWidget : public QWidget
{
    Q_OBJECT

public:
    FrmAStackWidget(QWidget *parent = Q_NULLPTR);

private:
	Ui::FrmAStackWidgetClass ui;
};
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")
#include "frmastackwidget.h"

#include <QApplication>
#include <QTextCodec>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    a.setFont(QFont("Microsoft Yahei", 9));

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

    FrmAStackWidget w;
	w.setWindowTitle(QStringLiteral("动态StackWidget"));
    w.show();
    return a.exec();
}
```

### `./frmastackwidget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>FrmAStackWidgetClass</class>
 <widget class="QWidget" name="FrmAStackWidgetClass">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>600</width>
    <height>400</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>FrmAStackWidget</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <property name="spacing">
    <number>12</number>
   </property>
   <property name="leftMargin">
    <number>12</number>
   </property>
   <property name="topMargin">
    <number>12</number>
   </property>
   <property name="rightMargin">
    <number>12</number>
   </property>
   <property name="bottomMargin">
    <number>12</number>
   </property>
   <item>
    <layout class="QHBoxLayout" name="horizontalLayout">
     <item>
      <widget class="QPushButton" name="pushButton_1">
       <property name="text">
        <string>PushButton1</string>
       </property>
      </widget>
     </item>
     <item>
      <widget class="QPushButton" name="pushButton_2">
       <property name="text">
        <string>PushButton2</string>
       </property>
      </widget>
     </item>
     <item>
      <widget class="QPushButton" name="pushButton_3">
       <property name="text">
        <string>PushButton3</string>
       </property>
      </widget>
     </item>
     <item>
      <widget class="QPushButton" name="pushButton_4">
       <property name="text">
        <string>PushButton4</string>
       </property>
      </widget>
     </item>
     <item>
      <widget class="QPushButton" name="pushButton_5">
       <property name="text">
        <string>PushButton5</string>
       </property>
      </widget>
     </item>
    </layout>
   </item>
   <item>
    <widget class="AStackWidget" name="aStackwidget" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Expanding" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <layoutdefault spacing="6" margin="11"/>
 <customwidgets>
  <customwidget>
   <class>AStackWidget</class>
   <extends>QWidget</extends>
   <header>astackwidget.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
