---
tags:
  - Qt
  - 网友提供
  - 高质量
---
# 头像设置工具 (`imagecropper`)

> 头像设置工具

## 效果图

![[QWidgetDemo_assert/imagecropper.jpg]]

## 分类

- 类型: 网友提供 `netfriend`
- 源码目录: `netfriend/imagecropper`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QPushButton QLabel QPixmap QHBoxLayout QPoint QLineEdit QString QSize QMessageBox QPainter QCheckBox QWidget QVBoxLayout QFormLayout QDialog QColor QMouseEvent QPainterPath QPen QFileDialog

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `base/imagecropperdialog.h`

```cpp
#ifndef IMAGECROPPER_H
#define IMAGECROPPER_H

#include <QWidget>
#include <QDialog>
#include <QPainter>
#include <QLabel>
#include <QPixmap>
#include <QString>
#include <QMessageBox>
#include <QHBoxLayout>
#include <QVBoxLayout>
#include <QPushButton>

#include "imagecropperlabel.h"

/*******************************************************
 *  Loacl private class, which do image-cropping
 *  Used in class ImageCropper
*******************************************************/
class ImageCropperDialogPrivate : public QDialog
{
    Q_OBJECT
public:
    ImageCropperDialogPrivate(const QPixmap &imageIn, QPixmap &outputImage,
                              int windowWidth, int windowHeight,
                              CropperShape shape, QSize cropperSize = QSize())
        : QDialog(0)
        , outputImage(outputImage)
    {
        this->setAttribute(Qt::WA_DeleteOnClose, true);
        this->setWindowTitle("Image Cropper");
        this->setMouseTracking(true);
        this->setModal(true);

        imageLabel = new ImageCropperLabel(windowWidth, windowHeight, this);
        imageLabel->setCropper(shape, cropperSize);
        imageLabel->setOutputShape(OutputShape::RECT);
        imageLabel->setOriginalImage(imageIn);
        imageLabel->enableOpacity(true);

        QHBoxLayout *btnLayout = new QHBoxLayout();
        btnOk = new QPushButton("OK", this);
        btnCancel = new QPushButton("Cancel", this);
        btnLayout->addStretch();
        btnLayout->addWidget(btnOk);
        btnLayout->addWidget(btnCancel);

        QVBoxLayout *mainLayout = new QVBoxLayout(this);
        mainLayout->addWidget(imageLabel);
        mainLayout->addLayout(btnLayout);

        connect(btnOk, &QPushButton::clicked, this, [this]() {
            this->outputImage = this->imageLabel->getCroppedImage();
            this->close();
        });
        connect(btnCancel, &QPushButton::clicked, this, [this]() {
            this->outputImage = QPixmap();
            this->close();
        });
    }

private:
    ImageCropperLabel *imageLabel;
    QPushButton *btnOk;
    QPushButton *btnCancel;
    QPixmap &outputImage;
};

/*******************************************************************
 *  class ImageCropperDialog
 *      create a instane of class ImageCropperDialogPrivate
 *      and get cropped image from the instance(after closing)
********************************************************************/
class ImageCropperDialog : QObject
{
public:
    static QPixmap getCroppedImage(const QString &filename, int windowWidth, int windowHeight,
                                   CropperShape cropperShape, QSize crooperSize = QSize())
    {
        QPixmap inputImage;
        QPixmap outputImage;

        if (!inputImage.load(filename)) {
            QMessageBox::critical(0, "Error", "Load image failed!", QMessageBox::Ok);
            return outputImage;
        }

        ImageCropperDialogPrivate *imageCropperDo =
            new ImageCropperDialogPrivate(inputImage, outputImage,
                                          windowWidth, windowHeight,
                                          cropperShape, crooperSize);
        imageCropperDo->exec();

        return outputImage;
    }
};

#endif // IMAGECROPPER_H
```

### `base/imagecropperlabel.cpp`

```cpp
﻿#include "imagecropperlabel.h"

#include <QPainter>
#include <QPainterPath>
#include <QMouseEvent>
#include <QDebug>
#include <QBitmap>

ImageCropperLabel::ImageCropperLabel(int width, int height, QWidget* parent) :
         QLabel(parent)
{
    this->setFixedSize(width, height);
    this->setAlignment(Qt::AlignCenter);
    this->setMouseTracking(true);

    borderPen.setWidth(1);
    borderPen.setColor(Qt::white);
    borderPen.setDashPattern(QVector<qreal>() << 3 << 3 << 3 << 3);
}

void ImageCropperLabel::setOriginalImage(const QPixmap &pixmap) {
    originalImage = pixmap;

    int imgWidth = pixmap.width();
    int imgHeight = pixmap.height();
    int labelWidth = this->width();
    int labelHeight = this->height();
    int imgWidthInLabel;
    int imgHeightInLabel;

    if (imgWidth * labelHeight < imgHeight * labelWidth) {
        scaledRate = labelHeight / double(imgHeight);
        imgHeightInLabel = labelHeight;
        imgWidthInLabel = int(scaledRate * imgWidth);
        imageRect.setRect((labelWidth - imgWidthInLabel) / 2, 0,
                          imgWidthInLabel, imgHeightInLabel);
    }
    else {
        scaledRate = labelWidth / double(imgWidth);
        imgWidthInLabel = labelWidth;
        imgHeightInLabel = int(scaledRate * imgHeight);
        imageRect.setRect(0, (labelHeight - imgHeightInLabel) / 2,
                          imgWidthInLabel, imgHeightInLabel);
    }

    tempImage = originalImage.scaled(imgWidthInLabel, imgHeightInLabel,
                                     Qt::KeepAspectRatio, Qt::SmoothTransformation);
    this->setPixmap(tempImage);

    if (cropperShape >= CropperShape::FIXED_RECT) {
        cropperRect.setWidth(int(cropperRect_.width() * scaledRate));
        cropperRect.setHeight(int(cropperRect_.height() * scaledRate));
    }
    resetCropperPos();
}


/*****************************************
 * set cropper's shape (and size)
*****************************************/
void ImageCropperLabel::setRectCropper() {
    cropperShape = CropperShape::RECT;
    resetCropperPos();
}

void ImageCropperLabel::setSquareCropper() {
    cropperShape = CropperShape::SQUARE;
    resetCropperPos();
}

void ImageCropperLabel::setEllipseCropper() {
    cropperShape = CropperShape::ELLIPSE;
    resetCropperPos();
}

void ImageCropperLabel::setCircleCropper() {
    cropperShape = CropperShape::CIRCLE;
    resetCropperPos();
}

void ImageCropperLabel::setFixedRectCropper(QSize size) {
    cropperShape = CropperShape::FIXED_RECT;
    cropperRect_.setSize(size);
    resetCropperPos();
}

void ImageCropperLabel::setFixedEllipseCropper(QSize size) {
    cropperShape = CropperShape::FIXED_ELLIPSE;
    cropperRect_.setSize(size);
    resetCropperPos();
}

// not recommended
void ImageCropperLabel::setCropper(CropperShape shape, QSize size) {
    cropperShape = shape;
    cropperRect_.setSize(size);
    resetCropperPos();
}

/*****************************************************************************
     * Set cropper's fixed size
    *****************************************************************************/
void ImageCropperLabel::setCropperFixedSize(int fixedWidth, int fixedHeight) {
    cropperRect_.setSize(QSize(fixedWidth, fixedHeight));
    resetCropperPos();
}

void ImageCropperLabel::setCropperFixedWidth(int fixedWidth) {
    cropperRect_.setWidth(fixedWidth);
    resetCropperPos();
}

void ImageCropperLabel::setCropperFixedHeight(int fixedHeight) {
    cropperRect_.setHeight(fixedHeight);
    resetCropperPos();
}

/**********************************************
 * Move cropper to the center of the image
 * And resize to default
**********************************************/
void ImageCropperLabel::resetCropperPos() {
    int labelWidth = this->width();
    int labelHeight = this->height();

    if (cropperShape == CropperShape::FIXED_RECT || cropperShape == CropperShape::FIXED_ELLIPSE) {
        cropperRect.setWidth(int(cropperRect_.width() * scaledRate));
        cropperRect.setHeight(int(cropperRect_.height() * scaledRate));
    }

    switch (cropperShape) {
        case CropperShape::UNDEFINED:
            break;
        case CropperShape::FIXED_RECT:
        case CropperShape::FIXED_ELLIPSE: {
            cropperRect.setRect((labelWidth - cropperRect.width()) / 2,
                             (labelHeight - cropperRect.height()) / 2,
                             cropperRect.width(), cropperRect.height());
            break;
        }
        case CropperShape::RECT:
        case CropperShape::SQUARE:
        case CropperShape::ELLIPSE:
        case CropperShape::CIRCLE: {
            int imgWidth = tempImage.width();
            int imgHeight = tempImage.height();
            int edge = int((imgWidth > imgHeight ? imgHeight : imgWidth) * 3 / 4.0);
            cropperRect.setRect((labelWidth - edge) / 2, (labelHeight - edge) / 2, edge, edge);
            break;
        }
    }
}

QPixmap ImageCropperLabel::getCroppedImage() {
    return getCroppedImage(this->outputShape);
}

QPixmap ImageCropperLabel::getCroppedImage(OutputShape shape) {
    int startX = int((cropperRect.left() - imageRect.left()) / scaledRate);
    int startY = int((cropperRect.top() - imageRect.top()) / scaledRate);
    int croppedWidth = int(cropperRect.width() / scaledRate);
    int croppedHeight = int(cropperRect.height() / scaledRate);

    QPixmap resultImage(croppedWidth, croppedHeight);
    resultImage = originalImage.copy(startX, startY, croppedWidth, croppedHeight);

    // Set ellipse mask (cut to ellipse shape)
    if (shape == OutputShape::ELLIPSE) {
        QSize size(croppedWidth, croppedHeight);
        QBitmap mask(size);
        QPainter painter(&mask);
        painter.setRenderHint(QPainter::Antialiasing);
        painter.setRenderHint(QPainter::SmoothPixmapTransform);
        painter.fillRect(0, 0, size.width(), size.height(), Qt::white);
        painter.setBrush(QColor(0, 0, 0));
        painter.drawRoundedRect(0, 0, size.width(), size.height(), 99, 99);
        resultImage.setMask(mask);
    }

    return resultImage;
}


void ImageCropperLabel::paintEvent(QPaintEvent *event) {
    // Draw original image
    QLabel::paintEvent(event);

    // Draw cropper and set some effects
    switch (cropperShape) {
        case CropperShape::UNDEFINED:
            break;
        case CropperShape::FIXED_RECT:
            drawRectOpacity();
            break;
        case CropperShape::FIXED_ELLIPSE:
            drawEllipseOpacity();
            break;
        case CropperShape::RECT:
            drawRectOpacity();
            drawSquareEdge(!ONLY_FOUR_CORNERS);
            break;
        case CropperShape::SQUARE:
            drawRectOpacity();
            drawSquareEdge(ONLY_FOUR_CORNERS);
            break;
        case CropperShape::ELLIPSE:
            drawEllipseOpacity();
            drawSquareEdge(!ONLY_FOUR_CORNERS);
            break;
        case CropperShape::CIRCLE:
            drawEllipseOpacity();
            drawSquareEdge(ONLY_FOUR_CORNERS);
            break;
    }

    // Draw cropper rect
    if (isShowRectBorder) {
        QPainter painter(this);
        painter.setPen(borderPen);
        painter.drawRect(cropperRect);
    }
}

void ImageCropperLabel::drawSquareEdge(bool onlyFourCorners) {
    if (!isShowDragSquare)
        return;

    // Four corners
    drawFillRect(cropperRect.topLeft(), dragSquareEdge, dragSquareColor);
    drawFillRect(cropperRect.topRight(), dragSquareEdge, dragSquareColor);
    drawFillRect(cropperRect.bottomLeft(), dragSquareEdge, dragSquareColor);
    drawFillRect(cropperRect.bottomRight(), dragSquareEdge, dragSquareColor);

    // Four edges
    if (!onlyFourCorners) {
        int centralX = cropperRect.left() + cropperRect.width() / 2;
        int centralY = cropperRect.top() + cropperRect.height() / 2;
        drawFillRect(QPoint(cropperRect.left(), centralY), dragSquareEdge, dragSquareColor);
        drawFillRect(QPoint(centralX, cropperRect.top()), dragSquareEdge, dragSquareColor);
        drawFillRect(QPoint(cropperRect.right(), centralY), dragSquareEdge, dragSquareColor);
        drawFillRect(QPoint(centralX, cropperRect.bottom()), dragSquareEdge, dragSquareColor);
    }
}

void ImageCropperLabel::drawFillRect(QPoint centralPoint, int edge, QColor color) {
    QRect rect(centralPoint.x() - edge / 2, centralPoint.y() - edge / 2, edge, edge);
    QPainter painter(this);
    painter.fillRect(rect, color);
}

// Opacity effect
void ImageCropperLabel::drawOpacity(const QPainterPath& path) {
    QPainter painterOpac(this);
    painterOpac.setOpacity(opacity);
    painterOpac.fillPath(path, QBrush(Qt::black));
}

void ImageCropperLabel::drawRectOpacity() {
    if (isShowOpacityEffect) {
        QPainterPath p1, p2, p;
        p1.addRect(imageRect);
        p2.addRect(cropperRect);
        p = p1.subtracted(p2);
        drawOpacity(p);
    }
}

void ImageCropperLabel::drawEllipseOpacity() {
    if (isShowOpacityEffect) {
        QPainterPath p1, p2, p;
        p1.addRect(imageRect);
        p2.addEllipse(cropperRect);
        p = p1.subtracted(p2);
        drawOpacity(p);
    }
}

bool ImageCropperLabel::isPosNearDragSquare(const QPoint& pt1, const QPoint& pt2) {
    return abs(pt1.x() - pt2.x()) * 2 <= dragSquareEdge
           && abs(pt1.y() - pt2.y()) * 2 <= dragSquareEdge;
}

int ImageCropperLabel::getPosInCropperRect(const QPoint &pt) {
    if (isPosNearDragSquare(pt, QPoint(cropperRect.right(), cropperRect.center().y())))
        return RECT_RIGHT;
    if (isPosNearDragSquare(pt, cropperRect.bottomRight()))
        return RECT_BOTTOM_RIGHT;
    if (isPosNearDragSquare(pt, QPoint(cropperRect.center().x(), cropperRect.bottom())))
        return RECT_BOTTOM;
    if (isPosNearDragSquare(pt, cropperRect.bottomLeft()))
        return RECT_BOTTOM_LEFT;
    if (isPosNearDragSquare(pt, QPoint(cropperRect.left(), cropperRect.center().y())))
        return RECT_LEFT;
    if (isPosNearDragSquare(pt, cropperRect.topLeft()))
        return RECT_TOP_LEFT;
    if (isPosNearDragSquare(pt, QPoint(cropperRect.center().x(), cropperRect.top())))
        return RECT_TOP;
    if (isPosNearDragSquare(pt, cropperRect.topRight()))
        return RECT_TOP_RIGHT;
    if (cropperRect.contains(pt, true))
        return RECT_INSIDE;
    return RECT_OUTSIZD;
}

/*************************************************
 *
 *  Change mouse cursor type
 *      Arrow, SizeHor, SizeVer, etc...
 *
*************************************************/

void ImageCropperLabel::changeCursor() {
    switch (cursorPosInCropperRect) {
        case RECT_OUTSIZD:
            setCursor(Qt::ArrowCursor);
            break;
        case RECT_BOTTOM_RIGHT: {
            switch (cropperShape) {
            case CropperShape::SQUARE:
            case CropperShape::CIRCLE:
            case CropperShape::RECT:
            case CropperShape::ELLIPSE:
                setCursor(Qt::SizeFDiagCursor);
                break;
            default:
                break;
            }
            break;
        }
        case RECT_RIGHT: {
            switch (cropperShape) {
            case CropperShape::RECT:
            case CropperShape::ELLIPSE:
                setCursor(Qt::SizeHorCursor);
                break;
            default:
                break;
            }
            break;
        }
        case RECT_BOTTOM: {
            switch (cropperShape) {
            case CropperShape::RECT:
            case CropperShape::ELLIPSE:
                setCursor(Qt::SizeVerCursor);
                break;
            default:
                break;
            }
            break;
        }
        case RECT_BOTTOM_LEFT: {
            switch (cropperShape) {
            case CropperShape::RECT:
            case CropperShape::ELLIPSE:
            case CropperShape::SQUARE:
            case CropperShape::CIRCLE:
                setCursor(Qt::SizeBDiagCursor);
                break;
            default:
                break;
            }
            break;
        }
        case RECT_LEFT: {
            switch (cropperShape) {
            case CropperShape::RECT:
            case CropperShape::ELLIPSE:
                setCursor(Qt::SizeHorCursor);
                break;
            default:
                break;
            }
            break;
        }
        case RECT_TOP_LEFT: {
            switch (cropperShape) {
            case CropperShape::RECT:
            case CropperShape::ELLIPSE:
            case CropperShape::SQUARE:
            case CropperShape::CIRCLE:
                setCursor(Qt::SizeFDiagCursor);
                break;
            default:
                break;
            }
            break;
        }
        case RECT_TOP: {
            switch (cropperShape) {
            case CropperShape::RECT:
            case CropperShape::ELLIPSE:
                setCursor(Qt::SizeVerCursor);
                break;
            default:
                break;
            }
            break;
        }
        case RECT_TOP_RIGHT: {
            switch (cropperShape) {
            case CropperShape::SQUARE:
            case CropperShape::CIRCLE:
            case CropperShape::RECT:
            case CropperShape::ELLIPSE:
                setCursor(Qt::SizeBDiagCursor);
                break;
            default:
                break;
            }
            break;
        }
        case RECT_INSIDE: {
            setCursor(Qt::SizeAllCursor);
            break;
        }
    }
}

/*****************************************************
 *
 *  Mouse Events
 *
*****************************************************/

void ImageCropperLabel::mousePressEvent(QMouseEvent *e) {
    currPos = lastPos = e->pos();
    isLButtonPressed = true;
}

void ImageCropperLabel::mouseMoveEvent(QMouseEvent *e) {
    currPos = e->pos();
    if (!isCursorPosCalculated) {
        cursorPosInCropperRect = getPosInCropperRect(currPos);
        changeCursor();
    }

    if (!isLButtonPressed)
        return;
    if (!imageRect.contains(currPos))
        return;

    isCursorPosCalculated = true;

    int xOffset = currPos.x() - lastPos.x();
    int yOffset = currPos.y() - lastPos.y();
    lastPos = currPos;

    int disX = 0;
    int disY = 0;

    // Move cropper
    switch (cursorPosInCropperRect) {
        case RECT_OUTSIZD:
            break;
        case RECT_BOTTOM_RIGHT: {
            disX = currPos.x() - cropperRect.left();
            disY = currPos.y() - cropperRect.top();
            switch (cropperShape) {
                case CropperShape::UNDEFINED:
                case CropperShape::FIXED_RECT:
                case CropperShape::FIXED_ELLIPSE:
                    break;
                case CropperShape::SQUARE:
                case CropperShape::CIRCLE:
                    setCursor(Qt::SizeFDiagCursor);
                    if (disX >= cropperMinimumWidth && disY >= cropperMinimumHeight) {
                        if (disX > disY && cropperRect.top() + disX <= imageRect.bottom()) {
                            cropperRect.setRight(currPos.x());
                            cropperRect.setBottom(cropperRect.top() + disX);
                            emit croppedImageChanged();
                        }
                        else if (disX <= disY && cropperRect.left() + disY <= imageRect.right()) {
                            cropperRect.setBottom(currPos.y());
                            cropperRect.setRight(cropperRect.left() + disY);
                            emit croppedImageChanged();
                        }
                    }
                    break;
                case CropperShape::RECT:
                case CropperShape::ELLIPSE:
                    setCursor(Qt::SizeFDiagCursor);
                    if (disX >= cropperMinimumWidth) {
                        cropperRect.setRight(currPos.x());
                        emit croppedImageChanged();
                    }
                    if (disY >= cropperMinimumHeight) {
                        cropperRect.setBottom(currPos.y());
                        emit croppedImageChanged();
                    }
                    break;
            }
            break;
        }
        case RECT_RIGHT: {
            disX = currPos.x() - cropperRect.left();
            switch (cropperShape) {
                case CropperShape::UNDEFINED:
                case CropperShape::FIXED_RECT:
                case CropperShape::FIXED_ELLIPSE:
                case CropperShape::SQUARE:
                case CropperShape::CIRCLE:
                    break;
                case CropperShape::RECT:
                case CropperShape::ELLIPSE:
                    if (disX >= cropperMinimumWidth) {
                        cropperRect.setRight(currPos.x());
                        emit croppedImageChanged();
                    }
                    break;
            }
            break;
        }
        case RECT_BOTTOM: {
            disY = currPos.y() - cropperRect.top();
            switch (cropperShape) {
                case CropperShape::UNDEFINED:
                case CropperShape::FIXED_RECT:
                case CropperShape::FIXED_ELLIPSE:
                case CropperShape::SQUARE:
                case CropperShape::CIRCLE:
                    break;
                case CropperShape::RECT:
                case CropperShape::ELLIPSE:
                    if (disY >= cropperMinimumHeight) {
                        cropperRect.setBottom(cropperRect.bottom() + yOffset);
                        emit croppedImageChanged();
                    }
                    break;
            }
            break;
        }
        case RECT_BOTTOM_LEFT: {
            disX = cropperRect.right() - currPos.x();
            disY = currPos.y() - cropperRect.top();
            switch (cropperShape) {
                case CropperShape::UNDEFINED:
                    break;
                case CropperShape::FIXED_RECT:
                case CropperShape::FIXED_ELLIPSE:
                case CropperShape::RECT:
                case CropperShape::ELLIPSE:
                    if (disX >= cropperMinimumWidth) {
                        cropperRect.setLeft(currPos.x());
                        emit croppedImageChanged();
                    }
                    if (disY >= cropperMinimumHeight) {
                        cropperRect.setBottom(currPos.y());
                        emit croppedImageChanged();
                    }
                    break;
                case CropperShape::SQUARE:
                case CropperShape::CIRCLE:
                    if (disX >= cropperMinimumWidth && disY >= cropperMinimumHeight) {
                        if (disX > disY && cropperRect.top() + disX <= imageRect.bottom()) {
                            cropperRect.setLeft(currPos.x());
                            cropperRect.setBottom(cropperRect.top() + disX);
                            emit croppedImageChanged();
                        }
                        else if (disX <= disY && cropperRect.right() - disY >= imageRect.left()) {
                            cropperRect.setBottom(currPos.y());
                            cropperRect.setLeft(cropperRect.right() - disY);
                            emit croppedImageChanged();
                        }
                    }
                    break;
            }
            break;
        }
        case RECT_LEFT: {
            disX = cropperRect.right() - currPos.x();
            switch (cropperShape) {
                case CropperShape::UNDEFINED:
                case CropperShape::FIXED_RECT:
                case CropperShape::FIXED_ELLIPSE:
                case CropperShape::SQUARE:
                case CropperShape::CIRCLE:
                    break;
                case CropperShape::RECT:
                case CropperShape::ELLIPSE:
                    if (disX >= cropperMinimumHeight) {
                        cropperRect.setLeft(cropperRect.left() + xOffset);
                        emit croppedImageChanged();
                    }
                    break;
            }
            break;
        }
        case RECT_TOP_LEFT: {
            disX = cropperRect.right() - currPos.x();
            disY = cropperRect.bottom() - currPos.y();
            switch (cropperShape) {
                case CropperShape::UNDEFINED:
                case CropperShape::FIXED_RECT:
                case CropperShape::FIXED_ELLIPSE:
                    break;
                case CropperShape::RECT:
                case CropperShape::ELLIPSE:
                    if (disX >= cropperMinimumWidth) {
                        cropperRect.setLeft(currPos.x());
                        emit croppedImageChanged();
                    }
                    if (disY >= cropperMinimumHeight) {
                        cropperRect.setTop(currPos.y());
                        emit croppedImageChanged();
                    }
                    break;
                case CropperShape::SQUARE:
                case CropperShape::CIRCLE:
                    if (disX >= cropperMinimumWidth && disY >= cropperMinimumHeight) {
                        if (disX > disY && cropperRect.bottom() - disX >= imageRect.top()) {
                            cropperRect.setLeft(currPos.x());
                            cropperRect.setTop(cropperRect.bottom() - disX);
                            emit croppedImageChanged();
                        }
                        else if (disX <= disY && cropperRect.right() - disY >= imageRect.left()) {
                            cropperRect.setTop(currPos.y());
                            cropperRect.setLeft(cropperRect.right() - disY);
                            emit croppedImageChanged();
                        }
                    }
                    break;
            }
            break;
        }
        case RECT_TOP: {
            disY = cropperRect.bottom() - currPos.y();
            switch (cropperShape) {
                case CropperShape::UNDEFINED:
                case CropperShape::FIXED_RECT:
                case CropperShape::FIXED_ELLIPSE:
                case CropperShape::SQUARE:
                case CropperShape::CIRCLE:
                    break;
                case CropperShape::RECT:
                case CropperShape::ELLIPSE:
                    if (disY >= cropperMinimumHeight) {
                        cropperRect.setTop(cropperRect.top() + yOffset);
                        emit croppedImageChanged();
                    }
                    break;
            }
            break;
        }
        case RECT_TOP_RIGHT: {
            disX = currPos.x() - cropperRect.left();
            disY = cropperRect.bottom() - currPos.y();
            switch (cropperShape) {
                case CropperShape::UNDEFINED:
                case CropperShape::FIXED_RECT:
                case CropperShape::FIXED_ELLIPSE:
                    break;
                case CropperShape::RECT:
                case CropperShape::ELLIPSE:
                    if (disX >= cropperMinimumWidth) {
                        cropperRect.setRight(currPos.x());
                        emit croppedImageChanged();
                    }
                    if (disY >= cropperMinimumHeight) {
                        cropperRect.setTop(currPos.y());
                        emit croppedImageChanged();
                    }
                    break;
                case CropperShape::SQUARE:
                case CropperShape::CIRCLE:
                    if (disX >= cropperMinimumWidth && disY >= cropperMinimumHeight) {
                        if (disX < disY && cropperRect.left() + disY <= imageRect.right()) {
                            cropperRect.setTop(currPos.y());
                            cropperRect.setRight(cropperRect.left() + disY);
                            emit croppedImageChanged();
                        }
                        else if (disX >= disY && cropperRect.bottom() - disX >= imageRect.top()) {
                            cropperRect.setRight(currPos.x());
                            cropperRect.setTop(cropperRect.bottom() - disX);
                            emit croppedImageChanged();
                        }
                    }
                    break;
            }
            break;
        }
        case RECT_INSIDE: {
            // Make sure the cropperRect is entirely inside the imageRecct
            if (xOffset > 0) {
                if (cropperRect.right() + xOffset > imageRect.right())
                    xOffset = 0;
            }
            else if (xOffset < 0) {
                if (cropperRect.left() + xOffset < imageRect.left())
                    xOffset = 0;
            }
            if (yOffset > 0) {
                if (cropperRect.bottom() + yOffset > imageRect.bottom())
                    yOffset = 0;
            }
            else if (yOffset < 0) {
                if (cropperRect.top() + yOffset < imageRect.top())
                    yOffset = 0;
            }
            cropperRect.moveTo(cropperRect.left() + xOffset, cropperRect.top() + yOffset);
            emit croppedImageChanged();
        }
        break;
    }

    repaint();
}

void ImageCropperLabel::mouseReleaseEvent(QMouseEvent *) {
    isLButtonPressed = false;
    isCursorPosCalculated = false;
    setCursor(Qt::ArrowCursor);
}
```

### `base/imagecropperlabel.h`

```cpp
﻿#ifndef IMAGECROPPERLABEL_H
#define IMAGECROPPERLABEL_H

#include <QLabel>
#include <QPixmap>
#include <QPen>

enum class CropperShape {
    UNDEFINED = 0,
    RECT = 1,
    SQUARE = 2,
    FIXED_RECT = 3,
    ELLIPSE = 4,
    CIRCLE = 5,
    FIXED_ELLIPSE = 6
};

enum class OutputShape {
    RECT = 0,
    ELLIPSE = 1
};

enum class SizeType {
    fixedSize = 0,
    fitToMaxWidth = 1,
    fitToMaxHeight = 2,
    fitToMaxWidthHeight = 3,
};

class ImageCropperLabel : public QLabel
{
    Q_OBJECT
public:
    ImageCropperLabel(int width, int height, QWidget *parent);

    void setOriginalImage(const QPixmap &pixmap);
    void setOutputShape(OutputShape shape) { outputShape = shape; }
    QPixmap getCroppedImage();
    QPixmap getCroppedImage(OutputShape shape);

    /*****************************************
     * Set cropper's shape
    *****************************************/
    void setRectCropper();
    void setSquareCropper();
    void setEllipseCropper();
    void setCircleCropper();
    void setFixedRectCropper(QSize size);
    void setFixedEllipseCropper(QSize size);
    void setCropper(CropperShape shape, QSize size); // not recommended

    /*****************************************************************************
     * Set cropper's fixed size
    *****************************************************************************/
    void setCropperFixedSize(int fixedWidth, int fixedHeight);
    void setCropperFixedWidth(int fixedWidht);
    void setCropperFixedHeight(int fixedHeight);

    /*****************************************************************************
     * Set cropper's minimum size
     * default: the twice of minimum of the edge lenght of drag square
    *****************************************************************************/
    void setCropperMinimumSize(int minWidth, int minHeight)
    {
        cropperMinimumWidth = minWidth;
        cropperMinimumHeight = minHeight;
    }
    void setCropperMinimumWidth(int minWidth) { cropperMinimumWidth = minWidth; }
    void setCropperMinimumHeight(int minHeight) { cropperMinimumHeight = minHeight; }

    /*************************************************
     * Set the size, color, visibility of rectangular border
    *************************************************/
    void setShowRectBorder(bool show) { isShowRectBorder = show; }
    QPen getBorderPen() { return borderPen; }
    void setBorderPen(const QPen &pen) { borderPen = pen; }

    /*************************************************
     * Set the size, color of drag square
    *************************************************/
    void setShowDragSquare(bool show) { isShowDragSquare = show; }
    void setDragSquareEdge(int edge) { dragSquareEdge = (edge >= 3 ? edge : 3); }
    void setDragSquareColor(const QColor &color) { dragSquareColor = color; }

    /*****************************************
     *  Opacity Effect
    *****************************************/
    void enableOpacity(bool b = true) { isShowOpacityEffect = b; }
    void setOpacity(double newOpacity) { opacity = newOpacity; }

signals:
    void croppedImageChanged();

protected:
    /*****************************************
     * Event
    *****************************************/
    virtual void paintEvent(QPaintEvent *event) override;
    virtual void mousePressEvent(QMouseEvent *e) override;
    virtual void mouseMoveEvent(QMouseEvent *e) override;
    virtual void mouseReleaseEvent(QMouseEvent *e) override;

private:
    /***************************************
     * Draw shapes
    ***************************************/
    void drawFillRect(QPoint centralPoint, int edge, QColor color);
    void drawRectOpacity();
    void drawEllipseOpacity();
    void drawOpacity(const QPainterPath &path); // shadow effect
    void drawSquareEdge(bool onlyFourCorners);

    /***************************************
     * Other utility methods
    ***************************************/
    int getPosInCropperRect(const QPoint &pt);
    bool isPosNearDragSquare(const QPoint &pt1, const QPoint &pt2);
    void resetCropperPos();
    void changeCursor();

    enum {
        RECT_OUTSIZD = 0,
        RECT_INSIDE = 1,
        RECT_TOP_LEFT,
        RECT_TOP,
        RECT_TOP_RIGHT,
        RECT_RIGHT,
        RECT_BOTTOM_RIGHT,
        RECT_BOTTOM,
        RECT_BOTTOM_LEFT,
        RECT_LEFT
    };

    const bool ONLY_FOUR_CORNERS = true;

private:
    QPixmap originalImage;
    QPixmap tempImage;

    bool isShowRectBorder = true;
    QPen borderPen;

    CropperShape cropperShape = CropperShape::UNDEFINED;
    OutputShape outputShape = OutputShape::RECT;

    QRect imageRect; // the whole image area in the label (not real size)
    QRect cropperRect; // a rectangle frame to choose image area (not real size)
    QRect cropperRect_; // cropper rect (real size)
    double scaledRate = 1.0;

    bool isLButtonPressed = false;
    bool isCursorPosCalculated = false;
    int cursorPosInCropperRect = RECT_OUTSIZD;
    QPoint lastPos;
    QPoint currPos;

    bool isShowDragSquare = true;
    int dragSquareEdge = 8;
    QColor dragSquareColor = Qt::white;

    int cropperMinimumWidth = dragSquareEdge * 2;
    int cropperMinimumHeight = dragSquareEdge * 2;

    bool isShowOpacityEffect = false;
    double opacity = 0.6;
};

#endif // IMAGECROPPERLABEL_H
```

### `example/imagecropperdemo.cpp`

```cpp
﻿#include "imagecropperdemo.h"
#include <QFormLayout>
#include <QColorDialog>
#include <QVBoxLayout>
#include <QFileDialog>
#include <QMessageBox>

ImageCropperDemo::ImageCropperDemo(QWidget *parent) :
    QDialog(parent)
{
    setupLayout();
    init();

    this->setAttribute(Qt::WA_DeleteOnClose, true);
    this->setWindowTitle("Image Cropper Demo");
}


void ImageCropperDemo::setupLayout()
{
    imgCropperLabel = new ImageCropperLabel(600, 500, this);
    imgCropperLabel->setFrameStyle(1);

    comboOutputShape = new QComboBox(this);
    comboCropperShape = new QComboBox(this);

    labelPreviewImage = new QLabel(this);

    editOriginalImagePath = new QLineEdit(this);
    btnChooseOriginalImagePath = new QPushButton(this);
    QHBoxLayout *hOriginalImagePathLayout = new QHBoxLayout();
    hOriginalImagePathLayout->addWidget(editOriginalImagePath);
    hOriginalImagePathLayout->addWidget(btnChooseOriginalImagePath);

    editCropperFixedWidth = new QLineEdit(this);
    editCropperFixedHeight = new QLineEdit(this);
    QHBoxLayout *hCropperFixedSizeLayout = new QHBoxLayout();
    hCropperFixedSizeLayout->addWidget(editCropperFixedWidth);
    hCropperFixedSizeLayout->addWidget(editCropperFixedHeight);

    editCropperMinWidth = new QLineEdit("8", this);
    editCropperMinHeight = new QLineEdit("8", this);
    QHBoxLayout *hCropperMinSizeLayout = new QHBoxLayout();
    hCropperMinSizeLayout->addWidget(editCropperMinWidth);
    hCropperMinSizeLayout->addWidget(editCropperMinHeight);

    checkEnableOpacity = new QCheckBox(this);
    sliderOpacity = new QSlider(Qt::Horizontal, this);

    checkShowDragSquare = new QCheckBox(this);
    editDragSquareEdge = new QLineEdit("8", this);
    checkShowRectBorder = new QCheckBox(this);

    labelRectBorderColor = new QLabel(this);
    btnChooseRectBorderCorlor = new QPushButton(this);
    QHBoxLayout *hRectBorderColorLayout = new QHBoxLayout();
    hRectBorderColorLayout->addWidget(labelRectBorderColor);
    hRectBorderColorLayout->addWidget(btnChooseRectBorderCorlor);

    labelDragSquareColor = new QLabel(this);
    btnChooseDragSquareColor = new QPushButton(this);
    QHBoxLayout *hDragSquareColorLayout = new QHBoxLayout();
    hDragSquareColorLayout->addWidget(labelDragSquareColor);
    hDragSquareColorLayout->addWidget(btnChooseDragSquareColor);

    QFormLayout *formLayout1 = new QFormLayout();
    formLayout1->addRow(new QLabel("Preview:"), labelPreviewImage);
    formLayout1->addRow(new QLabel("OriginalImage:", this), hOriginalImagePathLayout);
    formLayout1->addRow(new QLabel("OutputShape:", this), comboOutputShape);
    formLayout1->addRow(new QLabel("CropperShape:", this), comboCropperShape);
    formLayout1->addRow(new QLabel("FixedSize:", this), hCropperFixedSizeLayout);
    formLayout1->addRow(new QLabel("MinimumSize:", this), hCropperMinSizeLayout);

    QFormLayout *formLayout2 = new QFormLayout();
    formLayout2->addRow(new QLabel("EnableOpacity:", this), checkEnableOpacity);
    formLayout2->addRow(new QLabel("Opacity:", this), sliderOpacity);

    QFormLayout *formLayout3 = new QFormLayout();
    formLayout3->addRow(new QLabel("ShowDragSquare:", this), checkShowDragSquare);
    formLayout3->addRow(new QLabel("DragSquareEdge:", this), editDragSquareEdge);
    formLayout3->addRow(new QLabel("DragSquareColor:", this), hDragSquareColorLayout);

    QFormLayout *formLayout4 = new QFormLayout();
    formLayout4->addRow(new QLabel("ShowRectBorder:", this), checkShowRectBorder);
    formLayout4->addRow(new QLabel("RectBorderColor:", this), hRectBorderColorLayout);

    btnSavePreview = new QPushButton("Save", this);
    btnQuit = new QPushButton("Quit", this);
    QHBoxLayout *btnLayout = new QHBoxLayout();
    btnLayout->addStretch();
    btnLayout->addWidget(btnSavePreview);
    btnLayout->addStretch();
    btnLayout->addWidget(btnQuit);
    btnLayout->addStretch();

    QVBoxLayout *vLayout = new QVBoxLayout();
    vLayout->addLayout(formLayout1);
    vLayout->addStretch();
    vLayout->addLayout(formLayout2);
    vLayout->addStretch();
    vLayout->addLayout(formLayout3);
    vLayout->addStretch();
    vLayout->addLayout(formLayout4);
    vLayout->addStretch();
    vLayout->addLayout(btnLayout);

    mainLayout = new QHBoxLayout(this);
    mainLayout->addWidget(imgCropperLabel);
    mainLayout->addLayout(vLayout);
}

void ImageCropperDemo::init()
{
    imgCropperLabel->setRectCropper();
    editCropperFixedWidth->setEnabled(false);
    editCropperFixedHeight->setEnabled(false);

    labelPreviewImage->setFixedSize(96, 96);
    labelPreviewImage->setAlignment(Qt::AlignCenter);
    labelPreviewImage->setFrameStyle(QFrame::Panel | QFrame::Sunken);
    connect(imgCropperLabel, &ImageCropperLabel::croppedImageChanged,
            this, &ImageCropperDemo::onUpdatePreview);

    btnChooseOriginalImagePath->setIcon(QIcon("res/select-file.ico"));
    btnChooseOriginalImagePath->setFixedWidth(30);
    connect(btnChooseOriginalImagePath, &QPushButton::clicked,
            this, &ImageCropperDemo::onChooseOriginalImage);

    comboOutputShape->addItem("Rect/Square");
    comboOutputShape->addItem("Ellipse/Circle");
    connect(comboOutputShape, SIGNAL(currentIndexChanged(int)),
            this, SLOT(onOutputShapeChanged(int)));

    comboCropperShape->addItem("Rect");
    comboCropperShape->addItem("Square");
    comboCropperShape->addItem("FixedRect");
    comboCropperShape->addItem("Ellipse");
    comboCropperShape->addItem("Circle");
    comboCropperShape->addItem("FixedEllipse");
    connect(comboCropperShape, SIGNAL(currentIndexChanged(int)),
            this, SLOT(onCropperShapeChanged(int)));

    connect(editCropperFixedWidth, &QLineEdit::textChanged,
            this, &ImageCropperDemo::onFixedWidthChanged);
    connect(editCropperFixedHeight, &QLineEdit::textChanged,
            this, &ImageCropperDemo::onFixedHeightChanged);
    connect(editCropperMinWidth, &QLineEdit::textChanged,
            this, &ImageCropperDemo::onMinWidthChanged);
    connect(editCropperMinHeight, &QLineEdit::textChanged,
            this, &ImageCropperDemo::onMinHeightChanged);

    checkEnableOpacity->setCheckState(Qt::Checked);
    imgCropperLabel->enableOpacity(true);
    connect(checkEnableOpacity, &QCheckBox::stateChanged,
            this, &ImageCropperDemo::onEnableOpacityChanged);

    checkShowDragSquare->setCheckState(Qt::Checked);
    imgCropperLabel->setShowDragSquare(true);
    connect(checkShowDragSquare, &QCheckBox::stateChanged,
            this, &ImageCropperDemo::onShowDragSquareChanged);
    connect(editDragSquareEdge, &QLineEdit::textChanged,
            this, &ImageCropperDemo::onDragSquareEdgeChanged);

    sliderOpacity->setRange(0, 100);
    sliderOpacity->setValue(60);
    connect(sliderOpacity, &QSlider::valueChanged,
            this, &ImageCropperDemo::onOpacityChanged);

    checkShowRectBorder->setCheckState(Qt::Checked);
    connect(checkShowRectBorder, &QCheckBox::stateChanged,
            this, &ImageCropperDemo::onShowRectBorder);

    setLabelColor(labelRectBorderColor, Qt::white);
    btnChooseRectBorderCorlor->setIcon(QIcon("res/color-palette.ico"));
    btnChooseRectBorderCorlor->setFixedWidth(40);
    connect(btnChooseRectBorderCorlor, &QPushButton::clicked,
            this, &ImageCropperDemo::onChooseRectBorderColor);

    setLabelColor(labelDragSquareColor, Qt::white);
    btnChooseDragSquareColor->setIcon(QIcon("res/color-palette.ico"));
    btnChooseDragSquareColor->setFixedWidth(40);
    connect(btnChooseDragSquareColor, &QPushButton::clicked,
            this, &ImageCropperDemo::onChooseDragSquareColor);

    connect(btnSavePreview, &QPushButton::clicked,
            this, &ImageCropperDemo::onSaveCroppedImage);
    connect(btnQuit, &QPushButton::clicked,
            this, &ImageCropperDemo::close);

    imgCropperLabel->update();
}


/*****************************************************************************
 *
 *    slots
 *
*****************************************************************************/

void ImageCropperDemo::onChooseOriginalImage()
{
    QString filename = QFileDialog::getOpenFileName(this, "Select a picture", "",
                       "picture (*.jpg *.png *.bmp)");
    if (filename.isNull()) {
        return;
    }

    QPixmap pixmap;
    if (!pixmap.load(filename)) {
        QMessageBox::critical(this, "Error", "Load image failed", QMessageBox::Ok);
        return;
    }

    editOriginalImagePath->setText(filename);
    imgCropperLabel->setOriginalImage(pixmap);
    imgCropperLabel->update();
    onUpdatePreview();
    labelPreviewImage->setFrameStyle(0);
}

void ImageCropperDemo::onOutputShapeChanged(int idx)
{
    // Output: Rectangular
    if (idx == 0) {
        imgCropperLabel->setOutputShape(OutputShape::RECT);
    } else {
        imgCropperLabel->setOutputShape(OutputShape::ELLIPSE);
    }
    onUpdatePreview();
}

void ImageCropperDemo::onCropperShapeChanged(int idx)
{
    switch (CropperShape(idx + 1)) {
        case CropperShape::RECT: {
            imgCropperLabel->setRectCropper();
            editCropperFixedWidth->setEnabled(false);
            editCropperFixedHeight->setEnabled(false);
            editCropperMinWidth->setEnabled(true);
            editCropperMinHeight->setEnabled(true);
            checkShowDragSquare->setEnabled(true);
            editDragSquareEdge->setEnabled(true);
            btnChooseDragSquareColor->setEnabled(true);
            break;
        }
        case CropperShape::SQUARE: {
            imgCropperLabel->setSquareCropper();
            editCropperFixedWidth->setEnabled(false);
            editCropperFixedHeight->setEnabled(false);
            editCropperMinWidth->setEnabled(true);
            editCropperMinHeight->setEnabled(true);
            checkShowDragSquare->setEnabled(true);
            editDragSquareEdge->setEnabled(true);
            btnChooseDragSquareColor->setEnabled(true);
            break;
        }
        case CropperShape::FIXED_RECT: {
            imgCropperLabel->setFixedRectCropper(QSize(64, 64));
            editCropperFixedWidth->setEnabled(true);
            editCropperFixedHeight->setEnabled(true);
            editCropperMinWidth->setEnabled(false);
            editCropperMinHeight->setEnabled(false);
            editCropperFixedWidth->setText("64");
            editCropperFixedHeight->setText("64");
            checkShowDragSquare->setEnabled(false);
            editDragSquareEdge->setEnabled(false);
            btnChooseDragSquareColor->setEnabled(false);
            break;
        }
        case CropperShape::ELLIPSE: {
            imgCropperLabel->setEllipseCropper();
            editCropperFixedWidth->setEnabled(false);
            editCropperFixedHeight->setEnabled(false);
            editCropperMinWidth->setEnabled(true);
            editCropperMinHeight->setEnabled(true);
            checkShowDragSquare->setEnabled(true);
            editDragSquareEdge->setEnabled(true);
            btnChooseDragSquareColor->setEnabled(true);
            break;
        }
        case CropperShape::CIRCLE: {
            imgCropperLabel->setCircleCropper();
            editCropperFixedWidth->setEnabled(false);
            editCropperFixedHeight->setEnabled(false);
            editCropperMinWidth->setEnabled(true);
            editCropperMinHeight->setEnabled(true);
            checkShowDragSquare->setEnabled(true);
            editDragSquareEdge->setEnabled(true);
            btnChooseDragSquareColor->setEnabled(true);
            break;
        }
        case CropperShape::FIXED_ELLIPSE:
            imgCropperLabel->setFixedEllipseCropper(QSize(64, 64));
            editCropperFixedWidth->setEnabled(true);
            editCropperFixedHeight->setEnabled(true);
            editCropperMinWidth->setEnabled(false);
            editCropperMinHeight->setEnabled(false);
            editCropperFixedWidth->setText("64");
            editCropperFixedHeight->setText("64");
            checkShowDragSquare->setEnabled(false);
            editDragSquareEdge->setEnabled(false);
            btnChooseDragSquareColor->setEnabled(false);
            break;
        case CropperShape::UNDEFINED:
            break;
    }

    imgCropperLabel->update();
    onUpdatePreview();
}

void ImageCropperDemo::onEnableOpacityChanged(int state)
{
    if (state == Qt::Checked) {
        sliderOpacity->setEnabled(true);
        imgCropperLabel->enableOpacity(true);
    } else {
        sliderOpacity->setEnabled(false);
        imgCropperLabel->enableOpacity(false);
    }
    imgCropperLabel->update();
}

void ImageCropperDemo::onShowDragSquareChanged(int state)
{
    if (state == Qt::Checked) {
        editDragSquareEdge->setEnabled(true);
        btnChooseDragSquareColor->setEnabled(true);
        imgCropperLabel->setShowDragSquare(true);
    } else {
        editDragSquareEdge->setEnabled(false);
        btnChooseDragSquareColor->setEnabled(false);
        imgCropperLabel->setShowDragSquare(false);
    }
    imgCropperLabel->update();
}

void ImageCropperDemo::onDragSquareEdgeChanged(QString edge)
{
    imgCropperLabel->setDragSquareEdge(edge.toInt());
    imgCropperLabel->update();
}

void ImageCropperDemo::onOpacityChanged(int val)
{
    imgCropperLabel->setOpacity(val / 100.0);
    imgCropperLabel->update();
}

void ImageCropperDemo::onFixedWidthChanged(QString width)
{
    imgCropperLabel->setCropperFixedWidth(width.toInt());
    imgCropperLabel->update();
}

void ImageCropperDemo::onFixedHeightChanged(QString height)
{
    imgCropperLabel->setCropperFixedHeight(height.toInt());
    imgCropperLabel->update();
}

void ImageCropperDemo::onMinWidthChanged(QString width)
{
    imgCropperLabel->setCropperMinimumWidth(width.toInt());
    imgCropperLabel->update();
}

void ImageCropperDemo::onMinHeightChanged(QString height)
{
    imgCropperLabel->setMinimumHeight(height.toInt());
    imgCropperLabel->update();
}

void ImageCropperDemo::onShowRectBorder(int state)
{
    if (state == Qt::Checked) {
        btnChooseRectBorderCorlor->setEnabled(true);
        imgCropperLabel->setShowRectBorder(true);
    } else {
        btnChooseRectBorderCorlor->setEnabled(false);
        imgCropperLabel->setShowRectBorder(false);
    }
    imgCropperLabel->update();
}

void ImageCropperDemo::onChooseRectBorderColor()
{
    QColor color = QColorDialog::getColor(imgCropperLabel->getBorderPen().color(), this);
    if (color.isValid()) {
        setLabelColor(labelRectBorderColor, color);
        QPen pen = imgCropperLabel->getBorderPen();
        pen.setColor(color);
        imgCropperLabel->setBorderPen(pen);
        imgCropperLabel->update();
    }
}

void ImageCropperDemo::onChooseDragSquareColor()
{
    QColor color = QColorDialog::getColor(Qt::white, this);
    if (color.isValid()) {
        setLabelColor(labelDragSquareColor, color);
        imgCropperLabel->setDragSquareColor(color);
        imgCropperLabel->update();
    }
}

void ImageCropperDemo::onUpdatePreview()
{
    QPixmap preview = imgCropperLabel->getCroppedImage();
    preview = preview.scaled(labelPreviewImage->width(), labelPreviewImage->height(),
                             Qt::KeepAspectRatio, Qt::SmoothTransformation);
    labelPreviewImage->setPixmap(preview);
}

void ImageCropperDemo::onSaveCroppedImage()
{
    if (!labelPreviewImage->pixmap()) {
        QMessageBox::information(this, "Error", "There is no cropped image to save.", QMessageBox::Ok);
        return ;
    }

    QString filename = QFileDialog::getSaveFileName(this, "Save cropped image", "", "picture (*.png)");
    if (!filename.isNull()) {
        if (imgCropperLabel->getCroppedImage().save(filename, "PNG")) {
            QMessageBox::information(this, "Prompt", "Saved successfully", QMessageBox::Ok);
        } else {
            QMessageBox::information(this, "Error", "Save image failed!", QMessageBox::Ok);
        }
    }
}
```

### `example/imagecropperdemo.h`

```cpp
#ifndef TESTIMAGECROPPERLABEL_H
#define TESTIMAGECROPPERLABEL_H

#include <QObject>
#include <QDialog>
#include <QHBoxLayout>
#include <QLabel>
#include <QComboBox>
#include <QLineEdit>
#include <QCheckBox>
#include <QPushButton>
#include <QSlider>

#include "../base/imagecropperlabel.h"

class ImageCropperDemo : public QDialog
{
    Q_OBJECT
public:
    ImageCropperDemo(QWidget* parent = 0);

    void setupLayout();

    void init();

public slots:
    void onOutputShapeChanged(int idx);
    void onCropperShapeChanged(int idx);
    void onEnableOpacityChanged(int state);
    void onShowDragSquareChanged(int state);
    void onDragSquareEdgeChanged(QString edge);
    void onOpacityChanged(int val);
    void onFixedWidthChanged(QString width);
    void onFixedHeightChanged(QString height);
    void onMinWidthChanged(QString width);
    void onMinHeightChanged(QString height);
    void onShowRectBorder(int state);
    void onChooseRectBorderColor();
    void onChooseDragSquareColor();

    void onChooseOriginalImage();
    void onUpdatePreview();
    void onSaveCroppedImage();

private:
    void setLabelColor(QLabel* label, QColor color) {
        QPixmap pixmap(QSize(80, 25));
        pixmap.fill(color);
        label->setPixmap(pixmap);
    }

private:
    ImageCropperLabel* imgCropperLabel;
    QHBoxLayout* mainLayout;

    QLabel* labelPreviewImage;

    QComboBox* comboOutputShape;
    QComboBox* comboCropperShape;

    QLineEdit* editOriginalImagePath;
    QPushButton* btnChooseOriginalImagePath;

    QLineEdit* editCropperFixedWidth;
    QLineEdit* editCropperFixedHeight;
    QLineEdit* editCropperMinWidth;
    QLineEdit* editCropperMinHeight;

    QCheckBox* checkShowDragSquare;
    QCheckBox* checkEnableOpacity;
    QSlider* sliderOpacity;
    QLineEdit* editDragSquareEdge;

    QCheckBox* checkShowRectBorder;
    QLabel* labelRectBorderColor;
    QPushButton* btnChooseRectBorderCorlor;

    QLabel* labelDragSquareColor;
    QPushButton* btnChooseDragSquareColor;

    QPushButton* btnSavePreview;
    QPushButton* btnQuit;
};

#endif // TESTIMAGECROPPERLABEL_H
```

### `example/main.cpp`

```cpp
#include "mainwindow.h"

#include <QApplication>

int main(int argc, char *argv[])
{
    QApplication a(argc, argv);
    MainWindow w;
    w.show();
    return a.exec();
}
```

### `example/mainwindow.cpp`

```cpp
#include "mainwindow.h"
#include "imagecropperdemo.h"

#include "../base/imagecropperdialog.h"

#include <QHBoxLayout>
#include <QVBoxLayout>
#include <QFileDialog>


MainWindow::MainWindow(QWidget *parent)
    : QMainWindow(parent)
{
    setupLayout();
}

MainWindow::~MainWindow()
{
}

void MainWindow::setupLayout() {
    QWidget* centralWidget = new QWidget(this);

    btnCustomCrop = new QPushButton("Custom Crop", centralWidget);
    btnSimpleCrop = new QPushButton("Simple Crop", centralWidget);

    QVBoxLayout* mainLayout = new QVBoxLayout(centralWidget);
    mainLayout->addWidget(btnCustomCrop);
    mainLayout->addWidget(btnSimpleCrop);
    this->setCentralWidget(centralWidget);

    connect(btnCustomCrop, &QPushButton::clicked, this, &MainWindow::onCustomCrop);
    connect(btnSimpleCrop, &QPushButton::clicked, this, &MainWindow::onSimpleCrop);
}

void MainWindow::onCustomCrop() {
    ImageCropperDemo* dialog = new ImageCropperDemo(this);
    dialog->show();
}

void MainWindow::onSimpleCrop() {
    QMessageBox::information(this, "Prompt", "Please select a picture", QMessageBox::Ok);
    QString filename = QFileDialog::getOpenFileName(this, "Select image", "", "image (*.png *.jpg)");
    if (filename.isNull())
        return;

    //                                   *********
    //                                    *******
    //                                     *****
    //                                      ***
    //                                       *
    QPixmap image = ImageCropperDialog::getCroppedImage(filename, 600, 400, CropperShape::CIRCLE);
    if (image.isNull())
        return;

    QDialog* dialog = new QDialog(0);
    dialog->setAttribute(Qt::WA_DeleteOnClose, true);
    QLabel* label = new QLabel(dialog);
    label->setFixedSize(image.size());
    label->setPixmap(image);
    dialog->exec();
}
```

### `example/mainwindow.h`

```cpp
#ifndef MAINWINDOW_H
#define MAINWINDOW_H

#include <QMainWindow>
#include <QPushButton>

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow(QWidget *parent = 0);
    ~MainWindow();

public slots:
    void onCustomCrop();
    void onSimpleCrop();

private:
    void setupLayout();

private:
    QPushButton* btnCustomCrop;
    QPushButton* btnSimpleCrop;
};
#endif // MAINWINDOW_H
```
