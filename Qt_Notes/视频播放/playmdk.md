---
tags:
  - Qt
  - 视频播放
---
# MDK 视频播放 (`playmdk`)

> 视频播放mdk

## 效果图

![[QWidgetDemo_assert/playmdk.jpg]]

## 分类

- 类型: 视频播放 `video`
- 源码目录: `video/playmdk`

## 技术要点

**用到的 Qt 类**: QTextCodec QWidget QString QApplication QPushButton QOpenGLWidget QGLWidget QFont QStringList QFileDialog QVBoxLayout QHBoxLayout QComboBox QTime QMetaObject

**特性/模块**: Q_OBJECT

## 完整源码

### `./main.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")
#include "widget.h"
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

    Widget w;
    w.setWindowTitle("视频流播放mdk内核 (QQ: 517216493 WX: feiyangqingyun)");
    w.show();

    return a.exec();
}
```

### `./widget.cpp`

```cpp
﻿#pragma execution_character_set("utf-8")
#include "widget.h"
#include "ui_widget.h"
#include "qlineedit.h"
#include "qfiledialog.h"

Widget::Widget(QWidget *parent) : QWidget(parent), ui(new Ui::Widget)
{
    ui->setupUi(this);

    QStringList urls;
    urls << "http://vd3.bdstatic.com/mda-jennyc5ci1ugrxzi/mda-jennyc5ci1ugrxzi.mp4";
    urls << "rtsp://admin:Admin123456@192.168.0.15:554/media/video1";
    ui->cboxUrl->addItems(urls);
    ui->cboxUrl->setCurrentIndex(0);
}

Widget::~Widget()
{
    delete ui;
}

void Widget::on_btnSelect_clicked()
{
    QString fileName = QFileDialog::getOpenFileName();
    if (!fileName.isEmpty()) {
        ui->cboxUrl->addItem(fileName);
        ui->cboxUrl->lineEdit()->setText(fileName);
        if (ui->btnOpen->text() == "打开") {
            on_btnOpen_clicked();
        }
    }
}

void Widget::on_btnOpen_clicked()
{
    if (ui->btnOpen->text() == "打开") {
        ui->btnOpen->setText("关闭");
        QString url = ui->cboxUrl->currentText().trimmed();
        ui->playWidget->setUrl(url);
        ui->playWidget->play();
    } else {
        ui->btnOpen->setText("打开");
        ui->playWidget->stop();
    }
}
```

### `./widget.h`

```cpp
﻿#ifndef WIDGET_H
#define WIDGET_H

#include <QWidget>

QT_BEGIN_NAMESPACE
namespace Ui { class Widget; }
QT_END_NAMESPACE

class Widget : public QWidget
{
    Q_OBJECT

public:
    Widget(QWidget *parent = 0);
    ~Widget();

private slots:
    void on_btnSelect_clicked();
    void on_btnOpen_clicked();

private:
    Ui::Widget *ui;
};
#endif // WIDGET_H
```

### `mdk/include/MediaInfo.h`

```cpp
/*
 * Copyright (c) 2019-2023 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include "global.h"
#include "c/MediaInfo.h"
#include <cstring>
#include <unordered_map>
#include <vector>

MDK_NS_BEGIN

struct AudioCodecParameters {
    const char* codec;
    uint32_t codec_tag;
    const uint8_t* extra_data; /* without padding data */
    int extra_data_size;
    int64_t bit_rate;
    int profile;
    int level;
    float frame_rate;

    bool is_float;
    bool is_unsigned;
    bool is_planar;
    int raw_sample_size;

    int channels;
    int sample_rate;
    int block_align;
    int frame_size; /* const samples per channel in a frame */
};

struct AudioStreamInfo {
    int index;
    int64_t start_time; /* ms */
    int64_t duration; /* ms */
    int64_t frames;

// stream language is metadata["language"]
    std::unordered_map<std::string, std::string> metadata;
    AudioCodecParameters codec;
};

struct VideoCodecParameters {
    const char* codec;
    uint32_t codec_tag;
    const uint8_t* extra_data; /* without padding data */
    int extra_data_size;
    int64_t bit_rate;
    int profile;
    int level;
    float frame_rate;
    int format; /* pixel format */
    const char* format_name; /* pixel format name */

    int width;
    int height;
    int b_frames;
    float par;
};

struct VideoStreamInfo {
    int index;
    int64_t start_time;
    int64_t duration;
    int64_t frames;
    int rotation; // degree need to rotate clockwise

// stream language is metadata["language"]
    std::unordered_map<std::string, std::string> metadata;
    VideoCodecParameters codec;

    const uint8_t* image_data = nullptr; // audio cover art image data, can be jpeg, png etc.
    int image_size = 0;
};

struct SubtitleCodecParameters {
    const char* codec;
    uint32_t codec_tag = 0;
    const uint8_t* extra_data; /* without padding data */
    int extra_data_size;

    int width = 0;  /* display width. bitmap subtitles only */
    int height = 0; /* display height. bitmap subtitles only */
};

struct SubtitleStreamInfo {
    int index;
    int64_t start_time;
    int64_t duration;

// stream language is metadata["language"]
    std::unordered_map<std::string, std::string> metadata;
    SubtitleCodecParameters codec;
};

struct ChapterInfo {
    int64_t start_time = 0;
    int64_t end_time = 0;
    std::string title;
};

struct ProgramInfo {
    int id;
    std::vector<int> stream; // stream index
    std::unordered_map<std::string, std::string> metadata; // "service_name", "service_provider" etc.
};

struct MediaInfo
{
    int64_t start_time; // ms
    int64_t duration;
    int64_t bit_rate;
    int64_t size; // file size. IGNORE ME!
    const char* format;
    int streams;

    std::vector<ChapterInfo> chapters;
    std::unordered_map<std::string, std::string> metadata;
    std::vector<AudioStreamInfo> audio;
    std::vector<VideoStreamInfo> video;
    std::vector<SubtitleStreamInfo> subtitle;
    std::vector<ProgramInfo> program;
};

// the following functions MUST be built into user's code because user's c++ stl abi is unknown
// used by Player.mediaInfo()
static void from_c(const mdkMediaInfo* cinfo, MediaInfo* info)
{
    *info = MediaInfo();
    if (!cinfo)
        return;
    info->start_time = cinfo->start_time;
    info->duration = cinfo->duration;
    info->bit_rate = cinfo->bit_rate;
    info->size = cinfo->size;
    info->format = cinfo->format;
    info->streams = cinfo->streams;

    mdkStringMapEntry entry{};
    while (MDK_MediaMetadata(cinfo, &entry))
        info->metadata[entry.key] = entry.value;
    for (int i = 0; i < cinfo->nb_chapters; ++i) {
        const auto& cci = cinfo->chapters[i];
        ChapterInfo ci;
        ci.start_time = cci.start_time;
        ci.end_time = cci.end_time;
        if (cci.title)
            ci.title = cci.title;
        info->chapters.emplace_back(std::move(ci));
    }
    for (int i = 0; i < cinfo->nb_audio; ++i) {
        AudioStreamInfo si{};
        const auto& csi = cinfo->audio[i];
        si.index = csi.index;
        si.start_time = csi.start_time;
        si.duration = csi.duration;
        si.frames = csi.frames;
        MDK_AudioStreamCodecParameters(&csi, (mdkAudioCodecParameters*)&si.codec);
        mdkStringMapEntry e{};
        while (MDK_AudioStreamMetadata(&csi, &e))
            si.metadata[e.key] = e.value;
        info->audio.push_back(std::move(si));
    }
    for (int i = 0; i < cinfo->nb_video; ++i) {
        VideoStreamInfo si{};
        const auto& csi = cinfo->video[i];
        si.index = csi.index;
        si.start_time = csi.start_time;
        si.duration = csi.duration;
        si.frames = csi.frames;
        si.rotation = csi.rotation;
        MDK_VideoStreamCodecParameters(&csi, (mdkVideoCodecParameters*)&si.codec);
        mdkStringMapEntry e{};
        while (MDK_VideoStreamMetadata(&csi, &e))
            si.metadata[e.key] = e.value;
        si.image_data = MDK_VideoStreamData(&csi, &si.image_size, 0);
        info->video.push_back(std::move(si));
    }
    for (int i = 0; i < cinfo->nb_subtitle; ++i) {
        SubtitleStreamInfo si{};
        const auto& csi = cinfo->subtitle[i];
        si.index = csi.index;
        si.start_time = csi.start_time;
        si.duration = csi.duration;
        MDK_SubtitleStreamCodecParameters(&csi, (mdkSubtitleCodecParameters*)&si.codec);
        mdkStringMapEntry e{};
        while (MDK_SubtitleStreamMetadata(&csi, &e))
            si.metadata[e.key] = e.value;
        info->subtitle.push_back(std::move(si));
    }
    for (int i = 0; i < cinfo->nb_programs; ++i) {
        const auto& cpi = cinfo->programs[i];
        ProgramInfo pi;
        pi.id = cpi.id;
        pi.stream.assign(cpi.stream, cpi.stream + cpi.nb_stream);
        mdkStringMapEntry e{};
        while (MDK_ProgramMetadata(&cpi, &e))
            pi.metadata[e.key] = e.value;
        info->program.push_back(std::move(pi));
    }
}

MDK_NS_END
```

### `mdk/include/Player.h`

```cpp
/*
 * Copyright (c) 2019-2023 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include "global.h"
#include "MediaInfo.h"
#include "RenderAPI.h"
#include "c/Player.h"
#include "VideoFrame.h"
#include <cinttypes>
#include <cstdlib>
#include <map>
#include <set>
#include <vector>

MDK_NS_BEGIN

/*!
  \brief PrepareCallback
  \param position in callback is the timestamp of the 1st frame(video if exists) after seek, or <0 (TODO: error code as position) if prepare() failed.
  \param boost in callback can be set by user(*boost = true/false) to boost the first frame rendering. default is true.
  \return false to unload media immediately when media is loaded and MediaInfo is ready, true to continue.
    example: always return false can be used as media information reader
 */
using PrepareCallback = std::function<bool(int64_t position, bool* boost)>;

/*!
 * \brief The Player class
 * High level API with basic playback function.
 */
class AudioFrame;
class VideoFrame;
class Player
{
public:
/*!
  \brief foreignGLContextDestroyed()
  Release GL resources bound to the context.
  - MUST be called when a foreign OpenGL context previously used is being destroyed and player object is already destroyed. The context MUST be current.
  - If player object is still alive, setVideoSurfaceSize(-1, -1, ...) is preferred.
  - If forget to call both foreignGLContextDestroyed() and setVideoSurfaceSize(-1, -1, ...) in the context, resources will be released in the next draw in the same context.
     But the context may be destroyed later, then resource will never be released
*/
    static void foreignGLContextDestroyed() {
        MDK_foreignGLContextDestroyed();
    }

    Player(const Player&) = delete;
    Player& operator=(const Player&) = delete;
    Player(const mdkPlayerAPI* cp = nullptr)
        : p(cp)
        , owner_(!cp) {
        if (!p)
            p = mdkPlayerAPI_new();
    }
    virtual ~Player() {
        if (owner_)
            mdkPlayerAPI_delete(&p);
    }

    void setMute(bool value = true) {
        MDK_CALL(p, setMute, value);
        mute_ = value;
    }

    bool isMute() const { return mute_; }
/*!
  \brief setVolume
  Set audio volume level
  \param value linear volume level, range is >=0. 1.0 is source volume
  \param channel channel number, int value of AudioFormat::Channel, -1 for all channels.
  The same as ms log2(SpeakerPosition), see https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/ksmedia/ns-ksmedia-ksaudio_channel_config#remarks
  setVolume(value, -1) equals to setVolume(value)
 */
    void setVolume(float value) {
        MDK_CALL(p, setVolume, value);
        volume_ = value;
    }
    void setVolume(float value, int channel) {
        MDK_CALL(p, setChannelVolume, value, channel);
    }

    float volume() const { return volume_; }
/*!
  \brief setFrameRate
  Set frame rate, frames per seconds
  \param value frame rate
  - 0 (default): use frame timestamp, or default frame rate 25.0fps if stream has no timestamp
  - <0: render ASAP.
  - >0: target frame rate
 */
    void setFrameRate(float value) {
        MDK_CALL(p, setFrameRate, value);
    }
/*!
  \brief setMedia
  Set a new media url.  If url changed, will stop current playback, and reset active tracks, external tracks set by setMedia(url, type)
  // MUST call setActiveTracks() after setMedia(), otherwise the 1st track in the media is used
 */
    void setMedia(const char* url) {
        MDK_CALL(p, setMedia, url);
    }
    // Set individual source for type, e.g. audio track file. If url is not empty, an individual pipeline will be used for 'type' tracks.
    // If url is empty, use 'type' tracks in MediaType::Video url.
    // MUST be after main media setMedia(url).
    // TODO: default type is Unknown
    void setMedia(const char* url, MediaType type) {
        MDK_CALL(p, setMediaForType, url, (MDK_MediaType)type);
    }

    const char* url() const {
        return MDK_CALL(p, url);
    }

    void setPreloadImmediately(bool value = true) {
        MDK_CALL(p, setPreloadImmediately, value);
    }
/*!
  \brief setNextMedia
  Gapless play the next media after current media playback end
  \param flags seek flags if startPosition > 0, accurate or fast
  set(State::Stopped) only stops current media. Call setNextMedia(nullptr, -1) first to disable next media.
  Usually you can call `currentMediaChanged()` to set a callback which invokes `setNextMedia()`, then call `setMedia()`.
*/
    void setNextMedia(const char* url, int64_t startPosition = 0, SeekFlag flags = SeekFlag::FromStart) {
        MDK_CALL(p, setNextMedia, url, startPosition, MDKSeekFlag(flags));
    }
/*!
  \brief currentMediaChanged
  Set a callback which is invoked when current media is stopped and a new media is about to play, or when setMedia() is called.
  Call before setMedia() to take effect.
 */
    void currentMediaChanged(std::function<void()> cb) { // call before setMedia()
        current_cb_ = cb;
        mdkCurrentMediaChangedCallback callback;
        callback.cb = [](void* opaque){
            auto f = (std::function<void()>*)opaque;
            (*f)();
        };
        callback.opaque = current_cb_ ? (void*)&current_cb_ : nullptr;
        MDK_CALL(p, currentMediaChanged, callback);
    }
/*!
  \brief setActiveTracks
  \param type if type is MediaType::Unknown, select a program(usually for mpeg ts streams). must contains only 1 value, N, indicates using the Nth program's audio and video tracks.
  Otherwise, select a set of tracks of given type.
  \param tracks set of active track number, from 0~N. Invalid track numbers will be ignored
 */
    void setActiveTracks(MediaType type, const std::set<int>& tracks) {
        std::vector<int> ts(tracks.cbegin(), tracks.cend());
        MDK_CALL(p, setActiveTracks, MDK_MediaType(type), ts.data(), ts.size());
    }
    // backends can be: AudioQueue(Apple only), OpenSL(Android only), ALSA(linux only), XAudio2(Windows only), OpenAL
    void setAudioBackends(const std::vector<std::string>& names) {
        std::vector<const char*> s(names.size() + 1, nullptr);
        for (size_t i = 0; i < names.size(); ++i)
            s[i] = names[i].data();
        MDK_CALL(p, setAudioBackends, s.data());
    }

// see https://github.com/wang-bin/mdk-sdk/wiki/Player-APIs#void-setdecodersmediatype-type-const-stdvectorstdstring-names
    void setDecoders(MediaType type, const std::vector<std::string>& names) {
        std::vector<const char*> s(names.size() + 1, nullptr);
        for (size_t i = 0; i < names.size(); ++i)
            s[i] = names[i].data();
        MDK_CALL(p, setDecoders, MDK_MediaType(type), s.data());
    }

/*!
  \brief setTimeout
  callback ms: elapsed milliseconds
  callback return: true to abort current operation on timeout.
  A null callback can abort current operation.
  Negative timeout infinit.
  Default timeout is 10s
 */
    void setTimeout(int64_t ms, std::function<bool(int64_t ms)> cb = nullptr) {
        timeout_cb_ = cb;
        mdkTimeoutCallback callback;
        callback.cb = [](int64_t ms, void* opaque){
            auto f = (std::function<bool(int64_t ms)>*)opaque;
            return (*f)(ms);
        };
        callback.opaque = timeout_cb_ ? (void*)&timeout_cb_ : nullptr;
        MDK_CALL(p, setTimeout, ms, callback);
    }

/*!
  \brief prepare
  Preload a media and then becomes State::Paused. \sa PrepareCallback
  To play a media from a given position, call prepare(ms) then set(State::Playing)
  \param startPosition start from position, relative to media start position(i.e. MediaInfo.start_time)
  \param cb if startPosition > 0, same as callback of seek(), called after the first frame is decoded or load/seek/decode error. If startPosition == 0, called when media is loaded and mediaInfo is ready or load error.
  \param flags seek flag if startPosition != 0.
  For fast seek(has flag SeekFlag::Fast), the first frame is a key frame whose timestamp >= startPosition
  For accurate seek(no flag SeekFlag::Fast), the first frame is the nearest frame whose timestamp <= startPosition, but the position passed to callback is the key frame position <= startPosition
 */
    void prepare(int64_t startPosition = 0, PrepareCallback cb = nullptr, SeekFlag flags = SeekFlag::FromStart) {
        prepare_cb_ = cb;
        mdkPrepareCallback callback;
        callback.cb = [](int64_t position, bool* boost, void* opaque){
            auto f = (PrepareCallback*)opaque;
            return (*f)(position, boost);
        };
        callback.opaque = prepare_cb_ ? (void*)&prepare_cb_ : nullptr;
        MDK_CALL(p, prepare, startPosition, callback, MDKSeekFlag(flags));
    }
/*!
  \brief mediaInfo
  Current MediaInfo. You can call it in prepare() callback which is called when loaded or load failed.
  Some fields can change during playback, e.g. video frame size change(via MediaEvent), live stream duration change, realtime bitrate change.
  You may get an invalid value if mediaInfo() is called immediately after `set(State::Playing)` or `prepare()` because media is still opening but not loaded , i.e. mediaStatus() has no MediaStatus::Loaded flag.
*/
    const MediaInfo& mediaInfo() const {
        from_c(MDK_CALL(p, mediaInfo), &info_);
        return info_;
    }

/*!
  \brief set(State)
  Request a new state. It's async and may take effect later.
  set(State::Stopped) only stops current media. Call setNextMedia(nullptr, -1) before stop to disable next media.
  set(State::Stopped) will release all resouces and clear video renderer viewport. While a normal playback end will keep renderer resources
  and the last video frame. Manually call set(State::Stopped) to clear them.
  NOTE: the requested state is not queued. so set one state immediately after another may have no effect.
  e.g. State::Playing after State::Stopped may have no effect if playback have not been stopped and still in Playing state
  so the final state is State::Stopped. Current solution is waitFor(State::Stopped) before set(State::Playing).
  Usually no waitFor(State::Playing) because we want async load
*/
    Player& set(State value) {
        MDK_CALL(p, setState, MDK_State(value));
        return *this;
    }

    PlaybackState state() const {
        return (PlaybackState)MDK_CALL(p, state);
    }

    Player& onStateChanged(std::function<void(State)> cb) {
        state_cb_ = cb;
        mdkStateChangedCallback callback;
        callback.cb = [](MDK_State value, void* opaque){
            auto f = (std::function<void(PlaybackState)>*)opaque;
            (*f)(State(value));
        };
        callback.opaque = state_cb_ ? (void*)&state_cb_ : nullptr;
        MDK_CALL(p, onStateChanged, callback);
        return *this;
    }
/*!
  \brief waitFor
  If failed to open a media, e.g. invalid media, unsupported format, waitFor() will finish without state change
*/
    bool waitFor(State value, long timeout = -1) {
        return MDK_CALL(p, waitFor, (MDK_State)value, timeout);
    }

    MediaStatus mediaStatus() const {
        return (MediaStatus)MDK_CALL(p, mediaStatus);
    }
/*!
  \brief onMediaStatusChanged
  Add a callback to be invoked when MediaStatus is changed
  \param cb null to clear callbacks. return true
 */

#if (__cpp_attributes+0)
//[[deprecated("use 'onMediaStatus' instead")]]
#endif
    Player& onMediaStatusChanged(std::function<bool(MediaStatus)> cb) {
        if (!cb)
            return onMediaStatus(nullptr);
        return onMediaStatus([cb](MediaStatus, MediaStatus newValue){
            return cb(newValue);
        });
    }

    Player& onMediaStatus(std::function<bool(MediaStatus oldValue, MediaStatus newValue)> cb, CallbackToken* token = nullptr) {
        status_cb_ = cb;
        mdkMediaStatusCallback callback;
        callback.cb = [](MDK_MediaStatus oldValue, MDK_MediaStatus newValue, void* opaque){
            auto p = (Player*)opaque;
            return p->status_cb_(MediaStatus(oldValue), MediaStatus(newValue));
        };
        callback.opaque = status_cb_ ? this : nullptr;
        MDK_CALL(p, onMediaStatus, callback, token);
        return *this;
    }

    enum SurfaceType {
        Auto, // platform default type
        X11,
        GBM,
        Wayland,
    };
/*!
 * \brief updateNativeSurface
 * If surface is not created, create rendering context internally by createSurface() and attached to native surface
 * native surface MUST be not null before destroying player
 */
// type: ignored if win ptr does not change (request to resize)
    void updateNativeSurface(void* surface, int width = -1, int height = -1, SurfaceType type = SurfaceType::Auto) {
        MDK_CALL(p, updateNativeSurface, surface, width, height, (MDK_SurfaceType)type);
    }

    void createSurface(void* nativeHandle = nullptr, SurfaceType type = SurfaceType::Auto) {
        MDK_CALL(p, createSurface, nativeHandle, (MDK_SurfaceType)type);
    }

    void resizeSurface(int w, int h) {
        MDK_CALL(p, resizeSurface, w, h);
    }

    void showSurface() {
        MDK_CALL(p, showSurface);
    }

// vo_opaque: a ptr to identify the renderer. can be null, then it is the default vo/renderer.
    struct SnapshotRequest {
/* data: rgba or bgra data. Created internally or provided by user.
   If data is provided by user, stride,  height and width MUST be also set, and data MUST be valid until snapshot callback is finished.
 */
        uint8_t* data = nullptr;
        // result width of snapshot image set by user, or the same as current frame width if 0. no renderer transform.
        // if both requested width and height are < 0, then result image is scaled image of current frame with ratio=width/height. no renderer transform.
        // if only one of width and height < 0, then the result size is video renderer viewport size, and all transforms will be applied.
        // if both width and height == 0, then result size is region of interest size of video frame set by setPointMap(), or video frame size
        int width = 0;
        int height = 0;
        int stride = 0;
        bool subtitle = false; // not supported yet
    };
/* \brief SnapshotCallback
   snapshot callback.
   \param req result request. If null, snapshot failed. Otherwise req.width, height and stride are always >0, data is never null.
   \param frameTime captured frame timestamp(seconds)
   \param opaque user data
   \returns a file path to save as a file(jpeg is recommended, other formats depends on ffmpeg runtime). or empty to do nothing.
   Returned string will be freed internally(assume allocated by malloc family apis).
   FIXME: malloc in user code and free in mdk may crash if mixed debug and release(vcrt)
   Callback is called in a dedicated thread, so time-consuming operations(encode, file io etc.) are allowed in the callback.
 */
    using SnapshotCallback = std::function<std::string(SnapshotRequest*, double frameTime)>;
/*!
  \brief snapshot
  take a snapshot from current renderer. The result is in bgra format, or null on failure.
  When `snapshot()` is called, redraw is scheduled for `vo_opaque`'s renderer, then renderer will take a snapshot in rendering thread.
  So for a foreign context, if renderer's surface/window/widget is invisible or minimized, snapshot may do nothing because of system or gui toolkit painting optimization.
*/
    void snapshot(SnapshotRequest* request, SnapshotCallback cb, void* vo_opaque = nullptr) {
        snapshot_cb_ = cb;
        mdkSnapshotCallback callback;
        callback.cb = [](mdkSnapshotRequest* req, double frameTime, void* opaque){
            auto f = (SnapshotCallback*)opaque;
            auto file = (*f)((SnapshotRequest*)req, frameTime);
            if (file.empty())
                return (char*)nullptr;
            return MDK_strdup(file.data());
        };
        callback.opaque = snapshot_cb_ ? (void*)&snapshot_cb_ : nullptr;
        return MDK_CALL(p, snapshot, (mdkSnapshotRequest*)request, callback, vo_opaque);
    }

/*
  \brief setProperty
  Set additional properties. Can be used to store user data, or change player behavior if the property is defined internally.
  Predefined properties are:
  - "video.avfilter": ffmpeg avfilter filter graph string for video track. take effect immediately
  - "audio.avfilter": ffmpeg avfilter filter graph string for audio track. take effect immediately
  - "continue_at_end" or "keep_open": "0" or "1". do not stop playback when decode and render to end of stream. only set(State::Stopped) can stop playback. Useful for timeline preview.
  - "cc": "0" or "1"(default). enable closed caption decoding and rendering.
  - "subtitle": "0" or "1"(default). enable subtitle(including cc) rendering. setActiveTracks(MediaType::Subtitle, {...}) enables decoding only.
  - "avformat.some_name": avformat option, e.g. {"avformat.fpsprobesize": "0"}. if global option "demuxer.io=0", it also can be AVIOContext/URLProtocol option
  - "avio.some_name": AVIOContext/URLProtocol option, e.g. "avio.user_agent"
  - "avcodec.some_name": AVCodecContext option, will apply for all FFmpeg based video/audio/subtitle decoders. To set for a single decoder, use setDecoders() with options
  - "audio.decoder": audio decoder property, value is "key=value" or "key1=value1:key2=value2". override "decoder" property
  - "video.decoder": video decoder property, value is "key=value" or "key1=value1:key2=value2". override "decoder" property
  - "decoder": video and audio decoder property, value is "key=value" or "key1=value1:key2=value2"
 */
    void setProperty(const std::string& key, const std::string& value) {
        MDK_CALL(p, setProperty, key.data(), value.data());
    }

    std::string property(const std::string& key, const std::string& defaultValue = std::string()) const {
        auto value = MDK_CALL(p, getProperty, key.data());
        if (!value)
            return defaultValue;
        return value;
    }
// A vo/renderer (e.g. the default vo/renderer) is gfx context aware, i.e. can render in multiple gfx contexts with a single vo/renderer, but parameters(e.g. surface size)
// must be updated when switch to a new context. So per gfx context vo/renderer can be better because parameters are stored in vo/renderer.
/*!
  \brief getVideoFrame
  get current rendered frame, i.e. the decoded video frame rendered by renderVideo()
 */
    void getVideoFrame(VideoFrame* frame, void* vo_opaque = nullptr);
/*
  \brief setVideoSurfaceSize
  Window size, surface size or drawable size. Render callback(if exists) will be invoked if width and height > 0.
  Usually for foreign contexts, i.e. not use updateNativeSurface().
NOTE:
  If width or heigh < 0, corresponding video renderer (for vo_opaque) will be removed and gfx resources will be released(need the context to be current for GL).
  But subsequence call with this vo_opaque will create renderer again. So it can be used before destroying the renderer.
  OpenGL: resources must be released by setVideoSurfaceSize(-1, -1, ...) in a correct context. If player is destroyed before context, MUST call Player::foreignGLContextDestroyed() when destroying the context.
 */
    void setVideoSurfaceSize(int width, int height, void* vo_opaque = nullptr) {
        MDK_CALL(p, setVideoSurfaceSize, width, height, vo_opaque);
    }
/*!
  \brief setVideoViewport
  The rectangular viewport where the scene will be drawn relative to surface viewport.
  x, y, w, h are normalized to [0, 1]
*/
    void setVideoViewport(float x, float y, float w, float h, void* vo_opaque = nullptr) {
        MDK_CALL(p, setVideoViewport, x, y, w, h, vo_opaque);
    }
/*!
  \brief setAspectRatio
  Video display aspect ratio.
  IgnoreAspectRatio(0): ignore aspect ratio and scale to fit renderer viewport
  KeepAspectRatio(default): keep frame aspect ratio and scale as large as possible inside renderer viewport
  KeepAspectRatioCrop: keep frame aspect ratio and scale as small as possible outside renderer viewport
  other value > 0: like KeepAspectRatio, but keep given aspect ratio and scale as large as possible inside renderer viewport
  other value < 0: like KeepAspectRatioCrop, but keep given aspect ratio and scale as small as possible inside renderer viewport
 */
    void setAspectRatio(float value, void* vo_opaque = nullptr) {
        MDK_CALL(p, setAspectRatio, value, vo_opaque);
    }
/*!
  \brief rotate
  rotate around video frame center
  \param degree: 0, 90, 180, 270, counterclockwise
 */
    void rotate(int degree, void* vo_opaque = nullptr) {
        MDK_CALL(p, rotate, degree, vo_opaque);
    }
/*!
  \brief scale
  scale frame size. x, y can be < 0, means scale and flip.
*/
    void scale(float x, float y, void* vo_opaque = nullptr) {
        MDK_CALL(p, scale, x, y, vo_opaque);
    }

    enum MapDirection {
        FrameToViewport, // left-hand
        ViewportToFrame, // left-hand
    };
/*!
  \brief mapPoint
  map a point from one coordinates to another. a frame must be rendered. coordinates is normalized to [0, 1].
  \param x points to x coordinate of viewport or currently rendered video frame
  \param z not used
*/
    void mapPoint(MapDirection dir, float* x, float* y, float* z = nullptr, void* vo_opaque = nullptr) {
        MDK_CALL(p, mapPoint, MDK_MapDirection(dir), x, y, z, vo_opaque);
    }

/*!
  \brief setPointMap
  Can be called on any thread
  \param videoRoi: array of 2d point (x, y) in video frame. coordinate: top-left = (0, 0), bottom-right=(1, 1). set null to disable mapping
  \param viewRoi: array of 2d point (x, y) in video renderer. coordinate: top-left = (0, 0), bottom-right=(1, 1). null is the whole renderer.
  \param count: point count. only support 2. set 0 to disable mapping
 */
    void setPointMap(const float* videoRoi, const float* viewRoi = nullptr, int count = 2, void* vo_opaque = nullptr) {
        MDK_CALL(p, setPointMap, videoRoi, viewRoi, count, vo_opaque);
    }
/*
  RenderAPI
  RenderAPI provides platform/api dependent resources for video renderer and rendering context corresponding to vo_opaque. It's used by
  1. create internal render context via updateNativeSurface() using given api. MUST be called before any other functions have parameter vo_opaque and updateNativeSurface()!
    To use RenderAPI other than OpenGL, setRenderAPI() MUST be called before add/updateNativeSurface(), and vo_opaque MUST be the surface or nullptr.
    If vo_opaque is nullptr, i.e. the default, then all context will have the same RenderAPI type, and call setRenderAPI() once is enough.
    If vo_opaque is surface(not null), each surface can have it's own RenderAPI type.
    RenderAPI members will be initialized when a rendering context for surface is created, and keep valid in rendering functions like renderVideo()
  2. Set foreign context provided by user. setRenderAPI() and other functions with vo_opaque parameter can be called in any order
  3. render. renderVideo() will use the given api for vo_opaque

  If setRenderAPI() is not called by user, a default one (usually GLRenderAPI) is used, thus renderAPI() always not null.
  setRenderAPI() is not thread safe, so usually called before rendering starts, or native surface is set.
*/

/*!
  \brief setRenderAPI
  set render api for a vo, useful for non-opengl(no way to get current context)
  \param api
  To release gfx resources, set null api in rendering thread/context(required by vulkan)
 */
    Player& setRenderAPI(RenderAPI* api, void* vo_opaque = nullptr) {
        MDK_CALL(p, setRenderAPI, reinterpret_cast<mdkRenderAPI*>(api), vo_opaque);
        return *this;
    }
/*!
  \brief renderApi()
  get render api. For offscreen rendering, may only api type be valid in setRenderAPI(), and other members are filled internally, and used by user after renderVideo()
 */
    RenderAPI* renderAPI(void* vo_opaque = nullptr) const {
        return reinterpret_cast<RenderAPI*>(MDK_CALL(p, renderAPI, vo_opaque));
    }

/*!
   \brief renderVideo
  Render the next or current(redraw) frame. Foreign render context only (i.e. not created by createSurface()/updateNativeSurface()).
  OpenGL: Can be called in multiple foreign contexts for the same vo_opaque.
   \return timestamp of rendered frame, or < 0 if no frame is rendered. precision is microsecond
 */
    double renderVideo(void* vo_opaque = nullptr) {
        return MDK_CALL(p, renderVideo, vo_opaque);
    }
/*!
  \brief enqueue
  Send the frame to video renderer. You must call renderVideo() later in render thread
*/
    void enqueue(const VideoFrame& frame, void* vo_opaque = nullptr) {
        MDK_CALL2(p, enqueueVideo, frame.toC(), vo_opaque);
    }
/*!
  \brief setBackgroundColor
  r, g, b, a range is [0, 1]. default is 0. if out of range, background color will not be filled
 */
    void setBackgroundColor(float r, float g, float b, float a, void* vo_opaque = nullptr) {
        return MDK_CALL(p, setBackgroundColor, r, g, b, a, vo_opaque);
    }

    Player& set(VideoEffect effect, const float& values, void* vo_opaque = nullptr) {
        MDK_CALL(p, setVideoEffect, MDK_VideoEffect(effect), &values, vo_opaque);
        return *this;
    }
/*!
  \brief set
  Set output color space.
  \param value
    - invalid (ColorSpaceUnknown): renderer will try to use the value of decoded frame, and will send hdr10 metadata when possible. i.e. hdr videos will enable hdr display. Currently only supported by metal, and `MetalRenderAPI.layer` must be a `CAMetalLayer` ([example](https://github.com/wang-bin/mdkSwift/blob/master/Player.swift#L184))
    - hdr colorspace (ColorSpaceBT2100_PQ): no hdr metadata will be sent to the display, sdr will map to hdr. Can be used by the gui toolkits which support hdr swapchain but no api to change swapchain colorspace and format on the fly, see [Qt example](https://github.com/wang-bin/mdk-examples/blob/master/Qt/qmlrhi/VideoTextureNodePub.cpp#L83)
    - sdr color space (ColorSpaceBT709): the default. HDR videos will tone map to SDR.
*/
    Player& set(ColorSpace value, void* vo_opaque = nullptr) {
        MDK_CALL(p, setColorSpace, MDK_ColorSpace(value), vo_opaque);
        return *this;
    }

/*!
  \brief setRenderCallback
  set a callback which is invoked when the vo coresponding to vo_opaque needs to update/draw content, e.g. when a new frame is received in the renderer.
  Also invoked in setVideoSurfaceSize(), setVideoViewport(), setAspectRatio() and rotate(), take care of dead lock in callback and above functions.
  with vo_opaque, user can know which vo/renderer is rendering, useful for multiple renderers
  There may be no frames or playback not even started, but renderer update is required internally

  DO NOT call renderVideo() in the callback, otherwise will results in dead lock
*/
    void setRenderCallback(std::function<void(void* vo_opaque)> cb) { // per vo?
        render_cb_ = cb;
        mdkRenderCallback callback;
        callback.cb = [](void* vo_opaque, void* opaque){
            auto f = (std::function<void(void* vo_opaque)>*)opaque;
            (*f)(vo_opaque);
        };
        callback.opaque = render_cb_ ? (void*)&render_cb_ : nullptr;
        MDK_CALL(p, setRenderCallback, callback);
    }

/*!
  \brief onFrame
  A callback to be invoked before delivering a frame to renderers. Frame can be VideoFrame and AudioFrame(NOT IMPLEMENTED).
  The callback can be used as a filter.
  TODO: frames not in host memory
  \param cb callback to be invoked. returns pending number of frames. callback parameter is input and output frame. if input frame is an invalid frame, output a pending frame.
  For most filters, 1 input frame generates 1 output frame, then return 0.
 */
    template<class Frame>
    Player& onFrame(std::function<int(Frame&, int/*track*/)> cb);
/*!
  \brief position
  Current playback time in milliseconds. Relative to media's first timestamp, which usually is 0.
  If has active video tracks, it's currently presented video frame time. otherwise, it's audio time.
 */
    int64_t position() const {
        return MDK_CALL(p, position);
    }
/*!
  \brief seek
  \param pos seek target. if flags has SeekFlag::Frame, pos is frame count, otherwise it's milliseconds.
  If pos > media time range, e.g. INT64_MAX, will seek to the last frame of media for SeekFlag::AnyFrame, and the last key frame of media for SeekFlag::Fast.
  If pos > media time range with SeekFlag::AnyFrame, playback will stop unless setProperty("continue_at_end", "1") was called
  FIXME: a/v sync broken if SeekFlag::Frame|SeekFlag::FromNow.
  \param cb if succeeded, callback is called when stream seek finished and after the 1st frame decoded or decode error(e.g. video tracks disabled), ret(>=0) is the timestamp of the 1st frame(video if exists) after seek.
  If error(io, demux, not decode) occured(ret < 0, usually -1) or skipped because of unfinished previous seek(ret == -2), out of range(-4) or media unloaded(-3).
 */
    bool seek(int64_t pos, SeekFlag flags, std::function<void(int64_t ret)> cb = nullptr) {
        seek_cb_ = cb;
        mdkSeekCallback callback;
        callback.cb = [](int64_t ms, void* opaque){
            auto f = (std::function<void(int64_t)>*)opaque;
            (*f)(ms);
        };
        callback.opaque = seek_cb_ ? (void*)&seek_cb_ : nullptr;
        return MDK_CALL(p, seekWithFlags, pos, MDK_SeekFlag(flags), callback);
    }

    bool seek(int64_t pos, std::function<void(int64_t)> cb = nullptr) {
        return seek(pos, SeekFlag::Default, cb);
    }

    void setPlaybackRate(float value) {
        MDK_CALL(p, setPlaybackRate, value);
    }

    float playbackRate() const {
        return MDK_CALL(p, playbackRate);
    }
/*!
 * \brief buffered
 * get buffered undecoded data duration and size
 * \return buffered data(packets) duration
 */
    int64_t buffered(int64_t* bytes = nullptr) const {
        return MDK_CALL(p, buffered, bytes);
    }
/*!
  \brief bufferRange
  set duration range of buffered data.
  \param minMs default 1000. wait for buffered duration >= minMs when before popping a packet.
    If minMs < 0, then minMs, maxMs and drop will be reset to the default value.
    If minMs > 0, when packets queue becomes empty, `MediaStatus::Buffering` will be set until queue duration >= minMs, "reader.buffering" MediaEvent
    will be triggered.
    If minMs == 0, decode ASAP.
  \param maxMs default 4000. max buffered duration. Large value is recommended. Latency is not affected.
    If maxMs < 0, then maxMs and drop will be reset to the default value
    If maxMs == 0, same as INT64_MAX
  drop = true:
    drop old non-key frame packets to reduce buffered duration until < maxMs. If maxMs(!=0 or INT64_MAX) is smaller then key-frame duration, no drop effect.
    If maxMs == 0 or INT64_MAX, always drop old packets and keep at most 1 key-frame packet
  drop = false: wait for buffered duration < maxMs before pushing packets

  For realtime streams like(rtp, rtsp, rtmp sdp etc.), the default range is [0, INT64_MAX, true].
  Usually you don't need to call this api. This api can be used for low latency live videos, for example setBufferRange(0, INT64_MAX, true) will decode as soon as possible when media data received, and no accumulated delay.
 */
    void setBufferRange(int64_t minMs = -1, int64_t maxMs = -1, bool drop = false) {
        MDK_CALL(p, setBufferRange, minMs, maxMs, drop);
    }
/*!
  \brief switchBitrate
  A new media will be played later
  \param delay switch after at least delay ms. TODO: determined by buffered time, e.g. from high bit rate without enough buffered samples to low bit rate
  \param cb (true/false) called when finished/failed
  \param flags seek flags for the next url, accurate or fast
 */
    void switchBitrate(const char* url, int64_t delay = -1, std::function<void(bool)> cb = nullptr) {
        switch_cb_ = cb;
        SwitchBitrateCallback callback;
        callback.cb = [](bool value, void* opaque){
            auto f = (std::function<void(bool)>*)opaque;
            (*f)(value);
        };
        callback.opaque = switch_cb_ ? (void*)&switch_cb_ : nullptr;
        return MDK_CALL(p, switchBitrate, url, delay, callback);
    }
/*!
 * \brief switchBitrateSingalConnection
 * Only 1 media is loaded. The previous media is unloaded and the playback continues. When new media is preloaded, stop the previous media at some point
 * MUST call setPreloadImmediately(false) because PreloadImmediately for singal connection preload is not possible.
 * \return false if preload immediately
 * This will not affect next media set by user
 */
    bool switchBitrateSingleConnection(const char *url, std::function<void(bool)> cb = nullptr) {
        switch_cb_ = cb;
        SwitchBitrateCallback callback;
        callback.cb = [](bool value, void* opaque){
            auto f = (std::function<void(bool)>*)opaque;
            (*f)(value);
        };
        callback.opaque = switch_cb_ ? (void*)&switch_cb_ : nullptr;
        return MDK_CALL(p, switchBitrateSingleConnection, url, callback);
    }

/*!
  \brief onEvent
  Add/Remove a [MediaEvent](https://github.com/wang-bin/mdk-sdk/wiki/Types#class-mediaevent) listener, or remove listeners.
  callback return: true if event is processed and should stop dispatching.
 */
    Player& onEvent(std::function<bool(const MediaEvent&)> cb, CallbackToken* token = nullptr) {
        mdkMediaEventCallback callback{};
        if (!cb) {
            MDK_CALL(p, onEvent, callback, token ? &event_cb_key_[*token] : nullptr);
            if (token) {
                event_cb_.erase(*token);
                event_cb_key_.erase(*token);
            } else {
                event_cb_.clear();
                event_cb_key_.clear();
            }
        } else {
            static CallbackToken k = 1;
            event_cb_[k] = cb;
            callback.cb = [](const mdkMediaEvent* me, void* opaque){
                auto f = (std::function<bool(const MediaEvent&)>*)opaque;
                MediaEvent e;
                e.error = me->error;
                e.category = me->category;
                e.detail = me->detail;
                e.decoder.stream = me->decoder.stream;
                e.video.width = me->video.width;
                e.video.height = me->video.height;
                return (*f)(e);
            };
            callback.opaque = &event_cb_[k];
            CallbackToken t;
            MDK_CALL(p, onEvent, callback, &t);
            event_cb_key_[k] = t;
            if (token)
                *token = t;
            k++;
        }
        return *this;
    }
/*
  \brief record
  Start to record or stop recording current media by remuxing packets read. If media is not loaded, recorder will start when playback starts
  \param url destination. null or the same value as recording one to stop recording
  \param format forced format if unable to guess from url suffix
 */
    void record(const char* url = nullptr, const char* format = nullptr) {
        MDK_CALL(p, record, url, format);
    }

/*!
  \brief setLoop
  Set A-B loop repeat count.
  \param count repeat count. 0 to disable looping and stop when out of range(B)
 */
    void setLoop(int count) {
        MDK_CALL(p, setLoop, count);
    }
/*
  \brief onLoop
  add/remove a callback which will be invoked right before a new A-B loop
  \param cb callback with current loop count elapsed
 */
    Player& onLoop(std::function<void(int)> cb, CallbackToken* token = nullptr) {
        mdkLoopCallback callback{};
        if (!cb) {
            MDK_CALL(p, onLoop, callback, token ? &loop_cb_key_[*token] : nullptr);
            if (token) {
                loop_cb_.erase(*token);
                loop_cb_key_.erase(*token);
            } else {
                loop_cb_.clear();
                loop_cb_key_.clear();
            }
        } else {
            static CallbackToken k = 1;
            loop_cb_[k] = cb;
            callback.cb = [](int countNow, void* opaque){
                auto f = (std::function<void(int)>*)opaque;
                return (*f)(countNow);
            };
            callback.opaque = &loop_cb_[k];
            CallbackToken t;
            MDK_CALL(p, onLoop, callback, &t);
            loop_cb_key_[k] = t;
            if (token)
                *token = t;
            k++;
        }
        return *this;
    }
/*!
  \brief setRange
  Set A-B loop range, or playback range
  \param a loop position begin, in ms.
  \param b loop position end, in ms. -1, INT64_MAX or numeric_limit<int64_t>::max() indicates b is the end of media
 */
    void setRange(int64_t a, int64_t b = INT64_MAX) {
        MDK_CALL(p, setRange, a, b);
    }

/*!
  \brief onSync
  \param cb a callback invoked when about to render a frame. return expected current playback position(seconds), e.g. DBL_MAX(TimestampEOS) indicates render video frame ASAP.
  sync callback clock should handle pause, resume, seek and seek finish events
 */
    Player& onSync(std::function<double()> cb, int minInterval = 10) {
        sync_cb_ = cb;
        mdkSyncCallback callback;
        callback.cb = [](void* opaque){
            auto f = (std::function<double()>*)opaque;
            return (*f)();
        };
        callback.opaque = sync_cb_ ? (void*)&sync_cb_ : nullptr;
        MDK_CALL(p, onSync, callback, minInterval);
        return *this;
    }


#if !MDK_VERSION_CHECK(1, 0, 0)
#if (__cpp_attributes+0)
[[deprecated("use setDecoders(MediaType::Audio, names) instead")]]
#endif
    void setAudioDecoders(const std::vector<std::string>& names) {
        setDecoders(MediaType::Audio, names);
    }
#if (__cpp_attributes+0)
[[deprecated("use setDecoders(MediaType::Video, names) instead")]]
#endif
    void setVideoDecoders(const std::vector<std::string>& names) {
        setDecoders(MediaType::Video, names);
    }
#if (__cpp_attributes+0)
[[deprecated("use set(State) instead")]]
#endif
    void setState(PlaybackState value) {
        set(value);
    }
#endif
private:
    const mdkPlayerAPI* p = nullptr;
    bool owner_ = true;
    bool mute_ = false;
    float volume_ = 1.0f;
    std::function<void()> current_cb_ = nullptr;
    std::function<bool(int64_t ms)> timeout_cb_ = nullptr;
    std::function<bool(int64_t position, bool* boost)> prepare_cb_ = nullptr;
    std::function<void(State)> state_cb_ = nullptr;
    std::function<bool(MediaStatus, MediaStatus)> status_cb_ = nullptr;
    std::function<void(void* vo_opaque)> render_cb_ = nullptr;
    std::function<void(int64_t)> seek_cb_ = nullptr;
    std::function<void(bool)> switch_cb_ = nullptr;
    SnapshotCallback snapshot_cb_ = nullptr;
    std::function<int(VideoFrame&, int/*track*/)> video_cb_ = nullptr;
    std::function<double()> sync_cb_ = nullptr;
    std::map<CallbackToken, std::function<bool(const MediaEvent&)>> event_cb_; // rb tree, elements never destroyed
    std::map<CallbackToken,CallbackToken> event_cb_key_;
    std::map<CallbackToken, std::function<void(int)>> loop_cb_; // rb tree, elements never destroyed
    std::map<CallbackToken,CallbackToken> loop_cb_key_;

    mutable MediaInfo info_;
};


template<>
inline Player& Player::onFrame(std::function<int(VideoFrame&, int/*track*/)> cb)
{
    video_cb_ = cb;
    mdkVideoCallback callback;
    callback.cb = [](mdkVideoFrameAPI** pFrame/*in/out*/, int track, void* opaque){
        VideoFrame frame;
        frame.attach(*pFrame);
        auto f = (std::function<int(VideoFrame&, int)>*)opaque;
        auto pendings = (*f)(frame, track);
        *pFrame = frame.detach();
        return pendings;
    };
    callback.opaque = video_cb_ ? (void*)&video_cb_ : nullptr;
    MDK_CALL(p, onVideo, callback);
    return *this;
}

MDK_NS_END
```

### `mdk/include/RenderAPI.h`

```cpp
/*
 * Copyright (c) 2019-2023 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include "global.h"
#include <cstring>

MDK_NS_BEGIN

/*!
  \brief RenderAPI
  use concrete types in user code, for example D3D11RenderAPI
 */
struct RenderAPI {
    enum Type {
        Invalid,
        OpenGL = 1,
        Vulkan = 2,
        Metal = 3,
        D3D11 = 4,
        D3D12 = 5,
    };

    //Type type() const { return Type(type_ & 0xffff);}
protected:
    Type type_ = Type::Invalid; // high 16 bits: major + minor version, to unbreak abi for my flawed design

    Type versioned(Type t) const { return Type(t | (MDK_VERSION >> 8 << 16));}
};


struct GLRenderAPI final: RenderAPI {
    GLRenderAPI() {
        type_ = versioned(RenderAPI::OpenGL);
        memset(reserved, 0, sizeof(reserved));
    }
/*** Render Context Resources. Foreign context (provided by user) only ***/
    int fbo = -1; // if >=0, will draw in given fbo. no need to bind in user code
    int unused = 0;
/*
  \brief getProcAddress
  optional. can be null and then standard gl libraries will be searched.
  if not null, it's used to load gl functions
  \param name gl function name
  \param opaque user data, e.g. gl context handle
*/
    void* (*getProcAddress)(const char* name, void* opaque) = nullptr;
    void* (*getCurrentNativeContext)(void* opaque) = nullptr;
/*!
  \brief opaque
  optional. getProcAddress user data, e.g. a gl context handle.
*/
    void* opaque = nullptr;

/***
  Render Context Creation Options.
  as input, they are desired values to create an internal context(ignored if context is provided by user). as output, they are result values(if context is not provided by user)
***/
    enum class Profile : uint8_t {
        No,
        Core,
        Compatibility,
    };

    bool debug = false; /* default false. NOT IMPLENETED */
    int8_t egl = -1; /* default -1. -1: auto. 0: no, 1: try */
/* if any one of opengl and opengles is 0, then another is treated as 1 */
    int8_t opengl = -1; /* default -1. -1: auto. 0: no, 1: try */
    int8_t opengles = -1; /* default -1. -1: auto. 0: no, 1: try */
    Profile profile = Profile::Core; /* default 3. 0: no profile, 1: core profile, 2: compatibility profile */
    float version = 0; /* default 0, ignored if < 2.0. requested version major.minor. result version may < requested version if not supported */
    int8_t reserved[32];
};

struct MetalRenderAPI final: RenderAPI {
    MetalRenderAPI() {
        type_ = versioned(RenderAPI::Metal);
        memset(reserved, 0, sizeof(reserved));
    }
/*** Render Context Resources. Foreign context (provided by user) only ***/
// id<?> => void*: to be compatible with c++
    const void* device = nullptr; // MUST set if metal is provided by user
    const void* cmdQueue = nullptr; // optional. if not null, device can be null. currentQueue callback to share the same command buffer?
/* one of texture and currentRenderTarget MUST be set if metal is provided by user */
    const void* texture = nullptr; // optional. id<MTLTexture>. if not null, device can be null. usually for offscreen rendering. render target for MTLRenderPassDescriptor if encoder is not provided by user. set once for offscreen rendering
    const void* opaque = nullptr; // optional. callback opaque
    const void* (*currentRenderTarget)(const void* opaque) = nullptr; // optional. usually for on screen rendering. return id<MTLTexture>.
    // no encoder because we need own render pass
    const void* layer = nullptr; // optional. CAMetalLayer only used for appling colorspace parameters for hdr/sdr videos.
    const void* reserved[1];

/***
  Render Context Creation Options.
  as input, they are desired values to create an internal context(ignored if context is provided by user). as output, they are result values(if context is not provided by user)
***/
    // device options: macOS only
    int device_index = -1; // -1 will use system default device. callback with index+name?
};

/*!
  NOTE: include d3d11.h first to use D3D11RenderAPI
 */
#if defined(D3D11_SDK_VERSION)
struct D3D11RenderAPI : RenderAPI {
    D3D11RenderAPI(ID3D11DeviceContext* c = nullptr, ID3D11DeviceChild* r = nullptr) : context(c), rtv(r) {
        type_ = versioned(RenderAPI::D3D11);
        memset(reserved, 0, sizeof(reserved));
    }
/*** Render Context Resources. Foreign context (provided by user) only ***/
/*
  context and rtv can be set by user if user can provide. then rendering becomes foreign context mode.
  if rtv is not null, no need to set context
  \sa Player.setRenderAPI()
 */
    ID3D11DeviceContext* context = nullptr;
    // rtv or texture. usually user can provide a texture from gui easly, no d3d code to create a view
    ID3D11DeviceChild* rtv = nullptr; // optional. the render target(view). ID3D11RenderTargetView or ID3D11Texture2D. can be null if context is not null. if not null, no need to set context
    void* reserved[2];

/***
  Render Context Creation Options.
  as input, they are desired values to create an internal context(ignored if context is provided by user). as output, they are result values(if context is not provided by user)
***/
    bool debug = false;
    int buffers = 2; /* UWP must >= 2. */
    int adapter = 0; /* adapter index */
    float feature_level = 0; /* 0 is the highest */
    const char* vendor = nullptr; /* gpu vendor name */
};
#endif

/*!
  NOTE: include d3d12.h first to use D3D12RenderAPI
 */
#if defined(__d3d12_h__)// D3D12_SDK_VERSION: not defined in 19041
struct D3D12RenderAPI : RenderAPI {
    D3D12RenderAPI(ID3D12CommandQueue* cq = nullptr, ID3D12Resource* r = nullptr) : cmdQueue(cq), rt(r) {
        type_ = versioned(RenderAPI::D3D12);
    }
/*** Render Context Resources. Foreign context (provided by user) only ***/
    ID3D12CommandQueue* cmdQueue = nullptr; // optional. will create an internal queue if null.
    ID3D12Resource* rt = nullptr; // optional. the render target
    D3D12_CPU_DESCRIPTOR_HANDLE rtvHandle = {}; // optional
    void* reserved[2] = {};

    const void* opaque = nullptr; // optional. callback opaque
    ID3D12Resource* (*currentRenderTarget)(const void* opaque, UINT* index, UINT* count, D3D12_RESOURCE_STATES* state) = nullptr; // optional. usually for on screen rendering.
    void* reserved2[2] = {};

/***
  Render Context Creation Options.
  as input, they are desired values to create an internal context(ignored if context is provided by user). as output, they are result values(if context is not provided by user)
***/
    bool debug = false;
    int buffers = 2; /* must >= 2. */
    int adapter = 0; /* adapter index */
    float feature_level = 0; /* 0 is the highest */
    const char* vendor = nullptr; /* gpu vendor name */
};
#endif


// always declare
struct VulkanRenderAPI final : RenderAPI {
    VulkanRenderAPI() {
        type_ = versioned(RenderAPI::Vulkan);
        memset(reserved, 0, sizeof(reserved));
        memset(reserved_opt, 0, sizeof(reserved_opt));
    }

#if (VK_VERSION_1_0+0)
    VkInstance instance = VK_NULL_HANDLE; // OPTIONAL. shared instance. for internal created context but not foreign context, to load instance extensions
    VkPhysicalDevice phy_device = VK_NULL_HANDLE; // Optional to create internal context. MUST not null for foreign context. Must set if logical device is provided to create internal context.
    VkDevice device = VK_NULL_HANDLE; // Optional to create internal context as shared device. Required for foreign context.
    VkQueue graphics_queue = VK_NULL_HANDLE; // OPTIONAL. If null, will use gfx_queue_index. NOT required if vk is create internally
/*!
  \brief rt
  Used by offscreen rendering.
 */
    VkImage rt = VK_NULL_HANDLE; // VkImage? so can use qrhitexture.nativeTexture().object
    VkRenderPass render_pass = VK_NULL_HANDLE; // optional. If null(usually for offscreen rendering), final image layout is VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL
    void* opaque = nullptr;
/*!
  \brief renderTargetInfo
  Get render target image size
  \param format image format. MUST be set if framebuffer from beginFrame() is null
  \param finalLayout image final layout. No transition if undefined. Transition can also be in endFrame() callback if needed, then finalLayout here can be undefined.
  NOTE: assume transition is in the same graphics queue family.
  \return (render target)image count, e.g. swapchain image count.
 */
    int (*renderTargetInfo)(void* opaque, int* w, int* h, VkFormat* format, VkImageLayout* finalLayout); // return count
/*!
  \brief beginFrame
  Optional. Can be null(or not) for offscreen rendering if rt is not null.
  MUST be paired with endFrame()
  \param fb can be null, then will create internally. if not null, MUST set render_pass
  \param imgSem from present queue. can be null if fulfill any of
  // TODO: VkImage?
  1. present queue == gfx queue
  2. getCommandBuffer() is provided and submit in user code
  \return image index.
*/
    int (*beginFrame)(void* opaque, VkImageView* view/* = nullptr*/, VkFramebuffer* fb/*= nullptr*/, VkSemaphore* imgSem/* = nullptr*/) = nullptr;
    // int getNextImageView(); // not fbo, fbo is bound to render pass(can be dummy tmp). image view can also be used by compute pipeline. return index
/*!
  \brief currentCommandBuffer()
  if null, create pool internally(RTT)
 */
    VkCommandBuffer (*currentCommandBuffer)(void* opaque) = nullptr;
/*!
  \brief endFrame
  Optional. If null, frame is guaranteed to be rendered to image before executing the next command buffer in user code.
  If not null, user can wait for drawSem before using the image.
  MUST be paired with beginFrame()
  \param drawSem from gfx queue. can be null if fulfill any of
  1. present queue == gfx queue
  2. getCommandBuffer() is provided and submit in user code
  3. RTT offscreen rendering, i.e. rtv is set and beginFrame is null(user should wait for draw finish too)
 */
    void (*endFrame)(void* opaque, VkSemaphore* drawSem/* = nullptr*/) = nullptr; // can be null if offscreen. wait drawSem before present
#endif // (VK_VERSION_1_0+0)
    void* reserved[2];
/*
  Set by user and used internally even if device is provided by user
 */
    int graphics_family = -1; // MUST if graphics and transfer queue family are different
    int compute_family = -1; // optional. it's graphics_family if not set
    int transfer_family = -1; // optional. it's graphics_family if not set
    int present_family = -1; // optional. Must set if logical device is provided to create internal context
/***
  Render Context Creation Options.
  as input, they are desired values to create an internal context(ignored if context is provided by user). as output, they are result values(if context is not provided by user)
***/
    bool debug = false;
    uint8_t buffers = 2; // 2 for double-buffering
    int device_index = -1;
    uint32_t max_version = 0; // requires vulkan 1.1
    int gfx_queue_index = 0; // OPTIONAL
    int transfer_queue_index = -1; // OPTIONAL. if not set, will use gfx queue
    int compute_queue_index = -1; // OPTIONAL. if not set, will use gfx queue

    int depth = 8;
    //const char*
    uint8_t reserved_opt[32]; // color space etc.
};
MDK_NS_END
```

### `mdk/include/VideoFrame.h`

```cpp
/*
 * Copyright (c) 2019-2023 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include "global.h"
#include "c/VideoFrame.h"
#include <algorithm>

MDK_NS_BEGIN

enum class PixelFormat
{
    Unknown = 0,
    YUV420P,
    NV12,
    YUV422P,
    YUV444P,
    P010LE,
    P016LE,
    YUV420P10LE,
    UYVY422,
    RGB24,
    RGBA,
    RGBX,
    BGRA,
    BGRX,
    RGB565LE,
    RGB48LE,
    RGB48 = RGB48LE,
    GBRP,
    GBRP10LE,
    XYZ12LE,
    YUVA420P,
    BC1,
    BC3,
    RGBA64,         // name: "rgba64le"
    BGRA64,         // name: "bgra64le"
    RGBP16,         // name: "rgbp16le"
    RGBPF32,        // name: "rgbpf32le"
    BGRAF32,        // name: "bgraf32le"
};

static inline bool operator!(PixelFormat f) { return f == PixelFormat::Unknown; }

class VideoFrame
{
public:
    VideoFrame(const VideoFrame&) = delete;
    VideoFrame& operator=(const VideoFrame&) = delete;
    VideoFrame(VideoFrame&& that) {
        std::swap(p, that.p);
        std::swap(owner_, that.owner_);
    }
    VideoFrame& operator=(VideoFrame&& that)  {
        std::swap(p, that.p);
        std::swap(owner_, that.owner_);
        return *this;
    }

    VideoFrame() = default;
/*!
  \brief VideoFrame
  Construct a video frame for given format, size. If strides is not null, a single contiguous memory for all planes will be allocated.
  If data is not null, data is copied to allocated memory.
  \param width visual width
  \param height visual height
  \sa setBuffers
   NOTE: Unkine setBuffers(), no memory is allocated for null strides.
 */
    VideoFrame(int width, int height, PixelFormat format, int* strides/*in/out*/ = nullptr, uint8_t const** const data/*in/out*/ = nullptr) {
        p = mdkVideoFrameAPI_new(width, height, MDK_PixelFormat(int(format) - 1));
        if (data)
            MDK_CALL(p, setBuffers, data, strides);
    }

    VideoFrame(mdkVideoFrameAPI* pp) : p(pp) {}

    ~VideoFrame() {
        if (owner_)
            mdkVideoFrameAPI_delete(&p);
    }

// isValid() is true for EOS frame, but no data and timestamp() is TimestampEOS.
    bool isValid() const { return !!p; }
    explicit operator bool() const { return isValid();}

    void attach(mdkVideoFrameAPI* api) {
        if (owner_)
            mdkVideoFrameAPI_delete(&p);
        p = api;
        owner_ = false;
    }

    mdkVideoFrameAPI* detach() {
        auto ptr = p;
        p = nullptr;
        return ptr;
    }

    mdkVideoFrameAPI* toC() const {
        return p;
    }

    int planeCount() const { return MDK_CALL(p, planeCount); }

    int width(int plane = -1) const {
        return MDK_CALL(p, width, plane);
    }

    int height(int plane = -1) const {
        return MDK_CALL(p, height, plane);
    }

    PixelFormat format() const {
        return (PixelFormat)(int(MDK_CALL(p, format)) + 1);
    }
/*!
  \brief addBuffer
  Add an external buffer to nth plane, store external buffer data ptr. The old buffer will be released.
  \param data external buffer data ptr
  \param stride stride of data. if <=0, it's the stride of current format at this plane
  \param buf external buffer ptr. user should ensure the buffer is alive before frame is destroyed.
  \param bufDeleter to delete buf when frame is destroyed
 */
    bool addBuffer(const uint8_t* data, int stride, void* buf, void (*bufDeleter)(void** pBuf), int plane = -1) {
        return MDK_CALL(p, addBuffer, data, stride, buf, bufDeleter, plane);
    }
/*
   \brief setBuffers
   Add buffers with data copied from given source. Unlike constructor, a single contiguous memory for all planes is always allocated.
   If data is not null, data is copied to allocated memory.
   \param data array of source data planes, array size MUST >= plane count of format if not null. Can be null and allocate memory without copying.
   NOTE: data[i] will be filled with allocated plane address if necessary(data != null && strides != null).
   If data[0] != null but data[i] == null, assume copying from contiguous source data.
   \param strides array of plane strides, size MUST >= plane count of format if not null. Can be null and strides[i] can be <=0 indicating no padding bytes (for plane i).
   NOTE: strides[i] will be filled with allocated plane i stride if necessary(strides[i] <= 0)
 */
    void setBuffers(uint8_t const** const data, int* strides/*in/out*/ = nullptr) {
        MDK_CALL(p, setBuffers, data, strides);
    }

    const uint8_t* bufferData(int plane = 0) const {
        return MDK_CALL(p, bufferData, plane);
    }

    int bytesPerLine(int plane = 0) const {
        return MDK_CALL(p, bytesPerLine, plane);
    }

    void setTimestamp(double t) {
        return MDK_CALL(p, setTimestamp, t);
    }

    double timestamp() const {
        if (!p)
            return -1;
        return MDK_CALL(p, timestamp);
    }
/*!
  \brief to
  The result frame data is always on host memory. If it's already on host memory and the same as target format, return the current frame.
  NOTE: compressed input/output formats are not supported
  \param fmt output format. if invalid, same as format()
  \param width output width. if invalid(<=0), same as width()
  \param height output height. if invalid(<=0), same as height()
  if all output parameters(invalid) are the same as input, return self
  \return Invalid frame if failed
 */
    VideoFrame to(PixelFormat format, int width = -1, int height = -1) {
        return VideoFrame(MDK_CALL(p, to, MDK_PixelFormat(int(format)-1), width, height));
    }
/*!
  \brief save
  Saves the frame to the file with the given fileName, using the given image file format and quality factor.
  Save the original frame data if:
  - fileName extension is the same as format().name()
  - fileName has no extension, and format is null
  - format is the same as format().name()
  \param format if null, guess the format by fileName's suffix
  \param quality must be in the range 0.0 to 1.0 or -1. Specify 0 to obtain small compressed files, 100 for large uncompressed files, and -1 (the default) to use the default settings.
  \returns true if the frame was successfully saved; otherwise returns false.
 */
    bool save(const char* fileName, const char* format = nullptr, float quality = -1) const {
        return MDK_CALL(p, save, fileName, format, quality);
    }
private:
    mdkVideoFrameAPI* p = nullptr;
    bool owner_ = true;
};

MDK_NS_END
```

### `mdk/include/c/MediaInfo.h`

```cpp
/*
 * Copyright (c) 2019-2023 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include "global.h"

#ifdef __cplusplus
extern "C" {
#endif

typedef struct mdkAudioCodecParameters {
    const char* codec;
    uint32_t codec_tag;
    const uint8_t* extra_data; /* without padding data */
    int extra_data_size;
    int64_t bit_rate;
    int profile;
    int level;
    float frame_rate;

    bool is_float;
    bool is_unsigned;
    bool is_planar;
    int raw_sample_size;

    int channels;
    int sample_rate;
    int block_align;
    int frame_size; /* const samples per channel in a frame */

    char reserved[128]; /* color info etc. */
} mdkAudioCodecParameters;

typedef struct mdkAudioStreamInfo {
    int index;
    int64_t start_time; /* ms */
    int64_t duration; /* ms */
    int64_t frames;

    const void* priv;
} mdkAudioStreamInfo;

MDK_API void MDK_AudioStreamCodecParameters(const mdkAudioStreamInfo*, mdkAudioCodecParameters* p);
/* see document of mdkStringMapEntry */
MDK_API bool MDK_AudioStreamMetadata(const mdkAudioStreamInfo*, mdkStringMapEntry* entry);

typedef struct mdkVideoCodecParameters {
    const char* codec;
    uint32_t codec_tag;
    const uint8_t* extra_data; /* without padding data */
    int extra_data_size;
    int64_t bit_rate;
    int profile;
    int level;
    float frame_rate;
    int format;
    const char* format_name;

    int width;
    int height;
    int b_frames;

    float par;
    char reserved[128];
} mdkVideoCodecParameters;

typedef struct mdkVideoStreamInfo {
    int index;
    int64_t start_time;
    int64_t duration;
    int64_t frames;
    int rotation;

    const void* priv;
    // TODO: struct_size for ABI compatibility
} mdkVideoStreamInfo;

MDK_API void MDK_VideoStreamCodecParameters(const mdkVideoStreamInfo*, mdkVideoCodecParameters* p);
/* see document of mdkStringMapEntry */
MDK_API bool MDK_VideoStreamMetadata(const mdkVideoStreamInfo*, mdkStringMapEntry* entry);
MDK_API const uint8_t* MDK_VideoStreamData(const mdkVideoStreamInfo*, int* len, int flags);

typedef struct mdkSubtitleCodecParameters {
    const char* codec ;
    uint32_t codec_tag;
    const uint8_t* extra_data;
    int extra_data_size;

    int width;  /* display width. bitmap subtitles only */
    int height; /* display height. bitmap subtitles only */
} mdkSubtitleCodecParameters;

typedef struct mdkSubtitleStreamInfo {
    int index;
    int64_t start_time;
    int64_t duration;

    const void* priv;
} mdkSubtitleStreamInfo;

MDK_API void MDK_SubtitleStreamCodecParameters(const mdkSubtitleStreamInfo*, mdkSubtitleCodecParameters* p);
MDK_API bool MDK_SubtitleStreamMetadata(const mdkSubtitleStreamInfo*, mdkStringMapEntry* entry);

typedef struct mdkChapterInfo {
    int64_t start_time;
    int64_t end_time;
    const char* title; /* NULL if no title */

    const void* priv;
} mdkChapterInfo;

typedef struct mdkProgramInfo {
    int id;
    const int* stream; /* stream index */
    int nb_stream;

    const void* priv;
} mdkProgramInfo;

MDK_API bool MDK_ProgramMetadata(const mdkProgramInfo*, mdkStringMapEntry* entry);

typedef struct mdkMediaInfo
{
    int64_t start_time; /* ms */
    int64_t duration;
    int64_t bit_rate;
    int64_t size; /* file size. IGNORE THIS */
    const char* format;
    int streams;

    mdkAudioStreamInfo* audio;
    int nb_audio;
    mdkVideoStreamInfo* video;
    int nb_video;
    mdkSubtitleStreamInfo* subtitle;
    int nb_subtitle;

    const void* priv;

    mdkChapterInfo* chapters;
    int nb_chapters;

    mdkProgramInfo* programs;
    int nb_programs;
} mdkMediaInfo;

/* see document of mdkStringMapEntry */
MDK_API bool MDK_MediaMetadata(const mdkMediaInfo*, mdkStringMapEntry* entry);

#ifdef __cplusplus
}
#endif
```

### `mdk/include/c/Player.h`

```cpp
/*
 * Copyright (c) 2019-2022 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include "global.h"
#include "RenderAPI.h"
#include <stddef.h>

#ifdef __cplusplus
extern "C" {
#endif
struct mdkMediaInfo;
struct mdkAudioFrame;
struct mdkVideoFrameAPI;
struct mdkPlayer;

enum MDK_SurfaceType {
    MDK_SurfaceType_Auto, /* platform default type */
    MDK_SurfaceType_X11,
    MDK_SurfaceType_GBM,
    MDK_SurfaceType_Wayland,
};

typedef struct mdkCurrentMediaChangedCallback {
    void (*cb)(void* opaque);
    void* opaque;
} mdkCurrentMediaChangedCallback;

/*!
  \brief mdkPrepareCallback
  \param position in callback is the actual position, or <0 (TODO: error code as position) if prepare() failed.
  \param boost in callback can be set by user to boost the first frame rendering
  \return false to unload media immediately when media is loaded and MediaInfo is ready, true to continue.
    example: always return false can be used as media information reader
 */
typedef struct mdkPrepareCallback {
    bool (*cb)(int64_t position, bool* boost, void* opaque);
    void* opaque;
} mdkPrepareCallback;

typedef struct mdkRenderCallback {
    void (*cb)(void* vo_opaque, void* opaque);
    void* opaque;
} mdkRenderCallback;

typedef struct mdkVideoCallback {
    int (*cb)(struct mdkVideoFrameAPI** pFrame/*in/out*/, int track, void* opaque);
    void* opaque;
} mdkVideoCallback;

typedef struct SwitchBitrateCallback {
    void (*cb)(bool, void* opaque);
    void* opaque;
} SwitchBitrateCallback;

typedef struct mdkSeekCallback {
    void (*cb)(int64_t ms, void* opaque);
    void* opaque;
} mdkSeekCallback;

/*!
  \brief TimeoutCallback
  \param ms elapsed milliseconds
  \return true to abort current operation on timeout.
  A null callback can abort current operation.
  Negative timeout infinit.
  Default timeout is 10s
 */
typedef struct mdkTimeoutCallback {
    bool (*cb)(int64_t ms, void* opaque);
    void* opaque;
} mdkTimeoutCallback;

/*!
  \brief MediaEventCallback
  \return true if event is processed and should stop dispatching.
 */
typedef struct mdkMediaEventCallback {
    bool (*cb)(const mdkMediaEvent*, void* opaque);
    void* opaque;
} mdkMediaEventCallback;

typedef struct mdkLoopCallback {
    void (*cb)(int, void* opaque);
    void* opaque;
} mdkLoopCallback;

typedef struct mdkSnapshotRequest {
/* data: rgba or bgra data. Created internally or provided by user.
   If data is provided by user, stride,  height and width MUST be also set, and data MUST be valid until snapshot callback is finished.
 */
    uint8_t* data;
/*
   result width of snapshot image set by user, or the same as current frame width if 0. no renderer transform.
   if both requested width and height are < 0, then result image is scaled image of current frame with ratio=width/height. no renderer transform.
   if only one of width and height < 0, then the result size is video renderer viewport size, and all transforms will be applied.
   if both width and height == 0, then result size is region of interest size of video frame set by setPointMap(), or video frame size
*/
    int width;
    int height;
    int stride;
    bool subtitle; // not supported yet
} mdkSnapshotRequest;

enum MDK_MapDirection {
    MDK_MapDirection_FrameToViewport, // left-hand
    MDK_MapDirection_ViewportToFrame, // left-hand
};

typedef struct mdkSnapshotCallback {
/* \brief cb
   snapshot callback.
   \param req result request. If null, snapshot failed. Otherwise req.width, height and stride are always >0, data is never null.
   \param frameTime captured frame timestamp(seconds)
   \param opaque user data
   \returns null, or a file path to save as a file(jpeg is recommended, other formats depends on ffmpeg runtime).
   Returned string will be freed internally(assume allocated by malloc family apis).
   Callback is called in a dedicated thread, so time-consuming operations(encode, file io etc.) are allowed in the callback.
 */
    char* (*cb)(mdkSnapshotRequest* req, double frameTime, void* opaque);
    void* opaque;
} mdkSnapshotCallback;

typedef struct mdkSyncCallback {
    double (*cb)(void* opaque);
    void* opaque;
} mdkSyncCallback;


typedef struct mdkPlayerAPI {
    struct mdkPlayer* object;

    void (*setMute)(struct mdkPlayer*, bool value);
    void (*setVolume)(struct mdkPlayer*, float value);

/*!
  \brief setMedia
  Set a new media url.  If url changed, will stop current playback, and reset active tracks, external tracks set by setMedia(url, type)
  // MUST call setActiveTracks() after setMedia(), otherwise the 1st track in the media is used
 */
    void (*setMedia)(struct mdkPlayer*, const char* url);
/* Set individual source for type, e.g. audio track file. If url is not empty, an individual pipeline will be used for 'type' tracks.
  If url is empty, use 'type' tracks in MediaType::Video url.
  MUST be after main media setMedia(url).
  TODO: default type is Unknown
*/
    void (*setMediaForType)(struct mdkPlayer*, const char* url, MDK_MediaType type);
    const char* (*url)(struct mdkPlayer*);

    void (*setPreloadImmediately)(struct mdkPlayer*, bool value);
/*!
  \brief setNextMedia
  Gapless play the next media after current media playback end
  \param flags seek flags if startPosition > 0, accurate or fast
  setState(State::Stopped) only stops current media. Call setNextMedia(nullptr, -1) first to disable next media.
  Usually you can call `currentMediaChanged()` to set a callback which invokes `setNextMedia()`, then call `setMedia()`.
 */
    void (*setNextMedia)(struct mdkPlayer*, const char* url, int64_t startPosition, enum MDKSeekFlag flags);

/*!
  \brief currentMediaChanged
  Set a callback which is invoked when current media is stopped and a new media is about to play, or when setMedia() is called.
  Call before setMedia() to take effect.
 */
    void (*currentMediaChanged)(struct mdkPlayer*, mdkCurrentMediaChangedCallback cb);
/* backends can be: AudioQueue(Apple only), OpenSL(Android only), ALSA(linux only), XAudio2(Windows only), OpenAL
  ends with NULL
*/
    void (*setAudioBackends)(struct mdkPlayer*, const char** names);
    void (*setAudioDecoders)(struct mdkPlayer*, const char** names);
    void (*setVideoDecoders)(struct mdkPlayer*, const char* names[]);

    void (*setTimeout)(struct mdkPlayer*, int64_t value, mdkTimeoutCallback cb);
/*!
  \brief prepare
  Preload a media and then becomes State::Paused. \sa PrepareCallback
  To play a media from a given position, call prepare(ms) then setState(State::Playing)
  \param startPosition start from position, relative to media start position(i.e. MediaInfo.start_time)
  \param cb if startPosition > 0, same as callback of seek(), called after the first frame is decoded or load/seek/decode error. If startPosition == 0, called when media is loaded and mediaInfo is ready or load error.
  \param flags seek flag if startPosition != 0.
  For fast seek(has flag SeekFlag::Fast), the first frame is a key frame whose timestamp >= startPosition
  For accurate seek(no flag SeekFlag::Fast), the first frame is the nearest frame whose timestamp <= startPosition, but the position passed to callback is the key frame position <= startPosition
 */
    void (*prepare)(struct mdkPlayer*, int64_t startPosition, mdkPrepareCallback cb, enum MDKSeekFlag flags);
    const struct mdkMediaInfo* (*mediaInfo)(struct mdkPlayer*); /* NOT IMPLEMENTED*/

/*!
  \brief setState
  Request a new state. It's async and may take effect later.
  setState(State::Stopped) only stops current media. Call setNextMedia(nullptr, -1) before stop to disable next media.
  setState(State::Stopped) will release all resouces and clear video renderer viewport. While a normal playback end will keep renderer resources
  and the last video frame. Manually call setState(State::Stopped) to clear them.
  NOTE: the requested state is not queued. so set one state immediately after another may have no effect.
  e.g. State::Playing after State::Stopped may have no effect if playback have not been stopped and still in Playing state
  so the final state is State::Stopped. Current solution is waitFor(State::Stopped) before setState(State::Playing).
  Usually no waitFor(State::Playing) because we want async load
*/
    void (*setState)(struct mdkPlayer*, MDK_State value);
    MDK_State (*state)(struct mdkPlayer*);
    void (*onStateChanged)(struct mdkPlayer*, mdkStateChangedCallback);
    bool (*waitFor)(struct mdkPlayer*, MDK_State value, long timeout);

    MDK_MediaStatus (*mediaStatus)(struct mdkPlayer*);
/*!
  \brief onMediaStatusChanged
  Add a callback to be invoked when MediaStatus is changed
  \param cb null to clear callbacks
  DEPRECATED: use onMediaStatus instead
 */
    void (*onMediaStatusChanged)(struct mdkPlayer*, mdkMediaStatusChangedCallback);

/*!
 * \brief updateNativeSurface
 * If surface is not created, create rendering context internally by createSurface() and attached to native surface
 * native surface MUST be not null before destroying player
 type: ignored if win ptr does not change (request to resize)
 */
    void (*updateNativeSurface)(struct mdkPlayer*, void* surface, int width, int height, enum MDK_SurfaceType type);

    void (*createSurface)(struct mdkPlayer*, void* nativeHandle, enum MDK_SurfaceType type);
    void (*resizeSurface)(struct mdkPlayer*, int w, int h);
    void (*showSurface)(struct mdkPlayer*);

/*
  vo_opaque: a ptr to identify the renderer. can be null, then it is the default vo/renderer.
  A vo/renderer (e.g. the default vo/renderer) is gfx context aware, i.e. can render in multiple gfx contexts with a single vo/renderer, but parameters(e.g. surface size)
  must be updated when switch to a new context. So per gfx context vo/renderer can be better because parameters are stored in vo/renderer.
*/
/*!
  \brief getVideoFrame
  get current rendered frame, i.e. the decoded video frame rendered by renderVideo()
 */
    void (*getVideoFrame)(); /* NOT IMPLEMENTED*/
/*
  \brief setVideoSurfaceSize
  Window size, surface size or drawable size. Render callback(if exists) will be invoked if width and height > 0.
  Usually for foreign contexts, i.e. not use updateNativeSurface().
NOTE:
  If width or heigh < 0, corresponding video renderer (for vo_opaque) will be removed and gfx resources will be released(need the context to be current for GL).
  But subsequence call with this vo_opaque will create renderer again. So it can be used before destroying the renderer.
  OpenGL: resources must be released by setVideoSurfaceSize(-1, -1, ...) in a correct context. If player is destroyed before context, MUST call Player::foreignGLContextDestroyed() when destroying the context.
 */
    void (*setVideoSurfaceSize)(struct mdkPlayer*, int width, int height, void* vo_opaque);
    void (*setVideoViewport)(struct mdkPlayer*, float x, float y, float w, float h, void* vo_opaque);
/*!
  \brief setAspectRatio
  Video display aspect ratio.
  0: ignore aspect ratio and scale to fit renderer viewport
  FLT_EPSILON(default): keep frame aspect ratio and scale as large as possible inside renderer viewport
  -FLT_EPSILON: keep frame aspect ratio and scale as small as possible outside renderer viewport
  other value > 0: keep given aspect ratio and scale as large as possible inside renderer viewport
  other value < 0: keep given aspect ratio and scale as small as possible inside renderer viewport
 */
    void (*setAspectRatio)(struct mdkPlayer*, float value, void* vo_opaque);
    void (*rotate)(struct mdkPlayer*, int degree, void* vo_opaque);
    void (*scale)(struct mdkPlayer*, float x, float y, void* vo_opaque);
/*!
   \brief renderVideo
  Render the next or current(redraw) frame. Foreign render context only (i.e. not created by createSurface()/updateNativeSurface()).
  OpenGL: Can be called in multiple foreign contexts for the same vo_opaque.
   \return timestamp of rendered frame, or < 0 if no frame is rendered
 */
    double (*renderVideo)(struct mdkPlayer*, void* vo_opaque);
/*!
  \brief setBackgroundColor
  r, g, b, a range is [0, 1]. default is 0. if out of range or a == 0, background color will not be filled
 */
    void (*setBackgroundColor)(struct mdkPlayer*, float r, float g, float b, float a, void* vo_opaque);

/*!
  \brief setRenderCallback
  set a callback which is invoked when the vo coresponding to vo_opaque needs to update/draw content, e.g. when a new frame is received in the renderer.
  Also invoked in setVideoSurfaceSize(), setVideoViewport(), setAspectRatio() and rotate(), take care of dead lock in callback and above functions.
  with vo_opaque, user can know which vo/renderer is rendering, useful for multiple renderers
  There may be no frames or playback not even started, but renderer update is required internally
*/
    void (*setRenderCallback)(struct mdkPlayer*, mdkRenderCallback);

/*
  \brief onVideo
  Called before delivering frame to renderers. Can be used to apply filters.
 */
    void (*onVideo)(struct mdkPlayer*, mdkVideoCallback);
    void (*onAudio)(struct mdkPlayer*); // NOT IMPLEMENTED
/*
  \brief beforeVideoRender
  NOT IMPLEMENTED. Called after rendering a frame on renderer of vo_opaque on rendering thread. Can be used to apply GPU filters.
 */
    void (*beforeVideoRender)(struct mdkPlayer*, void (*)(struct mdkVideoFrameAPI*, void* vo_opaque));
/*
  \brief beforeVideoRender
  NOT IMPLEMENTED. Called after rendering a frame on renderer of vo_opaque on rendering thread. Can be used to draw a watermark.
 */
    void (*afterVideoRender)(struct mdkPlayer*, void (*)(struct mdkVideoFrameAPI*, void* vo_opaque));

    int64_t (*position)(struct mdkPlayer*);
/*!
  \brief seekWithFlags
  \param pos seek target. if flags has SeekFlag::Frame, pos is frame count, otherwise it's milliseconds.
  If pos > media time range, e.g. INT64_MAX, will seek to the last frame of media for SeekFlag::AnyFrame, and the last key frame of media for SeekFlag::Fast.
  If pos > media time range with SeekFlag::AnyFrame, playback will stop unless setProperty("continue_at_end", "1") was called
  FIXME: a/v sync broken if SeekFlag::Frame|SeekFlag::FromNow.
  \param cb if succeeded, callback is called when stream seek finished and after the 1st frame decoded or decode error(e.g. video tracks disabled), ret(>=0) is the timestamp of the 1st frame(video if exists) after seek.
  If error(io, demux, not decode) occured(ret < 0, usually -1) or skipped because of unfinished previous seek(ret == -2), out of range(-4) or media unloaded(-3).
 */
    bool (*seekWithFlags)(struct mdkPlayer*, int64_t pos, MDK_SeekFlag flags, mdkSeekCallback);
    bool (*seek)(struct mdkPlayer*, int64_t pos, mdkSeekCallback);

    void (*setPlaybackRate)(struct mdkPlayer*, float value);
    float (*playbackRate)(struct mdkPlayer*);
/*!
 * \brief buffered
 * get buffered data(packets) duration and size
 * \return buffered data duration
 */
    int64_t (*buffered)(struct mdkPlayer*, int64_t* bytes);
/*!
  \brief switchBitrate
  A new media will be played later
  \param delay switch after at least delay ms. TODO: determined by buffered time, e.g. from high bit rate without enough buffered samples to low bit rate
  \param cb (true/false) called when finished/failed
  \param flags seek flags for the next url, accurate or fast
 */
    void (*switchBitrate)(struct mdkPlayer*, const char* url, int64_t delay, SwitchBitrateCallback cb);
/*!
 * \brief switchBitrateSingalConnection
 * Only 1 media is loaded. The previous media is unloaded and the playback continues. When new media is preloaded, stop the previous media at some point
 * MUST call setPreloadImmediately(false) because PreloadImmediately for singal connection preload is not possible.
 * \return false if preload immediately
 * This will not affect next media set by user
 */
    bool (*switchBitrateSingleConnection)(struct mdkPlayer*, const char *url, SwitchBitrateCallback cb);

    void (*onEvent)(struct mdkPlayer*, mdkMediaEventCallback cb, MDK_CallbackToken* token);
/*!
  \brief bufferRange
  set duration range of buffered data.
  \param minMs default 1000. wait for buffered duration >= minMs when before popping a packet from to decode
    If minMs < 0, then minMs, maxMs and drop will be reset to the default value
  \param maxMs default 4000. max buffered duration.
    If maxMs < 0, then maxMs and drop will be reset to the default value
    If maxMs == 0, same as INT64_MAX
  drop = true: drop old non-key frame packets to reduce buffered duration until < maxMs.
  drop = false: wait for buffered duration < maxMs before pushing packets

  For realtime streams like(rtp, rtsp, rtmp, sdp etc.), the default range is [0, INT64_MAX, true].
  Usually you don't need to call this api. This api can be used for low latency live videos, for example setBufferRange(0, 1000, true) will decode as soon as possible when media data received, also it ensures the max delay of rendered video is 1s, and no accumulated delay.
 */
    void (*setBufferRange)(struct mdkPlayer*, int64_t minMs, int64_t maxMs, bool drop);
/*!
  \brief snapshot
  take a snapshot from current renderer. The result is in bgra format, or null on failure.
  When `snapshot()` is called, redraw is scheduled for `vo_opaque`'s renderer, then renderer will take a snapshot in rendering thread.
  So for a foreign context, if renderer's surface/window/widget is invisible or minimized, snapshot may do nothing because of system or gui toolkit painting optimization.
*/
    void (*snapshot)(struct mdkPlayer*, mdkSnapshotRequest* request, mdkSnapshotCallback cb, void* vo_opaque);

/*
  \brief setProperty
  Set additional properties. Can be used to store user data, or change player behavior if the property is defined internally.
  Predefined properties are:
  - "video.avfilter": ffmpeg avfilter filter graph string for video track. take effect immediately
  - "audio.avfilter": ffmpeg avfilter filter graph string for audio track. take effect immediately
  - "continue_at_end" or "keep_open": "0" or "1". do not stop playback when decode and render to end of stream. only set(State::Stopped) can stop playback. Useful for timeline preview.
  - "cc": "0" or "1"(default). enable closed caption decoding and rendering.
  - "subtitle": "0" or "1"(default). enable subtitle(including cc) rendering. setActiveTracks(MediaType::Subtitle, {...}) enables decoding only.
  - "avformat.some_name": avformat option, e.g. {"avformat.fpsprobesize": "0"}. if global option "demuxer.io=0", it also can be AVIOContext/URLProtocol option
  - "avio.some_name": AVIOContext/URLProtocol option, e.g. "avio.user_agent"
 */
    void (*setProperty)(struct mdkPlayer*, const char* key, const char* value);
/*!
  \brief setProperty
  \return value for key, or null if no such key
 */
    const char* (*getProperty)(struct mdkPlayer*, const char* key);
/*
  \brief record
  Start to record or stop recording current media by remuxing packets read. If media is not loaded, recorder will start when playback starts
  \param url destination. null or the same value as recording one to stop recording
  \param format forced format if unable to guess from url suffix
 */
    void (*record)(struct mdkPlayer*, const char* url, const char* format);

/*!
  \brief setLoopRange
  DEPRECATED! use setLoop+setRange instead
 */
    void (*setLoopRange)(struct mdkPlayer*, int count, int64_t a, int64_t b);
/*!
  \brief setLoop
  Set A-B loop repeat count.
  \param count repeat count. 0 to disable looping and stop when out of range(B)
 */
    void (*setLoop)(struct mdkPlayer*, int count);
/*
  \brief onLoop
  add/remove a callback which will be invoked right before a new A-B loop
  \param cb callback with current loop count elapsed
 */
    void (*onLoop)(struct mdkPlayer*, mdkLoopCallback cb, MDK_CallbackToken* token);
/*!
  \brief setRange
  Set A-B loop range, or playback range
  \param a loop position begin, in ms.
  \param b loop position end, in ms. -1, INT64_MAX or numeric_limit<int64_t>::max() indicates b is the end of media
 */
    void (*setRange)(struct mdkPlayer*, int64_t a, int64_t b);

/*
  RenderAPI
  RenderAPI provides platform/api dependent resources for video renderer and rendering context corresponding to vo_opaque. It's used by
  1. create internal render context via updateNativeSurface() using given api. MUST be called before any other functions have parameter vo_opaque and updateNativeSurface()!
    To use RenderAPI other than OpenGL, setRenderAPI() MUST be called before add/updateNativeSurface(), and vo_opaque MUST be the surface or nullptr.
    If vo_opaque is nullptr, i.e. the default, then all context will have the same RenderAPI type, and call setRenderAPI() once is enough.
    If vo_opaque is surface(not null), each surface can have it's own RenderAPI type.
    RenderAPI members will be initialized when a rendering context for surface is created, and keep valid in rendering functions like renderVideo()
  2. Set foreign context provided by user. setRenderAPI() and other functions with vo_opaque parameter can be called in any order
  3. render. renderVideo() will use the given api for vo_opaque

  If setRenderAPI() is not called by user, a default one (usually GLRenderAPI) is used, thus renderAPI() always not null.
  setRenderAPI() is not thread safe, so usually called before rendering starts, or native surface is set.
*/

/*!
  \brief setRenderAPI
  set render api for a vo, useful for non-opengl(no way to get current context)
  \param api
  To release gfx resources, set null api in rendering thread/context(required by vulkan)
 */
    void (*setRenderAPI)(struct mdkPlayer*, mdkRenderAPI* api, void* vo_opaque);
/*!
  \brief renderApi()
  get render api. For offscreen rendering, may only api type be valid in setRenderAPI(), and other members are filled internally, and used by user after renderVideo()
 */
    mdkRenderAPI* (*renderAPI)(struct mdkPlayer*, void* vo_opaque);

/*!
  \brief mapPoint
  map a point from one coordinates to another. a frame must be rendered. coordinates is normalized to [0, 1].
  \param x points to x coordinate of viewport or currently rendered video frame
  \param z not used
*/
    void (*mapPoint)(struct mdkPlayer*, enum MDK_MapDirection dir, float* x, float* y, float* z, void* vo_opaque);
/*!
  \brief onSync
  \param cb a callback invoked when about to render a frame. return expected current playback position(seconds), e.g. DBL_MAX(TimestampEOS) indicates render video frame ASAP.
  sync callback clock should handle pause, resume, seek and seek finish events
 */
    void (*onSync)(struct mdkPlayer*, mdkSyncCallback cb, int minInterval);

    void (*setVideoEffect)(struct mdkPlayer*, enum MDK_VideoEffect effect, const float* values, void* vo_opaque);
/*!
  \brief setActiveTracks
  \param type
  \param tracks set of active track number, from 0~N. Invalid track numbers will be ignored
 */
    void (*setActiveTracks)(struct mdkPlayer*, enum MDK_MediaType type, const int* tracks, size_t count);

    void (*setDecoders)(struct mdkPlayer*, enum MDK_MediaType type, const char* names[]);
/*!
  \brief setChannelVolume
  Set audio volume level
  \param value linear volume level, range is >=0. 1.0 is source volume
  \param channel channel number, int value of AudioFormat::Channel, -1 for all channels.
  The same as ms log2(SpeakerPosition), see https://docs.microsoft.com/en-us/windows-hardware/drivers/ddi/ksmedia/ns-ksmedia-ksaudio_channel_config#remarks
  setChannelVolume(value, -1) equals to setVolume(value)
 */
    void (*setChannelVolume)(struct mdkPlayer*, float value, int channel);
/*!
  \brief setFrameRate
  Set frame rate, frames per seconds
  \param value frame rate
  - 0 (default): use frame timestamp, or default frame rate 25.0fps if stream has no timestamp
  - <0: render ASAP.
  - >0: target frame rate
 */
    void (*setFrameRate)(struct mdkPlayer*, float value);
    void (*setPointMap)(struct mdkPlayer*, const float* videoRoi, const float* viewRoi, int count, void* vo_opaque);
    void (*setColorSpace)(struct mdkPlayer*, enum MDK_ColorSpace value, void* vo_opaque);
/*!
  \brief onMediaStatus
  Add or remove a callback to be invoked when MediaStatus is changed
  \param cb null to clear callbacks
 */
    void (*onMediaStatus)(struct mdkPlayer*, mdkMediaStatusCallback cb, MDK_CallbackToken* token);

/*!
  \brief size
  Struct size returned from runtime. Build time struct size may be different with runtime one, user MUST check
  1. size == 0: old runtime without extendable size support. members after size(and reserved) member are not available in runtime
  2. size > 0: new runtime with extendable size support. Before using members after size(and reserved), if offsetof(ThisType, Member) < size, it's safe to use the member
*/
    union {
        void* reserved2;
        int size;
    };

    void (*enqueueVideo)(struct mdkPlayer*, struct mdkVideoFrameAPI* frame, void* vo_opaque);
} mdkPlayerAPI;

MDK_API const mdkPlayerAPI* mdkPlayerAPI_new();
MDK_API void mdkPlayerAPI_delete(const struct mdkPlayerAPI**);
MDK_API void MDK_foreignGLContextDestroyed();

#ifdef __cplusplus
}
#endif
```

### `mdk/include/c/RenderAPI.h`

```cpp
/*
 * Copyright (c) 2019-2022 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include "global.h"

enum MDK_RenderAPI {
    MDK_RenderAPI_Invalid,
    MDK_RenderAPI_OpenGL = 1,
    MDK_RenderAPI_Vulkan = 2,
    MDK_RenderAPI_Metal = 3,
    MDK_RenderAPI_D3D11 = 4,
    MDK_RenderAPI_D3D12 = 5,
};

/*!
  \brief mdkRenderAPI
  use concrete types in user code, for example mdkD3D11RenderAPI
 */
typedef struct mdkRenderAPI mdkRenderAPI;

struct mdkGLRenderAPI {
    enum MDK_RenderAPI type;
/*** Render Context Resources. Foreign context (provided by user) only ***/
    int fbo; // if >=0, will draw in given fbo. no need to bind in user code
    int unused;
/*
  \brief getProcAddress
  optional. can be null and then standard gl libraries will be searched.
  if not null, it's used to load gl functions
  \param name gl function name
  \param opaque user data, e.g. gl context handle
*/
    void* (*getProcAddress)(const char* name, void* opaque);
    void* (*getCurrentNativeContext)(void* opaque);
/*!
  \brief opaque
  optional. getProcAddress user data, e.g. a gl context handle.
*/
    void* opaque;

/***
  Render Context Creation Options.
  as input, they are desired values to create an internal context(ignored if context is provided by user). as output, they are result values(if context is not provided by user)
***/
    bool debug; /* default false. NOT IMPLENETED */
    int8_t egl; /* default -1. -1: auto. 0: no, 1: yes */
/* if any one of opengl and opengles is 0, then another is treated as 1 */
    int8_t opengl; /* default -1. -1: auto. 0: no, 1: yes */
    int8_t opengles; /* default -1. -1: auto. 0: no, 1: yes */
    uint8_t profile; /* default 3. 0: no profile, 1: core profile, 2: compatibility profile */
    float version; /* default 0, ignored if < 2.0. requested version major.minor. result version may < requested version if not supported */
    int8_t reserved[32];
};

struct mdkMetalRenderAPI {
    enum MDK_RenderAPI type;
/*** Render Context Resources. Foreign context (provided by user) only ***/
// id<?> => void*: to be compatible with c++
    const void* device; // MUST set if metal is provided by user
    const void* cmdQueue; // optional. if not null, device can be null. currentQueue callback to share the same command buffer?
/* one of texture and currentRenderTarget MUST be set if metal is provided by user */
    const void* texture; // optional. id<MTLTexture>. if not null, device can be null. usually for offscreen rendering. render target for MTLRenderPassDescriptor if encoder is not provided by user. set once for offscreen rendering
    const void* opaque; // optional. callback opaque
    const void* (*currentRenderTarget)(const void* opaque); // optional. usually for on screen rendering. return id<MTLTexture>.
    const void* layer; // optional. CAMetalLayer only used for appling colorspace parameters for hdr/sdr videos.
    // no encoder because we need own render pass
    const void* reserved[1];

/***
  Render Context Creation Options.
  as input, they are desired values to create an internal context(ignored if context is provided by user). as output, they are result values(if context is not provided by user)
***/
    // device options: macOS only
    int device_index/*  = -1*/; // -1 will use system default device. callback with index+name?
};

/*!
  NOTE: include d3d11.h first to use D3D11RenderAPI
 */
#if defined(D3D11_SDK_VERSION)
struct mdkD3D11RenderAPI {
    enum MDK_RenderAPI type;
/*** Render Context Resources. Foreign context (provided by user) only ***/
/*
  context and rtv can be set by user if user can provide. then rendering becomes foreign context mode.
  if rtv is not null, no need to set context
  \sa Player.setRenderAPI()
 */
    ID3D11DeviceContext* context;
    // rtv or texture. usually user can provide a texture from gui easly, no d3d code to create a view
    ID3D11DeviceChild* rtv; // optional. the render target(view). ID3D11RenderTargetView or ID3D11Texture2D. can be null if context is not null. if not null, no need to set context
    void* reserved[2];

/***
  Render Context Creation Options.
  as input, they are desired values to create an internal context(ignored if context is provided by user). as output, they are result values(if context is not provided by user)
***/
    bool debug;
    int buffers; /* UWP must >= 2. */
    int adapter; /* adapter index */
    float feature_level; /* 0 is the highest */
    const char* vendor; /* since v0.17.0 */
};
#endif

/*!
  NOTE: include d3d12.h first to use D3D12RenderAPI
 */
#if defined(__d3d12_h__)// D3D12_SDK_VERSION: not defined in 19041
struct mdkD3D12RenderAPI {
    enum MDK_RenderAPI type;
/*** Render Context Resources. Foreign context (provided by user) only ***/
    ID3D12CommandQueue* cmdQueue; // optional. will create an internal queue if null.
    ID3D12Resource* rt; // optional. the render target
    D3D12_CPU_DESCRIPTOR_HANDLE rtvHandle; // optional
    void* reserved[2];

    const void* opaque; // optional. callback opaque
    ID3D12Resource* (*currentRenderTarget)(const void* opaque, UINT* index, UINT* count, D3D12_RESOURCE_STATES* state); // optional. usually for on screen rendering.
    void* reserved2[2];
/***
  Render Context Creation Options.
  as input, they are desired values to create an internal context(ignored if context is provided by user). as output, they are result values(if context is not provided by user)
***/
    bool debug;
    int buffers; /* must >= 2. */
    int adapter; /* adapter index */
    float feature_level; /* 0 is the highest */
    const char* vendor; /* gpu vendor name */
};
#endif

// always declare
struct mdkVulkanRenderAPI {
    enum MDK_RenderAPI type;

#if (VK_VERSION_1_0+0)
    VkInstance instance/* = VK_NULL_HANDLE*/; // OPTIONAL. shared instance. for internal created context but not foreign context, to load instance extensions
    VkPhysicalDevice phy_device/* = VK_NULL_HANDLE*/; // OPTIONAL to create internal context. MUST not null for foreign context. Must set if logical device is provided to create internal context.
    VkDevice device/* = VK_NULL_HANDLE*/; // Optional to create internal context as shared device. Required for foreign context.
    VkQueue graphics_queue/*/* = VK_NULL_HANDLE*/; // OPTIONAL. If null, will use gfx_queue_index. NOT required if vk is create internally
/*!
  \brief rt
  Used by offscreen rendering.
 */
    VkImage rt = VK_NULL_HANDLE; // VkImage? so can use qrhitexture.nativeTexture().object
    VkRenderPass render_pass = VK_NULL_HANDLE; // optional. If null(usually for offscreen rendering), final image layout is VK_IMAGE_LAYOUT_COLOR_ATTACHMENT_OPTIMAL
    void* opaque/* = nullptr*/;
/*!
  \brief renderTargetInfo
  Get render target image size
  \param format image format. MUST be set if framebuffer from beginFrame() is null
  \param finalLayout image final layout. No transition if undefined. Transition can also be in endFrame() callback if needed, then finalLayout here can be undefined.
  NOTE: assume transition is in the same graphics queue family.
  \return (render target)image count, e.g. swapchain image count.
 */
    int (*renderTargetInfo)(void* opaque, int* w, int* h, VkFormat* format, VkImageLayout* finalLayout); // return count
/*!
  \brief beginFrame
  Optional. Can be null(or not) for offscreen rendering if rt is not null.
  MUST be paired with endFrame()
  \param fb can be null, then will create internally. if not null, MUST set render_pass
  \param imgSem from present queue. can be null if fulfill any of
  // TODO: VkImage?
  1. present queue == gfx queue
  2. getCommandBuffer() is provided and submit in user code
  \return image index.
*/
    int (*beginFrame)(void* opaque, VkImageView* view/* = nullptr*/, VkFramebuffer* fb/*= nullptr*/, VkSemaphore* imgSem/* = nullptr*/)/* = nullptr*/;
    // int getNextImageView(); // not fbo, fbo is bound to render pass(can be dummy tmp). image view can also be used by compute pipeline. return index
/*!
  \brief currentCommandBuffer()
  if null, create pool internally(RTT)
 */
    VkCommandBuffer (*currentCommandBuffer)(void* opaque)/* = nullptr*/;
/*!
  \brief endFrame
  Optional. If null, frame is guaranteed to be rendered to image before executing the next command buffer in user code.
  If not null, user can wait for drawSem before using the image.
  MUST be paired with beginFrame()
  \param drawSem from gfx queue. can be null if fulfill any of
  1. present queue == gfx queue
  2. getCommandBuffer() is provided and submit in user code
  3. RTT offscreen rendering, i.e. rtv is set and beginFrame is null(user should wait for draw finish too)
 */
    void (*endFrame)(void* opaque, VkSemaphore* drawSem/* = nullptr*/)/*= nullptr*/; // can be null if offscreen. wait drawSem before present
#endif // (VK_VERSION_1_0+0)
    void* reserved[2];

/*
  Set by user and used internally even if device is provided by user
 */
    int graphics_family/*  = -1*/; // MUST if graphics and transfer queue family are different
    int compute_family/*  = -1*/; // optional. it's graphics_family if not set
    int transfer_family/*  = -1*/; // optional. it's graphics_family if not set
    int present_family/*  = -1*/; // optional. Must set if logical device is provided to create internal context
/***
  Render Context Creation Options.
  as input, they are desired values to create an internal context(ignored if context is provided by user). as output, they are result values(if context is not provided by user)
***/
    bool debug/* = false*/;
    uint8_t buffers/*  = 2*/; // 2 for double-buffering
    int device_index/*  = -1*/;
    uint32_t max_version/*  = 0*/; // requires vulkan 1.1
    int gfx_queue_index/*  = 0*/; // OPTIONAL
    int transfer_queue_index/*  = -1*/; // OPTIONAL. if not set, will use gfx queue
    int compute_queue_index/*  = -1*/; // OPTIONAL. if not set, will use gfx queue

    int depth/* = 8*/;
    //const char*
    uint8_t reserved_opt[32]; // color space etc.
};
```

### `mdk/include/c/VideoFrame.h`

```cpp
/*
 * Copyright (c) 2020-2021 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include "global.h"

#ifdef __cplusplus
extern "C" {
#endif
// fromPlanarYUV(w, h, pixdesc, data, strides)
// void* pixLayout()
// fromGL(id, internalfmt, w, h)
struct mdkVideoFrame;

enum MDK_PixelFormat {
    MDK_PixelFormat_Unknown = -1, // TODO: 0 in next major version
    MDK_PixelFormat_YUV420P,
    MDK_PixelFormat_NV12,
    MDK_PixelFormat_YUV422P,
    MDK_PixelFormat_YUV444P,
    MDK_PixelFormat_P010LE,
    MDK_PixelFormat_P016LE,
    MDK_PixelFormat_YUV420P10LE,
    MDK_PixelFormat_UYVY422,
    MDK_PixelFormat_RGB24,
    MDK_PixelFormat_RGBA,
    MDK_PixelFormat_RGBX,
    MDK_PixelFormat_BGRA,
    MDK_PixelFormat_BGRX,
    MDK_PixelFormat_RGB565LE,
    MDK_PixelFormat_RGB48LE,
    MDK_PixelFormat_RGB48 = MDK_PixelFormat_RGB48LE,  // name: "rgb48le"
    MDK_PixelFormat_GBRP,
    MDK_PixelFormat_GBRP10LE,
    MDK_PixelFormat_XYZ12LE,
    MDK_PixelFormat_YUVA420P,
    MDK_PixelFormat_BC1,
    MDK_PixelFormat_BC3,
    MDK_PixelFormat_RGBA64, // name: "rgba64le"
    MDK_PixelFormat_BGRA64, // name: "bgra64le"
    MDK_PixelFormat_RGBP16, // name: "rgbp16le"
    MDK_PixelFormat_RGBPF32, // name: "rgbpf32le"
    MDK_PixelFormat_BGRAF32, // name: "bgraf32le"
};

typedef struct mdkVideoFrameAPI {
    struct mdkVideoFrame* object;

    int (*planeCount)(struct mdkVideoFrame*);
    int (*width)(struct mdkVideoFrame*, int plane /*=-1*/);
    int (*height)(struct mdkVideoFrame*, int plane /*=-1*/);
    enum MDK_PixelFormat (*format)(struct mdkVideoFrame*);
    bool (*addBuffer)(struct mdkVideoFrame*, const uint8_t* data, int stride, void* buf, void (*bufDeleter)(void** pBuf), int plane);
    void (*setBuffers)(struct mdkVideoFrame*, uint8_t const** const data, int* strides/*in/out = nullptr*/);
    const uint8_t* (*bufferData)(struct mdkVideoFrame*, int plane);
    int (*bytesPerLine)(struct mdkVideoFrame*, int plane);
    void (*setTimestamp)(struct mdkVideoFrame*, double t);
    double (*timestamp)(struct mdkVideoFrame*);

    struct mdkVideoFrameAPI* (*to)(struct mdkVideoFrame*, enum MDK_PixelFormat format, int width/*= -1*/, int height/*= -1*/);
    bool (*save)(struct mdkVideoFrame*, const char* fileName, const char* format, float quality);

/* The followings are not implemented */
    struct mdkVideoFrameAPI* (*toHost)(struct mdkVideoFrame*);
    struct mdkVideoFrameAPI* (*fromGL)();
    struct mdkVideoFrameAPI* (*fromMetal)();
    struct mdkVideoFrameAPI* (*fromVk)();
    struct mdkVideoFrameAPI* (*fromD3D9)();
    struct mdkVideoFrameAPI* (*fromD3D11)();
    struct mdkVideoFrameAPI* (*fromD3D12)();
    void* reserved[13];
} mdkVideoFrameAPI;


MDK_API mdkVideoFrameAPI* mdkVideoFrameAPI_new(int width/*=0*/, int height/*=0*/, enum MDK_PixelFormat format/*=Unknown*/);
MDK_API void mdkVideoFrameAPI_delete(struct mdkVideoFrameAPI**);

#ifdef __cplusplus
}
#endif
```

### `mdk/include/c/global.h`

```cpp
/*
 * Copyright (c) 2019-2023 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include <stdbool.h> /* for swift/dart ffi gen. old swift may report error if -fcxx-module and can be workaround by #import <Metal/Metal.h> ifdef __OBJC__ */
#include <stdint.h>

#define MDK_VERSION_INT(major, minor, patch) \
    (((major&0xff)<<16) | ((minor&0xff)<<8) | (patch&0xff))
#define MDK_MAJOR 0
#define MDK_MINOR 23
#define MDK_MICRO 1
#define MDK_VERSION MDK_VERSION_INT(MDK_MAJOR, MDK_MINOR, MDK_MICRO)
#define MDK_VERSION_CHECK(a, b, c) (MDK_VERSION >= MDK_VERSION_INT(a, b, c))

#if defined(_WIN32)
#define MDK_EXPORT __declspec(dllexport)
#define MDK_IMPORT __declspec(dllimport)
#else
#define MDK_EXPORT __attribute__((visibility("default")))
#define MDK_IMPORT __attribute__((visibility("default")))
#endif

#ifdef BUILD_MDK_STATIC
# define MDK_API
#else
# if defined(BUILD_MDK_LIB)
#  define MDK_API MDK_EXPORT
# else
#  define MDK_API MDK_IMPORT
# endif
#endif

#ifdef __cplusplus
extern "C" {
#endif

/* TODO: global consts or macros for ffi */

/*!
  \brief CallbackToken
  A callback can be registered by (member)function onXXX(obj, callback, CallbackToken* token = nullptr). With the returned token we can remove the callback by onXXX(nullptr, token).
  Non-null callback(.opaque != null): register a callback and return a token(if not null).
  Null callback(.opaque == null) + non-null token: can remove the callback of given token.
  Null callback(.opaque == null) + null token: clear all callbacks.
 */
typedef uint64_t MDK_CallbackToken;

typedef enum MDK_MediaType {
    MDK_MediaType_Unknown = -1,
    MDK_MediaType_Video = 0,
    MDK_MediaType_Audio = 1,
    MDK_MediaType_Subtitle = 3,
} MDK_MediaType;

/*!
  \brief The MediaStatus enum
  Defines the io status of a media stream,
  Use flags_added/removed() to check the change, for example buffering after seek is Loaded|Prepared|Buffering, and changes to Loaded|Prepared|Buffered when seek completed
 */
typedef enum MDK_MediaStatus
{
    MDK_MediaStatus_NoMedia = 0, /* initial status, not invalid. // what if set an empty url and closed?*/
    MDK_MediaStatus_Unloaded = 1, /* unloaded // (TODO: or when a source(url) is set?)*/
    MDK_MediaStatus_Loading = 1<<1, /* opening and parsing the media */
    MDK_MediaStatus_Loaded = 1<<2, /* media is loaded and parsed. player is stopped state. mediaInfo() is available now */
    MDK_MediaStatus_Prepared = 1<<8, /* all tracks are buffered and ready to decode frames. tracks failed to open decoder are ignored*/
    MDK_MediaStatus_Stalled = 1<<3, /* insufficient buffering or other interruptions (timeout, user interrupt)*/
    MDK_MediaStatus_Buffering = 1<<4, /* when buffering starts */
    MDK_MediaStatus_Buffered = 1<<5, /* when buffering ends */
    MDK_MediaStatus_End = 1<<6, /* reached the end of the current media, no more data to read */
    MDK_MediaStatus_Seeking = 1<<7,
    MDK_MediaStatus_Invalid = 1<<31, /* failed to load media because of unsupport format or invalid media source */
} MDK_MediaStatus;

typedef struct mdkMediaStatusChangedCallback {
    bool (*cb)(MDK_MediaStatus, void* opaque);
    void* opaque;
} mdkMediaStatusChangedCallback;

typedef struct mdkMediaStatusCallback {
    bool (*cb)(MDK_MediaStatus oldValue, MDK_MediaStatus newValue, void* opaque);
    void* opaque;
} mdkMediaStatusCallback;
/*!
  \brief The State enum
  Current playback state. Set/Get by user
 */
typedef enum MDK_State {
    MDK_State_NotRunning,
    MDK_State_Stopped = MDK_State_NotRunning,
    MDK_State_Running,
    MDK_State_Playing = MDK_State_Running, /* start/resume to play*/
    MDK_State_Paused,
} MDK_State;
typedef MDK_State MDK_PlaybackState;

typedef struct mdkStateChangedCallback {
    void (*cb)(MDK_State, void* opaque);
    void* opaque;
} mdkStateChangedCallback;

typedef enum MDKSeekFlag {
    /* choose one of FromX */
    MDK_SeekFlag_From0       = 1,    /* relative to time 0*/
    MDK_SeekFlag_FromStart   = 1<<1, /* relative to media start position*/
    MDK_SeekFlag_FromNow     = 1<<2, /* relative to current position, the seek position can be negative*/
    MDK_SeekFlag_Frame       = 1<<6, /* Seek by frame. Seek target is frame count instead of milliseconds. Currently only FromNow|Frame is supported. BUG: avsync */
    /* combine the above values with one of the following*/
/* KeyFrame forward seek may fail(permission denied) near the end of media if there's no key frame after seek target position*/
    MDK_SeekFlag_KeyFrame    = 1<<8, /* fast key-frame seek, forward if Backward is not set. It's accurate seek without this flag. Accurate seek is slow and implies backward seek internally.*/
    MDK_SeekFlag_Fast        = MDK_SeekFlag_KeyFrame,
    MDK_SeekFlag_InCache     = 1 << 10, // try to seek in memory cache first. useful for speeding up network stream seeking.  Target position must be in range (position(), position() + Player.buffered()]

    MDK_SeekFlag_Backward    = 1 << 16,

    MDK_SeekFlag_Default     = MDK_SeekFlag_KeyFrame|MDK_SeekFlag_FromStart|MDK_SeekFlag_InCache
} MDK_SeekFlag;

/*!
  \brief VideoEffect
  per video renderer effect. set via Player.setVideoEffect(MDK_VideoEffect effect, const float*);
 */
enum MDK_VideoEffect {
    MDK_VideoEffect_Brightness,   /* [-1.0f, 1.0f], default 0 */
    MDK_VideoEffect_Contrast,     /* [-1.0f, 1.0f], default 0 */
    MDK_VideoEffect_Hue,          /* [-1.0f, 1.0f], default 0 */
    MDK_VideoEffect_Saturation,   /* [-1.0f, 1.0f], default 0 */
};

enum MDK_ColorSpace {
    MDK_ColorSpace_Unknown,
    MDK_ColorSpace_BT709,
    MDK_ColorSpace_BT2100_PQ,
    MDK_ColorSpace_scRGB,
    MDK_ColorSpace_ExtendedLinearDisplayP3,
    MDK_ColorSpace_ExtendedSRGB,
    MDK_ColorSpace_ExtendedLinearSRGB,
};

MDK_API int MDK_version();
/*!
  \brief javaVM
  deprecated. use MDK_setGlobalOptionPtr("jvm",..) or MDK_setGlobalOptionPtr("JavaVM",..) instead
  Set/Get current java vm
  \param vm null to get current vm
  \return vm before set
 */
MDK_API void* MDK_javaVM(void* vm);

typedef enum MDK_LogLevel {
    MDK_LogLevel_Off,
    MDK_LogLevel_Error,
    MDK_LogLevel_Warning,
    MDK_LogLevel_Info,
    MDK_LogLevel_Debug,
    MDK_LogLevel_All
} MDK_LogLevel;
MDK_API void MDK_setLogLevel(MDK_LogLevel value);
MDK_API MDK_LogLevel MDK_logLevel();
/* \brief setLogHandler
  If log handler is not set, i.e. setLogHandler() was not called, log is disabled.
  Set environment var `MDK_LOG=1` to enable log to stderr.
  If set to non-null handler, logs that >= logLevel() will be passed to the handler.
  If previous handler is set by user and not null, then call setLogHandler(nullptr) will print to stderr, and call setLogHandler(nullptr) again to silence the log
  To disable log, setLogHandler(nullptr) twice is better than simply setLogLevel(LogLevel::Off)
*/
typedef struct mdkLogHandler {
    void (*cb)(MDK_LogLevel, const char*, void* opaque);
    void* opaque;
} mdkLogHandler;
MDK_API void MDK_setLogHandler(mdkLogHandler);

/*
 keys for string/const char* value:
 - "avutil_lib", "avcodec_lib", "avformat_lib", "swresample_lib", "avfilter_lib": path to ffmpeg runtime libraries
 - "plugins_dir": plugins directory. MUST set before "plugins" if not in default dirs
 - "plugins": plugin filenames or paths in pattern "p1:p2:p3"
 - "MDK_KEY": license key for your product
 - "MDK_KEY_CODE_PAGE": license key code page used internally(windows only)
 - "ffmpeg.loglevel" or "ffmpeg.log": ffmpeg log level names, "trace", "debug", "verbose", "info", "warning", "error", "fatal", "panic", "quiet", or "${level}=${avclass}" to set AVClass log level(can be multiple times), e.g. "debug=http"
 - "ffmpeg.cpuflags": cpuflags for ffmpeg
 - "logLevel" or "log": can be "Off", "Error", "Warning", "Info", "Debug", "All". same as SetGlobalOption("logLevel", int(LogLevel))
 - "profiler.gpu": "0" or "1"
 - "R3DSDK_DIR": R3D dlls dir. default dir is working dir
*/
MDK_API void MDK_setGlobalOptionString(const char* key, const char* value);
/*
  keys:
  - "videoout.clear_on_stop": 0/1. clear renderer using background color if playback stops
  - "videoout.buffer_frames": N. max buffered frames to in the renderer
  - "videoout.hdr": auto send hdr metadata to display. overrides Player.set(ColorSpace)
  - "logLevel" or "log": raw int value of LogLevel
  - "profiler.gpu": true/false, 0/1
  - "demuxer.io": use io module for demuxer
        - 0: use demuxer's internal io
        - 1: default. prefer io module
        - 2: always use io module for all protocols
  - "demuxer.live_eos_timeout": read error if no data for the given milliseconds for a live stream. default is 5000

 */
MDK_API void MDK_setGlobalOptionInt32(const char* key, int value);

/*
  keys:
  - "sdr.white": sdr white level. usually it's 203, but some obs-studio default value is 300, so let user set the value
*/
MDK_API void MDK_setGlobalOptionFloat(const char* key, float value);
/*
  keys:
  - "jvm", "JavaVM": JavaVM*. android only
 */
MDK_API void MDK_setGlobalOptionPtr(const char* key, void* value);

MDK_API bool MDK_getGlobalOptionString(const char* key, const char** value);
MDK_API bool MDK_getGlobalOptionInt32(const char* key, int* value);
MDK_API bool MDK_getGlobalOptionPtr(const char* key, void** value);
/*
  events:
  {timestamp(ms), "render.video", "1st_frame"}: when the first frame is rendererd
  {error, "decoder.audio/video/subtitle", "open", stream}: decoder of a stream is open, or failed to open if error != 0. TODO: do not use "open"?
  {track, "decoder.video", "size", {width, height}}: video decoder output size change
  {0, "decoder.video", decoderName, stream}: video decoder output size change
  {track, "decoder.video", "size", {width, height}}: video decoder output size change. MediaInfo.video[track].codec.width/height also changes.
  {track, "video", "size", {width, height}}: video frame size change before rendering, e.g. change by a filter. MediaInfo.video[track].codec.width/height does not change.
  {progress 0~100, "reader.buffering"}: error is buffering progress
  {0/1, "thread.audio/video/subtitle", stream}: decoder thread is started (error = 1) and about to exit(error = 0)
  {error, "snapshot", saved_file if no error and error string if error < 0}
  {0, "cc"}: the 1st closed caption data is decoded. can be used in ui to show CC button.
*/
typedef struct mdkMediaEvent {
    int64_t error; /* result <0: error code(fourcc?). >=0: special value depending on event*/
    const char* category;
    const char* detail; /* if error, detail can be error string*/

    union {
        struct {
            int stream;
        } decoder;
        struct {
            int width;
            int height;
        } video;
    };
} mdkMediaEvent;

/*
bool MDK_SomeFunc(SomeStruct*, mdkStringMapEntry* entry)
entry: in/out, can not be null.
Input entry->priv is null:
The result entry points to the first entry containing the same key as entry->key, or the first entry if entry->key is null.
The result entry->priv is set to a new value by api.
Input entry->priv is not null(set by the api): the result entry points to the next entry.
return: true if entry is found, false if not.
*/
typedef struct mdkStringMapEntry {
    const char* key;    /* input: set by user to query .value field if priv is null
                           output: set by api if priv is not null (set by api) */
    const char* value;  /* output: set by api, or not touched if no such key */

    void* priv; /* input/output: set by api */
} mdkStringMapEntry;

/*
  \brief MDK_strdup
  Always use this if a duplicated string is needed. DO NOT call strdup() directly because may fail to free() it in mdk, for example
  if user code is built against msvc debug crt but mdk uses release crt, then free() in mdk will crash
 */
MDK_API char* MDK_strdup(const char* strSource);

#ifdef __cplusplus
}
#endif
```

### `mdk/include/c/module.h`

```cpp
/*
 * Copyright (c) 2020-2023 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include "MediaInfo.h"
#include "VideoFrame.h"
#include "RenderAPI.h"
#include "Player.h"
```

### `mdk/include/global.h`

```cpp
/*
 * Copyright (c) 2019-2023 WangBin <wbsecg1 at gmail.com>
 * This file is part of MDK
 * MDK SDK: https://github.com/wang-bin/mdk-sdk
 * Free for opensource softwares or non-commercial use.
 *
 * The above copyright notice and this permission notice shall be included
 * in all copies or substantial portions of the Software.
 */
#pragma once
#include "c/global.h"
#include <cassert>
#include <cfloat>
#include <functional>
#include <memory>
#include <string>
#include <type_traits>

#ifndef MDK_NS
#define MDK_NS mdk
#endif
# define MDK_NS_BEGIN namespace MDK_NS {
# define MDK_NS_END }
# define MDK_NS_PREPEND(X) ::MDK_NS::X

#define MDK_CALL(p, FN, ...) (assert(p->FN && "NOT IMPLEMENTED! Runtime version < build version?"), p->FN(p->object, ##__VA_ARGS__))
#define MDK_CALL2(p, FN, ...) (assert(p->size > 0 && offsetof(std::remove_reference<decltype(*p)>::type, FN) < (size_t)p->size && "NOT IMPLEMENTED! Upgrade your runtime library"), p->FN(p->object, ##__VA_ARGS__))

MDK_NS_BEGIN
// vs2013 no constexpr
static const double TimestampEOS = DBL_MAX;
static const double TimeScaleForInt = 1000.0; // ms
static const float IgnoreAspectRatio = 0; // stretch, ROI etc.
// aspect ratio > 0: keep the given aspect ratio and scale as large as possible inside target rectangle
static const float KeepAspectRatio = FLT_EPSILON; // expand using original aspect ratio
// aspect ratio < 0: keep the given aspect ratio and scale as small as possible outside renderer viewport
static const float KeepAspectRatioCrop = -FLT_EPSILON; // expand and crop using original aspect ratio

/*!
  \brief CallbackToken
  A callback can be registered by (member)function onXXX(callback, CallbackToken* token = nullptr). With the returned token we can remove the callback by onXXX(nullptr, token).
  Non-null callback: register a callback and return a token(if not null).
  Null callback + non-null token: can remove the callback of given token.
  Null callback + null token: clear all callbacks.
 */
using CallbackToken = uint64_t;
/*!
 * \brief is_flag
 * if enum E is of enum type, to enable flag(bit) operators, define
 * \code template<> struct is_flag<E> : std::true_type {}; \endcode
 */
template<typename T> struct is_flag; //
template<typename T>
using if_flag = std::enable_if<std::is_enum<T>::value && is_flag<T>::value>;
template<typename E, typename = if_flag<E>>
E operator~(E e1) { return E(~typename std::underlying_type<E>::type(e1));}
template<typename E, typename = if_flag<E>>
E operator|(E e1, E e2) { return E(typename std::underlying_type<E>::type(e1) | typename std::underlying_type<E>::type(e2));}
template<typename E, typename = if_flag<E>>
E operator^(E e1, E e2) { return E(typename std::underlying_type<E>::type(e1) ^ typename std::underlying_type<E>::type(e2));}
template<typename E, typename = if_flag<E>>
E operator&(E e1, E e2) { return E(typename std::underlying_type<E>::type(e1) & typename std::underlying_type<E>::type(e2));}
template<typename E, typename = if_flag<E>>
E& operator|=(E& e1, E e2) { return e1 = e1 | e2;}
template<typename E, typename = if_flag<E>>
E& operator^=(E& e1, E e2) { return e1 = e1 ^ e2;}
template<typename E, typename = if_flag<E>>
E& operator&=(E& e1, E e2) { return e1 = e1 & e2;}
// convenience functions to test whether a flag exists. REQUIRED by scoped enum
template<typename E>
bool test_flag(E e) { return typename std::underlying_type<E>::type(e);}
template<typename E1, typename E2>
bool test_flag(E1 e1, E2 e2) { return test_flag(e1 & e2);}
template<typename E>
bool flags_added(E oldFlags, E newFlags, E testFlags) { return test_flag(newFlags, testFlags) && !test_flag(oldFlags, testFlags);}
template<typename E>
bool flags_removed(E oldFlags, E newFlags, E testFlags) { return !test_flag(newFlags, testFlags) && test_flag(oldFlags, testFlags);}

enum class MediaType : int8_t {
    Unknown = -1,
    Video = 0,
    Audio = 1,
    Subtitle = 3,
};

/*!
  \brief The MediaStatus enum
  Defines the io status of a media stream,
  Use flags_added/removed() to check the change, for example buffering after seek is Loaded|Prepared|Buffering, and changes to Loaded|Prepared|Buffered when seek completed
 */
enum MediaStatus
{
    NoMedia = 0, // initial status, not invalid. // what if set an empty url and closed?
    Unloaded = 1, // unloaded // (TODO: or when a source(url) is set?)
    Loading = 1<<1, // opening and parsing the media
    Loaded = 1<<2, // media is loaded and parsed. player is stopped state. mediaInfo() is available now
    Prepared = 1<<8, // all tracks are buffered and ready to decode frames. tracks failed to open decoder are ignored
    Stalled = 1<<3, // insufficient buffering or other interruptions (timeout, user interrupt)
    Buffering = 1<<4, // when buffering starts
    Buffered = 1<<5, // when buffering ends
    End = 1<<6, // reached the end of the current media, no more data to read
    Seeking = 1<<7,
    Invalid = 1<<31, // failed to load media because of unsupport format or invalid media source
};
template<> struct is_flag<MediaStatus> : std::true_type {};
// MediaStatusCallback

/*!
  \brief The State enum
  Current playback state. Set/Get by user
 */
enum class State : int8_t {
    NotRunning,
    Stopped = NotRunning,
    Running,
    Playing = Running, /// start/resume to play
    Paused,
};
typedef State PlaybackState;

enum class SeekFlag {
    /// choose one of FromX
    From0       = 1,    /// relative to time 0. TODO: remove from api
    FromStart   = 1<<1, /// relative to media start position
    FromNow     = 1<<2, /// relative to current position, the seek position can be negative
    Frame       = 1<<6, /* Seek by frame. Seek target is frame count instead of milliseconds. Currently only FromNow|Frame is supported. BUG: avsync */
    /// combine the above values with one of the following
/* KeyFrame forward seek may fail(permission denied) near the end of media if there's no key frame after seek target position*/
    KeyFrame    = 1<<8, /* fast key-frame seek, forward if Backward is not set. It's accurate seek without this flag. Accurate seek is slow and implies backward seek internally.*/
    Fast        = KeyFrame,
    InCache     = 1 << 10, // try to seek in memory cache first. useful for speeding up network stream seeking.  Target position must be in range (position(), position() + Player.buffered()]

    Backward    = 1 << 16, // for KeyFrame seek only. Accurate seek implies Backward

    Default     = KeyFrame|FromStart|InCache
};
template<> struct is_flag<SeekFlag> : std::true_type {};

static inline int version() {
    return MDK_version();
}

enum LogLevel {
    Off,
    Error,
    Warning,
    Info,
    Debug,
    All
};
#if !MDK_VERSION_CHECK(1, 0, 0)
#if (__cpp_attributes+0)
[[deprecated("use SetGlobalOption(\"log\", LogLevel/*or name*/) instead")]]
#endif
static inline void setLogLevel(LogLevel value) {
    MDK_setLogLevel(MDK_LogLevel(value));
}
#endif

/* \brief setLogHandler
  If log handler is not set, i.e. setLogHandler() was not called, log is disabled.
  Set environment var `MDK_LOG=1` to enable log to stderr.
  If set to non-null handler, logs that >= logLevel() will be passed to the handler.
  If previous handler is set by user and not null, then call setLogHandler(nullptr) will print to stderr, and call setLogHandler(nullptr) again to silence the log
  To disable log, setLogHandler(nullptr) twice is better than simply setLogLevel(LogLevel::Off)
*/
static inline void setLogHandler(std::function<void(LogLevel, const char*)> cb) {
    static std::function<void(LogLevel, const char*)> scb;
    scb = cb;
    mdkLogHandler h;
    h.cb = [](MDK_LogLevel level, const char* msg, void* opaque){
        auto f = (std::function<void(LogLevel, const char*)>*)opaque;
        (*f)(LogLevel(level), msg);
    };
    h.opaque = scb ? (void*)&scb : nullptr;
    MDK_setLogHandler(h);
    static struct LogReset {
        ~LogReset() {
            mdkLogHandler stdh{};
            MDK_setLogHandler(stdh); // reset log handler std to ensure no log go to scb after scb destroyed
        }
    } reset;
}

/*
 keys:
 - "avutil_lib", "avcodec_lib", "avformat_lib", "swresample_lib", "avfilter_lib": path to ffmpeg runtime libraries
 - "plugins_dir": plugins directory. MUST set before "plugins" if not in default dirs
 - "plugins": plugin filenames or paths in pattern "p1:p2:p3"
 - "MDK_KEY": license key for your product
 - "MDK_KEY_CODE_PAGE": license key code page used internally(windows only)
 - "ffmpeg.loglevel" or "ffmpeg.log": ffmpeg log level names, "trace", "debug", "verbose", "info", "warning", "error", "fatal", "panic", "quiet", or "${level}=${avclass}" to set AVClass log level(can be multiple times), e.g. "debug=http"
 - "ffmpeg.cpuflags": cpuflags for ffmpeg
 - "logLevel" or "log": can be "Off", "Error", "Warning", "Info", "Debug", "All". same as SetGlobalOption("logLevel", int(LogLevel))
 - "profiler.gpu": "0" or "1"
 - "R3DSDK_DIR": R3D dlls dir. default dir is working dir
*/
static inline void SetGlobalOption(const char* key, const char* value)
{
    MDK_setGlobalOptionString(key, value);
}

/*!
  \return false if no such key
  keys:
  - "ffmpeg.configuration": ffmpeg major version. return false if no ffmpeg api was invoked internally.
 */
static inline bool GetGlobalOption(const char* key, const char** value)
{
    return MDK_getGlobalOptionString(key, value);
}

/*
  keys:
  - "videoout.clear_on_stop": 0/1. clear renderer using background color if playback stops
  - "videoout.buffer_frames": N. max buffered frames to in the renderer
  - "videoout.hdr": auto send hdr metadata to display. overrides Player.set(ColorSpace)
  - "logLevel" or "log": raw int value of LogLevel
  - "profiler.gpu": true/false, 0/1
  - "demuxer.io": use io module for demuxer
        - 0: use demuxer's internal io
        - 1: default. prefer io module
        - 2: always use io module for all protocols
  - "demuxer.live_eos_timeout": read error if no data for the given milliseconds for a live stream. default is 5000

 */
static inline void SetGlobalOption(const char* key, int value)
{
    MDK_setGlobalOptionInt32(key, value);
}

/*
  keys:
  - "sdr.white": sdr white level. usually it's 203, but some obs-studio default value is 300, so let user set the value
*/
static inline void SetGlobalOption(const char* key, float value)
{
    MDK_setGlobalOptionFloat(key, value);
}

/*!
  \return false if no such key
  keys:
  - "ffmpeg.version": ffmpeg major version. return false if no ffmpeg api was invoked internally.
 */
static inline bool GetGlobalOption(const char* key, int* value)
{
    return MDK_getGlobalOptionInt32(key, value);
}

// key: "logLevel"
static inline void SetGlobalOption(const char* key, LogLevel value)
{
    MDK_setGlobalOptionInt32(key, std::underlying_type<LogLevel>::type(value));
}
/*
  keys:
  - "jvm", "JavaVM": JavaVM*. android only. Required if not loaded by System.loadLibrary()
  - "X11Display": Display*
  - "DRMDevice": drm device path, for vaapi
  - "DRMFd": drm fd, for vaapi
 */
static inline void SetGlobalOption(const char* key, void* value)
{
    MDK_setGlobalOptionPtr(key, value);
}

static inline bool GetGlobalOption(const char* key, void** value)
{
    return MDK_getGlobalOptionPtr(key, value);
}

/*!
  \brief javaVM
  Set/Get current java vm
  \param vm null to get current vm
  \return vm before set
 */
#if !MDK_VERSION_CHECK(1, 0, 0)
#if (__cpp_attributes+0)
[[deprecated("use SetGlobalOption(\"jvm\", ptr) instead")]]
#endif
static inline void javaVM(void* vm) {
    return SetGlobalOption("jvm", vm);
}
#endif
/*
  events:
  {timestamp(ms), "render.video", "1st_frame"}: when the first frame is rendererd
  {error, "decoder.audio/video/subtitle", "open", stream}: decoder of a stream is open, or failed to open if error != 0. TODO: do not use "open"?
  {0, "decoder.video", decoderName, stream}: video decoder output size change
  {track, "decoder.video", "size", {width, height}}: video decoder output size change. MediaInfo.video[track].codec.width/height also changes.
  {track, "video", "size", {width, height}}: video frame size change before rendering, e.g. change by a filter. MediaInfo.video[track].codec.width/height does not change.
  {progress 0~100, "reader.buffering"}: error is buffering progress
  {0/1, "thread.audio/video/subtitle", stream}: decoder thread is started (error = 1) and about to exit(error = 0)
  {error, "snapshot", saved_file if no error and error string if error < 0}
  {0, "cc"}: the 1st closed caption data is decoded. can be used in ui to show CC button.
*/
class MediaEvent {
public:
    int64_t error = 0; // result <0: error code(fourcc?). >=0: special value depending on event
    std::string category;
    std::string detail; // if error, detail can be error string

    union {
        struct {
            int stream;
        } decoder;
        struct {
            int width;
            int height;
        } video;
    } /* TODO: data */;
};

/*!
  \brief VideoEffect
  per video renderer effect. set via Player.set(VideoEffect effect, const float&);
 */
enum class VideoEffect {
    Brightness,   /* [-1.0f, 1.0f], default 0 */
    Contrast,     /* [-1.0f, 1.0f], default 0 */
    Hue,          /* [-1.0f, 1.0f], default 0 */
    Saturation,   /* [-1.0f, 1.0f], default 0 */
};

enum ColorSpace {
    ColorSpaceUnknown,
    ColorSpaceBT709,
    ColorSpaceBT2100_PQ,
// scRGB, linear sRGB in extended component range. Scene-referred white level, D65 is 80nit. Used on windows
    ColorSpaceSCRGB,
    ColorSpaceExtendedLinearDisplayP3,
// sRGB in extended component range, sRGB transfer function. Available for macOS displays
    ColorSpaceExtendedSRGB,
// linear sRGB in extended component range. Display-referred white level
    ColorSpaceExtendedLinearSRGB,
};

MDK_NS_END
```

### `mdk/mdkinclude.h`

```cpp
﻿#ifndef MDKINCLUDE_H
#define MDKINCLUDE_H

//引入头文件
#include "qglobal.h"
#include "qdebug.h"
#include "qstringlist.h"

#if (QT_VERSION >= QT_VERSION_CHECK(5,5,0))
#include <QOpenGLWidget>
#define OpenGLWidget QOpenGLWidget
#else
#include <QGLWidget>
#define OpenGLWidget QGLWidget
#endif

#include "Player.h"
//using namespace mdk;

#ifdef Q_CC_MSVC
#pragma execution_character_set("utf-8")
#endif

#include "qdatetime.h"
#ifndef TIMEMS
#define TIMEMS qPrintable(QTime::currentTime().toString("HH:mm:ss zzz"))
#endif

#endif // MDKINCLUDE_H
```

### `mdk/mdkwidget.cpp`

```cpp
﻿#include "mdkwidget.h"
#include "qdebug.h"

MdkWidget::MdkWidget(QWidget *parent): OpenGLWidget(parent)
{
    player = new mdk::Player;
    player->setRenderCallback([this](void *) {
        QMetaObject::invokeMethod(this, "update", Qt::QueuedConnection);
    });
}

MdkWidget::~MdkWidget()
{
    makeCurrent();
    mdk::Player::foreignGLContextDestroyed();
    doneCurrent();
}

void MdkWidget::setUrl(const QString &url)
{
    player->setMedia(url.toUtf8().constData());
}

void MdkWidget::play()
{
    player->set(mdk::State::Playing);
}

void MdkWidget::pause()
{
    player->set(mdk::State::Paused);
}

void MdkWidget::stop()
{
    player->set(mdk::State::Stopped);
}

void MdkWidget::initializeGL()
{

}

void MdkWidget::resizeGL(int w, int h)
{
    float ratio = 1;
#if (QT_VERSION >= QT_VERSION_CHECK(5,6,0))
    ratio = devicePixelRatioF();
#endif
    player->setVideoSurfaceSize(w * ratio, h * ratio, this);
}

void MdkWidget::paintGL()
{
    player->renderVideo(this);
}
```

### `mdk/mdkwidget.h`

```cpp
﻿#ifndef MDKWIDGET_H
#define MDKWIDGET_H

#include "mdkinclude.h"
#include "qpainter.h"

class MdkWidget : public OpenGLWidget
{
    Q_OBJECT
public:
    explicit MdkWidget(QWidget *parent = 0);
    ~MdkWidget();

    void setUrl(const QString& url);
    void play();
    void pause();
    void stop();

protected:
    void initializeGL();
    void resizeGL(int w, int h);
    void paintGL();

private:
    mdk::Player *player;
};

#endif // MDKWIDGET_H
```

### `./widget.ui`

```xml
<?xml version="1.0" encoding="UTF-8"?>
<ui version="4.0">
 <class>Widget</class>
 <widget class="QWidget" name="Widget">
  <property name="geometry">
   <rect>
    <x>0</x>
    <y>0</y>
    <width>800</width>
    <height>600</height>
   </rect>
  </property>
  <property name="windowTitle">
   <string>Widget</string>
  </property>
  <layout class="QVBoxLayout" name="verticalLayout">
   <item>
    <widget class="MdkWidget" name="playWidget" native="true">
     <property name="sizePolicy">
      <sizepolicy hsizetype="Preferred" vsizetype="Expanding">
       <horstretch>0</horstretch>
       <verstretch>0</verstretch>
      </sizepolicy>
     </property>
    </widget>
   </item>
   <item>
    <layout class="QHBoxLayout" name="horizontalLayout">
     <item>
      <widget class="QComboBox" name="cboxUrl">
       <property name="sizePolicy">
        <sizepolicy hsizetype="Expanding" vsizetype="Fixed">
         <horstretch>0</horstretch>
         <verstretch>0</verstretch>
        </sizepolicy>
       </property>
       <property name="editable">
        <bool>true</bool>
       </property>
      </widget>
     </item>
     <item>
      <widget class="QPushButton" name="btnSelect">
       <property name="text">
        <string>选择</string>
       </property>
      </widget>
     </item>
     <item>
      <widget class="QPushButton" name="btnOpen">
       <property name="text">
        <string>打开</string>
       </property>
      </widget>
     </item>
    </layout>
   </item>
  </layout>
 </widget>
 <customwidgets>
  <customwidget>
   <class>MdkWidget</class>
   <extends>QWidget</extends>
   <header>mdkwidget.h</header>
   <container>1</container>
  </customwidget>
 </customwidgets>
 <resources/>
 <connections/>
</ui>
```

### `mdk/mdk.pri`

```makefile
DEFINES += mdkx
#下面表示Qt4.7及以下版本移除标记
lessThan(QT_MAJOR_VERSION, 5) {
lessThan(QT_MINOR_VERSION, 8) {
DEFINES -= mdkx
}}

#mdk组件最低支持msvc2013(msvc2013=12/msvc2015=14)
msvc {
lessThan(MSVC_VER, 12) {
DEFINES -= mdkx
}}

contains(DEFINES, mdkx) {
QT *= opengl
#Qt6单独提取出来了openglwidgets模块
greaterThan(QT_MAJOR_VERSION, 5) {
QT *= openglwidgets
}
win32 {
LIBS *= -lopengl32 -lGLU32
}

#需要启用c11标准(5.5及以下版本需要手动开启/6.0以后默认都开启了)
lessThan(QT_MAJOR_VERSION, 5) {
QMAKE_CXXFLAGS += -std=gnu++11
}
lessThan(QT_MAJOR_VERSION, 6) {
lessThan(QT_MINOR_VERSION, 6) {
QMAKE_CXXFLAGS += -std=gnu++11 # c++0x / c++11 / gnu++11
}}

contains(QT_ARCH, x86_64) {
path_lib = libwin64
} else {
path_lib = libwin32
}

#包含头文件
INCLUDEPATH += $$PWD/include
#链接库文件
LIBS += -L$$PWD/$$path_lib/ -lmdk

HEADERS += $$PWD/mdkinclude.h
HEADERS += $$PWD/mdkwidget.h
SOURCES += $$PWD/mdkwidget.cpp
}
```
