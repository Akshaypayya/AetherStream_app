# AetherStream (`aether_stream`) — Android Release APK

<div align="center">
  <img src="app_icon.png" width="128" height="128" alt="AetherStream Icon" />
  <br/>
  <h3>OTT Video Streaming & Native Media3 / AVPlayer Engine</h3>
  <p>
    <a href="https://github.com/Akshaypayya/AetherStream_app/raw/main/app-release.apk">
      <img src="https://img.shields.io/badge/Download-Release%20APK%20(76.03%20MB)-00F5D4?style=for-the-badge&logo=android&logoColor=black" alt="Download APK" />
    </a>
    <a href="https://github.com/Akshaypayya/AetherStream">
      <img src="https://img.shields.io/badge/Source%20Code-GitHub%20Repo-7928CA?style=for-the-badge&logo=github&logoColor=white" alt="Source Code" />
    </a>
  </p>
  <p>
    <img src="https://img.shields.io/badge/Version-v1.0.0--release-brightgreen?style=flat-square" alt="Version" />
    <img src="https://img.shields.io/badge/Platform-Android%20(Media3%20ExoPlayer)-blue?style=flat-square" alt="Platform" />
    <img src="https://img.shields.io/badge/Engine-Media3%20ExoPlayer%201.4.1-orange?style=flat-square" alt="Media3" />
    <img src="https://img.shields.io/badge/State-BLoC%20%2F%20Cubit-magenta?style=flat-square" alt="BLoC" />
    <img src="https://img.shields.io/badge/Architecture-ARM64%20%7C%20ARMv7%20%7C%20x86__64-purple?style=flat-square" alt="Architecture" />
  </p>
</div>

---

## 📥 Direct APK Download

| File | Size | Direct Link | SHA-256 Checksum |
| :--- | :--- | :--- | :--- |
| **`app-release.apk`** | **76.03 MB** | [⬇️ **Download Latest Release APK**](https://github.com/Akshaypayya/AetherStream_app/raw/main/app-release.apk) | `7ECE09B3381340D1BDDDD1AF27BFCDC033D95C9ECFFF635A56ACC21B9808B1A6` |

---

## 📱 Quick Installation Guide

### Method 1: Install Directly on Android Phone
1. Open this page on your Android phone and tap [**Download Release APK**](https://github.com/Akshaypayya/AetherStream_app/raw/main/app-release.apk).
2. Once the download finishes, open the notification or file manager.
3. If prompted with *"For your security, your phone is not allowed to install unknown apps from this source"*, tap **Settings** $\rightarrow$ toggle **Allow from this source** ON.
4. Tap **Install** $\rightarrow$ **Open** to launch **AetherStream**.

### Method 2: Install via ADB (USB Debugging)
If your phone or emulator is connected via USB with Developer Options / USB Debugging enabled:
```bash
adb install -r app-release.apk
```
To launch the app immediately via ADB:
```bash
adb shell am start -n io.aetherstream.app/.MainActivity
```

---

## 🎬 What is AetherStream?

**AetherStream** is an OTT video streaming application engineered with **Flutter**, **BLoC**, and **feature-first Clean Architecture**, backed by a custom native `PlatformView` + `MethodChannel`/`EventChannel` bridge to **AndroidX Media3 (`ExoPlayer`)** on Android (and `AVPlayer` on iOS).

### 🌟 Core Engineering Features in this APK

#### 1. Native Media3 ExoPlayer Bridge & In-Session ABR Capping
* Custom Kotlin `PlatformView` (`AetherNativePlayerView`) wrapping `androidx.media3:media3-exoplayer` (`HLS` & `MPEG-DASH`).
* **In-Session ABR Switching**: Seamlessly toggle between **Auto ABR** and manual resolution/bitrate caps (`DefaultTrackSelector`) without tearing down or re-initializing the player.

#### 2. Native Multi-Audio & Subtitle Track Discovery
* Parses available audio and subtitle/closed-caption tracks directly from live HLS (`.m3u8`) and MPEG-DASH (`.mpd`) manifests (`Tracks` / `TrackSelectionOverride`) and switches them natively in-player.

#### 3. Real-Time QoE & QoS Telemetry Engine
* **Native Telemetry Bridge**: Streams measured Time-to-First-Frame (`TTFF`), downstream bitrate, estimated bandwidth (`DefaultBandwidthMeter`), buffer health, and dropped frames from Media3 `AnalyticsListener` into `TelemetryBloc`.
* **In-Player "Stats for Nerds" HUD & QoE Studio**: Displays live metrics alongside a `CustomPainter` Bitrate vs. Bandwidth chart and network condition throttle simulator.

#### 4. Resilient Offline Fallback & Persistent Mini-Player
* Automatic fallback cascade from primary stream $\rightarrow$ secondary mirror $\rightarrow$ local cached/bundled asset.
* Floating dockable mini-player and lifecycle-aware background playback (`WidgetsBindingObserver`).

#### 5. Prime Video-Style "X-Ray" & AWS Architecture Explorer
* Timestamp-synced "X-Ray" scene intelligence cards.
* Interactive 5-stage **AWS Media Services Architecture Explorer** (`MediaLive`, `MediaConvert`, `MediaPackage v2`, `MediaTailor`, `CloudFront`) with live HLS/DASH manifest previews (client-side architecture blueprint).

---

## 🛠️ Technical Specifications

| Parameter | Specification |
| :--- | :--- |
| **App Name** | AetherStream |
| **Package ID** | `io.aetherstream.app` |
| **Build Target** | Android Release APK |
| **File Size** | 76.03 MB (79,726,205 bytes) |
| **Video Engine** | AndroidX Media3 ExoPlayer 1.4.1 (`PlatformView` + Channels) |
| **CPU ABIs Supported** | `arm64-v8a`, `armeabi-v7a`, `x86_64` |
| **State Management** | BLoC & Cubit (`flutter_bloc`) |
| **Source Code** | [https://github.com/Akshaypayya/AetherStream](https://github.com/Akshaypayya/AetherStream) |

---

## 👨‍💻 Developer & Source Repository

* **Full Source Code**: [github.com/Akshaypayya/AetherStream](https://github.com/Akshaypayya/AetherStream)
* **Author**: Akshay Chandran
* **Portfolio**: [Akshaypayya on GitHub](https://github.com/Akshaypayya)
