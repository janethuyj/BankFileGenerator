# Direct Debit Support — Design Document
**Date:** 2026-03-18
**Branch:** DDFile

---

## Context

Phase 1 of the NZ Westpac Direct Entry Generator supports Direct Credits only (MTS source `DC`, transaction codes `52` and `50`). This document covers Phase 2: adding Direct Debit support (MTS source `DD`, transaction code `00`).

The D record structure is identical — only positions 26–27 (transaction code) and 28–29 (MTS source) differ.

---

## Design: Option B — Top-level DC / DD mode switcher

### New state variable

```javascript
let currentMode = 'DC';  // 'DC' or 'DD'
```

`currentMts` is derived: `currentMode === 'DD' ? 'DD' : 'DC'`
`currentTxCode` for DD is always `'00'`; for DC it remains `'52'` or `'50'` as controlled by the existing sub-toggle.

---

### UI Changes

#### Header bar
- `DIRECT CREDITS` badge → updates to `DIRECT DEBITS` when DD mode is active

#### Step 1 — new top-level toggle (above Payment Type)
```
[ Direct Credits ]  [ Direct Debits ]
```
- When **Direct Credits**: show existing `Payroll (52) | General Payment (50)` sub-toggle
- When **Direct Debits**: hide the sub-toggle row entirely; tx code is locked to `00`

#### Step 2 — label renaming in DD mode
| Element | DC label | DD label |
|---|---|---|
| Step card title | Default Payer Account | Default Creditor Account |
| Name input label | Payer Name | Creditor Name |
| Table column header | Payer | Creditor |

Input IDs and form field IDs remain unchanged — only visible text changes.

#### Transaction table column header
- `Payer` column header → `Creditor` in DD mode

---

### JavaScript Changes

#### `setMode(mode)` — new function
```javascript
function setMode(mode) {
  currentMode = mode;
  // Show/hide sub-toggle
  // Update badge
  // Update Step 2 labels
  // If switching to DD, set currentTxCode = '00'
  // If switching to DC, restore currentTxCode to last DC code
  refreshPreview();
}
```

#### `buildDetailRecord(tx, seq)` — update MTS source
```javascript
const mts = currentMode === 'DD' ? 'DD' : 'DC';
const txCode = currentMode === 'DD' ? '00' : (tx.tx_code || currentTxCode);
```

#### `renderPreview` — D record card update
- MTS field in card will show `DD` correctly (already reads from the raw line)

---

### Files Modified
- `nz_westpac_generator.html` — only file to change

---

## Out of Scope
- Mixing DC and DD records in the same file
- Per-row mode override
