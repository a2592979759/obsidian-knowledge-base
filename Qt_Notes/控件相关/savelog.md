---
tags:
  - Qt
  - 控件相关
  - 高质量
---
# 日志重定向输出 (`savelog`)

> 日志重定向输出

## 效果图

![[QWidgetDemo_assert/savelog.jpg]]

## 分类

- 类型: 控件相关 `control`
- 源码目录: `control/savelog`
- 标注: **高质量**

## 技术要点

**用到的 Qt 类**: QString QTextCodec QObject QWidget QListWidgetItem QFile QTimer QLabel QScopedPointer QFrame QComboBox QCheckBox QMutex QMutexLocker QTcpServer QStringList QDateTime QApplication QDATE QDATETIMS

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmsavelog.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmsavelog.h"
#include "ui_frmsavelog.h"
#include "savelog.h"
#include "qdatetime.h"
#include "qtimer.h"
#include "qdebug.h"

frmSaveLog::frmSaveLog(QWidget *parent) : QWidget(parent), ui(new Ui::frmSaveLog)
{
    ui->setupUi(this);
    this->initForm();
}

frmSaveLog::~frmSaveLog()
{
    delete ui;
}

void frmSaveLog::initForm()
{
    //启动定时器追加数据
    count = 0;
    timer = new QTimer(this);
    connect(timer, SIGNAL(timeout()), this, SLOT(append()));
    timer->setInterval(100);

    //添加消息类型
    QStringList types, datas;
    types << "Debug" << "Info" << "Warning" << "Critical" << "Fatal";
    datas << "1" << "2" << "4" << "8" << "16";
    ui->cboxType->addItems(types);

    //添加消息类型到列表用于勾选设置哪些类型需要重定向
    int count = types.count();
    for (int i = 0; i < count; ++i) {
        QListWidgetItem *item = new QListWidgetItem;
        item->setText(types.at(i));
        item->setData(Qt::UserRole, datas.at(i));
        item->setCheckState(Qt::Checked);
        ui->listType->addItem(item);
    }

    //添加日志文件大小下拉框
    ui->cboxSize->addItem("不启用", 0);
    ui->cboxSize->addItem("5kb", 5);
    ui->cboxSize->addItem("10kb", 10);
    ui->cboxSize->addItem("30kb", 30);
    ui->cboxSize->addItem("1mb", 1024);

    ui->cboxRow->addItem("不启用", 0);
    ui->cboxRow->addItem("100条", 100);
    ui->cboxRow->addItem("500条", 500);
    ui->cboxRow->addItem("2000条", 2000);
    ui->cboxRow->addItem("10000条", 10000);

    //设置是否开启日志上下文打印比如行号、函数等
    SaveLog::Instance()->setUseContext(false);
    //设置文件存储目录
    SaveLog::Instance()->setPath(qApp->applicationDirPath() + "/log");
}

void frmSaveLog::append(const QString &flag)
{
    if (count >= 100) {
        count = 0;
        ui->txtMain->clear();
    }

    QString str1;
    int type = ui->cboxType->currentIndex();
    if (!ui->ckSave->isChecked()) {
        if (type == 0) {
            str1 = "Debug ";
        } else if (type == 1) {
            str1 = "Infox ";
        } else if (type == 2) {
            str1 = "Warnx ";
        } else if (type == 3) {
            str1 = "Error ";
        } else if (type == 4) {
            str1 = "Fatal ";
        }
    }

    QString str2 = QDateTime::currentDateTime().toString("yyyy-MM-dd HH:mm:ss");
    QString str3 = flag.isEmpty() ? "自动插入消息" : flag;
    QString msg = QString("%1当前时间: %2 %3").arg(str1).arg(str2).arg(str3);

    //开启网络重定向换成英文方便接收解析不乱码
    //对方接收解析的工具未必是utf8
    if (ui->ckNet->isChecked()) {
        msg = QString("%1time: %2 %3").arg(str1).arg(str2).arg("(QQ: 517216493 WX: feiyangqingyun)");
    }

    count++;
    ui->txtMain->append(msg);

    //根据不同的类型打印
    //TMD转换要分两部走不然msvc的debug版本会乱码(英文也一样)
    //char *data = msg.toUtf8().data();
    QByteArray buffer = msg.toUtf8();
    const char *data = buffer.constData();
    if (type == 0) {
        qDebug(data);
    } else if (type == 1) {
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
        qInfo(data);
#endif
    } else if (type == 2) {
        qWarning(data);
    } else if (type == 3) {
        qCritical(data);
    } else if (type == 4) {
        //调用下面这个打印完会直接退出程序
        qFatal(data);
    }
}

void frmSaveLog::on_btnLog_clicked()
{
    append("手动插入消息");
}

void frmSaveLog::on_ckTimer_stateChanged(int arg1)
{
    if (arg1 == 0) {
        timer->stop();
    } else {
        timer->start();
        on_btnLog_clicked();
    }
}

void frmSaveLog::on_ckNet_stateChanged(int arg1)
{
    SaveLog::Instance()->setListenPort(ui->txtPort->text().toInt());
    SaveLog::Instance()->setToNet(ui->ckNet->isChecked());
    on_btnLog_clicked();
}

void frmSaveLog::on_ckSave_stateChanged(int arg1)
{
    if (arg1 == 0) {
        SaveLog::Instance()->stop();
    } else {
        SaveLog::Instance()->start();
        on_btnLog_clicked();
    }
}

void frmSaveLog::on_cboxSize_currentIndexChanged(int index)
{
    int size = ui->cboxSize->itemData(index).toInt();
    SaveLog::Instance()->setMaxSize(size);
    on_btnLog_clicked();
}

void frmSaveLog::on_cboxRow_currentIndexChanged(int index)
{
    int row = ui->cboxRow->itemData(index).toInt();
    SaveLog::Instance()->setMaxRow(row);
    on_btnLog_clicked();
}

void frmSaveLog::on_listType_itemPressed(QListWidgetItem *item)
{
    //切换选中行状态
    item->setCheckState(item->checkState() == Qt::Checked ? Qt::Unchecked : Qt::Checked);

    //找到所有勾选的类型进行设置
    quint8 types = 0;
    int count = ui->listType->count();
    for (int i = 0; i < count; ++i) {
        QListWidgetItem *item = ui->listType->item(i);
        if (item->checkState() == Qt::Checked) {
            types += item->data(Qt::UserRole).toInt();
        }
    }

    SaveLog::Instance()->setMsgType((MsgType)types);
}
```

### `./frmsavelog.h`

```cpp
﻿#ifndef FRMSAVELOG_H
#define FRMSAVELOG_H

#include <QWidget>
#include <QListWidgetItem>

namespace Ui {
class frmSaveLog;
}

class frmSaveLog : public QWidget
{
    Q_OBJECT

public:
    explicit frmSaveLog(QWidget *parent = 0);
    ~frmSaveLog();

private:
    Ui::frmSaveLog *ui;
    int count;
    QTimer *timer;

private slots:
    void initForm();
    void append(const QString &flag = QString());

private slots:
    void on_btnLog_clicked();
    void on_ckTimer_stateChanged(int arg1);
    void on_ckNet_stateChanged(int arg1);
    void on_ckSave_stateChanged(int arg1);

    void on_cboxSize_currentIndexChanged(int index);
    void on_cboxRow_currentIndexChanged(int index);
    void on_listType_itemPressed(QListWidgetItem *item);
};

#endif // FRMSAVELOG_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmsavelog.h"
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

    frmSaveLog w;
    w.setWindowTitle("日志重定向示例 V2022 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./savelog.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "savelog.h"
#include "qmutex.h"
#include "qdir.h"
#include "qfile.h"
#include "qtcpsocket.h"
#include "qtcpserver.h"
#include "qdatetime.h"
#include "qapplication.h"
#include "qtimer.h"
#include "qtextstream.h"
#include "qstringlist.h"

#define QDATE qPrintable(QDate::currentDate().toString("yyyy-MM-dd"))
#define QDATETIMS qPrintable(QDateTime::currentDateTime().toString("yyyy-MM-dd-HH-mm-ss"))

//日志重定向
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
void Log(QtMsgType type, const QMessageLogContext &context, const QString &msg)
#else
void Log(QtMsgType type, const char *msg)
#endif
{
    //加锁,防止多线程中qdebug太频繁导致崩溃
    static QMutex mutex;
    QMutexLocker locker(&mutex);
    QString content;

    //这里可以根据不同的类型加上不同的头部用于区分
    int msgType = SaveLog::Instance()->getMsgType();
    switch (type) {
        case QtDebugMsg:
            if ((msgType & MsgType_Debug) == MsgType_Debug) {
                content = QString("Debug %1").arg(msg);
            }
            break;
#if (QT_VERSION >= QT_VERSION_CHECK(5,5,0))
        case QtInfoMsg:
            if ((msgType & MsgType_Info) == MsgType_Info) {
                content = QString("Infox %1").arg(msg);
            }
            break;
#endif
        case QtWarningMsg:
            if ((msgType & MsgType_Warning) == MsgType_Warning) {
                content = QString("Warnx %1").arg(msg);
            }
            break;
        case QtCriticalMsg:
            if ((msgType & MsgType_Critical) == MsgType_Critical) {
                content = QString("Error %1").arg(msg);
            }
            break;
        case QtFatalMsg:
            if ((msgType & MsgType_Fatal) == MsgType_Fatal) {
                content = QString("Fatal %1").arg(msg);
            }
            break;
    }

    //没有内容则返回
    if (content.isEmpty()) {
        return;
    }

    //加上打印代码所在代码文件、行号、函数名
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    if (SaveLog::Instance()->getUseContext()) {
        int line = context.line;
        QString file = context.file;
        QString function = context.function;
        if (line > 0) {
            content = QString("行号: %1  文件: %2  函数: %3\n%4").arg(line).arg(file).arg(function).arg(content);
        }
    }
#endif

    //还可以将数据转成html内容分颜色区分
    //将内容传给函数进行处理
    SaveLog::Instance()->save(content);
}

QScopedPointer<SaveLog> SaveLog::self;
SaveLog *SaveLog::Instance()
{
    if (self.isNull()) {
        static QMutex mutex;
        QMutexLocker locker(&mutex);
        if (self.isNull()) {
            self.reset(new SaveLog);
        }
    }

    return self.data();
}

SaveLog::SaveLog(QObject *parent) : QObject(parent)
{
    //必须用信号槽形式,不然提示 QSocketNotifier: Socket notifiers cannot be enabled or disabled from another thread
    //估计日志钩子可能单独开了线程
    connect(this, SIGNAL(send(QString)), SendLog::Instance(), SLOT(send(QString)));

    isRun = false;
    maxRow = currentRow = 0;
    maxSize = 128;
    toNet = false;
    useContext = false;

    //全局的文件对象,在需要的时候打开而不是每次添加日志都打开
    file = new QFile(this);
    //默认取应用程序根目录
    path = qApp->applicationDirPath();
    //默认取应用程序可执行文件名称
    QString str = qApp->applicationFilePath();
    QStringList list = str.split("/");
    name = list.at(list.count() - 1).split(".").at(0);
    fileName = "";

    //默认所有类型都输出
    msgType = MsgType(MsgType_Debug | MsgType_Info | MsgType_Warning | MsgType_Critical | MsgType_Fatal);
}

SaveLog::~SaveLog()
{
    this->stop();
}

void SaveLog::openFile(const QString &fileName)
{
    //当文件名改变时才新建和打开文件而不是每次都打开文件(效率极低)或者一开始打开文件
    if (this->fileName != fileName) {
        this->fileName = fileName;
        //先关闭之前的
        if (file->isOpen()) {
            file->close();
        }
        //重新设置新的日志文件
        file->setFileName(fileName);
        //以 Append 追加的形式打开
        file->open(QIODevice::WriteOnly | QIODevice::Append | QFile::Text);
    }
}

bool SaveLog::getUseContext()
{
    return this->useContext;
}

MsgType SaveLog::getMsgType()
{
    return this->msgType;
}

//安装日志钩子,输出调试信息到文件,便于调试
void SaveLog::start()
{
    if (isRun) {
        return;
    }

    isRun = true;
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    qInstallMessageHandler(Log);
#else
    qInstallMsgHandler(Log);
#endif
}

//卸载日志钩子
void SaveLog::stop()
{
    if (!isRun) {
        return;
    }

    isRun = false;
    this->clear();
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    qInstallMessageHandler(0);
#else
    qInstallMsgHandler(0);
#endif
}

void SaveLog::clear()
{
    currentRow = 0;
    fileName.clear();
    if (file->isOpen()) {
        file->close();
    }
}

void SaveLog::clearContent()
{
    currentRow = 0;
    if (file->isOpen()) {
        file->resize(0);
    }
}

void SaveLog::save(const QString &content)
{
    //如果重定向输出到网络则通过网络发出去,否则输出到日志文件
    if (toNet) {
        Q_EMIT send(content);
    } else {
        //目录不存在则先新建目录
        QDir dir(path);
        if (!dir.exists()) {
            dir.mkpath(path);
        }

        //日志存储规则有多种策略 优先级 行数>大小>日期
        //1: 设置了最大行数限制则按照行数限制来
        //2: 设置了大小则按照大小来控制日志文件
        //3: 都没有设置都存储到日期命名的文件,只有当日期变化了才会切换到新的日志文件
        bool needOpen = false;
        if (maxRow > 0) {
            currentRow++;
            if (fileName.isEmpty()) {
                needOpen = true;
            } else if (currentRow >= maxRow) {
                needOpen = true;
            }
        } else if (maxSize > 0) {
            //1MB=1024*1024 经过大量测试 QFile().size() 方法速度非常快
            //首次需要重新打开文件以及超过大小需要重新打开文件
            if (fileName.isEmpty()) {
                needOpen = true;
            } else if (file->size() > (maxSize * 1024)) {
                needOpen = true;
            }
        } else {
            //日期改变了才会触发
            QString fileName = QString("%1/%2_log_%3.txt").arg(path).arg(name).arg(QDATE);
            openFile(fileName);
        }

        if ((maxRow > 0 || maxSize > 0) && needOpen) {
            currentRow = 0;
            QString fileName = QString("%1/%2_log_%3.txt").arg(path).arg(name).arg(QDATETIMS);
            openFile(fileName);
        }

        //用文本流的输出速度更快
        QTextStream stream(file);
        stream << content << "\n";
    }
}

void SaveLog::setMaxRow(int maxRow)
{
    //这里可以限定最大最小值
    if (maxRow >= 0) {
        this->maxRow = maxRow;
        this->clear();
    }
}

void SaveLog::setMaxSize(int maxSize)
{
    //这里可以限定最大最小值
    if (maxSize >= 0) {
        this->maxSize = maxSize;
        this->clear();
    }
}

void SaveLog::setListenPort(int listenPort)
{
    SendLog::Instance()->setListenPort(listenPort);
}

void SaveLog::setToNet(bool toNet)
{
    this->toNet = toNet;
    if (toNet) {
        SendLog::Instance()->start();
    } else {
        SendLog::Instance()->stop();
    }
}

void SaveLog::setUseContext(bool useContext)
{
    this->useContext = useContext;
}

void SaveLog::setPath(const QString &path)
{
    this->path = path;
}

void SaveLog::setName(const QString &name)
{
    this->name = name;
}

void SaveLog::setMsgType(const MsgType &msgType)
{
    this->msgType = msgType;
}


//网络发送日志数据类
QScopedPointer<SendLog> SendLog::self;
SendLog *SendLog::Instance()
{
    if (self.isNull()) {
        static QMutex mutex;
        QMutexLocker locker(&mutex);
        if (self.isNull()) {
            self.reset(new SendLog);
        }
    }

    return self.data();
}

SendLog::SendLog(QObject *parent) : QObject(parent)
{
    listenPort = 6000;
    socket = NULL;

    //实例化网络通信服务器对象
    server = new QTcpServer(this);
    connect(server, SIGNAL(newConnection()), this, SLOT(newConnection()));
}

SendLog::~SendLog()
{
    if (socket != NULL) {
        socket->disconnectFromHost();
    }

    server->close();
}

void SendLog::newConnection()
{
    //限定就一个连接
    while (server->hasPendingConnections()) {
        socket = server->nextPendingConnection();
    }
}

void SendLog::setListenPort(int listenPort)
{
    this->listenPort = listenPort;
}

void SendLog::start()
{
    //启动端口监听
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
    server->listen(QHostAddress::AnyIPv4, listenPort);
#else
    server->listen(QHostAddress::Any, listenPort);
#endif
}

void SendLog::stop()
{
    if (socket != NULL) {
        socket->disconnectFromHost();
        socket = NULL;
    }

    server->close();
}

void SendLog::send(const QString &content)
{
    if (socket != NULL && socket->isOpen()) {
        socket->write(content.toUtf8());
        //socket->flush();
    }
}
```

### `./savelog.h`

```cpp
﻿#ifndef SAVELOG_H
#define SAVELOG_H

/**
 * 日志重定向输出 作者:feiyangqingyun(QQ:517216493) 2016-12-16
 * 1. 支持动态启动和停止。
 * 2. 支持日志存储的目录。
 * 3. 支持网络发出打印日志。
 * 4. 支持输出日志上下文信息比如所在代码文件、行号、函数名等。
 * 5. 支持设置日志文件大小限制，超过则自动分文件，默认128kb。
 * 6. 支持按照日志行数自动分文件，和日志大小条件互斥。
 * 7. 可选按照日期时间区分文件名存储日志。
 * 8. 日志文件命名规则优先级：行数》大小》日期。
 * 9. 自动加锁支持多线程。
 * 10. 可以分别控制哪些类型的日志需要重定向输出。
 * 11. 支持Qt4+Qt5+Qt6，开箱即用。
 * 12. 使用方式最简单，调用函数start()启动服务，stop()停止服务。
 */

#include <QObject>

class QFile;
class QTcpSocket;
class QTcpServer;

//消息类型
enum MsgType {
    MsgType_Debug = 0x0001,
    MsgType_Info = 0x0002,
    MsgType_Warning = 0x0004,
    MsgType_Critical = 0x0008,
    MsgType_Fatal = 0x0010,
};

#ifdef quc
class Q_DECL_EXPORT SaveLog : public QObject
#else
class SaveLog : public QObject
#endif

{
    Q_OBJECT
public:
    static SaveLog *Instance();
    explicit SaveLog(QObject *parent = 0);
    ~SaveLog();

private:
    static QScopedPointer<SaveLog> self;

    //是否在运行
    bool isRun;
    //文件最大行数 0表示不启用
    int maxRow, currentRow;
    //文件最大大小 0表示不启用 单位kb
    int maxSize;
    //是否重定向到网络
    bool toNet;
    //是否输出日志上下文
    bool useContext;

    //文件对象
    QFile *file;
    //日志文件路径
    QString path;
    //日志文件名称
    QString name;
    //日志文件完整名称
    QString fileName;
    //消息类型
    MsgType msgType;

private:
    void openFile(const QString &fileName);

public:
    bool getUseContext();
    MsgType getMsgType();

Q_SIGNALS:
    //发送内容信号
    void send(const QString &content);

public Q_SLOTS:
    //启动日志服务
    void start();
    //暂停日志服务
    void stop();

    //清空状态
    void clear();
    //清空内容
    void clearContent();
    //保存日志
    void save(const QString &content);

    //设置日志文件最大行数
    void setMaxRow(int maxRow);
    //设置日志文件最大大小 单位kb
    void setMaxSize(int maxSize);

    //设置监听端口
    void setListenPort(int listenPort);
    //设置是否重定向到网络
    void setToNet(bool toNet);
    //设置是否输出日志上下文
    void setUseContext(bool useContext);

    //设置日志文件存放路径
    void setPath(const QString &path);
    //设置日志文件名称
    void setName(const QString &name);
    //设置消息类型
    void setMsgType(const MsgType &msgType);
};

#ifdef quc
class Q_DECL_EXPORT SendLog : public QObject
#else
class SendLog : public QObject
#endif

{
    Q_OBJECT
public:
    static SendLog *Instance();
    explicit SendLog(QObject *parent = 0);
    ~SendLog();

private:
    static QScopedPointer<SendLog> self;

    //监听端口
    int listenPort;
    //网络通信对象
    QTcpSocket *socket;
    //网络监听服务器
    QTcpServer *server;

private slots:
    //新连接到来
    void newConnection();

public Q_SLOTS:
    //设置监听端口
    void setListenPort(int listenPort);

    //启动和停止服务
    void start();
    void stop();

    //发送日志
    void send(const QString &content);
};

#endif // SAVELOG_H
```

### `./frmsavelog.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmSaveLog</class>
 <widget class="QWidget" name="frmSaveLog">
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
  <layout class="QHBoxLayout" name="horizontalLayout">
   <item>
    <widget class="QTextEdit" name="txtMain"/>
   </item>
   <item>
    <widget class="QFrame" name="frame">
     <property name="minimumSize">
      <size>
       <width>200</width>
       <height>0</height>
      </size>
     </property>
     <property name="maximumSize">
      <size>
       <width>200</width>
       <height>16777215</height>
      </size>
     </property>
     <property name="frameShape">
      <enum>QFrame::Box</enum>
     </property>
     <property name="frameShadow">
      <enum>QFrame::Sunken</enum>
     </property>
     <layout class="QVBoxLayout" name="verticalLayout">
      <item>
       <layout class="QGridLayout" name="gridLayout">
        <item row="1" column="1">
         <widget class="QComboBox" name="cboxSize"/>
        </item>
        <item row="2" column="1">
         <widget class="QComboBox" name="cboxRow"/>
        </item>
        <item row="0" column="1">
         <widget class="QComboBox" name="cboxType">
          <property name="sizePolicy">
           <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
            <horstretch>0</horstretch>
            <verstretch>0</verstretch>
           </sizepolicy>
          </property>
         </widget>
        </item>
        <item row="1" column="0">
         <widget class="QLabel" name="labSize">
          <property name="text">
           <string>文件大小</string>
          </property>
         </widget>
        </item>
        <item row="2" column="0">
         <widget class="QLabel" name="labRow">
          <property name="text">
           <string>文件行数</string>
          </property>
         </widget>
        </item>
        <item row="0" column="0">
         <widget class="QLabel" name="labType">
          <property name="text">
           <string>消息类型</string>
          </property>
         </widget>
        </item>
        <item row="3" column="0">
         <widget class="QLabel" name="labPort">
          <property name="text">
           <string>监听端口</string>
          </property>
         </widget>
        </item>
        <item row="3" column="1">
         <widget class="QLineEdit" name="txtPort">
          <property name="text">
           <string>6000</string>
          </property>
         </widget>
        </item>
       </layout>
      </item>
      <item>
       <widget class="QListWidget" name="listType"/>
      </item>
      <item>
       <widget class="QCheckBox" name="ckSave">
        <property name="text">
         <string>开启日志重定向</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QCheckBox" name="ckNet">
        <property name="text">
         <string>日志输出到网络</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QCheckBox" name="ckTimer">
        <property name="text">
         <string>定时器打印消息</string>
        </property>
       </widget>
      </item>
      <item>
       <widget class="QPushButton" name="btnLog">
        <property name="text">
         <string>手动插入消息</string>
        </property>
       </widget>
      </item>
     </layout>
    </widget>
   </item>
  </layout>
 </widget>
 <resources/>
 <connections/>
</ui>
```
