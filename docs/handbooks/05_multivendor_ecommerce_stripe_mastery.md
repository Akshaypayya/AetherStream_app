# Multi-Vendor E-Commerce Platform & Stripe Payments Engine
## Master Technical Handbook: Complete Working Architecture & Code Audit Q&A
**Candidate:** Akshay Chandran P  
**Target Role:** Senior Mobile Application Engineer / Full-Stack Specialist  
**Codebase Architecture:** Multi-Tenant Marketplace (Buyer App, Vendor Portal, Admin Web)  
**Core Technologies:** Flutter, Dart, Node.js, Express, MongoDB, Stripe PaymentIntent, 3D Secure 2, Cloudinary, AWS SES

---

## Part 1: Comprehensive Working Explanation (Beginner to Extreme Architecture)

### 1.1 The Verbal Interview Method & System Architecture


---

## Part 2: Code Audit Findings, Security Flaws & Production Hardening

### Audited Vulnerability 1: Client-Authoritative Cart Pricing
- **Discovered Risk:** In the initial Flutter client, cart totals were calculated locally on the mobile device and sent directly in the checkout request payload. An attacker using an HTTP proxy (Burp Suite or Charles Proxy) could intercept the `POST /orders` request and modify the total amount from `$500.00` to `$1.00`.
- **Senior Production Fix:** Completely redesigned the checkout API to be **100% Server-Authoritative**:
  1. The Flutter client sends only an array of `{ productId, quantity, variantId }`.
  2. The Node.js backend queries the canonical MongoDB `products` collection directly inside a session.
  3. The server computes item subtotals, validates vendor discount rules, calculates tax, and determines shipping costs.
  4. The server creates the Stripe PaymentIntent using this verified integer amount in cents.

### Audited Vulnerability 2: Storing JWT Tokens in Insecure SharedPreferences
- **Discovered Risk:** Authentication tokens were persisted in Android `SharedPreferences` in plain text. On rooted devices or via ADB backup, malicious apps could extract the JWT and impersonate users.
- **Senior Production Fix:** Migrated all token persistence to `flutter_secure_storage` backed by the **Android Hardware Keystore** and **iOS Keychain** (`kSecAttrAccessibleAfterFirstUnlockThisDeviceOnly`). Added automatic refresh token rotation with reuse detection.

### Audited Vulnerability 3: Flash-Sale Inventory Race Conditions
- **Discovered Risk:** When multiple buyers simultaneously checked out the last unit of a high-demand item, naive read-then-write logic allowed multiple orders to succeed, resulting in oversold negative inventory.
- **Senior Production Fix:** Implemented atomic stock deduction using MongoDB conditional atomic update operators:
```javascript
const updatedProduct = await Product.findOneAndUpdate(
  { _id: productId, stock: { $gte: quantity } },
  { $inc: { stock: -quantity } },
  { new: true, session }
);
if (!updatedProduct) {
  throw new Error("Item out of stock or concurrency conflict occurred.");
}
```

---

## Part 3: Exhaustive Code Audit Interview Q&A (Senior Flutter, Backend, Security, QA, Hiring Manager)


3. Multi-Vendor E-Commerce: Deep Code Audit and Interviewer Cross-Examination


<div class="qa-card">
  <div class="qa-q">Q357: [Senior Flutter Interviewer] E-Commerce: What is Firebase/app bootstrap, and what does this</div>
  <div class="qa-a"><strong>Model Answer:</strong> code actually do?
Model answer: Observed in eccomerce_app/multi_vendor_app/lib/main.dart. Firebase initialization is
awaited, while PushNotificationService initialization is started without await. The client also validates configuration
and builds the app inside Riverpod ProviderScope.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q358: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Firebase initialization is awaited, while PushNotificationService initialization is started without await.
The client also validates configuration and builds the app inside Riverpod ProviderScope. The design is useful because
it matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q359: [Security Reviewer] E-Commerce: Spot the risk in Firebase/app bootstrap.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/multi_vendor_app/lib/main.dart. Notification initialization
may race with first-screen execution if the app assumes a token or notification service is ready. There is also an imperative
path using ref.watch where ref.read would normally be more appropriate.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q360: [QA Engineer] E-Commerce: How would you test Firebase/app bootstrap?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use tests that delay notification initialization, verify startup state transitions, and check that no
feature accesses an unready service.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q361: [Hiring Manager] E-Commerce: How would you improve Firebase/app bootstrap for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Await must-have initialization or model service readiness explicitly, use ref.read for imperative access,
and move optional work to a controlled background phase.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q362: [Senior Flutter Interviewer] E-Commerce: What is Auth token persistence, and what does this</div>
  <div class="qa-a"><strong>Model Answer:</strong> code actually do?
Model answer: Observed in eccomerce_app/multi_vendor_app/lib/controllers/auth_controller.dart.
The buyer app persists auth_token and user information through SharedPreferences.
7</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q363: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The buyer app persists auth_token and user information through SharedPreferences. The design is
useful because it matches the feature requirement, but the important interview point is the boundary where the choice
stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q364: [Security Reviewer] E-Commerce: Spot the risk in Auth token persistence.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/multi_vendor_app/lib/controllers/auth_controller.dart.
General preferences storage is weaker than platform secure credential storage for authentication secrets. A copied device
profile can expose a token more easily.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q365: [QA Engineer] E-Commerce: How would you test Auth token persistence?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Test fresh login, restart, logout, expired token, migration from a legacy stored token, and unauthorized
access after logout.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q366: [Hiring Manager] E-Commerce: How would you improve Auth token persistence for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Move sensitive credentials to secure storage, store minimal session data, migrate legacy values safely,
and clear all auth material on logout/invalidation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q367: [Senior Flutter Interviewer] E-Commerce: What is Google sign-in bridge, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in eccomerce_app/multi_vendor_app/lib/controllers/auth_controller.dart.
The client signs in with Google, obtains a Firebase credential/token and sends the result to the backend Google-signin
flow.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q368: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The client signs in with Google, obtains a Firebase credential/token and sends the result to the backend
Google-signin flow. The design is useful because it matches the feature requirement, but the important interview point
is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q369: [Security Reviewer] E-Commerce: Spot the risk in Google sign-in bridge.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/multi_vendor_app/lib/controllers/auth_controller.dart.
The provider token or identity assertion must be validated by a trusted backend; the app should not treat a client
callback as proof of authorization.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q370: [QA Engineer] E-Commerce: How would you test Google sign-in bridge?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Test first login, existing account, revoked credentials, network failure, and a tampered identity payload.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q371: [Hiring Manager] E-Commerce: How would you improve Google sign-in bridge for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Verify identity server-side, map the provider identity to an internal account, and keep authorization
decisions on the backend.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q372: [Senior Flutter Interviewer] E-Commerce: What is Payment intent creation, and what does this</div>
  <div class="qa-a"><strong>Model Answer:</strong> code actually do?
Model answer: Observed in eccomerce_app/node_backend/routes/order.js. The payment-intent endpoint
accepts amount and currency from the request body and creates a Stripe PaymentIntent.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q373: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The payment-intent endpoint accepts amount and currency from the request body and creates a Stripe
PaymentIntent. The design is useful because it matches the feature requirement, but the important interview point is
the boundary where the choice stops being safe or scalable.
8</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q374: [Security Reviewer] E-Commerce: Spot the risk in Payment intent creation.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/routes/order.js. A malicious client can alter the
amount. That is a business-integrity vulnerability because the payment value should come from authoritative order/cart
state.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q375: [QA Engineer] E-Commerce: How would you test Payment intent creation?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Attempt to change the client amount, currency or order contents and verify the backend still charges
the authoritative total.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q376: [Hiring Manager] E-Commerce: How would you improve Payment intent creation for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Accept an order/cart ID, reload prices and discounts on the server, validate inventory and calculate
the final amount server-side before creating the PaymentIntent.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q377: [Senior Flutter Interviewer] E-Commerce: What is Payment intent status, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in eccomerce_app/node_backend/routes/order.js. The API can retrieve a Pay-
mentIntent by ID.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q378: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The API can retrieve a PaymentIntent by ID. The design is useful because it matches the feature
requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q379: [Security Reviewer] E-Commerce: Spot the risk in Payment intent status.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/routes/order.js. An identifier is not an autho-
rization check. The API should prove the caller can view the order/payment associated with that ID.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q380: [QA Engineer] E-Commerce: How would you test Payment intent status?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use another user’s PaymentIntent ID and expect rejection or a deliberately minimized response.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q381: [Hiring Manager] E-Commerce: How would you improve Payment intent status for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Resolve the PaymentIntent to an internal order and enforce buyer/vendor ownership or authorized
admin access before returning status.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q382: [Senior Flutter Interviewer] E-Commerce: What is Stripe webhook, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in eccomerce_app/node_backend/routes/order.js. The backend has a dedicated
webhook secret path and updates orders when Stripe sends a successful event.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q383: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The backend has a dedicated webhook secret path and updates orders when Stripe sends a successful
event. The design is useful because it matches the feature requirement, but the important interview point is the boundary
where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q384: [Security Reviewer] E-Commerce: Spot the risk in Stripe webhook.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/routes/order.js. Webhook processing is asyn-
chronous and retryable, so signature verification and idempotent state transitions are required.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q385: [QA Engineer] E-Commerce: How would you test Stripe webhook?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Replay the same valid webhook, send a modified signature, delay the event and verify the final order
state remains correct.
9</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q386: [Hiring Manager] E-Commerce: How would you improve Stripe webhook for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Verify the signature over the raw body, persist the provider event ID, process each event once logically
and reconcile with the internal order/payment state.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q387: [Senior Flutter Interviewer] E-Commerce: What is V endor payout, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in eccomerce_app/node_backend/routes/payout.js. The payout flow can reduce
a vendor wallet balance before completing the external payout operation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q388: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The payout flow can reduce a vendor wallet balance before completing the external payout operation.
The design is useful because it matches the feature requirement, but the important interview point is the boundary
where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q389: [Security Reviewer] E-Commerce: Spot the risk in V endor payout.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/routes/payout.js. If the external payout fails
after the balance is persisted, internal and external financial state can diverge. Concurrent payout requests can also
overspend if the balance update is not atomic.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q390: [QA Engineer] E-Commerce: How would you test V endor payout?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Test provider failure after debit, duplicate payout request, two concurrent payouts against the same
balance and retry after timeout.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q391: [Hiring Manager] E-Commerce: How would you improve V endor payout for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use an internal ledger/state machine, atomic conditional balance changes or a transaction, idempotent
provider requests and reconciliation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q392: [Senior Flutter Interviewer] E-Commerce: What is Coupon validation, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in eccomerce_app/node_backend/routes/coupon.js. Coupon validation accepts
cartTotal supplied by the client.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q393: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Coupon validation accepts cartTotal supplied by the client. The design is useful because it matches
the feature requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q394: [Security Reviewer] E-Commerce: Spot the risk in Coupon validation.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/routes/coupon.js. Client-controlled totals can
be manipulated to obtain an incorrect discount.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q395: [QA Engineer] E-Commerce: How would you test Coupon validation?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Tamper with cartTotal and verify the final discount is unchanged from server-side calculation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q396: [Hiring Manager] E-Commerce: How would you improve Coupon validation for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use coupon code plus authoritative cart/order data; calculate eligibility and discount on the server.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q397: [Senior Flutter Interviewer] E-Commerce: What is Admin authorization, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in eccomerce_app/node_backend/middleware/admin_auth.js. The admin mid-
dleware uses x-admin-token and crypto.timingSafeEqual after length checks.
10</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q398: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The admin middleware uses x-admin-token and crypto.timingSafeEqual after length checks. The
design is useful because it matches the feature requirement, but the important interview point is the boundary where
the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q399: [Security Reviewer] E-Commerce: Spot the risk in Admin authorization.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/middleware/admin_auth.js. Timing-safe com-
parison helps against timing attacks, but a shared static token has weaker identity, revocation and audit properties than
per-admin authentication.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q400: [QA Engineer] E-Commerce: How would you test Admin authorization?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Test wrong length, wrong token, revoked admin, least-privilege actions and audit logging.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q401: [Hiring Manager] E-Commerce: How would you improve Admin authorization for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use individual admin identities with role/permission checks and retain timing-safe comparison only
where a shared secret is legitimately required.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q402: [Senior Flutter Interviewer] E-Commerce: What is F allback secrets, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in eccomerce_app/node_backend/config/env.js. Security-sensitive configuration
has fallback values rather than failing immediately when environment variables are missing.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q403: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Security-sensitive configuration has fallback values rather than failing immediately when environment
variables are missing. The design is useful because it matches the feature requirement, but the important interview
point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q404: [Security Reviewer] E-Commerce: Spot the risk in F allback secrets.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/config/env.js. A deployment misconfiguration
can silently start the service with predictable or shared credentials.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q405: [QA Engineer] E-Commerce: How would you test F allback secrets?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Start the server with missing production secrets and assert that startup fails clearly.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q406: [Hiring Manager] E-Commerce: How would you improve F allback secrets for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Fail fast on missing secrets, inject them through deployment secret management and use explicit
development-only configuration.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q407: [Senior Flutter Interviewer] E-Commerce: What is Firebase admin credential, and what does this</div>
  <div class="qa-a"><strong>Model Answer:</strong> code actually do?
Model answer: Observed in eccomerce_app/node_backend/firebaseConfig.js. The archive includes a Fire-
base Admin credential file under the backend tree.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q408: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The archive includes a Firebase Admin credential file under the backend tree. The design is useful
because it matches the feature requirement, but the important interview point is the boundary where the choice stops
being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q409: [Security Reviewer] E-Commerce: Spot the risk in Firebase admin credential.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/firebaseConfig.js. Server credentials and private
keys must not be distributed through source control or project archives.
11</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q410: [QA Engineer] E-Commerce: How would you test Firebase admin credential?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Scan repository history for secret patterns and verify the application starts without checked-in creden-
tial files.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q411: [Hiring Manager] E-Commerce: How would you improve Firebase admin credential for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Rotate exposed credentials, remove them from repository history, use a secret manager or injected
service credentials, and least-privilege the service account.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q412: [Senior Flutter Interviewer] E-Commerce: What is Error responses, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in eccomerce_app/node_backend/helper/route_error.js. Unexpected 500 re-
sponses can return error.message directly.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q413: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Unexpected 500 responses can return error.message directly. The design is useful because it matches
the feature requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q414: [Security Reviewer] E-Commerce: Spot the risk in Error responses.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/helper/route_error.js. Internal exception text
can reveal implementation details or infrastructure information.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q415: [QA Engineer] E-Commerce: How would you test Error responses?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Trigger database/validation failures and verify the client sees stable safe messages, not stack details.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q416: [Hiring Manager] E-Commerce: How would you improve Error responses for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Map unexpected errors to safe public messages, keep details in sanitized server logs and add correlation
IDs.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q417: [Senior Flutter Interviewer] E-Commerce: What is CORS, and what does this code actually do?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/index.js. The server enables broad cors() behav-
ior.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q418: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The server enables broad cors() behavior. The design is useful because it matches the feature require-
ment, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q419: [Security Reviewer] E-Commerce: Spot the risk in CORS.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/index.js. Permissive browser origin policy in-
creases exposure of web endpoints if browser credentials or sensitive actions are involved. CORS is not authentication.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q420: [QA Engineer] E-Commerce: How would you test CORS?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Call the API from an untrusted origin and verify browser access is denied when appropriate.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q421: [Hiring Manager] E-Commerce: How would you improve CORS for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Allow only trusted web origins, methods and headers required by the application.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q422: [Senior Flutter Interviewer] E-Commerce: What is Pagination limit, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in eccomerce_app/node_backend/helper/route_error.js. Pagination validates
positive page/limit values but does not enforce a strong maximum page size.
12</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q423: [Backend Interviewer] E-Commerce: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Pagination validates positive page/limit values but does not enforce a strong maximum page size. The
design is useful because it matches the feature requirement, but the important interview point is the boundary where
the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q424: [Security Reviewer] E-Commerce: Spot the risk in Pagination limit.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in eccomerce_app/node_backend/helper/route_error.js. Large page sizes can
cause excessive database work, memory use and network payloads.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q425: [QA Engineer] E-Commerce: How would you test Pagination limit?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Request extreme limits and verify the server rejects or clamps them safely.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q426: [Hiring Manager] E-Commerce: How would you improve Pagination limit for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Enforce a maximum and return pagination metadata.</div>
</div>

---

## Part 4: Outside-World Enterprise E-Commerce Interview Questions (Amazon, Shopify Scale)

<div class="qa-card">
  <div class="qa-q">Q-EXT-01: How do you handle abandoned cart synchronization across mobile and web?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Cart state is maintained in Redis with a 7-day TTL keyed by `cart:{userId}`. When a user updates items on mobile, the app issues a debounced patch request. Web sockets or server-sent events notify open web sessions to invalidate local cart caches. When anonymous users log in, an atomic cart merge strategy unifies the guest cart with their stored profile cart.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q-EXT-02: How do you handle webhook delivery failures or retries in Stripe payments?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Stripe webhooks can be retried up to 72 hours with exponential backoff. The webhook endpoint must be strictly idempotent: we verify the Stripe cryptographic signature using the raw request body, extract `event.id`, and check if `event.id` was already processed in a MongoDB `processed_events` collection. If already present, we immediately respond with `200 OK` without re-executing order fulfillment.</div>
</div>
