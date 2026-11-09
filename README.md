# AetherStream (`aether_stream`) — Android Release APK

<div align="center">
  <img src="app_icon.png" width="128" height="128" alt="AetherStream Icon" />
  <br/>
  <h3>Cinema in the Cloud • Enterprise OTT & Video Streaming Platform</h3>
  <p>
    <a href="https://github.com/Akshaypayya/AetherStream_app/raw/main/app-release.apk">
      <img src="https://img.shields.io/badge/Download-Release%20APK%20(75.5%20MB)-00F5D4?style=for-the-badge&logo=android&logoColor=black" alt="Download APK" />
    </a>
    <a href="https://github.com/Akshaypayya/AetherStream">
      <img src="https://img.shields.io/badge/Source%20Code-GitHub%20Repo-7928CA?style=for-the-badge&logo=github&logoColor=white" alt="Source Code" />
    </a>
  </p>
  <p>
    <img src="https://img.shields.io/badge/Version-v1.0.0--release-brightgreen?style=flat-square" alt="Version" />
    <img src="https://img.shields.io/badge/Platform-Android%207.0%2B%20(API%2024%2B)-blue?style=flat-square" alt="Platform" />
    <img src="https://img.shields.io/badge/Decoders-ExoPlayer%2060%20FPS%20HW-orange?style=flat-square" alt="ExoPlayer" />
    <img src="https://img.shields.io/badge/State-BLoC%20%2F%20Cubit-magenta?style=flat-square" alt="BLoC" />
    <img src="https://img.shields.io/badge/Architecture-ARM64%20%7C%20ARMv7%20%7C%20x86__64-purple?style=flat-square" alt="Architecture" />
  </p>
</div>

---

## 📥 Direct APK Download

| File | Size | Direct Link | SHA-256 Checksum |
| :--- | :--- | :--- | :--- |
| **`app-release.apk`** | **75.52 MB** | [⬇️ **Download Latest Release APK**](https://github.com/Akshaypayya/AetherStream_app/raw/main/app-release.apk) | `9D71D0D00B2C3DD2FC9A232DBA42359FC06970BC9645BB52A3A82485417F236F` |

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
adb shell am start -n com.example.ott_app/.MainActivity
```

---

## 🎬 What is AetherStream?

**AetherStream** is a production-grade Flutter OTT/streaming application engineered to demonstrate deep **Video & Media Domain Expertise**, **AWS Elemental Cloud Streaming Services**, **Real-Time QoE/QoS Video Telemetry**, **Clean Architecture with BLoC**, and **Adaptive UI Design**.

### 🌟 Core Engineering Features in this APK:

#### 1. Real-World Adaptive Bitrate (ABR) Streaming
* Seamless multi-rendition switching between **Auto ABR** and manual rungs:
  * `2160p 4K HDR10` (HEVC Main10, 15.8 Mbps)
  * `1080p Full HD` (AVC High, 6.4 Mbps)
  * `720p HD` (AVC, 3.2 Mbps)
  * `480p Data Saver` (AVC, 1.25 Mbps)
* **60 FPS Hardware Decoding**: Accelerated via Android ExoPlayer with zero decoder pipeline stalls or frame drops.

#### 2. AWS Elemental Media Services Delivery Pipeline
* **AWS Elemental MediaConvert**: QVBR Level 9 quality-defined variable bitrate with 2.0s closed GOP alignment.
* **AWS Elemental MediaPackage v2**: Just-in-Time CMAF fMP4 packaging generating dynamic HLS (`.m3u8`) and MPEG-DASH (`.mpd`) manifests with **SPEKE v2.0 Multi-DRM** (Widevine L1 & Apple FairPlay).
* **AWS Elemental MediaTailor**: Server-Side Ad Insertion (SSAI) splicing targeted ad pods directly into stream timelines using SCTE-35 broadcast markers.
* **Amazon CloudFront CDN**: Edge caching with short-lived cryptographic signed URL tokens and Origin Shield.

#### 3. Real-Time Video Analytics & QoE Engine
* **In-Player "Stats for Nerds" HUD**: Live overlay tracking active bitrate, codec, CMAF buffer health, dropped frames, and Time-To-First-Frame (TTFF).
* **QoE Analytics Studio**: Real-time ITU-T P.1203 Quality-of-Experience scoring and dual-series Bitrate vs. Bandwidth time-series charts.
* **Network Throttling Simulator**: Test real-time ABR step-down under simulated network conditions (*Fiber 25 Mbps*, *4G LTE 8.5 Mbps*, *Congested Cellular 3.8 Mbps*, *3G 1.6 Mbps*).

#### 4. Resumable Chunked Offline CMAF Downloads
* Resumable chunked downloading with HTTP 206 byte offset checkpoints.
* Seamless offline state recovery when toggling airplane mode or network loss.
* Offline CMAF vault with 1-tap playback under the Profile tab.

#### 5. Prime Video "X-Ray" Scene Intelligence
* Real-time metadata cards synchronized with video playback displaying cast in scene, production trivia, soundtrack stems, and transcode specs.

#### 6. Persistent Mini-Player Dock
* Smoothly minimize active video playback into a floating, dockable mini-player while browsing discovery, analytics, or the cloud lab.

---

## 🛠️ Technical Specifications

| Parameter | Specification |
| :--- | :--- |
| **App Name** | AetherStream |
| **Package ID** | `com.example.ott_app` |
| **Build Target** | Android Release APK |
| **File Size** | 75.52 MB (79,185,185 bytes) |
| **Min SDK** | Android 7.0 (API Level 24) |
| **Target / Compile SDK** | Android 14+ (API Level 34) |
| **CPU ABIs Supported** | `arm64-v8a`, `armeabi-v7a`, `x86_64` |
| **Video Engine** | Android MediaCodec / Google ExoPlayer |
| **State Management** | BLoC & Cubit (`flutter_bloc`) |
| **CI/CD** | GitHub Actions Automated Build & Test Pipeline |
| **Source Code** | [https://github.com/Akshaypayya/AetherStream](https://github.com/Akshaypayya/AetherStream) |

---

## 👨‍💻 Developer & Source Repository

* **Full Source Code**: [github.com/Akshaypayya/AetherStream](https://github.com/Akshaypayya/AetherStream)
* **Author**: Akshay Chandran
* **Portfolio**: [Akshaypayya on GitHub](https://github.com/Akshaypayya)
