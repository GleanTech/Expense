# Expense Tracker

An offline-first, single-file HTML ledger and expense-reporting tool. All records live in the browser's local IndexedDB — nothing is uploaded, no network service is required, and no CDN is used. JSON files handle backup, migration, and handover.

[简体中文](README.md)

## Contents

- [Project highlights](#project-highlights)
- [Run and deploy](#run-and-deploy)
- [Interface tour](#interface-tour)
- [Recording expenses](#recording-expenses)
- [Accounting periods](#accounting-periods)
- [Reimbursements](#reimbursements)
- [Invoice files](#invoice-files)
- [Dictionaries](#dictionaries)
- [Management, filters, and reports](#management-filters-and-reports)
- [Data management](#data-management)
- [Locking, PIN, and permissions](#locking-pin-and-permissions)
- [Languages and themes](#languages-and-themes)
- [Offline cache (sw.js)](#offline-cache-swjs)
- [Data format](#data-format)
- [Security and privacy](#security-and-privacy)
- [Compatibility and limits](#compatibility-and-limits)
- [Project files](#project-files)
- [FAQ](#faq)

## Project highlights

| Feature | Details |
| --- | --- |
| Single-file app | `index.html` inlines all CSS, JavaScript, and UI strings; opens by double-click |
| Local-first data | Records go to browser IndexedDB, never to a server; the UI states "Data stays on this device" |
| Works offline | Hosted builds cache the app shell with `sw.js`; registration is skipped under `file://` |
| Flexible periods | Standard calendar month, custom month start day, or explicit single-trip dates |
| Business vs private | Every record is tagged business / private / family and income / expense; reimbursable records add company and status |
| Multi-currency | RMB (CNY) / EUR / NTD (TWD) / USD by default; extensible via dictionaries |
| Linked dictionaries | Two-level payment methods, categories, and companies; add, rename, delete, drag to reorder |
| Full handover | Complete JSON snapshot (settings, dictionaries, soft-deleted records) plus a standalone dictionary file; FNV-1a checksums |
| Three UI languages | Simplified Chinese / Traditional Chinese / English — interface text only, user content untouched |
| Archive lock and PIN | Locked mode freezes editing; optional 4–6 digit PIN guards against accidental access |
| Data health | Last export time and unexported record count with backup warnings |

## Run and deploy

### Recommended: hosted URL + Add to Home Screen (iPhone / iPad)

1. Push `index.html` and `sw.js` to a GitHub repository and enable GitHub Pages.
2. Open the Pages URL in Safari, then tap **Share → Add to Home Screen**.
3. Launch from the home-screen icon for app-like behavior. After the first online load it works offline.

> Opening a local HTML file from the iOS Files app is unreliable: Quick Look previews it without executing JavaScript. Use a hosted URL instead.

### Desktop

Double-click `index.html` (`file://`). It is offline by nature; data stays in that browser's IndexedDB. Service Workers are not registered under `file://`, so export JSON backups regularly.

### First-time setup

A three-step wizard:

1. **Language** — Simplified Chinese / Traditional Chinese / English.
2. **Dataset** — ledger name; optionally pick a trip plan / approval file (only its name is stored).
3. **Period** — accounting mode and its parameters, then the startup check page.

The wizard also offers **Import ledger data** to restore from a JSON snapshot.

## Interface tour

| Screen | Contents |
| --- | --- |
| Startup check | Ledger name, last record time, archive status, period, plan file, data health; **Confirm** and **Quick entry** |
| Functions | Add record / Manage records / Data management |
| Add / edit record | Full field form, invoice picker, save |
| Manage records | Filters, bulk toolbar, record list, statistics, soft-deleted section, CSV / PDF export |
| Data management | Basic settings, language / theme / sw.js location, lock and PIN, import / export, dictionary file, initialize, finish cycle |

The header always shows the ledger name, current period, and archive status badge.

## Recording expenses

| Field | Details |
| --- | --- |
| Type | Expense / income |
| Purpose | Business / private / family (independent of income vs expense) |
| Amount | Up to two decimals; stored internally as integer cents |
| Currency | Dropdown + dictionary editor; RMB / EUR / NTD / USD by default |
| Payment method | Linked two-level dropdown: APP / credit card / cash / bank transfer, with sub-items |
| Category | Linked two-level dropdown: transport / meals / lodging / telecom / private / family / office / unclassified, with sub-items |
| Reimbursement | Reimbursable / not reimbursable; company dropdown appears when reimbursable |
| Reimbursement status | Not submitted / submitted / received; meaningful for reimbursable records |
| Brief note | Single line, up to 60 characters |
| Detailed note | Auto-growing textarea, up to 2000 characters |
| Invoice file | One per record; stores only the file name and relative path |
| Record time | Date + 24-hour time, saved as ISO with local UTC offset; shown as `YYYY/MM/DD HH:mm (UTC±HH:mm)` |

- Changing a top-level dropdown auto-selects the first child of the new parent, preventing stale combinations.
- Quick entry pre-fills: expense / business / RMB / APP / unclassified / reimbursable / not submitted.
- Amount format is validated on save; a reimbursable record without a company is rejected.
- When dictionaries change, stale values in records are highlighted as needing reclassification rather than silently rewritten.

## Accounting periods

| Mode | Range |
| --- | --- |
| Standard month | Chosen year + month, from the 1st to the last day |
| Custom cycle | Chosen start day (e.g. 26th), from that day to the day before next month's start day |
| Single trip | Explicit start and end dates |

- Each record carries a `periodRef` (dataset ID, mode, start, end). **Period membership is independent of the occurrence date.**
- Switching periods counts records outside the new range and warns; no record is moved, hidden, or deleted.
- The manage page groups records as **Before period · advance purchase**, **Within period**, **After period · late / extra expense**, and **Other period**. The period filter toggles current / all.
- Report filenames include the period bounds: `ledger_start_end.csv` / `.pdf`.

## Reimbursements

- Reimbursable records require a company (four defaults: Shanghai / Germany / Guangzhou / Taiwan; editable in dictionaries).
- Status flows: not submitted → submitted → received. List badges, filters, statistics, CSV, and PDF all include it.
- **Copy as new record** duplicates all business fields with a new ID and creation time, resets the occurrence date to now (editable), clears the invoice, and resets status to not submitted.
- Multi-select bulk actions: change company (dropdown + custom value), change status, or soft delete. All ask for confirmation and show the affected count.

## Invoice files

- Pick an invoice per record (PDF / JPG / JPEG / PNG / BMP). Only its name and relative path are stored.
- **Choose folder** builds a local manifest; **Check invoice links** reselects the folder and compares. Renamed or moved files appear in red as **missing**.
- On save, duplicate invoice names are detected and the occupying record is shown; **Save anyway** is available. Soft-deleted records are excluded.
- Invoice contents are never uploaded and no thumbnails are generated; checks compare names and relative paths only.

## Dictionaries

Five dictionaries are editable from the **Edit** buttons in forms: currencies, payments, categories, reimbursements, and statuses.

- Add, rename, and delete entries. The built-in **Unclassified** entry cannot be deleted.
- Payments, categories, and reimbursements support parent + child entries. Children can be reordered by dragging the handle or with ↑ / ↓ buttons.
- Deleting a referenced entry migrates affected records to **Unclassified** after confirmation.
- Dictionary seeds are fixed in Simplified Chinese; switching UI language changes labels only, never user data.
- **Standalone dictionary file**: export / import dictionaries independently to standardize categories across devices. The import preview shows item count, export time, and checksum. Existing values in records are not rewritten; mismatches are highlighted as needing reclassification.

## Management, filters, and reports

**Filters**: keyword (brief, note, category, invoice name), top-level category, currency, company, reimbursement status, current / all periods.

**Statistics**: income, expense, and net totals per currency; empty-state placeholder. Soft-deleted records are excluded.

**Soft delete**: deleting writes `deletedAt` and removes the record from the main list. The soft-deleted section can restore or permanently purge entries (confirmed). Entries older than 30 days trigger a backup / cleanup reminder; nothing is deleted automatically.

**CSV**: UTF-8 with BOM; date, time, type, purpose, amount, currency, payment, category, reimbursement, company, status, brief, note, invoice, plus a separate soft-deleted section at the end.

**PDF**: browser print / Save as PDF with a white-background print stylesheet. Includes title, ledger name, period, active filters, table, per-currency totals, and the soft-deleted section. **Group by company** splits reimbursable records into per-company sections with per-currency subtotals.

## Data management

| Action | Rule |
| --- | --- |
| Export ledger data | Downloads the full JSON snapshot; **the only action that updates last-export time and the health baseline** |
| Copy ledger data | Copies the same JSON to the clipboard; does not update health |
| Import ledger data | Empty database only: with data present → export → initialize → import |
| Validation | Checks `schemaVersion`, structure, and FNV-1a checksum; failures are blocked with a reason |
| Import preview | After validation: ledger name, period, record count, dictionary count, UI language, lock and PIN state, checksum, export time. Overwrite happens only after confirmation |
| Import semantics | Snapshot replacement: meta, dictionaries, and records (including soft-deleted) come entirely from the file |
| Initialize data | Export reminder → confirmation → clear everything and return to first-time setup |
| Finish / close cycle | Confirmation → lock database → export JSON backup → initialize for the next cycle; also available while locked |
| Data health | Last export time + unexported count (including soft-deleted); warns at ≥7 days or ≥20 records |
| Dictionary file | Export / import a dictionary-only JSON file to standardize categories across devices |

## Locking, PIN, and permissions

Locked archive freezes the dataset after a cycle ends:

| Action | Open | Locked |
| --- | --- | --- |
| View records, statistics, soft-deleted section, health | Yes | Yes |
| Export JSON / copy JSON / CSV / PDF | Yes | Yes |
| Switch UI language | Yes | Yes |
| Add / edit / delete / copy / bulk actions | Yes | No |
| Change reimbursement status, dictionaries, period, dataset settings | Yes | No |
| Initialize / import | Yes (import only into empty DB) | No |
| Unlock | — | Confirmation + PIN if enabled |

- PIN is off by default. Enabling requires confirmation plus a 4–6 digit number; disabling requires the current PIN.
- PIN is stored as `FNV-1a(salt + pin)` — **an accidental-access guard, not cryptographic security**.
- Forgot PIN: confirmation → forced export of the current state → initialize → import the backup.

## Languages and themes

- UI languages: Simplified Chinese (`zh-CN`), Traditional Chinese (`zh-TW`), English (`en`). Switching redraws the interface, updates `lang`, and persists the choice.
- Language can be changed while locked; ledger names, notes, and dictionary values are never translated.
- Themes: follow system / light / dark. Light uses a white background and a soft Morandi green accent (`#7C9A85`); dark uses muted green-gray surfaces.

## Offline cache (sw.js)

- `sw.js` caches the app shell (`./` and `./index.html`) under `expense-tracker-v2`, using cache-first with network backfill.
- By default it registers `./sw.js`. A custom `sw.js` URL can be set in **Data management**.
- A custom URL must be a public HTTPS address **on the same origin** as the page. Cross-origin URLs are rejected; `file://` and `http://` skip registration entirely.

## Data format

IndexedDB database `expense_tracker_db` (version 1):

| Object store | Key | Indexes |
| --- | --- | --- |
| `meta` | `key` | — |
| `dictionaries` | `id` | `type`, `parentId` |
| `records` | `id` | `occurredAt`, `currency`, `categoryL1`, `reimbursementEntity`, `reimbursementStatus`, `deletedAt`, `updatedAt` |

Full snapshot:

```jsonc
{
  "schemaVersion": 1,
  "exportedAt": "2026-11-26T14:19:00+08:00",
  "checksum": { "algorithm": "fnv1a", "recordCount": 12, "value": "a1b2c3d4" },
  "meta": {
    "key": "main",
    "datasetId": "uuid",
    "datasetName": "Germany trip, Nov 2026",
    "createdAt": "...", "updatedAt": "...",
    "locked": false,
    "pinEnabled": false, "pinSalt": null, "pinHash": null,
    "timezone": "device-local",
    "uiLanguage": "en", "theme": "system",
    "swMode": "default", "swUrl": "",
    "lastExportedAt": null, "lastExportRecordCount": 0,
    "periodMode": "trip",
    "period": {
      "year": 2026, "month": 11, "customDay": 26,
      "startDate": "2026-11-26", "endDate": "2026-12-25",
      "planDocName": "trip-approval.pdf"
    }
  },
  "dictionaries": [
    { "id": "uuid", "type": "categories", "label": "Transport", "parentId": null,
      "value": "Transport", "system": false, "sortOrder": 0 }
  ],
  "records": [
    {
      "id": "uuid", "createdAt": "...", "updatedAt": "...", "deletedAt": null,
      "deviceId": "uuid",
      "periodRef": { "datasetId": "uuid", "mode": "trip", "start": "2026-11-26", "end": "2026-12-25", "planDocName": "" },
      "type": "expense", "purpose": "business",
      "amountCents": 12345, "currency": "EUR",
      "paymentL1": "Credit card", "paymentL2": "VISA1",
      "categoryL1": "Transport", "categoryL2": "Taxi",
      "reimbursable": true, "reimbursementEntity": "Germany",
      "reimbursementStatus": "unsubmitted", "reimbursementStatusDate": null,
      "brief": "Airport to hotel", "note": "",
      "invoice": { "name": "invoice-001.pdf", "relativePath": "invoices/invoice-001.pdf", "status": "unchecked", "checkedAt": null },
      "occurredAt": "2026-11-26T09:30:00+08:00"
    }
  ]
}
```

Dictionary file:

```jsonc
{
  "schemaVersion": 1,
  "type": "dictionary",
  "exportedAt": "2026-11-26T14:19:00+08:00",
  "checksum": { "algorithm": "fnv1a", "itemCount": 60, "value": "d4c3b2a1" },
  "dictionaries": [ /* same dictionary items as above */ ]
}
```

Checksum: canonical JSON (sorted keys) over the fixed fields, hashed with 32-bit FNV-1a as 8 hex digits.

## Security and privacy

- No backend, telemetry, or CDN. GitHub hosts static files only.
- Invoices are referenced by name and path; file contents are never read or uploaded.
- PIN is a front-end hash that prevents accidental access only — it cannot stop someone with deliberate access to the device.
- Data does not travel with the HTML file: export JSON before switching devices, browsers, or clearing site data.
- Safari may evict storage for sites that are not visited. Home-screen installation plus regular exports reduce the risk.

## Compatibility and limits

- Safari iOS 16+ (home-screen mode recommended), Chrome / Edge desktop.
- Responsive layout for 320 / 768 / 1440 px; 44 px minimum touch targets, keyboard and screen-reader labels.
- Designed for thousands of records; bulk writes use IndexedDB transactions.
- No automatic multi-device sync (`deviceId` / `updatedAt` / `deletedAt` are reserved but unused in this version).
- Invoice files must remain at their original paths for link checks; the app does not bundle originals.

## Project files

```text
.
├── index.html   # application shell; inline CSS / JS / UI strings
├── sw.js        # optional Service Worker for offline shell caching
├── README.md    # Simplified Chinese documentation
└── README.en.md # this document
```

## FAQ

**Why is the Import button disabled?**
A database with data cannot be overwritten. Export ledger data, initialize the database, then import. Validation runs first; a preview must be confirmed before overwrite.

**Did my records disappear after switching periods?**
No. The manage page defaults to the current period. Switch the period filter to **All periods** to find records grouped under before / after / other periods.

**Why does an exported JSON fail validation?**
The file was edited or incompletely downloaded. Use the original file produced by this app; do not manually change `exportedAt`, `checksum`, or record content.

**Why doesn't the unexported count drop after copying JSON?**
Only **Export ledger data** (download) updates the baseline. **Copy ledger data** intentionally leaves it unchanged.

**Why won't a local HTML file open on iPhone?**
iOS Files Quick Look does not execute JavaScript. Use a hosted URL (e.g. GitHub Pages), open it in Safari, and add it to the home screen.

**Why does a custom sw.js URL fail to register?**
It must be same-origin and public HTTPS. `file://`, `http://`, localhost, private IPs, and cross-origin URLs are all rejected.
