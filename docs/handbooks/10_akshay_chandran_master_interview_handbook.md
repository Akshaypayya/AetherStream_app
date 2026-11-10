# Akshay Chandran P: Master Mobile Engineer Interview Playbook
## Recruiter Scoring Rubric Alignment, STAR Stories & Leadership Synthesis
**Candidate:** Akshay Chandran P  
**Role:** Senior Flutter / Mobile Application Engineer / Full-Stack Specialist  
**Experience:** 4+ Years Enterprise Production Mobile Development  
**Audience:** Technical Directors, Engineering Managers, Principal Architects & HR Talent Acquisition

---

## Section 1: Candidate Executive Summary
Akshay Chandran P is an experienced Senior Mobile Application Engineer specializing in high-performance Flutter architectures, real-time multimedia streaming (AWS Elemental, CMAF, DRM), FinTech compliance (Saudi SAR, Nafath KYC), enterprise cloud ERP systems, and secure full-stack integrations.

---

## Section 2: Mapping to the Corporate Scoring Rubric

| College/Recruiter Rubric Category | Akshay's Direct Production Equivalent | Concrete Project Evidence | Score Alignment |
| :--- | :--- | :--- | :--- |
| **JetPack Compose** | **Flutter Declarative UI & Impeller Pipeline** | Built multi-platform reactive interfaces across mobile, web, and TV with 60 FPS performance in *AetherStream*, *GrowK*, and *Payzo Books*. | **5.0 / 5.0** |
| **MVI / MVVM Architecture** | **BLoC / Cubit & Clean Architecture** | Strict event-driven unidirectional data flow with domain separation, repositories, and state machines in *AetherStream* and *AutoRide*. | **5.0 / 5.0** |
| **Kotlin, Coroutines, Flow** | **Dart Isolates, Async/Await & Streams** | Asynchronous stream transformers (`debounceTime`, `switchMap`), isolate workers, and native platform channels (Kotlin/Swift). | **4.8 / 5.0** |
| **RxJava** | **RxDart & StreamTransformers** | Advanced stream manipulation in live search, sensor telemetry, and audio visualizer pipelines. | **4.8 / 5.0** |
| **GitHub Actions / CI/CD** | **Automated Multi-Flavor CI/CD Pipelines** | Fastlane, GitHub Actions, automated test execution, code signing, and Play Store / TestFlight delivery. | **5.0 / 5.0** |
| **REST APIs & WebSockets** | **Dual-Channel Split-Brain Engines** | RESTful services, Socket.IO real-time multiplexing, and WebRTC P2P signaling in *SyncTalk* and *GrowK*. | **5.0 / 5.0** |
| **Dagger / Hilt** | **Riverpod & GetIt / Injectable DI** | Type-safe compile-time dependency injection and scoped lifetime containers across 792 Dart files in *Payzo Books*. | **4.9 / 5.0** |
| **Espresso & JUnit** | **Flutter Unit, Widget & Integration Tests** | `blocTest`, Mockito, `testWidgets`, and `integration_test` automation with high branch coverage. | **4.8 / 5.0** |

---

## Section 3: High-Impact STAR Behavioral Interview Stories

### Story 1: Solving 4K Video Jank Under Extreme Memory Constraints (AetherStream)
- **Situation:** Users reported severe stuttering and app crashes when streaming 4K 60 FPS content on mid-range Android devices.
- **Task:** Eliminate UI jank and reduce memory usage without compromising video quality.
- **Action:** Profiled using Flutter DevTools and Android Profiler. Discovered that the legacy `PlatformView` implementation was performing double memory copies across threads. Replaced it with ExoPlayer rendering directly into an Android `SurfaceTexture` registered with Flutter's Texture Layer. Added `RepaintBoundary` around the player controls to isolate overlay animations.
- **Result:** Frame rate locked at solid 60 FPS, memory footprint plummeted by 64%, and app crash rates dropped from 3.2% to under 0.05%.

### Story 2: Preventing High-Risk Race Conditions in Ride Dispatch (AutoRide)
- **Situation:** Drivers in high-density areas were encountering collision bugs where two drivers simultaneously accepted the same ride request.
- **Task:** Engineer a bulletproof concurrency mechanism preventing duplicate assignments.
- **Action:** Migrated direct database updates to atomic Firebase `runTransaction` blocks with optimistic locking. If another driver committed a state change first, the transaction aborted and gracefully notified the second driver.
- **Result:** Completely eliminated duplicate ride bookings across thousands of test dispatches with zero database corruption.

---

*(Handbook concludes with salary negotiation strategy, technical leadership framework, and career portfolio roadmap)*
