# SyncTalk: Real-Time Messaging & WebRTC Calling Engine
## Master Technical Handbook: Complete Working Architecture & Code Audit Q&A
**Candidate:** Akshay Chandran P  
**Target Role:** Senior Mobile Application Engineer / Real-Time Communications Specialist  
**Core Technologies:** Flutter, Node.js, TypeScript, Socket.IO, WebRTC, Redis Pub/Sub, Hive Encrypted Outbox, FCM Data Wakeup

---

## Part 1: Comprehensive Working Explanation (Beginner to Extreme Architecture)

### 1.1 The Verbal Interview Method & System Architecture


---

## Part 2: Code Audit Findings, Security Flaws & Production Hardening

### Audited Vulnerability 1: In-Flight Message Loss During Sudden Connection Drops
- **Discovered Risk:** In the initial prototype, sending messages while entering an elevator or subway tunnel resulted in dropped packets and lost messages without visual feedback to the user.
- **Senior Production Fix:** Engineered a **Local-First Encrypted Outbox Queue**:
  1. Messages are written immediately to local encrypted **Hive** storage with state `PENDING_SYNC` (clock icon).
  2. The message is pushed into a FIFO persistent dispatch queue.
  3. A background connectivity listener monitors Socket.IO socket states.
  4. Upon socket reconnection, the queue flushes pending messages sequentially. The server responds with an ACK payload containing the official server timestamp, updating status to `SENT` (single check), `DELIVERED` (double checks), and `READ` (blue checks).

### Audited Vulnerability 2: WebRTC Media Freezes Behind Corporate Symmetric NAT
- **Discovered Risk:** P2P audio/video calls failed when either peer was connected to a strict enterprise or university Wi-Fi network that prohibits direct UDP peer connections.
- **Senior Production Fix:** Deployed a geo-distributed **Coturn TURN (Traversal Using Relays around NAT)** cluster with dynamic temporary credentials generated via REST API. When STUN NAT hole-punching fails during ICE candidate gathering, the WebRTC peer connection automatically transitions media packet routing through the encrypted TURN relay without dropping the call.

---

## Part 3: Exhaustive Code Audit Interview Q&A (Senior Flutter, Backend, Security, QA, Hiring Manager)


7. SyncTalk: Deep Code Audit - Real-Time Messaging, Offline Queue, WebRTC and
Backend Security


<div class="qa-card">
  <div class="qa-q">Q572: [Senior Flutter Interviewer] SyncT alk: What is API base URL, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in sync talk/sync_talk_mobile/lib/core/services/api.dart. The mobile API client
has an API_BASE environment value with a LAN IP default.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q573: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The mobile API client has an API_BASE environment value with a LAN IP default. The design is
useful because it matches the feature requirement, but the important interview point is the boundary where the choice
stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q574: [Security Reviewer] SyncT alk: Spot the risk in API base URL.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_mobile/lib/core/services/api.dart. A developer-network
address is not a suitable production default and can leak environment assumptions into release builds.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q575: [QA Engineer] SyncT alk: How would you test API base URL?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Build release/staging variants and assert the resulting endpoint is correct.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q576: [Hiring Manager] SyncT alk: How would you improve API base URL for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Require explicit environment configuration and validate release endpoints in CI.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q577: [Senior Flutter Interviewer] SyncT alk: What is Dio 401 refresh, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in sync talk/sync_talk_mobile/lib/core/services/api.dart. The interceptor re-
freshes tokens on 401 and retries the original request.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q578: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The interceptor refreshes tokens on 401 and retries the original request. The design is useful because
it matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q579: [Security Reviewer] SyncT alk: Spot the risk in Dio 401 refresh.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_mobile/lib/core/services/api.dart. Concurrent 401s can
trigger multiple refresh operations, especially with rotating refresh tokens.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q580: [QA Engineer] SyncT alk: How would you test Dio 401 refresh?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Expire the token and issue many concurrent API calls; verify one refresh and correct retries.
25</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q581: [Hiring Manager] SyncT alk: How would you improve Dio 401 refresh for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use a single-flight refresh future/mutex and mark requests so retry occurs once.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q582: [Senior Flutter Interviewer] SyncT alk: What is Offline queue ID, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in sync talk/sync_talk_mobile/lib/core/services/offline_queue_service.dart.
Pending messages use a millisecond timestamp as a queue key.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q583: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Pending messages use a millisecond timestamp as a queue key. The design is useful because it matches
the feature requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q584: [Security Reviewer] SyncT alk: Spot the risk in Offline queue ID.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_mobile/lib/core/services/offline_queue_service.dart.
Two enqueue operations in the same millisecond can collide.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q585: [QA Engineer] SyncT alk: How would you test Offline queue ID?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Enqueue many messages concurrently and assert unique queue records.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q586: [Hiring Manager] SyncT alk: How would you improve Offline queue ID for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use UUIDs or another guaranteed-unique queue ID.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q587: [Senior Flutter Interviewer] SyncT alk: What is Offline delivery semantics, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_mobile/lib/core/services/offline_queue_service.dart.
The queue posts pending messages when connectivity returns and removes the item after success.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q588: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The queue posts pending messages when connectivity returns and removes the item after success. The
design is useful because it matches the feature requirement, but the important interview point is the boundary where
the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q589: [Security Reviewer] SyncT alk: Spot the risk in Offline delivery semantics.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_mobile/lib/core/services/offline_queue_service.dart.
A crash after server success but before local deletion can cause duplicate sends.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q590: [QA Engineer] SyncT alk: How would you test Offline delivery semantics?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Inject a crash between successful response and delete, then reconnect and verify no duplicate logical
message.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q591: [Hiring Manager] SyncT alk: How would you improve Offline delivery semantics for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use stable client message IDs plus server-side idempotency and explicit retry/backoff.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q592: [Senior Flutter Interviewer] SyncT alk: What is W ebR TC ICE configuration, and what does this</div>
  <div class="qa-a"><strong>Model Answer:</strong> code actually do?
Model answer: Observed in sync talk/sync_talk_mobile/lib/features/call/view/call_screen.dart. The
peer connection configuration includes STUN but no TURN server.
26</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q593: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The peer connection configuration includes STUN but no TURN server. The design is useful because
it matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q594: [Security Reviewer] SyncT alk: Spot the risk in W ebR TC ICE configuration.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_mobile/lib/features/call/view/call_screen.dart. Some
restrictive NAT/firewall conditions prevent direct connections, so calls can fail even when signaling works.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q595: [QA Engineer] SyncT alk: How would you test W ebR TC ICE configuration?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Test calls across restrictive networks and verify a TURN candidate can establish connectivity.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q596: [Hiring Manager] SyncT alk: How would you improve W ebR TC ICE configuration for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Add TURN relay infrastructure, secure credential provisioning and connection telemetry.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q597: [Senior Flutter Interviewer] SyncT alk: What is Secure session storage, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_mobile/lib/core/services/session.dart. Active session
storage uses FlutterSecureStorage while older Hive implementations are commented out.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q598: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Active session storage uses FlutterSecureStorage while older Hive implementations are commented
out. The design is useful because it matches the feature requirement, but the important interview point is the boundary
where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q599: [Security Reviewer] SyncT alk: Spot the risk in Secure session storage.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_mobile/lib/core/services/session.dart. Commented
legacy implementations create confusion and increase maintenance risk.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q600: [QA Engineer] SyncT alk: How would you test Secure session storage?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Audit logout, app restart, token replacement and migration behavior.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q601: [Hiring Manager] SyncT alk: How would you improve Secure session storage for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Keep one authoritative secure-storage path and remove obsolete code.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q602: [Senior Flutter Interviewer] SyncT alk: What is F orward-message authorization, and what does this</div>
  <div class="qa-a"><strong>Model Answer:</strong> code actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/modules/messages/messages.service.ts.
Forwarding checks destination membership but does not clearly validate access to the original message.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q603: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Forwarding checks destination membership but does not clearly validate access to the original message.
The design is useful because it matches the feature requirement, but the important interview point is the boundary
where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q604: [Security Reviewer] SyncT alk: Spot the risk in F orward-message authorization.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/messages/messages.service.ts.
Knowing a message ID should not grant access to content from an unrelated conversation.
27</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q605: [QA Engineer] SyncT alk: How would you test F orward-message authorization?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Attempt to forward a message from a conversation the requester does not belong to.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q606: [Hiring Manager] SyncT alk: How would you improve F orward-message authorization for produc-</div>
  <div class="qa-a"><strong>Model Answer:</strong> tion?
Model answer: Resolve the source message, verify membership in its source conversation, then create the forwarded
message.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q607: [Senior Flutter Interviewer] SyncT alk: What is Message reaction authorization, and what does this</div>
  <div class="qa-a"><strong>Model Answer:</strong> code actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/modules/messages/messages.service.ts.
Reaction handling mutates message reactions without the same explicit membership check used in other message oper-
ations.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q608: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Reaction handling mutates message reactions without the same explicit membership check used in
other message operations. The design is useful because it matches the feature requirement, but the important interview
point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q609: [Security Reviewer] SyncT alk: Spot the risk in Message reaction authorization.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/messages/messages.service.ts.
Object mutation without conversation authorization can become an IDOR-style flaw.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q610: [QA Engineer] SyncT alk: How would you test Message reaction authorization?</div>
  <div class="qa-a"><strong>Model Answer:</strong> React to another user’s inaccessible message ID and expect a 403.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q611: [Hiring Manager] SyncT alk: How would you improve Message reaction authorization for produc-</div>
  <div class="qa-a"><strong>Model Answer:</strong> tion?
Model answer: Verify conversation membership before any message mutation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q612: [Senior Flutter Interviewer] SyncT alk: What is Read-state authorization, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/modules/messages/messages.service.ts.
The read flow updates readBy but does not consistently verify conversation membership before mutation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q613: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The read flow updates readBy but does not consistently verify conversation membership before
mutation. The design is useful because it matches the feature requirement, but the important interview point is the
boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q614: [Security Reviewer] SyncT alk: Spot the risk in Read-state authorization.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/messages/messages.service.ts.
A caller may mutate another conversation’s read state or infer content-related metadata.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q615: [QA Engineer] SyncT alk: How would you test Read-state authorization?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use an unrelated message ID with an authenticated account and expect rejection.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q616: [Hiring Manager] SyncT alk: How would you improve Read-state authorization for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Authorize membership before changing read state and emitting read events.
28</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q617: [Senior Flutter Interviewer] SyncT alk: What is Report authorization, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/modules/messages/messages.service.ts.
Message report handling does not enforce the same membership boundary.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q618: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Message report handling does not enforce the same membership boundary. The design is useful because
it matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q619: [Security Reviewer] SyncT alk: Spot the risk in Report authorization.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/messages/messages.service.ts.
Unauthorized users can potentially create moderation records for objects they should not access.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q620: [QA Engineer] SyncT alk: How would you test Report authorization?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Report inaccessible messages and verify authorization failure.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q621: [Hiring Manager] SyncT alk: How would you improve Report authorization for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Verify source conversation membership, validate report reason and create an audit record.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q622: [Senior Flutter Interviewer] SyncT alk: What is Unread N+1, and what does this code actually do?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/conversations/conversations.service.ts.
Unread counts are calculated separately while listing conversations.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q623: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Unread counts are calculated separately while listing conversations. The design is useful because it
matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q624: [Security Reviewer] SyncT alk: Spot the risk in Unread N+1.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/conversations/conversations.service.ts.
One query plus many per-conversation queries can scale poorly.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q625: [QA Engineer] SyncT alk: How would you test Unread N+1?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Create hundreds of conversations and measure query count and response latency.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q626: [Hiring Manager] SyncT alk: How would you improve Unread N+1 for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Batch/aggregate counts or maintain well-scoped counters when product consistency requirements allow
it.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q627: [Senior Flutter Interviewer] SyncT alk: What is Regex search, and what does this code actually do?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/conversations/conversations.repository .ts
and messages.repository .ts. Search paths use regex matching over conversation/message data.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q628: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Search paths use regex matching over conversation/message data. The design is useful because it
matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.
29</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q629: [Security Reviewer] SyncT alk: Spot the risk in Regex search.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/conversations/conversations.repository .ts
and messages.repository .ts. Unbounded or poorly indexed regex search can consume CPU and scan large collections.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q630: [QA Engineer] SyncT alk: How would you test Regex search?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Benchmark short vs long queries and inspect database query plans.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q631: [Hiring Manager] SyncT alk: How would you improve Regex search for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Cap query length, rate-limit search, use text indexes or a dedicated search service for larger datasets.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q632: [Senior Flutter Interviewer] SyncT alk: What is Upload validation, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in sync talk/sync_talk_backend/src/middleware/upload.middleware.ts. Up-
loads have size restrictions and extension-based filtering.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q633: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Uploads have size restrictions and extension-based filtering. The design is useful because it matches
the feature requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q634: [Security Reviewer] SyncT alk: Spot the risk in Upload validation.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/middleware/upload.middleware.ts. Exten-
sions can be spoofed; content may still be malicious or unsafe to serve.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q635: [QA Engineer] SyncT alk: How would you test Upload validation?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Upload renamed files, unexpected MIME values and oversized payloads.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q636: [Hiring Manager] SyncT alk: How would you improve Upload validation for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Validate MIME/content, restrict file types, isolate storage, scan where appropriate and serve with safe
headers.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q637: [Senior Flutter Interviewer] SyncT alk: What is Public upload serving, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/app.ts. An uploads directory is exposed as
static content.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q638: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> An uploads directory is exposed as static content. The design is useful because it matches the feature
requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q639: [Security Reviewer] SyncT alk: Spot the risk in Public upload serving.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/app.ts. Untrusted files become directly web
accessible and may create content-execution or privacy risks.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q640: [QA Engineer] SyncT alk: How would you test Public upload serving?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Upload HTML/SVG-like content in a controlled environment and verify it is not executed or served
with dangerous content behavior.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q641: [Hiring Manager] SyncT alk: How would you improve Public upload serving for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use controlled object storage/download paths and safe response headers; do not expose an execution
path.
30</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q642: [Senior Flutter Interviewer] SyncT alk: What is In-memory presence, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/sockets/presence.gateway .ts. Presence state
is stored in process memory.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q643: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Presence state is stored in process memory. The design is useful because it matches the feature
requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q644: [Security Reviewer] SyncT alk: Spot the risk in In-memory presence.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/sockets/presence.gateway .ts. With multiple
instances each process sees only part of the connection set.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q645: [QA Engineer] SyncT alk: How would you test In-memory presence?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Run two backend instances and connect the same user through both; verify consistent presence.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q646: [Hiring Manager] SyncT alk: How would you improve In-memory presence for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use shared TTL state or a distributed presence protocol.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q647: [Senior Flutter Interviewer] SyncT alk: What is In-memory call state, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/sockets/call.socket.ts. Active calls and user
socket maps are maintained in memory.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q648: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Active calls and user socket maps are maintained in memory. The design is useful because it matches
the feature requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q649: [Security Reviewer] SyncT alk: Spot the risk in In-memory call state.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/sockets/call.socket.ts. Scaling to multiple
instances can break cross-instance call coordination.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q650: [QA Engineer] SyncT alk: How would you test In-memory call state?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Route caller and receiver through different instances and test invite/accept/hangup.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q651: [Hiring Manager] SyncT alk: How would you improve In-memory call state for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use shared signaling metadata/distributed routing and make reconnect reconstruction explicit.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q652: [Senior Flutter Interviewer] SyncT alk: What is Call room authorization, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/sockets/call.socket.ts. Call events accept
client-supplied room/participant identifiers.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q653: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Call events accept client-supplied room/participant identifiers. The design is useful because it matches
the feature requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q654: [Security Reviewer] SyncT alk: Spot the risk in Call room authorization.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/sockets/call.socket.ts. A client should not
be able to join or manipulate arbitrary call rooms by guessing IDs.
31</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q655: [QA Engineer] SyncT alk: How would you test Call room authorization?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Attempt to join another user’s room ID from an authenticated but unauthorized socket.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q656: [Hiring Manager] SyncT alk: How would you improve Call room authorization for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Resolve allowed participants from trusted server state and verify the authenticated socket identity.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q657: [Senior Flutter Interviewer] SyncT alk: What is Duplicate call gateways, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/sockets/call.gateway .ts and call.socket.ts.
There are overlapping call signaling implementations with different event contracts.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q658: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> There are overlapping call signaling implementations with different event contracts. The design is
useful because it matches the feature requirement, but the important interview point is the boundary where the choice
stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q659: [Security Reviewer] SyncT alk: Spot the risk in Duplicate call gateways.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/sockets/call.gateway .ts and call.socket.ts.
Duplicate infrastructure creates inconsistent behavior and makes security fixes easy to apply to the wrong path.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q660: [QA Engineer] SyncT alk: How would you test Duplicate call gateways?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Trace server startup imports and assert only the intended gateway handles events.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q661: [Hiring Manager] SyncT alk: How would you improve Duplicate call gateways for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Identify the registered implementation, remove stale paths and test one canonical contract.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q662: [Senior Flutter Interviewer] SyncT alk: What is Duplicate route prefixes, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/app.ts. Routes are mounted through multiple
prefixes including /api and /api/v1 forms.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q663: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Routes are mounted through multiple prefixes including /api and /api/v1 forms. The design is useful
because it matches the feature requirement, but the important interview point is the boundary where the choice stops
being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q664: [Security Reviewer] SyncT alk: Spot the risk in Duplicate route prefixes.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/app.ts. Duplicate public surfaces increase
maintenance and can produce inconsistent middleware behavior.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q665: [QA Engineer] SyncT alk: How would you test Duplicate route prefixes?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Compare auth and error behavior across every externally reachable prefix.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q666: [Hiring Manager] SyncT alk: How would you improve Duplicate route prefixes for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Choose one canonical API versioning scheme and deprecate legacy paths with explicit middleware.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q667: [Senior Flutter Interviewer] SyncT alk: What is Legacy chat socket, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in sync talk/sync_talk_backend/src/sockets/chat.socket.ts. An older socket path
can bypass the stronger membership checks in the newer message service.
32</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q668: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> An older socket path can bypass the stronger membership checks in the newer message service. The
design is useful because it matches the feature requirement, but the important interview point is the boundary where
the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q669: [Security Reviewer] SyncT alk: Spot the risk in Legacy chat socket.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/sockets/chat.socket.ts. Duplicate business
entry points can become authorization bypasses.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q670: [QA Engineer] SyncT alk: How would you test Legacy chat socket?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Send a message through the legacy socket as a non-member and verify rejection.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q671: [Hiring Manager] SyncT alk: How would you improve Legacy chat socket for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Centralize message operations in shared services and enforce membership on every transport.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q672: [Senior Flutter Interviewer] SyncT alk: What is Refresh token storage, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/modules/auth/auth.service.ts. Refresh-
token records are stored with token material and expiry data.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q673: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Refresh-token records are stored with token material and expiry data. The design is useful because it
matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q674: [Security Reviewer] SyncT alk: Spot the risk in Refresh token storage.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/auth/auth.service.ts. A database
compromise can become a session compromise if refresh credentials are directly reusable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q675: [QA Engineer] SyncT alk: How would you test Refresh token storage?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use a stolen database fixture and verify raw refresh credentials cannot be replayed.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q676: [Hiring Manager] SyncT alk: How would you improve Refresh token storage for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Hash or otherwise protect refresh credentials, rotate them, detect reuse and revoke token families on
theft indicators.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q677: [Senior Flutter Interviewer] SyncT alk: What is Query-string token, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in sync talk/sync_talk_backend/src/middleware/auth.ts. Socket authentication
accepts a token from handshake query data in addition to auth data.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q678: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Socket authentication accepts a token from handshake query data in addition to auth data. The
design is useful because it matches the feature requirement, but the important interview point is the boundary where
the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q679: [Security Reviewer] SyncT alk: Spot the risk in Query-string token.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/middleware/auth.ts. Query strings are more
likely to leak through logs, proxies and tracing.
33</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q680: [QA Engineer] SyncT alk: How would you test Query-string token?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Inspect proxy/access logs and ensure authentication credentials are absent from URLs.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q681: [Hiring Manager] SyncT alk: How would you improve Query-string token for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Prefer Socket.IO auth handshake or safer transport and remove reusable credentials from URLs.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q682: [Senior Flutter Interviewer] SyncT alk: What is Conversation deletion, and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/modules/conversations/conversations.repository .ts.
Conversation deletion and message deletion occur as separate operations.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q683: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Conversation deletion and message deletion occur as separate operations. The design is useful because
it matches the feature requirement, but the important interview point is the boundary where the choice stops being safe
or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q684: [Security Reviewer] SyncT alk: Spot the risk in Conversation deletion.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/conversations/conversations.repository .ts.
A failure between operations can leave partial data.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q685: [QA Engineer] SyncT alk: How would you test Conversation deletion?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Force a failure during child deletion and verify the system can recover consistently.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q686: [Hiring Manager] SyncT alk: How would you improve Conversation deletion for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use a database transaction where practical or an explicit asynchronous deletion state machine for
large histories.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q687: [Senior Flutter Interviewer] SyncT alk: What is Cursor pagination, and what does this code actually</div>
  <div class="qa-a"><strong>Model Answer:</strong> do?
Model answer: Observed in sync talk/sync_talk_backend/src/modules/messages/messages.repository .ts.
Message pagination relies on creation-time ordering.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q688: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Message pagination relies on creation-time ordering. The design is useful because it matches the
feature requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q689: [Security Reviewer] SyncT alk: Spot the risk in Cursor pagination.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/messages/messages.repository .ts.
Messages can share the same timestamp, creating ambiguous page boundaries and duplicates/skips.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q690: [QA Engineer] SyncT alk: How would you test Cursor pagination?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Insert multiple messages with identical timestamps and page through the collection.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q691: [Hiring Manager] SyncT alk: How would you improve Cursor pagination for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use a composite cursor such as createdAt plus unique message ID.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q692: [Senior Flutter Interviewer] SyncT alk: What is Push delivery reliability , and what does this code</div>
  <div class="qa-a"><strong>Model Answer:</strong> actually do?
Model answer: Observed in sync talk/sync_talk_backend/src/modules/messages/messages.service.ts.
Push notification work is triggered after message creation rather than through a durable outbox.
34</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q693: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Push notification work is triggered after message creation rather than through a durable outbox. The
design is useful because it matches the feature requirement, but the important interview point is the boundary where
the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q694: [Security Reviewer] SyncT alk: Spot the risk in Push delivery reliability .</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/src/modules/messages/messages.service.ts.
A process crash can save the message but lose the notification side effect.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q695: [QA Engineer] SyncT alk: How would you test Push delivery reliability?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Simulate a process failure after message commit but before notification completion.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q696: [Hiring Manager] SyncT alk: How would you improve Push delivery reliability for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use an outbox/queue with idempotent event processing and retry.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q697: [Senior Flutter Interviewer] SyncT alk: What is T est drift, and what does this code actually do?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/tests/modules/messages.service.test.ts and
related auth tests. Some tests use older import/data shapes and extensive mocks.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q698: [Backend Interviewer] SyncT alk: Why is this implementation choice useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Some tests use older import/data shapes and extensive mocks. The design is useful because it matches
the feature requirement, but the important interview point is the boundary where the choice stops being safe or scalable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q699: [Security Reviewer] SyncT alk: Spot the risk in T est drift.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Observed in sync talk/sync_talk_backend/tests/modules/messages.service.test.ts and
related auth tests. A green suite can provide false confidence when mocks no longer represent production contracts.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q700: [QA Engineer] SyncT alk: How would you test T est drift?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Break a real repository contract and verify the test suite actually fails.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q701: [Hiring Manager] SyncT alk: How would you improve T est drift for production?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Align tests with current modules, add integration coverage for authorization and persistence, and run
the same checks in CI.
8. Project-by-Project Senior Interview Deep Dive</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q702: Multi-V endor E-Commerce: Why are buyer, vendor and admin applications separate?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Their workflows and permissions are different enough to justify focused clients while sharing backend
business rules.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q703: Multi-V endor E-Commerce: How do you keep order history correct when product data changes?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Persist transaction snapshots such as the item name and price used for the order, rather than re-reading
the current catalog value later.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q704: Multi-V endor E-Commerce: Why should final inventory changes be atomic?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Multiple users can check out the last unit simultaneously. An atomic update or transaction prevents
overselling caused by stale reads.
35</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q705: Multi-V endor E-Commerce: What is the right source of truth for checkout?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The backend should be authoritative for price, discounts, inventory, payment state and order state.
The Flutter client is the user experience layer.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q706: GrowK: Why is financial state different from normal CRUD state?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Money, identity and compliance workflows have stronger correctness and audit requirements. Partial
failure and asynchronous provider states need explicit modeling.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q707: GrowK: How would you model KYC?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use explicit states such as not started, pending, verified, rejected and retryable based on provider
semantics. The UI should reflect authoritative backend/provider state.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q708: GrowK: Why is biometric authentication not a replacement for server authorization?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Biometrics prove local user presence to the device; the backend still decides whether the account and
operation are authorized.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q709: GrowK: What is the difference between a live gold price and a trade execution price?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A displayed market price can become stale. A trade execution price must come from authoritative
transaction confirmation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q710: AutoRide: Why should ride state be a state machine?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A ride has ordered transitions such as pending, accepted, started, completed and cancelled. Explicit
valid transitions prevent contradictory updates.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q711: AutoRide: How would you prevent two drivers from accepting one ride?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use an atomic conditional transition from pending to accepted so only one request succeeds.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q712: AutoRide: Why separate live location from trip history?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Live location is high-frequency ephemeral state, while history is durable business data. They have
different retention and performance needs.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q713: AutoRide: Why should ET A and fare be treated differently?</div>
  <div class="qa-a"><strong>Model Answer:</strong> ETA can be a client-facing estimate based on current conditions; fare is a financial business value that
should come from authoritative server rules.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q714: SkyHopper: Why use Flame?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Flame provides a game loop, components, collision handling and rendering patterns suited to contin-
uously updating 2D gameplay.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q715: SkyHopper: Why use Riverpod in a game?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It can manage app-level state such as settings, high score and game mode while Flame owns the
continuously updating game world.</div>
</div>

---

## Part 4: Outside-World Enterprise Messaging Questions (WhatsApp, Slack Scale)

<div class="qa-card">
  <div class="qa-q">Q-EXT-01: How do you scale a WebSocket chat architecture to support 10 million concurrent active connections?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Individual Node.js Socket.IO instances cannot hold 10M TCP sockets due to file descriptor limits. We deploy an horizontally scaled cluster of Socket.IO gateway pods behind AWS Network Load Balancers (NLB) with sticky sessions. Nodes are connected via Redis Streams / Pub-Sub clusters. When User A (on Node 1) messages User B (on Node 14), Node 1 publishes the packet to Redis channel `user:{userId}`, and Node 14 consumes the event and delivers it to User B's active socket.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q-EXT-02: How do you ensure End-to-End Encryption (E2EE) in mobile messaging?</div>
  <div class="qa-a"><strong>Model Answer:</strong> We implement the Signal Protocol (Double Ratchet Algorithm). Each device generates an identity key, signed prekey, and a pool of one-time prekeys published to the server. When initiating a chat, Client A fetches Client B's prekey bundle, establishes a shared master secret via X3DH (Extended Triple Diffie-Hellman), and derives ephemeral symmetric message keys that rotate after every single message, guaranteeing forward secrecy and break-in recovery.</div>
</div>
