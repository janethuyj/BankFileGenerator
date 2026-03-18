# NZ Westpac Direct Entry File Generator — Implementation Plan

## Overview

A self-contained single HTML file tool for finance teams to generate, import, and
re-download New Zealand Westpac Direct Entry bank files (fixed-length format) for
upload to Westpac Corporate Online.

**Target path:** `C:\WorkSpace\BankFileGenerator\nz_westpac_generator.html`
**Tech stack:** Vanilla HTML + CSS + JavaScript — no framework, no server, no install
**Phase:** 1 — Direct Credits only (Direct Debits deferred to Phase 2)

---

## File Format Reference

Source spec: Westpac Corporate Online — New Zealand Direct Entry file format (August 2011)

### Format rules

- Fixed-length records, exactly **180 characters** per record
- Records separated by `\r\n` (CRLF)
- Field padding: left-justified = blank-filled right; right-justified = zero-filled left
- Amounts in **cents** as integer (no decimal point), right-justified, zero-filled

### Header Record `A` — 180 chars

| Pos   | Len | Field           | Rule                                      |
|-------|-----|-----------------|-------------------------------------------|
| 1     | 1   | Record type     | Hardcoded `A`                             |
| 2–7   | 6   | Sequence number | Right-justified, zero-filled. Always `000001` for single-header files |
| 8–9   | 2   | Origin Bank     | Hardcoded `03` (Westpac NZ)               |
| 10–13 | 4   | Origin Branch   | User input. Right-justified, zero-filled  |
| 14–43 | 30  | Customer name   | Optional. Left-justified, blank-filled    |
| 44–49 | 6   | Customer number | Not used. Must be blank-filled            |
| 50–69 | 20  | Description     | Left-justified, blank-filled. Appears as "Other Party Name" on bank statement |
| 70–75 | 6   | Date            | `DDMMYY`. Must be a valid date            |
| 76–180| 105 | Blank           | Blank-filled                              |

### Detail Record `D` — 180 chars

| Pos     | Len | Field              | Rule                                                |
|---------|-----|--------------------|-----------------------------------------------------|
| 1       | 1   | Record type        | Hardcoded `D`                                       |
| 2–7     | 6   | Sequence number    | Right-justified, zero-filled. Auto-incremented from 2 |
| 8–9     | 2   | Payee Bank         | Valid NZ bank number. Right-justified, zero-filled  |
| 10–13   | 4   | Payee Branch       | Right-justified, zero-filled                        |
| 14–21   | 8   | Payee Account      | Right-justified, zero-filled                        |
| 22–25   | 4   | Payee Suffix       | Right-justified, zero-filled                        |
| 26–27   | 2   | Transaction code   | `52` = Payroll, `50` = General Payment              |
| 28–29   | 2   | MTS Source         | `DC` for Direct Credits (Phase 1 only)              |
| 30–44   | 15  | Amount             | Cents, no decimal. Right-justified, zero-filled     |
| 45–64   | 20  | Payee name         | Left-justified, blank-filled                        |
| 65–76   | 12  | Payee Particulars  | Appears on payee's statement. Left-justified, blank-filled |
| 77–88   | 12  | Payee Analysis code| Appears on payee's statement. Left-justified, blank-filled |
| 89–100  | 12  | Payee Reference    | Appears on payee's statement. Left-justified, blank-filled |
| 101–102 | 2   | Payer Bank         | Right-justified, zero-filled                        |
| 103–106 | 4   | Payer Branch       | Right-justified, zero-filled                        |
| 107–114 | 8   | Payer Account      | Right-justified, zero-filled                        |
| 115–118 | 4   | Payer Suffix       | Right-justified, zero-filled                        |
| 119–138 | 20  | Payer Name         | Left-justified, blank-filled                        |
| 139–180 | 42  | Spare              | Blank-filled                                        |

---

## Application Structure

Single HTML file with three logical areas:

```
nz_westpac_generator.html
├── <style>     — All CSS (DM Sans + DM Mono fonts from Google Fonts)
├── <body>
│   ├── <header>            — Logo, title, badges
│   └── .app-layout         — CSS Grid: 380px left | 1fr right
│       ├── .left-panel     — Scrollable form area (Steps 1–4)
│       └── .right-panel    — Sticky live preview
└── <script>    — All JavaScript state + logic
```

---

## UI Layout Detail

### Header bar (56px, sticky)
- Left: Logo mark "DE" + title "NZ Direct Entry Generator" + subtitle
- Right: Badges "DIRECT CREDITS" and "180 CHAR / RECORD"

### Left panel — 4 numbered steps

#### Step 1 — File Setup & Header
- **Payment Type toggle** (tab group): `Payroll (52)` | `General Payment (50)`
  - Sets `currentTxCode` state variable (`'52'` or `'50'`)
- **Processing Date** — `<input type="date">`, defaults to today, formatted as `DDMMYY` on output
- **File Description** — text input, max 20 chars, appears as "Other Party Name" on statement
- **Company Name** — text input, max 30 chars, optional
- **Origin Branch** — text input, max 4 digits (Origin Bank always `03`)

#### Step 2 — Default Payer Account
- **Payer Name** — text input, max 20 chars
- **Bank** (2) / **Branch** (4) / **Account** (8) / **Suffix** (4) — inline grid row
- Helper text: "Individual rows may override these defaults"

#### Step 3 — Transactions
- **Import mode tabs**: `Import CSV / Excel` | `Load Bank File (.txt)`

  **CSV Import panel:**
  - Drag-and-drop zone — accepts `.csv` and `.xlsx`
  - Hidden `<input type="file">` triggered by clicking zone
  - XLSX files: show message asking user to save as CSV first
  - Buttons: `⬇ Download CSV Template` | `✕ Clear All`
  - Status message area (hidden until file processed)

  **Bank File Import panel:**
  - Drag-and-drop zone — accepts `.txt`
  - Hidden `<input type="file">` triggered by clicking zone
  - On successful parse: auto-fills Step 1 & Step 2 fields from A record / first D record
  - Status message area

- **Transaction table** (always visible below import panels):
  - Columns: `#` | `Payee Name` | `Bank-Br-Acc-Sfx` | `Amount` | `Particulars` | `Payer` | (delete)
  - "Payer" column shows `custom` badge if row has payer overrides, else `default`
  - Empty state message when no transactions
  - Each row has `✕` delete button

#### Step 4 — Validate & Download
- Validation error/warning alert area
- `⚡ Generate & Preview` button — always visible
- `⬇ Download .txt` button — hidden until successful generation

### Right panel — File Preview
- **Preview header**: title + stats bar (Records count, Total Amount, File Size in bytes)
  - Stats hidden until first successful generation
- **Preview body**: scrollable
  - Before generation: placeholder with icon + instructions
  - After generation: one card per record
    - Card header: record type tag (gold for A, blue for D) + sequence number
    - Card fields: labelled field grid showing parsed values
    - Card raw: monospace 180-char raw line
  - Amount displayed as `$X,XXX.XX` in green

---

## JavaScript Architecture

### State variables

```javascript
let transactions = [];      // Array of transaction objects (from CSV or bank file parse)
let generatedContent = '';  // The final .txt file string, set on successful generate
let currentTxCode = '52';   // '52' (payroll) or '50' (general) — set by tab toggle
```

### Padding helpers

```javascript
const padL = (s, n) => String(s||'').substring(0,n).padEnd(n,' ');    // left-justify, blank-fill
const padR = (s, n) => String(s||'').substring(0,n).padStart(n,'0'); // right-justify, zero-fill
const padBlank = (n) => ' '.repeat(n);
```

### Core functions

| Function | Purpose |
|---|---|
| `buildHeaderRecord()` | Reads Step 1 form fields → returns exactly 180-char A record string |
| `buildDetailRecord(tx, seq)` | Takes transaction object + sequence number → returns 180-char D record |
| `validateAll()` | Returns `{ errors[], warns[] }` — checks all required fields and formats |
| `validateAndGenerate()` | Runs validation; on pass builds full file string, calls `renderPreview()` |
| `renderPreview(lines)` | Renders preview cards in right panel, updates stats bar |
| `downloadFile()` | Triggers browser download of `generatedContent` as `NZ_DE_DDMMYY.txt` |
| `downloadTemplate()` | Triggers browser download of blank CSV template |
| `parseBankFile(file)` | FileReader → parse A+D records → fill form fields + `transactions[]` |
| `parseCsv(text)` | Parse CSV text → validate headers → return row objects array |
| `loadTransactionsFromRows(rows, filename)` | Set `transactions[]`, call `refreshTxTable()`, show status |
| `refreshTxTable()` | Re-render the transaction table tbody from `transactions[]` |
| `removeTransaction(i)` | Splice from `transactions[]`, re-render table |
| `clearTransactions()` | Empty `transactions[]`, reset preview, hide download button |
| `setTxCode(code, btn)` | Update `currentTxCode`, update active tab style |
| `switchImportTab(tab, btn)` | Toggle CSV / bank file import panels |
| `refreshPreview()` | Called on any form field change — clears `generatedContent` and hides download button |
| `showStatus(id, type, msg)` | Show coloured alert in import status area |
| `escHtml(s)` | Sanitise strings for HTML output |
| `triggerDownload(content, filename, mime)` | Blob URL download helper |

### Amount handling
- UI accepts dollars with decimals e.g. `5491.13`
- Convert to cents: `Math.round(parseFloat(amount) * 100)`
- Display back as dollars: `(cents / 100).toFixed(2)`

### Date handling
- `<input type="date">` gives `YYYY-MM-DD`
- Convert to `DDMMYY` for A record: `dd + mm + yy.substring(2)`
- When parsing bank file date `DDMMYY` → convert back to `YYYY-MM-DD` for the input:
  - Year: `parseInt(yy) < 50 ? '20' + yy : '19' + yy`

### Sequence numbers
- Header A record: always `000001`
- Detail D records: start at `000002`, increment per row

---

## CSV Template Format

### Downloadable template columns

```
payee_name, payee_bank, payee_branch, payee_account, payee_suffix,
amount, particulars, analysis_code, reference,
payer_bank*, payer_branch*, payer_account*, payer_suffix*, payer_name*
```
*optional — overrides global payer defaults for that row only

### Example row (matches sample bank file)

```csv
payee_name,payee_bank,payee_branch,payee_account,payee_suffix,amount,particulars,analysis_code,reference,payer_bank,payer_branch,payer_account,payer_suffix,payer_name
Mr J B Hewson,12,3125,00074656,0000,5491.13,Resimac  Pay,,,,,,,
```

### CSV parsing rules
- First row must be header row with exact column names (lowercase, underscores)
- Required columns: `payee_name`, `payee_bank`, `payee_branch`, `payee_account`, `payee_suffix`, `amount`
- Optional columns: `particulars`, `analysis_code`, `reference`, and all `payer_*` fields
- Per-row payer override: only applied if at least `payer_bank`, `payer_branch`, `payer_account` are all non-empty
- Quoted CSV values supported
- XLSX: show message to save as CSV first (no external library dependency)

---

## Bank File (.txt) Import Rules

When loading an existing Westpac DE file:

1. **A record** → pre-fill form fields:
   - Origin Branch → Step 1 Branch field
   - Customer Name → Step 1 Company Name field
   - Description → Step 1 Description field
   - Date → Step 1 Date field (convert `DDMMYY` → `YYYY-MM-DD`)

2. **First D record** → pre-fill default payer fields (Step 2):
   - Payer Bank, Branch, Account, Suffix, Name
   - Then clear those fields on the first transaction object so it uses defaults

3. **Subsequent D records** → if payer fields match the defaults → clear them (use default)

4. **Transaction code** detected from `D` record positions 26–27 → set `currentTxCode` and update toggle UI

5. User can then: view transactions in table, delete individual rows, modify Step 1/2 fields, and re-download

---

## Validation Rules

### Errors (block generation)

| Check | Message |
|---|---|
| No processing date | "Processing date is required" |
| No origin branch | "Origin branch is required" |
| Origin branch non-numeric | "Origin branch must be numeric (up to 4 digits)" |
| No payer bank | "Default payer bank is required" |
| No payer branch | "Default payer branch is required" |
| No payer account | "Default payer account is required" |
| No payer name | "Default payer name is required" |
| No transactions | "No transactions — import a CSV or bank file" |
| Row: missing payee name | "Row N: Payee name is missing" |
| Row: non-numeric payee bank | "Row N: Payee bank must be numeric" |
| Row: non-numeric payee branch | "Row N: Payee branch must be numeric" |
| Row: non-numeric payee account | "Row N: Payee account must be numeric" |
| Row: invalid amount | "Row N: Amount must be a positive number" |

### Warnings (allow generation, shown after success)

| Check | Message |
|---|---|
| Empty description | "File description is empty — recommended for bank statement reconciliation" |

---

## Output File

- Filename: `NZ_DE_DDMMYY.txt` (using the processing date)
- Encoding: UTF-8 (bank expects ASCII — only use standard ASCII characters in inputs)
- Line endings: `\r\n` (CRLF) after every record including the last
- Each line: exactly 180 characters before the CRLF

### Example output (matches provided sample file)

```
A000001030175RESIMAC FINANCIAL SECURITIES                            120326                                                                                                         
D00000212312500074656000052DC000000000549113Mr J B Hewson       Resimac  Pay                        030175004895300000RESIMAC                                                       
D00000303150000038157000052DC000000000955593A  JB Rao           Resimac  Pay                        030175004895300000RESIMAC                                                       
```

> Note: sequence numbers in D records start at 2 (header takes 1)

---

## Styling Guide

- **Font**: `DM Sans` (UI) + `DM Mono` (monospace fields, raw preview) — Google Fonts
- **Theme**: Light professional
- **Accent colour**: Amber/orange `#b45309` (matches finance/banking gravitas)
- **Surface**: White cards on `#f5f4f0` warm off-white background
- **Borders**: Subtle `#e2e0d8`
- **Record type tags**: Gold for A records, Blue for D records
- **Amounts**: Green `#166534`
- **Error/warn/success/info**: Standard semantic colours with light backgrounds

---

## Out of Scope (Phase 1)

The following are deferred to future phases:

- Direct Debits (transaction code `00`, MTS source `DD`)
- Multiple header records within one file (different value dates per batch)
- XLSX import (user saves as CSV instead)
- Row-level editing in the transaction table (delete only in Phase 1)
- Audit log of changes to imported files
- Multi-file batch generation
- Any server-side component

---

## Deliverable

```
C:\WorkSpace\BankFileGenerator\
└── nz_westpac_generator.html    ← single self-contained file
```

Open in any modern browser. No build step, no dependencies to install, no network required after initial load (Google Fonts loads from CDN — can be inlined if offline use is needed).