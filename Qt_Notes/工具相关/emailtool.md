---
tags:
  - Qt
  - 工具相关
---
# 邮件发送工具 (`emailtool`)

> 邮件发送工具

## 效果图

![[QWidgetDemo_assert/emailtool.jpg]]

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/emailtool`

## 技术要点

**用到的 Qt 类**: QString QTextCodec QStringList QLabel QWidget QMessageBox QLineEdit QThread QMutex QFileDialog QPushButton QComboBox QApplication QScopedPointer QObject QFile QVBoxLayout QGridLayout QCheckBox QTextBrowser

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./frmemailtool.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmemailtool.h"
#include "ui_frmemailtool.h"
#include "qfiledialog.h"
#include "qmessagebox.h"
#include "sendemailthread.h"

frmEmailTool::frmEmailTool(QWidget *parent) : QWidget(parent), ui(new Ui::frmEmailTool)
{
    ui->setupUi(this);
    this->initForm();
}

frmEmailTool::~frmEmailTool()
{
    delete ui;
}

void frmEmailTool::initForm()
{    
    ui->cboxServer->setCurrentIndex(1);
    connect(SendEmailThread::Instance(), SIGNAL(receiveEmailResult(QString)),
            this, SLOT(receiveEmailResult(QString)));
    SendEmailThread::Instance()->start();
}

void frmEmailTool::on_btnSend_clicked()
{
    if (!check()) {
        return;
    }

    SendEmailThread::Instance()->setEmailTitle(ui->txtTitle->text());
    SendEmailThread::Instance()->setSendEmailAddr(ui->txtSenderAddr->text());
    SendEmailThread::Instance()->setSendEmailPwd(ui->txtSenderPwd->text());
    SendEmailThread::Instance()->setReceiveEmailAddr(ui->txtReceiverAddr->text());

    //设置好上述配置后,以后只要调用Append方法即可发送邮件
    SendEmailThread::Instance()->append(ui->txtContent->toHtml(), ui->txtFileName->text());
}

void frmEmailTool::on_btnSelect_clicked()
{
    QFileDialog dialog(this);
    dialog.setFileMode(QFileDialog::ExistingFiles);

    if (dialog.exec()) {
        ui->txtFileName->clear();
        QStringList files = dialog.selectedFiles();
        ui->txtFileName->setText(files.join(";"));
    }
}

bool frmEmailTool::check()
{
    if (ui->txtSenderAddr->text() == "") {
        QMessageBox::critical(this, "错误", "用户名不能为空!");
        ui->txtSenderAddr->setFocus();
        return false;
    }

    if (ui->txtSenderPwd->text() == "") {
        QMessageBox::critical(this, "错误", "用户密码不能为空!");
        ui->txtSenderPwd->setFocus();
        return false;
    }

    if (ui->txtSenderAddr->text() == "") {
        QMessageBox::critical(this, "错误", "发件人不能为空!");
        ui->txtSenderAddr->setFocus();
        return false;
    }

    if (ui->txtReceiverAddr->text() == "") {
        QMessageBox::critical(this, "错误", "收件人不能为空!");
        ui->txtReceiverAddr->setFocus();
        return false;
    }

    if (ui->txtTitle->text() == "") {
        QMessageBox::critical(this, "错误", "邮件标题不能为空!");
        ui->txtTitle->setFocus();
        return false;
    }

    return true;
}

void frmEmailTool::on_cboxServer_currentIndexChanged(int index)
{
    if (index == 2) {
        ui->cboxPort->setCurrentIndex(1);
        ui->ckSSL->setChecked(true);
    } else {
        ui->cboxPort->setCurrentIndex(0);
        ui->ckSSL->setChecked(false);
    }
}

void frmEmailTool::receiveEmailResult(QString result)
{
    QMessageBox::information(this, "提示", result);
}
```

### `./frmemailtool.h`

```cpp
﻿#ifndef FRMEMAILTOOL_H
#define FRMEMAILTOOL_H

#include <QWidget>

namespace Ui
{
class frmEmailTool;
}

class frmEmailTool : public QWidget
{
	Q_OBJECT

public:
    explicit frmEmailTool(QWidget *parent = 0);
    ~frmEmailTool();

private:
    Ui::frmEmailTool *ui;

private:
	bool check();

private slots:
    void initForm();
	void receiveEmailResult(QString result);

private slots:
	void on_btnSend_clicked();
	void on_btnSelect_clicked();
	void on_cboxServer_currentIndexChanged(int index);
};

#endif // FRMEMAILTOOL_H
```

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")

#include "frmemailtool.h"
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

    frmEmailTool w;
    w.setWindowTitle("邮件发送工具 V2021 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./sendemailthread.cpp`

```cpp
﻿#include "sendemailthread.h"
#include "smtpmime.h"

#pragma execution_character_set("utf-8")
#define TIMEMS qPrintable(QTime::currentTime().toString("hh:mm:ss zzz"))

QScopedPointer<SendEmailThread> SendEmailThread::self;
SendEmailThread *SendEmailThread::Instance()
{
    if (self.isNull()) {
        static QMutex mutex;
        QMutexLocker locker(&mutex);
        if (self.isNull()) {
            self.reset(new SendEmailThread);
        }
    }

    return self.data();
}

SendEmailThread::SendEmailThread(QObject *parent) : QThread(parent)
{
    stopped = false;
    emialTitle = "邮件标题";
    sendEmailAddr = "feiyangqingyun@126.com";
    sendEmailPwd = "123456789";
    receiveEmailAddr = "feiyangqingyun@163.com;517216493@qq.com";
    contents.clear();
    fileNames.clear();
}

SendEmailThread::~SendEmailThread()
{
    this->stop();
    this->wait(1000);
}

void SendEmailThread::run()
{
    while (!stopped) {
        int count = contents.count();
        if (count > 0) {
            mutex.lock();
            QString content = contents.takeFirst();
            QString fileName = fileNames.takeFirst();
            mutex.unlock();

            QString result;
            QStringList list = sendEmailAddr.split("@");
            QString tempSMTP = list.at(1).split(".").at(0);
            int tempPort = 25;

            //QQ邮箱端口号为465,必须启用SSL协议.
            if (tempSMTP.toUpper() == "QQ") {
                tempPort = 465;
            }

            SmtpClient smtp(QString("smtp.%1.com").arg(tempSMTP), tempPort, tempPort == 25 ? SmtpClient::TcpConnection : SmtpClient::SslConnection);
            smtp.setUser(sendEmailAddr);
            smtp.setPassword(sendEmailPwd);

            //构建邮件主题,包含发件人收件人附件等.
            MimeMessage message;
            message.setSender(new EmailAddress(sendEmailAddr));

            //逐个添加收件人
            QStringList receiver = receiveEmailAddr.split(';');
            for (int i = 0; i < receiver.size(); i++) {
                message.addRecipient(new EmailAddress(receiver.at(i)));
            }

            //构建邮件标题
            message.setSubject(emialTitle);

            //构建邮件正文
            MimeHtml text;
            text.setHtml(content);
            message.addPart(&text);

            //构建附件-报警图像
            if (fileName.length() > 0) {
                QStringList attas = fileName.split(";");
                foreach (QString tempAtta, attas) {
                    QFile *file = new QFile(tempAtta);
                    if (file->exists()) {
                        message.addPart(new MimeAttachment(file));
                    }
                }
            }

            if (!smtp.connectToHost()) {
                result = "邮件服务器连接失败";
            } else {
                if (!smtp.login()) {
                    result = "邮件用户登录失败";
                } else {
                    if (!smtp.sendMail(message)) {
                        result = "邮件发送失败";
                    } else {
                        result = "邮件发送成功";
                    }
                }
            }

            smtp.quit();
            if (!result.isEmpty()) {
                emit receiveEmailResult(result);
            }

            msleep(1000);
        }

        msleep(100);
    }

    stopped = false;
}

void SendEmailThread::stop()
{
    stopped = true;
}

void SendEmailThread::setEmailTitle(const QString &emailTitle)
{
    this->emialTitle = emailTitle;
}

void SendEmailThread::setSendEmailAddr(const QString &sendEmailAddr)
{
    this->sendEmailAddr = sendEmailAddr;
}

void SendEmailThread::setSendEmailPwd(const QString &sendEmailPwd)
{
    this->sendEmailPwd = sendEmailPwd;
}

void SendEmailThread::setReceiveEmailAddr(const QString &receiveEmailAddr)
{
    this->receiveEmailAddr = receiveEmailAddr;
}

void SendEmailThread::append(const QString &content, const QString &fileName)
{
    mutex.lock();
    contents.append(content);
    fileNames.append(fileName);
    mutex.unlock();
}
```

### `./sendemailthread.h`

```cpp
﻿#ifndef SENDEMAILTHREAD_H
#define SENDEMAILTHREAD_H

#include <QThread>
#include <QMutex>
#include <QStringList>

class SendEmailThread : public QThread
{
	Q_OBJECT
public:
    static SendEmailThread *Instance();
    explicit SendEmailThread(QObject *parent = 0);
    ~SendEmailThread();

protected:
	void run();

private:
    static QScopedPointer<SendEmailThread> self;
    QMutex mutex;
	volatile bool stopped;

	QString emialTitle;         //邮件标题
	QString sendEmailAddr;      //发件人邮箱
	QString sendEmailPwd;       //发件人密码
	QString receiveEmailAddr;   //收件人邮箱,可多个中间;隔开
	QStringList contents;       //正文内容
	QStringList fileNames;      //附件

signals:
    void receiveEmailResult(const QString &result);

public slots:    
    void stop();
    void setEmailTitle(const QString &emailTitle);
    void setSendEmailAddr(const QString &sendEmailAddr);
    void setSendEmailPwd(const QString &sendEmailPwd);
    void setReceiveEmailAddr(const QString &receiveEmailAddr);
    void append(const QString &content, const QString &fileName);

};

#endif // SENDEMAILTHREAD_H
```

### `./frmemailtool.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>frmEmailTool</class>
 <widget class="QWidget" name="frmEmailTool">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>764</width>
    <height>578</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Form</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <layout class="QGridLayout" name="gridLayout">
     <item row="2" column="4">
      <widget class="QPushButton" name="btnSend">
       <property name="cursor">
        <cursorShape>PointingHandCursor</cursorShape>
       </property>
       <property name="text">
        <string>发送</string>
       </property>
      </widget>
     </item>
     <item row="1" column="3">
      <widget class="QComboBox" name="cboxPort">
       <item>
        <property name="text">
         <string>25</string>
        </property>
       </item>
       <item>
        <property name="text">
         <string>465</string>
        </property>
       </item>
       <item>
        <property name="text">
         <string>587</string>
        </property>
       </item>
      </widget>
     </item>
     <item row="1" column="4">
      <widget class="QCheckBox" name="ckSSL">
       <property name="text">
        <string>SSL</string>
       </property>
      </widget>
     </item>
     <item row="1" column="2">
      <widget class="QLabel" name="labPort">
       <property name="text">
        <string>服务端口</string>
       </property>
      </widget>
     </item>
     <item row="3" column="4">
      <widget class="QPushButton" name="btnSelect">
       <property name="cursor">
        <cursorShape>PointingHandCursor</cursorShape>
       </property>
       <property name="text">
        <string>浏览</string>
       </property>
      </widget>
     </item>
     <item row="2" column="3">
      <widget class="QLineEdit" name="txtReceiverAddr">
       <property name="text">
        <string>feiyangqingyun@163.com;517216493@qq.com</string>
       </property>
      </widget>
     </item>
     <item row="3" column="1" colspan="3">
      <widget class="QLineEdit" name="txtFileName"/>
     </item>
     <item row="2" column="0">
      <widget class="QLabel" name="labTitle">
       <property name="text">
        <string>邮件标题</string>
       </property>
      </widget>
     </item>
     <item row="2" column="1">
      <widget class="QLineEdit" name="txtTitle">
       <property name="text">
        <string>测试邮件</string>
       </property>
      </widget>
     </item>
     <item row="1" column="1">
      <widget class="QComboBox" name="cboxServer">
       <item>
        <property name="text">
         <string>smtp.163.com</string>
        </property>
       </item>
       <item>
        <property name="text">
         <string>smtp.126.com</string>
        </property>
       </item>
       <item>
        <property name="text">
         <string>smtp.qq.com</string>
        </property>
       </item>
       <item>
        <property name="text">
         <string>smt.sina.com</string>
        </property>
       </item>
       <item>
        <property name="text">
         <string>smtp.sohu.com</string>
        </property>
       </item>
       <item>
        <property name="text">
         <string>smtp.139.com</string>
        </property>
       </item>
       <item>
        <property name="text">
         <string>smtp.189.com</string>
        </property>
       </item>
      </widget>
     </item>
     <item row="2" column="2">
      <widget class="QLabel" name="labReceiverAddr">
       <property name="text">
        <string>收件地址</string>
       </property>
      </widget>
     </item>
     <item row="3" column="0">
      <widget class="QLabel" name="labFileName">
       <property name="text">
        <string>选择附件</string>
       </property>
      </widget>
     </item>
     <item row="1" column="0">
      <widget class="QLabel" name="labServer">
       <property name="text">
        <string>服务地址</string>
       </property>
      </widget>
     </item>
     <item row="0" column="0">
      <widget class="QLabel" name="labSenderAddr">
       <property name="text">
        <string>用户名称</string>
       </property>
      </widget>
     </item>
     <item row="0" column="1">
      <widget class="QLineEdit" name="txtSenderAddr">
       <property name="text">
        <string>feiyangqingyun@126.com</string>
       </property>
      </widget>
     </item>
     <item row="0" column="2">
      <widget class="QLabel" name="labSenderPwd">
       <property name="text">
        <string>用户密码</string>
       </property>
      </widget>
     </item>
     <item row="0" column="3">
      <widget class="QLineEdit" name="txtSenderPwd">
       <property name="text">
        <string/>
       </property>
       <property name="echoMode">
        <enum>QLineEdit::Password</enum>
       </property>
      </widget>
     </item>
    </layout>
   </item>
   <item>
    <widget class="QTextBrowser" name="txtContent">
     <property name="readOnly">
      <bool>false</bool>
     </property>
     <property name="html">
      <string>&lt;!DOCTYPE HTML PUBLIC &quot;-//W3C//DTD HTML 4.0//EN&quot; &quot;http://www.w3.org/TR/REC-html40/strict.dtd&quot;&gt;
&lt;html&gt;&lt;head&gt;&lt;meta name=&quot;qrichtext&quot; content=&quot;1&quot; /&gt;&lt;style type=&quot;text/css&quot;&gt;
p, li { white-space: pre-wrap; }
&lt;/style&gt;&lt;/head&gt;&lt;body style=&quot; font-family:'SimSun'; font-size:9.07563pt; font-weight:400; font-style:normal;&quot;&gt;
&lt;p style=&quot; margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;&quot;&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;1&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;、最短的爱情哲理小说：“&lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;你应该嫁给我啦？&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;” “ &lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;不&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;” &lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;于是他俩又继续幸福地生活在一起&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;！&lt;br /&gt;&lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;2&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;、近年来中国最精彩的写实小说，全文八个字：&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:16pt; font-weight:600; font-style:italic; color:#ff007f;&quot;&gt;此地钱多人傻速来&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;  据说是发自杭州市宝石山下一出租房的汇款单上的简短附言，是该按摩女给家乡妹妹汇 款时随手涂鸦的，令无数专业作家汗颜！&lt;br /&gt;&lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;3&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;、最短的幽默小说 《夜》 男：疼么？女：恩！男：算了？女：别！&lt;/span&gt;&lt;/p&gt;
&lt;p style=&quot; margin-top:0px; margin-bottom:0px; margin-left:0px; margin-right:0px; -qt-block-indent:0; text-indent:0px;&quot;&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;4&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;、最短的荒诞小说：有一个面包走在街上，它觉得自己很饿，就把自己吃了。&lt;br /&gt;&lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;5&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;、世界最短言情小说：他死的那天，孩子出生了。&lt;br /&gt;&lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;6&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;、世界最短武侠小说：&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt; font-weight:600; color:#00aa00;&quot;&gt;高手被豆腐砸死了&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;。&lt;br /&gt;&lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;7&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;、世界最短科幻小说：最后一个地球人坐在家里，突然响起了敲门声。&lt;br /&gt;&lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;8&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;、世界最短悬疑小说：生，死，生。&lt;br /&gt;&lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;9&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;、世界最短推理小说：他死了，一定曾经活过。 &lt;br /&gt;1&lt;/span&gt;&lt;span style=&quot; font-family:'宋体'; font-size:11pt;&quot;&gt;0&lt;/span&gt;&lt;span style=&quot; font-family:'Times New Roman'; font-size:11pt;&quot;&gt;、世界最短恐怖小说：惊醒，身边躺着自己的尸体。&lt;/span&gt;&lt;/p&gt;&lt;/body&gt;&lt;/html&gt;</string>
     </property>
    </widget>
   </item>
  </layout>
 </widget>
 <tabstops>
  <tabstop>txtSenderAddr</tabstop>
  <tabstop>txtSenderPwd</tabstop>
  <tabstop>cboxServer</tabstop>
  <tabstop>cboxPort</tabstop>
  <tabstop>ckSSL</tabstop>
  <tabstop>txtTitle</tabstop>
  <tabstop>txtReceiverAddr</tabstop>
  <tabstop>btnSend</tabstop>
  <tabstop>txtFileName</tabstop>
  <tabstop>btnSelect</tabstop>
  <tabstop>txtContent</tabstop>
 </tabstops>
 <resources/>
 <connections/>
</ui>
```
