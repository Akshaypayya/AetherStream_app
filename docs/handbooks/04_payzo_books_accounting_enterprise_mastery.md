# Payzo Books: Enterprise Cloud Accounting & ERP Platform
## Comprehensive Technical Handbook & Interview Mastery Guide (Questions 791-990)
**Candidate:** Akshay Chandran P  
**Codebase Scale:** 792 Dart Source Files | 502 View/UI Files | 201 Data-Layer Files | 86 Utilities  
**Core Technologies:** Flutter, Dart, Riverpod, SQLite / Drift, ZATCA Phase 2 E-Invoicing, Android MediaStore Bridge, Figma Tokens

---

## Part 1: Comprehensive Working Explanation (Beginner to Extreme Architecture)

### 1.1 Beginner Level: What is Payzo Books?
Payzo Books is an enterprise-grade cloud accounting and business management platform designed for small-to-medium enterprises (SMEs) and accounting professionals. Unlike simple expense-tracking applications, Payzo Books implements standard double-entry bookkeeping workflows:
- **Sales & Invoicing:** Generating tax-compliant invoices, credit notes, and quotations with real-time tax computation.
- **Purchases & Vendor Bills:** Managing accounts payable, vendor statements, and expense categorizations.
- **Inventory & Stock Management:** Real-time stock quantity adjustments, item pricing, unit-of-measure conversions, and low-stock alerts.
- **Tax Compliance (Saudi ZATCA):** Fully compliant with Saudi Arabia ZATCA Phase 2 e-invoicing standards, generating cryptographic invoice stamps, chained SHA-256 hashes, and Base64-encoded TLV QR codes.
- **Reporting & Ledgers:** Profit and Loss (P&L), Balance Sheets, General Ledgers, and VAT return audits exported directly to PDF and Excel.

---

### 1.2 Extreme Architecture: Clean Architecture & 792 Dart Files Organization
Managing 792 source files in Flutter requires strict separation of concerns to avoid circular dependencies and state leakage. Payzo Books follows a **Feature-First Clean Architecture**:

```
lib/
├── core/                           # 86 files: Cross-cutting infrastructure
│   ├── database/                   # Drift (SQLite) database, migrations, DAOs
│   ├── network/                    # Dio HTTP client, SSL pinning, retry interceptors
│   ├── tokens/                     # Figma design tokens (typography, color, spacing)
│   ├── utils/                      # Currency formatting, date parsers, math helpers
│   └── errors/                     # Failure models, Result<T> monads
├── data/                           # 201 files: Persistence, API clients, Repositories
│   ├── models/                     # DTOs with Freezed / JSON serialization
│   ├── repositories/               # Concrete repository implementations
│   └── services/                   # MediaStore SAF bridge, PDF/Excel generators
└── views/                          # 502 files: UI Screens, Widgets & State Notifiers
    ├── invoices/                   # Invoice list, creation, line item tables, preview
    ├── bills/                      # Vendor bill workflows, expense entry
    ├── inventory/                  # Items catalogue, stock adjustments
    ├── reports/                    # Financial ledger views, tax summaries
    └── settings/                   # Organization profiles, tax rates, currencies
```

#### Why Riverpod for 792 Files?
1. **Compile-Time Dependency Safety:** In an enterprise codebase of this scale, `ProviderNotFoundException` at runtime is intolerable. Riverpod providers are top-level final declarations verified at compile-time.
2. **Fine-Grained Selectors:** In an invoice editor with 50 line items, editing line item #12 must NOT rebuild line item #1 through #11 or the entire table. `ref.watch(invoiceProvider.select((inv) => inv.items[index]))` guarantees that only the modified row re-renders.
3. **Automatic Lifecycle Disposal (`autoDispose`):** When leaving an invoice creation screen, `AutoDisposeNotifierProvider` automatically cleans up line-item controllers and cached totals from memory, preventing memory bloat.

---

### 1.3 Financial Arithmetic: Eliminating Floating-Point Errors
Standard IEEE 754 floating-point math (`double` in Dart) cannot represent decimal fractions like `0.1` precisely in binary (`0.1 + 0.2 = 0.30000000000000004`). In enterprise accounting, a 1-cent discrepancy can fail a government tax audit.  
**Payzo Books Solution:**
All monetary amounts are stored as integer **Minor Currency Units** (cents / fils / halalas) or processed via the `decimal` package:
```dart
final subtotal = Decimal.parse("100.10");
final vatRate = Decimal.parse("0.15");
final vatAmount = (subtotal * vatRate).toDecimal(scaleOnInfinitePrecision: 2);
final grandTotal = subtotal + vatAmount;
```

---

### 1.4 Saudi ZATCA Phase 2 E-Invoicing Compliance
For Saudi Arabia compliance, every B2B and B2C invoice requires:
1. **Cryptographic Stamp:** An ECDSA secp256k1 digital signature over the invoice UBL 2.1 XML content.
2. **Invoice Hash:** SHA-256 hash of the normalized XML document chained to the previous invoice's hash (forming a tamper-evident blockchain-style ledger).
3. **ZATCA QR Code:** A Base64-encoded TLV (Tag-Length-Value) data structure containing Seller Name, VAT Number, Timestamp, Total with VAT, and Cryptographic Stamp.

---

### 1.5 Android MediaStore & Scoped Storage Bridge
In modern Android (API 29+ Android 10, 11, 12, 13, 14), accessing raw file paths via `dart:io` in public storage throws `PermissionDeniedException`.  
**How Payzo Books Exports Invoices & Excel Reports:**
- Implemented a custom Platform Channel invoking Android's **`MediaStore` API**:
  1. PDF/Excel is generated in the app's sandboxed cache directory.
  2. Kotlin platform channel creates a new entry in `MediaStore.Downloads.EXTERNAL_CONTENT_URI` with `IS_PENDING = 1`.
  3. Writes bytes into the opened `ContentResolver.openOutputStream()`.
  4. Flushes and marks `IS_PENDING = 0`, making the invoice immediately visible in the device's Downloads folder without requiring dangerous storage permissions.

---

## Part 2: Code Audit Findings, Architectural Risks & Production Fixes

### Finding 1: Memory Leaks During Large Ledger PDF Generation
- **Problem:** Generating a 100-page transaction report with embedded company logos allocated over 400MB of RAM, causing low-memory crashes on budget devices.
- **Production Fix:** Moved PDF generation into a dedicated background Isolate worker using streaming byte sinks, writing directly to disk in chunks rather than assembling the entire document in heap memory.

### Finding 2: Offline SQLite Sync Conflicts
- **Problem:** When two accountants edited the same invoice concurrently (one offline on mobile, one online on the web portal), offline sync simply overwrote cloud changes.
- **Production Fix:** Introduced an optimistic concurrency control column `version_id`. If `client.version_id != server.version_id`, the sync engine flags a conflict and presents a side-by-side reconciliation screen.

---

## Part 3: Exhaustive Interview Q&A Section (Questions 791 - 990)
*The following 200 questions and model answers represent the complete, unabridged interview preparation handbook for Payzo Books and Figma design token systems.*


Akshay Chandran P | Payzo Books + Figma Interview Q&A
AKSHAY CHANDRAN P
PAYZO BOOKS + FIGMA
INTERVIEW Q&A HANDBOOK
Continuation volume for source-grounded interview preparation
Questions 791-990
Prepared from the supplied Payzo Books source ZIP plus documented Figma/UI implementation knowledge from the existing resume 
and prior interview material.
SOURCE & INTERVIEW RULES
• Payzo Books questions are grounded in the attached project ZIP. The supplied source contains 792 Dart files, with feature-heavy 
view, data/repository, model and utility layers.
• Source landmarks used include invoice/bill controllers and repositories, expense controllers, pagination providers, BaseApiService, 
notification services, import/export repositories, download models and the Android MainActivity MethodChannel.
• Figma questions are grounded in your established resume positioning and earlier interview material around Figma-to-Flutter 
implementation. No separate Figma design file was audited here.
• Use first-person statements only for work you can personally defend. The ZIP proves what the code contains; it does not by itself 
prove authorship of every line.
• When a question exposes a code weakness, answer honestly: current behavior -> risk -> production fix -> test.
QUICK CONTENTS
• Part I - Payzo Books (Questions 791-944)
• Part II - Figma / UI-UX (Questions 945-976)
• Part III - Combined Payzo + Figma Scenarios (Questions 977-990)
• Recommended answer patterns: definition -> project example -> edge case -> trade-off; bug -> evidence -> root cause -> fix -> 
regression test.
PART I - PAYZO BOOKS
1. Payzo Books - Project Overview & Ownership


<div class="qa-card">
  <div class="qa-q">Q791: Give me a 30-second overview of Payzo Books.</div>
  <div class="qa-a"><strong>Model Answer:</strong> Payzo Books is a Flutter-based accounting and business management application. I worked on a large 
feature set covering invoices, bills, expenses, customers, vendors, products, taxes, payments, purchase workflows, 
notifications, imports/exports and dashboard views. The Flutter client uses Riverpod-based state management and a 
repository/service structure to communicate with Node.js REST services and MongoDB-backed data.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q792: What was your role in Payzo Books?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I worked primarily on the Flutter application: implementing business workflows, reusable UI, state 
management, API integration, validation, calculations, debugging and workflow improvements. I also worked at the 
integration boundary with the Node.js REST services, because many accounting features depend on request payloads, 
response models and backend validation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q793: Why is Payzo Books technically interesting compared with a basic CRUD application?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The complexity comes from business rules and state transitions, not just forms. Invoices and bills 
involve customers or vendors, line items, quantities, prices, taxes, discounts, currencies, accounts, dates, approvals and 
document states. The app also includes pagination, search, imports/exports, notifications, file handling and platform-
specific download behavior.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q794: What major business areas are present in Payzo Books?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The codebase contains invoice and recurring-invoice flows, bill and recurring-bill flows, expenses, 
customers, vendors, products, taxes, quotations, RFQs, purchase orders, journal entries, payment-due data, 
cash-flow/dashboard data, pending requests, notifications and document download/import/export features.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q795: How large is the Payzo Books codebase?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The supplied ZIP contains 792 Dart source files. The structure is dominated by 502 view/UI files, 201 
data-layer files and 86 utility files. That size matters in an interview because it demonstrates that the application 
involves feature coordination and architectural boundaries rather than a small single-screen project.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q796: How is the Payzo project organized at a high level?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The code has a clear separation between views, data models, repositories, services, providers/notifiers 
and utilities. Feature areas are grouped under the view tree, while reusable data access and models are organized 
under data/repository, data/models and data/services. Utilities cover routing, scaling, connectivity and common 
widgets.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q797: Why do you use a repository layer?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The repository acts as a boundary between UI/state logic and HTTP details. A feature can ask a 
repository for an invoice list, customer details or bill data without embedding URL construction, headers and JSON 
mapping inside every widget. That improves separation of concerns and makes the data source easier to replace or test.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q798: What is the role of BaseApiService?</div>
  <div class="qa-a"><strong>Model Answer:</strong> BaseApiService centralizes common HTTP behavior such as GET, POST, PUT, DELETE, multipart and file-
upload operations, header construction, token handling and JSON mapping. Feature repositories then stay focused on 
the business endpoint and model type instead of duplicating transport logic.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q799: Why are models separated from repositories?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A model represents the shape of data, while a repository represents how that data is obtained or 
changed. Keeping them separate means serialization concerns do not leak into UI code and API access does not become 
tightly coupled to widget structure.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q800: What is the role of the import_data.dart file?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It functions as a large barrel-style import surface for commonly used Flutter widgets, utilities, routes, 
providers and project components. It can reduce repetitive import statements, but the trade-off is that it can make 
dependencies less explicit and can increase accidental coupling.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q801: What would you explain first if an interviewer says 'walk me through Payzo'?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would start with one representative workflow, such as creating an invoice. I would explain the screen, 
the Riverpod state that holds the form, how reference/master data is fetched, how validation and calculations happen, 
how the payload is constructed, how the repository/service sends it, and how success or backend validation errors 
return to the UI.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q802: Which project feature best demonstrates business-logic reasoning?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Invoice and bill creation are strong examples because the payload depends on multiple interacting 
rules: line items, tax type, discount mode, dates, accounts, currency and totals. The implementation explicitly builds 
line-item data and calculates subtotals, tax and global discounts before constructing the request model.
2. Payzo Books - Flutter Architecture, Riverpod & State Management</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q803: Why is Riverpod important in Payzo Books?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The application has many independent async data sources and form states. Riverpod provides providers 
for repository results, lists, selections, form fields and derived values, allowing screens to react to focused pieces of state 
rather than carrying everything in local widget state.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q804: Where do you see different kinds of Riverpod state in Payzo?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The code uses FutureProvider for fetched data, Provider for derived or injected dependencies, 
StateProvider for simple values, and StateNotifierProvider for richer state machines such as pagination and form-
oriented controllers.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q805: What is an example of derived state in Payzo?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The items providers derive filtered lists such as active items, stockable items or items by usage type 
from the shared item model. The derived provider watches the source state and exposes only the subset a feature 
needs.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q806: How would you explain ref.read versus ref.watch?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I use ref.watch when the consumer should rebuild or react to state changes, and ref.read when I only 
need the current value or a notifier/service without subscribing. Choosing correctly limits unnecessary rebuilds and 
keeps intent clear.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q807: Why use StateNotifierProvider for pagination?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Pagination has several related state values: current page, accumulated records, loading status, search 
text, total count, error state and whether another page exists. Keeping those fields together makes the 
fetch/append/search transitions explicit.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q808: How does Payzo pagination work for lists such as bills?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The bill pagination notifier uses a fixed page size of 15, fetches page zero initially, then increments the 
page for load-more requests. It keeps an accumulated list, total count and hasNextPage flag, and it deduplicates 
appended records by bill ID.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q809: How is search handled in the pagination notifier?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A search request fetches page zero with the query and replaces the current result set. The state stores 
the active query and the implementation checks that the returned query still matches the latest state before committing 
the result, which helps prevent stale search responses from overwriting newer results.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q810: Why is the query check important?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Network responses can return out of order. If a user types one query and then another, the first request 
may finish after the second. Comparing the returned query with the current state is a simple guard against displaying 
an outdated result.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q811: What does _isLoadingMore protect against?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It prevents overlapping load-more operations. Without a separate guard, repeated scroll callbacks could 
issue multiple requests for the same next page and create duplicate or inconsistent state transitions.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q812: Why are duplicate IDs removed when appending?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Pagination endpoints can occasionally return overlapping records because of server-side changes, 
retries or boundary conditions. The notifier builds a set of existing IDs and only adds unseen records, reducing 
duplicate rows in the accumulated UI state.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q813: What is a reasonable improvement to this pagination design?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would standardize the pattern across bills, customers, products and expenses using a reusable 
pagination state abstraction, add explicit request cancellation or debouncing for search, cap page size on the server, 
and test race conditions, empty pages, refresh and concurrent load-more calls.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q814: What would you watch for in Riverpod rebuild performance?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would inspect provider granularity and use select or narrower providers where necessary. A form 
with dozens of fields should not cause an entire screen tree to rebuild for one field change. Profiling should guide 
where provider decomposition actually matters.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q815: Why is feature-level provider organization useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Invoice, bill, expense, customer and product flows have different state lifecycles. Keeping their 
providers close to the feature reduces global coupling and makes it easier to understand who owns and resets a piece of 
state.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q816: How does Payzo reset state after successful invoice creation?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The invoice submission flow invalidates the invoice data provider, resets the invoice form providers, 
updates the navigation state and then returns to the appropriate root screen. The important idea is to invalidate stale 
list data and clear the transient form state together.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q817: What is an example of provider-based selection logic?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The item provider exposes selected-item state and derived collections such as active items and stockable 
items. A feature can select an item once and let dependent providers react to that selection without duplicating filtering 
logic.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q818: What is one architectural weakness you would look for in a large Riverpod application?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Too much cross-feature state can create hidden dependencies. I would look for providers that directly 
mutate unrelated feature state, broad barrel imports, UI controllers with excessive business logic and duplicated 
provider patterns that should be consolidated.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q819: How would you test a StateNotifier such as bill pagination?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would test initial load, successful append, empty next page, duplicate records, search, stale search 
response, refresh, network error and recovery. The goal is to verify state transitions independently from widget 
rendering.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q820: What is your preferred mental model for async UI state?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Think in explicit states: loading, success, empty, error and retryable failure, plus feature-specific 
transitional states such as submitting or searching. The UI should reflect the authoritative state rather than infer 
success from a button callback.
3. Payzo Books - Networking, Authentication & API Design</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q821: How is the API layer structured?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Feature repositories call a shared BaseApiService. The base service exposes generic methods that accept 
a URL, request body where required, and a fromJson function so the response can be converted into the correct model 
type.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q822: Why use a generic fromJson callback?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It keeps the HTTP layer independent of individual models. The transport method can return T while the 
feature decides how the JSON should be converted, which avoids hard-coding accounting entities into the base client.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q823: What HTTP methods are supported by BaseApiService?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The service includes GET list, GET object, POST, PUT, DELETE, PATCH and several multipart/file-related 
methods. This allows invoice, bill, customer, vendor, expense and document workflows to share consistent transport 
behavior.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q824: How are authorization headers built?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The base service reads the access token from shared-preference storage and builds an Authorization 
header along with common application headers such as transaction and company identifiers.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q825: How does the current code handle a 401 response?</div>
  <div class="qa-a"><strong>Model Answer:</strong> For several generic API methods, a 401 triggers a login flow using stored login credentials, then the 
request is retried once with the new access token. In a production redesign, I would prefer a refresh-token flow with 
single-flight refresh and would avoid storing reusable passwords.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q826: Why should token refresh be single-flight?</div>
  <div class="qa-a"><strong>Model Answer:</strong> If several requests receive 401 at the same time, starting multiple refresh operations can race and 
overwrite tokens. A single-flight mechanism lets one refresh operation run while other requests await the same future.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q827: What is a key security concern with current logging?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Some code paths log request/response information and status details. Even when authorization headers 
are partially redacted, sensitive response bodies can still contain confidential business data. Production logs should be 
structured, minimal and privacy-aware.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q828: Why should hardcoded company identifiers and transaction values be reviewed?</div>
  <div class="qa-a"><strong>Model Answer:</strong> They may be environment-specific or incorrect for another tenant or deployment. Configuration and 
tenant context should come from validated runtime configuration or authenticated server context rather than magic 
constants.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q829: What is the advantage of returning typed models from repositories?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The UI can work with Dart objects instead of loosely typed dynamic maps. That improves autocomplete, 
compile-time checks and separation between API transport format and presentation code.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q830: How do you handle API validation errors in invoice creation?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The submission controller checks the response type and maps backend validation details back to the 
relevant field error providers, such as invoice number or order number. That creates a direct feedback path from 
server validation to the UI.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q831: Why is backend validation still necessary if the Flutter app validates fields?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Client validation improves UX but cannot be trusted for correctness or security. The API remains 
authoritative for unique references, business rules, permissions and financial calculations.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q832: What is the benefit of having multipart helpers?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Invoices, expenses and document workflows may need to send files alongside structured form data. 
Multipart helpers centralize the transport pattern so feature code can focus on which file and metadata are being sent.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q833: Why is Base64 file encoding sometimes used, and what is the trade-off?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Base64 allows binary content to travel inside JSON, which can simplify certain APIs, but it increases 
payload size and memory usage. Multipart upload is usually preferable for larger files when the backend supports it.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q834: How would you make the API layer more testable?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Inject the HTTP client behind a narrow interface, isolate URL building from request execution, inject 
token providers, use typed error objects and write repository tests against mocked transport responses.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q835: How would you design error mapping?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Map transport failures, authentication failures, server validation errors and domain-level errors into 
explicit application error types. The UI should consume those types rather than parsing arbitrary exception strings.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q836: What is wrong with treating every non-200 response as a generic exception?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It loses useful semantics. A 400 validation error, 401 authentication error, 403 authorization error, 404 
missing record and 500 server failure need different UI and retry behavior.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q837: How does an endpoint repository typically expose data to Riverpod?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A repository provider is created first, then a FutureProvider or family provider reads the repository and 
awaits its method. This keeps dependency construction separate from the widget tree.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q838: Why use FutureProvider.family for details pages?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A family lets the same provider definition fetch different entities based on an ID, such as invoice details. 
Riverpod can then cache and manage the async state separately for each identifier.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q839: What would you inspect when an API works in Postman but fails in Flutter?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would compare URL, HTTP method, headers, authorization, content type, body shape, encoding, 
date/number formatting, multipart fields and environment URLs. Then I would inspect the raw response status and 
body in a safe development log.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q840: What is a good interview example of cross-layer debugging in Payzo?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Invoice creation is a strong example: trace the value from the widget field into Riverpod state, into the 
payload builder, through the HTTP request, into the backend validation response, and back into field-specific error 
state.
4. Payzo Books - Invoice, Bill & Accounting Workflows</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q841: Walk me through creating an invoice in Payzo.</div>
  <div class="qa-a"><strong>Model Answer:</strong> When the invoice screen opens, the controller prefetches master data such as customers, branches, bank 
accounts, items and currency, and can also fetch invoice data used for reference generation. The user fills header and 
line-item fields, validation runs, line items are converted into a request payload, totals are calculated, the invoice 
service submits the data, and success or validation errors are reflected back into the UI.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q842: How does invoice initialization reduce perceived latency?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The controller starts several independent data fetches together with Future.wait. That allows customer, 
branch, bank account, item and currency data to load concurrently instead of strictly serially.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q843: How are default invoice values populated?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The initialization code can select the first branch and first currency, set the invoice date to today, and set 
the due date to the corresponding next-month date. The values are written into the relevant Riverpod providers so the 
UI starts with usable defaults.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q844: How is automatic invoice reference generation implemented?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The controller can fetch existing invoices, find numeric suffixes matching the expected reference 
pattern, determine the maximum numeric value, increment it, pad it to five digits and build the next reference. Users 
can also choose manual reference entry.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q845: What is the limitation of client-side reference generation?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Two clients can calculate the same next value concurrently. The server must still enforce uniqueness. A 
safer architecture would use an authoritative server-side sequence or uniqueness check and handle conflict responses 
gracefully.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q846: What validation happens for invoice references?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The code requires a non-empty value, allows letters, digits, hyphens and slashes, and checks the fetched 
invoice list for duplicates. Backend validation remains necessary because another user can create the same reference 
between the client check and submission.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q847: How is invoice date validated?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The controller parses the date, rejects invalid values and prevents an invoice date from being in the 
future. Error state is stored in a dedicated provider so the UI can show field-level feedback.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q848: How is due date validated?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The due date is parsed and checked so it cannot occur before the invoice date. This is a domain rule that 
should also be enforced by the backend.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q849: How are invoice line items represented in state?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Each line is tracked through item-specific providers keyed by an item or line identifier. The submission 
controller reads item ID, name, quantity, unit price, unit, tax and discount values for each active line and builds the 
final payload.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q850: How is an invoice line subtotal calculated?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The base amount is quantity multiplied by unit price. If item-level discount is active, the code applies 
either a percentage or fixed amount, caps the discount at the base amount, then rounds the net line subtotal.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q851: How is tax calculated on invoice lines?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The code resolves an effective tax rate, multiplies the discounted line subtotal by that rate, rounds the 
result and adds it to the accumulated tax total. The request also carries tax metadata such as tax ID and tax type.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q852: How are global and item-level discounts distinguished?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The discount state has a mode/level. Item-level mode places discounts on individual lines, while global 
mode uses an accumulated discount amount in the header. The payload identifies the chosen method so the backend 
can interpret it correctly.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q853: Why are discount percentage and amount fields normalized to numeric zero instead of null?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The implementation was adjusted so a discount field is represented consistently as 0.0 when that 
discount type is not active. That avoids ambiguity for the API and matches the expected request shape.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q854: Why is rounding important in accounting workflows?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Money calculations can accumulate floating-point precision differences. The code rounds line and total 
values to two decimal places at controlled boundaries. In a stronger production design, monetary arithmetic should use 
decimal-safe representations or a backend-authoritative calculation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q855: How would you test invoice totals?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would test no tax/no discount, fixed discount, percentage discount, tax-inclusive, tax-exclusive, 
multiple lines, zero or invalid quantities, discount greater than subtotal, rounding boundaries and reconciliation 
between displayed and submitted totals.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q856: What is the difference between display totals and authoritative totals?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The Flutter client can calculate totals for immediate UX, but the backend should validate or recalculate 
the authoritative accounting result. The client must not be the security boundary for financial amounts.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q857: What happens after successful invoice creation?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The code obtains the generated invoice number from the response where available, shows a success 
message, invalidates invoice list data, resets form providers, updates navigation state and returns to the root screen.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q858: What happens when the backend returns invoice validation errors?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The controller iterates through validation details and maps known fields such as invoice number and 
order number into their corresponding error providers, allowing the form to present the backend reason at the right 
field.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q859: How does bill creation differ conceptually from invoice creation?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A bill uses vendor-oriented data and bill-specific fields but follows a similar pattern: validate 
authentication and required header fields, gather line-item state, calculate tax/discount values, build a typed payload 
and submit it through the service/repository boundary.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q860: What validation is visible in bill submission?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The bill submission controller checks authentication, vendor selection, bill reference, text field 
length/pattern constraints, optional order number formatting and the presence of at least one valid line item before 
submitting.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q861: How do bill line items handle discount types?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The payload builder can represent discount as fixed or percentage and carries both amount and 
percentage fields. The implementation normalizes the non-active representation so the backend receives explicit 
numeric values.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q862: What is a good domain-driven way to improve invoice/bill form logic?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Move calculation and validation into testable domain services or pure functions, keep Riverpod notifiers 
focused on orchestration, and have one canonical money/tax model so invoice and bill calculations cannot drift 
independently.
5. Payzo Books - Expenses, Tax, VAT & Financial Accuracy</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q863: Walk me through the expense calculation flow.</div>
  <div class="qa-a"><strong>Model Answer:</strong> The expense controller reads amount and selected tax information from providers, determines the 
applicable tax rate, distinguishes taxable/non-taxable or exempt cases, and calculates subtotal, tax amount and total. 
For inclusive tax, it derives the pre-tax base from total/(1+rate) and treats the difference as tax.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q864: How is an inclusive tax amount calculated?</div>
  <div class="qa-a"><strong>Model Answer:</strong> For a tax-inclusive entered amount, the code calculates base amount as entered amount divided by one 
plus the tax rate fraction. It then rounds the base, subtracts it from the total to derive tax, and stores the resulting 
subtotal, tax and total in providers.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q865: Why is tax rate resolution defensive?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Tax data can come from several model shapes or APIs. The helper first checks known model types and 
likely fields, then falls back to map-based keys. The broader lesson is to normalize external data once instead of 
spreading dynamic parsing throughout the UI.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q866: How are non-taxable and exempt expenses handled?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The calculation forces the tax rate to zero for configured non-taxable/exempt states and for zero-rated 
selections. That prevents a normal rate from being accidentally applied when the business rule says there is no tax.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q867: What is a major risk when calculating money with double?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Double is a binary floating-point representation and can introduce precision artifacts. For accounting 
systems, exact decimal or integer minor-unit arithmetic is safer for authoritative calculations.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q868: How would you improve the expense calculation code?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Create pure calculation functions with explicit inputs and outputs, use a decimal-safe money 
representation, centralize tax-rule interpretation, add table-driven tests and keep UI state updates outside the 
calculation engine.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q869: What expense workflow features are present?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The code contains expense creation and editing, expense account selection, paid-through account 
selection, vendor/customer/branch selection, currency selection, tax selection, document attachments, import/export 
and pagination/search-related infrastructure.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q870: What is the purpose of paid-through account selection?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It identifies the account from which the expense is considered paid. In an accounting product, that 
relationship matters because the transaction affects both the expense side and the cash/bank/other account side.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q871: Why is VAT terminology important in Payzo?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A recent code change explicitly updates tax labels to VAT and refines tax parsing/input formatting and 
workflow checks. That shows how product terminology can be tied directly to domain rules and UI validation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q872: What is a workflow-limit check?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It is a business rule that prevents a transaction from proceeding when a configured workflow or 
approval limit is exceeded. The important engineering point is to keep such rules visible, testable and aligned between 
UI and backend.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q873: How would you prevent accounting calculation regressions?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use deterministic unit tests for calculations, fixtures for representative tax and discount combinations, 
snapshot tests for payloads, and backend contract tests for the authoritative API rules.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q874: What is an invoice state model?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The exact states depend on the product, but conceptually a document may move through draft, 
submitted, approved/finalized, paid/partially paid, overdue, rejected or cancelled. Transitions should be controlled by 
business rules rather than arbitrary UI actions.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q875: How should money formatting differ from money calculation?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Formatting is presentation: currency symbol, locale and decimal display. Calculation is business logic. 
The underlying value should keep exact precision until the UI boundary converts it into a display string.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q876: What is the risk of recalculating totals in multiple screens?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Different screens can drift. I would centralize the rule in one domain service or calculator and have 
invoice preview, edit, submit and detail screens consume the same normalized result.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q877: How would you test an expense with inclusive VAT?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Test zero tax, normal tax, exempt tax, zero-rated tax, decimal boundaries, rounding, empty amount, 
very large amounts, invalid text and switching tax selection after entering the amount.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q878: How does Payzo handle tax reference data?</div>
  <div class="qa-a"><strong>Model Answer:</strong> There are separate repositories/providers for all tax lists, associated taxes and specific tax data. The 
invoice and expense flows consume these providers to populate selectors and resolve IDs/rates.
6. Payzo Books - Customers, Vendors, Products & Procurement</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q879: What master data does invoice creation depend on?</div>
  <div class="qa-a"><strong>Model Answer:</strong> At minimum, customer, branch, bank account, item, currency, tax and unit information may be needed. 
Payzo has dedicated repositories/providers for these sources so the form can use selected IDs and display values.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q880: How are customers and vendors handled architecturally?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The code provides listing, detail, update and delete repositories plus dedicated 
views/components/providers. Pagination notifiers maintain the list state while feature screens handle detail and 
editing.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q881: What is a good pattern for a customer details screen?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use a family provider keyed by customer ID to fetch detail data, keep editing state local or feature-
scoped, and invalidate the relevant list/detail providers after a successful update.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q882: Why use a separate product pagination provider?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Product lists can be large, so the UI loads records in pages rather than all at once. The provider also 
supports search, refresh, duplicate-ID protection and selection synchronization.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q883: What is the point of item-derived providers?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Selectors often need filtered subsets such as active items or stockable items. Derived providers keep 
these business filters in one place and allow every consumer to share the same rule.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q884: What does a repository such as GetBankAccountRepository provide?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It exposes operations such as fetching all bank accounts, fetching accounts for a specific vendor, getting 
supported banks and creating a bank account. This keeps bank-specific API logic outside invoice widgets.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q885: Why might bank accounts be fetched by vendor ID?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The available bank accounts can depend on the selected party. Parameterized providers allow the UI to 
re-fetch or react when the vendor context changes.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q886: How do purchase orders, RFQs and quotations fit the same architecture?</div>
  <div class="qa-a"><strong>Model Answer:</strong> They follow the same broad pattern: list/detail models, repositories for data access, providers for async 
state, dedicated screens/components and document download helpers. That consistency is useful for a product with 
many document types.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q887: What would you watch for when sharing components across invoice and bill flows?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Shared components should abstract stable behavior without hiding domain differences. Invoice and bill 
have different parties and fields, so forcing everything into one giant widget can create conditional complexity.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q888: How do you avoid monolithic forms?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Break the form into field groups and reusable widgets, keep field-specific state localized where practical, 
move calculations to pure services, and keep submission orchestration in a controller rather than every widget making 
its own API call.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q889: How would you debug an item selector that shows stale data?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Check provider invalidation, cached FutureProvider state, item ID/name mapping, whether the screen 
uses watch or read appropriately, and whether a previous async response is overwriting the latest selection.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q890: What is a useful code-review question for master-data modules?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Ask whether the list, search, detail and update flows agree on IDs, nullability, loading/error states and 
invalidation strategy. Inconsistent contracts create subtle bugs across dependent screens.

Akshay Chandran P | Payzo Books + Figma Interview Q&A
7. Payzo Books - Pagination, Search, Caching & UI Performance</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q891: Why does Payzo use a page size of 15?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The pagination notifiers define a fixed page size of 15. It is a practical UI/network trade-off for list 
screens, although the best value should be validated using actual response sizes, latency and device performance.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q892: How does the product provider decide whether another page exists?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It compares accumulated or current record counts with the total count returned by the API and also 
checks whether the response contains usable records. This avoids continuing to request pages after the dataset is 
exhausted.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q893: Why is local filtering sometimes used on top of server pagination?</div>
  <div class="qa-a"><strong>Model Answer:</strong> For already-loaded records, local filtering can make simple in-memory searches fast. But if users expect 
global search across the whole dataset, the server must receive the query because only the loaded page is otherwise 
available.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q894: How would you improve search UX?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Debounce keystrokes, cancel or ignore stale requests, show a clear searching state, preserve empty/error 
distinction, and avoid resetting the entire screen until the new query result is ready.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q895: What does refresh need to do besides fetch page zero?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It should clear the active search state when appropriate, reset accumulated records and page counters, 
invalidate cached providers if necessary, clear selection state where relevant, and then fetch a fresh first page.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q896: Why can pagination still show stale data after refresh?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A prior request may finish after the refresh. Without request identity or cancellation, that response can 
overwrite current state. A monotonically increasing request token or cancellation strategy makes refresh deterministic.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q897: What are good list performance practices for Payzo?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use lazy lists, avoid rebuilding every row when one row changes, keep network calls out of build, reuse 
item widgets, paginate large datasets, cache stable master data and profile expensive animations or layout work.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q898: How would you diagnose a slow invoice screen?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Measure the time spent fetching master data, building the form, parsing models, recalculating totals, 
rendering large widgets and loading assets. Then optimize the measured bottleneck instead of changing architecture 
blindly.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q899: What is a good candidate for memoization or derived providers?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Stable lookup data such as active items or filtered item collections can be derived once and reused 
through providers. The key is to avoid expensive transformation work inside build methods.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q900: How would you prevent excessive provider notifications on a multi-line invoice?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use line-specific provider families, keep aggregate totals in a focused notifier, avoid watching unrelated 
state from each line widget and update totals only when relevant numeric fields change.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q901: How would you handle an API returning 200 records for a page size of 15?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would verify the backend contract, inspect whether the client is passing the expected parameter 
names, and make the page-size assumption explicit. The UI should trust the API contract rather than infer pagination 
semantics from one response.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q902: What is the difference between cache invalidation and state reset?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Invalidation tells a data provider that its fetched value is stale and should be recomputed. State reset 
clears transient UI/form state. They often need to happen together after a mutation, but they solve different problems.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q903: What would you use profiling for in Payzo?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Flutter DevTools can help measure rebuilds, frame time, CPU work, memory and network-related 
behavior. I would profile list scrolling, invoice form edits and large detail/preview screens because those are likely to 
have meaningful UI work.
8. Payzo Books - Import, Export, Files, PDF & Native Android Bridge</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q904: What import/export capabilities exist in Payzo?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The code contains Excel-based expense import/export workflows, sample-file download, file validation, 
import processing and export APIs. Similar download helpers exist for invoices, bills, quotes, RFQs and orders.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q905: How does expense import work conceptually?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The repository exposes a sample download, file validation endpoint, import endpoint and export 
functions. The import flow passes the uploaded file along with configuration and column-mapping data, allowing the 
backend to validate the file before committing the import.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q906: Why validate an import file before importing it?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Validation separates data-quality feedback from the commit step. The user can correct mapping or 
invalid rows before the server writes business transactions, reducing partial or corrupted imports.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q907: What is an import mapping?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It connects spreadsheet columns to expected accounting fields. Keeping the mapping explicit lets the 
backend validate the input against the intended schema rather than guessing column meaning.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q908: How does Payzo download generated files on Android?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The Flutter code uses a MethodChannel named payzo_books/downloads and invokes 
saveFileToDownloads. The Android MainActivity receives the bytes and metadata, writes them through MediaStore 
Downloads and marks the file as no longer pending.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q909: Why use a MethodChannel instead of only Dart code?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Some platform services are naturally native. Writing into Android's public Downloads collection using 
MediaStore is a platform-specific operation, so a small native bridge keeps the Flutter feature simple while delegating 
the OS-specific API to Kotlin.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q910: Why is MediaStore with IS_PENDING useful?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The native code inserts the file into Downloads with IS_PENDING set, writes the bytes, then clears the 
flag. That reduces the chance that another process sees an incomplete file while it is still being written.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q911: What happens on Android versions below API 29 in the current bridge?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The implementation returns false for versions below Android Q. A production implementation should 
define the required legacy behavior explicitly if those versions are supported, or document the minimum supported 
API level.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q912: How should a MethodChannel be designed safely?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use a stable channel name, a small set of typed method names, validate arguments, return structured 
success/error values and keep platform code focused on the platform operation. Avoid putting business logic inside the 
native bridge.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q913: What are good tests for file download?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Test missing filename/bytes, unsupported Android version, successful save, failed URI creation, write 
failure, correct MIME type, correct file name, duplicate downloads and reopening the downloaded file.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q914: Why is MIME type important?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It tells the operating system and applications how to handle the downloaded file. The bridge accepts a 
MIME type so PDF, XLSX and other generated documents can be associated correctly.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q915: What is the main memory concern when passing bytes over a MethodChannel?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Large files may be copied across the Dart/native boundary, increasing memory pressure. For large 
documents, a file-path or streaming design can be safer than passing entire byte arrays.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q916: What other file operations are present?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The project uses file picker, path provider, open_file and share_plus-style functionality, along with 
dedicated download and attachment helpers in several feature modules.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q917: How would you design a cross-platform download service?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Expose one Dart interface such as saveDocument, implement Android-specific storage behind the 
MethodChannel and provide platform-specific implementations for other targets. The rest of the app should not know 
about MediaStore.
9. Payzo Books - Notifications, Connectivity & Application Lifecycle</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q918: How are push notifications implemented?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Firebase Messaging provides the device token and foreground/opened-app streams, while 
flutter_local_notifications displays local notifications. The notification service centralizes permission requests, 
initialization and foreground handling.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q919: How is the FCM token stored?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The app retrieves the token and saves it into SharedPreferences. That token can then be used by 
application workflows that need to register the device with backend services.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q920: What happens when a notification is tapped?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The local notification initialization has a callback that receives the response and currently logs the 
payload. The Firebase opened-app listener also logs the event. A fuller product implementation would route the 
payload into the appropriate document or feature.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q921: Why is notification initialization in app startup important?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Permissions, token retrieval and listeners must be established early enough to avoid missing important 
events. The current implementation starts those operations from the root application's initialization.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q922: What would you improve in notification startup?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Await required initialization steps where ordering matters, handle permission denial explicitly, use 
stable notification-channel configuration, avoid creating a random channel on every message, and route notification 
payloads to a typed navigation event.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q923: Why is generating a random Android notification channel ID questionable?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Android notification channels are persistent OS objects. Creating a new random channel for every 
message can create many channels and makes user settings harder to manage. A stable channel ID per notification 
category is preferable.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q924: How does connectivity monitoring fit into the app?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The root MaterialApp builder wraps the application in MonitorConnectionView, providing a centralized 
place to surface connectivity state instead of every feature implementing its own offline banner.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q925: Should a connectivity widget be treated as an authoritative network state?</div>
  <div class="qa-a"><strong>Model Answer:</strong> No. Connectivity signals indicate network availability, not whether a specific API is reachable or 
whether a request succeeded. Feature logic should still handle timeouts, server failures and stale data.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q926: What is important when initializing Flutter plugins?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Ensure Flutter bindings are ready, initialize Firebase and shared services before they are used, handle 
asynchronous startup ordering, and make sure failures cannot leave the root application in an unusable state.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q927: What does ScaledWidgetsFlutterBinding do in this project?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Payzo configures a design-width-based scale factor using a 435-unit reference width. This provides a 
consistent scaling strategy across device widths, although responsive layouts still need proper constraints rather than 
relying only on global scaling.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q928: What is the risk of relying too heavily on global scaling?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A fixed design-width scaling system can distort or over-constrain layouts on very wide screens or 
unusual aspect ratios. Responsive components should still use constraints, flex layouts and breakpoints.
10. Payzo Books - Debugging, Security, Testing & Code Review</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q929: What is your debugging method for Payzo?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I trace the problem across layers: user input, provider state, controller logic, payload construction, HTTP 
request, server response, model mapping and final widget state. That prevents me from assuming the first visible UI 
error is the root cause.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q930: How would you debug a failed invoice submission?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Capture the exact validation state, inspect the generated payload, verify required IDs and numeric 
fields, compare the request with the API contract, inspect the HTTP status and structured error response, then trace the 
mapped field error back into the provider.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q931: What if the UI shows success but the invoice list does not update?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Check whether the mutation succeeded, whether the invoice list provider was invalidated, whether the 
API cache is stale, and whether navigation occurred before the refresh. Mutation completion and list consistency are 
separate concerns.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q932: What security issue should be discussed around client-side credentials?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The current generic 401 recovery path uses stored login credentials to perform a login again. A stronger 
mobile architecture would use short-lived access tokens plus refresh tokens stored securely, with password storage 
avoided entirely.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q933: What is the risk of SharedPreferences for sensitive values?</div>
  <div class="qa-a"><strong>Model Answer:</strong> SharedPreferences is convenient but not designed as a secure secret vault. Sensitive tokens or 
credentials should use platform-backed secure storage, and the server should remain authoritative for authorization.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q934: Why should request/response logging be controlled?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Accounting data can contain customers, amounts, banking information and document details. 
Development logs are useful, but production logging should redact sensitive fields, avoid full payloads and use 
structured observability with access controls.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q935: What should a secure file import validate?</div>
  <div class="qa-a"><strong>Model Answer:</strong> File type, size, content structure, expected columns, row counts, mapping, malicious content and server-
side authorization. The client should not assume a spreadsheet extension alone proves a file is safe.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q936: How would you improve invoice reference uniqueness?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Keep the client-side duplicate check for UX, but enforce uniqueness with a database constraint or 
server-side sequence. Return a conflict response that the Flutter form maps to the reference field.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q937: What should unit tests cover in an accounting app?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Money calculations, tax and discount rules, date validation, reference generation, state transitions, 
request builders and error mapping. These are deterministic and high-value because regressions can affect financial 
correctness.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q938: What should widget tests cover?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Loading/success/error/empty states, validation messages, dynamic line items, tax/discount selector 
behavior, disabled/enabled submit states and navigation outcomes for representative screens.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q939: What should integration tests cover?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Login, invoice creation, bill creation, expense creation, imports/exports, notification-driven navigation 
and representative master-data flows against a controlled backend environment.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q940: How would you review a large Payzo pull request?</div>
  <div class="qa-a"><strong>Model Answer:</strong> First understand the user-facing behavior, then check architecture boundaries, state ownership, 
payload correctness, error handling, security, performance, tests and regressions. I prefer small focused changes and 
will ask for decomposition when a PR mixes unrelated concerns.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q941: What is a good production hardening principle from this project?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Treat the mobile client as untrusted, make backend rules authoritative, minimize sensitive logging, 
centralize networking, use secure storage for secrets, add deterministic tests for financial rules, and measure 
performance before optimizing.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q942: What is one refactoring you would prioritize?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Extract shared accounting calculations and validation into pure, testable domain services. The current 
controllers contain significant orchestration and calculation logic, so moving stable rules out of UI-facing controllers 
would reduce duplication and improve testability.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q943: What is one refactoring you would avoid doing blindly?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would not rewrite the entire application into a different architecture just for style. The existing 
feature/repository/provider structure is serviceable; I would improve boundaries incrementally where duplicated logic 
or hard-to-test behavior creates a real problem.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q944: How do you discuss flaws in your own project during interviews?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I describe what the current code does, why the behavior matters, the risk it creates, and the production-
grade fix I would make. I do not pretend a code review finding was already fixed if the supplied code does not prove 
that.
PART II - FIGMA / UI-UX
11. Figma - Developer Handoff & Design Understanding</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q945: Why is Figma on your resume?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I use Figma as part of the development workflow to inspect product designs, understand visual 
hierarchy and spacing, and translate UI/UX requirements into Flutter implementations. I treat Figma as a source for 
design intent, not as a substitute for responsive engineering.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q946: What do you look for first in a Figma handoff?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I look beyond the default screenshot: layout constraints, spacing, typography, component states, 
loading/empty/error behavior, assets, interaction details, localization implications and the responsive behavior that is 
not always obvious from one frame.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q947: How do you translate a Figma design into Flutter?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I first identify the page structure and reusable components, map typography and spacing tokens, define 
responsive constraints, then implement the smallest reusable widget hierarchy. After that I compare the rendered 
result with the design and iterate on measurable differences.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q948: Why should you not blindly copy Figma pixel values?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A Figma frame represents a particular viewport. Production Flutter UI must survive different screen 
sizes, text lengths, keyboard states, accessibility settings and localization. I use the design as a reference while 
preserving responsive behavior.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q949: What is the difference between design intent and implementation detail?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Design intent is what the user should perceive: hierarchy, emphasis, spacing rhythm, interaction and 
state. Implementation detail is how Flutter achieves it. The implementation can differ from the exact Figma layer 
structure as long as the product behavior and visual intent remain correct.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q950: What are Auto Layout concepts useful to a Flutter developer?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Auto Layout communicates how elements size and position relative to each other. In Flutter, the closest 
implementation ideas are constraints, Row/Column, Expanded/Flexible, Wrap, padding, alignment and intrinsic sizing. 
The goal is to preserve the same resizing relationships.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q951: What does responsive design mean when starting from Figma?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It means identifying which elements stretch, wrap, stack, disappear or change layout at different widths. 
A single desktop or mobile frame is not enough; I look for the rules that should remain stable as the viewport changes.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q952: How do you handle a Figma design with many repeated cards?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I identify the repeated structure and build a reusable Flutter card widget with parameters for title, 
subtitle, icon, status and actions. That keeps the UI consistent and makes future design changes cheaper.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q953: How do you handle component variants in implementation?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I model the meaningful UI states explicitly, such as default, loading, disabled, selected, error and 
success. Those states become widget parameters or state-driven variants rather than duplicated one-off widgets.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q954: What Figma information is important for typography?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Font family, weight, size, line height, letter spacing and hierarchy are important. I also consider text 
scaling, overflow, wrapping and localization because the Flutter implementation must remain usable outside the exact 
design text length.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q955: How do you reproduce spacing accurately?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I use the design measurements as a starting point, establish consistent spacing tokens where the product 
uses a rhythm, and then validate the rendered UI at multiple screen sizes rather than scattering arbitrary values 
through widgets.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q956: How do you handle icons from Figma?</div>
  <div class="qa-a"><strong>Model Answer:</strong> First determine whether an icon is a platform/icon-font element or a custom asset. For custom assets I 
prefer an appropriate SVG or raster asset and preserve the intended size, tint and alignment rather than redrawing it 
unnecessarily.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q957: How do you decide whether an image should be an asset or remote data?</div>
  <div class="qa-a"><strong>Model Answer:</strong> The design file only tells me what should appear. The product requirement tells me where it comes 
from. Static branding or UI assets belong in the app bundle, while user/content images typically come from the backend 
or storage service.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q958: What do you check before exporting a Figma asset?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Correct dimensions, color space where relevant, transparent background if needed, naming, vector 
versus raster choice, file size and whether the asset needs multiple variants.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q959: What is a design token?</div>
  <div class="qa-a"><strong>Model Answer:</strong> A named design value such as color, spacing, radius or typography style that is reused consistently. 
Tokens reduce visual drift and make changes easier than hard-coding unique values on every screen.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q960: How would you turn Figma colors into Flutter theme values?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would map the product palette into ThemeData and semantic color roles rather than scattering literal 
colors across widgets. That makes light/dark or brand changes easier and improves consistency.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q961: What do you do when Figma and the API data disagree?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I first identify whether the design uses placeholder content or the real product contract. Then I preserve 
the actual business/data requirement and adapt the UI to handle realistic lengths, null values, loading and error states.
12. Figma - Responsive UI, QA & Product Collaboration</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q962: How do you validate a Flutter implementation against Figma?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Compare structure, typography, spacing, component states, iconography, alignment and interaction. I 
check at the reference viewport first, then at smaller/larger widths to make sure the implementation is not only pixel-
close at one size.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q963: How do you handle a design that looks good on iPhone but breaks on Android?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I look for hardcoded widths, unsafe text assumptions, platform-specific font metrics, asset sizing and 
keyboard/inset behavior. Then I replace fragile constraints with responsive Flutter layout primitives.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q964: How do you handle long text that is shorter in Figma?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I test long realistic strings, add wrapping or truncation where the UX requires it, and ensure 
surrounding widgets can reflow. The production UI should be driven by behavior, not by the exact placeholder text 
length.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q965: How do you handle empty, loading and error states if Figma shows only the success state?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I treat those states as part of the feature specification and design lightweight states that preserve 
hierarchy and provide clear recovery. A production screen is not complete until the non-success states are defined.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q966: What is visual regression testing?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It is the practice of comparing rendered UI against a known baseline so unintended visual changes are 
detected. Even without a dedicated visual-test tool, disciplined screenshot comparison during reviews can catch 
spacing, typography and alignment regressions.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q967: How does Figma help code review?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It gives reviewers a reference for intended UI structure and behavior. A pull request can be checked 
against the design while the code review separately checks architecture, state, API handling and tests.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q968: How do you communicate a design mismatch to a designer?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I show the exact screen/state, describe the mismatch objectively and explain whether the difference 
came from responsive behavior, API data or an implementation constraint. Then we agree whether the design or code 
should change.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q969: What if the designer asks for a 4-pixel change late in development?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I first assess whether it is a cosmetic adjustment or a token/structure issue. If it is small and local, I make 
it safely; if it reveals a shared component problem, I update the component so other screens stay consistent.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q970: How do you work with product managers on design requirements?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I clarify the user goal, required states, API dependencies, edge cases and acceptance criteria before 
implementation. Figma defines much of the visual intent, while product requirements define behavior and priority.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q971: What does good UI/UX implementation mean to you?</div>
  <div class="qa-a"><strong>Model Answer:</strong> It means the product looks coherent and also behaves correctly under real data, errors, latency, 
localization, accessibility settings and different screen sizes. Visual fidelity is necessary, but not sufficient.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q972: How do you handle Figma designs for Arabic/RTL content?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I do not simply mirror strings. I test directionality, alignment, icon placement, mixed Arabic/Latin text, 
numbers, truncation, wrapping and navigation behavior. The layout should be direction-aware rather than manually 
positioned for one language.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q973: How would you build a responsive dashboard from Figma?</div>
  <div class="qa-a"><strong>Model Answer:</strong> First identify desktop/tablet/mobile layout modes, then define breakpoints and component 
rearrangement rules. In Flutter Web I would use constraints, flexible grids, adaptive navigation and lazy content rather 
than a single fixed-width composition.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q974: What is the relationship between Figma Auto Layout and Flutter constraints?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Both describe relationships rather than only absolute coordinates. Flutter constraints are enforced by 
the layout system, so I translate the intent using parent constraints, flexible children, alignment and size limits.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q975: How do you keep a large UI system consistent?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Use shared theme values, reusable components, semantic state models, common spacing/radius rules 
and design references from Figma. Consistency should be enforced by component APIs and review practices, not 
memory.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q976: How do you handle accessibility when a Figma design has small text?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Accessibility remains a product requirement. I preserve readable contrast and scalable text, ensure hit 
targets are practical, and verify the layout under increased text scaling rather than blindly matching a small design 
label.
PART III - COMBINED PAYZO + FIGMA SCENARIOS
13. Payzo + Figma - Scenario, Behavioral & Interview Follow-Ups</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q977: Tell me about a challenging problem you solved in Payzo.</div>
  <div class="qa-a"><strong>Model Answer:</strong> A strong example is invoice and bill workflow validation. The forms depended on multiple providers 
and business rules, so I had to trace state, calculations and payload construction together. The practical lesson was to 
separate field validation, domain calculations and API submission so each layer could be debugged independently.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q978: How would you answer 'What part of Payzo would you redesign?'</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would keep the existing feature/repository/provider direction but extract shared accounting 
calculations and validation into pure domain services, standardize pagination, strengthen API error types and improve 
secure token handling. That is incremental improvement rather than a risky rewrite.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q979: What is the most important lesson from working on an accounting application?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Correctness has to be explicit. A UI can look fine while a wrong tax rate, discount mode or reference rule 
creates a financial problem. I therefore think in terms of authoritative business rules, deterministic calculations, 
validation and regression tests.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q980: How would you explain an issue you found in your own code?</div>
  <div class="qa-a"><strong>Model Answer:</strong> State the current behavior, identify the consequence, explain the evidence that led you to the root cause, 
then describe the safer fix and the regression test. That is more credible than claiming the code was perfect.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q981: What if an interviewer asks whether you built the entire Payzo application?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I would be precise: I worked on significant Flutter features and workflows within the product. I would 
not claim sole ownership of the entire codebase or every backend component.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q982: How do you prove that you understand a project rather than memorizing its code?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I can explain why the architecture is structured that way, trace data through multiple layers, discuss 
edge cases, identify trade-offs and propose a safe improvement. Those abilities are more meaningful than reciting 
filenames.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q983: How do you use Figma when requirements are ambiguous?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I use the design to clarify intended interaction and visual hierarchy, then confirm behavior with 
product requirements and API constraints. I document assumptions rather than silently choosing behavior that might 
conflict with the product.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q984: What if Figma says one thing but the existing component system says another?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I determine whether the design is a one-off or a reusable product pattern. If the new design should 
become the standard, I update the shared component or token system rather than adding another incompatible 
implementation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q985: How do you balance pixel fidelity with maintainability?</div>
  <div class="qa-a"><strong>Model Answer:</strong> Keep the visual intent accurate but implement it using reusable, responsive primitives. A pixel-perfect 
one-off screen that breaks on another device is not a successful production implementation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q986: How would you respond if QA reports 'UI doesn't match Figma'?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I reproduce the exact device/viewport and state, compare the implementation with the design, identify 
whether the difference is layout, typography, asset, data or responsive behavior, fix the root cause and re-check 
adjacent states.

Akshay Chandran P | Payzo Books + Figma Interview Q&A</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q987: What if a design works only with mock data but fails with real Payzo data?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I treat realistic data as part of the requirement. I would test long names, missing fields, large totals, 
multiple lines and slow responses, then update the component layout and state handling to survive those cases.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q988: How do you collaborate with designers and QA?</div>
  <div class="qa-a"><strong>Model Answer:</strong> I use shared terminology, reference exact screens/states, provide reproducible steps and separate visual 
issues from functional defects. The goal is to solve the product problem rather than debate whether the bug belongs to 
design or development.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q989: What is your approach when using AI to implement a Figma-driven screen?</div>
  <div class="qa-a"><strong>Model Answer:</strong> AI can accelerate scaffolding or repetitive widget code, but I validate the actual design, responsive 
behavior, state model and package APIs. I review every generated change rather than accepting a visual approximation 
without engineering validation.</div>
</div>

<div class="qa-card">
  <div class="qa-q">Q990: What would you say if asked about your strongest combination of skills?</div>
  <div class="qa-a"><strong>Model Answer:</strong> My strongest combination is Flutter/Dart development plus practical product integration: Riverpod state 
management, clean architecture, REST APIs, financial/business workflows, testing, debugging and Figma-to-Flutter UI 
implementation.</div>
</div>
