# Ironclad Mobile Security SDK: RASP & Defense-in-Depth
## Master Technical Handbook: Architecture, Cryptography & Threat Prevention
**Candidate:** Akshay Chandran P  
**Target Role:** Senior Mobile Application Engineer / Mobile Security Specialist  
**Core Technologies:** Flutter, Dart, Kotlin, Swift, Platform Channels, RASP, Hardware Keystore, Secure Enclave, SPKI Cert Pinning

---

## Part 1: Comprehensive Working Explanation (Beginner to Extreme Architecture)

### 1.1 Beginner Level: What is RASP (Runtime Application Self-Protection)?
Standard mobile apps trust the operating system blindly. However, attackers can run rooted phones, install reverse-engineering tools like **Frida** or **Xposed**, attach debuggers, intercept network traffic via proxies (Burp Suite), or modify application code to bypass paywalls and authentication checks.  
**Ironclad** is an enterprise-grade mobile security SDK that embeds proactive defense mechanisms directly into the mobile binary:
- Continuously inspects device integrity at runtime.
- Detects debugging, hooking, emulation, and code tampering.
- Encrypts sensitive credentials in hardware tamper-resistant chips.
- Terminate execution or alert backend SIEM systems when threats are detected.

---

### 1.2 Extreme Senior Level: RASP Heuristics & Threat Detection Matrix

| Threat Vector | Detection Mechanism | Implementation Layer |
| :--- | :--- | :--- |
| **Root Detection (Android)** | Checks for `su` binary paths (`/system/bin/su`, `/system/xbin/su`), Magisk mount points in `/proc/mounts`, `test-keys` build tags in `Build.TAGS`, and attempts to execute `which su`. | Native C++ / NDK & Kotlin Platform Channel |
| **Jailbreak Detection (iOS)** | Checks for Cydia / Sileo app paths (`/Applications/Cydia.app`), sandbox integrity violations (attempting to write outside `/private`), and symbolic link checks on `/Applications`. | Native Objective-C / Swift Platform Channel |
| **Frida & Hooking Detection** | Scans memory mappings in `/proc/self/maps` for `frida-agent.so` or `gadget.so`; checks if standard Frida communication ports (TCP 27042) are listening; detects D-Bus tampering. | Native C++ layer via JNI |
| **Anti-Debugging** | Invokes Linux `ptrace(PTRACE_TRACEME, 0, 1, 0)` on Android; checks `sysctl` `P_TRACED` flag in iOS kernel structures. If a debugger is attached, the app exits immediately with exit code 0. | Native Posix Kernel System Calls |
| **Emulator Detection** | Evaluates CPU architecture strings, QEMU drivers, hardware fingerprints (`goldfish`, `ranchu`), telephony provider states, and battery sensor data. | Native Android/iOS Sensor APIs |

---

### 1.3 Hardware Keystore AES-256 Envelope Encryption
To protect local databases (Hive / SQLite) and user session tokens:
1. **Master Key Generation:** The app requests the Android Keystore (backed by StrongBox / TEE) or iOS Secure Enclave to generate an AES-256 key pair.
2. **Envelope Encryption:**
   - The hardware Master Key encrypts a randomly generated ephemeral **Data Encryption Key (DEK)**.
   - The DEK encrypts the database records using AES-256-GCM with a unique 96-bit initialization vector (IV) per record.
   - Even if an attacker extracts raw SQLite or Hive database files via root ADB access, the database is unreadable without the Master Key locked inside the physical hardware chip.

---

### 1.4 Network Security: SPKI Certificate Pinning
To defeat Man-In-The-Middle (MITM) proxy tools like Charles or Burp Suite:
- The SDK pins the Subject Public Key Info (SPKI) SHA-256 cryptographic digest of the backend server's public key.
- During the TLS 1.3 handshake, the mobile network stack computes the SHA-256 hash of the received server certificate's public key.
- If the hash differs from the hardcoded pin, the TLS socket terminates immediately with a certificate verification exception, preventing eavesdropping on sensitive API payloads.

---

## Part 2: Code Audit Findings, Security Risks & Production Hardening

### Audited Vulnerability 1: Frida Method Replacement Bypassing Security Checks
- **Discovered Risk:** In naive Flutter implementations, calling `bool isDeviceRooted()` from Dart allows an attacker using Frida to hook the Dart/Java bridge and force the return value to `false`.
- **Senior Production Fix:** Moved threat evaluations into native obfuscated C++ binaries compiled via Android NDK. Rather than returning a simple boolean to Dart, the security verification derives an ephemeral cryptographic session token: if the device is tampered, the derivation algorithm fails silently, causing subsequent backend API calls to fail signature validation.

### Audited Vulnerability 2: Key Revocation on Biometric Enrollment Changes
- **Discovered Risk:** If a phone's lock screen was compromised and a thief enrolled their own fingerprint, existing cryptographic keys could still be unlocked.
- **Senior Production Fix:** Configured the Android Keystore key with `setUserAuthenticationParameters(0, KeyProperties.AUTH_BIOMETRIC_STRONG)` and `setInvalidatedByBiometricEnrollment(true)`. If a new fingerprint is enrolled into the operating system, the hardware Master Key is instantly and irreversibly destroyed.

---

## Part 3: Exhaustive Security Q&A Section (OWASP Mobile Top 10)

<div class="qa-card">
  <div class="qa-q">Q-SEC-01: How do you structure a modular security SDK for publication on pub.dev?</div>
  <div class="qa-a"><strong>Model Answer:</strong> We follow Flutter's Federated Plugin Architecture: `ironclad_platform_interface` defines abstract platform interfaces; `ironclad_android` and `ironclad_ios` implement native platform channels; and `ironclad` exposes clean, developer-friendly Dart APIs with strong type definitions and mockable interfaces for unit testing.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q-SEC-02: What is the difference between Widevine L1, L2, and L3?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Widevine L1 processes all cryptographic operations and video frame decoding inside the hardware Trusted Execution Environment (TEE); raw decrypted video frames never touch main OS memory, enabling 4K and 1080p streaming. Widevine L2 performs crypto in the TEE but decodes video in software. Widevine L3 performs all decryption and decoding in user-space software, restricting video quality to 480p SD due to vulnerability to screen scraping and memory dumping.</div>
</div>
