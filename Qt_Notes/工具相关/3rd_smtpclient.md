---
tags:
  - Qt
  - 工具相关
---
# SMTP 邮件库 (`3rd_smtpclient`)

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/3rd_smtpclient`

## 技术要点

**用到的 Qt 类**: QString QByteArray QList QObject QFile QSslSocket QTcpSocket QFileInfo QCryptographicHash QIODevice QDateTime QTime QUIT

**特性/模块**: Q_OBJECT

## 完整源码

### `./emailaddress.cpp`

```cpp
﻿#include "emailaddress.h"

EmailAddress::EmailAddress(const QString &address, const QString &name)
{
	this->address = address;
	this->name = name;
}

EmailAddress::~EmailAddress()
{
}

void EmailAddress::setName(const QString &name)
{
	this->name = name;
}

void EmailAddress::setAddress(const QString &address)
{
	this->address = address;
}

const QString &EmailAddress::getName() const
{
	return name;
}

const QString &EmailAddress::getAddress() const
{
	return address;
}
```

### `./emailaddress.h`

```cpp
﻿#ifndef EMAILADDRESS_H
#define EMAILADDRESS_H

#include <QObject>

class EmailAddress : public QObject
{
	Q_OBJECT
public:
	EmailAddress(const QString &address, const QString &name = "");
	~EmailAddress();

	void setName(const QString &name);
	void setAddress(const QString &address);

	const QString &getName() const;
	const QString &getAddress() const;

private:
	QString name;
	QString address;

};

#endif // EMAILADDRESS_H
```

### `./mimeattachment.cpp`

```cpp
﻿#include "mimeattachment.h"
#include <QFileInfo>

MimeAttachment::MimeAttachment(QFile *file)
	: MimeFile(file)
{
}

MimeAttachment::~MimeAttachment()
{
}

void MimeAttachment::prepare()
{
	this->header += "Content-disposition: attachment\r\n";
	MimeFile::prepare();
}
```

### `./mimeattachment.h`

```cpp
﻿#ifndef MIMEATTACHMENT_H
#define MIMEATTACHMENT_H

#include <QFile>
#include "mimepart.h"
#include "mimefile.h"

class MimeAttachment : public MimeFile
{
	Q_OBJECT
public:
	MimeAttachment(QFile *file);
	~MimeAttachment();

protected:
	virtual void prepare();

};

#endif // MIMEATTACHMENT_H
```

### `./mimecontentformatter.cpp`

```cpp
﻿#include "mimecontentformatter.h"

MimeContentFormatter::MimeContentFormatter(int max_length) :
	max_length(max_length)
{}

QString MimeContentFormatter::format(const QString &content, bool quotedPrintable) const
{
	QString out;

	int chars = 0;

	for (int i = 0; i < content.length() ; ++i) {
		chars++;

		if (!quotedPrintable) {
			if (chars > max_length) {
				out.append("\r\n");
				chars = 1;
			}
		} else {
			if (content.at(i) == '\n') {       // new line
				out.append(content.at(i));
				chars = 0;
				continue;
			}

			if ((chars > max_length - 1)
			        || ((content.at(i) == '=') && (chars > max_length - 3))) {
				out.append('=');
				out.append("\r\n");
				chars = 1;
			}

		}

		out.append(content.at(i));
	}

	return out;
}

void MimeContentFormatter::setMaxLength(int l)
{
	max_length = l;
}

int MimeContentFormatter::getMaxLength() const
{
	return max_length;
}
```

### `./mimecontentformatter.h`

```cpp
﻿#ifndef MIMECONTENTFORMATTER_H
#define MIMECONTENTFORMATTER_H

#include <QObject>
#include <QByteArray>

class MimeContentFormatter : public QObject
{
	Q_OBJECT
public:
	MimeContentFormatter(int max_length = 76);

	void setMaxLength(int l);
	int getMaxLength() const;

	QString format(const QString &content, bool quotedPrintable = false) const;

protected:
	int max_length;

};

#endif // MIMECONTENTFORMATTER_H
```

### `./mimefile.cpp`

```cpp
﻿#include "mimefile.h"
#include <QFileInfo>

MimeFile::MimeFile(QFile *file)
{
	this->file = file;
	this->cType = "application/octet-stream";
	this->cName = QFileInfo(*file).fileName();
	this->cEncoding = Base64;
}

MimeFile::~MimeFile()
{
	delete file;
}

void MimeFile::prepare()
{
	file->open(QIODevice::ReadOnly);
	this->content = file->readAll();
	file->close();
	MimePart::prepare();
}
```

### `./mimefile.h`

```cpp
﻿#ifndef MIMEFILE_H
#define MIMEFILE_H

#include "mimepart.h"
#include <QFile>

class MimeFile : public MimePart
{
	Q_OBJECT
public:

	MimeFile(QFile *f);
	~MimeFile();

protected:
	QFile *file;
	virtual void prepare();

};

#endif // MIMEFILE_H
```

### `./mimehtml.cpp`

```cpp
﻿#include "mimehtml.h"

MimeHtml::MimeHtml(const QString &html) : MimeText(html)
{
	this->cType = "text/html";
}

MimeHtml::~MimeHtml() {}

void MimeHtml::setHtml(const QString &html)
{
	this->text = html;
}

const QString &MimeHtml::getHtml() const
{
	return text;
}

void MimeHtml::prepare()
{
	MimeText::prepare();
}
```

### `./mimehtml.h`

```cpp
﻿#ifndef MIMEHTML_H
#define MIMEHTML_H

#include "mimetext.h"

class MimeHtml : public MimeText
{
	Q_OBJECT
public:
	MimeHtml(const QString &html = "");
	~MimeHtml();

	void setHtml(const QString &html);
	const QString &getHtml() const;

protected:
	virtual void prepare();

};

#endif // MIMEHTML_H
```

### `./mimeinlinefile.cpp`

```cpp
﻿#include "mimeinlinefile.h"

MimeInlineFile::MimeInlineFile(QFile *f)
	: MimeFile(f)
{
}

MimeInlineFile::~MimeInlineFile()
{}

void MimeInlineFile::prepare()
{
	this->header += "Content-Disposition: inline\r\n";
	MimeFile::prepare();
}
```

### `./mimeinlinefile.h`

```cpp
﻿#ifndef MIMEINLINEFILE_H
#define MIMEINLINEFILE_H

#include "mimefile.h"

class MimeInlineFile : public MimeFile
{
public:

	MimeInlineFile(QFile *f);
	~MimeInlineFile();

protected:
	virtual void prepare();

};

#endif // MIMEINLINEFILE_H
```

### `./mimemessage.cpp`

```cpp
﻿#include "mimemessage.h"

#include <QDateTime>
#include "quotedprintable.h"
#include <typeinfo>

MimeMessage::MimeMessage(bool createAutoMimeContent) :
    hEncoding(MimePart::_8Bit)
{
    if (createAutoMimeContent) {
        this->content = new MimeMultiPart();
    }
}

MimeMessage::~MimeMessage()
{
}

MimePart &MimeMessage::getContent()
{
    return *content;
}

void MimeMessage::setContent(MimePart *content)
{
    this->content = content;
}

void MimeMessage::setSender(EmailAddress *e)
{
    this->sender = e;
}

void MimeMessage::addRecipient(EmailAddress *rcpt, RecipientType type)
{
    switch (type) {
        case To:
            recipientsTo << rcpt;
            break;

        case Cc:
            recipientsCc << rcpt;
            break;

        case Bcc:
            recipientsBcc << rcpt;
            break;
    }
}

void MimeMessage::addTo(EmailAddress *rcpt)
{
    this->recipientsTo << rcpt;
}

void MimeMessage::addCc(EmailAddress *rcpt)
{
    this->recipientsCc << rcpt;
}

void MimeMessage::addBcc(EmailAddress *rcpt)
{
    this->recipientsBcc << rcpt;
}

void MimeMessage::setSubject(const QString &subject)
{
    this->subject = subject;
}

void MimeMessage::addPart(MimePart *part)
{
    if (typeid(*content) == typeid(MimeMultiPart)) {
        ((MimeMultiPart *) content)->addPart(part);
    };
}

void MimeMessage::setHeaderEncoding(MimePart::Encoding hEnc)
{
    this->hEncoding = hEnc;
}

const EmailAddress &MimeMessage::getSender() const
{
    return *sender;
}

const QList<EmailAddress *> &MimeMessage::getRecipients(RecipientType type) const
{
    switch (type) {
        default:
        case To:
            return recipientsTo;

        case Cc:
            return recipientsCc;

        case Bcc:
            return recipientsBcc;
    }
}

const QString &MimeMessage::getSubject() const
{
    return subject;
}

const QList<MimePart *> &MimeMessage::getParts() const
{
    if (typeid(*content) == typeid(MimeMultiPart)) {
        return ((MimeMultiPart *) content)->getParts();
    } else {
        QList<MimePart *> *res = new QList<MimePart *>();
        res->append(content);
        return *res;
    }
}

QString MimeMessage::toString()
{
    QString mime;
    mime = "From:";

    QByteArray name = sender->getName().toUtf8();
    if (name != "") {
        switch (hEncoding) {
            case MimePart::Base64:
                mime += " =?utf-8?B?" + QByteArray().append(name).toBase64() + "?=";
                break;

            case MimePart::QuotedPrintable:
                mime += " =?utf-8?Q?" + QuotedPrintable::encode(QByteArray().append(name)).replace(' ', "_").replace(':', "=3A") + "?=";
                break;

            default:
                mime += " " + name;
        }
    }

    mime += " <" + sender->getAddress() + ">\r\n";

    mime += "To:";
    QList<EmailAddress *>::iterator it;
    int i;

    for (i = 0, it = recipientsTo.begin(); it != recipientsTo.end(); ++it, ++i) {
        if (i != 0) {
            mime += ",";
        }

        QByteArray name = (*it)->getName().toUtf8();
        if (name != "") {
            switch (hEncoding) {
                case MimePart::Base64:
                    mime += " =?utf-8?B?" + QByteArray().append(name).toBase64() + "?=";
                    break;

                case MimePart::QuotedPrintable:
                    mime += " =?utf-8?Q?" + QuotedPrintable::encode(QByteArray().append(name)).replace(' ', "_").replace(':', "=3A") + "?=";
                    break;

                default:
                    mime += " " + name;
            }
        }

        mime += " <" + (*it)->getAddress() + ">";
    }

    mime += "\r\n";
    if (recipientsCc.size() != 0) {
        mime += "Cc:";
    }

    for (i = 0, it = recipientsCc.begin(); it != recipientsCc.end(); ++it, ++i) {
        if (i != 0) {
            mime += ",";
        }

        QByteArray name = (*it)->getName().toUtf8();
        if (name != "") {
            switch (hEncoding) {
                case MimePart::Base64:
                    mime += " =?utf-8?B?" + QByteArray().append(name).toBase64() + "?=";
                    break;

                case MimePart::QuotedPrintable:
                    mime += " =?utf-8?Q?" + QuotedPrintable::encode(QByteArray().append(name)).replace(' ', "_").replace(':', "=3A") + "?=";
                    break;

                default:
                    mime += " " + name;
            }
        }

        mime += " <" + (*it)->getAddress() + ">";
    }

    if (recipientsCc.size() != 0) {
        mime += "\r\n";
    }

    mime += "Subject: ";

    switch (hEncoding) {
        case MimePart::Base64:
            mime += "=?utf-8?B?" + QByteArray().append(subject.toUtf8()).toBase64() + "?=";
            break;

        case MimePart::QuotedPrintable:
            mime += "=?utf-8?Q?" + QuotedPrintable::encode(QByteArray().append(subject.toUtf8())).replace(' ', "_").replace(':', "=3A") + "?=";
            break;

        default:
            mime += subject;
    }

    mime += "\r\n";
    mime += "MIME-Version: 1.0\r\n";

    mime += content->toString();
    return mime;
}
```

### `./mimemessage.h`

```cpp
﻿#ifndef MIMEMESSAGE_H
#define MIMEMESSAGE_H

#include "mimepart.h"
#include "mimemultipart.h"
#include "emailaddress.h"
#include <QList>

class MimeMessage : public QObject
{
public:

	enum RecipientType {
		To,                 // primary
		Cc,                 // carbon copy
		Bcc                 // blind carbon copy
	};


	MimeMessage(bool createAutoMimeConent = true);
	~MimeMessage();

	void setSender(EmailAddress *e);
	void addRecipient(EmailAddress *rcpt, RecipientType type = To);
	void addTo(EmailAddress *rcpt);
	void addCc(EmailAddress *rcpt);
	void addBcc(EmailAddress *rcpt);
	void setSubject(const QString &subject);
	void addPart(MimePart *part);

	void setHeaderEncoding(MimePart::Encoding);

	const EmailAddress &getSender() const;
	const QList<EmailAddress *> &getRecipients(RecipientType type = To) const;
	const QString &getSubject() const;
	const QList<MimePart *> &getParts() const;

	MimePart &getContent();
	void setContent(MimePart *content);

	virtual QString toString();


protected:
	EmailAddress *sender;
	QList<EmailAddress *> recipientsTo, recipientsCc, recipientsBcc;
	QString subject;
	MimePart *content;

	MimePart::Encoding hEncoding;

};

#endif // MIMEMESSAGE_H
```

### `./mimemultipart.cpp`

```cpp
﻿#include "mimemultipart.h"
#include "stdlib.h"
#include <QTime>
#include <QCryptographicHash>

const QString MULTI_PART_NAMES[] = {
    "multipart/mixed",         //    Mixed
    "multipart/digest",        //    Digest
    "multipart/alternative",   //    Alternative
    "multipart/related",       //    Related
    "multipart/report",        //    Report
    "multipart/signed",        //    Signed
    "multipart/encrypted"      //    Encrypted
};

MimeMultiPart::MimeMultiPart(MultiPartType type)
{
    this->type = type;
    this->cType = MULTI_PART_NAMES[this->type];
    this->cEncoding = _8Bit;

    QCryptographicHash md5(QCryptographicHash::Md5);
    md5.addData(QByteArray().append(rand()));
    cBoundary = md5.result().toHex();
}

MimeMultiPart::~MimeMultiPart()
{

}

void MimeMultiPart::addPart(MimePart *part)
{
    parts.append(part);
}

const QList<MimePart *> &MimeMultiPart::getParts() const
{
    return parts;
}

void MimeMultiPart::prepare()
{
    QList<MimePart *>::iterator it;
    content = "";

    for (it = parts.begin(); it != parts.end(); it++) {
        content += "--" + cBoundary.toUtf8() + "\r\n";
        (*it)->prepare();
        content += (*it)->toString().toUtf8();
    };

    content += "--" + cBoundary.toUtf8() + "--\r\n";
    MimePart::prepare();
}

void MimeMultiPart::setMimeType(const MultiPartType type)
{
    this->type = type;
    this->cType = MULTI_PART_NAMES[type];
}

MimeMultiPart::MultiPartType MimeMultiPart::getMimeType() const
{
    return type;
}
```

### `./mimemultipart.h`

```cpp
﻿#ifndef MIMEMULTIPART_H
#define MIMEMULTIPART_H

#include "mimepart.h"

class MimeMultiPart : public MimePart
{
	Q_OBJECT
public:
	enum MultiPartType {
		Mixed           = 0,            // RFC 2046, section 5.1.3
		Digest          = 1,            // RFC 2046, section 5.1.5
		Alternative     = 2,            // RFC 2046, section 5.1.4
		Related         = 3,            // RFC 2387
		Report          = 4,            // RFC 6522
		Signed          = 5,            // RFC 1847, section 2.1
		Encrypted       = 6             // RFC 1847, section 2.2
	};

	MimeMultiPart(const MultiPartType type = Related);
	~MimeMultiPart();

	void setMimeType(const MultiPartType type);
	MultiPartType getMimeType() const;

	const QList<MimePart *> &getParts() const;

	void addPart(MimePart *part);
	virtual void prepare();

protected:
	QList<MimePart *> parts;
	MultiPartType type;

};

#endif // MIMEMULTIPART_H
```

### `./mimepart.cpp`

```cpp
﻿#include "mimepart.h"
#include "quotedprintable.h"

MimePart::MimePart()
{
	cEncoding = _7Bit;
	prepared = false;
	cBoundary = "";
}

MimePart::~MimePart()
{
	return;
}

void MimePart::setContent(const QByteArray &content)
{
	this->content = content;
}

void MimePart::setHeader(const QString &header)
{
	this->header = header;
}

void MimePart::addHeaderLine(const QString &line)
{
	this->header += line + "\r\n";
}

const QString &MimePart::getHeader() const
{
	return header;
}

const QByteArray &MimePart::getContent() const
{
	return content;
}

void MimePart::setContentId(const QString &cId)
{
	this->cId = cId;
}

const QString &MimePart::getContentId() const
{
	return this->cId;
}

void MimePart::setContentName(const QString &cName)
{
	this->cName = cName;
}

const QString &MimePart::getContentName() const
{
	return this->cName;
}

void MimePart::setContentType(const QString &cType)
{
	this->cType = cType;
}

const QString &MimePart::getContentType() const
{
	return this->cType;
}

void MimePart::setCharset(const QString &charset)
{
	this->cCharset = charset;
}

const QString &MimePart::getCharset() const
{
	return this->cCharset;
}

void MimePart::setEncoding(Encoding enc)
{
	this->cEncoding = enc;
}

MimePart::Encoding MimePart::getEncoding() const
{
	return this->cEncoding;
}

MimeContentFormatter &MimePart::getContentFormatter()
{
	return this->formatter;
}

QString MimePart::toString()
{
	if (!prepared) {
		prepare();
	}

	return mimeString;
}

void MimePart::prepare()
{
	mimeString = QString();

	mimeString.append("Content-Type: ").append(cType);

	if (cName != "") {
		mimeString.append("; name=\"").append(cName).append("\"");
	}

	if (cCharset != "") {
		mimeString.append("; charset=").append(cCharset);
	}

	if (cBoundary != "") {
		mimeString.append("; boundary=").append(cBoundary);
	}

	mimeString.append("\r\n");

	mimeString.append("Content-Transfer-Encoding: ");

	switch (cEncoding) {
		case _7Bit:
			mimeString.append("7bit\r\n");
			break;

		case _8Bit:
			mimeString.append("8bit\r\n");
			break;

		case Base64:
			mimeString.append("base64\r\n");
			break;

		case QuotedPrintable:
			mimeString.append("quoted-printable\r\n");
			break;
	}

    if (cId.toInt() != 0) {
		mimeString.append("Content-ID: <").append(cId).append(">\r\n");
	}

	mimeString.append(header).append("\r\n");

	switch (cEncoding) {
		case _7Bit:
			mimeString.append(QString(content).toLatin1());
			break;

		case _8Bit:
			mimeString.append(content);
			break;

		case Base64:
			mimeString.append(formatter.format(content.toBase64()));
			break;

		case QuotedPrintable:
			mimeString.append(formatter.format(QuotedPrintable::encode(content), true));
			break;
	}

	mimeString.append("\r\n");
	prepared = true;
}
```

### `./mimepart.h`

```cpp
﻿#ifndef MIMEPART_H
#define MIMEPART_H

#include <QObject>
#include "mimecontentformatter.h"

class MimePart : public QObject
{
	Q_OBJECT
public:
	enum Encoding {
		_7Bit,
		_8Bit,
		Base64,
		QuotedPrintable
	};

	MimePart();
	~MimePart();

	const QString &getHeader() const;
	const QByteArray &getContent() const;

	void setContent(const QByteArray &content);
	void setHeader(const QString &header);

	void addHeaderLine(const QString &line);

	void setContentId(const QString &cId);
	const QString &getContentId() const;

	void setContentName(const QString &cName);
	const QString &getContentName() const;

	void setContentType(const QString &cType);
	const QString &getContentType() const;

	void setCharset(const QString &charset);
	const QString &getCharset() const;

	void setEncoding(Encoding enc);
	Encoding getEncoding() const;

	MimeContentFormatter &getContentFormatter();

	virtual QString toString();
	virtual void prepare();

protected:
	QString header;
	QByteArray content;

	QString cId;
	QString cName;
	QString cType;
	QString cCharset;
	QString cBoundary;
	Encoding cEncoding;

	QString mimeString;
	bool prepared;

	MimeContentFormatter formatter;

};

#endif // MIMEPART_H
```

### `./mimetext.cpp`

```cpp
﻿#include "mimetext.h"

MimeText::MimeText(const QString &txt)
{
	this->text = txt;
	this->cType = "text/plain";
	this->cCharset = "utf-8";
	this->cEncoding = _8Bit;
}

MimeText::~MimeText() { }

void MimeText::setText(const QString &text)
{
	this->text = text;
}

const QString &MimeText::getText() const
{
	return text;
}

void MimeText::prepare()
{
	this->content.clear();
    this->content.append(text.toUtf8());
	MimePart::prepare();
}
```

### `./mimetext.h`

```cpp
﻿#ifndef MIMETEXT_H
#define MIMETEXT_H

#include "mimepart.h"

class MimeText : public MimePart
{
public:

	MimeText(const QString &text = "");
	~MimeText();

	void setText(const QString &text);
	const QString &getText() const;

protected:
	QString text;
	void prepare();

};

#endif // MIMETEXT_H
```

### `./quotedprintable.cpp`

```cpp
﻿#include "quotedprintable.h"

QString &QuotedPrintable::encode(const QByteArray &input)
{
	QString *output = new QString();

	char byte;
	const char hex[] = {'0', '1', '2', '3', '4', '5', '6', '7', '8', '9', 'A', 'B', 'C', 'D', 'E', 'F'};

	for (int i = 0; i < input.length() ; ++i) {
		byte = input.at(i);

		if ((byte == 0x20) || ((byte >= 33) && (byte <= 126) && (byte != 61))) {
			output->append(byte);
		} else {
			output->append('=');
			output->append(hex[((byte >> 4) & 0x0F)]);
			output->append(hex[(byte & 0x0F)]);
		}
	}

	return *output;
}


QByteArray &QuotedPrintable::decode(const QString &input)
{
	//                    0  1  2  3  4  5  6  7  8  9  :  ;  <  =  >  ?  @  A   B   C   D   E   F
	const int hexVal[] = {0, 1, 2, 3, 4, 5, 6, 7, 8, 9, 0, 0, 0, 0, 0, 0, 0, 10, 11, 12, 13, 14, 15};

	QByteArray *output = new QByteArray();

	for (int i = 0; i < input.length(); ++i) {
		if (input.at(i).toLatin1() == '=') {
			int index = ++i;
			output->append((hexVal[input.at(index).toLatin1() - '0'] << 4) + hexVal[input.at(++i).toLatin1() - '0']);
		} else {
			output->append(input.at(i).toLatin1());
		}
	}

	return *output;
}
```

### `./quotedprintable.h`

```cpp
﻿#ifndef QUOTEDPRINTABLE_H
#define QUOTEDPRINTABLE_H

#include <QObject>
#include <QByteArray>

class QuotedPrintable : public QObject
{
	Q_OBJECT
public:

	static QString &encode(const QByteArray &input);
	static QByteArray &decode(const QString &input);

private:
	QuotedPrintable();
};

#endif // QUOTEDPRINTABLE_H
```

### `./smtpclient.cpp`

```cpp
﻿#include "smtpclient.h"

SmtpClient::SmtpClient(const QString &host, int port, ConnectionType connectionType) :
	name("localhost"),
	authMethod(AuthPlain),
	connectionTimeout(5000),
	responseTimeout(5000)
{
	setConnectionType(connectionType);

	this->host = host;
	this->port = port;
}

SmtpClient::~SmtpClient() {}

void SmtpClient::setUser(const QString &user)
{
	this->user = user;
}

void SmtpClient::setPassword(const QString &password)
{
	this->password = password;
}

void SmtpClient::setAuthMethod(AuthMethod method)
{
	this->authMethod = method;
}

void SmtpClient::setHost(QString &host)
{
	this->host = host;
}

void SmtpClient::setPort(int port)
{
	this->port = port;
}

void SmtpClient::setConnectionType(ConnectionType ct)
{
	this->connectionType = ct;

	switch (connectionType) {
		case TcpConnection:
			socket = new QTcpSocket(this);
			break;
#ifdef ssl
		case SslConnection:
		case TlsConnection:
			socket = new QSslSocket(this);
#endif
	}
}

const QString &SmtpClient::getHost() const
{
	return this->host;
}

const QString &SmtpClient::getUser() const
{
	return this->user;
}

const QString &SmtpClient::getPassword() const
{
	return this->password;
}

SmtpClient::AuthMethod SmtpClient::getAuthMethod() const
{
	return this->authMethod;
}

int SmtpClient::getPort() const
{
	return this->port;
}

SmtpClient::ConnectionType SmtpClient::getConnectionType() const
{
	return connectionType;
}

const QString &SmtpClient::getName() const
{
	return this->name;
}

void SmtpClient::setName(const QString &name)
{
	this->name = name;
}

const QString &SmtpClient::getResponseText() const
{
	return responseText;
}

int SmtpClient::getResponseCode() const
{
	return responseCode;
}

QTcpSocket *SmtpClient::getSocket()
{
	return socket;
}

int SmtpClient::getConnectionTimeout() const
{
	return connectionTimeout;
}

void SmtpClient::setConnectionTimeout(int msec)
{
	connectionTimeout = msec;
}

int SmtpClient::getResponseTimeout() const
{
	return responseTimeout;
}

void SmtpClient::setResponseTimeout(int msec)
{
	responseTimeout = msec;
}

bool SmtpClient::connectToHost()
{
	switch (connectionType) {
		case TlsConnection:
		case TcpConnection:
			socket->connectToHost(host, port);
			break;
#ifdef ssl
		case SslConnection:
			((QSslSocket *) socket)->connectToHostEncrypted(host, port);
			break;
#endif
	}

	if (!socket->waitForConnected(connectionTimeout)) {
		emit smtpError(ConnectionTimeoutError);
		return false;
	}

	try {
		// Wait for the server's response
		waitForResponse();

		// If the response code is not 220 (Service ready)
		// means that is something wrong with the server
		if (responseCode != 220) {
			emit smtpError(ServerError);
			return false;
		}

		// Send a EHLO/HELO message to the server
		// The client's first command must be EHLO/HELO
		sendMessage("EHLO " + name);

		// Wait for the server's response
		waitForResponse();

		// The response code needs to be 250.
		if (responseCode != 250) {
			emit smtpError(ServerError);
			return false;
		}

		if (connectionType == TlsConnection) {
			// send a request to start TLS handshake
			sendMessage("STARTTLS");

			// Wait for the server's response
			waitForResponse();

			// The response code needs to be 220.
			if (responseCode != 220) {
				emit smtpError(ServerError);
				return false;
			};
#ifdef ssl
			((QSslSocket *) socket)->startClientEncryption();
			if (!((QSslSocket *) socket)->waitForEncrypted(connectionTimeout)) {
				qDebug() << ((QSslSocket *) socket)->errorString();
				emit smtpError(ConnectionTimeoutError);
                return false;
			}
#endif
			// Send ELHO one more time
			sendMessage("EHLO " + name);

			// Wait for the server's response
			waitForResponse();

			// The response code needs to be 250.
			if (responseCode != 250) {
				emit smtpError(ServerError);
				return false;
			}
		}
	} catch (ResponseTimeoutException) {
		return false;
	}

	return true;
}

bool SmtpClient::login()
{
	return login(user, password, authMethod);
}

bool SmtpClient::login(const QString &user, const QString &password, AuthMethod method)
{
	try {
		if (method == AuthPlain) {
			// Sending command: AUTH PLAIN base64('\0' + username + '\0' + password)
            sendMessage("AUTH PLAIN " + QByteArray().append((char) 0).append(user.toUtf8()).append((char) 0).append(password.toUtf8()).toBase64());

			// Wait for the server's response
			waitForResponse();

			// If the response is not 235 then the authentication was faild
			if (responseCode != 235) {
				emit smtpError(AuthenticationFailedError);
				return false;
			}
		} else if (method == AuthLogin) {
			// Sending command: AUTH LOGIN
			sendMessage("AUTH LOGIN");

			// Wait for 334 response code
			waitForResponse();

			if (responseCode != 334) {
				emit smtpError(AuthenticationFailedError);
				return false;
			}

			// Send the username in base64
            sendMessage(QByteArray().append(user.toUtf8()).toBase64());

			// Wait for 334
			waitForResponse();

			if (responseCode != 334) {
				emit smtpError(AuthenticationFailedError);
				return false;
			}

			// Send the password in base64
            sendMessage(QByteArray().append(password.toUtf8()).toBase64());

			// Wait for the server's responce
			waitForResponse();

			// If the response is not 235 then the authentication was faild
			if (responseCode != 235) {
				emit smtpError(AuthenticationFailedError);
				return false;
			}
		}
	} catch (ResponseTimeoutException e) {
		// Responce Timeout exceeded
		emit smtpError(AuthenticationFailedError);
		return false;
	}

	return true;
}

bool SmtpClient::sendMail(MimeMessage &email)
{
	try {
		// Send the MAIL command with the sender
		sendMessage("MAIL FROM: <" + email.getSender().getAddress() + ">");

		waitForResponse();

		if (responseCode != 250) {
			return false;
		}

		// Send RCPT command for each recipient
		QList<EmailAddress *>::const_iterator it, itEnd;

		// To (primary recipients)
		for (it = email.getRecipients().begin(), itEnd = email.getRecipients().end();
		        it != itEnd; ++it) {
			sendMessage("RCPT TO: <" + (*it)->getAddress() + ">");
			waitForResponse();

			if (responseCode != 250) {
				return false;
			}
		}

		// Cc (carbon copy)
		for (it = email.getRecipients(MimeMessage::Cc).begin(), itEnd = email.getRecipients(MimeMessage::Cc).end();
		        it != itEnd; ++it) {
			sendMessage("RCPT TO: <" + (*it)->getAddress() + ">");
			waitForResponse();

			if (responseCode != 250) {
				return false;
			}
		}

		// Bcc (blind carbon copy)
		for (it = email.getRecipients(MimeMessage::Bcc).begin(), itEnd = email.getRecipients(MimeMessage::Bcc).end();
		        it != itEnd; ++it) {
			sendMessage("RCPT TO: <" + (*it)->getAddress() + ">");
			waitForResponse();

			if (responseCode != 250) {
				return false;
			}
		}

		// Send DATA command
		sendMessage("DATA");
		waitForResponse();

		if (responseCode != 354) {
			return false;
		}

		sendMessage(email.toString());

		// Send \r\n.\r\n to end the mail data
		sendMessage(".");

		waitForResponse();

		if (responseCode != 250) {
			return false;
		}
	} catch (ResponseTimeoutException) {
		return false;
	}

	return true;
}

void SmtpClient::quit()
{
	sendMessage("QUIT");
}

void SmtpClient::waitForResponse()
{
	do {
		if (!socket->waitForReadyRead(responseTimeout)) {
			emit smtpError(ResponseTimeoutError);
			throw ResponseTimeoutException();
		}

		while (socket->canReadLine()) {
			// Save the server's response
			responseText = socket->readLine();

			// Extract the respose code from the server's responce (first 3 digits)
			responseCode = responseText.left(3).toInt();

			if (responseCode / 100 == 4) {
				emit smtpError(ServerError);
			}

			if (responseCode / 100 == 5) {
				emit smtpError(ClientError);
			}

			if (responseText.at(3) == ' ') {
				return;
			}
		}
	} while (true);
}

void SmtpClient::sendMessage(const QString &text)
{
	socket->write(text.toUtf8() + "\r\n");
}
```

### `./smtpclient.h`

```cpp
﻿#ifndef SMTPCLIENT_H
#define SMTPCLIENT_H

#include <QtGui>
#include <QtNetwork>
#if (QT_VERSION >= QT_VERSION_CHECK(5,0,0))
#include <QtWidgets>
#endif
#include "mimemessage.h"

class SmtpClient : public QObject
{
	Q_OBJECT
public:
	enum AuthMethod {
		AuthPlain,
		AuthLogin
	};

	enum SmtpError {
		ConnectionTimeoutError,
		ResponseTimeoutError,
		AuthenticationFailedError,
		ServerError,    // 4xx smtp error
		ClientError     // 5xx smtp error
	};

	enum ConnectionType {
		TcpConnection,
		SslConnection,
		TlsConnection       // STARTTLS
	};

	SmtpClient(const QString &host = "locahost", int port = 25, ConnectionType ct = TcpConnection);

	~SmtpClient();

	const QString &getHost() const;
	void setHost(QString &host);

	int getPort() const;
	void setPort(int port);

	const QString &getName() const;
	void setName(const QString &name);

	ConnectionType getConnectionType() const;
	void setConnectionType(ConnectionType ct);

	const QString &getUser() const;
	void setUser(const QString &host);

	const QString &getPassword() const;
	void setPassword(const QString &password);

	SmtpClient::AuthMethod getAuthMethod() const;
	void setAuthMethod(AuthMethod method);

	const QString &getResponseText() const;
	int getResponseCode() const;

	int getConnectionTimeout() const;
	void setConnectionTimeout(int msec);

	int getResponseTimeout() const;
	void setResponseTimeout(int msec);

	QTcpSocket *getSocket();

	bool connectToHost();
	bool login();
	bool login(const QString &user, const QString &password, AuthMethod method = AuthLogin);

	bool sendMail(MimeMessage &email);
	void quit();

protected:
	QTcpSocket *socket;
	QString host;
	int port;
	ConnectionType connectionType;
	QString name;

	QString user;
	QString password;
	AuthMethod authMethod;

	int connectionTimeout;
	int responseTimeout;

	QString responseText;
	int responseCode;

	class ResponseTimeoutException {};
    void waitForResponse();
	void sendMessage(const QString &text);

signals:
	void smtpError(SmtpError e);

};

#endif // SMTPCLIENT_H
```

### `./smtpmime.h`

```cpp
﻿#ifndef SMTPMIME_H
#define SMTPMIME_H

#include "smtpclient.h"
#include "mimepart.h"
#include "mimehtml.h"
#include "mimeattachment.h"
#include "mimemessage.h"
#include "mimetext.h"
#include "mimeinlinefile.h"
#include "mimefile.h"

#endif // SMTPMIME_H
```

### `./3rd_smtpclient.pri`

```makefile
HEADERS += \
    $$PWD/emailaddress.h \
    $$PWD/mimeattachment.h \
    $$PWD/mimecontentformatter.h \
    $$PWD/mimefile.h \
    $$PWD/mimehtml.h \
    $$PWD/mimeinlinefile.h \
    $$PWD/mimemessage.h \
    $$PWD/mimemultipart.h \
    $$PWD/mimepart.h \
    $$PWD/mimetext.h \
    $$PWD/quotedprintable.h \
    $$PWD/smtpclient.h \
    $$PWD/smtpmime.h
           
SOURCES += \
    $$PWD/emailaddress.cpp \
    $$PWD/mimeattachment.cpp \
    $$PWD/mimecontentformatter.cpp \
    $$PWD/mimefile.cpp \
    $$PWD/mimehtml.cpp \
    $$PWD/mimeinlinefile.cpp \
    $$PWD/mimemessage.cpp \
    $$PWD/mimemultipart.cpp \
    $$PWD/mimepart.cpp \
    $$PWD/mimetext.cpp \
    $$PWD/quotedprintable.cpp \
    $$PWD/smtpclient.cpp
```
