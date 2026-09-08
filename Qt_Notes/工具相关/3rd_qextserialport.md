---
tags:
  - Qt
  - 工具相关
---
# 串口通信库 (`3rd_qextserialport`)

## 分类

- 类型: 工具相关 `tool`
- 源码目录: `tool/3rd_qextserialport`

## 技术要点

**用到的 Qt 类**: QWriteLocker QIODevice QString QReadLocker QLatin1String QObject QByteArray QReadWriteLock QSocketNotifier QWinEventNotifier QDebug QESP QRegExp QESP2 QList QMutexLocker QThread QRingBuffer QIODevicePrivateLinearBuffer QLatin1Char

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./qextserialport.cpp`

```cpp
﻿/****************************************************************************
** Copyright (c) 2000-2003 Wayne Roth
** Copyright (c) 2004-2007 Stefan Sander
** Copyright (c) 2007 Michal Policht
** Copyright (c) 2008 Brandon Fosdick
** Copyright (c) 2009-2010 Liam Staskawicz
** Copyright (c) 2011 Debao Zhang
** All right reserved.
** Web: http://code.google.com/p/qextserialport/
**
** Permission is hereby granted, free of charge, to any person obtaining
** a copy of this software and associated documentation files (the
** "Software"), to deal in the Software without restriction, including
** without limitation the rights to use, copy, modify, merge, publish,
** distribute, sublicense, and/or sell copies of the Software, and to
** permit persons to whom the Software is furnished to do so, subject to
** the following conditions:
**
** The above copyright notice and this permission notice shall be
** included in all copies or substantial portions of the Software.
**
** THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
** EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
** MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
** NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
** LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
** OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
** WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
**
****************************************************************************/

#include "qextserialport.h"
#include "qextserialport_p.h"
#include <stdio.h>
#include <QtCore/QDebug>
#include <QtCore/QReadLocker>
#include <QtCore/QWriteLocker>

/*!
    \class PortSettings

    \brief The PortSettings class contain port settings

    Structure to contain port settings.

    \code
    BaudRateType BaudRate;
    DataBitsType DataBits;
    ParityType Parity;
    StopBitsType StopBits;
    FlowType FlowControl;
    long Timeout_Millisec;
    \endcode
*/

QextSerialPortPrivate::QextSerialPortPrivate(QextSerialPort *q)
	: lock(QReadWriteLock::Recursive), q_ptr(q)
{
	lastErr = E_NO_ERROR;
	settings.BaudRate = BAUD9600;
	settings.Parity = PAR_NONE;
	settings.FlowControl = FLOW_OFF;
	settings.DataBits = DATA_8;
	settings.StopBits = STOP_1;
	settings.Timeout_Millisec = 10;
	settingsDirtyFlags = DFE_ALL;

	platformSpecificInit();
}

QextSerialPortPrivate::~QextSerialPortPrivate()
{
	platformSpecificDestruct();
}

void QextSerialPortPrivate::setBaudRate(BaudRateType baudRate, bool update)
{
	switch (baudRate) {
#ifdef Q_OS_WIN

		//Windows Special
		case BAUD14400:
		case BAUD56000:
		case BAUD128000:
		case BAUD256000:
			QESP_PORTABILITY_WARNING() << "QextSerialPort Portability Warning: POSIX does not support baudRate:" << baudRate;
#elif defined(Q_OS_UNIX)

		//Unix Special
		case BAUD50:
		case BAUD75:
		case BAUD134:
		case BAUD150:
		case BAUD200:
		case BAUD1800:
#  ifdef B76800
		case BAUD76800:
#  endif
#  if defined(B230400) && defined(B4000000)
		case BAUD230400:
		case BAUD460800:
		case BAUD500000:
		case BAUD576000:
		case BAUD921600:
		case BAUD1000000:
		case BAUD1152000:
		case BAUD1500000:
		case BAUD2000000:
		case BAUD2500000:
		case BAUD3000000:
		case BAUD3500000:
		case BAUD4000000:
#  endif
			QESP_PORTABILITY_WARNING() << "QextSerialPort Portability Warning: Windows does not support baudRate:" << baudRate;
#endif

		case BAUD110:
		case BAUD300:
		case BAUD600:
		case BAUD1200:
		case BAUD2400:
		case BAUD4800:
		case BAUD9600:
		case BAUD19200:
		case BAUD38400:
		case BAUD57600:
		case BAUD115200:
#if defined(Q_OS_WIN) || defined(Q_OS_MAC)
		default:
#endif
			settings.BaudRate = baudRate;
			settingsDirtyFlags |= DFE_BaudRate;

			if (update && q_func()->isOpen()) {
				updatePortSettings();
			}

			break;
#if !(defined(Q_OS_WIN) || defined(Q_OS_MAC))

		default:
			QESP_WARNING() << "QextSerialPort does not support baudRate:" << baudRate;
#endif
	}
}

void QextSerialPortPrivate::setParity(ParityType parity, bool update)
{
	switch (parity) {
		case PAR_SPACE:
			if (settings.DataBits == DATA_8) {
#ifdef Q_OS_WIN
				QESP_PORTABILITY_WARNING("QextSerialPort Portability Warning: Space parity with 8 data bits is not supported by POSIX systems.");
#else
				QESP_WARNING("Space parity with 8 data bits is not supported by POSIX systems.");
#endif
			}

			break;

#ifdef Q_OS_WIN

		/*mark parity - WINDOWS ONLY*/
		case PAR_MARK:
			QESP_PORTABILITY_WARNING("QextSerialPort Portability Warning:  Mark parity is not supported by POSIX systems");
			break;
#endif

		case PAR_NONE:
		case PAR_EVEN:
		case PAR_ODD:
			break;

		default:
			QESP_WARNING() << "QextSerialPort does not support Parity:" << parity;
	}

	settings.Parity = parity;
	settingsDirtyFlags |= DFE_Parity;

	if (update && q_func()->isOpen()) {
		updatePortSettings();
	}
}

void QextSerialPortPrivate::setDataBits(DataBitsType dataBits, bool update)
{
	switch (dataBits) {

		case DATA_5:
			if (settings.StopBits == STOP_2) {
				QESP_WARNING("QextSerialPort: 5 Data bits cannot be used with 2 stop bits.");
			} else {
				settings.DataBits = dataBits;
				settingsDirtyFlags |= DFE_DataBits;
			}

			break;

		case DATA_6:
#ifdef Q_OS_WIN
			if (settings.StopBits == STOP_1_5) {
				QESP_WARNING("QextSerialPort: 6 Data bits cannot be used with 1.5 stop bits.");
			} else
#endif
			{
				settings.DataBits = dataBits;
				settingsDirtyFlags |= DFE_DataBits;
			}

			break;

		case DATA_7:
#ifdef Q_OS_WIN
			if (settings.StopBits == STOP_1_5) {
				QESP_WARNING("QextSerialPort: 7 Data bits cannot be used with 1.5 stop bits.");
			} else
#endif
			{
				settings.DataBits = dataBits;
				settingsDirtyFlags |= DFE_DataBits;
			}

			break;

		case DATA_8:
#ifdef Q_OS_WIN
			if (settings.StopBits == STOP_1_5) {
				QESP_WARNING("QextSerialPort: 8 Data bits cannot be used with 1.5 stop bits.");
			} else
#endif
			{
				settings.DataBits = dataBits;
				settingsDirtyFlags |= DFE_DataBits;
			}

			break;

		default:
			QESP_WARNING() << "QextSerialPort does not support Data bits:" << dataBits;
	}

	if (update && q_func()->isOpen()) {
		updatePortSettings();
	}
}

void QextSerialPortPrivate::setStopBits(StopBitsType stopBits, bool update)
{
	switch (stopBits) {

		/*one stop bit*/
		case STOP_1:
			settings.StopBits = stopBits;
			settingsDirtyFlags |= DFE_StopBits;
			break;

#ifdef Q_OS_WIN

		/*1.5 stop bits*/
		case STOP_1_5:
			QESP_PORTABILITY_WARNING("QextSerialPort Portability Warning: 1.5 stop bit operation is not supported by POSIX.");

			if (settings.DataBits != DATA_5) {
				QESP_WARNING("QextSerialPort: 1.5 stop bits can only be used with 5 data bits");
			} else {
				settings.StopBits = stopBits;
				settingsDirtyFlags |= DFE_StopBits;
			}

			break;
#endif

		/*two stop bits*/
		case STOP_2:
			if (settings.DataBits == DATA_5) {
				QESP_WARNING("QextSerialPort: 2 stop bits cannot be used with 5 data bits");
			} else {
				settings.StopBits = stopBits;
				settingsDirtyFlags |= DFE_StopBits;
			}

			break;

		default:
			QESP_WARNING() << "QextSerialPort does not support stop bits: " << stopBits;
	}

	if (update && q_func()->isOpen()) {
		updatePortSettings();
	}
}

void QextSerialPortPrivate::setFlowControl(FlowType flow, bool update)
{
	settings.FlowControl = flow;
	settingsDirtyFlags |= DFE_Flow;

	if (update && q_func()->isOpen()) {
		updatePortSettings();
	}
}

void QextSerialPortPrivate::setTimeout(long millisec, bool update)
{
	settings.Timeout_Millisec = millisec;
	settingsDirtyFlags |= DFE_TimeOut;

	if (update && q_func()->isOpen()) {
		updatePortSettings();
	}
}

void QextSerialPortPrivate::setPortSettings(const PortSettings &settings, bool update)
{
	setBaudRate(settings.BaudRate, false);
	setDataBits(settings.DataBits, false);
	setStopBits(settings.StopBits, false);
	setParity(settings.Parity, false);
	setFlowControl(settings.FlowControl, false);
	setTimeout(settings.Timeout_Millisec, false);
	settingsDirtyFlags = DFE_ALL;

	if (update && q_func()->isOpen()) {
		updatePortSettings();
	}
}


void QextSerialPortPrivate::_q_canRead()
{
	qint64 maxSize = bytesAvailable_sys();

	if (maxSize > 0) {
		char *writePtr = readBuffer.reserve(size_t(maxSize));
		qint64 bytesRead = readData_sys(writePtr, maxSize);

		if (bytesRead < maxSize) {
			readBuffer.chop(maxSize - bytesRead);
		}

		Q_Q(QextSerialPort);
		Q_EMIT q->readyRead();
	}
}

/*! \class QextSerialPort

    \brief The QextSerialPort class encapsulates a serial port on both POSIX and Windows systems.

    \section1 Usage
    QextSerialPort offers both a polling and event driven API.  Event driven
    is typically easier to use, since you never have to worry about checking
    for new data.

    \bold Example
    \code
    QextSerialPort *port = new QextSerialPort("COM1");
    connect(port, SIGNAL(readyRead()), myClass, SLOT(onDataAvailable()));
    port->open();

    void MyClass::onDataAvailable()
    {
        QByteArray data = port->readAll();
        processNewData(usbdata);
    }
    \endcode

    \section1 Compatibility
    The user will be notified of errors and possible portability conflicts at run-time
    by default.

    For example, if a application has used BAUD1800, when it is runing under unix, you
    will get following message.

    \code
    QextSerialPort Portability Warning: Windows does not support baudRate:1800
    \endcode

    This behavior can be turned off by defining macro QESP_NO_WARN (to turn off all warnings)
    or QESP_NO_PORTABILITY_WARN (to turn off portability warnings) in the project.


    \bold Author: Stefan Sander, Michal Policht, Brandon Fosdick, Liam Staskawicz, Debao Zhang
*/

/*!
  \enum QextSerialPort::QueryMode

  This enum type specifies query mode used in a serial port:

  \value Polling
     asynchronously read and write
  \value EventDriven
     synchronously read and write
*/

/*!
    \fn void QextSerialPort::dsrChanged(bool status)
    This signal is emitted whenever dsr line has changed its state. You may
    use this signal to check if device is connected.

    \a status true when DSR signal is on, false otherwise.
 */


/*!
    \fn QueryMode QextSerialPort::queryMode() const
    Get query mode.
 */

/*!
    Default constructor.  Note that the name of the device used by a QextSerialPort is dependent on
    your OS. Possible naming conventions and their associated OS are:

    \code

    OS Constant       Used By         Naming Convention
    -------------     -------------   ------------------------
    Q_OS_WIN          Windows         COM1, COM2
    Q_OS_IRIX         SGI/IRIX        /dev/ttyf1, /dev/ttyf2
    Q_OS_HPUX         HP-UX           /dev/tty1p0, /dev/tty2p0
    Q_OS_SOLARIS      SunOS/Slaris    /dev/ttya, /dev/ttyb
    Q_OS_OSF          Digital UNIX    /dev/tty01, /dev/tty02
    Q_OS_FREEBSD      FreeBSD         /dev/ttyd0, /dev/ttyd1
    Q_OS_OPENBSD      OpenBSD         /dev/tty00, /dev/tty01
    Q_OS_LINUX        Linux           /dev/ttyS0, /dev/ttyS1
    <none>                            /dev/ttyS0, /dev/ttyS1
    \endcode

    This constructor assigns the device name to the name of the first port on the specified system.
    See the other constructors if you need to open a different port. Default \a mode is EventDriven.
    As a subclass of QObject, \a parent can be specified.
*/

QextSerialPort::QextSerialPort(QextSerialPort::QueryMode mode, QObject *parent)
	: QIODevice(parent), d_ptr(new QextSerialPortPrivate(this))
{
#ifdef Q_OS_WIN
	setPortName(QLatin1String("COM1"));

#elif defined(Q_OS_IRIX)
	setPortName(QLatin1String("/dev/ttyf1"));

#elif defined(Q_OS_HPUX)
	setPortName(QLatin1String("/dev/tty1p0"));

#elif defined(Q_OS_SOLARIS)
	setPortName(QLatin1String("/dev/ttya"));

#elif defined(Q_OS_OSF) //formally DIGITAL UNIX
	setPortName(QLatin1String("/dev/tty01"));

#elif defined(Q_OS_FREEBSD)
	setPortName(QLatin1String("/dev/ttyd1"));

#elif defined(Q_OS_OPENBSD)
	setPortName(QLatin1String("/dev/tty00"));

#else
	setPortName(QLatin1String("/dev/ttyS0"));
#endif
	setQueryMode(mode);
}

/*!
    Constructs a serial port attached to the port specified by name.
    \a name is the name of the device, which is windowsystem-specific,
    e.g."COM1" or "/dev/ttyS0". \a mode
*/
QextSerialPort::QextSerialPort(const QString &name, QextSerialPort::QueryMode mode, QObject *parent)
	: QIODevice(parent), d_ptr(new QextSerialPortPrivate(this))
{
	setQueryMode(mode);
	setPortName(name);
}

/*!
    Constructs a port with default name and specified \a settings.
*/
QextSerialPort::QextSerialPort(const PortSettings &settings, QextSerialPort::QueryMode mode, QObject *parent)
	: QIODevice(parent), d_ptr(new QextSerialPortPrivate(this))
{
	Q_D(QextSerialPort);
	setQueryMode(mode);
	d->setPortSettings(settings);
}

/*!
    Constructs a port with specified \a name , \a mode and \a settings.
*/
QextSerialPort::QextSerialPort(const QString &name, const PortSettings &settings, QextSerialPort::QueryMode mode, QObject *parent)
	: QIODevice(parent), d_ptr(new QextSerialPortPrivate(this))
{
	Q_D(QextSerialPort);
	setPortName(name);
	setQueryMode(mode);
	d->setPortSettings(settings);
}

/*!
    Opens a serial port and sets its OpenMode to \a mode.
    Note that this function does not specify which device to open.
    Returns true if successful; otherwise returns false.This function has no effect
    if the port associated with the class is already open.  The port is also
    configured to the current settings, as stored in the settings structure.
*/
bool QextSerialPort::open(OpenMode mode)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (mode != QIODevice::NotOpen && !isOpen()) {
		d->open_sys(mode);
	}

	return isOpen();
}


/*! \reimp
    Closes a serial port.  This function has no effect if the serial port associated with the class
    is not currently open.
*/
void QextSerialPort::close()
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (isOpen()) {
		// Be a good QIODevice and call QIODevice::close() before really close()
		//  so the aboutToClose() signal is emitted at the proper time
		QIODevice::close(); // mark ourselves as closed
		d->close_sys();
		d->readBuffer.clear();
	}
}

/*!
    Flushes all pending I/O to the serial port.  This function has no effect if the serial port
    associated with the class is not currently open.
*/
void QextSerialPort::flush()
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (isOpen()) {
		d->flush_sys();
	}
}

/*! \reimp
    Returns the number of bytes waiting in the port's receive queue.  This function will return 0 if
    the port is not currently open, or -1 on error.
*/
qint64 QextSerialPort::bytesAvailable() const
{
	QWriteLocker locker(&d_func()->lock);

	if (isOpen()) {
		qint64 bytes = d_func()->bytesAvailable_sys();

		if (bytes != -1) {
			return bytes + d_func()->readBuffer.size()
			       + QIODevice::bytesAvailable();
		} else {
			return -1;
		}
	}

	return 0;
}

/*! \reimp

*/
bool QextSerialPort::canReadLine() const
{
	QReadLocker locker(&d_func()->lock);
	return QIODevice::canReadLine() || d_func()->readBuffer.canReadLine();
}

/*!
 * Set desired serial communication handling style. You may choose from polling
 * or event driven approach. This function does nothing when port is open; to
 * apply changes port must be reopened.
 *
 * In event driven approach read() and write() functions are acting
 * asynchronously. They return immediately and the operation is performed in
 * the background, so they doesn't freeze the calling thread.
 * To determine when operation is finished, QextSerialPort runs separate thread
 * and monitors serial port events. Whenever the event occurs, adequate signal
 * is emitted.
 *
 * When polling is set, read() and write() are acting synchronously. Signals are
 * not working in this mode and some functions may not be available. The advantage
 * of polling is that it generates less overhead due to lack of signals emissions
 * and it doesn't start separate thread to monitor events.
 *
 * Generally event driven approach is more capable and friendly, although some
 * applications may need as low overhead as possible and then polling comes.
 *
 * \a mode query mode.
 */
void QextSerialPort::setQueryMode(QueryMode mode)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (mode != d->queryMode) {
		d->queryMode = mode;
	}
}

/*!
    Sets the \a name of the device associated with the object, e.g. "COM1", or "/dev/ttyS0".
*/
void QextSerialPort::setPortName(const QString &name)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);
	d->port = name;
}

/*!
    Returns the name set by setPortName().
*/
QString QextSerialPort::portName() const
{
	QReadLocker locker(&d_func()->lock);
	return d_func()->port;
}

QextSerialPort::QueryMode QextSerialPort::queryMode() const
{
	QReadLocker locker(&d_func()->lock);
	return d_func()->queryMode;
}

/*!
    Reads all available data from the device, and returns it as a QByteArray.
    This function has no way of reporting errors; returning an empty QByteArray()
    can mean either that no data was currently available for reading, or that an error occurred.
*/
QByteArray QextSerialPort::readAll()
{
	int avail = this->bytesAvailable();
	return (avail > 0) ? this->read(avail) : QByteArray();
}

/*!
    Returns the baud rate of the serial port.  For a list of possible return values see
    the definition of the enum BaudRateType.
*/
BaudRateType QextSerialPort::baudRate() const
{
	QReadLocker locker(&d_func()->lock);
	return d_func()->settings.BaudRate;
}

/*!
    Returns the number of data bits used by the port.  For a list of possible values returned by
    this function, see the definition of the enum DataBitsType.
*/
DataBitsType QextSerialPort::dataBits() const
{
	QReadLocker locker(&d_func()->lock);
	return d_func()->settings.DataBits;
}

/*!
    Returns the type of parity used by the port.  For a list of possible values returned by
    this function, see the definition of the enum ParityType.
*/
ParityType QextSerialPort::parity() const
{
	QReadLocker locker(&d_func()->lock);
	return d_func()->settings.Parity;
}

/*!
    Returns the number of stop bits used by the port.  For a list of possible return values, see
    the definition of the enum StopBitsType.
*/
StopBitsType QextSerialPort::stopBits() const
{
	QReadLocker locker(&d_func()->lock);
	return d_func()->settings.StopBits;
}

/*!
    Returns the type of flow control used by the port.  For a list of possible values returned
    by this function, see the definition of the enum FlowType.
*/
FlowType QextSerialPort::flowControl() const
{
	QReadLocker locker(&d_func()->lock);
	return d_func()->settings.FlowControl;
}

/*!
    \reimp
    Returns true if device is sequential, otherwise returns false. Serial port is sequential device
    so this function always returns true. Check QIODevice::isSequential() documentation for more
    information.
*/
bool QextSerialPort::isSequential() const
{
	return true;
}

/*!
    Return the error number, or 0 if no error occurred.
*/
ulong QextSerialPort::lastError() const
{
	QReadLocker locker(&d_func()->lock);
	return d_func()->lastErr;
}

/*!
    Returns the line status as stored by the port function.  This function will retrieve the states
    of the following lines: DCD, CTS, DSR, and RI.  On POSIX systems, the following additional lines
    can be monitored: DTR, RTS, Secondary TXD, and Secondary RXD.  The value returned is an unsigned
    long with specific bits indicating which lines are high.  The following constants should be used
    to examine the states of individual lines:

    \code
    Mask        Line
    ------      ----
    LS_CTS      CTS
    LS_DSR      DSR
    LS_DCD      DCD
    LS_RI       RI
    LS_RTS      RTS (POSIX only)
    LS_DTR      DTR (POSIX only)
    LS_ST       Secondary TXD (POSIX only)
    LS_SR       Secondary RXD (POSIX only)
    \endcode

    This function will return 0 if the port associated with the class is not currently open.
*/
unsigned long QextSerialPort::lineStatus()
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (isOpen()) {
		return d->lineStatus_sys();
	}

	return 0;
}

/*!
  Returns a human-readable description of the last device error that occurred.
*/
QString QextSerialPort::errorString()
{
	Q_D(QextSerialPort);
	QReadLocker locker(&d->lock);

	switch (d->lastErr) {
		case E_NO_ERROR:
			return tr("No Error has occurred");

		case E_INVALID_FD:
			return tr("Invalid file descriptor (port was not opened correctly)");

		case E_NO_MEMORY:
			return tr("Unable to allocate memory tables (POSIX)");

		case E_CAUGHT_NON_BLOCKED_SIGNAL:
			return tr("Caught a non-blocked signal (POSIX)");

		case E_PORT_TIMEOUT:
			return tr("Operation timed out (POSIX)");

		case E_INVALID_DEVICE:
			return tr("The file opened by the port is not a valid device");

		case E_BREAK_CONDITION:
			return tr("The port detected a break condition");

		case E_FRAMING_ERROR:
			return tr("The port detected a framing error (usually caused by incorrect baud rate settings)");

		case E_IO_ERROR:
			return tr("There was an I/O error while communicating with the port");

		case E_BUFFER_OVERRUN:
			return tr("Character buffer overrun");

		case E_RECEIVE_OVERFLOW:
			return tr("Receive buffer overflow");

		case E_RECEIVE_PARITY_ERROR:
			return tr("The port detected a parity error in the received data");

		case E_TRANSMIT_OVERFLOW:
			return tr("Transmit buffer overflow");

		case E_READ_FAILED:
			return tr("General read operation failure");

		case E_WRITE_FAILED:
			return tr("General write operation failure");

		case E_FILE_NOT_FOUND:
			return tr("The %1 file doesn't exists").arg(this->portName());

		case E_PERMISSION_DENIED:
			return tr("Permission denied");

		case E_AGAIN:
			return tr("Device is already locked");

		default:
			return tr("Unknown error: %1").arg(d->lastErr);
	}
}

/*!
   Destructs the QextSerialPort object.
*/
QextSerialPort::~QextSerialPort()
{
	if (isOpen()) {
		close();
	}

	delete d_ptr;
}

/*!
    Sets the flow control used by the port to \a flow.  Possible values of flow are:
    \code
        FLOW_OFF            No flow control
        FLOW_HARDWARE       Hardware (RTS/CTS) flow control
        FLOW_XONXOFF        Software (XON/XOFF) flow control
    \endcode
*/
void QextSerialPort::setFlowControl(FlowType flow)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (d->settings.FlowControl != flow) {
		d->setFlowControl(flow, true);
	}
}

/*!
    Sets the parity associated with the serial port to \a parity.  The possible values of parity are:
    \code
        PAR_SPACE       Space Parity
        PAR_MARK        Mark Parity
        PAR_NONE        No Parity
        PAR_EVEN        Even Parity
        PAR_ODD         Odd Parity
    \endcode
*/
void QextSerialPort::setParity(ParityType parity)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (d->settings.Parity != parity) {
		d->setParity(parity, true);
	}
}

/*!
    Sets the number of data bits used by the serial port to \a dataBits.  Possible values of dataBits are:
    \code
        DATA_5      5 data bits
        DATA_6      6 data bits
        DATA_7      7 data bits
        DATA_8      8 data bits
    \endcode

    \bold note:
    This function is subject to the following restrictions:
    \list
    \o 5 data bits cannot be used with 2 stop bits.
    \o 1.5 stop bits can only be used with 5 data bits.
    \o 8 data bits cannot be used with space parity on POSIX systems.
    \endlist
    */
void QextSerialPort::setDataBits(DataBitsType dataBits)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (d->settings.DataBits != dataBits) {
		d->setDataBits(dataBits, true);
	}
}

/*!
    Sets the number of stop bits used by the serial port to \a stopBits.  Possible values of stopBits are:
    \code
        STOP_1      1 stop bit
        STOP_1_5    1.5 stop bits
        STOP_2      2 stop bits
    \endcode

    \bold note:
    This function is subject to the following restrictions:
    \list
    \o 2 stop bits cannot be used with 5 data bits.
    \o 1.5 stop bits cannot be used with 6 or more data bits.
    \o POSIX does not support 1.5 stop bits.
    \endlist
*/
void QextSerialPort::setStopBits(StopBitsType stopBits)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (d->settings.StopBits != stopBits) {
		d->setStopBits(stopBits, true);
	}
}

/*!
    Sets the baud rate of the serial port to \a baudRate.  Note that not all rates are applicable on
    all platforms.  The following table shows translations of the various baud rate
    constants on Windows(including NT/2000) and POSIX platforms.  Speeds marked with an *
    are speeds that are usable on both Windows and POSIX.
    \code

      RATE          Windows Speed   POSIX Speed
      -----------   -------------   -----------
       BAUD50                   X          50
       BAUD75                   X          75
      *BAUD110                110         110
       BAUD134                  X         134.5
       BAUD150                  X         150
       BAUD200                  X         200
      *BAUD300                300         300
      *BAUD600                600         600
      *BAUD1200              1200        1200
       BAUD1800                 X        1800
      *BAUD2400              2400        2400
      *BAUD4800              4800        4800
      *BAUD9600              9600        9600
       BAUD14400            14400           X
      *BAUD19200            19200       19200
      *BAUD38400            38400       38400
       BAUD56000            56000           X
      *BAUD57600            57600       57600
       BAUD76800                X       76800
      *BAUD115200          115200      115200
       BAUD128000          128000           X
       BAUD230400               X      230400
       BAUD256000          256000           X
       BAUD460800               X      460800
       BAUD500000               X      500000
       BAUD576000               X      576000
       BAUD921600               X      921600
       BAUD1000000              X     1000000
       BAUD1152000              X     1152000
       BAUD1500000              X     1500000
       BAUD2000000              X     2000000
       BAUD2500000              X     2500000
       BAUD3000000              X     3000000
       BAUD3500000              X     3500000
       BAUD4000000              X     4000000
    \endcode
*/

void QextSerialPort::setBaudRate(BaudRateType baudRate)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (d->settings.BaudRate != baudRate) {
		d->setBaudRate(baudRate, true);
	}
}

/*!
    For Unix:

    Sets the read and write timeouts for the port to \a millisec milliseconds.
    Note that this is a per-character timeout, i.e. the port will wait this long for each
    individual character, not for the whole read operation.  This timeout also applies to the
    bytesWaiting() function.

    \bold note:
    POSIX does not support millisecond-level control for I/O timeout values.  Any
    timeout set using this function will be set to the next lowest tenth of a second for
    the purposes of detecting read or write timeouts.  For example a timeout of 550 milliseconds
    will be seen by the class as a timeout of 500 milliseconds for the purposes of reading and
    writing the port.  However millisecond-level control is allowed by the select() system call,
    so for example a 550-millisecond timeout will be seen as 550 milliseconds on POSIX systems for
    the purpose of detecting available bytes in the read buffer.

    For Windows:

    Sets the read and write timeouts for the port to \a millisec milliseconds.
    Setting 0 indicates that timeouts are not used for read nor write operations;
    however read() and write() functions will still block. Set -1 to provide
    non-blocking behaviour (read() and write() will return immediately).

    \bold note: this function does nothing in event driven mode.
*/
void QextSerialPort::setTimeout(long millisec)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (d->settings.Timeout_Millisec != millisec) {
		d->setTimeout(millisec, true);
	}
}

/*!
    Sets DTR line to the requested state (\a set default to high).  This function will have no effect if
    the port associated with the class is not currently open.
*/
void QextSerialPort::setDtr(bool set)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (isOpen()) {
		d->setDtr_sys(set);
	}
}

/*!
    Sets RTS line to the requested state \a set (high by default).
    This function will have no effect if
    the port associated with the class is not currently open.
*/
void QextSerialPort::setRts(bool set)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);

	if (isOpen()) {
		d->setRts_sys(set);
	}
}

/*! \reimp
    Reads a block of data from the serial port.  This function will read at most maxlen bytes from
    the serial port and place them in the buffer pointed to by data.  Return value is the number of
    bytes actually read, or -1 on error.

    \warning before calling this function ensure that serial port associated with this class
    is currently open (use isOpen() function to check if port is open).
*/
qint64 QextSerialPort::readData(char *data, qint64 maxSize)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);
	qint64 bytesFromBuffer = 0;

	if (!d->readBuffer.isEmpty()) {
		bytesFromBuffer = d->readBuffer.read(data, maxSize);

		if (bytesFromBuffer == maxSize) {
			return bytesFromBuffer;
		}
	}

	qint64 bytesFromDevice = d->readData_sys(data + bytesFromBuffer, maxSize - bytesFromBuffer);

	if (bytesFromDevice < 0) {
		return -1;
	}

	return bytesFromBuffer + bytesFromDevice;
}

/*! \reimp
    Writes a block of data to the serial port.  This function will write len bytes
    from the buffer pointed to by data to the serial port.  Return value is the number
    of bytes actually written, or -1 on error.

    \warning before calling this function ensure that serial port associated with this class
    is currently open (use isOpen() function to check if port is open).
*/
qint64 QextSerialPort::writeData(const char *data, qint64 maxSize)
{
	Q_D(QextSerialPort);
	QWriteLocker locker(&d->lock);
	return d->writeData_sys(data, maxSize);
}

#include "moc_qextserialport.cpp"
```

### `./qextserialport.h`

```cpp
﻿/****************************************************************************
** Copyright (c) 2000-2003 Wayne Roth
** Copyright (c) 2004-2007 Stefan Sander
** Copyright (c) 2007 Michal Policht
** Copyright (c) 2008 Brandon Fosdick
** Copyright (c) 2009-2010 Liam Staskawicz
** Copyright (c) 2011 Debao Zhang
** All right reserved.
** Web: http://code.google.com/p/qextserialport/
**
** Permission is hereby granted, free of charge, to any person obtaining
** a copy of this software and associated documentation files (the
** "Software"), to deal in the Software without restriction, including
** without limitation the rights to use, copy, modify, merge, publish,
** distribute, sublicense, and/or sell copies of the Software, and to
** permit persons to whom the Software is furnished to do so, subject to
** the following conditions:
**
** The above copyright notice and this permission notice shall be
** included in all copies or substantial portions of the Software.
**
** THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
** EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
** MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
** NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
** LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
** OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
** WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
**
****************************************************************************/

#ifndef _QEXTSERIALPORT_H_
#define _QEXTSERIALPORT_H_

#include <QtCore/QIODevice>
#include "qextserialport_global.h"
#ifdef Q_OS_UNIX
#include <termios.h>
#endif
/*line status constants*/
// ### QESP2.0 move to enum
#define LS_CTS  0x01
#define LS_DSR  0x02
#define LS_DCD  0x04
#define LS_RI   0x08
#define LS_RTS  0x10
#define LS_DTR  0x20
#define LS_ST   0x40
#define LS_SR   0x80

/*error constants*/
// ### QESP2.0 move to enum
#define E_NO_ERROR                   0
#define E_INVALID_FD                 1
#define E_NO_MEMORY                  2
#define E_CAUGHT_NON_BLOCKED_SIGNAL  3
#define E_PORT_TIMEOUT               4
#define E_INVALID_DEVICE             5
#define E_BREAK_CONDITION            6
#define E_FRAMING_ERROR              7
#define E_IO_ERROR                   8
#define E_BUFFER_OVERRUN             9
#define E_RECEIVE_OVERFLOW          10
#define E_RECEIVE_PARITY_ERROR      11
#define E_TRANSMIT_OVERFLOW         12
#define E_READ_FAILED               13
#define E_WRITE_FAILED              14
#define E_FILE_NOT_FOUND            15
#define E_PERMISSION_DENIED         16
#define E_AGAIN                     17

enum BaudRateType {
#if defined(Q_OS_UNIX) || defined(qdoc)
	BAUD50 = 50,                //POSIX ONLY
	BAUD75 = 75,                //POSIX ONLY
	BAUD134 = 134,              //POSIX ONLY
	BAUD150 = 150,              //POSIX ONLY
	BAUD200 = 200,              //POSIX ONLY
	BAUD1800 = 1800,            //POSIX ONLY
#  if defined(B76800) || defined(qdoc)
	BAUD76800 = 76800,          //POSIX ONLY
#  endif
#  if (defined(B230400) && defined(B4000000)) || defined(qdoc)
	BAUD230400 = 230400,        //POSIX ONLY
	BAUD460800 = 460800,        //POSIX ONLY
	BAUD500000 = 500000,        //POSIX ONLY
	BAUD576000 = 576000,        //POSIX ONLY
	BAUD921600 = 921600,        //POSIX ONLY
	BAUD1000000 = 1000000,      //POSIX ONLY
	BAUD1152000 = 1152000,      //POSIX ONLY
	BAUD1500000 = 1500000,      //POSIX ONLY
	BAUD2000000 = 2000000,      //POSIX ONLY
	BAUD2500000 = 2500000,      //POSIX ONLY
	BAUD3000000 = 3000000,      //POSIX ONLY
	BAUD3500000 = 3500000,      //POSIX ONLY
	BAUD4000000 = 4000000,      //POSIX ONLY
#  endif
#endif //Q_OS_UNIX
#if defined(Q_OS_WIN) || defined(qdoc)
	BAUD14400 = 14400,          //WINDOWS ONLY
	BAUD56000 = 56000,          //WINDOWS ONLY
	BAUD128000 = 128000,        //WINDOWS ONLY
	BAUD256000 = 256000,        //WINDOWS ONLY
#endif  //Q_OS_WIN
	BAUD110 = 110,
	BAUD300 = 300,
	BAUD600 = 600,
	BAUD1200 = 1200,
	BAUD2400 = 2400,
	BAUD4800 = 4800,
	BAUD9600 = 9600,
	BAUD19200 = 19200,
	BAUD38400 = 38400,
	BAUD57600 = 57600,
	BAUD115200 = 115200
};

enum DataBitsType {
	DATA_5 = 5,
	DATA_6 = 6,
	DATA_7 = 7,
	DATA_8 = 8
};

enum ParityType {
	PAR_NONE,
	PAR_ODD,
	PAR_EVEN,
#if defined(Q_OS_WIN) || defined(qdoc)
	PAR_MARK,               //WINDOWS ONLY
#endif
	PAR_SPACE
};

enum StopBitsType {
	STOP_1,
#if defined(Q_OS_WIN) || defined(qdoc)
	STOP_1_5,               //WINDOWS ONLY
#endif
	STOP_2
};

enum FlowType {
	FLOW_OFF,
	FLOW_HARDWARE,
	FLOW_XONXOFF
};

/**
 * structure to contain port settings
 */
struct PortSettings {
	BaudRateType BaudRate;
	DataBitsType DataBits;
	ParityType Parity;
	StopBitsType StopBits;
	FlowType FlowControl;
	long Timeout_Millisec;
};

class QextSerialPortPrivate;
class QEXTSERIALPORT_EXPORT QextSerialPort: public QIODevice
{
	Q_OBJECT
	Q_DECLARE_PRIVATE(QextSerialPort)
	Q_ENUMS(QueryMode)
	Q_PROPERTY(QString portName READ portName WRITE setPortName)
	Q_PROPERTY(QueryMode queryMode READ queryMode WRITE setQueryMode)
public:
	enum QueryMode {
		Polling,
		EventDriven
	};

	explicit QextSerialPort(QueryMode mode = EventDriven, QObject *parent = 0);
	explicit QextSerialPort(const QString &name, QueryMode mode = EventDriven, QObject *parent = 0);
	explicit QextSerialPort(const PortSettings &s, QueryMode mode = EventDriven, QObject *parent = 0);
	QextSerialPort(const QString &name, const PortSettings &s, QueryMode mode = EventDriven, QObject *parent = 0);

	~QextSerialPort();

	QString portName() const;
	QueryMode queryMode() const;
	BaudRateType baudRate() const;
	DataBitsType dataBits() const;
	ParityType parity() const;
	StopBitsType stopBits() const;
	FlowType flowControl() const;

	bool open(OpenMode mode);
	bool isSequential() const;
	void close();
	void flush();
	qint64 bytesAvailable() const;
	bool canReadLine() const;
	QByteArray readAll();

	ulong lastError() const;

	ulong lineStatus();
	QString errorString();

public Q_SLOTS:
	void setPortName(const QString &name);
	void setQueryMode(QueryMode mode);
	void setBaudRate(BaudRateType);
	void setDataBits(DataBitsType);
	void setParity(ParityType);
	void setStopBits(StopBitsType);
	void setFlowControl(FlowType);
	void setTimeout(long);

	void setDtr(bool set = true);
	void setRts(bool set = true);

Q_SIGNALS:
	void dsrChanged(bool status);

protected:
	qint64 readData(char *data, qint64 maxSize);
	qint64 writeData(const char *data, qint64 maxSize);

private:
	Q_DISABLE_COPY(QextSerialPort)

#ifdef Q_OS_WIN
	Q_PRIVATE_SLOT(d_func(), void _q_onWinEvent(HANDLE))
#endif
	Q_PRIVATE_SLOT(d_func(), void _q_canRead())

	QextSerialPortPrivate *const d_ptr;
};

#endif
```

### `./qextserialport_global.h`

```cpp
﻿/****************************************************************************
** Copyright (c) 2000-2003 Wayne Roth
** Copyright (c) 2004-2007 Stefan Sander
** Copyright (c) 2007 Michal Policht
** Copyright (c) 2008 Brandon Fosdick
** Copyright (c) 2009-2010 Liam Staskawicz
** Copyright (c) 2011 Debao Zhang
** All right reserved.
** Web: http://code.google.com/p/qextserialport/
**
** Permission is hereby granted, free of charge, to any person obtaining
** a copy of this software and associated documentation files (the
** "Software"), to deal in the Software without restriction, including
** without limitation the rights to use, copy, modify, merge, publish,
** distribute, sublicense, and/or sell copies of the Software, and to
** permit persons to whom the Software is furnished to do so, subject to
** the following conditions:
**
** The above copyright notice and this permission notice shall be
** included in all copies or substantial portions of the Software.
**
** THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
** EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
** MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
** NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
** LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
** OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
** WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
**
****************************************************************************/

#ifndef QEXTSERIALPORT_GLOBAL_H
#define QEXTSERIALPORT_GLOBAL_H

#include <QtCore/QtGlobal>

#ifdef QEXTSERIALPORT_BUILD_SHARED
#  define QEXTSERIALPORT_EXPORT Q_DECL_EXPORT
#elif defined(QEXTSERIALPORT_USING_SHARED)
#  define QEXTSERIALPORT_EXPORT Q_DECL_IMPORT
#else
#  define QEXTSERIALPORT_EXPORT
#endif

// ### for compatible with old version. should be removed in QESP 2.0
#ifdef _TTY_NOWARN_
#  define QESP_NO_WARN
#endif
#ifdef _TTY_NOWARN_PORT_
#  define QESP_NO_PORTABILITY_WARN
#endif

/*if all warning messages are turned off, flag portability warnings to be turned off as well*/
#ifdef QESP_NO_WARN
#  define QESP_NO_PORTABILITY_WARN
#endif

/*macros for warning and debug messages*/
#ifdef QESP_NO_PORTABILITY_WARN
#  define QESP_PORTABILITY_WARNING  while (false)qWarning
#else
#  define QESP_PORTABILITY_WARNING qWarning
#endif /*QESP_NOWARN_PORT*/

#ifdef QESP_NO_WARN
#  define QESP_WARNING while (false)qWarning
#else
#  define QESP_WARNING qWarning
#endif /*QESP_NOWARN*/

#endif // QEXTSERIALPORT_GLOBAL_H
```

### `./qextserialport_p.h`

```cpp
﻿/****************************************************************************
** Copyright (c) 2000-2003 Wayne Roth
** Copyright (c) 2004-2007 Stefan Sander
** Copyright (c) 2007 Michal Policht
** Copyright (c) 2008 Brandon Fosdick
** Copyright (c) 2009-2010 Liam Staskawicz
** Copyright (c) 2011 Debao Zhang
** All right reserved.
** Web: http://code.google.com/p/qextserialport/
**
** Permission is hereby granted, free of charge, to any person obtaining
** a copy of this software and associated documentation files (the
** "Software"), to deal in the Software without restriction, including
** without limitation the rights to use, copy, modify, merge, publish,
** distribute, sublicense, and/or sell copies of the Software, and to
** permit persons to whom the Software is furnished to do so, subject to
** the following conditions:
**
** The above copyright notice and this permission notice shall be
** included in all copies or substantial portions of the Software.
**
** THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
** EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
** MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
** NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
** LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
** OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
** WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
**
****************************************************************************/

#ifndef _QEXTSERIALPORT_P_H_
#define _QEXTSERIALPORT_P_H_

//
//  W A R N I N G
//  -------------
//
// This file is not part of the QESP API.  It exists for the convenience
// of other QESP classes.  This header file may change from version to
// version without notice, or even be removed.
//
// We mean it.
//

#include "qextserialport.h"
#include <QtCore/QReadWriteLock>
#ifdef Q_OS_UNIX
#  include <termios.h>
#elif (defined Q_OS_WIN)
#  include <QtCore/qt_windows.h>
#endif
#include <stdlib.h>

// This is QextSerialPort's read buffer, needed by posix system.
// ref: QRingBuffer & QIODevicePrivateLinearBuffer
class QextReadBuffer
{
public:
	inline QextReadBuffer(size_t growth = 4096)
		: len(0), first(0), buf(0), capacity(0), basicBlockSize(growth)
	{
	}

	~QextReadBuffer()
	{
		delete  buf;
	}

	inline void clear()
	{
		first = buf;
		len = 0;
	}

	inline int size() const
	{
		return len;
	}

	inline bool isEmpty() const
	{
		return len == 0;
	}

	inline int read(char *target, int size)
	{
		int r = qMin(size, len);

		if (r == 1) {
			*target = *first;
			--len;
			++first;
		} else {
			memcpy(target, first, r);
			len -= r;
			first += r;
		}

		return r;
	}

	inline char *reserve(size_t size)
	{
		if ((first - buf) + len + size > capacity) {
			size_t newCapacity = qMax(capacity, basicBlockSize);

			while (newCapacity < len + size) {
				newCapacity *= 2;
			}

			if (newCapacity > capacity) {
				// allocate more space
				char *newBuf = new char[newCapacity];
				memmove(newBuf, first, len);
				delete  buf;
				buf = newBuf;
				capacity = newCapacity;
			} else {
				// shift any existing data to make space
				memmove(buf, first, len);
			}

			first = buf;
		}

		char *writePtr = first + len;
		len += (int)size;
		return writePtr;
	}

	inline void chop(int size)
	{
		if (size >= len) {
			clear();
		} else {
			len -= size;
		}
	}

	inline void squeeze()
	{
		if (first != buf) {
			memmove(buf, first, len);
			first = buf;
		}

		size_t newCapacity = basicBlockSize;

		while (newCapacity < size_t(len)) {
			newCapacity *= 2;
		}

		if (newCapacity < capacity) {
			char *tmp = static_cast<char *>(realloc(buf, newCapacity));

			if (tmp) {
				buf = tmp;
				capacity = newCapacity;
			}
		}
	}

	inline QByteArray readAll()
	{
		char *f = first;
		int l = len;
		clear();
		return QByteArray(f, l);
	}

	inline int readLine(char *target, int size)
	{
		int r = qMin(size, len);
		char *eol = static_cast<char *>(memchr(first, '\n', r));

		if (eol) {
			r = 1 + (eol - first);
		}

		memcpy(target, first, r);
		len -= r;
		first += r;
		return int(r);
	}

	inline bool canReadLine() const
	{
		return memchr(first, '\n', len);
	}

private:
	int len;
	char *first;
	char *buf;
	size_t capacity;
	size_t basicBlockSize;
};

class QWinEventNotifier;
class QReadWriteLock;
class QSocketNotifier;

class QextSerialPortPrivate
{
	Q_DECLARE_PUBLIC(QextSerialPort)
public:
	QextSerialPortPrivate(QextSerialPort *q);
	~QextSerialPortPrivate();
	enum DirtyFlagEnum {
		DFE_BaudRate = 0x0001,
		DFE_Parity = 0x0002,
		DFE_StopBits = 0x0004,
		DFE_DataBits = 0x0008,
		DFE_Flow = 0x0010,
		DFE_TimeOut = 0x0100,
		DFE_ALL = 0x0fff,
		DFE_Settings_Mask = 0x00ff //without TimeOut
	};
	mutable QReadWriteLock lock;
	QString port;
	PortSettings settings;
	QextReadBuffer readBuffer;
	int settingsDirtyFlags;
	ulong lastErr;
	QextSerialPort::QueryMode queryMode;

	// platform specific members
#ifdef Q_OS_UNIX
	int fd;
	QSocketNotifier *readNotifier;
	struct termios currentTermios;
	struct termios oldTermios;
#elif (defined Q_OS_WIN)
	HANDLE handle;
	OVERLAPPED overlap;
	COMMCONFIG commConfig;
	COMMTIMEOUTS commTimeouts;
	QWinEventNotifier *winEventNotifier;
	DWORD eventMask;
	QList<OVERLAPPED *> pendingWrites;
	QReadWriteLock *bytesToWriteLock;
#endif

	/*fill PortSettings*/
	void setBaudRate(BaudRateType baudRate, bool update = true);
	void setDataBits(DataBitsType dataBits, bool update = true);
	void setParity(ParityType parity, bool update = true);
	void setStopBits(StopBitsType stopbits, bool update = true);
	void setFlowControl(FlowType flow, bool update = true);
	void setTimeout(long millisec, bool update = true);
	void setPortSettings(const PortSettings &settings, bool update = true);

	void platformSpecificDestruct();
	void platformSpecificInit();
	void translateError(ulong error);
	void updatePortSettings();

	qint64 readData_sys(char *data, qint64 maxSize);
	qint64 writeData_sys(const char *data, qint64 maxSize);
	void setDtr_sys(bool set = true);
	void setRts_sys(bool set = true);
	bool open_sys(QIODevice::OpenMode mode);
	bool close_sys();
	bool flush_sys();
	ulong lineStatus_sys();
	qint64 bytesAvailable_sys() const;

#ifdef Q_OS_WIN
	void _q_onWinEvent(HANDLE h);
#endif
	void _q_canRead();

	QextSerialPort *q_ptr;
};

#endif //_QEXTSERIALPORT_P_H_
```

### `./qextserialport_unix.cpp`

```cpp
﻿/****************************************************************************
** Copyright (c) 2000-2003 Wayne Roth
** Copyright (c) 2004-2007 Stefan Sander
** Copyright (c) 2007 Michal Policht
** Copyright (c) 2008 Brandon Fosdick
** Copyright (c) 2009-2010 Liam Staskawicz
** Copyright (c) 2011 Debao Zhang
** All right reserved.
** Web: http://code.google.com/p/qextserialport/
**
** Permission is hereby granted, free of charge, to any person obtaining
** a copy of this software and associated documentation files (the
** "Software"), to deal in the Software without restriction, including
** without limitation the rights to use, copy, modify, merge, publish,
** distribute, sublicense, and/or sell copies of the Software, and to
** permit persons to whom the Software is furnished to do so, subject to
** the following conditions:
**
** The above copyright notice and this permission notice shall be
** included in all copies or substantial portions of the Software.
**
** THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
** EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
** MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
** NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
** LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
** OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
** WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
**
****************************************************************************/

#include "qextserialport.h"
#include "qextserialport_p.h"
#include <fcntl.h>
#include <stdio.h>
#include <errno.h>
#include <unistd.h>
#include <sys/time.h>
#include <sys/ioctl.h>
#include <sys/select.h>
#include <QtCore/QMutexLocker>
#include <QtCore/QDebug>
#include <QtCore/QSocketNotifier>

void QextSerialPortPrivate::platformSpecificInit()
{
	fd = 0;
	readNotifier = 0;
}

/*!
    Standard destructor.
*/
void QextSerialPortPrivate::platformSpecificDestruct()
{
}

static QString fullPortName(const QString &name)
{
	if (name.startsWith(QLatin1Char('/'))) {
		return name;
	}

	return QLatin1String("/dev/") + name;
}

bool QextSerialPortPrivate::open_sys(QIODevice::OpenMode mode)
{
	Q_Q(QextSerialPort);

	//note: linux 2.6.21 seems to ignore O_NDELAY flag
	if ((fd = ::open(fullPortName(port).toLatin1() , O_RDWR | O_NOCTTY | O_NDELAY)) != -1) {

		/*In the Private class, We can not call QIODevice::open()*/
		q->setOpenMode(mode);             // Flag the port as opened
		::tcgetattr(fd, &oldTermios);    // Save the old termios
		currentTermios = oldTermios;   // Make a working copy
		::cfmakeraw(&currentTermios);   // Enable raw access

		/*set up other port settings*/
		currentTermios.c_cflag |= CREAD | CLOCAL;
		currentTermios.c_lflag &= (~(ICANON | ECHO | ECHOE | ECHOK | ECHONL | ISIG));
		currentTermios.c_iflag &= (~(INPCK | IGNPAR | PARMRK | ISTRIP | ICRNL | IXANY));
		currentTermios.c_oflag &= (~OPOST);
		currentTermios.c_cc[VMIN] = 0;
#ifdef _POSIX_VDISABLE  // Is a disable character available on this system?
		// Some systems allow for per-device disable-characters, so get the
		//  proper value for the configured device
		const long vdisable = ::fpathconf(fd, _PC_VDISABLE);
		currentTermios.c_cc[VINTR] = vdisable;
		currentTermios.c_cc[VQUIT] = vdisable;
		currentTermios.c_cc[VSTART] = vdisable;
		currentTermios.c_cc[VSTOP] = vdisable;
		currentTermios.c_cc[VSUSP] = vdisable;
#endif //_POSIX_VDISABLE
		settingsDirtyFlags = DFE_ALL;
		updatePortSettings();

		if (queryMode == QextSerialPort::EventDriven) {
			readNotifier = new QSocketNotifier(fd, QSocketNotifier::Read, q);
			q->connect(readNotifier, SIGNAL(activated(int)), q, SLOT(_q_canRead()));
		}

		return true;
	} else {
		translateError(errno);
		return false;
	}
}

bool QextSerialPortPrivate::close_sys()
{
	// Force a flush and then restore the original termios
	flush_sys();
	// Using both TCSAFLUSH and TCSANOW here discards any pending input
	::tcsetattr(fd, TCSAFLUSH | TCSANOW, &oldTermios);   // Restore termios
	::close(fd);

	if (readNotifier) {
		delete readNotifier;
		readNotifier = 0;
	}

	return true;
}

bool QextSerialPortPrivate::flush_sys()
{
	::tcdrain(fd);
	return true;
}

qint64 QextSerialPortPrivate::bytesAvailable_sys() const
{
	int bytesQueued;

	if (::ioctl(fd, FIONREAD, &bytesQueued) == -1) {
		return (qint64) - 1;
	}

	return bytesQueued;
}

/*!
    Translates a system-specific error code to a QextSerialPort error code.  Used internally.
*/
void QextSerialPortPrivate::translateError(ulong error)
{
	switch (error) {
		case EBADF:
		case ENOTTY:
			lastErr = E_INVALID_FD;
			break;

		case EINTR:
			lastErr = E_CAUGHT_NON_BLOCKED_SIGNAL;
			break;

		case ENOMEM:
			lastErr = E_NO_MEMORY;
			break;

		case EACCES:
			lastErr = E_PERMISSION_DENIED;
			break;

		case EAGAIN:
			lastErr = E_AGAIN;
			break;
	}
}

void QextSerialPortPrivate::setDtr_sys(bool set)
{
	int status;
	::ioctl(fd, TIOCMGET, &status);

	if (set) {
		status |= TIOCM_DTR;
	} else {
		status &= ~TIOCM_DTR;
	}

	::ioctl(fd, TIOCMSET, &status);
}

void QextSerialPortPrivate::setRts_sys(bool set)
{
	int status;
	::ioctl(fd, TIOCMGET, &status);

	if (set) {
		status |= TIOCM_RTS;
	} else {
		status &= ~TIOCM_RTS;
	}

	::ioctl(fd, TIOCMSET, &status);
}

unsigned long QextSerialPortPrivate::lineStatus_sys()
{
	unsigned long Status = 0, Temp = 0;
	::ioctl(fd, TIOCMGET, &Temp);

	if (Temp & TIOCM_CTS) {
		Status |= LS_CTS;
	}

	if (Temp & TIOCM_DSR) {
		Status |= LS_DSR;
	}

	if (Temp & TIOCM_RI) {
		Status |= LS_RI;
	}

	if (Temp & TIOCM_CD) {
		Status |= LS_DCD;
	}

	if (Temp & TIOCM_DTR) {
		Status |= LS_DTR;
	}

	if (Temp & TIOCM_RTS) {
		Status |= LS_RTS;
	}

	if (Temp & TIOCM_ST) {
		Status |= LS_ST;
	}

	if (Temp & TIOCM_SR) {
		Status |= LS_SR;
	}

	return Status;
}

/*!
    Reads a block of data from the serial port.  This function will read at most maxSize bytes from
    the serial port and place them in the buffer pointed to by data.  Return value is the number of
    bytes actually read, or -1 on error.

    \warning before calling this function ensure that serial port associated with this class
    is currently open (use isOpen() function to check if port is open).
*/
qint64 QextSerialPortPrivate::readData_sys(char *data, qint64 maxSize)
{
	int retVal = ::read(fd, data, maxSize);

	if (retVal == -1) {
		lastErr = E_READ_FAILED;
	}

	return retVal;
}

/*!
    Writes a block of data to the serial port.  This function will write maxSize bytes
    from the buffer pointed to by data to the serial port.  Return value is the number
    of bytes actually written, or -1 on error.

    \warning before calling this function ensure that serial port associated with this class
    is currently open (use isOpen() function to check if port is open).
*/
qint64 QextSerialPortPrivate::writeData_sys(const char *data, qint64 maxSize)
{
	int retVal = ::write(fd, data, maxSize);

	if (retVal == -1) {
		lastErr = E_WRITE_FAILED;
	}

	return (qint64)retVal;
}

static void setBaudRate2Termios(termios *config, int baudRate)
{
#ifdef CBAUD
	config->c_cflag &= (~CBAUD);
	config->c_cflag |= baudRate;
#else
	::cfsetispeed(config, baudRate);
	::cfsetospeed(config, baudRate);
#endif
}

/*
    All the platform settings was performed in this function.
*/
void QextSerialPortPrivate::updatePortSettings()
{
	if (!q_func()->isOpen() || !settingsDirtyFlags) {
		return;
	}

	if (settingsDirtyFlags & DFE_BaudRate) {
		switch (settings.BaudRate) {
			case BAUD50:
				setBaudRate2Termios(&currentTermios, B50);
				break;

			case BAUD75:
				setBaudRate2Termios(&currentTermios, B75);
				break;

			case BAUD110:
				setBaudRate2Termios(&currentTermios, B110);
				break;

			case BAUD134:
				setBaudRate2Termios(&currentTermios, B134);
				break;

			case BAUD150:
				setBaudRate2Termios(&currentTermios, B150);
				break;

			case BAUD200:
				setBaudRate2Termios(&currentTermios, B200);
				break;

			case BAUD300:
				setBaudRate2Termios(&currentTermios, B300);
				break;

			case BAUD600:
				setBaudRate2Termios(&currentTermios, B600);
				break;

			case BAUD1200:
				setBaudRate2Termios(&currentTermios, B1200);
				break;

			case BAUD1800:
				setBaudRate2Termios(&currentTermios, B1800);
				break;

			case BAUD2400:
				setBaudRate2Termios(&currentTermios, B2400);
				break;

			case BAUD4800:
				setBaudRate2Termios(&currentTermios, B4800);
				break;

			case BAUD9600:
				setBaudRate2Termios(&currentTermios, B9600);
				break;

			case BAUD19200:
				setBaudRate2Termios(&currentTermios, B19200);
				break;

			case BAUD38400:
				setBaudRate2Termios(&currentTermios, B38400);
				break;

			case BAUD57600:
				setBaudRate2Termios(&currentTermios, B57600);
				break;
#ifdef B76800

			case BAUD76800:
				setBaudRate2Termios(&currentTermios, B76800);
				break;
#endif

			case BAUD115200:
				setBaudRate2Termios(&currentTermios, B115200);
				break;
#if defined(B230400) && defined(B4000000)

			case BAUD230400:
				setBaudRate2Termios(&currentTermios, B230400);
				break;

			case BAUD460800:
				setBaudRate2Termios(&currentTermios, B460800);
				break;

			case BAUD500000:
				setBaudRate2Termios(&currentTermios, B500000);
				break;

			case BAUD576000:
				setBaudRate2Termios(&currentTermios, B576000);
				break;

			case BAUD921600:
				setBaudRate2Termios(&currentTermios, B921600);
				break;

			case BAUD1000000:
				setBaudRate2Termios(&currentTermios, B1000000);
				break;

			case BAUD1152000:
				setBaudRate2Termios(&currentTermios, B1152000);
				break;

			case BAUD1500000:
				setBaudRate2Termios(&currentTermios, B1500000);
				break;

			case BAUD2000000:
				setBaudRate2Termios(&currentTermios, B2000000);
				break;

			case BAUD2500000:
				setBaudRate2Termios(&currentTermios, B2500000);
				break;

			case BAUD3000000:
				setBaudRate2Termios(&currentTermios, B3000000);
				break;

			case BAUD3500000:
				setBaudRate2Termios(&currentTermios, B3500000);
				break;

			case BAUD4000000:
				setBaudRate2Termios(&currentTermios, B4000000);
				break;
#endif
#ifdef Q_OS_MAC

			default:
				setBaudRate2Termios(&currentTermios, settings.BaudRate);
				break;
#endif
		}
	}

	if (settingsDirtyFlags & DFE_Parity) {
		switch (settings.Parity) {
			case PAR_SPACE:
				/*space parity not directly supported - add an extra data bit to simulate it*/
				settingsDirtyFlags |= DFE_DataBits;
				break;

			case PAR_NONE:
				currentTermios.c_cflag &= (~PARENB);
				break;

			case PAR_EVEN:
				currentTermios.c_cflag &= (~PARODD);
				currentTermios.c_cflag |= PARENB;
				break;

			case PAR_ODD:
				currentTermios.c_cflag |= (PARENB | PARODD);
				break;
		}
	}

	/*must after Parity settings*/
	if (settingsDirtyFlags & DFE_DataBits) {
		if (settings.Parity != PAR_SPACE) {
			currentTermios.c_cflag &= (~CSIZE);

			switch (settings.DataBits) {
				case DATA_5:
					currentTermios.c_cflag |= CS5;
					break;

				case DATA_6:
					currentTermios.c_cflag |= CS6;
					break;

				case DATA_7:
					currentTermios.c_cflag |= CS7;
					break;

				case DATA_8:
					currentTermios.c_cflag |= CS8;
					break;
			}
		} else {
			/*space parity not directly supported - add an extra data bit to simulate it*/
			currentTermios.c_cflag &= ~(PARENB | CSIZE);

			switch (settings.DataBits) {
				case DATA_5:
					currentTermios.c_cflag |= CS6;
					break;

				case DATA_6:
					currentTermios.c_cflag |= CS7;
					break;

				case DATA_7:
					currentTermios.c_cflag |= CS8;
					break;

				case DATA_8:
					/*this will never happen, put here to Suppress an warning*/
					break;
			}
		}
	}

	if (settingsDirtyFlags & DFE_StopBits) {
		switch (settings.StopBits) {
			case STOP_1:
				currentTermios.c_cflag &= (~CSTOPB);
				break;

			case STOP_2:
				currentTermios.c_cflag |= CSTOPB;
				break;
		}
	}

	if (settingsDirtyFlags & DFE_Flow) {
		switch (settings.FlowControl) {
			case FLOW_OFF:
				currentTermios.c_cflag &= (~CRTSCTS);
				currentTermios.c_iflag &= (~(IXON | IXOFF | IXANY));
				break;

			case FLOW_XONXOFF:
				/*software (XON/XOFF) flow control*/
				currentTermios.c_cflag &= (~CRTSCTS);
				currentTermios.c_iflag |= (IXON | IXOFF | IXANY);
				break;

			case FLOW_HARDWARE:
				currentTermios.c_cflag |= CRTSCTS;
				currentTermios.c_iflag &= (~(IXON | IXOFF | IXANY));
				break;
		}
	}

	/*if any thing in currentTermios changed, flush*/
	if (settingsDirtyFlags & DFE_Settings_Mask) {
		::tcsetattr(fd, TCSAFLUSH, &currentTermios);
	}

	if (settingsDirtyFlags & DFE_TimeOut) {
		int millisec = settings.Timeout_Millisec;

		if (millisec == -1) {
			::fcntl(fd, F_SETFL, O_NDELAY);
		} else {
			//O_SYNC should enable blocking ::write()
			//however this seems not working on Linux 2.6.21 (works on OpenBSD 4.2)
			::fcntl(fd, F_SETFL, O_SYNC);
		}

		::tcgetattr(fd, &currentTermios);
		currentTermios.c_cc[VTIME] = millisec / 100;
		::tcsetattr(fd, TCSAFLUSH, &currentTermios);
	}

	settingsDirtyFlags = 0;
}
```

### `./qextserialport_win.cpp`

```cpp
﻿/****************************************************************************
** Copyright (c) 2000-2003 Wayne Roth
** Copyright (c) 2004-2007 Stefan Sander
** Copyright (c) 2007 Michal Policht
** Copyright (c) 2008 Brandon Fosdick
** Copyright (c) 2009-2010 Liam Staskawicz
** Copyright (c) 2011 Debao Zhang
** All right reserved.
** Web: http://code.google.com/p/qextserialport/
**
** Permission is hereby granted, free of charge, to any person obtaining
** a copy of this software and associated documentation files (the
** "Software"), to deal in the Software without restriction, including
** without limitation the rights to use, copy, modify, merge, publish,
** distribute, sublicense, and/or sell copies of the Software, and to
** permit persons to whom the Software is furnished to do so, subject to
** the following conditions:
**
** The above copyright notice and this permission notice shall be
** included in all copies or substantial portions of the Software.
**
** THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND,
** EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF
** MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND
** NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE
** LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION
** OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION
** WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.
**
****************************************************************************/

#include "qextserialport.h"
#include "qextserialport_p.h"
#include <QtCore/QThread>
#include <QtCore/QReadWriteLock>
#include <QtCore/QMutexLocker>
#include <QtCore/QDebug>
#include <QtCore/QMetaType>

#if QT_VERSION >= QT_VERSION_CHECK(5, 0, 0)
#include <QtCore/QWinEventNotifier>
#else
#include <QtCore/private/qwineventnotifier_p.h>
#endif

#if QT_VERSION >= QT_VERSION_CHECK(6, 0, 0)
#include <QtCore5Compat/QRegExp>
#else
#include <QtCore/QRegExp>
#endif

void QextSerialPortPrivate::platformSpecificInit()
{
    handle = INVALID_HANDLE_VALUE;
    ZeroMemory(&overlap, sizeof(OVERLAPPED));
    overlap.hEvent = CreateEvent(NULL, true, false, NULL);
    winEventNotifier = 0;
    bytesToWriteLock = new QReadWriteLock;
}

void QextSerialPortPrivate::platformSpecificDestruct()
{
    CloseHandle(overlap.hEvent);
    delete bytesToWriteLock;
}


/*!
    \internal
    COM ports greater than 9 need \\.\ prepended

    This is only need when open the port.
*/
static QString fullPortNameWin(const QString &name)
{
    QRegExp rx(QLatin1String("^COM(\\d+)"));
    QString fullName(name);
    if (rx.indexIn(fullName) >= 0) {
        fullName.prepend(QLatin1String("\\\\.\\"));
    }

    return fullName;
}

bool QextSerialPortPrivate::open_sys(QIODevice::OpenMode mode)
{
    Q_Q(QextSerialPort);
    DWORD confSize = sizeof(COMMCONFIG);
    commConfig.dwSize = confSize;
    DWORD dwFlagsAndAttributes = 0;

    if (queryMode == QextSerialPort::EventDriven) {
        dwFlagsAndAttributes += FILE_FLAG_OVERLAPPED;
    }

    /*open the port*/
    handle = CreateFileW((wchar_t *)fullPortNameWin(port).utf16(), GENERIC_READ | GENERIC_WRITE,
                         0, NULL, OPEN_EXISTING, dwFlagsAndAttributes, NULL);

    if (handle != INVALID_HANDLE_VALUE) {
        q->setOpenMode(mode);
        /*configure port settings*/
        GetCommConfig(handle, &commConfig, &confSize);
        GetCommState(handle, &(commConfig.dcb));

        /*set up parameters*/
        commConfig.dcb.fBinary = TRUE;
        commConfig.dcb.fInX = FALSE;
        commConfig.dcb.fOutX = FALSE;
        commConfig.dcb.fAbortOnError = FALSE;
        commConfig.dcb.fNull = FALSE;
        /* Dtr default to true. See Issue 122*/
        commConfig.dcb.fDtrControl = TRUE;
        /*flush all settings*/
        settingsDirtyFlags = DFE_ALL;
        updatePortSettings();

        //init event driven approach
        if (queryMode == QextSerialPort::EventDriven) {
            if (!SetCommMask(handle, EV_TXEMPTY | EV_RXCHAR | EV_DSR)) {
                QESP_WARNING() << "failed to set Comm Mask. Error code:" << GetLastError();
                return false;
            }

            winEventNotifier = new QWinEventNotifier(overlap.hEvent, q);
            qRegisterMetaType<HANDLE>("HANDLE");
            q->connect(winEventNotifier, SIGNAL(activated(HANDLE)), q, SLOT(_q_onWinEvent(HANDLE)), Qt::DirectConnection);
            WaitCommEvent(handle, &eventMask, &overlap);
        }

        return true;
    }

    return false;
}

bool QextSerialPortPrivate::close_sys()
{
    flush_sys();
    CancelIo(handle);

    if (CloseHandle(handle)) {
        handle = INVALID_HANDLE_VALUE;
    }

    if (winEventNotifier) {
        winEventNotifier->setEnabled(false);
        winEventNotifier->deleteLater();
        winEventNotifier = 0;
    }

    foreach (OVERLAPPED *o, pendingWrites) {
        CloseHandle(o->hEvent);
        delete o;
    }

    pendingWrites.clear();
    return true;
}

bool QextSerialPortPrivate::flush_sys()
{
    FlushFileBuffers(handle);
    return true;
}

qint64 QextSerialPortPrivate::bytesAvailable_sys() const
{
    DWORD Errors;
    COMSTAT Status;

    if (ClearCommError(handle, &Errors, &Status)) {
        return Status.cbInQue;
    }

    return (qint64) - 1;
}

/*
    Translates a system-specific error code to a QextSerialPort error code.  Used internally.
*/
void QextSerialPortPrivate::translateError(ulong error)
{
    if (error & CE_BREAK) {
        lastErr = E_BREAK_CONDITION;
    } else if (error & CE_FRAME) {
        lastErr = E_FRAMING_ERROR;
    } else if (error & CE_IOE) {
        lastErr = E_IO_ERROR;
    } else if (error & CE_MODE) {
        lastErr = E_INVALID_FD;
    } else if (error & CE_OVERRUN) {
        lastErr = E_BUFFER_OVERRUN;
    } else if (error & CE_RXPARITY) {
        lastErr = E_RECEIVE_PARITY_ERROR;
    } else if (error & CE_RXOVER) {
        lastErr = E_RECEIVE_OVERFLOW;
    } else if (error & CE_TXFULL) {
        lastErr = E_TRANSMIT_OVERFLOW;
    }
}

/*
    Reads a block of data from the serial port.  This function will read at most maxlen bytes from
    the serial port and place them in the buffer pointed to by data.  Return value is the number of
    bytes actually read, or -1 on error.

    \warning before calling this function ensure that serial port associated with this class
    is currently open (use isOpen() function to check if port is open).
*/
qint64 QextSerialPortPrivate::readData_sys(char *data, qint64 maxSize)
{
    DWORD bytesRead = 0;
    bool failed = false;

    if (queryMode == QextSerialPort::EventDriven) {
        OVERLAPPED overlapRead;
        ZeroMemory(&overlapRead, sizeof(OVERLAPPED));

        if (!ReadFile(handle, (void *)data, (DWORD)maxSize, &bytesRead, &overlapRead)) {
            if (GetLastError() == ERROR_IO_PENDING) {
                GetOverlappedResult(handle, &overlapRead, &bytesRead, true);
            } else {
                failed = true;
            }
        }
    } else if (!ReadFile(handle, (void *)data, (DWORD)maxSize, &bytesRead, NULL)) {
        failed = true;
    }

    if (!failed) {
        return (qint64)bytesRead;
    }

    lastErr = E_READ_FAILED;
    return -1;
}

/*
    Writes a block of data to the serial port.  This function will write len bytes
    from the buffer pointed to by data to the serial port.  Return value is the number
    of bytes actually written, or -1 on error.

    \warning before calling this function ensure that serial port associated with this class
    is currently open (use isOpen() function to check if port is open).
*/
qint64 QextSerialPortPrivate::writeData_sys(const char *data, qint64 maxSize)
{
    DWORD bytesWritten = 0;
    bool failed = false;

    if (queryMode == QextSerialPort::EventDriven) {
        OVERLAPPED *newOverlapWrite = new OVERLAPPED;
        ZeroMemory(newOverlapWrite, sizeof(OVERLAPPED));
        newOverlapWrite->hEvent = CreateEvent(NULL, true, false, NULL);

        if (WriteFile(handle, (void *)data, (DWORD)maxSize, &bytesWritten, newOverlapWrite)) {
            CloseHandle(newOverlapWrite->hEvent);
            delete newOverlapWrite;
        } else if (GetLastError() == ERROR_IO_PENDING) {
            // writing asynchronously...not an error
            QWriteLocker writelocker(bytesToWriteLock);
            pendingWrites.append(newOverlapWrite);
        } else {
            QESP_WARNING() << "QextSerialPort write error:" << GetLastError();
            failed = true;

            if (!CancelIo(newOverlapWrite->hEvent)) {
                QESP_WARNING("QextSerialPort: couldn't cancel IO");
            }

            if (!CloseHandle(newOverlapWrite->hEvent)) {
                QESP_WARNING("QextSerialPort: couldn't close OVERLAPPED handle");
            }

            delete newOverlapWrite;
        }
    } else if (!WriteFile(handle, (void *)data, (DWORD)maxSize, &bytesWritten, NULL)) {
        failed = true;
    }

    if (!failed) {
        return (qint64)bytesWritten;
    }

    lastErr = E_WRITE_FAILED;
    return -1;
}

void QextSerialPortPrivate::setDtr_sys(bool set)
{
    EscapeCommFunction(handle, set ? SETDTR : CLRDTR);
}

void QextSerialPortPrivate::setRts_sys(bool set)
{
    EscapeCommFunction(handle, set ? SETRTS : CLRRTS);
}

ulong QextSerialPortPrivate::lineStatus_sys(void)
{
    unsigned long Status = 0, Temp = 0;
    GetCommModemStatus(handle, &Temp);

    if (Temp & MS_CTS_ON) {
        Status |= LS_CTS;
    }

    if (Temp & MS_DSR_ON) {
        Status |= LS_DSR;
    }

    if (Temp & MS_RING_ON) {
        Status |= LS_RI;
    }

    if (Temp & MS_RLSD_ON) {
        Status |= LS_DCD;
    }

    return Status;
}

/*
  Triggered when there's activity on our HANDLE.
*/
void QextSerialPortPrivate::_q_onWinEvent(HANDLE h)
{
    Q_Q(QextSerialPort);

    if (h == overlap.hEvent) {
        if (eventMask & EV_RXCHAR) {
            if (q->sender() != q && bytesAvailable_sys() > 0) {
                _q_canRead();
            }
        }

        if (eventMask & EV_TXEMPTY) {
            /*
              A write completed.  Run through the list of OVERLAPPED writes, and if
              they completed successfully, take them off the list and delete them.
              Otherwise, leave them on there so they can finish.
            */
            qint64 totalBytesWritten = 0;
            QList<OVERLAPPED *> overlapsToDelete;

            foreach (OVERLAPPED *o, pendingWrites) {
                DWORD numBytes = 0;

                if (GetOverlappedResult(handle, o, &numBytes, false)) {
                    overlapsToDelete.append(o);
                    totalBytesWritten += numBytes;
                } else if (GetLastError() != ERROR_IO_INCOMPLETE) {
                    overlapsToDelete.append(o);
                    QESP_WARNING() << "CommEvent overlapped write error:" << GetLastError();
                }
            }

            if (q->sender() != q && totalBytesWritten > 0) {
                QWriteLocker writelocker(bytesToWriteLock);
                Q_EMIT q->bytesWritten(totalBytesWritten);
            }

            foreach (OVERLAPPED *o, overlapsToDelete) {
                OVERLAPPED *toDelete = pendingWrites.takeAt(pendingWrites.indexOf(o));
                CloseHandle(toDelete->hEvent);
                delete toDelete;
            }
        }

        if (eventMask & EV_DSR) {
            if (lineStatus_sys() & LS_DSR) {
                Q_EMIT q->dsrChanged(true);
            } else {
                Q_EMIT q->dsrChanged(false);
            }
        }
    }

    WaitCommEvent(handle, &eventMask, &overlap);
}

void QextSerialPortPrivate::updatePortSettings()
{
    if (!q_ptr->isOpen() || !settingsDirtyFlags) {
        return;
    }

    //fill struct : COMMCONFIG
    if (settingsDirtyFlags & DFE_BaudRate) {
        commConfig.dcb.BaudRate = settings.BaudRate;
    }

    if (settingsDirtyFlags & DFE_Parity) {
        commConfig.dcb.Parity = (BYTE)settings.Parity;
        commConfig.dcb.fParity = (settings.Parity == PAR_NONE) ? FALSE : TRUE;
    }

    if (settingsDirtyFlags & DFE_DataBits) {
        commConfig.dcb.ByteSize = (BYTE)settings.DataBits;
    }

    if (settingsDirtyFlags & DFE_StopBits) {
        switch (settings.StopBits) {
            case STOP_1:
                commConfig.dcb.StopBits = ONESTOPBIT;
                break;

            case STOP_1_5:
                commConfig.dcb.StopBits = ONE5STOPBITS;
                break;

            case STOP_2:
                commConfig.dcb.StopBits = TWOSTOPBITS;
                break;
        }
    }

    if (settingsDirtyFlags & DFE_Flow) {
        switch (settings.FlowControl) {
            /*no flow control*/
            case FLOW_OFF:
                commConfig.dcb.fOutxCtsFlow = FALSE;
                commConfig.dcb.fRtsControl = RTS_CONTROL_DISABLE;
                commConfig.dcb.fInX = FALSE;
                commConfig.dcb.fOutX = FALSE;
                break;

            /*software (XON/XOFF) flow control*/
            case FLOW_XONXOFF:
                commConfig.dcb.fOutxCtsFlow = FALSE;
                commConfig.dcb.fRtsControl = RTS_CONTROL_DISABLE;
                commConfig.dcb.fInX = TRUE;
                commConfig.dcb.fOutX = TRUE;
                break;

            /*hardware flow control*/
            case FLOW_HARDWARE:
                commConfig.dcb.fOutxCtsFlow = TRUE;
                commConfig.dcb.fRtsControl = RTS_CONTROL_HANDSHAKE;
                commConfig.dcb.fInX = FALSE;
                commConfig.dcb.fOutX = FALSE;
                break;
        }
    }

    //fill struct : COMMTIMEOUTS
    if (settingsDirtyFlags & DFE_TimeOut) {
        if (queryMode != QextSerialPort::EventDriven) {
            int millisec = settings.Timeout_Millisec;

            if (millisec == -1) {
                commTimeouts.ReadIntervalTimeout = MAXDWORD;
                commTimeouts.ReadTotalTimeoutConstant = 0;
            } else {
                commTimeouts.ReadIntervalTimeout = millisec;
                commTimeouts.ReadTotalTimeoutConstant = millisec;
            }

            commTimeouts.ReadTotalTimeoutMultiplier = 0;
            commTimeouts.WriteTotalTimeoutMultiplier = millisec;
            commTimeouts.WriteTotalTimeoutConstant = 0;
        } else {
            commTimeouts.ReadIntervalTimeout = MAXDWORD;
            commTimeouts.ReadTotalTimeoutMultiplier = 0;
            commTimeouts.ReadTotalTimeoutConstant = 0;
            commTimeouts.WriteTotalTimeoutMultiplier = 0;
            commTimeouts.WriteTotalTimeoutConstant = 0;
        }
    }


    if (settingsDirtyFlags & DFE_Settings_Mask) {
        SetCommConfig(handle, &commConfig, sizeof(COMMCONFIG));
    }

    if ((settingsDirtyFlags & DFE_TimeOut)) {
        SetCommTimeouts(handle, &commTimeouts);
    }

    settingsDirtyFlags = 0;
}
```

### `./3rd_qextserialport.pri`

```makefile
#将当前目录加入到头文件路径
INCLUDEPATH += $$PWD

HEADERS += $$PWD/qextserialport.h
HEADERS += $$PWD/qextserialport_global.h
HEADERS += $$PWD/qextserialport_p.h
SOURCES += $$PWD/qextserialport.cpp

win32 {
SOURCES += $$PWD/qextserialport_win.cpp
}
unix {
SOURCES += $$PWD/qextserialport_unix.cpp
}
```
