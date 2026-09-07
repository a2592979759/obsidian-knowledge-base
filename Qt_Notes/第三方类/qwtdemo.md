---
tags:
  - Qt
  - 第三方类
---
# 无插件qwt示例 (`qwtdemo`)

> 无插件qwt示例

## 效果图

![[QWidgetDemo_assert/qwtdemo.jpg]]

## 分类

- 类型: 第三方类 `third`
- 源码目录: `third/qwtdemo`

## 技术要点

**用到的 Qt 类**: QWidget QColor QPointF QPalette QString QLabel QEvent QPainter QSize QComboBox QFrame QHBoxLayout QVBoxLayout QRectF QApplication QObject QMainWindow QToolBar QTimer QVector

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmmain.cpp`

```cpp
﻿#include "frmmain.h"
#include "ui_frmmain.h"
#include "qwt.h"
#include "qwt_dial.h"
#include "qwt_plot.h"

frmMain::frmMain(QWidget *parent) : QWidget(parent), ui(new Ui::frmMain)
{
    ui->setupUi(this);
}

frmMain::~frmMain()
{
    delete ui;
}
```

### `./frmmain.h`

```cpp
#ifndef FRMMAIN_H
#define FRMMAIN_H

#include <QWidget>

namespace Ui {
class frmMain;
}

class frmMain : public QWidget
{
    Q_OBJECT

public:
    explicit frmMain(QWidget *parent = 0);
    ~frmMain();

private:
    Ui::frmMain *ui;
};

#endif // FRMMAIN_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmmain.h"
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

    frmMain w;
    w.setWindowTitle("qwt无插件示例 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `examples/animation/main.cpp`

```cpp
#include <qapplication.h>
#include "plot.h"

#ifndef QWT_NO_OPENGL
#define USE_OPENGL 1
#endif

#if USE_OPENGL
#include <qgl.h>
#include <qwt_plot_glcanvas.h>
#else
#include <qwt_plot_canvas.h>
#endif

int main ( int argc, char **argv )
{
#if USE_OPENGL
#if QT_VERSION >= 0x040600 && QT_VERSION < 0x050000
    // on my box QPaintEngine::OpenGL2 has serious problems, f.e:
    // the lines of a simple drawRect are wrong.

    QGL::setPreferredPaintEngine( QPaintEngine::OpenGL );
#endif
#endif

    QApplication a( argc, argv );

    Plot plot;

#if USE_OPENGL
    QwtPlotGLCanvas *canvas = new QwtPlotGLCanvas();
    canvas->setFrameStyle( QwtPlotGLCanvas::NoFrame );
#else
    QwtPlotCanvas *canvas = new QwtPlotCanvas();
    canvas->setFrameStyle( QFrame::NoFrame );
    canvas->setPaintAttribute( QwtPlotCanvas::BackingStore, false );
#endif

    plot.setCanvas( canvas );
    plot.setCanvasBackground( QColor( 30, 30, 50 ) );

    plot.resize( 400, 400 );
    plot.show();

    return a.exec();
}
```

### `examples/animation/plot.cpp`

```cpp
#include <qapplication.h>
#include <qwt_math.h>
#include <qwt_symbol.h>
#include <qwt_curve_fitter.h>
#include <qwt_plot_curve.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_layout.h>
#include <qevent.h>
#include "plot.h"

class Curve: public QwtPlotCurve
{
public:
    void setTransformation( const QTransform &transform )
    {
        d_transform = transform;
    }

    virtual void updateSamples( double phase )
    {
        setSamples( d_transform.map( points( phase ) ) );
    }

private:
    virtual QPolygonF points( double phase ) const = 0;

private:
    QTransform d_transform;
};

class Curve1: public Curve
{
public:
    Curve1()
    {
        setPen( QColor( 150, 150, 200 ), 2 );
        setStyle( QwtPlotCurve::Lines );

        QwtSplineCurveFitter *curveFitter = new QwtSplineCurveFitter();
        curveFitter->setSplineSize( 150 );
        setCurveFitter( curveFitter );

        setCurveAttribute( QwtPlotCurve::Fitted, true );

        QwtSymbol *symbol = new QwtSymbol( QwtSymbol::XCross );
        symbol->setPen( Qt::yellow );
        symbol->setSize( 7 );

        setSymbol( symbol );

        // somewhere to the left
        QTransform transform;
        transform.scale( 1.5, 1.0 );
        transform.translate( 1.5, 3.0 );

        setTransformation( transform );
    }

    virtual QPolygonF points( double phase ) const
    {
        QPolygonF points;

        const int numSamples = 15;
        for ( int i = 0; i < numSamples; i++ )
        {
            const double v = 6.28 * double( i ) / double( numSamples - 1 );
            points += QPointF( qSin( v - phase ), v );
        }

        return points;
    }
};

class Curve2: public Curve
{
public:
    Curve2()
    {
        setStyle( QwtPlotCurve::Sticks );
        setPen( QColor( 200, 150, 50 ) );

        setSymbol( new QwtSymbol( QwtSymbol::Ellipse,
            QColor( Qt::gray ), QColor( Qt::yellow ), QSize( 5, 5 ) ) );
    }

private:
    virtual QPolygonF points( double phase ) const
    {
        QPolygonF points;

        const int numSamples = 50;
        for ( int i = 0; i < numSamples; i++ )
        {
            const double v = 10.0 * i / double( numSamples - 1 );
            points += QPointF( v, qCos( 3.0 * ( v + phase ) ) );
        }

        return points;
    }
};

class Curve3: public Curve
{       
public: 
    Curve3()
    {
        setStyle( QwtPlotCurve::Lines );
        setPen( QColor( 100, 200, 150 ), 2 );

        QwtSplineCurveFitter* curveFitter = new QwtSplineCurveFitter();
        curveFitter->setFitMode( QwtSplineCurveFitter::ParametricSpline );
        curveFitter->setSplineSize( 200 );
        setCurveFitter( curveFitter );

        setCurveAttribute( QwtPlotCurve::Fitted, true );

        // somewhere in the top right corner
        QTransform transform;
        transform.translate( 7.0, 7.5 );
        transform.scale( 2.0, 2.0 );

        setTransformation( transform );
    }   

private:
    virtual QPolygonF points( double phase ) const
    {
        QPolygonF points;

        const int numSamples = 9;
        for ( int i = 0; i < numSamples; i++ )
        {
            const double v = i * 2.0 * M_PI / ( numSamples - 1 );
            points += QPointF( qSin( v - phase ), qCos( 3.0 * ( v + phase ) ) );
        }

        return points;
    }
};  

class Curve4: public Curve
{       
public: 
    Curve4()
    {
        setStyle( QwtPlotCurve::Lines );
        setPen( Qt::red, 2 );

        initSamples();

        // somewhere in the center
        QTransform transform;
        transform.translate( 7.0, 3.0 );
        transform.scale( 1.5, 1.5 );

        setTransformation( transform );
    }   

private:
    virtual QPolygonF points( double phase ) const
    {
        const double speed = 0.05;

        const double s = speed * qSin( phase );
        const double c = qSqrt( 1.0 - s * s );

        for ( int i = 0; i < d_points.size(); i++ )
        {
            const QPointF p = d_points[i];

            const double u = p.x();
            const double v = p.y();

            d_points[i].setX( u * c - v * s );
            d_points[i].setY( v * c + u * s );
        }

        return d_points;
    }

    void initSamples()
    {
        const int numSamples = 15;

        for ( int i = 0; i < numSamples; i++ )
        {
            const double angle = i * ( 2.0 * M_PI / ( numSamples - 1 ) );

            QPointF p( qCos( angle ), qSin( angle ) );
            if ( i % 2 )
                p *= 0.4;
            
            d_points += p;
        }
    }

private:
    mutable QPolygonF d_points;
};  

Plot::Plot( QWidget *parent ):
    QwtPlot( parent)
{
    setAutoReplot( false );

    setTitle( "Animated Curves" );

    // hide all axes
    for ( int axis = 0; axis < QwtPlot::axisCnt; axis++ )
        enableAxis( axis, false );

    plotLayout()->setCanvasMargin( 10 );

    d_curves[0] = new Curve1();
    d_curves[1] = new Curve2();
    d_curves[2] = new Curve3();
    d_curves[3] = new Curve4();

    updateCurves();

    for ( int i = 0; i < CurveCount; i++ )
        d_curves[i]->attach( this );

    d_time.start();
    ( void )startTimer( 40 );
}

void Plot::timerEvent( QTimerEvent * )
{
    updateCurves();
    replot();
}

void Plot::updateCurves()
{
    const double speed = 2 * M_PI / 25000.0; // a cycle every 25 seconds

    const double phase = d_time.elapsed() * speed;
    for ( int i = 0; i < CurveCount; i++ )
        d_curves[i]->updateSamples( phase );
}
```

### `examples/animation/plot.h`

```cpp
#include <qwt_plot.h>
#include <qdatetime.h>

class Curve;

class Plot: public QwtPlot
{
public:
    Plot( QWidget * = NULL);

protected:
    virtual void timerEvent( QTimerEvent * );

private:
    void updateCurves();

    enum { CurveCount = 4 };
    Curve *d_curves[CurveCount];

    QTime d_time;
};
```

### `examples/barchart/barchart.cpp`

```cpp
#include "barchart.h"
#include <qwt_plot_renderer.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_multi_barchart.h>
#include <qwt_column_symbol.h>
#include <qwt_plot_layout.h>
#include <qwt_legend.h>
#include <qwt_scale_draw.h>

BarChart::BarChart( QWidget *parent ):
    QwtPlot( parent )
{
    setAutoFillBackground( true );

    setPalette( Qt::white );
    canvas()->setPalette( QColor( "LemonChiffon" ) );

    setTitle( "Bar Chart" );

    setAxisTitle( QwtPlot::yLeft, "Whatever" );
    setAxisTitle( QwtPlot::xBottom, "Whatever" );

    d_barChartItem = new QwtPlotMultiBarChart( "Bar Chart " );
    d_barChartItem->setLayoutPolicy( QwtPlotMultiBarChart::AutoAdjustSamples );
    d_barChartItem->setSpacing( 20 );
    d_barChartItem->setMargin( 3 );

    d_barChartItem->attach( this );

    insertLegend( new QwtLegend() );

    populate();
    setOrientation( 0 );

    setAutoReplot( true );
}

void BarChart::populate()
{
    static const char *colors[] = { "DarkOrchid", "SteelBlue", "Gold" };

    const int numSamples = 5;
    const int numBars = sizeof( colors ) / sizeof( colors[0] );

    QList<QwtText> titles;
    for ( int i = 0; i < numBars; i++ )
    {
        QString title("Bar %1");
        titles += title.arg( i );
    }
    d_barChartItem->setBarTitles( titles );
    d_barChartItem->setLegendIconSize( QSize( 10, 14 ) );

    for ( int i = 0; i < numBars; i++ )
    {
        QwtColumnSymbol *symbol = new QwtColumnSymbol( QwtColumnSymbol::Box );
        symbol->setLineWidth( 2 );
        symbol->setFrameStyle( QwtColumnSymbol::Raised );
        symbol->setPalette( QPalette( colors[i] ) );

        d_barChartItem->setSymbol( i, symbol );
    }
    
    QVector< QVector<double> > series;
    for ( int i = 0; i < numSamples; i++ )
    {
        QVector<double> values;
        for ( int j = 0; j < numBars; j++ )
            values += ( 2 + qrand() % 8 );

        series += values;
    }

    d_barChartItem->setSamples( series );
}

void BarChart::setMode( int mode )
{
    if ( mode == 0 )
    {
        d_barChartItem->setStyle( QwtPlotMultiBarChart::Grouped );
    }
    else
    {
        d_barChartItem->setStyle( QwtPlotMultiBarChart::Stacked );
    }
}

void BarChart::setOrientation( int orientation )
{
    QwtPlot::Axis axis1, axis2;

    if ( orientation == 0 )
    {
        axis1 = QwtPlot::xBottom;
        axis2 = QwtPlot::yLeft;

        d_barChartItem->setOrientation( Qt::Vertical );
    }
    else
    {
        axis1 = QwtPlot::yLeft;
        axis2 = QwtPlot::xBottom;

        d_barChartItem->setOrientation( Qt::Horizontal );
    }

    setAxisScale( axis1, 0, d_barChartItem->dataSize() - 1, 1.0 );
    setAxisAutoScale( axis2 );

    QwtScaleDraw *scaleDraw1 = axisScaleDraw( axis1 );
    scaleDraw1->enableComponent( QwtScaleDraw::Backbone, false );
    scaleDraw1->enableComponent( QwtScaleDraw::Ticks, false );

    QwtScaleDraw *scaleDraw2 = axisScaleDraw( axis2 );
    scaleDraw2->enableComponent( QwtScaleDraw::Backbone, true );
    scaleDraw2->enableComponent( QwtScaleDraw::Ticks, true );

    plotLayout()->setAlignCanvasToScale( axis1, true );
    plotLayout()->setAlignCanvasToScale( axis2, false );

    plotLayout()->setCanvasMargin( 0 );
    updateCanvasMargins();

    replot();
}

void BarChart::exportChart()
{
    QwtPlotRenderer renderer;
    renderer.exportTo( this, "barchart.pdf" );
}
```

### `examples/barchart/barchart.h`

```cpp
#ifndef _BAR_CHART_H_

#include <qwt_plot.h>

class QwtPlotMultiBarChart;

class BarChart: public QwtPlot
{
    Q_OBJECT

public:
    BarChart( QWidget * = NULL );

public Q_SLOTS:
    void setMode( int );
    void setOrientation( int );
    void exportChart();

private:
    void populate();

    QwtPlotMultiBarChart *d_barChartItem;
};

#endif
```

### `examples/barchart/main.cpp`

```cpp
#include <qapplication.h>
#include <qmainwindow.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include <qcombobox.h>
#include "barchart.h"

class MainWindow: public QMainWindow
{
public:
    MainWindow( QWidget * = NULL );

private:
    BarChart *d_chart;
};

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    d_chart = new BarChart( this );
    setCentralWidget( d_chart );

    QToolBar *toolBar = new QToolBar( this );

    QComboBox *typeBox = new QComboBox( toolBar );
    typeBox->addItem( "Grouped" );
    typeBox->addItem( "Stacked" );
    typeBox->setSizePolicy( QSizePolicy::Fixed, QSizePolicy::Fixed );

    QComboBox *orientationBox = new QComboBox( toolBar );
    orientationBox->addItem( "Vertical" );
    orientationBox->addItem( "Horizontal" );
    orientationBox->setSizePolicy( QSizePolicy::Fixed, QSizePolicy::Fixed );

    QToolButton *btnExport = new QToolButton( toolBar );
    btnExport->setText( "Export" );
    btnExport->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    connect( btnExport, SIGNAL( clicked() ), d_chart, SLOT( exportChart() ) );

    toolBar->addWidget( typeBox );
    toolBar->addWidget( orientationBox );
    toolBar->addWidget( btnExport );
    addToolBar( toolBar );

    d_chart->setMode( typeBox->currentIndex() );
    connect( typeBox, SIGNAL( currentIndexChanged( int ) ),
             d_chart, SLOT( setMode( int ) ) );

    d_chart->setOrientation( orientationBox->currentIndex() );
    connect( orientationBox, SIGNAL( currentIndexChanged( int ) ),
             d_chart, SLOT( setOrientation( int ) ) );
}

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWindow mainWindow;

    mainWindow.resize( 600, 400 );
    mainWindow.show();

    return a.exec();
}
```

### `examples/bode/complexnumber.h`

```cpp
#ifndef _COMPLEX_NUMBER_H_
#define _COMPLEX_NUMBER_H_

#include <math.h>

class ComplexNumber
{
public:
    ComplexNumber() ;
    ComplexNumber( double r, double i = 0.0 );

    double real() const;
    double imag() const;

    friend ComplexNumber operator*(
        const ComplexNumber &, const ComplexNumber & );

    friend ComplexNumber operator+(
        const ComplexNumber &, const ComplexNumber & );

    friend ComplexNumber operator-(
        const ComplexNumber &, const ComplexNumber & );
    friend ComplexNumber operator/(
        const ComplexNumber &, const ComplexNumber & );

private:
    double d_real;
    double d_imag;
};

inline ComplexNumber::ComplexNumber():
    d_real( 0.0 ),
    d_imag( -0.0 )
{
}

inline ComplexNumber::ComplexNumber( double re, double im ):
    d_real( re ),
    d_imag( im )
{
}

inline double ComplexNumber::real() const
{
    return d_real;
}

inline double ComplexNumber::imag() const
{
    return d_imag;
}

inline ComplexNumber operator+(
    const ComplexNumber &x1, const ComplexNumber &x2 )
{
    return ComplexNumber( x1.d_real + x2.d_real, x1.d_imag + x2.d_imag );
}

inline ComplexNumber operator-(
    const ComplexNumber &x1, const ComplexNumber &x2 )
{
    return ComplexNumber( x1.d_real - x2.d_real, x1.d_imag - x2.d_imag );
}

inline ComplexNumber operator*(
    const ComplexNumber &x1, const ComplexNumber &x2 )
{
    return ComplexNumber( x1.d_real * x2.d_real - x1.d_imag * x2.d_imag,
        x1.d_real * x2.d_imag + x2.d_real * x1.d_imag );
}

inline ComplexNumber operator/(
    const ComplexNumber &x1, const ComplexNumber &x2 )
{
    double denom = x2.d_real * x2.d_real + x2.d_imag * x2.d_imag;

    return ComplexNumber(
               ( x1.d_real * x2.d_real + x1.d_imag * x2.d_imag ) / denom,
               ( x1.d_imag * x2.d_real - x2.d_imag * x1.d_real ) / denom
           );
}

#endif
```

### `examples/bode/main.cpp`

```cpp
#include <qapplication.h>
#include "mainwindow.h"

int main ( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWindow w;
    w.resize( 540, 400 );
    w.show();

    return a.exec();
}
```

### `examples/bode/mainwindow.cpp`

```cpp
#include <qregexp.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include <qlabel.h>
#include <qlayout.h>
#include <qstatusbar.h>
#include <qprinter.h>
#include <qpicture.h>
#include <qpainter.h>
#include <qprintdialog.h>
#include <qwt_counter.h>
#include <qwt_picker_machine.h>
#include <qwt_plot_zoomer.h>
#include <qwt_plot_panner.h>
#include <qwt_plot_renderer.h>
#include <qwt_text.h>
#include <qwt_math.h>
#include "pixmaps.h"
#include "plot.h"
#include "mainwindow.h"

class Zoomer: public QwtPlotZoomer
{
public:
    Zoomer( int xAxis, int yAxis, QWidget *canvas ):
        QwtPlotZoomer( xAxis, yAxis, canvas )
    {
        setTrackerMode( QwtPicker::AlwaysOff );
        setRubberBand( QwtPicker::NoRubberBand );

        // RightButton: zoom out by 1
        // Ctrl+RightButton: zoom out to full size

        setMousePattern( QwtEventPattern::MouseSelect2,
            Qt::RightButton, Qt::ControlModifier );
        setMousePattern( QwtEventPattern::MouseSelect3,
            Qt::RightButton );
    }
};

//-----------------------------------------------------------------
//
//      bode.cpp -- A demo program featuring QwtPlot and QwtCounter
//
//      This example demonstrates the mapping of different curves
//      to different axes in a QwtPlot widget. It also shows how to
//      display the cursor position and how to implement zooming.
//
//-----------------------------------------------------------------

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    d_plot = new Plot( this );

    const int margin = 5;
    d_plot->setContentsMargins( margin, margin, margin, 0 );

    setContextMenuPolicy( Qt::NoContextMenu );

    d_zoomer[0] = new Zoomer( QwtPlot::xBottom, QwtPlot::yLeft,
        d_plot->canvas() );
    d_zoomer[0]->setRubberBand( QwtPicker::RectRubberBand );
    d_zoomer[0]->setRubberBandPen( QColor( Qt::green ) );
    d_zoomer[0]->setTrackerMode( QwtPicker::ActiveOnly );
    d_zoomer[0]->setTrackerPen( QColor( Qt::white ) );

    d_zoomer[1] = new Zoomer( QwtPlot::xTop, QwtPlot::yRight,
         d_plot->canvas() );

    d_panner = new QwtPlotPanner( d_plot->canvas() );
    d_panner->setMouseButton( Qt::MidButton );

    d_picker = new QwtPlotPicker( QwtPlot::xBottom, QwtPlot::yLeft,
        QwtPlotPicker::CrossRubberBand, QwtPicker::AlwaysOn,
        d_plot->canvas() );
    d_picker->setStateMachine( new QwtPickerDragPointMachine() );
    d_picker->setRubberBandPen( QColor( Qt::green ) );
    d_picker->setRubberBand( QwtPicker::CrossRubberBand );
    d_picker->setTrackerPen( QColor( Qt::white ) );

    setCentralWidget( d_plot );

    QToolBar *toolBar = new QToolBar( this );

    QToolButton *btnZoom = new QToolButton( toolBar );
    btnZoom->setText( "Zoom" );
    btnZoom->setIcon( QPixmap( zoom_xpm ) );
    btnZoom->setCheckable( true );
    btnZoom->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    toolBar->addWidget( btnZoom );
    connect( btnZoom, SIGNAL( toggled( bool ) ), SLOT( enableZoomMode( bool ) ) );

#ifndef QT_NO_PRINTER
    QToolButton *btnPrint = new QToolButton( toolBar );
    btnPrint->setText( "Print" );
    btnPrint->setIcon( QPixmap( print_xpm ) );
    btnPrint->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    toolBar->addWidget( btnPrint );
    connect( btnPrint, SIGNAL( clicked() ), SLOT( print() ) );
#endif

    QToolButton *btnExport = new QToolButton( toolBar );
    btnExport->setText( "Export" );
    btnExport->setIcon( QPixmap( print_xpm ) );
    btnExport->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    toolBar->addWidget( btnExport );
    connect( btnExport, SIGNAL( clicked() ), SLOT( exportDocument() ) );

    toolBar->addSeparator();

    QWidget *hBox = new QWidget( toolBar );

    QHBoxLayout *layout = new QHBoxLayout( hBox );
    layout->setSpacing( 0 );
    layout->addWidget( new QWidget( hBox ), 10 ); // spacer
    layout->addWidget( new QLabel( "Damping Factor", hBox ), 0 );
    layout->addSpacing( 10 );

    QwtCounter *cntDamp = new QwtCounter( hBox );
    cntDamp->setRange( 0.0, 5.0 );
    cntDamp->setSingleStep( 0.01 );
    cntDamp->setValue( 0.0 );

    layout->addWidget( cntDamp, 0 );

    ( void )toolBar->addWidget( hBox );

    addToolBar( toolBar );
#ifndef QT_NO_STATUSBAR
    ( void )statusBar();
#endif

    enableZoomMode( false );
    showInfo();

    connect( cntDamp, SIGNAL( valueChanged( double ) ),
        d_plot, SLOT( setDamp( double ) ) );

    connect( d_picker, SIGNAL( moved( const QPoint & ) ),
        SLOT( moved( const QPoint & ) ) );
    connect( d_picker, SIGNAL( selected( const QPolygon & ) ),
        SLOT( selected( const QPolygon & ) ) );
}

#ifndef QT_NO_PRINTER

void MainWindow::print()
{
    QPrinter printer( QPrinter::HighResolution );

    QString docName = d_plot->title().text();
    if ( !docName.isEmpty() )
    {
        docName.replace ( QRegExp ( QString::fromLatin1 ( "\n" ) ), tr ( " -- " ) );
        printer.setDocName ( docName );
    }

    printer.setCreator( "Bode example" );
    printer.setOrientation( QPrinter::Landscape );

    QPrintDialog dialog( &printer );
    if ( dialog.exec() )
    {
        QwtPlotRenderer renderer;

        if ( printer.colorMode() == QPrinter::GrayScale )
        {
            renderer.setDiscardFlag( QwtPlotRenderer::DiscardBackground );
            renderer.setDiscardFlag( QwtPlotRenderer::DiscardCanvasBackground );
            renderer.setDiscardFlag( QwtPlotRenderer::DiscardCanvasFrame );
            renderer.setLayoutFlag( QwtPlotRenderer::FrameWithScales );
        }

        renderer.renderTo( d_plot, printer );
    }
}

#endif

void MainWindow::exportDocument()
{
    QwtPlotRenderer renderer;
    renderer.exportTo( d_plot, "bode.pdf" );
}

void MainWindow::enableZoomMode( bool on )
{
    d_panner->setEnabled( on );

    d_zoomer[0]->setEnabled( on );
    d_zoomer[0]->zoom( 0 );

    d_zoomer[1]->setEnabled( on );
    d_zoomer[1]->zoom( 0 );

    d_picker->setEnabled( !on );

    showInfo();
}

void MainWindow::showInfo( QString text )
{
    if ( text == QString::null )
    {
        if ( d_picker->rubberBand() )
            text = "Cursor Pos: Press left mouse button in plot region";
        else
            text = "Zoom: Press mouse button and drag";
    }

#ifndef QT_NO_STATUSBAR
    statusBar()->showMessage( text );
#endif
}

void MainWindow::moved( const QPoint &pos )
{
    QString info;
    info.sprintf( "Freq=%g, Ampl=%g, Phase=%g",
        d_plot->invTransform( QwtPlot::xBottom, pos.x() ),
        d_plot->invTransform( QwtPlot::yLeft, pos.y() ),
        d_plot->invTransform( QwtPlot::yRight, pos.y() )
    );
    showInfo( info );
}

void MainWindow::selected( const QPolygon & )
{
    showInfo();
}
```

### `examples/bode/mainwindow.h`

```cpp
#include <qmainwindow.h>

class QwtPlotZoomer;
class QwtPlotPicker;
class QwtPlotPanner;
class Plot;
class QPolygon;

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow( QWidget *parent = 0 );

private Q_SLOTS:
    void moved( const QPoint & );
    void selected( const QPolygon & );

#ifndef QT_NO_PRINTER
    void print();
#endif

    void exportDocument();
    void enableZoomMode( bool );

private:
    void showInfo( QString text = QString::null );

    Plot *d_plot;

    QwtPlotZoomer *d_zoomer[2];
    QwtPlotPicker *d_picker;
    QwtPlotPanner *d_panner;
};
```

### `examples/bode/pixmaps.h`

```cpp
#ifndef PIXMAPS_H
#define PIXMAPS_H

static const char *print_xpm[] =
{
    "32 32 12 1",
    "a c #ffffff",
    "h c #ffff00",
    "c c #ffffff",
    "f c #dcdcdc",
    "b c #c0c0c0",
    "j c #a0a0a4",
    "e c #808080",
    "g c #808000",
    "d c #585858",
    "i c #00ff00",
    "# c #000000",
    ". c None",
    "................................",
    "................................",
    "...........###..................",
    "..........#abb###...............",
    ".........#aabbbbb###............",
    ".........#ddaaabbbbb###.........",
    "........#ddddddaaabbbbb###......",
    ".......#deffddddddaaabbbbb###...",
    "......#deaaabbbddddddaaabbbbb###",
    ".....#deaaaaaaabbbddddddaaabbbb#",
    "....#deaaabbbaaaa#ddedddfggaaad#",
    "...#deaaaaaaaaaa#ddeeeeafgggfdd#",
    "..#deaaabbbaaaa#ddeeeeabbbbgfdd#",
    ".#deeefaaaaaaa#ddeeeeabbhhbbadd#",
    "#aabbbeeefaaa#ddeeeeabbbbbbaddd#",
    "#bbaaabbbeee#ddeeeeabbiibbadddd#",
    "#bbbbbaaabbbeeeeeeabbbbbbaddddd#",
    "#bjbbbbbbaaabbbbeabbbbbbadddddd#",
    "#bjjjjbbbbbbaaaeabbbbbbaddddddd#",
    "#bjaaajjjbbbbbbaaabbbbadddddddd#",
    "#bbbbbaaajjjbbbbbbaaaaddddddddd#",
    "#bjbbbbbbaaajjjbbbbbbddddddddd#.",
    "#bjjjjbbbbbbaaajjjbbbdddddddd#..",
    "#bjaaajjjbbbbbbjaajjbddddddd#...",
    "#bbbbbaaajjjbbbjbbaabdddddd#....",
    "###bbbbbbaaajjjjbbbbbddddd#.....",
    "...###bbbbbbaaajbbbbbdddd#......",
    "......###bbbbbbjbbbbbddd#.......",
    ".........###bbbbbbbbbdd#........",
    "............###bbbbbbd#.........",
    "...............###bbb#..........",
    "..................###..........."
};


static const char *zoom_xpm[] =
{
    "32 32 8 1",
    "# c #000000",
    "b c #c0c0c0",
    "a c #ffffff",
    "e c #585858",
    "d c #a0a0a4",
    "c c #0000ff",
    "f c #00ffff",
    ". c None",
    "..######################........",
    ".#a#baaaaaaaaaaaaaaaaaa#........",
    "#aa#baaaaaaaaaaaaaccaca#........",
    "####baaaaaaaaaaaaaaaaca####.....",
    "#bbbbaaaaaaaaaaaacccaaa#da#.....",
    "#aaaaaaaaaaaaaaaacccaca#da#.....",
    "#aaaaaaaaaaaaaaaaaccaca#da#.....",
    "#aaaaaaaaaabe###ebaaaaa#da#.....",
    "#aaaaaaaaa#########aaaa#da#.....",
    "#aaaaaaaa###dbbbb###aaa#da#.....",
    "#aaaaaaa###aaaaffb###aa#da#.....",
    "#aaaaaab##aaccaaafb##ba#da#.....",
    "#aaaaaae#daaccaccaad#ea#da#.....",
    "#aaaaaa##aaaaaaccaab##a#da#.....",
    "#aaaaaa##aacccaaaaab##a#da#.....",
    "#aaaaaa##aaccccaccab##a#da#.....",
    "#aaaaaae#daccccaccad#ea#da#.....",
    "#aaaaaab##aacccaaaa##da#da#.....",
    "#aaccacd###aaaaaaa###da#da#.....",
    "#aaaaacad###daaad#####a#da#.....",
    "#acccaaaad##########da##da#.....",
    "#acccacaaadde###edd#eda#da#.....",
    "#aaccacaaaabdddddbdd#eda#a#.....",
    "#aaaaaaaaaaaaaaaaaadd#eda##.....",
    "#aaaaaaaaaaaaaaaaaaadd#eda#.....",
    "#aaaaaaaccacaaaaaaaaadd#eda#....",
    "#aaaaaaaaaacaaaaaaaaaad##eda#...",
    "#aaaaaacccaaaaaaaaaaaaa#d#eda#..",
    "########################dd#eda#.",
    "...#dddddddddddddddddddddd##eda#",
    "...#aaaaaaaaaaaaaaaaaaaaaa#.####",
    "...########################..##."
};

#endif
```

### `examples/bode/plot.cpp`

```cpp
#include <qwt_math.h>
#include <qwt_scale_engine.h>
#include <qwt_symbol.h>
#include <qwt_plot_grid.h>
#include <qwt_plot_marker.h>
#include <qwt_plot_curve.h>
#include <qwt_legend.h>
#include <qwt_text.h>
#include <qwt_plot_canvas.h>
#include <qmath.h>
#include "complexnumber.h"
#include "plot.h"

#if QT_VERSION < 0x040601
#define qExp(x) ::exp(x)
#define qAtan2(y, x) ::atan2(y, x)
#endif

static void logSpace( double *array, int size, double xmin, double xmax )
{
    if ( ( xmin <= 0.0 ) || ( xmax <= 0.0 ) || ( size <= 0 ) )
        return;

    const int imax = size - 1;

    array[0] = xmin;
    array[imax] = xmax;

    const double lxmin = log( xmin );
    const double lxmax = log( xmax );
    const double lstep = ( lxmax - lxmin ) / double( imax );

    for ( int i = 1; i < imax; i++ )
        array[i] = qExp( lxmin + double( i ) * lstep );
}

Plot::Plot( QWidget *parent ):
    QwtPlot( parent )
{
    setAutoReplot( false );

    setTitle( "Frequency Response of a Second-Order System" );

    QwtPlotCanvas *canvas = new QwtPlotCanvas();
    canvas->setBorderRadius( 10 );

    setCanvas( canvas );
    setCanvasBackground( QColor( "MidnightBlue" ) );

    // legend
    QwtLegend *legend = new QwtLegend;
    insertLegend( legend, QwtPlot::BottomLegend );

    // grid
    QwtPlotGrid *grid = new QwtPlotGrid;
    grid->enableXMin( true );
    grid->setMajorPen( Qt::white, 0, Qt::DotLine );
    grid->setMinorPen( Qt::gray, 0 , Qt::DotLine );
    grid->attach( this );

    // axes
    enableAxis( QwtPlot::yRight );
    setAxisTitle( QwtPlot::xBottom, "Normalized Frequency" );
    setAxisTitle( QwtPlot::yLeft, "Amplitude [dB]" );
    setAxisTitle( QwtPlot::yRight, "Phase [deg]" );

    setAxisMaxMajor( QwtPlot::xBottom, 6 );
    setAxisMaxMinor( QwtPlot::xBottom, 9 );
    setAxisScaleEngine( QwtPlot::xBottom, new QwtLogScaleEngine );

    // curves
    d_curve1 = new QwtPlotCurve( "Amplitude" );
    d_curve1->setRenderHint( QwtPlotItem::RenderAntialiased );
    d_curve1->setPen( Qt::yellow );
    d_curve1->setLegendAttribute( QwtPlotCurve::LegendShowLine );
    d_curve1->setYAxis( QwtPlot::yLeft );
    d_curve1->attach( this );

    d_curve2 = new QwtPlotCurve( "Phase" );
    d_curve2->setRenderHint( QwtPlotItem::RenderAntialiased );
    d_curve2->setPen( Qt::cyan );
    d_curve2->setLegendAttribute( QwtPlotCurve::LegendShowLine );
    d_curve2->setYAxis( QwtPlot::yRight );
    d_curve2->attach( this );

    // marker
    d_marker1 = new QwtPlotMarker();
    d_marker1->setValue( 0.0, 0.0 );
    d_marker1->setLineStyle( QwtPlotMarker::VLine );
    d_marker1->setLabelAlignment( Qt::AlignRight | Qt::AlignBottom );
    d_marker1->setLinePen( Qt::green, 0, Qt::DashDotLine );
    d_marker1->attach( this );

    d_marker2 = new QwtPlotMarker();
    d_marker2->setLineStyle( QwtPlotMarker::HLine );
    d_marker2->setLabelAlignment( Qt::AlignRight | Qt::AlignBottom );
    d_marker2->setLinePen( QColor( 200, 150, 0 ), 0, Qt::DashDotLine );
    d_marker2->setSymbol( new QwtSymbol( QwtSymbol::Diamond,
        QColor( Qt::yellow ), QColor( Qt::green ), QSize( 8, 8 ) ) );
    d_marker2->attach( this );

    setDamp( 0.0 );

    setAutoReplot( true );
}

void Plot::showData( const double *frequency, const double *amplitude,
    const double *phase, int count )
{
    d_curve1->setSamples( frequency, amplitude, count );
    d_curve2->setSamples( frequency, phase, count );
}

void Plot::showPeak( double freq, double amplitude )
{
    QString label;
    label.sprintf( "Peak: %.3g dB", amplitude );

    QwtText text( label );
    text.setFont( QFont( "Helvetica", 10, QFont::Bold ) );
    text.setColor( QColor( 200, 150, 0 ) );

    d_marker2->setValue( freq, amplitude );
    d_marker2->setLabel( text );
}

void Plot::show3dB( double freq )
{
    QString label;
    label.sprintf( "-3 dB at f = %.3g", freq );

    QwtText text( label );
    text.setFont( QFont( "Helvetica", 10, QFont::Bold ) );
    text.setColor( Qt::green );

    d_marker1->setValue( freq, 0.0 );
    d_marker1->setLabel( text );
}

//
// re-calculate frequency response
//
void Plot::setDamp( double damping )
{
    const bool doReplot = autoReplot();
    setAutoReplot( false );

    const int ArraySize = 200;

    double frequency[ArraySize];
    double amplitude[ArraySize];
    double phase[ArraySize];

    // build frequency vector with logarithmic division
    logSpace( frequency, ArraySize, 0.01, 100 );

    int i3 = 1;
    double fmax = 1;
    double amax = -1000.0;

    for ( int i = 0; i < ArraySize; i++ )
    {
        double f = frequency[i];
        const ComplexNumber g =
            ComplexNumber( 1.0 ) / ComplexNumber( 1.0 - f * f, 2.0 * damping * f );

        amplitude[i] = 20.0 * log10( qSqrt( g.real() * g.real() + g.imag() * g.imag() ) );
        phase[i] = qAtan2( g.imag(), g.real() ) * ( 180.0 / M_PI );

        if ( ( i3 <= 1 ) && ( amplitude[i] < -3.0 ) )
            i3 = i;
        if ( amplitude[i] > amax )
        {
            amax = amplitude[i];
            fmax = frequency[i];
        }

    }

    double f3 = frequency[i3] - ( frequency[i3] - frequency[i3 - 1] )
        / ( amplitude[i3] - amplitude[i3 -1] ) * ( amplitude[i3] + 3 );

    showPeak( fmax, amax );
    show3dB( f3 );
    showData( frequency, amplitude, phase, ArraySize );

    setAutoReplot( doReplot );

    replot();
}
```

### `examples/bode/plot.h`

```cpp
#ifndef _PLOT_H_
#define _PLOT_H_

#include <qwt_plot.h>

class QwtPlotCurve;
class QwtPlotMarker;

class Plot: public QwtPlot
{
    Q_OBJECT

public:
    Plot( QWidget *parent );

public Q_SLOTS:
    void setDamp( double damping );

private:
    void showData( const double *frequency, const double *amplitude,
        const double *phase, int count );
    void showPeak( double freq, double amplitude );
    void show3dB( double freq );

    QwtPlotCurve *d_curve1;
    QwtPlotCurve *d_curve2;
    QwtPlotMarker *d_marker1;
    QwtPlotMarker *d_marker2;
};

#endif
```

### `examples/controls/dialbox.cpp`

```cpp
#include <qlabel.h>
#include <qlayout.h>
#include <qwt_dial.h>
#include <qwt_dial_needle.h>
#include <qwt_scale_engine.h>
#include <qwt_transform.h>
#include <qwt_round_scale_draw.h>
#include "dialbox.h"

DialBox::DialBox( QWidget *parent, int type ):
    QWidget( parent )
{
    d_dial = createDial( type );

    d_label = new QLabel( this );
    d_label->setAlignment( Qt::AlignCenter );

    QVBoxLayout *layout = new QVBoxLayout( this );;
    layout->setSpacing( 0 );
    layout->addWidget( d_dial, 10 );
    layout->addWidget( d_label );

    connect( d_dial, SIGNAL( valueChanged( double ) ), 
        this, SLOT( setNum( double ) ) );

    setNum( d_dial->value() );
}

QwtDial *DialBox::createDial( int type ) const
{
    QwtDial *dial = new QwtDial();
    dial->setTracking( true );
    dial->setFocusPolicy( Qt::StrongFocus );
    dial->setObjectName( QString( "Dial %1" ).arg( type + 1 ) );

    QColor needleColor( Qt::red );

    switch( type )
    {
        case 0:
        {
            dial->setOrigin( 135.0 );
            dial->setScaleArc( 0.0, 270.0 );
            dial->setScaleMaxMinor( 4 );
            dial->setScaleMaxMajor( 10 );
            dial->setScale( -100.0, 100.0 );

            needleColor = QColor( "Goldenrod" );

            break;
        }
        case 1:
        {
            dial->setOrigin( 135.0 );
            dial->setScaleArc( 0.0, 270.0 );
            dial->setScaleMaxMinor( 10 );
            dial->setScaleMaxMajor( 10 );
            dial->setScale( 10.0, 0.0 );

            QwtRoundScaleDraw *scaleDraw = new QwtRoundScaleDraw();
            scaleDraw->setSpacing( 8 );
            scaleDraw->enableComponent( 
                QwtAbstractScaleDraw::Backbone, false );
            scaleDraw->setTickLength( QwtScaleDiv::MinorTick, 2 );
            scaleDraw->setTickLength( QwtScaleDiv::MediumTick, 4 );
            scaleDraw->setTickLength( QwtScaleDiv::MajorTick, 8 );
            dial->setScaleDraw( scaleDraw );

            break;
        }
        case 2:
        {
            dial->setOrigin( 150.0 );
            dial->setScaleArc( 0.0, 240.0 );

            QwtLinearScaleEngine *scaleEngine = new QwtLinearScaleEngine( 2 );
            scaleEngine->setTransformation( new QwtPowerTransform( 2 ) );
            dial->setScaleEngine( scaleEngine );

            QList< double > ticks[ QwtScaleDiv::NTickTypes ];
            ticks[ QwtScaleDiv::MajorTick ] << 0 << 4 
                << 16 << 32 << 64 << 96 << 128;
            ticks[ QwtScaleDiv::MediumTick ] << 24 << 48 << 80 << 112;
            ticks[ QwtScaleDiv::MinorTick ] 
                << 0.5 << 1 << 2 
                << 7 << 10 << 13
                << 20 << 28 
                << 40 << 56 
                << 72 << 88 
                << 104 << 120; 
 
            dial->setScale( QwtScaleDiv( 0, 128, ticks ) );
            break;
        }
        case 3:
        {
            dial->setOrigin( 135.0 );
            dial->setScaleArc( 0.0, 270.0 );
            dial->setScaleMaxMinor( 9 );
            dial->setScaleEngine( new QwtLogScaleEngine );
            dial->setScale( 1.0e-2, 1.0e2 );

            break;
        }
        case 4:
        {
            dial->setOrigin( 225.0 );
            dial->setScaleArc( 0.0, 360.0 );
            dial->setScaleMaxMinor( 5 );
            dial->setScaleStepSize( 20 );
            dial->setScale( 100.0, -100.0 );
            dial->setWrapping( true );
            dial->setTotalSteps( 40 );
            dial->setMode( QwtDial::RotateScale );
            dial->setValue( 70.0 );

            needleColor = QColor( "DarkSlateBlue" );

            break;
        }
        case 5:
        {
            dial->setOrigin( 45.0 );
            dial->setScaleArc( 0.0, 225.0 );
            dial->setScaleMaxMinor( 5 );
            dial->setScaleMaxMajor( 10 );
            dial->setScale( 0.0, 10.0 );

            break;
        }
    }

    QwtDialSimpleNeedle *needle = new QwtDialSimpleNeedle(
        QwtDialSimpleNeedle::Arrow, true, needleColor,
        QColor( Qt::gray ).light( 130 ) );
    dial->setNeedle( needle );

    //const QColor base( QColor( "DimGray" ) );
    const QColor base( QColor( Qt::darkGray ).dark( 150 ) );

    QPalette palette;
    palette.setColor( QPalette::Base, base );
    palette.setColor( QPalette::Window, base.dark( 150 ) );
    palette.setColor( QPalette::Mid, base.dark( 110 ) );
    palette.setColor( QPalette::Light, base.light( 170 ) );
    palette.setColor( QPalette::Dark, base.dark( 170 ) );
    palette.setColor( QPalette::Text, base.dark( 200 ).light( 800 ) );
    palette.setColor( QPalette::WindowText, base.dark( 200 ) );

    dial->setPalette( palette );
    dial->setLineWidth( 4 );
    dial->setFrameShadow( QwtDial::Sunken );

    return dial;
}

void DialBox::setNum( double v )
{
    QString text;
    text.setNum( v, 'f', 2 );

    d_label->setText( text );
}
```

### `examples/controls/dialbox.h`

```cpp
#ifndef _DIAL_BOX_H_
#define _DIAL_BOX_H_

#include <qwidget.h>

class QLabel;
class QwtDial;

class DialBox: public QWidget
{
    Q_OBJECT
public:
    DialBox( QWidget *parent, int type );

private Q_SLOTS:
    void setNum( double v );

private:
    QwtDial *createDial( int type ) const;

    QwtDial *d_dial;
    QLabel *d_label;
};

#endif
```

### `examples/controls/dialtab.cpp`

```cpp
#include "dialtab.h"
#include "dialbox.h"
#include <qlayout.h>

DialTab::DialTab( QWidget *parent ):
    QWidget( parent )
{
    QGridLayout *layout = new QGridLayout( this );

    const int numRows = 3;
    for ( int i = 0; i < 2 * numRows; i++ )
    {
        DialBox *dialBox = new DialBox( this, i );
        layout->addWidget( dialBox, i / numRows, i % numRows );
    }
}
```

### `examples/controls/dialtab.h`

```cpp
#ifndef _DIAL_TAB_H
#define _DIAL_TAB_H 1

#include <qwidget.h>

class DialTab: public QWidget
{
public:
    DialTab( QWidget *parent = NULL );
};

#endif
```

### `examples/controls/knobbox.cpp`

```cpp
#include <qlabel.h>
#include <qlayout.h>
#include <qwt_knob.h>
#include <qwt_scale_engine.h>
#include <qwt_transform.h>
#include "knobbox.h"

KnobBox::KnobBox( QWidget *parent, int knobType ):
    QWidget( parent )
{
    d_knob = createKnob( knobType );
    d_knob->setKnobWidth( 100 );

    d_label = new QLabel( this );
    d_label->setAlignment( Qt::AlignCenter );

    QVBoxLayout *layout = new QVBoxLayout( this );;
    layout->setSpacing( 0 );
    layout->addWidget( d_knob, 10 );
    layout->addWidget( d_label );
    layout->addStretch( 10 );

    connect( d_knob, SIGNAL( valueChanged( double ) ), 
        this, SLOT( setNum( double ) ) );

    setNum( d_knob->value() );
}

QwtKnob *KnobBox::createKnob( int knobType ) const
{
    QwtKnob *knob = new QwtKnob();
    knob->setTracking( true );

    switch( knobType )
    {
        case 0:
        {
            knob->setKnobStyle( QwtKnob::Sunken );
            knob->setMarkerStyle( QwtKnob::Nub );
            knob->setWrapping( true );
            knob->setNumTurns( 4 );
            knob->setScaleStepSize( 10.0 );
            knob->setScale( 0, 400 );
            knob->setTotalSteps( 400 );
            break;
        }
        case 1:
        {
            knob->setKnobStyle( QwtKnob::Sunken );
            knob->setMarkerStyle( QwtKnob::Dot );
            break;
        }
        case 2:
        {
            knob->setKnobStyle( QwtKnob::Sunken );
            knob->setMarkerStyle( QwtKnob::Tick );
            
            QwtLinearScaleEngine *scaleEngine = new QwtLinearScaleEngine( 2 );
            scaleEngine->setTransformation( new QwtPowerTransform( 2 ) );
            knob->setScaleEngine( scaleEngine );

            QList< double > ticks[ QwtScaleDiv::NTickTypes ];
            ticks[ QwtScaleDiv::MajorTick ] << 0 << 4 
                << 16 << 32 << 64 << 96 << 128;
            ticks[ QwtScaleDiv::MediumTick ] << 24 << 48 << 80 << 112;
            ticks[ QwtScaleDiv::MinorTick ] 
                << 0.5 << 1 << 2 
                << 7 << 10 << 13
                << 20 << 28 
                << 40 << 56 
                << 72 << 88 
                << 104 << 120; 
 
            knob->setScale( QwtScaleDiv( 0, 128, ticks ) );

            knob->setTotalSteps( 100 );
            knob->setStepAlignment( false );
            knob->setSingleSteps( 1 );
            knob->setPageSteps( 5 );

            break;
        }
        case 3:
        {
            knob->setKnobStyle( QwtKnob::Flat );
            knob->setMarkerStyle( QwtKnob::Notch );
            knob->setScaleEngine( new QwtLogScaleEngine() );
            knob->setScaleStepSize( 1.0 );
            knob->setScale( 0.1, 1000.0 );
            knob->setScaleMaxMinor( 10 );
            break;
        }
        case 4:
        {
            knob->setKnobStyle( QwtKnob::Raised );
            knob->setMarkerStyle( QwtKnob::Dot );
            knob->setWrapping( true );
            break;
        }
        case 5:
        {
            knob->setKnobStyle( QwtKnob::Styled );
            knob->setMarkerStyle( QwtKnob::Triangle );
            knob->setTotalAngle( 180.0 );
            knob->setScale( 100, -100 );
            break;
        }
    }

    return knob;
}

void KnobBox::setNum( double v )
{
    QString text;
    text.setNum( v, 'f', 2 );

    d_label->setText( text );
}
```

### `examples/controls/knobbox.h`

```cpp
#ifndef _KNOB_BOX_H_
#define _KNOB_BOX_H_

#include <qwidget.h>

class QLabel;
class QwtKnob;

class KnobBox: public QWidget
{
    Q_OBJECT
public:
    KnobBox( QWidget *parent, int knobType );

private Q_SLOTS:
    void setNum( double v );

private:
    QwtKnob *createKnob( int knobType ) const;

    QwtKnob *d_knob;
    QLabel *d_label;
};

#endif
```

### `examples/controls/knobtab.cpp`

```cpp
#include "knobtab.h"
#include "knobbox.h"
#include <qlayout.h>

KnobTab::KnobTab( QWidget *parent ):
    QWidget( parent )
{
    QGridLayout *layout = new QGridLayout( this );

    const int numRows = 3;
    for ( int i = 0; i < 2 * numRows; i++ )
    {
        KnobBox *knobBox = new KnobBox( this, i );
        layout->addWidget( knobBox, i / numRows, i % numRows );
    }
}
```

### `examples/controls/knobtab.h`

```cpp
#ifndef _KNOB_TAB_H
#define _KNOB_TAB_H 1

#include <qwidget.h>

class KnobTab: public QWidget
{
public:
    KnobTab( QWidget *parent = NULL );
};

#endif
```

### `examples/controls/main.cpp`

```cpp
#include <qapplication.h>
#include <qtabwidget.h>
#include "slidertab.h"
#include "wheeltab.h"
#include "knobtab.h"
#include "dialtab.h"


int main ( int argc, char **argv )
{
    QApplication a( argc, argv );

    QTabWidget tabWidget;

    SliderTab *sliderTab = new SliderTab();
    sliderTab->setAutoFillBackground( true );
    sliderTab->setPalette( QColor( "DimGray" ) );

    WheelTab *wheelTab = new WheelTab();
    wheelTab->setAutoFillBackground( true );
    wheelTab->setPalette( QColor( "Silver" ) );

    KnobTab *knobTab = new KnobTab();
    knobTab->setAutoFillBackground( true );
    knobTab->setPalette( Qt::darkGray );

    DialTab *dialTab = new DialTab();
    dialTab->setAutoFillBackground( true );
    dialTab->setPalette( Qt::darkGray );

    tabWidget.addTab( new SliderTab, "Slider" );
    tabWidget.addTab( new WheelTab, "Wheel/Thermo" );
    tabWidget.addTab( knobTab, "Knob" );
    tabWidget.addTab( dialTab, "Dial" );

    tabWidget.resize( 800, 600 );
    tabWidget.show();

    return a.exec();
}
```

### `examples/controls/sliderbox.cpp`

```cpp
#include <qlabel.h>
#include <qlayout.h>
#include <qwt_slider.h>
#include <qwt_scale_engine.h>
#include <qwt_transform.h>
#include "sliderbox.h"

SliderBox::SliderBox( int sliderType, QWidget *parent ):
    QWidget( parent )
{
    d_slider = createSlider( sliderType );

    QFlags<Qt::AlignmentFlag> alignment;

    if ( d_slider->orientation() == Qt::Horizontal )
    {
        if ( d_slider->scalePosition() == QwtSlider::TrailingScale )
            alignment = Qt::AlignBottom;
        else
            alignment = Qt::AlignTop;

        alignment |= Qt::AlignHCenter;
    }
    else
    {
        if ( d_slider->scalePosition() == QwtSlider::TrailingScale )
            alignment = Qt::AlignRight;
        else
            alignment = Qt::AlignLeft;

        alignment |= Qt::AlignVCenter;
    }

    d_label = new QLabel( this );
    d_label->setAlignment( alignment );
    d_label->setFixedWidth( d_label->fontMetrics().width( "10000.9" ) );

    connect( d_slider, SIGNAL( valueChanged( double ) ), SLOT( setNum( double ) ) );

    QBoxLayout *layout;
    if ( d_slider->orientation() == Qt::Horizontal )
        layout = new QHBoxLayout( this );
    else
        layout = new QVBoxLayout( this );

    layout->addWidget( d_slider );
    layout->addWidget( d_label );

    setNum( d_slider->value() );
}

QwtSlider *SliderBox::createSlider( int sliderType ) const
{
    QwtSlider *slider = new QwtSlider();

    switch( sliderType )
    {
        case 0:
        {
            slider->setOrientation( Qt::Horizontal );
            slider->setScalePosition( QwtSlider::TrailingScale );
            slider->setTrough( true );
            slider->setGroove( false );
            slider->setSpacing( 0 );
            slider->setHandleSize( QSize( 30, 16 ) );
            slider->setScale( 10.0, -10.0 ); 
            slider->setTotalSteps( 8 ); 
            slider->setSingleSteps( 1 ); 
            slider->setPageSteps( 1 ); 
            slider->setWrapping( true );
            break;
        }
        case 1:
        {
            slider->setOrientation( Qt::Horizontal );
            slider->setScalePosition( QwtSlider::NoScale );
            slider->setTrough( true );
            slider->setGroove( true );
            slider->setScale( 0.0, 1.0 );
            slider->setTotalSteps( 100 );
            slider->setSingleSteps( 1 );
            slider->setPageSteps( 5 );
            break;
        }
        case 2:
        {
            slider->setOrientation( Qt::Horizontal );
            slider->setScalePosition( QwtSlider::LeadingScale );
            slider->setTrough( false );
            slider->setGroove( true );
            slider->setHandleSize( QSize( 12, 25 ) );
            slider->setScale( 1000.0, 3000.0 );
            slider->setTotalSteps( 200.0 );
            slider->setSingleSteps( 2 );
            slider->setPageSteps( 10 );
            break;
        }
        case 3:
        {
            slider->setOrientation( Qt::Horizontal );
            slider->setScalePosition( QwtSlider::TrailingScale );
            slider->setTrough( true );
            slider->setGroove( true );

            QwtLinearScaleEngine *scaleEngine = new QwtLinearScaleEngine( 2 );
            scaleEngine->setTransformation( new QwtPowerTransform( 2 ) );
            slider->setScaleEngine( scaleEngine );
            slider->setScale( 0.0, 128.0 );
            slider->setTotalSteps( 100 );
            slider->setStepAlignment( false );
            slider->setSingleSteps( 1 );
            slider->setPageSteps( 5 );
            break;
        }
        case 4:
        {
            slider->setOrientation( Qt::Vertical );
            slider->setScalePosition( QwtSlider::TrailingScale );
            slider->setTrough( false );
            slider->setGroove( true );
            slider->setScale( 100.0, 0.0 );
            slider->setInvertedControls( true );
            slider->setTotalSteps( 100 );
            slider->setPageSteps( 5 );
            slider->setScaleMaxMinor( 5 );
            break;
        }
        case 5:
        {
            slider->setOrientation( Qt::Vertical );
            slider->setScalePosition( QwtSlider::NoScale );
            slider->setTrough( true );
            slider->setGroove( false );
            slider->setScale( 0.0, 100.0 );
            slider->setTotalSteps( 100 );
            slider->setPageSteps( 10 );
            break;
        }
        case 6:
        {
            slider->setOrientation( Qt::Vertical );
            slider->setScalePosition( QwtSlider::LeadingScale );
            slider->setTrough( true );
            slider->setGroove( true );
            slider->setScaleEngine( new QwtLogScaleEngine );
            slider->setStepAlignment( false );
            slider->setHandleSize( QSize( 20, 32 ) );
            slider->setBorderWidth( 1 );
            slider->setScale( 1.0, 1.0e4 );
            slider->setTotalSteps( 100 );
            slider->setPageSteps( 10 );
            slider->setScaleMaxMinor( 9 );
            break;
        }
    }

    if ( slider )
    {
        QString name( "Slider %1" );
        slider->setObjectName( name.arg( sliderType ) );
    }

    return slider;
}

void SliderBox::setNum( double v )
{
    QString text;
    text.setNum( v, 'f', 2 );

    d_label->setText( text );
}
```

### `examples/controls/sliderbox.h`

```cpp
#ifndef _SLIDER_BOX_H_
#define _SLIDER_BOX_H_ 1

#include <qwidget.h>

class QLabel;
class QwtSlider;

class SliderBox: public QWidget
{
    Q_OBJECT
public:
    SliderBox( int sliderType, QWidget *parent = NULL );

private Q_SLOTS:
    void setNum( double v );

private:
    QwtSlider *createSlider( int sliderType ) const;

    QwtSlider *d_slider;
    QLabel *d_label;
};

#endif
```

### `examples/controls/slidertab.cpp`

```cpp
#include "slidertab.h"
#include "sliderbox.h"
#include <qlayout.h>

SliderTab::SliderTab( QWidget *parent ):
    QWidget( parent )
{
    int i;

    QBoxLayout *hLayout = createLayout( Qt::Vertical );
    for ( i = 0; i < 4; i++ )
        hLayout->addWidget( new SliderBox( i ) );
    hLayout->addStretch();

    QBoxLayout *vLayout = createLayout( Qt::Horizontal );
    for ( ; i < 7; i++ )
        vLayout->addWidget( new SliderBox( i ) );

    QBoxLayout *mainLayout = createLayout( Qt::Horizontal, this );
    mainLayout->addLayout( vLayout );
    mainLayout->addLayout( hLayout, 10 );
}

QBoxLayout *SliderTab::createLayout( 
    Qt::Orientation orientation, QWidget *widget )
{
    QBoxLayout *layout =
        new QBoxLayout( QBoxLayout::LeftToRight, widget );

    if ( orientation == Qt::Vertical )
        layout->setDirection( QBoxLayout::TopToBottom );

    layout->setSpacing( 20 );
    layout->setMargin( 0 );

    return layout;
}
```

### `examples/controls/slidertab.h`

```cpp
#ifndef _SLIDER_TAB_H
#define _SLIDER_TAB_H 1

#include <qwidget.h>

class QBoxLayout;

class SliderTab: public QWidget
{
public:
    SliderTab( QWidget *parent = NULL );

private:
    QBoxLayout *createLayout( Qt::Orientation,
        QWidget *widget = NULL );
};

#endif
```

### `examples/controls/wheelbox.cpp`

```cpp
#include <qlabel.h>
#include <qlayout.h>
#include <qwt_wheel.h>
#include <qwt_thermo.h>
#include <qwt_scale_engine.h>
#include <qwt_transform.h>
#include <qwt_color_map.h>
#include "wheelbox.h"

WheelBox::WheelBox( Qt::Orientation orientation,
        int type, QWidget *parent ):
    QWidget( parent )
{
    QWidget *box = createBox( orientation, type );
    d_label = new QLabel( this );
    d_label->setAlignment( Qt::AlignHCenter | Qt::AlignTop );

    QBoxLayout *layout = new QVBoxLayout( this );
    layout->addWidget( box );
    layout->addWidget( d_label );

    setNum( d_wheel->value() );

    connect( d_wheel, SIGNAL( valueChanged( double ) ), 
        this, SLOT( setNum( double ) ) );
}

QWidget *WheelBox::createBox( 
    Qt::Orientation orientation, int type ) 
{
    d_wheel = new QwtWheel();
    d_wheel->setValue( 80 );
    d_wheel->setWheelWidth( 20 );
    d_wheel->setMass( 1.0 );

    d_thermo = new QwtThermo();
    d_thermo->setOrientation( orientation );

    if ( orientation == Qt::Horizontal )
    {
        d_thermo->setScalePosition( QwtThermo::LeadingScale );
        d_wheel->setOrientation( Qt::Vertical );
    }
    else
    {
        d_thermo->setScalePosition( QwtThermo::TrailingScale );
        d_wheel->setOrientation( Qt::Horizontal );
    }

    switch( type )
    {
        case 0:
        {
            QwtLinearColorMap *colorMap = new QwtLinearColorMap(); 
            colorMap->setColorInterval( Qt::blue, Qt::red );
            d_thermo->setColorMap( colorMap );

            break;
        }
        case 1:
        {
            QwtLinearColorMap *colorMap = new QwtLinearColorMap();
            colorMap->setMode( QwtLinearColorMap::FixedColors );

            int idx = 4;

            colorMap->setColorInterval( Qt::GlobalColor( idx ),
                Qt::GlobalColor( idx + 10 ) );
            for ( int i = 1; i < 10; i++ )
            {
                colorMap->addColorStop( i / 10.0, 
                    Qt::GlobalColor( idx + i ) );
            }

            d_thermo->setColorMap( colorMap );
            break;
        }
        case 2:
        {
            d_wheel->setRange( 10, 1000 );
            d_wheel->setSingleStep( 1.0 );

            d_thermo->setScaleEngine( new QwtLogScaleEngine );
            d_thermo->setScaleMaxMinor( 10 );

            d_thermo->setFillBrush( Qt::darkCyan );
            d_thermo->setAlarmBrush( Qt::magenta );
            d_thermo->setAlarmLevel( 500.0 );

            d_wheel->setValue( 800 );

            break;
        }
        case 3:
        {
            d_wheel->setRange( -1000, 1000 );
            d_wheel->setSingleStep( 1.0 );
            d_wheel->setPalette( QColor( "Tan" ) );

            QwtLinearScaleEngine *scaleEngine = new QwtLinearScaleEngine();
            scaleEngine->setTransformation( new QwtPowerTransform( 2 ) );

            d_thermo->setScaleMaxMinor( 5 );
            d_thermo->setScaleEngine( scaleEngine );

            QPalette pal = palette();
            pal.setColor( QPalette::Base, Qt::darkGray );
            pal.setColor( QPalette::ButtonText, QColor( "darkKhaki" ) );

            d_thermo->setPalette( pal );
            break;
        }
        case 4:
        {
            d_wheel->setRange( -100, 300 );
            d_wheel->setInverted( true );

            QwtLinearColorMap *colorMap = new QwtLinearColorMap(); 
            colorMap->setColorInterval( Qt::darkCyan, Qt::yellow );
            d_thermo->setColorMap( colorMap );

            d_wheel->setValue( 243 );

            break;
        }
        case 5:
        {
            d_thermo->setFillBrush( Qt::darkCyan );
            d_thermo->setAlarmBrush( Qt::magenta );
            d_thermo->setAlarmLevel( 60.0 );

            break;
        }
        case 6:
        {
            d_thermo->setOriginMode( QwtThermo::OriginMinimum );
            d_thermo->setFillBrush( QBrush( "DarkSlateBlue" ) );
            d_thermo->setAlarmBrush( QBrush( "DarkOrange" ) );
            d_thermo->setAlarmLevel( 60.0 );

            break;
        }
        case 7:
        {
            d_wheel->setRange( -100, 100 );

            d_thermo->setOriginMode( QwtThermo::OriginCustom );
            d_thermo->setOrigin( 0.0 );
            d_thermo->setFillBrush( Qt::darkBlue );

            break;
        }
    }

    double min = d_wheel->minimum();
    double max = d_wheel->maximum();

    if ( d_wheel->isInverted() )
        qSwap( min, max );

    d_thermo->setScale( min, max );
    d_thermo->setValue( d_wheel->value() );

    connect( d_wheel, SIGNAL( valueChanged( double ) ), 
        d_thermo, SLOT( setValue( double ) ) );

    QWidget *box = new QWidget();

    QBoxLayout *layout;

    if ( orientation == Qt::Horizontal )
        layout = new QHBoxLayout( box );
    else
        layout = new QVBoxLayout( box );

    layout->addWidget( d_thermo, Qt::AlignCenter );
    layout->addWidget( d_wheel );

    return box;
}

void WheelBox::setNum( double v )
{
    QString text;
    text.setNum( v, 'f', 2 );

    d_label->setText( text );
}
```

### `examples/controls/wheelbox.h`

```cpp
#ifndef _WHEEL_BOX_H_
#define _WHEEL_BOX_H_ 1

#include <qwidget.h>

class QLabel;
class QwtThermo;
class QwtWheel;

class WheelBox: public QWidget
{
    Q_OBJECT
public:
    WheelBox( Qt::Orientation, 
        int type, QWidget *parent = NULL );

private Q_SLOTS:
    void setNum( double v );

private:
    QWidget *createBox( Qt::Orientation, int type );

private:
    QwtWheel *d_wheel;
    QwtThermo *d_thermo;
    QLabel *d_label;
};

#endif
```

### `examples/controls/wheeltab.cpp`

```cpp
#include "wheeltab.h"
#include "wheelbox.h"
#include <qlayout.h>

WheelTab::WheelTab( QWidget *parent ):
    QWidget( parent )
{
    const int numBoxes = 4;

    QGridLayout *layout1 = new QGridLayout();
    for ( int i = 0; i < numBoxes; i++ )
    {
        WheelBox *box = new WheelBox( Qt::Vertical, i );
        layout1->addWidget( box, i / 2, i % 2 );
    }

    QGridLayout *layout2 = new QGridLayout();
    for ( int i = 0; i < numBoxes; i++ )
    {
        WheelBox *box = new WheelBox( Qt::Horizontal, i + numBoxes );
        layout2->addWidget( box, i / 2, i % 2 );
    }

    QHBoxLayout *layout = new QHBoxLayout( this );
    layout->addLayout( layout1, 2 );
    layout->addLayout( layout2, 5 );
}
```

### `examples/controls/wheeltab.h`

```cpp
#ifndef _WHEEL_TAB_H
#define _WHEEL_TAB_H 1

#include <qwidget.h>

class WheelTab: public QWidget
{
public:
    WheelTab( QWidget *parent = NULL );
};

#endif
```

### `examples/cpuplot/cpupiemarker.cpp`

```cpp
#include <qpainter.h>
#include <qwt_scale_map.h>
#include <qwt_plot_curve.h>
#include "cpuplot.h"
#include "cpupiemarker.h"

CpuPieMarker::CpuPieMarker()
{
    setZ( 1000 );
    setRenderHint( QwtPlotItem::RenderAntialiased, true );
}

int CpuPieMarker::rtti() const
{
    return QwtPlotItem::Rtti_PlotUserItem;
}

void CpuPieMarker::draw( QPainter *painter,
    const QwtScaleMap &, const QwtScaleMap &,
    const QRectF &rect ) const
{
    const CpuPlot *cpuPlot = static_cast<CpuPlot *> ( plot() );

    const QwtScaleMap yMap = cpuPlot->canvasMap( QwtPlot::yLeft );

    const int margin = 5;

    QRectF pieRect;
    pieRect.setX( rect.x() + margin );
    pieRect.setY( rect.y() + margin );
    pieRect.setHeight( yMap.transform( 80.0 ) );
    pieRect.setWidth( pieRect.height() );

    const int dataType[] = { CpuPlot::User, CpuPlot::System, CpuPlot::Idle };

    int angle = static_cast<int>( 5760 * 0.75 );
    for ( unsigned int i = 0;
        i < sizeof( dataType ) / sizeof( dataType[0] ); i++ )
    {
        const QwtPlotCurve *curve = cpuPlot->cpuCurve( dataType[i] );
        if ( curve->dataSize() > 0 )
        {
            const int value = static_cast<int>( 5760 * curve->sample( 0 ).y() / 100.0 );

            painter->save();
            painter->setBrush( QBrush( curve->brush().color(), Qt::SolidPattern ) );
            if ( value != 0 )
                painter->drawPie( pieRect, -angle, -value );
            painter->restore();

            angle += value;
        }
    }
}
```

### `examples/cpuplot/cpupiemarker.h`

```cpp
//-----------------------------------------------------------------
// This class shows how to extend QwtPlotItems. It displays a
// pie chart of user/total/idle cpu usage in percent.
//-----------------------------------------------------------------

#include <qwt_plot_item.h>

class CpuPieMarker: public QwtPlotItem
{
public:
    CpuPieMarker();

    virtual int rtti() const;

    virtual void draw( QPainter *,
        const QwtScaleMap &, const QwtScaleMap &, const QRectF & ) const;
};
```

### `examples/cpuplot/cpuplot.cpp`

```cpp
#include <qapplication.h>
#include <qlayout.h>
#include <qlabel.h>
#include <qpainter.h>
#include <qwt_plot_layout.h>
#include <qwt_plot_curve.h>
#include <qwt_scale_draw.h>
#include <qwt_scale_widget.h>
#include <qwt_legend.h>
#include <qwt_legend_label.h>
#include <qwt_plot_canvas.h>
#include "cpupiemarker.h"
#include "cpuplot.h"

class TimeScaleDraw: public QwtScaleDraw
{
public:
    TimeScaleDraw( const QTime &base ):
        baseTime( base )
    {
    }
    virtual QwtText label( double v ) const
    {
        QTime upTime = baseTime.addSecs( static_cast<int>( v ) );
        return upTime.toString();
    }
private:
    QTime baseTime;
};

class Background: public QwtPlotItem
{
public:
    Background()
    {
        setZ( 0.0 );
    }

    virtual int rtti() const
    {
        return QwtPlotItem::Rtti_PlotUserItem;
    }

    virtual void draw( QPainter *painter,
        const QwtScaleMap &, const QwtScaleMap &yMap,
        const QRectF &canvasRect ) const
    {
        QColor c( Qt::white );
        QRectF r = canvasRect;

        for ( int i = 100; i > 0; i -= 10 )
        {
            r.setBottom( yMap.transform( i - 10 ) );
            r.setTop( yMap.transform( i ) );
            painter->fillRect( r, c );

            c = c.dark( 110 );
        }
    }
};

class CpuCurve: public QwtPlotCurve
{
public:
    CpuCurve( const QString &title ):
        QwtPlotCurve( title )
    {
        setRenderHint( QwtPlotItem::RenderAntialiased );
    }

    void setColor( const QColor &color )
    {
        QColor c = color;
        c.setAlpha( 150 );

        setPen( QPen( Qt::NoPen ) );
        setBrush( c );
    }
};

CpuPlot::CpuPlot( QWidget *parent ):
    QwtPlot( parent ),
    dataCount( 0 )
{
    setAutoReplot( false );

    QwtPlotCanvas *canvas = new QwtPlotCanvas();
    canvas->setBorderRadius( 10 );

    setCanvas( canvas );

    plotLayout()->setAlignCanvasToScales( true );

    QwtLegend *legend = new QwtLegend;
    legend->setDefaultItemMode( QwtLegendData::Checkable );
    insertLegend( legend, QwtPlot::RightLegend );

    setAxisTitle( QwtPlot::xBottom, " System Uptime [h:m:s]" );
    setAxisScaleDraw( QwtPlot::xBottom,
        new TimeScaleDraw( cpuStat.upTime() ) );
    setAxisScale( QwtPlot::xBottom, 0, HISTORY );
    setAxisLabelRotation( QwtPlot::xBottom, -50.0 );
    setAxisLabelAlignment( QwtPlot::xBottom, Qt::AlignLeft | Qt::AlignBottom );

    /*
     In situations, when there is a label at the most right position of the
     scale, additional space is needed to display the overlapping part
     of the label would be taken by reducing the width of scale and canvas.
     To avoid this "jumping canvas" effect, we add a permanent margin.
     We don't need to do the same for the left border, because there
     is enough space for the overlapping label below the left scale.
     */

    QwtScaleWidget *scaleWidget = axisWidget( QwtPlot::xBottom );
    const int fmh = QFontMetrics( scaleWidget->font() ).height();
    scaleWidget->setMinBorderDist( 0, fmh / 2 );

    setAxisTitle( QwtPlot::yLeft, "Cpu Usage [%]" );
    setAxisScale( QwtPlot::yLeft, 0, 100 );

    Background *bg = new Background();
    bg->attach( this );

    CpuPieMarker *pie = new CpuPieMarker();
    pie->attach( this );

    CpuCurve *curve;

    curve = new CpuCurve( "System" );
    curve->setColor( Qt::red );
    curve->attach( this );
    data[System].curve = curve;

    curve = new CpuCurve( "User" );
    curve->setColor( Qt::blue );
    curve->setZ( curve->z() - 1 );
    curve->attach( this );
    data[User].curve = curve;

    curve = new CpuCurve( "Total" );
    curve->setColor( Qt::black );
    curve->setZ( curve->z() - 2 );
    curve->attach( this );
    data[Total].curve = curve;

    curve = new CpuCurve( "Idle" );
    curve->setColor( Qt::darkCyan );
    curve->setZ( curve->z() - 3 );
    curve->attach( this );
    data[Idle].curve = curve;

    showCurve( data[System].curve, true );
    showCurve( data[User].curve, true );
    showCurve( data[Total].curve, false );
    showCurve( data[Idle].curve, false );

    for ( int i = 0; i < HISTORY; i++ )
        timeData[HISTORY - 1 - i] = i;

    ( void )startTimer( 1000 ); // 1 second

    connect( legend, SIGNAL( checked( const QVariant &, bool, int ) ),
        SLOT( legendChecked( const QVariant &, bool ) ) );
}

void CpuPlot::timerEvent( QTimerEvent * )
{
    for ( int i = dataCount; i > 0; i-- )
    {
        for ( int c = 0; c < NCpuData; c++ )
        {
            if ( i < HISTORY )
                data[c].data[i] = data[c].data[i-1];
        }
    }

    cpuStat.statistic( data[User].data[0], data[System].data[0] );

    data[Total].data[0] = data[User].data[0] + data[System].data[0];
    data[Idle].data[0] = 100.0 - data[Total].data[0];

    if ( dataCount < HISTORY )
        dataCount++;

    for ( int j = 0; j < HISTORY; j++ )
        timeData[j]++;

    setAxisScale( QwtPlot::xBottom,
        timeData[HISTORY - 1], timeData[0] );

    for ( int c = 0; c < NCpuData; c++ )
    {
        data[c].curve->setRawSamples(
            timeData, data[c].data, dataCount );
    }

    replot();
}

void CpuPlot::legendChecked( const QVariant &itemInfo, bool on )
{
    QwtPlotItem *plotItem = infoToItem( itemInfo );
    if ( plotItem )
        showCurve( plotItem, on );
}

void CpuPlot::showCurve( QwtPlotItem *item, bool on )
{
    item->setVisible( on );

    QwtLegend *lgd = qobject_cast<QwtLegend *>( legend() );

    QList<QWidget *> legendWidgets = 
        lgd->legendWidgets( itemToInfo( item ) );

    if ( legendWidgets.size() == 1 )
    {
        QwtLegendLabel *legendLabel =
            qobject_cast<QwtLegendLabel *>( legendWidgets[0] );

        if ( legendLabel )
            legendLabel->setChecked( on );
    }

    replot();
}

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    QWidget vBox;
    vBox.setWindowTitle( "Cpu Plot" );

    CpuPlot *plot = new CpuPlot( &vBox );
    plot->setTitle( "History" );

    const int margin = 5;
    plot->setContentsMargins( margin, margin, margin, margin );

    QString info( "Press the legend to en/disable a curve" );

    QLabel *label = new QLabel( info, &vBox );

    QVBoxLayout *layout = new QVBoxLayout( &vBox );
    layout->addWidget( plot );
    layout->addWidget( label );

    vBox.resize( 600, 400 );
    vBox.show();

    return a.exec();
}
```

### `examples/cpuplot/cpuplot.h`

```cpp
#include <qwt_plot.h>
#include "cpustat.h"

#define HISTORY 60 // seconds

class QwtPlotCurve;

class CpuPlot : public QwtPlot
{
    Q_OBJECT
public:
    enum CpuData
    {
        User,
        System,
        Total,
        Idle,

        NCpuData
    };

    CpuPlot( QWidget * = 0 );
    const QwtPlotCurve *cpuCurve( int id ) const
    {
        return data[id].curve;
    }

protected:
    void timerEvent( QTimerEvent *e );

private Q_SLOTS:
    void legendChecked( const QVariant &, bool on );

private:
    void showCurve( QwtPlotItem *, bool on );

    struct
    {
        QwtPlotCurve *curve;
        double data[HISTORY];
    } data[NCpuData];

    double timeData[HISTORY];

    int dataCount;
    CpuStat cpuStat;
};
```

### `examples/cpuplot/cpustat.cpp`

```cpp
#include <qstringlist.h>
#include <qfile.h>
#include <qtextstream.h>
#include "cpustat.h"

CpuStat::CpuStat()
{
    lookUp( procValues );
}

QTime CpuStat::upTime() const
{
    QTime t( 0, 0, 0 );
    for ( int i = 0; i < NValues; i++ )
        t = t.addSecs( int( procValues[i] / 100 ) );

    return t;
}

void CpuStat::statistic( double &user, double &system )
{
    double values[NValues];

    lookUp( values );

    double userDelta = values[User] + values[Nice]
        - procValues[User] - procValues[Nice];
    double systemDelta = values[System] - procValues[System];

    double totalDelta = 0;
    for ( int i = 0; i < NValues; i++ )
        totalDelta += values[i] - procValues[i];

    user = userDelta / totalDelta * 100.0;
    system = systemDelta / totalDelta * 100.0;

    for ( int j = 0; j < NValues; j++ )
        procValues[j] = values[j];
}

void CpuStat::lookUp( double values[NValues] ) const
{
    QFile file( "/proc/stat" );
#if 1
    if ( !file.open( QIODevice::ReadOnly ) )
#else
    if ( true )
#endif
    {
        static double dummyValues[][NValues] =
        {
            { 103726, 0, 23484, 819556 },
            { 103783, 0, 23489, 819604 },
            { 103798, 0, 23490, 819688 },
            { 103820, 0, 23490, 819766 },
            { 103840, 0, 23493, 819843 },
            { 103875, 0, 23499, 819902 },
            { 103917, 0, 23504, 819955 },
            { 103950, 0, 23508, 820018 },
            { 103987, 0, 23510, 820079 },
            { 104020, 0, 23513, 820143 },
            { 104058, 0, 23514, 820204 },
            { 104099, 0, 23520, 820257 },
            { 104121, 0, 23525, 820330 },
            { 104159, 0, 23530, 820387 },
            { 104176, 0, 23534, 820466 },
            { 104215, 0, 23538, 820523 },
            { 104245, 0, 23541, 820590 },
            { 104267, 0, 23545, 820664 },
            { 104311, 0, 23555, 820710 },
            { 104355, 0, 23565, 820756 },
            { 104367, 0, 23567, 820842 },
            { 104383, 0, 23572, 820921 },
            { 104396, 0, 23577, 821003 },
            { 104413, 0, 23579, 821084 },
            { 104446, 0, 23588, 821142 },
            { 104521, 0, 23594, 821161 },
            { 104611, 0, 23604, 821161 },
            { 104708, 0, 23607, 821161 },
            { 104804, 0, 23611, 821161 },
            { 104895, 0, 23620, 821161 },
            { 104993, 0, 23622, 821161 },
            { 105089, 0, 23626, 821161 },
            { 105185, 0, 23630, 821161 },
            { 105281, 0, 23634, 821161 },
            { 105379, 0, 23636, 821161 },
            { 105472, 0, 23643, 821161 },
            { 105569, 0, 23646, 821161 },
            { 105666, 0, 23649, 821161 },
            { 105763, 0, 23652, 821161 },
            { 105828, 0, 23661, 821187 },
            { 105904, 0, 23666, 821206 },
            { 105999, 0, 23671, 821206 },
            { 106094, 0, 23676, 821206 },
            { 106184, 0, 23686, 821206 },
            { 106273, 0, 23692, 821211 },
            { 106306, 0, 23700, 821270 },
            { 106341, 0, 23703, 821332 },
            { 106392, 0, 23709, 821375 },
            { 106423, 0, 23715, 821438 },
            { 106472, 0, 23721, 821483 },
            { 106531, 0, 23727, 821517 },
            { 106562, 0, 23732, 821582 },
            { 106597, 0, 23736, 821643 },
            { 106633, 0, 23737, 821706 },
            { 106666, 0, 23742, 821768 },
            { 106697, 0, 23744, 821835 },
            { 106730, 0, 23748, 821898 },
            { 106765, 0, 23751, 821960 },
            { 106799, 0, 23754, 822023 },
            { 106831, 0, 23758, 822087 },
            { 106862, 0, 23761, 822153 },
            { 106899, 0, 23763, 822214 },
            { 106932, 0, 23766, 822278 },
            { 106965, 0, 23768, 822343 },
            { 107009, 0, 23771, 822396 },
            { 107040, 0, 23775, 822461 },
            { 107092, 0, 23780, 822504 },
            { 107143, 0, 23787, 822546 },
            { 107200, 0, 23795, 822581 },
            { 107250, 0, 23803, 822623 },
            { 107277, 0, 23810, 822689 },
            { 107286, 0, 23810, 822780 },
            { 107313, 0, 23817, 822846 },
            { 107325, 0, 23818, 822933 },
            { 107332, 0, 23818, 823026 },
            { 107344, 0, 23821, 823111 },
            { 107357, 0, 23821, 823198 },
            { 107368, 0, 23823, 823284 },
            { 107375, 0, 23824, 823377 },
            { 107386, 0, 23825, 823465 },
            { 107396, 0, 23826, 823554 },
            { 107422, 0, 23830, 823624 },
            { 107434, 0, 23831, 823711 },
            { 107456, 0, 23835, 823785 },
            { 107468, 0, 23838, 823870 },
            { 107487, 0, 23840, 823949 },
            { 107515, 0, 23843, 824018 },
            { 107528, 0, 23846, 824102 },
            { 107535, 0, 23851, 824190 },
            { 107548, 0, 23853, 824275 },
            { 107562, 0, 23857, 824357 },
            { 107656, 0, 23863, 824357 },
            { 107751, 0, 23868, 824357 },
            { 107849, 0, 23870, 824357 },
            { 107944, 0, 23875, 824357 },
            { 108043, 0, 23876, 824357 },
            { 108137, 0, 23882, 824357 },
            { 108230, 0, 23889, 824357 },
            { 108317, 0, 23902, 824357 },
            { 108412, 0, 23907, 824357 },
            { 108511, 0, 23908, 824357 },
            { 108608, 0, 23911, 824357 },
            { 108704, 0, 23915, 824357 },
            { 108801, 0, 23918, 824357 },
            { 108891, 0, 23928, 824357 },
            { 108987, 0, 23932, 824357 },
            { 109072, 0, 23943, 824361 },
            { 109079, 0, 23943, 824454 },
            { 109086, 0, 23944, 824546 },
            { 109098, 0, 23950, 824628 },
            { 109108, 0, 23955, 824713 },
            { 109115, 0, 23957, 824804 },
            { 109122, 0, 23958, 824896 },
            { 109132, 0, 23959, 824985 },
            { 109142, 0, 23961, 825073 },
            { 109146, 0, 23962, 825168 },
            { 109153, 0, 23964, 825259 },
            { 109162, 0, 23966, 825348 },
            { 109168, 0, 23969, 825439 },
            { 109176, 0, 23971, 825529 },
            { 109185, 0, 23974, 825617 },
            { 109193, 0, 23977, 825706 },
            { 109198, 0, 23978, 825800 },
            { 109206, 0, 23978, 825892 },
            { 109212, 0, 23981, 825983 },
            { 109219, 0, 23981, 826076 },
            { 109225, 0, 23981, 826170 },
            { 109232, 0, 23984, 826260 },
            { 109242, 0, 23984, 826350 },
            { 109255, 0, 23986, 826435 },
            { 109268, 0, 23987, 826521 },
            { 109283, 0, 23990, 826603 },
            { 109288, 0, 23991, 826697 },
            { 109295, 0, 23993, 826788 },
            { 109308, 0, 23994, 826874 },
            { 109322, 0, 24009, 826945 },
            { 109328, 0, 24011, 827037 },
            { 109338, 0, 24012, 827126 },
            { 109347, 0, 24012, 827217 },
            { 109354, 0, 24017, 827305 },
            { 109367, 0, 24017, 827392 },
            { 109371, 0, 24019, 827486 },
        };
        static int counter = 0;

        for ( int i = 0; i < NValues; i++ )
            values[i] = dummyValues[counter][i];

        counter = ( counter + 1 )
            % ( sizeof( dummyValues ) / sizeof( dummyValues[0] ) );
    }
    else
    {
        QTextStream textStream( &file );
        do
        {
            QString line = textStream.readLine();
            line = line.trimmed();
            if ( line.startsWith( "cpu " ) )
            {
                const QStringList valueList =
                    line.split( " ",  QString::SkipEmptyParts );
                if ( valueList.count() >= 5 )
                {
                    for ( int i = 0; i < NValues; i++ )
                        values[i] = valueList[i+1].toDouble();
                }
                break;
            }
        }
        while( !textStream.atEnd() );
    }
}
```

### `examples/cpuplot/cpustat.h`

```cpp
#include <qdatetime.h>

class CpuStat
{
public:
    CpuStat();
    void statistic( double &user, double &system );
    QTime upTime() const;

    enum Value
    {
        User,
        Nice,
        System,
        Idle,

        NValues
    };

private:
    void lookUp( double[NValues] ) const;
    double procValues[NValues];
};
```

### `examples/curvdemo1/curvdemo1.cpp`

```cpp

#include <qwt_scale_map.h>
#include <qwt_plot_curve.h>
#include <qwt_symbol.h>
#include <qwt_math.h>
#include <qcolor.h>
#include <qpainter.h>
#include <qapplication.h>
#include <qframe.h>

//------------------------------------------------------------
//      curvdemo1
//
//  This example program features some of the different
//  display styles of the QwtPlotCurve class
//------------------------------------------------------------


//
//   Array Sizes
//
const int Size = 27;
const int CurvCnt = 6;

//
//   Arrays holding the values
//
double xval[Size];
double yval[Size];
QwtScaleMap xMap;
QwtScaleMap yMap;

class MainWin : public QFrame
{
public:
    MainWin();

protected:
    virtual void paintEvent( QPaintEvent * );
    void drawContents( QPainter *p );

private:
    void shiftDown( QRect &rect, int offset ) const;

    QwtPlotCurve d_curves[CurvCnt];
};

MainWin::MainWin()
{
    int i;

    xMap.setScaleInterval( -0.5, 10.5 );
    yMap.setScaleInterval( -1.1, 1.1 );

    //
    //  Frame style
    //
    setFrameStyle( QFrame::Box | QFrame::Raised );
    setLineWidth( 2 );
    setMidLineWidth( 3 );

    //
    // Calculate values
    //
    for( i = 0; i < Size; i++ )
    {
        xval[i] = double( i ) * 10.0 / double( Size - 1 );
        yval[i] = qSin( xval[i] ) * qCos( 2.0 * xval[i] );
    }

    //
    //  define curve styles
    //
    i = 0;

    d_curves[i].setSymbol( new QwtSymbol( QwtSymbol::Cross, Qt::NoBrush,
        QPen( Qt::black ), QSize( 5, 5 ) ) );
    d_curves[i].setPen( Qt::darkGreen );
    d_curves[i].setStyle( QwtPlotCurve::Lines );
    d_curves[i].setCurveAttribute( QwtPlotCurve::Fitted );
    i++;

    d_curves[i].setSymbol( new QwtSymbol( QwtSymbol::Ellipse, Qt::yellow,
        QPen( Qt::blue ), QSize( 5, 5 ) ) );
    d_curves[i].setPen( Qt::red );
    d_curves[i].setStyle( QwtPlotCurve::Sticks );
    i++;

    d_curves[i].setPen( Qt::darkBlue );
    d_curves[i].setStyle( QwtPlotCurve::Lines );
    i++;

    d_curves[i].setPen( Qt::darkBlue );
    d_curves[i].setStyle( QwtPlotCurve::Lines );
    d_curves[i].setRenderHint( QwtPlotItem::RenderAntialiased );
    i++;

    d_curves[i].setPen( Qt::darkCyan );
    d_curves[i].setStyle( QwtPlotCurve::Steps );
    i++;

    d_curves[i].setSymbol( new QwtSymbol( QwtSymbol::XCross, Qt::NoBrush,
        QPen( Qt::darkMagenta ), QSize( 5, 5 ) ) );
    d_curves[i].setStyle( QwtPlotCurve::NoCurve );
    i++;


    //
    // attach data
    //
    for( i = 0; i < CurvCnt; i++ )
        d_curves[i].setRawSamples( xval, yval, Size );
}

void MainWin::shiftDown( QRect &rect, int offset ) const
{
    rect.translate( 0, offset );
}

void MainWin::paintEvent( QPaintEvent *event )
{
    QFrame::paintEvent( event );

    QPainter painter( this );
    painter.setClipRect( contentsRect() );
    drawContents( &painter );
}


//
//  REDRAW CONTENTS
//
void MainWin::drawContents( QPainter *painter )
{
    int deltay, i;

    QRect r = contentsRect();

    deltay = r.height() / CurvCnt - 1;

    r.setHeight( deltay );

    //
    //  draw curves
    //
    for ( i = 0; i < CurvCnt; i++ )
    {
        xMap.setPaintInterval( r.left(), r.right() );
        yMap.setPaintInterval( r.top(), r.bottom() );

        painter->setRenderHint( QPainter::Antialiasing,
            d_curves[i].testRenderHint( QwtPlotItem::RenderAntialiased ) );
        d_curves[i].draw( painter, xMap, yMap, r );

        shiftDown( r, deltay );
    }

    //
    // draw titles
    //
    r = contentsRect();     // reset r
    painter->setFont( QFont( "Helvetica", 8 ) );

    const int alignment = Qt::AlignTop | Qt::AlignHCenter;

    painter->setPen( Qt::black );

    painter->drawText( 0, r.top(), r.width(), painter->fontMetrics().height(),
        alignment, "Style: Line/Fitted, Symbol: Cross" );
    shiftDown( r, deltay );

    painter->drawText( 0, r.top(), r.width(), painter->fontMetrics().height(),
        alignment, "Style: Sticks, Symbol: Ellipse" );
    shiftDown( r, deltay );

    painter->drawText( 0 , r.top(), r.width(), painter->fontMetrics().height(),
        alignment, "Style: Lines, Symbol: None" );
    shiftDown( r, deltay );

    painter->drawText( 0 , r.top(), r.width(), painter->fontMetrics().height(),
        alignment, "Style: Lines, Symbol: None, Antialiased" );
    shiftDown( r, deltay );

    painter->drawText( 0, r.top(), r.width(), painter->fontMetrics().height(),
        alignment, "Style: Steps, Symbol: None" );
    shiftDown( r, deltay );

    painter->drawText( 0, r.top(), r.width(), painter->fontMetrics().height(),
        alignment, "Style: NoCurve, Symbol: XCross" );
}

int main ( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWin w;

    w.resize( 300, 600 );
    w.show();

    return a.exec();
}
```

### `examples/dials/attitude_indicator.cpp`

```cpp
﻿#include "attitude_indicator.h"
#include <qwt_point_polar.h>
#include <qwt_round_scale_draw.h>
#include <qevent.h>
#include <qpainter.h>
#include <qpolygon.h>
#include <qpainterpath.h>

AttitudeIndicatorNeedle::AttitudeIndicatorNeedle( const QColor &color )
{
    QPalette palette;
    palette.setColor( QPalette::Text, color );
    setPalette( palette );
}

void AttitudeIndicatorNeedle::drawNeedle( QPainter *painter,
    double length, QPalette::ColorGroup colorGroup ) const
{
    double triangleSize = length * 0.1;
    double pos = length - 2.0;

    QPainterPath path;
    path.moveTo( pos, 0 );
    path.lineTo( pos - 2 * triangleSize, triangleSize );
    path.lineTo( pos - 2 * triangleSize, -triangleSize );
    path.closeSubpath();

    painter->setBrush( palette().brush( colorGroup, QPalette::Text ) );
    painter->drawPath( path );

    double l = length - 2;
    painter->setPen( QPen( palette().color( colorGroup, QPalette::Text ), 3 ) );
    painter->drawLine( QPointF( 0.0, -l ), QPointF( 0.0, l ) );
}

AttitudeIndicator::AttitudeIndicator(
    QWidget *parent ):
    QwtDial( parent ),
    d_gradient( 0.0 )
{
    QwtRoundScaleDraw *scaleDraw = new QwtRoundScaleDraw();
    scaleDraw->enableComponent( QwtAbstractScaleDraw::Backbone, false );
    scaleDraw->enableComponent( QwtAbstractScaleDraw::Labels, false );
    setScaleDraw( scaleDraw );

    setMode( RotateScale );
    setWrapping( true );

    setOrigin( 270.0 );

    setScaleMaxMinor( 0 );
    setScaleStepSize( 30.0 );
    setScale( 0.0, 360.0 );

    const QColor color = palette().color( QPalette::Text );
    setNeedle( new AttitudeIndicatorNeedle( color ) );
}

void AttitudeIndicator::setGradient( double gradient )
{
    if ( gradient < -1.0 )
        gradient = -1.0;
    else if ( gradient > 1.0 )
        gradient = 1.0;

    if ( d_gradient != gradient )
    {
        d_gradient = gradient;
        update();
    }
}

void AttitudeIndicator::drawScale( QPainter *painter, 
    const QPointF &center, double radius ) const
{
    const double offset = 4.0;

    const QPointF p0 = qwtPolar2Pos( center, offset, 1.5 * M_PI );

    const double w = innerRect().width();

    QPainterPath path;
    path.moveTo( qwtPolar2Pos( p0, w, 0.0 ) );
    path.lineTo( qwtPolar2Pos( path.currentPosition(), 2 * w, M_PI ) );
    path.lineTo( qwtPolar2Pos( path.currentPosition(), w, 0.5 * M_PI ) );
    path.lineTo( qwtPolar2Pos( path.currentPosition(), w, 0.0 ) );

    painter->save();
    painter->setClipPath( path ); // swallow 180 - 360 degrees

    QwtDial::drawScale( painter, center, radius );

    painter->restore();
}

void AttitudeIndicator::drawScaleContents( QPainter *painter,
        const QPointF &, double ) const
{
    int dir = 360 - qRound( origin() - value() ); // counter clockwise
    int arc = 90 + qRound( gradient() * 90 );

    const QColor skyColor( 38, 151, 221 );

    painter->save();
    painter->setBrush( skyColor );
    painter->drawChord( scaleInnerRect(),
        ( dir - arc ) * 16, 2 * arc * 16 );
    painter->restore();
}

void AttitudeIndicator::keyPressEvent( QKeyEvent *event )
{
    switch( event->key() )
    {
        case Qt::Key_Plus:
        {
            setGradient( gradient() + 0.05 );
            break;
        }
        case Qt::Key_Minus:
        {
            setGradient( gradient() - 0.05 );
            break;
        }
        default:
            QwtDial::keyPressEvent( event );
    }
}
```

### `examples/dials/attitude_indicator.h`

```cpp
#include <qwt_dial.h>
#include <qwt_dial_needle.h>

class AttitudeIndicatorNeedle: public QwtDialNeedle
{
public:
    AttitudeIndicatorNeedle( const QColor & );

protected:
    virtual void drawNeedle( QPainter *,
        double length, QPalette::ColorGroup ) const;
};

class AttitudeIndicator: public QwtDial
{
    Q_OBJECT

public:
    AttitudeIndicator( QWidget *parent = NULL );

    double angle() const { return value(); }
    double gradient() const { return d_gradient; }

public Q_SLOTS:
    void setGradient( double );
    void setAngle( double angle ) { setValue( angle ); }

protected:
    virtual void keyPressEvent( QKeyEvent * );

    virtual void drawScale( QPainter *, 
        const QPointF &center, double radius ) const;

    virtual void drawScaleContents( QPainter *painter,
        const QPointF &center, double radius ) const;

private:
    double d_gradient;
};
```

### `examples/dials/cockpit_grid.cpp`

```cpp
#include <qlayout.h>
#include <qtimer.h>
#include <qwt_analog_clock.h>
#include <qwt_round_scale_draw.h>
#include "attitude_indicator.h"
#include "speedo_meter.h"
#include "cockpit_grid.h"

CockpitGrid::CockpitGrid( QWidget *parent ):
    QFrame( parent )
{
    setAutoFillBackground( true );

    setPalette( colorTheme( QColor( Qt::darkGray ).dark( 150 ) ) );

    QGridLayout *layout = new QGridLayout( this );
    layout->setSpacing( 5 );
    layout->setMargin( 0 );

    int i;
    for ( i = 0; i < 3; i++ )
    {
        QwtDial *dial = createDial( i );
        layout->addWidget( dial, 0, i );
    }

    for ( i = 0; i < layout->columnCount(); i++ )
        layout->setColumnStretch( i, 1 );
}

QwtDial *CockpitGrid::createDial( int pos )
{
    QwtDial *dial = NULL;
    switch( pos )
    {
        case 0:
        {
            d_clock = new QwtAnalogClock( this );
#if 0
            // disable minor ticks
            d_clock->scaleDraw()->setTickLength( QwtScaleDiv::MinorTick, 0 );
#endif

            const QColor knobColor = QColor( Qt::gray ).light( 130 );

            for ( int i = 0; i < QwtAnalogClock::NHands; i++ )
            {
                QColor handColor = QColor( Qt::gray ).light( 150 );
                int width = 8;

                if ( i == QwtAnalogClock::SecondHand )
                {
                    handColor = Qt::gray;
                    width = 5;
                }

                QwtDialSimpleNeedle *hand = new QwtDialSimpleNeedle(
                    QwtDialSimpleNeedle::Arrow, true, handColor, knobColor );
                hand->setWidth( width );

                d_clock->setHand( static_cast<QwtAnalogClock::Hand>( i ), hand );
            }

            QTimer *timer = new QTimer( d_clock );
            timer->connect( timer, SIGNAL( timeout() ),
                d_clock, SLOT( setCurrentTime() ) );
            timer->start( 1000 );

            dial = d_clock;
            break;
        }
        case 1:
        {
            d_speedo = new SpeedoMeter( this );
            d_speedo->setScaleStepSize( 20.0 );
            d_speedo->setScale( 0.0, 240.0 );
            d_speedo->scaleDraw()->setPenWidth( 2 );

            QTimer *timer = new QTimer( d_speedo );
            timer->connect( timer, SIGNAL( timeout() ),
                this, SLOT( changeSpeed() ) );
            timer->start( 50 );

            dial = d_speedo;
            break;
        }
        case 2:
        {
            d_ai = new AttitudeIndicator( this );
            d_ai->scaleDraw()->setPenWidth( 3 );

            QTimer *gradientTimer = new QTimer( d_ai );
            gradientTimer->connect( gradientTimer, SIGNAL( timeout() ),
                this, SLOT( changeGradient() ) );
            gradientTimer->start( 100 );

            QTimer *angleTimer = new QTimer( d_ai );
            angleTimer->connect( angleTimer, SIGNAL( timeout() ),
                this, SLOT( changeAngle() ) );
            angleTimer->start( 100 );

            dial = d_ai;
            break;
        }

    }

    if ( dial )
    {
        dial->setReadOnly( true );
        dial->setLineWidth( 4 );
        dial->setFrameShadow( QwtDial::Sunken );
    }
    return dial;
}

QPalette CockpitGrid::colorTheme( const QColor &base ) const
{
    QPalette palette;
    palette.setColor( QPalette::Base, base );
    palette.setColor( QPalette::Window, base.dark( 150 ) );
    palette.setColor( QPalette::Mid, base.dark( 110 ) );
    palette.setColor( QPalette::Light, base.light( 170 ) );
    palette.setColor( QPalette::Dark, base.dark( 170 ) );
    palette.setColor( QPalette::Text, base.dark( 200 ).light( 800 ) );
    palette.setColor( QPalette::WindowText, base.dark( 200 ) );

    return palette;
}

void CockpitGrid::changeSpeed()
{
    static double offset = 0.8;

    double speed = d_speedo->value();

    if ( ( speed < 7.0 && offset < 0.0 ) ||
        ( speed > 203.0 && offset > 0.0 ) )
    {
        offset = -offset;
    }

    static int counter = 0;
    switch( counter++ % 12 )
    {
        case 0:
        case 2:
        case 7:
        case 8:
            break;
        default:
            d_speedo->setValue( speed + offset );
    }
}

void CockpitGrid::changeAngle()
{
    static double offset = 0.05;

    double angle = d_ai->angle();
    if ( angle > 180.0 )
        angle -= 360.0;

    if ( ( angle < -5.0 && offset < 0.0 ) ||
        ( angle > 5.0 && offset > 0.0 ) )
    {
        offset = -offset;
    }

    d_ai->setAngle( angle + offset );
}

void CockpitGrid::changeGradient()
{
    static double offset = 0.005;

    double gradient = d_ai->gradient();

    if ( ( gradient < -0.05 && offset < 0.0 ) ||
        ( gradient > 0.05 && offset > 0.0 ) )
    {
        offset = -offset;
    }

    d_ai->setGradient( gradient + offset );
}
```

### `examples/dials/cockpit_grid.h`

```cpp
#include <qframe.h>
#include <qpalette.h>

class QwtDial;
class QwtAnalogClock;
class SpeedoMeter;
class AttitudeIndicator;

class CockpitGrid: public QFrame
{
    Q_OBJECT

public:
    CockpitGrid( QWidget *parent = NULL );

private Q_SLOTS:
    void changeSpeed();
    void changeGradient();
    void changeAngle();

private:
    QPalette colorTheme( const QColor & ) const;
    QwtDial *createDial( int pos );

    QwtAnalogClock *d_clock;
    SpeedoMeter *d_speedo;
    AttitudeIndicator *d_ai;
};
```

### `examples/dials/compass_grid.cpp`

```cpp
#include <qlayout.h>
#include <qwt_compass.h>
#include <qwt_compass_rose.h>
#include <qwt_dial_needle.h>
#include "compass_grid.h"

CompassGrid::CompassGrid( QWidget *parent ):
    QFrame( parent )
{
    QPalette p = palette();
    p.setColor( backgroundRole(), Qt::gray );
    setPalette( p );

    setAutoFillBackground( true );

    QGridLayout *layout = new QGridLayout( this );
    layout->setSpacing( 5 );
    layout->setMargin( 0 );

    int i;
    for ( i = 0; i < 6; i++ )
    {
        QwtCompass *compass = createCompass( i );
        layout->addWidget( compass, i / 3, i % 3 );
    }

    for ( i = 0; i < layout->columnCount(); i++ )
        layout->setColumnStretch( i, 1 );
}

QwtCompass *CompassGrid::createCompass( int pos )
{
    int c;

    QPalette palette0;
    for ( c = 0; c < QPalette::NColorRoles; c++ )
    {
        const QPalette::ColorRole colorRole =
            static_cast<QPalette::ColorRole>( c );

        palette0.setColor( colorRole, QColor() );
    }

    palette0.setColor( QPalette::Base,
        palette().color( backgroundRole() ).light( 120 ) );
    palette0.setColor( QPalette::WindowText,
        palette0.color( QPalette::Base ) );

    QwtCompass *compass = new QwtCompass( this );
    compass->setLineWidth( 4 );
    compass->setFrameShadow(
        pos <= 2 ? QwtCompass::Sunken : QwtCompass::Raised );

    switch( pos )
    {
        case 0:
        {
            /*
              A compass with a rose and no needle. Scale and rose are
              rotating.
             */
            compass->setMode( QwtCompass::RotateScale );

            QwtSimpleCompassRose *rose = new QwtSimpleCompassRose( 16, 2 );
            rose->setWidth( 0.15 );

            compass->setRose( rose );
            break;
        }
        case 1:
        {
            /*
              A windrose, with a scale indicating the main directions only
             */
            QMap<double, QString> map;
            map.insert( 0.0, "N" );
            map.insert( 90.0, "E" );
            map.insert( 180.0, "S" );
            map.insert( 270.0, "W" );

            compass->setScaleDraw( new QwtCompassScaleDraw( map ) );

            QwtSimpleCompassRose *rose = new QwtSimpleCompassRose( 4, 1 );
            compass->setRose( rose );

            compass->setNeedle(
                new QwtCompassWindArrow( QwtCompassWindArrow::Style2 ) );
            compass->setValue( 60.0 );
            break;
        }
        case 2:
        {
            /*
              A compass with a rotating needle in darkBlue. Shows
              a ticks for each degree.
             */

            palette0.setColor( QPalette::Base, Qt::darkBlue );
            palette0.setColor( QPalette::WindowText,
                               QColor( Qt::darkBlue ).dark( 120 ) );
            palette0.setColor( QPalette::Text, Qt::white );

            QwtCompassScaleDraw *scaleDraw = new QwtCompassScaleDraw();
            scaleDraw->enableComponent( QwtAbstractScaleDraw::Ticks, true );
            scaleDraw->enableComponent( QwtAbstractScaleDraw::Labels, true );
            scaleDraw->enableComponent( QwtAbstractScaleDraw::Backbone, false );
            scaleDraw->setTickLength( QwtScaleDiv::MinorTick, 1 );
            scaleDraw->setTickLength( QwtScaleDiv::MediumTick, 1 );
            scaleDraw->setTickLength( QwtScaleDiv::MajorTick, 3 );

            compass->setScaleDraw( scaleDraw );

            compass->setScaleMaxMajor( 36 );
            compass->setScaleMaxMinor( 5 );

            compass->setNeedle(
                new QwtCompassMagnetNeedle( QwtCompassMagnetNeedle::ThinStyle ) );
            compass->setValue( 220.0 );

            break;
        }
        case 3:
        {
            /*
              A compass without a frame, showing numbers as tick labels.
              The origin is at 220.0
             */
            palette0.setColor( QPalette::Base,
                palette().color( backgroundRole() ) );
            palette0.setColor( QPalette::WindowText, Qt::blue );

            compass->setLineWidth( 0 );

            QMap<double, QString> map;
            for ( double d = 0.0; d < 360.0; d += 60.0 )
            {
                QString label;
                label.sprintf( "%.0f", d );
                map.insert( d, label );
            }

            QwtCompassScaleDraw *scaleDraw = 
                new QwtCompassScaleDraw( map );
            scaleDraw->enableComponent( QwtAbstractScaleDraw::Ticks, true );
            scaleDraw->enableComponent( QwtAbstractScaleDraw::Labels, true );
            scaleDraw->enableComponent( QwtAbstractScaleDraw::Backbone, true );
            scaleDraw->setTickLength( QwtScaleDiv::MinorTick, 0 );
            scaleDraw->setTickLength( QwtScaleDiv::MediumTick, 0 );
            scaleDraw->setTickLength( QwtScaleDiv::MajorTick, 3 );

            compass->setScaleDraw( scaleDraw );

            compass->setScaleMaxMajor( 36 );
            compass->setScaleMaxMinor( 5 );

            compass->setNeedle( new QwtDialSimpleNeedle( QwtDialSimpleNeedle::Ray,
                true, Qt::white ) );
            compass->setOrigin( 220.0 );
            compass->setValue( 20.0 );
            break;
        }
        case 4:
        {
            /*
             A compass showing another needle
             */
            QwtCompassScaleDraw *scaleDraw = new QwtCompassScaleDraw();
            scaleDraw->enableComponent( QwtAbstractScaleDraw::Ticks, true );
            scaleDraw->enableComponent( QwtAbstractScaleDraw::Labels, true );
            scaleDraw->enableComponent( QwtAbstractScaleDraw::Backbone, false );
            scaleDraw->setTickLength( QwtScaleDiv::MinorTick, 0 );
            scaleDraw->setTickLength( QwtScaleDiv::MediumTick, 0 );
            scaleDraw->setTickLength( QwtScaleDiv::MajorTick, 3 );

            compass->setScaleDraw( scaleDraw );

            compass->setNeedle( new QwtCompassMagnetNeedle(
                QwtCompassMagnetNeedle::TriangleStyle, Qt::white, Qt::red ) );
            compass->setValue( 220.0 );
            break;
        }
        case 5:
        {
            /*
             A compass with a yellow on black ray
             */
            palette0.setColor( QPalette::WindowText, Qt::black );

            compass->setNeedle( new QwtDialSimpleNeedle( QwtDialSimpleNeedle::Ray,
                false, Qt::yellow ) );
            compass->setValue( 315.0 );
            break;
        }
    }

    QPalette newPalette = compass->palette();
    for ( c = 0; c < QPalette::NColorRoles; c++ )
    {
        const QPalette::ColorRole colorRole =
            static_cast<QPalette::ColorRole>( c );

        if ( palette0.color( colorRole ).isValid() )
            newPalette.setColor( colorRole, palette0.color( colorRole ) );
    }

    for ( int i = 0; i < QPalette::NColorGroups; i++ )
    {
        const QPalette::ColorGroup colorGroup =
            static_cast<QPalette::ColorGroup>( i );

        const QColor light =
            newPalette.color( colorGroup, QPalette::Base ).light( 170 );
        const QColor dark = newPalette.color( colorGroup, QPalette::Base ).dark( 170 );
        const QColor mid = compass->frameShadow() == QwtDial::Raised
            ? newPalette.color( colorGroup, QPalette::Base ).dark( 110 )
            : newPalette.color( colorGroup, QPalette::Base ).light( 110 );

        newPalette.setColor( colorGroup, QPalette::Dark, dark );
        newPalette.setColor( colorGroup, QPalette::Mid, mid );
        newPalette.setColor( colorGroup, QPalette::Light, light );
    }

    compass->setPalette( newPalette );

    return compass;
}
```

### `examples/dials/compass_grid.h`

```cpp
#include <qframe.h>
class QwtCompass;

class CompassGrid: public QFrame
{
public:
    CompassGrid( QWidget *parent = NULL );

private:
    QwtCompass *createCompass( int pos );
};
```

### `examples/dials/dials.cpp`

```cpp
#include <qapplication.h>
#include <qtabwidget.h>
#include "compass_grid.h"
#include "cockpit_grid.h"

//-----------------------------------------------------------------
//
//      dials.cpp -- A demo program featuring QwtDial and friends
//
//-----------------------------------------------------------------

int main ( int argc, char **argv )
{
    QApplication a( argc, argv );

    QTabWidget tabWidget;
    tabWidget.addTab( new CompassGrid, "Compass" );
    tabWidget.addTab( new CockpitGrid, "Cockpit" );

    tabWidget.show();

    return a.exec();
}
```

### `examples/dials/speedo_meter.cpp`

```cpp
#include <qpainter.h>
#include <qwt_dial_needle.h>
#include <qwt_round_scale_draw.h>
#include "speedo_meter.h"

SpeedoMeter::SpeedoMeter( QWidget *parent ):
    QwtDial( parent ),
    d_label( "km/h" )
{
    QwtRoundScaleDraw *scaleDraw = new QwtRoundScaleDraw();
    scaleDraw->setSpacing( 8 );
    scaleDraw->enableComponent( QwtAbstractScaleDraw::Backbone, false );
    scaleDraw->setTickLength( QwtScaleDiv::MinorTick, 0 );
    scaleDraw->setTickLength( QwtScaleDiv::MediumTick, 4 );
    scaleDraw->setTickLength( QwtScaleDiv::MajorTick, 8 );
    setScaleDraw( scaleDraw );

    setWrapping( false );
    setReadOnly( true );

    setOrigin( 135.0 );
    setScaleArc( 0.0, 270.0 );

    QwtDialSimpleNeedle *needle = new QwtDialSimpleNeedle(
        QwtDialSimpleNeedle::Arrow, true, Qt::red,
        QColor( Qt::gray ).light( 130 ) );
    setNeedle( needle );
}

void SpeedoMeter::setLabel( const QString &label )
{
    d_label = label;
    update();
}

QString SpeedoMeter::label() const
{
    return d_label;
}

void SpeedoMeter::drawScaleContents( QPainter *painter,
    const QPointF &center, double radius ) const
{
    QRectF rect( 0.0, 0.0, 2.0 * radius, 2.0 * radius - 10.0 );
    rect.moveCenter( center );

    const QColor color = palette().color( QPalette::Text );
    painter->setPen( color );

    const int flags = Qt::AlignBottom | Qt::AlignHCenter;
    painter->drawText( rect, flags, d_label );
}
```

### `examples/dials/speedo_meter.h`

```cpp
#include <qstring.h>
#include <qwt_dial.h>

class SpeedoMeter: public QwtDial
{
public:
    SpeedoMeter( QWidget *parent = NULL );

    void setLabel( const QString & );
    QString label() const;

protected:
    virtual void drawScaleContents( QPainter *painter,
        const QPointF &center, double radius ) const;

private:
    QString d_label;
};
```

### `examples/distrowatch/barchart.cpp`

```cpp
#include "barchart.h"
#include <qwt_plot_renderer.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_barchart.h>
#include <qwt_column_symbol.h>
#include <qwt_plot_layout.h>
#include <qwt_legend.h>
#include <qwt_scale_draw.h>

class DistroScaleDraw: public QwtScaleDraw
{
public:
    DistroScaleDraw( Qt::Orientation orientation, const QStringList &labels ):
        d_labels( labels )
    {
        setTickLength( QwtScaleDiv::MinorTick, 0 );
        setTickLength( QwtScaleDiv::MediumTick, 0 );
        setTickLength( QwtScaleDiv::MajorTick, 2 );

        enableComponent( QwtScaleDraw::Backbone, false );

        if ( orientation == Qt::Vertical )
        {
            setLabelRotation( -60.0 );
        }
        else
        {
            setLabelRotation( -20.0 );
        }

        setLabelAlignment( Qt::AlignLeft | Qt::AlignVCenter );
    }

    virtual QwtText label( double value ) const
    {
        QwtText lbl;

        const int index = qRound( value );
        if ( index >= 0 && index < d_labels.size() )
        {
            lbl = d_labels[ index ];
        }
            
        return lbl;
    }

private:
    const QStringList d_labels;
};

class DistroChartItem: public QwtPlotBarChart
{
public:
    DistroChartItem():
        QwtPlotBarChart( "Page Hits" )
    {
        setLegendMode( QwtPlotBarChart::LegendBarTitles );
        setLegendIconSize( QSize( 10, 14 ) );
        setLayoutPolicy( AutoAdjustSamples );
        setLayoutHint( 4.0 ); // minimum width for a single bar

        setSpacing( 10 ); // spacing between bars
    }

    void addDistro( const QString &distro, const QColor &color )
    {
        d_colors += color;
        d_distros += distro;
        itemChanged();
    }

    virtual QwtColumnSymbol *specialSymbol(
        int index, const QPointF& ) const
    {
        // we want to have individual colors for each bar

        QwtColumnSymbol *symbol = new QwtColumnSymbol( QwtColumnSymbol::Box );
        symbol->setLineWidth( 2 );
        symbol->setFrameStyle( QwtColumnSymbol::Raised );

        QColor c( Qt::white );
        if ( index >= 0 && index < d_colors.size() )
            c = d_colors[ index ];

        symbol->setPalette( c );
    
        return symbol;
    }

    virtual QwtText barTitle( int sampleIndex ) const
    {
        QwtText title;
        if ( sampleIndex >= 0 && sampleIndex < d_distros.size() )
            title = d_distros[ sampleIndex ];

        return title;
    }

private:
    QList<QColor> d_colors;
    QList<QString> d_distros;
};

BarChart::BarChart( QWidget *parent ):
    QwtPlot( parent )
{
    const struct 
    {
        const char *distro;
        const int hits;
        QColor color;

    } pageHits[] =
    {
        { "Arch", 1114, QColor( "DodgerBlue" ) },
        { "Debian", 1373, QColor( "#d70751" ) },
        { "Fedora", 1638, QColor( "SteelBlue" ) },
        { "Mageia", 1395, QColor( "Indigo" ) },
        { "Mint", 3874, QColor( 183, 255, 183 ) },
        { "openSuSE", 1532, QColor( 115, 186, 37 ) },
        { "Puppy", 1059, QColor( "LightSkyBlue" ) },
        { "Ubuntu", 2391, QColor( "FireBrick" ) }
    };

    setAutoFillBackground( true );
    setPalette( QColor( "Linen" ) );

    QwtPlotCanvas *canvas = new QwtPlotCanvas();
    canvas->setLineWidth( 2 );
    canvas->setFrameStyle( QFrame::Box | QFrame::Sunken );
    canvas->setBorderRadius( 10 );

    QPalette canvasPalette( QColor( "Plum" ) );
    canvasPalette.setColor( QPalette::Foreground, QColor( "Indigo" ) );
    canvas->setPalette( canvasPalette );

    setCanvas( canvas );

    setTitle( "DistroWatch Page Hit Ranking, April 2012" );

    d_barChartItem = new DistroChartItem();

    QVector< double > samples;

    for ( uint i = 0; i < sizeof( pageHits ) / sizeof( pageHits[ 0 ] ); i++ )
    {
        d_distros += pageHits[ i ].distro;
        samples += pageHits[ i ].hits;

        d_barChartItem->addDistro( 
            pageHits[ i ].distro, pageHits[ i ].color );
    }

    d_barChartItem->setSamples( samples );

    d_barChartItem->attach( this );

    insertLegend( new QwtLegend() );

    setOrientation( 0 );
    setAutoReplot( false );
}

void BarChart::setOrientation( int o )
{
    const Qt::Orientation orientation =
        ( o == 0 ) ? Qt::Vertical : Qt::Horizontal;

    int axis1 = QwtPlot::xBottom;
    int axis2 = QwtPlot::yLeft;

    if ( orientation == Qt::Horizontal )
        qSwap( axis1, axis2 );

    d_barChartItem->setOrientation( orientation );

    setAxisTitle( axis1, "Distros" );
    setAxisMaxMinor( axis1, 3 );
    setAxisScaleDraw( axis1, new DistroScaleDraw( orientation, d_distros ) );

    setAxisTitle( axis2, "Hits per day ( HPD )" );
    setAxisMaxMinor( axis2, 3 );

    QwtScaleDraw *scaleDraw = new QwtScaleDraw();
    scaleDraw->setTickLength( QwtScaleDiv::MediumTick, 4 );
    setAxisScaleDraw( axis2, scaleDraw );

    plotLayout()->setCanvasMargin( 0 );
    replot();
}

void BarChart::exportChart()
{
    QwtPlotRenderer renderer;
    renderer.exportTo( this, "distrowatch.pdf" );
}
```

### `examples/distrowatch/barchart.h`

```cpp
#ifndef _BAR_CHART_H_

#include <qwt_plot.h>
#include <qstringlist.h>

class DistroChartItem;

class BarChart: public QwtPlot
{
    Q_OBJECT

public:
    BarChart( QWidget * = NULL );

public Q_SLOTS:
    void setOrientation( int );
    void exportChart();

private:
    void populate();

    DistroChartItem *d_barChartItem;
    QStringList d_distros;
};

#endif
```

### `examples/distrowatch/main.cpp`

```cpp
#include <qapplication.h>
#include <qmainwindow.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include <qcombobox.h>
#include "barchart.h"

class MainWindow: public QMainWindow
{
public:
    MainWindow( QWidget * = NULL );

private:
    BarChart *d_chart;
};

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    d_chart = new BarChart( this );
    setCentralWidget( d_chart );

    QToolBar *toolBar = new QToolBar( this );

    QComboBox *orientationBox = new QComboBox( toolBar );
    orientationBox->addItem( "Vertical" );
    orientationBox->addItem( "Horizontal" );
    orientationBox->setSizePolicy( QSizePolicy::Fixed, QSizePolicy::Fixed );

    QToolButton *btnExport = new QToolButton( toolBar );
    btnExport->setText( "Export" );
    btnExport->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    connect( btnExport, SIGNAL( clicked() ), d_chart, SLOT( exportChart() ) );

    toolBar->addWidget( orientationBox );
    toolBar->addWidget( btnExport );
    addToolBar( toolBar );

    d_chart->setOrientation( orientationBox->currentIndex() );
    connect( orientationBox, SIGNAL( currentIndexChanged( int ) ),
             d_chart, SLOT( setOrientation( int ) ) );
}

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWindow mainWindow;

    mainWindow.resize( 600, 400 );
    mainWindow.show();

    return a.exec();
}
```

### `examples/event_filter/canvaspicker.cpp`

```cpp

```

### `examples/event_filter/canvaspicker.h`

```cpp
#include <qobject.h>

class QPoint;
class QCustomEvent;
class QwtPlot;
class QwtPlotCurve;

class CanvasPicker: public QObject
{
    Q_OBJECT
public:
    CanvasPicker( QwtPlot *plot );
    virtual bool eventFilter( QObject *, QEvent * );

    virtual bool event( QEvent * );

private:
    void select( const QPoint & );
    void move( const QPoint & );
    void moveBy( int dx, int dy );

    void release();

    void showCursor( bool enable );
    void shiftPointCursor( bool up );
    void shiftCurveCursor( bool up );

    QwtPlot *plot();
    const QwtPlot *plot() const;

    QwtPlotCurve *d_selectedCurve;
    int d_selectedPoint;
};
```

### `examples/event_filter/colorbar.cpp`

```cpp
#include <qevent.h>
#include <qpixmap.h>
#include <qimage.h>
#include <qpainter.h>
#include "colorbar.h"

ColorBar::ColorBar( Qt::Orientation o, QWidget *parent ):
    QWidget( parent ),
    d_orientation( o ),
    d_light( Qt::white ),
    d_dark( Qt::black )
{
#ifndef QT_NO_CURSOR
    setCursor( Qt::PointingHandCursor );
#endif
}

void ColorBar::setOrientation( Qt::Orientation o )
{
    d_orientation = o;
    update();
}

void ColorBar::setLight( const QColor &light )
{
    d_light = light;
    update();
}

void ColorBar::setDark( const QColor &dark )
{
    d_dark = dark;
    update();
}

void ColorBar::setRange( const QColor &light, const QColor &dark )
{
    d_light = light;
    d_dark = dark;
    update();
}

void ColorBar::mousePressEvent( QMouseEvent *e )
{
    if( e->button() ==  Qt::LeftButton )
    {
        // emit the color of the position where the mouse click
        // happened

        const QPixmap pm = QPixmap::grabWidget( this );
        const QRgb rgb = pm.toImage().pixel( e->x(), e->y() );

        Q_EMIT selected( QColor( rgb ) );
        e->accept();
    }
}

void ColorBar::paintEvent( QPaintEvent * )
{
    QPainter painter( this );
    drawColorBar( &painter, rect() );
}

void ColorBar::drawColorBar( QPainter *painter, const QRect &rect ) const
{
    int h1, s1, v1;
    int h2, s2, v2;

    d_light.getHsv( &h1, &s1, &v1 );
    d_dark.getHsv( &h2, &s2, &v2 );

    painter->save();
    painter->setClipRect( rect );
    painter->setClipping( true );

    painter->fillRect( rect, d_dark );

    const int sectionSize = 2;

    int numIntervals;
    if ( d_orientation == Qt::Horizontal )
        numIntervals = rect.width() / sectionSize;
    else
        numIntervals = rect.height() / sectionSize;

    for ( int i = 0; i < numIntervals; i++ )
    {
        QRect section;
        if ( d_orientation == Qt::Horizontal )
        {
            section.setRect( rect.x() + i * sectionSize, rect.y(),
                sectionSize, rect.height() );
        }
        else
        {
            section.setRect( rect.x(), rect.y() + i * sectionSize,
                rect.width(), sectionSize );
        }

        const double ratio = i / static_cast<double>( numIntervals );

        QColor c;
        c.setHsv( h1 + qRound( ratio * ( h2 - h1 ) ),
            s1 + qRound( ratio * ( s2 - s1 ) ),
            v1 + qRound( ratio * ( v2 - v1 ) ) );

        painter->fillRect( section, c );
    }

    painter->restore();
}
```

### `examples/event_filter/colorbar.h`

```cpp
#include <qwidget.h>

class ColorBar: public QWidget
{
    Q_OBJECT

public:
    ColorBar( Qt::Orientation = Qt::Horizontal, QWidget * = NULL );

    virtual void setOrientation( Qt::Orientation );
    Qt::Orientation orientation() const { return d_orientation; }

    void setRange( const QColor &light, const QColor &dark );
    void setLight( const QColor &light );
    void setDark( const QColor &dark );

    QColor light() const { return d_light; }
    QColor dark() const { return d_dark; }

Q_SIGNALS:
    void selected( const QColor & );

protected:
    virtual void mousePressEvent( QMouseEvent * );
    virtual void paintEvent( QPaintEvent * );

    void drawColorBar( QPainter *, const QRect & ) const;

private:
    Qt::Orientation d_orientation;
    QColor d_light;
    QColor d_dark;
};
```

### `examples/event_filter/event_filter.cpp`

```cpp
//-----------------------------------------------------------------
//      A demo program showing how to use event filtering
//-----------------------------------------------------------------

#include <qapplication.h>
#include <qmainwindow.h>
#include <qwhatsthis.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include "plot.h"
#include "canvaspicker.h"
#include "scalepicker.h"

int main ( int argc, char **argv )
{
    QApplication a( argc, argv );

    QMainWindow mainWindow;
    QToolBar *toolBar = new QToolBar( &mainWindow );
    QAction *action = QWhatsThis::createAction( toolBar );
    toolBar->addAction( action );
    mainWindow.addToolBar( toolBar );

    Plot *plot = new Plot( &mainWindow );

    // The canvas picker handles all mouse and key
    // events on the plot canvas

    ( void ) new CanvasPicker( plot );

    // The scale picker translates mouse clicks
    // int o clicked() signals

    ScalePicker *scalePicker = new ScalePicker( plot );
    a.connect( scalePicker, SIGNAL( clicked( int, double ) ),
        plot, SLOT( insertCurve( int, double ) ) );

    mainWindow.setCentralWidget( plot );

    mainWindow.resize( 540, 400 );
    mainWindow.show();

    const char *text =
        "An useless plot to demonstrate how to use event filtering.\n\n"
        "You can click on the color bar, the scales or move the wheel.\n"
        "All points can be moved using the mouse or the keyboard.";
    plot->setWhatsThis( text );

    int rv = a.exec();
    return rv;
}
```

### `examples/event_filter/plot.cpp`

```cpp
#include "plot.h"
#include "colorbar.h"
#include <qevent.h>
#include <qwt_plot_layout.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_grid.h>
#include <qwt_plot_curve.h>
#include <qwt_symbol.h>
#include <qwt_scale_widget.h>
#include <qwt_wheel.h>
#include <stdlib.h>

Plot::Plot( QWidget *parent ):
    QwtPlot( parent )
{
    setTitle( "Interactive Plot" );

    setCanvasColor( Qt::darkCyan );

    QwtPlotGrid *grid = new QwtPlotGrid;
    grid->setMajorPen( Qt::white, 0, Qt::DotLine );
    grid->attach( this );

    // axes

    setAxisScale( QwtPlot::xBottom, 0.0, 100.0 );
    setAxisScale( QwtPlot::yLeft, 0.0, 100.0 );

    // Avoid jumping when label with 3 digits
    // appear/disappear when scrolling vertically

    QwtScaleDraw *sd = axisScaleDraw( QwtPlot::yLeft );
    sd->setMinimumExtent( sd->extent( axisWidget( QwtPlot::yLeft )->font() ) );

    plotLayout()->setAlignCanvasToScales( true );

    insertCurve( Qt::Vertical, Qt::blue, 30.0 );
    insertCurve( Qt::Vertical, Qt::magenta, 70.0 );
    insertCurve( Qt::Horizontal, Qt::yellow, 30.0 );
    insertCurve( Qt::Horizontal, Qt::white, 70.0 );

    replot();

    // ------------------------------------
    // We add a color bar to the left axis
    // ------------------------------------

    QwtScaleWidget *scaleWidget = axisWidget( yLeft );
    scaleWidget->setMargin( 10 ); // area for the color bar
    d_colorBar = new ColorBar( Qt::Vertical, scaleWidget );
    d_colorBar->setRange( Qt::red, Qt::darkBlue );
    d_colorBar->setFocusPolicy( Qt::TabFocus );

    connect( d_colorBar, SIGNAL( selected( const QColor & ) ),
        SLOT( setCanvasColor( const QColor & ) ) );

    // we need the resize events, to lay out the color bar
    scaleWidget->installEventFilter( this );

    // ------------------------------------
    // We add a wheel to the canvas
    // ------------------------------------

    d_wheel = new QwtWheel( canvas() );
    d_wheel->setOrientation( Qt::Vertical );
    d_wheel->setRange( -100, 100 );
    d_wheel->setValue( 0.0 );
    d_wheel->setMass( 0.2 );
    d_wheel->setTotalAngle( 4 * 360.0 );

    connect( d_wheel, SIGNAL( valueChanged( double ) ),
        SLOT( scrollLeftAxis( double ) ) );

    // we need the resize events, to lay out the wheel
    canvas()->installEventFilter( this );

    d_colorBar->setWhatsThis(
        "Selecting a color will change the background of the plot." );
    scaleWidget->setWhatsThis(
        "Selecting a value at the scale will insert a new curve." );
    d_wheel->setWhatsThis(
        "With the wheel you can move the visible area." );
    axisWidget( xBottom )->setWhatsThis(
        "Selecting a value at the scale will insert a new curve." );
}

void Plot::setCanvasColor( const QColor &c )
{
    setCanvasBackground( c );
    replot();
}

void Plot::scrollLeftAxis( double value )
{
    setAxisScale( yLeft, value, value + 100.0 );
    replot();
}

bool Plot::eventFilter( QObject *object, QEvent *e )
{
    if ( e->type() == QEvent::Resize )
    {
        const QSize size = static_cast<QResizeEvent *>( e )->size();
        if ( object == axisWidget( yLeft ) )
        {
            const QwtScaleWidget *scaleWidget = axisWidget( yLeft );

            const int margin = 2;

            // adjust the color bar to the scale backbone
            const int x = size.width() - scaleWidget->margin() + margin;
            const int w = scaleWidget->margin() - 2 * margin;
            const int y = scaleWidget->startBorderDist();
            const int h = size.height() -
                scaleWidget->startBorderDist() - scaleWidget->endBorderDist();

            d_colorBar->setGeometry( x, y, w, h );
        }
        if ( object == canvas() )
        {
            const int w = 16;
            const int h = 50;
            const int margin = 2;

            const QRect cr = canvas()->contentsRect();
            d_wheel->setGeometry(
                cr.right() - margin - w, cr.center().y() - h / 2, w, h );
        }
    }

    return QwtPlot::eventFilter( object, e );
}

void Plot::insertCurve( int axis, double base )
{
    Qt::Orientation o;
    if ( axis == yLeft || axis == yRight )
        o = Qt::Horizontal;
    else
        o = Qt::Vertical;

    QRgb rgb = static_cast<QRgb>( rand() );
    insertCurve( o, QColor( rgb ), base );
    replot();
}

void Plot::insertCurve( Qt::Orientation o,
    const QColor &c, double base )
{
    QwtPlotCurve *curve = new QwtPlotCurve();

    curve->setPen( c );
    curve->setSymbol( new QwtSymbol( QwtSymbol::Ellipse,
        Qt::gray, c, QSize( 8, 8 ) ) );

    double x[10];
    double y[sizeof( x ) / sizeof( x[0] )];

    for ( uint i = 0; i < sizeof( x ) / sizeof( x[0] ); i++ )
    {
        double v = 5.0 + i * 10.0;
        if ( o == Qt::Horizontal )
        {
            x[i] = v;
            y[i] = base;
        }
        else
        {
            x[i] = base;
            y[i] = v;
        }
    }

    curve->setSamples( x, y, sizeof( x ) / sizeof( x[0] ) );
    curve->attach( this );
}
```

### `examples/event_filter/plot.h`

```cpp
#include <qwt_plot.h>

class ColorBar;
class QwtWheel;

class Plot: public QwtPlot
{
    Q_OBJECT
public:
    Plot( QWidget *parent = NULL );
    virtual bool eventFilter( QObject *, QEvent * );

public Q_SLOTS:
    void setCanvasColor( const QColor & );
    void insertCurve( int axis, double base );

private Q_SLOTS:
    void scrollLeftAxis( double );

private:
    void insertCurve( Qt::Orientation, const QColor &, double base );

    ColorBar *d_colorBar;
    QwtWheel *d_wheel;
};
```

### `examples/event_filter/scalepicker.cpp`

```cpp
#include "scalepicker.h"
#include <qwt_plot.h>
#include <qwt_scale_widget.h>
#include <qevent.h>
#include <qmath.h>

ScalePicker::ScalePicker( QwtPlot *plot ):
    QObject( plot )
{
    for ( uint i = 0; i < QwtPlot::axisCnt; i++ )
    {
        QwtScaleWidget *scaleWidget = plot->axisWidget( i );
        if ( scaleWidget )
            scaleWidget->installEventFilter( this );
    }
}

bool ScalePicker::eventFilter( QObject *object, QEvent *event )
{
    if ( event->type() == QEvent::MouseButtonPress )
    {
        QwtScaleWidget *scaleWidget = qobject_cast<QwtScaleWidget *>( object );
        if ( scaleWidget )
        {
            QMouseEvent *mouseEvent = static_cast<QMouseEvent *>( event );
            mouseClicked( scaleWidget, mouseEvent->pos() );

            return true;
        }
    }

    return QObject::eventFilter( object, event );
}

void ScalePicker::mouseClicked( const QwtScaleWidget *scale, const QPoint &pos )
{
    QRect rect = scaleRect( scale );

    int margin = 10; // 10 pixels tolerance
    rect.setRect( rect.x() - margin, rect.y() - margin,
        rect.width() + 2 * margin, rect.height() +  2 * margin );

    if ( rect.contains( pos ) ) // No click on the title
    {
        // translate the position in a value on the scale

        double value = 0.0;
        int axis = -1;

        const QwtScaleDraw *sd = scale->scaleDraw();
        switch( scale->alignment() )
        {
            case QwtScaleDraw::LeftScale:
            {
                value = sd->scaleMap().invTransform( pos.y() );
                axis = QwtPlot::yLeft;
                break;
            }
            case QwtScaleDraw::RightScale:
            {
                value = sd->scaleMap().invTransform( pos.y() );
                axis = QwtPlot::yRight;
                break;
            }
            case QwtScaleDraw::BottomScale:
            {
                value = sd->scaleMap().invTransform( pos.x() );
                axis = QwtPlot::xBottom;
                break;
            }
            case QwtScaleDraw::TopScale:
            {
                value = sd->scaleMap().invTransform( pos.x() );
                axis = QwtPlot::xTop;
                break;
            }
        }
        Q_EMIT clicked( axis, value );
    }
}

// The rect of a scale without the title
QRect ScalePicker::scaleRect( const QwtScaleWidget *scale ) const
{
    const int bld = scale->margin();
    const int mjt = qCeil( scale->scaleDraw()->maxTickLength() );
    const int sbd = scale->startBorderDist();
    const int ebd = scale->endBorderDist();

    QRect rect;
    switch( scale->alignment() )
    {
        case QwtScaleDraw::LeftScale:
        {
            rect.setRect( scale->width() - bld - mjt, sbd,
                mjt, scale->height() - sbd - ebd );
            break;
        }
        case QwtScaleDraw::RightScale:
        {
            rect.setRect( bld, sbd,
                mjt, scale->height() - sbd - ebd );
            break;
        }
        case QwtScaleDraw::BottomScale:
        {
            rect.setRect( sbd, bld,
                scale->width() - sbd - ebd, mjt );
            break;
        }
        case QwtScaleDraw::TopScale:
        {
            rect.setRect( sbd, scale->height() - bld - mjt,
                scale->width() - sbd - ebd, mjt );
            break;
        }
    }
    return rect;
}
```

### `examples/event_filter/scalepicker.h`

```cpp
#include <qobject.h>
#include <qrect.h>

class QwtPlot;
class QwtScaleWidget;

class ScalePicker: public QObject
{
    Q_OBJECT
public:
    ScalePicker( QwtPlot *plot );
    virtual bool eventFilter( QObject *, QEvent * );

Q_SIGNALS:
    void clicked( int axis, double value );

private:
    void mouseClicked( const QwtScaleWidget *, const QPoint & );
    QRect scaleRect( const QwtScaleWidget * ) const;
};
```

### `examples/friedberg/friedberg2007.cpp`

```cpp
#include "friedberg2007.h"

// Temperature 2007 from Friedberg somewhere in Germany
// See: http://wetter61169.de

Temperature friedberg2007[] =
{
    /* 01.01 */ Temperature( 2.6, 9.8, 7.07862 ),
    /* 02.01 */ Temperature( 0.8, 5.8, 3.6993 ),
    /* 03.01 */ Temperature( 2, 7, 5.02388 ),
    /* 04.01 */ Temperature( 5.3, 7.8, 6.37778 ),
    /* 05.01 */ Temperature( 5.6, 7.7, 6.83149 ),
    /* 06.01 */ Temperature( 7.2, 8.9, 8.0816 ),
    /* 07.01 */ Temperature( 4.2, 9.9, 7.54704 ),
    /* 08.01 */ Temperature( 3.5, 8.9, 6.71951 ),
    /* 09.01 */ Temperature( 8.2, 12.9, 10.8594 ),
    /* 10.01 */ Temperature( 6.3, 11.9, 9.76424 ),
    /* 11.01 */ Temperature( 3.9, 9.2, 6.18223 ),
    /* 12.01 */ Temperature( 6.9, 9.7, 8.44236 ),
    /* 13.01 */ Temperature( 9, 12.3, 10.6649 ),
    /* 14.01 */ Temperature( 1.8, 10.8, 7.23438 ),
    /* 15.01 */ Temperature( -2.8, 1.8, -0.518403 ),
    /* 16.01 */ Temperature( -0.6, 4.5, 2.39479 ),
    /* 17.01 */ Temperature( 4.3, 10.2, 7.23472 ),
    /* 18.01 */ Temperature( 9.1, 13.6, 10.9316 ),
    /* 19.01 */ Temperature( 6.9, 12.4, 9.4128 ),
    /* 20.01 */ Temperature( 7.1, 13.3, 10.5083 ),
    /* 21.01 */ Temperature( 3.5, 9.6, 6.10871 ),
    /* 22.01 */ Temperature( -1.8, 6, 2.89028 ),
    /* 23.01 */ Temperature( -5.4, 1.7, -2.46678 ),
    /* 24.01 */ Temperature( -5.3, -1.3, -3.71483 ),
    /* 25.01 */ Temperature( -7.5, 3.3, -3.36736 ),
    /* 26.01 */ Temperature( -11.1, 0.3, -5.50662 ),
    /* 27.01 */ Temperature( 0.2, 3.2, 1.95345 ),
    /* 28.01 */ Temperature( 1.9, 5.2, 3.43633 ),
    /* 29.01 */ Temperature( 4.4, 9.1, 6.24236 ),
    /* 30.01 */ Temperature( 2.3, 11.5, 6.03114 ),
    /* 31.01 */ Temperature( 4.6, 10.2, 6.04192 ),

    /* 01.02 */ Temperature( 4.8, 13.8, 7.87674 ),
    /* 02.02 */ Temperature( 5.7, 10, 7.28646 ),
    /* 03.02 */ Temperature( 2.9, 8.2, 5.71771 ),
    /* 04.02 */ Temperature( -1.5, 7.2, 4.71319 ),
    /* 05.02 */ Temperature( -2.6, 4.4, 1.23542 ),
    /* 06.02 */ Temperature( 0.3, 9.2, 2.59965 ),
    /* 07.02 */ Temperature( -0.4, 2.4, 0.641667 ),
    /* 08.02 */ Temperature( -1.7, 3.8, 0.811458 ),
    /* 09.02 */ Temperature( 0.7, 7, 3.58328 ),
    /* 10.02 */ Temperature( 1, 6, 3.51181 ),
    /* 11.02 */ Temperature( 4.7, 9.6, 6.14913 ),
    /* 12.02 */ Temperature( 5.3, 8.7, 6.80552 ),
    /* 13.02 */ Temperature( 4.4, 10.3, 6.84552 ),
    /* 14.02 */ Temperature( 2.6, 6.5, 4.58681 ),
    /* 15.02 */ Temperature( -0.8, 13.4, 6.38542 ),
    /* 16.02 */ Temperature( -3, 14.4, 4.11336 ),
    /* 17.02 */ Temperature( 0.5, 13, 5.87457 ),
    /* 18.02 */ Temperature( -2.2, 14.1, 4.36528 ),
    /* 19.02 */ Temperature( 3.9, 5.6, 4.63737 ),
    /* 20.02 */ Temperature( -0.4, 9.2, 4.37014 ),
    /* 21.02 */ Temperature( -1.9, 5.5, 1.85675 ),
    /* 22.02 */ Temperature( 1, 13.1, 5.41176 ),
    /* 23.02 */ Temperature( 1.9, 13.9, 7.74251 ),
    /* 24.02 */ Temperature( 3.8, 9.6, 7.19306 ),
    /* 25.02 */ Temperature( 5.8, 10.8, 7.80312 ),
    /* 26.02 */ Temperature( 5.2, 10.4, 6.79481 ),
    /* 27.02 */ Temperature( 3.2, 7.4, 5.22986 ),
    /* 28.02 */ Temperature( 6.4, 13.4, 9.13356 ),

    /* 01.03 */ Temperature( 4.6, 11.4, 7.70554 ),
    /* 02.03 */ Temperature( 3.4, 10.9, 5.98408 ),
    /* 03.03 */ Temperature( 2.9, 10.5, 5.45675 ),
    /* 04.03 */ Temperature( -0.7, 16.8, 7.29585 ),
    /* 05.03 */ Temperature( 4.2, 13.4, 8.35862 ),
    /* 06.03 */ Temperature( 3, 13, 7.76644 ),
    /* 07.03 */ Temperature( 2, 13.3, 8.24618 ),
    /* 08.03 */ Temperature( -0.8, 15, 6.11765 ),
    /* 09.03 */ Temperature( -0.7, 11, 5.7568 ),
    /* 10.03 */ Temperature( 1.2, 14.4, 6.61389 ),
    /* 11.03 */ Temperature( -1.7, 18, 6.66146 ),
    /* 12.03 */ Temperature( -0.6, 21.9, 8.9816 ),
    /* 13.03 */ Temperature( -0.9, 19.6, 9.08299 ),
    /* 14.03 */ Temperature( 5.3, 18.9, 10.5562 ),
    /* 15.03 */ Temperature( 2, 20.5, 9.65156 ),
    /* 16.03 */ Temperature( 0.2, 16.7, 7.8699 ),
    /* 17.03 */ Temperature( 4.5, 10.6, 7.87535 ),
    /* 18.03 */ Temperature( 2.7, 9.7, 6.71806 ),
    /* 19.03 */ Temperature( 0.4, 10.9, 3.92404 ),
    /* 20.03 */ Temperature( -2, 12.7, 4.01359 ),
    /* 21.03 */ Temperature( 0.3, 6.8, 3.00382 ),
    /* 22.03 */ Temperature( 0.9, 4.2, 2.2816 ),
    /* 23.03 */ Temperature( 2, 5.7, 3.39233 ),
    /* 24.03 */ Temperature( 3.9, 9.3, 6.41076 ),
    /* 25.03 */ Temperature( 4.2, 19.1, 9.92182 ),
    /* 26.03 */ Temperature( 2.3, 22, 12.5716 ),
    /* 27.03 */ Temperature( 4.9, 20.6, 13.4568 ),
    /* 28.03 */ Temperature( 0.3, 22.8, 10.755 ),
    /* 29.03 */ Temperature( 1.8, 17.2, 9.43924 ),
    /* 30.03 */ Temperature( 1.9, 19.8, 10.25 ),
    /* 31.03 */ Temperature( 6.7, 17, 11.1324 ),

    /* 01.04 */ Temperature( 5.7, 22, 12.8457 ),
    /* 02.04 */ Temperature( 6.4, 22.1, 13.3847 ),
    /* 03.04 */ Temperature( 5.8, 17.5, 10.5614 ),
    /* 04.04 */ Temperature( 2.8, 16.2, 8.06574 ),
    /* 05.04 */ Temperature( -0.6, 20.8, 9.18062 ),
    /* 06.04 */ Temperature( 2.1, 24, 13.0069 ),
    /* 07.04 */ Temperature( 5.3, 16.2, 10.2771 ),
    /* 08.04 */ Temperature( 0.1, 20.7, 9.79861 ),
    /* 09.04 */ Temperature( 0.3, 18.9, 10.0087 ),
    /* 10.04 */ Temperature( 4, 16.4, 11.4208 ),
    /* 11.04 */ Temperature( 2.3, 23.4, 13.083 ),
    /* 12.04 */ Temperature( 7, 29.4, 16.5826 ),
    /* 13.04 */ Temperature( 10.6, 31.5, 19.2249 ),
    /* 14.04 */ Temperature( 11.8, 34, 21.441 ),
    /* 15.04 */ Temperature( 11.6, 33.8, 21.0201 ),
    /* 16.04 */ Temperature( 8.7, 31.1, 18.7885 ),
    /* 17.04 */ Temperature( 5.5, 27.2, 16.1432 ),
    /* 18.04 */ Temperature( 6.1, 17.2, 10.6688 ),
    /* 19.04 */ Temperature( -0.6, 21.3, 10.4806 ),
    /* 20.04 */ Temperature( 5.9, 21.6, 12.6257 ),
    /* 21.04 */ Temperature( 2.1, 21.6, 11.0858 ),
    /* 22.04 */ Temperature( 3.9, 25.9, 14.2108 ),
    /* 23.04 */ Temperature( 3.1, 27.8, 15.7111 ),
    /* 24.04 */ Temperature( 13.7, 29, 19.6397 ),
    /* 25.04 */ Temperature( 9.8, 31.6, 19.601 ),
    /* 26.04 */ Temperature( 8.2, 32.4, 20.0389 ),
    /* 27.04 */ Temperature( 11.8, 32.1, 21.0726 ),
    /* 28.04 */ Temperature( 12.6, 33.3, 21.6993 ),
    /* 29.04 */ Temperature( 10.5, 27.4, 19.1206 ),
    /* 30.04 */ Temperature( 5.3, 26.4, 15.0972 ),

    /* 01.05 */ Temperature( 6.9, 25.3, 15.2802 ),
    /* 02.05 */ Temperature( 4.3, 26.2, 14.8401 ),
    /* 03.05 */ Temperature( 7.1, 28.5, 17.2145 ),
    /* 04.05 */ Temperature( 11, 28.5, 18.537 ),
    /* 05.05 */ Temperature( 12, 28, 18.1672 ),
    /* 06.05 */ Temperature( 10.4, 29, 18.3844 ),
    /* 07.05 */ Temperature( 13, 18.1, 15.0028 ),
    /* 08.05 */ Temperature( 10.7, 18.3, 13.2014 ),
    /* 09.05 */ Temperature( 10.8, 14.4, 12.5208 ),
    /* 10.05 */ Temperature( 11.9, 23.5, 16.9632 ),
    /* 11.05 */ Temperature( 9.8, 16.9, 15.0795 ),
    /* 12.05 */ Temperature( 9.2, 19.6, 13.8521 ),
    /* 13.05 */ Temperature( 8.9, 26.3, 16.2028 ),
    /* 14.05 */ Temperature( 11.1, 17.5, 13.2934 ),
    /* 15.05 */ Temperature( 6.5, 17, 11.7743 ),
    /* 16.05 */ Temperature( 4.9, 13.6, 9.75625 ),
    /* 17.05 */ Temperature( 6.8, 16.6, 9.96701 ),
    /* 18.05 */ Temperature( 2.4, 21.2, 11.4311 ),
    /* 19.05 */ Temperature( 8.2, 24.4, 15.4188 ),
    /* 20.05 */ Temperature( 14.1, 31.7, 21.3303 ),
    /* 21.05 */ Temperature( 11, 30.9, 21.5359 ),
    /* 22.05 */ Temperature( 13.8, 31, 21.5177 ),
    /* 23.05 */ Temperature( 16, 27.8, 21.0271 ),
    /* 24.05 */ Temperature( 15, 34, 23.4142 ),
    /* 25.05 */ Temperature( 14.3, 31.8, 22.8903 ),
    /* 26.05 */ Temperature( 13.6, 33.1, 22.6156 ),
    /* 27.05 */ Temperature( 11.2, 23.4, 16.6192 ),
    /* 28.05 */ Temperature( 9.6, 13.1, 11.3222 ),
    /* 29.05 */ Temperature( 8.3, 11.2, 10.3529 ),
    /* 30.05 */ Temperature( 4.2, 20.8, 12.6218 ),
    /* 31.05 */ Temperature( 9.2, 23.6, 15.1073 ),

    /* 01.06 */ Temperature( 10.8, 24.4, 16.3205 ),
    /* 02.06 */ Temperature( 13, 26.5, 18.9649 ),
    /* 03.06 */ Temperature( 14, 25.1, 18.5398 ),
    /* 04.06 */ Temperature( 13, 28, 20.2139 ),
    /* 05.06 */ Temperature( 14, 28.8, 20.438 ),
    /* 06.06 */ Temperature( 14, 30.4, 21.7821 ),
    /* 07.06 */ Temperature( 17, 34.8, 25.3087 ),
    /* 08.06 */ Temperature( 17.9, 35.7, 25.7872 ),
    /* 09.06 */ Temperature( 17.8, 31.6, 22.0788 ),
    /* 10.06 */ Temperature( 15.5, 33.4, 22.4458 ),
    /* 11.06 */ Temperature( 16.6, 28.3, 19.8797 ),
    /* 12.06 */ Temperature( 14, 27.3, 20.2566 ),
    /* 13.06 */ Temperature( 13.2, 28.2, 19.4233 ),
    /* 14.06 */ Temperature( 12.7, 30, 20.1427 ),
    /* 15.06 */ Temperature( 15.2, 22.6, 18.5917 ),
    /* 16.06 */ Temperature( 13.2, 24, 17.7014 ),
    /* 17.06 */ Temperature( 11.7, 27.9, 19.8229 ),
    /* 18.06 */ Temperature( 15.9, 27.2, 20.3358 ),
    /* 19.06 */ Temperature( 12.6, 33.7, 22.2427 ),
    /* 20.06 */ Temperature( 15.7, 30.8, 23.7507 ),
    /* 21.06 */ Temperature( 14.8, 22.6, 18.2538 ),
    /* 22.06 */ Temperature( 12.4, 21.3, 15.9969 ),
    /* 23.06 */ Temperature( 12.6, 21.6, 15.8149 ),
    /* 24.06 */ Temperature( 13, 26, 18.4176 ),
    /* 25.06 */ Temperature( 12.9, 24.4, 17.1299 ),
    /* 26.06 */ Temperature( 10.8, 18.8, 13.2913 ),
    /* 27.06 */ Temperature( 9.9, 18.8, 13.5465 ),
    /* 28.06 */ Temperature( 12, 19.8, 14.8434 ),
    /* 29.06 */ Temperature( 12, 19, 15.155 ),
    /* 30.06 */ Temperature( 12.4, 22.4, 17.1354 ),

    /* 01.07 */ Temperature( 12.1, 24.9, 19.1639 ),
    /* 02.07 */ Temperature( 15.7, 24.3, 18.4554 ),
    /* 03.07 */ Temperature( 12.7, 17.2, 14.6564 ),
    /* 04.07 */ Temperature( 11.2, 19, 13.9529 ),
    /* 05.07 */ Temperature( 11.5, 19, 14.6422 ),
    /* 06.07 */ Temperature( 12.4, 22, 16.6146 ),
    /* 07.07 */ Temperature( 11.6, 24, 17.666 ),
    /* 08.07 */ Temperature( 9, 28, 19.1351 ),
    /* 09.07 */ Temperature( 11.3, 21.5, 16.5271 ),
    /* 10.07 */ Temperature( 11.3, 20.2, 14.2326 ),
    /* 11.07 */ Temperature( 10.2, 19.2, 14.0649 ),
    /* 12.07 */ Temperature( 13.2, 23.1, 16.6346 ),
    /* 13.07 */ Temperature( 15, 27, 19.6844 ),
    /* 14.07 */ Temperature( 13.4, 32.4, 23.845 ),
    /* 15.07 */ Temperature( 15, 38.2, 26.8559 ),
    /* 16.07 */ Temperature( 16.1, 36.5, 26.4483 ),
    /* 17.07 */ Temperature( 19.7, 30.5, 24.189 ),
    /* 18.07 */ Temperature( 14.2, 29.3, 22.1363 ),
    /* 19.07 */ Temperature( 16.4, 25.9, 19.0819 ),
    /* 20.07 */ Temperature( 16.2, 30.8, 22.151 ),
    /* 21.07 */ Temperature( 14, 24.3, 18.6573 ),
    /* 22.07 */ Temperature( 13.2, 24.5, 18.3301 ),
    /* 23.07 */ Temperature( 10.6, 23.4, 16.6903 ),
    /* 24.07 */ Temperature( 13.2, 20.8, 16.2743 ),
    /* 25.07 */ Temperature( 12.2, 25.8, 18.8267 ),
    /* 26.07 */ Temperature( 11.9, 28.9, 20.5522 ),
    /* 27.07 */ Temperature( 17.6, 25.8, 21.5691 ),
    /* 28.07 */ Temperature( 16.6, 24.6, 19.2295 ),
    /* 29.07 */ Temperature( 13, 19, 15.9021 ),
    /* 30.07 */ Temperature( 9.6, 19.7, 13.875 ),
    /* 31.07 */ Temperature( 8, 22, 14.5284 ),

    /* 01.08 */ Temperature( 7.6, 27.5, 17.5684 ),
    /* 02.08 */ Temperature( 9.2, 22.2, 16.1035 ),
    /* 03.08 */ Temperature( 12.7, 25.3, 18.2958 ),
    /* 04.08 */ Temperature( 8.6, 31.3, 19.7941 ),
    /* 05.08 */ Temperature( 10.3, 32.7, 21.492 ),
    /* 06.08 */ Temperature( 10, 33.4, 22.4431 ),
    /* 07.08 */ Temperature( 16.8, 22.6, 19.5583 ),
    /* 08.08 */ Temperature( 13.5, 16.7, 15.0264 ),
    /* 09.08 */ Temperature( 13.2, 18.8, 15.6003 ),
    /* 10.08 */ Temperature( 14.6, 27.9, 18.8292 ),
    /* 11.08 */ Temperature( 16.3, 26.4, 20.3837 ),
    /* 12.08 */ Temperature( 12.1, 28.7, 19.9892 ),
    /* 13.08 */ Temperature( 15, 27.4, 19.7542 ),
    /* 14.08 */ Temperature( 11.3, 28.3, 20.5656 ),
    /* 15.08 */ Temperature( 18.6, 28.4, 23.1215 ),
    /* 16.08 */ Temperature( 16, 23.6, 19.491 ),
    /* 17.08 */ Temperature( 12.6, 22, 17.0437 ),
    /* 18.08 */ Temperature( 8.5, 25.7, 16.5589 ),
    /* 19.08 */ Temperature( 13.4, 25.8, 18.0543 ),
    /* 20.08 */ Temperature( 10.9, 21.5, 16.1306 ),
    /* 21.08 */ Temperature( 10.6, 19.2, 14.6177 ),
    /* 22.08 */ Temperature( 14, 24.6, 17.3841 ),
    /* 23.08 */ Temperature( 13.8, 30.4, 20.6125 ),
    /* 24.08 */ Temperature( 12.3, 30.3, 20.7622 ),
    /* 25.08 */ Temperature( 12.8, 30.2, 21.6736 ),
    /* 26.08 */ Temperature( 15, 29.3, 21.266 ),
    /* 27.08 */ Temperature( 12.9, 25.9, 18.791 ),
    /* 28.08 */ Temperature( 9.3, 24.6, 16.2833 ),
    /* 29.08 */ Temperature( 10.8, 25, 16.8459 ),
    /* 30.08 */ Temperature( 8.2, 24.4, 15.9267 ),
    /* 31.08 */ Temperature( 14.1, 20.5, 16.6128 ),

    /* 01.09 */ Temperature( 13.4, 21.9, 16.2205 ),
    /* 02.09 */ Temperature( 12, 20.7, 16.0882 ),
    /* 03.09 */ Temperature( 10.8, 21.3, 14.7913 ),
    /* 04.09 */ Temperature( 7.8, 18.2, 12.2747 ),
    /* 05.09 */ Temperature( 8.1, 22.2, 12.9406 ),
    /* 06.09 */ Temperature( 10, 23.8, 13.8785 ),
    /* 07.09 */ Temperature( 10.7, 21.2, 15.4823 ),
    /* 08.09 */ Temperature( 12.4, 21, 15.8194 ),
    /* 09.09 */ Temperature( 12.7, 16.9, 14.7212 ),
    /* 10.09 */ Temperature( 10.3, 17.7, 12.9271 ),
    /* 11.09 */ Temperature( 10.6, 20.8, 14.4788 ),
    /* 12.09 */ Temperature( 10.8, 21.9, 15.0184 ),
    /* 13.09 */ Temperature( 6.9, 24.6, 14.5222 ),
    /* 14.09 */ Temperature( 8.1, 24, 15.6583 ),
    /* 15.09 */ Temperature( 8.8, 22.8, 15.941 ),
    /* 16.09 */ Temperature( 3.1, 24.5, 14.1486 ),
    /* 17.09 */ Temperature( 12.4, 21.2, 16.0497 ),
    /* 18.09 */ Temperature( 7.8, 16.1, 12.024 ),
    /* 19.09 */ Temperature( 5.3, 18.1, 10.3003 ),
    /* 20.09 */ Temperature( 6.4, 20.3, 12.3177 ),
    /* 21.09 */ Temperature( 6, 23.8, 13.6247 ),
    /* 22.09 */ Temperature( 5.7, 27, 14.6847 ),
    /* 23.09 */ Temperature( 7.8, 28, 16.6238 ),
    /* 24.09 */ Temperature( 9.6, 24.9, 16.7191 ),
    /* 25.09 */ Temperature( 8.4, 17.6, 12.636 ),
    /* 26.09 */ Temperature( 4.3, 18.9, 10.0809 ),
    /* 27.09 */ Temperature( 9.4, 11.2, 10.3344 ),
    /* 28.09 */ Temperature( 7.7, 12.6, 10.5337 ),
    /* 29.09 */ Temperature( 9.8, 15.3, 11.9306 ),
    /* 30.09 */ Temperature( 9.6, 21.1, 13.6635 ),

    /* 01.10 */ Temperature( 8.9, 24.5, 14.8163 ),
    /* 02.10 */ Temperature( 13.5, 20.2, 16.1628 ),
    /* 03.10 */ Temperature( 12.5, 18, 15.4691 ),
    /* 04.10 */ Temperature( 13.8, 25, 17.2073 ),
    /* 05.10 */ Temperature( 9.1, 23.2, 14.6181 ),
    /* 06.10 */ Temperature( 6.4, 23.4, 12.8625 ),
    /* 07.10 */ Temperature( 4.6, 22.1, 11.0052 ),
    /* 08.10 */ Temperature( 2, 22.2, 10.1677 ),
    /* 09.10 */ Temperature( 7.8, 21.6, 12.2139 ),
    /* 10.10 */ Temperature( 7.1, 22.7, 13.0115 ),
    /* 11.10 */ Temperature( 6.1, 21.2, 11.4333 ),
    /* 12.10 */ Temperature( 4.3, 15.2, 10.6104 ),
    /* 13.10 */ Temperature( 5.8, 23, 12.8875 ),
    /* 14.10 */ Temperature( 1, 23, 9.72986 ),
    /* 15.10 */ Temperature( 1, 19.3, 9.33021 ),
    /* 16.10 */ Temperature( 8.5, 20.4, 13.2639 ),
    /* 17.10 */ Temperature( 6.8, 17.3, 11.8174 ),
    /* 18.10 */ Temperature( 5.2, 15.6, 9.06076 ),
    /* 19.10 */ Temperature( 2.7, 13.5, 7.1309 ),
    /* 20.10 */ Temperature( -0.2, 15.8, 6.01667 ),
    /* 21.10 */ Temperature( 2.6, 6.1, 4.9441 ),
    /* 22.10 */ Temperature( -0.8, 13.2, 4.50694 ),
    /* 23.10 */ Temperature( -0.4, 13.3, 4.71007 ),
    /* 24.10 */ Temperature( 2.9, 8.1, 5.96979 ),
    /* 25.10 */ Temperature( 6.3, 10.5, 8.01206 ),
    /* 26.10 */ Temperature( 7, 10.8, 8.14965 ),
    /* 27.10 */ Temperature( 6.6, 9.7, 7.7809 ),
    /* 28.10 */ Temperature( 1.7, 10.8, 6.95728 ),
    /* 29.10 */ Temperature( 2.2, 9.9, 6.62917 ),
    /* 30.10 */ Temperature( 5.8, 15, 8.76181 ),
    /* 31.10 */ Temperature( 0.7, 15, 6.01528 ),

    /* 01.11 */ Temperature( -0.2, 9.7, 3.75842 ),
    /* 02.11 */ Temperature( 6.4, 9.6, 8.00138 ),
    /* 03.11 */ Temperature( 8.7, 13.1, 10.5676 ),
    /* 04.11 */ Temperature( 8, 11.8, 9.54306 ),
    /* 05.11 */ Temperature( 5.8, 15.9, 8.52345 ),
    /* 06.11 */ Temperature( 5.5, 10.8, 7.16493 ),
    /* 07.11 */ Temperature( 5.5, 8.9, 7.30172 ),
    /* 08.11 */ Temperature( 7, 11.7, 8.96701 ),
    /* 09.11 */ Temperature( 2.5, 8.4, 4.86528 ),
    /* 10.11 */ Temperature( 3.7, 9, 5.20828 ),
    /* 11.11 */ Temperature( 2.8, 10.6, 6.80756 ),
    /* 12.11 */ Temperature( 2.7, 9.5, 5.07647 ),
    /* 13.11 */ Temperature( 0.1, 5.4, 3.3945 ),
    /* 14.11 */ Temperature( -0.7, 7.9, 2.02234 ),
    /* 15.11 */ Temperature( -1.8, 6.5, 1.07778 ),
    /* 16.11 */ Temperature( -4.4, 5.1, -0.693772 ),
    /* 17.11 */ Temperature( -0.3, 3.4, 1.33229 ),
    /* 18.11 */ Temperature( -0.4, 4.3, 2.4622 ),
    /* 19.11 */ Temperature( 1.8, 3.6, 2.78282 ),
    /* 20.11 */ Temperature( 1.3, 5.6, 2.95979 ),
    /* 21.11 */ Temperature( 1.6, 5.7, 3.62284 ),
    /* 22.11 */ Temperature( 3.1, 7.3, 5.60277 ),
    /* 23.11 */ Temperature( 4.2, 7.7, 6.28166 ),
    /* 24.11 */ Temperature( -0.5, 11.5, 3.25931 ),
    /* 25.11 */ Temperature( -1, 8.8, 2.86505 ),
    /* 26.11 */ Temperature( 1.2, 6.8, 3.09414 ),
    /* 27.11 */ Temperature( -0.8, 7.5, 3.17805 ),
    /* 28.11 */ Temperature( -2.8, 3.1, -0.920139 ),
    /* 29.11 */ Temperature( -2.6, 1.7, -0.491696 ),
    /* 30.11 */ Temperature( 1.3, 6.5, 3.85 ),

    /* 01.12 */ Temperature( 4.1, 8.7, 5.88924 ),
    /* 02.12 */ Temperature( 4.8, 9, 6.81667 ),
    /* 03.12 */ Temperature( 3.5, 8.5, 6.23633 ),
    /* 04.12 */ Temperature( 2.7, 6.6, 4.63045 ),
    /* 05.12 */ Temperature( 4.3, 8.6, 6.85993 ),
    /* 06.12 */ Temperature( 5.5, 9.3, 7.79201 ),
    /* 07.12 */ Temperature( 3.1, 13.4, 8.79444 ),
    /* 08.12 */ Temperature( 2.6, 6.3, 4.67093 ),
    /* 09.12 */ Temperature( 3, 10.4, 5.75724 ),
    /* 10.12 */ Temperature( 4.1, 6.8, 5.31834 ),
    /* 11.12 */ Temperature( 4.1, 7.4, 5.28993 ),
    /* 12.12 */ Temperature( 3.9, 6.4, 4.64479 ),
    /* 13.12 */ Temperature( 1.7, 9.1, 4.15363 ),
    /* 14.12 */ Temperature( 0.4, 1.8, 0.934602 ),
    /* 15.12 */ Temperature( -4.5, 2.1, -1.17292 ),
    /* 16.12 */ Temperature( -5, 4.8, -2.17431 ),
    /* 17.12 */ Temperature( -5.6, 6.1, -1.35448 ),
    /* 18.12 */ Temperature( -4.9, 6.4, -1.25502 ),
    /* 19.12 */ Temperature( -4.4, 6.6, -1.02396 ),
    /* 20.12 */ Temperature( -7.3, 5.2, -2.63854 ),
    /* 21.12 */ Temperature( -8.5, 5.7, -3.58333 ),
    /* 22.12 */ Temperature( -7.9, -5.3, -6.13438 ),
    /* 23.12 */ Temperature( -6.1, -4.4, -5.23472 ),
    /* 24.12 */ Temperature( -4.6, -3.3, -3.84291 ),
    /* 25.12 */ Temperature( -4.9, -2.8, -3.9066 ),
    /* 26.12 */ Temperature( -4.7, -1.9, -3.10379 ),
    /* 27.12 */ Temperature( -1.9, -0.2, -0.679791 ),
    /* 28.12 */ Temperature( -1.8, 0.5, -0.521875 ),
    /* 29.12 */ Temperature( -2.2, 2.3, -0.430796 ),
    /* 30.12 */ Temperature( 0.9, 5.2, 2.83437 ),
    /* 31.12 */ Temperature( -1, 8.3, 2.27093 )
};
```

### `examples/friedberg/friedberg2007.h`

```cpp
#ifndef _FRIEDBERG_2007_H_
#define _FRIEDBERG_2007_H_

class Temperature
{
public:
    Temperature():
        minValue( 0.0 ),
        maxValue( 0.0 ),
        averageValue( 0.0 )
    {
    }

    Temperature( double min, double max, double average ):
        minValue( min ),
        maxValue( max ),
        averageValue( average )
    {
    }

    double minValue;
    double maxValue;
    double averageValue;
};

extern Temperature friedberg2007[];

#endif
```

### `examples/friedberg/main.cpp`

```cpp
#include <qapplication.h>
#include <qmainwindow.h>
#include <qcombobox.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include "plot.h"

class MainWindow: public QMainWindow
{
public:
    MainWindow( QWidget * = NULL );

private:
    Plot *d_plot;
};

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    d_plot = new Plot( this );
    setCentralWidget( d_plot );

    QToolBar *toolBar = new QToolBar( this );

    QComboBox *typeBox = new QComboBox( toolBar );
    typeBox->addItem( "Bars" );
    typeBox->addItem( "Tube" );
    typeBox->setCurrentIndex( 1 );
    typeBox->setSizePolicy( QSizePolicy::Fixed, QSizePolicy::Fixed );

    QToolButton *btnExport = new QToolButton( toolBar );
    btnExport->setText( "Export" );
    btnExport->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    connect( btnExport, SIGNAL( clicked() ), d_plot, SLOT( exportPlot() ) );

    toolBar->addWidget( typeBox );
    toolBar->addWidget( btnExport );
    addToolBar( toolBar );

    d_plot->setMode( typeBox->currentIndex() );
    connect( typeBox, SIGNAL( currentIndexChanged( int ) ),
             d_plot, SLOT( setMode( int ) ) );
}

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWindow w;
    w.setObjectName( "MainWindow" );
    w.resize( 600, 400 );
    w.show();

    return a.exec();
}
```

### `examples/friedberg/plot.cpp`

```cpp
#include "plot.h"
#include "friedberg2007.h"
#include <qwt_plot_zoomer.h>
#include <qwt_plot_panner.h>
#include <qwt_plot_marker.h>
#include <qwt_plot_grid.h>
#include <qwt_plot_curve.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_intervalcurve.h>
#include <qwt_legend.h>
#include <qwt_interval_symbol.h>
#include <qwt_symbol.h>
#include <qwt_series_data.h>
#include <qwt_text.h>
#include <qwt_scale_draw.h>
#include <qwt_plot_renderer.h>
#include <qdatetime.h>

class Grid: public QwtPlotGrid
{
public:
    Grid()
    {
        enableXMin( true );
        setMajorPen( Qt::white, 0, Qt::DotLine );
        setMinorPen( Qt::gray, 0, Qt::DotLine );
    }

    virtual void updateScaleDiv( const QwtScaleDiv &xScaleDiv,
        const QwtScaleDiv &yScaleDiv )
    {
        QwtScaleDiv scaleDiv( xScaleDiv.lowerBound(), 
            xScaleDiv.upperBound() );

        scaleDiv.setTicks( QwtScaleDiv::MinorTick,
            xScaleDiv.ticks( QwtScaleDiv::MinorTick ) );
        scaleDiv.setTicks( QwtScaleDiv::MajorTick,
            xScaleDiv.ticks( QwtScaleDiv::MediumTick ) );

        QwtPlotGrid::updateScaleDiv( scaleDiv, yScaleDiv );
    }
};

class YearScaleDraw: public QwtScaleDraw
{
public:
    YearScaleDraw()
    {
        setTickLength( QwtScaleDiv::MajorTick, 0 );
        setTickLength( QwtScaleDiv::MinorTick, 0 );
        setTickLength( QwtScaleDiv::MediumTick, 6 );

        setLabelRotation( -60.0 );
        setLabelAlignment( Qt::AlignLeft | Qt::AlignVCenter );

        setSpacing( 15 );
    }

    virtual QwtText label( double value ) const
    {
        return QDate::longMonthName( int( value / 30 ) + 1 );
    }
};

Plot::Plot( QWidget *parent ):
    QwtPlot( parent )
{
    setObjectName( "FriedbergPlot" );
    setTitle( "Temperature of Friedberg/Germany" );

    setAxisTitle( QwtPlot::xBottom, "2007" );
    setAxisScaleDiv( QwtPlot::xBottom, yearScaleDiv() );
    setAxisScaleDraw( QwtPlot::xBottom, new YearScaleDraw() );

    setAxisTitle( QwtPlot::yLeft,
        QString( "Temperature [%1C]" ).arg( QChar( 0x00B0 ) ) );

    QwtPlotCanvas *canvas = new QwtPlotCanvas();
    canvas->setPalette( Qt::darkGray );
    canvas->setBorderRadius( 10 );

    setCanvas( canvas );

    // grid
    QwtPlotGrid *grid = new Grid;
    grid->attach( this );

    insertLegend( new QwtLegend(), QwtPlot::RightLegend );

    const int numDays = 365;
    QVector<QPointF> averageData( numDays );
    QVector<QwtIntervalSample> rangeData( numDays );

    for ( int i = 0; i < numDays; i++ )
    {
        const Temperature &t = friedberg2007[i];
        averageData[i] = QPointF( double( i ), t.averageValue );
        rangeData[i] = QwtIntervalSample( double( i ),
            QwtInterval( t.minValue, t.maxValue ) );
    }

    insertCurve( "Average", averageData, Qt::black );
    insertErrorBars( "Range", rangeData, Qt::blue );

    // LeftButton for the zooming
    // MidButton for the panning
    // RightButton: zoom out by 1
    // Ctrl+RighButton: zoom out to full size

    QwtPlotZoomer* zoomer = new QwtPlotZoomer( canvas );
    zoomer->setRubberBandPen( QColor( Qt::black ) );
    zoomer->setTrackerPen( QColor( Qt::black ) );
    zoomer->setMousePattern( QwtEventPattern::MouseSelect2,
        Qt::RightButton, Qt::ControlModifier );
    zoomer->setMousePattern( QwtEventPattern::MouseSelect3,
        Qt::RightButton );

    QwtPlotPanner *panner = new QwtPlotPanner( canvas );
    panner->setMouseButton( Qt::MidButton );
}

QwtScaleDiv Plot::yearScaleDiv() const
{
    const int days[] = { 31, 28, 31, 30, 31, 30, 31, 31, 30, 31, 30, 31 };

    QList<double> mediumTicks;
    mediumTicks += 0.0;
    for ( uint i = 0; i < sizeof( days ) / sizeof( days[0] ); i++ )
        mediumTicks += mediumTicks.last() + days[i];

    QList<double> minorTicks;
    for ( int i = 1; i <= 365; i += 7 )
        minorTicks += i;

    QList<double> majorTicks;
    for ( int i = 0; i < 12; i++ )
        majorTicks += i * 30 + 15;

    QwtScaleDiv scaleDiv( mediumTicks.first(), mediumTicks.last() + 1, 
        minorTicks, mediumTicks, majorTicks );
    return scaleDiv;
}

void Plot::insertCurve( const QString& title,
    const QVector<QPointF>& samples, const QColor &color )
{
    d_curve = new QwtPlotCurve( title );
    d_curve->setRenderHint( QwtPlotItem::RenderAntialiased );
    d_curve->setStyle( QwtPlotCurve::NoCurve );
    d_curve->setLegendAttribute( QwtPlotCurve::LegendShowSymbol );

    QwtSymbol *symbol = new QwtSymbol( QwtSymbol::XCross );
    symbol->setSize( 4 );
    symbol->setPen( color );
    d_curve->setSymbol( symbol );

    d_curve->setSamples( samples );
    d_curve->attach( this );
}

void Plot::insertErrorBars(
    const QString &title,
    const QVector<QwtIntervalSample>& samples,
    const QColor &color )
{
    d_intervalCurve = new QwtPlotIntervalCurve( title );
    d_intervalCurve->setRenderHint( QwtPlotItem::RenderAntialiased );
    d_intervalCurve->setPen( Qt::white );

    QColor bg( color );
    bg.setAlpha( 150 );
    d_intervalCurve->setBrush( QBrush( bg ) );
    d_intervalCurve->setStyle( QwtPlotIntervalCurve::Tube );

    d_intervalCurve->setSamples( samples );
    d_intervalCurve->attach( this );
}

void Plot::setMode( int style )
{
    if ( style == Tube )
    {
        d_intervalCurve->setStyle( QwtPlotIntervalCurve::Tube );
        d_intervalCurve->setSymbol( NULL );
        d_intervalCurve->setRenderHint( QwtPlotItem::RenderAntialiased, true );
    }
    else
    {
        d_intervalCurve->setStyle( QwtPlotIntervalCurve::NoCurve );

        QColor c( d_intervalCurve->brush().color().rgb() ); // skip alpha

        QwtIntervalSymbol *errorBar =
            new QwtIntervalSymbol( QwtIntervalSymbol::Bar );
        errorBar->setWidth( 8 ); // should be something even
        errorBar->setPen( c );

        d_intervalCurve->setSymbol( errorBar );
        d_intervalCurve->setRenderHint( QwtPlotItem::RenderAntialiased, false );
    }

    replot();
}

void Plot::exportPlot()
{
    QwtPlotRenderer renderer;
    renderer.exportTo( this, "friedberg.pdf" );
}
```

### `examples/friedberg/plot.h`

```cpp
#ifndef _PLOT_H_
#define _PLOT_H_

#include <qwt_plot.h>
#include <qwt_scale_div.h>
#include <qwt_series_data.h>

class QwtPlotCurve;
class QwtPlotIntervalCurve;

class Plot: public QwtPlot
{
    Q_OBJECT

public:
    enum Mode
    {
        Bars,
        Tube
    };

    Plot( QWidget * = NULL );

public Q_SLOTS:
    void setMode( int );
    void exportPlot();

private:
    void insertCurve( const QString &title,
        const QVector<QPointF> &, const QColor & );

    void insertErrorBars( const QString &title,
        const QVector<QwtIntervalSample> &,
        const QColor &color );


    QwtScaleDiv yearScaleDiv() const;

    QwtPlotIntervalCurve *d_intervalCurve;
    QwtPlotCurve *d_curve;
};

#endif
```

### `examples/itemeditor/editor.cpp`

```cpp
#include "editor.h"
#include <qwt_plot.h>
#include <qwt_plot_canvas.h>
#include <qwt_scale_map.h>
#include <qwt_plot_shapeitem.h>
#include <qevent.h>

class Overlay: public QwtWidgetOverlay
{
public:
    Overlay( QWidget *parent, Editor *editor ):
        QwtWidgetOverlay( parent ),
        d_editor( editor )
    {
        switch( editor->mode() )
        {
            case Editor::NoMask:
            {
                setMaskMode( QwtWidgetOverlay::NoMask );
                setRenderMode( QwtWidgetOverlay::AutoRenderMode );
                break;
            }
            case Editor::Mask:
            {
                setMaskMode( QwtWidgetOverlay::MaskHint );
                setRenderMode( QwtWidgetOverlay::AutoRenderMode );
                break;
            }
            case Editor::AlphaMask:
            {
                setMaskMode( QwtWidgetOverlay::AlphaMask );
                setRenderMode( QwtWidgetOverlay::AutoRenderMode );
                break;
            }
            case Editor::AlphaMaskRedraw:
            {
                setMaskMode( QwtWidgetOverlay::AlphaMask );
                setRenderMode( QwtWidgetOverlay::DrawOverlay );
                break;
            }
            case Editor::AlphaMaskCopyMask:
            {
                setMaskMode( QwtWidgetOverlay::AlphaMask );
                setRenderMode( QwtWidgetOverlay::CopyAlphaMask );
                break;
            }
        }
    }

protected:
    virtual void drawOverlay( QPainter *painter ) const
    {
        d_editor->drawOverlay( painter );
    }

    virtual QRegion maskHint() const
    {
        return d_editor->maskHint();
    }

private:
    Editor *d_editor;
};

Editor::Editor( QwtPlot* plot ):
    QObject( plot ),
    d_isEnabled( false ),
    d_overlay( NULL ),
    d_mode( Mask )
{
    setEnabled( true );
}


Editor::~Editor()
{
    delete d_overlay;
}

QwtPlot *Editor::plot()
{
    return qobject_cast<QwtPlot *>( parent() );
}

const QwtPlot *Editor::plot() const
{
    return qobject_cast<const QwtPlot *>( parent() );
}

void Editor::setMode( Mode mode )
{
    d_mode = mode;
}

Editor::Mode Editor::mode() const
{
    return d_mode;
}

void Editor::setEnabled( bool on )
{
    if ( on == d_isEnabled )
        return;

    QwtPlot *plot = qobject_cast<QwtPlot *>( parent() );
    if ( plot )
    {
        d_isEnabled = on;

        if ( on )
        {
            plot->canvas()->installEventFilter( this );
        }
        else
        {
            plot->canvas()->removeEventFilter( this );

            delete d_overlay;
            d_overlay = NULL;
        }
    }
}

bool Editor::isEnabled() const
{
    return d_isEnabled;
}

bool Editor::eventFilter( QObject* object, QEvent* event )
{
    QwtPlot *plot = qobject_cast<QwtPlot *>( parent() );
    if ( plot && object == plot->canvas() )
    {
        switch( event->type() )
        {
            case QEvent::MouseButtonPress:
            {
                const QMouseEvent* mouseEvent =
                    dynamic_cast<QMouseEvent* >( event );

                if ( d_overlay == NULL && 
                    mouseEvent->button() == Qt::LeftButton  )
                {
                    const bool accepted = pressed( mouseEvent->pos() );
                    if ( accepted )
                    {
                        d_overlay = new Overlay( plot->canvas(), this );

                        d_overlay->updateOverlay();
                        d_overlay->show();
                    }
                }

                break;
            }
            case QEvent::MouseMove:
            {
                if ( d_overlay )
                {
                    const QMouseEvent* mouseEvent =
                        dynamic_cast< QMouseEvent* >( event );

                    const bool accepted = moved( mouseEvent->pos() );
                    if ( accepted )
                        d_overlay->updateOverlay();
                }

                break;
            }
            case QEvent::MouseButtonRelease:
            {
                const QMouseEvent* mouseEvent =
                    static_cast<QMouseEvent* >( event );

                if ( d_overlay && mouseEvent->button() == Qt::LeftButton )
                {
                    released( mouseEvent->pos() );

                    delete d_overlay;
                    d_overlay = NULL;
                }

                break;
            }
            default:
                break;
        }

        return false;
    }

    return QObject::eventFilter( object, event );
}

bool Editor::pressed( const QPoint& pos )
{
    d_editedItem = itemAt( pos );
    if ( d_editedItem )
    {
        d_currentPos = pos;
        setItemVisible( d_editedItem, false );

        return true;
    }

    return false; // don't accept the position
}

bool Editor::moved( const QPoint& pos )
{
    if ( plot() == NULL )
        return false;

    const QwtScaleMap xMap = plot()->canvasMap( d_editedItem->xAxis() );
    const QwtScaleMap yMap = plot()->canvasMap( d_editedItem->yAxis() );

    const QPointF p1 = QwtScaleMap::invTransform( xMap, yMap, d_currentPos );
    const QPointF p2 = QwtScaleMap::invTransform( xMap, yMap, pos );

#if QT_VERSION >= 0x040600
    const QPainterPath shape = d_editedItem->shape().translated( p2 - p1 );
#else
    const double dx = p2.x() - p1.x();
    const double dy = p2.y() - p1.y();

    QPainterPath shape = d_editedItem->shape();
    for ( int i = 0; i < shape.elementCount(); i++ )
    {
        const QPainterPath::Element &el = shape.elementAt( i );
        shape.setElementPositionAt( i, el.x + dx, el.y + dy );
    }
#endif

    d_editedItem->setShape( shape );
    d_currentPos = pos;

    return true;
}

void Editor::released( const QPoint& pos )
{
    Q_UNUSED( pos );

    if ( d_editedItem  )
    {
        raiseItem( d_editedItem );
        setItemVisible( d_editedItem, true );
    }
}

QwtPlotShapeItem* Editor::itemAt( const QPoint& pos ) const
{
    const QwtPlot *plot = this->plot();
    if ( plot == NULL )
        return NULL;

    // translate pos into the plot coordinates
    double coords[ QwtPlot::axisCnt ];
    coords[ QwtPlot::xBottom ] =
        plot->canvasMap( QwtPlot::xBottom ).invTransform( pos.x() );
    coords[ QwtPlot::xTop ] =
        plot->canvasMap( QwtPlot::xTop ).invTransform( pos.x() );
    coords[ QwtPlot::yLeft ] =
        plot->canvasMap( QwtPlot::yLeft ).invTransform( pos.y() );
    coords[ QwtPlot::yRight ] =
        plot->canvasMap( QwtPlot::yRight ).invTransform( pos.y() );

    QwtPlotItemList items = plot->itemList();
    for ( int i = items.size() - 1; i >= 0; i-- )
    {
        QwtPlotItem *item = items[ i ];
        if ( item->isVisible() &&
            item->rtti() == QwtPlotItem::Rtti_PlotShape )
        {
            QwtPlotShapeItem *shapeItem = static_cast<QwtPlotShapeItem *>( item );
            const QPointF p( coords[ item->xAxis() ], coords[ item->yAxis() ] );

            if ( shapeItem->boundingRect().contains( p )
                && shapeItem->shape().contains( p ) )
            {
                return shapeItem;
            }
        }
    }

    return NULL;
}

QRegion Editor::maskHint() const
{
    return maskHint( d_editedItem );
}

QRegion Editor::maskHint( QwtPlotShapeItem *shapeItem ) const
{
    const QwtPlot *plot = this->plot();
    if ( plot == NULL || shapeItem == NULL )
        return QRegion();

    const QwtScaleMap xMap = plot->canvasMap( shapeItem->xAxis() );
    const QwtScaleMap yMap = plot->canvasMap( shapeItem->yAxis() );

    QRect rect = QwtScaleMap::transform( xMap, yMap,
        shapeItem->shape().boundingRect() ).toRect();

    const int m = 5; // some margin for the pen
    return rect.adjusted( -m, -m, m, m );
}

void Editor::drawOverlay( QPainter* painter ) const
{
    const QwtPlot *plot = this->plot();
    if ( plot == NULL || d_editedItem == NULL )
        return;

    const QwtScaleMap xMap = plot->canvasMap( d_editedItem->xAxis() );
    const QwtScaleMap yMap = plot->canvasMap( d_editedItem->yAxis() );

    painter->setRenderHint( QPainter::Antialiasing,
        d_editedItem->testRenderHint( QwtPlotItem::RenderAntialiased ) );
    d_editedItem->draw( painter, xMap, yMap,
        plot->canvas()->contentsRect() );
}

void Editor::raiseItem( QwtPlotShapeItem *shapeItem )
{
    const QwtPlot *plot = this->plot();
    if ( plot == NULL || shapeItem == NULL )
        return;

    const QwtPlotItemList items = plot->itemList();
    for ( int i = items.size() - 1; i >= 0; i-- )
    {
        QwtPlotItem *item = items[ i ];
        if ( shapeItem == item )
            return;

        if ( item->isVisible() &&
            item->rtti() == QwtPlotItem::Rtti_PlotShape )
        {
            shapeItem->setZ( item->z() + 1 );
            return;
        }
    }
}

void Editor::setItemVisible( QwtPlotShapeItem *item, bool on )
{
    if ( plot() == NULL || item == NULL || item->isVisible() == on )
        return;

    const bool doAutoReplot = plot()->autoReplot();
    plot()->setAutoReplot( false );

    item->setVisible( on );

    plot()->setAutoReplot( doAutoReplot );

    /*
      Avoid replot with a full repaint of the canvas. 
      For special combinations - f.e. using the 
      raster paint engine on a remote display -
      this makes a difference.
     */

    QwtPlotCanvas *canvas =
        qobject_cast<QwtPlotCanvas *>( plot()->canvas() );
    if ( canvas )
        canvas->invalidateBackingStore();

    plot()->canvas()->update( maskHint( item ) );

}
```

### `examples/itemeditor/editor.h`

```cpp
#ifndef _EDITOR_H_
#define _EDITOR_H_

#include <qobject.h>
#include <qregion.h>
#include <qpointer.h>
#include <qwt_widget_overlay.h>

class QwtPlot;
class QwtPlotShapeItem;
class QPainter;
class QPoint;

class Editor: public QObject
{
    Q_OBJECT

public:
    enum Mode
    {
        NoMask,
        Mask,
        AlphaMask,
        AlphaMaskRedraw,
        AlphaMaskCopyMask
    };

    Editor( QwtPlot * );
    virtual ~Editor();

    const QwtPlot *plot() const;
    QwtPlot *plot();

    virtual void setEnabled( bool on );
    bool isEnabled() const;

    void drawOverlay( QPainter * ) const;
    QRegion maskHint() const;

    virtual bool eventFilter( QObject *, QEvent *);

    void setMode( Mode mode );
    Mode mode() const;

private:
    bool pressed( const QPoint & );
    bool moved( const QPoint & );
    void released( const QPoint & );

    QwtPlotShapeItem* itemAt( const QPoint& ) const;
    void raiseItem( QwtPlotShapeItem * );

    QRegion maskHint( QwtPlotShapeItem * ) const;
    void setItemVisible( QwtPlotShapeItem *item, bool on );

    bool d_isEnabled;
    QPointer<QwtWidgetOverlay> d_overlay;

    // Mouse positions
    QPointF d_currentPos;
    QwtPlotShapeItem* d_editedItem;

    Mode d_mode;
};

#endif
```

### `examples/itemeditor/main.cpp`

```cpp
#include "plot.h"
#include <qapplication.h>
#include <qmainwindow.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include <qcombobox.h>

class MainWindow: public QMainWindow
{
public:
    MainWindow( QWidget * = NULL );
};

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    Plot *plot = new Plot( this );
    setCentralWidget( plot );

    QToolBar *toolBar = new QToolBar( this );

    QComboBox *modeBox = new QComboBox( toolBar );
    modeBox->addItem( "No Mask" );
    modeBox->addItem( "Mask" );
    modeBox->addItem( "Alpha Mask" );
    modeBox->addItem( "Alpha Mask/Redraw" );
    modeBox->addItem( "Alpha Mask/Copy Mask" );
    modeBox->setCurrentIndex( 1 );
    modeBox->setSizePolicy( QSizePolicy::Fixed, QSizePolicy::Fixed );

    connect( modeBox, SIGNAL( currentIndexChanged( int ) ),
             plot, SLOT( setMode( int ) ) );

    QToolButton *btnExport = new QToolButton( toolBar );
    btnExport->setText( "Export" );
    btnExport->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    connect( btnExport, SIGNAL( clicked() ), plot, SLOT( exportPlot() ) );

    toolBar->addWidget( modeBox );
    toolBar->addWidget( btnExport );
    addToolBar( toolBar );

}

int main( int argc, char **argv )
{
    QApplication app( argc, argv );

    MainWindow window;
    window.resize( 600, 400 );
    window.show();

    return app.exec();
}
```

### `examples/itemeditor/plot.cpp`

```cpp
#include "plot.h"
#include "editor.h"
#include <qwt_plot_shapeitem.h>
#include <qwt_plot_magnifier.h>
#include <qwt_plot_canvas.h>
#include <qwt_legend.h>
#include <qwt_plot_renderer.h>

class Legend: public QwtLegend
{
protected:
    virtual QWidget *createWidget( const QwtLegendData &data ) const
    {
        QWidget *w = QwtLegend::createWidget( data );
        if ( w )
        {
            w->setStyleSheet(
                "border-radius: 5px;"
                "padding: 2px;"
                "background: LemonChiffon;"
            );
        }

        return w;
    }
};

Plot::Plot( QWidget *parent ):
    QwtPlot( parent )
{
    setAutoReplot( false );

    setTitle( "Movable Items" );

    const int margin = 5;
    setContentsMargins( margin, margin, margin, margin );

    setAutoFillBackground( true );
    setPalette( QColor( "DimGray" ).lighter( 110 ) );

    QwtPlotCanvas *canvas = new QwtPlotCanvas();
#if 0
    // a gradient making a replot slow on X11
    canvas->setStyleSheet(
        "border: 2px solid Black;"
        "border-radius: 15px;"
        "background-color: qlineargradient( x1: 0, y1: 0, x2: 0, y2: 1,"
            "stop: 0 LemonChiffon, stop: 0.5 PaleGoldenrod, stop: 1 LemonChiffon );"
    );
#else
    canvas->setStyleSheet(
        "border: 2px inset DimGray;"
        "border-radius: 15px;"
        "background: LemonChiffon;"
    );
#endif

    setCanvas( canvas );
    insertLegend( new Legend(), QwtPlot::RightLegend );

    populate();

    updateAxes();
    for ( int axis = 0; axis < QwtPlot::axisCnt; axis++ )
        setAxisAutoScale( axis, false );

    d_editor = new Editor( this );
    ( void ) new QwtPlotMagnifier( canvas );
}

void Plot::populate()
{
    addShape( "Rectangle", ShapeFactory::Rect, "RoyalBlue", 
        QPointF( 30.0, 50.0 ), QSizeF( 40.0, 50.0 ) );
    addShape( "Ellipse", ShapeFactory::Ellipse, "IndianRed", 
        QPointF( 80.0, 130.0 ), QSizeF( 50.0, 40.0 ) );
    addShape( "Ring", ShapeFactory::Ring, "DarkOliveGreen", 
        QPointF( 30.0, 165.0 ), QSizeF( 40.0, 40.0 ) );
    addShape( "Triangle", ShapeFactory::Triangle, "SandyBrown", 
        QPointF( 165.0, 165.0 ), QSizeF( 60.0, 40.0 ) );
    addShape( "Star", ShapeFactory::Star, "DarkViolet", 
        QPointF( 165.0, 50.0 ), QSizeF( 40.0, 50.0 ) );
    addShape( "Hexagon", ShapeFactory::Hexagon, "DarkSlateGray", 
        QPointF( 120.0, 70.0 ), QSizeF( 50.0, 50.0 ) );

}

void Plot::addShape( const QString &title,
    ShapeFactory::Shape shape, const QColor &color, 
    const QPointF &pos, const QSizeF &size )
{
    QwtPlotShapeItem *item = new QwtPlotShapeItem( title );
    item->setItemAttribute( QwtPlotItem::Legend, true );
    item->setLegendMode( QwtPlotShapeItem::LegendShape );
    item->setLegendIconSize( QSize( 20, 20 ) );
    item->setRenderHint( QwtPlotItem::RenderAntialiased, true );
    item->setShape( ShapeFactory::path( shape, pos, size ) );

    QColor fillColor = color;
    fillColor.setAlpha( 200 );

    QPen pen( color, 3 );
    pen.setJoinStyle( Qt::MiterJoin );
    item->setPen( pen );
    item->setBrush( fillColor );

    item->attach( this );
}

void Plot::exportPlot()
{
    QwtPlotRenderer renderer;
    renderer.exportTo( this, "shapes.pdf" );
}

void Plot::setMode( int mode )
{
    d_editor->setMode( static_cast<Editor::Mode>( mode ) );
}
```

### `examples/itemeditor/plot.h`

```cpp
#ifndef _PLOT_H
#define _PLOT_H

#include <qwt_plot.h>
#include "shapefactory.h"

class QColor;
class QSizeF;
class QPointF;
class Editor;

class Plot: public QwtPlot
{
    Q_OBJECT

public:
    Plot( QWidget *parent = NULL );

public Q_SLOTS:
    void exportPlot();
    void setMode( int );
    
private:
    void populate();

    void addShape( const QString &title,
        ShapeFactory::Shape, const QColor &,
        const QPointF &, const QSizeF & );

    Editor *d_editor;
};

#endif
```

### `examples/itemeditor/shapefactory.cpp`

```cpp
#include "shapefactory.h"

QPainterPath ShapeFactory::path( Shape shape, 
    const QPointF &pos, const QSizeF &size )
{
    QRectF rect;
    rect.setSize( size );
    rect.moveCenter( pos );
    
    QPainterPath path;

    switch( shape )
    {
        case Rect:
        {
            path.addRect( rect );
            break;
        }
        case Triangle:
        {
            QPolygonF triangle;
            triangle += rect.bottomLeft();
            triangle += QPointF( rect.center().x(), rect.top() );
            triangle += rect.bottomRight();

            path.addPolygon( triangle );
            break;
        }
        case Ellipse:
        {
            path.addEllipse( rect );
            break;
        }
        case Ring:
        {
            path.addEllipse( rect );

            const double w = 0.25 * rect.width();
            path.addEllipse( rect.adjusted( w, w, -w, -w ) );
            break;
        }
        case Star:
        {
            const double cos30 = 0.866025;

            const double dy = 0.25 * size.height();
            const double dx = 0.5 * size.width() * cos30 / 3.0;

            double x1 = pos.x() - 3 * dx;
            double y1 = pos.y() - 2 * dy;

            const double x2 = x1 + 1 * dx;
            const double x3 = x1 + 2 * dx;
            const double x4 = x1 + 3 * dx;
            const double x5 = x1 + 4 * dx;
            const double x6 = x1 + 5 * dx;
            const double x7 = x1 + 6 * dx;

            const double y2 = y1 + 1 * dy;
            const double y3 = y1 + 2 * dy;
            const double y4 = y1 + 3 * dy;
            const double y5 = y1 + 4 * dy;

            QPolygonF star;
            star += QPointF( x4, y1 );
            star += QPointF( x5, y2 );
            star += QPointF( x7, y2 );
            star += QPointF( x6, y3 );
            star += QPointF( x7, y4 );
            star += QPointF( x5, y4 );
            star += QPointF( x4, y5 );
            star += QPointF( x3, y4 );
            star += QPointF( x1, y4 );
            star += QPointF( x2, y3 );
            star += QPointF( x1, y2 );
            star += QPointF( x3, y2 );

            path.addPolygon( star );
            break;
        }
        case Hexagon:
        {
            const double cos30 = 0.866025; 

            const double dx = 0.5 * size.width() - cos30;
            const double dy = 0.25 * size.height();

            double x1 = pos.x() - dx;
            double y1 = pos.y() - 2 * dy;

            const double x2 = x1 + 1 * dx;
            const double x3 = x1 + 2 * dx;

            const double y2 = y1 + 1 * dy;
            const double y3 = y1 + 3 * dy;
            const double y4 = y1 + 4 * dy;

            QPolygonF hexagon;
            hexagon += QPointF( x2, y1 );
            hexagon += QPointF( x3, y2 );
            hexagon += QPointF( x3, y3 );
            hexagon += QPointF( x2, y4 );
            hexagon += QPointF( x1, y3 );
            hexagon += QPointF( x1, y2 );

            path.addPolygon( hexagon );
            break;
        }
    };

    path.closeSubpath();
    return path;
}
```

### `examples/itemeditor/shapefactory.h`

```cpp
#ifndef _SHAPE_FACTORY_H_
#define _SHAPE_FACTORY_H_

#include <qpainterpath.h>

namespace ShapeFactory
{
    enum Shape
    {
        Rect,
        Triangle,
        Ellipse,
        Ring,
        Star,
        Hexagon
    };

    QPainterPath path( Shape, const QPointF &, const QSizeF & );
};

#endif
```

### `examples/legends/main.cpp`

```cpp
#include <qapplication.h>
#include "mainwindow.h"

int main ( int argc, char **argv )
{
    QApplication a( argc, argv );
    a.setStyle( "Windows" );

    MainWindow w;
    w.resize( 700, 500 );
    w.show();

    return a.exec();
}
```

### `examples/legends/mainwindow.cpp`

```cpp
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include <qlayout.h>
#include <qwt_plot_renderer.h>
#include <qwt_plot_canvas.h>
#include "plot.h"
#include "panel.h"
#include "mainwindow.h"

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    d_plot = new Plot();

    Settings settings;
    settings.legend.isEnabled = true;
    settings.legend.position = QwtPlot::BottomLegend;

    settings.legendItem.isEnabled = false;
    settings.legendItem.numColumns = 1;
    settings.legendItem.alignment = Qt::AlignRight | Qt::AlignVCenter;
    settings.legendItem.backgroundMode = 0;
    settings.legendItem.size = d_plot->canvas()->font().pointSize();

    settings.curve.numCurves = 4;
    settings.curve.title = "Curve";
    
    d_panel = new Panel();
    d_panel->setSettings( settings );

    QWidget *box = new QWidget( this );
    QHBoxLayout *layout = new QHBoxLayout( box );
    layout->addWidget( d_plot, 10 );
    layout->addWidget( d_panel );

    setCentralWidget( box );

    QToolBar *toolBar = new QToolBar( this );

    QToolButton *btnExport = new QToolButton( toolBar );
    btnExport->setText( "Export" );
    toolBar->addWidget( btnExport );

    addToolBar( toolBar );

    updatePlot();

    connect( d_panel, SIGNAL( edited() ), SLOT( updatePlot() ) );
    connect( btnExport, SIGNAL( clicked() ), SLOT( exportPlot() ) );
}

void MainWindow::updatePlot()
{
    d_plot->applySettings( d_panel->settings() );
}

void MainWindow::exportPlot()
{
    QwtPlotRenderer renderer;
    renderer.exportTo( d_plot, "legends.pdf" );
}
```

### `examples/legends/mainwindow.h`

```cpp
#include <qmainwindow.h>

class Plot;
class Panel;

class MainWindow : public QMainWindow
{
    Q_OBJECT

public:
    MainWindow( QWidget *parent = 0 );

private Q_SLOTS:
    void updatePlot();
    void exportPlot();

private:
    Plot *d_plot;
    Panel *d_panel;
};
```

### `examples/legends/panel.cpp`

```cpp
#include "panel.h"
#include "settings.h"
#include <qcheckbox.h>
#include <qspinbox.h>
#include <qcombobox.h>
#include <qgroupbox.h>
#include <qlayout.h>
#include <qlabel.h>
#include <qlineedit.h>
#include <qwt_plot.h>
#include <qwt_plot_legenditem.h>

Panel::Panel( QWidget *parent ):
    QWidget( parent )
{
    // create widgets

    d_legend.checkBox = new QCheckBox( "Enabled" );

    d_legend.positionBox = new QComboBox();
    d_legend.positionBox->addItem( "Left", QwtPlot::LeftLegend );
    d_legend.positionBox->addItem( "Right", QwtPlot::RightLegend );
    d_legend.positionBox->addItem( "Bottom", QwtPlot::BottomLegend );
    d_legend.positionBox->addItem( "Top", QwtPlot::TopLegend );
    d_legend.positionBox->addItem( "External", QwtPlot::TopLegend + 1 );

    d_legendItem.checkBox = new QCheckBox( "Enabled" );

    d_legendItem.numColumnsBox = new QSpinBox();
    d_legendItem.numColumnsBox->setRange( 0, 10 );
    d_legendItem.numColumnsBox->setSpecialValueText( "Unlimited" );

    d_legendItem.hAlignmentBox = new QComboBox();
    d_legendItem.hAlignmentBox->addItem( "Left", Qt::AlignLeft );
    d_legendItem.hAlignmentBox->addItem( "Centered", Qt::AlignHCenter );
    d_legendItem.hAlignmentBox->addItem( "Right", Qt::AlignRight );
    
    d_legendItem.vAlignmentBox = new QComboBox();
    d_legendItem.vAlignmentBox->addItem( "Top", Qt::AlignTop );
    d_legendItem.vAlignmentBox->addItem( "Centered", Qt::AlignVCenter );
    d_legendItem.vAlignmentBox->addItem( "Bottom", Qt::AlignBottom );

    d_legendItem.backgroundBox = new QComboBox();
    d_legendItem.backgroundBox->addItem( "Legend",
        QwtPlotLegendItem::LegendBackground );
    d_legendItem.backgroundBox->addItem( "Items",
        QwtPlotLegendItem::ItemBackground );

    d_legendItem.sizeBox = new QSpinBox();
    d_legendItem.sizeBox->setRange( 8, 22 );

    d_curve.numCurves = new QSpinBox();
    d_curve.numCurves->setRange( 0, 99 );

    d_curve.title = new QLineEdit();

    // layout

    QGroupBox *legendBox = new QGroupBox( "Legend" );
    QGridLayout *legendBoxLayout = new QGridLayout( legendBox );

    int row = 0;
    legendBoxLayout->addWidget( d_legend.checkBox, row, 0, 1, -1 );

    row++;
    legendBoxLayout->addWidget( new QLabel( "Position" ), row, 0 );
    legendBoxLayout->addWidget( d_legend.positionBox, row, 1 );


    QGroupBox *legendItemBox = new QGroupBox( "Legend Item" );
    QGridLayout *legendItemBoxLayout = new QGridLayout( legendItemBox );

    row = 0;
    legendItemBoxLayout->addWidget( d_legendItem.checkBox, row, 0, 1, -1 );

    row++;
    legendItemBoxLayout->addWidget( new QLabel( "Columns" ), row, 0 );
    legendItemBoxLayout->addWidget( d_legendItem.numColumnsBox, row, 1 );

    row++;
    legendItemBoxLayout->addWidget( new QLabel( "Horizontal" ), row, 0 );
    legendItemBoxLayout->addWidget( d_legendItem.hAlignmentBox, row, 1 );

    row++;
    legendItemBoxLayout->addWidget( new QLabel( "Vertical" ), row, 0 );
    legendItemBoxLayout->addWidget( d_legendItem.vAlignmentBox, row, 1 );

    row++;
    legendItemBoxLayout->addWidget( new QLabel( "Background" ), row, 0 );
    legendItemBoxLayout->addWidget( d_legendItem.backgroundBox, row, 1 );

    row++;
    legendItemBoxLayout->addWidget( new QLabel( "Size" ), row, 0 );
    legendItemBoxLayout->addWidget( d_legendItem.sizeBox, row, 1 );

    QGroupBox *curveBox = new QGroupBox( "Curves" );
    QGridLayout *curveBoxLayout = new QGridLayout( curveBox );

    row = 0;
    curveBoxLayout->addWidget( new QLabel( "Number" ), row, 0 );
    curveBoxLayout->addWidget( d_curve.numCurves, row, 1 );

    row++;
    curveBoxLayout->addWidget( new QLabel( "Title" ), row, 0 );
    curveBoxLayout->addWidget( d_curve.title, row, 1 );

    QVBoxLayout *layout = new QVBoxLayout( this );
    layout->addWidget( legendBox );
    layout->addWidget( legendItemBox );
    layout->addWidget( curveBox );
    layout->addStretch( 10 );

    connect( d_legend.checkBox, 
        SIGNAL( stateChanged( int ) ), SIGNAL( edited() ) );
    connect( d_legend.positionBox, 
        SIGNAL( currentIndexChanged( int ) ), SIGNAL( edited() ) );

    connect( d_legendItem.checkBox, 
        SIGNAL( stateChanged( int ) ), SIGNAL( edited() ) );
    connect( d_legendItem.numColumnsBox, 
        SIGNAL( valueChanged( int ) ), SIGNAL( edited() ) );
    connect( d_legendItem.hAlignmentBox, 
        SIGNAL( currentIndexChanged( int ) ), SIGNAL( edited() ) );
    connect( d_legendItem.vAlignmentBox, 
        SIGNAL( currentIndexChanged( int ) ), SIGNAL( edited() ) );
    connect( d_legendItem.backgroundBox, 
        SIGNAL( currentIndexChanged( int ) ), SIGNAL( edited() ) );
    connect( d_curve.numCurves, 
        SIGNAL( valueChanged( int ) ), SIGNAL( edited() ) );
    connect( d_legendItem.sizeBox, 
        SIGNAL( valueChanged( int ) ), SIGNAL( edited() ) );
    connect( d_curve.title, 
        SIGNAL( textEdited( const QString & ) ), SIGNAL( edited() ) );
}

void Panel::setSettings( const Settings &settings)
{
    blockSignals( true );

    d_legend.checkBox->setCheckState(
        settings.legend.isEnabled ? Qt::Checked : Qt::Unchecked );
    d_legend.positionBox->setCurrentIndex( settings.legend.position );

    d_legendItem.checkBox->setCheckState(
        settings.legendItem.isEnabled ? Qt::Checked : Qt::Unchecked );

    d_legendItem.numColumnsBox->setValue( settings.legendItem.numColumns );

    int align = settings.legendItem.alignment;

    if ( align & Qt::AlignLeft )
        d_legendItem.hAlignmentBox->setCurrentIndex( 0 );
    else if ( align & Qt::AlignRight )
        d_legendItem.hAlignmentBox->setCurrentIndex( 2 );
    else
        d_legendItem.hAlignmentBox->setCurrentIndex( 1 );

    if ( align & Qt::AlignTop )
        d_legendItem.vAlignmentBox->setCurrentIndex( 0 );
    else if ( align & Qt::AlignBottom )
        d_legendItem.vAlignmentBox->setCurrentIndex( 2 );
    else
        d_legendItem.vAlignmentBox->setCurrentIndex( 1 );

    d_legendItem.backgroundBox->setCurrentIndex( 
        settings.legendItem.backgroundMode );

    d_legendItem.sizeBox->setValue( settings.legendItem.size );

    d_curve.numCurves->setValue( settings.curve.numCurves );
    d_curve.title->setText( settings.curve.title );

    blockSignals( false );
}

Settings Panel::settings() const
{
    Settings s;

    s.legend.isEnabled = 
        d_legend.checkBox->checkState() == Qt::Checked;
    s.legend.position = d_legend.positionBox->currentIndex();

    s.legendItem.isEnabled = 
        d_legendItem.checkBox->checkState() == Qt::Checked;
    s.legendItem.numColumns = d_legendItem.numColumnsBox->value();
    
    int align = 0;

    int hIndex = d_legendItem.hAlignmentBox->currentIndex();
    if ( hIndex == 0 )
        align |= Qt::AlignLeft;
    else if ( hIndex == 2 )
        align |= Qt::AlignRight;
    else
        align |= Qt::AlignHCenter;

    int vIndex = d_legendItem.vAlignmentBox->currentIndex();
    if ( vIndex == 0 )
        align |= Qt::AlignTop;
    else if ( vIndex == 2 )
        align |= Qt::AlignBottom;
    else
        align |= Qt::AlignVCenter;

    s.legendItem.alignment = align;

    s.legendItem.backgroundMode = 
            d_legendItem.backgroundBox->currentIndex();
    s.legendItem.size = d_legendItem.sizeBox->value();

    s.curve.numCurves = d_curve.numCurves->value();
    s.curve.title = d_curve.title->text();

    return s;
}
```

### `examples/legends/panel.h`

```cpp
#ifndef _PANEL_
#define _PANEL_

#include "settings.h"
#include <qwidget.h>

class QCheckBox;
class QComboBox;
class QSpinBox;
class QLineEdit;

class Panel: public QWidget
{
    Q_OBJECT

public:
    Panel( QWidget *parent = NULL );

    void setSettings( const Settings &);
    Settings settings() const;
    
Q_SIGNALS:
    void edited();

private:
    struct
    {
        QCheckBox *checkBox;
        QComboBox *positionBox;

    } d_legend;

    struct
    {
        QCheckBox *checkBox;
        QSpinBox *numColumnsBox;
        QComboBox *hAlignmentBox;
        QComboBox *vAlignmentBox;
        QComboBox *backgroundBox;
        QSpinBox *sizeBox;

    } d_legendItem;

    struct
    {
        QSpinBox *numCurves;
        QLineEdit *title;

    } d_curve;
};

#endif
```

### `examples/legends/plot.cpp`

```cpp
#include "plot.h"
#include "settings.h"
#include <qwt_plot_curve.h>
#include <qwt_plot_legenditem.h>
#include <qwt_legend.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_grid.h>
#include <qwt_plot_layout.h>

class LegendItem: public QwtPlotLegendItem
{
public:
    LegendItem()
    {
        setRenderHint( QwtPlotItem::RenderAntialiased );

        QColor color( Qt::white );

        setTextPen( color );
#if 1
        setBorderPen( color );

        QColor c( Qt::gray );
        c.setAlpha( 200 );

        setBackgroundBrush( c );
#endif
    }
};

class Curve: public QwtPlotCurve
{
public:
    Curve( int index ):
        d_index( index )
    {
        setRenderHint( QwtPlotItem::RenderAntialiased );
        initData();
    }

    void setCurveTitle( const QString &title )
    {
        QString txt("%1 %2");
        setTitle( QString( "%1 %2" ).arg( title ).arg( d_index ) );
    }

    void initData()
    {
        QVector<QPointF> points;

        double y = qrand() % 1000;

        for ( double x = 0.0; x <= 1000.0; x += 100.0 )
        {
            double off = qrand() % 200 - 100;
            if ( y + off > 980.0 || y + off < 20.0 )
                off = -off;

            y += off;

            points += QPointF( x, y );
        }

        setSamples( points );
    }

private:
    const int d_index;
};

Plot::Plot( QWidget *parent ):
    QwtPlot( parent ),
    d_externalLegend( NULL ),
    d_legendItem( NULL ),
    d_isDirty( false )
{
    QwtPlotCanvas *canvas = new QwtPlotCanvas();
    canvas->setFocusIndicator( QwtPlotCanvas::CanvasFocusIndicator );
    canvas->setFocusPolicy( Qt::StrongFocus );
    canvas->setPalette( Qt::black );
    setCanvas( canvas );

    setAutoReplot( false );

    setTitle( "Legend Test" );
    setFooter( "Footer" );

    // grid
    QwtPlotGrid *grid = new QwtPlotGrid;
    grid->enableXMin( true );
    grid->setMajorPen( Qt::gray, 0, Qt::DotLine );
    grid->setMinorPen( Qt::darkGray, 0, Qt::DotLine );
    grid->attach( this );

    // axis
    setAxisScale( QwtPlot::yLeft, 0.0, 1000.0 );
    setAxisScale( QwtPlot::xBottom, 0.0, 1000.0 );
}

Plot::~Plot()
{
    delete d_externalLegend;
}

void Plot::insertCurve()
{
    static int counter = 1;

    const char *colors[] = 
    { 
        "LightSalmon",
        "SteelBlue",
        "Yellow",
        "Fuchsia",
        "PaleGreen",
        "PaleTurquoise",
        "Cornsilk",
        "HotPink",
        "Peru",
        "Maroon"
    };
    const int numColors = sizeof( colors ) / sizeof( colors[0] );

    QwtPlotCurve *curve = new Curve( counter++ );
    curve->setPen( QColor( colors[ counter % numColors ] ), 2 );
    curve->attach( this );
}

void Plot::applySettings( const Settings &settings )
{
    d_isDirty = false;
    setAutoReplot( true );

    if ( settings.legend.isEnabled )
    {
        if ( settings.legend.position > QwtPlot::TopLegend )
        {
            if ( legend() )
            {
                // remove legend controlled by the plot
                insertLegend( NULL );
            }

            if ( d_externalLegend == NULL )
            {
                d_externalLegend = new QwtLegend();
                d_externalLegend->setWindowTitle("Plot Legend");

                connect( 
                    this, 
                    SIGNAL( legendDataChanged( const QVariant &, 
                        const QList<QwtLegendData> & ) ),
                    d_externalLegend, 
                    SLOT( updateLegend( const QVariant &, 
                        const QList<QwtLegendData> & ) ) );

                d_externalLegend->show();

                // populate the new legend
                updateLegend();
            }
        }
        else
        {
            delete d_externalLegend;
            d_externalLegend = NULL;

            if ( legend() == NULL || 
                plotLayout()->legendPosition() != settings.legend.position )
            {
                insertLegend( new QwtLegend(), 
                    QwtPlot::LegendPosition( settings.legend.position ) );
            }
        }
    }
    else
    {
        insertLegend( NULL );

        delete d_externalLegend;
        d_externalLegend = NULL;
    }

    if ( settings.legendItem.isEnabled )
    {
        if ( d_legendItem == NULL )
        {
            d_legendItem = new LegendItem();
            d_legendItem->attach( this );
        }

        d_legendItem->setMaxColumns( settings.legendItem.numColumns );
        d_legendItem->setAlignment( Qt::Alignment( settings.legendItem.alignment ) );
        d_legendItem->setBackgroundMode(
            QwtPlotLegendItem::BackgroundMode( settings.legendItem.backgroundMode ) );
        if ( settings.legendItem.backgroundMode == 
            QwtPlotLegendItem::ItemBackground )
        {
            d_legendItem->setBorderRadius( 4 );
            d_legendItem->setMargin( 0 );
            d_legendItem->setSpacing( 4 );
            d_legendItem->setItemMargin( 2 );
        }
        else
        {
            d_legendItem->setBorderRadius( 8 );
            d_legendItem->setMargin( 4 );
            d_legendItem->setSpacing( 2 );
            d_legendItem->setItemMargin( 0 );
        }

        QFont font = d_legendItem->font();
        font.setPointSize( settings.legendItem.size );
        d_legendItem->setFont( font );
    }
    else
    {
        delete d_legendItem;
        d_legendItem = NULL;
    }

    QwtPlotItemList curveList = itemList( QwtPlotItem::Rtti_PlotCurve );
    if ( curveList.size() != settings.curve.numCurves )
    {
        while ( curveList.size() > settings.curve.numCurves )
        {
            QwtPlotItem* curve = curveList.takeFirst();
            delete curve;
        }
        
        for ( int i = curveList.size(); i < settings.curve.numCurves; i++ )
            insertCurve();
    }

    curveList = itemList( QwtPlotItem::Rtti_PlotCurve );
    for ( int i = 0; i < curveList.count(); i++ )
    {
        Curve* curve = static_cast<Curve*>( curveList[i] );
        curve->setCurveTitle( settings.curve.title );

        int sz = 0.5 * settings.legendItem.size;
        curve->setLegendIconSize( QSize( sz, sz ) );
    }

    setAutoReplot( false );
    if ( d_isDirty )
    {
        d_isDirty = false;
        replot();
    }
}

void Plot::replot()
{
    if ( autoReplot() )
    {
        d_isDirty = true;
        return;
    }

    QwtPlot::replot();
}
```

### `examples/legends/plot.h`

```cpp
#ifndef _PLOT_H_
#define _PLOT_H_

#include <qwt_plot.h>

class Settings;
class LegendItem;
class QwtLegend;

class Plot: public QwtPlot
{
    Q_OBJECT

public:
    Plot( QWidget *parent = NULL );
    virtual ~Plot();

public Q_SLOTS:
    void applySettings( const Settings & );

public:
    virtual void replot();

private:
    void insertCurve();

    QwtLegend *d_externalLegend;
    LegendItem *d_legendItem;
    bool d_isDirty;
};

#endif
```

### `examples/legends/settings.h`

```cpp
#ifndef _SETTINGS_
#define _SETTINGS_

#include <qstring.h>

class Settings
{
public:
    Settings()
    {
        legend.isEnabled = false;
        legend.position = 0;

        legendItem.isEnabled = false;
        legendItem.numColumns = 0;
        legendItem.alignment = 0;
        legendItem.backgroundMode = 0;
        legendItem.size = 12;

        curve.numCurves = 0;
        curve.title = "Curve";
    }
    
    struct
    {
        bool isEnabled;
        int position;
    } legend;

    struct
    {
        bool isEnabled;
        int numColumns;
        int alignment;
        int backgroundMode;
        int size;
        
    } legendItem;
    
    struct
    {
        int numCurves;
        QString title;
    } curve;
};

#endif
```

### `examples/oscilloscope/curvedata.cpp`

```cpp
#include "curvedata.h"
#include "signaldata.h"

const SignalData &CurveData::values() const
{
    return SignalData::instance();
}

SignalData &CurveData::values()
{
    return SignalData::instance();
}

QPointF CurveData::sample( size_t i ) const
{
    return SignalData::instance().value( i );
}

size_t CurveData::size() const
{
    return SignalData::instance().size();
}

QRectF CurveData::boundingRect() const
{
    return SignalData::instance().boundingRect();
}
```

### `examples/oscilloscope/curvedata.h`

```cpp
#include <qwt_series_data.h>
#include <qpointer.h>

class SignalData;

class CurveData: public QwtSeriesData<QPointF>
{
public:
    const SignalData &values() const;
    SignalData &values();

    virtual QPointF sample( size_t i ) const;
    virtual size_t size() const;

    virtual QRectF boundingRect() const;
};
```

### `examples/oscilloscope/knob.cpp`

```cpp
#include "knob.h"
#include <qwt_math.h>
#include <qpen.h>
#include <qwt_knob.h>
#include <qwt_round_scale_draw.h>
#include <qwt_scale_engine.h>
#include <qlabel.h>
#include <qevent.h>

Knob::Knob( const QString &title, double min, double max, QWidget *parent ):
    QWidget( parent )
{
    QFont font( "Helvetica", 10 );

    d_knob = new QwtKnob( this );
    d_knob->setFont( font );

    QwtScaleDiv scaleDiv =
        d_knob->scaleEngine()->divideScale( min, max, 5, 3 );

    QList<double> ticks = scaleDiv.ticks( QwtScaleDiv::MajorTick );
    if ( ticks.size() > 0 && ticks[0] > min )
    {
        if ( ticks.first() > min )
            ticks.prepend( min );
        if ( ticks.last() < max )
            ticks.append( max );
    }
    scaleDiv.setTicks( QwtScaleDiv::MajorTick, ticks );
    d_knob->setScale( scaleDiv );

    d_knob->setKnobWidth( 50 );

    font.setBold( true );
    d_label = new QLabel( title, this );
    d_label->setFont( font );
    d_label->setAlignment( Qt::AlignTop | Qt::AlignHCenter );

    setSizePolicy( QSizePolicy::MinimumExpanding,
        QSizePolicy::MinimumExpanding );

    connect( d_knob, SIGNAL( valueChanged( double ) ),
        this, SIGNAL( valueChanged( double ) ) );
}

QSize Knob::sizeHint() const
{
    QSize sz1 = d_knob->sizeHint();
    QSize sz2 = d_label->sizeHint();

    const int w = qMax( sz1.width(), sz2.width() );
    const int h = sz1.height() + sz2.height();

    int off = qCeil( d_knob->scaleDraw()->extent( d_knob->font() ) );
    off -= 15; // spacing

    return QSize( w, h - off );
}

void Knob::setValue( double value )
{
    d_knob->setValue( value );
}

double Knob::value() const
{
    return d_knob->value();
}

void Knob::setTheme( const QColor &color )
{
    d_knob->setPalette( color );
}

QColor Knob::theme() const
{
    return d_knob->palette().color( QPalette::Window );
}

void Knob::resizeEvent( QResizeEvent *event )
{
    const QSize sz = event->size();
    const QSize hint = d_label->sizeHint();

    d_label->setGeometry( 0, sz.height() - hint.height(),
        sz.width(), hint.height() );

    const int knobHeight = d_knob->sizeHint().height();

    int off = qCeil( d_knob->scaleDraw()->extent( d_knob->font() ) );
    off -= 15; // spacing

    d_knob->setGeometry( 0, d_label->pos().y() - knobHeight + off,
        sz.width(), knobHeight );
}
```

### `examples/oscilloscope/knob.h`

```cpp
#ifndef _KNOB_H_
#define _KNOB_H_

#include <qwidget.h>

class QwtKnob;
class QLabel;

class Knob: public QWidget
{
    Q_OBJECT

    Q_PROPERTY( QColor theme READ theme WRITE setTheme )

public:
    Knob( const QString &title,
        double min, double max, QWidget *parent = NULL );

    virtual QSize sizeHint() const;

    void setValue( double value );
    double value() const;

    void setTheme( const QColor & );
    QColor theme() const;

Q_SIGNALS:
    double valueChanged( double );

protected:
    virtual void resizeEvent( QResizeEvent * );

private:
    QwtKnob *d_knob;
    QLabel *d_label;
};

#endif
```

### `examples/oscilloscope/main.cpp`

```cpp
#include <qapplication.h>
#include "mainwindow.h"
#include "samplingthread.h"

int main( int argc, char **argv )
{
    QApplication app( argc, argv );
    app.setPalette( Qt::darkGray );

    MainWindow window;
    window.resize( 800, 400 );

    SamplingThread samplingThread;
    samplingThread.setFrequency( window.frequency() );
    samplingThread.setAmplitude( window.amplitude() );
    samplingThread.setInterval( window.signalInterval() );

    window.connect( &window, SIGNAL( frequencyChanged( double ) ),
        &samplingThread, SLOT( setFrequency( double ) ) );
    window.connect( &window, SIGNAL( amplitudeChanged( double ) ),
        &samplingThread, SLOT( setAmplitude( double ) ) );
    window.connect( &window, SIGNAL( signalIntervalChanged( double ) ),
        &samplingThread, SLOT( setInterval( double ) ) );

    window.show();

    samplingThread.start();
    window.start();

    bool ok = app.exec();

    samplingThread.stop();
    samplingThread.wait( 1000 );

    return ok;
}
```

### `examples/oscilloscope/mainwindow.cpp`

```cpp
#include "mainwindow.h"
#include "plot.h"
#include "knob.h"
#include "wheelbox.h"
#include <qwt_scale_engine.h>
#include <qlabel.h>
#include <qlayout.h>

MainWindow::MainWindow( QWidget *parent ):
    QWidget( parent )
{
    const double intervalLength = 10.0; // seconds

    d_plot = new Plot( this );
    d_plot->setIntervalLength( intervalLength );

    d_amplitudeKnob = new Knob( "Amplitude", 0.0, 200.0, this );
    d_amplitudeKnob->setValue( 160.0 );

    d_frequencyKnob = new Knob( "Frequency [Hz]", 0.1, 20.0, this );
    d_frequencyKnob->setValue( 17.8 );

    d_intervalWheel = new WheelBox( "Displayed [s]", 1.0, 100.0, 1.0, this );
    d_intervalWheel->setValue( intervalLength );

    d_timerWheel = new WheelBox( "Sample Interval [ms]", 0.0, 20.0, 0.1, this );
    d_timerWheel->setValue( 10.0 );

    QVBoxLayout* vLayout1 = new QVBoxLayout();
    vLayout1->addWidget( d_intervalWheel );
    vLayout1->addWidget( d_timerWheel );
    vLayout1->addStretch( 10 );
    vLayout1->addWidget( d_amplitudeKnob );
    vLayout1->addWidget( d_frequencyKnob );

    QHBoxLayout *layout = new QHBoxLayout( this );
    layout->addWidget( d_plot, 10 );
    layout->addLayout( vLayout1 );

    connect( d_amplitudeKnob, SIGNAL( valueChanged( double ) ),
        SIGNAL( amplitudeChanged( double ) ) );
    connect( d_frequencyKnob, SIGNAL( valueChanged( double ) ),
        SIGNAL( frequencyChanged( double ) ) );
    connect( d_timerWheel, SIGNAL( valueChanged( double ) ),
        SIGNAL( signalIntervalChanged( double ) ) );

    connect( d_intervalWheel, SIGNAL( valueChanged( double ) ),
        d_plot, SLOT( setIntervalLength( double ) ) );
}

void MainWindow::start()
{
    d_plot->start();
}

double MainWindow::frequency() const
{
    return d_frequencyKnob->value();
}

double MainWindow::amplitude() const
{
    return d_amplitudeKnob->value();
}

double MainWindow::signalInterval() const
{
    return d_timerWheel->value();
}
```

### `examples/oscilloscope/mainwindow.h`

```cpp
#include <qwidget.h>

class Plot;
class Knob;
class WheelBox;

class MainWindow : public QWidget
{
    Q_OBJECT

public:
    MainWindow( QWidget * = NULL );

    void start();

    double amplitude() const;
    double frequency() const;
    double signalInterval() const;

Q_SIGNALS:
    void amplitudeChanged( double );
    void frequencyChanged( double );
    void signalIntervalChanged( double );

private:
    Knob *d_frequencyKnob;
    Knob *d_amplitudeKnob;
    WheelBox *d_timerWheel;
    WheelBox *d_intervalWheel;

    Plot *d_plot;
};
```

### `examples/oscilloscope/plot.cpp`

```cpp
#include "plot.h"
#include "curvedata.h"
#include "signaldata.h"
#include <qwt_plot_grid.h>
#include <qwt_plot_layout.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_marker.h>
#include <qwt_plot_curve.h>
#include <qwt_plot_directpainter.h>
#include <qwt_curve_fitter.h>
#include <qwt_painter.h>
#include <qevent.h>

class Canvas: public QwtPlotCanvas
{
public:
    Canvas( QwtPlot *plot = NULL ):
        QwtPlotCanvas( plot )
    {
        // The backing store is important, when working with widget
        // overlays ( f.e rubberbands for zooming ).
        // Here we don't have them and the internal
        // backing store of QWidget is good enough.

        setPaintAttribute( QwtPlotCanvas::BackingStore, false );
        setBorderRadius( 10 );

        if ( QwtPainter::isX11GraphicsSystem() )
        {
#if QT_VERSION < 0x050000
            // Even if not liked by the Qt development, Qt::WA_PaintOutsidePaintEvent
            // works on X11. This has a nice effect on the performance.

            setAttribute( Qt::WA_PaintOutsidePaintEvent, true );
#endif

            // Disabling the backing store of Qt improves the performance
            // for the direct painter even more, but the canvas becomes
            // a native window of the window system, receiving paint events
            // for resize and expose operations. Those might be expensive
            // when there are many points and the backing store of
            // the canvas is disabled. So in this application
            // we better don't disable both backing stores.

            if ( testPaintAttribute( QwtPlotCanvas::BackingStore ) )
            {
                setAttribute( Qt::WA_PaintOnScreen, true );
                setAttribute( Qt::WA_NoSystemBackground, true );
            }
        }

        setupPalette();
    }

private:
    void setupPalette()
    {
        QPalette pal = palette();

#if QT_VERSION >= 0x040400
        QLinearGradient gradient;
        gradient.setCoordinateMode( QGradient::StretchToDeviceMode );
        gradient.setColorAt( 0.0, QColor( 0, 49, 110 ) );
        gradient.setColorAt( 1.0, QColor( 0, 87, 174 ) );

        pal.setBrush( QPalette::Window, QBrush( gradient ) );
#else
        pal.setBrush( QPalette::Window, QBrush( color ) );
#endif

        // QPalette::WindowText is used for the curve color
        pal.setColor( QPalette::WindowText, Qt::green );

        setPalette( pal );
    }
};

Plot::Plot( QWidget *parent ):
    QwtPlot( parent ),
    d_paintedPoints( 0 ),
    d_interval( 0.0, 10.0 ),
    d_timerId( -1 )
{
    d_directPainter = new QwtPlotDirectPainter();

    setAutoReplot( false );
    setCanvas( new Canvas() );

    plotLayout()->setAlignCanvasToScales( true );

    setAxisTitle( QwtPlot::xBottom, "Time [s]" );
    setAxisScale( QwtPlot::xBottom, d_interval.minValue(), d_interval.maxValue() );
    setAxisScale( QwtPlot::yLeft, -200.0, 200.0 );

    QwtPlotGrid *grid = new QwtPlotGrid();
    grid->setPen( Qt::gray, 0.0, Qt::DotLine );
    grid->enableX( true );
    grid->enableXMin( true );
    grid->enableY( true );
    grid->enableYMin( false );
    grid->attach( this );

    d_origin = new QwtPlotMarker();
    d_origin->setLineStyle( QwtPlotMarker::Cross );
    d_origin->setValue( d_interval.minValue() + d_interval.width() / 2.0, 0.0 );
    d_origin->setLinePen( Qt::gray, 0.0, Qt::DashLine );
    d_origin->attach( this );

    d_curve = new QwtPlotCurve();
    d_curve->setStyle( QwtPlotCurve::Lines );
    d_curve->setPen( canvas()->palette().color( QPalette::WindowText ) );
    d_curve->setRenderHint( QwtPlotItem::RenderAntialiased, true );
    d_curve->setPaintAttribute( QwtPlotCurve::ClipPolygons, false );
    d_curve->setData( new CurveData() );
    d_curve->attach( this );
}

Plot::~Plot()
{
    delete d_directPainter;
}

void Plot::start()
{
    d_clock.start();
    d_timerId = startTimer( 10 );
}

void Plot::replot()
{
    CurveData *data = static_cast<CurveData *>( d_curve->data() );
    data->values().lock();

    QwtPlot::replot();
    d_paintedPoints = data->size();

    data->values().unlock();
}

void Plot::setIntervalLength( double interval )
{
    if ( interval > 0.0 && interval != d_interval.width() )
    {
        d_interval.setMaxValue( d_interval.minValue() + interval );
        setAxisScale( QwtPlot::xBottom,
            d_interval.minValue(), d_interval.maxValue() );

        replot();
    }
}

void Plot::updateCurve()
{
    CurveData *data = static_cast<CurveData *>( d_curve->data() );
    data->values().lock();

    const int numPoints = data->size();
    if ( numPoints > d_paintedPoints )
    {
        const bool doClip = !canvas()->testAttribute( Qt::WA_PaintOnScreen );
        if ( doClip )
        {
            /*
                Depending on the platform setting a clip might be an important
                performance issue. F.e. for Qt Embedded this reduces the
                part of the backing store that has to be copied out - maybe
                to an unaccelerated frame buffer device.
            */

            const QwtScaleMap xMap = canvasMap( d_curve->xAxis() );
            const QwtScaleMap yMap = canvasMap( d_curve->yAxis() );

            QRectF br = qwtBoundingRect( *data,
                d_paintedPoints - 1, numPoints - 1 );

            const QRect clipRect = QwtScaleMap::transform( xMap, yMap, br ).toRect();
            d_directPainter->setClipRegion( clipRect );
        }

        d_directPainter->drawSeries( d_curve,
            d_paintedPoints - 1, numPoints - 1 );
        d_paintedPoints = numPoints;
    }

    data->values().unlock();
}

void Plot::incrementInterval()
{
    d_interval = QwtInterval( d_interval.maxValue(),
        d_interval.maxValue() + d_interval.width() );

    CurveData *data = static_cast<CurveData *>( d_curve->data() );
    data->values().clearStaleValues( d_interval.minValue() );

    // To avoid, that the grid is jumping, we disable
    // the autocalculation of the ticks and shift them
    // manually instead.

    QwtScaleDiv scaleDiv = axisScaleDiv( QwtPlot::xBottom );
    scaleDiv.setInterval( d_interval );

    for ( int i = 0; i < QwtScaleDiv::NTickTypes; i++ )
    {
        QList<double> ticks = scaleDiv.ticks( i );
        for ( int j = 0; j < ticks.size(); j++ )
            ticks[j] += d_interval.width();
        scaleDiv.setTicks( i, ticks );
    }
    setAxisScaleDiv( QwtPlot::xBottom, scaleDiv );

    d_origin->setValue( d_interval.minValue() + d_interval.width() / 2.0, 0.0 );

    d_paintedPoints = 0;
    replot();
}

void Plot::timerEvent( QTimerEvent *event )
{
    if ( event->timerId() == d_timerId )
    {
        updateCurve();

        const double elapsed = d_clock.elapsed() / 1000.0;
        if ( elapsed > d_interval.maxValue() )
            incrementInterval();

        return;
    }

    QwtPlot::timerEvent( event );
}

void Plot::resizeEvent( QResizeEvent *event )
{
    d_directPainter->reset();
    QwtPlot::resizeEvent( event );
}

void Plot::showEvent( QShowEvent * )
{
    replot();
}

bool Plot::eventFilter( QObject *object, QEvent *event )
{
    if ( object == canvas() && 
        event->type() == QEvent::PaletteChange )
    {
        d_curve->setPen( canvas()->palette().color( QPalette::WindowText ) );
    }

    return QwtPlot::eventFilter( object, event );
}
```

### `examples/oscilloscope/plot.h`

```cpp
#include <qwt_plot.h>
#include <qwt_interval.h>
#include <qwt_system_clock.h>

class QwtPlotCurve;
class QwtPlotMarker;
class QwtPlotDirectPainter;

class Plot: public QwtPlot
{
    Q_OBJECT

public:
    Plot( QWidget * = NULL );
    virtual ~Plot();

    void start();
    virtual void replot();

    virtual bool eventFilter( QObject *, QEvent * );

public Q_SLOTS:
    void setIntervalLength( double );

protected:
    virtual void showEvent( QShowEvent * );
    virtual void resizeEvent( QResizeEvent * );
    virtual void timerEvent( QTimerEvent * );

private:
    void updateCurve();
    void incrementInterval();

    QwtPlotMarker *d_origin;
    QwtPlotCurve *d_curve;
    int d_paintedPoints;

    QwtPlotDirectPainter *d_directPainter;

    QwtInterval d_interval;
    int d_timerId;

    QwtSystemClock d_clock;
};
```

### `examples/oscilloscope/samplingthread.cpp`

```cpp
#include "samplingthread.h"
#include "signaldata.h"
#include <qwt_math.h>
#include <math.h>

#if QT_VERSION < 0x040600
#define qFastSin(x) ::sin(x)
#endif

SamplingThread::SamplingThread( QObject *parent ):
    QwtSamplingThread( parent ),
    d_frequency( 5.0 ),
    d_amplitude( 20.0 )
{
}

void SamplingThread::setFrequency( double frequency )
{
    d_frequency = frequency;
}

double SamplingThread::frequency() const
{
    return d_frequency;
}

void SamplingThread::setAmplitude( double amplitude )
{
    d_amplitude = amplitude;
}

double SamplingThread::amplitude() const
{
    return d_amplitude;
}

void SamplingThread::sample( double elapsed )
{
    if ( d_frequency > 0.0 )
    {
        const QPointF s( elapsed, value( elapsed ) );
        SignalData::instance().append( s );
    }
}

double SamplingThread::value( double timeStamp ) const
{
    const double period = 1.0 / d_frequency;

    const double x = ::fmod( timeStamp, period );
    const double v = d_amplitude * qFastSin( x / period * 2 * M_PI );

    return v;
}
```

### `examples/oscilloscope/samplingthread.h`

```cpp
#include <qwt_sampling_thread.h>

class SamplingThread: public QwtSamplingThread
{
    Q_OBJECT

public:
    SamplingThread( QObject *parent = NULL );

    double frequency() const;
    double amplitude() const;

public Q_SLOTS:
    void setAmplitude( double );
    void setFrequency( double );

protected:
    virtual void sample( double elapsed );

private:
    virtual double value( double timeStamp ) const;

    double d_frequency;
    double d_amplitude;
};
```

### `examples/oscilloscope/signaldata.cpp`

```cpp
#include "signaldata.h"
#include <qvector.h>
#include <qmutex.h>
#include <qreadwritelock.h>

class SignalData::PrivateData
{
public:
    PrivateData():
        boundingRect( 1.0, 1.0, -2.0, -2.0 ) // invalid
    {
        values.reserve( 1000 );
    }

    inline void append( const QPointF &sample )
    {
        values.append( sample );

        // adjust the bounding rectangle

        if ( boundingRect.width() < 0 || boundingRect.height() < 0 )
        {
            boundingRect.setRect( sample.x(), sample.y(), 0.0, 0.0 );
        }
        else
        {
            boundingRect.setRight( sample.x() );

            if ( sample.y() > boundingRect.bottom() )
                boundingRect.setBottom( sample.y() );

            if ( sample.y() < boundingRect.top() )
                boundingRect.setTop( sample.y() );
        }
    }

    QReadWriteLock lock;

    QVector<QPointF> values;
    QRectF boundingRect;

    QMutex mutex; // protecting pendingValues
    QVector<QPointF> pendingValues;
};

SignalData::SignalData()
{
    d_data = new PrivateData();
}

SignalData::~SignalData()
{
    delete d_data;
}

int SignalData::size() const
{
    return d_data->values.size();
}

QPointF SignalData::value( int index ) const
{
    return d_data->values[index];
}

QRectF SignalData::boundingRect() const
{
    return d_data->boundingRect;
}

void SignalData::lock()
{
    d_data->lock.lockForRead();
}

void SignalData::unlock()
{
    d_data->lock.unlock();
}

void SignalData::append( const QPointF &sample )
{
    d_data->mutex.lock();
    d_data->pendingValues += sample;

    const bool isLocked = d_data->lock.tryLockForWrite();
    if ( isLocked )
    {
        const int numValues = d_data->pendingValues.size();
        const QPointF *pendingValues = d_data->pendingValues.data();

        for ( int i = 0; i < numValues; i++ )
            d_data->append( pendingValues[i] );

        d_data->pendingValues.clear();

        d_data->lock.unlock();
    }

    d_data->mutex.unlock();
}

void SignalData::clearStaleValues( double limit )
{
    d_data->lock.lockForWrite();

    d_data->boundingRect = QRectF( 1.0, 1.0, -2.0, -2.0 ); // invalid

    const QVector<QPointF> values = d_data->values;
    d_data->values.clear();
    d_data->values.reserve( values.size() );

    int index;
    for ( index = values.size() - 1; index >= 0; index-- )
    {
        if ( values[index].x() < limit )
            break;
    }

    if ( index > 0 )
        d_data->append( values[index++] );

    while ( index < values.size() - 1 )
        d_data->append( values[index++] );

    d_data->lock.unlock();
}

SignalData &SignalData::instance()
{
    static SignalData valueVector;
    return valueVector;
}
```

### `examples/oscilloscope/signaldata.h`

```cpp
#ifndef _SIGNAL_DATA_H_
#define _SIGNAL_DATA_H_ 1

#include <qrect.h>

class SignalData
{
public:
    static SignalData &instance();

    void append( const QPointF &pos );
    void clearStaleValues( double min );

    int size() const;
    QPointF value( int index ) const;

    QRectF boundingRect() const;

    void lock();
    void unlock();

private:
    SignalData();
    SignalData( const SignalData & );
    SignalData &operator=( const SignalData & );

    virtual ~SignalData();

    class PrivateData;
    PrivateData *d_data;
};

#endif
```

### `examples/oscilloscope/wheelbox.cpp`

```cpp
#include "wheelbox.h"
#include <qwt_wheel.h>
#include <qlcdnumber.h>
#include <qlabel.h>
#include <qlayout.h>
#include <qevent.h>
#include <qapplication.h>

class Wheel: public QwtWheel
{
public:
    Wheel( WheelBox *parent ):
        QwtWheel( parent )
    {
        setFocusPolicy( Qt::WheelFocus );
        parent->installEventFilter( this );
    }

    virtual bool eventFilter( QObject *object, QEvent *event )
    {
        if ( event->type() == QEvent::Wheel )
        {
            const QWheelEvent *we = static_cast<QWheelEvent *>( event );

            QWheelEvent wheelEvent( QPoint( 5, 5 ), we->delta(),
                we->buttons(), we->modifiers(),
                we->orientation() );

            QApplication::sendEvent( this, &wheelEvent );
            return true;
        }
        return QwtWheel::eventFilter( object, event );
    }
};

WheelBox::WheelBox( const QString &title,
        double min, double max, double stepSize, QWidget *parent ):
    QWidget( parent )
{

    d_number = new QLCDNumber( this );
    d_number->setSegmentStyle( QLCDNumber::Filled );
    d_number->setAutoFillBackground( true );
    d_number->setFixedHeight( d_number->sizeHint().height() * 2 );
    d_number->setFocusPolicy( Qt::WheelFocus );

    QPalette pal( Qt::black );
    pal.setColor( QPalette::WindowText, Qt::green );
    d_number->setPalette( pal );

    d_wheel = new Wheel( this );
    d_wheel->setOrientation( Qt::Vertical );
    d_wheel->setInverted( true );
    d_wheel->setRange( min, max );
    d_wheel->setSingleStep( stepSize );
    d_wheel->setPageStepCount( 5 );
    d_wheel->setFixedHeight( d_number->height() );

    d_number->setFocusProxy( d_wheel );

    QFont font( "Helvetica", 10 );
    font.setBold( true );

    d_label = new QLabel( title, this );
    d_label->setFont( font );

    QHBoxLayout *hLayout = new QHBoxLayout;
    hLayout->setContentsMargins( 0, 0, 0, 0 );
    hLayout->setSpacing( 2 );
    hLayout->addWidget( d_number, 10 );
    hLayout->addWidget( d_wheel );

    QVBoxLayout *vLayout = new QVBoxLayout( this );
    vLayout->addLayout( hLayout, 10 );
    vLayout->addWidget( d_label, 0, Qt::AlignTop | Qt::AlignHCenter );

    connect( d_wheel, SIGNAL( valueChanged( double ) ),
        d_number, SLOT( display( double ) ) );
    connect( d_wheel, SIGNAL( valueChanged( double ) ),
        this, SIGNAL( valueChanged( double ) ) );
}

void WheelBox::setTheme( const QColor &color )
{
    d_wheel->setPalette( color );
}

QColor WheelBox::theme() const
{
    return d_wheel->palette().color( QPalette::Window );
}

void WheelBox::setValue( double value )
{
    d_wheel->setValue( value );
    d_number->display( value );
}

double WheelBox::value() const
{
    return d_wheel->value();
}
```

### `examples/oscilloscope/wheelbox.h`

```cpp
#ifndef _WHEELBOX_H_
#define _WHEELBOX_H_

#include <qwidget.h>

class QwtWheel;
class QLabel;
class QLCDNumber;

class WheelBox: public QWidget
{
    Q_OBJECT
    Q_PROPERTY( QColor theme READ theme WRITE setTheme )

public:
    WheelBox( const QString &title,
        double min, double max, double stepSize,
        QWidget *parent = NULL );

    void setTheme( const QColor & );
    QColor theme() const;

    void setUnit( const QString & );
    QString unit() const;

    void setValue( double value );
    double value() const;

Q_SIGNALS:
    double valueChanged( double );

private:
    QLCDNumber *d_number;
    QwtWheel *d_wheel;
    QLabel *d_label;

    QString d_unit;
};

#endif
```

### `examples/radio/ampfrm.cpp`

```cpp
#include "ampfrm.h"
#include <qwt_knob.h>
#include <qwt_thermo.h>
#include <qwt_round_scale_draw.h>
#include <qwt_math.h>
#include <qlayout.h>
#include <qlabel.h>
#include <qfont.h>
#include <qpen.h>
#include <qevent.h>

#if QT_VERSION < 0x040600
#define qFastSin(x) ::sin(x)
#define qFastCos(x) ::cos(x)
#endif

class Knob: public QWidget
{
public:
    Knob( const QString &title, double min, double max, QWidget *parent ):
        QWidget( parent )
    {
        d_knob = new QwtKnob( this );
        d_knob->setScale( min, max );
        d_knob->setTotalSteps( 0 ); // disable
        d_knob->setScaleMaxMajor( 10 );

        d_knob->setKnobStyle( QwtKnob::Raised );
        d_knob->setKnobWidth( 50 );
        d_knob->setBorderWidth( 2 );
        d_knob->setMarkerStyle( QwtKnob::Notch );
        d_knob->setMarkerSize( 8 );

        d_knob->scaleDraw()->setTickLength( QwtScaleDiv::MinorTick, 4 );
        d_knob->scaleDraw()->setTickLength( QwtScaleDiv::MediumTick, 4 );
        d_knob->scaleDraw()->setTickLength( QwtScaleDiv::MajorTick, 6 );

        d_label = new QLabel( title, this );
        d_label->setAlignment( Qt::AlignTop | Qt::AlignHCenter );

        setSizePolicy( QSizePolicy::MinimumExpanding,
            QSizePolicy::MinimumExpanding );
    }

    virtual QSize sizeHint() const
    {
        QSize sz1 = d_knob->sizeHint();
        QSize sz2 = d_label->sizeHint();

        const int w = qMax( sz1.width(), sz2.width() );
        const int h = sz1.height() + sz2.height();

        int off = qCeil( d_knob->scaleDraw()->extent( d_knob->font() ) );
        off -= 10; // spacing

        return QSize( w, h - off );
    }

    void setValue( double value )
    {
        d_knob->setValue( value );
    }

    double value() const
    {
        return d_knob->value();
    }

protected:
    virtual void resizeEvent( QResizeEvent *e )
    {
        const QSize sz = e->size();

        int h = d_label->sizeHint().height();

        d_label->setGeometry( 0, sz.height() - h, sz.width(), h );

        h = d_knob->sizeHint().height();
        int off = qCeil( d_knob->scaleDraw()->extent( d_knob->font() ) );
        off -= 10; // spacing

        d_knob->setGeometry( 0, d_label->pos().y() - h + off,
            sz.width(), h );
    }

private:
    QwtKnob *d_knob;
    QLabel *d_label;
};

class Thermo: public QWidget
{
public:
    Thermo( const QString &title, QWidget *parent ):
        QWidget( parent )
    {
        d_thermo = new QwtThermo( this );
        d_thermo->setPipeWidth( 6 );
        d_thermo->setScale( -40, 10 );
        d_thermo->setFillBrush( Qt::green );
        d_thermo->setAlarmBrush( Qt::red );
        d_thermo->setAlarmLevel( 0.0 );
        d_thermo->setAlarmEnabled( true );

        QLabel *label = new QLabel( title, this );
        label->setAlignment( Qt::AlignTop | Qt::AlignLeft );

        QVBoxLayout *layout = new QVBoxLayout( this );
        layout->setMargin( 0 );
        layout->setSpacing( 0 );
        layout->addWidget( d_thermo, 10 );
        layout->addWidget( label );
    }

    void setValue( double value )
    {
        d_thermo->setValue( value );
    }

private:
    QwtThermo *d_thermo;
};

AmpFrame::AmpFrame( QWidget *p ):
    QFrame( p )
{
    d_knbVolume = new Knob( "Volume", 0.0, 10.0, this );
    d_knbBalance = new Knob( "Balance", -10.0, 10.0, this );
    d_knbTreble = new Knob( "Treble", -10.0, 10.0, this );
    d_knbBass = new Knob( "Bass", -10.0, 10.0, this );

    d_thmLeft = new Thermo( "Left [dB]", this );
    d_thmRight = new Thermo( "Right [dB]", this );

    QHBoxLayout *layout = new QHBoxLayout( this );
    layout->setSpacing( 0 );
    layout->setMargin( 10 );
    layout->addWidget( d_knbVolume );
    layout->addWidget( d_knbBalance);
    layout->addWidget( d_knbTreble);
    layout->addWidget( d_knbBass );
    layout->addSpacing( 20 );
    layout->addStretch( 10 );
    layout->addWidget( d_thmLeft );
    layout->addSpacing( 10 );
    layout->addWidget( d_thmRight );

    d_knbVolume->setValue( 7.0 );
    ( void )startTimer( 50 );
}

void AmpFrame::timerEvent( QTimerEvent * )
{
    static double phs = 0;

    //
    //  This amplifier generates its own input signal...
    //

    const double sig_bass = ( 1.0 + 0.1 * d_knbBass->value() )
        * qFastSin( 13.0 * phs );
    const double sig_mid_l = qFastSin( 17.0 * phs );
    const double sig_mid_r = qFastCos( 17.5 * phs );
    const double sig_trbl_l = 0.5 * ( 1.0 + 0.1 * d_knbTreble->value() )
        * qFastSin( 35.0 * phs );
    const double sig_trbl_r = 0.5 * ( 1.0 + 0.1 * d_knbTreble->value() )
        * qFastSin( 34.0 * phs );

    double sig_l = 0.05 * d_master * d_knbVolume->value()
        * qwtSqr( sig_bass + sig_mid_l + sig_trbl_l );
    double sig_r = 0.05 * d_master * d_knbVolume->value()
        * qwtSqr( sig_bass + sig_mid_r + sig_trbl_r );

    double balance = 0.1 * d_knbBalance->value();
    if ( balance > 0 )
        sig_l *= ( 1.0 - balance );
    else
        sig_r *= ( 1.0 + balance );

    if ( sig_l > 0.01 )
        sig_l = 20.0 * log10( sig_l );
    else
        sig_l = -40.0;

    if ( sig_r > 0.01 )
        sig_r = 20.0 * log10( sig_r );
    else
        sig_r = - 40.0;

    d_thmLeft->setValue( sig_l );
    d_thmRight->setValue( sig_r );

    phs += M_PI / 100;
    if ( phs > M_PI )
        phs = 0;
}

void AmpFrame::setMaster( double v )
{
    d_master = v;
}
```

### `examples/radio/ampfrm.h`

```cpp
#include <qframe.h>

class Knob;
class Thermo;

class AmpFrame : public QFrame
{
    Q_OBJECT
public:
    AmpFrame( QWidget * );

public Q_SLOTS:
    void setMaster( double v );

protected:
    void timerEvent( QTimerEvent * );

private:
    Knob *d_knbVolume;
    Knob *d_knbBalance;
    Knob *d_knbTreble;
    Knob *d_knbBass;
    Thermo *d_thmLeft;
    Thermo *d_thmRight;
    double d_master;
};
```

### `examples/radio/mainwindow.cpp`

```cpp
#include <qlayout.h>
#include "tunerfrm.h"
#include "ampfrm.h"
#include "mainwindow.h"

MainWindow::MainWindow():
    QWidget()
{
    TunerFrame *frmTuner = new TunerFrame( this );
    frmTuner->setFrameStyle( QFrame::Panel | QFrame::Raised );

    AmpFrame *frmAmp = new AmpFrame( this );
    frmAmp->setFrameStyle( QFrame::Panel | QFrame::Raised );

    QVBoxLayout *layout = new QVBoxLayout( this );
    layout->setMargin( 0 );
    layout->setSpacing( 0 );
    layout->addWidget( frmTuner );
    layout->addWidget( frmAmp );

    connect( frmTuner, SIGNAL( fieldChanged( double ) ),
        frmAmp, SLOT( setMaster( double ) ) );

    frmTuner->setFreq( 90.0 );

    setPalette( QPalette( QColor( 192, 192, 192 ) ) );
    updateGradient();
}

void MainWindow::resizeEvent( QResizeEvent * )
{
    // Qt 4.7.1: QGradient::StretchToDeviceMode is buggy on X11
    updateGradient();
}

void MainWindow::updateGradient()
{
    QPalette pal = palette();

    const QColor buttonColor = pal.color( QPalette::Button );
    const QColor midLightColor = pal.color( QPalette::Midlight );

    QLinearGradient gradient( rect().topLeft(), rect().topRight() );
    gradient.setColorAt( 0.0, midLightColor );
    gradient.setColorAt( 0.7, buttonColor );
    gradient.setColorAt( 1.0, buttonColor );

    pal.setBrush( QPalette::Window, gradient );
    setPalette( pal );
}
```

### `examples/radio/mainwindow.h`

```cpp
#include <qwidget.h>

class MainWindow : public QWidget
{
public:
    MainWindow();

protected:
    virtual void resizeEvent( QResizeEvent * );

private:
    void updateGradient();
};
```

### `examples/radio/radio.cpp`

```cpp
#include <qapplication.h>
#include "mainwindow.h"

int main ( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWindow w;
    w.show();

    return a.exec();
}
```

### `examples/radio/tunerfrm.cpp`

```cpp
#include <qlayout.h>
#include <qlabel.h>
#include <qwt_wheel.h>
#include <qwt_slider.h>
#include <qwt_thermo.h>
#include <qwt_math.h>
#include "tunerfrm.h"

#if QT_VERSION < 0x040600
#define qFastSin(x) ::sin(x)
#define qFastCos(x) ::cos(x)
#endif

class TuningThermo: public QWidget
{
public:
    TuningThermo( QWidget *parent ):
        QWidget( parent )
    {
        d_thermo = new QwtThermo( this );
        d_thermo->setOrientation( Qt::Horizontal );
        d_thermo->setScalePosition( QwtThermo::NoScale );
        d_thermo->setScale( 0.0, 1.0 );
        d_thermo->setFillBrush( Qt::green );

        QLabel *label = new QLabel( "Tuning", this );
        label->setAlignment( Qt::AlignCenter );

        QVBoxLayout *layout = new QVBoxLayout( this );
        layout->setMargin( 0 );
        layout->addWidget( d_thermo );
        layout->addWidget( label );

        setFixedWidth( 3 * label->sizeHint().width() );
    }

    void setValue( double value )
    {
        d_thermo->setValue( value );
    }

private:
    QwtThermo *d_thermo;
};

TunerFrame::TunerFrame( QWidget *parent ):
    QFrame( parent )
{
    const double freqMin = 87.5;
    const double freqMax = 108;

    d_sliderFrequency = new QwtSlider( this );
    d_sliderFrequency->setOrientation( Qt::Horizontal );
    d_sliderFrequency->setScalePosition( QwtSlider::TrailingScale );
    d_sliderFrequency->setScale( freqMin, freqMax );
    d_sliderFrequency->setTotalSteps( 
        qRound( ( freqMax - freqMin ) / 0.01 ) );
    d_sliderFrequency->setSingleSteps( 1 );
    d_sliderFrequency->setPageSteps( 10 );
    d_sliderFrequency->setScaleMaxMinor( 5 );
    d_sliderFrequency->setScaleMaxMajor( 12 );
    d_sliderFrequency->setHandleSize( QSize( 80, 20 ) );
    d_sliderFrequency->setBorderWidth( 1 );

    d_thermoTune = new TuningThermo( this );

    d_wheelFrequency = new QwtWheel( this );
    d_wheelFrequency->setMass( 0.5 );
    d_wheelFrequency->setRange( 87.5, 108 );
    d_wheelFrequency->setSingleStep( 0.01 );
    d_wheelFrequency->setPageStepCount( 10 );
    d_wheelFrequency->setTotalAngle( 3600.0 );
    d_wheelFrequency->setFixedHeight( 30 );


    connect( d_wheelFrequency, SIGNAL( valueChanged( double ) ), SLOT( adjustFreq( double ) ) );
    connect( d_sliderFrequency, SIGNAL( valueChanged( double ) ), SLOT( adjustFreq( double ) ) );

    QVBoxLayout *mainLayout = new QVBoxLayout( this );
    mainLayout->setMargin( 10 );
    mainLayout->setSpacing( 5 );
    mainLayout->addWidget( d_sliderFrequency );

    QHBoxLayout *hLayout = new QHBoxLayout;
    hLayout->setMargin( 0 );
    hLayout->addWidget( d_thermoTune, 0 );
    hLayout->addStretch( 5 );
    hLayout->addWidget( d_wheelFrequency, 2 );

    mainLayout->addLayout( hLayout );
}

void TunerFrame::adjustFreq( double frq )
{
    const double factor = 13.0 / ( 108 - 87.5 );

    const double x = ( frq - 87.5 ) * factor;
    const double field = qwtSqr( qFastSin( x ) * qFastCos( 4.0 * x ) );

    d_thermoTune->setValue( field );

    if ( d_sliderFrequency->value() != frq )
        d_sliderFrequency->setValue( frq );
    if ( d_wheelFrequency->value() != frq )
        d_wheelFrequency->setValue( frq );

    Q_EMIT fieldChanged( field );
}

void TunerFrame::setFreq( double frq )
{
    d_wheelFrequency->setValue( frq );
}
```

### `examples/radio/tunerfrm.h`

```cpp
#include <qframe.h>

class QwtWheel;
class QwtSlider;
class TuningThermo;

class TunerFrame : public QFrame
{
    Q_OBJECT
public:
    TunerFrame( QWidget *p );

Q_SIGNALS:
    void fieldChanged( double f );

public Q_SLOTS:
    void setFreq( double frq );

private Q_SLOTS:
    void adjustFreq( double frq );

private:
    QwtWheel *d_wheelFrequency;
    TuningThermo *d_thermoTune;
    QwtSlider *d_sliderFrequency;
};
```

### `examples/rasterview/main.cpp`

```cpp
#include <qapplication.h>
#include <qmainwindow.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include <qcombobox.h>
#include <qlabel.h>
#include "plot.h"

class MainWindow: public QMainWindow
{
public:
    MainWindow( QWidget * = NULL );
};

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    Plot *plot = new Plot( this );
    setCentralWidget( plot );

    QToolBar *toolBar = new QToolBar( this );

    QComboBox *rasterBox = new QComboBox( toolBar );
    rasterBox->addItem( "Wikipedia" );

    toolBar->addWidget( new QLabel( "Data ", toolBar ) );
    toolBar->addWidget( rasterBox );
    toolBar->addSeparator();

    QComboBox *modeBox = new QComboBox( toolBar );
    modeBox->addItem( "Nearest Neighbour" );
    modeBox->addItem( "Bilinear Interpolation" );

    toolBar->addWidget( new QLabel( "Resampling ", toolBar ) );
    toolBar->addWidget( modeBox );

    toolBar->addSeparator();

    QToolButton *btnExport = new QToolButton( toolBar );
    btnExport->setText( "Export" );
    btnExport->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    toolBar->addWidget( btnExport );

    addToolBar( toolBar );

    connect( modeBox, SIGNAL( activated( int ) ), plot, SLOT( setResampleMode( int ) ) );
    connect( btnExport, SIGNAL( clicked() ), plot, SLOT( exportPlot() ) );
}

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWindow mainWindow;
    mainWindow.resize( 600, 400 );
    mainWindow.show();

    return a.exec();
}
```

### `examples/rasterview/plot.cpp`

```cpp
#include "plot.h"
#include <qwt_color_map.h>
#include <qwt_plot_spectrogram.h>
#include <qwt_plot_layout.h>
#include <qwt_matrix_raster_data.h>
#include <qwt_scale_widget.h>
#include <qwt_plot_magnifier.h>
#include <qwt_plot_panner.h>
#include <qwt_plot_renderer.h>
#include <qwt_plot_grid.h>
#include <qwt_plot_canvas.h>

class RasterData: public QwtMatrixRasterData
{
public:
    RasterData()
    {
        const double matrix[] =
        {
            1, 2, 4, 1,
            6, 3, 5, 2,
            4, 2, 1, 5,
            5, 4, 2, 3
        };

        QVector<double> values;
        for ( uint i = 0; i < sizeof( matrix ) / sizeof( double ); i++ )
            values += matrix[i];

        const int numColumns = 4;
        setValueMatrix( values, numColumns );

        setInterval( Qt::XAxis,
            QwtInterval( -0.5, 3.5, QwtInterval::ExcludeMaximum ) );
        setInterval( Qt::YAxis,
            QwtInterval( -0.5, 3.5, QwtInterval::ExcludeMaximum ) );
        setInterval( Qt::ZAxis, QwtInterval( 1.0, 6.0 ) );
    }
};

class ColorMap: public QwtLinearColorMap
{
public:
    ColorMap():
        QwtLinearColorMap( Qt::darkBlue, Qt::darkRed )
    {
        addColorStop( 0.2, Qt::blue );
        addColorStop( 0.4, Qt::cyan );
        addColorStop( 0.6, Qt::yellow );
        addColorStop( 0.8, Qt::red );
    }
};

Plot::Plot( QWidget *parent ):
    QwtPlot( parent )
{
    QwtPlotCanvas *canvas = new QwtPlotCanvas();
    canvas->setBorderRadius( 10 );
    setCanvas( canvas );

#if 0
    QwtPlotGrid *grid = new QwtPlotGrid();
    grid->setPen( Qt::DotLine );
    grid->attach( this );
#endif

    d_spectrogram = new QwtPlotSpectrogram();
    d_spectrogram->setRenderThreadCount( 0 ); // use system specific thread count

    d_spectrogram->setColorMap( new ColorMap() );

    d_spectrogram->setData( new RasterData() );
    d_spectrogram->attach( this );

    const QwtInterval zInterval = d_spectrogram->data()->interval( Qt::ZAxis );
    // A color bar on the right axis
    QwtScaleWidget *rightAxis = axisWidget( QwtPlot::yRight );
    rightAxis->setColorBarEnabled( true );
    rightAxis->setColorBarWidth( 40 );
    rightAxis->setColorMap( zInterval, new ColorMap() );

    setAxisScale( QwtPlot::yRight, zInterval.minValue(), zInterval.maxValue() );
    enableAxis( QwtPlot::yRight );

    plotLayout()->setAlignCanvasToScales( true );

    setAxisScale( QwtPlot::xBottom, 0.0, 3.0 );
    setAxisMaxMinor( QwtPlot::xBottom, 0 );
    setAxisScale( QwtPlot::yLeft, 0.0, 3.0 );
    setAxisMaxMinor( QwtPlot::yLeft, 0 );

    QwtPlotMagnifier *magnifier = new QwtPlotMagnifier( canvas );
    magnifier->setAxisEnabled( QwtPlot::yRight, false );

    QwtPlotPanner *panner = new QwtPlotPanner( canvas );
    panner->setAxisEnabled( QwtPlot::yRight, false );
}

void Plot::exportPlot()
{
    QwtPlotRenderer renderer;
    renderer.exportTo( this, "rasterview.pdf" );
}

void Plot::setResampleMode( int mode )
{
    RasterData *data = static_cast<RasterData *>( d_spectrogram->data() );
    data->setResampleMode(
        static_cast<QwtMatrixRasterData::ResampleMode>( mode ) );

    replot();
}
```

### `examples/rasterview/plot.h`

```cpp
#include <qwt_plot.h>
#include <qwt_plot_spectrogram.h>

class Plot: public QwtPlot
{
    Q_OBJECT

public:
    Plot( QWidget * = NULL );

public Q_SLOTS:
    void exportPlot();
    void setResampleMode( int );

private:
    QwtPlotSpectrogram *d_spectrogram;
};
```

### `examples/realtime/incrementalplot.cpp`

```cpp
#include <qwt_plot.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_curve.h>
#include <qwt_symbol.h>
#include <qwt_plot_directpainter.h>
#include <qwt_painter.h>
#include "incrementalplot.h"
#include <qpaintengine.h>

class CurveData: public QwtArraySeriesData<QPointF>
{
public:
    CurveData()
    {
    }

    virtual QRectF boundingRect() const
    {
        if ( d_boundingRect.width() < 0.0 )
            d_boundingRect = qwtBoundingRect( *this );

        return d_boundingRect;
    }

    inline void append( const QPointF &point )
    {
        d_samples += point;
    }

    void clear()
    {
        d_samples.clear();
        d_samples.squeeze();
        d_boundingRect = QRectF( 0.0, 0.0, -1.0, -1.0 );
    }
};

IncrementalPlot::IncrementalPlot( QWidget *parent ):
    QwtPlot( parent ),
    d_curve( NULL )
{
    d_directPainter = new QwtPlotDirectPainter( this );

    if ( QwtPainter::isX11GraphicsSystem() )
    {
#if QT_VERSION < 0x050000
        canvas()->setAttribute( Qt::WA_PaintOutsidePaintEvent, true );
#endif
        canvas()->setAttribute( Qt::WA_PaintOnScreen, true );
    }

    d_curve = new QwtPlotCurve( "Test Curve" );
    d_curve->setData( new CurveData() );
    showSymbols( true );

    d_curve->attach( this );

    setAutoReplot( false );
}

IncrementalPlot::~IncrementalPlot()
{
    delete d_curve;
}

void IncrementalPlot::appendPoint( const QPointF &point )
{
    CurveData *data = static_cast<CurveData *>( d_curve->data() );
    data->append( point );

    const bool doClip = !canvas()->testAttribute( Qt::WA_PaintOnScreen );
    if ( doClip )
    {
        /*
           Depending on the platform setting a clip might be an important
           performance issue. F.e. for Qt Embedded this reduces the
           part of the backing store that has to be copied out - maybe
           to an unaccelerated frame buffer device.
         */
        const QwtScaleMap xMap = canvasMap( d_curve->xAxis() );
        const QwtScaleMap yMap = canvasMap( d_curve->yAxis() );

        QRegion clipRegion;

        const QSize symbolSize = d_curve->symbol()->size();
        QRect r( 0, 0, symbolSize.width() + 2, symbolSize.height() + 2 );

        const QPointF center =
            QwtScaleMap::transform( xMap, yMap, point );
        r.moveCenter( center.toPoint() );
        clipRegion += r;

        d_directPainter->setClipRegion( clipRegion );
    }

    d_directPainter->drawSeries( d_curve,
        data->size() - 1, data->size() - 1 );
}

void IncrementalPlot::clearPoints()
{
    CurveData *data = static_cast<CurveData *>( d_curve->data() );
    data->clear();

    replot();
}

void IncrementalPlot::showSymbols( bool on )
{
    if ( on )
    {
        d_curve->setStyle( QwtPlotCurve::NoCurve );
        d_curve->setSymbol( new QwtSymbol( QwtSymbol::XCross,
            Qt::NoBrush, QPen( Qt::white ), QSize( 4, 4 ) ) );
    }
    else
    {
        d_curve->setPen( Qt::white );
        d_curve->setStyle( QwtPlotCurve::Dots );
        d_curve->setSymbol( NULL );
    }

    replot();
}
```

### `examples/realtime/incrementalplot.h`

```cpp
#ifndef _INCREMENTALPLOT_H_
#define _INCREMENTALPLOT_H_ 1

#include <qwt_plot.h>

class QwtPlotCurve;
class QwtPlotDirectPainter;

class IncrementalPlot : public QwtPlot
{
    Q_OBJECT

public:
    IncrementalPlot( QWidget *parent = NULL );
    virtual ~IncrementalPlot();

    void appendPoint( const QPointF & );
    void clearPoints();

public Q_SLOTS:
    void showSymbols( bool );

private:
    QwtPlotCurve *d_curve;
    QwtPlotDirectPainter *d_directPainter;
};

#endif // _INCREMENTALPLOT_H_
```

### `examples/realtime/main.cpp`

```cpp
#include <qapplication.h>
#include "mainwindow.h"

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWindow w;
    w.show();

    return a.exec();
}
```

### `examples/realtime/mainwindow.cpp`

```cpp
#include <qlabel.h>
#include <qlayout.h>
#include <qstatusbar.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include <qspinbox.h>
#include <qcheckbox.h>
#include <qwhatsthis.h>
#include <qpixmap.h>
#include "randomplot.h"
#include "mainwindow.h"
#include "start.xpm"
#include "clear.xpm"

class MyToolBar: public QToolBar
{
public:
    MyToolBar( MainWindow *parent ):
        QToolBar( parent )
    {
    }
    void addSpacing( int spacing )
    {
        QLabel *label = new QLabel( this );
        addWidget( label );
        label->setFixedWidth( spacing );
    }
};

class Counter: public QWidget
{
public:
    Counter( QWidget *parent,
            const QString &prefix, const QString &suffix,
            int min, int max, int step ):
        QWidget( parent )
    {
        QHBoxLayout *layout = new QHBoxLayout( this );

        if ( !prefix.isEmpty() )
            layout->addWidget( new QLabel( prefix + " ", this ) );

        d_counter = new QSpinBox( this );
        d_counter->setRange( min, max );
        d_counter->setSingleStep( step );
        layout->addWidget( d_counter );

        if ( !suffix.isEmpty() )
            layout->addWidget( new QLabel( QString( " " ) + suffix, this ) );
    }

    void setValue( int value ) { d_counter->setValue( value ); }
    int value() const { return d_counter->value(); }

private:
    QSpinBox *d_counter;
};

MainWindow::MainWindow()
{
    addToolBar( toolBar() );
#ifndef QT_NO_STATUSBAR
    ( void )statusBar();
#endif

    d_plot = new RandomPlot( this );
    const int margin = 4;
    d_plot->setContentsMargins( margin, margin, margin, margin );

    setCentralWidget( d_plot );

    connect( d_startAction, SIGNAL( toggled( bool ) ), this, SLOT( appendPoints( bool ) ) );
    connect( d_clearAction, SIGNAL( triggered() ), d_plot, SLOT( clear() ) );
    connect( d_symbolType, SIGNAL( toggled( bool ) ), d_plot, SLOT( showSymbols( bool ) ) );
    connect( d_plot, SIGNAL( running( bool ) ), this, SLOT( showRunning( bool ) ) );
    connect( d_plot, SIGNAL( elapsed( int ) ), this, SLOT( showElapsed( int ) ) );

    initWhatsThis();

    setContextMenuPolicy( Qt::NoContextMenu );
}

QToolBar *MainWindow::toolBar()
{
    MyToolBar *toolBar = new MyToolBar( this );

    toolBar->setAllowedAreas( Qt::TopToolBarArea | Qt::BottomToolBarArea );
    setToolButtonStyle( Qt::ToolButtonTextUnderIcon );

    d_startAction = new QAction( QPixmap( start_xpm ), "Start", toolBar );
    d_startAction->setCheckable( true );
    d_clearAction = new QAction( QPixmap( clear_xpm ), "Clear", toolBar );
    QAction *whatsThisAction = QWhatsThis::createAction( toolBar );
    whatsThisAction->setText( "Help" );

    toolBar->addAction( d_startAction );
    toolBar->addAction( d_clearAction );
    toolBar->addAction( whatsThisAction );

    setIconSize( QSize( 22, 22 ) );

    QWidget *hBox = new QWidget( toolBar );

    d_symbolType = new QCheckBox( "Symbols", hBox );
    d_symbolType->setChecked( true );

    d_randomCount =
        new Counter( hBox, "Points", QString::null, 1, 100000, 100 );
    d_randomCount->setValue( 1000 );

    d_timerCount = new Counter( hBox, "Delay", "ms", 0, 100000, 100 );
    d_timerCount->setValue( 0 );

    QHBoxLayout *layout = new QHBoxLayout( hBox );
    layout->setMargin( 0 );
    layout->setSpacing( 0 );
    layout->addSpacing( 10 );
    layout->addWidget( new QWidget( hBox ), 10 ); // spacer
    layout->addWidget( d_symbolType );
    layout->addSpacing( 5 );
    layout->addWidget( d_randomCount );
    layout->addSpacing( 5 );
    layout->addWidget( d_timerCount );

    showRunning( false );

    toolBar->addWidget( hBox );

    return toolBar;
}

void MainWindow::appendPoints( bool on )
{
    if ( on )
        d_plot->append( d_timerCount->value(),
                        d_randomCount->value() );
    else
        d_plot->stop();
}

void MainWindow::showRunning( bool running )
{
    d_randomCount->setEnabled( !running );
    d_timerCount->setEnabled( !running );
    d_startAction->setChecked( running );
    d_startAction->setText( running ? "Stop" : "Start" );
}

void MainWindow::showElapsed( int ms )
{
    QString text;
    text.setNum( ms );
    text += " ms";

    statusBar()->showMessage( text );
}

void MainWindow::initWhatsThis()
{
    const char *text1 =
        "Zooming is enabled until the selected area gets "
        "too small for the significance on the axes.\n\n"
        "You can zoom in using the left mouse button.\n"
        "The middle mouse button is used to go back to the "
        "previous zoomed area.\n"
        "The right mouse button is used to unzoom completely.";

    const char *text2 =
        "Number of random points that will be generated.";

    const char *text3 =
        "Delay between the generation of two random points.";

    const char *text4 =
        "Start generation of random points.\n\n"
        "The intention of this example is to show how to implement "
        "growing curves. The points will be generated and displayed "
        "one after the other.\n"
        "To check the performance, a small delay and a large number "
        "of points are useful. To watch the curve growing, a delay "
        " > 300 ms and less points are better.\n"
        "To inspect the curve, stacked zooming is implemented using the "
        "mouse buttons on the plot.";

    const char *text5 = "Remove all points.";

    d_plot->setWhatsThis( text1 );
    d_randomCount->setWhatsThis( text2 );
    d_timerCount->setWhatsThis( text3 );
    d_startAction->setWhatsThis( text4 );
    d_clearAction->setWhatsThis( text5 );
}
```

### `examples/realtime/mainwindow.h`

```cpp
#ifndef _MAINWINDOW_H_
#define _MAINWINDOW_H_ 1

#include <qmainwindow.h>
#include <qaction.h>

class QSpinBox;
class QPushButton;
class RandomPlot;
class Counter;
class QCheckBox;

class MainWindow: public QMainWindow
{
    Q_OBJECT
public:
    MainWindow();

private Q_SLOTS:
    void showRunning( bool );
    void appendPoints( bool );
    void showElapsed( int );

private:
    QToolBar *toolBar();
    void initWhatsThis();

private:
    Counter *d_randomCount;
    Counter *d_timerCount;
    QCheckBox *d_symbolType;
    QAction *d_startAction;
    QAction *d_clearAction;
    RandomPlot *d_plot;
};

#endif
```

### `examples/realtime/randomplot.cpp`

```cpp
#include <qglobal.h>
#include <qtimer.h>
#include <qwt_plot_grid.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_layout.h>
#include <qwt_scale_widget.h>
#include <qwt_scale_draw.h>
#include "scrollzoomer.h"
#include "randomplot.h"

const unsigned int c_rangeMax = 1000;

class Zoomer: public ScrollZoomer
{
public:
    Zoomer( QWidget *canvas ):
        ScrollZoomer( canvas )
    {
#if 0
        setRubberBandPen( QPen( Qt::red, 2, Qt::DotLine ) );
#else
        setRubberBandPen( QPen( Qt::red ) );
#endif
    }

    virtual QwtText trackerTextF( const QPointF &pos ) const
    {
        QColor bg( Qt::white );

        QwtText text = QwtPlotZoomer::trackerTextF( pos );
        text.setBackgroundBrush( QBrush( bg ) );
        return text;
    }

    virtual void rescale()
    {
        QwtScaleWidget *scaleWidget = plot()->axisWidget( yAxis() );
        QwtScaleDraw *sd = scaleWidget->scaleDraw();

        double minExtent = 0.0;
        if ( zoomRectIndex() > 0 )
        {
            // When scrolling in vertical direction
            // the plot is jumping in horizontal direction
            // because of the different widths of the labels
            // So we better use a fixed extent.

            minExtent = sd->spacing() + sd->maxTickLength() + 1;
            minExtent += sd->labelSize(
                scaleWidget->font(), c_rangeMax ).width();
        }

        sd->setMinimumExtent( minExtent );

        ScrollZoomer::rescale();
    }
};

RandomPlot::RandomPlot( QWidget *parent ):
    IncrementalPlot( parent ),
    d_timer( 0 ),
    d_timerCount( 0 )
{
    setFrameStyle( QFrame::NoFrame );
    setLineWidth( 0 );

    plotLayout()->setAlignCanvasToScales( true );

    QwtPlotGrid *grid = new QwtPlotGrid;
    grid->setMajorPen( Qt::gray, 0, Qt::DotLine );
    grid->attach( this );

    setCanvasBackground( QColor( 29, 100, 141 ) ); // nice blue

    setAxisScale( xBottom, 0, c_rangeMax );
    setAxisScale( yLeft, 0, c_rangeMax );

    replot();

    // enable zooming

    ( void ) new Zoomer( canvas() );
}

QSize RandomPlot::sizeHint() const
{
    return QSize( 540, 400 );
}

void RandomPlot::appendPoint()
{
    double x = qrand() % c_rangeMax;
    x += ( qrand() % 100 ) / 100;

    double y = qrand() % c_rangeMax;
    y += ( qrand() % 100 ) / 100;

    IncrementalPlot::appendPoint( QPointF( x, y ) );

    if ( --d_timerCount <= 0 )
        stop();
}

void RandomPlot::append( int timeout, int count )
{
    if ( !d_timer )
    {
        d_timer = new QTimer( this );
        connect( d_timer, SIGNAL( timeout() ), SLOT( appendPoint() ) );
    }

    d_timerCount = count;

    Q_EMIT running( true );
    d_timeStamp.start();

    QwtPlotCanvas *plotCanvas = qobject_cast<QwtPlotCanvas *>( canvas() );
    plotCanvas->setPaintAttribute( QwtPlotCanvas::BackingStore, false );

    d_timer->start( timeout );
}

void RandomPlot::stop()
{
    Q_EMIT elapsed( d_timeStamp.elapsed() );

    if ( d_timer )
    {
        d_timer->stop();
        Q_EMIT running( false );
    }

    QwtPlotCanvas *plotCanvas = qobject_cast<QwtPlotCanvas *>( canvas() );
    plotCanvas->setPaintAttribute( QwtPlotCanvas::BackingStore, true );
}

void RandomPlot::clear()
{
    clearPoints();
    replot();
}
```

### `examples/realtime/randomplot.h`

```cpp
#ifndef _RANDOMPLOT_H_
#define _RANDOMPLOT_H_ 1

#include "incrementalplot.h"
#include <qdatetime.h>

class QTimer;

class RandomPlot: public IncrementalPlot
{
    Q_OBJECT

public:
    RandomPlot( QWidget *parent );

    virtual QSize sizeHint() const;

Q_SIGNALS:
    void running( bool );
    void elapsed( int ms );

public Q_SLOTS:
    void clear();
    void stop();
    void append( int timeout, int count );

private Q_SLOTS:
    void appendPoint();

private:
    void initCurve();

    QTimer *d_timer;
    int d_timerCount;

    QTime d_timeStamp;
};

#endif // _RANDOMPLOT_H_
```

### `examples/realtime/scrollbar.cpp`

```cpp
#include <qstyle.h>
#include <qstyleoption.h>
#include "scrollbar.h"

ScrollBar::ScrollBar( QWidget * parent ):
    QScrollBar( parent )
{
    init();
}

ScrollBar::ScrollBar( Qt::Orientation o,
        QWidget *parent ):
    QScrollBar( o, parent )
{
    init();
}

ScrollBar::ScrollBar( double minBase, double maxBase,
        Qt::Orientation o, QWidget *parent ):
    QScrollBar( o, parent )
{
    init();
    setBase( minBase, maxBase );
    moveSlider( minBase, maxBase );
}

void ScrollBar::init()
{
    d_inverted = orientation() == Qt::Vertical;
    d_baseTicks = 1000000;
    d_minBase = 0.0;
    d_maxBase = 1.0;
    moveSlider( d_minBase, d_maxBase );

    connect( this, SIGNAL( sliderMoved( int ) ), SLOT( catchSliderMoved( int ) ) );
    connect( this, SIGNAL( valueChanged( int ) ), SLOT( catchValueChanged( int ) ) );
}

void ScrollBar::setInverted( bool inverted )
{
    if ( d_inverted != inverted )
    {
        d_inverted = inverted;
        moveSlider( minSliderValue(), maxSliderValue() );
    }
}

bool ScrollBar::isInverted() const
{
    return d_inverted;
}

void ScrollBar::setBase( double min, double max )
{
    if ( min != d_minBase || max != d_maxBase )
    {
        d_minBase = min;
        d_maxBase = max;

        moveSlider( minSliderValue(), maxSliderValue() );
    }
}

void ScrollBar::moveSlider( double min, double max )
{
    const int sliderTicks = qRound( ( max - min ) /
        ( d_maxBase - d_minBase ) * d_baseTicks );

    // setRange initiates a valueChanged of the scrollbars
    // in some situations. So we block
    // and unblock the signals.

    blockSignals( true );

    setRange( sliderTicks / 2, d_baseTicks - sliderTicks / 2 );
    int steps = sliderTicks / 200;
    if ( steps <= 0 )
        steps = 1;

    setSingleStep( steps );
    setPageStep( sliderTicks );

    int tick = mapToTick( min + ( max - min ) / 2 );
    if ( isInverted() )
        tick = d_baseTicks - tick;

    setSliderPosition( tick );
    blockSignals( false );
}

double ScrollBar::minBaseValue() const
{
    return d_minBase;
}

double ScrollBar::maxBaseValue() const
{
    return d_maxBase;
}

void ScrollBar::sliderRange( int value, double &min, double &max ) const
{
    if ( isInverted() )
        value = d_baseTicks - value;

    const int visibleTicks = pageStep();

    min = mapFromTick( value - visibleTicks / 2 );
    max = mapFromTick( value + visibleTicks / 2 );
}

double ScrollBar::minSliderValue() const
{
    double min, dummy;
    sliderRange( value(), min, dummy );

    return min;
}

double ScrollBar::maxSliderValue() const
{
    double max, dummy;
    sliderRange( value(), dummy, max );

    return max;
}

int ScrollBar::mapToTick( double v ) const
{
    const double pos = ( v - d_minBase ) / ( d_maxBase - d_minBase ) * d_baseTicks;
    return static_cast<int>( pos );
}

double ScrollBar::mapFromTick( int tick ) const
{
    return d_minBase + ( d_maxBase - d_minBase ) * tick / d_baseTicks;
}

void ScrollBar::catchValueChanged( int value )
{
    double min, max;
    sliderRange( value, min, max );
    Q_EMIT valueChanged( orientation(), min, max );
}

void ScrollBar::catchSliderMoved( int value )
{
    double min, max;
    sliderRange( value, min, max );
    Q_EMIT sliderMoved( orientation(), min, max );
}

int ScrollBar::extent() const
{
    QStyleOptionSlider opt;
    opt.init( this );
    opt.subControls = QStyle::SC_None;
    opt.activeSubControls = QStyle::SC_None;
    opt.orientation = orientation();
    opt.minimum = minimum();
    opt.maximum = maximum();
    opt.sliderPosition = sliderPosition();
    opt.sliderValue = value();
    opt.singleStep = singleStep();
    opt.pageStep = pageStep();
    opt.upsideDown = invertedAppearance();
    if ( orientation() == Qt::Horizontal )
        opt.state |= QStyle::State_Horizontal;
    return style()->pixelMetric( QStyle::PM_ScrollBarExtent, &opt, this );
}
```

### `examples/realtime/scrollbar.h`

```cpp
#ifndef _SCROLLBAR_H
#define _SCROLLBAR_H 1

#include <qscrollbar.h>

class ScrollBar: public QScrollBar
{
    Q_OBJECT

public:
    ScrollBar( QWidget *parent = NULL );
    ScrollBar( Qt::Orientation, QWidget *parent = NULL );
    ScrollBar( double minBase, double maxBase,
        Qt::Orientation o, QWidget *parent = NULL );

    void setInverted( bool );
    bool isInverted() const;

    double minBaseValue() const;
    double maxBaseValue() const;

    double minSliderValue() const;
    double maxSliderValue() const;

    int extent() const;

Q_SIGNALS:
    void sliderMoved( Qt::Orientation, double, double );
    void valueChanged( Qt::Orientation, double, double );

public Q_SLOTS:
    virtual void setBase( double min, double max );
    virtual void moveSlider( double min, double max );

protected:
    void sliderRange( int value, double &min, double &max ) const;
    int mapToTick( double ) const;
    double mapFromTick( int ) const;

private Q_SLOTS:
    void catchValueChanged( int value );
    void catchSliderMoved( int value );

private:
    void init();

    bool d_inverted;
    double d_minBase;
    double d_maxBase;
    int d_baseTicks;
};

#endif
```

### `examples/realtime/scrollzoomer.cpp`

```cpp
#include <qevent.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_layout.h>
#include <qwt_scale_engine.h>
#include <qwt_scale_widget.h>
#include "scrollbar.h"
#include "scrollzoomer.h"

class ScrollData
{
public:
    ScrollData():
        scrollBar( NULL ),
        position( ScrollZoomer::OppositeToScale ),
        mode( Qt::ScrollBarAsNeeded )
    {
    }

    ~ScrollData()
    {
        delete scrollBar;
    }

    ScrollBar *scrollBar;
    ScrollZoomer::ScrollBarPosition position;
    Qt::ScrollBarPolicy mode;
};

ScrollZoomer::ScrollZoomer( QWidget *canvas ):
    QwtPlotZoomer( canvas ),
    d_cornerWidget( NULL ),
    d_hScrollData( NULL ),
    d_vScrollData( NULL ),
    d_inZoom( false )
{
    for ( int axis = 0; axis < QwtPlot::axisCnt; axis++ )
        d_alignCanvasToScales[ axis ] = false;

    if ( !canvas )
        return;

    d_hScrollData = new ScrollData;
    d_vScrollData = new ScrollData;
}

ScrollZoomer::~ScrollZoomer()
{
    delete d_cornerWidget;
    delete d_vScrollData;
    delete d_hScrollData;
}

void ScrollZoomer::rescale()
{
    QwtScaleWidget *xScale = plot()->axisWidget( xAxis() );
    QwtScaleWidget *yScale = plot()->axisWidget( yAxis() );

    if ( zoomRectIndex() <= 0 )
    {
        if ( d_inZoom )
        {
            xScale->setMinBorderDist( 0, 0 );
            yScale->setMinBorderDist( 0, 0 );

            QwtPlotLayout *layout = plot()->plotLayout();

            for ( int axis = 0; axis < QwtPlot::axisCnt; axis++ )
                layout->setAlignCanvasToScale( axis, d_alignCanvasToScales[ axis ] );

            d_inZoom = false;
        }
    }
    else
    {
        if ( !d_inZoom )
        {
            /*
             We set a minimum border distance.
             Otherwise the canvas size changes when scrolling,
             between situations where the major ticks are at
             the canvas borders (requiring extra space for the label)
             and situations where all labels can be painted below/top
             or left/right of the canvas.
             */
            int start, end;

            xScale->getBorderDistHint( start, end );
            xScale->setMinBorderDist( start, end );

            yScale->getBorderDistHint( start, end );
            yScale->setMinBorderDist( start, end );

            QwtPlotLayout *layout = plot()->plotLayout();
            for ( int axis = 0; axis < QwtPlot::axisCnt; axis++ )
            {
                d_alignCanvasToScales[axis] = 
                    layout->alignCanvasToScale( axis );
            }

            layout->setAlignCanvasToScales( false );

            d_inZoom = true;
        }
    }

    QwtPlotZoomer::rescale();
    updateScrollBars();
}

ScrollBar *ScrollZoomer::scrollBar( Qt::Orientation orientation )
{
    ScrollBar *&sb = ( orientation == Qt::Vertical )
        ? d_vScrollData->scrollBar : d_hScrollData->scrollBar;

    if ( sb == NULL )
    {
        sb = new ScrollBar( orientation, canvas() );
        sb->hide();
        connect( sb,
            SIGNAL( valueChanged( Qt::Orientation, double, double ) ),
            SLOT( scrollBarMoved( Qt::Orientation, double, double ) ) );
    }
    return sb;
}

ScrollBar *ScrollZoomer::horizontalScrollBar() const
{
    return d_hScrollData->scrollBar;
}

ScrollBar *ScrollZoomer::verticalScrollBar() const
{
    return d_vScrollData->scrollBar;
}

void ScrollZoomer::setHScrollBarMode( Qt::ScrollBarPolicy mode )
{
    if ( hScrollBarMode() != mode )
    {
        d_hScrollData->mode = mode;
        updateScrollBars();
    }
}

void ScrollZoomer::setVScrollBarMode( Qt::ScrollBarPolicy mode )
{
    if ( vScrollBarMode() != mode )
    {
        d_vScrollData->mode = mode;
        updateScrollBars();
    }
}

Qt::ScrollBarPolicy ScrollZoomer::hScrollBarMode() const
{
    return d_hScrollData->mode;
}

Qt::ScrollBarPolicy ScrollZoomer::vScrollBarMode() const
{
    return d_vScrollData->mode;
}

void ScrollZoomer::setHScrollBarPosition( ScrollBarPosition pos )
{
    if ( d_hScrollData->position != pos )
    {
        d_hScrollData->position = pos;
        updateScrollBars();
    }
}

void ScrollZoomer::setVScrollBarPosition( ScrollBarPosition pos )
{
    if ( d_vScrollData->position != pos )
    {
        d_vScrollData->position = pos;
        updateScrollBars();
    }
}

ScrollZoomer::ScrollBarPosition ScrollZoomer::hScrollBarPosition() const
{
    return d_hScrollData->position;
}

ScrollZoomer::ScrollBarPosition ScrollZoomer::vScrollBarPosition() const
{
    return d_vScrollData->position;
}

void ScrollZoomer::setCornerWidget( QWidget *w )
{
    if ( w != d_cornerWidget )
    {
        if ( canvas() )
        {
            delete d_cornerWidget;
            d_cornerWidget = w;
            if ( d_cornerWidget->parent() != canvas() )
                d_cornerWidget->setParent( canvas() );

            updateScrollBars();
        }
    }
}

QWidget *ScrollZoomer::cornerWidget() const
{
    return d_cornerWidget;
}

bool ScrollZoomer::eventFilter( QObject *object, QEvent *event )
{
    if ( object == canvas() )
    {
        switch( event->type() )
        {
            case QEvent::Resize:
            {
                int left, top, right, bottom;
                canvas()->getContentsMargins( &left, &top, &right, &bottom );

                QRect rect;
                rect.setSize( static_cast<QResizeEvent *>( event )->size() );
                rect.adjust( left, top, -right, -bottom );

                layoutScrollBars( rect );
                break;
            }
            case QEvent::ChildRemoved:
            {
                const QObject *child =
                    static_cast<QChildEvent *>( event )->child();

                if ( child == d_cornerWidget )
                    d_cornerWidget = NULL;
                else if ( child == d_hScrollData->scrollBar )
                    d_hScrollData->scrollBar = NULL;
                else if ( child == d_vScrollData->scrollBar )
                    d_vScrollData->scrollBar = NULL;
                break;
            }
            default:
                break;
        }
    }
    return QwtPlotZoomer::eventFilter( object, event );
}

bool ScrollZoomer::needScrollBar( Qt::Orientation orientation ) const
{
    Qt::ScrollBarPolicy mode;
    double zoomMin, zoomMax, baseMin, baseMax;

    if ( orientation == Qt::Horizontal )
    {
        mode = d_hScrollData->mode;
        baseMin = zoomBase().left();
        baseMax = zoomBase().right();
        zoomMin = zoomRect().left();
        zoomMax = zoomRect().right();
    }
    else
    {
        mode = d_vScrollData->mode;
        baseMin = zoomBase().top();
        baseMax = zoomBase().bottom();
        zoomMin = zoomRect().top();
        zoomMax = zoomRect().bottom();
    }

    bool needed = false;
    switch( mode )
    {
        case Qt::ScrollBarAlwaysOn:
            needed = true;
            break;
        case Qt::ScrollBarAlwaysOff:
            needed = false;
            break;
        default:
        {
            if ( baseMin < zoomMin || baseMax > zoomMax )
                needed = true;
            break;
        }
    }
    return needed;
}

void ScrollZoomer::updateScrollBars()
{
    if ( !canvas() )
        return;

    const int xAxis = QwtPlotZoomer::xAxis();
    const int yAxis = QwtPlotZoomer::yAxis();

    int xScrollBarAxis = xAxis;
    if ( hScrollBarPosition() == OppositeToScale )
        xScrollBarAxis = oppositeAxis( xScrollBarAxis );

    int yScrollBarAxis = yAxis;
    if ( vScrollBarPosition() == OppositeToScale )
        yScrollBarAxis = oppositeAxis( yScrollBarAxis );


    QwtPlotLayout *layout = plot()->plotLayout();

    bool showHScrollBar = needScrollBar( Qt::Horizontal );
    if ( showHScrollBar )
    {
        ScrollBar *sb = scrollBar( Qt::Horizontal );
        sb->setPalette( plot()->palette() );
        sb->setInverted( !plot()->axisScaleDiv( xAxis ).isIncreasing() );
        sb->setBase( zoomBase().left(), zoomBase().right() );
        sb->moveSlider( zoomRect().left(), zoomRect().right() );

        if ( !sb->isVisibleTo( canvas() ) )
        {
            sb->show();
            layout->setCanvasMargin( layout->canvasMargin( xScrollBarAxis )
                + sb->extent(), xScrollBarAxis );
        }
    }
    else
    {
        if ( horizontalScrollBar() )
        {
            horizontalScrollBar()->hide();
            layout->setCanvasMargin( layout->canvasMargin( xScrollBarAxis )
                - horizontalScrollBar()->extent(), xScrollBarAxis );
        }
    }

    bool showVScrollBar = needScrollBar( Qt::Vertical );
    if ( showVScrollBar )
    {
        ScrollBar *sb = scrollBar( Qt::Vertical );
        sb->setPalette( plot()->palette() );
        sb->setInverted( !plot()->axisScaleDiv( yAxis ).isIncreasing() );
        sb->setBase( zoomBase().top(), zoomBase().bottom() );
        sb->moveSlider( zoomRect().top(), zoomRect().bottom() );

        if ( !sb->isVisibleTo( canvas() ) )
        {
            sb->show();
            layout->setCanvasMargin( layout->canvasMargin( yScrollBarAxis )
                + sb->extent(), yScrollBarAxis );
        }
    }
    else
    {
        if ( verticalScrollBar() )
        {
            verticalScrollBar()->hide();
            layout->setCanvasMargin( layout->canvasMargin( yScrollBarAxis )
                - verticalScrollBar()->extent(), yScrollBarAxis );
        }
    }

    if ( showHScrollBar && showVScrollBar )
    {
        if ( d_cornerWidget == NULL )
        {
            d_cornerWidget = new QWidget( canvas() );
            d_cornerWidget->setAutoFillBackground( true );
            d_cornerWidget->setPalette( plot()->palette() );
        }
        d_cornerWidget->show();
    }
    else
    {
        if ( d_cornerWidget )
            d_cornerWidget->hide();
    }

    layoutScrollBars( canvas()->contentsRect() );
    plot()->updateLayout();
}

void ScrollZoomer::layoutScrollBars( const QRect &rect )
{
    int hPos = xAxis();
    if ( hScrollBarPosition() == OppositeToScale )
        hPos = oppositeAxis( hPos );

    int vPos = yAxis();
    if ( vScrollBarPosition() == OppositeToScale )
        vPos = oppositeAxis( vPos );

    ScrollBar *hScrollBar = horizontalScrollBar();
    ScrollBar *vScrollBar = verticalScrollBar();

    const int hdim = hScrollBar ? hScrollBar->extent() : 0;
    const int vdim = vScrollBar ? vScrollBar->extent() : 0;

    if ( hScrollBar && hScrollBar->isVisible() )
    {
        int x = rect.x();
        int y = ( hPos == QwtPlot::xTop )
            ? rect.top() : rect.bottom() - hdim + 1;
        int w = rect.width();

        if ( vScrollBar && vScrollBar->isVisible() )
        {
            if ( vPos == QwtPlot::yLeft )
                x += vdim;
            w -= vdim;
        }

        hScrollBar->setGeometry( x, y, w, hdim );
    }
    if ( vScrollBar && vScrollBar->isVisible() )
    {
        int pos = yAxis();
        if ( vScrollBarPosition() == OppositeToScale )
            pos = oppositeAxis( pos );

        int x = ( vPos == QwtPlot::yLeft )
            ? rect.left() : rect.right() - vdim + 1;
        int y = rect.y();

        int h = rect.height();

        if ( hScrollBar && hScrollBar->isVisible() )
        {
            if ( hPos == QwtPlot::xTop )
                y += hdim;

            h -= hdim;
        }

        vScrollBar->setGeometry( x, y, vdim, h );
    }
    if ( hScrollBar && hScrollBar->isVisible() &&
        vScrollBar && vScrollBar->isVisible() )
    {
        if ( d_cornerWidget )
        {
            QRect cornerRect(
                vScrollBar->pos().x(), hScrollBar->pos().y(),
                vdim, hdim );
            d_cornerWidget->setGeometry( cornerRect );
        }
    }
}

void ScrollZoomer::scrollBarMoved(
    Qt::Orientation o, double min, double max )
{
    Q_UNUSED( max );

    if ( o == Qt::Horizontal )
        moveTo( QPointF( min, zoomRect().top() ) );
    else
        moveTo( QPointF( zoomRect().left(), min ) );

    Q_EMIT zoomed( zoomRect() );
}

int ScrollZoomer::oppositeAxis( int axis ) const
{
    switch( axis )
    {
        case QwtPlot::xBottom:
            return QwtPlot::xTop;
        case QwtPlot::xTop:
            return QwtPlot::xBottom;
        case QwtPlot::yLeft:
            return QwtPlot::yRight;
        case QwtPlot::yRight:
            return QwtPlot::yLeft;
        default:
            break;
    }

    return axis;
}
```

### `examples/realtime/scrollzoomer.h`

```cpp
#ifndef _SCROLLZOOMER_H
#define _SCROLLZOOMER_H

#include <qglobal.h>
#include <qwt_plot_zoomer.h>
#include <qwt_plot.h>

class ScrollData;
class ScrollBar;

class ScrollZoomer: public QwtPlotZoomer
{
    Q_OBJECT
public:
    enum ScrollBarPosition
    {
        AttachedToScale,
        OppositeToScale
    };

    ScrollZoomer( QWidget * );
    virtual ~ScrollZoomer();

    ScrollBar *horizontalScrollBar() const;
    ScrollBar *verticalScrollBar() const;

    void setHScrollBarMode( Qt::ScrollBarPolicy );
    void setVScrollBarMode( Qt::ScrollBarPolicy );

    Qt::ScrollBarPolicy vScrollBarMode () const;
    Qt::ScrollBarPolicy hScrollBarMode () const;

    void setHScrollBarPosition( ScrollBarPosition );
    void setVScrollBarPosition( ScrollBarPosition );

    ScrollBarPosition hScrollBarPosition() const;
    ScrollBarPosition vScrollBarPosition() const;

    QWidget* cornerWidget() const;
    virtual void setCornerWidget( QWidget * );

    virtual bool eventFilter( QObject *, QEvent * );

    virtual void rescale();

protected:
    virtual ScrollBar *scrollBar( Qt::Orientation );
    virtual void updateScrollBars();
    virtual void layoutScrollBars( const QRect & );

private Q_SLOTS:
    void scrollBarMoved( Qt::Orientation o, double min, double max );

private:
    bool needScrollBar( Qt::Orientation ) const;
    int oppositeAxis( int ) const;

    QWidget *d_cornerWidget;

    ScrollData *d_hScrollData;
    ScrollData *d_vScrollData;

    bool d_inZoom;
    bool d_alignCanvasToScales[ QwtPlot::axisCnt ];
};

#endif
```

### `examples/refreshtest/circularbuffer.cpp`

```cpp
#include "circularbuffer.h"
#include <math.h>

CircularBuffer::CircularBuffer( double interval, size_t numPoints ):
    d_y( NULL ),
    d_referenceTime( 0.0 ),
    d_startIndex( 0 ),
    d_offset( 0.0 )
{
    fill( interval, numPoints );
}

void CircularBuffer::fill( double interval, size_t numPoints )
{
    if ( interval <= 0.0 || numPoints < 2 )
        return;

    d_values.resize( numPoints );
    d_values.fill( 0.0 );

    if ( d_y )
    {
        d_step = interval / ( numPoints - 2 );
        for ( size_t i = 0; i < numPoints; i++ )
            d_values[i] = d_y( i * d_step );
    }

    d_interval = interval;
}

void CircularBuffer::setFunction( double( *y )( double ) )
{
    d_y = y;
}

void CircularBuffer::setReferenceTime( double timeStamp )
{
    d_referenceTime = timeStamp;

    const double startTime = ::fmod( d_referenceTime, d_values.size() * d_step );

    d_startIndex = int( startTime / d_step ); // floor
    d_offset = ::fmod( startTime, d_step );
}

double CircularBuffer::referenceTime() const
{
    return d_referenceTime;
}

size_t CircularBuffer::size() const
{
    return d_values.size();
}

QPointF CircularBuffer::sample( size_t i ) const
{
    const int size = d_values.size();

    int index = d_startIndex + i;
    if ( index >= size )
        index -= size;

    const double x = i * d_step - d_offset - d_interval;
    const double y = d_values.data()[index];

    return QPointF( x, y );
}

QRectF CircularBuffer::boundingRect() const
{
    return QRectF( -1.0, -d_interval, 2.0, d_interval );
}
```

### `examples/refreshtest/circularbuffer.h`

```cpp
#ifndef _CIRCULAR_BUFFER_H_
#define _CIRCULAR_BUFFER_H_

#include <qwt_series_data.h>
#include <qvector.h>

class CircularBuffer: public QwtSeriesData<QPointF>
{
public:
    CircularBuffer( double interval = 10.0, size_t numPoints = 1000 );
    void fill( double interval, size_t numPoints );

    void setReferenceTime( double );
    double referenceTime() const;

    virtual size_t size() const;
    virtual QPointF sample( size_t i ) const;

    virtual QRectF boundingRect() const;

    void setFunction( double( *y )( double ) );

private:
    double ( *d_y )( double );

    double d_referenceTime;
    double d_interval;
    QVector<double> d_values;

    double d_step;
    int d_startIndex;
    double d_offset;
};

#endif
```

### `examples/refreshtest/main.cpp`

```cpp
#include "mainwindow.h"
#include <qapplication.h>

#ifndef QWT_NO_OPENGL
#if QT_VERSION >= 0x040600 && QT_VERSION < 0x050000
#define USE_OPENGL 1
#endif
#endif

#if USE_OPENGL
#include <qgl.h>
#endif

int main( int argc, char **argv )
{
#if USE_OPENGL
    // on my box QPaintEngine::OpenGL2 has serious problems, f.e:
    // the lines of a simple drawRect are wrong.

    QGL::setPreferredPaintEngine( QPaintEngine::OpenGL );
#endif

    QApplication a( argc, argv );

    MainWindow mainWindow;
    mainWindow.resize( 600, 400 );
    mainWindow.show();

    return a.exec();
}
```

### `examples/refreshtest/mainwindow.cpp`

```cpp
#include <qstatusbar.h>
#include <qlabel.h>
#include <qlayout.h>
#include <qevent.h>
#include <qdatetime.h>
#include <qwt_plot_canvas.h>
#include "panel.h"
#include "plot.h"
#include "mainwindow.h"

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    QWidget *w = new QWidget( this );

    d_panel = new Panel( w );

    d_plot = new Plot( w );

    QHBoxLayout *hLayout = new QHBoxLayout( w );
    hLayout->addWidget( d_panel );
    hLayout->addWidget( d_plot, 10 );

    setCentralWidget( w );

    d_frameCount = new QLabel( this );
    statusBar()->addWidget( d_frameCount, 10 );

    applySettings( d_panel->settings() );

    connect( d_panel, SIGNAL( settingsChanged( const Settings & ) ),
        this, SLOT( applySettings( const Settings & ) ) );
}

bool MainWindow::eventFilter( QObject *object, QEvent *event )
{
    if ( object == d_plot->canvas() && event->type() == QEvent::Paint )
    {
        static int counter;
        static QTime timeStamp;

        if ( !timeStamp.isValid() )
        {
            timeStamp.start();
            counter = 0;
        }
        else
        {
            counter++;

            const double elapsed = timeStamp.elapsed() / 1000.0;
            if ( elapsed >= 1 )
            {
                QString fps;
                fps.setNum( qRound( counter / elapsed ) );
                fps += " Fps";

                d_frameCount->setText( fps );

                counter = 0;
                timeStamp.start();
            }
        }
    }

    return QMainWindow::eventFilter( object, event );
}

void MainWindow::applySettings( const Settings &settings )
{
    d_plot->setSettings( settings );

    // the canvas might have been recreated
    d_plot->canvas()->removeEventFilter( this );
    d_plot->canvas()->installEventFilter( this );
}
```

### `examples/refreshtest/mainwindow.h`

```cpp
#ifndef _MAIN_WINDOW_H_
#define _MAIN_WINDOW_H_

#include <qmainwindow.h>

class Plot;
class Panel;
class QLabel;
class Settings;

class MainWindow: public QMainWindow
{
    Q_OBJECT

public:
    MainWindow( QWidget *parent = NULL );
    virtual bool eventFilter( QObject *, QEvent * );

private Q_SLOTS:
    void applySettings( const Settings & );

private:
    Plot *d_plot;
    Panel *d_panel;
    QLabel *d_frameCount;
};

#endif
```

### `examples/refreshtest/panel.cpp`

```cpp
#include "panel.h"
#include <qlabel.h>
#include <qcombobox.h>
#include <qspinbox.h>
#include <qcheckbox.h>
#include <qlayout.h>
#include <qwt_plot_curve.h>

class SpinBox: public QSpinBox
{
public:
    SpinBox( int min, int max, int step, QWidget *parent ):
        QSpinBox( parent )
    {
        setRange( min, max );
        setSingleStep( step );
    }
};

class CheckBox: public QCheckBox
{
public:
    CheckBox( const QString &title, QWidget *parent ):
        QCheckBox( title, parent )
    {
    }

    void setChecked( bool checked )
    {
        setCheckState( checked ? Qt::Checked : Qt::Unchecked );
    }

    bool isChecked() const
    {
        return checkState() == Qt::Checked;
    }
};

Panel::Panel( QWidget *parent ):
    QTabWidget( parent )
{
    setTabPosition( QTabWidget::West );

    addTab( createPlotTab( this ), "Plot" );
    addTab( createCanvasTab( this ), "Canvas" );
    addTab( createCurveTab( this ), "Curve" );

    setSettings( Settings() );

    connect( d_numPoints, SIGNAL( valueChanged( int ) ), SLOT( edited() ) );
    connect( d_updateInterval, SIGNAL( valueChanged( int ) ), SLOT( edited() ) );
    connect( d_curveWidth, SIGNAL( valueChanged( int ) ), SLOT( edited() ) );

    connect( d_paintCache, SIGNAL( stateChanged( int ) ), SLOT( edited() ) );
    connect( d_paintOnScreen, SIGNAL( stateChanged( int ) ), SLOT( edited() ) );
    connect( d_immediatePaint, SIGNAL( stateChanged( int ) ), SLOT( edited() ) );
#ifndef QWT_NO_OPENGL
    connect( d_openGL, SIGNAL( stateChanged( int ) ), SLOT( edited() ) );
#endif

    connect( d_curveAntialiasing, SIGNAL( stateChanged( int ) ), SLOT( edited() ) );
    connect( d_curveClipping, SIGNAL( stateChanged( int ) ), SLOT( edited() ) );
    connect( d_curveFiltering, SIGNAL( stateChanged( int ) ), SLOT( edited() ) );
    connect( d_lineSplitting, SIGNAL( stateChanged( int ) ), SLOT( edited() ) );
    connect( d_curveFilled, SIGNAL( stateChanged( int ) ), SLOT( edited() ) );

    connect( d_updateType, SIGNAL( currentIndexChanged( int ) ), SLOT( edited() ) );
    connect( d_gridStyle, SIGNAL( currentIndexChanged( int ) ), SLOT( edited() ) );
    connect( d_curveType, SIGNAL( currentIndexChanged( int ) ), SLOT( edited() ) );
    connect( d_curvePen, SIGNAL( currentIndexChanged( int ) ), SLOT( edited() ) );
}

QWidget *Panel::createPlotTab( QWidget *parent )
{
    QWidget *page = new QWidget( parent );

    d_updateInterval = new SpinBox( 0, 1000, 10, page );
    d_numPoints = new SpinBox( 10, 1000000, 1000, page );

    d_updateType = new QComboBox( page );
    d_updateType->addItem( "Repaint" );
    d_updateType->addItem( "Replot" );

    int row = 0;

    QGridLayout *layout = new QGridLayout( page );

    layout->addWidget( new QLabel( "Updates", page ), row, 0 );
    layout->addWidget( d_updateInterval, row, 1 );
    layout->addWidget( new QLabel( "ms", page ), row++, 2 );

    layout->addWidget( new QLabel( "Points", page ), row, 0 );
    layout->addWidget( d_numPoints, row++, 1 );

    layout->addWidget( new QLabel( "Update", page ), row, 0 );
    layout->addWidget( d_updateType, row++, 1 );

    layout->addLayout( new QHBoxLayout(), row++, 0 );

    layout->setColumnStretch( 1, 10 );
    layout->setRowStretch( row, 10 );

    return page;
}

QWidget *Panel::createCanvasTab( QWidget *parent )
{
    QWidget *page = new QWidget( parent );

    d_gridStyle = new QComboBox( page );
    d_gridStyle->addItem( "None" );
    d_gridStyle->addItem( "Solid" );
    d_gridStyle->addItem( "Dashes" );

    d_paintCache = new CheckBox( "Paint Cache", page );
    d_paintOnScreen = new CheckBox( "Paint On Screen", page );
    d_immediatePaint = new CheckBox( "Immediate Paint", page );
#ifndef QWT_NO_OPENGL
    d_openGL = new CheckBox( "OpenGL", page );
#endif

    int row = 0;

    QGridLayout *layout = new QGridLayout( page );
    layout->addWidget( new QLabel( "Grid", page ), row, 0 );
    layout->addWidget( d_gridStyle, row++, 1 );

    layout->addWidget( d_paintCache, row++, 0, 1, -1 );
    layout->addWidget( d_paintOnScreen, row++, 0, 1, -1 );
    layout->addWidget( d_immediatePaint, row++, 0, 1, -1 );
#ifndef QWT_NO_OPENGL
    layout->addWidget( d_openGL, row++, 0, 1, -1 );
#endif

    layout->addLayout( new QHBoxLayout(), row++, 0 );

    layout->setColumnStretch( 1, 10 );
    layout->setRowStretch( row, 10 );

    return page;
}

QWidget *Panel::createCurveTab( QWidget *parent )
{
    QWidget *page = new QWidget( parent );

    d_curveType = new QComboBox( page );
    d_curveType->addItem( "Wave" );
    d_curveType->addItem( "Noise" );

    d_curveAntialiasing = new CheckBox( "Antialiasing", page );
    d_curveClipping = new CheckBox( "Clipping", page );
    d_curveFiltering = new CheckBox( "Filtering", page );
    d_lineSplitting = new CheckBox( "Split Lines", page );

    d_curveWidth = new SpinBox( 0, 10, 1, page );

    d_curvePen = new QComboBox( page );
    d_curvePen->addItem( "Solid" );
    d_curvePen->addItem( "Dotted" );

    d_curveFilled = new CheckBox( "Filled", page );

    int row = 0;

    QGridLayout *layout = new QGridLayout( page );
    layout->addWidget( new QLabel( "Type", page ), row, 0 );
    layout->addWidget( d_curveType, row++, 1 );

    layout->addWidget( d_curveAntialiasing, row++, 0, 1, -1 );
    layout->addWidget( d_curveClipping, row++, 0, 1, -1 );
    layout->addWidget( d_curveFiltering, row++, 0, 1, -1 );
    layout->addWidget( d_lineSplitting, row++, 0, 1, -1 );

    layout->addWidget( new QLabel( "Width", page ), row, 0 );
    layout->addWidget( d_curveWidth, row++, 1 );

    layout->addWidget( new QLabel( "Style", page ), row, 0 );
    layout->addWidget( d_curvePen, row++, 1 );

    layout->addWidget( d_curveFilled, row++, 0, 1, -1 );

    layout->addLayout( new QHBoxLayout(), row++, 0 );

    layout->setColumnStretch( 1, 10 );
    layout->setRowStretch( row, 10 );

    return page;
}

void Panel::edited()
{
    const Settings s = settings();
    Q_EMIT settingsChanged( s );
}


Settings Panel::settings() const
{
    Settings s;

    s.grid.pen = QPen( Qt::black, 0 );

    switch( d_gridStyle->currentIndex() )
    {
        case 0:
            s.grid.pen.setStyle( Qt::NoPen );
            break;
        case 2:
            s.grid.pen.setStyle( Qt::DashLine );
            break;
    }

    s.curve.pen.setStyle( d_curvePen->currentIndex() == 0 ?
        Qt::SolidLine : Qt::DotLine );
    s.curve.pen.setWidth( d_curveWidth->value() );
    s.curve.brush.setStyle( ( d_curveFilled->isChecked() ) ?
        Qt::SolidPattern : Qt::NoBrush );
    s.curve.numPoints = d_numPoints->value();
    s.curve.functionType = static_cast<Settings::FunctionType>(
        d_curveType->currentIndex() );
    if ( d_curveClipping->isChecked() )
        s.curve.paintAttributes |= QwtPlotCurve::ClipPolygons;
    else
        s.curve.paintAttributes &= ~QwtPlotCurve::ClipPolygons;
    if ( d_curveFiltering->isChecked() )
        s.curve.paintAttributes |= QwtPlotCurve::FilterPoints;
    else
        s.curve.paintAttributes &= ~QwtPlotCurve::FilterPoints;

    if ( d_curveAntialiasing->isChecked() )
        s.curve.renderHint |= QwtPlotItem::RenderAntialiased;
    else
        s.curve.renderHint &= ~QwtPlotItem::RenderAntialiased;

    s.curve.lineSplitting = ( d_lineSplitting->isChecked() );

    s.canvas.useBackingStore = ( d_paintCache->isChecked() );
    s.canvas.paintOnScreen = ( d_paintOnScreen->isChecked() );
    s.canvas.immediatePaint = ( d_immediatePaint->isChecked() );
#ifndef QWT_NO_OPENGL
    s.canvas.openGL = ( d_openGL->isChecked() );
#endif

    s.updateInterval = d_updateInterval->value();
    s.updateType = static_cast<Settings::UpdateType>( d_updateType->currentIndex() );

    return s;
}

void Panel::setSettings( const Settings &s )
{
    d_numPoints->setValue( s.curve.numPoints );
    d_updateInterval->setValue( s.updateInterval );
    d_updateType->setCurrentIndex( s.updateType );

    switch( s.grid.pen.style() )
    {
        case Qt::NoPen:
        {
            d_gridStyle->setCurrentIndex( 0 );
            break;
        }
        case Qt::DashLine:
        {
            d_gridStyle->setCurrentIndex( 2 );
            break;
        }
        default:
        {
            d_gridStyle->setCurrentIndex( 1 ); // Solid
        }
    }

    d_paintCache->setChecked( s.canvas.useBackingStore );
    d_paintOnScreen->setChecked( s.canvas.paintOnScreen );
    d_immediatePaint->setChecked( s.canvas.immediatePaint );
#ifndef QWT_NO_OPENGL
    d_openGL->setChecked( s.canvas.openGL );
#endif

    d_curveType->setCurrentIndex( s.curve.functionType );
    d_curveAntialiasing->setChecked(
        s.curve.renderHint & QwtPlotCurve::RenderAntialiased );

    d_curveClipping->setChecked(
        s.curve.paintAttributes & QwtPlotCurve::ClipPolygons );
    d_curveFiltering->setChecked(
        s.curve.paintAttributes & QwtPlotCurve::FilterPoints );

    d_lineSplitting->setChecked( s.curve.lineSplitting );

    d_curveWidth->setValue( s.curve.pen.width() );
    d_curvePen->setCurrentIndex(
        s.curve.pen.style() == Qt::SolidLine ? 0 : 1 );
    d_curveFilled->setChecked( s.curve.brush.style() != Qt::NoBrush );
}
```

### `examples/refreshtest/panel.h`

```cpp
#ifndef _PANEL_H_
#define _PANEL_H_ 1

#include "settings.h"
#include <qtabwidget.h>

class QComboBox;
class SpinBox;
class CheckBox;

class Panel: public QTabWidget
{
    Q_OBJECT

public:
    Panel( QWidget * = NULL );

    Settings settings() const;
    void setSettings( const Settings & );

Q_SIGNALS:
    void settingsChanged( const Settings & );

private Q_SLOTS:
    void edited();

private:
    QWidget *createPlotTab( QWidget * );
    QWidget *createCanvasTab( QWidget * );
    QWidget *createCurveTab( QWidget * );

    SpinBox *d_numPoints;
    SpinBox *d_updateInterval;
    QComboBox *d_updateType;

    QComboBox *d_gridStyle;
    CheckBox *d_paintCache;
    CheckBox *d_paintOnScreen;
    CheckBox *d_immediatePaint;
#ifndef QWT_NO_OPENGL
    CheckBox *d_openGL;
#endif

    QComboBox *d_curveType;
    CheckBox *d_curveAntialiasing;
    CheckBox *d_curveClipping;
    CheckBox *d_curveFiltering;
    CheckBox *d_lineSplitting;
    SpinBox  *d_curveWidth;
    QComboBox *d_curvePen;
    CheckBox *d_curveFilled;
};

#endif
```

### `examples/refreshtest/plot.cpp`

```cpp
#include <qglobal.h>
#include <qwt_painter.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_grid.h>
#include <qwt_plot_curve.h>
#include <qwt_plot_layout.h>
#include <qwt_scale_widget.h>
#include <qwt_scale_draw.h>
#ifndef QWT_NO_OPENGL
#include <qevent.h>
#include <qwt_plot_glcanvas.h>
#endif
#include "plot.h"
#include "circularbuffer.h"
#include "settings.h"

static double wave( double x )
{
    const double period = 1.0;
    const double c = 5.0;

    double v = ::fmod( x, period );

    const double amplitude = qAbs( x - qRound( x / c ) * c ) / ( 0.5 * c );
    v = amplitude * qSin( v / period * 2 * M_PI );

    return v;
}

static double noise( double )
{
    return 2.0 * ( qrand() / ( static_cast<double>( RAND_MAX ) + 1 ) ) - 1.0;
}

#ifndef QWT_NO_OPENGL
class GLCanvas: public QwtPlotGLCanvas
{
public:
    GLCanvas( QwtPlot *parent = NULL ):
        QwtPlotGLCanvas( parent )
    {
        setContentsMargins( 1, 1, 1, 1 );
    }

protected:
    virtual void paintEvent( QPaintEvent *event )
    {
        QPainter painter( this );
        painter.setClipRegion( event->region() );

        QwtPlot *plot = qobject_cast< QwtPlot *>( parent() );
        if ( plot )
            plot->drawCanvas( &painter );

        painter.setPen( palette().foreground().color() );
#if QT_VERSION >= 0x050000
        painter.drawRect( rect().adjusted( 1, 1, 0, 0 ) );
#else
        painter.drawRect( rect().adjusted( 0, 0, -1, -1 ) );
#endif
    }
};
#endif

Plot::Plot( QWidget *parent ):
    QwtPlot( parent ),
    d_interval( 10.0 ), // seconds
    d_timerId( -1 )
{
    // Assign a title
    setTitle( "Testing Refresh Rates" );

    QwtPlotCanvas *canvas = new QwtPlotCanvas();
    canvas->setFrameStyle( QFrame::Box | QFrame::Plain );
    canvas->setLineWidth( 1 );
    canvas->setPalette( Qt::white );

    setCanvas( canvas );

    alignScales();

    // Insert grid
    d_grid = new QwtPlotGrid();
    d_grid->attach( this );

    // Insert curve
    d_curve = new QwtPlotCurve( "Data Moving Right" );
    d_curve->setPen( Qt::black );
    d_curve->setData( new CircularBuffer( d_interval, 10 ) );
    d_curve->attach( this );

    // Axis
    setAxisTitle( QwtPlot::xBottom, "Seconds" );
    setAxisScale( QwtPlot::xBottom, -d_interval, 0.0 );

    setAxisTitle( QwtPlot::yLeft, "Values" );
    setAxisScale( QwtPlot::yLeft, -1.0, 1.0 );

    d_clock.start();

    setSettings( d_settings );
}

//
//  Set a plain canvas frame and align the scales to it
//
void Plot::alignScales()
{
    // The code below shows how to align the scales to
    // the canvas frame, but is also a good example demonstrating
    // why the spreaded API needs polishing.

    for ( int i = 0; i < QwtPlot::axisCnt; i++ )
    {
        QwtScaleWidget *scaleWidget = axisWidget( i );
        if ( scaleWidget )
            scaleWidget->setMargin( 0 );

        QwtScaleDraw *scaleDraw = axisScaleDraw( i );
        if ( scaleDraw )
            scaleDraw->enableComponent( QwtAbstractScaleDraw::Backbone, false );
    }

    plotLayout()->setAlignCanvasToScales( true );
}

void Plot::setSettings( const Settings &s )
{
    if ( d_timerId >= 0 )
        killTimer( d_timerId );

    d_timerId = startTimer( s.updateInterval );

    d_grid->setPen( s.grid.pen );
    d_grid->setVisible( s.grid.pen.style() != Qt::NoPen );

    CircularBuffer *buffer = static_cast<CircularBuffer *>( d_curve->data() );
    if ( s.curve.numPoints != buffer->size() ||
            s.curve.functionType != d_settings.curve.functionType )
    {
        switch( s.curve.functionType )
        {
            case Settings::Wave:
                buffer->setFunction( wave );
                break;
            case Settings::Noise:
                buffer->setFunction( noise );
                break;
            default:
                buffer->setFunction( NULL );
        }

        buffer->fill( d_interval, s.curve.numPoints );
    }

    d_curve->setPen( s.curve.pen );
    d_curve->setBrush( s.curve.brush );

    d_curve->setPaintAttribute( QwtPlotCurve::ClipPolygons,
        s.curve.paintAttributes & QwtPlotCurve::ClipPolygons );
    d_curve->setPaintAttribute( QwtPlotCurve::FilterPoints,
        s.curve.paintAttributes & QwtPlotCurve::FilterPoints );

    d_curve->setRenderHint( QwtPlotItem::RenderAntialiased,
        s.curve.renderHint & QwtPlotItem::RenderAntialiased );

#ifndef QWT_NO_OPENGL
    if ( s.canvas.openGL )
    {
        QwtPlotGLCanvas *plotCanvas = qobject_cast<QwtPlotGLCanvas *>( canvas() );
        if ( plotCanvas == NULL )
        {
            plotCanvas = new GLCanvas();
            plotCanvas->setPalette( QColor( "khaki" ) );

            setCanvas( plotCanvas );
        }
    }
    else
#endif
    {
        QwtPlotCanvas *plotCanvas = qobject_cast<QwtPlotCanvas *>( canvas() );
        if ( plotCanvas == NULL )
        {
            plotCanvas = new QwtPlotCanvas();
            plotCanvas->setFrameStyle( QFrame::Box | QFrame::Plain );
            plotCanvas->setLineWidth( 1 );
            plotCanvas->setPalette( Qt::white );

            setCanvas( plotCanvas );
        }

        plotCanvas->setAttribute( Qt::WA_PaintOnScreen, s.canvas.paintOnScreen );

        plotCanvas->setPaintAttribute(
            QwtPlotCanvas::BackingStore, s.canvas.useBackingStore );
        plotCanvas->setPaintAttribute(
            QwtPlotCanvas::ImmediatePaint, s.canvas.immediatePaint );
    }

    QwtPainter::setPolylineSplitting( s.curve.lineSplitting );

    d_settings = s;
}

void Plot::timerEvent( QTimerEvent * )
{
    CircularBuffer *buffer = static_cast<CircularBuffer *>( d_curve->data() );
    buffer->setReferenceTime( d_clock.elapsed() / 1000.0 );

    if ( d_settings.updateType == Settings::RepaintCanvas )
    {
        // the axes in this example doesn't change. So all we need to do
        // is to repaint the canvas.

        QMetaObject::invokeMethod( canvas(), "replot", Qt::DirectConnection );
    }
    else
    {
        replot();
    }
}
```

### `examples/refreshtest/plot.h`

```cpp
#ifndef _PLOT_H_
#define _PLOT_H_ 1

#include <qwt_plot.h>
#include <qwt_system_clock.h>
#include "settings.h"

class QwtPlotGrid;
class QwtPlotCurve;

class Plot: public QwtPlot
{
    Q_OBJECT

public:
    Plot( QWidget* = NULL );

public Q_SLOTS:
    void setSettings( const Settings & );

protected:
    virtual void timerEvent( QTimerEvent *e );

private:
    void alignScales();

    QwtPlotGrid *d_grid;
    QwtPlotCurve *d_curve;

    QwtSystemClock d_clock;
    double d_interval;

    int d_timerId;

    Settings d_settings;
};

#endif
```

### `examples/refreshtest/settings.h`

```cpp
#ifndef _SETTINGS_H_
#define _SETTINGS_H_

#include <qpen.h>
#include <qbrush.h>

class Settings
{
public:
    enum FunctionType
    {
        NoFunction = -1,

        Wave,
        Noise
    };

    enum UpdateType
    {
        RepaintCanvas,
        Replot
    };

    Settings()
    {
        grid.pen = Qt::NoPen;
        grid.pen.setCosmetic( true );

        curve.brush = Qt::NoBrush;
        curve.numPoints = 1000;
        curve.functionType = Wave;
        curve.paintAttributes = 0;
        curve.renderHint = 0;
        curve.lineSplitting = true;

        canvas.useBackingStore = false;
        canvas.paintOnScreen = false;
        canvas.immediatePaint = true;
#ifndef QWT_NO_OPENGL
        canvas.openGL = false;
#endif

        updateType = RepaintCanvas;
        updateInterval = 20;
    }

    struct gridSettings
    {
        QPen pen;
    } grid;

    struct curveSettings
    {
        QPen pen;
        QBrush brush;
        uint numPoints;
        FunctionType functionType;
        int paintAttributes;
        int renderHint;
        bool lineSplitting;
    } curve;

    struct canvasSettings
    {
        bool useBackingStore;
        bool paintOnScreen;
        bool immediatePaint;

#ifndef QWT_NO_OPENGL
        bool openGL;
#endif
    } canvas;

    UpdateType updateType;
    int updateInterval;
};

#endif
```

### `examples/scatterplot/main.cpp`

```cpp
#include <qapplication.h>
#include "mainwindow.h"

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWindow w;
    w.resize( 800, 600 );
    w.show();

    return a.exec();
}
```

### `examples/scatterplot/mainwindow.cpp`

```cpp
#include "mainwindow.h"
#include "plot.h"
#include <qmath.h>

static double randomValue()
{
    // a number between [ 0.0, 1.0 ]
    return ( qrand() % 100000 ) / 100000.0;
}

MainWindow::MainWindow()
{
    d_plot = new Plot( this );
    d_plot->setTitle( "Scatter Plot" );
    setCentralWidget( d_plot );

    // a million points
    setSamples( 100000 );
}

void MainWindow::setSamples( int numPoints )
{
    QPolygonF samples;

    for ( int i = 0; i < numPoints; i++ )
    {
        const double x = randomValue() * 24.0 + 1.0;
        const double y = ::log( 10.0 * ( x - 1.0 ) + 1.0 ) 
            * ( randomValue() * 0.5 + 0.9 );

        samples += QPointF( x, y );
    }

    d_plot->setSamples( samples );
}
```

### `examples/scatterplot/mainwindow.h`

```cpp
#ifndef _MAINWINDOW_H_
#define _MAINWINDOW_H_ 1

#include <qmainwindow.h>

class Plot;

class MainWindow: public QMainWindow
{
    Q_OBJECT

public:
    MainWindow();

private:
    void setSamples( int samples );

private:
    Plot *d_plot;
};

#endif
```

### `examples/scatterplot/plot.cpp`

```cpp
#include "plot.h"
#include <qwt_plot_magnifier.h>
#include <qwt_plot_panner.h>
#include <qwt_plot_picker.h>
#include <qwt_picker_machine.h>
#include <qwt_plot_curve.h>

class DistancePicker: public QwtPlotPicker
{
public:
    DistancePicker( QWidget *canvas ):
        QwtPlotPicker( canvas )
    {
        setTrackerMode( QwtPicker::ActiveOnly );
        setStateMachine( new QwtPickerDragLineMachine() );
        setRubberBand( QwtPlotPicker::PolygonRubberBand );
    }

    virtual QwtText trackerTextF( const QPointF &pos ) const
    {
        QwtText text;

        const QPolygon points = selection();
        if ( !points.isEmpty() )
        {
            QString num;
            num.setNum( QLineF( pos, invTransform( points[0] ) ).length() );

            QColor bg( Qt::white );
            bg.setAlpha( 200 );

            text.setBackgroundBrush( QBrush( bg ) );
            text.setText( num );
        }
        return text;
    }
};

Plot::Plot( QWidget *parent ):
    QwtPlot( parent ),
    d_curve( NULL )
{
    canvas()->setStyleSheet(
        "border: 2px solid Black;"
        "border-radius: 15px;"
        "background-color: qlineargradient( x1: 0, y1: 0, x2: 0, y2: 1,"
            "stop: 0 LemonChiffon, stop: 1 PaleGoldenrod );"
    );

    // attach curve
    d_curve = new QwtPlotCurve( "Scattered Points" );
    d_curve->setPen( QColor( "Purple" ) );

    // when using QwtPlotCurve::ImageBuffer simple dots can be
    // rendered in parallel on multicore systems.
    d_curve->setRenderThreadCount( 0 ); // 0: use QThread::idealThreadCount()

    d_curve->attach( this );

    setSymbol( NULL );

    // panning with the left mouse button
    (void )new QwtPlotPanner( canvas() );

    // zoom in/out with the wheel
    QwtPlotMagnifier *magnifier = new QwtPlotMagnifier( canvas() );
    magnifier->setMouseButton( Qt::NoButton );

    // distanve measurement with the right mouse button
    DistancePicker *picker = new DistancePicker( canvas() );
    picker->setMousePattern( QwtPlotPicker::MouseSelect1, Qt::RightButton );
    picker->setRubberBandPen( QPen( Qt::blue ) );
}

void Plot::setSymbol( QwtSymbol *symbol )
{
    d_curve->setSymbol( symbol );

    if ( symbol == NULL )
    {
        d_curve->setStyle( QwtPlotCurve::Dots );
    }
}

void Plot::setSamples( const QVector<QPointF> &samples )
{
    d_curve->setPaintAttribute( 
        QwtPlotCurve::ImageBuffer, samples.size() > 1000 );

    d_curve->setSamples( samples );
}
```

### `examples/scatterplot/plot.h`

```cpp
#ifndef _PLOT_H_
#define _PLOT_H_ 1

#include <qwt_plot.h>

class QwtPlotCurve;
class QwtSymbol;

class Plot : public QwtPlot
{
    Q_OBJECT

public:
    Plot( QWidget *parent = NULL );

    void setSymbol( QwtSymbol * );
    void setSamples( const QVector<QPointF> &samples );

private:
    QwtPlotCurve *d_curve;
};

#endif // _PLOT_H_
```

### `examples/simpleplot/simpleplot.cpp`

```cpp
#include <qapplication.h>
#include <qwt_plot.h>
#include <qwt_plot_curve.h>
#include <qwt_plot_grid.h>
#include <qwt_symbol.h>
#include <qwt_legend.h>

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    QwtPlot plot;
    plot.setTitle( "Plot Demo" );
    plot.setCanvasBackground( Qt::white );
    plot.setAxisScale( QwtPlot::yLeft, 0.0, 10.0 );
    plot.insertLegend( new QwtLegend() );

    QwtPlotGrid *grid = new QwtPlotGrid();
    grid->attach( &plot );

    QwtPlotCurve *curve = new QwtPlotCurve();
    curve->setTitle( "Some Points" );
    curve->setPen( Qt::blue, 4 ),
    curve->setRenderHint( QwtPlotItem::RenderAntialiased, true );

    QwtSymbol *symbol = new QwtSymbol( QwtSymbol::Ellipse,
        QBrush( Qt::yellow ), QPen( Qt::red, 2 ), QSize( 8, 8 ) );
    curve->setSymbol( symbol );

    QPolygonF points;
    points << QPointF( 0.0, 4.4 ) << QPointF( 1.0, 3.0 )
        << QPointF( 2.0, 4.5 ) << QPointF( 3.0, 6.8 )
        << QPointF( 4.0, 7.9 ) << QPointF( 5.0, 7.1 );
    curve->setSamples( points );

    curve->attach( &plot );

    plot.resize( 600, 400 );
    plot.show();

    return a.exec();
}
```

### `examples/sinusplot/sinusplot.cpp`

```cpp
#include <qapplication.h>
#include <qlayout.h>
#include <qwt_plot.h>
#include <qwt_plot_marker.h>
#include <qwt_plot_curve.h>
#include <qwt_legend.h>
#include <qwt_point_data.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_panner.h>
#include <qwt_plot_magnifier.h>
#include <qwt_text.h>
#include <qwt_symbol.h>
#include <qwt_math.h>
#include <math.h>

//-----------------------------------------------------------------
//              simple.cpp
//
//      A simple example which shows how to use QwtPlot connected
//      to a data class without any storage, calculating each values
//      on the fly.
//-----------------------------------------------------------------

class FunctionData: public QwtSyntheticPointData
{
public:
    FunctionData( double( *y )( double ) ):
        QwtSyntheticPointData( 100 ),
        d_y( y )
    {
    }

    virtual double y( double x ) const
    {
        return d_y( x );
    }

private:
    double( *d_y )( double );
};

class ArrowSymbol: public QwtSymbol
{
public:
    ArrowSymbol()
    {
        QPen pen( Qt::black, 0 );
        pen.setJoinStyle( Qt::MiterJoin );

        setPen( pen );
        setBrush( Qt::red );

        QPainterPath path;
        path.moveTo( 0, 8 );
        path.lineTo( 0, 5 );
        path.lineTo( -3, 5 );
        path.lineTo( 0, 0 );
        path.lineTo( 3, 5 );
        path.lineTo( 0, 5 );

        QTransform transform;
        transform.rotate( -30.0 );
        path = transform.map( path );

        setPath( path );
        setPinPoint( QPointF( 0, 0 ) );

        setSize( 10, 14 );
    }
};

class Plot : public QwtPlot
{
public:
    Plot( QWidget *parent = NULL );

protected:
    virtual void resizeEvent( QResizeEvent * );

private:
    void populate();
    void updateGradient();
};


Plot::Plot( QWidget *parent ):
    QwtPlot( parent )
{
    setAutoFillBackground( true );
    setPalette( QPalette( QColor( 165, 193, 228 ) ) );
    updateGradient();

    setTitle( "A Simple QwtPlot Demonstration" );
    insertLegend( new QwtLegend(), QwtPlot::RightLegend );

    // axes
    setAxisTitle( xBottom, "x -->" );
    setAxisScale( xBottom, 0.0, 10.0 );

    setAxisTitle( yLeft, "y -->" );
    setAxisScale( yLeft, -1.0, 1.0 );

    // canvas
    QwtPlotCanvas *canvas = new QwtPlotCanvas();
    canvas->setLineWidth( 1 );
    canvas->setFrameStyle( QFrame::Box | QFrame::Plain );
    canvas->setBorderRadius( 15 );

    QPalette canvasPalette( Qt::white );
    canvasPalette.setColor( QPalette::Foreground, QColor( 133, 190, 232 ) );
    canvas->setPalette( canvasPalette );

    setCanvas( canvas );

    // panning with the left mouse button
    ( void ) new QwtPlotPanner( canvas );

    // zoom in/out with the wheel
    ( void ) new QwtPlotMagnifier( canvas );

    populate();
}

void Plot::populate()
{
    // Insert new curves
    QwtPlotCurve *cSin = new QwtPlotCurve( "y = sin(x)" );
    cSin->setRenderHint( QwtPlotItem::RenderAntialiased );
    cSin->setLegendAttribute( QwtPlotCurve::LegendShowLine, true );
    cSin->setPen( Qt::red );
    cSin->attach( this );

    QwtPlotCurve *cCos = new QwtPlotCurve( "y = cos(x)" );
    cCos->setRenderHint( QwtPlotItem::RenderAntialiased );
    cCos->setLegendAttribute( QwtPlotCurve::LegendShowLine, true );
    cCos->setPen( Qt::blue );
    cCos->attach( this );

    // Create sin and cos data
    cSin->setData( new FunctionData( ::sin ) );
    cCos->setData( new FunctionData( ::cos ) );

    // Insert markers

    //  ...a horizontal line at y = 0...
    QwtPlotMarker *mY = new QwtPlotMarker();
    mY->setLabel( QString::fromLatin1( "y = 0" ) );
    mY->setLabelAlignment( Qt::AlignRight | Qt::AlignTop );
    mY->setLineStyle( QwtPlotMarker::HLine );
    mY->setYValue( 0.0 );
    mY->attach( this );

    //  ...a vertical line at x = 2 * pi
    QwtPlotMarker *mX = new QwtPlotMarker();
    mX->setLabel( QString::fromLatin1( "x = 2 pi" ) );
    mX->setLabelAlignment( Qt::AlignLeft | Qt::AlignBottom );
    mX->setLabelOrientation( Qt::Vertical );
    mX->setLineStyle( QwtPlotMarker::VLine );
    mX->setLinePen( Qt::black, 0, Qt::DashDotLine );
    mX->setXValue( 2.0 * M_PI );
    mX->attach( this );

    const double x = 7.7;

    // an arrow at a specific position
    QwtPlotMarker *mPos = new QwtPlotMarker( "Marker" );
    mPos->setRenderHint( QwtPlotItem::RenderAntialiased, true );
    mPos->setItemAttribute( QwtPlotItem::Legend, true );
    mPos->setSymbol( new ArrowSymbol() );
    mPos->setValue( QPointF( x, ::sin( x ) ) );
    mPos->setLabel( QString( "x = %1" ).arg( x ) );
    mPos->setLabelAlignment( Qt::AlignRight | Qt::AlignBottom );
    mPos->attach( this );
}

void Plot::updateGradient()
{
    QPalette pal = palette();

    const QColor buttonColor = pal.color( QPalette::Button );

    QLinearGradient gradient( rect().topLeft(), rect().bottomLeft() );
    gradient.setColorAt( 0.0, Qt::white );
    gradient.setColorAt( 0.7, buttonColor );
    gradient.setColorAt( 1.0, buttonColor );

    pal.setBrush( QPalette::Window, gradient );
    setPalette( pal );
}

void Plot::resizeEvent( QResizeEvent *event )
{
    QwtPlot::resizeEvent( event );

    // Qt 4.7.1: QGradient::StretchToDeviceMode is buggy on X11
    updateGradient();
}

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    Plot *plot = new Plot();

    // We put a dummy widget around to have
    // so that Qt paints a widget background
    // when resizing

    QWidget window;
    QHBoxLayout *layout = new QHBoxLayout( &window );
    layout->setContentsMargins( 0, 0, 0, 0 );
    layout->addWidget( plot );

    window.resize( 600, 400 );
    window.show();

    return a.exec();
}
```

### `examples/spectrogram/main.cpp`

```cpp
#include <qapplication.h>
#include <qmainwindow.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include <qcombobox.h>
#include <qslider.h>
#include <qlabel.h>
#include <qcheckbox.h>
#include "plot.h"
#include "qwt_color_map.h"

class MainWindow: public QMainWindow
{
public:
    MainWindow( QWidget * = NULL );

private:
    Plot *d_plot;
};

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    d_plot = new Plot( this );

    setCentralWidget( d_plot );

    QToolBar *toolBar = new QToolBar( this );

#ifndef QT_NO_PRINTER
    QToolButton *btnPrint = new QToolButton( toolBar );
    btnPrint->setText( "Print" );
    btnPrint->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    toolBar->addWidget( btnPrint );
    connect( btnPrint, SIGNAL( clicked() ),
        d_plot, SLOT( printPlot() ) );

    toolBar->addSeparator();
#endif

    toolBar->addWidget( new QLabel("Color Map " ) );
    QComboBox *mapBox = new QComboBox( toolBar );
    mapBox->addItem( "RGB" );
    mapBox->addItem( "Indexed Colors" );
    mapBox->addItem( "Hue" );
    mapBox->addItem( "Alpha" );
    mapBox->setSizePolicy( QSizePolicy::Fixed, QSizePolicy::Fixed );
    toolBar->addWidget( mapBox );
    connect( mapBox, SIGNAL( currentIndexChanged( int ) ),
             d_plot, SLOT( setColorMap( int ) ) );

    toolBar->addWidget( new QLabel( " Opacity " ) );
    QSlider *slider = new QSlider( Qt::Horizontal );
    slider->setRange( 0, 255 );
    slider->setValue( 255 );
    connect( slider, SIGNAL( valueChanged( int ) ), 
        d_plot, SLOT( setAlpha( int ) ) );

    toolBar->addWidget( slider );
    toolBar->addWidget( new QLabel("   " ) );

    QCheckBox *btnSpectrogram = new QCheckBox( "Spectrogram", toolBar );
    toolBar->addWidget( btnSpectrogram );
    connect( btnSpectrogram, SIGNAL( toggled( bool ) ),
        d_plot, SLOT( showSpectrogram( bool ) ) );

    QCheckBox *btnContour = new QCheckBox( "Contour", toolBar );
    toolBar->addWidget( btnContour );
    connect( btnContour, SIGNAL( toggled( bool ) ),
        d_plot, SLOT( showContour( bool ) ) );

    addToolBar( toolBar );

    btnSpectrogram->setChecked( true );
    btnContour->setChecked( false );

}

int main( int argc, char **argv )
{
    QApplication a( argc, argv );
    a.setStyle( "Windows" );

    MainWindow mainWindow;
    mainWindow.resize( 600, 400 );
    mainWindow.show();

    return a.exec();
}
```

### `examples/spectrogram/plot.cpp`

```cpp
#include <qprinter.h>
#include <qprintdialog.h>
#include <qnumeric.h>
#include <qwt_color_map.h>
#include <qwt_plot_spectrogram.h>
#include <qwt_scale_widget.h>
#include <qwt_scale_draw.h>
#include <qwt_plot_zoomer.h>
#include <qwt_plot_panner.h>
#include <qwt_plot_layout.h>
#include <qwt_plot_renderer.h>
#include "plot.h"

class MyZoomer: public QwtPlotZoomer
{
public:
    MyZoomer( QWidget *canvas ):
        QwtPlotZoomer( canvas )
    {
        setTrackerMode( AlwaysOn );
    }

    virtual QwtText trackerTextF( const QPointF &pos ) const
    {
        QColor bg( Qt::white );
        bg.setAlpha( 200 );

        QwtText text = QwtPlotZoomer::trackerTextF( pos );
        text.setBackgroundBrush( QBrush( bg ) );
        return text;
    }
};

class SpectrogramData: public QwtRasterData
{
public:
    SpectrogramData()
    {
        setInterval( Qt::XAxis, QwtInterval( -1.5, 1.5 ) );
        setInterval( Qt::YAxis, QwtInterval( -1.5, 1.5 ) );
        setInterval( Qt::ZAxis, QwtInterval( 0.0, 10.0 ) );
    }

    virtual double value( double x, double y ) const
    {
        const double c = 0.842;
        //const double c = 0.33;

        const double v1 = x * x + ( y - c ) * ( y + c );
        const double v2 = x * ( y + c ) + x * ( y + c );

        return 1.0 / ( v1 * v1 + v2 * v2 );
    }
};

class LinearColorMapRGB: public QwtLinearColorMap
{
public:
    LinearColorMapRGB():
        QwtLinearColorMap( Qt::darkCyan, Qt::red, QwtColorMap::RGB )
    {
        addColorStop( 0.1, Qt::cyan );
        addColorStop( 0.6, Qt::green );
        addColorStop( 0.95, Qt::yellow );
    }
};

class LinearColorMapIndexed: public QwtLinearColorMap
{
public:
    LinearColorMapIndexed():
        QwtLinearColorMap( Qt::darkCyan, Qt::red, QwtColorMap::Indexed )
    {
        addColorStop( 0.1, Qt::cyan );
        addColorStop( 0.6, Qt::green );
        addColorStop( 0.95, Qt::yellow );
    }
};

class HueColorMap: public QwtColorMap
{
public:
    // class backported from Qwt 6.2

    HueColorMap():
        d_hue1(0),
        d_hue2(359),
        d_saturation(150),
        d_value(200)
    {
        updateTable();

    }

    virtual QRgb rgb( const QwtInterval &interval, double value ) const
    {
        if ( qIsNaN(value) )
            return 0u;

        const double width = interval.width();
        if ( width <= 0 )
            return 0u;

        if ( value <= interval.minValue() )
            return d_rgbMin;

        if ( value >= interval.maxValue() )
            return d_rgbMax;

        const double ratio = ( value - interval.minValue() ) / width;
        int hue = d_hue1 + qRound( ratio * ( d_hue2 - d_hue1 ) );

        if ( hue >= 360 )
        {
            hue -= 360;

            if ( hue >= 360 )
                hue = hue % 360;
        }

        return d_rgbTable[hue];
    }

    virtual unsigned char colorIndex( const QwtInterval &, double ) const
    {
        // we don't support indexed colors
        return 0;
    }


private:
    void updateTable()
    {
        for ( int i = 0; i < 360; i++ )
            d_rgbTable[i] = QColor::fromHsv( i, d_saturation, d_value ).rgb();

        d_rgbMin = d_rgbTable[ d_hue1 % 360 ];
        d_rgbMax = d_rgbTable[ d_hue2 % 360 ];
    }

    int d_hue1, d_hue2, d_saturation, d_value; 
    QRgb d_rgbMin, d_rgbMax, d_rgbTable[360];
};

class AlphaColorMap: public QwtAlphaColorMap
{
public:
    AlphaColorMap()
    {
        //setColor( QColor("DarkSalmon") );
        setColor( QColor("SteelBlue") );
    }
};

Plot::Plot( QWidget *parent ):
    QwtPlot( parent ),
    d_alpha(255)
{
    d_spectrogram = new QwtPlotSpectrogram();
    d_spectrogram->setRenderThreadCount( 0 ); // use system specific thread count
    d_spectrogram->setCachePolicy( QwtPlotRasterItem::PaintCache );

    QList<double> contourLevels;
    for ( double level = 0.5; level < 10.0; level += 1.0 )
        contourLevels += level;
    d_spectrogram->setContourLevels( contourLevels );

    d_spectrogram->setData( new SpectrogramData() );
    d_spectrogram->attach( this );

    const QwtInterval zInterval = d_spectrogram->data()->interval( Qt::ZAxis );

    // A color bar on the right axis
    QwtScaleWidget *rightAxis = axisWidget( QwtPlot::yRight );
    rightAxis->setTitle( "Intensity" );
    rightAxis->setColorBarEnabled( true );

    setAxisScale( QwtPlot::yRight, zInterval.minValue(), zInterval.maxValue() );
    enableAxis( QwtPlot::yRight );

    plotLayout()->setAlignCanvasToScales( true );

    setColorMap( Plot::RGBMap );

    // LeftButton for the zooming
    // MidButton for the panning
    // RightButton: zoom out by 1
    // Ctrl+RighButton: zoom out to full size

    QwtPlotZoomer* zoomer = new MyZoomer( canvas() );
    zoomer->setMousePattern( QwtEventPattern::MouseSelect2,
        Qt::RightButton, Qt::ControlModifier );
    zoomer->setMousePattern( QwtEventPattern::MouseSelect3,
        Qt::RightButton );

    QwtPlotPanner *panner = new QwtPlotPanner( canvas() );
    panner->setAxisEnabled( QwtPlot::yRight, false );
    panner->setMouseButton( Qt::MidButton );

    // Avoid jumping when labels with more/less digits
    // appear/disappear when scrolling vertically

    const QFontMetrics fm( axisWidget( QwtPlot::yLeft )->font() );
    QwtScaleDraw *sd = axisScaleDraw( QwtPlot::yLeft );
    sd->setMinimumExtent( fm.width( "100.00" ) );

    const QColor c( Qt::darkBlue );
    zoomer->setRubberBandPen( c );
    zoomer->setTrackerPen( c );
}

void Plot::showContour( bool on )
{
    d_spectrogram->setDisplayMode( QwtPlotSpectrogram::ContourMode, on );
    replot();
}

void Plot::showSpectrogram( bool on )
{
    d_spectrogram->setDisplayMode( QwtPlotSpectrogram::ImageMode, on );
    d_spectrogram->setDefaultContourPen( 
        on ? QPen( Qt::black, 0 ) : QPen( Qt::NoPen ) );

    replot();
}

void Plot::setColorMap( int type )
{
    QwtScaleWidget *axis = axisWidget( QwtPlot::yRight );
    const QwtInterval zInterval = d_spectrogram->data()->interval( Qt::ZAxis );

    d_mapType = type;

    int alpha = d_alpha;
    switch( type )
    {
        case Plot::HueMap:
        {
            d_spectrogram->setColorMap( new HueColorMap() );
            axis->setColorMap( zInterval, new HueColorMap() );
            break;
        }
        case Plot::AlphaMap:
        {
            alpha = 255;
            d_spectrogram->setColorMap( new AlphaColorMap() );
            axis->setColorMap( zInterval, new AlphaColorMap() );
            break;
        }
        case Plot::IndexMap:
        {
            d_spectrogram->setColorMap( new LinearColorMapIndexed() );
            axis->setColorMap( zInterval, new LinearColorMapIndexed() );
            break;
        }
        case Plot::RGBMap:
        default:
        {
            d_spectrogram->setColorMap( new LinearColorMapRGB() );
            axis->setColorMap( zInterval, new LinearColorMapRGB() );
        }
    }
    d_spectrogram->setAlpha( alpha );

    replot();
}

void Plot::setAlpha( int alpha )
{
    // setting an alpha value doesn't make sense in combination
    // with a color map interpolating the alpha value

    d_alpha = alpha;
    if ( d_mapType != Plot::AlphaMap )
    {
        d_spectrogram->setAlpha( alpha );
        replot();
    }
}

#ifndef QT_NO_PRINTER

void Plot::printPlot()
{
    QPrinter printer( QPrinter::HighResolution );
    printer.setOrientation( QPrinter::Landscape );
    printer.setOutputFileName( "spectrogram.pdf" );

    QPrintDialog dialog( &printer );
    if ( dialog.exec() )
    {
        QwtPlotRenderer renderer;

        if ( printer.colorMode() == QPrinter::GrayScale )
        {
            renderer.setDiscardFlag( QwtPlotRenderer::DiscardBackground );
            renderer.setDiscardFlag( QwtPlotRenderer::DiscardCanvasBackground );
            renderer.setDiscardFlag( QwtPlotRenderer::DiscardCanvasFrame );
            renderer.setLayoutFlag( QwtPlotRenderer::FrameWithScales );
        }

        renderer.renderTo( this, printer );
    }
}

#endif
```

### `examples/spectrogram/plot.h`

```cpp
#include <qwt_plot.h>
#include <qwt_plot_spectrogram.h>

class Plot: public QwtPlot
{
    Q_OBJECT

public:
    enum ColorMap
    {
        RGBMap,
        IndexMap,
        HueMap,
        AlphaMap
    };

    Plot( QWidget * = NULL );

public Q_SLOTS:
    void showContour( bool on );
    void showSpectrogram( bool on );
    void setColorMap( int );
    void setAlpha( int );

#ifndef QT_NO_PRINTER
    void printPlot();
#endif

private:
    QwtPlotSpectrogram *d_spectrogram;

    int d_mapType;
    int d_alpha;
};
```

### `examples/stockchart/griditem.cpp`

```cpp
#include "griditem.h"
#include <qwt_scale_map.h>
#include <qwt_painter.h>
#include <qpainter.h>

GridItem::GridItem():
    QwtPlotItem( QwtText( "Grid" ) ),
    m_orientations( Qt::Horizontal | Qt::Vertical ),
    m_gridAttributes( AutoUpdate | FillCanvas ),
    m_isXMinEnabled( false ),
    m_isYMinEnabled( false )
{
    setItemInterest( QwtPlotItem::ScaleInterest, true );
    setZ( 10.0 );
}

GridItem::~GridItem()
{
}

int GridItem::rtti() const
{
    return QwtPlotItem::Rtti_PlotUserItem + 99; // something
}

void GridItem::setGridAttribute( GridAttribute attribute, bool on )
{
    if ( bool( m_gridAttributes & attribute ) == on )
        return;

    if ( on )
        m_gridAttributes |= attribute;
    else
        m_gridAttributes &= ~attribute;

    itemChanged();
}

bool GridItem::testGridAttribute( GridAttribute attribute ) const
{
    return m_gridAttributes & attribute;
}

void GridItem::setOrientations( Qt::Orientations orientations )
{
    if ( m_orientations != orientations )
    {
        m_orientations = orientations;
        itemChanged();
    }
}

Qt::Orientations GridItem::orientations() const
{
    return m_orientations;
}

void GridItem::enableXMin( bool enabled )
{
    if ( enabled != m_isXMinEnabled )
    {
        m_isXMinEnabled = enabled;
        itemChanged();
    }
}

bool GridItem::isXMinEnabled() const
{
    return m_isXMinEnabled;
}

void GridItem::enableYMin( bool enabled )
{
    if ( enabled != m_isYMinEnabled )
    {
        m_isYMinEnabled = enabled;
        itemChanged();
    }
}

bool GridItem::isYMinEnabled() const
{
    return m_isYMinEnabled;
}

void GridItem::setXDiv( const QwtScaleDiv &scaleDiv )
{
    if ( m_xScaleDiv != scaleDiv )
    {
        m_xScaleDiv = scaleDiv;
        itemChanged();
    }
}

void GridItem::setYDiv( const QwtScaleDiv &scaleDiv )
{
    if ( m_yScaleDiv != scaleDiv )
    {
        m_yScaleDiv = scaleDiv;
        itemChanged();
    }
}

void GridItem::setPalette( const QPalette &palette )
{
    if ( m_palette != palette )
    {
        m_palette = palette;
        itemChanged();
    }
}

QPalette GridItem::palette() const
{
    return m_palette;
}

void GridItem::draw( QPainter *painter,
    const QwtScaleMap &xMap, const QwtScaleMap &yMap,
    const QRectF &canvasRect ) const
{
    const bool doAlign = QwtPainter::roundingAlignment( painter );

    const QRectF area = QwtScaleMap::invTransform( xMap, yMap, canvasRect );

    QList<double> xValues;
    if ( m_orientations & Qt::Horizontal )
    {
        xValues = m_xScaleDiv.ticks( QwtScaleDiv::MajorTick );

        if ( m_isXMinEnabled )
        {
            xValues += m_xScaleDiv.ticks( QwtScaleDiv::MediumTick );
            xValues += m_xScaleDiv.ticks( QwtScaleDiv::MinorTick );
        }

        if ( m_gridAttributes & FillCanvas )
        {
            xValues += area.left();
            xValues += area.right();
        }

        qSort( xValues );
    }

    QList<double> yValues;
    if ( m_orientations & Qt::Vertical )
    {
        yValues = m_yScaleDiv.ticks( QwtScaleDiv::MajorTick );

        if ( m_isYMinEnabled )
        {
            yValues += m_yScaleDiv.ticks( QwtScaleDiv::MediumTick );
            yValues += m_yScaleDiv.ticks( QwtScaleDiv::MinorTick );
        }

        if ( m_gridAttributes & FillCanvas )
        {
            yValues += area.top();
            yValues += area.bottom();
        }

        qSort( yValues );
    }

    painter->setPen( Qt::NoPen );

    if ( ( m_orientations & Qt::Horizontal ) &&
        ( m_orientations & Qt::Vertical ) )
    {
        for ( int i = 1; i < xValues.size(); i++ )
        {
            double x1 = xMap.transform( xValues[i - 1] );
            double x2 = xMap.transform( xValues[i] );

            if ( doAlign )
            {
                x1 = qRound( x1 );
                x2 = qRound( x2 );
            }

            for ( int j = 1; j < yValues.size(); j++ )
            {
                const QRectF rect( xValues[i - 1], yValues[j - 1],
                    xValues[i] - xValues[i - 1], yValues[j] - yValues[j - 1] );

                painter->setBrush( brush( i - 1, j - 1, rect ) );

                double y1 = yMap.transform( yValues[j - 1] );
                double y2 = yMap.transform( yValues[j] );

                if ( doAlign )
                {
                    y1 = qRound( y1 );
                    y2 = qRound( y2 );
                }

                QwtPainter::drawRect( painter, x1, y1, x2 - x1, y2 - y1 );
            }
        }
    }
    else if ( m_orientations & Qt::Horizontal )
    {
        for ( int i = 1; i < xValues.size(); i++ )
        {
            const QRectF rect( xValues[i - 1], area.top(),
                xValues[i] - xValues[i - 1], area.bottom() );

            painter->setBrush( brush( i - 1, 0, rect ) );

            double x1 = xMap.transform( xValues[i - 1] );
            double x2 = xMap.transform( xValues[i] );

            if ( doAlign )
            {
                x1 = qRound( x1 );
                x2 = qRound( x2 );
            }

            QwtPainter::drawRect( painter,
                x1, canvasRect.top(), x2 - x1, canvasRect.height() );
        }
    }
    else if ( m_orientations & Qt::Vertical )
    {
        for ( int i = 1; i < yValues.size(); i++ )
        {
            const QRectF rect( area.left(), yValues[i - 1],
                area.width(), yValues[i] - yValues[i - 1] );

            painter->setBrush( brush( 0, i - 1, rect ) );

            double y1 = yMap.transform( yValues[i - 1] );
            double y2 = yMap.transform( yValues[i] );

            if ( doAlign )
            {
                y1 = qRound( y1 );
                y2 = qRound( y2 );
            }

            QwtPainter::drawRect( painter, canvasRect.left(), y1,
                                  canvasRect.width(), y2 - y1 );
        }
    }
}

const QwtScaleDiv &GridItem::xScaleDiv() const
{
    return m_xScaleDiv;
}

const QwtScaleDiv &GridItem::yScaleDiv() const
{
    return m_yScaleDiv;
}

void GridItem::updateScaleDiv( 
    const QwtScaleDiv& xScaleDiv, const QwtScaleDiv& yScaleDiv )
{
    if ( m_gridAttributes & AutoUpdate )
    {
        setXDiv( xScaleDiv );
        setYDiv( yScaleDiv );
    }
}

QBrush GridItem::brush( int row, int column, const QRectF & ) const
{
    /*
        We need some sort of origin to avoid, that the brush
        changes for the same rectangle when panning
     */
    if ( ( row + column ) % 2 )
        return QBrush( m_palette.brush( QPalette::Base ) );
    else
        return QBrush( m_palette.brush( QPalette::AlternateBase ) );
}
```

### `examples/stockchart/griditem.h`

```cpp
#ifndef _GRID_ITEM_H_
#define _GRID_ITEM_H_

#include <qwt_plot_item.h>
#include <qwt_scale_div.h>
#include <qpalette.h>

class GridItem: public QwtPlotItem
{
public:
    enum GridAttribute
    {
        AutoUpdate = 0x01,
        FillCanvas       = 0x02
    };

    typedef QFlags<GridAttribute> GridAttributes;

    explicit GridItem();
    virtual ~GridItem();

    virtual int rtti() const;

    void setGridAttribute( GridAttribute, bool on = true );
    bool testGridAttribute( GridAttribute ) const;

    void setOrientations( Qt::Orientations );
    Qt::Orientations orientations() const;

    void enableXMin( bool );
    bool isXMinEnabled() const;

    void enableYMin( bool );
    bool isYMinEnabled() const;

    void setXDiv( const QwtScaleDiv &sx );
    const QwtScaleDiv &xScaleDiv() const;

    void setYDiv( const QwtScaleDiv &sy );
    const QwtScaleDiv &yScaleDiv() const;

    void setPalette( const QPalette & );
    QPalette palette() const;

    virtual void draw( QPainter *p,
        const QwtScaleMap &xMap, const QwtScaleMap &yMap,
        const QRectF &rect ) const;

    virtual void updateScaleDiv(
        const QwtScaleDiv &xMap, const QwtScaleDiv &yMap );

protected:
    virtual QBrush brush( int row, int column, const QRectF & ) const;

private:
    Qt::Orientations m_orientations;
    GridAttributes m_gridAttributes;

    QwtScaleDiv m_xScaleDiv;
    QwtScaleDiv m_yScaleDiv;

    bool m_isXMinEnabled;
    bool m_isYMinEnabled;

    QPalette m_palette;
};

Q_DECLARE_OPERATORS_FOR_FLAGS( GridItem::GridAttributes )

#endif
```

### `examples/stockchart/legend.cpp`

```cpp
#include "legend.h"
#include <qwt_legend_data.h>
#include <qwt_text.h>
#include <qwt_plot_item.h>
#include <qtreeview.h>
#include <qlayout.h>
#include <qstyle.h>
#include <qstandarditemmodel.h>
#include <qitemdelegate.h>
#include <qpainter.h>

static void qwtRenderBackground( QPainter *painter,
    const QRectF &rect, const QWidget *widget )
{
    if ( widget->testAttribute( Qt::WA_StyledBackground ) )
    {
        QStyleOption opt;
        opt.initFrom( widget );
        opt.rect = rect.toAlignedRect();

        widget->style()->drawPrimitive(
            QStyle::PE_Widget, &opt, painter, widget);
    }
    else
    {
        const QBrush brush =
            widget->palette().brush( widget->backgroundRole() );

        painter->fillRect( rect, brush );
    }
}

class LegendTreeView: public QTreeView
{
public:
    LegendTreeView( Legend * );

    QStandardItem *rootItem( int rtti );
    QStandardItem *insertRootItem( int rtti );

    QList<QStandardItem *> itemList( const QwtPlotItem * );

    virtual QSize sizeHint() const;
    virtual QSize minimumSizeHint() const;
};

LegendTreeView::LegendTreeView( Legend *legend ):
    QTreeView( legend )
{
    setFrameStyle( NoFrame );
    viewport()->setBackgroundRole(QPalette::Background);
    viewport()->setAutoFillBackground( false );

    setRootIsDecorated( true );
    setHeaderHidden( true );

    QStandardItemModel *model = new QStandardItemModel();

    setModel( model );

    // we want unstyled items
    setItemDelegate( new QItemDelegate( this ) );
}

QStandardItem *LegendTreeView::rootItem( int rtti )
{
    QStandardItemModel *mdl =
        qobject_cast<QStandardItemModel *>( model() );

    for ( int row = 0; row < mdl->rowCount(); row++ )
    {
        QStandardItem *item = mdl->item( row );
        if ( item->data() == rtti )
            return item;
    }

    return NULL;
}

QList<QStandardItem *> LegendTreeView::itemList( 
    const QwtPlotItem *plotItem ) 
{
    QList<QStandardItem *> itemList;

    const QStandardItem *rootItem = this->rootItem( plotItem->rtti() );
    if ( rootItem )
    {
        for ( int i = 0; i < rootItem->rowCount(); i++ )
        {
            QStandardItem *item = rootItem->child( i );
        
            const QVariant key = item->data();
        
            if ( key.canConvert<qlonglong>() )
            {
                const qlonglong ptr = key.value<qlonglong>();
                if ( ptr == qlonglong( plotItem ) )
                    itemList += item;
            }
        }
    }

    return itemList;
}

QStandardItem *LegendTreeView::insertRootItem( int rtti )
{
    QStandardItem *item = new QStandardItem();
    item->setEditable( false );
    item->setData( rtti );

    switch( rtti )
    {
        case QwtPlotItem::Rtti_PlotTradingCurve:
        {
            item->setText( "Curves" );
            break;
        }
        case QwtPlotItem::Rtti_PlotZone:
        {
            item->setText( "Zones" );
            break;
        }
        case QwtPlotItem::Rtti_PlotMarker:
        {
            item->setText( "Events" );
            break;
        }
        default:
            break;
    }

    QStandardItemModel *mdl =
        qobject_cast<QStandardItemModel *>( model() );

    mdl->appendRow( item );
    setExpanded( mdl->index( mdl->rowCount() - 1, 0 ), true );

    return item;
}

QSize LegendTreeView::minimumSizeHint() const
{
    return QSize( -1, -1 );
}

QSize LegendTreeView::sizeHint() const
{
    QStyleOptionViewItem styleOption;
    styleOption.initFrom( this );

    const QAbstractItemDelegate *delegate = itemDelegate();

    const QStandardItemModel *mdl =
        qobject_cast<const QStandardItemModel *>( model() );

    int w = 0;
    int h = 0;

    for ( int row = 0; row < mdl->rowCount(); row++ )
    {
        const QStandardItem *rootItem = mdl->item( row );

        int wRow = 0;
        for ( int i = 0; i < rootItem->rowCount(); i++ )
        {
            const QSize hint = delegate->sizeHint( styleOption, 
                rootItem->child( i )->index() );

            wRow = qMax( wRow, hint.width() );
            h += hint.height();
        }

        const QSize rootHint = delegate->sizeHint( 
            styleOption, rootItem->index() );

        wRow = qMax( wRow + indentation(), rootHint.width() );
        if ( wRow > w )
            w = wRow;

        if ( rootIsDecorated() )
            w += indentation();

        h += rootHint.height();
    }

    int left, right, top, bottom;
    getContentsMargins( &left, &top, &right, &bottom );

    w += left + right;
    h += top + bottom;

    return QSize( w, h );
}

Legend::Legend( QWidget *parent ):
    QwtAbstractLegend( parent )
{
    d_treeView = new LegendTreeView( this );

    QVBoxLayout *layout = new QVBoxLayout( this );
    layout->setContentsMargins( 0, 0, 0, 0 );
    layout->addWidget( d_treeView );

    connect( d_treeView, SIGNAL( clicked( const QModelIndex & ) ),
        this, SLOT( handleClick( const QModelIndex & ) ) );
}

Legend::~Legend()
{
}

void Legend::renderLegend( QPainter *painter,
    const QRectF &rect, bool fillBackground ) const
{
    if ( fillBackground )
    {
        if ( autoFillBackground() ||
            testAttribute( Qt::WA_StyledBackground ) )
        {
            qwtRenderBackground( painter, rect, d_treeView );
        }
    }

    QStyleOptionViewItem styleOption;
    styleOption.initFrom( this );
    styleOption.decorationAlignment = Qt::AlignCenter;

    const QAbstractItemDelegate *delegate = d_treeView->itemDelegate();

    const QStandardItemModel *mdl =
        qobject_cast<const QStandardItemModel *>( d_treeView->model() );

    painter->save();
    painter->translate( rect.topLeft() );

    for ( int row = 0; row < mdl->rowCount(); row++ )
    {
        const QStandardItem *rootItem = mdl->item( row );

        styleOption.rect = d_treeView->visualRect( rootItem->index() );
        if ( !styleOption.rect.isEmpty() )
            delegate->paint( painter, styleOption, rootItem->index() );

        for ( int i = 0; i < rootItem->rowCount(); i++ )
        {
            const QStandardItem *item = rootItem->child( i );

            styleOption.rect = d_treeView->visualRect( item->index() );
            if ( !styleOption.rect.isEmpty() )
            {
                delegate->paint( painter, styleOption, item->index() );
            }
        }
    }
    painter->restore();
}

bool Legend::isEmpty() const
{
    return d_treeView->model()->rowCount() == 0;
}

int Legend::scrollExtent( Qt::Orientation orientation ) const
{
    Q_UNUSED( orientation );

    return style()->pixelMetric( QStyle::PM_ScrollBarExtent );
}

void Legend::updateLegend( const QVariant &itemInfo,
    const QList<QwtLegendData> &data )
{
    QwtPlotItem *plotItem = qvariant_cast<QwtPlotItem *>( itemInfo );

    QStandardItem *rootItem = d_treeView->rootItem( plotItem->rtti() );
    QList<QStandardItem *> itemList = d_treeView->itemList( plotItem );

    while ( itemList.size() > data.size() )
    {
        QStandardItem *item = itemList.takeLast();
        rootItem->removeRow( item->row() );
    }

    if ( !data.isEmpty() )
    {
        if ( rootItem == NULL )
            rootItem = d_treeView->insertRootItem( plotItem->rtti() );

        while ( itemList.size() < data.size() )
        {
            QStandardItem *item = new QStandardItem();
            item->setEditable( false );
            item->setData( qlonglong( plotItem ) );
            item->setCheckable( true );
            item->setCheckState( plotItem->isVisible() ?
                Qt::Checked : Qt::Unchecked );

            itemList += item;
            rootItem->appendRow( item );
        }

        for ( int i = 0; i < itemList.size(); i++ )
            updateItem( itemList[i], data[i] );
    }
    else
    {
        if ( rootItem && rootItem->rowCount() == 0 )
            d_treeView->model()->removeRow( rootItem->row() );
    }

    d_treeView->updateGeometry();
}

void Legend::updateItem( QStandardItem *item, const QwtLegendData &data )
{
    const QVariant titleValue = data.value( QwtLegendData::TitleRole );

    QwtText title;
    if ( titleValue.canConvert<QwtText>() )
    {
        item->setText( title.text() );
        title = titleValue.value<QwtText>();
    }
    else if ( titleValue.canConvert<QString>() )
    {
        title.setText( titleValue.value<QString>() );
    }
    item->setText( title.text() );

    const QVariant iconValue = data.value( QwtLegendData::IconRole );

    QPixmap pm;
    if ( iconValue.canConvert<QPixmap>() )
        pm = iconValue.value<QPixmap>();

    item->setData(pm, Qt::DecorationRole);
}

void Legend::handleClick( const QModelIndex &index )
{
    const QStandardItemModel *model =
        qobject_cast<QStandardItemModel *>( d_treeView->model() );

    const QStandardItem *item = model->itemFromIndex( index );
    if ( item->isCheckable() )
    {
        const qlonglong ptr = item->data().value<qlonglong>();
    
        Q_EMIT checked( (QwtPlotItem *)ptr, 
            item->checkState() == Qt::Checked, 0 );
    }
}
```

### `examples/stockchart/legend.h`

```cpp
#ifndef _LEGEND_H_
#define _LEGEND_H_

#include <qwt_abstract_legend.h>

class LegendTreeView;
class QStandardItem;
class QModelIndex;
class QwtPlotItem;

class Legend : public QwtAbstractLegend
{
    Q_OBJECT

public:
    explicit Legend( QWidget *parent = NULL );
    virtual ~Legend();

    virtual void renderLegend( QPainter *,
        const QRectF &, bool fillBackground ) const;

    virtual bool isEmpty() const;

    virtual int scrollExtent( Qt::Orientation ) const;

Q_SIGNALS:
    void checked( QwtPlotItem *plotItem, bool on, int index );

public Q_SLOTS:
    virtual void updateLegend( const QVariant &,
        const QList<QwtLegendData> & );

private Q_SLOTS:
    void handleClick( const QModelIndex & );

private:
    void updateItem( QStandardItem *, const QwtLegendData & );

    LegendTreeView *d_treeView;
};

#endif
```

### `examples/stockchart/main.cpp`

```cpp
#include <qapplication.h>
#include <qmainwindow.h>
#include <qcombobox.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include "plot.h"

class MainWindow: public QMainWindow
{
public:
    MainWindow( QWidget * = NULL );

private:
    Plot *d_plot;
};

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    d_plot = new Plot( this );
    setCentralWidget( d_plot );

    QToolBar *toolBar = new QToolBar( this );

    QComboBox *typeBox = new QComboBox( toolBar );
    typeBox->addItem( "Bars" );
    typeBox->addItem( "CandleSticks" );
    typeBox->setCurrentIndex( 1 );
    typeBox->setSizePolicy( QSizePolicy::Fixed, QSizePolicy::Fixed );

    QToolButton *btnExport = new QToolButton( toolBar );
    btnExport->setText( "Export" );
    btnExport->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    connect( btnExport, SIGNAL( clicked() ), d_plot, SLOT( exportPlot() ) );

    toolBar->addWidget( typeBox );
    toolBar->addWidget( btnExport );
    addToolBar( toolBar );

    d_plot->setMode( typeBox->currentIndex() );
    connect( typeBox, SIGNAL( currentIndexChanged( int ) ),
        d_plot, SLOT( setMode( int ) ) );
}

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWindow w;
    w.resize( 600, 400 );
    w.show();

    return a.exec();
}
```

### `examples/stockchart/plot.cpp`

```cpp
#include "plot.h"
#include "legend.h"
#include "griditem.h"
#include "quotefactory.h"
#include <qwt_legend.h>
#include <qwt_plot_tradingcurve.h>
#include <qwt_plot_marker.h>
#include <qwt_plot_zoneitem.h>
#include <qwt_plot_renderer.h>
#include <qwt_plot_zoomer.h>
#include <qwt_plot_panner.h>
#include <qwt_legend_label.h>
#include <qwt_date.h>
#include <qwt_date_scale_engine.h>
#include <qwt_date_scale_draw.h>

class Zoomer: public QwtPlotZoomer
{
public:
    Zoomer( QWidget *canvas ):
        QwtPlotZoomer( canvas )
    {
        setRubberBandPen( QColor( Qt::darkGreen ) );
        setTrackerMode( QwtPlotPicker::AlwaysOn );
    }

protected:
    virtual QwtText trackerTextF( const QPointF &pos ) const
    {
        const QDateTime dt = QwtDate::toDateTime( pos.x() );

        QString s;
        s += QwtDate::toString( QwtDate::toDateTime( pos.x() ),
            "MMM dd hh:mm ", QwtDate::FirstThursday );

        QwtText text( s );
        text.setColor( Qt::white );

        QColor c = rubberBandPen().color();
        text.setBorderPen( QPen( c ) );
        text.setBorderRadius( 6 );
        c.setAlpha( 170 );
        text.setBackgroundBrush( c );

        return text;
    }
};

class DateScaleDraw: public QwtDateScaleDraw
{
public:
    DateScaleDraw( Qt::TimeSpec timeSpec ):
        QwtDateScaleDraw( timeSpec )
    {
        // as we have dates from 2010 only we use
        // format strings without the year

        setDateFormat( QwtDate::Millisecond, "hh:mm:ss:zzz\nddd dd MMM" );
        setDateFormat( QwtDate::Second, "hh:mm:ss\nddd dd MMM" );
        setDateFormat( QwtDate::Minute, "hh:mm\nddd dd MMM" );
        setDateFormat( QwtDate::Hour, "hh:mm\nddd dd MMM" );
        setDateFormat( QwtDate::Day, "ddd dd MMM" );
        setDateFormat( QwtDate::Week, "Www" );
        setDateFormat( QwtDate::Month, "MMM" );
    }
};

class ZoneItem: public QwtPlotZoneItem
{
public:
    ZoneItem( const QString &title )
    {
        setTitle( title );
        setZ( 11 ); // on top the the grid
        setOrientation( Qt::Vertical );
        setItemAttribute( QwtPlotItem::Legend, true );
    }

    void setColor( const QColor &color )
    {
        QColor c = color;

        c.setAlpha( 100 );
        setPen( c );

        c.setAlpha( 20 );
        setBrush( c );
    }

    void setInterval( const QDate &date1, const QDate &date2 )
    {
        const QDateTime dt1( date1, QTime(), Qt::UTC );
        const QDateTime dt2( date2, QTime(), Qt::UTC );

        QwtPlotZoneItem::setInterval( QwtDate::toDouble( dt1 ),
            QwtDate::toDouble( dt2 ) );
    }
};

Plot::Plot( QWidget *parent ):
    QwtPlot( parent )
{
    setTitle( "Trading Chart" );

    QwtDateScaleDraw *scaleDraw = new DateScaleDraw( Qt::UTC );
    QwtDateScaleEngine *scaleEngine = new QwtDateScaleEngine( Qt::UTC );

    setAxisTitle( QwtPlot::xBottom, QString( "2010" ) );
    setAxisScaleDraw( QwtPlot::xBottom, scaleDraw );
    setAxisScaleEngine( QwtPlot::xBottom, scaleEngine );
    setAxisLabelRotation( QwtPlot::xBottom, -50.0 );
    setAxisLabelAlignment( QwtPlot::xBottom, Qt::AlignLeft | Qt::AlignBottom );

    setAxisTitle( QwtPlot::yLeft, QString( "Price [EUR]" ) );

#if 0
    QwtLegend *legend = new QwtLegend;
    legend->setDefaultItemMode( QwtLegendData::Checkable );
    insertLegend( legend, QwtPlot::RightLegend );
#else
    Legend *legend = new Legend;
    insertLegend( legend, QwtPlot::RightLegend );
#endif

    populate();

    // LeftButton for the zooming
    // MidButton for the panning
    // RightButton: zoom out by 1
    // Ctrl+RighButton: zoom out to full size

    Zoomer* zoomer = new Zoomer( canvas() );
    zoomer->setMousePattern( QwtEventPattern::MouseSelect2,
        Qt::RightButton, Qt::ControlModifier );
    zoomer->setMousePattern( QwtEventPattern::MouseSelect3,
        Qt::RightButton );

    QwtPlotPanner *panner = new QwtPlotPanner( canvas() );
    panner->setMouseButton( Qt::MidButton );

    connect( legend, SIGNAL( checked( QwtPlotItem *, bool, int ) ),
        SLOT( showItem( QwtPlotItem *, bool ) ) );
}

void Plot::populate()
{
    GridItem *gridItem = new GridItem();
#if 0
    gridItem->setOrientations( Qt::Horizontal );
#endif
    gridItem->attach( this );

    const Qt::GlobalColor colors[] =
    {
        Qt::red,
        Qt::blue,
        Qt::darkCyan,
        Qt::darkMagenta,
        Qt::darkYellow
    };

    const int numColors = sizeof( colors ) / sizeof( colors[0] );

    for ( int i = 0; i < QuoteFactory::NumStocks; i++ )
    {
        QuoteFactory::Stock stock = static_cast<QuoteFactory::Stock>( i );

        QwtPlotTradingCurve *curve = new QwtPlotTradingCurve();
        curve->setTitle( QuoteFactory::title( stock ) );
        curve->setOrientation( Qt::Vertical );
        curve->setSamples( QuoteFactory::samples2010( stock ) );

        // as we have one sample per day a symbol width of
        // 12h avoids overlapping symbols. We also bound
        // the width, so that is is not scaled below 3 and
        // above 15 pixels.

        curve->setSymbolExtent( 12 * 3600 * 1000.0 );
        curve->setMinSymbolWidth( 3 );
        curve->setMaxSymbolWidth( 15 );

        const Qt::GlobalColor color = colors[ i % numColors ];

        curve->setSymbolPen( color );
        curve->setSymbolBrush( QwtPlotTradingCurve::Decreasing, color );
        curve->setSymbolBrush( QwtPlotTradingCurve::Increasing, Qt::white );
        curve->attach( this );

        showItem( curve, true );
    }

    for ( int i = 0; i < 2; i++ )
    {
        QwtPlotMarker *marker = new QwtPlotMarker();

        marker->setTitle( QString( "Event %1" ).arg( i + 1 ) );
        marker->setLineStyle( QwtPlotMarker::VLine );
        marker->setLinePen( colors[ i % numColors ], 0, Qt::DashLine );
        marker->setVisible( false );

        QDateTime dt( QDate( 2010, 1, 1 ) );
        dt = dt.addDays( 77 * ( i + 1 ) );
        
        marker->setValue( QwtDate::toDouble( dt ), 0.0 );

        marker->setItemAttribute( QwtPlotItem::Legend, true );

        marker->attach( this );
    }

    // to show how QwtPlotZoneItem works

    ZoneItem *zone1 = new ZoneItem( "Zone 1");
    zone1->setColor( Qt::darkBlue );
    zone1->setInterval( QDate( 2010, 3, 10 ), QDate( 2010, 3, 27 ) );
    zone1->setVisible( false );
    zone1->attach( this );

    ZoneItem *zone2 = new ZoneItem( "Zone 2");
    zone2->setColor( Qt::darkMagenta );
    zone2->setInterval( QDate( 2010, 8, 1 ), QDate( 2010, 8, 24 ) );
    zone2->setVisible( false );
    zone2->attach( this );

}

void Plot::setMode( int style )
{
    QwtPlotTradingCurve::SymbolStyle symbolStyle =
        static_cast<QwtPlotTradingCurve::SymbolStyle>( style );

    QwtPlotItemList curves = itemList( QwtPlotItem::Rtti_PlotTradingCurve );
    for ( int i = 0; i < curves.size(); i++ )
    {
        QwtPlotTradingCurve *curve =
            static_cast<QwtPlotTradingCurve *>( curves[i] );
        curve->setSymbolStyle( symbolStyle );
    }

    replot();
}

void Plot::showItem( QwtPlotItem *item, bool on )
{
    item->setVisible( on );
    replot();
}

void Plot::exportPlot()
{
    QwtPlotRenderer renderer;
    renderer.exportTo( this, "stockchart.pdf" );
}
```

### `examples/stockchart/plot.h`

```cpp
#ifndef _PLOT_H_
#define _PLOT_H_

#include <qwt_plot.h>

class Plot: public QwtPlot
{
    Q_OBJECT

public:
    Plot( QWidget * = NULL );

public Q_SLOTS:
    void setMode( int );
    void exportPlot();

private Q_SLOTS:
    void showItem( QwtPlotItem *, bool on );

private:
    void populate();
};

#endif
```

### `examples/stockchart/quotefactory.cpp`

```cpp
#include "quotefactory.h"
#include <qwt_date.h>

typedef struct
{
    int day;

    double open;
    double high;
    double low;
    double close;

} t_Data2010;

static t_Data2010 bmwData[] =
{
    { 3, 31.82, 32.46, 31.82, 32.05 },
    { 4, 31.96, 32.41, 31.78, 32.31 },
    { 5, 32.45, 33.04, 32.36, 32.81 },
    { 6, 32.65, 33.20, 32.38, 33.10 },
    { 7, 33.33, 33.43, 32.51, 32.65 },
    { 10, 32.99, 33.05, 32.11, 32.17 },
    { 11, 32.26, 32.26, 31.10, 31.24 },
    { 12, 31.03, 31.52, 31.01, 31.42 },
    { 13, 31.61, 32.18, 31.50, 31.89 },
    { 14, 32.05, 32.13, 31.36, 31.63 },
    { 17, 31.82, 32.12, 31.43, 32.10 },
    { 18, 32.33, 32.45, 31.65, 32.43 },
    { 19, 32.30, 32.39, 31.67, 31.80 },
    { 20, 32.00, 32.19, 31.16, 31.16 },
    { 21, 31.14, 31.37, 30.32, 30.70 },
    { 24, 30.31, 30.79, 30.05, 30.14 },
    { 25, 30.00, 30.53, 29.40, 30.25 },
    { 26, 29.93, 30.14, 29.38, 29.59 },
    { 27, 29.95, 30.28, 29.49, 29.55 },
    { 28, 29.90, 31.30, 29.85, 30.96 },
    { 31, 30.69, 31.31, 30.56, 31.07 },
    { 32, 31.05, 31.28, 30.58, 31.17 },
    { 33, 31.28, 31.77, 31.01, 31.23 },
    { 34, 31.32, 31.53, 30.21, 30.33 },
    { 35, 30.25, 30.28, 29.43, 29.92 },
    { 38, 30.00, 30.45, 29.33, 29.61 },
    { 39, 29.75, 30.07, 29.35, 29.62 },
    { 40, 29.89, 30.12, 29.55, 29.67 },
    { 41, 29.81, 29.87, 29.02, 29.49 },
    { 42, 29.59, 29.84, 28.28, 29.00 },
    { 45, 29.00, 29.29, 28.46, 28.65 },
    { 46, 28.90, 29.45, 28.60, 29.41 },
    { 47, 29.68, 29.77, 29.35, 29.61 },
    { 48, 29.58, 29.76, 28.45, 29.42 },
    { 49, 29.22, 30.43, 29.01, 30.43 },
    { 52, 30.65, 30.67, 30.06, 30.26 },
    { 53, 30.35, 30.52, 29.53, 29.69 },
    { 54, 29.79, 29.87, 29.18, 29.49 },
    { 55, 29.25, 29.82, 29.06, 29.38 },
    { 56, 29.69, 30.00, 29.55, 29.78 },
    { 59, 30.20, 30.58, 29.95, 30.44 },
    { 60, 30.57, 31.47, 30.49, 31.34 },
    { 61, 31.40, 31.76, 31.08, 31.65 },
    { 62, 31.50, 31.80, 31.34, 31.56 },
    { 63, 31.63, 32.45, 31.63, 32.37 },
    { 66, 32.40, 32.54, 31.81, 31.99 },
    { 67, 31.83, 32.29, 31.58, 32.13 },
    { 68, 32.06, 32.33, 31.81, 32.26 },
    { 69, 32.17, 33.26, 32.16, 32.69 },
    { 70, 32.85, 32.94, 32.44, 32.54 },
    { 73, 32.62, 32.92, 32.54, 32.64 },
    { 74, 32.78, 32.97, 32.55, 32.76 },
    { 75, 32.83, 33.04, 32.45, 32.47 },
    { 76, 32.43, 32.56, 31.98, 32.10 },
    { 77, 32.42, 32.49, 32.02, 32.06 },
    { 80, 31.92, 32.65, 31.87, 32.50 },
    { 81, 32.69, 33.44, 32.61, 33.15 },
    { 82, 33.33, 33.51, 32.92, 33.38 },
    { 83, 33.50, 34.10, 33.49, 34.04 },
    { 84, 33.94, 34.35, 33.81, 34.20 },
    { 87, 34.40, 34.73, 34.01, 34.12 },
    { 88, 34.26, 34.43, 33.71, 33.78 },
    { 89, 33.88, 34.29, 33.78, 34.18 },
    { 90, 35.11, 35.49, 34.97, 35.15 },
    { 95, 35.40, 35.45, 35.15, 35.41 },
    { 96, 35.34, 35.41, 34.77, 34.80 },
    { 97, 34.80, 35.06, 34.44, 34.53 },
    { 98, 34.88, 35.05, 34.64, 34.86 },
    { 101, 35.25, 35.39, 34.99, 35.12 },
    { 102, 35.06, 35.38, 34.88, 35.35 },
    { 103, 35.06, 35.58, 34.88, 35.51 },
    { 104, 35.59, 35.61, 35.09, 35.33 },
    { 105, 35.15, 36.19, 35.15, 35.56 },
    { 108, 35.45, 35.78, 35.10, 35.31 },
    { 109, 36.56, 37.08, 36.41, 36.79 },
    { 110, 36.75, 36.99, 36.37, 36.58 },
    { 111, 36.63, 37.12, 35.93, 36.25 },
    { 112, 36.60, 37.40, 36.33, 37.28 },
    { 115, 37.60, 37.85, 37.26, 37.82 },
    { 116, 37.85, 37.96, 37.06, 37.06 },
    { 117, 36.80, 37.28, 36.14, 36.79 },
    { 118, 36.70, 36.90, 36.19, 36.78 },
    { 119, 36.83, 37.62, 36.70, 37.13 },
    { 122, 37.08, 37.50, 36.72, 37.38 },
    { 123, 37.51, 37.56, 35.38, 35.84 },
    { 124, 36.61, 36.62, 35.42, 35.98 },
    { 125, 35.45, 37.38, 35.45, 36.42 },
    { 126, 35.78, 36.90, 35.05, 35.48 },
    { 129, 36.23, 37.74, 36.20, 37.68 },
    { 130, 36.87, 38.19, 36.73, 38.18 },
    { 131, 37.97, 39.35, 37.74, 39.00 },
    { 132, 39.35, 40.06, 39.15, 39.52 },
    { 133, 39.42, 39.88, 38.46, 38.62 },
    { 136, 38.38, 39.59, 38.25, 38.72 },
    { 137, 39.10, 39.65, 38.90, 39.65 },
    { 138, 38.15, 38.70, 36.97, 37.00 },
    { 139, 37.44, 37.55, 35.43, 36.18 },
    { 140, 36.20, 36.57, 35.28, 36.03 },
    { 143, 36.30, 36.38, 35.41, 36.14 },
    { 144, 35.56, 35.67, 34.64, 35.29 },
    { 145, 35.80, 36.32, 35.50, 35.76 },
    { 146, 36.30, 37.33, 36.06, 37.21 },
    { 147, 37.42, 37.88, 37.02, 37.67 },
    { 150, 37.57, 38.09, 37.49, 37.97 },
    { 151, 37.96, 38.38, 36.98, 38.06 },
    { 152, 37.80, 38.46, 37.37, 38.46 },
    { 153, 39.24, 39.55, 38.94, 39.22 },
    { 154, 39.35, 39.40, 37.82, 38.10 },
    { 157, 37.40, 38.55, 37.40, 38.24 },
    { 158, 38.33, 38.54, 37.31, 37.68 },
    { 159, 37.85, 38.98, 37.76, 38.91 },
    { 160, 38.85, 40.92, 38.68, 40.65 },
    { 161, 40.95, 41.27, 39.72, 40.08 },
    { 164, 40.59, 40.85, 39.56, 39.76 },
    { 165, 39.35, 40.05, 39.34, 39.85 },
    { 166, 40.18, 40.41, 38.80, 39.03 },
    { 167, 38.91, 39.96, 38.74, 39.70 },
    { 168, 39.85, 40.87, 39.82, 40.71 },
    { 171, 41.70, 42.33, 41.43, 41.80 },
    { 172, 41.55, 41.88, 41.06, 41.51 },
    { 173, 41.11, 42.01, 41.07, 41.49 },
    { 174, 41.97, 42.19, 41.25, 41.36 },
    { 175, 41.36, 41.38, 40.22, 40.36 },
    { 178, 40.66, 41.64, 40.36, 41.26 },
    { 179, 40.84, 40.88, 39.87, 39.90 },
    { 180, 40.10, 40.61, 39.80, 40.06 },
    { 181, 39.56, 39.56, 38.08, 38.20 },
    { 182, 38.83, 39.20, 37.79, 37.88 },
    { 185, 38.10, 38.53, 37.91, 38.11 },
    { 186, 38.29, 39.27, 38.29, 39.00 },
    { 187, 38.70, 39.87, 38.62, 39.75 },
    { 188, 39.62, 39.97, 38.87, 38.91 },
    { 189, 39.30, 39.39, 38.60, 39.15 },
    { 192, 39.30, 39.30, 38.87, 38.90 },
    { 193, 39.00, 42.14, 39.00, 42.13 },
    { 194, 42.42, 42.71, 40.99, 41.54 },
    { 195, 41.75, 42.94, 41.36, 42.26 },
    { 196, 42.26, 43.29, 41.80, 42.15 },
    { 199, 41.85, 42.09, 41.17, 41.35 },
    { 200, 42.00, 42.12, 40.60, 41.07 },
    { 201, 41.30, 41.80, 40.61, 40.92 },
    { 202, 40.83, 42.35, 40.79, 41.97 },
    { 203, 41.95, 42.24, 41.58, 41.99 },
    { 206, 42.17, 42.29, 41.61, 42.11 },
    { 207, 42.24, 42.49, 41.21, 41.50 },
    { 208, 41.68, 41.88, 40.41, 40.72 },
    { 209, 40.77, 41.22, 40.40, 40.72 },
    { 210, 40.44, 41.40, 39.96, 41.31 },
    { 213, 41.46, 42.01, 41.02, 41.87 },
    { 214, 42.75, 44.04, 42.75, 43.16 },
    { 215, 43.14, 43.83, 42.49, 43.68 },
    { 216, 43.69, 44.99, 43.47, 44.51 },
    { 217, 44.90, 45.38, 43.72, 43.90 },
    { 220, 44.49, 44.60, 43.97, 44.31 },
    { 221, 44.35, 44.40, 43.15, 43.35 },
    { 222, 43.05, 43.08, 42.33, 42.40 },
    { 223, 42.30, 42.92, 40.78, 41.90 },
    { 224, 42.02, 42.22, 41.28, 41.88 },
    { 227, 42.08, 42.29, 41.40, 41.81 },
    { 228, 41.81, 43.10, 41.74, 43.10 },
    { 229, 43.02, 43.59, 42.76, 43.50 },
    { 230, 43.68, 44.07, 42.66, 42.84 },
    { 231, 42.84, 42.92, 41.74, 41.87 },
    { 234, 42.00, 42.31, 41.60, 41.86 },
    { 235, 41.56, 41.76, 41.10, 41.52 },
    { 236, 41.22, 41.97, 40.83, 41.44 },
    { 237, 41.56, 41.96, 41.35, 41.69 },
    { 238, 41.60, 41.81, 40.74, 41.76 },
    { 241, 41.76, 41.90, 40.94, 41.21 },
    { 242, 40.50, 41.67, 40.15, 41.67 },
    { 243, 42.00, 42.99, 41.38, 42.91 },
    { 244, 42.64, 43.89, 42.64, 43.60 },
    { 245, 43.60, 44.53, 43.26, 44.10 },
    { 248, 44.17, 44.20, 43.47, 44.03 },
    { 249, 43.97, 44.31, 43.51, 43.94 },
    { 250, 43.72, 44.99, 43.60, 44.99 },
    { 251, 44.70, 45.74, 44.51, 45.40 },
    { 252, 45.00, 46.87, 44.99, 46.21 },
    { 255, 46.65, 47.05, 45.91, 46.44 },
    { 256, 46.30, 47.12, 46.21, 47.12 },
    { 257, 46.98, 47.56, 46.88, 47.25 },
    { 258, 47.18, 47.45, 46.82, 47.35 },
    { 259, 47.81, 48.03, 47.10, 47.41 },
    { 262, 47.37, 49.12, 47.22, 49.12 },
    { 263, 48.85, 49.42, 48.45, 48.48 },
    { 264, 48.48, 48.70, 47.57, 48.08 },
    { 265, 48.49, 48.69, 47.49, 48.29 },
    { 266, 48.09, 50.53, 48.03, 50.35 },
    { 269, 50.15, 50.35, 49.60, 50.15 },
    { 270, 49.80, 50.69, 49.31, 50.67 },
    { 271, 51.00, 51.84, 50.64, 51.06 },
    { 272, 50.90, 52.15, 50.50, 51.44 },
    { 273, 51.44, 51.44, 49.12, 49.30 },
    { 276, 49.06, 49.19, 47.92, 48.22 },
    { 277, 48.37, 49.96, 47.82, 49.96 },
    { 278, 49.77, 50.05, 49.13, 49.49 },
    { 279, 49.31, 50.25, 48.81, 50.00 },
    { 280, 50.26, 50.29, 49.42, 50.07 },
    { 283, 50.20, 50.62, 49.82, 49.87 },
    { 284, 49.44, 50.49, 49.06, 50.20 },
    { 285, 50.40, 50.49, 49.88, 50.07 },
    { 286, 50.50, 50.50, 49.74, 50.00 },
    { 287, 50.08, 50.25, 49.19, 49.45 },
    { 290, 49.23, 49.42, 48.58, 49.00 },
    { 291, 48.99, 49.69, 48.84, 49.12 },
    { 292, 49.09, 49.60, 48.90, 49.60 },
    { 293, 49.54, 50.09, 49.31, 50.02 },
    { 294, 50.19, 50.44, 49.54, 50.03 },
    { 297, 50.31, 51.02, 50.20, 50.72 },
    { 298, 50.49, 50.94, 50.12, 50.44 },
    { 299, 50.04, 50.45, 49.10, 49.88 },
    { 300, 50.15, 50.48, 49.53, 49.85 },
    { 301, 49.49, 51.65, 49.44, 51.51 },
    { 304, 51.77, 52.99, 51.65, 52.96 },
    { 305, 52.70, 52.70, 52.10, 52.35 },
    { 306, 50.75, 52.38, 50.65, 51.64 },
    { 307, 52.05, 54.15, 52.00, 54.08 },
    { 308, 54.14, 54.99, 53.76, 54.06 },
    { 311, 53.69, 53.77, 52.86, 53.41 },
    { 312, 53.40, 54.98, 53.22, 54.91 },
    { 313, 54.60, 54.70, 53.33, 53.75 },
    { 314, 54.00, 54.49, 53.60, 54.42 },
    { 315, 53.33, 55.90, 52.85, 55.29 },
    { 318, 55.07, 56.52, 54.90, 56.06 },
    { 319, 55.68, 55.83, 54.62, 54.62 },
    { 320, 54.72, 54.73, 53.87, 54.30 },
    { 321, 54.96, 56.30, 54.94, 56.30 },
    { 322, 56.34, 56.73, 55.65, 56.67 },
    { 325, 57.33, 58.90, 57.30, 57.69 },
    { 326, 57.15, 58.62, 56.39, 56.47 },
    { 327, 57.01, 59.12, 56.48, 59.12 },
    { 328, 59.10, 60.00, 58.84, 59.90 },
    { 329, 59.31, 59.76, 58.13, 59.25 },
    { 332, 59.75, 59.91, 57.74, 57.74 },
    { 333, 57.70, 59.24, 57.22, 57.93 },
    { 334, 58.35, 60.90, 58.35, 60.90 },
    { 335, 61.69, 63.80, 61.55, 63.80 },
    { 336, 63.70, 65.49, 63.48, 63.69 },
    { 339, 64.00, 64.53, 62.75, 62.81 },
    { 340, 63.00, 64.49, 62.40, 63.98 },
    { 341, 63.50, 63.50, 61.90, 61.90 },
    { 342, 62.42, 62.66, 58.88, 60.20 },
    { 343, 60.50, 62.99, 60.39, 62.52 },
    { 346, 62.00, 63.44, 62.00, 63.44 },
    { 347, 63.40, 63.44, 62.14, 62.47 },
    { 348, 62.00, 62.83, 61.40, 62.49 },
    { 349, 62.40, 63.26, 61.79, 62.80 },
    { 350, 62.95, 63.15, 61.80, 61.95 },
    { 353, 61.90, 63.23, 61.64, 63.15 },
    { 354, 63.40, 64.80, 62.92, 64.80 },
    { 355, 64.98, 65.11, 64.30, 64.37 },
    { 356, 64.55, 64.69, 63.24, 63.26 },
    { 360, 62.70, 62.70, 59.12, 59.22 },
    { 361, 59.69, 59.98, 57.66, 58.25 },
    { 362, 58.10, 58.92, 58.08, 58.72 },
    { 363, 59.10, 59.47, 58.62, 58.85 }
};

static t_Data2010 porscheData[] =
{
    { 3, 43.00, 43.96, 42.80, 43.37 },
    { 4, 43.15, 45.00, 43.00, 44.77 },
    { 5, 45.75, 46.50, 45.41, 45.65 },
    { 6, 45.67, 48.56, 45.32, 48.28 },
    { 7, 48.78, 48.81, 47.39, 48.00 },
    { 10, 48.26, 49.18, 47.86, 48.35 },
    { 11, 48.35, 48.65, 46.73, 47.05 },
    { 12, 46.51, 47.65, 46.35, 47.37 },
    { 13, 48.10, 48.70, 47.00, 48.13 },
    { 14, 48.10, 48.20, 46.79, 47.85 },
    { 17, 47.85, 48.57, 47.58, 48.10 },
    { 18, 47.85, 48.00, 46.51, 47.65 },
    { 19, 47.24, 47.62, 45.86, 46.40 },
    { 20, 46.51, 46.61, 44.87, 45.00 },
    { 21, 45.00, 45.11, 42.92, 43.50 },
    { 24, 43.00, 43.83, 42.48, 42.97 },
    { 25, 42.47, 43.37, 41.90, 43.23 },
    { 26, 43.00, 43.00, 41.55, 42.28 },
    { 27, 42.80, 42.83, 41.65, 41.72 },
    { 28, 40.91, 41.50, 40.10, 41.11 },
    { 31, 40.85, 41.85, 40.81, 41.55 },
    { 32, 41.69, 43.16, 41.28, 42.87 },
    { 33, 43.47, 43.53, 42.30, 42.47 },
    { 34, 42.67, 42.85, 40.95, 41.15 },
    { 35, 40.81, 40.82, 39.56, 40.03 },
    { 38, 40.00, 40.94, 38.45, 38.95 },
    { 39, 38.65, 38.95, 37.83, 38.24 },
    { 40, 38.30, 38.65, 37.92, 38.30 },
    { 41, 38.40, 39.88, 37.91, 38.36 },
    { 42, 38.60, 38.84, 36.06, 36.99 },
    { 45, 37.31, 37.58, 35.85, 36.06 },
    { 46, 36.45, 36.78, 35.90, 36.78 },
    { 47, 37.01, 37.84, 36.14, 37.42 },
    { 48, 37.40, 37.73, 36.03, 37.16 },
    { 49, 36.90, 38.00, 36.72, 37.97 },
    { 52, 37.52, 38.12, 37.14, 37.14 },
    { 53, 37.22, 37.53, 36.34, 36.69 },
    { 54, 36.88, 36.93, 35.94, 36.55 },
    { 55, 36.35, 37.06, 35.75, 36.09 },
    { 56, 36.70, 37.05, 36.10, 36.90 },
    { 59, 37.10, 37.74, 36.78, 37.63 },
    { 60, 37.65, 38.58, 37.65, 38.56 },
    { 61, 38.35, 39.60, 38.35, 39.42 },
    { 62, 39.39, 40.15, 39.10, 39.70 },
    { 63, 39.75, 40.60, 39.10, 40.35 },
    { 66, 40.40, 40.40, 39.55, 39.97 },
    { 67, 40.05, 40.10, 39.13, 39.90 },
    { 68, 39.78, 40.55, 39.52, 40.37 },
    { 69, 39.86, 42.53, 39.62, 42.34 },
    { 70, 42.75, 44.73, 42.66, 43.03 },
    { 73, 43.27, 43.49, 42.60, 42.65 },
    { 74, 42.78, 43.78, 42.78, 43.78 },
    { 75, 43.73, 44.00, 42.57, 43.46 },
    { 76, 44.10, 44.51, 43.50, 44.51 },
    { 77, 44.40, 44.70, 44.04, 44.04 },
    { 80, 44.00, 44.05, 43.03, 43.69 },
    { 81, 43.13, 43.51, 42.08, 43.17 },
    { 82, 42.89, 44.71, 42.65, 44.20 },
    { 83, 44.31, 44.47, 43.59, 44.22 },
    { 84, 44.15, 45.15, 44.00, 45.13 },
    { 87, 45.45, 46.10, 45.20, 45.51 },
    { 88, 45.76, 46.10, 44.83, 45.17 },
    { 89, 45.60, 45.60, 44.90, 45.19 },
    { 90, 45.60, 46.46, 45.60, 46.37 },
    { 95, 46.00, 47.44, 46.00, 47.24 },
    { 96, 47.22, 47.48, 45.76, 46.04 },
    { 97, 45.05, 45.55, 44.04, 44.41 },
    { 98, 44.88, 45.44, 44.44, 44.99 },
    { 101, 45.20, 45.57, 44.88, 45.35 },
    { 102, 45.10, 46.03, 45.02, 45.71 },
    { 103, 46.02, 46.60, 45.54, 46.30 },
    { 104, 46.44, 46.60, 45.72, 46.04 },
    { 105, 45.80, 46.35, 44.67, 44.67 },
    { 108, 44.50, 45.17, 43.79, 43.83 },
    { 109, 45.39, 46.00, 44.66, 45.92 },
    { 110, 46.00, 46.46, 45.26, 46.26 },
    { 111, 46.29, 46.64, 44.94, 45.20 },
    { 112, 45.69, 46.22, 45.22, 45.97 },
    { 115, 46.30, 46.71, 45.85, 46.69 },
    { 116, 46.48, 46.48, 45.01, 45.01 },
    { 117, 44.60, 45.03, 42.97, 44.03 },
    { 118, 43.50, 44.38, 42.80, 43.76 },
    { 119, 43.40, 44.33, 42.85, 43.69 },
    { 122, 43.70, 43.70, 42.38, 42.71 },
    { 123, 42.95, 42.95, 40.39, 40.53 },
    { 124, 39.99, 40.15, 38.76, 39.95 },
    { 125, 39.05, 40.25, 37.17, 37.40 },
    { 126, 36.30, 37.25, 34.80, 35.58 },
    { 129, 37.54, 38.19, 37.12, 37.92 },
    { 130, 37.79, 38.08, 37.30, 38.08 },
    { 131, 37.94, 39.99, 37.78, 39.73 },
    { 132, 39.80, 40.20, 39.19, 39.73 },
    { 133, 39.35, 39.40, 36.61, 36.72 },
    { 136, 36.29, 38.48, 36.29, 37.58 },
    { 137, 38.33, 38.58, 37.64, 38.47 },
    { 138, 37.77, 38.42, 36.63, 36.67 },
    { 139, 36.40, 36.67, 33.83, 34.72 },
    { 140, 33.85, 34.69, 32.89, 34.07 },
    { 143, 34.49, 35.03, 33.15, 34.40 },
    { 144, 33.40, 33.42, 32.15, 32.54 },
    { 145, 33.28, 34.19, 32.76, 33.49 },
    { 146, 33.85, 35.77, 33.78, 35.49 },
    { 147, 35.99, 36.35, 35.08, 35.53 },
    { 150, 35.24, 35.81, 35.17, 35.34 },
    { 151, 35.21, 36.10, 34.42, 35.24 },
    { 152, 34.55, 35.20, 34.29, 34.85 },
    { 153, 35.63, 36.09, 35.29, 35.70 },
    { 154, 35.98, 35.98, 34.38, 34.50 },
    { 157, 34.45, 34.65, 32.94, 33.26 },
    { 158, 33.50, 33.65, 31.60, 31.91 },
    { 159, 32.42, 33.29, 32.00, 33.22 },
    { 160, 33.10, 33.97, 32.50, 33.58 },
    { 161, 33.97, 35.12, 33.83, 34.85 },
    { 164, 34.90, 35.70, 34.87, 34.97 },
    { 165, 34.40, 34.58, 32.86, 33.46 },
    { 166, 33.87, 33.87, 32.51, 32.87 },
    { 167, 33.17, 34.65, 32.62, 34.60 },
    { 168, 35.58, 35.64, 34.69, 35.06 },
    { 171, 36.52, 37.19, 36.00, 37.00 },
    { 172, 36.87, 37.48, 36.44, 36.85 },
    { 173, 36.38, 36.98, 36.05, 36.40 },
    { 174, 36.00, 36.50, 34.81, 35.08 },
    { 175, 35.21, 36.24, 34.53, 36.04 },
    { 178, 36.76, 37.24, 36.35, 37.16 },
    { 179, 36.49, 36.88, 35.75, 35.81 },
    { 180, 35.92, 36.28, 35.08, 35.29 },
    { 181, 35.00, 35.00, 33.49, 33.54 },
    { 182, 34.00, 34.40, 33.51, 33.51 },
    { 185, 33.80, 34.14, 33.60, 33.74 },
    { 186, 33.75, 35.79, 33.75, 34.96 },
    { 187, 34.85, 35.88, 34.48, 35.88 },
    { 188, 36.00, 36.64, 35.75, 35.96 },
    { 189, 36.41, 36.80, 35.85, 36.72 },
    { 192, 36.60, 37.22, 36.55, 37.08 },
    { 193, 37.10, 38.47, 37.10, 38.08 },
    { 194, 38.19, 38.35, 37.15, 37.49 },
    { 195, 37.35, 37.81, 36.60, 36.87 },
    { 196, 36.76, 37.22, 36.50, 36.74 },
    { 199, 36.51, 36.83, 35.95, 36.08 },
    { 200, 36.12, 36.35, 35.07, 35.42 },
    { 201, 35.40, 36.30, 34.87, 35.08 },
    { 202, 35.00, 37.66, 34.75, 37.47 },
    { 203, 37.70, 39.50, 37.55, 39.12 },
    { 206, 39.43, 39.46, 38.78, 39.18 },
    { 207, 39.30, 39.56, 38.69, 38.98 },
    { 208, 39.00, 39.19, 38.00, 38.23 },
    { 209, 38.10, 39.42, 37.13, 38.72 },
    { 210, 38.88, 39.38, 38.22, 38.82 },
    { 213, 39.26, 39.50, 38.72, 39.01 },
    { 214, 39.07, 40.04, 38.74, 39.10 },
    { 215, 38.85, 39.76, 38.71, 39.29 },
    { 216, 39.30, 39.99, 39.13, 39.53 },
    { 217, 39.50, 40.00, 38.06, 38.32 },
    { 220, 38.60, 39.55, 38.37, 39.38 },
    { 221, 39.48, 39.58, 38.18, 38.56 },
    { 222, 38.58, 38.58, 37.01, 37.31 },
    { 223, 37.32, 37.78, 36.42, 36.82 },
    { 224, 37.30, 37.30, 36.04, 36.53 },
    { 227, 37.00, 37.31, 36.30, 37.12 },
    { 228, 37.00, 38.17, 36.88, 38.00 },
    { 229, 38.10, 38.65, 37.60, 38.58 },
    { 230, 38.60, 39.25, 37.50, 37.88 },
    { 231, 37.85, 37.93, 36.92, 37.26 },
    { 234, 37.53, 38.09, 36.99, 37.64 },
    { 235, 37.59, 37.59, 36.35, 36.80 },
    { 236, 36.50, 36.88, 35.10, 35.93 },
    { 237, 36.40, 36.75, 36.00, 36.35 },
    { 238, 36.50, 36.96, 35.79, 36.78 },
    { 241, 36.91, 37.62, 36.80, 37.15 },
    { 242, 36.45, 36.78, 36.00, 36.74 },
    { 243, 36.82, 38.55, 36.34, 38.26 },
    { 244, 38.67, 39.39, 38.12, 39.26 },
    { 245, 39.28, 39.53, 38.83, 39.29 },
    { 248, 39.40, 39.49, 39.03, 39.28 },
    { 249, 39.30, 39.30, 38.56, 38.80 },
    { 250, 38.55, 39.04, 38.06, 38.89 },
    { 251, 39.00, 39.03, 38.50, 38.90 },
    { 252, 38.90, 39.43, 38.15, 38.30 },
    { 255, 38.76, 38.76, 37.88, 38.09 },
    { 256, 38.37, 38.60, 37.90, 38.42 },
    { 257, 38.69, 39.13, 38.11, 38.68 },
    { 258, 38.27, 38.46, 37.31, 37.41 },
    { 259, 37.88, 38.10, 37.31, 37.60 },
    { 262, 37.62, 37.75, 37.12, 37.52 },
    { 263, 37.50, 37.50, 36.82, 36.83 },
    { 264, 36.95, 37.33, 36.12, 36.12 },
    { 265, 36.07, 36.15, 34.55, 35.62 },
    { 266, 35.29, 36.61, 35.07, 36.53 },
    { 269, 36.60, 36.94, 36.12, 36.50 },
    { 270, 36.15, 36.15, 35.19, 35.51 },
    { 271, 35.51, 36.30, 35.13, 35.56 },
    { 272, 35.96, 36.85, 35.42, 36.33 },
    { 273, 36.50, 36.83, 36.04, 36.31 },
    { 276, 36.44, 36.51, 34.64, 34.74 },
    { 277, 34.75, 35.10, 34.45, 34.99 },
    { 278, 35.45, 35.45, 35.01, 35.20 },
    { 279, 35.50, 35.50, 34.72, 35.10 },
    { 280, 34.80, 35.33, 34.66, 35.25 },
    { 283, 35.12, 36.47, 35.12, 36.33 },
    { 284, 36.12, 37.65, 35.83, 37.16 },
    { 285, 37.40, 40.35, 37.18, 39.00 },
    { 286, 39.30, 40.75, 39.03, 40.34 },
    { 287, 40.80, 42.35, 40.30, 41.53 },
    { 290, 42.00, 42.80, 41.35, 42.70 },
    { 291, 42.70, 43.16, 38.75, 38.97 },
    { 292, 38.60, 40.00, 37.70, 39.79 },
    { 293, 39.61, 39.79, 38.00, 38.40 },
    { 294, 38.29, 38.29, 37.25, 37.60 },
    { 297, 37.73, 38.39, 37.49, 38.06 },
    { 298, 38.02, 38.19, 37.60, 37.99 },
    { 299, 37.90, 38.36, 37.31, 37.49 },
    { 300, 37.40, 37.81, 37.05, 37.19 },
    { 301, 37.00, 37.29, 35.92, 36.81 },
    { 304, 37.00, 37.21, 36.69, 36.90 },
    { 305, 37.00, 37.19, 36.83, 37.01 },
    { 306, 37.97, 38.08, 37.38, 37.65 },
    { 307, 38.39, 38.96, 38.01, 38.71 },
    { 308, 38.70, 40.38, 38.60, 40.02 },
    { 311, 40.02, 40.77, 40.01, 40.60 },
    { 312, 40.40, 43.71, 40.40, 43.54 },
    { 313, 43.00, 44.04, 41.28, 43.33 },
    { 314, 43.00, 44.54, 43.00, 43.94 },
    { 315, 43.30, 44.55, 42.54, 44.55 },
    { 318, 44.10, 47.18, 44.05, 46.60 },
    { 319, 46.45, 47.84, 45.97, 46.55 },
    { 320, 46.40, 47.57, 46.22, 47.38 },
    { 321, 47.85, 49.12, 47.62, 48.97 },
    { 322, 49.30, 50.00, 47.94, 50.00 },
    { 325, 50.20, 53.75, 50.10, 52.43 },
    { 326, 51.90, 54.94, 50.21, 53.07 },
    { 327, 53.20, 56.41, 53.20, 56.41 },
    { 328, 56.98, 59.63, 56.76, 59.27 },
    { 329, 59.00, 60.20, 53.65, 57.70 },
    { 332, 58.30, 59.40, 55.65, 55.88 },
    { 333, 55.85, 58.80, 54.50, 57.93 },
    { 334, 59.37, 61.72, 58.86, 61.43 },
    { 335, 62.93, 65.00, 62.20, 64.54 },
    { 336, 65.50, 66.50, 63.82, 64.80 },
    { 339, 66.00, 66.50, 65.05, 65.70 },
    { 340, 67.60, 69.88, 67.29, 68.57 },
    { 341, 67.90, 67.99, 63.65, 64.69 },
    { 342, 65.90, 66.18, 61.14, 62.02 },
    { 343, 61.42, 66.11, 61.29, 66.00 },
    { 346, 65.63, 67.60, 65.20, 67.19 },
    { 347, 66.99, 67.08, 65.07, 65.19 },
    { 348, 64.90, 65.66, 63.35, 65.50 },
    { 349, 65.07, 66.10, 63.21, 63.48 },
    { 350, 64.28, 64.72, 61.50, 62.06 },
    { 353, 62.00, 63.54, 61.88, 62.25 },
    { 354, 63.00, 63.80, 62.53, 63.75 },
    { 355, 63.79, 64.94, 63.00, 63.56 },
    { 356, 63.62, 64.25, 62.34, 62.48 },
    { 360, 62.21, 62.40, 57.51, 59.40 },
    { 361, 60.00, 60.81, 59.41, 60.07 },
    { 362, 60.10, 60.65, 60.00, 60.30 },
    { 363, 60.30, 60.51, 59.40, 59.66 }
};

static t_Data2010 daimlerData[] =
{
    { 3, 37.24, 37.60, 36.96, 37.55 },
    { 4, 37.50, 37.56, 36.87, 37.24 },
    { 5, 37.19, 37.33, 36.62, 37.25 },
    { 6, 36.85, 36.95, 36.35, 36.72 },
    { 7, 36.92, 37.15, 36.24, 36.94 },
    { 10, 37.19, 37.67, 37.04, 37.20 },
    { 11, 37.34, 37.40, 36.16, 36.28 },
    { 12, 36.00, 36.38, 35.60, 36.19 },
    { 13, 36.60, 37.19, 36.47, 37.10 },
    { 14, 37.00, 37.26, 36.31, 36.49 },
    { 17, 36.58, 37.25, 36.25, 37.12 },
    { 18, 36.65, 36.87, 35.73, 36.71 },
    { 19, 36.44, 36.52, 35.33, 35.60 },
    { 20, 35.90, 36.17, 34.69, 34.76 },
    { 21, 34.40, 34.66, 33.28, 34.18 },
    { 24, 33.56, 34.08, 33.22, 33.50 },
    { 25, 33.15, 33.95, 32.70, 33.86 },
    { 26, 33.46, 33.58, 32.60, 32.92 },
    { 27, 33.53, 33.80, 32.32, 32.32 },
    { 28, 32.85, 33.97, 32.69, 33.42 },
    { 31, 33.00, 33.74, 32.96, 33.57 },
    { 32, 33.51, 33.88, 32.92, 33.76 },
    { 33, 33.96, 34.95, 33.90, 34.34 },
    { 34, 34.49, 34.69, 33.01, 33.10 },
    { 35, 33.31, 33.31, 32.01, 32.32 },
    { 38, 32.79, 33.54, 32.60, 33.31 },
    { 39, 33.57, 33.85, 32.98, 33.33 },
    { 40, 33.42, 34.10, 33.34, 33.63 },
    { 41, 33.86, 33.90, 32.26, 32.77 },
    { 42, 32.93, 33.21, 31.74, 32.46 },
    { 45, 32.55, 33.11, 31.88, 32.01 },
    { 46, 32.25, 32.69, 31.82, 32.62 },
    { 47, 33.10, 33.47, 32.90, 33.04 },
    { 48, 33.04, 33.35, 29.92, 31.50 },
    { 49, 31.20, 32.32, 30.90, 32.30 },
    { 52, 32.60, 32.62, 31.36, 31.40 },
    { 53, 31.68, 31.90, 31.00, 31.25 },
    { 54, 31.45, 31.49, 30.43, 30.94 },
    { 55, 30.75, 31.23, 30.10, 30.35 },
    { 56, 30.56, 31.09, 30.26, 30.66 },
    { 59, 31.15, 31.55, 30.74, 31.34 },
    { 60, 31.40, 32.06, 31.24, 31.75 },
    { 61, 31.49, 32.25, 31.42, 31.95 },
    { 62, 31.75, 32.29, 31.57, 31.91 },
    { 63, 32.01, 33.10, 32.01, 32.99 },
    { 66, 32.97, 33.20, 32.73, 33.01 },
    { 67, 32.90, 32.99, 32.25, 32.76 },
    { 68, 32.83, 33.26, 32.58, 33.12 },
    { 69, 32.88, 33.56, 32.88, 33.19 },
    { 70, 33.20, 33.60, 33.03, 33.53 },
    { 73, 33.65, 33.83, 33.31, 33.33 },
    { 74, 33.60, 34.26, 33.51, 34.11 },
    { 75, 34.49, 34.60, 33.93, 34.35 },
    { 76, 34.35, 34.78, 34.17, 34.64 },
    { 77, 34.60, 34.89, 34.26, 34.38 },
    { 80, 34.19, 34.56, 33.92, 34.40 },
    { 81, 34.49, 34.74, 34.10, 34.45 },
    { 82, 34.55, 34.65, 33.78, 34.49 },
    { 83, 34.63, 35.19, 34.50, 35.01 },
    { 84, 34.85, 35.34, 34.78, 34.98 },
    { 87, 35.27, 35.53, 34.85, 34.95 },
    { 88, 35.28, 35.35, 34.34, 34.56 },
    { 89, 34.73, 34.99, 34.43, 34.85 },
    { 90, 35.08, 35.49, 35.06, 35.40 },
    { 95, 35.52, 35.84, 35.26, 35.51 },
    { 96, 35.60, 35.85, 35.28, 35.42 },
    { 97, 35.29, 35.36, 34.79, 35.18 },
    { 98, 35.50, 35.69, 35.10, 35.36 },
    { 101, 35.68, 35.74, 35.10, 35.41 },
    { 102, 35.43, 36.05, 34.98, 36.00 },
    { 103, 36.25, 36.74, 36.00, 36.67 },
    { 104, 36.80, 36.90, 36.19, 36.72 },
    { 105, 36.74, 37.38, 36.30, 36.54 },
    { 108, 36.49, 36.76, 36.12, 36.31 },
    { 109, 38.80, 39.24, 38.48, 39.00 },
    { 110, 39.06, 39.30, 38.38, 38.45 },
    { 111, 38.46, 38.96, 37.72, 37.85 },
    { 112, 38.21, 39.11, 37.81, 38.87 },
    { 115, 39.26, 39.52, 38.75, 39.47 },
    { 116, 39.26, 39.90, 37.93, 37.93 },
    { 117, 37.92, 38.18, 36.82, 37.76 },
    { 118, 37.85, 39.00, 37.68, 38.79 },
    { 119, 38.80, 39.30, 38.37, 38.81 },
    { 122, 38.31, 38.83, 38.08, 38.62 },
    { 123, 38.87, 38.88, 37.00, 37.37 },
    { 124, 37.72, 37.72, 36.60, 37.02 },
    { 125, 36.84, 37.74, 36.63, 36.93 },
    { 126, 36.25, 36.96, 35.30, 35.85 },
    { 129, 36.75, 38.01, 36.63, 38.01 },
    { 130, 37.33, 38.74, 37.17, 38.65 },
    { 131, 38.35, 40.17, 38.07, 39.79 },
    { 132, 40.20, 41.54, 40.02, 41.37 },
    { 133, 41.00, 41.38, 40.15, 40.58 },
    { 136, 40.21, 41.54, 40.01, 41.08 },
    { 137, 41.32, 41.92, 40.88, 41.92 },
    { 138, 41.40, 41.85, 39.85, 40.20 },
    { 139, 40.60, 40.64, 37.37, 38.40 },
    { 140, 38.15, 38.95, 37.11, 38.83 },
    { 143, 39.00, 39.13, 37.38, 38.49 },
    { 144, 37.26, 37.56, 36.67, 37.19 },
    { 145, 37.50, 38.87, 37.50, 38.24 },
    { 146, 38.60, 40.18, 38.60, 39.94 },
    { 147, 40.55, 40.67, 39.95, 40.30 },
    { 150, 40.35, 41.08, 40.31, 41.00 },
    { 151, 40.75, 41.43, 40.02, 40.99 },
    { 152, 40.50, 40.99, 39.97, 40.78 },
    { 153, 41.75, 41.99, 41.12, 41.26 },
    { 154, 41.55, 41.66, 39.87, 40.38 },
    { 157, 39.65, 40.61, 39.54, 40.08 },
    { 158, 40.15, 40.41, 39.47, 40.24 },
    { 159, 40.60, 42.05, 40.38, 41.94 },
    { 160, 41.72, 43.74, 41.64, 43.26 },
    { 161, 43.44, 43.88, 42.14, 42.74 },
    { 164, 43.20, 43.36, 42.38, 42.52 },
    { 165, 42.00, 42.79, 41.72, 42.33 },
    { 166, 42.46, 42.60, 40.78, 41.14 },
    { 167, 41.00, 42.44, 40.94, 42.31 },
    { 168, 42.47, 43.39, 42.35, 43.12 },
    { 171, 44.07, 44.79, 44.03, 44.50 },
    { 172, 44.15, 44.49, 43.74, 44.47 },
    { 173, 43.92, 44.65, 43.78, 43.95 },
    { 174, 44.49, 44.69, 43.25, 43.42 },
    { 175, 42.98, 42.98, 41.74, 42.01 },
    { 178, 42.14, 43.36, 41.86, 43.24 },
    { 179, 42.50, 42.72, 41.07, 41.28 },
    { 180, 41.62, 42.49, 41.32, 41.92 },
    { 181, 41.47, 41.82, 40.17, 40.42 },
    { 182, 40.90, 41.49, 40.17, 40.28 },
    { 185, 40.56, 40.85, 39.95, 39.95 },
    { 186, 40.15, 41.95, 40.15, 41.21 },
    { 187, 41.00, 41.99, 40.56, 41.94 },
    { 188, 42.01, 42.33, 41.15, 41.58 },
    { 189, 41.80, 41.97, 41.39, 41.63 },
    { 192, 41.80, 41.95, 41.38, 41.56 },
    { 193, 41.40, 43.81, 41.40, 43.81 },
    { 194, 44.03, 44.48, 43.06, 43.54 },
    { 195, 43.47, 44.35, 42.82, 43.28 },
    { 196, 43.39, 44.70, 42.94, 43.05 },
    { 199, 43.04, 43.22, 42.20, 42.63 },
    { 200, 42.65, 42.88, 41.30, 41.42 },
    { 201, 41.63, 42.05, 40.72, 40.96 },
    { 202, 40.79, 42.51, 40.76, 42.26 },
    { 203, 42.03, 42.83, 41.74, 42.49 },
    { 206, 42.74, 43.17, 42.36, 43.15 },
    { 207, 43.18, 43.38, 40.81, 41.34 },
    { 208, 41.90, 41.97, 41.27, 41.47 },
    { 209, 41.60, 42.11, 41.13, 41.44 },
    { 210, 41.31, 41.67, 40.79, 41.38 },
    { 213, 41.14, 41.58, 40.64, 41.40 },
    { 214, 41.33, 42.10, 41.33, 42.00 },
    { 215, 41.90, 42.10, 41.48, 41.68 },
    { 216, 41.63, 42.44, 41.56, 42.12 },
    { 217, 42.43, 42.75, 40.84, 40.97 },
    { 220, 41.53, 41.99, 41.10, 41.97 },
    { 221, 41.79, 41.79, 40.84, 41.22 },
    { 222, 40.85, 40.97, 39.92, 40.19 },
    { 223, 40.01, 40.40, 38.53, 39.07 },
    { 224, 39.48, 39.90, 38.89, 39.15 },
    { 227, 39.41, 40.14, 39.23, 39.96 },
    { 228, 40.26, 41.10, 40.13, 41.05 },
    { 229, 40.87, 41.10, 40.50, 40.62 },
    { 230, 40.97, 41.22, 39.65, 39.88 },
    { 231, 39.71, 39.95, 39.10, 39.23 },
    { 234, 39.18, 39.53, 38.90, 39.15 },
    { 235, 38.94, 39.05, 38.17, 38.66 },
    { 236, 38.46, 38.82, 37.64, 38.29 },
    { 237, 38.50, 38.63, 37.85, 38.08 },
    { 238, 37.99, 38.40, 37.60, 38.40 },
    { 241, 38.55, 38.83, 37.77, 38.12 },
    { 242, 37.50, 38.36, 37.03, 38.36 },
    { 243, 38.50, 40.49, 38.30, 40.46 },
    { 244, 40.29, 41.42, 40.22, 41.00 },
    { 245, 41.06, 42.24, 40.94, 41.65 },
    { 248, 41.72, 41.86, 41.08, 41.25 },
    { 249, 40.99, 41.36, 40.65, 41.35 },
    { 250, 41.25, 42.06, 41.12, 42.00 },
    { 251, 41.99, 43.15, 41.79, 43.08 },
    { 252, 43.00, 44.02, 42.86, 43.78 },
    { 255, 44.31, 44.65, 43.66, 43.67 },
    { 256, 43.57, 44.20, 43.54, 44.15 },
    { 257, 44.09, 44.49, 43.94, 44.12 },
    { 258, 43.70, 44.22, 43.62, 44.06 },
    { 259, 44.30, 44.74, 43.62, 44.47 },
    { 262, 44.47, 45.50, 44.35, 45.50 },
    { 263, 45.38, 45.90, 45.26, 45.63 },
    { 264, 45.01, 45.19, 44.23, 44.85 },
    { 265, 44.83, 45.13, 43.78, 44.35 },
    { 266, 44.08, 46.17, 44.00, 46.13 },
    { 269, 46.12, 46.69, 45.85, 46.41 },
    { 270, 46.25, 46.64, 45.55, 46.23 },
    { 271, 46.38, 46.85, 46.01, 46.28 },
    { 272, 45.98, 47.59, 45.88, 46.46 },
    { 273, 46.32, 46.86, 45.11, 45.51 },
    { 276, 45.19, 45.19, 43.66, 43.78 },
    { 277, 43.74, 44.88, 43.59, 44.69 },
    { 278, 45.05, 45.19, 44.03, 44.24 },
    { 279, 44.50, 45.49, 44.19, 45.37 },
    { 280, 45.20, 45.61, 44.62, 45.44 },
    { 283, 45.65, 46.30, 45.45, 45.92 },
    { 284, 46.02, 47.73, 45.66, 47.42 },
    { 285, 47.80, 48.21, 47.22, 47.94 },
    { 286, 48.00, 48.04, 47.17, 47.40 },
    { 287, 47.50, 48.13, 47.18, 47.72 },
    { 290, 47.64, 48.10, 47.24, 47.78 },
    { 291, 47.50, 47.80, 46.86, 47.04 },
    { 292, 46.80, 47.90, 46.75, 47.80 },
    { 293, 47.66, 49.12, 47.65, 49.03 },
    { 294, 48.97, 49.51, 48.63, 49.31 },
    { 297, 49.70, 50.05, 49.37, 49.70 },
    { 298, 49.22, 49.56, 48.30, 48.74 },
    { 299, 48.40, 49.10, 47.53, 47.62 },
    { 300, 48.10, 49.05, 46.74, 46.98 },
    { 301, 46.76, 47.73, 46.35, 47.43 },
    { 304, 48.10, 48.37, 47.33, 47.65 },
    { 305, 47.42, 48.62, 47.22, 48.41 },
    { 306, 48.50, 49.15, 48.10, 48.34 },
    { 307, 49.10, 50.14, 48.80, 50.00 },
    { 308, 50.08, 50.45, 48.85, 48.94 },
    { 311, 49.08, 49.10, 48.52, 48.94 },
    { 312, 48.72, 50.28, 48.65, 50.04 },
    { 313, 49.76, 49.91, 48.70, 49.17 },
    { 314, 49.56, 49.75, 48.98, 49.56 },
    { 315, 48.60, 50.23, 47.92, 49.88 },
    { 318, 49.26, 51.40, 49.26, 50.89 },
    { 319, 50.50, 50.93, 49.47, 49.47 },
    { 320, 49.46, 49.69, 48.98, 49.27 },
    { 321, 50.04, 50.96, 49.80, 50.81 },
    { 322, 50.88, 51.00, 50.18, 50.74 },
    { 325, 50.99, 51.59, 50.31, 50.56 },
    { 326, 50.16, 51.26, 49.51, 49.51 },
    { 327, 50.12, 52.04, 49.73, 52.04 },
    { 328, 52.00, 52.63, 51.52, 51.93 },
    { 329, 51.47, 51.93, 50.65, 51.54 },
    { 332, 51.93, 52.06, 49.79, 49.79 },
    { 333, 50.00, 50.93, 49.01, 49.87 },
    { 334, 50.51, 51.96, 50.09, 51.94 },
    { 335, 52.30, 53.64, 51.88, 53.49 },
    { 336, 53.00, 54.78, 52.68, 53.95 },
    { 339, 53.95, 54.18, 53.40, 53.62 },
    { 340, 53.68, 54.37, 52.61, 54.15 },
    { 341, 53.84, 54.05, 52.97, 53.26 },
    { 342, 53.85, 54.14, 52.15, 53.30 },
    { 343, 53.54, 54.93, 53.35, 54.87 },
    { 346, 55.00, 55.00, 54.42, 54.76 },
    { 347, 54.50, 54.87, 53.72, 54.11 },
    { 348, 53.81, 54.20, 53.17, 53.90 },
    { 349, 53.71, 54.58, 53.71, 54.41 },
    { 350, 54.07, 54.48, 53.57, 53.68 },
    { 353, 53.88, 54.50, 53.78, 53.88 },
    { 354, 54.02, 54.94, 53.95, 54.66 },
    { 355, 54.70, 55.05, 54.33, 54.33 },
    { 356, 54.30, 54.52, 54.04, 54.07 },
    { 360, 53.30, 53.45, 51.29, 51.57 },
    { 361, 51.67, 51.84, 51.02, 51.50 },
    { 362, 51.50, 51.62, 51.23, 51.32 },
    { 363, 51.50, 51.70, 50.61, 50.73 }
};

QVector<QwtOHLCSample> QuoteFactory::samples2010( Stock stock )
{
    const t_Data2010 *data = NULL;
    int numSamples = 0;

    switch( stock )
    {
        case BMW:
        {
            data = bmwData;
            numSamples = sizeof( bmwData ) / sizeof( t_Data2010 );
            break;
        }
        case Daimler:
        {
            data = daimlerData;
            numSamples = sizeof( daimlerData ) / sizeof( t_Data2010 );
            break;
        }
        case Porsche:
        {
            data = porscheData;
            numSamples = sizeof( porscheData ) / sizeof( t_Data2010 );
            break;
        }
        default:
            break;
    }

    QVector<QwtOHLCSample> samples;
    samples.reserve( numSamples );

    QDateTime year2010( QDate( 2010, 1, 1 ), QTime( 0, 0 ), Qt::UTC );

    for ( int i = 0; i < numSamples; i++ )
    {
        const t_Data2010 &ohlc = data[ i ];

        samples += QwtOHLCSample( 
            QwtDate::toDouble( year2010.addDays( ohlc.day ) ),
            ohlc.open, ohlc.high, ohlc.low, ohlc.close );
    }

    return samples;
}

QString QuoteFactory::title( Stock stock )
{
    switch( stock )
    {
        case BMW:
            return "BMW";
        case Daimler:
            return "Daimler";
        case Porsche:
            return "Porsche";
        default:
            break;
    }

    return "Unknown";
}
```

### `examples/stockchart/quotefactory.h`

```cpp
#ifndef _QUOTE_FACTORY_H_
#define _QUOTE_FACTORY_H_

#include <qwt_series_data.h>

class QuoteFactory
{
public:
    enum Stock
    {
        BMW,
        Daimler,
        Porsche,

        NumStocks
    };

    static QVector<QwtOHLCSample> samples2010( Stock );
    static QString title( Stock );
};

#endif
```

### `examples/sysinfo/sysinfo.cpp`

```cpp
#include <qapplication.h>
#include <qwidget.h>
#include <qfont.h>
#include <qlabel.h>
#include <qgroupbox.h>
#include <qlayout.h>
#include <qwt_thermo.h>
#include <qwt_color_map.h>

class ValueBar: public QWidget
{
public:
    ValueBar( Qt::Orientation orientation,
              const QString &text, QWidget *parent, double value = 0.0 ):
        QWidget( parent )
    {
        d_label = new QLabel( text, this );
        d_label->setFont( QFont( "Helvetica", 10 ) );

        d_thermo = new QwtThermo( this );
        d_thermo->setOrientation( orientation );
        d_thermo->setScale( 0.0, 100.0 );
        d_thermo->setValue( value );
        d_thermo->setFont( QFont( "Helvetica", 8 ) );
        d_thermo->setPipeWidth( 6 );
        d_thermo->setScaleMaxMajor( 6 );
        d_thermo->setScaleMaxMinor( 5 );
        d_thermo->setFillBrush( Qt::darkMagenta );

#if 0
        QwtLinearColorMap *colorMap =
            new QwtLinearColorMap( Qt::blue, Qt::red );

        colorMap->addColorStop( 0.2, Qt::yellow );
        colorMap->addColorStop( 0.3, Qt::cyan );
        colorMap->addColorStop( 0.4, Qt::green );
        colorMap->addColorStop( 0.5, Qt::magenta );
        colorMap->setMode( QwtLinearColorMap::FixedColors );
        d_thermo->setColorMap( colorMap );
#endif

        QVBoxLayout *layout = new QVBoxLayout( this );
        layout->setMargin( 0 );
        layout->setSpacing( 0 );

        if ( orientation == Qt::Horizontal )
        {
            d_label->setAlignment( Qt::AlignCenter );
            d_thermo->setScalePosition( QwtThermo::LeadingScale );
            layout->addWidget( d_label );
            layout->addWidget( d_thermo );
        }
        else
        {
            d_label->setAlignment( Qt::AlignRight );
            d_thermo->setScalePosition( QwtThermo::TrailingScale );
            layout->addWidget( d_thermo, 10, Qt::AlignHCenter );
            layout->addWidget( d_label, 0 );
        }
    }

    void setValue( double value )
    {
        d_thermo->setValue( value );
    }

private:
    QLabel *d_label;
    QwtThermo *d_thermo;
};

class SysInfo : public QFrame
{
public:
    SysInfo( QWidget *parent = NULL ):
        QFrame( parent )
    {
        QGroupBox *memBox = new QGroupBox( "Memory Usage", this );
        memBox->setFont( QFont( "Helvetica", 10 ) );

        QVBoxLayout *memLayout = new QVBoxLayout( memBox );
        memLayout->setMargin( 15 );
        memLayout->setSpacing( 5 );

        Qt::Orientation o = Qt::Horizontal;
        memLayout->addWidget( new ValueBar( o, "Used", memBox, 57 ) );
        memLayout->addWidget( new ValueBar( o, "Shared", memBox, 17 ) );
        memLayout->addWidget( new ValueBar( o, "Cache", memBox, 30 ) );
        memLayout->addWidget( new ValueBar( o, "Buffers", memBox, 22 ) );
        memLayout->addWidget( new ValueBar( o, "Swap Used", memBox, 57 ) );
        memLayout->addWidget( new QWidget( memBox ), 10 ); // spacer

        QGroupBox *cpuBox = new QGroupBox( "Cpu Usage", this );
        cpuBox->setFont( QFont( "Helvetica", 10 ) );

        QHBoxLayout *cpuLayout = new QHBoxLayout( cpuBox );
        cpuLayout->setMargin( 15 );
        cpuLayout->setSpacing( 5 );

        o = Qt::Vertical;
        cpuLayout->addWidget( new ValueBar( o, "User", cpuBox, 57 ) );
        cpuLayout->addWidget( new ValueBar( o, "Total", cpuBox, 73 ) );
        cpuLayout->addWidget( new ValueBar( o, "System", cpuBox, 16 ) );
        cpuLayout->addWidget( new ValueBar( o, "Idle", cpuBox, 27 ) );

        QHBoxLayout *layout = new QHBoxLayout( this );
        layout->setMargin( 10 );
        layout->addWidget( memBox, 10 );
        layout->addWidget( cpuBox, 0 );
    }
};

int main ( int argc, char **argv )
{
    QApplication a( argc, argv );

    SysInfo info;
    info.resize( info.sizeHint().expandedTo( QSize( 600, 400 ) ) );
    info.show();

    int rv = a.exec();
    return rv;
}
```

### `examples/tvplot/main.cpp`

```cpp
#include <qapplication.h>
#include <qmainwindow.h>
#include <qtoolbar.h>
#include <qtoolbutton.h>
#include <qcombobox.h>
#include "tvplot.h"

class MainWindow: public QMainWindow
{
public:
    MainWindow( QWidget * = NULL );

private:
    TVPlot *d_plot;
};

MainWindow::MainWindow( QWidget *parent ):
    QMainWindow( parent )
{
    d_plot = new TVPlot( this );
    setCentralWidget( d_plot );

    QToolBar *toolBar = new QToolBar( this );

    QComboBox *typeBox = new QComboBox( toolBar );
    typeBox->addItem( "Outline" );
    typeBox->addItem( "Columns" );
    typeBox->addItem( "Lines" );
    typeBox->addItem( "Column Symbol" );
    typeBox->setCurrentIndex( typeBox->count() - 1 );
    typeBox->setSizePolicy( QSizePolicy::Fixed, QSizePolicy::Fixed );

    QToolButton *btnExport = new QToolButton( toolBar );
    btnExport->setText( "Export" );
    btnExport->setToolButtonStyle( Qt::ToolButtonTextUnderIcon );
    connect( btnExport, SIGNAL( clicked() ), d_plot, SLOT( exportPlot() ) );

    toolBar->addWidget( typeBox );
    toolBar->addWidget( btnExport );
    addToolBar( toolBar );

    d_plot->setMode( typeBox->currentIndex() );
    connect( typeBox, SIGNAL( currentIndexChanged( int ) ),
             d_plot, SLOT( setMode( int ) ) );
}

int main( int argc, char **argv )
{
    QApplication a( argc, argv );

    MainWindow mainWindow;

    mainWindow.resize( 600, 400 );
    mainWindow.show();

    return a.exec();
}
```

### `examples/tvplot/tvplot.cpp`

```cpp
#include "tvplot.h"
#include <qwt_plot_layout.h>
#include <qwt_plot_canvas.h>
#include <qwt_plot_renderer.h>
#include <qwt_legend.h>
#include <qwt_legend_label.h>
#include <qwt_plot_grid.h>
#include <qwt_plot_histogram.h>
#include <qwt_column_symbol.h>
#include <qwt_series_data.h>
#include <qpen.h>
#include <stdlib.h>

class Histogram: public QwtPlotHistogram
{
public:
    Histogram( const QString &, const QColor & );

    void setColor( const QColor & );
    void setValues( uint numValues, const double * );
};

Histogram::Histogram( const QString &title, const QColor &symbolColor ):
    QwtPlotHistogram( title )
{
    setStyle( QwtPlotHistogram::Columns );

    setColor( symbolColor );
}

void Histogram::setColor( const QColor &color )
{
    QColor c = color;
    c.setAlpha( 180 );
    setBrush( QBrush( c ) );
}

void Histogram::setValues( uint numValues, const double *values )
{
    QVector<QwtIntervalSample> samples( numValues );
    for ( uint i = 0; i < numValues; i++ )
    {
        QwtInterval interval( double( i ), i + 1.0 );
        interval.setBorderFlags( QwtInterval::ExcludeMaximum );

        samples[i] = QwtIntervalSample( values[i], interval );
    }

    setData( new QwtIntervalSeriesData( samples ) );
}

TVPlot::TVPlot( QWidget *parent ):
    QwtPlot( parent )
{
    setTitle( "Watching TV during a weekend" );

    QwtPlotCanvas *canvas = new QwtPlotCanvas();
    canvas->setPalette( Qt::gray );
    canvas->setBorderRadius( 10 );
    setCanvas( canvas );

    plotLayout()->setAlignCanvasToScales( true );

    setAxisTitle( QwtPlot::yLeft, "Number of People" );
    setAxisTitle( QwtPlot::xBottom, "Number of Hours" );

    QwtLegend *legend = new QwtLegend;
    legend->setDefaultItemMode( QwtLegendData::Checkable );
    insertLegend( legend, QwtPlot::RightLegend );

    populate();

    connect( legend, SIGNAL( checked( const QVariant &, bool, int ) ),
        SLOT( showItem( const QVariant &, bool ) ) );

    replot(); // creating the legend items

    QwtPlotItemList items = itemList( QwtPlotItem::Rtti_PlotHistogram );
    for ( int i = 0; i < items.size(); i++ )
    {
        if ( i == 0 )
        {
            const QVariant itemInfo = itemToInfo( items[i] );

            QwtLegendLabel *legendLabel =
                qobject_cast<QwtLegendLabel *>( legend->legendWidget( itemInfo ) );
            if ( legendLabel )
                legendLabel->setChecked( true );

            items[i]->setVisible( true );
        }
        else
        {
            items[i]->setVisible( false );
        }
    }

    setAutoReplot( true );
}

void TVPlot::populate()
{
    QwtPlotGrid *grid = new QwtPlotGrid;
    grid->enableX( false );
    grid->enableY( true );
    grid->enableXMin( false );
    grid->enableYMin( false );
    grid->setMajorPen( Qt::black, 0, Qt::DotLine );
    grid->attach( this );

    const double juneValues[] = { 7, 19, 24, 32, 10, 5, 3 };
    const double novemberValues[] = { 4, 15, 22, 34, 13, 8, 4 };

    Histogram *histogramJune = new Histogram( "Summer", Qt::red );
    histogramJune->setValues(
        sizeof( juneValues ) / sizeof( double ), juneValues );
    histogramJune->attach( this );

    Histogram *histogramNovember = new Histogram( "Winter", Qt::blue );
    histogramNovember->setValues(
        sizeof( novemberValues ) / sizeof( double ), novemberValues );
    histogramNovember->attach( this );
}

void TVPlot::exportPlot()
{
    QwtPlotRenderer renderer;
    renderer.exportTo( this, "tvplot.pdf" );
}

void TVPlot::setMode( int mode )
{
    QwtPlotItemList items = itemList( QwtPlotItem::Rtti_PlotHistogram );

    for ( int i = 0; i < items.size(); i++ )
    {
        QwtPlotHistogram *histogram = static_cast<QwtPlotHistogram *>( items[i] );
        if ( mode < 3 )
        {
            histogram->setStyle( static_cast<QwtPlotHistogram::HistogramStyle>( mode ) );
            histogram->setSymbol( NULL );

            QPen pen( Qt::black, 0 );
            if ( mode == QwtPlotHistogram::Lines )
                pen.setBrush( histogram->brush() );

            histogram->setPen( pen );
        }
        else
        {
            histogram->setStyle( QwtPlotHistogram::Columns );

            QwtColumnSymbol *symbol = new QwtColumnSymbol( QwtColumnSymbol::Box );
            symbol->setFrameStyle( QwtColumnSymbol::Raised );
            symbol->setLineWidth( 2 );
            symbol->setPalette( QPalette( histogram->brush().color() ) );

            histogram->setSymbol( symbol );
        }
    }
}

void TVPlot::showItem( const QVariant &itemInfo, bool on )
{
    QwtPlotItem *plotItem = infoToItem( itemInfo );
    if ( plotItem )
        plotItem->setVisible( on );
}
```

### `examples/tvplot/tvplot.h`

```cpp
#ifndef _TV_PLOT_H_

#include <qwt_plot.h>

class TVPlot: public QwtPlot
{
    Q_OBJECT

public:
    TVPlot( QWidget * = NULL );

public Q_SLOTS:
    void setMode( int );
    void exportPlot();

private:
    void populate();

private Q_SLOTS:
    void showItem( const QVariant &, bool on );
};

#endif
```

### `./frmmain.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmMain</class>
 <widget class="QWidget" name="frmMain">
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
    <widget class="QwtPlot" name="frame">
     <property name="frameShape">
      <enum>QFrame::StyledPanel</enum>
     </property>
     <property name="frameShadow">
      <enum>QFrame::Raised</enum>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>QwtPlot</class>
   <extends>QFrame</extends>
   <header location="global">qwt_plot.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```
