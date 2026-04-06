# MPV 播放器集成指南

## 概述

本指南将帮助你在 DanDanPlayForAndroid 项目中集成 MPV 播放器。我们将基于 [mpv-android](https://github.com/mpv-android/mpv-android) 项目的实现来完成集成。

## 前置条件

1. **Linux 或 macOS 环境**：编译 mpv 原生代码需要 Linux 或 macOS（Windows 需要 WSL）
2. **Android SDK 和 NDK**：需要 r29 或更新版本
3. **Git**：用于克隆项目

## 第一步：获取 mpv-android 项目的核心文件

### 1.1 克隆 mpv-android 项目

```bash
git clone https://github.com/mpv-android/mpv-android.git
cd mpv-android
```

### 1.2 编译原生库（.so 文件）

进入 `buildscripts` 目录并执行：

```bash
cd buildscripts
./download.sh  # 下载依赖
./buildall.sh --arch arm64 mpv    # 编译 64 位 ARM
./buildall.sh --arch armeabi-v7a mpv  # 编译 32 位 ARM（可选）
./buildall.sh --arch x86_64 mpv    # 编译 64 位 x86（可选）
```

编译完成后，你会在 `buildscripts/build/app/src/main/jniLibs/` 目录下找到编译好的 .so 文件。

## 第二步：复制必要的文件到 DanDanPlayForAndroid 项目

### 2.1 复制原生库文件

将编译好的 .so 文件复制到 `player_component/libs/` 目录：

```
player_component/libs/
├── arm64-v8a/
│   ├── libmpv.so
│   ├── libavcodec.so
│   ├── libavformat.so
│   ├── libavutil.so
│   ├── libswresample.so
│   ├── libswscale.so
│   ├── libass.so
│   └── ... (其他依赖库)
├── armeabi-v7a/
│   └── (同上)
└── x86_64/
    └── (同上)
```

### 2.2 复制 Java/Kotlin 绑定文件

从 mpv-android 项目复制以下文件到 `player_component/src/main/java/com/xyoye/player/kernel/impl/mpv/` 目录：

1. `MPVLib.kt` - MPV 核心库
2. `BaseMPVView.kt` - 基础视图
3. `MPVView.kt` - MPV 视图（可选）
4. `Utils.kt` - 工具类（如果需要）

## 第三步：实现 MPV 播放器集成

我们已经为你创建了基础的播放器框架。现在需要完善 `MpvVideoPlayer.kt`，使其与实际的 MPV 库一起工作。

### 3.1 更新 player_component/build.gradle.kts

添加以下配置：

```kotlin
android {
    sourceSets {
        getByName("main") {
            jniLibs.srcDir("libs")
        }
    }
}
```

### 3.2 完善 MpvVideoPlayer.kt

在 `MpvVideoPlayer.kt` 中，你需要：

1. 导入 MPV 相关类
2. 初始化 MPV 实例
3. 实现所有播放器接口
4. 设置事件监听器

## MPV 播放器功能特性

### 硬件解码支持

MPV 支持多种硬件解码方式：

```kotlin
// 在 setOptions() 方法中添加
mpv.setOption("hwdec", "auto-safe")  // 自动选择安全的硬件解码方式
mpv.setOption("vo", "gpu")          // 使用 GPU 渲染
mpv.setOption("gpu-api", "vulkan")  // 使用 Vulkan API（可选）
```

### 倍速播放

```kotlin
override fun setSpeed(speed: Float) {
    mpv.setProperty("speed", speed)
    currentSpeed = speed
}
```

### ASS 字幕支持

MPV 通过 libass 原生支持 ASS 字幕：

```kotlin
mpv.setOption("sub-ass", "yes")  // 启用 ASS 字幕
mpv.setOption("sub-auto", "fuzzy")  // 自动加载字幕
```

### 多音轨/字幕轨切换

```kotlin
// 获取音轨列表
val audioTracks = mpv.getProperty("track-list")

// 选择音轨
mpv.setProperty("aid", trackId)

// 选择字幕
mpv.setProperty("sid", trackId)
```

## 详细集成步骤总结

1. **编译 mpv-android 原生库**（需要 Linux/macOS）
2. **复制 .so 文件**到 `player_component/libs/`
3. **复制 Java/Kotlin 绑定文件**到项目中
4. **完善 MpvVideoPlayer.kt**的实际实现
5. **测试播放器功能**

## 常见问题

### Q: 我没有 Linux/macOS 环境怎么办？

A: 你可以：
1. 使用 WSL2 (Windows Subsystem for Linux)
2. 寻找预编译的 mpv-android 库
3. 请有 Linux/macOS 的朋友帮忙编译

### Q: 编译失败怎么办？

A: 请参考 mpv-android 项目的 [buildscripts/README.md](https://github.com/mpv-android/mpv-android/blob/master/buildscripts/README.md) 文件。

### Q: 如何添加更多 MPV 配置选项？

A: 可以参考 [MPV 官方手册](https://mpv.io/manual/master/) 来了解所有可用的配置选项。

## 下一步

完成以上步骤后，你就可以在播放器设置中选择 "MPV Player" 来使用 MPV 播放器了！

MPV 播放器将提供：
- ✅ 硬件解码支持
- ✅ 任意倍速播放 (0.01x - 100x)
- ✅ 完美的 ASS 字幕渲染
- ✅ 多音轨/字幕轨切换
- ✅ 高质量视频渲染
- ✅ 广泛的视频格式兼容性
