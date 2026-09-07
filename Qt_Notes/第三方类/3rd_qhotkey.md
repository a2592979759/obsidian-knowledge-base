---
tags:
  - Qt
  - 第三方类
---
# 全局热键库 (`3rd_qhotkey`)

## 分类

- 类型: 第三方类 `third`
- 源码目录: `third/3rd_qhotkey`

## 技术要点

**用到的 Qt 类**: QHotkey QHotkeyPrivate QHotkeyPrivateX11 QString QKeySequence QHotkeyPrivateWin QHotkeyPrivateMac QObject QTimer QX11Info QThread QByteArray QNativeInterface QX11Application QMetaMethod QAbstractNativeEventFilter QStringLiteral QDebug QMetaObject QHash

**特性/模块**: Q_OBJECT、connect

## 完整源码

### `./qhotkey.cpp`

```cpp
#include "qhotkey.h"
#include "qhotkey_p.h"
#include <QCoreApplication>
#include <QAbstractEventDispatcher>
#include <QMetaMethod>
#include <QThread>
#include <QDebug>

Q_LOGGING_CATEGORY(logQHotkey, "QHotkey")

void QHotkey::addGlobalMapping(const QKeySequence &shortcut, QHotkey::NativeShortcut nativeShortcut)
{
#if QT_VERSION >= QT_VERSION_CHECK(6, 0, 0)
	const int key = shortcut[0].toCombined();
#else
	const int key = shortcut[0];
#endif

	QMetaObject::invokeMethod(QHotkeyPrivate::instance(), "addMappingInvoked", Qt::QueuedConnection,
							  Q_ARG(Qt::Key, Qt::Key(key & ~Qt::KeyboardModifierMask)),
							  Q_ARG(Qt::KeyboardModifiers, Qt::KeyboardModifiers(key & Qt::KeyboardModifierMask)),
							  Q_ARG(QHotkey::NativeShortcut, nativeShortcut));
}

bool QHotkey::isPlatformSupported()
{
	return QHotkeyPrivate::isPlatformSupported();
}

QHotkey::QHotkey(QObject *parent) :
	QObject(parent),
	_keyCode(Qt::Key_unknown),
	_modifiers(Qt::NoModifier),
	_registered(false)
{}

QHotkey::QHotkey(const QKeySequence &shortcut, bool autoRegister, QObject *parent) :
	QHotkey(parent)
{
	setShortcut(shortcut, autoRegister);
}

QHotkey::QHotkey(Qt::Key keyCode, Qt::KeyboardModifiers modifiers, bool autoRegister, QObject *parent) :
	QHotkey(parent)
{
	setShortcut(keyCode, modifiers, autoRegister);
}

QHotkey::QHotkey(QHotkey::NativeShortcut shortcut, bool autoRegister, QObject *parent) :
	QHotkey(parent)
{
	setNativeShortcut(shortcut, autoRegister);
}

QHotkey::~QHotkey()
{
	if(_registered)
		QHotkeyPrivate::instance()->removeShortcut(this);
}

QKeySequence QHotkey::shortcut() const
{
	if(_keyCode == Qt::Key_unknown)
		return QKeySequence();

#if QT_VERSION >= QT_VERSION_CHECK(6, 0, 0)
	return QKeySequence((_keyCode | _modifiers).toCombined());
#else
	return QKeySequence(static_cast<int>(_keyCode | _modifiers));
#endif
}

Qt::Key QHotkey::keyCode() const
{
	return _keyCode;
}

Qt::KeyboardModifiers QHotkey::modifiers() const
{
	return _modifiers;
}

QHotkey::NativeShortcut QHotkey::currentNativeShortcut() const
{
	return _nativeShortcut;
}

bool QHotkey::isRegistered() const
{
	return _registered;
}

bool QHotkey::setShortcut(const QKeySequence &shortcut, bool autoRegister)
{
	if(shortcut.isEmpty())
		return resetShortcut();
	if(shortcut.count() > 1) {
		qCWarning(logQHotkey, "Keysequences with multiple shortcuts are not allowed! "
							  "Only the first shortcut will be used!");
	}

#if QT_VERSION >= QT_VERSION_CHECK(6, 0, 0)
	const int key = shortcut[0].toCombined();
#else
	const int key = shortcut[0];
#endif

	return setShortcut(Qt::Key(key & ~Qt::KeyboardModifierMask),
			Qt::KeyboardModifiers(key & Qt::KeyboardModifierMask),
			autoRegister);
}

bool QHotkey::setShortcut(Qt::Key keyCode, Qt::KeyboardModifiers modifiers, bool autoRegister)
{
	if(_registered) {
		if(autoRegister) {
			if(!QHotkeyPrivate::instance()->removeShortcut(this))
				return false;
		} else
			return false;
	}

	if(keyCode == Qt::Key_unknown) {
		_keyCode = Qt::Key_unknown;
		_modifiers = Qt::NoModifier;
		_nativeShortcut = NativeShortcut();
		return true;
	}

	_keyCode = keyCode;
	_modifiers = modifiers;
	_nativeShortcut = QHotkeyPrivate::instance()->nativeShortcut(keyCode, modifiers);
	if(_nativeShortcut.isValid()) {
		if(autoRegister)
			return QHotkeyPrivate::instance()->addShortcut(this);
		return true;
	}

	qCWarning(logQHotkey) << "Unable to map shortcut to native keys. Key:" << keyCode << "Modifiers:" << modifiers;
	_keyCode = Qt::Key_unknown;
	_modifiers = Qt::NoModifier;
	_nativeShortcut = NativeShortcut();
	return false;
}

bool QHotkey::resetShortcut()
{
	if(_registered &&
	   !QHotkeyPrivate::instance()->removeShortcut(this)) {
		return false;
	}

	_keyCode = Qt::Key_unknown;
	_modifiers = Qt::NoModifier;
	_nativeShortcut = NativeShortcut();
	return true;
}

bool QHotkey::setNativeShortcut(QHotkey::NativeShortcut nativeShortcut, bool autoRegister)
{
	if(_registered) {
		if(autoRegister) {
			if(!QHotkeyPrivate::instance()->removeShortcut(this))
				return false;
		} else
			return false;
	}

	if(nativeShortcut.isValid()) {
		_keyCode = Qt::Key_unknown;
		_modifiers = Qt::NoModifier;
		_nativeShortcut = nativeShortcut;
		if(autoRegister)
			return QHotkeyPrivate::instance()->addShortcut(this);
		return true;
	} 

	_keyCode = Qt::Key_unknown;
	_modifiers = Qt::NoModifier;
	_nativeShortcut = NativeShortcut();
	return true;
}

bool QHotkey::setRegistered(bool registered)
{
	if(_registered && !registered)
		return QHotkeyPrivate::instance()->removeShortcut(this);
	if(!_registered && registered) {
		if(!_nativeShortcut.isValid())
			return false;
		return QHotkeyPrivate::instance()->addShortcut(this);
	}
	return true;
}



// ---------- QHotkeyPrivate implementation ----------

QHotkeyPrivate::QHotkeyPrivate()
{
	Q_ASSERT_X(qApp, Q_FUNC_INFO, "QHotkey requires QCoreApplication to be instantiated");
	qApp->eventDispatcher()->installNativeEventFilter(this);
}

QHotkeyPrivate::~QHotkeyPrivate()
{
	if(!shortcuts.isEmpty())
		qCWarning(logQHotkey) << "QHotkeyPrivate destroyed with registered shortcuts!";
	if(qApp && qApp->eventDispatcher())
		qApp->eventDispatcher()->removeNativeEventFilter(this);
}

QHotkey::NativeShortcut QHotkeyPrivate::nativeShortcut(Qt::Key keycode, Qt::KeyboardModifiers modifiers)
{
	Qt::ConnectionType conType = (QThread::currentThread() == thread() ?
									  Qt::DirectConnection :
									  Qt::BlockingQueuedConnection);
	QHotkey::NativeShortcut res;
	if(!QMetaObject::invokeMethod(this, "nativeShortcutInvoked", conType,
								  Q_RETURN_ARG(QHotkey::NativeShortcut, res),
								  Q_ARG(Qt::Key, keycode),
								  Q_ARG(Qt::KeyboardModifiers, modifiers))) {
		return QHotkey::NativeShortcut();
	}
	return res;
}

bool QHotkeyPrivate::addShortcut(QHotkey *hotkey)
{
	if(hotkey->_registered)
		return false;

	Qt::ConnectionType conType = (QThread::currentThread() == thread() ?
									  Qt::DirectConnection :
									  Qt::BlockingQueuedConnection);
	bool res = false;
	if(!QMetaObject::invokeMethod(this, "addShortcutInvoked", conType,
								  Q_RETURN_ARG(bool, res),
								  Q_ARG(QHotkey*, hotkey))) {
		return false;
	}

	if(res)
		emit hotkey->registeredChanged(true);
	return res;
}

bool QHotkeyPrivate::removeShortcut(QHotkey *hotkey)
{
	if(!hotkey->_registered)
		return false;

	Qt::ConnectionType conType = (QThread::currentThread() == thread() ?
									  Qt::DirectConnection :
									  Qt::BlockingQueuedConnection);
	bool res = false;
	if(!QMetaObject::invokeMethod(this, "removeShortcutInvoked", conType,
								  Q_RETURN_ARG(bool, res),
								  Q_ARG(QHotkey*, hotkey))) {
		return false;
	}

	if(res)
		emit hotkey->registeredChanged(false);
	return res;
}

void QHotkeyPrivate::activateShortcut(QHotkey::NativeShortcut shortcut)
{
	QMetaMethod signal = QMetaMethod::fromSignal(&QHotkey::activated);
	for(QHotkey *hkey : shortcuts.values(shortcut))
		signal.invoke(hkey, Qt::QueuedConnection);
}

void QHotkeyPrivate::releaseShortcut(QHotkey::NativeShortcut shortcut)
{
	QMetaMethod signal = QMetaMethod::fromSignal(&QHotkey::released);
	for(QHotkey *hkey : shortcuts.values(shortcut))
		signal.invoke(hkey, Qt::QueuedConnection);
}

void QHotkeyPrivate::addMappingInvoked(Qt::Key keycode, Qt::KeyboardModifiers modifiers, QHotkey::NativeShortcut nativeShortcut)
{
	mapping.insert({keycode, modifiers}, nativeShortcut);
}

bool QHotkeyPrivate::addShortcutInvoked(QHotkey *hotkey)
{
	QHotkey::NativeShortcut shortcut = hotkey->_nativeShortcut;

	if(!shortcuts.contains(shortcut)) {
		if(!registerShortcut(shortcut)) {
			qCWarning(logQHotkey) << QHotkey::tr("Failed to register %1. Error: %2").arg(hotkey->shortcut().toString(), error);
			return false;
		}
	}

	shortcuts.insert(shortcut, hotkey);
	hotkey->_registered = true;
	return true;
}

bool QHotkeyPrivate::removeShortcutInvoked(QHotkey *hotkey)
{
	QHotkey::NativeShortcut shortcut = hotkey->_nativeShortcut;

	if(shortcuts.remove(shortcut, hotkey) == 0)
		return false;
	hotkey->_registered = false;
	emit hotkey->registeredChanged(true);
	if(shortcuts.count(shortcut) == 0) {
		if (!unregisterShortcut(shortcut)) {
			qCWarning(logQHotkey) << QHotkey::tr("Failed to unregister %1. Error: %2").arg(hotkey->shortcut().toString(), error);
			return false;
		}
		return true;
	}
	return true;
}

QHotkey::NativeShortcut QHotkeyPrivate::nativeShortcutInvoked(Qt::Key keycode, Qt::KeyboardModifiers modifiers)
{
	if(mapping.contains({keycode, modifiers}))
		return mapping.value({keycode, modifiers});

	bool ok1 = false;
	auto k = nativeKeycode(keycode, ok1);
	bool ok2 = false;
	auto m = nativeModifiers(modifiers, ok2);
	if(ok1 && ok2)
		return {k, m};
	return {};
}



QHotkey::NativeShortcut::NativeShortcut() :
	key(),
	modifier(),
	valid(false)
{}

QHotkey::NativeShortcut::NativeShortcut(quint32 key, quint32 modifier) :
	key(key),
	modifier(modifier),
	valid(true)
{}

bool QHotkey::NativeShortcut::isValid() const
{
	return valid;
}

bool QHotkey::NativeShortcut::operator ==(QHotkey::NativeShortcut other) const
{
	return (key == other.key) &&
		   (modifier == other.modifier) &&
		   valid == other.valid;
}

bool QHotkey::NativeShortcut::operator !=(QHotkey::NativeShortcut other) const
{
	return (key != other.key) ||
		   (modifier != other.modifier) ||
		   valid != other.valid;
}

QHOTKEY_HASH_SEED qHash(QHotkey::NativeShortcut key)
{
	return qHash(key.key) ^ qHash(key.modifier);
}

QHOTKEY_HASH_SEED qHash(QHotkey::NativeShortcut key, QHOTKEY_HASH_SEED seed)
{
	return qHash(key.key, seed) ^ qHash(key.modifier, seed);
}
```

### `./qhotkey.h`

```cpp
#ifndef QHOTKEY_H
#define QHOTKEY_H

#include <QObject>
#include <QKeySequence>
#include <QPair>
#include <QLoggingCategory>

#ifdef QHOTKEY_SHARED
#	ifdef QHOTKEY_LIBRARY
#		define QHOTKEY_EXPORT Q_DECL_EXPORT
#	else
#		define QHOTKEY_EXPORT Q_DECL_IMPORT
#	endif
#else
#	define QHOTKEY_EXPORT
#endif

#if QT_VERSION >= QT_VERSION_CHECK(6, 0, 0)
	#define QHOTKEY_HASH_SEED size_t
#else
	#define QHOTKEY_HASH_SEED uint
#endif

//! A class to define global, systemwide Hotkeys
class QHOTKEY_EXPORT QHotkey : public QObject
{
	Q_OBJECT
	//! @private
	friend class QHotkeyPrivate;

	//! Specifies whether this hotkey is currently registered or not
	Q_PROPERTY(bool registered READ isRegistered WRITE setRegistered NOTIFY registeredChanged)
	//! Holds the shortcut this hotkey will be triggered on
	Q_PROPERTY(QKeySequence shortcut READ shortcut WRITE setShortcut RESET resetShortcut)

public:
	//! Defines shortcut with native keycodes
	class QHOTKEY_EXPORT NativeShortcut {
	public:
		//! The native keycode
		quint32 key;
		//! The native modifiers
		quint32 modifier;

		//! Creates an invalid native shortcut
		NativeShortcut();
		//! Creates a valid native shortcut, with the given key and modifiers
		NativeShortcut(quint32 key, quint32 modifier = 0);

		//! Checks, whether this shortcut is valid or not
		bool isValid() const;

		//! Equality operator
		bool operator ==(NativeShortcut other) const;
		//! Inequality operator
		bool operator !=(NativeShortcut other) const;

	private:
		bool valid;
	};

	//! Adds a global mapping of a key sequence to a replacement native shortcut
	static void addGlobalMapping(const QKeySequence &shortcut, NativeShortcut nativeShortcut);

	//! Checks if global shortcuts are supported by the current platform
	static bool isPlatformSupported();

	//! Default Constructor
	explicit QHotkey(QObject *parent = nullptr);
	//! Constructs a hotkey with a shortcut and optionally registers it
	explicit QHotkey(const QKeySequence &shortcut, bool autoRegister = false, QObject *parent = nullptr);
	//! Constructs a hotkey with a key and modifiers and optionally registers it
	explicit QHotkey(Qt::Key keyCode, Qt::KeyboardModifiers modifiers, bool autoRegister = false, QObject *parent = nullptr);
	//! Constructs a hotkey from a native shortcut and optionally registers it
	explicit QHotkey(NativeShortcut shortcut, bool autoRegister = false, QObject *parent = nullptr);
	~QHotkey() override;

	//! @readAcFn{QHotkey::registered}
	bool isRegistered() const;
	//! @readAcFn{QHotkey::shortcut}
	QKeySequence shortcut() const;
	//! @readAcFn{QHotkey::shortcut} - the key only
	Qt::Key keyCode() const;
	//! @readAcFn{QHotkey::shortcut} - the modifiers only
	Qt::KeyboardModifiers modifiers() const;

	//! Get the current native shortcut
	NativeShortcut currentNativeShortcut() const;

public slots:
	//! @writeAcFn{QHotkey::registered}
	bool setRegistered(bool registered);

	//! @writeAcFn{QHotkey::shortcut}
	bool setShortcut(const QKeySequence &shortcut, bool autoRegister = false);
	//! @writeAcFn{QHotkey::shortcut}
	bool setShortcut(Qt::Key keyCode, Qt::KeyboardModifiers modifiers, bool autoRegister = false);
	//! @resetAcFn{QHotkey::shortcut}
	bool resetShortcut();

	//! Set this hotkey to a native shortcut
	bool setNativeShortcut(QHotkey::NativeShortcut nativeShortcut, bool autoRegister = false);

signals:
	//! Will be emitted if the shortcut is pressed
	void activated(QPrivateSignal);

	//! Will be emitted if the shortcut press is released
	void released(QPrivateSignal);

	//! @notifyAcFn{QHotkey::registered}
	void registeredChanged(bool registered);

private:
	Qt::Key _keyCode;
	Qt::KeyboardModifiers _modifiers;

	NativeShortcut _nativeShortcut;
	bool _registered;
};

QHOTKEY_HASH_SEED QHOTKEY_EXPORT qHash(QHotkey::NativeShortcut key);
QHOTKEY_HASH_SEED QHOTKEY_EXPORT qHash(QHotkey::NativeShortcut key, QHOTKEY_HASH_SEED seed);

QHOTKEY_EXPORT Q_DECLARE_LOGGING_CATEGORY(logQHotkey)

Q_DECLARE_METATYPE(QHotkey::NativeShortcut)

#endif // QHOTKEY_H
```

### `./qhotkey_mac.cpp`

```cpp
#include "qhotkey.h"
#include "qhotkey_p.h"
#include <Carbon/Carbon.h>
#include <QDebug>

class QHotkeyPrivateMac : public QHotkeyPrivate
{
public:
	// QAbstractNativeEventFilter interface
	bool nativeEventFilter(const QByteArray &eventType, void *message, _NATIVE_EVENT_RESULT *result) override;

	static OSStatus hotkeyPressEventHandler(EventHandlerCallRef nextHandler, EventRef event, void* data);
	static OSStatus hotkeyReleaseEventHandler(EventHandlerCallRef nextHandler, EventRef event, void* data);

protected:
	// QHotkeyPrivate interface
	quint32 nativeKeycode(Qt::Key keycode, bool &ok) Q_DECL_OVERRIDE;
	quint32 nativeModifiers(Qt::KeyboardModifiers modifiers, bool &ok) Q_DECL_OVERRIDE;
	bool registerShortcut(QHotkey::NativeShortcut shortcut) Q_DECL_OVERRIDE;
	bool unregisterShortcut(QHotkey::NativeShortcut shortcut) Q_DECL_OVERRIDE;

private:
	static bool isHotkeyHandlerRegistered;
	static QHash<QHotkey::NativeShortcut, EventHotKeyRef> hotkeyRefs;
};
NATIVE_INSTANCE(QHotkeyPrivateMac)

bool QHotkeyPrivate::isPlatformSupported()
{
	return true;
}

bool QHotkeyPrivateMac::isHotkeyHandlerRegistered = false;
QHash<QHotkey::NativeShortcut, EventHotKeyRef> QHotkeyPrivateMac::hotkeyRefs;

bool QHotkeyPrivateMac::nativeEventFilter(const QByteArray &eventType, void *message, _NATIVE_EVENT_RESULT *result)
{
	Q_UNUSED(eventType)
	Q_UNUSED(message)
	Q_UNUSED(result)
	return false;
}

quint32 QHotkeyPrivateMac::nativeKeycode(Qt::Key keycode, bool &ok)
{
	// Constants found in NSEvent.h from AppKit.framework
	ok = true;
	switch (keycode) {
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
	case Qt::Key_Escape:
		return kVK_Escape;
	case Qt::Key_CapsLock:
		return kVK_CapsLock;
	case Qt::Key_Option:
		return kVK_Option;
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
		ok = false;
		break;
	}

	UTF16Char ch = keycode;

	CFDataRef currentLayoutData;
	TISInputSourceRef currentKeyboard = TISCopyCurrentASCIICapableKeyboardLayoutInputSource();

	if (currentKeyboard == NULL)
		return 0;

	currentLayoutData = (CFDataRef)TISGetInputSourceProperty(currentKeyboard, kTISPropertyUnicodeKeyLayoutData);
	CFRelease(currentKeyboard);
	if (currentLayoutData == NULL)
		return 0;

	UCKeyboardLayout* header = (UCKeyboardLayout*)CFDataGetBytePtr(currentLayoutData);
	UCKeyboardTypeHeader* table = header->keyboardTypeList;

	uint8_t *data = (uint8_t*)header;
	for (quint32 i=0; i < header->keyboardTypeCount; i++) {
		UCKeyStateRecordsIndex* stateRec = 0;
		if (table[i].keyStateRecordsIndexOffset != 0) {
			stateRec = reinterpret_cast<UCKeyStateRecordsIndex*>(data + table[i].keyStateRecordsIndexOffset);
			if (stateRec->keyStateRecordsIndexFormat != kUCKeyStateRecordsIndexFormat) stateRec = 0;
		}

		UCKeyToCharTableIndex* charTable = reinterpret_cast<UCKeyToCharTableIndex*>(data + table[i].keyToCharTableIndexOffset);
		if (charTable->keyToCharTableIndexFormat != kUCKeyToCharTableIndexFormat) continue;

		for (quint32 j=0; j < charTable->keyToCharTableCount; j++) {
			UCKeyOutput* keyToChar = reinterpret_cast<UCKeyOutput*>(data + charTable->keyToCharTableOffsets[j]);
			for (quint32 k=0; k < charTable->keyToCharTableSize; k++) {
				if (keyToChar[k] & kUCKeyOutputTestForIndexMask) {
					long idx = keyToChar[k] & kUCKeyOutputGetIndexMask;
					if (stateRec && idx < stateRec->keyStateRecordCount) {
						UCKeyStateRecord* rec = reinterpret_cast<UCKeyStateRecord*>(data + stateRec->keyStateRecordOffsets[idx]);
						if (rec->stateZeroCharData == ch) {
							ok = true;
							return k;
						}
					}
				}
				else if (!(keyToChar[k] & kUCKeyOutputSequenceIndexMask) && keyToChar[k] < 0xFFFE) {
					if (keyToChar[k] == ch) {
						ok = true;
						return k;
					}
				}
			}
		}
	}
	return 0;
}

quint32 QHotkeyPrivateMac::nativeModifiers(Qt::KeyboardModifiers modifiers, bool &ok)
{
	quint32 nMods = 0;
	if (modifiers & Qt::ShiftModifier)
		nMods |= shiftKey;
	if (modifiers & Qt::ControlModifier)
		nMods |= cmdKey;
	if (modifiers & Qt::AltModifier)
		nMods |= optionKey;
	if (modifiers & Qt::MetaModifier)
		nMods |= controlKey;
	if (modifiers & Qt::KeypadModifier)
		nMods |= kEventKeyModifierNumLockMask;
	ok = true;
	return nMods;
}

bool QHotkeyPrivateMac::registerShortcut(QHotkey::NativeShortcut shortcut)
{
	if (!this->isHotkeyHandlerRegistered)
	{
		EventTypeSpec pressEventSpec;
		pressEventSpec.eventClass = kEventClassKeyboard;
		pressEventSpec.eventKind = kEventHotKeyPressed;
		InstallApplicationEventHandler(&QHotkeyPrivateMac::hotkeyPressEventHandler, 1, &pressEventSpec, NULL, NULL);

		EventTypeSpec releaseEventSpec;
		releaseEventSpec.eventClass = kEventClassKeyboard;
		releaseEventSpec.eventKind = kEventHotKeyReleased;
		InstallApplicationEventHandler(&QHotkeyPrivateMac::hotkeyReleaseEventHandler, 1, &releaseEventSpec, NULL, NULL);
	}

	EventHotKeyID hkeyID;
	hkeyID.signature = shortcut.key;
	hkeyID.id = shortcut.modifier;

	EventHotKeyRef eventRef = 0;
	OSStatus status = RegisterEventHotKey(shortcut.key,
										  shortcut.modifier,
										  hkeyID,
										  GetApplicationEventTarget(),
										  0,
										  &eventRef);
	if (status != noErr) {
		error = QString::number(status);
		return false;
	} else {
		this->hotkeyRefs.insert(shortcut, eventRef);
		return true;
	}
}

bool QHotkeyPrivateMac::unregisterShortcut(QHotkey::NativeShortcut shortcut)
{
	EventHotKeyRef eventRef = QHotkeyPrivateMac::hotkeyRefs.value(shortcut);
	OSStatus status = UnregisterEventHotKey(eventRef);
	if (status != noErr) {
		error = QString::number(status);
		return false;
	} else {
		this->hotkeyRefs.remove(shortcut);
		return true;
	}
}

OSStatus QHotkeyPrivateMac::hotkeyPressEventHandler(EventHandlerCallRef nextHandler, EventRef event, void* data)
{
	Q_UNUSED(nextHandler);
	Q_UNUSED(data);

	if (GetEventClass(event) == kEventClassKeyboard &&
		GetEventKind(event) == kEventHotKeyPressed) {
		EventHotKeyID hkeyID;
		GetEventParameter(event,
						  kEventParamDirectObject,
						  typeEventHotKeyID,
						  NULL,
						  sizeof(EventHotKeyID),
						  NULL,
						  &hkeyID);
		hotkeyPrivate->activateShortcut({hkeyID.signature, hkeyID.id});
	}

	return noErr;
}

OSStatus QHotkeyPrivateMac::hotkeyReleaseEventHandler(EventHandlerCallRef nextHandler, EventRef event, void* data)
{
	Q_UNUSED(nextHandler);
	Q_UNUSED(data);

	if (GetEventClass(event) == kEventClassKeyboard &&
		GetEventKind(event) == kEventHotKeyReleased) {
		EventHotKeyID hkeyID;
		GetEventParameter(event,
											kEventParamDirectObject,
											typeEventHotKeyID,
											NULL,
											sizeof(EventHotKeyID),
											NULL,
											&hkeyID);
		hotkeyPrivate->releaseShortcut({hkeyID.signature, hkeyID.id});
	}

	return noErr;
}
```

### `./qhotkey_p.h`

```cpp
#ifndef QHOTKEY_P_H
#define QHOTKEY_P_H

#include "qhotkey.h"
#include <QAbstractNativeEventFilter>
#include <QMultiHash>
#include <QMutex>
#include <QGlobalStatic>

#if QT_VERSION >= QT_VERSION_CHECK(6, 0, 0)
	#define _NATIVE_EVENT_RESULT qintptr
#else
	#define _NATIVE_EVENT_RESULT long
#endif

class QHOTKEY_EXPORT QHotkeyPrivate : public QObject, public QAbstractNativeEventFilter
{
	Q_OBJECT

public:
	QHotkeyPrivate();//singleton!!!
	~QHotkeyPrivate();

	static QHotkeyPrivate *instance();
	static bool isPlatformSupported();

	QHotkey::NativeShortcut nativeShortcut(Qt::Key keycode, Qt::KeyboardModifiers modifiers);

	bool addShortcut(QHotkey *hotkey);
	bool removeShortcut(QHotkey *hotkey);

protected:
	void activateShortcut(QHotkey::NativeShortcut shortcut);
	void releaseShortcut(QHotkey::NativeShortcut shortcut);

	virtual quint32 nativeKeycode(Qt::Key keycode, bool &ok) = 0;//platform implement
	virtual quint32 nativeModifiers(Qt::KeyboardModifiers modifiers, bool &ok) = 0;//platform implement

	virtual bool registerShortcut(QHotkey::NativeShortcut shortcut) = 0;//platform implement
	virtual bool unregisterShortcut(QHotkey::NativeShortcut shortcut) = 0;//platform implement

	QString error;

private:
	QHash<QPair<Qt::Key, Qt::KeyboardModifiers>, QHotkey::NativeShortcut> mapping;
	QMultiHash<QHotkey::NativeShortcut, QHotkey*> shortcuts;

	Q_INVOKABLE void addMappingInvoked(Qt::Key keycode, Qt::KeyboardModifiers modifiers, QHotkey::NativeShortcut nativeShortcut);
	Q_INVOKABLE bool addShortcutInvoked(QHotkey *hotkey);
	Q_INVOKABLE bool removeShortcutInvoked(QHotkey *hotkey);
	Q_INVOKABLE QHotkey::NativeShortcut nativeShortcutInvoked(Qt::Key keycode, Qt::KeyboardModifiers modifiers);
};

#define NATIVE_INSTANCE(ClassName) \
	Q_GLOBAL_STATIC(ClassName, hotkeyPrivate) \
	\
	QHotkeyPrivate *QHotkeyPrivate::instance()\
	{\
		return hotkeyPrivate;\
	}

#endif // QHOTKEY_P_H
```

### `./qhotkey_win.cpp`

```cpp
#include "qhotkey.h"
#include "qhotkey_p.h"
#include <qt_windows.h>
#include <algorithm>
#include <QDebug>
#include <QList>
#include <QTimer>

#define HKEY_ID(nativeShortcut) (((nativeShortcut.key ^ (nativeShortcut.modifier << 8)) & 0x0FFF) | 0x7000)

#if !defined(MOD_NOREPEAT)
#define MOD_NOREPEAT 0x4000
#endif

class QHotkeyPrivateWin : public QHotkeyPrivate
{
public:
	QHotkeyPrivateWin();
	// QAbstractNativeEventFilter interface
	bool nativeEventFilter(const QByteArray &eventType, void *message, _NATIVE_EVENT_RESULT *result) override;

protected:
	void pollForHotkeyRelease();
	// QHotkeyPrivate interface
	quint32 nativeKeycode(Qt::Key keycode, bool &ok) Q_DECL_OVERRIDE;
	quint32 nativeModifiers(Qt::KeyboardModifiers modifiers, bool &ok) Q_DECL_OVERRIDE;
	bool registerShortcut(QHotkey::NativeShortcut shortcut) Q_DECL_OVERRIDE;
	bool unregisterShortcut(QHotkey::NativeShortcut shortcut) Q_DECL_OVERRIDE;

private:
	static QString formatWinError(DWORD winError);
	QTimer pollTimer;
	QList<QHotkey::NativeShortcut> polledShortcuts;
};
NATIVE_INSTANCE(QHotkeyPrivateWin)

QHotkeyPrivateWin::QHotkeyPrivateWin(){
	pollTimer.setInterval(50);
	connect(&pollTimer, &QTimer::timeout, this, &QHotkeyPrivateWin::pollForHotkeyRelease);
}

bool QHotkeyPrivate::isPlatformSupported()
{
	return true;
}

bool QHotkeyPrivateWin::nativeEventFilter(const QByteArray &eventType, void *message, _NATIVE_EVENT_RESULT *result)
{
	Q_UNUSED(eventType)
	Q_UNUSED(result)

	MSG* msg = static_cast<MSG*>(message);
	if(msg->message == WM_HOTKEY) {
		QHotkey::NativeShortcut shortcut = {HIWORD(msg->lParam), LOWORD(msg->lParam)};
		this->activateShortcut(shortcut);
		if (this->polledShortcuts.empty())
			this->pollTimer.start();
		this->polledShortcuts.append(shortcut);
	}

	return false;
}

void QHotkeyPrivateWin::pollForHotkeyRelease()
{
	auto it = std::remove_if(this->polledShortcuts.begin(), this->polledShortcuts.end(), [this](const QHotkey::NativeShortcut &shortcut) {
		bool pressed = (GetAsyncKeyState(shortcut.key) & (1 << 15)) != 0;
		if (!pressed)
			this->releaseShortcut(shortcut);
		return !pressed;
	});
	this->polledShortcuts.erase(it, this->polledShortcuts.end());
	if (this->polledShortcuts.empty())
		this->pollTimer.stop();
}

quint32 QHotkeyPrivateWin::nativeKeycode(Qt::Key keycode, bool &ok)
{
	ok = true;
	if(keycode <= 0xFFFF) {//Try to obtain the key from it's "character"
		const SHORT vKey = VkKeyScanW(static_cast<WCHAR>(keycode));
		if(vKey > -1)
			return LOBYTE(vKey);
	}

	//find key from switch/case --> Only finds a very small subset of keys
	switch (keycode)
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
	case Qt::Key_CapsLock:
		return VK_CAPITAL;
	case Qt::Key_NumLock:
		return VK_NUMLOCK;
	case Qt::Key_ScrollLock:
		return VK_SCROLL;

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

	case Qt::Key_Menu:
		return VK_APPS;
	case Qt::Key_Help:
		return VK_HELP;
	case Qt::Key_MediaNext:
		return VK_MEDIA_NEXT_TRACK;
	case Qt::Key_MediaPrevious:
		return VK_MEDIA_PREV_TRACK;
	case Qt::Key_MediaPlay:
		return VK_MEDIA_PLAY_PAUSE;
	case Qt::Key_MediaStop:
		return VK_MEDIA_STOP;
	case Qt::Key_VolumeDown:
		return VK_VOLUME_DOWN;
	case Qt::Key_VolumeUp:
		return VK_VOLUME_UP;
	case Qt::Key_VolumeMute:
		return VK_VOLUME_MUTE;
	case Qt::Key_Mode_switch:
		return VK_MODECHANGE;
	case Qt::Key_Select:
		return VK_SELECT;
	case Qt::Key_Printer:
		return VK_PRINT;
	case Qt::Key_Execute:
		return VK_EXECUTE;
	case Qt::Key_Sleep:
		return VK_SLEEP;
	case Qt::Key_Period:
		return VK_DECIMAL;
	case Qt::Key_Play:
		return VK_PLAY;
	case Qt::Key_Cancel:
		return VK_CANCEL;

	case Qt::Key_Forward:
		return VK_BROWSER_FORWARD;
	case Qt::Key_Refresh:
		return VK_BROWSER_REFRESH;
	case Qt::Key_Stop:
		return VK_BROWSER_STOP;
	case Qt::Key_Search:
		return VK_BROWSER_SEARCH;
	case Qt::Key_Favorites:
		return VK_BROWSER_FAVORITES;
	case Qt::Key_HomePage:
		return VK_BROWSER_HOME;

	case Qt::Key_LaunchMail:
		return VK_LAUNCH_MAIL;
	case Qt::Key_LaunchMedia:
		return VK_LAUNCH_MEDIA_SELECT;
	case Qt::Key_Launch0:
		return VK_LAUNCH_APP1;
	case Qt::Key_Launch1:
		return VK_LAUNCH_APP2;

	case Qt::Key_Massyo:
		return VK_OEM_FJ_MASSHOU;
	case Qt::Key_Touroku:
		return VK_OEM_FJ_TOUROKU;

	default:
		if(keycode <= 0xFFFF)
			return static_cast<BYTE>(keycode);
		else {
			ok = false;
			return 0;
		}
	}
}

quint32 QHotkeyPrivateWin::nativeModifiers(Qt::KeyboardModifiers modifiers, bool &ok)
{
	quint32 nMods = 0;
	if (modifiers & Qt::ShiftModifier)
		nMods |= MOD_SHIFT;
	if (modifiers & Qt::ControlModifier)
		nMods |= MOD_CONTROL;
	if (modifiers & Qt::AltModifier)
		nMods |= MOD_ALT;
	if (modifiers & Qt::MetaModifier)
		nMods |= MOD_WIN;
	ok = true;
	return nMods;
}

bool QHotkeyPrivateWin::registerShortcut(QHotkey::NativeShortcut shortcut)
{
	BOOL ok = RegisterHotKey(NULL,
							 HKEY_ID(shortcut),
							 shortcut.modifier + MOD_NOREPEAT,
							 shortcut.key);
	if(ok)
		return true;
	else {
		error = QHotkeyPrivateWin::formatWinError(::GetLastError());
		return false;
	}
}

bool QHotkeyPrivateWin::unregisterShortcut(QHotkey::NativeShortcut shortcut)
{
	BOOL ok = UnregisterHotKey(NULL, HKEY_ID(shortcut));
	if(ok)
		return true;
	else {
		error = QHotkeyPrivateWin::formatWinError(::GetLastError());
		return false;
	}
}

QString QHotkeyPrivateWin::formatWinError(DWORD winError)
{
	wchar_t *buffer = NULL;
	DWORD num = FormatMessageW(FORMAT_MESSAGE_ALLOCATE_BUFFER | FORMAT_MESSAGE_FROM_SYSTEM,
							   NULL,
							   winError,
							   0,
							   (LPWSTR)&buffer,
							   0,
							   NULL);
	if(buffer) {
		QString res = QString::fromWCharArray(buffer, num);
		LocalFree(buffer);
		return res;
	} else
		return QString();
}
```

### `./qhotkey_x11.cpp`

```cpp
#include "qhotkey.h"
#include "qhotkey_p.h"

#if QT_VERSION >= QT_VERSION_CHECK(6, 2, 0)
	#include <QGuiApplication>
#else
	#include <QDebug>
	#include <QX11Info>
#endif

#include <QThreadStorage>
#include <QTimer>
#include <X11/Xlib.h>
#include <xcb/xcb.h>

//compatibility to pre Qt 5.8
#ifndef Q_FALLTHROUGH
#define Q_FALLTHROUGH() (void)0
#endif

class QHotkeyPrivateX11 : public QHotkeyPrivate
{
public:
	// QAbstractNativeEventFilter interface
	bool nativeEventFilter(const QByteArray &eventType, void *message, _NATIVE_EVENT_RESULT *result) override;

protected:
	// QHotkeyPrivate interface
	quint32 nativeKeycode(Qt::Key keycode, bool &ok) Q_DECL_OVERRIDE;
	quint32 nativeModifiers(Qt::KeyboardModifiers modifiers, bool &ok) Q_DECL_OVERRIDE;
	static QString getX11String(Qt::Key keycode);
	bool registerShortcut(QHotkey::NativeShortcut shortcut) Q_DECL_OVERRIDE;
	bool unregisterShortcut(QHotkey::NativeShortcut shortcut) Q_DECL_OVERRIDE;

private:
	static const QVector<quint32> specialModifiers;
	static const quint32 validModsMask;
	xcb_key_press_event_t prevHandledEvent;
	xcb_key_press_event_t prevEvent;

	static QString formatX11Error(Display *display, int errorCode);

	class HotkeyErrorHandler {
	public:
		HotkeyErrorHandler();
		~HotkeyErrorHandler();

		static bool hasError;
		static QString errorString;

	private:
		XErrorHandler prevHandler;

		static int handleError(Display *display, XErrorEvent *error);
	};
};
NATIVE_INSTANCE(QHotkeyPrivateX11)

bool QHotkeyPrivate::isPlatformSupported()
{
#if QT_VERSION >= QT_VERSION_CHECK(6, 2, 0)
	return qGuiApp->nativeInterface<QNativeInterface::QX11Application>();
#else
	return QX11Info::isPlatformX11();
#endif
}

const QVector<quint32> QHotkeyPrivateX11::specialModifiers = {0, Mod2Mask, LockMask, (Mod2Mask | LockMask)};
const quint32 QHotkeyPrivateX11::validModsMask = ShiftMask | ControlMask | Mod1Mask | Mod4Mask;

bool QHotkeyPrivateX11::nativeEventFilter(const QByteArray &eventType, void *message, _NATIVE_EVENT_RESULT *result)
{
	Q_UNUSED(eventType)
	Q_UNUSED(result)

	auto *genericEvent = static_cast<xcb_generic_event_t *>(message);
	if (genericEvent->response_type == XCB_KEY_PRESS) {
		xcb_key_press_event_t keyEvent = *static_cast<xcb_key_press_event_t *>(message);
		this->prevEvent = keyEvent;
		if (this->prevHandledEvent.response_type == XCB_KEY_RELEASE) {
			if(this->prevHandledEvent.time == keyEvent.time) return false;
		}
		this->activateShortcut({keyEvent.detail, keyEvent.state & QHotkeyPrivateX11::validModsMask});
	} else if (genericEvent->response_type == XCB_KEY_RELEASE) {
		xcb_key_release_event_t keyEvent = *static_cast<xcb_key_release_event_t *>(message);
		this->prevEvent = keyEvent;
		QTimer::singleShot(50, [this, keyEvent] {
			if(this->prevEvent.time == keyEvent.time && this->prevEvent.response_type == keyEvent.response_type && this->prevEvent.detail == keyEvent.detail){
				this->releaseShortcut({keyEvent.detail, keyEvent.state & QHotkeyPrivateX11::validModsMask});
			}
		});
		this->prevHandledEvent = keyEvent;
	}

	return false;
}

QString QHotkeyPrivateX11::getX11String(Qt::Key keycode)
{
	switch(keycode){

		case Qt::Key_MediaLast :
		case Qt::Key_MediaPrevious :
			return QStringLiteral("XF86AudioPrev");
		case Qt::Key_MediaNext :
			return QStringLiteral("XF86AudioNext");
		case Qt::Key_MediaPause :
		case Qt::Key_MediaPlay :
		case Qt::Key_MediaTogglePlayPause :
			return QStringLiteral("XF86AudioPlay");
		case Qt::Key_MediaRecord :
			return QStringLiteral("XF86AudioRecord");
		case Qt::Key_MediaStop :
			return QStringLiteral("XF86AudioStop");
		default :
			return QKeySequence(keycode).toString(QKeySequence::NativeText);
	}
}

quint32 QHotkeyPrivateX11::nativeKeycode(Qt::Key keycode, bool &ok)
{
	QString keyString = getX11String(keycode);

	KeySym keysym = XStringToKeysym(keyString.toLatin1().constData());
	if (keysym == NoSymbol) {
		//not found -> just use the key
		if(keycode <= 0xFFFF)
			keysym = keycode;
		else
			return 0;
	}

#if QT_VERSION >= QT_VERSION_CHECK(6, 2, 0)
	const QNativeInterface::QX11Application *x11Interface = qGuiApp->nativeInterface<QNativeInterface::QX11Application>();
#else
	const bool x11Interface = QX11Info::isPlatformX11();
#endif

	if(x11Interface) {
#if QT_VERSION >= QT_VERSION_CHECK(6, 2, 0)
		Display *display = x11Interface->display();
#else
		Display *display = QX11Info::display();
#endif
		auto res = XKeysymToKeycode(display, keysym);
		if(res != 0)
			ok = true;
		return res;
	}
	return 0;
}

quint32 QHotkeyPrivateX11::nativeModifiers(Qt::KeyboardModifiers modifiers, bool &ok)
{
	quint32 nMods = 0;
	if (modifiers & Qt::ShiftModifier)
		nMods |= ShiftMask;
	if (modifiers & Qt::ControlModifier)
		nMods |= ControlMask;
	if (modifiers & Qt::AltModifier)
		nMods |= Mod1Mask;
	if (modifiers & Qt::MetaModifier)
		nMods |= Mod4Mask;
	ok = true;
	return nMods;
}

bool QHotkeyPrivateX11::registerShortcut(QHotkey::NativeShortcut shortcut)
{
#if QT_VERSION >= QT_VERSION_CHECK(6, 2, 0)
	const QNativeInterface::QX11Application *x11Interface = qGuiApp->nativeInterface<QNativeInterface::QX11Application>();
	Display *display = x11Interface->display();
#else
	const bool x11Interface = QX11Info::isPlatformX11();
	Display *display = QX11Info::display();
#endif

	if(!display || !x11Interface)
		return false;

	HotkeyErrorHandler errorHandler;
	for(quint32 specialMod : QHotkeyPrivateX11::specialModifiers) {
		XGrabKey(display,
				 shortcut.key,
				 shortcut.modifier | specialMod,
				 DefaultRootWindow(display),
				 True,
				 GrabModeAsync,
				 GrabModeAsync);
	}
	XSync(display, False);

	if(errorHandler.hasError) {
		error = errorHandler.errorString;
		this->unregisterShortcut(shortcut);
		return false;
	}
	return true;
}

bool QHotkeyPrivateX11::unregisterShortcut(QHotkey::NativeShortcut shortcut)
{
#if QT_VERSION >= QT_VERSION_CHECK(6, 2, 0)
	Display *display = qGuiApp->nativeInterface<QNativeInterface::QX11Application>()->display();
#else
	Display *display = QX11Info::display();
#endif

	if(!display)
		return false;

	HotkeyErrorHandler errorHandler;
	for(quint32 specialMod : QHotkeyPrivateX11::specialModifiers) {
		XUngrabKey(display,
				   shortcut.key,
				   shortcut.modifier | specialMod,
				   XDefaultRootWindow(display));
	}
	XSync(display, False);

	if(HotkeyErrorHandler::hasError) {
		error = HotkeyErrorHandler::errorString;
		return false;
	}
	return true;
}

QString QHotkeyPrivateX11::formatX11Error(Display *display, int errorCode)
{
	char errStr[256];
	XGetErrorText(display, errorCode, errStr, 256);
	return QString::fromLatin1(errStr);
}



// ---------- QHotkeyPrivateX11::HotkeyErrorHandler implementation ----------

bool QHotkeyPrivateX11::HotkeyErrorHandler::hasError = false;
QString QHotkeyPrivateX11::HotkeyErrorHandler::errorString;

QHotkeyPrivateX11::HotkeyErrorHandler::HotkeyErrorHandler()
{
	prevHandler = XSetErrorHandler(&HotkeyErrorHandler::handleError);
}

QHotkeyPrivateX11::HotkeyErrorHandler::~HotkeyErrorHandler()
{
	XSetErrorHandler(prevHandler);
	hasError = false;
	errorString.clear();
}

int QHotkeyPrivateX11::HotkeyErrorHandler::handleError(Display *display, XErrorEvent *error)
{
	switch (error->error_code) {
	case BadAccess:
	case BadValue:
	case BadWindow:
		if (error->request_code == 33 || //grab key
			error->request_code == 34) {// ungrab key
			hasError = true;
			errorString = QHotkeyPrivateX11::formatX11Error(display, error->error_code);
			return 1;
		}
		Q_FALLTHROUGH();
		// fall through
	default:
		return 0;
	}
}
```

### `./3rd_qhotkey.pri`

```makefile
CONFIG += C++11
HEADERS += $$PWD/qhotkey.h
HEADERS += $$PWD/qhotkey_p.h
SOURCES += $$PWD/qhotkey.cpp

win32 {
LIBS += -luser32
SOURCES += $$PWD/qhotkey_win.cpp
}

unix:!macx {
QT += x11extras
LIBS += -lX11
SOURCES += $$PWD/qhotkey_x11.cpp
}

macx {
LIBS += -framework Carbon
SOURCES += $$PWD/qhotkey_mac.cpp
}
```
