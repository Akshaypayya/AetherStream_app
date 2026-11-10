# Payzo Books: Enterprise Cloud Accounting & ERP Platform
## Master Technical Handbook: Beginner to Extreme Senior Architecture
**Project:** Payzo Books Enterprise ERP  
**Engineer:** Akshay Chandran P  
**Tech Stack:** Flutter, Dart, Riverpod, SQLite/Drift, ZATCA Phase 2 E-Invoicing, Android MediaStore Bridge, Excel/PDF Generation, 792 Dart Files

---

## Section 1: End-to-End ERP Architecture & Working Explanation

### 1.1 Beginner Level: What is Enterprise Accounting Software?
Small and medium businesses must track every financial movement: issuing sales invoices to clients, recording vendor bills, tracking payments, calculating taxes (VAT / GST), and managing inventory. If an accounting software is slow, loses an invoice, or calculates 1 cent incorrectly, the company faces tax penalties or audits.  
**Payzo Books** provides complete business accounting directly on mobile and web with 792 Dart files structured into Clean Architecture modules.

---

### 1.2 Extreme Senior Level: Modular Codebase & Riverpod State Architecture
With 792 files, monolithic state management creates unmaintainable spaghetti code. Payzo Books implements **Clean Architecture with Feature-First Modularization**:

```
lib/
├── core/                  # Database, Network Client, Storage, Theme, Currency
│   ├── database/          # Drift (SQLite) ORM with Type Converters
│   ├── network/           # Dio Interceptors, SSL Pinning, Retry Logic
│   └── errors/            # Failure models & Result<T> monads
└── features/
    ├── invoices/          # Invoices, Quotations, Credit Notes
    ├── bills/             # Vendor Bills, Expenses, Purchase Orders
    ├── inventory/         # Stock Items, Low Stock Alerts, Warehouses
    ├── zatca/             # Saudi ZATCA Phase 2 XML/QR Generator
    └── reports/           # Profit & Loss, Balance Sheet, VAT Return
```

#### Riverpod Dependency Graph
- Uses `AutoDisposeNotifierProvider` for form state so invoice line-item memory is reclaimed upon closing the editor.
- Uses `FutureProvider.family` for cached ledger lookups with background revalidation.
- `ref.watch(invoiceProvider.select((inv) => inv.totalAmount))` ensures that adding a line item updates only the summary bar without re-rendering individual table inputs.

---

### 1.3 Financial Arithmetic: The Floating-Point Catastrophe
Standard IEEE 754 floating-point math (`double` in Dart) cannot represent decimal fractions like `0.1` precisely in binary:
```dart
// The Double Trap:
double price = 100.10;
double vatRate = 0.15;
print(price * vatRate); // Prints 15.015000000000002 -> Fails tax audit!
```
**Payzo Books Solution:**
All monetary amounts are stored as integer **Minor Currency Units** (cents / fils / halalas) or processed via the `decimal` package:
```dart
final subtotal = Decimal.parse("100.10");
final vat = (subtotal * Decimal.parse("0.15")).toDecimal(scaleOnInfinitePrecision: 2);
```

---

### 1.4 ZATCA Phase 2 E-Invoicing Compliance (Saudi Tax Authority)
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

## Section 2: Code Audit Findings, ERP Architecture Risks & Fixes

### Finding 1: Memory Leaks in Multi-Page PDF Generation
- **Problem:** Generating a 100-page transaction report with embedded company logos allocated 400MB of RAM, causing low-memory crashes on budget Android devices.
- **Production Fix:** Implemented chunked PDF rendering using an Isolate worker: pages are streamed and compressed to disk incrementally rather than building the complete document in heap memory.

### Finding 2: Offline SQLite Sync Conflicts
- **Problem:** A user edited an invoice offline while an accountant edited the same invoice on the web portal. Upon reconnection, offline changes overwrote the cloud.
- **Production Fix:** Added an optimistic concurrency control column `version_id`. If `client.version_id != server.version_id`, the sync engine raises a merge conflict prompt displaying line-by-line diffs.

---

## Section 3: Enterprise Accounting & ERP Q&A

### Q1: How do you structure an offline-first sync engine in Flutter?
**Answer:**  
In Payzo Books, all read/write operations execute against local SQLite (Drift). A sync table records every mutation: `entity_id`, `table_name`, `operation` (INSERT, UPDATE, DELETE), `payload_json`, and `sync_status`. A background sync service monitors connectivity. When online, it pushes mutations sequentially inside a single database transaction. If the server accepts, the mutation is cleared; if it fails, it applies exponential backoff retry.

---

*(Handbook continues with full 40 questions covering ZATCA, MediaStore, and large-scale ERP architecture)*
