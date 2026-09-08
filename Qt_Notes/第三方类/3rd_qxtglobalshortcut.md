---
tags:
  - Qt
  - 第三方类
---
# QXT 全局快捷键库 (`3rd_qxtglobalshortcut`)

## 分类

- 类型: 第三方类 `third`
- 源码目录: `third/3rd_qxtglobalshortcut`

## 技术要点

**用到的 Qt 类**: QString QX11Info QKeySequence QCFType QAbstractEventDispatcher QObject QWidget QRect QCFString QHash QApplication QStringList QPoint QPair QByteArray QVector QMap QAbstractNativeEventFilter QPixmap QLibrary

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./qxtglobal.cpp`

```cpp

/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#include "qxtglobal.h"

/*!
    \headerfile <QxtGlobal>
    \title Global Qxt Declarations
    \inmodule QxtCore

    \brief The <QxtGlobal> header provides basic declarations and
    is included by all other Qxt headers.
 */

/*!
    \macro QXT_VERSION
    \relates <QxtGlobal>

    This macro expands a numeric value of the form 0xMMNNPP (MM =
    major, NN = minor, PP = patch) that specifies Qxt's version
    number. For example, if you compile your application against Qxt
    0.4.0, the QXT_VERSION macro will expand to 0x000400.

    You can use QXT_VERSION to use the latest Qt features where
    available. For example:
    \code
    #if QXT_VERSION >= 0x000400
        qxtTabWidget->setTabMovementMode(QxtTabWidget::InPlaceMovement);
    #endif
    \endcode

    \sa QXT_VERSION_STR, qxtVersion()
 */

/*!
    \macro QXT_VERSION_STR
    \relates <QxtGlobal>

    This macro expands to a string that specifies Qxt's version number
    (for example, "0.4.0"). This is the version against which the
    application is compiled.

    \sa qxtVersion(), QXT_VERSION
 */

/*!
    \relates <QxtGlobal>

    Returns the version number of Qxt at run-time as a string (for
    example, "0.4.0"). This may be a different version than the
    version the application was compiled against.

    \sa QXT_VERSION_STR
 */
const char * qxtVersion()
{
    return QXT_VERSION_STR;
}

/*!
\headerfile <QxtPimpl>
\title The Qxt private implementation
\inmodule QxtCore

\brief The <QxtPimpl> header provides tools for hiding
details of a class.

Application code generally doesn't have to be concerned about hiding its
implementation details, but when writing library code it is important to
maintain a constant interface, both source and binary. Maintaining a constant
source interface is easy enough, but keeping the binary interface constant
means moving implementation details into a private class. The PIMPL, or
d-pointer, idiom is a common method of implementing this separation. QxtPimpl
offers a convenient way to connect the public and private sides of your class.

\section1 Getting Started
Before you declare the public class, you need to make a forward declaration
of the private class. The private class must have the same name as the public
class, followed by the word Private. For example, a class named MyTest would
declare the private class with:
\code
class MyTestPrivate;
\endcode

\section1 The Public Class
Generally, you shouldn't keep any data members in the public class without a
good reason. Functions that are part of the public interface should be declared
in the public class, and functions that need to be available to subclasses (for
calling or overriding) should be in the protected section of the public class.
To connect the private class to the public class, include the
QXT_DECLARE_PRIVATE macro in the private section of the public class. In the
example above, the private class is connected as follows:
\code
private:
    QXT_DECLARE_PRIVATE(MyTest)
\endcode

Additionally, you must include the QXT_INIT_PRIVATE macro in the public class's
constructor. Continuing with the MyTest example, your constructor might look
like this:
\code
MyTest::MyTest() {
    // initialization
    QXT_INIT_PRIVATE(MyTest);
}
\endcode

\section1 The Private Class
As mentioned above, data members should usually be kept in the private class.
This allows the memory layout of the private class to change without breaking
binary compatibility for the public class. Functions that exist only as
implementation details, or functions that need access to private data members,
should be implemented here.

To define the private class, inherit from the template QxtPrivate class, and
include the QXT_DECLARE_PUBLIC macro in its public section. The template
parameter should be the name of the public class. For example:
\code
class MyTestPrivate : public QxtPrivate<MyTest> {
public:
    MyTestPrivate();
    QXT_DECLARE_PUBLIC(MyTest)
};
\endcode

\section1 Accessing Private Members
Use the qxt_d() function (actually a function-like object) from functions in
the public class to access the private class. Similarly, functions in the
private class can invoke functions in the public class by using the qxt_p()
function (this one's actually a function).

For example, assume that MyTest has methods named getFoobar and doBaz(),
and MyTestPrivate has a member named foobar and a method named doQuux().
The code might resemble this example:
\code
int MyTest::getFoobar() {
    return qxt_d().foobar;
}

void MyTestPrivate::doQuux() {
    qxt_p().doBaz(foobar);
}
\endcode
*/

/*! 
 * \macro QXT_DECLARE_PRIVATE(PUB)
 * \relates <QxtPimpl>
 * Declares that a public class has a related private class.
 *
 * This shuold be put in the private section of the public class. The
 * parameter \a PUB must be the name of the public class.
 */

/*!
 * \macro QXT_DECLARE_PUBLIC(PUB)
 * \relates <QxtPimpl>
 * Declares that a private class has a related public class named \a PUB.
 *
 * This may be put anywhere in the declaration of the private class. The parameter is the name of the public class.
 */

/*!
 * \macro QXT_INIT_PRIVATE(PUB)
 * \relates <QxtPimpl>
 * Initializes resources owned by the private class.
 *
 * This should be called from the public class's constructor,
 * before qxt_d() is used for the first time. The parameter \a PUB must be
 * the name of the public class.
 */

/*!
 * \macro QXT_D(PUB)
 * \relates <QxtPimpl>
 * Returns a reference in the current scope named "d" to the private class
 * associated with the public class \a PUB.
 *
 * This function is only available in a class using QXT_DECLARE_PRIVATE().
 */

/*!
 * \macro QXT_P(PUB)
 * \relates <QxtPimpl>
 * Creates a reference in the current scope named "q" to the public class
 * named \a PUB.
 *
 * This macro only works in a class using QXT_DECLARE_PUBLIC().
 */

/*!
 * \fn QxtPrivate<PUB>& PUB::qxt_d()
 * \relates <QxtPimpl>
 * Returns a reference to the private class.
 *
 * This function is only available in a class using \a QXT_DECLARE_PRIVATE.
 */

/*!
 * \fn const QxtPrivate<PUB>& PUB::qxt_d() const
 * \relates <QxtPimpl>
 * Returns a const reference to the private class.
 *
 * This function is only available in a class using \a QXT_DECLARE_PRIVATE.
 * This overload will be automatically used in const functions.
 */

/*!
 * \fn PUB& QxtPrivate::qxt_p()
 * \relates <QxtPimpl>
 * Returns a reference to the public class.
 *
 * This function is only available in a class using QXT_DECLARE_PUBLIC().
 */

/*!
 * \fn const PUB& QxtPrivate::qxt_p() const
 * \relates <QxtPimpl>
 * Returns a const reference to the public class.
 *
 * This function is only available in a class using QXT_DECLARE_PUBLIC().
 * This overload will be automatically used in const functions.
 */
```

### `./qxtglobal.h`

```cpp

/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#ifndef QXTGLOBAL_H
#define QXTGLOBAL_H

#include <QtGlobal>

#define QXT_VERSION 0x000700
#define QXT_VERSION_STR "0.7.0"

//--------------------------global macros------------------------------

#ifndef QXT_NO_MACROS

#endif // QXT_NO_MACROS

//--------------------------export macros------------------------------

#define QXT_DLLEXPORT DO_NOT_USE_THIS_ANYMORE

#if !defined(QXT_STATIC) && !defined(QXT_DOXYGEN_RUN)
#    if defined(BUILD_QXT_CORE)
#        define QXT_CORE_EXPORT Q_DECL_EXPORT
#    else
#        define QXT_CORE_EXPORT Q_DECL_IMPORT
#    endif
#else
#    define QXT_CORE_EXPORT
#endif // BUILD_QXT_CORE
 
#if !defined(QXT_STATIC) && !defined(QXT_DOXYGEN_RUN)
#    if defined(BUILD_QXT_GUI)
#        define QXT_GUI_EXPORT Q_DECL_EXPORT
#    else
#        define QXT_GUI_EXPORT Q_DECL_IMPORT
#    endif
#else
#    define QXT_GUI_EXPORT
#endif // BUILD_QXT_GUI
 
#if !defined(QXT_STATIC) && !defined(QXT_DOXYGEN_RUN)
#    if defined(BUILD_QXT_NETWORK)
#        define QXT_NETWORK_EXPORT Q_DECL_EXPORT
#    else
#        define QXT_NETWORK_EXPORT Q_DECL_IMPORT
#    endif
#else
#    define QXT_NETWORK_EXPORT
#endif // BUILD_QXT_NETWORK
 
#if !defined(QXT_STATIC) && !defined(QXT_DOXYGEN_RUN)
#    if defined(BUILD_QXT_SQL)
#        define QXT_SQL_EXPORT Q_DECL_EXPORT
#    else
#        define QXT_SQL_EXPORT Q_DECL_IMPORT
#    endif
#else
#    define QXT_SQL_EXPORT
#endif // BUILD_QXT_SQL
 
#if !defined(QXT_STATIC) && !defined(QXT_DOXYGEN_RUN)
#    if defined(BUILD_QXT_WEB)
#        define QXT_WEB_EXPORT Q_DECL_EXPORT
#    else
#        define QXT_WEB_EXPORT Q_DECL_IMPORT
#    endif
#else
#    define QXT_WEB_EXPORT
#endif // BUILD_QXT_WEB
 
#if !defined(QXT_STATIC) && !defined(QXT_DOXYGEN_RUN)
#    if defined(BUILD_QXT_BERKELEY)
#        define QXT_BERKELEY_EXPORT Q_DECL_EXPORT
#    else
#        define QXT_BERKELEY_EXPORT Q_DECL_IMPORT
#    endif
#else
#    define QXT_BERKELEY_EXPORT
#endif // BUILD_QXT_BERKELEY

#if !defined(QXT_STATIC) && !defined(QXT_DOXYGEN_RUN)
#    if defined(BUILD_QXT_ZEROCONF)
#        define QXT_ZEROCONF_EXPORT Q_DECL_EXPORT
#    else
#        define QXT_ZEROCONF_EXPORT Q_DECL_IMPORT
#    endif
#else
#    define QXT_ZEROCONF_EXPORT
#endif // QXT_ZEROCONF_EXPORT

#if defined(BUILD_QXT_CORE) || defined(BUILD_QXT_GUI) || defined(BUILD_QXT_SQL) || defined(BUILD_QXT_NETWORK) || defined(BUILD_QXT_WEB) || defined(BUILD_QXT_BERKELEY) || defined(BUILD_QXT_ZEROCONF)
#   define BUILD_QXT
#endif

QXT_CORE_EXPORT const char* qxtVersion();

#ifndef QT_BEGIN_NAMESPACE
#define QT_BEGIN_NAMESPACE
#endif

#ifndef QT_END_NAMESPACE
#define QT_END_NAMESPACE
#endif

#ifndef QT_FORWARD_DECLARE_CLASS
#define QT_FORWARD_DECLARE_CLASS(Class) class Class;
#endif

/****************************************************************************
** This file is derived from code bearing the following notice:
** The sole author of this file, Adam Higerd, has explicitly disclaimed all
** copyright interest and protection for the content within. This file has
** been placed in the public domain according to United States copyright
** statute and case law. In jurisdictions where this public domain dedication
** is not legally recognized, anyone who receives a copy of this file is
** permitted to use, modify, duplicate, and redistribute this file, in whole
** or in part, with no restrictions or conditions. In these jurisdictions,
** this file shall be copyright (C) 2006-2008 by Adam Higerd.
****************************************************************************/

#define QXT_DECLARE_PRIVATE(PUB) friend class PUB##Private; QxtPrivateInterface<PUB, PUB##Private> qxt_d;
#define QXT_DECLARE_PUBLIC(PUB) friend class PUB;
#define QXT_INIT_PRIVATE(PUB) qxt_d.setPublic(this);
#define QXT_D(PUB) PUB##Private& d = qxt_d()
#define QXT_P(PUB) PUB& p = qxt_p()

template <typename PUB>
class QxtPrivate
{
public:
    virtual ~QxtPrivate()
    {}
    inline void QXT_setPublic(PUB* pub)
    {
        qxt_p_ptr = pub;
    }

protected:
    inline PUB& qxt_p()
    {
        return *qxt_p_ptr;
    }
    inline const PUB& qxt_p() const
    {
        return *qxt_p_ptr;
    }
    inline PUB* qxt_ptr()
    {
        return qxt_p_ptr;
    }
    inline const PUB* qxt_ptr() const
    {
        return qxt_p_ptr;
    }

private:
    PUB* qxt_p_ptr;
};

template <typename PUB, typename PVT>
class QxtPrivateInterface
{
    friend class QxtPrivate<PUB>;
public:
    QxtPrivateInterface()
    {
        pvt = new PVT;
    }
    ~QxtPrivateInterface()
    {
        delete pvt;
    }

    inline void setPublic(PUB* pub)
    {
        pvt->QXT_setPublic(pub);
    }
    inline PVT& operator()()
    {
        return *static_cast<PVT*>(pvt);
    }
    inline const PVT& operator()() const
    {
        return *static_cast<PVT*>(pvt);
    }
    inline PVT * operator->()
    {
	return static_cast<PVT*>(pvt);
    }
    inline const PVT * operator->() const
    {
	return static_cast<PVT*>(pvt);
    }
private:
    QxtPrivateInterface(const QxtPrivateInterface&) { }
    QxtPrivateInterface& operator=(const QxtPrivateInterface&) { }
    QxtPrivate<PUB>* pvt;
};

#endif // QXT_GLOBAL
```

### `./qxtglobalshortcut.cpp`

```cpp
#include "qxtglobalshortcut.h"
/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#include "qxtglobalshortcut_p.h"
#include <QAbstractEventDispatcher>
#include <QtDebug>

#ifndef Q_OS_MAC
int QxtGlobalShortcutPrivate::ref = 0;
#   if QT_VERSION < QT_VERSION_CHECK(5,0,0)
QAbstractEventDispatcher::EventFilter QxtGlobalShortcutPrivate::prevEventFilter = 0;
#   endif
#endif // Q_OS_MAC
QHash<QPair<quint32, quint32>, QxtGlobalShortcut*> QxtGlobalShortcutPrivate::shortcuts;

QxtGlobalShortcutPrivate::QxtGlobalShortcutPrivate() : enabled(true), key(Qt::Key(0)), mods(Qt::NoModifier)
{
#ifndef Q_OS_MAC
    if (ref == 0) {
#   if QT_VERSION < QT_VERSION_CHECK(5,0,0)
        prevEventFilter = QAbstractEventDispatcher::instance()->setEventFilter(eventFilter);
#   else
        QAbstractEventDispatcher::instance()->installNativeEventFilter(this);
#endif
    }
    ++ref;
#endif // Q_OS_MAC
}

QxtGlobalShortcutPrivate::~QxtGlobalShortcutPrivate()
{
#ifndef Q_OS_MAC
    --ref;
    if (ref == 0) {
        QAbstractEventDispatcher *ed = QAbstractEventDispatcher::instance();
        if (ed != 0) {
#   if QT_VERSION < QT_VERSION_CHECK(5,0,0)
            ed->setEventFilter(prevEventFilter);
#   else
            ed->removeNativeEventFilter(this);
#   endif
        }
    }
#endif // Q_OS_MAC
}

bool QxtGlobalShortcutPrivate::setShortcut(const QKeySequence& shortcut)
{
    Qt::KeyboardModifiers allMods = Qt::ShiftModifier | Qt::ControlModifier | Qt::AltModifier | Qt::MetaModifier;
    key = shortcut.isEmpty() ? Qt::Key(0) : Qt::Key((shortcut[0] ^ allMods) & shortcut[0]);
    mods = shortcut.isEmpty() ? Qt::KeyboardModifiers(0) : Qt::KeyboardModifiers(shortcut[0] & allMods);
    const quint32 nativeKey = nativeKeycode(key);
    const quint32 nativeMods = nativeModifiers(mods);
    const bool res = registerShortcut(nativeKey, nativeMods);
    if (res)
        shortcuts.insert(qMakePair(nativeKey, nativeMods), &qxt_p());
    else
        qWarning() << "QxtGlobalShortcut failed to register:" << QKeySequence(key + mods).toString();
    return res;
}

bool QxtGlobalShortcutPrivate::unsetShortcut()
{
    bool res = false;
    const quint32 nativeKey = nativeKeycode(key);
    const quint32 nativeMods = nativeModifiers(mods);
    if (shortcuts.value(qMakePair(nativeKey, nativeMods)) == &qxt_p())
        res = unregisterShortcut(nativeKey, nativeMods);
    if (res)
        shortcuts.remove(qMakePair(nativeKey, nativeMods));
    else
        qWarning() << "QxtGlobalShortcut failed to unregister:" << QKeySequence(key + mods).toString();
    key = Qt::Key(0);
    mods = Qt::KeyboardModifiers(0);
    return res;
}

void QxtGlobalShortcutPrivate::activateShortcut(quint32 nativeKey, quint32 nativeMods)
{
    QxtGlobalShortcut* shortcut = shortcuts.value(qMakePair(nativeKey, nativeMods));
    if (shortcut && shortcut->isEnabled())
        emit shortcut->activated();
}

/*!
    \class QxtGlobalShortcut
    \inmodule QxtWidgets
    \brief The QxtGlobalShortcut class provides a global shortcut aka "hotkey".

    A global shortcut triggers even if the application is not active. This
    makes it easy to implement applications that react to certain shortcuts
    still if some other application is active or if the application is for
    example minimized to the system tray.

    Example usage:
    \code
    QxtGlobalShortcut* shortcut = new QxtGlobalShortcut(window);
    connect(shortcut, SIGNAL(activated()), window, SLOT(toggleVisibility()));
    shortcut->setShortcut(QKeySequence("Ctrl+Shift+F12"));
    \endcode

    \bold {Note:} Since Qxt 0.6 QxtGlobalShortcut no more requires QxtApplication.
 */

/*!
    \fn QxtGlobalShortcut::activated()

    This signal is emitted when the user types the shortcut's key sequence.

    \sa shortcut
 */

/*!
    Constructs a new QxtGlobalShortcut with \a parent.
 */
QxtGlobalShortcut::QxtGlobalShortcut(QObject* parent)
        : QObject(parent)
{
    QXT_INIT_PRIVATE(QxtGlobalShortcut);
}

/*!
    Constructs a new QxtGlobalShortcut with \a shortcut and \a parent.
 */
QxtGlobalShortcut::QxtGlobalShortcut(const QKeySequence& shortcut, QObject* parent)
        : QObject(parent)
{
    QXT_INIT_PRIVATE(QxtGlobalShortcut);
    setShortcut(shortcut);
}

/*!
    Destructs the QxtGlobalShortcut.
 */
QxtGlobalShortcut::~QxtGlobalShortcut()
{
    if (qxt_d().key != 0)
        qxt_d().unsetShortcut();
}

/*!
    \property QxtGlobalShortcut::shortcut
    \brief the shortcut key sequence

    \bold {Note:} Notice that corresponding key press and release events are not
    delivered for registered global shortcuts even if they are disabled.
    Also, comma separated key sequences are not supported.
    Only the first part is used:

    \code
    qxtShortcut->setShortcut(QKeySequence("Ctrl+Alt+A,Ctrl+Alt+B"));
    Q_ASSERT(qxtShortcut->shortcut() == QKeySequence("Ctrl+Alt+A"));
    \endcode
 */
QKeySequence QxtGlobalShortcut::shortcut() const
{
    return QKeySequence(qxt_d().key | qxt_d().mods);
}

bool QxtGlobalShortcut::setShortcut(const QKeySequence& shortcut)
{
    if (qxt_d().key != 0)
        qxt_d().unsetShortcut();
    return qxt_d().setShortcut(shortcut);
}

/*!
    \property QxtGlobalShortcut::enabled
    \brief whether the shortcut is enabled

    A disabled shortcut does not get activated.

    The default value is \c true.

    \sa setDisabled()
 */
bool QxtGlobalShortcut::isEnabled() const
{
    return qxt_d().enabled;
}

void QxtGlobalShortcut::setEnabled(bool enabled)
{
    qxt_d().enabled = enabled;
}

/*!
    Sets the shortcut \a disabled.

    \sa enabled
 */
void QxtGlobalShortcut::setDisabled(bool disabled)
{
    qxt_d().enabled = !disabled;
}
```

### `./qxtglobalshortcut.h`

```cpp
#ifndef QXTGLOBALSHORTCUT_H
/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#define QXTGLOBALSHORTCUT_H

#include "qxtglobal.h"
#include <QObject>
#include <QKeySequence>
class QxtGlobalShortcutPrivate;

class QXT_GUI_EXPORT QxtGlobalShortcut : public QObject
{
    Q_OBJECT
    QXT_DECLARE_PRIVATE(QxtGlobalShortcut)
    Q_PROPERTY(bool enabled READ isEnabled WRITE setEnabled)
    Q_PROPERTY(QKeySequence shortcut READ shortcut WRITE setShortcut)

public:
    explicit QxtGlobalShortcut(QObject* parent = 0);
    explicit QxtGlobalShortcut(const QKeySequence& shortcut, QObject* parent = 0);
    virtual ~QxtGlobalShortcut();

    QKeySequence shortcut() const;
    bool setShortcut(const QKeySequence& shortcut);

    bool isEnabled() const;

public Q_SLOTS:
    void setEnabled(bool enabled = true);
    void setDisabled(bool disabled = true);

Q_SIGNALS:
    void activated();
};

#endif // QXTGLOBALSHORTCUT_H
```

### `./qxtglobalshortcut_mac.cpp`

```cpp
#include <Carbon/Carbon.h>
/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#include "qxtglobalshortcut_p.h"
#include <QMap>
#include <QHash>
#include <QtDebug>
#include <QApplication>

typedef QPair<uint, uint> Identifier;
static QMap<quint32, EventHotKeyRef> keyRefs;
static QHash<Identifier, quint32> keyIDs;
static quint32 hotKeySerial = 0;
static bool qxt_mac_handler_installed = false;

OSStatus qxt_mac_handle_hot_key(EventHandlerCallRef nextHandler, EventRef event, void* data)
{
    Q_UNUSED(nextHandler);
    Q_UNUSED(data);
    if (GetEventClass(event) == kEventClassKeyboard && GetEventKind(event) == kEventHotKeyPressed)
    {
        EventHotKeyID keyID;
        GetEventParameter(event, kEventParamDirectObject, typeEventHotKeyID, NULL, sizeof(keyID), NULL, &keyID);
        Identifier id = keyIDs.key(keyID.id);
        QxtGlobalShortcutPrivate::activateShortcut(id.second, id.first);
    }
    return noErr;
}

quint32 QxtGlobalShortcutPrivate::nativeModifiers(Qt::KeyboardModifiers modifiers)
{
    quint32 native = 0;
    if (modifiers & Qt::ShiftModifier)
        native |= shiftKey;
    if (modifiers & Qt::ControlModifier)
        native |= cmdKey;
    if (modifiers & Qt::AltModifier)
        native |= optionKey;
    if (modifiers & Qt::MetaModifier)
        native |= controlKey;
    if (modifiers & Qt::KeypadModifier)
        native |= kEventKeyModifierNumLockMask;
    return native;
}

quint32 QxtGlobalShortcutPrivate::nativeKeycode(Qt::Key key)
{
    UTF16Char ch;
    // Constants found in NSEvent.h from AppKit.framework
    switch (key)
    {
    case Qt::Key_Return:
        return kVK_Return;
    case Qt::Key_Enter:
        return kVK_ANSI_KeypadEnter;
    case Qt::Key_Tab:
        return kVK_Tab;
    case Qt::Key_Space:
        return kVK_Space;
    case Qt::Key_Backspace:
        return kVK_Delete;
    case Qt::Key_Control:
        return kVK_Command;
    case Qt::Key_Shift:
        return kVK_Shift;
    case Qt::Key_CapsLock:
        return kVK_CapsLock;
    case Qt::Key_Option:
        return kVK_Option;
    case Qt::Key_Meta:
        return kVK_Control;
    case Qt::Key_F17:
        return kVK_F17;
    case Qt::Key_VolumeUp:
        return kVK_VolumeUp;
    case Qt::Key_VolumeDown:
        return kVK_VolumeDown;
    case Qt::Key_F18:
        return kVK_F18;
    case Qt::Key_F19:
        return kVK_F19;
    case Qt::Key_F20:
        return kVK_F20;
    case Qt::Key_F5:
        return kVK_F5;
    case Qt::Key_F6:
        return kVK_F6;
    case Qt::Key_F7:
        return kVK_F7;
    case Qt::Key_F3:
        return kVK_F3;
    case Qt::Key_F8:
        return kVK_F8;
    case Qt::Key_F9:
        return kVK_F9;
    case Qt::Key_F11:
        return kVK_F11;
    case Qt::Key_F13:
        return kVK_F13;
    case Qt::Key_F16:
        return kVK_F16;
    case Qt::Key_F14:
        return kVK_F14;
    case Qt::Key_F10:
        return kVK_F10;
    case Qt::Key_F12:
        return kVK_F12;
    case Qt::Key_F15:
        return kVK_F15;
    case Qt::Key_Help:
        return kVK_Help;
    case Qt::Key_Home:
        return kVK_Home;
    case Qt::Key_PageUp:
        return kVK_PageUp;
    case Qt::Key_Delete:
        return kVK_ForwardDelete;
    case Qt::Key_F4:
        return kVK_F4;
    case Qt::Key_End:
        return kVK_End;
    case Qt::Key_F2:
        return kVK_F2;
    case Qt::Key_PageDown:
        return kVK_PageDown;
    case Qt::Key_F1:
        return kVK_F1;
    case Qt::Key_Left:
        return kVK_LeftArrow;
    case Qt::Key_Right:
        return kVK_RightArrow;
    case Qt::Key_Down:
        return kVK_DownArrow;
    case Qt::Key_Up:
        return kVK_UpArrow;
    default:
        ;
    }

    if (key == Qt::Key_Escape)	ch = 27;
    else if (key == Qt::Key_Return) ch = 13;
    else if (key == Qt::Key_Enter) ch = 3;
    else if (key == Qt::Key_Tab) ch = 9;
    else ch = key;

    CFDataRef currentLayoutData;
    TISInputSourceRef currentKeyboard = TISCopyCurrentKeyboardInputSource();

    if (currentKeyboard == NULL)
        return 0;

    currentLayoutData = (CFDataRef)TISGetInputSourceProperty(currentKeyboard, kTISPropertyUnicodeKeyLayoutData);
    CFRelease(currentKeyboard);
    if (currentLayoutData == NULL)
        return 0;

    UCKeyboardLayout* header = (UCKeyboardLayout*)CFDataGetBytePtr(currentLayoutData);
    UCKeyboardTypeHeader* table = header->keyboardTypeList;

    uint8_t *data = (uint8_t*)header;
    // God, would a little documentation for this shit kill you...
    for (quint32 i=0; i < header->keyboardTypeCount; i++)
    {
        UCKeyStateRecordsIndex* stateRec = 0;
        if (table[i].keyStateRecordsIndexOffset != 0)
        {
            stateRec = reinterpret_cast<UCKeyStateRecordsIndex*>(data + table[i].keyStateRecordsIndexOffset);
            if (stateRec->keyStateRecordsIndexFormat != kUCKeyStateRecordsIndexFormat) stateRec = 0;
        }

        UCKeyToCharTableIndex* charTable = reinterpret_cast<UCKeyToCharTableIndex*>(data + table[i].keyToCharTableIndexOffset);
        if (charTable->keyToCharTableIndexFormat != kUCKeyToCharTableIndexFormat) continue;

        for (quint32 j=0; j < charTable->keyToCharTableCount; j++)
        {
            UCKeyOutput* keyToChar = reinterpret_cast<UCKeyOutput*>(data + charTable->keyToCharTableOffsets[j]);
            for (quint32 k=0; k < charTable->keyToCharTableSize; k++)
            {
                if (keyToChar[k] & kUCKeyOutputTestForIndexMask)
                {
                    long idx = keyToChar[k] & kUCKeyOutputGetIndexMask;
                    if (stateRec && idx < stateRec->keyStateRecordCount)
                    {
                        UCKeyStateRecord* rec = reinterpret_cast<UCKeyStateRecord*>(data + stateRec->keyStateRecordOffsets[idx]);
                        if (rec->stateZeroCharData == ch) return k;
                    }
                }
                else if (!(keyToChar[k] & kUCKeyOutputSequenceIndexMask) && keyToChar[k] < 0xFFFE)
                {
                    if (keyToChar[k] == ch) return k;
                }
            } // for k
        } // for j
    } // for i
    return 0;
}

bool QxtGlobalShortcutPrivate::registerShortcut(quint32 nativeKey, quint32 nativeMods)
{
    if (!qxt_mac_handler_installed)
    {
        EventTypeSpec t;
        t.eventClass = kEventClassKeyboard;
        t.eventKind = kEventHotKeyPressed;
        InstallApplicationEventHandler(&qxt_mac_handle_hot_key, 1, &t, NULL, NULL);
    }

    EventHotKeyID keyID;
    keyID.signature = 'cute';
    keyID.id = ++hotKeySerial;

    EventHotKeyRef ref = 0;
    bool rv = !RegisterEventHotKey(nativeKey, nativeMods, keyID, GetApplicationEventTarget(), 0, &ref);
    if (rv)
    {
        keyIDs.insert(Identifier(nativeMods, nativeKey), keyID.id);
        keyRefs.insert(keyID.id, ref);
    }
    return rv;
}

bool QxtGlobalShortcutPrivate::unregisterShortcut(quint32 nativeKey, quint32 nativeMods)
{
    Identifier id(nativeMods, nativeKey);
    if (!keyIDs.contains(id)) return false;

    EventHotKeyRef ref = keyRefs.take(keyIDs[id]);
    keyIDs.remove(id);
    return !UnregisterEventHotKey(ref);
}
```

### `./qxtglobalshortcut_p.h`

```cpp
#ifndef QXTGLOBALSHORTCUT_P_H
/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#define QXTGLOBALSHORTCUT_P_H

#include "qxtglobalshortcut.h"
#include <QAbstractEventDispatcher>
#include <QKeySequence>
#include <QHash>

#if QT_VERSION >= QT_VERSION_CHECK(5,0,0)
#include <QAbstractNativeEventFilter>
#endif


class QxtGlobalShortcutPrivate : public QxtPrivate<QxtGlobalShortcut>
#if QT_VERSION >= QT_VERSION_CHECK(5,0,0) && !defined(Q_OS_MAC)
        ,public QAbstractNativeEventFilter
#endif
{
public:
    QXT_DECLARE_PUBLIC(QxtGlobalShortcut)
    QxtGlobalShortcutPrivate();
    ~QxtGlobalShortcutPrivate();

    bool enabled;
    Qt::Key key;
    Qt::KeyboardModifiers mods;

    bool setShortcut(const QKeySequence& shortcut);
    bool unsetShortcut();

    static bool error;
#ifndef Q_OS_MAC
    static int ref;
#if QT_VERSION < QT_VERSION_CHECK(5,0,0)
    static QAbstractEventDispatcher::EventFilter prevEventFilter;
    static bool eventFilter(void* message);
#else
    virtual bool nativeEventFilter(const QByteArray & eventType, void * message, long * result);
#endif // QT_VERSION < QT_VERSION_CHECK(5,0,0)
#endif // Q_OS_MAC

    static void activateShortcut(quint32 nativeKey, quint32 nativeMods);

private:
    static quint32 nativeKeycode(Qt::Key keycode);
    static quint32 nativeModifiers(Qt::KeyboardModifiers modifiers);

    static bool registerShortcut(quint32 nativeKey, quint32 nativeMods);
    static bool unregisterShortcut(quint32 nativeKey, quint32 nativeMods);

    static QHash<QPair<quint32, quint32>, QxtGlobalShortcut*> shortcuts;
};

#endif // QXTGLOBALSHORTCUT_P_H
```

### `./qxtglobalshortcut_win.cpp`

```cpp
﻿#include "qxtglobalshortcut_p.h"
/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#include <qt_windows.h>


#if QT_VERSION < QT_VERSION_CHECK(5,0,0)
bool QxtGlobalShortcutPrivate::eventFilter(void* message)
{
#else
bool QxtGlobalShortcutPrivate::nativeEventFilter(const QByteArray & eventType,
    void * message, long * result)
{
    Q_UNUSED(eventType);
    Q_UNUSED(result);
#endif
    MSG* msg = static_cast<MSG*>(message);
    if (msg->message == WM_HOTKEY)
    {
        const quint32 keycode = HIWORD(msg->lParam);
        const quint32 modifiers = LOWORD(msg->lParam);
        activateShortcut(keycode, modifiers);
    }

#if QT_VERSION < QT_VERSION_CHECK(5,0,0)
    return prevEventFilter ? prevEventFilter(message) : false;
#else
	return false;
#endif
}


quint32 QxtGlobalShortcutPrivate::nativeModifiers(Qt::KeyboardModifiers modifiers)
{
    // MOD_ALT, MOD_CONTROL, (MOD_KEYUP), MOD_SHIFT, MOD_WIN
    quint32 native = 0;
    if (modifiers & Qt::ShiftModifier)
        native |= MOD_SHIFT;
    if (modifiers & Qt::ControlModifier)
        native |= MOD_CONTROL;
    if (modifiers & Qt::AltModifier)
        native |= MOD_ALT;
    if (modifiers & Qt::MetaModifier)
        native |= MOD_WIN;
    // TODO: resolve these?
    //if (modifiers & Qt::KeypadModifier)
    //if (modifiers & Qt::GroupSwitchModifier)
    return native;
}

quint32 QxtGlobalShortcutPrivate::nativeKeycode(Qt::Key key)
{
    switch (key)
    {
    case Qt::Key_Escape:
        return VK_ESCAPE;
    case Qt::Key_Tab:
    case Qt::Key_Backtab:
        return VK_TAB;
    case Qt::Key_Backspace:
        return VK_BACK;
    case Qt::Key_Return:
    case Qt::Key_Enter:
        return VK_RETURN;
    case Qt::Key_Insert:
        return VK_INSERT;
    case Qt::Key_Delete:
        return VK_DELETE;
    case Qt::Key_Pause:
        return VK_PAUSE;
    case Qt::Key_Print:
        return VK_PRINT;
    case Qt::Key_Clear:
        return VK_CLEAR;
    case Qt::Key_Home:
        return VK_HOME;
    case Qt::Key_End:
        return VK_END;
    case Qt::Key_Left:
        return VK_LEFT;
    case Qt::Key_Up:
        return VK_UP;
    case Qt::Key_Right:
        return VK_RIGHT;
    case Qt::Key_Down:
        return VK_DOWN;
    case Qt::Key_PageUp:
        return VK_PRIOR;
    case Qt::Key_PageDown:
        return VK_NEXT;
    case Qt::Key_F1:
        return VK_F1;
    case Qt::Key_F2:
        return VK_F2;
    case Qt::Key_F3:
        return VK_F3;
    case Qt::Key_F4:
        return VK_F4;
    case Qt::Key_F5:
        return VK_F5;
    case Qt::Key_F6:
        return VK_F6;
    case Qt::Key_F7:
        return VK_F7;
    case Qt::Key_F8:
        return VK_F8;
    case Qt::Key_F9:
        return VK_F9;
    case Qt::Key_F10:
        return VK_F10;
    case Qt::Key_F11:
        return VK_F11;
    case Qt::Key_F12:
        return VK_F12;
    case Qt::Key_F13:
        return VK_F13;
    case Qt::Key_F14:
        return VK_F14;
    case Qt::Key_F15:
        return VK_F15;
    case Qt::Key_F16:
        return VK_F16;
    case Qt::Key_F17:
        return VK_F17;
    case Qt::Key_F18:
        return VK_F18;
    case Qt::Key_F19:
        return VK_F19;
    case Qt::Key_F20:
        return VK_F20;
    case Qt::Key_F21:
        return VK_F21;
    case Qt::Key_F22:
        return VK_F22;
    case Qt::Key_F23:
        return VK_F23;
    case Qt::Key_F24:
        return VK_F24;
    case Qt::Key_Space:
        return VK_SPACE;
    case Qt::Key_Asterisk:
        return VK_MULTIPLY;
    case Qt::Key_Plus:
        return VK_ADD;
    case Qt::Key_Comma:
        return VK_SEPARATOR;
    case Qt::Key_Minus:
        return VK_SUBTRACT;
    case Qt::Key_Slash:
        return VK_DIVIDE;
    case Qt::Key_MediaNext:
        return VK_MEDIA_NEXT_TRACK;
    case Qt::Key_MediaPrevious:
        return VK_MEDIA_PREV_TRACK;
    case Qt::Key_MediaPlay:
        return VK_MEDIA_PLAY_PAUSE;
    case Qt::Key_MediaStop:
        return VK_MEDIA_STOP;
        // couldn't find those in VK_*
        //case Qt::Key_MediaLast:
        //case Qt::Key_MediaRecord:
    case Qt::Key_VolumeDown:
        return VK_VOLUME_DOWN;
    case Qt::Key_VolumeUp:
        return VK_VOLUME_UP;
    case Qt::Key_VolumeMute:
        return VK_VOLUME_MUTE;

        // numbers
    case Qt::Key_0:
    case Qt::Key_1:
    case Qt::Key_2:
    case Qt::Key_3:
    case Qt::Key_4:
    case Qt::Key_5:
    case Qt::Key_6:
    case Qt::Key_7:
    case Qt::Key_8:
    case Qt::Key_9:
        return key;

        // letters
    case Qt::Key_A:
    case Qt::Key_B:
    case Qt::Key_C:
    case Qt::Key_D:
    case Qt::Key_E:
    case Qt::Key_F:
    case Qt::Key_G:
    case Qt::Key_H:
    case Qt::Key_I:
    case Qt::Key_J:
    case Qt::Key_K:
    case Qt::Key_L:
    case Qt::Key_M:
    case Qt::Key_N:
    case Qt::Key_O:
    case Qt::Key_P:
    case Qt::Key_Q:
    case Qt::Key_R:
    case Qt::Key_S:
    case Qt::Key_T:
    case Qt::Key_U:
    case Qt::Key_V:
    case Qt::Key_W:
    case Qt::Key_X:
    case Qt::Key_Y:
    case Qt::Key_Z:
        return key;

    default:
        return 0;
    }
}

bool QxtGlobalShortcutPrivate::registerShortcut(quint32 nativeKey, quint32 nativeMods)
{
    return RegisterHotKey(0, nativeMods ^ nativeKey, nativeMods, nativeKey);
}

bool QxtGlobalShortcutPrivate::unregisterShortcut(quint32 nativeKey, quint32 nativeMods)
{
    return UnregisterHotKey(0, nativeMods ^ nativeKey);
}
```

### `./qxtglobalshortcut_x11.cpp`

```cpp
#include "qxtglobalshortcut_p.h"
/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#include <QVector>
#if QT_VERSION < QT_VERSION_CHECK(5,0,0)
#   include <QX11Info>
#else
#   include <QApplication>
//#   include <qpa/qplatformnativeinterface.h>
#   include <xcb/xcb.h>
#endif
#include <X11/Xlib.h>
#include "QX11Info"

namespace {

const QVector<quint32> maskModifiers = QVector<quint32>()
    << 0 << Mod2Mask << LockMask << (Mod2Mask | LockMask);

typedef int (*X11ErrorHandler)(Display *display, XErrorEvent *event);

class QxtX11ErrorHandler {
public:
    static bool error;

    static int qxtX11ErrorHandler(Display *display, XErrorEvent *event)
    {
        Q_UNUSED(display);
        switch (event->error_code)
        {
            case BadAccess:
            case BadValue:
            case BadWindow:
                if (event->request_code == 33 /* X_GrabKey */ ||
                        event->request_code == 34 /* X_UngrabKey */)
                {
                    error = true;
                    //TODO:
                    //char errstr[256];
                    //XGetErrorText(dpy, err->error_code, errstr, 256);
                }
        }
        return 0;
    }

    QxtX11ErrorHandler()
    {
        error = false;
        m_previousErrorHandler = XSetErrorHandler(qxtX11ErrorHandler);
    }

    ~QxtX11ErrorHandler()
    {
        XSetErrorHandler(m_previousErrorHandler);
    }

private:
    X11ErrorHandler m_previousErrorHandler;
};

bool QxtX11ErrorHandler::error = false;

class QxtX11Data {
public:
    QxtX11Data()
    {
#if QT_VERSION < QT_VERSION_CHECK(5,0,0)
        m_display = QX11Info::display();
#else

        m_display = QX11Info::display();
#endif
    }

    bool isValid()
    {
        return m_display != 0;
    }

    Display *display()
    {
        Q_ASSERT(isValid());
        return m_display;
    }

    Window rootWindow()
    {
        return DefaultRootWindow(display());
    }

    bool grabKey(quint32 keycode, quint32 modifiers, Window window)
    {
        QxtX11ErrorHandler errorHandler;

        for (int i = 0; !errorHandler.error && i < maskModifiers.size(); ++i) {
            XGrabKey(display(), keycode, modifiers | maskModifiers[i], window, True,
                     GrabModeAsync, GrabModeAsync);
        }

        if (errorHandler.error) {
            ungrabKey(keycode, modifiers, window);
            return false;
        }

        return true;
    }

    bool ungrabKey(quint32 keycode, quint32 modifiers, Window window)
    {
        QxtX11ErrorHandler errorHandler;

        foreach (quint32 maskMods, maskModifiers) {
            XUngrabKey(display(), keycode, modifiers | maskMods, window);
        }

        return !errorHandler.error;
    }

private:
    Display *m_display;
};

} // namespace

#if QT_VERSION < QT_VERSION_CHECK(5,0,0)
bool QxtGlobalShortcutPrivate::eventFilter(void *message)
{
    XEvent *event = static_cast<XEvent *>(message);
    if (event->type == KeyPress)
    {
        XKeyEvent *key = reinterpret_cast<XKeyEvent *>(event);
        unsigned int keycode = key->keycode;
        unsigned int keystate = key->state;
#else
bool QxtGlobalShortcutPrivate::nativeEventFilter(const QByteArray & eventType,
    void *message, long *result)
{
    Q_UNUSED(result);

    xcb_key_press_event_t *kev = 0;
    if (eventType == "xcb_generic_event_t") {
        xcb_generic_event_t *ev = static_cast<xcb_generic_event_t *>(message);
        if ((ev->response_type & 127) == XCB_KEY_PRESS)
            kev = static_cast<xcb_key_press_event_t *>(message);
    }

    if (kev != 0) {
        unsigned int keycode = kev->detail;
        unsigned int keystate = 0;
        if(kev->state & XCB_MOD_MASK_1)
            keystate |= Mod1Mask;
        if(kev->state & XCB_MOD_MASK_CONTROL)
            keystate |= ControlMask;
        if(kev->state & XCB_MOD_MASK_4)
            keystate |= Mod4Mask;
        if(kev->state & XCB_MOD_MASK_SHIFT)
            keystate |= ShiftMask;
#endif
        activateShortcut(keycode,
            // Mod1Mask == Alt, Mod4Mask == Meta
            keystate & (ShiftMask | ControlMask | Mod1Mask | Mod4Mask));
    }
#if QT_VERSION < QT_VERSION_CHECK(5,0,0)
    return prevEventFilter ? prevEventFilter(message) : false;
#else
	return false;
#endif
}

quint32 QxtGlobalShortcutPrivate::nativeModifiers(Qt::KeyboardModifiers modifiers)
{
    // ShiftMask, LockMask, ControlMask, Mod1Mask, Mod2Mask, Mod3Mask, Mod4Mask, and Mod5Mask
    quint32 native = 0;
    if (modifiers & Qt::ShiftModifier)
        native |= ShiftMask;
    if (modifiers & Qt::ControlModifier)
        native |= ControlMask;
    if (modifiers & Qt::AltModifier)
        native |= Mod1Mask;
    if (modifiers & Qt::MetaModifier)
        native |= Mod4Mask;

    // TODO: resolve these?
    //if (modifiers & Qt::MetaModifier)
    //if (modifiers & Qt::KeypadModifier)
    //if (modifiers & Qt::GroupSwitchModifier)
    return native;
}

quint32 QxtGlobalShortcutPrivate::nativeKeycode(Qt::Key key)
{
    QxtX11Data x11;
    if (!x11.isValid())
        return 0;

    KeySym keysym = XStringToKeysym(QKeySequence(key).toString().toLatin1().data());
    if (keysym == NoSymbol)
        keysym = static_cast<ushort>(key);

    return XKeysymToKeycode(x11.display(), keysym);
}

bool QxtGlobalShortcutPrivate::registerShortcut(quint32 nativeKey, quint32 nativeMods)
{
    QxtX11Data x11;
    return x11.isValid() && x11.grabKey(nativeKey, nativeMods, x11.rootWindow());
}

bool QxtGlobalShortcutPrivate::unregisterShortcut(quint32 nativeKey, quint32 nativeMods)
{
    QxtX11Data x11;
    return x11.isValid() && x11.ungrabKey(nativeKey, nativeMods, x11.rootWindow());
}
```

### `./qxtwindowsystem.cpp`

```cpp
#include "qxtwindowsystem.h"
/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#include <QStringList>

/*!
    \class QxtWindowSystem
    \inmodule QxtWidgets
    \brief The QxtWindowSystem class provides means for accessing native windows.

    \bold {Note:} The underlying window system might or might not allow one to alter
    states of windows belonging to other processes.

    \warning QxtWindowSystem is portable in principle, but be careful while
    using it since you are probably about to do something non-portable.

    \section1 Advanced example usage:
    \code
    class NativeWindow : public QWidget {
        public:
            NativeWindow(WId wid) {
                QWidget::create(wid, false, false); // window, initializeWindow, destroyOldWindow
            }
            ~NativeWindow() {
                QWidget::destroy(false, false); // destroyWindow, destroySubWindows
            }
    };
    \endcode

    \code
    WindowList windows = QxtWindowSystem::windows();
    QStringList titles = QxtWindowSystem::windowTitles();
    bool ok = false;
    QString title = QInputDialog::getItem(0, "Choose Window", "Choose a window to be hid:", titles, 0, false, &ok);
    if (ok)
    {
        int index = titles.indexOf(title);
        if (index != -1)
        {
            NativeWindow window(windows.at(index));
            window.hide();
        }
    }
    \endcode

    \bold {Note:} Currently supported platforms are \bold X11 and \bold Windows.
 */

/*!
    \fn QxtWindowSystem::windows()

    Returns the list of native window system identifiers.

    \sa QApplication::topLevelWidgets(), windowTitles()
 */

/*!
    \fn QxtWindowSystem::activeWindow()

    Returns the native window system identifier of the active window if any.

    \sa QApplication::activeWindow()
 */

/*!
    \fn QxtWindowSystem::findWindow(const QString& title)

    Returns the native window system identifier of the window if any with given \a title.

    Example usage:
    \code
    WId wid = QxtWindowSystem::findWindow("Mail - Kontact");
    QPixmap screenshot = QPixmap::grabWindow(wid);
    \endcode

    \sa QWidget::find()
 */

/*!
    \fn QxtWindowSystem::windowAt(const QPoint& pos)

    Returns the native window system identifier of the window if any at \a pos.

    \sa QApplication::widgetAt()
 */

/*!
    \fn QxtWindowSystem::windowTitle(WId window)

    Returns the title of the native \a window.

    \sa QWidget::windowTitle(), windowTitles()
 */

/*!
    \fn QxtWindowSystem::windowTitles()

    Returns a list of native window titles.

    \sa QWidget::windowTitle(), windowTitle(), windows()
 */

/*!
    \fn QxtWindowSystem::windowGeometry(WId window)

    Returns the geometry of the native \a window.

    \sa QWidget::frameGeometry()
 */

/*!
    \fn QxtWindowSystem::idleTime()

    Returns the system "idle time" ie. the time since last user input
    in milliseconds.
 */

QStringList QxtWindowSystem::windowTitles()
{
    WindowList windows = QxtWindowSystem::windows();
    QStringList titles;
    foreach(WId window, windows)
    titles += QxtWindowSystem::windowTitle(window);
    return titles;
}
```

### `./qxtwindowsystem.h`

```cpp
#ifndef QXTWINDOWSYSTEM_H
/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#define QXTWINDOWSYSTEM_H

#include <QWidget>
#include "qxtglobal.h"

typedef QList<WId> WindowList;

class /*QXT_GUI_EXPORT*/ QxtWindowSystem
{
public:
    static WindowList windows();
    static WId activeWindow();
    static WId findWindow(const QString& title);
    static WId windowAt(const QPoint& pos);
    static QString windowTitle(WId window);
    static QStringList windowTitles();
    static QRect windowGeometry(WId window);

    static uint idleTime();
};

#endif // QXTWINDOWSYSTEM_H
```

### `./qxtwindowsystem_mac.cpp`

```cpp
#ifndef QXTWINDOWSYSTEM_MAC_CPP
#define QXTWINDOWSYSTEM_MAC_CPP

#include "qxtwindowsystem.h"
#include "qxtwindowsystem_mac.h"

// WId to return when error
#define WINDOW_NOT_FOUND (WId)(0)

WindowList qxt_getWindowsForPSN(ProcessSerialNumber *psn)
{
    static CGSConnection connection = _CGSDefaultConnection();

    WindowList wlist;
    if (!psn) return wlist;

    CGError err((CGError)noErr);

    // get onnection for given process psn
    CGSConnection procConnection;
    err = CGSGetConnectionIDForPSN(connection, psn, &procConnection);
    if (err != noErr) return wlist;

    /* get number of windows open by given process
       in Mac OS X an application may have multiple windows, which generally
       represent documents. It is also possible that there is no window even
       though there is an application, it can simply not have any documents open. */

    int windowCount(0);
    err = CGSGetOnScreenWindowCount(connection, procConnection, &windowCount);
    // if there are no windows open by this application, skip
    if (err != noErr || windowCount == 0) return wlist;

    // get list of windows
    int windowList[windowCount];
    int outCount(0);
    err = CGSGetOnScreenWindowList(connection, procConnection, windowCount, windowList, &outCount);

    if (err != noErr || outCount == 0) return wlist;

    for (int i=0; i<outCount; ++i)
    {
        wlist += windowList[i];
    }

    return wlist;
}


WindowList QxtWindowSystem::windows()
{
    WindowList wlist;
    ProcessSerialNumber psn = {0, kNoProcess};

    // iterate over list of processes
    OSErr err;
    while ((err = ::GetNextProcess(&psn)) == noErr)
    {
        wlist += qxt_getWindowsForPSN(&psn);
    }

    return wlist;
}

WId QxtWindowSystem::activeWindow()
{
    ProcessSerialNumber psn;
    OSErr err(noErr);
    err = ::GetFrontProcess(&psn);
    if (err != noErr) return WINDOW_NOT_FOUND;

    // in Mac OS X, first window for given PSN is always the active one
    WindowList wlist = qxt_getWindowsForPSN(&psn);

    if (wlist.count() > 0)
        return wlist.at(0);

    return WINDOW_NOT_FOUND;
}

QString QxtWindowSystem::windowTitle(WId window)
{
    CGSValue windowTitle;
    CGError err((CGError)noErr);
    static CGSConnection connection = _CGSDefaultConnection();

    // This code is so dirty I had to wash my hands after writing it.

    // most of CoreGraphics private definitions ask for CGSValue as key but since
    // converting strings to/from CGSValue was dropped in 10.5, I use CFString, which
    // apparently also works.

    // FIXME: Not public API function. Can't compile with OS X 10.8
    // err = CGSGetWindowProperty(connection, window, (CGSValue)CFSTR("kCGSWindowTitle"), &windowTitle);
    if (err != noErr) return QString();

    // this is UTF8 encoded
    return QCFString::toQString((CFStringRef)windowTitle);
}

QRect QxtWindowSystem::windowGeometry(WId window)
{
    CGRect rect;
    static CGSConnection connection = _CGSDefaultConnection();

    CGError err = CGSGetWindowBounds(connection, window, &rect);
    if (err != noErr) return QRect();

    return QRect(rect.origin.x, rect.origin.y, rect.size.width, rect.size.height);
}

/* This method is the only one that is not a complete hack
   from Quartz Event Services
   http://developer.apple.com/library/mac/#documentation/Carbon/Reference/QuartzEventServicesRef/Reference/reference.html
*/
uint QxtWindowSystem::idleTime()
{
    // CGEventSourceSecondsSinceLastEventType returns time in seconds as a double
    // also has extremely long name
    double idle = 1000 * ::CGEventSourceSecondsSinceLastEventType(kCGEventSourceStateCombinedSessionState, kCGAnyInputEventType);
    return (uint)idle;
}


// these are copied from X11 implementation
WId QxtWindowSystem::findWindow(const QString& title)
{
    WId result = 0;
    WindowList list = windows();
    foreach(const WId &wid, list)
    {
        if (windowTitle(wid) == title)
        {
            result = wid;
            break;
        }
    }
    return result;
}

WId QxtWindowSystem::windowAt(const QPoint& pos)
{
    WId result = 0;
    WindowList list = windows();
    for (int i = list.size() - 1; i >= 0; --i)
    {
        WId wid = list.at(i);
        if (windowGeometry(wid).contains(pos))
        {
            result = wid;
            break;
        }
    }
    return result;
}

#endif // QXTWINDOWSYSTEM_MAC_CPP
```

### `./qxtwindowsystem_mac.h`

```cpp

/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

/****************************************************************************
 **
 ** Copyright (C) Marcin Jakubowski <mjakubowski (at) gmail (dot) com
 **
 ** Defines some of required CoreGraphics private parts. This code may
 ** break at any time and uses *undocumented* features of Mac OS X.
 **
 ** Also defines QCFString, part of Qt code found in src/corelib/kernel
 ** qcoremac_p.h file.
 ****************************************************************************/

#ifndef QXTWINDOWSYSTEM_MAC_H
#define QXTWINDOWSYSTEM_MAC_H

//
//  CGSPrivate.h
//  Header file for undocumented CoreGraphics SPI
//
//  Arranged by Nicholas Jitkoff
//  Based on CGSPrivate.h by Richard Wareham
//
//  Contributors:
//    Austin Sarner: Shadows
//    Jason Harris: Filters, Shadows, Regions
//    Kevin Ballard: Warping
//    Steve Voida: Workspace notifications
//    Tony Arnold: Workspaces notifications enum filters
//    Ben Gertzfield: CGSRemoveConnectionNotifyProc
//
//  Changes:
//    2.3 - Added the CGSRemoveConnectionNotifyProc method with the help of Ben Gertzfield
//    2.2 - Moved back to CGSPrivate, added more enums to the CGSConnectionNotifyEvent
//    2.1 - Added spaces notifications
//    2.0 - Original Release

#include <Carbon/Carbon.h>

#define CGSConnectionID CGSConnection
#define CGSWindowID CGSWindow
#define CGSDefaultConnection _CGSDefaultConnection()

#ifdef __cplusplus
extern "C" {
#endif

  typedef int CGSConnection;
  typedef int CGSWindow;
  typedef int CGSWorkspace;
  typedef void* CGSValue;

  /* Get the default connection for the current process. */
  extern CGSConnection _CGSDefaultConnection(void);

  /* /MJakubowski/ Get connection for given PSN */
  extern CGError CGSGetConnectionIDForPSN(const CGSConnection connection, ProcessSerialNumber * psn, CGSConnection * targetConnection);

  // Get on-screen window counts and lists.
  extern CGError CGSGetOnScreenWindowCount(const CGSConnection cid, CGSConnection targetCID, int* outCount);
  extern CGError CGSGetOnScreenWindowList(const CGSConnection cid, CGSConnection targetCID, int count, int* list, int* outCount);
  // Position
  extern CGError CGSGetWindowBounds(CGSConnection cid, CGSWindowID wid, CGRect *outBounds);
  extern CGError CGSGetScreenRectForWindow(const CGSConnection cid, CGSWindow wid, CGRect *outRect);

  // Properties
  extern CGError CGSGetWindowProperty(const CGSConnection cid, CGSWindow wid, CGSValue key, CGSValue *outValue);


#ifdef __cplusplus
}
#endif

/* QCFString from Qt */
#include <QString>
template <typename T>
class QCFType
{
public:
    inline QCFType(const T &t = 0) : type(t) {}
    inline QCFType(const QCFType &helper) : type(helper.type) { if (type) CFRetain(type); }
    inline ~QCFType() { if (type) CFRelease(type); }
    inline operator T() { return type; }
    inline QCFType operator =(const QCFType &helper)
    {
        if (helper.type)
            CFRetain(helper.type);
        CFTypeRef type2 = type;
        type = helper.type;
        if (type2)
            CFRelease(type2);
        return *this;
    }
    inline T *operator&() { return &type; }
    static QCFType constructFromGet(const T &t)
    {
        CFRetain(t);
        return QCFType<T>(t);
    }
protected:
    T type;
};

class QCFString : public QCFType<CFStringRef>
{
public:
    inline QCFString(const QString &str) : QCFType<CFStringRef>(0), string(str) {}
    inline QCFString(const CFStringRef cfstr = 0) : QCFType<CFStringRef>(cfstr) {}
    inline QCFString(const QCFType<CFStringRef> &other) : QCFType<CFStringRef>(other) {}
    operator QString() const;
    operator CFStringRef() const;
    static QString toQString(CFStringRef cfstr);
    static CFStringRef toCFStringRef(const QString &str);
private:
    QString string;
};

#endif // QXTWINDOWSYSTEM_MAC_H
```

### `./qxtwindowsystem_win.cpp`

```cpp
﻿#include "qxtwindowsystem.h"
/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#ifndef _WIN32_WINNT
#define _WIN32_WINNT 0x0500
#endif
#include <qt_windows.h>
#include <qglobal.h> // QT_WA

static WindowList qxt_Windows;

BOOL CALLBACK qxt_EnumWindowsProc(HWND hwnd, LPARAM lParam)
{
    Q_UNUSED(lParam);
    if (::IsWindowVisible(hwnd))
        qxt_Windows += (WId)hwnd;
    return true;
}

WindowList QxtWindowSystem::windows()
{
    qxt_Windows.clear();
    HDESK hdesk = ::OpenInputDesktop(0, false, DESKTOP_READOBJECTS);
    ::EnumDesktopWindows(hdesk, qxt_EnumWindowsProc, 0);
    ::CloseDesktop(hdesk);
    return qxt_Windows;
}

WId QxtWindowSystem::activeWindow()
{
    return (WId)::GetForegroundWindow();
}

WId QxtWindowSystem::findWindow(const QString& title)
{
#if QT_VERSION < QT_VERSION_CHECK(5,0,0)
    QT_WA({
        return (WId)::FindWindow(NULL, (TCHAR*)title.utf16());
    }, {
        return (WId)::FindWindowA(NULL, title.toLocal8Bit());
    });
#else
    return (WId)::FindWindow(NULL, (TCHAR*)title.utf16());
#endif
}

WId QxtWindowSystem::windowAt(const QPoint& pos)
{
    POINT pt;
    pt.x = pos.x();
    pt.y = pos.y();
    return (WId)::WindowFromPoint(pt);
}

QString QxtWindowSystem::windowTitle(WId window)
{
    QString title;
    int len = ::GetWindowTextLength((HWND)window);
    if (len >= 0)
    {
        TCHAR* buf = new TCHAR[len+1];
        len = ::GetWindowText((HWND)window, buf, len+1);
#if QT_VERSION < QT_VERSION_CHECK(5,0,0)
        QT_WA({
            title = QString::fromUtf16((const ushort*)buf, len);
        }, {
            title = QString::fromLocal8Bit((const char*)buf, len);
        });
#else
            title = QString::fromUtf16((const ushort*)buf, len);
#endif
        delete[] buf;
    }
    return title;
}

QRect QxtWindowSystem::windowGeometry(WId window)
{
    RECT rc;
    QRect rect;
    if (::GetWindowRect((HWND)window, &rc))
    {
        rect.setTop(rc.top);
        rect.setBottom(rc.bottom);
        rect.setLeft(rc.left);
        rect.setRight(rc.right);
    }
    return rect;
}

uint QxtWindowSystem::idleTime()
{
    uint idle = -1;
    LASTINPUTINFO info;
    info.cbSize = sizeof(LASTINPUTINFO);
    if (::GetLastInputInfo(&info))
        idle = ::GetTickCount() - info.dwTime;
    return idle;
}
```

### `./qxtwindowsystem_x11.cpp`

```cpp
#include "qxtwindowsystem.h"
/****************************************************************************
** Copyright (c) 2006 - 2011, the LibQxt project.
** See the Qxt AUTHORS file for a list of authors and copyright holders.
** All rights reserved.
**
** Redistribution and use in source and binary forms, with or without
** modification, are permitted provided that the following conditions are met:
**     * Redistributions of source code must retain the above copyright
**       notice, this list of conditions and the following disclaimer.
**     * Redistributions in binary form must reproduce the above copyright
**       notice, this list of conditions and the following disclaimer in the
**       documentation and/or other materials provided with the distribution.
**     * Neither the name of the LibQxt project nor the
**       names of its contributors may be used to endorse or promote products
**       derived from this software without specific prior written permission.
**
** THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS" AND
** ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE IMPLIED
** WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
** DISCLAIMED. IN NO EVENT SHALL <COPYRIGHT HOLDER> BE LIABLE FOR ANY
** DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL DAMAGES
** (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES;
** LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND
** ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT
** (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
** SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
**
** <http://libqxt.org>  <foundation@libqxt.org>
*****************************************************************************/

#include <QLibrary>
#include <QX11Info>
#include <X11/Xutil.h>

static WindowList qxt_getWindows(Atom prop)
{
    WindowList res;
    Atom type = 0;
    int format = 0;
    uchar* data = 0;
    ulong count, after;
    Display* display = QX11Info::display();
    Window window = QX11Info::appRootWindow();
    if (XGetWindowProperty(display, window, prop, 0, 1024 * sizeof(Window) / 4, False, AnyPropertyType,
                           &type, &format, &count, &after, &data) == Success)
    {
        Window* list = reinterpret_cast<Window*>(data);
        for (uint i = 0; i < count; ++i)
            res += list[i];
        if (data)
            XFree(data);
    }
    return res;
}

WindowList QxtWindowSystem::windows()
{
    static Atom net_clients = 0;
    if (!net_clients)
        net_clients = XInternAtom(QX11Info::display(), "_NET_CLIENT_LIST_STACKING", True);

    return qxt_getWindows(net_clients);
}

WId QxtWindowSystem::activeWindow()
{
    static Atom net_active = 0;
    if (!net_active)
        net_active = XInternAtom(QX11Info::display(), "_NET_ACTIVE_WINDOW", True);

    return qxt_getWindows(net_active).value(0);
}

WId QxtWindowSystem::findWindow(const QString& title)
{
    Window result = 0;
    WindowList list = windows();
    foreach (const Window &wid, list)
    {
        if (windowTitle(wid) == title)
        {
            result = wid;
            break;
        }
    }
    return result;
}

WId QxtWindowSystem::windowAt(const QPoint& pos)
{
    Window result = 0;
    WindowList list = windows();
    for (int i = list.size() - 1; i >= 0; --i)
    {
        WId wid = list.at(i);
        if (windowGeometry(wid).contains(pos))
        {
            result = wid;
            break;
        }
    }
    return result;
}

QString QxtWindowSystem::windowTitle(WId window)
{
    QString name;
    char* str = 0;
    if (XFetchName(QX11Info::display(), window, &str))
        name = QString::fromLatin1(str);
    if (str)
        XFree(str);
    return name;
}

QRect QxtWindowSystem::windowGeometry(WId window)
{
    int x, y;
    uint width, height, border, depth;
    Window root, child;
    Display* display = QX11Info::display();
    XGetGeometry(display, window, &root, &x, &y, &width, &height, &border, &depth);
    XTranslateCoordinates(display, window, root, x, y, &x, &y, &child);

    static Atom net_frame = 0;
    if (!net_frame)
        net_frame = XInternAtom(QX11Info::display(), "_NET_FRAME_EXTENTS", True);

    QRect rect(x, y, width, height);
    Atom type = 0;
    int format = 0;
    uchar* data = 0;
    ulong count, after;
    if (XGetWindowProperty(display, window, net_frame, 0, 4, False, AnyPropertyType,
                           &type, &format, &count, &after, &data) == Success)
    {
        // _NET_FRAME_EXTENTS, left, right, top, bottom, CARDINAL[4]/32
        if (count == 4)
        {
            long* extents = reinterpret_cast<long*>(data);
            rect.adjust(-extents[0], -extents[2], extents[1], extents[3]);
        }
        if (data)
            XFree(data);
    }
    return rect;
}

typedef struct {
    Window  window;     /* screen saver window - may not exist */
    int     state;      /* ScreenSaverOff, ScreenSaverOn, ScreenSaverDisabled*/
    int     kind;       /* ScreenSaverBlanked, ...Internal, ...External */
    unsigned long    til_or_since;   /* time til or since screen saver */
    unsigned long    idle;      /* total time since last user input */
    unsigned long   eventMask; /* currently selected events for this client */
} XScreenSaverInfo;

typedef XScreenSaverInfo* (*XScreenSaverAllocInfo)();
typedef Status (*XScreenSaverQueryInfo)(Display* display, Drawable drawable, XScreenSaverInfo* info);

static XScreenSaverAllocInfo _xScreenSaverAllocInfo = 0;
static XScreenSaverQueryInfo _xScreenSaverQueryInfo = 0;

uint QxtWindowSystem::idleTime()
{
    static bool xssResolved = false;
    if (!xssResolved) {
        QLibrary xssLib(QLatin1String("Xss"), 1);
        if (xssLib.load()) {
            _xScreenSaverAllocInfo = (XScreenSaverAllocInfo) xssLib.resolve("XScreenSaverAllocInfo");
            _xScreenSaverQueryInfo = (XScreenSaverQueryInfo) xssLib.resolve("XScreenSaverQueryInfo");
            xssResolved = true;
        }
    }

    uint idle = 0;
    if (xssResolved)
    {
        XScreenSaverInfo* info = _xScreenSaverAllocInfo();
        const int screen = QX11Info::appScreen();
        unsigned long rootWindow = (unsigned long)QX11Info::appRootWindow(screen);
        _xScreenSaverQueryInfo(QX11Info::display(), (Drawable) rootWindow, info);
        idle = info->idle;
        if (info)
            XFree(info);
    }
    return idle;
}
```

### `./x11info.cpp`

```cpp
/*
 * Copyright (C) 2013  Il'inykh Sergey (rion)
 *
 * This program is free software; you can redistribute it and/or
 * modify it under the terms of the GNU General Public License
 * as published by the Free Software Foundation; either version 2
 * of the License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this library; if not, write to the Free Software
 * Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA
 *
 */

#include "x11info.h"

#ifdef HAVE_QT5
# include <X11/Xlib.h>
# include <xcb/xcb.h>
# include <QtGlobal>
#else
# include <QX11Info>
#endif


Display* X11Info::display()
{
#ifdef HAVE_QT5
	if (!_display) {
		_display = XOpenDisplay(NULL);
	}
	return _display;
#else
	return QX11Info::display();
#endif
}

unsigned long X11Info::appRootWindow(int screen)
{
#ifdef HAVE_QT5
	return screen == -1?
				XDefaultRootWindow(display()) :
				XRootWindowOfScreen(XScreenOfDisplay(display(), screen));
#else
	return QX11Info::appRootWindow(screen);
#endif
}

int X11Info::appScreen()
{
#ifdef HAVE_QT5
	#error X11Info::appScreen not implemented for Qt5! You must skip this plugin.
#else
	return QX11Info::appScreen();
#endif
}

#ifdef HAVE_QT5
xcb_connection_t *X11Info::xcbConnection()
{
	if (!_xcb) {
		_xcb = xcb_connect(NULL, &_xcbPreferredScreen);
		Q_ASSERT(_xcb);
	}
	return _xcb;
}

xcb_connection_t* X11Info::_xcb = 0;
#endif

Display* X11Info::_display = 0;
int X11Info::_xcbPreferredScreen = 0;
```

### `./x11info.h`

```cpp
/*
 * Copyright (C) 2013  Il'inykh Sergey (rion)
 *
 * This program is free software; you can redistribute it and/or
 * modify it under the terms of the GNU General Public License
 * as published by the Free Software Foundation; either version 2
 * of the License, or (at your option) any later version.
 *
 * This program is distributed in the hope that it will be useful,
 * but WITHOUT ANY WARRANTY; without even the implied warranty of
 * MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
 * GNU General Public License for more details.
 *
 * You should have received a copy of the GNU General Public License
 * along with this library; if not, write to the Free Software
 * Foundation, Inc., 51 Franklin Street, Fifth Floor, Boston, MA 02110-1301 USA
 *
 */

#ifndef X11INFO_H
#define X11INFO_H

typedef struct _XDisplay Display;
#ifdef HAVE_QT5
typedef struct xcb_connection_t xcb_connection_t;
#endif

class X11Info
{
	static Display *_display;
#ifdef HAVE_QT5
	static xcb_connection_t *_xcb;
#endif
	static int _xcbPreferredScreen;

public:
	static Display* display();
	static unsigned long appRootWindow(int screen = -1);
	static int appScreen();
#ifdef HAVE_QT5
	static xcb_connection_t* xcbConnection();
	static inline int xcbPreferredScreen() { return _xcbPreferredScreen; }
#endif
};

#endif // X11INFO_H
```

### `./3rd_qxtglobalshortcut.pri`

```makefile
DEFINES += BUILD_QXT_CORE BUILD_QXT_GUI

HEADERS += $$PWD/qxtglobal.h
HEADERS += $$PWD/qxtglobalshortcut_p.h
HEADERS += $$PWD/qxtglobalshortcut.h

SOURCES += $$PWD/qxtglobal.cpp
SOURCES += $$PWD/qxtglobalshortcut.cpp

unix:!macx {
CONFIG += X11
QT += x11extras
HEADERS += $$PWD/x11info.h
SOURCES += $$PWD/qxtwindowsystem_x11.cpp
SOURCES += $$PWD/qxtglobalshortcut_x11.cpp
}

macx {
QMAKE_LFLAGS += -framework Carbon -framework CoreFoundation
HEADERS += $$PWD/qxtwindowsystem_mac.h
SOURCES += $$PWD/qxtwindowsystem_mac.cpp
SOURCES += $$PWD/qxtglobalshortcut_mac.cpp
}

win32 {
LIBS += -luser32
SOURCES += $$PWD/qxtwindowsystem_win.cpp
SOURCES += $$PWD/qxtglobalshortcut_win.cpp
}
```
