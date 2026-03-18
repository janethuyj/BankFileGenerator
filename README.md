# NZ Westpac Direct Entry Generator

A single-file browser tool for generating New Zealand Westpac Direct Entry bank files for upload to Westpac Corporate Online.

No installation, no server, no dependencies — open the HTML file in any modern browser and it works.

---

## Features

- Generate fixed-length 180-character Direct Entry files (NZ Westpac format)
- Import transactions from CSV
- Live file preview with labelled field breakdown per record
- Download the generated `.txt` file ready for bank upload
- Per-row payer account override support
- Validation with clear error messages before generation

---

## Usage

1. Open `nz_westpac_generator.html` in a browser
2. **Step 1** — Set the processing date, file description, company name, and your Westpac origin branch
3. **Step 2** — Enter your default payer bank account details
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

## File Format

Generates Westpac NZ Direct Entry format (August 2011 spec):

- Fixed-length records, exactly **180 characters** per record
- CRLF (`\r\n`) line endings
- One `A` (header) record followed by one `D` (detail) record per transaction
- Amounts stored as cents (integer, no decimal point)
- Output filename: `NZ_DE_DDMMYY.txt`

---

## Scope

**Phase 1 (current):** Direct Credits only — transaction codes `52` (Payroll) and `50` (General Payment)

Deferred to future phases: Direct Debits, XLSX import, row-level editing, multiple header records.

---

## Deliverable

```
BankFileGenerator/
└── nz_westpac_generator.html    ← open in any modern browser
```
