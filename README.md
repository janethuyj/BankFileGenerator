# Bank File Generators

Single-file browser tools for generating bank payment files for upload to Westpac Corporate Online (NZ and AU).

No installation, no server, no dependencies — open the HTML file in any modern browser and it works.

---

## Tools

### `nz_westpac_generator.html` — NZ Direct Entry (180-char format)

Generates Westpac NZ Direct Entry files per the August 2011 internal spec.

- Fixed-length **180-character** records
- `A` header record + one `D` detail record per transaction
- Output filename: `NZ_DE_DDMMYY.txt`

### `nz_westpac_bacho_generator.html` — NZ BACHO (160-char format)

Generates Westpac NZ BACHO (Bankers' Association Clearing House Organisation) files per the official spec v3.1, November 2019.

- Fixed-length **160-character** records
- `A` header + `D` detail records + `D` balancing record + `T` trailer with hash total
- DE User ID field (allocated by Westpac), full 4-digit year date format (`DDMMCCYY`)
- Output filename: `NZ_BACHO_DDMMYYYY.txt`

### `au_westpac_generator.html` — AU ABA Direct Entry (120-char format)

Generates Australian ABA (Australian Bankers' Association) Direct Entry files per the Westpac AU Corporate Online spec, April 2015.

- Fixed-length **120-character** records
- `0` header + `1` detail records + `7` trailer
- Supports mixed credits and debits in a single file — transaction code is per-record
- Separate credit and debit totals in trailer; net total = `|credits − debits|`
- BSB format `DDD-DDD`; account numbers right-justified, blank-filled (alphanumeric)
- APCA User ID (6-digit, allocated by Australian Payments Clearing Association)
- Trace record (your own BSB + account) embedded in each detail record
- Lodgement reference (18 chars) and remitter name (16 chars) per detail record
- **Mixed mode** — set Default Transaction Code to "Mixed" and supply `transaction_code` per CSV row
- Output filename: `AU_DE_DDMMYY.txt`

---

## Features

### NZ tools (both)

- **Direct Credits** (transaction codes `52` Payroll and `50` General Payment) and **Direct Debits** (transaction code `00`)
- Top-level transaction mode toggle — switch between Direct Credits and Direct Debits
- Import transactions from CSV
- Live file preview with labelled field breakdown per record
- Download the generated `.txt` file ready for bank upload
- Per-row payer account override support
- Validation with clear error messages before generation

### AU ABA tool

- Mixed DC/DD in one file — each row can be credit (codes `50`–`57`) or debit (code `13`)
- Default Transaction Code dropdown: `50 General Credit`, `53 Pay/Salary`, `13 Debit`, or `Mixed`
- In Mixed mode: each CSV row must supply `transaction_code`; rows without one fail validation
- In specific-code mode: per-row `transaction_code` overrides the default with a warning
- Live file preview with per-field colour coding (`0`=gold, `1`=blue, `7`=green)
- Download template CSV with AU-specific columns

---

## Usage

### NZ tools

1. Open the relevant HTML file in a browser
2. **Step 1** — Select **Direct Credits** or **Direct Debits**, set the processing date, company/customer name, and your Westpac origin branch (BACHO also requires a DE User ID)
3. **Step 2** — Enter your account details (trading name, bank, branch, account, suffix)
4. **Step 3** — Import a CSV file containing payee transactions
5. **Step 4** — Click **⚡ Generate & Preview**, review the output, then **⬇ Download .txt**

### AU ABA tool

1. Open `au_westpac_generator.html` in a browser
2. **Step 1** — Set processing date, reel sequence, financial institution, name of user, APCA User ID, file description, and default transaction code
3. **Step 2** — Enter your BSB, account number, and remitter name (trace record fields)
4. **Step 3** — Import a CSV file containing payee transactions
5. **Step 4** — Click **⚡ Generate & Preview**, review the output, then **⬇ Download .txt**

---

## CSV Format

### NZ tools

Download the template from the app, or create a CSV with these columns:

| Column | Required | Description |
|---|---|---|
| `payee_name` | Yes | Payee name (max 20 chars) |
| `payee_bank` | Yes | Payee bank number (2 digits) |
| `payee_branch` | Yes | Payee branch number (4 digits) |
| `payee_account` | Yes | Payee account number (up to 8 digits) |
| `payee_suffix` | Yes | Payee account suffix (up to 4 digits) |
| `amount` | Yes | Amount in dollars e.g. `1500.00` |
| `particulars` | No | Appears on payee's statement (max 12 chars) |
| `analysis_code` | No | Appears on payee's statement (max 12 chars) |
| `reference` | No | Appears on payee's statement (max 12 chars) |
| `payer_bank` | No | Override default payer bank for this row |
| `payer_branch` | No | Override default payer branch for this row |
| `payer_account` | No | Override default payer account for this row |
| `payer_suffix` | No | Override default payer suffix for this row |
| `payer_name` | No | Override default payer name for this row |

Per-row payer override is applied only when `payer_bank`, `payer_branch`, and `payer_account` are all provided.

### AU ABA tool

Download the template from the app, or create a CSV with these columns:

| Column | Required | Description |
|---|---|---|
| `payee_name` | Yes | Title of account / payee name (max 32 chars) |
| `payee_bsb` | Yes | Payee BSB in format `032-000` |
| `payee_account` | Yes | Payee account number (max 9 chars, alphanumeric) |
| `amount` | Yes | Amount in dollars e.g. `1500.00` (max `99999999.99`) |
| `transaction_code` | No* | Transaction code: credits `50`–`57`, debit `13`. **Required when mode is Mixed** |
| `lodgement_reference` | No | Appears on payee's bank statement (max 18 chars) |
| `remitter_name` | No | Override remitter name for this row (max 16 chars) |
| `withholding_tax` | No | Withholding tax in dollars (default `0.00`) |
| `indicator` | No | Blank normally; `N` for new BSB/name details |

\* `transaction_code` is required per row when the Default Transaction Code is set to **Mixed**.

---

## File Formats

### NZ Direct Entry (nz_westpac_generator.html)

- Fixed-length records, exactly **180 characters** per record
- CRLF (`\r\n`) line endings
- One `A` (header) record followed by one `D` (detail) record per transaction
- Amounts stored as cents (integer, no decimal point)
- Output filename: `NZ_DE_DDMMYY.txt`

### NZ BACHO (nz_westpac_bacho_generator.html)

- Fixed-length records, exactly **160 characters** per record
- CRLF (`\r\n`) line endings
- One `A` (header) + one `D` (detail) per transaction + one `D` (balancing) + one `T` (trailer)
- Hash total in T record: sum of all D record account fields, rightmost 7 digits
- Amounts stored as cents (integer, no decimal point)
- Output filename: `NZ_BACHO_DDMMYYYY.txt`

### AU ABA Direct Entry (au_westpac_generator.html)

- Fixed-length records, exactly **120 characters** per record
- CRLF (`\r\n`) line endings
- Record sequence: one `0` (header) + one `1` (detail) per transaction + one `7` (trailer)
- Amounts stored as cents (integer, no decimal point)
- Trailer contains separate credit total, debit total, and net total (`|credits − debits|`)
- Processing date format: `DDMMYY`
- Output filename: `AU_DE_DDMMYY.txt`

#### AU Record Layout Summary

| Record | Type | Length | Key Fields |
|--------|------|--------|------------|
| Header | `0` | 120 | Reel seq, financial institution, name of user, APCA User ID, file description, processing date |
| Detail | `1` | 120 | Payee BSB, payee account, transaction code, amount, payee name, lodgement reference, trace BSB, trace account, remitter name, withholding tax |
| Trailer | `7` | 120 | Net total, credit total, debit total, record count |

---

## Deliverables

```
BankFileGenerator/
├── nz_westpac_generator.html       ← NZ Direct Entry (180-char), open in any modern browser
├── nz_westpac_bacho_generator.html ← NZ BACHO (160-char), open in any modern browser
└── au_westpac_generator.html       ← AU ABA Direct Entry (120-char), open in any modern browser
```
