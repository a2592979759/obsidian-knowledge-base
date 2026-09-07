---
tags:
  - Qt
  - 网友提供
---
# 图片3D效果切换 (`imageviewwindow`)

> 图片3D效果切换

## 效果图

![[QWidgetDemo_assert/imageviewwindow.jpg]]

## 分类

- 类型: 网友提供 `netfriend`
- 源码目录: `netfriend/imageviewwindow`

## 技术要点

**用到的 Qt 类**: QString QPropertyAnimation QTextCodec QPointF QMap QPainter QPixmap QSize QWidget QTimer QGraphicsView QGraphicsScene QParallelAnimationGroup QGraphicsObject QGraphicsSceneMouseEvent QCursor QRectF QDebug QStyleOptionGraphicsItem QResizeEvent

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./graphicspixmap.cpp`

```cpp
#include "graphicspixmap.h"

#include <QGraphicsSceneMouseEvent>
#include <QPainter>
#include <QCursor>
#include <QDebug>

GraphicsPixmap::GraphicsPixmap() : QGraphicsObject()
{
	setCacheMode(DeviceCoordinateCache);
}

void GraphicsPixmap::mousePressEvent(QGraphicsSceneMouseEvent *event)
{
	QGraphicsObject::mousePressEvent(event);
	if (event->button() == Qt::LeftButton)
	{
		emit clicked();
	}
}

void GraphicsPixmap::setItemOffset(QPointF ponit)
{
	prepareGeometryChange();
	offset = ponit;
	update();
}

QPointF GraphicsPixmap::itemoffset()
{
	return offset;
}

void GraphicsPixmap::setPixmap(const QPixmap& pixmap)
{
	pixSize = pixmap.size();
	pix = pixmap;
}

void GraphicsPixmap::setPixmapSize(QSize size)
{
	pixSize = size;
}

QSize GraphicsPixmap::pixsize()
{
	return pixSize;
}

QRectF GraphicsPixmap::boundingRect() const
{
	return QRectF(offset, pix.size() / pix.devicePixelRatio());
}

void GraphicsPixmap::paint(QPainter *painter, const QStyleOptionGraphicsItem *, QWidget *)
{
	painter->setRenderHint(QPainter::SmoothPixmapTransform, true);
	painter->drawPixmap(offset, pix.scaled(pixSize, Qt::KeepAspectRatio, Qt::SmoothTransformation));
}
```

### `./graphicspixmap.h`

```cpp
#ifndef GRAPHICSPIXMAP_H
#define GRAPHICSPIXMAP_H

#include <QObject>
#include <QGraphicsObject> 
#include <QPixmap>

class GraphicsPixmap : public QGraphicsObject
{
	Q_OBJECT
	Q_PROPERTY(QPointF itemoffset READ itemoffset WRITE setItemOffset)
	Q_PROPERTY(QSize itemsize READ pixsize WRITE setPixmapSize)

public:
	GraphicsPixmap();

public:
	QRectF boundingRect() const Q_DECL_OVERRIDE;
	void setItemOffset(QPointF ponit);
	QPointF itemoffset();
	QSize pixsize();
	void setPixmap(const QPixmap& pixmap);
	void setPixmapSize(QSize size);

signals:
	void clicked();

private:
	void mousePressEvent(QGraphicsSceneMouseEvent *event) Q_DECL_OVERRIDE;
	void paint(QPainter *, const QStyleOptionGraphicsItem *, QWidget *) Q_DECL_OVERRIDE;

private:
	QPixmap pix;
	QPointF offset;
	QSize pixSize;
};

#endif // GRAPHICSPIXMAP_H
```

### `./graphicsview.cpp`

```cpp
#include "graphicsview.h"

GraphicsView::GraphicsView(QGraphicsScene *scene)
	: QGraphicsView(scene)
{

}

GraphicsView::~GraphicsView()
{

}

void GraphicsView::resizeEvent(QResizeEvent *event)
{
	QGraphicsView::resizeEvent(event);
	fitInView(sceneRect(), Qt::KeepAspectRatio);
}
```

### `./graphicsview.h`

```cpp
#ifndef GRAPHICSVIEW_H
#define GRAPHICSVIEW_H

#include <QGraphicsView>

class GraphicsView : public QGraphicsView
{
	Q_OBJECT

public:
	GraphicsView(QGraphicsScene *scene);
	~GraphicsView();

protected:
	void resizeEvent(QResizeEvent *event) Q_DECL_OVERRIDE;	
};

#endif // GRAPHICSVIEW_H
```

### `./imageviewwindow.cpp`

```cpp
﻿#include "imageviewwindow.h"
#include "graphicsview.h"
#include "graphicspixmap.h"

#include <QParallelAnimationGroup>
#include <QPropertyAnimation>
#include <QDebug>
#include <QTimer>

const int image_conunt = 5;
const int image_yoffset = 40;
const int image_xoffset = 60;
ImageViewWindow::ImageViewWindow(QWidget *parent)
	: QWidget(parent)
	, m_isStart(false)
{
	ui.setupUi(this);
	initControl();
}

ImageViewWindow::~ImageViewWindow()
{

}

void ImageViewWindow::initControl()
{
	//场景
	m_scene = new QGraphicsScene(QRect(0, 0, 876, 368), this);	
	//图片信息
	m_imgMapInfolst << QMap<QString, QString>{
		{ "zIndex"  , "1"   }, 
		{ "width"   , "120" }, 
		{ "height"  , "150" }, 
		{ "top"     , "71"  }, 
		{ "left"    , "134" }, 
		{ "opacity" , "0.6" }
	};
	m_imgMapInfolst << QMap<QString, QString>{
		{ "zIndex", "2"   },
		{ "width", "130" },
		{ "height", "170" },
		{ "top", "61" },
		{ "left", "0" },
		{ "opacity", "0.7" }
	};
	m_imgMapInfolst << QMap<QString, QString>{
		{ "zIndex", "3"   },
		{ "width", "170" },
		{ "height", "218" },
		{ "top", "37" },
		{ "left", "110" },
		{ "opacity", "0.8" }
	};
	m_imgMapInfolst << QMap<QString, QString>{
		{ "zIndex", "4"   },
		{ "width", "224" },
		{ "height", "288" },
		{ "top", "0" },
		{ "left", "262" },
		{ "opacity", "1" }
	};
	m_imgMapInfolst << QMap<QString, QString>{
		{ "zIndex", "3"   },
		{ "width", "170" },
		{ "height", "218" },
		{ "top", "37" },
		{ "left", "468" },
		{ "opacity", "0.8" }
	};
	m_imgMapInfolst << QMap<QString, QString>{
		{ "zIndex", "2"   },
		{ "width", "130" },
		{ "height", "170" },
		{ "top", "61" },
		{ "left", "620" },
		{ "opacity", "0.7" }
	};
	m_imgMapInfolst << QMap<QString, QString>{
		{ "zIndex", "1"   },
		{ "width", "120" },
		{ "height", "150" },
		{ "top", "71" },
		{ "left", "496" },
		{ "opacity", "0.6" }
	};

	//场景中添加图片元素
	for (int index = 0; index < m_imgMapInfolst.size(); index++)
	{
		const auto imageInfoMap = m_imgMapInfolst[index];
		const QString&& centerImg = QString(":/ImageViewWindow/Resources/%1.jpg").arg(index + 1);
		const QPixmap&& pixmap = QPixmap(centerImg);
		GraphicsPixmap *item = new GraphicsPixmap();
		item->setPixmap(pixmap);
		item->setPixmapSize(QSize(imageInfoMap["width"].toInt(), imageInfoMap["height"].toInt()));
		item->setItemOffset(QPointF(imageInfoMap["left"].toInt() + image_xoffset, imageInfoMap["top"].toInt() + image_yoffset));
		item->setZValue(imageInfoMap["zIndex"].toInt());
		item->setOpacity(imageInfoMap["opacity"].toFloat());
		m_items << item;
		m_scene->addItem(item);
	}

	//left button
	GraphicsPixmap *leftBtn = new GraphicsPixmap();
	leftBtn->setCursor(QCursor(Qt::PointingHandCursor));
	leftBtn->setPixmap(QPixmap(":/ImageViewWindow/Resources/Wblog_left.png"));
	leftBtn->setItemOffset(QPointF(12, image_yoffset + 124));
	leftBtn->setZValue(5);
	m_scene->addItem(leftBtn);
	connect(leftBtn, SIGNAL(clicked()), this, SLOT(onLeftBtnClicked()));
	//right button
	GraphicsPixmap *rightBtn = new GraphicsPixmap();
	rightBtn->setCursor(QCursor(Qt::PointingHandCursor));
	rightBtn->setPixmap(QPixmap(":/ImageViewWindow/Resources/Wblog_right.png"));
	rightBtn->setItemOffset(QPointF(836, image_yoffset + 124));
	rightBtn->setZValue(5);
	m_scene->addItem(rightBtn);
	connect(rightBtn, SIGNAL(clicked()), this, SLOT(onRightBtnClicked()));

	//视图
	GraphicsView *view = new GraphicsView(m_scene);
	view->setFrameShape(QFrame::NoFrame);
	view->setParent(this);
	view->setViewportUpdateMode(QGraphicsView::BoundingRectViewportUpdate);
	view->setBackgroundBrush(QColor(46, 46, 46));
	view->setCacheMode(QGraphicsView::CacheBackground);
	view->setRenderHints(QPainter::Antialiasing | QPainter::SmoothPixmapTransform);
	ui.viewlayout->addWidget(view);

	//动画: 大小，位置
	m_group = new QParallelAnimationGroup;
	for (int i = 0; i < m_items.count(); ++i) {
		QPropertyAnimation *anim = new QPropertyAnimation(m_items[i], "itemoffset");
		QPropertyAnimation *anims = new QPropertyAnimation(m_items[i], "itemsize");
		m_animationMap.insert(m_items[i], anim);
		m_animationsMap.insert(m_items[i], anims);
		anim->setDuration(1000);
		anims->setDuration(1000);
		anim->setEasingCurve(QEasingCurve::OutQuad);
		anims->setEasingCurve(QEasingCurve::OutQuad);
		m_group->addAnimation(anim);
		m_group->addAnimation(anims);
	}
	//定时切换图片
	m_timer = new QTimer(this);
	m_timer->setInterval(2000);
	connect(m_timer, &QTimer::timeout, [this](){
		nextPlay();
	});
	connect(m_group, &QParallelAnimationGroup::finished, [this](){
		m_isStart = false;
		m_timer->start();
	});
	m_timer->start();
}

void ImageViewWindow::onLeftBtnClicked()
{
	//鼠标点击的时候，先暂停定时器预览
	m_timer->stop();
	//上一张
	lastPlay();
}

void ImageViewWindow::onRightBtnClicked()
{
	//鼠标点击的时候，先暂停定时器预览
	m_timer->stop();
	//下一张
	nextPlay();
}

void ImageViewWindow::play()
{
	for (int index = 0; index < m_imgMapInfolst.size(); index++)
	{
		const auto item = m_items[index];
		QPropertyAnimation *anim = m_animationMap.value(item);
		QPropertyAnimation *anims = m_animationsMap.value(item);
		const auto imageInfoMap = m_imgMapInfolst[index];
		item->setZValue(imageInfoMap["zIndex"].toInt());
		item->setOpacity(imageInfoMap["opacity"].toFloat());
		QPointF pointf(imageInfoMap["left"].toInt() + image_xoffset, imageInfoMap["top"].toInt() + image_yoffset);
        const QString&& centerImg = QString(":/ImageViewWindow/Resources/%1.jpg").arg(index + 1);
		anim->setStartValue(item->itemoffset());
		anims->setStartValue(item->pixsize());
		anim->setEndValue(pointf);
		anims->setEndValue(QSize(imageInfoMap["width"].toInt(), imageInfoMap["height"].toInt()));
	}
	m_isStart = true;
}


void ImageViewWindow::nextPlay()
{
	m_group->stop();
	auto firstItem = m_items.takeAt(0);
	m_items << firstItem;
	play();
	m_group->start();
}

void ImageViewWindow::lastPlay()
{
	m_group->stop();
	auto lastItem = m_items.takeAt(m_items.size() - 1);
	m_items.prepend(lastItem);
	play();
	m_group->start();
}
```

### `./imageviewwindow.h`

```cpp
﻿#ifndef IMAGEVIEWWINDOW_H
#define IMAGEVIEWWINDOW_H

#include <QtWidgets/QWidget>
#include "ui_imageviewwindow.h"

class QTimer;
class QPropertyAnimation;
class GraphicsPixmap;
class QGraphicsScene;
class QParallelAnimationGroup;
class ImageViewWindow : public QWidget
{
	Q_OBJECT

public:
	ImageViewWindow(QWidget *parent = 0);
	~ImageViewWindow();

private:
	void initControl();
	void nextPlay();
	void lastPlay();
	void play();

private slots:
	void onLeftBtnClicked();
	void onRightBtnClicked();

private:
	Ui::ImageViewWindowClass ui;
	QGraphicsScene* m_scene;
    QList<QMap<QString, QString> > m_imgMapInfolst;
	QList<GraphicsPixmap *> m_items;
	QParallelAnimationGroup *m_group;
	QMap<GraphicsPixmap *, QPropertyAnimation *> m_animationMap;
	QMap<GraphicsPixmap *, QPropertyAnimation *> m_animationsMap;
	QTimer* m_timer;
	bool m_isStart;
};

#endif // IMAGEVIEWWINDOW_H
```

### `./main.cpp`

```cpp
#pragma execution_character_set("utf-8")

#include "imageviewwindow.h"
#include <QtWidgets/QApplication>
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

	ImageViewWindow w;
	w.show();
	return a.exec();
}
```

### `./imageviewwindow.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>ImageViewWindowClass</class>
 <widget class="QWidget" name="ImageViewWindowClass">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>1062</width>
    <height>538</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>ImageViewWindow</string>
  </property>
  <layout class="QVBoxLayout" name="viewlayout">
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
 <layoutdefault spacing="6" margin="11"/>
 <resources>
  <include location="imageviewwindow.qrc"/>
 </resources>
 <connections/>
</ui>
```
