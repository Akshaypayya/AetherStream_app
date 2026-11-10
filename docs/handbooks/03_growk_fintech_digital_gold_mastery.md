# GrowK: Enterprise FinTech, Digital Gold & Saudi Banking Platform
## Master Technical Handbook: Complete Working Architecture & Code Audit Q&A
**Candidate:** Akshay Chandran P  
**Target Role:** Senior Mobile Application Engineer / FinTech Architecture Specialist  
**Regulatory & Market Scope:** Saudi Arabian Capital Markets Authority (CMA), SAMA Compliance, Saudi SAR, Nafath KYC  
**Core Technologies:** Flutter, Dart, BLoC, WebSockets, Dual-Token OIDC, Hardware Keystore, Biometric Signing, RTL Localization

---

## Part 1: Comprehensive Working Explanation (Beginner to Extreme Architecture)

### 1.1 The Verbal Interview Method & System Architecture


---

## Part 2: Code Audit Findings, FinTech Risks & Production Hardening

### Audited Vulnerability 1: Floating-Point Rounding in Bullion Calculations
- **Discovered Risk:** The initial prototype used Dart `double` for SAR pricing and gold weights. Because IEEE-754 binary floating-point numbers cannot represent exact decimal fractions (e.g., `0.1 + 0.2 = 0.30000000000000004`), accumulated fractional halala discrepancies occurred over thousands of fractional gold transactions, which would fail Saudi central bank financial audits.
- **Senior Production Fix:** Completely migrated all financial arithmetic to integer **Minor Currency Units** (Halalas where 1 SAR = 100 Halalas) and milligrams (1g = 1,000mg) combined with the `decimal` package:
```dart
final gramsToBuy = Decimal.parse("2.540");
final pricePerGramHalalas = Decimal.fromInt(28450); // 284.50 SAR
final totalCostHalalas = (gramsToBuy * pricePerGramHalalas).toDecimal(scaleOnInfinitePrecision: 0);
```

### Audited Vulnerability 2: Live Price Slippage & Market Arbitrage
- **Discovered Risk:** Gold prices on international bullion exchanges (LBMA) fluctuate every millisecond. If a user presses "Buy" while viewing an older price quote, the company incurs slippage losses if market prices surge before the transaction executes.
- **Senior Production Fix:** Designed a **30-Second Cryptographic Quote Lock Engine**:
  1. The server issues a signed quote with a unique `quote_id`, `rate_per_gram`, `expires_at` timestamp, and an HMAC-SHA256 signature.
  2. Flutter UI renders a circular countdown progress timer indicating quote validity.
  3. The final trade execution payload must include `quote_id` and the user's biometric signature.
  4. The backend validates that the order was received before `expires_at`. If expired, the transaction is rejected, prompting the user to accept an updated live rate.

---

## Part 3: Exhaustive Code Audit Interview Q&A (Senior Flutter, Backend, Security, QA, Hiring Manager)


4. GrowK: FinTech Code Audit, Security Review and Deep Interview


<div class="qa-card">
  <div class="qa-q">Q427: [Senior Flutter Interviewer] GrowK: What is TLS validation override, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in growk/lib/main.dart. The HTTP override configures a badCertificateCallback that
accepts certificates.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q428: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The HTTP override configures a badCertificateCallback that accepts certificates. The design is useful
because it matches the feature requirement, but the important interview point is the boundary where the choice stops
being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q429: [Security Reviewer] GrowK: Spot the risk in TLS validation override.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/main.dart. Returning true bypasses normal certificate validation and can
undermine TLS against active network attackers.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q430: [QA Engineer] GrowK: How would you test TLS validation override?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Run against an invalid/self-signed certificate in a controlled test and verify the client rejects it.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q431: [Hiring Manager] GrowK: How would you improve TLS validation override for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use normal trust validation. If pinning is required, implement a carefully designed pinning strategy
without a trust-all callback.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q432: [Senior Flutter Interviewer] GrowK: What is Client credential flow, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in growk/lib/core/network/network_service.dart. The mobile app obtains a gate-
way token using client_credentials and uses a clientId/clientSecret configuration.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q433: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The mobile app obtains a gateway token using client_credentials and uses a clientId/clientSecret
configuration. The design is useful because it matches the feature requirement, but the important interview point is the
boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q434: [Security Reviewer] GrowK: Spot the risk in Client credential flow.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/core/network/network_service.dart. A native mobile binary cannot
safely protect a confidential client secret. A client_credentials flow assumes a trusted confidential client.
13</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q435: [QA Engineer] GrowK: How would you test Client credential flow?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Static-analyze the release artifact for secrets and test unauthorized attempts against the gateway.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q436: [Hiring Manager] GrowK: How would you improve Client credential flow for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Move confidential token acquisition behind a trusted backend or use a public-client flow such as
Authorization Code with PKCE when appropriate for the identity provider.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q437: [Senior Flutter Interviewer] GrowK: What is Access/refresh token storage, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in growk/lib/core/network/network_service.dart. Access and refresh tokens are
stored through SharedPreferencesHelper.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q438: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Access and refresh tokens are stored through SharedPreferencesHelper. The design is useful because it
matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q439: [Security Reviewer] GrowK: Spot the risk in Access/refresh token storage.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/core/network/network_service.dart. Preferences are not the strongest
storage boundary for long-lived authentication material.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q440: [QA Engineer] GrowK: How would you test Access/refresh token storage?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Test app restart, logout, token expiry, refresh failure and device compromise assumptions.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q441: [Hiring Manager] GrowK: How would you improve Access/refresh token storage for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use platform secure storage, minimize token lifetime, rotate/revoke refresh credentials and clear them
on logout/session invalidation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q442: [Senior Flutter Interviewer] GrowK: What is Sensitive token logging, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in growk/lib/core/network/network_service.dart. Diagnostic logs include token
prefixes and request/response information.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q443: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Diagnostic logs include token prefixes and request/response information. The design is useful because
it matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q444: [Security Reviewer] GrowK: Spot the risk in Sensitive token logging.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/core/network/network_service.dart. Even partial credentials and
authorization headers create unnecessary sensitive telemetry exposure.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q445: [QA Engineer] GrowK: How would you test Sensitive token logging?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Run debug and release logging tests and assert authorization headers/tokens are absent.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q446: [Hiring Manager] GrowK: How would you improve Sensitive token logging for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Redact secrets completely and log only safe metadata such as endpoint, status, latency and correlation
ID.
14</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q447: [Senior Flutter Interviewer] GrowK: What is OTP logging, and what does this code actually do?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/features/login/controller/otp_controller.dart. Login diagnostics can
print phone/OTP-related data.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q448: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Login diagnostics can print phone/OTP-related data. The design is useful because it matches the
feature requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q449: [Security Reviewer] GrowK: Spot the risk in OTP logging.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/features/login/controller/otp_controller.dart. OTP values are tem-
porary authentication secrets and must not be persisted in logs.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q450: [QA Engineer] GrowK: How would you test OTP logging?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Perform a login attempt and inspect logs for absence of the OTP value.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q451: [Hiring Manager] GrowK: How would you improve OTP logging for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Remove secret logging; use sanitized attempt IDs and test-environment fixtures instead.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q452: [Senior Flutter Interviewer] GrowK: What is IBAN logging/handling, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in growk/lib/features/bank_details/controller/bank_details_controller.dart.
Bank details validation includes diagnostic handling around IBAN values.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q453: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Bank details validation includes diagnostic handling around IBAN values. The design is useful because
it matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q454: [Security Reviewer] GrowK: Spot the risk in IBAN logging/handling.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/features/bank_details/controller/bank_details_controller.dart.
Financial identifiers should be minimized and masked in logs.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q455: [QA Engineer] GrowK: How would you test IBAN logging/handling?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Verify masked logging and test invalid/valid IBAN flows without exposing the full identifier.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q456: [Hiring Manager] GrowK: How would you improve IBAN logging/handling for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Keep raw IBANs in memory only as needed, mask diagnostic output and apply strict access/retention
controls.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q457: [Senior Flutter Interviewer] GrowK: What is T oken refresh concurrency , and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in growk/lib/core/network/network_service.dart. 401 responses can trigger refresh
logic and expired gateway tokens can trigger token acquisition.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q458: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> 401 responses can trigger refresh logic and expired gateway tokens can trigger token acquisition. The
design is useful because it matches the feature requirement, but the important interview point is the boundary where
the choice stops being safe or scalable.
15</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q459: [Security Reviewer] GrowK: Spot the risk in T oken refresh concurrency .</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/core/network/network_service.dart. Several requests can refresh or
acquire tokens at the same time, causing redundant calls or refresh-token conflicts.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q460: [QA Engineer] GrowK: How would you test T oken refresh concurrency?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Fire many concurrent requests after token expiry and verify only one refresh operation occurs.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q461: [Hiring Manager] GrowK: How would you improve T oken refresh concurrency for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use a shared refresh future/mutex and one retry path, then fail the session cleanly if refresh fails.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q462: [Senior Flutter Interviewer] GrowK: What is Goal pagination, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in growk/lib/features/goals/provider/goal_list_pagination.dart. The project
contains an in-memory collection/page slicing approach for goals.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q463: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The project contains an in-memory collection/page slicing approach for goals. The design is useful
because it matches the feature requirement, but the important interview point is the boundary where the choice stops
being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q464: [Security Reviewer] GrowK: Spot the risk in Goal pagination.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/features/goals/provider/goal_list_pagination.dart. If the full col-
lection is fetched before paging, memory and network cost still scale with total records.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q465: [QA Engineer] GrowK: How would you test Goal pagination?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Create a large dataset and compare network payload, memory and incremental loading behavior.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q466: [Hiring Manager] GrowK: How would you improve Goal pagination for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use server-driven pagination with stable ordering/cursor metadata for large datasets.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q467: [Senior Flutter Interviewer] GrowK: What is Goal date calculations, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in growk/lib/features/goals/services/goal_calculator.dart. Installment logic uses
different daily/weekly/monthly/yearly rules.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q468: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Installment logic uses different daily/weekly/monthly/yearly rules. The design is useful because it
matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q469: [Security Reviewer] GrowK: Spot the risk in Goal date calculations.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/features/goals/services/goal_calculator.dart. Month lengths, leap
years, end-date inclusivity and business semantics can create off-by-one or wrong installment counts.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q470: [QA Engineer] GrowK: How would you test Goal date calculations?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Test leap years, month ends, same-day ranges and each recurrence frequency.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q471: [Hiring Manager] GrowK: How would you improve Goal date calculations for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Document the business rule, use date-safe calculations and table-driven boundary tests.
16</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q472: [Senior Flutter Interviewer] GrowK: What is Standing instruction share, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in growk/lib/features/goals/goal_detail_page/controller/standing_instruction_controller.dart.
The sharing flow uses a RenderBox obtained from context and force-unwraps it.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q473: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The sharing flow uses a RenderBox obtained from context and force-unwraps it. The design is useful
because it matches the feature requirement, but the important interview point is the boundary where the choice stops
being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q474: [Security Reviewer] GrowK: Spot the risk in Standing instruction share.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/features/goals/goal_detail_page/controller/standing_instruction_controller.dart.
The expected RenderBox may not exist, causing a runtime failure. Shared bank information also deserves privacy
controls.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q475: [QA Engineer] GrowK: How would you test Standing instruction share?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Trigger share from lifecycle edge cases and test null RenderBox behavior.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q476: [Hiring Manager] GrowK: How would you improve Standing instruction share for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Null-check layout objects and keep sharing explicit, masked and user initiated.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q477: [Senior Flutter Interviewer] GrowK: What is Ref type casting, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in growk/lib/features/goals/goal_detail_page/controller/standing_instruction_controller.dart.
A helper casts a generic Ref toward WidgetRef.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q478: [Backend Interviewer] GrowK: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A helper casts a generic Ref toward WidgetRef. The design is useful because it matches the feature
requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q479: [Security Reviewer] GrowK: Spot the risk in Ref type casting.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in growk/lib/features/goals/goal_detail_page/controller/standing_instruction_controller.dart.
The cast suggests UI-specific coupling inside a controller and can fail if the actual type differs.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q480: [QA Engineer] GrowK: How would you test Ref type casting?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Unit-test the controller without a widget context and ensure no runtime cast is required.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q481: [Hiring Manager] GrowK: How would you improve Ref type casting for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Keep widget-only capabilities in the UI layer or pass the specific dependency the helper needs.</div>
</div>

---

## Part 4: Outside-World Enterprise FinTech Questions (Revolut, Robinhood Scale)

<div class="qa-card">
  <div class="qa-q">Q-EXT-01: How do you protect a FinTech app against Man-In-The-Middle (MITM) attacks and malicious Wi-Fi proxies?</div>
  <div class="qa-a"><strong>Model Answer:</strong> We implement Subject Public Key Info (SPKI) certificate pinning directly in the HTTP client (using `SecurityContext` in Dart or native network security configs). Even if an attacker installs a custom root CA certificate onto the user's phone, the app rejects connections unless the server's public key matches the pinned SHA-256 fingerprint compiled into the app binary.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q-EXT-02: How do you handle race conditions during token refresh when multiple API calls fail simultaneously with 401 Unauthorized?</div>
  <div class="qa-a"><strong>Model Answer:</strong> We implement an asynchronous request queue mutex in the HTTP interceptor. When the first 401 response is intercepted, subsequent failed requests are queued into an in-memory lock. Only a single refresh token request is dispatched. Once the new access token is received, the interceptor updates the default headers, retries all paused requests concurrently, and releases the queue lock.</div>
</div>
