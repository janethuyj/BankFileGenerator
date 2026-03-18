# NZ Westpac Bank File Generators

Single-file browser tools for generating New Zealand Westpac bank payment files for upload to Westpac Corporate Online.

No installation, no server, no dependencies — open the HTML file in any modern browser and it works.

---

## Tools

### `nz_westpac_generator.html` — Direct Entry (180-char format)

Generates Westpac NZ Direct Entry files per the August 2011 internal spec.

- Fixed-length **180-character** records
- `A` header record + one `D` detail record per transaction
- Output filename: `NZ_DE_DDMMYY.txt`

### `nz_westpac_bacho_generator.html` — BACHO (160-char format)

Generates Westpac NZ BACHO (Bankers' Association Clearing House Organisation) files per the official spec v3.1, November 2019.

- Fixed-length **160-character** records
- `A` header + `D` detail records + `D` balancing record + `T` trailer with hash total
- DE User ID field (allocated by Westpac), full 4-digit year date format (`DDMMCCYY`)
- Output filename: `NZ_BACHO_DDMMYYYY.txt`

---

## Features (both tools)

- **Direct Credits** (transaction codes `52` Payroll and `50` General Payment) and **Direct Debits** (transaction code `00`)
- Top-level transaction mode toggle — switch between Direct Credits and Direct Debits
- Import transactions from CSV
- Live file preview with labelled field breakdown per record
- Download the generated `.txt` file ready for bank upload
- Per-row payer account override support
- Validation with clear error messages before generation

---

## Usage

1. Open the relevant HTML file in a browser
2. **Step 1** — Select **Direct Credits** or **Direct Debits**, set the processing date, company/customer name, and your Westpac origin branch (BACHO also requires a DE User ID)
3. **Step 2** — Enter your account details (trading name, bank, branch, account, suffix)
4. **Step 3** — Import a CSV file containing payee transactions
5. **Step 4** — Click **⚡ Generate & Preview**, review the output, then **⬇ Download .txt**

---

## CSV Format

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

---

## File Formats

### Direct Entry (nz_westpac_generator.html)

- Fixed-length records, exactly **180 characters** per record
- CRLF (`\r\n`) line endings
- One `A` (header) record followed by one `D` (detail) record per transaction
- Amounts stored as cents (integer, no decimal point)
- Output filename: `NZ_DE_DDMMYY.txt`

### BACHO (nz_westpac_bacho_generator.html)

- Fixed-length records, exactly **160 characters** per record
- CRLF (`\r\n`) line endings
- One `A` (header) + one `D` (detail) per transaction + one `D` (balancing) + one `T` (trailer)
- Hash total in T record: sum of all D record account fields, rightmost 7 digits
- Amounts stored as cents (integer, no decimal point)
- Output filename: `NZ_BACHO_DDMMYYYY.txt`

---

## Deliverables

```
BankFileGenerator/
├── nz_westpac_generator.html       ← Direct Entry (180-char), open in any modern browser
└── nz_westpac_bacho_generator.html ← BACHO (160-char), open in any modern browser
```
